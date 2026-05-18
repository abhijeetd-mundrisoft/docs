# JIRA: CC-10256 - Support Configurable Settings for IMAGE_TEXT_BOTTOM & TEXT_IMAGE_BOTTOM BlockTypes

## Status: Ready for Review
**Component**: Backend, Authoring Core, API, Schema & Validation, Documentation
**Priority**: High
**Assignee**: Antigravity

## Description

Implement full settings configuration support for the symmetrical `Image text bottom` (`IMAGE_TEXT_BOTTOM`) and `Text image bottom` (`TEXT_IMAGE_BOTTOM`) blockTypes inside the SimplyAuthor system. This aligns these layout-configurable components with the scalable, configuration-driven architecture defined in `block_configuration_plan.md` to support settings (Layout, Media Layout, Shape, Interaction, Background) dynamically.

## Functional Requirements

1. **Settings Default Initialization**:
   - Newly created or upgraded `IMAGE_TEXT_BOTTOM` and `TEXT_IMAGE_BOTTOM` blocks must be added to the layout-configurable blocks group.
   - Both blocks should receive standard layout default configurations:
     - `layout.contentArea` = `"regular"`
     - `layout.sectionSpacing` = `"narrow"`
     - `layout.verticalSpacingTop` = `25`
     - `layout.verticalSpacingBottom` = `25`
     - `layout.verticalSpacingLinked` = `true`
     - `background.style` = `"Light"`
   - Add type-specific settings defaults:
     - `mediaLayout.imageSize` = `"regular"` (options: `compact`, `regular`, `large`)
     - `mediaLayout.imagePosition` = `"bottom"` (for both blocks) (options: `top`, `bottom`)
     - `shape.roundedCorners` = `0` (range 0–30)
     - `interaction.openImageOnClick` = `true` (toggle boolean)

2. **Validation Rules**:
   - Standard semantic validators apply, but both blocks specifically override standard ranges:
     - `mediaLayout.imagePosition` strictly validated to `"top"` or `"bottom"`.
     - `mediaLayout.imageSize` strictly validated to `"compact"`, `"regular"`, or `"large"`.
     - `layout.verticalSpacingTop` & `layout.verticalSpacingBottom` strictly validated to `0-120` (standard layout block range is up to `150`).
     - `shape.roundedCorners` strictly validated to `0-30` (standard block layout border radius range is up to `50`).
   - If settings JSON is invalid or out of these ranges, throw clean, descriptive `IllegalArgumentException`.

3. **AI Schema Documentation**:
   - `BlockSchemaUtil` must declare the optional `"settings"` property for `"IMAGE_TEXT_BOTTOM"` and `"TEXT_IMAGE_BOTTOM"` under `basicTypes` schema.

4. **Documentation**:
   - Document JSON structures for `"IMAGE_TEXT_BOTTOM"` and `"TEXT_IMAGE_BOTTOM"` settings under `Media Blocks` in `docs/block-configuration/block_json_structures.md`.

## Subtasks

- [x] **CC-10256.1**: Add both blocks to standard layout default groups and implement block-type-specific defaults for media layout, shape, and interaction in `BlockContentFactory.java`.
- [x] **CC-10256.2**: Add strict range validation checking in `BlockContentValidator.java` under `validateImageTextBottomContent` and `validateTextImageBottomContent` utilizing a reusable helper, and support `top`/`bottom` in `imagePosition`.
- [x] **CC-10256.3**: Add `"settings"` property declaration to `"IMAGE_TEXT_BOTTOM"` and `"TEXT_IMAGE_BOTTOM"` block schemas in `BlockSchemaUtil.java`.
- [x] **CC-10256.4**: Document the new JSON structure of both block settings in `docs/block-configuration/block_json_structures.md`.
- [x] **CC-10256.5**: Create this JIRA ticket documentation.
- [ ] **CC-10256.6**: Write backend unit tests in `BlockContentValidatorTest.java` and `BlockContentFactoryTest.java` verifying defaults and validation ranges.
- [ ] **CC-10256.7**: Run all backend tests and verify success.

## Edge Case Checklist

- **Backward Compatibility**: Existing blocks with null/empty settings must default gracefully without throwing validation or deserialization errors.
- **Input Spacing Range**: Invalid vertical spacing values (e.g. negative values or greater than 120) must be rejected with appropriate range error messages.
- **Rounded Corners Range**: Invalid rounded corners values (e.g. negative values or greater than 30) must be rejected with appropriate range error messages.
- **Image Sizing Option**: Invalid imageSize options must throw an IllegalArgumentException from the generic mediaLayout validator.
- **Image Position Option**: Positioning configurations (setting `top` or `bottom`) must be fully valid and allowed.
- **Malformed JSON**: Any malformed JSON settings string should be caught gracefully and throw appropriate IllegalArgumentException.
