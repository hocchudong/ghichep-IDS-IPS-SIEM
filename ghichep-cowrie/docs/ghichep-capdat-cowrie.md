# Tài liệu ghi chép về Cowrie

Tài liệu về các bước cài đặt Cowrie

## 1. Cài đặt Cowrie trên CentOS Server 7.4 - 64 bit

- Tham khảo: 

  - `(1)`: https://qiita.com/pypypyo14/items/f399366b34b8dfcb7aa1
	
	- `(2)`: https://leifdreizler.com/2016/Installing-Cowrie/

### 1.1 Bước chuẩn bị

#### 1.1.1 Các điều kiện
- Máy chủ cài OS CentOS Server 7.4 - 64 bit trắng (vừa được cài đặt)
- Có kết nối ra internet để tải gói.
- Cần tắt selinux và mở firewalld để xử lý forward port và mở các rule cần thiết.


### 1.1.2. Mô hình & IP Planning


### 1.2. Các bước cài đặt cơ bản


- Đặt hostname

	```sh
	hostnamectl set-hostname srv01cowrie
	```

- Đặt IP cho các NICs (cần ít nhất 1 NIC có kết nối internet)

	```sh
	echo "Setup IP  ens32"
	nmcli con modify ens32 ipv4.addresses 192.168.52.50/24
	nmcli con modify ens32 ipv4.gateway 192.168.52.1
	nmcli con modify ens32 ipv4.dns 8.8.8.8
	nmcli con modify ens32 ipv4.method manual
	nmcli con mod ens32 connection.autoconnect yes

	echo "Setup IP  ens33"
	nmcli con modify ens33 ipv4.addresses 10.10.10.50/24
	nmcli con modify ens33 ipv4.method manual
	nmcli con mod ens33 connection.autoconnect yes
	```

- Tắt selinux (Vẫn có cách mở selinux nhưng sẽ hướng dẫn sau :) )

	```sh
	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
	```

- Update OS và khởi động lại trước khi cài đặt.

	```sh
	yum -y update
	```

### 1.3. Bước cài đặt chính

#### 1.3.1. Cấu hình các rule của firewall 

- Kiểm tra firewalld xem có hoạt động hay không và các rule đang mở là gì

	```sh
	systemctl status firewalld
	```	

	```sh
	sudo firewall-cmd --list-services --zone=public --permanent
	```
	
	- Kết quả của các lệnh trên: http://prntscr.com/jkkdot

- Ý tưởng của cowrie là: 
  - Thay đổi port ssh mặc định (port 22) sang port mới .
  - Cài đặt cowrie và sử dụng port 22 cho cowrie.
  - Foward port 22 sang port 2222 để cowrie sử dụng và đánh lạc hướng attacker, khi attacker sử thử ssh vào port 22 thì cowrie vẫn phản hồi bình thường và không hề biết đó là do cowrie đang giả lập ssh. Thậm chí, attacker có thể ssh thành công nhưng không ảnh hưởng tới hệ thống thật (ssh server thực sự hay nói cách khác là máy chủ)
	
- Thay đổi port ssh mặc định là 22 sang port mới là 2255 (có thể chỉnh theo ý bạn nhưng cần khai báo đúng trong các bước sau).

	```sh
	sed -i 's/#Port 22/Port 2255/g' /etc/ssh/sshd_config
	```
	
- Sao chép file cấu hình ssh trong firewalld

	```sh
	sudo cp -ip /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/ssh.xml
	```

- Sửa dòng <port protocol="tcp" port="22"/> thành dòng <port protocol="tcp" port="2255"/> để cho phép ssh vào port 2255 ở file `/etc/firewalld/services/ssh.xml` bằng lệnh dưới.

	```sh
	sed -i 's/port="22"/port="2255"/g' /etc/firewalld/services/ssh.xml
	```
	
