# 5 Mẫu Thiết Kế Skill — Design Patterns

Nguồn: Anthropic "The Complete Guide to Building Skills for Claude" Ch.5
Đã dịch, bổ sung ví dụ OpenClaw.

---

## Chọn Hướng Tiếp Cận

| Hướng | Mô tả | Ví dụ |
|-------|-------|-------|
| **Từ vấn đề** | "Tao cần setup workspace" → skill chọn tool phù hợp | Người dùng mô tả kết quả, skill xử lý tool |
| **Từ công cụ** | "Tao có Suno connected" → skill dạy cách dùng tối ưu | Người dùng có access, skill cung cấp chuyên môn |

Hầu hết skill nghiêng về 1 hướng. Biết hướng nào giúp chọn pattern đúng.

---

## Pattern 1: Tuần Tự (Sequential Workflow)

**Dùng khi:** Quy trình nhiều bước theo thứ tự cố định.

```
Bước 1 → Bước 2 → Bước 3 → Bước 4
   ↓         ↓         ↓         ↓
Xác thực  Xác thực  Xác thực  Kết quả
```

**Kỹ thuật chính:**
- Thứ tự bước rõ ràng
- Phụ thuộc giữa các bước (output bước 1 = input bước 2)
- Xác thực ở mỗi giai đoạn
- Hướng dẫn rollback khi lỗi

**OpenClaw ví dụ — Onboard dự án:**
```markdown
### Bước 1: Tạo workspace
exec: mkdir -p project/{src,docs,tests}

### Bước 2: Khởi tạo cấu hình
write: project/package.json với dependencies

### Bước 3: Cài đặt
exec: cd project && npm install
Chờ: cài xong không lỗi

### Bước 4: Xác nhận
exec: npm test
Nếu PASS → báo thành công
Nếu FAIL → rollback bước 2, sửa config
```

---

## Pattern 2: Phối Hợp Đa Nguồn (Multi-Source Coordination)

**Dùng khi:** Workflow trải qua nhiều dịch vụ/công cụ.

```
Phase 1 (Tool A) → Phase 2 (Tool B) → Phase 3 (Tool C) → Phase 4 (Tool D)
```

**Kỹ thuật chính:**
- Tách phase rõ ràng
- Truyền dữ liệu giữa các phase
- Xác thực trước khi sang phase tiếp
- Xử lý lỗi tập trung

**OpenClaw ví dụ — Nghiên cứu + Viết + Publish:**
```markdown
### Phase 1: Nghiên cứu (web_search + web_fetch)
1. Tìm kiếm 10 nguồn
2. Trích xuất nội dung
3. Tạo tóm tắt

### Phase 2: Phân tích (memory_search + read)
1. Tìm ngữ cảnh liên quan trong memory
2. So sánh với nghiên cứu mới
3. Tạo báo cáo phân tích

### Phase 3: Viết (write)
1. Tạo bản nháp
2. Áp dụng style guide
3. Lưu file

### Phase 4: Phân phối (message)
1. Gửi qua Telegram
2. Tạo tóm tắt ngắn
3. Đính kèm file
```

---

## Pattern 3: Tinh Chỉnh Lặp (Iterative Refinement)

**Dùng khi:** Chất lượng output cải thiện qua nhiều vòng.

```
Bản nháp → Kiểm tra → Sửa → Kiểm tra lại → ... → Bản cuối
```

**Kỹ thuật chính:**
- Tiêu chí chất lượng rõ ràng
- Vòng cải tiến có giới hạn (tránh loop vô tận)
- Script xác thực nếu có thể
- Biết khi nào dừng

**OpenClaw ví dụ — Viết tiểu thuyết (Omni Forge):**
```markdown
### Bản nháp
1. Agent 1 viết beat
2. Agent 2 viết beat (tham khảo Agent 1)
3. Agent 3 viết beat (tham khảo Agent 1+2)

### Kiểm tra chất lượng
1. Micro-council chọn khung tốt nhất
2. Mix highlights từ 3 bản
3. Chấm điểm

### Vòng tinh chỉnh
1. Grind: sửa lỗi cấp câu
2. Elevate: nâng cấp nghệ thuật
3. Cross-check: kiểm tra liên chương

### Điều kiện dừng
- Council PASS (≥7/10) → lưu FINAL
- FAIL lần 1 → sửa + review lại
- FAIL lần 2 → dừng, hỏi người dùng
```

---

## Pattern 4: Chọn Công Cụ Theo Ngữ Cảnh (Context-Aware Selection)

**Dùng khi:** Cùng mục tiêu nhưng tool khác nhau tùy tình huống.

```
Phân tích ngữ cảnh → Cây quyết định → Chọn tool → Thực thi
```

**Kỹ thuật chính:**
- Tiêu chí quyết định rõ ràng
- Phương án dự phòng
- Minh bạch về lý do chọn

**OpenClaw ví dụ — Tải file thông minh:**
```markdown
### Cây quyết định

1. Kiểm tra loại file và kích thước
2. Chọn phương thức:
   - File > 100MB: dùng IDM (multi-thread download)
   - File media: dùng yt-dlp hoặc web_fetch
   - File text/code: dùng web_fetch trực tiếp
   - Cần đăng nhập: dùng browser automation

### Thực thi
Gọi tool phù hợp theo quyết định

### Giải thích
Nói cho người dùng TẠI SAO chọn phương thức đó
```

---

## Pattern 5: Chuyên Môn Lĩnh Vực (Domain Intelligence)

**Dùng khi:** Skill thêm kiến thức chuyên ngành vượt xa tool access.

```
Kiểm tra quy tắc → PASS? → Thực thi → Ghi nhật ký
                  → FAIL? → Đánh dấu → Tạo case
```

**Kỹ thuật chính:**
- Kiến thức chuyên môn nhúng trong logic
- Kiểm tra trước hành động
- Tài liệu hóa toàn diện
- Quản trị rõ ràng

**OpenClaw ví dụ — Review hợp đồng:**
```markdown
### Trước khi phân tích (Compliance Check)
1. Đọc file hợp đồng
2. Áp dụng 41 danh mục rủi ro CUAD:
   - Điều khoản chấm dứt
   - Giới hạn trách nhiệm
   - Bồi thường
3. Ghi kết quả kiểm tra

### Phân tích
NẾU compliance PASS:
  - Tạo báo cáo rủi ro
  - Đề xuất redline
  - So sánh với chuẩn thị trường
NẾU KHÔNG:
  - Đánh dấu điều khoản nguy hiểm
  - Tạo case review

### Nhật ký kiểm toán
- Ghi mọi kiểm tra compliance
- Lưu quyết định phân tích
- Tạo báo cáo audit trail
```

---

## Bảng Chọn Pattern

| Tình huống | Pattern | Ví dụ |
|-----------|---------|-------|
| Bước A xong mới làm B | **Tuần tự** | CI/CD, deploy, onboard |
| Dùng nhiều tool/API khác nhau | **Đa nguồn** | Research + Write + Publish |
| Output cần cải thiện qua nhiều vòng | **Tinh chỉnh lặp** | Viết văn, thiết kế, code review |
| Cùng việc nhưng tool khác tùy context | **Chọn theo ngữ cảnh** | Download, storage, notification |
| Cần kiến thức chuyên ngành | **Chuyên môn** | Pháp lý, tài chính, y tế |
| Phức tạp, kết hợp nhiều yếu tố | **Kết hợp 2-3 patterns** | SuperBuild RPI = Tuần tự + Đa nguồn |
