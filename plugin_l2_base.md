# Plugin: Base L2 Node (Node Plugin)

## 1. Overview

- **Plugin Name:** Base Mainnet Node
- **Plugin Type:** Node
- **Category:** chain
- **Version:** 1.0.0 (align with a specific Base node version, e.g., OP Stack version)
- **Author:** Coinbase / Optimism / Cypherpunk Finance Team
- **License:** (Check OP Stack / Base Node Software License)
- **Description:** Installs and manages a local Base Mainnet Layer 2 node. This plugin functions as a **node** plugin of category `chain`, providing an L2 network endpoint for CypherpunkOS and other installed applications.

## 2. Purpose and Scope

This plugin enables users to run their own Base Mainnet node, enhancing decentralization and providing a private RPC endpoint for their L2 interactions on the Base network. As a **node** plugin (category `chain`), it focuses on node operations and exposing necessary services (like RPC) to the CypherpunkOS ecosystem.

## 3. Key Features

## 4. Source Material & Node Software

*   **Base Node Software:** Base is built on the OP Stack (Optimism). Running a Base node typically involves running an OP Stack execution client (like `op-geth`) and an OP Stack rollup client (`op-node`).
    *   Official Base documentation: [https://docs.base.org/building-with-base/nodes/run-a-base-node](https://docs.base.org/building-with-base/nodes/run-a-base-node)
    *   Optimism Monorepo: [https://github.com/ethereum-optimism/optimism](https://github.com/ethereum-optimism/optimism)
*   **Docker Images:** Official or community-vetted Docker images for `op-geth` and `op-node` configured for Base mainnet.

## 5. Core Components & Dependencies

1.  **Base L2 Node Service (e.g., `op_geth_base`, `op_node_base` in `docker-compose.yml`):**
    *   Runs `op-geth` (Execution Client for Base) and `op-node` (Rollup Client for Base).
    *   Requires persistent storage for its blockchain data.
    *   Requires an L1 Ethereum RPC endpoint. CypherpunkOS will provide the *effective* L1 RPC URL (user-configured external L1 RPC, or the RPC from the active local L1 node provider like the default Nodeset app) to this plugin via the `APP_ETHEREUM_L1_RPC_URL` environment variable.
    *   Exposes the Base L2 RPC endpoint on a defined internal port (e.g., `8545`).
2.  **Status Service (Conceptual):**
    *   Implements the `/app-status` endpoint.
    *   Queries the local `op-node`/`op-geth` for sync status and health.
    *   Reports logs.

## 6. `cypherpunk-plugin.yml` (Base L2 Node Plugin Manifest)

```yaml
id: base-mainnet-node
name: Base Mainnet Node
version: "1.0.0" # Corresponds to a specific Base node software version bundle
plugin_type: node # This is a node plugin
category: chain # Specifies the type of node
description: Run a local Base Mainnet Layer 2 node.
developer: Coinbase / Cypherpunk Finance Team
# port: 8551 # Default Base RPC port (often same as OP Stack), if exposed by this plugin
# ... other common manifest fields ...

dependencies:
  # Requires an L1 RPC endpoint to be configured in CypherpunkOS
  - capability: ethereum_l1_rpc_available

# custom_fields:
#   node_software_version: "vX.Y.Z" # Specific version of OP Stack / Base node software
#   data_dir: "/data/base-mainnet"
```

## 7. `docker-compose.yml` (Conceptual for Base L2 Plugin)

```yaml
version: "3.7"
services:
  op_geth_base:
    image: ethereumoptimism/op-geth:<latest_tag_for_base>
    restart: unless-stopped
    command:
      - "--datadir=/data/op-geth"
      - "--http"
      - "--http.addr=0.0.0.0"
      - "--http.port=8547" # Internal EL RPC for this L2's op-node
      - "--http.api=eth,net,web3,debug"
      - "--ws"
      - "--ws.addr=0.0.0.0"
      - "--ws.port=8548"
      - "--ws.api=eth,net,web3,debug"
      - "--authrpc.jwtsecret=/data/op-geth/jwt.secret"
      # Base specific network params (e.g., --rollup.config=/config/rollup.json)
      # /config/rollup.json would be part of the Docker image or a configmap
    volumes:
      - "base_el_data:/data/op-geth"
      # - ./rollup_base.json:/config/rollup.json:ro # Example if providing custom rollup config
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"

  op_node_base:
    image: ethereumoptimism/op-node:<latest_tag_for_base>
    restart: unless-stopped
    depends_on:
      - op_geth_base
    command:
      # APP_ETHEREUM_L1_RPC_URL is the EFFECTIVE L1 RPC (local L1 Node App or user's external L1)
      # This ENV var name should be standardized by CypherpunkOS for L2 plugins needing L1 RPC.
      - "--l1=${APP_ETHEREUM_L1_RPC_URL}"
      - "--l2=http://op_geth_base:8547" # Internal RPC to this L2's op-geth
      - "--l2.jwt-secret=/data/jwt/jwt.secret"
      - "--rpc.addr=0.0.0.0"
      - "--rpc.port=8545" # L2 RPC exposed by this plugin
      - "--network=base-mainnet" # Or --rollup.config pointing to a Base config file
    volumes:
      - "base_el_data/jwt.secret:/data/jwt/jwt.secret:ro"
      - "base_rollup_data:/data/op-node"
      # - ./rollup_base.json:/config/rollup.json:ro
    ports:
      - "8545" # Docker will assign a host port, CypherpunkOS manages internal access via service name
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"

  status_service_base:
    image: <your_repo>/cypherpunk-l2-status-service:latest
    restart: unless-stopped
    environment:
      - "NODE_TYPE=op_stack"
      - "L2_RPC_URL=http://op_node_base:8545"
      - "L1_RPC_URL=${APP_ETHEREUM_L1_RPC_URL}" # Effective L1 RPC for status checks
      - "LOG_SOURCE_CONTAINER_NAMES=op_geth_base,op_node_base"
      - "APP_VERSION=1.0.0"
      - "NETWORK_NAME=Base Mainnet (Local Provider)"
      - "CHAIN_ID=8453"
    ports:
      - "<status_api_internal_port>:80"

volumes:
  base_el_data:
  base_rollup_data:
```

## 8. Key Implementation Details
*   The L1 RPC URL (e.g., `APP_ETHEREUM_L1_RPC_URL`) provided to this plugin's `op-node` service will be the one CypherpunkOS has determined based on user preference (external L1 RPC or active local L1 node provider app like Nodeset).
*   The `exports.sh` for this plugin defines the RPC URL and Chain ID that CypherpunkOS *can* use if this local Base node is the chosen RPC provider for the `base_mainnet` network type.

## 9. User Interaction
*   User installs "Base Node (Local)" plugin if they want to run a local Base node.
*   If not installed, or if an external Base RPC is configured and prioritized by the user in CypherpunkOS settings, apps needing Base access will use the external RPC.
*   If this local plugin *is* the active RPC provider for Base (i.e., no overriding external Base RPC is set by user), other apps selecting "Base Mainnet" will be configured by CypherpunkOS to use its exported `APP_L2_BASE_RPC_URL`.

## 10. Considerations
*   **Resource Intensive:** Remains a consideration for users choosing to run this locally.
*   **L1 RPC Dependency:** The L2 node plugin's stability is tied to the stability of the L1 RPC it's configured to use (which could be from a local L1 node app like Nodeset or an external service configured by the user). 