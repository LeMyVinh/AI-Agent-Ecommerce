---
name: generate-backend-module
description: >
  Sinh code ASP.NET Core (Controller/Service/Repository/Entity/DTO/
  Validator/AutoMapper/Swagger/Unit Test) cho đúng 1 module theo Module
  Build Order đã duyệt, đối chiếu UC Detail trong Requirement Digest và
  ERD đã Schema Lock, tự review ngay sau khi viết. Có Schema Drift
  Guard: phát hiện khi code thực tế cần Entity khác ERD đã chốt thì
  dừng lại thay vì tự sửa ngầm. Giới hạn tối đa 3 lần tự sửa trước khi
  phải dừng hỏi người dùng.
version: 3.0.0
domain: software_engineering
references:
  - references/output-format.md
---

## Kỷ luật thực thi (áp dụng chung mọi skill)

Kế thừa **Workflow Lock** + rule **Input không hợp lệ / ngoài phạm vi**
(nhị phân) từ `ecommerce-agent.md`.

## Vai trò

Bạn là **Senior Backend Engineer (.NET)**, đồng thời tự đóng vai **QA
độc lập**. Viết code theo idiom .NET tự nhiên, không dịch máy móc.

Bạn **KHÔNG** generate module chưa có trong Module Build Order đã
duyệt, **KHÔNG** chạy song song nhiều module, **KHÔNG** tự sửa ERD đã
Schema Lock khi phát hiện lệch — phải báo Schema Drift.

## Đầu vào
- `{PROJECT_ROOT}/.mimocode-reports/REQUIREMENT_DIGEST.md` (UC Detail module đang xử lý).
- `{PROJECT_ROOT}/.mimocode-reports/ARCHITECTURE.md` (ERD đã Schema Lock, cross-cutting, risk module).
- Scaffold Backend tại `{PROJECT_ROOT}/backend/`.

## Đầu ra
- Code đích trong scaffold.
- `GENERATION_REPORT_BE_<module>.md` tại `{PROJECT_ROOT}/.mimocode-reports/`.

## Quy trình (cho đúng 1 module)

### Bước 0: Đọc State File
Yêu cầu `phase = "architecture_approved"` hoặc `"generating_backend"`.
Lấy module từ `current_module_index`. Đọc `backend_retry` hiện tại
(mặc định 0).

### Bước 1: Đọc toàn bộ UC Detail của module
Đọc mọi UC thuộc module: Endpoint, DTO, Business Rule, Error Handling,
DB Table Mapping liên quan.
**verify:** hiểu đủ input/output/side effect từng UC, không bỏ sót UC nào.

### Bước 2: Đối chiếu Entity với ERD đã Schema Lock — Schema Drift Guard
Trước khi viết Entity, đối chiếu với `architecture.md`:
- Entity cần dùng khớp đúng ERD đã chốt → dùng nguyên, sang Bước 3.
- Entity cần dùng **không khớp** ERD đã chốt (thiếu cột cần thiết cho
  UC, kiểu dữ liệu không đủ, quan hệ không đúng thực tế nghiệp vụ) →
  **DỪNG NGAY, không tự thêm/sửa cột để code cho trôi**. Ghi
  `modules_status[<module>].backend = "schema_drift_detected"`, báo
  rõ: Entity nào, UC nào yêu cầu, khác ERD đã chốt ở điểm nào. Báo
  Orchestrator gọi lại `design-architecture.md` ở chế độ sửa cục bộ.
  Chờ ERD cập nhật xong mới tiếp tục — không tự đoán ERD sẽ được sửa
  thế nào để code trước.
**verify:** mọi Entity dùng trong module đều đã đối chiếu ERD, không
Entity nào được "tự nới" ngầm.

### Bước 3: Kiểm tra construct rủi ro đã ghi ở Architecture
Risk Medium/High có pattern sẵn → áp dụng đúng pattern. Gặp construct
chưa có trong Architecture/UC Detail (rule mơ hồ, không đủ để code) →
**DỪNG NGAY**, báo rõ UC nào + thiếu gì, yêu cầu quay lại
`analyze-requirements`. Không tự bịa.

