# Kiến thức tổng quan về docker

## Docker là gì ?
**Docker** là một nền tảng đóng gói ứng dụng và môi trường của nó vào một `container`. Mỗi `container` chạy độc lập, giống nhẹ, nhanh, giống nhau ở mọi nơi.

Các thành phần chính của **Docker** là: 
- `Dockerfile` : mô tả lệnh để build một image.
- `Docker Image` : lằ một image sau khi build `Dockerfile` thành công.
- `Docker Container` : là một instance đang chạy image, có thể hiểu như là object của một class vậy.
- `Docker Engine` : là program chạy container.
- `Docker Hub` : là kho chứa image giống như github chứa các repo.

Lifecycle như sau : `Dockerfile -> build -> image -> run -> container`.

Ưu điểm lớn nhát của **Docker** đó là chạy ứng dụng giống nhau 100% ở tất cả môi trường khác nhau như server, laptop, máy dev, ... khác mà không bị lỗi phiên bản, thiếu thư viện hay khác OS, ... Khi nó chạy, nó đóng gói toàn bộ `application + dependency + runtime + config` vào một `container` và nó luôn sử dụng môi trường đó, không phụ thuộc vào máy host, đồng nhất ở mọi nơi.

### Docker Container là gì ?
Thực chất nó chỉ là công cụ dùng Linux để tạo ra **Container** = **process** + **namespace** + **cgroups**

Docker kết hợp 3 công nghẹ cẩu Linux : 
- Namespaces - cô lập tài nguyên.
- cgroups - giới hạn tài nguyên.
- UnionFS (OverlayFS) - filesystem dạng lớp.
- Capability + Seccomp - sandbox bảo mật.





### Các kiến thức bổ sung

####  Container là gì ?
**Container** thì được hiểu như y nghĩa của nó `container` và bên trong nó sẽ gồm các thùng hàng như : `application + dependency + runtime + config` và đóng gói lại thành một đơn vị duy nhất -> `container`.


||Container|Virtual Machine|
|-|-|-|
|Boot time|Vài giây|Vài phút|
|Resource|Rất nhẹ|Nặng|
|Kernel|Dùng chung kernel với host|Mỗi vm có kernel riêng|
|Target|Chạy app|Chạy OS đầy đủ|

- Nó dùng chung `kernel` hệ điều hành nên :
    - khởi động rất nhanh
    - nhẹ hơn máy ảo vm rất nhiều
    - chạy được hàng trăm container trên 1 máy


##### Kernel là gì ?
**Kernel** là lõi của hệ điều hành (Linux, Windows, ...), nó nằm ở phần trung tâm của hệ điều hành: `User application` -> `System libraries` -> `Kernel` -> `Hardware` và nó làm nhiệm vụ quản lý CPU, RAM, DISK, NETWORK, .... 

##### Namespaces là gì ?
**Namespaces** là cơ chế của `linux kernel` để cô lập tài nguyện giữa các `process`. Mỗi `container` thực chất là một nhóm `process`, và nhờ `namespaces`, các `process` đó thấy được: 
- PID riêng
- network riêng
- filesystem riêng
- user riêng
- hostname riêng

-> Và chúng nghĩ rằng mình là một máy độc lập

- Có 8 loại namespace chính :

|Namespaces|Ý nghĩa|
|-|-|
|pid|Mỗi container có PID riêng|
|net|Mỗi container có interface riêng (eth0), iptables riêng|
|mnt|Mỗi container có filesystem root riêng (/), mount riêng|
|ipc|Không container nào truy cập shared memory của container khác|
|uts|Container đặt hostname riêng|
|user|Root trong container không phải root thật trên host (an toàn hơn)|
|cgroup|Cô lập dùng để tránh lộ cấu trúc cgroups host|
|time|Cô lập clock/time|

##### OverlayFS (Union filesystem) là gì ?
Là một filesystem dạng lớp `layered`, cho phép ghép nhiều filesystem lại thành một filesystem duy nhất. Nó làm được nhờ cơ chế `read-only layers` từ **Docker Image** và `read-write layers` của **Container**. Khi **Docker Container** chạy, Docker tạo ra một filesystem như sau : **Container FS** = `Image Layers (read-only)` + `Container Layer (read-write)`.

- Lấy ví dụ dễ hiểu như sau : `Dockerfile` -> sinh ra `Docker Image` thì mỗi dòng `RUN/COPY/ADD` trong `Dockerfile` sẽ tạo ra một `layer read-only` và kết quả cuối cùng là nhiều `layer read-only` xếp chồng lên nhau:
```dockerfile
Layer 1: FROM ubuntu
Layer 2: RUN apt update
Layer 3: RUN apt install python3
Layer 4: COPY app.py /app/
Layer 5: RUN pip install -r requirements.txt
```
- Khi chạy `container` thì `docker` sẽ thêm một lớp `writable` và sau đó `container` ghi vào `Writable Container Layer

![alt text](images/1.png)`