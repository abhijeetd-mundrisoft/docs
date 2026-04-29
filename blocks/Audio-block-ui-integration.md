# 🚀 Step 1 – Upload Audio File

## 📚 Asset Library Upload (Image / Video / Audio)

Use the same upload API for all media assets that should appear in the asset library.

### API

```
POST /api/files/upload
```

### Request

* `multipart/form-data`
* Fields:
  * `file` (required)
  * Optional metadata:
    * `altText`
    * `description`
    * `tag`
    * `sessionId`

### Supported media (common)

* **Image:** `image/png`, `image/jpeg`, `image/webp`, `image/gif`, `image/svg+xml`
* **Video:** `video/mp4`, `video/webm`, `video/ogg`, `video/quicktime`
* **Audio:** `audio/mpeg`, `audio/wav`, `audio/mp4`, `audio/ogg`, `audio/opus`

### Response mapping for UI/Asset Library

Store these for asset-card rendering and block binding:

* `fileId` (primary key for block content)
* `fileUrl` (preview/playback/download URL)
* `originalFilename`
* `contentType` (to identify image/video/audio card type)
* `fileSize`

---

### API

```
POST /api/files/upload
```

### Request

* `multipart/form-data`
* Fields:

  * `file` (required)
  * Optional metadata:
    * `altText`
    * `description`
    * `tag`
    * `sessionId`

---

### ✅ Response (Important)

```json
{
  "data": {
    "fileId": "f111-222-333",
    "fileUrl": "/api/files/f111-222-333",
    "originalFilename": "intro.mp3",
    "contentType": "audio/mpeg",
    "fileSize": 2405069
  }
}
```

---

### 🎯 UI Responsibility

Store:

* `fileId` ✅ (mandatory for AUDIO_PLAYER block save)
* Optional for display: `originalFilename`, `contentType`, `fileSize`

Supported audio formats include: `MP3`, `WAV`, `M4A`, `OGG`, `OPUS`, `OGA`.

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
        "text": "<h2>Lesson Intro</h2>"
      }
    },
    {
      "type": "AUDIO_PLAYER",
      "orderIndex": 2,
      "content": {
        "fileId": "f111-222-333",
        "title": "Introduction Audio",
        "caption": "Optional short caption"
      }
    }
  ]
}
```

---

### ⚠️ Important Rules

* `type` must be exactly `AUDIO_PLAYER`
* Always send uploaded audio id as `content.fileId`
* `title` and `caption` are optional
* `orderIndex` controls block order in lesson

---

# 📥 Response Handling (Very Important)

```json
{
  "data": {
    "createdBlocks": [
      {
        "id": "block-1",
        "content": "{\"fileId\":\"f111-222-333\",\"title\":\"Introduction Audio\",\"caption\":\"Optional short caption\"}"
      }
    ]
  }
}
```

---

### 🎯 UI Parsing Note

`content` may come back as a JSON string in block responses.  
UI should parse safely before binding to editor state.

---

# 📦 AUDIO_PLAYER Content Structure

| Field | Required | Description |
| ----- | -------- | ----------- |
| `fileId` | ✅ | Audio file ID from `/api/files/upload` |
| `title` | ❌ | Display title in player UI |
| `caption` | ❌ | Optional helper text |

---

# ▶️ Play Audio

### API

```
GET /api/files/{fileId}
```

### Behavior

* Returns file stream for uploaded audio
* Use URL in `<audio>` source:

```html
<audio controls src="/api/files/f111-222-333"></audio>
```

---

# 🔄 Bulk Update (Optional)

### API

```
PUT /api/blocks/{lessonId}/bulk-update
```

Used for:

* Reordering blocks
* Updating `title` / `caption`
* Replacing audio (`content.fileId` with new uploaded file id)

---

# ✅ Complete Flow (Quick Checklist)

1. Upload audio via `/api/files/upload` -> get `fileId`
2. Create block using `type: AUDIO_PLAYER` and `content.fileId`
3. Parse returned `content` safely
4. Render playback using `/api/files/{fileId}`

---
