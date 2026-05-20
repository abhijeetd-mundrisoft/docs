# JIRA Ticket: CC-514 - Attachment Block Configuration Support

## Description
Extend standard `RESOURCE_FILE` (Attachment) block functionality to support granular layout configurations including section spacing and vertical spacing.

## Functional Changes Made

* **Schema Evolution**: Updated `BlockSchemaUtil` to include `settings` field for the `RESOURCE_FILE` block type, enabling AI course generation to configure its layout dynamically and aligning documentation with system capabilities.
* **Validation Integration**: Leveraged existing `BlockContentValidator` logic to enforce strict validation for `RESOURCE_FILE` blocks. This ensures properties like `sectionSpacing` (enum: narrow, regular, wide) and `verticalSpacing` (range: 0-150) are correctly persisted and sanitized.
* **Factory Defaults**: Updated `BlockContentFactory` to provide standardized default settings (`sectionSpacing: narrow`, `verticalSpacing: 25`) when a new Attachment block is initialized, ensuring visual consistency across the platform.
* **Documentation**: Added comprehensive JSON structure sample for the `RESOURCE_FILE` block type in `docs/block-configuration/block_json_structures.md`.

## Sample JSON
```json
{
  "type": "RESOURCE_FILE",
  "content": {
    "fileId": "attachment-uuid-01",
    "displayName": "System_Requirements.pdf",
    "description": "Technical specifications for the application.",
    "allowDownload": true
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "background": {
      "style": "Light"
    }
  }
}
```

## Worklog Entries
* Added layout and background configuration support for RESOURCE_FILE (Attachment) blocks.
* Enabled sectionSpacing and verticalSpacing validation for attachment containers.
* Updated BlockContentFactory to initialize default layout settings for RESOURCE_FILE type.
* Extended AI schema in BlockSchemaUtil to support Attachment block settings.
* Documented Attachment block JSON structure in block_json_structures.md.
* Created JIRA ticket document CC-514 for tracking backend styling support.
