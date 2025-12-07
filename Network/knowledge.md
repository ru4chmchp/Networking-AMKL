# Network

> Đây là tổng hợp kiến thức về Network, dựa trên mô hình OSI và các giao thức, thuật ngữ liên quan.

---

## Mô hình OSI


| STT | Tên lớp          | Tên tiếng Anh | Chức năng chính                                         |
| --- | ---------------- | ------------- | ------------------------------------------------------- |
| 7   | Ứng dụng         | Application   | Giao tiếp trực tiếp với người dùng                      |
| 6   | Trình bày        | Presentation  | Mã hóa, nén, chuyển đổi định dạng                       |
| 5   | Phiên            | Session       | Quản lý phiên làm việc giữa hai thiết bị                |
| 4   | Giao vận         | Transport     | Phân mảnh dữ liệu, kiểm soát luồng, đảm bảo truyền nhận |
| 3   | Mạng             | Network       | Định tuyến, gán địa chỉ IP                              |
| 2   | Liên kết dữ liệu | Data Link     | Gán địa chỉ MAC, phát hiện lỗi frame                    |
| 1   | Vật lý           | Physical      | Truyền tín hiệu điện/tín hiệu quang                     |

---

### Lớp 1: Vật lý (Physical Layer)

- **Chức năng chính**: Truyền các bit 0 và 1 qua các môi trường vật lý như dây đồng, cáp quang, sóng radio.
- **Thiết bị**: Hub, dây cáp, NIC (Card mạng), sóng Wi-Fi.
- **Ví dụ**: Dữ liệu biến thành tín hiệu điện và đi qua dây RJ45.

---

### Lớp 2: Liên kết dữ liệu (Data Link Layer)

- **Chức năng chính**: Đóng gói bit thành **frame**, địa chỉ hóa bằng **MAC Address**, phát hiện lỗi bằng **CRC**.
- **Thiết bị**: Switch.
- **Giao thức**: Ethernet, PPP, VLAN, ARP.
- **Ví dụ**: Switch gửi frame từ MAC A đến MAC B trong mạng LAN.

---

### Lớp 3: Mạng (Network Layer)

- **Chức năng chính**: Định tuyến và gán **địa chỉ IP** để có thể kết nối nhiều mạng khác nhau.
- **Thiết bị**: Router.
- **Giao thức**: IP (IPv4, IPv6), ICMP, BGP, OSPF.
- **Ví dụ**: Router dùng IP để chuyển gói tin từ mạng A sang mạng B.

---

### Lớp 4: Giao vận (Transport Layer)

- **Chức năng chính**: Phân mảnh gói tin, kiểm soát luồng, đảm bảo tính toàn vẹn (**TCP**) hoặc tốc độ (**UDP**).
- **Giao thức**: TCP, UDP.
- **Ví dụ**: Trình duyệt dùng TCP để tải trang web đầy đủ, đúng thứ tự.

---

### Lớp 5: Phiên (Session Layer)

- **Chức năng chính**: Tạo, duy trì và kết thúc phiên truyền dữ liệu. Xác định điểm checkpoint và xử lý nếu phiên bị gián đoạn.
- **Giao thức**: NetBIOS, RPC, PPTP.
- **Ví dụ**: Đăng nhập từ xa bằng SSH là một phiên làm việc.

---

### Lớp 6: Trình bày (Presentation Layer)

- **Chức năng chính**: Mã hóa (SSL/TLS), chuyển đổi định dạng (ASCII, JPEG, PNG, MPEG), nén dữ liệu (gzip).
- **Ví dụ**: Trình duyệt dùng HTTPS sử dụng TLS/SSL để mã hóa dữ liệu.

---

### Lớp 7: Ứng dụng (Application Layer)

- **Chức năng chính**: Cung cấp giao diện cuối cùng cho người dùng, giao tiếp trực tiếp với phần mềm ứng dụng.
- **Giao thức**: HTTP, DNS, FTP, SMTP, SSH.
- **Ví dụ**: Trình duyệt gọi HTTP GET để tải trang từ Web Server.

---

## Chi tiết các thuật ngữ

### Layer 1: Vật lý

- **PoE (Power over Ethernet)**: Cho phép cùng lúc truyền dữ liệu và cấp nguồn cho thiết bị mạng qua cùng một sợi cáp Ethernet.
- **Serial (Tuần tự)**: Là hình thức truyền phổ biến nhất trên các đường truyền xa (WAN, ...), dữ liệu sẽ truyền từng bit một nối tiếp nhau trên một hoặc vài dây bán dẫn.

---

