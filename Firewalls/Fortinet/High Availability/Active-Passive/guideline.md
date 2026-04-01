# Cấu hình dự phòng NGFW theo cơ chế: Active/Passive

## High Availability (HA) là gì?​
Định nghĩa: HA là tính năng dự phòng giúp mạng lưới của bạn không bị gián đoạn (giảm thiểu downtime) khi thiết bị FortiGate gặp sự cố.
Tầm quan trọng: Trong mạng lưới, FortiGate là một điểm lỗi đơn lẻ (Single Point of Failure). Nếu nó hỏng (mất điện, lỗi vật lý, tấn công), mọi hoạt động kinh doanh, truy cập mạng sẽ dừng lại, gây thiệt hại lớn.
Giải pháp: Cấu hình HA là ghép nhiều thiết bị FortiGate (từ 2 đến 4 thiết bị) thành một cụm (Cluster). Khi một thiết bị lỗi, các thiết bị còn lại sẽ tự động tiếp quản (Failover) việc xử lý lưu lượng, đảm bảo hệ thống luôn hoạt động.
## Điều kiện cần để cấu hình HA​
- Để cấu hình HA thành công, các thiết bị FortiGate phải đáp ứng các yêu cầu sau:
- Đồng bộ Thiết bị:
    - Phải cùng dòng phần cứng (Model) và cùng phiên bản hệ điều hành FortiOS.
    - Phải có đầy đủ License cho tất cả các thiết bị trong cụm.
- Kết nối Dự phòng (Cáp mạng):
    - Mạng phải được thiết kế sao cho đường truyền vật lý vẫn hoạt động và có thể chuyển hướng lưu lượng đến thiết bị dự phòng khi thiết bị chính lỗi.
- Cổng Heartbeat (Nhịp tim):
    - Phải có ít nhất một cổng vật lý được sử dụng để kết nối các thiết bị HA lại với nhau. Cổng này dùng để gửi/nhận thông tin đồng bộ cấu hình và tín hiệu "tôi còn sống" (Heartbeat).
    - Cổng Heartbeat không được gán địa chỉ IP (tĩnh, DHCP, hay PPPoE).
    - Lưu ý kết nối:
    - Active-Passive: Có thể kết nối trực tiếp hai thiết bị bằng cáp chéo.

## Cấu hình HA Active - Passive

<div align="center">

  <img src="/images/image183.png" alt="Network Topology" width="100%"/>

</div>

Đây là sơ đồ cấu hình HA, đầu tiên ta sẽ vào 10.120.170.238 để cấu hình nó làm node Active.

Sau đó vào System > HA và cấu hình như sau : 
- Mode: Active-Passive 
    - 1 con dự phòng, 1 con chạy, khi active chết thì passive lên thay.
- Device priority: 140
    - Độ ưu tiên để chọn node nào làm active.
- Group name : HA
    - Tên cluster HA, 2 thiết bị phải đặt giống nhau, nếu k thì sẽ k join cluster đc.
- Password : xác thực join Cluster
- Monitor Interfaces: port1
    - nếu Wan chết thì chuyeenr sang con còn lại.
- Heartbeat Interfaces: port4, port5
    - 2 firewall gửi tín hiệu sống (heartbeat) cho nhau.
- Heartbeat Interface Priority : 
    - Độ ưu tiên của từng link heartbeat


<div align="center">

  <img src="/images/image184.png" alt="Network Topology" width="100%"/>

</div>

Sau đó ta vào Fortigate-B để cấu hình HA Passive cho nó
```bash
config system ha
    set mode a-p
    set group-name "HA"
    set password Sunivy111!
    set hbdev "port4" 50 "port5" 50
    set monitor port1
end
```

<div align="center">

  <img src="/images/image185.png" alt="Network Topology" width="100%"/>

</div>

Sau đó chúng ta vào GUI check

<div align="center">

  <img src="/images/image186.png" alt="Network Topology" width="100%"/>

</div>

Done nhưng còn 1 bước là vì cái này có cấu hình LACP nên phải tạo port-channel thêm cho con Passive. Nhưng trước hết thì chỉnh LACP mode trên Fortigate về active như sau : 

<div align="center">

  <img src="/images/image188.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image189.png" alt="Network Topology" width="100%"/>

</div>

Sau đó ta mới config lại Switch, vì đây là switch standalone, k phải stack nên cấu hình channel-group không được gộp vào thay vào đó phải tạo 2 group cho 2 con node active và passive

- Cấu hình channel-group 1
```bash
int range Gi0/0 - 1
    channel-group 1 mode active
int port-channel 1
    switchport trunk encapsulation dot1q
    switchport mode trunk
```

- Cấu hình channel-group 2
```bash
int range Gi1/0 - 1
    channel-group 2 mode active
int port-channel 2
    switchport trunk encapsulation dot1q
    switchport mode trunk
```

Sau đó `show etherchannel summary`

<div align="center">

  <img src="/images/image190.png" alt="Network Topology" width="100%"/>

</div>

Tiến hành ping thử 

<div align="center">

  <img src="/images/image191.png" alt="Network Topology" width="100%"/>

</div>

Sau đó ta tắt Fortigate-A đi thử HA

<div align="center">

  <img src="/images/image192.png" alt="Network Topology" width="100%"/>

</div>

Vậy là thành công

Sau đó vào giao diện System > HA check

<div align="center">

  <img src="/images/image193.png" alt="Network Topology" width="100%"/>

</div>

oke done