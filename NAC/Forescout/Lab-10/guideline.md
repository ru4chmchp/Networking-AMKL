# Cấu hình cổng Captive Portal Forescout

Trong môi trường mạng có nhiều thiết bị không hỗ trợ 802.1X như thiết bị BYOD, khách mời, hoặc thiết bị không xác định, Captive Portal là một phương án kiểm soát truy cập hiệu quả và dễ triển khai. Forescout hỗ trợ cơ chế Captive Portal cho phép người dùng xác thực qua trình duyệt web trước khi được cấp quyền truy cập vào mạng.

Tính năng này đặc biệt phù hợp với:

- Doanh nghiệp có nhiều người dùng di động, thiết bị không domain join
- Khu vực Wi-Fi công cộng hoặc mạng khách
- Môi trường cần kiểm soát truy cập tạm thời, không yêu cầu cài đặt agent

Trong bài lab này, bạn sẽ thực hành:

- Bật tính năng Captive Portal trong Forescout
- Cấu hình trang xác thực, cơ chế xử lý và quyền truy cập sau đăng nhập
- Tích hợp với AD để xác thực
- Kiểm tra khả năng redirect và phân quyền truy cập theo người dùng

## Mục Lục:
1. Khái niệm Captive Portal​
2. Cấu hình cổng Captive Portal Forescout​
3. Cấu hình DNS Enforce và Guest Management​
4. Kiểm tra​
5. Kết luận​
​
## 1. Khái niệm Captive Portal​

Captive Portal là một trang web xuất hiện khi bạn kết nối vào WiFi công cộng, yêu cầu bạn đăng nhập, đồng ý điều khoản hoặc nhập thông tin trước khi được truy cập Internet. Nó giúp quản lý người dùng, tăng bảo mật và thường dùng tại quán cà phê, khách sạn, sân bay...

Mục đích sử dụng Captive Portal

- Kiểm soát và quản lý người dùng truy cập mạng.
- Thu thập thông tin khách hàng phục vụ cho marketing hoặc cải thiện dịch vụ.
- Hiển thị quảng cáo, thông báo hoặc các chương trình khuyến mãi trước khi cho phép truy cập Internet.
- Đảm bảo người dùng đồng ý với điều khoản sử dụng mạng, giảm thiểu rủi ro pháp lý cho nhà cung cấp dịch vụ.
- Giới hạn băng thông, thời gian truy cập hoặc số lượng thiết bị truy cập mạng.

<div align="center">
  <img src="/images/image548.png" alt="Network Topology" width="100%"/>
</div>

## 2. Cấu hình cổng Captive Portal Forescout​

Vào Policy -> Add -> Chọn Corporate/Guest Control

<div align="center">
  <img src="/images/image549.png" alt="Network Topology" width="100%"/>
</div>

Bước tiếp theo là Đặt tên cũng như chọn Segment. Ở bước Guest giao diện hiện ra như sau


<div align="center">
  <img src="/images/image550.png" alt="Network Topology" width="100%"/>
</div>

Khi người dùng trái phép truy cập, sẽ hiện trang đăng nhập để họ tự khai báo, đăng ký vào hệ thống như một Guest.
Sau khi đăng ký & xác nhận, user sẽ được coi là "Signed-In Guest" (khách đã đăng nhập).

Tùy chọn đăng nhập : 

- Show a Login page link...: Hiển thị trang đăng nhập cho khách tự đăng ký truy cập đầy đủ mạng như Logged In Guests.
    - Network access requests are automatically approved: Yêu cầu truy cập mạng của khách sẽ tự động được phê duyệt (không được chọn trong hình).
    - Guests must be approved by the sponsor they provide, or by: (Được chọn): Khách chỉ truy cập được khi sponsor do khách cung cấp (hoặc danh sách email sponsor nhập vào ô bên phải) phê duyệt.
- Fine-tune Registration Options:

    - Allow guests to skip login and have limited access only: Tùy chọn cho phép khách bỏ qua đăng nhập, chỉ truy cập mạng giới hạn.

Ở bước Sub-rule chúng ta thấy được các rule nhỏ được tạo sẵn là: Corporate Hosts, Signed-in Guests, Guest Hosts.

