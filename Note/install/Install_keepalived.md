### Install keepalived Centos 7

### 1. Mô hình Lab Keepalived IP Failover

- Trong mô hình lab của bài này, ta sẽ có 2 Nginx Web Server (bạn có thể đổi thành HAProxy tùy ý) phục vụ xử lý request HTTP Web cơ bản. Hai Nginx WEB1 và WEB2 này sẽ được cấu hình dùng chung một VIP là 10.2.3.169. Bình thường thì VIP này sẽ do node Master phụ trách, node Backup sẽ ở trạng thái chờ.

<img src="/Note/img/lab-keepalived-nginx-1.jpg">

- Khi có sự cố xảy ra với node Master như die server hay dịch vụ die thì node Backup sẽ nhận lấy VIP này và chịu trách nhiệm xử lý tiếp nội dung dịch vụ đang chạy cụ thể ở bài lab này là Nginx Web Server.

<img src="/Note/img/lab-keepalived-nginx-2.jpg">

### 2. Cài đặt keepalived

- Keepalived

```
yum groupinstall -y "Development Tools" 
yum install -y gcc kernel-headers kernel-devel curl gcc openssl-devel libnl3-devel net-snmp-devel psmisc ipset-libs
yum install -y keepalived
```
- Nginx

```
yum install -y epel-release 
yum install -y nginx
systemctl start nginx
```

- Config nginx

```
+ WEB1

rm -rf /usr/share/nginx/html/*
vi /usr/share/nginx/html/index.html
<h1> BAN DANG TRUY CAP WEB1  </h1>

+ WEB2

rm -rf /usr/share/nginx/html/*
vi /usr/share/nginx/html/index.html
<h1> BAN DANG TRUY CAP WEB2  </h1>
```

- Config service keepalived

```
Cấu hình cho phép gắn địa chỉ IP ảo lên card mạng và IP Forward:

echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/sysctl.conf
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

### 3. File config keepalived

- Để cấu hình dịch vụ keepalived, ta cần phải chỉnh sửa file /etc/keepalived/keepalived.conf. Một số block cấu hình đáng chú ý trong file này như sau:

```
global_defs {
   notification_email {
        anhnt@gmail.com
   }
   notification_email_from anhnt01@gmail.com
   smtp_server x.x.x.x
   smtp_connect_timeout 30

}

vrrp_script chk_haproxy {
        script "command"     
        interval <time>
        weight <n>
}

