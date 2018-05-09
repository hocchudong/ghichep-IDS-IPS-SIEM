# Tổng quan

IDS/IPS là gì? IDS/IPS dùng để làm gì?

Nếu ví firewall là cái khóa cửa để bảo vệ hệ thống của bạn thì có thể coi IDS là một cái camera giúp theo dõi những gì đang vào/ra trong hệ thống.

Theo wikipedia: IDS (intrusion detection system - hệ thống phát hiện xâm nhập) là một thiết bị hoặc ứng dụng phần mềm giám sát mạng, hệ thống máy tính 
về những hoạt động ác ý hoặc các vi phạm chính sách. Bất kỳ hoạt động hoặc vi phạm nào được phát hiện thường báo cáo cho quản trị viên hoặc 
thu thập tập trung bằng hệ thống SIEM (Security information and event management - hệ thống bảo mật và quản lý sự kiện). Một hệ thống SIEM kết 
hợp các kết quả đầu ra từ nhiều nguồn và sử dụng các kỹ thuật lọc báo động để phân biệt hoạt động ác ý từ các báo động sai lầm.

IPS (Intrusion Prevention System - hệ thống ngăn chặn xâm nhập) có các chức năng của một IDS, ngoài ra bổ sung thêm khả năng ngăn ngừa 
các hoạt động xâm nhập không mong muốn.

![IDS-IPS-graphic-e1507560087608](/Images/IDS-IPS-graphic-e1507560087608.jpg)

# Phân loại IDS theo phạm vi giám sát

Dựa vào phạm vi giám sát, IDS được chia thành 02 loại

## Network-based IDS (NIDS):

Là những IDS giám sát trên toàn bộ mạng. Nguồn thông tin chủ yếu của NIDS là các gói dữ liệu đang lưu thông trên mạng. NIDS thường 
được lắp đặt tại ngõ vào của mạng (Inline), có thể đứng trước hoặc  sau firewall. Như hình dưới:

![1_0](/Images/1_0.png)

## Host-based IDS (HIDS)

Là những IDS giám sát hoạt động của từng máy tính riêng biệt. Do vậy, nguồn thông tin chủ yếu của HIDS ngoài lưu lượng đến và đi trên máy chủ còn có hệ 
thống dữ liệu system log và system audit.

![2](/Images/2.png)

# Tham khảo 

- [https://vi.wikipedia.org/wiki/H%E1%BB%87_th%E1%BB%91ng_ph%C3%A1t_hi%E1%BB%87n_x%C3%A2m_nh%E1%BA%ADp](https://vi.wikipedia.org/wiki/H%E1%BB%87_th%E1%BB%91ng_ph%C3%A1t_hi%E1%BB%87n_x%C3%A2m_nh%E1%BA%ADp)
- [https://www.rokasecurity.com/ids-vs-ips/](https://www.rokasecurity.com/ids-vs-ips/)
- [http://www.ipmac.vn/technology-corner/bao-mat-he-thong-voi-he-thong-idsips-phan-1](http://www.ipmac.vn/technology-corner/bao-mat-he-thong-voi-he-thong-idsips-phan-1)