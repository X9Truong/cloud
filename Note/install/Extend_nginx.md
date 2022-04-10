### Tạo một vhost

- Vhost viết tắt của Virtual host là kỹ thuật cho phép nhiều website có thể chia sẻ chung một IP. Thuật ngữ này bắt nguồn từ apache.

* Cấu hình vhost cho nginx và cấu hình php-fpm để vhost xử lý được file php.

```
/usr/local/nginx/conf/nginx.conf - Đây là file cấu hình chung của nginx
/usr/local/nginx/conf.d/vhost.example.com.conf - Đây chính là file cấu hình cho vhost
vim /usr/local/nginx/conf/nginx.conf
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /usr/local/nginx/conf/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    include /usr/local/nginx/conf.d/*.conf;
}
```
```
* Chuyển log vào /var/log/nginx không để trong thư mục cài đặt
* Chuyển pid file vào /var/run không để trong thư mục cài đặt
* Bỏ comment dòng log_format
* Sử dụng đường dẫn tuyệt đối hoàn toàn
* Thêm dòng include /usr/local/nginx/conf.d/*.conf;
* Thêm dòng user nginx;
```
* Tạo sẵn một directory để chứa log nginx và config

```
mkdir -p /var/log/nginx
chown -R nginx:nginx /var/log/nginx
mkdir -p /usr/local/nginx/conf.d/
```

* Update config

```
vi /usr/local/nginx/conf.d/vhost.example.com.conf
server{
     listen       80;
     server_name vhost.example.com www.vhost.example.com;
     root /home/www/vhost.example.com;
     error_log  /var/log/nginx/vhost.example.com_error.log error;
     access_log  /var/log/nginx/vhost.example.com_access.log  main;
     location /{
         index index.html index.php;
     }
}
```

* Tạo document root cho vhost vhost.example.com

```
mkdir -p /home/www/vhost.example.com
chown -R nginx:nginx /home/www/vhost.example.com
```
* Tạo nội dung cho website

```
vim /home/www/vhost.example.com/test.html
<html><body>Welcome to website</body></html>
```

* Config file hosts

```
vim /etc/hosts
10.2.3.169 vhost.example.com
```

### Cấu hình để nginx xử lý php

- Bản thân nginx không hỗ trợ xử lý php. Việc xử lý php sẽ do một service khác đảm nhận. Nginx sẽ forward request đến service này và nhận kết quả về. Service này là php-fpm.

* Install php-fpm

`yum instal php-fpm -y`

* Update config

```
/etc/php-fpm.conf
/etc/php-fpm.d/www.conf
/usr/local/nginx/conf.d/vhost.example.com.conf

vim /etc/php-fpm.conf
include=/etc/php-fpm.d/*.conf
[global]
pid = /var/run/php-fpm/php-fpm.pid
error_log = /var/log/php-fpm/error.log
log_level = warning
daemonize = yes

vim /etc/php-fpm.d/www.conf
[www]
listen = /tmp/php_fpm.sock
listen.allowed_clients = 127.0.0.1
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
user = apache
group = apache
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
slowlog = /var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path] = /var/lib/php/session

vim /usr/local/nginx/conf.d/vhost.example.com.conf
 location ~ \.php {
    fastcgi_pass unix:/tmp/php_fpm.sock;
    fastcgi_index   index.php;
    include         /usr/local/nginx/conf/fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME $document_root/$fastcgi_script_name;
}
```

- Mục đích của đoạn cấu hình trên là: Toàn bộ các request đến các file có tận cùng là php sẽ được đẩy đến unix socket mà php-fpm lắng nghe để biên dịch. Chính vì thế listen.owner và listen.group của unix socket config là nginx:nginx

* Test

```
vim /home/www/vhost.example.com/test.php
<?php
echo "Test php";
?>
```

* Restart service

```
systemctl restart nginx  
systemctl restart php-fpm service
```

--> `10.2.3.69/test.php`

