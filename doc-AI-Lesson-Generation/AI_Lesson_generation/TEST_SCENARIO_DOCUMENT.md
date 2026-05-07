# Test Scenario Document (TSD)

**Feature:** AI Lesson Generation (Revised)  
**Requirement Source:** `doc/AI_Lesson_generation/REQUIREMENT_EXPANSION_DOCUMENT.md`

---

#1. Functional Test Cases
Cover all valid scenarios.

| TC ID | Scenario | Preconditions | Test Steps | Expected Result |
|---|---|---|---|---|
| FT-01 | Add content with files only | Valid JWT and workspace context | Call `POST /lesson/generation/content` with valid files | `200`, content accepted, `sessionId` returned, `fileIds` returned |
| FT-02 | Add content with URLs only | Valid JWT | Call content API with valid URL array only | `200`, `sessionId` returned, `sourceSummary.urlCount > 0` |
| FT-03 | Add content with text only | Valid JWT | Call content API with non-empty text only | `200`, `sessionId` returned, `sourceSummary.hasText=true` |
| FT-04 | Add mixed sources | Valid JWT | Call content API with files + URLs + text | `200`, source summary reflects all provided source types |
| FT-05 | Generate lesson async start (valid request) | Valid `sessionId`, source references, valid lesson fields | Call `POST /lesson/generation/generate` | `202`, immediate start response with `sessionId` and `jobStatus=IN_PROGRESS` |
| FT-06 | Generate with `TEXTUAL_ONLY` and quiz count | Valid request with `lessonContentType=TEXTUAL_ONLY`, `quizQuestionCount=4` | Start generation and wait completion | Lesson generated successfully with semantically textual blocks and quiz aligned to requested count |
| FT-07 | Generate with optional `toneStyle` present | Valid request with supported `toneStyle` | Start generation | Request accepted and processed |
| FT-08 | Generate with optional `quizQuestionCount` omitted | Valid request without quiz count | Start generation | Request accepted; default quiz behavior applies per implementation |
| FT-09 | Retry failed generation | Existing failed session | Call `POST /lesson/generation/retry/{sessionId}` | `202`, retry starts for failed session |
| FT-10 | Post-success lesson usability | Successful generation completed | Open lesson in editor and preview | Lesson opens; blocks editable; preview works |
| FT-11 | Platform key billing success flow | Platform key mode active | Run successful generation | Usage recorded, cost deducted, transaction + consumption summary updated |
| FT-12 | Custom key billing success flow | Custom/external key mode active | Run successful generation | Usage/transaction trail exists with zero-effective deduction |

---

#2. Edge Case Tests
Include boundary values and rare conditions.

| TC ID | Edge Case | Input/Condition | Expected Result |
|---|---|---|---|
| EC-01 | Quiz lower boundary | `quizQuestionCount=1` | Accepted |
| EC-02 | Quiz upper boundary | `quizQuestionCount=6` | Accepted |
| EC-03 | Minimum valid source payload | Exactly one valid source (single file OR one URL OR text) | Accepted |
| EC-04 | OpenAI file upload partial failure | Provider file upload fails but extracted text exists | Flow continues via extracted-text path if supported; warning logged |
| EC-05 | Notification failure after persistence | Force notification subsystem failure | Lesson remains persisted; notification failure logged and retry path invoked |
| EC-06 | Retry while running | Retry requested when job status is `IN_PROGRESS` | Conflict response (`409`) and no duplicate execution |
| EC-07 | Platform BLOCK with no custom key | Low credit threshold status BLOCK | Generation blocked before provider call |
| EC-08 | Platform BLOCK with custom key active | Low platform balance + custom key mode | Generation allowed via custom-key bypass |
| EC-09 | Semantic output mismatch | AI returns non-text/interactive/video block for textual mode | Semantic validation fails (`422`), no partial publish |

---

#3. Negative Test Cases
Invalid inputs, missing fields, incorrect formats.

| TC ID | Scenario | Invalid Input | Expected Result |
|---|---|---|---|
| NT-01 | Missing auth header | No Authorization header | `401` |
| NT-02 | Expired token | Expired JWT | `401` |
| NT-03 | No source provided | Empty files/urls/text | `400` validation error |
| NT-04 | Missing lesson title | blank `lessonTitle` | `400` |
| NT-05 | Missing description | blank `description` | `400` |
| NT-06 | Missing audience | blank `audience` | `400` |
| NT-07 | Invalid tone style | unsupported `toneStyle` | `400` |
| NT-08 | Unsupported content type | `lessonContentType != TEXTUAL_ONLY` | `400` |
| NT-09 | Invalid quiz count low | `quizQuestionCount=0` | `400` |
| NT-10 | Invalid quiz count high | `quizQuestionCount=7` | `400` |
| NT-11 | Invalid quiz type | non-numeric quiz count | `400` |
| NT-12 | Invalid module ownership | `moduleId` belongs to another workspace | `403` |
| NT-13 | Invalid/unknown source refs | `fileIds` not in caller workspace/session | `400/403/404` per contract |
| NT-14 | Invalid URL format | malformed URL payload | `400` |
| NT-15 | Oversized file payload | file/total size beyond limit | `413` or `400` based on enforcement point |
| NT-16 | Provider timeout/failure | downstream AI timeout/error | `502/504` mapped error, failed state recorded |

---

#4. Validation Test Cases
Field-level and business rule validations.

## Field-level validations

| TC ID | Field | Test Data | Expected Result |
|---|---|---|---|
| VT-01 | `files/urls/text` | all absent | Reject |
| VT-02 | `lessonTitle` | blank/whitespace | Reject |
| VT-03 | `description` | blank/whitespace | Reject |
| VT-04 | `audience` | blank/whitespace | Reject |
| VT-05 | `toneStyle` | valid allowed value | Accept |
| VT-06 | `toneStyle` | unsupported value | Reject |
| VT-07 | `lessonContentType` | `TEXTUAL_ONLY` | Accept |
| VT-08 | `lessonContentType` | unsupported value | Reject |
| VT-09 | `quizQuestionCount` | 1, 6 | Accept |
| VT-10 | `quizQuestionCount` | 0, 7, negative, non-numeric | Reject |
| VT-11 | `sessionId` | omitted at ingest | Server issues session id |
| VT-12 | `sessionId` | missing/invalid at generate/retry stage | Reject |
| VT-13 | `moduleId` | valid module in workspace | Accept |
| VT-14 | `moduleId` | invalid/non-owned | Reject/forbid |

## Business rule validations

| TC ID | Rule | Test | Expected Result |
|---|---|---|---|
| VT-15 | Auth + workspace required | valid user, invalid workspace context | Reject |
| VT-16 | Source ownership | use foreign workspace source IDs | Reject |
| VT-17 | Textual semantic enforcement | AI returns non-textual content | Semantic rejection (`422`) |
| VT-18 | Quiz count consistency | generated quiz count != requested count | Semantic rejection (`422`) |
| VT-19 | Retry idempotency safety | duplicate retry attempts | No duplicate persistence; conflict behavior enforced |
| VT-20 | Billing mode behavior | platform vs custom key | Correct deduction behavior per mode |

---

#5. API Test Cases
Include:
- request/response validation
- status codes
- error handling

## Request/Response validation

| TC ID | API | Validation Focus | Expected Result |
|---|---|---|---|
| API-01 | `POST /lesson/generation/content` | Request schema + mixed payload support | Valid requests accepted; response envelope fields present (`success`, `status`, `message`, `data`, `timestamp`) |
| API-02 | `POST /lesson/generation/generate` | Required JSON fields + enums | Valid request returns start payload with `sessionId`, `jobStatus` |
| API-03 | `POST /lesson/generation/retry/{sessionId}` | Path param + request body | Valid failed session retry returns `202` start payload |

## Status codes

| TC ID | API | Condition | Expected Status |
|---|---|---|---|
| API-04 | content | success | `200` |
| API-05 | generate | async start success | `202` |
| API-06 | retry | retry start success | `202` |
| API-07 | all | validation failure | `400` |
| API-08 | all | unauthorized | `401` |
| API-09 | generate | forbidden module/workspace access | `403` |
| API-10 | generate/retry | not found session/source/module | `404` |
| API-11 | generate/retry | conflict on active request | `409` |
| API-12 | generate | semantic validation failure | `422` |
| API-13 | generate | rate limit (if enabled) | `429` |
| API-14 | generate | provider timeout/failure | `502/504` |
| API-15 | all | internal server error | `500` |

## Error handling

| TC ID | Scenario | Expected Result |
|---|---|---|
| API-16 | Business validation error payload | Structured error response with safe message |
| API-17 | Provider exception mapping | Correct mapped status (`502/504`) and safe error body |
| API-18 | Sensitive data leakage check | No API key/raw file/full prompt leakage in response |
| API-19 | Correlation/session traceability | Error and start payload contain session context for tracking |

---

#6. Performance Test Scenarios
large data load  
concurrent users

| TC ID | Scenario | Load Profile | Success Criteria |
|---|---|---|---|
| PF-01 | Large source ingestion | Near-max file size and URL/text limits | Validation and processing stable; proper reject when exceeding limits |
| PF-02 | Concurrent content ingestion | High parallel content requests | Stable throughput; no crash/deadlock |
| PF-03 | Concurrent generate starts | High parallel generate requests | Quick `202` response; queue remains bounded |
| PF-04 | Async worker saturation | Sustained generation load | No starvation; controlled failure/timeout behavior |
| PF-05 | Provider latency spike | Inject slow provider | Requests fail gracefully with mapped errors and retryable state |
| PF-06 | Persistence under concurrency | Multiple concurrent successful jobs | No partial writes; transactional integrity preserved |

---

#7. Security Test Scenarios
unauthorized access  
input tampering  
injection attempts

| TC ID | Scenario | Test | Expected Result |
|---|---|---|---|
| ST-01 | Unauthorized access | Call APIs without JWT | `401` |
| ST-02 | Expired/invalid JWT | Use invalid token | `401` |
| ST-03 | Cross-workspace tampering | Send foreign `sessionId`/`fileIds`/`moduleId` | `403/404` and no data exposure |
| ST-04 | Input tampering | Modify request fields to bypass `TEXTUAL_ONLY` | Rejected by validation |
| ST-05 | URL injection attempt | Malicious URL payloads | Sanitized/rejected based on URL validation policy |
| ST-06 | Text injection attempt | SQL/script/prompt injection strings in text fields | No server-side execution; controlled handling |
| ST-07 | File upload tampering | Unsupported/disguised file type | Rejected by file validation |
| ST-08 | Error disclosure hardening | Trigger backend/provider failures | No sensitive internal details in API response |
| ST-09 | Log safety check | Trigger failures with sensitive input | Logs do not include API keys/raw file content/full prompts |

---

**Coverage Note:** This document provides full RED-aligned coverage with unique test scenarios across functional, edge, negative, validation, API, performance, and security dimensions.

