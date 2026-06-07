# Cấu hình HPS Inspection với agent và agentless trên Forescout​

Để đảm bảo khả năng giám sát và kiểm soát thiết bị đầu cuối một cách toàn diện, Forescout sử dụng HPS Inspection Engine (Host Property Scanner) – một thành phần thuộc Endpoint Module, chuyên thu thập thông tin chi tiết từ các máy tính chạy hệ điều hành Windows. Thông tin này bao gồm: phiên bản hệ điều hành, domain, phần mềm đã cài, trạng thái antivirus, bản vá, dịch vụ đang chạy, v.v.

HPS có thể hoạt động theo hai chế độ:

- Agent-based (có cài agent) – sử dụng SecureConnector, cho phép thu thập thông tin thiết bị một cách chủ động, liên tục, và có thể thực thi các hành động nâng cao trực tiếp trên endpoint.
- Agentless (không cài agent) – sử dụng cơ chế Remote Inspection, truy vấn thiết bị Windows qua các giao thức như WMI, Remote Registry, RPC/SMB. Phù hợp cho hệ thống không muốn triển khai phần mềm trên máy trạm.

Mỗi phương pháp đều có ưu điểm riêng:

- Agent-based phù hợp với môi trường kiểm soát chặt, yêu cầu giám sát liên tục
- Agentless phù hợp với môi trường linh hoạt, không muốn tác động đến người dùng cuối

Trong bài lab này, bạn sẽ thực hành:

- Cấu hình HPS Inspection cho cả hai phương án: SecureConnector và Remote Inspection
- Kiểm tra và so sánh khả năng thu thập thông tin ở cả hai mô hình
- Xử lý một số lỗi thường gặp trong quá trình truy vấn

Đây là bước quan trọng để giúp hệ thống NAC hoạt động hiệu quả, đáp ứng cả yêu cầu bảo mật và linh hoạt khi triển khai trong nhiều môi trường doanh nghiệp khác nhau.

## Mục lục:
1. Giới thiệu công cụ HPS Inspection trong Forescout​
1.1 Khái niệm​
1.2 Chức năng chính của HPS Inspection​
1.3 Chế độ hoạt động​
1.4 Tùy chọn cấu hình​
2. Cấu hình SecureConnector (Agent Forescout)​
2.1 Cấu hình HPS Inspection Engine​
2.2 Cài đặt SecureConnector​
2.3 Gỡ cài đặt SecureConnector​
3. Cấu hình Remote Inspection (Agentless)​
3.1 Cấu hình Remote Inspection cho các máy tính WORKGROUP​
3.2 Cấu hình Remote Inspection cho các máy tính Join Domain​
4. Kết luận​
​
## 1. Giới thiệu công cụ HPS Inspection trong Forescout​

### 1.1 Khái niệm​

HPS Inspection Engine (Host Property Scanner) là một thành phần quan trọng của Forescout Endpoint Module, cho phép nền tảng Forescout truy cập, thu thập thông tin và quản lý toàn diện các thiết bị đầu cuối chạy hệ điều hành Microsoft Windows. Đây là một trong những mô-đun cơ sở giúp tăng cường khả năng giám sát và kiểm soát mạng của giải pháp Forescout.

<div align="center">
  <img src="/images/image572.png" alt="Network Topology" width="100%"/>
</div>

### 1.2 Chức năng chính của HPS Inspection

Thông qua plugin HPS, nền tảng Forescout eyeSight có thể:

- Kết nối và thu thập thông tin từ các điểm cuối Windows;
- Phân loại và đánh giá chức năng mạng của các thiết bị đầu cuối;
- Thực hiện kiểm tra sâu (deep inspection) nhằm khai thác chi tiết hệ điều hành, trạng thái bảo mật, dịch vụ đang chạy, ứng dụng được cài đặt và nhiều thuộc tính hệ thống khác.

Bên cạnh đó, Forescout eyeControl sử dụng HPS để thực thi các hành động như kiểm soát truy cập, khắc phục sự cố và quản lý bảo mật tại các điểm cuối.

### 1.3 Chế độ hoạt động

HPS Inspection hỗ trợ hai chế độ thu thập thông tin:

- Agentless: Thông qua tính năng Remote Inspection, hệ thống có thể truy vấn các thiết bị Windows nằm trong các phân loại:
    - Windows Manageable Domain (Current)
    - Windows Manageable Local
Việc truy vấn được thực hiện qua các giao thức quản lý từ xa như SSH, MS-WMI và MS-RRP mà không cần cài đặt phần mềm lên điểm cuối.

