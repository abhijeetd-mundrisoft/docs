# Sorting Activity BlockType Settings Requirement Analysis

This document provides a detailed requirement analysis and implementation approach for adding configurable **Layout** settings (`sectionSpacing`, `verticalSpacing`) for the `SORTING_ACTIVITY` blockType.

## Proposed Changes

### 1. Configuration Model Changes

#### Schema Updates
* Introduce `sectionSpacing` (Segmented control: `narrow`, `regular`, `wide` with default `narrow`) and `verticalSpacing` (Numeric: `0-150` with default `25`) within the `layout` setting group.

#### Settings JSON Structure
```json
{
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    }
  }
}
```

#### Serialization and Deserialization
* The backend's Jackson configuration will naturally deserialize this `settings` node as a generic JSON map or string, which is standard across all block types. Missing or legacy configuration payloads will deserialize safely, allowing the factory and renderer to apply default layout values.

### 2. Backend / Domain Logic

#### Reusable Abstraction Opportunities
* **Shared Utilities**: We can leverage the existing `BlockContentValidator` for applying generic spacing validation. Since `BlockContentValidator.validateBlockSettings()` globally parses and validates the `layout` group, no custom `SORTING_ACTIVITY`-specific validation logic needs to be written.
* **Factory Fallbacks**: We will centralize the layout spacing default values within `BlockContentFactory` by adding `"SORTING_ACTIVITY"` to the `layoutBlocks` constant. This guarantees proper `verticalSpacingTop` and `verticalSpacingBottom` mapping.
* **Schema Definition**: We will register the `SORTING_ACTIVITY` block inside `BlockSchemaUtil.java` (for AI generation schema) to reflect the `layout` shared settings.
* **Block Definition**: Added `SORTING_ACTIVITY` to `Block.BlockType` enum.

### 3. Rendering and Styling Integration

* **Layout Integration**: The frontend renderer will extract `settings.layout.sectionSpacing` to apply the appropriate CSS wrapping container classes. It will feed `settings.layout.verticalSpacing` into global spacing utility classes (padding top/bottom).
* **Extensibility**: Keeping these values inside the `layout` object ensures they plug directly into existing container layouts without custom CSS overrides specifically for `SORTING_ACTIVITY`.

### 4. Validation Requirements

* **Enum Validation**: The existing `validateLayoutSettings` method inherently restricts `sectionSpacing` to `["narrow", "regular", "wide"]`.
* **Numeric Validation**: The existing `validateLayoutSettings` inherently enforces that `verticalSpacing` falls within the `0-150` boundary via the `validateRange` utility.
* **Invalid Configuration Fallback Behavior**: The validator will throw an `IllegalArgumentException` on invalid data. For missing data, the frontend renderer and `BlockContentFactory` will gracefully apply the `narrow` and `25` defaults.

### 5. Extensibility Considerations

* **Directional Spacing**: Driven by `BlockContentFactory`, `verticalSpacing` is mapped to `verticalSpacingTop` and `verticalSpacingBottom` with `verticalSpacingLinked: true`, easily supporting unlinked top/bottom spacing in the future.
* **Responsive Layouts**: The configuration object naturally supports extension with `verticalSpacingMobile` or `verticalSpacingDesktop` overrides when the platform requires it.
* **Reusable Layout Token Systems**: By utilizing the exact same keys (`sectionSpacing`, `verticalSpacing`) as native blocks (like `TEXT`, `VIDEO`), we ensure standard design token utilization.

### 6. Migration / Compatibility Considerations

* **Impact on Existing Blocks**: Legacy blocks currently lack this configuration. The implementation is fully backward-compatible as the renderer will adopt the default values (`narrow`, `25px`) if `settings.layout` is missing.
* **Schema Evolution**: The `settings` object is open for extension.
