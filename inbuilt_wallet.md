# Inbuilt Wallet for Cypherpunk Finance

## 1. Objective

To provide a secure, self-hosted wallet integrated within CypherpunkOS. This wallet will allow users to generate/import private keys, manage their crypto assets (ETH and ERC20 tokens initially) on Ethereum L1 and supported L2s, view balances, and send/receive transactions through their configured local or external RPC endpoints. It aims to provide a convenient and private way to manage funds directly on the user's server, especially as a destination for funds processed through privacy tools like the Tornado Cash plugin.

## 2. Core Features

1.  **Wallet Creation & Import:**
    *   Generate new wallets (private key/mnemonic phrase).
    *   Import existing wallets using mnemonic phrase or private key.
    *   Secure storage of encrypted private keys/mnemonics on the server's storage, with user-defined strong password encryption. Access to wallet functions will require this password.
2.  **Asset Management:**
    *   Display ETH and ERC20 token balances for L1 and configured/selected L2s.
    *   Ability to add/remove custom ERC20 tokens.
    *   Fetch token balances and metadata using the configured RPC endpoint for the selected network.
3.  **Transaction Capabilities:**
    *   Send ETH and ERC20 tokens.
    *   Construct and sign transactions locally within the wallet backend.
    *   Broadcast signed transactions via the configured RPC endpoint for the selected network.
    *   Gas estimation and configurable gas settings (priority, max fee).
    *   Display transaction history for each account/network.
4.  **Network Switching (In-App):**
    *   Seamlessly switch between Ethereum L1 and any user-configured L2 networks (Base, Arbitrum, etc.) for which an RPC endpoint is available (local plugin or external).
    *   The wallet backend will consume the `APP_NETWORK_CONFIGS_JSON` (or similar structure provided by CypherpunkOS) to know which networks are available and their respective RPCs, chain IDs, etc.
