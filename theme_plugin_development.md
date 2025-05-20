# Theme Extension Development for Cypherpunk Finance

## 1. Overview

- **Plugin Type:** Extension
- **Category:** Theme
- **Purpose:** To allow users to customize the visual appearance (look and feel) of the main CypherpunkOS Web UI (Dashboard).
- **Mechanism:** Theme extensions provide CSS overrides and potentially other static assets (images, fonts) that CypherpunkOS can apply.

This document outlines the architecture and development process for UI Theme Extensions for CypherpunkOS. Themes are a specific type of **extension** that modify the CypherpunkOS dashboard's appearance.

## 2. Key Concepts

## 3. Theme Extension Structure

A theme extension will be packaged as a standard CypherpunkOS plugin (Docker container structure, though it might not run a long-lived process). The key is its manifest and the assets it provides.

*   **`cypherpunk-app.yml` (Manifest File):**
    ```yaml
    id: "my-custom-theme"
    name: "My Custom Theme Name"
    version: "1.0.0"
    plugin_type: extension    # Themes are a type of extension
    category: theme        # Specifies this extension is a theme
    description: "A brief description of what the theme looks like."
    developer: "Your Name/Handle"
    # repo: "<url_to_theme_source_code_if_any>"
    # website: "<url_to_theme_preview_or_info_if_any>"

    custom_fields:
      main_css_file: "theme/main.css"  # Path within the plugin's container to the primary CSS file
      # thumbnail_image: "theme/thumbnail.png" # Optional: Path to a preview image for App Store
      # assets_dir: "theme/assets/" # Optional: Path to a directory of other assets (images, fonts)
    ```
*   **Theme Assets Directory (e.g., `/theme/` inside the container):**
```
my-custom-theme/
├── css/
│   └── theme.css         # Main CSS file for the theme
├── assets/               # Optional: for images, custom fonts, etc.
│   ├── background.png
│   └── my-custom-font.woff2
├── cypherpunk-app.yml    # Manifest file for the plugin
├── README.md             # Optional: Description, preview images, instructions
└── LICENSE               # Optional: License for your theme
```

## 4. Developing Your Theme

**Prerequisites:**
*   Familiarity with CSS.
*   A local instance of CypherpunkOS (or a developer environment that can simulate theme loading).

**Steps:**

1.  **Create Plugin Directory:**
    Set up the directory structure as outlined in "Theme Plugin Structure."

2.  **Create `cypherpunk-app.yml`:**
    Populate the manifest file with your theme's details.

3.  **Write Theme CSS (`css/theme.css`):**
    *   **Start with CSS Variables:** Begin by overriding the official CypherpunkOS CSS variables. This is the recommended way to achieve broad stylistic changes.
        ```css
        /* Example: css/theme.css */
        :root {
          --cf-primary-color: #FF69B4; /* Hot Pink */
          --cf-background-color: #282c34; /* Dark Slate Gray */
          --cf-text-color: #abb2bf;    /* Light Gray */
          --cf-accent-color: #61AFEF;  /* Bright Blue */
          --cf-font-family: 'Arial', sans-serif;
          /* ... and so on for other official variables */
        }
        ```
    *   **Style Specific Components (if needed):** If overriding CSS variables isn't enough, you can target specific CypherpunkOS UI elements using their documented class names.
        ```css
        /* Example: Styling a specific component */
        .cf-sidebar {
          background-color: var(--cf-accent-color);
          border-right: 2px solid var(--cf-primary-color);
        }

        .cf-button-primary {
          background-color: var(--cf-primary-color);
          color: var(--cf-background-color);
        }
        ```
    *   **Use Custom Assets:**
        ```css
        body {
          background-image: url('../assets/my-background.png');
        }

        @font-face {
          font-family: 'MyCustomFont';
          src: url('../assets/my-custom-font.woff2') format('woff2');
        }

        :root {
          --cf-font-family: 'MyCustomFont', var(--cf-font-family-fallback, sans-serif);
        }
        ```

4.  **(Optional) Add Assets:**
    Place any images, fonts, or other static assets in the `/assets/` directory.

5.  **Testing Your Theme:**
    *   CypherpunkOS will provide a mechanism for developers to load and test unpublished themes (e.g., by placing the theme directory in a specific `themes` folder within the CypherpunkOS data directory).
    *   The CypherpunkOS dashboard will have a theme selector where your sideloaded theme should appear if correctly structured.
    *   Use browser developer tools to inspect elements, debug CSS, and iterate on your design.

