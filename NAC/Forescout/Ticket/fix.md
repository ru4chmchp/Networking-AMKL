# Những lỗi gặp phải khi triển khai Forescout

Trong quá trình triển khai thực tế hệ thống Forescout NAC, dù theo mô hình đơn giản hay tích hợp sâu với hạ tầng mạng(ở đây mình triển khai trong PNETLAB nên chủ yếu fix các vấn đề kết nối trong đó), việc phát sinh lỗi là điều khó tránh khỏi. Các lỗi này có thể đến từ sai sót cấu hình, thiếu tương thích thiết bị mạng, giới hạn bản quyền, hoặc chưa hiểu rõ cơ chế hoạt động của Forescout.

Bài viết này sẽ tổng hợp các lỗi phổ biến mình đã gặp trong quá trình triển khai Forescout trên nên tảng PNETLAB, đi kèm nguyên nhân, cách khắc phục, và bài học kinh nghiệm để giúp bạn rút ngắn thời gian xử lý sự cố và triển khai hệ thống ổn định, hiệu quả hơn.

## Mục lục:
​
1. [Cấu hình AD, LDAP và Radius]()
2. [Cấu hình Switch]()
3. [Cấu hình SPAN PORT thiết bị Cisco PNETLAB]()
4. [Cấu hình Radius setting]()
5. [Cấu hình Switch ở mode HA Active Passive Forescout]()




## 1. Cấu hình AD, LDAP và Radius

Khi bạn cấu hình để Join vào AD sẽ có lỗi như sau
​<div align="center">
  <img src="/images/image435.png" alt="Network Topology" width="100%"/>
</div>

Thì đây là lỗi đặt sai tên Server Name của User Director, đúng đó phải là hostname của Active Directory

​<div align="center">
  <img src="/images/image436.png" alt="Network Topology" width="100%"/>
</div>

Okey vậy là done

2. [Cấu hình Switch]()

Khi bạn đã cấu hình xong switch và tiến hành test thử trong Forescout sẽ có lỗi sau (lưu ý ở đây trong PNETLAB và thiết bị là Cisco): 

​<div align="center">
  <img src="/images/image440.png" alt="Network Topology" width="100%"/>
</div>

thì bạn sẽ thấy ở trên 3 lỗi để không thể find MAC addresses, lỗi này vì giới hạn hỗ trợ giao thức SNMP của image Cisco VIOS, ta chỉ cần chỉnh MAC Read/Write Method từ Automatic sang SNMP RO (Read only) và CLI có nghĩa là ở đây đọc bằng giao thức SNMP và ghi bằng CLI vì SNMP nó giới hạn hỗ trợ không thể tạo ROW mới trong VLAN Table nên phải chuyển sang CLI để ghi thay vì RW của SNMP

​<div align="center">
  <img src="/images/image445.png" alt="Network Topology" width="100%"/>
</div>

Kết quả sau khi đổi 

​<div align="center">
  <img src="/images/image447.png" alt="Network Topology" width="100%"/>
</div>

3. [cấu hình SPAN PORT thiết bị Cisco PNETLAB]()

Ở đây mình sẽ có trường hợp như này, khi cấu hinh monitor trên Cisco xong và check ra như này, bạn thấy ổn.
​<div align="center">
  <img src="/images/image448.png" alt="Network Topology" width="100%"/>
</div>

Nhưng khi kiểm tra bên Forescout

​<div align="center">
  <img src="/images/image449.png" alt="Network Topology" width="100%"/>
</div>

Thì bạn sẽ thấy lưu lượng untagged ?????

Ở đây bạn chỉ cần cấu hình thêm 2 tùy chọn nữa là 

​<div align="center">
  <img src="/images/image441.png" alt="Network Topology" width="100%"/>
</div>

để tùy chọn Native, thì switch sẽ lột bỏ cac tag, còn replicate lệnh này yêu cầu Switch giữ nguyên trạng thái đóng gói có gắn tag.

​<div align="center">
  <img src="/images/image450.png" alt="Network Topology" width="100%"/>
</div>

Sau đó check, ta đã có traffic

​<div align="center">
  <img src="/images/image451.png" alt="Network Topology" width="100%"/>
</div>

Done

## 4. Cấu hình Radius setting [Link](https://forescout.my.site.com/support/s/article/Radius-subprocess-radiusd-exited)

nếu bạn kiểm tra dot1x status mà ra kết quả như sau 
​<div align="center">
  <img src="/images/image491.png" alt="Network Topology" width="100%"/>
</div>

Sau đó ta tiến hành `fstool radiusd start`, nếu không có lỗi thì oke, nhưng nếu hiện lỗi như hình bên dưới thì ta biết nguyên do 

​<div align="center">
  <img src="/images/image493.png" alt="Network Topology" width="100%"/>
</div>

> Error: rlm_ldap (ldap__LAB): Bind with (anonymous) to 
ldap://WIN-8V4S3MFT74T.lab.intern:3269 failed: Can't contact LDAP server

radiusd không kết nối được đến AD (Winserver) qua LDAP port 3269 → crash ngay!

Ta tiến hành vào Tool > Options > Radius > Radius setting, ta đổi từ catalog over tls sang catalog bình thường.

​<div align="center">
  <img src="/images/image494.png" alt="Network Topology" width="100%"/>
</div>

Sau đó `fstool radiusd restart`, đợi một chút

​<div align="center">
  <img src="/images/image495.png" alt="Network Topology" width="100%"/>
</div>

​<div align="center">
  <img src="/images/image496.png" alt="Network Topology" width="100%"/>
</div>

oke done


## 4. Cấu hình Switch ở mode HA Active Passive Forescout

Nếu ta connect với switch ở mode HA thì có lỗi như này đối với các dòng switch cisco cũ

​<div align="center">
  <img src="/images/image512.png" alt="Network Topology" width="100%"/>
</div>

​<div align="center">
  <img src="/images/image513.png" alt="Network Topology" width="100%"/>
</div>

​<div align="center">
  <img src="/images/image514.png" alt="Network Topology" width="100%"/>
</div>

Thi đây là các lỗi về cipher và algorithm của ssh, do phiên bản ssh của cisco quá cũ nên phải ép Forescout chấp nhận với các parameter sau [link](https://forescout.my.site.com/support/s/article/Resolving-SSH-Connection-Failure-with-Cisco-Switch-After-CounterACT-9-1-x-Upgrade)


```bash
-o Ciphers=+aes128-cbc -o KexAlgorithms=+diffie-hellman-group14-sha1,diffie-hellman-group1-sha1 -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa -p 22
```

Option -p 22 có thể thêm vào nếu như forescout LISTEN PORT 2222

nếu k được thì chúng ta còn một cách nữa đó là update crypto policy

​<div align="center">
  <img src="/images/image516.png" alt="Network Topology" width="100%"/>
</div>

Ở trong hình mình đã update, nếu cấu hình của bạn là DEFAULT thì nên update thành LEGACY

```bash
update-crypto-policies --set LEGACY
```

sau đó thử lại là được.