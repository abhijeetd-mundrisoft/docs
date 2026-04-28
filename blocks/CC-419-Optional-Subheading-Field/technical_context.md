# Technical Context - Optional Subheading Field (CC-419)

## 1. Logical Flow of Code Changes

The implementation follows a top-down approach from API documentation to the core validation logic:
1.  **API Documentation**: Update the `BlockController` to reflect the optional status of the subheading field in the OpenAPI documentation.
2.  **AI Integration**: Update the `BlockSchemaUtil` to ensure the AI model is aware that subheadings are optional during course generation.
3.  **Service Layer (Validation)**: Modify the validation logic in `BlockContentValidator` and `TemplateBlockContentValidator` to remove mandatory checks for the `subheading` field.
4.  **Service Layer (Sanitization)**: Ensure `BlockContentFactory` handles potential null/empty subheadings during the sanitization process.

## 2. Required Code Modifications

### 2.1 Validation Layer

**Files**: 
- `com.mundrisoft.courseforge.service.BlockContentValidator`
- `com.mundrisoft.courseforge.service.TemplateBlockContentValidator`

**Changes**:
In `validateParagraphWithSubheadingContent` and `validateSubheadingContent`, remove the `throw` statements associated with the subheading field being null or empty.

```java
// Logic to be updated
Object subheading = content.get("subheading");
// Remove the check: 
// if (subheading == null || !(subheading instanceof String) || ((String) subheading).trim().isEmpty()) {
//     throw new IllegalArgumentException("subheading text is required...");
// }
```

### 2.2 AI Schema Utility

**File**: `com.mundrisoft.courseforge.util.BlockSchemaUtil`

**Changes**:
Update the `BLOCK_SCHEMA_JSON` constant to change the metadata for subheading from `(required)` to `(optional)`.

```json
"TEXT_WITH_SUBHEADING": {
  "content": {
    "subheading": "string (optional)",
    "text": "string (required)"
  }
}
```

## 3. Sample JSON Structure

### Updated TEXT_WITH_SUBHEADING
```json
{
  "subheading": "", 
  "text": "<p>This is the mandatory paragraph content.</p>"
}
```

### Updated SUBHEADING_ONLY (Optional)
```json
{
  "subheading": ""
}
```

## 4. CRUD Operations & Persistence
- **Create/Update**: The persistence layer (JPA) remains unchanged as content is stored as a JSON string in the `content` column of the `Block` entity. The `BlockContentValidator` will now allow payloads with empty subheadings to proceed to the service layer for saving.
- **Read**: No changes required. The JSON returned to the client will contain either an empty string or null for the subheading.

## 5. Implementation Plan

### Step 1: Development
1.  Modify `BlockContentValidator.java` and `TemplateBlockContentValidator.java` to relax subheading validation.
2.  Update `BlockSchemaUtil.java` with the new schema hints for AI.
3.  Update `BlockController.java` annotations for accurate Swagger documentation.

### Step 2: Testing
1.  **Unit Testing**: Verify `BlockContentValidator` with a `Map` containing only `text`.
2.  **Integration Testing**: Perform `POST /api/blocks` and `PUT /api/blocks/{id}` with an empty subheading and verify `200 OK`.
3.  **AI Verification**: Run a mock AI generation to ensure subheadings are treated as optional in the prompts.

### Step 3: Deployment
1.  Standard CI/CD pipeline deployment to staging.
2.  Verification in staging environment with manual UI/API tests.
3.  Production deployment.

### Step 4: Rollback Strategy
1.  Revert the changes in the validator classes to restore the mandatory requirement.
2.  Redeploy the previous stable version of the backend service.

## 6. API Impact Analysis

| Endpoint | Modification Required | Description |
| :--- | :--- | :--- |
| `POST /api/blocks` | Yes (Validation Layer) | Update validation logic to allow empty subheadings. |
| `PUT /api/blocks/{id}` | Yes (Validation Layer) | Update validation logic to allow clearing of subheadings. |
| `POST /api/blocks/{lessonId}/bulk` | Indirectly (Via Validator) | Bulk creation will use the updated validation logic. |
| `GET /api/blocks/block-types` | Yes (Documentation) | Update the documentation returned by this endpoint. |

## 7. Consistency Rules
- **Layered Architecture**: Maintain the existing `Controller -> Service -> Repository` layering. Business logic must reside in the Service layer, and validation must be triggered at the boundary.
- **DTO Usage**: All API responses must continue to use `ApiResponseDto<T>` for consistency with the established REST contract.
- **Validation Standards**: Use annotation-based validation where possible, and reserve manual checks in the Service layer for cross-field invariants.
- **Import Style**: Always use explicit import statements (e.g., `import com.mundrisoft.courseforge.service.BlockContentValidator;`) rather than referring to classes via direct package paths.
- **Minimal Edits**: Apply changes only to the files and logic paths directly related to this requirement to minimize regression risks in legacy code.

## 8. Potential Impact on Other Modules
- **SCORM Export**: The parser in `com.mundrisoft.courseforge.scorm.editor.parser.RiseParser` or equivalent must handle missing subheading nodes gracefully.
- **Search Indexing**: If subheadings are indexed for search, the indexing logic should skip null/empty values to avoid pollution.
