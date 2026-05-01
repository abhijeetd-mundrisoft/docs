# Phase 2 — Development Plan (SCORM course export: layout + journey blocks)

**Goal:** Implement Phase 2 blocks with stable rendering and media packaging in both SCORM 1.2 and SCORM 2004, while explicitly applying Phase 1 lessons learned (JSON tolerance, media path normalization, CSS/JS parity, and runtime-safe fallbacks).

**References:** `SCORM_EXPORT_REQUIREMENT_ANALYSIS.md` §9 (Phase 2), `SCORM_EXPORT_SOLUTION_DESIGN.md` §6–8, `SCORM_EXPORT_PHASE_WISE_DEVELOPMENT_PLAN.md` (Phase 2), `SCORM_EXPORT_PHASE1_DEVELOPMENT_PLAN.md`, `SCORM_EXPORT_PHASE3_DEVELOPMENT_PLAN.md` (next phase scope).

**Depends on:** Phase 0 + Phase 1 foundation already in repo (`ScormBlockHtmlDispatcher`, `ScormBlockRenderContext`, `export.render.blocks` renderers, dual-strategy media extraction).

---

## 1. Scope

### In scope (Phase 2 block types)

| Group | Block types | Notes |
|------|-------------|-------|
| Visual layout | `IMAGE_GRID`, `QUOTE_WITH_IMAGE` | Multi-image and mixed text+image rendering. |
| Journey content | `TIMELINE_VIEW`, `STEP_FLOW`, `STEP_MARKER` | Structured sequence layouts; mostly static presentation in this phase. |
| Messaging | `ANNOUNCEMENT`, `ANNOUNCEMENT_NOTE` | Styled informational blocks; no backend runtime coupling. |

### Out of scope

- `CHECKPOINT` and interactive gating runtime behavior (Phase 4).
- Audio, code, and carousel blocks (`AUDIO_PLAYER`, `CODE_EXAMPLE`, `QUOTE_CAROUSEL`, `IMAGE_CAROUSEL` — Phase 3); action buttons and remaining high-runtime blocks (Phase 4).
- API contract changes to `/export/course`.
- SCORM editor module changes.

---

## 2. Phase 1 learnings to carry forward (mandatory)

These are not optional; they directly address issues observed during Phase 1 implementation and smoke testing:

1. **JSON schema drift is real**  
   - Subheading payloads arrived as rich HTML in some courses (`<h2>...</h2>`), not plain text.  
   - Phase 2 renderers must support both plain and rich content safely.

2. **Media type alias mismatch causes wrong folder resolution**  
   - Example observed: image paths accidentally routed to `../media/videos/...`.  
   - Phase 2 must normalize media aliases centrally (`image/images`, `video/videos`, etc.) and avoid string-literal drift.

3. **Top-level vs nested payload fields vary by block**  
   - Phase 2 extraction must check both common locations (`fileId`, nested image/media arrays/objects).

4. **SCORM 1.2 vs 2004 parity can drift silently**  
   - For every renderer/media rule/CSS/JS change, update both template trees and both strategies in same PR.

5. **CSS/JS assets must be treated as deliverables, not follow-up work**  
   - Visual blocks require matching style rules in both `export-templates/scorm12` and `export-templates/scorm2004`.

6. **Fallback behavior must remain learner-safe**  
   - On malformed JSON or missing media, render non-breaking placeholder and continue export.

---

## 3. Work breakdown (recommended order)

| Step | Task | Done when |
|------|------|-----------|
| **2.1** | JSON payload spike for all Phase 2 blocks from real DB data (top-level + nested forms). | Short schema matrix committed to docs or ticket comments. |
| **2.2** | Add renderers under `...export.render.blocks` (same package as Phase 1 block renderers). | Classes compile and register without duplicate `BlockType`. |
| **2.3** | Implement renderers: `IMAGE_GRID`, `QUOTE_WITH_IMAGE`. | Multi-image/quote rendering works with missing optional fields. |
| **2.4** | Implement renderers: `TIMELINE_VIEW`, `STEP_FLOW`, `STEP_MARKER`. | Ordered sequence rendering stable for mixed/partial content. |
| **2.5** | Implement renderers: `ANNOUNCEMENT`, `ANNOUNCEMENT_NOTE`. | Distinct styles and safe HTML insertion behavior. |
| **2.6** | Extend media extraction in both strategies for Phase 2 image-bearing blocks (especially image lists). | Export ZIP contains required media IDs for Phase 2 fixtures. |
| **2.7** | Add/extend CSS for Phase 2 classes in both SCORM template trees. | No unstyled Phase 2 blocks in manual package review. |
| **2.8** | Add JS only where needed (namespaced, zero backend dependency at runtime). | JS files packaged and referenced in both versions, no global collisions. |
| **2.9** | Add tests (unit + extraction + regression + package checks). | Focused suite green in CI/local. |
| **2.10** | Manual dual-SCORM smoke with fixture course containing all Phase 2 blocks. | LMS/SCORM Cloud evidence attached to ticket. |

