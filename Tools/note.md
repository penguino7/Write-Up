# Tools — Notes

## 📚 Mục lục

| Tool | Mô tả | Giai đoạn |
|------|-------|-----------|
| [EyeWitness](#eyewitness) | Chụp ảnh màn hình hàng loạt web service, nhận diện default credential | Reconnaissance |
| [Aquatone](#aquatone) | Chụp ảnh và tổng hợp visual report các web host từ nhiều nguồn đầu vào | Reconnaissance |
| [Nmap](#nmap) | Network scanner — phát hiện host, port, service, OS, chạy script | Reconnaissance / Enumeration |

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
