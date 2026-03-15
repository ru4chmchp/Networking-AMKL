# Cài đặt các thành phần trong mô hình và cấu hình các thông tin cơ bản: hostname, DNS, NTP, LACP,...

Đầu tiên sau khi download image và chạy trong Pnetlab hoặc EVE-NG thì ta khởi động máy ảo chứa image Fortigate. Ta nhập tài khoảng là admin và không nhập password, nhấn thẳng enter.

<div align="center">

  <img src="/images/image copy.png" alt="Network Topology" width="100%"/>

</div>

Sau khi vào được console config thì ta tiến hành config ip đơn giản để truy cập web từ bên ngoài vào như sau.
<div align="center">

  <img src="/images/image copy 2.png" alt="Network Topology" width="100%"/>

</div>

Ta config ip trong subnet 10.120.170.0/24.

<div align="center">

  <img src="/images/image111.png" alt="Network Topology" width="100%"/>

</div>

Sau đó ta tiến hành ping gateway để kiểm tra.

<div align="center">

  <img src="/images/image112.png" alt="Network Topology" width="100%"/>

</div>

Sau đó truy cập vào ip của port1 từ trình duyệt.

<div align="center">

  <img src="/images/image113.png" alt="Network Topology" width="100%"/>

</div>

Ta sẽ đăng ký tài khoảng trên Support Fortinet hoặc có thể chọn các image không yêu cầu Evaluation License từ việc đăng ký tài khoảng trên Fortinet.

Sau khi nhập email và password vào, đợi một chút cho nó reboot lại thì ta sẽ vào được giao diện 

<div align="center">

  <img src="/images/image114.png" alt="Network Topology" width="100%"/>

</div>


Sau khi vào được giao diện ta sẽ tiến hành cấu hình các thông tin như : Hostname, NTP, DNS, ...

Đầu tiên vào System > Settiings

<div align="center">

  <img src="/images/image115.png" alt="Network Topology" width="100%"/>

</div>

Ta tiến hành chỉnh tên Hostname, NTP, timezone, ...

<div align="center">

  <img src="/images/image116.png" alt="Network Topology" width="100%"/>

</div>

phần NTP có thể custom bằng CLI như sau : 

<div align="center">

  <img src="/images/image117.png" alt="Network Topology" width="100%"/>

</div>

Tiếp đến là config DNS : Vào Network > DNS

<div align="center">

  <img src="/images/image118.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image119.png" alt="Network Topology" width="100%"/>

</div>

Vậy là xong phần config basic, tiếp đến là cấu hình LACP

Đầu tiên ta cần hiểu LACP là gì ?

**LACP** (Link Aggregation Control Protocol) là giao thức dùng để gộp nhiều đường link vật lý thành một đường link logic duy nhất. Chuẩn này được định nghĩa trong IEEE 802.3ad (hiện nay nằm trong IEEE 802.1AX).

- Mục đích của LACP
  - Tăng băng thông
  - Dự phòng (redundancy)
  - Load balancing

<div align="center">

  <img src="/images/image120.png" alt="Network Topology" width="100%"/>

</div>

Đầu tiên ta vào Network > interface, sau đó tạo một interface mới.

<div align="center">

  <img src="/images/image121.png" alt="Network Topology" width="100%"/>

</div>

Sau đó nhập vào các thông tin như sau :

<div align="center">

  <img src="/images/image122.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image123.png" alt="Network Topology" width="100%"/>

</div>

Sau đó tiếp tục tạo 2 interface như sau :

<div align="center">

  <img src="/images/image124.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image125.png" alt="Network Topology" width="100%"/>

</div>

Sau khi tạo xong ta thấy được

<div align="center">

  <img src="/images/image126.png" alt="Network Topology" width="100%"/>

</div>

Tiếp đến ta sẽ tiến hành config Switch, Tạo port channel ở Switch

<div align="center">

  <img src="/images/image127.png" alt="Network Topology" width="100%"/>

</div>

Sau đó config mode trunk cho port channel 1

<div align="center">

  <img src="/images/image128.png" alt="Network Topology" width="100%"/>

</div>

Sau đó kiểm tra

<div align="center">

  <img src="/images/image129.png" alt="Network Topology" width="100%"/>

</div>

Tiếp đến config VLAN Access cho 2 port 0/2 và 0/3

<div align="center">

  <img src="/images/image130.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image131.png" alt="Network Topology" width="100%"/>

</div>

Sau đó kiểm tra một lần nữa

<div align="center">

  <img src="/images/image132.png" alt="Network Topology" width="100%"/>

</div>

Sau khi config xong switch thì ta vào CLI của Fortigate để cấu hình LACP mode

<div align="center">

  <img src="/images/image133.png" alt="Network Topology" width="100%"/>

</div>

Sau đó kiểm tra 

<div align="center">

  <img src="/images/image135.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image134.png" alt="Network Topology" width="100%"/>

</div>


Vậy là LACP đã hoạt động, ta tiến hành ping kiểm tra

<div align="center">

  <img src="/images/image136.png" alt="Network Topology" width="100%"/>

</div>

Sau đó ta ngắt 1 dây kiểm tra Failover

<div align="center">

  <img src="/images/image137.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image139.png" alt="Network Topology" width="100%"/>

</div>

<div align="center">

  <img src="/images/image140.png" alt="Network Topology" width="100%"/>

</div>

Tiến hành ping lại

<div align="center">

  <img src="/images/image138.png" alt="Network Topology" width="100%"/>

</div>

Vậy là ping thành công chứng tỏ hoạt động, check trên switch



