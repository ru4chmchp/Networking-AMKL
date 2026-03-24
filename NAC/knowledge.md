# Network Access Control (NAC)

---

## 1. Overview (What)

**Network Access Control (NAC)** là giải pháp bảo mật mạng cho phép tổ chức kiểm soát quyền truy cập dựa trên **danh tính người dùng**, **loại thiết bị**, và **trạng thái tuân thủ bảo mật (security posture)** — đảm bảo chỉ các thực thể hợp lệ và tuân thủ chính sách mới được phép kết nối vào hạ tầng mạng.

> NAC là thành phần cốt lõi trong kiến trúc **Zero Trust Network Access (ZTNA)**, thực thi nguyên tắc **"never trust, always verify"** ở tầng network.

---

## 2. Tại sao cần NAC? (Why)

| Yêu cầu | Vấn đề nếu không có NAC |
|---|---|
| **Security** | Không kiểm soát được thiết bị/người dùng trái phép truy cập vào tài nguyên nội bộ |
| **Privacy** | Không phân tách được luồng truy cập vào dữ liệu nhạy cảm |
| **Compliance** | Không đáp ứng được các tiêu chuẩn GDPR, HIPAA, SOX, ISO 27001 |
| **Zero Trust Alignment** | Không có cơ chế xác minh liên tục, phụ thuộc vào perimeter security |
| **Lateral Movement Prevention** | Sau khi bị xâm nhập, attacker tự do di chuyển trong mạng nội bộ |

---

## 3. Kiến trúc & Thành phần NAC

### 3.1 Core Components

```
[Endpoint Device]
       │
       ▼
[Network Device: Switch / WLC / VPN GW]
       │  802.1X / MAB / Web Auth
       ▼
[NAC Policy Engine]  ◄──── [Identity Store: AD / LDAP / IdP]
       │                   [Posture DB: Patch level, AV, EDR]
       ▼
[Policy Decision Point (PDP)]
       │  RADIUS / TACACS+ / REST
       ▼
[Policy Enforcement Point (PEP)]
       │  VLAN assignment / ACL / SGT / dACL
       ▼
[Network Resource Access]
```

### 3.2 Capabilities Matrix

| Capability | Mô tả | Công nghệ / Giao thức |
|---|---|---|
| **Visibility** | Phát hiện và định danh tất cả thiết bị trên mạng | Active: NMAP, WMI, SNMP, SSH — Passive: SPAN, DHCP, NetFlow, IPFIX, DPI, AI/ML profiling |
| **Authentication** | Xác minh danh tính người dùng / thiết bị | IEEE 802.1X, EAP-TLS, EAP-PEAP, MAB, RADIUS, TACACS+, MFA, X.509 Certificate |
| **Policy Definition** | Định nghĩa quy tắc truy cập theo điều kiện | Rule engine: role, device type, OS, auth method, posture, location, time-of-day |
| **Authorization** | Gán quyền truy cập tương ứng sau xác thực | VLAN assignment, dACL, SGT (Cisco TrustSec), downloadable ACL |
| **Posture Assessment** | Đánh giá trạng thái bảo mật của thiết bị | Agent-based (ISE Agent, Dissolvable Agent), Agentless scan, Compliance check (AV, patch, EDR) |
| **Enforcement** | Cho phép / từ chối / cách ly truy cập | Tích hợp 2 chiều với Firewall, IPS, EDR, SIEM |
| **Accounting** | Ghi nhận log truy cập cho audit | RADIUS Accounting, Syslog, SNMP Trap, SIEM integration |

---

## 4. Cơ chế hoạt động (How)

### 4.1 Luồng xử lý chuẩn (802.1X Flow)

```
Endpoint ──EAPOL Start──► Switch (Authenticator)
                                  │
                         RADIUS Access-Request
                                  │
                                  ▼
                          NAC / RADIUS Server (Authentication Server)
                                  │
                         [1] Xác thực credential (AD / LDAP / Certificate)
                         [2] Posture Assessment (AV status, patch level, OS compliance)
                         [3] Evaluate policy (role, device type, location, time)
                                  │
                         RADIUS Access-Accept / Reject
                         + VLAN / ACL / SGT attributes
                                  │
                                  ▼
                          Switch áp dụng policy:
                          - VLAN assignment
                          - dACL (downloadable ACL)
                          - SGT tagging (TrustSec)
```

### 4.2 Phương thức xác thực

| Phương thức | Mô tả | Use Case |
|---|---|---|
| **802.1X + EAP-TLS** | Certificate-based mutual authentication | Managed corporate devices |
| **802.1X + PEAP/MSCHAPv2** | Username/password qua PEAP tunnel | Domain-joined workstations |
| **MAB (MAC Auth Bypass)** | Xác thực dựa trên MAC address | IoT, printers, IP phones |
| **Web Authentication** | Captive portal với credentials | Guest users, BYOD |
| **Certificate + MFA** | Kết hợp certificate và OTP/Push | High-security environments |

---

## 5. Phân loại NAC

### 5.1 Theo thời điểm kiểm tra

