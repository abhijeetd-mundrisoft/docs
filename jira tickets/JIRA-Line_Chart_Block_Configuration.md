# JIRA Ticket: CC-516 - Line Chart Block Configuration Settings

## Description

Extend the `Line Chart` (TREND_CHAT) block functionality to support granular configurations for layout, heading, and styling. This includes section spacing, vertical spacing, title typography, line color, and line rendering style.

## Functional Changes Made

- **Settings Integration**: Added support for a nested `chart` settings object containing `layout`, `heading`, and `style` configurations.
- **Background Support**: Added support for standard `background` settings (Light, Dark, Image, etc.) at the `settings` root level.
- **Validation Logic**: Implemented strict validation in `BlockContentValidator` for:
    - `chart.layout.sectionSpacing`: Enum values (`narrow`, `regular`, `wide`).
    - `chart.layout.verticalSpacing`: Numeric range validation (0–150).
    - `chart.heading.titleStyle`: Enum values (`h1`, `h2`, `h3`, `h4`, `h5`).
    - `chart.style.chartColor`: HEX color validation.
    - `chart.style.lineStyle`: Enum values (`straight`, `curved`).
    - Standard `background` settings.
- **Default Factory Settings**: Updated `BlockContentFactory` to initialize `TREND_CHAT` blocks with:
    - Default chart configurations under `settings.chart`.
    - Default background style (`Light`) under `settings.background`.
- **Schema Documentation**: Updated `BlockSchemaUtil` to include the new `chart` and `background` settings in the AI generation schema.

## Sample JSON

```json
{
  "type": "TREND_CHAT",
  "content": {
    "chartTitle": "Monthly Revenue Trends",
    "xAxisLabel": "Month",
    "yAxisLabel": "Revenue (USD)",
    "items": [
      { "title": "Jan", "value": 4500, "orderIndex": 1 },
      { "title": "Feb", "value": 5200, "orderIndex": 2 }
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
      "style": {
        "chartColor": "#FF631E",
        "lineStyle": "curved"
      }
    },
    "background": {
      "style": "Light"
    }
  }
}
```

## Worklog Entries

- Implemented configurable `Layout`, `Heading`, `Style`, and `Background` settings for the Line Chart (TREND_CHAT) block.
- Added `validateChartSettings` to `BlockContentValidator` for enum, numeric range, and HEX color validation.
- Updated `BlockContentFactory` to provide default chart and background settings for new TREND_CHAT blocks.
- Extended `BlockSchemaUtil` to document the new `chart` settings object and `background` support.
- Updated `block_json_structures.md` with a complete Line Chart configuration sample.
- Verified that `TREND_CHAT` validation remains robust and backward compatible.
