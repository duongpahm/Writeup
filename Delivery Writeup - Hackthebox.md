![[Screenshot 2026-01-29 at 15.18.29.png]]
## Port Scan
Thực hiện sử dụng nmap để scan port, ta thấy nmap tìm được 3 port TCP mở, SSH(22), HTTP(80), và có một máy chủ HTTP thứ hai trên cổng 8065:
```bash
┌──(root㉿duongpahm)-[/home/duongpahm]
└─# nmap -sVC 10.129.192.122       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-01 10:27 -0500
Nmap scan report for 10.129.192.122
Host is up (0.055s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-title: Welcome
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.01 seconds
```
![[Screenshot 2026-02-01 at 22.46.31.png]]
Dựa trên phiên bản OpenSSH, máy chủ có thể đang chạy Debian Buster (10). Các tập lệnh HTTP cho TCP 8065 hiển thị chuỗi “Mattermost”, vì vậy nó có thể là một phiên bản của phần mềm thay thế Slack mã nguồn mở đó.
## Website
Trang web này thực chất không có chức năng gì cụ thể, nhưng có đề cập đến việc kiểm tra bộ phận hỗ trợ để được hỗ trợ về các vấn đề liên quan đến email:
![[Screenshot 2026-02-01 at 22.37.05.png]]
Liên kết dẫn đến helpdesk.delivery.htb. Tôi sẽ thêm cả tên miền phụ đó và tên miền chính (delivery.htb) vào tệp /etc/hosts của mình.
Nhấp vào liên kết “CONTACT US” sẽ hiển thị một cửa sổ khác:
![[Screenshot 2026-02-01 at 22.49.00.png]]
Liên kết HelpDesk giống như liên kết ở trên. Liên kết máy chủ MatterMost là helpdesk.htb:8065, điều này giải thích cho cổng khác. Cũng có một số gợi ý ở đây về đường dẫn. Tôi cần có email @delivery.htb để có quyền truy cập vào máy chủ MatterMost.
![[Screenshot 2026-02-01 at 22.52.49.png]]
## helpdesk.delivery.htb - TCP 80
