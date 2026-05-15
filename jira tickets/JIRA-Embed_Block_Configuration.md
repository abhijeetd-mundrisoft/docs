# JIRA Ticket: CC-513 - Embed Block Configuration Support

## Description
Extend standard `EMBED` block functionality to support granular layout configurations including section spacing, vertical spacing, embed size, and border visibility.

## Functional Changes Made

* **Schema Evolution**: Updated `BlockSchemaUtil` to include `embedSize` in `mediaLayout` and `showBorder` in `border` setting groups, ensuring AI generation and backend documentation are aligned with new capabilities.
* **Validation Integration**: Enabled comprehensive validation for the `EMBED` block type in `BlockContentValidator`. This includes:
    * `layout`: `sectionSpacing` (enum: narrow, regular, wide) and `verticalSpacing` (numeric range 0-150).
    * `mediaLayout`: `embedSize` (enum: compact, regular, large).
    * `border`: `showBorder` (boolean toggle).
* **Factory Defaults**: Updated `BlockContentFactory` to automatically initialize new `EMBED` blocks with standard configuration defaults, ensuring immediate consistency without manual authoring effort.
* **Documentation**: Added comprehensive JSON structure sample for the `EMBED` block type in `docs/block-configuration/block_json_structures.md`.

## Sample JSON
```json
{
  "type": "EMBED",
  "content": {
    "originalUrl": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
    "embedUrl": "https://www.youtube.com/embed/dQw4w9WgXcQ",
    "title": "Rick Astley - Never Gonna Give You Up",
    "showMetaData": true,
    "resolvedBy": "RAW_IFRAME"
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "embedSize": "regular"
    },
    "border": {
      "showBorder": true
    },
    "background": {
      "style": "Light"
    }
  }
}
```

## Worklog Entries
* Added layout, mediaLayout, and border configuration support for EMBED blocks.
* Implemented embedSize validation and default layout styling support.
* Extended border styling logic to support showBorder configuration handling for embed containers.
* Updated validation and factory logic for default style and enum validation handling.
* Added Embed block settings to AI schema and documented embed styling configuration.
* Added EMBED JSON sample to `docs/block-configuration/block_json_structures.md`.
* Created a new JIRA ticket document for tracking (CC-513).
