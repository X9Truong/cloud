### Turning OS

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager

echo "net.netfilter.nf_conntrack_tcp_be_liberal = 1
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.tcp_keepalive_time = 6
net.ipv4.tcp_keepalive_intvl = 3
net.ipv4.tcp_keepalive_probes = 6
net.ipv4.ip_forward = 1
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0" >> /etc/sysctl.conf
modprobe ip_conntrack
sysctl -p
```
* Nếu có lỗi chạy thêm lệnh : modprobe ip_conntrack
 
`https://www.cyberciti.biz/faq/linux-kernel-etcsysctl-conf-security-hardening/`


`echo "fs.file-max=500000" >> /etc/sysctl.conf`

* /etc/security/limits.conf

```
# End of file
* hard nofile 65536
* soft nofile 45000
```

* /etc/security/limits.d/20-nproc.conf

```
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

*          soft    nproc     40000
root       soft    nproc     unlimited
```

* Lưu history command
```
touch /var/log/cmd_history
chmod 0666 /var/log/cmd_history
/etc/profile
export PROMPT_COMMAND='{ date "+[ %Y%m%d %H:%M:%S `whoami` `tty`] `history 1 | { read x cmd; echo "$cmd"; }`"; } \
     >> /var/log/cmd_history'
source /etc/profile

```
* Cài đặt package

`yum groupinstall "Development Tools" -y`


Turning gitlab

* Storing Git data in an alternative directory

```
git_data_dirs({
  "default" => { "path" => "/mnt/nas/git-data"" }
})
```

gitlab_rails['backup_path'] = your_path

* Config logo

-> Admin -> Appearance

https://docs.gitlab.com/omnibus/settings/configuration.html


* Change multi line

```
:s/^/#

Ex: :1;10s/^/# this will comment out line 1-10

:%s/^/#/        will comment out all lines in file
```

```
Visual Mode
The other method you can use to comment out multiple lines is to use Visual Mode.

To do this, press ESC and navigate to the lines you want to comment out.

Press CTRL + V to enable Visual Mode.

Using the up and down arrow key, highlight the lines you wish to comment out.

Once you have the lines selected, press the SHIFT + I keys to enter insert mode.

Enter your command symbol, for example, # sign, and press the ESC key. Vim will comment out all the highlighted lines.
```

* Uncomment

`:%s/^#/`

* Change network

```
echo "Setup IP  ens34"
nmcli c modify ens34 ipv4.addresses 192.168.1.192/24
nmcli c modify ens34 ipv4.gateway 192.168.1.1
nmcli c modify ens34 ipv4.dns 8.8.8.8
nmcli c modify ens34 ipv4.method manual
nmcli con mod ens34 connection.autoconnect yes
```

### Note

* Remove packages

`yum remove`

* Update packages

`yum update httpd`

* List packages by yum list

`yum list openssh`

* Search packages by yum search

`yum search firefox`

* Receive information about packages 

`yum info firefox`

* Check update

`yum check-update`

`yum grouplist`

`yum groupinstall 'MySQL Database'`

`yum groupremove 'DNS Name Server'`

`yum groupupdate 'DNS Name Server'`

* Repo list active

`yum repolist`

`yum repolist all`

`yum clean all`

`yum history list`

** Download packages RPM 

`yum install yum-utils -y`

`yumdownloader unbound`

`yum localinstall unbound-1.6.6-1.el7.x86_64.rpm`

### Stop start daemontools

```
$ for x in $(cat front_iplist); do echo ==$x== ; ssh $x "" svstat /service/mysqlrouter ""; done
$ for x in $(cat front_iplist); do echo ==$x== ; ssh $x "" svc -u /service/mysqlrouter ""; done
$ for x in $(cat front_iplist); do echo ==$x== ; ssh $x "" svstat /service/mysqlrouter ""; done
```


### Change line 

