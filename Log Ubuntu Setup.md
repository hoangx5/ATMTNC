
# MANUAL
### Lấy file trên USB vào UbuntuSV
#### ●  Cắm và phân vùng cho USB 
	lsblk

*"sdb/sdb1 là usb"*
#### ●  tạo thư mục
	sudo mkdir -p /mnt/usb
#### ●  mount
	sudo mount /dev/sdb1 /mnt/usb
#### ●  check
	ls /mnt/usb
****
### Cài package
	sudo dpkg -i [đường dẫn]
****
### Thiết lập mạng
#### Thiết lập Mạng tạm
**set ip FireWall/VPN/Router**

	sudo ip addr add 10.0.0.1/29 dev ens37
	
	sudo ip link set ens37 up

**set ip FireWall/Router**

	sudo ip addr add 10.0.0.2/29 dev ens37

	sudo ip link set ens37 up

#### Thiết lập NetPlan (vĩnh viễn)
	sudo nano /etc/netplan/00-installer-config.yaml

	sudo netplan apply
****
### Lệnh test
#### ●  Lệnh tra thay cho ping

	ping -c 5 10.0.0.2
	
**Lệnh test routing**

	ip route
	
# LOG
### 1. Cài các package cần thiết
*"Các package gồm: nano; ping; dhcp"*

#### Các bước chuẩn bị:
- Cắm USB vào máy thật
- Download các package từ nguồn chính thức
- Chép package vào USB: E:/UbuntuSV Setup/
- Rút USB ra cắm lại
- Ở VMware chọn kết nối vào 
- Thực hiện các bước mount USB
- Cắm và phân vùng cho USB

#### Thực hiện:
##### Trước khi thực hiện

	lsblk

*"sdb/sdb1 là usb"*


##### Tạo thư mục

	sudo mkdir -p /mnt/usb

##### Mount

	sudo mount /dev/sdb1 /mnt/usb

##### Check

	ls /mnt/usb

##### Cài trình soạn thảo offline

	sudo dpkg -i /mnt/usb/UbuntuSV/nano

##### Cài lệnh ping

	sudo dpkg -i /mnt/usb/UbuntuSV/iputils-ping

##### Cài dhcp

	sudo dpkg -i /mnt/usb/UbuntuSV/isc-dhcp

### 2. Setup Netplan UbuntuSV

#### Lệnh vào chỉnh sửa netplan

	sudo nano /etc/netplan/00-installer-config.yaml

#### Cấu hình netplan netplan

>FireWall/VPN
	
	ethernets:
	   ens33:
		 dhcp4: false
		 addresses:
		   - 192.168.145.1/24

	   ens37:
		 dhcp4: false
		 addresses:
		   - 10.0.0.1/29
		 routes:
		   - to: 192.168.150.0/24
			 via: 10.0.0.2
	 version: 2

>FireWall

	network:
	 ethernets:
	   ens33:
		 dhcp4: false
		 addresses:
		   - 192.168.150.1/24
	   ens37:
		 dhcp4: false
		 addresses:
		   - 10.0.0.2/29
		 routes:
		   - to: 192.168.145.0/24
			 via: 10.0.0.1
	 version: 2
	 
#### Lệnh thử

	sudo netplan generate

	sudo netplan try

#### Lệnh áp dụng

	sudo netplan apply
	
### 3. Test ping

>FireWall/VPN

	ping -c 4 10.0.0.2
	ping -c 4 192.168.150.1

>Firewall

	ping -c 4 10.0.0.2
	ping -c 4 192.168.145.1

### 4. Bật chức năng routing

	sudo nano /etc/sysctl.conf

#### Thêm vào sysctl.conf

	net.ipv4.ip_forward=1

#### Lệnh áp dụng

	sudo sysctl -p

#### Lệnh test xem đã bật routing chưa

	cat /proc/sys/net/ipv4/ip_forward

>kết quả mong đợi

	1

#### Lệnh test xem các routing hiện tại

	ip route

### 5.Cấu hình DHCP

#### Cấu hình interface DHCP

	sudo nano /etc/default/isc-dhcp-server

>Thêm vào isc-dhcp-server
	
	INTERFACESv4="ens33"
	
#### Cấu hình dhcpd.conf

	sudo nano /etc/dhcp/dhcpd.conf

#### Thêm vào dhcpd.conf

>FireWall/VPN

	authoritative;

	subnet 192.168.145.0 netmask 255.255.255.0 {
		range 192.168.145.10 192.168.145.200;

		option subnet-mask 255.255.255.0;
		option routers 192.168.145.1;

		option domain-name-servers 8.8.8.8;

		default-lease-time 600;
		max-lease-time 7200;
	}

>FireWall

	authoritative;

	subnet 192.168.150.0 netmask 255.255.255.0 {
		range 192.168.150.10 192.168.150.200;

		option subnet-mask 255.255.255.0;
		option routers 192.168.150.1;

		option domain-name-servers 8.8.8.8;

		default-lease-time 600;
		max-lease-time 7200;
	}

#### Kết quả sau khi cấu hình dhcpd.conf

>Kết quả FireWall/VPN

	DHCP range:
	192.168.145.10 → 192.168.145.200

	Gateway tự động:
	192.168.145.1
	
>Kết quả FireWall

	DHCP range:
	192.168.150.10 → 192.168.145.200

	Gateway tự động:
	192.168.150.1
	
#### Lệnh kiểm tra kết quả
	
	sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
	
#### Lệnh khởi động lại để áp dụng

	sudo systemctl restart isc-dhcp-server
	sudo systemctl enable isc-dhcp-server