# Plugin: Infura RPC Provider

## 1. Overview

- **Plugin Name:** Infura RPC Provider
- **Plugin Type:** Extension (RPC Provider)
- **Version:** 1.0.0
- **Author:** Cypherpunk Finance Team
- **License:** MIT (or specify)
- **Description:** Integrates Infura as an RPC provider, allowing users to connect to various supported blockchains via their Infura account. This plugin functions as an **extension**, integrating into the CypherpunkOS network settings UI rather than providing a standalone user interface.

## 2. Purpose and Scope

This plugin enables users to utilize their Infura API keys (Project IDs) to access blockchain networks. It simplifies RPC endpoint configuration by providing "Infura" as a direct connection option for supported chains within the CypherpunkOS settings, abstracting the need to manually enter RPC URLs. As an **extension**, it enhances the core functionality of CypherpunkOS.

## 3. Key Features

-   **API Key Configuration:** Securely stores the user's Infura Project ID.
-   **Dynamic RPC Option:** Adds "Infura" to the list of connection types for chains supported by Infura and configured in CypherpunkOS.
-   **Automatic RPC URL Generation:** Constructs the correct Infura RPC endpoint based on the selected chain and stored Project ID.
-   **Chain Support Management:** Maintains an internal list of chains supported by Infura and only offers the "Infura" connection option for these chains.

## 4. User Experience (UX)

### 4.1. Plugin Installation and Setup

1.  **Installation:** User installs the "Infura RPC Provider" plugin from the App Store.
2.  **Configuration:**
    *   Navigate to the "Apps" or "Plugins" page within CypherpunkOS settings.
    *   Select the "Infura RPC Provider" plugin and click "Settings".
    *   User is prompted to enter their "Infura Project ID".
    *   The Project ID is validated for basic format.
    *   Input field for the Project ID, with a save button.

### 4.2. Chain Connection Configuration (within CypherpunkOS Network Settings)

1.  User navigates to the global network settings in CypherpunkOS, or to a specific chain's configuration page.
2.  When configuring a specific chain (e.g., Ethereum Mainnet), there is a "Connection Source" or "RPC Provider" selection.
3.  If the Infura plugin is installed and a Project ID is configured:
    *   For chains supported by Infura (e.g., Ethereum Mainnet, Polygon), "Infura" will appear as an option in the dropdown/list (alongside "Default Node", "Manual RPC", etc.).
    *   Example: For Ethereum Mainnet, connection options might be: "Local Nodeset", "Manual RPC", "Infura".
4.  If the user selects "Infura":
    *   CypherpunkOS will use the Infura plugin to generate the RPC URL (e.g., `https://mainnet.infura.io/v3/YOUR_PROJECT_ID`).
    *   This URL is then used as the RPC endpoint for that chain.
5.  If a chain is not supported by Infura, the "Infura" option will not be displayed for that chain's configuration.

## 5. Technical Details

### 5.1. Plugin Manifest (`cypherpunk-app.yml`)

```yaml
id: infura-rpc-provider
name: Infura RPC Provider
version: 1.0.0
plugin_type: extension # Updated plugin_type
category: rpc_provider # Specifies the type of extension
description: Connect to blockchains using your Infura Project ID.
developer: Cypherpunk Finance Team
# ... other common manifest fields ...

custom_fields:
  supported_chains: # Chain IDs recognized by CypherpunkOS
    - ethereum_mainnet
    - goerli_testnet
    - sepolia_testnet
    - polygon_mainnet
    - polygon_mumbai_testnet
    - arbitrum_one_mainnet
    - optimism_mainnet
    # Add more as supported by Infura and CypherpunkOS
  rpc_url_template: "https://{network_subdomain}.infura.io/v3/{projectId}"
  # Mapping from CypherpunkOS chainId to Infura specific network identifier (subdomain)
  network_map:
    ethereum_mainnet: "mainnet"
    goerli_testnet: "goerli"
    sepolia_testnet: "sepolia"
    polygon_mainnet: "polygon-mainnet"
    polygon_mumbai_testnet: "polygon-mumbai"
    arbitrum_one_mainnet: "arbitrum-mainnet"
    optimism_mainnet: "optimism-mainnet"
```

### 5.2. Configuration Storage

-   The Infura Project ID (e.g., `infura_project_id`) is stored securely by CypherpunkOS as part of the plugin's settings.

### 5.3. Core Logic (Conceptual - to be implemented by the plugin host or CypherpunkOS)

-   **`isChainSupported(chainId: string): boolean`**: Checks if the given `chainId` (CypherpunkOS internal ID) is in the plugin's `supported_chains` list.
-   **`getRpcUrl(chainId: string, settings: {projectId: string}): string | null`**:
    *   Retrieves the `network_subdomain` from `network_map` using `chainId`.
    *   Constructs the RPC URL using `rpc_url_template`, the found `network_subdomain`, and the stored `projectId`.

### 5.4. Integration with CypherpunkOS

-   CypherpunkOS discovers installed `rpc_provider` plugins.
-   When configuring a chain, CypherpunkOS iterates through these plugins.
-   For each plugin, it calls `isChainSupported(chainId)`.
-   If true and the plugin is configured (e.g., Infura Project ID entered), CypherpunkOS adds an option like "Infura" to the chain's RPC source selection.
-   If selected, CypherpunkOS calls `getRpcUrl(chainId, plugin_settings)` to get the endpoint.

## 6. Supported Chains (Example CypherpunkOS Chain IDs)

-   `ethereum_mainnet`
-   `goerli_testnet`
-   `sepolia_testnet`
-   `polygon_mainnet`
-   `polygon_mumbai_testnet`
-   `arbitrum_one_mainnet` (for Arbitrum One)
-   `optimism_mainnet` (for OP Mainnet)
-   (This list should align with Infura's offerings and CypherpunkOS's defined chain IDs)

## 7. Success Criteria

-   **SC1:** User can install the Infura RPC Provider plugin from the App Store.
-   **SC2:** User can navigate to the plugin's settings page and enter their Infura Project ID. The Project ID is saved by CypherpunkOS.
-   **SC3:** When configuring a chain supported by Infura (e.g., `ethereum_mainnet`) in CypherpunkOS network settings, the "Infura" option appears as an RPC source.
-   **SC4:** If "Infura" is selected, CypherpunkOS successfully uses the correct Infura RPC endpoint (e.g., `https://mainnet.infura.io/v3/<PROJECT_ID>`) for that chain.
-   **SC5:** For a chain *not* listed in the Infura plugin's `supported_chains`, the "Infura" option does *not* appear in CypherpunkOS network settings for that chain.
-   **SC6:** If the Infura Project ID is not configured in the plugin settings, the "Infura" option does not appear for any chain, or is greyed out.

## 8. Future Enhancements

-   Live API Key validation against Infura.
-   Dynamic fetching/updating of supported chains/network subdomains from Infura (if an API exists).
-   Displaying usage statistics or rate limit information (if Infura API allows and plugin can access this). 