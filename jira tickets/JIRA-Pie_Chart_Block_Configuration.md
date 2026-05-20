# JIRA Ticket: CC-517 - Pie Chart Block Configuration Settings

## Description

Extend the `Pie Chart` (DISTRIBUTION_CHAT) block functionality to support granular configurations for layout, heading, and data display. This includes section spacing, vertical spacing, title typography, and value display formatting (numeric vs percentage).

## Functional Changes Made

- **Settings Integration**: Added support for a nested `chart` settings object containing `layout`, `heading`, and `dataDisplay` configurations.
- **Background Support**: Added support for standard `background` settings (Light, Dark, Image, etc.) at the `settings` root level.
- **Validation Logic**: Updated `BlockContentValidator` to implement strict validation for:
    - `chart.layout.sectionSpacing`: Enum values (`narrow`, `regular`, `wide`).
    - `chart.layout.verticalSpacing`: Numeric range validation (0–150).
    - `chart.heading.titleStyle`: Enum values (`h1`, `h2`, `h3`, `h4`, `h5`).
    - `chart.dataDisplay.showValuesAs`: Enum values (`numeric`, `percentage`).
    - Standard `background` settings.
- **Default Factory Settings**: Updated `BlockContentFactory` to initialize `DISTRIBUTION_CHAT` blocks with:
    - Default chart configurations under `settings.chart`.
    - Default background style (`Light`) under `settings.background`.
- **Schema Documentation**: Updated `BlockSchemaUtil` to include the new `chart` and `background` settings in the AI generation schema.

## Sample JSON

```json
{
  "type": "DISTRIBUTION_CHAT",
  "content": {
    "chartTitle": "Market Share Distribution",
    "items": [
      { "title": "Product A", "value": 40, "orderIndex": 1 },
      { "title": "Product B", "value": 30, "orderIndex": 2 }
    ]
  },
  "settings": {
    "chart": {
      "layout": {
        "sectionSpacing": "narrow",
        "verticalSpacing": 25
      },
      "heading": {
        "titleStyle": "h2"
      },
      "dataDisplay": {
        "showValuesAs": "percentage"
      }
    },
    "background": {
      "style": "Light"
    }
  }
}
```

## Worklog Entries

- Implemented configurable `Layout`, `Heading`, `Data Display`, and `Background` settings for the Pie Chart (DISTRIBUTION_CHAT) block.
- Updated `validateChartSettings` in `BlockContentValidator` to include `dataDisplay.showValuesAs` validation.
- Updated `BlockContentFactory` to provide default chart and background settings for new DISTRIBUTION_CHAT blocks.
- Extended `BlockSchemaUtil` to document the new `chart` settings object and `background` support for Pie Charts.
- Updated `block_json_structures.md` with a complete Pie Chart configuration sample.
- Verified that `DISTRIBUTION_CHAT` validation remains robust and backward compatible.
