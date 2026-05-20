# Implementation Plan - Add Video Block Support

This plan details the steps to implement support for the "Video" block type within the SCORM Editor (Rise 360). This will allow the system to identify, extract, and display video blocks from Articulate Rise exports.

## Proposed Changes

> [!WARNING]
> **Manager Override (May 2026):** The slide content parsing logic below has been implemented but is currently **commented out** in the codebase. As per manager feedback, the current focus is exclusively on the media-level API (`/scorm/editor/sessions/{sessionId}/videos`) for listing videos, rather than editing video blocks within the slide structure. The code can be uncommented later if slide-level video editing is required.

### 1. SCORM Editor Model
**File**: `ContentBlock.java`
- Add a case for `"video"` in the `getDisplayType()` switch statement to return the human-readable label **"Video"**.

### 2. Rise 360 Parser
**File**: `RiseParser.java`
- Add a new constant `FIELD_VIDEO = "video"`.
- Update `parseSlideContent` to check for `media.video` within items.
- Implement/use a media source extraction helper to resolve the video URL, consistent with the `audio` implementation.
- Update `writeBackBlocks` to advance the block index past `video` blocks without mutating the JSON (as media replacement is handled via a dedicated asset API).

### 3. Tests
**File**: `RiseParserTest.java`
- Add a new test case `parseSlideContent_videoVariant_extractsMetadataCorrectly` to verify that a video block in the course JSON is correctly converted to a `ContentBlock` of type `video` with the expected URL.

## Verification Plan

### Automated Tests
- Run `mvn test -Dtest=RiseParserTest` using JDK 17 to verify all tests pass.

### Manual Verification
- Verify that the JSON output for a slide containing a video now includes a block with `type: "video"`.
