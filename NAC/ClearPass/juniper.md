

ge-0/0/0 (Cổng vật lý)
   └── unit 0 (Giao diện logic phụ trách)
          └── family ethernet-switching (Chạy tính năng Layer 2 / Switchport)
                 ├── interface-mode trunk (Chế độ Trunking)
                 └── vlan members [ VLAN_10 VLAN_20 ] (Các VLAN được đi qua)


! =====================================================================
! 1. CẤU HÌNH LAYER 2: VLAN, PORT ACCESS VÀ PORT TRUNK
! =====================================================================
# Tạo các VLAN hệ thống và VLAN Quản trị
set vlans VLAN_MGMT vlan-id 100
set vlans VLAN_USER vlan-id 200

# Cấu hình cổng Access (Cắm PC/Thiết bị của người dùng vào cổng ge-0/0/1)
set interfaces ge-0/0/1 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/1 unit 0 family ethernet-switching vlan members VLAN_USER

# Cấu hình cổng Trunk (Kết nối Uplink sang Switch khác hoặc Router tại cổng ge-0/0/0)
# Từ khóa "members all" cho phép tất cả VLAN đi qua đường Trunk này
set interfaces ge-0/0/0 unit 0 family ethernet-switching interface-mode trunk
set interfaces ge-0/0/0 unit 0 family ethernet-switching vlan members all


! =====================================================================
! 2. CẤU HÌNH LAYER 3: IP MANAGEMENT (Tương đương Interface Vlan bên Cisco)
! =====================================================================
# Đặt địa chỉ IP cho Giao diện logic irb.100
set interfaces irb unit 100 family inet address 172.16.100.200/24

# Liên kết interface irb.100 vào VLAN_MGMT để kích hoạt cổng IP này
set vlans VLAN_MGMT l3-interface irb.100

# Cấu hình Default Gateway để Switch có thể giao tiếp liên mạng (Đi Router/ClearPass)
set routing-options static route 0.0.0.0/0 next-hop 172.16.100.1

! =====================================================================
! 3. BẬT DỊCH VỤ HỆ THỐNG (MẬT KHẨU ROOT VÀ SSH)
! =====================================================================
# Đặt mật khẩu Root (Bắt buộc phải có thì Juniper mới cho phép gõ lệnh commit)
set system root-authentication plain-text-password
! (Sau lệnh này, bạn tự nhập mật khẩu quản trị và xác nhận lại trên màn hình)


# Bật dịch vụ SSH Version 2 và cấu hình bảo mật
set system services ssh protocol-version v2
set system services ssh root-login allow
set system services ssh connection-limit 5


# DHCP relay
set forwarding-options dhcp-relay server-group DHCP-relay <CLEARPASS_VIP>			
set forwarding-options dhcp-relay active-server-group DHCP-relay			
set forwarding-options dhcp-relay group DHCP-relay interface irb.10	

! =====================================================================
! 4. CẤU HÌNH RADIUS - NHÁNH 1: XÁC THỰC SSH QUẢN TRỊ SWITCH
! =====================================================================
# Khai báo thông tin Server RADIUS cho dịch vụ hệ thống
set system radius-server 172.16.100.10 port 1812
set system radius-server 172.16.100.10 accounting-port 1813
set system radius-server 172.16.100.10 secret "SHARED_KEY"
set system radius-server 172.16.100.10 source-address 172.16.100.200

# Cấu hình thứ tự hỏi pass: Ưu tiên RADIUS trước, nếu RADIUS sập mới dùng Pass Local
set system authentication-order password
set system authentication-order radius

## Bản đồ phân quyền: Cấp quyền super-user (toàn quyền) cho tài khoản từ xa được ClearPass chấp nhận
set system login user remote class super-user

! 5


set access radius-server <CLEARPASS_VIP> secret <SHARED_KEY>			
set access radius-server <CLEARPASS_VIP> source-address <IP_SwitchJuniper>			
			
set access profile CPPM accounting-order radius			
set access profile CPPM authentication-order radius			
set access profile CPPM radius authentication-server <CLEARPASS_VIP>			
set access profile CPPM radius accounting-server <CLEARPASS_VIP>			
set protocols dot1x authenticator authentication-profile-name CPPM