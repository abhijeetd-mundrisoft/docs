# SCORM Export Requirement Analysis

## 1. Objective

Define requirements for stabilizing and extending Course SCORM export so that:

- Existing export flows (`SCORM_1_2`, `SCORM_2004`) remain backward compatible.
- All supported `BlockType` values render consistently in exported lessons.
- Unsupported blocks are tracked, prioritized, and implemented incrementally.
- Validation, media packaging, and tracking behavior are reliable and testable.

## 2. Scope

### In Scope

- Course export flow via `/export/course`.
- SCORM 1.2 and SCORM 2004 strategy behavior.
- Block rendering coverage based on `Block.BlockType` enum in `Block.java`.
- Functional and non-functional requirements for export quality.
- Implementation roadmap for pending block types.

### Out of Scope

- Full frontend authoring UX redesign.
- Legacy endpoint contract changes.
- DB schema-breaking migrations.
- LMS-specific custom runtime logic beyond current SCORM packaging expectations.

## 3. Current System Flow (As-Is)

1. Request enters `CourseExportController` (`POST /export/course`).
2. `CourseExportService.exportCourseSync()` validates:
   - Export type availability.
   - Tracking configuration.
   - Quiz existence for `QUIZ_RESULT` tracking.
3. Strategy resolution through `ExportStrategy` map.
4. Strategy export execution (`Scorm12ExportStrategy` or `Scorm2004ExportStrategy`):
   - Create workspace.
   - Generate course + lesson HTML.
   - Copy SCORM templates/assets.
   - Copy media referenced by blocks.
   - Generate manifest.
   - Zip output package.
5. Controller returns file download response.

## 4. Block Type Coverage Analysis

Reference source: `Block.BlockType` in `Block.java`.

### 4.1 Implemented in SCORM export rendering switch (both 1.2 and 2004)

- `TITLE`
- `TEXT`
- `IMAGE`
- `VIDEO`
- `QUIZ`
- `SUMMARY`
- `HEADING_TEXT`
- `HEADING_ONLY`
- `QUOTE`
- `QUOTE_LARGE_TEXT`
- `QUOTE_BOXED`
- `TWO_COLUMN_TEXT`
- `THREE_COLUMN_TEXT`
- `TWO_COLUMN_IMAGE`
- `THREE_COLUMN_IMAGE`
- `IMAGE_TEXT_BOTTOM`
- `TEXT_IMAGE_BOTTOM`
- `IMAGE_TEXT_RIGHT`
- `TEXT_IMAGE_LEFT`
- `TWO_COLUMN_VIDEO`
- `THREE_COLUMN_VIDEO`
- `VIDEO_TEXT_BOTTOM`
- `TEXT_VIDEO_BOTTOM`
- `VIDEO_TEXT_RIGHT`
- `TEXT_VIDEO_LEFT`
- `FLASH_CARDS_GRID`
- `TAB`
- `ACCORDION`

### 4.2 Pending / unsupported in SCORM rendering switch

- `AUDIO_PLAYER`
- `CODE_EXAMPLE`
- `DRAG_AND_DROP`
- `FLASH_CARDS` (legacy alias behavior should be confirmed)
- `FLASH_CARDS_STACK`
- `INTERACTIVE_IMAGE`
- `TIMELINE_VIEW`
- `STEP_FLOW`
- `IMAGE_GRID`
- `QUOTE_CAROUSEL`
- `IMAGE_CAROUSEL`
- `BAR_CHART`
- `TREND_CHAT`
- `DISTRIBUTION_CHAT`
- `BANNER_IMAGE`
- `BULLETED_LIST`
- `NUMBERED_LIST`
- `CHECKBOX_LIST`
- `RESOURCE_FILE`
- `TEXT_WITH_SUBHEADING`
- `SUBHEADING_ONLY`
- `QUOTE_WITH_IMAGE`
- `SECTION_BREAK`
- `STEP_MARKER`
- `SPACING`
- `CHECKPOINT`
- `ANNOUNCEMENT`
- `ANNOUNCEMENT_NOTE`
- `ACTION_BUTTON`
- `ACTION_BUTTON_GROUP`
- `SORT_AND_LEARN`
- `EMBED`

## 5. Problem Statement

Current SCORM export has partial block coverage. Unsupported blocks produce fallback output (`Unsupported block type`), causing:

- Content fidelity loss in exported lessons.
- Inconsistent learner experience across course types.
- Higher support burden for course creators.
- Tracking/reporting mismatch when interactive content is missing.

## 6. Functional Requirements (To-Be)

### FR-1: Export Initiation and Validation

- System must keep current endpoint contract stable.
- System must continue validating tracking parameters exactly as current behavior:
  - `COURSE_COMPLETION` requires `completionThreshold`.
  - `QUIZ_RESULT` requires `passingScore` and at least one quiz block.

### FR-2: Block Rendering Coverage

- Each `BlockType` must have one of:
  - Native SCORM render implementation, or
  - Explicit approved fallback mapping.
- Unsupported runtime message must not appear in final learner-facing export for any production-enabled block type.

### FR-3: Media Extraction and Packaging

- Media IDs embedded in block content must be resolved and copied into package.
- Missing files must be logged with context (course/module/lesson/block) and fallback UI should remain usable.
- Media path generation must be deterministic and compatible with both SCORM versions.

