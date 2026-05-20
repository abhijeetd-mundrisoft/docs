# Task Breakdown: Block Configuration Implementation

This document breaks down the implementation plan into actionable tasks to track progress across the team.

## Phase 1: Foundation (Completed ✅)
- [x] Database Migration: Add `settings` column to `blocks` and `template_blocks`.
- [x] Entity Layer: Update `Block.java` and `TemplateBlock.java` with the `settings` field.
- [x] DTO Layer: Update `BlockRequestDto`, `BlockResponseDto`, and `TemplateBlockUpdateDto`.
- [x] Mapping Layer: Update `BlockMapper` and `TemplateBlockMapper`.

## Phase 2: Logic & Bulk Operations (Completed ✅)
- [x] Update `BlockCreateItemDto` and `BulkBlockUpdateItemDto` for bulk operations.
- [x] Implement settings persistence in `BlockService.bulkCreateBlocks`.
- [x] Implement settings persistence in `BlockService.bulkUpdateBlocks`.
- [x] Implement settings handling in `CourseTemplateController` (Bulk Create/Update).

## Phase 3: Component-Specific JSON Definition (In Progress 🏗️)
For each of the ~33 components, we need to define the JSON schema and document it in `block_json_structures.md`.

### 1. Text Blocks
- [x] Paragraph (`TEXT`)
- [x] Paragraph with heading (`HEADING_TEXT`)
- [x] Paragraph with subheading (`TEXT_WITH_SUBHEADING`)
- [x] Heading only (`HEADING_ONLY`)
- [x] Subheading only (`SUBHEADING_ONLY`)
- [ ] Two column text (`TWO_COLUMN_TEXT`)
- [ ] Three column text (`THREE_COLUMN_TEXT`)

### 2. Announcement & Quotes
- [x] Announcement (`ANNOUNCEMENT`)
- [x] Announcement Note (`ANNOUNCEMENT_NOTE`)
- [x] Quote (`QUOTE`)
- [ ] Quote Boxed (`QUOTE_BOXED`)
- [x] Quote Carousel (`QUOTE_CAROUSEL`)

### 3. Media & Interactive
- [ ] Image (`IMAGE`)
- [ ] Video (`VIDEO`)
- [ ] Audio Player (`AUDIO_PLAYER`)
- [ ] Tab (`TAB`)
- [ ] Accordion (`ACCORDION`)
- [ ] Flash Cards (`FLASH_CARDS_GRID`)
- [ ] Interactive Image (`INTERACTIVE_IMAGE`)
- [x] Image Carousel (`IMAGE_CAROUSEL`)
- [x] Two Column Grid (`TWO_COLUMN_IMAGE`)
- [x] Three Column Grid (`THREE_COLUMN_IMAGE`)
- [x] Four Column Grid (`IMAGE_GRID`)
- [x] Table (`TABLE`)

### 4. Charts & Lists
- [ ] Bulleted List (`BULLETED_LIST`)
- [ ] Numbered List (`NUMBERED_LIST`)
- [ ] Checkbox List (`CHECKBOX_LIST`)
- [ ] Bar Chart (`BAR_CHART`)

*Note: Add other blocks from the Appendix as we progress.*

## Phase 4: Verification & Testing (Pending ⏳)
- [ ] **API Validation**: Verify that invalid JSON in `settings` doesn't crash the server.
- [ ] **Template Cloning**: Verify that when a course is created from a template, all `settings` are copied correctly.
- [ ] **Bulk Performance**: Test bulk save with 50+ blocks to ensure JSON processing is efficient.
- [ ] **UI Integration**: Verify with the UI team that the retrieved JSON matches their component requirements.

---

> [!TIP]
> Mark items with `[x]` as you complete them to keep the team aligned.
