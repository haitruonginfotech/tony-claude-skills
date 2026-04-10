---
name: screenshot-elementor-template
description: Generate Elementor v4 JSON templates from images with high accuracy and import compatibility.
---

# From Screenshot to Elementor Template

## Input
- Image (UI, screenshot, section, or full page)

---

## Output
- Valid Elementor v4 JSON ONLY
- No explanation
- Directly importable
- Must follow Output Types

---

## Output Types

### 1. Container Template
Use when:
- Single section only

Output:
- MUST return JSON EXACTLY following structure below:

```json
{
  "content": [],
  "type": "container"
}
```

### 2. Full Page Template
Use when:
- Multiple sections

Output:
- JSON including:
  - content
  - page_settings
  - version: "0.4"
  - title
  - type: "page"

```json
{
  "content": [],
  "page_settings": [],
  "version": "0.4",
  "title": "Generated Page",
  "type": "page"
}
```

---

## Output Decision Rule
- 1 section → Container
- Multiple sections → Page

---

## Output Execution Order
1. Detect layout type
2. Detect patterns
3. Build structure
4. Generate JSON

---

# 🔥 CRITICAL: ID GENERATION RULE

## ID Rules
- Every element MUST have unique id
- Use 8-character lowercase hex string

### Example:
- "a3f9c1d2"
- "7b82aa91"

### NEVER:
- Reuse IDs
- Use incremental IDs (id1, id2)
- Use readable names

---

# 🔥 CRITICAL: STRUCTURE RULE (REAL ELEMENTOR)

## Container
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": false,
  "settings": {},
  "elements": []
}
```
## Inner Container
```json
{
  "id": "xxxxxxxx",
  "elType": "container",
  "isInner": true,
  "settings": {},
  "elements": []
}
```
## Widget
```json
{
  "id": "xxxxxxxx",
  "elType": "widget",
  "widgetType": "heading",
  "isInner": false,
  "settings": {},
  "elements": []
}
```
---

# 🔥 LAYOUT SYSTEM

## Split Layout (Image + Content)

### Structure:
- Parent container (isInner: false)
- 2 inner containers (isInner: true)

### Padding (MANDATORY - STRICT):
- Desktop: 60 60 60 60
- Tablet: 60 20 60 20
- Mobile: 40 20 40 20

Rules:

* MUST match EXACT values above
* MUST NOT modify spacing values
* MUST NOT infer or optimize spacing
* ANY deviation = INVALID OUTPUT

### JSON Reference (MUST FOLLOW)

```json
"padding": {
  "unit": "px",
  "top": "60",
  "right": "60",
  "bottom": "60",
  "left": "60",
  "isLinked": true
},
"padding_tablet": {
  "unit": "px",
  "top": "60",
  "right": "20",
  "bottom": "60",
  "left": "20",
  "isLinked": false
},
"padding_mobile": {
  "unit": "px",
  "top": "40",
  "right": "20",
  "bottom": "40",
  "left": "20",
  "isLinked": false
}
```

---

## Grid Layout (IMPORTANT)

When detecting grid:

```json
"container_type": "grid",
"grid_columns_grid": {
  "unit": "fr",
  "size": 4
},
"grid_gaps": {
  "column": "20",
  "row": "20",
  "unit": "px"
}
```

---

# 🔥 RESPONSIVE RULE

- Use:
  - padding
  - padding_tablet
  - padding_mobile

- Always include:
"isLinked": true or false

---

# 🔥 STYLE SYSTEM (ADVANCED)

## Typography
If detected:
```json
"typography_typography": "custom",
"typography_font_family": "Inter"
```
---

## Global Style (IMPORTANT)

Use when possible:
```json
"__globals__": {
  "title_color": "globals/colors?id=xxxx"
}
```
---

## 🔥 GLOBAL COLOR RULE (UPDATED)

If heading is:
- Orange → MUST use Global Color: Accent

---

## Dynamic Content (OPTIONAL)
```json
"__dynamic__": {
  "title": "[elementor-tag id=\"xxxx\" name=\"page-title\"]"
}
```
---

## 🔥 TEXT EDITOR GLOBAL STYLE RULE (CRITICAL)

Applies ONLY to:
- widgetType: "text-editor"

Rules:
- MUST use global typography: `text`
- MUST use global color: `text`

### Required Structure

```json
"__globals__": {
  "typography_typography": "globals/typography?id=text",
  "text_color": "globals/colors?id=text",
  "title_color": "globals/colors?id=accent"
}
```

## 🔥 CUSTOM CSS RULES (CRITICAL)

- MUST use `selector` for root element
- MUST scope all styles under `selector`
- MUST avoid global class selectors

Example:

```css
selector {
  display: flex;
  align-items: center;
}
```

---

# 🔥 BACKGROUND RULE

If container uses background image:

## REQUIRED SETTINGS

```json
"background_background": "classic",
"background_image": {
  "url": "...",
  "id": xxx,
  "source": "library"
},
"background_position": "center center",
"background_repeat": "no-repeat",
"background_size": "cover"
```
---

# 🔥 CONSTRAINTS (STRICT)

- MUST match Elementor schema
- MUST include:
  - id
  - elType
  - settings
  - elements
- NO:
  - comments
  - explanation
  - invalid fields

---

# 🔥 OUTPUT REQUIREMENTS

- Valid JSON
- No trailing commas
- Correct nesting
- Ready to import

---

# 🔥 REFERENCE (REAL STRUCTURE)

## Page Output
```json
{
  "content": [],
  "page_settings": [],
  "version": "0.4",
  "title": "Generated Page",
  "type": "page"
}
```
## Container Output
```json
{
  "content": []
}
```

# 🔥 FINAL RULE

- ALWAYS prioritize:
  1. Valid structure
  2. Import success
  3. Clean hierarchy

---

## When to Use
- Convert image → Elementor layout