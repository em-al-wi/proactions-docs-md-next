# Prime Integration

This guide covers the integration of ProActions in the **Prime Windows Client**, including the Command Palette (Prime 8.2025.06+) and the ProActions Panel (requires ProActions Hub).

## Overview

ProActions can be integrated into Prime through two distinct approaches:

1. **Command Palette Integration** - Native Prime 8 command palette (version 8.2025.06+)
2. **ProActions Panel** - Web-based panel with full ProActions UI (requires ProActions Hub)

Both approaches require a properly configured ProActions configuration at `/SysConfig/pro-actions.ai-kit.yaml`. See [Configuration](../../setup/configuration.md) for setup instructions.

## Prerequisites

Before configuring Prime integration, ensure you have:

- **ProActions Configuration**: `/SysConfig/pro-actions.ai-kit.yaml` properly configured
- **Prime Client**: Prime 8 (for Command Palette) or any Prime version with macro support (for Panel)
- **File System Access**: Ability to edit `/SysConfig/` files
- **Prime Configuration Rights**: Permissions to modify Prime cfg files

For Prime Panel:

- **ProActions Hub**: Installed and accessible
- **Swing Web Application**: ProActions installed on Swing (see [Installation](../../setup/installation.md))
- **MRAS Connection** (Prime 7 only): Configured in `repositories.cfg` (see [Prime 7 MRAS Configuration](#step-3-configure-mras-connection-prime-7-only))

## Approach 1: Command Palette (Prime 8.2025.06+)

The Command Palette integration brings ProActions directly into Prime 8's native command palette interface, providing a seamless user experience.

### Minimum Version

**Required:** Prime 8 version `8.2025.06` or higher

### Features

- Native integration with Prime's command palette
- Keyboard shortcut activation
- Access to all configured ProActions workflows
- Consistent user experience with Prime interface
- No additional infrastructure required

### Configuration Steps

#### Step 1: Enable Command Palette

Edit the Prime preferences configuration file:

**File:** `/SysConfig/XsmilePrefs.cfg`

Add the `commandPalette` element with `enable="yes"`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<preferences>
  <!-- ... other preferences ... -->

  <!-- Enable Command Palette -->
  <commandPalette enable="yes" />

  <!-- ... other preferences ... -->
</preferences>
```

#### Step 2: Configure Keyboard Shortcut

Define the keyboard shortcut to open the Command Palette:

**File:** `/SysConfig/XsmileKbd.cfg`

Add the following configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<keyboard>
  <context type="Text">
    <!-- ... other keyboard shortcuts ... -->
    <key code="37" description="K" >
      <!-- Open Command Palette with Ctrl+Shift+P -->
      <modifier type="C" function="proActionsCommandPalette" shortcut="Ctrl+K" />
    </key>
    <!-- ... other keyboard shortcuts ... -->
  </context>

</keyboard>
```

**Note:** You can customize the keyboard shortcut. Common alternatives:

- `Ctrl+K` (consistent with Swing)
- `Ctrl+P` (VS Code style)

#### Step 3: Enable Actions in Configuration

To make an action from a panel using ProActions available in the Prime command palette (Prime 8 only), you must explicitly enable it in your action definition using the `prime` property:

```yaml
- title: "Generate Summary"
  hidden: true
  prime:
    commandPalette:
      enable: true
  flow:
    # ... workflow steps ...
```

Using this approach you can register actions from a panel that uses ProActions in the native Prime Command palette. Best practice is to set the action to `hidden: true` to avoid having the actions appear twice in the command palette, or use a separate configuration file for the panel's ProActions configuration.

#### Step 4: Verify Configuration

1. **Restart Prime** to load the new configuration
2. **Open any document** in Prime
3. **Press the configured shortcut** (e.g., `Ctrl+K`)
4. **Verify** the command palette opens with ProActions visible

### Approach 2: ProActions Panel (Requires ProActions Hub)

The ProActions Panel provides the full web-based ProActions interface within Prime, including all UI features like forms, menus, and advanced interactions.

### Prerequisites

- **ProActions Hub** installed and accessible
- **Swing Web Application** with ProActions installed
- **ProActions Hub URL** accessible from Prime client machines

### Features

- Full ProActions web interface
- All UI features: forms, modals, progress bars
- Text selection menus and one-click menus
- Advanced workflows with complex user interactions
- Shared configuration with Swing

### Configuration Steps

#### Step 1: Create ProActions Macro

Create a JavaScript macro to open the ProActions panel:

**File:** `/SysConfig/Macros/ProActions.js`

```javascript
var app = new ActiveXObject("methode.application");
app.ActiveWindow.OpenWebPanel(
  "https://<YOUR-SWING-HOST>/swing/proactions/primePanel/?mode=chrome&ISMActions=allow"
);
```

**Important:** Replace `<YOUR-SWING-HOST>` with your actual Swing server hostname.

**URL Parameters:**

- `mode=chrome` - Displays panel in Chrome
- `ISMActions=allow` - Allows Prime automation actions (required for ProActions in Prime)

#### Step 2: Configure Keyboard Shortcut

Add a keyboard shortcut to trigger the macro:

**File:** `/SysConfig/XsmileKbd.cfg`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<keyboard>
  <context type="Text">
  <!-- ... other keyboard shortcuts ... -->

  <key code="37" description="K">
    <!-- Open ProActions Panel with Ctrl+K -->
    <modifier type="C" shortcut="Ctrl+K">
      <function name="macro" d="/SysConfig/Macros/ProActions.js" />
    </modifier>
  </key>
  <!-- ... other keyboard shortcuts ... -->
  </context>
</keyboard>
```

#### Step 3: Configure MRAS Connection (Prime 7 Only)

If you're using **Prime 7**, you need to configure an MRAS connection in the Prime client's repository.cfg configuration.

**Note:** Prime 8 users can skip this step.

**File:** `repositories.cfg` (Prime client configuration)

Add an `<mras>` entry to your Methode domain configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<repositories>
  <methodeDomain name="YourMethodeDomain">
    <ns orbInit="-orbinitref YourMethodeDomain=corbaloc:iiop:methode-server.example.com:3900/NameService"
        name="YourMethodeDomain"/>
    <nc ncPath="/EOM/Notifiers/YourNotifier"
        ncName="YourNotifier" />

    <!-- MRAS connection for ProActions Panel (Prime 7) -->
    <mras uri="https://methode-server.example.com/swing/api"
          connectionId="Editorial"/>
  </methodeDomain>
</repositories>
```

**Key Configuration:**

- `uri` - The Swing API endpoint (replace with your Swing server URL)
- `connectionId` - Connection identifier (typically "Editorial")

**Important:**

- Only the `<mras>` entry is relevant for ProActions Panel functionality
- Replace `methode-server.example.com` with your actual server hostname

#### Step 4: Verify Panel Access

1. **Restart Prime** to load the new macro and keyboard shortcut (and MRAS configuration for Prime 7)
2. **Open any document** in Prime
3. **Press the configured shortcut** (e.g., `Ctrl+K`)
4. **Verify** the ProActions panel opens in a new window

### Panel Action Attributes

The ProActions Panel supports additional action attributes for organizing and controlling action visibility:

#### Category

Use the `category` attribute to create collapsible sub-categories within a section:

```yaml
actions:
  - title: "Replace Headline"
    category: "Headline & Summary"
    flow:
      # ... workflow steps ...

  - title: "Suggest Headlines"
    category: "Headline & Summary"
    flow:
      # ... workflow steps ...

  - title: "Rephrase Paragraph"
    category: "Rephrase"
    flow:
      # ... workflow steps ...
```

Actions with the same `category` value are grouped together in a collapsible section within the panel.

#### Hidden in Panel

Use the `hiddenInPanel` attribute to hide an action from the panel while keeping it available through other triggers:

```yaml
actions:
  - title: "Internal Helper Action"
    hiddenInPanel: true
    flow:
      # ... workflow steps ...
```

#### Panel Context

Use the `panelContext` attribute to specify when an action should appear in the "Context Actions" area at the top of the panel. This allows actions to be displayed only when specific conditions are met.

The `panelContext` is an array that can include:

**For Text Selection:**

```yaml
- title: "Process Selected Text"
  panelContext:
    - forTextSelection: true
  flow:
    # ... workflow steps ...
```

**For Specific Elements:**

```yaml
- title: "Process Image"
  panelContext:
    - forElement: "web-image"
  flow:
    # ... workflow steps ...
```

**For XPath Matches:**

```yaml
- title: "Replace Headline"
  panelContext:
    - forXPath: "doc/story/grouphead/headline/p"
  flow:
    # ... workflow steps ...

- title: "Replace Summary"
  panelContext:
    - forXPath: "doc/story/summary/p"
  flow:
    # ... workflow steps ...
```

**Multiple Context Conditions:**

```yaml
- title: "Context-Aware Action"
  panelContext:
    - forTextSelection: true
    - forXPath: "doc/story/text/p"
  flow:
    # ... workflow steps ...
```

Actions configured with `panelContext` will only appear in the "Context Actions" section when the cursor is in a matching context.

### Next Steps

- **[Configuration Basics](./basics.md)** - Learn configuration fundamentals
- **[Service Configuration](./service-configuration.md)** - Configure AI services
- **[Forms and Inputs](./forms-and-inputs.md)** - Create interactive forms (Panel)
- **[Menus and Triggers](./menus-and-triggers.md)** - Configure menus (Panel)
- **[Your First Flow](../../getting-started/your-first-flow.md)** - Build your first workflow

## Further Reading

- **[Installation Guide](../../setup/installation.md)** - Install ProActions on Swing
- **[Keyboard Shortcuts](../../setup/keyboard-shortcuts.md)** - Customize shortcuts
- **[Working with Variables](../../getting-started/working-with-variables.md)** - Use variables in workflows
- **[Step Library Overview](../../step-library/overview.md)** - Available workflow steps
