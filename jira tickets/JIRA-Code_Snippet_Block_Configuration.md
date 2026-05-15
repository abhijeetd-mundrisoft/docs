### Description

Extend the `CODE_EXAMPLE` (Code Snippet) block functionality to support configurable layout and styling settings. This includes section spacing, vertical spacing, and customizable code background color, ensuring consistency with the reusable block configuration architecture.

### Functional Changes Made

* **Validation Integration**: Updated `BlockContentValidator` to support `CODE_EXAMPLE` block settings validation. Added a new `validateStyleSettings` method to handle the `codeBackgroundColor` property.
* **Factory Defaults**: Updated `BlockContentFactory` to provide default layout and style settings for new `CODE_EXAMPLE` blocks, ensuring backward compatibility and a consistent initial state.
* **Schema Association**: Defined the `settings` structure for `CODE_EXAMPLE` in `BlockSchemaUtil`, allowing AI-driven generation and backend validation to recognize the new configuration options.
* **Documentation**: Updated `block_json_structures.md` with a sample JSON for the `CODE_EXAMPLE` block including its new settings.

### Sample JSON

```json
{
  "type": "CODE_EXAMPLE",
  "content": {
    "code": "public class HelloWorld {\n    public static void main(String[] args) {\n        System.out.println(\"Hello, World!\");\n    }\n}",
    "caption": "Java Hello World Example"
  },
  "settings": {
    "layout": {
      "sectionSpacing": "narrow",
      "verticalSpacingTop": 25,
      "verticalSpacingBottom": 25,
      "verticalSpacingLinked": true
    },
    "style": {
      "codeBackgroundColor": "#FAFAFA"
    }
  }
}
```

### Worklog Entries

* Added layout and style configuration support for CODE_EXAMPLE blocks.
* Implemented `codeBackgroundColor` validation in `BlockContentValidator`.
* Updated `BlockContentFactory` to include `CODE_EXAMPLE` in shared layout and style default initialization.
* Extended `BlockSchemaUtil` with `settings` definition for Code Snippet blocks.
* Added `CODE_EXAMPLE` JSON sample to `docs\block-configuration\block_json_structures.md`.
* Created a new JIRA ticket document for tracking (CC-515).
* Verified validation logic for `sectionSpacing` and `codeBackgroundColor` HEX/token values.
