# Technical Context - Update AI Models and Endpoints

## Proposed Changes

### 1. Data Model Updates (SQL Migration)
A new Liquibase/Flyway migration is required to register the new Gemini models and update the Gemini provider's base URL.

#### AI Provider Update
Update the Google Gemini provider to use the `v1beta` endpoint to support preview and advanced models.

```sql
-- Update Gemini provider to v1beta for both text and image generation
UPDATE ai_provider 
SET base_url = 'https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent',
    image_url = 'https://generativelanguage.googleapis.com/v1beta/models/{model}:predict',
    updated_at = NOW()
WHERE id = 'ap-gemini-001';
```

#### AI Model Registration
Add/Update models in the `ai_model` table.

```sql
-- Text Generation Models
INSERT INTO ai_model (id, provider_id, model_name, display_name, model_type, description, is_active, is_default, created_at, updated_at)
VALUES 
('am-gemini-3.1-pro', 'ap-gemini-001', 'gemini-3.1-pro-preview', 'Gemini 3.1 Pro', 'TEXT', 'The most intelligent model for complex reasoning, long-form content generation, and agentic coding. Features a 1-million-token context window.', true, false, NOW(), NOW()),
('am-gemini-3-flash', 'ap-gemini-001', 'gemini-3-flash-preview', 'Gemini 3 Flash', 'TEXT', 'The "Workhorse" model. Delivers frontier-class performance at high speeds. Ideal for generating quiz questions, lesson summaries, and automated grading.', true, false, NOW(), NOW()),
('am-gemini-3.1-flash-lite', 'ap-gemini-001', 'gemini-3.1-flash-lite-preview', 'Gemini 3.1 Flash-Lite', 'TEXT', 'The ultra-fast, cost-efficient model. Optimized for high-frequency, lightweight tasks like chat translation, data extraction, and sub-second UI updates.', true, false, NOW(), NOW()),
('am-gemini-3.1-deep-think', 'ap-gemini-001', 'gemini-3.1-deep-think', 'Gemini 3 Deep Think', 'TEXT', 'An experimental reasoning model designed for PhD-level scientific research, advanced math, and high-stakes logical verification.', true, false, NOW(), NOW());

-- Image Generation Models
INSERT INTO ai_model (id, provider_id, model_name, display_name, model_type, description, is_active, is_default, created_at, updated_at)
VALUES 
('am-gemini-3.1-flash-image', 'ap-gemini-001', 'gemini-3.1-flash-image-preview', 'Nano Banana 2', 'IMAGE', 'The high-efficiency flagship. Optimized for rapid, high-volume generation and native image editing with Google Search grounding for real-world accuracy.', true, false, NOW(), NOW()),
('am-gemini-3-pro-image', 'ap-gemini-001', 'gemini-3-pro-image-preview', 'Nano Banana Pro', 'IMAGE', 'The professional design engine. Features a reasoning core for studio-quality 4K visuals, complex layouts, and precise text rendering for infographics and posters.', true, false, NOW(), NOW()),
('am-gemini-2.5-flash-image', 'ap-gemini-001', 'gemini-2.5-flash-image', 'Nano Banana', 'IMAGE', 'The legacy high-speed model. Best for low-latency, repetitive tasks where efficiency and speed are prioritized over high-fidelity artistic detail.', true, false, NOW(), NOW());
```

### 2. Service Layer Changes

#### [GeminiAIProvider.java](file:///c:/PVS/simpliCourse-New/course-forge-backend/src/main/java/com/mundrisoft/courseforge/service/GeminiAIProvider.java)
- Ensure the `callGeminiApi` method correctly handles the `v1beta` payload if there are structural differences (unlikely for basic `generateContent`, but worth verifying).
- Verification of `recordUsageFromRoot` to ensure token counts are still correctly extracted from `usageMetadata`.

#### [GeminiImageProvider.java](file:///c:/PVS/simpliCourse-New/course-forge-backend/src/main/java/com/mundrisoft/courseforge/service/GeminiImageProvider.java)
- Update the hardcoded fallback model if necessary (currently `am-imagen-3-fast`).
- Ensure the `v1beta` endpoint's `predict` payload for Imagen models remains compatible with the new 3.x series.

### 3. API Endpoint Impact
- **`GET /api/ai-config/models`**: This endpoint (or equivalent) will now return the new models for both text and image generation.
- **AI-driven Course Generation**: All endpoints like `/course/metadata`, `/course/outline`, and `/course/full-course` will benefit from the more capable models if they are set as defaults or explicitly selected.

## Sample JSON Structure
When retrieving available models for a workspace/provider, the response will include entries like:

```json
{
  "modelId": "am-gemini-3.1-pro",
  "displayName": "Gemini 3.1 Pro",
  "modelType": "TEXT",
  "description": "The most intelligent model for complex reasoning, long-form content generation, and agentic coding...",
  "provider": "GEMINI"
}
```

## Implementation Plan
1.  **Database Migration**: Create a new SQL migration file to insert the new models and update the provider URL.
2.  **Configuration Verification**: Verify that `AiModelService` and `AiProviderConfigService` correctly pick up the new database entries upon refresh/startup.
3.  **Integration Testing**: 
    - Perform a test text generation request using `Gemini 3.1 Pro`.
    - Perform a test image generation request using `Nano Banana 2`.
4.  **Cost/Pricing Configuration**: Ensure `AiModelPricingService` is updated or configured to handle the costs associated with these new models (if applicable).

## Potential Impact on Other Modules
- **Billing/Usage**: New models will start generating usage records. Ensure the `model_name` strings match what the AI providers expect so that token tracking remains accurate.
- **AI Course Generation**: If a new model (like Gemini 3.1 Pro) is set as default, the quality and structure of generated courses may change significantly. Regression testing on course generation is recommended.
