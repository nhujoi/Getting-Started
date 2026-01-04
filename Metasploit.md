# Metasploit Notes

Dùng `nmap` để check scan xong xem xét nếu bất kỳ ứng dụng/dịch vụ nào có public exploits / SSH hoặc ftp.

--> `searchsploit` to search for a specific application: `searchsploit openssh 7.2`

Sau đó bắt đầu dùng Metasploit:
`msfconsole` --> vào mode

1. `search exploit [...]` : tìm exploit --> `use [...]`
2. `show options` : setting --> `set ... [port]` : mục tiêu exploit
3. `check` : chạy một check để đảm bảo server dễ bị tấn công --> `exploit`

---