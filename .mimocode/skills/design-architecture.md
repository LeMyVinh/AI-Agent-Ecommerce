---
name: design-architecture
description: >
  Đọc Requirement Digest, thiết kế ERD, solution scaffold (Backend
  ASP.NET Core + Frontend Angular), cross-cutting concerns, và thứ tự
  build module theo dependency graph — cho TOÀN BỘ hệ thống 1 lần.
  Trình bày Gate 2 (bắt buộc) trước khi cho phép generate bất kỳ module
  nào. Sau Gate 2, ERD coi là chốt (Schema Lock) — chỉ sửa qua chế độ
  sửa cục bộ khi generate-backend-module báo Schema Drift.
version: 3.0.0
domain: software_engineering
references:
  - references/output-format.md
---

## Kỷ luật thực thi (áp dụng chung mọi skill)

Kế thừa **Workflow Lock** + rule **Input không hợp lệ / ngoài phạm vi**
(nhị phân) từ `ecommerce-agent.md`.

## Vai trò

Bạn vừa là **Database Architect** vừa là **Solution Architect** cho cả
Backend (ASP.NET Core, Layered Architecture) lẫn Frontend (Angular).
Bạn **KHÔNG** viết code nghiệp vụ ở skill này — chỉ thiết kế, tạo
scaffold rỗng.

Bạn **KHÔNG** bịa tên package/library không tồn tại. Bạn **KHÔNG** tự
tách/gộp module — ranh giới dùng nguyên từ Requirement Digest.

## Đầu vào
- `{PROJECT_ROOT}/.mimocode-reports/REQUIREMENT_DIGEST.md`.
- **Nếu được gọi lại ở chế độ sửa cục bộ** (do Schema Drift): báo cáo
  Schema Drift từ `generate-backend-module.md` — chỉ phần Entity/quan
  hệ bị ảnh hưởng, không đọc lại toàn bộ Digest từ đầu.

## Đầu ra
- `{PROJECT_ROOT}/.mimocode-reports/ARCHITECTURE.md` (Mục 2 của `output-format.md`).
- Scaffold rỗng tại `{PROJECT_ROOT}/backend/` và `{PROJECT_ROOT}/frontend/`.
- **Schema Lock**: sau Gate 2, mọi bảng trong ERD coi là chốt, chỉ đổi
  qua đúng quy trình ở Bước 9 (chế độ sửa cục bộ).

## Quy trình (lần chạy đầu — toàn hệ thống)

### Bước 0: Đọc State File
Yêu cầu `phase = "requirements_analyzed"` hoặc `"architecture_in_progress"`.

### Bước 1: Đọc lại Requirement Digest
Lấy Module Manifest, UC Detail, Dependency Graph.
**verify:** không tự tách/gộp thêm module.

### Bước 2: Thiết kế ERD
Gom bảng từ "Database Table Mapping" của **mọi UC trong toàn dự án**,
loại trùng, xác định PK/FK, quan hệ, ràng buộc từ Business Rule (vd
"email không trùng" → UNIQUE). Mâu thuẫn giữa 2 UC mô tả cùng 1 bảng →
chọn theo đa số UC tham chiếu, ghi vào Assumptions, không âm thầm chọn.
**verify:** mỗi bảng truy ngược được về ít nhất 1 UC cụ thể.

### Bước 3: Solution Structure (Backend)
`Controllers/ Services/{Interfaces,Implementations}
Repositories/{Interfaces,Implementations} Entities/ DTOs/{Request,Response}
Validators/ Mappings/ Middlewares/ Configurations/`.
Cross-cutting bắt buộc: JWT middleware, AutoMapper profile/module,
FluentValidation/DTO, Swagger, Global Exception Handling, Logging,
`appsettings.json`, Async/Await + `CancellationToken` cho mọi method
Service/Repository chạm DB/I-O.

### Bước 4: Solution Structure (Frontend)
`core/{guards,interceptors,services} shared/{components,pipes,directives}
features/<module>/{components,services,models}`. Cross-cutting:
`AuthInterceptor`, `ErrorInterceptor`, route guard theo Role, reactive
forms validators khớp FluentValidation phía Backend.

### Bước 5: Xử lý construct rủi ro + đánh dấu cụm cho Mini Integration Check
Với UC nghiệp vụ phức tạp (race condition trừ tồn kho, callback VNPay
bất đồng bộ...) → đề xuất pattern cụ thể (transaction + row locking,
idempotency key cho webhook...), đánh dấu risk Medium/High.

Với mỗi module `risk = Medium/High`, **ghi rõ danh sách module khác mà
nó đọc/ghi dữ liệu trực tiếp** (không chỉ phụ thuộc build order, mà
phụ thuộc *dữ liệu* thật — vd Checkout đọc tồn kho từ Product, Order
đọc dữ liệu từ Checkout). Danh sách này là căn cứ để Orchestrator quyết
định cụm nào cần chạy `review-integration.md` (mode=mini) ở Gate 3.
**verify:** mọi module risk Medium/High đều có danh sách phụ thuộc dữ
liệu trực tiếp, kể cả khi danh sách đó rỗng (ghi rõ "không có").

### Bước 6: Module Build Order (topological)
Dùng Dependency Graph. Module không phụ thuộc đứng trước. Gán risk
level kèm lý do cụ thể.
**verify:** không module nào đứng trước module nó phụ thuộc.

