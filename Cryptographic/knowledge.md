# Tổng hợp các kiến thức về mật mã học

## Hash Function (hàm băm)

- Mục đích là : toàn vẹn dữ liệu.
- Tính chất : một chiều, không thể đảo ngược, đầu ra luôn có định.
- Có 3 đặc tính quan trọng :
    - Deterministic : cùng input -> cùng output
    - Fast to compute : Tính nhanh
    - Pre-image resistant : Không thể tìm input từ output
- Các loại Hash phổ biến :
    - SHA-2 Family (an toàn nhất hiện nay)
        - SHA-256 → 256-bit output (32 bytes)
        - SHA-384 → 384-bit output (48 bytes)
        - SHA-512 → 512-bit output (64 bytes)
    - SHA-3 Family (tiêu chuẩn mới)
        - SHA3-256
        - SHA3-512
    - Cũ 
        - MD5 (128-bit) -> Quá ngắn + thuật toán yếu => có thể bị tấn công Birthday Attack
        - SHA-1 (160-bit) -> cũng có thể bị như trên hoặc bị Chosen-Prefix Collision

- Ứng dụng :
    - Kiểm tra toàn vẹn file
    - Hash mật khâu lưu vào DB
    - Chữ ký số ,....

## Hash Function (hàm băm)

- Mục đích là : toàn vẹn dữ liệu.
- Tính chất : một chiều, không thể đảo ngược, đầu ra luôn có định.
- Có 3 đặc tính quan trọng :
    - Deterministic : cùng input -> cùng output
    - Fast to compute : Tính nhanh
    - Pre-image resistant : Không thể tìm input từ output
- Các loại Hash phổ biến :
    - SHA-2 Family (an toàn nhất hiện nay)
        - SHA-256 → 256-bit output (32 bytes)
        - SHA-384 → 384-bit output (48 bytes)
        - SHA-512 → 512-bit output (64 bytes)
    - SHA-3 Family (tiêu chuẩn mới)
        - SHA3-256
        - SHA3-512
    - Cũ 
        - MD5 (128-bit) -> Quá ngắn + thuật toán yếu => có thể bị tấn công Birthday Attack
        - SHA-1 (160-bit) -> cũng có thể bị như trên hoặc bị Chosen-Prefix Collision

- Ứng dụng :
    - Kiểm tra toàn vẹn file
    - Hash mật khâu lưu vào DB
    - Chữ ký số ,....

