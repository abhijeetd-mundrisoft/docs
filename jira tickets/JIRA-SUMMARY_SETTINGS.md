# JIRA Ticket: SUMMARY Settings Block Configuration Support

## Description

Implement settings configuration support for the **`Summary` (`SUMMARY`)** blockType to allow users to customize layout, section spacing, vertical padding, and background styles.

### Business Value
Allows course creators to adjust summary block containers visually (e.g., width boundaries, spacing buffers) to fit design needs while maintaining strict and predictable vertical layout proportions.

---

## Technical Specifications

### 1. Settings Schema Attributes
- **`layout.contentArea`**: Segmented control. Values: `compact`, `regular`, `large`. Default: `regular`.
- **`layout.sectionSpacing`**: Segmented control. Values: `narrow`, `regular`, `wide`. Default: `narrow`.
- **`layout.verticalSpacingTop` / `layout.verticalSpacingBottom`**: Slider control. Range: `0–120`. Default: `25`. Linked by default (`layout.verticalSpacingLinked` = `true`).
- **`background.style`**: Visual presets. Values: `Light`, `Gray`, `Theme`, `Theme Tint`, `Dark`, `Black`, `Custom`, `Image`. Default: `Light`.

### 2. Validation Constraints
- Vertical spacing parameters must reside strictly within `[0, 120]` for `SUMMARY` blockType.
- Invalid enums for `contentArea` or `sectionSpacing` must throw validation exceptions.
- Backward compatibility: Legacy blocks mapped to `{}` or missing layout options must gracefully fallback to the factory defaults without breaking existing content.

---

## Task Checklist & Subtasks

- **[x] CF-401**: Register `SUMMARY` for standard native layout defaults mapping in `BlockContentFactory.java`.
- **[x] CF-402**: Overload settings validation in `BlockContentValidator.java` to accept context-aware block types.
- **[x] CF-403**: Implement dynamic spacing limit (`0–120`) specifically for `SUMMARY` in `BlockContentValidator.java`.
- **[x] CF-404**: Update `BlockSchemaUtil.java` to include settings parameters inside the AI generation prompt schema.
- **[x] CF-405**: Add comprehensive unit testing verifying happy paths, boundary validations, and edge cases.
- **[x] CF-406**: Document JSON structure in `docs/block-configuration/block_json_structures.md`.

---

## Impact Assessment
- **Symmetric Balance**: Reuses CourseForge's centralized layout system, ensuring future layout features automatically apply to `SUMMARY`.
- **Zero Regression**: Existing native blocks are 100% unaffected because their vertical spacing range of `[0, 150]` remains strictly enforced and fully compatible.
- **Robustness**: Validation exceptions protect database persistence from corrupt state values.
