# Cấu hình Backup và Restore cấu hình thiết bị Firewall Fortigate

Backup và Restore là các chức năng quan trọng nhằm bảo vệ và duy trì cấu hình của thiết bị Firewall FortiGate. Việc thiết lập cơ chế sao lưu giúp lưu giữ cấu hình an toàn, thuận tiện cho việc khôi phục nhanh chóng khi xảy ra sự cố hoặc lỗi cấu hình. Nhờ đó, quá trình quản trị hệ thống được đảm bảo tính liên tục và giảm thiểu rủi ro vận hành.

## 1. Cấu hình Backup

Đầu tiên thiết lập sẵn 1 chính sách trong policy 

<div align="center">
  <img src="/images/image169.png" alt="Network Topology" width="100%"/>
</div>

Bây giờ sẽ tiến hành sao lưu để có khi sự cố bất ngờ xảy ra thì sẽ có bản dự phòng
- B1: Bấm vào mũi tên xuống phần admin bên góc trên tay phải màn hình
- B2: Chọn Backup System Configuration ( có thể tùy chọn loại file, cài đặt password ) sau đó nhấn OK thì bản backup tại thời điểm đó sẽ được sao lưu xuống

<div align="center">
  <img src="/images/image170.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image171.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image172.png" alt="Network Topology" width="100%"/>
</div>

## 2. Cấu hình Restore


Sau khi sao lưu xong chúng ta tạo thêm một chính sách khác để thử khôi phục lại firewall về tình trạng chỉ có 1 rules đã thiết lập khi chưa backup

<div align="center">
  <img src="/images/image173.png" alt="Network Topology" width="100%"/>
</div>

Bây giờ sẽ tiến hành khôi phục lại dữ liệu và cấu hình về tình trạng lúc đầu (lúc chỉ có 1 chính sách)
- B1: Bấm vào mũi tên xuống phần admin bên góc trên tay phải màn hình
- B2: Chọn Restore System Configuration --> Upload file backup đã tải trước đó lên --> Điền đúng pass

<div align="center">
  <img src="/images/image174.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image175.png" alt="Network Topology" width="100%"/>
</div>

Sau khi OK sẽ hiện ra yêu cầu reboot, hãy nhấn ok
<div align="center">
  <img src="/images/image176.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image177.png" alt="Network Topology" width="100%"/>
</div>

oke vậy thành công nhá

