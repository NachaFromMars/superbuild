# Cấu Trúc Skill — YAML Frontmatter & Progressive Disclosure

Nguồn: Anthropic "The Complete Guide to Building Skills for Claude" (Jan 2026)
Đã dịch và tối ưu cho OpenClaw.

---

## Cấu trúc thư mục chuẩn

```
ten-skill/
├── SKILL.md              ← Bắt buộc, tên CHÍNH XÁC (phân biệt hoa thường)
├── scripts/              ← Tùy chọn: code thực thi (Python, Bash, JS...)
├── references/           ← Tùy chọn: tài liệu tra cứu
└── assets/               ← Tùy chọn: mẫu, icon, font
```

### Quy tắc đặt tên

| Đúng | Sai | Lý do |
|------|-----|-------|
| `notion-project-setup` | `Notion Project Setup` | Dùng kebab-case |
| `suno-reforge` | `suno_reforge` | Không dấu gạch dưới |
| `superbuild` | `SuperBuild` | Không viết hoa |

### Quy tắc file

- File chính **phải** tên `SKILL.md` (không phải `skill.md`, `SKILL.MD`, `Skill.md`)
- **Không** có `README.md` trong thư mục skill (dùng SKILL.md thay thế)
- README chỉ dùng ở repo GitHub cấp trên (cho người đọc, không cho agent)

---

## Progressive Disclosure — 3 Tầng

Nguyên tắc: agent chỉ nạp thông tin khi cần, không đổ hết vào context.

| Tầng | Khi nào nạp | Nội dung |
|------|------------|----------|
| **1. Frontmatter** | Luôn luôn (system prompt) | Tên, mô tả, trigger — đủ để biết skill này LÀM GÌ |
| **2. SKILL.md body** | Khi agent quyết định skill liên quan | Hướng dẫn đầy đủ, quy trình, ví dụ |
| **3. Linked files** | Khi agent cần chi tiết cụ thể | references/, scripts/, assets/ |

### Lợi ích
- Tiết kiệm token: chỉ nạp tầng 1 cho mọi skill (~50-100 token)
- Nạp tầng 2-3 chỉ khi thực sự cần (~500-5000 token)
- Tránh "context bloat" khi cài nhiều skills

---

## YAML Frontmatter — Phần Quan Trọng Nhất

Frontmatter quyết định agent có nạp skill không. Sai ở đây = skill vô dụng.

### Format tối thiểu

```yaml
---
name: ten-skill
description: Làm gì. Dùng khi người dùng nói [cụm từ cụ thể].
---
```

### Các trường

| Trường | Bắt buộc | Quy tắc |
|--------|----------|---------|
| `name` | ✅ | kebab-case, khớp tên thư mục, không viết hoa/khoảng trắng |
| `description` | ✅ | Phải có CẢ HAI: (1) làm gì + (2) khi nào dùng. Dưới 1024 ký tự |
| `license` | ❌ | MIT, Apache-2.0... nếu open-source |
| `compatibility` | ❌ | 1-500 ký tự, yêu cầu môi trường |
| `metadata` | ❌ | Key-value tùy ý: author, version, mcp-server... |

### Description — Cách Viết

**Công thức:** `[Làm gì] + [Khi nào dùng] + [Khả năng chính]`

**✅ Tốt:**
```yaml
description: Phân tích file Figma và tạo tài liệu handoff cho dev.
  Dùng khi người dùng upload file .fig, nói "design specs",
  "component documentation", hoặc "design-to-code handoff".
```

**❌ Tệ:**
```yaml
# Quá mơ hồ
description: Giúp làm dự án.

# Thiếu trigger
description: Tạo hệ thống tài liệu đa trang phức tạp.

# Quá kỹ thuật, không có trigger người dùng
description: Triển khai mô hình entity Project với quan hệ phân cấp.
```

### Bảo mật Frontmatter

**CẤM:**
- Dấu ngoặc nhọn XML `< >` (có thể inject vào system prompt)
- Tên skill chứa "claude" hoặc "anthropic" (reserved)
- Thực thi code trong YAML

**Lý do:** Frontmatter xuất hiện trong system prompt. Nội dung độc hại có thể inject hướng dẫn.

---

## 3 Loại Skill Phổ Biến

| Loại | Mục đích | Ví dụ |
|------|---------|-------|
| **Tạo tài liệu/sản phẩm** | Output nhất quán, chất lượng cao | frontend-design, docx, pptx |
| **Tự động hóa workflow** | Quy trình nhiều bước có phương pháp | skill-creator, project-setup |
| **Nâng cấp MCP** | Thêm kiến thức cho tool access | sentry-code-review |

### OpenClaw tương đương

| Anthropic | OpenClaw |
|-----------|---------|
| MCP server | Tool (read, write, exec, browser...) |
| MCP + Skill | Tool + Skill (kiến thức cách dùng tool) |
| Skill folder upload | Thư mục trong `skills/` hoặc `~/.openclaw/skills/` |
| skill-creator | Skill `skill-creator` trên ClawHub |

---

## Viết Hướng Dẫn Tốt

### Cấu trúc đề xuất cho SKILL.md body

```markdown
# Tên Skill

## Hướng dẫn

### Bước 1: [Tên bước]
Giải thích rõ ràng.

Ví dụ:
\`\`\`bash
python scripts/fetch_data.py --project-id PROJECT_ID
\`\`\`
Kết quả mong đợi: [mô tả thành công]

### Bước 2: ...

## Ví dụ

### Ví dụ 1: [Tình huống phổ biến]
Người dùng nói: "Tạo chiến dịch marketing mới"
Hành động:
1. Lấy danh sách chiến dịch hiện có
2. Tạo chiến dịch mới với tham số
Kết quả: Chiến dịch được tạo với link xác nhận

## Xử lý lỗi
Lỗi: [Thông báo lỗi phổ biến]
Nguyên nhân: [Tại sao]
Giải pháp: [Cách sửa]
```

### Nguyên tắc viết

| ✅ Nên | ❌ Không nên |
|--------|-------------|
| Cụ thể, có thể hành động | Mơ hồ ("xác thực dữ liệu") |
| Bullet points, numbered lists | Đoạn văn dài |
| Hướng dẫn quan trọng ĐẦU file | Chôn hướng dẫn quan trọng ở cuối |
| Lặp lại điểm quan trọng | Nói 1 lần rồi hy vọng agent nhớ |
| SKILL.md < 5,000 từ | Nhồi mọi thứ vào 1 file |

### Khi agent "lười" không làm theo

Thêm vào SKILL.md:
```markdown
## Ghi chú hiệu suất
- Làm kỹ, không vội
- Chất lượng quan trọng hơn tốc độ
- KHÔNG bỏ qua bước xác thực
```

> **Mẹo:** Thêm vào prompt người dùng hiệu quả hơn thêm vào SKILL.md.

---

## Tiêu Chí Thành Công

Đặt trước khi build, đo sau khi xong:

**Định lượng:**
- Skill kích hoạt đúng 90% truy vấn liên quan
- Hoàn thành workflow trong X lần gọi công cụ
- 0 lỗi API mỗi workflow

**Định tính:**
- Người dùng không cần nhắc bước tiếp theo
- Workflow hoàn thành không cần sửa
- Kết quả nhất quán giữa các phiên
