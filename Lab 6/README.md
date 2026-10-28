# LAB 6 - Xác thực Active Directory và chính sách truy cập trên pfSense

## 1. Thông tin sinh viên

- Họ và tên : Phạm Ngọc Anh Khôi
- MSSV: 1150070023
- Lớp: 11_TMĐT
- Bài Lab 6: Xác thực Active Directory và chính sách truy cập trên pfSense
Link youtube: https://www.youtube.com/watch?v=aLcafOY4Gqs

## 2. Phiên bản môi trường

- **Thành phần:** VMware Workstation (không dùng VirtualBox như đề)
- **Thành phần:** pfSense CE 2.7.2-RELEASE (amd64), 2 GB RAM, 2 CPU, 4 card mạng
- **Thành phần:** Windows Server 2025 Datacenter Evaluation, domain `lab6.test`, AD DS + DNS + AD CS
- **Thành phần:** Kali Linux, card `eth0`, IP 10.6.0.21/24
- **Thành phần:** Kali Linux (máy thứ hai), card `eth0`, IP 10.6.0.22/24
- **Thành phần:** Ubuntu Server, nginx 1.28.3, card `ens33`

## 3. Cách dựng môi trường

### 3.1. Sơ đồ mạng

- Card pfSense: Adapter 1; Kiểu mạng VMware: NAT; Vai trò: WAN; IP pfSense: DHCP
- Card pfSense: Adapter 2; Kiểu mạng VMware: LAN Segment `LAN`; Vai trò: LAN; IP pfSense: 10.6.0.1/24
- Card pfSense: Adapter 3; Kiểu mạng VMware: LAN Segment `dmz-net`; Vai trò: DMZ; IP pfSense: 172.16.6.1/24
- Card pfSense: Adapter 4; Kiểu mạng VMware: Host-only; Vai trò: MGMT (quản trị); IP pfSense: -

### 3.2. Các máy trong lab

- Máy: DC01; Mạng ảo: LAN Segment `LAN`; IP: 10.6.0.10/24, GW 10.6.0.1, DNS 10.6.0.10; Ghi chú: AD DS, DNS, CA `LAB6-CA`
- Máy: Client-A (Kali); Mạng ảo: LAN Segment `LAN`; IP: 10.6.0.21/24, GW 10.6.0.1; Ghi chú: tài khoản thử `sv_a`
- Máy: Client-B (Kali 2); Mạng ảo: LAN Segment `LAN`; IP: 10.6.0.22/24, GW 10.6.0.1; Ghi chú: tài khoản thử `sv_b`
- Máy: DMZ-WEB (Ubuntu); Mạng ảo: LAN Segment `dmz-net`; IP: 172.16.6.10, .20, .30, .40 /24, GW 172.16.6.1; Ghi chú: nginx, 4 IP mô phỏng 4 nhóm site

### 3.3. Các bước dựng

1. **pfSense:** tạo VM 4 card mạng như bảng 3.1, gán interface WAN, LAN, DMZ, MGMT, đặt IP cho LAN và DMZ, tắt DHCP server.
2. **DC01:** đặt IP tĩnh, cài AD DS + DNS, tạo forest `lab6.test`, tạo OU `LAB6-Users` có user `sv_a`, `sv_b` và service account `svc_pfsense_ldap` (không cấp Domain Admin).
3. **DMZ-WEB:** gắn `dmz-net`, đặt 4 IP bằng `ip addr add`, route mặc định về 172.16.6.1. Cài nginx bằng cách tạm đổi card sang NAT, cài xong đổi lại `dmz-net` và đặt lại IP.
4. **Client-A, Client-B:** gắn LAN Segment `LAN`, tắt NetworkManager cho `eth0`, đặt IP tĩnh và route mặc định về 10.6.0.1.
5. **LDAP:** System > User Manager > Authentication Servers, tạo `LAB6-AD` (LDAPS cổng 636, SSL/TLS, Peer CA `LAB6-CA`, Base DN `DC=lab6,DC=test`, container `OU=LAB6-Users,DC=lab6,DC=test`, Bind DN `CN=svc_pfsense_ldap,OU=LAB6-Users,DC=lab6,DC=test`).
6. **Alias:** `CLIENT_A`, `CLIENT_B`, `POLICY_CLIENTS`, `GENERAL_SITE` (.10), `JOB_SITE` (.20), `SOCIAL_SITE` (.30), `RESTRICTED_SITE` (.40), `WEB_PORTS` (80, 443).
7. **Schedule:** `LUNCH_WINDOW` (T2-T6, 11:30-13:00), `WORK_HOURS` (T2-T6, 08:00-17:00), `TEST_NOW` (khoảng ngắn để test nhanh).
8. **Rule:** tạo theo từng tình huống ở mục 4 (rule LAN và rule DMZ), nhớ Apply Changes và Reset States sau mỗi lần sửa.

