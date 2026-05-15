# JIRA-521: Heading and Subheading Spacing Configuration

## Description
Implement support for configurable heading and subheading spacing for text blocks. This ensures that authors can control the internal spacing between headings/subheadings and the body text within a block.

## Affected Block Types
- `HEADING_TEXT`
- `HEADING_ONLY`
- `TEXT_WITH_SUBHEADING`
- `SUBHEADING_ONLY`

## Changes Implemented

### 1. Backend Validation (`BlockContentValidator.java`)
- Added validation for `headingSpacing` and `subheadingSpacing` in `settings`.
- Range: `0-100`.
- Added `validateSubheadingSettings` method.
- Updated `validateHeadingSettings` to support spacing.
- Added 4 new unit tests covering valid/invalid spacing values.

### 2. Factory Defaults (`BlockContentFactory.java`)
- Injected default values for new spacing settings:
  - `headingSpacing`: `20` (Default for HEADING_TEXT, HEADING_ONLY).
  - `subheadingSpacing`: `15` (Default for TEXT_WITH_SUBHEADING, SUBHEADING_ONLY).
- Added `headingStyle` (H2) and `titleStyle` (H3) defaults.

### 3. Schema Documentation (`BlockSchemaUtil.java`)
- Added `heading` and `subheading` objects to `sharedSettings`.
- Updated schema descriptions for all affected text blocks to include these new settings.

### 4. Reference Documentation (`block_json_structures.md`)
- Updated JSON samples for `HEADING_TEXT`, `HEADING_ONLY`, `TEXT_WITH_SUBHEADING`, and `SUBHEADING_ONLY` to reflect the new settings structure.

## Verification
- Ran `BlockContentValidatorTest`: All 29 tests passed (including 4 new tests).
- Verified defaults injection logic in `BlockContentFactory`.
- Verified documentation accuracy.
