# Phân Tích Chuyên Sâu: Quy Trình Enumeration (Máy Nibbles)

Tài liệu này giải thích chi tiết **tại sao** các kỹ thuật quét mạng cụ thể được sử dụng, ý nghĩa của từng tham số cấu hình, và giá trị thông tin thu được trong quá trình kiểm thử xâm nhập (Pentest).

---

## 1. Giai đoạn 1: Quét Sơ Bộ (Initial Scan)

Đây là bước tiếp cận đầu tiên để có cái nhìn tổng quan về mục tiêu mà không làm hệ thống bị quá tải hoặc tốn quá nhiều thời gian.

### Chiến thuật sử dụng
Thực hiện quét 1.000 cổng phổ biến nhất (Top 1000 ports) bằng Nmap.
```
nmap -sV --open -oA nibbles_initial_scan 10.129.42.190
```

### Phân tích tham số (Flags)
* **Service Version (`-sV`):**
    * *Ý nghĩa:* Yêu cầu công cụ không chỉ kiểm tra cổng mở hay đóng, mà phải giao tiếp với cổng đó để xác định chính xác phần mềm và phiên bản đang chạy (Ví dụ: thay vì chỉ báo "Web", nó báo "Apache 2.4.18").
    * *Giá trị:* Đây là thông tin quan trọng nhất để tìm kiếm các lỗ hổng bảo mật (CVE) đã biết liên quan đến phiên bản phần mềm cụ thể đó.
* **Open Ports Only (`--open`):**
    * *Ý nghĩa:* Chỉ hiển thị kết quả các cổng đang mở.
    * *Giá trị:* Giúp lọc bỏ nhiễu từ hàng trăm cổng bị đóng (Closed) hoặc bị chặn (Filtered), giúp báo cáo ngắn gọn và dễ đọc hơn.
* **Output All (`-oA`):**
    * *Ý nghĩa:* Xuất kết quả ra đồng thời 3 định dạng: XML, Nmap, và Greppable.
    * *Giá trị:* Cực kỳ quan trọng để lưu trữ bằng chứng. Nếu mục tiêu thay đổi hoặc offline, hacker vẫn còn dữ liệu gốc để phân tích mà không cần quét lại.

### Kết quả thu được
* Phát hiện cổng **22 (SSH)** và **80 (HTTP)** đang mở.
* Dự đoán hệ điều hành là **Ubuntu** dựa trên thông tin phiên bản dịch vụ trả về.

---

## 2. Giai đoạn 2: Quét Toàn Bộ Cổng (Full Port Scan)

Sau khi có thông tin ban đầu, cần mở rộng phạm vi để đảm bảo không bỏ sót bất kỳ dịch vụ ẩn nào.

### Chiến thuật sử dụng
Quét toàn bộ dải cổng TCP từ 1 đến 65.535.
```
nmap -p- --open -oA nibbles_full_tcp_scan 10.129.42.190
```

### Phân tích tham số
* **Scan All Ports (`-p-`):**
    * *Lý do sử dụng:* Mặc định các công cụ quét chỉ kiểm tra 1.000 cổng phổ biến. Tuy nhiên, quản trị viên hệ thống thường áp dụng nguyên tắc "Bảo mật bằng cách che giấu" (Security through obscurity), chuyển các dịch vụ quan trọng sang các cổng lạ (ví dụ: SSH ở cổng 2222, Web Admin ở cổng 8080 hoặc 9999).
    * *Giá trị:* Đảm bảo bao quát 100% bề mặt tấn công của hệ thống.

### Kết quả thu được
* Xác nhận chắc chắn rằng mục tiêu **chỉ có** cổng 22 và 80 mở.
* Loại bỏ khả năng tồn tại các "cửa hậu" (backdoors) hoặc dịch vụ quản trị ẩn trên các cổng phi tiêu chuẩn.

---

## 3. Giai đoạn 3: Kiểm tra Banner Thủ công (Banner Grabbing)

Đôi khi các công cụ quét tự động có thể bị đánh lừa bởi tường lửa hoặc cấu hình sai. Việc kiểm tra thủ công là bước xác thực cần thiết.

### Chiến thuật sử dụng
Sử dụng công cụ **Netcat** (nc) để kết nối trực tiếp đến cổng dịch vụ.
```
nc -nv 10.129.42.190 22
```

### Phân tích tham số
* **No DNS (`-n`):** Bỏ qua bước phân giải tên miền giúp kết nối nhanh hơn, chỉ làm việc trực tiếp với IP.
* **Verbose (`-v`):** Hiển thị chi tiết quá trình bắt tay và kết nối để người dùng quan sát được phản hồi từ máy chủ.

### Tại sao cần bước này?
* Nmap hoạt động dựa trên cơ sở dữ liệu chữ ký (signatures). Netcat cho phép hacker nhìn thấy chính xác những gì máy chủ gửi lại (raw data).
* Ví dụ: Một dịch vụ SSH có thể gửi về chuỗi `SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8`. Chuỗi này xác nhận chính xác phiên bản phần mềm và bản vá hệ điều hành.

---

## 4. Giai đoạn 4: Quét bằng Script (Nmap Scripting Engine - NSE)

Đây là bước chuyển từ "phát hiện" sang "thu thập thông tin sâu" tự động hóa.

### Chiến thuật 1: Sử dụng Script Mặc định
Chạy các script an toàn để lấy thông tin cơ bản.
```
nmap -sC -p 22,80 -oA nibbles_script_scan 10.129.42.190
```

* **Tham số (`-sC`):** Chạy một tập hợp các script được coi là an toàn (safe) và phổ biến.
* *Mục đích:* Tự động lấy SSH Hostkey, tiêu đề trang web (HTML Title), kiểm tra chứng chỉ SSL/TLS mà không cần làm thủ công.

### Chiến thuật 2: Liệt kê Web chuyên sâu (HTTP Enum)
Dùng script chuyên dụng để tìm thư mục ẩn.

```
nmap -sV --script=http-enum -oA nibbles_nmap_http_enum 10.129.42.190
```

* **Tham số (`--script=http-enum`):**
    * *Cơ chế:* Script này hoạt động giống như một "từ điển". Nó sẽ thử ghép các đường dẫn phổ biến (như `/admin`, `/login`, `/robots.txt`, `/backup`) vào địa chỉ web để xem có tồn tại không.
    * *Giá trị:* Giúp phát hiện nhanh các thư mục nhạy cảm hoặc trang quản trị mà không cần dùng đến các công cụ nặng nề như Dirbuster hay Gobuster ngay lập tức.

### Kết quả thu được
* Trong trường hợp máy Nibbles, các script này không tìm thấy lỗi bảo mật nghiêm trọng ngay lập tức.
* Tuy nhiên, nó xác nhận trang web không có tiêu đề, gợi ý đây có thể là một trang web sơ sài hoặc đang trong quá trình phát triển.

---

## 5. Tổng kết Quy trình

Quy trình liệt kê trên tuân theo mô hình **"Cái Phễu"**:

1.  **Quét nhanh (Initial Scan):** Để xác định hướng tấn công (SSH & Web).
2.  **Quét rộng (Full Scan):** Để đảm bảo không bỏ sót ngóc ngách nào.
3.  **Kiểm tra chéo (Manual Verification):** Để xác thực độ tin cậy của dữ liệu.
4.  **Quét sâu (Script Scan):** Để tự động hóa việc tìm kiếm các cấu hình sai hoặc thông tin nhạy cảm.