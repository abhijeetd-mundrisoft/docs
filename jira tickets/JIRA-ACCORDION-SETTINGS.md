# JIRA-ACCORDION-SETTINGS: Implement Configurable Layout, Heading, Behavior, and Interaction Settings for Accordion

## Description
Extend Accordion block functionality to support configurable layout spacing, semantic heading behavior, accordion expansion interaction, and image zoom/lightbox interaction settings.

## Functional Changes Made

### Validation Integration
Enabled layout, heading, behavior, and interaction validation for Accordion block settings in `BlockContentValidator`, ensuring that properties like `sectionSpacing`, `verticalSpacing`, `headingStyle`, `expansionMode`, and `openImageOnClick` are validated consistently.

### Interaction Integration
Integrated reusable accordion interaction behavior utilities supporting both `single_open` and `multi_open` expansion modes.

### Media Interaction Support
Integrated reusable lightbox/image zoom interaction utilities for images rendered inside Accordion items.

### Default Handling
Updated block factory/default configuration handling to apply fallback values when layout, heading, or interaction configuration is absent or invalid.

### Schema Association
Defined Accordion block schema mappings in `BlockSchemaUtil` to inherit shared reusable layout and interaction settings.

### Layout Integration
Integrated reusable spacing utilities into renderer/layout engine for consistent block rendering behavior.

### Accessibility Integration
Ensured semantic heading rendering and keyboard-accessible accordion interaction behavior.

### Serialization Support
Updated serialization/deserialization handling for layout and interaction configuration persistence.

### Reusable Infrastructure
Extended centralized spacing utilities, heading helpers, interaction handlers, and validators to support reusable block configuration patterns.

---

## Sample JSON

```json
{
  "id": "accordion-001",
  "type": "ACCORDION",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "heading": {
      "headingStyle": "H2"
    },
    "behavior": {
      "expansionMode": "single_open"
    },
    "interaction": {
      "openImageOnClick": true
    }
  },
  "content": [
    {
      "title": "What is Artificial Intelligence?",
      "content": "Artificial Intelligence refers to systems capable of performing tasks that normally require human intelligence.",
      "orderIndex": 1
    },
    {
      "title": "What is Machine Learning?",
      "content": "Machine Learning is a subset of AI focused on learning patterns from data.",
      "orderIndex": 2
    }
  ]
}
```

---

## WorkLog Entries
* Added layout configuration support for ACCORDION blocks.
* Added sectionSpacing validation and reusable spacing handling support.
* Added verticalSpacing range validation and normalization handling.
* Added headingStyle validation and semantic heading rendering support.
* Added expansionMode configuration handling for accordion interaction behavior.
* Added image lightbox interaction support for accordion item images.
* Extended reusable interaction utilities for single-open and multi-open accordion behavior.
* Updated validation and factory logic for default interaction and layout handling.
* Added Accordion support to AI schema configuration.
* Added unit tests for accordion configuration validation and interaction scenarios.
* Verified valid and invalid Accordion configuration handling.
* Added Accordion JSON sample to `docs\block-configuration\block_json_structures.md`
* Created a new JIRA ticket document for tracking.
