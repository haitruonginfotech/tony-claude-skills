# 🔥 STRICT Page Header Processing Skill

If the target section is a Page Header, you MUST ABANDON default generative logic and strictly conform to the exact JSON mapping below. Deviating from these rules will break the environment.

## 1. 🏛️ The Main Container Name
- You MUST rename the top-level container's internal Elementor title to `Page header`. 
- **Requirement:** Add `"_title": "Page header"` inside the top-level container's `settings`.
- Do NOT just name it "Container".

## 2. 📏 Minimum Heights (MANDATORY EXACT MATCH)
You MUST set these exact min_heights within the container's `settings`. Do not override them with Figma layout heights. You must include the `sizes` empty array:
```json
"min_height": { "unit": "px", "size": 350, "sizes": [] },
"min_height_tablet": { "unit": "px", "size": 300, "sizes": [] },
"min_height_mobile": { "unit": "px", "size": 240, "sizes": [] }
```

## 3. 🎯 Alignment
Inside the main container settings, alignment must be controlled via flexbox settings ONLY:
- **Center Aligned (Default)**: `"flex_justify_content": "center"`, `"flex_align_items": "center"`
- **Left Aligned**: `"flex_justify_content": "flex-start"`, `"flex_align_items": "flex-start"`

## 4. 🧩 The Heading Widget (Dynamic Tag REQUIRED)
The main title MUST be a dynamic Elementor tag, not static text, structured exactly like this:
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {
    "title": "Add Your Heading Text Here",
    "header_size": "h1",
    // You may map extracted font family and sizes here, BUT MUST USE __dynamic__:
    "__dynamic__": {
      "title": "[elementor-tag id=\"5f50efd\" name=\"page-title\" settings=\"%7B%7D\"]"
    },
    // The color MUST use the global title color:
    "__globals__": {
      "title_color": "globals/colors?id=7c31574" 
    }
  },
  "elements": []
}
```

## 5. 🍞 Breadcrumbs vs Text Widget
If there is a subtext like "Home / Useful Links" below the heading, identify if it acts as breadcrumbs.

**Scenario A: Breadcrumbs detected**
You MUST inject the `elementor-breadcrumbs-widget`. Do not use a generic text widget.
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "elementor-breadcrumbs-widget",
  "isInner": false,
  "settings": {
    "seperator": "/",
    "align": "center", // or "left" depending on Figma
    "__globals__": {
      "typography_typography": "globals/typography?id=text",
      "title_color": "globals/colors?id=7c31574",
      "title_hover_color": "globals/colors?id=7c31574"
    }
  },
  "elements": []
}
```

**Scenario B: Normal Text detected (Fallback)**
If it's just raw text, use the standard `heading` or `text-editor` widget.
- ❌ **CRITICAL ERROR TO AVOID**: NEVER EVER inject raw HTML tags like `<p style="text-align: center;">Home / Useful Links</p>` into the text field. Elementor handles this natively.
- ✅ Set the pure text string in the `title` or `editor` field. 
- ✅ Control alignment via the native `align` setting (e.g., `"align": "center"`).
- ✅ Force globals.

Example of standard text fallback:
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {
    "title": "Home / Useful Links", 
    "align": "center", // ALIGNMENT HANDLED HERE, NOT HTML
    "__globals__": {
      "typography_typography": "globals/typography?id=text",
      "title_color": "globals/colors?id=text" 
    }
  },
  "elements": []
}
```

## 6. Referencing the Example
You MUST view and strictly copy the node structures from `resources/page-header-2026-04-10.json` if you are ever unsure of the exact syntax.
