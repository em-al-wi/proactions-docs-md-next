# Image Outpainting with Stability AI

Expand images using AI outpainting to fill in space in any direction.

## Overview

This example demonstrates how to:

- Work with images in ProActions workflows
- Create custom component buttons to trigger ProActions
- Get information about selected image objects (SELECTED_OBJECT)
- Use EDAPI to download and update binary content
- Customize step inputs and outputs
- Integrate with Stability AI services

## Requirements

- Stability AI API access
- Image editing permissions in Méthode
- Custom component integration (optional)

## Use Case

Outpainting extends images beyond their original boundaries by using AI to generate additional content. This is useful for:

- Adjusting image dimensions for different layouts
- Converting portrait images to landscape (or vice versa)
- Creating header/banner versions of existing images
- Filling in cropped areas

## Configuration

### Action Definition

```yaml
# /SysConfig/ProActions/sections/stability-ai.ai-kit.section.yaml
section: StabilityAI

actions:
  - title: 'Outpaint with StabilityAI'
    id: "stabilityai_outpaint"
    isAvailable: "{{ client.getSelectedElement() === 'web-image' }}"
    flow:
      # Get selected image information
      - step: SELECTED_OBJECT
        outputs:
          - type: object
            name: selectedObject

      # Read high-res version of the image
      - step: EDAPI_OBJECT_CONTENT
        format: highres
        inputs:
          - type: text
            name: "selectedObject.id"

      # Send image to Stability AI's outpaint endpoint
      - step: STABILITY_AI_OUTPAINT
        left: 200
        right: 200
        up: 200
        down: 200
        inputs:
          - type: imageName
            name: "selectedObject.name"

      # Create a new version of the selected image
      - step: UPDATE_BINARY_CONTENT
        inputs:
          - type: text
            name: "selectedObject.id"

      # Show notification to user
      - step: SHOW_NOTIFICATION
        notificationType: success
        message: "Outpainting {{ flowContext.selectedObject.name }} successful."
        title: "StabilityAI"
```

### Main Configuration

```yaml
# pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES: !include /SysConfig/ProActions/services.ai-kit.services.yaml
  SECTIONS:
    - !include /SysConfig/ProActions/sections/stability-ai.ai-kit.section.yaml
```

### Custom Component Button (Optional)

Add a button to the image toolbar in Swing:

```javascript
// /methode/methdig/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/pro-actions-custom/image_toolbar.js

eidosmedia.webclient.commands.add({
  name: "custom.proactions.stability.outpaint",
  label: "Outpaint",
  isActive: function (ctx) {
    return true;
  },
  isEnabled: function (ctx) {
    if (ctx.component.getType() === ctx.activeDocument.CONSTANTS.BLOCK_IMAGE) {
      return true;
    }
    return false;
  },
  action: function (ctx, params, callbacks) {
    SwingProActions.executeAction(ctx, "stabilityai_outpaint");
  },
});

// Add button to image toolbar
eidosmedia.webclient.actions.editor.components.addButton({
  action: "custom.proactions.stability.outpaint",
  label: "Outpaint using AI",
  icon: "fas fa-expand-arrows-alt",
  allowReadOnly: false,
});
```

## How It Works

### Step 1: Conditional Availability

The `isAvailable` condition ensures the action only appears for images:

```yaml
isAvailable: "{{ client.getSelectedElement() === 'web-image' }}"
```

This expression:

- Returns `true` when an image is selected
- Returns `false` otherwise
- Hides the action when not applicable

### Step 2: Get Selected Object

The SELECTED_OBJECT step retrieves information about the selected image:

```yaml
- step: SELECTED_OBJECT
  outputs:
    - type: object
      name: selectedObject
```

This stores object metadata in `flowContext.selectedObject`:

```javascript
{
  id: "12345",
  name: "photo.jpg",
  type: "Image",
  // ... other metadata
}
```

### Step 3: Download Image Content

The EDAPI_OBJECT_CONTENT step downloads the image binary data:

```yaml
- step: EDAPI_OBJECT_CONTENT
  format: highres
  inputs:
    - type: text
      name: "selectedObject.id"
```

**Key points:**

- `format: highres` - Gets the high-resolution version
- `inputs` - Customizes which variable to use for object ID
- Uses `selectedObject.id` from the previous step

### Step 4: Process with Stability AI

The STABILITY_AI_OUTPAINT step extends the image:

```yaml
- step: STABILITY_AI_OUTPAINT
  left: 200    # Extend 200 pixels to the left
  right: 200   # Extend 200 pixels to the right
  up: 200      # Extend 200 pixels upward
  down: 200    # Extend 200 pixels downward
  inputs:
    - type: imageName
      name: "selectedObject.name"
```

**Parameters:**

- `left`, `right`, `up`, `down` - Pixels to extend in each direction
- `inputs` - Maps `selectedObject.name` to the expected input variable

### Step 5: Update Image in Méthode

The UPDATE_BINARY_CONTENT step saves the outpainted image:

```yaml
- step: UPDATE_BINARY_CONTENT
  inputs:
    - type: text
      name: "selectedObject.id"
```

This:

- Creates a new version of the image
- Preserves metadata
- Updates the binary content with the outpainted result

## Best Practices

1. **Check Availability**: Use `isAvailable` to show actions only when appropriate
2. **High-Res Images**: Always use `format: highres` for best quality
3. **Reasonable Sizes**: Don't extend too far (max ~400px per side for quality)
4. **User Feedback**: Show progress for long operations
5. **Error Handling**: Wrap in TRY blocks - image processing can fail
6. **Custom Inputs**: Use the `inputs` array to map variables correctly

## Troubleshooting

**Action doesn't appear:**

- Check `isAvailable` condition
- Verify an image is actually selected
- Check browser console for JavaScript errors

**Image download fails:**

- Verify EDAPI credentials are correct
- Check object ID is valid
- Ensure user has read permissions

**Outpainting fails:**

- Check Stability AI API key configuration
- Verify image format is supported (JPEG, PNG)
- Check image size isn't too large
- Review Stability AI service logs

**Update fails:**

- Verify user has write permissions
- Check object isn't locked by another user

**Custom button not showing:**

- Verify JavaScript file is loaded
- Check component type detection logic
- Ensure SwingProActions is available
- Check browser console for errors

## Related Examples

- [Content Import](./content-import.md) - Upload and process images
- [Progress Bar](./progress-bar.md) - Show progress during operations