6.  **(Optional) Create `README.md` and Previews:**
    Write a `README.md` file for your theme. Include:
    *   A description of the theme.
    *   Installation instructions (if any beyond selecting it in the UI).
    *   Screenshots or preview images (you can also link these in the `gallery` section of your manifest).

## 5. CSS Variables Reference (Preliminary - Subject to Change)
This section will be populated by the CypherpunkOS development team with the official list of CSS variables available for theming. Themes should prioritize using these variables.

| Variable Name                 | Default (Hacker Theme) | Description                                       |
|-------------------------------|------------------------|---------------------------------------------------|
| `--cf-primary-color`          | `#00FF00`              | Main interactive/accent color (e.g., buttons, links) |
| `--cf-secondary-color`        | `#00AA00`              | Secondary accent color                            |
| `--cf-background-color`       | `#0A0A0A`              | Overall page background                           |
| `--cf-surface-color`          | `#1A1A1A`              | Background for cards, modals, distinct surfaces   |
| `--cf-text-color-primary`     | `#E0E0E0`              | Primary text color on backgrounds                 |
| `--cf-text-color-secondary`   | `#A0A0A0`              | Secondary/muted text color                        |
| `--cf-text-on-primary-color`  | `#000000`              | Text color for elements using primary color as bg |
| `--cf-border-color`           | `#333333`              | Default border color for elements                 |
| `--cf-font-family-base`       | `monospace`            | Base font family for the UI                       |
| `--cf-font-family-heading`    | `monospace`            | Font family for headings                          |
| `--cf-font-size-base`         | `16px`                 | Base font size                                    |
| `--cf-error-color`            | `#FF4136`              | Color for error messages/indicators               |
| `--cf-warning-color`          | `#FF851B`              | Color for warning messages/indicators             |
| `--cf-success-color`          | `#2ECC40`              | Color for success messages/indicators             |
| `--cf-info-color`             | `#0074D9`              | Color for informational messages/indicators       |
| *... (more variables for padding, spacing, shadows, etc. will be added)* |

## 6. Best Practices

*   **Prioritize CSS Variables:** Utilize the official CSS variables for broad changes before targeting specific components. This ensures better compatibility with future UI updates.
*   **Accessibility:**
    *   Ensure sufficient color contrast (e.g., use tools to check WCAG AA/AAA compliance).
    *   Test readability with different font choices and sizes.
*   **Performance:**
    *   Avoid overly complex CSS selectors.
    *   Optimize images and other assets.
    *   Minimize the use of `@import` within your CSS file if possible.
*   **Responsiveness:** Test your theme across different screen sizes if the CypherpunkOS UI is designed to be responsive.
*   **Targeting UI Elements:**
    *   If you need to style elements not covered by CSS variables, use the most stable and high-level class names provided by CypherpunkOS.
    *   Avoid selectors based on highly nested structures or auto-generated class names, as these are prone to break with UI updates.
*   **Keep it Simple:** Start with a few key changes and iterate.
*   **Test Thoroughly:** Check all major sections and components of the CypherpunkOS UI to ensure your theme applies correctly and doesn't introduce visual bugs.
*   **Respect User Experience:** While themes allow for creativity, ensure the UI remains usable and intuitive.

## 7. Packaging and Distribution

*   **Packaging:** A theme plugin is simply its directory (e.g., `my-custom-theme/`) containing all the defined files. For distribution via the App Store, this directory might be zipped.
*   **App Store Submission:** Details on how to submit themes to the CypherpunkOS App Store will be provided once the App Store infrastructure is in place. This will likely involve a review process.

## 8. Future Considerations

*   **Light/Dark Mode Toggle:** Allowing themes to define variants for light and dark modes, or having CypherpunkOS provide a base toggle that themes can adapt to.
*   **Product Icon Theming:** Potentially extending theming to UI icons (like VS Code's Product Icon Themes), if the UI heavily relies on stylable icons (e.g., SVGs).
*   **User-configurable Theme Settings:** Allowing themes to expose a few simple configuration options (e.g., picking from a few accent colors within the theme), though this adds complexity.

---
This document provides a foundational plan. Specific CSS variable names, class names for theming, and the exact testing/sideloading mechanism will be finalized during CypherpunkOS UI development. 