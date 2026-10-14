# Website quản lý và tìm kiếm phòng trọ dành cho sinh viên

Ứng dụng hỗ trợ sinh viên tìm phòng, tìm gần trường, so sánh/lưu phòng, tìm người ở ghép, trò chuyện, đặt lịch xem, theo dõi hợp đồng và hóa đơn. Chủ trọ quản lý khu trọ/phòng; ADMIN quản lý tài khoản, xác minh và báo cáo.

Dữ liệu demo lấy **Đại học Phenikaa** làm trường trung tâm mặc định. Trang chủ, tìm phòng và bản đồ đọc lựa chọn này từ API; người dùng vẫn có thể đổi sang trường khác trong danh mục.

## Nhóm thực hiện

- Lê Anh Minh — 23010228
- Hà Đức Thắng — 23010284

## Công nghệ

- Frontend: React, Vite, JavaScript, React Router, Axios, Bootstrap 5, Leaflet/OpenStreetMap.
- Backend: Node.js, Express.js, REST API, Socket.IO, JWT, bcrypt, Zod.
- Dữ liệu: MySQL và Prisma ORM.
- Kiến trúc: modular monolith trong npm workspaces.

## Cấu trúc

~~~
.
├── client/                 # React + Vite
├── server/                 # Express, Socket.IO, Prisma
│   ├── prisma/             # schema, migration, seed
│   └── src/modules/        # module nghiệp vụ
├── docs/                   # tài liệu kiến trúc/API/ERD/demo
├── docker-compose.yml      # MySQL phát triển
└── .env.example
~~~

## Cài đặt

Yêu cầu Node.js 20+ và Docker Desktop (để chạy MySQL).

1. Sao chép file môi trường cho cả backend và frontend, sau đó thay các giá trị mật khẩu/JWT_SECRET dành riêng cho máy phát triển.

~~~powershell
Copy-Item .env.example .env
Copy-Item client\.env.example client\.env
~~~
2. Chạy MySQL:

~~~bash
docker compose up -d
~~~

3. Cài dependency cho cả hai workspace:

~~~bash
npm install
~~~

4. Tạo Prisma client, áp migration đã có và seed:

~~~bash
npm run prisma:generate
npm run prisma:deploy
npm run prisma:seed
~~~

5. Chạy frontend và backend:

~~~bash
npm run dev
~~~

Frontend mặc định chạy tại http://localhost:5173 và backend tại http://localhost:4000.

Nếu cổng MySQL `3306` đang được ứng dụng khác sử dụng, hãy đổi đồng thời hai dòng trong `.env`, ví dụ `MYSQL_PORT=13306` và `DATABASE_URL="mysql://student_rental:...@localhost:13306/student_rental"`, rồi chạy lại Docker.

### Phục vụ bản build React từ Express

Chế độ này phù hợp khi cần chạy một tiến trình Express phục vụ cả API và giao diện. Vite dev server vẫn là lựa chọn mặc định trong quá trình phát triển.

1. Trước khi build, đặt `VITE_API_URL=/api` và để trống `VITE_SOCKET_URL` trong `client/.env` để client dùng cùng origin với Express.
2. Chạy `npm run build` để tạo `client/dist`.
3. Trong `.env`, thêm `SERVE_CLIENT=true` và đặt `CLIENT_URL=http://localhost:4000`.
4. Chạy server: `npm run start --workspace=server`, rồi mở http://localhost:4000.

Khi cả `SERVE_CLIENT=true` và thư mục `client/dist` tồn tại, Express phục vụ asset build và trả `index.html` cho deep link của SPA. Các đường dẫn `/api/*` và `/uploads/*` vẫn được xử lý trước SPA fallback.

## Kiểm tra

~~~bash
npm run test
npm run build
~~~

Để chạy thêm các integration test với MySQL đã migrate/seed, đặt `DATABASE_URL` đúng database local và bật `RUN_DB_TESTS=1`. Ví dụ PowerShell khi dùng cổng `13306`:

~~~powershell
$env:RUN_DB_TESTS='1'
$env:DATABASE_URL='mysql://student_rental:change-this-local-db-password@127.0.0.1:13306/student_rental'
npm run test --workspace=server -- --runInBand
~~~

Các integration test tạo rồi tự dọn dữ liệu thử nghiệm, gồm luồng phòng, upload, hợp đồng–hóa đơn và Socket.IO.

## Đăng nhập và đăng ký

Người dùng có thể đăng nhập bằng một trong hai định danh:

- **Tên đăng nhập** không cần email, ví dụ `student`.
- **Email**, ví dụ `student@gmail.com`.

Khi đăng ký, `username` và `email` đều là trường tùy chọn riêng lẻ nhưng phải có **ít nhất một** trường. Nếu cung cấp cả hai, cùng một tài khoản có thể đăng nhập bằng một trong hai. Tên đăng nhập có 3–30 ký tự, bắt đầu bằng chữ hoặc số, chỉ gồm chữ cái Latin, chữ số, dấu chấm (`.`), gạch dưới (`_`) hoặc gạch ngang (`-`), và không chứa `@`. Hệ thống chuẩn hóa tên đăng nhập về chữ thường và so khớp không phân biệt hoa/thường: `Student` và `student` là cùng một tên đăng nhập.

## Tài khoản demo

Các tài khoản chỉ dành cho môi trường development; password được bcrypt hash trong database.

| Vai trò | Tên đăng nhập | Email (cách đăng nhập khác) | Password |
| --- | --- | --- | --- |
| ADMIN | admin | admin@gmail.com | 123 |
| STUDENT | student | student@gmail.com | 123 |
| LANDLORD | landlord | chutro@gmail.com | 123 |

Ví dụ, tài khoản sinh viên có thể đăng nhập bằng `student` hoặc `student@gmail.com`, cùng mật khẩu `123`. Mật khẩu `123` chỉ áp dụng cho seed development; mật khẩu đăng ký mới vẫn phải có ít nhất 8 ký tự.

## REST API

API dùng prefix /api, response có dạng:

~~~json
{ "success": true, "message": "...", "data": {} }
~~~

Xem danh mục endpoint và contract đang triển khai trong [docs/api.md](docs/api.md).

## Ảnh chụp màn hình

Thêm ảnh chụp giao diện sau khi chạy ứng dụng tại đây.

## Giới hạn hiện tại

- Bản đồ dùng OpenStreetMap/Leaflet. Chủ trọ có thể nhập địa chỉ và chủ động bấm tìm vị trí; server dùng Nominatim/OpenStreetMap, vị trí khu trọ cùng chủ đã lưu và Photon làm nguồn đối chiếu cho lưu lượng nhỏ, có cache và giới hạn tốc độ. Khi triển khai quy mô lớn cần tự host hoặc thay bằng provider có hợp đồng phù hợp.
- Liên kết Zalo/Telegram chỉ được lưu do người dùng cung cấp; ứng dụng không gọi API không chính thức.
- Room vẫn có thể nhận URL ảnh trong body JSON; đồng thời LANDLORD/ADMIN sở hữu room có thể tải ảnh JPEG/PNG/WebP cục bộ qua API. Cloud storage chưa được tích hợp.

## Hướng phát triển

- Thay map/geocoding provider qua service adapter.
- Tích hợp cloud storage và thanh toán chính thức khi có yêu cầu.
- Bổ sung end-to-end test, audit log và triển khai production.
