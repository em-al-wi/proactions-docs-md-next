# Installation

This guide walks you through installing ProActions on your Swing server or Prime.

## Prerequisites

Before installing ProActions, ensure you have:

- **Server Access**: SSH or file system access to your Méthode server
- **File Permissions**: Write permissions for the Swing web application directory
- **SysConfig Access**: Ability to edit files in `/SysConfig/`
- **Administrative Rights**: Permissions to modify system configuration

## Installation Steps

### Step 1: Obtain ProActions Build

ProActions is distributed as a compiled JavaScript bundle. Obtain the latest release build:

- Download from your Eidosmedia Artifactory, or
- Receive from your Eidosmedia account manager

The package should contain:

- `pro-actions-{version}.js` - The compiled extension bundle
- `pro-actions-config.js` - Default configuration file (optional)

### Step 2: Deploy Extension Files

Deploy the ProActions extension to your Swing server:

1. **Navigate to the plugins directory:**
   
   ```bash
   cd ~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/
   ```

2. **Create the ProActions directory (if it doesn't exist):**
   
   ```bash
   mkdir -p pro-actions
   ```

3. **Copy the extension files:**
   
   ```bash
   # Copy the main bundle
   cp /path/to/pro-actions-{version}.js pro-actions/
   
   # Copy the configuration file
   cp /path/to/pro-actions-config.js pro-actions/
   ```

4. **Verify file permissions:**
   
   ```bash
   ls -la pro-actions/
   # Files should be readable by the tomcat-swing
   ```

### Step 3: Create Core Configuration File

If you don't have a `pro-actions-config.js` file, use the one from the release bundle:

**File:** `~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/pro-actions/pro-actions-config.js`

```javascript
(() => {
  window.SwingProActionsConfig = {
    // Fallback keyboard shortcut for opening ProActions
    // (when editor context shortcut is not available)
    openHotkey: 'cmd+k,ctrl+k',

    // Training/Development mode
    // Set to true to enable local configuration testing
    trainingMode: false,
  };
})();
```

### Step 4: Register Keyboard Shortcuts

Configure keyboard shortcuts in the Swing keyboard configuration file:

**File:** `/SysConfig/XsmileKbd.cfg`

Add the following configuration within the `<context type="Text">` or the `<context type="all">` section:

```xml
<context type="Text">
  <!-- ... other keyboard shortcuts ... -->

  <key description="K" charCode="75">
    <!-- Open ProActions command palette -->
    <modifier type="C" shortcut="Ctrl+K">
      <function name="editorExtension" command="custom.openSwingProActions"/>
    </modifier>

    <!-- Instant action (optional) -->
    <modifier type="CS" shortcut="Ctrl+Shift+K">
      <function name="editorExtension" command="custom.instantSwingProActions"/>
    </modifier>
  </key>

  <!-- ... other keyboard shortcuts ... -->
</context>
```

**Note:** Adjust the keyboard shortcuts according to your preferences and existing shortcuts.

### Step 5: Create AI-Kit Configuration

Create the main AI-Kit configuration file:

**File:** `/SysConfig/pro-actions.ai-kit.yaml`

```yaml
AI_KIT:
  SERVICES:
    HUB:
      # ProActions Hub endpoint (recommended for API key management)
      endpoint: "{BASE_URL}/swing/proactions"
      target: openai
      defaultBehavior: |
        You are an assistant who helps journalists write their content.
        Answer without unnecessary explanations.
        Answer in English unless instructed otherwise.

  SECTIONS:
    # Your action sections will be included here
    # - !include /SysConfig/ProActions/sections/my-actions.yaml
```

### Step 6: Verify Installation

1. **Clear browser cache** (or open in incognito/private mode)

2. **Open Swing** in your browser

3. **Test the keyboard shortcut:**
   
   - Open any document in the editor
   - Press `Ctrl+K` (or `Cmd+K` on Mac)
   - The ProActions command palette should appear

4. **Check browser console** (F12) for any errors

## Directory Structure After Installation

After successful installation, your directory structure should look like this:

```
~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/
└── plugins/
    └── pro-actions/
        ├── pro-actions-{version}.js
        └── pro-actions-config.js

/SysConfig/
├── XsmileKbd.cfg
├── pro-actions.ai-kit.yaml
└── ProActions/
    └── sections/
        └── (your custom action files)
```

## Updating ProActions

To update to a newer version:

1. **Backup current installation:**
   
   ```bash
   mv pro-actions-{version}.js pro-actions-{version}.js_
   ```

2. **Replace the bundle:**
   
   ```bash
   cp /path/to/new-pro-actions-{version}.js pro-actions/pro-actions-{version}.js
   ```

3. **Review configuration changes:**
   
   - Check release notes for configuration changes :warning:
   - Update `pro-actions-config.js` if needed
   - Update action configurations if required

4. **Clear cache:**
   
   - In development: Refresh browser (F5)
   - In production: Clear Swing configuration cache and restart Swing.

5. **Test the update:**
   
   - Verify ProActions opens with `Ctrl+K`
   - Test existing workflows
   - Check browser console for errors

## Optional: Install ProActions Hub

For enhanced service integration and secure API key management, consider installing ProActions Hub:

**ProActions Hub provides:**

- Centralized API key management
- Support for 40+ AI providers
- Proxy configuration
- Content extraction tools
- EDAPI authentication

Consult your Eidosmedia technical contact or account manager for ProActions Hub installation instructions.

## Post-Installation

After successful installation:

1. **Configure services** - Set up API integrations (see [Configuration](./configuration.md))
2. **Create your first workflow** - Follow the [Getting Started](../getting-started/your-first-flow.md) guide
3. **Configure keyboard shortcuts** - Customize shortcuts for your team (see [Keyboard Shortcuts](./keyboard-shortcuts.md))

## Troubleshooting Installation

### ProActions doesn't load

**Symptoms:** No response when pressing `Ctrl+K`, no ProActions in console

**Solutions:**

- Verify `pro-actions-bundle.js` exists and is readable
- Check browser console (F12) for loading errors
- Clear browser cache completely
- Try incognito/private browsing mode

### Keyboard shortcut not working

**Symptoms:** `Ctrl+K` doesn't open ProActions

**Solutions:**

- Verify `/SysConfig/XsmileKbd.cfg` is configured correctly
- Check for conflicting keyboard shortcuts (also in browser extensions)
- Reload the editor context
- Try the fallback hotkey (configured in `pro-actions-config.js`)

### Configuration file errors

**Symptoms:** ProActions loads but actions don't appear

**Solutions:**

- Validate YAML syntax in `pro-actions.ai-kit.yaml`
- Check for typos in file paths
- Ensure files are saved with UTF-8 encoding
- Review browser console for configuration errors

### Version conflicts

**Symptoms:** Features not working as documented

**Solutions:**

- Verify you're using a compatible ProActions version
- Update all related configuration files
- Consult release notes for breaking changes

## Uninstallation

To remove ProActions:

1. **Remove extension files:**
   
   ```bash
   rm -rf ~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/pro-actions
   ```

2. **Remove keyboard shortcuts:**
   
   - Edit `/SysConfig/XsmileKbd.cfg`
   - Remove ProActions keyboard shortcut entries

3. **Remove configuration files (optional):**
   
   ```bash
   /SysConfig/pro-actions.ai-kit.yaml
   /SysConfig/ProActions/
   ```

4. **Clear browser cache** and reload

## Next Steps

- **[Configure ProActions](./configuration.md)** - Set up services and create workflows
- **[Configure Keyboard Shortcuts](./keyboard-shortcuts.md)** - Customize shortcuts
- **[Create Your First Flow](../getting-started/your-first-flow.md)** - Build your first automation
