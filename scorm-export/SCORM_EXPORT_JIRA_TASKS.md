# Course SCORM export — Jira tasks (5 phases = 5 tickets)

Create **exactly five** issues in Jira. For each ticket: paste **Summary (Jira title)** into the Jira **Summary** field, then **Description** and **Acceptance criteria**.

**References:** `SCORM_EXPORT_REQUIREMENT_ANALYSIS.md`, `SCORM_EXPORT_SOLUTION_DESIGN.md`, `SCORM_EXPORT_PHASE_WISE_DEVELOPMENT_PLAN.md`  
**APIs:** `POST /export/course`, `GET /export/types`, `GET /export/history`

---

## Ticket 1 — Phase 0

**Summary (Jira title):** SCORM export — Phase 0: Safe export foundation + regression tests (SCORM 1.2 & 2004; no new block types)

**Description:**  
Prepare course → SCORM export so later phases can add new lesson block types without breaking what works today. **No** new author-facing features in this phase. Export must match **today** for existing courses. Verify **SCORM 1.2** and **SCORM 2004**.

**Acceptance criteria:**

1. Sample course exports as **SCORM 1.2** and **SCORM 2004**; no regression vs current behavior (lessons + media).  
2. Automated tests in CI; pipeline green.  
3. (Optional) Clearer failure logs with course / lesson / block context; no secrets.

---

## Ticket 2 — Phase 1

**Summary (Jira title):** SCORM export — Phase 1: Lists, headings, spacing, section breaks, banner & resource file blocks

**Description:**  
Export support for: `BULLETED_LIST`, `NUMBERED_LIST`, `CHECKBOX_LIST`, `TEXT_WITH_SUBHEADING`, `SUBHEADING_ONLY`, `SPACING`, `SECTION_BREAK`, `BANNER_IMAGE`, `RESOURCE_FILE` (including packaging files for banner/resource). Both **SCORM 1.2** and **SCORM 2004**.

**Acceptance criteria:**

1. All listed block types render in export (not unsupported placeholder).  
2. Media for banner/resource included in ZIP where applicable.  
3. Tests + dual SCORM smoke on a fixture course.  
4. No regression on block types that already worked before Phase 1.

---

## Ticket 3 — Phase 2

**Summary (Jira title):** SCORM export — Phase 2: Image grid, quote-with-image, timeline, step flow & announcements

**Description:**  
Export support for: `IMAGE_GRID`, `QUOTE_WITH_IMAGE`, `TIMELINE_VIEW`, `STEP_FLOW`, `STEP_MARKER`, `ANNOUNCEMENT`, `ANNOUNCEMENT_NOTE`. (**`CHECKPOINT`** is Phase 4 — it gates progress until prior interactives complete.) Include any needed package assets for **SCORM 1.2** and **SCORM 2004**.

**Acceptance criteria:**

1. All listed block types export and display per product reference.  
2. Media packaged where needed; asset checklist verified for both SCORM versions.  
3. Tests + dual SCORM smoke; no regression on Phase 0–1 blocks on the same fixture.

---

## Ticket 4 — Phase 3

**Summary (Jira title):** SCORM export — Phase 3: Audio, code sample & carousels (SCORM 1.2 & 2004)

**Detailed engineering plan:** `SCORM_EXPORT_PHASE3_DEVELOPMENT_PLAN.md` (work breakdown, Phase 1–2 lessons, dual CSS/JS, extraction, tests).

**Description:**  
Export support for: `AUDIO_PLAYER`, `CODE_EXAMPLE`, `QUOTE_CAROUSEL`, `IMAGE_CAROUSEL`. Packaged audio paths, safe code presentation, carousel markup + media + basic a11y per product rules. Both **SCORM 1.2** and **SCORM 2004**.

**Acceptance criteria:**

1. All listed block types render and behave per agreed spec.  
2. Dual SCORM smoke (1.2 + 2004); tests on a fixture course.  
3. No regression on phases 0–2 on the regression fixture.

---

## Ticket 5 — Phase 4

**Summary (Jira title):** SCORM export — Phase 4: Action buttons, charts, interactives, checkpoint gating, embed, flash & legacy FLASH_CARDS

**Description:**  
Export support for: `ACTION_BUTTON`, `ACTION_BUTTON_GROUP`, `BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT`, `INTERACTIVE_IMAGE`, `DRAG_AND_DROP`, `SORT_AND_LEARN`, **`CHECKPOINT`** (learner cannot proceed until required interactives **before** the checkpoint in the lesson are complete), `EMBED`, `FLASH_CARDS_STACK`. Spike/decide chart rendering approach. **EMBED** requires product-approved security (allowlist / sanitize). Interactives and checkpoint gating must work in LMS **without** calling back to the app. Product decision on legacy **`FLASH_CARDS`**. Both **SCORM 1.2** and **SCORM 2004**.

**Acceptance criteria:**

1. Chart approach documented; action buttons, interactives, embed, and flash stack export per spec (or explicit approved fallback).  
2. **`CHECKPOINT`:** gating verified on a lesson with multiple interactives above the checkpoint; no learner deadlock on failure/degrade paths (product-agreed).  
3. **EMBED** meets signed-off security policy; legacy `FLASH_CARDS` behavior documented and implemented or deferred with approval.  
4. Dual SCORM smoke (1.2 + 2004); QA evidence for high-risk blocks; no regression on phases 0–3 on the regression fixture.

---

*Optional later (not one of the five): parity / performance / ops hardening — track as separate backlog if needed.*
