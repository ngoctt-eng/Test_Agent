# Test Cases: Tạo Video AI từ Hình ảnh và Văn bản

Dựa trên yêu cầu từ `ide.md` và tiêu chuẩn QA từ `agents/tester.md`.

## 1. Functional Testing (Kiểm thử Chức năng)

### TC-001: Tạo video thành công với ảnh hợp lệ và giọng nam
*   **Mục tiêu:** Xác minh hệ thống tạo được video từ ảnh và text với giọng nam.
*   **Preconditions:** Người dùng đã truy cập trang tạo video.
*   **Test Data:**
    *   Ảnh: `portrait_male.jpg` (Rõ nét, thấy mặt, định dạng JPG, < 5MB).
    *   Text: "Xin chào, đây là video thử nghiệm giọng nam."
    *   Giọng: Nam (Male).
*   **Steps:**
    1.  Click "Upload Image" và chọn `portrait_male.jpg`.
    2.  Nhập text vào ô Script.
    3.  Chọn giới tính giọng: "Nam".
    4.  Nhấn nút "Tạo video".
*   **Expected Results:**
    *   Hệ thống hiển thị trạng thái đang xử lý.
    *   Video được tạo thành công sau < 2 phút.
    *   thời lượng video tối đa là 30s                           
    *   Video có hình ảnh nhân vật chuyển động môi khớp với lời thoại.
    *   Âm thanh là giọng nam rõ ràng.

### TC-002: Tạo video thành công với ảnh hợp lệ và giọng nữ
*   **Mục tiêu:** Xác minh hệ thống tạo được video từ ảnh và text với giọng nữ.
*   **Test Data:**
    *   Ảnh: `portrait_female.png` (Rõ nét, JPG).
    *   Text: "Xin chào, đây là video thử nghiệm giọng nữ."
    *   Giọng: Nữ (Female).
*   **Steps:**
    1.  Upload ảnh `portrait_female.png`.
    2.  Nhập text.
    3.  Chọn giới tính giọng: "Nữ".
    4.  Nhấn nút "Tạo video".
*   **Expected Results:**
    *   Video được tạo thành công.
    *   Âm thanh là giọng nữ.
    *   Lip-sync chính xác.

### TC-003: Validation - Không tải ảnh
*   **Mục tiêu:** Kiểm tra xử lý khi thiếu ảnh đầu vào.
*   **Steps:**
    1.  Không upload ảnh.
    2.  Nhập text và chọn giọng.
    3.  Nhấn nút "Tạo video".
*   **Expected Results:**
    *   Nút "Tạo video" bị disabled hoặc hiển thị thông báo lỗi "Vui lòng tải ảnh lên".

### TC-004: Validation - Không nhập text
*   **Mục tiêu:** Kiểm tra xử lý khi thiếu nội dung văn bản.
*   **Steps:**
    1.  Upload ảnh hợp lệ.
    2.  Để trống ô text.
    3.  Nhấn nút "Tạo video".
*   **Expected Results:**
    *   Hiển thị thông báo lỗi "Vui lòng nhập nội dung văn bản".

### TC-005: Validation - Text vượt quá giới hạn
*   **Mục tiêu:** Kiểm tra giới hạn ký tự (giả định 1000 ký tự).
*   **Steps:**
    1.  Upload ảnh hợp lệ.
    2.  Nhập đoạn văn bản > 1000 ký tự.
    3.  Nhấn nút "Tạo video".
*   **Expected Results:**
    *   Hiển thị cảnh báo "Văn bản không được quá 1000 ký tự".
    *   Không cho phép tạo video.

### TC-006: Validation - Định dạng ảnh không hỗ trợ
*   **Mục tiêu:** Kiểm tra hệ thống từ chối file không phải ảnh (VD: .txt, .pdf, .exe).
*   **Steps:**
    1.  Upload file `document.pdf`.
*   **Expected Results:**
    *   Thông báo lỗi "Định dạng file không hỗ trợ. Vui lòng chọn JPG hoặc PNG".

## 2. Non-Functional Testing (Phi chức năng)

### TC-PERF-001: Thời gian xử lý (Performance)
*   **Mục tiêu:** Đảm bảo thời gian tạo video nằm trong giới hạn chấp nhận được.
*   **Steps:**
    1.  Thực hiện tạo video với script dài (khoảng 30 giây thời lượng nói).
    2.  Bấm đo thời gian từ lúc nhấn "Tạo video" đến khi video sẵn sàng.
*   **Expected Results:**
    *   Thời gian xử lý < 2 phút (như yêu cầu trong `ide.md`).

### TC-QUAL-001: Chất lượng Lip-sync
*   **Mục tiêu:** Đảm bảo chuyển động môi tự nhiên.
*   **Steps:**
    1.  Tạo video với các phụ âm khó (B, P, M).
    2.  Quan sát kỹ vùng miệng nhân vật trong video kết quả.
*   **Expected Results:**
    *   Môi nhân vật khép/mở đồng bộ với âm thanh, không bị trễ hoặc lệch quá nhiều.

### TC-SEC-001: Bảo mật data người dùng
*   **Mục tiêu:** Đảm bảo ảnh người dùng không bị lộ hoặc lưu vĩnh viễn (nếu có chính sách xóa).
*   **Steps:**
    1.  Upload ảnh và tạo video.
    2.  Kiểm tra đường dẫn ảnh gốc sau 24h (nếu có quy định xóa temp file).
*   **Expected Results:**
    *   File ảnh tạm nên bị xóa hoặc không thể truy cập công khai nếu không có quyền.

## 3. Edge Cases (Trường hợp Góc cạnh)

### TC-EDGE-001: Ảnh không rõ khuôn mặt
*   **Input:** Ảnh phong cảnh hoặc ảnh chụp sau lưng (không có mặt).
*   **Expected:** Hệ thống cảnh báo "Không tìm thấy khuôn mặt trong ảnh" hoặc xử lý thất bại có thông báo rõ ràng.

### TC-EDGE-002: Text chứa ký tự đặc biệt hoặc ngôn ngữ lạ
*   **Input:** Text chứa emoji, ký tự toán học, hoặc ngôn ngữ không được hỗ trợ (nếu hệ thống chỉ hỗ trợ Tiếng Việt/Anh).
*   **Expected:** Hệ thống xử lý được (bỏ qua ký tự lạ) hoặc thông báo lỗi cụ thể, không bị crash.
