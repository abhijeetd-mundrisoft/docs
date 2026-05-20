# Implementation Plan - Add Multiple Choice Block Support

This plan details the steps to implement support for the "Multiple Choice" block type within the SCORM Editor (Rise 360). This will allow users to edit Knowledge Check questions and their respective options directly.

## Proposed Changes

### 1. SCORM Editor Model
**File**: `ContentBlock.java`
- Add a case for `"multipleChoice"` in the `getDisplayType()` switch statement to return the human-readable label **"Multiple Choice"**.

### 2. Rise 360 Parser
**File**: `RiseParser.java`
- Add constants for `FIELD_QUESTION`, `FIELD_ANSWERS`, `FIELD_IS_CORRECT`.
- Update `parseSlideContent` to detect `knowledgeCheck` items with the `multipleChoice` variant.
- Implement parsing logic to extract the question text and the list of answers (options) into a structured Map.
- Update `writeBackBlocks` to implement the mutation logic for `multipleChoice` blocks, allowing users to save edits to the question text and option titles.

### 3. Tests
**File**: `RiseParserTest.java`
- Add `parseSlideContent_multipleChoiceVariant_extractsQuestionAndOptionsCorrectly`.
- Add `writeBackBlocks_multipleChoiceVariant_updatesQuestionAndOptions`.

## Verification Plan

### Automated Tests
- Run `mvn test -Dtest=RiseParserTest` using JDK 17.

### Manual Verification
- Export a slide with a Knowledge Check and verify it appears as a "Multiple Choice" block in the editor.
- Edit the question and options, save, and verify the changes persist in the SCORM package.
