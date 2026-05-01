# Course Generation API — Full Technical Specification

This document includes:

- Step 1 — UI Material Collection  
- Step 2 — Metadata Generation  
- Step 3 — Lesson Outline  
- Step 4 — Full Course Generation (Block-Based)  
- Endpoints  
- Request/Response Formats  
- Prompts  
- Explanations  

---

## Architecture Overview

### File Handling Strategy

The system uses a **hybrid approach** for file processing:

1. **PDF and Text Files**:
   - Uploaded to OpenAI Files API
   - Referenced via `file_id` in Responses API
   - Available across all steps (Step 2, 3, 4)

2. **DOCX and PPTX Files**:
   - Text extracted locally using Apache POI
   - Included in prompt text (not uploaded to OpenAI)
   - Only available in Step 2 (metadata generation)
   - Content is merged with user-provided text

3. **File IDs Flow**:
   - Each uploaded file is stored in the LMS and assigned an **application file ID** (`appFileId`).
   - Step 2 response returns `fileIds` as these **application file IDs** for **all** source files (PDF, DOCX, PPTX, text).
   - Internally, for each `appFileId` the system may store:
     - `openAiFileId` (for PDF/text files uploaded to OpenAI)
     - `extractedText` (for DOCX/PPTX files extracted locally)
   - Step 3 and Step 4 receive the same `fileIds` from Step 2 and resolve them to stored metadata / OpenAI file IDs / extracted text.

### AI Data Retention

- `openAiFileId` and `extractedText` are **temporary AI artifacts**.
- They are used across Steps 2 → 3 → 4 to influence metadata, outline, and full course.
- After Step 4 (full course generation) completes successfully, these AI-specific fields **can be cleared** for the associated `fileIds` (while the original files remain in LMS storage).

### AI Model Configuration

- **Model**: `gpt-4o-mini` (cost-effective for course generation)
- **API**: Responses API when files are attached, Chat Completions API otherwise
- **Provider**: OpenAI (default), Gemini (fallback)

### Validation & Limits

All steps enforce validation limits to ensure:
- Requests stay within OpenAI's constraints
- Reasonable usage bounds
- Cost optimization

See Step 2 for detailed validation limits.

---

# Step 1 — (UI Only)

Admin uploads:

- Files  
- URLs  
- Text snippets  

No backend call is made in this step.  
The UI submits these materials directly to Step 2.

---

# Step 2 — Generate Course Metadata

## Endpoint
POST /course/metadata

**Authentication**: Bearer token required in Authorization header

---

## Request Body (multipart/form-data)

| Field   | Type                    | Description               | Validation Limits         |
|---------|-------------------------|---------------------------|---------------------------|
| files[] | File array              | Source files (PDF, DOCX, PPTX, text) | Max 10 files, 50MB each, 200MB total |
| text    | string (optional)       | Pasted user text          | Max 50,000 characters     |
| urls    | array<string> (optional)| Additional references     | Max 10 URLs               |

### Supported File Types

- **PDF files**: Uploaded to OpenAI Files API, referenced via `file_id`
- **DOCX files**: Text extracted locally, included in prompt text (not uploaded)
- **PPTX files**: Text extracted locally, included in prompt text (not uploaded)
- **Text files** (.txt, .md, etc.): Uploaded to OpenAI Files API, referenced via `file_id`

**Note**: DOCX and PPTX files are NOT supported by OpenAI's Responses API via `input_file`, so their text is extracted locally and included in the prompt.

### Backend Flow

1. **Validate request parameters**:
   - File count (max 10)
   - Individual file size (max 50MB)
   - Total file size (max 200MB)
   - URL count (max 10)

2. **Extract text from DOCX/PPTX files**:
   - Text extracted locally using Apache POI
   - Combined with user-provided text
   - Validated: Max 200,000 chars extracted text, Max 250,000 chars combined

3. **Upload supported files to OpenAI**:
   - Only PDF and text files are uploaded
   - DOCX/PPTX are skipped (already extracted)
   - Receives `fileIds` for uploaded files

4. **Combine text sources**:
   - User-provided text + extracted DOCX/PPTX text
   - Included in prompt as "Source content"

