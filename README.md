# VÍ HẠT MẦM 6.0 – TRƯỜNG THCS NGUYỄN TRÃI, TP. ĐÀ NẴNG

Phiên bản thử nghiệm chạy trên Vercel Functions + Upstash Redis, giao diện HTML/CSS/JavaScript tiếng Việt. Chỉ có **hai vai trò**: học sinh và giáo viên. Đây chưa phải nền tảng đã được chứng nhận để lưu dữ liệu trẻ em ngoài môi trường thử nghiệm có kiểm soát.

## Chức năng

**Học sinh:** đăng ký, đăng nhập; gieo/nhận mầm; mở mã QR trong ví; lịch sử; khảo sát tự nguyện; hồ sơ (đổi họ tên, lớp, mật khẩu, PIN); nhắn tin riêng với giáo viên.

**Giáo viên:** tổng quan, duyệt quyền nhận, quản lý danh mục, đổi mã QR một lần, biểu đồ và báo cáo CSV; sửa tên hồ sơ; nhắn tin riêng với từng học sinh. Mật khẩu giáo viên đổi tại biến `ADMIN_PASSWORD` trong Vercel, không đổi trong giao diện.

**Tin nhắn:** lưu trên Redis, kiểm tra hai phía tham gia ở API; học sinh không thể đọc/nhắn tin học sinh khác. Có giới hạn 800 ký tự/tin, 12 yêu cầu gửi/phút/IP và ID tài khoản, danh sách cập nhật khoảng 10 giây/lần. **Không mã hóa đầu cuối**: người vận hành máy chủ/database được cấp quyền kỹ thuật có thể truy cập dữ liệu. Không gửi thông tin nhạy cảm, dùng kênh trực tiếp khi khẩn cấp.

## Cách triển khai

1. Giải nén ZIP, đẩy nội dung gốc lên GitHub (`public`, `api`, `tests`, `package.json`, `vercel.json`).
2. Vercel → New Project → import repository, Framework Preset: **Other**, Root Directory: `./`.
3. Kết nối Upstash Redis và khai báo các biến môi trường (không đưa bí mật vào GitHub):

```env
KV_REST_API_URL=https://...upstash.io
KV_REST_API_TOKEN=<token_thuc_te>
SESSION_SECRET=<chuoi_ngau_nhien_it_nhat_32_ky_tu>
ADMIN_USERNAME=quantri
ADMIN_PASSWORD=<mat_khau_giao_vien_manh>
```

Có thể thay hai biến `KV_...` bằng `UPSTASH_REDIS_REST_URL` và `UPSTASH_REDIS_REST_TOKEN`. Chọn Production (và Preview nếu dùng), Save, rồi Redeploy.
4. Kiểm tra `https://TEN-DU-AN.vercel.app/api/index?action=health` trả `ok:true` và `storage:"connected"`.
5. Đăng nhập giáo viên với `ADMIN_USERNAME`, `ADMIN_PASSWORD`; tạo tài khoản học sinh thử nghiệm rồi kiểm tra theo kịch bản dưới đây.

## Kiểm thử thực tế

- Học sinh A đăng ký, gieo mầm; học sinh B đăng ký, chưa thể nhận trước khi giáo viên duyệt.
- Giáo viên duyệt B; B nhận bằng PIN, thấy QR; giáo viên đổi QR; quét lại QR bị chặn.
- Học sinh A gửi tin cho giáo viên; giáo viên thấy tin và trả lời; học sinh B **không** được xem cuộc trò chuyện A ↔ giáo viên.
- A cập nhật tên/lớp; đổi PIN bằng mật khẩu cũ; đổi mật khẩu buộc đăng nhập lại. Giáo viên đổi tên hiển thị trong hồ sơ.
- Kiểm tra tin nhắn/biểu đồ trên hai trình duyệt khác nhau, tải lại trang và đăng nhập lại.
- Xác thực chặn truy cập trái quyền; thử tải 2 người cùng nhận một gói; kiểm tra báo cáo CSV và Logs của Vercel.

