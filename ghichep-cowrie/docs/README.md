## Tài liệu ghi chép về Cowrie

### Mô tả tóm lược về Cowrie

- Honeypot là giải pháp được xây dựng với ý tưởng để đánh lạc hướng hoặc dựng lên các hệ thống bẫy các attacker xâm nhập vào hệ thống. Từ việc bẫy hoặc đánh lạc hướng này, ta sẽ thu thập các hành vi và cách thức tấn công của attacker và từ đó có phương pháp phòng chống cho hệ thống thật. Ngoài ra, các hệ thống Honeypot cũng phục vụ việc điều tra các cuộc tấn công hoặc cách thức tấn công trên Internet. 
- Có rất nhiều giải pháp nguồn mở cho ý tưởng Honeypot này, Cowrie là một trong số sản phẩm đó.
- Cowrie làm một ứng dụng theo nhóm  sản phẩm Honeypot và tiếp cận theo hướng Honeypot cho SSH, hay nói cách khác nó dựa vào SSH để triển khai Honeypot.
- Có thể triển khai Cowrie trên từng máy linux (Standalone node) hoặc triển khai theo kiểu client-server.
- Cowrie được phát triển bởi:  Michel Oosterhof
- Mã nguồn của Cowrie được công bố tại: https://github.com/micheloosterhof/cowrie

Hình dưới sẽ minh họa ý tưởng của Cowrie

![Minh họa ý tưởng Cowrie](../images/image1.png)

Giải thích tóm lược về ý tưởng của Cowrie như sau:

- Thực hiện thay đổi port mặc định của SSH sang port mới, trong ảnh trên là từ `22` sang `22222`.
- Người dùng sử dụng port `22222` để SSH, đây mới là port SSH thực sự sau khi thay đổi.
- Cấu hình các rule trên iptables hoặc firewalld hoặc ufw để mở các port và protocol cần thiết.
- Cấu hình forward port `22` sang port `2222` để Cowrie nhận được các traffic khi thực hiện truy cập vào port 22. Attacker sẽ dùng port này để thử ssh/
- Lý thú của Cowrie là nó đáp trả được các request với port 22 (port mặc định SSH) và trả về cho attacker. Điểm mấu chốt của Cowrie nói riêng và các hệ thống Honeypot nói chung là ở đây. Nó hoàn toàn đánh lừa được các attacker, làm cho chúng tưởng nhầm đây là hệ thống thât.
- Ngoài việc bắt các log về SSH thông thường, Cowrie còn bắt được được các log khi sử dụng lệnh hoặc thao tác ... khi attacker thực hiện.


### Danh mục các tài liệu và ghi chép về Cowrie

- Mô tả tóm lược về Cowrie

- Mô tả về kiến trúc của Cowrie

- [Hướng dẫn cài đặt Cowrie trong môi trường LAB](./ghichep-capdat-cowrie.md)

- Các lưu ý về Cowrie





