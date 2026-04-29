# Course Forge Backend Rules

This document defines safe rules for ongoing development in `course-forge-backend`.
It is intentionally hybrid: preserve existing behavior in legacy areas, enforce stronger standards for new code, and migrate incrementally without breaking production behavior.

## 0) Project Reality Snapshot (Observed Baseline)

- Architecture is primarily layered Spring Boot: `Controller -> Service -> Repository -> Entity/DB`.
- Existing API style mostly uses `ResponseEntity<ApiResponseDto<T>>`, with some exceptions (`Map<String, Object>` errors and stream/file responses).
- Validation uses Jakarta Bean Validation in many places, but not uniformly.
- Exception handling exists in global handlers and local `try/catch`, which creates mixed patterns.
- Data access is Spring Data JPA with a combination of derived queries and custom `@Query` (JPQL/native).

These rules must not force breaking changes in old modules.

## 1) Critical Safety Rules (Non-Negotiable)

1. Do not break existing APIs.
   - No response contract changes for existing endpoints unless explicitly approved and versioned.
   - Any breaking API change (status codes, payload keys, semantics, removal) MUST ship on a new versioned path; see **§8) API Versioning Rules**.
   - Keep existing status codes and payload keys stable for current clients.
2. Do not change existing public method signatures in controllers/services used by current flows unless explicitly requested.
3. Do not introduce DB schema-breaking changes.
   - No destructive migration (drop/rename) without approved migration plan.
   - Additive schema changes only by default.
4. Maintain backward compatibility.
   - New behavior must be opt-in (feature flag, new endpoint, or fallback-safe path).
5. Do not rewrite legacy modules as part of unrelated tickets.
6. Security and secrets:
   - Never add hardcoded credentials, API keys, or secrets in code or properties.
   - New secrets must come from environment or approved secret management.
   - Extended security constraints for new code: **§11) Security Rules (Extended)**.

## 2) Existing Code Guidelines (When Touching Old Code)

1. Match local style before improving style.
   - Preserve current patterns in the file/module unless changes are requested.
2. Keep edits minimal and scoped.
   - Fix the requested issue without broad refactor.
3. Avoid moving logic across layers in legacy code unless required for bug fix or explicitly requested.
4. Preserve current response shape in touched endpoints.
   - If an endpoint currently returns `ApiResponseDto`, keep it.
   - If it currently returns stream/resource for downloads, keep it.
5. If an old area has ad-hoc exception handling, do not rewrite all of it in one go.
   - Apply improvements only within touched code and only if safe.
6. When adding to legacy services/repositories, reuse existing transaction/query patterns there.

## 3) Strict Rules for New Code

These are mandatory for all newly created endpoints/services/repositories/classes.
These rules are not retroactive for existing endpoints/classes unless explicitly approved, versioned, and client-safe.

### 3.1 Architecture and Layering

1. Use strict layering:
   - `Controller`: HTTP contract, validation trigger, auth context extraction.
   - `Service`: business rules, orchestration, transaction boundary.
   - `Repository`: persistence only.
2. Controllers must not contain business logic or repository calls.
3. New cross-module business flow must start in service layer, not utility class.

### 3.2 API Contract and DTO Rules

1. New REST endpoints must use request/response DTOs.
2. Do not expose JPA entities directly in new API responses.
3. Use `ResponseEntity<ApiResponseDto<T>>` for new JSON APIs.
4. For binary/file download endpoints, return resource/stream responses; keep metadata in headers.
5. If integrating third-party SDKs (for example payment objects), map to internal response DTOs instead of returning SDK models.

### 3.3 Validation Rules

1. Validate all new request bodies with `@Valid`.
2. Use field-level constraints in DTOs (`@NotBlank`, `@Size`, `@Email`, etc.) rather than manual null/empty checks in controller.
3. Path/query/header constraints should use method-level validation where possible.
4. Service-layer invariant checks are still required for business rules not expressible via bean validation.

### 3.4 Exception Handling Rules

1. New code must use typed custom exceptions for business/domain failures.
2. Do not use broad `catch (Exception)` in controllers for normal control flow.
3. New exceptions should be handled by centralized advice and converted into stable `ApiResponseDto` error format.
4. Include actionable error messages and avoid leaking stack traces/internal implementation details in API responses.

### 3.5 Logging Rules

1. Use `@Slf4j` and structured, contextual logs.
2. Log at boundaries:
   - `info` for major state transitions,
   - `warn` for recoverable anomalies,
   - `error` for failures with context.
3. Never log secrets, tokens, passwords, or raw sensitive payloads.
4. Avoid noisy per-item logs in loops unless debug-level and justified.
5. Observability requirements for new flows: **§12) Observability Rules**.

### 3.6 Database and Transaction Rules

