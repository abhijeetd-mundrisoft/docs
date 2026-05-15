# [CC-520] Implement Video text Left Block Configuration

## Description
Implement support for configurable layout, styling, playback, and positioning settings for the `TEXT_VIDEO_LEFT` blockType.

## Functional Goal
Add support for:
- **Section Spacing**: `narrow`, `regular`, `wide`
- **Vertical Spacing**: `0–120` (default `25`)
- **Video Width**: `compact`, `regular`, `large`
- **Rounded Corners**: `0–30` (default `0`)
- **Allow Forward Seeking**: `true`/`false`
- **Video Position**: `left`, `right` (default `left`)

## Technical Tasks
- Update `BlockContentValidator` to include `TEXT_VIDEO_LEFT` content validation.
- Update `BlockContentFactory` to provide default settings for `TEXT_VIDEO_LEFT`.
- Update `BlockSchemaUtil` to document `TEXT_VIDEO_LEFT` settings.
- Update `block_json_structures.md` with `TEXT_VIDEO_LEFT` JSON sample.
- Add unit tests for `TEXT_VIDEO_LEFT` configuration and validation.

## Definition of Done
- [x] Backend validation logic updated.
- [x] Factory default initialization implemented.
- [x] AI Schema documentation updated.
- [x] JSON structure documentation updated.
- [ ] Unit tests passed.
- [ ] Backward compatibility verified.
