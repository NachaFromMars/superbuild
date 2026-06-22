# Tối ưu gọi công cụ (Tool Optimization)

Mỗi lần gọi công cụ tốn token. Gọi thông minh = ít token hơn + nhanh hơn + kết quả tốt hơn.

---

## Nguyên tắc cốt lõi

### 1. Gom lô (Batch) — Gọi song song khi không phụ thuộc

❌ **Sai:**
```
Gọi read(file1) → đợi → Gọi read(file2) → đợi → Gọi read(file3) → đợi
```

✅ **Đúng:**
```
Gọi read(file1) + read(file2) + read(file3) cùng lúc → đợi 1 lần
```

**Quy tắc:** Nếu các lệnh gọi không phụ thuộc kết quả lẫn nhau → gom vào 1 lượt.

### 2. Lọc trước khi đọc — Không nạp cả file 10,000 dòng

❌ **Sai:** `read(file.md)` → nạp toàn bộ 50KB vào ngữ cảnh
✅ **Đúng:** `read(file.md, offset=50, limit=30)` → chỉ đọc 30 dòng cần thiết

Áp dụng:
- `web_fetch` với `maxChars` để giới hạn kết quả web
- `read` với `offset` + `limit` cho file lớn
- `web_search` với `count` để giới hạn số kết quả

### 3. Ghi file thay vì giữ trong ngữ cảnh

Khi xử lý dữ liệu lớn:
```
Bước 1: Đọc dữ liệu → Xử lý → Ghi kết quả ra file
Bước 2: Xóa dữ liệu thô khỏi ngữ cảnh (nén)
Bước 3: Đọc lại file kết quả khi cần
```

Kết quả trung gian nặng KHÔNG nên nằm trong ngữ cảnh.

---

## Kỹ thuật nâng cao

### Gọi công cụ lập trình (Programmatic Tool Calling — PTC)

Khái niệm từ API level của Claude: thay vì gọi 10 công cụ = 10 lượt suy luận, viết 1 script gọi cả 10 → chỉ 1 lượt suy luận.

**Áp dụng cho OpenClaw:**
Dùng `exec` với script khi cần xử lý lô:
```powershell
# Thay vì gọi read 5 lần riêng biệt
Get-ChildItem *.md | ForEach-Object { 
    echo "=== $($_.Name) ==="; 
    Get-Content $_ -Head 5 
}
```

### Tìm kiếm công cụ (Tool Search)

Khi có nhiều skill, không nạp hết cùng lúc. Cơ chế nạp dần của OpenClaw:
- SKILL.md liệt kê bảng "tình huống → file"
- Agent chỉ đọc file khi gặp tình huống tương ứng
- Tiết kiệm ~85% token so với nạp hết

### Ví dụ công cụ (Tool Use Examples)

Khi viết task cho sub-agent, bao gồm ví dụ cụ thể:
```
Ví dụ output mong đợi:
- File: rpi/oauth2/RESEARCH.md
- Format: ## Kết luận\n**Phán quyết:** ĐI\n**Lý do:** [3 bullet points]
```

Ví dụ cụ thể giúp agent hiểu đúng format → giảm retry.

---

## Bảng tối ưu theo tình huống

| Tình huống | Kỹ thuật | Token tiết kiệm |
|-----------|----------|-----------------|
| Đọc nhiều file cùng lúc | Gom lô (batch read) | ~50% (giảm roundtrip) |
| File lớn, chỉ cần 1 phần | Đọc có offset/limit | ~80% (chỉ nạp đoạn cần) |
| Kết quả web dài | maxChars trong web_fetch | ~60% (cắt rác) |
| Xử lý 10+ file | exec script thay vì 10 lệnh read | ~70% (1 roundtrip) |
| Sub-agent cần knowledge | Ghi vào task description, không spawn rồi feed | ~30% (giảm tin nhắn) |
| Nhiều skill definitions | Nạp dần theo bảng điều hướng | ~85% (chỉ đọc khi cần) |

---

## Thứ tự ưu tiên tối ưu

1. **Gom lô** — dễ nhất, hiệu quả nhất
2. **Giới hạn kích thước** — đọc/fetch có limit
3. **Ghi file** — kết quả trung gian ra file, không giữ ngữ cảnh
4. **Script hóa** — dùng exec cho batch operations
5. **Nạp dần** — skill/reference chỉ đọc khi cần

---

## Checklist tối ưu

- [ ] Có đang gọi nhiều read/fetch riêng lẻ mà có thể gom?
- [ ] File đọc có cần toàn bộ hay chỉ cần 1 phần?
- [ ] Kết quả trung gian có đang nằm trong ngữ cảnh?
- [ ] Có thể dùng exec script thay vì nhiều lệnh riêng?
