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
GitLab should be available at http://192.168.138.200

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

  Verifying  : gitlab-ce-13.0.0-ce.0.el7.x86_64                                                                                                              1/1

Installed:
  gitlab-ce.x86_64 0:13.0.0-ce.0.el7

Complete!
```

* Seting basic for gitlab

`http://192.168.138.200/`

`Change password`

```
From now on, use the below credentials to log in as the administrator:

Username: root
Password: <your-new-password>
```

* Configure GitLab URL

```
cd /etc/gitlab/
vim gitlab.rb
external_url 'http://gitlab.hakase-labs.co'
```

* Generate SSL Let's encrypt and DHPARAM certificate

```
yum -y install letsencrypt

letsencrypt certonly --standalone -d gitlab.hakase-labs.co
```

* Next, create new 'ssl' directory under the GitLab configuration directory '/etc/gitlab/'.


`mkdir -p /etc/gitlab/ssl/`

```
sudo openssl dhparam -out /etc/gitlab/ssl/dhparams.pem 2048
chmod 600 /etc/gitlab/ssl/*
```

* Enable Nginx HTTPS for GitLab


```
cd /etc/gitlab/
vim gitlab.rb
And change HTTP to HTTPS on the external_url line.
external_url 'https://gitlab.hakase-labs.co'.

Then paste the following configuration under the 'external_url' line configuration.


nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/letsencrypt/live/gitlab.hakase-labs.co/fullchain.pem"
nginx['ssl_certificate_key'] = "/etc/letsencrypt/live/gitlab.hakase-labs.co/privkey.pem"
nginx['ssl_dhparam'] = "/etc/gitlab/ssl/dhparams.pem"

```
Finally, apply the GitLab configuration using the following command.


`gitlab-ctl reconfigure`

* Install gitlab runner

```
# Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Give it permissions to execute
sudo chmod +x /usr/local/bin/gitlab-runner
ln -s /usr/local/bin/gitlab-runner /usr/bin/gitlab-runner
echo "gitlab-runner ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Create a GitLab CI user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as service
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

* Change other user use gitlab runner

```
vim /etc/systemd/system/gitlab-runner.service
[Service]
User=myuser
```

* install docker on server

`curl -sSL https://get.docker.com | sudo sh`

* Add user to docker group

```
sudo usermod -aG docker $(whoami)
sudo usermod -aG docker gitlab-runner
```
`before_script` – ý là trước khi thực hiện script. Ở đây ta khai báo before_script ở “root level” nên nó sẽ được áp dụng cho tất cả các job.


* Shared runners pipeline minutes quota

```
To change the pipelines minutes quota:

Go to Admin Area > Settings > CI/CD.
Expand Continuous Integration and Deployment.
In the Pipeline minutes quota box, enter the maximum number of minutes.
Click Save changes for the changes to take effect.
```
https://docs.gitlab.com/ee/user/admin_area/settings/continuous_integration.html#shared-runners-pipeline-minutes-quota

https://docs.gitlab.com/runner/install/docker.html



