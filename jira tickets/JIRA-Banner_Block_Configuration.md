# Banner Block Configuration

## Title
Implement Configuration-Driven Layout Settings for Banner Block (`BANNER_IMAGE`)

## Objective
Enable customizable Layout settings (Section Spacing and Vertical Spacing) for the `BANNER_IMAGE` block type. This will leverage the backend's shared layout configuration structure (`settings.layout`) to ensure consistency, strict validation, and backward compatibility.

## Description
The `BANNER_IMAGE` block currently only supports content validation (image and alt text). We need to extend it to support the new unified block configuration architecture, allowing users to configure Section Spacing and Vertical Spacing without introducing breaking changes to existing blocks.

This task involves:
1.  **Factory Defaults:** Updating `BlockContentFactory.java` to inject default `settings.layout` values (`sectionSpacing`: "narrow", `verticalSpacingTop`: 25, `verticalSpacingBottom`: 25) when a `BANNER_IMAGE` block is created or migrated.
2.  **Schema Definition:** Updating `BlockSchemaUtil.java` to expose the optional `settings` object containing layout configurations.
3.  **Documentation:** Adding a full JSON structural example to `block_json_structures.md`.

## Technical Approach
*   **Validator (`BlockContentValidator.java`):** No explicit code changes are required in `validateBannerImageContent`. The `settings.layout` structure is centrally validated in the main block validation logic, which handles range validation (0-150) for vertical spacing inherently.
*   **Factory (`BlockContentFactory.java`):** Add `"BANNER_IMAGE"` to the `layoutBlocks` set within the `createSettings` method so it receives the standard default layout configurations.

## Acceptance Criteria
- [ ] `BANNER_IMAGE` blocks automatically initialize with a `settings.layout` object containing `sectionSpacing: "narrow"` and linked vertical spacing of 25.
- [ ] The backend API accurately validates `settings.layout.verticalSpacingTop` and `settings.layout.verticalSpacingBottom` against the allowed 0-150 range.
- [ ] The `/api/blocks/schema` endpoint explicitly exposes `settings` for the `BANNER_IMAGE` block type.
- [ ] Legacy `BANNER_IMAGE` blocks without a `settings` object continue to render and validate successfully (Backward Compatibility).
- [ ] Documentation (`block_json_structures.md`) accurately reflects the newly supported layout schema for `BANNER_IMAGE`.
