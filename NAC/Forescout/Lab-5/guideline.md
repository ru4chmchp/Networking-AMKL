# Cấu hình tích hợp với Active Directory (AD), LDAP, VLAN và RADIUS trên Forescout​

Để triển khai chính sách kiểm soát truy cập theo người dùng một cách chặt chẽ và linh hoạt, việc tích hợp Forescout với các thành phần hạ tầng như Active Directory (AD), LDAP, VLAN và RADIUS là bước không thể thiếu. Những thành phần này đóng vai trò nền tảng trong việc xác thực, phân quyền và điều khiển quyền truy cập của người dùng và thiết bị vào hệ thống mạng.

- AD/LDAP giúp Forescout truy vấn danh tính người dùng, nhóm, OU – từ đó cho phép định danh chính xác ai đang sử dụng thiết bị nào.
- RADIUS đóng vai trò là giao thức trung gian giữa thiết bị mạng (switch, controller Wi-Fi…) và hệ thống xác thực trung tâm (AD/LDAP), hỗ trợ xác thực 802.1X.
- VLAN là công cụ thực thi chính sách sau xác thực, dùng để cách ly thiết bị theo nhóm, phân quyền truy cập hoặc cô lập thiết bị vi phạm.


Bài lab này sẽ hướng dẫn bạn:

- Cấu hình tích hợp Forescout với AD/LDAP để định danh người dùng
- Thiết lập RADIUS để xác thực truy cập mạng
- Áp dụng chính sách điều khiển VLAN động dựa trên profile người dùng hoặc thiết bị

Đây là bước quan trọng để hệ thống NAC có thể kiểm soát truy cập theo danh tính và trạng thái bảo mật của thiết bị, nâng cao đáng kể khả năng bảo vệ mạng doanh nghiệp.

## Mục lục

1. [Giới thiệu về Active Directory và LDAP​]()
    1.1 [Active Directory (AD)​]()
    1.2 [LDAP (Lightweight Directory Access Protocol)​]()
2. [Cấu hình tích hợp Forescout (AD, LDAP, RADIUS, VLAN)]()
    2.1 [Tích hợp LDAP​]()
    2.2 [Join Domain và cấu hình RADIUS​]()
    2.3 [Cấu hình thêm VLAN​]()
3. [Kết luận]()

## 1. Giới thiệu về Active Directory và LDAP

### 1.1 Active Directory (AD)
> **Active Directory** (AD) : Là dịch vụ thư mục (directory service) do Microsoft phát triển, được sử dụng rộng rãi trong các hệ thống doanh nghiệp để quản lý người dùng, thiết bị, quyền truy cập và chính sách bảo mật trong môi trường Windows. AD tổ chức dữ liệu theo dạng cây phân cấp gồm các đối tượng như user, group, computer và OU (Organizational Unit).

Các chức năng chính của AD:
- Xác thực người dùng và máy tính
- Quản lý tập trung tài khoản, nhóm, và quyền truy cập
- Cung cấp nền tảng cho các dịch vụ như Group Policy, DNS tích hợp, và Kerberos

<div align="center">
  <img src="/images/image348.png" alt="Network Topology" width="100%"/>
</div>

### 1.2 LDAP (Lightweight Directory Access Protocol)​

> **Lighwight Directory Access Protocol** (LDAP) : Là giao thức chuẩn dùng để truy cập và truy vấn thông tin trong dịch vụ thư mục, bao gồm cả Active Directory và các hệ thống tương đương (OpenLDAP, Oracle Directory, v.v.). LDAP sử dụng cấu trúc dạng cây và cho phép ứng dụng bên thứ ba – như Forescout – truy cập thông tin người dùng, nhóm, OU... để phục vụ cho việc xác thực và phân quyền.

Trong bối cảnh Forescout:
- LDAP là giao thức trung gian để Forescout giao tiếp với AD
- Forescout sử dụng LDAP để truy vấn thông tin user, kiểm tra nhóm, và thực hiện các rule phân loại thiết bị theo danh tính người dùng

Việc hiểu rõ vai trò của AD và LDAP giúp triển khai tích hợp chính xác, đảm bảo Forescout có thể khai thác tối đa thông tin phục vụ cho kiểm soát truy cập và phân quyền theo người dùng.

<div align="center">
  <img src="/images/image349.png" alt="Network Topology" width="100%"/>
</div>

## 2. Cấu hình tích hợp Forescout (AD, LDAP, RADIUS, VLAN)

### 2.1 Tích hợp LDAP

Vào Tool -> Options -> User Directory -> Nhấn Add để thêm Server. Giao diện như hình bên dưới

Ở bước 1 Nhập Tên cũng như là Type chọn Microsoft Active Directory

Các tùy chọn checkbox:

- Use as directory: Máy chủ này sẽ được sử dụng để truy vấn thông tin thư mục (directory information) như chi tiết người dùng hoặc nhóm từ Active Directory.
- Use for authentication: Máy chủ này cũng sẽ được sử dụng để xác thực người dùng (authentication) khi họ đăng nhập hoặc thực hiện các hành động yêu cầu xác thực.
- Use for CounterACT login: Cho phép sử dụng máy chủ này để xác thực đăng nhập vào giao diện quản trị của CounterACT.
- Include parent groups: Bao gồm các nhóm cha (parent groups) của người dùng trong quá trình truy vấn, hữu ích khi quản lý các nhóm lồng nhau trong AD.

> Lưu ý ở đây Name phải lấy tên host của AD đặt nha.

<div align="center">
  <img src="/images/image350.png" alt="Network Topology" width="100%"/>
</div>

Bước 2: Điền các trường thông tin

- Address: Nhập địa chỉ IP của máy chủ Active Directory.
- Port: Cổng kết nối (mặc định 636), tích "Use TLS" để bật mã hóa. Hoặc không tích để dùng port 389
- Accessed By: Chọn "All CounterACT Devices" (khuyến nghị) hoặc chọn thiết bị Forescout cụ thể.
- Domain: Nhập tên miền, ví dụ: intern.local.
- Administrator: Nhập tên tài khoản quản trị viên AD.
- Password: Nhập mật khẩu của tài khoản quản trị viên.
- Verify Password: Nhập lại mật khẩu để xác nhận.
- Additional Domain Aliases: Chọn Specify và nhập NetBIOS của máy chủ vào

<div align="center">
  <img src="/images/image351.png" alt="Network Topology" width="100%"/>
</div>

Bước 3: Nhập tài khoản Test để thử nghiệm (bạn có thể bấm finish luôn) sau đó nhấn Apply để lưu cấu hình

<div align="center">
  <img src="/images/image352.png" alt="Network Topology" width="100%"/>
</div>

Sau đó finish and apply

<div align="center">
  <img src="/images/image408.png" alt="Network Topology" width="100%"/>
</div

<div align="center">
  <img src="/images/image407.png" alt="Network Topology" width="100%"/>
</div

### 2.2 Join Domain và cấu hình RADIUS
Nhớ start Plugin trước khi thực hiện 

<div align="center">
  <img src="/images/image404.png" alt="Network Topology" width="100%"/>
</div>

Bước 1: Vào Tool -> Options -> RADIUS -> Nhấn Add để thêm Nguồn xác thực RADIUS

<div align="center">
  <img src="/images/image355.png" alt="Network Topology" width="100%"/>
</div

Chọn vào tên đặt ở trong User Directory để thêm Nguồn xác thực

Bước 2: Nhấn Join và nhập tài khoản quản trị AD, Nhấn YES để tiếp tục khi điền xong tài khoản.

<div align="center">
  <img src="/images/image410.png" alt="Network Topology" width="100%"/>
</div>

Kết quả sau khi Join

<div align="center">
  <img src="/images/image411.png" alt="Network Topology" width="100%"/>
</div>

Tiến hành Test, đầu tiên ta chuột phải vào AD và nhấn vào configure để add account để test

<div align="center">
  <img src="/images/image412.png" alt="Network Topology" width="100%"/>
</div>

Sau đó ta nhấn vào nút test bên phải rồi đợi kết quả

<div align="center">
  <img src="/images/image414.png" alt="Network Topology" width="100%"/>
</div>

Vậy là xong Radius

### 2.3 Cấu hình thêm VLAN

Vào Tool -> Options -> Channel -> Vlan -> Add. Popup Add Vlan hiện ra như sau

<div align="center">
  <img src="/images/image415.png" alt="Network Topology" width="100%"/>
</div>

Nhập Vlan mình muốn quản lý cũng như IP

<div align="center">
  <img src="/images/image416.png" alt="Network Topology" width="100%"/>
</div>

Kết quả sau khi add xong
<div align="center">
  <img src="/images/image417.png" alt="Network Topology" width="100%"/>
</div>


## 3. Kết luận

- Việc tích hợp Forescout với Active Directory (AD), LDAP, RADIUS và VLAN giúp hệ thống NAC có khả năng xác định rõ danh tính người dùng, xác thực truy cập mạng, và thực thi chính sách phân quyền ở mức hạ tầng mạng một cách tự động và chính xác.
- Thông qua AD/LDAP, Forescout truy vấn được thông tin người dùng, nhóm, đơn vị tổ chức (OU), từ đó gắn kết danh tính với từng thiết bị. Giao thức RADIUS kết hợp với 802.1X cho phép xác thực tập trung và kiểm soát chặt chẽ ai được phép truy cập vào hệ thống mạng. Sau khi xác thực, thiết bị sẽ được phân VLAN động tương ứng với vai trò hoặc mức độ tuân thủ chính sách bảo mật.
- Bài lab này cung cấp nền tảng quan trọng để bạn xây dựng các policy kiểm soát truy cập nâng cao theo mô hình zero trust – nơi mọi truy cập đều được xác thực, phân quyền, và giám sát chặt chẽ từ lúc kết nối đến lúc rời mạng.
- Đây là một trong những bước cốt lõi để triển khai Forescout NAC bài bản, hướng đến một hệ thống mạng thông minh, an toàn, và dễ quản lý ở quy mô lớn.