## 4. Các tình huống đã thực hiện và kết quả

- STT: 0; Tình huống: LDAP `sv_a`, `sv_b`; Cách thực hiện: LAB6-AD qua LDAPS 636; Cách kiểm tra: Diagnostics > Authentication báo authenticated successfully với cả hai user; Kết quả: **PASS**
- STT: 0; Tình huống: Captive Portal; Cách thực hiện: Zone `LAB6_USERS`, interface LAN, backend `LAB6-AD`; Cách kiểm tra: Login từ Client-A / Client-B; Kết quả: **CHƯA CÓ MINH CHỨNG** (cập nhật sau khi làm xong)
- STT: TH1; Tình huống: Client-A bị chặn JOB_SITE; Cách thực hiện: Block `CLIENT_A -> JOB_SITE` TCP/80 (log) trên Pass `CLIENT_A -> GENERAL_SITE`; Cách kiểm tra: `curl` .10 ra 200 OK, `curl` .20 timeout; Firewall Log có block 10.6.0.21 -> 172.16.6.20:80; State Table có 10.6.0.21 -> .10:80; Kết quả: **PASS**
- STT: TH2; Tình huống: Cấm RESTRICTED_SITE cho A và B; Cách thực hiện: Block `POLICY_CLIENTS -> RESTRICTED_SITE` TCP/80 (log), Pass `POLICY_CLIENTS -> GENERAL_SITE`; Cách kiểm tra: Test lần lượt 10.6.0.21 và 10.6.0.22: .10 được, .40 timeout; log có block cả hai IP; Kết quả: **PASS**
- STT: TH3; Tình huống: Client-B chỉ vào SOCIAL_SITE trong giờ nghỉ; Cách thực hiện: Pass `CLIENT_B -> SOCIAL_SITE` gắn schedule, Block cùng đối tượng nằm dưới; Cách kiểm tra: Schedule active: .30 ra 200 OK. Hết schedule: .30 timeout, log block 10.6.0.22 -> .30. Sau đó gắn lại `LUNCH_WINDOW`; Kết quả: **PASS**
- STT: TH4; Tình huống: Chỉ cho HTTP/HTTPS trong giờ hành chính; Cách thực hiện: Pass `POLICY_CLIENTS -> Any` port `WEB_PORTS` (schedule `WORK_HOURS`), Block `POLICY_CLIENTS -> Any`; Cách kiểm tra: Trong giờ: curl web được, ping 8.8.8.8 mất 100% gói. Ngoài giờ (giả lập bằng `TEST_NOW`): curl HTTP và HTTPS đều timeout; Kết quả: **PASS**
- STT: TH5; Tình huống: Cô lập DMZ khỏi LAN; Cách thực hiện: Tab DMZ: Block `DMZ subnets -> LAN subnets` (log), Allow `DMZ subnets -> Any` bên dưới; Cách kiểm tra: Client-A vào .10 được; DMZ-WEB ping 10.6.0.10 phải bị chặn; Kết quả: **RULE ĐÃ TẠO, CHƯA CÓ ẢNH TEST DMZ -> LAN** (cập nhật PASS sau khi test và chụp)
- STT: TH6; Tình huống: Block và Reject; Cách thực hiện: -; Cách kiểm tra: -; Kết quả: **KHÔNG THỰC HIỆN** (theo yêu cầu)
- STT: TH7; Tình huống: Đọc Firewall Log, State Table; Cách thực hiện: -; Cách kiểm tra: Log và State đã chụp trong TH1 - TH3; Kết quả: **KHÔNG LÀM RIÊNG** (theo yêu cầu)

## 5. Lỗi gặp phải và cách khắc phục

