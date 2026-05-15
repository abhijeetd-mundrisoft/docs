# Implementation Plan - Add "Three Column" Block Support

This plan details the steps to implement support for the "Three Column" block type within the SCORM Editor (Rise 360). This will map the Rise `variant: "three column"` to a single `ContentBlock` of type `"three column"`.

## Proposed Changes

### 1. SCORM Editor Model
**File**: `src/main/java/com/mundrisoft/courseforge/scorm/editor/model/ContentBlock.java`
- Add a case for `"three column"` in the `getDisplayType()` switch statement to return the human-readable label **"Three Column"**.

### 2. Rise 360 Parser
**File**: `src/main/java/com/mundrisoft/courseforge/scorm/editor/parser/RiseParser.java`

**Parsing Logic (`parseSlideContent`)**:
- Extend the check for `"two column"` to also include `"three column"`.
- If it is a `"three column"` block, extract the array of items. For each item, extract the `heading` and `paragraph` fields.
- Assemble this into a `List<Map<String, String>>` and emit a **single** `ContentBlock` with `type: "three column"`.

**Write-Back Logic (`writeBackBlocks`)**:
- Extend the check for `"two column"` to also include `"three column"`.
- Read the modified `ContentBlock` (type `"three column"`).
- Cast the content back to `List<Map<String, Object>>` and iterate over the original JSON `items` array, updating the `heading` and `paragraph` strings for each column.

### 3. Tests
**File**: `src/test/java/com/mundrisoft/courseforge/scorm/editor/parser/RiseParserTest.java`
- Add `parseSlideContent_threeColumnVariant_extractsMetadataCorrectly` to verify the three-column structure is extracted into a single block.
- Add `writeBackBlocks_threeColumnVariant_updatesFields` to verify that edits are correctly written back.

## Verification Plan

### Automated Tests
- Run `mvn test -Dtest=RiseParserTest` using JDK 17 to ensure the new tests pass and no regressions occur.

### Manual Verification
- Verify that a slide with a three-column layout correctly surfaces a single block with `type: "three column"` and three text sets in the API response.