### Bước 4: Viết code — Entity → DTO → Validator → Repository → Service → Controller → AutoMapper Profile → Swagger annotation
- Naming/error handling theo convention Architecture Design.
- Global Exception Middleware xử lý lỗi tập trung.
- Mọi method Service/Repository chạm DB/I-O: `async` + nhận `CancellationToken`.
- Comment giải thích "tại sao" cho Business Rule quan trọng.
- Giữ nguyên Business Rule đúng như BD/DD, kể cả tiểu tiết.
**verify:** code build được, khớp scaffold đã duyệt, Entity khớp đúng
ERD đã Schema Lock (không lệch cột/kiểu/ràng buộc nào chưa qua Bước 2).

### Bước 5: Viết Unit Test cho Service layer
Happy path, input rỗng/null, input biên, lỗi nghiệp vụ theo Business
Rule. Có toolchain → chạy thử, ghi kết quả thật. Không có → ghi rõ giới
hạn, không Pass giả.

### Bước 6: Auto Self-Review (UC ↔ Code)
Bảng theo Mục 4b của `output-format.md`. Không hàm mồ côi, không thiếu UC.

### Bước 7: Generalization Check (Mục 4c)
Đủ 5 nhóm, không bỏ nhóm nào mà không ghi lý do N/A.

### Bước 8: Tự Review kỹ thuật
Compile/Null Reference/Async Issue/SQL Injection/Performance
(N+1)/Naming/Dependency (chiều Controller→Service→Repository)/Security
(JWT Role đúng theo bảng Actor). Phát hiện lỗi → sửa ngay tại Bước 4.

### Bước 9: Quyết định tiếp tục
- Còn Fail hoặc Generalization Check thiếu nhóm không N/A hợp lệ:
  - `backend_retry < 3` → quay lại Bước 4, `backend_retry += 1`, ghi
    `modules_status[<module>].backend = "verify_fail"`.
  - `backend_retry >= 3` → **DỪNG CỨNG**, không tự sửa thêm. Báo rõ
    module nào, lỗi gì đã thử sửa 3 lần vẫn còn, để người dùng quyết
    định (có thể do chính Requirement/Architecture chưa đủ rõ, không
    phải lỗi viết code đơn thuần) — không đếm là input ngoài phạm vi.
- Pass → ghi `modules_status[<module>].backend = "pass"`,
  `backend_retry = 0`, in "Backend module <tên>: hoàn tất", báo
  Orchestrator chuyển sang `generate-frontend-module` cho module này.

## Quy tắc

### MUST
- Generate đúng 1 module tại một thời điểm, đúng thứ tự.
- Đối chiếu Entity với ERD đã Schema Lock TRƯỚC khi viết code (Bước 2)
  — không phát hiện lệch sau khi đã code xong.
- Dẫn chiếu UC Detail cho mọi quyết định không hiển nhiên.
- In Step Log ở mọi bước 0-9.
- Self-Review + Generalization Check cho mọi module, kể cả khi Pass.
- Dừng cứng sau 3 lần tự sửa không thành công, không lặp vô hạn.

### MUST NOT
- Không generate module chưa có trong Module Build Order.
- Không tự sửa/mở rộng Entity khi phát hiện lệch ERD — phải báo Schema
  Drift, không tự quyết định thay `design-architecture.md`.
- Không tự sửa khi đang ở vai QA (Bước 8) mà không quay lại Bước 4 minh bạch.
- Không đánh dấu Pass khi còn khác biệt logic chưa giải thích.
- Không bịa API/package .NET chưa xác minh tồn tại.
- Không bỏ qua Role-based authorization theo Overview.

## Xử lý lỗi
- UC thiếu chi tiết để code → dừng, báo rõ, không đoán.
- Entity lệch ERD đã Schema Lock → dừng, báo Schema Drift, không tự sửa.
- Không có toolchain test tự động → ghi rõ giới hạn, không Pass giả.
- Vượt ngưỡng 3 lần retry → dừng, báo người dùng, không tự lặp thêm.
