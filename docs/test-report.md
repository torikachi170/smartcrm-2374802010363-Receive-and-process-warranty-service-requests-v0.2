# Test Report — Smart CRM L2 (BT3)

Lần chạy đầy đủ gần nhất ngày **13/11/2026** trong clone repo GitHub:
`.\.venv\Scripts\python.exe -m pytest test -q` · **39 passed, 0 failed,
0 skipped** trong 3.40 giây. PostgreSQL integration tests đã chạy trong lượt
này. Có một cảnh báo deprecation từ `starlette.testclient` về `httpx`.

Ảnh `TC-01.png`–`TC-12.png` hiện có được chụp ngày 08/11/2026; ảnh TC-12 của
lượt đó thể hiện SKIPPED. Chúng không phải ảnh của lần chạy 13/11 và không
được dùng để chứng minh lần chạy mới. Cần thay ảnh minh chứng bằng ảnh terminal
thật từ lần chạy gần nhất trước khi nộp; không dựng ảnh thay thế từ văn bản.

Lần chạy đầy đủ ngày 06/11/2026:
`.\.venv\Scripts\python.exe -m pytest test -v` · **39 passed**.

Lần xác minh ngày 08/11/2026 trong clone repo GitHub có **26 passed,
13 skipped, 0 failed** vì thiếu `TEST_DATABASE_URL` và PostgreSQL không sẵn
sàng. TC-01–TC-11 chạy riêng và PASS; TC-12 bị SKIPPED. Ảnh lưu cho các case
ngày 08/11 là ảnh chụp trang hiển thị stdout/stderr đã capture từ tiến trình
pytest, không phải ảnh terminal gốc. Các ảnh này chỉ là bằng chứng lịch sử
của lần chạy đó.

| Mã | Tiền điều kiện | Các bước | Dữ liệu thử | Kết quả mong đợi | Ưu tiên | Kết quả thực tế | PASS/FAIL và ảnh |
|---|---|---|---|---|---|---|---|
| TC-01 (luồng đúng, unit) | Không cần dịch vụ ngoài | Gọi `AuthService.login`, xác minh token | `demo` / `demo-password` | Token có subject `demo`; thời hạn hết hạn lớn hơn thời điểm phát hành | Cao | Token có subject `demo`; `exp > iat` | PASS; ảnh cần chụp: `docs/screenshots/TC-01.png` |
| TC-02 (dữ liệu sai, unit) | Không cần dịch vụ ngoài | Gọi đăng nhập với mật khẩu sai | `demo` / `wrong-password` | Phát sinh `InvalidCredentials` | Trung bình | Phát sinh `InvalidCredentials` như mong đợi | PASS; ảnh cần chụp: `docs/screenshots/TC-02.png` |
| TC-03 (bảo mật, API) | API test dùng fixture và store giả | Gọi `POST /api/classify` không gửi token | Mô tả hợp lệ; không có `Authorization` | HTTP 401; mã lỗi `UNAUTHORIZED` | Cao | HTTP 401; `error.code=UNAUTHORIZED` | PASS; ảnh cần chụp: `docs/screenshots/TC-03.png` |
| TC-04 (luồng đúng, API) | API test dùng danh mục sự cố giả | Đăng nhập rồi gọi `POST /api/classify` | `Máy sạc không vào` | HTTP 200; nhóm `SAC`, mức ưu tiên `TRUNG_BINH` | Cao | HTTP 200; `SAC`; `TRUNG_BINH` | PASS; ảnh cần chụp: `docs/screenshots/TC-04.png` |
| TC-05 (luồng đúng, API) | API test dùng danh mục sự cố giả | Đăng nhập rồi gọi `POST /api/classify` | `Loa rè khi gọi điện thoại` | HTTP 200; nhóm dự phòng `KHAC` | Trung bình | HTTP 200; nhóm `KHAC` | PASS; ảnh cần chụp: `docs/screenshots/TC-05.png` |
| TC-06 (biên dưới, API) | API test dùng danh mục sự cố giả | Gọi phân loại với mô tả 9 ký tự | `"x" * 9` | HTTP 400; mã `VALIDATION_FAILED`; lỗi chỉ ra `issue_desc` | Cao | HTTP 400; `VALIDATION_FAILED`; có trường `issue_desc` | PASS; ảnh cần chụp: `docs/screenshots/TC-06.png` |
| TC-07 (biên dưới, API) | API test dùng danh mục sự cố giả | Gọi phân loại với mô tả đúng ngưỡng tối thiểu | `"x" * 10` | HTTP 200; nhóm dự phòng `KHAC` | Cao | HTTP 200; nhóm `KHAC` | PASS; ảnh cần chụp: `docs/screenshots/TC-07.png` |
| TC-08 (biên trên, API) | API test dùng danh mục sự cố giả | Gọi phân loại với mô tả đúng ngưỡng tối đa | `"x" * 2000` | HTTP 200; nhóm dự phòng `KHAC` | Trung bình | HTTP 200; nhóm `KHAC` | PASS; ảnh cần chụp: `docs/screenshots/TC-08.png` |
| TC-09 (biên trên, API) | API test dùng danh mục sự cố giả | Gọi phân loại với mô tả vượt ngưỡng tối đa | `"x" * 2001` | HTTP 400; mã `VALIDATION_FAILED` | Cao | HTTP 400; `VALIDATION_FAILED` | PASS; ảnh cần chụp: `docs/screenshots/TC-09.png` |
| TC-10 (dữ liệu sai, API) | Store khách hàng mới, chưa có số điện thoại thử | Tạo khách với số điện thoại đã được lưu | SĐT `0900000001` dùng cho hai khách | Lần tạo thứ hai trả HTTP 409, mã `PHONE_EXISTS`; không tạo khách trùng | Cao | HTTP 409; `error.code=PHONE_EXISTS` | PASS; ảnh cần chụp: `docs/screenshots/TC-10.png` |
| TC-11 (toàn vẹn dữ liệu, API) | Có khách và thiết bị thuộc khách đó | Gửi yêu cầu xóa khách; đọc lại store | Khách có một thiết bị | HTTP 409, mã `CUSTOMER_HAS_DEVICES`; khách và thiết bị vẫn tồn tại | Cao | HTTP 409; `CUSTOMER_HAS_DEVICES`; khách và thiết bị vẫn còn trong store | PASS; ảnh cần chụp: `docs/screenshots/TC-11.png` |
| TC-12 (tái lập, integration) | `TEST_DATABASE_URL` trỏ database thử riêng | Chạy schema và seed hai lần; đếm danh mục | Schema/seed trong repo | Không lỗi; có đúng 6 danh mục, không nhân đôi | Cao | 06/11: PASS; 08/11: SKIPPED; 13/11: nằm trong suite 39 passed, 0 skipped | PASS trong suite 06/11 và 13/11; ảnh TC-12 hiện có chỉ thể hiện SKIPPED ngày 08/11, cần thay ảnh |

