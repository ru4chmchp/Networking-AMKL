```bash
aaa new-model

! --- 1. ĐỊNH NGHĨA RADIUS SERVER ---
radius-server vsa send 
radius server <radiusname>
 address ipv4 <ip radius server> auth-port 1812 acct-port 1813
 key <share-key>
 exit

! --- 2. GÔM SERVER VÀO GROUP ---
aaa group server radius <groupname>
 server name <radiusname>
 exit

! --- 3. CẤU HÌNH AAA (AUTHENTICATION / AUTHORIZATION / ACCOUNTING) ---
! Đăng nhập Switch: Ưu tiên local, sập thì dùng group
aaa authentication login default local group <groupname>

! Sửa lỗi % Error in authentication khi gõ lệnh 'enable'
aaa authentication enable default enable group <groupname>

! Cấp quyền User (Shell Access)
aaa authorization exec default local group <groupname> 

! Xác thực và Cấp quyền cho thiết bị cắm đầu cuối (802.1X / MAC-Auth)
aaa authentication dot1x default group <groupname>
aaa authorization network default group <groupname>
aaa accounting dot1x default start-stop group <groupname>

! --- 4. KÍCH HOẠT 802.1X VÀ COA (DYNAMIC AUTHORIZATION) ---
dot1x system-auth-control

aaa server radius dynamic-author
 client <CLEARPASS_VIP> server-key <SHARED_KEY>
 port 3799
 auth-type all
 exit

! --- 3. ÁP DỤNG VÀO CÁC ĐƯỜNG VTY (SSH ACCESS) ---
line vty 0 4
 ! Sử dụng danh sách AAA default để xác thực (Đã sửa ở phần trước)
 login authentication default
 
 ! BẮT BUỘC: Thêm dòng này để đồng bộ quyền Privilege 15 khi đăng nhập thành công
 authorization exec default
 
 ! Chỉ cho phép kết nối qua SSH, chặn hoàn toàn Telnet để bảo mật
 transport input ssh
 exit
enable secret level 15 0 <password1>
```




```bash

! --- 1. CẤU HÌNH ĐIỀU KIỆN CẦN ĐỂ TẠO KEY SSH ---
ip domain-name <domain_name>

! Tạo tài khoản admin local có quyền cao nhất (Privilege 15)
username admin privilege 15 secret <password>

! --- 2. TẠO KEY RSA VÀ BẬT SSH VERSION 2 ---
! Lưu ý: Nếu Switch hỏi có muốn ghi đè key cũ không, hãy chọn 'yes'
crypto key generate rsa modulus 2048
ip ssh version 2
```




``bash
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