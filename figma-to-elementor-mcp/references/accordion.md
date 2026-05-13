# 🔥 STRICT Accordion Processing Skill

If the target section contains an **Accordion / FAQ**, you MUST ABANDON default generative logic and strictly conform to the exact JSON mapping below. Deviating from these rules will break the environment.

## 1. 🏛️ Detection Criteria
An accordion is present when the Figma design contains:
- A **collapsible list of items** — each item has a clickable title that expands to reveal content
- Commonly used for **FAQ sections**, feature lists, or any expandable Q&A pattern
- Items are stacked vertically with a visible divider or separator between them
- A toggle icon (plus/minus or chevron) is visible at the right of each title

---

## 2. ⚠️ CRITICAL: Widget Type
Always use **`widgetType: "nested-accordion"`** — NOT the legacy `"accordion"` widget type.

The accordion is a widget placed inside its parent section container. It is NOT a standalone section.

---

## 3. 🎛️ The Accordion Widget (Full Template)
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "nested-accordion",
  "isInner": false,
  "settings": {
    "_title": "Accordion",
    "items": [
      { "item_title": "Question or title for item 1", "_id": "xxxxxxxx" },
      { "item_title": "Question or title for item 2", "_id": "xxxxxxxx" },
      { "item_title": "Question or title for item 3", "_id": "xxxxxxxx" }
    ],
    "accordion_item_title_position_horizontal": "stretch",
    "accordion_item_title_icon_position": "end",
    "accordion_item_title_icon": {
      "value": {
        "url": "https://yoursite.com/wp-content/uploads/icon-plus.svg",
        "id": 982
      },
      "library": "svg"
    },
    "accordion_item_title_icon_active": {
      "value": {
        "url": "https://yoursite.com/wp-content/uploads/icon-minus.svg",
        "id": 984
      },
      "library": "svg"
    },
    "faq_schema": "",
    "accordion_border_normal_border": "none",
    "content_border_border": "none",
    "accordion_padding": { "unit": "px", "top": "12", "right": "0", "bottom": "12", "left": "0", "isLinked": false },
    "accordion_padding_tablet": { "unit": "px", "top": "11", "right": "0", "bottom": "11", "left": "0", "isLinked": false },
    "accordion_padding_mobile": { "unit": "px", "top": "10", "right": "0", "bottom": "10", "left": "0", "isLinked": false },
    "icon_size": { "unit": "px", "size": 12, "sizes": [] },
    "custom_css": "selector .e-n-accordion .e-n-accordion-item+.e-n-accordion-item {\n    border-top: 1px solid #5F5F5F;\n}\nselector .e-n-accordion-item[open] div[aria-labelledby^=\"e-n-accordion-item-\"] {\n    padding-bottom: 16px;\n}",
    "__globals__": {
      "title_typography_typography": "globals/typography?id=primary",
      "normal_title_color": "globals/colors?id=primary",
      "hover_title_color": "globals/colors?id=accent",
      "active_title_color": "globals/colors?id=primary",
      "normal_icon_color": "globals/colors?id=primary",
      "hover_icon_color": "globals/colors?id=accent",
      "active_icon_color": "globals/colors?id=primary"
    }
  },
  "elements": []
}
```

### Critical settings explained:
- `items`: one entry per accordion item — `item_title` is the clickable header, `_id` is a unique 7-char hex string.
- `accordion_item_title_icon_position: "end"` — icon on the right side of the title.
- `accordion_border_normal_border: "none"` and `content_border_border: "none"` — borders are handled entirely via `custom_css`, NOT native border settings.
- `custom_css` (MANDATORY — copy exactly, only change `#5F5F5F` if Figma uses a different divider color):
  - First rule adds a `border-top` between items.
  - Second rule adds `padding-bottom: 16px` to open panel content.
- `_title: "Accordion"` — must be set.

---

## 4. 🖼️ Icon Resolution (Plus / Minus SVGs)
The toggle icons are SVG files. Follow this workflow before building JSON:

1. Locate the expand icon (plus) and collapse icon (minus) in the Figma node tree.
2. Export each via `figma_get_component_image` with **`format: "SVG"`**.
3. Upload both to WordPress Media Library via `saap-elementor-mcp`.
4. Use the resulting `attachment_id` and permanent `url` for:
   - `accordion_item_title_icon` → plus/expand icon
   - `accordion_item_title_icon_active` → minus/collapse icon

❌ Do NOT use Font Awesome or any icon library for these — always use uploaded SVGs matching the Figma design.

---

## 5. 🃏 Each Item Panel (Content Container)
For every entry in `settings.items`, there MUST be a corresponding child container inside `elements` — **in the same order**. Each panel container:

```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "isLocked": true,
  "settings": {
    "_title": "item #1",
    "content_width": "full",
    "padding": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true }
  },
  "elements": [
    {
      "id": "xxxxxxxx",
      "elType": "widget",
      "widgetType": "text-editor",
      "isInner": false,
      "settings": {
        "editor": "<p>Answer or body content for this accordion item.</p>",
        "__globals__": {
          "link_color": "globals/colors?id=text"
        }
      },
      "elements": []
    }
  ]
}
```

### Rules:
- `isLocked: true` is MANDATORY on every panel container — do not omit.
- `_title` must follow the pattern `"item #N"` (1-indexed).
- `padding: 0` on the container — vertical spacing is handled by `accordion_padding` in the widget settings and `custom_css`.
- The `text-editor` inside uses `__globals__.link_color` only — no other globals needed.
- Use exact answer text from Figma `content_data`. Wrap in `<p>` tags.
- Number of panel containers MUST exactly match number of entries in `settings.items`.

---

## 6. 📖 Referencing the Example
You MUST read and strictly copy the node structures from `resources/elementor-accordion-995-2026-05-13.json` before writing any JSON for this widget type.