---

## 4. Implementation guidance (codebase-specific)

### 4.1 Renderer placement and conventions

- Add new classes under:  
  `course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/render/blocks/`
- Keep one renderer per `BlockType` for traceability and easier fallback debugging.
- Reuse `ScormBlockRenderContext` callbacks:
  - `insertHtmlContent` for rich text
  - `escapeHtml` for plain labels/titles
  - `mediaPathResolver` for packaged media links

### 4.2 Shared helpers

- Continue using `BlockRendererSupport` patterns (or extend only when duplication is justified).
- Add utilities for:
  - ordered item extraction (`orderIndex` aware)
  - flexible read from both array/object forms
  - safe optional key lookup

### 4.3 Strategy integration points

- Keep existing Phase 0 pattern:
  - dispatcher first (`ScormBlockHtmlDispatcher`)
  - legacy fallback second (`generateBlockHtmlLegacy`)
- Media extraction extension remains in:
  - `Scorm12ExportStrategy.extractFileIdsFromBlockContent(...)`
  - `Scorm2004ExportStrategy.extractFileIdsFromBlockContent(...)`

### 4.4 JS/CSS packaging (important for this phase)

- CSS changes in both:
  - `.../resources/export-templates/scorm12/css/blocks.css`
  - `.../resources/export-templates/scorm2004/css/blocks.css`
- If JS required for timeline/step variants:
  - add under both template JS folders
  - ensure lesson template includes script references for both versions
  - namespace functions/events by block type (`window.scormPhase2...` or module closure) to avoid collisions.

---

## 5. Media handling rules for Phase 2

| Block type | File IDs to extract | Fallback behavior |
|-----------|---------------------|-------------------|
| `IMAGE_GRID` | all image item `fileId`s (array + nested variants) | skip missing images, keep layout stable |
| `QUOTE_WITH_IMAGE` | quote image `fileId` (top-level and nested) | render quote text without image if missing |
| `TIMELINE_VIEW` | image/icon file IDs if schema provides | render text-only step |
| `STEP_FLOW` | image/icon file IDs if schema provides | render text-only step |
| `STEP_MARKER` | optional icon/marker image ID | render marker fallback |
| `ANNOUNCEMENT*` | generally none unless schema has media | render plain announcement block |

**Rule:** use deterministic relative media paths and avoid direct filesystem hints in HTML.

---

## 6. Testing strategy (Phase 2)

| Layer | What to verify |
|------|-----------------|
| Unit (renderers) | valid + partial + malformed JSON for each Phase 2 block; no uncaught exceptions |
| Unit (media extraction) | all expected `fileId`s discovered and deduplicated |
| Regression | existing Phase 0/1 tests still pass |
| Package validation | unzip and assert: lesson HTML + media files + manifest references |
| Manual | one fixture with all Phase 2 blocks in both SCORM versions |

### Must-have assertions from Phase 1 incidents

- No raw escaped HTML where rich text is expected (unless explicitly plain-only field).
- No image paths under `../media/videos/...` for image-based blocks.
- No `Unsupported block type` for Phase 2 blocks in fixture export.
- CSS classes used by renderers exist in both template CSS files.

---

## 7. Definition of done (Phase 2)

- [ ] All Phase 2 blocks render in SCORM 1.2 and 2004 without unsupported fallback.
- [ ] Required media for Phase 2 blocks packaged correctly; missing files degrade gracefully.
- [ ] CSS/JS assets required by Phase 2 shipped in both SCORM template trees.
- [ ] Automated tests and focused smoke tests pass.
- [ ] Manual LMS/SCORM Cloud verification evidence attached.
- [ ] No regressions on Phase 0/1 blocks.

---

## 8. Risks and mitigations

| Risk | Mitigation |
|------|------------|
| JSON shape variability per block | upfront schema matrix + tolerant parser helpers |
| 1.2/2004 behavior drift | single renderer implementation, mirrored template updates in same PR |
| Media folder/type mismatch | centralized media-type normalization and path assertions in tests |
| JS collisions in lesson runtime | namespaced JS, no global generic selectors/handlers |
| CSS regressions | keep block-specific classes and avoid broad selectors |

---

## 9. Suggested timeline (indicative)

| Horizon | Outcome |
|---------|---------|
| **0.5 day** | payload schema spike + fixture readiness |
| **1–1.5 days** | renderers + extraction updates |
| **0.5–1 day** | CSS/JS parity in both templates |
| **0.5 day** | tests + package assertions |
| **0.5 day** | dual-SCORM manual smoke + sign-off |

Total: ~**1 sprint** (or **1–2 sprints** if JS behavior and schema variance are high).

---

## 10. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-04-29 | Initial Phase 2 implementation plan created with explicit Phase 1 lessons learned and CSS/JS parity requirements. |
| 1.1 | 2026-04-30 | Out-of-scope list aligned with phase plan: carousels in **Phase 3**, action buttons in **Phase 4**. |

