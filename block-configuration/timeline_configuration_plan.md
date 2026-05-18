# Timeline BlockType Settings Requirement Analysis

This document provides a detailed requirement analysis and implementation approach for adding configurable **Layout** (`sectionSpacing`, `verticalSpacing`), **Heading** (`headingStyle`), and **Interaction** (`openImageOnClick`) settings for the Timeline block.

## Proposed Changes

### 1. Configuration Model Changes

#### Schema Updates
* **Layout**: Keep existing standard `sectionSpacing` (`narrow`, `regular`, `wide`) and `verticalSpacing` (`0-150`).
* **Heading**: We will extend the heading style configurations to support paired hierarchies: `H2_H3`, `H3_H4`, `H4_H5`.
* **Interaction**: Rely on the existing `openImageOnClick` boolean interaction property.

#### Settings JSON Structure
```json
{
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

### 2. Backend / Domain Logic

#### Reusable Abstraction Opportunities
* **Shared Validators**: We will extend `BlockContentValidator.validateHeadingSettings()` to support the new paired enum values (`H2_H3`, `H3_H4`, `H4_H5`). `validateLayoutSettings()` and `validateInteractionSettings()` already natively support our spacing and boolean flags.
* **Factory Fallbacks**: We will centralize the default values inside `BlockContentFactory`. We will add `TIMELINE_VIEW` to the `layoutBlocks` constant for automatic spacing overrides, and explicitly initialize `headingStyle = H2_H3` and `openImageOnClick = true`.
* **Schema Definition**: We will register the `TIMELINE_VIEW` block inside `BlockSchemaUtil.java` mapping its UI settings structure.
* **Block Definition**: Mapped `TIMELINE` as an alias inside `Block.parseBlockType()` to map safely to `TIMELINE_VIEW`.

### 3. Rendering and Interaction Integration

* **Layout Integration**: Frontend consumes `sectionSpacing` and `verticalSpacing` as standard layout padding/margin tokens.
* **Typography Interpretation**: The renderer will split the `H2_H3` string. The first element (`H2`) will map to the `<TimelineTitle>` tag. The second element (`H3`) will cascade down to map into the `<TimelineEventHeader>` tags recursively.
* **Interaction**: The media rendering pipeline inside the Timeline nodes will conditionally wrap image components in a Lightbox trigger based on `interaction.openImageOnClick`.

### 4. Validation Requirements

* **Heading Pair Validation**: Update string validation check logic to permit `["H2_H3", "H3_H4", "H4_H5"]`.
* **Invalid Configuration Fallback Behavior**: Missing configurations safely fallback via Factory defaults. Invalid enums correctly throw `IllegalArgumentException`.

### 5. Extensibility Considerations

* **Media Interaction**: Decoupling the `interaction` block paves the way for `openVideoModal` or `enableKeyboardNavigation` inside interactive components.
* **Advanced Typography Mapping**: Paired heading combinations allow us to maintain accessible, sequentially logical document outlines without requiring the user to manually configure every single node level in the tree.

### 6. Migration / Compatibility Considerations

* **Impact on Existing Blocks**: Legacy blocks lack this configuration. The frontend and factory will safely normalize this missing data and automatically apply `H2_H3`, `openImageOnClick=true`, and `narrow/25` spacing. No migration scripts are needed.
