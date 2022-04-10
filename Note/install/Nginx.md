### Compiling and Installing NGINX from Source

<img src="/Note/img/nginx1.png">

There are several types of installation:

```
* Docker image
* Nginx repository
* Operating Systems repositories
* Already compiled package(.rpm .dep .exe)
* Compiling Binary
```

### Step 1: Installing NGINX dependencies

* PCRE — supports regular expressions. Required by the NGINX Core and Rewrite modules.
* zlib — Supports header compression. Required by the NGINX Gzip module.
* OpenSSL — Supports the HTTPS protocol. Required by the NGINX SSL module and others.
* GCC — Compile binaries

```
http_ssl_module: Hỗ trợ https
pcre: Hỗ trợ regular expression matching
file-aio: Hỗ trợ input/output bất đồng bộ
realip_module: Hỗ trợ lấy real ip của origin server nằm đằng sau proxy
gzip_static_module: Hỗ trợ nén data trả về cho những client có hỗ trợ gzip.
```

### Installing NGINX Dependencies

Prior to compiling NGINX Open Source from source, you need to install libraries for its dependencies:

* PCRE – Supports regular expressions. Required by the NGINX Core and Rewrite modules.

```
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.44.tar.gz
tar -zxf pcre-8.44.tar.gz
cd pcre-8.44
./configure
make
sudo make install
```

* zlib – Supports header compression. Required by the NGINX Gzip module.

```
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
sudo make install
```
* OpenSSL – Supports the HTTPS protocol. Required by the NGINX SSL module and others.

```
wget http://www.openssl.org/source/openssl-1.1.1g.tar.gz
tar -zxf openssl-1.1.1g.tar.gz
cd openssl-1.1.1g
./Configure darwin64-x86_64-cc --prefix=/usr
make
sudo make install
```

### Downloading the Sources
```
wget http://nginx.org/download/nginx-1.20.2.tar.gz
tar zxf nginx-1.20.2.tar.g
cd nginx-1.20.2
```

* Configuring the Build Options

- Configure options are specified with the ./configure script that sets up various NGINX parameters, including paths to source and configuration files, compiler options, connection processing methods, and the list of modules. The script finishes by creating the Makefile required to compile the code and install NGINX Open Source.
```
./configure --with-http_ssl_module --with-file-aio  --with-http_realip_module --with-http_stub_status_module --without-http_scgi_module --without-http_uwsgi_module --with-stream \
--with-pcre=../pcre-8.44 --with-zlib=../zlib-1.2.11
make
make install
```

* Start service nginx

```
systemctl start nginx
systemctl status nginx
```
* Check nginx
```
[root@kms nginx-1.20.2]# curl -I 127.0.0.1
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Thu, 27 Jan 2022 15:49:28 GMT
Content-Type: text/html
Content-Length: 4833
Last-Modified: Fri, 16 May 2014 15:12:48 GMT
Connection: keep-alive
ETag: "53762af0-12e1"
Accept-Ranges: bytes

```
* Explain config
```
[root@ kms ~]# vim /etc/nginx/nginx.conf       	 ---  Master profile
user  nginx;		 --- definition worker Process managed users
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;	  --- Define error log path information
pid        /var/run/nginx.pid;				  --- definition pid File path information


events {
    worker_connections  1024;		          ---One worker The process can receive 1024 access requests at the same time
}


http {
    include       /etc/nginx/mime.types;	  --- Load a configuration file
    default_type  application/octet-stream;	  --- Specifies the default identification file type

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';		
				       --- Define the format of the log	
    access_log  /var/log/nginx/access.log  main;  --- Specify log path 

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;		--- Time out

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;  --- Load a configuration file
 }
```

* Server area information

```
/etc/nginx/conf.d/default.conf  ------ Extended configuration(Virtual host profile)
 [root@ lb01 conf.d]# cat default.conf
server {
    listen       80;			--- Specifies the listening port
    server_name  localhost;		--- Specify website domain name

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;	--- Define the location of the site directory
        index  index.html index.htm;	--- Define the front page file(The page displayed when visiting)
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;--- Display page information gracefully
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

### Reference

```
https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/
http://nginx.org/en/download.html

```