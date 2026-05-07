# Impact Analysis Document

**Feature:** AI Lesson Generation (Revised)  
**Reference:** `doc/AI_Lesson_generation/REQUIREMENT_EXPANSION_DOCUMENT.md`  
**Goal:** Assess system impact for async single-lesson generation with mixed content sources, strict `TEXTUAL_ONLY` mode, and platform-vs-custom-key credit behavior aligned to AI Course Generation.

---

## 0) Requirement Traceability Coverage

This impact analysis explicitly maps all RED sections:
- Refined requirement (scope in/out, success criteria)
- Functional scenarios
- Edge cases
- Validation rules
- API contract
- Pagination requirements
- Logging requirements
- Performance considerations
- Security considerations
- Failure scenarios

### RED-to-Impact Mapping Matrix

| RED section | Covered in impact doc |
|---|---|
| #1 Refined Requirement | `0) Requirement Traceability Coverage`, `1) Affected Modules`, `2) API Impact`, `4) Billing, Usage, and Key-Mode Impact` |
| #1 Scope (Out / TBD) | `0) Requirement Traceability Coverage -> Scope (Out / TBD) Impact` |
| #1 Success Criteria | `0) Requirement Traceability Coverage -> Success Criteria Impact`, `5) Validation and Semantic Enforcement Impact`, `6) Async and Retry Impact` |
| #2 Functional Scenarios | `1) Affected Modules`, `2) API Impact`, `6) Async and Retry Impact` |
| #3 Edge Cases | `5) Validation and Semantic Enforcement Impact`, `6) Async and Retry Impact`, `11) Operational and Observability Impact -> Failure-Handling Impact` |
| #4 Validation Rules | `5) Validation and Semantic Enforcement Impact`, `9) Security and Compliance Impact` |
| #5 API Contract | `2) API Impact`, `6) Async and Retry Impact` |
| #6 Pagination Requirements | `11) Operational and Observability Impact -> Pagination Impact` |
| #7 Logging Requirements | `11) Operational and Observability Impact -> Logging Impact` |
| #8 Performance Considerations | `1) Affected Modules -> Async Execution`, `11) Operational and Observability Impact -> Performance Impact` |
| #9 Security Considerations | `9) Security and Compliance Impact` |
| #10 Failure Scenarios | `10) Risk Matrix (Aligned to RED)`, `11) Operational and Observability Impact -> Failure-Handling Impact` |

### Scope (Out / TBD) Impact
- Final status-retrieval contract remains optional and affects whether a job table/status endpoint is mandatory.
- Retry guard semantics (same-session reuse vs new-session) directly impact idempotency model and job-state transitions.
- UI consumption representation for custom-key usage (blank/null/dash) must be aligned across reports to avoid dashboard inconsistency.

### Success Criteria Impact
- Immediate async confirmation drives mandatory non-blocking generate-start API behavior.
- Early input rejection drives stricter pre-flight validation before async scheduling.
- Platform-vs-custom-key parity drives strict reuse of existing billing/usage logic paths instead of introducing new parallel logic.

---

## 1) Affected Modules

### Controllers
- New/additive lesson-generation controller endpoints for:
  - Add Content (`POST /lesson/generation/content`)
  - Generate Async Start (`POST /lesson/generation/generate`)
  - Retry (`POST /lesson/generation/retry/{sessionId}`) recommended
  - Optional status endpoint (`GET /lesson/generation/status/{sessionId}`) if implemented
- Existing lesson editor/retrieval endpoints may be reused for post-success navigation.

### Services
- New orchestration service for async lesson generation pipeline:
  - source resolution
  - prompt construction
  - provider invocation
  - semantic validation (`TEXTUAL_ONLY`, quiz count)
  - lesson/block persistence
  - completion/failure notification
- Reuse/extend existing services:
  - `OpenAiFileService` (file upload + extract text + source material mapping)
  - `AIProviderFactory`, `OpenAIProvider`, `GeminiAIProvider`
  - `ApiUsageService`, `BillingBalanceService`, `FeatureBalanceThresholdService`
  - `WorkspaceAiConfigService` (effective key/model resolution + custom key behavior)
  - `WorkspaceAiActivityService` (AI activity gating)
  - notification service pattern from course generation

### Validation Layer
- New lesson-generation validation service required to enforce:
  - at least one source (`files`/`urls`/`text`)
  - mandatory lesson fields (`lessonTitle`, `description`, `audience`)
  - `lessonContentType` must be exactly `TEXTUAL_ONLY`
  - `quizQuestionCount` in range 1..6 (if provided)
  - source limits (type/count/size/URL validation)
  - semantic output validation (reject non-textual unsupported blocks)

