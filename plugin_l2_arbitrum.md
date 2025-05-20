# Plugin: Arbitrum L2 Node (Node Plugin)

## 1. Overview

- **Plugin Name:** Arbitrum One Node
- **Plugin Type:** Node
- **Category:** chain
- **Version:** 1.0.0 (align with a specific Arbitrum node version)
- **Author:** Offchain Labs / Cypherpunk Finance Team
- **License:** (Check Arbitrum Node Software License)
- **Description:** Installs and manages a local Arbitrum One Layer 2 node. This plugin functions as a **node** plugin of category `chain`, providing an L2 network endpoint for CypherpunkOS and other installed applications.

## 2. Purpose and Scope

This plugin enables users to run their own Arbitrum One node, enhancing decentralization and providing a private RPC endpoint for their L2 interactions. As a **node** plugin (category `chain`), it focuses on node operations and exposing necessary services (like RPC) to the CypherpunkOS ecosystem.

## 3. Key Features

## 4. Source Material & Node Software

*   **Arbitrum Node Software:** Running an Arbitrum One node typically involves using Nitro, the current Arbitrum technology stack. This requires a synced L1 Ethereum node and the Arbitrum node software itself.
    *   Official Arbitrum documentation: [https://docs.arbitrum.io/node-running/how-tos/running-a-full-node](https://docs.arbitrum.io/node-running/how-tos/running-a-full-node)
*   **Docker Images:** Official or community-vetted Docker images for Arbitrum Nitro nodes configured for Arbitrum One mainnet will be used (e.g., `offchainlabs/nitro-node`).

## 5. Core Components & Dependencies

1.  **Arbitrum L2 Node Service (`arbitrum-node` in `docker-compose.yml`):**
    *   Runs the Arbitrum Nitro node software.
    *   Requires persistent storage for Arbitrum blockchain data.
    *   Needs configuration for Arbitrum One network and connection to an L1 Ethereum RPC endpoint. CypherpunkOS will provide the *effective* L1 RPC URL (user-configured external L1 RPC, or RPC from active local L1 node provider like Nodeset) to this plugin via `APP_ETHEREUM_L1_RPC_URL`.
    *   Exposes the Arbitrum L2 RPC endpoint (e.g., port `8547`).
2.  **Status Service (Conceptual):**
    *   Implements the `/app-status` endpoint.
    *   Queries the local Arbitrum node for sync status and health.
    *   Reports logs.

## 6. `cypherpunk-plugin.yml` (Arbitrum L2 Node Plugin Manifest)

```yaml
id: arbitrum-one-node
name: Arbitrum One Node
version: "1.0.0" # Corresponds to a specific Arbitrum node software version bundle
plugin_type: node # This is a node plugin
category: chain # Specifies the type of node
description: Run a local Arbitrum One Layer 2 node.
developer: Offchain Labs / Cypherpunk Finance Team
# port: 8547 # Default Arbitrum RPC port, if exposed directly by this plugin for CypherpunkOS
# ... other common manifest fields ...

dependencies:
  # Requires an L1 RPC endpoint to be configured in CypherpunkOS
  - capability: ethereum_l1_rpc_available

# custom_fields:
#   node_software_version: "vX.Y.Z" # Specific version of Arbitrum node software
#   data_dir: "/data/arbitrum-one"
```

## 7. `docker-compose.yml` (Conceptual for Arbitrum L2 Plugin)

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

## 8. Key Implementation Details
*   The Arbitrum node service will use the L1 RPC URL (`APP_ETHEREUM_L1_RPC_URL`) provided by CypherpunkOS.
*   This plugin exports its local RPC details for CypherpunkOS to use if this local Arbitrum node is chosen as the active RPC provider for the `arbitrum_one` network type.

## 9. User Interaction
*   User installs the "Arbitrum One Node (Local)" plugin if they want to run a local Arbitrum node.
*   If no external Arbitrum RPC is set by the user, and this plugin is running and synced, apps selecting "Arbitrum One" will be configured to use this local node's RPC.

## 10. Considerations
*   **L1 RPC Dependency:** Stability is tied to the effective L1 RPC source.
*   **Resource Usage:** Significant.
*   **Sync Time:** Can be lengthy.
*   **Nitro Configuration:** Requires correct L1 endpoint and Arbitrum One network parameters.
*   **Updates:** Plugin needs to manage updates to the `offchainlabs/nitro-node` image. 