```
n=$(wc -l < infile)
paste -d '' <(yes '<before>' | head -n $n) \
            infile                         \
            <(yes '</after>' | head -n $n)
```

```
<before>line 1</after>
<before>line 2</after>
<before>line 3</after>
```
`sed 's/.*/<before> & and <after each line>/' input.txt > output.txt`


```
s/x/y/ - thay thế x bởi y
.* - lấy toàn bộ dòng
& - nhận văn bản phù hợp
```
`awk '{print before, $0, after}' before="<before>" after="and <after each line" input.txt > output.txt`

`Điều này chỉ in nội dung của biến before, $0 (dòng hiện tại) và biến after, cách nhau bởi khoảng trắng. Các biến được đặt sau biểu thức awk`

```
:%s/.*/<before> & and <after each line>/
:wq! out.txt
```

```
ex in.txt '+%s/.*/<before> & and <after each line>/' '+wq out.txt'
vi in.txt '+%s/.*/<before> & and <after each line>/' '+wq! out.txt'
```

```
ip route get 1 | sed 's/^.*src \([^ ]*\).*$/\1/;q'
/usr/sbin/ip route get 1 | awk '{print $NF;exit}'
Get ip
```
 
```
#!/bin/bash
export BASE_DIR={{hostvars[inventory_hostname][app_folder]['BASE_DIR']}}
export APP_HOME=${BASE_DIR}/{{app_folder | default('app', true)}}
export APP_NAME={{hostvars[inventory_hostname][item]['alias_name']| default(item)}}
export APP_ID={{hostvars[inventory_hostname][item]['APP_ID']}}
export APP_IP={{ansible_ssh_host}}
export ENV={{ENVIROMENT}}
export MIN_MEMORY={{hostvars[inventory_hostname][item]['MIN_MEMORY']| default('512m')}}
export MAX_MEMORY={{hostvars[inventory_hostname][item]['MAX_MEMORY']| default('2048m')}}
export MIN_PermSize={{hostvars[inventory_hostname][item]['MIN_PermSize'] | default('64m')}}
export MAX_PermSize={{hostvars[inventory_hostname][item]['MAX_PermSize'] | default('128m')}}
export LAUNCH_TARGET="{{hostvars[inventory_hostname][item]['LAUNCH_TARGET']}}"
export APP_VERSION={{APP_VERSION}}
${APP_HOME}/bin/rack.sh $1
```

```
activemq.url=failover:({% set list= [] %}{% for item in  groups['activemq'] %}{{list.append("tcp://"+hostvars[item]["ansible_host"]+":61616")}}{%endfor%}{{list| join(',')}})

###################################################
# HA
###################################################
locker.impl=etcd

locker.etcd.retries=15
locker.etcd.timeout=16000
locker.etcd.heartbeat=2000

locker.mysql.retries=0
locker.mysql.timeout=12000
locker.mysql.heartbeat=3000

etcd.peers={% set list= [] %}{% for item in  groups['etcd'] %}{{list.append("http://"+hostvars[item]["ansible_host"]+":2379")}}{%endfor%}{{list| join(',')}}
#################################################
# Redis
#################################################
redis.common.url=redis-sentinel://{% set list= [] %}{% for item in  groups['redis'] %}{{list.append(hostvars[item]["ansible_host"]+":26379")}}{%endfor%}{{list| join(',')}}/0?master={{REDIS_MASTER_NAME}}
redis.common.password={{REDIS_ENC_PASS}}

redis.session.url=redis-sentinel://{% set list= [] %}{% for item in  groups['redis'] %}{{list.append(hostvars[item]["ansible_host"]+":26379")}}{%endfor%}{{list| join(',')}}/1?master={{REDIS_MASTER_NAME}}
redis.session.password={{REDIS_ENC_PASS}}

redis.payment.url=redis-sentinel://{% set list= [] %}{% for item in  groups['redis'] %}{{list.append(hostvars[item]["ansible_host"]+":26379")}}{%endfor%}{{list| join(',')}}/2?master={{REDIS_MASTER_NAME}}
redis.payment.password={{REDIS_ENC_PASS}}

redis.customer.url=redis-sentinel://{% set list= [] %}{% for item in  groups['redis'] %}{{list.append(hostvars[item]["ansible_host"]+":26379")}}{%endfor%}{{list| join(',')}}/3?master={{REDIS_MASTER_NAME}}
redis.customer.password={{REDIS_ENC_PASS}}

redis.wallet.url=redis-sentinel://{% set list= [] %}{% for item in  groups['redis'] %}{{list.append(hostvars[item]["ansible_host"]+":26379")}}{%endfor%}{{list| join(',')}}/4?master={{REDIS_MASTER_NAME}}
redis.wallet.password={{REDIS_ENC_PASS}}

redis.history.url=redis-sentinel://{% set list= [] %}{% for item in  groups['redis'] %}{{list.append(hostvars[item]["ansible_host"]+":26379")}}{%endfor%}{{list| join(',')}}/5?master={{REDIS_MASTER_NAME}}
redis.history.password={{REDIS_ENC_PASS}}
```


