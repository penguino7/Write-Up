WMI : Kho dữ liệu về các phần cứng , phần mềm, các tiến trình và dịch vụ đang chạy 
Get-WmiObject : là lệnh truy vấn trên powershell để lấy dữ liệy từ kho WMI 
Get-WmiObject -Class win32_OperatingSystem | select Version,BuildNumber : Lớp chứa thông tin về hệ điều hành đang cài trên máy -> lấy version và số bản dựng của bản cập nhật
Thư mục gốc : C:\ -> Thư mục gốc , còn gọi là phân vùng khởi động , là nơi cài đặt hệ điều hành 
Cấu trúc thư mục : 
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


| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |
