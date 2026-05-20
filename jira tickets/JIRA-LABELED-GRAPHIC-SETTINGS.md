# JIRA-LABELED-GRAPHIC-SETTINGS: Implement Configurable Layout, Media Layout, Style, Shape, and Interaction Settings for Labeled Graphic

## Description
Extend standard LABELED_GRAPHIC block functionality to support configurable layout spacing, media sizing, marker styling, image corner radius, and interactive image zoom behavior.

## Functional Changes Made

### Validation Integration
Enabled layout, style, shape, media layout, and interaction validation for the LABELED_GRAPHIC block type in `BlockContentValidator`, ensuring that properties like `sectionSpacing`, `verticalSpacing`, `imageSize`, `markerColor`, `roundedCorners`, and `openImageOnClick` are validated and normalized correctly.

### Media Layout Integration
Extended shared media layout utilities to support configurable image sizing behavior for labeled graphics using reusable responsive sizing patterns.

### Styling Integration
Added configurable marker color and border radius styling support using centralized color validation and reusable styling abstractions.

### Interaction Behavior Integration
Extended shared interactive media handling to conditionally enable image zoom/lightbox functionality for the main labeled image based on configuration values.

### Factory Defaults
Updated `BlockContentFactory` to initialize default Labeled Graphic configuration values for spacing, styling, media layout, and interaction behavior when new LABELED_GRAPHIC blocks are created.

### Schema Association
Defined the LABELED_GRAPHIC block in `BlockSchemaUtil` to inherit reusable shared layout, styling, media, shape, and interaction settings, enabling AI generation and renderer configuration support dynamically.

---

## Sample JSON

```json
{
  "id": "labeled-graphic-001",
  "type": "INTERACTIVE_IMAGE",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "mediaLayout": {
      "imageSize": "compact"
    },
    "style": {
      "markerColor": "#FFFFFF"
    },
    "shape": {
      "roundedCorners": 0
    },
    "interaction": {
      "openImageOnClick": true
    }
  },
  "content": {
    "backgroundImage": {
      "fileId": "labeled-graphic-bg-001",
      "altText": "Interactive labeled graphic background"
    },
    "markers": [
      {
        "id": "marker-1",
        "x": 35,
        "y": 42,
        "title": "Feature A",
        "description": "Description for Feature A"
      }
    ]
  }
}
```

---

## WorkLog Entries
* Added configurable layout support for LABELED_GRAPHIC blocks.
* Added sectionSpacing and verticalSpacing validation support.
* Extended reusable media sizing utilities for labeled image rendering.
* Added markerColor validation and theme token support.
* Added roundedCorners styling support for interactive media blocks.
* Extended shared media interaction utilities for openImageOnClick support.
* Updated validation and renderer logic for image interaction behavior handling.
* Added reusable border radius handling for media containers.
* Added LABELED_GRAPHIC configuration support to AI schema documentation.
* Added unit tests for labeled graphic configuration validation scenarios.
* Verified valid and invalid labeled graphic configuration handling.
* Added LABELED_GRAPHIC JSON sample to `docs\block-configuration\block_json_structures.md`
* Created a new JIRA ticket document for tracking.
