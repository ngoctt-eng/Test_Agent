# Business Analysis: Tạo Video AI từ Hình ảnh và Văn bản

## 1. Tổng quan (Overview)
Tính năng cho phép người dùng tạo video động từ một hình ảnh tĩnh và nội dung văn bản (kịch bản), đồng thời tích hợp giọng đọc (Voiceover) được tùy chỉnh theo giới tính (Nam/Nữ) để phù hợp với ngữ cảnh của video.

## 2. Phân tích Yêu cầu (Requirements Analysis)

### 2.1. Yêu cầu đầu vào (Input)
*   **Hình ảnh (Source Image):**
    *   Người dùng tải lên 1 hình ảnh tĩnh (định dạng JPG, PNG).
    *   Yêu cầu ảnh rõ nét, có thể nhận diện khuôn mặt nếu là video nói chuyện (Talking Head).
*   **Văn bản (Script/Text):**
    *   Người dùng nhập nội dung văn bản muốn chuyển thành lời nói trong video.
    *   Giới hạn ký tự (ví dụ: tối đa 1000 ký tự cho mỗi lần tạo).
*   **Cấu hình giọng nói (Voice Settings):**
    *   **Giới tính (Gender):** Tùy chọn Nam (Male) hoặc Nữ (Female).
    *   (Tùy chọn nâng cao): Chọn tone giọng (Vui vẻ, Nghiêm túc, Kể chuyện).

### 2.2. Yêu cầu xử lý (Processing)
*   **Image-to-Video Engine:** Sử dụng mô hình AI để tạo chuyển động cho ảnh (ví dụ: SadTalker, D-ID, HeyGen, hoặc các mô hình open-source như EMO).
*   **Text-to-Speech (TTS) Engine:** Chuyển đổi văn bản thành âm thanh với giới tính đã chọn. Cần Lipsync (đồng bộ môi) khớp với âm thanh.
*   **Rendering:** Kết hợp video hình ảnh động và file âm thanh thành video hoàn chỉnh (MP4).

### 2.3. Đầu ra (Output)
*   Video định dạng MP4.
*   Độ phân giải tối thiểu 720p hoặc 1080p.
*   Hình ảnh nhân vật chuyển động môi khớp với lời thoại.

## 3. Luồng người dùng (User Flow)
1.  **Start:** Người dùng truy cập giao diện tạo video.
2.  **Upload:** Người dùng tải ảnh lên.
3.  **Input Text:** Người dùng nhập kịch bản (text).
4.  **Select Voice:** Người dùng chọn giới tính giọng đọc (Nam/Nữ).
5.  **Generate:** Nhấn nút "Tạo video".
6.  **Processing:** Hệ thống xử lý (Loading...).
7.  **Preview/Download:** Người dùng xem trước video và tải xuống.

## 4. Yêu cầu Phi chức năng (Non-Functional Requirements)
*   **Hiệu năng:** Thời gian render không quá lâu (ví dụ: < 2 phút cho video 30 giây).
*   **Chất lượng:** Giọng đọc tự nhiên, không quá máy móc. Chuyển động môi (Lip-sync) chính xác.
*   **Bảo mật:** Không lưu trữ hình ảnh người dùng quá thời gian quy định (nếu cần bảo mật riêng tư).

## 5. Đề xuất Kỹ thuật (Technical Focus)
*   **Backend:** Python (FastAPI/Django).
*   **AI Models:**
    *   TTS: Google TTS, ElevenLabs, OpenAI TTS.
    *   Video Gen: SadTalker, Wav2Lip, hoặc tích hợp API bên thứ 3 (D-ID, HeyGen) nếu cần chất lượng cao nhanh chóng.
    *   Local processing: Nếu chạy local, yêu cầu GPU mạnh.

## 6. Tiêu chí chấp nhận (Acceptance Criteria)
*   Video xuất ra chạy mượt mà.
*   Giọng nói đúng giới tính đã chọn.
*   Môi nhân vật cử động đồng bộ với âm thanh phát ra.
