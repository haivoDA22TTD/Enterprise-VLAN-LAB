# 🏢 Smart Enterprise Network with Port Security

Hạ tầng mạng doanh nghiệp thông minh đa phân vùng được mô phỏng trên nền tảng **Cisco Packet Tracer**. Dự án triển khai giải pháp tự động hóa tòa nhà bằng công nghệ IoT quản lý tập trung, kết hợp chính sách bảo mật lớp Access kiểm soát toàn diện thiết bị ngoại vi đầu cuối tại phân vùng văn phòng.

---

## 🗺️ Sơ Đồ Kiến Trúc Hệ Thống (System Architecture)

```mermaid
graph TD
    %% Định nghĩa các node thiết bị dựa trên sơ đồ thực tế
    A[🛡️ Router 2811] <--> B[🔥 ASA 5506-X]
    B <--> C[🎛️ Layer 3 Core Switch]
    
    subgraph Phân Vùng Văn Phòng (Bên Trái)
        C <--> D[🔌 Switch Access Left]
        D <--> E1[💻 PC_KinhTe - VLAN 10]
        D <--> E2[💻 PC_NhanSu - VLAN 20]
        D <--> E3[🖨️ May_In]
        D -. X .- F[⚠️ Laptop_Intern - Bị Khóa/Shutdown]
    end

    subgraph Phân Vùng Quản Lý & Tự Động Hóa IoT (Bên Phải)
        C <--> G[🔌 Switch Access Right]
        G <--> H1[💻 PC_GiamDoc - VLAN 30]
        G <--> H2[💻 PC_KyThuat - VLAN 40]
        G <--> I[🖥️ Dau_Ghi_Camera - IoT Server]
        G <--> J1[📡 Motion Detector IoT]
        G <--> J2[🚪 Smart_Door]
        G <--> J3[📷 Webcam]
    end

    %% Định nghĩa style màu sắc trực quan cho các vùng
    style A fill:#a2d2ff,stroke:#333,stroke-width:2px
    style B fill:#ffafcc,stroke:#333,stroke-width:2px
    style C fill:#ffc8dd,stroke:#333,stroke-width:2px
    style E1 fill:#bde0fe,stroke:#333,stroke-width:2px
    style E2 fill:#fff3b0,stroke:#333,stroke-width:2px
    style F fill:#ff4d6d,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5
    style I fill:#d8f3dc,stroke:#333,stroke-width:2px
    style J1 fill:#e8e8e4,stroke:#333,stroke-width:2px
    style J2 fill:#e8e8e4,stroke:#333,stroke-width:2px
```

---

## 🛠️ Công Nghệ & Giải Pháp Triển Khai


| Phân Vùng | Thiết Bị Đầu Cuối | Công Nghệ & Giải Pháp | Trạng Thái Demo |
| :--- | :--- | :--- | :--- |
| **Hạ tầng Mạng** | `Core Switch` `Router 2811` `ASA 5506-X` | Chia VLAN phòng ban (VLAN 10/20/30/40), định tuyến Inter-VLAN | ![Passed](https://shields.io) |
| **Hệ Thống IoT** | `Smart_Door` `Motion Detector` `Dau_Ghi_Camera` | Tự động hóa qua cơ chế **IoT Conditions** trên nền tảng Web quản lý | ![Passed](https://shields.io) |
| **Bảo Mật Cổng** | `Switch Access Left` | Ngăn chặn thiết bị lạ truy cập Internet bằng **Port Security Violation Shutdown** | ![Passed](https://shields.io) |

---

## 🧪 Cơ Chế Hoạt Động (Demo Logic)

### 1. Cơ chế tự động hóa IoT (Khu vực bên phải)
* **Kịch bản**: Nhân viên di chuyển qua lại tại khu vực cửa ra vào.
* **Logic hoạt động**: `Motion Detector` phát hiện chuyển động $\rightarrow$ truyền tín hiệu số về Server `Dau_Ghi_Camera` ($192.168.60.3$) $\rightarrow$ Hệ thống tự động kích hoạt luật mở khóa `Unlock` cho thiết bị `Smart_Door`. Khi không còn người, cửa tự động phản hồi đóng và chốt khóa lại để bảo mật an ninh tòa nhà.

### 2. Cơ chế an ninh Port Security (Khu vực bên trái)
* **Kịch bản**: Kẻ gian rút dây mạng từ máy `PC_KinhTe` (Cổng `Fa0/2`) ra để kết nối thiết bị lạ `Laptop_Intern` vào hệ thống mạng công ty.
* **Logic hoạt động**: `Switch Access Left` phát hiện địa chỉ MAC của `Laptop_Intern` không khớp với địa chỉ **MAC Sticky** hợp lệ duy nhất đã lưu trước đó. Cơ chế bảo mật lập tức đưa cổng `Fa0/2` vào trạng thái lỗi **Err-Disabled (Shutdown)**. Cổng mạng chuyển sang màu đỏ rực, ngắt toàn bộ lưu lượng Internet và cô lập hoàn toàn thiết bị tấn công.
