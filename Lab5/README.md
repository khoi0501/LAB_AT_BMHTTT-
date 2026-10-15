# LAB 3 – Thiết lập mô hình tường lửa pfSense

## 1. Thông tin sinh viên

- **Họ tên:** Phạm Ngọc Anh Khôi
- **MSSV:** 1150070023
- **Lab:** LAB 5 – Thiết lập mô hình tường lửa pfSense
- **Video:** https://www.youtube.com/watch?v=bpNnxr-Jvdg

---

## 2. Môi trường thực hiện

- **pfSense:** pfSense CE
- **Windows Server:** Windows Server 2025 – dùng làm Domain Controller (DC)
- **Ubuntu Server:** dùng làm máy kiểm thử LAN-Test
- **Kali Linux:** dùng thay thế máy DMZ-Web để kiểm thử firewall
- **VMware:** môi trường máy ảo

### Sơ đồ địa chỉ IP

| Thiết bị | Interface / Mạng | Địa chỉ IP |
|---|---|---|
| pfSense WAN | Bridged | 192.168.1.10/24 |
| pfSense LAN | Host-only / VMnet1 | 10.0.0.1/8 |
| pfSense DMZ | LAN Segment `dmz-net` | 172.16.0.1/16 |
| Windows Server (DC) | LAN | 10.0.0.2/8 |
| Ubuntu Server (LAN-Test) | LAN | 10.0.0.3/8 |
| Máy host | VMnet1 | 10.0.0.100/8 |
| Kali Linux (DMZ) | `dmz-net` | 172.16.0.2/16 |

- Gateway LAN: `10.0.0.1`
- Gateway DMZ: `172.16.0.1`
- DNS kiểm thử: `8.8.8.8`

> **Lưu ý:** Theo yêu cầu bài lab, DMZ-Web là Windows Server. Do em không có máy Windows Server thứ hai nên sử dụng Kali Linux làm máy thay thế để kiểm thử các rule firewall ở mạng DMZ.

---

## 3. Thiết lập pfSense

pfSense được cấu hình với 3 card mạng:

1. **WAN:** Bridged
2. **LAN:** Host-only – VMnet1
3. **DMZ:** LAN Segment – `dmz-net`

Các địa chỉ IP chính:

- WAN: `192.168.1.10/24`
- LAN: `10.0.0.1/8`
- DMZ: `172.16.0.1/16`

DHCP được tắt và các máy trong lab sử dụng IP tĩnh.

---

# 4. Scenario 1 – Chặn ICMP nhưng cho phép DNS và Web

## Mục tiêu

Thực hiện firewall rule trên LAN:

- Chặn ICMP từ LAN ra ngoài.
- Cho phép DNS TCP/UDP port 53.
- Cho phép HTTP port 80.
- Cho phép HTTPS port 443.

## Rule đã cấu hình

Thứ tự rule:

1. Anti-Lockout Rule
2. Block ICMP LAN → Any
3. Allow DNS LAN → Any – TCP/UDP 53
4. Allow Web LAN → Any – TCP 80
5. Allow HTTPS LAN → Any – TCP 443
6. Default allow LAN → Any – Disabled
7. Default allow LAN IPv6 → Any – Disabled

Sau khi cấu hình, em **Apply Changes** và **Reset States**.

## Kiểm tra

### Ping

```bash
ping 8.8.8.8
```

Kết quả:

```text
4 packets transmitted, 0 received, 100% packet loss
```

=> Ping bị chặn đúng theo rule ICMP.

### DNS

```cmd
nslookup example.com 8.8.8.8
```

Kết quả phân giải tên miền thành công.

=> DNS vẫn được phép.

### HTTPS

```cmd
curl.exe -4 --resolve example.com:443:104.20.23.154 https://example.com
```

Kết quả trả về nội dung của **Example Domain**.

=> HTTPS hoạt động bình thường.

**Kết luận:** Scenario 1 đạt yêu cầu: ICMP bị chặn nhưng DNS, HTTP/HTTPS vẫn được phép.

---

# 5. Scenario 2 – Chỉ cho phép DC truy cập Internet

## Mục tiêu

Chỉ cho phép máy Domain Controller có IP `10.0.0.2` truy cập ra ngoài. Các máy khác trong mạng LAN bị chặn.

## Rule đã cấu hình

Thứ tự rule:

1. Anti-Lockout Rule
2. Allow DC to Any – Source `10.0.0.2`
3. Block LAN to Any – Source `LAN subnets`
4. Default allow LAN → Any – Disabled
5. Default allow LAN IPv6 → Any – Disabled

Sau khi cấu hình, em **Apply Changes** và **Reset States**.

## Kiểm tra trên DC

Từ Windows Server:

```cmd
ping 8.8.8.8
```

