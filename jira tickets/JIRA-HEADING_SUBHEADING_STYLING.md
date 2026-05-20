# JIRA Ticket: HEADING_ONLY & SUBHEADING_ONLY Styling Integration (CC-473)

## Summary
Implement backend configuration-driven styling support for `HEADING_ONLY` and `SUBHEADING_ONLY` block types. This includes extending the JSON `settings` schema to support standardized `layout` and `background` configurations, while ensuring full compatibility with existing content validation rules (including the optional subheading requirement from CC-419).

## Technical Scope
- **Validation Logic**: Update `BlockContentValidator.java` to handle `HEADING_ONLY` and `SUBHEADING_ONLY` block types.
- **Settings Schema**: Implement validation for the following groups within the `settings` JSON:
    - **Layout**: `contentArea` (enum), `sectionSpacing` (enum), `verticalSpacingTop/Bottom` (0-150), `borderRadius` (0-150), and `verticalSpacingLinked` (boolean).
    - **Background**: `style` (preset enum), `customColor` (hex + contrastMode), and `image` (fileId + crop metadata).
- **Content Flexibility**: Maintain support for HTML content in `heading` and `subheading` fields, ensuring `subheading` remains optional for `SUBHEADING_ONLY`.
- **Documentation**: Update `BlockSchemaUtil.java` to reflect the new styling settings for these specific block types to support AI generation.

## Acceptance Criteria
- [ ] `HEADING_ONLY` blocks must support mandatory `heading` content.
- [ ] `SUBHEADING_ONLY` blocks must support optional `subheading` content.
- [ ] The `settings` object must be persisted and validated against the styling schema.
- [ ] `layout.contentArea` must accept: `compact`, `regular`, `large`.
- [ ] `layout.sectionSpacing` must accept: `narrow`, `regular`, `wide`.
- [ ] `background.style` must accept the 8 standard presets (Light, Gray, Theme, Theme Tint, Dark, Black, Custom, Image).
- [ ] Numeric values for spacing and border radius must be constrained between `0` and `150`.
- [ ] If `background.style` is `Custom`, a valid `hex` code is required.
- [ ] If `background.style` is `Image`, a valid `fileId` (UUID) is required.

## Testing Scope
- **Validation Testing**: Verify that invalid enums or out-of-range numeric values (e.g., verticalSpacing = 200) trigger a `400 Bad Request`.
- **Conditional Validation**: Test that `Custom` style fails without `hex` and `Image` style fails without `fileId`.
- **Persistence Testing**: Ensure styling settings are correctly saved to the database and returned in API responses.
- **Backward Compatibility**: Verify that blocks without `settings` (null/empty) still load with default values.

## Dependencies
- **CC-419**: Infrastructure for optional subheading fields.
- **CC-473 Core**: Base implementation of styling enums and validation utilities.

## Future Extensibility Considerations
- **Typography Overrides**: Future support for per-block font family and font size overrides within the `settings` object.
- **Advanced Animations**: Hooking layout settings into frontend transition triggers.

## Sample JSON

### HEADING_ONLY
```json
{
  "type": "HEADING_ONLY",
  "content": {
    "heading": "<h1>Our Strategic Vision</h1>"
  },
  "settings": {
    "layout": {
      "contentArea": "large",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 40,
      "verticalSpacingBottom": 40,
      "verticalSpacingLinked": true,
      "borderRadius": 0
    },
    "background": {
      "style": "Theme"
    }
  }
}
```

### SUBHEADING_ONLY
```json
{
  "type": "SUBHEADING_ONLY",
  "content": {
    "subheading": "<h3>Exploring the Architecture</h3>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "regular",
      "verticalSpacingTop": 20,
      "verticalSpacingBottom": 20,
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
