## 1. Refined Requirement

Build a production-ready `EMBED` block capability for lesson authoring where authors can paste either a URL or iframe snippet, resolve it through backend normalization and safety checks, preview the resolved result, and save it through existing block persistence APIs without breaking existing contracts.

Expanded scope:
- Add/maintain resolver endpoint `POST /api/blocks/embed/resolve`.
- Use existing block save/update APIs (`/api/blocks/{lessonId}/bulk`, `/bulk-update`) with `type = EMBED`.
- Persist canonical EMBED content in `blocks.content` JSON object.
- Enforce server-side validation and security constraints for persisted URLs.
- Keep response wrapper and auth behavior consistent with platform standards (`ApiResponseDto`, JWT).

Non-goals for this requirement:
- SCORM import/export parity for embed.
- Full third-party provider parity across all websites.
- New block-specific save endpoint.

---

## 2. Functional Scenarios

1. **Resolve URL successfully**
   - Author inputs valid embeddable URL.
   - UI calls resolve API.
   - Backend returns `resolved=true` with canonical fields and metadata.
   - UI shows preview and enables save.

2. **Resolve iframe successfully**
   - Author inputs valid single iframe HTML snippet.
   - Backend extracts URL, validates it, resolves provider/open-graph/fallback.
   - UI receives normalized payload and renders preview.

3. **Resolve fails due to unsupported/unsafe input**
   - Backend returns `resolved=false` with reason (HTTP 200 contract for functional failure path).
   - UI shows reason and blocks save (recommended default).

4. **Save embed block via bulk create**
   - UI sends `type: "EMBED"` and resolved content in `blocks[]`.
   - Backend validates required fields and persists block.
   - UI parses response and reflects saved state.

5. **Update existing embed block via bulk update**
   - Author changes input, re-resolves, and updates content.
   - Backend replaces content and preserves ordering semantics via `orderIndex`.

6. **Reopen lesson with saved embed**
   - Existing EMBED block content is fetched.
   - UI hydrates and renders iframe from persisted `embedUrl`.

---

## 3. Edge Cases

- Empty or whitespace-only `input`.
- Oversized `input` payload (high character length iframe/html).
- Iframe with missing/invalid `src`.
- Non-HTTP(S) scheme (`javascript:`, `file:`, `data:`).
- Redirect chain exceeds allowed max redirects.
- Host resolves to private/internal network target (SSRF prevention).
- Provider metadata unavailable but fallback is still possible.
- Mixed casing / encoded URLs requiring normalization.
- Save attempt with `resolved=false` payload (manual tampering or stale client state).
- Save attempt missing required `originalUrl` or `embedUrl`.
- Metadata fields present but malformed type (e.g., boolean sent as string).

---

## 4. Validation Rules

### 4.1 Field-level Validation
- Resolve request:
  - `input`: required, non-blank, bounded max length.
- Persisted EMBED content:
  - `originalUrl`: required, valid URL format.
  - `embedUrl`: required, valid URL format.
  - Optional fields must match expected types (`title`, `description`, `thumbnail`, `provider`, `providerUrl`, `favicon`, `embedType`, `showMetaData`, `resolvedBy`).

### 4.2 Business-level Validation
- Persisted URLs must satisfy safety rules (scheme and network safety).
- Unsafe/blocked targets must not produce savable canonical URL pair.
- Resolution strategy marker must be in allowed set when present (`OEMBED`, `OPEN_GRAPH`, `RAW_IFRAME`).
- API contracts remain backward compatible for non-EMBED block types.

---

## 5. API Contract

