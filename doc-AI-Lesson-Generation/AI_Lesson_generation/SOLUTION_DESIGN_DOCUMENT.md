#1. Overview

## Feature summary
AI Lesson Generation enables users to generate a single lesson asynchronously from mixed source inputs (files, URLs, and free text). The generation contract is a 3-step flow:
1. Add Content
2. Lesson Info
3. Generate

The generated output must enforce `lessonContentType=TEXTUAL_ONLY`.

## Business objective
- Reduce manual effort in lesson authoring by automating first-draft creation.
- Provide non-blocking UX by returning immediate async-start response.
- Ensure generated lesson is directly usable in editor/preview.
- Preserve billing and usage governance parity with existing AI Course Generation behavior.

## Reference to requirement
- Requirement source: `doc/AI_Lesson_generation/REQUIREMENT_EXPANSION_DOCUMENT.md`
- Impact source: `doc/AI_Lesson_generation/IMPACT_ANALYSIS_DOCUMENT.md`

---

#2. Architecture Flow

## End-to-end flow (Controller -> Service -> Repository)

### A) Add Content (`POST /lesson/generation/content`) — sync API
1. **Controller** validates auth token and request shape.
2. **Validation service** enforces source-level rules (at least one of files/urls/text, file/url constraints).
3. **File service** stores uploaded files in `file_metadata`.
4. **OpenAiFileService**:
   - uploads provider-supported files (e.g., PDF/text) and stores `openai_file_id`,
   - extracts text for non-provider-native formats and stores `extracted_text`.
5. **Repository** writes/updates `course_source_material` mappings.
6. **Controller** returns `200` with `sessionId`, `fileIds`, and source summary.

### B) Generate (`POST /lesson/generation/generate`) — sync start + async execution
1. **Controller** validates auth/workspace and lesson info fields.
2. **Validation service** enforces:
   - required fields (`lessonTitle`, `description`, `audience`),
   - `lessonContentType=TEXTUAL_ONLY`,
   - `quizQuestionCount` in `1..6` if provided,
   - `sessionId` and ownership checks for sources/module.
3. **Feature threshold service** checks low-credit status.
4. **Controller** delegates to async orchestration service and immediately returns `202 IN_PROGRESS`.
5. **Async service** executes pipeline:
   - resolve sources from `file_metadata` + `course_source_material`,
   - build prompt with lesson context,
   - invoke provider via `AIProviderFactory`,
   - semantically validate AI output (textual-only + quiz constraints),
   - persist lesson + blocks transactionally,
   - trigger success/failure notification.

### C) Retry (`POST /lesson/generation/retry/{sessionId}`) — sync start + async execution
1. **Controller** validates session + caller ownership.
2. **Service** ensures retry eligibility (failed state only).
3. **Service** rejects retry if already in progress (`409`).
4. **Service** starts async re-execution and returns `202`.

## Sync/Async behavior
- **Sync**: request validation, auth, ownership checks, async job kickoff, immediate response.
- **Async**: provider invocation, output validation, persistence, notification, job lifecycle updates.

---

#3. Service Interaction Diagram (Textual)

```text
LessonGenerationController
  -> Jwt/Auth utility
  -> WorkspaceAiActivityService
  -> LessonGenerationValidationService
  -> FeatureBalanceThresholdService
  -> LessonGenerationService

LessonGenerationService
  -> FileService
  -> OpenAiFileService
      -> CourseSourceMaterialRepository
  -> LessonGenerationAsyncService

LessonGenerationAsyncService
  -> WorkspaceAiConfigService
  -> AIProviderFactory
      -> OpenAIProvider / GeminiAIProvider
  -> SemanticOutputValidationService
  -> LessonRepository
  -> BlockRepository
  -> ApiUsageService
      -> BillingBalanceService
          -> WorkspaceBalanceTransactionRepository
  -> NotificationService (email/event)
  -> Optional LessonGenerationJobRepository
```

## External dependencies
- AI Provider APIs (OpenAI/Gemini as configured)
- Notification channel (email service)

---

#4. API Contracts

## API 1: Add Content
- **Endpoint:** `/lesson/generation/content`
- **Method:** `POST`
- **Content-Type:** `multipart/form-data` (and optional JSON mode for URL/text-only path)

### Request Payload (logical)
```json
{
  "files": ["MultipartFile[] optional"],
  "urls": ["https://example.com/ref"],
  "text": "optional source text",
  "sessionId": "optional-uuid"
}
```

### Response Payload (200)
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

