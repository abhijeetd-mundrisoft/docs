# 🚀 Step 1 – Upload Attachment

### API

```
POST /api/blocks/attachments/upload
```

### Request

* `multipart/form-data`
* Fields:

  * `file` (required)
  * `lessonId` (required)

---

### ✅ Response (Important)

```json
{
  "data": {
    "fileId": "f111-222-333",
    "fileUrl": "/api/files/f111-222-333",
    "originalFilename": "handout.zip",
    "contentType": "application/zip",
    "fileSize": 24050692
  }
}
```

---

### 🎯 UI Responsibility

Store:

* `fileId` ✅ (mandatory)
* Optional: name, size, type

👉 Repeat for each file

---

# 🗑️ Step 1b – Discard Upload (If User Cancels)

### API

```
DELETE /api/files/{fileId}
```

---

### ✅ Success

```json
{
  "success": true,
  "message": "File deleted successfully"
}
```

### ❌ Error (409 – In Use)

```json
{
  "error": "File cannot be deleted because it is referenced..."
}
```

---

### 🎯 UI Logic

* If user removes file before save → call DELETE
* If 409 → file already used → show message

---

# 🚀 Step 2 – Bulk Create Blocks

### API

```
POST /api/blocks/{lessonId}/bulk
```

---

### Request Body

```json
{
  "blocks": [
    {
      "type": "TITLE",
      "orderIndex": 1,
      "content": {
        "text": "<h2>Week 1</h2>"
      }
    },
    {
      "type": "ATTACHMENT",
      "orderIndex": 2,
      "content": {
        "fileId": "f111-222-333",
        "displayName": "SCORM package",
        "description": "Download file",
        "allowDownload": true
      }
    }
  ]
}
```

---

### ⚠️ Important Rules

* `fileId` must come from upload API
* Always send inside `content.fileId`
* `orderIndex` controls order

---

# 📥 Response Handling (Very Important)

```json
{
  "data": {
    "createdBlocks": [
      {
        "id": "block-1",
        "content": "{\"fileId\":\"f111\"}"
      }
    ]
  }
}
```

---

# 📦 ATTACHMENT Content Structure

| Field           | Required | Description |
| --------------- | -------- | ----------- |
| `fileId`        | ✅        | From upload |
| `displayName`   | ❌        | UI label    |
| `description`   | ❌        | Notes       |
| `allowDownload` | ❌        | true/false  |

---

# ⬇️ Download File

### API

```
GET /api/files/{fileId}
```

### Behavior

* Returns file stream
* `Content-Disposition = attachment` (forced download)

---

# 🔄 Bulk Update (Optional)

```
PUT /api/blocks/{lessonId}/bulk-update
```

Used for:

* Reordering
* Updating content
* Replacing file

---

# ✅ Complete Flow (Quick Checklist)

1. Upload file → get `fileId`
2. (Optional) Delete if user cancels
3. Create blocks → use `fileId`
4. Parse response content
5. Download via `/api/files/{fileId}`

---
