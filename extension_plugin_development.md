# CypherpunkOS Extension Plugin Development Guide

## 1. Introduction

This guide provides instructions and best practices for developing **Extension Plugins** (`plugin_type: extension`) for CypherpunkOS. Unlike App Plugins (which have their own distinct UIs and typically run in dedicated Docker containers), Extension Plugins primarily modify, enhance, or integrate with the core CypherpunkOS frontend or backend services.

Extensions are loaded directly by CypherpunkOS, often by the frontend for UI modifications or by the backend for new configurations and services. This model is inspired by how applications like VS Code load extensions to augment functionality and appearance.

Extensions can serve various purposes, such as:
-   **UI Customization Extensions (Themes):** Altering the look and feel of the CypherpunkOS dashboard (e.g., providing CSS for a Solarized theme).
-   **Service/Configuration Extensions (RPC Providers, etc.):** Adding new backend capabilities or configuration options that are managed via the CypherpunkOS UI and backend (e.g., an Infura RPC Provider that adds new RPC sources to network settings based on a user-provided API key).

CypherpunkOS manages the lifecycle of these extensions (installation, activation) and integrates them based on their declared `category` and manifest definitions.

## 2. Core Concepts of an Extension Plugin

-   **Direct Integration:** Extensions are typically sets of assets (CSS, JS, images, manifest) and metadata that CypherpunkOS directly consumes. They often do not run their own persistent Docker processes unless they offer a significant, standalone backend service (which might then blur the line with a headless App Plugin).
-   **Frontend Loading for UI Extensions:** Themes and UI-modifying extensions have their assets (CSS, JS) loaded directly by the CypherpunkOS frontend.
-   **Backend Handling for Configuration Extensions:** Extensions providing new service configurations (like RPC providers) define their settings UI via their manifest; CypherpunkOS frontend renders this, and the CypherpunkOS backend stores and utilizes the configuration.
-   **Manifest-Driven:** The `cypherpunk-plugin.yml` manifest is crucial. It defines `plugin_type: extension`, a specific `category` (e.g., `theme`, `rpc_provider`), and any custom fields or schemas needed for its operation.
-   **No Dedicated Docker Container (Usually):** For simple themes or metadata-driven extensions (like the RPC provider example below), a dedicated running Docker container for the extension itself is not required. The "packaging" might still leverage Docker image format for distribution via the App Store, but only to deliver the manifest and assets.

## 3. General Plugin Structure (for non-Dockerized Extensions)

An extension plugin is typically a directory containing its manifest and assets:

```
my-custom-extension/
├── cypherpunk-plugin.yml    # Manifest file for the plugin
├── assets/               # Specific assets for the extension (e.g., CSS files, icons, images)
├── README.md             # Optional: Plugin-specific README
└── LICENSE               # Optional: License for your extension
```
For distribution, this directory would be packaged (e.g., as a ZIP archive or a minimal Docker image containing these files).

## 4. Developing a UI Customization Extension (Example: Theme)

This type of extension modifies the visual appearance of the CypherpunkOS dashboard.

**Example Use Case:** A "Solarized Theme" extension that applies the Solarized color palette (dark variant) to the CypherpunkOS UI.

### 4.1. `cypherpunk-plugin.yml` for Theme Extension

```yaml
id: "solarized-dark-theme"
name: "Solarized Dark Theme"
version: "1.0.0"
plugin_type: extension
category: theme

description: "A Solarized Dark theme for the CypherpunkOS dashboard."
developer: "Your Name"
icon: "assets/icon.png" # Icon for theme selector / App Store listing

custom_fields:
  main_css_file: "assets/css/solarized-dark.css"  # Path within the plugin package to its primary CSS file
  # preview_image: "assets/preview.png" # Optional: Larger preview for App Store / theme details
```

### 4.2. Theme Assets

The extension package would contain the CSS file and any other related assets (fonts, images).

