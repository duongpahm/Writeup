
![[Screenshot 2026-01-28 at 10.27.46.png]]

## Recon 
Như các lab khác, đầu tiên chúng ta sử dụng nmap để thực hiện scan ip
```bash
┌──(root㉿duongpahm)-[/home/duongpahm]
└─# nmap -sVC 10.49.157.12
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-27 22:37 -0500
Nmap scan report for 10.49.157.12
Host is up (0.11s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-title: Apache Tomcat/9.0.30
|_http-favicon: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.76 seconds 
```
Kết quả scan cho thấy target là một máy chủ Ubuntu đang vận hành dịch vụ Apache Tomcat version 9.0.30. Ta thấy cổng 22 đang mở service ssh và 8009 đang mở trên service ajp13 và cổng 8080 là http.
Ta thấy cổng 8009 được mở, ta sử dụng công cụ khai thác có sẵn trên Github.
[Ghostcat-CNVD](https://github.com/00theway/Ghostcat-CNVD-2020-10487?source=post_page-----23a9a1ae4a23---------------------------------------)
```bash
git clone https://github.com/00theway/Ghostcat-CNVD-2020-10487.git
cd Ghostcat-CNVD-2020-10487
```

Sau khi clone công cụ khai thác trên Github, ta thực hiện đọc file (`read`) và thực thi mã (`eval`). Với mục tiêu `10.49.157.12` của bạn, cấu trúc lệnh sẽ như sau:![[Screenshot 2026-01-28 at 11.12.22.png]]
Kết quả cho thấy khai thác thành công lỗ hổng **Ghostcat** và trích xuất được thông tin cực kỳ giá trị từ file `web.xml`. Ta thu được username:password trong phần kết quả: `skyfuck:8730281lkjlkjdqlksalks`. 
Bây giờ chúng ta thực hiện thử truy cập thông qua ssh bằng câu lệnh dưới đây:
```bash
ssh skyfuck@10.49.157.12
```
![[Screenshot 2026-01-28 at 11.18.26.png]]
![[Screenshot 2026-01-28 at 11.19.47.png]]
Kiểm tra các file khoá bí mật PGP, hầu hết các khóa bí mật PGP đều được bảo vệ bằng một mật khẩu. Bạn không thể sử dụng nó nếu không có mật khẩu này. Bạn có thể dùng công cụ **John the Ripper** để bẻ khóa, ta lưu file ra máy của mình sau đó bẻ khoá bằng wordlist:
![[Screenshot 2026-01-28 at 11.27.24.png]]
Bây giờ bạn đã có đầy đủ mọi thứ để lấy được thông tin đăng nhập của user tiếp theo. Hãy quay lại cửa sổ SSH của user **skyfuck** trên máy mục tiêu và thực hiện các bước cuối cùng này:
1. Nhập khóa (Import) vào keyring
Đầu tiên, bạn cần đưa cái chìa khóa `tryhackme.asc` vào hệ thống quản lý khóa của GPG:
```bash
gpg --import tryhackme.asc
```
2. Giải mã file `credential.pgp`
Bây giờ, hãy dùng chìa khóa đã nhập để mở file chứa thông tin đăng nhập:
```bash
gpg --decrypt credential.pgp
```
 3. Nhập Passphrase
Khi hệ thống hiện bảng hỏi mật khẩu (hoặc yêu cầu nhập ở terminal), hãy nhập: **`alexandru`**
![[Screenshot 2026-01-28 at 11.32.25.png]]
Ta đã giải mã thành công nội dung của tệp `credential.pgp`.
Thông tin đăng nhập mới của chúng ta là:
- **Username:** `merlin`
- **Password:** `asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`
Bây giờ bạn đang ở user `skyfuck`, hãy thực hiện chuyển đổi sang user `merlin`![[Screenshot 2026-01-28 at 11.34.55.png]]
Đầu tiên, ta di chuyển về thư mục gốc, rồi kiểm tra ta thu được flag user đầu tiên
![[Screenshot 2026-01-28 at 11.40.31.png]]
Thực hiện kiểm tra quyền hạn của user merlin bằng 
```bash
merlin@ubuntu:~$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
merlin@ubuntu:~$ id 
uid=1000(merlin) gid=1000(merlin) groups=1000(merlin),4(adm),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare)
```
Kết quả cho thấy ta vừa tìm thấy con đường "tà đạo" nhanh nhất để lên ngôi vị cao nhất của hệ thống này. Việc `/usr/bin/zip` được phép chạy dưới quyền **root** mà không cần mật khẩu (`NOPASSWD`) là một lỗi cấu hình kinh điển trong thế giới Linux Privilege Escalation.
Vì  có quyền `sudo zip`, ta có thể nén bất kỳ file nào trên hệ thống (kể cả file của Root) vào một file tạm, sau đó xem nội dung của nó, đầu tiên nén file flag của Root vào thư mục `/tmp`, sau đó dùng `unzip` với tham số `-p` (pipe) để đọc nội dung file mà không cần giải nén ra đĩa
![[Screenshot 2026-01-28 at 11.50.38.png]]
