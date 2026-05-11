# 🔥 STRICT List Processing Skill

If the target section is a **List**, you MUST ABANDON default generative logic and strictly conform to the exact JSON mapping below. Deviating from these rules will break the environment.

## 1. 🏛️ Detection Criteria
A section qualifies as "List" when ALL of the following are true:
- A section containing **multiple items stacked vertically**
- Each item is a **two-column row**: image on the left (~48%), text block on the right (~48%)
- Items are separated by a **divider** or visible spacing between them
- The text block typically contains: a main title, optional sub-title, body text, and a button/link

> **How to distinguish from Split Hero:** Split Hero is a single row (one image + one text block). List has multiple repeating rows, each with its own image and text.

---

## 2. ⚠️ CRITICAL: Layout Architecture
The root container holds a **repeating sequence** of row containers and dividers:

```
Root Container (dark background)
├── Row Container 1  (isInner, flex_direction: row)
│   ├── Image Column (isInner, width: 48%)
│   └── Text Column  (isInner, width: 48%, flex_direction: column)
├── Divider
├── Row Container 2
│   ├── Image Column
│   └── Text Column
├── Divider
└── Row Container N  ← NO divider after the last item
```

- The divider goes **between** items — do NOT add a divider after the last row.
- Derive the number of items from Figma's `page_structure` or `content_data`.

---

## 3. 📐 Root Container Settings
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": false,
  "settings": {
    "background_background": "classic",
    "background_color": "#0A0A0A",
    "padding": { "unit": "px", "top": "60", "right": "60", "bottom": "60", "left": "60", "isLinked": true },
    "padding_tablet": { "unit": "px", "top": "60", "right": "30", "bottom": "60", "left": "30", "isLinked": false },
    "padding_mobile": { "unit": "px", "top": "40", "right": "20", "bottom": "40", "left": "20", "isLinked": false }
  },
  "elements": []
}
```
- Derive `background_color` from Figma. Default is dark `#0A0A0A`.
- Note: tablet padding uses `30px` sides (not 20px) — match this exactly.

---

## 4. 🔲 Each Row Container (one per list item)
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {
    "flex_direction": "row",
    "content_width": "full",
    "flex_gap": { "column": "40", "row": "40", "isLinked": true, "unit": "px", "size": 40 },
    "flex_gap_tablet": { "column": "20", "row": "20", "isLinked": true, "unit": "px", "size": 20 },
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true }
  },
  "elements": []
}
```
- `flex_direction: "row"` is MANDATORY — this is what creates the side-by-side layout.
- Place image column and text column directly inside `elements`.

---

## 5. 🖼️ Image Column Container
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {
    "width": { "unit": "%", "size": 48 },
    "content_width": "full",
    "boxed_width": { "unit": "%", "size": 50, "sizes": [] },
    "boxed_width_tablet": { "unit": "%", "size": "", "sizes": [] },
    "boxed_width_mobile": { "unit": "%", "size": "", "sizes": [] },
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true }
  },
  "elements": []
}
```
- `width: 48%` — leaves room for the gap between columns.
- `boxed_width` is only set for desktop (50%), left empty for tablet and mobile.

---

## 6. 📝 Text Column Container
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {
    "flex_direction": "column",
    "width": { "unit": "%", "size": 48 },
    "content_width": "full",
    "boxed_width": { "unit": "%", "size": 50, "sizes": [] },
    "boxed_width_tablet": { "unit": "%", "size": "", "sizes": [] },
    "boxed_width_mobile": { "unit": "%", "size": "", "sizes": [] },
    "gap": { "unit": "px", "row": "16" },
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true }
  },
  "elements": []
}
```
- `gap: { row: "16" }` controls spacing between text elements inside the column.
- Place widgets in order: main heading → sub-heading (optional) → text-editor → button.

---

## 7. 🖼️ The Image Widget
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "image",
  "isInner": false,
  "settings": {
    "image": {
      "url": "https://yoursite.com/wp-content/uploads/item-image.png",
      "id": 857,
      "size": "",
      "alt": "Item Name",
      "source": "library"
    },
    "image_size": "full",
    "width": { "unit": "%", "size": 100 }
  },
  "elements": []
}
```
- `image_size: "full"` and `width: 100%` are MANDATORY — image fills the full column width.
- Set `alt` from the item name in Figma `content_data`.

