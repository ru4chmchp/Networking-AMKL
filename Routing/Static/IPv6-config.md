# IPv6 config để ra ngoài network từ trong PNETLab

---

## Các bước chuẩn bị

---

### Thứ 1: Tạo thêm card mạng & Cloud

- Tạo thêm **một card mạng** để lấy IPv6 từ bên ngoài.
- Để chế độ **Bridge** để tự động nhận IPv6 thông qua auto-config.

<div align="center">
  <img src="/images/config_ipv6_1.webp" alt="Network Topology" width="100%"/>
</div>

Tiếp theo, vào trong Lab của **PNETLab**, tạo thêm một **cloud mới**:

<div align="center">
  <img src="/images/IPv6_config_2.webp" alt="Network Topology" width="100%"/>
</div>

> **Nhớ rằng:**
>
> - Card mạng đầu tiên (cloud0) gắn để NAT IPv4 cố định (PNETLab Web GUI cần IPv4).
> - Muốn dùng IPv6 phải tạo thêm card thứ hai → `cloud1` (không phải NAT hay Bridge gì nữa, chỉ cần Cloud).
> - Muốn thêm nữa → tiếp tục `cloud2`, `cloud3`, v.v.

---

### Thứ 2: Triển khai IPv6

Có **3 cách cấp phát địa chỉ IPv6**:

- Thủ công
- SLAAC (Stateless Address Autoconfiguration)
- DHCPv6

Nhưng trước tiên cần bật tính năng định tuyến IPv6 trên router:

```plaintext
Router(config)# ipv6 unicast-routing
```

> Đây là lệnh bắt buộc, vì:
> Kích hoạt chức năng định tuyến IPv6.
> Xây dựng bảng định tuyến, bật Router Advertisement (RA).
> Nếu thiếu, router không chuyển tiếp được gói IPv6, các giao thức định tuyến không hoạt động.

#### Cấu hình thủ công

```plaintext
Router(config)# interface GigabitEthernet3
Router(config-if)# ipv6 address 2001:DB8:AABB:1::1/64
Router(config-if)# no shutdown
```

> IPv6 dài 128 bit, chia thành 8 nhóm, mỗi nhóm 4 ký tự hex, cách nhau dấu :.
> Có thể rút gọn bằng bỏ số 0 đầu hoặc chuỗi liên tiếp nhóm 0 → :: (chỉ dùng một lần).
> Ví dụ: 2001:0DB8:AABB:0001:0000:0000:0001/64 → 2001:DB8:AABB:1::1/64.

#### SLAAC (Stateless Address Autoconfiguration)

Cho phép host tự tạo địa chỉ IPv6 Global Unicast dựa vào Prefix nhận được từ RA và EUI-64/Random:

```plaintext
Router(config)# interface Gi3
Router(config-if)# ipv6 address autoconfig
Router(config-if)# ipv6 enable
Router(config-if)# no shutdown
```

 <div align="center"> <img src="/images/config_ipv6_3.webp" alt="SLAAC Config" width="100%"/> </div>

> Host sẽ gửi Router Solicitation (RS), nhận Router Advertisement (RA) và tạo IPv6 GUA.
> Đồng thời, tự sinh ra địa chỉ Link-Local.

#### DHCPv6

Giống DHCP v4, cần DHCPv6 Server để cấp phát:

Router(config)# interface GigabitEthernet3
Router(config-if)# ipv6 address dhcp
Router(config-if)# ipv6 enable
Router(config-if)# no shutdown

#### So sánh các phương pháp

| Đặc điểm           | SLAAC (Stateless)                   | Stateful DHCPv6                     | Stateless DHCPv6                      |
| ------------------ | ----------------------------------- | ----------------------------------- | ------------------------------------- |
| Cần Server?        | Không                            | Cần                              | Cần (chỉ cung cấp thông tin DNS)   |
| Cấp IP Address?    | Tự tạo từ Prefix + EUI-64/Random | DHCPv6 Server cấp phát & quản lý | Host tự tạo, server chỉ cấp DNS    |
| Quản lý trạng thái | Không                            | Có                               | Không                              |
| Cung cấp DNS?      | Không (cần RDNSS/O-flag+DHCPv6)  | Có                               | Có                                 |
| Kiểm soát địa chỉ  | Ít kiểm soát                        | Kiểm soát chặt chẽ, dễ quản lý      | Địa chỉ IP do host, DNS từ server     |
| Độ phức tạp        | Đơn giản                            | Phức tạp hơn                        | Trung bình                            |
| M-flag (RA)        | 0                                   | 1                                   | 0                                     |
| O-flag (RA)        | 0 hoặc 1                            | 1                                   | 1                                     |
| Ứng dụng           | Mạng lớn, IoT, khách, di động       | Doanh nghiệp, Data Center, server   | Mạng gia đình, văn phòng nhỏ, kết hợp |

### Thứ 3: Định tuyến IPv6

Có 2 cách:

    Qua địa chỉ Next-Hop IPv6 Global Unicast

    Qua địa chỉ Link-Local + Interface (thường dùng hơn)

#### Cách 1: Next-Hop GUA

```plaintext
ipv6 route <destination>/<prefix> <next_hop_GUA>

Router(config)# ipv6 route 2001:DB8:CAFE::/64 2001:DB8:ABCD:1::2

```

#### Cách 2: Next-Hop Link-Local + Interface

```plaintext
ipv6 route <destination>/<prefix> <interface> <next_hop_Link-Local>

Router(config)# ipv6 route ::/0 GigabitEthernet3 fe80::21b:17ff:fe00:1412

```

> Dùng lệnh ip -6 route để xem Link-Local next hop.
> Chọn đúng next hop (thường là modem hoặc router upstream).

<div align="center"> <img src="/images/config_ipv6_4.webp" alt="Next Hop Link-Local" width="100%"/> </div>

- Dòng đầu thường là local next hop của router.

- Dòng thứ hai thường là Link-Local của modem phát wifi.

- Lấy Link-Local modem làm next hop để ra Internet.