vrrp_instance string {
    state MASTER|BACKUP
    interface string
    mcast_src_ip @IP
    virtual_router_id num
    priority num
    advert_int num
    smtp_alert
    authentication {
        auth_type PASS|AH
        auth_pass string
    }
    virtual_ipaddress { # Block limited to 20 IP addresses
        @IP
        @IP
    }
    notify_master "/path_to_script/script_fault.sh <arg_list>"
    notify_backup "/path_to_script/script_fault.sh <arg_list>"
    notify_fault "/path_to_script/script_fault.sh <arg_list>"
}
```

* global_defs: Cấu hình thông tin toàn cục (global) cho keepalived như gửi email thông báo tới đâu, tên của cluster đang cấu hình.
* vrrp_script: Chứa script, lệnh thực thi hoặc đường dẫn tới script kiểm tra dịch vụ (Ví dụ: nếu dịch vụ này down thì keepalived sẽ tự chuyển VIP sang 1 server khác).
* vrrp_instance: Thông tin chi tiết về 1 server vật lý trong nhóm dùng chung VRRP.Bao gồm các thông tin như interface dùng để liên lạc của server này, độ ưu tiên để virtual IP tương ứng với interface, cách thức chứng thực, script kiểm tra dịch vụ….

### * Cấu hình block vrrp_instance

– Trong các phần giải thích dưới, router sẽ đồng nghĩa với máy chủ dịch vụ.

* state (MASTER|BACKUP): Chỉ trạng thái MASTER hoặc BACKUP được sử dụng bởi máy chủ. Nếu là MASTER thì máy chủ này có nhiệm vụ nhận và xử lý các gói tin từ host đi lên. Nếu con MASTER die, những con BACKUP này sẽ dựa vào 1 cơ chế bầu chọn và nhảy lên làm Master.
* interface: Chỉ định cổng mạng nào sẽ sử dụng cho hoạt động IP Failover – VRRP
* mcast_src_ip: Địa chỉ IP thực của card mạng Interface của máy chủ tham gia vào VRRP. Các gói tin trao đổi giữa các VRRP Router sử dụng địa chỉ thực này.
* virtual_router_id: Định danh cho các router (ở đây là máy chủ dịch vụ) thuộc cùng 1 nhóm VRRP. Hiểu nôm na là 1 router có thể tham gia nhiều nhóm VRRP (các nhóm hoạt động động lập nhau), và VRRP-ID là tên gọi của từng nhóm.
* priority: Chỉ định độ ưu tiên của VRRP router (tức độ ưu tiên máy chủ dịch vụ trong quá trình bầu chọn MASTER). Các VRRP Router trong cùng một VRRP Group tiến hành bầu chọn Master sử dụng giá trị priority đã cấu hình cho máy chủ đó. Priority có giá trị từ 0 đến 255. Nguyên tắc có bản: Priority cao nhất thì nó là Master, nếu priority bằng nhau thì IP cao hơn là Master.
* advert_int: Thời gian giữa các lần gửi gói tin VRRP advertisement (đơn vị giây).
* smtp_alert: Kích hoạt thông báo bằng email SMTP khi trạng thái MASTER có sự thay đổi.
* authentication: Chỉ định hình thức chứng thực trong VRRP. `auth_type`, sử dụng hình thức mật khẩu plaintext hay mã hoá AH. `auth_pass`, chuỗi mật khẩu chỉ chấp nhận 8 kí tự.
* virtual_ipaddress: Địa chỉ IP ảo của nhóm VRRP đó (Chính là địa chỉ dùng làm gateway cho các host). Các gói tin trao đổi, làm việc với host đều sử dụng địa chỉ ảo này.
* notify_master: Chỉ định chạy shell script nếu có sự kiện thay đổi về trạng thái MASTER.
* notify_backup: Chỉ định chạy shell script nếu có sự kiện thay đổi về trạng thái BACKUP.
* notify_fault: Chỉ định chạy shell script nếu có sự kiện thay đổi về trạng thái thất bại (fault).

### 3.1 Cấu hình Keepalived trên WEB1 – MASTER

- Giờ chúng ta sẽ cấu hình cho máy chủ WEB1 làm máy chủ MASTER trong mô hình Failover IP giữa 2 WEB Server. Tức máy chủ WEB1 sẽ là người xử lý các request HTTP đến thông qua Virtual IP `10.2.3.110` và khi mà máy chủ WEB1 chết thì máy chủ WEB2 sẽ tự động lên làm MASTER và đảm nhận xử lý IP Virtual. Ta sẽ cho độ ưu tiên `priority` là 100 và state là `MASTER`. Có Virtual_router_id giống nhau ví dụ là 50 .


```
[root@web1]# vi /etc/keepalived/keepalived.conf

vrrp_script chk_nginx {
        script "killall -0 nginx"     
        interval 2 # check every 2 seconds
        weight 4 # add 4 points of prioty if OK
}

vrrp_instance VI_1 {
    state MASTER  # Can be the same on both instances, whichever starts first will be the master, or choose MASTER/BACKUP
    interface eth0 
    mcast_src_ip 10.2.3.110 # IP server master
    virtual_router_id 50 # Needs to be the same on both instances, and needs to be unique if using multicast, does not matter with unicast
    priority 100 # Can be the same on both instances, unless using MASTER/BACKUP then the bigger number is for the master
    advert_int 1
    authentication {
        auth_type AH
        auth_pass 12345678
    }
    virtual_ipaddress {
        10.2.3.169 # The VIP that keepalived will monitor
    }
    track_script 
    {
        chk_nginx
    }
}
```

### 3.2 Cấu hình Keepalived trên WEB2 – BACKUP

- Ta sẽ cho độ ưu tiên `priority` là 98 và state là `BACKUP`. Có Virtual_router_id giống nhau ví dụ là 50 

```
[root@web2]# vi /etc/keepalived/keepalived.conf

