### Install Memcached

```
yum install memcached -y
yum install libmemcached -y
```
Memcached should now be installed on your CentOS system as a service, along with the tools that required you to test its connectivity. Now we can proceed further to secure its configuration settings.

### Securing Memcached Configuration Settings

```
vi /etc/sysconfig/memcached
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 127.0.0.1 -U 0" 
```

Let’s discuss each of the above parameters in detail.

```
PORT : The port used by Memcached to run.
USER : The start-up daemon for Memcached service.
MAXCONN : The value used to set max simultaneous connections to 1024. For busy web servers, you can increase to any number based on your requirements.
CACHESIZE : Set cache size memory to 2048. For busy servers, you can increase up to 4GB.
OPTIONS : Set the IP address of the server, so that Apache or Nginx web servers can connect to it.
```

* Restart and enable Memcached service

```
systemctl restart memcached
systemctl enable memcached
memcached-tool 127.0.0.1 stats
```

* Install Memcached PHP extension

`yum install php-pecl-memcache -y`

* Install Memcached Perl Library

`yum install perl-Cache-Memcached -y`

* Install Memcached Python Library

`yum install python-memcached`

* Restart Web Server

```
systemctl restart httpd
systemctl restart nginx
```









