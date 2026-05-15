# Extended Blocks Sample JSON Reference (CC-473)

This document provides a complete collection of sample JSON payloads for all block types extended with the new `layout` and `background` styling infrastructure.

## 1. Paragraph Block (`TEXT`)
*Scenario: Regular width with a standard Gray preset background.*

```json
{
  "type": "TEXT",
  "content": {
    "text": "<p>This is a standard paragraph block using the Gray preset style.</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "regular",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "background": {
      "style": "Gray"
    }
  }
}
```

## 2. Heading with Text (`HEADING_TEXT`)
*Scenario: Compact width with an Image background and custom crop.*

```json
{
  "type": "HEADING_TEXT",
  "content": {
    "heading": "<h2>Innovative Design</h2>",
    "text": "<p>Our approach combines aesthetics with high-performance functionality.</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "compact",
      "sectionSpacing": "wide",
      "verticalSpacingTop": 50,
      "verticalSpacingBottom": 50,
      "verticalSpacingLinked": true,
      "borderRadius": 12
    },
    "background": {
      "style": "Image",
      "image": {
        "fileId": "forest-bg-01",
        "altText": "Lush green forest",
        "crop": { "x": 0, "y": 20, "width": 100, "height": 60 }
      }
    }
  }
}
```

## 3. Heading Only (`HEADING_ONLY`)
*Scenario: Large width with a Custom HEX color and Auto contrast handling.*

```json
{
  "type": "HEADING_ONLY",
  "content": {
    "heading": "<h1>Join Our Global Team</h1>"
  },
  "settings": {
    "layout": {
      "contentArea": "large",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 80,
      "verticalSpacingBottom": 20,
      "verticalSpacingLinked": false
    },
    "background": {
      "style": "Custom",
      "customColor": {
        "hex": "#2c3e50",
        "contrastMode": "Auto"
      }
    }
  }
}
```

## 4. Text with Subheading (`TEXT_WITH_SUBHEADING`)
*Scenario: Theme Tint background with rounded corners.*

```json
{
  "type": "TEXT_WITH_SUBHEADING",
  "content": {
    "subheading": "Core Values",
    "text": "<p>Integrity, Innovation, and Inclusion drive everything we do.</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "regular",
      "borderRadius": 20
    },
    "background": {
      "style": "Theme Tint"
    }
  }
}
```

## 5. Subheading Only (`SUBHEADING_ONLY`)
*Scenario: Minimalist Black background with high vertical spacing.*

```json
{
  "type": "SUBHEADING_ONLY",
  "content": {
    "subheading": "Section 4: Advanced Analytics"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true,
      "borderRadius": 8
    },
    "background": {
      "style": "Custom",
      "customColor": {
        "hex": "#F4A261",
        "contrastMode": "Auto"
      }
    }
  }
}
```

## 6. Table Block (`TABLE`)
*Scenario: Wide section with cell spacing and a Custom background.*

```json
{
  "type": "TABLE",
  "content": {
    "rows": 3,
    "columns": 2,
    "data": [
      [
        {"content": "<strong>Feature</strong>", "hAlign": "left", "vAlign": "middle"},
        {"content": "<strong>Benefit</strong>", "hAlign": "left", "vAlign": "middle"}
      ],
      [
        {"content": "Live Support", "hAlign": "left", "vAlign": "top"},
        {"content": "Instant resolution of issues", "hAlign": "left", "vAlign": "top"}
      ],
      [
        {"content": "Cloud Sync", "hAlign": "left", "vAlign": "top"},
        {"content": "Access data from anywhere", "hAlign": "left", "vAlign": "top"}
      ]
    ],
    "config": {
      "hasHeaderRow": true,
      "hasHeaderColumn": false,
      "bordered": true
    }
  },
  "settings": {
    "layout": {
      "contentArea": "large",
      "sectionSpacing": "wide",
      "verticalSpacingTop": 40,
      "verticalSpacingBottom": 40,
      "verticalSpacingLinked": true
    },
    "tableLayout": {
      "cellSpacing": "regular"
    },
    "background": {
      "style": "Custom",
      "customColor": {
        "hex": "#ffffff",
        "contrastMode": "Auto"
      }
    }
  }
}
```
