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