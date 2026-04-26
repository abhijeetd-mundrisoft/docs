# Functional Context Document - `course-forge-backend`

## A. System Overview

The application is a backend platform for **AI-assisted course authoring and SCORM operations**.

From a business perspective, it serves four connected goals:
- Manage users and organizations (workspaces) with role-based capabilities.
- Allow teams to create and maintain structured learning content (courses, modules, lessons, blocks).
- Provide AI-assisted content generation with usage metering and credit-based billing.
- Support SCORM lifecycle operations: convert source files to SCORM, edit SCORM packages, preview, and export.

**Business domain:** EdTech / LMS content authoring and distribution platform (with multi-tenant workspace management and monetized AI usage).

## B. Business Modules

1. **Identity and Access Management**
   - Handles login, token validation, password reset/change, and account status transitions.
   - Ensures users can only operate within valid workspaces and allowed roles.

2. **Workspace and Subscription Governance**
   - Manages workspace plan state (Freemium, Individual, etc.), subscription change requests, and approvals.
   - Enforces org-level limits and entitlement behavior.

3. **Organization Administration**
   - Allows tenant admins to manage organization profile, billing contact data, subscription info, and invoice retrieval.

4. **Course Authoring (Core Content)**
   - Supports creation and maintenance of course hierarchy:
     `Course -> Module -> Lesson -> Block`.
   - Includes template-based course creation and content edits/reordering/status updates.

5. **AI Course Generation and AI Utilities**
   - Generates course metadata, outlines, and full courses from source text/files/URLs.
   - Supports text/image generation utilities while enforcing usage and threshold policies.

6. **Usage Metering and Credit Wallet**
   - Tracks AI usage and cost attribution.
   - Maintains workspace balance and transaction ledger.

7. **Payments and Billing Events**
   - Uses payment session creation + webhook lifecycle to top up workspace AI credits.
   - Protects against duplicate or invalid payment state transitions.

8. **SCORM Conversion**
   - Converts supported input formats (e.g., PDF/PPTX/MP4/DOCX) to SCORM with sync and async flows.
   - Supports authenticated and public API-key based consumers.

9. **SCORM Editor**
   - Enables upload of SCORM ZIP, slide-level text/media editing, preview serving, and downloadable export.

## C. Business Flow Mapping

### Flow 1: User Login and Workspace Eligibility
- **Flow name:** User authentication with organization eligibility checks
- **Step-by-step process:**
  1. User submits credentials.
  2. System validates identity and account state.
  3. System verifies workspace eligibility (active membership/non-expired context for applicable users).
  4. JWT/session context is issued for authorized access.
  5. User may call token validation to keep session trust current.
- **APIs involved:**
  - `POST /auth/login`
  - `POST /auth/validate`
- **Business rules applied:**
  - Non-active account states are blocked.
  - Expired/ineligible workspace context prevents successful access for affected roles.
  - Password-reset-required states force password lifecycle completion.
- **Input -> Output transformation:**
  - **Input:** username/email + password (+ existing token on validate).
  - **Output:** authenticated user context (token, role/workspace eligibility), or explicit denial reason.

### Flow 2: Subscription Upgrade Request to Approval
- **Flow name:** Workspace plan change governance
- **Step-by-step process:**
  1. Tenant user submits subscription change request.
  2. System checks request eligibility and prevents duplicates.
  3. Super admin reviews pending requests.
  4. Super admin approves/rejects request.
  5. On approval, workspace plan and limits are switched and initialized.
- **APIs involved:**
  - `POST /workspaces/subscription-change-request`
  - `GET /admin/subscription-requests/pending`
  - `POST /admin/subscription-requests/{requestId}/approve`
  - `POST /admin/subscription-requests/{requestId}/reject`
  - `POST /workspaces/switch-subscription/{workspaceId}`
- **Business rules applied:**
  - Only eligible source plans can request upgrade.
  - Duplicate pending requests are rejected.
  - Final state changes are role-gated (admin approval).
- **Input -> Output transformation:**
  - **Input:** workspace request intent + admin decision.
  - **Output:** updated request status plus changed workspace plan/entitlements (on approval).

