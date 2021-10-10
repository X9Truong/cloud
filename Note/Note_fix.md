```
Failed to start LSB: Bring up/down networking.

systemctl stop NetworkManager
systemctl disable NetworkManager

  
cat /sys/module/kvm_intel/parameters/nested
  
echo "options kvm-intel nested=y" >> /etc/modprobe.d/dist.conf
  
sudo groupadd docker
sudo usermod -aG docker name_user  
  
  
yum install yum-utils -y
yum clean all
yum-complete-transaction


dmesg | grep kvm


git config --global user.name "anhbka"
git config --global user.email "anhnt@mail.com"
cat ~/.gitconfig
config --list


sudo kill `sudo lsof -t -i:8989`
```

Turning gitlab

* Storing Git data in an alternative directory

```
git_data_dirs({
  "default" => { "path" => "/var/opt/gitlab/git-data" },
  "alternative" => { "path" => "/mnt/nas/git-data" }
})
```

gitlab_rails['backup_path'] = your_path

