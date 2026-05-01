# Phase 4 - Development Plan (SCORM course export: hardest blocks)

**Goal:** Ship learner-visible SCORM export support for hardest remaining blocks with safe runtime behavior, security guardrails, and dual-version parity for **SCORM 1.2** and **SCORM 2004**.

Phase 4 scope aligns with:
- `SCORM_EXPORT_REQUIREMENT_ANALYSIS.md` (FR-2/3/4/5/6, NFR-1..5, Phase 4 scope),
- `SCORM_EXPORT_SOLUTION_DESIGN.md` (§6-8, §10-13),
- `SCORM_EXPORT_PHASE_WISE_DEVELOPMENT_PLAN.md` (Phase 4),
- and lessons learned from Phases 1-3 (renderer registry, media extraction parity, JS/CSS packaging, manifest/resource integrity).

---

## 1. Scope

### In scope (Phase 4 block types)

| Group | Block types | Notes |
|------|-------------|-------|
| Action | `ACTION_BUTTON`, `ACTION_BUTTON_GROUP` | Navigation/link behavior must remain LMS-safe and deterministic. |
| Charts | `BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT` | Decide static HTML/SVG vs lightweight JS rendering approach before implementation. |
| Interactives | `INTERACTIVE_IMAGE`, `DRAG_AND_DROP`, `SORT_AND_LEARN` | Requires runtime state and completion signaling hooks. |
| Gating | `CHECKPOINT` | Must gate progression until required prior interactives are complete. |
| Security-sensitive | `EMBED` | Render only from canonical safe payload; no raw HTML injection from author input. |
| Legacy/variant | `FLASH_CARDS_STACK` (+ legacy `FLASH_CARDS`) | Confirm canonical behavior and migration/alias strategy. |

### Out of scope

- Export API contract changes (`POST /export/course` remains unchanged).
- Full frontend authoring redesign.
- DB-breaking migrations.
- New LMS-specific custom adapters beyond current SCORM package model.

---

## 2. Phase 1-3 lessons to enforce in Phase 4

1. **Dual SCORM parity is mandatory**
   - Every CSS/JS/template change must be mirrored under both:
     - `export-templates/scorm12/...`
     - `export-templates/scorm2004/...`
   - No one-sided asset additions.

2. **Asset inclusion has three gates**
   - File exists in template tree.
   - Strategy copies file to workspace.
   - Manifest references final packaged resource.

3. **Media extraction must match render-time path resolution**
   - For every new nested shape (`items`, hotspots, chart assets), extraction + `getLocalMediaPath` must stay consistent.

4. **Progressive enhancement over brittle JS**
   - JS failures must degrade to safe non-breaking UI, not white/blank areas or blocked navigation loops.

5. **Avoid runtime collisions**
   - Namespaced initializers and scoped selectors only (same principle used in `lesson-carousel.js`).

6. **Security-first rendering for embedded/external content**
   - Never render raw author-provided HTML/script directly in exported lesson files.

7. **Checkpoint must not deadlock learners**
   - Define fail-open/fail-safe policy explicitly before coding.

---

## 3. Prerequisites and decisions (must close first)

- [ ] Confirm canonical JSON payloads for each Phase 4 block from real `blocks.content` data.
- [ ] Finalize chart rendering strategy:
  - static/SVG output, or
  - lightweight JS runtime render.
- [ ] Define interactive completion contract (event names + persisted state shape).
- [ ] Define `CHECKPOINT` rule set:
  - which block types are counted as interactive,
  - what counts as completion,
  - behavior when an upstream interactive is invalid/unloadable.
- [ ] Approve `EMBED` security policy for export:
  - `https` requirement,
  - allowlist/sanitized metadata rendering,
  - fallback messaging.
- [ ] Confirm legacy `FLASH_CARDS` behavior:
  - map to `FLASH_CARDS_STACK` or keep separate.

---

## 4. Work breakdown (recommended order)

| Step | Task | Done when |
|------|------|-----------|
| **4.1** | Payload matrix for all Phase 4 types (required/optional keys, legacy variants). | Documented fixtures + parser rules agreed. |
| **4.2** | Implement `EmbedScormRenderer` and secure fallback behavior. | Valid canonical payload renders; invalid payload degrades safely. |
| **4.3** | Implement `ActionButtonScormRenderer` + `ActionButtonGroupScormRenderer` parity in both SCORM versions. | Buttons render and behave per expected link/action policy. |
| **4.4** | Implement chart renderers (`BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT`) using selected strategy. | Output stable offline; no backend runtime dependency. |
| **4.5** | Implement interactive renderers (`INTERACTIVE_IMAGE`, `DRAG_AND_DROP`, `SORT_AND_LEARN`). | UI renders with basic interaction and completion emission. |
| **4.6** | Add/extend JS runtime (`lesson-interactive.js` and/or module files) in both template trees. | Namespaced init, no global collisions, graceful degradation. |
| **4.7** | Implement `CHECKPOINT` renderer + runtime gating wiring. | Next/progression respects completion contract without deadlock. |
| **4.8** | Implement `FLASH_CARDS_STACK` + legacy `FLASH_CARDS` mapping decision. | Behavior documented and tested across legacy content. |
| **4.9** | Extend media extraction for all new nested file references in both strategies. | ZIP contains all referenced assets and no broken links. |
| **4.10** | Add CSS in both SCORM trees for all new block classes/states/fallbacks. | No unstyled blocks in exported package. |
| **4.11** | Register renderers and remove legacy switch cases where fully migrated. | Dispatcher-first path active for Phase 4 types. |
| **4.12** | Automated tests + manual LMS smoke + release notes. | CI green and dual-SCORM manual evidence captured. |

---

## 5. Codebase implementation map

### Renderers

