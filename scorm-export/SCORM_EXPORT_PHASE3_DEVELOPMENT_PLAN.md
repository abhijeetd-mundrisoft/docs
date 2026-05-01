# Phase 3 — Development Plan (SCORM course export: audio, code, carousels)

**Goal:** Ship learner-visible HTML for **`AUDIO_PLAYER`**, **`CODE_EXAMPLE`**, **`QUOTE_CAROUSEL`**, and **`IMAGE_CAROUSEL`** in both **SCORM 1.2** and **SCORM 2004**, with media packaged under the export ZIP and offline-safe behavior. Align with **FR-2**, **FR-3**, **FR-4** (where carousels need light JS), and **NFR-1–NFR-5** in `SCORM_EXPORT_REQUIREMENT_ANALYSIS.md`, and with renderer/media patterns in `SCORM_EXPORT_SOLUTION_DESIGN.md` (§6–8, §12 Phase 3, incremental extraction).

**References:** `SCORM_EXPORT_REQUIREMENT_ANALYSIS.md` §6–9 (Phase 3), `SCORM_EXPORT_SOLUTION_DESIGN.md` (renderer context, media extraction, §12 Phase 3), `SCORM_EXPORT_PHASE_WISE_DEVELOPMENT_PLAN.md` (Phase 3 goals, activities, exit), `SCORM_EXPORT_PHASE1_DEVELOPMENT_PLAN.md`, `SCORM_EXPORT_PHASE2_DEVELOPMENT_PLAN.md`.

**Depends on:** Phase 0–2 complete: `ScormBlockHtmlDispatcher`, `ScormBlockHtmlRenderer` registry, `ScormBlockRenderContext`, renderers under `export/render/blocks/`, shared `BlockRendererSupport`, dual-strategy `extractFileIdsFromBlockContent` extensions, dual `export-templates/scorm12` and `scorm2004` trees.

---

## 1. Scope

### In scope (Phase 3 block types)

| Group | Block types | Notes |
|------|-------------|--------|
| Media | `AUDIO_PLAYER` | `<audio controls>` (or product-agreed controls); audio `fileId` → packaged path; correct **media type** (`audio` / `audios` alias) — same class of bug as Phase 2 images landing under `videos/`. |
| Content | `CODE_EXAMPLE` | Safe presentation: escaped or sanitized; optional `<pre><code>` + language class; **no** `eval` / inline script from author content. |
| Carousels | `QUOTE_CAROUSEL`, `IMAGE_CAROUSEL` | Static-first HTML; **minimal namespaced JS** for prev/next and focus if product requires more than CSS-only; all slide media extracted. |

### Out of scope

- **`ACTION_BUTTON`**, **`ACTION_BUTTON_GROUP`** — **Phase 4** (navigation / runtime-adjacent; see phase-wise plan).
- Charts, interactives, checkpoint, embed, flash — **Phase 4**.
- New REST APIs or `POST /export/course` contract changes.
- SCORM editor import/preview (unless explicitly in scope elsewhere).

---

## 2. Lessons from Phase 1 & Phase 2 (carry forward — mandatory)

These items come from **actual Phase 1/2 implementation and smoke issues**; Phase 3 must treat them as non-negotiable.

1. **JSON schema drift**  
   Real payloads vary: nested objects (`image`, `items`, `quotes`), array roots, alternate keys. Use `BlockRendererSupport` patterns (`parseMap`, `parseList`, `readItems`, `string`, tolerant loops). **Canonical shapes** for Phase 3 blocks are captured in **Appendix A** (production API sample); still allow tolerant fallbacks for older or partial payloads.

2. **Media type and folder aliases**  
   Wrong resolver category (e.g. image → `videos/`) breaks LMS playback. Reuse the same **`mediaPathResolver`** / `getLocalMediaPath` normalization as Phase 2; add **unit tests** for `AUDIO_PLAYER` and each carousel slide using **`audio`** (or agreed alias) vs **`images`**.

3. **Dual SCORM tree parity**  
   Every change to **`blocks.css`**, new **JS**, or lesson **template** references must land in **both** `export-templates/scorm12/...` and `export-templates/scorm2004/...` in the **same PR**. Drift here caused repeated defects in Phase 2.

4. **CSS/JS are deliverables**  
   Do not merge renderer-only PRs without styles. For carousels, plan **keyboard** (focus trap / roving tabindex per phase-wise activity), visible focus ring, and `aria-live` / labels if slides change.

5. **LMS font and character coverage**  
   Phase 2 showed decorative Unicode (e.g. typographic quotes) can render as **replacement glyphs** in embedded players. Prefer **ASCII + CSS**, **HTML entities**, or **inline SVG** for critical icons when carousels use decorative marks.

