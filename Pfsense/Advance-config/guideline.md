# 🚦 Advanced Lab: Load-Balancing with Dual WAN on pfSense/Hot Backup

## 🗺️ Topology

<div align="center">

<img src="/images/advance_topology.webp" alt="Network Topology" width="100%"/>

</div>

---

## 🎯 Mục tiêu

Triển khai **cân bằng tải (Load Balancing)** để phân phối lưu lượng mạng qua hai đường WAN, giúp tăng hiệu suất, độ tin cậy và khả năng chịu lỗi của hệ thống.

---

## 🛠️ Các bước thực hiện

### 🔹 Bước 1: Tạo thêm card mạng (mô phỏng WAN2)

- Mở **Virtual Network Editor** → Add Network → Gán **subnet** và **default gateway** cho card mạng mới.

<div align="center">

<img src="/images/subnet_2.webp" alt="Create Subnet" width="100%"/>

</div>

- Gắn thêm **Network Adapter** vào máy ảo PNETLab → chọn card mạng vừa tạo.

<div align="center">

<img src="/images/add_network_adapter.webp" alt="Add Adapter" width="100%"/>

</div>

---

### 🔹 Bước 2: Tạo Cloud mới trong PNETLab

- Tạo một **cloud mới** để mô phỏng WAN2 (adapter mới sẽ là `Cloud 1`).

<div align="center">

<img src="/images/add_cloud.webp" alt="Add Cloud" width="100%"/>

</div>

---

### 🔹 Bước 3: Cấu hình Router thứ 2 đi ra Cloud1

- Tạo một router kết nối tới `Cloud 1` và gán IP thuộc mạng `192.168.214.0/24`.
- Định tuyến mặc định qua cloud này.

<div align="center">

<img src="/images/routing_r2.webp" alt="Routing R2" width="100%"/>

</div>

---

### 🔹 Bước 4: Tạo WAN2 trên pfSense

- Kéo dây từ **pfSense** đến **Router 2** và đặt tên interface là `WAN2`.

<div align="center">

<img src="/images/wan2.webp" alt="WAN2" width="100%"/>

</div>

---

### 🔹 Bước 5: Cấu hình Gateway cho WAN2

- Vào phần **System → Routing → Gateway** để thêm gateway mới cho WAN2.

<div align="center">

<img src="/images/routing_gateway.webp" alt="Gateway WAN2" width="100%"/>

</div>

---

### 🔹 Bước 6: Tạo Gateway Groups (Failover/Load Balance)

- Vào `System → Routing → Gateway Groups` → tạo nhóm gateway:
  - `Tier 1` cho cả 2 WAN nếu muốn cân bằng tải.
  - `Tier 1 + Tier 2` nếu muốn failover(Hot backup).

<div align="center">

<img src="/images/load_balanced.webp" width="100%"/>
<img src="/images/status_gateway.webp" width="100%"/>
<img src="/images/status_gateway_groups.webp" width="100%"/>

</div>

---

### 🔹 Bước 7: Thêm Rule chuyển tiếp Dual Gateway

- Vào `Firewall → Rules → LAN` → chỉnh rule mặc định:
  - Nhấn **Advanced Options** → chọn Gateway Group thay vì default gateway.

<div align="center">

<img src="/images/gateway_dual.webp" width="100%"/>
<img src="/images/rules_change_gateway.webp" width="100%"/>

</div>

---

### 🔹 Bước 8: Kiểm tra hoạt động Load Balancing

- Thử **shutdown Router chính** để kiểm tra failover.
- Nếu vẫn ping được ra `8.8.8.8` qua đường còn lại → thành công!
- Thử tải một cái gì đó nặng thì nó sẽ cân bằng tải 2 đường WAN để kiểm tra.

<div align="center">

<img src="/images/test.webp" alt="Test Load Balancing" width="100%"/>

</div>

---

## ✅ Kết luận

Bạn đã hoàn thành mô hình **Dual WAN Load-Balancing** bằng pfSense. Đây là một kỹ năng quan trọng trong quản trị mạng giúp đảm bảo hệ thống hoạt động ổn định kể cả khi mất một đường kết nối.

---

🧠 _Chúc bạn thực hành thành công!_