### Persistence
- Reuse:
  - `LessonRepository`, `BlockRepository`, `FileService`, `CourseSourceMaterialRepository`
  - usage/billing persistence (`ApiUsageRepository`, `WorkspaceBalanceTransactionRepository`)
- Optional new persistence:
  - `lesson_generation_job` table (or equivalent) for reliable status + retry + idempotency.

### Async Execution
- Reuse async executor pattern (similar to course generation).
- Add lesson-generation-specific bounded executor and retry-safe state transitions.

---

## 2) API Impact

### New Additive APIs
- `POST /lesson/generation/content`
- `POST /lesson/generation/generate` (returns immediate 202 start response)
- `POST /lesson/generation/retry/{sessionId}` (recommended)
- Optional `GET /lesson/generation/status/{sessionId}`

### Request/Response Semantics Impact
- Introduces 3-step workflow contract:
  1. Add Content
  2. Lesson Info
  3. Generate
- `generate` endpoint must be non-blocking and return start context (`sessionId`, `IN_PROGRESS`).
- Validation and error mapping must align with RED:
  - 400/401/403/404/409/422/429/5xx scenarios.

### Unaffected APIs
- Existing `/course/*` generation flows remain unchanged (no contract break expected).

---

## 3) Data and Database Impact

### Existing Tables Impacted
- `file_metadata`
  - stores source files and contextual linkage (`sessionId`-linked ingestion)
- `course_source_material`
  - stores `openai_file_id` and/or `extracted_text` by LMS file id
- `lessons`
  - stores generated lesson entity
- `blocks`
  - stores generated textual blocks and quiz block
- `api_usage`
  - stores AI usage records with feature/session traceability
- `workspace_balance_transaction`
  - stores deduction/top-up trails and display-consumption values

### Optional New Table
- `lesson_generation_job` (recommended if status/retry is needed robustly)
  - columns likely include: `session_id`, `workspace_id`, `user_id`, `status`, `error`, `retry_count`, `started_at`, `ended_at`.

### Migration Scope
- No mandatory migration if job table is deferred.
- Migration required only if introducing persistent job tracking.

---

## 4) Billing, Usage, and Key-Mode Impact

### Platform Key Mode
- Must follow existing course-generation behavior:
  - threshold gate checks (WARN/BLOCK)
  - provider usage cost computed
  - deductions recorded in workspace balance/transactions
  - consumption summaries updated

### Custom/External Key Mode
- Must preserve same behavior as course generation:
  - usage records still written for audit
  - effective cost/deduction should be zero
  - transaction trail still present (for observability/reporting consistency)

### Feature Tagging Impact
- New feature code should be explicit (`AI_LESSON_GENERATION`) or mapped consistently.
- Incorrect feature tagging can pollute existing `COURSE_GENERATION` metrics.

---

## 5) Validation and Semantic Enforcement Impact

### Input Validation Impacts
- Stronger pre-flight validation needed before async start:
  - no-content rejection
  - lesson field mandatory checks
  - `toneStyle` optional but constrained to allowed values when provided
  - quiz range checks
  - strict content type check (`TEXTUAL_ONLY` only)
  - `sessionId` lifecycle validation (optional at ingest, required for generate/retry/status flows once issued)
  - optional `moduleId` existence + workspace ownership validation
  - source ownership and workspace checks

### AI Output Validation Impacts
- Semantic validator required to reject:
  - non-textual unsupported block types (video/interactive/etc.)
  - malformed lesson output
  - quiz question count mismatch when user specifies count
- No partial publish on semantic violation.

---

## 6) Async and Retry Impact

### Async Start Contract
- Generate API should return immediately with start response and session context.
- Heavy operations shift to async worker:
  - extraction/upload
  - provider call
  - parse/validate
  - persistence
  - notification

### Retry Semantics
- Retry API (recommended) must:
  - allow retry only for failed terminal state
  - reject retry if current state is `IN_PROGRESS` (409 conflict)
  - avoid duplicate lesson/block persistence

### Idempotency Risk
- Without explicit job state control, duplicate requests may create multiple lessons.
- Session-based idempotency keying is strongly recommended.

---

## 7) Dependency Mapping

