# Smart CRM — Mekong Mobile

Prototype Track SE cho luồng L2: tiếp nhận, quản lý khách hàng/thiết bị và đề
xuất phân loại yêu cầu bảo hành.

## Chức năng hiện có

- Đăng nhập bằng một tài khoản demo cấu hình qua môi trường.
- CRUD khách hàng và thiết bị qua API và giao diện web.
- Không xóa khách hàng còn thiết bị/phiếu tham chiếu hoặc thiết bị đã có phiếu;
  API trả `409 Conflict`. Không có thao tác xóa phiếu.
- Đề xuất nhóm sự cố và mức ưu tiên từ mô tả; kết quả chỉ là đề xuất, không tự
  tạo phiếu bảo hành.
- API health check tại `/health`; log request dạng JSON trên stdout, có
  `request_id` và không ghi request body, mật khẩu hay token.

Prototype chưa triển khai tạo/lập danh sách phiếu, trạng thái phiếu, tài khoản
nhân viên thật, đăng ký người dùng hoặc phân quyền theo vai trò.

## Chạy toàn hệ thống bằng Docker Compose

### Chạy trên máy tính mới (Windows, khuyến nghị Docker)

Máy mới chỉ cần Git và Docker Desktop đang chạy; không cần cài Python hoặc
PostgreSQL trên máy host để mở ứng dụng.

1. Mở PowerShell và clone repo public:

```powershell
git clone https://github.com/torikachi170/smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2.git
Set-Location .\smartcrm-2374802010363-Receive-and-process-warranty-service-requests-v0.2
```

2. Tạo cấu hình local từ mẫu. Không dùng lệnh này để ghi đè `.env` đã có:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

`.env.example` có giá trị mẫu chạy được cho demo local. Nếu dùng môi trường
khác, thay mật khẩu/khóa mẫu bằng giá trị riêng; không commit hoặc gửi `.env`.
`POSTGRES_PASSWORD` chỉ nên gồm chữ, số hoặc dấu gạch nối để phù hợp URL kết
nối hiện tại; `JWT_SECRET_KEY` phải dài ít nhất 32 ký tự.

Có thể kiểm tra cấu hình trước khi khởi động (tùy chọn; không cần cho luồng
setup nhanh):

```powershell
docker compose config --quiet
```

3. Build và khởi động (lệnh setup chính thứ hai):

```powershell
docker compose up -d --build
```

Đợi cả `db` và `api` hiện `healthy`:

```powershell
docker compose ps
```

Kết quả mong đợi:

- `db` và `api` có trạng thái `healthy`.
- `Invoke-RestMethod http://127.0.0.1:8000/health` trả về `status: ok`.
- Mở `http://127.0.0.1:8000/login` để dùng giao diện; `/docs` mở Swagger UI.
- API health endpoint chỉ xác nhận ứng dụng phản hồi; kiểm tra riêng trạng thái
  database bằng `docker compose ps`.

Nếu cổng 8000 hoặc 5432 đang được dùng, đổi `APP_PORT` hoặc `DB_PORT` trong
`.env`, rồi chạy lại `docker compose up -d`. Nếu Compose báo thiếu biến, kiểm
tra `.env` có được tạo từ `.env.example` trong đúng thư mục dự án hay chưa.
Xem log khởi động bằng:

```powershell
docker compose logs --tail 100 api db
```

Để dừng ứng dụng mà vẫn giữ dữ liệu demo:

```powershell
docker compose down
```

`docker compose down -v` xóa volume PostgreSQL và toàn bộ dữ liệu trong đó;
chỉ dùng khi chủ ý dựng lại dữ liệu demo từ đầu.

Trên macOS/Linux, thay lệnh tạo `.env` bằng `cp .env.example .env`; các lệnh
Docker Compose còn lại tương tự.

### Cấu hình và khởi động (tham khảo)

Yêu cầu: Docker Desktop (hoặc Docker Engine có Compose) đang chạy.

1. Tạo `.env` từ mẫu **chỉ khi chưa có**:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

2. Mở `.env` và bảo đảm có đủ `POSTGRES_DB`, `POSTGRES_USER`,
   `POSTGRES_PASSWORD`, `DB_PORT` và `APP_PORT`. Nếu đã có `.env` từ phiên bản
   cũ, hãy bổ sung các biến còn thiếu, không sao chép đè lên file hiện tại.
   Đặt `POSTGRES_PASSWORD`, `DEMO_PASSWORD` và `JWT_SECRET_KEY` thành giá trị
   cục bộ của bạn. Mật khẩu PostgreSQL chỉ nên gồm chữ/số/dấu gạch nối để phù
   hợp URL kết nối. Khóa JWT cần ít nhất 32 ký tự. Không commit `.env` hay
   chia sẻ bí mật.

   `APP_PORT` là cổng mở trên máy của bạn. Nếu đặt `APP_PORT=3000`, mở
   `http://127.0.0.1:3000/login`; nếu đặt `APP_PORT=8000`, mở
   `http://127.0.0.1:8000/login`.

3. Build và khởi động:

```powershell
docker compose up -d --build
```

Đợi dịch vụ `db` và `api` chuyển sang trạng thái `healthy`:

```powershell
docker compose ps
```

Mở `http://127.0.0.1:<APP_PORT>/` hoặc `http://127.0.0.1:<APP_PORT>/login`;
đường dẫn gốc tự chuyển đến trang đăng nhập. Swagger UI ở cùng cổng tại `/docs`.
Tên đăng nhập mặc định trong tệp mẫu là `demo`; mật khẩu là giá trị
`DEMO_PASSWORD` do bạn đặt trong `.env`. Nếu giữ nguyên mẫu thì mật khẩu mẫu là
`change-this-demo-password` (chỉ dành cho máy local).

