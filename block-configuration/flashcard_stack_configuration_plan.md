# Flashcard Stack Settings Requirement Analysis

This document outlines the requirement analysis and implementation strategy for adding configurable **Layout** (`sectionSpacing`, `verticalSpacing`) and **Hint** (`interactiveCardHint`) settings for the Flashcard Stack block.

## Proposed Changes

### 1. Configuration Model Changes

#### Schema Updates
* **Layout**: Standard `sectionSpacing` (`narrow`, `regular`, `wide`) and `verticalSpacing` (`0-150`).
* **Hint**: Standard `hint` group containing `interactiveCardHint` (boolean).

#### Settings JSON Structure
```json
{
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacing": 25
    },
    "hint": {
      "interactiveCardHint": true
    }
  }
}
```

### 2. Backend / Domain Logic

#### Reusable Abstraction Opportunities
* **Shared Validators**: The generic layout and hint validators will automatically validate these fields once the alias is mapped.
* **Factory Fallbacks**: We will add `FLASH_CARDS_STACK` to the `layoutBlocks` constant inside `BlockContentFactory` to auto-resolve vertical spacing constraints. We will also initialize the block explicitly with `hint = { interactiveCardHint: true }`.
* **Schema Definition**: We will register the `FLASH_CARDS_STACK` block inside `BlockSchemaUtil.java`.
* **Block Definition**: Mapped `FLASHCARD_STACK` as an alias inside `Block.parseBlockType()`.

### 3. Rendering and Interaction Integration

* **Layout Integration**: `sectionSpacing` and `verticalSpacing` behave identically to other blocks. The card stack containers will adjust padding/margin dynamically based on these presets.
* **Hint Integration**: The renderer checks `hint.interactiveCardHint` to decide whether to mount the localized "Click to flip card" instructional guidance on the deck stack element.

### 4. Validation Requirements

* **Hint Validation**: Enforce that `interactiveCardHint` is strictly a `Boolean` primitive.
* **Fallback Behavior**: Malformed JSON results in `IllegalArgumentException`. Missing keys correctly trigger default fallback mechanisms inside `BlockContentFactory` and the frontend.

### 5. Extensibility Considerations

* **Layout Systems**: Using the generic layout block allows us to easily support unlinked top/bottom spacing, responsive spacing, or breakpoint-specific overrides in the future.
* **Interaction Guidance**: The new `hint` object can be expanded to include other mobile-friendly swipe triggers or accessibility system descriptors.

### 6. Migration / Compatibility Considerations

* **Impact on Existing Blocks**: Legacy `FLASH_CARDS_STACK` objects lack the `hint` and `layout` keys. Our factory and frontend normalization pipelines will safely infer defaults, retaining parity with current platform behavior.
