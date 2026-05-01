# SCORM Export Enhancement - Solution Design Document

## 1. Overview

### Feature Summary

Enhance SCORM export in the existing LMS backend to support pending `BlockType` values using an incremental, backward-compatible design. The implementation extends current `Scorm12ExportStrategy` and `Scorm2004ExportStrategy` behavior without changing endpoint contracts, package structure, or existing class/module names.

### Design Goals

- **Extensibility:** Add new block types without growing switch-case complexity.
- **Backward compatibility:** Existing supported block types must continue to render exactly as today.
- **Minimal disruption:** Preserve current flow (`Controller -> Service -> Strategy`) and packaging logic.
- **Incremental rollout:** New architecture coexists with old logic during migration.
- **Regression safety:** New paths are isolated and guarded by fallback behavior.

---

## 2. Current System Analysis

### Existing Flow

1. `CourseExportController` receives `/export/course`.
2. `CourseExportService.exportCourseSync()` validates request/tracking and selects `ExportStrategy`.
3. `Scorm12ExportStrategy` / `Scorm2004ExportStrategy`:
   - Generate lesson HTML from blocks.
   - Extract and copy media.
   - Generate manifest.
   - Create ZIP output.

### Current Block Rendering Model

- Block HTML generation is currently switch-case driven in strategy classes (`generateBlockHtml(...)`).
- Unsupported blocks return fallback text: `Unsupported block type`.

### Current Limitations

- Renderer logic is centralized and large, difficult to extend safely.
- Media extraction rules are partially hardcoded and focused on existing media-rich blocks.
- SCORM 1.2 and 2004 can drift due to duplicated rendering paths.
- Any change in switch-case increases regression risk across implemented blocks.

### Risk in Current State

- Coverage gap for pending block types causes learner-visible degradation.
- Increased maintenance cost per block type.
- Slow delivery for new block implementations.

---

## 3. Proposed Architecture (Incremental, NOT Redesign)

### Key Principle

Introduce a handler/registry layer **on top of existing strategy code**, while preserving all current classes and flow.

### Architectural Addition

- Add `BlockRenderer` interface (new extension point).
- Add `RendererFactory`/registry for block-to-renderer lookup.
- Add strategy-local integration point in existing `generateBlockHtml(...)`.
- Keep existing switch-case as compatibility path.

### Co-existence Strategy

At runtime inside existing strategy:

1. Attempt to resolve renderer from registry for current block type.
2. If renderer exists -> render via handler.
3. If renderer does not exist -> fallback to current switch-case path (unchanged behavior).
4. If both fail -> keep existing unsupported fallback text + warning log.

This ensures no break for old blocks and no forced migration in one release.

---

## 4. Migration Strategy (Very Important)

### Step 1: Introduce Interface

- Add `BlockRenderer` contract with `supports(...)` and `render(...)`.
- Add render context object to avoid leaking strategy internals.

### Step 2: Add Factory/Registry

- Register renderer beans at startup.
- Build immutable map from `BlockType` to renderer.
- Log duplicate or conflicting registrations as startup errors.

### Step 3: Route New Blocks to New System

- Implement renderers only for pending phase-1 block types first.
- Strategy method checks registry first, then current switch-case.

### Step 4: Optional Gradual Migration of Existing Blocks

- Migrate low-risk existing blocks one by one (optional).
- Keep switch-case branch until migration completion and regression confidence.

### Zero-Regression Guardrails

- Existing blocks are untouched initially.
- Contract and response format unchanged.
- Unsupported fallback remains as final safety net.
- Add regression tests before enabling migrated existing blocks.

---

## 5. Detailed Component Design

> Note: Class names below are conceptual and should be added under existing export-related packages without restructuring existing modules.

### 5.1 `BlockRenderer` Interface (Pseudo Code)

```java
public interface BlockRenderer {
    Block.BlockType getSupportedType();

    /**
     * Returns HTML fragment for a single block.
     */
    String render(Block block, ScormRenderContext context);
}
```

### 5.2 `ScormRenderContext` (Pseudo Code)

```java
public class ScormRenderContext {
    private final String exportId;
    private final String courseId;
    private final String moduleId;
    private final String lessonId;
    private final int blockNumber;
    private final ObjectMapper objectMapper;
    private final java.util.function.Function<String, String> escapeHtmlFn;
    private final java.util.function.Function<String, String> sanitizeHtmlFn;
    // Optional: scorm version flags, asset path resolvers
}
```

### 5.3 `RendererFactory` (Pseudo Code)

