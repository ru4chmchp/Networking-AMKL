# Cấu hình sao lưu và khôi phục dữ liệu Forescout​

Trong quá trình vận hành hệ thống Forescout NAC, việc cấu hình, tích hợp, và xây dựng chính sách thường mất nhiều thời gian và công sức. Vì vậy, để đảm bảo tính an toàn cấu hình, phục hồi nhanh khi có sự cố hoặc chuẩn bị cho việc nâng cấp hệ thống, việc sao lưu (backup) định kỳ và khôi phục (restore) dữ liệu là một yêu cầu bắt buộc trong mọi môi trường triển khai thực tế.

Forescout hỗ trợ các cơ chế sao lưu cấu hình hệ thống, bao gồm:

- Backup thủ công qua giao diện web
- Tự động backup định kỳ qua FTP/SFTP
- Khôi phục toàn bộ cấu hình từ file backup trong trường hợp hỏng hóc, lỗi thiết bị, hoặc chuyển đổi hệ thống

Bài lab này sẽ hướng dẫn bạn cách thực hiện quá trình backup và restore dữ liệu cấu hình trên Forescout CounterACT, đảm bảo hệ thống luôn sẵn sàng phục hồi trong mọi tình huống và giảm thiểu tối đa rủi ro mất cấu hình.

## Mục Lục:
1. Cấu hình Backup cho Forescout​
2. Restore​ cho Forescout
3. Kết luận​

## Cấu hình Backup cho Forescout

Trước tiên để cấu hình backup thì ta phải có một ftp server đã, ta vào mở Server Manager, chọn Manage -> Add Roles and Features. Sau đó ở phần Sever Roles chọn Web Server (IIS)

<div align="center">
  <img src="/images/image418.png" alt="Network Topology" width="100%"/>
</div>

Trong mục Web Server (IIS) -> FTP Server, tích chọn cả FTP Service và FTP Extensibility.

<div align="center">
  <img src="/images/image419.png" alt="Network Topology" width="100%"/>
</div>

Mở IIS Manager (trong Tools). Chuột phải vào Sites -> chọn Add FTP Site.

<div align="center">
  <img src="/images/image421.png" alt="Network Topology" width="100%"/>
</div>
Đặt tên Site và chọn đường dẫn thư mục chia sẻ (Physical path).
<div align="center">
  <img src="/images/image422.png" alt="Network Topology" width="100%"/>
</div>
Binding and SSL Settings: Chọn IP Address, Port mặc định là 21, chọn No SSL (để thử nghiệm) hoặc cấu hình chứng chỉ nếu cần bảo mật.
<div align="center">
  <img src="/images/image423.png" alt="Network Topology" width="100%"/>
</div>
Authentication: Chọn Basic (để dùng user Windows).
Authorization: Chọn user/nhóm người dùng và quyền truy cập (Read/Write).
<div align="center">
  <img src="/images/image424.png" alt="Network Topology" width="100%"/>
</div>

Sau đó chọn cái FTP Site mà bạn đã tạo. Nhấp đúp vào mục FTP Authorization Rules. hãy nhấn Add Allow Rule.

<div align="center">
  <img src="/images/image425.png" alt="Network Topology" width="100%"/>
</div>

Sau đó ta vào thư mục backup đã tạo khi nãy, click chuột phải và ấn vào Properties > Security > Edit > Add 

<div align="center">
  <img src="/images/image426.png" alt="Network Topology" width="100%"/>
</div>

Check

<div align="center">
  <img src="/images/image427.png" alt="Network Topology" width="100%"/>
</div>

Sau đó ta tiến hành vào Forescout

Vào Tool -> Options -> Advanced -> Backup. Giao diện sẽ hiện ra như sau: Nhập các thông số của Backup Server và chọn Protocol để chuyển file Backup tới

<div align="center">
  <img src="/images/image428.png" alt="Network Topology" width="100%"/>
</div>

Ở option Encryption Password nhập mật khẩu để tiến hành mã hoá file backup
<div align="center">
  <img src="/images/image429.png" alt="Network Topology" width="100%"/>
</div>

Ở option System Backup Enable lên chúng ta có thể cài đặt schedule hoặc backup now để backup cho toàn bộ hệ thống

<div align="center">
  <img src="/images/image430.png" alt="Network Topology" width="100%"/>
</div>

Ở option Component Backup chúng ta có thể chọn các thành phần để backup (như hình) và cũng có backup schedule

<div align="center">
  <img src="/images/image431.png" alt="Network Topology" width="100%"/>
</div>


<div align="center">
  <img src="/images/image432.png" alt="Network Topology" width="100%"/>
</div>

Sau đó ta check thử 

<div align="center">
  <img src="/images/image433.png" alt="Network Topology" width="100%"/>
</div>

## Restore cho Forescout

Vào Shell nhập lệnh fstool restore <địa chỉ file backup trong forescout> -> Nhập password decrypt -> Nhấn yes. Sau đó Appliance sẽ được Restart

