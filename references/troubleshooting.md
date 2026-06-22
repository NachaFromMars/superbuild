# Xử Lý Sự Cố — Troubleshooting Guide

Nguồn: Anthropic "The Complete Guide to Building Skills for Claude" Ch.5 (Patterns & Troubleshooting)
Đã dịch, bổ sung lỗi thường gặp OpenClaw.

---

## 1. Skill Không Upload Được

### Lỗi: "Could not find SKILL.md"

**Nguyên nhân:** File không đặt tên đúng.

**Sửa:**
- Đổi tên chính xác `SKILL.md` (phân biệt hoa thường)
- Kiểm tra: `ls -la` hoặc `Get-ChildItem` phải hiện `SKILL.md`
- KHÔNG phải: `skill.md`, `SKILL.MD`, `Skill.md`

### Lỗi: "Invalid frontmatter"

**Nguyên nhân:** Sai format YAML.

**Lỗi phổ biến:**
```yaml
# ❌ Thiếu dấu phân cách ---
name: my-skill
description: Does things

# ❌ Ngoặc kép không đóng
name: my-skill
description: "Does things

# ❌ Thụt lề sai
name: my-skill
 description: Does things

# ✅ Đúng
---
name: my-skill
description: Does things
---
```

### Lỗi: "Invalid skill name"

**Nguyên nhân:** Tên có khoảng trắng hoặc viết hoa.

```yaml
# ❌ Sai
name: My Cool Skill
name: my_cool_skill
name: MyCoolSkill

# ✅ Đúng
name: my-cool-skill
```

---

## 2. Skill Không Kích Hoạt

**Triệu chứng:** Skill không bao giờ tự nạp.

### Checklist chẩn đoán

| Kiểm tra | Cách sửa |
|----------|---------|
| Description quá chung? | Thêm trigger phrases cụ thể |
| Thiếu từ khóa người dùng hay nói? | Liệt kê 3-5 cụm từ kích hoạt |
| Quá kỹ thuật? | Dùng ngôn ngữ bình thường |
| Có liên quan đến file type? | Ghi rõ loại file |

### Debug nhanh

Hỏi agent:
> "Khi nào mày sẽ dùng skill [tên-skill]?"

Agent trích dẫn description → thấy thiếu gì → sửa.

### Ví dụ sửa description

```yaml
# Trước (không kích hoạt)
description: Phân tích dự án phức tạp

# Sau (kích hoạt đúng)
description: Quy trình phát triển tính năng có cấu trúc.
  Dùng khi người dùng nói "SuperBuild", "RPI", "build tính năng",
  "quy trình agentic", hoặc bắt đầu dự án mới cần nghiên cứu
  trước khi code.
```

---

## 3. Skill Kích Hoạt Quá Nhiều

**Triệu chứng:** Skill nạp cho truy vấn không liên quan.

### Giải pháp

**1. Thêm negative trigger:**
```yaml
description: Phân tích dữ liệu CSV nâng cao cho mô hình thống kê.
  KHÔNG DÙNG cho khám phá dữ liệu đơn giản hoặc tạo biểu đồ cơ bản.
```

**2. Thu hẹp scope:**
```yaml
# ❌ Quá rộng
description: Xử lý tài liệu

# ✅ Cụ thể
description: Xử lý tài liệu PDF pháp lý cho review hợp đồng
```

**3. Phân biệt rõ với skill khác:**
```yaml
description: Quy trình RPI cho dự án CODE/TECH. Dùng "SuperBuild".
  Không dùng cho viết tiểu thuyết (dùng Omni Forge) hoặc
  quản lý dự án đang chạy (dùng OpenBuild).
```

---

## 4. Hướng Dẫn Không Được Tuân Theo

**Triệu chứng:** Skill nạp nhưng agent không làm đúng theo.

### Nguyên nhân phổ biến

