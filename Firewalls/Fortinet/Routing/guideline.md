# Cấu hình Routing Static route, OSPF và Gateway trên thiết bị NGFW Fortigate

## Static Route (Định tuyến tĩnh)

> **Định nghĩa** : Static Route là cách bạn cấu hình thủ công đường đi của gói tin trên FortiGate. Bạn nói với firewall: “Muốn đi tới mạng A thì hãy gửi qua gateway B”

- Đặc điểm:
  - Admin tự khai báo (không tự học)
  - Không thay đổi nếu mạng có sự cố
  - Đơn giản, dễ kiểm soát
  - Phù hợp mạng nhỏ hoặc route cố định

<div align="center">
  <img src="/images/image155.png" alt="Network Topology" width="100%"/>
</div>

Đầu tiên chúng ta sẽ vào CLI của Firewall và cấu hình IP như sau 

<div align="center">
  <img src="/images/image156.png" alt="Network Topology" width="100%"/>
</div>

Sau đó chúng ta sẽ kiểm tra thử IP đã được cấu hình chưa :

<div align="center">
  <img src="/images/image157.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến cấu hình static route cho phép lưu lượng ra ngoài Internet

<div align="center">
  <img src="/images/image158.png" alt="Network Topology" width="100%"/>
</div>

Và sau đó tiến hành kiểm tra bằng lệnh ping : 

<div align="center">
  <img src="/images/image159.png" alt="Network Topology" width="100%"/>
</div>

Vậy là đã config thành công static route cho phép lưu lượng ra ngoài internet, ta tiến hành vào GUI check thử.

<div align="center">
  <img src="/images/image160.png" alt="Network Topology" width="100%"/>
</div>

Sau khi đăng nhập vào giao diện :

<div align="center">
  <img src="/images/image161.png" alt="Network Topology" width="100%"/>
</div>

Ta tiếp tục vào Network > Static Routes

<div align="center">
  <img src="/images/image162.png" alt="Network Topology" width="100%"/>
</div>

Vậy là thành công

## Gateway (Default Gateway)

> **Định nghĩa** : Gateway là địa chỉ IP của thiết bị trung gian mà FortiGate (hoặc client) sẽ gửi traffic tới khi: Không biết đường đi cụ thể, hoặc đi ra Interneth.

Hiểu đơn giản: Gateway = “cổng ra ngoài”

Đặc điểm:

- Thường là router hoặc modem ISP
- Là route mặc định (0.0.0.0/0)
- Bắt buộc để đi Internet

Ở phần trên ta đã cấu hình phần Gateway trong phần Static Routes nên sẽ không cấu hình tiếp.

## OSPF (Open Shortest Path First)

> **Định nghĩa** : OSPF là một giao thức định tuyến động (Dynamic Routing Protocol) giúp các thiết bị như FortiGate tự động trao đổi thông tin route với nhau.

Hiểu đơn giản: Các router “nói chuyện với nhau” để biết đường đi tốt nhất

Đặc điểm:
- Tự động học route
- Tự cập nhật khi mạng thay đổi
- Dựa trên thuƯật toán SPF (Shortest Path First)
- Phù hợp mạng lớn, nhiều thiết bị

Ví dụ thực tế:

- FortiGate A kết nối FortiGate B. Khi B có mạng mới → A tự biết mà không cần config thêm
- Giảm công sức quản trị rất nhiều so với static route



