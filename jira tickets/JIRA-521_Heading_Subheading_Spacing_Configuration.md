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
- Consolidated all heading and subheading internal spacing under the `headingSpacing` key within the `layout` object.
- Removed the `subheadingSpacing` key to maintain a unified configuration schema.
- Range: `0-100`.
- Added/Updated unit tests covering valid/invalid `headingSpacing` values in `layout`.

### 2. Factory Defaults (`BlockContentFactory.java`)
- Injected default `headingSpacing` values into the `layout` map for all affected blocks:
  - `HEADING_TEXT`, `HEADING_ONLY`: `20`.
  - `TEXT_WITH_SUBHEADING`, `SUBHEADING_ONLY`: `15`.
- Reused the existing `layout` variable in `createSettings` to avoid compilation errors due to variable redefinition.
- Removed separate `heading` and `subheading` object initialization.

### 3. Schema Documentation (`BlockSchemaUtil.java`)
- Added `heading` and `subheading` objects to `sharedSettings`.
- Updated schema descriptions for all affected text blocks to include these new settings.

### 4. Reference Documentation (`block_json_structures.md`)
- Updated JSON samples for `HEADING_TEXT`, `HEADING_ONLY`, `TEXT_WITH_SUBHEADING`, and `SUBHEADING_ONLY` to reflect the new settings structure.

## Verification
- Ran `BlockContentValidatorTest`: All 29 tests passed (including 4 new tests).
- Verified defaults injection logic in `BlockContentFactory`.
- Verified documentation accuracy.
