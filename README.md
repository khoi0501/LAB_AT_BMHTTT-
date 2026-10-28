1. Thông tin sinh viên

* Họ và tên: Phạm Ngọc Anh Khôi
* MSSV: 1150070023
* Lớp: 11_TMĐT
* Bài Lab 8: Local DNS Attack - DNS Spoofing và Cache Poisoning trong mạng ảo cô lập
Link youtube: https://www.youtube.com/watch?v=azcPRS29BLY
2. Phiên bản môi trường

* Máy ảo hóa: VMware Workstation
* pfSense: kế thừa cấu hình từ các lab trước
* WEB-PROXY: máy proxy dùng để áp chính sách Web Protection
* Victim/Client: Ubuntu 26.04.1 LTS
* DNS-SERVER (nếu có dùng tới trong lab): Ubuntu 26.04.1 LTS, BIND9
* Attacker: Kali Linux
* Windows Server 2025: dùng cho phần domain/AD liên quan

3. Cách dựng môi trường

Đầu tiên em tắt pfSense rồi tạo snapshot trước khi đụng vào gì hết, để lỡ cấu hình sai còn có đường quay lại. Sau đó em thêm Adapter 5 cho pfSense, attach qua Internal Network đặt tên dns-lab, vô Interfaces > Assignments gán thành OPT mới, đặt IP tĩnh 10.9.0.1/24 và không chọn Upstream Gateway cho nó. Tạo xong interface em làm luôn bộ rule cô lập DNSLAB, cho phép DNS-SERVER hỏi AUTH-DNS, cho Victim ra web DMZ, còn lại chặn hết qua LAN-DOMAIN với MGMT để không ảnh hưởng mấy lab trước.

Tiếp theo em dựng DNS-SERVER trên Ubuntu, đặt IP tĩnh 10.9.0.10, cài BIND9 với mấy tool kiểm thử, chép bộ config chuẩn phát sẵn vào rồi named-checkconf/named-checkzone để coi zone lab9.test với zone reverse load có OK không, xong mới start lại service.

Victim với Attacker em cấu hình bằng Netplan (không sửa tay resolv.conf vì systemd-resolved hay ghi đè), Victim trỏ DNS về 10.9.0.10, Kali thì gắn thêm IP 10.9.0.200 trên card dns-lab, cài Scapy/Wireshark, bật Promiscuous Mode = Allow VMs để bắt được gói giữa các máy trong Internal Network. Cuối cùng em chạy thử baseline, hỏi www.external.lab9.test coi có ra đúng 172.16.6.10 chưa rồi mới qua phần tấn công.

4. Các tình huống đã thực hiện và kết quả

Task 1 em đổi /etc/hosts trên Victim, thêm dòng trỏ www.lab9.test qua 10.9.0.200. Kết quả là curl/trình duyệt load ra trang giả của Kali, nhưng dig hỏi thẳng DNS-SERVER thì vẫn ra đúng 172.16.6.10, vì dig không đọc file hosts.

Task 2 em chạy script spoof trên Kali ở chế độ nhắm vào Victim, Kali rình query của Victim gửi qua DNS-SERVER rồi gửi ngay 1 phản hồi giả về trước, giả đúng Transaction ID với port. Lúc thắng được cuộc đua thì Victim nhận tạm 10.9.0.200 thay vì 172.16.6.10, coi Wireshark thấy có cả 2 gói trả lời (giả với thật) cùng về Victim.

Task 3 em chuyển hướng tấn công qua DNS-SERVER, Kali giả mạo phản hồi từ phía AUTH-DNS gửi về resolver trước bản thật. Khi BIND9 chấp nhận thì record sai 10.9.0.200 bị lưu luôn vào cache trong vòng TTL 120 giây, lúc đó bao nhiêu client dùng chung resolver hỏi lại cũng dính hết chứ không riêng gì Victim. Sau đó em flush cache với khôi phục /etc/hosts, tắt script spoof, kiểm tra lại thấy mọi thứ trả về đúng 172.16.6.10 như ban đầu.

5. Lỗi gặp phải và cách khắc phục

Có lúc Victim không phân giải được tên miền, em kiểm tra lại thì do DNS 10.9.0.10 chưa áp dụng đúng qua Netplan, với BIND cũng chưa listen đúng địa chỉ, sau khi sửa lại netplan với named.conf.options thì hết.

Có đợt DNS-SERVER không hỏi được qua APP-TEST, nguyên nhân là named.conf.local khai forward zone sai, với rule trên pfSense từ DNSLAB qua 172.16.6.20:53 đang bị block, em mở lại rule với sửa đúng forward zone thì query đi qua được.

Kali có lúc không bắt được gói nào trong Wireshark, kiểm tra lại thì Promiscuous Mode đang để Deny, em đổi qua Allow VMs với chọn đúng lại interface dns-lab thì thấy query bình thường.

Script spoof chạy mà không in ra dòng forged, em soát lại domain với mode đang gõ có đúng không, với có lần là do query đang lấy từ cache cũ, em flush cache DNS-SERVER rồi thử lại thì script bắt được query mới và in ra kết quả.

Có lần Task 2 làm xong ngon mà qua Task 3 không ăn, em kiểm tra thì do resolver không chịu nhận forged reply vì query tuple (port/transaction id) bị lệch so với query thật, soát lại script với thử lại lần nữa thì mới qua được.
