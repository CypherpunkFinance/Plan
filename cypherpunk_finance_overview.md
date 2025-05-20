# Cypherpunk Finance: All of DeFi under your control

## Background and Motivation

This project aims to create an "Umbrel for Ethereum," providing a user-friendly home server experience focused on Ethereum (L1) and its Layer 2 (L2) scaling solutions, along with Decentralized Finance (DeFi) applications and privacy tools. The goal is to empower users to run their own Ethereum L1 nodes, selected L2 nodes (or use external ones), interact with DeFi protocols directly across these networks, manage their digital assets with enhanced privacy and control using an **inbuilt wallet**, and leverage privacy-enhancing technologies like **Tornado Cash via a dedicated plugin**. It mirrors the convenience and philosophy of Umbrel but is tailored to the multi-chain Ethereum ecosystem with a strong emphasis on financial sovereignty and privacy.

Current self-hosting solutions for Ethereum L1 and L2 infrastructure can be complex. Cypherpunk Finance seeks to simplify this by offering a curated App Store of L1/L2 node plugins (for local hosting), DeFi/tooling applications, and **UI theme plugins**, packaged for easy one-click installation and management, while also allowing users to connect to their preferred external RPC providers.

## Repository Structure

CypherpunkOS and its ecosystem will be organized into the following primary repositories:

*   **`cypherpunkos`**: This is the main repository for the CypherpunkOS core system.
    *   Contains the main backend service responsible for installing and managing plugins, coordinating the starting/stopping of their services, and fetching logs.
    *   Contains the primary frontend (Web UI/Dashboard) for browsing available plugins, installing/starting/stopping them, managing system settings, and viewing logs.
    *   A "plugin" is a general term for any installable component. The specific `plugin_type` in its manifest will define its behavior and how it integrates.

*   **`cypherpunkos-plugins`**: This repository will host the official collection of all installable components: "app", "extension", and "node" type plugins. This repository will look and function similarly to the `umbrel-apps` repository. CypherpunkOS will pull from this repository to build its app store locally, using a method similar to how Umbrel processes its app repository.
    *   Plugins will still have a `plugin_type` field in their manifest (e.g., `app`, `extension`, `node`) to define their behavior and integration with CypherpunkOS.
    *   **Apps** are plugins that have their own distinct frontend/UI, separate from the main CypherpunkOS control panel (e.g., Uniswap, Aave). CypherpunkOS will proxy access to these UIs.
    *   **Extensions** are plugins that primarily modify or enhance the CypherpunkOS control panel itself or provide backend functionalities without a dedicated UI (e.g., RPC Provider plugins, Theme plugins).
    *   **Nodes** are plugins specifically for running and managing backend services that provide data or network access (e.g., Ethereum L1 clients, L2 node clients, Subgraph nodes).

*   **`plan`**: (This current repository/directory)
    *   Contains all the planning documents, architecture diagrams, and detailed specifications for implementing CypherpunkOS and its various components.

## Key Challenges and Analysis

1.  **Core Ethereum L1 Node Management (Local Hosting Option):**
    *   The **Nodeset app** will be the default pre-packaged L1 Ethereum node runner application for users wishing to run a local L1 node. The system should be flexible to potentially accommodate other L1 node manager apps in the future.
    *   Ensuring stable and efficient operation of an Ethereum execution client and a consensus client via the active local L1 node app.
    *   Handling L1 updates and forks seamlessly through the active local L1 node app.
    *   Managing L1 blockchain storage for the active local L1 node app.
    *   Providing clear dashboard data for L1 node status (sync progress, peer info, errors) from the active local L1 node app or a configured external L1 RPC, via standardized APIs.
    *   Ensuring comprehensive logging for all L1 node activities (local or indicating connection to external).

2.  **L2 Node Management (via L2-specific Plugin Apps - for local hosting):**
    *   Each L2 (e.g., Base, Arbitrum) will be a separate, installable plugin app *for users wishing to run a local L2 node*.
    *   These L2 plugins must reliably run the respective L2 node software and depend on a configured L1 RPC source (local or external).
    *   Exposing standardized RPC endpoints for CypherpunkOS to provide to other apps.

3.  **External RPC and Subgraph Integration:**
    *   Allowing users to configure and prioritize global external L1/L2 RPC endpoints and (optionally) global or per-app/per-network external subgraph URLs in CypherpunkOS settings.
    *   Managing the reliability and security implications of user-provided external endpoints.
    *   Ensuring the dashboard clearly indicates the source of RPC/subgraph data for each network/app.