vrrp_script chk_nginx {
        script "killall -0 nginx"     
        interval 2 # check every 2 seconds
        weight 4 # add 4 points of prioty if OK
}

vrrp_instance VI_1 {
    state BACKUP # Can be the same on both instances, whichever starts first will be the master, or choose MASTER/BACKUP
    interface eth0
    mcast_src_ip 10.2.3.111 # IP server slave
    virtual_router_id 50  # Needs to be the same on both instances, and needs to be unique if using multicast, does not matter with unicast
    priority 98 # Can be the same on both instances, unless using MASTER/BACKUP then the bigger number is for the master
    advert_int 1
    authentication {
        auth_type AH
        auth_pass 12345678
    }
    virtual_ipaddress {
        10.2.3.169 # The VIP that keepalived will monitor
    }
    track_script 
    {
        chk_nginx
    }
}
```



### Start keepalived and iptables

```
systemctl start keepalived
systemctl enable keepalived
```

- Cấu hình firewall rule cho phép VRRP giao tiếp sử dụng dãi địa chỉ multicast 224.0.0.0/8 qua giao thức VRRP (112) trên card mạng mà chúng ta cấu hình Keepalived hoạt động. Nếu bạn sử dụng AH (51) khi cấu hình mật khẩu chứng thực 2 bên thì thêm rule cho giao thức AH nữa.

```
iptables -I INPUT -i eth0 -d 224.0.0.0/8 -p vrrp -j ACCEPT
iptables -I INPUT -i eth0 -d 224.0.0.0/8 -p ah -j ACCEPT
iptables -I OUTPUT -o eth0 -s 224.0.0.0/8 -p vrrp -j ACCEPT
iptables -I OUTPUT -o eth0 -s 224.0.0.0/8 -p ah -j ACCEPT.
service iptables save
```

- Check website with IP VIP:

<img src="/Note/img/web_1.png">

### 5. Check log

- Chúng ta sẽ kiểm tra xem trên server WEB1 đảm nhận vai trò MASTER đã nhận được IP ảo `10.2.3.169` hay chưa nhé. Trong trường hợp server MASTER down, thì địa chỉ ảo VIP 10.2.3.169 sẽ được tự động gán cho máy chủ WEB2 BACKUP lên làm MASTER.

```
[root@mail01 ~]# ip a sh ens32
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:a8:8e:5c brd ff:ff:ff:ff:ff:ff
    inet 10.2.3.110/24 brd 10.2.3.255 scope global noprefixroute ens32
       valid_lft forever preferred_lft forever
    inet 10.2.3.169/32 scope global ens32
       valid_lft forever preferred_lft forever
    inet6 fe80::5e29:4566:8609:e7d6/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
[root@mail01 ~]#
```
- Check log keepalived

```
Jan 18 18:33:25 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) Transition to MASTER STATE
Jan 18 18:33:26 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) Entering MASTER STATE
Jan 18 18:33:26 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) setting protocol VIPs.
Jan 18 18:33:26 mail01 Keepalived_vrrp[13795]: Sending gratuitous ARP on ens32 for 10.2.3.169
Jan 18 18:33:26 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) Sending/queueing gratuitous ARPs on ens32 for 10.2.3.169
Jan 18 18:33:26 mail01 Keepalived_vrrp[13795]: Sending gratuitous ARP on ens32 for 10.2.3.169
```


- Check node master off service nginx:

* ssh to node master 
```
ssh root@10.2.3.110
service nginx stop
```
- Check IP:

```
[root@mail01 ~]# ip a sh ens32
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:a8:8e:5c brd ff:ff:ff:ff:ff:ff
    inet 10.2.3.110/24 brd 10.2.3.255 scope global noprefixroute ens32
       valid_lft forever preferred_lft forever
    inet6 fe80::5e29:4566:8609:e7d6/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

- Check log keepalived:

