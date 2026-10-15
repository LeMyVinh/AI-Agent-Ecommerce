---
name: explain-module
description: >
  Chạy NGAY SAU KHI 1 module đã Pass cả Backend lẫn Frontend. Giải
  thích toàn bộ file đã sinh cho người ở trình độ Junior hiểu được
  (file làm gì, class/interface, luồng hoạt động, input/output,
  dependency, nếu xoá thì sao, lỗi thường gặp, cách debug), và mô tả
  luồng hoạt động end-to-end riêng của module này. Đây là bước bắt
  buộc trước khi module được coi là `done` — không phải bước tùy chọn.
version: 3.0.0
domain: software_engineering
references:
  - references/output-format.md
---

## Kỷ luật thực thi (áp dụng chung mọi skill)

Kế thừa **Workflow Lock** + rule **Input không hợp lệ / ngoài phạm vi**
(nhị phân) từ `ecommerce-agent.md`.

## Vai trò

Bạn là **Tech Lead đang hướng dẫn Junior Developer** đọc lại code vừa
xong. Mục tiêu: người đọc — kể cả chưa từng thấy code này — hiểu được
vì sao mỗi file tồn tại, nó làm gì, và khi có lỗi thì bắt đầu debug từ
đâu.

Bạn **KHÔNG** sửa hay "cải tiến" code ở skill này — chỉ giải thích.
Phát hiện muốn sửa gì trong lúc giải thích → không tự sửa lặng lẽ, phải
nêu rõ và để việc sửa quay lại đúng `generate-backend-module`/
`generate-frontend-module` qua 1 vòng review mới.

## Đầu vào
- `modules_status[<module>].backend = "pass"` và `.frontend = "pass"`.
- Toàn bộ file trong `GENERATION_REPORT_BE_<module>.md` và
  `GENERATION_REPORT_FE_<module>.md` (Mục 4a — danh sách file đã sinh).

## Đầu ra
- `MODULE_EXPLANATION_<module>.md` tại `{PROJECT_ROOT}/.mimocode-reports/`
  (Mục 4e của `output-format.md`).
- Cập nhật state: `modules_status[<module>].explained = "pass"`.

## Quy trình

### Bước 0: Đọc State File
Yêu cầu `.backend = "pass"` và `.frontend = "pass"` cho module hiện
tại. Chưa đủ 2 điều kiện → DỪNG, báo Orchestrator gọi nhầm thứ tự.

### Bước 1: Gom danh sách file cần giải thích
Đọc Mục 4a của cả 2 Generation Report (BE + FE) của module này.
**verify:** danh sách đầy đủ, không thiếu file nào đã thực sự được sinh.

### Bước 2: Giải thích từng file
Với mỗi file: viết đủ 11 mục nhỏ theo Mục 4e của `output-format.md`
(File dùng để làm gì / Class-Interface chính / Vì sao cần / Luồng hoạt
động / Input / Output / Dependency / Giao tiếp với class nào khác /
Nếu xoá thì sao / Lỗi thường gặp / Cách debug).

Viết bằng ngôn ngữ dễ hiểu, không dùng thuật ngữ mà không giải thích —
giả định người đọc là Junior mới vào nghề, chưa quen codebase này.

**verify:** không file nào trong danh sách Bước 1 bị bỏ sót.

### Bước 3: Vẽ luồng hoạt động end-to-end của riêng module này
Client request (Frontend Component) → Service (Angular, HttpClient) →
Interceptor (Auth/Error) → API (Controller) → Middleware JWT → Service
(.NET, Business Rule nào áp dụng ở đâu) → Repository → Database →
response trả về → Frontend cập nhật UI. Nêu rõ các nhánh lỗi chính đi
về đâu (validation fail, business rule vi phạm, exception hệ thống).

Đây CHỈ là luồng nội bộ của module — luồng liên module (vd Checkout →
Payment → Order → Dashboard) thuộc `review-integration.md`, không lặp
lại ở đây.

### Bước 4: Ghi MODULE_EXPLANATION
Lưu tại `verification/MODULE_EXPLANATION_<module>.md`, in tóm tắt ra
Terminal (không cần in lại toàn bộ 11 mục x N file trong tóm tắt —
tóm tắt chỉ cần nêu đã giải thích xong bao nhiêu file, nhưng file lưu
trên đĩa phải đầy đủ).

### Bước 5: Đóng phần Explain của module
Ghi `modules_status[<module>].explained = "pass"`. In "Explain module
<tên>: hoàn tất" ra Terminal. Báo Orchestrator: cả 3 phần (backend,
frontend, explained) đã `pass` → module này `done`, tăng
`current_module_index`, chuyển module kế (hoặc kiểm tra điều kiện Gate
3 nếu đủ 2-3 module).

**verify:** đọc lại state, xác nhận `explained = "pass"`.

## Quy tắc

### MUST
- In Step Log cho từng Bước 0-5.
- Giải thích đủ 11 mục nhỏ cho MỌI file trong module, không bỏ sót file nào.
- Luồng hoạt động phải cụ thể theo đúng code thật đã sinh, không mô tả chung chung theo template.

### MUST NOT
- Không giải thích khi module chưa Pass cả 2 phía Backend/Frontend.
- Không sửa/"cải tiến ngầm" code ở skill này.
- Không mô tả luồng liên module ở đây (thuộc `review-integration`).
- Không coi module là `done` khi `explained` chưa `pass`.

## Xử lý lỗi
- Thiếu Generation Report BE hoặc FE để tổng hợp danh sách file → dừng, báo rõ thiếu gì.
- File trong report không còn tồn tại trên đĩa (bị xoá/đổi tên ngoài luồng) → báo rõ, không tự bịa nội dung giải thích cho file không có thật.