- Agent-based: Sử dụng SecureConnector, một ứng dụng nhẹ được cài trên điểm cuối để:
    - Gửi thông tin chi tiết về thiết bị về lại hệ thống Forescout;
    - Hỗ trợ thực hiện các hành động kiểm soát hoặc khắc phục từ xa.

* Lưu ý: Khi sử dụng SecureConnector, cần đảm bảo mở cổng TCP 10003 để thiết bị có thể thiết lập kênh kết nối tunnel về Forescout appliance.

### 1.4 Tùy chọn cấu hình 

Người quản trị có thể cấu hình HPS Inspection Engine cho nhiều mục đích khác nhau, bao gồm:

- Thêm hoặc cập nhật thông tin xác thực truy cập domain Windows;
- Cấu hình các thông số cài đặt chung cho SecureConnector;
- Xác định tùy chọn khi triển khai hành động Start Windows Updates;
- Thiết lập phương pháp phân giải và giá trị mặc định cho các tham số;
- Xác định chi tiết các phương thức kiểm tra và các tuỳ chọn kết nối cần sử dụng.

## 2. Cấu hình SecureConnector (Agent Forescout)​

### 2.1 Cấu hình HPS Inspection Engine​
Đầu tiên các bạn vào Tools -> Option -> HPS Inspection Engine giao diện hiện ra như sau

Tab Remote Inspection

<div align="center">
  <img src="/images/image573.png" alt="Network Topology" width="100%"/>
</div>

- Domain Credentials

    - Danh sách tài khoản quản trị: Đây là các tài khoản có quyền admin local/domain, dùng để Forescout kết nối và thao tác với endpoint trong các domain hoặc workgroup khác nhau.
    - Add/Edit/Remove: Thêm, sửa, xóa thông tin tài khoản quản trị.

- Endpoint Remote Inspection method: Phương thức kiểm tra endpoint từ xa. Có 3 mode là

    - Using MS-WMI: Sử dụng dịch vụ Windows Management Instrumentation để giao tiếp và thu thập thông tin từ endpoint.
    - Using MS-RRP (Remote Registry): Sử dụng dịch vụ Remote Registry để truy vấn và thu thập thông tin cấu hình hệ thống từ registry của endpoint
    - Kết hợp cả hai, tùy theo khả năng endpoint hỗ trợ

- When Remote Inspection uses MS-WMI, run scripts with

    - MS-WMI: Chạy script qua dịch vụ WMI (không hỗ trợ script tương tác trên mọi endpoint).
    - CounterACT fsprocsvc: Dùng dịch vụ riêng của Forescout (fsprocsvc.exe), tải về máy đích để chạy script tương tác, hỗ trợ nhiều tác vụ hơn như update AV, diệt virus, v.v


- Authentication Method (Phương thức xác thực):

    - NTLMv1: Phiên bản cũ, bảo mật yếu, dễ bị tấn công (như pass-the-hash, relay attack). Nên vô hiệu hóa hoàn toàn trong môi trường hiện đại.
    - NTLMv2: Cải tiến hơn NTLMv1, hỗ trợ mã hóa mạnh hơn và xác thực phiên tốt hơn. Tuy nhiên, vẫn không có xác thực hai chiều như Kerberos, nên chỉ nên dùng khi Kerberos không khả dụng.
    - Kerberos: Giao thức xác thực mặc định trong môi trường domain (Active Directory), bảo mật cao hơn, hỗ trợ xác thực hai chiều, ít bị tấn công hơn. Nên ưu tiên sử dụng Kerberos thay vì NTLM nếu có thể


- Minimum SMB Protocol Version (Phiên bản giao thức SMB tối thiểu):

    - SMBv3.1.1: Chỉ cho phép endpoint hỗ trợ SMB từ phiên bản 3.1.1 trở lên, giảm rủi ro bảo mật.

- Require SMB signing:

    - Nếu bật, endpoint phải bật SMB signing mới được quản lý từ xa, tăng bảo mật chống tấn công dạng relay trên SMB.

Chuyển qua tab SecureConector

<div align="center">
  <img src="/images/image574.png" alt="Network Topology" width="100%"/>
</div>

- Upgrade Mode:

    - Automatically upgrade Windows endpoints managed by SecureConnector to current SecureConnector version: Nếu bật, các endpoint Windows sẽ tự động được nâng cấp lên phiên bản SecureConnector mới nhất do CounterACT quản lý.

