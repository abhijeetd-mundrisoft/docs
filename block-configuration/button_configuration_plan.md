# Button Settings Requirement Analysis

This document outlines the requirement analysis and implementation strategy for adding configurable **Layout** (`sectionSpacing`, `verticalSpacing`, `buttonPlacement`, `buttonSize`, `buttonSizeValue`), **Content Display** (`showDescriptionText`), **Style** (`buttonColor`), and **Shape** (`roundedCorners`) settings for the Button block.

## Proposed Changes

### 1. Configuration Model Changes

#### Schema Updates
* **Layout**: Standard `sectionSpacing` (`narrow`, `regular`, `wide`), `verticalSpacing` (`0-150`), `buttonPlacement` (`left`, `right`), `buttonSize` (`compact`, `regular`, `large`), and `buttonSizeValue` (numeric width, e.g. 170).
* **Content Display**: Standard `showDescriptionText` (supports boolean toggle or string `"on"` / `"off"`).
* **Style**: Standard `buttonColor` object (supports both predefined theme tokens and custom HEX values via `type`/`value` nesting).
* **Shape**: Standard `roundedCorners` (radius slider from `0-50`).

#### Settings JSON Structure
```json
{
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

### 2. Backend / Domain Logic

#### Reusable Abstraction Opportunities
* **Shared Spacing & Shapes**: Spacing presets, numeric ranges, border-radius, and hexadecimal color validators are generic and fully utilized.
* **Factory Fallbacks**: We will add `ACTION_BUTTON` to the `layoutBlocks` constant inside `BlockContentFactory` to auto-resolve vertical spacing constraints. We will also initialize the block explicitly with fully loaded layout, contentDisplay, style, and shape defaults.
* **Schema Definition**: We will register the `ACTION_BUTTON` block inside `BlockSchemaUtil.java`.
* **Block Definition**: Mapped `BUTTON` as an alias inside `Block.parseBlockType()`.

### 3. Rendering and Styling Integration

* **Layout Integration**: Margins, vertical spacings, and alignment placements (`left`, `right`) apply layout calculation classes consistently. Predefined sizes and customizable widths configure CSS attributes.
* **Content Display Integration**: Renderer checks `contentDisplay.showDescriptionText` to conditionally render description blocks while retaining layout aesthetics without extra gaps.
* **Style Integration**: Custom color pickers apply hexadecimal values or map predefined theme color tokens onto the button element.
* **Shape Integration**: Border radius values from `0-50` map onto the CSS `border-radius` property on the button.

### 4. Validation Requirements

* **Placement Validation**: Ensure `buttonPlacement` is strictly `"left"` or `"right"`.
* **Sizing Validation**: Ensure `buttonSize` is strictly `"compact"`, `"regular"`, or `"large"`.
* **Sizing Value Validation**: Ensure custom numeric width `buttonSizeValue` is a valid positive number within `[0-1000]`.
* **Description Validation**: Ensure `showDescriptionText` is a boolean or maps to `"on"` / `"off"`.
* **Color Validation**: Check if nested HEX values match hexadecimal syntax or represent valid theme variables.
* **Shape Validation**: Verify that the rounded corners value falls within `[0-50]`.
* **Fallback Behavior**: Malformed JSON results in `IllegalArgumentException`. Missing keys correctly trigger default fallback mechanisms inside `BlockContentFactory` and the frontend.

### 5. Extensibility Considerations

* **Layout Systems**: Future support for centring, justification/full-width alignment, or breakpoint-specific sizing overrides.
* **Advanced Styling**: Hover, focus, active states, gradients, or dark/light mode contrast overrides.

### 6. Migration / Compatibility Considerations

* **Impact on Existing Blocks**: Legacy `ACTION_BUTTON` blocks lacking these keys will safely fallback to standard default aesthetics, ensuring backward compatibility.
