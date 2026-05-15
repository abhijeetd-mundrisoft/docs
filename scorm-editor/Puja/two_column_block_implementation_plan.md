# Implementation Plan - Add "Two Column" Block Support

This plan details the steps to implement support for the "Two Column" block type within the SCORM Editor (Rise 360). This will map the Rise `variant: "two column"` to a single `ContentBlock` of type `"two column"`.

## Proposed Changes

### 1. SCORM Editor Model
**File**: `src/main/java/com/mundrisoft/courseforge/scorm/editor/model/ContentBlock.java`
- Add a case for `"two column"` in the `getDisplayType()` switch statement to return the human-readable label **"Two Column"**.

### 2. Rise 360 Parser
**File**: `src/main/java/com/mundrisoft/courseforge/scorm/editor/parser/RiseParser.java`

**Parsing Logic (`parseSlideContent`)**:
- Add a specific check for `normalizedVariant.equals("two column")` before or within the `"text"` type processing.
- If it is a `"two column"` block, extract the array of items. For each item, extract the `heading` and `paragraph` fields.
- Assemble this into a `List<Map<String, String>>` and emit a **single** `ContentBlock` with `type: "two column"`.
- Skip the generic `"text"` block processing (which would incorrectly emit 4 separate blocks: Heading, Body, Heading, Body) by using an `else` or `continue`.

**Write-Back Logic (`writeBackBlocks`)**:
- Add a specific check for `normalizedVariant.equals("two column")` in the `"text"` processing block.
- Read the modified `ContentBlock` (type `"two column"`).
- Cast the content back to `List<Map<String, Object>>` and iterate over the original JSON `items` array, updating the `heading` and `paragraph` strings for each column.
- Skip the generic `"text"` write-back logic for this variant.

### 3. Tests
**File**: `src/test/java/com/mundrisoft/courseforge/scorm/editor/parser/RiseParserTest.java`
- Add `parseSlideContent_twoColumnVariant_extractsMetadataCorrectly` to verify the dual-column structure is extracted into a single block.
- Add `writeBackBlocks_twoColumnVariant_updatesFields` to verify that edits to either column are correctly written back to the Rise JSON.

## Verification Plan

### Automated Tests
- Run `mvn test -Dtest=RiseParserTest` using JDK 17 to ensure the new tests pass and no existing block parsing logic is broken.

### Manual Verification
- Verify that a slide with a two-column layout correctly surfaces a single block with `type: "two column"` and two text sets in the API response.
