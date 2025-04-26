# 🛠️ Basic Configuration to Ping Over the Network and Join Interface pfSense

<div align="center">

  <img src="/Firewall/images/topology_example1.png" alt="Network Topology" width="100%"/>

</div>

## 🎯 Mục tiêu

Cấu hình router để có thể ping ra ngoài Internet bằng địa chỉ `8.8.8.8` và chuẩn bị kết nối pfSense.

---

## 🧠 Tổng quan mô hình mạng

Mô hình mạng gồm hai dải IP:

- `10.10.10.0/24`
- `192.168.10.0/24`

<div align="center">

  <img src="/Firewall/images/topology_example1.png" alt="Network Topology" width="100%"/>

</div>

---

## ⚙️ Cấu hình router để kết nối Internet

Để router có thể ra ngoài mạng, ta cần cấu hình định tuyến tĩnh. Nhưng trước hết, cần xác định:

1. **Subnet IP** của máy thật đang sử dụng (ví dụ: `172.16.214.0/24`).
2. **Default Gateway** của card mạng VMware dạng NAT.

<div align="center">

  <img src="/Firewall/images/subnet.png" alt="Subnet IP Info" width="100%"/>
  <br/>
  <img src="/Firewall/images/default_gateway.png" alt="Default Gateway NAT" width="100%"/>

</div>

> ✅ Ví dụ: Default gateway là `172.16.214.2`.

---

## 🧩 Cấu hình interface router

Đặt địa chỉ IP cho cổng `Gig0/0` của router sao cho cùng lớp mạng với subnet NAT:

```bash
Router(config)# interface gig0/0
Router(config-if)# ip address 172.16.214.135 255.255.255.0
Router(config-if)# no shutdown
```

## 📌 Thiết lập định tuyến tĩnh

Câu lệnh định tuyến tĩnh để router ra ngoài mạng:

```bash
Router(config)# ip route 0.0.0.0 0.0.0.0 172.16.214.2
```

+ 0.0.0.0 0.0.0.0: đại diện cho mọi mạng.

+ 172.16.214.2: default gateway (NAT) mà router sẽ chuyển tiếp.

## ✅ Kiểm tra kết nối

Dùng lệnh sau để kiểm tra:

```bash
Router# ping 8.8.8.8
```

Nếu ping thành công, cấu hình đã đúng. Nếu không:

+ Kiểm tra IP trên interface

+ Kiểm tra default gateway

+ Kiểm tra kết nối mạng máy ảo

> 💡 Gợi ý: Nên dùng chế độ NAT để tránh xung đột mạng và dễ cấu hình hơn trong môi trường lab.

## 🛡️ Cấu hình cơ bản pfSense Firewall và truy cập Web Interface

### 🚀 Khởi động pfSense và cấu hình Interface WAN, LAN

<div align="center"> <img src="/Firewall/images/WAN_LAN_interface.png" alt="WAN and LAN Interface" width="100%"/> </div>

+ Gán IP cho WAN và LAN theo hướng dẫn trong file PDF.

+ LAN: Interface để truy cập giao diện Web pfSense.

+ WAN: Interface hướng ra ngoài Internet.

> ⚡ Lưu ý: Không thể truy cập giao diện Web pfSense qua cổng WAN vì để đảm bảo an toàn bảo mật, tránh bị hacker tấn công (bruteforce, exploit, ...)nên pfsense đã chặn truy cập web interface qua WAN interface.

### 🧹 Xử lý ban đầu khi chưa có Gateway

Sau khi gán IP cho LAN và WAN, pfSense chưa thể ping ra ngoài vì:

+ Chưa thiết lập gateway.

+ pfSense mặc định chặn các gói tin từ ngoài vào (Firewall Rules).

+ Chưa cấu hình Rule cho phép truy cập.

✅ Để xử lý tạm thời:

1. Vào shell pfSense (chọn Option 8 tại terminal).
  
2. Tắt firewall pfSense tạm thời:

```bash
pfctl -d
```

3. Gán Gateway tạm thời:

```bash
route add default 10.10.10.1
```
+ 10.10.10.1 là địa chỉ IP của cổng Gi0/1 trên router.

>🔥 Đây chỉ là gán gateway tạm thời. Gán mặc định vĩnh viễn cần thực hiện qua Web Interface sau này.

### 🧪 Kiểm tra kết nối từ pfSense

Tại shell pfSense:

```bash
ping 8.8.8.8
```
+ Nếu ping thành công → pfSense đã ra Internet.

## 🌉 Cấu hình định tuyến để truy cập Web Interface pfSense từ ngoài

1. Định tuyến tĩnh trên Router

Định tuyến tới mạng 192.168.10.0/24 từ router:

```bash
Router(config)# ip route 192.168.10.0 255.255.255.0 10.10.10.2
```

2. Định tuyến tĩnh trên máy thật (Windows) nếu dùng

Mở PowerShell với quyền Administrator, chạy:

```bash
New-NetRoute -DestinationPrefix "10.10.10.0/24" -InterfaceIndex 12 -NextHop 172.16.214.135
New-NetRoute -DestinationPrefix "192.168.10.0/24" -InterfaceIndex 12 -NextHop 172.16.214.135
```
Giải thích:

+ `DestinationPrefix`: Mạng muốn định tuyến tới.

+ `InterfaceIndex`: Card mạng sử dụng NAT (xem bằng Get-NetIPInterface).

+ `NextHop`: IP router (172.16.214.135).

3. Định tuyến tĩnh trên máy thật (Linux) nếu dùng

Mở terminal với quyền root hoặc thêm sudo trước mỗi lệnh, rồi chạy:

```bash
# Định tuyến tới mạng 10.10.10.0/24
sudo ip route add 10.10.10.0/24 via 172.16.214.135 

# Định tuyến tới mạng 192.168.10.0/24
sudo ip route add 192.168.10.0/24 via 172.16.214.135
```

> ⚡ Lưu ý:

+ Phải định tuyến tới mạng 10.10.10.0/24 trước, rồi mới tới 192.168.10.0/24.

+ Nếu mô hình mạng khác, cần chỉnh lại địa chỉ IP, tên interface cho phù hợp.

### 🌐 Truy cập giao diện Web pfSense

+ Mở trình duyệt → nhập IP LAN của pfSense để truy cập WebGUI.

> Nếu không truy cập được, kiểm tra lại định tuyến, IP, firewall,...