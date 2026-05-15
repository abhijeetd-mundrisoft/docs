# Checklist Block Implementation

The Checklist block is implemented as an atomic unit to allow editing multiple items within a single block interface.

## JSON Structure (Rise 360)
```json
{
  "type": "list",
  "variant": "checklist",
  "items": [
    { "paragraph": "Task 1" },
    { "paragraph": "Task 2" }
  ]
}
```

## Backend Mapping
- **Type**: `checklist`
- **Content**: `List<String>` (e.g., `["Task 1", "Task 2"]`)

## Display Name
"Checklist"
