# Cấu Hình Public Certificate và DNS cho ForeScout​

Trong các hệ thống NAC (Network Access Control), việc triển khai chứng chỉ số (Public Certificate) và cấu hình DNS đóng vai trò then chốt nhằm đảm bảo tính bảo mật, xác thực và khả năng truy cập tin cậy. Đặc biệt với Forescout, việc cấu hình đúng SSL certificate và phân giải DNS chuẩn xác là điều kiện bắt buộc để triển khai các tính năng như Captive Portal, SecureConnector, hoặc tích hợp các giải pháp bên ngoài thông qua giao tiếp HTTPS. Lab này hướng dẫn chi tiết cách thực hiện cấu hình Public Certificate và DNS phù hợp với môi trường doanh nghiệp thực tế.


## Mục Lục:

1. Cấu hình DNS cho ForeScout​
2. Cấu hình Public Certificate cho ForeScout​
3. Cấu hình HTTP Redirection cho ForeScout​
4. Kết luận​

## 1. Cấu hình DNS cho ForeScout​
Cấu hình DNS cho ForeScout đóng vai trò quan trọng trong việc đảm bảo hệ thống có thể phân giải tên miền để truy cập thiết bị dễ dàng, đồng thời hỗ trợ tương tác hiệu quả với các endpoint thông qua DNS.
- Truy cập Tools -> Options -> General -> Mail and DNS.
- Tại mục DNS to Reference, điền thông tin DNS Domain và DNS Server Address(es) phù hợp với hệ thống.
- Sau khi hoàn tất, nhấn Apply để áp dụng thay đổi.


<div align="center">
  <img src="/images/image528.png" alt="Network Topology" width="100%"/>
</div>

Kiểm tra cấu hình DNS trên ForeScout

- Truy cập Options -> Modules -> DNS Client.
- Tại mục Test Address, nhập địa chỉ IP của thiết bị cần kiểm tra.
- Nhấn Test để xác minh xem thiết bị đã được cấu hình DNS đúng trên máy chủ DNS hay chưa.

<div align="center">
  <img src="/images/image529.png" alt="Network Topology" width="100%"/>
</div>

Kiểm tra DNS Query trên DNS Server trong ForeScout

- Truy cập Options -> Modules -> DNS Query Extension.
- Tại mục DN Search, nhập tên miền (domain) bạn muốn truy vấn.
- Nhấn Test để thực hiện kiểm tra.

<div align="center">
  <img src="/images/image530.png" alt="Network Topology" width="100%"/>
</div>

Xác minh phân giải hostname sau cấu hình DNS

- Sau khi hoàn tất cấu hình DNS và thực hiện các bước kiểm tra, truy cập Options -> Appliance.
- Tại đây, bạn sẽ thấy ForeScout Appliance đã được phân giải tên miền (hostname) đúng như thông tin DNS đã cấu hình trên DNS Server.

<div align="center">
  <img src="/images/image531.png" alt="Network Topology" width="100%"/>
</div>

Khởi động lại các plugin dịch vụ qua SSH trên ForeScout Appliance

- Đăng nhập vào thiết bị ForeScout thông qua giao thức SSH với tài khoản của cliadmin.
- Sau khi đăng nhập thành công, thực hiện các lệnh sau để khởi động lại các plugin cần thiết:

```bash
fstool webapi restart
fstool webreports restart
```

Sau khi hoàn tất cấu hình DNS và khởi động lại các plugin cần thiết, bạn có thể tiến hành kiểm tra khả năng kết nối tới thiết bị ForeScout bằng tên miền đã cấu hình:

<div align="center">
  <img src="/images/image532.png" alt="Network Topology" width="100%"/>
</div>

## Cấu hình Public Certificate cho ForeScout

Mặc dù cấu hình DNS giúp truy cập thiết bị ForeScout thông qua tên miền, nhưng vẫn có thể gặp cảnh báo “Your connection is not private” nếu hệ thống vẫn sử dụng chứng chỉ riêng (private certificate).

<div align="center">
  <img src="/images/image533.png" alt="Network Topology" width="100%"/>
</div>

Để giải quyết vấn đề này, cần chuyển sang sử dụng chứng chỉ công khai (Public Certificate) bằng cách:

- Truy cập Options -> Certificates -> Trusted Certificates.
- Chọn Add để thêm các chứng chỉ tin cậy đã chuẩn bị sẵn.
- Đảm bảo rằng Public Certificate được import là hợp lệ và được cấp bởi tổ chức CA uy tín.

<div align="center">
  <img src="/images/image534.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo tiến hành chọn certificate cần tin tường vào và click chọn Enable trusting this certificate rồi nhấn Next.

<div align="center">
  <img src="/images/image535.png" alt="Network Topology" width="100%"/>
</div>

Tại phần Issuer Chain Ta sẽ thấy thông tin certificate vừa được import vào tiến hành chọn Next để tiếp tục.

