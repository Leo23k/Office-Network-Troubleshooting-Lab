# Ca sự cố mạng

Tám ca đã gây lỗi, chẩn đoán, sửa và kiểm tra lại trên Packet Tracer.

## Ca 1: Cổng switch bị shutdown

| Mục | Nội dung |
|---|---|
| Gây lỗi | SW1: `interface fa0/5`, `shutdown` |
| Triệu chứng | PC2 mất mạng, các PC khác bình thường |
| Phạm vi | 1 máy |
| Chẩn đoán | `show interfaces status` trên SW1: Fa0/5 ở trạng thái `disabled`, VLAN 10 |
| Sửa | `interface fa0/5`, `no shutdown` |
| Kiểm tra lại | Fa0/5 báo `changed state to up` |

## Ca 2: ACL chặn nhầm web của VLAN GUEST

| Mục | Nội dung |
|---|---|
| Gây lỗi | R1: ACL `BLOCK-WEB` chặn tcp 80 từ 192.168.20.0/24 tới 192.168.10.5, gắn `ip access-group BLOCK-WEB in` trên g0/0.20 |
| Triệu chứng | PC4 mở `http://192.168.10.5` báo Request Timeout, các dịch vụ khác vẫn chạy |
| Phạm vi | Cả VLAN 20, chỉ riêng dịch vụ web |
| Chẩn đoán | `show access-lists` trên R1: dòng `deny tcp 192.168.20.0 0.0.0.255 host 192.168.10.5 eq www` có 12 match |
| Sửa | Gỡ `ip access-group` khỏi g0/0.20, xóa ACL `BLOCK-WEB`, `write` |
| Kiểm tra lại | PC4 mở được trang web của SRV1 |

## Ca 3: Máy in offline do đổi IP

| Mục | Nội dung |
|---|---|
| Gây lỗi | Đổi IP PRN1 từ 192.168.10.20 thành 192.168.10.77 |
| Triệu chứng | Người dùng không in được, `ping 192.168.10.20` từ PC1 mất 4/4 gói |
| Phạm vi | 1 thiết bị, các máy khác bình thường |
| Chẩn đoán | Cổng và VLAN đúng. `ping 192.168.10.77` có Reply, tức máy in còn sống nhưng đổi IP |
| Sửa | Đặt lại IP cố định 192.168.10.20 cho PRN1 |
| Kiểm tra lại | `ping 192.168.10.20` từ PC1 có Reply 4/4 |

## Ca 4: DNS cấu hình sai

| Mục | Nội dung |
|---|---|
| Gây lỗi | PC1 đặt IP tĩnh 192.168.10.50, mask 255.255.255.0, gateway 192.168.10.1, DNS 192.168.10.99 |
| Triệu chứng | Ping gateway và máy in được, nhưng `nslookup intranet.local` báo `DNS request timed out` |
| Phạm vi | 1 máy |
| Chẩn đoán | Ping IP được nên mạng thông. `ipconfig` và `nslookup` cho thấy máy hỏi DNS server 192.168.10.99 không tồn tại |
| Sửa | Chuyển PC1 về DHCP, nhận DNS 192.168.10.5 |
| Kiểm tra lại | `nslookup intranet.local` trả về 192.168.10.5 |

## Ca 5: Cổng switch gán sai VLAN

| Mục | Nội dung |
|---|---|
| Gây lỗi | SW1: `interface fa0/6`, `switchport access vlan 20` (cổng của PC3) |
| Triệu chứng | PC3 giữ IP 192.168.10.102 nhưng ping gateway 192.168.10.1 timeout 4/4 |
| Phạm vi | 1 máy, các máy khác bình thường |
| Chẩn đoán | `show vlan brief` trên SW1: Fa0/6 nằm trong VLAN 20 GUEST thay vì VLAN 10 OFFICE |
| Sửa | `interface fa0/6`, `switchport access vlan 10` |
| Kiểm tra lại | Fa0/6 về VLAN 10, ping gateway có Reply (2 gói đầu timeout trong lúc chờ ARP) |

## Ca 6: Cáp đứt

| Mục | Nội dung |
|---|---|
| Gây lỗi | Dùng công cụ Delete xóa dây PC3 tới SW1 |
| Triệu chứng | PC3 mất mạng, sơ đồ không còn dây |
| Phạm vi | 1 máy |
| Chẩn đoán | `show interfaces status` trên SW1: Fa0/6 `notconnect` |
| Sửa | Nối lại dây Copper Straight-Through từ PC3 FastEthernet0 tới SW1 Fa0/6 |
| Kiểm tra lại | Fa0/6 `connected`, VLAN 10 |

## Ca 7: VLAN GUEST không nhận được DHCP

| Mục | Nội dung |
|---|---|
| Gây lỗi | R1: `interface g0/0.20`, `no ip helper-address 192.168.10.5` |
| Triệu chứng | PC4 chọn DHCP báo `DHCP request failed`, không có IP, gateway 0.0.0.0 |
| Phạm vi | Cả VLAN 20 (máy mới hoặc máy xin lại IP), VLAN 10 vẫn bình thường |
| Chẩn đoán | Máy ở VLAN khác với máy chủ DHCP nên cần relay trên sub-interface g0/0.20. Lệnh `ip helper-address` đã bị gỡ |
| Sửa | `interface g0/0.20`, `ip helper-address 192.168.10.5`, `write` |
| Kiểm tra lại | PC4 nhận 192.168.20.102, gateway 192.168.20.1, DNS 192.168.10.5 |

## Ca 8: Trunk giữa switch và router bị đặt sai

| Mục | Nội dung |
|---|---|
| Gây lỗi | SW1: `interface g0/1`, `switchport mode access` |
| Triệu chứng | PC1 ping 192.168.20.1 timeout 4/4. Mọi VLAN mất liên lạc với router |
| Phạm vi | Cả hai VLAN, mất định tuyến giữa các VLAN |
| Chẩn đoán | `show interfaces trunk` không còn cổng nào, `show vlan brief` thấy Gig0/1 nằm trong VLAN 1 như cổng thường |
| Sửa | `interface g0/1`, `switchport mode trunk` |
| Kiểm tra lại | `show interfaces trunk` thấy Gig0/1 trunking 802.1q. Ping 192.168.20.1 có Reply sau vài gói đầu timeout trong lúc cổng ổn định lại |

## Kiểm tra đường đi bằng tracert

`tracert 192.168.10.5` từ PC4 (VLAN 20): chặng 1 là 192.168.20.1 (R1, sub-interface g0/0.20), chặng 2 là 192.168.10.5 (SRV1). Gói đi qua router đúng một lần, khớp với mô hình router-on-a-stick.

Ghi chú: đã thử đặt mask 255.255.255.252 cho PC1 nhưng Packet Tracer vẫn ping được gateway nên không tái hiện được lỗi, ca này không đưa vào.

Mẫu ticket cho ca 3:

```
Tiêu đề: Máy in văn phòng báo offline
Triệu chứng: người dùng không in được, ping 192.168.10.20 không có Reply
Phạm vi ảnh hưởng: 1 thiết bị (máy in), các máy khác bình thường
Đã kiểm tra: cổng Fa0/3 và VLAN 10 đúng, PC1 ping gateway được
Nguyên nhân: máy in bị đổi IP thành 192.168.10.77
Cách xử lý: đặt lại IP cố định 192.168.10.20 cho máy in
Kết quả kiểm tra: ping 192.168.10.20 có Reply
```
