# Plugin: Aave DeFi Application

## 1. Overview

- **Plugin Name:** Aave Client
- **Plugin Type:** App
- **Version:** 1.0.0

## 2. Purpose and Scope

This plugin allows users to connect to the Aave V3 money markets on various supported networks (initially Ethereum L1) directly from their CypherpunkOS instance. It provides a graphical interface for supplying, borrowing, and managing positions. As an **app**, it operates with its own UI, launched from CypherpunkOS.

## 3. Key Features

## 4. Source Material

*   **Primary Frontend Source:** The official Aave Interface repository: `aave/interface`.
*   **Subgraph Information:** Aave's official subgraph deployment details for L1 and various L2s will be used for default `network_configs`.
*   **Contract Addresses:** Aave documentation and `aave/aave-address-book` or similar repositories for correct contract addresses per network.

## 5. Core Components & Dependencies (per running instance)

1.  **Frontend Service (`frontend` in `docker-compose.yml`):**
    *   **Image:** A Docker image built from a fork of the `aave/interface` web app.
    *   **Functionality:** Serves the Aave web UI. This UI will be modified/configured to:
        *   Read environment variables provided by CypherpunkOS to determine the selected network (L1 or a specific L2), its RPC URL (`APP_SELECTED_NETWORK_RPC_URL`), Chain ID (`APP_SELECTED_NETWORK_CHAIN_ID`), relevant contract addresses (e.g., `REACT_APP_POOL_ADDRESSES_PROVIDER_ADDRESS`), and the Subgraph URL to use (`REACT_APP_SUBGRAPH_URL`).
        *   Use the `APP_SELECTED_NETWORK_RPC_URL` for all on-chain read/write operations.
        *   **Oracle Price Feeds:** The Aave protocol inherently relies on Chainlink oracles on-chain. The frontend will display these prices (sourced via Aave SDKs or subgraphs), but the core logic uses on-chain oracles.
    *   **Data Source Strategy (for UI, transactions are via selected RPC):**
        *   **RPC:** Always uses `APP_SELECTED_NETWORK_RPC_URL` (User External > Active Local L1/L2 Node App).
        *   **Subgraph URL:** Uses `APP_SELECTED_SUBGRAPH_URL` (User External App-Specific Subgraph > User External Network-Wide Subgraph > App Manifest Default Public Subgraph for Network > Optional Local Shared Graph Node Service).
2.  **Status Service (Conceptual):**
    *   Implements `/app-status`.
    *   Reports basic status, app version, and currently configured network/RPC source.

## 6. `cypherpunk-plugin.yml` (Aave Plugin Manifest)

```yaml
id: aave-app
name: Aave Client
version: 1.0.0
plugin_type: app
description: Interact with the Aave V3 protocol.
developer: Aave Community / Cypherpunk Finance Team
website: "https://app.aave.com"
repo: "<link_to_cypherpunk_aave_plugin_repo>"
support: "<link_to_support_channel>"

supported_networks: # Network IDs CypherpunkOS recognizes
  - "ethereum_l1"
  - "base_mainnet"
  - "arbitrum_one"
  - "optimism_mainnet"

network_configs:
  - network_id: "ethereum_l1"
    config:
      chain_id_ref: 1
      # Aave V3 Mainnet
      pool_addresses_provider_address: "0xa97684ead0e402dC232d5A977953DF7ECBaB3CDb"
      ui_pool_data_provider_address: "0x91c0eA31b49B69Ea18607702c5d58D2A9420Cb02"
      default_subgraph_url: "https://api.thegraph.com/subgraphs/name/aave/protocol-v3"

  - network_id: "base_mainnet"
    config:
      chain_id_ref: 8453
      # Aave V3 Base (Illustrative, check official Aave docs)
      pool_addresses_provider_address: "0x...BaseAaveV3PoolAddressesProvider..."
      ui_pool_data_provider_address: "0x...BaseAaveV3UiPoolDataProvider..."
      default_subgraph_url: "https://api.studio.thegraph.com/query/<id>/aave-v3-base/<version>" # Placeholder

  - network_id: "arbitrum_one"
    config:
      chain_id_ref: 42161
      pool_addresses_provider_address: "0xa97684ead0e402dC232d5A977953DF7ECBaB3CDb" # V3 on Arbitrum
      ui_pool_data_provider_address: "0x0657ac840ba305973f736857249F0A0917532291"
      default_subgraph_url: "https://api.thegraph.com/subgraphs/name/aave/protocol-v3-arbitrum"

  - network_id: "optimism_mainnet"
    config:
      chain_id_ref: 10
      pool_addresses_provider_address: "0xa97684ead0e402dC232d5A977953DF7ECBaB3CDb" # V3 on Optimism
      ui_pool_data_provider_address: "0x69FA688f1Dc47d4B5d8029D5a35FB7a548310654"
      default_subgraph_url: "https://api.thegraph.com/subgraphs/name/aave/protocol-v3-optimism"

dependencies: [] # Relies on CypherpunkOS to provide a compatible RPC for a supported network

port: 3001 
path: "/aave"
status_endpoint: "/app-status"
```

