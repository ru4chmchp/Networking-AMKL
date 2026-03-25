# Cấu hình Object và Security Policy trên NGFW Fortigate

## 1. Cấu hình Address Object

Đây là bước định nghĩa "Ai" sẽ được áp dụng chính sách. Thay vì gõ IP thủ công vào Policy, ta tạo vật thể (Object) để dễ quản lý.

- Truy cập Policy & Objects > Addresses.
- Nhấn Create New > Address.
- Name: Nhập LAN_SUBNET (hoặc tên tùy ý dễ nhớ).
- Type: Chọn Subnet
- IP/Netmask: Nhập dải mạng (Ví dụ: 10.10.10.0/24).
- Interface: Chọn cổng nối với LAN
- Nhấn OK

<div align="center">
  <img src="/images/image163.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image164.png" alt="Network Topology" width="100%"/>
</div>

Vì cái này mình đã tạo sẵn nên chỉ xem lại.

## 2. Cấu hình Security Policy

Đây là bước tạo "luật" để cho phép hoặc chặn lưu lượng di chuyển giữa các vùng mạng.
- Truy cập Policy & Objects > Firewall Policy.
- Nhấn Create New.
- Name: Nhập Allow_LAN_to_Internet.
- Incoming Interface: Chọn cổng LAN (ví dụ: port2).
- Outgoing Interface: Chọn cổng WAN (ví dụ: port1).
- Source: Chọn Object LAN_SUBNET vừa tạo ở Bước 1.
- Destination: Chọn all (nghĩa là đi tới bất kỳ đâu trên Internet).
- Service: Chọn ALL (hoặc lọc cụ thể DNS, HTTP, HTTPS tùy yêu cầu).
- Action: Đảm bảo là Accept.
- NAT: Gạt thanh gạt sang ON (để IP nội bộ thấy được Internet).
- Nhấn OK

<div align="center">
  <img src="/images/image165.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image166.png" alt="Network Topology" width="100%"/>
</div>

Ta có topology bữa trước đã cấu hình nơi các bài trước và tiến hành check thử ping từ VPC 192.168.10.5 ra 8.8.8.8

<div align="center">
  <img src="/images/image167.png" alt="Network Topology" width="100%"/>
</div>

Sau đó vào Forward Traffic check

<div align="center">
  <img src="/images/image168.png" alt="Network Topology" width="100%"/>
</div>

- Sau khi hoàn thành bài Lab cấu hình Object và Security Policy trên FortiGate, chúng ta rút ra được các điểm mấu chốt sau:
Tầm quan trọng của Object: Việc quản lý địa chỉ thông qua Object giúp cấu hình tường lửa trở nên khoa học, dễ quản lý và giảm thiểu sai sót khi mở rộng hệ thống.
- Vai trò của Security Policy: Chính sách bảo mật là thành phần cốt lõi quyết định tính an toàn của mạng. Một Policy đúng cần đảm bảo chính xác về Source, Destination và Service.
- Cơ chế NAT & Logging: Việc bật NAT giúp máy nội bộ ra được Internet, trong khi Logging là "tai mắt" của người làm bảo mật, giúp giám sát và xử lý sự cố kịp thời qua các bản ghi traffic.
- Kết quả: Hệ thống đã hoạt động đúng thiết kế, máy Linux trong vùng LAN truy cập Internet ổn định và mọi lưu lượng đều được Firewall kiểm soát, ghi nhật ký đầy đủ.


