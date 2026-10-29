# Ví Hạt Mầm 7.1 – Sửa lỗi “Không tìm thấy chức năng” ở mục Tin nhắn

## Nguyên nhân
Thông báo xuất phát từ API khi `action` không trùng với chức năng được máy chủ hỗ trợ. Mã nguồn bản 7.0 có các API `chat-list`, `chat-key`, `chat-thread`, nhưng phiên bản giao diện và API triển khai trên Vercel có thể khác nhau nếu chỉ cập nhật một phần repository, deploy ở nhánh khác hoặc trình duyệt còn giữ JS cũ. Không thể xác định chắc chắn nguyên nhân thực tế nếu chưa xem bản deploy của người dùng.

## Cách cập nhật
1. Giải nén gói ZIP 7.1. Tải toàn bộ nội dung gốc (`api/`, `public/`, `tests/`, `vercel.json`, `package.json`...) lên đúng GitHub repository đang kết nối dự án Vercel. Không tải nguyên ZIP lên repo.
2. Vercel → Project → Settings → Git: kiểm tra repository và production branch; Root Directory phải là `./`.
3. Deployments: mở deployment mới nhất và xác nhận trạng thái Ready, đúng commit đã tải lên. Nếu thay đổi biến môi trường, Redeploy.
4. Mở `https://TEN-MIEN-CUA-BAN/api/index?action=health`. Kết quả phải có `"version":"7.1.0"`, `"features":{"chatE2EE":true,...}` và `"storage":"connected"`.
5. Mở `https://TEN-MIEN-CUA-BAN/api/index?action=capabilities`. Kết quả phải có `"chatE2EE":true` và `"chatActions"` gồm `chat-list`, `chat-key`, `chat-thread`.
6. Nhấn Ctrl+Shift+R (Windows) hoặc Cmd+Shift+R (Mac) rồi đăng nhập lại. Vào Tin nhắn.
7. Giáo viên cần đăng nhập ít nhất một lần để tài khoản được tạo ở database và mở mục Tin nhắn để khởi tạo khóa; học sinh sau đó mới có thể chọn giáo viên và nhắn tin.

## Nếu chưa đúng
- `health` không chứa `version` hoặc `version` khác `7.1.0`: Vercel đang chạy API cũ hoặc sai deployment. Kiểm tra root directory, nhánh Git, tệp `api/index.js` và trạng thái build.
- `health` báo lỗi Redis: kiểm tra `KV_REST_API_URL`, `KV_REST_API_TOKEN`, kết nối Upstash và redeploy.
- `capabilities` báo không tìm thấy chức năng: server vẫn chưa dùng file API bản 7.1.
- Có khóa công khai trên server nhưng thiết bị không còn khóa riêng trong IndexedDB: không xóa/ghi đè khóa tùy tiện, cần quy trình khôi phục khóa được xác minh, và các tin cũ có thể không khôi phục được.

## An toàn
Đây là bản thử nghiệm E2EE, chưa được kiểm toán độc lập và chưa bảo đảm an toàn với dữ liệu học sinh thật. Không gửi mật khẩu, Redis token hay SESSION_SECRET khi nhờ hỗ trợ.
