# Tóm tắt chiến thuật tấn công tổng thể

## 1. Khai thác lỗ hổng
-> Có được quyền truy cập ban đầu (thường là **Reverse Shell**).

## 2. Nâng cấp Shell
-> Dùng **Python/Stty** để thao tác cho dễ (Full TTY).

## 3. Thu thập thông tin (Enumeration)
-> Chạy **LinPEAS** hoặc `sudo -l` để tìm đường leo quyền.

* **PrivEsc Checklist:** Tham khảo **HackTricks** hay **PayloadsAllTheThings**.
* **Enumeration Scripts (Tool tự động):**
    * **LinPEAS** (`./linpeas.sh`): *Lưu ý: Chạy tool này rất "ồn" (tạo nhiều log), dễ bị Admin hoặc Antivirus phát hiện.*

## 4. Leo quyền (PrivEsc)
-> Dùng lỗi Kernel hoặc Sudo để đạt quyền **Root**.

* **Kernel Exploits:** Tham khảo `dirtycow.github.io`.
* **Vulnerable Software:** Kiểm tra bằng `dpkg -l` hoặc xem trong `C:\Program Files`.
* **User Privileges (`sudo -l`):**
    * Lệnh này kiểm tra xem user hiện tại được phép chạy lệnh gì với quyền root mà không cần mật khẩu.
    * Tra cứu trên trang **GTFOBins**. Trang này hướng dẫn cách lợi dụng các lệnh hợp pháp (như `vim`, `echo`, `nmap`) để bẻ khóa lấy quyền shell root.
* **Scheduled Tasks:**
    * Hệ thống thường có các tác vụ chạy định kỳ. Nếu bạn có quyền **Ghi (write)** vào file script mà hệ thống chạy định kỳ, hoặc ghi vào thư mục cấu hình (`/etc/crontab`) -> Bạn sửa nội dung file script đó, thêm vào một câu lệnh Reverse Shell.
    * Khi đến giờ chạy, hệ thống (với quyền root) sẽ tự động chạy mã độc của bạn.
* **Exposed Credentials (Lộ thông tin xác thực):**
    * Tìm mật khẩu bị lưu hỏng trong file code (`config.php`), file log, hoặc lịch sử gõ lệnh (`.bash_history`).

### + SSH Key
* **Trộm chìa khóa (Steal Key):**
    * Nếu đọc được file `/root/.ssh/id_rsa` (Private Key).
    * Copy nội dung về máy bạn, set quyền `chmod 600 id_rsa` (bắt buộc, nếu không SSH sẽ từ chối).
    * Đăng nhập: `ssh -i id_rsa root@IP`
* **Cài chìa khóa (Plant Key):**
    * Nếu ghi được vào file `/root/.ssh/authorized_keys`.
    * Bạn tạo key trên máy bạn (`ssh-keygen`), lấy Public Key (`.pub`) dán vào file `authorized_keys` trên máy nạn nhân.
    * Từ đó về sau bạn có thể SSH thẳng vào root mà không cần mật khẩu.

## 5. Duy trì truy cập (Persistence)
-> Cài **Web Shell** hoặc mở **Bind Shell** để sau này quay lại cho dễ.
