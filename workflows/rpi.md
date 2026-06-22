# Quy trình RPI — Nghiên cứu → Kế hoạch → Triển khai

Quy trình phát triển tính năng có cấu trúc, với cổng chặn (quality gate) ở mỗi bước. Ngăn lãng phí token vào tính năng không khả thi.

---

## Tổng quan

```
┌─────────────────┐    ĐI     ┌─────────────────┐   DUYỆT   ┌─────────────────┐
│  1. NGHIÊN CỨU  │─────────▶│  2. KẾ HOẠCH    │──────────▶│  3. TRIỂN KHAI   │
│     (Research)   │          │     (Plan)       │          │     (Implement)  │
└─────────────────┘          └─────────────────┘          └─────────────────┘
        │                            │                            │
     DỪNG                        SỬA LẠI                     THẤT BẠI
        │                            │                            │
        ▼                            ▼                            ▼
    Bỏ cuộc                   Quay lại chỉnh               Sửa + thử lại
    (Tiết kiệm token)        kế hoạch                     (tối đa 2 lần)
```

---

## Bước 0: Khởi tạo

Khi nhận yêu cầu "RPI cho [tên]" hoặc "SuperBuild [tên]":

1. Tạo thư mục: `rpi/{ten-tinh-nang}/`
2. Copy mẫu: `workflows/templates/REQUEST.md` → `rpi/{ten}/REQUEST.md`
3. Điền mô tả tính năng vào REQUEST.md
4. Tạo MEMORY-TASK và MEMORY-FORGE nếu dự án lớn

---

## Bước 1: NGHIÊN CỨU (Research)

### Mục tiêu
Phân tích tính khả thi TRƯỚC KHI viết code. Trả lời: "Có nên làm không?"

### Quy trình
Spawn sub-agent nghiên cứu:

```
sessions_spawn(
    task="""
    ## Nhiệm vụ: Nghiên cứu tính khả thi — [tên tính năng]
    
    ## Đầu vào
    Đọc file: rpi/{ten}/REQUEST.md
    
    ## Phân tích (5 phần)
    1. Phân tích yêu cầu — tính năng cần gì, người dùng cần gì?
    2. Phân tích kỹ thuật — dùng thư viện/framework nào, có gì sẵn?
    3. Phân tích rủi ro — có gì có thể hỏng, phụ thuộc ngoài?
    4. Ước tính công sức — bao nhiêu agent/step/token?
    5. Phán quyết — ĐI (feasible) hay DỪNG (infeasible) + lý do
    
    ## Đầu ra
    Lưu tại: rpi/{ten}/RESEARCH.md
    Dùng template: workflows/templates/RESEARCH.md
    """,
    model="claudible/claude-sonnet-4.5",
    label="rpi-research-{ten}"
)
```

### Cổng chặn
| Phán quyết | Hành động |
|-----------|----------|
| **ĐI** | Qua Bước 2 |
| **ĐI CÓ ĐIỀU KIỆN** | Liệt kê điều kiện, qua Bước 2 nếu chấp nhận |
| **DỪNG** | Dừng lại, giải thích lý do, đề xuất thay thế |

---

## Bước 2: KẾ HOẠCH (Plan)

### Mục tiêu
Chia tính năng thành các giai đoạn nhỏ, mỗi giai đoạn hoàn thành độc lập.

### Quy trình
Spawn sub-agent lên kế hoạch:

```
sessions_spawn(
    task="""
    ## Nhiệm vụ: Lên kế hoạch triển khai — [tên tính năng]
    
    ## Đầu vào
    - Đọc: rpi/{ten}/REQUEST.md (yêu cầu)
    - Đọc: rpi/{ten}/RESEARCH.md (phân tích khả thi)
    
    ## Yêu cầu
    1. Chia thành 3-6 giai đoạn (phases)
    2. Mỗi giai đoạn có: deliverables rõ ràng, file cần tạo/sửa, tiêu chí hoàn thành
    3. Giai đoạn phải tuần tự — giai đoạn sau dựa trên giai đoạn trước
    4. Mỗi giai đoạn hoàn thành trong < 50% ngữ cảnh
    
    ## Đầu ra
    Lưu tại: rpi/{ten}/PLAN.md
    Dùng template: workflows/templates/PLAN.md
    """,
    model="claudible/claude-opus-4.6",
    label="rpi-plan-{ten}"
)
```

