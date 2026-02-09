# 🛡️ Flutter Performance Gate & Anti-Jank Rules

Mục tiêu: Đảm bảo ứng dụng chạy mượt mà ở mức 60/120 FPS và không rò rỉ bộ nhớ.
Agent phụ trách: @performance_expert

---

### 1. Triệt tiêu "Jank" (Lag giao diện)
- **Quy tắc Build():** Tuyệt đối không thực hiện các tác vụ nặng (tính toán logic phức tạp, gọi API, khởi tạo Object lớn) bên trong hàm `build()`.
  - *Lỗi nặng:* `Text(calculateComplexHash(data))`
  - *Sửa:* Tính toán trước trong Logic Layer và truyền kết quả vào Widget.
- **Const Widgets:** Bắt buộc sử dụng từ khóa `const` cho tất cả các Widget không thay đổi. Điều này giúp Flutter bỏ qua việc rebuild các node này.
- **RepaintBoundary:** Bao bọc các Widget có hiệu ứng animation phức tạp (hoặc các phần thường xuyên thay đổi) bằng `RepaintBoundary` để cô lập vùng render.
- **ListView Optimization:** Luôn sử dụng `ListView.builder` hoặc `CustomScrollView` cho danh sách dài. Tuyệt đối không dùng `SingleChildScrollView` bọc `Column` chứa hàng trăm item.

### 2. Quản lý Bộ nhớ (Anti-Memory Leak)
- **Dispose Mandatory:** Tất cả các Controller (TextEditingController, AnimationController, ScrollController) và StreamSubscription phải được `dispose()` hoặc `cancel()` trong hàm `dispose()` của State.
- **Image Optimization:** Sử dụng `cacheWidth` và `cacheHeight` khi load ảnh từ mạng để tránh việc load ảnh 4K vào một khung hình 100x100.
- **Closure References:** Cẩn thận với các callback truyền vào các lớp Singleton hoặc Global Listener; hãy đảm bảo chúng được gỡ bỏ khi Widget bị hủy.

### 3. State Management Best Practices (Tùy chọn theo dự án)

#### [Nếu dùng BLoC]
- Sử dụng `buildWhen` và `listenWhen` để lọc các state không cần thiết, tránh rebuild lại toàn bộ UI khi chỉ một phần nhỏ thay đổi.

#### [Nếu dùng Provider/Riverpod]
- Ưu tiên sử dụng `.select()` thay vì watch toàn bộ Object để chỉ lắng nghe những thuộc tính cần thiết.

### 4. Tiêu chuẩn Kiểm tra của Agent
- **Độ trễ:** Mọi hàm xử lý dữ liệu đồng bộ (Sync) trên Main Thread không được vượt quá $16ms$. Nếu quá, Agent phải yêu cầu chuyển sang dùng `Isolate` (Compute function).
- **Complexity:** Cảnh báo các vòng lặp lồng nhau có độ phức tạp từ $O(n^2)$ trở lên trên các tập dữ liệu lớn.