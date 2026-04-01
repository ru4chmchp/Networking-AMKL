# Cấu hình dự phòng NGFW theo cơ chế: Active/Active

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
    - Lưu ý kết nối: Active-Active (hoặc > 2 thiết bị): Phải kết nối thông qua một Switch Layer 2.

## Hướng dẫn cấu hình HA Active - ACtive

Hoạt động:
- Tất cả các thiết bị trong cụm (Primary và Secondary) đều xử lý lưu lượng mạng đồng thời.
- Primary có nhiệm vụ cân bằng tải (Load Balancing) và phân phối lưu lượng giữa các thành viên.
- Ưu điểm: Tận dụng được sức mạnh của tất cả các thiết bị.


<div align="center">
  <img src="/images/image301.png" alt="Network Topology" width="100%"/>
</div>

Đầu tiên vào 10.120.170.241 của Fortigate-C cấu hình

<div align="center">
  <img src="/images/image302.png" alt="Network Topology" width="100%"/>
</div>

Chọn mode Active-Active, cấu hình như sauu 

<div align="center">
  <img src="/images/image303.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến ta vào 10.120.170.242 của Fortigate-D cấu hìnhD

nhưng lưu ý đặt Priority cao nhất tầm 200 để nó làm Primary

<div align="center">
  <img src="/images/image304.png" alt="Network Topology" width="100%"/>
</div>

Và node cuối cùng cũng như vậy nhưng priority cao hơn node đầu tiên 

<div align="center">
  <img src="/images/image305.png" alt="Network Topology" width="100%"/>
</div>

Và đợi một chút để nó sync

<div align="center">
  <img src="/images/image306.png" alt="Network Topology" width="100%"/>
</div>

Sau một khoản thời gian

<div align="center">
  <img src="/images/image307.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image308.png" alt="Network Topology" width="100%"/>
</div>

Done
