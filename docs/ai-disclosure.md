# Bảng khai báo sử dụng công cụ AI

| Công cụ | Áp dụng ở phần nào | Dùng vào việc gì | Đã kiểm chứng / chỉnh sửa thế nào |
|---|---|---|---|
| Claude (Anthropic) — trợ lý AI dạng chat | Mục 1 — User Story, tiêu chí chấp nhận, NFR, quy tắc, bảng truy vết | Yêu cầu soạn và sửa User Story theo INVEST/MoSCoW, viết tiêu chí Given–When–Then, đề xuất NFR có ngưỡng số cho luồng L2 | Em chọn luồng L2 và duyệt phạm vi/WON'T; đối chiếu với case study và tài liệu buổi 3–6; chạy 6 phép kiểm nhất quán tự động trên toàn hồ sơ |
| Claude (Anthropic) | Mục 2 — Use Case Diagram và đặc tả UC3 | Đề xuất danh sách use case, quan hệ include/extend và soạn luồng chính/luồng ngoại lệ; sinh file .drawio | Kiểm tra đường nối không xuyên qua use case khác; đối chiếu tên use case với bảng thuật ngữ; sửa lỗi bố cục phát hiện ở bản buổi 4 |
| Claude (Anthropic) | Mục 3 — Sơ đồ kiến trúc và 3 câu lập luận | Soạn sơ đồ 4 lớp và 3 câu lập luận theo khuôn "Vì NFRx…, tôi chọn…, đánh đổi là…" | Đối chiếu từng câu với mã NFR, ngưỡng và đánh đổi; kiểm tra tên bảng ở lớp Lưu trữ khớp ERD |
| Claude (Anthropic) | Mục 4 — ERD, SQL DDL, hợp đồng API; db/schema.sql | Đề xuất 5 bảng, quan hệ, index và viết DDL skeleton, bảng endpoint | Kiểm tra cú pháp DDL bằng bộ phân tích PostgreSQL (pglast); đối chiếu mọi trường API với cột ERD; soi 5 lỗi ERD |
| Claude (Anthropic) | Mục 5 — Wireframe 3 màn hình | Đề xuất bố cục 3 màn hình và vị trí hiển thị các thông báo ngoại lệ; vẽ ảnh wireframe | Đối chiếu từng trường hiển thị với cột trong ERD và từng luồng ngoại lệ với vị trí thông báo |
| Claude (Anthropic) | Thiết lập môi trường, README, định dạng tài liệu | Hướng dẫn cài Git/Python/PostgreSQL, tạo repo GitHub, soạn README và gộp PDF | Em tự cài đặt và chạy smoke test trên máy (buổi 2) |
| GitHub Copilot | BT2 — API phân loại, test ràng buộc PostgreSQL và README | Đề xuất cấu trúc FastAPI theo 4 lớp, luật phân loại từ khóa, test và hướng dẫn chạy | Em đối chiếu hợp đồng API và DDL; phần này được kiểm thử lại trong lần cập nhật BT2 ở dưới |
| Copilot SDK trong VS Code | BT2 — CRUD khách hàng/thiết bị, đóng gói, giao diện, README, test và bản Word | Hỗ trợ tạo service/repository/API, UI ba màn hình, Docker/Compose, tài liệu và test hồi quy | Lần kiểm tra khi chuẩn bị BT2 từng ghi nhận 36 test PASS; đây là kết quả lịch sử, không phải kết quả mới nhất của BT3. |
| Copilot SDK trong VS Code | BT3 — đối chiếu tài liệu Buổi 13–16, README, API reference, Deployment Guide, checklist bảo mật và bản Word làm việc | Hỗ trợ tổng hợp yêu cầu, soạn/cập nhật hướng dẫn chạy máy mới, phát hiện điểm không khớp giữa API contract và routes hiện có, đề xuất các bước hoàn thiện | Sinh viên cần đối chiếu từng nội dung với mã và kết quả thật. Lần chạy `python -m pytest test -q` ngày 13/11/2026 trong clone repo ghi nhận 39 passed, 0 failed, 0 skipped; Docker CLI không có trong môi trường lúc kiểm tra nên chưa xác minh clean-room Docker. Bản Word và tài liệu còn các mục cần cập nhật ảnh/minh chứng trước khi nộp. |

Kiểm chứng thủ công đã thực hiện trước đó: `db/schema.sql` được chạy trên PostgreSQL (16, môi trường thử) cùng các phép thử ràng buộc nghiệp vụ. Lần chạy toàn bộ suite ngày 13/11/2026 ghi nhận 39 passed, 0 failed, 0 skipped; có một cảnh báo deprecation từ `starlette.testclient` về `httpx`. Đây là kết quả kiểm thử Python/PostgreSQL trong clone repo, không phải xác nhận Docker clean-room.

**Tôi xác nhận đã đọc, hiểu và chịu trách nhiệm về toàn bộ nội dung nộp.**
Nguyễn Hoàng Minh Nhật — MSSV 2374802010363 — 09/10/2026
