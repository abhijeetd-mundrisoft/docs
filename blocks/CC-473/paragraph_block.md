# Paragraph Block Configuration (CC-473)

This document outlines the backend configuration and validation rules for the `TEXT` block type (Paragraph).

## Settings Schema

Paragraph blocks utilize common `layout` and `background` styling groups.

### 1. Layout Settings

| Property | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `layout.contentArea` | String | `compact`, `regular`, `large` | `regular` | Controls the maximum content width. |
| `layout.sectionSpacing` | String | `narrow`, `regular`, `wide` | `narrow` | Preset spacing values. |
| `layout.verticalSpacing` | Integer | `0-150` | `25` | Base vertical spacing value. |
| `layout.verticalSpacingTop` | Integer | `0-150` | `25` | Padding at the top of the block. |
| `layout.verticalSpacingBottom` | Integer | `0-150` | `25` | Padding at the bottom of the block. |
| `layout.verticalSpacingLinked` | Boolean | `true`, `false` | `true` | UI flag for linked spacing. |
| `layout.borderRadius` | Integer | `0-150` | `0` | Corner rounding of the block. |

### 2. Background Settings

| Property | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `background.style` | String | `Light`, `Gray`, `Theme`, `Theme Tint`, `Dark`, `Black`, `Custom`, `Image` | `Light` | Predefined background style. |
| `background.customColor.hex` | String | HEX code | - | Required for `Custom` style. |
| `background.customColor.contrastMode` | String | `Auto`, `Light`, `Dark` | `Auto` | Accessibility contrast handling. |
| `background.image.fileId` | String | UUID | - | Required for `Image` style. |
| `background.image.crop` | Object | `{x, y, w, h}` | - | Image crop coordinates. |

## Sample Request (JSON)

```json
{
  "type": "TEXT",
  "content": {
    "text": "<p>Sample paragraph content.</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "regular",
      "verticalSpacingTop": 30,
      "verticalSpacingBottom": 30,
      "verticalSpacingLinked": true
    },
    "background": {
      "style": "Image",
      "image": {
        "fileId": "bg-image-01",
        "altText": "Forest background",
        "crop": { "x": 0, "y": 10, "width": 100, "height": 80 }
      }
    },
    "typography": {
      "fontSize": "16px",
      "textAlign": "left"
    }
  }
}
```

## Validation Rules

1. **Grouped Structure**: Settings must be placed within their respective `layout` or `background` objects.
2. **Conditional Validation**:
   - `Custom` style requires `customColor.hex`.
   - `Image` style requires `background.image.fileId`.
3. **Enum Integrity**: All string-based preset values are strictly validated against allowed enums.
4. **Range Enforcement**: All numeric inputs (spacing, radius, crop) are constrained to valid ranges.
