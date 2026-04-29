# Functional Context - Optional Subheading Field (CC-419)

## 1. Requirement Overview
The requirement is to make the `Subheading` field optional for specific course block types. Currently, the system enforces a mandatory check on both `Subheading` and `Paragraph` (text) fields. The goal is to allow users to leave the subheading empty while keeping the paragraph content mandatory.

## 2. Current vs. Expected Functionality

### Current Functionality
- For block types like `TEXT_WITH_SUBHEADING`, the system requires both `subheading` and `text` fields to be non-empty strings.
- Saving a block without a subheading results in a validation error (`400 Bad Request`).
- AI generation always generates a subheading for these block types.

### Expected Functionality
- For `TEXT_WITH_SUBHEADING` blocks, the `subheading` field is optional, but the `text` field remains mandatory.
- For `SUBHEADING_ONLY` blocks, the `subheading` field is optional.
- The system allows saving and rendering blocks with only paragraph content.
- AI generation is updated to treat subheadings as optional.

## 3. Acceptance Criteria (AC)
- **AC 1**: Given a user deletes the subheading but keeps the paragraph in a `TEXT_WITH_SUBHEADING` block, when saved, then the system must allow the operation.
- **AC 2**: The system must render correctly on the backend/SCORM export with only the paragraph field populated.
- **AC 3**: The paragraph (`text`) field must remain mandatory for `TEXT_WITH_SUBHEADING` blocks.

## 4. Expected Results
- Users can successfully save blocks without a subheading.
- No regression in other block types.
- API validation remains robust for mandatory fields (like the paragraph text).

## 5. Related Module Folder
This functionality is related to the **`blocks`** module within the `/docs` repository.

## 6. Affected Modules
The following backend modules are affected:
- **Validation Module**: Logic within `BlockContentValidator` and `TemplateBlockContentValidator`.
- **Content Factory Module**: Logic within `BlockContentFactory` regarding default values.
- **AI Schema Utility**: `BlockSchemaUtil` providing the AI prompt context.
- **API Documentation**: Swagger/OpenAPI annotations in `BlockController`.

*Note: Frontend/UI changes are excluded from this analysis.*
