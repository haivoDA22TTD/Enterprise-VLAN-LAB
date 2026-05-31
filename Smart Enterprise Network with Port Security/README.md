# 🏢 Smart Enterprise Network with Port Security

Hạ tầng mạng doanh nghiệp thông minh mô phỏng trên nền tảng **Cisco Packet Tracer**. Dự án triển khai giải pháp tự động hóa tòa nhà bằng công nghệ IoT quản lý tập trung, kết hợp chính sách bảo mật lớp Access kiểm soát toàn diện thiết bị ngoại vi đầu cuối.

---

## 🗺️ Sơ Đồ Kiến Trúc Hệ Thống (System Architecture)

```mermaid
graph TD
    %% Định nghĩa các node thiết bị với các icon mô phỏng
    A[🌐 WAN / Internet / Cloud] <--> B[🛡️ Firewall / Core Router]
    
    subgraph Vùng_Switch_Core [Hạ Tầng Core Switch]
        B <--> C[🎛️ Layer 3 Core Switch]
    end

    subgraph Vùng_Văn_Phòng_Bên_Trái [Phân Vùng Văn Phòng - VLAN 50]
        C <--> D[🔌 Switch Access Left]
        D <--> E[💻 PC_KinhTe <br> 'Thiết bị Hợp lệ']
        D -. X .- F[⚠️ Laptop_Intern <br> 'Kẻ Xâm Nhập - Bị Shutdown']
    end

    subgraph Vùng_IoT_Bên_Phải [Phân Vùng Quản Lý & IoT - VLAN 60]
        C <--> G[🔌 Switch Access Right]
        G <--> H[🖥️ Server_Dau_Ghi <br> 'IoT Center Management']
        G <--> I[📡 Motion Detector <br> 'Cảm biến IoT']
        G <--> J[🚪 Smart Door <br> 'Cửa tự động']
    end

    %% Định nghĩa style màu sắc cho các phân vùng cho chuyên nghiệp
    style A fill:#a2d2ff,stroke:#333,stroke-width:2px
    style B fill:#ffafcc,stroke:#333,stroke-width:2px
    style C fill:#ffc8dd,stroke:#333,stroke-width:2px
    style E fill:#bde0fe,stroke:#333,stroke-width:2px
    style F fill:#ff4d6d,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5
    style H fill:#d8f3dc,stroke:#333,stroke-width:2px
    style I fill:#e8e8e4,stroke:#333,stroke-width:2px
    style J fill:#e8e8e4,stroke:#333,stroke-width:2px
```

---

## 🛠️ Công Nghệ & Giải Pháp Triển Khai


| Phân Vùng | Thiết Bị & Công Nghệ | Giải Pháp Bảo Mật / Tự Động Hóa | Trạng Thái Demo |
| :--- | :--- | :--- | :--- |
| **Hạ tầng Mạng** | `Cisco Core Switch` `VLAN 50/60` | Định tuyến và chia vùng độc lập bảo mật | ![Passed](https://shields.io) |
| **Phòng IoT** | `Smart Door` `Motion Sensor` `Server` | Tự động hóa qua cơ chế **IoT Conditions** trên Web Browser | ![Passed](https://shields.io) |
| **Văn Phòng** | `Switch Access` `Sticky MAC` | Ngăn chặn truy cập Internet trái phép bằng **Port Security Violation Shutdown** | ![Passed](https://shields.io) |

---

## 🧪 Cơ Chế Hoạt Động (Demo Logic)

*   **Cơ chế tự động hóa IoT**: Khi thiết bị `Motion Detector` phát hiện có người di chuyển tới gần $\rightarrow$ `Server_Dau_Ghi` nhận tín hiệu $\rightarrow$ Gửi lệnh `Unlock` kích hoạt mở **Smart Door**. Sau khi nhân sự đi qua, cửa tự động phản hồi đóng và khóa lại.
*   **Cơ chế an ninh Port Security**: Khi `Laptop_Intern` (máy lạ) cố tình đấu nối trực tiếp vào cổng mạng của `PC_KinhTe` $\rightarrow$ `Switch Access` nhận diện sai địa chỉ MAC Sticky $\rightarrow$ Kích hoạt trạng thái **Violation Shutdown** ngay lập tức. Cổng mạng chuyển sang màu đỏ (Err-Disabled), chặn đứng toàn bộ lưu lượng Internet và cô lập hoàn toàn thiết bị lạ khỏi hệ thống.
