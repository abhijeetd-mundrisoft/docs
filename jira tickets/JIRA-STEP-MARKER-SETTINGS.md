# JIRA-STEP-MARKER-SETTINGS: Implement Configurable Layout and Numbering Settings for Step Marker

## Description
Extend Numbered Divider / Step Marker block functionality to support configurable layout spacing and numbering behavior including section spacing presets, vertical spacing controls, numbering restart toggles, and configurable restart numbering values.

## Functional Changes Made

### Validation Integration
Enabled layout and numbering-specific validation for Numbered Divider / Step Marker block settings in `BlockContentValidator`, ensuring properties like `sectionSpacing`, `verticalSpacing`, `restartSequence`, and `restartAt` are validated consistently.

### Conditional Validation
Added conditional validation handling to ensure `restartAt` is validated only when `restartSequence` is enabled.

### Default Handling
Updated block factory/default configuration handling to apply fallback values when layout or numbering configuration is absent or invalid.

### Schema Association
Defined Numbered Divider / Step Marker block schema mappings in `BlockSchemaUtil` to inherit shared reusable layout and numbering settings.

### Layout Integration
Integrated reusable spacing resolution utilities into renderer/layout engine for consistent block spacing application.

### Numbering Integration
Integrated reusable numbering sequence utilities to support configurable numbering restart behavior.

### Serialization Support
Updated serialization/deserialization handling for layout and numbering configuration persistence.

### Reusable Infrastructure
Extended centralized spacing utilities, numbering helpers, and validators to support reusable block configuration patterns.

---

## Sample JSON

```json id="v59oq5"
{
  "id": "step-marker-001",
  "type": "STEP_MARKER",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "numbering": {
      "restartSequence": true,
      "restartAt": 1
    }
  },
  "content": {}
}
```

---

## WorkLog Entries
* Added layout configuration support for NUMBERED DIVIDER / STEP MARKER blocks.
* Added sectionSpacing validation and default spacing handling.
* Added verticalSpacing range validation and normalization support.
* Added restartSequence toggle validation and numbering restart logic.
* Added restartAt conditional validation and sequencing support.
* Extended common layout utilities to support reusable spacing resolution.
* Extended numbering utilities to support restart sequence handling.
* Updated validation and factory logic for default numbering/layout handling.
* Added Numbered Divider / Step Marker support to AI schema configuration.
* Added unit tests for spacing and numbering validation scenarios.
* Verified valid and invalid Numbered Divider / Step Marker configuration handling.
* Added Numbered Divider / Step Marker JSON sample to `docs\block-configuration\block_json_structures.md`
* Created a new JIRA ticket document for tracking.
