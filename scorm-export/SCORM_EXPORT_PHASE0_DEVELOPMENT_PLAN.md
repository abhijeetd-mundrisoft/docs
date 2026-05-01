# Phase 0 — Development Plan (SCORM course export foundation)

**Goal:** Add safe extension points for lesson HTML (and optionally media collection) so Phase 1+ can register new block renderers **without** changing today’s exported output for existing supported types.

**References:** `SCORM_EXPORT_SOLUTION_DESIGN.md`, `SCORM_EXPORT_PHASE_WISE_DEVELOPMENT_PLAN.md` (Phase 0), Jira ticket text in `SCORM_EXPORT_JIRA_TASKS.md` (Ticket 1).

---

## 1. Scope

### In scope

- Registry (or factory) for **optional** per–`BlockType` HTML renderers.
- **Both** `Scorm12ExportStrategy` and `Scorm2004ExportStrategy` use the same resolution rule: renderer if present and successful → else **existing** `switch` in `generateBlockHtml`.
- Automated tests proving **no regression** for at least one already-supported block type.
- (Optional) Structured logging when renderer path fails before fallback.
- (Optional) Same “extension first, legacy fallback” pattern for **media file ID** collection used when copying assets into the ZIP.

### Out of scope

- Any **new** `BlockType` output in export (Phase 1+).
- API changes to `POST /export/course` or response contract.
- SCORM editor flows (different feature area).
- Package / class **renames** of existing public types (incremental change only).

---

## 2. Prerequisites

- [ ] Agreed **fixture course** (or minimal in-memory fixture) used for manual + automated checks.
- [ ] Baseline: export that course **before** changes; store HTML snippet or ZIP hash / checklist (team choice) for comparison **after**.

---

## 3. Work breakdown (recommended order)

| Step | Task | Done when |
|------|------|-----------|
| **0.1** | Add `BlockRenderer` (or equivalent) interface: `BlockType` + `render(Block, ScormBlockRenderContext)` returning HTML fragment. | Compiles; no wiring to strategies yet. |
| **0.2** | Add `ScormBlockRenderContext` (or reuse minimal DTO): export/course/lesson/module ids, `blockNumber`, `ObjectMapper`, escape/sanitize callbacks as needed by renderers. | Both strategies can construct it without duplicating large logic. |
| **0.3** | Add `ScormBlockRendererRegistry` / factory: `List<BlockRenderer>` → `Map<BlockType, BlockRenderer>`; **fail startup** (or test) if two renderers claim same type. | Spring bean; duplicate registration covered by test. |
| **0.4** | Refactor **shared** dispatch: e.g. `ScormBlockHtmlDispatcher` (or package-private helper) used by **both** `Scorm12ExportStrategy` and `Scorm2004ExportStrategy` so 1.2 and 2004 cannot drift. | Single code path for “try renderer → legacy switch”. |
| **0.5** | Change `generateBlockHtml` in both strategies to call dispatcher first, then existing switch (move switch body to `generateBlockHtmlLegacy` private method if cleaner). | **Zero** registered renderers → output identical to pre-Phase-0 for all types. |
| **0.6** | (Optional) Media: `BlockMediaExtractor` registry + legacy `extractFileIdsFromBlockContent` fallback; same dual-strategy usage pattern. | With no extractors registered, same file set as today for fixture course. |
| **0.7** | (Optional) Logging: warn with export + course + lesson + block + type when renderer throws; no secrets. | One reviewed log line in test/stage. |
| **0.8** | Tests: (a) duplicate renderer registration, (b) legacy path for e.g. `TEXT`/`TITLE` unchanged, (c) optional integration smoke calling dispatcher with empty registry. | CI green. |
| **0.9** | Manual: export fixture **SCORM 1.2** and **SCORM 2004**; LMS or SCORM Cloud import; spot-check lesson + media. | Evidence attached to Jira / Confluence. |

---

## 4. Likely code locations (adjust after first spike)

| Area | Location (typical) |
|------|---------------------|
| SCORM 1.2 block HTML | `course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/strategy/Scorm12ExportStrategy.java` — `generateBlockHtml` |
| SCORM 2004 block HTML | `course-forge-backend/src/main/java/com/mundrisoft/courseforge/export/strategy/Scorm2004ExportStrategy.java` — `generateBlockHtml` |
| New types | Prefer **same package** as strategies or `.../export/render/` under existing `export` tree — **no** new top-level module unless team agrees. |
| Spring registration | `@Component` renderers + constructor injection of registry, or `@Bean` factory method in existing export config if you have one. |

---

## 5. Testing strategy (Phase 0)

| Layer | What to prove |
|-------|----------------|
| **Unit** | With **empty** registry, `generateBlockHtml` output for `TEXT` (or `TITLE`) matches pre-refactor string (golden) or normalized form. |
| **Unit** | Duplicate `BlockType` in two renderer beans → startup failure or test assertion. |
| **Integration** (if feasible) | Full `export()` or package build with fixture course completes for 1.2 and 2004; no new renderer beans. |
| **Manual** | Same as integration if automated export test is heavy — minimum once per release of Phase 0. |

---

## 6. Definition of done (Phase 0)

- [ ] Both SCORM strategies use shared dispatch; **no** new block types shipped.
- [ ] CI tests above pass; no regression on chosen golden block(s).
- [ ] Manual dual SCORM check on agreed fixture (or documented waiver with lead approval).
- [ ] Jira Phase 0 ticket closed; Phase 1 branch can start.

---

## 7. Risks and mitigations

| Risk | Mitigation |
|------|------------|
| 1.2 vs 2004 behavior drift | One shared dispatcher/helper; both strategies delegate to it. |
| Accidental HTML change | Golden test on legacy path; PR review focused on `generateBlockHtml` diff. |
| Spring circular dependency | Keep registry depending only on `List<BlockRenderer>`; strategies inject registry only. |

---

## 8. Suggested timeline (indicative)

| Horizon | Outcome |
|---------|---------|
| **0.5–1 dev-day** | Interfaces + registry + wiring; still no behavior change. |
| **+0.5–1 dev-day** | Tests + optional logging + optional media hook. |
| **+0.5 day** | QA manual dual SCORM + Jira sign-off. |

---

## 9. Document control

| Version | Date | Notes |
|---------|------|--------|
| 1.0 | 2026-04-29 | Initial Phase 0 development plan. |
