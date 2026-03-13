# VLAN - TRUNK - ACCESS - DHCP

---

## Mục tiêu

Hướng dẫn tạo VLAN, cấu hình DHCP Server trên pfSense để cấp IP cho các máy trong từng VLAN.

---

<div align="center">

<img src="/images/topology_dhcp.webp" alt="Network Topology" width="100%"/>

</div>

---

## Các bước thực hiện

### 1. Chuẩn bị môi trường

Như sơ đồ trên, ta cần tạo **3 VLAN** và cấu hình DHCP cho từng VLAN.

Sau khi hoàn tất phần **Basic Configuration**, truy cập vào giao diện Web pfSense để bắt đầu cấu hình.

---

### 2. Tạo VLAN trên pfSense

- Vào menu: **Interfaces ➔ Assignments ➔ VLANs** để tạo các VLAN như topology.

<div align="center">

<img src="/images/create_vlan.webp" alt="Create VLAN" width="80%"/>

</div>

---

### 3. Gán VLAN vào Interface ảo

- Sau khi tạo VLAN, tiếp tục vào **Interfaces ➔ Assignments** để thêm interface ảo cho từng VLAN.

<div align="center">

<img src="/images/interface_assignments.webp" alt="Interface Assignments" width="80%"/>

</div>

Kết quả: ta sẽ có 3 interface ảo tương ứng với 3 VLAN.

---

### 4. Cấu hình DHCP Server cho từng VLAN

- Vào **Services ➔ DHCP Server**, bật DHCP cho từng VLAN interface.

<div align="center">

<img src="/images/DHCP_server.webp" alt="Enable DHCP Server" width="80%"/>

</div>

- Cấu hình dải IP cho mỗi VLAN (ví dụ: 192.168.30.10 ~ 192.168.30.100).

<div align="center">

<img src="/images/Pool.webp" alt="DHCP Pool" width="80%"/>

</div>

Thực hiện tương tự cho các VLAN còn lại.

---

### 5. Cấu hình VLAN - Trunk - Access trên Switch

#### a. Tạo VLAN và gán Access Port

Ví dụ: gán VLAN 30 cho cổng Gi0/0

```bash
Switch# configure terminal
Switch(config)# vlan 30
Switch(config-vlan)# exit
Switch(config)# interface Gi0/0
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 30
```

 <div align="center"> <img src="/images/vlan.webp" alt="Switch VLAN Configuration" width="80%"/> </div> 

#### b. Cấu hình Trunk Port giữa Switch và pfSense

+ Trunk đường kết nối Switch ⇔ pfSense để truyền nhiều VLAN qua cùng một link.
  
```bash
Switch# configure terminal
Switch(config)# interface Gi0/1
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
```
 <div align="center"> <img src="/images/switch_config.webp" alt="Switch Trunk Configuration" width="80%"/> </div> 

### 6. Kiểm tra kết quả

+ Vào máy ảo hoặc VPC, kiểm tra xem có nhận IP từ DHCP Server không.

 <div align="center"> <img src="/images/dhcp_allocate_ip.webp" alt="DHCP Allocate IP" width="80%"/> </div> 

## Hoàn thành!

Giờ đây các thiết bị thuộc từng VLAN đã tự động nhận IP từ pfSense qua DHCP Server 🎉.
