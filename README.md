# VÍ HẠT MẦM 5.0 – Trường THCS Hoàng Diệu, Đà Nẵng

Ứng dụng tiếng Việt gồm **hai vai trò Học sinh và Giáo viên**, chạy trên Vercel (HTML/CSS/JS + Serverless Functions), dữ liệu dùng chung qua Upstash Redis REST. Đây là **phiên bản thử nghiệm/pilot**, chưa được chứng nhận để xử lý dữ liệu trẻ em trong triển khai chính thức.

## Các sửa lỗi quan trọng ở 5.0

- **Nhận mầm:** học sinh đăng ký, giáo viên duyệt quyền nhận, học sinh chọn gói có sẵn và nhập PIN 4 số; API xác nhận quyền, danh mục và quy tắc phân phối. Khi không đủ điều kiện, hiển thị thông báo cụ thể.
- **Mã QR:** tạo QR **tại máy chủ**, mã hóa dưới dạng hình SVG và trả về cho học sinh sở hữu mã qua API `qr-image`; không lệ thuộc CDN thư viện QR. QR chứa token ngẫu nhiên, không chứa tên học sinh. Học sinh có thể mở lại QR trong Ví của tôi; giáo viên dùng nội dung quét để đổi quà đúng một lần.
- **Giáo viên:** thẻ KPI và giao diện thống kê, 4 biểu đồ (gói hỗ trợ; danh mục; xu hướng theo tháng; khảo sát trước–sau), dữ liệu thật từ Redis, xuất 3 CSV (tổng hợp, giao dịch không định danh, khảo sát mã giả danh).
- **Bảo toàn dữ liệu:** dùng tiếp namespace Redis `seedwallet:hoangdieu:v4:state`, để tài khoản/gói của bản 4.0 không mất khi nâng cấp. **Sao lưu Redis trước khi nâng cấp.**
- **Kiểm thử:** `npm test` gồm 5 bài, trong đó 1 kiểm thử tích hợp giả lập Redis kiểm tra đăng ký → gieo → chặn nhận trước duyệt → duyệt → nhận → tạo QR → chặn dùng QR sai vai trò → đổi 1 lần → chặn đổi lại → khảo sát → báo cáo.

## Cài đặt Vercel

1. Giải nén ZIP, đưa **nội dung** thư mục dự án lên GitHub (thư mục `public/`, `api/`, `tests/`, cùng `package.json`, `vercel.json` ở thư mục gốc).
2. Vercel → New Project → Import repository → Framework Preset **Other**, Root Directory `./`, để build mặc định.
3. Trên Upstash tạo Redis Database; lấy REST URL và REST TOKEN. Thêm biến môi trường trên Vercel (Production, và Preview nếu dùng):

```env
KV_REST_API_URL=https://...upstash.io
KV_REST_API_TOKEN=...
SESSION_SECRET=<khoa_ngau_nhien_it_nhat_32_ky_tu>
ADMIN_USERNAME=quantri
ADMIN_PASSWORD=<mat_khau_manh_it_nhat_12_ky_tu>
```

   Có thể dùng tên `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN` thay hai biến KV; không cần cả hai cặp. Tuyệt đối không đưa token/mật khẩu lên GitHub.
4. Deploy/Redeploy. Kiểm tra `/api/index?action=health` trả về `ok:true`, `storage:"connected"` khi cấu hình tốt.
5. Dùng tài khoản `ADMIN_USERNAME` và `ADMIN_PASSWORD` đã khai báo để vào giao diện giáo viên. Đây là tài khoản giáo viên duy nhất do biến môi trường quản lý; không có mật khẩu mặc định trong mã nguồn.

## Kịch bản kiểm tra thực tế

1. Đăng ký hai học sinh thử nghiệm với mã khác nhau, mật khẩu tối thiểu 10 ký tự, PIN 4 số.
2. Học sinh A gieo một gói (ví dụ Bữa sáng căn tin).
3. Học sinh B thử nhận trước khi được duyệt → bị từ chối; giáo viên duyệt tại **Xét duyệt**.
4. Học sinh B chọn **Nhận mầm**, nhập PIN (thử sai một lần trước), xác nhận nhận → xuất hiện QR.
5. Mở **Ví của tôi**, xác minh QR có thể hiện lại. Giáo viên vào **Đổi mã QR**, quét bằng ứng dụng camera hoặc sao chép chuỗi ký tự, dán và xác nhận. Thử lại cùng mã → phải bị từ chối.
6. Giáo viên xem **Thống kê & Báo cáo**: biểu đồ và các số KPI thay đổi; tải ba CSV.
7. Kiểm tra bằng hai thiết bị, ẩn danh trình duyệt, sau khi tải lại trang, và đăng xuất/đăng nhập.
8. Kiểm tra Vercel → Logs để phát hiện lỗi API nếu có.

## Giới hạn & kiểm soát cần có trước sử dụng thật

- Redis hiện lưu toàn bộ dữ liệu trong một khóa JSON, cập nhật CAS nguyên tử. Phù hợp thử nghiệm nhỏ, **không phù hợp mở rộng lớn** hay bảo đảm an toàn dữ liệu trẻ em theo tiêu chuẩn vận hành chính thức. Cần chuyển sang mô hình dữ liệu phân tách, sao lưu, nhật ký bất biến, giám sát và kiểm toán.
- PIN 4 chữ số có entropy thấp; đã có giới hạn số lần gọi API theo cửa sổ thời gian, nhưng để dùng thật cần chống dò đoán theo từng tài khoản, quy trình cấp lại PIN xác thực danh tính và khóa tạm thời.
- Bộ khảo sát trong ứng dụng là biểu mẫu thí điểm, chưa phải thang tâm lý chuẩn hóa; biểu đồ trước/sau mô tả dữ liệu quan sát, **không** chứng minh tác động.
- Không sử dụng thông tin học sinh thật nếu chưa có văn bản chấp thuận của nhà trường, sự đồng thuận phù hợp, chính sách lưu/xóa dữ liệu và đánh giá an toàn độc lập.
- Giao diện sử dụng Chart.js và Lucide qua CDN; mất Internet với CDN có thể khiến biểu đồ hoặc biểu tượng không tải được, nhưng QR không phụ thuộc CDN.

## Phát triển cục bộ & kiểm thử

```bash
node --version  # Node.js 20 trở lên
npm test
npm install -g vercel
vercel dev
```

`vercel dev` cần tài khoản và cấu hình biến môi trường. Không cần cài thêm NPM dependency để tạo QR: mã tạo QR được đóng gói trong `api/vendor/QRCode/` (mã nguồn Apache-2.0 của dự án qrcode-terminal, kèm LICENSE).

Nếu dùng cùng Redis của phiên bản 4.0, hãy sao lưu `seedwallet:hoangdieu:v4:state` trước khi triển khai và **giữ nguyên SESSION_SECRET** để giải mã mã QR còn hiệu lực. Đổi SESSION_SECRET sẽ làm các phiên và mã đang giữ không thể khôi phục.
