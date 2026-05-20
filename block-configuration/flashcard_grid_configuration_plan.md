# Flashcard Grid Settings Requirement Analysis

This document outlines the requirement analysis and implementation strategy for adding configurable **Layout** (`sectionSpacing`, `verticalSpacing`, `gridSize`) and **Hint** (`interactiveCardHint`) settings for the Flashcard Grid block.

## Proposed Changes

### 1. Configuration Model Changes

#### Schema Updates
* **Layout**: Introduce `gridSize` (Dropdown: `auto`, `compact`, `regular`, `large`) into the existing layout settings schema alongside `sectionSpacing` and `verticalSpacing`.
* **Hint**: Introduce a new `hint` object group with `interactiveCardHint` (boolean).

#### Settings JSON Structure
```json
{
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25,
      "gridSize": "auto"
    },
    "hint": {
      "interactiveCardHint": true
    }
  }
}
```

### 2. Backend / Domain Logic

#### Reusable Abstraction Opportunities
* **Shared Validators**: We will update `BlockContentValidator.validateLayoutSettings()` to explicitly support and restrict `gridSize`. We will also add a generic `validateHintSettings()` method to cleanly handle `interactiveCardHint` and future interactive instructions.
* **Factory Fallbacks**: We will add `FLASH_CARDS_GRID` to the `layoutBlocks` constant inside `BlockContentFactory` to auto-resolve vertical spacing constraints. We will also initialize the block explicitly with `gridSize = auto` and `hint = { interactiveCardHint: true }`.
* **Schema Definition**: We will register the `FLASH_CARDS_GRID` block inside `BlockSchemaUtil.java`.
* **Block Definition**: Mapped `FLASHCARD_GRID` as an alias inside `Block.parseBlockType()`.

### 3. Rendering and Interaction Integration

* **Layout Integration**: `sectionSpacing` and `verticalSpacing` behave identically to other blocks. The `gridSize` token will be fed to the CSS Grid abstraction layer (e.g., `grid-cols-compact` vs `grid-cols-auto`) determining how many cards fit securely on a row.
* **Hint Integration**: The renderer checks `hint.interactiveCardHint` to decide whether to mount the localized "Click to flip" instructional UI element overlaid or placed beside the card elements.

### 4. Validation Requirements

* **Grid Size Validation**: Reject any `gridSize` values not strictly matching `["auto", "compact", "regular", "large"]`.
* **Hint Validation**: Enforce that `interactiveCardHint` is strictly a `Boolean` primitive.
* **Fallback Behavior**: Malformed JSON results in `IllegalArgumentException`. Missing keys correctly trigger default fallback mechanisms inside `BlockContentFactory` and the frontend.

### 5. Extensibility Considerations

* **Layout Systems**: Placing `gridSize` in the shared layout block allows us to reuse this same configuration for `GALLERY` grids, `TEAM` grids, and `PRICING_TABLE` grids moving forward.
* **Interaction Guidance**: The new `hint` object can be expanded to include `swipeHint` for mobile sliders or `audioHint` for accessibility triggers.

### 6. Migration / Compatibility Considerations

* **Impact on Existing Blocks**: Legacy `FLASH_CARDS_GRID` objects lack the `hint` and `layout.gridSize` keys. Our factory and frontend normalization pipelines will safely infer `auto` and `true` respectively, retaining parity with current platform behavior.
