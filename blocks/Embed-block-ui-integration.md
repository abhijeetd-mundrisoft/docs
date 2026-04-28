# Step 1 – Resolve embed (URL or iframe)

Use this **before** saving an `EMBED` block so the server returns normalized `originalUrl`, `embedUrl`, and optional metadata (oEmbed for YouTube/Vimeo, optional Open Graph, otherwise raw link/iframe).

### API

```
POST /api/blocks/embed/resolve
```

*(Full URL with app `context-path`: base + `/api/blocks/embed/resolve` — same prefix as other block APIs.)*

### Auth

* Same as other protected block routes: `Authorization: Bearer <JWT>`
* Missing/invalid token → **401** (handled by JWT interceptor)

### Request

* `Content-Type: application/json`
* Body:

```json
{
  "input": "https://www.youtube.com/watch?v=VIDEO_ID"
}
```

or

```json
{
  "input": "<iframe src=\"https://player.vimeo.com/video/123456\" width=\"640\" height=\"360\" frameborder=\"0\" allow=\"autoplay; fullscreen; picture-in-picture\" allowfullscreen></iframe>"
}
```

**Validation (400 if invalid)**

* `input` is required, non-blank, max **20000** characters.

---

### Blocked words (optional server policy)

The backend can reject embeds whose **raw input** or **normalized URL** contains configured substrings (e.g. vulgar or banned terms). This is **substring** matching after **URL-decoding**, case-insensitive.

**Configuration** (`application.properties`)

| Property | Meaning |
| -------- | ------- |
| `embed.resolve.blocked-words` | Comma-separated list of substrings to block (empty = disabled). Every non-empty term is used as-is (including single-character terms). |

**Resolver behavior**

* If a match is found, the API still returns **HTTP 200** with **`data.resolved: false`** and a generic **`reason`** (the server does not echo the matched word).
* Typical message: `This URL is not allowed by content policy.`

**UI**

* Treat like any other `resolved: false` outcome; show `data.reason` to the user.

**Caveat**

* Substring rules can false-positive (e.g. a benign path segment containing a blocked substring, or a very short term such as a single letter). Prefer longer, specific phrases in the list when you need stricter matching.

---

### Response (important)

HTTP is **200** even when the URL cannot be embedded. Always inspect **`data.resolved`**.

**Success (`resolved: true`)**

```json
{
  "success": true,
  "status": 200,
  "message": "Embed resolved successfully",
  "data": {
    "resolved": true,
    "reason": null,
    "input": "https://www.youtube.com/watch?v=VIDEO_ID",
    "originalUrl": "https://www.youtube.com/watch?v=VIDEO_ID",
    "embedUrl": "https://www.youtube.com/embed/VIDEO_ID",
    "title": "…",
    "description": null,
    "thumbnail": "https://…",
    "provider": "YouTube",
    "providerUrl": "https://www.youtube.com/",
    "favicon": null,
    "embedType": "video",
    "showMetaData": true,
    "resolvedBy": "OEMBED"
  }
}
```

**Failure (`resolved: false`)** — blocked/unsafe URL, resolver disabled, or other policy outcome (still **200**)

```json
{
  "success": true,
  "status": 200,
  "message": "Embed could not be resolved",
  "data": {
    "resolved": false,
    "reason": "URL resolves to a blocked network address",
    "input": "https://…",
    "originalUrl": null,
    "embedUrl": null, 
    "title": null,
    "description": null,
    "thumbnail": null,
    "provider": null,
    "providerUrl": null,
    "favicon": null,
    "embedType": null,
    "showMetaData": null,
    "resolvedBy": null
  }
}
```

**`resolvedBy` when `resolved` is true**

| Value           | Meaning                                      |
| --------------- | -------------------------------------------- |
| `OEMBED`        | Provider oEmbed (YouTube / Vimeo)            |
| `OPEN_GRAPH`    | Metadata from HTML `og:*` tags               |
| `RAW_IFRAME`    | Fallback: use `embedUrl` as iframe `src`   |

---

### UI responsibility (resolve step)

* Call resolve when the user pastes a URL or iframe and confirms (e.g. “Insert” / “Preview”).
* If **`resolved === false`**: show `data.reason`; do not proceed to save unless product allows manual override (backend validation on save still requires safe **https** URLs).
* If **`resolved === true`**: copy fields from `data` into the block editor state; preview iframe should use **`embedUrl`** as `src`.
* Optionally persist the raw paste in **`input`** on the block for audit/debug.

