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

## 6) What Not to Enforce Retroactively

The following are desired long-term improvements, but must not be forced as blanket refactor on existing code:

- Full DTO-only conversion for every legacy endpoint.
- Full exception-model unification in one release.
- Full security architecture rewrite in unrelated tickets.
- Large repository/query rewrites without measured bottleneck evidence.

## 7) PR Checklist for New Code

- Does this change preserve backward compatibility?
- Are request/response DTOs used for new JSON APIs?
- Is validation annotation-based and complete?
- Are exceptions typed and centrally handled?
- Are transactions and query patterns performance-safe?
- Are logs useful and free from sensitive data?
- Are tests added/updated for new behavior?

