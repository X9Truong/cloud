### Install postfix mail

* Remove sendmail

`yum remove sendmail`

* Disable selinux

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

* Install postfix

`yum -y install postfix cyrus-sasl-plain mailx`

* Setup prostfix MTA as default system.

`alternatives --set mta /usr/sbin/postfix`

* If result output:

`/usr/sbin/postfix has not been configured as an alternative for mta`

--> use command `alternatives --set mta /usr/sbin/sendmail.postfix`

* Restart and enabled postfix

```
systemctl restart postfix
systemctl enable postfix
```

```
[root@mail ~]# systemctl status postfix
● postfix.service - Postfix Mail Transport Agent
   Loaded: loaded (/usr/lib/systemd/system/postfix.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2020-05-30 11:09:38 +07; 7s ago
 Main PID: 25474 (master)
   CGroup: /system.slice/postfix.service
           ├─25474 /usr/libexec/postfix/master -w
           ├─25475 pickup -l -t unix -u
           └─25476 qmgr -l -t unix -u

May 30 11:09:37 mail systemd[1]: Starting Postfix Mail Transport Agent...
May 30 11:09:38 mail postfix/postfix-script[25472]: starting the Postfix mail system
May 30 11:09:38 mail postfix/master[25474]: daemon started -- version 2.10.1, configuration /etc/postfix
May 30 11:09:38 mail systemd[1]: Started Postfix Mail Transport Agent.
[root@mail ~]#
```

* Config postfix

```
vi /etc/postfix/main.cf
myhostname = hostname.example.com
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
```

* Configure Postfix SASL Credentials

Creat file `touch /etc/postfix/sasl_passwd` and write :

`[smtp.gmail.com]:587 username:password`

```
username: address use send mail
password: password email use send mail
```
* Permission for the newly created file

```
postmap /etc/postfix/sasl_passwd
chown root:postfix /etc/postfix/sasl_passwd*
chmod 640 /etc/postfix/sasl_passwd*
systemctl reload postfix
```

* Allow account gmail

`https://myaccount.google.com/lesssecureapps`

* Test

`echo "Setup postfix mail server" | mail -s "Mail test server" anh.ict.bkhn@gmail.com`


<img src="/Note/img/999.jpg">






























