# 🌱 Ví Hạt Mầm 4.0 – Trường THCS Hoàng Diệu, Đà Nẵng

**Bản thử nghiệm nhiều thiết bị** dựa theo 2 ảnh giao diện học sinh và giáo viên. Giao diện hoàn toàn tiếng Việt, responsive, HTML/CSS/JS thuần, Vercel Functions và Upstash Redis REST. Không cần Next.js, PostgreSQL hoặc build frontend.

## Chức năng

**Học sinh:** Đăng ký bằng mã HS, tên, lớp, mật khẩu và PIN 4 số; Gieo mầm (4 danh mục, lời nhắn); Nhận mầm sau khi giáo viên duyệt và xác nhận PIN; QR đổi quà một lần; Ví của tôi; Khảo sát trước/sau tự nguyện.

**Giáo viên:** Tổng quan (số liệu thật); xét duyệt quyền nhận, khóa/mở tài khoản; quản lý danh mục, hạn mức; nhận mã QR và xác nhận đổi; biểu đồ, Jain/Gini, khảo sát và xuất CSV; nhật ký hoạt động.

**LƯU Ý:** Ảnh tham khảo chỉ dùng định hướng UI. Các số liệu trong giao diện được tính từ kho dữ liệu thật. Không cấy số liệu minh họa 128, 97, 92%, 36 hoặc kết quả tâm lý giả định.

## 1. Đưa lên GitHub

Giải nén ZIP; mở https://github.com/new, tạo repository `vi-hat-mam` (có thể chọn Private). Tải **toàn bộ nội dung bên trong thư mục** lên repository sao cho thư mục `api/`, `public/` và `vercel.json` nằm ở gốc. Không đưa `.env`, mật khẩu, token lên GitHub.

## 2. Đưa lên Vercel