- Actions:

    - Automatically run SecureConnector when using the Disable External Device action: Khi thực hiện hành động chặn thiết bị ngoại vi (Disable External Device), SecureConnector sẽ tự động được chạy trên endpoint. Đây là yêu cầu bắt buộc để thực hiện hành động này.
    - Automatically run SecureConnector on Windows endpoints to increase frequency of Kill Process and other Kill application actions: Cho phép SecureConnector tự động chạy để tăng tần suất thực thi các hành động như Kill Process (kết thúc tiến trình) hoặc các hành động Kill ứng dụng khác trên endpoint.
    - Automatically run SecureConnector when using the Disable Dual Homed action: Khi thực hiện hành động chặn Dual Homed (thiết bị có nhiều kết nối mạng), SecureConnector sẽ tự động được kích hoạt. Đây cũng là yêu cầu bắt buộc để thực hiện hành động này.
    - Show balloon messages at desktop: Hiển thị thông báo dạng balloon trên desktop người dùng khi SecureConnector thực thi các hành động.


- Detection

    - Use SecureConnector to learn MAC addresses from the local ARP table: Nếu bật, SecureConnector sẽ lấy thông tin địa chỉ MAC từ bảng ARP cục bộ trên endpoint để hỗ trợ nhận diện thiết bị.


- SecureConnector Password Protection

    - SecureConnector Password Protection / Retype: Thiết lập và xác nhận mật khẩu bảo vệ cho SecureConnector, giúp ngăn chặn việc gỡ bỏ hoặc thay đổi cấu hình trái phép trên endpoint.

- Alternative Appliances

    - Additional Appliance Connections: Khai báo thêm các kết nối đến các thiết bị CounterACT Appliance khác (nếu có) để tăng tính sẵn sàng hoặc dự phòng.

- Client-Server Connection

    - Minimum supported TLS version: Chọn phiên bản TLS tối thiểu cho kết nối bảo mật giữa SecureConnector và CounterACT server (ở đây là TLS 1.2).
    - CounterACT server verifies SecureConnector client certificate chain: Nếu bật, CounterACT sẽ xác thực chuỗi chứng chỉ của SecureConnector để tăng bảo mật.
    - Check SecureConnector client certificate revocation status: Kiểm tra trạng thái thu hồi chứng chỉ của SecureConnector (ở đây không bật).
    - Additional CDPs for CRL: Khai báo thêm các đường dẫn đến danh sách thu hồi chứng chỉ (CRL).
    - Soft-fail OCSP requests: Nếu bật, sẽ cho phép bỏ qua lỗi OCSP nhẹ khi kiểm tra trạng thái chứng chỉ.

Tiếp theo là tab Test (Tab khác để mặc định): Nhập địa chỉ endpoint mà bạn muốn test. Nhấn Apply và Test để kiểm tra

<div align="center">
  <img src="/images/image575.png" alt="Network Topology" width="100%"/>
</div>

### 2.2 Cài đặt SecureConnector

Tiến hành cài đặt SecureConnector: Vào link https://<ip-fs>/sc.jsp để tải SecureConnector về máy endpoint. Giao diện như sau

<div align="center">
  <img src="/images/image576.png" alt="Network Topology" width="100%"/>
</div>


Chọn platform mà endpoint sử dụng. Ở đây mình dùng Window
Giải thích chi tiết các mode cài:
   - Permanent As Service:
       - SecureConnector chạy như một dịch vụ hệ thống (Windows Service, macOS Daemon), khởi động cùng hệ điều hành, không phụ thuộc vào phiên đăng nhập người dùng.
       - Thường dùng cho các môi trường cần bảo vệ liên tục, hoặc với các thiết bị không phải lúc nào cũng có người dùng đăng nhập.
   - Permanent As Application:
       - SecureConnector chạy như một ứng dụng thông thường, chỉ hoạt động khi người dùng đăng nhập.
       Dễ dàng tương tác, có thể hiện biểu tượng trên systray, phù hợp cho môi trường người dùng cuối.
   - Dissolvable:
       - SecureConnector chỉ tồn tại tạm thời, tự động xóa sau khi hoàn thành nhiệm vụ.
       Phù hợp cho các kịch bản kiểm tra nhanh, không muốn để lại agent trên máy
Sau khi tải xuống nó sẽ hiện thông báo popup

<div align="center">
  <img src="/images/image577.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image578.png" alt="Network Topology" width="100%"/>
</div>

