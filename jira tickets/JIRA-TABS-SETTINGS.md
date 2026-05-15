# JIRA-TABS-SETTINGS: Implement Configurable Layout and Interaction Settings for Tabs

## Description
Extend standard TABS block functionality to support configurable layout spacing and interactive image behavior including zoom/lightbox enablement.

## Functional Changes Made

### Validation Integration
Enabled layout-specific and interaction-specific validation for the TABS block type in `BlockContentValidator`, ensuring that properties like `sectionSpacing`, `verticalSpacing`, and `openImageOnClick` are validated and normalized correctly.

### Interaction Behavior Integration
Extended shared interactive media handling to conditionally enable image zoom/lightbox functionality inside tab panels based on configuration values.

### Factory Defaults
Updated `BlockContentFactory` to initialize default Tabs configuration values for spacing and media interaction behavior when new TABS blocks are created.

### Schema Association
Defined the TABS block in `BlockSchemaUtil` to inherit reusable shared layout and interaction settings, enabling AI generation and renderer configuration support dynamically.

---

## Sample JSON

```json
{
  "id": "tabs-001",
  "type": "TAB",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "interaction": {
      "openImageOnClick": true
    }
  },
  "content": [
    {
      "title": "Overview",
      "content": "<p>Tab content here</p>",
      "orderIndex": 1,
      "images": [
        {
          "url": "https://example.com/image1.png",
          "alt": "Overview image"
        }
      ]
    }
  ]
}
```

---

## WorkLog Entries
* Added configurable layout support for TABS blocks.
* Added sectionSpacing validation and default spacing handling support.
* Extended common layout utility handling for verticalSpacing configuration.
* Added reusable image interaction configuration support for interactive blocks.
* Updated validation and renderer logic for media interaction behavior handling.
* Added openImageOnClick support to AI schema and documented interaction configuration.
* Added unit tests for Tabs configuration validation scenarios.
* Verified valid and invalid interaction configuration handling.
* Added TABS JSON sample to `docs\block-configuration\block_json_structures.md`
* Created a new JIRA ticket document for tracking.