### Image Resolution Workflow (MANDATORY — before building JSON)
1. Use `figma_get_component_image` with the image node's `nodeId` and `format: "PNG"`. If overlay fills exist, follow **Step 0 Path A/B** from SKILL.md.
2. Upload to WordPress Media Library via `saap-elementor-mcp`.
3. Use the resulting `attachment_id` and permanent `url`.

---

## 8. 🔤 Main Heading Widget (MANDATORY)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {
    "title": "Item Title Here",
    "typography_typography": "custom",
    "typography_font_weight": "700",
    "typography_text_transform": "uppercase",
    "typography_font_size": { "unit": "px", "size": 40, "sizes": [] },
    "typography_font_size_tablet": { "unit": "px", "size": 32, "sizes": [] },
    "typography_font_size_mobile": { "unit": "px", "size": 28, "sizes": [] },
    "hp_enable_gradient_text": "yes"
  },
  "elements": []
}
```
- `typography_text_transform: "uppercase"` and `typography_font_weight: "700"` are MANDATORY.
- `hp_enable_gradient_text: "yes"` enables the theme's gradient text effect — always include.
- Derive font size from Figma if different from defaults.

---

## 9. 🏷️ Sub-heading Widget (OPTIONAL — only if Figma shows a subtitle)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {
    "title": "Subtitle Here",
    "header_size": "h5",
    "title_color": "#F2DCB3"
  },
  "elements": []
}
```
- `header_size: "h5"` is MANDATORY for the sub-heading.
- `title_color: "#F2DCB3"` — derive from Figma design tokens if different.
- Only include if the Figma design has a visible subtitle below the main title.

---

## 10. 📄 The Text Widget
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "text-editor",
  "isInner": false,
  "settings": {
    "editor": "<p>Item description text here.</p>",
    "__globals__": {
      "typography_typography": "globals/typography?id=text"
    }
  },
  "elements": []
}
```
- Use exact text from Figma `content_data`. Wrap in `<p>` tags.

---

## 11. 🔘 The Button Widget
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "button",
  "isInner": false,
  "settings": {
    "text": "View Product",
    "link": { "url": "#" },
    "typography_typography": "custom",
    "typography_text_transform": "uppercase",
    "typography_letter_spacing": { "unit": "px", "size": 2, "sizes": [] },
    "typography_font_size": { "unit": "px", "size": 14, "sizes": [] },
    "selected_icon": { "value": "fas fa-arrow-right", "library": "fa-solid" },
    "icon_align": "left",
    "__globals__": {
      "color": "globals/colors?id=text"
    }
  },
  "elements": []
}
```
- `typography_text_transform: "uppercase"` and `typography_letter_spacing: 2px` are MANDATORY.
- `selected_icon` with `fas fa-arrow-right` — use unless Figma specifies a different icon.
- Replace `link.url` with the actual product/page URL from Figma if available.

---

## 12. ➖ The Divider Widget (between items)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "divider",
  "isInner": false,
  "settings": {
    "color": "#F2DCB3",
    "gap": { "unit": "px", "size": 0, "sizes": [] },
    "text": "Divider"
  },
  "elements": []
}
```
- Place ONE divider **after each row container except the last**.
- `color: "#F2DCB3"` — derive from Figma design tokens if different.
- `gap: 0` — no extra spacing beyond the root container's padding.

---

## 13. 📖 Referencing the Example
You MUST read and strictly copy the node structures from `resources/elementor-list-889-2026-05-11.json` before writing any JSON for this section type.
