# Mẹo thực chiến (Battle-Tested Tips)

Kinh nghiệm từ đội ngũ Claude Code (Boris Cherny, Thariq), tổng hợp và dịch sang ngữ cảnh OpenClaw.

---

## Quy trình hàng ngày

### Bắt đầu bằng kế hoạch, không bắt đầu bằng code
- Luôn lên kế hoạch trước khi triển khai
- Kế hoạch = rẻ (ít token). Code sai = đắt (nhiều token + thời gian sửa)
- Dùng quy trình RPI: Nghiên cứu → Kế hoạch → Triển khai

### Lưu thay đổi thường xuyên
- Commit ngay khi 1 nhiệm vụ hoàn thành
- Đừng để 10 file thay đổi chưa commit
- Commit = checkpoint, có thể quay lại nếu hỏng

### Đơn giản tốt hơn phức tạp
- Với việc nhỏ (< 30 phút): làm trực tiếp, không cần workflow
- Với việc vừa (30 phút - 2 giờ): dùng sub-agent đơn lẻ
- Với việc lớn (> 2 giờ): dùng quy trình RPI đầy đủ
- **Không thiết kế workflow cho việc mà 1 lệnh exec giải quyết được**

---

## Thiết kế tác tử

### Dùng model khác để kiểm tra
- Viết bằng Opus → review bằng Sonnet (hoặc ngược lại)
- Model khác nhau bắt lỗi khác nhau
- Rẻ hơn nhiều so với để 1 model tự review chính mình

### Tác tử chuyên biệt > tác tử đa năng
- "Tác tử phân tích CRM" tốt hơn "Tác tử backend"
- Ngữ cảnh hẹp = chất lượng cao hơn
- Gắn skill cụ thể vào agent thay vì cho đọc hết

### Nén thủ công ở 50%
- Đừng đợi hệ thống tự nén
- Ở 50% ngữ cảnh → ghi checkpoint → nén
- Agent sau khi nén = tươi mới, chất lượng cao lại

---

## Tổ chức dự án

### File hướng dẫn ngắn gọn
- MEMORY.md: tinh lọc, chỉ giữ cái quan trọng
- SKILL.md: bảng điều hướng, không viết dài
- Task description: 10-20 dòng, có context + output + ràng buộc

### Cấu trúc thư mục rõ ràng
```
dự-án/
├── rpi/           ← Quy trình RPI (mỗi tính năng 1 folder)
├── references/    ← Tài liệu tham khảo
├── MEMORY-TASK.md ← Tiến độ
└── MEMORY-FORGE.md ← Nhật ký
```

### Giao tiếp qua file, không qua ngữ cảnh
- Agent 1 xong → ghi file → Agent 2 đọc file
- Không truyền data lớn qua lệnh spawn
- File = nguồn sự thật duy nhất (single source of truth)

---

## Gỡ lỗi

### Khi agent "ngu đi"
1. Kiểm tra mức ngữ cảnh — có thể đã > 70%
2. Nén ngay, ghi checkpoint trước
3. Nếu vẫn tệ: spawn agent mới với ngữ cảnh sạch

### Khi output sai format
1. Thêm ví dụ cụ thể vào task description
2. Ví dụ > mô tả trừu tượng
3. "Output PHẢI theo format: ## Kết luận\n**Phán quyết:** ĐI/DỪNG"

### Khi sub-agent không trả kết quả
1. Kiểm tra `subagents list`
2. Nếu chết → respawn với cùng task
3. Nếu chạy quá lâu (> 20 phút) → kill + respawn
4. Giới hạn `runTimeoutSeconds` khi spawn

---

## Mẹo OpenClaw riêng

### Heartbeat = cơ hội bảo trì
- Dùng heartbeat để kiểm tra sub-agent, dọn memory, commit code
- Không chỉ reply HEARTBEAT_OK — tận dụng mỗi lượt

### Cron cho timing chính xác
- Nhắc nhở, kiểm tra định kỳ → dùng cron
- Nhiệm vụ cần ngữ cảnh session → dùng heartbeat
- Đừng tạo 10 cron job khi 1 heartbeat checklist đủ

### Sub-agent auto-announce
- Sub-agent tự báo khi xong — không cần poll
- Chỉ check `subagents list` khi cần can thiệp
- Spawn → quên đi → đợi announce

---

## Quy tắc vàng

1. **Kế hoạch trước, code sau**
2. **Commit thường xuyên**
3. **Nén ở 50%, không đợi 90%**
4. **1 agent = 1 việc**
5. **File > "nhớ trong đầu"**
6. **Đơn giản > phức tạp**
7. **Ví dụ cụ thể > mô tả trừu tượng**
