# LAB 1: BẮT GÓI TIN TELNET VÀ SSH

### 1. Thông tin sinh viên
- **Họ và tên:** Nguyễn Phước Thịnh
- **MSSV:** 1150070040
- **Lớp:** 11_TMĐT
- **Link Video YouTube:** https://www.youtube.com/watch?v=RpG_ujTLglE

---

### 2. Môi trường thực hành
- **Client & Attacker (Máy thật):** Windows 11 (PuTTY 0.85, Wireshark).
- **Server (Máy ảo VMware):** Kali Linux (IP mạng NAT - VMnet8).
  - Dịch vụ: Telnet (Port 23) & SSH (Port 22).

---

### 3. Nội dung đã thực hiện
1. Cấu hình Telnet và OpenSSH trên Kali Linux, tạo tài khoản sinh viên.
2. Dùng Wireshark bắt gói Telnet (cổng 23) -> phân tích TCP Stream.
3. Thử nghiệm đổi mật khẩu phức tạp trên Telnet -> kiểm chứng vẫn bị lộ Plaintext.
4. Dùng Wireshark bắt gói SSH (cổng 22) -> kiểm tra Host-key fingerprint và phân tích gói tin mã hóa.
5. Demo đăng nhập SSH bằng cặp khóa (SSH Key qua PuTTYgen) không dùng mật khẩu.
6. Hoàn thành trả lời 11 câu hỏi trong báo cáo.

---

### 4. Kết quả đạt được
- **Telnet:** Hoàn toàn không mã hóa. Bắt trọn vẹn Username, Password và các lệnh thực thi ở dạng văn bản rõ (Plaintext).
- **SSH:** Toàn bộ payload được mã hóa bảo mật (Encrypted packet), chỉ thấy metadata; không thể đọc được nội dung thông tin hay mật khẩu.

---

### 5. Lệnh kiểm tra lại trên Server (Kali Linux)
```bash
# Kiểm tra cổng 22 và 23 đang lắng nghe
ss -ltn
```
*Vị trí bắt gói trên Wireshark (Windows): Chọn card mạng **VMware Network Adapter VMnet8**.*