1. https://vercel.com/new → Import Git Repository → chọn `vi-hat-mam`.
2. Framework Preset: **Other**. Root Directory: `./`. Không cần Build Command.
3. Vào Marketplace → Upstash Redis (https://vercel.com/marketplace), cài và kết nối dự án. Trong Upstash, tạo Redis Database và chọn vùng gần Việt Nam nếu khả dụng.
4. Vercel → Project → Settings → Environment Variables, đặt **5 biến** (tên phân biệt hoa/thường):

| Key | Value |
|---|---|
| `KV_REST_API_URL` | Giá trị REST URL từ Upstash (hoặc dùng `UPSTASH_REDIS_REST_URL`) |
| `KV_REST_API_TOKEN` | REST Token từ Upstash (hoặc dùng `UPSTASH_REDIS_REST_TOKEN`) |
| `SESSION_SECRET` | Chuỗi **ngẫu nhiên tối thiểu 32 ký tự**, giữ bí mật |
| `ADMIN_USERNAME` | `quantri` (hoặc tên giáo viên tự chọn) |
| `ADMIN_PASSWORD` | Mật khẩu mạnh **tối thiểu 12 ký tự** |

Có thể chọn Production và Preview nếu cần. Bật Sensitive cho các bí mật nếu Vercel hỗ trợ. **Không đặt thêm dấu nháy** trong giá trị. Tạo bí mật trên máy: `node -e "console.log(require('crypto').randomBytes(48).toString('base64url'))"`.

5. Deploy/Redeploy sau khi thêm biến. Vào `https://<ten-du-an>.vercel.app/api?action=health` (đây là mẫu địa chỉ) để xem `{"ok":true,"storage":"connected"}` khi Redis hoạt động.
6. Đăng nhập giáo viên với đúng `ADMIN_USERNAME` và `ADMIN_PASSWORD`. Tạo 2 tài khoản học sinh giả lập, giáo viên duyệt 1 tài khoản, thử gieo/nhận/đổi QR. Dùng trình duyệt khác để xác minh dữ liệu đồng bộ.

## 3. Cấu trúc

```
public/index.html     Giao diện chính
public/theme.css      Thiết kế pastel theo hai ảnh tham khảo
public/app.js         Reactivity, màn hình học sinh/giáo viên
api/index.js          Xử lý đăng nhập, phân quyền, dữ liệu, QR, khảo sát
vercel.json           Vercel Functions, routing, security headers
tests/                Kiểm thử logic
.env.example          Tên biến môi trường, không có bí mật thật
```

## 4. Giới hạn cần hiểu trước khi áp dụng trong trường

- **Đây là bản pilot, chưa phải sản phẩm production đã kiểm thử an ninh.** Tối đa vài lớp và dữ liệu giả; không đưa hồ sơ học sinh thật lên khi chưa có phê duyệt, đồng thuận và đánh giá bảo vệ dữ liệu.
- Redis lưu trạng thái dưới một khóa JSON với thao tác CAS nguyên tử. Phù hợp demo; khi mở rộng cần mô hình dữ liệu chuẩn hóa, backup/khôi phục, quản trị khóa và kiểm thử tải.
- Giá trị gói là **tham khảo**, giao diện không thu hay chuyển tiền thật. Trường phải có quy trình tiếp nhận/đối soát hỗ trợ.
- PIN 4 số là lớp xác nhận bổ sung, **không đủ** làm thông tin đăng nhập duy nhất; API có giới hạn tần suất nhưng cần thêm bảo vệ theo tài khoản, kiểm thử bảo mật, giám sát/alert và lưu trữ theo chính sách.
- DAPS là ẩn danh giữa người gieo và người nhận, không ẩn danh tuyệt đối trước giáo viên có quyền quản trị. Giáo viên có khả năng xem dữ liệu nhạy cảm theo vai trò; cần đào tạo và cam kết bảo mật.
- Khảo sát chỉ gồm **3 câu mẫu chưa thẩm định** và không tạo được kết luận về hiệu quả can thiệp hay chỉ số SUS; xuất CSV để kiểm tra độc lập. Không yêu cầu học sinh trả lời như điều kiện nhận hỗ trợ.
- `SESSION_SECRET` phải cố định cho các phiên triển khai cùng kho Redis; thay khóa sẽ khiến phiên và QR đã cấp trước đó không thể giải mã. Phải lập kế hoạch luân chuyển khóa khi nâng cấp.

## 5. Kiểm thử

Có Node.js 20+: chạy `npm test` (kiểm thử đơn vị). Khuyến nghị kiểm thử bổ sung: xác thực sai, quyền truy cập trái vai trò, hai học sinh nhận đồng thời, đổi lại mã QR, mã hết hạn, API mất Redis, trình duyệt di động, giới hạn chi phí và khả năng khôi phục dữ liệu. Kiểm thử tự động chưa thay thế thử nghiệm thực tế trên Vercel.

## Phiên bản 4.0 – những cập nhật

- Nhận diện **Trường THCS Hoàng Diệu, Đà Nẵng** trên trang chủ, trang đăng nhập, màn hình học sinh/giáo viên và chân trang.
- Khảo sát bổ sung xác nhận tham gia **tự nguyện**, được kiểm tra ở phía máy chủ; không ảnh hưởng quyền nhận hỗ trợ.
- Giới hạn thao tác xác nhận PIN theo tài khoản; hỗ trợ kiểm tra kết nối Redis qua API sức khỏe.
- Thay namespace Redis thành `seedwallet:hoangdieu:v4:state` để tránh va chạm dữ liệu phiên bản cũ. **Không tự động chuyển dữ liệu cũ.**
- Bổ sung trạng thái focus dễ nhìn, bố cục mobile và tùy chọn giảm chuyển động.

### Kiểm tra nhanh trước khi trình diễn

1. Mở `/api?action=health`, xác nhận `ok: true` (đường dẫn API được định tuyến theo cấu hình Vercel).
2. Đăng nhập giáo viên bằng biến `ADMIN_USERNAME` và `ADMIN_PASSWORD`.
3. Đăng ký hai tài khoản học sinh **giả lập**, xét duyệt một tài khoản, gieo mầm ở tài khoản còn lại.
4. Nhận gói bằng PIN, mở QR trong Ví cá nhân và dùng tài khoản giáo viên xác nhận đổi.
5. Quét lại mã QR cũ để kiểm tra bị từ chối; thử đăng nhập trên trình duyệt khác để kiểm tra đồng bộ dữ liệu.
6. Thử xuất CSV, kiểm tra chỉ chứa dữ liệu phù hợp; thử khảo sát trước–sau khi đã tự nguyện đồng ý.

**Quan trọng:** Không nên sử dụng dữ liệu học sinh thật trước khi nhà trường phê duyệt, thực hiện đánh giá bảo vệ dữ liệu và kiểm thử bảo mật/tải. Ứng dụng là **pilot có backend**, chưa phải hệ thống production được kiểm toán. Việc tạo gói hỗ trợ chỉ ghi cam kết, **không xử lý thanh toán**.