6. **Rich vs escaped HTML**  
   Use `BlockRendererSupport.richOrEscaped` (or `insertHtmlContent` / `escapeHtml` via context) consistently; avoid double-escaping subheadings or stripping HTML where the app stores rich text.

7. **Ordering and visibility**  
   Phase 2 `STEP_FLOW` introduced **`orderIndex`** sorting and **`isHidden`** omission. Carousels may expose `orderIndex` or `isHidden` on slides — mirror the same rules if the product JSON includes them.

8. **Inline vs stylesheet behavior**  
   Where dynamic values (e.g. opacity) must override template CSS, prefer **computed safe inline styles** built in Java (no author-controlled CSS strings) or narrowly scoped classes — avoid overriding entire hero surfaces with wrong defaults (see overlay tint lessons from journey blocks).

9. **Legacy switch shrink**  
   After each renderer is registered, remove the corresponding **`generateBlockHtmlLegacy`** branches in **`Scorm12ExportStrategy`** and **`Scorm2004ExportStrategy`** so dispatch stays single-path for shipped types.

10. **Manifest and paths**  
    Lesson HTML must use **package-relative** media URLs consistent with existing blocks; no backend URLs at runtime (**FR-4**, **FR-5**).

---

## 3. Prerequisites

- [ ] Phase 2 merged; regression fixture still exports Phase 0–2 blocks green in CI.
- [ ] **Payload spike (0.5–1 day):** real `blocks.content` for all four Phase 3 types from staging/DB — **baseline documented in Appendix A** (2026-04-30 API sample). Extend matrix if new keys appear (`slides` vs `items`, etc.).
- [ ] **Product decisions:** `AUDIO_PLAYER` — default to muted or not; transcript attachment if any; `CODE_EXAMPLE` — syntax highlight yes/no (if yes, smallest static approach); carousels — autoplay allowed in SCORM or forbidden; loop; swipe on touch.

---

## 4. Work breakdown (recommended order)

| Step | Task | Done when |
|------|------|-----------|
| **3.1** | Payload schema matrix (audio fields, code fields, carousel slide list shapes, nested `fileId`s). | Table or ticket appendix + 1–2 golden JSON fixtures (repo only if approved). |
| **3.2** | Implement **`AudioPlayerScormRenderer`** (`@Component`, `ScormBlockHtmlRenderer`). | Valid JSON → `<audio>` + correct `src`; missing file → placeholder + no throw. |
| **3.3** | Implement **`CodeExampleScormRenderer`**. | Code escaped/safe; language class optional; long lines wrap without breaking layout. |
| **3.4** | Implement **`QuoteCarouselScormRenderer`** and **`ImageCarouselScormRenderer`** (or one parameterized class **only if** `supportedType()` uniqueness is preserved per bean). | First slide visible; others reachable via controls; a11y baseline agreed with product. |
| **3.5** | **Media extraction:** extend `extractFileIdsFromBlockContent` in **`Scorm12ExportStrategy`** and **`Scorm2004ExportStrategy`** for all four types (audio id, carousel slide ids, nested maps). Prefer shared private helper if logic duplicates. | ZIP contains referenced audio/images; tests assert ids collected. |
| **3.6** | **CSS:** `blocks.css` in **both** `scorm12` and `scorm2004` — player chrome, code block, carousel layout, focus states, reduced-motion friendly transitions. | Manual package review shows no unstyled blocks. |
| **3.7** | **JS (if required):** e.g. `lesson-carousel.js` under both `.../js/` trees (avoid **phase-prefixed** asset names in shipped packages); **namespaced** init (IIFE + scoped selectors on `.scorm-carousel`). | No global collisions with `tabs.js`, `flashcards.js`, `quiz.js`, `simple-navigation.js`. |
| **3.8** | **Lesson template:** if new scripts added, update **`html/lesson-template.html`** in **both** SCORM versions (mirror existing `<script src="../js/...">` order — load after core navigation if dependencies matter). | Unzip export; carousel works without 404 script. |
| **3.9** | **Registry + legacy:** register beans; remove legacy `case` branches for Phase 3 types from both strategies. | Dispatcher-only path for these enums. |
| **3.10** | **Tests:** renderer unit tests (valid / partial / bad JSON); extraction tests; optional thin export integration test. | CI green. |
| **3.11** | **Manual:** export fixture **1.2** + **2004**; SCORM Cloud + one target LMS — keyboard, audio play, code readability, carousel. | Evidence for Jira / internal QA. |
| **3.12** | **Comms:** update supported-blocks list / `SCORM_EXPORT_TICKETS.md` / release note. | Stakeholders informed. |

---

## 5. Implementation guidance (codebase-specific)

### 5.1 Renderer placement

- Add new classes under:  
  `course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/render/blocks/`  
- Package: `com.mundrisoft.courseforge.export.render.blocks`  
- One Spring `@Component` per `BlockType` (or strictly one `supportedType()` per bean if splitting implementations).

