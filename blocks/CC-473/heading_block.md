# Heading Block Configuration (CC-473)

This document defines the backend `settings` JSON schema and validation rules for `HEADING_TEXT` and `HEADING_ONLY` block types.

## Settings Schema

Heading blocks support common `layout` and `background` styling configurations.

### 1. Layout Settings

| Property | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `layout.contentArea` | String | `compact`, `regular`, `large` | `regular` | Controls the maximum content width. |
| `layout.sectionSpacing` | String | `narrow`, `regular`, `wide` | `narrow` | Preset spacing values. |
| `layout.verticalSpacing` | Integer | `0-150` | `25` | Uniform padding (base value). |
| `layout.verticalSpacingTop` | Integer | `0-150` | `25` | Padding at the top of the block. |
| `layout.verticalSpacingBottom` | Integer | `0-150` | `25` | Padding at the bottom of the block. |
| `layout.verticalSpacingLinked` | Boolean | `true`, `false` | `true` | Whether top and bottom spacing are linked in UI. |
| `layout.borderRadius` | Integer | `0-150` | `0` | Corner rounding of the block container. |

### 2. Background Settings

| Property | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `background.style` | String | `Light`, `Gray`, `Theme`, `Theme Tint`, `Dark`, `Black`, `Custom`, `Image` | `Light` | Predefined background style. |
| `background.customColor.hex` | String | HEX code (e.g., `#ffffff`) | - | Required if style is `Custom`. |
| `background.customColor.contrastMode` | String | `Auto`, `Light`, `Dark` | `Auto` | Contrast handling for text/graphics. |
| `background.image.fileId` | String | UUID | - | Required if style is `Image`. |
| `background.image.crop` | Object | `{x, y, width, height}` | - | Crop metadata for the image. |

## Sample JSON

```json
{
  "type": "HEADING_TEXT",
  "content": {
    "heading": "<h2>Section Title</h2>",
    "text": "<p>Supporting content description.</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true,
      "borderRadius": 8
    },
    "background": {
      "style": "Custom",
      "customColor": {
        "hex": "#f0f4f8",
        "contrastMode": "Auto"
      }
    }
  }
}
```

## Validation Rules

1. **Style Enums**:
   - `layout.contentArea` must be one of `['compact', 'regular', 'large']`.
   - `layout.sectionSpacing` must be one of `['narrow', 'regular', 'wide']`.
   - `background.style` must be one of the 8 predefined types.
   - `background.customColor.contrastMode` must be one of `['Auto', 'Light', 'Dark']`.

2. **Numeric Ranges**:
   - All spacing and radius fields must be between `0` and `150`.
   - `crop` metadata values must be positive integers.

3. **Conditional Requirements**:
   - If `background.style` is `Custom`, `background.customColor.hex` is required.
   - If `background.style` is `Image`, `background.image.fileId` is required.
