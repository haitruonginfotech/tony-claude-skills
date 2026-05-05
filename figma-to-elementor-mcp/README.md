# figma-to-elementor-mcp

Skill chuyển đổi Figma design (qua MCP metadata hoặc screenshot) thành Elementor v4 JSON template, đảm bảo layout, spacing, typography và responsive chính xác — chỉ build từng section độc lập, không ảnh hưởng đến header/footer toàn cục.

---

## Cách hoạt động

1. Nhận Node ID từ Figma (hoặc screenshot)
2. Dùng `figma-console` MCP để lấy metadata cấu trúc
3. Detect loại section → đọc reference file tương ứng
4. Export ảnh/SVG từ Figma → upload WordPress Media Library
5. Build Elementor v4 JSON và trả về raw JSON

---

## Section Types được nhận diện

| Section Type | Reference File | Resource JSON |
|---|---|---|
| Page Header | `references/page-header.md` | `resources/page-header-2026-04-10.json` |
| Heading + Icon Box Grid | `references/heading-with-icon-box-grid.md` | `resources/heading-with-icon-box-grid.json` |
| Generic (fallback) | — | — |

---

## Cấu trúc thư mục

```
figma-to-elementor-mcp/
├── SKILL.md                          # Skill chính (entry point)
├── README.md                         # File này
├── references/
│   ├── page-header.md                # Quy tắc bắt buộc cho Page Header
│   └── heading-with-icon-box-grid.md # Quy tắc bắt buộc cho Heading + Icon Box Grid
└── resources/
    ├── page-header-2026-04-10.json   # Ví dụ JSON thực tế: Page Header
    ├── heading-with-icon-box-grid.json # Ví dụ JSON thực tế: Icon Box Grid
    └── figma-page-data.json          # Ví dụ Figma metadata input
```

---

## MCP Dependencies

| MCP | Dùng để |
|---|---|
| `figma-console` (`figma-console-mcp@latest`) | Đọc metadata Figma, export ảnh/SVG |
| `saap-elementor-mcp` | Upload media lên WordPress, import Elementor JSON |

---

## Changelog

### v1.04 — 2026-05-05
- **Bắt buộc SVG** cho icon trong section Heading + Icon Box Grid (không dùng PNG/JPEG)
- Thêm README.md để theo dõi changelog

### v1.03 — 2026-05-05
- Thêm section type mới: **Heading + Icon Box Grid**
  - Tạo `references/heading-with-icon-box-grid.md` với đầy đủ quy tắc mapping
  - Grid container (`container_type: "grid"`) responsive 5→3→1 columns
  - Border-radius đặc biệt: top-left = 0, 3 góc còn lại = 20px
  - Icon: export SVG qua `figma_get_component_image`, upload WordPress trước khi build JSON
- SKILL.md: Section Type Detection mở rộng thành danh sách ưu tiên (Page Header → Icon Box Grid → Generic)

### v1.02 — 2026-05-05
- **Image Resolution Workflow** hoàn chỉnh 3 bước (thay thế placeholder mặc định):
  1. `figma_get_component_image(nodeId, format: "PNG")` → Figma S3 URL
  2. Upload lên WordPress Media Library qua MCP → URL vĩnh viễn
  3. Dùng WordPress URL trong Elementor JSON (không dùng Figma S3 URL vì expire 30 ngày)
- Placeholder chỉ còn là fallback cuối cùng khi tất cả MCP tools thất bại
- Thêm bước Image Resolution vào Execution Flow (trước bước build JSON)
- Đổi tên `figma-console-scp` → `figma-console` cho đúng tên MCP thực tế
- Add `figma-console` MCP vào `~/.claude/settings.json`

### v1.01 — (trước 2026-05-05)
- Thêm section type: **Page Header**
  - Tạo `references/page-header.md` với mapping bắt buộc
  - Dynamic tag cho heading title (`__dynamic__`)
  - Breadcrumbs widget vs text widget fallback
  - Min-heights bắt buộc: 350px desktop, 300px tablet, 240px mobile

### v1.00 — (ban đầu)
- Skill khởi tạo: chuyển đổi Figma MCP metadata → Elementor v4 JSON
- Hai input modes: Figma MCP và Screenshot fallback
- Mapping rules: layout, typography, design tokens, widget optimization
- ID generation: 8-char hex, unique mỗi lần chạy
- Responsive defaults tự động inject vào top-level containers