### Layer 2: Liên kết dữ liệu

- **CRC/FCS (Frame Check Sequence - thường là CRC-32)**: Một trường 4 byte dùng để phát hiện lỗi trong quá trình truyền dữ liệu (bit bị lật, nhiễu). CRC được tính toán tại bên gửi, gửi kèm frame. Bên nhận tính lại CRC; nếu không khớp, frame sẽ bị loại bỏ.
  - **Ví dụ**: Dữ liệu gửi là $1101011011$, polynomial sinh là $10011$. CRC là phần dư khi chia dữ liệu mở rộng cho polynomial đó trong hệ nhị phân (sử dụng phép XOR).
- **MAC Address**: Là địa chỉ phần cứng được gán cho mỗi card mạng (NIC). Layer 2 sử dụng MAC để gửi dữ liệu trong mạng LAN; switch Layer 2 đọc MAC để quyết định chuyển frame đến cổng nào.
- **EtherType**: Là trường 2 byte trong header Ethernet chỉ định giao thức lớp 3 phía trên là gì.
  - **Ví dụ**: `0x0800` là IPv4, `0x0806` là ARP, `0x86DD` là IPv6.
- **Ethernet (IEEE 802.3)**: Là chuẩn truyền dữ liệu phổ biến trong mạng LAN. Nhiệm vụ chính là đóng gói dữ liệu thành Ethernet frame, sử dụng địa chỉ MAC để định danh thiết bị, truyền dữ liệu trong cùng một mạng LAN (không đi qua router).
- **PPP (Point-to-Point Protocol)**: Là giao thức truyền dữ liệu điểm-đến-điểm giữa hai thiết bị mạng, thường dùng cho kết nối dial-up, serial, đường truyền WAN. Chức năng mới là xác thực.
- **PPPoE (Point-to-Point Protocol over Ethernet)**: Cho phép chạy PPP qua mạng Ethernet (vốn là broadcast và multi-access).
- **VLAN (Virtual LAN)**: Có nghĩa là mạng LAN ảo. Là việc chia một mạng vật lý ra thành nhiều mạng logic độc lập. Các thiết bị trong VLAN khác nhau không thể giao tiếp trực tiếp với nhau dù cắm cùng một switch. Tuy nhiên, nếu nhóm các thiết bị không cùng vị trí vật lý nhưng lại thuộc cùng 1 VLAN, các thiết bị đó vẫn có thể giao tiếp với nhau.
  - **Lợi ích**: Tăng tính bảo mật (ngăn truy cập trực tiếp đến các thiết bị chứa dữ liệu công ty), giới hạn broadcast trong mạng LAN lớn, quản lý dễ hơn.
  - **Lưu ý**: Hai VLAN khác nhau cần giao tiếp thì bắt buộc phải định tuyến và đi qua router hoặc switch Layer 3.
  - **Các loại port chính trong VLAN**:
    - **Access Port**: Dùng cho thiết bị đầu cuối kết nối đến switch.
    - **Trunk Port**: Dùng để kết nối giữa các switch với nhau hoặc giữa switch với router.
- **ARP (Address Resolution Protocol)**: Là giao thức Layer 2.5, giúp các thiết bị biết được MAC của IP cụ thể trong cùng LAN.
  - **Chức năng**: Ánh xạ IP thành MAC trong mạng LAN. Bắt buộc khi gói tin IP đi qua LAN (IP không truyền trực tiếp mà phải qua MAC).
  - **Các dạng gói tin ARP**: ARP Request/Reply.

---

### Layer 3: Mạng

- **Routing (Định tuyến)**: Là việc xác định đường đi từ nguồn đến đích của gói tin, có thể qua nhiều mạng khác nhau.
  - **Các loại định tuyến**:
    - **Static Routing**: Tự cấu hình tuyến đường, không tự cập nhật khi thay đổi mạng.
    - **Dynamic Routing**: Tự động cập nhật tuyến đường khi có thay đổi trong mạng.
      - **IGP (Interior Gateway Protocol)**: Dùng để định tuyến trong cùng một hệ thống tự trị (Autonomous System - AS). Ví dụ: RIP, OSPF, EIGRP.
      - **EGP (Exterior Gateway Protocol)**: Dùng để định tuyến giữa các hệ thống tự trị khác nhau. Ví dụ: BGP.

#### IGP (Interior Gateway Protocol)

