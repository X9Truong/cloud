## Install gitlab on Centos 7

For system performance purposes, it is recommended to configure the kernel's swappiness setting to a low value like 10:

```
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
cat /proc/sys/vm/swappiness
```

* Requirement: 

```
CPU: 4 Core
RAM: 4 GB
Disk: 100 GB
```

* Install repo and udpate

```
yum install epel-release -y
yum update -y
init 6


sudo yum install -y curl policycoreutils-python openssh-server openssh-clients
sudo yum install -y postfix
sudo systemctl enable postfix.service
sudo systemctl start postfix.service
```

* Set host name

```
hostnamectl set-hostname gitlab
```

* Disabled firewalld and selinux

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

* Install chrony and sync time, restart server

```
timedatectl set-timezone Asia/Ho_Chi_Minh

yum -y install chrony
systemctl enable chronyd.service
systemctl restart chronyd.service
chronyc sources
timedatectl set-local-rtc 0

init 6
````

* Install Gitlab

```
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
```

* Install repo gitlab

`curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash`

* Install gitlab CE

`sudo EXTERNAL_URL="http://192.168.239.200" yum install -y gitlab-ce`

```
Gitlab use IP 192.168.239.200 for EXTERNAL_URL = IP
If you want use domain => update EXTERNAL_URL
```
* Result

```
Chef Client finished, 543/1469 resources updated in 03 minutes 24 seconds
gitlab Reconfigured!

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.



     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/


Thank you for installing GitLab!
GitLab should be available at http://192.168.239.200

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

  Verifying  : gitlab-ce-13.0.0-ce.0.el7.x86_64                                                                                                              1/1

Installed:
  gitlab-ce.x86_64 0:13.0.0-ce.0.el7

Complete!
```

* Seting basic for gitlab

`http://192.168.239.200/`

`Change password`

```
From now on, use the below credentials to log in as the administrator:

Username: root
Password: <your-new-password>
```