```
tail -100f /var/log/messages
Jan 18 18:41:28 mail01 Keepalived_vrrp[13795]: /usr/bin/killall -0 nginx exited with status 1
Jan 18 18:41:28 mail01 Keepalived_vrrp[13795]: VRRP_Script(chk_nginx) failed
Jan 18 18:41:28 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) Changing effective priority from 104 to 100
Jan 18 18:41:29 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) Received advert with higher priority 102, ours 100
Jan 18 18:41:29 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) IPSEC-AH : Syncing seq_num - Decrement seq
Jan 18 18:41:29 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) Entering BACKUP STATE
Jan 18 18:41:29 mail01 Keepalived_vrrp[13795]: VRRP_Instance(VI_1) removing protocol VIPs.
```

* ssh to node slave

```
[root@mail02 ~]# ip a sh ens32
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:a8:ec:00 brd ff:ff:ff:ff:ff:ff
    inet 10.2.3.111/24 brd 10.2.3.255 scope global noprefixroute ens32
       valid_lft forever preferred_lft forever
    inet 10.2.3.169/32 scope global ens32
       valid_lft forever preferred_lft forever
    inet6 fe80::3267:4f29:1232:cff5/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
[root@mail02 ~]#
```
- Check log keepalived:
```
tail  -100f /var/log/messages
Jan 18 18:41:24 mail02 Keepalived_vrrp[7213]: VRRP_Instance(VI_1) forcing a new MASTER election
Jan 18 18:41:25 mail02 Keepalived_vrrp[7213]: VRRP_Instance(VI_1) Transition to MASTER STATE
Jan 18 18:41:26 mail02 Keepalived_vrrp[7213]: VRRP_Instance(VI_1) Entering MASTER STATE
Jan 18 18:41:26 mail02 Keepalived_vrrp[7213]: VRRP_Instance(VI_1) setting protocol VIPs.
Jan 18 18:41:26 mail02 Keepalived_vrrp[7213]: Sending gratuitous ARP on ens32 for 10.2.3.169
```

--> IP VIP đã được chuyển sang node slave

- Check website with IP VIP:

<img src="/Note/img/web_2.png">

--> Keepalived change to host slave

* Cấu hình Server không khôi phục quyền MASTER

- Trong một số trường hợp, bạn không muốn server WEB1 vừa chết xong quay lại chiếm quyền làm MASTER trong giữa 2 máy chủ Keepalived:

+ Cấu hình thêm `nopreempt`.

```
[root@mail02 ~]# cat /etc/keepalived/keepalived.conf
vrrp_script chk_nginx {
        script "killall -0 nginx"
        interval 2 # check every 2 seconds
        weight 4 # add 4 points of prioty if OK
}

vrrp_instance VI_1 {
    state BACKUP # Can be the same on both instances, whichever starts first will be the master, or choose MASTER/BACKUP
    interface ens32
    mcast_src_ip 10.2.3.111 # IP server slave
    virtual_router_id 50  # Needs to be the same on both instances, and needs to be unique if using multicast, does not matter with unicast
    priority 98 # Can be the same on both instances, unless using MASTER/BACKUP then the bigger number is for the master
    advert_int 1
    nopreempt 
    authentication {
        auth_type AH
        auth_pass 12345678
    }
    virtual_ipaddress {
        10.2.3.169 # The VIP that keepalived will monitor
    }
    track_script
    {
        chk_nginx
    }
}
```

### Một số tham số config keepalived

- A check script is a script written in the language of your choice which is executed regularly. This script needs to have a return value: 0 for "everything is fine", 1 (or other than 0) for "something went wrong".
This value is used by Keepalived to take action. Scripts are defined like this:

```
vrrp_script chk_myscript {
  script       "/usr/local/bin/mycheckscript.sh"
  interval 2   # check every 2 seconds
  fall 2       # require 2 failures for OK
  rise 2       # require 2 successes for OK
}
```

