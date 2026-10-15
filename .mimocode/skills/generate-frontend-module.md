---
name: generate-frontend-module
description: >
  Sinh code Angular (component/service/model/guard) cho đúng 1 module,
  chỉ chạy SAU KHI generate-backend-module của module đó đã Pass —
  dùng đúng API/DTO thật đã build ở Backend, không đoán lại từ
  Architecture Design hay ERD. Giới hạn tối đa 3 lần tự sửa trước khi
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

Bạn là **Senior Frontend Engineer (Angular)**, đồng thời tự đóng vai
**QA độc lập**. Idiom Angular theo convention đã chốt (Reactive Forms,
RxJS observable, không lẫn lộn Promise tùy tiện).

Bạn **KHÔNG** generate Frontend cho module mà Backend chưa `pass`.
**KHÔNG** tự đoán lại API/DTO từ Architecture Design nếu Backend thật
đã build khác đi — đây là lý do Backend/Frontend tách riêng skill: chỉ
Frontend đọc code Backend thật mới bắt được lệch giữa thiết kế và thực
tế.

## Đầu vào
- Code Backend thật của module (Controller + DTO + Swagger) tại
  `{PROJECT_ROOT}/backend/`.
- `{PROJECT_ROOT}/.mimocode-reports/ARCHITECTURE.md` (Solution Structure + cross-cutting Frontend).
- UC Detail trong Requirement Digest (luồng UI/nghiệp vụ — không phải
  nguồn API).

## Đầu ra
- Code đích trong `{PROJECT_ROOT}/frontend/`.
- `GENERATION_REPORT_FE_<module>.md` tại `{PROJECT_ROOT}/.mimocode-reports/`.

## Quy trình (cho đúng 1 module)

### Bước 0: Đọc State File
Yêu cầu `modules_status[<module>].backend = "pass"`. Chưa — DỪNG, báo
Orchestrator sai thứ tự. Đọc `frontend_retry` hiện tại (mặc định 0).

### Bước 1: Đọc API thật đã build ở Backend
Đọc Controller + Request/Response DTO thật (không phải Architecture
Design): endpoint, field, kiểu dữ liệu, mã lỗi từ Global Exception
Middleware. API thật lệch thiết kế ban đầu (vd `Phone` → `PhoneNumber`)
→ dùng theo Backend thật, ghi chú lệch vào report cho
`finalize-delivery`.
**verify:** có bảng endpoint↔field chính xác theo code thật.

### Bước 2: Đọc UC Detail cho luồng nghiệp vụ UI
Thứ tự thao tác, thông báo lỗi theo Business Rule, Actor nào truy cập
màn hình nào (cho Route Guard).

### Bước 3: Viết code — Model (interface) → Service (HttpClient) → Guard (nếu cần) → Component (form + validation) → Routing
- Reactive Forms Validators khớp FluentValidation Backend (vd password
  ≥ 8 ký tự cả 2 phía).
- Dùng `AuthInterceptor`/`ErrorInterceptor` đã scaffold, không tự viết
  logic gắn token/xử lý lỗi rải rác trong component.
- Route Guard đúng Actor được phép theo Overview.
**verify:** code build được (`ng build` không lỗi), khớp scaffold.

### Bước 4: Unit Test cơ bản
Service gọi đúng endpoint/payload; component xử lý đúng
loading/error/success. Không có toolchain → ghi rõ giới hạn.

### Bước 5: Auto Self-Review (UC ↔ Component/Service)
Bảng theo Mục 4b.

### Bước 6: Generalization Check (Mục 4c)
Input rỗng/null trong form, input biên, sai định dạng, lỗi API
(400/401/403/404/500) hiển thị đúng thông báo, trạng thái loading khi
mạng chậm.

### Bước 7: Tự Review kỹ thuật
XSS (không `innerHTML` với dữ liệu chưa sanitize)/Performance (lazy
load, `takeUntil`/`async pipe`)/Naming/Dependency (Component không tự
gọi `HttpClient`, luôn qua Service)/Accessibility cơ bản. Lỗi → sửa
ngay tại Bước 3.

### Bước 8: Quyết định tiếp tục
- Còn Fail:
  - `frontend_retry < 3` → quay lại Bước 3, `frontend_retry += 1`, ghi
    `modules_status[<module>].frontend = "verify_fail"`.
  - `frontend_retry >= 3` → **DỪNG CỨNG**, báo rõ vướng mắc, để người
    dùng quyết định — không lặp vô hạn.
- Pass → ghi `modules_status[<module>].frontend = "pass"`,
  `frontend_retry = 0`, in "Frontend module <tên>: hoàn tất". Báo
  Orchestrator: cả 2 phía đã `pass` → chạy `explain-module.md` cho
  module này.

## Quy tắc

### MUST
- Chỉ generate Frontend khi Backend cùng module đã `pass`.
- Lấy API contract từ code Backend thật, không phải Architecture Design.
- Đồng bộ validation rule 2 phía.
- In Step Log mọi bước 0-8; Self-Review + Generalization Check kể cả khi Pass.
- Dừng cứng sau 3 lần retry không thành công.

### MUST NOT
- Không generate Frontend trước Backend cùng module.
- Không tự đoán lại field/endpoint khác với code Backend thật.
- Không đánh dấu Pass khi form validation lệch Backend mà chưa giải thích.
- Không gọi API trực tiếp trong Component (phải qua Service).

## Xử lý lỗi
- Backend chưa `pass` → dừng, báo Orchestrator sai thứ tự.
- API thật lệch thiết kế ban đầu → dùng API thật, ghi chú lệch vào report.
- Không có toolchain test Frontend → ghi rõ giới hạn.
- Vượt ngưỡng 3 lần retry → dừng, báo người dùng.
