# SCORM Course Export — Phase-wise Development Plan

This plan aligns with `SCORM_EXPORT_REQUIREMENT_ANALYSIS.md` and `SCORM_EXPORT_SOLUTION_DESIGN.md`. It is incremental: no package or class renames, no API contract breaks, existing block behavior preserved until explicitly migrated.

---

## Guiding principles

- Ship in small vertical slices (render + media + tests per slice where applicable).
- **SCORM 1.2 and SCORM 2004** must be verified for each phase that touches rendering or assets.
- Regression: snapshot or integration checks on **already supported** block types before merge.
- Exit each phase with **definition of done** (DoD) below.

### Definition of done (every phase)

- [ ] New scope covered by automated tests (unit and/or integration as agreed).
- [ ] Manual smoke: export one fixture course for **1.2** and **2004**, open in LMS or SCORM Cloud.
- [ ] No change to public export API contract.
- [ ] Release notes / internal changelog entry for supported blocks (if new blocks ship).

---

## Phase 0 — Foundation (no learner-visible change)

**Goal:** Safe extension points and routing without changing output for existing blocks.

| Area | Activities |
|------|------------|
| Rendering | Introduce registry-first dispatch with fallback to **existing** switch-case for all types today. |
| Media | Introduce optional per-block media extraction hook; default path = **current** extraction only. |
| Observability | Structured logs: export id, course, module, lesson, block id, block type, phase (optional). |
| Tests | Tests for: registry wiring, duplicate registration failure, “unknown type still uses legacy switch”. |

**Exit:** Same HTML for current supported blocks; export still succeeds end-to-end.

---

## Phase 1 — Low-risk content blocks + critical media

**Goal:** High authoring value, low interaction complexity; extend media rules where files are obvious.

**Blocks (priority order within phase):**

1. `BULLETED_LIST`, `NUMBERED_LIST`, `CHECKBOX_LIST`
2. `TEXT_WITH_SUBHEADING`, `SUBHEADING_ONLY`
3. `SPACING`, `SECTION_BREAK`
4. `BANNER_IMAGE`, `RESOURCE_FILE` (include media packaging rules)

**Activities:**

- Implement handlers/renderers for each type above.
- Add media extraction for `BANNER_IMAGE` and `RESOURCE_FILE` (and any shared file references used by lists if applicable).
- HTML/CSS aligned with existing export templates.

**Exit:** Phase 1 blocks render correctly in both SCORM versions; no regression on Phase 0 blocks.

---

## Phase 2 — Layout and structured “journey” blocks

**Goal:** Richer layouts and guided learning UI; may need template CSS/JS assets copied with package.

**Blocks:**

- `IMAGE_GRID`, `QUOTE_WITH_IMAGE`
- `TIMELINE_VIEW`, `STEP_FLOW`, `STEP_MARKER`
- `ANNOUNCEMENT`, `ANNOUNCEMENT_NOTE`

**Note:** `CHECKPOINT` is **not** in this phase — it gates progress until prior interactives complete (SCORM runtime + navigation). It ships in **Phase 4** with other high-runtime blocks.

**Activities:**

- Per-block renderer + content JSON tolerance (optional fields).
- Media extraction for multi-image / quote-with-image.
- Asset checklist: ensure any new static JS/CSS for these blocks is included in **both** SCORM template copies.

**Exit:** Fixture courses covering each Phase 2 block type export and play in LMS; no regression on Phase 0–1.

---

## Phase 3 — Audio, code, and carousels

**Goal:** Audio and code presentation plus **quote/image carousels** (mostly static HTML/CSS + packaged media). **Action buttons** ship in **Phase 4** with navigation/runtime-adjacent work.

**Blocks:**

- `AUDIO_PLAYER`, `CODE_EXAMPLE`
- `QUOTE_CAROUSEL`, `IMAGE_CAROUSEL`

**Activities:**

- `AUDIO_PLAYER`: `<audio>` + packaged file paths; extraction rules.
- `CODE_EXAMPLE`: escaped/safe presentation; optional print-friendly styling.
- Carousels: keyboard focus and basic a11y in export HTML where feasible; media extraction for slide items.

**Exit:** Phase 3 blocks render and behave in both SCORM versions; no regression on Phase 0–2.

**Implementation plan:** `SCORM_EXPORT_PHASE3_DEVELOPMENT_PLAN.md` (work breakdown, Phase 1–2 lessons applied, dual SCORM CSS/JS, extraction, tests, codebase paths).

