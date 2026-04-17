# Cấu hình Tính sẵn sàng cao (HA)

> **High Availability** (HA) Pairing trong Forescout là một giải pháp dự phòng và phục hồi hệ thống được thiết kế để đảm bảo tính sẵn sàng của các dịch vụ, giảm thiểu thời gian ngừng hoạt động (downtime) khi có sự cố hệ thống. Giải pháp này có thể áp dụng cho cả Forescout Platform Appliances và Enterprise Manager.

## Mục lục:
1. [Kiến trúc và Mô hình hoạt động](#1-kiến-trúc-và-mô-hình-hoạt-động)

2. [Cơ chế chuyển đổi dự phòng (Failover)​](#2-cơ-chế-chuyển-đổi-dự-phòng-failover)

3. [Yêu cầu hệ thống và Cấp phép (Licensing)](#3-yêu-cầu-hệ-thống-và-cấp-phép-licensing)

4. [Triển khai](#4-triển-khai)

## 1. Kiến trúc và mô hình hoạt động

Mô hình Active / Standby: HA được triển khai theo một cặp gồm hai thiết bị. Trong đó, node Active (Đang hoạt động) chịu trách nhiệm quản lý các hoạt động khám phá, đánh giá và kiểm soát mạng (NAC). Node Standby (Dự phòng) có hệ điều hành luôn chạy và sẵn sàng tiếp quản hệ thống nếu node Active gặp sự cố.

<div align="center">
  <img src="/images/image249.png" alt="Network Topology" width="100%"/>
</div>

Đồng bộ hóa: Hai thiết bị (node Primary và node Secondary) được đồng bộ hóa dữ liệu với nhau thông qua một cặp cáp kết nối trực tiếp dự phòng (redundant pair of interconnecting cables).

Kết nối giao tiếp: Khi quản trị, bạn sẽ giao tiếp với cặp thiết bị HA thông qua một địa chỉ IP ảo (shared/virtual IP) qua cổng 22/TCP. Nếu cần truy cập trực tiếp vào giao diện dòng lệnh (CLI) của từng thiết bị vật lý riêng biệt trong cặp HA, bạn cần sử dụng cổng 2222/TCP (dấu nhắc lệnh sẽ bắt đầu bằng chữ Miniroot)

## 2. Cơ chế chuyển đổi dự phòng (Failover)

Giám sát trạng thái: Node Active và Standby sẽ liên tục giám sát các bản cập nhật hoạt động của nhau mỗi giây.

Thời gian Failover: Mặc định, quá trình chuyển đổi (failover) sang node Standby sẽ diễn ra sau 60 giây kể từ khi node này phát hiện node Active bị ngừng hoạt động. Node Standby thông thường sẽ mất từ 10 phút trở lên để chính thức trở thành node Active mới.

Các nguyên nhân kích hoạt Failover: Quá trình chuyển đổi dự phòng sẽ được kích hoạt nếu xảy ra một trong các trường hợp sau:

- Lỗi hệ thống: Node Active bị sập nguồn (outage), hoặc hệ thống lưu trữ RAID bị hỏng một số ổ đĩa.
- Bảo trì hệ thống: Node Active bị tắt máy hoặc khởi động "lạnh" (cold boot).
- Lỗi cổng quản lý mạng (Management interface failure) trên node Active (chỉ áp dụng nếu bạn đã thiết lập cấu hình kiểm tra các địa chỉ IP pingable trước đó).
- Kích hoạt thủ công: Quản trị viên chạy lệnh `fstool ha stop -f` trên node Active.

Trường hợp KHÔNG kích hoạt Failover: Việc dịch vụ Forescout bị lỗi hoặc dừng hoạt động (ví dụ: ở cấp độ TCP stack) sẽ không kích hoạt quá trình chuyển đổi failover.

## 3. Yêu cầu hệ thống và Cấp phép (Licensing)

License: Nếu triển khai theo giấy phép **Flexx Licensing**, tính năng HA cho Appliances bắt buộc phải có license **Forescout Platform eyeRecover**. Tuy nhiên, đối với tính năng HA trên Enterprise Manager thì đã được bao gồm sẵn trong license eyeSight

<div align="center">
  <img src="/images/image434.png" alt="Network Topology" width="100%"/>
</div>

Thiết bị không hỗ trợ HA: Các dòng thiết bị sau không được hỗ trợ để chạy HA: `CT-R, VCT-R, 4130, 5110 và Flexx Virtual cấu hình x-small`

Điều kiện chuyển đổi thiết bị thành HA: Để cấu hình hai thiết bị thành một cặp HA, cả hai phải có số lượng cổng Ethernet bằng nhau, dung lượng ổ đĩa của node Secondary tối thiểu phải bằng hoặc lớn hơn node Primary, và cả hai phải chạy cùng một phiên bản Forescout Platform

## 4. Triển khai

Đầu tiên chúng ta cân License **Forescout Platform eyeRecover** để có thể triển khai HA trong mode Flexx.

<div align="center">
  <img src="/images/image251.png" alt="Network Topology" width="100%"/>
</div>


HA là cài từ đầu, không add được từ 1 con standalone nên ta phải cài lại.

<div align="center">
  <img src="/images/image253.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image254.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image255.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image256.png" alt="Network Topology" width="100%"/>
</div>

Ở đây sẽ có 2 mode, thì chúng ta chọn Flexx vì mode này sẽ cho phép license dùng chung cho các node, còn Per Applicationn mode thì mỗi node sẽ có 1 license riêng.

<div align="center">
  <img src="/images/image257.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image258.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image259.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image260.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image261.png" alt="Network Topology" width="100%"/>
</div>



<div align="center">
  <img src="/images/image262.png" alt="Network Topology" width="100%"/>
</div>



<div align="center">
  <img src="/images/image263.png" alt="Network Topology" width="100%"/>
</div>





<div align="center">
  <img src="/images/image264.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image265.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image266.png" alt="Network Topology" width="100%"/>
</div>


<div align="center">
  <img src="/images/image267.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image268.png" alt="Network Topology" width="100%"/>
</div>

Ở đây ta có một lưu ý đó là phải chạy câu lệnh `fstool ha_setup --ha_reset` ở node Active để node Secondary có thể kết nối đến và triển khai HA

<div align="center">
  <img src="/images/image269.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image270.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image271.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image272.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image273.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image274.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image275.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image276.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image277.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image278.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image279.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image280.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image281.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image282.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image283.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image284.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image285.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image286.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image287.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image288.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image289.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image290.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image291.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image292.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image293.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image294.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image295.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image296.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image297.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image298.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image299.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image300.png" alt="Network Topology" width="100%"/>
</div>

Done