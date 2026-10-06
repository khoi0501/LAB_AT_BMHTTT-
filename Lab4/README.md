# README.md -- LAB 4: Khảo sát và đánh giá bề mặt mạng bằng Nmap

## 1. Thông tin sinh viên

-   **Họ tên:** Phạm Ngọc Anh Khôi
-   **MSSV:** 1150070023
-   **Tên Lab:** LAB 4 -- Khảo sát và đánh giá bề mặt mạng bằng Nmap
Link Youtube: https://www.youtube.com/watch?v=7_-QnF4VGvY

## 2. Phiên bản môi trường

-   **Máy kiểm tra:** Kali Linux
-   **Máy mục tiêu:** Metasploitable 2
-   **Mạng sử dụng:** VMware Host-Only
-   **Kali Linux:** IP `192.168.164.129/24`
-   **Metasploitable 2:** IP `192.168.164.128/24`
-   **Nmap:** 7.99
-   **Kiểu môi trường:** Hai máy ảo Kali Linux và Metasploitable 2 được
    kết nối trong cùng mạng lab Host-Only.

## 3. Cách dựng môi trường

Em dựng môi trường gồm 2 máy ảo:

1.  Mở máy ảo Kali Linux.
2.  Mở máy ảo Metasploitable 2.
3.  Cấu hình card mạng của hai máy ảo sử dụng mạng **Host-Only**.
4.  Kiểm tra địa chỉ IP trên Kali bằng lệnh:

``` bash
ip -br addr
```

Kali nhận IP:

``` text
192.168.164.129/24
```

5.  Trên Metasploitable 2 kiểm tra IP bằng:

``` bash
ifconfig
```

Metasploitable 2 nhận IP:

``` text
192.168.164.128
```

6.  Kiểm tra kết nối giữa hai máy bằng:

``` bash
ping -c 4 192.168.164.128
```

Kết quả ping thành công nên em tiếp tục thực hiện các bước quét Nmap.

## 4. Các tình huống đã thực hiện

### Tình huống 1 -- Kiểm tra IP Kali

Lệnh thực hiện:

``` bash
ip -br addr
```

Kết quả: Kali có IP `192.168.164.129/24`.

**Kết quả: PASS**

------------------------------------------------------------------------

### Tình huống 2 -- Kiểm tra IP Metasploitable 2

Lệnh thực hiện:

``` bash
ifconfig
```

Kết quả: Metasploitable 2 có IP `192.168.164.128`.

**Kết quả: PASS**

------------------------------------------------------------------------

### Tình huống 3 -- Kiểm tra các host đang hoạt động

Lệnh thực hiện:

``` bash
sudo nmap -sn 192.168.164.0/24
```

Nmap phát hiện 4 host đang hoạt động trong mạng, trong đó có Kali và
Metasploitable 2.

**Kết quả: PASS**

------------------------------------------------------------------------

### Tình huống 4 -- Quét TCP

Lệnh thực hiện:

``` bash
nmap -sT 192.168.164.128
```

Kết quả phát hiện nhiều cổng TCP đang mở, ví dụ FTP, SSH, Telnet, SMTP,
HTTP, SMB, MySQL và PostgreSQL.

**Kết quả: PASS**

------------------------------------------------------------------------

### Tình huống 5 -- Phát hiện phiên bản dịch vụ

Lệnh thực hiện:

``` bash
sudo nmap -sV 192.168.164.128
```

Kết quả hiển thị tên dịch vụ và phiên bản. Ví dụ có `vsftpd 2.3.4`,
`OpenSSH 4.7p1`, `Apache httpd 2.2.8`, `MySQL 5.0.51a`.

**Kết quả: PASS**

------------------------------------------------------------------------

### Tình huống 6 -- Xác định hệ điều hành

Lệnh thực hiện:

``` bash
sudo nmap -O 192.168.164.128
```

Kết quả Nmap nhận diện hệ điều hành là Linux, thông tin hiển thị khoảng
Linux 2.6.9 - 2.6.33.

**Kết quả: PASS**

------------------------------------------------------------------------

### Tình huống 7 -- Kiểm tra SMB bằng NSE

Lệnh thực hiện:

``` bash
sudo nmap -p 139,445 --script smb-os-discovery 192.168.164.128
```

Kết quả cho thấy cổng 139 và 445 đang mở. Script `smb-os-discovery` trả
về thông tin Samba, máy có tên `metasploitable` và domain `localdomain`.

**Kết quả: PASS**

------------------------------------------------------------------------

### Tình huống 8 -- Xuất kết quả quét

Lệnh thực hiện:

``` bash
sudo nmap -sV -O -oA Lab4_Report_Khoi 192.168.164.128
```

Sau đó chuyển XML sang HTML:

``` bash
xsltproc Lab4_Report_Khoi.xml -o Lab4_Report_Khoi.html
```

Kiểm tra các file:

``` bash
ls -la Lab4_Report_Khoi*
```

Kết quả có các file:

``` text
Lab4_Report_Khoi.gnmap
Lab4_Report_Khoi.html
Lab4_Report_Khoi.nmap
Lab4_Report_Khoi.xml
```

**Kết quả: PASS**

## 5. Tổng hợp kết quả

  STT   Tình huống                     Kết quả
  ----- ------------------------------ ---------
  1     Kiểm tra IP Kali               Đã hoàn thành 
  2     Kiểm tra IP Metasploitable 2   Đã hoàn thành 
  3     Host Discovery                 Đã hoàn thành 
  4     TCP Scan                       Đã hoàn thành 
  5     Service/Version Detection      Đã hoàn thành 
  6     OS Detection                   Đã hoàn thành 
  7     NSE SMB Discovery              Đã hoàn thành 
  8     Xuất file báo cáo              Đã hoàn thành 

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1 -- Sudo yêu cầu mật khẩu

Khi chạy lệnh Nmap với `sudo`, Terminal hiện:

``` text
[sudo] password for kali:
```

Em nhập mật khẩu tài khoản Kali. Khi nhập mật khẩu thì Terminal không
hiện ký tự, đây là bình thường. Sau đó em nhấn Enter để tiếp tục.

**Kết quả:** Khắc phục được, lệnh chạy bình thường.

### Lỗi 2 -- Kết quả quét dài

Một số lệnh Nmap trả về khá nhiều dòng nên không thể chụp hết trong một
màn hình.

Em chia phần minh chứng thành các ảnh phù hợp. Riêng phần xuất báo cáo
em chia thành 2 ảnh để thể hiện được cả quá trình tạo file và danh sách
file kết quả.

**Kết quả:** Khắc phục được, vẫn nhìn rõ nội dung cần minh chứng.

## 7. Kết luận

Qua bài Lab 4, em đã thực hiện được các bước khảo sát bề mặt mạng bằng
Nmap trên môi trường máy ảo. Em kiểm tra được IP, tìm host đang hoạt
động, quét các cổng TCP, nhận diện dịch vụ và phiên bản, xác định hệ
điều hành, thực hiện NSE đối với SMB và lưu kết quả quét thành các file
báo cáo.

Toàn bộ các tình huống em thực hiện trong môi trường lab đều cho kết quả
**PASS**.
