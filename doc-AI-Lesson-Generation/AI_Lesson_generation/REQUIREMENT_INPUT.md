# AI Lesson Generation — Revised Requirement Input

## 1) Problem Statement

Authors need to generate **a single lesson** using AI, using one or more content sources and lesson configuration parameters, while the UI provides a guided 3-step workflow:

1. **Add Content**
2. **Lesson Info**
3. **Generate**

The system must trigger **asynchronous lesson generation** after the user confirms, show a non-blocking success state, and notify the user upon completion (email and/or UI update depending on implementation).

AI credit consumption for lesson generation must follow the same billing rules as existing AI course generation:
- **Platform AI model**: deduct credits using the platform billing logic and create transaction-history/consumption-summary entries.
- **External AI model**: do **not** deduct credits, but still record a transaction-history entry for traceability (consumption blank/null).

Additionally, when lesson generation completes:
- The lesson must open in the Lesson Editor.
- All blocks must be editable.
- Preview must work correctly.
- On failure: log the failure and notify the user; provide retry.

## 2) Business Goal

- Enable faster lesson creation from AI using guided inputs.
- Ensure predictable output quality via a constrained lesson format:
  - Textual only
  - Textual with interactive elements
  - Optional quiz question count
- Integrate AI usage tracking with billing/accounting so authors can understand and manage AI consumption.
- Provide a production-grade UX: async processing, progress messaging, error notification, and retry.

## 3) User Flow

### Step 1 — Add Content

Given the user navigates to **“Generate Lesson with AI”**, when the page loads, then the system must display a 3-step flow:
- **Add Content**
- **Lesson Info**
- **Generate**

Given the user is on **“Add Content”**, when the user uploads files, adds URLs, or pastes text, then the system must accept and display **multiple content sources**.

Given no content is added, when the user clicks **Next**, then the system must block navigation and show validation errors.

Validation rules at this step:
- Reject oversized/invalid files (with clear UI error).
- Reject invalid URL formats if URLs are provided.
- Reject content payloads that exceed configured limits.

### Step 2 — Lesson Info Configuration

Given the user is on **“Lesson Info”**, when the step loads, then the system must allow:

1. **Lesson Title** (mandatory)
2. **Description** (mandatory)
3. **Audience** (mandatory)
4. **Tone & Style** (optional)
5. **Lesson Content Type** (mandatory):
   - `TEXTUAL_ONLY` (Text-only blocks)
   - `TEXTUAL_WITH_INTERACTIVE_ELEMENTS` (Text + interactive blocks)
6. **Number of quiz questions** (optional)

Validation rules at this step:
- Given lesson content type is not selected, when proceeding, then validation must be shown.
- If quiz count is provided:
  - Validate range (min/max per product rules; expected range derived from current requirement: **1–6**).

Content-type enforcement:
- If the user selects **Textual only**, then only static/text blocks must be generated.
- If the user selects **Textual with interactive elements**, then the generated structure must include:
  - Text blocks
  - Interactive-compatible blocks as defined by the product’s block allow-list (**exact allow-list TBD in backend technical design**).

### Step 3 — Generate

Given the user is on **“Generate”**, when the user clicks **Generate**, then the system must:
- Send all inputs to the AI service.
- Trigger **asynchronous lesson generation**.
- Show a success state instead of blocking the UI.

Confirmation message on trigger:
“Lesson Generation Started!
Your will receive an email notification when it's ready.”

The UI must provide a CTA:
- **Go to Courses**

### Post Generation

Given lesson generation completes successfully, when the user accesses the lesson, then:
- The lesson must open in the **Lesson Editor**.
- All blocks must be editable.
- Preview must work correctly.

Given generation fails, when an error occurs, then:
- The system must log the failure.
- The user must be notified (UI and/or email if applicable).
- A **Retry** option must be available.

## 4) Affected Modules

### Frontend
- UI wizard and validation:
  - Add Content (files/URLs/text)
  - Lesson Info (mandatory fields, enums, quiz range)
  - Generate step (async start, confirmation message, CTA)
- UI behavior:
  - Block-next navigation when required data missing
  - Show generated lesson in Lesson Editor
  - Provide retry UX on failure

### Backend
- New API endpoints for AI lesson generation:
  - Upload/ingest content step
  - Generate lesson step (sync DTO or async job token; requirement mandates async)
- Prompt building and AI invocation for lesson structure generation
- AI response parsing and semantic validation:
  - Block allow-list enforcement based on content type
  - Quiz question count enforcement if quiz count provided
