# JIRA-522: Video Text Bottom Spacing, Layout and Playback Configuration

## Description

Extend the `VIDEO_TEXT_BOTTOM` and `TEXT_VIDEO_BOTTOM` block types to support granular layout, styling, playback, and positioning configurations. This ensures consistency with other media-text layout blocks like `VIDEO_TEXT_RIGHT`.

## Functional Changes Made

* **Validation Integration**: Enabled `layout`, `mediaLayout`, `shape`, and `playback` settings validation for `VIDEO_TEXT_BOTTOM` and `TEXT_VIDEO_BOTTOM` in `BlockContentValidator`.
* **Standardization**: Renamed `videoSize` to `videoWidth` across the shared media layout settings to align with functional requirements.
* **Media Layout Extension**: Added support for `top` and `bottom` positions in `videoPosition` setting.
* **Factory Defaults**: Updated `BlockContentFactory` to provide standard layout, media sizing, and playback defaults for these block types during initialization.
* **Schema Association**: Updated `BlockSchemaUtil` to include settings in the AI generation schema, allowing dynamic configuration of these blocks.

## Sample JSON

```json
{
  "type": "VIDEO_TEXT_BOTTOM",
  "content": {
    "video": {
      "title": "Module Overview",
      "fileId": "video-uuid-001"
    },
    "text": "<p>This is the description text that appears above or below the video.</p>"
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "videoWidth": "regular",
      "videoPosition": "bottom"
    },
    "shape": {
      "borderRadius": 0
    },
    "playback": {
      "forwardSeeking": true
    },
    "background": {
      "style": "Light"
    }
  }
}
```
