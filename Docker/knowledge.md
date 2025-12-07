# Kiến thức tổng quan về docker

## Docker là gì ?
> **Docker** là một nền tảng đóng gói ứng dụng và môi trường của nó vào một `container`. Mỗi `container` chạy độc lập, giống nhẹ, nhanh, giống nhau ở mọi nơi.

### Các kiến thức về docker

## Container là gì ?
> **Container** thì được hiểu như y nghĩa của nó `container` và bên trong nó sẽ gồm các thung hàng như : `application + dependency + runtime + config` và đóng gói lại thành một đơn vị duy nhất -> `container`
- Nó dùng chung `kernel` hệ điều hành nên :
    - khởi động rất nhanh
    - nhẹ hơn máy ảo vm rất nhiều
    - chạy được hàng trăm container trên 1 máy

## Kernel là gì ?
> **Kernel** là lõi của hệ điều hành (Linux, Windows, ...), nó nằm ở phần trung tâm của hệ điều hành: `User application` -> `System libraries` -> `Kernel` -> `Hardware` và nó làm nhiệm vụ quản lý CPU, RAM, DISK, NETWORK, .... 