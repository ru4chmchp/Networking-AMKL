# Port-forwarding

## Mục tiêu

Mục tiêu là **truy cập từ WAN interface** với IP WAN `10.10.10.2:8080`  
→ vào **LAN interface** IP `192.168.1.1:80` thông qua port forwarding.

Thao tác khá đơn giản, chỉ cần vài bước cấu hình.

---

## Topology Lab

<div align="center">
  <img src="/images/topology_portforwarding.webp" alt="Topology Port Forwarding" width="100%"/>
</div>

---

## Các bước thực hiện

### 1️. Tạo Port Forwarding Rule

Vào:

```plaintext
Firewall > NAT > Port Forward
```

 <div align="center"> <img src="/images/port_forwarding_config.webp" alt="Port Forwarding Config" width="100%"/> </div>

- Interface: WAN

- Destination: WAN address

- Redirect target IP: 192.168.1.1 (địa chỉ LAN)

- Redirect target port: HTTP (80)

> Vì chúng ta muốn người ngoài truy cập thông qua IP WAN, nên interface để là WAN.
> Port cần forward là 8080 từ WAN → về 80 của LAN.

### 2️. Kiểm tra Rule tự động sinh ra

Sau khi lưu xong, vào:

```plaintext
Firewall > Rules > WAN
```

 <div align="center"> <img src="/images/rules_wan_port_forwarding.webp" alt="Rules WAN Port Forwarding" width="100%"/> </div>

- Hệ thống sẽ tự động thêm một rule để cho phép traffic đi qua port forwarding.

Vậy là hoàn tất!

Giờ bạn có thể truy cập:

```plaintext
http://10.10.10.2:8080
```

và nó sẽ tự forward về:

```plaintext
http://192.168.1.1:80
```
