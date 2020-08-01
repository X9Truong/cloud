### Tạo Gateway Server trên Centos 7

Tạo Internet Gateway Server trên Centos 7

Hãy tưởng tượng, trong văn phòng của bạn có 2 subnetwork, một cho mạng người dùng, một dành cho các máy chủ. Cả 2 subnet này đều muốn sử dụng Internet trong khi ta chỉ có 01 line Internet của ISP kéo vào với con modem 1 cổng duy nhất. Switch L3 thì chưa mua được, mà ta lại thừa 1 con Workstation có những 4 cổng mạng. Vậy đây là lúc ta nên tiết kiệm để ghi điểm với sếp. Đó chính là việc dựng 1 con Gateway Server cho 02 cái mạng trên để thông ra Internet. Tất nhiên kỹ thuật được sử dụng ở đây chỉ có thể là NAT (thực ra ở đâu dùng Internet chả vậy).

Mô hình mạng như sau:

<img src="/Note/img/9.jpg">

Ở Gateway server ta sẽ cần 03 cổng mạng, một cho subnet người dùng, một cho subnet máy chủ, và một để kết nối vào con router/modem mà ISP họ cấp cho mình. Các traffic sẽ đi từ mạng private (192.168.0.0 và 172.16.0.0) qua Gateway server và ra mạng Internet. Ta gọi tương ứng là mạng LAN và WAN

Trước tiên ta setup cho WAN interface tại cổng mạng thứ nhât là eth0. Thông thường thì sau khi nối dây từ cổng mạng eth0 này vào thiết bị modem của ISP để kết nối ra Internet, ta chỉ cần để chế độ DHCP và chờ thiết bị router/modem của ISP cấp phát cho IP động của Internet. là xong. Việc của ta là khai báo đúng các thông tin thuê bao Internet trong router/modem, không hơn không kém.

Các bước cần làm như sau:

` vi /etc/sysconfig/network-scripts/ifcfg-eth0`

Trong đó chú ý như sau là được:

```
TYPE="Ethernet"

BOOTPROTO="dhcp" 
 
DEVICE="eth0" 

ONBOOT="yes"
```

Tiếp theo ta chuyển sang cấu hình ở cổng mạng thứ 2, chẳng hạn ở cổng mạng kết nối vào local network 192.168.0.0/24 với một địa chỉ IP tĩnh như sau:

`vi /etc/sysconfig/network-scripts/ifcfg-eth1` 

Trong đó quan trọng nhất là:


```
BOOTPROTO="static"

IPADDR="192.168.0.1"

NETMASK="255.255.255.0"

DEVICE=eth1

ONBOOT = yes 
```

Vậy là đã xong, trên Linux Gateway Server này đã có 1 cánh cổng để ra mạng Internet và một cánh cổng – chính là địa chỉ 192.168.0.1 để kết nối vào LAN 192.168.0.0/24. Đương nhiên đối với những máy tính trong LAN 192.168.0.0/24 này thì 192.168.0.1 chính là gateway của chúng. Restart network cho Gateway server để active các cấu hình mới.

`systemctl restart network.service` 

Ta sẽ lấy 1 máy client trong LAN này và setup mạng cho nó như sau (tất nhiên phải nối nó qua 1 con Switch đã config network 192.168.0.0/24 và con Switch này có uplink cắm vào cổng mạng eth1 trên Gateway server như hình vẽ)

```
IPADDR="192.168.0.10"

NETMASK="255.255.255.0"

GATEWAY= "192.168.0.1"
```

Rồi cũng restart lại network để máy client này nhận IP 192.168.0.10

Quay trở lại với Gateway Server, ta cần cấu hình IP forwarding như sau để thực hiện tính năng NAT:

```
vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
sudo sysctl –w
```

Sau đó cấu hình NAT sử dụng iptables, trong đó enable IP masquerading thông qua các lệnh firewalld-cmd như sau

Trước tiên tạo LAN network 192.168.0.0/24 ở trên như một trusted zone.

`firewall-cmd --permanent --zone=trusted --add-source=192.168.0.0/24`

Sau đó đưa LAN này vào zone internal

`firewall-cmd --change-interface=eth1 --zone=internal --permanent`

Đưa WAN ứng với interface eth0 vào zone external

`firewall-cmd --change-interface=eth0 --zone=external --permanent`

Cấu hình masquarade cho external WAN

`firewall-cmd --zone=external --add-masquerade --permanent`

Thêm cấu hình DNS

`firewall-cmd --zone=internal --add-service=dns –-permanent`

Cuối cùng là quan trọng nhất là cấu hình NAT để pass traffic từ LAN sang WAN:

`firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s 192.168.0.0/24`

Và không quên reload lại firewalld:

`firewall-cmd –reload`

Lúc này quay trở lại máy client 192.168.0.10, ta sẽ thấy thật tuyệt vời, máy này đã ra được Internet

```
ping www.google.com (216.58.210.196): 56 data bytes
64 bytes from 216.58.210.196: seq=0 ttl=50 time=55.799 ms
64 bytes from 216.58.210.196: seq=1 ttl=50 time=65.751 ms
64 bytes from 216.58.210.196: seq=2 ttl=50 time=54.878 ms
64 bytes from 216.58.210.196: seq=3 ttl=50 time=54.186 ms
```
Làm tương tự với eth2 cho local network 172.16.0.0/16 của mạng máy chủ. Ta có thể tùy chọn một IP bất kỳ làm gateway cho mạng này, chẳng hạn 172.16.2.254 cho đẹp. Như vậy ta đã biến 1 con Workstation để không này thành một Gateway server với 1 cánh cổng cho mạng 192.168.0.0/24 và 1 cánh cổng cho mạng 172.16.0.0/16  mà không làm thay đổi cấu trúc mạng đã có mà không cần phải mua Switch Layer 3. Điều này là rất quan trọng để tiết kiệm chi phí và hơn thế nữa, ta còn có thể Monitor cũng như áp dụng các Policy network trên các cánh cổng (interface) này để bảo vệ cho các mạng con trên tùy theo yêu cầu của đơn vị.


