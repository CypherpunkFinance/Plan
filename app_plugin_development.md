# CypherpunkOS App Plugin Development Guide

## 1. Introduction

This guide provides instructions and best practices for developing **App Plugins** (`plugin_type: app`) for CypherpunkOS. App plugins are typically user-facing applications with their own distinct web-based user interfaces (UIs), such as DeFi dashboards (e.g., Uniswap, Aave) or privacy tools (e.g., Tornado Cash).

CypherpunkOS manages the lifecycle of these apps, provides them with necessary network configurations (like RPC endpoints and contract addresses for the selected blockchain), and proxies access to their UIs through the main CypherpunkOS dashboard.

This guide is inspired by the Umbrel app ecosystem ([GitHub - getumbrel/umbrel-apps](https://github.com/getumbrel/umbrel-apps)) and adapts its principles to the CypherpunkOS architecture.

For a concrete example of an App Plugin, please refer to the `plan/plugin_uniswap.md` document.

## 2. Core Concepts of an App Plugin

-   **Separate UI:** App plugins serve their own HTML/CSS/JavaScript frontends. CypherpunkOS does not render their UI directly but provides access to it, typically via a reverse proxy.
-   **Dockerized:** Each app plugin runs as one or more Docker containers defined in a `docker-compose.yml` file.
-   **Configurable via Environment Variables:** CypherpunkOS injects network configurations and other essential data into the app plugin's containers via environment variables. The app must be designed to read and use these variables.
-   **Network Agnostic (within its capabilities):** An app should be able to operate on any network it supports (declared in its manifest) by using the RPC endpoints and contract addresses provided by CypherpunkOS for the user's selected network.

## 3. Plugin Structure

An app plugin is a directory containing all its necessary files. This directory is typically structured as follows:

```
my-custom-app/
├── cypherpunk-plugin.yml    # Manifest file for the plugin
├── docker-compose.yml    # Docker Compose service definitions
├── Dockerfile            # Dockerfile to build the main app image
├── src/                  # Application source code (e.g., Node.js, Python, or static web assets)
├── assets/               # Optional: static assets like icons, images for the app UI
├── README.md             # Optional: Plugin-specific README
└── LICENSE               # Optional: License for your app plugin
```

## 4. Key Files

### 4.1. `cypherpunk-plugin.yml` (Manifest File)

This YAML file describes your app plugin to CypherpunkOS. It includes metadata for the App Store, defines how the app should be run, and lists its dependencies and capabilities.

**Key Fields for App Plugins:**

```yaml
id: "my-uniswap-clone-app"  # Unique identifier (e.g., lowercase, kebab-case)
name: "My Uniswap Clone"      # Human-readable display name for the App Store and UI
version: "1.0.0"             # Semantic versioning (e.g., 1.0.0, 1.0.1-beta)
plugin_type: app             # Must be 'app' for app plugins

description: "A brief description of your app and its functionality."
developer: "Your Name/Handle"
website: "https://your-app-website.com" # Optional: Official website of the app or project
repo: "https://github.com/yourname/my-uniswap-clone-app" # Optional: Source code repository
support: "https://your-support-forum.com" # Optional: Link to support channel/docs

# UI & Proxy Configuration
port: 3000                   # The internal port your app's web server listens on inside the container
path: "/my-uniswap-clone"    # The path on CypherpunkOS where this app will be accessible (e.g., http://cypherpunk.local/my-uniswap-clone)

# App Store Presentation
gallery:                     # Optional: List of preview image paths (relative to plugin root) for the App Store
  - "assets/screenshot1.png"
  - "assets/gallery/image2.jpg"
icon: "assets/icon.svg"      # Optional: Path to the app icon (SVG preferred, 256x256, no rounded corners)
release_notes: |             # Optional: Markdown formatted release notes for this version
  - New feature X added
  - Fixed bug Y

# Network Configuration (Crucial for DeFi apps)
supported_networks: # Network IDs CypherpunkOS recognizes (e.g., from cypherpunk_finance_overview.md)
  - "ethereum_l1"
  - "base_mainnet"
  - "arbitrum_one"

network_configs: # App-specific default configurations per network
  - network_id: "ethereum_l1"
    config:
      chain_id_ref: 1 # For CypherpunkOS to verify against the RPC source
      # App-specific contract addresses or other defaults for this network
      ROUTER_ADDRESS: "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D"
      DEFAULT_SUBGRAPH_URL: "https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v3"
  - network_id: "base_mainnet"
    config:
      chain_id_ref: 8453
      ROUTER_ADDRESS: "0x...BaseRouterAddress..."
      DEFAULT_SUBGRAPH_URL: "https://api.studio.thegraph.com/query/.../base-uniswap-v3/..."
  # ... more network configurations as needed

dependencies: [] # Typically empty for apps, as they rely on CypherpunkOS for network RPCs.
                 # Could list capabilities like `inbuilt_wallet_api_access` if the app needs to interact with core OS features.

status_endpoint: "/app-status" # Optional: A path within your app that CypherpunkOS can query for health/status.
```

### 4.2. `docker-compose.yml`

This file defines the Docker services that make up your app plugin. For most simple web apps, this will be a single service for the frontend/backend.

```yaml
version: "3.7"

services:
  app_frontend: # Name your service (e.g., 'frontend', 'app')
    image: your_dockerhub_username/my-uniswap-clone-app:${APP_VERSION:-latest} # Use APP_VERSION for updates
    restart: unless-stopped
    environment:
      # --- Standard CypherpunkOS Injected Variables (Examples) ---
      # These are injected by CypherpunkOS based on user's network selection in the CypherpunkOS UI
      # or based on the default network if the app doesn't have internal network switching.
      - "APP_SELECTED_NETWORK_ID=ethereum_l1"               # e.g., ethereum_l1, base_mainnet
      - "APP_SELECTED_NETWORK_NAME=Ethereum Mainnet (External RPC)" # Human-readable, indicates source
      - "APP_SELECTED_NETWORK_RPC_URL=https://user.external.rpc.com" # The actual RPC URL to use
      - "APP_SELECTED_NETWORK_CHAIN_ID=1"                   # Actual chain ID of the selected network
      - "APP_SELECTED_SUBGRAPH_URL=https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v3" # Effective subgraph URL

      # --- App-Specific Variables from network_configs (Example for Uniswap on Ethereum L1) ---
      # CypherpunkOS injects these based on APP_SELECTED_NETWORK_ID and the plugin's manifest
      - "ROUTER_ADDRESS=0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D"
      # - "ANOTHER_CONTRACT_ADDRESS=0x..."

      # --- Other Environment Variables for your App ---
      - "NODE_ENV=production"
      - "PORT=3000" # The port your app listens on *inside* the container (must match manifest.port)
      # - "LOG_LEVEL=info"

    # volumes:
      # - app_data:/app/data # Example for persisting data if your app needs it

    # ports:
      # - "3000" # DO NOT expose ports directly here. CypherpunkOS App Proxy handles external access based on manifest.port & manifest.path.

    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

# volumes:
  # app_data: # Define the volume if used
```

**Important Notes for `docker-compose.yml`:**
-   **Image Versioning:** Use `APP_VERSION` (injected by CypherpunkOS based on manifest `version`) to tag your Docker images for updates.
-   **No Direct Port Exposure:** Do not use the `ports:` mapping to expose your app to the host. CypherpunkOS's App Proxy will handle this based on `port` and `path` in your `cypherpunk-plugin.yml`.
-   **Environment Variables:** Your application *must* be designed to pick up its configuration (RPC URLs, contract addresses, etc.) from these environment variables.

### 4.3. `Dockerfile`

This file builds the Docker image for your application service(s).

**Example (Node.js web app):**

```dockerfile
# Stage 1: Build the application
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build # Assuming a build step for your frontend (e.g., React, Vue, Angular)

# Stage 2: Serve the application
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/build ./build
COPY package*.json ./
RUN npm install --only=production

# If you have a simple server (e.g., Express) to serve the build & handle API
# COPY server.js .

# If just serving static files with a generic server like 'serve'
# RUN npm install -g serve
# CMD ["serve", "-s", "build", "-l", "3000"] 

EXPOSE 3000 # Expose the port your app listens on (must match manifest & docker-compose)
CMD ["npm", "start"] # Command to run your application
```

**Dockerfile Best Practices:**
-   Use official base images and pin versions (e.g., `node:18.16-alpine`).
-   Employ multi-stage builds to keep final images small.
-   Minimize layers and ensure efficient layer caching.
-   Run applications as a non-root user for better security.

## 5. Developing the Application Logic

Many DeFi apps have open-source frontends (e.g., Uniswap Interface, Aave Interface). The typical process involves:

1.  **Forking the Original UI:** Create a fork of the official or a community-trusted version of the application's frontend.
2.  **Adapting for Environment Variable Configuration:** This is the most critical step.
    *   Identify where the original UI hardcodes or fetches RPC endpoints, chain IDs, contract addresses, and subgraph URLs.
    *   Modify the code to read these values from environment variables (e.g., `process.env.APP_SELECTED_NETWORK_RPC_URL`, `process.env.ROUTER_ADDRESS` in a Node.js context).
    *   Ensure your web3 provider (ethers.js, web3.js) is instantiated dynamically using these environment variables.
    *   If the app supports multiple networks internally, ensure it can use the full `APP_NETWORK_CONFIGS_JSON` provided by CypherpunkOS (see `cypherpunk_finance_overview.md` for its structure) to allow in-app network switching.
3.  **Build Process:** Ensure your Dockerfile correctly builds and serves the modified application.
4.  **UI/UX Considerations:**
    *   Clearly display the currently active network and the source of its RPC/subgraph data (e.g., "Ethereum Mainnet via Local Node", "Base via Infura Extension").
    *   Ensure a smooth user experience for any interactions that depend on the dynamic network configuration.

## 6. Environment Variables from CypherpunkOS

CypherpunkOS will inject the following types of environment variables into your app's primary service container:

-   **`APP_VERSION`**: The version string from your plugin's `cypherpunk-plugin.yml`.
-   **`APP_SELECTED_NETWORK_ID`**: The CypherpunkOS internal ID of the network the user has currently selected for this app (e.g., `ethereum_l1`, `base_mainnet`).
-   **`APP_SELECTED_NETWORK_NAME`**: A human-readable name for the selected network, potentially indicating the RPC source (e.g., "Ethereum Mainnet (Local Node)", "Base Mainnet (External RPC)").
-   **`APP_SELECTED_NETWORK_RPC_URL`**: The actual JSON-RPC URL that your app **must** use for all blockchain interactions for the selected network.
-   **`APP_SELECTED_NETWORK_CHAIN_ID`**: The numeric chain ID for `APP_SELECTED_NETWORK_RPC_URL`.
-   **`APP_SELECTED_SUBGRAPH_URL`**: The GraphQL endpoint URL for the subgraph that your app should use for the selected network. This is resolved by CypherpunkOS based on user settings and your app's manifest defaults.
-   **Custom Variables from `network_configs`**: Any key-value pairs you define under `config` for the `APP_SELECTED_NETWORK_ID` in your `cypherpunk-plugin.yml` will be injected as environment variables (e.g., `ROUTER_ADDRESS`, `DEFAULT_SUBGRAPH_URL`).
-   **`APP_NETWORK_CONFIGS_JSON`**: A JSON string containing an array of all fully resolved network configurations (RPC, Subgraph, contracts) that your app supports and are available. This is for apps that implement their own in-app network switcher.

Your application logic **must** prioritize these injected variables.

## 7. Data Persistence

If your app needs to store data persistently (e.g., user settings specific to the app, cached data), use Docker volumes:

1.  Define a volume in your `docker-compose.yml` (e.g., `app_data`).
2.  Mount this volume to a path inside your container (e.g., `/app/data`).
3.  Ensure your application writes its persistent data to this path.

**Note:** Data in volumes persists across app restarts and updates. It is removed when the app is uninstalled.

## 8. Status Endpoint (`/app-status`)

Optionally, your app can implement an HTTP endpoint (e.g., at `/app-status` relative to its root) that CypherpunkOS can query to check its health and gather basic information.

This endpoint should return a JSON response, for example:

```json
{
  "status": "running", // or "degraded", "error", "initializing"
  "version": "1.0.0", // App's own version
  "message": "Application is healthy",
  "selected_network_id": "ethereum_l1",
  "rpc_url_in_use": "http://nodeset.internal:8545"
}
```

## 9. Testing Guidelines

-   **Test all supported networks:** Ensure your app functions correctly with the configurations for each network listed in `supported_networks`.
-   **Test with different RPC sources:** Verify behavior when CypherpunkOS provides RPCs from a local node, an RPC Provider extension (like Infura), or a manually entered external RPC.
-   **Test data persistence:** Restart your app via the CypherpunkOS UI to ensure any data written to volumes is correctly persisted and reloaded.
-   **Test updates:** Simulate an app update to ensure data migration (if any) works and persistent data is not lost.
-   **Test UI responsiveness and clarity:** Ensure the UI is clear about the active network and data sources.

## 10. Example: Uniswap Plugin

Refer to `plan/plugin_uniswap.md` for a detailed plan of how a Uniswap App Plugin would be structured, including its manifest, Docker Compose (conceptual), and how it handles environment variables for network configuration.

---
This guide provides a comprehensive overview. Specific API details for interacting with CypherpunkOS core services (if needed beyond environment variables) will be documented in the main CypherpunkOS Developer Guide. 