### FR-4: Interactive Behavior

- Interactive blocks (e.g., tabs, flash cards, accordions, drag/drop) must load required JS/CSS assets.
- Exported package must work offline in LMS runtime without dependency on app backend.

### FR-5: Manifest and Navigation Integrity

- Every generated lesson HTML file must be present in manifest resources.
- Sequence/order must reflect module and lesson order from database.

### FR-6: Error Handling

- Hard failures (manifest generation error, zip failure) should fail export.
- Soft failures (single missing media item) should log warning and continue when safe.

## 7. Non-Functional Requirements

### NFR-1: Backward Compatibility

- No breaking changes to existing API request/response contracts.
- Existing successful exports must continue to work unchanged.

### NFR-2: Performance

- Export should complete within acceptable time for large courses (target SLA to be finalized).
- Avoid repeated DB calls inside nested loops where batching is feasible.

### NFR-3: Observability

- Structured logs required for each export phase and block rendering failures.
- Include export ID and block context in warning/error logs.

### NFR-4: Reliability

- Temporary workspace cleanup must run on both success and failure.
- Zip output integrity must be verifiable before response.

### NFR-5: Security

- No secrets in logs.
- File operations must remain constrained to approved temp/export directories.

## 8. Gap Analysis Summary

- **Coverage gap:** 33 block types pending.
- **Content risk:** visual, interactive, and resource-rich blocks are currently highest impact.
- **Media gap:** extraction logic currently favors image/video layouts; additional blocks need explicit file-ID extraction rules.
- **Consistency gap:** SCORM 1.2 and 2004 should share common block rendering implementation to avoid divergence.

## 9. Prioritized Implementation Plan

### Phase 1 (High Priority, low-medium complexity)

- Implement: `BULLETED_LIST`, `NUMBERED_LIST`, `CHECKBOX_LIST`, `TEXT_WITH_SUBHEADING`, `SUBHEADING_ONLY`, `SPACING`, `SECTION_BREAK`, `BANNER_IMAGE`, `RESOURCE_FILE`.
- Add media extraction for `BANNER_IMAGE` and `RESOURCE_FILE`.
- Add snapshot tests for rendered HTML fragments.

### Phase 2 (High value interactive/layout)

- Implement: `IMAGE_GRID`, `QUOTE_WITH_IMAGE`, `TIMELINE_VIEW`, `STEP_FLOW`, `STEP_MARKER`, `CHECKPOINT`, `ANNOUNCEMENT`, `ANNOUNCEMENT_NOTE`.
- Add/extend JS + CSS where needed.

### Phase 3 (Complex interactive/custom embeds)

- Implement: `AUDIO_PLAYER`, `CODE_EXAMPLE`, `INTERACTIVE_IMAGE`, `DRAG_AND_DROP`, `SORT_AND_LEARN`, `ACTION_BUTTON`, `ACTION_BUTTON_GROUP`, `EMBED`.
- Define sanitization and security rules for `EMBED`.

### Phase 4 (Carousels/charts/legacy alignment)

- Implement: `QUOTE_CAROUSEL`, `IMAGE_CAROUSEL`, `BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT`, `FLASH_CARDS_STACK`.
- Confirm handling strategy for legacy `FLASH_CARDS` (map to `FLASH_CARDS_GRID` or dedicated renderer).

## 10. Acceptance Criteria

- All production-enabled block types in enum render without unsupported fallback text.
- SCORM 1.2 and 2004 produce equivalent learner-visible output for the same course content.
- Export passes validation and packaging tests for:
  - Mixed media lessons.
  - Interactive blocks.
  - Tracking (`COURSE_COMPLETION`, `QUIZ_RESULT`).
- Automated regression suite confirms no behavior break for already implemented block types.

## 11. Test Strategy

- **Unit tests:** block renderer methods, media extraction functions, fallback behavior.
- **Integration tests:** full strategy export on fixture courses with mixed block sets.
- **Package validation tests:** unzip and assert file presence, manifest references, lesson HTML links.
- **Manual LMS smoke tests:** SCORM Cloud or equivalent LMS for 1.2 and 2004.

## 12. Risks and Mitigations

- **Risk:** High variance in block content JSON structures.  
  **Mitigation:** Per-block schema validation before render; resilient parsing with typed DTOs where possible.

- **Risk:** SCORM runtime JS conflicts with new interactions.  
  **Mitigation:** Isolate block scripts; namespace events; add browser/LMS compatibility smoke tests.

- **Risk:** Export time increase with heavy media blocks.  
  **Mitigation:** Optimize file copy path, deduplicate media IDs, and improve logging for hotspots.

## 13. Open Decisions Needed

- Final priority order from product/content team for pending block types.
- Whether all pending blocks are mandatory for both SCORM versions in the same release.
- Fallback policy for unsupported blocks in non-production environments (warn vs hard fail).
- Definition of export SLA targets by course size tiers.

## 14. Deliverables

- Updated SCORM export renderer coverage for approved phases.
- Automated tests and sample fixture courses.
- Release notes listing newly supported block types.
- Migration/playbook note for legacy block alias handling (especially `FLASH_CARDS`).
