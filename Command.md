# Cheat Sheet 

## 1. Trinh Sát & Kiểm Tra Hệ Thống (Reconnaissance)

*   **Kiểm tra các file có quyền SUID (Set Owner User ID):**
    ```bash
    find / -perm -4000 -type f 2>/dev/null
    ```
*   **Kiểm tra các tiến trình đang chạy ẩn nội bộ của quyền root:**
    ```bash
    ps aux | grep root
    ```
*   **Kiểm tra các cổng đang lắng nghe (Dùng `netstat`):**
    ```bash
    netstat -tulnp
    ```
*   **Kiểm tra các cổng đang lắng nghe (Dùng `ss` nếu không có `netstat`):**
    ```bash
    ss -tulnp
    ```

## 2. Kết Nối Từ Xa & Chuyển Tiếp Cổng (Port Forwarding)

*   **Chuyển cổng (Local Port Forwarding) khi dùng SSH:**
    ```bash
    ssh -L 2121:127.0.0.1:21 sshuser@154.57.164.72 -p 31873
    ```
*   **Kết nối từ xa trên Windows:** Sử dụng công cụ mặc định `Remote Desktop Connection` (RDP).
*   **Kết nối từ xa từ Linux sang Windows (Dùng `xfreerdp`):**
    ```bash
    xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>
    ```

## 3. Xử Lý Wordlist & Tạo Ký Tự (Wordlist Manipulation)

*   **Lọc mật khẩu theo Regex (Độ dài >= 10, chứa chữ hoa, chữ thường và số):**
    ```bash
    awk 'length(\$0) >= 10 && /[A-Z]/ && /[a-z]/ && /[0-9]/' rockyou.txt > pass_da_loc.txt
    ```
*   **Lệnh tạo chuỗi số tăng dần (Cờ `-w` để đảm bảo độ dài ký tự bằng nhau):**
    ```bash
    seq -w 1 9999
    ```

## 4. Tấn Công Dò Thám (Brute Force)

*   **Brute force mã OTP bằng `ffuf`:**
    ```bash
    ffuf -w <FILE_NAME> -u <TARGET_HOST> -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "session=<SESSION_COOKIE>" -d "otp=FUZZ" -fr "Invalid 2FA Code"
    ```

## 5. Tài Nguyên & Danh Sách Hữu Ích (Wordlists & Datasets)

*   **Danh sách Username phổ biến:** [SecLists Usernames](https://github.com)
*   **Danh sách thông tin đăng nhập mặc định:** [SecLists Default Credentials](https://github.com)
*   **Tài khoản mặc định của hệ thống điều khiển công nghiệp (ICS/SCADA):** [SCADAPASS](https://github.com)
*   **Dữ liệu tên các thành phố trên thế giới:** [World Cities Dataset](https://github.com)

echo -n ... -> cờ -n sẽ bó ký tự xuống dòng 
base64 -w 0 -> ép các ký tự phải ở trên  một dòng duy nhất
tr -d ' -' -> Xóa toàn bộ dấu cách và dấu -
