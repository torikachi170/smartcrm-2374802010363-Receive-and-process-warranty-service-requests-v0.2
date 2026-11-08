# Test Plan — Smart CRM L2 (BT3)

## 1. Phạm vi kiểm thử

Kiểm thử những chức năng đã có trong prototype: đăng nhập demo, bảo vệ API
bằng token, đề xuất nhóm sự cố, CRUD khách hàng/thiết bị, kiểm tra dữ liệu và
các ràng buộc PostgreSQL. Ưu tiên các quy tắc số điện thoại/serial duy nhất,
độ dài mô tả 10–2000 ký tự, từ chối xóa bản ghi đang được tham chiếu và khả
năng chạy lại schema/seed.

**Chủ ý không kiểm thử:** tạo phiếu và vòng đời phiếu, phân quyền theo vai trò
nhân viên thật, hiệu năng/tải và nghiệm thu người dùng. Các chức năng này chưa
được hiện thực hoặc nằm ngoài phạm vi prototype hiện tại.

## 2. Loại kiểm thử, số lượng và cách chọn dữ liệu

Kế hoạch gồm 12 test case: 2 unit test service, 9 API test và 1 integration
test PostgreSQL. Các test case gồm luồng đúng, biên độ dài mô tả, dữ liệu sai,
bảo mật/toàn vẹn dữ liệu và tính tái lập.

Phân vùng tương đương dùng cho thông tin đăng nhập, mô tả hợp lệ/không hợp lệ,
số điện thoại trùng/sai định dạng và bản ghi có/không có tham chiếu. Giá trị
biên được chọn tại 9, 10, 2000 và 2001 ký tự cho mô tả. Test không dựa vào thứ
tự chạy; dữ liệu API dùng store giả được tạo mới theo fixture, còn test
PostgreSQL tạo schema cô lập.

## 3. Tiêu chí thành công

- Có 12 test case được chạy thật, có kết quả thực tế và ảnh chụp tương ứng.
- Bao phủ đủ năm loại: luồng đúng, giá trị biên, dữ liệu sai,
  bảo mật/chất lượng dữ liệu và tính tái lập.
- Mỗi kết quả PASS khớp đầy đủ mã trạng thái, nội dung phản hồi và tác dụng
  phụ mong đợi; ghi FAIL nếu khác bất kỳ điểm nào.
- Sau khi sửa lỗi, toàn bộ test unit/API phải PASS bằng một lệnh. Test
  PostgreSQL phải PASS khi có `TEST_DATABASE_URL` trỏ tới database thử riêng.
- Không còn lỗi Critical đang mở; mọi lỗi còn lại có lý do và hướng xử lý.

## 4. Môi trường và dữ liệu thử

- Python 3.14; cài dependency chính xác bằng
  `python -m pip install -r requirements.lock`.
- Chạy toàn bộ suite bằng một lệnh:
  `python -m pytest test -v`.
- Test PostgreSQL dùng `TEST_DATABASE_URL` trỏ tới database thử riêng. Nếu
  chưa cấu hình biến này, pytest đánh dấu nhóm test PostgreSQL là skipped.
  Không dùng database có dữ liệu thật.
- API test dùng `TestClient` và store giả được dựng trong fixture; test
  PostgreSQL tạo schema ngẫu nhiên và dọn schema sau khi chạy.
- Lưu ảnh chụp kết quả thật theo mã test trong `docs/screenshots/`. Không đưa
  `.env`, mật khẩu hoặc token vào ảnh.