### Flow 3: Manual Course Authoring Lifecycle
- **Flow name:** Create and enrich course structure
- **Step-by-step process:**
  1. Author creates a course.
  2. Author adds modules.
  3. Author adds lessons under modules.
  4. Author adds/updates blocks inside lessons.
  5. Author fetches full course details for publishing/review.
- **APIs involved:**
  - `POST /courses`
  - `POST /modules`
  - `POST /lessons`
  - `POST /blocks` (plus update/reorder/status APIs)
  - `GET /courses/{id}/details`
- **Business rules applied:**
  - Workspace course limits must not be exceeded.
  - Hierarchy integrity must be preserved (valid parent course/module/lesson).
  - Content field requirements and media constraints are enforced.
- **Input -> Output transformation:**
  - **Input:** course metadata + structured content payloads.
  - **Output:** persistent learning structure consumable for UI rendering/export.

### Flow 4: AI-Assisted Course Generation Pipeline
- **Flow name:** Source-to-course generation flow
- **Step-by-step process:**
  1. User provides learning inputs (files/text/URLs).
  2. System validates source quality and boundaries.
  3. Metadata is generated and returned.
  4. Outline is generated from metadata.
  5. Full course generation is triggered asynchronously.
  6. Usage/cost is recorded against workspace balance.
- **APIs involved:**
  - `POST /course/metadata`
  - `POST /course/outline`
  - `POST /course/full-course`
  - usage/balance endpoints under `/usage/*` and `/billing/balance`
- **Business rules applied:**
  - At least one valid content source is required.
  - File/URL/text limits and accessibility checks apply.
  - Balance threshold can warn or block expensive operations.
  - Outline/full-course prerequisites must be satisfied.
- **Input -> Output transformation:**
  - **Input:** raw learning source data.
  - **Output:** structured metadata -> outline -> generated course artifacts + cost entries.

### Flow 5: Payment to Credit Wallet Top-up
- **Flow name:** Stripe-backed AI credit recharge
- **Step-by-step process:**
  1. User requests payment session for credit amount.
  2. Checkout occurs externally.
  3. Success callback/webhook confirms payment state.
  4. System marks payment history status progression.
  5. On completion, workspace balance and transaction ledger are updated.
- **APIs involved:**
  - `POST /payment/create-session`
  - `GET /payment/session-details`
  - `POST /payment/success`
  - `POST /payment/webhook`
  - `GET /billing/balance`
- **Business rules applied:**
  - Amount min/max bounds are enforced.
  - Credit addition occurs only on completed payment state.
  - Duplicate or out-of-order state transitions are ignored/guarded.
- **Input -> Output transformation:**
  - **Input:** requested recharge amount + payment events.
  - **Output:** confirmed credit increment and auditable financial ledger entry.

### Flow 6: SCORM Conversion (Async Polling)
- **Flow name:** File-to-SCORM conversion with polling lifecycle
- **Step-by-step process:**
  1. User submits source file for conversion.
  2. System validates file type/size/rules and creates conversion job.
  3. Client polls job status until terminal state.
  4. If successful, user downloads generated SCORM package.
- **APIs involved:**
  - `POST /scorm/polling/convert`
  - `GET /scorm/polling/status/{jobId}`
  - `GET /scorm/polling/download/{jobId}`
  - Public equivalent: `/scorm/public/*` (API-key protected)
- **Business rules applied:**
  - Only configured formats and size limits are accepted.
  - Download is allowed only for completed jobs.
  - Job TTL/cleanup behavior bounds data availability window.
- **Input -> Output transformation:**
  - **Input:** source document/media file.
  - **Output:** SCORM ZIP + conversion status timeline.

### Flow 7: SCORM Editor (Upload -> Edit -> Preview -> Export)
- **Flow name:** SCORM package editing lifecycle
- **Step-by-step process:**
  1. User uploads SCORM ZIP and receives editing session.
  2. System parses course structure and slide content.
  3. User updates blocks and/or replaces media assets.
  4. User previews package via session-scoped asset URLs.
  5. User exports and downloads edited ZIP.