4.  **DeFi, Wallet, & Privacy Tool App Integration (Multi-Network Plugin Architecture):**
    *   Defining a clear manifest (`cypherpunk-plugin.yml`) for apps to declare `supported_networks` and default `network_configs` (contract addresses, public subgraph URLs).
    *   CypherpunkOS to compile a comprehensive set of network configurations (RPCs, subgraph URLs, contract addresses) for *all* supported and available networks for an app, and provide this to the app at startup. **Apps will then handle in-app network switching using this provided configuration set.**
    *   Prioritization for data sources for an app on a specific network: User-configured global external RPC/Subgraph for that network > Local active L1/L2 Node Plugin RPC > App manifest's default public subgraph URL.
    *   **Handling App Data Sources on L2s (e.g., Subgraphs):** Default to public L2 subgraph URLs from app manifest (overridable by user global settings) if no specific local/advanced subgraph solution is active.
    *   Maintenance of forked frontends to support smooth in-app network switching and dynamic data source/contract address usage.
    *   **Secure Inbuilt Wallet Implementation:** Designing and implementing a secure backend for key storage, transaction signing, and nonce management for the inbuilt wallet is critical. **Finding a suitable, secure, and self-hostable open-source backend wallet core or library is a key research task. Building one from scratch is a significant undertaking.**
    *   **Tornado Cash Plugin Security:** Ensuring the chosen/developed Tornado Cash frontend is secure, handles notes correctly, and manages proving keys safely.
    *   **User Workflow for Privacy Enhancement:** Designing a smooth UX for users to transfer funds from external sources, through the Tornado Cash plugin, and into their Cypherpunk Finance inbuilt wallet.

5.  **User Experience (UX):**
    *   Intuitive dashboard for managing local L1/L2 node plugins and **centralized external RPC/subgraph configurations.**
    *   **Application UIs must allow users to seamlessly switch between any of their configured and supported networks (L1 or L2s) without needing to go back to CypherpunkOS settings for each switch.**
    *   Clear visual cues within apps and the dashboard about the currently active network and the source of its data (local node, external RPC, public/external subgraph).
    *   Intuitive UI for the **inbuilt wallet** (account creation/import, balance display, send/receive, transaction history, network switching).
    *   Clear and secure UI for the **Tornado Cash plugin** (deposits, withdrawals, note management, proving key status).
    *   Guidance on operational security when using privacy tools.

6.  **Security:** (As previously defined, with added consideration for trusting user-provided external endpoints).
    *   Protecting user funds and private keys.
    *   Securely managing access to L1 and L2 RPC endpoints (local or external).
    *   Implementing robust authentication and authorization for CypherpunkOS and individual apps.
    *   **Inbuilt Wallet Security:** Protecting encrypted private keys at rest (e.g., on server disk) and ensuring secure transaction signing processes are paramount. The risk profile differs from client-side software wallets or hardware wallets and must be clearly communicated.
    *   **Tornado Cash Note Security:** Educating users on the importance of securely backing up Tornado Cash notes, as the plugin itself may not store them long-term or unencrypted.

7.  **App Ecosystem Development:** (As previously defined, with emphasis on apps being multi-network aware from a single configuration bundle).
    *   Encouraging developers to build and package Ethereum L1/L2 tools and DeFi apps that can leverage both local and external data sources.
    *   Providing comprehensive documentation for creating L1/L2 node plugins and application plugins (including multi-network support and external endpoint usage).
    *   Potentially attracting developers for more privacy-focused tools or wallet enhancements.

**8. Update Management for CypherpunkOS and Plugins:**
    *   **CypherpunkOS Core Updates:**
        *   Mechanism for notifying users of available OS updates (e.g., via dashboard).
        *   Process for applying OS updates:
            *   Minor updates should ideally be a one-click process from the UI.
            *   Major architectural updates might require a more guided process (e.g., OS re-flash for the system partition), with a strong emphasis on **preserving user data and application data stored on a separate partition/drive**.
            *   Clear communication regarding the nature and impact of any update.
        *   Robust rollback strategies and clear recovery guidance in case of failure, especially for major updates.
        *   Considerations for underlying system package updates versus CypherpunkOS-specific application updates.
        *   How to handle updates that might require system reboots or significant downtime (with user warnings).
    *   **Plugin (App) Updates:**
        *   The App Store should clearly indicate available updates for installed plugins (Nodeset, L2 Node Plugins, DeFi Apps, etc.).
        *   One-click update process for plugins from the App Store UI.
        *   Versioning: Plugins must have clear versioning (`version` field in `cypherpunk-plugin.yml`). CypherpunkOS should track installed versions and available versions.
        *   Dependency Management: How to handle updates if a plugin update requires a newer version of CypherpunkOS or another dependency? Clear messaging to the user is key.
        *   Data Migration: For plugins that store persistent data (e.g., L1/L2 node data, wallet data, app settings), robust mechanisms for how data will be migrated or preserved during an update. This includes both the application's own data and any OS-level configuration or metadata related to the app. This is especially critical for node plugins to avoid re-syncing.
        *   Testing and Vetting: Rigorous internal testing and potentially phased rollouts or community beta testing for significant updates. Actively monitor community feedback post-release to address unforeseen issues promptly.
    *   **Security Considerations for Updates:**
        *   Ensuring update packages are signed and verified to prevent malicious updates.
        *   Communicating the nature of updates (security fixes, new features, breaking changes) to users clearly.
    *   **User Control and Automation:**
        *   Should users have an option for automatic updates (opt-in)?
        *   Providing clear changelogs for all updates (OS and plugins).