### 5.2 Context and helpers

- Inject **`ScormBlockRenderContext`**: `escapeHtml`, `insertHtmlContent`, `mediaPathResolver`, `objectMapper`.  
- Reuse **`BlockRendererSupport`** for JSON maps/lists and string extraction; extend **only** if Phase 3 needs shared helpers (e.g. slide iteration) — keep additions small and tested.

### 5.3 Strategy integration

- **`Scorm12ExportStrategy`** / **`Scorm2004ExportStrategy`**:  
  - `extractFileIdsFromBlockContent(BlockType, contentJson)` — add branches or shared collector for `AUDIO_PLAYER`, `CODE_EXAMPLE`, `QUOTE_CAROUSEL`, `IMAGE_CAROUSEL`.  
  - `generateBlockHtmlLegacy` — remove completed `case`s when renderers ship.  
- Keep **`ScormBlockHtmlDispatcher`** as first hop before legacy fallback.

### 5.4 Templates and assets

| Asset | Paths (update **both** trees) |
|--------|-------------------------------|
| Block styles | `src/main/resources/export-templates/scorm12/css/blocks.css` and `.../scorm2004/css/blocks.css` |
| Block behavior JS | `.../scorm12/js/*.js` and `.../scorm2004/js/*.js` |
| Lesson shell | `.../scorm12/html/lesson-template.html` and `.../scorm2004/html/lesson-template.html` |

### 5.5 Carousel JS design constraints

- **No network calls** to Course Forge API at runtime.  
- Init by **querySelectorAll** scoped to `.lesson-quote-carousel`, `.lesson-image-carousel` (or agreed BEM) to avoid touching unrelated DOM.  
- Prefer **progressive enhancement**: if JS fails, first slide + “static stack” or links still usable where product allows.

---

## 6. Media handling rules (Phase 3)

| Block type | File IDs to extract (confirmed keys — see Appendix A) | Fallback |
|------------|--------------------------------------------------------|----------|
| `AUDIO_PLAYER` | Top-level `fileId` (audio asset). `title` / `caption` are text only. | Placeholder + log if `fileId` missing (**FR-3**, **FR-6**). |
| `CODE_EXAMPLE` | None in current schema (`code` string only). | N/A |
| `QUOTE_CAROUSEL` | For each object in `quotes[]`: `image.fileId` when `image` is a map. | Skip slide without valid image; keep carousel shell. |
| `IMAGE_CAROUSEL` | For each object in `items[]`: `fileId`. Sort slides by `orderIndex` when rendering if present. | Skip item without `fileId`; preserve order. |

API may include `url` (app absolute URL); **export HTML must use** `mediaPathResolver(fileId, …)` **only**, never embed authoring `http://…/api/files/…` links in the package.

See also `SCORM_EXPORT_SOLUTION_DESIGN.md` §7 extraction bullets for Phase 3.

---

## 7. Testing strategy

| Layer | What to prove |
|-------|----------------|
| **Unit (renderers)** | Valid, minimal, malformed, and empty JSON for each type; no uncaught exceptions; stable class names for CSS hooks. |
| **Unit (extraction)** | All expected `fileId`s collected; **audio** uses audio media folder mapping in path assertions where tests resolve paths. |
| **Regression** | Phase 0–2 renderer and extraction tests remain green. |
| **Package** | Unzipped export: lesson references exist; media files on disk; no `Unsupported block type` for Phase 3 types on fixture. |
| **Manual** | Keyboard-only navigation on carousels; audio plays in strict LMS; code block readable and copy-friendly. |

---

## 8. Definition of done (Phase 3)

- [ ] All four block types render in SCORM 1.2 and 2004 without unsupported fallback for normal authoring payloads.
- [ ] Required media packaged; missing media degrades gracefully with logging (**FR-3**, **FR-6**).
- [ ] CSS complete in **both** template trees; JS (if any) namespaced, duplicated, and referenced from both lesson templates.
- [ ] Automated tests merged; CI green.
- [ ] Dual SCORM manual smoke documented.
- [ ] No regression on Phase 0–2 blocks on the regression fixture.

---

## 9. Risks and mitigations

| Risk | Mitigation |
|------|------------|
| Carousel a11y gaps | Keyboard + focus + `aria-roledescription` / labels per WCAG targets; product sign-off on “minimal” vs “full” behavior. |
| Audio codec / LMS variance | Test MP3 (widely supported); document unsupported format fallback. |
| Code block XSS | Escape by default; never reflect raw author strings into script context. |
| JS collisions | Namespace + scoped selectors; avoid `document.querySelector` without root. |
| Scope creep (syntax highlight lib) | Defer to Phase 5 or Phase 4 only with explicit approval and weight budget. |

---

## 10. Suggested timeline (indicative)

