# Tools — Notes

## 📚 Mục lục

| Tool | Mô tả | Giai đoạn |
|------|-------|-----------|
| [EyeWitness](#eyewitness) | Chụp ảnh màn hình hàng loạt web service, nhận diện default credential | Reconnaissance |
| [Aquatone](#aquatone) | Chụp ảnh và tổng hợp visual report các web host từ nhiều nguồn đầu vào | Reconnaissance |

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
