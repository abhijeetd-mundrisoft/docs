# Subheading Block Configurations (CC-473)

This document defines the configuration and validation rules for `SUBHEADING_ONLY` and `TEXT_WITH_SUBHEADING` block types.

## Feature Overview
- **Optional Fields**: The `subheading` field is now **optional** for both block types.
- **Styling Integration**: Both blocks support the standardized `layout` and `background` styling groups.

## 1. SUBHEADING_ONLY
A dedicated block for section sub-headers.

### Content Structure
| Property | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| `subheading` | String | **No** | The subheading text (HTML supported). |

### Sample JSON
```json
{
  "type": "SUBHEADING_ONLY",
  "content": {
    "subheading": "<h3>Chapter 1: Getting Started</h3>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 20,
      "verticalSpacingBottom": 20,
      "verticalSpacingLinked": true
    },
    "background": {
      "style": "Theme Tint"
    }
  }
}
```

## 2. TEXT_WITH_SUBHEADING
A block combining a subheading with a paragraph of text.

### Content Structure
| Property | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| `subheading` | String | **No** | The optional subheading text. |
| `text` | String | **Yes** | The main paragraph text (HTML supported). |

### Sample JSON
```json
{
  "type": "TEXT_WITH_SUBHEADING",
  "content": {
    "subheading": "Key Takeaways",
    "text": "<ul><li>Item 1</li><li>Item 2</li></ul>"
  },
  "settings": {
    "layout": {
      "contentArea": "large",
      "sectionSpacing": "wide",
      "verticalSpacingTop": 40,
      "verticalSpacingBottom": 40,
      "borderRadius": 12
    },
    "background": {
      "style": "Gray"
    }
  }
}
```

## Styling Schema (Shared)
Both blocks utilize the standard `layout` and `background` keys:
- **Layout**: `contentArea`, `sectionSpacing`, `verticalSpacingTop/Bottom`, `borderRadius`.
- **Background**: `style`, `customColor` (hex, contrastMode), `image` (fileId, altText, crop).
