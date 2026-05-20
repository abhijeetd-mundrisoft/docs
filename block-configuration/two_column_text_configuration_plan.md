# Two Column Text Settings Requirement Analysis

This document outlines the requirement analysis and implementation strategy for adding robust configurable **Layout** (`sectionSpacing`, `verticalSpacing`) settings for the Two Column Text block type. It supports both flat-value configurations and nested object structures to ensure complete flexibility and backward compatibility.

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
        "value": 25,
        "linked": true
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

#### Dual-Mode Validation Strategy
To support both configuration formats cleanly in `BlockContentValidator.validateLayoutSettings()`, we will enhance the spacing validator to:
1. **Section Spacing**:
   * If `sectionSpacing` is a `String`, validate it directly.
   * If `sectionSpacing` is a `Map`, extract the `"preset"` key and validate it.
2. **Vertical Spacing**:
   * If `verticalSpacing` is a `Number`, validate it using the generic `validateRange` utility.
   * If `verticalSpacing` is a `Map`, extract the `"value"` and validate it using `validateRange` (and optionally validate `"linked"` as a Boolean).

#### Factory Spacing Defaults
`TWO_COLUMN_TEXT` is already registered as a core layout block inside `BlockContentFactory.java` and receives flat default values of `sectionSpacing: "narrow"` and `verticalSpacingTop/Bottom: 25`. It is fully backwards compatible out-of-the-box.

#### AI Schema Mapping
Register `TWO_COLUMN_TEXT` nested layout schemas inside `BlockSchemaUtil.java`.

### 3. Validation Requirements

* **Section Spacing**: Restrict presets strictly to `["narrow", "regular", "wide"]`.
* **Vertical Spacing**: Enforce spacing bounds strictly between `0-150`.
* **Nested Formats**: Extract and validate nested `preset` and `value`/`linked` properties safely with descriptive failure handling.

## Verification Plan

### Automated Tests
* We will write dedicated unit tests inside `BlockLayoutSettingsTest.java` verifying that:
  1. Flat configurations for two column layout validate cleanly.
  2. Nested preset and spacing configurations validate successfully.
  3. Invalid nested presets or vertical spacing numbers out of range are rejected with standard validation exceptions.

### Manual Verification
* Build and execute the test runner suite utilizing `mvn test`.
