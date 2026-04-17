# Cấu hình xác thực cơ bản (802.1X)


Trong các mô hình triển khai NAC hiện đại, 802.1X là giao thức xác thực cốt lõi, cho phép kiểm soát chặt chẽ quyền truy cập vào mạng ngay từ lớp truy cập (access layer). Với 802.1X, mỗi thiết bị khi kết nối vào switch hoặc access point bắt buộc phải được xác thực thông qua máy chủ RADIUS, thường kết hợp với hệ thống danh tính trung tâm như Active Directory.

Cơ chế này giúp đảm bảo rằng:
- Chỉ người dùng/thiết bị hợp lệ mới được phép truy cập mạng
- Có thể áp dụng chính sách truy cập động (VD: VLAN, ACL) tùy theo danh tính
- Tăng khả năng giám sát và phản ứng khi có hành vi bất thường

Forescout hỗ trợ triển khai xác thực 802.1X theo mô hình out-of-band, kết hợp với các thiết bị mạng (switch, controller Wi-Fi) để giám sát trạng thái xác thực và thực thi chính sách một cách gián tiếp thông qua SNMP, RADIUS hoặc API.

Bài viết/lab này sẽ hướng dẫn bạn cấu hình 802.1X cơ bản bao gồm:

- Cấu hình switch làm Authenticator
- Tích hợp máy chủ RADIUS với AD
- Kiểm tra thiết bị đầu cuối (Supplicant) xác thực thành công
- Quan sát trạng thái xác thực trên Forescout

Đây là bước đầu tiên để triển khai mô hình kiểm soát truy cập dựa trên danh tính, đảm bảo an toàn truy cập mạng ngay từ lớp vật lý.

## Mục lục
1. Cấu hình RADIUS trên Forescout​
2. Cấu hình trên Switch Cisco
  2.1 Cấu hình SSH
  2.2 Cấu hình SNMP
  2.3 Cấu hình Radius
3. Test Service Authen 802.1x​
4. Kết luận​

## 1. Cấu hình RADIUS trên Forescout​

Sau khi thêm nguồn xác thực AD như bài viết trước chúng ta sẽ tiếp tục với cấu hình Pre-Admission Authorization
Vào Tools -> Options -> RADIUS -> Pre-Admission Authorization

<div align="center">
  <img src="/images/image458.png" alt="Network Topology" width="100%"/>
</div>

Nhấn Add thêm các Policy giao diện hiện ra như sau

<div align="center">
  <img src="/images/image459.png" alt="Network Topology" width="100%"/>
</div>

Nhấn Add ở trên dòng Condition để thêm điều kiện ở đây mình chọn EAP và MAB

<div align="center">
  <img src="/images/image460.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo bạn có thể gán VLAN sau khi xác thực hoặc là tick vào ô Deny Accees để không cho xác thực.

<div align="center">
  <img src="/images/image461.png" alt="Network Topology" width="100%"/>
</div>

Sau khi bạn add xong có thể đổi priority cho các Rule nếu bạn muốn bằng cách nhấn Move Up hoặc Move Down. Sau khi cấu hình xong giao diện như hình

<div align="center">
  <img src="/images/image462.png" alt="Network Topology" width="100%"/>
</div>

Tới tab RADIUS Settings bạn có thể điều chỉnh các cấu hình của RADIUS Server

<div align="center">
  <img src="/images/image463.png" alt="Network Topology" width="100%"/>
</div>

Bước tiếp theo vào Policy Manager, chọn vào thư mục Policy mà bạn muốn quản lý, Nhấn Add để thêm Policy giám sát

<div align="center">
  <img src="/images/image464.png" alt="Network Topology" width="100%"/>
</div>

Giao diện Create Policy mở ra bạn hãy chọn template RADIUS -> 802.1x Enforcement -> Endpoint Authorization. Nhấn next

<div align="center">
  <img src="/images/image465.png" alt="Network Topology" width="100%"/>
</div>

Bước 2 là sẽ nhập tên của Policy

<div align="center">
  <img src="/images/image466.png" alt="Network Topology" width="100%"/>
</div>

Ở bước 3 chúng ta chọn cách các thiết bị endpoint xác thực. Có 3 ô tích phân biệt kĩ hơn các kiểu xác thực bao gồm : 

- Phân biệt giữa máy và người dùng
- Phân biệt giữa chứng chỉ và tài khoản
- Phân biệt truy cập bằng MAC (MAB)

<div align="center">
  <img src="/images/image467.png" alt="Network Topology" width="100%"/>
</div>

Cuối cùng sẽ là xem lại cái Sub-Rules mình vừa tạo

<div align="center">
  <img src="/images/image468.png" alt="Network Topology" width="100%"/>
</div>

Done 
<div align="center">
  <img src="/images/image469.png" alt="Network Topology" width="100%"/>
</div>

## 2. Cấu hình trên Switch Cisco

### 2.1 Cấu hình SSH

