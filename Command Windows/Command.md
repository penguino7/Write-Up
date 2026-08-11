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
* [📁 Thao tác với Thư mục](#-thao-tác-với-thư-mục)
  * [1. Xem thư mục hiện tại](#1-xem-thư-mục-hiện-tại)
  * [2. Liệt kê nội dung thư mục](#2-liệt-kê-nội-dung-thư-mục)
  * [3. Xem cấu trúc cây thư mục](#3-xem-cấu-trúc-cây-thư-mục)
  * [4. Chuyển đổi thư mục](#4-chuyển-đổi-thư-mục)
  * [5. Tạo thư mục mới](#5-tạo-thư-mục-mới)
  * [6. Xóa thư mục](#6-xóa-thư-mục)
  * [7. Đổi tên hoặc Di chuyển thư mục](#7-đổi-tên-hoặc-di-chuyển-thư-mục)
  * [8. Sao chép thư mục](#8-sao-chép-thư-mục)
* [📄 Thao tác với File](#-thao-tác-với-file)
  * [1. Tạo file mới](#1-tạo-file-mới)
  * [2. Xem nội dung file](#2-xem-nội-dung-file)
  * [3. Sao chép file](#3-sao-chép-file)
  * [4. Di chuyển & Đổi tên file](#4-di-chuyển--đổi-tên-file)
  * [5. Xóa file](#5-xóa-file)
  * [6. Tìm kiếm trong file](#6-tìm-kiếm-trong-file)

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
