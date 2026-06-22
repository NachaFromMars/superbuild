# Quản lý bộ nhớ đa tầng (Memory Patterns)

Agent mất trí nhớ sau mỗi session. File là trí nhớ duy nhất bền vững.

---

## Kiến trúc bộ nhớ OpenClaw

```
MEMORY.md              ← Tầng 1: Trí nhớ dài hạn (luôn nạp ở session chính)
├── memory/
│   ├── YYYY-MM-DD.md  ← Tầng 2: Nhật ký hàng ngày (nạp khi cần)
│   ├── active-missions.json  ← Tầng 2: Nhiệm vụ đang chạy
│   └── heartbeat-state.json  ← Tầng 2: Trạng thái kiểm tra định kỳ
├── MEMORY-TASK-*.md   ← Tầng 2: Tiến độ dự án cụ thể
└── MEMORY-FORGE-*.md  ← Tầng 2: Nhật ký forge từng step
```

---

## 3 tầng bộ nhớ

### Tầng 1: NÓNG (Hot) — Luôn nạp
- **MEMORY.md** — trí nhớ dài hạn tinh lọc
- **AGENTS.md** — quy ước hành vi
- **SOUL.md** — tính cách
- **USER.md** — thông tin người dùng

**Quy tắc:** Tổng tầng 1 < 500 dòng. Nếu quá → tinh lọc, chuyển chi tiết xuống tầng 2.

### Tầng 2: ẤM (Warm) — Nạp khi cần
- `memory/YYYY-MM-DD.md` — nhật ký gần đây
- `MEMORY-TASK-*.md` — tiến độ dự án
- `MEMORY-FORGE-*.md` — nhật ký forge

**Quy tắc:** Dùng `memory_search` để tìm, `memory_get` để đọc đúng đoạn cần. Không đọc toàn bộ.

### Tầng 3: LẠNH (Cold) — Lưu trữ
- Nhật ký cũ hơn 30 ngày
- Dự án đã hoàn thành
- Thông tin lịch sử ít dùng

**Quy tắc:** Nén hoặc archive. Có thể tìm lại qua `memory_search` nhưng không nạp thường xuyên.

---

## Quy trình cập nhật bộ nhớ

### Sau mỗi session quan trọng
1. Cập nhật `memory/YYYY-MM-DD.md` — ghi sự kiện chính
2. Nếu đang forge: cập nhật `MEMORY-FORGE-*.md`
3. Nếu có quyết định quan trọng: cập nhật `MEMORY.md`

### Định kỳ (mỗi vài ngày)
1. Đọc nhật ký gần đây → tinh lọc vào `MEMORY.md`
2. Xóa thông tin lỗi thời khỏi `MEMORY.md`
3. Archive nhật ký cũ

### Khi bắt đầu dự án mới
1. Tạo `MEMORY-TASK-[tên].md` — theo dõi tiến độ
2. Tạo `MEMORY-FORGE-[tên].md` — nhật ký từng step
3. Thêm tóm tắt vào `MEMORY.md`

---

## So sánh nguồn gốc

| Claude Code | OpenClaw | Ghi chú |
|------------|---------|---------|
| `CLAUDE.md` (gốc) | `MEMORY.md` + `AGENTS.md` | Tách hành vi vs kiến thức |
| `~/.claude/CLAUDE.md` (toàn cục) | `SOUL.md` + `USER.md` | Tính cách + info người dùng |
| `.claude/agent-memory/` (bộ nhớ agent) | `MEMORY-TASK-*.md` | Bộ nhớ theo dự án |
| Ancestor loading | Tầng 1 luôn nạp | Tương đương |
| Descendant loading | Tầng 2 nạp khi cần | Tương đương |
| `/memory` (lệnh) | `memory_search` + `memory_get` | Công cụ recall |

---

## Mẹo thực tế

### Viết MEMORY.md hiệu quả
- **Ghi quyết định**, không ghi quá trình ("Chọn PostgreSQL vì X" chứ không "Đã thảo luận MySQL vs PostgreSQL trong 30 phút")
- **Ghi trạng thái**, không ghi lịch sử ("Ch100 DONE" chứ không "Ch98 done ngày A, Ch99 done ngày B, Ch100 done ngày C")
- **Ghi bài học**, không ghi lỗi ("Cần check output size > 0 trước khi tiếp" chứ không "Lỗi vì output rỗng ngày X")

### Tên file nhất quán
```
MEMORY-TASK-SUPERBUILD.md     ← Tiến độ dự án SuperBuild
MEMORY-FORGE-SUPERBUILD.md    ← Nhật ký forge SuperBuild
MEMORY-TASK-CITADEL.md        ← Tiến độ dự án Citadel
```

### Khi bộ nhớ phình
Dấu hiệu: `MEMORY.md` > 300 dòng, hoặc mất > 10 giây để đọc.
Giải pháp:
1. Tách phần dự án cũ đã xong → archive
2. Gộp điểm số rải rác thành bảng tóm tắt
3. Xóa TODO đã hoàn thành

---

## Checklist bộ nhớ

- [ ] MEMORY.md < 300 dòng?
- [ ] Mỗi dự án đang chạy có MEMORY-TASK riêng?
- [ ] Nhật ký hôm nay đã cập nhật?
- [ ] Quyết định quan trọng đã ghi?
- [ ] Thông tin lỗi thời đã xóa?
