# 🔥 STRICT Image Box Grid Processing Skill

If the target section is an **Image Box Grid**, you MUST ABANDON default generative logic and strictly conform to the exact JSON mapping below. Deviating from these rules will break the environment.

## 1. 🏛️ Detection Criteria
A section qualifies as "Image Box Grid" when ALL of the following are true:
- A **centered heading** sits at the top of the section
- Below the heading is a **grid of cards**, each card containing:
  - A **large raster image** (PNG/JPEG photo or illustration) that fills the full card width
  - A **title** and **description text** below the image
- The section may have a dark or colored background

> **Quick visual test:** If the card graphic is a large photo/illustration filling the card width → Image Box Grid. If it is a small transparent icon (30–60px) → use `icon-box-grid.md` instead.

---

## 2. ⚠️ CRITICAL: Widget Architecture
Unlike Icon Box Grid, this section does **NOT** use separate card containers with image + text-editor children.

**Each card MUST be a single native `widgetType: "image-box"` widget** placed directly inside the grid container. This widget bundles image + title + description into one unit.

❌ Do NOT create individual card containers with image and text-editor children.
✅ Use `widgetType: "image-box"` directly inside the grid container's `elements`.

---

## 3. 📐 Root Container Settings
Derive background color and image from Figma. Structural padding defaults:
```json
"settings": {
  "padding": { "unit": "px", "top": "60", "right": "60", "bottom": "60", "left": "60", "isLinked": true },
  "padding_tablet": { "unit": "px", "top": "60", "right": "20", "bottom": "60", "left": "20", "isLinked": false },
  "padding_mobile": { "unit": "px", "top": "40", "right": "20", "bottom": "40", "left": "20", "isLinked": false },
  "background_background": "classic",
  "background_color": "#0A0A0A",
  "overflow": "hidden",
  "_title": "Image Box Grid"
}
```
- Set `background_color` from Figma's fill (dark or light, as designed — do NOT default to white).
- If Figma has a decorative background image, also populate `background_image`, `background_position`, and `background_repeat`.

---

## 4. 🔤 The Heading Widget (MANDATORY)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {
    "title": "Your Section Title Here",
    "align": "center",
    "_z_index": 1,
    "__globals__": {
      "typography_typography": "globals/typography?id=primary"
    }
  },
  "elements": []
}
```
- Use `globals/typography?id=primary` — NOT `id=8e88322`.
- Use `_z_index: 1` to ensure it renders above any background image.

---

## 5. 🔲 The Grid Container
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {
    "container_type": "grid",
    "presetTitle": "Grid",
    "presetIcon": "eicon-container-grid",
    "grid_rows_grid": { "unit": "fr", "size": 1, "sizes": [] },
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true }
  },
  "elements": []
}
```
- Derive `grid_columns_grid` (desktop/tablet/mobile) from the number of columns in Figma.
- Add `grid_gaps` only if Figma explicitly specifies gap values.
- Place `image-box` widgets directly inside `elements` — no intermediate card containers.

---

## 6. 🖼️ Each Image Box Widget (CRITICAL — native Elementor widget)
Each card is a **single `image-box` widget**, NOT a container with children:

```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "image-box",
  "isInner": false,
  "settings": {
    "image": {
      "url": "https://yoursite.com/wp-content/uploads/card-image.jpg",
      "id": 408,
      "size": "",
      "alt": "",
      "source": "library"
    },
    "title_text": "Card Title Here",
    "description_text": "Card description text here.",
    "text_align": "start",
    "image_size": { "unit": "%", "size": 100, "sizes": [] }
  },
  "elements": []
}
```
- `image_size`: always `{ "unit": "%", "size": 100 }` so the image fills the full card width.
- `text_align`: use `"start"` (left-aligned) unless Figma shows centered text.
- `title_text` and `description_text`: use exact strings from Figma `content_data`.

### Image Resolution Workflow (MANDATORY — before building JSON)
1. Use `figma_get_component_image` with the image node's `nodeId` and **`format: "PNG"`** (or `"JPEG"` for photographic content).
2. Upload to WordPress Media Library via `saap-elementor-mcp`.
3. Use the resulting `attachment_id` and permanent `url` in the widget's `image` field.

❌ Do NOT use `widgetType: "image"` or `widgetType: "icon"` for cards in this section type.
❌ Do NOT use `format: "SVG"` for card images.

---

## 7. 🔘 Optional Button Below Grid
If Figma includes a CTA button after the grid, add a `button` widget (NOT `text-editor`):
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "button",
  "isInner": false,
  "settings": {
    "text": "button label",
    "align": "center",
    "_z_index": 1
  },
  "elements": []
}
```
- Add icon, colors, and padding derived from Figma design tokens.
- Use `_z_index: 1` if the section has a background image.

---

## 8. 📖 Referencing the Example
You MUST read and strictly copy the node structures from `resources/elementor-image-box-grid-424-2026-05-06.json` before writing any JSON for this section type.
