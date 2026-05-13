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
  1. **Figma MCP Mode (Primary)**: The user provides a Node ID. You use the `figma-console` MCP tools to retrieve structural metadata and images directly from Figma.
  2. **Screenshot Mode (Fallback)**: An uploaded image of a specific UI section.
- **Output**: Pure, valid Elementor v4 JSON (MUST be a Container blob, not a Page blob). 
  - ❌ ZERO explanations before or after the JSON.
  - ❌ ZERO markdown wrappers around the output if requested as raw text.
  - ✅ Must be strictly copy-pasteable/importable into the Elementor environment.

---

## 🧠 Figma to Elementor Mapping Rules

### 1. MCP JSON → Elementor Container Data
Translate the parsed JSON arrays (`page_structure`, `layout_grid`) from the MCP tools directly into Elementor Flexbox Container Directives:
- **`page_structure` elements** → Elementor `container` (elType: "container"). Use `position` and `size` to determine absolute/relative dimensions.
- **Direction & Alignment** → Infer `flex_direction`, `justify_content`, and `align_items` from the JSON `alignment` fields and relative `position` coordinates of child nodes.
- **Padding & Margin** → Derive paddings from `position.x_offset` and `y` properties, aided by `design_tokens_used.spacing`.
- **Backgrounds & Images** → Map `fills` (image/solid colors) and `stroke` inside `page_structure` to Elementor `background_color`, `background_image`, or `border`.

#### 🖼️ Image Resolution Workflow (MANDATORY for IMAGE fills)
When a node contains an `IMAGE` fill (identified by `"type": "IMAGE"` or an `imageRef` hash in the metadata), you MUST follow this exact sequence — do NOT use placeholders unless all steps fail:

**⚠️ Step 0 — Pre-check for overlay fills (CRITICAL — do this before exporting):**
Before calling `figma_get_component_image`, inspect the node's `fills` array via `figma_get_component_for_development`. A background node often has **multiple fills stacked**: an IMAGE fill PLUS one or more SOLID fills used as a dark/color overlay (e.g. `{ type: "SOLID", color: {r:0,g:0,b:0}, opacity: 0.4 }`).

`figma_get_component_image` renders ALL fills into a single PNG — the overlay gets permanently baked into the exported image. This breaks Elementor's background overlay feature and makes it impossible to adjust opacity later.

**When multiple fills are detected (IMAGE + SOLID overlay):**

- Note the SOLID fill's `color` and `opacity` — you'll need these values for the Elementor overlay settings.
- If multiple nodes overlap at the same position (e.g. two background rectangles stacked), prefer the node with **`scaleMode: "FILL"`** and the higher node ID — this is typically the designer's most recent, intended version.

**Path A — Figma Desktop Bridge connected in Design mode** (cleanest output):
1. Use `figma_execute` to temporarily hide all SOLID overlay fills on the node:
   ```js
   const node = await figma.getNodeByIdAsync("NODE_ID");
   const fills = [...node.fills];
   node.fills = fills.map((f, i) => i === 0 ? f : { ...f, visible: false });
   return { ok: true };
   ```
2. Proceed to Step 1 to export the clean photo.
3. After upload, restore the overlay fills:
   ```js
   const node = await figma.getNodeByIdAsync("NODE_ID");
   const fills = [...node.fills];
   node.fills = fills.map(f => ({ ...f, visible: true }));
   return { ok: true };
   ```
4. In the Elementor JSON, apply the overlay via container settings (NOT as a baked-in image):
   ```json
   "background_overlay_background": "classic",
   "background_overlay_color": "#000000",
   "background_overlay_opacity": { "unit": "px", "size": 0.4, "sizes": [] }
   ```
   Use the `opacity` value from the Figma SOLID fill.

**Path B — Desktop Bridge in Dev mode (read-only) or not connected:**
- Export the node as-is (overlay will be baked into the PNG).
- Do NOT add any Elementor background overlay on top — the image is already darkened.
- The result is visually approximate. For pixel-perfect output, ask the user to switch Figma to Design mode and rerun via Path A.
- To check which mode is active: call `figma_get_status` and check `editorType` — `"design"` = write access, `"dev"` = read-only.

**Step 1 — Export image from Figma via MCP:**
Call `figma_get_component_image` with the node's `nodeId`. If multiple fill-only nodes overlap at the same position, prefer the one with `scaleMode: "FILL"`:
```
figma_get_component_image(nodeId: "208:463", format: "PNG", scale: 2)
```
This returns a **temporary Figma S3 URL** (valid ~30 days).

**Step 2 — Upload to WordPress Media Library:**
Call `elementor-mcp-sideload-image` (or the relevant WordPress MCP for the current site) to upload the image:
```
elementor-mcp-sideload-image(url: "<figma_s3_url>", title: "section-bg")
```
This returns a WordPress `attachment_id` and permanent `url`.

**Step 3 — Use WordPress URL in Elementor JSON:**
Use the permanent WordPress URL (never the Figma S3 URL) in the `background_image` or `image` widget settings:
```json
"background_image": {
  "url": "https://yoursite.com/wp-content/uploads/2026/section-bg.png",
  "id": 123
}
```

**Fallback** (only if MCP tools are unavailable or all steps fail): use a placeholder URL (`https://placehold.co/1920x1080?text=Figma+Image+Placeholder`). Do NOT add any comment inside the JSON — comments are invalid in JSON and will break import. Notify the user in your text response that this placeholder must be replaced with a real image.

