# Kiến trúc Hệ thống Quản lý Thư viện

## Mô tả các luồng cơ bản cho Role Thủ thư

### 1. Luồng Đăng nhập (Login)

#### 1.1. Mô tả tổng quan
Thủ thư đăng nhập vào hệ thống để có quyền truy cập các chức năng quản lý thư viện.

#### 1.2. Các bước thực hiện

1. **Truy cập trang đăng nhập**
   - Thủ thư truy cập URL: `/login` hoặc `/`
   - Hệ thống hiển thị form đăng nhập với các trường:
     - Username/Email
     - Password
     - Role selection (nếu có nhiều role)

2. **Nhập thông tin đăng nhập**
   - Thủ thư nhập username/email và password
   - Click nút "Đăng nhập" hoặc submit form

3. **Xác thực thông tin**
   - Hệ thống nhận request POST đến `/login`
   - Servlet/Controller xử lý:
     - Validate input (không để trống, format hợp lệ)
     - Kiểm tra username/email có tồn tại trong database
     - Kiểm tra password có khớp với hash trong database
     - Kiểm tra role của user có phải là "LIBRARIAN" (thủ thư) không
     - Kiểm tra tài khoản có bị khóa/vô hiệu hóa không

4. **Xử lý kết quả đăng nhập**

   **Trường hợp thành công:**
   - Tạo session cho user
   - Lưu thông tin user vào session:
     - User ID
     - Username
     - Role (LIBRARIAN)
     - Tên thủ thư
   - Tạo cookie (nếu có chức năng "Remember me")
   - Ghi log đăng nhập thành công
   - Redirect đến trang dashboard của thủ thư: `/librarian/dashboard`

   **Trường hợp thất bại:**
   - Hiển thị thông báo lỗi phù hợp:
     - "Tên đăng nhập hoặc mật khẩu không đúng"
     - "Tài khoản đã bị khóa"
     - "Bạn không có quyền truy cập với role này"
   - Giữ nguyên form đăng nhập
   - Ghi log đăng nhập thất bại (để bảo mật)

#### 1.3. Bảo mật
- Password được hash (bcrypt/argon2) trước khi lưu vào database
- Có cơ chế giới hạn số lần đăng nhập sai (ví dụ: 5 lần)
- Session timeout sau một khoảng thời gian không hoạt động
- Sử dụng HTTPS trong môi trường production

---

### 2. Luồng Đăng xuất (Logout)

#### 2.1. Mô tả tổng quan
Thủ thư đăng xuất khỏi hệ thống để kết thúc phiên làm việc và bảo mật thông tin.

#### 2.2. Các bước thực hiện

1. **Yêu cầu đăng xuất**
   - Thủ thư click nút "Đăng xuất" trên giao diện
   - Hoặc truy cập trực tiếp URL: `/logout`

2. **Xử lý đăng xuất**
   - Servlet/Controller nhận request GET hoặc POST đến `/logout`
   - Thực hiện các bước:
     - Ghi log đăng xuất (user ID, thời gian)
     - Xóa thông tin user khỏi session
     - Invalidate session: `session.invalidate()`
     - Xóa cookie (nếu có)
     - Clear các biến liên quan đến authentication

3. **Kết quả**
   - Redirect đến trang đăng nhập: `/login`
   - Hiển thị thông báo "Đã đăng xuất thành công" (nếu có)
   - User không thể truy cập các trang yêu cầu authentication nữa

#### 2.3. Bảo mật
- Đảm bảo session được invalidate hoàn toàn
- Clear tất cả dữ liệu nhạy cảm khỏi session
- Redirect về trang đăng nhập sau khi logout

---

### 3. Luồng Mượn sách (Checkout)

#### 3.1. Mô tả tổng quan
Thủ thư thực hiện quy trình cho độc giả mượn sách, ghi nhận thông tin mượn và cập nhật trạng thái sách.

#### 3.2. Các bước thực hiện

1. **Truy cập trang checkout**
   - Thủ thư đã đăng nhập vào hệ thống
   - Truy cập trang: `/librarian/checkout` hoặc từ dashboard
   - Hệ thống kiểm tra quyền truy cập (role LIBRARIAN)

2. **Nhập thông tin mượn sách**
   - Form checkout yêu cầu:
     - **Mã độc giả** (Reader ID) hoặc quét thẻ thư viện
     - **Mã sách** (Book ID) hoặc ISBN, hoặc quét mã vạch
     - **Ngày mượn** (mặc định: ngày hiện tại)
     - **Ngày hẹn trả** (tự động tính dựa trên quy định, có thể chỉnh sửa)

