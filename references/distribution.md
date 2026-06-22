# Phân Phối & Đóng Gói Skill — Distribution Guide

Nguồn: Anthropic "The Complete Guide to Building Skills for Claude" Ch.4
Đã dịch, bổ sung quy trình OpenClaw/ClawHub.

---

## Đóng Gói Skill

### Cấu trúc ZIP chuẩn

```
ten-skill.zip
└── ten-skill/
    ├── SKILL.md
    ├── scripts/
    │   └── *.py, *.sh, *.mjs
    ├── references/
    │   └── *.md
    └── assets/
        └── templates, icons...
```

### Quy tắc đóng gói

- ZIP phải chứa 1 thư mục gốc duy nhất
- Tên thư mục gốc = tên skill (kebab-case)
- SKILL.md nằm trong thư mục gốc (không ở ngoài)
- Không kèm file tạm: `.git/`, `node_modules/`, `__pycache__/`
- Không kèm file nhạy cảm: `.env`, API keys, tokens
- Không kèm `README.md` TRONG thư mục skill

### PowerShell đóng gói nhanh

```powershell
# Đóng gói skill (loại trừ file tạm)
$skillDir = "~/.openclaw/workspace/skills/ten-skill"
$exclude = @(".git", "node_modules", "__pycache__", ".env")
Compress-Archive -Path "$skillDir\*" -DestinationPath "$env:TEMP\ten-skill.zip" -Force
```

---

## Kênh Phân Phối

### 1. OpenClaw — Cài đặt trực tiếp

```
Giải nén vào:
~/.openclaw/workspace/skills/ten-skill/    ← skill workspace
~/.openclaw/skills/ten-skill/              ← skill hệ thống
```

Agent tự phát hiện skill qua danh sách `<available_skills>`.

### 2. ClawHub — Marketplace

```bash
# Publish
clawhub publish ten-skill/

# Install
clawhub install ten-skill

# Update
clawhub update ten-skill
```

Tham khảo skill `clawhub` để biết chi tiết CLI.

### 3. GitHub — Open Source

```
your-repo/
├── README.md          ← Cho người đọc (human-facing)
├── ten-skill/
│   ├── SKILL.md       ← Cho agent (agent-facing)
│   ├── scripts/
│   └── references/
└── examples/
    └── demo.md
```

> **Quan trọng:** README.md ở cấp REPO (cho người), SKILL.md ở cấp SKILL (cho agent). Không trộn lẫn.

### 4. Telegram/Message — Gửi trực tiếp

```
1. Đóng gói ZIP
2. message(action=send, filePath=..., caption="Skill package...")
3. Người nhận giải nén vào skills/
```

---

## Viết README Cho GitHub

### Template README

```markdown
# Tên Skill

Mô tả ngắn 1-2 câu.

## Cài đặt

### OpenClaw
\`\`\`bash
git clone https://github.com/you/ten-skill.git
cp -r ten-skill ~/.openclaw/workspace/skills/
\`\`\`

### ClawHub
\`\`\`bash
clawhub install ten-skill
\`\`\`

## Sử dụng

Nói với agent:
> "Dùng [tên skill] để [mô tả task]"

## Ví dụ

[Screenshot hoặc output mẫu]

## Yêu cầu

- OpenClaw v1.x+
- Python 3.10+ (nếu có scripts)

## License

MIT
```

---

## Positioning — Cách Mô Tả Skill

### ✅ Tập trung vào kết quả

```
"Skill SuperBuild giúp team setup workspace dự án hoàn chỉnh
trong vài phút — bao gồm nghiên cứu, kế hoạch, và pipeline
triển khai — thay vì mất hàng giờ setup thủ công."
```

### ❌ Tập trung vào kỹ thuật

```
"Skill SuperBuild là thư mục chứa YAML frontmatter và Markdown
instructions gọi các tool sessions_spawn và exec."
```

### Kể câu chuyện MCP + Skill (Anthropic analogy)

```
Tool cung cấp NHÀN BẾP chuyên nghiệp: truy cập công cụ,
nguyên liệu, thiết bị.

Skill cung cấp CÔNG THỨC: hướng dẫn từng bước để tạo
sản phẩm có giá trị.

Kết hợp = người dùng hoàn thành task phức tạp mà không cần
tự tìm hiểu mọi bước.
```

---

## Versioning

### Quy tắc version

```
MAJOR.MINOR.PATCH
1.0.0 → 1.1.0 → 1.1.1 → 2.0.0
```

| Thay đổi | Bump |
|----------|------|
| Sửa lỗi nhỏ, cập nhật text | PATCH (1.0.x) |
| Thêm tính năng, thêm reference | MINOR (1.x.0) |
| Breaking change, đổi cấu trúc | MAJOR (x.0.0) |

### Ghi trong metadata

```yaml
---
name: superbuild
description: ...
metadata:
  author: Nacharium
  version: 1.1.0
---
```

---

## Checklist Trước Khi Publish

- [ ] Tên thư mục kebab-case
- [ ] SKILL.md tồn tại (đúng tên)
- [ ] YAML frontmatter có `---` delimiters
- [ ] `name` kebab-case, không viết hoa
- [ ] `description` có WHAT + WHEN
- [ ] Không có `< >` trong YAML
- [ ] Không có API keys, tokens, .env
- [ ] Không có README.md trong thư mục skill
- [ ] Hướng dẫn rõ ràng, có ví dụ
- [ ] Xử lý lỗi có hướng dẫn
- [ ] references/ được link đúng
- [ ] Đã test trigger (5 đúng + 5 sai)
- [ ] Đã test chức năng (2-3 happy path)
- [ ] Version ghi trong metadata
