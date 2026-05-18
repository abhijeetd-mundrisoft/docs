# JIRA: CC-10255 - Support Configurable Settings for IMAGE_TEXT_RIGHT & TEXT_IMAGE_LEFT BlockTypes

## Status: Ready for Review
**Component**: Backend, Authoring Core, API, Schema & Validation, Documentation
**Priority**: High
**Assignee**: Antigravity

## Description

Implement full settings configuration support for the symmetrical `Image text right` (`IMAGE_TEXT_RIGHT`) and `Text image left` (`TEXT_IMAGE_LEFT`) blockTypes inside the SimplyAuthor system. This aligns these multi-element layout components with the scalable, configuration-driven architecture defined in `block_configuration_plan.md` to support settings (Layout, Media Layout, Shape, Interaction, Background) dynamically.

## Functional Requirements

1. **Settings Default Initialization**:
   - Newly created or upgraded `IMAGE_TEXT_RIGHT` and `TEXT_IMAGE_LEFT` blocks must be added to the layout-configurable blocks group.
   - Both blocks should receive standard layout default configurations:
     - `layout.contentArea` = `"regular"`
     - `layout.sectionSpacing` = `"narrow"`
     - `layout.verticalSpacingTop` = `25`
     - `layout.verticalSpacingBottom` = `25`
     - `layout.verticalSpacingLinked` = `true`
     - `background.style` = `"Light"`
   - Add type-specific settings defaults:
     - `mediaLayout.imageSize` = `"regular"` (options: `compact`, `regular`, `large`)
     - `mediaLayout.imagePosition` = `"right"` (for `IMAGE_TEXT_RIGHT`) or `"left"` (for `TEXT_IMAGE_LEFT`) (options: `left`, `right`)
     - `shape.roundedCorners` = `0` (range 0–30)
     - `interaction.openImageOnClick` = `true` (toggle boolean)

2. **Validation Rules**:
   - Standard semantic validators apply, but both blocks specifically override standard ranges:
     - `mediaLayout.imagePosition` strictly validated to `"left"` or `"right"`.
     - `mediaLayout.imageSize` strictly validated to `"compact"`, `"regular"`, or `"large"`.
     - `layout.verticalSpacingTop` & `layout.verticalSpacingBottom` strictly validated to `0-120` (standard layout block range is up to `150`).
     - `shape.roundedCorners` strictly validated to `0-30` (standard block layout border radius range is up to `50`).
   - If settings JSON is invalid or out of these ranges, throw clean, descriptive `IllegalArgumentException`.

3. **AI Schema Documentation**:
   - `BlockSchemaUtil` must declare the optional `"settings"` property for `"IMAGE_TEXT_RIGHT"` and `"TEXT_IMAGE_LEFT"` under `basicTypes` schema.

4. **Documentation**:
   - Document JSON structures for `"IMAGE_TEXT_RIGHT"` and `"TEXT_IMAGE_LEFT"` settings under `Media Blocks` in `docs/block-configuration/block_json_structures.md`.

## Subtasks

- [x] **CC-10255.1**: Add both blocks to standard layout default groups and implement block-type-specific defaults for media layout, shape, and interaction in `BlockContentFactory.java`.
- [x] **CC-10255.2**: Add strict range validation checking in `BlockContentValidator.java` under `validateImageTextRightContent` and `validateTextImageLeftContent` utilizing a reusable helper.
- [x] **CC-10255.3**: Add `"settings"` property declaration to `"IMAGE_TEXT_RIGHT"` and `"TEXT_IMAGE_LEFT"` block schemas in `BlockSchemaUtil.java`.
- [x] **CC-10255.4**: Document the new JSON structure of both block settings in `docs/block-configuration/block_json_structures.md`.
- [x] **CC-10255.5**: Create this JIRA ticket documentation.
- [ ] **CC-10255.6**: Write backend unit tests in `BlockContentValidatorTest.java` and `BlockContentFactoryTest.java` verifying defaults and validation ranges.
- [ ] **CC-10255.7**: Run all backend tests and verify success.

## Edge Case Checklist

- **Backward Compatibility**: Existing blocks with null/empty settings must default gracefully without throwing validation or deserialization errors.
- **Input Spacing Range**: Invalid vertical spacing values (e.g. negative values or greater than 120) must be rejected with appropriate range error messages.
- **Rounded Corners Range**: Invalid rounded corners values (e.g. negative values or greater than 30) must be rejected with appropriate range error messages.
- **Image Sizing Option**: Invalid imageSize options must throw an IllegalArgumentException from the generic mediaLayout validator.
- **Image Position Option**: Symmetrical position configurations (e.g. setting IMAGE_TEXT_RIGHT to `"left"` or vice-versa) must be fully valid and allowed.
- **Malformed JSON**: Any malformed JSON settings string should be caught gracefully and throw appropriate IllegalArgumentException.
