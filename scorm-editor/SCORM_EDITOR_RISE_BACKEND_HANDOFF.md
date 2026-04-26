# Rise 360 SCORM Editor Backend - Developer Handoff

## Scope of this document

This document captures the **current backend implementation status** of the SCORM Editor flow, with emphasis on the **Rise 360 path** that is actively completed and usable.  
It is intended to help the next developer resume work quickly without re-tracing controller/service/parser internals.

---

## 1) High-level end-to-end flow

### Entry point

- Base API path: `/scorm/editor`
- Main controller: `scorm/editor/controller/ScormEditorController.java`

### Runtime lifecycle for one edit session

1. **Upload zip**
   - Endpoint: `POST /scorm/editor/upload`
   - Service chain:
     - `ScormUploadService.processUpload(...)`
     - `ScormParserService.parse(...)`
     - `ScormParserService.loadCourseStructure(...)`
   - Output includes `sessionId`, `detected tool`, `status`, and `courseStructure`.

2. **Load slide content**
   - Endpoint: `GET /scorm/editor/sessions/{sessionId}/slides/{slideId}`
   - Service chain:
     - `EditSessionStore.get(sessionId)`
     - `ScormParserService.loadSlideContent(...)`
   - For Rise, this reads from Base64 course JSON embedded in `scormcontent/index.html`.

3. **Update blocks**
   - Endpoint: `PUT /scorm/editor/sessions/{sessionId}/slides/{slideId}/blocks`
   - Service chain:
     - `ScormContentEditorService.updateBlocks(...)`
     - Parser write-back (`RiseParser.writeBackBlocks(...)` for Rise)
   - Updates are persisted directly into extracted package files.

4. **Media list/replace/serve**
   - Endpoints:
     - `GET /sessions/{sessionId}/images`
     - `GET /sessions/{sessionId}/audios`
     - `POST /sessions/{sessionId}/media/upload`
     - `GET /sessions/{sessionId}/media/serve?mediaId=...`
   - Service: `ScormMediaService`
   - Media replacement writes binary directly into extracted package path.

5. **Preview**
   - Endpoint: `GET /sessions/{sessionId}/preview`
   - Service chain:
     - `ScormPreviewService.preparePreview(...)`
     - `ScormParserService.updateIndexHtmlOptions(session, true)` (Rise-specific flag)
     - `ScormMediaService.resolvePreviewPath(...)`
   - Asset serving endpoint used by iframe:
     - `GET /sessions/{sessionId}/assets/**`

6. **Export + download**
   - Endpoints:
     - `POST /sessions/{sessionId}/export`
     - `GET /sessions/{sessionId}/download`
   - Service chain:
     - `ScormParserService.updateIndexHtmlOptions(session, false)` before export
     - `ScormExportService.exportPackage(...)`
     - `ScormExportService.streamDownload(...)`

---

## 2) Session model and storage layout

### Session state holder

- `scorm/editor/model/EditSession.java`
- Stored in memory via `EditSessionStore` (`ConcurrentHashMap`)
- Session-level lock: `ReentrantReadWriteLock`

### Status usage

- `READY`: usable for read/edit/media/preview/export
- `EXPORTED`: still readable/editable and downloadable
- `FAILED`: parsing/validation failure path

### Per-session filesystem structure

Under configured temp dir (`scorm.editor.temp-dir`), each session creates:

- `{sessionId}/original.zip`
- `{sessionId}/extracted/...` (working copy)
- `{sessionId}/parsed/course.json`
- `{sessionId}/export/course_updated.zip`

Example session id format: `scorm_<12-char-random>`.

---

## 3) Rise 360 parser behavior (current implemented logic)

Implementation: `scorm/editor/parser/RiseParser.java`

### Detection/validation

- `canParse(...)` expects:
  - `scormcontent/` directory
  - `scormdriver/` directory
  - `scormcontent/index.html`
- `validate(...)` verifies index contains Rise data marker (`deserialize(...)` or `window.courseData = "..."`).

### Course structure extraction

- Reads embedded Base64 course JSON from `scormcontent/index.html`.
- Maps `course.lessons[]` to `SlideInfo`.
- Uses 1-based slide ids.

### Slide content extraction (Rise -> ContentBlock list)

