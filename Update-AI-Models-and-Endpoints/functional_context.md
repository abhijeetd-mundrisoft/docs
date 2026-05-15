# Functional Context - Update AI Models and Endpoints

## Requirement Overview
The goal of this requirement is to enhance the AI capabilities of the platform by introducing support for the latest Google Gemini 3.x models for both text and image generation. This includes updating the model registry with new display names, descriptions, and transitioning to a more appropriate API endpoint to support these preview/new models.

## Current vs Expected Functionality

### Current Functionality
- The system currently uses older Gemini models (e.g., Gemini 1.5, 2.5 series).
- Display names and descriptions are based on older branding.
- API endpoints are split between `v1` (Text) and `v1beta` (Image).
- Limited support for the latest reasoning and high-efficiency models.

### Expected Functionality
- Support for a new suite of **Text Generation Models**:
    - **Gemini 3.1 Pro**: Intelligent reasoning and long-form content.
    - **Gemini 3 Flash**: High-speed "Workhorse" for quizzes and summaries.
    - **Gemini 3.1 Flash-Lite**: Ultra-fast, cost-efficient for lightweight tasks.
    - **Gemini 3.1 Deep Think**: Experimental reasoning for high-stakes logic.
- Support for a new suite of **Image Generation Models** (Nano Banana series):
    - **Nano Banana 2** (gemini-3.1-flash-image-preview): High-efficiency flagship.
    - **Nano Banana Pro** (gemini-3-pro-image-preview): Professional design engine.
    - **Nano Banana** (gemini-2.5-flash-image): Legacy high-speed model.
- Unified or updated API endpoints to ensure compatibility with these preview models.

## Acceptance Criteria
1.  All new models mentioned in the requirement must be available in the `ai_model` configuration table.
2.  Each model must have the correct `model_name` (API identifier), `display_name`, and `description` as specified.
3.  The Gemini API provider `base_url` must be updated to the appropriate endpoint (typically `v1beta` for these models).
4.  The system must correctly route requests to these new models when selected.
5.  Billing and usage tracking must continue to work correctly for the new models.

## Affected Modules
- **`course-forge-backend`**: Specifically the AI provider services (`GeminiAIProvider`, `GeminiImageProvider`), configuration services (`AiModelService`, `AiProviderConfigService`), and the database migrations.

## Related Folder
- `docs/ai`

## Frontend Exclusion
- All UI/Frontend changes (dropdown updates, display changes in the browser) are excluded from this analysis.
