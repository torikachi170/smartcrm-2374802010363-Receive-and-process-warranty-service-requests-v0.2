# Bug Log

## BUG-001 · Major — Tra cứu khách hàng lỗi khi truy vấn PostgreSQL

| Dòng | Nội dung |
|---|---|
| 1. Mã và mức độ | BUG-001 · Major — endpoint danh sách/tra cứu khách không hoạt động với PostgreSQL |
| 2. Phát hiện bởi | `test_repository_crud_and_referenced_delete_guards` |
| 3. Triệu chứng | PostgreSQL báo `AmbiguousParameter` khi tìm khách bằng số điện thoại |
| 4. Cách tái lập | Cấu hình `TEST_DATABASE_URL`, chạy test repository với `phone` có giá trị |
| 5. Nguyên nhân | SQL dùng `:phone IS NULL OR phone = :phone`; PostgreSQL không suy ra kiểu của tham số trong nhánh `IS NULL` |
| 6. Cách sửa | Chỉ thêm điều kiện `WHERE phone = :phone` khi có phone; nếu không thì truy vấn toàn bộ danh sách |
| 7. Commit và xác nhận | Bản sửa `list_customers` được đưa vào [commit 0b868899](https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2/commit/0b8688998d3c362a9853cb0da596f410069fafb), cùng phần CRUD khách hàng/thiết bị. Test tích hợp `test_repository_crud_and_referenced_delete_guards` tra cứu số điện thoại sau khi tạo khách thành công trong lần chạy đầy đủ 06/11/2026 (39 passed). |

## BUG-002 · Minor — README thiếu hướng dẫn dựng trên máy mới

| Dòng | Nội dung |
|---|---|
| 1. Mã và mức độ | BUG-002 · Minor — thiếu quy trình từ clone repo đến chạy hệ thống trên máy tính mới |
| 2. Phát hiện bởi | Đối chiếu README với tiêu chí C1 Buổi 16 và yêu cầu giảng viên |
| 3. Triệu chứng | README cũ bắt đầu từ dự án đã có sẵn trên máy, không hướng dẫn clone; trình tự khởi động còn có bước kiểm tra cấu hình ngoài hai lệnh setup chính |
| 4. Cách tái lập | Đọc README theo góc nhìn người nhận repo lần đầu và thử theo các bước đã ghi |
| 5. Nguyên nhân | Tài liệu tập trung vào cách chạy của người phát triển hiện tại, chưa mô tả clean checkout |
| 6. Cách sửa | Bổ sung clone, tạo `.env` mà không ghi đè, build/khởi động, kiểm tra trạng thái dịch vụ, hướng dẫn xử lý sự cố và cảnh báo giữ dữ liệu |
| 7. Commit và xác nhận | [Commit 0188d40](https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2/commit/0188d4067f0b7647b0490403560108418093d3cb). README và Deployment Guide hiện có hai lệnh setup chính sau khi clone; chạy thực tế Docker trên máy sạch vẫn chưa xác minh vì Docker CLI không có trong môi trường kiểm tra. |

## BUG-003 · Minor — Hợp đồng API chưa phân biệt thiết kế với triển khai

| Dòng | Nội dung |
|---|---|
| 1. Mã và mức độ | BUG-003 · Minor — tài liệu hợp đồng có thể khiến người đọc hiểu nhầm route phiếu là API đang hoạt động |
| 2. Phát hiện bởi | Đối chiếu `docs/api-contract.md` với OpenAPI được tạo từ ứng dụng FastAPI |
| 3. Triệu chứng | Tài liệu cũ liệt kê các endpoint phiếu trong khi OpenAPI hiện tại chỉ đăng ký 13 operation cho health, login, khách hàng, thiết bị và phân loại |
| 4. Cách tái lập | So sánh routes trong `app.openapi()["paths"]` với API contract |
| 5. Nguyên nhân | Hợp đồng thiết kế cũ không được đánh dấu rõ là chưa hiện thực |
| 6. Cách sửa | Gắn cảnh báo phân biệt thiết kế với runtime; thêm API reference cho đúng 13 operation hiện có và loại các route phiếu chưa triển khai |
| 7. Commit và xác nhận | [Commit 69c8bfd](https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2/commit/69c8bfd4a890155b5dd7b646fe16287c4bfdf89a). So sánh tự động cho kết quả 13 operation được tài liệu hóa, không thiếu và không thừa. |

## BUG-004 · Medium — Dependency được pin có advisory bảo mật đã biết

| Dòng | Nội dung |
|---|---|
| 1. Mã và mức độ | BUG-004 · Medium — `pytest` 8.4.2 và `opentelemetry-api` 1.45.1 thuộc các phiên bản bị advisory cảnh báo |
| 2. Phát hiện bởi | `pip-audit` chạy ngày 15/11/2026 trên `requirements.txt` và `requirements.lock` |
| 3. Triệu chứng | Audit phát hiện PYSEC-2026-1845 cho pytest (CVSS 6.8, Medium) và CVE-2026-126360 cho opentelemetry-api (CVSS 5.3, Medium) |
| 4. Cách tái lập | Cài các phiên bản cũ từ lock rồi chạy `python -m pip_audit -r requirements.lock` |
| 5. Nguyên nhân | Các phiên bản cũ vẫn được pin trong manifest/lock dù advisory đã có phiên bản sửa |
| 6. Cách sửa | Nâng pytest lên 9.0.3 và opentelemetry-api lên 1.45.2; chạy lại suite, `pip check` và audit |
| 7. Commit và xác nhận | [Commit dc205a6](https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2/commit/dc205a6b2a6b199e0d77eb7a8fbf4bde385a8a4a). Suite: 39 passed; `pip check`: no broken requirements; `pip-audit -r requirements.txt` và `pip-audit -r requirements.lock`: no known vulnerabilities ngày 15/11/2026. |
