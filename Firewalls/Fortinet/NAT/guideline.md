# Cấu hình NAT trên NGFW:

SNAT (cho phép các thiết bị truy cập intetnet từ bên trong bao gồm PAT và Static NAT)

- Source Network Address Translation (SNAT) là quá trình thay đổi địa chỉ IP nguồn của gói tin khi gói tin đi từ mạng nội bộ ra ngoài Internet.
- Mục đích:
  - Cho phép thiết bị nội bộ dùng IP private truy cập Internet
  - Che giấu IP nội bộ
  - Tiết kiệm IP public

Các loại SNAT
1. PAT (Port Address Translation) : Port Address Translation là dạng NAT phổ biến nhất.
    - Nhiều IP private dùng 1 IP public
    - Phân biệt bằng port
2. Cấu hình PAT

Đầu tiên vào Policy & Object > Firewall Policy > Create New

<div align="center">

  <img src="/images/image141.png" alt="Network Topology" width="100%"/>

</div>

Sau đó nhập vào các mục như hình, cái này mình sẽ sử dụng tiếp tục cấu hình bên basic-config làm tiếp nha

<div align="center">

  <img src="/images/image142.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image144.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image143.png" alt="Network Topology" width="100%"/>

</div>

Sau đó tiến hành thực hiện ping kiểm tra
<div align="center">

  <img src="/images/image145.png" alt="Network Topology" width="100%"/>

</div>
<div align="center">

  <img src="/images/image146.png" alt="Network Topology" width="100%"/>

</div>

3. Static NAT (Outbound) : Static NAT là ánh xạ 1 IP private ↔ 1 IP public cố định.
4. Cấu hình Static NAT
Chúng ta vào Policy & Object > IP Pools để tạo Pool

<div align="center">

  <img src="/images/image147.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image148.png" alt="Network Topology" width="100%"/>

</div>

Sau đó gắn nó vào policy và test

<div align="center">

  <img src="/images/image149.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image150.png" alt="Network Topology" width="100%"/>

</div>

oke vậy là vẫn ping thành công.

DNAT (Destination NAT) 
- Destination Network Address Translation (DNAT) là quá trình thay đổi địa chỉ IP đích của gói tin khi traffic đi từ Internet vào mạng nội bộ.
Mục đích
- Cho phép thiết bị bên ngoài truy cập server nội bộ

Các lọai DNAT

1. Port Forwarding
Port Forwarding là chuyển port từ IP public vào server nội bộ.
2. Cấu hình
Đầu tiên vào Policy & Object > Virtual IPs

<div align="center">

  <img src="/images/image151.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image152.png" alt="Network Topology" width="100%"/>

</div>

3. Static NAT (Inbound)
Trường hợp 1 IP public map trực tiếp vào 1 server nội bộ.
4. Cấu hình
Cũng vào như trên nhưng config khác
<div align="center">

  <img src="/images/image153.png" alt="Network Topology" width="100%"/>

</div>
