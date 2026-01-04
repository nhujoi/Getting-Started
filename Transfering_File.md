
# Các phương pháp Chuyển Tập tin (File Transfer) trong Pentest

Trong quá trình kiểm tra thâm nhập, việc chuyển các công cụ khai thác hoặc tập lệnh sang máy chủ mục tiêu (hoặc lấy dữ liệu về) là rất quan trọng. Dưới đây là các phương pháp phổ biến để thực hiện việc này khi đã có quyền truy cập cơ bản (reverse shell).

## 1. Sử dụng giao thức HTTP (wget hoặc cURL)

Đây là phương pháp thiết lập máy tấn công trở thành một máy chủ web tạm thời để máy mục tiêu tải dữ liệu về.

* **Thiết lập máy chủ:** Tại thư mục chứa file cần chuyển trên máy tấn công, khởi chạy một máy chủ HTTP đơn giản (thường dùng module của Python).
* **Tải xuống:** Trên máy mục tiêu, sử dụng các công cụ tải xuống dòng lệnh phổ biến để kéo file về từ địa chỉ IP và cổng của máy tấn công:
    * **wget:** Công cụ tải xuống tiêu chuẩn.
    * **cURL:** Sử dụng thay thế nếu máy mục tiêu không có `wget`.

## 2. Sử dụng SCP (Secure Copy)

Phương pháp này dùng để sao chép file an toàn qua giao thức SSH.

* **Điều kiện:** Bạn cần phải có thông tin xác thực (tên đăng nhập và mật khẩu hoặc khóa SSH) của người dùng trên máy chủ từ xa.
* **Cách thức:** Chạy lệnh sao chép từ máy cục bộ, chỉ định đường dẫn file nguồn và đích đến trên máy chủ từ xa.

## 3. Mã hóa Base64 (Vượt tường lửa)

Phương pháp này hữu ích trong trường hợp máy chủ từ xa có tường lửa ngăn chặn việc tải xuống trực tiếp các tập tin từ bên ngoài.

* **Nguyên lý:** Chuyển đổi file (đặc biệt là file nhị phân) thành dạng chuỗi ký tự văn bản (text string).
* **Quy trình:**
    1.  **Mã hóa:** Tại máy tấn công, mã hóa file cần chuyển sang định dạng Base64.
    2.  **Sao chép:** Copy toàn bộ chuỗi ký tự Base64 đó.
    3.  **Giải mã:** Paste chuỗi ký tự vào máy mục tiêu và thực hiện giải mã ngược lại thành file gốc.

## 4. Xác thực dữ liệu (Verification)

Sau khi chuyển file, cần kiểm tra để đảm bảo file không bị lỗi hoặc hỏng trong quá trình truyền tải.

* **Kiểm tra định dạng:** Sử dụng công cụ kiểm tra loại file để xác nhận file trên máy mục tiêu đúng là định dạng mong muốn (ví dụ: file thực thi ELF).
* **Kiểm tra tính toàn vẹn (Checksum):**
    * Tính mã băm (ví dụ: MD5) của file gốc trên máy tấn công.
    * Tính mã băm của file vừa tải về trên máy mục tiêu.
    * So sánh hai mã băm này, nếu trùng khớp hoàn toàn nghĩa là file được truyền thành công và nguyên vẹn.