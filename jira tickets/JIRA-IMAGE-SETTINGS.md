# JIRA: CC-10254 - Support Configurable Settings for IMAGE BlockType

## Status: Ready for Review
**Component**: Backend, Authoring Core, API, Schema & Validation, Documentation
**Priority**: High
**Assignee**: Antigravity

## Description

Implement full settings configuration support for the standard `IMAGE` blockType inside the SimplyAuthor system. This aligns the basic `IMAGE` component with the scalable, configuration-driven architecture defined in `block_configuration_plan.md` to support settings (Layout, Shape, Interaction, Background) dynamically.

## Functional Requirements

1. **Settings Default Initialization**:
   - Newly created or upgraded `IMAGE` blocks must be added to the layout-configurable blocks group.
   - `IMAGE` blocks should receive standard layout default configurations:
     - `layout.contentArea` = `"regular"`
     - `layout.sectionSpacing` = `"narrow"`
     - `layout.verticalSpacingTop` = `25`
     - `layout.verticalSpacingBottom` = `25`
     - `layout.verticalSpacingLinked` = `true`
     - `background.style` = `"Light"`
   - Add type-specific settings defaults for `IMAGE` blocks:
     - `shape.roundedCorners` = `0` (range 0–30)
     - `interaction.openImageOnClick` = `true` (toggle boolean)

2. **Validation Rules**:
   - Standard semantic validators apply, but `IMAGE` blocks specifically override standard ranges:
     - `layout.verticalSpacingTop` & `layout.verticalSpacingBottom` strictly validated to `0-120` (standard layout block range is up to `150`).
     - `shape.roundedCorners` strictly validated to `0-30` (standard block layout border radius range is up to `50`).
   - If settings JSON is invalid or out of these ranges, throw clean, descriptive `IllegalArgumentException`.

3. **AI Schema Documentation**:
   - `BlockSchemaUtil` must declare the optional `"settings"` property for `"IMAGE"` under `basicTypes` schema.

4. **Documentation**:
   - Document JSON structures for `"IMAGE"` settings under `Media Blocks` in `docs/block-configuration/block_json_structures.md`.

## Subtasks

- [x] **CC-10254.1**: Add `"IMAGE"` to standard layout default groups and implement block-type-specific defaults for shape and interaction in `BlockContentFactory.java`.
- [x] **CC-10254.2**: Add strict range validation checking in `BlockContentValidator.java` under `validateImageContent`.
- [x] **CC-10254.3**: Add `"settings"` property declaration to `"IMAGE"` block schema in `BlockSchemaUtil.java`.
- [x] **CC-10254.4**: Document the new JSON structure of `IMAGE` block settings in `docs/block-configuration/block_json_structures.md`.
- [x] **CC-10254.5**: Create this JIRA ticket documentation.
- [ ] **CC-10254.6**: Write backend unit tests in `BlockContentValidatorTest.java` and `BlockContentFactoryTest.java` verifying defaults and validation ranges.
- [ ] **CC-10254.7**: Run all backend tests and verify success.

## Edge Case Checklist

- **Backward Compatibility**: Existing image blocks with null/empty settings must default gracefully without throwing validation or deserialization errors.
- **Input Spacing Range**: Invalid vertical spacing values (e.g. negative values or greater than 120) must be rejected with appropriate range error messages.
- **Rounded Corners Range**: Invalid rounded corners values (e.g. negative values or greater than 30) must be rejected with appropriate range error messages.
- **Malformed JSON**: Any malformed JSON settings string should be caught gracefully and throw appropriate IllegalArgumentException.
