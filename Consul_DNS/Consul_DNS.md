<h2>Consul Cluster Setup on CentOS 7</h2>

```
ssh-keygen
ssh-copy-id root@ansible1
ssh-copy-id root@ansible2
ssh-copy-id root@ansible3
```

<table class=""><thead><tr><th>Short Hostname</th><th>IP Address</th></tr></thead><tbody><tr><td>consul-01</td><td>192.168.10.10</td></tr><tr><td>consul-02</td><td>192.168.10.11</td></tr><tr><td>consul-03</td><td>192.168.10.12</td></tr></tbody></table>


```
cd /usr/local/bin
sudo curl -o consul.zip https://releases.hashicorp.com/consul/1.6.0/consul_1.6.0_linux_amd64.zip
unzip consul.zip
sudo rm -f consul.zip
sudo mkdir -p /etc/consul.d/scripts
sudo mkdir /var/consul
```

* Creat a consul secret 

`consul keygen` # save key

If you don’t have this value, you can get it from the consul server from /var/consul/serf/local.keyring file. 

*  Create a config file on all three servers.

```
vi /etc/consul.d/config.json
{
    "bootstrap_expect": 3,
    "client_addr": "0.0.0.0",
    "datacenter": "Us-Central",
    "data_dir": "/var/consul",
    "domain": "consul",
    "enable_script_checks": true,
    "dns_config": {
        "enable_truncate": true,
        "only_passing": true
    },
    "enable_syslog": true,
    "encrypt": "V6VE8oSCY9sKEjBIC5PXdrOjDElNGH4ieb97Y6fw1Cc=",
    "leave_on_terminate": true,
    "log_level": "INFO",
    "rejoin_after_leave": true,
    "server": true,
    "start_join": [
        "192.168.30.128",
        "192.168.30.129",
        "192.168.30.130"
    ],
    "retry_join": [
        "192.168.30.128",
        "192.168.30.129",
        "192.168.30.130"
    ],	
    "ui": true
}
```
<ul style="text-align: justify;"><li><strong>bootstrap_expect</strong> : số lượng Consul mode Server cần có để chạy trong một data center Consul Cluster.</li><li><strong>client_addr</strong> : địa chỉ IP mà Consul sẽ listen interface, bao gồm các dịch vụ HTTP và DNS.</li><li><strong>datacenter</strong> : quy định thông tin tên Cluster Data Center hoạt động, giả sử có nhiều cụm Cluster name khác nhau, thì tương ứng các datacenter name khác nhau. Các consul server phải cùng hoạt động chung 1 data center và 1 domain.</li><li><strong>data_dir</strong> : thư mục chứa dữ liệu hoạt động của Consul.</li><li><strong>domain</strong> :&nbsp;mặc định Consul sẽ xử lý các query dns đến domain ‘consul.’ . Bạn có thể tuỳ biến domain riêng cho hệ thống nội bộ, khớp với domain các dịch vụ trong mạng nội bộ của bạn.</li><li><strong>enable_script_checks</strong> : kích hoạt sử dụng cơ chế health check qua script.</li><li><strong>dns_config</strong> : một số option tinh chỉnh cho service DNS Consul Server.</li><li><strong>enable_syslog</strong> : kích hoạt syslog cho Consul.</li><li><strong>encrypt</strong> : chuỗi kí tự dùng để mã hoá trao đổi cơ bản giữa các Consul member trong Consul Cluster.</li><li><strong>log_level</strong> : cấp độ log được ghi lại.</li><li><strong>log_file</strong> : thông tin file log dịch vụ Consul ghi log.</li><li><strong>rejoin_after_leave</strong> : kích hoạt cho option ‘retry_join’.</li><li><strong>server</strong> : chạy Consul Agent ở chế độ Server, thay vì Client Mode.</li><li><strong>start_join</strong> : danh sách các máy chủ mà Consul Agent cần</li><li><strong>retry_join</strong> : thử join vào danh sách các máy chủ trong cụm Cluster Consul, nếu lần đầu ‘<strong>start_join</strong>‘ bị thất bại.</li><li><strong>ui&nbsp;</strong>: kích hoạt giao diện dashboard quản lý consul.</li></ul>



* Create a Consul Service


```
vi /etc/systemd/system/consul.service
[Unit]
Description=Consul Startup process
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c '/usr/local/bin/consul agent -config-dir /etc/consul.d/'
TimeoutStartSec=0

[Install]
WantedBy=default.target
```

* Firewalld

```
firewall-cmd --add-port={8300,8301,8302,8400,8500,8600}/tcp --permanent
firewall-cmd --add-port={8301,8302,8600}/udp --permanent
firewall-cmd --reload
```
* Iptables

```
iptables -I INPUT -p tcp --match multiport --dports 8300,8301,8302,8400,8500,8600 -j ACCEPT
iptables -I INPUT -p udp --match multiport --dports 8301,8302,8600 -j ACCEPT
```


* Reload the system daemons

`systemctl daemon-reload`


* Bootstrap and Start the Cluster

Step 1: On consul-1 server, start the consul service

`systemctl start consul`

Step 2: Start consul on other two servers (Consul-2 and consul-3) using the following command.

`systemctl start consul`

Step 3: Check the cluster status by executing the following command.

```
[root@ansible1 bin]# /usr/local/bin/consul members
Node      Address              Status  Type    Build  Protocol  DC          Segment
ansible1  192.168.30.128:8301  alive   server  1.6.0  2         us-central  <all>
ansible2  192.168.30.129:8301  alive   server  1.6.0  2         us-central  <all>
ansible3  192.168.30.130:8301  alive   server  1.6.0  2         us-central  <all>
```


### Access Consul UI

`http://<consul-IP>:8500/ui`

* Demo

`https://demo.consul.io/ui/dc1/services`

`https://www.consul.io/api/agent/service.html#register-service`






https://cuongquach.com/cai-dat-consul-agent-client-mode.html
https://computingforgeeks.com/how-to-setup-consul-cluster-on-centos-rhel/
https://cuongquach.com/trien-khai-consul-cluster-service-discovery.html
https://cuongquach.com/truy-van-dns-trong-consul.html
https://devopscube.com/hsetup-configure-consul-agent-client-mode/
https://cuongquach.com/dang-ky-co-che-kiem-tra-healthcheck-tren-consul.html










