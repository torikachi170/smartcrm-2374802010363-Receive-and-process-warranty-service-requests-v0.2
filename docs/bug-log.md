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
| 7. Commit và xác nhận | [Commit ad1e6fe](https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2/commit/ad1e6fe64e98b2fd159fa91477e991648009d5fe). README và Deployment Guide hiện có hai lệnh setup chính sau khi clone; chạy thực tế Docker trên máy sạch vẫn chưa xác minh vì Docker CLI không có trong môi trường kiểm tra. |

## BUG-003 · Minor — Hợp đồng API chưa phân biệt thiết kế với triển khai

| Dòng | Nội dung |
|---|---|
| 1. Mã và mức độ | BUG-003 · Minor — tài liệu hợp đồng có thể khiến người đọc hiểu nhầm route phiếu là API đang hoạt động |
| 2. Phát hiện bởi | Đối chiếu `docs/api-contract.md` với OpenAPI được tạo từ ứng dụng FastAPI |
| 3. Triệu chứng | Tài liệu cũ liệt kê các endpoint phiếu trong khi OpenAPI hiện tại chỉ đăng ký 13 operation cho health, login, khách hàng, thiết bị và phân loại |
| 4. Cách tái lập | So sánh routes trong `app.openapi()["paths"]` với API contract |
| 5. Nguyên nhân | Hợp đồng thiết kế cũ không được đánh dấu rõ là chưa hiện thực |
| 6. Cách sửa | Gắn cảnh báo phân biệt thiết kế với runtime; thêm API reference cho đúng 13 operation hiện có và loại các route phiếu chưa triển khai |
| 7. Commit và xác nhận | [Commit c49b3ea](https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2/commit/c49b3ea55ffd2d0a70f35a6f39952444018c78af). So sánh tự động cho kết quả 13 operation được tài liệu hóa, không thiếu và không thừa. |