- Lesson persistence integration:
  - Create/update lesson and blocks in DB as per existing lesson/block models
- AI credit consumption integration:
  - Platform model: deduct credits and record usage
  - External model: no deduction but record traceability entry
- Failure handling:
  - log failure
  - notify user
  - provide retry capability

## 5) Existing Feature References (if any)

- AI course generation pipeline:
  - `CourseGenerationController` (`/course/metadata`, `/course/outline`, `/course/full-course`)
  - `CourseMetadataService`, `CourseOutlineService`, `CourseFullCourseService`
  - `OpenAiFileService` (file upload + extracted text handling)
  - AI provider stack: `AIProviderFactory`, providers, `AiResponseParser`
- Lesson editor and lesson/block persistence:
  - `LessonController`, `LessonService`
  - block persistence via existing `blocks` entity/table and JSON `content` column
- Billing and credit logic:
  - feature thresholds: `FeatureBalanceThresholdService`
  - usage tracking: `ApiUsageService` and billing deduction in `BillingBalanceService`
  - custom-key behavior: `WorkspaceAiConfigService`

## 6) Data Impact (DB/API)

### DB Impact

Expected to reuse existing persistence model:
- Lesson entities (table: `lessons`)
- Block entities (table: `blocks`)
- Source mapping table used in course generation flow (table: `course_source_material`) for temporary extracted text and OpenAI file IDs

New DB objects (optional; depends on async implementation):
- Optional job tracking table for lesson generation status history (if polling/status endpoints are implemented)

### API Impact

New lesson-generation APIs (exact endpoints TBD in technical design) must follow the flow:
- **Add Content step**
  - Upload files / ingest URLs / ingest raw text
  - Return a session/correlation id and application-level source ids (`fileIds`)
- **Generate step**
  - Accept session id + intermediate lesson info fields
  - Trigger async generation and return “started” response immediately

### Email/Notification
- On generation start and/or completion:
  - Send email notification when ready (implementation depends on existing email service patterns).

## 7) Expected Output

### Output 1 — Async start response

When the user clicks Generate, the backend must:
- Start the async job
- Return an immediate success/started response
- Ensure the UI shows:
  “Lesson Generation Started!
Your will receive an email notification when it's ready.”

### Output 2 — Generated lesson artifact

When generation completes successfully, output the full lesson:
- Lesson title and description (based on inputs)
- Blocks ordered and structured for the requested content type:
  - `TEXTUAL_ONLY` → static/text blocks only
  - `TEXTUAL_WITH_INTERACTIVE_ELEMENTS` → mix of text + interactive blocks
- Quiz blocks must honor the user-provided quiz count (if quiz count is provided)
- Lesson must be editable and previewable in Lesson Editor.

### Output 3 — Failure outcome

On failure:
- Log failure server-side.
- Notify the user.
- Provide retry capability.

## 8) Constraints (performance, security, etc.)

### Performance
- Lesson generation must be asynchronous.
- For large uploaded documents, extraction and AI calls must use configured timeouts and executor pools.
- UI must not block during generation.

### Security
- JWT authentication and workspace scoping must be enforced.
- ModuleId (if provided) must be validated against caller workspace permissions.
- Generated content must be validated server-side before persistence.

### Validation / Quality
- Validate required fields at each step.
- Enforce content-type block allow-list.
- Enforce quiz count constraints.

### AI Credit Consumption Rules (must be enforced)

#### Given user is using **Platform AI Model**
When lesson generation is triggered, then:
- AI credit consumption must be calculated using the same logic as AI Course Generation:
  - Low-credit blocking via feature thresholds
  - Token cost calculation from provider usage
  - Balance deduction on workspace
  - Usage tracked for `Feature Used: AI Lesson Generation`
- Transaction History entry must be created with:
  - Feature Used: AI Lesson Generation
  - Date & Time
  - Used By
  - Consumption amount
- Consumption must be reflected in Consumption Summary under legend:
  - AI Lesson Generation

#### Given user is using **External AI Model**
When lesson generation is triggered, then:
- AI credit consumption must NOT be calculated.
- Entry must still be created in Transaction History with:
  - Feature Used: AI Lesson Generation
  - Date & Time
  - Used By
  - Consumption: blank/null

Notes for backend alignment:
- “External AI Model” must map to the platform’s custom-provider-key mode (no balance deduction) behavior.

