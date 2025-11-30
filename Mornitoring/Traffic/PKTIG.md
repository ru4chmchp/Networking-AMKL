# Xây Dựng Hệ Thống Giám Sát và Phân Tích Lưu Lượng Mạng

> Triển khai một hệ thống giám sát mạng toàn diện (Network Monitoring Pipeline) sử dụng các công cụ mã nguồn mở để thu thập, xử lý, lưu trữ và trực quan hóa lưu lượng mạng (NetFlow & sFlow). Hệ thống được mô phỏng trên nền tảng PNETLab với kiến trúc phân tán để tối ưu hiệu năng và tài nguyên.

**Lưu ý quan trọng**: Do hạn chế về RAM trong môi trường lab, hệ thống được thiết kế tách biệt Exporter và Collector. Trong đó Collector được triển khai trên VPS riêng. Nếu bạn có đủ tài nguyên RAM, có thể triển khai toàn bộ trên PNETLab.

## Mục Lục

- [Tổng quan kiến trúc](#tổng-quan-kiến-trúc)
- [Mô hình mạng](#mô-hình-mạng)
- [Hướng dẫn triển khai chi tiết](#hướng-dẫn-triển-khai-chi-tiết)
  - [1. Flow Exporters (Softflowd & OVS)](#1-flow-exporters-softflowd--ovs)
  - [2. Flow Collector (pmacct)](#2-flow-collector-pmacct)
  - [3. Message Queue (Kafka)](#3-message-queue-kafka)
  - [4. Data Forwarder (Telegraf)](#4-data-forwarder-telegraf)
  - [5. Storage (InfluxDB)](#5-storage-influxdb)
  - [6. Enrich (Python + GeoIP Mapping)](#6-enrich-python--geoip-mapping)
  - [7. Visualization (Grafana)](#7-visualization-grafana)
- [Kết quả và đánh giá](#kết-quả-và-đánh-giá)

## Tổng quan kiến trúc

Hệ thống hoạt động theo quy trình pipeline xử lý dữ liệu luồng (flow data) từ thu thập đến trực quan hóa:

<div align="center">
<img src="/images/tq.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

**Quy trình xử lý dữ liệu**:

1. **Thu thập (Collection)**: Dữ liệu NetFlow/sFlow từ thiết bị mạng (pfSense, OVS) gửi về Collector
2. **Streaming**: Collector đẩy dữ liệu vào Kafka topic theo định dạng JSON
3. **Xử lý (Processing)**: Telegraf tiêu thụ dữ liệu từ Kafka và chuyển đổi sang định dạng time-series
4. **Lưu trữ (Storage)**: InfluxDB lưu trữ dữ liệu dưới dạng chuỗi thời gian
5. **Làm giàu dữ liệu (Enrich)**: Python script sử dụng GeoIP để bổ sung thông tin địa lý, ASN
6. **Trực quan hóa (Visualization)**: Grafana truy vấn InfluxDB để hiển thị dashboard giám sát

## Mô hình mạng

<div align="center">
<img src="/images/model.png" alt="Mô hình mạng triển khai" width="1200"/>
</div>

**Thành phần hệ thống**:
- **Exporters**: pfSense (Softflowd) và Open vSwitch (sFlow)
- **Collector**: pmacct trên VPS riêng biệt
- **Processing Stack**: Kafka + Telegraf + InfluxDB
- **Visualization**: Grafana với dashboard tùy chỉnh

## Hướng dẫn triển khai chi tiết

### 1. Flow Exporters (Softflowd & OVS)

#### a. Cài đặt và cấu hình Softflowd trên pfSense (NetFlow)
- Có thể sử dụng OPNsense, Cisco, ...
**Bước 1: Cài đặt Softflowd Package**
- Truy cập pfSense Web GUI → System → Package Manager
- Tìm và cài đặt package "softflowd"

<div align="center">
<img src="/images/softflowd.png" alt="Cài đặt Softflowd trên pfSense" width="1200"/>
</div>

**Bước 2: Cấu hình Softflowd**
- Services → Softflowd → Configuration
- Chọn interface WAN làm nguồn export flow data
- Cấu hình collector IP và port (mặc định: 2045)

<div align="center">
<img src="/images/config-softflowd.png" alt="Cấu hình Softflowd" width="1200"/>
</div>
<div align="center"> 
<img src="/images/config-add.png" alt="Cấu hình bổ sung Softflowd" width="1200"/> 
</div>

#### b. Cài đặt và cấu hình Open vSwitch trên Linux (sFlow)
- Có thể sử dụng vEOS, Jupiter, ... để thay thế.
**Bước 1: Cài đặt Open vSwitch**

```bash
# Trên Ubuntu/Debian
sudo apt update
sudo apt install openvswitch-switch

# Trên CentOS/RHEL
sudo yum install openvswitch
```

**Bước 2: Cấu hình sFlow Agent**

```bash
# Tạo bridge OVS
sudo ovs-vsctl add-br br0

# Thêm port vật lý vào bridge
sudo ovs-vsctl add-port br0 eth0

# Cấu hình sFlow
sudo ovs-vsctl -- --id=@sflow create sflow agent=eth0 \
target=\"152.94.194.150:6343\" header=128 sampling=512 \
polling=10 -- set bridge br0 sflow=@sflow
```

<div align="center"> 
<img src="/images/openv.png" alt="Cấu hình Open vSwitch sFlow" width="1200"/> 
</div>

### 2. Flow Collector (pmacct)
- Có thể tham khảo các Collector khác như Flowd của ELK, Akvorado của FreeISP (Pháp), ...
- Có các package đi kèm nfacctd và sfacctd
#### a. Cài đặt pmacct

```bash
# Trên Ubuntu/Debian
sudo apt install pmacct

# Hoặc biên dịch từ source
git clone https://github.com/pmacct/pmacct
cd pmacct
./autogen.sh
./configure
make
sudo make install
```
#### b. Cấu hình Nfacctd (NetFlow Collector)

<div align="center"> <img src="/images/nfacctd.png" alt="Cấu hình Nfacctd" width="1200"/> </div>

#### c. Cấu hình Sfacctd (sFlow Collector)

<div align="center"> <img src="/images/sfacctd.png" alt="Cấu hình Sfacctd" width="1200"/> </div>


**Khởi động dịch vụ**

```bash
sudo systemctl start nfacctd
sudo systemctl start sfacctd
sudo systemctl enable nfacctd
sudo systemctl enable sfacctd
```

### 3. Message Queue (Kafka)

#### a. Cài đặt Kafka

```bash
# Tải và giải nén Kafka
wget https://downloads.apache.org/kafka/3.4.0/kafka_2.13-3.4.0.tgz
tar -xzf kafka_2.13-3.4.0.tgz
cd kafka_2.13-3.4.0
```

#### b. Cấu hình server.properties

<div align="center"> <img src="/images/kafka-server.png" alt="Cấu hình Kafka Server" width="1200"/> </div>

#### c. Tạo topic

<div align="center"> <img src="/images/topic.png" alt="Tạo Kafka Topics" width="1200"/> </div>

#### d. Kiểm tra dữ liệu

<div align="center"> <img src="/images/image.png" alt="Kiểm tra dữ liệu Kafka" width="1200"/> </div>

**Lý do sử dụng Kafka**: Pmacct không hỗ trợ gửi trực tiếp đến InfluxDB. Kafka đóng vai trò buffer và cho phép xử lý real-time, dễ dàng mở rộng hệ thống sau này.

### 4. Data Forwarder (Telegraf)

#### a. Cài đặt Telegraf

```bash
# Trên Ubuntu/Debian
wget -q https://repos.influxdata.com/influxdata-archive.key
sudo apt-key add influxdata-archive.key
echo "deb https://repos.influxdata.com/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/influxdata.list
sudo apt update && sudo apt install telegraf
```

#### b. Cấu hình Input Kafka

<div align="center"> <img src="/images/telegraf.png" alt="Cấu hình Telegraf Input" width="1200"/> </div><div align="center"> <img src="/images/influxdb.png" alt="Cấu hình Telegraf Output" width="1200"/> </div>

### 5. Storage (InfluxDB)

#### a. Cài đặt InfluxDB

```bash
# Trên Ubuntu/Debian
wget -q https://repos.influxdata.com/influxdata-archive.key
sudo apt-key add influxdata-archive.key
echo "deb https://repos.influxdata.com/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/influxdata.list
sudo apt update && sudo apt install influxdb
sudo systemctl start influxdb
sudo systemctl enable influxdb
```

#### b. Cấu hình Bucket
- Phần này mình quên chụp nên các bạn có thể tự cấu hình, thông thường khi cấu hình xong dữ liệu netflow và sflow sẽ trả về kafka_consumer

#### c. Hiển thị dữ liệu

<div align="center"> <img src="/images/check.png" alt="Kiểm tra dữ liệu InfluxDB" width="1200"/> </div>

### 6. Enrich (Python + GeoIP Mapping)

#### a. Cài đặt thư viện cần thiết

```bash
pip install influxdb geoip2 pandas requests
```

#### b. Tải GeoIP database

```bash
wget https://geolite.maxmind.com/download/geoip/database/GeoLite2-City.tar.gz
wget https://geolite.maxmind.com/download/geoip/database/GeoLite2-ASN.tar.gz
tar -xzf GeoLite2-City.tar.gz
tar -xzf GeoLite2-ASN.tar.gz
```

#### c. Viết code
- Phần này các bạn có thể tự viết vì đơn giản mình không giỏi phần này nên không biết các chỉ các bạn nhưng phần mapping hãy cố tối ưu bằng các cách như chia luồng code để mapping, lưu các ip đã mapping vào cache để tránh mapping lại, phân các dữ liệu thành các gói 500 - 1000 để giải quyết sẽ tối ưu thời gian mapping hơn.
- Sau đó tạo service để chạy sau một khoảng thời gian nào đó.

### 7. Visualization (Grafana)

#### a. Cài đặt Grafana

```bash
# Trên Ubuntu/Debian
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

#### b. Cấu hình Datasource

- Phần này các bạn có thể tìm InfluxDB và import vào sau đó chuẩn bị các query truy vấn bằng Flux(Ngôn ngữ truy vấn của InfluxDB)

#### c. Tạo Dashboard

- Phần này có thể nhờ GPT filter, chọn những kiểu data phù hợp với kiểu hiển thị dữ liệu

<div align="center"> <img src="/images/visual.png" alt="Dashboard Grafana" width="1200"/> </div>

### Kết quả và đánh giá
Thành công đạt được

    ✅Hệ thống thu thập và xử lý real-time NetFlow/sFlow
    ✅Kiến trúc phân tán, dễ mở rộng
    ✅Dữ liệu được làm giàu với thông tin địa lý, ASN
    ✅Dashboard trực quan với đa dạng biểu đồ
    ✅Khả năng giám sát toàn diện lưu lượng mạng

Ưu điểm hệ thống

    Mã nguồn mở: Giảm chi phí triển khai
    Scalable: Dễ dàng mở rộng qua Kafka
    Real-time: Xử lý dữ liệu thời gian thực
    Flexible: Hỗ trợ cả NetFlow và sFlow

Hạn chế và hướng phát triển

    Độ phức tạp cao, cần kiến thức về multiple technologies
    Có thể tối ưu hiệu năng với cluster InfluxDB
    Bổ sung cảnh báo tự động (alerting)
    Tích hợp machine learning cho anomaly detection