```bash
ip domain name <dommain name>
username admin privilege 15 secret <passsword>
crypto key generate rsa modulus 2048 
ip ssh version 2
line vty 0 15
    transport input ssh
    login local
    privilege level 15
    exit
```

### 2.2 Cấu hình SNMP

```bash
snmp-server view FS iso included
snmp-server view FS internet included
snmp-server view FS interfaces included
snmp-server view FS ip included
snmp-server view FS dot1dBridge included
snmp-server view FS 1.3.6.1.2.1.17.7 included
snmp-server view FS 1.3.6.1.2.1.17.7.1.4 included
snmp-server group FS_GROUP v3 priv read FS write FS
snmp-server user <username> FS_GROUP v3 auth md5 <auth password> priv aes 128 <priv password>
snmp-server host <fs ip> version 3 priv <username>
snmp-server enable traps
```
### 2.3 Cấu hình Radius

```bash
aaa new-model
aaa authentication login default local
radius server <radiusname>
    address ipv4 <ip radius server> auth-port <port-auth> acct-port <port-acct>
    key <share-key>
    exit
radius-server dead-criteria time 10 tries 1
radius-server source-ports extended
radius-server deadtime 30 
radius-server attribute 32 include-in-access-req
radius-server vsa send cisco-nas-port
aaa group server radius <groupname>
    server name <radiusname>
    exit
aaa authentication dot1x default group <groupname>
aaa authorization network default group <groupname>
aaa accounting dot1x default start-stop group <groupname>
dot1x system-auth-control
aaa server radius dynamic-author
    exit
aaa session-id common

eap profile <profile name>
    method peap
    exit
int <interface>
    switchport mode access
    switchport access vlan <vlan id>
    dot1x authenticator profile name FS
    dot1x pae authenticator
    dot1x timeout tx-period 15
    dot1x max-req 3
    dot1x max-reauth-req 3
    dot1x timeout auth-period 60
    authentication event fail action authorize vlan <vlan id>
    authentication event no-response action authorize vlan <vlan id>
    authentication order dot1x
    authentication priority dot1x
    authentication port-control auto
    authentication periodic
    authentication timer reauththenticate 60
```

Tiếp đến add Switch như các bài trước và ở 802.1x, nhập secret là key ở phần radius server đã nhập khi nãy.

<div align="center">
  <img src="/images/image470.png" alt="Network Topology" width="100%"/>
</div>

Tiến hành vào máy win client, vào cmd nhập `services.msc` và tìm service Wired AutoConfig, start nó lên

<div align="center">
  <img src="/images/image471.png" alt="Network Topology" width="100%"/>
</div>

Sau đó vào Control Panel > Network and Internet > Network Connections > Chọn Ethernet > Chuột phải Properties.

<div align="center">
  <img src="/images/image472.png" alt="Network Topology" width="100%"/>
</div>

Click vào phần settings sau đó un check Verify the server's identity

<div align="center">
  <img src="/images/image473.png" alt="Network Topology" width="100%"/>
</div>

Nhấp oke và tiếp tục vào phần Additional Settings

<div align="center">
  <img src="/images/image474.png" alt="Network Topology" width="100%"/>
</div>

Nhấn vào phần credentials và tiến hành nhập tài khoảng đã tạo

<div align="center">
  <img src="/images/image475.png" alt="Network Topology" width="100%"/>
</div>

Sau khi nhập vào xong ta đợi nó authentication

<div align="center">
  <img src="/images/image476.png" alt="Network Topology" width="100%"/>
</div>

Done
## 3. Test Service Authen 802.1x​

Vào policy kiểm tra ta thấy được đã hiện user xác thực.

<div align="center">
  <img src="/images/image477.png" alt="Network Topology" width="100%"/>
</div>

Done

## 4. Kết luận

- Việc triển khai xác thực 802.1X là một bước quan trọng trong việc xây dựng hệ thống kiểm soát truy cập mạng chặt chẽ, giúp đảm bảo rằng chỉ người dùng và thiết bị hợp lệ mới được phép truy cập vào hệ thống.
- Thông qua sự kết hợp giữa Switch, RADIUS Server và Active Directory, quá trình xác thực trở nên tập trung, dễ quản lý và có thể gắn chính sách truy cập theo danh tính người dùng, nhóm hoặc thiết bị.
- Forescout hỗ trợ giám sát trạng thái xác thực 802.1X theo mô hình out-of-band, giúp theo dõi, ghi nhận và áp dụng chính sách mà không ảnh hưởng trực tiếp đến luồng lưu lượng, đồng thời hỗ trợ xử lý các tình huống vi phạm như cách ly hoặc chặn thiết bị không tuân thủ.
- Đây là nền tảng cốt lõi để triển khai các mô hình NAC nâng cao hơn như Zero Trust, phân vùng động theo người dùng, và phản ứng tự động khi phát hiện thiết bị bất thường.