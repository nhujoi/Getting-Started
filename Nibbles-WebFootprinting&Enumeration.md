# Tóm tắt: Nibbles - Web Footprinting & Enumeration

Tài liệu này ghi lại quá trình thu thập thông tin web, từ việc xác định công nghệ, tìm kiếm thư mục ẩn, đến việc suy luận ra thông tin đăng nhập quản trị viên thông qua các lỗi cấu hình.

---

## 1. Nhận diện Ứng dụng Web (Web Identification)

**Bước 1:** Sử dụng công cụ `whatweb` để xác định công nghệ trang web.
* *Kết quả:* Chỉ nhận diện được máy chủ Apache chạy trên Ubuntu, chưa xác định được CMS cụ thể.
```
whatweb 10.129.42.190
```

**Bước 2:** Kiểm tra mã nguồn HTML (Page Source) bằng `curl`.
* *Kết quả:* Phát hiện một dòng chú thích (comment) HTML tiết lộ thư mục ẩn tên là **/nibbleblog/**.
```
curl http://10.129.42.190
```

**Bước 3:** Quét lại thư mục ẩn vừa tìm thấy.
* *Kết quả:* Xác định được công nghệ là **Nibbleblog** (CMS viết bằng PHP), jQuery và HTML5.
```
whatweb http://10.129.42.190/nibbleblog
```

---

## 2. Tìm kiếm Lỗ hổng & Liệt kê Thư mục

Sau khi xác định được CMS là Nibbleblog, chúng ta tìm kiếm trên Google và phát hiện CMS này có lỗ hổng **File Upload** (cho phép tải mã độc), nhưng cần quyền Admin để khai thác.

**Bước 4:** Sử dụng `Gobuster` để liệt kê các tệp tin và thư mục con trong `/nibbleblog/`.
* *Mục đích:* Tìm trang đăng nhập hoặc các file cấu hình quan trọng.
```
gobuster dir -u http://10.129.42.190/nibbleblog/ --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt
```

* *Kết quả:* Tìm thấy `admin.php` (trang quản trị), `README`, `content`, `plugins`...

**Bước 5:** Kiểm tra phiên bản phần mềm.
* Đọc file `README` để xem phiên bản hiện tại có khớp với phiên bản bị lỗi không.
* *Kết quả:* Phiên bản **v4.0.3** - Khả năng cao dính lỗ hổng File Upload đã tìm thấy.
```
curl http://10.129.42.190/nibbleblog/README
```

---

## 3. Truy tìm Thông tin Đăng nhập (Credential Hunting)

Để khai thác, cần phải đăng nhập được vào `admin.php`.

### Vấn đề gặp phải
Việc thử mật khẩu sai quá nhiều lần (Brute-force) đã kích hoạt cơ chế bảo vệ **"Blacklist protection"**, khiến địa chỉ IP của kẻ tấn công bị khóa. Do đó, không thể dùng công cụ dò mật khẩu như Hydra.

### Giải pháp: Khai thác lỗi Directory Listing
Do máy chủ bật tính năng liệt kê thư mục (Directory Listing), chúng ta có thể xem nội dung các thư mục nhạy cảm trong đường dẫn `/content/`.

**Bước 6:** Kiểm tra file `users.xml`.
* *Mục đích:* Xác thực tên người dùng.
* *Kết quả:* Tên đăng nhập là **admin**. File này cũng hiển thị danh sách các IP đã bị khóa (blacklist).
```
curl -s http://10.129.42.190/nibbleblog/content/private/users.xml | xmllint --format -
```

**Bước 7:** Kiểm tra file `config.xml`.
* *Mục đích:* Tìm manh mối về mật khẩu trong cấu hình trang web.
* *Kết quả:* Tìm thấy nhiều từ khóa liên quan đến **"Nibbles"** (tiêu đề trang, email `admin@nibbles.com`).
```
curl -s http://10.129.42.190/nibbleblog/content/private/config.xml | xmllint --format -
```

---

## 4. Kết luận & Suy luận Mật khẩu

Tổng hợp các dữ kiện thu thập được:
1.  CMS là Nibbleblog v4.0.3.
2.  Tên đăng nhập là **admin**.
3.  Không thể brute-force vì bị chặn IP.
4.  Tên máy chủ (Box name) và các từ khóa trong file cấu hình đều là **Nibbles**.

$\rightarrow$ **Suy luận:** Mật khẩu quản trị viên là **nibbles**.
* *Kết quả:* Đăng nhập thành công vào trang quản trị, mở đường cho bước khai thác tiếp theo (tải shell lên server).