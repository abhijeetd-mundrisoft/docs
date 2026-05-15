# [CC-519] Implement Video text right Block Configuration

## Description
Implement support for configurable layout, styling, playback, and positioning settings for the `VIDEO_TEXT_RIGHT` blockType.

## Functional Goal
Add support for:
- **Section Spacing**: `narrow`, `regular`, `wide`
- **Vertical Spacing**: `0–120` (default `25`)
- **Video Width**: `compact`, `regular`, `large`
- **Rounded Corners**: `0–30` (default `0`)
- **Allow Forward Seeking**: `true`/`false`
- **Video Position**: `left`, `right`

## Technical Tasks
- Update `BlockContentValidator` to include `videoPosition` and `roundedCorners` validation.
- Update `BlockContentFactory` to provide default settings for `VIDEO_TEXT_RIGHT`.
- Update `BlockSchemaUtil` to document `videoPosition` and `VIDEO_TEXT_RIGHT` settings.
- Update `block_json_structures.md` with `VIDEO_TEXT_RIGHT` JSON sample.
- Add unit tests for `VIDEO_TEXT_RIGHT` configuration and validation.

## Definition of Done
- [x] Backend validation logic updated.
- [x] Factory default initialization implemented.
- [x] AI Schema documentation updated.
- [x] JSON structure documentation updated.
- [ ] Unit tests passed.
- [ ] Backward compatibility verified.