## Lệnh chạy riêng để chụp minh chứng

Các lệnh dưới đây đã chạy riêng ngày 08/11/2026; ảnh chụp stdout/stderr thật
được lưu theo mã test. TC-12 hiện cho kết quả SKIPPED vì PostgreSQL không sẵn
sàng; không thay trạng thái này thành PASS.

| Ảnh | Lệnh |
|---|---|
| `TC-01.png` | `python -m pytest test/test_auth_service.py::test_login_returns_token_with_expected_subject -v` |
| `TC-02.png` | `python -m pytest test/test_auth_service.py::test_login_rejects_invalid_credentials -v` |
| `TC-03.png` | `python -m pytest test/test_classify_api.py::test_protected_endpoint_requires_access_token -v` |
| `TC-04.png` | `python -m pytest test/test_classify_api.py::test_classify_returns_category_and_default_priority -v` |
| `TC-05.png` | `python -m pytest test/test_classify_api.py::test_classify_falls_back_to_other_category -v` |
| `TC-06.png` | `python -m pytest test/test_classify_api.py::test_classify_rejects_short_description_with_contract_status -v` |
| `TC-07.png` | `python -m pytest test/test_classify_api.py::test_classify_accepts_minimum_description_length -v` |
| `TC-08.png` | `python -m pytest test/test_classify_api.py::test_classify_accepts_maximum_description_length -v` |
| `TC-09.png` | `python -m pytest test/test_classify_api.py::test_classify_rejects_description_over_contract_limit -v` |
| `TC-10.png` | `python -m pytest test/test_customer_api.py::test_customer_duplicate_phone_returns_conflict -v` |
| `TC-11.png` | `python -m pytest test/test_customer_api.py::test_customer_with_device_cannot_be_deleted -v` |
| `TC-12.png` | `python -m pytest test/test_postgres_constraints.py::test_schema_and_seed_can_be_applied_repeatedly -v` |

## Tổng hợp

- Lần chạy đầy đủ gần nhất ngày 13/11: **39 passed, 0 failed, 0 skipped**, gồm test
  PostgreSQL; một cảnh báo deprecation từ `starlette.testclient` về `httpx`.
- Lần chạy đầy đủ ngày 06/11 cũng ghi nhận **39 passed**.
- Lần chạy ngày 08/11: **26 passed, 13 skipped, 0 failed**; TC-12 bị skip
  trong lần đó nhưng PASS trong lần chạy 13/11.
- Bao phủ đủ năm loại: **Có**.
- Ảnh testcase hiện có là ảnh của ngày 08/11; cần chụp lại từ terminal cho
  lần chạy mới, đặc biệt TC-12, và không dùng ảnh SKIPPED cũ làm bằng chứng
  PASS.
- Ảnh kết quả toàn bộ suite ngày 08/11:
  `docs/screenshots/FULL-SUITE-2026-11-08.png`.
- Chưa có ảnh chụp terminal gốc cho suite ngày 13/11. Bổ sung ảnh đó và ảnh
  của các test case theo yêu cầu nộp trước khi chốt PDF.
