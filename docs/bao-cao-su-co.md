# Báo cáo sự cố mạng văn phòng

## Tóm tắt

Trên mạng lab gồm router R1, switch SW1, máy chủ SRV1, máy in PRN1 và 5 máy trạm, đã gây và xử lý 8 sự cố. Tất cả được chẩn đoán bằng lệnh kiểm tra, sửa và kiểm tra lại. Mỗi sự cố chỉ ảnh hưởng một máy, một thiết bị hoặc một VLAN.

## Bảng tổng hợp

| # | Sự cố | Phạm vi | Dấu hiệu | Lệnh chẩn đoán | Cách sửa |
|---|---|---|---|---|---|
| 1 | Cổng Fa0/5 bị shutdown | PC2 | PC2 mất mạng | `show interfaces status` | `no shutdown` |
| 2 | ACL chặn nhầm web của VLAN GUEST | VLAN 20 | PC4 mở web báo Request Timeout | `show access-lists` | Gỡ ACL khỏi g0/0.20 |
| 3 | Máy in đổi IP sang .77 | PRN1 | Ping .20 mất 4/4 gói | `ping` .20 và .77 | Đặt lại IP .20 |
| 4 | DNS cấu hình sai | PC1 | Ping IP được, `nslookup` timeout | `nslookup`, `ipconfig` | Về DHCP |
| 5 | Cổng Fa0/6 sai VLAN | PC3 | Ping gateway timeout | `show vlan brief` | Đặt lại VLAN 10 |
| 6 | Cáp PC3 đứt | PC3 | PC3 mất mạng, sơ đồ không có dây | `show interfaces status` | Nối lại cáp |
| 7 | Thiếu DHCP relay trên g0/0.20 | VLAN 20 | PC4 báo DHCP request failed | Xem cấu hình g0/0.20 | Thêm `ip helper-address` |
| 8 | Trunk Gig0/1 bị đặt mode access | Cả hai VLAN | Ping gateway VLAN 20 timeout | `show interfaces trunk`, `show vlan brief` | `switchport mode trunk` |

## Cách phân biệt nhanh

- Một máy mất mạng, các máy khác bình thường: kiểm tra cáp, cổng switch (shutdown, sai VLAN) và cấu hình IP của máy đó. Sự cố 1, 4, 5, 6.
- Một thiết bị không tới được dù cổng và VLAN đúng: kiểm tra IP của thiết bị, ping thử IP khác. Sự cố 3.
- Cả một VLAN lỗi: kiểm tra router, ACL và DHCP relay. Sự cố 2, 7.
- Cả hai VLAN mất liên lạc với router: kiểm tra đường trunk giữa switch và router. Sự cố 8.
- Ping IP được nhưng gõ tên không được: lỗi DNS. Sự cố 4.

## Thứ tự kiểm tra đề xuất

1. Xem sơ đồ và đèn trạng thái của cổng.
2. `ipconfig` trên máy lỗi để xem IP, gateway, DNS.
3. Ping gateway, rồi ping máy khác cùng VLAN, rồi máy VLAN khác.
4. Trên switch: `show interfaces status`, `show vlan brief`.
5. Trên router: `show ip interface brief`, `show access-lists`, cấu hình sub-interface.
6. `nslookup` nếu chỉ tên miền lỗi.

## Bài học

- Ping gateway được thì lớp 1 đến lớp 3 trong VLAN đã ổn, lỗi nằm xa hơn.
- Ping bằng IP và bằng tên cho kết quả khác nhau thì nghi ngờ DNS.
- Gói ping đầu tiên có thể timeout trong lúc chờ ARP, không nên kết luận ngay khi chỉ mất 1 đến 2 gói.
- Sau mỗi lần sửa phải kiểm tra lại bằng đúng lệnh đã dùng để phát hiện lỗi.

## Hạn chế

- Sai subnet mask (255.255.255.252) không tái hiện được trong Packet Tracer nên không đưa vào.
- Chi tiết từng ca: [incidents.md](incidents.md).
