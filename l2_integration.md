# Cypherpunk Finance: L2 Integration and Multi-Network App Architecture

## Introduction

This document outlines how Layer 2 (L2) scaling solutions are integrated into Cypherpunk Finance (primarily as optional local node plugins), and how application plugins (like DeFi apps) can support and interact with these L2 networks and Ethereum L1 using a centralized configuration approach managed by CypherpunkOS.

## L2s as Plugins (for Local Hosting Option)

For users wishing to run their own L2 nodes, Cypherpunk Finance implements each supported L2 network as an individual, installable plugin. This provides a local RPC source if the user chooses it over an external one.

### L2 Node Plugin Responsibilities:

1.  **Node Software Management:** Package and run the official node software for the specific L2.
2.  **RPC Endpoint Exposure:** Expose the L2 node's RPC endpoint(s) via `exports.sh` for CypherpunkOS to potentially use.
3.  **Status and Log Reporting:** Implement the standard Cypherpunk App Status API (`/app-status`) and log to `stdout`/`stderr`.
4.  **Dependency on L1 Node:** L2 nodes require an L1 Ethereum node RPC endpoint to function. The L2 plugin's `cypherpunk-app.yml` manifest must declare a dependency indicating its need for an L1 RPC (e.g., `"ethereum_l1_rpc_available": true` or similar generic capability identifier). CypherpunkOS will then provide the *effective* L1 RPC endpoint (user-configured external L1 RPC, or the RPC from the active local L1 node provider like Nodeset) to the L2 plugin via a standardized environment variable (e.g., `APP_ETHEREUM_L1_RPC_URL`).
5.  **Updates:** Manage updates for the L2 node software it contains.

### L2 Node Plugin Manifest (`cypherpunk-app.yml` example - Base L2):

```yaml
manifestVersion: 1
id: "l2-base" # Unique ID for this L2 plugin
name: "Base Node (Local)"
version: "1.0.0"
plugin_type: "node"
description: "Runs a local Base L2 node, providing a local RPC endpoint for the Base network if selected by the user."
developer: "Cypherpunk Finance Community"
website: "https://base.org"
repo: "<link_to_cypherpunk_l2_base_plugin_repo>"
support: "<link_to_support_channel>"
dependencies:
  # Declares a need for an L1 RPC provider to be configured in CypherpunkOS.
  # It doesn't hardcode "nodeset". CypherpunkOS will provide the effective L1 RPC.
  - "ethereum_l1_rpc_provider_available" # Or a similar generic identifier

port: 8545 # Default internal L2 RPC port for op-node
path: "/l2-nodes/base"
status_endpoint: "/app-status"

# Variables this plugin will export for CypherpunkOS to use if this local node is active and prioritized for this L2 type
exports:
  - "APP_L2_BASE_RPC_URL=http://op_node_base:8545" # Example internal service name and port
  - "APP_L2_BASE_CHAIN_ID=8453"
  - "APP_L2_BASE_NETWORK_NAME=Base Mainnet (Local Provider)"

# Describes the network this plugin *can provide locally*
network_provided:
  id: "base_mainnet" # Network ID CypherpunkOS uses to identify this type of network
  name: "Base Mainnet"
  chain_id: 8453
  type: "l2_optimistic_rollup"
  l1_dependency_type: "ethereum_l1" # Indicates it needs a generic L1 Ethereum RPC
  explorer_url: "https://basescan.org"
```

### `docker-compose.yml` for an L2 Node Plugin: (Conceptual, as previously defined)
(The L1 RPC URL it receives, e.g., `APP_ETHEREUM_L1_RPC_URL`, is the *effective* L1 RPC from CypherpunkOS settings).

## Application Plugins with Multi-Network Support

Application plugins are designed to operate on Ethereum L1 and/or one or more L2 networks. They achieve this by consuming a comprehensive configuration bundle from CypherpunkOS at startup and managing network switching internally within their UI.

### Declaring Network Support in App `cypherpunk-app.yml`:

*   **`supported_networks`**: Array of network type IDs (e.g., `"ethereum_l1"`, `"base_mainnet"`) the app is built to work with.
*   **`network_configs`**: Array of objects. Each object contains app-specific default configurations for a `network_id` from `supported_networks`. This includes default public subgraph URLs, specific contract addresses for the app on that network, etc. This is fallback information if the user hasn't specified global external overrides for subgraphs for this network or for this app on this network.
*   **`dependencies`**: Generally empty or for non-RPC specific integrations. Apps rely on CypherpunkOS to provide the RPC and other network details for any of their `supported_networks` that are active in the system.