---

# Step 2 – Create / update EMBED blocks

Same block APIs as other types (multipart create, bulk create, bulk update). Content is a **JSON object** (not a list).

### Bulk create (example)

```
POST /api/blocks/{lessonId}/bulk
```

### Request body (excerpt)

```json
{
  "blocks": [
    {
      "type": "EMBED",
      "orderIndex": 3,
      "content": {
        "input": "https://www.youtube.com/watch?v=VIDEO_ID",
        "originalUrl": "https://www.youtube.com/watch?v=VIDEO_ID",
        "embedUrl": "https://www.youtube.com/embed/VIDEO_ID",
        "title": "Optional title from resolver",
        "description": "",
        "thumbnail": "https://…",
        "provider": "YouTube",
        "providerUrl": "https://www.youtube.com/",
        "favicon": "",
        "showMetaData": true,
        "resolvedBy": "OEMBED"
      }
    }
  ]
}
```

---

### Important rules

* **`originalUrl`** and **`embedUrl`** are **required** on save; backend validates them and applies **SSRF-safe** checks (**https only** for persisted content in the current validator).
* Prefer values returned from **`POST /api/blocks/embed/resolve`** so `originalUrl` / `embedUrl` stay aligned with server normalization.
* **`fileUrl`** is **not** set by the backend for `EMBED` (do not rely on multipart file upload for this block type).
* `orderIndex` controls order like any other block.

---

# EMBED content structure

| Field            | Required | Description |
| ---------------- | -------- | ----------- |
| `originalUrl`    | Yes      | Canonical page URL (https), from resolver |
| `embedUrl`       | Yes      | Iframe `src` (https), from resolver |
| `input`          | No       | Original user paste (audit) |
| `title`          | No       | Display / card title |
| `description`    | No       | Subtitle or snippet |
| `thumbnail`      | No       | Preview image URL |
| `provider`       | No       | e.g. “YouTube”, “Vimeo” |
| `providerUrl`    | No       | Link to provider site |
| `favicon`        | No       | Small icon URL (if ever filled) |
| `showMetaData`   | No       | Default `true` if omitted in factory |
| `resolvedBy`     | No       | `OEMBED` \| `OPEN_GRAPH` \| `RAW_IFRAME` (validated if present) |

---

# Response handling (create / bulk)

* Parse returned block `content` as JSON like other map-shaped blocks.
* Editor preview: **`<iframe src="{embedUrl}" … />`** with your usual sandbox / `allow` attributes policy.

---

# Bulk update (optional)

```
PUT /api/blocks/{lessonId}/bulk-update
```

Use for reordering or updating `content` after another resolve call (e.g. user changes URL → resolve again → PATCH/bulk-update).

---

# Server configuration (reference for QA / devops)

Backend properties (defaults in `application.properties`):

| Property                         | Role |
| -------------------------------- | ---- |
| `embed.resolve.enabled`          | Master switch for resolver |
| `embed.resolve.allow-http`       | Allow `http` for **fetch** (persisted content still https-only in validator) |
| `embed.resolve.open-graph-enabled` | Try Open Graph when not YouTube/Vimeo oEmbed |
| `embed.resolve.max-redirects`    | Redirect cap for fetches |
| `embed.resolve.read-timeout-ms`  | Read timeout |

---

# Complete flow (quick checklist)

1. User pastes URL or single iframe → call **`POST /api/blocks/embed/resolve`** with `{ "input": "…" }`.
2. If **`data.resolved`** is false → show **`data.reason`**; stop or let user fix input.
3. Map **`data`** into **`content`** (at minimum `originalUrl`, `embedUrl`; carry optional metadata).
4. Create or update block with **`type`: `"EMBED"`** and that **`content`** via existing block APIs.
5. Render preview from **`embedUrl`**; optional chrome from `title`, `thumbnail`, `provider`, `showMetaData`.

---

# Related docs

* End-to-end product plan: `docs/blocks/embed-block-end-to-end-plan.md`
* Similar UI/API pattern (upload + bulk): `docs/blocks/Attachment-block-ui-integration.md`
