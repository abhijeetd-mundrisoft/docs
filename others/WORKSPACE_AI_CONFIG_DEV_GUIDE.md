# Developer Guide: Workspace AI Configuration CRUD Integration

This document outlines the requirements and implementation details for the Workspace AI Configuration system. Your task is to implement the CRUD (Create, Read, Update, Delete) operations that allow users to manage their AI settings via the UI.

---

## 1. System Overview
The system allows organizations (Workspaces) to:
*   Override system-default AI models (e.g., choice of GPT-4o vs GPT-4o-mini).
*   **BYOK (Bring Your Own Key)**: Use their own OpenAI API keys to bypass platform billing.

## 2. Database Tables
1.  **`workspace_ai_config`**: Stores the **current active settings**. One row per workspace/provider.
2.  **`workspace_ai_config_history`**: An **audit table**. Every time settings change, a snapshot must be saved here.

## 3. Core Dependencies (Already Implemented)
You do **not** need to implement the following; simply use them:
*   **`EncryptionUtil`**: Handles AES-256 encryption/decryption of API keys and SHA-256 fingerprinting.
*   **`WorkspaceAiConfigService`**: Contains the logic for resolving keys during AI calls.
*   **`AiModelService`**: Provides the list of active models for dropdowns.

---

## 4. Key Requirement: Cache Management ⚠️
Because AI providers (like `OpenAIProvider`) check these settings on every request, we use an in-memory `ConcurrentHashMap` cache in `WorkspaceAiConfigService` for high performance.

### **The Cache Rule:**
**Whenever you UPDATE or INSERT a configuration, you MUST update the cache.**

In the codebase, I have provided a helper method for this:
```java
private void invalidateAndRebuildCache(String workspaceId, String providerCode, WorkspaceAiConfig config) {
    String key = workspaceId + ":" + providerCode;
    cache.put(key, config);
}
```
**If you forget to update the cache during a `saveConfig` operation, the AI providers will continue using the OLD keys or models until the server restarts.**

---

## 5. Implementation Steps for CRUD

### A. Implementing `saveConfig`
When saving from the UI:
1.  **Check existing**: Use `repository.findByWorkspaceIdAndProviderCode` to see if a row exists.
2.  **Encrypt**: If the user provides a new API key, use `EncryptionUtil.encrypt(rawKey)` before saving it to the `api_key` column.
3.  **Masking check**: If the incoming API key is `********`, do NOT update the encrypted value in DB (this means the user kept the old key).
4.  **History**: call `recordHistory(config)` to create the audit snapshot.
5.  **Refresh Cache**: Call `invalidateAndRebuildCache()` immediately after `repository.save()`.

### B. Implementing `getConfigDto`
When loading for the UI:
1.  **Mask Secrets**: Use `EncryptionUtil.maskApiKey()` to ensure the real encrypted key is never sent to the frontend.
2.  **Fallback**: If no config exists, return a default DTO with `useCustomKey = false`.

---

## 6. Endpoints to Implement (Currently Commented Out)
Location: `WorkspaceAiConfigController.java`
*   `GET /api/workspace-ai-config`: Fetch current settings.
*   `POST /api/workspace-ai-config`: Save/Update settings.
*   `GET /api/workspace-ai-config/models`: Populate dropdowns using `aiModelService.getActiveModelsByType()`.

## 7. Security Context
Always resolve the `workspaceId` using `AuthUtil.getCurrentWorkspaceId()`. Never trust a `workspaceId` passed directly in a request body unless the user is a SUPER_ADMIN.
