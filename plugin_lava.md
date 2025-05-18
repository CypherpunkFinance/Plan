# Plugin: Lava Network RPC Provider

## 1. Overview

- **Plugin Name:** Lava Network RPC Provider
- **Plugin Type:** RPC Provider
- **Version:** 1.0.0
- **Author:** Cypherpunk Finance Team
- **License:** MIT (or specify)
- **Description:** Integrates Lava Network as an RPC provider, allowing users to connect to various supported blockchains via Lava's decentralized network of providers.

## 2. Purpose and Scope

This plugin enables users to utilize Lava Network for RPC access to multiple blockchains. It aims to simplify RPC endpoint configuration by providing "Lava" as a direct connection option for supported chains, leveraging Lava's public RPC infrastructure. This abstracts the need for users to manually find and manage individual RPC provider URLs for chains accessible through Lava.

## 3. Key Features

-   **Simplified Access:** Leverages Lava Network's public RPC access points. **Assumption:** No user-specific API key is required for the basic public service tier.
-   **Dynamic RPC Option:** Adds "Lava" to the list of connection types for chains supported by Lava Network and configured in CypherpunkOS.
-   **Centralized RPC Endpoint Logic:** Constructs or utilizes the appropriate Lava Network RPC endpoint for the selected chain.
-   **Chain Support Management:** Maintains a list of chains supported by Lava Network. (This could be static initially, with future enhancements to dynamically fetch it).

## 4. User Experience (UX)

### 4.1. Plugin Installation and Setup

1.  **Installation:** User installs the "Lava Network RPC Provider" plugin from the App Store.
2.  **Configuration (Assumed Minimal for Public Access):
    *   Navigate to the "Apps" or "Plugins" page within CypherpunkOS settings.
    *   Select the "Lava Network RPC Provider" plugin.
    *   **Assumption:** For basic public RPC access, no specific API key or user credentials are required. The settings page might be informational or allow selection of a preferred public Lava gateway if multiple options become available. If Lava introduces consumer-specific access tokens or staking requirements for RPC usage, this section would need to include input fields for such credentials.

### 4.2. Chain Connection Configuration (within CypherpunkOS Network Settings)

1.  User navigates to the global network settings in CypherpunkOS, or to a specific chain's configuration page.
2.  When configuring a specific chain supported by Lava (e.g., Ethereum Mainnet), "Lava" will appear as an option in the "Connection Source" or "RPC Provider" selection (alongside "Default Node", "Manual RPC", other RPC Provider Plugins like "Infura", etc.).
3.  If the user selects "Lava":
    *   CypherpunkOS will use the Lava Network plugin to determine and use the correct RPC URL for that chain via the Lava Network.
    *   Example hypothetical URL: `https://public-gateway.lava.network/rpc/{lava_chain_spec_id}/json-rpc`
4.  If a chain is not supported by the Lava Network plugin's configuration, the "Lava" option will not be displayed.

## 5. Technical Details

### 5.1. Plugin Manifest (`cypherpunk-app.yml`)

```yaml
id: lava-network-rpc-provider
name: Lava Network RPC Provider
version: 1.0.0
app_type: rpc_provider
description: Connect to blockchains using the Lava Network.
developer: Cypherpunk Finance Team
# ... other common manifest fields ...

custom_fields:
  # The official Lava documentation for consumers would be needed to confirm the exact gateway URL and chain identifiers.
  # This is a hypothetical structure.
  lava_public_gateway: "https://rpc.testnet2.lava.network/lava-sdk" # Example, actual mainnet gateway TBD
  supported_chains: # CypherpunkOS internal chain IDs
    - ethereum_mainnet
    - polygon_mainnet
    - cosmos_hub_mainnet
    - osmosis_mainnet
    # Add more as identified from Lava Network's supported list
    # (e.g., via their `show_all_chains` API or documentation)

  # Mapping from CypherpunkOS chainId to Lava specific chain spec ID
  # These IDs need to be sourced from Lava's official chain specifications.
  # The URL structure would also be based on Lava SDK/API consumer documentation.
  chain_spec_map:
    ethereum_mainnet: "ETH1" # Hypothetical Lava Spec ID for Ethereum Mainnet
    polygon_mainnet: "POLYGON1" # Hypothetical Lava Spec ID for Polygon Mainnet
    cosmos_hub_mainnet: "COS5" # Hypothetical Lava Spec ID for CosmosHub
    osmosis_mainnet: "OSMOSIS" # Hypothetical

  # The RPC URL template might vary based on how Lava exposes its services (e.g., gRPC, JSON-RPC, TendermintRPC)
  # For EVM chains, CypherpunkOS primarily needs JSON-RPC.
  # This template assumes a path-based routing on a Lava gateway.
  rpc_url_template: "{lava_public_gateway}/{lava_chain_spec_id}/json-rpc" # Highly Hypothetical
  # Alternative: Lava SDK might handle endpoint discovery, requiring different integration.
```

