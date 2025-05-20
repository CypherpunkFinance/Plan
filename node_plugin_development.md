# CypherpunkOS Node Plugin Development Guide

## 1. Introduction

This guide details the development of **Node Plugins** (`plugin_type: node`) for CypherpunkOS. Node plugins are backend services that typically provide data, network access, or indexing capabilities to CypherpunkOS and other installed App Plugins.

Unlike App Plugins, Node Plugins usually do not have their own distinct user interface accessible to the end-user directly. Instead, they expose service endpoints (e.g., JSON-RPC, GraphQL) that CypherpunkOS or other plugins consume.

Examples of Node Plugins include:
-   **Blockchain Nodes:** L1 clients (e.g., Ethereum Execution + Consensus), L2 clients (e.g., Base, Arbitrum).
    -   `category: chain`
-   **Subgraph Nodes:** Graph Protocol nodes for indexing specific subgraphs.
    -   `category: subgraph_node`
-   **Other Indexers/Backend Services:** Custom data indexers or other backend-only services.
    -   `category: indexer` (or other descriptive category)

Node Plugins run as Dockerized services managed by CypherpunkOS.

## 2. Core Concepts of a Node Plugin

-   **Backend Service:** Primarily provides backend functionality and exposes service endpoints.
-   **No Direct User UI:** Users interact with these nodes indirectly, e.g., by CypherpunkOS using their RPC for network access, or an App Plugin querying a subgraph node.
-   **Dockerized:** Runs as one or more Docker containers defined in a `docker-compose.yml` file.
-   **Configurable via Environment Variables:** CypherpunkOS injects necessary configurations (e.g., L1 RPC endpoint for an L2 node, data directory paths).
-   **Exports Endpoints:** May export information about its service endpoints (e.g., RPC URL, chain ID) for CypherpunkOS to use or provide to other plugins.

## 3. Plugin Structure

```
my-custom-node-plugin/
├── cypherpunk-plugin.yml # Manifest file
├── docker-compose.yml    # Docker Compose service definitions
├── Dockerfile            # Dockerfile for the main node service (if not using pre-built images)
├── (optional) scripts/   # Helper scripts (e.g., for entrypoint, health checks)
├── (optional) config/    # Default configuration files for the node software
├── README.md             # Plugin-specific README
└── LICENSE               # License for your plugin code
```

## 4. Key Files

### 4.1. `cypherpunk-plugin.yml` (Manifest File)

**Key Fields for Node Plugins:**

```yaml
id: "my-base-l2-node"
name: "My Base L2 Node"
version: "1.0.0"
plugin_type: node
category: chain # e.g., chain, subgraph_node, indexer

description: "Runs a local Base Mainnet Layer 2 node."
developer: "Your Name/Org"
# ... other common fields: repo, support, icon ...

# Dependencies (Example for an L2 Node)
# dependencies:
#   - capability: "ethereum_l1_rpc_available" # Signals need for an L1 RPC

# Service Port (Internal Docker port, not exposed to host directly by plugin)
# port: 8545 # The primary RPC/service port of the node inside the container

# Custom fields specific to this node type, used by CypherpunkOS or the plugin itself
custom_fields:
  data_dir_container: "/data/base-mainnet" # Where the node stores its data inside the container
  # For blockchain nodes, CypherpunkOS might manage data persistence on the host
  # and map it to this container path.

# Information this node plugin exports for CypherpunkOS and other plugins
exports:
  # Example for an L2 node
  - "APP_L2_BASE_RPC_URL=http://my-base-l2-node_service_name:8545" # Internal service name from docker-compose
  - "APP_L2_BASE_CHAIN_ID=8453"
  - "APP_L2_BASE_NETWORK_NAME=Base Mainnet (Local Node)"
  # Example for a Subgraph Node
  # - "APP_SUBGRAPH_UNISWAP_MAINNET_HTTP_URL=http://my-subgraph-node_service_name:8000/subgraphs/name/uniswap/uniswap-v3"
  # - "APP_SUBGRAPH_UNISWAP_MAINNET_WS_URL=ws://my-subgraph-node_service_name:8001/subgraphs/name/uniswap/uniswap-v3"

status_endpoint: "/status" # Optional: Path for a simple status check service within the plugin
```