5. **Send to OpenAI**:
   - PDF/text files: Attached via `file_id` in Responses API
   - Combined text: Included in prompt text
   - URLs: Included in prompt text
   - Uses `gpt-4o-mini` model via Responses API

---

## Prompt (Internal Use - Optimized)

Generate course metadata (title, description, audience, goal) from the provided sources.

Source content:
[User-provided text + extracted DOCX/PPTX text content]

Reference URLs:
[URL 1]
[URL 2]
...

Note: X file(s) attached via API. Analyze their content.

Return JSON only: {"title":"","description":"","audience":"","goal":""}

### File Attachment Format

PDF and text files are attached via Responses API:
```json
{
  "type": "input_file",
  "file_id": "file-abc123"
}
```

DOCX/PPTX content is included directly in the prompt text (not as file attachments).

---

## Validation Limits

The following limits are enforced to ensure requests stay within OpenAI's constraints and reasonable usage bounds:

| Parameter | Limit | Description |
|-----------|-------|-------------|
| Max Files | 10 | Maximum number of files per request |
| Max File Size | 50 MB | Maximum size per individual file |
| Max Total Size | 200 MB | Maximum combined size of all files |
| Max User Text | 50,000 chars | Maximum user-provided text length |
| Max Extracted Text | 200,000 chars | Maximum extracted text from DOCX/PPTX files |
| Max Combined Text | 250,000 chars | Maximum combined text (user + extracted) |
| Max URLs | 10 | Maximum number of reference URLs |

**Configuration**: These limits are configurable via `application.properties`:
```properties
course.metadata.validation.max-files=10
course.metadata.validation.max-file-size-mb=50
course.metadata.validation.max-total-size-mb=200
course.metadata.validation.max-text-length=50000
course.metadata.validation.max-extracted-text-length=200000
course.metadata.validation.max-combined-text-length=250000
course.metadata.validation.max-urls=10
```

---

## Response Body
{
  "title": "Leadership Essentials for New Managers",
  "description": "A foundational course introducing leadership principles...",
  "audience": "New managers, supervisors, team leads...",
  "goal": "Develop leadership capabilities.",
  "fileIds": ["file-abc123", "file-xyz789"]
}

**Note**: `fileIds` array contains the **application file IDs** for all uploaded source files (PDF, DOCX, PPTX, text).  
Internally, for each `fileId` the system may have:
- an `openAiFileId` (for PDF/text files uploaded to OpenAI), or
- `extractedText` (for DOCX/PPTX files processed locally).
These internal fields are not exposed in the API response.

---

# Step 3 — Generate Lessons (Outline Generation)

The AI generates only lessons[].  
Backend/UI merges lessons + metadata into a final outline.

---

## Endpoint
POST /course/outline

**Authentication**: Bearer token required in Authorization header

---

## Request Body (application/json)
{
  "metadata": {
    "title": "Leadership Essentials for New Managers",
    "description": "A foundational course introducing leadership principles...",
    "audience": "New managers, supervisors, team leads...",
    "goal": "Develop leadership capabilities.",
    "fileIds": ["file-abc123", "file-xyz789"]
  },
  "optional": {
    "desiredLessons": 6,
    "tone": "Professional",
    "language": "English",
    "courseType": "Other"
  }
}

**Note**: `metadata.fileIds` are the same **application file IDs** returned from Step 2 (for all source files).  
For PDF/text files, the backend reuses the stored `openAiFileId` to attach them again to OpenAI.  
For DOCX/PPTX files, the backend can reuse the stored `extractedText` to influence the outline prompt.

---

## Backend Flow

1. **Receive metadata (including fileIds) from Step 2**
2. **Build optimized prompt** with metadata context
3. **Send to OpenAI**:
   - Files attached via `file_id` in Responses API (if fileIds provided)
   - Uses `gpt-4o-mini` model via Responses API
   - If no files, uses Chat Completions API

---

## Prompt (Internal Use - Optimized)

Generate lesson plan for this course.

Context:
Title: {{title}}  
Description: {{description}}  
Audience: {{audience}}  
Goal: {{goal}}  
Desired Lessons: {{desiredLessons}}  
Tone: {{tone}}  
Language: {{language}}  

Note: {{fileIds.length}} file(s) attached via API. Use their content to inform lesson structure.

Return JSON only: {"lessons":[{"title":"","description":""}]}

