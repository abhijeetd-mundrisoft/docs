# AI Course Generation Code Flow and Debug Guide

## 1) Main API Flow (Controller Layer)

- `POST /course/metadata` -> `CourseGenerationController.generateMetadata()`
- `POST /course/outline` -> `CourseGenerationController.generateOutline()`
- `POST /course/full-course` -> `CourseGenerationController.generateFullCourse()`

These are the three core steps for course generation.

## 2) Step-by-Step Service Flow

### Step A: Metadata generation (`/course/metadata`)

1. `CourseGenerationController.generateMetadata()`
2. `CourseMetadataValidationService` validates files, URLs, and text length.
3. `FileService.storeFile()` stores source files in local/LMS storage.
4. `OpenAiFileService.extractTextFromFiles()` extracts text from DOCX/PPTX.
5. `OpenAiFileService.uploadFiles()` uploads supported files (PDF/text) to OpenAI Files API.
6. `CourseMetadataService.generateMetadata()` builds metadata prompt.
7. `TextPromptService.generateText()` dispatches to provider.
8. `OpenAIProvider.generateText(...)` or `OpenAIProvider.generateTextWithFiles...(...)`.
9. AI response parsed and returned as `CourseMetadataResponseDto`.

### Step B: Outline generation (`/course/outline`)

1. `CourseGenerationController.generateOutline()`
2. `CourseOutlineService.generateOutline()`
3. `CourseOutlineService.resolveOpenAiFileIds()` reads mapping from `CourseSourceMaterialRepository`.
4. `TextPromptService.generateText()`
5. `OpenAIProvider` call
6. Parse JSON -> `CourseOutlineResponseDto`.

### Step C: Full course generation (`/course/full-course`)

1. `CourseGenerationController.generateFullCourse()`
2. `CourseFullCourseService.generateFullCourseAsync()`
3. Pipeline stages:
   - `generateCourseStructure(...)` (single/chunked generation)
   - `generateImagesForCourse(...)` (optional image generation)
   - `CoursePersistenceService.persistCourse(...)`
   - `CourseGenerationEmailService` success/failure email
   - `cleanupSourceMaterials(...)`
4. OpenAI usage and billing entries happen during provider calls.

## 3) Uploading Document, Adding URL, Text, Prompt

### In `/course/metadata`

- Documents: multipart `files`
- Free text: multipart `text`
- URLs: multipart `urls` (JSON array string)
- Session: multipart `sessionId`

Controller handling:
- `CourseGenerationController.parseUrls(...)`
- `CourseGenerationController.combineText(...)`

Document handling:
- Stored in LMS: `FileService.storeFile(...)`
- Extract DOCX/PPTX text: `OpenAiFileService.extractTextFromFiles(...)`
- Upload PDF/text to OpenAI: `OpenAiFileService.uploadFiles(...)`
- Mapping persisted in `course_source_material` table via `CourseSourceMaterialRepository`.

## 4) How OpenAI Is Used

### Provider selection

- `WorkspaceAiConfigService.getConfiguredProvider(workspaceId, "TEXT")`
- `AIProviderFactory.getProvider(providerCode)` returns provider bean (`openaiProvider`, etc.).

### API paths in OpenAI provider

- No files -> Chat Completions (`/v1/chat/completions`)
- With files -> Responses API (`/v1/responses`) with `input_file`.

Key methods:
- `OpenAIProvider.generateText(...)`
- `OpenAIProvider.generateTextRaw(...)`
- `OpenAIProvider.callOpenAIApiWithTimeout(...)`
- `OpenAIProvider.callResponsesApiWithTimeout(...)`

## 5) AI Transaction Deduction and Usage Tracking

1. OpenAI response contains token usage.
2. `OpenAIProvider.recordTextUsageFromRoot(...)` or `recordTextUsageFromResponsesRoot(...)`
3. Calls `ApiUsageService.recordUsageForCurrentUser(...)`
4. Persists row in `api_usage` table via `ApiUsageRepository`.
5. Deducts balance via `BillingBalanceService.deduct(...)`
6. Ledger row inserted in `workspace_balance_transaction` via `WorkspaceBalanceTransactionRepository`.

## 6) Platform Key vs External (Custom) Key

### Resolution

- `WorkspaceAiConfigService.getEffectiveApiKey(workspaceId, providerCode, modelType, systemKey)`
  - If custom key enabled (`useCustomKey=true`) -> decrypt and use workspace key.
  - Else -> use platform/system key.

### Credit behavior

- If custom key active, cost is treated as zero for billing deductions.
- Checkpoints:
  - `OpenAIProvider.ensureSufficientCredits(...)`
  - `workspaceAiConfigService.isCustomKeyActive(...)`
  - `ApiUsageService.recordUsage(...)` sets zero cost for custom key usage.

## 7) Essential Breakpoints (Set These First)

Set breakpoints at these method entries for end-to-end navigation:

### Controller breakpoints
- `CourseGenerationController.generateMetadata()`
- `CourseGenerationController.generateOutline()`
- `CourseGenerationController.generateFullCourse()`

### File/source breakpoints
- `FileService.storeFile()`
- `OpenAiFileService.extractTextFromFiles()`
- `OpenAiFileService.uploadFiles()`
- `CourseSourceMaterialRepository.findByFileId(...)` caller sites

### Prompt and provider breakpoints
- `CourseMetadataService.generateMetadata()`
- `CourseOutlineService.generateOutline()`
- `CourseFullCourseService.generateCourseStructureSingle()`
- `CourseFullCourseService.generateCourseChunk()`
- `TextPromptService.generateText()`
- `OpenAIProvider.generateTextRaw()`
- `OpenAIProvider.callOpenAIApiWithTimeout()`
- `OpenAIProvider.callResponsesApiWithTimeout()`

### Usage and billing breakpoints
- `OpenAIProvider.recordTextUsageFromRoot()`
- `OpenAIProvider.recordTextUsageFromResponsesRoot()`
- `ApiUsageService.recordUsage()`
- `BillingBalanceService.deduct()`

### Key resolution breakpoints
- `WorkspaceAiConfigService.getEffectiveApiKey()`
- `WorkspaceAiConfigService.isCustomKeyActive()`
- `WorkspaceAiConfigService.getEffectiveModelId()`

## 8) Suggested Debug Run Order

1. Trigger `/course/metadata` with one file + text + URL.
2. Verify:
   - file stored
   - text extracted
   - OpenAI file uploaded
   - metadata response parsed
3. Trigger `/course/outline`.
4. Trigger `/course/full-course`.
5. Confirm usage + deduction entries are recorded.

Use a single `sessionId` across all three calls to track one generation journey.
