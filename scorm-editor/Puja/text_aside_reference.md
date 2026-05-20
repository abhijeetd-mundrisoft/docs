# Articulate Rise 360 - Text Aside Block Reference

The **Text Aside** block is a layout variant that places a smaller piece of text (Heading) next to a larger main paragraph.

## JSON Structure
The block is defined as a `text` item with the `text aside` variant.

```json
{
  "type": "text",
  "variant": "text aside",
  "items": [
    {
      "heading": "<p>Aside Content</p>",
      "paragraph": "<p>Main Paragraph Content</p>"
    }
  ]
}
```

## Atomic Mapping in SCORM Editor
To ensure the layout remains intact, the SCORM Editor treats this as a single **Atomic Block**.

*   **Block Type**: `text aside`
*   **Field 1 (heading)**: Mapped to the "Aside" text.
*   **Field 2 (paragraph)**: Mapped to the "Main" text.

## How to Edit via API
When sending an update, the content should be a JSON object containing both fields:

```json
{
  "blockId": 5,
  "type": "text aside",
  "content": {
    "heading": "New Aside Text",
    "paragraph": "New Main Content"
  }
}
```