**9. UI Theme Management (via Theme Plugins):**
    *   Themes are now considered a type of `extension` (`plugin_type: extension`, `category: theme`).
    *   Defining a clear structure for theme extensions (e.g., CSS overrides, image assets).
    *   Mechanism for CypherpunkOS to apply and switch between installed themes (default "hacker" theme and user-installed ones).
    *   Ensuring theme changes do not break UI functionality or readability.
    *   Security considerations for theme extensions (e.g., preventing malicious CSS/JS injection if themes are allowed more than CSS). For simplicity, initial themes might be restricted to CSS and static assets.
    *   Packaging and validation process for theme extensions in the App Store.

## High-level Overview of "Cypherpunk Finance"

"Cypherpunk Finance" OS (CypherpunkOS) is a personal home server OS enabling users to self-host Ethereum L1 and L2 nodes (or connect to external ones) and interact with DeFi applications in a sovereign and flexible manner.

**Core Functionality:**

*   **Operating System (CypherpunkOS):** Linux-based, optimized for running Ethereum L1/L2 clients (if hosted locally) and related services.
*   **Centralized Network Configuration:** CypherpunkOS settings will allow users to:
    *   Configure external RPC endpoints for L1 Ethereum and various L2 network types (e.g., Base, Arbitrum). These are global settings.
    *   Optionally, configure preferred external subgraph URLs (globally per network, or even per-app-per-network for advanced users).
*   **L1 Ethereum Node Access:** Via the default local Nodeset app, a potential alternative local L1 app, or a user-configured global external L1 RPC.
*   **L2 Node Access:** Via installable local L2 node plugins, or a user-configured global external RPC for that L2 type.
*   **Inbuilt Wallet:** A core feature of CypherpunkOS, providing a secure backend (potentially based on existing open-source solutions like Sequence Sidekick or a carefully vetted/developed custom solution) and UI for managing user funds (ETH, ERC20s) across L1 and configured L2s. It allows for key generation/import and transaction signing locally on the user's server.
*   **App Store:** Marketplace for:
    *   **Node Plugins:** (e.g., Nodeset L1, Base L2, Arbitrum L2, Uniswap Subgraph Node - running backend services).
    *   **App Plugins:** (e.g., Uniswap, Aave, Tornado Cash - with their own UIs).
    *   **Extension Plugins:** (e.g., Infura RPC, Lava RPC, UI Themes - extending CypherpunkOS functionality/UI).
*   **Web-Based UI (Dashboard):**
    *   Manages CypherpunkOS settings, including the global external RPC/subgraph configurations.
    *   Manages local node plugins (Nodeset, L2s).
    *   Manages Cypherpunk Apps: When an app is started, CypherpunkOS provides it with a full configuration bundle for all its supported networks, assembled based on global settings and local plugin availability. The dashboard itself does not handle per-app RPC selection after initial setup if the app supports in-app switching.
    *   Displays status/logs, clearly indicating data/RPC sources.
    *   **Integrated access to the Inbuilt Wallet.**
    *   Interface to manage and launch the Tornado Cash plugin.
    *   **RPC Provider Plugins:** (Now considered `plugin_type: extension`) A type of extension that allows users to configure third-party RPC services.
        *   **Installation & Configuration:** Users install an RPC Provider Plugin from the App Store and configure it with their API key/credentials via the plugin's settings page.
        *   **Functionality:** Once configured, the plugin makes itself available as an RPC source. CypherpunkOS, when displaying network configuration options for a specific chain, will query installed and configured RPC Provider Plugins.
        *   **Chain-Specific Options:** If an RPC Provider Plugin (e.g., Infura) supports the currently selected chain (e.g., Ethereum Mainnet), it will appear as a connection option (e.g., "Infura") alongside "Local Node" or "Manual RPC".
        *   **Automatic URL Generation:** Selecting the provider (e.g., "Infura") allows CypherpunkOS to retrieve the correct RPC URL from the plugin, using the stored API key and the specific chain. The user doesn't manually enter the URL for this option.
        *   **Provider Specificity:** Each RPC Provider Plugin declares which chains it supports. The option to use a specific provider will only appear for those supported chains. This mechanism allows for easy addition of new RPC providers by simply adding a new plugin.