**Directory Structure within plugin package:**
```
solarized-dark-theme/
├── cypherpunk-plugin.yml
└── assets/
    ├── css/
    │   └── solarized-dark.css
    ├── icon.png
    # ├── (optional) fonts/
    # └── (optional) images/
    # └── preview.png
```

**`assets/css/solarized-dark.css` (Example Content):**
```css
/* Solarized Dark Theme for CypherpunkOS */
:root {
  /* Solarized Dark Palette Mappings to CypherpunkOS CSS Variables */
  --cf-background-color: #002b36; /* base03 */
  --cf-surface-color: #073642;    /* base02 */
  /* ... other Solarized color mappings to --cf-variables ... */
  --cf-primary-color: #268bd2;    /* blue */
}
/* ... any other specific overrides ... */
```
(Refer to CypherpunkOS documentation for the official list of CSS variables.)

### 4.3. How it Works (Frontend Loading)

1.  **Installation:** User installs the theme extension. CypherpunkOS unpacks its files (manifest, assets) into a designated extensions directory (e.g., `/data/cypherpunkos/extensions/solarized-dark-theme/`).
2.  **Discovery:** The CypherpunkOS frontend, on startup or when themes are managed:
    *   Scans the extensions directory for manifests with `plugin_type: extension` and `category: theme`.
    *   Populates the theme selector UI with discovered themes.
3.  **Activation:** User selects "Solarized Dark Theme" in CypherpunkOS Settings -> Appearance.
4.  **Application:**
    *   The CypherpunkOS frontend reads the `main_css_file` path (e.g., `assets/css/solarized-dark.css`) from the active theme's manifest.
    *   It constructs the full path to this CSS file within the installed extension's directory.
    *   The frontend dynamically loads this CSS file into the main dashboard page (e.g., by adding a `<link rel="stylesheet">` tag or injecting the CSS content). This overrides the default dashboard styles.

### 4.4. Dockerfile/docker-compose.yml (for packaging only)

If using Docker images for distribution (instead of simple archives), the Dockerfile would be minimal, just to package the files:

```dockerfile
# Dockerfile for packaging a theme extension
FROM scratch
COPY cypherpunk-plugin.yml /cypherpunk-plugin.yml
COPY assets/ /assets/
```
No `docker-compose.yml` would be needed for the theme extension itself to *run*, as it doesn't start any services.

## 5. Developing a Service/Configuration Extension (Example: RPC Provider)

This extension allows users to configure CypherpunkOS to use a third-party RPC service.

**Example Use Case:** An Infura RPC Provider extension.

### 5.1. `cypherpunk-plugin.yml` for RPC Provider Extension

```yaml
id: "infura-rpc-ext"
name: "Infura RPC Provider"
version: "1.0.0"
plugin_type: extension
category: rpc_provider

description: "Adds Infura as a selectable RPC provider in CypherpunkOS network settings."
developer: "Your Name"
icon: "assets/infura_icon.svg"

# Defines fields CypherpunkOS frontend should render in this plugin's settings UI
settings_schema:
  - name: "infura_project_id"
    label: "Infura Project ID"
    type: "password" # Input type (text, password, boolean, select etc.)
    required: true
    placeholder: "Enter your Infura Project ID"
    description: "Your API key for accessing Infura services."

custom_fields:
  # Metadata CypherpunkOS backend uses to construct RPC URLs and determine support
  rpc_url_template: "https://{network_subdomain}.infura.io/v3/{infura_project_id}"
  supported_chains: # CypherpunkOS internal chain IDs this provider supports
    - "ethereum_l1"
    - "goerli_testnet"
  network_map: # Maps CypherpunkOS chainID to Infura's specific network identifier
    ethereum_l1: "mainnet"
    goerli_testnet: "goerli"
```

### 5.2. Extension Assets

Typically just the manifest and an icon:
```
infura-rpc-ext/
├── cypherpunk-plugin.yml
└── assets/
    └── infura_icon.svg
```