### 2. Styling (Design Tokens & Typography)
- **Colors** (`design_tokens_used.colors`): Extract hex/rgba keys. Map to Elementor `background_color`, borders, or typography colors. 
- **Typography** (`design_tokens_used.typography`): 
  - Translate properties exactly to Elementor's `typography_font_family`, `typography_font_size`, `typography_font_weight`, `typography_line_height`, etc.
  - Set `typography_typography: "custom"`.
- **Content** (`content_data`): Use these exact string values for headings and text widgets. Avoid hallucinating text.
- **Borders & Shadows** (`radii`, `stroke`): Map directly to `border_radius` and `border_width`/`border_color`.

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
   - **Route A: Figma MCP Mode**:
     - The user provides a Node ID or you extract it. If missing, ask for it.
     - **Action**: Retrieve the fully parsed JSON metadata (which includes `page_structure`, `design_tokens_used`, `content_data`, `layout_grid`) via `figma-console` MCP tools. Analyze this payload thoroughly as your layout blueprint.
   - **Route B: Screenshot Input**:
     - If an image is provided instead, perform a deep visual analysis to estimate layout structure, padding, flexbox alignment, and colors visually.

2. **Section Type Detection (CRITICAL — check in order)**:
   Identify the section type from the Figma metadata before building any JSON. Each type has a dedicated reference file — you MUST read it before proceeding.

   - **Page Header**
     *Definition: A full-width banner with a background image or color, containing a main heading and optional breadcrumbs. Center- or left-aligned.*
     **Trigger**: If detected, **MUST read** `references/page-header.md` AND `resources/page-header-2026-04-10.json` before writing any JSON.

   - **Icon Box Grid**
     *Definition: A section with a centered heading at the top, followed by a grid of cards where each card contains a **small, transparent/monochrome SVG icon** (typically 30–60px) and a text description below it.*
     **Trigger**: If detected, **MUST read** `references/icon-box-grid.md` AND `resources/elementor-icon-box-grid-304-2026-05-06.json` before writing any JSON.

   - **Image Box Grid**
     *Definition: A section with a centered heading at the top, followed by a grid of cards where each card contains a **large raster image** (PNG/JPEG photo or illustration that fills the full card width) and a title + description below it. Often has a dark or colored background.*
     **Trigger**: If detected, **MUST read** `references/image-box-grid.md` AND `resources/elementor-image-box-grid-424-2026-05-06.json` before writing any JSON.

   - **Split Hero**
     *Definition: A two-column horizontal section — one column is a large image, the other contains text content (heading, body text, and/or button). Either image-left/text-right or text-left/image-right.*
     **Trigger**: If detected, **MUST read** `references/split-hero.md` AND `resources/elementor-split-hero-763-2026-05-08.json` before writing any JSON.

   - **Centered Text Hero**
     *Definition: A full-width section with a background image, where all content (heading, body text, button) is centered both horizontally and vertically in a single column. No column split. Used as a standalone promotional or CTA banner.*
     **Trigger**: If detected, **MUST read** `references/centered-text-hero.md` AND `resources/elementor-centered-text-hero-842-2026-05-11.json` before writing any JSON.

   - **List**
     *Definition: A section with multiple items stacked vertically, each item being a two-column row (image ~48% left + text block ~48% right), separated by dividers. Text block typically contains a main title, optional subtitle, body text, and a button.*
     **Trigger**: If detected, **MUST read** `references/list.md` AND `resources/elementor-list-889-2026-05-11.json` before writing any JSON.

   - **Accordion**
     *Definition: A collapsible list of items (FAQ, features, Q&A) where each item has a clickable title that expands to reveal content. Items are stacked vertically with a divider between them and a toggle icon (plus/minus) at the right of each title.*
     **Trigger**: If detected anywhere inside the section, **MUST read** `references/accordion.md` AND `resources/elementor-accordion-995-2026-05-13.json` before writing any JSON.

   > **Icon vs Image distinction:**
   > - **Icon Box**: graphic is small (30–60px), sits on transparent or flat-color background, monochrome or flat-color SVG → **Icon Box Grid**
   > - **Image Box**: graphic is a large photo or illustration filling most of the card area, raster format → **Image Box Grid**

   - **Generic Section** (fallback)
     If the section does not match any pattern above, proceed with the general mapping rules in Section 🧠 above.

   ❌ Do NOT construct JSON from memory for any recognized section type. If you do not invoke a tool to read the reference files, you will fail the task.

3. **Image Resolution (BEFORE building JSON)**:
   - Scan all nodes in `page_structure` for `IMAGE` fills or `imageRef` values.
   - For each image found, execute the **Image Resolution Workflow** (Step 0→1→2→3 above) to obtain a permanent WordPress URL **before** writing the Elementor JSON.
   - This ensures the final JSON contains real, permanent URLs — not Figma S3 links that expire.

4. **Translate to Elementor Schema**: Map the extracted Auto Layout (or visual structure) to Elementor's Flexbox Container Directives and CSS variables.

5. **Widget Identification**: Optimize blocks into native Elementor widgets (e.g., Icon Box, Button, Image) to prevent generic div soup.

6. **Compile JSON (INDIVIDUAL SECTION PRESERVATION)**: 
   - Construct the JSON blob with the root acting as a standalone `container`. 
   - **Do NOT** output global `page_settings` or `type: "page"` wrapping, as this overrides user headers/footers.
   - Validate against trailing commas and ensure ID uniqueness.

7. **Final Delivery**: Return ONLY the absolute raw JSON text.
