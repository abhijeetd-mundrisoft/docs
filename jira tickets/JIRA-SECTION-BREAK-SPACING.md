# JIRA-SECTION-BREAK-SPACING: Implement Configurable Layout Settings for Section Break

## Description
Currently, the `SECTION_BREAK` (Divider) block type does not support configurable layout settings such as section spacing and vertical margins. This task involves updating the backend to support these settings, providing defaults, and validating the inputs.

## Requirements
- Add `sectionSpacing` (enum: narrow, regular, wide) support for `SECTION_BREAK`.
- Add `verticalSpacing` (range: 0-150) support for `SECTION_BREAK`.
- Ensure top and bottom spacing are linked for `SECTION_BREAK`.
- Update the block schema to reflect these changes.
- Ensure backward compatibility for existing blocks without settings.

## Proposed Changes
- **BlockSchemaUtil**: Update the canonical JSON schema to include `settings` for `SECTION_BREAK`.
- **BlockContentFactory**: Add `SECTION_BREAK` to the layout defaults injection logic.
- **BlockContentValidator**: Implement validation for `verticalSpacing` and ensure `SECTION_BREAK` is validated.

## Acceptance Criteria
- `SECTION_BREAK` blocks created via the API should have default settings if none are provided.
- Custom settings passed in the `settings` field should be validated and persisted.
- Invalid values for `sectionSpacing` or `verticalSpacing` should return a 400 Bad Request error.
- The `verticalSpacing` value should be correctly mapped to both top and bottom margins when provided.
