# Quản lý ngữ cảnh (Context Engineering)

Ngữ cảnh (context window) là tài nguyên hữu hạn. Agent hoạt động tốt nhất khi ngữ cảnh gọn, tập trung. Khi ngữ cảnh phình, chất lượng suy giảm.

---

## Vùng nguy hiểm

| Mức sử dụng | Trạng thái | Hành động |
|-------------|-----------|----------|
| 0-30% | 🟢 Thoải mái | Làm việc bình thường |
| 30-50% | 🟡 Cảnh giác | Cân nhắc nén (compact) thủ công |
| 50-70% | 🟠 Nguy hiểm | **NÊN nén ngay** — chất lượng bắt đầu giảm |
| 70-100% | 🔴 Vùng ngu | Agent mất khả năng theo dõi chỉ dẫn, bỏ sót chi tiết, lặp lại |

**Quy tắc vàng:** Nén thủ công ở mốc **50%** — đừng đợi hệ thống tự nén ở 90%.

---

## Nguyên tắc giữ ngữ cảnh gọn

### 1. File hướng dẫn gốc phải ngắn
- MEMORY.md, AGENTS.md, SKILL.md: **tối đa 150 dòng** (lý tưởng 60 dòng)
- Dài hơn = agent bỏ sót phần cuối, không tuân thủ đều
- Dùng bảng điều hướng: ghi tóm tắt ở file gốc, chi tiết ở file con

### 2. Nạp dần (progressive disclosure)
- **Không đọc hết tất cả file cùng lúc**
- Đọc file nào khi cần file đó — theo bảng điều hướng trong SKILL.md
- Ví dụ OpenClaw: `references/` chỉ đọc khi gặp tình huống tương ứng

### 3. Chia nhỏ nhiệm vụ
- Mỗi nhiệm vụ phải **hoàn thành trong < 50% ngữ cảnh**
- Nếu nhiệm vụ lớn hơn → tách thành sub-agent (`sessions_spawn`)
- Sub-agent có ngữ cảnh riêng, không chiếm ngữ cảnh chính

### 4. Kết quả trung gian = ghi file
- Đừng giữ kết quả dài trong đầu (ngữ cảnh)
- Ghi ra file → đọc lại khi cần
- Ví dụ: nghiên cứu xong → ghi RESEARCH.md, xóa khỏi ngữ cảnh

---

## Kỹ thuật nén hiệu quả

### Nén có hướng dẫn
Khi nén, chỉ rõ cần giữ gì:
- "Nén nhưng giữ lại: quyết định thiết kế, lỗi đã gặp, step hiện tại"
- Không nén mù → mất context quan trọng

### Ghi trước khi nén
Trước khi nén:
1. Cập nhật memory task (step hiện tại, trạng thái)
2. Cập nhật memory forge (kết quả step vừa xong)
3. Ghi checkpoint file nếu đang giữa nhiệm vụ

---

## Áp dụng cho OpenClaw

| Khái niệm gốc (Claude Code) | Tương đương OpenClaw |
|------------------------------|---------------------|
| CLAUDE.md (file hướng dẫn gốc) | AGENTS.md + SOUL.md + MEMORY.md |
| /compact (lệnh nén) | Tự động sau ~80% hoặc thủ công qua restart |
| Ancestor loading (nạp file tổ tiên) | MEMORY.md luôn nạp ở session chính |
| Descendant loading (nạp file con) | memory/*.md chỉ nạp khi đọc |
| Context window | Session context — tương tự, giới hạn bởi model |

### Mẹo thực tế cho OpenClaw
- Session chính (chat trực tiếp): giữ gọn, delegate task nặng cho sub-agent
- Sub-agent: mỗi cái có ngữ cảnh riêng ~200K token → tận dụng
- Cron job: chạy riêng biệt, không chia sẻ ngữ cảnh với session chính
- Memory file: thay thế cho "ghi nhớ trong đầu" — bền vững qua session

---

## Checklist nhanh

- [ ] File hướng dẫn < 150 dòng?
- [ ] Đang ở < 50% ngữ cảnh?
- [ ] Nhiệm vụ hiện tại đủ nhỏ để hoàn thành trong 1 session?
- [ ] Kết quả trung gian đã ghi file chưa?
- [ ] Cần nén? → Ghi memory trước, nén sau