## 7. `docker-compose.yml` (Conceptual for Aave Plugin)

```yaml
version: "3.7"
services:
  frontend:
    image: your_repo/cypherpunk-aave-interface:latest
    restart: unless-stopped
    environment:
      # --- Injected by CypherpunkOS --- 
      - "APP_SELECTED_NETWORK_ID=ethereum_l1"
      - "APP_SELECTED_NETWORK_NAME=Ethereum Mainnet"
      - "APP_SELECTED_NETWORK_RPC_URL=${APP_ETHEREUM_L1_RPC_URL}"
      - "APP_SELECTED_NETWORK_CHAIN_ID=1"
      # From app's network_configs for selected network, potentially overridden by user's external subgraph config
      - "REACT_APP_POOL_ADDRESSES_PROVIDER_ADDRESS=0xa97684ead0e402dC232d5A977953DF7ECBaB3CDb"
      - "REACT_APP_UI_POOL_DATA_PROVIDER_ADDRESS=0x91c0eA31b49B69Ea18607702c5d58D2A9420Cb02"
      # Effective subgraph URL (User External > Manifest Default Public > Future Local Shared)
      - "REACT_APP_SUBGRAPH_URL=https://api.thegraph.com/subgraphs/name/aave/protocol-v3"
      # --- End of CypherpunkOS injected variables ---

      # Other Aave UI specific env vars (e.g., market specific, feature flags)
    ports:
      - "3001"

  # status_service: ... (as before, reports configured network/RPC)
```

**Environment Variable Handling by the Forked Aave UI:**
*   The Aave interface code must be adapted to use the injected `APP_SELECTED_...` environment variables for network parameters, RPC, and subgraph URLs, and `REACT_APP_...` variables for contract addresses corresponding to the selected network.
*   The UI must clearly display the active network.

## 8. Build & Deployment Process for the Plugin
(Similar to Uniswap: Fork, adapt for ENV var based config, Dockerize, define manifest with comprehensive `network_configs`, test across L1/L2s with various RPC/subgraph source combinations.)

## 9. User Interaction Flow for Network Selection
(Identical to Uniswap: User selects network in CypherpunkOS dashboard for Aave; CypherpunkOS injects correct effective RPC/subgraph URLs and configs.)

## 10. Considerations & Challenges
*   **Clarity on RPC/Subgraph Source:** UI must be very clear about whether it's using local or external RPCs, and default public vs. user-provided subgraphs.
*   **Frontend Fork Maintenance.**
*   **Comprehensive and Accurate `network_configs`:** Crucial for Aave's multi-network support.
*   **UI/UX for Multi-Network Aave.** 