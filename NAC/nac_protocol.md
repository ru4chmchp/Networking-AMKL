# NAC Protocols 

## Mục lục

1. [NAC Protocol Stack — Toàn cảnh](#1-nac-protocol-stack--toàn-cảnh)
2. [IEEE 802.1X — Port-Based Network Access Control](#2-ieee-8021x--port-based-network-access-control)
3. [EAP — Extensible Authentication Protocol](#3-eap--extensible-authentication-protocol)
4. [RADIUS — AAA Transport](#4-radius--aaa-transport)
5. [CoA — Change of Authorization](#5-coa--change-of-authorization)
6. [TACACS+](#6-tacacs)
7. [DHCP — Role trong NAC](#7-dhcp--role-trong-nac)
8. [SNMP — Discovery & Enforcement](#8-snmp--discovery--enforcement)
9. [NetFlow / IPFIX & Syslog](#9-netflow--ipfix--syslog)

---

# PHẦN I — GIAO THỨC

---

## 1. NAC Protocol Stack — Toàn cảnh

NAC không phải một giao thức duy nhất mà là **một tập hợp giao thức phân tầng**, mỗi tầng đảm nhiệm một chức năng độc lập. Hiểu rõ từng tầng là nền tảng để debug và thiết kế giải pháp NAC.

```
╔══════════════════════════════════════════════════════════════════════════╗
║                         NAC PROTOCOL STACK                              ║
╠═══════════════════════╦══════════════════════════╦═══════════════════════╣
║  Tầng chức năng       ║  Giao thức / Công nghệ   ║  Port / Layer         ║
╠═══════════════════════╬══════════════════════════╬═══════════════════════╣
║  Discovery            ║  DHCP passive snooping   ║  UDP 67/68            ║
║  Ai đang có mặt?      ║  ARP monitoring          ║  L2 broadcast         ║
║                       ║  SNMP v3 polling         ║  UDP 161              ║
║                       ║  NMAP active scan        ║  ICMP / TCP           ║
║                       ║  CDP / LLDP              ║  L2 multicast         ║
║                       ║  SPAN / RSPAN (passive)  ║  Port mirror          ║
╠═══════════════════════╬══════════════════════════╬═══════════════════════╣
║  Authentication       ║  IEEE 802.1X             ║  EAPOL (L2)           ║
║  Đây là ai?           ║  EAP-TLS / PEAP / FAST   ║  Inside 802.1X        ║
║                       ║  MAB (MAC Auth Bypass)   ║  RADIUS               ║
║                       ║  Web Auth / Captive      ║  HTTP/HTTPS           ║
╠═══════════════════════╬══════════════════════════╬═══════════════════════╣
║  AAA Transport        ║  RADIUS Authentication   ║  UDP 1812             ║
║  Kênh xác thực        ║  RADIUS Accounting       ║  UDP 1813             ║
║                       ║  CoA / Disconnect        ║  UDP 3799 (RFC 5176)  ║
║                       ║  TACACS+ (device admin)  ║  TCP 49               ║
╠═══════════════════════╬══════════════════════════╬═══════════════════════╣
║  Posture              ║  WMI (Windows query)     ║  TCP 135 / 445        ║
║  Thiết bị có an toàn? ║  SSH (Linux/Unix query)  ║  TCP 22               ║
║                       ║  Agent-based (FS SC)     ║  TCP 10003            ║
║                       ║  SMB / NetBIOS           ║  TCP 445              ║
╠═══════════════════════╬══════════════════════════╬═══════════════════════╣
║  Enforcement          ║  RADIUS CoA              ║  UDP 3799             ║
║  Áp chính sách        ║  SNMP Write              ║  UDP 161              ║
║                       ║  ACL / VLAN / SGT        ║  Via RADIUS AVP       ║
║                       ║  Firewall API            ║  REST / HTTPS         ║
╠═══════════════════════╬══════════════════════════╬═══════════════════════╣
║  Monitoring           ║  NetFlow / IPFIX         ║  UDP 2055 / 4739      ║
║  Hành vi sau truy cập ║  Syslog                  ║  UDP 514 / TCP 6514   ║
║                       ║  SNMP Trap               ║  UDP 162              ║
╠═══════════════════════╬══════════════════════════╬═══════════════════════╣
║  Integration          ║  REST API                ║  TCP 443              ║
║  Hệ sinh thái         ║  pxGrid (Cisco)          ║  TCP 8910             ║
║                       ║  Syslog / CEF            ║  UDP 514              ║
╚═══════════════════════╩══════════════════════════╩═══════════════════════╝
```

**Luồng tổng thể của một NAC session:**

```
[Endpoint kết nối vào switch]
         │
         ▼
  [DISCOVERY] ──── DHCP snooping + ARP + SPAN traffic
         │         → Forescout/NAC biết: IP, MAC, hostname, device type
         │
         ▼
  [AUTHENTICATION] ── 802.1X / MAB / Web Auth → RADIUS
         │            → Xác minh danh tính người dùng / thiết bị
         │
         ▼
  [POSTURE CHECK] ─── WMI / SSH / Agent
         │            → Kiểm tra AV, patch, disk encryption...
         │
         ▼
  [POLICY DECISION] ── NAC engine đối chiếu với policy
         │             → Kết quả: Allow / Quarantine / Remediate / Deny
         │
         ▼
  [ENFORCEMENT] ─── CoA / SNMP Write / ACL / VLAN change
         │          → Switch / WLC áp dụng thay đổi tức thì
         │
         ▼
  [MONITORING] ─── NetFlow + Syslog + SNMP Trap
                   → Theo dõi hành vi ongoing, phát hiện anomaly
```

---

## 2. IEEE 802.1X — Port-Based Network Access Control

### 2.1 Tổng quan

**IEEE 802.1X** (chuẩn gần nhất: IEEE 802.1X-2020) là cơ chế kiểm soát truy cập **tầng 2 (Layer 2)** hoạt động trên cơ sở **Port-Based Network Access Control**. Một switch port ở trạng thái *unauthorized* cho đến khi quá trình xác thực hoàn tất thành công — đây là cơ chế ngăn chặn tuyệt đối ở điểm vào của mạng.

### 2.2 Ba thành phần trong mô hình 802.1X

```
┌────────────────────┐          ┌─────────────────────┐          ┌─────────────────────┐
│     SUPPLICANT     │          │    AUTHENTICATOR    │          │  AUTHENTICATION     │
│                    │          │                     │          │      SERVER         │
│  Endpoint device   │◄─EAPOL──►│  Network device     │◄─RADIUS─►│                    │
│  - Windows PC      │          │  - Cisco Switch     │          │  - Cisco ISE        │
│  - macOS           │          │  - Aruba WLC        │          │  - Aruba ClearPass  │
│  - Linux           │          │  - Fortinet FW      │          │  - Forescout        │
│  - wpa_supplicant  │          │  - VPN Gateway      │          │  - FreeRADIUS       │
└────────────────────┘          └─────────────────────┘          └─────────────────────┘
        │                                  │                                │
  EAP over LAN                    Forward EAP payload               Validate identity
  (EAPOL frames)                  via RADIUS tunnel                 Apply policy
  Ethertype 0x888E                UDP 1812 / 1813                   Return attributes
```

| Thành phần | Chức năng kỹ thuật | Giao thức sử dụng |
|---|---|---|
| **Supplicant** | Khởi tạo và phản hồi EAP exchange; giữ session state | EAPOL (Ethertype `0x888E`) |
| **Authenticator** | Chặn tất cả L2 traffic (trừ EAPOL); relay EAP giữa supplicant và server | EAPOL inbound, RADIUS outbound |
| **Authentication Server** | Xác minh credential; áp dụng policy; trả về RADIUS attributes (VLAN, ACL, SGT) | RADIUS (UDP 1812/1813) |

### 2.3 Luồng xác thực 802.1X đầy đủ

```
Supplicant               Authenticator (Switch)              Auth Server (RADIUS)
    │                            │                                    │
    │──── EAPOL-Start ──────────►│                                    │
    │                            │                                    │
    │◄─── EAP-Request/Identity ──│                                    │
    │──── EAP-Response/Identity ►│                                    │
    │                            │──── RADIUS Access-Request ────────►│
    │                            │     [EAP-Response/Identity inside] │
    │                            │                                    │ [Lookup user]
    │                            │◄─── RADIUS Access-Challenge ───────│
    │                            │     [EAP-Request/Method]           │
    │◄─── EAP-Request/Method ────│                                    │
    │                            │                                    │
    │   ╔══ EAP Method exchange (TLS handshake / PEAP tunnel) ══╗     │
    │   ║  (có thể nhiều round-trip tùy EAP method)             ║     │
    │   ╚════════════════════════════════════════════════════════╝     │
    │                            │                                    │
    │──── EAP-Response/Method ──►│                                    │
    │                            │──── RADIUS Access-Request ────────►│
    │                            │                                    │ [Verify creds]
    │                            │◄─── RADIUS Access-Accept ──────────│
    │                            │     Tunnel-Type = VLAN (13)        │
    │                            │     Tunnel-Medium-Type = 802 (6)   │
    │                            │     Tunnel-Private-Group-Id = 100  │
    │                            │     Cisco-AVPair = "ip:inacl#1=..." │
    │◄─── EAP-Success ───────────│                                    │
    │                            │                                    │
    │     ════ ACCESS GRANTED ════│ [Port moves to AUTHORIZED state]  │
    │                            │──── RADIUS Accounting-Start ──────►│
```

### 2.4 Port States và Fallback Mechanisms

| State | Điều kiện | Traffic được phép | Hành vi |
|---|---|---|---|
| **Unauthorized** | Mặc định khi device kết nối | EAPOL only (`0x888E`) | Chặn toàn bộ data |
| **Authorized** | Xác thực thành công | Full access theo VLAN/ACL | Policy từ RADIUS được áp dụng |
| **Guest VLAN** | 802.1X không có phản hồi (timeout) | Giới hạn theo Guest VLAN policy | Cho non-dot1x devices vào mạng hạn chế |
| **Auth-Fail VLAN** | Xác thực thất bại (sai password) | Giới hạn theo Fail VLAN policy | Tránh hard-block, cho user remediate |
| **Critical VLAN** | RADIUS server unreachable | Theo Critical VLAN policy | Duy trì hoạt động khi server down |
| **Restricted VLAN** | Posture fail | Quarantine / Remediation network | Redirect đến remediation portal |

### 2.5 MAC Authentication Bypass (MAB)

MAB là fallback khi thiết bị **không hỗ trợ 802.1X supplicant** (IoT, IP phone, printer, camera).

```
Switch (Authenticator)                    RADIUS Server
      │                                         │
      │  [Device kết nối, KHÔNG gửi EAPOL]      │
      │  [Switch chờ dot1x timeout ~30s]         │
      │                                         │
      │──── RADIUS Access-Request ─────────────►│
      │     User-Name = "00-1A-2B-3C-4D-5E"     │  ← MAC address thay username
      │     User-Password = "001a2b3c4d5e"       │  ← MAC address thay password
      │     Service-Type = Call-Check (10)       │
      │                                         │
      │     [RADIUS tra cứu MAC trong DB]        │
      │     [hoặc gửi sang profiling engine]     │
      │                                         │
      │◄─── RADIUS Access-Accept ───────────────│
      │     Tunnel-Private-Group-Id = 200 (IoT VLAN)
```

> ⚠️ **Điểm yếu bảo mật của MAB**: Attacker có thể **spoof MAC address** của một thiết bị đã được whitelist. Biện pháp giảm thiểu: kết hợp MAB với **device profiling** (DHCP fingerprint, behavior analysis) để validate thêm.

---

## 3. EAP — Extensible Authentication Protocol

### 3.1 Vị trí trong stack

**EAP (RFC 3748)** là *carrier framework* — nó không tự xác thực mà cung cấp **cơ chế vận chuyển** để các phương thức xác thực cụ thể (TLS, MSCHAPv2...) chạy bên trong. EAP chạy bên trong EAPOL (L2) ở phía supplicant-switch và bên trong RADIUS payload ở phía switch-server.

```
┌───────────────────────────────────────────────┐
│           Application / Auth Method            │
│  (EAP-TLS / PEAP / EAP-FAST / EAP-TTLS)       │
├───────────────────────────────────────────────┤
│                  EAP Framework                 │  ← RFC 3748
├─────────────────────────┬─────────────────────┤
│    EAPOL (802.1X)       │  RADIUS (AVP 79)    │
│  (Supplicant ↔ Switch)  │  (Switch ↔ Server)  │
└─────────────────────────┴─────────────────────┘
```

### 3.2 So sánh các EAP Methods

| EAP Method | Xác thực Client | Xác thực Server | Tunnel | Bảo mật | PKI yêu cầu | Use Case |
|---|---|---|---|---|---|---|
| **EAP-TLS** | X.509 Certificate | X.509 Certificate | TLS mutual | ★★★★★ | Cả 2 phía | Managed corp devices |
| **PEAP/MSCHAPv2** | Username + Password | Server Certificate | TLS (server only) | ★★★☆☆ | Server only | Domain-joined workstations |
| **EAP-FAST** | PAC token / Cert | PAC token / Cert | TLS | ★★★★☆ | Optional | Cisco environments |
| **EAP-TTLS/PAP** | Username + Password | Server Certificate | TLS | ★★★☆☆ | Server only | Mixed OS environments |
| **EAP-MD5** | MD5(password) | Không | Không | ★☆☆☆☆ | Không | Legacy — **không dùng** |

### 3.3 EAP-TLS Handshake (chuẩn production)

```
Supplicant                                          RADIUS Server
    │                                                     │
    │──── EAP-Response/Identity ─────────────────────────►│
    │                                                     │
    │◄─── EAP-Request/TLS [Start] ────────────────────────│
    │                                                     │
    │──── TLS ClientHello ───────────────────────────────►│
    │     (TLS version, cipher suites, random nonce)      │
    │                                                     │
    │◄─── TLS ServerHello ────────────────────────────────│
    │     (Chosen cipher suite, Server Certificate)       │
    │     [Supplicant validates Server Cert vs trusted CA] │
    │                                                     │
    │──── TLS Client Certificate ────────────────────────►│
    │     CertificateVerify, ChangeCipherSpec, Finished   │
    │     [Server validates Client Cert vs trusted CA]    │
    │     [Server checks cert CN/SAN, validity, CRL/OCSP] │
    │                                                     │
    │◄─── TLS ChangeCipherSpec + Finished ────────────────│
    │                                                     │
    │◄─── EAP-Success ────────────────────────────────────│
```

**Hạ tầng bắt buộc cho EAP-TLS:**

```
PKI Infrastructure
├── Root CA (internal hoặc Microsoft AD CS)
│   ├── Intermediate CA (khuyến nghị)
│   │   ├── RADIUS Server Certificate
│   │   │   ├── CN: radius.corp.local
│   │   │   └── Extended Key Usage: Server Authentication (1.3.6.1.5.5.7.3.1)
│   │   └── Endpoint Certificates (cấp qua GPO / SCEP / NDES / MDM)
│   │       ├── CN: hostname hoặc UPN
│   │       └── Extended Key Usage: Client Authentication (1.3.6.1.5.5.7.3.2)
└── CRL Distribution Point / OCSP Responder
    └── Phải accessible từ RADIUS server để revocation check
```

---

## 4. RADIUS — AAA Transport

### 4.1 Kiến trúc và cơ chế

**RADIUS (RFC 2865/2866)** là giao thức AAA chuẩn de facto trong NAC. Hoạt động theo mô hình **client-server**: NAS (Network Access Server — switch/WLC) là RADIUS client, NAC engine là RADIUS server.

```
Bảo mật RADIUS packet:
┌──────────────────────────────────────────────────────────────┐
│  RADIUS Packet Structure                                     │
├──────────┬─────────┬──────────────┬─────────────────────────┤
│  Code    │  Ident  │  Length      │  Authenticator (16 bytes)│
│  1 byte  │  1 byte │  2 bytes     │  (MD5 hash)              │
├──────────┴─────────┴──────────────┴─────────────────────────┤
│  Attributes (TLV format: Type, Length, Value)               │
│  - User-Password: XOR với MD5(shared-secret + authenticator)│
│  - Tất cả fields khác: PLAINTEXT (không mã hóa)             │
│  ⚠ Dùng RADSEC (RADIUS over TLS) để mã hóa toàn bộ         │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 RADIUS Packet Types

| Code | Tên | Hướng | Mô tả |
|---|---|---|---|
| `1` | **Access-Request** | NAS → Server | Gửi credential và thông tin ngữ cảnh để xác thực |
| `2` | **Access-Accept** | Server → NAS | Xác thực thành công; kèm authorization attributes |
| `3` | **Access-Reject** | Server → NAS | Xác thực thất bại |
| `11` | **Access-Challenge** | Server → NAS | Yêu cầu thêm thông tin (EAP multi-round) |
| `4` | **Accounting-Request** | NAS → Server | Start / Interim-Update / Stop session |
| `5` | **Accounting-Response** | Server → NAS | ACK accounting record |
| `40` | **Disconnect-Request** | Server → NAS | Ngắt kết nối session (RFC 5176 CoA) |
| `41` | **Disconnect-ACK/NAK** | NAS → Server | Phản hồi Disconnect-Request |
| `43` | **CoA-Request** | Server → NAS | Thay đổi authorization của session đang active |
| `44` | **CoA-ACK/NAK** | NAS → Server | Phản hồi CoA-Request |

### 4.3 RADIUS Attributes quan trọng trong NAC

**Standard Attributes (RFC 2865):**

| Attribute | ID | Type | Mô tả và ví dụ |
|---|---|---|---|
| `User-Name` | 1 | String | Username hoặc MAC (MAB): `jsmith@corp.com` |
| `User-Password` | 2 | String | Encrypted password |
| `Framed-IP-Address` | 8 | IP Addr | IP của client: `10.10.1.100` |
| `Filter-Id` | 11 | String | Tên ACL áp dụng: `CORP-ACL` |
| `Service-Type` | 6 | Integer | `Framed(2)` = network; `Call-Check(10)` = MAB |
| `Session-Timeout` | 27 | Integer | Giây: `28800` = 8 tiếng |
| `Called-Station-Id` | 30 | String | Wireless: `AA-BB-CC-DD-EE-FF:SSID-Name` |
| `Calling-Station-Id` | 31 | String | MAC client: `00-11-22-33-44-55` |
| `NAS-Port-Id` | 87 | String | `GigabitEthernet1/0/5` |
| `NAS-IP-Address` | 4 | IP Addr | IP của switch/NAS |

**VLAN Assignment Attributes (RFC 3580):**

```
# Ví dụ RADIUS Access-Accept response với VLAN assignment:
Tunnel-Type         = VLAN (13)
Tunnel-Medium-Type  = 802 (6)
Tunnel-Private-Group-Id = "100"    ← VLAN ID dạng string
```

**Cisco VSA (Vendor-Specific Attribute, ID 26):**

```
# dACL (downloadable ACL):
Cisco-AVPair = "ip:inacl#1=permit ip 10.10.1.0 0.0.0.255 any"
Cisco-AVPair = "ip:inacl#2=deny ip any any"

# URL Redirect (captive portal):
Cisco-AVPair = "url-redirect=https://remediation.corp.com/portal"
Cisco-AVPair = "url-redirect-acl=REDIRECT-ACL"

# Security Group Tag (TrustSec):
Cisco-AVPair = "cts:security-group-tag=0010-0"
```

### 4.4 RADIUS Shared Secret và bảo mật

```
⚠ RADIUS Authentication không mã hóa toàn bộ packet
   → Dùng IPSec hoặc RADSEC (RFC 6614) trong môi trường production

Shared Secret requirements:
  - Tối thiểu 22 ký tự random
  - Khác nhau cho từng NAS client
  - Không dùng dictionary words

Ví dụ cấu hình Cisco IOS:
radius server FORESCOUT-PRIMARY
 address ipv4 10.0.0.10 auth-port 1812 acct-port 1813
 key 7 <encrypted-key>
 timeout 5
 retransmit 3
```

---

## 5. CoA — Change of Authorization

### 5.1 Vị trí và vai trò

**CoA (RFC 5176)** là extension quan trọng nhất của RADIUS trong NAC hiện đại. Nó cho phép NAC server **chủ động thay đổi quyền truy cập** của một session *đang active* mà **không cần ngắt kết nối vật lý**. Đây là cơ chế thực thi policy động — nền tảng của automated incident response trong NAC.

```
Không có CoA (static):
  Endpoint kết nối → Xác thực → Nhận VLAN → [VLAN cố định mãi mãi]
  → NAC muốn quarantine: phải wait session timeout hoặc force disconnect

Có CoA (dynamic):
  Endpoint đang online trên VLAN 100
       │
       ▼
  Forescout phát hiện vi phạm (AV outdated)
       │
       ▼  CoA-Request → Switch
  Switch ngay lập tức thay đổi port VLAN: 100 → 999 (Quarantine)
       │
       ▼
  Endpoint nhận IP mới trong subnet 999, bị redirect đến remediation portal
```

### 5.2 CoA Flow chi tiết

```
NAC Server (CoA Client)               NAS / Switch (CoA Server)
        │                                         │
        │──── CoA-Request (UDP 3799) ────────────►│
        │     Code: 43                            │
        │     Attributes:                         │
        │       Calling-Station-Id: <MAC client>  │  ← Identify session
        │       NAS-IP-Address: <switch IP>        │
        │       Tunnel-Type: VLAN (13)             │  ← New policy
        │       Tunnel-Private-Group-Id: "999"     │
        │                                         │ [Switch tìm session theo MAC]
        │                                         │ [Apply new VLAN to port]
        │◄─── CoA-ACK ────────────────────────────│  ← Thành công
        │     hoặc                                │
        │◄─── CoA-NAK (Error-Cause attribute) ────│  ← Thất bại + lý do
```

**CoA Error-Cause codes quan trọng:**

| Code | Tên | Ý nghĩa |
|---|---|---|
| `201` | Residual Session Context Removed | Session đã bị xóa |
| `202` | Invalid EAP Conversation | EAP state không hợp lệ |
| `401` | Unsupported Attribute | Switch không hỗ trợ attribute được gửi |
| `402` | Missing Attribute | Thiếu attribute bắt buộc |
| `403` | NAS Identification Mismatch | NAS-IP hoặc NAS-Identifier không khớp |
| `404` | Invalid Request | Request không hợp lệ |
| `501` | Administratively Prohibited | Switch từ chối theo policy cục bộ |
| `503` | Session Context Not Found | Không tìm thấy session (MAC không tồn tại) |

### 5.3 Disconnect-Request

Khác với CoA (thay đổi quyền), **Disconnect-Request (Code 40)** ngắt hẳn session:

```
Forescout gửi Disconnect-Request
       │
       ▼
Switch xóa authentication session
Switch đưa port về Unauthorized state
Endpoint phải re-authenticate từ đầu
```

> **Use case thực tế**: Sau khi endpoint hoàn thành remediation (AV updated, patch installed), Forescout gửi Disconnect → Switch force re-auth → RADIUS reevaluate posture → cấp lại full access VLAN.

---

## 6. TACACS+

### 6.1 So sánh chi tiết RADIUS vs TACACS+

| Tiêu chí | RADIUS (RFC 2865) | TACACS+ (RFC 8907) |
|---|---|---|
| **Transport** | UDP (connectionless) | TCP (connection-oriented, reliable) |
| **Port** | 1812 (auth), 1813 (acct) | 49 |
| **Mã hóa** | Chỉ mã hóa password field | **Mã hóa toàn bộ packet body** |
| **AAA model** | Gộp chung trong một giao thức | **Tách biệt hoàn toàn** Authentication / Authorization / Accounting |
| **Authorization granularity** | Per-session (VLAN, ACL) | **Per-command** (từng lệnh CLI) |
| **Multiprotocol** | Có (PPP, SLIP, EAP...) | Hạn chế |
| **Use case chính** | **User/device network access** | **Device administration** (SSH/Telnet vào switch, router) |
| **Vendor** | Open standard | Cisco proprietary → RFC 8907 (2020) |

### 6.2 TACACS+ trong môi trường NAC

TACACS+ **không** dùng để authenticate endpoint devices. Vai trò của nó trong hạ tầng NAC là kiểm soát **admin access vào thiết bị mạng**:

```
Network Admin SSH vào Switch
        │
        ▼
Switch gửi TACACS+ Authentication-Request
        │  username: admin_john
        │  password: ******
        ▼
TACACS+ Server (Forescout / ISE / TACACS+ daemon)
        │
        ├── Authentication: Verify credential vs AD
        │
        ├── Authorization: Check role of admin_john
        │     → Role: "Network-ReadOnly"
        │     → Permitted commands: show *, ping, traceroute
        │     → Denied commands: conf t, write mem, reload
        │
        └── Accounting: Log every command executed
              → "admin_john ran: show running-config at 14:35:02"
```

### 6.3 TACACS+ Command Authorization (cấu hình Cisco IOS)

```cisco
! Cấu hình switch để dùng TACACS+ cho device management
aaa new-model
tacacs server NAC-TACACS
 address ipv4 10.0.0.10
 key TacacsSecret123

aaa authentication login default group tacacs+ local
aaa authorization exec default group tacacs+ local
aaa authorization commands 15 default group tacacs+ local
aaa accounting commands 15 default start-stop group tacacs+
```

---

## 7. DHCP — Role trong NAC

### 7.1 DHCP Fingerprinting — Device Identification

DHCP Option field trong DISCOVER/REQUEST packet mang đặc trưng riêng của từng OS/thiết bị, là nguồn dữ liệu **passive discovery** đầu tiên và quan trọng nhất của Forescout.

**Các DHCP Options dùng để fingerprint:**

| Option | Tên | Dữ liệu | Ví dụ phân tích |
|---|---|---|---|
| **55** | Parameter Request List | Danh sách options client yêu cầu | Windows 10: `1,15,3,6,44,46,47,31,33,121,249,252` |
| **60** | Vendor Class Identifier | Self-reported device class | `MSFT 5.0` = Windows; `android-dhcp-10` = Android 10 |
| **12** | Hostname | Tên máy | `DESKTOP-ABC123`, `iPhone`, `hikvision-cam-01` |
| **61** | Client Identifier | Thường là MAC + hardware type | Verify MAC không bị spoof |
| **77** | User Class | Application-specific | `iPXE`, `MSUC` (Lync/Teams phone) |
| **43** | Vendor Specific | Device-specific info | IP phones: Cisco/Avaya config server URL |

**Ví dụ nhận diện OS qua Option 55:**

```
Windows 10/11:    1, 15, 3, 6, 44, 46, 47, 31, 33, 121, 249, 252
macOS:            1, 121, 3, 6, 15, 119, 252, 95, 44, 46
Ubuntu Linux:     1, 28, 2, 3, 15, 6, 119, 12, 44, 47, 26, 121, 42
iOS iPhone:       1, 121, 3, 6, 15, 119, 252, 95, 44, 46
Android:          1, 33, 3, 6, 15, 28, 51, 58, 59
Cisco IP Phone:   1, 66, 150, 3, 6
```

### 7.2 DHCP Snooping — L2 Enforcement

**DHCP Snooping** trên switch ngăn DHCP rogue server và cung cấp **IP-MAC binding table** cho NAC:

```
Switch DHCP Snooping Binding Table:
╔══════════════════╦═══════════════╦══════╦═══════════╦════════════╗
║  MAC Address     ║  IP Address   ║ VLAN ║  Port     ║ Lease (s)  ║
╠══════════════════╬═══════════════╬══════╬═══════════╬════════════╣
║ 00:11:22:33:44:55║ 10.10.1.100  ║ 100  ║ Gi0/1     ║ 86400      ║
║ AA:BB:CC:DD:EE:FF║ 10.10.1.101  ║ 100  ║ Gi0/2     ║ 43200      ║
╚══════════════════╩═══════════════╩══════╩═══════════╩════════════╝

Trusted ports  (uplink → DHCP server):  Cho phép DHCP Reply
Untrusted ports (access → client):     Chặn DHCP Reply, chỉ cho DHCP Request
```

### 7.3 DHCP trong Forescout Enforcement

Forescout có thể làm **DHCP server** (tùy chọn) để kiểm soát IP assignment như một cơ chế enforcement:

```
Non-compliant device → DHCP Request
Forescout DHCP server (enforcement mode):
  └── Assign IP trong Quarantine subnet (10.10.99.0/24)
  └── Default gateway = Forescout NIC (inline mode) hoặc dedicated quarantine GW
  └── DNS = Forescout → redirect queries to remediation portal
```

---

## 8. SNMP — Discovery & Enforcement

### 8.1 SNMP trong Discovery

Forescout dùng SNMP để **query switch** lấy thông tin topology và đối chiếu với passive discovery data:

**OID quan trọng Forescout sử dụng:**

| MIB / OID | Thông tin thu thập | Mục đích |
|---|---|---|
| `1.3.6.1.2.1.17.4.3.1.1` — `dot1dTpFdbAddress` | MAC address table | Biết MAC nào kết nối vào port nào |
| `1.3.6.1.2.1.17.4.3.1.2` — `dot1dTpFdbPort` | Bridge port number | Map MAC → bridge port |
| `1.3.6.1.2.1.31.1.1.1.1` — `ifName` | Interface tên | Map port number → interface name (Gi0/1) |
| `1.3.6.1.2.1.4.22.1.2` — `ipNetToMediaPhysAddress` | ARP table | Map IP ↔ MAC |
| `1.3.6.1.2.1.2.2.1.8` — `ifOperStatus` | Port operational status | Port up/down |
| `1.3.6.1.4.1.9.9.46.1.3.1.1.2` — Cisco VLAN MIB | VLAN per port | VLAN assignment hiện tại |
| `1.0.8802.1.1.2.1.4` — LLDP MIB | LLDP neighbor table | Topology discovery |
| `1.3.6.1.4.1.9.9.23.1.2.1` — CDP MIB | CDP neighbor | Cisco topology |

### 8.2 SNMP Version Comparison

| Version | Auth | Encryption | Khuyến nghị |
|---|---|---|---|
| **v1** | Community string (plaintext) | Không | ❌ Không dùng |
| **v2c** | Community string (plaintext) | Không | ⚠ Chỉ trong môi trường isolated |
| **v3 authNoPriv** | SHA / MD5 | Không | ⚠ Hạn chế |
| **v3 authPriv** | SHA-256 / SHA-512 | AES-128 / AES-256 | ✅ **Bắt buộc trong production** |

### 8.3 SNMP Write — Enforcement (Forescout)

Khi **RADIUS CoA không khả dụng**, Forescout dùng SNMP Write để thay đổi VLAN:

```
# Forescout gửi SNMP SET để chuyển port sang Quarantine VLAN
# OID: vmVlan (Cisco VLAN Membership MIB)
snmpset -v3 -u fsuser -l authPriv -a SHA -A "AuthPass" -x AES -X "PrivPass" \
  10.0.0.1 \
  1.3.6.1.4.1.9.9.68.1.2.2.1.2.5 i 999
#                                          ^port index  ^new VLAN ID

# Disable port (nếu cần isolation cứng)
snmpset -v3 -u fsuser -l authPriv ... \
  10.0.0.1 \
  1.3.6.1.2.1.2.2.1.7.5 i 2
#                        ^ ifAdminStatus: 1=up, 2=down
```

> **Thứ tự ưu tiên enforcement của Forescout**: `RADIUS CoA` > `SSH CLI` > `SNMP Write`

---

## 9. NetFlow / IPFIX & Syslog

### 9.1 NetFlow / IPFIX — Behavioral Analysis

NetFlow (Cisco) / IPFIX (RFC 7011) cung cấp **flow-level visibility** — Forescout sử dụng để phát hiện anomaly sau khi device đã được cấp quyền vào mạng.

**Cấu trúc một Flow Record:**

```
Flow Record Example:
  Src IP:    10.10.1.100
  Dst IP:    10.10.2.0/24 (multiple hosts — scan pattern)
  Protocol:  TCP
  Dst Port:  445, 3389, 22, 80 (diverse ports — lateral movement)
  Bytes:     5,240
  Packets:   67
  Start:     14:35:01
  End:       14:35:03
  Flags:     SYN (no established connections — scan)
```

**Forescout phân tích NetFlow để phát hiện:**

| Pattern | Indicator | Action |
|---|---|---|
| Scan nhiều IP trong LAN | Lateral movement | Quarantine ngay lập tức |
| Traffic đến unexpected dst IP | C2 communication | Alert + Block |
| High volume outbound | Data exfiltration | Rate-limit + Alert |
| DNS query bất thường | DNS tunneling | Block DNS, Alert |
| Sudden new port traffic | Malware activity | Investigate |

### 9.2 Syslog — Audit và Correlation

```
Forescout nhận Syslog từ:
├── Switch / WLC: Authentication events, port up/down
├── Firewall: Permit/deny connections
├── RADIUS Server: Access-Accept/Reject records
└── Forescout tự sinh: Policy match, enforcement actions

Forescout gửi Syslog/CEF đến:
├── SIEM (Splunk, QRadar, Microsoft Sentinel)
├── SOAR (Palo Alto XSOAR, Splunk SOAR)
└── Syslog collector

Format CEF ví dụ:
CEF:0|Forescout|CounterACT|8.3|NAC:Quarantine|Device Quarantined|7|
  src=10.10.1.100 smac=00:11:22:33:44:55 
  msg=Device quarantined: AV not updated (30 days)
  reason=Policy:AV-Compliance-Check
  action=VLAN-Change:100->999
```

---