```
vrrp_script chk_courier_F {
       script "/etc/keepalived/check_script.sh  courier FAULT"
       timeout 35
       rise 2
       interval 5
}

vrrp_script chk_courier_B {
        script "/etc/keepalived/check_script.sh  courier  BACKUP"
        interval 5
        timeout 35
        rise 2
        weight  1
}
vrrp_script chk_network_F {
        script  "/etc/keepalived/check_network.sh"
        rise 2
        timeout 35
        fall 7
        interval 5
        }


vrrp_instance courier {
    state  MASTER
    interface eth0
    virtual_router_id 25
    advert_int 5
    priority  100
    unicast_src_ip 100.77.189.139

    unicast_peer {

        100.77.189.137

    }
    authentication {
        auth_type PASS
        auth_pass 1111
    }

     track_script {
        chk_courier_B
        chk_courier_F
        chk_network_F
        }


    notify_master "/etc/keepalived/notify.sh courier  MASTER"
    notify_backup "/etc/keepalived/notify.sh courier  BACKUP"
    notify_fault  "/etc/keepalived/notify.sh courier FAULT"
}
```

* interval: How often the script should be run (1 second).
* timeout: How long to wait for the script to return (5 seconds).
* rise: How many times the script must return successfully in order for the host to be considered “healthy.” In this example, the script must return successfully 3 times. This helps to prevent a “flapping” condition where a single failure (or success) causes the Keepalived state to quickly flip back and forth.
* fall: How many times the script must return unsuccessfully (or time out) in order for the host to be considered “unhealthy.” This functions as the reverse of the rise directive.


### Configuring Keepalived for unicast heartbeat

* Unicast Keepalived

- So have configured haproxy and keepalived for our mariadb cluster, we've then moved VLANs around and found that we don't have multicast support in place on some switches for some VLANs.

- Unfortunately, this is a problem because the default keepalived setup (the one from the last post) uses multicast to find out where the other loadbalancers are. Without it, no dice. Or, more precisely, a lot of dice, since each loadbalancer will then become the master loadbalancer and you'll get some weird behaviour.

- So, unicast it will be then. As far as I can tell, the only really issue with this is some light security concerns that seem to require local access to the network anyway, so there's nothing stopping a switch to unicast.

- To do so, you'll need to tweak the keepalived config:


```
vrrp_instance VI_01 {
  interface eno16777984
  state MASTER # BACKUP on the backup
  virtual_router_id 51
  priority 100 # 50 on the backup
  unicast_src_ip XXX.XXX.XXX.XXX

  unicast_peer {
    XXX.XXX.XXX.XXX
  }
}
```


- So all we're really doing here is adding a unicast_src_ip (the loadbalancer IP) and a unicast_peer (the other loadbalancer(s))

- And obviously you'll need to set up the virtual address as before.

- With this configured on each loadbalancer and the services restarted, we're back to having working loadbalancers, without the need to reconfigure switches to support multicast.


### MASTER node


```
vrrp_instance HAPROXY_VIPS {
    state MASTER
    interface eth1 # interface connected to VRRP net
    virtual_router_id 51
    priority 101 # number here must be > than on BACKUP
    advert_int 1
    unicast_src_ip <MASTER-vrrp-net-ip>
    unicast_peer {
      <backup-vrrp-net-ip>
    }
    authentication {
        auth_type PASS
        auth_pass 1111 # make it whatever you want, but only the first 8 chars will be used
    }
    virtual_ipaddress {
        X.X.X.X/32 brd X.X.X.255 dev eth0 label eth0:0
        X.X.X.Y/32 brd X.X.X.255 dev eth0 label eth0:1
        X.X.X.Z/32 brd X.X.X.255 dev eth0 label eth0:2
    }
  
}
```

### BACKUP node

```
vrrp_instance HAPROXY_VIPS {
    state BACKUP
    interface eth1
    virtual_router_id 51
    priority 100 # must be lower than MASTER
    advert_int 1
    unicast_src_ip <BACKUP-vrrp-net-ip>
    unicast_peer {
      <master-vrrp-net-ip>
    }
    authentication {
        auth_type PASS
        auth_pass 1111 # same as MASTER
    }
    virtual_ipaddress {
        X.X.X.X/32 brd X.X.X.255 dev eth0 label eth0:0
        X.X.X.Y/32 brd X.X.X.255 dev eth0 label eth0:1
        X.X.X.Z/32 brd X.X.X.255 dev eth0 label eth0:2
    }
  
}
```