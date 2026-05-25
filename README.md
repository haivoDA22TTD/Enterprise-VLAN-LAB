# 🏢 Enterprise VLAN Lab 

Dự án mô phỏng và cấu hình hệ thống mạng VLAN cho doanh nghiệp quy mô vừa. Hệ thống sử dụng Switch Layer 3 làm Core Switch để thực hiện định tuyến liên VLAN (Inter-VLAN Routing) nhằm tối ưu hóa hiệu suất truyền tải và tăng cường tính bảo mật.

---

## 📍 Trạng thái dự án hiện tại

- [x] Khởi tạo và phân hoạch 4 VLAN chức năng.
- [x] Cấu hình đường truyền **Trunking (802.1Q)** giữa Core Switch và các Access Switch.
- [x] Cấu hình **SVI (Switch Virtual Interface)** trên Core Switch làm Default Gateway.
- [x] Kích hoạt tính năng định tuyến IP (`ip routing`).
- [x] **Kiểm tra thành công:** Tất cả các máy tính thuộc VLAN 10, 20, 30, 40 đã thông suốt và ping được lẫn nhau.

---

## 🗺️ Quy hoạch mạng & Phân chia VLAN


| VLAN ID | Tên VLAN | Phân vùng chức năng | Mạng Subnet | Default Gateway |
| :---: | :--- | :--- | :--- | :--- |
| **10** | VLAN_KT | Phòng Kế toán | `192.168.10.0/24` | `192.168.10.1` |
| **20** | VLAN_NS | Phòng Nhân sự | `192.168.20.0/24` | `192.168.20.1` |
| **30** | VLAN_KTKT | Phòng Kỹ thuật | `192.168.30.0/24` | `192.168.30.1` |
| **40** | VLAN_BGH | Ban Giám đốc | `192.168.40.0/24` | `192.168.40.1` |

---

## 💻 Kịch bản cấu hình cốt lõi (Core Switch)

Dưới đây là các lệnh cấu hình chính đã thực hiện trên Switch Layer 3:

```text
! 1. Khởi tạo các VLAN
Core-SW# configure terminal
Core-SW(config)# vlan 10
Core-SW(config-vlan)# name VLAN_KT
Core-SW(config)# vlan 20
Core-SW(config-vlan)# name VLAN_NS
Core-SW(config)# vlan 30
Core-SW(config-vlan)# name VLAN_KTKT
Core-SW(config)# vlan 40
Core-SW(config-vlan)# name VLAN_BGH
Core-SW(config-vlan)# exit

! 2. Bật tính năng định tuyến
Core-SW(config)# ip routing

! 3. Cấu hình địa chỉ IP cho các SVI (Interface VLAN)
Core-SW(config)# interface vlan 10
Core-SW(config-if)# ip address 192.168.10.1 255.255.255.0
Core-SW(config)# interface vlan 20
Core-SW(config-if)# ip address 192.168.20.1 255.255.255.0
Core-SW(config)# interface vlan 30
Core-SW(config-if)# ip address 192.168.30.1 255.255.255.0
Core-SW(config)# interface vlan 40
Core-SW(config-if)# ip address 192.168.40.1 255.255.255.0
Core-SW(config-if)# exit

! 4. Cấu hình cổng kết nối xuống Access Switch làm đường Trunk
Core-SW(config)# interface range gigabitEthernet 1/0/1 - 2
Core-SW(config-if)# switchport trunk encapsulation dot1q
Core-SW(config-if)# switchport mode trunk
Core-SW(config-if)# switchport trunk allowed vlan 10,20,30,40
Core-SW(config-if)# end
Core-SW# write memory
```

---

## 🚀 Kế hoạch phát triển tiếp theo (To-Do List)

Để chuẩn hóa mô hình mạng theo tiêu chuẩn Enterprise thực tế, các bước tiếp theo cần triển khai bao gồm:

- [ ] **DHCP Relay Agent:** Cấu hình `ip helper-address` để cấp IP tự động từ Server tập trung.
- [ ] **Security (ACL):** Thiết lập Access Control List để chặn liên lạc giữa các phòng ban nhạy cảm (ví dụ: Chặn VLAN Kỹ thuật truy cập VLAN Kế toán).
- [ ] **Management VLAN:** Tạo VLAN độc lập (ví dụ: VLAN 99) để quản trị thiết bị từ xa qua SSH thay vì dùng VLAN 1 mặc định.
- [ ] **Internet Routing:** Cấu hình Static Route hoặc OSPF nối lên Router biên để đi ra ngoài Internet.