- **APIs involved:**
  - `POST /scorm/editor/upload`
  - `GET /scorm/editor/sessions/{sessionId}/slides/{slideId}`
  - `PUT /scorm/editor/sessions/{sessionId}/slides/{slideId}/blocks`
  - `POST /scorm/editor/sessions/{sessionId}/media/upload`
  - `GET /scorm/editor/sessions/{sessionId}/preview`
  - `POST /scorm/editor/sessions/{sessionId}/export`
  - `GET /scorm/editor/sessions/{sessionId}/download`
- **Business rules applied:**
  - Session must be in ready/exported state for read/edit/export actions.
  - Tool-specific package behavior (e.g., Rise options) is preserved on preview/export.
  - Asset serving blocks path traversal and invalid file access.
- **Input -> Output transformation:**
  - **Input:** SCORM ZIP + edit operations.
  - **Output:** revised SCORM content and exportable edited package.

## D. Business Rules

1. **Account and Access Rules**
   - Inactive/suspended/pending states cannot proceed through normal login.
   - Password-reset-required states must complete password flow before normal usage.
   - Workspace validity influences whether business actions are allowed.

2. **Role-Based Decision Rules**
   - Tenant admin, regular author/user, and super admin actions differ significantly.
   - Subscription approvals/rejections and certain governance actions are admin-only.

3. **Subscription and Entitlement Rules**
   - Upgrade requests follow eligibility and uniqueness rules.
   - Plan change triggers updated limitation and entitlement behavior.

4. **Course Authoring Rules**
   - Hierarchical parent-child constraints are strict.
   - Workspace course limits and content validation constraints apply before persistence.

5. **AI Generation Rules**
   - Source validation (count, size, text length, URL quality) gates generation.
   - Threshold checks can return warning or hard block based on available credits.
   - Full generation requires prior metadata/outline quality conditions.

6. **Billing and Wallet Rules**
   - Recharge amount bounds are enforced.
   - Wallet update is tied to payment completion status only.
   - Transaction records are used for traceability and reporting.

7. **SCORM Rules**
   - Conversion/editor actions are constrained by file/session validity.
   - Downloads and exports are state-dependent.
   - Session-scoped preview access governs editable package exposure.

## E. Data Meaning

### Core Business Entities

- **User**
  - Represents a person who can authenticate and perform authoring/admin actions.
  - Important meaning fields: account status, verification/password lifecycle state, identity details.

- **Workspace**
  - Represents an organization/business tenant boundary.
  - Important meaning fields: plan type, active/expiry context, organizational profile/billing context.

- **UserWorkspace**
  - Represents the user’s role and relationship to a workspace.
  - Important meaning fields: role assignment and membership status used for authorization.

- **Course**
  - Represents a complete learning product.
  - Important meaning fields: title/description/objectives, owner/workspace association, publish/readiness context.

- **Module / Lesson / Block**
  - Represents the educational structure inside a course.
  - Important meaning fields:
    - Ordering fields: sequencing in learner journey.
    - Status fields: lifecycle state (draft/active-like behavior).
    - Content fields: actual teaching content payload.

- **CourseTemplate (and template sub-entities)**
  - Represents reusable starting blueprints for faster authoring.
  - Important meaning fields: template scope, default structure/content.

- **ApiUsage**
  - Represents tracked AI consumption events.
  - Important meaning fields: provider/model operation, token/cost metrics, workspace attribution.

- **WorkspaceBalance**
  - Represents remaining AI credit wallet for a workspace.
  - Important meaning fields: current credit amount used for generation eligibility.

- **WorkspaceBalanceTransaction**
  - Represents immutable financial/credit movement history.
  - Important meaning fields: type (debit/credit), source (payment/usage), amount, timestamp.

- **StripePaymentHistory**
  - Represents payment lifecycle state machine per checkout.
  - Important meaning fields: external session/payment identifiers, status progression, credited amount.

