# Checkbox List Block Implementation

The Checkbox List block is implemented as an atomic unit, sharing the same logic as the Checklist block.

## JSON Structure (Rise 360)
```json
{
  "type": "list",
  "variant": "checkbox",
  "items": [
    { "paragraph": "Item 1" },
    { "paragraph": "Item 2" }
  ]
}
```

## Backend Mapping
- **Type**: `checkbox` (Atomic)
- **Content**: `List<String>`
