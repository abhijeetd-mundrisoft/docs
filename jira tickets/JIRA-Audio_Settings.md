# JIRA Ticket: Configurable Settings for Audio Block Type

## Description
Implement support for configurable `Layout` and `Playback` settings for the `Audio` block type (backend: `AUDIO_PLAYER`). This will enable authors to control section spacing, vertical spacing, and playback behavior (like forward seeking restriction) for audio content.

## Acceptance Criteria
- [ ] `AUDIO_PLAYER` block supports `layout` settings:
    - `sectionSpacing` (enum: `narrow`, `regular`, `wide`, default `narrow`)
    - `verticalSpacingTop` (numeric: `0-150`, default `25`)
    - `verticalSpacingBottom` (numeric: `0-150`, default `25`)
    - `verticalSpacingLinked` (boolean, default `true`)
- [ ] `AUDIO_PLAYER` block supports `playback` settings:
    - `forwardSeeking` (boolean, default `true`)
- [ ] Settings are validated on the backend in `BlockContentValidator`.
- [ ] Default settings are automatically applied for new blocks in `BlockContentFactory`.
- [ ] `BlockSchemaUtil` is updated to include the new settings for AI generation.
- [ ] Documentation is updated with JSON samples.
- [ ] Existing `AUDIO_PLAYER` blocks remain functional (backward compatibility).

## Technical Details
- **Component**: `Audio` (Backend: `AUDIO_PLAYER`)
- **Setting Categories**:
    - `Media > Layout`
    - `Media > Playback`
- **JSON Structure**:
```json
"settings": {
  "layout": {
    "sectionSpacing": "narrow",
    "verticalSpacingTop": 25,
    "verticalSpacingBottom": 25,
    "verticalSpacingLinked": true
  },
  "playback": {
    "forwardSeeking": true
  },
  "background": {
    "style": "Light"
  }
}
```

## Testing Requirements
- Unit tests for validation logic.
- Unit tests for factory defaults.
- Integration tests for persistence.
