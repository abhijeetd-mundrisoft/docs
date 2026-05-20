# Implementation Plan - Block Styling & Configuration Support

This document outlines the backend implementation plan for supporting granular configuration settings (spacing, padding, layout, alignment, image width, behavior controls, etc.) for approximately 33 authoring components in the SimplyAuthor ecosystem.

## Goal Description

Implement a unified configuration framework for native authoring components. This includes adding a persistent `settings` layer to store component-specific styling and behavior across:
1. **Native Blocks** (Lesson Creation)
2. **Course Templates**
3. **Lesson Templates**

## User Review Required

> [!IMPORTANT]
> The implementation involves adding a generic `settings` field (JSON/Map) to all block entities. This ensures maximum flexibility for the UI team to pass any configuration key-value pairs (e.g., `backgroundColor`, `paddingTop`, `imageWidth`).

> [!WARNING]
> Database migrations are required for the `blocks` and `template_blocks` tables to add the `settings` column.

## Proposed Changes

### [Phase 1] Database & Entity Layer

#### [NEW] [V20240512__add_block_settings.sql](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/resources/db/migration/V20240512__add_block_settings.sql)
- Migration to add `settings` column (JSON type) to the `blocks` and `template_blocks` tables.

#### [MODIFY] [Block.java](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/java/com/mundrisoft/courseforge/entity/Block.java)
- Add `private String settings;` field to store configuration JSON.

#### [MODIFY] [TemplateBlock.java](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/java/com/mundrisoft/courseforge/entity/TemplateBlock.java)
- Add `private String settings;` field to store configuration JSON.

---

### [Phase 2] DTO & Mapping Layer

#### [MODIFY] Block DTOs
Update the following DTOs to include the `settings` field:
- `BlockRequestDto`
- `BlockResponseDto`
- `BlockUpdateRequestDto`
- `TemplateBlockResponseDto`
- `TemplateBlockUpdateDto`

#### [MODIFY] [BlockMapper.java](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/java/com/mundrisoft/courseforge/mapper/BlockMapper.java)
- Update conversion logic to map `settings` between Entity and DTO.

#### [MODIFY] [TemplateBlockMapper.java](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/java/com/mundrisoft/courseforge/mapper/TemplateBlockMapper.java)
- Update conversion logic to map `settings` between Entity and DTO.

---

### [Phase 3] API & Service Layer

#### [MODIFY] [BlockController.java](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/java/com/mundrisoft/courseforge/controller/BlockController.java)
- Update `createBlock` and `updateBlock` to accept `settings` as an optional request parameter (JSON string).
- Ensure `settings` are passed to the `Block` entity during creation and update.

#### [MODIFY] [CourseTemplateController.java](file:///d:/SimplyAuthor/project/course-forge-backend/src/main/java/com/mundrisoft/courseforge/controller/CourseTemplateController.java)
- Update bulk update/create logic to handle the `settings` field from DTOs for `TemplateBlock` persistence.

## Verification Plan

### Automated Tests
- Integration tests for `BlockController` to verify `settings` persistence during block creation and update.
- Integration tests for `CourseTemplateController` to verify `settings` persistence in templates.
- Unit tests for `BlockMapper` to ensure correct JSON mapping.

### Manual Verification
- Use Postman to create a block with `settings`: `{"backgroundColor": "#f0f0f0", "padding": "20px"}`.
- Verify the settings are returned in the GET response.
- Create a course from a template and verify that the block settings are correctly copied from the template to the new course.
- Verify that updating a block's settings preserves the existing content.

---

## Appendix: Authoring Components List for Configuration

| Category | Supported Block Types |
| :--- | :--- |
| **1. Text Blocks** | `TITLE`, `TEXT`, `HEADING_TEXT`, `HEADING_ONLY`, `SUBHEADING_ONLY`, `TEXT_WITH_SUBHEADING`, `TWO_COLUMN_TEXT`, `THREE_COLUMN_TEXT` |
| **2. Announcement** | `ANNOUNCEMENT`, `ANNOUNCEMENT_NOTE` |
| **3. Image Blocks** | `IMAGE`, `BANNER_IMAGE`, `TWO_COLUMN_IMAGE`, `THREE_COLUMN_IMAGE`, `IMAGE_GRID` |
| **4. Video Blocks** | `VIDEO`, `TWO_COLUMN_VIDEO`, `THREE_COLUMN_VIDEO` |
| **5. Media Blocks** | `AUDIO_PLAYER`, `RESOURCE_FILE`, `EMBED` |
| **6. Quote Blocks** | `QUOTE`, `QUOTE_LARGE_TEXT`, `QUOTE_BOXED`, `QUOTE_WITH_IMAGE`, `QUOTE_CAROUSEL` |
| **7. Quiz Blocks** | `QUIZ`, `CHECKPOINT` |
| **8. Interactive** | `DRAG_AND_DROP`, `FLASH_CARDS_GRID`, `FLASH_CARDS_STACK`, `INTERACTIVE_IMAGE`, `TAB`, `ACCORDION`, `TIMELINE_VIEW`, `STEP_FLOW`, `SORT_AND_LEARN`, `ACTION_BUTTON`, `ACTION_BUTTON_GROUP` |
| **9. Charts Blocks** | `BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT` |
| **10. List Blocks** | `BULLETED_LIST`, `NUMBERED_LIST`, `CHECKBOX_LIST` |
| **11. Layout & Flow** | `IMAGE_TEXT_BOTTOM`, `TEXT_IMAGE_BOTTOM`, `IMAGE_TEXT_RIGHT`, `TEXT_IMAGE_LEFT`, `VIDEO_TEXT_BOTTOM`, `TEXT_VIDEO_BOTTOM`, `VIDEO_TEXT_RIGHT`, `TEXT_VIDEO_LEFT`, `SECTION_BREAK`, `SPACING`, `STEP_MARKER` |
| **12. Summary** | `SUMMARY` |
| **13. Tables** | `TABLE` |

