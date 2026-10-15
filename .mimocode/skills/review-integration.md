---
name: review-integration
description: >
  Review điểm nối giữa các module — không review lại logic nội bộ
  từng module (đã Pass ở generate-backend-module/generate-frontend-module).
  Hỗ trợ 2 mode: `mini` — chạy ngay trong Gate 3 của Orchestrator cho
  1 cụm module risk Medium/High vừa hoàn tất, chỉ soi đúng điểm nối dữ
  liệu đã được design-architecture đánh dấu, nhanh và hẹp; `full` —
  chạy đúng 1 lần sau khi TOÀN BỘ module đã done, review build/security/
  performance/luồng liên module toàn hệ thống. Mode mini không thay thế
  mode full — mini bắt lỗi sớm ở đúng chỗ rủi ro cao, full vẫn là cổng
  kiểm tra cuối cùng bắt buộc.
version: 1.0.0
domain: software_engineering
references:
  - references/output-format.md
---

## Kỷ luật thực thi (áp dụng chung mọi skill)

Kế thừa **Workflow Lock** + rule **Input không hợp lệ / ngoài phạm vi**
(nhị phân) từ `ecommerce-agent.md`.

## Vai trò

Bạn là **QA Lead / Release Engineer** — kiểm tra hệ thống ở mức tổng
thể, đặc biệt **điểm nối giữa các module**, nơi lỗi thường ẩn nhất (vd
Order tạo ra nhưng Dashboard không cập nhật doanh thu, Checkout trừ tồn
kho nhưng Payment fail không hoàn lại).

Bạn **KHÔNG** sửa code nghiệp vụ ở đây — phát hiện lỗi liên module thì
báo rõ để quay lại đúng module qua `generate-backend-module`/
`generate-frontend-module`. Bạn **KHÔNG** review lại logic nội bộ từng
module đã Pass (tránh trùng việc).

---

## MODE: mini (gọi từ Gate 3 của Orchestrator)

### Khi nào được gọi
Orchestrator gọi mode này ngay sau khi 1 cụm 2-3 module vừa hoàn tất cả
3 phần (backend/frontend/explained), **và** cụm đó chứa ít nhất 1 module
`risk = Medium/High` có danh sách "phụ thuộc dữ liệu trực tiếp" (đã ghi
ở Bước 5 của `design-architecture.md`) trỏ tới module khác đã `done`.

### Đầu vào
- Danh sách đúng cặp module/điểm nối do Orchestrator truyền vào (không
  tự ý mở rộng phạm vi ra module khác trong cụm không liên quan).
- Code thật của các module liên quan (`{PROJECT_ROOT}/backend/`,
  `{PROJECT_ROOT}/frontend/`).
- Phần "Cụm rủi ro & phụ thuộc dữ liệu" trong `{PROJECT_ROOT}/.mimocode-reports/ARCHITECTURE.md`.

### Quy trình

**Bước m1: Xác định đúng phạm vi hẹp**
Chỉ soi đúng điểm nối đã được truyền vào — không build toàn solution,
không chạy Security/Performance review toàn hệ thống (việc đó để dành
`mode = full`).
**verify:** phạm vi kiểm tra khớp đúng danh sách được truyền, không
thừa không thiếu.

**Bước m2: Kiểm tra điểm nối dữ liệu**
Với mỗi cặp module: module sau có đọc đúng dữ liệu module trước ghi
không (field, kiểu, đơn vị); có transaction/rollback nhất quán khi 1
bên fail giữa chừng không (vd Checkout fail thì tồn kho có được hoàn
lại không). Bằng chứng cụ thể: trích dẫn file:hàm hoặc test chạy thật.
**verify:** mỗi điểm nối trong phạm vi đều có kết luận kèm bằng chứng.

**Bước m3: Ghi kết quả**
In bảng ngắn (điểm nối | Pass/Fail | bằng chứng) trực tiếp vào phản
hồi — không cần file report riêng, chỉ cần ghi vào log Gate 3 của
Orchestrator.
- Fail → báo rõ module/điểm nối cần sửa, giữ nguyên `phase` hiện tại
  của Orchestrator (không cho cụm tiếp theo chạy), để quay lại đúng
  `generate-backend-module`/`generate-frontend-module`.
- Pass → báo Orchestrator tiếp tục cụm kế tiếp.

### Quy tắc riêng mode mini
- **MUST NOT** build toàn solution, chạy Security/Performance toàn hệ
  thống, hoặc review lại module không nằm trong danh sách được truyền.
- **MUST NOT** thay thế `mode = full` — mọi kết quả Pass ở đây vẫn cần
  được xác nhận lại nhanh (không lặp lại chi tiết) ở `mode = full`.

---

## MODE: full (chạy đúng 1 lần, sau khi hết toàn bộ module)

### Đầu vào
- Toàn bộ code các module `done`.
- `{PROJECT_ROOT}/.mimocode-reports/ARCHITECTURE.md`, mọi `GENERATION_REPORT_BE/FE_*`,
  `MODULE_EXPLANATION_*`.
