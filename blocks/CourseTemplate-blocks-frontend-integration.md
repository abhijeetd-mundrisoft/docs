# Course template blocks — frontend integration

This guide covers **template lesson blocks** under `CourseTemplateController`: list, read, bulk create, bulk update, reorder, and delete. **Authentication and token handling are omitted** (your UI team already owns that); every call below still needs your standard `Authorization: Bearer …` header.

## Base URL

From `application.properties`:

- **Context path:** `/api`
- **Controller prefix:** `/course-templates`

All paths below are relative to: **`{origin}/api/course-templates`**

Example: `GET https://<host>/api/course-templates/lessons/{lessonId}/blocks`

---

## Envelope: `ApiResponseDto`

Successful JSON shape (simplified):

```json
{
  "success": true,
  "status": 200,
  "message": "…",
  "data": { },
  "count": null,
  "timestamp": "2026-04-17T12:00:00"
}
```

Errors use `success: false`, HTTP status in `status`, and an `error` field (see `ApiResponseDto` in the backend). Validation issues may include a `details` array.

---

## Template block in responses (`TemplateBlockResponseDto`)

Used for **list**, **get**, **reorder**, and each item inside bulk create/update payloads.

| Field | Type | Notes |
|--------|------|--------|
| `id` | string (UUID) | Block id |
| `lessonId` | string | Template lesson id |
| `type` | string | Same enum names as lesson blocks, e.g. `PROCESS`, `TIMELINE`, `TITLE` |
| `contentJson` | **string** | JSON **serialized as a string** (parse on the client with `JSON.parse` if you need an object/array) |
| `orderIndex` | number | Block order **within the lesson** (1-based after server normalization) |
| `createdAt` / `updatedAt` | ISO datetime | |

### List-shaped block types (`contentJson`)

For types such as **`PROCESS`**, **`TIMELINE`**, and **`FLASH_CARDS`**, `contentJson` is a **JSON array** of items; each item should carry its own **`orderIndex`** for authoring.

**Read path behavior:** the API **sorts that root array by each item’s `orderIndex`** (ascending) when returning `contentJson`, so the UI can render panels in order without sorting locally. **Writes** should still send sensible `orderIndex` values per item so updates stay predictable.

---

## Endpoints

### 1. List blocks for a lesson

`GET /lessons/{lessonId}/blocks`

- **Response `data`:** `TemplateBlockResponseDto[]` (array).
- Blocks are ordered by **`orderIndex`** (lesson-level order).

### 2. Get one block

`GET /template-blocks/{blockId}`

- **Response `data`:** single `TemplateBlockResponseDto`.

### 3. Bulk create blocks

`POST /lessons/{lessonId}/blocks`

**Body:** `BulkBlockCreateRequestDto`

```json
{
  "blocks": [
    {
      "type": "PROCESS",
      "content": [
        {
          "type": "intro",
          "title": "Overview",
          "description": "<p>…</p>",
          "isHidden": false,
          "orderIndex": 1
        },
        {
          "type": "step",
          "title": "Step 1",
          "description": "<p>…</p>",
          "isHidden": false,
          "orderIndex": 2
        }
      ],
      "orderIndex": 2
    }
  ]
}
```

- **`type`:** string, must match a supported block type enum (uppercase in examples; server accepts normalized values as implemented).
- **`content`:** object or array — sent as JSON; stored after validation and optional defaults (`ensureRequiredFields`).
- **`orderIndex`:** required on each create item (`BlockCreateItemDto`); block position in the lesson.

**Response `data`:** `BulkTemplateBlockCreateResponseDto`

- `createdBlocks`: `TemplateBlockResponseDto[]`
- `totalCreated`: number

### 4. Bulk update blocks

`PUT /lessons/{lessonId}/blocks`

**Body:** `TemplateBlocksBulkUpdateDto`

```json
{
  "blocks": [
    {
      "id": "existing-block-uuid",
      "type": "PROCESS",
      "content": [ … ],
      "order": 1
    }
  ]
}
```

**Important naming:** bulk update uses **`order`** (not `orderIndex`) for the block’s position in the lesson. **`content`** is a `JsonNode` in the API — send a normal JSON object/array; the server stringifies it for validation.

After a successful update, the server **renormalizes** lesson block order to contiguous `1..n` by `orderIndex`.

**Response `data`:** `BulkTemplateBlockUpdateResponseDto`

- `updatedBlocks`: `TemplateBlockResponseDto[]`
- `totalUpdated`: number

### 5. Reorder blocks (IDs only)

`PATCH /lessons/{lessonId}/blocks/reorder`

**Body:** `BlocksReorderDto`

```json
{
  "blockIds": ["id-1", "id-2", "id-3"]
}
```

- Must include **every** block id for that lesson, in the desired order.
- **Response `data`:** `TemplateBlockResponseDto[]` (full blocks after save).

### 6. Delete a block

`DELETE /template-blocks/{blockId}`

- **Response:** follow existing `ApiResponseDto` pattern for delete (message / data as implemented).

---

## Practical notes for the UI

1. **`contentJson` is a string** on reads — parse once when binding to your editor model.
2. **Create** uses `orderIndex` on each block row; **bulk update** uses **`order`** for the same concept — align your form/state field names to avoid silent bugs.
3. For **PROCESS / TIMELINE / FLASH_CARDS**, treat inner list **`orderIndex`** as the source of truth when saving; rely on the API’s sorted array on load if you want a simple render path.
4. **Swagger / OpenAPI:** generate or refresh the client from the running app’s OpenAPI doc if you use codegen (paths and DTO names should match this controller).

---

## Reference (backend)

| Piece | Location |
|--------|-----------|
| Routes | `CourseTemplateController` — `@RequestMapping("/course-templates")` |
| Response mapping & list sort | `TemplateBlockMapper`, `JsonListOrderIndexSorter` |
| Create body | `BulkBlockCreateRequestDto`, `BlockCreateItemDto` |
| Update body | `TemplateBlocksBulkUpdateDto`, `TemplateBlockUpdateDto` |
| Reorder body | `BlocksReorderDto` |
