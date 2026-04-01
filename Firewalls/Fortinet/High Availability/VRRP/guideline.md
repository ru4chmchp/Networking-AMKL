# Cấu hình VRRPP

VRRP : Virtual Router Redundancy Protocol (VRRP) là một giao thức dùng để tạo gateway dự phòng cho mạng LAN. Nó cho phép nhiều router hoặc firewall chia sẻ cùng một địa chỉ IP ảo (Virtual IP) để tránh mất kết nối khi thiết bị chính bị lỗi


<div align="center">
  <img src="/images/image309.png" alt="Network Topology" width="100%"/>
</div>

## Tiến hành cấu hình VRRP

Các bước cấu hình Interface đã làm nhiều nên mình sẽ không làm lại, giờ chi tập chung vào cấu hình VRRP

Đầu tiên ta vào terminal Fortigate-G

<div align="center">
  <img src="/images/image310.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo ta vào Fortigate-F cấu hình tương tự nhưng priority thấp hơn làm backup

<div align="center">
  <img src="/images/image311.png" alt="Network Topology" width="100%"/>
</div>

Sau đó check config

<div align="center">
  <img src="/images/image312.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image313.png" alt="Network Topology" width="100%"/>
</div>

Check ping 

<div align="center">
  <img src="/images/image314.png" alt="Network Topology" width="100%"/>
</div>

Sau đó ngắt 1 dây, tiếng hành ping kiểm tra lại

<div align="center">
  <img src="/images/image315.png" alt="Network Topology" width="100%"/>
</div>

Tiến hành check config

<div align="center">
  <img src="/images/image316.png" alt="Network Topology" width="100%"/>
</div>

Done