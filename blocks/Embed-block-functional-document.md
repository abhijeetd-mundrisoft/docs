# EMBED Block Functional Document

## Document Metadata

- **Module:** Course Authoring - Lesson Blocks
- **Feature:** EMBED Block
- **Version:** 1.0
- **Status:** Draft for implementation alignment
- **Primary audience:** Product, QA, frontend, backend

---

## 1. Business Objective

Enable course authors to include trusted external interactive/media content in lessons by pasting a URL or iframe input, while enforcing backend safety and preserving existing block save contracts.

Expected business value:
- Faster content authoring for externally hosted learning assets.
- Consistent embed behavior across editor sessions.
- Controlled risk through server-side resolution and policy checks.

---

## 2. Scope

### 2.1 In Scope
- Resolve input via backend endpoint: `POST /api/blocks/embed/resolve`.
- Normalize and persist canonical EMBED content in `blocks.content`.
- Create/update EMBED blocks through existing block APIs (`bulk` and `bulk-update` flows).
- Render persisted embed via `embedUrl` in learner/editor preview.

### 2.2 Out of Scope
- SCORM import/export parity for EMBED.
- Provider-specific custom UX beyond returned metadata.
- Full internet-wide provider support guarantees.

---

## 3. Actors and User Journey

### 3.1 Actors
- **Author:** Pastes input, resolves, previews, saves block.
- **Backend Resolver/API:** Validates, normalizes, enriches metadata, applies policy.
- **Learner:** Consumes rendered embed in lesson player.

### 3.2 Journey
1. Author enters URL/iframe in editor.
2. UI calls resolve API.
3. UI checks `data.resolved`.
4. On success, UI stores canonical fields.
5. UI saves block with `type = EMBED`.
6. Learner sees embedded content from persisted `embedUrl`.

---

## 4. Functional Requirements

### FR-1 Input Resolution
System shall accept URL or iframe input through `POST /api/blocks/embed/resolve` and return a normalized response model.

### FR-2 Resolution Decision
System shall always provide a resolution decision in response payload via `data.resolved`.

### FR-3 Canonical URLs
For resolved embeds, system shall return canonical `originalUrl` and `embedUrl` used for persistence/rendering.

### FR-4 Metadata Enrichment
System may return optional metadata (`title`, `description`, `thumbnail`, `provider`, `providerUrl`, `favicon`, `embedType`) when available.

### FR-5 Save Contract
System shall save EMBED content through existing block APIs with:
- `type = "EMBED"`
- `content` as JSON object

### FR-6 Required Persisted Fields
System shall require `originalUrl` and `embedUrl` for valid EMBED persistence.

### FR-7 Deterministic Failure Messaging
When resolution fails for policy/unsupported input, system shall return `resolved=false` and a user-consumable `reason`.

### FR-8 Backward-Compatible Surface
System shall not introduce breaking changes to existing block create/update endpoints.

---

## 5. Business Rules

- BR-1: Resolve request input is mandatory and non-blank.
- BR-2: UI must treat `resolved=false` as unresolved regardless of HTTP 200.
- BR-3: UI should not save unresolved embed by default.
- BR-4: Persisted EMBED block content is object-shaped (not list-shaped).
- BR-5: Resolver strategy indicator (`resolvedBy`) is informational but useful for debugging and analytics.

---

## 6. API Interaction Specification

### 6.1 Resolve API
- **Endpoint:** `POST /api/blocks/embed/resolve`
- **Auth:** Bearer JWT
- **Request body:**
```json
{ "input": "https://example.com/video" }
```

### 6.2 Resolve Response (Key Fields)

| Field | Type | Description |
| --- | --- | --- |
| `resolved` | boolean | Final resolver decision |
| `reason` | string/null | Failure reason when unresolved |
| `originalUrl` | string/null | Canonical source URL |
| `embedUrl` | string/null | URL used in iframe |
| `resolvedBy` | string/null | `OEMBED` / `OPEN_GRAPH` / `RAW_IFRAME` |

### 6.3 Save APIs
- `POST /api/blocks/{lessonId}/bulk`
- `PUT /api/blocks/{lessonId}/bulk-update`

Save payload must include `type: "EMBED"` and `content` object containing required URL fields.

---

## 7. EMBED Content Data Contract

| Field | Required | Description |
| --- | --- | --- |
| `originalUrl` | Yes | Canonical source URL |
| `embedUrl` | Yes | iframe source URL |
| `input` | No | Original pasted text |
| `title` | No | Display title |
| `description` | No | Display description |
| `thumbnail` | No | Preview image URL |
| `provider` | No | Provider name |
| `providerUrl` | No | Provider base URL |
| `favicon` | No | Site icon URL |
| `embedType` | No | Content type hint |
| `showMetaData` | No | UI metadata toggle |
| `resolvedBy` | No | Resolution strategy marker |

---

## 8. Failure Scenarios and Handling

### 8.1 Resolve Validation Failure
- **Case:** Missing/invalid `input`
- **Expected:** 400 response
- **UI action:** Show validation error; keep input editable

### 8.2 Unauthorized Access
- **Case:** Missing/invalid token
- **Expected:** 401 response
- **UI action:** Session/auth recovery flow

### 8.3 Unresolvable/Blocked Input
- **Case:** Policy failure, unsupported source, unsafe target
- **Expected:** HTTP 200 with `resolved=false`, reason populated
- **UI action:** Show reason and prevent save by default

### 8.4 Save Validation Failure
- **Case:** Missing required EMBED fields
- **Expected:** Save API returns validation error
- **UI action:** Keep block state and prompt correction

---

## 9. Non-Functional Requirements

- NFR-1: Resolver outputs should be consistent for identical input and config.
- NFR-2: Existing block APIs must remain stable for non-EMBED types.
- NFR-3: EMBED flow should not degrade general lesson/block save performance.

---

## 10. Acceptance Criteria

- AC-1: Valid URL resolves with `resolved=true` and canonical URLs.
- AC-2: Valid iframe input resolves consistently.
- AC-3: Unsupported/policy-blocked input returns `resolved=false` with readable reason.
- AC-4: `type=EMBED` block can be created and updated via existing block APIs.
- AC-5: Persisted embed reloads correctly in editor.
- AC-6: Learner preview renders from persisted `embedUrl`.
- AC-7: Unauthorized calls return 401 for resolve and save endpoints.

---

## 11. Dependencies and Assumptions

- Existing JWT-protected block API framework is available.
- Existing block persistence model (`blocks` table with JSON content) is active.
- Resolver policy/config is managed server-side.
- Frontend editor follows resolve-then-save workflow.

