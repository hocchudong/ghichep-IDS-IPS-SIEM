# Mô hình

Sử dụng Snort ở chế độ IDS thì ta sẽ cài đặt Snort trên một máy chủ trong hệ thống (Outline), khi đó, ta cấu hình mirror port để thiết bị SW đẩy dữ liệu 
tới Snort.

*Lưu ý*: Dữ liệu hệ thống của bạn lớn thì phải thiết kế băng thông cho máy chủ Snort và cấu hình máy chủ cũng phải đủ lớn

![4_snort-ips-4-638_525af_0](/Images/4_snort-ips-4-638_525af_0.jpg)

Nếu bạn muốn sử dụng Snort ở chế độ IPS thì phải thiết lập cho phép Snort đứng trước hoặc sau firewall (Inline). Việc chạy Snort như IPS thì các 
rule sẽ có thêm action: drop, reject, sdrop

![maxresdefault](/Images/maxresdefault.jpg)

# Cài đặt

# Tham khảo