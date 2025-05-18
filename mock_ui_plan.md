# Cypherpunk Finance - Mock UI Implementation Plan

## 1. Objective of the Mock UI

To create a static, clickable HTML/CSS/JS prototype of the Cypherpunk Finance dashboard (CypherpunkOS UI). This mock UI will visually represent the key user flows and interfaces for managing the system, installing/configuring plugins (L1 Nodeset, L2 Nodes, Applications), managing the inbuilt wallet, and viewing system status. 

The purpose is to:
*   Validate the user experience (UX) and user interface (UI) design concepts derived from the planning documents.
*   Provide a clear visual target for frontend and backend development.
*   Allow for early feedback on usability and layout.
*   Serve as a communication tool for stakeholders.

**This mock UI will have no real backend functionality.** All interactions will lead to pre-defined static states or visual feedback only.

## 2. Key UI Sections to Mock

Based on `plan/cypherpunk_finance_overview.md` and other plugin plans, the mock UI should include representations of the following sections and user flows:

**A. Main Dashboard / Home Screen:**
*   **Overall System Status:**
    *   L1 Node Status (e.g., "Ethereum Mainnet: Synced - Nodeset (Local)" or "Ethereum Mainnet: Connected - External RPC") - with mock sync percentage/block number.
    *   Brief status of key installed L2 Node Plugins (e.g., "Base: Synced - Local Plugin", "Arbitrum: Error - External RPC Unreachable").
    *   Quick overview of installed Application Plugins and their basic status (e.g., Uniswap: Running, Aave: Stopped).
    *   Alerts/Notifications area (mock alerts for "L2 Node Out of Sync", "App Update Available").
*   **Navigation:** Clear sidebar or top navigation to all other sections.
*   **Resource Usage Overview (Mocked):** Basic charts/graphs for CPU, Memory, Disk usage (simulating data that might come from `docker stats` or system monitoring).

**B. Settings Section:**
*   **System Settings:**
    *   Mock UI for changing system password, theme (dark/light), update settings.
*   **Network Configuration (Crucial Mockup):**
    *   **L1 Ethereum Configuration:**
        *   Option to use Local L1 Node (Nodeset - shown as default, with its status).
        *   Option to add/edit/select an External L1 RPC URL (fields for Name/Label, RPC URL, Chain ID (auto-filled or validated), Test Connection button - mock success/failure).
        *   Clear indication of which L1 source is currently active.
    *   **L2 Network Configuration:**
        *   List of supported L2 network types (e.g., Base, Arbitrum One, Optimism).
        *   For each L2 type, an option to:
            *   Use a Local L2 Node Plugin (if available in app store and installed - shows status).
            *   Add/edit/select an External L2 RPC URL (fields for Name/Label, RPC URL, Chain ID, Test Connection button - mock success/failure).
        *   Clear indication of the active RPC source for each L2 type.
    *   **(Optional) Subgraph URL Configuration:**
        *   Area to configure global default subgraph URLs for specific networks (e.g., "Default Ethereum Mainnet Subgraph URL for Analytics").
        *   (Advanced) Area to configure specific subgraph URLs for a particular App on a particular Network (e.g., "Uniswap on Base Subgraph URL").