- STT: 1; Lỗi: LDAP test báo `Authentication failed`; Nguyên nhân: Nhớ sai mật khẩu `sv_a` (kiểm tra bằng `ValidateCredentials` trả về False); Cách khắc phục: Reset mật khẩu `sv_a`, `sv_b` bằng `Set-ADAccountPassword`, kiểm tra lại ra True
- STT: 2; Lỗi: `Select a container` báo `Could not connect to the LDAP server`, ping DC01 bị timeout; Nguyên nhân: Windows Firewall trên DC01 chặn ICMP và TCP 389 từ pfSense; Cách khắc phục: Tạo rule inbound cho phép 389 và ICMP từ 10.6.0.1 (sau đó thêm 636), Test Port thành công
- STT: 3; Lỗi: Port 389 thông nhưng vẫn không bind được, báo `A more secure authentication method is required`; Nguyên nhân: Windows Server 2025 không cho simple bind không mã hóa qua 389; hạ `LDAPServerIntegrity` về 1 không có tác dụng; Cách khắc phục: Chuyển sang LDAPS: cài AD CS, tạo CA `LAB6-CA`, import CA vào pfSense, thêm Host Override cho `WIN-AT1G9RQS8C9.lab6.test`, đổi LAB6-AD sang cổng 636, SSL/TLS
- STT: 4; Lỗi: Bind DN theo đề không tồn tại; Nguyên nhân: Service account nằm trong `OU=LAB6-Users` chứ không phải `CN=Users`; Cách khắc phục: Dùng Bind DN `CN=svc_pfsense_ldap,OU=LAB6-Users,DC=lab6,DC=test`
- STT: 5; Lỗi: `gpupdate /force` báo không tìm thấy domain controller; Nguyên nhân: Vừa restart NTDS/DNS nên bản ghi DNS chưa đăng ký lại; Cách khắc phục: `ipconfig /registerdns`, restart Netlogon, kiểm tra `nltest /dsgetdc:lab6.test`, chạy lại `gpupdate`
- STT: 6; Lỗi: `certutil -ca.cert file.cer` báo too many arguments; Nguyên nhân: PowerShell hiểu sai tham số có dấu chấm; Cách khắc phục: Viết `certutil "-ca.cert" C:\lab6-ca.cer`
- STT: 7; Lỗi: Tạo rule báo `CLIENT_A is not a valid source IP address or alias`; Nguyên nhân: Alias chưa tạo hoặc chưa Apply Changes; Cách khắc phục: Tạo alias, Apply Changes rồi mới tạo rule
- STT: 8; Lỗi: Client-A không vào được cả 172.16.6.10; Nguyên nhân: Rule cũ của LAB 5 còn trên tab LAN (`Block LAN to Any`, `Allow DC to Any`) và `Default allow` đang bị tắt; Cách khắc phục: Xóa hai rule cũ, bật lại `Default allow LAN to any rule`
- STT: 9; Lỗi: Ubuntu ping 172.16.6.1 báo `Destination Host Unreachable`; Nguyên nhân: Card Ubuntu chưa nằm cùng segment `dmz-net` với cổng DMZ của pfSense; Cách khắc phục: Đổi Network Adapter của Ubuntu sang LAN Segment `dmz-net`
- STT: 10; Lỗi: Ubuntu báo `Network is unreachable`, mất 4 IP DMZ; Nguyên nhân: Sau khi đổi card mạng, cấu hình `ip addr` (chỉ lưu trong bộ nhớ) bị mất; Cách khắc phục: Đặt lại 4 IP và route mặc định bằng `ip addr add`, `ip route add`
- STT: 11; Lỗi: `nginx.service could not be found`, mạng DMZ không ra Internet; Nguyên nhân: Chưa cài nginx; DMZ không có đường ra Internet; Cách khắc phục: Tạm đổi card sang NAT, `networkctl reconfigure ens33` (máy không có `dhclient`), `apt install nginx`, rồi đổi lại `dmz-net` và đặt lại IP
- STT: 12; Lỗi: Kali ping 10.6.0.1 báo `Destination Host Unreachable`, `ip addr flush` không xóa IP cũ `172.16.0.2/16`; Nguyên nhân: Card Kali chưa vào segment `LAN`; NetworkManager tự gán lại IP và default route cũ; Cách khắc phục: Đổi sang LAN Segment `LAN`, `nmcli device set eth0 managed no`, xóa IP và route cũ (`ip addr del`, `ip route del default via 172.16.0.1`)
- STT: 13; Lỗi: Dán nhiều lệnh `sudo` cùng lúc thì lệnh `flush` không chạy; Nguyên nhân: `sudo` hỏi mật khẩu giữa chừng nên nuốt lệnh; Cách khắc phục: Nhập mật khẩu trước, chạy từng lệnh
- STT: 14; Lỗi: State Table lọc `172.16.6.10` báo trống; Nguyên nhân: State kết nối đã hết hạn; Cách khắc phục: Chạy lại `curl` rồi bấm Filter ngay
- STT: 15; Lỗi: Rule gắn schedule hết hiệu lực nhưng traffic vẫn đi; Nguyên nhân: Schedule không active thì rule coi như không có, traffic rơi xuống rule dưới; Cách khắc phục: Thêm rule Block nằm dưới rule Pass có schedule (TH3, TH4)