| Loại | Cơ chế | Ưu điểm | Nhược điểm |
|---|---|---|---|
| **Pre-admission NAC** | Đánh giá và xác thực **trước khi** cấp quyền truy cập | Ngăn chặn thiết bị không tuân thủ ngay từ đầu | Phức tạp khi onboard thiết bị mới |
| **Post-admission NAC** | Giám sát hành vi và trạng thái **sau khi** cấp quyền | Phát hiện và phản ứng với sự cố real-time | Thiết bị đã vào mạng trước khi bị chặn |

### 5.2 Theo phương thức triển khai agent

| Loại | Cơ chế | Use Case |
|---|---|---|
| **Agent-based** | Cài persistent agent hoặc dissolvable agent trên endpoint để thu thập posture data | Managed corporate devices, high-security environments |
| **Agentless** | Scan từ xa qua SNMP, WMI, SSH, NMAP hoặc passive traffic analysis | IoT, BYOD, unmanaged devices |

### 5.3 Theo vị trí kiểm tra trong mạng

| Loại | Cơ chế | Ghi chú |
|---|---|---|
| **Inline NAC** | Traffic đi qua NAC appliance trực tiếp | Kiểm soát tốt hơn, single point of failure |
| **Out-of-band NAC** | NAC chỉ điều phối chính sách, traffic không đi qua | Scalable hơn, ít ảnh hưởng đến performance |

---

## 6. Use Cases

| Use Case | Mô tả | NAC Mechanism |
|---|---|---|
| **BYOD Management** | Kiểm soát thiết bị cá nhân truy cập mạng doanh nghiệp | Certificate enrollment, posture check, VLAN segmentation |
| **Guest Network** | Cho phép khách truy cập Internet, cách ly khỏi mạng nội bộ | Captive portal, Guest VLAN, bandwidth throttling |
| **IoT Onboarding** | Phát hiện, phân loại và phân vùng thiết bị IoT | MAB, AI/ML device profiling, IoT VLAN, micro-segmentation |
| **Incident Response** | Tự động cách ly thiết bị bị compromise | CoA (Change of Authorization), quarantine VLAN, firewall push |
| **Third-party / Partner** | Kiểm soát truy cập của nhà thầu, đối tác | Role-based VLAN, time-limited access, limited ACL |
| **Medical Devices** | Quản lý thiết bị y tế kết nối mạng | Agentless profiling, strict VLAN isolation, ACL enforcement |
| **Compliance** | Đảm bảo tuân thủ GDPR, HIPAA, SOX | Posture check, audit logging, automated remediation |

---

## 7. Tích hợp hệ sinh thái bảo mật

```
                    ┌─────────────────────────────────────┐
                    │           NAC Policy Engine          │
                    └──────────────┬──────────────────────┘
            ┌─────────────────────┼────────────────────────┐
            ▼                     ▼                         ▼
     [Identity Store]      [Security Tools]         [Network Devices]
     - Active Directory    - Firewall (FW)          - Switches (802.1X)
     - LDAP / IdP          - IPS / IDS              - Wireless LAN Controller
     - PKI / CA            - EDR / XDR              - VPN Gateway
     - MFA Provider        - SIEM / SOAR            - SD-WAN
     - CMDB                - Vulnerability Scanner   - Firewall (SGT/TrustSec)
```

**Giao thức tích hợp chính:**
- **RADIUS (RFC 2865/2866)** — Authentication, Authorization, Accounting
- **TACACS+ (RFC 8907)** — Device administration
- **CoA — Change of Authorization (RFC 5176)** — Dynamic policy update (quarantine, reauthentication)
- **REST API / pxGrid** — Tích hợp với SIEM, SOAR, Firewall
- **SNMP / Syslog** — Monitoring & logging

---

## 8. Các giải pháp NAC phổ biến

| Vendor | Sản phẩm | Đặc điểm nổi bật |
|---|---|---|
| **Cisco** | Identity Services Engine (ISE) | Tích hợp sâu với hạ tầng Cisco, hỗ trợ TrustSec / SGT, pxGrid |
| **Aruba (HPE)** | ClearPass Policy Manager | Multi-vendor support mạnh, profiling AI/ML |
| **Forescout** | eyeControl | Agentless, mạnh về IoT/OT visibility |
| **Portnox** | Cloud-native NAC | Cloud-delivered, dễ triển khai, phù hợp SMB |
| **Fortinet** | FortiNAC | Tích hợp với Fortinet Security Fabric |
| **Microsoft** | NPS + Conditional Access | Tích hợp với Azure AD / Entra ID |

---

## 9. Tham khảo

- IEEE 802.1X — Port-Based Network Access Control
- RFC 2865 — Remote Authentication Dial-In User Service (RADIUS)
- RFC 5176 — Dynamic Authorization Extensions to RADIUS (CoA)
- RFC 8907 — The TACACS+ Protocol
- NIST SP 800-162 — Guide to Attribute Based Access Control (ABAC)
- NIST SP 800-207 — Zero Trust Architecture