| Horizon | Outcome |
|---------|---------|
| **0.5–1 day** | Payload spike + product decisions |
| **1–2 days** | Four renderers + extraction + tests |
| **0.5–1 day** | Dual CSS + optional JS + template wiring |
| **0.5 day** | Dual SCORM manual smoke + sign-off |

*Adjust for carousel complexity if product requires swipe, autoplay, or sync with sidebar.*

---

## 11. Likely code locations (quick map)

| Area | Location |
|------|----------|
| Renderers | `course-forge-backend/.../export/render/blocks/*ScormRenderer.java` |
| Shared parsing | `.../export/render/blocks/BlockRendererSupport.java` |
| Dispatch | `.../export/render/ScormBlockHtmlDispatcher.java`, `ScormBlockHtmlRendererRegistry.java` |
| Media + legacy HTML | `.../export/strategy/Scorm12ExportStrategy.java`, `Scorm2004ExportStrategy.java` |
| Templates | `course-forge-backend/src/main/resources/export-templates/scorm{12,2004}/` |

---

## Appendix A — Real `content` JSON (API sample, 2026-04-30)

Shapes below are from a live **blocks list** response (`type` + `content` string). Implementations should parse the **inner** JSON from `block.content`.

### A.1 `AUDIO_PLAYER`

```json
{
  "title": "aasd",
  "fileId": "7e60f06b-1ccb-4340-b6ca-3b1e3896952a",
  "caption": "aaa"
}
```

| Field | Role |
|-------|------|
| `fileId` | **Required** for packaging and `<audio src>`. |
| `title` | Optional heading / label above player. |
| `caption` | Optional short description (plain or rich per app; render safely). |

### A.2 `CODE_EXAMPLE`

```json
{
  "code": "<!DOCTYPE html>\\n<html>\\n..."
}
```

| Field | Role |
|-------|------|
| `code` | **Required** body; string may contain angle brackets and quotes — **must be escaped** (or safely encoded) for `<pre><code>`; never parse as HTML in the LMS. |

No `language` / `fileId` in this sample; add tolerant reads if schema grows.

### A.3 `IMAGE_CAROUSEL`

```json
{
  "items": [
    {
      "url": "http://localhost:8080/api/files/70da69b9-9a4d-4734-86ae-80ff3cf12610",
      "fileId": "70da69b9-9a4d-4734-86ae-80ff3cf12610",
      "altText": "",
      "caption": "",
      "orderIndex": 1
    }
  ]
}
```

| Field | Role |
|-------|------|
| `items` | Array of slides; **primary** key in sample. |
| `items[].fileId` | **Packaging + `img src`.** |
| `items[].url` | App convenience; **ignore for SCORM** `href`/`src` (offline + auth). |
| `items[].orderIndex` | Sort ascending before render when all slides have it. |
| `items[].altText`, `items[].caption` | Accessibility / caption text. |

Tolerant parsers may also accept `slides` or a root array if older payloads exist.

### A.4 `QUOTE_CAROUSEL`

```json
{
  "quotes": [
    {
      "text": "all things are wild and free",
      "image": { "fileId": "819750d8-0b6c-46bb-8459-fa89165baa5d", "altText": "" },
      "author": "abhi"
    }
  ]
}
```

| Field | Role |
|-------|------|
| `quotes` | Array of slides; **primary** key in sample (not `items`). |
| `quotes[].text` | Quote body (plain in sample; use `richOrEscaped` if HTML appears). |
| `quotes[].author` | Attribution line. |
| `quotes[].image` | Map with `fileId` for background/side image per slide. |

### A.5 Same lesson, out of Phase 3 scope (reference only)

The same API response may include **`QUOTE`** (single quote + image) — not part of Phase 3 export scope here, but useful for validator/renderer parity elsewhere:

```json
{
  "text": " Life is like riding a bicycle...",
  "image": { "fileId": "7c520310-210b-4c8f-93aa-8049d0b9d6ac", "altText": "" },
  "author": "abhi"
}
```

---

## 12. Document control

| Version | Date | Notes |
|---------|------|--------|
| 1.0 | 2026-04-30 | Initial Phase 3 plan: consolidates Phase 1/2 lessons, codebase paths (`blocks` package), dual template/CSS/JS, extraction and testing; scope per phase-wise plan (audio, code, carousels; actions in Phase 4). |
| 1.1 | 2026-04-30 | **Appendix A:** real API `content` JSON for `AUDIO_PLAYER`, `CODE_EXAMPLE`, `IMAGE_CAROUSEL`, `QUOTE_CAROUSEL`; media table + solution design extraction lines aligned; `url` vs `fileId` export rule documented. |
| 1.2 | 2026-04-30 | Carousel script shipped as **`lesson-carousel.js`** (not phase-prefixed); work breakdown **3.7** updated accordingly. |