### 4.2. `docker-compose.yml`

Defines the Docker service(s) for the node.

**Example: L2 Node (Base - simplified from `plugin_l2_base.md`)**
```yaml
version: "3.7"
services:
  op_node_base: # Service name used in exports (e.g., http://op_node_base:8545)
    image: ethereumoptimism/op-node:<latest_tag_for_base>
    restart: unless-stopped
    environment:
      # APP_ETHEREUM_L1_RPC_URL is injected by CypherpunkOS based on user's L1 config
      - "OP_NODE_L1_ETH_RPC=${APP_ETHEREUM_L1_RPC_URL}"
      - "OP_NODE_L2_ENGINE_AUTH_RPC=http://op_geth_base:8551" # Example internal connection
      - "OP_NODE_RPC_ENABLE_ADMIN=true"
      - "OP_NODE_RPC_ADDR=0.0.0.0"
      - "OP_NODE_RPC_PORT=8545"
      # ... other op-node configurations for Base ...
    volumes:
      - "base_op_node_data:/app/data" # Persistent data for op-node
    # No ports exposed to host here; CypherpunkOS manages access if needed

  op_geth_base:
    image: ethereumoptimism/op-geth:<latest_tag_for_base>
    restart: unless-stopped
    environment:
      # ... op-geth configurations for Base ...
    volumes:
      - "base_op_geth_data:/app/data" # Persistent data for op-geth

volumes:
  base_op_node_data:
  base_op_geth_data:
```

**Example: Subgraph Node (Conceptual)**
```yaml
version: "3.7"
services:
  graph_node:
    image: graphprotocol/graph-node:latest # Use a specific stable version
    restart: unless-stopped
    environment:
      # POSTGRES_DB_URL: Injected by CypherpunkOS if using a shared DB service, or use a linked service
      - "POSTGRES_DB_URL=postgresql://user:pass@postgres_service:5432/graph-node"
      # ETHEREUM_RPC_URL: For the specific network this subgraph indexes (e.g., mainnet archive)
      # This would be injected by CypherpunkOS (e.g., APP_ETHEREUM_L1_ARCHIVE_RPC_URL)
      - "ETHEREUM_RPC_URL=${APP_SELECTED_NETWORK_ARCHIVE_RPC_URL}"
      - "GRAPH_NODE_ADMIN_PORT=8020"
      - "GRAPH_NODE_INDEX_PORT=8030"
      - "GRAPH_NODE_QUERY_PORT=8000"
      - "GRAPH_NODE_WS_PORT=8001"
      - "IPFS_ENDPOINT=https://api.thegraph.com/ipfs/" # Or a local IPFS node
      # SUBGRAPH_IPFS_HASH: This could be a custom field in manifest or passed at deployment time
      # - "SUBGRAPH_TO_SYNC_IPFS_HASH=Qm..."
    volumes:
      - graph_node_data:/var/lib/graphprotocol/graph-node/
    # Ports 8000 (query), 8001 (ws), 8020 (admin), 8030 (index) are internal.
    # CypherpunkOS would use these to construct exported URLs if needed.

  postgres_service: # Example of a bundled PostgreSQL
    image: postgres:14
    restart: unless-stopped
    environment:
      - "POSTGRES_USER=user"
      - "POSTGRES_PASSWORD=pass"
      - "POSTGRES_DB=graph-node"
    volumes:
      - graph_db_data:/var/lib/postgresql/data/

volumes:
  graph_node_data:
  graph_db_data:
```

### 4.3. `Dockerfile` (if applicable)

If you are not using pre-built Docker images for the node software (e.g., `ethereumoptimism/op-node`), you will need a Dockerfile to build your node service.
-   Follow best practices for the specific node software you are packaging.
-   Ensure it respects data directories and can be configured via environment variables passed by `docker-compose.yml`.

## 5. Key Development Considerations