5.  **Security:**
    *   Strong encryption for stored keys (e.g., AES-256-GCM using a key derived from user's master CypherpunkOS password).
    *   No exposure of private keys to the frontend UI; all signing operations happen in a secure backend component of the wallet app.
    *   Regular prompts for password for sensitive operations (e.g., sending transactions, exporting keys).
    *   Clear options for wallet backup (encrypted seed phrase/keys) and restoration guidance.
    *   **Defense-in-depth:** Consider filesystem permissions, minimizing attack surface of the wallet backend service, and regular security audits if custom code is significant.
6.  **User Interface:**
    *   Integrated into the CypherpunkOS dashboard.
    *   Clear display of accounts, assets per network, balances, and transaction history.
    *   User-friendly send/receive forms.
    *   Clear indication of which network is currently active for the wallet.
7.  **Privacy-Enhancing Workflow Integration:**
    *   Designed to easily receive funds from privacy-preserving tools like the Tornado Cash plugin, providing a local, self-controlled destination for anonymized funds.

## 3. Architecture (Conceptual)

The Inbuilt Wallet will likely consist of:

1.  **Backend Service (Wallet Core - running as a Cypherpunk App or Core OS Service):**
    *   Manages encrypted wallet storage.
    *   Handles cryptographic operations (key generation, derivation, transaction signing).
    *   Interacts with L1/L2 RPCs (from `APP_NETWORK_CONFIGS_JSON`) for on-chain data and transaction broadcasting.
    *   Exposes a secure local API (e.g., authenticated REST or gRPC over internal Docker network) for the frontend UI.
2.  **Frontend UI (Part of CypherpunkOS Dashboard):**
    *   Communicates with the Wallet Core backend API.
    *   Handles user interaction, displays information, initiates transaction requests (without handling keys).

## 4. Potential Wallet Backend Technologies/Approaches

The choice of backend technology is critical for security and functionality. Options include:

1.  **Adapt Existing Open-Source Server-Side Wallet Solution:**
    *   **Sequence Sidekick:** ([https://sequence.xyz/blog/sequence-sidekick-self-hosted-web3-backend-companion](https://sequence.xyz/blog/sequence-sidekick-self-hosted-web3-backend-companion))
        *   *Pros:* Open-source, Dockerized, TypeScript/Fastify, multi-chain EVM, offers querying, contract management, potential nonce/gas handling.
        *   *Cons/Investigation Needed:* Primarily designed as an application backend. Its key management (KMS integration, Sequence Smart Wallets) needs thorough evaluation to ensure it can be adapted for secure, user-controlled, server-encrypted keys where CypherpunkOS/user password is the root of trust, rather than an app or cloud KMS. Resource footprint (Redis, PostgreSQL) needs consideration.
    *   **wallet-chain-node (the-web3/dapplink-labs):** ([https://github.com/the-web3/wallet-chain-node](https://github.com/the-web3/wallet-chain-node))
        *   *Pros:* Open-source, Go-based, multi-chain (broader than EVM), API service (gRPC).
        *   *Cons/Investigation Needed:* States **"not yet audited, and should not be used in any production systems."** This requires extensive security auditing and hardening if considered. Suitability of gRPC for the CypherpunkOS UI.
    *   **nicsaul/mywalletservice:** ([https://github.com/nicsaul/mywalletservice](https://github.com/nicsaul/mywalletservice))
        *   *Pros:* Self-hosted, JavaScript (Node.js).
        *   *Cons/Investigation Needed:* Appears to be a smaller project. Requires deep dive into security, feature set (multi-network, ERC20, robust key handling, nonce management), and maintenance status.

2.  **Build Custom Wallet Backend using Libraries:**
    *   **Key Management & Crypto:**
        *   Libraries: `ethers.js`, `web3.js` (JavaScript); `go-ethereum/accounts`, `go-ethereum/crypto` (Go); `ethers-rs`, `rust-secp256k1`, `web3` (Rust).
        *   Key Storage: Encrypted files (e.g., AES-256-GCM with key derived from user's master password) on the server's filesystem.
    *   **Transaction Logic:** Constructing, signing, and broadcasting transactions for ETH and ERC20s across L1/L2s.
    *   **Blockchain Interaction:** Using client libraries (`ethers.js`, `go-ethereum/ethclient`, etc.) to interact with the configured RPC endpoints provided in `APP_NETWORK_CONFIGS_JSON`.
    *   **Backend Framework:** Node.js (e.g., Express, Fastify), Go (e.g., net/http, Gin), or Rust (e.g., Actix, Rocket) for the API service exposed to the UI.
    *   *Pros:* Full control over security model and features. Can be tailored precisely to Cypherpunk Finance needs.
    *   *Cons:* Significant development effort. High security burden. Requires deep expertise in cryptography and blockchain interaction.

**Initial Decision Point (Phase 0):** A thorough evaluation of Sequence Sidekick vs. a custom Go/Rust backend using established libraries is recommended. A custom Node.js backend is also possible but Go/Rust might offer better performance and security primitives for this type of service.

## 5. `cypherpunk-app.yml` (Inbuilt Wallet Plugin Manifest - if packaged as a distinct app)

```yaml
manifestVersion: 1
id: "cypherpunk-wallet"
name: "Cypherpunk Inbuilt Wallet"
version: "1.0.0"
app_type: "wallet_utility"
description: "Securely manage your crypto assets on L1 and L2s directly on your Cypherpunk Finance server."
developer: "Cypherpunk Finance Team"
repo: "<link_to_cypherpunk_wallet_repo>"

supported_networks: # Dynamically populated/used based on APP_NETWORK_CONFIGS_JSON
  - "ethereum_l1"
  - "base_mainnet"
  - "arbitrum_one"
  # ... etc.

dependencies: [] # Relies on CypherpunkOS to provide APP_NETWORK_CONFIGS_JSON

port: <internal_api_port_for_wallet_backend>
path: "/wallet"
status_endpoint: "/app-status" # Reports backend health, e.g., DB connection, basic API responsive
```

## 6. `docker-compose.yml` (Conceptual for Inbuilt Wallet App)

```yaml
version: "3.7"
services:
  wallet_backend:
    image: your_repo/cypherpunk-wallet-backend:latest # Or chosen open-source base image
    restart: unless-stopped
    environment:
      - "APP_NETWORK_CONFIGS_JSON=${CYPHERPUNK_OS_PROVIDED_NETWORK_CONFIGS_JSON}" # Standardized var from CypherpunkOS
      - "WALLET_DATA_PATH=/data/wallet"
      # Secrets for wallet encryption master key (if not solely derived from user password at runtime) managed by CypherpunkOS
    volumes:
      - "wallet_data:/data/wallet" # For encrypted key files, local transaction cache/db
    ports:
      - "<internal_api_port>"
    logging:
      driver: "json-file"
      options:
        max-size: "20m"
        max-file: "3"

volumes:
  wallet_data: # Stores encrypted wallet files
```

## 7. Interaction with CypherpunkOS and Other Apps

*   **Network Configurations:** The wallet backend receives the `APP_NETWORK_CONFIGS_JSON` from CypherpunkOS, enabling it to populate its network switcher and interact with any L1/L2 the user has configured (either via local plugins or external RPCs).
*   **Receiving from Tornado Cash:**
    1.  User uses the Tornado Cash plugin to withdraw funds to a fresh address.
    2.  The user obtains this fresh withdrawal address from their Cypherpunk Inbuilt Wallet (on the desired network, L1 or L2).
    3.  The Tornado Cash plugin uses the selected network's RPC (via CypherpunkOS) to broadcast the withdrawal transaction to this new address.
    4.  The Inbuilt Wallet, monitoring this address on the selected network (via its configured RPC), will eventually show the new balance.

## 8. Key Implementation Details & Challenges

*   **Security of Key Storage & Operations:** This is the absolute highest priority. Thoroughly vet encryption methods, key derivation, and ensure the backend service has minimal privileges and attack surface.
*   **Transaction Signing Logic:** Must be implemented correctly and securely, with no private key leakage to the frontend or logs.
*   **Nonce Management:** Robust nonce management across multiple accounts and networks is critical to prevent stuck or failed transactions.
*   **Multi-Network State Management:** The backend needs to manage account balances, transaction history, and nonces distinctly for each network a user interacts with.
*   **User Password Handling:** Securely deriving an encryption key from the user's main CypherpunkOS password for wallet data encryption. Handling password changes.
*   **Backup/Restore:** Providing a simple, secure, and reliable way for users to back up all necessary wallet data (encrypted seeds/keys) and understand the restore process.
*   **Gas Estimation on L2s:** L2s can have different fee mechanisms than L1; the wallet needs to handle this for accurate gas estimation.
*   **Error Handling & Reporting:** Clearly communicating blockchain errors or wallet operational errors to the user via the UI.

## 9. User Flow: Privacy-Enhanced Fund Transfer

1.  **User has an external wallet (e.g., MetaMask) with funds on L1.**
2.  **User has Cypherpunk Finance running with:**
    *   L1 access configured (local Nodeset or external L1 RPC).
    *   (Optional) An L2 like Base configured (local plugin or external L2 RPC).
    *   Tornado Cash plugin installed.
    *   Inbuilt Wallet initialized with a new, empty account (e.g., `CF_Account_1_L1` on L1, `CF_Account_1_Base` on Base).
3.  **Deposit to Tornado Cash (L1):**
    *   User opens Tornado Cash plugin, selects L1 network.
    *   Connects their external wallet (MetaMask) to the Tornado Cash UI (which uses the configured L1 RPC).
    *   Deposits ETH (or other supported asset) into a Tornado Cash pool from their external wallet.
    *   Securely saves the generated Tornado Cash note.
4.  **Withdraw from Tornado Cash to Inbuilt Wallet (e.g., on L1 or L2 Base):**
    *   User decides to withdraw to their local Cypherpunk Wallet on, for example, Base.
    *   In the Inbuilt Wallet, they select/view their Base account (`CF_Account_1_Base`) and copy its address.
    *   User opens Tornado Cash plugin, selects Base network (if TC supports Base and it's configured).
        *   The TC plugin now uses the configured Base RPC.
    *   User initiates a withdrawal, providing the saved note and the `CF_Account_1_Base` address as the recipient.
    *   The TC frontend generates the zk-proof. The transaction is signed (this step is tricky - usually TC frontends integrate with browser wallets for withdrawal signing; for full self-hosted withdrawal, the TC plugin might need to integrate with the Inbuilt Wallet's signing capability if the note itself doesn't act as a private key for withdrawal, or the user uses a relayer).
        *   *Alternative for withdrawal signing:* User could potentially use their external wallet (MetaMask) still connected to the TC interface, but ensure MetaMask is now configured to use the local Base RPC for this transaction. The recipient address would be their Inbuilt Wallet Base address.
    *   The withdrawal transaction is broadcast over the configured Base RPC.
5.  **Funds in Inbuilt Wallet:**
    *   The Inbuilt Wallet (monitoring its Base account via the configured Base RPC) eventually shows the received funds.
    *   These funds are now in a new address within the user's self-hosted Cypherpunk Finance wallet, with a degree of privacy from the original L1 deposit, depending on TC's effectiveness and user's operational security.

This flow highlights the synergy between the privacy tool and the self-hosted wallet, leveraging the user's own node infrastructure where possible. 