- Mở mở rule 22 cho phép bên ngoài có thể sử dụng port 22 (lúc này port 22 đã được forward (bước dưới) sang port mà cowrie xử lý - ví dụ này là port 2222)

	```sh
	sudo firewall-cmd --add-port=22/tcp --zone=public --permanent
	```

- Mở port 2222 (port mà cowrie sử dụng)

	```sh
	sudo firewall-cmd --add-port=2222/tcp --zone=public --permanent
	```

- Thực hiện foward port 22 sang port 2222

	```sh
	sudo firewall-cmd --permanent --zone=public --add-forward-port=port=22:proto=tcp:toport=2222
	```

- Load lại các rule vừa cấu hình cho firewalld ở trên

	```sh
	sudo firewall-cmd --reload
	```

- Kiểm tra lại các rule đã khai báo trong firewalld

```sh
sudo firewall-cmd --list-all --zone=public --permanent
```

  - Kết quả: http://prntscr.com/jkkjah
	
	
- Khởi động lại SSH
	
	```sh
	systemctl restart sshd
	```

- Nếu khởi động lại ssh báo lỗi thì thực hiện khởi động lại OS và đăng nhập ssh với port 2255.
	
	
#### 1.3.2. Thực hiện cài đặt cowrie

- Tạo user để cài cowrie bởi vì cowrie không dùng tài khoản root để đảm bảo an toàn.

	```sh
	su -

	useradd Cowrie
	```

- Đặt pass cho tài khoản `Cowrie`, trong màn hình nhắc lệnh hãy nhập mật khẩu của bạn chọn.

	```sh
	passwd Cowrie
	```

- Cài đặt các gói phần mềm cần thiết

	```sh
	yum groupinstall -y "Development Tools"

	yum install -y python-devel python-setuptools python-virtualenv

	easy_install pip
	```

- Chuyển sang user `Cowrie` và tải bộ cài của Cowrie

	```sh
	su - Cowrie

	git clone http://github.com/micheloosterhof/cowrie

	cd cowrie/

	cp -ip cowrie.cfg.dist cowrie.cfg
	```


- Thực hiện cài cowrie trong môi trường virtualenv
	
	```sh
	virtualenv cowrie-env

	source cowrie-env/bin/activate

	pip install --upgrade pip

	pip install --upgrade -r requirements.txt
	```

- Thoát ra khỏi môi trường virtualenv và thực hiện start cowrie

	```sh
	deactivate
	```

- Khởi động cowrie bằng một trong hai tùy chọn dưới

	```sh
	./bin/cowrie start
	
	(Nếu đứng tại thư mục chứa mã nguồn của cowie)
	```
	
	hoặc
	
	```sh
	/home/Cowrie/cowrie/bin/cowrie start
	```
	
- Nếu thành công kết quả sẽ báo như sau: 

	```sh
	[Cowrie@srv01cowrie cowrie]$ /home/Cowrie/cowrie/bin/cowrie start
	Using default Python virtual environment "/home/Cowrie/cowrie/cowrie-env"
	Starting cowrie: [twistd   --umask 0022 --pidfile var/run/cowrie.pid --logger cowrie.python.logfile.logger cowrie ]...
	```


#### 1.3.3. Thực hiện kiểm tra

- Đứng trên máy cowrie vừa cài, thực hiện quan sát theo thời gian thực file log.


	```sh
	tailf  /home/Cowrie/cowrie/log/cowrie.log
	```

- Đứng trên một máy khác hoặc dùng chương trình ssh và thử ssh với port 22, sau đó quan sát log theo bước trên.

 - Thử đăng nhập sai: http://prntscr.com/jkkuko
 
 - Thử đăng nhập đúng: http://prntscr.com/jkkt6t . Nhưng lưu ý, lúc này attacker đăng nhập đúng với một tài khoản root và nằm trên host khác (host do cowrie giả lập ;) ). Và lúc này attacker sẽ thực hiện các thao tác lệnh thì cowie sẽ lưu lại hết: http://prntscr.com/jkktwq
 
 




























