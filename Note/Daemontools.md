### Centos Daemontoolsmontoos

1. Install wget gcc and wget
    yum install gcc wget -y

2. Create a folder setup and wget the source
```
   mkdir -p /package
   chmod 1755 /package
   cd /package
   wget http://cr.yp.to/daemontools/daemontools-0.76.tar.gz
   tar -xzvf daemontools-0.76.tar.gz
   rm daemontools-0.76.tar.gz
   mkdir /package/daemontools-0.76
   mv package src /package/daemontools-0.76/
```   

3. Install the daemontools

```
   echo gcc -O2 -include /usr/include/errno.h > src/conf-cc
   ./package/install
```  
4. Fix Deamontools for CentOS 6.x. To do that simply remove the added line from /etc/inittab then issue this:
```
    echo "start on runlevel [12345]" > /etc/init/svscan.conf
    echo "respawn" >> /etc/init/svscan.conf
    echo "exec /command/svscanboot" >> /etc/init/svscan.conf
```
   
5. Start svscan

    /command/svscanboot &
    
6. That’s it. Now you can check it using command below:
    ps -ef|grep svscan
```
[root@ansible1 daemontools-0.76]# ps -ef | grep svs
root       6145   1226  0 23:51 pts/0    00:00:00 /bin/sh /command/svscanboot
root       6147   6145  0 23:51 pts/0    00:00:00 svscan /service
root       6199   1226  0 23:51 pts/0    00:00:00 grep --color=auto svs
```	

```
yum -y install psmisc
[root@ansible1 daemontools-0.76]# pstree -a -p 6145
svscanboot,6145 /command/svscanboot
  +-readproctitle,6148 service errors:...
  +-svscan,6147 /service  
```

```
vim /etc/systemd/system/daemontools.service 
[Unit]
Description=daemontools Start supervise
After=getty.target
[Service]
Type=simple
User=root
Group=root
Restart=always
ExecStart=/command/svscanboot /dev/ttyS0
TimeoutSec=0
[Install]
WantedBy=multi-user.target
```

```
systemctl start daemontools.service
systemctl status daemontools.service
systemctl enable daemontools.service
```
7. Creating services see this http://isotope11.com/blog/manage-your-services-with-daemontools

8. Daemontools chef cookbook for centos https://github.com/CPAN-API/prepan-cookbooks/tree/master/site-cookbooks/daemontools


