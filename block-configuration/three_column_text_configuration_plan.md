# Three Column Text Settings Requirement Analysis

This document outlines the requirement analysis and implementation strategy for adding robust configurable **Layout** (`sectionSpacing`, `verticalSpacing`) settings for the Three Column Text block type. It leverages the robust dual-mode validation architecture built for Two Column Text, ensuring high code reusability.

## Proposed Changes

### 1. Configuration Model Changes

#### Settings JSON Structure
The block supports both flat and nested JSON structures. 

**Option A (Nested Structure as requested):**
```json
{
  "settings": {
    "layout": {
      "sectionSpacing": {
        "preset": "narrow"
      },
      "verticalSpacing": {
        "value": 25
      }
    }
  }
}
```

**Option B (Flat Structure):**
```json
{
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25,
      "verticalSpacingLinked": true
    }
  }
}
```

### 2. Backend / Domain Logic

#### Code Reusability
* **Shared Validators**: `BlockContentValidator.validateLayoutSettings()` already natively supports both flat and nested `sectionSpacing` and `verticalSpacing` formats. No backend changes are required for validation logic!
* **Factory Fallbacks**: `THREE_COLUMN_TEXT` is already registered inside `layoutBlocks` in `BlockContentFactory.java` and receives default settings.
* **Schema Definition**: Already registered in `BlockSchemaUtil.java`.

### 3. Validation Requirements

* **Section Spacing**: Restrict presets strictly to `["narrow", "regular", "wide"]` (flat or nested).
* **Vertical Spacing**: Enforce spacing bounds strictly between `0-150`.

## Verification Plan

### Automated Tests
* We will write dedicated unit tests inside `BlockLayoutSettingsTest.java` verifying that:
  1. Flat configurations for `THREE_COLUMN_TEXT` validate cleanly.
  2. Nested preset and spacing configurations validate successfully.
  3. Invalid nested presets or vertical spacing numbers out of range are rejected with standard validation exceptions.

### Manual Verification
* Build and execute the test runner suite utilizing `mvn test`.