Kết quả:

```text
4 packets sent
4 packets received
0% loss
```

=> DC được phép truy cập Internet.

## Kiểm tra trên LAN-Test

Từ Ubuntu Server:

```bash
ping -c 4 8.8.8.8
```

Kết quả:

```text
4 packets transmitted, 0 received, 100% packet loss
```

=> Máy LAN-Test bị chặn.

**Kết luận:** Scenario 2 đạt yêu cầu: chỉ DC `10.0.0.2` được phép truy cập Internet.

---

# 6. Scenario 3 – Cô lập DMZ khỏi LAN

## Mục tiêu

Kiểm tra khả năng:

- Cho phép DMZ truy cập LAN ở trạng thái ban đầu.
- Sau đó chặn DMZ truy cập LAN.
- DMZ vẫn được phép truy cập Internet.

## Bước 1 – Cho phép ICMP trên DC

Trên Windows Server chạy CMD với quyền Administrator:

```cmd
netsh advfirewall firewall add rule name="LAB-Allow-ICMPv4-Echo" protocol=icmpv4:8,any dir=in action=allow
```

Kết quả:

```text
Ok.
```

Rule này chỉ được tạo tạm thời để kiểm tra ping.

## Bước 2 – Kiểm tra trạng thái ban đầu

Trên pfSense tạo rule:

- Pass DMZ subnets → Any

Sau đó **Apply Changes** và **Reset States**.

Từ Kali Linux `172.16.0.2`:

```bash
ping -c 4 10.0.0.2
```

Kết quả:

```text
4 packets transmitted, 4 received, 0% packet loss
```

=> DMZ có thể truy cập DC ở trạng thái ban đầu.

## Bước 3 – Chặn DMZ truy cập LAN

Tạo rule Block đặt phía trên rule Allow:

1. **Block DMZ subnets → LAN subnets**
2. **Allow DMZ subnets → Any**

Sau đó **Apply Changes** và **Reset States**.

### Kiểm tra DMZ → LAN

Từ Kali:

```bash
ping -c 4 10.0.0.2
```

Kết quả:

```text
4 packets transmitted, 0 received, 100% packet loss
```

=> DMZ không còn truy cập được LAN.

### Kiểm tra DMZ → Internet

```bash
ping -c 4 8.8.8.8
```

Kết quả:

```text
4 packets transmitted, 4 received, 0% packet loss
```

=> DMZ vẫn truy cập Internet được.

### Kiểm tra DNS

```bash
nslookup example.com 8.8.8.8
```

DNS phân giải thành công.

## Xóa rule ICMP tạm thời trên DC

Sau khi kiểm tra xong:

```cmd
netsh advfirewall firewall delete rule name="LAB-Allow-ICMPv4-Echo"
```

Kết quả:

```text
Ok.
```

Rule firewall **Block DMZ subnets → LAN subnets** vẫn được giữ lại để thể hiện kết quả Scenario 3.

**Kết luận:** Scenario 3 đạt yêu cầu: DMZ bị cô lập khỏi LAN nhưng vẫn có thể truy cập Internet.

---

# 7. Tổng kết kết quả

| Scenario | Nội dung | Kết quả |
|---|---|---|
| 1 | Chặn ICMP, cho phép DNS/Web | PASS |
| 2 | Chỉ DC được truy cập Internet | PASS |
| 3 | Cô lập DMZ khỏi LAN | PASS |

---

# 8. Một số lỗi gặp phải và cách xử lý

## Lỗi 1 – Kali không có địa chỉ IPv4 trong mạng DMZ

Kali ban đầu chưa nhận được IPv4 khi kết nối vào `dmz-net`.

Em cấu hình IP tĩnh bằng:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.method manual ipv4.addresses 172.16.0.2/16 ipv4.gateway 172.16.0.1 ipv4.dns 8.8.8.8
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

Sau đó Kali có:

```text
IP:      172.16.0.2/16
Gateway: 172.16.0.1
DNS:     8.8.8.8
```

## Lỗi 2 – Kali chưa kết nối đúng mạng DMZ

Trong VMware, Network Adapter của Kali được chuyển sang:

```text
LAN Segment: dmz-net
```

Sau đó kiểm tra lại kết nối và cấu hình IP.

---

# 9. Kết luận

Qua 3 scenario, em đã thực hiện kiểm thử các rule firewall trên pfSense gồm:

- Kiểm soát ICMP, DNS và Web.
- Chỉ cho phép Domain Controller truy cập Internet.
- Cô lập mạng DMZ khỏi mạng LAN nhưng vẫn cho phép DMZ truy cập Internet.

Các kết quả kiểm thử đều đúng với rule đã cấu hình và cả 3 scenario đều đạt **PASS**.
