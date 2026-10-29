# Chính sách nhắn tin an toàn – THCS Nguyễn Trãi

- Chỉ học sinh và giáo viên xác thực được nhắn tin; không có hội thoại học sinh–học sinh.
- Mã hóa trên trình duyệt bằng ECDH P-256, HKDF-SHA256 và AES-256-GCM. Redis chỉ lưu ciphertext, IV, salt, tài khoản tham gia, thời gian và trạng thái đọc.
- Khóa riêng được giữ trong IndexedDB trên **một thiết bị/trình duyệt**, không đồng bộ hay sao lưu. Mất thiết bị đồng nghĩa không đọc được nội dung cũ. Không được xóa dữ liệu trình duyệt trước khi có quy trình chuyển khóa an toàn.
- Khóa công khai do máy chủ phân phối chưa có kiểm chứng độc lập dấu vân tay: **không bảo đảm chống máy chủ độc hại thay thế khóa trong quá trình khởi tạo**. Vì vậy không được tuyên bố đạt E2EE đã kiểm toán hay bảo mật tuyệt đối.
- Tin nhắn mã hóa lưu tối đa 30 ngày theo thời gian tạo; dọn theo thao tác ghi, nội dung quá hạn bị ẩn ngay khi đọc. Có thể xóa từng tin cho cả hai phía; sao lưu cơ sở dữ liệu vẫn có thể còn bản mã trong thời gian lưu backup.
- Báo cáo hành vi được giữ tối đa 90 ngày trong hệ thống và giáo viên có màn hình tiếp nhận. Báo cáo không tự gửi nội dung tin đã mã hóa; chỉ tạo dấu vết metadata và lý do. Giáo viên phối hợp cán bộ bảo vệ trẻ em tiếp nhận chứng cứ do người báo cáo tự nguyện cung cấp trực tiếp. Có phương án bảo vệ người báo cáo và xử lý nguy cơ khẩn cấp ngoài chat.
- Quy trình xử lý: tiếp nhận -> đánh giá mức nguy cơ -> phản hồi/giới thiệu hỗ trợ -> lập hồ sơ tối thiểu -> xóa theo lịch; phân quyền người được xem hồ sơ sự vụ, không công khai danh tính học sinh.
- Không dùng chat thay kênh cứu trợ khẩn cấp. Phải có văn bản đồng ý của nhà trường, phụ huynh/người giám hộ và thông báo minh bạch phù hợp pháp luật trước khi dùng dữ liệu thật.
- Kiểm thử trước khi vận hành: thay khóa công khai, đổi thiết bị, mở hai phiên, đọc chéo vai trò, XSS, CSRF, chống dò mật khẩu, flood, Redis concurrency, retention job, khôi phục backup, báo cáo nguy cơ và quyền xóa dữ liệu.
- Audit không lưu plaintext. Quản trị hạ tầng có thể xem ciphertext và metadata, nhưng không mặc nhiên giải mã nội dung. Mã JavaScript cấp từ máy chủ vẫn là điểm tin cậy nhạy cảm.