---

## Phase 4 — Hardest blocks last (action buttons, charts, interactives, embed, flash)

**Goal:** Highest complexity: **navigation actions**, charts, deep interactivity, embed security, and flash variants — bundle LMS/runtime and security work in the **final** content phase.

**Blocks:**

- `ACTION_BUTTON`, `ACTION_BUTTON_GROUP`
- `BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT`
- `INTERACTIVE_IMAGE`, `DRAG_AND_DROP`, `SORT_AND_LEARN`
- `CHECKPOINT` (**gating:** learner cannot proceed until required interactives **above** the checkpoint in the lesson are complete — needs runtime state + SCORM API + navigation control)
- `EMBED`
- `FLASH_CARDS_STACK`
- **Legacy:** confirm and document behavior for `FLASH_CARDS` (canonical mapping vs. dedicated UI).

**Activities:**

- Action buttons: links / behaviors per product rules; align with lesson navigation where applicable.
- Decide chart rendering approach (static SVG vs. lightweight chart lib vs. server-side image — product/engineering decision).
- Interactives: minimal JS, namespaced, no dependency on backend at runtime.
- **Checkpoint gating:** define which block types count as “interactive” for completion; persist progress (e.g. suspend data); integrate with lesson navigation (Next / continue) for **SCORM 1.2** and **2004**; avoid deadlock if an upstream interactive fails.
- `EMBED`: **product-approved** allowlist or sanitization policy before any raw embed ships.
- Align `FLASH_CARDS` with product (same learner experience as grid vs. explicit legacy).
- Error handling: soft-fail to safe placeholder + log if content invalid (policy TBD with product).

**Exit:** Security review for `EMBED` and user-generated HTML paths; LMS smoke on strict browsers; checkpoint gating verified with mixed interactives; all above types supported or explicitly deferred with approved fallback.

---

## Phase 5 (optional) — Consistency and hardening

**Goal:** Reduce drift between 1.2 and 2004 and improve operations.

| Track | Activities |
|-------|------------|
| Parity | Shared rendering for identical block output across versions where manifests differ only. |
| Performance | Deduplicate media IDs; reduce redundant JSON parse per block if measured. |
| Ops | Export SLA metrics, alerting on failure rate (if infra available). |

**Exit:** Documented parity matrix; no functional regression.

---

## Cross-cutting work (parallel, small batches)

| Topic | When |
|-------|------|
| Fixture courses | Add/update after each phase for CI or manual regression. |
| Documentation | Update functional doc + internal “supported blocks” list per release. |
| `FLASH_CARDS` alias | Close in Phase 4 with product sign-off. |

---

## Suggested timeline (indicative)

Adjust to team capacity. Rough order: **Phase 0 (1 sprint) → Phase 1 (1 sprint) → Phase 2 (1–2 sprints) → Phase 3 (1 sprint) → Phase 4 (2+ sprints; hardest blocks) → Phase 5 as needed.**

---

## Risk register (short)

| Risk | Mitigation |
|------|------------|
| LMS variance | SCORM Cloud + one customer LMS per release. |
| `EMBED` abuse | Allowlist/sanitize; feature flag if needed. |
| Chart library weight | Prefer smallest viable approach. |
| Scope creep | Ship phase boundaries; defer “nice to have” to Phase 5. |

---

## Document control

| Version | Date | Notes |
|---------|------|--------|
| 1.0 | 2026-04-29 | Initial phase plan from requirement + solution design. |
| 1.1 | 2026-04-29 | Moved difficult blocks (charts, interactives, embed, flash stack) to **Phase 4**; Phase 3 initially scoped audio, code, and action buttons (see 1.3 for carousel vs. action split). |
| 1.2 | 2026-04-29 | **`CHECKPOINT`** moved from Phase 2 to **Phase 4** — product meaning: gate until prior interactives complete (SCORM runtime + navigation). |
| 1.3 | 2026-04-30 | **`QUOTE_CAROUSEL`**, **`IMAGE_CAROUSEL`** moved to **Phase 3**; **`ACTION_BUTTON`**, **`ACTION_BUTTON_GROUP`** moved to **Phase 4**. |
| 1.4 | 2026-04-30 | Added **`SCORM_EXPORT_PHASE3_DEVELOPMENT_PLAN.md`** and linked it from Phase 3; Phase 3 exit criteria unchanged. |
