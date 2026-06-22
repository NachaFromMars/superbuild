---
name: superbuild
description: "Kiến thức nền tảng về thiết kế tác tử (agentic engineering) + quy trình phát triển RPI (Nghiên cứu → Kế hoạch → Triển khai) cho OpenClaw. Bao gồm: 5 mẫu thiết kế, testing framework, troubleshooting, YAML frontmatter guide, đóng gói/phân phối skill. Dùng khi: (1) thiết kế workflow mới, (2) spawn tác tử phụ, (3) quản lý ngữ cảnh/bộ nhớ, (4) bắt đầu tính năng mới cần quy trình chuẩn, (5) tối ưu gọi công cụ, (6) build/test/publish skill mới, (7) debug skill không kích hoạt đúng. Triggers: superbuild, RPI, agentic, thiết kế tác tử, workflow mới, context engineering, agent design, build skill, test skill, publish skill, skill không hoạt động."
---

# SuperBuild v1.1 — Kiến Thức Tác Tử & Quy Trình Phát Triển

Skill gộp 3 vai trò:
- **Kho kiến thức** — tra cứu khi cần, không nạp hết cùng lúc
- **Quy trình hành động** — RPI (Nghiên cứu → Kế hoạch → Triển khai)
- **Hướng dẫn skill** — build, test, debug, đóng gói, phân phối skill

Nguồn gốc:
- claude-code-best-practice (shanraisshan, Mar 2026)
- The Complete Guide to Building Skills for Claude (Anthropic, Jan 2026)
- Đã dịch và tối ưu cho OpenClaw native.

---

## Bảng điều hướng — Gặp gì đọc nấy

### Kiến thức tác tử (Agentic Engineering)

| Tình huống | Đọc file |
|-----------|----------|
| Ngữ cảnh (context) sắp đầy, agent bắt đầu "ngu" | `references/context-engineering.md` |
| Cần phối hợp nhiều tác tử, lên workflow mới | `references/orchestration.md` |
| Spawn tác tử phụ, chọn model, phân quyền công cụ | `references/agent-design.md` |
| Bộ nhớ phình, cần tổ chức lại memory | `references/memory-patterns.md` |
| Muốn gọi công cụ ít token hơn, nhanh hơn | `references/tool-optimization.md` |
| Cần mẹo nhanh, kinh nghiệm thực chiến | `references/tips.md` |

### Build & Ship Skill (mới v1.1)

| Tình huống | Đọc file |
|-----------|----------|
| Tạo skill mới, cấu trúc thư mục, viết YAML frontmatter | `references/skill-structure.md` |
| Chọn design pattern cho skill/workflow | `references/design-patterns.md` |
| Test skill: trigger, chức năng, hiệu suất | `references/testing.md` |
| Skill lỗi: không kích hoạt, trigger sai, agent không tuân theo | `references/troubleshooting.md` |
| Đóng gói ZIP, publish GitHub/ClawHub, viết README | `references/distribution.md` |

---

## Quy trình RPI — Phát triển tính năng có cấu trúc

Khi bắt đầu tính năng/dự án mới, dùng quy trình 3 bước:

```
NGHIÊN CỨU (Research) → KẾ HOẠCH (Plan) → TRIỂN KHAI (Implement)
```

Chi tiết: `workflows/rpi.md`

### Kích hoạt
- "RPI cho [tên tính năng]"
- "Nghiên cứu + lên kế hoạch + triển khai [X]"
- "SuperBuild [tên dự án]"

### Đầu ra
Mỗi tính năng tạo thư mục riêng:
```
rpi/{tên-tính-năng}/
├── REQUEST.md    ← Mô tả ban đầu
├── RESEARCH.md   ← Phân tích khả thi (ĐI/DỪNG)
├── PLAN.md       ← Kế hoạch chia giai đoạn
└── IMPLEMENT.md  ← Nhật ký triển khai
```

Mẫu sẵn: `workflows/templates/`

### Handoff → OpenBuild
Khi RPI Plan xong và verdict = **ĐI**:
1. Chuyển sang skill **OpenBuild** (`/intake`)
2. Dùng `RESEARCH.md` + `PLAN.md` làm input cho PRD
3. OpenBuild tiếp quản: Scope → Build → Verify → Ship → Retro

```
SuperBuild (nghĩ)  ──RESEARCH.md + PLAN.md──►  OpenBuild (làm)
```

---

## Nguyên tắc cốt lõi

1. **Nạp dần** (progressive disclosure) — 3 tầng: frontmatter → SKILL.md body → references/
2. **Tác tử chuyên biệt** — mỗi agent 1 việc, không dùng agent "đa năng"
3. **Cổng chặn** (quality gate) — mỗi bước phải thẩm định trước khi qua bước sau
4. **Chia nhỏ** — mỗi nhiệm vụ hoàn thành trong < 50% ngữ cảnh
5. **Ghi chép** — cập nhật memory sau mỗi bước, không "nhớ trong đầu"

---

## Quick Checklist — Trước Khi Ship Skill

```
□ Thư mục kebab-case, SKILL.md đúng tên
□ YAML có --- delimiters, name + description
□ Description có WHAT + WHEN + trigger phrases
□ Không < > trong YAML, không API keys trong file
□ Hướng dẫn rõ ràng, có ví dụ, có xử lý lỗi
□ SKILL.md < 5,000 từ (chi tiết sang references/)
□ Test trigger: 5 đúng + 5 sai
□ Test chức năng: 2-3 happy path
□ Version ghi trong metadata
```

Chi tiết: `references/distribution.md` (Checklist đầy đủ)

---

## 5 Mẫu Thiết Kế

| # | Tên | Khi dùng | Chi tiết |
|---|-----|---------|----------|
| 1 | Tuần tự | Bước A → B → C cố định | `references/design-patterns.md` |
| 2 | Đa nguồn | Phối hợp nhiều tool/API | ↑ |
| 3 | Tinh chỉnh lặp | Output cải thiện qua nhiều vòng | ↑ |
| 4 | Chọn theo ngữ cảnh | Cùng việc, tool khác tùy context | ↑ |
| 5 | Chuyên môn | Cần kiến thức lĩnh vực (pháp lý, tài chính...) | ↑ |

---

## Ghi chú
- Nguồn 1: shanraisshan/claude-code-best-practice (Mar 2026)
- Nguồn 2: Anthropic "The Complete Guide to Building Skills for Claude" (Jan 2026)
- Đã dịch và tối ưu cho OpenClaw (sessions_spawn, cron, memory system)
- Phiên bản: 1.1.0