### Error Responses
- `400`: Missing/invalid source payload, validation failure
- `401`: Unauthorized
- `403`: Forbidden workspace access
- `413`: Payload too large
- `500`: Internal failure

## API 2: Generate Lesson (Async Start)
- **Endpoint:** `/lesson/generation/generate`
- **Method:** `POST`
- **Content-Type:** `application/json`

### Request Payload
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

### Response Payload (202)
```json
{
  "success": true,
  "status": 202,
  "message": "Lesson Generation Started! You will receive an email notification when it's ready.",
  "data": {
    "sessionId": "uuid-string",
    "jobStatus": "IN_PROGRESS",
    "cta": "Go to Courses"
  },
  "timestamp": "2026-04-29T16:00:00"
}
```

### Error Responses
- `400`: Field/business validation failure
- `401`: Unauthorized
- `403`: Forbidden
- `404`: Session/source/module not found
- `409`: Duplicate active request/retry conflict
- `422`: Semantic output validation failure
- `429`: Rate limit exceeded (if enabled)
- `502/504`: Provider/downstream timeout/failure
- `500`: Internal failure

## API 3: Retry Failed Generation
- **Endpoint:** `/lesson/generation/retry/{sessionId}`
- **Method:** `POST`

### Request Payload
```json
{
  "reason": "optional"
}
```

### Response Payload (202)
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

### Error Responses
- `404`: Session not found
- `409`: Retry not allowed (already running)
- `500`: Retry initialization failure

---

#5. Database Design

## Tables impacted
- `file_metadata` (existing): uploaded source files
- `course_source_material` (existing): file-to-provider mapping and extracted text
- `lessons` (existing): generated lesson persistence
- `blocks` (existing): generated block persistence
- `api_usage` (existing): usage + feature tracking
- `workspace_balance_transaction` (existing): deductions and transaction history

## New tables (if any)
### Recommended: `lesson_generation_job`
Purpose: robust async lifecycle, status visibility, idempotency, retry control.

### Proposed columns
- `id` (PK, UUID)
- `session_id` (unique with workspace scope)
- `workspace_id`
- `user_id`
- `status` (`IN_PROGRESS`, `FAILED`, `COMPLETED`)
- `request_payload` (JSONB/TEXT, optional sanitized)
- `error_code` (nullable)
- `error_message` (nullable, sanitized)
- `retry_count` (default 0)
- `started_at`
- `completed_at` (nullable)
- `created_at`
- `updated_at`

## Indexing strategy
- `lesson_generation_job(session_id)` unique index
- `lesson_generation_job(workspace_id, status, created_at desc)` composite index
- `course_source_material(file_id)` unique index (already expected)
- `api_usage(workspace_id, created_at)` for reporting windows
- `workspace_balance_transaction(workspace_id, created_at)` for ledger queries

## Migration requirements
- Optional migration for `lesson_generation_job`.
- No mandatory schema change if status/retry is implemented without persistent job table (not recommended).

---

#6. Pagination Strategy

## APIs needing pagination
- **No pagination required** for:
  - Add Content
  - Generate Async Start
  - Retry
- Pagination applies only to optional listing APIs (job history/usage views).

## Standard parameters
- `page`: default `0`
- `size`: default `20`, max `100`
- `sort`: default `createdAt,desc`, allowed `createdAt`, `status`

## Performance reasoning
- Listing endpoints can grow large and require bounded page sizes to avoid expensive full scans.
- Composite indexes on `(workspace_id, created_at)` and `(workspace_id, status, created_at)` support predictable query latency.

---

#7. Caching Strategy

## What to cache
- Workspace AI configuration (`provider`, effective model, custom-key mode flags)
- Provider metadata/availability checks (short-lived)
- Validation allow-lists (tone styles, enums) if externally sourced

## Cache layer
- Phase 1: In-memory (Caffeine/Spring Cache) for low complexity
- Phase 2 (if multi-node scaling required): Redis for shared cache coherence

## TTL and invalidation
- Workspace AI config: TTL 5-15 min; explicit invalidate on config update
- Provider metadata: TTL 1-5 min
- Enum/validation lists: TTL 30-60 min
- Never cache request-specific generation results (high cardinality + freshness critical)

---

#8. Error Handling Strategy

## Error categories
- Validation errors (`400`)
- Authentication/authorization (`401`, `403`)
- Not found (`404`)
- Conflict/idempotency (`409`)
- Semantic AI output errors (`422`)
- Provider/downstream errors (`502`, `504`)
- System/internal errors (`500`)

