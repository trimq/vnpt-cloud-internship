# Tìm hiểu VMWare, cấu hình IP và card mạng cho máy ảo Ubuntu server

## Mục lục
###[1. Add 2 card mạng trên máy Ubuntu Server và cấu hình](#add_nic)
###[2. Thiết lập IP tĩnh và IP động cho máy ảo (chỉnh sửa file và dùng câu lệnh)](#set_ip)

## Ghi chú
* VMWare trong bài lab là phiên bản VMware Workstation 12
* Hệ điều hành chủ là Ubuntu Desktop 14.04
* Trong bài lab có sử dụng ssh client có sẵn của Ubuntu

---
###[1. Add 2 card mạng trên máy Ubuntu Server và cấu hình](#add_nic)
- Hiện tại VMware sử dụng để lab đang có 3 NIC:  vmnet8 (NAT), vmnet0 (bridged), vmnet1 (host-only)   
- Tiến hành add thêm 2 NIC chế độ Host-only là vmnet2 và vmnet3 như sau:
	+ Add NIC vmnet2 chế độ host-only với địa chỉ: 172.31.0.0/24
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPX0VkSjc5WTRtYnM)
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPWjJYTkh2cC02Q2c)
	+ Add NIC vmnet3 cũng chế độ host-only với địa chỉ: 172.31.10.0/24
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPYU5meUVtTVJKNkk)
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPRlRiVUoxSW91am8)

- Tiến hành add thêm card mạng cho máy ảo Ubuntu Server như sau
 		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPeVpRMXU1N3FUdjQ)
 	+ Chọn Network Adapter rồi click Next
 		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPYUJxSkJMWHlwcDQ)
 	+ Chọn chế độ custom rồi chọn /dev/vmnet2 rồi click Finish
 		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPWFhJdlNxeVE5TVU)
 	+ Tiến hành add thêm 1 card chế độ host-only và chọn /dev/vmnet3
 		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPM1J2UzFuSnd4akU)
- Kiểm tra lại 3 card mạng của máy ảo Ubuntu
		
- Sau đó bật máy ảo lên và kiểm tra xem máy ảo đã nhận đầy đủ card mạng chưa. Mặc định ban đầu khi cài đặt, máy ảo thiết lập chỉ có 1 card mạng (eth0) chế độ Bridge nên khi kiểm tra chỉ thấy địa chỉ 192.168.1.42 tương ứng chế độ bridge:
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPNDJrd0ZFYjJSbDA)

	**Chú ý**
	Để kiểm tra các interface có thể sử dụng lệnh `ifconfig` hoặc `landscape-sysinfo` (mặc định khi cài ubuntu server có cài đặt gói landscape-common nên có thể sử dụng lệnh này)
- Để xin cấp IP cho hai card mạng mới (eth1 và eth2) trên máy ảo, sử dụng hai lệnh `dhclient eth1` và `dhclient eth2` . Tiếp đó, gõ lệnh `landscape-sysinfo` kiểm tra lại xem máy ảo đã được cấp IP cho hai card mới chưa: 	
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPQ1ZLRUFUYVVkeEU)

###[2. Thiết lập IP tĩnh và IP động cho máy ảo (chỉnh sửa file và dùng câu lệnh)](#set_ip)
####a. Cấu hình bằng cách chỉnh sửa file /etc/network/interfaces:
- Mặc định ban đầu khi cấu hình card mạng cho máy ảo, các máy ảo được cấp IP động nhờ dhcp server (ở chế độ bridge thì card eth0 chế độ bridge chung dhcp server với máy thật, còn chế độ host-only sẽ là 1 dhcp server ảo). Dùng vi mở file cấu hình các interfaces để kiểm tra: `sudo vi /etc/network/interfaces` (để edit file dùng vi, nhấn phím *i* để chuyển sang mode insert):
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPOHN5ZjdyZWRCNkE)
- Ta tiến hành cấu hình IP tĩnh cho card eth0 trong file *interfaces* như sau:
	```sh
		auto eth0
		iface eth0 inet static
		ipaddress 192.168.1.42/24
		gateway 192.168.1.1
	```
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPT1lRdWl4YmFFUUk)
- Nhấn ESC (chuyển sang mode thường), gõ `:wq` để lưu file và thoát. Sau đó gõ lệnh `sudo ifdown -a && sudo ifup -a` để khởi động lại toàn bộ các card mạng. Tiến hành ping thử ra internet kiểm tra thông mạng `ping -c 4 google.com`
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPY0dqTXp0NTVMSEE)

	**Chú ý:**
	Trong bài lab sử dụng ssh client có sẵn của hệ điều hành chủ (Ubuntu Desktop 14.04) để ssh vào máy ảo Ubuntu server để tiến hành ping (do việc ping trong máy ảo ra internet tạo ra rất nhiều bản tin duplicate nên dễ gây rối mắt và khó quan sát)
- Ping thử kết nối giữa máy host và máy ảo (máy ảo có 1 card chế độ bridge nên cùng mạng với máy thật): 
	+ Từ máy ảo (địa chỉ 192.168.1.42 ping ra máy host có địa chỉ 192.168.1.87):
		```sh
			ping -c 4 192.167.1.87
		```
	+ Ping từ máy host sang máy ảo:
		```sh
			ping -c 4 192.168.1.42
		```
	+ Kết quả
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPV2hiclVZN3l6UFE)
- Tiến hành cấu hình IP tĩnh tương tự cho 2 card host-only còn lại:
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPSE9yUkVRU2Y1akk)
- Khởi động lại toàn bộ các card mạng: `sudo ifdown -a && sudo ifup -a`

####b. Cấu hình IP tĩnh sử dụng lệnh:
- Để cấu hình IP tĩnh cho máy ảo, ngoài cách chỉnh sửa file /etc/network/interfaces, ta có thể sử dụng lệnh để cấu hình (cấu hình này sẽ bị mất khi khởi động lại máy). Trước hết cấu hình cho eth0:
	```sh
		ifconfig eth0 192.168.1.48 netmask 255.255.255.0
		route add default gw 192.168.1.1
	```
- Khởi động các card mạng: `sudo ifdown -a && sudo ifup -a` (hoặc chỉ riêng eth0: `sudo ifdown eth0 && sudo ifup eth0`). Tiến hành ping thử từ host vào máy ảo: `ping -c 4 192.168.1.48` và ping từ máy ảo ra host: `ping -c 4 192.168.1.87`. Kết quả:
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPV2hiclVZN3l6UFE)

- Sử dụng một máy ảo khác (tên mininet cũng chạy ubuntu server 14.04), có một card eth0 chế độ host-only vmnet2 (172.31.0.0), tức là nối chung 1 bridge ảo với máy Ubuntu-Internship đang xét. Giờ ta cũng tiến hành cấu hình IP tĩnh cho máy ảo này đổi địa chỉ từ 172.31.0.129 sang 172.31.0.155:
	```sh
		ifconfig eth0 172.31.0.155 netmask 255.255.255.0
		route add default gw 172.31.0.155
	```

- Ta tiến hành thử ping để xem kết nối giữa hai máy ảo ubuntu server. Do tiện đang ssh từ host vào máy ảo Ubuntu-Internship nên để trực quan ta ping luôn trên terminal của host (session ssh vào Ubuntu-Internship) vào máy mininet: `ping 172.31.0.155 -c 2` và ping từ máy mininet sang Ubuntu-Internship `ping -c 2 172.31.0.128`. Kết quả ping thành công:
		![alt text](https://drive.google.com/uc?id=0Bw96fRvq9ILPc0dzQl9PZ0M0ZEU)
	



