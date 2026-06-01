# 🌐 Cisco Packet Tracer Lab: OSPF Routing & Site-to-Site IPsec VPN

[![Packet Tracer Version](https://shields.io)](https://cisco.com)
[![Network Security](https://shields.io)]()
[![Routing Protocol](https://shields.io)]()

Dự án này triển khai một mô hình giả lập hệ thống mạng doanh nghiệp gồm hai chi nhánh: **Trụ sở chính (HQ)** và **Chi nhánh phụ (Branch)** kết nối thông suốt thông qua nhà cung cấp dịch vụ mạng **Internet (ISP)**. Hệ thống kết hợp giao thức định tuyến động **OSPF** nội bộ và giải pháp bảo mật kênh truyền **Site-to-Site IPsec VPN**.

---

## 🗺️ Sơ đồ Kiến trúc Mạng (Topology)

Mô hình mạng được xây dựng trên các dòng Router tối ưu trong môi trường giả lập:
*   **HQ-Router (Cisco 1941)**: Đóng vai trò Router biên đầu não tại trụ sở chính.
*   **BR-Router (Cisco 1841)**: Đóng vai trò Router biên tại chi nhánh.
*   **ISP-Router (Cisco 2911)**: Giả lập môi trường mạng Internet công cộng.

```text
[ Vùng HQ LAN ]                              [ Mạng WAN Internet ]                            [ Vùng Branch LAN ]
192.168.10.0/24                                   (ISP Cloud)                                   192.168.20.0/24

      |                                                |                                               |
  [HQ-Switch]                                          |                                          [BR-Switch]

      |                                                |                                               |
[HQ-Router (1941)] <===== (Đường hầm Mã hóa IPsec VPN) =========================================> [BR-Router (1841)]
   [Gi0/1]: .1                                         |                                            [Fa0/1]: .1
   [Gi0/0]: 100.1.1.1 <-----------------> [ISP-Router (2911)] <-----------------------------------> [Fa0/0]: 200.1.1.1
                                         [Gi0/0]: 100.1.1.2
                                         [Gi0/1]: 200.1.1.2
```

---

## 🔢 Quy hoạch Địa chỉ IP (IP Planning)


| Phân vùng | Thiết bị / Giao diện | Địa chỉ IP | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **HQ LAN** | HQ-Router (`Gi0/1`) | `192.168.10.1` | `255.255.255.0` | N/A |
| **HQ LAN** | HQ-PC2 | `192.168.10.20` | `255.255.255.0` | `192.168.10.1` |
| **Branch LAN**| BR-Router (`Fa0/1`) | `192.168.20.1` | `255.255.255.0` | N/A |
| **Branch LAN**| BR-PC1 | `192.168.20.10` | `255.255.255.0` | `192.168.20.1` |
| **WAN HQ** | HQ-Router (`Gi0/0`) | `100.1.1.1` | `255.255.255.252`| N/A |
| **WAN BR** | BR-Router (`Fa0/0`) | `200.1.1.1` | `255.255.255.252`| N/A |

---

## ⚙️ Kịch bản Cấu hình Chi tiết (Tóm tắt CLI)

### 1. Kích hoạt Gói Bảo mật trên HQ-Router (Cisco 1941)
*Do Router 1941 yêu cầu bản quyền để kích hoạt tập lệnh Crypto, ta thực hiện nạp giấy phép thử nghiệm và reload:*
```cisco
HQ-Router(config)# license boot module c1900 technology-package securityk9
HQ-Router(config)# end
HQ-Router# write memory
HQ-Router# reload
```

### 2. Định tuyến & Môi trường WAN
*   **Nội bộ (LAN)**: Kích hoạt định tuyến động **OSPF Area 0** cho các dải mạng Private tại mỗi chi nhánh.
*   **Biên (WAN)**: Khai báo các tuyến đường tĩnh (`Static Route / Default Route`) trỏ thẳng về Gateway của nhà mạng ISP để thông mạch Internet Public.
*   **⚠️ Lưu ý cốt lõi**: `ISP-Router` đóng vai trò là Internet công cộng, **không chạy OSPF** và hoàn toàn không biết thông tin về mạng nội bộ `192.168.x.x`.

### 3. Thiết lập IPsec VPN (Site-to-Site)
Kênh truyền an toàn sử dụng cơ chế bảo mật hai giai đoạn:
*   **Phase 1 (ISAKMP)**: Mã hóa bằng thuật toán `AES`, băm xác thực `SHA`, phương thức xác thực `Pre-Share Key` (`cisco123`) và trao đổi khóa qua `Diffie-Hellman Group 2`.
*   **Phase 2 (IPsec Transform Set)**: Áp dụng đóng gói dữ liệu qua `esp-aes` và `esp-sha-hmac`.
*   **Traffic Mapping**: Kiểm soát dữ liệu bằng mở rộng Access Control List (`ACL 110`) để ép luồng traffic Private giữa 2 chi nhánh đi vào đường hầm bảo mật.

---

## 🧪 Kịch bản Kiểm tra Nghiệm thu (Verification)

Hệ thống giả lập được chứng minh hoạt động hoàn hảo thông qua các bước kiểm tra thực tế:

1.  **Xóa Cache kẹt của máy ảo**: Thực hiện lệnh `arp -d` trên các máy trạm để làm sạch dữ liệu bảng MAC cũ trong Packet Tracer.
2.  **Kiểm tra tính thông suốt qua VPN**: Từ máy **HQ-PC2** (`192.168.10.20`), thực hiện lệnh ping liên tục sang **BR-PC1** (`192.168.20.10`).
    *   *Kết quả*: Gói tin đầu tiên bị Timeout do thiết bị bận thực hiện bắt tay thiết lập khóa Phase 1/Phase 2. Từ gói thứ 2 trở đi, kết nối trả về **`Reply` thông suốt với 0% loss**.
3.  **Xác minh trạng thái bảo mật**: Truy cập CLI của Router biên, gõ lệnh:
    ```cisco
    HQ-Router# show crypto isakmp sa
    ```
    *   *Kết quả thành công*: Hệ thống trả về trạng thái **`QM_IDLE`** — khẳng định đường hầm mã hóa đã dựng lên thành công và dữ liệu đang được bảo vệ tuyệt đối.

---

## 🛠️ Lưu ý về Giới hạn Giả lập (Software Image Limitation)
Mô hình này tập trung sâu vào giải pháp định tuyến OSPF và kết nối VPN Site-to-Site. Tính năng kết nối VPN di động từ bên ngoài mạng Internet vào hệ thống (`VPN Client-to-Site`) tạm thời được lược bỏ bằng CLI do các giới hạn về mã nguồn hệ điều hành IOS bị cắt bớt tính năng xác thực mở rộng (`xauth`) trên các dòng Router phân khúc nhỏ của phiên bản Cisco Packet Tracer hiện tại. 

Giải pháp thay thế thực tế được đề xuất: Người dùng từ xa sẽ thực hiện VPN Client kết nối bắc cầu vào mạng Branch, sau đó tận dụng đường hầm Site-to-Site ổn định sẵn có để truy xuất dữ liệu an toàn về HQ.

---
*Dự án được thực hiện, cấu hình thủ công và đóng gói hoàn chỉnh bởi Người phát triển mạng.*

