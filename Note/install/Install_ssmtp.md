### SSMTP

`yum install ssmtp -y`

`yum --enablerepo=extras install epel-release -y`

### Configuration
 
```
/etc/ssmtp/ssmtp.conf
/etc/ssmtp/revaliases
```

### GMail

`vi /etc/ssmtp/ssmtp.conf`

```
root=gmail@gmail.com
mailhub=smtp.gmail.com:587
AuthUser=gmail_username
AuthPass=gmail_password
AuthMethod=LOGIN
UseSTARTTLS=YES
UseTLS=YES
```

```
/etc/ssmtp/revaliases
root:gmail@gmail.com:smtp.gmail.com:587
```

### Test run

`ssmtp delivery@address.com`

```
To: anhnt@mail.com
Subject: sSMTP test

... it works!
```

### Send a file

```
To: delivery@address.com
Subject: sSMTP test

... it works!
```

`ssmtp delivery@address.com < /email/msg.txt`

### Using sSMTP with PHP

sSMTP can be also used as a drop-in replacement of sendmail to send mail through PHP using the native PHP  mail()  function: all we need to do is to configure the sendmail_path parameter within our php.ini file in the following way:

`sendmail_path = /usr/sbin/ssmtp -t`


`https://www.binarytides.com/linux-mailx-command/`







