# SCORM Editor - Column Block JSON Structures

This document provides the JSON structures for the **Two Column** and **Three Column** blocks as they are emitted by the backend API and as they should be sent back during a save/update operation.

## 1. Two Column Block

### Backend Response (GET)
When fetching slide content, a "two column" block will have the following structure:

```json
{
  "blockId": 4,
  "type": "two column",
  "content": [
    {
      "heading": "Left Column Heading",
      "paragraph": "This is the text content for the left column."
    },
    {
      "heading": "Right Column Heading",
      "paragraph": "This is the text content for the right column."
    }
  ]
}
```

---

## 2. Three Column Block

### Backend Response (GET)
A "three column" block follows the same pattern but with three items in the list:

```json
{
  "blockId": 5,
  "type": "three column",
  "content": [
    {
      "heading": "Column 1 Heading",
      "paragraph": "Text for column 1."
    },
    {
      "heading": "Column 2 Heading",
      "paragraph": "Text for column 2."
    },
    {
      "heading": "Column 3 Heading",
      "paragraph": "Text for column 3."
    }
  ]
}
```

---

## 3. Continue Block (Divider Button)

### Backend Response (GET)
The "continue" block extracts the button text into a single string.

```json
{
  "blockId": 6,
  "type": "continue",
  "content": "Continue"
}
```

### Save Payload (POST/PUT)
```json
{
  "blockId": 6,
  "type": "continue",
  "content": "Click to Proceed"
}
```

---

## 4. Statement / Impact Text (Variants a, b, c, d)

### Backend Response (GET)
These blocks are grouped by their variant name.

```json
{
  "blockId": 7,
  "type": "a",
  "content": "This is a statement."
}
```

### Save Payload (POST/PUT)
```json
{
  "blockId": 7,
  "type": "a",
  "content": "Updated statement text."
}
```

---

---

## 5. Paragraph with Subheading

### Backend Response (GET)
This block combines heading and body text into a single content object.

```json
{
  "blockId": 8,
  "type": "heading paragraph",
  "content": {
    "heading": "The Subheading",
    "paragraph": "The body text paragraph..."
  }
}
```

### Save Payload (POST/PUT)
```json
{
  "blockId": 8,
  "type": "heading paragraph",
  "content": {
    "heading": "Updated Heading",
    "paragraph": "Updated paragraph text."
  }
}
```

---

## Technical Notes for UI
- **Heading/Paragraph**: Used in multi-column blocks.
- **Content (String)**: Used for single-field blocks like "continue".
- **Atomic Updates**: Always send the full list or string in the `content` field. The backend updates the original `course.json` fields appropriately.
