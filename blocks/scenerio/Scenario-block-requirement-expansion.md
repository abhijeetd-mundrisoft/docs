## 1. Refined Requirement

Introduce a new lesson block type `SCENARIO` that supports branching, scene-based interactive conversations similar to Rise 360 scenario blocks.  
The block must persist a complete scenario graph (scenes -> slides -> responses -> transitions), character/pose setup, and scene background configuration through existing block APIs without breaking current block contracts.

Core capability expectations:
- Multiple scenes per scenario.
- Multiple slides per scene (`text` and `dialogue` minimum).
- Response-based branching (`goTo: next|slide|end`) with optional explicit `nextSlide`.
- Character with emotion-specific poses.
- Scene background media and behavior toggles (e.g., `blurBackground`, `slidesHaveCharacterByDefault`).
- Backend validation to prevent invalid graphs (broken references, impossible transitions, malformed payloads).

Out of scope (for v1 unless explicitly added):
- Runtime analytics/scoring for learner decisions.
- AI-generated scenario authoring.
- SCORM import/export parity for full scenario branching semantics.

---

## 2. Functional Scenarios

1. **Create scenario block with default scene**
   - Author adds `SCENARIO` block.
   - UI initializes one scene with starter slides and valid defaults.
   - Backend accepts save via bulk create.

2. **Add and edit scene**
   - Author edits scene title, background image, blur toggle, character visibility defaults.
   - Backend persists updated scene configuration.

3. **Add and edit slides**
   - Author adds `text` or `dialogue` slide in scene.
   - Author sets title/description/emotion/hasCharacter.
   - Backend validates allowed slide type and required fields.

4. **Add dialogue responses and branching**
   - Author defines response options with action, feedback, emotion, and navigation target.
   - Response can:
     - continue to next slide (`goTo=next`)
     - jump to specific scene+slide (`goTo=slide`)
     - end scenario (`goTo=end`)
   - Backend validates all references.

5. **Replay path**
   - Author configures response to jump back to first slide for replay.
   - Backend ensures referenced scene/slide exists.

6. **Update existing scenario block**
   - Author reorders scenes/slides or changes branching logic.
   - Backend updates content via bulk update and preserves valid graph.

7. **Load and render existing scenario**
   - UI fetches lesson blocks and hydrates scenario graph.
   - Runtime renderer follows transitions based on learner responses.

---

## 3. Edge Cases

- Empty `items`/no scene in scenario.
- Scene exists but has no slides.
- `dialogue` slide with zero responses when responses are required by UX.
- Duplicate IDs across scenes/slides/responses.
- `goTo=slide` but `nextSlide.scene` or `nextSlide.slide` missing.
- `nextSlide` points to non-existent scene or slide.
- Cyclic paths causing infinite loops with no possible end node.
- Invalid emotion key not present in character poses.
- `hasCharacter=true` while scene has no valid character config.
- `background.media.image` present with missing required image fields.
- HTML payloads in `description`/`feedback` containing unsupported tags or unsafe content.
- Extremely large scenario payload causing performance issues.

---

## 4. Validation Rules

### 4.1 Field-level validations
- Block `type` must be `SCENARIO`.
- Root `content` must be an object containing scenario data (recommended key: `items` list of scenes).
- Scene:
  - `id` required, unique in scenario.
  - `title` required (non-blank, length bounded).
  - `slides` required, non-empty.
- Slide:
  - `id` required, unique within scenario.
  - `type` allowed values: `text`, `dialogue` (v1).
  - `goTo` allowed values: `next`, `slide`, `end`.
  - `description` required for both slide types.
  - `emotion` required (must map to allowed emotion set).
- Response (for dialogue):
  - `id` required, unique within slide.
  - `action` allowed values (at least): `continue`, `tryAgain`.
  - `goTo` required (`next|slide|end`).
  - if `goTo=slide`, `nextSlide.scene` and `nextSlide.slide` required.

### 4.2 Business-level validations
- At least one terminal path (`goTo=end`) must be reachable.
- All slide jump references must resolve.
- No orphan scene/slide references.
- Optional: enforce max counts (scenes, slides per scene, responses per slide) for performance.
- Optional: sanitize rich text fields (`description`, `feedback`) with existing HTML policy.

