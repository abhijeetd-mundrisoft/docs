# Phase 4 — INTERACTIVE_IMAGE SCORM export (development plan)

**Status:** Implemented in `course-forge-backend` (registry renderer, dual SCORM CSS, media ZIP extraction in both strategies, unit tests).

**Goal:** Export `INTERACTIVE_IMAGE` (labeled graphic) as static HTML: background image, percent-positioned pins, in-page marker panels (anchor navigation), optional marker image and https-only embed — with **SCORM 1.2** and **SCORM 2004** CSS parity and correct media ZIP extraction.

## Frontend reference

- Preview: [`course-forge-frontend/src/features/editor/components/blocks/preview/labeled-graphics/LabeledGraphicPreview.tsx`](../../course-forge-frontend/src/features/editor/components/blocks/preview/labeled-graphics/LabeledGraphicPreview.tsx) (`labeled-graphics` UI type; API stores `INTERACTIVE_IMAGE`).

## Backend contract

- [`course-forge-backend/src/main/java/com/mundrisoft/courseforge/service/BlockContentValidator.java`](../../course-forge-backend/src/main/java/com/mundrisoft/courseforge/service/BlockContentValidator.java)

## SCORM / LMS standards (locked)

- **`description`:** `insertHtmlContent()` (same trust model as other rich blocks).
- **Marker embed:** `https` iframe only; otherwise fallback message (align with `EmbedScormRenderer` policy).
- **Interaction:** pins are `<a href="#panel-id">`; panels below stage; no required JS.
- **Dual parity:** same CSS in `export-templates/scorm12/css/blocks.css` and `export-templates/scorm2004/css/blocks.css`.

## Implementation map (delivered)

| Item | Location |
|------|----------|
| Renderer | [`course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/render/blocks/InteractiveImageScormRenderer.java`](../../course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/render/blocks/InteractiveImageScormRenderer.java) |
| Media ZIP `fileId` extraction | [`Scorm12ExportStrategy.java`](../../course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/strategy/Scorm12ExportStrategy.java), [`Scorm2004ExportStrategy.java`](../../course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/strategy/Scorm2004ExportStrategy.java) — `extractFileIdsFromBlockContent` branch for `INTERACTIVE_IMAGE` |
| CSS (dual tree) | [`export-templates/scorm12/css/blocks.css`](../../course-forge-backend/src/main/resources/export-templates/scorm12/css/blocks.css), [`export-templates/scorm2004/css/blocks.css`](../../course-forge-backend/src/main/resources/export-templates/scorm2004/css/blocks.css) |
| Tests | [`JourneyBlockRenderersTest.java`](../../course-forge-backend/src/test/java/com/mundrisoft/courseforge/export/render/blocks/JourneyBlockRenderersTest.java) |

## Runtime behavior (summary)

- Background from `backgroundImage` (fallback `image`) via `mediaPathResolver` (`images`).
- Pins are in-page anchors to `section` panels below the stage (`#lesson-interactive-marker-{blockNumber}-{id}`).
- Marker `description`: `insertHtmlContent`; marker `media` `image` / `embed` (`https` iframe only, else missing panel).
