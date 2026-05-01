# Phase 4 - Flash Cards Stack Development Plan (SCORM Export)

**Scope:** Implement `FLASH_CARDS_STACK` export support with explicit legacy mapping for `FLASH_CARDS`, while preserving existing `FLASH_CARDS_GRID` behavior in SCORM 1.2 and 2004.

This plan is based on:
- `docs/scorm-export/SCORM_EXPORT_PHASE4_DEVELOPMENT_PLAN.md`
- existing SCORM flashcard runtime (`export-templates/scorm12/js/flashcards.js`, `scorm2004/js/flashcards.js`)
- existing legacy export path for `FLASH_CARDS_GRID` in strategy classes
- backend schema/validator support in `BlockSchemaUtil` and `BlockContentValidator`

---

## 1) Goals

- Add learner-visible export support for `FLASH_CARDS_STACK`.
- Keep backward compatibility for legacy `FLASH_CARDS`.
- Reuse one runtime behavior for all flashcard variants.
- Avoid breaking existing `FLASH_CARDS_GRID` exports.

---

## 2) Current State (as-is)

- `FLASH_CARDS_GRID` already renders via legacy strategy method (`generateFlashCardsBlockContent`).
- Runtime JS for click-to-flip already exists and is packaged (`flashcards.js`).
- CSS classes for flashcards exist in both SCORM template trees.
- Schema docs already define:
  - `FLASH_CARDS_GRID` content array of `{ front, back, orderIndex }`
  - `FLASH_CARDS_STACK` same schema
  - legacy alias mention for `FLASH_CARDS`.
- Frontend stack editor commonly stores cards as:
  - `content.cards[]` (object shape), or
  - array content in legacy flows.

---

## 3) Contract and Mapping Rules

## Canonical export input (target)

Array of card objects:
- `front` (required)
- `back` (required)
- `orderIndex` (optional but preferred for deterministic output)

## Accepted payload variants (tolerant parser)

1. Root array:
```json
[
  {"front":"Q1","back":"A1","orderIndex":1}
]
```

2. Object with `cards`:
```json
{
  "cards":[{"front":"Q1","back":"A1"}]
}
```

3. Object with `items` (fallback compatibility):
```json
{
  "items":[{"front":"Q1","back":"A1"}]
}
```

## Type mapping policy

- `FLASH_CARDS_STACK` -> render as stack layout.
- `FLASH_CARDS_GRID` -> keep current grid render behavior.
- `FLASH_CARDS` (legacy alias) -> map to `FLASH_CARDS_GRID` renderer/layout (no contract break).

---

## 4) Rendering Strategy

## Recommended implementation

Introduce renderer-based implementation for:
- `FLASH_CARDS_STACK` (new renderer)
- optionally `FLASH_CARDS_GRID` (migrate legacy switch path to renderer for consistency)

Both can share one helper that:
- parses variants (`array`, `cards`, `items`)
- sanitizes/escapes card content using existing context helpers
- sorts by `orderIndex` when present
- applies card cap (e.g., max 20, aligned with editor limits)

## HTML output expectations

- Stack container class (e.g., `.lesson-flashcards-stack`) for stacked visual.
- Card class hooks reused where possible (`.flashcard-card`, `.flashcard-inner`, `.flashcard-front`, `.flashcard-back`) so existing `flashcards.js` can still flip cards.
- Graceful empty state message if no valid cards.

---

## 5) JS/CSS Asset Strategy

- **JS:** reuse existing `flashcards.js` for click-to-flip (no new runtime file needed initially).
- **CSS:** add stack-specific layout classes in both:
  - `export-templates/scorm12/css/blocks.css`
  - `export-templates/scorm2004/css/blocks.css`
- Ensure non-breaking style coexistence with existing grid flashcards.

Optional enhancement (later): keyboard and ARIA improvements in `flashcards.js` for stack order navigation.

---

## 6) Code Touchpoints

### Backend renderers
- `course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/render/blocks/`
  - add `FlashCardsStackScormRenderer`
  - optional: `FlashCardsGridScormRenderer` (migration from legacy)

### Strategy classes
- `Scorm12ExportStrategy`
- `Scorm2004ExportStrategy`

Tasks:
- ensure dispatcher routes `FLASH_CARDS_STACK`.
- keep/remove legacy switch branch based on migration decision.

### Templates
- `src/main/resources/export-templates/scorm12/css/blocks.css`
- `src/main/resources/export-templates/scorm2004/css/blocks.css`
- verify `flashcards.js` copy + manifest references remain present.

---

## 7) Validation and Error Handling

- Skip invalid cards (missing both `front`/`back`) rather than fail whole export.
- If all cards invalid -> render safe empty-state panel.
- Never throw export-level exception due to one malformed flashcard block.

---

## 8) Test Plan

## Unit (renderer)
- `FLASH_CARDS_STACK` with `content.cards` shape renders cards.
- `FLASH_CARDS_STACK` with root array renders cards.
- ordering by `orderIndex`.
- empty/invalid content shows fallback message.

## Regression
- Existing `FLASH_CARDS_GRID` output remains stable.
- Existing flashcards runtime JS still flips stack cards.

## Integration
- Export fixture lesson containing:
  - one `FLASH_CARDS_GRID`
  - one `FLASH_CARDS_STACK`
  - one legacy `FLASH_CARDS` block (if present in DB fixtures)
- Validate both SCORM versions.

---

## 9) Acceptance Criteria

- `FLASH_CARDS_STACK` renders in SCORM 1.2 and 2004.
- Legacy `FLASH_CARDS` behavior is explicitly documented and stable.
- No `Unsupported block type` for stack blocks.
- Flip interaction works with packaged runtime JS.
- Tests pass with no regression on existing flashcard/grid behavior.

---

## 10) Suggested Delivery Slices

1. **Slice A:** Parser helper + `FLASH_CARDS_STACK` renderer + tests.
2. **Slice B:** Stack CSS in both SCORM templates + visual QA.
3. **Slice C:** Legacy alias verification (`FLASH_CARDS`) + strategy cleanup.
4. **Slice D:** End-to-end SCORM smoke and doc/ticket updates.

---

## 11) Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| Stack and grid CSS conflict | Namespace stack classes; reuse only flip behavior hooks. |
| Mixed payload shapes from older content | Tolerant parser for `array`, `cards`, `items`. |
| Legacy alias ambiguity (`FLASH_CARDS`) | Lock one mapping policy and document it in release notes. |
| Runtime regressions in LMS | Keep JS unchanged initially; do focused smoke on both SCORM versions. |

---

## 12) Document Control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-04-30 | Initial implementation plan for `FLASH_CARDS_STACK` + `FLASH_CARDS` mapping in SCORM export. |
