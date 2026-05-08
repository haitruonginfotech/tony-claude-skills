# 🔥 STRICT Split Hero Processing Skill

If the target section is a **Split Hero**, you MUST ABANDON default generative logic and strictly conform to the exact JSON mapping below. Deviating from these rules will break the environment.

## 1. 🏛️ Detection Criteria
A section qualifies as "Split Hero" when ALL of the following are true:
- The layout is **two columns side by side** (horizontal, NOT a grid)
- One column contains a **large raster image**
- The other column contains **text content**: a heading, body text, and/or a button
- Either arrangement is valid: **image-left + text-right**, or **text-left + image-right**

> **How to distinguish from Image Box Grid:** Split Hero is a single row with 2 columns (one image, one text block). Image Box Grid is a multi-card repeating grid where every card has the same image+text structure.

---

## 2. ⚠️ CRITICAL: Layout Architecture
This section uses `flex_direction: "row"` at the root level — NOT a CSS grid. The two columns are direct `isInner: true` child containers of the root.

Structure overview:
```
Root Container (flex_direction: row)
├── Column A — isInner container (image OR text)
└── Column B — isInner container (text OR image)
```

---

## 3. 📐 Root Container Settings
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": false,
  "settings": {
    "_title": "Split hero",
    "flex_direction": "row",
    "flex_align_items": "center",
    "flex_gap": { "column": "40", "row": "40", "isLinked": true, "unit": "px", "size": 40 },
    "flex_gap_tablet": { "column": "30", "row": "30", "isLinked": true, "unit": "px", "size": 30 },
    "flex_gap_mobile": { "column": "20", "row": "20", "isLinked": true, "unit": "px", "size": 20 },
    "flex_direction_mobile": "column",
    "padding": { "unit": "px", "top": "60", "right": "60", "bottom": "60", "left": "60", "isLinked": true },
    "padding_tablet": { "unit": "px", "top": "60", "right": "20", "bottom": "60", "left": "20", "isLinked": false },
    "padding_mobile": { "unit": "px", "top": "40", "right": "20", "bottom": "40", "left": "20", "isLinked": false }
  },
  "elements": []
}
```
- `_title` MUST be `"Split hero"`.
- `flex_direction_mobile: "column"` is MANDATORY — ensures the two columns stack vertically on mobile instead of squeezing side by side.
- Derive `background_color` or `background_image` from Figma if the section has a background fill. If present, follow the **Image Resolution Workflow (Step 0→3)** in SKILL.md.
- Adjust `flex_gap` if Figma specifies a different column gap.

---

## 4. 🖼️ Image Column Container
The column that holds the image is an `isInner: true` container with zero padding:
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {
    "content_width": "full",
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true }
  },
  "elements": [ ]
}
```
- Place the `image` widget (see Section 6) directly inside `elements`.

---

## 5. 📝 Text Column Container
The column that holds the text content is an `isInner: true` container with zero padding:
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {
    "content_width": "full",
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true }
  },
  "elements": [ ]
}
```
- Place heading → text-editor → button (in that order) inside `elements`.
- Only include widgets that exist in the Figma design — not all three are always present.

---

## 6. 🖼️ The Image Widget
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "image",
  "isInner": false,
  "settings": {
    "image": {
      "url": "https://yoursite.com/wp-content/uploads/split-hero-image.jpg",
      "id": 759,
      "size": "",
      "alt": "",
      "source": "library"
    }
  },
  "elements": []
}
```

### Image Resolution Workflow (MANDATORY — before building JSON)
1. Use `figma_get_component_image` with the image node's `nodeId` and **`format: "PNG"`** (or `"JPEG"` for photographic content). If the node has a SOLID overlay fill, follow **Step 0 Path A/B** from SKILL.md first.
2. Upload to WordPress Media Library via `saap-elementor-mcp`.
3. Use the resulting `attachment_id` and permanent `url` in the `image` field.

---

## 7. 🔤 The Heading Widget
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {
    "title": "Your Heading Here",
    "__globals__": {
      "title_color": "globals/colors?id=accent",
      "typography_typography": "globals/typography?id=primary"
    }
  },
  "elements": []
}
```
- Use exact text from Figma `content_data`. Never hallucinate text.
- Always use `__globals__` for color and typography — never hardcode.

---

## 8. 📄 The Text Widget
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "text-editor",
  "isInner": false,
  "settings": {
    "editor": "<p>Your body text here.</p>",
    "__globals__": {
      "typography_typography": "globals/typography?id=text",
      "text_color": "globals/colors?id=text"
    }
  },
  "elements": []
}
```
- Wrap text in `<p>` tags inside the `editor` field.
- Use exact text from Figma `content_data`.

---

## 9. 🔘 The Button Widget (if present in Figma)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "button",
  "isInner": false,
  "settings": {
    "text": "Button Label",
    "__globals__": {
      "button_text_color": "globals/colors?id=text",
      "typography_typography": "globals/typography?id=text"
    }
  },
  "elements": []
}
```
- Only include if the Figma design has a button in the text column.
- Derive label text, colors, and padding from Figma design tokens.

---

## 10. 🔄 Image-Left vs Image-Right Variants
Control the visual order purely by the **order of the two child containers** inside the root's `elements` array:

- **Image-Left / Text-Right** (default): `[image_container, text_container]`
- **Image-Right / Text-Left**: `[text_container, image_container]`

Do NOT use CSS `order` or `flex_direction: "row-reverse"` — just swap the array order.

---

## 11. 📖 Referencing the Example
You MUST read and strictly copy the node structures from `resources/elementor-split-hero-763-2026-05-08.json` before writing any JSON for this section type.