```
yum install -y httpd-tools
htpasswd -c /etc/nginx/.htpasswd nginx
cat /etc/nginx/.htpasswd
nano /etc/nginx/nginx.conf
```
server {
	listen       80 default_server;
	listen       [::]:80 default_server;
	server_name  _;
	root         /usr/share/nginx/html;

	auth_basic "Private Property";
	auth_basic_user_file /etc/nginx/.htpasswd;
```

`systemctl reload nginx`

* Kích hoạt mode debug docker

```
# vi /etc/docker/daemon.json
{
"debug": true
}

systemctl restart docker

docker info | grep -i debug.*server

tail -f /var/log/messages | grep -i docker
```


```
---
- name: filter test
  hosts: localhost
  vars: 
    
    names: [ {
        "first": "Paul",
        "last": "Thompson",
        "mobile": "+1-234-31245543",
        "ctc": "100000",
        "address": {
          "city": "LasVegas",
          "country": "USA"
        }
      },
      {
        "first": "Rod",
        "last": "Johnson",
        "mobile": "+1-584-31551209",
        "ctc": "300000",
        "address": {
          "city": "Boston",
          "country": "USA"
        }
      },
      {
        "first": "Sarav",
        "last": "AK",
        "mobile": "+919876543210",
        "ctc": "200000",
        "address": {
          "city": "Chennai",
          "country": "India"
        }
      }]
  tasks:
  
  # Map Filter only selective attributes from list of objects [{},{}]
  - name: Select and Extract only the cities 
    debug: 
      msg="{{ names | map(attribute='address') | map(attribute='city')}}"
  # using attirubtes with list of objects [{},{}] - Selecting only mobile numbers
  - name: Select and Extract only mobile numbers
    debug:
      msg: "{{ names | map(attribute='mobile') }}"
  # Select Attributes Joined with Comma in Singleline ( By Default it returns a List)
  - debug: msg={{ names | map(attribute='first') | join(',') }} 
  - debug: msg={{ names | map(attribute='last') | join(',') }} 
  # Convert the lastname to uppercase
  - debug: msg={{ names | map(attribute='last') | map('upper') }} 
  # Convert the CTC attriute to float value
  - debug: msg={{ names | map(attribute='ctc') | map('float') }}
  # Appending USD to each CTC value and print
  - debug: msg={{ names | map(attribute='ctc') | product(['USD']) | map('join',' ')}}
```  

### Backup etcd 

```
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/backup/etcd.db
  
Check status 

ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/backup/etcd.db
  
```


### Restore etcd Using Snapshot Backup

`ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/etcd.db`


If you want to use a specific data directory for the restore, you can add the location using the --data-dir flag as shown below.

`ETCDCTL_API=3 etcdctl --data-dir /opt/etcd snapshot restore /opt/backup/etcd.db`



```
CREATE USER 'replica'@'192.%.%.%' IDENTIFIED BY 'R3plic4$';

ALTER USER 'replica'@'192.%.%.%' IDENTIFIED WITH mysql_native_password BY 'R3plic4$';

GRANT REPLICATION SLAVE ON *.* TO 'replica'@'192.%.%.%';

CHANGE MASTER TO
MASTER_HOST='192.168.30.136',
MASTER_USER='replica' ,
MASTER_PASSWORD='R3plic4$',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=157;

set GLOBAL super_read_only=1;
select @@global.read_only, @@global.super_read_only;
```

`https://quantrimang.com/cong-nghe/14-lenh-linux-thu-vi-trong-terminal-160595`

```
---
- hosts: mysql
  gather_facts: no
  tasks:
    -  debug:
#        var: hostvars[inventory_hostname]['groups']['mysql']
        var: hostvars[inventory_hostname]['groups']['mysql']
#    - name: Debug
#      debug:
#        var: hostvars
#  tasks:
#    - debug: var=ansible_host
    - debug: var=inventory_hostname
#    - debug: var=hostvars
    - name: Debug name
      shell: "ls /opt"
      register: check
      changed_when: false
      when:
#        - ansible_host == "192.168.30.136"
        - hostvars[inventory_hostname]['mode'] == "slave"
    - name: check
      debug: var=check
    - name: Debug var mode
      debug: var=hostvars[inventory_hostname]['MAX_MEMORY']
    - debug: var=hostvars[inventory_hostname]['zone']
    - debug: var=hostvars[inventory_hostname]['APP_ID']
    - debug: var=hostvars[inventory_hostname]['launch_target']

all:
  hosts:
    localhost:
      ansible_host: 127.0.0.1
  children:
    mysql:
      hosts:
        dba:
          ansible_host: 10.2.3.110
          mode: active
          MIN_MEMORY: 2048m
          MAX_MEMORY: 4096m
          zone: 1a
          APP_ID: 1
          launch_target: "bo.rate.writer.RateWriterStartup"
          APP_NAME: RateWriter
        dba2:
          ansible_host: 10.2.3.111
          mode: slave
          MIN_MEMORY: 2048m
          MAX_MEMORY: 4096m
          zone: 1b
          APP_ID: 2
          launch_target: "bo.rate.writer.RateWriterStartup"
          APP_NAME: RateWriter
    nginx:
      hosts:
        dba:
          ansible_host: 10.2.3.110
          mode: active
          APP_ID: 3
          MIN_MEMORY: 1024m
          MAX_MEMORY: 2048m
        dba2:
          ansible_host: 10.2.3.111
          mode: active
          APP_ID: 4
          MIN_MEMORY: 1024m
          MAX_MEMORY: 2048m
```

```
gg: go first line
dG : delete all line
```
		  
```
yum -y install bash-completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
exec bash
```



```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: kube-01
```


```
Commands shared in the PPT
kubectl run nginx --image=nginx   (deployment)
kubectl run nginx --image=nginx --restart=Never   (pod)
kubectl run nginx --image=nginx --restart=OnFailure   (job)  
kubectl run nginx --image=nginx  --restart=OnFailure --schedule="* * * * *" (cronJob)

kubectl run nginx -image=nginx --restart=Never --port=80 --namespace=myname --command --serviceaccount=mysa1 --env=HOSTNAME=local --labels=bu=finance,env=dev  --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi' --dry-run -o yaml - /bin/sh -c 'echo hello world'

kubectl run frontend --replicas=2 --labels=run=load-balancer-example --image=busybox  --port=8080
kubectl expose deployment frontend --type=NodePort --name=frontend-service --port=6262 --target-port=8080
kubectl set serviceaccount deployment frontend myuser
kubectl create service clusterip my-cs --tcp=5678:8080 --dry-run -o yaml


If you use vi you can set the config ':set expandtab' to convert any tabs into spaces.
```