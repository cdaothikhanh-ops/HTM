# EDU-NET – Hệ Thống Mạng Trung Tâm Đào Tạo CNTT Tập Trung

Dự án thiết kế và triển khai hạ tầng mạng trên **GNS3** phối hợp **Windows Server 2022** cho trung tâm đào tạo CNTT gồm **4 phòng Lab** và **1 khu văn phòng**.

## 🛠️ Công Nghệ Triển Khai
* **VLAN (802.1Q):** Phân chia logic mạng giữa các phòng Lab và khối văn phòng.
* **Inter-VLAN Routing:** Định tuyến trực tiếp trên Core Switch Layer 3.
* **DHCP Relay Agent:** Sử dụng `ip helper-address` chuyển tiếp yêu cầu cấp IP động từ máy trạm về Windows Server.
* **Bảo mật Standard ACL:** Giới hạn quyền Telnet vào Core Switch, chỉ cho phép dải IP của Quản trị viên (IT).

---

## 🗺️ Quy Hoạch Địa Chỉ IP

| Khu vực | VLAN | Network Address | Subnet Mask | Default Gateway |
| :--- | :---: | :--- | :--- | :--- |
| **Lab 1** | 10 | `192.168.10.0` | `255.255.255.192` (/26) | `192.168.10.1` |
| **Lab 2** | 20 | `192.168.10.64` | `255.255.255.192` (/26) | `192.168.10.65` |
| **Lab 3** | 30 | `192.168.10.128` | `255.255.255.192` (/26) | `192.168.10.129` |
| **Lab 4** | 40 | `192.168.10.192` | `255.255.255.192` (/26) | `192.168.10.193` |
| **Văn phòng** | 50 | `192.168.20.0` | `255.255.255.0` (/24) | `192.168.20.1` |
| **IT/Server** | 99 | `192.168.99.0` | `255.255.255.0` (/24) | `192.168.99.1` |

> 📌 **Windows Server 2022 IP:** `192.168.99.10` / Subnet Mask: `255.255.255.0`

---

## 💻 Cấu Hình Core Switch (Cisco CLI)

### Định tuyến Inter-VLAN & DHCP Relay Agent
```ios
Switch_Core1# configure terminal
Switch_Core1(config)# ip routing

! Cấu hình các VLAN và IP Helper
interface vlan 10
 ip address 192.168.10.1 255.255.255.192
 ip helper-address 192.168.99.10
exit

interface vlan 20
 ip address 192.168.10.65 255.255.255.192
 ip helper-address 192.168.99.10
exit

interface vlan 30
 ip address 192.168.10.129 255.255.255.192
 ip helper-address 192.168.99.10
exit

interface vlan 40
 ip address 192.168.10.193 255.255.255.192
 ip helper-address 192.168.99.10
exit

interface vlan 50
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.99.10
exit
! Tạo tài khoản admin và ACL chặn truy cập trái phép
username admin privilege 15 secret 123456
access-list 1 permit 192.168.99.0 0.0.0.255

line vty 0 4
 login local
 transport input telnet
 access-class 1 in
end
write memory
interface vlan 99
 ip address 192.168.99.1 255.255.255.0
exit
