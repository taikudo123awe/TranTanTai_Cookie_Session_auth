# API Xác thực bằng Session với Node.js, Express & MongoDB

Đây là một dự án API backend mẫu, xây dựng một hệ thống xác thực người dùng an toàn và đầy đủ chức năng bằng cách sử dụng session. Dự án này sử dụng các thư viện phổ biến và đáng tin cậy trong hệ sinh thái Node.js.

Khi người dùng đăng nhập, một session sẽ được tạo trên server và lưu trữ trong MongoDB. Một cookie chứa ID của session (mặc định là `connect.sid`) sẽ được gửi về client để xác thực cho các yêu cầu tiếp theo.

## Các công nghệ và tính năng chính

-   **Framework:** Express.js
-   **Cơ sở dữ liệu:** MongoDB với Mongoose ODM.
-   **Xác thực:** Quản lý session bằng `express-session`.
-   **Lưu trữ Session:** Lưu trữ session một cách bền bỉ trong MongoDB bằng `connect-mongo`, giúp người dùng không bị đăng xuất khi server khởi động lại.
-   **Bảo mật:**
    -   Mật khẩu được băm (hash) an toàn bằng `bcryptjs` trước khi lưu vào database.
    -   Sử dụng `httpOnly` cookie để ngăn chặn truy cập từ JavaScript phía client (chống tấn công XSS).
-   **API:** Cung cấp các endpoint RESTful cho các chức năng Đăng ký, Đăng nhập, Đăng xuất và truy cập tài nguyên được bảo vệ.

## Yêu cầu

-   [Node.js](https://nodejs.org/) (phiên bản 16.x trở lên)
-   npm (đi kèm với Node.js)
-   [MongoDB](https://www.mongodb.com/try/download/community) phải được cài đặt và đang chạy trên máy của bạn.

## Cài đặt & Khởi chạy

1.  **Thiết lập thư mục dự án:**
    Tạo một thư mục dự án và đặt các file `app.js`, `models/User.js`, `routes/auth.js` vào đúng cấu trúc.

2.  **Mở Terminal hoặc Command Prompt:**
    Di chuyển vào thư mục gốc của dự án.

3.  **Cài đặt các dependency:**
    Chạy lệnh sau để cài đặt tất cả các thư viện cần thiết:
    ```bash
    npm install express mongoose express-session connect-mongo cookie-parser bcryptjs
    ```

4.  **Chắc chắn rằng MongoDB đang chạy:**
    Hãy đảm bảo dịch vụ MongoDB của bạn đã được khởi động.

5.  **Khởi động máy chủ:**
    ```bash
    node app.js
    ```
    Bạn sẽ thấy thông báo: `Server running on http://localhost:3000`

## Mô tả các API Endpoints

Tất cả các endpoint đều có tiền tố là `/auth`.

| Phương thức | Endpoint | Mô tả | Yêu cầu Body (JSON) |
| :--- | :--- | :--- | :--- |
| **POST** | `/auth/register` | Đăng ký một tài khoản người dùng mới. Mật khẩu sẽ được tự động băm. | `{ "username": "...", "password": "..." }` |
| **POST** | `/auth/login` | Đăng nhập người dùng. Nếu thành công, server sẽ tạo session và trả về cookie. | `{ "username": "...", "password": "..." }` |
| **GET** | `/auth/profile` | Truy cập thông tin người dùng (được bảo vệ). Yêu cầu phải có cookie session hợp lệ. | (Không có) |
| **GET** | `/auth/logout` | Đăng xuất người dùng, hủy session trên server và xóa cookie ở client. | (Không có) |

---

## Hướng dẫn Test bằng Postman

Postman là công cụ lý tưởng để kiểm tra luồng xác thực này.

### 1. Đăng ký tài khoản (`/auth/register`)

-   **Method:** `POST`
-   **URL:** `http://localhost:3000/auth/register`
-   **Body:** Chọn `raw` và `JSON`. Nhập:
    ```json
    {
        "username": "testuser",
        "password": "password123"
    }
    ```
-   **Kết quả:** Bạn sẽ nhận được thông báo `{ "message": "User registered successfully!" }`.

### 2. Đăng nhập (`/auth/login`)

-   **Method:** `POST`
-   **URL:** `http://localhost:3000/auth/login`
-   **Body:** Sử dụng tài khoản bạn vừa tạo:
    ```json
    {
        "username": "testuser",
        "password": "password123"
    }
    ```
-   **Kết quả:**
    -   Bạn sẽ nhận được thông báo `{ "message": "Login successful!" }`.
    -   Quan trọng nhất: Kiểm tra tab **Cookies** trong Postman, bạn sẽ thấy một cookie mới tên là `connect.sid` đã được server gửi về. Postman sẽ tự động lưu và gửi cookie này cho các request tiếp theo tới `localhost:3000`.

![Postman Login with Session Cookie](https://i.imgur.com/g8vJk9k.png)

### 3. Truy cập route được bảo vệ (`/auth/profile`)

-   **Method:** `GET`
-   **URL:** `http://localhost:3000/auth/profile`
-   **Kết quả:**
    -   Do Postman tự động gửi kèm cookie `connect.sid`, request của bạn sẽ được xác thực thành công.
    -   Bạn sẽ nhận được thông tin của người dùng (không bao gồm mật khẩu), ví dụ:
        ```json
        {
            "_id": "63d8a7c1...",
            "username": "testuser",
            "__v": 0
        }
        ```

### 4. Đăng xuất (`/auth/logout`)

-   **Method:** `GET`
-   **URL:** `http://localhost:3000/auth/logout`
-   **Kết quả:**
    -   Bạn sẽ nhận được thông báo `{ "message": "Logout successful!" }`.
    -   Kiểm tra lại tab **Cookies**, bạn sẽ thấy cookie `connect.sid` đã bị xóa.

**Sau khi đăng xuất, nếu bạn thử gọi lại API `/auth/profile`, bạn sẽ nhận được lỗi `401 Unauthorized`, chứng tỏ session đã bị hủy thành công.**
