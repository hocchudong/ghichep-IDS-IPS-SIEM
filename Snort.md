# Snort là gì?

Snort là một kiểu IDS, thực hiện giám sát các gói tin ra vào hệ thống.

- Snort là một mã nguồn mở miễn phí với nhiều tính năng trong việc bảo vệ hệ thống bên trong, phát hiện sự tấn công từ bên ngoài vào hệ thống.
- Snort được viết bởi Martin Roesch vào năm 1998. Hiện tại, Snort được phát triển bởi Sourcefire, nơi mà Roesch đang là người sáng lập và CTO, 
và được sở hữu bởi Cisco từ năm 2013.

# Mô hình kiến trúc của Snort

![kien-truc-snort](../Images/kien-truc-snort.jpg)

Trong mô hình kiến trúc trên, hệ thống Snort được chia thành 4 phần:

- Module Decoder: Xử lý giải mã các gói tin
- Module Preprocessors: Tiền xử lý
- Module Detection Engine: Phát hiện
- Module Logging and Alerting System: Lưu log và cảnh báo

## Module Decoder

Snort sử dụng thư viện pcap để bắt mọi gói tin trên mạng lưu thông qua hệ thống. Gói tin sau khi được giải mã sẽ đưa vào Module tiền xử lý.

![the-ipv6-snort-plugin-at-deepsec-2014-20-638](../Images/the-ipv6-snort-plugin-at-deepsec-2014-20-638.jpg)

## Module Preprocessors

Module này rất quan trọng đối với bất kỳ hệ thống nào để có thể chuẩn bị gói dữ liệu đưa vào cho Module phát hiện phân tích. 03 nhiệm vụ chính

- Kết hợp lại các gói tin: Khi một dữ liệu lớn được gửi đi, thông tin sẽ không đóng gói toàn bộ vào một gói tin mà thực hiện phân mảnh, 
chia thành nhiều gói tin rồi mới gửi đi. Khi Snort nhận được các gói tin này, nó phải thực hiện kết nối lại để có gói tin ban đầu. 
Module tiền xử lý giúp Snort có thể hiểu được các phiên làm việc khác nhau.

- Giải mã và chuẩn hóa giao thức (decode/normalize): cCông việc phát hiện xâm nhập dựa trên dấu hiệu nhận dạng nhiều khi thất bại 
khi kiểm tra các giao thức có dữ liệu có thể được biểu diễn dưới nhiều dạng khác nhau. Ví dụ: một Web server có thể nhận nhiều dạng URL: 
URL viết dưới dạng hexa/unicode hay URL chấp nhận dấu / hay \. Nếu Snort chỉ thực hiện đơn thuần việc so sánh dữ liệu với dấu hiệu 
nhận dạng sẽ xảy ra tình trạng bỏ sót hành vi xâm nhập. Do vậy, 1 số Module tiền xử lý của Snort phải có nhiệm vụ giải mã và chỉnh sửa, 
sắp xếp lại các thông tin đầu vào.

- Phát hiện các xâm nhập bất thường (nonrule/anormal):các plugin dạng này thường để xử lý với các xâm nhập không thể hoặc rất khó 
phát hiện bằng các luật thông thường. Phiển bản hiện tại của Snort có đi kèm 2 plugin giúp phát hiện xâm nhập bất thường đó là portscan 
và bo (backoffice). Portscan dùng để đưa ra cảnh báo khi kẻ tấn công thực hiện quét cổng để tìm lỗ hổng. Bo dùng để đưa ra cảnh báo khi 
hệ thống nhiễm trojan backoffice.

## Module Detect Engine

Đây là module quan trọng nhất của Snort.

# Tham khảo 

- [http://baomatmang.net/snort-la-gi-cau-truc-cua-snort/](http://baomatmang.net/snort-la-gi-cau-truc-cua-snort/)
- [https://www.rokasecurity.com/ids-vs-ips/](https://www.rokasecurity.com/ids-vs-ips/)
- [https://viblo.asia/p/network-tim-hieu-co-che-cach-hoat-dong-cua-ids-phan-2-pDljMbe5RVZn](https://viblo.asia/p/network-tim-hieu-co-che-cach-hoat-dong-cua-ids-phan-2-pDljMbe5RVZn)