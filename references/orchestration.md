# Mẫu phối hợp đa tác tử (Orchestration Patterns)

Khi hệ thống có nhiều tác tử (agent), cần quy ước rõ: ai làm gì, ai gọi ai, dữ liệu chảy thế nào.

---

## Mẫu cốt lõi: Lệnh → Tác tử → Kỹ năng

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  LỆNH        │────▶│  TÁC TỬ      │────▶│  KỸ NĂNG     │
│  (Entry)     │     │  (Worker)    │     │  (Knowledge) │
└──────────────┘     └──────────────┘     └──────────────┘
```

| Vai trò | Claude Code | OpenClaw tương đương |
|---------|-------------|---------------------|
| **Lệnh** (Command) — điểm vào, nhận yêu cầu người dùng | `.claude/commands/*.md` | Tin nhắn trực tiếp, lệnh cron, hoặc trigger trong SKILL.md |
| **Tác tử** (Agent) — thực hiện công việc cụ thể | `.claude/agents/*.md` | `sessions_spawn` với task mô tả rõ |
| **Kỹ năng** (Skill) — kiến thức chuyên môn, đọc khi cần | `.claude/skills/*/SKILL.md` | `skills/*/SKILL.md` (đọc qua `read`) |

---

## 2 kiểu kỹ năng

### Kỹ năng gắn sẵn (Agent Skill)
- Nạp vào tác tử **ngay khi khởi tạo** — tác tử "biết sẵn" kiến thức này
- OpenClaw: ghi nội dung kỹ năng trực tiếp vào `task` khi `sessions_spawn`

```
sessions_spawn(task="[Nội dung kỹ năng] + [Nhiệm vụ cụ thể]")
```

### Kỹ năng độc lập (Standalone Skill)
- Gọi riêng khi cần — không gắn vào tác tử nào
- OpenClaw: agent chính đọc SKILL.md rồi tự thực hiện

```
Đọc references/context-engineering.md → Áp dụng kiến thức → Không cần spawn
```

---

## Mẫu phối hợp thường gặp

### 1. Bộ điều phối (Orchestrator)
Một agent chính quản lý, spawn nhiều agent phụ tuần tự.

```
Agent chính (Orchestrator)
├── spawn Tác tử 1: Nghiên cứu → trả kết quả
├── spawn Tác tử 2: Lên kế hoạch (dựa trên kết quả 1) → trả kết quả
└── spawn Tác tử 3: Triển khai (dựa trên kế hoạch 2) → trả kết quả
```

**Quy tắc:** Tác tử phụ KHÔNG spawn tác tử khác. Chỉ bộ điều phối mới spawn.

**OpenClaw:**
```
Tin nhắn Nấng → Agent chính xử lý → sessions_spawn(task1)
→ Kết quả 1 → sessions_spawn(task2, context=kết_quả_1)
→ Kết quả 2 → sessions_spawn(task3, context=kết_quả_2)
→ Báo cáo cuối
```

### 2. Song song (Parallel Fan-out)
Spawn nhiều agent cùng lúc, gom kết quả.

```
Agent chính
├── spawn Tác tử A: Phân tích backend
├── spawn Tác tử B: Phân tích frontend  (đồng thời)
└── spawn Tác tử C: Phân tích database  (đồng thời)
→ Gom kết quả A + B + C → Tổng hợp
```

**OpenClaw:** 3 lệnh `sessions_spawn` liên tiếp, mỗi cái auto-announce khi xong.

### 3. Đường ống (Pipeline)
Đầu ra agent này = đầu vào agent sau. Tuần tự nghiêm ngặt.

```
Tác tử 1 (Thu thập dữ liệu)
→ output file
→ Tác tử 2 (Phân tích, đọc output 1)
→ output file
→ Tác tử 3 (Viết báo cáo, đọc output 2)
```

**Quy tắc:** Giao tiếp qua FILE, không qua ngữ cảnh. Mỗi agent đọc file đầu vào, ghi file đầu ra.

---

## Nguyên tắc phối hợp

1. **Trách nhiệm đơn** — mỗi tác tử chỉ làm 1 việc
2. **Bộ điều phối quản lý** — tác tử phụ không tự spawn nhau
3. **Giao tiếp qua file** — không truyền data lớn qua ngữ cảnh
4. **Mô tả rõ ràng** — task description phải cụ thể: "Phân tích file X, trả kết quả Y, lưu tại Z"
5. **Cổng chặn** — kiểm tra output mỗi step trước khi chuyển step sau

---

## Ví dụ thực tế OpenClaw

### Forge tiểu thuyết (đang dùng)
```
Agent chính (Main session)
├── spawn Agent viết (Opus 4.6): viết 1 beat → lưu file
├── Đọc output → spawn Agent biên tập: review beat
├── PASS → spawn Agent viết: beat tiếp
└── Lặp đến hết chương → Council thẩm định
```

### Phát triển tính năng (SuperBuild RPI)
```
Agent chính
├── spawn Agent nghiên cứu: phân tích khả thi → RESEARCH.md
├── ĐI → spawn Agent lên kế hoạch: viết PLAN.md
├── Duyệt → spawn Agent lập trình: triển khai từng giai đoạn
│   ├── Giai đoạn 1 → code review → PASS → Giai đoạn 2
│   └── ...
└── Hoàn thành → IMPLEMENT.md
```