- Add/update under:
  - `course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/render/blocks/`
- Contract:
  - `ScormBlockHtmlRenderer`
  - `ScormBlockRenderContext`
  - helper reuse via `BlockRendererSupport`

### Strategy integration

- Validate in both:
  - `Scorm12ExportStrategy`
  - `Scorm2004ExportStrategy`
- Focus points:
  - `extractFileIdsFromBlockContent(...)`
  - template JS/CSS copy section
  - manifest resources inclusion
  - fallback/legacy branch cleanup

### Template assets

- CSS:
  - `src/main/resources/export-templates/scorm12/css/blocks.css`
  - `src/main/resources/export-templates/scorm2004/css/blocks.css`
- JS (new or extended, namespaced):
  - `.../scorm12/js/*.js`
  - `.../scorm2004/js/*.js`
- lesson template wiring (if new scripts required):
  - `.../scorm12/html/lesson-template.html`
  - `.../scorm2004/html/lesson-template.html`

---

## 6. Block-specific implementation notes

### 6.1 EMBED

- Render from canonical safe fields (`originalUrl`, `embedUrl`, metadata).
- `embedUrl` must be `https`.
- No raw HTML/iframe injection from raw `input`.
- Fallback UI when embed unavailable (safe message + optional source link).

### 6.2 ACTION_BUTTON / ACTION_BUTTON_GROUP

- Support deterministic link actions in export context.
- Guard against invalid destinations; render disabled/non-clickable fallback when invalid.
- Keep keyboard and focus styles accessible.

### 6.3 Charts (`BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT`)

- Use one agreed rendering strategy for all 3 chart types.
- Ensure chart data validation and clear empty-state messaging.
- Avoid heavyweight JS dependencies unless approved.

### 6.4 INTERACTIVE_IMAGE / DRAG_AND_DROP / SORT_AND_LEARN

- Render static shell + runtime enhancement.
- Emit standardized completion events to shared runtime channel.
- Handle invalid content with localized safe fallback per block.

### 6.5 CHECKPOINT

- Observe prior-in-lesson interactive completion state.
- Gate continue/next only when rules pass.
- Define and implement timeout/error behavior to avoid permanent lockouts.

### 6.6 FLASH_CARDS_STACK (+ `FLASH_CARDS`)

- Confirm mapping from legacy content.
- Keep behavior backward-compatible for existing authored lessons.

---

## 7. JS/CSS runtime design constraints

- No backend/API calls from SCORM runtime.
- Namespace all runtime modules and events.
- Initialization must be idempotent.
- Respect reduced-motion where animations are used.
- Provide visible loading/error/empty states.
- Keep controls keyboard reachable.

---

## 8. Media extraction rules (Phase 4)

Extraction must be explicit per block shape and mirrored in both strategies:

- `INTERACTIVE_IMAGE`: base image + hotspot/overlay media IDs.
- `DRAG_AND_DROP`: draggable item media (if schema includes images/icons).
- `SORT_AND_LEARN`: card/item media where present.
- Charts: extract any file-backed assets if schema includes icons/backgrounds.
- `EMBED`: no file extraction by default (URL-based), unless thumbnail packaging is explicitly introduced.

All extracted IDs must be deduplicated and copied with consistent `mediaPathResolver` semantics.

---

## 9. Testing strategy

### Unit tests

- Renderer happy/partial/invalid JSON cases for each block.
- Security-focused tests for `EMBED` URL validation and fallback.
- Completion-state logic tests for checkpoint runtime helpers.

### Integration tests

- Strategy-level export with mixed Phase 0-4 blocks.
- Verify dispatcher + fallback behavior.
- Verify media extraction for nested payloads.

### Package validation

- Unzip and assert:
  - lesson HTML fragments present,
  - JS/CSS assets present in package,
  - manifest references valid resources,
  - no unsupported fallback text for Phase 4 block types under valid payloads.

### Manual LMS smoke

- SCORM 1.2 and 2004 in SCORM Cloud and one target LMS.
- Validate:
  - action buttons,
  - chart rendering,
  - interactive completion,
  - checkpoint gating,
  - embed fallback and allowed embeds.

---

## 10. Definition of done (Phase 4)

- [ ] All Phase 4 block types render in both SCORM versions for canonical payloads.
- [ ] `CHECKPOINT` gating works for required prior interactives without deadlock.
- [ ] `EMBED` rendering follows safe export policy (no raw HTML injection).
- [ ] Required JS/CSS assets copied and wired in both template trees.
- [ ] Media extraction rules implemented and validated where applicable.
- [ ] Regression suite for Phase 0-3 remains green.
- [ ] Manual LMS smoke evidence captured and shared.

---

## 11. Risks and mitigations

| Risk | Mitigation |
|------|------------|
| Runtime complexity causes JS regressions | Keep modular, namespaced scripts and per-block smoke tests. |
| Checkpoint deadlocks learners | Define fail-safe behavior and test broken/partial upstream interactives. |
| Embed security drift | Enforce strict renderer-side URL checks and safe metadata escaping. |
| SCORM 1.2/2004 divergence | Enforce mirrored asset/code changes in same PR. |
| Performance impact from interactive runtime | Keep scripts lightweight; lazy init per visible block. |

---

## 12. Suggested timeline (indicative)

| Horizon | Outcome |
|---------|---------|
| 1-2 days | Payload/decision closure (charts/checkpoint/embed/flash mapping) |
| 3-5 days | EMBED + action + chart renderer tranche |
| 4-6 days | Interactive trio + runtime wiring |
| 2-3 days | Checkpoint gating integration + hardening |
| 1-2 days | Full QA, parity checks, release notes |

---

## 13. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-04-30 | Initial full Phase 4 plan covering hardest blocks, runtime strategy, security, and testing gates. |
