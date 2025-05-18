# Cypherpunk Finance - Planning Documents Overview

This directory contains all the planning documents for the Cypherpunk Finance project.

## Document Index

Below is a list of the planning documents and a brief description of their content:

-   **`cypherpunk_finance_overview.md`**:
    The master planning document. It details the project's background, motivation, key challenges, high-level architecture, core functionality, plugin system, comprehensive implementation plan (phased approach), README outline, documentation structure, API/tooling requirements, and lessons learned. This is the central document to understand the entire scope of CypherpunkOS.

-   **`c4_cypherpunkos_architecture.puml`**:
    A C4 PlantUML file describing the architecture of the CypherpunkOS base system. It includes System Context, Container, and Component diagrams to visualize the software structure.
    *   **Note for Visualization:** To view this `.puml` file as a diagram, you need to have Java and Graphviz installed on your system. PlantUML uses Graphviz (specifically the `dot` executable) to render diagrams. You can then use a PlantUML plugin in your IDE (like VS Code or IntelliJ) or a command-line tool to generate the image.

-   **`inbuilt_wallet.md`**:
    Details the design, features, user experience, technical specifications, and security considerations for the integrated self-hosted wallet within CypherpunkOS.

-   **`l2_integration.md`**:
    Focuses on the integration of Layer 2 (L2) solutions, including how L2 node plugins will work, how applications will become L2-aware, and the user experience for managing L2 connections.

-   **`mock_ui_plan.md`**:
    Outlines the plan for developing mock user interfaces (UIs) for various parts of CypherpunkOS, including the dashboard, app management, wallet, and settings.

-   **`plugin_aave.md`**:
    Describes the Aave DeFi plugin, including its purpose, features, user interface flow, technical details (like manifest and network configuration), and success criteria for L1 integration.

-   **`plugin_infura.md`**:
    Details the Infura RPC Provider plugin. It covers its overview, purpose, key features, user experience for setup and chain configuration, technical specifications (including the manifest and supported chains), and success criteria.

-   **`plugin_l2_arbitrum.md`**:
    Specific planning document for the Arbitrum L2 node plugin, covering its features, how it integrates with CypherpunkOS, and its dependencies (like requiring a configured L1 RPC).

-   **`plugin_l2_base.md`**:
    Specific planning document for the Base L2 node plugin, outlining its features, integration strategy with CypherpunkOS, and its reliance on an L1 RPC source.

-   **`plugin_lava.md`**:
    Describes the Lava Network RPC Provider plugin. This document details its purpose, features (assuming public access initially), user experience, technical specifications (with hypothetical manifest details pending official consumer docs), and open questions regarding Lava Network's consumer-facing API.

-   **`plugin_tornado_cash.md`**:
    Focuses on the Tornado Cash plugin, detailing its features for enhancing privacy, UI/UX considerations, technical aspects of integrating a secure frontend, note management, and proving key handling.

-   **`plugin_uniswap.md`**:
    Outlines the Uniswap DeFi plugin, covering its purpose, features, user interface, technical details for L1 integration (manifest, network config), and how it would use the `APP_NETWORK_CONFIGS_JSON`.

-   **`theme_plugin_development.md`**:
    Describes the architecture and development process for UI theme plugins. This includes the structure of a theme plugin, how CypherpunkOS will apply themes, security considerations, and the manifest (`cypherpunk-app.yml`) specifications for `app_type: theme`.

## Directory Structure
```
plan/
├── c4_cypherpunkos_architecture.puml
├── cypherpunk_finance_overview.md
├── inbuilt_wallet.md
├── l2_integration.md
├── mock_ui_plan.md
├── overview.md  <-- This file
├── plugin_aave.md
├── plugin_infura.md
├── plugin_l2_arbitrum.md
├── plugin_l2_base.md
├── plugin_lava.md
├── plugin_tornado_cash.md
├── plugin_uniswap.md
└── theme_plugin_development.md
``` 