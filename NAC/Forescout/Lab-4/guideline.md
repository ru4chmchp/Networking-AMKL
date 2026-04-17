# Cấu hình thêm các thiết bị Network Device vào NAC​

Sau khi triển khai thành công Forescout CounterACT lên hạ tầng ảo hóa, bước tiếp theo trong quá trình xây dựng hệ thống NAC là kết nối các thiết bị mạng (Network Devices) – như switch, router, firewall, và nền tảng ảo hóa vSphere (ESXi/vCenter) – vào hệ thống Forescout để bắt đầu thu thập thông tin và kiểm soát truy cập.

Việc cấu hình Network Devices là bước thiết yếu để Forescout có thể:
- Giao tiếp với thiết bị hạ tầng qua các giao thức như SNMP, SSH, Telnet
- Tích hợp với vCenter Server để phát hiện và kiểm soát máy ảo đang hoạt động trong môi trường ESXi
- Thu thập thông tin kết nối (MAC, IP, cổng switch, VM-Host mapping)
- Điều khiển hành vi thiết bị đầu cuối (VD: chuyển VLAN, cách ly, chặn truy cập)

Bài Lab này sẽ hướng dẫn cách thêm các thiết bị mạng và hệ thống vSphere vào hệ thống NAC, kiểm tra trạng thái kết nối, và đảm bảo rằng Forescout có thể giám sát đầy đủ thông tin cần thiết để phục vụ cho các chính sách kiểm soát truy cập ở các lab tiếp theo.

## Mục lục
1. [Mô hình bài Lab]()

2. [Cấu hình Add Switch Cisco Catalyst 9000v vào Forescout]()

3. [Cấu hình Add vCenter vào Forescout]()

4. [Kết luận]()

## 1. Mô hình bài Lab

## 2.Cấu hình Add Switch Cisco Catalyst 9000v vào Forescout

Để add 1 switch vào Forescout thì ta phải cấu hình remote access, ở bài này thì đó là ssh cho switch và tiếp theo là cấu hình SNMP trên switch, chúng ta sẽ cấu hình SNMPv3 để tăng tính bảo mật. Và cuối cùng là cấu hình span port để mirror lưu lượng về Forescout
 
```bash
Switch> enable
Switch# configure terminal
Switch(config)# monitor session 1 source interface Gi1/0/7 - 8 both
Switch(config)# monitor session 1 destination interface Gi1/0/6 encapsulation replicate 
Switch(config)# end
Switch# wr
```

Kiểm tra

<div align="center">
  <img src="/images/image335.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến là cấu hình SNMPv3

```bash
Switch> enable
Switch# configure terminal
Switch(config)# snmp-server view TREAD 1.3.6.1 included
Switch(config)# snmp-server view TREAD 1.0.8802.1.1.2 included
Switch(config)# snmp-server view TWRITE 1.3.6.1 included
Switch(config)# snmp-server view TWRITE 1.0.8802.1.1.2 included
Switch(config)# snmp-server group TGROUP v3 auth read TREAD write TWRITE
Switch(config)# snmp-server user forescout TGROUP v3 auth sha <auth-password> priv aes 128 <privacy-password>
Switch(config)# snmp-server host <ip fs> version 3 auth forescout
Switch(config)# snmp-server enable traps
Switch(config)# end
Switch# wr
```

Thay <auth-password> và <privacy-password> bằng mật khẩu thực tế bạn nhập trên ForeScout (trong bước SNMP). <ip fs> là ip của ForeScout

Trên Forescout chúng ta vào Option -> Switch -> Add

<div align="center">
  <img src="/images/image336.png" alt="Network Topology" width="100%"/>
</div>

Cửa sổ "Select Test Method" hiện ra cho phép bạn chọn cách ForeScout kiểm tra kết nối với switch mới:
- By profile - use a profile to provide common switch management settings: Sử dụng một "profile" (hồ sơ) đã được định nghĩa trước. Profile này chứa các thiết lập chung (như thông tin đăng nhập CLI, SNMP, v.v.) để áp dụng cho nhiều switch.​
- Non-profile - manually enter all switch management settings: Không dùng profile, mà sử dụng các thiết lập thủ công.​

Bước tiếp theo nhập IP quản trị của Cisco Catalyst 9000v

