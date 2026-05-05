# 🔥 STRICT Heading + Icon Box Grid Processing Skill

If the target section is a **Heading + Icon Box Grid**, you MUST ABANDON default generative logic and strictly conform to the exact JSON mapping below. Deviating from these rules will break the environment.

## 1. 🏛️ Detection Criteria
A section qualifies as "Heading + Icon Box Grid" when ALL of the following are true:
- A **centered heading** sits at the top of the section
- Below the heading is a **grid of cards**, each card containing:
  - An **SVG icon** at the top
  - A **text description** below the icon
- Cards may have a consistent border, shadow, and background color

## 2. 📐 Root Container Settings
The outer container MUST use these exact settings:
```json
"settings": {
  "padding": { "unit": "px", "top": "60", "right": "60", "bottom": "60", "left": "60", "isLinked": true },
  "padding_tablet": { "unit": "px", "top": "60", "right": "20", "bottom": "60", "left": "20", "isLinked": false },
  "padding_mobile": { "unit": "px", "top": "40", "right": "20", "bottom": "40", "left": "20", "isLinked": false },
  "flex_direction": "column",
  "flex_align_items": "center",
  "background_background": "classic",
  "background_color": "#FFFFFF"
}
```

## 3. 🔤 The Heading Widget (MANDATORY)
The heading MUST use global tokens — never hardcode color or typography:
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {
    "title": "Your Section Title Here",
    "align": "center",
    "__globals__": {
      "title_color": "globals/colors?id=accent",
      "typography_typography": "globals/typography?id=8e88322"
    }
  },
  "elements": []
}
```

## 4. 🔲 The Grid Container (MANDATORY EXACT MATCH)
The grid container MUST use `container_type: "grid"` — NOT a flexbox row. Responsive columns are required:
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {
    "container_type": "grid",
    "grid_columns_grid": { "unit": "fr", "size": 5 },
    "grid_columns_grid_tablet": { "unit": "fr", "size": 3 },
    "grid_columns_grid_mobile": { "unit": "fr", "size": 1 },
    "grid_gaps": { "column": "20", "row": "20", "unit": "px" },
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true },
    "grid_rows_grid": { "unit": "fr", "size": 1, "sizes": [] },
    "presetTitle": "Grid",
    "presetIcon": "eicon-container-grid"
  },
  "elements": [ /* icon box card containers here */ ]
}
```
- Adjust `grid_columns_grid.size` to match the number of columns from Figma (default 5 desktop, 3 tablet, 1 mobile).
- Do NOT change the `grid_gaps` unless Figma explicitly specifies a different gap.

## 5. 🃏 Each Icon Box Card Container
Every card is a **column container** with these EXACT structural rules:

### Border Radius Rule (CRITICAL)
The top-left corner is always SQUARE (0), the other three corners are rounded (20px):
```json
"border_radius": { "unit": "px", "top": "0", "right": "20", "bottom": "20", "left": "20", "isLinked": false }
```

### Full Card Container Template:
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {
    "flex_direction": "column",
    "flex_align_items": "flex-start",
    "flex_align_items_mobile": "center",
    "flex_gap_mobile": { "column": "10", "row": "10", "isLinked": true, "unit": "px", "size": 10 },
    "background_background": "classic",
    "padding": { "unit": "px", "top": "25", "right": "20", "bottom": "25", "left": "20", "isLinked": false },
    "border_radius": { "unit": "px", "top": "0", "right": "20", "bottom": "20", "left": "20", "isLinked": false },
    "border_border": "solid",
    "border_width": { "unit": "px", "top": "1", "right": "1", "bottom": "1", "left": "1", "isLinked": true },
    "border_color": "#FBC8B0",
    "box_shadow_box_shadow_type": "yes",
    "box_shadow_box_shadow": { "horizontal": 0, "vertical": 4, "blur": 10, "spread": 0, "color": "rgba(0, 0, 0, 0.05)" },
    "__globals__": {
      "background_color": "globals/colors?id=7c31574"
    }
  },
  "elements": [ /* icon widget, then text-editor widget */ ]
}
```

## 6. 🎨 The Icon Widget (SVG ONLY — MANDATORY)

Icons in this section type are **always SVG**. Never export as PNG or JPEG — SVG preserves crisp edges at all screen sizes and allows CSS color overrides.

### Icon Resolution Workflow (MANDATORY — do this before building JSON)
1. Use `figma_get_component_image` with the icon's `nodeId` and **`format: "SVG"`** to export the SVG from Figma.
2. Upload the SVG to WordPress Media Library via `saap-elementor-mcp`.
3. Use the resulting WordPress `attachment_id` and `url` in the widget.

❌ Do NOT use `format: "PNG"` or `format: "JPEG"` for icons in this section type.

```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "icon",
  "isInner": false,
  "settings": {
    "selected_icon": {
      "value": {
        "url": "https://yoursite.com/wp-content/uploads/icon-name.svg",
        "id": 1155
      },
      "library": "svg"
    },
    "icon_color": "#E87A2E",
    "icon_size": { "unit": "px", "size": 30 },
    "size": { "unit": "px", "size": 60, "sizes": [] },
    "__globals__": {
      "primary_color": "globals/colors?id=e3a9709"
    }
  },
  "elements": []
}
```
- `icon_size` = the visual icon size inside the box (default 30px)
- `size` = the overall clickable/display box size (default 60px)
- Always use `"library": "svg"` for uploaded SVGs.

## 7. 📝 The Text Widget
Use `text-editor` (NOT `heading`) for card body text. Use global tokens — never hardcode color or typography:
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "text-editor",
  "isInner": false,
  "settings": {
    "editor": "Your card description text here",
    "align_mobile": "center",
    "__globals__": {
      "typography_typography": "globals/typography?id=text",
      "text_color": "globals/colors?id=text"
    }
  },
  "elements": []
}
```
- ❌ NEVER inject raw HTML like `<p style="text-align: center;">` into the `editor` field for alignment. Use `align_mobile` instead.
- ✅ Use exact text strings from `content_data` in the Figma metadata.

## 8. 📌 Optional Footer Text Below Grid
If the Figma design includes a sub-text or contact note below the grid (e.g. "Please contact our Secretariat..."), add a standalone `text-editor` widget after the grid container:
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "text-editor",
  "isInner": false,
  "settings": {
    "editor": "<p>Your footer note text here.</p>",
    "align": "center",
    "_element_width": "initial",
    "_element_custom_width": { "unit": "px", "size": 768, "sizes": [] },
    "__globals__": {
      "typography_typography": "globals/typography?id=text",
      "text_color": "globals/colors?id=4e41099",
      "link_color": "globals/colors?id=accent"
    }
  },
  "elements": []
}
```

## 9. 📖 Referencing the Example
You MUST view and strictly copy the node structures from `resources/heading-with-icon-box-grid.json` if you are ever unsure of the exact syntax.