```text
LessonGenerationController
  -> Jwt/Auth context
  -> WorkspaceAiActivityService
  -> FeatureBalanceThresholdService
  -> LessonGenerationValidationService
  -> FileService
  -> OpenAiFileService
      -> CourseSourceMaterialRepository
  -> LessonGenerationAsyncService
      -> WorkspaceAiConfigService
      -> AIProviderFactory -> OpenAIProvider/GeminiAIProvider
      -> AiResponseParser + semantic validator
      -> LessonRepository + BlockRepository
      -> ApiUsageService + BillingBalanceService (provider usage path)
      -> Notification service
      -> Optional LessonGenerationJobRepository
```

---

## 8) Backward Compatibility and Breaking Change Assessment

### Breaking Changes
- Expected none (feature is additive).
- No required modification to existing course-generation contracts.

### Backward Compatibility Risks
- Shared validator changes may unintentionally alter existing course flow behavior.
- Shared provider usage tagging may skew dashboards if feature labels are incorrect.
- If status endpoint is not implemented, async completion UX can be ambiguous.
- Inconsistent custom-key consumption representation may create reporting mismatch across endpoints.

---

## 9) Security and Compliance Impact

- JWT auth required for all lesson-generation endpoints.
- Workspace ownership validation mandatory for:
  - source file ids
  - session ids
  - optional module id
- Input sanitization needed for URL/text and generated block payload.
- Must not log sensitive data:
  - API keys
  - raw file contents
  - full prompt payloads

---

## 10) Risk Matrix (Aligned to RED)

### High
- Credit/deduction mismatch between platform and custom key modes.
- Duplicate async execution due to missing idempotency or retry guards.
- Authorization gaps enabling cross-workspace data access.

### Medium
- Semantic validator misses non-textual output leakage.
- Notification failure after successful persistence causes perceived failure.
- Executor saturation under high concurrency (extract/upload/provider/persist contention).

### Low
- Minor reporting legend inconsistencies for lesson-generation feature labels.

---

## 11) Operational and Observability Impact

### Pagination Impact
- No pagination required for:
  - add content
  - async generate start
  - retry start
- Pagination only applies if optional history/status listing APIs are introduced.
- If listing APIs are added later, align defaults/limits with RED guidance (`page`, `size`, `sort`).

### Logging Impact
- Request logs must include: endpoint, workspace/user/session context, source counts, and key generation settings (`lessonContentType`, `quizQuestionCount`).
- Response logs must include: status, success/failure, session id, job status, elapsed duration.
- Error logs must include categorized failures and provider metadata (without secrets).
- Performance logs should capture stage timings: ingest, extraction, upload, provider call, parse, semantic validation, persistence, notification.
- Logging policy must avoid sensitive payloads: raw file content, full prompt bodies, API keys.

### Performance Impact
- Async execution is mandatory to keep UI initiation non-blocking.
- Throughput pressure points:
  - file extraction and upload
  - provider latency/timeouts
  - persistence of lesson + block graph
- Requires bounded executors, queue protection, and retry-with-timeout strategy.
- Billing/usage recording must not delay async-start API response.

### Failure-Handling Impact
- Failure modes need explicit terminal states and retry policy:
  - provider timeout/downstream failure
  - malformed or non-textual AI output
  - quiz-count mismatch
  - persistence rollback scenarios
  - notification failure after successful persistence
- Duplicate active generation/retry must return conflict and block duplicate execution.
- Usage tracking failure must be treated as billing observability incident with compensating controls.

---

## 12) Recommended Implementation Guardrails

- Introduce lesson-generation-specific validation service and semantic validator.
- Add explicit feature code constants for usage/billing.
- Reuse existing custom-key bypass and usage-recording path, do not fork logic.
- Use bounded executor + workspace-level concurrency guard.
- Implement robust retry state machine (`FAILED` -> `IN_PROGRESS`; reject while running).
- Add structured logs with `workspaceId`, `userId`, `sessionId`, `jobStatus`, and stage timings.

---

## 13) Overall Impact Summary

The requirement introduces **moderate-to-high implementation impact** across controller, async orchestration, validation, and billing/usage integration layers, but remains **non-breaking and additive** if implemented with isolated lesson-generation APIs and explicit feature tagging.  
The most critical correctness areas are:
- strict `TEXTUAL_ONLY` semantic enforcement,
- idempotent async/retry lifecycle,
- and exact parity with existing platform-vs-custom-key billing behavior.

---

## 14) Scenario-ID Traceability (RED FS/EC)

### Functional Scenario Coverage