<div align="center">
  <img src="/images/image551.png" alt="Network Topology" width="100%"/>
</div>

Corporate Hosts

<div align="center">
  <img src="/images/image552.png" alt="Network Topology" width="100%"/>
</div>

Signed-in Guests

<div align="center">
  <img src="/images/image553.png" alt="Network Topology" width="100%"/>
</div>

Guest Hosts

<div align="center">
  <img src="/images/image554.png" alt="Network Topology" width="100%"/>
</div>

## 3. Cấu hình DNS Enforce và Guest Management​

DNS Enforce Plugin là một thành phần của Forescout Core Extensions Module, cho phép triển khai các hành động chính sách dựa trên HTTP trong các tình huống mà không thể kiểm tra lưu lượng trạng thái (stateful traffic). Đây là một giải pháp hiệu quả dành cho mạng từ xa hoặc phân đoạn mạng không được quản lý.

Cách hoạt động của DNS Enforce Plugin

- Khi một thiết bị khách (guest) kết nối vào mạng, plugin chuyển hướng DNS thông qua một địa chỉ IP mục tiêu (Target IP), được cấu hình là DNS server chính.
- Plugin đánh giá yêu cầu DNS từ máy khách:
    - Nếu có chính sách HTTP dựa trên yêu cầu đó, plugin trả về IP chuyển hướng.
    - Thiết bị sau đó gửi truy vấn HTTP đến IP chuyển hướng và được xử lý theo hành động HTTP quy định (ví dụ như đăng ký khách, đăng nhập...).
- Plugin không hoạt động hoàn toàn nếu thiết bị sử dụng proxy IP để duyệt web

Ví dụ minh họa quy trình hoạt động

1. Thiết bị gửi yêu cầu DHCP để vào mạng.
2. DHCP trả về cấu hình DNS với IP mục tiêu (Target IP).
3. Thiết bị gửi truy vấn DNS đến IP mục tiêu.
4. Forescout kiểm tra chính sách → nếu có chuyển hướng HTTP, trả về IP chuyển hướng.
5. Thiết bị gửi yêu cầu HTTP/HTTPS đến IP chuyển hướng.
6. Forescout xử lý yêu cầu và hiển thị nội dung theo chính sách quy định.

<div align="center">
  <img src="/images/image555.png" alt="Network Topology" width="100%"/>
</div>

Cách cấu hình DNS Enforce: Vào Tools -> Options -> Module -> DNS Enforce. Giao diện hiện ra như sau

<div align="center">
  <img src="/images/image556.png" alt="Network Topology" width="100%"/>
</div>

Giải thích các thành phần DNS Enforce:

Target IP: Địa chỉ IP đích mà các truy vấn DNS sẽ được chuyển tiếp tới hoặc kiểm soát. Lưu ý bạn phải đặt chung subnet với 1 cổng mạng của bạn (ở đây mình đặt chung với subnet của mgmt) nếu không bạn sẽ gặp lỗi Error: The management address network (x.x.x.x) and the target IP address network (y.y.y.y) are not the same.

<div align="center">
  <img src="/images/image557.png" alt="Network Topology" width="100%"/>
</div>

- Port: Cổng mạng được sử dụng cho dịch vụ DNS (mặc định là 53).
- TTL (Restricted): Thời gian sống (Time-To-Live) của các bản ghi DNS cho các host bị hạn chế, tính bằng giây (ở đây là 30 giây).
- Enable DNS resolution forwarding for unrestricted hosts: Cho phép chuyển tiếp phân giải DNS cho các host không bị giới hạn.
- TTL (Unrestricted): TTL cho các host không bị giới hạn (ở đây là 120 giây).
- Enable HTTPS listener: Bật chức năng lắng nghe qua giao thức HTTPS.
- Enable HTTP proxy mode: Bật chế độ proxy HTTP.

Module Guest Management của Forescout giúp doanh nghiệp quản lý truy cập mạng của khách mạng một cách an toàn và dễ dàng. Nó cung cấp:

- Cổng đăng ký khách mời tự động hoặc có nhân viên phê duyệt.
- Quyền truy cập hạn chế cho thiết bị khách, tách biệt với mạng nội bộ.
- Tùy chỉnh giao diện và các bước đăng ký.
- Giám sát và quản lý trạng thái truy cập khách theo thời gian.
- Tăng cường bảo mật, giảm gánh nặng cho bộ phận IT và thuận tiện cho khách.

Cấu hình Guest Management: Vào Tools -> Options -> Module -> Guest Management. Giao diện như sau

<div align="center">
  <img src="/images/image559.png" alt="Network Topology" width="100%"/>
</div>

Bạn có thể tự thêm trước thông tin của guest thủ công hoặc là thêm bằng file CSV với nút Import

<div align="center">
  <img src="/images/image560.png" alt="Network Topology" width="100%"/>
</div>

Chuyển qua tab Password Policy

<div align="center">
  <img src="/images/image561.png" alt="Network Topology" width="100%"/>
</div>

Các thành phần chính:

- Password minimum length: Độ dài tối thiểu của mật khẩu khách mời (ở đây là 6 ký tự).
- Các tùy chọn yêu cầu mật khẩu: Bạn có thể chọn các tiêu chí bắt buộc cho mật khẩu khách, bao gồm:
    - Ít nhất X ký tự viết hoa (upper case alphabetic characters)
    - Ít nhất X ký tự viết thường (lower case alphabetic characters)
    - Ít nhất X chữ số (digits): Tùy chọn này đang được bật với giá trị là 1 (mật khẩu phải có ít nhất 1 chữ số).
    - Ít nhất X ký tự đặc biệt (special characters)

Tiếp tục là tab User Policy

<div align="center">
  <img src="/images/image562.png" alt="Network Topology" width="100%"/>
</div>

Allow invalid email addresses (Cho phép địa chỉ email không hợp lệ): Khi ô này được chọn, hệ thống sẽ cho phép tạo tài khoản khách mời mà không bắt buộc kiểm tra tính hợp lệ của địa chỉ email. Điều này nghĩa là khách mời có thể nhập email sai định dạng hoặc không tồn tại, nhưng vẫn được chấp nhận để đăng ký tài khoản truy cập mạng.

Chuyển qua tab Sponsors

<div align="center">
  <img src="/images/image563.png" alt="Network Topology" width="100%"/>
</div>

Sponsor là nhân viên nội bộ có quyền phê duyệt, từ chối hoặc thu hồi truy cập mạng của khách, với phân quyền linh hoạt theo vai trò và thường được sử dụng trong môi trường yêu cầu xác thực nội bộ để tăng cường bảo mật.

Các thành phần chính

- All domain members are sponsors: Khi tích chọn mục này, tất cả thành viên thuộc domain (theo Active Directory hoặc user directory của công ty) sẽ được gán quyền sponsor (nhà tài trợ). Điều này tức là bất kỳ ai thuộc domain đều có quyền phê duyệt, từ chối hoặc thu hồi quyền truy cập của các khách mời trên mạng thông qua Guest Management Portal. Nếu không chọn, bạn có thể chỉ định từng cá nhân hoặc nhóm cụ thể làm sponsor.
- Danh sách sponsors:
    - Email/Active Directory Group: Email cá nhân hoặc tên nhóm AD được chọn làm sponsor.
    - Type: Kiểu sponsor (“Sponsor” hoặc "Admin Sponsor").
    - Description: Ghi chú mô tả (có thể để trống).
- Add/Edit/Remove: Các nút ở cạnh phải hỗ trợ thêm, sửa hoặc xóa sponsor thủ công.

Mục tiếp theo là phần cấu hình Terms and Conditions (Điều khoản và Điều kiện) trong module Guest Management của Forescout. Đây là nơi quản trị viên thiết lập các điều khoản mà khách mời và/hoặc nhà tài trợ phải đồng ý trước khi truy cập mạng doanh nghiệp.

<div align="center">
  <img src="/images/image564.png" alt="Network Topology" width="100%"/>
</div>

Các thành phần chính

- Enable guest Terms and Conditions. Khi chọn mục này, hệ thống sẽ yêu cầu khách mời chấp nhận điều khoản sử dụng trước khi được cấp quyền truy cập mạng. Có thể trình bày điều khoản dưới hai dạng:
    - URL: Dẫn đến trang web chứa nội dung điều khoản. Quản trị viên điền đường dẫn và có thể kiểm tra bằng nút Test.
    - Text: Soạn trực tiếp nội dung điều khoản trong giao diện quản trị (nút Edit).