## Exception handling flow
1. Controller catches typed exceptions from service layer.
2. Global exception handler maps to standardized API response envelope.
3. Error response includes safe message + correlation/session context.
4. Stack traces logged internally only.

## Retry/fallback mechanisms
- Async generation retries only via explicit retry API (controlled).
- Provider timeout/failure marks job failed and retryable.
- Notification failure should not roll back persisted lesson; schedule notification retry.
- No controller-level business retries; retry orchestration stays in service layer.

---

#9. Logging Strategy

## Request logging
- endpoint, method, workspaceId, userId, sessionId, source counts, generation config (`lessonContentType`, `quizQuestionCount`)

## Response logging
- status code, success/failure, sessionId, job status, elapsed time

## Error logging
- categorized error type, provider metadata/request id (if available), sanitized error details
- do not log full prompts, API keys, raw file content

## Correlation ID usage
- Use `sessionId` as primary domain correlation key across sync + async flow.
- Add technical trace/correlation ID at request entry for infrastructure tracing.

---

#10. Performance Considerations

## Expected data size
- Source payload may include multiple files + URLs + text.
- Extracted text and AI response payloads can be large for lesson generation.

## Query optimization
- Use indexed lookups for session/workspace scoped queries.
- Batch persistence for blocks where possible.
- Keep transactions scoped to persistence boundaries only.

## Bottlenecks
- Provider latency/timeouts
- File extraction/upload throughput
- Async executor saturation
- Persistence hot paths under concurrency

## Load considerations
- Bounded async executors and queue limits
- Workspace-level concurrency control to reduce contention
- Timeout and failure mapping to avoid thread starvation
- Non-blocking generate start should remain fast under load

---

#11. Security Considerations

## Authentication
- JWT required for all lesson-generation APIs.

## Authorization
- Enforce workspace ownership for:
  - `sessionId`
  - source `fileIds`
  - optional `moduleId`

## Input validation
- Strict validation for files/URLs/text and lesson fields.
- Enforce `TEXTUAL_ONLY` and quiz range rules.
- Validate and sanitize generated block payload before persistence.

## Data protection
- Never expose provider keys or sensitive internals in responses/logs.
- Prevent cross-workspace access via guessed identifiers.
- Apply secure storage and access controls for uploaded source files.

---

#12. Dependency & Impact Mapping

## Modules impacted
- Controllers: new lesson-generation endpoints
- Services: validation/orchestration/async/persistence integration
- Repositories: lesson, block, source material, usage, transaction, optional job
- Provider integrations: OpenAI/Gemini via factory
- Billing/usage: usage recording + deduction pipeline

## APIs affected
- New: content, generate, retry (and optional status/listing)
- Existing `/course/*` APIs remain unchanged

## Backward compatibility risks
- Shared validator/provider usage tagging may unintentionally affect existing flows if not isolated.
- Reporting representation differences for custom-key consumption can cause UI inconsistency.

## Required refactoring
- Introduce dedicated lesson-generation validation + semantic validation service.
- Introduce async orchestration service if not already present.
- Add explicit feature code constants for usage tracking.

---

#13. Edge Cases Handling

## Special scenarios
- No source input -> reject early.
- Invalid `lessonContentType` or quiz count -> reject early.
- Provider file upload failure with extracted text available -> continue when policy allows.
- Retry requested during `IN_PROGRESS` -> return conflict.

## Failure conditions
- Provider timeout/failure -> mark failed, notify, allow retry.
- Semantic output invalid (`TEXTUAL_ONLY` violation, quiz mismatch) -> reject, no partial publish.
- Persistence failure -> rollback and mark failed.
- Notification failure after success -> keep lesson persisted, retry notification asynchronously.
- Platform low-credit BLOCK + no custom key -> block generation.
- Platform low-credit BLOCK + custom key active -> allow generation with zero-effective consumption.

---

#14. Open Questions / Assumptions

## Missing inputs / clarifications needed
1. Final decision on status retrieval model:
   - polling endpoint required now, or notification-only is sufficient?
2. Retry semantics:
   - same session reuse mandatory, or allow new session per retry?
3. Reporting representation for custom-key consumption:
   - should UI display `0`, `-`, or blank?
4. Exact allowed values for `toneStyle` and max lengths for mandatory fields.
5. Whether provider fallback (OpenAI -> Gemini) is required or optional per workspace config.

## Assumptions used in this design
- Existing billing and usage pipeline from course generation is reused for consistency.
- Business logic remains strictly in service layer; controllers only orchestrate request/response.
- Optional `lesson_generation_job` table is recommended for production-grade retry/idempotency.