### 5.3. How it Works (Frontend Renders Settings, Backend Stores/Uses Config)

1.  **Installation:** User installs the RPC provider extension. Its files are placed in the extensions directory.
2.  **Discovery:** CypherpunkOS frontend/backend discovers the extension via its manifest.
3.  **Configuration UI:**
    *   User navigates to CypherpunkOS Settings -> Plugins -> Infura RPC Provider.
    *   The CypherpunkOS frontend reads the `settings_schema` from the `infura-rpc-ext` manifest and dynamically renders a settings form (e.g., an input field for "Infura Project ID").
4.  **Saving Configuration:**
    *   User enters their Infura Project ID and saves.
    *   The CypherpunkOS frontend sends the `{ "infura_project_id": "USER_KEY" }` data to the CypherpunkOS **core backend** via a secure API call, identifying the plugin ID (`infura-rpc-ext`).
    *   The CypherpunkOS core backend securely stores this configuration (e.g., in its database), associated with `infura-rpc-ext`.
5.  **Integration with Network Settings:**
    *   When the user is in CypherpunkOS global network settings for a chain (e.g., Ethereum L1):
        *   The CypherpunkOS frontend requests available RPC provider options from the core backend.
        *   The core backend iterates through all installed extensions with `category: rpc_provider`.
        *   For each, it checks its manifest's `supported_chains` and if its required configuration (e.g., `infura_project_id` for the Infura extension) is present in the stored settings.
        *   If the Infura extension supports "ethereum_l1" and has a Project ID configured, the backend informs the frontend to list "Infura" as an option.
    *   If "Infura" is selected as the RPC source for Ethereum L1:
        *   The CypherpunkOS core backend, when an App Plugin later requests RPC configuration for Ethereum L1, will use the Infura extension's manifest (`rpc_url_template`, `network_map`) and the stored `infura_project_id` to generate the full Infura RPC URL. This URL is then provided to the App Plugin.

### 5.4. Dockerfile/docker-compose.yml (for packaging only)

Similar to themes, if distributed as a Docker image, it's just for packaging the manifest and icon:
```dockerfile
FROM scratch
COPY cypherpunk-plugin.yml /cypherpunk-plugin.yml
COPY assets/ /assets/
```

## 6. General Development Considerations for Extensions

-   **Manifest is Key:** Your `cypherpunk-plugin.yml` is the primary interface with CypherpunkOS.
-   **`settings_schema`:** Use this to define any user-configurable parameters for your extension. The CypherpunkOS frontend will render the UI for these.
-   **`custom_fields`:** Use this for metadata that the CypherpunkOS backend needs to make your extension functional (like URL templates for RPC providers).
-   **Simplicity:** Keep extensions as simple as possible. If an extension starts requiring complex backend logic or its own persistent state beyond simple configuration, consider if it should be an App Plugin instead.
-   **Security:** While CypherpunkOS handles secure storage of settings, be mindful of any data your extension might cause to be displayed or handled.

## 7. Testing Extensions

-   **RPC Provider:**
    *   Verify the settings UI (defined by `settings_schema`) renders correctly within CypherpunkOS plugin settings.
    *   Confirm your provider appears as an option for supported chains in CypherpunkOS network settings only after its required settings (e.g., API key) are configured.
    *   Test that an App Plugin correctly receives and can use the RPC URL generated via your extension for a specific chain.
-   **Theme:**
    *   Ensure the theme appears in the CypherpunkOS theme selector.
    *   Visually inspect all parts of the CypherpunkOS UI to confirm styles apply correctly and maintain usability (contrast, readability).

---
This guide outlines a frontend-centric loading model for many types of Extension Plugins. The CypherpunkOS core system (both frontend and backend) will provide the necessary APIs and mechanisms to discover, load, configure, and integrate these extensions based on their manifests. More complex extensions requiring their own backend logic might still use a traditional Dockerized service approach, which would be detailed separately. 