### 5.1 Resolve Embed
- **Endpoint:** `/api/blocks/embed/resolve`
- **Method:** `POST`
- **Request payload:**
```json
{
  "input": "https://www.youtube.com/watch?v=VIDEO_ID"
}
```
- **Response payload (success path):**
```json
{
  "success": true,
  "status": 200,
  "message": "Embed resolved successfully",
  "data": {
    "resolved": true,
    "reason": null,
    "input": "https://www.youtube.com/watch?v=VIDEO_ID",
    "originalUrl": "https://www.youtube.com/watch?v=VIDEO_ID",
    "embedUrl": "https://www.youtube.com/embed/VIDEO_ID",
    "title": "Example",
    "description": null,
    "thumbnail": "https://...",
    "provider": "YouTube",
    "providerUrl": "https://www.youtube.com/",
    "favicon": null,
    "embedType": "video",
    "showMetaData": true,
    "resolvedBy": "OEMBED"
  }
}
```
- **Response payload (functional failure path):**
```json
{
  "success": true,
  "status": 200,
  "message": "Embed could not be resolved",
  "data": {
    "resolved": false,
    "reason": "URL resolves to a blocked network address",
    "input": "https://...",
    "originalUrl": null,
    "embedUrl": null
  }
}
```
- **Error responses:**
  - `400`: invalid request payload (missing/invalid input).
  - `401`: unauthorized.

### 5.2 Save via existing block APIs
- **Endpoints:**
  - `POST /api/blocks/{lessonId}/bulk`
  - `PUT /api/blocks/{lessonId}/bulk-update`
- **Method:** `POST` / `PUT`
- **Request payload (excerpt):**
```json
{
  "blocks": [
    {
      "type": "EMBED",
      "orderIndex": 3,
      "content": {
        "originalUrl": "https://www.youtube.com/watch?v=VIDEO_ID",
        "embedUrl": "https://www.youtube.com/embed/VIDEO_ID",
        "title": "Optional",
        "showMetaData": true,
        "resolvedBy": "OEMBED"
      }
    }
  ]
}
```
- **Error responses:**
  - `400`: invalid embed content (required fields missing/invalid).
  - `401`: unauthorized.

---

## 6. Pagination Requirements

No dedicated list endpoint is introduced for EMBED feature itself; therefore no additional pagination is required.

Existing lesson/block retrieval pagination behavior remains unchanged and out of scope for this requirement.

---

## 7. Logging Requirements

- **Request logs**
  - Log resolver entry with correlation context (user/workspace/request id), without logging sensitive raw payloads excessively.
- **Response logs**
  - Log resolution outcome (`resolved`, `resolvedBy`, reason category), not full large HTML bodies.
- **Error logs**
  - Log validation failures as `warn` with safe context.
  - Log resolver/service failures as `error` with stack trace.
- **Performance logs**
  - Capture resolve latency and external fetch timing for diagnostics.

Log redaction rules:
- Do not log secrets/tokens.
- Avoid storing full untrusted iframe HTML in high-verbosity logs by default.

---

## 8. Performance Considerations

- Resolver may perform external fetches/oEmbed calls; enforce strict connect/read timeout.
- Limit response body size and redirect depth to avoid slow/hanging calls.
- Consider lightweight metadata fallback to avoid repeated expensive provider calls.
- Ensure save path remains fast (validation should be CPU-light, no unnecessary remote calls on save).
- For repeated same URLs, optional caching can be considered in later iteration (not mandatory in current scope).

---

## 9. Security Considerations

- **Authentication**
  - Resolver and save APIs require JWT-authenticated user context.
- **Authorization**
  - Access is controlled by existing lesson/block authorization model.
- **Data exposure risks**
  - Avoid exposing internal network details in `reason`.
  - Avoid leaking blocked-word internals or resolver internals in public responses.
- **Input sanitization**
  - Strict URL parsing and scheme checks.
  - Iframe parsing constraints (single valid iframe, expected attributes).
  - SSRF protections for host/IP resolution and redirect handling.
- **Persistence safety**
  - Reject unsafe URLs before persistence.

---

## 10. Failure Scenarios

1. **Malformed input JSON**
   - Expected: 400 with validation error payload.

2. **Unauthorized request**
   - Expected: 401 from interceptor/auth layer.

3. **Input cannot be normalized to safe URL**
   - Expected: 200 with `resolved=false`, meaningful `reason`.

4. **Provider/OEmbed timeout or network failure**
   - Expected: graceful fallback or `resolved=false`; no crash, no partial save.

5. **Blocked/private target detected**
   - Expected: `resolved=false` with policy-safe message.

6. **Save request missing required canonical fields**
   - Expected: 400 from block content validation.

7. **Client sends stale/invalid resolvedBy**
   - Expected: validation failure if outside allowed values.

8. **Unexpected server exception**
   - Expected: 500 with standard error wrapper; error logged with context.

