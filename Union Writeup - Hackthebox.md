![[Screenshot 2026-01-28 at 17.19.26.png]]
## Port scanning
Như thường lệ với các bài lab, chúng ta bắt đầu bằng việc quét nmap để liệt kê thông tin.
```bash
┌──(root㉿duongpahm)-[/home/duongpahm]
└─# nmap -sVC 10.129.96.75                   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-28 05:26 -0500
Nmap scan report for 10.129.96.75
Host is up (0.27s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.24 seconds
```
## Enumeration
Kết quả scan trả về cho thấy, hiện tại chỉ mở cổng 80 và được chạy trên service http, bây giờ ta thực hiện truy cập vào giao diện web thông qua port 80.
![[Screenshot 2026-01-28 at 17.29.31.png]]
Khi truy cập giao diện website, chúng ta có thể thấy một chức năng kiểm tra điều kiện tham gia đơn giản, dùng để xác thực liệu người dùng có đủ tư cách tham dự vòng loại tháng 11 (November Qualifier) hay không.

Khi thực hiện tương tác và thử nghiệm các chức năng, hệ thống đã chuyển hướng  tới một liên kết khác, tại đó cho phép nhập một chuỗi _flag_. 
![[Screenshot 2026-01-28 at 17.40.47.png]]
Khi thử nhập ngẫu nhiên một chuỗi _flag_ theo định dạng của HTB, hệ thống không phản hồi hay sinh ra hành vi đáng chú ý nào. Có thể xác định tồn tại hai chức năng chính: cơ chế xác minh tư cách người chơi và cơ chế kiểm tra _flag_. Cả hai đều là các điểm đầu vào (input points) tiềm năng, nơi có thể tồn tại các lỗ hổng liên quan đến xử lý dữ liệu đầu vào như các dạng tấn công chèn (injection).
![[Screenshot 2026-01-28 at 17.43.13.png]]
![[Screenshot 2026-01-28 at 17.43.31.png]]
	