---

## 5. API Contract

### Endpoint
- Existing block APIs (no new save endpoint required):
  - `POST /api/blocks/{lessonId}/bulk`
  - `PUT /api/blocks/{lessonId}/bulk-update`
  - Existing block read endpoints for hydration.

### Method
- `POST` for create
- `PUT` for update

### Request payload (excerpt)
```json
{
  "blocks": [
    {
      "type": "SCENARIO",
      "orderIndex": 3,
      "content": {
        "items": [
          {
            "id": "scene-1",
            "title": "Scene 1",
            "slides": [
              {
                "id": "slide-1",
                "type": "text",
                "goTo": "next",
                "title": "Scenario Title",
                "emotion": "neutral",
                "description": "<p>Intro</p>",
                "hasCharacter": true,
                "responses": []
              }
            ],
            "character": {
              "id": "photo-female-00021",
              "name": "Laura",
              "poses": {
                "neutral": { "id": "326", "src": "..." },
                "happy": { "id": "321", "src": "..." }
              }
            },
            "background": {
              "media": {
                "image": { "src": "...", "type": "image" }
              }
            },
            "blurBackground": true,
            "slidesHaveCharacterByDefault": true
          }
        ]
      }
    }
  ]
}
```

### Response payload
- Standard existing wrapper:
  - `ApiResponseDto` with created/updated block DTO(s).
  - `content` may be returned as JSON string in block response DTO depending on current mapper behavior.

### Error responses
- `400`: invalid scenario structure / broken branching references / invalid enums.
- `401`: unauthorized.
- `404`: lesson/block not found (existing behavior paths).
- `500`: unexpected server error.

---

## 6. Pagination Requirements

No new pagination requirement for scenario save/update itself.  
Scenario content is a single block payload and should be constrained by payload size/field limits.

Existing lesson block listing pagination remains unchanged.

---

## 7. Logging Requirements

- **Request logs**
  - Log scenario create/update intent with `lessonId`, block count, user/workspace context.
  - Avoid logging full scenario payload (can be large).

- **Response logs**
  - Log success with block IDs and scenario scene/slide counts.

- **Error logs**
  - Validation failures at `warn` with concise reason.
  - Unexpected failures at `error` with stack trace and correlation identifiers.

- **Performance logs**
  - Log validation duration for large scenario payloads.
  - Optional warn threshold if validation exceeds configured latency.

---

## 8. Performance Considerations

- Scenario payload can become large (nested lists and media metadata).
- Implement efficient validation:
  - single-pass index maps for scene/slide lookup
  - avoid repeated nested scans for each reference
- Enforce practical limits:
  - max scenes per block
  - max slides per scene
  - max responses per dialogue
  - max HTML length per field
- Consider compression/size guard at request level for very large content.

---

## 9. Security Considerations

- **Authentication:** JWT required (existing protected block APIs).
- **Authorization:** reuse lesson/workspace authorization checks from existing block flows.
- **Data exposure risks:** avoid returning internal validation stack traces.
- **Input sanitization:**
  - sanitize HTML in `description` and `feedback`.
  - validate external media URLs for character/background assets.
  - reject script/event-handler injection in rich text.
- **Integrity:** enforce branching reference correctness to prevent malformed runtime graphs.

---

## 10. Failure Scenarios

1. **Broken branching target**
   - `goTo=slide` points to missing scene/slide.
   - Expected: 400 with precise validation message.

2. **Invalid enum values**
   - unsupported `goTo`, `type`, `emotion`, or `action`.
   - Expected: 400.

3. **Oversized scenario payload**
   - too many nested elements or large HTML.
   - Expected: 400 with limit message.

4. **Unsafe rich text**
   - HTML contains disallowed tags/attributes.
   - Expected: 400 or sanitized save based on policy.

5. **Unauthorized request**
   - missing/invalid token.
   - Expected: 401.

6. **Concurrent edits overwrite**
   - simultaneous updates by different users.
   - Expected: current behavior applies; if optimistic locking is absent, last write wins (document as risk).

7. **Runtime dead-end path**
   - no valid progression/end from a learner-selected branch.
   - Expected: prevent at validation time where detectable; otherwise runtime should fail gracefully with fallback/end message.