Supported extracted block families:

- Text/list/quote:
  - `heading`
  - `bodyText`
  - `listItem`
  - `quote`
  - `quoteSource`
- Audio:
  - Extracts from `media.audio`
  - Emits block type `audio` with map content (`url`)
- Interactive:
  - `accordion`
  - `tabs`
  - `flashcard`
  - `flashcardgrid`
  - Normalized in API as `accordion`, `tabs`, `flashcards`

### Developed block types list (current)

This is the current list of block/component types already developed in SCORM Editor backend.

- Rise (`RiseParser`)
  - `heading`
  - `bodyText`
  - `listItem`
  - `quote`
  - `quoteSource`
  - `audio` (read in slide blocks; media file replacement is via media API)
  - `accordion`
  - `tabs`
  - `flashcards` (mapped from Rise `flashcard` / `flashcardgrid`)

- Storyline (`StorylineParser`) - basic support
  - `heading`
  - `bodyText`
  - `audio` (local src extraction only)

- Captivate (`CaptivateParser`)
  - no developed editable block types yet (stub only)

### Write-back behavior

- Re-opens `scormcontent/index.html`, decodes embedded Base64 JSON, mutates lesson items, re-encodes, and rewrites file.
- Text/list/quote and supported interactive blocks are written back.
- `audio` blocks are consumed for block-index continuity but media JSON is not mutated in write-back; binary replacement is expected via media API.

### Rise preview/export flag

- `updateIndexHtmlOptions(...)` toggles:
  - `const allowOutsideDriver = true` for preview
  - `const allowOutsideDriver = false` for export

---

## 4) Other tool support status

### Storyline (`StorylineParser`)

- Implemented for basic parse + write-back using HTML selectors.
- Supports extraction of headings/body text and local audio references.
- Media replacement still uses generic media API (same as Rise).

### Captivate (`CaptivateParser`)

- Currently a stub:
  - `validate(...)` throws `ScormValidationException`
  - parse/write methods throw `UnsupportedOperationException`
- Detectable, but intentionally blocked from editing flow.

---

## 5) Error handling and response pattern

### Success responses

- Most JSON success endpoints return `ResponseEntity<ApiResponseDto<T>>`.

### Error responses

- `ScormEditorExceptionHandler` is scoped to SCORM editor controller package.
- Returns `Map<String,Object>` shape:
  - `success: false`
  - `error: ...`
  - `timestamp: ...`
- Handles:
  - `SessionNotFoundException` -> 404
  - `IllegalArgumentException` -> 400
  - `IllegalStateException` -> 409
  - `MaxUploadSizeExceededException` -> 413
  - generic/IO -> 500

Note: this differs from `ApiResponseDto` error style used in some success-oriented controller methods.

---

## 6) Concurrency and security controls already present

### Concurrency

- Per-session lock is used consistently:
  - Read lock for list/load/preview resolve/export read paths.
  - Write lock for content write and media replace.

### Security checks

- Zip extraction has Zip-Slip prevention.
- Media serving/replacement validates normalized path stays inside `extractedDir`.
- Preview assets endpoint blocks path traversal through media path validation.

---

## 7) What is already tested

### Rise parser tests

- `RiseParserTest` covers:
  - variant normalization (`subheading`, underscore variants)
  - field emission policy correctness
  - write-back index stability
  - no duplicate paragraph behavior in fixture

### Preview tests

- `ScormPreviewServiceTest` covers:
  - empty preview when entry file missing
  - URL generation with configured base URL
  - request URI to relative asset extraction

### Content-type resolution

- `PreviewContentTypeResolverTest` covers extension-based MIME mapping basics.

---

## 8) Known gaps / pending development items

1. **Captivate support is not implemented**  
   Parser is placeholder only.

2. **Some Rise block families remain unsupported**  
   Current interactive support is limited to accordion/tabs/flashcards variants.

3. **In-memory session store only**  
   Restart loses active sessions; no persistence layer for resumable edits.

4. **Mixed error contract**  
   Editor advice returns map-based error payloads while many APIs return `ApiResponseDto` on success.

5. **Cleanup scheduler delay expression should be re-verified**  
   `@Scheduled(fixedDelayString = "${scorm.editor.cleanup-interval-minutes:30}000" + "0")` is unusual and should be validated against intended minute->ms conversion behavior.

