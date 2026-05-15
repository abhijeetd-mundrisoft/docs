# JIRA Ticket: CC-473 - Implement configuration-driven formatting and styling support for TABLE block

## Summary
Implement configurable formatting and styling support for the `TABLE` block type using a reusable/common backend architecture. This includes support for content area sizing, section spacing, vertical spacing, cell spacing, and common background styles (including custom colors and images).

## Technical Scope
- **Validation**: Update `BlockContentValidator` to support new table-specific settings (`cellSpacing`) and integrate with existing common layout/background validation.
- **Factory Defaults**: Update `BlockContentFactory` to provide default styling settings for new `TABLE` blocks.
- **Schema Documentation**: Update `BlockSchemaUtil` to reflect the new supported settings for AI-driven generation.
- **DTOs/Mappers**: Ensure the existing flexible `settings` (JSON) architecture correctly handles the new fields.
- **Unit Testing**: Implement comprehensive tests for valid/invalid styling configurations.

## Acceptance Criteria
- [ ] `TABLE` block supports `contentArea` (Compact/Regular/Large).
- [ ] `TABLE` block supports `sectionSpacing` (Narrow/Regular/Wide).
- [ ] `TABLE` block supports linked `verticalSpacing` (0-150).
- [ ] `TABLE` block supports `cellSpacing` (Narrow/Regular/Wide) under a "Table Layout" group.
- [ ] `TABLE` block supports common background styles: Light, Gray, Theme, Theme Tint, Dark, Black, Custom, and Image.
- [ ] Custom background color persists HEX and contrast mode (Auto/Light/Dark).
- [ ] Image background persists fileId and crop metadata.
- [ ] Validation fails for invalid enum values or out-of-range numeric values.
- [ ] Persistence uses the existing `settings` JSON column in the `blocks` table.

## Testing Scope
- **Unit Tests**: `BlockContentValidatorTest` to verify all new settings and validation rules.
- **Payload Validation**: Test with example JSON payloads for various styling combinations.

## Dependencies
- Existing `Block` entity and `settings` JSON column.
- Common formatting architecture in `BlockContentValidator`.

## Future Extensibility
- Independent top/bottom vertical spacing.
- Custom padding values and directional spacing.
- Responsive spacing behavior.

## Sample JSON
```json
{
  "type": "TABLE",
  "content": {
    "rows": 2,
    "columns": 2,
    "data": [
      [
        {"content": "Header 1", "hAlign": "center", "vAlign": "middle"},
        {"content": "Header 2", "hAlign": "center", "vAlign": "middle"}
      ],
      [
        {"content": "Cell 1", "hAlign": "left", "vAlign": "top"},
        {"content": "Cell 2", "hAlign": "left", "vAlign": "top"}
      ]
    ],
    "config": {
      "hasHeaderRow": true,
      "hasHeaderColumn": false,
      "bordered": true
    }
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "tableLayout": {
      "cellSpacing": "narrow"
    },
    "background": {
      "style": "Custom",
      "customColor": {
        "hex": "#f0f0f0",
        "contrastMode": "Auto"
      }
    }
  }
}
```
