WMI : Kho dữ liệu về các phần cứng , phần mềm, các tiến trình và dịch vụ đang chạy 
Get-WmiObject : là lệnh truy vấn trên powershell để lấy dữ liệy từ kho WMI 
Get-WmiObject -Class win32_OperatingSystem | select Version,BuildNumber : Lớp chứa thông tin về hệ điều hành đang cài trên máy -> lấy version và số bản dựng của bản cập nhật