- **SubscriptionChangeRequest**
  - Represents tenant request for plan transition.
  - Important meaning fields: request status, requested plan, reviewer/admin action context.

- **EditSession (SCORM Editor)**
  - Represents temporary editing workspace for one uploaded SCORM package.
  - Important meaning fields: session status, detected tool type, temp/extracted directories, error message.

- **CourseStructure / SlideContent (SCORM Editor Models)**
  - Represents parsed course navigation and editable slide blocks.
  - Important meaning fields:
    - Slide identity and ordering.
    - Block type/content map used for UI editing and write-back.

## F. Edge Cases

1. **Authentication and membership edge cases**
   - Valid credentials but invalid account/workspace state still results in denial.
   - Token may be structurally valid but rejected by business state checks.

2. **Subscription flow edge cases**
   - Duplicate pending requests.
   - Requests from ineligible plans.
   - Admin decision races (already processed request).

3. **Course generation edge cases**
   - Malformed/empty source payload combinations.
   - Oversized files/URLs/text exceeding configured caps.
   - Insufficient credits at threshold-check stage.

4. **Payment edge cases**
   - Duplicate webhook delivery.
   - Invalid signature or out-of-order payment events.
   - Session exists but payment not yet in creditable state.

5. **SCORM conversion edge cases**
   - Unsupported file types.
   - Conversion job expired or removed before download.
   - Polling requests on unknown job IDs.

6. **SCORM editor edge cases**
   - Session status not ready for operation.
   - Missing slide/media path.
   - Path traversal attempts in asset retrieval.
   - Parser mismatch/unsupported package layout.

## G. External Dependencies

1. **Stripe**
   - Used to collect payments and drive credit wallet top-ups.
   - Business role: monetization and purchase confirmation.

2. **AI providers (OpenAI, Gemini)**
   - Used for text/image generation and course generation tasks.
   - Business role: productivity acceleration for content authors.

3. **OpenAI Files API**
   - Used to process uploaded source material for generation workflows.
   - Business role: enrich AI context from user-provided documents.

4. **LibreOffice conversion engine**
   - Used to transform office/media source inputs into SCORM-compatible assets.
   - Business role: broad input compatibility for course publishing.

5. **Email providers (SMTP/ZeptoMail)**
   - Used for account and business notifications (password flows, subscription actions, generation notifications).
   - Business role: lifecycle communication and operational alerts.

6. **MySQL**
   - Used for durable persistence of identity, content, billing, and usage state.
   - Business role: single source of truth for product operations.

## H. Functional Risks

1. **Cross-module dependency risk**
   - Access rules combine account state, workspace state, and role state.
   - Small changes in one domain can cause hidden authorization regressions.

2. **Generation + billing coupling risk**
   - AI generation eligibility depends on threshold and wallet behavior.
   - Incorrect ordering or error handling can result in blocked valid users or unbilled usage.

3. **Subscription transition risk**
   - Plan changes affect limits, access, and feature behavior at once.
   - Incomplete transition logic can create entitlement inconsistencies.

4. **SCORM flow complexity risk**
   - Multiple formats, async job lifecycle, parser-specific handling, and temp session state create high change sensitivity.

5. **Hierarchical content integrity risk**
   - Course/module/lesson/block relationships depend on strict parent existence and order constraints.
   - Partial updates can break learner-facing structure if validations are bypassed.

6. **Payment event reliability risk**
   - Real-world webhook retries and async race conditions can cause duplicate/missed crediting without strict state guards.

## I. FUNCTIONAL RULES

1. Do not break existing business workflows (auth, authoring, billing, SCORM, generation).
2. Preserve all business validations and decision gates (status checks, limits, eligibility, thresholds).
3. Maintain backward-compatible behavior for current clients and integrations.
4. Reuse established business logic paths; avoid parallel or duplicate rule implementations.
5. Do not bypass service-layer business logic when adding/changing endpoints.
6. Keep role and workspace boundary semantics unchanged unless explicitly re-designed.
7. Ensure payment-crediting and usage metering remain consistent and auditable.
8. Preserve conversion/editor session-state gating and SCORM safety controls.
