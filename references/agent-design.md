# Thiết kế tác tử (Agent Design)

Tác tử tốt = chuyên biệt, giới hạn công cụ, mô tả rõ ràng. Tác tử tệ = "làm hết mọi thứ" với quyền truy cập toàn bộ.

---

## Nguyên tắc thiết kế

### 1. Chuyên biệt theo tính năng, không theo vai trò chung

❌ **Sai:** "Tác tử backend", "Tác tử frontend", "Tác tử QA"
✅ **Đúng:** "Tác tử CRM", "Tác tử xử lý thanh toán", "Tác tử viết chapter"

Lý do: Tác tử chung chung không có ngữ cảnh đủ sâu. Tác tử chuyên biệt biết chính xác domain của mình.

### 2. Giới hạn công cụ (Principle of Least Privilege)

Mỗi tác tử chỉ được dùng công cụ cần thiết:

| Loại tác tử | Công cụ nên cho | Công cụ KHÔNG cho |
|-------------|----------------|-------------------|
| Nghiên cứu | web_search, web_fetch, read | write, edit, exec |
| Lập trình | read, write, edit, exec | web_search (trừ khi cần) |
| Biên tập/Review | read | write, exec |
| Thu thập dữ liệu | web_fetch, read, write | exec, edit |

### 3. Mô tả nhiệm vụ phải cụ thể

❌ **Sai:** "Viết code cho tính năng chat"
✅ **Đúng:** "Viết WebSocket service cho real-time chat. Đọc schema từ packages/db/schema/chat.ts. Tạo file tại apps/api/src/services/chat-ws.ts. Dùng Redis pub/sub cho presence. Trả kết quả: file path + số endpoint + test coverage."

Bao gồm:
- **Đầu vào:** file/data nào cần đọc
- **Đầu ra:** file nào cần tạo, format kết quả
- **Ràng buộc:** patterns cần tuân theo, giới hạn

---

## Chọn model cho tác tử

| Độ phức tạp | Model đề xuất | Ví dụ |
|-------------|--------------|-------|
| Đơn giản — thu thập, format, copy | Haiku / Sonnet | Lấy dữ liệu API, format báo cáo |
| Trung bình — phân tích, review, viết | Sonnet / Opus | Code review, viết plan, phân tích |
| Phức tạp — sáng tạo, kiến trúc, viết sâu | Opus | Viết tiểu thuyết, thiết kế hệ thống, refactor lớn |

**Quy tắc ngón tay cái:** Dùng model nhẹ nhất có thể hoàn thành nhiệm vụ. Tiết kiệm token = nhiều lượt spawn hơn.

**Ngoại lệ:** Khi chất lượng quan trọng hơn tốc độ → luôn dùng Opus (ví dụ: forge tiểu thuyết).

---

## Cấu trúc task khi spawn (OpenClaw)

```python
sessions_spawn(
    task="""
    ## Nhiệm vụ: [Tên cụ thể]
    
    ## Ngữ cảnh
    - [Thông tin nền cần biết]
    - [File cần đọc trước khi bắt đầu]
    
    ## Yêu cầu
    1. [Bước 1 cụ thể]
    2. [Bước 2 cụ thể]
    3. [Bước 3 cụ thể]
    
    ## Đầu ra
    - Lưu kết quả tại: [đường dẫn file]
    - Format: [markdown/json/code]
    
    ## Ràng buộc
    - [Giới hạn 1]
    - [Giới hạn 2]
    """,
    model="claudible/claude-opus-4.6",  # hoặc model phù hợp
    label="ten-ngan-gon"
)
```

### Mẫu task tốt

**Nghiên cứu:**
```
Nhiệm vụ: Phân tích tính khả thi của tính năng OAuth2
Đọc: rpi/oauth2/REQUEST.md
Phân tích: độ phức tạp, thư viện cần dùng, rủi ro, thời gian ước tính
Kết luận: ĐI hoặc DỪNG với lý do
Lưu tại: rpi/oauth2/RESEARCH.md
```

**Lập trình:**
```
Nhiệm vụ: Triển khai giai đoạn 1 — Database schema cho CRM
Đọc kế hoạch: rpi/crm/PLAN.md (Giai đoạn 1)
Tham khảo patterns: packages/db/schema/ (các schema hiện có)
Tạo: packages/db/schema/crm.ts + migration file
Test: viết unit test cho schema validation
Lưu nhật ký: rpi/crm/IMPLEMENT.md
```

---

## Chống mẫu (Anti-patterns)

| Mẫu sai | Vấn đề | Mẫu đúng |
|---------|--------|----------|
| Agent spawn agent | Mất kiểm soát, ngữ cảnh rối | Chỉ orchestrator spawn |
| Task mô tả 1 dòng | Agent đoán sai, output lệch | Task 10-20 dòng có context + requirements |
| Cho agent truy cập hết | Rủi ro ghi đè file quan trọng | Giới hạn công cụ theo vai trò |
| Dùng Opus cho mọi việc | Tốn token, chậm | Match model với độ phức tạp |
| Không kiểm tra output | Lỗi lan sang step sau | Cổng chặn sau mỗi agent |

---

## Checklist thiết kế tác tử

- [ ] Tác tử có tên chuyên biệt (không phải "backend-agent")?
- [ ] Task description ≥ 10 dòng với context + requirements + output?
- [ ] Công cụ đã giới hạn đúng vai trò?
- [ ] Model phù hợp độ phức tạp?
- [ ] Có cổng chặn kiểm tra output?
- [ ] Output lưu file (không chỉ in ra ngữ cảnh)?
