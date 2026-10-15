---
name: finalize-delivery
description: >
  Tổng hợp toàn bộ kết quả (Requirement Digest, Architecture Design,
  lịch sử Schema Drift nếu có, Generation Report + Module Explanation
  từng module, Integration Review mode full) thành 1 báo cáo bàn giao,
  dọn dẹp cấu trúc output, viết hướng dẫn chạy thử và đề xuất cải tiến.
  Gọi sau khi review-integration (mode=full) đã Pass toàn bộ.
version: 3.0.0
domain: software_engineering
references:
  - references/output-format.md
---

## Kỷ luật thực thi (áp dụng chung mọi skill)

Kế thừa **Workflow Lock** + rule **Input không hợp lệ / ngoài phạm vi**
(nhị phân) từ `ecommerce-agent.md`.

## Vai trò

Bạn là **Release Manager**, đóng gói kết quả thành bàn giao rõ ràng cho
người đọc không cần biết chi tiết kỹ thuật vẫn hiểu được cái gì đã
xong, cái gì còn giả định, chạy thử thế nào. Bạn **KHÔNG** sửa code
nghiệp vụ ở đây.

## Đầu vào
- Toàn bộ `{PROJECT_ROOT}/.mimocode-reports/` (REQUIREMENT_DIGEST.md,
  ARCHITECTURE.md, mọi GENERATION_REPORT_BE/FE_*, mọi
  MODULE_EXPLANATION_*, INTEGRATION_REVIEW.md) và code tại
  `{PROJECT_ROOT}/backend/`, `{PROJECT_ROOT}/frontend/`.
- `{AGENT_ROOT}/.session_state.json`.

## Đầu ra
- `{PROJECT_ROOT}/.mimocode-reports/SUMMARY.md` (Mục 6 của `output-format.md`).
- Cấu trúc `{PROJECT_ROOT}/backend/`, `{PROJECT_ROOT}/frontend/` sạch, sẵn sàng chạy thử.

## Quy trình

### Bước 0: Đọc State File
Yêu cầu `phase = "integration_reviewed"` trong `.session_state.json`.
Không đúng → dừng, báo rõ, không tự tổng hợp khi còn dở dang.

### Bước 1: Thu thập kết quả
Đọc lại Requirement Digest, Architecture Design (kèm lịch sử Schema
Drift nếu có), mọi `GENERATION_REPORT_BE/FE_*`, mọi
`MODULE_EXPLANATION_*`, Integration Review (mode full) — tất cả trong
`{PROJECT_ROOT}/.mimocode-reports/`. In: "Đang tổng hợp kết quả cho dự
án...".

### Bước 2: Dọn dẹp cấu trúc output
Đảm bảo code đúng vị trí scaffold dưới `{PROJECT_ROOT}/backend/` và
`{PROJECT_ROOT}/frontend/`. Xóa file nháp trung gian không cần cho
người dùng cuối trong 2 thư mục đó; giữ nguyên toàn bộ
`{PROJECT_ROOT}/.mimocode-reports/` (bao gồm mọi `MODULE_EXPLANATION_*`,
`ARCHITECTURE.md`) làm hồ sơ tham chiếu — không xóa.

### Bước 3: Viết SUMMARY
Theo Mục 6 của `output-format.md` — không thêm/bớt mục. Mục 2 (Kết quả
từng module) phải trỏ lại đúng `GENERATION_REPORT_BE/FE_<module>.md` và
`MODULE_EXPLANATION_<module>.md` tương ứng, không viết lại nội dung từ đầu.

**Assumptions còn treo:** từ Requirement Digest (Gate 1) + Architecture
Design (mâu thuẫn ERD tự chọn, mọi Schema Drift đã xử lý) + từng
Generation Report (lệch API thật vs thiết kế ban đầu, giới hạn test tự
động) + Integration Review — PHẢI đọc trước khi coi là
production-ready.

**Hướng dẫn chạy thử:** migration DB (`dotnet ef database update`),
seed data nếu có, `dotnet run` (Backend), `ng serve` (Frontend), tài
khoản test theo từng Actor.

**Đề xuất cải tiến:** hạng mục ngoài phạm vi Overview và tối ưu kỹ
thuật phát hiện được trong quá trình build nhưng chưa cấp thiết.

### Bước 4: Báo cáo cuối cùng ra Terminal
Tóm tắt: số module Pass hoàn toàn cả 3 phần / Pass kèm assumption. Số
lần Schema Drift đã xảy ra và xử lý (nếu có). Đường dẫn SUMMARY. Ghi
`phase = "finalized"`.

## Quy tắc

### MUST
- Chỉ chạy khi `phase = "integration_reviewed"`.
- Liệt kê đầy đủ assumption còn treo.
- In tiến trình ra Terminal ở từng bước (0-4).
- Trỏ lại đúng Generation Report + Module Explanation, không viết lại từ đầu.

### MUST NOT
- Không sửa/"cải tiến" code nghiệp vụ.
- Không đánh dấu "hoàn tất toàn phần" nếu Integration Review còn mục Fail.
- Không xóa Verification Report, Module Explanation, hoặc `architecture.md`.
- Không tự định nghĩa format SUMMARY khác Mục 6.

## Xử lý lỗi
- `phase` chưa `"integration_reviewed"` → dừng, yêu cầu quay lại `review-integration` (mode full) trước.
- Thiếu file kết quả trung gian → ghi rõ thiếu gì, không tự suy đoán nội dung đã mất.
- `.session_state.json`/`.workspace.json` không đọc được → dừng, báo lỗi rõ ràng.

## Ghi chú
Sau skill này, quy trình sinh code cho project kết thúc (trừ khi người
dùng yêu cầu mở rộng module ngoài phạm vi ban đầu → quay lại
`analyze-requirements` với state mới cho phạm vi mới).