- Kết quả các lần `mode = mini` đã chạy trước đó (nếu có) — dùng để
  không lặp lại review chi tiết các điểm nối đã Pass, chỉ xác nhận
  nhanh còn hiệu lực.

### Đầu ra
- `INTEGRATION_REVIEW.md` tại `{PROJECT_ROOT}/.mimocode-reports/`.

### Quy trình

**Bước 0: Đọc State**
Yêu cầu `phase = "all_modules_done"` — mọi `modules_status[*]` (cả
`backend`, `frontend`, `explained`) đều `"pass"`. Còn dở dang → DỪNG,
báo rõ module nào.

**Bước 1: Build toàn solution**
Backend: build toàn solution, bắt lỗi tham chiếu chéo giữa module
(thiếu đăng ký DI...). Frontend: build toàn workspace, kiểm tra routing
tổng hợp không đè route module này lên module khác.
**verify:** build Pass cả 2 phía, không warning nghiêm trọng.

**Bước 2: Kiểm tra luồng nghiệp vụ liên module**
Dựa trên Overview mục "Quy trình nghiệp vụ tổng quan" — dựng lại luồng
thật: Duyệt sản phẩm → Đăng nhập/Đăng ký → Cart → Wishlist→Cart →
Checkout → COD/VNPay → Order tạo → (VNPay) callback cập nhật thanh
toán → Order Management phản ánh đúng trạng thái → Dashboard cập nhật
số liệu → Review chỉ tạo được sau khi Order đã nhận hàng.

Với các điểm nối **đã Pass ở mode mini trước đó** → xác nhận nhanh còn
hiệu lực (code không đổi từ lúc đó), không review lại chi tiết. Với
điểm nối **chưa từng qua mini check** (module risk Low, hoặc phụ thuộc
không được đánh dấu risk cao) → review đầy đủ như bình thường.
**verify:** mỗi điểm nối có bằng chứng cụ thể, không kết luận chung chung.

**Bước 3: Security review toàn hệ thống**
- JWT: mọi endpoint không public có `[Authorize]` đúng Role theo Overview.
- CORS: chỉ origin Frontend thật, không `AllowAnyOrigin` kèm `AllowCredentials`.
- SQL Injection: toàn bộ qua EF Core/parameterized.
- XSS: Frontend không render dữ liệu người dùng qua `innerHTML` chưa sanitize (đặc biệt Review/Rating).
- Secrets: connection string/JWT secret ở `appsettings`/biến môi trường, không hardcode.

**Bước 4: Performance review toàn hệ thống**
N+1 query ở trang danh sách; pagination cho mọi API danh sách lớn;
index DB cho cột hay filter/sort; Dashboard aggregate không tính lại
toàn bộ mỗi lần gọi nếu dữ liệu lớn — đề xuất cache nếu cần.

**Bước 5: Đối chiếu Schema Lock**
Xác nhận mọi Schema Drift phát sinh trong quá trình build (nếu có) đã
được xử lý xong qua `design-architecture.md`, không còn Entity nào ở
trạng thái chờ quyết định.
**verify:** không còn báo cáo Schema Drift nào dang dở.

**Bước 6: Viết Integration Test**
End-to-end tối thiểu cho luồng chính ở Bước 2 nếu có toolchain. Không
có → ghi rõ giới hạn, review thủ công có trích dẫn cụ thể.

**Bước 7: Ghi Integration Review Report**
Theo Mục 5 của `output-format.md`.

**Bước 8: Quyết định**
- Có mục Fail → DỪNG, báo rõ module/điểm nối cần sửa ở skill generate
  tương ứng, giữ `phase = "all_modules_done"`.
- Toàn bộ Pass → ghi `phase = "integration_reviewed"`, báo Orchestrator
  gọi `finalize-delivery.md`.

### Quy tắc riêng mode full

**MUST**
- Chỉ chạy khi mọi module đã `pass` cả 3 phần.
- Kiểm tra đủ 5 nhóm: build, luồng liên module, security, performance,
  Schema Lock — không bỏ nhóm nào.
- Mỗi kết luận Pass/Fail có bằng chứng cụ thể.
- Tận dụng kết quả mini check trước đó để không lặp lại công sức,
  nhưng vẫn xác nhận lại còn hiệu lực.

**MUST NOT**
- Không tự sửa code nghiệp vụ.
- Không đánh dấu "hoàn tất" khi còn mục Fail.
- Không review lại logic nội bộ từng module đã Pass ở bước generate.
- Không bỏ qua điểm nối chỉ vì "có thể" đã được mini check — nếu không
  chắc đã được check, coi như chưa và review đầy đủ.

## Xử lý lỗi
- Còn module chưa `pass` đủ 3 phần → dừng, yêu cầu hoàn tất trước
  (áp dụng cho mode full).
- Phát hiện lỗi liên module (mini hoặc full) → báo rõ, không tự vá tạm.
- Không có toolchain test tích hợp → ghi rõ giới hạn, review thủ công thay thế.