Sau khi cài Agent bạn có thể quản lý được endpoint

<div align="center">
  <img src="/images/image579.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image580.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image581.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image582.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image583.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image584.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image585.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image586.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image587.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image588.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image589.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image590.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image591.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image592.png" alt="Network Topology" width="100%"/>
</div>
<div align="center">
  <img src="/images/image593.png" alt="Network Topology" width="100%"/>
</div>

### 2.3 Gỡ cài đặt SecureConnector

Vào đường dẫn cài đặt SecureConnector chọn shortcut Uninstall

<div align="center">
  <img src="/images/image594.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image595.png" alt="Network Topology" width="100%"/>
</div>

Khi bạn cấu hình HPS Inspection Engine, bạn có thể bật bảo vệ bằng mật khẩu cho SecureConnector trên các endpoint. Khi bảo vệ bằng mật khẩu được bật, User cố gắng Gỡ cài đặt SecureConnector sẽ được nhắc nhập mật khẩu.

<div align="center">
  <img src="/images/image596.png" alt="Network Topology" width="100%"/>
</div>

Hoặc bạn có thể dùng CLI để uninstall với lệnh

```powershell

SecureConnector.exe -uninstall [-silent] [-p <password>]

```

## 3. Cấu hình Remote Inspection (Agentless)​

Remote Inspection sử dụng MS-WMI, MS-RRP và các giao thức thông dụng để quản lý thiết bị windows

### 3.1 Cấu hình Remote Inspection cho các máy tính WORKGROUP​

Thêm user được cấu hình trong HPS Engine Inspector để forescout có thể Remote Execute được vào máy tính không có Agent

<div align="center">
  <img src="/images/image597.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image598.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image598.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image600.png" alt="Network Topology" width="100%"/>
</div>

Đặt mật khẩu giống như HPS inspection engine

<div align="center">
  <img src="/images/image601.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image602.png" alt="Network Topology" width="100%"/>
</div>

Mở các rule FW để có thể Remote

<div align="center">
  <img src="/images/image603.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image604.png" alt="Network Topology" width="100%"/>
</div>

Mở Rule WMI

<div align="center">
  <img src="/images/image605.png" alt="Network Topology" width="100%"/>
</div>

Mở rule Remote Service

<div align="center">
  <img src="/images/image606.png" alt="Network Topology" width="100%"/>
</div>

Vào service chạy các Service WMI Remote Registry và các Service quan trọng khác

<div align="center">
  <img src="/images/image607.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image608.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image609.png" alt="Network Topology" width="100%"/>
</div>

<div align="center">
  <img src="/images/image610.png" alt="Network Topology" width="100%"/>
</div>

Kiểm tra dịch vụ remote

<div align="center">
  <img src="/images/image611.png" alt="Network Topology" width="100%"/>
</div>

Cuối cùng là mở các cấu hình mạng chia sẻ tài nguyên

<div align="center">
  <img src="/images/image612.png" alt="Network Topology" width="100%"/>
</div>

### 3.2 Cấu hình Remote Inspection cho các máy tính Join Domain​
Bạn làm tương tự phần 1 (Workgroup) với Nhóm user trong Domain nhé ^^


## 4. Kết luận

- HPS Inspection Engine là một trong những thành phần cốt lõi giúp Forescout NAC thu thập thông tin chi tiết từ các endpoint Windows, phục vụ cho việc phân loại, đánh giá tuân thủ và áp dụng chính sách bảo mật phù hợp.
- Việc triển khai hai phương thức thu thập – agent (SecureConnector) và agentless (Remote Inspection) – mang lại sự linh hoạt tối đa cho doanh nghiệp:

    - Với agent, bạn có khả năng giám sát liên tục, tương tác sâu với endpoint, phù hợp với môi trường kiểm soát chặt chẽ.
    - Với agentless, bạn giảm được chi phí triển khai, không tác động đến người dùng cuối nhưng vẫn đảm bảo thu thập đầy đủ thông tin nếu đáp ứng đúng yêu cầu hạ tầng (mở port, quyền quản trị…).

- Qua bài lab này, bạn đã hiểu rõ cách cấu hình cả hai mô hình, cách thức hoạt động, yêu cầu kỹ thuật và những lưu ý quan trọng trong quá trình triển khai thực tế.
- Đây là bước thiết lập nền tảng để các chính sách NAC hoạt động chính xác, hiệu quả – từ nhận diện thiết bị, đánh giá rủi ro, cho đến kiểm soát truy cập và phản ứng tự động.