- **RIP (Routing Information Protocol)**: Là giao thức định tuyến động, thuộc nhóm **distance vector**. Mỗi router chỉ quan tâm đến số hop và router hàng xóm.

  - Router sẽ gửi toàn bộ bảng định tuyến cho tất cả neighbor 30s/lần. Mỗi router có destination network và metric = số hop (1 router là + 1 hop nếu đi qua). Router sẽ chọn đường đi có số hop nhỏ nhất.
  - **Các thông số cần nhớ**:
    - Thời gian update định kỳ: 30s
    - Số hop tối đa: 15 (16 = vô cực --> unreachable)
    - Multicast RIP v2: 224.0.0.9
    - Giao thức: UDP port 520

  | Bước | Diễn giải                                                                                               |
  | :--- | :------------------------------------------------------------------------------------------------------ |
  | 1️⃣   | Mỗi router chỉ **biết trực tiếp neighbor** và các mạng kết nối trực tiếp                                |
  | 2️⃣   | Định kỳ (30s), router sẽ gửi **toàn bộ bảng định tuyến của mình** cho neighbor                          |
  | 3️⃣   | Router nhận thông tin từ neighbor, cộng thêm 1 vào metric (mỗi hop +1)                                  |
  | 4️⃣   | Nếu thấy route mới có metric **tốt hơn (ít hop hơn)**, sẽ **cập nhật bảng định tuyến**                  |
  | 5️⃣   | Quá trình này lặp đi lặp lại, và sau vài vòng, tất cả router trong mạng đều học được route đến mọi mạng |

- **EIGRP (Enhanced Interior Gateway Routing Protocol)**: Là giao thức định tuyến động, thuộc nhóm **Hybrid**. Có tính chất **distance vector** (gửi thông tin cho neighbor) và có tính chất **link state** (lưu topology table, tính toán thông minh).

  - **Công thức tính Metrics**: $metric = [(10^7 / bandwidth) + delay] * 256$
  - **Nguyên tắc hoạt động**: Router sẽ gửi Hello packet định kỳ (5s với LAN và 60s trên WAN) để tạo neighbor discovery. Sau đó, neighbor trao đổi routing update.
  - **DUAL (Diffusing Update Algorithm)**: Tính toán nhanh nhất router đến đích. Khi topology thay đổi, không phải tính toán toàn mạng mà chỉ tính phần thay đổi.
  - **Lưu trữ thông tin**:
    - **Neighbor Table**: Danh sách neighbor.
    - **Topology Table**: Tất cả route học được, kể cả backup route.
    - **Routing Table**: Chỉ chứa best route.

- **OSPF (Open Shortest Path First)**: Là **link state routing protocol**, dựa trên topology mạng chứ không dựa vào neighbor như distance vector.

  | Giai đoạn          | Ý nghĩa                                                                    |
  | :----------------- | :------------------------------------------------------------------------- |
  | Neighbor discovery | Router gửi Hello packet để tìm neighbor (multicast 224.0.0.5)              |
  | Trao đổi database  | Gửi LSA (Link State Advertisement) để xây dựng LSDB                        |
  | Tính toán          | Chạy **SPF (Shortest Path First – thuật toán Dijkstra)** để tính best path |
  | Tạo routing table  | Kết quả tính toán đưa vào bảng định tuyến                                  |

  - **Các thành phần chính trong OSPF**:

  | Thành phần                         | Ý nghĩa                                                                          |
  | :--------------------------------- | :------------------------------------------------------------------------------- |
  | **Router ID (RID)**                | Số nhận dạng router (thường lấy IP loopback cao nhất hoặc IP interface cao nhất) |
  | **LSA (Link State Advertisement)** | Gói tin mô tả trạng thái link, gửi cho toàn mạng                                 |
  | **LSDB (Link State Database)**     | Cơ sở dữ liệu topology toàn mạng                                                 |
  | **SPF algorithm**                  | Thuật toán Dijkstra tính best path                                               |

  - **Các loại router khác trong OSPF**

  | Loại                                     | Vai trò                                              |
  | ---------------------------------------- | ---------------------------------------------------- |
  | Internal router                          | Tất cả interface trong cùng area                     |
  | Backbone router                          | Có interface trong area 0                            |
  | Area Border Router (ABR)                 | Nối area 0 với area khác                             |
  | Autonomous System Boundary Router (ASBR) | Nối OSPF với giao thức định tuyến khác (RIP, BGP...) |

  - Trong OSPF thì người ta không dùng subnet mask mà người ta dùng wildcarrd mask , ngược lại với subnet mask. wildcard mast = 255.255.255.255 - subnet mask.
  - 🌐 Area – vùng OSPF:

    - Mục đích: Giảm tải tính toán và giảm kích thước LSDB.

    - Area 0 (backbone): Phải có, là xương sống.

    - Có thể có các area khác: area 1, area 2, …

    - Area khác muốn giao tiếp phải đi qua area 0.

