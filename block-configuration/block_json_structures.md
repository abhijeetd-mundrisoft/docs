# Block Configuration JSON Structures

This document provides reference JSON structures for the `settings` field of various authoring components. 
All blocks follow a consistent **Layout** and **Background** nesting pattern.

---

## 1. Text Blocks

### Paragraph Block (`TEXT`)
**UI Sections**: Layout, Background

```json
{
  "type": "TEXT",
  "content": {
    "text": "<p>This is a standard paragraph block using the Gray preset style.</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",       // Options: compact, regular, large
      "sectionSpacing": "regular",    // Options: narrow, regular, wide
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "background": {
      "style": "Gray"                 // Options: Light, Gray, Theme, Dark, Image, Custom
    }
  }
}
```

### Heading with Text (`HEADING_TEXT`)
**UI Sections**: Layout, Background

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
        "url": "https://images.unsplash.com/...",
        "crop": { "x": 0, "y": 20, "width": 100, "height": 60 }
      }
    }
  }
}
```

### Heading Only (`HEADING_ONLY`)
**UI Sections**: Layout, Background

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

### Text with Subheading (`TEXT_WITH_SUBHEADING`)
**UI Sections**: Layout, Background

```json
{
  "type": "TEXT_WITH_SUBHEADING",
  "content": {
    "subheading": "<h3>Our Mission</h3>",
    "text": "<p>To empower creators worldwide through intuitive design tools.</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "regular",
      "verticalSpacingTop": 30,
      "verticalSpacingBottom": 30,
      "verticalSpacingLinked": true
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

## 2. Announcement Blocks

### Statement (`ANNOUNCEMENT`)
**UI Sections**: Layout, Background, Divider

```json
{
  "type": "ANNOUNCEMENT",
  "content": {
    "text": "<p>This is a major announcement for all users.</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "wide",
      "verticalSpacingTop": 40,
      "verticalSpacingBottom": 40,
      "verticalSpacingLinked": true
    },
    "background": {
      "style": "Theme"
    },
    "divider": {
      "show": true,
      "size": "medium",               // Options: small, medium, large
      "spacing": "regular"            // Options: narrow, regular, wide
    }
  }
}
```

### Note (`ANNOUNCEMENT_NOTE`)
**UI Sections**: Layout, Background, Note Styling

```json
{
  "type": "ANNOUNCEMENT_NOTE",
  "content": {
    "text": "<p>Please remember to save your work before exiting.</p>",
    "icon": "info"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "regular",
      "verticalSpacingTop": 20,
      "verticalSpacingBottom": 20,
      "verticalSpacingLinked": true,
      "borderRadius": 8
    },
    "background": {
      "style": "Light"
    },
    "note": {
      "size": "small",                // Options: small, medium, large
      "color": "#FFEFE9",
      "showIcon": true,
      "iconColor": "#FF631E"
    }
  }
}
```

---

## 3. Quote Blocks

### Standard Quote (`QUOTE`)
**UI Sections**: Layout, Background, Avatar

```json
{
  "type": "QUOTE",
  "content": {
    "text": "<p>The only way to do great work is to love what you do.</p>",
    "author": "Steve Jobs"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "regular",
      "verticalSpacingTop": 30,
      "verticalSpacingBottom": 30,
      "verticalSpacingLinked": true,
      "quotePadding": "medium"        // Options: small, medium, large
    },
    "background": {
      "style": "Gray"
    },
    "avatar": {
      "show": true,
      "fileId": "avatar-01",
      "size": "small"                 // Options: small, medium, large
    }
  }
}
```

### Quote on Image (`QUOTE_WITH_IMAGE`)
**UI Sections**: Layout, Background, Overlay, Avatar

```json
{
  "type": "QUOTE_WITH_IMAGE",
  "content": {
    "quoteText": "<p>In the middle of every difficulty lies opportunity.</p>",
    "author": "Albert Einstein"
  },
  "settings": {
    "layout": {
      "contentArea": "large",
      "sectionSpacing": "none",
      "verticalSpacingTop": 0,
      "verticalSpacingBottom": 0,
      "verticalSpacingLinked": true,
      "quotePadding": "large",
      "alignment": "center"           // Options: left, center, right
    },
    "background": {
      "style": "Image",
      "image": {
        "fileId": "scenic-bg-02",
        "url": "https://images.unsplash.com/..."
      }
    },
    "overlay": {
      "type": "dark",                 // Options: none, dark, light
      "opacity": 40
    },
    "showQuotationIcon": true,
    "avatar": {
      "show": true,
      "fileId": "avatar-einstein",
      "size": "medium"
    }
  }
}
```

### Quote Carousel (`QUOTE_CAROUSEL`)
**UI Sections**: Layout, Background, Carousel Settings

```json
{
  "type": "QUOTE_CAROUSEL",
  "content": {
    "quotes": [
      {
        "text": "<p>Innovation distinguishes between a leader and a follower.</p>",
        "author": "Steve Jobs",
        "image": { "fileId": "avatar-01" }
      },
      {
        "text": "<p>Stay hungry, stay foolish.</p>",
        "author": "Steve Jobs",
        "image": { "fileId": "avatar-01" }
      }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "regular",
      "verticalSpacingTop": 30,
      "verticalSpacingBottom": 30,
      "verticalSpacingLinked": true,
      "showBorder": true,
      "quotePadding": "medium"
    },
    "background": {
      "style": "Light"
    },
    "carousel": {
      "showAvatars": true,
      "avatarSize": "small"
    }
  }
}
```

---

---

## 4. Media Blocks

### Image Carousel (`IMAGE_CAROUSEL`)
**UI Sections**: Layout, Shape, Interaction, Background

```json
{
  "type": "IMAGE_CAROUSEL",
  "content": {
    "items": [
      {
        "fileId": "image-01",
        "altText": "Product showcase",
        "caption": "Version 1.0"
      },
      {
        "fileId": "image-02",
        "altText": "Team photo",
        "caption": "Our office"
      }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "shape": {
      "borderRadius": 0                // Range 0-50
    },
    "interaction": {
      "openImageOnClick": true        // Enable zoom on click
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Two Column Grid (`TWO_COLUMN_IMAGE`)
**UI Sections**: Layout, Shape, Interaction, Background

```json
{
  "type": "TWO_COLUMN_IMAGE",
  "content": {
    "items": [
      {
        "fileId": "image-col-1",
        "altText": "Left feature",
        "caption": "Our Design"
      },
      {
        "fileId": "image-col-2",
        "altText": "Right feature",
        "caption": "Our Technology"
      }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "shape": {
      "borderRadius": 0
    },
    "interaction": {
      "openImageOnClick": true
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Three Column Grid (`THREE_COLUMN_IMAGE`)
**UI Sections**: Layout, Shape, Interaction, Background

```json
{
  "type": "THREE_COLUMN_IMAGE",
  "content": {
    "items": [
      { "fileId": "img-1", "altText": "Feature 1" },
      { "fileId": "img-2", "altText": "Feature 2" },
      { "fileId": "img-3", "altText": "Feature 3" }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "shape": {
      "borderRadius": 0
    },
    "interaction": {
      "openImageOnClick": true
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Four Column Grid (`IMAGE_GRID`)
**UI Sections**: Layout, Shape, Interaction, Background

```json
{
  "type": "IMAGE_GRID",
  "content": {
    "items": [
      { "fileId": "img-1", "altText": "Item 1" },
      { "fileId": "img-2", "altText": "Item 2" },
      { "fileId": "img-3", "altText": "Item 3" },
      { "fileId": "img-4", "altText": "Item 4" }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "shape": {
      "borderRadius": 0
    },
    "interaction": {
      "openImageOnClick": true
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Audio Player (`AUDIO_PLAYER`)
**UI Sections**: Layout, Playback, Background

```json
{
  "type": "AUDIO_PLAYER",
  "content": {
    "fileId": "audio-01",
    "title": "Module Overview",
    "caption": "Listen to the introduction of this module."
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",    // Options: narrow, regular, wide
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "playback": {
      "forwardSeeking": true        // Enable/disable forward scrubbing
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Video (`VIDEO`)
**UI Sections**: Layout, Media Layout, Shape, Playback, Background

```json
{
  "type": "VIDEO",
  "content": {
    "title": "Introduction to AI",
    "description": "A brief overview of artificial intelligence.",
    "fileId": "video-01"
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "videoSize": "regular"        // Options: compact, regular, large
    },
    "shape": {
      "borderRadius": 0             // Range: 0-50
    },
    "playback": {
      "forwardSeeking": true        // Enable/disable forward scrubbing
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Embed (`EMBED`)
**UI Sections**: Layout, Media Layout, Border, Background

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
      "sectionSpacing": "narrow",    // Options: narrow, regular, wide
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "embedSize": "regular"        // Options: compact, regular, large
    },
    "border": {
      "showBorder": true            // Toggle border visibility
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Table (`TABLE`)
**UI Sections**: Layout, Table Layout, Background

```json
{
  "type": "TABLE",
  "content": {
    "rows": [
      {
        "cells": [
          { "text": "Header 1", "isHeader": true },
          { "text": "Header 2", "isHeader": true }
        ]
      },
      {
        "cells": [
          { "text": "Data 1", "isHeader": false },
          { "text": "Data 2", "isHeader": false }
        ]
      }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "tableLayout": {
      "cellSpacing": "narrow"
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

> [!NOTE]
> This document is updated to match the **Layout/Background nesting** schema.


### Subheading Only (`SUBHEADING_ONLY`)
**Scenario**: Minimalist Black background with high vertical spacing.*

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