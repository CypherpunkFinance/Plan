# Plugin: Tornado Cash Application

## 1. Overview

- **Plugin Name:** Tornado Cash Client
- **Plugin Type:** App
- **Version:** 1.0.0

## 2. Purpose and Scope

This plugin allows users to interact with Tornado Cash pools on supported networks (e.g., Ethereum L1) for enhanced transaction privacy. It provides a graphical interface for deposits, withdrawals, and note management. As an **app**, it operates with its own UI, launched from CypherpunkOS, and relies on CypherpunkOS for network configurations.

## 3. Key Features

## 4. Source Material

*   **Smart Contracts:** The plugin will interact with the officially deployed and audited Tornado Cash smart contracts. Contract addresses will need to be verified for each supported network.
*   **Frontend Source:** Several open-source Tornado Cash frontends have existed. A well-maintained, audited, and community-trusted fork would need to be identified or a new secure interface developed if no suitable open-source frontend is readily adaptable. Given the sensitivities, security and verifiability of the chosen frontend code are paramount.
    *   Example historical reference (may not be current or secure): `tornadocash/ui` or `tornadocash/classic-ui` (research needed for a viable base).
*   **Zero-Knowledge Proofs:** Tornado Cash relies on zk-SNARKs. The frontend will need to handle proof generation (usually done client-side in JavaScript/WASM) and potentially download proving keys.

## 5. Core Components & Dependencies

1.  **Frontend Service (`frontend` in `docker-compose.yml`):**
    *   **Image:** A Docker image built from a secure and audited Tornado Cash UI fork/implementation.
    *   **Functionality:** Serves the Tornado Cash web UI. The UI must:
        *   Securely generate and manage user notes (secrets). This is critical and needs robust client-side handling. Exporting/importing encrypted notes should be supported.
        *   Handle deposit and withdrawal flows.
        *   Generate zk-SNARK proofs client-side for withdrawals.
        *   Interact with the Tornado Cash smart contracts via the `APP_SELECTED_NETWORK_RPC_URL` provided by CypherpunkOS.
        *   Adapt to the selected network (`APP_SELECTED_NETWORK_CHAIN_ID`) for contract addresses and RPC.
    *   **Proving Keys:** The frontend will likely need access to Tornado Cash proving keys. These are typically large files and should be either:
        *   Bundled with the Docker image (increases size but ensures availability).
        *   Downloaded on demand by the frontend from a trusted, verifiable source (e.g., IPFS hash specified in the plugin). CypherpunkOS could potentially facilitate or cache these.
2.  **Status Service (Conceptual):**
    *   Implements `/app-status`.
    *   Reports basic status (e.g., "running", app version), configured network, and potentially the status of proving key availability.

## 6. `cypherpunk-app.yml` (Tornado Cash Plugin Manifest)

```yaml
id: tornadocash-app
name: Tornado Cash Client
version: 1.0.0
plugin_type: app
description: Make private transactions with Tornado Cash.
developer: Tornado Cash Community / Cypherpunk Finance Team
website: "<official_tornado_cash_info_site_if_any_or_protocol_spec>"
repo: "<link_to_cypherpunk_tornado_cash_plugin_repo>"
support: "<link_to_support_channel>"

# Tornado Cash has deployments on L1 and some L2s.
supported_networks:
  - "ethereum_l1"
  - "arbitrum_one" # Example, verify actual L2 deployments
  # Add other L2 network IDs if Tornado Cash contracts are deployed and supported

network_configs:
  - network_id: "ethereum_l1"
    config:
      chain_id_ref: 1
      # Contract addresses for various TC pools on Mainnet
      eth_0_1_address: "0x12D66f87A04A9E220743712cE6d90b665261626f"
      eth_1_address: "0x47CE0C6eD5B0Ce3d3A51fdb1C52DC66a7c3c2936"
      # ... other pools (DAI, cDAI, USDC, USDT, WBTC)
      # Proving key URIs/hashes for L1 pools if downloaded on demand
      proving_key_0_1_url: "ipfs://<hash_for_0.1_eth_proving_key>"

  - network_id: "arbitrum_one"
    config:
      chain_id_ref: 42161
      # Contract addresses for TC pools on Arbitrum One (illustrative)
      eth_0_1_address: "0x...ArbitrumTC_0.1ETH..."
      # Proving key URIs/hashes for Arbitrum pools
      proving_key_0_1_url: "ipfs://<hash_for_arbitrum_0.1_eth_proving_key>"

dependencies: [] # Relies on CypherpunkOS to provide RPC for the selected network

port: 3002 # Internal port for the frontend (example)
path: "/tornado-cash" # Access path via CypherpunkOS proxy
status_endpoint: "/app-status"

# Security Note: This plugin interacts with privacy-enhancing smart contracts.
# The security of the chosen frontend code and the handling of user notes/secrets are critical.
# Users should understand the operational security required for using Tornado Cash effectively.
security_implications: "high"
```