**Plugin (App) System - "Cypherpunk App Framework":**

1.  **Containerization & Asset Delivery:**
    *   **App Plugins (`plugin_type: app`) and Node Plugins (`plugin_type: node`)** run their services in isolated Docker containers managed by CypherpunkOS.
    *   **Extension Plugins (`plugin_type: extension`)** primarily consist of metadata (manifest) and static assets (CSS, JS, images). While they might be packaged and distributed via a minimal Docker image for consistency in the App Store, they typically do not run their own persistent Docker services. Their assets are loaded and utilized directly by the CypherpunkOS frontend or their configuration is processed by the CypherpunkOS backend based on their manifest.

2.  **App Packaging:**
    *   **`docker-compose.yml`:** Standard Docker Compose for service definition (primarily for `app` and `node` plugins, can be minimal or absent for simple `extension` plugins that don't run services).
    *   **`cypherpunk-plugin.yml` (Manifest File):** Crucial for all plugin types.
        *   `id`, `name`, `version`, `description`, `developer`, `website`, `repo`, `support`, `port` (if exposing UI/service), `gallery`, `path` (to UI entry point if `plugin_type: app`), `status_endpoint`.
        *   `plugin_type`: Defines how the plugin integrates and behaves. Key types:
            *   `app`: A full application with its own distinct user interface (e.g., DeFi app like Uniswap). CypherpunkOS will proxy to its UI.
            *   `extension`: Modifies or enhances the main CypherpunkOS. This includes:
                *   RPC Providers (e.g., Infura, Alchemy) that add configuration options to CypherpunkOS settings.
                *   UI Themes that change the look and feel of the CypherpunkOS dashboard.
                *   Other backend services or UI augmentations without a full separate UI.
            *   `node`: Manages a backend service, typically providing data or network access (e.g., blockchain node, subgraph node, indexer).
            *   (Legacy type `chain` should be mapped to `node`).
        *   `category` (Optional): Further specifies the type of plugin. Examples:
            *   For `plugin_type: extension`: `rpc_provider`, `theme`.
            *   For `plugin_type: node`: `l1_node`, `l2_node`, `subgraph_node`, `indexer`.
        *   `supported_networks`: Lists network type IDs (e.g., `ethereum_l1`, `base_mainnet`). (Likely not applicable for `plugin_type: theme`)
        *   `network_configs`: Provides app-specific defaults (contract addresses, default public subgraph URLs) per network ID. This is fallback information if no user-specific external subgraph is configured.
        *   `dependencies`: Becomes less critical for basic RPC access if external RPCs are configured. Apps might still depend on `cap:ethereum_l1_rpc_available` or `cap:base_mainnet_rpc_available` to signal to CypherpunkOS which networks they are interested in receiving configurations for.
    *   **`exports.sh` (Especially for `plugin_type: node` like Nodeset and L2 Node Plugins):** These plugins export their RPC details. For Nodeset: `export APP_NODESET_L1_RPC_URL=...`, `APP_NODESET_L1_CHAIN_ID=...`. For an L2 plugin like `l2-base`: `export APP_L2_BASE_RPC_URL=...`, `APP_L2_BASE_CHAIN_ID=...`.

3.  **App Logging and Status Reporting:** (As previously defined). For apps using external RPCs, their status will primarily reflect frontend operation and connectivity to that external RPC.
4.  **App Proxy & Security:** (As previously defined).
5.  **Environment Variable Injection by CypherpunkOS:**
    *   CypherpunkOS will provide applications with a mechanism to receive configurations for *all* their supported networks for which a valid RPC endpoint (external or local) and subgraph URL (external or default) can be determined.
    *   **Example:** An environment variable `APP_NETWORK_CONFIGS_JSON` could contain a JSON string. This JSON would be an array of objects, each object representing a network the app supports and is configured for:
        ```json
        [
          {
            "networkId": "ethereum_l1",
            "networkName": "Ethereum Mainnet (Local Nodeset)", // or "(External RPC)"
            "rpcUrl": "http://nodeset.internal:8545",
            "chainId": 1,
            "subgraphUrl": "https://api.thegraph.com/subgraphs/name/uniswap/uniswap-v3", // Effective URL
            "contracts": { "router": "0x...", "factory": "0x..." } // From app's manifest
          },
          {
            "networkId": "base_mainnet",
            "networkName": "Base Mainnet (External RPC)",
            "rpcUrl": "https://user-provided-base.rpc.com",
            "chainId": 8453,
            "subgraphUrl": "https://user-provided-uniswap-base.subgraph.com", // User override
            "contracts": { "router": "0x...", "factory": "0x..." } // From app's manifest
          }
        ]
        ```
    *   The application frontend is then responsible for parsing this and allowing the user to switch between these pre-configured networks within the app UI.

6.  **Development and Submission:** (As previously defined, with emphasis on testing with both local and external endpoints).
7.  **Core "System" Apps (Managed by specific L1/L2 Node Plugins for local hosting):**
    *   The **Nodeset app** is the default for L1 Ethereum node operations. If another L1 node manager app is active, it takes this role.
    *   L2 Node Plugins manage their respective L2 node clients.
    *   The active local L1 node app (e.g., Nodeset) or L2 node app would expose necessary environment variables (RPC URL, ChainID) for CypherpunkOS to then provide to other apps.

This structure allows for a modular and extensible system where core infrastructure can be self-hosted or externally sourced, and users can easily add DeFi applications that leverage this flexible backend.

## Comprehensive Project Implementation Plan

This plan outlines a phased approach to building Cypherpunk Finance.

**Phase 0: Foundation & Core L1 (MVP)**
*   **Goal:** Establish basic CypherpunkOS, L1 node management (local via Nodeset OR external RPC config), minimal app framework, basic dashboard.
*   **Key Deliverables:**
    1.  Basic CypherpunkOS: Container management, settings UI for **global external L1 RPC configuration**.
    2.  Nodeset app (default local L1 provider): Installs/runs L1 clients, reports to dashboard.
    3.  App Framework: Apps receive config for *one* network (L1), based on external L1 RPC > local Nodeset.
    4.  Dashboard: Displays L1 status. UI to set L1 source.
    5.  Simple test app using the configured L1 RPC.
    6.  Initial `README.md`, basic developer and user docs for L1 (local/external).
    7.  Initial research and decision on core technology/approach for the Inbuilt Wallet backend (evaluate existing open-source vs. plan for custom build with selected libraries).

**Phase 1: Key DeFi Apps (L1) & Enhanced UX & External Subgraphs**
*   **Goal:** Introduce DeFi apps on L1 (using local/external RPC), allow external subgraph config, improve dashboard.
*   **Key Deliverables:**
    1.  Dashboard: UI for global external subgraph URL configuration (per network, and optionally per-app-per-network).
    2.  Uniswap/Aave Plugins (L1): Consume L1 config (RPC from external/Nodeset; Subgraph from external/app-default). **No in-app switching yet; network context set by CypherpunkOS at launch.**
    3.  Enhanced Dashboard: App store UI. Improved status/log display, clearly indicating if local/external RPC/subgraph is used.
    4.  Refined App Framework: `/app-status` API. CypherpunkOS correctly injects external or local RPC/subgraph URLs.
    5.  User documentation for L1 apps with external source options.

**NEW Phase (Example: Phase 2a - Privacy & Wallet Foundation)**
*   **Goal:** Introduce core privacy capabilities with Tornado Cash and the inbuilt wallet.
*   **Key Deliverables:**
    1.  **Inbuilt Wallet Backend & Basic UI:** Implement/Integrate chosen wallet backend. Secure key storage (encrypted), L1 ETH/ERC20 balance display, send/receive on L1 (using configured L1 RPC). Multi-account support. **Decision from Phase 0 on build vs. adapt will drive this heavily.**
    2.  **Tornado Cash Plugin (L1 ETH Pools):** Secure frontend for Tornado Cash ETH pools on L1. Handles note generation/management, proof generation (client-side), deposits/withdrawals using configured L1 RPC. Proving key management.
    3.  **Dashboard Integration:** Basic UI for accessing wallet and Tornado Cash plugin.
    4.  **Initial User Flow Documentation:** Guide for basic private transfer from external wallet -> TC (L1) -> Inbuilt Wallet (L1).
    5.  Security review of wallet key handling, chosen backend, and TC note management.

**Phase 2b: L2 Node Plugins & App L2 Compatibility (with External RPC/Subgraph Priority)**
*   **Goal:** Introduce L2 support and enable DeFi apps (including Wallet & TC if L2 contracts exist) on L2s.
*   **Key Deliverables:**
    *   (As previously defined for L2 plugins and app framework updates for L2).
    *   **Inbuilt Wallet L2 Support:** Extend wallet to support L2 networks (balances, send/receive) using `APP_NETWORK_CONFIGS_JSON`.
    *   **Tornado Cash Plugin L2 Support:** If viable TC contracts exist on supported L2s, extend plugin to support them, using network selection similar to Uniswap/Aave.

**Phase 3: Advanced Features & Ecosystem Growth**
*   **Goal:** Enhance data sovereignty (optional local subgraphs), developer tooling, community, and UI customization.
*   **Key Deliverables:**
    1.  (Optional/Advanced) Shared Graph Node Service in CypherpunkOS: For users wanting to self-index specific L1/L2 subgraphs. Apps can then be pointed to this.
    2.  Developer Tooling enhancements.
    3.  Community App Store support.
    4.  Security Audits.
    5.  Expanded L2 Plugin/external L2 support.
    6.  **Theme Plugin Support:** Framework for installing and applying UI themes. Release with default "hacker" theme and allow community contributions.

**Ongoing Throughout All Phases:** (As previously defined)

## README.md Outline
*   **Project Title:** Cypherpunk Finance
*   **Tagline:** Your Self-Hosted Gateway to Ethereum and DeFi on L1 & L2s – Your Nodes, Your Rules, Your Choice of RPC.
*   **Key Features:**
    *   Configure global L1/L2 RPC endpoints (use your own external nodes/services).
    *   Optionally run local L1 (Nodeset) and L2 node plugins.
    *   DeFi Apps automatically use your configured network settings.
    *   Seamlessly switch between supported networks (L1/L2s) *within* DeFi application UIs.
    *   **Secure Inbuilt Wallet:** Manage your assets locally on L1/L2s.
    *   **Privacy Tools:** Access applications like Tornado Cash through a self-hosted interface.
    *   Easy workflow for enhancing privacy of funds using Tornado Cash and the inbuilt wallet.
    *   **UI Customization:** Install custom themes (default "hacker" theme included) via theme plugins.
*   **Software Architecture Overview:** (CypherpunkOS, Nodeset as default L1 app, other L1 app potential, L2 Node Plugins, external RPC/subgraph config, Application Plugins, **Theme Plugins,** Docker).
*   **Installation Guide for CypherpunkOS.**
*   **Getting Started:**
    *   Configuring Global Network Settings (L1/L2 RPCs, Optional Subgraphs).
    *   (Optional) Installing Local Node Plugins (Nodeset, L2s).
    *   Using Application Plugins (and their in-app network switchers).
    *   Setting up and using the Inbuilt Wallet.
    *   Using the Tornado Cash plugin for private transactions.
*   **Developing Your Own Apps/Plugins:** (Including how to support external RPCs/subgraphs).
*   **(New) Developing and Installing Themes.**
*   **Example Application Plugins:**
    *   Tornado Cash Interface (Multi-network where applicable)

## Documentation Structure Outline
**I. Introduction**
    *   Core Concepts (CypherpunkOS, Plugin Types: Apps, Extensions, Nodes, Nodeset as default L1 chain plugin)

**II. User Guide**
    *   2.  The CypherpunkOS Dashboard:
        *   **Configuring Global External L1/L2 RPC Endpoints**
        *   **(Optional) Configuring Global/Per-App External Subgraph URLs**
        *   **(New) Installing and Managing Themes**
    *   3.  Managing L1 Ethereum: Using the Default Nodeset App vs. External L1 RPC.
    *   4.  Managing L2 Networks: Using Local L2 Plugins vs. External L2 RPCs.
    *   5.  Using Application Plugins:
        *   Understanding Automatic Network Configuration.
        *   **Using In-App Network Switchers.**
    *   (New) **5. Using the Cypherpunk Inbuilt Wallet**
        *   Security Model and Risks of a Self-Hosted Server Wallet
        *   Creating and Importing Wallets (Password, Encryption Details)
        *   Wallet Backup and Recovery (Critical Information)
    *   (New) **7. Using the Tornado Cash Plugin**
        *   Introduction to Tornado Cash Concepts & Risks
        *   Making Deposits
        *   Securely Managing Notes
        *   Performing Withdrawals (to Inbuilt Wallet or external)
        *   Operational Security Best Practices
    *   (New) **8. Privacy Workflow: Tornado Cash to Inbuilt Wallet**

**III. Developer Guide**
    *   1.  Cypherpunk Finance Architecture:
        *   **Plugin Categories: Apps, Extensions, Nodes (based on `plugin_type` and `category`).**
        *   **Centralized Network Configuration and Endpoint Resolution by CypherpunkOS.**
        *   **Application Responsibility for In-App Network Switching.**
        *   **(New) Inbuilt Wallet Backend Architecture (Chosen approach, security considerations)**
        *   **(New) Theme Plugin Architecture and Integration.**
    *   2.  Developing L2 Node Plugins: (No major change, still about local node provision).
    *   3.  Developing Application Plugins (`plugin_type: app`):
        *   Consuming `APP_NETWORK_CONFIGS_JSON`.
        *   Implementing an in-app network switcher.
        *   Designing UIs for multi-network display.
        *   **(New) Interacting with the Inbuilt Wallet (if APIs are exposed for other apps)**
    *   (New) 4. Developing Extension Plugins (`plugin_type: extension`):
        *   Integrating with CypherpunkOS UI (if applicable).
        *   Providing backend services.
        *   Example: RPC Provider extension, Theme extension.
        *   Manifest details: `plugin_type: extension`, optional `category`.
    *   (Renumber) 5. Developing Node Plugins (`plugin_type: node`):
        *   (Previously "Developing Chain Plugins").
        *   Packaging node software (blockchain clients, subgraph services, etc.), data management, exposing service endpoints (RPC, GraphQL).
        *   Manifest details: `plugin_type: node`, specific `category` (e.g., `l1_node`, `l2_node`, `subgraph_node`).
    *   (Renumber) 6. Developing Theme Extensions (Specific type of `extension`):
        *   Theme structure (CSS, assets).
        *   Manifest file (`cypherpunk-plugin.yml` with `plugin_type: extension`, `category: theme`).

## API Reference**
    *   2. Environment Variables provided to App Plugins: Details on `APP_NETWORK_CONFIGS_JSON` structure.

## Project Status Board
*   [ ] Phase 0: Foundation & Core L1 (MVP - with external L1 RPC option)
*   [ ] Phase 1: Key DeFi Apps (L1) & Enhanced UX & External Subgraphs
*   [ ] NEW Phase (Example: Phase 2a - Privacy & Wallet Foundation)
*   [ ] Design and Plan Inbuilt Wallet
*   [ ] Implement Inbuilt Wallet Backend (Phase 2a)
*   [ ] Implement Inbuilt Wallet Frontend UI (Phase 2a)
*   [ ] Design and Plan Tornado Cash Plugin (Phase 2a)
*   [ ] Implement Tornado Cash Plugin Frontend (Phase 2a)
*   [ ] Test Privacy Workflow (TC -> Inbuilt Wallet) (Phase 2a)
*   [ ] Phase 2b: L2 Node Plugins & App L2 Compatibility (with External RPC/Subgraph Priority)
*   [ ] Design and Implement External RPC/Subgraph Configuration System
*   [ ] Update App Framework for External Endpoint Prioritization

## API/Tooling Requirements & Analysis
(Largely the same, but with added emphasis on CypherpunkOS needing a secure datastore for user-provided URLs and potentially tools to test connectivity to these external endpoints.)

1.  **Ethereum Execution Client JSON-RPC API (for L1 via local L1 Node App, and for L2 nodes within L2 Plugins)**
2.  **Ethereum Consensus Client Beacon API (RESTful) (for L1 via local L1 Node App)**

6.  **Wallet Backend Technologies (for Inbuilt Wallet - to be researched/selected):**
    *   **Option A: Adapt Existing Open-Source Server-Side Wallet Solution:**
        *   **Sequence Sidekick:** ([https://sequence.xyz/blog/sequence-sidekick-self-hosted-web3-backend-companion](https://sequence.xyz/blog/sequence-sidekick-self-hosted-web3-backend-companion)) Open-source, Dockerized, TypeScript/Fastify backend. Offers querying, contract management, potential KMS integration. Need to verify its key management and signing can be adapted for user-controlled, server-encrypted keys rather than app-managed or cloud KMS.
        *   **wallet-chain-node (the-web3/dapplink-labs):** ([https://github.com/the-web3/wallet-chain-node](https://github.com/the-web3/wallet-chain-node)) Open-source Go-based API service for multiple chains. gRPC interface. **Crucially, states it's unaudited and not for production, requiring significant review/hardening if considered.**
        *   **nicsaul/mywalletservice:** ([https://github.com/nicsaul/mywalletservice](https://github.com/nicsaul/mywalletservice)) Self-hosted JS-based service. Needs deep evaluation for security, features, and maintenance.
    *   **Option B: Build Custom Wallet Backend using Libraries:**
        *   **Key Management & Crypto:** Libraries like `ethers.js`, `web3.js` (JavaScript), `go-ethereum/accounts` (Go), `ethers-rs` (Rust) for key generation (BIP39/32), signing. Standard crypto libraries (AES-GCM etc.) for key encryption at rest.
        *   **Blockchain Interaction:** `ethers.js`/`web3.js`/`ethclient`(Go)/`ethers-rs`(Rust) for RPC calls (balances, nonces, gas, broadcast tx).
        *   **Backend Framework:** Node.js (Express/Fastify), Go (net/http, Gin), or Rust (Actix/Rocket) for the API service.
    *   **API for UI:** Secure local REST or gRPC API for CypherpunkOS dashboard/wallet UI.

## Lessons
*   Centralizing network RPC configurations in CypherpunkOS while allowing apps to dynamically switch between them offers the best balance of user convenience (configure once) and app usability (switch easily in-app). Requires robust data passing from OS to App.
*   Environment variables ... (e.g., `APP_ETHEREUM_L1_RPC_URL` from CypherpunkOS, `APP_L2_BASE_RPC_URL` from L2 plugin). Clarify that `APP_ETHEREUM_L1_RPC_URL` is the *effective* L1 RPC from CypherpunkOS.
*   Pinning Docker images ... for both Cypherpunk Apps and the Ethereum clients managed by Nodeset (or other L1 apps) /L2 Plugins.
*   Managing L2s as distinct plugins (now `plugin_type: node`, `category: l2_node`) adds modularity but requires robust inter-plugin communication (or CypherpunkOS mediation) for RPC endpoints and app configuration.
*   Local subgraph hosting per-app-per-L2 is likely too resource-intensive for most users; a strategy involving public L2 subgraphs by default for UI data is more pragmatic, while transactions use the local L2 node.
*   (Add new) Integrating privacy tools like Tornado Cash requires extreme care in frontend security, note management, and user education due to inherent risks and regulatory attention.
*   (Add new) A self-hosted wallet is a powerful feature for sovereignty but demands a very high standard of security in its design and implementation.
*   (Add new) Choosing between building a core component like a wallet from scratch versus adapting an existing open-source project involves trade-offs between development speed, feature completeness, and the effort required for security auditing and customization.
*   (Add new) Server-hosted wallets present a different security model and set of responsibilities for the user compared to client-side or hardware wallets; this must be clearly communicated.

## Executor's Feedback or Assistance Requests

*(To be updated by Executor)*
*   Initial setup. Awaiting first specific task from Planner.
*   The scope has expanded significantly. Resource planning and phased implementation are crucial. Managing L2 node plugins and ensuring apps can seamlessly switch contexts is a primary architectural challenge.

## Lessons

*(To be updated as the project progresses)*
*   Umbrel's app system relies heavily on Docker, `docker-compose.yml` for service definition, and an `umbrel-app.yml` manifest for metadata and App Store integration. This will be the model for Cypherpunk Apps and Node Plugins. (Note: `umbrel-app.yml` will be `cypherpunk-plugin.yml` for this project).
*   The `app_proxy` service in Umbrel is key for managing access and authentication to apps.
*   Environment variables (especially those prefixed with `APP_` or specific to Nodeset like `APP_NODESET_EXECUTION_RPC_URL`, and new ones like `APP_L2_BASE_RPC_URL`) are crucial for inter-app communication and configuration.
*   Pinning Docker images to `sha256` digests is a best practice for security and reproducibility for both Cypherpunk Apps and the Ethereum clients managed by Nodeset/L2 Plugins.
*   Multi-architecture Docker builds are important for supporting diverse hardware (ARM, x86).
*   Community app stores are a supported concept, allowing broader app distribution.
*   Comprehensive, real-time status display (sync progress, errors) and accessible logs are crucial for user trust and diagnostics in a self-hosted node/app environment.
*   Detailed status/logging APIs are crucial for system observability.
*   Managing L2s as distinct plugins (now `plugin_type: node`, `category: l2_node`) adds modularity but requires robust inter-plugin communication for RPC endpoints and app configuration by CypherpunkOS.
*   Local subgraph hosting per-app-per-L2 is likely too resource-intensive for most users; a strategy involving public L2 subgraphs by default for UI data is more pragmatic, while transactions use the local L2 node. 