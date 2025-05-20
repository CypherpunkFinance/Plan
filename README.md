# Cypherpunk Finance - Plan Documentation

This directory contains all the planning documents for the Cypherpunk Finance project.
For the overall project repository structure and a detailed explanation of plugin types (apps, extensions, chains), please refer to the "Repository Structure" and "Plugin (App) System - Cypherpunk App Framework" sections in **`cypherpunk_finance_overview.md`**.

## Document Index

-   **`cypherpunk_finance_overview.md`**: The master planning document. Details project background, repository structure, plugin architecture (apps, extensions, chains), core functionality, implementation phases, and more. **Start here to understand the project.**

-   **`c4_cypherpunkos_architecture.puml`**: A C4 PlantUML file describing the software architecture of CypherpunkOS, including System Context, Container, and Component diagrams.
    *   **Note for Visualization:** To view this `.puml` file as a diagram, you need to have Java and Graphviz installed on your system. PlantUML uses Graphviz (specifically the `dot` executable) to render diagrams. You can then use a PlantUML plugin in your IDE or a command-line tool to generate the image.

-   **`app_plugin_development.md`**: A guide detailing how to develop `plugin_type: app` plugins (e.g., Uniswap, Aave) for CypherpunkOS, based on Umbrel's app system principles.

-   **`extension_plugin_development.md`**: A guide for developing Extension Plugins, covering examples like RPC Providers (metadata-driven) and UI Themes (CSS-based), emphasizing frontend loading and manifest-driven integration (`plugin_type: extension`).

-   **`inbuilt_wallet.md`**: Details the design, features, and technical specifications for the integrated self-hosted wallet.

-   **`l2_integration.md`**: Focuses on Layer 2 (L2) solution integration, including L2 chain plugins and L2-awareness for apps.

-   **`mock_ui_plan.md`**: Outlines the plan for developing mock user interfaces (UIs) for CypherpunkOS.

-   **`plugin_aave.md`**: Describes the Aave DeFi plugin (`plugin_type: app`).

-   **`plugin_infura.md`**: Details the Infura RPC Provider plugin (`plugin_type: extension`, `category: rpc_provider`).

-   **`plugin_l2_arbitrum.md`**: Planning for the Arbitrum L2 node plugin (`plugin_type: chain`).

-   **`plugin_l2_base.md`**: Planning for the Base L2 node plugin (`plugin_type: chain`).

-   **`plugin_lava.md`**: Describes the Lava Network RPC Provider plugin (`plugin_type: extension`, `category: rpc_provider`).

-   **`plugin_tornado_cash.md`**: Focuses on the Tornado Cash plugin (`plugin_type: app`).

-   **`plugin_uniswap.md`**: Outlines the Uniswap DeFi plugin (`plugin_type: app`).

## Directory Structure

```
plan/
├── README.md  <-- This file
├── app_plugin_development.md
├── c4_cypherpunkos_architecture.puml
├── cypherpunk_finance_overview.md
├── extension_plugin_development.md
├── inbuilt_wallet.md
├── l2_integration.md
├── mock_ui_plan.md
├── plugin_aave.md
├── plugin_infura.md
├── plugin_l2_arbitrum.md
├── plugin_l2_base.md
├── plugin_lava.md
├── plugin_tornado_cash.md
├── plugin_uniswap.md
``` 