Chạy kiểm thử tự động: `node --test tests/*.test.js` với Node.js 20+. Các bài kiểm thử sử dụng Redis giả lập trong bộ nhớ, không kết nối Upstash thật.

## Tách dữ liệu trường cũ và trường mới

Phiên bản này sử dụng Redis key **`seedwallet:nguyentrai:v6:state`** nên **không trộn dữ liệu của Trường THCS Hoàng Diệu**. Tài khoản/giao dịch cũ không tự chuyển sang trường mới. Nếu cần di chuyển dữ liệu, phải có sự cho phép và quy trình chuyển dữ liệu an toàn; không nên sao chép tài khoản học sinh giữa hai trường một cách tự động.

## Hạn chế trước triển khai chính thức

- Toàn bộ dữ liệu nghiệp vụ và tin nhắn hiện nằm trong một JSON Redis, cập nhật CAS: chỉ phù hợp thử nghiệm quy mô nhỏ. Chưa có backup tự động, quản lý vòng đời dữ liệu, xóa hội thoại, kiểm thử tải lớn hay kiểm toán độc lập.
- Chat lưu tối đa 1.200 tin gần nhất toàn hệ thống, mỗi lần hiển thị tối đa 100 tin cuối. Đây không phải giải pháp lưu trữ hồ sơ lâu dài, không bảo đảm lưu đủ bằng chứng xử lý vụ việc.
- Phụ huynh/nhà trường cần thông báo và đồng thuận thích hợp, chỉ định giáo viên phụ trách, quy trình xử lý báo cáo nhạy cảm và bảo mật dữ liệu trẻ em. Không sử dụng chat để xử lý nguy hiểm khẩn cấp.
- PIN chỉ có bốn chữ số, cần thêm cơ chế khóa theo tài khoản, kiểm thử bảo mật ứng dụng, phân quyền kỹ thuật, xác minh nguồn hỗ trợ trước khi giao nhận, giám sát và sao lưu.
- Dữ liệu khảo sát mô phỏng, không được coi là bằng chứng thực nghiệm KHKT. Không đưa dữ liệu học sinh thật vào cho đến khi có phê duyệt và đánh giá bảo mật.


## Bản 7.0 – Chat mã hóa đầu cuối (thử nghiệm bảo mật)
- Thêm `public/e2ee.js` dùng Web Crypto; không tải thư viện mã hóa bên thứ ba.
- Chat chỉ lưu bản mã AES-GCM, metadata; giao diện giải mã trên thiết bị. Khóa ECDH không xuất được, lưu IndexedDB ở từng trình duyệt.
- Người nhận cần vào mục Chat một lần để đăng ký khóa. **Chỉ hỗ trợ một thiết bị/trình duyệt cho mỗi tài khoản**; không có phục hồi khóa.
- Tin nhắn có thời hạn 30 ngày, cho phép xóa riêng từng tin và báo cáo sự cố.
- Đọc file `CHINH-SACH-CHAT-AN-TOAN.md` trước khi dùng.
- **Chưa đủ để triển khai thực tế cho trẻ em:** khóa công khai chưa xác minh độc lập, cần audit hệ thống và hỗ trợ khôi phục khóa, xử lý sự cố, bảo đảm nghĩa vụ pháp lý.
- Dùng cùng namespace Redis bản 6.0 để giữ tài khoản, nhưng tin nhắn **plaintext từ bản 6.0 không được hiển thị nữa**. Nếu từng chat thật bằng bản 6.0, phải xóa dữ liệu plaintext cũ bằng quy trình quản trị bảo mật trước triển khai.

## Bản 7.1 – Chẩn đoán lỗi Tin nhắn
- Thêm `/api/index?action=capabilities` xác nhận API chat đã triển khai và phiên bản máy chủ.
- `/api/index?action=health` trả thêm `version: 7.1.0` và danh sách tính năng chat.
- Giao diện kiểm tra API trước khi tạo khóa và thông báo rõ khi backend chưa tương thích.
- Tắt cache dài hạn của các tài nguyên tĩnh trên Vercel để hạn chế giao diện cũ.
- Chi tiết: `HUONG-DAN-SUA-LOI-CHAT.md`.
