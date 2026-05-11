# 🔥 STRICT Centered Text Hero Processing Skill

If the target section is a **Centered Text Hero**, you MUST ABANDON default generative logic and strictly conform to the exact JSON mapping below. Deviating from these rules will break the environment.

## 1. 🏛️ Detection Criteria
A section qualifies as "Centered Text Hero" when ALL of the following are true:
- A **full-width section** with a background image or color
- All content (heading, body text, button) is **horizontally and vertically centered**
- There is **no column split** — everything sits in a single centered column
- Typically used as a promotional or CTA banner

> **How to distinguish from similar sections:**
> - vs **Page Header**: Page Header uses a dynamic page title + breadcrumbs and appears at the top of every page. Centered Text Hero is a standalone promotional section with custom copy and a CTA button.
> - vs **Split Hero**: Split Hero has two columns (image + text side by side). Centered Text Hero has no columns — everything is centered in one container.

---

## 2. ⚠️ CRITICAL: Layout Architecture
This section uses a **single root container** with no inner containers. All widgets (heading, text-editor, button) are placed **directly** inside the root container's `elements` array.

❌ Do NOT create inner column containers.
✅ Place all widgets directly inside the root container.

---

## 3. 📐 Root Container Settings
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": false,
  "settings": {
    "_title": "Centered text",
    "min_height": { "unit": "px", "size": 500, "sizes": [] },
    "min_height_tablet": { "unit": "px", "size": 400, "sizes": [] },
    "min_height_mobile": { "unit": "px", "size": 300, "sizes": [] },
    "flex_justify_content": "center",
    "flex_align_items": "center",
    "background_background": "classic",
    "background_image": {
      "url": "https://yoursite.com/wp-content/uploads/hero-bg.jpg",
      "id": 829,
      "size": "",
      "alt": "",
      "source": "library"
    },
    "background_position": "center center",
    "background_repeat": "no-repeat",
    "background_size": "cover",
    "padding": { "unit": "px", "top": "60", "right": "60", "bottom": "60", "left": "60", "isLinked": true },
    "padding_tablet": { "unit": "px", "top": "60", "right": "20", "bottom": "60", "left": "20", "isLinked": false },
    "padding_mobile": { "unit": "px", "top": "40", "right": "20", "bottom": "40", "left": "20", "isLinked": false }
  },
  "elements": []
}
```
- `_title` MUST be `"Centered text"`.
- `flex_justify_content: "center"` + `flex_align_items: "center"` are both MANDATORY for centering.
- Derive `min_height` from Figma's section height if specified; use the defaults above otherwise.
- Background image MUST be resolved via the **Image Resolution Workflow** (see Section 4).

---

## 4. 🖼️ Background Image (MANDATORY — follow SKILL.md Image Resolution Workflow)
This section always has a full-width background image. You MUST execute **Step 0 → Step 1 → Step 2 → Step 3** from the SKILL.md Image Resolution Workflow before writing any JSON.

**If Path A was used (clean export + Elementor overlay):** add overlay settings:
```json
"background_overlay_background": "classic",
"background_overlay_color": "#000000",
"background_overlay_opacity": { "unit": "px", "size": 0.4, "sizes": [] }
```
Use the `opacity` value from the Figma SOLID fill.

**If Path B was used (overlay baked into image):** do NOT add any `background_overlay_*` settings.

---

## 5. 🔤 The Heading Widget (MANDATORY)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {
    "title": "Your Heading Here",
    "align": "center",
    "__globals__": {
      "title_color": "globals/colors?id=7c31574",
      "typography_typography": "globals/typography?id=primary"
    }
  },
  "elements": []
}
```
- `align: "center"` is MANDATORY.
- Use exact text from Figma `content_data`. Never hallucinate text.
- Always use `__globals__` for color and typography.

---

## 6. 📄 The Text Widget (if present in Figma)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "text-editor",
  "isInner": false,
  "settings": {
    "editor": "<p>Your body text here.</p>",
    "align": "center",
    "__globals__": {
      "typography_typography": "globals/typography?id=text",
      "text_color": "globals/colors?id=text"
    }
  },
  "elements": []
}
```
- `align: "center"` is MANDATORY.
- Only include if body text exists in the Figma design.

---

## 7. 🔘 The Button Widget (if present in Figma)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "button",
  "isInner": false,
  "settings": {
    "text": "Button Label",
    "align": "center",
    "__globals__": {
      "button_text_color": "globals/colors?id=text",
      "typography_typography": "globals/typography?id=text"
    }
  },
  "elements": []
}
```
- `align: "center"` is MANDATORY.
- Only include if a button exists in the Figma design.
- Derive label text, colors, and padding from Figma design tokens.

---

## 8. 📖 Referencing the Example
You MUST read and strictly copy the node structures from `resources/elementor-centered-text-hero-842-2026-05-11.json` before writing any JSON for this section type.