1. New writes must execute inside service-layer `@Transactional` boundaries.
2. Use `@Transactional(readOnly = true)` for read-only service methods when appropriate.
3. Avoid N+1 query patterns in new data access code.
   - Prefer fetch joins/projections/batch strategies when reading graph data.
4. Add pagination for list endpoints expected to grow.
5. Prefer repository methods that return only needed columns/objects for heavy queries.

### 3.7 Naming and Code Quality

1. Follow existing naming convention: `*Controller`, `*Service`, `*Repository`, `*Dto`, `*Exception`.
2. New class names must be meaningful and versionless (no numeric suffixes like `Util3`).
3. Keep utility classes stateless and domain-neutral.
4. Add focused unit/integration tests for new logic paths.

## 4) Migration Strategy (Incremental, Non-Breaking)

1. Standardize response contracts gradually.
   - New endpoints: strict `ApiResponseDto<T>`.
   - Existing mixed endpoints: keep current contract; migrate only with explicit versioning or client agreement.
2. Introduce adapters/wrappers for legacy behavior.
   - Wrap legacy service outputs into DTOs at boundary layer without changing internal logic first.
3. Use deprecation path for old endpoints/methods.
   - Mark as deprecated, document replacement, keep overlap period, then remove only with approval.
4. Move controller-heavy logic to services opportunistically.
   - Only when touching that flow for feature/bug work.
5. Consolidate exception handling over time.
   - Route new modules to centralized advice first, then gradually align legacy modules.
6. Security hardening migration:
   - Replace hardcoded secrets with env-backed config incrementally, starting with highest-risk keys.

## 5) Performance Rules (Important)

These performance rules are mandatory for new development and optional for legacy flows unless explicitly requested.

1. No unbounded list fetches in new endpoints.
   - Use pagination (`page`, `size`, optional sorting) by default.
2. Avoid repeated DB calls inside loops.
   - Batch fetch related records and map in memory.
3. Use targeted queries/projections for read-heavy endpoints.
4. Prefer bulk operations when updating/deleting multiple rows.
5. For expensive workflows (exports, AI generation, conversion), run asynchronously where appropriate and return trackable status.
6. Profile before optimizing legacy hotspots.
   - Keep performance fixes evidence-based (query timings, endpoint latency, logs/metrics).
7. Async and long-running work MUST follow **§9) Async Processing Rules**; caching MUST follow **§10) Caching Rules (Redis)** where applicable.

## 6) What Not to Enforce Retroactively

The following are desired long-term improvements, but must not be forced as blanket refactor on existing code:

- Full DTO-only conversion for every legacy endpoint.
- Full exception-model unification in one release.
- Full security architecture rewrite in unrelated tickets.
- Large repository/query rewrites without measured bottleneck evidence.

## 7) PR Checklist for New Code

- Does this change preserve backward compatibility?
- Are breaking API changes only on versioned paths per **§8**?
- Are request/response DTOs used for new JSON APIs?
- Is validation annotation-based and complete?
- Are exceptions typed and centrally handled?
- Are transactions and query patterns performance-safe?
- Are long-running or heavy operations async per **§9**?
- Is Redis caching (if any) service-layer only with TTL/invalidation per **§10**?
- Are logs useful, structured, and free from sensitive data per **§12**?
- Are third-party calls behind adapters with timeouts/retries per **§13**?
- Are critical writes idempotent or duplicate-safe per **§14**?
- Are public or abuse-prone endpoints throttled or designed for limits per **§15**?
- Does new code live under the package layout in **§16**?
- Are unit tests (service) and integration tests (where applicable) added or updated per **§17**?
- Are authorization checks applied at the controller/service boundary per **§11**?

## 8) API Versioning Rules

1. Existing unversioned paths MUST NOT change in breaking ways.
2. Any breaking API change MUST be introduced only on a new versioned base path.
3. Version format MUST be a path prefix: `/api/v{n}/...` or project-consistent `/v{n}/...` matching existing conventions; MUST NOT version via query string alone.
4. Old version endpoints MUST remain callable until an approved deprecation and removal window.
5. New clients MUST target the newest stable version; AI MUST NOT collapse multiple versions into one response shape without explicit approval.

## 9) Async Processing Rules

1. Controllers MUST NOT block request threads for work that routinely exceeds acceptable gateway timeout (large file processing, full-course exports, bulk transforms, long external API chains).
2. For large file processing, exports, and orchestrated external API work, AI MUST prefer: enqueue job → process in `@Async` worker / message listener / scheduler → persist status → client polls or receives callback/webhook where applicable.
3. Synchronous controller methods MUST return quickly with a resource identifier, status URL, or stream only when the payload is already prepared and bounded.
4. Background processors MUST use the same service-layer transaction and idempotency patterns as synchronous flows.
5. Legacy endpoints MUST NOT be switched from sync to async without versioning or explicit approval (backward compatibility).

## 10) Caching Rules (Redis)

