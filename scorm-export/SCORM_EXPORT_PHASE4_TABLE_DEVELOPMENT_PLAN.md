# Phase 4 — TABLE block SCORM export (development plan)

**Status:** Implemented in `course-forge-backend` (`TableScormRenderer`, dual SCORM CSS, `JourneyBlockRenderersTest`).

**Goal:** Export `TABLE` as static, LMS-friendly HTML that mirrors course preview behavior, with **SCORM 1.2** and **SCORM 2004** parity (same markup and CSS classes in both template trees).

**Related docs:**

- `SCORM_EXPORT_PHASE4_DEVELOPMENT_PLAN.md` (Phase 4 umbrella).
- `SCORM_EXPORT_SOLUTION_DESIGN.md` (registry + dispatcher pattern).

---

## 1. Scope

### In scope

| Area | Behavior |
|------|----------|
| Payload | Canonical `rows`, `columns`, `data` (2D cell grid); optional `config.hasHeaderRow`, `config.bordered`; legacy `columns` + `rows` without `data` merged like preview (first row from columns, header forced). |
| Cells | String or object with `content` (HTML via `insertHtmlContent`); optional `hAlign` / `vAlign` mapped to inline `style`. |
| Empty grid | Short `lesson-table-empty` note with `role="note"`. |

### Out of scope

- REST or `BlockContentValidator` changes.
- `hasHeaderColumn` from schema unless editor and preview implement it.
- Merged table cells or spreadsheet features.

---

## 2. Preview and backend references

**Preview:** `course-forge-frontend/src/features/editor/components/blocks/preview/blockRenderer.tsx` — `TABLE` / `table` case.

**Validation:** `course-forge-backend/.../BlockContentValidator.java` — `validateTableContent`.

**Example payload:** `BlockController` OpenAPI description for `TABLE`.

---

## 3. Architecture

- **`TableScormRenderer`** — `@Component` implementing `ScormBlockHtmlRenderer` with `supportedType()` = `TABLE`.
- **`ScormBlockHtmlDispatcher`** — resolves renderer before legacy switch (`Scorm12ExportStrategy` / `Scorm2004ExportStrategy`).
- **CSS** — `lesson-table-wrapper`, `lesson-table-wrapper--plain`, `lesson-table`, `lesson-table-empty` in both `export-templates/scorm12/css/blocks.css` and `export-templates/scorm2004/css/blocks.css`.

**Media:** No `extractFileIdsFromBlockContent` change by default; cells are HTML strings without required nested `fileId` in the validator model.

---

## 4. Verification checklist

- [ ] SCORM 1.2 and 2004 packages include `blocks.css` rules for `.lesson-table*`.
- [ ] Lesson with header row shows `<thead>`; `hasHeaderRow: false` shows only `<tbody>`.
- [ ] `bordered: false` adds `lesson-table-wrapper--plain`.
- [ ] Wide tables scroll horizontally (`overflow-x: auto` on wrapper).

---

## 5. Tests

`course-forge-backend/src/test/java/com/mundrisoft/courseforge/export/render/blocks/JourneyBlockRenderersTest.java`:

- Canonical `data` + `hasHeaderRow` + cell align.
- Body-only when `hasHeaderRow` false.
- Legacy `columns` + `rows`.
- Plain wrapper when `bordered` false.
- Empty `data` → empty note.
