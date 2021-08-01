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







 