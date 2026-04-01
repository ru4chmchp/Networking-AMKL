# Active License Forescout​

Để hệ thống Forescout hoạt động đầy đủ các tính năng như phát hiện thiết bị, kiểm soát truy cập, tích hợp với các nền tảng bảo mật khác và triển khai theo mô hình doanh nghiệp, việc kích hoạt bản quyền (license) là bắt buộc. License là yếu tố xác định số lượng thiết bị có thể quản lý, thời hạn sử dụng hệ thống, cũng như các module tính năng nâng cao được phép triển khai (Policy, Control, Integration, v.v.).

Forescout sử dụng Centralized Licensing System (CLS), cho phép quản lý license tập trung từ Forescout Portal. Sau khi đăng ký bản quyền, người quản trị cần thực hiện các bước:
- Đăng ký thiết bị (Appliance ID) trên portal
- Nhập mã bản quyền (license key)
- Tải về file giấy phép
- Kích hoạt trực tiếp hoặc ngoại tuyến trên thiết bị Forescout

Bài hướng dẫn này sẽ giúp bạn thực hiện đầy đủ quy trình kích hoạt license Forescout, đảm bảo hệ thống hoạt động hợp pháp, đầy đủ tính năng và sẵn sàng triển khai ở quy mô sản xuất.

## Mục lục:
1. [Đăng kí tài khoản Portal​](#1-yêu-cầu-tài-nguyên-trên-vmware-cho-counter-act)
2. [Kích hoạt license Offline​](#2-mô-hình-lab-forescout)
3. [Kích hoạt license Online​](#3-triển-khai-forescout-nac-trên-vmware)
​
## 1. Đăng kí tài khoản Portal

Để kích hoạt license bạn phải có tài khoản Customer Portal hoặc Partner Portal. Truy cập vào trang https://forescout.my.site.com/ để tiến hành đăng kí. Giao diện Web như sau:

<div align="center">
  <img src="/images/image234.png" alt="Network Topology" width="100%"/>
</div>

Bấm vào Customer Portal hoặc Partner Portal để đăng kí tài khoản. Với Customer Portal bạn hãy điền hết tất cả các thông tin để tạo tài khoản:

<div align="center">
  <img src="/images/image235.png" alt="Network Topology" width="100%"/>
</div>

Còn với Partner Portal giao diện như sau:

<div align="center">
  <img src="/images/image236.png" alt="Network Topology" width="100%"/>
</div>

## 2. Kích hoạt license Offline​

Sau khi đăng kí xong giao diện support hiện ra, vào Support -> License bạn sẽ nhìn thấy các Module bạn được cấp giấy phép (Bạn phải request License NFR của hãng)

<div align="center">
  <img src="/images/image237.png" alt="Network Topology" width="100%"/>
</div>

Lướt xuống dưới phần Deployment, Nhấn Add Deployment Cấu hình các module bạn sẽ triển khai cùng với số lượng thiết bị bạn sẽ đăng kí

<div align="center">
  <img src="/images/image238.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image239.png" alt="Network Topology" width="100%"/>
</div>

Vào Forescout Console chọn Tools -> Option -> Licenses

<div align="center">
  <img src="/images/image240.png" alt="Network Topology" width="100%"/>
</div>

Sau đó nhấn vào > Switch to Flexx Licensing và nhập Deployment ID: để download license đã add khi nãy xuống

<div align="center">
  <img src="/images/image241.png" alt="Network Topology" width="100%"/>
</div>

Quay lại với trang Portal lướt tới License Management -> Chọn Activate License -> Đưa file .licenserequest vào

<div align="center">
  <img src="/images/image242.png" alt="Network Topology" width="100%"/>
</div>

Sau khi đưa file .licenserequest xong nó sẽ hiện ra nút download với file License.fsl

<div align="center">
  <img src="/images/image243.png" alt="Network Topology" width="100%"/>
</div>

Vào Forescout Console upload file license, chờ 1 lúc, chúng ta đã kích hoạt 
thành công

<div align="center">
  <img src="/images/image244.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image247.png" alt="Network Topology" width="100%"/>
</div>

## 3. Kích hoạt license Online​

Lúc đăng nhập vào Forescout Console giao diện Customer Verification hiện ra

<div align="center">
  <img src="/images/image245.png" alt="Network Topology" width="100%"/>
</div>

Tiến hành login và login tài khoản ở trang Portal và Cho phép ứng dụng Forescout truy cập.

Bước tiếp theo vào Tools -> Options -> Licenses -> Activate.

<div align="center">
  <img src="/images/image246.png" alt="Network Topology" width="100%"/>
</div>

Nhập Deployment ID (nằm ở Portal phần Deployment)

<div align="center">
  <img src="/images/image248.png" alt="Network Topology" width="100%"/>
</div>

Nhấn Activate License và một hộp thoại mở ra hiển thị tiến trình kích hoạt, tiếp theo là một hộp thoại cho biết giấy phép đã được kích hoạt thành công

<div align="center">
  <img src="/images/image247.png" alt="Network Topology" width="100%"/>
</div>

Việc kích hoạt license là bước bắt buộc để hệ thống Forescout có thể vận hành đầy đủ tính năng, đảm bảo tuân thủ đúng quy định bản quyền và có thể mở rộng theo nhu cầu thực tế của doanh nghiệp.

Thông qua bài hướng dẫn này, bạn đã nắm được quy trình cấp phát và active license Forescout theo cả hai phương thức online và offline. Việc quản lý license tập trung qua hệ thống Forescout Centralized Licensing Portal cũng giúp theo dõi trạng thái bản quyền dễ dàng, hỗ trợ việc gia hạn, nâng cấp và quản lý nhiều appliance trong cùng hệ thống.

Đây là bước nền tảng để triển khai Forescout trong môi trường sản xuất, đảm bảo hệ thống hoạt động ổn định, hợp pháp và sẵn sàng tích hợp các tính năng nâng cao như kiểm soát truy cập theo chính sách, tích hợp với các hệ thống bảo mật khác, và mở rộng số lượng thiết bị giám sát theo nhu cầu thực tế.
