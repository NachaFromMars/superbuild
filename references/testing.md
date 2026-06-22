# Kiểm Thử Skill — Testing Framework

Nguồn: Anthropic "The Complete Guide to Building Skills for Claude" Ch.3
Đã dịch, bổ sung quy trình OpenClaw.

---

## 3 Cấp Kiểm Thử

| Cấp | Phương thức | Khi nào dùng |
|-----|------------|-------------|
| **Thủ công** | Chạy truy vấn trực tiếp, quan sát | Phát triển nhanh, mọi skill |
| **Kịch bản** | Danh sách test case, chạy tuần tự | Skill quan trọng, cần lặp lại |
| **Tự động** | Script/cron chạy đánh giá hàng loạt | Skill production, nhiều người dùng |

> **Mẹo quan trọng:** Lặp trên 1 task khó cho đến khi thành công, RỒI MỚI trích xuất thành skill. Hiệu quả hơn test rộng ngay từ đầu.

---

## 1. Kiểm Thử Kích Hoạt (Trigger Tests)

**Mục tiêu:** Skill nạp đúng lúc, không nạp sai lúc.

### Test cases cần có

**Phải kích hoạt ✅:**
```
- Truy vấn rõ ràng: "SuperBuild dự án chat"
- Diễn đạt khác: "Dùng RPI để build tính năng mới"
- Viết tắt: "RPI cho real-time chat"
- Ngữ cảnh ngầm: "Tao cần quy trình agentic cho dự án"
```

**KHÔNG được kích hoạt ❌:**
```
- Không liên quan: "Thời tiết hôm nay?"
- Gần nhưng khác: "Viết tiểu thuyết" (đó là Omni Forge)
- Mơ hồ: "Giúp tao code" (quá chung)
```

### Cách kiểm tra

**OpenClaw:** Hỏi trực tiếp:
> "Khi nào mày sẽ dùng skill superbuild?"

Agent sẽ trích dẫn description. Điều chỉnh nếu thiếu trigger.

### Sửa khi kích hoạt sai

| Vấn đề | Giải pháp |
|--------|----------|
| Không kích hoạt | Thêm trigger phrases vào description |
| Kích hoạt quá nhiều | Thu hẹp scope, thêm negative triggers |
| Kích hoạt nhầm skill | Phân biệt rõ scope giữa các skill |

**Ví dụ negative trigger:**
```yaml
description: Phân tích dữ liệu CSV nâng cao. Dùng cho
  mô hình thống kê, hồi quy, clustering. KHÔNG DÙNG cho
  khám phá dữ liệu đơn giản (dùng data-viz thay thế).
```

---

## 2. Kiểm Thử Chức Năng (Functional Tests)

**Mục tiêu:** Skill tạo output đúng.

### Template test case

```
Test: [Tên test]
Cho: [Điều kiện đầu vào]
Khi: [Skill thực thi]
Thì:
  - [Output mong đợi 1]
  - [Output mong đợi 2]
  - [Không có lỗi API]
```

### Ví dụ cho SuperBuild RPI

```
Test: RPI Research Phase
Cho: Yêu cầu "SuperBuild real-time-chat cho Citadel"
Khi: Agent chạy bước Nghiên cứu
Thì:
  - File rpi/real-time-chat/REQUEST.md được tạo
  - File rpi/real-time-chat/RESEARCH.md được tạo
  - RESEARCH.md có 5 phần: yêu cầu, kỹ thuật, rủi ro, ước tính, phán quyết
  - Phán quyết là ĐI hoặc DỪNG (không bỏ trống)
```

```
Test: RPI Quality Gate
Cho: RESEARCH.md có phán quyết DỪNG
Khi: Agent đọc phán quyết
Thì:
  - KHÔNG tạo PLAN.md
  - Báo người dùng: "Dự án không khả thi vì [lý do]"
  - Không spawn sub-agent triển khai
```

### Kiểm thử Edge Cases

- Input rỗng hoặc mơ hồ
- File đã tồn tại (ghi đè hay skip?)
- Sub-agent timeout hoặc fail
- Context phình quá 80%

---

## 3. So Sánh Hiệu Suất (Performance Comparison)

**Mục tiêu:** Chứng minh skill cải thiện kết quả so với baseline.

### Template so sánh

```
                    Không có skill    Có skill
─────────────────────────────────────────────
Tin nhắn qua lại    15               2-3
Lỗi API             3                0
Token tiêu thụ      12,000           6,000
Thời gian           25 phút          10 phút
Kết quả nhất quán   Không            Có
```

### Cách đo trong OpenClaw

1. **Token:** Dùng `session_status` trước và sau
2. **Lỗi:** Đếm retry/fail trong output
3. **Thời gian:** Ghi timestamp bắt đầu/kết thúc
4. **Nhất quán:** Chạy cùng yêu cầu 3-5 lần, so sánh output

---

## Quy Trình Test Đề Xuất cho OpenClaw Skill

### Trước khi publish

```
1. ✅ Trigger test (5 truy vấn ĐÚNG + 5 truy vấn SAI)
2. ✅ Functional test (2-3 test case chính)
3. ✅ Edge case test (input lỗi, timeout, file trùng)
4. ✅ Performance so sánh (ít nhất 1 baseline)
```

### Sau khi publish

```
1. Thu thập phản hồi người dùng
2. Monitor under/over-triggering
3. Cập nhật description + instructions
4. Tăng version trong metadata
```

---

## Checklist Nhanh

- [ ] Skill kích hoạt đúng ≥90% truy vấn liên quan
- [ ] Skill KHÔNG kích hoạt cho truy vấn không liên quan
- [ ] Workflow hoàn thành không cần sửa từ người dùng
- [ ] 0 lỗi API trong happy path
- [ ] Kết quả nhất quán qua 3+ lần chạy
- [ ] Edge cases xử lý đúng (không crash, có thông báo)
