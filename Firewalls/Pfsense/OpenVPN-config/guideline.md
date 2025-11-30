# OpenVPN với pfSense

## Mục tiêu

Cấu hình VPN truy cập từ xa bằng **OpenVPN trên pfSense** nhằm tạo kết nối bảo mật từ máy client bên ngoài vào hệ thống mạng nội bộ.

---

## Sơ đồ tổng quan

<div align="center">
  <img src="/images/model.webp" alt="Network Topology" width="100%"/>
</div>

---

## Topology Lab

<div align="center">
  <img src="/images/topology_openvpn.webp" alt="Topology OpenVPN" width="100%"/>
</div>

---Bạn đã cấu hình thành công OpenVPN truy cập từ xa trên pfSense. Từ client bên ngoài, bạn có thể truy cập vào mạng nội bộ một cách bảo mật và an toàn qua VPN tunnel.

## Yêu cầu chuẩn bị

- Đã hoàn thành cấu hình cơ bản pfSense (`Basic Config`)
- Đã truy cập được vào Web GUI của pfSense
- Có topology lab như hình

---

## Các bước cấu hình OpenVPN

<div align="center">
  <img src="/images/step_openvpn.webp" alt="OpenVPN Steps" width="100%"/>
</div>

---

### Bước 1: Tạo Certificate Authority (CA)

Vào menu:  
`System > Cert. Manager > CAs`

<div align="center">
  <img src="/images/create_CA.webp" width="100%"/>
  <img src="/images/CA_step1.webp" width="100%"/>
  <img src="/images/CA_step_2.webp" width="100%"/>
</div>

---

### Bước 2: Tạo chứng chỉ Server và User

#### 2.1 Server Certificate

<div align="center">
  <img src="/images/CEA_Server.webp" width="100%"/>
  <img src="/images/CEA_server_1.webp" width="100%"/>
  <img src="/images/CEA_server_2.webp" width="100%"/>
  <img src="/images/CEA_server_3.webp" width="100%"/>
</div>

#### 2.2 User Certificate

<div align="center">
  <img src="/images/CEA_user_1.webp" width="100%"/>
  <img src="/images/CEA_user_2.webp" width="100%"/>
  <img src="/images/CEA_user_3.webp" width="100%"/>
</div>

---

### Bước 3: Cài đặt Plugin OpenVPN Export

Vào:  
`System > Package Manager > Available Packages`  
Tìm và cài **openvpn-client-export**

<div align="center">
  <img src="/images/open_vpn_install.webp" width="100%"/>
</div>

---

#### Gặp lỗi khi cài đặt?

Một số hệ thống gặp lỗi sau khi cài đặt:

<div align="center">
  <img src="/images/error.webp" width="100%"/>
</div>

##### Cách khắc phục lỗi:

Chạy lệnh trong pfSense Shell:

```bash
pkg
pkg upgrade
pkg-static -d update
env ASSUME_ALWAYS_YES=yes pkg-static bootstrap -f
pkg-static update -f
certctl rehash
```

### Bước 4: Cấu hình OpenVPN Server

#### 4.1 Sử dụng Wizard

Vào: `VPN > OpenVPN > Wizards`

<div align="center"> <img src="/images/openvpn_config.webp" width="100%"/> </div>

#### 4.2 Xem lại cấu hình ở phần Servers

Vào: ` VPN > OpenVPN > Servers`

<div align="center"> <img src="/images/openvpn_config2.webp" width="100%"/> </div>

> Lưu ý 2 mục quan trọng:

- IPv4 Tunnel Network: mạng ảo dành cho client VPN
  Ví dụ: 10.0.8.0/24

- IPv4 Local Network: mạng nội bộ thật phía sau pfSense
  Ví dụ: 192.168.74.0/24

 <div align="center"> <img src="/images/open_vpn_final.webp" width="100%"/> </div>

### Bước 5: Xuất file cấu hình cho client

Vào: `VPN > OpenVPN > Client Export`

Chọn cấu hình Bundled Configuration phù hợp với hệ điều hành (Linux) (Vì tôi dùng linux).

  <div align="center"> <img src="/images/export_config.webp" width="100%"/> </div>

## Kiểm tra kết nối từ phía Client

Trên Linux, Sau khi tải về và giải nén, bạn sẽ có 2 file:

 <div align="center"> <img src="/images/2files.webp" width="100%"/> </div>

```bash
 sudo openvpn --config pfSense-UDP4-1194-vpn01.ovpn
```

Kết quả kết nối thành công:

  <div align="center"> <img src="/images/success.webp" width="100%"/> </div>

Kiểm tra trạng thái trên pfSense

<div align="center"> <img src="/images/congratulation.webp" width="100%"/> </div>

## Kết luận

Bạn đã cấu hình thành công OpenVPN truy cập từ xa trên pfSense. Từ client bên ngoài, bạn có thể truy cập vào mạng nội bộ một cách bảo mật và an toàn qua VPN tunnel.
