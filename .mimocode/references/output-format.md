# output-format.md — Nguồn sự thật duy nhất cho mọi định dạng output

Mọi skill (`analyze-requirements`, `design-architecture`,
`generate-backend-module`, `generate-frontend-module`,
`explain-module`, `review-integration`, `finalize-delivery`) PHẢI dùng
đúng format ở đây. Không skill nào được tự bịa format khác.

---

## 0. Step Log (bắt buộc in ra Terminal ở MỌI bước của MỌI skill)

```
[<Skill>] Bước <n>/<tổng số bước> — <tên bước>
→ Đang làm: <mô tả ngắn hành động thật đang thực hiện>
→ Kết quả: <kết quả cụ thể, không nói chung chung "đã xong">
```

Không gộp nhiều bước rồi tóm tắt 1 lần ở cuối. Trước khi kết thúc lượt,
rà lại: bước nào đã chạy mà chưa có Step Log → chưa hoàn tất, phải bổ
sung.

---

## 1. Requirement Digest (output của `analyze-requirements`, toàn dự án 1 lần)

### 1a. Module Manifest
| Module | Actor liên quan | Số UC | Priority cao nhất | Phụ thuộc module khác |
|---|---|---|---|---|

### 1b. UC Detail (nguồn: sheet DD trong `{PROJECT_ROOT}/docs/`)
| Mã CN | UC ID | Module | Actor | Priority | Endpoint(s) | Request DTO | Response DTO | Business Rule tóm tắt | Ghi chú/giả định |
|---|---|---|---|---|---|---|---|---|---|

Cột "Ghi chú/giả định" chỉ điền khi tài liệu thực sự thiếu — ghi rõ
"cần xác nhận: <câu hỏi cụ thể>", không tự bịa.

### 1c. Dependency Graph
```
Authentication → (none)
Product Management → Category/Brand
...
```

Phải đủ chi tiết mức module-tới-module (không chỉ tên module) — đây là
căn cứ cho Module Build Order và cho việc xác định "phụ thuộc dữ liệu
trực tiếp" dùng ở Mục 2.5 dưới đây.

---

## 2. Architecture Design Document (output của `design-architecture`, ERD chốt 1 lần — Schema Lock)

1. **ERD tóm tắt** — Tên bảng | Cột chính (PK/FK) | Quan hệ | Constraint.
2. **Solution Structure** — cây thư mục Backend (Layered) + Frontend (Angular).
3. **Cross-cutting Setup** — Hạng mục | Cách triển khai | File cấu hình.
4. **Module Build Order** — thứ tự | risk (Low/Medium/High) | lý do.
5. **Cụm rủi ro & phụ thuộc dữ liệu trực tiếp** (dùng cho Mini
   Integration Check ở Gate 3):

   | Module (risk Medium/High) | Đọc/ghi dữ liệu trực tiếp của module nào | Điểm nối cần soi khi tới Gate 3 |
   |---|---|---|

   Module risk Medium/High nhưng không có phụ thuộc dữ liệu trực tiếp
   → ghi rõ "không có", không bỏ trống dòng.

### 2.6. Lịch sử Schema Drift (chỉ xuất hiện nếu có phát sinh)

Mỗi lần `generate-backend-module` báo Schema Drift và được xử lý qua
chế độ sửa cục bộ, ghi thêm 1 dòng — không xoá lịch sử cũ:

| Thời điểm (module đang code lúc phát sinh) | Entity | Thay đổi | Module `done` bị ảnh hưởng | Phương án đã chọn |
|---|---|---|---|---|

---

## 3. Human Review Gate (Gate 2 sau `design-architecture`, Gate cục bộ khi có Schema Drift)

```
📋 <Tên Gate> — cần bạn xác nhận trước khi tiếp tục

<Nội dung đầy đủ cần duyệt — KHÔNG rút gọn thành "xem file để biết chi tiết">

>> Gõ "ok" để tiếp tục, hoặc nêu rõ điều chỉnh bạn muốn.
```

Gate cục bộ (Schema Drift có xung đột thật) dùng đúng khối trên nhưng
chỉ trình bày phần Entity/module bị ảnh hưởng, không cần lặp lại toàn
bộ ERD hệ thống.

## 3b. Input không hợp lệ / ngoài phạm vi — nhị phân, không đếm lần

**Sai định dạng nhưng vẫn thuộc bước đang hỏi (không giới hạn số lần):**
```
⚠️ Chưa hợp lệ cho bước hiện tại: <lý do cụ thể, vd "số 9 không có trong danh sách 1-6">
<lặp lại nguyên văn câu hỏi/định dạng cần của bước hiện tại>
```