## 7. `docker-compose.yml` (Conceptual for Tornado Cash Plugin)

```yaml
version: "3.7"
services:
  frontend:
    image: your_repo/cypherpunk-tornado-cash-interface:latest
    restart: unless-stopped
    environment:
      # --- Injected by CypherpunkOS --- 
      - "APP_SELECTED_NETWORK_ID=ethereum_l1"
      - "APP_SELECTED_NETWORK_NAME=Ethereum Mainnet"
      - "APP_SELECTED_NETWORK_RPC_URL=${APP_ETHEREUM_L1_RPC_URL}"
      - "APP_SELECTED_NETWORK_CHAIN_ID=1"
      # From app's network_configs for selected network, injected by CypherpunkOS
      - "REACT_APP_ETH_0_1_ADDRESS=0x12D66f87A04A9E220743712cE6d90b665261626f"
      - "REACT_APP_PROVING_KEY_0_1_URL=ipfs://..."
      # --- End of CypherpunkOS injected variables ---
    ports:
      - "3002"
    # Potential volume for persistent encrypted note storage if not purely client-side browser storage.
    # volumes:
    #  - tornado_notes:/app/data # Path would depend on UI implementation

  # status_service: ... (if separate)

# volumes:
#   tornado_notes: {}
```

**Environment Variable Handling & Frontend Adaptation:**
*   The UI must be configured using the injected environment variables for network details, contract addresses, and potentially proving key locations.
*   Crucially, the handling of user-generated notes (the secret for withdrawals) must be done securely on the client-side. The plugin should offer robust ways for users to back up and restore these notes, possibly encrypted.

## 8. Build & Deployment Process

1.  **Select/Fork/Develop Secure Frontend:** This is the most critical step. The chosen UI code must be thoroughly vetted for security and correctness.
2.  **Integrate Network Configuration:** Adapt the frontend to use dynamic network configurations provided by CypherpunkOS via environment variables.
3.  **Handle Proving Keys:** Decide on bundling vs. on-demand secure download.
4.  **Dockerfile:** Create a Dockerfile for the frontend service.
5.  **Package Plugin:** Create `cypherpunk-app.yml` and `docker-compose.yml`.
6.  **Rigorous Testing:** Test deposit/withdrawal on L1 and supported L2s, using both local and external RPCs. Test note management.

## 9. User Interaction Flow

*   User installs Tornado Cash plugin.
*   User selects a supported network (e.g., "Ethereum Mainnet" or "Arbitrum One") within the Tornado Cash app UI (the app receives all valid network configs from CypherpunkOS at startup).
*   The app uses the RPC for the selected network (provided by CypherpunkOS, sourced from user's global external RPC or local node plugin).
*   User performs deposit: generates a note, submits transaction.
*   User performs withdrawal: provides note, frontend generates proof, submits transaction.

## 10. Considerations & Challenges

*   **Security of Frontend Code:** This is paramount. Using unaudited or questionable frontend code is a major risk.
*   **Note Management:** Secure client-side handling, backup, and restoration of user notes is vital. Loss of a note means loss of funds in that deposit.
*   **Proving Key Distribution/Management:** Ensuring users get legitimate proving keys.
*   **Legal and Regulatory Landscape:** Tornado Cash has faced significant regulatory scrutiny. The Cypherpunk Finance platform must make users aware of their own responsibilities to comply with local laws.
*   **RPC Provider Trust:** While transactions go through the user's chosen RPC, an adversarial RPC *could* potentially try to link deposit/withdrawal IPs if not used with a VPN/Tor, though the core Tornado Cash protocol aims to break on-chain linkage. The use of a local node (Nodeset or L2 plugin) mitigates this specific RPC-level risk.
*   **Maintaining `network_configs`:** Keeping contract addresses for Tornado Cash pools updated across different networks.

This plugin focuses on providing a *self-hosted interface* to existing, deployed smart contracts. It does not alter the Tornado Cash protocol itself. 