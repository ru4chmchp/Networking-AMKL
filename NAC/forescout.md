
**PHẦN II — FORESCOUT**

10. [Forescout — Định vị & So sánh](#10-forescout--định-vị--so-sánh)
11. [Forescout — Kiến trúc hệ thống](#11-forescout--kiến-trúc-hệ-thống)
12. [Forescout — Device Discovery & Profiling](#12-forescout--device-discovery--profiling)
13. [Forescout — Posture Assessment](#13-forescout--posture-assessment)
14. [Forescout — Policy Engine](#14-forescout--policy-engine)
15. [Forescout — Enforcement Engine](#15-forescout--enforcement-engine)
16. [Forescout — Tích hợp hệ sinh thái](#16-forescout--tích-hợp-hệ-sinh-thái)
17. [Lab Guide — Forescout từ đầu](#17-lab-guide--forescout-từ-đầu)



# PHẦN II — FORESCOUT

---

## 10. Forescout — Định vị & So sánh

### 10.1 Điểm khác biệt cốt lõi

Forescout được thiết kế với triết lý **"See everything, control everything"** — ưu tiên **agentless visibility** trước, enforcement sau. Khác với Cisco ISE hay Aruba ClearPass (802.1X-centric), Forescout hoạt động được **ngay cả khi không có 802.1X** trên switch.

```
Cisco ISE approach:        Forescout approach:
─────────────────          ──────────────────
"Authentication first"     "Visibility first"
  │                          │
  ▼                          ▼
802.1X mandatory           Discover ALL devices
  │                        (agentless, no change to network)
  ▼                          │
If no 802.1X: limited      Then enforce based on
  │                        full context
  ▼
IoT/OT: problem            IoT/OT: strength
```

### 10.2 So sánh Forescout vs Cisco ISE vs Aruba ClearPass

| Tiêu chí | Forescout | Cisco ISE | Aruba ClearPass |
|---|---|---|---|
| **Phương thức chính** | Agentless-first | 802.1X-first | 802.1X-first |
| **IoT/OT Visibility** | ★★★★★ | ★★★☆☆ | ★★★☆☆ |
| **OT/ICS Protocol** | ★★★★★ (eyeInspect) | ★★☆☆☆ | ★★☆☆☆ |
| **Hoạt động không cần 802.1X** | ✅ Có | ⚠ Hạn chế | ⚠ Hạn chế |
| **Multi-vendor network** | ★★★★☆ | ★★★☆☆ (Cisco-centric) | ★★★★☆ |
| **Deployment complexity** | Trung bình | Cao | Trung bình |
| **AI/ML Device Classification** | ★★★★★ | ★★★☆☆ | ★★★★☆ |
| **Healthcare/Medical Device** | ★★★★★ | ★★★☆☆ | ★★★☆☆ |
| **Licensing model** | Per-device | Per-device + feature | Per-device |

### 10.3 Forescout Product Portfolio

| Product | Tên cũ | Chức năng |
|---|---|---|
| **eyeControl** | CounterACT | Core engine: Discovery, Policy, Enforcement |
| **eyeSight** | (Module trong CounterACT) | Extended device visibility & classification |
| **eyeSegment** | — | Network segmentation, micro-segmentation |
| **eyeInspect** | SecurityMatters (acquired 2019) | OT/ICS passive protocol inspection (Modbus, DNP3, EtherNet/IP...) |
| **eyeExtend** | (Integration platform) | Third-party integration marketplace |
| **SilentDefense** | (OT Network Security) | OT-specific threat detection |

---

## 11. Forescout — Kiến trúc hệ thống

### 11.1 Kiến trúc tổng thể

```
                    ┌──────────────────────────────────────────────┐
                    │           Forescout Console                   │
                    │       (Web UI — HTTPS / port 443)            │
                    └───────────────────┬──────────────────────────┘
                                        │
                    ┌───────────────────▼──────────────────────────┐
                    │         Enterprise Manager (EM)              │
                    │  - Centralized management                    │
                    │  - Policy distribution                       │
                    │  - Aggregated device inventory               │
                    │  - Report & dashboard                        │
                    │  - HA: Active/Standby                        │
                    └──────┬──────────────────────┬───────────────┘
                           │                      │
              ┌────────────▼────────┐  ┌──────────▼───────────────┐
              │  CounterACT         │  │  CounterACT               │
              │  Appliance / VM     │  │  Appliance / VM           │
              │  (Site A)           │  │  (Site B)                 │
              │  - Network sensor   │  │  - Network sensor         │
              │  - Policy engine    │  │  - Policy engine          │
              │  - Active scanner   │  │  - Active scanner         │
              │  - Plugin manager   │  │  - Plugin manager         │
              └────────┬────────────┘  └──────────┬───────────────┘
                       │                          │
         ┌─────────────▼──────────────────────────▼──────────────┐
         │                  Network Infrastructure                │
         │   Core Switch ── Distribution ── Access Switch / WLC  │
         └──────────────────────────┬─────────────────────────────┘
                                    │
         ┌──────────────────────────▼─────────────────────────────┐
         │                      Endpoints                         │
         │  Windows PC / Mac / Linux / IoT / Printer / IP Cam    │
         └────────────────────────────────────────────────────────┘
```

### 11.2 CounterACT Appliance — Interfaces

Một CounterACT appliance có **hai loại interface** với chức năng hoàn toàn khác nhau:

```
CounterACT Appliance
├── Management Interface (eth0)
│   ├── IP: 10.0.0.10/24
│   ├── Kết nối đến: Enterprise Manager, Console, Admin SSH
│   ├── Giao tiếp với: Switch SNMP, WMI, SSH, RADIUS
│   └── Outbound: LDAP/AD, external APIs
│
└── Monitor Interface (eth1, eth2...)
    ├── NO IP ADDRESS (promiscuous mode)
    ├── Kết nối: SPAN/Mirror port trên switch
    ├── Lắng nghe: DHCP, ARP, DNS, NetFlow, EAPOL
    └── Không gửi traffic (passive only)
```

### 11.3 Deployment Modes

| Mode | Vị trí trong mạng | Cơ chế | Enforcement capability |
|---|---|---|---|
| **SPAN / Mirror Mode** | Out-of-band | Nhận copy traffic từ switch SPAN port | Gián tiếp: CoA, SNMP Write, SSH |
| **Switch Plugin Mode** | Out-of-band | API/SNMP/SSH tới switch | Gián tiếp: VLAN, port disable |
| **802.1X Integration** | Out-of-band | Forescout làm RADIUS server hoặc proxy | RADIUS attributes: VLAN, ACL, SGT |
| **Inline Mode** | In-path (inline) | Traffic thực sự đi qua appliance | Trực tiếp: drop packet, rate-limit |
| **DHCP Mode** | Out-of-band hoặc inline | Forescout làm DHCP server | IP assignment control |

> **Lab thường triển khai**: **SPAN Mode** (visibility) + **Switch Plugin** (SNMP/CoA enforcement) — đây là mode phổ biến nhất và ít ảnh hưởng đến hạ tầng hiện có nhất.

---

## 12. Forescout — Device Discovery & Profiling

### 12.1 Discovery Pipeline — Passive Methods

Passive methods **không tạo traffic** mà chỉ lắng nghe. Đây là ưu điểm lớn của Forescout — phát hiện thiết bị ngay từ khoảnh khắc đầu tiên mà không cần bất kỳ agent hoặc cấu hình trên endpoint.

```
Passive Discovery Sources:
┌──────────────────────────────────────────────────────────────────┐
│  DHCP Snooping                                                   │
│    Monitor port nhận DHCP DISCOVER/REQUEST/ACK                   │
│    → Lấy: MAC, hostname (option 12), OS fingerprint (option 55)  │
│    → Phát hiện: ngay khi device kết nối (trước khi có IP)        │
├──────────────────────────────────────────────────────────────────┤
│  ARP Monitoring                                                  │
│    Lắng nghe ARP request/reply trên SPAN traffic                │
│    → Lấy: IP-to-MAC mapping real-time                            │
│    → Phát hiện: IP conflict, gratuitous ARP (VLAN change)        │
├──────────────────────────────────────────────────────────────────┤
│  SPAN / Port Mirror Traffic Analysis                             │
│    Deep Packet Inspection trên mirrored traffic                  │
│    → HTTP User-Agent: browser type, OS version                   │
│    → mDNS / Bonjour: Apple devices, service names               │
│    → SMB NetBIOS: Windows hostname, workgroup, domain            │
│    → LLDP / CDP frames: device type, model, vendor              │
│    → EAPOL: 802.1X authentication events                        │
├──────────────────────────────────────────────────────────────────┤
│  NetFlow / IPFIX                                                 │
│    Flow records từ switch/router                                 │
│    → Traffic behavior: port usage, volume, destinations          │
└──────────────────────────────────────────────────────────────────┘
```

### 12.2 Discovery Pipeline — Active Methods

Active methods **tạo traffic** để thu thập thông tin chi tiết hơn. Chỉ kích hoạt sau khi passive discovery đã xác nhận device tồn tại.

```
Active Discovery Methods:
┌──────────────────────────────────────────────────────────────────┐
│  NMAP Scan                                                       │
│    Host discovery: ping sweep (ICMP, TCP SYN)                   │
│    Port scan: identify open services                             │
│    OS detection: TCP/IP stack fingerprinting                     │
│    Service version detection                                     │
├──────────────────────────────────────────────────────────────────┤
│  SNMP Query                                                      │
│    OID polling: sysDescr (OS info), sysName (hostname)          │
│    Walk device MIBs để lấy system information                    │
├──────────────────────────────────────────────────────────────────┤
│  WMI (Windows Management Instrumentation)                        │
│    Requires: domain credentials hoặc local admin                │
│    Win32_OperatingSystem: OS version, build, patch level         │
│    Win32_ComputerSystem: domain, model, username                 │
│    AntivirusProduct (SecurityCenter2): AV name, status, sig date │
│    Win32_QuickFixEngineering: installed patches (KB numbers)     │
│    Win32_Process: running processes                              │
├──────────────────────────────────────────────────────────────────┤
│  SSH (Linux / macOS / Network Devices)                           │
│    uname -a: OS và kernel version                                │
│    cat /etc/os-release: distro info                              │
│    ps aux: running processes                                     │
│    netstat / ss: open ports                                      │
├──────────────────────────────────────────────────────────────────┤
│  HTTP/HTTPS Probing                                              │
│    GET request → Server header: thiết bị tự khai báo            │
│    Ví dụ: Hikvision camera, Cisco IP Phone, Ubiquiti AP          │
└──────────────────────────────────────────────────────────────────┘
```

### 12.3 Device Classification Engine

```
Raw Data Inputs
(DHCP options, ARP, NMAP, HTTP headers, SNMP, WMI, traffic behavior)
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│            Forescout Classification Engine              │
│                                                         │
│  1. Rule-based matching                                 │
│     IF DHCP Option 60 contains "android-dhcp"          │
│     → Classify: Android Mobile Device                   │
│                                                         │
│  2. DHCP Fingerprint Database                          │
│     Match Option 55 sequence against known patterns     │
│                                                         │
│  3. TCP/IP Stack Fingerprinting                        │
│     TTL, window size, TCP options → OS detection        │
│                                                         │
│  4. HTTP User-Agent parsing                            │
│     "Mozilla/5.0 (Windows NT 10.0; Win64)" → Win10     │
│                                                         │
│  5. OUI (MAC prefix) lookup                            │
│     00:50:56:xx:xx:xx → VMware virtual machine         │
│                                                         │
│  6. AI/ML Behavioral Model                             │
│     Traffic patterns + port usage → device type        │
│                                                         │
│  7. Protocol-specific detection                        │
│     mDNS "_airplay._tcp" → Apple device                │
│     BACnet traffic → Building management device        │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
                Device Profile Result:
                ┌─────────────────────────────────┐
                │  MAC:     00:1A:2B:3C:4D:5E      │
                │  IP:      10.10.1.100             │
                │  Type:    IP Camera               │
                │  Vendor:  Hikvision               │
                │  OS:      Embedded Linux          │
                │  Model:   DS-2CD2143G0-I          │
                │  Port:    Switch01 Gi0/5 VLAN 100 │
                │  Risk:    Medium                  │
                │  Status:  New / Unverified        │
                └─────────────────────────────────┘
```

### 12.4 Device Inventory Attributes

| Category | Attributes | Data Source |
|---|---|---|
| **Network** | IP, MAC, VLAN, Switch, Port | DHCP, ARP, SNMP |
| **Identity** | Hostname, FQDN, Domain | DHCP opt.12, NetBIOS, DNS, WMI |
| **Classification** | Device Type, OS, OS Version, Vendor, Model | DHCP FP, NMAP, WMI, HTTP |
| **User** | Logged-in username, AD Group | WMI, Kerberos snooping, AD |
| **Security Posture** | AV product/status, Patch level, Encryption | WMI, SSH, Agent |
| **Connectivity** | First seen, Last seen, Online/Offline | Internal tracking |
| **Risk** | Risk score, Anomaly flags | Forescout scoring engine |
| **Compliance** | Policy compliance status, Violations | Policy evaluation result |

---

## 13. Forescout — Posture Assessment

### 13.1 Agentless Posture (WMI-based — Windows)

Forescout dùng **WMI remote query** để kiểm tra trạng thái bảo mật của Windows endpoint mà không cần cài agent:

```
Forescout WMI Queries:
┌──────────────────────────────────────────────────────────────────┐
│  AV Status (Security Center 2)                                   │
│  SELECT * FROM AntiVirusProduct WHERE ... (Win32 SEC2 namespace) │
│  → displayName, productState (0x1000 = enabled, 0x1010 = updated)│
├──────────────────────────────────────────────────────────────────┤
│  OS Patch Level                                                  │
│  SELECT HotFixID, InstalledOn FROM Win32_QuickFixEngineering     │
│  → List of installed KBs → compare vs required KB list          │
├──────────────────────────────────────────────────────────────────┤
│  BitLocker / Disk Encryption                                     │
│  SELECT ProtectionStatus FROM Win32_EncryptableVolume            │
│  → 0 = Unprotected, 1 = Protected, 2 = Protection Suspended     │
├──────────────────────────────────────────────────────────────────┤
│  Windows Firewall                                                │
│  Get-NetFirewallProfile -Profile Domain/Private/Public           │
│  → Enabled: True/False                                           │
├──────────────────────────────────────────────────────────────────┤
│  Running Processes                                               │
│  SELECT Name, ExecutablePath FROM Win32_Process                  │
│  → Detect unauthorized software                                  │
│  → Verify security agent running (EDR, DLP...)                   │
└──────────────────────────────────────────────────────────────────┘
```

### 13.2 Agent-based Posture (SecureConnector)

**Forescout SecureConnector** là persistent agent cung cấp deeper posture và real-time monitoring:

```
SecureConnector capabilities:
├── Real-time posture monitoring (không cần poll từ server)
├── Deeper compliance check (registry, file hash, certificate store)
├── Self-remediation actions:
│   ├── Trigger Windows Update
│   ├── Enable/disable Windows services
│   ├── Run remediation scripts
│   └── Display user notification popup
├── Continuous authentication (user identity tracking)
└── Communication: TCP 10003 (Forescout ↔ SecureConnector)
```

### 13.3 Posture Check Matrix

| Check | Agentless (WMI) | Agentless (SSH) | SecureConnector |
|---|---|---|---|
| OS patch level | ✅ | ✅ | ✅ |
| AV installed & updated | ✅ | ⚠ Limited | ✅ |
| Disk encryption | ✅ | ✅ | ✅ |
| Windows Firewall | ✅ | N/A | ✅ |
| Running process check | ✅ | ✅ | ✅ |
| Registry key check | ⚠ Limited | N/A | ✅ |
| Real-time change detection | ❌ (poll-based) | ❌ | ✅ |
| User notification | ❌ | ❌ | ✅ |
| Self-remediation | ❌ | ❌ | ✅ |
| **Cần credentials trên endpoint** | ✅ Domain admin | ✅ SSH key/pass | ❌ Agent tự auth |

---

## 14. Forescout — Policy Engine

### 14.1 Cấu trúc Policy

Mỗi policy trong Forescout gồm 3 thành phần: **Scope** (áp dụng cho ai), **Conditions** (điều kiện nào), và **Actions** (làm gì khi match).

```
╔═══════════════════════════════════════════════════════════════════╗
║                   FORESCOUT POLICY STRUCTURE                     ║
╠═══════════════════════════════════════════════════════════════════╣
║  Name: "Windows Compliance — AV & Patch Enforcement"             ║
╠═══════════════════════════════════════════════════════════════════╣
║  SCOPE (Áp dụng cho):                                            ║
║    ├── Device Type = Windows PC                                  ║
║    └── IP belongs to: 10.10.0.0/16 (Corporate networks)         ║
╠═══════════════════════════════════════════════════════════════════╣
║  CONDITIONS (Sub-rules — AND/OR logic):                          ║
║    ├── [Group: Non-Compliant] ◄──── trigger enforcement          ║
║    │   ├── [OR] AV Product: NOT installed                        ║
║    │   ├── [OR] AV Signatures: last updated > 7 days ago         ║
║    │   └── [OR] Critical OS Patches: ANY missing                 ║
║    │                                                             ║
║    └── [Group: Compliant] ◄──── trigger access grant            ║
║        ├── AV Product: installed AND running                     ║
║        ├── AV Signatures: updated within 3 days                  ║
║        └── Critical OS Patches: ALL installed                    ║
╠═══════════════════════════════════════════════════════════════════╣
║  ACTIONS (Per sub-rule):                                         ║
║    ├── [Non-Compliant]:                                          ║
║    │   ├── Assign VLAN: 999 (Quarantine)                        ║
║    │   ├── HTTP Redirect: https://remediation.corp.com           ║
║    │   ├── Send SNMP Trap to SIEM                               ║
║    │   └── Send Email to security@corp.com                       ║
║    │                                                             ║
║    └── [Compliant]:                                              ║
║        └── Assign VLAN: 100 (Corporate)                         ║
╚═══════════════════════════════════════════════════════════════════╝
```

### 14.2 Policy Condition Properties (chọn lọc)

| Category | Property | Ví dụ giá trị |
|---|---|---|
| **Network** | IP Address, Subnet, VLAN, Switch Port | `10.10.1.0/24`, VLAN `100` |
| **Classification** | Device Type, OS Family, OS Version | `Windows PC`, `iOS`, `Hikvision Camera` |
| **Identity** | Domain membership, AD Group, Username | `CORP\Domain Users`, logged-in user |
| **Compliance** | AV Product, AV Updated, Patch Count | `McAfee`, updated < 7 days |
| **Running Processes** | Process Name running/not-running | `CrowdStrikeFalconSensor.exe` |
| **Open Ports** | Port X is open | Port `3389` open (RDP exposed) |
| **Network Traffic** | Bytes sent, Destinations, Protocol | > 100MB to external IP |
| **Authentication** | 802.1X status, Auth method used | `EAP-TLS`, `MAB`, `Unauthenticated` |
| **Time** | Time-of-day, Day-of-week | Mon–Fri 08:00–18:00 |
| **Risk Score** | Forescout risk score | Score > 70 |
| **Custom Properties** | Plugin-defined properties | `CrowdStrike Prevention Policy = Active` |

### 14.3 Policy Priority và Processing

```
Devices in scope
      │
      ▼
Policy 1 (Priority: Highest)
  "Known Safe — IT Assets (Domain + EAP-TLS + AV Compliant)"
      │ Match? → Execute actions → [STOP or CONTINUE]
      │ No match?
      ▼
Policy 2 (Priority: 2)
  "Windows Non-Compliant — Quarantine"
      │ Match? → Quarantine VLAN 999
      │ No match?
      ▼
Policy 3 (Priority: 3)
  "Guest / BYOD — Limited Access"
      │ Match? → Guest VLAN 200
      │ No match?
      ▼
Policy N (Priority: Lowest — Catch-all)
  "Unknown Device — Block / Quarantine"
      └── Action: Quarantine VLAN 999 + Alert

─────────────────────────────────────────────
Best practices:
  ✅ Whitelist known-good ở trên cùng
  ✅ Specific rules trước, generic rules sau
  ✅ Catch-all "deny unknown" ở cuối cùng
  ✅ Dùng "Continue" action khi muốn áp nhiều policy
```

---

## 15. Forescout — Enforcement Engine

### 15.1 Network Enforcement Methods

```
Enforcement Decision Tree (Forescout):
                    ┌─────────────────────────────┐
                    │    Policy match: Quarantine  │
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │   Is 802.1X session active?  │
                    └──────┬───────────────┬───────┘
                          YES              NO
                           │               │
             ┌─────────────▼──┐   ┌────────▼──────────────┐
             │  Send RADIUS   │   │  Is SNMP Write         │
             │  CoA-Request   │   │  configured on switch? │
             │  (UDP 3799)    │   └────────┬───────────────┘
             └───────┬────────┘           │
                     │              YES   │   NO
                     │         ┌──────────┘   │
                     │         ▼              ▼
                     │  ┌──────────────┐  ┌───────────────┐
                     │  │  SNMP Write  │  │  SSH CLI       │
                     │  │  vmVlan OID  │  │  to switch     │
                     │  └──────┬───────┘  └───────┬───────┘
                     │         │                  │
                     └────┬────┘                  │
                          └──────────┬────────────┘
                                     ▼
                           Port VLAN changed to 999
                           Device gets new DHCP IP
```

### 15.2 Enforcement Actions — Chi tiết kỹ thuật

**Network-level Actions:**

| Action | Cơ chế | Điều kiện cần có |
|---|---|---|
| **VLAN Change** | RADIUS CoA `Tunnel-Private-Group-Id` | 802.1X session active + CoA support trên switch |
| **VLAN Change (no 802.1X)** | SNMP Write `vmVlan` OID | SNMP v3 Write access + Cisco VLAN MIB |
| **Port Disable** | SNMP Write `ifAdminStatus=2` | SNMP v3 Write access |
| **Apply ACL** | RADIUS CoA `Filter-Id` hoặc `Cisco-AVPair=ip:inacl#...` | Switch hỗ trợ dACL |
| **HTTP Redirect** | RADIUS `Cisco-AVPair=url-redirect=...` | Switch hỗ trợ redirect ACL + inline captive portal |
| **Deauthenticate Wireless** | WLC SNMP / REST API | WLC credentials configured |

**Host-level Actions (qua WMI / SSH / SecureConnector):**

| Action | Cơ chế | Use Case |
|---|---|---|
| **Start/Stop Windows Service** | WMI `Win32_Service.StartService()` | Bật lại Windows Defender |
| **Run Script** | WMI `Win32_Process.Create()` + SMB file copy | Chạy remediation script |
| **Kill Process** | WMI `Win32_Process.Terminate()` | Dừng unauthorized process |
| **User Notification** | SecureConnector popup | Thông báo user về vi phạm |
| **Force Windows Update** | WMI command | Trigger Windows Update |

### 15.3 Full Enforcement Flow — End-to-End

```
T+00s: Windows PC kết nối vào switch port Gi0/1
         │
T+02s: [PASSIVE] Forescout nhận DHCP DISCOVER qua SPAN
         → MAC: AA:BB:CC:11:22:33, DHCP Option 55 → Windows
         │
T+03s: [PASSIVE] ARP broadcast → IP: 10.10.1.50 resolved
         │
T+05s: [ACTIVE] SNMP query switch → "AA:BB:CC:11:22:33 trên Gi0/1, VLAN 100"
         │
T+08s: [ACTIVE] WMI query endpoint:
         → OS: Windows 10 22H2
         → AV: Windows Defender, last updated: 45 days ago ← vi phạm!
         → Patch: KB5025221 missing ← vi phạm!
         │
T+09s: [POLICY ENGINE] Evaluate:
         → Rule: "Windows Non-Compliant" → MATCH
         → Action: Quarantine VLAN 999
         │
T+09s: [ENFORCEMENT] Forescout checks:
         → 802.1X session active on Gi0/1? NO (MAB only)
         → SNMP Write configured? YES
         │
T+10s: [SNMP WRITE] SET vmVlan on Gi0/1 = 999
         → Switch ACKs
         │
T+11s: PC loses IP (10.10.1.50 no longer valid in VLAN 999)
         │
T+13s: PC sends DHCP DISCOVER in new VLAN 999
         → Receives: 10.10.99.50 (Quarantine subnet)
         → Default GW: 10.10.99.1 → routes to remediation server only
         │
T+15s: [HTTP REDIRECT] Browser request → HTTP 302 → https://remediation.corp.com
         → User sees: "Your device is not compliant. Update AV and patches."
         │
T+XX: User completes remediation (AV updated, patches installed)
         │
T+XX: [WMI RE-CHECK] Forescout polls again:
         → AV: updated 2 minutes ago ✅
         → Patches: all KB installed ✅
         │
T+XX: [ENFORCEMENT] Forescout SNMP Write: vmVlan Gi0/1 = 100
         → PC gets DHCP in Corporate VLAN → Full access restored
         │
T+XX: [ACCOUNTING] Event logged to SIEM:
         "Device AA:BB:CC:11:22:33 quarantined 14:35:10 → restored 14:52:33"
```

---

## 16. Forescout — Tích hợp hệ sinh thái

### 16.1 eyeExtend Integration Modules

```
Forescout eyeExtend Platform
         │
         ├── Security Stack
         │   ├── CrowdStrike Falcon  → Nhận EDR alert → trigger quarantine
         │   ├── Palo Alto NGFW      → Push dynamic address group
         │   ├── Splunk SIEM         → Forward device context + events
         │   ├── Microsoft Sentinel  → CEF log forwarding
         │   └── Cortex XDR          → Bi-directional incident response
         │
         ├── Identity & MDM
         │   ├── Microsoft Azure AD / Entra ID → User/device identity
         │   ├── Microsoft Intune    → MDM compliance status
         │   ├── Okta                → SSO context
         │   └── CyberArk            → PAM integration
         │
         ├── Network
         │   ├── Cisco ISE           → pxGrid: share session context
         │   ├── Cisco FMC           → Push IP-to-SGT mapping
         │   └── VMware NSX          → Micro-segmentation tags
         │
         └── ITSM / Workflow
             ├── ServiceNow          → Auto-create incident ticket
             ├── Jira                → Create security issue
             └── PagerDuty           → On-call alerting
```

### 16.2 RADIUS Integration Modes

| Mode | Mô tả | Khi nào dùng |
|---|---|---|
| **RADIUS Server** | Forescout xử lý 802.1X trực tiếp (built-in RADIUS) | Small-medium deployment, không có ISE/NPS |
| **RADIUS Proxy** | Forescout nhận Access-Request, forward đến ISE/NPS, nhận kết quả, bổ sung posture context rồi trả về switch | Khi đã có ISE/NPS, muốn thêm Forescout posture |
| **CoA Client only** | Forescout không làm RADIUS; chỉ gửi CoA sau posture | ISE/NPS làm auth chính, Forescout làm enforcement |
| **Passive RADIUS sniffer** | Forescout nghe RADIUS traffic qua SPAN | Visibility-only, không enforce |

---

## Tham khảo kỹ thuật

| Tài liệu | Nguồn |
|---|---|
| **IEEE 802.1X-2020** | Port-Based Network Access Control |
| **RFC 2865 / 2866** | RADIUS Authentication / Accounting |
| **RFC 3748** | Extensible Authentication Protocol (EAP) |
| **RFC 5176** | Dynamic Authorization Extensions to RADIUS (CoA) |
| **RFC 5216** | EAP-TLS Authentication Protocol |
| **RFC 6614** | Transport Layer Security (TLS) Encryption for RADIUS (RADSEC) |
| **RFC 7011** | Specification of the IPFIX Protocol |
| **RFC 8907** | TACACS+ Protocol |
| **NIST SP 800-207** | Zero Trust Architecture |
| **NIST SP 800-162** | Attribute Based Access Control (ABAC) |
| **Forescout eyeControl Admin Guide** | docs.forescout.com |
| **Forescout eyeExtend Connect SDK** | github.com/Forescout |