Do NOT include: overview, objectives, audience, or metadata fields—only lessons array.  

---

## Response Body

{
  "lessons": [
    {
      "title": "Lesson 1: Understanding Leadership Roles",
      "description": "Introduction to leadership responsibilities."
    }
  ]
}

---

# Step 4 — Generate Full Course (Block-Based)

This step produces the final block-renderable course.

---

## Endpoint
POST /course/full-course

**Authentication**: Bearer token required in Authorization header

---

## Request Body (application/json)
{
  "metadata": {
    "title": "Leadership Essentials for New Managers",
    "description": "A foundational course...",
    "audience": "New managers...",
    "goal": "Develop leadership capability.",
    "tone": "Professional",
    "language": "English"
  },
  "outline": {
    "overview": "A foundational course...",
    "objectives": [],
    "audience": "New managers...",
    "lessons": [
      {
        "title": "Lesson 1: Understanding Leadership Roles",
        "description": "Introduction to leadership responsibilities."
      }
    ]
  },
  "questionsPerLesson": 3,
  "fileIds": ["file-abc123", "file-xyz789"]
}

**Note**: `fileIds` are the same **application file IDs** from Step 2 (for all source files).  
For PDF/text files, the backend reuses the stored `openAiFileId` to attach them again to OpenAI.  
For DOCX/PPTX files, the backend can reuse the stored `extractedText` to influence full-course generation.

---

## Backend Flow

1. **Receive metadata, outline, and fileIds from previous steps**
2. **Build optimized prompt** with block schema, metadata, and outline
3. **Send to OpenAI**:
   - Files attached via `file_id` in Responses API (if fileIds provided)
   - Uses `gpt-4o-mini` model via Responses API
   - If no files, uses Chat Completions API

---

## Step 4 Prompt (Internal Use - Optimized)

Generate complete course for LMS. Follow EXACT block definitions below.

BLOCK_SCHEMA: {{BLOCK_SCHEMA_JSON}}

Includes: basicTypes, textLayouts, imageLayouts, videoLayouts, quoteLayouts, quizTypes.

Metadata:
Title: {{title}}
Description: {{description}}
Audience: {{audience}}
Goal: {{goal}}
Tone: {{tone}}
Language: {{language}}

Outline:
{{lessons}}

Note: {{fileIds.length}} file(s) attached via API. Use their content for course generation.

Rules:
- Output: {"modules":[{"title":"","lessons":[{"title":"","blocks":[{"type":"","content":{}}]}]}]}
- Each lesson: 5-12 blocks
- Final block: QUIZ (follow BLOCK_SCHEMA)
- Quiz questions: {{questionsPerLesson}} per lesson
- Tone: {{tone}}
- Language: {{language}}
- Return JSON only

---

## Response Body Example

{
  "modules": [
    {
      "title": "Module 1: Leadership Foundations",
      "lessons": [
        {
          "title": "Lesson 1: Understanding Leadership Roles",
          "blocks": [
            {
              "type": "HEADING_TEXT",
              "content": {
                "heading": "Understanding Leadership Roles",
                "text": "Leadership involves guiding and motivating a team."
              }
            },
            {
              "type": "TEXT",
              "content": {
                "text": "Leaders help create clarity, direction, and support."
              }
            },
            {
              "type": "IMAGE",
              "content": {
                "altText": "Leadership concept",
                "fileId": "file-xyz789"
              }
            },
            {
              "type": "QUOTE_BOXED",
              "content": {
                "text": "A leader is one who knows the way, goes the way, and shows the way.",
                "author": "John Maxwell"
              }
            },
            {
              "type": "QUIZ",
              "content": [
                {
                  "quizType": "single-choice",
                  "question": "What is the primary role of a leader?",
                  "options": ["To guide the team", "To ignore the team"],
                  "correctAnswer": "To guide the team"
                },
                {
                  "quizType": "true-false",
                  "question": "Leadership involves inspiring people.",
                  "correctAnswer": "true"
                },
                {
                  "quizType": "fill-in-the-blanks",
                  "question": "A leader must always ____ responsibility.",
                  "correctAnswer": "take"
                }
              ]
            }
          ]
        }
      ]
    }
  ],
  "message": "Course Generation Complete!"
}