3. **Xác thực thông tin**
   - Hệ thống validate:
     - Độc giả có tồn tại và đang hoạt động không
     - Độc giả có bị khóa tài khoản không
     - Độc giả có đang nợ quá hạn không (có thể chặn hoặc cảnh báo)
     - Sách có tồn tại trong hệ thống không
     - Sách có đang available (có sẵn) không
     - Số lượng sách còn lại có đủ không

4. **Kiểm tra giới hạn mượn**
   - Kiểm tra số sách độc giả đang mượn:
     - Không vượt quá giới hạn mượn (ví dụ: tối đa 5 cuốn)
   - Kiểm tra loại độc giả (sinh viên, giáo viên, v.v.) để áp dụng quy định phù hợp

5. **Xử lý checkout**

   **Trường hợp thành công:**
   - Tạo bản ghi mượn sách (Loan/Borrowing record) trong database:
     - Loan ID (tự động)
     - Reader ID
     - Book ID
     - Librarian ID (thủ thư thực hiện)
     - Ngày mượn
     - Ngày hẹn trả
     - Trạng thái: "ACTIVE" hoặc "BORROWED"
   - Cập nhật trạng thái sách:
     - Số lượng available giảm đi 1
     - Trạng thái copy cụ thể: "BORROWED"
   - Cập nhật số sách đang mượn của độc giả
   - Ghi log transaction
   - Hiển thị thông báo thành công và thông tin:
     - Mã phiếu mượn
     - Thông tin sách
     - Thông tin độc giả
     - Ngày hẹn trả
   - In phiếu mượn (nếu có chức năng)

   **Trường hợp thất bại:**
   - Hiển thị thông báo lỗi cụ thể:
     - "Độc giả không tồn tại"
     - "Sách không có sẵn"
     - "Độc giả đã mượn quá số lượng cho phép"
     - "Độc giả đang nợ quá hạn"
   - Giữ nguyên form để thủ thư có thể sửa và thử lại

#### 3.3. Các trường hợp đặc biệt
- **Mượn nhiều sách cùng lúc**: Cho phép thêm nhiều sách vào một phiếu mượn
- **Gia hạn**: Có thể gia hạn ngày trả ngay khi checkout (nếu quy định cho phép)
- **Đặt chỗ trước**: Nếu sách đang được đặt chỗ, ưu tiên cho người đặt chỗ

---

### 4. Luồng Trả sách (Checkin)

#### 4.1. Mô tả tổng quan
Thủ thư thực hiện quy trình nhận sách trả từ độc giả, kiểm tra tình trạng sách và cập nhật trạng thái.

#### 4.2. Các bước thực hiện

1. **Truy cập trang checkin**
   - Thủ thư đã đăng nhập vào hệ thống
   - Truy cập trang: `/librarian/checkin` hoặc từ dashboard
   - Hệ thống kiểm tra quyền truy cập (role LIBRARIAN)

2. **Nhập thông tin trả sách**
   - Form checkin yêu cầu:
     - **Mã phiếu mượn** (Loan ID) hoặc quét mã vạch
     - Hoặc nhập **Mã sách** (Book ID) để tìm phiếu mượn tương ứng
     - **Ngày trả** (mặc định: ngày hiện tại)
     - **Tình trạng sách** (tùy chọn: tốt, hư hỏng nhẹ, hư hỏng nặng)

3. **Tìm kiếm phiếu mượn**
   - Hệ thống tìm bản ghi mượn sách:
     - Nếu nhập Loan ID: tìm trực tiếp
     - Nếu nhập Book ID: tìm phiếu mượn đang active của sách đó
   - Validate:
     - Phiếu mượn có tồn tại không
     - Phiếu mượn có đang ở trạng thái "ACTIVE" không
     - Sách có đang được mượn không

4. **Kiểm tra thông tin**
   - Hiển thị thông tin chi tiết:
     - Thông tin độc giả
     - Thông tin sách
     - Ngày mượn
     - Ngày hẹn trả
     - Số ngày đã mượn
     - Số ngày quá hạn (nếu có)

5. **Xử lý checkin**

   **Trường hợp thành công:**
   - Cập nhật bản ghi mượn sách:
     - Trạng thái: "RETURNED"
     - Ngày trả thực tế
     - Tình trạng sách khi trả
     - Librarian ID (thủ thư thực hiện)
   - Cập nhật trạng thái sách:
     - Số lượng available tăng lên 1
     - Trạng thái copy: "AVAILABLE"
   - Cập nhật số sách đang mượn của độc giả (giảm đi 1)
   - Xử lý phí (nếu có):
     - Tính phí quá hạn (nếu trả muộn)
     - Tính phí hư hỏng (nếu sách bị hư hỏng)
     - Tạo bản ghi phí (nếu có)
   - Ghi log transaction
   - Hiển thị thông báo thành công:
     - "Trả sách thành công"
     - Thông tin phí (nếu có)
     - Số sách còn đang mượn của độc giả

   **Trường hợp thất bại:**
   - Hiển thị thông báo lỗi:
     - "Không tìm thấy phiếu mượn"
     - "Sách này chưa được mượn"
     - "Phiếu mượn đã được xử lý trả sách rồi"
   - Giữ nguyên form để thủ thư có thể sửa và thử lại