| Nguyên nhân | Giải pháp |
|-------------|----------|
| **Hướng dẫn quá dài** | Giữ gọn, dùng bullet points, chuyển chi tiết sang references/ |
| **Hướng dẫn bị chôn** | Đặt điều quan trọng ĐẦU file, dùng header `## QUAN TRỌNG` |
| **Ngôn ngữ mơ hồ** | Thay "xác thực đúng cách" bằng "kiểm tra: tên không rỗng, ngày không quá khứ" |
| **Agent "lười"** | Thêm `## Ghi chú: Làm kỹ, không bỏ bước` |

### Ví dụ: Mơ hồ → Cụ thể

```markdown
# ❌ Mơ hồ
Đảm bảo xác thực mọi thứ đúng cách.

# ✅ Cụ thể
QUAN TRỌNG: Trước khi gọi create_project, xác thực:
- Tên dự án không rỗng
- Ít nhất 1 thành viên được gán
- Ngày bắt đầu không ở quá khứ
```

### Kỹ thuật nâng cao

> Với xác thực quan trọng, viết script thay vì dựa vào hướng dẫn ngôn ngữ.
> Code xác định; diễn giải ngôn ngữ thì không.

```markdown
Trước khi submit, chạy: `python scripts/validate.py --input data.json`
Nếu FAIL → sửa theo thông báo lỗi
Nếu PASS → tiếp tục
```

---

## 5. Context Phình / Chậm

**Triệu chứng:** Skill chậm, response kém chất lượng.

### Nguyên nhân

- SKILL.md quá lớn (>5,000 từ)
- Quá nhiều skill bật cùng lúc
- Mọi thứ inline thay vì progressive disclosure

### Giải pháp

**1. Tối ưu kích thước SKILL.md:**
```
TRƯỚC: SKILL.md 8,000 từ (mọi thứ trong 1 file)
SAU:   SKILL.md 2,000 từ + 4 files trong references/
```

**2. Giảm skill đồng thời:**
- Đánh giá nếu >20 skill bật cùng lúc
- Gom skill liên quan thành "pack"
- Tắt skill không dùng thường xuyên

**3. Progressive disclosure:**
```markdown
# Trong SKILL.md
Tham khảo `references/api-patterns.md` cho:
- Giới hạn tần suất
- Phân trang
- Mã lỗi
```

---

## 6. Lỗi Đặc Thù OpenClaw

### Sub-agent spawn rồi im lặng

**Nguyên nhân:** Agent hết context, bị compaction, hoặc crash.

**Sửa:**
- Kiểm tra `subagents list`
- Nếu agent running >20 phút mà không có output → kill + respawn
- Ghi checkpoint vào file trước khi spawn (để resume)

### Skill không xuất hiện trong danh sách

**Nguyên nhân:** Thư mục sai vị trí hoặc thiếu SKILL.md.

**Sửa:**
- Kiểm tra skill nằm trong `~/.openclaw/workspace/skills/` hoặc `~/.openclaw/skills/`
- Chạy `openclaw skills check` để audit
- Khởi động lại gateway nếu cần

### Memory conflict giữa skill và workspace

**Nguyên nhân:** Skill ghi file trùng tên với file workspace khác.

**Sửa:**
- Skill nên ghi vào thư mục riêng (vd: `rpi/ten-du-an/`)
- Không ghi trực tiếp vào gốc workspace
- Dùng prefix hoặc namespace cho file output

---

## Quick Reference — Lỗi → Sửa

| Lỗi | Sửa nhanh |
|-----|----------|
| Không upload được | Kiểm tra tên file `SKILL.md` + YAML `---` |
| Không kích hoạt | Sửa description, thêm trigger phrases |
| Kích hoạt quá nhiều | Thêm negative trigger, thu hẹp scope |
| Không tuân theo | Hướng dẫn ngắn hơn, cụ thể hơn, đầu file |
| Chậm/kém | Tách SKILL.md nhỏ hơn, dùng references/ |
| Sub-agent chết | Kill + respawn, ghi checkpoint |
| Không thấy skill | Kiểm tra đường dẫn + chạy `openclaw skills check` |