Schema và danh mục mẫu được nạp tự động khi PostgreSQL khởi tạo volume lần đầu.
`db/schema.sql` và `db/seed.sql` có thể chạy lặp lại trên cùng schema. Sửa dữ
liệu khởi tạo của volume hiện có không tự chạy lại các script. Chỉ khi cần xóa
toàn bộ dữ liệu demo để dựng mới, có thể chạy `docker compose down -v`; thao tác
này xóa dữ liệu trong volume.

## Chạy local không dùng Docker

Yêu cầu Python 3.14 và PostgreSQL 16 trở lên; PostgreSQL phải có sẵn và
`DATABASE_URL` trong `.env` phải trỏ đúng database.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.lock
```

Chuẩn bị database local và nạp schema/danh mục:

```powershell
createdb smart_crm_l2
psql -d smart_crm_l2 -v ON_ERROR_STOP=1 -f db/schema.sql
psql -d smart_crm_l2 -v ON_ERROR_STOP=1 -f db/seed.sql
```

Chạy ứng dụng:

```powershell
python -m uvicorn src.presentation.api:app --reload --host 127.0.0.1 --port 8000
```

Mở `http://127.0.0.1:8000/` hoặc `http://127.0.0.1:8000/login`; đường dẫn
gốc tự chuyển đến trang đăng nhập. Tài khoản demo đọc từ `DEMO_USERNAME` và
`DEMO_PASSWORD` trong `.env`.

## API khách hàng và thiết bị

Tất cả API nghiệp vụ cần `Authorization: Bearer <JWT>` lấy từ
`POST /api/auth/login`.

| Phương thức | Đường dẫn | Chức năng |
|---|---|---|
| GET / POST | `/api/customers` | Liệt kê/tra cứu hoặc tạo khách hàng |
| GET / PATCH / DELETE | `/api/customers/{id}` | Xem, sửa, hoặc xóa khách chưa được tham chiếu |
| GET / POST | `/api/customers/{id}/devices` | Liệt kê hoặc thêm thiết bị |
| GET / PATCH / DELETE | `/api/devices/{id}` | Xem, sửa, hoặc xóa thiết bị chưa có phiếu |
| POST | `/api/classify` | Đề xuất nhóm sự cố và mức ưu tiên |

DELETE trả `409` nếu bản ghi còn được dữ liệu khác tham chiếu; không có xóa dây
chuyền. Số điện thoại phải gồm 10 chữ số bắt đầu bằng 0; `phone` và serial/IMEI
phải duy nhất. Vì prototype chưa có vai trò quản lý, API che số điện thoại theo
NFR2; khi sửa hồ sơ, để trống trường điện thoại sẽ giữ nguyên giá trị hiện có.

## Kiểm thử

Chạy toàn bộ test bằng một lệnh:

```powershell
python -m pytest test -v
```

Lệnh trên chạy cả test unit/API và test PostgreSQL. Test PostgreSQL dùng
database riêng được chỉ định bởi `TEST_DATABASE_URL` trong `.env`; nếu chưa
cấu hình biến này, pytest sẽ đánh dấu các test PostgreSQL là skipped. Không
trỏ `TEST_DATABASE_URL` tới database có dữ liệu thật.

Có thể chạy riêng test PostgreSQL:

```powershell
python -m pytest test/test_postgres_constraints.py -v
```

Test PostgreSQL tạo schema ngẫu nhiên, xóa riêng schema đó khi hoàn tất và
không được trỏ tới database có dữ liệu thật.

## Tài liệu dự án

- `docs/srs.md` — yêu cầu và quy tắc nghiệp vụ.
- `docs/architecture.md` — kiến trúc đang hiện thực.
- `docs/api-contract.md` — hợp đồng API theo thiết kế; đối chiếu trạng thái triển khai trước khi dùng làm tài liệu vận hành.
- `docs/api-reference.md` — API reference của các endpoint hiện được triển khai.
- `docs/thay-doi-thiet-ke.md` — nhật ký điều chỉnh thiết kế BT2.
- `docs/test-plan.md` và `docs/test-report.md` — kế hoạch và báo cáo kiểm thử BT3.
- `docs/deployment-guide.md` — hướng dẫn dựng trên máy mới.
- `docs/checklist-bao-mat.md` — tự đánh giá bảo mật và minh chứng.
- `docs/ai-disclosure.md` — khai báo công cụ hỗ trợ.
- `docs/bug-log.md` — lỗi đã tái lập và xác nhận sửa.
- `BT2_2374802010363_NguyenHoangMinhNhat.docx` — bản Word có thể chỉnh sửa;
  người nộp tự xuất PDF sau khi bổ sung ảnh minh chứng thực tế.

Đối với BT3, cần hoàn thiện tài liệu và ảnh minh chứng, kiểm tra repo public,
đẩy tag `v1.0`, rồi nộp một PDF duy nhất qua VLU E-learning theo yêu cầu
workshop Buổi 16.

## Đăng ký tài khoản

Prototype cố ý chưa có chức năng đăng ký. Tài khoản hiện là một tài khoản demo
cấu hình tại máy chạy qua `DEMO_USERNAME` và `DEMO_PASSWORD`, không có bảng nhân
viên/người dùng, quy trình xác minh, đổi mật khẩu hoặc phân quyền. Tự mở đăng ký
sẽ tạo thêm rủi ro tài khoản giả mạo và cần thiết kế bảo mật riêng; chức năng
này nằm ngoài phạm vi prototype hiện tại.
