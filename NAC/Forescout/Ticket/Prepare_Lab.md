Đầu tiên lab này thực hiện cần cài đặt một số thứ sau:

1. PNETLAB và Ishare2
    Để cài được PNETLAB, ở đây mình xin cài PNETLAB version 6, được cài từ ubuntu 20.04 build lên
    Đầu tiên chúng ta sẽ tải image ubuntu server 20.04 về và run như cài máy ảo bình thường sau đó chúng ta, sau đó
    ```bash
        su root
    ```
    wget pnetlab.sh của mình về và chạy
    ```bash
        bash pnetlab.sh
    ```
    Sau khi chạy xong thì chúng ta vào giao diện CLI và tiếp tục install ishare2
    ```bash
       wget -O /usr/sbin/ishare2 https://raw.githubusercontent.com/ishare2-org/ishare2-cli/main/ishare2 && chmod +x /usr/sbin/ishare2 && ishare2
    ```
    Cài tiếp tục xong là hoàn thành bước chuẩn bị cho PNETLAB
    Trước khi chạy thì ta cần tải một số package bị thiếu trong quá trình install
    ```bash
       sudo apt install libsdl2-dev libsdl2-2.0-0 -y;
    ```
    Vậy là xong, có thể sử dụng

<div align="center">
  <img src="/images/image154.png" alt="Network Topology" width="100%"/>
</div>

2. Cài đặt ISO Forescout vào Pnetlab

Đầu tiên ta phải download được iso pnetlab, sau đó dùng Filezilla để chuyển file từ local sang máy Pnetlab
<div align="center">
  <img src="/images/image437.png" alt="Network Topology" width="100%"/>
</div>

**Lưu ý ** phải đặt tên folder đúng thì mới xuất hiện image trong template

Tiếp đến cd vào thư mục vừa tạo và đổi tên file iso thành cdrom.iso

```bash
mv Forescout_9.1.5_.iso cdrom.iso
```

Sau đó tạo hdd driver cho image này 

```bash
/opt/qemu/bin/qemu-img create -f qcow2 hda.qcow2 300G
```

Tiếp đến vào Pnetlab và tạo node sử dụng

​<div align="center">
  <img src="/images/image438.png" alt="Network Topology" width="100%"/>
</div>

**Lưu ý** phải click vào UEFI để tắt secure boot, nếu bản pnetlab thấp hơn 6.0 thì dùng lệnh như sau ở phần QEMU Option : 
```bash
-machine type=q35,accel=kvm -cpu host -drive if=pflash,format=raw,readonly=on,file=/usr/share/OVMF/OVMF_CODE.fd -drive if=pflash,format=raw,file=/usr/share/OVMF/OVMF_VARS.fd
```

Thay lệnh trên vào phần QEMU Option để chạy thành công.
​<div align="center">
  <img src="/images/image439.png" alt="Network Topology" width="100%"/>
</div>

Done

2. Upgrade từ Pnetlab 5.3 lên 6.0 ko cần cài lại hdh 
```bash
   sudo apt update && apt upgrade -y
   sudo do-release-upgrade
```