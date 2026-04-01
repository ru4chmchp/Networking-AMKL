# Triển khai, cài đặt Forescout NAC trên VMWare​

Để triển khai giải pháp Network Access Control (NAC) Forescout vào hệ thống mạng doanh nghiệp, bước đầu tiên là cài đặt và cấu hình thành phần core của giải pháp – Forescout CounterACT. Trong môi trường thực tế, Forescout thường được triển khai dưới dạng appliance ảo (OVA) trên nền tảng ảo hóa như VMware, cho phép dễ dàng quản lý, mở rộng và tích hợp với hạ tầng sẵn có.

Bài Lab này sẽ hướng dẫn từng bước triển khai Forescout CounterACT lên môi trường VMWare, bao gồm:
- Import appliance OVA
- Cấu hình thông tin mạng (IP, gateway, DNS, NTP)
- Thiết lập thông tin ban đầu để sẵn sàng cho cấu hình sâu hơn

Việc nắm vững quy trình cài đặt là nền tảng để triển khai các lab tiếp theo như cấu hình chính sách, tích hợp với switch, AD, firewall, hoặc xây dựng kiến trúc NAC hoàn chỉnh cho doanh nghiệp.

## Mục lục

1. [Yêu cầu tài nguyên trên VMware cho Counter ACT](#1-yêu-cầu-tài-nguyên-trên-vmware-cho-counter-act)
2. [Mô hình Lab Forescout](#2-mô-hình-lab-forescout)
3. [Triển khai Forescout NAC trên VMware](#3-triển-khai-forescout-nac-trên-vmware)

Khi triển khai bạn sẽ cần các tài nguyên cần thiết bao gồm một appliance ảo (OVF) chứa hệ thống Forescout CounterACT và 1 file vmdk để lưu trữ nội dung của ổ cứng máy ảo.

<div align="center">
  <img src="/images/image178.png" alt="Network Topology" width="100%"/>
</div>

## 1. Yêu cầu tài nguyên trên VMware cho Counter ACT

Có nhiều appliance ảo (OVF) và yêu cầu tối thiểu của từng loại như sau:

<div align="center">
  <img src="/images/image179.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image180.png" alt="Network Topology" width="100%"/>
</div>

## 2. Mô hình Lab Forescout

## 3. Triển khai Forescout NAC trên VMware

Đầu tiên ta sẽ tạo 1 Virtual Machine mới trên VMWare bằng cách vào Menu-> VM and Templates ->Chọn thư mục -> Chuột phải vào thư mục chọn Deploy OVF Template...

<div align="center">
  <img src="/images/image181.png" alt="Network Topology" width="100%"/>
</div>

Lúc này cửa sổ Deploy OVF Template sẽ mở lên ta chọn URL nếu đã được đẩy lên hoặc Local file để chọn trong máy của mình để tiến hành tạo 1 VM từ file ovf.

Chọn 1 model mà mình muốn triển khai và file vmdk (nếu cấn kiểm tra tính toàn vẹn thì thêm file manifest)
- VCT-100.ovf
  - Đây là file OVF (Open Virtualization Format)
  - Nó giống như file cấu hình của máy ảo
  - Bên trong chứa:
    - CPU, RAM
    - Network adapter
    - Disk sẽ gắn (trỏ tới file .vmdk)
    - OS type (Linux/Windows…)
- CounterACT-9.1.2-602.iso_v.vmdk
  - Đây là file VMDK (Virtual Machine Disk)
  - Chính là ổ cứng ảo của máy
  - Bên trong chứa:
    - Hệ điều hành (ở đây là CounterACT – của Forescout NAC)
    - File system, dữ liệu, application
- CounterACT-9.1.2-602.iso_v.vmdk.mf
  - Đây là file Manifest (.mf)
  - Dùng để kiểm tra tính toàn vẹn (integrity)
  - Bên trong chứa:
    - Hash (SHA1/SHA256) của:
        - file .ovf
        - file .vmdk

<div align="center">
  <img src="/images/image182.png" alt="Network Topology" width="100%"/>
</div>

Ở các bước tiếp theo chúng ta đặt tên, chọn Resouce và Storage, Network

<div align="center">
  <img src="/images/image194.png" alt="Network Topology" width="100%"/>
</div>

Và finish

<div align="center">
  <img src="/images/image195.png" alt="Network Topology" width="100%"/>
</div>

Lưu ý:
- Có thể 1 số VMWare không hỗ trợ Redhat 9 nên khi cài đặt bạn hãy đổi qua Other Linux (64-bit) để chạy được

<div align="center">
  <img src="/images/image196.png" alt="Network Topology" width="100%"/>
</div>

Sau khi deploy xong sẽ có giao diện như này 

<div align="center">
  <img src="/images/image197.png" alt="Network Topology" width="100%"/>
</div>

Ta tiến hành Run máy và vào được giao diện như thế này

<div align="center">
  <img src="/images/image198.png" alt="Network Topology" width="100%"/>
</div>

 Trong mục options ta sẽ chọn 1 rồi nhấn Enter để cấu hình Forescout.

 <div align="center">
  <img src="/images/image198.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo ở phần HA mode Forescout nếu cấu hình HA (active-passive) thì ta chọn 2 hoặc 3, trong bài này ta không cấu hình HA nên sẽ chọn 1 để cấu hình Stardard rồi nhấn Enter.

<div align="center">
  <img src="/images/image198.png" alt="Network Topology" width="100%"/>
</div>

Ở bước cài đặt ban đầu của nền tảng Forescout nhấn yes để tiếp tục cấu hình từ Forescout Console

<div align="center">
  <img src="/images/image201.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo, ở phần Installation Type, ta chọn 1 khi chỉ quản lý một thiết bị Forescout Appliance, hoặc chọn 2 khi quản lý nhiều thiết bị Forescout thông qua Enterprise Manager, rồi nhấn Enter1

<div align="center">
  <img src="/images/image201.png" alt="Network Topology" width="100%"/>
</div>

 Tiếp theo, ở phần Licensing Mode, chọn 1 nếu sử dụng license riêng cho từng thiết bị Forescout, trong đó số lượng endpoint được quy định cố định. Chọn 2 nếu muốn license được quản lý tập trung trên Enterprise Manager hoặc Standalone Appliance, ở chế độ này số lượng endpoint sẽ được phân bổ theo từng lần triển khai.

<div align="center">
  <img src="/images/image203.png" alt="Network Topology" width="100%"/>
</div>

Bước tiếp theo ta sẽ nhập mô tả VM

<div align="center">
  <img src="/images/image204.png" alt="Network Topology" width="100%"/>
</div>

Ở bước Administrator Password, nhập mật khẩu từ 6-15 ký tự, bao gồm ít nhất một ký tự không phải chữ cái, rồi nhấn Enter. Mật khẩu này được sử dụng để truy cập CLI (user: cliadmin) và Forescout Console (user: admin).

<div align="center">
  <img src="/images/image205.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo, trong phần Network Settings, chọn interface dành cho Management, sau đó cấu hình địa chỉ IP, subnet mask, gateway, và DNS, rồi nhấn Enter.

<div align="center">
  <img src="/images/image206.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo, kiểm tra lại thông tin đã cấu hình cho Forescout. Nếu cấu hình chính xác, chọn D để tiếp tục. Nếu muốn chỉnh sửa, chọn R để cấu hình lại. Nếu cần kiểm tra kết nối mạng, chọn T để thực hiện Ping.

Sau khi quá trình cài đặt Forescout hoàn tất, hệ thống sẽ hiển thị thông báo xác nhận. Lưu ý địa chỉ URL để truy cập và tải Forescout Console. Trong bài này, URL truy cập là: https://10.30.195.222/install.

<div align="center">
  <img src="/images/image207.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến ta đăng nhập vào CLi với user cliadmin/password đã đặt

<div align="center">
  <img src="/images/image214.png" alt="Network Topology" width="100%"/>
</div>

Sau đó ta vào shell với mật khẩu đã đặt

<div align="center">
  <img src="/images/image215.png" alt="Network Topology" width="100%"/>
</div>

Với SSH bạn chỉ cần nhập lệnh ssh_root_password_login enable sau đó nhập Mật khẩu người dùng và Shell pass để enable SSH

<div align="center">
  <img src="/images/image217.png" alt="Network Topology" width="100%"/>
</div>

Sau đó thử SSH vào

<div align="center">
  <img src="/images/image216.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo đó ta vào địa chỉ trên và tải Console về 

<div align="center">
  <img src="/images/image208.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến đăng nhập vào Console như đã setup

<div align="center">
  <img src="/images/image209.png" alt="Network Topology" width="100%"/>
</div>

Sau khi login vào thì ta tiến hành setup

<div align="center">
  <img src="/images/image210.png" alt="Network Topology" width="100%"/>
</div>

Phần License này sẽ làm sau nên skip nó qua

<div align="center">
  <img src="/images/image211.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến là NTP Server

<div align="center">
  <img src="/images/image212.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến là phần cấu hình Mail setup nhận thông báo và cảnh báo từ hệ thống Forescout, sau khi bạn nhập xong email giao diện sẽ như hình sau.

<div align="center">
  <img src="/images/image213.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến là phần cấu hình Mail setup nhận thông báo và cảnh báo từ hệ thống Forescout, sau khi bạn nhập xong email giao diện sẽ như hình sau.

<div align="center">
  <img src="/images/image218.png" alt="Network Topology" width="100%"/>
</div>

Bước tiếp theo là cấu hình Forescout tích hợp với Active Directory hoặc hệ thống xác thực khác, ở đây mình chưa cấu hình nên nhấn Skip.

<div align="center">
  <img src="/images/image219.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo đó là Domains Setup, ở bước này mình cũng bỏ qua vì chưa cấu hình.

<div align="center">
  <img src="/images/image220.png" alt="Network Topology" width="100%"/>
</div>

Bước tiếp theo là cấu hình Authentication Servers, ở đây mình chưa cấu hình nên nhấn Skip.

<div align="center">
  <img src="/images/image221.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo là bước cấu hình Internal Network (phạm vi địa chỉ IP nội bộ mà CounterACT sẽ quản lý), ở đây mình chọn toàn bộ IPv4

<div align="center">
  <img src="/images/image222.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image223.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo đó sẽ là bước Enforcement Mode, giải thích như sau:
- Full Enforcement: Hệ thống sẽ áp dụng đầy đủ các chính sách kiểm soát đối với các thiết bị trong mạng.
- Partial Enforcement: Một số tính năng như Threat Protection, HTTP Actions, Virtual Firewall sẽ bị vô hiệu hóa.
- NAT Detection: Giúp xác định các thiết bị đang sử dụng NAT để tránh lỗi kết nối.
- Auto Discovery: Hệ thống tự động quét và nhận diện thiết bị trên mạng.

<div align="center">
  <img src="/images/image224.png" alt="Network Topology" width="100%"/>
</div>>

Bước tiếp theo đó là cấu hình cổng mornitor và response, nhấn vào nút channel để tiến hành Add channel mới

<div align="center">
  <img src="/images/image225.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image226.png" alt="Network Topology" width="100%"/>
</div>


Sau khi nhấn Channel giao diện Add channel hiện ra như sau
- Monitor Interface: Chọn giao diện để theo dõi lưu lượng mạng.
- Response Interface: Chọn giao diện để thực hiện phản hồi hoặc áp dụng chính sách.
- Interface List: Hiển thị danh sách các giao diện mạng đang được quản lý (ens192, ens224, ens256).
- Traffic Metrics: Hiển thị thông tin về VLAN, lưu lượng mạng, Unicast, Broadcast, nhưng hiện tại chưa có dữ liệu nào.
- Advanced Options: Cung cấp các tùy chọn nâng cao để tinh chỉnh cấu hình.

<div align="center">
  <img src="/images/image227.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image228.png" alt="Network Topology" width="100%"/>
</div>

Bước tiếp theo là kết nối các Switch trong hệ thống mạng , ở đây mình chưa cấu hình nên nhấn Skip.

<div align="center">
  <img src="/images/image229.png" alt="Network Topology" width="100%"/>
</div>

Bước tiếp theo là cấu hình policy, ở đây mình chưa cấu hình nên nhấn Next.

<div align="center">
  <img src="/images/image231.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo là Inventory Discovery: Cung cấp góc nhìn toàn diện về mạng, hiển thị các tiến trình đang chạy trên thiết bị, cổng mở, và nhiều thông tin hệ thống khác.

<div align="center">
  <img src="/images/image230.png" alt="Network Topology" width="100%"/>
</div>

Cuối cùng là bước Review các thông tin từ nãy giờ mình đã cấu hình, giao diện như sau

<div align="center">
  <img src="/images/image232.png" alt="Network Topology" width="100%"/>
</div>

Nhấn Finish

Đợi 1 lúc để Forescout Console thực hiện cấu hình. Sau khi cấu hình xong giao diện của Forescout Console như sau

<div align="center">
  <img src="/images/image233.png" alt="Network Topology" width="100%"/>
</div>

Việc triển khai thành công Forescout CounterACT trên nền tảng VMware là bước khởi đầu quan trọng trong quá trình xây dựng hệ thống Network Access Control toàn diện. Ở bước này, chúng ta đã hoàn tất quá trình import appliance, cấu hình mạng cơ bản và chuẩn bị cho các cấu hình chuyên sâu phía sau.
- Triển khai trên môi trường ảo hóa giúp tăng tính linh hoạt, dễ dàng mở rộng và thuận tiện trong việc quản lý – đặc biệt phù hợp với hạ tầng doanh nghiệp hiện đại. Việc nắm vững các bước triển khai ban đầu sẽ giúp bạn tiết kiệm thời gian khi triển khai thực tế cũng như đảm bảo hệ thống Forescout vận hành ổn định, sẵn sàng tích hợp với các thành phần mạng khác.
- Đây chính là nền móng cho các Lab tiếp theo như tích hợp switch, AD, thiết lập chính sách phát hiện – kiểm soát thiết bị, và xây dựng hệ thống NAC có khả năng phản ứng tự động trước các mối đe dọa trong mạng nội bộ.