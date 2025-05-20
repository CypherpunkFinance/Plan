# Plugin: Uniswap DeFi Application

## 1. Overview

- **Plugin Name:** Uniswap Client
- **Plugin Type:** App
- **Version:** 1.0.0

Provide a self-hosted interface for swapping tokens and managing liquidity on Uniswap (v2 and v3 primarily). The plugin will allow users to interact with Uniswap on Ethereum Mainnet (L1) or any compatible L2 network. CypherpunkOS will provide the necessary RPC endpoint (local via an active L1/L2 Node App or user-configured external) and subgraph URL (user-configured external or default public from this plugin's manifest) for the selected network.

## 2. Source Material

*   **Primary Frontend Source:** The official Uniswap Interface repository: `Uniswap/interface`.
*   **Subgraph Information:** Official Uniswap subgraph deployment details for L1 and various L2s will be used for default `network_configs`.

## 3. Core Components & Dependencies (per running instance)

1.  **Frontend Service (`frontend` in `docker-compose.yml`):**
    *   **Image:** A Docker image built from a fork of the `Uniswap/interface` web app.
    *   **Functionality:** Serves the Uniswap web UI. This UI will be modified/configured to:
        *   Read environment variables provided by CypherpunkOS to determine the selected network (L1 or a specific L2), its RPC URL (`APP_SELECTED_NETWORK_RPC_URL`), Chain ID (`APP_SELECTED_NETWORK_CHAIN_ID`), relevant contract addresses (e.g., `REACT_APP_V3_ROUTER_ADDRESS`), and the Subgraph URL to use (`REACT_APP_SUBGRAPH_URL`).
        *   Use the `APP_SELECTED_NETWORK_RPC_URL` for all on-chain read/write operations.
    *   **Data Source Strategy (for UI, transactions are via selected RPC):**
        *   **RPC:** Always uses `APP_SELECTED_NETWORK_RPC_URL` (User External > Active Local L1/L2 Node App).
        *   **Subgraph URL:** Uses `APP_SELECTED_SUBGRAPH_URL` (User External App-Specific Subgraph > User External Network-Wide Subgraph > App Manifest Default Public Subgraph for Network > Optional Local Shared Graph Node Service).
2.  **Status Service (Conceptual):**
    *   Implements `/app-status`.
    *   Reports basic status, app version, and currently configured network/RPC source.

## 4. `cypherpunk-plugin.yml` (Uniswap Plugin Manifest)

```yaml
id: uniswap-app
name: Uniswap Client
version: 1.0.0
plugin_type: app
description: Swap tokens and manage liquidity on Uniswap.
developer: Uniswap Labs / Cypherpunk Finance Team
website: "https://app.uniswap.org"
repo: "<link_to_cypherpunk_uniswap_plugin_repo>"
support: "<link_to_support_channel>"

supported_networks: # Network IDs CypherpunkOS recognizes and can provide RPC for
  - "ethereum_l1"
  - "base_mainnet"
  - "arbitrum_one"

network_configs:
  - network_id: "ethereum_l1"
    config:
      chain_id_ref: 1 # For CypherpunkOS to verify against the RPC source
      v2_router_address: "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D"
      v3_router_address: "0xE592427A0AEce92De3Edee1F18E0157C05861564"
      v3_factory_address: "0x1F98431c8aD98523631AE4a59f267346ea31F984"
      default_subgraph_v2_url: "https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v2"
      default_subgraph_v3_url: "https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v3"

  - network_id: "base_mainnet"
    config:
      chain_id_ref: 8453
      v3_router_address: "0x2626664c2603336E57B271c5C0b26F421741e481"
      v3_factory_address: "0x33128a8fC17869897dcE68Ed026d694621f6FDfD"
      default_subgraph_v3_url: "https://api.studio.thegraph.com/query/<id>/uniswap-v3-base/<version>" # Actual URL TBD

  - network_id: "arbitrum_one"
    config:
      chain_id_ref: 42161
      v3_router_address: "0xE592427A0AEce92De3Edee1F18E0157C05861564"
      v3_factory_address: "0x1F98431c8aD98523631AE4a59f267346ea31F984"
      default_subgraph_v3_url: "https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v3-arbitrum"

dependencies: [] # Relies on CypherpunkOS to provide a compatible RPC for one of the supported_networks

port: 3000
path: "/uniswap"
status_endpoint: "/app-status"
```

## 5. `docker-compose.yml` (Conceptual for Uniswap Plugin)

```yaml
version: "3.7"
services:
  frontend:
    image: your_repo/cypherpunk-uniswap-interface:latest
    restart: unless-stopped
    environment:
      # --- Injected by CypherpunkOS based on user's network selection & external/local RPC/subgraph configs ---
      - "APP_SELECTED_NETWORK_ID=ethereum_l1"               # e.g., ethereum_l1, base_mainnet
      - "APP_SELECTED_NETWORK_NAME=Ethereum Mainnet (External RPC)" # Human-readable, indicates source if external
      - "APP_SELECTED_NETWORK_RPC_URL=https://user.external.rpc.com" # The actual RPC URL to use
      - "APP_SELECTED_NETWORK_CHAIN_ID=1"                   # Actual chain ID of the selected network
      
      # These are from this app's network_configs for the APP_SELECTED_NETWORK_ID, injected by CypherpunkOS
      - "REACT_APP_V2_ROUTER_ADDRESS=0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D"
      - "REACT_APP_V3_ROUTER_ADDRESS=0xE592427A0AEce92De3Edee1F18E0157C05861564"
      - "REACT_APP_V3_FACTORY_ADDRESS=0x1F98431c8aD98523631AE4a59f267346ea31F984"
      
      # This is the effective subgraph URL (User External > Manifest Default Public > Future Local Shared)
      - "REACT_APP_SUBGRAPH_V2_URL=https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v2"
      - "REACT_APP_SUBGRAPH_V3_URL=https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v3"
      # --- End of CypherpunkOS injected variables ---
    ports:
      - "3000"

  # status_service: ... (as before)
```

**Environment Variable Handling by the Forked Uniswap UI:**
*   The frontend must robustly use the `APP_SELECTED_NETWORK_RPC_URL` and `APP_SELECTED_NETWORK_CHAIN_ID` to instantiate its web3 provider.
*   It must use the `REACT_APP_` prefixed variables for contract addresses and subgraph URLs, which are dynamically set by CypherpunkOS based on the selected network and user overrides.

## 6. Build & Deployment Process for the Plugin
(Largely as before, with emphasis on testing against various combinations of local/external RPCs and subgraphs for L1 and L2s.)

## 7. User Interaction Flow for Network Selection
(Largely as before, CypherpunkOS dashboard allows selection of a supported network, and CypherpunkOS handles providing the correct (potentially external) RPC and subgraph URLs to the plugin.)

## 8. Considerations & Challenges
*   **Clear Indication of RPC/Subgraph Source:** The UI must clearly show if it's using a local node RPC vs. an external one, and a default public subgraph vs. a user-provided external one, so the user understands the trust/privacy implications for each connection.
*   **Frontend Fork Maintenance & Dynamic Configuration:** This remains a key challenge. 