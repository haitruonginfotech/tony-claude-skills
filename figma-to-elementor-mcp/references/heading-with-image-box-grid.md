# 🔥 STRICT Heading + Image Box Grid Processing Skill

If the target section is a **Heading + Image Box Grid**, you MUST ABANDON default generative logic and strictly conform to the exact JSON mapping below. Deviating from these rules will break the environment.

## 1. 🏛️ Detection Criteria
A section qualifies as "Heading + Image Box Grid" when ALL of the following are true:
- A **centered heading** sits at the top of the section
- Below the heading is a **grid of cards**, each card containing:
  - A **raster image** (PNG or JPEG — not an SVG vector icon) at the top
  - A **text description** below the image
- Cards may have a consistent border, shadow, and background color

> **How to distinguish from Heading + Icon Box Grid:**
> If the card visual asset is a vector/icon → use `heading-with-icon-box-grid.md`.
> If the card visual asset is a photo, illustration, or bitmap image → use **this file**.

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
    "grid_columns_grid": { "unit": "fr", "size": 4 },
    "grid_columns_grid_tablet": { "unit": "fr", "size": 2 },
    "grid_columns_grid_mobile": { "unit": "fr", "size": 1 },
    "grid_gaps": { "column": "20", "row": "20", "unit": "px" },
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true },
    "grid_rows_grid": { "unit": "fr", "size": 1, "sizes": [] },
    "presetTitle": "Grid",
    "presetIcon": "eicon-container-grid"
  },
  "elements": [ /* image box card containers here */ ]
}
```
- Adjust `grid_columns_grid.size` to match the number of columns from Figma (default 4 desktop, 2 tablet, 1 mobile for image grids).
- Do NOT change the `grid_gaps` unless Figma explicitly specifies a different gap.

## 5. 🃏 Each Image Box Card Container
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
  "elements": [ /* image widget, then text-editor widget */ ]
}
```

## 6. 🖼️ The Image Widget (PNG/JPEG — MANDATORY)

Card images are **raster images** (PNG or JPEG). Never export as SVG — use the `image` widget, not the `icon` widget.

### Image Resolution Workflow (MANDATORY — do this before building JSON)
1. Use `figma_get_component_image` with the image node's `nodeId` and **`format: "PNG"`** (or `"JPEG"` if the image is photographic) to export from Figma.
2. Upload to WordPress Media Library via `saap-elementor-mcp`.
3. Use the resulting WordPress `attachment_id` and `url` in the widget.

❌ Do NOT use `widgetType: "icon"` for this section type.
❌ Do NOT use `format: "SVG"` for raster card images.

```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "image",
  "isInner": false,
  "settings": {
    "image": {
      "url": "https://yoursite.com/wp-content/uploads/card-image.png",
      "id": 1200
    },
    "image_size": "full",
    "width": { "unit": "px", "size": 80, "sizes": [] },
    "align": "left",
    "align_mobile": "center"
  },
  "elements": []
}
```
- `width` = derive from Figma node `size.width` (default 80px if not specified).
- `align`: use `"left"` to match card's `flex_align_items: "flex-start"`, `"center"` for mobile.
- `image_size`: always `"full"` to avoid WordPress thumbnail cropping.

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
If the Figma design includes a sub-text or contact note below the grid, add a standalone `text-editor` widget after the grid container:
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
You MUST view and strictly copy the node structures from `resources/heading-with-image-box-grid.json` if you are ever unsure of the exact syntax.
> ⚠️ If `resources/heading-with-image-box-grid.json` does not exist yet, use `resources/heading-with-icon-box-grid.json` as the structural reference and apply the image widget substitution rules above.
