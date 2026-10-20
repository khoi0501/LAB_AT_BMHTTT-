# LAB 7 - Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy

## 1. Thông tin sinh viên

* Họ và tên: Phạm Ngọc Anh Khôi
* MSSV: 1150070023
* Lớp: 11_TMĐT
* Bài Lab 7: Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy
* Link YouTube: https://www.youtube.com/watch?v=FF0nRf_IinI

## 2. Phiên bản môi trường

* Máy thật chạy VMware Workstation, các máy ảo bên dưới
* pfSense CE (firewall/gateway), LAN 10.6.0.1, DMZ 172.16.6.1
* DC01: Windows Server 2025 Datacenter Evaluation (AD/DNS), IP 10.6.0.10
* DMZ-WEB: Ubuntu + Nginx (website mô phỏng *.lab7.test), IP 172.16.6.10
* Client-A: Kali Linux, IP 10.6.0.21
* Client-B: Kali Linux 2, IP 10.6.0.22
* WEB-PROXY: Ubuntu 26.04.1 LTS (clone từ Ubuntu), IP 10.6.0.30
* Phần mềm trên WEB-PROXY: Squid 7.2-2ubuntu2.2 (nghe localhost 3128), E2Guardian 5.5.9-2 (nghe 10.6.0.30:8080), ClamAV daemon 1.5.4
* Đề ghi Ubuntu 24.04 nhưng máy em dùng 26.04.1, vẫn chạy bình thường

## 3. Cách dựng môi trường

1. Dùng lại các máy từ LAB 6: pfSense, DC01, Client-A, Client-B, DMZ-WEB.
2. Tạo máy WEB-PROXY (clone Ubuntu), card ens37 nối mạng lab6-lan, IP 10.6.0.30/24, gateway 10.6.0.1, DNS 10.6.0.10. Card ens33 là NAT để cài gói. Ping thử 10.6.0.1, 10.6.0.10, 172.16.6.1 và 8.8.8.8 đều được.
3. Trên WEB-PROXY cài gói: `sudo apt install -y squid e2guardian clamav-daemon clamav-freshclam`.
4. Cấu hình E2Guardian nghe cổng 8080, chuyển tiếp sang Squid (`proxyip = 127.0.0.1`, `proxyport = 3128`), bật quét ClamAV.
5. Trên DC01 tạo zone DNS `lab7.test`, thêm các host (normal, education, download, china, mideast, jobs, weapons, politics, religion, streaming, entertainment, games, blockedgame) đều trỏ về 172.16.6.10.
6. Trên pfSense tạo alias: LAB7_CLIENTS (10.6.0.21, 10.6.0.22), LAB7_PROXY (10.6.0.30), WEB_PORTS (80, 443).
7. Trên pfSense tạo rule LAN L7_1 đến L7_6: cho DNS tới DC01, cho client đi tới proxy 8080, cho mail ports, chặn client đi thẳng web (L7_4), cho proxy ra web, cho proxy tới DMZ-WEB.
8. Trên Client-A/B đặt proxy trình duyệt là 10.6.0.30:8080.
9. Đưa các domain vào list của E2Guardian: hard block (china, mideast, jobs, blockedgame), warning có bypass 300 giây (weapons, politics, religion, streaming), exception (normal, education, download).

## 4. Các tình huống đã thực hiện và kết quả

* Direct web không qua proxy (normal.lab7.test, https://example.com): HTTP 000 / HTTPS 000, bị pfSense chặn bằng rule L7_4.
* Web bình thường qua proxy (normal.lab7.test): HTTP 200, truy cập được.
* Education (education.lab7.test): vào bình thường, không cảnh báo.
* Hard block Geo-Sim (china.lab7.test, mideast tương tự): hiện trang E2Guardian Access Denied, không có bypass.
* Hard block Job Search (jobs.lab7.test): hiện trang Access Denied, không có bypass.
* Warning + temporary bypass (weapons.lab7.test): hiện trang chặn có link Acknowledge, bấm vào thì xem tạm 5 phút.
* Antivirus (download.lab7.test/files/eicar.com.txt): bị chặn, log ghi INFECTED, Eicar-Signature.
* File thường 10MB.bin: HTTP 200, log ghi SCANNED.
* Giới hạn kích thước: 100MB.bin tải được (200), 110MB.bin bị 403, Squid log TCP_DENIED_REPLY.
* Chưa làm / chưa có ảnh: Schedule 60 phút cho entertainment/games và HTTPS inspection (phần mở rộng).

## 5. Lỗi gặp phải và cách khắc phục

* Log E2Guardian báo `ClamD error: Access denied`, file bị DENIED. Nguyên nhân là clamd chạy user clamav, e2guardian chạy user e2guardian nên không đọc được file tạm của nhau. Em chạy `sudo usermod -aG e2guardian clamav`, restart clamav-daemon (chờ 15 giây) rồi restart e2guardian.
* Giới hạn 110MB không hoạt động vì E2Guardian chưa chuyển qua Squid, do `proxyip` và `proxyport` trong e2guardian.conf đang bị comment (#). Em backup file, dùng sed bỏ # và đặt `proxyip = 127.0.0.1`, `proxyport = 3128`, restart e2guardian.
* File .bin bị chặn với lý do "Banned file extension". Em chỉnh lại list, khôi phục `exceptionsitelist` từ bản backup rồi restart e2guardian.
* Phiên bản Ubuntu khác đề (26.04 thay vì 24.04). Em kiểm tra bằng `apt-cache policy` thấy gói có đủ nên cài bình thường.
