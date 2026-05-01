# SCORM Course Export — Task list (tracker-ready)

One line per task. Copy into your tool as separate issues/tickets.

**Jira (5 phase tickets, Summary / Description / AC):** `SCORM_EXPORT_JIRA_TASKS.md` — this file stays the **detailed checklist** only.

---

## Phase 0 (foundation — one parent ticket)

- Course SCORM export — Foundation: internals + tests + dual SCORM smoke (no new block types yet).
  - Sub-task: lesson HTML safe new/old path (SCORM 1.2 + 2004).
  - Sub-task: media ZIP safe new/old path (SCORM 1.2 + 2004).
  - Sub-task: tests in CI (including existing block types / legacy path).
  - Sub-task (optional): structured logs on failure; no secrets.
  - Sub-task: QA — LMS or SCORM Cloud import evidence (1.2 + 2004).

---

## Phase 1

- Export: `BULLETED_LIST`, `NUMBERED_LIST`, `CHECKBOX_LIST`.
- Export: `TEXT_WITH_SUBHEADING`, `SUBHEADING_ONLY`.
- Export: `SPACING`, `SECTION_BREAK`.
- Export: `BANNER_IMAGE` and `RESOURCE_FILE` (render + package media).
- Tests + dual SCORM smoke for Phase 1 blocks.
- Update supported-blocks / release note for authors or support.

---

## Phase 2

- Export: `IMAGE_GRID`, `QUOTE_WITH_IMAGE` (+ media where needed).
- Export: `TIMELINE_VIEW`, `STEP_FLOW`, `STEP_MARKER`.
- Export: `ANNOUNCEMENT`, `ANNOUNCEMENT_NOTE`.
- Verify any new CSS/JS is present in both SCORM 1.2 and 2004 packages.
- Tests + dual SCORM smoke for Phase 2 blocks.

---

## Phase 3

Engineering breakdown: `SCORM_EXPORT_PHASE3_DEVELOPMENT_PLAN.md`.

- Export: `AUDIO_PLAYER`, `CODE_EXAMPLE`.
- Export: `QUOTE_CAROUSEL`, `IMAGE_CAROUSEL`.
- Tests + dual SCORM smoke for Phase 3 blocks.

---

## Phase 4 (hardest blocks last)

- Spike: chart rendering approach for `BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT`.
- Export: `ACTION_BUTTON`, `ACTION_BUTTON_GROUP`.
- Export: chart blocks per spike decision (or document approved fallback).
- Export: `INTERACTIVE_IMAGE`, `DRAG_AND_DROP`, `SORT_AND_LEARN`.
- Export: `CHECKPOINT` (gate until prior interactives in lesson complete; SCORM runtime + navigation).
- Product + security: `EMBED` policy (allowlist / sanitize); then implement.
- Export: `FLASH_CARDS_STACK`; product decision on legacy `FLASH_CARDS` behavior.
- Agree invalid JSON / missing media behavior for interactive and embed blocks.
- Tests + dual SCORM smoke + security sign-off for embed and high-interaction blocks.

---

## Phase 5 (optional)

- Parity matrix: learner-visible export for 1.2 vs 2004; fix or document gaps.
- Performance pass on large courses (measure before/after).
- Ops: export failure visibility (if monitoring stack available).

---

## Cross-cutting

- Maintain fixture course(s) for export regression (manual or CI).
- After each phase merge: refresh “supported blocks in course SCORM export” for stakeholders.

