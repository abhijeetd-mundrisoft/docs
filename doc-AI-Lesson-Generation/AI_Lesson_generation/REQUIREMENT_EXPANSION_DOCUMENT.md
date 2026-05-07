# Requirement Expansion Document (RED)

**Feature:** AI Lesson Generation (Revised)  
**Stack:** Spring Boot backend — controllers, services, repositories, provider integrations, async executors, billing/usage services.  
**INPUT requirement:** Generate a lesson asynchronously using content sources (files/URLs/text) and lesson inputs; enforce `TEXTUAL_ONLY` content mode; apply platform-vs-custom-key credit behavior same as AI Course Generation.

---

## #1. Refined Requirement

### Scope (in)

- Provide backend capability to generate a **single lesson asynchronously** from one or more content sources:
  - uploaded documents
  - URLs
  - raw text
- Support a 3-step workflow contract:
  1. Add Content
  2. Lesson Info
  3. Generate
- Lesson Info fields:
  - `lessonTitle` (mandatory)
  - `description` (mandatory)
  - `audience` (mandatory)
  - `toneStyle` (optional)
  - `lessonContentType` (mandatory, **only `TEXTUAL_ONLY` allowed**)
  - `quizQuestionCount` (optional, if provided must be 1–6)
- Generate endpoint must:
  - return immediate confirmation/start response (non-blocking UX)
  - run AI generation asynchronously
  - notify user on completion/failure
  - support retry for failed attempts
- Post-success expectation:
  - lesson opens in Lesson Editor
  - blocks are editable
  - preview works
- Credit consumption must follow existing course-generation behavior:
  - Platform model: cost calculated + deducted + reflected in transaction history and consumption summary
  - External/custom key: no effective deduction, but usage/transaction entry still recorded

### Scope (out / TBD)

- Final async status-retrieval API contract (polling endpoint vs event-driven-only).
- Exact retry guard semantics (same session reuse vs new session).
- Exact reporting representation for “blank/null” consumption in UI DTOs.

### Success criteria

- User can start async lesson generation with valid inputs and receive immediate confirmation.
- System rejects invalid inputs early (no content, missing mandatory lesson fields, wrong content type, invalid quiz count, oversize sources).
- Billing and usage records correctly follow platform-vs-custom-key rules.

---

## #2. Functional Scenarios

| ID | Actor | Flow |
|----|-------|------|
| FS-01 | Authenticated user | Opens Generate Lesson page; 3-step flow is available. |
| FS-02 | Authenticated user | Adds multiple content sources (files + URLs + text) and proceeds. |
| FS-03 | System | Blocks transition to next step when no content is provided. |
| FS-04 | Authenticated user | Enters lesson info with mandatory fields and `TEXTUAL_ONLY` content type. |
| FS-05 | System | Blocks generate when lesson info validation fails. |
| FS-06 | Authenticated user | Clicks Generate; backend returns immediate “started” response with session/job context. |
| FS-07 | System (async) | Resolves sources, builds prompt, calls AI provider, validates output, persists lesson/blocks, sends notification. |
| FS-08 | System | On success, lesson becomes available for editor/preview path. |
| FS-09 | System | On failure, logs error, triggers notification, marks request retryable. |
| FS-10 | Billing/Usage | Platform model: usage cost is calculated and deducted with transaction history + consumption summary updates. |
| FS-11 | Billing/Usage | External/custom key mode: no effective deduction, but usage trail/transaction record still created. |

---

## #3. Edge Cases

| ID | Condition | Expected behavior |
|----|-----------|-------------------|
| EC-01 | No content provided | Reject with validation error before generation starts. |
| EC-02 | `quizQuestionCount=0`/`7`/negative/non-numeric | Reject with validation error; allowed range only 1–6 if provided. |
| EC-03 | `lessonContentType` sent as unsupported value | Reject request; only `TEXTUAL_ONLY` accepted. |
| EC-04 | AI returns non-text/interactive/video blocks while mode is textual-only | Fail semantic validation and reject output (no partial publish). |
| EC-05 | Oversized file or too many files | Reject upload/request early. |
| EC-06 | URL unreachable or invalid | Reject URL input per validation policy. |
| EC-07 | OpenAI file upload fails but extracted text exists | Continue using extracted-text path if policy allows; log warning. |
| EC-08 | Async job succeeds but notification fails | Keep lesson persisted; log and retry notification. |
| EC-09 | Retry triggered while job still IN_PROGRESS | Return conflict/validation response; no duplicate execution. |
| EC-10 | Platform balance BLOCK with no custom key | Block generation start. |
| EC-11 | Platform balance BLOCK but custom key active | Allow generation per custom-key bypass logic. |

