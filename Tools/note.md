# Tools — Notes

## 📚 Mục lục

| Tool | Mô tả | Giai đoạn |
|------|-------|-----------|
| [EyeWitness](#eyewitness) | Chụp ảnh màn hình hàng loạt web service, nhận diện default credential | Reconnaissance |

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