- Enable sponsor Terms and Conditions. Khi bật mục này, nhà tài trợ (sponsor) cũng bắt buộc phải đọc và chấp nhận điều khoản trước khi duyệt quyền cho khách mời. Cách nhập liệu tương tự phần trên: có thể dùng URL hoặc nhập trực tiếp văn bản.

Tiếp theo là Guest Notifications. Mục này giúp doanh nghiệp chủ động thông báo cho khách về trạng thái tài khoản truy cập mạng và cách nhận thông báo.

<div align="center">
  <img src="/images/image565.png" alt="Network Topology" width="100%"/>
</div>

Các thành phần chính
- Enable guest notifications via
    - Email: Khi tick chọn mục này, hệ thống sẽ gửi thông báo trạng thái tới khách qua email.
    - SMS: Khi tick chọn mục này, khách sẽ nhận thông báo qua tin nhắn SMS.
- Notify guests when their guest account status changes to: Bạn có thể bật/tắt việc thông báo cho khách khi trạng thái tài khoản truy cập có thay đổi:
    - Account Pending: Báo khách biết yêu cầu đang chờ duyệt.
    - Account Approval: Báo khi quyền truy cập đã được phê duyệt/chấp nhận.
    - Account Rejection: Báo khi yêu cầu bị từ chối.
    - Account Revocation: Báo khi quyền truy cập bị thu hồi.
    - Account Expiration: Báo trước hoặc khi tài khoản hết hạn sử dụng.

Cuối cùng là giao diện cấu hình Sponsor Notifications: Tính năng này cho phép doanh nghiệp quyết định khi nào cần thông báo qua email cho các sponsor (người phê duyệt/sponsor nội bộ) về trạng thái của tài khoản khách mời.

<div align="center">
  <img src="/images/image566.png" alt="Network Topology" width="100%"/>
</div>

Notify sponsors when the guest account is set to: Danh sách các trạng thái tài khoản khách mời có thể gửi thông báo đến sponsor, bao gồm:

- Account Pending: Khi khách gửi yêu cầu truy cập, sponsor nhận thông báo là có yêu cầu mới cần duyệt.
- Account Approval: Khi tài khoản được duyệt thành công, sponsor nhận thông báo xác nhận.
- Account Rejection: Khi tài khoản bị từ chối, sponsor cũng được thông báo để tiếp nhận hoặc phản hồi nếu cần.
- Account Revocation: Khi tài khoản bị thu hồi, sponsor sẽ được thông báo.

## 4. Kiểm tra
Khi bạn kết nối vào mạng bạn sẽ được cấp IP và DNS là DNS Enforce của Forescout

<div align="center">
  <img src="/images/image567.png" alt="Network Topology" width="100%"/>
</div>

Lúc bạn truy cập vào 1 trang web giao diện captive portal sẽ hiện ra như sau

<div align="center">
  <img src="/images/image568.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image569.png" alt="Network Topology" width="100%"/>
</div>
Sau khi bạn điền tài khoản vào đây thì nó sẽ cho bạn truy cập các tài nguyên như bình thường

<div align="center">
  <img src="/images/image570.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image571.png" alt="Network Topology" width="100%"/>
</div>

## 5. Kết luận

- Cấu hình Captive Portal trên Forescout giúp kiểm soát hiệu quả các thiết bị không hỗ trợ 802.1X hoặc không cài agent, đặc biệt phù hợp cho môi trường có người dùng khách, BYOD hoặc thiết bị không định danh.
- Với cơ chế xác thực qua trình duyệt, hệ thống có thể nhận diện người dùng, áp dụng chính sách phân quyền và ghi log truy cập mà không cần thay đổi hạ tầng nhiều.
- Lab này giúp bạn nắm được cách cấu hình Captive Portal, và kiểm soát truy cập dựa trên trạng thái xác thực. Đây là một phần quan trọng trong việc hoàn thiện giải pháp NAC toàn diện trên Forescout.