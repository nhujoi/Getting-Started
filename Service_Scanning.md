# Tổng quan về Quét dịch vụ (Service Scanning)

Bước đầu tiên và quan trọng nhất trong việc khám phá mục tiêu là xác định hệ điều hành (OS) và các dịch vụ đang chạy.

* **Server (Máy chủ):** Các máy tính chuyên dụng lưu trữ dịch vụ để người dùng tương tác.
* **Mục tiêu:** Tìm kiếm các dịch vụ bị cấu hình sai hoặc có lỗ hổng (vulnerability) để ép dịch vụ thực hiện các hành động ngoài ý muốn (ví dụ: thực thi mã từ xa).

### Kiến thức về Port (Cổng)
* **Dải cổng:** Từ `1` đến `65,535`.
* **Well-known ports:** Từ `1` đến `1,023` (dành cho các dịch vụ đặc quyền/hệ thống).
* **Port 0:** Là cổng dự phòng (wildcard). Nếu một dịch vụ cố gắng gán vào Port 0, hệ thống sẽ tự động chuyển nó sang cổng khả dụng tiếp theo trên 1,024.

---

## 2. Công cụ Nmap (Network Mapper)

Nmap là tiêu chuẩn vàng trong ngành để tự động hóa quá trình quét cổng và phát hiện dịch vụ.

### Các câu lệnh phổ biến

| Lệnh | Ý nghĩa |
| :--- | :--- |
| `nmap <IP>` | Quét 1,000 cổng TCP phổ biến nhất (chế độ mặc định). |
| `-sV` | **Version scan:** Xác định phiên bản cụ thể của dịch vụ đang chạy. |
| `-sC` | **Default scripts:** Chạy bộ script kiểm tra cơ bản mặc định. |
| `-p-` | Quét toàn bộ 65,535 cổng TCP (quét kỹ). |
| `-A` | **Aggressive scan:** Quét tích cực, bao gồm OS detection, version, script, và traceroute. |

> **Mẹo:** Dựa vào banner phiên bản (ví dụ: *OpenSSH 8.2p1 Ubuntu*), bạn có thể tra cứu changelog trên Google để suy ra chính xác phiên bản hệ điều hành (ví dụ: *Ubuntu 20.04*).

---

## 3. Kỹ thuật Banner Grabbing

Kỹ thuật này dùng để nhận diện nhanh dịch vụ bằng cách "đọc" thông tin chào mừng (banner) mà dịch vụ gửi về khi vừa thiết lập kết nối.

**Sử dụng Netcat:**
nc -nv 10.129.42.253 21


**Sử dụng Nmap Script:**
nmap -sV --script=banner -p21 10.129.42.253


---

## 4. Chi tiết các dịch vụ mục tiêu phổ biến

### A. FTP (File Transfer Protocol) - Port 21
Dịch vụ truyền file, thường là điểm yếu do chứa dữ liệu nhạy cảm hoặc cấu hình sai cho phép người lạ truy cập.

* **Kiểm tra ẩn danh:** Quan sát nếu Nmap báo *Anonymous FTP login allowed* (Code 230).
* **Kết nối và tải file:**

ftp -p <IP>

Đăng nhập với user: anonymous (mật khẩu để trống hoặc nhập bất kỳ)
ls # Liệt kê danh sách file cd pub # Chuyển vào thư mục công khai (nếu có) get file.txt # Tải file về máy tấn công


### B. SMB (Server Message Block) - Port 445
Rất phổ biến trên Windows và Linux (qua Samba), dùng để chia sẻ file và máy in trong mạng LAN.

**Xác định OS qua SMB:**
nmap --script smb-os-discovery.nse -p445 <IP>


**Liệt kê các thư mục chia sẻ (Shares):**
smbclient -N -L \<IP>

-N: No password (đăng nhập ẩn danh)
-L: List shares

**Kết nối vào một Share cụ thể:**
smbclient -U <username> \<IP><share_name>


### C. SNMP (Simple Network Management Protocol) - Port 161/162
Giao thức quản lý thiết bị mạng. Nếu nắm được **Community String**, bạn có thể lấy thông tin cấu trúc mạng, tiến trình đang chạy, và đôi khi là mật khẩu.

* **Community Strings mặc định cần thử:** `public`, `private`.

**Truy vấn dữ liệu (Enumeration):**
snmpwalk -v 2c -c public <IP> 1.3.6.1.2.1.1.5.0


**Brute-force Community String (Dùng tool onesixtyone):**
onesixtyone -c dict.txt <IP>


---

## 5. Sử dụng Nmap Scripts (NSE)

NSE giúp mở rộng sức mạnh của Nmap từ quét cổng đơn thuần sang kiểm tra lỗ hổng (CVE), brute-force và thám thính nâng cao.

* **Tìm kiếm script có sẵn trong máy:** `locate scripts/<tên_dịch_vụ>`
* **Cấu trúc lệnh:**
    ```
    nmap --script <tên_script> -p <port> <IP>
    ```

**Ví dụ: Kiểm tra lỗ hổng trên Citrix:**
nmap --script citrix-enum-apps -p 443 <IP>