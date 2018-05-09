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

Nếu chưa có ethtool thì cài đặt thêm
```sh
apt install ethtool -y
```

**NOTE** (đã test cấu hình lro, gro và thấy lỗi nhưng cứ docs lại đây, bạn có thể ko cần cấu hình đoạn sau khi lab, còn khi chạy product thì nên nghiên cứu kỹ):
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

Bắt đầu thực hiện cài đặt trước các gói mà Snort cần để chạy:
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

Khi cài đặt có một số `warning` nhưng việc cài đặt vẫn diễn ra bình thường, ko vấn đề gì cả.

## Cài đặt Snort

Cài đặt thêm một số lib cho Snort
```sh
apt install -y zlib1g-dev liblzma-dev openssl libssl-dev libnghttp2-dev
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

Sử dụng một user khác root để chạy Snort
```sh
# tạo user và group snort
groupadd snort
useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort

# Tạo các thư mục Snort
mkdir /etc/snort
mkdir /etc/snort/rules
mkdir /etc/snort/rules/iplists
mkdir /etc/snort/preproc_rules
mkdir /etc/snort/lib/snort_dynamicrules
mkdir /etc/snort/so_rules

# tạo file để lưu trữ rule và danh sách IP
touch /etc/snort/rules/iplists/black_list.rules
touch /etc/snort/rules/iplists/white_list.rules
touch /etc/snort/rules/local.rules
touch /etc/snort/sid-msg.map

# Tạo thư mục lưu trữ log
mkdir /var/log/snort
mkdir /var/log/snort/archived_logs

# Phân quyền
chmod -R 5775 /etc/snort
chmod -R 5775 /var/log/snort
chmod -R 5775 /var/log/snort/archived_logs
chmod -R 5775 /etc/snort/so_rules
chmod -R 5775 /usr/local/lib/snort_dynamicrules

# Chuyển quyền
chown -R snort:snort /etc/snort
chown -R snort:snort /var/log/snort
chown -R snort:snort /usr/local/lib/snort_dynamicrules
```

Snort cần vài file cấu hình, ta sử dụng các file có sẵn trong source
```sh
cd ~/snort_src/snort-2.9.11.1/etc/
cp *.conf* /etc/snort
cp *.map /etc/snort
cp *.dtd /etc/snort

cd ~/snort_src/snort-2.9.11.1/src/dynamic_preprocessors/build/usr/local/lib/snort_dynamicpreprocessor/
cp * /usr/local/lib/snort_dynamicpreprocessor
```

Tới đây, cơ bản Snort đã được cấu hình xong, ta chỉnh lại một số thông số trong file `/etc/snort/snort.conf` trước khi chạy
```sh
sed -i "s/include \$RULE\_PATH/#include \$RULE\_PATH/ " /etc/snort/snort.conf
```

Sửa thủ công một số thông số khác bằng cách sử dụng lệnh `vi /etc/snort/snort.conf`
```sh
# LINE 45 thay bằng internal network. Nếu muốn dải mạng external thì nên xài !$HOME_NET
ipvar HOME_NET 10.0.0.0/24

# LINE 104
var RULE_PATH /etc/snort/rules
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules

var WHITE_LIST_PATH /etc/snort/rules/iplists
var BLACK_LIST_PATH /etc/snort/rules/iplists

# Sử dụng file local.rules thì tại dòng 546 ta bỏ dấu #
include $RULE_PATH/local.rules
```

Sau khi cấu hình xong, ta verify lại file một lần bằng lệnh
```sh
sudo snort -T -i eth0 -c /etc/snort/snort.conf
```

Viết một rule đơn giản để test Snort Detection. ta mở file `/etc/snort/rules/local.rules` và thêm dòng sau
```sh
alert icmp any any -> $HOME_NET any (msg:"ICMP test detected"; GID:1; sid:10000001; rev:001; classtype:icmp-event;)
```

Tiếp tục thêm dòng cấu hình sau vào fule /etc/snort/sid-msg.map để bật trigger cảnh báo
```sh
#v2
1 || 10000001 || 001 || icmp-event || 0 || ICMP Test detected || url,tools.ietf.org/html/rfc792
```

Chạy lệnh sau để chắc chắn đã cấu hình đúng
```sh
snort -T -c /etc/snort/snort.conf -i eth0
``` 

Một số tham số trong lệnh chạy
```sh
-A console
-q
-u snort
-g snort
-c /etc/snort/snort.conf
-i eth0
```

Thực hiện chạy lệnh sau để test
```sh
/usr/local/bin/snort -A console -q -u snort -g snort -c /etc/snort/snort.conf -i eth0
```

Tiến hành ping tới IP của eth0 sẽ có log alert được xuất hiện trên màn hình console. Nếu bạn ctrl-c để dừng Snort, các thông tin sẽ lưu vào trong 
thư mục /var/log/snort với tên snort.log.nnnnnn (số có thể khác).

# Tham khảo