6. **Pagination params currently unused in slide endpoint**  
   `page/size/sort` exist in controller signature for `getSlideContent` but not applied in parser/service.

7. **No dedicated end-to-end integration suite for full upload->edit->export flow**  
   Existing tests are mostly unit-level around parser and preview helpers.

---

## 9) Suggested resume plan (safe order)

1. Add/confirm integration test for complete Rise happy path:
   upload -> load slide -> update block -> preview -> export -> download.
2. Verify/adjust cleanup scheduler fixed delay expression.
3. Align error response contract for SCORM editor APIs (without breaking frontend expectations).
4. Expand Rise block support incrementally for additional item variants.
5. Decide and implement persistence strategy for edit sessions if cross-restart continuity is required.
6. Implement Captivate parser only as separate scoped work item.

---

## 10) Quick reference: main classes by responsibility

- API layer: `ScormEditorController`
- Upload/extract/validation/session creation: `ScormUploadService`
- Parser orchestration + course/slide loading: `ScormParserService`
- Rise parsing + write-back: `RiseParser`
- Slide content mutation orchestration: `ScormContentEditorService`
- Media listing/replace/serve + preview entry discovery: `ScormMediaService`
- Preview URL + asset path extraction: `ScormPreviewService`
- Export + stream download: `ScormExportService`
- Session in-memory registry + directory deletion: `EditSessionStore`
- Stale session cleanup scheduler: `EditSessionCleanupService`
- SCORM editor exception mapping: `ScormEditorExceptionHandler`

---

## 11) Block/component extension guide (merged)

Use this section when adding new editable Rise blocks/components without changing unrelated flows.

### Scope for extension work

For Rise, block editing is centralized in `scorm/editor/parser/RiseParser.java`.

When adding a new component, changes are usually limited to:

- `parseSlideContent(...)`
- `writeBackBlocks(...)`
- helper methods in `RiseParser`
- `RiseParserTest`

### Existing block contract conventions

`ContentBlock`:

- `blockId` (int): ordered identifier
- `type` (string): stable frontend-facing key
- `content` (Object):
  - string for simple text
  - map for media payload
  - list of maps for composite blocks

### Standard implementation pattern

1. Identify Rise JSON shape from real export fixtures.
2. Add parse mapping (Rise JSON -> `ContentBlock`).
3. Add write-back mapping (`ContentBlock` -> Rise JSON fields).
4. Preserve parse/write symmetry and block order.
5. Keep `blockIndex` consumption accurate in `writeBackBlocks`.

### Compatibility guardrails

1. Do not rename existing block `type` values.
2. Do not change endpoint contracts/status codes.
3. Keep edits scoped to the target block family.
4. Continue using media APIs for binary replacement (`/media/upload`).
5. Do not modify session/preview/export lifecycle as part of block additions.

### Suggested helper template

- `parse<NewBlockType>(JsonNode item, List<ContentBlock> blocks, int blockId)`
- `writeBack<NewBlockType>(JsonNode item, List<ContentBlock> blocks, int blockIndex)`

Wire helpers into the main parse/write loops with variant/type checks.

### Testing checklist for each new block

1. Parse test (assert emitted `type` + payload shape)
2. Write-back round-trip (parse -> modify -> writeBack -> parse)
3. Variant normalization test (if variant-driven)
4. Index stability test with mixed block families

### Practical delivery workflow

1. Create a small Rise sample for the target block.
2. Implement parse path + tests.
3. Implement write-back path + round-trip tests.
4. Run/verify existing Rise parser tests.
5. Manual API sanity flow: upload -> get slide -> update -> preview -> export.

### Common pitfalls

- Ambiguous variant checks: normalize before comparisons.
- Block index drift: increment only when a block is actually consumed.
- Overwriting full JSON nodes: patch minimal fields only.
- Rewriting external URLs: keep `http/https/data` URLs untouched.

### Definition of done (new block/component)

1. Block appears in `GET slide` response with stable schema.
2. Edits persist through `PUT blocks`.
3. Round-trip parser tests pass.
4. Existing tests remain green.
5. Preview/export behavior remains unchanged.

