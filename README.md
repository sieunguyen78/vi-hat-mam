# 🌱 Ví Hạt Mầm – SEED WALLET 2.5

Website chia sẻ học đường ẩn danh hai chiều (DAPS), **giao diện hoàn toàn bằng tiếng Việt**. Đây là bộ nguồn **Vercel không dùng Next.js, không cần SQL**. Cấu trúc chỉ có `public/` (giao diện), `api/index.js` (máy chủ), các tệp cấu hình và test.

## Chức năng chính

- Học sinh tự đăng ký với mã học sinh, tên, lớp, mật khẩu >= 10 ký tự và PIN 4 chữ số dùng riêng khi nhận.
- Quản trị viên đăng nhập bằng thông tin thiết lập trong Environment Variables, duyệt quyền nhận hỗ trợ, khóa/mở tài khoản, giao quyền điểm đổi quà.
- Gieo mầm theo danh mục, nhập lời nhắn; hệ thống ghi **cam kết** hỗ trợ, chưa xử lý tiền thật.
- Nhận mầm sau khi được xác nhận quyền; thuật toán ưu tiên lượt nhận thấp nhất trong danh mục (fair-share theo số lượt, không chấm điểm nghèo khó).
- QR do máy chủ phát hành, token ngẫu nhiên, lưu HMAC SHA-256 để xác minh và bản mã AES-256-GCM để người nhận xem lại. Có hạn 7 ngày và chỉ được đổi một lần bằng cập nhật có điều kiện.
- Quyền điểm đổi quà có thể nhập mã từ ứng dụng quét QR; không xem danh tính người nhận trong màn hình đổi quà.
- Báo cáo Jain/Gini/bao phủ, biểu đồ Chart.js, xuất CSV; khảo sát trước–sau 3 câu **minh họa**, lưu cặp theo mã giả danh trong CSV.
- Mã nguồn không có dữ liệu người dùng mẫu thực tế.

## Triển khai trên Vercel (không cần máy chủ riêng)

1. Giải nén ZIP. Đưa **các tệp bên trong thư mục này** lên một repository GitHub mới, để thư mục `public` và `api` nằm ngay thư mục gốc repository.
2. Mở [vercel.com/new](https://vercel.com/new), đăng nhập GitHub, chọn repository rồi `Import`. `Framework Preset`: **Other**; `Root Directory`: `./`; **không đặt Build Command**.
3. Từ **Vercel Marketplace**, thêm database **Upstash Redis** cho project (có thể chịu phí tùy gói/sử dụng). Sau khi kết nối, kiểm tra `KV_REST_API_URL`/`KV_REST_API_TOKEN` hoặc `UPSTASH_REDIS_REST_URL`/`UPSTASH_REDIS_REST_TOKEN`. Nếu được cấp tên biến khác, đặt thêm các biến đúng tên trong project.
4. Trong `Project Settings > Environment Variables`, thêm:

   - `SESSION_SECRET`: chuỗi ngẫu nhiên tối thiểu 32 ký tự (có thể dùng `openssl rand -hex 32`). Giữ bí mật, không đưa lên GitHub.
   - `ADMIN_USERNAME`: ví dụ `quantri`.
   - `ADMIN_PASSWORD`: mật khẩu quản trị mạnh, tối thiểu 12 ký tự. Không dùng lại mật khẩu ở dịch vụ khác.
   - Redis REST URL và token như bước 3, nếu tích hợp chưa tự thêm.

5. **Redeploy** sau khi khai báo biến môi trường. Vào `https://<ten-du-an>.vercel.app/api/index?action=health`: nếu kết nối tốt, nhận `{"ok":true,"storage":"connected"}`.
6. Vào trang chính để đăng nhập quản trị, tạo tài khoản học sinh mẫu, xét duyệt, gieo, nhận và đổi quà. Lưu ý tài khoản quản trị chỉ được khởi tạo từ **biến môi trường trên máy chủ**, không có mật khẩu mặc định trong giao diện.

## Luồng thử nghiệm hai thiết bị

1. Trên thiết bị A, học sinh 1 đăng ký -> giáo viên duyệt học sinh nhận trên trang Quản trị.
2. Trên thiết bị B, học sinh 2 đăng ký -> gieo gói bữa sáng.
3. Trên A, học sinh 1 đăng nhập -> Nhận mầm -> nhập PIN -> hiển thị mã QR.
4. Tạo tài khoản điểm đổi quà bằng cách quản trị viên chuyển vai trò học sinh thử nghiệm sang `Điểm đổi quà`. Trên thiết bị C, đăng nhập vai trò này -> dán nội dung QR -> xác nhận.
5. Thử xác nhận cùng một mã lần nữa: máy chủ phải từ chối.
6. Quản trị viên mở Nghiên cứu -> tải CSV.

## Chạy cục bộ

Cần Node.js >=20 và Vercel CLI. Trong thư mục nguồn: `npm install -g vercel`, sau đó `vercel link`, `vercel env pull .env.local`, `vercel dev`. Mở URL local do CLI báo. Nếu chỉ mở tệp HTML trực tiếp, API sẽ không hoạt động.

## Mô hình dữ liệu và giới hạn quan trọng

Bản này dùng **một bản ghi JSON** trên Upstash Redis, cập nhật qua CAS (Lua compare-and-set) để tránh ghi đè khi nhiều người nhận/đổi cùng lúc. Phù hợp trình diễn hoặc thử nghiệm có kiểm soát, **không phù hợp triển khai quy mô lớn**, và không thay thế cơ sở dữ liệu giao dịch chuẩn với sao lưu/khôi phục. Cần kiểm tra giới hạn bộ nhớ, quota và dung lượng request. Một số tiện ích phụ thuộc CDN (Lucide, QRCode, Chart.js, Google Fonts); website cần Internet để hiển thị trọn vẹn.

PIN 4 số không được dùng để đăng nhập, chỉ xác nhận việc nhận; mật khẩu đăng nhập >=10 ký tự. Xác thực dùng cookie HttpOnly, Secure, SameSite=Lax có chữ ký HMAC và kiểm tra quyền tại server. Có giới hạn tần suất cơ bản qua Redis nhưng **chưa có WAF/chống bot toàn diện, email xác minh, SSO học đường, phân tách nhiệm vụ phê duyệt, nhật ký bất biến, kiểm toán an ninh, khôi phục tài khoản và cơ chế hết hạn giữ dữ liệu**.

**Dữ liệu học sinh là dữ liệu cá nhân, có thể là dữ liệu nhạy cảm khi nói đến hoàn cảnh cần hỗ trợ.** Không nhập tên/sự kiện thực vào môi trường demo công khai. Trước khi áp dụng thực tế, nhà trường phải rà soát cơ sở pháp lý, thông báo và xin chấp thuận cần thiết, phân quyền cán bộ được phép truy cập, đánh giá tác động, quy định lưu/xóa dữ liệu, bảo mật thiết bị và sao lưu, cùng quy trình hỗ trợ ngoài đời. Giáo viên quản trị có quyền thấy danh tính tài khoản; ẩn danh là **giữa học sinh với học sinh**, không phải ẩn danh tuyệt đối với cơ quan quản lý hệ thống.

**Khảo sát**: Ba câu 1–5 chỉ là mẫu thiết kế; chưa được thẩm định tâm lý, chưa thể dùng để khẳng định giảm mặc cảm/tăng lòng tự trọng. Phải có đề cương thực nghiệm và sự đồng thuận hợp lệ với trẻ vị thành niên trước khi thu thập dữ liệu thật. CSV dùng mã giả danh nên vẫn có nguy cơ liên kết lại; quản lý tệp xuất có kiểm soát.

**Tiền và hàng**: Không có ví tiền thật hoặc cổng thanh toán; nghiệp vụ là cam kết hỗ trợ và xác nhận phát quà. Nhà trường cần quản lý hiện vật, tài chính và kiểm kê riêng.

## Kiểm thử

`npm test` chạy các bài kiểm thử công thức công bằng và mật khẩu. Kết quả chưa thay thế kiểm thử API tích hợp có Redis, kiểm thử đồng thời nhiều yêu cầu, bảo mật ứng dụng và kiểm thử thực địa.
