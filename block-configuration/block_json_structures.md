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
      "borderRadius": 12,
      "headingSpacing": 20
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
      "verticalSpacingLinked": false,
      "headingSpacing": 0
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
      "verticalSpacingLinked": true,
      "headingSpacing": 15
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Two Column Text (`TWO_COLUMN_TEXT`)
**UI Sections**: Layout, Background

```json
{
  "type": "TWO_COLUMN_TEXT",
  "content": {
    "column1": "<p>This is left column content.</p>",
    "column2": "<p>This is right column content.</p>"
  },
  "settings": {
    "layout": {
      "sectionSpacing": {
        "preset": "narrow"
      },
      "verticalSpacing": {
        "value": 25,
        "linked": true
      }
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Three Column Text (`THREE_COLUMN_TEXT`)
**UI Sections**: Layout, Background

```json
{
  "type": "THREE_COLUMN_TEXT",
  "content": {
    "columns": [
      {
        "title": "Column 1",
        "body": "<p>First column content</p>"
      },
      {
        "title": "Column 2",
        "body": "<p>Second column content</p>"
      },
      {
        "title": "Column 3",
        "body": "<p>Third column content</p>"
      }
    ]
  },
  "settings": {
    "layout": {
      "sectionSpacing": {
        "preset": "narrow"
      },
      "verticalSpacing": {
        "value": 25
      }
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

### Large Text Quote (`QUOTE_LARGE_TEXT`)
**UI Sections**: Layout, Behavior, Typography, Background

```json
{
  "type": "QUOTE_LARGE_TEXT",
  "content": {
    "text": "The future belongs to those who believe in the beauty of their dreams.",
    "author": "Eleanor Roosevelt"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",         // Options: compact, regular, large
      "sectionSpacing": "narrow",      // Options: narrow, regular, wide
      "verticalSpacingTop": 25,        // Range: 0-150
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true,
      "quoteSpacing": 20               // Range: 0-100
    },
    "behavior": {
      "showAuthor": true               // Toggle author visibility
    },
    "typography": {
      "authorSize": "regular"          // Options: compact, regular, large
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Boxed Quote (`QUOTE_BOXED`)
**UI Sections**: Layout, Behavior, Typography, Background

```json
{
  "type": "QUOTE_BOXED",
  "content": {
    "text": "Success is not final, failure is not fatal: it is the courage to continue that counts.",
    "author": "Winston Churchill"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true,
      "quoteSpacing": 20
    },
    "behavior": {
      "showAuthor": true,
      "showIcon": true,                 // Toggle quote icon visibility (New)
      "iconColor": "#666666"            // Color of the quote icon (New)
    },
    "typography": {
      "authorSize": "regular"
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Bulleted List (`BULLETED_LIST`)
**UI Sections**: Layout, Typography, Background

```json
{
  "type": "BULLETED_LIST",
  "content": {
    "items": [
      { "text": "First bullet item", "orderIndex": 0 },
      { "text": "Second bullet item", "orderIndex": 1 }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true,
      "indent": 20                   // Range: 0-100
    },
    "typography": {
      "bulletColor": "#000000"       // Bullet color
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Numbered List (`NUMBERED_LIST`)
**UI Sections**: Layout, Typography, Background

```json
{
  "type": "NUMBERED_LIST",
  "content": {
    "items": [
      { "text": "First numbered item", "orderIndex": 0 },
      { "text": "Second numbered item", "orderIndex": 1 }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true,
      "indent": 20                   // Range: 0-100
    },
    "typography": {
      "bulletColor": "#000000"       // Number color
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Checkbox List (`CHECKBOX_LIST`)
**UI Sections**: Layout, Typography, Background

```json
{
  "type": "CHECKBOX_LIST",
  "content": {
    "items": [
      { "text": "First checkbox item", "checked": false, "orderIndex": 0 },
      { "text": "Second checkbox item", "checked": true, "orderIndex": 1 }
    ]
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true,
      "indent": 20                   // Range: 0-100
    },
    "typography": {
      "bulletColor": "#000000"       // Checkbox color
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Accordion (`ACCORDION`)

```json
{
  "id": "accordion-001",
  "type": "ACCORDION",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "heading": {
      "headingStyle": "H2"
    },
    "behavior": {
      "expansionMode": "single_open"
    }
  },
  "content": [
    {
      "title": "What is Artificial Intelligence?",
      "content": "Artificial Intelligence refers to systems capable of performing tasks that normally require human intelligence.",
      "orderIndex": 1,
      "iconStyle": "Arrow",
      "iconPosition": "Right"
    },
    {
      "title": "What is Machine Learning?",
      "content": "Machine Learning is a subset of AI focused on learning patterns from data.",
      "orderIndex": 2,
      "iconStyle": "Arrow",
      "iconPosition": "Right"
    }
  ]
}
```

### Scenario (`SCENARIO`)

```json
{
  "id": "scenario-block-001",
  "type": "SCENARIO",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "heading": {
      "headingStyle": "H2"
    }
  },
  "content": {
    "title": "Customer Support Escalation",
    "description": "A customer reports an issue with delayed shipment and requests escalation.",
    "scenarios": [
      {
        "heading": "Initial Response",
        "text": "Support agent acknowledges the issue and gathers order details."
      },
      {
        "heading": "Escalation Handling",
        "text": "Issue is escalated to logistics for expedited investigation."
      }
    ]
  }
}
```

### Tabs (`TAB`)

```json
{
  "id": "tabs-001",
  "type": "TAB",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "interaction": {
      "openImageOnClick": true
    }
  },
  "content": [
    {
      "title": "Overview",
      "content": "<p>Tab content here</p>",
      "orderIndex": 1,
      "images": [
        {
          "url": "https://example.com/image1.png",
          "alt": "Overview image"
        }
      ]
    }
  ]
}
```

### Timeline (`TIMELINE`)

```json
{
  "id": "timeline-001",
  "type": "TIMELINE",
  "content": {
    "title": "History of Web Development",
    "events": [
      {
        "title": "HTML Introduced",
        "description": "The first version of HTML was introduced.",
        "image": "html-history.png"
      },
      {
        "title": "CSS Adoption",
        "description": "CSS became standard for styling websites.",
        "image": "css-history.png"
      }
    ]
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "heading": {
      "headingStyle": "H2_H3"
    },
    "interaction": {
      "openImageOnClick": true
    }
  }
}
```

### Flashcard Grid (`FLASHCARD_GRID`)

```json
{
  "id": "flashcard-grid-001",
  "type": "FLASHCARD_GRID",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25,
      "gridSize": "auto"
    },
    "hint": {
      "interactiveCardHint": true
    }
  },
  "content": {
    "cards": [
      {
        "front": "What is Java?",
        "back": "A programming language."
      },
      {
        "front": "What is JVM?",
        "back": "Java Virtual Machine."
      }
    ]
  }
}
```

### Flashcard Stack (`FLASHCARD_STACK`)

```json
{
  "id": "flashcard-stack-001",
  "type": "FLASHCARD_STACK",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "hint": {
      "interactiveCardHint": true
    }
  },
  "content": {
    "cards": [
      {
        "front": "What is encapsulation?",
        "back": "Wrapping data and methods together into a single unit."
      },
      {
        "front": "What is inheritance?",
        "back": "A mechanism where one class acquires properties of another class."
      }
    ]
  }
}
```

### Button (`BUTTON`)

```json
{
  "type": "BUTTON",
  "content": {
    "label": "Start Course",
    "description": "Click to begin the learning module.",
    "url": "https://example.com"
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25,
      "buttonPlacement": "right",
      "buttonSize": "compact",
      "buttonSizeValue": 170
    },
    "contentDisplay": {
      "showDescriptionText": true
    },
    "style": {
      "buttonColor": {
        "type": "hex",
        "value": "#FF631E"
      }
    },
    "shape": {
      "roundedCorners": 20
    }
  }
}
```

### Button Stack (`BUTTON_STACK`)

```json
{
  "type": "BUTTON_STACK",
  "content": {
    "buttons": [
      {
        "label": "Start Course",
        "description": "Begin the learning module.",
        "url": "https://example.com/start"
      },
      {
        "label": "View Resources",
        "description": "Open supporting materials.",
        "url": "https://example.com/resources"
      }
    ]
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25,
      "buttonPlacement": "right",
      "buttonSize": "compact",
      "buttonSizeValue": 170,
      "buttonSpacing": "narrow",
      "spacingValue": 25
    },
    "contentDisplay": {
      "showDescriptionText": true
    },
    "style": {
      "buttonColor": {
        "type": "hex",
        "value": "#FF631E"
      }
    },
    "shape": {
      "roundedCorners": 20
    }
  }
}
```

### Labeled Graphic (`INTERACTIVE_IMAGE`)

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

### Process (`STEP_FLOW`)

```json
{
  "id": "process-block-001",
  "type": "STEP_FLOW",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "heading": {
      "headingStyle": "H2"
    },
    "label": {
      "showStepLabel": true,
      "stepLabelText": "Step"
    },
    "interaction": {
      "openImageOnClick": true
    }
  },
  "content": [
    {
      "type": "intro",
      "title": "Introduction",
      "description": "<p>Overview of the process</p>",
      "isHidden": false,
      "orderIndex": 1
    },
    {
      "type": "step",
      "title": "Prepare Ingredients",
      "description": "<p>Collect all required materials before starting.</p>",
      "isHidden": false,
      "orderIndex": 2,
      "image": {
        "url": "https://example.com/process-step-1.png",
        "alt": "Ingredients preparation"
      }
    }
  ]
}
```

---

## 4. Media Blocks

### Banner Image (`BANNER_IMAGE`)
**UI Sections**: Layout, Background

```json
{
  "type": "BANNER_IMAGE",
  "content": {
    "image": {
      "altText": "A scenic mountain view",
      "fileId": "banner-uuid-1234"
    },
    "caption": "Figure 1: Mountain landscape"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Image Block (`IMAGE`)
**UI Sections**: Layout, Shape, Interaction, Background

```json
{
  "type": "IMAGE",
  "content": {
    "title": "Innovative Solution Diagram",
    "description": "System architecture visualization showing components interaction.",
    "fileId": "image-uuid-12345",
    "altText": "Architecture Diagram"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",       // Options: compact, regular, large
      "sectionSpacing": "narrow",     // Options: narrow, regular, wide
      "verticalSpacingTop": 25,       // Range: 0-120
      "verticalSpacingBottom": 25,    // Range: 0-120
      "verticalSpacingLinked": true
    },
    "shape": {
      "roundedCorners": 0             // Range: 0-30
    },
    "interaction": {
      "openImageOnClick": true        // Toggle to allow zoom modal preview
    },
    "background": {
      "style": "Light"                // Options: Light, Gray, Theme, Dark, Image, Custom
    }
  }
}
```

---

### Image Text Right Block (`IMAGE_TEXT_RIGHT`)
**UI Sections**: Layout, Media Layout, Shape, Interaction, Background

```json
{
  "type": "IMAGE_TEXT_RIGHT",
  "content": {
    "image": {
      "title": "Diagram Title",
      "fileId": "image-uuid-12345"
    },
    "text": "<p>Enter text here</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",       // Options: compact, regular, large
      "sectionSpacing": "narrow",     // Options: narrow, regular, wide
      "verticalSpacingTop": 25,       // Range: 0-120
      "verticalSpacingBottom": 25,    // Range: 0-120
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "imageSize": "regular",         // Options: compact, regular, large
      "imagePosition": "right"        // Options: left, right (default: right)
    },
    "shape": {
      "roundedCorners": 0             // Range: 0-30
    },
    "interaction": {
      "openImageOnClick": true        // Toggle to allow zoom modal preview
    },
    "background": {
      "style": "Light"                // Options: Light, Gray, Theme, Dark, Image, Custom
    }
  }
}
```

### Text Image Left Block (`TEXT_IMAGE_LEFT`)
**UI Sections**: Layout, Media Layout, Shape, Interaction, Background

```json
{
  "type": "TEXT_IMAGE_LEFT",
  "content": {
    "text": "<p>Enter text here</p>",
    "image": {
      "title": "Diagram Title",
      "fileId": "image-uuid-12345"
    }
  },
  "settings": {
    "layout": {
      "contentArea": "regular",       // Options: compact, regular, large
      "sectionSpacing": "narrow",     // Options: narrow, regular, wide
      "verticalSpacingTop": 25,       // Range: 0-120
      "verticalSpacingBottom": 25,    // Range: 0-120
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "imageSize": "regular",         // Options: compact, regular, large
      "imagePosition": "left"         // Options: left, right (default: left)
    },
    "shape": {
      "roundedCorners": 0             // Range: 0-30
    },
    "interaction": {
      "openImageOnClick": true        // Toggle to allow zoom modal preview
    },
    "background": {
      "style": "Light"                // Options: Light, Gray, Theme, Dark, Image, Custom
    }
  }
}
```

---

### Image Text Bottom Block (`IMAGE_TEXT_BOTTOM`)
**UI Sections**: Layout, Media Layout, Shape, Interaction, Background

```json
{
  "type": "IMAGE_TEXT_BOTTOM",
  "content": {
    "image": {
      "title": "Diagram Title",
      "fileId": "image-uuid-12345"
    },
    "text": "<p>Enter text here</p>"
  },
  "settings": {
    "layout": {
      "contentArea": "regular",       // Options: compact, regular, large
      "sectionSpacing": "narrow",     // Options: narrow, regular, wide
      "verticalSpacingTop": 25,       // Range: 0-120
      "verticalSpacingBottom": 25,    // Range: 0-120
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "imageSize": "regular",         // Options: compact, regular, large
      "imagePosition": "bottom"       // Options: top, bottom (default: bottom)
    },
    "shape": {
      "roundedCorners": 0             // Range: 0-30
    },
    "interaction": {
      "openImageOnClick": true        // Toggle to allow zoom modal preview
    },
    "background": {
      "style": "Light"                // Options: Light, Gray, Theme, Dark, Image, Custom
    }
  }
}
```

### Text Image Bottom Block (`TEXT_IMAGE_BOTTOM`)
**UI Sections**: Layout, Media Layout, Shape, Interaction, Background

```json
{
  "type": "TEXT_IMAGE_BOTTOM",
  "content": {
    "text": "<p>Enter text here</p>",
    "image": {
      "title": "Diagram Title",
      "fileId": "image-uuid-12345"
    }
  },
  "settings": {
    "layout": {
      "contentArea": "regular",       // Options: compact, regular, large
      "sectionSpacing": "narrow",     // Options: narrow, regular, wide
      "verticalSpacingTop": 25,       // Range: 0-120
      "verticalSpacingBottom": 25,    // Range: 0-120
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "imageSize": "regular",         // Options: compact, regular, large
      "imagePosition": "bottom"       // Options: top, bottom (default: bottom)
    },
    "shape": {
      "roundedCorners": 0             // Range: 0-30
    },
    "interaction": {
      "openImageOnClick": true        // Toggle to allow zoom modal preview
    },
    "background": {
      "style": "Light"                // Options: Light, Gray, Theme, Dark, Image, Custom
    }
  }
}
```

---

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

### Attachment (`RESOURCE_FILE`)
**UI Sections**: Layout, Background

```json
{
  "type": "RESOURCE_FILE",
  "content": {
    "fileId": "attachment-uuid-01",
    "displayName": "User_Manual.pdf",
    "description": "Comprehensive guide for all system features.",
    "allowDownload": true
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",    // Options: narrow, regular, wide
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
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
      "videoWidth": "regular"        // Options: compact, regular, large
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

### Video text bottom (`VIDEO_TEXT_BOTTOM`)
**UI Sections**: Layout, Media Layout, Shape, Playback, Background

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
      "sectionSpacing": "narrow",      // Options: narrow, regular, wide
      "verticalSpacingTop": 25,        // Range: 0-120
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "videoWidth": "regular",          // Options: compact, regular, large
      "videoPosition": "top"         // Options: left, right, top, bottom
    },
    "shape": {
      "borderRadius": 0                // Range: 0-30
    },
    "playback": {
      "forwardSeeking": true           // Toggle forward scrubbing
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Text video bottom (`TEXT_VIDEO_BOTTOM`)
**UI Sections**: Layout, Media Layout, Shape, Playback, Background

```json
{
  "type": "TEXT_VIDEO_BOTTOM",
  "content": {
    "text": "<p>This is the description text that appears above or below the video.</p>",
    "video": {
      "title": "Module Overview",
      "fileId": "video-uuid-001"
    }
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",      // Options: narrow, regular, wide
      "verticalSpacingTop": 25,        // Range: 0-120
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "videoWidth": "regular",          // Options: compact, regular, large
      "videoPosition": "bottom"         // Options: left, right, top, bottom
    },
    "shape": {
      "borderRadius": 0                // Range: 0-30
    },
    "playback": {
      "forwardSeeking": true           // Toggle forward scrubbing
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

## 5. Reference Blocks

### Code Snippet (`CODE_EXAMPLE`)
**UI Sections**: Layout, Background

```json
{
  "type": "CODE_EXAMPLE",
  "content": {
    "code": "function helloWorld() {\n  console.log('Hello, world!');\n}",
    "caption": "A simple JavaScript function."
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",    // Options: narrow, regular, wide
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
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

## 6. Layout Blocks

### Section Break (`SECTION_BREAK`)
**UI Sections**: Layout

```json
{
  "type": "SECTION_BREAK",
  "content": {
    "style": "solid"                // Options: solid, dashed
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",    // Options: narrow, regular, wide
      "verticalSpacing": 25          // Linked top/bottom spacing (0-150)
    }
  }
}
```

### Numbered Divider / Step Marker (`STEP_MARKER`)

```json id="v59oq5"
{
  "id": "step-marker-001",
  "type": "STEP_MARKER",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "numbering": {
      "restartSequence": true,
      "restartAt": 1
    }
  },
  "content": {}
}
```

### Spacer / Spacing (`SPACING`)

```json id="8l8j4o"
{
  "id": "spacer-block-001",
  "type": "SPACING",
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    }
  },
  "content": {
    "height": "medium"
  }
}
```

---

### Multiple Choice Knowledge Check (`MULTIPLE_CHOICE`)

```json id="5dly98"
{
  "id": "multiple-choice-001",
  "type": "MULTIPLE_CHOICE",
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "feedbackStyle": {
      "correctAnswerColor": "#FF631E",
      "incorrectAnswerColor": "#000000"
    }
  },
  "content": {
    "question": "What is the capital of France?",
    "options": [
      "Berlin",
      "Madrid",
      "Paris",
      "Rome"
    ],
    "correctAnswer": 2
  }
}
```

---

### Multiple Response Knowledge Check (`MULTIPLE_RESPONSE`)

```json id="j6a9pd"
{
  "id": "multiple-response-001",
  "type": "MULTIPLE_RESPONSE",
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "feedbackStyle": {
      "correctAnswerColor": "#FF631E",
      "incorrectAnswerColor": "#000000"
    }
  },
  "content": {
    "question": "Which of the following are programming languages?",
    "options": [
      "Java",
      "HTML",
      "Python",
      "CSS"
    ],
    "correctAnswers": [0, 2]
  }
}
```

### Fill in the Blank Knowledge Check (`FILL_IN_THE_BLANK`)

```json id="r1v2pz"
{
  "id": "fill-in-the-blank-001",
  "type": "FILL_IN_THE_BLANK",
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "feedbackStyle": {
      "correctAnswerColor": "#FF631E",
      "incorrectAnswerColor": "#000000"
    }
  },
  "content": {
    "question": "The capital of France is ____.",
    "answers": [
      "Paris"
    ]
  }
}
```

### Matching Knowledge Check (`MATCHING`)

```json id="m2t3ch"
{
  "id": "matching-001",
  "type": "MATCHING",
  "settings": {
    "layout": {
      "contentArea": "regular",
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "feedbackStyle": {
      "correctAnswerColor": "#FF631E",
      "incorrectAnswerColor": "#000000"
    }
  },
  "content": {
    "question": "Match the countries with their capitals.",
    "pairs": [
      {
        "left": "France",
        "right": "Paris"
      },
      {
        "left": "Japan",
        "right": "Tokyo"
      }
    ]
  }
}
```

---

### Subheading Only (`SUBHEADING_ONLY`)
**UI Sections**: Layout, Background

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
      "borderRadius": 8,
      "headingSpacing": 0
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

---

## 6. Chart Blocks

### Line Chart (`TREND_CHAT`)
**UI Sections**: Chart > Layout, Chart > Heading, Chart > Style

```json
{
  "type": "TREND_CHAT",
  "content": {
    "chartTitle": "Monthly Revenue Trends",
    "xAxisLabel": "Month",
    "yAxisLabel": "Revenue (USD)",
    "items": [
      { "title": "Jan", "value": 4500, "orderIndex": 1 },
      { "title": "Feb", "value": 5200, "orderIndex": 2 },
      { "title": "Mar", "value": 4800, "orderIndex": 3 },
      { "title": "Apr", "value": 6100, "orderIndex": 4 }
    ]
  },
  "settings": {
    "chart": {
      "layout": {
        "sectionSpacing": "narrow",    // Options: narrow, regular, wide
        "verticalSpacing": 25           // Range: 0-150
      },
      "heading": {
        "titleStyle": "h2"              // Options: h1, h2, h3, h4, h5
      },
      "style": {
        "chartColor": "#FF631E",        // HEX Color
        "lineStyle": "curved"           // Options: straight, curved
      }
    },
    "background": {
      "style": "Light"                  // Options: Light, Gray, Theme, Dark, Image, Custom
    }
  }
}

### Pie Chart (`DISTRIBUTION_CHAT`)
**UI Sections**: Chart > Layout, Chart > Heading, Chart > Data Display

```json
{
  "type": "DISTRIBUTION_CHAT",
  "content": {
    "chartTitle": "Market Share Distribution",
    "items": [
      { "title": "Product A", "value": 40, "orderIndex": 1 },
      { "title": "Product B", "value": 30, "orderIndex": 2 },
      { "title": "Product C", "value": 20, "orderIndex": 3 },
      { "title": "Others", "value": 10, "orderIndex": 4 }
    ]
  },
  "settings": {
    "chart": {
      "layout": {
        "sectionSpacing": "narrow",    // Options: narrow, regular, wide
        "verticalSpacing": 25           // Range: 0-150
      },
      "heading": {
        "titleStyle": "h2"              // Options: h1, h2, h3, h4, h5
      },
      "dataDisplay": {
        "showValuesAs": "percentage"    // Options: numeric, percentage
      }
    },
    "background": {
      "style": "Light"
    }
  }
}
```

### Bar Chart (`BAR_CHART`)
**UI Sections**: Chart > Layout, Chart > Heading, Chart > Style, Chart > Data Display

```json
{
  "type": "BAR_CHART",
  "content": {
    "chartTitle": "Quarterly Sales Visualizer",
    "xAxisLabel": "Quarter",
    "yAxisLabel": "Sales (USD)",
    "items": [
      { "title": "Q1", "value": 120.5, "orderIndex": 1 },
      { "title": "Q2", "value": 145.2, "orderIndex": 2 },
      { "title": "Q3", "value": 98.4, "orderIndex": 3 },
      { "title": "Q4", "value": 180.0, "orderIndex": 4 }
    ]
  },
  "settings": {
    "chart": {
      "layout": {
        "contentArea": "regular",       // Options: compact, regular, large
        "sectionSpacing": "narrow",     // Options: narrow, regular, wide
        "verticalSpacing": 25,          // Range: 0-120 (linked top/bottom)
        "blockPadding": 25              // Range: 0-120 (chart internal padding)
      },
      "heading": {
        "headingLevel": "H2",           // Options: H1, H2, H3, H4, H5
        "titleStyle": "default"         // Options: default, minimal, bold, accent
      },
      "style": {
        "chartColor": "#FF631E"         // HEX Color
      },
      "dataDisplay": {
        "showValuesAs": "numeric"       // Options: numeric, percentage
      }
    },
    "background": {
      "style": "Light"                  // Options: Light, Gray, Theme, Dark, Image, Custom
    }
  }
}
```

---

## 7. Layout & Flow Blocks

### Continue / Checkpoint (`CHECKPOINT`)
**UI Sections**: Layout, Button

```json
{
  "type": "CHECKPOINT",
  "content": {
    "buttonText": "Continue",
    "completionType": "ALL",           // Options: NONE, PREVIOUS, ALL
    "locked": true,
    "hintText": "Please complete all blocks above to continue."
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",      // Options: narrow, regular, wide
      "verticalSpacingTop": 25,        // Range: 0-150
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "button": {
      "buttonSize": "regular",         // Options: compact, regular, large
      "buttonSizeValue": 760,          // Range: 100-960
      "buttonColor": "#FF631E",        // HEX Color
      "roundedCorners": 0              // Range: 0-50
    }
  }
}
```

### Sorting Activity (`SORTING_ACTIVITY`)
**UI Sections**: Layout

```json
{
  "type": "VIDEO_TEXT_RIGHT",
  "content": {
    "text": "<p>This is the description text that appears to the right of the video in the desktop layout.</p>",
    "video": {
      "title": "Module Overview",
      "fileId": "video-uuid-001"
    }
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",      // Options: narrow, regular, wide
      "verticalSpacingTop": 25,        // Range: 0-120
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "videoWidth": "regular",          // Options: compact, regular, large
      "videoPosition": "right"         // Options: left, right
    },
    "shape": {
      "borderRadius": 0                // Range: 0-30
    },
    "playback": {
      "forwardSeeking": true           // Toggle forward scrubbing
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Video text Left (`TEXT_VIDEO_LEFT`)
**UI Sections**: Layout, Media Layout, Shape, Playback, Background

```json
{
  "type": "TEXT_VIDEO_LEFT",
  "content": {
    "text": "<p>This is the description text that appears to the left of the video in the desktop layout.</p>",
    "video": {
      "title": "Module Overview",
      "fileId": "video-uuid-001"
    }
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",      // Options: narrow, regular, wide
      "verticalSpacingTop": 25,        // Range: 0-120
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "mediaLayout": {
      "videoWidth": "regular",          // Options: compact, regular, large
      "videoPosition": "left"          // Options: left, right
    },
    "shape": {
      "borderRadius": 0                // Range: 0-30
    },
    "playback": {
      "forwardSeeking": true           // Toggle forward scrubbing
    },
    "background": {
      "style": "Light"
    }
  }
}
```

---

### Sorting Activity (`SORTING_ACTIVITY`)
**UI Sections**: Layout

```json
{
  "id": "sorting-activity-001",
  "type": "SORTING_ACTIVITY",
  "content": {
    "title": "Sort the Steps",
    "items": [
      "Analyze requirements",
      "Implement logic",
      "Write tests",
      "Deploy changes"
    ]
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",      // Options: narrow, regular, wide
      "verticalSpacing": 25            // Range: 0-150
    }
  }
}
```
```