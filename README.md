# 🧱 Networking-AMKL

> Đây là branch hướng dẫn về các phần mềm ảo hóa phổ biến như VMware Workstation Pro và Proxmox VE 8. Tài liệu này phù hợp cho người mới bắt đầu làm lab, thử nghiệm, hoặc xây dựng môi trường ảo hóa cho học tập và nghiên cứu.

---

## 📚 Mục Lục

- [🧱 Networking-AMKL](#-networking-amkl)
  - [📚 Mục Lục](#-mục-lục)
  - [🚀 VMware và Proxmox VE](#-vmware-và-proxmox-ve)
    - [🖥 VMware Workstation Pro là gì?](#-vmware-workstation-pro-là-gì)
    - [🖧 Proxmox VE 8 là gì?](#-proxmox-ve-8-là-gì)
  - [🔍 So sánh VMware Workstation Pro và Proxmox VE 8](#-so-sánh-vmware-workstation-pro-và-proxmox-ve-8)
  - [🔗 Tải File Cần Thiết](#-tải-file-cần-thiết)
  - [💻 Hướng Dẫn Cài Đặt](#-hướng-dẫn-cài-đặt)
    - [📁 Yêu Cầu Hệ Thống](#-yêu-cầu-hệ-thống)
    - [🖥 Cài Đặt VMware Workstation Pro](#-cài-đặt-vmware-workstation-pro)
      - [Các bước cài đặt trên Linux](#các-bước-cài-đặt-trên-linux)
    - [🧑‍🔧 Cài Đặt Proxmox VE 8 (Bare-metal)](#-cài-đặt-proxmox-ve-8-bare-metal)

---

## 🚀 VMware và Proxmox VE

### 🖥 VMware Workstation Pro là gì?

**VMware Workstation Pro** là một phần mềm ảo hóa dạng **Type 2 hypervisor**, cài như ứng dụng trên Windows hoặc Linux. Nó cho phép bạn tạo và chạy nhiều máy ảo (VM) trên máy tính cá nhân.

> **Ứng dụng phổ biến:** lab, thử nghiệm, học tập, phát triển phần mềm.

> 🔧 **Tính năng nổi bật:**
>
> - Hỗ trợ nhiều hệ điều hành guest: Windows, Linux, BSD,...
> - Snapshot máy ảo
> - Kéo/thả file giữa host và guest
> - Hỗ trợ nhiều dạng mạng ảo (NAT, Host-only, Bridge)

---

### 🖧 Proxmox VE 8 là gì?

**Proxmox Virtual Environment (Proxmox VE)** là một nền tảng ảo hóa mã nguồn mở dạng **Type 1 hypervisor (bare-metal)**, kết hợp cả **KVM** và **LXC**.

> Proxmox VE 8 được cài trực tiếp lên phần cứng vật lý, không cần OS trung gian → hiệu năng cao, ổn định cho môi trường production.

> 🚀 **Tính năng nổi bật:**
>
> - Web GUI quản lý hiện đại
> - Hỗ trợ cả VM (KVM) và Container (LXC)
> - Hỗ trợ clustering, HA, Ceph Storage
> - Tích hợp backup/snapshot
> - Mã nguồn mở, miễn phí (bản enterprise có hỗ trợ kỹ thuật)

---

## 🔍 So sánh VMware Workstation Pro và Proxmox VE 8

| Tiêu chí | 🖥 VMware Workstation Pro | 🖧 Proxmox VE 8 |
|---------|---------------------------|----------------|
| **Loại Hypervisor** | Type 2 (chạy trên OS) | Type 1 (bare-metal) |
| **Cài đặt trên** | Windows / Linux (app desktop) | Máy chủ vật lý |
| **Mục tiêu sử dụng** | Desktop / Lab cá nhân | Server / Hạ tầng production |
| **Hiệu năng** | Trung bình | Cao |
| **Giao diện** | GUI Desktop | Web-based GUI |
| **Hỗ trợ Container (LXC)** | ❌ Không | ✅ Có |
| **Hỗ trợ KVM / QEMU** | ❌ Không | ✅ Có |
| **Cluster, HA, Live migration** | ❌ Không | ✅ Có |
| **Chi phí** | Trả phí (gần đây miễn phí) | Miễn phí (bản enterprise có phí) |

---

## 🔗 Tải File Cần Thiết

- [📥 Proxmox VE 8](https://www.proxmox.com/en/downloads)
- [📥 VMware Workstation Pro](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)
- [📥 Google Drive (file OVA)](https://drive.google.com/drive/u/0/folders/1wC-rrIvU577-PyBZiMVC7315iHevf4v9)

---

## 💻 Hướng Dẫn Cài Đặt

### 📁 Yêu Cầu Hệ Thống

- CPU hỗ trợ ảo hóa (Intel VT-x / AMD-V)
- RAM: Tối thiểu **8GB** (khuyến nghị **16GB** trở lên)
- Dung lượng ổ cứng trống: **≥ 50GB**
- File cài đặt: `.iso` hoặc `.ova`

---

### 🖥 Cài Đặt VMware Workstation Pro

> ✅ **Windows**: Cài đặt bình thường bằng file `.exe`  
> 🐧 **Linux**: Cần cài thêm một số gói phụ thuộc trước khi sử dụng

#### Các bước cài đặt trên Linux

1. **Cập nhật hệ thống:**

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Cài đặt kernel headers tương ứng với phiên bản đang dùng:**

   ```bash
   uname -r  # Xem phiên bản kernel
   sudo apt install linux-headers-$(uname -r)
   ```

3. **Cài đặt các gói phụ thuộc cần thiết:**

   ```bash
   sudo apt install build-essential dkms linux-headers-$(uname -r)
   ```

4. **Khắc phục lỗi thiếu 2 gói vmware khi chạy: Clone và cài đặt thủ công các module:**

   - [🔗 Link 1 - mkubecek/vmware-host-modules](https://github.com/mkubecek/vmware-host-modules/tree/master)

   - [🔗 Link 2 - bytium/vm-host-modules](https://github.com/bytium/vm-host-modules/)

5. **Hoàn tất và khởi chạy VMware như bình thường.**

---

### 🧑‍🔧 Cài Đặt Proxmox VE 8 (Bare-metal)

> Bạn có thể cài đặt trực tiếp lên máy vật lý hoặc test lab bằng VMware, Mình có thầy một anh dựng Proxmox VE8 trên VMware Workstation Pro khá hay, nếu mọi người muốn tìm hiểu về Proxmox thì theo link sau:

- 🔍 Video hướng dẫn cài đặt Proxmox trên VMware: [🎥 Xem tại đây](https://www.youtube.com/watch?v=F0Ta8pMyo7w&t=2158s)

> Nếu bạn muốn dựng lab lớn, Proxmox VE là phần mềm mã nguồn mở rất đáng giá để thay thế cho VMware ESXi.
