# Arbitrum One L2 Node Plugin for Cypherpunk Finance

## 1. Objective

To provide an installable Cypherpunk App that runs a full Arbitrum One L2 node *for users who choose to self-host this L2*. This plugin will enable users to have local RPC access to the Arbitrum One network, which CypherpunkOS can then provide to other Cypherpunk Apps if no external Arbitrum RPC is configured by the user for the `arbitrum_one` network type. It will also report its sync status and logs to the CypherpunkOS dashboard.

## 2. Source Material & Node Software

*   **Arbitrum Node Software:** Running an Arbitrum One node typically involves using Nitro, the current Arbitrum technology stack. This requires a synced L1 Ethereum node and the Arbitrum node software itself.
    *   Official Arbitrum documentation: [https://docs.arbitrum.io/node-running/how-tos/running-a-full-node](https://docs.arbitrum.io/node-running/how-tos/running-a-full-node)
*   **Docker Images:** Official or community-vetted Docker images for Arbitrum Nitro nodes configured for Arbitrum One mainnet will be used (e.g., `offchainlabs/nitro-node`).

## 3. Core Components & Dependencies

1.  **Arbitrum L2 Node Service (`arbitrum-node` in `docker-compose.yml`):**
    *   Runs the Arbitrum Nitro node software.
    *   Requires persistent storage for Arbitrum blockchain data.
    *   Needs configuration for Arbitrum One network and connection to an L1 Ethereum RPC endpoint. CypherpunkOS will provide the *effective* L1 RPC URL (user-configured external L1 RPC, or RPC from active local L1 node provider like Nodeset) to this plugin via `APP_ETHEREUM_L1_RPC_URL`.
    *   Exposes the Arbitrum L2 RPC endpoint (e.g., port `8547`).
2.  **Status Service (Conceptual):**
    *   Implements the `/app-status` endpoint.
    *   Queries the local Arbitrum node for sync status and health.
    *   Reports logs.

## 4. `cypherpunk-app.yml` (Arbitrum L2 Node Plugin Manifest)

```yaml
manifestVersion: 1
id: "l2-arbitrum"
name: "Arbitrum One Node (Local)"
version: "1.0.0"
app_type: "l2_node"
description: "Runs a local Arbitrum One L2 node, providing a local RPC endpoint if selected by the user as the source for Arbitrum One RPC."
developer: "Cypherpunk Finance Community / Offchain Labs"
website: "https://arbitrum.io"
repo: "<link_to_cypherpunk_l2_arbitrum_plugin_repo>"
support: "<link_to_support_channel>"

dependencies:
  - "cap:ethereum_l1_rpc_available" # Generic capability for L1 RPC

port: 8547 # Default internal Arbitrum L2 RPC port
path: "/l2-nodes/arbitrum"
status_endpoint: "/app-status"

exports:
  - "APP_L2_ARBITRUM_RPC_URL=http://arbitrum_node:8547"
  - "APP_L2_ARBITRUM_CHAIN_ID=42161"
  - "APP_L2_ARBITRUM_NETWORK_NAME=Arbitrum One (Local Provider)"

network_provided:
  id: "arbitrum_one"
  name: "Arbitrum One"
  chain_id: 42161
  type: "l2_optimistic_rollup"
  l1_dependency_type: "ethereum_l1"
  explorer_url: "https://arbiscan.io"
```

## 5. `docker-compose.yml` (Conceptual for Arbitrum L2 Plugin)

```yaml
version: "3.7"
services:
  arbitrum_node:
    image: offchainlabs/nitro-node:latest # Or a specific tag for Arbitrum One
    restart: unless-stopped
    environment:
      # APP_ETHEREUM_L1_RPC_URL is the EFFECTIVE L1 RPC (local L1 Node App or user's external L1)
      # The actual ENV var name Arbitrum Nitro node expects for L1 might differ, adjust as needed.
      - "ARB_NITRO_NODE__PARENT__CHAIN__RPC__URL=${APP_ETHEREUM_L1_RPC_URL}" 
      - "ARB_NITRO_NODE__HTTP__ADDR=0.0.0.0"
      - "ARB_NITRO_NODE__HTTP__PORT=8547"
      - "ARB_NITRO_NODE__HTTP__API=eth,net,web3,arb,debug"
      - "ARB_NITRO_NODE__HTTP__VHOSTS=*"
      - "ARB_NITRO_NODE__HTTP__CORS_DOMAIN=*"
      # Ensure correct Arbitrum One chain ID and other configs are implicitly set by image or specific flags
    volumes:
      - "arbitrum_data:/home/user/.arbitrum/mainnet" # Path might vary based on image specifics
    ports:
      - "8547" # Docker assigns host port; CypherpunkOS uses service name for internal access
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"

  status_service_arbitrum:
    image: <your_repo>/cypherpunk-l2-status-service:latest
    restart: unless-stopped
    environment:
      - "NODE_TYPE=arbitrum_nitro"
      - "L2_RPC_URL=http://arbitrum_node:8547"
      - "L1_RPC_URL=${APP_ETHEREUM_L1_RPC_URL}" # Effective L1 RPC
      - "LOG_SOURCE_CONTAINER_NAMES=arbitrum_node"
      - "APP_VERSION=1.0.0"
      - "NETWORK_NAME=Arbitrum One (Local Provider)"
      - "CHAIN_ID=42161"
    ports:
      - "<status_api_internal_port>:80"

volumes:
  arbitrum_data:
```

## 6. Key Implementation Details
*   The Arbitrum node service will use the L1 RPC URL (`APP_ETHEREUM_L1_RPC_URL`) provided by CypherpunkOS.
*   This plugin exports its local RPC details for CypherpunkOS to use if this local Arbitrum node is chosen as the active RPC provider for the `arbitrum_one` network type.

## 7. User Interaction
*   User installs the "Arbitrum One Node (Local)" plugin if they want to run a local Arbitrum node.
*   If no external Arbitrum RPC is set by the user, and this plugin is running and synced, apps selecting "Arbitrum One" will be configured to use this local node's RPC.

## 8. Considerations
*   **L1 RPC Dependency:** Stability is tied to the effective L1 RPC source.
*   **Resource Usage:** Significant.
*   **Sync Time:** Can be lengthy.
*   **Nitro Configuration:** Requires correct L1 endpoint and Arbitrum One network parameters.
*   **Updates:** Plugin needs to manage updates to the `offchainlabs/nitro-node` image. 