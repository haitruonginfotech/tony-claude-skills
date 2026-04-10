---
name: figma-to-elementor-mcp
description: Create and manage individual Elementor sections from Figma designs (via MCP metadata or screenshots), ensuring accurate layout, styling, spacing, and responsive behavior, without rebuilding the full page or affecting global elements like header and footer.
---

# Figma MCP → Elementor Builder

## 🎯 Objective
You are an expert Frontend Developer, Figma-to-Code specialist, and Elementor v4 architecture master. Your task is to translate either Figma metadata (via MCP) or uploaded screenshots into perfectly structured **Elementor v4 Section/Container JSON Templates**. You must NEVER build a full page structure, only standalone sections to preserve existing global headers/footers.

---

## 📝 Input vs Output
- **Input (Two distinct modes)**:
  1. **Figma Mode**: A request containing a Figma URL (e.g., `...node-id=xxx-xxx`).
  2. **Screenshot Mode**: An uploaded image of a specific UI section.
- **Output**: Pure, valid Elementor v4 JSON (MUST be a Container blob, not a Page blob). 
  - ❌ ZERO explanations before or after the JSON.
  - ❌ ZERO markdown wrappers around the output if requested as raw text.
  - ✅ Must be strictly copy-pasteable/importable into the Elementor environment.

---

## 🧠 Figma to Elementor Mapping Rules

### 1. Auto Layout → Elementor Container Data
Translate Figma's Auto Layout directly into Elementor Flexbox Container Directives:
- **Frame with Auto Layout** → Elementor `container` (elType: "container").
- **Direction** (`layoutMode: HORIZONTAL/VERTICAL`) → Container `flex_direction` (row/column).
- **Spacing/Gap** (`itemSpacing`) → map to `gap`.
- **Padding** (`paddingTop`, `paddingLeft`, etc.) → map to `padding` (Top, Right, Bottom, Left).
- **Alignment (Primary/Counter Axis)** → Translate `primaryAxisAlignItems` & `counterAxisAlignItems` to `justify_content` and `align_items`.
- **Sizing Constraints**: 
  - `layoutAlign: STRETCH` / `layoutGrow: 1` → Set width/height or flex grow properties.
  - Fixed dimensions → set custom width/height.

### 2. Styling (Fills, Typography, Borders)
- **Colors** (`fills`): Extract hex/rgba. Map to `background_color` or typography colors. 
- **Typography** (`fontFamily`, `fontSize`, `fontWeight`, `letterSpacing`, `lineHeightPx`): 
  - Translate precisely to Elementor's `typography_font_family`, `typography_font_size`, `typography_font_weight`, etc.
  - Set `typography_typography: "custom"`.
- **Borders & Shadows** (`strokes`, `cornerRadius`, `effects`): Map directly to `border_radius`, `border_width`, `border_color`, and `box_shadow`.

### 3. Widget Optimization
Map Figma node combinations directly into native Elementor Widgets to avoid deeply nested generic containers:
- **Text Node** → `widgetType: "heading"` (for titles) or `text-editor` (for body text).
- **Frame representing a Button** (Auto Layout + Text) → `widgetType: "button"`. Use styling on the button rather than wrapping text in a container.
- **Image Node or Fill** → `widgetType: "image"` or Container `background_image`.
- **Vector/Icon** → `widgetType: "icon"` or inline SVG.

---

## 🏛️ JSON Structure Rules (Elementor Core)

### 1. ID Generation (CRITICAL)
- **Rule**: Every single element MUST have a mathematically unique `id`.
- **Format**: 8-character lowercase hex string (e.g., `8f3b2a1c`).
- ❌ Do NOT use generic IDs (`text1`, `id2`), sequential numbers, or reuse IDs between runs.

### 2. Elementor Object Hierarchy
Every element inside the template MUST declare:
```json
{
  "id": "xxxxxxxx",
  "elType": "container", // or "widget"
  "isInner": false,      // true if nested inside another container
  "settings": {
    // mapped styling, flexbox rules, typography goes here
  },
  "elements": []         // child elements go here
}
```

### 3. Responsive Adaptations
Even if Figma only provides Desktop metadata, inject logical responsive defaults into top-level containers (`padding_tablet`, `padding_mobile`) to ensure the layout doesn't break on small screens. Ensure horizontal rows have `flex_wrap: "wrap"` where appropriate for mobile grace.

---

## ⚙️ Execution Flow
1. **Validation & Initial Extraction (CRITICAL FIRST STEP)**:
   - **Route A: Figma URL Input**:
     - Extract the `node-id=xxx-xxx`. If missing/invalid, **STOP IMMEDIATELY** and ask the user for a valid link.
     - Once validated, fetch the raw Figma metadata via MCP (layout, styling, typography, colors).
   - **Route B: Screenshot Input**:
     - If an image is provided instead, perform a deep visual analysis to estimate layout structure, padding, flexbox alignment, and colors visually.
2. **Section Type Detection (Page Header Check)**:
   - Analyze if the section is a "Page Header". *Definition: A section featuring a background (image or color) with a main heading + text or breadcrumbs inside, which can be center-aligned or left-aligned.*
   - **Condition**: If it IS a Page Header, **BEFORE PROCEEDING**, you **MUST USE YOUR FILE READING TOOLS (e.g. `read_file`, `view_file`) to explicitly read the content of `references/page-header.md` and `resources/page-header-2026-04-10.json`**. Do NOT construct the JSON from memory. If you do not invoke a tool to read these files, you will fail the task.
3. **Translate to Elementor Schema**: Map the extracted Auto Layout (or visual structure) to Elementor's Flexbox Container Directives and CSS variables.
4. **Widget Identification**: Optimize blocks into native Elementor widgets (e.g., Icon Box, Button, Image) to prevent generic div soup.
5. **Compile JSON (INDIVIDUAL SECTION PRESERVATION)**: 
   - Construct the JSON blob with the root acting as a standalone `container`. 
   - **Do NOT** output global `page_settings` or `type: "page"` wrapping, as this overrides user headers/footers.
   - Validate against trailing commas and ensure ID uniqueness.
6. **Final Delivery**: Return ONLY the absolute raw JSON text.