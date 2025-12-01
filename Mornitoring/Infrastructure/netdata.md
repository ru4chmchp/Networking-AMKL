# Hệ Thống Giám Sát Mạng Với Netdata Và Machine Learning Để Phát Hiện Bất Thường

Dự án này triển khai một hệ thống giám sát mạng toàn diện sử dụng **Netdata** để theo dõi hiệu năng hạ tầng (Router, Switch) và dịch vụ (Web Server), kết hợp với mô hình **Machine Learning (Isolation Forest)** để phát hiện các bất thường trong thời gian thực.

---

## Mục lục
- [Tổng quan](#tổng-quan)
- [Mô hình hệ thống](#mô-hình-hệ-thống)
- [Yêu cầu cài đặt](#yêu-cầu-cài-đặt)
- [Triển khai Netdata](#triển-khai-netdata)
- [Cấu hình Giám sát](#cấu-hình-giám-sát)
    - [Giám sát Hạ tầng (SNMP)](#1-giám-sát-hạ-tầng-qua-snmp)
    - [Giám sát Dịch vụ (Apache)](#2-giám-sát-dịch-vụ-apache)
- [Cấu hình Cảnh báo (Alerts)](#cấu-hình-cảnh-báo-alerts)
- [Tích hợp Machine Learning](#tích-hợp-machine-learning)
- [Kết quả](#kết-quả)

---

## Tổng quan
Hệ thống giải quyết các vấn đề giám sát thủ công truyền thống bằng cách cung cấp khả năng quan sát thời gian thực (real-time) với độ trễ thấp.
* **Thu thập dữ liệu:** Netdata (Agent-based & SNMP).
* **Lưu trữ:** InfluxDB (Time-series database).
* **Phân tích:** Python & Isolation Forest (Anomaly Detection).
* **Cảnh báo:** Email (thông qua `msmtp`).

## Mô hình hệ thống
Mô hình mạng được mô phỏng trên **PnetLab** bao gồm:
* **Router Core & Firewall (pfSense):** Quản lý kết nối ra Internet và phân vùng mạng.
* **Vùng DMZ:** Chứa Web Server (Apache).
* **Vùng Nội bộ (LAN):** Chứa Netdata Monitoring Server.
* **Thiết bị mạng:** Switch Layer 3 (Cisco).


<div align="center">
<img src="/images/netdata.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>


---

## Yêu cầu cài đặt
* **OS:** Ubuntu Server (cho Netdata Server).
* **Môi trường:** PnetLab (để mô phỏng mạng).
* **Phần mềm:**
    * Netdata
    * InfluxDB & Telegraf
    * Python 3 (thư viện `scikit-learn`, `pandas`)
    * Msmtp (để gửi mail)

---

## Triển khai Netdata

Cài đặt Netdata trên Ubuntu Server bằng lệnh one-line:

```bash
wget -O /tmp/netdata-kickstart.sh [https://get.netdata.cloud/kickstart.sh](https://get.netdata.cloud/kickstart.sh) && sh /tmp/netdata-kickstart.sh
Sau khi cài đặt, truy cập Dashboard tại: http://<IP-Server>:19999
```

<div align="center">
<img src="/images/topo.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

Ở đây nếu cái trên các node khác như Pfsense hoặc OPNsense thì sẽ có cách cài khác : [Click vào đây](https://learn.netdata.cloud/docs/netdata-agent/installation/pfsense)

## Cấu hình giám sát

### 1. Giám sát hạ tầng qua SNMP

Cấu hình Netdata để thu thập metrics từ Switch/Router Cisco. File cấu hình: `/etc/netdata/go.d/snmp.conf`


<div align="center">
<img src="/images/conf.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

Sau đó kiểm tra trong phân hiển thị SNMP của Netdata

<div align="center">
<img src="/images/cpu.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

### 2. Giám sát dịch vụ apache

Sau khi đã thiết lập giám sát hạ tầng, ở phần này  tiến hành cấu hình giám sát thượng tầng, cụ thể là các dịch vụ đang chạy trên máy chủ. Để demo chức năng này, chọn giám sát dịch vụ Apache HTTP Server đang chạy trên máy chủ web (Web Server) tại vùng DMZ của mô hình mạng.

Netdata có hỗ trợ plugin apache sử dụng go.d.plugin, tương tự như cách thu thập SNMP. Cấu hình cũng được thực hiện bằng cách tạo một file .conf trong thư mục `/etc/netdata/go.d/`, ví dụ là `apache.conf`

<div align="center">
<img src="/images/apache.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

Sau khi lưu cấu hình, restart lại Netdata là có thể thấy ngay thông tin về tình trạng hoạt động của Apache trên giao diện Netdata, bao gồm các chỉ số như: Số lượng request hiện tại, lượng truy cập mỗi giây, tỷ lệ xử lý request theo worker, trạng thái các
worker (Busy/Idle),...

<div align="center">
<img src="/images/apa1.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

## Cấu hình cảnh báo alerts

### 1. Cấu hình cảnh báo các thiết bị hạ tầng
Netdata hỗ trợ cấu hình cảnh báo thông qua các file `.conf` trong thư mục `health.d`. Các file này cũng sử dụng cú pháp YAML (tương tự cấu hình thu thập trong `go.d`) để định nghĩa các rule cảnh báo (alert rule). Mỗi rule sẽ được gắn với một biểu đồ (chart) cụ thể và hoạt động bằng cách đánh giá giá trị thực tế của một chỉ số (metric) so với các ngưỡng (threshold) do người dùng định nghĩa.

- Cách hoạt động cơ bản của một rule cảnh báo bao gồm các bước:
  - Xác định chart cần theo dõi.
  - Lấy một giá trị cụ thể từ chart đó (dùng calc, lookup,…).
  - So sánh giá trị đó với ngưỡng bằng biểu thức warn và crit.
  - Nếu điều kiện đúng, cảnh báo sẽ được kích hoạt.
  - Có thể thêm các điều kiện như delay, repeat, hoặc thông tin chi tiết qua summary, info.


<div align="center">
<img src="/images/alert.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

- Giải thích:
  - Cảnh báo này áp dụng cho biểu đồ snmp.device_uptime, tức là thời gian hoạt động (uptime) của thiết bị được giám sát qua SNMP.
  - Biểu thức calc: $now - $last_collected_t tính thời gian đã trôi qua kể từ lần thu thập cuối cùng.
  - Nếu thời gian này lớn hơn 6 lần chu kỳ cập nhật ($update_every), hệ thống kết luận là thiết bị không còn phản hồi nữa, kích hoạt cảnh báo ở mức crit.
  - Thêm delay: down 20s để hệ thống chỉ kích hoạt cảnh báo nếu trạng thái xấu kéo dài ít nhất 20 giây, tránh các cảnh báo "giả" do ngắt kết nối tạm thời.
  - repeat: 300s yêu cầu nếu cảnh báo vẫn tiếp diễn, thì cứ 5 phút cảnh báo sẽ lặp lại một lần.
  - Cảnh báo này sẽ gửi tới nhóm sysadmin.

<div align="center">
<img src="/images/alert1.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

- Giải thích:
  - Rule này cũng áp dụng lên biểu đồ uptime của thiết bị SNMP.
  - Sử dụng lookup: average -30s để tính trung bình giá trị uptime trong 30 giây gần nhất, đảm bảo ổn định trước khi cảnh báo.
  - Nếu uptime nhỏ hơn 300 giây (tức là mới bật chưa đầy 5 phút), cảnh báo ở mức warn được kích hoạt.
  - Gửi cảnh báo này về nhóm silent, tức là chỉ ghi log hoặc hiển thị trên dashboard, không gửi email vì việc thiết bị khởi động lại không nhất thiết nghiêm trọng.
  - summary và info cung cấp thông tin rõ ràng về nội dung cảnh báo.

Khi tắt thiết bị sẽ có cảnh báo sau : 

<div align="center">
<img src="/images/alert2.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

### 2. Cấu hình cảnh báo dịch vụ
Cấu hình alert apache cũng như vậy :

<div align="center">
<img src="/images/alert3.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

Cảnh báo sẽ hiển thị nêu apache không phản hồi : 


<div align="center">
<img src="/images/alert4.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

### 3. Cấu hình gửi Email

#### a. Cấu hình msmtp
Msmtp là một dịch vụ gửi mail nhẹ và dễ cấu hình — để đảm nhận việc gửi thư từ dòng lệnh. Việc cài đặt và cấu hình msmtp yêu cầu thiết lập tài khoản email gửi đi, SMTP server, cổng, mật khẩu ứng dụng (đối với Gmail hoặc các dịch vụ yêu cầu xác thực), v.v. Phần này có thể hỏi chatGPT để cài.

#### b. Cấu hình gửi mail trong Netdata
Netdata sử dụng file cấu hình health_alarm_notify.conf để bật hoặc tắt các kênh gửi cảnh báo (gồm Slack, Discord, Email, Telegram, v.v.). Trong trường hợp của chúng em, chỉ sử dụng Email (phiên bản miễn phí chỉ dùng được Email với Discord), nên cần bật các dòng sau trong file:

<div align="center">
<img src="/images/alert5.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

Gán người nhận email theo nhóm cảnh báo: 

<div align="center">
<img src="/images/alert6.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

Check mail để xem thử có cảnh báo không: 

<div align="center">
<img src="/images/alert7.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

<div align="center">
<img src="/images/alert8.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

## Tích hợp machine learning

- Mục tiêu cải tiến:
  - Nâng cao khả năng phát hiện sự cố bất thường trong thời gian thực.
  - Tự động hóa phân tích thay vì giám sát thủ công.
  - Giảm thời gian phản hồi trước các tình huống bất thường.

- Lựa chọn mô hình học máy, so sánh giữa các mô hình:
  - Z-score: đơn giản, nhanh
  - Isolation Forest: tốt cho dữ liệu bất thường nhỏ lẻ (được lựa chọn)
  - Prophet: mạnh với chu kỳ, trend

Lý do chọn Isolation Forest: phù hợp dữ liệu Bandwidth theo thời gian, có hỗ trợ ngưỡng bất thường (yhat_upper, yhat_lower).

### 1. Mô hình Isolation Forest

Mô hình Isolation Forest
- Isolation Forest (IF) là một thuật toán học máy không giám sát được thiết kế chuyên biệt để phát hiện điểm bất thường (anomaly detection) trong tập dữ liệu. Khác với các phương pháp dựa trên mật độ (density-based) hay khoảng cách (distance-based), Isolation Forest dựa trên ý tưởng rằng:
- Nguyên lý hoạt động
  - Mô hình xây dựng nhiều cây nhị phân (isolation trees) bằng cách ngẫu nhiên chọn một đặc trưng (feature) và một giá trị chia (split value).
  - Các điểm dữ liệu được chia tách liên tục đến khi mỗi điểm bị cô lập trong một nút lá.
  - Số bước trung bình cần để cô lập một điểm gọi là path length.
  - Điểm bất thường thường yêu cầu ít bước hơn để bị cô lập → path length ngắn hơn → anomaly score cao hơn.

- Ưu điểm
  - Hiệu quả, tốc độ cao: Thích hợp với tập dữ liệu lớn và đa chiều.
  - Không yêu cầu dữ liệu phải phân phối chuẩn (non-parametric).
  - Có thể mở rộng và áp dụng với nhiều loại dữ liệu khác nhau.

- Tham số quan trọng
  - contamination: Xác định tỷ lệ ước tính của điểm bất thường trong tập dữ liệu. Ví dụ: contamination=0.01 nghĩa là giả định khoảng 1% dữ liệu là bất thường.
  - n_estimators: Số lượng cây trong rừng.
  - max_samples: Số lượng mẫu dữ liệu dùng để xây dựng mỗi cây

- Ứng dụng trong dự án
Trong dự án này, Isolation Forest được sử dụng để:
  - Huấn luyện trên dữ liệu "bình thường" thu thập từ hệ thống (clean data).
  - Sau đó mô hình dự đoán điểm mới để phát hiện các bất thường (anomalies) như: tăng đột biến bandwidth.

- Pipeline học máy tích hợp
Thu thập dữ liệu: Netdata → API Prometheus(dùng để cào dữ liệu apiv1 từ trang web để lấy dữ liệu trực tiếp)→ Telegraf -> InfluxDB -> Python model ->GPT Azure -> Mail cảnh báo

- Tiền xử lý: lọc các metric cần thiết, chuẩn hóa thời gian

<div align="center">
<img src="/images/ml.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

- Sau đó huấn liện bằng dữ liệu đã được lọc, có nghĩa là chọn những dữ liệu bình thường để huấn liện, sau đó dùng mô hình dữ liệu bình thường này để detect các dữ liệu bất thường và gửi dữ liệu bất thường cho GPT Azure phân tích (Repo này về GPT free bởi azure trên github), Azure phân tích và render ra file .pdf để gửi về mail qua mailgun, mình dùng 2 server gửi mail khác nhau để tách riêng biệt.


<div align="center">
<img src="/images/ml2.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>

<div align="center">
<img src="/images/ml3.png" alt="Tổng quan kiến trúc hệ thống" width="1200"/>
</div>


Căn bản thì đây xong rồi nhưng phần tích hợp machine learning do đồ án mình làm từ hồi tháng 6 mà mình đã xóa repo đi nên việc coi kỹ các thư mục cộng với việc triển khai bị lở dở, các bạn có thể tự làm thử.





