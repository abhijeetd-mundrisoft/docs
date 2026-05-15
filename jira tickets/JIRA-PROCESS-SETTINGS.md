# JIRA-PROCESS-SETTINGS: Implement Configurable Layout, Heading, Label, and Interaction Settings for Process

## Description
Extend standard PROCESS (STEP_FLOW) block functionality to support configurable layout spacing, heading hierarchy, step label rendering, and interactive image behavior.

## Functional Changes Made

### Validation Integration
Enabled layout, heading, label, and interaction-specific validation for the PROCESS (STEP_FLOW) block type in `BlockContentValidator`, ensuring proper validation for:
* `sectionSpacing`
* `verticalSpacing`
* `headingStyle`
* `showStepLabel`
* `stepLabelText`
* `openImageOnClick`

### Factory Defaults
Updated `BlockContentFactory` to initialize default PROCESS settings for:
* section spacing (`narrow`)
* vertical spacing (`25`)
* heading style (`H2`)
* label visibility (`true`)
* label text (`Step`)
* image interaction behavior (`true`)

### Schema Association
Defined the STEP_FLOW block in `BlockSchemaUtil` to inherit reusable shared layout, heading, label, and interaction setting schemas, enabling AI generation and renderer configuration support dynamically.

---

## Sample JSON

```json
{
  "id": "process-block-001",
  "type": "STEP_FLOW",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "heading": {
      "headingStyle": "H2"
    },
    "label": {
      "showStepLabel": true,
      "stepLabelText": "Step"
    },
    "interaction": {
      "openImageOnClick": true
    }
  },
  "content": [
    {
      "type": "intro",
      "title": "Introduction",
      "description": "<p>Overview of the process</p>",
      "isHidden": false,
      "orderIndex": 1
    },
    {
      "type": "step",
      "title": "Prepare Ingredients",
      "description": "<p>Collect all required materials before starting.</p>",
      "isHidden": false,
      "orderIndex": 2,
      "image": {
        "url": "https://example.com/process-step-1.png",
        "alt": "Ingredients preparation"
      }
    }
  ]
}
```

---

## WorkLog Entries
* Added configurable layout support for PROCESS blocks.
* Added sectionSpacing and verticalSpacing validation handling.
* Implemented headingStyle configuration and semantic heading rendering support.
* Added conditional stepLabel rendering and custom label text configuration handling.
* Extended shared interaction utilities to support image zoom/lightbox configuration.
* Updated validation and factory logic for toggle and text-based settings.
* Added PROCESS block configuration support to AI schema and renderer integration.
* Added unit tests for Process configuration validation scenarios.
* Verified valid and invalid Process configuration handling.
* Added PROCESS JSON sample to `docs\block-configuration\block_json_structures.md`
* Created a new JIRA ticket document for tracking.