**Note:** The `lava_public_gateway`, `chain_spec_map`, and `rpc_url_template` are critical and would need to be defined based on official Lava Network documentation for RPC consumers. The example `rpc.testnet2.lava.network` is from the provider setup context and may differ for direct consumer RPC calls or mainnet.

### 5.2. Configuration Storage

-   **Assumption:** No user-specific API keys are stored if using a public, unauthenticated tier of Lava Network. If future versions support authenticated access (e.g., via user-provided Lava account details or tokens), these would be stored securely by CypherpunkOS as part of the plugin's settings.

### 5.3. Core Logic (Conceptual - to be implemented by CypherpunkOS based on plugin manifest)

-   **`isChainSupported(chainId: string): boolean`**: Checks if the given CypherpunkOS `chainId` is in the plugin's `supported_chains` list and has a mapping in `chain_spec_map`.
-   **`getRpcUrl(chainId: string): string | null`**:
    *   Retrieves the `lava_chain_spec_id` from `chain_spec_map` using the CypherpunkOS `chainId`.
    *   Constructs the RPC URL using the `rpc_url_template`, the `lava_public_gateway`, and the `lava_chain_spec_id`.
    *   If Lava provides an SDK or a different mechanism for endpoint resolution, this logic would adapt accordingly.

### 5.4. Integration with CypherpunkOS

-   CypherpunkOS discovers installed `rpc_provider` plugins like this Lava plugin.
-   When a user configures RPC settings for a specific chain, CypherpunkOS checks if this plugin supports the chain via `isChainSupported`.
-   If supported (and assumed no further user config needed for public tier), "Lava" is presented as an RPC source option.
-   Upon selection, CypherpunkOS uses `getRpcUrl` to obtain the RPC endpoint for that chain to be used by the system and other apps.

## 6. Supported Chains (Example CypherpunkOS Chain IDs)

This list would be populated based on official Lava Network documentation and the `chain_spec_map`.

-   `ethereum_mainnet`
-   `polygon_mainnet`
-   `cosmos_hub_mainnet`
-   `osmosis_mainnet`
-   *(Others as supported by Lava and mapped in the plugin)*

## 7. Success Criteria

-   **SC1:** User can install the Lava Network RPC Provider plugin from the App Store.
-   **SC2:** (Assuming no settings needed for public tier) The plugin is active post-installation.
-   **SC3:** When configuring a chain supported by Lava (e.g., `ethereum_mainnet`) in CypherpunkOS network settings, the "Lava" option appears as an RPC source.
-   **SC4:** If "Lava" is selected, CypherpunkOS successfully uses the correct Lava Network RPC endpoint for that chain (actual endpoint structure TBD from Lava docs).
-   **SC5:** For a chain *not* listed in the Lava plugin's `supported_chains` or `chain_spec_map`, the "Lava" option does *not* appear.

## 8. Future Enhancements

-   Dynamically fetch the list of supported chains and their Lava Spec IDs from a Lava Network API endpoint (e.g., mainnet version of `show_all_chains`).
-   Support for user-configurable Lava Network gateways (if applicable).
-   Integration with potential Lava SDKs if they simplify consumer access.
-   Support for authenticated access if Lava provides consumer-specific API keys, tokens, or requires staking for enhanced service tiers.
-   Allow selection of different API interfaces if Lava provides more than just JSON-RPC via the same spec ID (e.g., gRPC, TendermintRPC, REST), though CypherpunkOS primarily uses JSON-RPC for EVM compatibility.

## 9. Open Questions/Items Requiring Lava Documentation

-   What is the official mainnet public RPC gateway URL for consumers?
-   What is the definitive list of `chain_spec_id`s for chains available on Lava mainnet (e.g., for Ethereum, Polygon, Cosmos chains, etc.)?
-   What is the exact RPC URL structure for consumers to access specific chains and interfaces (e.g., JSON-RPC for Ethereum)?
-   Are there any authentication requirements (API keys, tokens) for consumers using the public RPC service? What are the rate limits? 