<div align="center">
  <img src="/images/image337.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo là bước cấu hình thông tin đăng nhập để ForeScout truy cập vào Command Line Interface (CLI) của switch

Trước tiên ở Switch ta setuo SSH
```bash
Switch> enable
Switch# configure terminal
Switch(config)# username admin privilege 15 password 0 <matkhau_cua_ban>
Switch(config)# ip domain name intern.local
Switch(config)# crypto key generate rsa modulus 2048
Switch(config)# ip ssh version 2
Switch(config)# line vty 0 4
Switch(config)# login local
Switch(config)# transport input ssh
Switch(config)# end
Switch# wr
```

<div align="center">
  <img src="/images/image338.png" alt="Network Topology" width="100%"/>
</div>

Kế tiếp là bước cấu hình thông tin SNMP để ForeScout giao tiếp với Switch. SNMP có 3 version SNMPv1, SNMPv2 và SNMPv3 để tăng tính Secure ta nên dùng SNMPv3, ta nhập user giống với khai báo trên switch. Sau đó chọn thuật toán ở phần Authentication là SHA giống với cấu hình SNMPv3 ví dụ ở trên sau đó nhập password. Còn ở phần Privacy ta chọn thuật toán AES-128 sau đó nhập password rồi nhấn Next.

<div align="center">
  <img src="/images/image339.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo là bước Permissio. Ở bước "Add Switch" phần Permissions, ta cấu hình các quyền cho ForeScout khi quản lý switch. Ở đây, ta chọn toàn bộ quyền để:
- Discovery Permissions: Cho phép switch phát hiện các switch khác trong mạng qua các giao thức CDP, FDP, và LLDP.
- MAC Permissions: Xem được địa chỉ MAC của các thiết bị đầu cuối (endpoints) kết nối với switch, đồng thời thực hiện các hành động như chặn cổng (block port), áp dụng ACL, hoặc gán VLAN (Assign VLAN).
- ARP Permissions: Xem thông tin bảng ARP của switch để ánh xạ IP-MAC, và thực hiện hành động xóa các địa chỉ IP dư thừa liên kết với địa chỉ MAC.

Trong phần Advanced, ta có thể điều chỉnh các thông số như thời gian lấy thông tin MAC, ARP, hoặc dữ liệu từ CDP, FDP, LLDP. Sau khi chọn quyền, nhấn "Next" để tiếp tục.


<div align="center">
  <img src="/images/image340.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image341.png" alt="Network Topology" width="100%"/>
</div>

Tiếp đến phần ACL,SGT,802.1X ta chưa cấu hình nên bỏ qua


<div align="center">
  <img src="/images/image342.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image343.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image344.png" alt="Network Topology" width="100%"/>
</div>

oke nhấn Finish, ở đây có một lưu ý là chúng ta phải kiểm tra Module Switch đã bật chưa nhá, nếu chưa thì phải Start : Vào Options > Modules > Network > Switch không là nó hiện ARP Pending

<div align="center">
  <img src="/images/image347.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image346.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image345.png" alt="Network Topology" width="100%"/>
</div>


## 3. Cấu hình Add vCenter vào Forescout

## 4. Kết luận
- Việc cấu hình và tích hợp các thiết bị mạng như switch, router, firewall và hạ tầng ảo hóa vSphere (vCenter/ESXi) vào hệ thống Forescout là bước quan trọng giúp NAC thu thập đầy đủ thông tin về các điểm kết nối trong mạng, phục vụ cho việc phát hiện thiết bị và thực thi chính sách kiểm soát truy cập.
- Nhờ kết nối với các thiết bị này, Forescout có thể giám sát chính xác vị trí thiết bị đầu cuối (port switch, VM host), đưa ra hành động kiểm soát như chuyển VLAN, cách ly, hoặc cảnh báo khi có vi phạm chính sách bảo mật.
- Đây là nền tảng để triển khai các chức năng nâng cao hơn trong các lab tiếp theo như: phát hiện thiết bị theo profile, xác thực người dùng, kiểm tra trạng thái bảo mật của endpoint, và phản ứng tự động khi có sự cố.
- Tích hợp đầy đủ hạ tầng mạng giúp tăng hiệu quả giám sát và kiểm soát, đồng thời tận dụng tối đa sức mạnh của nền tảng NAC Forescout trong việc bảo vệ mạng doanh nghiệp.