#### EGP (Exterior Gateway Protocol)

- **BGP (Border Gateway Protocol)** : Là giao thức sử dụng **Path Vector Protocol** khác biệt với Distance Vector.
  - BGP dùng để kết nối giữa các AS(Autonomous Systems) - ví dụ giữa Viettel, VNPT, ...
  - Chọn đường đi tốt nhất giữa các mạng lớn.
  - Có 2 loại BGP là eBGP và iBGP : e là giữa 2 AS khác nhau, i là trong cùng một AS
  - Path Vector Protocol làm việc giữa các AS và tránh loop bằng cách ghi nhớ đường đi chứ không chỉ khoảng cách hay cost, trong path vector mỗi khi router khi quảng bá router sẽ đính kèm thêm thông tin dduognwf đã đi qua thường là chuỗi số AS(AS-Path)

#### Thuật ngữ khác

- **Loopback**: Là một loại interface ảo, không gắn với cổng vật lý nào trên router.
  - Nó luôn ở trạng thái up miễn là router đang bật.
  - Có địa chỉ IP riêng, thường dùng để làm Router ID hoặc để test.
  - Mọi gói tin gửi đến IP loopback sẽ quay về chính router đó, không bao giờ ra cổng vật lý.
- **Classful vs Classless**

  - **Classful**: Khi router chạy thì không gửi kèm subnet mask trong routing update. Router ngầm hiểu dựa trên địa chỉ class (A, B, C), không thể chia nhỏ subnet ngoài class (không hỗ trợ VLSM).

  | Class | Prefix | Subnet mask   | IP range                  |
  | :---- | :----- | :------------ | :------------------------ |
  | A     | /8     | 255.0.0.0     | 1.0.0.0 – 126.0.0.0       |
  | B     | /16    | 255.255.0.0   | 128.0.0.0 – 191.255.0.0   |
  | C     | /24    | 255.255.255.0 | 192.0.0.0 – 223.255.255.0 |

  - **Classless**: Khi router chạy thì gửi kèm subnet mask trong routing update. Hỗ trợ VLSM (Variable Length Subnet Masking) và Supernetting.
    - Vì Classful gây lãng phí địa chỉ IP, người ta dùng Classless để chia subnet nhỏ cho mạng nhỏ, hoặc gộp subnet lớn để giảm kích thước bảng định tuyến.

- **AS** : là một mạng hay một tập hợp mạng do một tổ chức quản lý, chạy cùng chính sách định tuyến và có ASN duy nhất.
  - **ASN (Autonomous System Number)** là một số duy nhất để nhận dạng AS trên internet.
    - Có 2 loại là Public ASN và Private ASN : Public là từ 1-64511 kết nối trực tiếp internet, số con lại 64512-65534 dùng nội bộ, không công bố ra.
  - Tại sao cần AS : Internet co hàng ngàn nhà mạng, data center, cloud provider, để quản lý cần chia các vùng mạng độc lập, BGP sẽ dựa vào AS-Path để biểt route đi qua những AS nào tránh loop.
- **Subnetting** là kỹ thuật chia nhỏ một mạng lớn thành nhiều mạng con hơn (subnet), nó giúp tối ưu số IP, giảm broadcast domain.
  - Ví dụ như chia mạng 192.168.1.0/24 ra thành 7 mạng thì chúng ta sẽ làm như sau :
    - xác đinh số subnet cần chia là 7 thì 2^n =7 nếu không được ta lấy số lớn hơn là 8 thì 2^3 = 8 >7
    - có nghĩa là chúng ta mượn 3 bits từ hosts thì mạng sẽ thành 192.168.1.0/27
    - chúng ta tính địa chỉ như sau 2^(32- số / (ở đây là 27)) = 2^(32-27=5) = 32 địa chỉ.
    - Nhưng trừ net và broadcast ra thì còn 32-2 = 30 hosts khả dụng. từ đó tính tiếp.
- **IPv4 và IPv6** :
  - IPv4: 32 bit, broadcast, NAT, header phức tạp, thiếu địa chỉ
  - IPv6: 128 bit, multicast & anycast, không NAT, header đơn giản, bảo mật tích hợp
- **NAT (Network Address Translation)** là kỹ thuật chuyển đổi IP của gói tin khi đi qua router hoặc firewall. Giúp một hay nhiều thiết bị nội bộ dùng chung 1 địa chỉ IP public để kết nối ra Internet.
