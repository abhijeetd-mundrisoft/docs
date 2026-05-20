# JIRA-MATCHING-SETTINGS: Implement Configurable Layout and Feedback Style Settings for Matching

## Description
Extend Matching block functionality to support configurable layout spacing, content width behavior, and feedback-state styling including correct and incorrect answer color customization. Introduced `MATCHING` as a top-level block type.

## Functional Changes Made

### Validation Integration
Enabled layout and feedback-style validation for Matching block settings in `BlockContentValidator`, ensuring that properties like `contentArea`, `sectionSpacing`, `verticalSpacing`, `correctAnswerColor`, and `incorrectAnswerColor` are validated consistently.

### Styling Integration
Integrated reusable feedback-state styling utilities to support configurable correct and incorrect answer color rendering.

### Default Handling
Updated block factory/default configuration handling to apply fallback values when layout or feedback-style configuration is absent or invalid.

### Schema Association
Defined Matching block schema mappings in `BlockSchemaUtil` to inherit shared reusable layout and styling settings.

### Layout Integration
Integrated reusable spacing and content-width resolution utilities into renderer/layout engine for consistent block rendering behavior.

### Serialization Support
Updated serialization/deserialization handling for layout and feedback-style configuration persistence.

### Reusable Infrastructure
Extended centralized spacing utilities, color helpers, and validators to support reusable block configuration patterns.

---

## Sample JSON

```json id="m2t3ch"
{
  "id": "matching-001",
  "type": "MATCHING",
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "feedbackStyle": {
      "correctAnswerColor": "#FF631E",
      "incorrectAnswerColor": "#000000"
    }
  },
  "content": {
    "question": "Match the countries with their capitals.",
    "pairs": [
      {
        "left": "France",
        "right": "Paris"
      },
      {
        "left": "Japan",
        "right": "Tokyo"
      }
    ]
  }
}
```

---

## WorkLog Entries
* Added `MATCHING` to `BlockType` enum as a top-level block.
* Added layout configuration support for MATCHING blocks.
* Added contentArea validation and responsive container width handling.
* Added sectionSpacing validation and default spacing handling.
* Added verticalSpacing range validation and normalization support.
* Added correctAnswerColor and incorrectAnswerColor validation support.
* Extended common styling utilities to support reusable feedback-state color handling.
* Updated validation and factory logic for default layout and feedback-style handling.
* Added MATCHING support to AI schema configuration.
* Added unit tests for layout and feedback-style validation scenarios.
* Verified valid and invalid MATCHING configuration handling.
* Added MATCHING JSON sample to `docs\block-configuration\block_json_structures.md`
* Created a new JIRA ticket document for tracking.
