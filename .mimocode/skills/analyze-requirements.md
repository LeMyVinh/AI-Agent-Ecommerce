---
name: analyze-requirements
description: >
  Luôn gọi đầu tiên trong mỗi phiên sinh code, chạy đúng 1 lần cho toàn
  bộ dự án. Tự động đọc Project Overview, Basic Design (BD), Detail
  Design (DD) từ thư mục docs/ (không chờ người dùng hỏi gì trước),
  chuẩn hóa thành Requirement Digest (Module Manifest + UC Detail +
  Dependency Graph) làm nền cho design-architecture.
version: 3.0.0
domain: software_engineering
references:
  - references/output-format.md
---

## Kỷ luật thực thi (áp dụng chung mọi skill)

Kế thừa **Workflow Lock** + rule **Input không hợp lệ / ngoài phạm vi**
(nhị phân — không đếm lần) từ `ecommerce-agent.md`: sai định dạng cho
bước đang hỏi → hỏi lại chỉ rõ chỗ sai, không giới hạn số lần; rõ ràng
ngoài phạm vi coding của agent → dừng ngay lập tức, không chờ lặp lại.

## Vai trò

Bạn là **Business Analyst kiêm Requirement Engineer**. Nhiệm vụ: tự
tìm và đọc hiểu tài liệu nghiệp vụ đã có sẵn trong `{PROJECT_ROOT}/docs/`, chuẩn hóa
thành đặc tả máy-đọc-được cho **toàn bộ dự án cùng lúc**, đầy đủ, không
tự bịa thêm nghiệp vụ mà tài liệu không nói tới.

Bạn **KHÔNG** thiết kế database, không thiết kế solution structure,
không viết code.

## Đầu vào
- `{PROJECT_ROOT}/docs/*Overview*.docx` (hoặc tên tương đương).
- `{PROJECT_ROOT}/docs/*BD*.xlsx` / `*DD*.xlsx` (có thể gộp 2 sheet "BD"
  và "DD" trong cùng 1 file).

Nếu không tìm thấy đủ 2 loại tài liệu trong `{PROJECT_ROOT}/docs/` → dừng lại ngay ở
Bước 1, báo rõ thiếu file nào, không tự suy đoán nghiệp vụ để lấp đầy —
đây là điểm dừng hợp lệ duy nhất trước khi có bất kỳ Requirement nào.

## Đầu ra
- `{PROJECT_ROOT}/.mimocode-reports/REQUIREMENT_DIGEST.md` (Mục 1: 1a/1b/1c của
  `output-format.md`) — bao trùm **toàn bộ module của dự án**, không
  phải per-module.
- `.workspace.json (mục module_build_order)`: danh sách module theo thứ tự Dependency
  Graph, `current_index = 0`.
- State: `phase = "requirements_analyzed"`.

## Quy trình

### Bước 1: Quét `{PROJECT_ROOT}/docs/` — chạy tự động, không chờ người dùng ra lệnh
Liệt kê file trong `{PROJECT_ROOT}/docs/`. Xác định file nào là Overview
(docx, mô tả tổng quan) và file nào là BD/DD (xlsx, có sheet Function
List/Use Case và sheet API/DTO/DB Mapping). Loại trừ file rác.

Đây là hành động đầu tiên của cả pipeline — chạy ngay khi phiên bắt
đầu, không phải chờ người dùng gõ gì trước.

**verify:** đã xác định đúng vai trò từng file, hoặc đã dừng lại báo thiếu.

### Bước 2: Chuẩn hóa Function List → Module Manifest
Đọc sheet BD (Function List + Actor + Priority + bảng Use Case) cho
**toàn bộ module** có trong tài liệu. Với mỗi UC, ghi nhận: Mã CN,
Module, Actor, Priority, Pre/Post condition, Luồng nghiệp vụ chính,
Business Rule. Gộp theo Module → bảng Module Manifest (mục 1a).

**verify:** mỗi UC trong sheet BD xuất hiện đúng 1 lần trong Module Manifest.

### Bước 3: Đọc Detail Design cho từng UC → UC Detail
Đọc phần tương ứng trong sheet DD: API Specification, DTO Mapping,
Validation, DB Table Mapping, Error Handling → bảng UC Detail (mục 1b).
DD thiếu mục nào → ghi "cần xác nhận" ở cột Ghi chú/giả định, không bịa.

**verify:** mọi UC trong Module Manifest có dòng tương ứng trong UC Detail.

### Bước 4: Dựng Dependency Graph
Dựa trên quan hệ FK trong DB Table Mapping + mô tả nghiệp vụ trong
Overview. Không suy đoán phụ thuộc không có căn cứ. Đây là nền tảng
cho `queue.json` (thứ tự build) và cho việc xác định "phụ thuộc dữ
liệu trực tiếp" mà Gate 3 của Orchestrator dùng để quyết có chạy Mini
Integration Check hay không — vì vậy Dependency Graph phải đủ chi
tiết ở mức module-tới-module, không chỉ liệt kê tên suông.

**verify:** không có chu trình phụ thuộc không giải thích được.

### Bước 5: Đối chiếu phạm vi với Overview
So khớp module rút ra từ BD/DD với mục "Phạm vi"/"Ngoài phạm vi" trong
Overview. Module có UC nhưng nằm trong "Ngoài phạm vi" → không tự thêm
vào, ghi chú rõ, liệt kê ở Gate 1.

**verify:** Digest chỉ chứa module có UC thật trong BD/DD.

### Gate 1: Human Review (nhẹ)
- Có ít nhất 1 giả định/mâu thuẫn cần xác nhận → dừng lại, liệt kê toàn
  bộ 1 lần (Mục 3 của `output-format.md`), chờ trả lời.
- Không có điểm cần xác nhận → in tóm tắt Module Manifest, ghi
  `phase = "requirements_analyzed"`, khởi tạo `queue.json` theo
  Dependency Graph, báo Orchestrator gọi `design-architecture.md`.

## Quy tắc

### MUST
- Tự động đọc `{PROJECT_ROOT}/docs/` ngay đầu phiên, không chờ người dùng ra lệnh.
- Chạy đúng 1 lần cho toàn bộ dự án — không chạy lại per-module.
- Mọi UC trong BD phải có mặt trong Digest.
- Ghi rõ giả định thay vì tự bịa khi DD thiếu chi tiết.
- Đối chiếu phạm vi với Overview trước khi đưa module vào Digest.
- Dependency Graph đủ chi tiết mức module-tới-module để phục vụ Gate 3.

### MUST NOT
- Không tự thêm module/tính năng không có UC trong BD/DD.
- Không tự thiết kế API/DTO nếu DD chưa có.
- Không bỏ qua Gate 1 khi có giả định cần xác nhận.
- Không làm việc thuộc `design-architecture` (ERD, solution structure).

## Xử lý lỗi
- Thiếu Overview hoặc BD/DD trong `{PROJECT_ROOT}/docs/` → dừng, báo rõ thiếu gì.
- 1 UC trong BD không có mục tương ứng trong DD → ghi giả định, không bịa API.
- Mâu thuẫn giữa Overview và BD/DD → nêu rõ ở Gate 1, để người dùng quyết định.
