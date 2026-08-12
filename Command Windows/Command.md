* **WMI**: Kho dữ liệu về các phần cứng, phần mềm, các tiến trình và dịch vụ đang chạy.
* **Get-WmiObject**: Là lệnh truy vấn trên PowerShell để lấy dữ liệu từ kho WMI.
* **Get-WmiObject -Class win32_OperatingSystem | select Version,BuildNumber**: Lớp chứa thông tin về hệ điều hành đang cài trên máy \(\rightarrow\) lấy version và số bản dựng của bản cập nhật.

---

### Thư mục gốc (`C:\`)
Còn gọi là phân vùng khởi động, là nơi cài đặt hệ điều hành.

**Cấu trúc thư mục:**

```text
C:\ (System Drive)
│
├── 📂 Perflogs ─────────────── Chứa log hiệu năng (Mặc định trống)
├── 📂 Program Files ────────── Phần mềm 64-bit (hoặc 32-bit trên OS 32-bit)
├── 📂 Program Files (x86) ──── Phần mềm 32-bit/16-bit (chỉ có trên OS 64-bit)
├── 🔒 ProgramData (Ẩn) ─────── Dữ liệu ứng dụng dùng chung cho mọi User
│
├── 📂 Users ────────────────── Thư mục chứa tài khoản người dùng
│   ├── ⚙️ Default ──────────── Hồ sơ mẫu để cấu hình khi tạo User mới
│   ├── 🔄 Public ───────────── Thư mục chia sẻ file chung giữa các User
│   └── 📂 [Tên_User]
│       └── 🔒 AppData (Ẩn) ─── Dữ liệu cấu hình riêng của từng User
│           ├── 🌐 Roaming ──── Dữ liệu đồng bộ theo Profile (từ điển, cấu hình...)
│           ├── 💻 Local ────── Dữ liệu cố định trên máy này, không đồng bộ mạng
│           └── 🛡️ LocalLow ─── Dữ liệu có mức độ tin cậy thấp (Protected Mode)
│
└── 📂 Windows ──────────────── Thư mục cốt lõi của Hệ điều hành
    ├── ⚙️ System / System32 ── Chứa file thư viện DLL lõi và Windows API
    ├── ⚙️ SysWOW64 ─────────── Chứa DLL 32-bit phục vụ chạy app trên OS 64-bit
    └── 📦 WinSxS ───────────── Kho lưu trữ cấu phần (Component Store), bản cập nhật
```

## Table of Contents

