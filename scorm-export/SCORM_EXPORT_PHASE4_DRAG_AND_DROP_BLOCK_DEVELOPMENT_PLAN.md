# Phase 4 — Standalone `DRAG_AND_DROP` block SCORM export

**Status:** Implemented in `course-forge-backend`.

**Goal:** Export standalone `DRAG_AND_DROP` lesson blocks using the **same DOM and runtime** as quiz drag-and-drop questions (`quiz.js`, `scormManager.trackQuizInteraction`, `lessonScores` / `setQuizScore`), without duplicating scoring logic.

---

## 1. Behavior

| Area | Detail |
|------|--------|
| Payload | Same as quiz DnD: `draggableItems`, `targetItems`, `answerMappings`; optional `instructions` (HTML via `insertHtmlContent`). Default stem when instructions missing. |
| Markup | Outer `lesson-quiz lesson-drag-drop-block` + `quiz-question[data-quiz-type="drag-and-drop"]` + shared activity HTML + `quiz-submit` / `quiz-feedback`. |
| Activity HTML | [`DragAndDropScormHtmlBuilder`](course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/render/blocks/DragAndDropScormHtmlBuilder.java) used by [`DragAndDropBlockScormRenderer`](course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/render/blocks/DragAndDropBlockScormRenderer.java) and by `Scorm12ExportStrategy` / `Scorm2004ExportStrategy` quiz path (`generateDragAndDropQuizQuestionHtml`). |

---

## 2. Course preview note

Standalone `DRAG_AND_DROP` is not wired in [`blockRenderer.tsx`](course-forge-frontend/src/features/editor/components/blocks/preview/blockRenderer.tsx) today; quiz DnD preview lives in [`DragAndDropQuizPreview.tsx`](course-forge-frontend/src/features/editor/components/blocks/preview/quiz/DragAndDropQuizPreview.tsx). SCORM uses exported quiz-style markup and existing `.quiz-drag-*` CSS.

---

## 3. Tests

[`JourneyBlockRenderersTest`](course-forge-backend/src/test/java/com/mundrisoft/courseforge/export/render/blocks/JourneyBlockRenderersTest.java): wrapper classes, stem, activity markers, default instructions.

---

## 4. Verification

- Lesson with only `DRAG_AND_DROP` blocks still loads `quiz.js` (lesson template).
- Submit records interaction and updates score like a single-question quiz.