```java
@Component
public class RendererFactory {
    private final Map<Block.BlockType, BlockRenderer> rendererMap;

    public RendererFactory(List<BlockRenderer> renderers) {
        Map<Block.BlockType, BlockRenderer> map = new EnumMap<>(Block.BlockType.class);
        for (BlockRenderer renderer : renderers) {
            Block.BlockType type = renderer.getSupportedType();
            if (map.containsKey(type)) {
                throw new IllegalStateException("Duplicate renderer for " + type);
            }
            map.put(type, renderer);
        }
        this.rendererMap = Collections.unmodifiableMap(map);
    }

    public Optional<BlockRenderer> find(Block.BlockType type) {
        return Optional.ofNullable(rendererMap.get(type));
    }
}
```

### 5.4 Strategy Adapter Integration (Pseudo Code)

```java
private String generateBlockHtml(Block block, int blockNumber) {
    ScormRenderContext context = buildRenderContext(block, blockNumber);

    // New path
    Optional<BlockRenderer> renderer = rendererFactory.find(block.getType());
    if (renderer.isPresent()) {
        try {
            return renderer.get().render(block, context);
        } catch (Exception ex) {
            log.warn("Renderer failed, fallback to legacy switch. exportId={}, lessonId={}, blockId={}, type={}",
                    context.getExportId(), context.getLessonId(), block.getId(), block.getType(), ex);
        }
    }

    // Existing path (unchanged)
    return generateBlockHtmlLegacySwitch(block, blockNumber);
}
```

### 5.5 Reuse of Existing Classes

- Continue using `CourseExportController`, `CourseExportService`, `ExportStrategy`, `Scorm12ExportStrategy`, `Scorm2004ExportStrategy`.
- Reuse existing helper methods for:
  - HTML escaping/sanitization.
  - JSON parsing.
  - media path resolution.
  - manifest and ZIP generation.

No existing class renaming or package redesign is required.

---

## 6. Rendering Strategy

### New Block Types

- Implement each pending block type as an isolated renderer class.
- Renderers output HTML fragments compatible with existing SCORM CSS/JS templates.
- Renderer-specific parsing must tolerate missing optional keys.

### Existing Block Types

- Continue via existing switch-case path in initial release.
- Optional migration only after test coverage is in place.

### Fallback Strategy

1. Renderer present + success -> use renderer output.
2. Renderer present + failure -> warning log, fallback to legacy switch.
3. No renderer + legacy case exists -> use legacy output.
4. No renderer + no legacy case -> keep unsupported placeholder (existing behavior).

---

## 7. Media Handling Design

### Design Objective

Extend existing media extraction logic; do not rewrite copy/packaging pipeline.

### Incremental Extension

- Add `BlockMediaExtractor` abstraction for new blocks while retaining current extraction code.
- Strategy media collection flow:
  1. Try extractor for block type.
  2. If none found, run existing extraction logic.
  3. Merge and deduplicate file IDs using `Set<String>`.

### New Extraction Rules (aligned to phase plan when each block ships)

- `BANNER_IMAGE` -> image `fileId` (Phase 1).
- `RESOURCE_FILE` -> resource `fileId` and optional thumbnail (Phase 1).
- `IMAGE_GRID` -> list of image `fileId`s (Phase 2).
- `QUOTE_WITH_IMAGE` -> quote image `fileId` (Phase 2).
- `AUDIO_PLAYER` -> top-level `fileId` in content JSON (Phase 3).
- `IMAGE_CAROUSEL` -> each entry in `items[]`: `fileId` (and optional `url` for app UI only; export resolves packaged path from `fileId`).
- `QUOTE_CAROUSEL` -> each entry in `quotes[]`: nested `image.fileId` (and optional `url` if present in future shapes).
- `INTERACTIVE_IMAGE` -> base image + hotspot attachment IDs (Phase 4).

### Logging for Missing Media

- Warn with context: `exportId`, `courseId`, `moduleId`, `lessonId`, `blockId`, `blockType`, `fileId`.
- Continue export unless missing file is marked mandatory by block rule.

---

## 8. SCORM Compatibility Design

### Compatibility Guarantee

- Reuse existing manifest generation and packaging logic in both strategies.
- Renderer output must be SCORM-template-compatible HTML/CSS/JS.
- Tracking behavior remains unchanged and controlled by current configuration injection logic.

### SCORM 1.2 and 2004 Coherence

