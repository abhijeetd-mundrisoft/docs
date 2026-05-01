# Phase 1 — Development Plan (SCORM course export: lists, typography, layout chrome, banner & resource)

**Goal:** Ship learner-visible HTML for **low-risk** block types and ensure **banner** and **resource** assets are packaged in the SCORM ZIP. **SCORM 1.2** and **SCORM 2004** stay aligned; no change to public export API.

**References:** `SCORM_EXPORT_SOLUTION_DESIGN.md` §6–7, `SCORM_EXPORT_PHASE_WISE_DEVELOPMENT_PLAN.md` (Phase 1), `SCORM_EXPORT_JIRA_TASKS.md` (Ticket 2), `SCORM_EXPORT_REQUIREMENT_ANALYSIS.md` §9 Phase 1.

**Depends on:** Phase 0 complete (`ScormBlockHtmlRenderer`, `ScormBlockHtmlDispatcher`, `ScormBlockRenderContext`, `ScormBlockHtmlRendererRegistry`).

---

## 1. Scope

### In scope

| Block types | Notes |
|-------------|--------|
| `BULLETED_LIST`, `NUMBERED_LIST`, `CHECKBOX_LIST` | Semantic lists; checkbox list is presentation-only in export (no LMS form submission unless product says otherwise). |
| `TEXT_WITH_SUBHEADING`, `SUBHEADING_ONLY` | Heading + body or subheading-only; reuse escaping / rich-text rules consistent with `TEXT` / headings. |
| `SPACING`, `SECTION_BREAK` | Non-interactive layout chrome; minimal CSS classes aligned with lesson template. |
| `BANNER_IMAGE`, `RESOURCE_FILE` | Render + **media extraction** so image/resource files appear under package `media/` and resolve in lesson HTML. |

### Out of scope

- Phase 2+ block types (`IMAGE_GRID`, `QUOTE_WITH_IMAGE`, timelines, announcements, etc.).
- **`CHECKPOINT`** and other runtime-gated blocks (Phase 4).
- New REST APIs or changes to `POST /export/course` response contract.
- SCORM **editor** import/preview flows.

---

## 2. Prerequisites

- [ ] Phase 0 merged; fixture course baseline for **TEXT** / **TITLE** / existing blocks unchanged.
- [ ] **Content JSON contract** per block type: spike 0.5–1 day — capture real `blocks.content` JSON from staging or author UI (or export DB samples) for each Phase 1 type; document optional vs required keys in a short internal note (or appendix in solution design).
- [ ] Product rules: **checkbox list** — read-only in export vs. interactive; **resource file** — download link label, open in new tab, icon; **banner** — aspect ratio / responsive behavior at minimum.

---

## 3. Work breakdown (recommended order)

| Step | Task | Done when |
|------|------|-----------|
| **1.1** | **Spike JSON** for all nine types; list edge cases (empty list, single item, missing `fileId`). | Written reference (table or sample JSON files in repo **only if** team approves — otherwise Confluence/wiki). |
| **1.2** | **Shared HTML support** (if needed): small `@Component` or stateless helper (e.g. `ScormExportHtmlSupport`) exposing `escapeHtml`, `insertHtmlContent`, media path resolution — **or** inject only what renderers need without duplicating strategy private methods. | Renderers compile; no copy-paste of large parsing utilities. |
| **1.3** | Implement **`ScormBlockHtmlRenderer`** `@Component` beans (one class per type **or** one class per logical group, e.g. `ListBlocksScormRenderer` covering three list enums — **each** `supportedType()` still unique across beans). | Spring starts; no duplicate `BlockType` registration. |
| **1.4** | **Remove** covered types from `generateBlockHtmlLegacy` **default** path: either add `case` branches that are unreachable when beans exist, or **delete** those cases once renderers are always registered (preferred: delete cases so legacy switch shrinks). | Dispatcher returns renderer HTML; legacy no longer lists Phase 1 types (falls through to unsupported only for not-yet-built types). |
| **1.5** | **Media extraction** for `BANNER_IMAGE` and `RESOURCE_FILE`: extend `extractFileIdsFromBlockContent` in **both** strategies **or** introduce a shared helper used by both (avoid 1.2/2004 drift). Per solution design: banner → image `fileId`; resource → file + optional thumbnail `fileId`. | ZIP contains referenced files; missing file → warn + safe placeholder in HTML (per FR-3 / NFR-3). |
| **1.6** | **Manifest**: confirm copied files are referenced where existing pipeline expects (no new manifest schema unless already required for file types). | Import smoke shows resources load. |
| **1.7** | **CSS**: add minimal rules to **both** SCORM template lesson CSS (1.2 + 2004) for list spacing, section break, spacing, banner, resource link styling — keep specificity low to match existing templates. | Visual parity acceptable to product on fixture. |
| **1.8** | **Tests**: unit tests per renderer (golden HTML fragments or normalized substring asserts); media extraction unit tests for JSON variants; optional thin integration test that export completes with fixture course. | CI green. |
| **1.9** | **Manual**: export fixture **1.2** + **2004**; LMS or SCORM Cloud; verify lists, headings, breaks, banner image, downloadable resource. | Evidence for Jira Ticket 2. |
| **1.10** | **Stakeholder comms**: update internal “supported blocks in course SCORM export” list / release note (`SCORM_EXPORT_TICKETS.md` checklist tick — if team tracks there). | Support/authors informed. |