1. Caching MUST be applied only in the **service layer** (or dedicated cache service beans called from services); MUST NOT be implemented in controllers or repositories as the primary cache boundary.
2. AI MUST define explicit TTL for every cache key; MUST NOT introduce unbounded permanent caches for mutable domain data without invalidation rules.
3. AI MUST document or implement invalidation on writes that affect cached reads (update/delete path clears or refreshes affected keys).
4. Cache keys MUST be namespaced to avoid collisions; MUST NOT embed secrets in keys.
5. Read-heavy, stable-scoped endpoints MAY use caching; write-heavy or strongly consistent reads MUST NOT rely on stale cache without acceptance criteria.

## 11) Security Rules (Extended)

1. API responses and logs MUST NOT include secrets, tokens, refresh tokens, passwords, full payment PAN, or full raw PII; identifiers in logs and APIs MUST be opaque or minimized to what the operation requires.
2. AI MUST assume token-based authentication for new endpoints; MUST NOT add anonymous public write surfaces without explicit requirement and threat review.
3. Hardcoded secrets are forbidden (reiterates **§1.6**); configuration MUST resolve from environment or approved secret stores only.
4. Authorization MUST be enforced at the boundary: controller method (role/scope check) and/or service method (ownership, tenant, resource ACL) for every mutating and sensitive read path; MUST NOT rely on obscurity or client-only checks.
5. New endpoints MUST validate input per **§3.3** and MUST NOT trust client-supplied identifiers without server-side ownership checks where applicable.

## 12) Observability Rules

1. New service entry points for non-trivial operations MUST emit one structured `info` (or `debug` where high volume) log at start with correlation/trace identifiers when available.
2. Successful completion of non-trivial operations MUST emit one structured `info` log at exit with outcome summary (counts, ids), without sensitive payloads.
3. Errors MUST be logged at `error` with exception type and message; API responses MUST NOT include stack traces or internal diagnostics per **§3.4**.
4. Loops MUST NOT log per iteration at `info`/`warn`; use aggregated counters or `debug` with sampling justification.
5. Reuse **§3.5** for levels and secret prohibition; this section adds mandatory entry/exit/error discipline for new flows.

## 13) External Integration Rules

1. All third-party HTTP/SDK calls MUST go through a dedicated adapter or integration service; controllers MUST NOT call vendor SDKs directly.
2. Third-party request/response DTOs MUST NOT be returned from application REST APIs; map to internal DTOs at the adapter boundary.
3. Every outbound integration MUST use configured connect and read timeouts; MUST NOT rely on infinite default timeouts.
4. Retries MUST be bounded (max attempts, backoff, jitter) and MUST NOT retry non-idempotent operations unless explicitly safe; respect vendor rate limits.

## 14) Idempotency Rules

1. APIs for submissions, payments, refunds, fulfillment, and retry-prone mutations MUST be designed duplicate-safe: idempotency keys, natural unique constraints, or safe replays documented in code.
2. AI MUST use database unique constraints or transactional checks to prevent double application of the same logical operation when clients retry.
3. Idempotency tokens (headers or body fields) MUST be validated in the service layer; duplicate requests MUST return the same logical outcome as the first success without duplicate side effects.
4. Legacy flows without idempotency MUST NOT be altered globally; new endpoints or new write paths MUST comply.

## 15) Rate Limiting Rules

1. AI MUST assume public or unauthenticated-adjacent endpoints may be throttled at the edge; implementations MUST NOT depend on unbounded client goodwill.
2. New public list/search/export endpoints MUST use pagination, caps, or job-based access per **§5** and **§9** to reduce abuse surface.
3. AI MUST NOT generate tight loops of unauthenticated endpoints that enable scraping or resource exhaustion without mitigation (auth, rate limits, or async job pattern).

## 16) Project Structure Rules (New Code)

1. New types MUST reside under the existing root package, in standard packages: `...controller`, `...service` (or `...service.impl` if project uses it), `...repository`, `...dto` (or `...model.dto`), `...exception`, `...config`, matching existing module layout.
2. New adapters for external systems MUST live under `...integration`, `...client`, or `...adapter` (pick one convention already present in the module); MUST NOT place vendor clients in `controller`.
3. Cross-cutting concerns (security, cache, observability config) MUST use `...config` or established infrastructure packages, not ad-hoc packages at root.

## 17) Testing Rules

1. New business logic in services MUST have unit tests covering success, domain failure, and critical edge cases.
2. New repositories or query methods with non-trivial JPQL/native SQL MUST have integration tests against a test database or Testcontainers when the project already uses that pattern; if no pattern exists, add the minimum integration test that proves the query contract.
3. AI MUST NOT satisfy coverage only with thin controller tests that mock the entire service graph; service-layer tests are mandatory for new rules-heavy code.
4. Tests MUST NOT embed real secrets; use test fixtures and test properties only.

