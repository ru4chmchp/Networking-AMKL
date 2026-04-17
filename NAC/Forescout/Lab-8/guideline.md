# Hướng dẫn phân vùng Segment và tạo thư mục Policy​

Khi triển khai Forescout trong các hệ thống mạng lớn, việc tổ chức cấu trúc chính sách (Policy) một cách khoa học là rất quan trọng để đảm bảo dễ quản lý, dễ mở rộng và giảm rủi ro sai lệch trong quá trình kiểm soát truy cập. Một trong những bước đầu tiên cần thực hiện là phân vùng mạng (Segment) và tạo thư mục chính sách (Policy Folders).
Bài lab này sẽ hướng dẫn bạn:

- Cách tạo và quản lý các Segment trong Forescout
- Cách tạo thư mục và sắp xếp các policy logic, tối ưu cho việc vận hành và mở rộng

Đây là bước khởi đầu quan trọng trước khi xây dựng các chính sách kiểm soát truy cập chi tiết và triển khai quy mô lớn.

## Mục lục:
1. Khái niệm​
    1.1 Segment​
    1.2 Policy Folder​
2. Cấu hình phân vùng​
    2.1 Tạo Segment​
    2.2 Tạo Folder Policy và cấu hình Policy ​
3. Kết luận​

## 1. Khái niệm​

### 1.1 Segment

Segment là cách chia hệ thống mạng thành các khu vực logic dựa trên địa chỉ IP, thiết bị, hoặc vị trí triển khai thực tế (như văn phòng, chi nhánh, trung tâm dữ liệu...). Việc phân vùng giúp xác định rõ phạm vi kiểm soát và áp dụng chính sách theo từng khu vực cụ thể.

Ví dụ 1 segment VLAN MGMT 10.10.10.0/24

<div align="center">
  <img src="/images/image479.png" alt="Network Topology" width="100%"/>
</div>

### 1.2 Policy Folder

Policy Folder là nơi nhóm các chính sách theo từng mục đích quản lý – ví dụ: phát hiện thiết bị, kiểm tra tuân thủ, xử lý vi phạm, phân nhóm truy cập... Việc tạo thư mục chính sách giúp cấu trúc chính sách rõ ràng, tránh xung đột và tăng khả năng kiểm soát.


Ví dụ về 1 Policy folder Lau3 áp dụng vào Segment VLAN_MGMT

<div align="center">
  <img src="/images/image480.png" alt="Network Topology" width="100%"/>
</div>

## 2. Cấu hình phân vùng​

### 2.1 Tạo Segment​

Vào Tool -> Segment Manager -> Nhấn vào dấu + -> Nhập tên Segment

<div align="center">
  <img src="/images/image481.png" alt="Network Topology" width="100%"/>
</div>

Sau khi tao xong chúng ta thêm IP Range vào segment. Nhấn Add -> Thêm IP range vào Segment đó -> Nhấn OK để hoàn tất

<div align="center">
  <img src="/images/image483.png" alt="Network Topology" width="100%"/>
</div>

### 2.2 Tạo Folder Policy và cấu hình Policy

Sau khi cấu hình xong nó sẽ chưa thể phân loại được chúng ta phải tạo thêm 1 vài policy cho nó phân loại và kiểm tra tuân thủ thiết bị

<div align="center">
  <img src="/images/image484.png" alt="Network Topology" width="100%"/>
</div>

Nhấn Add để mở Policy Wizard. Có rất nhiều template hiện ra để áp dụng hoặc mình có thể tự custom. Giao diện như sau

<div align="center">
  <img src="/images/image485.png" alt="Network Topology" width="100%"/>
</div>

Ở đây mình sẽ chọn template phân loại Primary Classification. Cụ thể Chính sách Primary Classification trong Forescout giúp phân loại thiết bị trong mạng thành các nhóm cụ thể như máy in, thiết bị VoIP, Windows, Linux... để quản lý hiệu quả hơn. Nếu thiết bị không khớp với nhóm nào, nó sẽ vào Unclassified, có thể được phân loại thủ công -> Nhấn next

<div align="center">
  <img src="/images/image486.png" alt="Network Topology" width="100%"/>
</div>

Đặt tên cho Policy (Ở đây mình để mặc định luôn)

<div align="center">
  <img src="/images/image487.png" alt="Network Topology" width="100%"/>
</div>

Chọn segment mình sẽ quản lý. Có thể chọn tất cả hoặc các segment mình tự tạo

<div align="center">
  <img src="/images/image488.png" alt="Network Topology" width="100%"/>
</div>

Cuối cùng bạn sẽ thấy các thiết bị được phân loại nằm ở Sub-rules -> Nhấn finish -> Nhấn Apply để lưu policy

<div align="center">
  <img src="/images/image489.png" alt="Network Topology" width="100%"/>
</div>

Quay trở lại với Home chúng ta thấy được Segment đã được phân loại

<div align="center">
  <img src="/images/image490.png" alt="Network Topology" width="100%"/>
</div>

## 3. Kết luận​
- Việc phân vùng mạng (Segment) và tổ chức thư mục chính sách (Policy Folder) là bước nền tảng quan trọng giúp hệ thống Forescout hoạt động hiệu quả, dễ quản lý và có khả năng mở rộng linh hoạt theo quy mô doanh nghiệp.
- Thông qua phân vùng mạng, người quản trị có thể áp dụng các chính sách phù hợp với từng khu vực, chi nhánh hoặc nhóm thiết bị cụ thể, thay vì áp dụng một cách dàn trải và thiếu kiểm soát. Điều này giúp tăng cường độ chính xác và giảm thiểu sai sót trong quá trình kiểm soát truy cập.
- Bên cạnh đó, việc tạo thư mục chính sách theo chức năng hoặc khu vực giúp bạn dễ dàng theo dõi, chỉnh sửa, phân quyền và bảo trì hệ thống chính sách theo thời gian. Đây là yếu tố quan trọng trong các hệ thống lớn với hàng trăm đến hàng nghìn policy đang vận hành song song.
- Với cấu trúc hợp lý ngay từ đầu, bạn sẽ tiết kiệm được rất nhiều thời gian khi mở rộng triển khai, xử lý sự cố hoặc đánh giá hệ thống bảo mật theo định kỳ.