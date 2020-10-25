* MySQL 8 on CentOS 7

```
rpm -Uvh https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
sed -i 's/enabled=1/enabled=0/' /etc/yum.repos.d/mysql-community.repo
yum --enablerepo=mysql80-community install mysql-community-server -y
service mysqld start
grep "A temporary password" /var/log/mysqld.log
mysql_secure_installation
service mysqld restart
chkconfig mysqld on

chmod 777 /public/config
cd /var/www/html/tcexam
	find . -exec chown -R apache:apache {} \;
	find . -type f -exec chmod 544 {} \;
	find cache/ -type f -exec chmod 644 {} \;
	find cache/backup -type f -exec chmod 644 {} \;
	find cache/lang -type f -exec chmod 544 {} \;
	find admin/log/ -type f -exec chmod 644 {} \;
	find public/log/ -type f -exec chmod 644 {} \;
	find . -type d -exec chmod 755 {} \;
	
```
yum-config-manager --enable remi-php71
yum install php php-mysql
yum -y install httpd php php–mysql mariadb sqlite php–dom php–mbstring php–gd php–pdo wget bzip2 php php–mysql php–dom php–mbstring php–gd php–pdo curl ImageMagick libdbi-dbd-mysql php-gd php-curl zbar gd

```
yum install php72-php php-cli
GD curl ImageMagick Yum texlive ZBar mysql-server PHP httpd php-mysql libdbi-dbd-mysql php-gd php-curl mod_php72w php72w-opcache php72w-pdo php72w-mysql php72w-mbstring
```