# Implementation Plan - Multiple Response Quiz Block Support

Support for the Articulate Rise 360 "Multiple Response" quiz block. This block allows users to select multiple correct answers.

## Proposed Changes

### [Component] SCORM Editor Backend

#### [MODIFY] [RiseParser.java](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/java/com/mundrisoft/courseforge/scorm/editor/parser/RiseParser.java)
- Update `parseSlideContent` to detect the `knowledgeCheck` type with the `multipleResponse` variant.
- Reuse the atomic mapping logic (Question + Answers array) used for `multipleChoice`.
- Update `writeBackBlocks` to handle persistence for the `multipleResponse` block type.

#### [MODIFY] [ContentBlock.java](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/java/com/mundrisoft/courseforge/scorm/editor/model/ContentBlock.java)
- Register the `multipleResponse` type in the display type mapping to show as "Multiple Response Quiz".

## JSON structure for Updates
The content for this block is an object:
```json
{
  "blockId": 15,
  "type": "multipleResponse",
  "content": {
    "question": "Which of these are fruits?",
    "options": [
      { "title": "Apple", "isCorrect": true },
      { "title": "Carrot", "isCorrect": false },
      { "title": "Banana", "isCorrect": true }
    ]
  }
}
```

## Verification Plan

### Automated Tests
- **RiseParserTest.java**:
    - Add `parseSlideContent_multipleResponseVariant_extractsMetadataCorrectly` to verify multiple correct answers are preserved.
    - Add `writeBackBlocks_multipleResponseVariant_updatesQuestionAndAnswers` to verify persistence.
- Run `mvn test -Dtest=RiseParserTest`.

### Manual Verification
- Verify the block appears with the label "Multiple Response Quiz" in the editor response.
- Verify that updating the question and answer states through the API correctly persists to the course JSON.