---

## #4. Validation Rules

### Field-level

| Field | Rules |
|------|-------|
| `files`/`urls`/`text` | At least one source required (across all source types). |
| `lessonTitle` | Required, non-blank, max length configured. |
| `description` | Required, non-blank, max length configured. |
| `audience` | Required, non-blank, max length configured. |
| `toneStyle` | Optional; if present must match allowed enum/list. |
| `lessonContentType` | Required and must equal `TEXTUAL_ONLY`. |
| `quizQuestionCount` | Optional; if provided must be integer in `1..6`. |
| `sessionId` | Optional at ingest, required for generate/status/retry flows once issued. |
| `moduleId` (if used) | Optional; if supplied, must exist and pass ownership/workspace checks. |
| Files | Validate type/count/per-file size/total size. |
| URLs | Validate syntax/scheme/count/length and optional accessibility checks. |

### Business-level

- Authenticated user + workspace context required.
- Source references (`fileIds`) must belong to caller workspace/session.
- Generated lesson output must be semantically valid for textual-only mode.
- Quiz block content must match requested count when `quizQuestionCount` is provided.
- Async retry must be idempotent-safe and avoid duplicate persistence.
- Credit/usage behavior must match existing course-generation pipeline:
  - **Feature threshold gate** + provider checks for platform model
  - **Custom key bypass** for effective consumption
  - Usage records always written for audit

---

## #5. API Contract

> Path names are proposed and should align with project route conventions.

### Contract A — Add Content

| Item | Specification |
|------|----------------|
| **Endpoint** | `{base}/lesson/generation/content` |
| **Method** | `POST` |
| **Content-Type** | `multipart/form-data` (or mixed support with JSON for URL/text-only mode) |

**Request payload (logical fields)**

| Field | Type | Required |
|------|------|----------|
| `files` | `MultipartFile[]` | Optional (required only if no URL/text provided) |
| `urls` | `String[]` | Optional (required only if no file/text provided) |
| `text` | `String` | Optional (required only if no file/URL provided) |
| `sessionId` | `String` | Optional |

**Response payload (200)**
```json
{
  "success": true,
  "status": 200,
  "message": "Content accepted",
  "data": {
    "sessionId": "uuid-string",
    "fileIds": ["lms-file-id-1"],
    "sourceSummary": {
      "fileCount": 1,
      "urlCount": 1,
      "hasText": true
    }
  },
  "timestamp": "2026-04-29T16:00:00"
}
```

**Error responses**

| HTTP | Condition |
|------|-----------|
| 400 | Missing/invalid source input; source validation failure. |
| 401 | Unauthorized. |
| 403 | Forbidden workspace access. |
| 413 | File payload too large. |
| 500 | Internal server failure. |

---

### Contract B — Generate Lesson (Async Start)

| Item | Specification |
|------|----------------|
| **Endpoint** | `{base}/lesson/generation/generate` |
| **Method** | `POST` |
| **Content-Type** | `application/json` |

**Request payload (JSON)**
```json
{
  "sessionId": "uuid-string",
  "fileIds": ["lms-file-id-1"],
  "urls": ["https://example.com/ref"],
  "text": "optional input text",
  "lessonTitle": "Phishing Awareness Basics",
  "description": "Intro lesson on phishing detection",
  "audience": "Beginner employees",
  "toneStyle": "PROFESSIONAL",
  "lessonContentType": "TEXTUAL_ONLY",
  "quizQuestionCount": 4,
  "moduleId": "optional-module-id"
}
```

