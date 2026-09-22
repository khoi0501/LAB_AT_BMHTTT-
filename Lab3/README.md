LAB 3 : NHẬN DIỆN VÀ ỨNG PHÓ CÁC MỐI ĐE DỌA ĐẾN AN TOÀN THÔNG TIN

## 1. Thông tin sinh viên

- Họ và tên: Phạm Ngọc Anh Khôi
- MSSV: 1150070023
- Lớp: 11_TMĐT
- Tên lab: LAB 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin
- Link Youtube: https://www.youtube.com/watch?v=5hDv9yIkHvo
## 2. Phiên bản môi trường thực hành

- VMware Workstation Pro 26H1
- Windows Server 2025 Datacenter Evaluation

## 3. Cách dựng môi trường

- Tạo máy ảo Windows Server 2025 trên VMware Workstation.
- Tạo snapshot ban đầu với tên LAB3_CLEAN.
- Tạo các thư mục:
  - C:\LAB3\Evidence
  - C:\LAB3\Tools
  - C:\LAB3\Downloads
  - C:\LAB3\Assets
- Kiểm tra Windows Defender và Windows Firewall.
- Cài đặt Sysmon 15.22, Autoruns 14.3 và Process Explorer 17.14.
- Cài đặt Wireshark 4.6.8 và Npcap 1.89.
- Thiết lập Audit Policy để ghi nhận sự kiện đăng nhập.
- Tạo tài khoản lab3user để thực hiện các bài kiểm tra xác thực.

## 4. Các tình huống đã thực hiện

### TH1 – Baseline
- Kiểm tra thông tin hệ điều hành.
- Kiểm tra Windows Defender.
- Kiểm tra Windows Firewall.
- Kiểm tra cấu hình mạng.
- Ghi nhận danh sách tiến trình đang chạy.

### TH2 – Kiểm tra Malware
- Sử dụng file kiểm thử EICAR.
- Kiểm tra Windows Defender phát hiện và cách ly file.
- Lưu bằng chứng phát hiện vào thư mục Evidence.

### TH3 – Authentication và Password
- Bật Audit Policy cho Logon.
- Tạo tài khoản lab3user.
- Thực hiện đăng nhập sai và đăng nhập đúng.
- Kiểm tra các Event ID 4624, 4625 và 4648.
- Thực hiện đổi mật khẩu và kiểm tra lại log.

### TH4 – Persistence và Process
- Kiểm tra Sysmon.
- Kiểm tra Registry Run Key.
- Kiểm tra Scheduled Task.
- Tạo listener localhost 127.0.0.1:8080 và kiểm tra trạng thái Listen.
- Kiểm tra tiến trình sở hữu port bằng PowerShell và Process Explorer.

## 5. Kết quả

- TH1: Đã thực hiện
- TH2: Đã thực hiện
- TH3: Đã thực hiện
- TH4: Đã thực hiện
- TH5: Chưa thực hiện được
- TH6: Chưa thực hiện được  
- TH7: Chưa thực hiện được

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1 – Audit Policy
Lệnh auditpol sử dụng GUID trong tài liệu không chạy được trên Windows Server.

Cách khắc phục: sử dụng tên Subcategory trực tiếp:

auditpol /set /subcategory:"Logon" /success:enable /failure:enable

### Lỗi 2 – Đăng nhập tài khoản lab3user
Ban đầu tài khoản lab3user không đăng nhập được bằng runas.

Cách khắc phục: thêm tài khoản lab3user vào nhóm Users của máy:

Add-LocalGroupMember -Group "Users" -Member "lab3user"

Sau đó tài khoản có thể đăng nhập từ màn hình Windows.
