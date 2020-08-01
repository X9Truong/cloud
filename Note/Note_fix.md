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