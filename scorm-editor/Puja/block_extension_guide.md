# Guide: Adding New Blocks to SCORM Editor (Rise 360)

This document provides a standard template and checklist for extending the SCORM Editor backend to support new Articulate Rise 360 block types.

---

## **1. Analysis Phase**
Before coding, identify the JSON structure of the new block in the `index.html` (inside the base64-encoded `window.courseData`).

- **Type**: (e.g., `text`, `interactive`, `knowledgeCheck`)
- **Variant**: (e.g., `process`, `sorting`, `statement`)
- **Item Structure**: Are there multiple items? (e.g., steps in a process, cards in a sorting activity).
- **Editable Fields**: Identify which fields the UI needs to edit (e.g., `title`, `description`, `caption`, `date`).

---

## **2. Implementation Steps**

### **Step A: Update Content Model**
File: `ContentBlock.java`
- Update the `getDisplayType()` method to map the internal block type string to a human-readable name for the UI.

### **Step B: Implement Parsing Logic**
File: `RiseParser.java`
- In `parseSlideContent()`, add a check for the new `type` or `variant`.
- For interactive blocks, register the new variant in the condition that calls `parseInteractiveElement()`.
- If the block structure is unique, update `parseInteractiveElement()` to correctly extract the fields into a `Map`.

### **Step C: Implement Write-Back Logic**
File: `RiseParser.java`
- In `writeBackBlocks()`, ensure the new block type is permitted to enter the write-back loop.
- Update `writeBackInteractiveElement()` to map the updated content from the `ContentBlock` back into the Jackson `JsonNode` structure.
- **Critical**: Ensure the `contentArrayIndex` logic is maintained to prevent "index drift" when multiple sub-items exist.

---

## **3. Verification Phase**

### **Step D: Create Unit Test**
File: `RiseParserTest.java`
- Create a round-trip test case that:
    1.  Mocks the `index.html` with a sample JSON containing the new block.
    2.  Parses the content to verify the fields are extracted correctly.
    3.  Modifies the content programmatically.
    4.  Writes the block back.
    5.  Parses again to verify the final JSON contains the updated values.

---

## **Checklist for Pull Request**
- [ ] Added display name in `ContentBlock.java`.
- [ ] Added variant support in `RiseParser.parseSlideContent`.
- [ ] Added variant support in `RiseParser.writeBackBlocks`.
- [ ] Handled JSON field mapping in `parseInteractiveElement` / `writeBackInteractiveElement`.
- [ ] Added 2-3 unit tests in `RiseParserTest.java`.
- [ ] Verified round-trip stability (Parse -> Edit -> Write -> Parse).