### Cổng chặn
- Người dùng DUYỆT kế hoạch trước khi triển khai
- Nếu cần sửa → chỉnh PLAN.md → duyệt lại

---

## Bước 3: TRIỂN KHAI (Implement)

### Mục tiêu
Thực hiện từng giai đoạn theo PLAN.md, mỗi giai đoạn có review.

### Quy trình — Lặp cho mỗi giai đoạn

```
Với mỗi Giai đoạn N trong PLAN.md:

1. KHÁM PHÁ — Đọc code hiện có, hiểu patterns
   → Spawn agent nhẹ (Sonnet) đọc + phân tích

2. TRIỂN KHAI — Viết code theo kế hoạch
   → Spawn agent lập trình (Opus) với:
   - Context từ bước khám phá
   - Deliverables cụ thể từ PLAN.md
   - Patterns cần tuân theo

3. TỰ KIỂM TRA — Agent tự chạy test, lint
   → Phần của bước 2 (cùng agent)

4. REVIEW — Agent khác kiểm tra chất lượng
   → Spawn agent review (Sonnet) đọc diff + đánh giá

5. CỔNG CHẶN — Kiểm tra kết quả
   - PASS → qua giai đoạn tiếp
   - FAIL → sửa + review lại (tối đa 2 lần)
   - Vẫn FAIL → dừng, báo người dùng

6. GHI NHẬT KÝ — Cập nhật IMPLEMENT.md + memory
```

### Mẫu spawn triển khai

```
sessions_spawn(
    task="""
    ## Nhiệm vụ: Triển khai giai đoạn {N} — [tên giai đoạn]
    
    ## Ngữ cảnh
    - Kế hoạch: rpi/{ten}/PLAN.md (đọc Giai đoạn {N})
    - Code hiện có: [kết quả khám phá]
    - Giai đoạn trước: [tóm tắt kết quả]
    
    ## Deliverables
    1. [File cần tạo/sửa + mô tả]
    2. [File cần tạo/sửa + mô tả]
    
    ## Ràng buộc
    - Tuân theo patterns hiện có
    - Viết test cho code mới
    - Không break code cũ
    
    ## Đầu ra
    - Code files đã tạo/sửa
    - Cập nhật: rpi/{ten}/IMPLEMENT.md (Giai đoạn {N})
    """,
    model="claudible/claude-opus-4.6",
    label="rpi-impl-{ten}-phase{N}"
)
```

---

## Khi nào DÙNG / KHÔNG DÙNG RPI

| Dùng RPI | Không dùng RPI |
|----------|---------------|
| Tính năng mới > 2 giờ | Sửa bug đơn giản |
| Cần nhiều agent phối hợp | Thay đổi < 30 phút |
| Rủi ro cao, cần nghiên cứu | Công việc routine |
| Dự án mới từ đầu | Chỉ cập nhật docs |
| Kiến trúc phức tạp | Refactor nhỏ |

---

## Cấu trúc thư mục đầu ra

```
rpi/{ten-tinh-nang}/
├── REQUEST.md      ← Mô tả ban đầu (Bước 0)
├── RESEARCH.md     ← Phân tích khả thi (Bước 1)
├── PLAN.md         ← Kế hoạch chia giai đoạn (Bước 2)
└── IMPLEMENT.md    ← Nhật ký triển khai (Bước 3)
```

---

## Xử lý lỗi

| Lỗi | Xử lý |
|-----|-------|
| Sub-agent chết giữa chừng | Đọc output file → respawn từ bước cuối |
| Test fail | Sửa + chạy lại (tối đa 2 lần) → báo user |
| Build fail | Kiểm tra import/type → sửa → rebuild |
| Giai đoạn FAIL 2 lần | DỪNG, báo user quyết định |
| Agent chạy quá 20 phút | Kill + respawn cùng task |