**Rõ ràng ngoài phạm vi coding E-commerce của agent (dừng ngay, kể cả lần đầu):**
```
⚠️ Yêu cầu này ngoài phạm vi của agent (agent chỉ sinh code E-commerce
theo tài liệu ở docs/, không làm việc khác). Agent dừng lượt này.

Câu hỏi/nội dung đang chờ trả lời:
<lặp lại nguyên văn câu hỏi/nội dung của bước đang chờ>

Trả lời đúng nội dung trên để tiếp tục workflow.
```

---

## 4. Generation Report (mỗi module, cả BE lẫn FE)

### 4a. Danh sách file đã sinh
| File | Vai trò | Ghi chú |
|---|---|---|

### 4b. Self-Review đối chiếu UC ↔ Code
| UC ID | Business Rule | Đã implement ở đâu (file:hàm) | Trạng thái | Ghi chú |
|---|---|---|---|---|

### 4c. Generalization Check (không được bỏ nhóm nào mà không giải thích N/A)
- Input rỗng/null:
- Input biên:
- Input sai kiểu/định dạng:
- Trường hợp lỗi nghiệp vụ:
- Trường hợp đồng thời (race condition) nếu liên quan tồn kho/thanh toán:

### 4d. Risk & Trạng thái cuối
- Risk: Low/Medium/High — lý do cụ thể.
- Trạng thái: `pass` / `verify_fail` / `schema_drift_detected` (nếu
  fail hoặc drift, nêu rõ cần sửa gì / đang chờ gì).
- Số lần đã tự sửa (`retry_count`) trong lần build này.

### 4e. Giải thích Module (output của `explain-module`, per file, đủ 11 mục nhỏ)

```markdown
### Giải thích: <đường dẫn file>

- **File này dùng để làm gì:**
- **Class/Interface chính:**
- **Vì sao cần tạo nó:**
- **Luồng hoạt động:**
- **Input:**
- **Output:**
- **Dependency (inject gì, vì sao):**
- **Giao tiếp với class nào khác, như thế nào:**
- **Nếu xoá file này thì chuyện gì xảy ra:**
- **Lỗi thường gặp khi sửa file này:**
- **Cách debug khi file này gây lỗi:**
```

Kèm 1 đoạn **Luồng hoạt động của riêng module này** (request → response,
qua từng layer) — không phải luồng liên module (đó là việc của
`review-integration`).

---

## 5. Integration Review Report

### 5a. Mini Integration Check (output mode=mini, ghi vào log Gate 3 — không cần file riêng)

```markdown
### Mini Integration Check — Cụm module: <danh sách module>

| Điểm nối | Pass/Fail | Bằng chứng (file:hàm hoặc test) |
|---|---|---|

Kết luận: Pass toàn bộ / Fail — cần quay lại module nào
```

### 5b. Full Integration Review (output mode=full, `INTEGRATION_REVIEW.md`)

```markdown
### Integration Review — Toàn hệ thống

| Hạng mục | Kết quả | Ghi chú |
|---|---|---|
| Build toàn solution (Backend) | Pass/Fail | |
| Build toàn solution (Frontend) | Pass/Fail | |
| Luồng tích hợp liên module (điểm nối chưa qua mini check) | Pass/Fail | |
| Xác nhận lại điểm nối đã Pass ở mini check | Pass/Fail | |
| Schema Lock — không còn Schema Drift dang dở | Pass/Fail | |
| Security tổng thể (JWT, SQL Injection, XSS, CORS) | Pass/Fail | |
| Performance tổng thể (N+1, pagination, index) | Pass/Fail | |
```

---

## 6. Summary Report (output `finalize-delivery`)

1. **Tổng quan**: số module Backend/Frontend/Explain đã Pass, phạm vi đã build.
2. **Kết quả từng module**: bảng module | risk cuối | assumption còn treo.
3. **Assumptions còn treo**: đầy đủ, bao gồm Schema Drift đã xử lý (nếu có) — PHẢI đọc trước khi deploy.
4. **Hướng dẫn chạy thử**: migration DB, seed data, chạy Backend, chạy Frontend, tài khoản test.
5. **Đề xuất cải tiến**: các điểm có thể tối ưu thêm.

Mỗi mục 2 phải trỏ lại được `GENERATION_REPORT_BE/FE_<module>.md` và
`MODULE_EXPLANATION_<module>.md` tương ứng — không tổng hợp lại từ đầu.

---

## Quy tắc chung cho mọi format trên
- Không được chỉ nói "đã tạo xong, xem file để biết chi tiết" cho các
  mục có bảng/nội dung bắt buộc — phải in **toàn bộ nội dung thật**
  trực tiếp vào phản hồi.
- Không đổi tên cột, không đổi thứ tự mục.
- Bảng dài vẫn phải in đủ, không cắt bớt hàng.