**Response payload (202)**
```json
{
  "success": true,
  "status": 202,
  "message": "Lesson Generation Started! Your will receive an email notification when it's ready.",
  "data": {
    "sessionId": "uuid-string",
    "jobStatus": "IN_PROGRESS",
    "cta": "Go to Courses"
  },
  "timestamp": "2026-04-29T16:00:00"
}
```

**Error responses**

| HTTP | Condition |
|------|-----------|
| 400 | Field/business validation failure (incl. unsupported content type). |
| 401 | Unauthorized. |
| 403 | Workspace/module permission denied. |
| 404 | Session/source/module not found. |
| 409 | Duplicate active request / retry conflict. |
| 422 | Semantic AI output validation failure. |
| 429 | Rate limit exceeded (if enabled). |
| 502 / 504 | Provider/downstream timeout/failure. |
| 500 | Internal server failure. |

---

### Contract C — Retry Failed Generation (Recommended)

| Item | Specification |
|------|----------------|
| **Endpoint** | `{base}/lesson/generation/retry/{sessionId}` |
| **Method** | `POST` |

**Request payload (JSON)**
```json
{
  "reason": "optional"
}
```

**Response payload (202)**
```json
{
  "success": true,
  "status": 202,
  "message": "Lesson regeneration started",
  "data": {
    "sessionId": "uuid-string",
    "jobStatus": "IN_PROGRESS"
  }
}
```

**Error responses**

| HTTP | Condition |
|------|-----------|
| 404 | Session not found. |
| 409 | Retry not allowed (already running). |
| 500 | Retry initialization failure. |

---

#6. Pagination Requirements

**Not required** for:
- Add Content
- Generate (async start)
- Retry

Pagination is required only for optional listing APIs (job history/usage views).

If added, recommended:
- `page`: default 0
- `size`: default 20, max 100
- `sort`: `createdAt,desc` (allowed: `createdAt`, `status`)

---

#7. Logging Requirements

| Category | What to log |
|----------|-------------|
| **Request logs** | endpoint, workspaceId, userId, sessionId, source counts, high-level generation config (`lessonContentType`, `quizQuestionCount`). |
| **Response logs** | HTTP status, success flag, sessionId, job status, elapsed time. |
| **Error logs** | categorized errors, provider metadata/requestId, safe validation details, stack traces for internal failures. |
| **Performance logs** | timings for ingest/extract/upload/provider/parse/validate/persist/notify and async queue wait stats. |

Logging constraints:
- Do not log raw file bodies, full prompts, or API keys.

---

#8. Performance Considerations

- Async processing is mandatory to avoid long blocking requests.
- Main bottlenecks:
  - provider latency
  - source extraction/upload throughput
  - persistence on high concurrency
- Use bounded executor queues and workspace-level concurrency control.
- Keep persistence transactional and batch-oriented for blocks.
- Apply timeouts and controlled retries for provider calls.
- Credit/usage computation must not block UI start response.
- In custom/external key mode, keep cost zero while still recording usage trail.

---

#9. Security Considerations

| Area | Requirement |
|------|-------------|
| **Authentication** | JWT required for all lesson-generation endpoints. |
| **Authorization** | Enforce workspace ownership for session/source/module references. |
| **Data exposure risks** | Prevent cross-workspace access via guessed IDs; hide internal errors/prompts/keys. |
| **Input sanitization** | Validate and normalize URLs/text, validate generated block JSON/schema, reject unsupported block types/content. |

---

#10. Failure Scenarios

| Failure | Expected behavior |
|---------|-------------------|
| Missing/invalid source input | Reject request early with validation error. |
| Provider timeout/failure | Mark job failed, log, notify user, enable retry. |
| Malformed AI output | Reject semantically; no partial publish. |
| Output violates textual-only policy | Reject and mark generation failed. |
| Quiz mismatch | Reject and notify. |
| Persistence failure | Rollback transaction, mark failure, notify. |
| Notification failure | Keep generated lesson state, retry notification asynchronously. |
| Duplicate active generation/retry | Return conflict and avoid duplicate work. |
| Platform low-credit BLOCK | Block generation start. |
| Custom key mode + low platform balance | Allow generation with zero-effective consumption. |
| Usage tracking failure | Treat as high-priority observability/billing incident; apply compensating logging/retry policy. |


