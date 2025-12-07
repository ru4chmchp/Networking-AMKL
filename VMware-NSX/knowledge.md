# Kiến thức cần thiết 

1. **SDN (Software-Defined Networking)** : Là mạng được định nghĩa bằng phần mềm. Vậy nó có nghĩa là gì ?
- Hãy lấy ví dụ : `trong mô hính mạng truyền thống thì mỗi thiết bị mạng như router, switch đề có "bộ não riêng" và tự đưa ra quyết định chuyển tiếp gói tin`. 
- Thì trong **SDN**, kiến trúc này sẽ cho phép tách biệt phần `bộ não` ra khỏi `cơ thể`. Và **SDN** chuyển quyền kiểm soát các cơ thể riêng lẻ sang tập chung, có nghĩa là một bộ não tập trung sẽ kiểm soát các cơ thể riêng lẻ.
- 2 phần tách biệt này trong **SDN** gọi là :
    - **Control Plane** : Đây là phần được tách ra và đừa về trung tâm quản lý gọi là **SDN Controller**. Controller này sẽ ra lệnh cho toàn bộ thiết bị mạng.
    - **Data Plane** : Đơn giản đây là phần cứng như `switch, router`, chúng chỉ chuyển tiếp gói tin theo lệnh của Controller.
- Kiến trúc 3 lớp của **SDN**
    - **Application Layer** : Chứa các ứng dụng và dịch vụ như 