* 📂 [Thao tác với Thư mục](#thao-tác-với-thư-mục)
    * [1. Xem thư mục hiện tại](#1-xem-thư-mục-hiện-tại)
    * [2. Liệt kê nội dung thư mục](#2-liệt-kê-nội-dung-thư-mục)
    * [3. Xem cấu trúc cây thư mục](#3-xem-cấu-trúc-cây-thư-mục)
    * [4. Chuyển đổi thư mục](#4-chuyển-đổi-thư-mục)
    * [5. Tạo thư mục mới](#5-tạo-thư-mục-mới)
    * [6. Xóa thư mục](#6-xóa-thư-mục)
    * [7. Đổi tên hoặc Di chuyển thư mục](#7-đổi-tên-hoặc-di-chuyển-thư-mục)
    * [8. Sao chép thư mục](#8-sao-chép-thư-mục)
    * [9. Quản lý quyền truy cập bằng icacls](#9-quản-lý-quyền-truy-cập-bằng-icacls)
* 📄 [Thao tác với File](#thao-tác-với-file)
    * [1. Tạo file mới](#1-tạo-file-mới)
    * [2. Xem nội dung file](#2-xem-nội-dung-file)
    * [3. Sao chép file](#3-sao-chép-file)
    * [4. Di chuyển & Đổi tên file](#4-di-chuyển--đổi-tên-file)
    * [5. Xóa file](#5-xóa-file)
    * [6. Tìm kiếm trong file](#6-tìm-kiếm-trong-file)
* 🌐 [Thao tác mạng với smbclient](#thao-tác-mạng-với-smbclient)
    * [1. Giải thích về smbclient](#1-giải-thích-về-smbclient)
    * [2. Liệt kê thư mục đang chia sẻ](#2-liệt-kê-thư-mục-đang-chia-sẻ)
    * [3. Kết nối thư mục chia sẻ cụ thể](#3-kết-nối-thư-mục-chia-sẻ-cụ-thể)
    * [4. Kết nối ẩn danh (Anonymous/Guest)](#4-kết-nối-ẩn-danh-anonymousguest)
    * [5. Kết nối từ máy thuộc Domain mạng doanh nghiệp](#5-kết-nối-từ-máy-thuộc-domain-mạng-doanh-nghiệp)
    * [6. Các lệnh thao tác khi kết nối thành công](#6-các-lệnh-thao-tác-khi-kết-nối-thành-công)
* ⚙️ [Windows Registry](#windows-registry)
    * [1. Khái niệm và Vai trò](#1-khái-niệm-và-vai-trò)
    * [2. Cấu trúc Registry (Hives, Keys, Values)](#2-cấu-trúc-registry-hives-keys-values)
    * [3. Các Registry Hives chính](#3-các-registry-hives-chính)
    * [4. Các kiểu dữ liệu phổ biến](#4-các-kiểu-dữ-liệu-phổ-biến)
    * [5. Quản lý Registry với Registry Editor (regedit)](#5-quản-lý-registry-với-registry-editor-regedit)
    * [6. Bài thực hành: Tạo Key và Value cơ bản](#6-bài-thực-hành-tạo-key-và-value-cơ-bản)
---

## 📁 Thao tác với Thư mục

### 1. Xem thư mục hiện tại

| Môi trường | Lệnh |
| :--- | :--- |
| **CMD** | `cd` |
| **PowerShell** | `Get-Location` hoặc `pwd` |

---

### 2. Liệt kê nội dung thư mục

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD** | `dir` | Liệt kê cơ bản |
| **CMD** | `dir /a` | Hiển thị tất cả file/thư mục (bao gồm file ẩn) |
| **CMD** | `dir /b` | Chỉ hiển thị tên (định dạng ngắn gọn) |
| **CMD** | `dir /s` | Liệt kê đệ quy toàn bộ thư mục con |
| **PowerShell** | `Get-ChildItem` (hoặc `ls`) | Liệt kê cơ bản |
| **PowerShell** | `Get-ChildItem -Force` | Hiển thị tất cả (bao gồm file ẩn/hệ thống) |
| **PowerShell** | `Get-ChildItem -Recurse` | Liệt kê đệ quy |

---

### 3. Xem cấu trúc cây thư mục

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD** | `tree` | Hiển thị sơ đồ cây các thư mục con |
| **CMD** | `tree /F` | Hiển thị sơ đồ cây bao gồm cả **thư mục và các file** bên trong (`/F`) |
| **CMD** | `tree /A` | Sử dụng ký tự ASCII tiêu chuẩn để vẽ cây (`/A`) |
| **CMD** | `tree /F /A` | Kết hợp hiển thị cả file và vẽ bằng ký tự ASCII chuẩn |
| **PowerShell** | `tree /F` | Sử dụng lệnh `tree` của hệ thống trong PowerShell |
| **PowerShell** | `Get-ChildItem -Recurse` | Liệt kê đệ quy danh sách dạng bảng |

---

### 4. Chuyển đổi thư mục

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD / PowerShell** | `cd path\to\folder` | Chuyển đến thư mục chỉ định |
| **CMD / PowerShell** | `cd ..` | Quay lại thư mục cha (lên 1 cấp) |
| **CMD / PowerShell** | `cd \` | Quay về thư mục gốc của ổ đĩa |
| **CMD** | `cd /d D:\folder` | Chuyển sang ổ đĩa khác (cần cờ `/d`) |
| **PowerShell** | `Set-Location D:\folder` | Chuyển sang ổ đĩa/thư mục khác |

---

### 5. Tạo thư mục mới

| Môi trường | Lệnh | Mô tả |
| :--- | :--- | :--- |
| **CMD** | `mkdir TenThuMuc` | Tạo thư mục mới |
| **CMD** | `mkdir Sub1\Sub2\Sub3` | Tạo nhiều cấp thư mục lồng nhau cùng lúc |
| **PowerShell** | `New-Item -ItemType Directory -Name "TenThuMuc"` | Tạo thư mục mới |

---

### 6. Xóa thư mục

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD** | `rmdir TenThuMuc` | Chỉ xóa thư mục rỗng |
| **CMD** | `rmdir /s TenThuMuc` | Xóa đệ quy kèm thư mục con (có hỏi xác nhận) |
| **CMD** | `rmdir /s /q TenThuMuc` | Xóa đệ quy im lặng, không hỏi xác nhận (`/q`) |
| **PowerShell** | `Remove-Item TenThuMuc` | Xóa thư mục rỗng |
| **PowerShell** | `Remove-Item TenThuMuc -Recurse` | Xóa đệ quy toàn bộ thư mục con |
| **PowerShell** | `Remove-Item TenThuMuc -Recurse -Force` | Ép buộc xóa cả file ẩn/chỉ đọc (`-Force`) |

---

### 7. Đổi tên hoặc Di chuyển thư mục

| Môi trường | Lệnh | Mô tả |
| :--- | :--- | :--- |
| **CMD** | `ren TenCu TenMoi` | Đổi tên thư mục |
| **CMD** | `move ThuMucNguon ThuMucDich` | Di chuyển thư mục sang vị trí khác |
| **PowerShell** | `Rename-Item -Path "TenCu" -NewName "TenMoi"` | Đổi tên thư mục |
| **PowerShell** | `Move-Item -Path "Nguon" -Destination "Dich"` | Di chuyển thư mục |

---

### 8. Sao chép thư mục

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD** | `xcopy Nguon Dich /E /I /H` | `/E`: Sao chép cả thư mục rỗng<br>`/I`: Tự tạo thư mục đích nếu chưa có<br>`/H`: Sao chép cả file ẩn/hệ thống |
| **CMD** | `robocopy Nguon Dich /E /ZB` | `/E`: Sao chép đệ quy<br>`/ZB`: Chế độ sao chép an toàn (khôi phục khi đứt đoạn) |
| **PowerShell** | `Copy-Item Nguon Dich -Recurse` | Sao chép đệ quy toàn bộ thư mục con |

---

### 9. Quản lý quyền truy cập bằng icacls

`icacls` (Integrity Control Access Control List) là công cụ dòng lệnh mạnh mẽ trên Windows dùng để xem, sửa đổi, sao lưu và khôi phục danh sách kiểm soát truy cập (ACL) của tệp tin và thư mục.

> ⚠️ **Lưu ý:** Bạn phải chạy Command Prompt (CMD) hoặc PowerShell với quyền **Administrator** để thực thi các lệnh này.

#### Các cờ (Flags) điều khiển phổ biến
* `/grant`: Cấp quyền cho một người dùng hoặc nhóm cụ thể.
* `/deny`: Từ chối quyền một cách rõ ràng (Quyền deny luôn được ưu tiên trước quyền grant).
* `/remove`: Xóa bỏ phân quyền của một người dùng/nhóm khỏi danh sách.
* `/reset`: Đặt lại quyền về mặc định (sử dụng các quyền thừa hưởng từ thư mục cha).
* `/t`: Áp dụng lệnh cho thư mục hiện tại và tất cả các thư mục/tệp tin con bên trong nó (Đệ quy).
* `/c`: Tiếp tục thực hiện lệnh ngay cả khi gặp lỗi tệp tin (ví dụ: lỗi Access Denied ở một file lẻ).

#### Các ký hiệu quyền hạn (Permissions)
* `F` (Full Control): Toàn quyền (Đọc, ghi, xóa, sửa và thay đổi phân quyền).
* `M` (Modify): Quyền sửa đổi, xóa tệp nhưng không được thay đổi cấu hình bảo mật.
* `RX` (Read and Execute): Chỉ được đọc và chạy các tệp thực thi.
* `R` (Read): Chỉ được phép xem nội dung.
* `W` (Write): Chỉ được phép ghi hoặc thay đổi nội dung bên trong.

#### Các lệnh mẫu thực tế

* **Xem phân quyền hiện tại của thư mục:**
  ```cmd
  icacls "C:\Thư_Mục_Của_Bạn"
  ```

* **Cấp toàn quyền (Full Control) cho một User cụ thể (ví dụ: `Administrator`):**
  ```cmd
  icacls "C:\Data" /grant Administrator:F
  ```

* **Cấp quyền Đọc và Ghi cho User áp dụng cho cả các thư mục con bên trong (`/t`):**
  ```cmd
  icacls "C:\Data" /grant Username:(R,W) /t
  ```

* **Chặn quyền truy cập (Deny) của một User:**
  ```cmd
  icacls "C:\Data" /deny BadUser:F
  ```

* **Xóa bỏ cấu hình phân quyền riêng của một User (Trả lại trạng thái như chưa từng phân quyền):**
  ```cmd
  icacls "C:\Data" /remove Username
  ```

* **Khôi phục thư mục về quyền mặc định (Thừa hưởng từ thư mục cha):**
  ```cmd
  icacls "C:\Data" /reset /t
  ```

* **Sao lưu toàn bộ cấu hình quyền của thư mục ra một file text:**
  ```cmd
  icacls "C:\Data" /save "C:\Backup\acl_backup.txt" /t
  ```

* **Khôi phục lại quyền từ file sao lưu trước đó:**
  ```cmd
  icacls "C:\Data" /restore "C:\Backup\acl_backup.txt"
  ```

#### Các cài đặt Kế thừa quyền (Inheritance Flags)

Khi bạn xem quyền bằng lệnh `icacls`, các ký hiệu này sẽ xuất hiện ngay sau tên người dùng (ví dụ: `Administrator:(I)(F)` hoặc `User:(OI)(CI)(IO)(M)`) để chỉ định cách quyền hạn lan truyền từ thư mục cha xuống cấp con.

* **`(I)` - Permission inherited from parent container (Quyền được kế thừa từ cha):**
  * **Ý nghĩa:** Quyền này không phải do bạn đặt trực tiếp tại thư mục này, mà nó tự động "thừa hưởng" từ thư mục cha cấp cao hơn truyền xuống.
* **`(OI)` - Object inherit (Kế thừa đối tượng):**
  * **Ý nghĩa:** Các **tệp tin (file)** được tạo ra bên trong thư mục này sẽ tự động nhận quyền từ thư mục cha. Nó không áp dụng cho thư mục con.
* **`(CI)` - Container inherit (Kế thừa vùng chứa):**
  * **Ý nghĩa:** Các **thư mục con (sub-folder)** được tạo ra bên trong thư mục này sẽ tự động nhận quyền từ thư mục cha. Nó không áp dụng cho tệp tin.
* **`(IO)` - Inherit only (Chỉ kế thừa):**
  * **Ý nghĩa:** Quyền này **không áp dụng** cho chính thư mục hiện tại, mà nó chỉ đóng vai trò làm khuôn mẫu để truyền xuống cho các file hoặc thư mục con bên trong mà thôi.
* **`(NP)` - Do not propagate inherit (Không lan truyền kế thừa):**
  * **Ý nghĩa:** Quyền này chỉ truyền xuống **đúng 1 cấp** (cho file và thư mục con trực tiếp bên trong nó). Các thư mục cháu, chắt sâu hơn nữa sẽ không nhận được quyền này.

---

#### Các tổ hợp thường gặp trong thực tế

Để dễ hình dung, các cờ này thường đi chung với nhau thành các bộ quy tắc:

* **`(OI)(CI)`**: Cả file và thư mục con bên trong đều nhận quyền. Đây là chế độ mặc định của Windows khi bạn tạo mới một thư mục.
* **`(OI)(CI)(IO)`**: Bản thân thư mục cha không bị ảnh hưởng bởi quyền này, nhưng toàn bộ file và thư mục con bên trong nó sẽ dính quyền.
* **`(CI)(IO)`**: Chỉ áp dụng quyền cho các thư mục con bên trong, bản thân thư mục cha và các file lẻ không bị ảnh hưởng.
* **`(OI)(NP)`**: Chỉ áp dụng quyền cho thư mục cha và các file trực tiếp cấp dưới nó, các thư mục con và cấp sâu hơn bị chặn, không nhận quyền.


## 📄 Thao tác với File

### 1. Tạo file mới

| Môi trường | Lệnh | Mô tả |
| :--- | :--- | :--- |
| **CMD** | `type nul > file.txt` | Tạo file rỗng |
| **CMD** | `echo noi_dung > file.txt` | Tạo file kèm nội dung ban đầu |
| **PowerShell** | `New-Item -ItemType File -Name "file.txt"` | Tạo file rỗng |

---

### 2. Xem nội dung file

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD** | `type file.txt` | Đọc toàn bộ nội dung file |
| **PowerShell** | `Get-Content file.txt` | Đọc nội dung file |
| **PowerShell** | `Get-Content file.txt -Head 10` | Chỉ đọc 10 dòng đầu tiên (`-Head`) |
| **PowerShell** | `Get-Content file.txt -Tail 10` | Chỉ đọc 10 dòng cuối cùng (`-Tail`) |

---

### 3. Sao chép file

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD** | `copy file1.txt file2.txt` | Sao chép file sang file mới |
| **CMD** | `copy *.txt D:\Backup\` | Sao chép toàn bộ file `.txt` sang thư mục khác |
| **CMD** | `copy /Y file1.txt file2.txt` | Sao chép và ghi đè không hỏi lại (`/Y`) |
| **PowerShell** | `Copy-Item file1.txt file2.txt` | Sao chép file |
| **PowerShell** | `Copy-Item file1.txt file2.txt -Force` | Ép buộc ghi đè nếu file đã tồn tại |

---

### 4. Di chuyển & Đổi tên file

| Môi trường | Lệnh | Mô tả |
| :--- | :--- | :--- |
| **CMD** | `ren file_cu.txt file_moi.txt` | Đổi tên file |
| **CMD** | `move file.txt D:\Destination\` | Di chuyển file sang thư mục mới |
| **PowerShell** | `Rename-Item file_cu.txt file_moi.txt` | Đổi tên file |
| **PowerShell** | `Move-Item file.txt D:\Destination\` | Di chuyển file |

---

### 5. Xóa file

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD** | `del file.txt` | Xóa file chỉ định |
| **CMD** | `del /f file.txt` | Ép buộc xóa file chỉ đọc (Read-only) (`/f`) |
| **CMD** | `del /q *.log` | Xóa tất cả file `.log` không hỏi xác nhận (`/q`) |
| **PowerShell** | `Remove-Item file.txt` | Xóa file |
| **PowerShell** | `Remove-Item file.txt -Force` | Ép buộc xóa file ẩn/chỉ đọc |

---

### 6. Tìm kiếm trong file

| Môi trường | Lệnh | Mô tả / Cờ (Flag) |
| :--- | :--- | :--- |
| **CMD** | `findstr "chuoi" file.txt` | Tìm dòng chứa từ khóa |
| **CMD** | `findstr /i "chuoi" file.txt` | Tìm kiếm không phân biệt hoa/thường (`/i`) |
| **CMD** | `findstr /s /i "chuoi" *.txt` | Tìm đệ quy trong tất cả thư mục con (`/s`) |
| **PowerShell** | `Select-String -Path file.txt -Pattern "chuoi"` | Tìm dòng chứa từ khóa |

## 🌐 Thao tác mạng với smbclient

### 1. Giải thích về smbclient
`smbclient` là công cụ dòng lệnh trên Linux/Unix, hoạt động tương tự FTP client nhưng dành riêng cho giao thức **SMB/CIFS** (mặc định của Windows và Samba server). Nó giúp máy Linux kết nối, quản lý, tải lên hoặc tải xuống các file nằm trên thư mục chia sẻ từ xa.

### 2. Liệt kê thư mục đang chia sẻ
Kiểm tra xem máy chủ mục tiêu đang chia sẻ những thư mục nào.
* **Cú pháp:** `smbclient -L //<IP_hoặc_Tên_Máy_Chủ> -U <username>`
* **Ví dụ:** `smbclient -L //192.168.1.50 -U lananh`

### 3. Kết nối thư mục chia sẻ cụ thể
Truy cập trực tiếp vào thư mục đích để bắt đầu quản lý file.
* **Cú pháp:** `smbclient //<IP_hoặc_Tên_Máy_Chủ>/<Tên_Thư_Mục> -U <username>`
* **Ví dụ:** `smbclient //192.168.1.50/TaiLieu -U lananh`

### 4. Kết nối ẩn danh (Anonymous/Guest)
Sử dụng khi thư mục từ xa được cấu hình chia sẻ công khai không mật khẩu.
* **Cú pháp:** `smbclient //<IP_hoặc_Tên_Máy_Chủ>/<Tên_Thư_Mục> -N`
* **Ví dụ:** `smbclient //192.168.1.50/PublicData -N`

### 5. Kết nối từ máy thuộc Domain mạng doanh nghiệp
Sử dụng khi tài khoản thuộc một vùng quản lý Domain cụ thể trong công ty.
* **Cú pháp:** `smbclient //<IP_hoặc_Tên_Máy_Chủ>/<Tên_Thư_Mục> -U <username> -W <DOMAIN_NAME>`
* **Ví dụ:** `smbclient //192.168.1.50/KếToán -U lananh -W COMPANY_DOMAIN`

### 6. Các lệnh thao tác khi kết nối thành công
Sau khi kết nối thành công, giao diện sẽ đổi thành `smb: \>`. Bạn dùng các lệnh sau:

| Câu lệnh | Chức năng chi tiết | Ví dụ thực tế |
| :--- | :--- | :--- |
| **`ls`** | Liệt kê file/thư mục trên máy chủ | `smb: \> ls` |
| **`cd`** | Di chuyển giữa các thư mục từ xa | `smb: \> cd ThuMucCon` |
| **`lcd`** | Thay đổi thư mục hiện hành trên máy Linux local | `smb: \> lcd /home/user/Downloads` |
| **`get`** | Tải file từ máy chủ từ xa về máy Linux | `smb: \> get BaoCao.xlsx` |
| **`put`** | Tải file từ máy Linux lên máy chủ từ xa | `smb: \> put DuAn.zip` |
| **`mkdir`** | Tạo thư mục mới trên máy chủ từ xa | `smb: \> mkdir Backup` |
| **`rm`** | Xóa file trên máy chủ từ xa | `smb: \> rm ThongTinCu.txt` |
| **`exit`** | Thoát khỏi giao diện smbclient | `smb: \> exit` |

## ⚙️ Windows Registry

### 1. Khái niệm & Vai trò
**Windows Registry** là cơ sở dữ liệu phân cấp trung tâm được Windows và hầu hết các ứng dụng lưu trữ thông tin cấu hình. Registry chứa toàn bộ các thiết lập liên quan đến:
* Hồ sơ người dùng (User Profiles)
* Cấu hình phần mềm, phần cứng
* Các dịch vụ hệ thống (Services)
* Chính sách bảo mật (Security Policies)
* Các thiết lập tùy chỉnh của hệ điều hành

> ⚠️ **Lưu ý quan trọng:** Bất kỳ sai sót nào khi chỉnh sửa Registry đều có thể khiến ứng dụng hoặc các thành phần của Windows bị lỗi. Bạn nên luôn kiểm tra trên môi trường Lab/máy ảo hoặc sao lưu trước khi thực hiện thay đổi.

---

### 2. Cấu trúc Registry (Hives, Keys, Values)
Registry được tổ chức theo mô hình cây phân cấp tương tự như thư mục và tệp tin trong hệ thống tệp Windows:

| Thành phần | Mô tả | Tương đương trong File System |
| :--- | :--- | :--- |
| **Hive** | Phần cao nhất của Registry, chứa nhóm các thiết lập hệ thống hoặc user. | Ổ đĩa (`C:\`, `D:\`) |
| **Key** | Thư mục chứa bên trong Hive. Một Key có thể chứa các Key con hoặc Value. | Thư mục (Folder) |
| **Subkey** | Thẻ/Key nằm bên trong một Key khác để tổ chức cấu hình. | Thư mục con (Subfolder) |
| **Value** | Giá trị cấu hình cụ thể nằm trong Key. | Tập tin (File) |

#### Một Registry Value bao gồm 3 thành phần chính:
1. **Name:** Tên định danh thiết lập.
2. **Type:** Kiểu dữ liệu lưu trữ.
3. **Data:** Giá trị dữ liệu cấu hình thực tế.

*Ví dụ về đường dẫn Registry:*
```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion
