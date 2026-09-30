

* **Cắm và phân vùng cho USB**

lsblk

*//sdb/sdb1 là usb*



*//tạo thư mục*

sudo mkdir -p /mnt/usb



*//mount*

sudo mount /dev/sdb1 /mnt/usb



*//check*

ls /mnt/usb



*//cài trình soạn thảo offline*

sudo dpkg -i /mnt/usb/nano\_8.0-1\_amd64.deb



* **Cài trình soạn thảo văn bản online**

sudo apt update

sudo apt install nano



* **Lệnh kiểm tra thay cho ping**

ip route get 10.0.0.2



* **Thiết lập NetPlan**

sudo nano /etc/netplan/00-installer-config.yaml

sudo netplan apply



* **Thiết lập mạng tạm**

***//set ip FireWall/VPN/Router*** 

sudo ip addr add 10.0.0.1/29 dev ens37

sudo ip link set ens37 up



***//set ip FireWall/Router***

sudo ip addr add 10.0.0.2/29 dev ens37

sudo ip link set ens37 up



* **Log**
1. Cài trình soạn thảo văn bản nano

\-Cắm USB vào máy thật

\-Tải package vào USB

\-Cắm USB vào UbuntuSV

\-Thực hiện các bước mount USB



//Cắm và phân vùng cho USB



lsblk

*//sdb/sdb1 là usb*



*//tạo thư mục*

sudo mkdir -p /mnt/usb



*//mount*

sudo mount /dev/sdb1 /mnt/usb



*//check*

ls /mnt/usb



//cài trình soạn thảo offline

sudo dpkg -i /mnt/usb/nano



//cài lệnh ping

sudo dpkg -i /mnt/usb/iputils-ping





2\. Setup Netplan UbuntuSV

sudo nano /etc/netplan/00-installer-config.yaml



sudo netplan generate

sudo netplan try



sudo netplan apply



//FireWall/VPN

network:

&#x20; ethernets:

&#x20;   ens33:

&#x20;     dhcp4: false

&#x20;     addresses:

&#x20;       - 192.168.145.1/25



&#x20;   ens37:

&#x20;     dhcp4: false

&#x20;     addresses:

&#x20;       - 10.0.0.1/29



&#x20;     routes:

&#x20;       - to: 192.168.145.128/25

&#x20;         via: 10.0.0.2



&#x20; version: 2



//FireWall

network:

&#x20; ethernets:

&#x20;   ens33:

&#x20;     dhcp4: false

&#x20;     addresses:

&#x20;       - 192.168.145.129/25



&#x20;   ens37:

&#x20;     dhcp4: false

&#x20;     addresses:

&#x20;       - 10.0.0.2/29



&#x20;     routes:

&#x20;       - to: 192.168.145.0/25

&#x20;         via: 10.0.0.1

&#x20; version: 2



3\. Test ping

//FireWall/VPN

ip route get 10.0.0.2



//Firewall

ip route get 10.0.0.1



4\. Bật chức năng routing

sudo nano /etc/sysctl.conf



//thêm

net.ipv4.ip\_forward=1



//áp dụng

sudo sysctl -p



//test xem đã bật routing chưa

cat /proc/sys/net/ipv4/ip\_forward

=> 1



//test xem các routing hiện tại

ip route

