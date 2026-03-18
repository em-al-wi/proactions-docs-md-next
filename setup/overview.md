# Setup Overview

ProActions is an integrated automation framework for Eidosmedia's Swing Web Client and Prime Windows Client. This section guides you through the installation and configuration process.

## What You'll Need

Before setting up ProActions, ensure you have:

- **Swing or Prime Installation**: ProActions runs within Swing Web Client or Prime Windows Client
- **Server Access**: Ability to deploy files to the Méthode server
- **Configuration Access**: Permissions to edit configuration files in `/SysConfig/`
- **ProActions Hub** (Recommended): For secure API key management and service integration

## Setup Components

ProActions consists of several components:

### 1. ProActions

ProActions provides:

- Command palette (Ctrl+K) for quick action access
- Extension system for loading additional functionality
- Platform integration with Swing and Prime
- Base UI components
- YAML-based workflow configuration
- 80+ workflow steps for various operations
- Service integrations (OpenAI, DeepL, ElevenLabs, Stability AI, etc.)
- Form builder and UI components
- Multi-language support (English, Spanish, French, Italian, German)

### 2. ProActions Hub (Optional but Recommended)

A server component that provides:

- Centralized API key management
- Unified interface for 40+ AI providers
- EDAPI-based authentication and access control
- Proxy configuration
- Content extraction tools

## Setup Process Overview

The complete setup involves these steps:

1. **[Installation](./installation.md)**: Deploy ProActions files to your Swing/Prime server
2. **[Configuration](./configuration.md)**: Set up services and create your first workflows
3. **[Keyboard Shortcuts](./keyboard-shortcuts.md)**: Configure keyboard shortcuts for quick access

## Platform Support

ProActions works on both:

- **Swing Web Client**: Full browser-based integration
- **Prime Windows Client**: Desktop application integration

The same configuration files work across both platforms, with some platform-specific features available.

## Configuration File Structure

ProActions uses YAML configuration files organized as follows:

```
/SysConfig/
├── XsmileKbd.cfg                                 # Keyboard shortcuts
├── pro-actions.ai-kit.yaml                       # Main AI-Kit configuration
└── ProActions/
    ├── services.ai-kit.services.yaml             # Service configurations
    ├── oneclick.ai-kit.menu.yaml                 # One-click menu configuration
    └── sections/
        ├── headlines.ai-kit.section.yaml         # Example: headline generation
        ├── translation.ai-kit.section.yaml       # Example: translation workflows
        └── my-custom-actions.ai-kit.section.yaml # Your custom actions
```

## Deployment Locations

### Extension Files

ProActions extension files are deployed to:

```
~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/pro-actions/
```

This directory contains:

- `pro-actions-{version}.js` - The compiled ProActions bundle
- `pro-actions-config.js` - Core configuration file

### Configuration Files

Workflow and action configurations are stored in:

```
/SysConfig/
```

This location is accessible through the Méthode CMS configuration system.

## Development vs. Production

### Development Mode

In development mode:

- Changes to YAML files are reflected on browser refresh (F5)
- Debug information available in browser console
- Toggle development mode via ProActions menu

### Production Mode

In production:

- Configuration cache must be cleared after changes
- Changes require explicit cache refresh
- Error handling is more robust
- Performance optimizations enabled

## VS Code Integration (Optional)

For better YAML editing experience, you can configure VS Code with ProActions schema validation:

```json
{
  "yaml.customTags": ["!include scalar"],
  "yaml.schemas": {
    "https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/ai-kit.schema.json": [
      "*.ai-kit.yaml",
      "*.ai-kit.yaml*"
    ]
  }
}
```

This provides:

- Auto-completion for step names and properties
- Validation of YAML structure
- Inline documentation for configuration options

## Next Steps

1. **Start with [Installation](./installation.md)** to deploy ProActions to your server
2. **Follow [Configuration](./configuration.md)** to set up services and create your first workflows
3. **Configure [Keyboard Shortcuts](./keyboard-shortcuts.md)** for quick access
4. **Explore [Getting Started](../getting-started/your-first-flow.md)** to create your first workflow

## Getting Help

If you encounter issues during setup:

1. Check the browser console (F12) for error messages
2. Verify file permissions on the server
3. Ensure configuration files have valid YAML syntax
4. Review the [Troubleshooting](#troubleshooting) section below

## Troubleshooting

### ProActions doesn't appear in Swing

- Verify extension files are deployed to the correct directory
- Check that `pro-actions-config.js` is loaded
- Clear browser cache and reload
- Check browser console for loading errors

### Keyboard shortcut not working

- Verify `/SysConfig/XsmileKbd.cfg` is configured correctly
- Check for conflicts with other shortcuts
- Try the fallback hotkey configuration in `pro-actions-config.js`

### Configuration changes not visible

- In development mode: Refresh browser (F5)
- In production: Clear Swing configuration cache
- Verify YAML syntax is valid
- Check file permissions

### Actions not appearing

- Ensure YAML files are properly included in `pro-actions.ai-kit.yaml`
- Check section and action syntax
- Verify permissions (forUsers, forGroups, forTeams)
- Check `disabled` or `hidden` flags
