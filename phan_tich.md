# Phân Tích Lỗ Hổng Sử Dụng System.out.println() Trên Môi Trường Production

Dưới đây là 3 lý do chí mạng tại sao tuyệt đối không được sử dụng `System.out.println()` và `e.printStackTrace()` trên môi trường Production thực tế:

1. **Gây nghẽn hệ thống và giảm hiệu năng nghiêm trọng (I/O Blocking):**
   Hàm `System.out.println()` là một phương thức đồng bộ (synchronized). Khi gọi hàm này, luồng xử lý chính (Main thread) của ứng dụng sẽ bị giữ lại để chờ hệ điều hành ghi chuỗi ký tự ra luồng đầu ra (Console/Log file). Đối với một chức năng có lượng truy cập cực kỳ lớn trong một thời điểm ngắn như **Flash Sale**, việc hàng ngàn luồng cùng bị nghẽn (Block) tại các lệnh `println()` sẽ khiến CPU quá tải, phản hồi API chậm hoặc thậm chí làm treo toàn bộ cụm Server.

2. **Mất vết dữ liệu và không thể lưu trữ lâu dài (No Persistence & Log Rotation):**
   Dữ liệu in ra từ `System.out.println()` chỉ đẩy trực tiếp vào bộ đệm Console (stdout). Khi ứng dụng chạy trên Docker/Kubernetes hoặc khi Server bị khởi động lại (Restart), toàn bộ vết log lưu trong bộ nhớ tạm thời của Console sẽ bị xóa sạch hoàn toàn. Hệ thống không có cơ chế tự động ghi file theo ngày (Log Rotation) hay đẩy log về các hệ thống quản lý tập trung (như ELK Stack, Datadog), biến ứng dụng thành một "hộp đen" không thể điều tra khi có lỗi ẩn phát sinh ngầm.

3. **Thiếu thông tin ngữ cảnh nghiệp vụ và phân cấp cấp độ lỗi (No Log Levels):**
   Các chuỗi ký tự từ `println()` xuất hiện rời rạc, không chứa mốc thời gian (Timestamp) chính xác đến mili-giây, không có tên Class/Method đang chạy, và không có ID luồng (Thread ID). Hơn nữa, nó cào bằng mọi loại thông tin. Sử dụng thư viện Logging (như SLF4J/Logback) giúp phân loại rõ ràng: Tin nhắn nghiệp vụ thường ngày dùng `INFO`, cảnh báo dùng `WARN`, và lỗi nghiêm trọng ném ra Exception dùng `ERROR`, giúp Dev dễ dàng bộ lọc, tìm đúng vết lỗi trong vài giây thay vì phải đọc thủ công hàng triệu dòng text lộn xộn.