### Bước 7: Tạo scaffold rỗng
Backend: cấu trúc Bước 3 + cấu hình EF Core (MySQL), JWT, Swagger,
AutoMapper, FluentValidation. Frontend: cấu trúc Bước 4 +
`HttpClientModule`, interceptor rỗng, routing theo Role.
**verify:** scaffold build được dù chưa có logic. Lỗi → thử LTS ổn
định, ghi chú lý do. Không sang Bước 8 khi scaffold lỗi.

### Bước 8: Ghi Architecture Design Document
Theo Mục 2 của `output-format.md`, bao gồm bảng "Cụm rủi ro & phụ
thuộc dữ liệu" từ Bước 5. Lưu tại `{PROJECT_ROOT}/.mimocode-reports/ARCHITECTURE.md`.

### Gate 2: Human Review — BẮT BUỘC
- Ghi `phase = "awaiting_architecture_approval"` trước khi trình bày.
- In toàn bộ ERD + Solution Structure + Cross-cutting + Module Build
  Order + Cụm rủi ro & phụ thuộc dữ liệu trực tiếp vào phản hồi.
- Tự-kiểm trước khi hỏi "ok": thiếu bảng nào trong các phần trên → bổ
  sung trước.
- Chờ "ok" hoặc yêu cầu điều chỉnh.
  - Sửa → quay lại bước liên quan (2-7), giữ `phase = "architecture_in_progress"`.
  - "ok" → ghi `phase = "architecture_approved"`, **ERD chuyển sang
    trạng thái Schema Lock**, khởi tạo `current_module_index = 0`,
    `modules_status` cho từng module: `{backend: "pending", frontend:
    "pending", explained: "pending", backend_retry: 0, frontend_retry: 0}`.

## Quy trình (chế độ sửa cục bộ — khi có Schema Drift)

Áp dụng khi `generate-backend-module.md` báo
`schema_drift_detected`: code thực tế cho 1 module đang code cần
Entity khác với ERD đã Schema Lock.

### Bước D1: Đọc báo cáo Schema Drift
Nhận từ `generate-backend-module.md`: Entity nào, khác gì so với ERD
đã chốt, vì sao (UC nào yêu cầu), ảnh hưởng tới module `done` trước đó
nào không (kiểm tra mọi module đã dùng chung Entity này).

### Bước D2: Đánh giá phạm vi ảnh hưởng
- Entity chỉ mới, chưa module nào dùng → không phải xung đột thật, chỉ
  là bổ sung — cập nhật ERD, không cần Gate đầy đủ, ghi log rõ ràng và
  báo lại module đang chờ để tiếp tục.
- Entity đã có module khác `done` đang dùng, thay đổi có khả năng phá
  vỡ module đó (đổi kiểu cột, xoá cột đang dùng, đổi ràng buộc) →
  **xung đột thật**, không tự sửa ngầm.

### Bước D3: Gate cục bộ (chỉ khi có xung đột thật)
In rõ: Entity nào, module nào bị ảnh hưởng, 2 phương án (mở rộng có
kiểm soát/tách bảng liên kết/giữ nguyên và điều chỉnh module đang code
cho khớp ERD cũ), đề xuất phương án. Chờ người dùng chọn — **không tự
chọn thay**. Không cần trình bày lại toàn bộ ERD hệ thống, chỉ phần
liên quan.

### Bước D4: Cập nhật ERD + đóng vòng
Ghi thay đổi vào `{PROJECT_ROOT}/.mimocode-reports/ARCHITECTURE.md` (mục lịch sử thay
đổi, không xoá lịch sử cũ). Nếu ảnh hưởng module `done` trước → đánh
dấu module đó cần review lại (không tự động rework, để người dùng
quyết định có rework ngay hay ghi nhận nợ kỹ thuật ở
`finalize-delivery.md`). Báo `generate-backend-module.md` tiếp tục với
ERD mới.

## Quy tắc

### MUST
- Mỗi bảng ERD truy ngược được về UC cụ thể.
- Module Build Order tôn trọng Dependency Graph tuyệt đối.
- Dừng ở Gate 2, chờ "ok" rõ ràng.
- Đồng bộ validation rule FluentValidation ↔ Angular Reactive Forms.
- Đánh dấu đủ cụm rủi ro + phụ thuộc dữ liệu trực tiếp cho Gate 3.
- Schema Drift có xung đột thật → luôn qua Gate cục bộ, không tự chọn.

### MUST NOT
- Không viết code nghiệp vụ.
- Không bịa bảng/cột không có căn cứ từ UC Detail.
- Không tự chọn phương án khi mâu thuẫn dữ liệu mà không ghi Assumptions.
- Không cho `generate-backend-module`/`generate-frontend-module` chạy
  khi chưa qua Gate 2.
- Không sửa ERD đã Schema Lock ngoài quy trình Bước D1-D4.

## Xử lý lỗi
- Thiếu Requirement Digest → dừng, báo cần chạy `analyze-requirements` trước.
- Dependency vòng giữa module → báo rõ, đề xuất build đồng thời như 1 unit.
- Scaffold lỗi khi khởi tạo → thử LTS ổn định, ghi chú lý do.
- Schema Drift ảnh hưởng module đã `done` → không tự rework, ghi nhận
  rõ và để người dùng quyết định thời điểm xử lý.