<div align="center">
  <img src="/images/image536.png" alt="Network Topology" width="100%"/>
</div>

Tại phần Trusted for Subsystems tiến hành chọn All subsystems để tin tưởng chứng chỉ này cho tất cả các plugins đang chạy trên ForeScout rồi nhấn Finish.

<div align="center">
  <img src="/images/image537.png" alt="Network Topology" width="100%"/>
</div>

Sau khi thêm xong chúng ta sẽ thấy được các thông tin của chứng chỉ

<div align="center">
  <img src="/images/image538.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo Để import public để ForeScout sử dụng, tiến hành vào phần Options -> Certificates -> System Certificates chọn Add from PKCS#12.

<div align="center">
  <img src="/images/image539.png" alt="Network Topology" width="100%"/>
</div>

Tùy theo định dạng của public certificate mà các thông tin cần điền sẽ khác nhau. Trong trường hợp sử dụng định dạng .pfx, bắt buộc phải nhập mật khẩu tương ứng để giải mã certificate.

<div align="center">
  <img src="/images/image540.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo, tại mục Certificate Details, bạn cần kiểm tra kỹ các thông tin của chứng chỉ vừa được thêm vào như Tên, Ngày hết hạn, Tổ chức phát hành (Issuer), v.v. Sau khi xác nhận đầy đủ và chính xác, nhấn Next để tiếp tục.

<div align="center">
  <img src="/images/image541.png" alt="Network Topology" width="100%"/>
</div>

Tiếp theo, tại phần Issuer Chain, tiến hành import các chứng chỉ gốc (Root CA) và trung gian (Intermediate CA) tương ứng để đảm bảo chứng chỉ chính được xác thực đầy đủ. Sau khi hoàn tất, nhấn Next để chuyển sang bước tiếp theo.

<div align="center">
  <img src="/images/image542.png" alt="Network Topology" width="100%"/>
</div>

Tại phần Scope, chọn các plugin hoặc chức năng mà certificate này sẽ được sử dụng, ví dụ như Captive Portal hoặc Web Console. Sau khi lựa chọn phù hợp, nhấn Next để tiếp tục.

<div align="center">
  <img src="/images/image543.png" alt="Network Topology" width="100%"/>
</div>

Tại phần Description, nhập mô tả ngắn gọn giúp nhận diện mục đích hoặc phạm vi sử dụng của chứng chỉ. Sau đó, tick chọn Enable presenting this certificate để kích hoạt chứng chỉ, rồi nhấn Finish để hoàn tất quá trình cấu hình.

<div align="center">
  <img src="/images/image544.png" alt="Network Topology" width="100%"/>
</div>

Sau khi đã import thành công public certificate, tiến hành tắt tùy chọn "Enable presenting this certificate" trên các chứng chỉ private/self-signed hiện có. Việc này nhằm đảm bảo ForeScout sẽ sử dụng chứng chỉ mới được import làm mặc định để trình bày trong các kết nối.

<div align="center">
  <img src="/images/image545.png" alt="Network Topology" width="100%"/>
</div>

Sau khi áp dụng chứng chỉ cho các plugin, quá trình cập nhật có thể mất khoảng 5–10 phút. Sau khi hoàn tất, truy cập lại GUI của ForeScout, bạn sẽ thấy không còn xuất hiện cảnh báo “Your connection is not private”. Điều này xác nhận rằng hệ thống đã bắt đầu sử dụng chứng chỉ hợp lệ vừa được import.

## 3. Cấu hình HTTP Redirection cho ForeScout

Sau khi đã cấu hình xong 2 bước ở trên, lúc này các thông báo HTTP cũng Authenticate HTTP vẫn sử dụng IP thay vì DNS. Tiến hành vào phần Options -> NAC -> HTTP Redirection tiến hành chọn Attempt redirect using DNS name rồi nhấn Apply lúc này hệ thống sẽ chuyển sang DNS cho việc thông báo HTTP cũng như Authenticate HTTP.

<div align="center">
  <img src="/images/image546.png" alt="Network Topology" width="100%"/>
</div>

Kiểm tra: Tiến hành gửi 1 thông báo HTTP Notification tới Endpoint

<div align="center">
  <img src="/images/image547.png" alt="Network Topology" width="100%"/>
</div>

## 4. Kết luận​

Việc cấu hình đúng Public Certificate và DNS là bước thiết yếu để đảm bảo các dịch vụ của Forescout hoạt động ổn định, bảo mật và có thể tích hợp hiệu quả với các hệ thống bên ngoài. Thực hiện đúng lab này giúp xác thực giao tiếp HTTPS, tránh lỗi trình duyệt, đồng thời tạo nền tảng cho các tính năng như Captive Portal, SecureConnector và API hoạt động trơn tru trong môi trường doanh nghiệp.