### CypherpunkOS Role in Multi-Network App Management:

1.  **Centralized Network Configuration:** Users define their preferred RPC sources (local L1/L2 plugin, or external URL) and preferred subgraph URLs (external, or fallback to app defaults) for each network type (e.g., `ethereum_l1`, `base_mainnet`) in the main CypherpunkOS settings.
2.  **Startup Configuration Bundle for Apps:** When an application plugin starts, CypherpunkOS:
    *   Identifies all networks listed in the app's `supported_networks`.
    *   For each of these networks, it assembles a complete configuration:
        *   **RPC URL:** Prioritizes User's global external RPC for that network > RPC from active local L1/L2 plugin for that network.
        *   **Chain ID:** From the definitive source (local plugin manifest or could be part of external RPC config).
        *   **Subgraph URL:** Prioritizes User's global external subgraph URL (if specified for this app/network or just for the network) > App's default public subgraph URL from its `network_configs` for that network > (Future) Local shared Graph Node service URL.
        *   **Contract Addresses & Other App-specifics:** From the app's `network_configs` for that network.
    *   This collection of configurations (one object per supported & available network) is passed to the app, typically via a single environment variable like `APP_NETWORK_CONFIGS_JSON`.
3.  **App Responsibility for In-App Switching:**
    *   The application plugin (its frontend) parses `APP_NETWORK_CONFIGS_JSON` at startup.
    *   It populates an in-app network switcher UI with the available configured networks.
    *   When the user selects a network within the app, the app internally switches its web3 provider to use the `rpcUrl` and `chainId` for that network from the bundle, and updates its data sources to use the corresponding `subgraphUrl` and contract addresses.
    *   **No re-configuration via CypherpunkOS dashboard is needed for the app to switch between these pre-configured networks.**

### L2 Data Sync & App Operation (e.g., Uniswap on Base):

*   **Transactions:** Always routed through the `rpcUrl` for the network currently selected *within the app*, which was originally sourced by CypherpunkOS (external user RPC or local L2 node plugin RPC).
*   **UI Data (Subgraphs):** Uses the `subgraphUrl` for the network currently selected *within the app*, sourced by CypherpunkOS (external user config or app manifest default).

## Workflow Example: User wants to use Uniswap, switching between L1 and Base

1.  In CypherpunkOS Settings:
    *   User has Nodeset (local L1) active.
    *   User has configured an external RPC URL for `base_mainnet` (e.g., `https://rpc.base.org`).
    *   User has *not* configured any custom subgraph URLs.
2.  User installs/starts the "Uniswap Interface" plugin.
3.  CypherpunkOS provides `APP_NETWORK_CONFIGS_JSON` to Uniswap containing (simplified):
    ```json
    [
      {
        "networkId": "ethereum_l1", "networkName": "Ethereum Mainnet (Local Nodeset)",
        "rpcUrl": "http://nodeset.internal:8545", "chainId": 1,
        "subgraphUrl": "https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v3", /* from Uniswap manifest */
        "contracts": { "v3_router_address": "0x...L1..." }
      },
      {
        "networkId": "base_mainnet", "networkName": "Base Mainnet (External RPC)",
        "rpcUrl": "https://rpc.base.org", "chainId": 8453,
        "subgraphUrl": "https://api.studio.thegraph.com/query/<id>/uniswap-v3-base/<v>", /* from Uniswap manifest */
        "contracts": { "v3_router_address": "0x...Base..." }
      }
    ]
    ```
4.  Uniswap app UI loads, potentially defaulting to L1. It shows a network switcher with "Ethereum Mainnet" and "Base Mainnet".
5.  User performs a swap on L1 (uses `http://nodeset.internal:8545`).
6.  User switches to "Base Mainnet" *within the Uniswap UI*.
7.  Uniswap app internally changes its provider to use `https://rpc.base.org` and Base contract/subgraph details from the startup JSON.
8.  User performs a swap on Base (uses `https://rpc.base.org`).

This allows dynamic in-app network changes without CypherpunkOS intervention after initial app startup, based on a centrally managed set of network endpoint preferences.

This model provides significant flexibility, allowing users to balance self-hosting rigor with the convenience of external services, per network, and per data type (RPC vs. subgraph). 