#### 4.3. Xử lý các trường hợp đặc biệt

- **Trả quá hạn:**
  - Tính số ngày quá hạn
  - Tính phí quá hạn theo quy định
  - Cảnh báo độc giả
  - Có thể chặn mượn tiếp nếu nợ quá nhiều

- **Sách bị hư hỏng:**
  - Ghi nhận mức độ hư hỏng
  - Tính phí bồi thường (nếu có)
  - Cập nhật trạng thái sách: "DAMAGED" hoặc "REPAIR"
  - Có thể chuyển sang quy trình sửa chữa/thay thế

- **Trả sách không đúng:**
  - Kiểm tra ISBN/Mã sách có khớp không
  - Cảnh báo nếu không khớp
  - Yêu cầu xác nhận lại

- **Trả sách chưa đến hạn:**
  - Cho phép trả sớm
  - Không tính phí
  - Cập nhật bình thường

#### 4.4. Báo cáo và thống kê
- Sau khi checkin, có thể hiển thị:
  - Tổng số sách đã trả trong ngày
  - Số sách quá hạn
  - Doanh thu phí (nếu có)

---

### 5. Luồng tổng hợp - Quy trình làm việc của Thủ thư

#### 5.1. Phiên làm việc điển hình

1. **Bắt đầu ca làm việc**
   - Đăng nhập vào hệ thống
   - Kiểm tra dashboard: số sách cần xử lý, sách quá hạn, v.v.

2. **Xử lý mượn sách (Checkout)**
   - Độc giả đến mượn sách
   - Thủ thư thực hiện checkout
   - In phiếu mượn (nếu cần)

3. **Xử lý trả sách (Checkin)**
   - Độc giả đến trả sách
   - Thủ thư kiểm tra sách
   - Thực hiện checkin
   - Thu phí (nếu có)

4. **Kết thúc ca làm việc**
   - Xem báo cáo trong ngày
   - Đăng xuất khỏi hệ thống

#### 5.2. Các chức năng bổ sung
- Tìm kiếm thông tin mượn sách
- Gia hạn thời gian mượn
- Xem lịch sử giao dịch
- In báo cáo
- Quản lý phí và thanh toán

---

### 6. Các thành phần kỹ thuật liên quan

#### 6.1. Database Tables (tham khảo)
- `users` - Thông tin người dùng (bao gồm thủ thư)
- `readers` - Thông tin độc giả
- `books` - Thông tin sách
- `book_copies` - Thông tin các bản copy của sách
- `loans` - Bản ghi mượn sách
- `fees` - Phí quá hạn, hư hỏng
- `sessions` - Quản lý session đăng nhập

#### 6.2. Servlets/Controllers cần thiết
- `LoginServlet` - Xử lý đăng nhập
- `LogoutServlet` - Xử lý đăng xuất
- `CheckoutServlet` - Xử lý mượn sách
- `CheckinServlet` - Xử lý trả sách
- `LibrarianDashboardServlet` - Trang chủ của thủ thư

#### 6.3. JSP Pages cần thiết
- `login.jsp` - Trang đăng nhập
- `librarian/dashboard.jsp` - Dashboard thủ thư
- `librarian/checkout.jsp` - Trang mượn sách
- `librarian/checkin.jsp` - Trang trả sách
- `librarian/loan-details.jsp` - Chi tiết phiếu mượn

#### 6.4. Security Filters
- `AuthenticationFilter` - Kiểm tra đăng nhập cho các trang yêu cầu auth
- `RoleFilter` - Kiểm tra role (chỉ LIBRARIAN mới truy cập được các trang thủ thư)

---

### 7. Lưu ý triển khai

- Tất cả các luồng cần có validation đầy đủ ở cả client-side và server-side
- Cần có cơ chế logging để audit trail
- Cần có xử lý exception và error handling
- Cần có transaction management để đảm bảo tính nhất quán dữ liệu
- UI/UX cần thân thiện, dễ sử dụng cho thủ thư
- Có thể tích hợp quét mã vạch/QR code để tăng tốc độ xử lý

