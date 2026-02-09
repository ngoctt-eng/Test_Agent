# Tài liệu Đặc tả Nghiệp vụ & Kỹ thuật: Hệ thống AI Chat (v0.1)

## 1. Thông tin chung

**Mã Task:** [CP-212](https://www.google.com/search?q=https://ecomobi.atlassian.net/browse/CP-212) 

**Mục tiêu:** Tích hợp hệ thống AI Chat hỗ trợ Creator trên nền tảng Passio.

**Nền tảng:** Mobile App (Home Passio).


* 
**Design:** [Figma Link](https://www.figma.com/design/0KDnIO00OUGYTyRDA614IF/Other?node-id-5959-20092&t=kc2rjPLSJJGpL.Pu-1).



---

## 2. Luồng nghiệp vụ & Giao diện tại Màn hình Home (Home Passio)

### 2.1. Thành phần điều hướng AI Chat

| Thành phần | Mô tả nghiệp vụ | Ghi chú kỹ thuật |
| --- | --- | --- |
| **Floating Button** | Nút luôn hiển thị tại Home Passio. Khi tap sẽ mở màn hình AI Chat.

 | Không được phép tắt button này.

 |
| **Prompt Item** | Hiển thị danh sách các câu lệnh mẫu (prompts) được setup và active từ Admin.

 | Click vào item để mở AI Chat và tự động gửi câu hỏi tương ứng.

 |
| **Mini Conversation** | Hiển thị tiêu đề của session chat gần nhất dạng nổi (floating).

 | - Chỉ hiển thị tại Home Passio.

<br>

<br>- Ẩn khi user nhấn "X" hoặc kill app.<br>

<br>- Hiển thị lại khi user đóng một session chat bất kỳ.

 |

---

## 3. Đặc tả Chi tiết Màn hình Chat (Chat Interface)

### 3.1. Header & Quản lý Session

* 
**Animation Header:** Chỉ hiển thị 01 lần duy nhất trong lần đầu mở chat để hướng dẫn cách đóng khung chat.


* 
**Nút Close:** Thoát khung chat và quay về màn hình Home.


* 
**Conversation Name:** * Tự động tạo tên dựa trên câu hỏi đầu tiên của người dùng.


* Cho phép người dùng đổi tên (Độ dài: 1 - 50 ký tự).





### 3.2. Body Chat & Tin nhắn

* 
**Trạng thái mới:** Hiển thị lời chào: *"Passio xin chào Bạn đang cần hỗ trợ về vấn đề gì ạ?"* khi chưa có dữ liệu chat.


* 
**Tin nhắn User:** Hiển thị nội dung câu hỏi.


* 
**Tin nhắn AI:** * Hiển thị nội dung phản hồi từ AI dựa trên ngữ cảnh và tài liệu huấn luyện.


* **Xử lý Link:** Link in-app mở trực tiếp trong ứng dụng; Link ngoài mở qua WebView.


* **Phản hồi (Feedback):** Cho phép Like/Dislike cho từng câu trả lời. Chỉ được đánh giá 1 lần (chọn 1 trong 2, nút còn lại sẽ bị ẩn/highlight theo lựa chọn).





### 3.3. Cơ chế Suy nghĩ (Thinking Mode)

* 
**Thinking Block:** Hiển thị khi hệ thống đang xử lý câu hỏi (Typing indicator).


* **Full Reasoning:**
* Hiển thị toàn bộ quá trình "thought" của AI.


* Mở dưới dạng **Bottomsheet** khi user tap vào "Thinking" (Chỉ mở được sau khi AI đã hoàn tất câu trả lời).





---

## 4. Quản lý Lịch sử Chat (Side Bar / History)

* 
**Hiển thị:** Danh sách tối đa 10 đoạn chat gần nhất.


* 
**Thông tin:** Description hiển thị tối đa 2 dòng (quá 2 dòng hiển thị dấu "...").


* 
**Sắp xếp:** Theo thời gian cập nhật mới nhất (Sort by last updated descending).


* 
**Lưu trữ:** Lưu tối đa 60 ngày.


* **Tính năng:**
* 
**Chọn session:** Hiển thị lại toàn bộ data chat cũ.


* 
**Xóa (Delete):** Nhấn giữ (hold) session để hiển thị popup xác nhận xóa.


* 
**Đoạn chat mới:** Nút clear dữ liệu cũ để bắt đầu hội thoại mới.





---

## 5. Các ràng buộc kỹ thuật & Logic xử lý (Tech Notes)

### 5.1. Ràng buộc Input

* 
**Giới hạn:** Tối đa 355 ký tự.


* 
**Nút Gửi:** * Disable khi chưa nhập text hoặc khi hệ thống đang trong quá trình trả lời.


* Chỉ cho phép nhập/gửi 01 câu hỏi tại một thời điểm. Không xử lý đồng thời nhiều câu hỏi (spam).





### 5.2. Đồng bộ & Lưu trữ

* 
**Multi-device:** Hỗ trợ sync dữ liệu chat khi login trên nhiều thiết bị.


* 
**Persistence:** Giữ lịch sử chat ngay cả khi user kill app hoặc logout.



### 5.3. Xử lý lỗi & Edge Cases

* 
**Mạng yếu/AI chậm:** Hiển thị animation loading kéo dài.


* 
**Lỗi API/Timeout:** Hiển thị thông báo *"Sorry, something went wrong"* kèm nút **Retry** ngay dưới đoạn chat.


* 
**Trùng lặp chủ đề:** Nếu user tạo session mới với chủ đề đã tồn tại, hệ thống thông báo và hiển thị button điều hướng tới session cũ.

---

# Hệ Thống AI Chat - Tài Liệu Đặc Tả API (Full Specification)

## 1. Chat API v1 (Streaming & Reasoning)

Sử dụng khi cần tương tác trực tiếp với AI, hỗ trợ hiển thị quá trình suy luận (reasoning) và trả về các thành phần UI có cấu trúc.

### Thông tin kết nối

* 
**Endpoint:** `/v1/agents/chat/v1` 


* 
**Method:** `POST` 


* 
**Auth:** `Bearer Token` 


* 
**Servers:** * Dev: `https://dev.assistants-api.ecomobi.com` 


* Prod: `https://assistants-api.ecomobi.com` 





### Request Body

```json
{
  [cite_start]"message": "Ecomobi rank là gì?", // Câu hỏi của user [cite: 35]
  [cite_start]"conversationId": "conv_abc123", // ID để track conversation (tự tạo nếu trống) [cite: 35]
  "options": {
    [cite_start]"streamFormat": "ai-sdk", // Mặc định [cite: 37]
    [cite_start]"includeReasoning": true, // Hiển thị Phase 1 reasoning [cite: 37, 46]
    [cite_start]"includeStructuredData": true // Trả về structured UI components [cite: 37]
  }
}

```

### Response Sample (Streamed Data)

Dữ liệu được trả về theo từng chunk với các kiểu `type` khác nhau:

```json
// 1. Phân đoạn suy luận (Reasoning)
[cite_start]{"type":"reasoning","content":"Đang phân tích câu hỏi của bạn và tìm kiếm tài liệu liên quan..."} [cite: 48]

// 2. Phân đoạn nội dung (Content Delta)
[cite_start]{"type":"content","delta":"Ecomobi "} [cite: 51, 52]
[cite_start]{"type":"content","delta":"rank "} [cite: 53]
[cite_start]{"type":"content","delta":"là hệ thống..."} [cite: 60]

// 3. Kết thúc (Done)
[cite_start]{"type":"done", "finishReason": "stop", "conversationId":"conv_abc123"} [cite: 142]

```

---

## 2. Conversations API (Quản lý Danh sách)

Lấy danh sách tất cả các cuộc hội thoại của người dùng hiện tại.

### Thông tin kết nối

* 
**Endpoint:** `/v1/conversations` 


* 
**Method:** `GET` 


* 
**Params:** `page` (mặc định 1), `limit` (mặc định 20, max 100).



### Response Sample

```json
{
  "success": true,
  "data": [
    {
      [cite_start]"id": "conv_abc123", // Unique ID [cite: 315]
      [cite_start]"title": "Hỏi về Ecomobi rank và commission", // Tự động gen [cite: 315]
      [cite_start]"createdAt": "2025-11-10T08:30:00.000Z", [cite: 265]
      [cite_start]"updatedAt": "2025-11-10T09:45:00.000Z", [cite: 267]
      [cite_start]"messageCount": 12 [cite: 272]
    }
  ],
  "pagination": {
    "page": 1,
    "total": 45,
    "totalPages": 3,
    [cite_start]"hasNextPage": true [cite: 297, 301, 305, 306]
  }
}

```

---

## 3. Get Conversation Messages API (Chi tiết Hội thoại)

Truy xuất toàn bộ tin nhắn trong một cuộc hội thoại cụ thể với hỗ trợ phân trang.

### Thông tin kết nối

* 
**Endpoint:** `/v1/conversations/:id/messages` 


* 
**Method:** `GET` 


* 
**Params:** `page` (mặc định 1), `limit` (mặc định 50), `sortOrder` (asc/desc).



### Response Sample

```json
{
  "success": true,
  "data": [
    {
      [cite_start]"id": "msg_001", [cite: 180]
      [cite_start]"role": "user", [cite: 182]
      [cite_start]"content": "Ecomobi rank là gì?", [cite: 184]
      [cite_start]"createdAt": "2025-11-10T08:30:15.000Z", [cite: 186]
      [cite_start]"metadata": null [cite: 188]
    },
    {
      [cite_start]"id": "msg_002", [cite: 192]
      [cite_start]"role": "assistant", [cite: 194]
      [cite_start]"content": "Ecomobi rank là hệ thống xếp hạng publisher...", [cite: 196]
      "metadata": {
        [cite_start]"agentId": "ecomobi-agent", [cite: 202]
        [cite_start]"toolsUsed": ["retrieveDocuments"] [cite: 204]
      }
    }
  ],
  "pagination": {
    "page": 1,
    "total": 120,
    [cite_start]"hasNextPage": true [cite: 211, 214, 224]
  }
}

```

---

## 4. Bảng mã lỗi (Error Handling)

Cần xử lý các mã lỗi sau để đảm bảo trải nghiệm người dùng:

| Mã lỗi | Trạng thái | Mô tả |
| --- | --- | --- |
| **200** | OK | Request thành công.

 |
| **400** | Bad Request | Thiếu `message` hoặc dữ liệu không hợp lệ.

 |
| **401** | Unauthorized | Token hết hạn hoặc không có quyền truy cập.

 |
| **403** | Access Denied | Cố gắng truy cập hội thoại của người dùng khác.

 |
| **500** | Internal Error | Lỗi hệ thống từ server.

 |