| RED ID | Impact location |
|---|---|
| FS-01 | `2) API Impact -> Request/Response Semantics Impact` |
| FS-02 | `1) Affected Modules -> Validation Layer`, `2) API Impact` |
| FS-03 | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts` |
| FS-04 | `5) Validation and Semantic Enforcement Impact` |
| FS-05 | `5) Validation and Semantic Enforcement Impact`, `2) API Impact` |
| FS-06 | `2) API Impact`, `6) Async and Retry Impact -> Async Start Contract` |
| FS-07 | `1) Affected Modules -> Services`, `6) Async and Retry Impact` |
| FS-08 | `1) Affected Modules -> Controllers/Services`, `3) Data and Database Impact` |
| FS-09 | `6) Async and Retry Impact`, `11) Operational and Observability Impact -> Failure-Handling Impact` |
| FS-10 | `4) Billing, Usage, and Key-Mode Impact -> Platform Key Mode` |
| FS-11 | `4) Billing, Usage, and Key-Mode Impact -> Custom/External Key Mode` |

### Edge Case Coverage

| RED ID | Impact location |
|---|---|
| EC-01 | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts` |
| EC-02 | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts` |
| EC-03 | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts` |
| EC-04 | `5) Validation and Semantic Enforcement Impact -> AI Output Validation Impacts` |
| EC-05 | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts` |
| EC-06 | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts`, `9) Security and Compliance Impact` |
| EC-07 | `1) Affected Modules -> Services`, `11) Operational and Observability Impact -> Failure-Handling Impact` |
| EC-08 | `11) Operational and Observability Impact -> Failure-Handling Impact`, `10) Risk Matrix` |
| EC-09 | `6) Async and Retry Impact -> Retry Semantics`, `10) Risk Matrix` |
| EC-10 | `4) Billing, Usage, and Key-Mode Impact -> Platform Key Mode` |
| EC-11 | `4) Billing, Usage, and Key-Mode Impact -> Custom/External Key Mode` |

---

## 15) Requirement Checklist Matrix (Final Verification)

| Requirement checkpoint (from RED) | Covered? | Impact location |
|---|---|---|
| 3-step workflow: Add Content -> Lesson Info -> Generate | Yes | `2) API Impact -> Request/Response Semantics Impact` |
| Async generate returns immediate start response | Yes | `2) API Impact`, `6) Async and Retry Impact` |
| Source inputs support files/urls/text with at-least-one rule | Yes | `5) Validation and Semantic Enforcement Impact` |
| Mandatory lesson fields (`lessonTitle`, `description`, `audience`) | Yes | `5) Validation and Semantic Enforcement Impact` |
| `lessonContentType` strict to `TEXTUAL_ONLY` | Yes | `5) Validation and Semantic Enforcement Impact` |
| `quizQuestionCount` optional but constrained to `1..6` | Yes | `5) Validation and Semantic Enforcement Impact` |
| `toneStyle` optional and restricted to allowed set when provided | Yes | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts` |
| `sessionId` optional at ingest but required for later generate/retry/status lifecycle | Yes | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts`, `6) Async and Retry Impact` |
| Optional `moduleId` must pass existence and workspace ownership checks | Yes | `5) Validation and Semantic Enforcement Impact -> Input Validation Impacts`, `9) Security and Compliance Impact` |
| Retry support for failed attempts with conflict protection | Yes | `6) Async and Retry Impact` |
| Semantic output validation and no partial publish | Yes | `5) Validation and Semantic Enforcement Impact` |
| Post-success availability in lesson editor flow | Yes | `1) Affected Modules`, `3) Data and Database Impact` |
| Platform key mode: normal cost + deduction + reporting | Yes | `4) Billing, Usage, and Key-Mode Impact -> Platform Key Mode` |
| Custom key mode: usage trail kept, zero-effective deduction | Yes | `4) Billing, Usage, and Key-Mode Impact -> Custom/External Key Mode` |
| API error families (400/401/403/404/409/422/429/5xx) reflected | Yes | `2) API Impact -> Request/Response Semantics Impact` |
| Pagination not required for add/generate/retry | Yes | `11) Operational and Observability Impact -> Pagination Impact` |
| Logging requirements (request/response/error/perf + safe logging) | Yes | `11) Operational and Observability Impact -> Logging Impact` |
| Performance requirements for async and bounded execution | Yes | `1) Affected Modules -> Async Execution`, `11) Performance Impact` |
| Security requirements (authz/workspace ownership/sanitization) | Yes | `9) Security and Compliance Impact` |
| Failure scenario handling and retryability expectations | Yes | `11) Failure-Handling Impact`, `10) Risk Matrix` |