- Share renderer implementations where possible.
- Allow version-specific rendering only when required, controlled via `ScormRenderContext`.
- Keep strategy-specific manifest logic as-is.

---

## 9. Performance Considerations

- Keep current export pipeline; optimize hotspots incrementally.
- Cache renderer and extractor maps at startup (O(1) lookup).
- Deduplicate media IDs before file service calls.
- Avoid repeated JSON parsing in same block path where feasible.
- Optionally introduce bounded parallel media copy for large exports (feature-flagged, after baseline metrics).

No architectural overhaul required.

---

## 10. Error Handling Strategy

### Existing Behavior Preservation

- Hard failures (manifest/zip/workspace) continue to fail export.
- Soft failures (single block render/media miss) continue when safe.

### Structured Logging Extension

- Add consistent log keys:
  - `exportId`, `courseId`, `moduleId`, `lessonId`, `blockId`, `blockType`, `phase`.
- Log levels:
  - `info`: phase transitions.
  - `warn`: block render fallback, missing media.
  - `error`: unrecoverable export failure.

### Exception Boundaries

- Renderer exceptions are caught at strategy boundary and routed to fallback.
- Service/controller contracts remain unchanged.

---

## 11. Testing Strategy

### Regression Safety First

- Keep existing tests passing without modification.
- Add focused tests for new renderer/factory/extractor logic only.

### Recommended Test Layers

1. **Unit tests**
   - Renderer output for valid/partial/invalid JSON content.
   - Factory duplicate registration guard.
   - Extractor file ID extraction and deduplication.

2. **Integration tests**
   - `Scorm12ExportStrategy` and `Scorm2004ExportStrategy` with mixed old+new blocks.
   - Verify fallback path invokes legacy switch when renderer absent/fails.

3. **Package validation tests**
   - Verify lesson files, media files, manifest entries, and ZIP integrity.

4. **Manual smoke**
   - SCORM Cloud (or equivalent LMS) for both 1.2 and 2004 packages.

---

## 12. Phase-wise Implementation Plan

Aligned to requirement phases with safe rollout:

### Phase 0: Foundation

- Add `BlockRenderer`, `RendererFactory`, context object.
- Integrate registry-first + legacy-switch fallback in both strategies.
- Add baseline logs and regression tests.

### Phase 1: Low-risk blocks (as requirement doc)

- `BULLETED_LIST`, `NUMBERED_LIST`, `CHECKBOX_LIST`, `TEXT_WITH_SUBHEADING`, `SUBHEADING_ONLY`, `SPACING`, `SECTION_BREAK`, `BANNER_IMAGE`, `RESOURCE_FILE`.
- Add extraction rules for `BANNER_IMAGE`, `RESOURCE_FILE`.

### Phase 2: Layout + journey (static presentation)

- `IMAGE_GRID`, `QUOTE_WITH_IMAGE`, `TIMELINE_VIEW`, `STEP_FLOW`, `STEP_MARKER`, `ANNOUNCEMENT`, `ANNOUNCEMENT_NOTE`.

### Phase 3: Audio, code, carousels

- `AUDIO_PLAYER`, `CODE_EXAMPLE`, `QUOTE_CAROUSEL`, `IMAGE_CAROUSEL`.

### Phase 4: Hardest blocks last (action buttons, charts, interactives, checkpoint gating, embed, flash)

- `ACTION_BUTTON`, `ACTION_BUTTON_GROUP`, `BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT`, `INTERACTIVE_IMAGE`, `DRAG_AND_DROP`, `SORT_AND_LEARN`, `CHECKPOINT`, `EMBED`, `FLASH_CARDS_STACK`.
- Introduce sanitization policy for embed content.
- Validate legacy alias handling for `FLASH_CARDS`.

### Release Gating per Phase

- Unit + integration + package tests pass.
- No regression in existing implemented block types.
- Manual LMS smoke completed for both SCORM versions.

---

## 13. Backward Compatibility

### Existing Exports Remain Unchanged

- Current endpoint, request, and response contracts are unchanged.
- Existing strategy and packaging flow remains intact.
- Existing implemented block types keep using current switch-case unless explicitly migrated later.

### No Breaking Behavior

- New renderer path is additive and optional per block type.
- Legacy switch-case remains the default fallback for old blocks.
- Unsupported fallback text remains final safety mechanism until full coverage is achieved.

### Operational Compatibility

- No package restructuring.
- No class/module renaming.
- No schema-breaking DB changes.
- No API contract changes for clients.

This design is fully incremental and plugin-oriented for the current codebase.
