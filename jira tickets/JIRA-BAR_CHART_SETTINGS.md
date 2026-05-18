# JIRA Ticket: CC-518 - Bar Chart Block Configuration Settings

## Description

Extend the `Bar Chart` (`BAR_CHART`) block type to support customizable layout, styling, heading, spacing, padding, and data display settings. These new configuration parameters should be grouped under a nested `chart` settings object alongside standard block-level `background` styling to align with established line and pie chart configuration models.

## Functional Changes Made

- **Settings Integration**: Added support for customizable settings nested under the `"chart"` and `"background"` objects within the block's `settings` property.
- **Factory Default Settings**: Updated `BlockContentFactory` to initialize newly created `BAR_CHART` blocks with standard default settings:
  - `chart.layout.contentArea` = `"regular"` (Options: `compact`, `regular`, `large`)
  - `chart.layout.sectionSpacing` = `"narrow"` (Options: `narrow`, `regular`, `wide`)
  - `chart.layout.verticalSpacing` = `25` (Range: `0-120`)
  - `chart.layout.blockPadding` = `25` (Range: `0-120`)
  - `chart.heading.headingLevel` = `"H2"` (Options: `H1`, `H2`, `H3`, `H4`, `H5`)
  - `chart.heading.titleStyle` = `"default"` (Options: `default`, `minimal`, `bold`, `accent`)
  - `chart.style.chartColor` = `"#FF631E"` (Valid HEX format)
  - `chart.dataDisplay.showValuesAs` = `"numeric"` (Options: `numeric`, `percentage`)
  - `background.style` = `"Light"` (Options: `Light`, `Gray`, `Theme`, `Dark`, `Image`, `Custom`)
- **Strict Backend Validation**: Extended the centralized `validateChartSettings` validator within `BlockContentValidator.java` to perform:
  - Numeric boundary range validation (e.g., vertical spacing and block padding bounds `[0, 120]`).
  - Enum set membership checks for layout sizes, heading levels, visual title presets, and data formatting types.
  - HEX value correctness validation for block colors.
- **Schema Documentation**: Expanded `BLOCK_SCHEMA_JSON` inside `BlockSchemaUtil.java` to define and document the new configurable parameters.
- **JSON Sample Documentation**: Documented the full `BAR_CHART` configuration sample under the Chart Blocks section of `docs/block-configuration/block_json_structures.md`.

## Sample JSON

```json
{
  "type": "BAR_CHART",
  "content": {
    "chartTitle": "Quarterly Sales Visualizer",
    "xAxisLabel": "Quarter",
    "yAxisLabel": "Sales (USD)",
    "items": [
      { "title": "Q1", "value": 120.5, "orderIndex": 1 },
      { "title": "Q2", "value": 145.2, "orderIndex": 2 }
    ]
  },
  "settings": {
    "chart": {
      "layout": {
        "contentArea": "regular",
        "sectionSpacing": "narrow",
        "verticalSpacing": 25,
        "blockPadding": 25
      },
      "heading": {
        "headingLevel": "H2",
        "titleStyle": "default"
      },
      "style": {
        "chartColor": "#FF631E"
      },
      "dataDisplay": {
        "showValuesAs": "numeric"
      }
    },
    "background": {
      "style": "Light"
    }
  }
}
```

## Testing & Verification Checklist

- [x] Verify block creation generates the appropriate default settings.
- [x] Verify valid settings payloads pass validation successfully.
- [x] Verify invalid spacing/padding bounds (e.g., negative or > 120 values) throw appropriate `IllegalArgumentException` validation errors.
- [x] Verify incorrect enums for content size, heading levels, title presets, or data formatting modes trigger proper validation failures.
- [x] Verify malformed hex codes for `chartColor` are rejected cleanly.
- [x] Verify backward compatibility with empty settings `"{}"` or missing attributes is fully preserved.