**C. App Store:**
*   **Browse View:**
    *   Grid/list of available apps (Nodeset, L2 Node Plugins like Base/Arbitrum, Application Plugins like Uniswap/Aave/Tornado Cash, Inbuilt Wallet if it's an "app").
    *   Each app card showing: Icon, Name, Short Description, Category (L1 Node, L2 Node, DeFi, Privacy, Wallet), Version, Developer.
    *   Filter/Search functionality (mocked).
*   **App Detail View (when an app is clicked):**
    *   Full Description, Screenshots (gallery), Version History (mocked), Developer Info, Links (website, repo, support).
    *   "Install" button (becomes "Uninstall" or "Manage" if installed).
    *   Dependencies listed (e.g., "Requires L1 Ethereum RPC configuration").
    *   Supported Networks (for DeFi/Tooling apps, e.g., "Ethereum L1, Base, Arbitrum One").

**D. Installed Apps Management Area (per app type or a unified list):**
*   **For L1 Node Manager (Nodeset):**
    *   Status: Synced (percentage, current/highest block, ETA), Syncing, Error.
    *   Start/Stop/Restart buttons (mock actions).
    *   View Logs button (leads to a mock log viewer page).
    *   Configuration (if any specific to Nodeset beyond global L1 RPC if it *were* external, unlikely if Nodeset *is* the local provider).
*   **For L2 Node Plugins (e.g., Base Node Plugin):**
    *   Similar to L1: Status, Start/Stop/Restart, View Logs.
    *   Shows which L1 RPC it's currently using (from CypherpunkOS global settings).
*   **For Application Plugins (e.g., Uniswap, Aave, Tornado Cash):**
    *   Status: Running, Stopped, Error.
    *   Start/Stop/Restart/Uninstall buttons.
    *   View Logs button.
    *   **Network Selection UI (Mock - if the app itself doesn't have in-app switching, or to show CypherpunkOS providing the initial config bundle):**
        *   Dropdown to select active network for this app instance from its `supported_networks` for which an RPC is configured in CypherpunkOS.
        *   Shows current RPC & Subgraph source for the selected network (e.g., "RPC: External - MyAlchemyBase", "Subgraph: Default Public").
        *   *Note:* If the app has its *own* in-app network switcher (preferred final design), this section in CypherpunkOS might just show the list of networks the app was *launched with* (from `APP_NETWORK_CONFIGS_JSON`).
        *   Link to "Open App" (mock - opens a new tab or placeholder page).

**E. Inbuilt Wallet UI:**
*   **Main View:**
    *   Account selector/creator (mock multiple accounts).
    *   **In-App Network Switcher** (critical: dropdown for L1, installed/configured L2s).
    *   Selected network's balance (ETH and mock ERC20s).
    *   Transaction history (mocked entries).
*   **Send/Receive Modals/Pages:**
    *   Forms for Send (recipient, amount, asset, gas settings - mock estimations).
    *   Display Receive address (QR code).
*   **Settings/Security:**
    *   Mock UI for backup (show seed phrase - clearly marked as mock), change password, add custom tokens.

**F. Log Viewer:**
*   A generic page that can display mock log outputs for any selected app/node.
*   Basic filtering (info, warning, error - mock) and search (mock).

## 3. Design Principles/Look and Feel

*   **Clean & Modern:** Inspired by Umbrel's UI - simple, aesthetically pleasing, uncluttered.
*   **Dark Mode First:** Default to a dark theme, with a light theme option if time permits for the mock.
*   **Informative:** Clearly display status, warnings, and errors without overwhelming the user.
*   **Intuitive Navigation:** Easy to find different sections and settings.
*   **Responsive (Basic):** Should be viewable on desktop. Full mobile responsiveness is a plus but not critical for the initial mock if it adds too much time.
*   **Visual Consistency:** Use consistent iconography, typography, and layout patterns.

## 4. Technology Stack Recommendation (for the Mock UI)

*   **HTML5, CSS3, JavaScript (Vanilla JS or a simple framework/library):**
    *   **Why:** No complex build setup needed for a static mock. Easy to deploy (can be run directly from filesystem or any static host).
    *   Frameworks like **Tailwind CSS** (for rapid styling utility classes) or **Bootstrap** could be used for layout and components to speed up development.
    *   If a little interactivity beyond simple link navigation is needed (e.g., toggling mock modals, dropdowns), vanilla JS or a lightweight library like Alpine.js or petite-vue could be used.
*   **No Backend Database or Logic:** All data will be hardcoded in the HTML or simple JS objects for display purposes.
*   **Iconography:** FontAwesome, Material Icons, or SVG icons.

## 5. Step-by-Step Implementation Plan for the Executor

1.  **Setup Basic HTML Structure & Global Styles:**
    *   Create `index.html` as the main entry point.
    *   Set up a main CSS file (`style.css`) for global styles, fonts (e.g., Inter, Roboto), and color palette (dark theme focus).
    *   Implement the main layout: sidebar navigation and main content area.

2.  **Mock Main Dashboard / Home Screen:**
    *   Create static HTML for the L1/L2 status overview (hardcoded values for sync status, block numbers).
    *   Add placeholders for app plugin status.
    *   Create mock resource usage charts (can be static images or simple CSS bars).
    *   Implement the navigation links (pointing to `#` or other mock pages).

3.  **Mock Settings - Network Configuration UI:**
    *   Create HTML forms for L1 RPC (external vs. local Nodeset choice).
    *   Create UI for listing L2 types and configuring external RPCs for them (or showing local L2 plugin status if installed - mock this part).
    *   Mock UI for subgraph URL configurations.
    *   Buttons like "Save" or "Test Connection" will have no action or just show a mock success/failure message.

4.  **Mock App Store UI:**
    *   **Browse View:** Create HTML for app cards (Uniswap, Aave, Tornado, Nodeset, Base L2, Arbitrum L2, Inbuilt Wallet). Use placeholder icons/text.
    *   **App Detail View:** Create a template page for app details. Populate with mock data for one or two apps.
    *   "Install" buttons can change text to "Uninstall" on click (JS toggle) but do no actual operation.

5.  **Mock Installed Apps Management UI:**
    *   **L1 Nodeset Page:** Static display of sync status, logs button (links to mock log page).
    *   **L2 Node Plugin Page (e.g., for Base):** Similar to L1 Nodeset page.
    *   **Application Plugin Page (e.g., for Uniswap):**
        *   Status, Start/Stop (mock toggles).
        *   If demonstrating initial config (not in-app switching): Mock network selection dropdown. This dropdown, when changed, could use JS to show different hardcoded RPC/subgraph sources below it.
        *   "Open App" button (links to a placeholder page or external site if appropriate for mock).

6.  **Mock Inbuilt Wallet UI:**
    *   Create pages/sections for account overview (mock balances, transaction list).
    *   **Mock the In-App Network Switcher** (dropdown with L1, Base, Arbitrum - changing selection updates a display string like "Current Network: Base Mainnet").
    *   Basic forms for Send/Receive (non-functional).
    *   Mock UI for Backup/Settings.

7.  **Mock Log Viewer Page:**
    *   A simple page with `<pre>` tags containing sample log output (can have a few examples for different apps).

8.  **Interactivity & Refinements (Basic JS):**
    *   Use JavaScript for simple toggles (e.g., Install/Uninstall button text, modal popups for Send/Receive, expanding log details).
    *   Ensure navigation links work between the mocked pages.

9.  **Review and Iterate:** Get feedback on the mock UI and make adjustments to layout/flow.

## 6. Success Criteria for the Mock UI

*   All key UI sections listed in (2) are visually represented as static HTML pages.
*   Navigation between all mocked sections is functional.
*   The look and feel align with the design principles (clean, modern, dark-theme oriented).
*   Key user flows can be clicked through (e.g., navigating to settings, viewing an app in the store, opening an installed app's management page, switching networks *within* the mock wallet UI).
*   The mock UI clearly demonstrates how users would configure external RPCs vs. use local node plugins.
*   The mock UI shows how multi-network app configurations would be handled (even if the app handles internal switching, CypherpunkOS passes the initial bundle).
*   The mock UI is self-contained (no external backend dependencies) and can be easily viewed in a browser.

This plan provides a roadmap for the Executor to create a visual prototype of Cypherpunk Finance. The focus is on accurately representing the user interface and core interactions as planned, without implementing the underlying logic. 