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
echo "export PROMPT_COMMAND='{ date "+[ %Y%m%d %H:%M:%S `whoami` `tty`] `history 1 | { read x cmd; echo "$cmd"; }`"; } \
     >> /var/log/cmd_history'" >> /etc/profile
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
 