-   **Configuration:** Design your node plugin to be configurable via environment variables. CypherpunkOS will pass necessary information (like L1 RPC endpoints for L2 nodes, database connection strings, data paths).
-   **Data Persistence:** Node plugins (especially blockchain and subgraph nodes) require significant persistent storage. Define volumes in your `docker-compose.yml`. CypherpunkOS will manage the host paths for these volumes.
-   **Resource Management:** Be mindful of CPU, memory, and disk space requirements. Provide sensible defaults and document resource needs.
-   **Endpoint Exposure (`exports`):** If your node provides a service endpoint (RPC, GraphQL), declare it in the `exports` section of your `cypherpunk-plugin.yml`. CypherpunkOS uses this to:
    *   Provide the endpoint to other App Plugins.
    *   Potentially use the endpoint itself (e.g., for dashboard status).
    The URL should use the Docker Compose service name and internal port.
-   **Status/Health Checks (`status_endpoint`):** Optionally provide a simple HTTP endpoint within your plugin that CypherpunkOS can query for basic health status (e.g., sync progress, connection status).
-   **Logging:** Ensure your node software logs to `stdout/stderr` so Docker and CypherpunkOS can capture and display logs.

## 6. Example 1: L2 Blockchain Node (e.g., Base)

-   **`plugin_type: node`**, **`category: chain`**
-   **Core Task:** Run `op-node` and `op-geth` (or equivalents for other L2s like Arbitrum Nitro).
-   **Dependencies:** Requires an L1 RPC endpoint (`APP_ETHEREUM_L1_RPC_URL` injected by CypherpunkOS).
-   **Configuration:** Receives L1 RPC URL, JWT secret path (if applicable), data directory paths.
-   **Exports:** `APP_L2_BASE_RPC_URL` (e.g., `http://op_node_base_service:8545`), `APP_L2_BASE_CHAIN_ID`.
-   Refer to `plan/plugin_l2_base.md` for more specific (though older `plugin_type: chain`) details that can be adapted.

## 7. Example 2: Subgraph Node (e.g., Uniswap V3 Mainnet Subgraph)

-   **`plugin_type: node`**, **`category: subgraph_node`**
-   **Core Task:** Run `graph-node` service, potentially with a bundled or linked PostgreSQL database.
-   **Dependencies:**
    *   Requires an Archive Node RPC for the network the subgraph indexes (e.g., `APP_ETHEREUM_L1_ARCHIVE_RPC_URL` for a mainnet subgraph, injected by CypherpunkOS).
    *   Requires a PostgreSQL database.
    *   Requires an IPFS node/gateway for fetching subgraph manifests and data.
-   **Configuration:** Receives PostgreSQL connection string, Ethereum RPC URL, IPFS endpoint. The specific subgraph to sync (e.g., by deployment ID `Qm...` or subgraph name `uniswap/uniswap-v3`) would ideally be configurable, perhaps via `custom_fields` in the manifest or a setting if CypherpunkOS provides a UI for node plugin settings.
    ```yaml
    # Example custom_fields for a specific subgraph node in cypherpunk-plugin.yml
    custom_fields:
      subgraph_name: "uniswap/uniswap-v3"
      subgraph_deployment_id: "Qmabcdef..."
      network_rpc_env_var: "APP_ETHEREUM_L1_ARCHIVE_RPC_URL" # Tells graph-node which env var to use
    ```
-   **Exports:** `APP_SUBGRAPH_UNISWAP_MAINNET_HTTP_URL` (e.g., `http://graph_node_service:8000/subgraphs/name/uniswap/uniswap-v3`), `APP_SUBGRAPH_UNISWAP_MAINNET_WS_URL`.
-   **Implementation Notes:**
    *   The `graph-node` Docker container would be configured to connect to its PostgreSQL DB and the Ethereum archive node.
    *   A startup script or entrypoint for the `graph-node` container might be needed to use the `subgraph_deployment_id` to deploy/sync the specific subgraph if not handled by `graph-node` environment variables directly.

## 8. Testing Node Plugins

-   Ensure the Docker services start correctly and are stable.
-   Verify data is being persisted in the specified volumes.
-   Check logs for errors or correct operation.
-   If an RPC/GraphQL endpoint is exported, test connectivity to it from another container or using tools like `curl` or a GraphQL client from within the Docker network if possible, or by temporarily exposing a port for testing.
-   For blockchain nodes, monitor sync progress.
-   For subgraph nodes, check indexing status and query the subgraph for data.

---
This guide provides a framework for developing Node Plugins. The specifics will vary greatly depending on the type of node software being packaged. Always refer to the official documentation for the node software itself for configuration and operational details. 