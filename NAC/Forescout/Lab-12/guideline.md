# Cấu hình VLAN cách ly cho thiết bị không tuân thủ trong Forescout và phân luồng truy cập cho user và nhân viên

Cấu hình VLAN cách ly cho thiết bị không tuân thủ và phân luồng truy cập giữa user và nhân viên là một trong những chức năng cốt lõi của hệ thống NAC. Thay vì chặn hoàn toàn truy cập khi phát hiện vi phạm, giải pháp hiệu quả là đưa thiết bị vào VLAN cách ly với quyền hạn giới hạn, đồng thời áp dụng các chính sách phân quyền rõ ràng giữa các nhóm người dùng như nhân viên nội bộ và user thông thường.

- Hệ thống Forescout hỗ trợ triển khai thông qua:
    - Phân tích trạng thái thiết bị (antivirus, domain join, OS version, v.v.) để xác định mức độ tuân thủ
    - Tích hợp với hạ tầng mạng (switch, wireless controller) qua SNMP, RADIUS hoặc API để tự động chuyển VLAN
    - Xây dựng policy động nhằm phân loại thiết bị theo vai trò (user, nhân viên) và trạng thái bảo mật
- Trong bài lab này, bạn sẽ thực hiện:
    - Tạo chính sách phát hiện thiết bị không tuân thủ
    - Cấu hình VLAN cách ly cho thiết bị vi phạm
    - Thiết lập phân luồng truy cập giữa user và nhân viên theo từng VLAN riêng biệt
    - Áp dụng hành động tự động chuyển VLAN khi phát hiện vi phạm
    - Kiểm tra và xác minh thiết bị được phân đúng VLAN (user, nhân viên hoặc cách ly)

Giải pháp này giúp tăng cường bảo mật mạng bằng cách cô lập các thiết bị không an toàn, đồng thời đảm bảo mỗi nhóm người dùng chỉ được truy cập vào tài nguyên phù hợp với vai trò của mình, giảm thiểu rủi ro và vẫn duy trì hoạt động ổn định của hệ thống.

## Mục lục:

1. Mô hình triển khai​
2. Cấu hình VLAN cách ly​
3. Kết luận​

## 1. Mô hình triển khai 

<div align="center">
  <img src="/images/image613.png" alt="Network Topology" width="100%"/>
</div>

## 2. Cấu hình VLAN cách ly

Vào Policy Manager -> Add Policy thuộc nhóm Complience như sau:

<div align="center">
  <img src="/images/image614.png" alt="Network Topology" width="100%"/>
</div>

​Ở đây mình chọn Template Policy Personal Firewall Comlience. Policy này sẽ giám sát và cách ly khi không bật hoặc không thể quản lý Window Firewall hoặc các loại FW khác.

<div align="center">
  <img src="/images/image615.png" alt="Network Topology" width="100%"/>
</div>

- Tới phần Condition bạn có thể thêm các điều kiện ví dụ như cách ly qua 1 VLAN khác, các bước làm như sau:
    - Nhấn vào Sub-rules bạn muốn sửa (mình chọn FW not active) -> Phần Action chỉ có các mode thông báo và phân loại -> Nhấn Add​

<div align="center">
  <img src="/images/image616.png" alt="Network Topology" width="100%"/>
</div>

- Search Action Assign to VLAN và nhập ID VLAN để di chuyển. Action này sẽ move host qua VLAN được cách ly -> Nhấn OK (yêu cầu phải có Agent mới được nhé ^^)​
- Cuối cùng nhấn Finish và Policy sẽ hoạt động

<div align="center">
  <img src="/images/image617.png" alt="Network Topology" width="100%"/>
</div>

## 3. Kết luận 

- Việc triển khai VLAN cách ly cho thiết bị không tuân thủ giúp hệ thống mạng vừa duy trì được tính bảo mật, vừa đảm bảo tính liên tục dịch vụ bằng cách không chặn hoàn toàn thiết bị mà vẫn kiểm soát được hành vi truy cập.
- Thông qua chính sách tự động trong Forescout, các thiết bị vi phạm – như không có antivirus, không domain join, hoặc có hệ điều hành không hợp lệ – sẽ được phát hiện kịp thời và chuyển sang VLAN cách ly bằng các cơ chế như SNMP hoặc RADIUS VLAN Assignment.
- Giải pháp này không chỉ giúp ngăn chặn mối nguy từ bên trong, mà còn hỗ trợ người dùng tự khắc phục sự cố trong một môi trường hạn chế, trước khi được cấp lại quyền truy cập vào mạng chính.
- Đây là một trong những tính năng then chốt để triển khai mô hình NAC hiệu quả và thực tiễn, đặc biệt trong các tổ chức có số lượng thiết bị đầu cuối lớn, đa dạng và thường xuyên thay đổi.