---

## 4. Implementation notes

### 4.1 Renderer pattern (Phase 0 infrastructure)

- Each renderer: `@Component`, implements `ScormBlockHtmlRenderer`, `supportedType()` returns one enum constant.
- Use `ScormBlockRenderContext.objectMapper()` for JSON; tolerate missing keys; never throw uncaught exceptions — return safe placeholder HTML and log at `warn` if parsing fails (align with accordion/tab patterns in strategies).
- **Shared output:** Prefer identical HTML for 1.2 and 2004 unless manifest-specific markup is required (unlikely for Phase 1).

### 4.2 Lists (`BULLETED_LIST` / `NUMBERED_LIST` / `CHECKBOX_LIST`)

- Map content model to `<ul>`, `<ol>`, and checkbox list as `<ul class="...">` with `<li><input type="checkbox" disabled> …` or purely decorative bullets — **lock with product**.
- Nested lists: if JSON supports nesting, render one level first; defer deep nesting if schema unclear.

### 4.3 `TEXT_WITH_SUBHEADING` / `SUBHEADING_ONLY`

- Mirror app semantics: subheading level (`h3`/`h4`) vs title block (`h2`) — confirm with UI so export hierarchy matches course player.

### 4.4 `SPACING` / `SECTION_BREAK`

- Prefer `<div class="lesson-spacing" aria-hidden="true"></div>` and `<hr class="lesson-section-break" />` (or design tokens from templates). Avoid layout that breaks SCORM player chrome.

### 4.5 `BANNER_IMAGE` / `RESOURCE_FILE`

- Reuse patterns from `IMAGE` / `getLocalMediaPath` where applicable.
- Resource: link `href` to packaged relative path; set `download` attribute only if filename is safe (`sanitizeFilename` if exists).
- Log **exportId** if available in media copy path (may require threading export id into collection — optional improvement; minimum: course/lesson/block from existing logs).

---

## 5. Likely code locations

| Area | Location |
|------|----------|
| HTML renderers | `course-forge-backend/.../export/render/blocks/` — `*ScormRenderer.java` `@Component` classes. |
| Legacy switch trim | `Scorm12ExportStrategy` / `Scorm2004ExportStrategy` — `generateBlockHtmlLegacy` |
| Media IDs | `extractFileIdsFromBlockContent` in **both** strategies, or new shared class called from both. |
| Templates / CSS | `export-templates/scorm12/...`, `export-templates/scorm2004/...` (lesson CSS or shared asset paths — follow existing structure). |

---

## 6. Testing strategy

| Layer | What to prove |
|-------|----------------|
| **Unit** | Each new renderer: valid JSON → expected tags/classes; invalid / empty → non-breaking placeholder. |
| **Unit** | `extractFileIdsFromBlockContent` (or shared helper) collects primary + thumbnail IDs for `RESOURCE_FILE`; banner collects image id. |
| **Regression** | Phase 0 golden tests still pass; existing block types still render via legacy or existing renderers unchanged. |
| **Manual** | One course containing all Phase 1 block types; SCORM Cloud + one target LMS. |

---

## 7. Definition of done (Phase 1)

- [ ] All nine block types render **without** “Unsupported block type” in normal authoring scenarios.
- [ ] `BANNER_IMAGE` and `RESOURCE_FILE` media present in package when `fileId`s are valid; warnings + degraded UI when missing.
- [ ] Automated tests merged; CI green.
- [ ] Dual SCORM manual smoke documented (Ticket 2 AC).
- [ ] No regression on pre–Phase 1 supported blocks on the regression fixture.

---

## 8. Risks and mitigations

| Risk | Mitigation |
|------|------------|
| **JSON schema drift** vs author UI | Spike real payloads early; version-tolerant parsing. |
| **1.2 / 2004 drift** in extraction or CSS | Single shared helper for file IDs; shared renderer classes. |
| **Checkbox list** semantics | Product decision documented in spike output. |
| **Resource security** | Links point only to packaged files; no arbitrary external URLs from `RESOURCE_FILE` unless schema allows — validate with security if URL-type resources exist. |

---

## 9. Suggested timeline (indicative)

| Horizon | Outcome |
|---------|---------|
| **0.5–1 dev-day** | JSON spike + list + spacing/section renderers + tests. |
| **0.5–1 dev-day** | Subheading blocks + banner/resource renderers. |
| **0.5 dev-day** | Media extraction + manifest/ZIP verification + CSS. |
| **0.5 dev-day** | Dual SCORM QA, docs, merge. |

**Total:** ~**1–2 sprints** depending on JSON complexity and QA round-trips (aligns with phase plan “~1 sprint” at full velocity).

---

## 10. Document control

| Version | Date | Notes |
|---------|------|--------|
| 1.0 | 2026-04-29 | Initial Phase 1 development plan. |
