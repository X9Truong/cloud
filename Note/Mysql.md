### Quản lý MySQL User Permissions (quyền thao tác của MySQL user)

* Gán quyền truy cập vào database cho user mới bằng lệnh sau:

```
GRANT ALL PRIVILEGES ON newdb.* TO 'username'@'localhost'


SELECT – user có quyền đọc (read) database bằng lệnh select
CREATE – họ có thể tạo bảng mới
DROP – cho phép người dùng xóa bảng
DELETE – users có thể xóa dòng khỏi bảng
INSERT – giúp user thêm dòng vào bảng
UPDATE – giúp cập nhật dòng
GRANT OPTION – có thể gán hoặc xóa quyền của user khác

GRANT CREATE ON newdb.* TO 'username'@'localhost'

REVOKE permission_type ON newdb.* TO 'username'@'localhost'

SHOW GRANTS username

FLUSH PRIVILEGES

```
```
# Preconfiguration setup

groupadd mysql
useradd -r -g mysql -s /bin/false mysql


# Beginning of source-build specific instructions
tar zxvf mysql-VERSION.tar.gz
cd mysql-VERSION
mkdir bld
cd bld
cmake ..
make
make install

# End of source-build specific instructions

# Postinstallation setup
cd /usr/local/mysql
mkdir mysql-files
chown mysql:mysql mysql-files
chmod 750 mysql-files
bin/mysqld --initialize --user=mysql
bin/mysql_ssl_rsa_setup
bin/mysqld_safe --user=mysql &

# Next command is optional
cp support-files/mysql.server /etc/init.d/mysql.server
```

