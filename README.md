# Networking-AMKL

Đây là branch hướng dẫn về các phần mềm ảo hóa 

## 🚀 VMware Worstation Pro và Proxmox VE 8

# 🖥️ VMware Workstation Pro là gì?

**VMware Workstation Pro** là một phần mềm ảo hóa dạng **Type 2 hypervisor**, được cài đặt như một ứng dụng trên hệ điều hành Windows hoặc Linux. Nó cho phép người dùng tạo và chạy nhiều máy ảo (VM) trên một máy tính cá nhân.

Phần mềm này rất phổ biến trong môi trường **lab, thử nghiệm, học tập hoặc phát triển phần mềm**, nhờ giao diện đồ họa thân thiện và dễ sử dụng. Tuy nhiên, vì chạy trên nền hệ điều hành host, hiệu năng sẽ thấp hơn so với các hypervisor cấp thấp (bare-metal).

> 🔧 **Tính năng nổi bật:**
> - Hỗ trợ nhiều hệ điều hành guest: Windows, Linux, BSD, v.v.
> - Snapshot máy ảo
> - Kéo/thả file giữa host và guest
> - Hỗ trợ network ảo (NAT, Host-only, Bridge)

---

# 🖧  Proxmox VE 8 là gì?

**Proxmox Virtual Environment (Proxmox VE)** là một nền tảng ảo hóa mã nguồn mở dạng **Type 1 hypervisor (bare-metal)**. Phiên bản 8 là bản mới nhất tính đến hiện tại. Proxmox VE kết hợp cả **ảo hóa bằng KVM (Kernel-based Virtual Machine)** và **container LXC (Linux Containers)**, cung cấp một giải pháp mạnh mẽ cho việc triển khai máy chủ ảo trong môi trường production hoặc lab chuyên sâu.

Proxmox VE được cài trực tiếp lên phần cứng vật lý, không cần hệ điều hành trung gian, giúp tối ưu hiệu năng và tài nguyên. Giao diện quản trị qua trình duyệt cực kỳ trực quan, kèm theo nhiều tính năng nâng cao như: High Availability (HA), live migration, backup, snapshot, firewall, và cluster management.

> 🚀 **Tính năng nổi bật:**
> - Web GUI quản lý mạnh mẽ
> - Quản lý cả VM (KVM) và Container (LXC)
> - Hỗ trợ clustering, HA, và Ceph storage
> - Backup/restore theo lịch trình
> - Mã nguồn mở và miễn phí (có bản trả phí cho hỗ trợ doanh nghiệp)

## 🔍 So sánh VMware Workstation Pro và Proxmox VE 8

| Tiêu chí | 🖥️ VMware Workstation Pro | 🖧 Proxmox VE 8 |
|---------|---------------------------|----------------|
| **Loại Hypervisor** | Type 2 (chạy trên OS) | Type 1 (bare-metal, chạy trực tiếp trên phần cứng) |
| **Cài đặt trên** | Windows / Linux (như app desktop) | Máy chủ vật lý (bare-metal) |
| **Mục tiêu sử dụng** | Ảo hóa cho desktop/lab cá nhân | Ảo hóa server, hạ tầng production |
| **Hiệu năng** | Trung bình (chạy thông qua hệ điều hành host) | Cao (chạy trực tiếp trên phần cứng) |
| **Giao diện** | GUI desktop (native app) | Web-based GUI (quản lý qua trình duyệt) |
| **Hỗ trợ container (LXC)** | ❌ Không hỗ trợ | ✅ Có sẵn |
| **Hỗ trợ KVM / QEMU** | ❌ Không | ✅ Dùng KVM/QEMU trực tiếp |
| **Hỗ trợ HA / Cluster** | ❌ Không | ✅ Có tích hợp cluster, HA, live migration |
| **Giá** | Trả phí, nhưng các phiên bảng mới gần đây đã miễn phí rồi | Miễn phí (có bản enterprise trả phí để support) |

## 🔗 Tải file OVA

Tải tại đây: [Proxmox-VE8 link](https://www.proxmox.com/en/downloads)
	     [VMware Workstation Pro](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)
	     [Google Drive link](https://drive.google.com/drive/u/0/folders/1wC-rrIvU577-PyBZiMVC7315iHevf4v9)

## 💻 Hướng Dẫn Cài Đặt Lab Ảo Hóa

## 📌 Giới Thiệu

Dự án này cung cấp lab ảo hoá cho việc học tập và thử nghiệm với 2 lựa chọn:
- 🖥️ **VMware Workstation Pro** (dành cho máy cá nhân)
- 🧑‍🔧 **Proxmox VE 8** (dành cho máy chủ hoặc lab nâng cao)

---

## 📁 Yêu Cầu

- Bộ xử lý hỗ trợ ảo hóa (Intel VT-x hoặc AMD-V)
- RAM tối thiểu 8GB (khuyến nghị 16GB)
- Dung lượng trống ≥ 50GB
- File `.ova` hoặc `.iso` cài đặt hệ thống

---

## 🔽 Tải File Cần Thiết

| Tên File | Link Tải | Ghi chú |
|----------|----------|---------|
| Proxmox VE ISO | [Link bên trên](#🔗 Tải file OVA) | File `.iso` dùng để boot cài Proxmox |
| VMware Workstation Pro (.ova) || Máy ảo đã cấu hình sẵn |

---

## 🖥️ Cài Đặt với VMware Workstation Pro

### Bước 1: Nhập File OVA

1. Mở VMware Workstation Pro.
2. Chọn `File > Open`, trỏ đến file `.ova`.
3. Chọn nơi lưu máy ảo và nhấn `Import`.
4. Khởi động máy ảo và cấu hình nếu cần.

### Bước 2: Cấu Hình Mạng

- Cấu hình chế độ **Bridged** hoặc **NAT** tùy vào môi trường.
- Kiểm tra IP và kết nối mạng.

---

## 🧑‍🔧 Cài Đặt Proxmox VE 8 (Bare-metal)

### Bước 1: Tạo USB Boot

- Dùng [balenaEtcher](https://etcher.io/) hoặc Rufus để tạo USB từ file `.iso`.

### Bước 2: Cài Đặt Proxmox

1. Boot máy từ USB.
2. Chọn "Install Proxmox VE".
3. Làm theo hướng dẫn → Đặt hostname, password, IP tĩnh.

### Bước 3: Truy cập Web UI

- Truy cập tại: `https://IP-may-server:8006`
- Đăng nhập bằng `root` và mật khẩu đã tạo.

---

## 🛠️ Một Số Lệnh Hữu Ích (Proxmox)

```bash
# Cập nhật hệ thống
apt update && apt full-upgrade -y

# Xem trạng thái VM
qm list

# Dừng VM có ID 100
qm stop 100
