# Mô hình

Sử dụng Snort ở chế độ IDS thì ta sẽ cài đặt Snort trên một máy chủ trong hệ thống (Outline), khi đó, ta cấu hình mirror port để thiết bị SW đẩy dữ liệu 
tới Snort.

**Lưu ý**: Dữ liệu hệ thống của bạn lớn thì phải thiết kế băng thông cho máy chủ Snort và cấu hình máy chủ cũng phải đủ lớn

![4_snort-ips-4-638_525af_0](/Images/4_snort-ips-4-638_525af_0.jpg)

Nếu bạn muốn sử dụng Snort ở chế độ IPS thì phải thiết lập cho phép Snort đứng trước hoặc sau firewall (Inline). Việc chạy Snort như IPS thì các 
rule sẽ có thêm action: drop, reject, sdrop

![maxresdefault](/Images/maxresdefault.jpg)

![ids-ips-660x330](/Images/ids-ips-660x330.jpg)

# Cài đặt

Tôi thực hiện cài đặt Snort có chức năng là một NIDS outline trong hệ thống theo guilde hướng dẫn từ trang chủ của Snort.

Các package sẽ được cài đặt là:
- Snort: đọc, xử lý phân tích theo các rule được thiết lập.
- Barnyard2: Phần mềm lấy output của Snort và ghi vào CSLD SQL
- PulledPork: Tự động tải các Snort rule miễn phí mới nhất
- BASE: một giao diện đồ họa nền web cho việc xem các Snort event.

## Chuẩn bị

Bạn chuẩn bị một máy chủ cài đặt hệ điều hành Ubuntu server 16.04 x64. Thực hiện update hệ thống
```sh
apt update && apt dist-upgrade -y
```

Một số network card có tính năng gọi là "Large Receive Offload" (lro) và "Generic Receive Offload" (gro). khi kích hoạt tính năng này, 
network card thực hiện lắp ráp lại packet trước khi chúng được xử lý bởi kernel. mặc định, Snort sẽ xóa các packet lớn hơn default snaplen là 
1518 byte. Thêm vào đó, LRO, GRO có thể là nguyên nhân của vấn đề Stream5 target-based reassembly. Vì thế, nên tắt LRO và GRO.

Để tắt LRO và GRO ta sử dụng lệnh ethtool vào file cấu hình interface `/etc/network/interfaces`. Thêm 02 dòng sau vào mỗi interface
```sh
post-up ethtool -K eth0 gro off
post-up ethtool -K eth0 lro off
```

Thay thế đúng tên interface của máy bạn và thực hiện khởi động lại interface
```sh
ifconfig eth0 down && ifconfig eth0 up
```

Kiểm tra lại thông số
```sh
ethtool -K eth0 |grep receive-offload
```

Thực hiện cài đặt trước các gói mà Snort cần để chạy:
- pcap
- PCRE
- Libdnet
- DAQ

```sh
apt install -y build-essential
apt install -y libpcap-dev libpcre3-dev libdumbnet-dev
apt install -y bison flex
```

Tạo một thư mục chứa toàn bộ source code cài đặt và cài đặt DAQ
```sh
mkdir -p ~/snort_src
cd ~/snort_src
wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz
tar -xzvf daq-2.0.6.tar.gz
cd daq-2.0.6
./configure
make
make install
```

## Cài đặt Snort

Cài đặt thêm một số lib cho Snort
```sh
apt install zlib1g-dev liblzma-dev openssl libssl-dev libnghttp2-dev
```

Cài đặt Snort
```sh
cd ~/snort_src
wget https://www.snort.org/downloads/snort/snort-2.9.11.1.tar.gz
tar -xzvf snort-2.9.11.1.tar.gz
cd snort-2.9.11.1
./configure --enable-sourcefire
make
make install
```

Chạy lệnh sau để cập nhật thư viện chia sẻ
```sh
ldconfig
```

Đưa liên kết các thư viện của Snort vào /usr/sbin
```sh
ln -s /usr/local/bin/snort /usr/sbin/snort
```

Kiểm tra lại version của Snort sau khi cài xong
```sh
snort -V
```

## Cấu hình Snort chạy NIDS mode



# Tham khảo