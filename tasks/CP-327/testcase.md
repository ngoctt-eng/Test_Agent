## 2. Chi tiết Test Case (Detailed Test Cases)

Dưới đây là danh sách chi tiết các trường hợp kiểm thử, bao gồm điều kiện, các bước thực hiện, kết quả mong muốn và trạng thái thực tế.

| ID | Module/Category | Mục đích / Các bước thực hiện (Description/Steps) | Kết quả mong muốn (Expected Result) | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| **1** | Product and Reward | **Button Add:** Hiển thị | Hiển thị như design | Medium | **Pass** |
| **2** | Product and Reward | **Click button:** Hệ thống hiển thị thông tin product để user điền | Hệ thống hiển thị thông tin product để user điền | Medium | **Pass** |
| **3** | Product ID | **Hiển thị:** Hiển thị dropdown list ID sản phẩm | Hiển thị dropdown list ID sản phẩm | Medium | **Pass** |
| **4** | Product ID | **User nhập ký tự số:** User nhập ký tự số | Hiển thị list ID chứa kí tự user nhập | Medium | **Pass** |
| **5** | Product ID | **Validate format:** Nhập ký tự không phải số hoặc ký tự đặc biệt | Hệ thống báo lỗi hoặc không cho phép nhập (Nếu ID là dạng số) | High | *(New)* |
| **6** | Product Name | **Hiển thị:** Hệ thống tự fill tên sp dựa theo ID được chọn | Hệ thống tự fill tên sp dựa theo ID được chọn | Medium | **Pass** |
| **7** | Product Name | **Validate:** Disable field không cho edit | Disable field không cho edit | Medium | **Pass** |
| **8** | Product URL | **Hiển thị:** Hệ thống tự fill link của sp dựa theo ID được chọn | Hệ thống tự fill link của sp dựa theo ID được chọn | Medium | **Pass** |
| **9** | Product URL | **Validate:** Disable field không cho edit | Disable field không cho edit | Medium | **Pass** |
| **10** | Product Description | **Cho phép user điền thông tin sản phẩm** | Cho phép user điền thông tin sản phẩm | Medium | **Pass** |
| **11** | Product Description | **Validate format:** Kiểm tra giới hạn ký tự (vượt quá MaxLength) | Không cho phép nhập tiếp hoặc hiển thị cảnh báo quá giới hạn | Medium | *(New)* |
| **12** | Rewards | **Hiển thị:** Hiển thị 2 option (Commission, Free sample) | Hiển thị 2 option (Commission, Free sample) | Medium | **Pass** |
| **13** | Rewards | **Tích chọn Commission:** Hiển thị input field nhập hoa hồng nhận được | Hiển thị input field nhập hoa hồng nhận được | Medium | **Pass** |
| **14** | Commission | **Validate format:** Nhập số âm hoặc số > 100 | Hiển thị validate "Commission must be between 0 and 100" | High | *(New)* |
| **15** | Commission | **Validate format:** Nhập ký tự đặc biệt/chữ | Chỉ cho phép nhập số (integer/decimal) | High | *(New)* |
| **16** | Rewards | **Tích chọn Free sample:** Campaign này cho phép request sản phẩm mẫu | Campaign này cho phép request sản phẩm mẫu | Medium | **Pass** |
| **17** | Delete product | **Hiển thị:** Hiển thị như design | Hiển thị như design | Medium | **Pass** |
| **18** | Delete product | **Click button:** Hiển thị popup confirm | Hiển thị popup confirm | Medium | **Pass** |
| **19** | Popup confirm | **Text:** "Are you sure to delete this product? This action cannot be undone" | Hiển thị đúng text cảnh báo | Medium | **Pass** |
| **20** | Popup confirm | **Button Cancel:** Click button | Hủy bỏ thao tác xóa sản phẩm | Medium | *(Blank)* |
| **21** | Popup confirm | **Button Confirm:** Click button | Hệ thống xóa sản phẩm tương ứng khỏi danh sách | Medium | *(Blank)* |
| **22** | Condition | **Max sample:** Enter max sample quantity/KOL (Place holder) | Enter max sample quantity/KOL | Medium | **Pass** |
| **23** | Condition | **Validate Max sample:** Nhập số không phải số tự nhiên (số âm, thập phân, chữ) | Chỉ cho phép nhập số tự nhiên (Positive Integer) | Medium | *(New)* |
| **24** | Condition | **User nhập thông tin:** Nhập giá trị nhỏ hơn tổng số SKU được tích free sample | Cho phép user nhập | Medium | **Pass** |
| **25** | Condition | **User nhập thông tin:** Nhập giá trị bằng tổng số SKU được tích free sample | Cho phép user nhập | Medium | **Pass** |
| **26** | Condition | **User nhập thông tin:** Nhập giá trị lớn hơn tổng số SKU được tích free sample | Hiển thị validate "Please enter a value less than or equal to the total number of Free Sample products!" | Medium | **Pass** |
| **27** | Content type accepted | **Hiển thị:** Hiển thị dropdown list gồm các giá trị (Tiktok video, Tiktok livestream, Others) | Hiển thị dropdown list chính xác | Medium | *(Blank)* |
| **28** | Content type accepted | **Tooltip:** "The system will only automatically run the acceptance job for the Tiktok Video type. Other types require manual review" | Hiển thị tooltip đúng nội dung | Medium | **Pass** |
| **29** | Content type accepted | **Validate:** Cho phép chọn nhiều giá trị | Cho phép chọn nhiều giá trị | Medium | **Pass** |
| **30** | Total Content | **Validate:** Chỉ cho phép nhập số tự nhiên | Chỉ cho phép nhập số tự nhiên | Medium | **Pass** |
| **31** | Required Product Count | **Validate:** Chỉ cho phép nhập số tự nhiên | Chỉ cho phép nhập số tự nhiên | Medium | **Pass** |
| **32** | Campaign | **Default:** Hiển thị checkbox Cash chưa được tích | Hiển thị checkbox Cash chưa được tích | Medium | **Pass** |
| **33** | Rewards | **Tích chọn Cash:** Hiển thị field Cash receive | Hiển thị field Cash receive | Medium | **Pass** |
| **34** | Rewards | **Field Cash receive:** Chỉ cho nhập số tự nhiên | Chỉ cho nhập số tự nhiên | Medium | **Pass** |
| **35** | UGC campaign detail | **Campaign có 5 sp trở xuống:** Hiển thị mỗi sản phẩm 1 dòng (Product ID và Product name) | Hiển thị đúng danh sách sản phẩm | Medium | **Pass** |
| **36** | UGC campaign detail | **Hover:** Hold chuột vào Product ID | Hiển thị tooltip thông tin sản phẩm (ID, Name, URL, Reward) | Medium | **Pass** |
| **37** | UGC campaign detail | **Hover:** Hold chuột vào Product name | Hiển thị tooltip thông tin sản phẩm | Medium | **Pass** |
| **38** | UGC campaign detail | **Campaign có trên 5 sp:** Hiển thị tối đa 3 sản phẩm và button "+ x more product" | Hiển thị tối đa 3 sản phẩm và nút xem thêm | Medium | *(Blank)* |
| **39** | Tab submission | **Variant (Camp chỉ có 1 sp):** Hiển thị tất cả các sản phẩm mẫu do Creator đăng ký (Product ID Variant) | Hiển thị đúng thông tin variant | Medium | **Pass** |
| **40** | Tab submission | **Variant (Camp có nhiều sp):** Hiển thị mỗi sp 1 dòng (VD: Tẩy trang hồng, Son màu đỏ...) | Hiển thị mỗi sản phẩm một dòng | Medium | *(Blank)* |
| **41** | Tab submission | **Content URL:** Hiển thị link Content URL mà Creator submit nghiệm thu | Hiển thị link Content URL | Medium | **Pass** |
| **42** | Tab submission | **Click link:** Click link Content URL | Hệ thống điều hướng đến link đích | Medium | **Pass** |
| **43** | Man submission detail | **Content Type:** Hiển thị loại content mà Creator chọn khi submit video nghiệm thu | Hiển thị đúng loại content | Medium | **Pass** |
| **44** | Man submission detail | **Variant (Camp chỉ có 1 sp):** Hiển thị danh sách sản phẩm (Product ID, Name, Variant) | Hiển thị tất cả sản phẩm mẫu Creator đăng ký | Medium | **Pending** |
| **45** | Man submission detail | **Camp có nhiều sp:** Hiển thị mỗi sp 1 dòng. Hiển thị đúng Content Type. | Hiển thị mỗi sp 1 dòng. Hiển thị đúng Content Type. | High | **Pending** |
| **46** | Man submission detail | **Admin chuyển tay trạng thái sub và BE chưa validate:** Hiển thị content type Unidentified | Hiển thị content type Unidentified | High | **Pending** |
| **47** | Man submission detail | **Category lấy về không đạt điều kiện:** Hiển thị content type Not found | Hiển thị content type Not found | High | **Pending** |
| **48** | Required Product Count | **Validate hiển thị:** Hiển thị số lượng sản phẩm Product Unique thuộc Campaign | Hiển thị số lượng sản phẩm chính xác | High | **Fail** |
| **49** | Video Acceptance | **Content type Tiktok video:** Validate video nghiệm thu | Hệ thống tự động chạy job nghiệm thu | High | *(Blank)* |
| **50** | Video Acceptance | **Content type các type còn lại:** Validate video nghiệm thu | Admin phải review thủ công | High | *(Blank)* |
| **51** | Video Acceptance | **Content type các type còn lại:** Validate video nghiệm thu | Admin phải review thủ công | High | *(Blank)* |