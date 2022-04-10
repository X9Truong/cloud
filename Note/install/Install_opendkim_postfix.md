### Configure DomainKeys (OpenDKIM) with Postfix on CentOS 7


### OpenDKIM

OpenDKIM là một mã nguồn mở của hệ thống xác thực người gửi DKIM (Domain Keys Identified Mail), là hệ thống xác thực email được thiết kế để phát hiện các email giả mạo bằng cách cung cấp một cơ chế cho phép trao đổi thư nhận được để kiểm tra xem thư đến từ miền có được ủy quyền bởi miền đó không. Người nhận có thể xác thực chữ ký điện tử đi kèm với thư bằng cách sử dụng chữ ký xác thực được công khai trên DNS.

#### Config on server
```
1. vi /etc/security/limit.conf
Edit file limit.conf -->  (update content like bellow)
*     soft   nofile  150000
*     hard   nofile  200000
2. vi /etc/security/limits.d/20-nproc.conf
Edit file 20-nproc.conf --> (update content like bellow ) 
*          soft    nproc     40000
root       soft    nproc     unlimited

3. vi /etc/sysctl.conf 
Edit file systctl.conf --> (update content like bellow ) 
fs.file-max=500000

4. vi /etc/profile
Edit file profile --> (update content like bellow) 
export PROMPT_COMMAND='{ date "+[ %Y%m%d %H:%M:%S `whoami` `tty`] `history 1 | { read x cmd; echo "$cmd"; }`"; } \
     >> /var/log/cmd_history'
5. sysctl -p 
6. touch /var/log/cmd_history
7. chmod 777 /var/log/cmd_history
touch  /etc/postfix/postfix-files
```

### Install OpenDKIM

```
yum install curl wget vim openssl man -y
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum repolist
yum install opendkim postfix -y 
```
```
/etc/opendkim.conf - configuration file of opendkim
/etc/opendkim/keytable — defines the path of the private key for the domain
/etc/opendkim/signingtable - tells OpenDKIM how to apply the keys.
/etc/opendkim/TrustedHosts - defines which hosts are allowed to use keys.
```
* Config postfix
```
/etc/postfix/main.cf
myhostname = example.com
mydomain = example.com
inet_interfaces = all
inet_protocols = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain, example.com
relay_domains = $mydestination  
```


```
systemctl enable postfix
systemctl start postfix
```

* Run below Command to create keys

```
MYDOMAIN=example.com
mkdir -p /etc/opendkim/keys/$MYDOMAIN
cd /etc/opendkim/keys/$MYDOMAIN
opendkim-genkey --restrict --nosubdomains --selector=default --domain=$MYDOMAIN --bits=2048 --directory=/etc/opendkim/keys

selector - This parameter is responsible for the name of the key with which mail will be subscribed. In order for mail to be signed by the main domain example.com, we specify the selector "default".
domain - this is the domain name.
bits - key strength, recommend >=2048

```
Lệnh trên sẽ tạo ra hai tệp default.private và default.txt . Bạn có thể tạo nhiều khóa DKIM cho 2 domain khác nhau và định cấu hình bằng máy chủ postfix của mình.

### Config record DKIM trên domain
`https://mediatemple.net/community/products/dv/115003098072/how-do-i-add-a-dkim-txt-record-to-my-domain`

* Thay đổi permission
```
chown -R opendkim:opendkim /etc/opendkim
chmod go-rw /etc/opendkim/keys
```

* Thay đổi tên file 

```
mv /etc/opendkim/keys/default.private /etc/opendkim/keys/$MYDOMAIN.private
mv /etc/opendkim/keys/default.txt /etc/opendkim/keys/$MYDOMAIN.txt
```

```
chown root:opendkim /etc/opendkim/keys/*
chmod 640 /etc/opendkim/keys/*
```

## Cấu hình OpenDKIM

Theo mặc định, OpenDKIM được đặt thành chế độ xác minh (v), xác minh việc nhận chữ ký DKIM của thư email. Để kích hoạt chế độ ký tên cho các email gửi đi, chúng ta cần nhập “Mode sv”

`Mode v --> Mode sv # sign and verify outgoing messages`


```
vim /etc/opendkim.conf
PidFile	/tmp/opendkim.pid
Mode	sv
Syslog	yes
SyslogSuccess	yes
LogWhy	yes
UserID	postfix:postfix
Socket	local:/var/spool/postfix/private/opendkim
Umask	002
SendReports	yes
Canonicalization	relaxed/relaxed
Domain	*
MinimumKeyBits	1024
ExternalIgnoreList	refile:/etc/opendkim/TrustedHosts
InternalHosts	refile:/etc/opendkim/TrustedHosts
PeerList	X.X.X.X
OversignHeaders	From
QueryCache	yes
KeyTable                refile:/etc/opendkim/KeyTable
SigningTable            refile:/etc/opendkim/SigningTable
SignatureAlgorithm      rsa-sha256

```


LogWhy yes  --> to see this error log messages:
```
postfix_1                         | Feb 16 22:13:11 santa-maria opendkim[290]: 8888F3239D2: no signing domain match for 'stephane-klein.info'
postfix_1                         | Feb 16 22:13:11 santa-maria opendkim[290]: 8888F3239D2: no signing subdomain match for 'stephane-klein.info'
postfix_1                         | Feb 16 22:13:11 santa-maria opendkim[290]: 8888F3239D2: no signature data
```
Umask 002 --> Assigns permissions so that only you and members of your group have read/write access to files, and read/write/search access to directories you own. All others have read access only to your files, and read/search to your directories

Domain `*`  --> Multi domain

* Config  /etc/opendkim/TrustedHosts
```
/etc/opendkim/TrustedHosts
127.0.0.1
::1
#host.example.com
#192.168.1.0/24
# Danh sách domain
*.example.com
```
* Config /etc/opendkim/KeyTable

`default._domainkey.example.com example.com:default:/etc/opendkim/keys/example.com.private`


* Config  /etc/opendkim/SigningTable
```
*@example.com default._domainkey.example.com
```

* Thêm OpenDKIM vào Postfix bằng cách thêm đoạn dưới vào /etc/postfix/main.cf

```
vim  /etc/postfix/main.cf

smtpd_milters = inet:127.0.0.1:8891 # Socket inet:8891@localhost - use port 8891 at localhost for communication
non_smtpd_milters = $smtpd_milters  # Postfix to forward all mail to the opendkim first by adding local milter
milter_default_action = accept
milter_protocol = 2
sender_dependent_default_transport_maps= radnmap{smtp1,smtp2,smtp3,smtp4,smtp5}
```
* Config master.cf
```
smtp1    unix   -       -       n       -       -       smtp
        -o syslog_name=postfix-smtp1
        -o smtp_destination_rate_delay=1s
        -o fallback_transport_maps=radnmap{smtp2,smtp3,smtp4,smtp5}
smtp2    unix   -       -       n       -       -       smtp
        -o syslog_name=postfix-smtp2
        -o smtp_destination_rate_delay=1s
        -o fallback_transport_maps=radnmap{smtp1,smtp3,smtp4,smtp5}
smtp3    unix   -       -       n       -       -       smtp
        -o syslog_name=postfix-smtp3
        -o smtp_destination_rate_delay=1s
        -o fallback_transport_maps=radnmap{smtp1,smtp2,smtp4,smtp5}
smtp4    unix   -       -       n       -       -       smtp
        -o syslog_name=postfix-smtp4
        -o smtp_destination_rate_delay=1s
        -o fallback_transport_maps=radnmap{smtp1,smtp2,smtp3,smtp5}
smtp5    unix   -       -       n       -       -       smtp
        -o syslog_name=postfix-smtp5
        -o smtp_destination_rate_delay=1s
        -o fallback_transport_maps=radnmap{smtp1,smtp2,smtp3,smtp4}
```

Change config file:
```
/usr/lib/systemd/system/postfix.service
[Unit]
Description=Postfix Mail Transport Agent
After=syslog.target network.target
Conflicts=sendmail.service exim.service

[Service]
Type=forking
PIDFile=/var/spool/postfix/pid/master.pid
EnvironmentFile=-/etc/sysconfig/network
ExecStartPre=-/usr/libexec/postfix/aliasesdb
ExecStartPre=-/usr/libexec/postfix/chroot-update
ExecStart=/usr/sbin/postfix start
ExecReload=/usr/sbin/postfix reload
ExecStop=/usr/sbin/postfix stop

[Install]
WantedBy=multi-user.target
```

* Now we can restart OpenDKIM and check status:

`systemctl restart opendkim && systemctl status opendkim`

```
smtpd_recipient_restrictions = 
    permit_mynetworks  # The purpose of the smtpd_recipient_restrictions feature is to control how Postfix replies to the RCPT TO command
    check_helo_access hash:/etc/postfix/helo_access
	reject_unknown_helo_hostname
	reject_unauth_destination

smtpd_client_restrictions		#Reject all client commands
smtpd_helo_restrictions		#Reject HELO/EHLO information
smtpd_sender_restrictions		#Reject MAIL FROM information
smtpd_recipient_restrictions	#Required	Reject RCPT TO information
smtpd_data_restrictions		#Reject DATA command
smtpd_end_of_data_restrictions		#Reject END-OF-DATA command
smtpd_etrn_restrictions		#Reject ETRN command
```
queue_run_delay #(default: 1000 seconds) Thời gian giữa các lần quét queue bị hoãn bởi người quản lý queue.

minimal_backoff_time #(default: 1000 seconds) Thời gian giữa các lần quét queue bị hoãn bởi người quản lý queue

maximal_backoff_time #(default: 4000 seconds) Thời gian tối đa giữa các lần gửi tin nhắn hoãn lại.

maximal_queue_lifetime #(default: 5 days) Coi thư là không gửi được, khi gửi không thành công với lỗi tạm thời và thời gian trong queue đã đạt đến giới hạn maxal_queue_lifetime .

bounce_queue_lifetime #(default: 5 days, available with Postfix version 2.1 and later) How long a MAILER-DAEMON message stays in the queue before it is considered undeliverable. Specify 0 for mail that should be tried only once.

smtp_fallback_relay # Theo mặc định, thư được trả lại cho người gửi khi không tìm thấy điểm đến và việc gửi thư bị hoãn lại khi không thể truy cập được điểm đến.

default_process_limit # configuration parameter gives direct control over how many daemon processes Postfix will run. As of Postfix 2.0 the default limit is 100 smtp client processes, 100 smtp server processes, and so on. This may overwhelm systems with little memory, as well as networks with low bandwidth.

recipient_delimiter #


address_verify_positive_expire_time # (default: 31d) The time after which a successful probe expires from the address verification cache.

Specify a non-zero time value (an integral value plus an optional one-letter suffix that specifies the time unit). Time units: s (seconds), m (minutes), h (hours), d (days), w (weeks). The default time unit is d (days).

address_verify_poll_delay # 
Sự chậm trễ giữa các truy vấn để hoàn thành một yêu cầu xác minh địa chỉ đang được tiến hành.
Độ trễ bỏ phiếu mặc định là 3 giây.
Chỉ định giá trị thời gian khác 0 (giá trị tích phân cộng với hậu tố một chữ cái tùy chọn chỉ định đơn vị thời gian). Đơn vị thời gian: s (giây), m (phút), h (giờ), d (ngày), w (tuần). Đơn vị thời gian mặc định là s (giây).


default_destination_concurrency_limit #(default: 20) The default maximal number of parallel deliveries to the same destination.

default_destination_concurrency_limit = 10 # A destination concurrency limit of 10 for SMTP delivery seems enough to noticeably load a system without bringing it to its knees. Be careful when changing this to a much larger number.

default_extra_recipient_limit #(default: 1000) Giá trị mặc định cho giới hạn bổ sung cho mỗi lần gửi email được áp dụng cho số lượng người nhận trong bộ nhớ. Không gian người nhận bổ sung này được dành riêng cho các trường hợp khi bộ lập lịch của trình quản lý queue Postfix xếp trước một tin nhắn với một tin nhắn khác và đột nhiên cần một số vùng người nhận bổ sung cho tin nhắn đã chọn để tránh giảm hiệu suất.

extract_recipient_limit #  (default: 10240) Số lượng địa chỉ người nhận tối đa mà Postfix sẽ trích xuất từ ​​tiêu đề thư khi thư được gửi bằng " sendmail -t ".

message_size_limit # (default: 10240000) The maximal size in bytes of a message, including envelope information.
qmgr_message_recipient_limit # (default: 20000) Số lượng người nhận tối đa
qmgr_message_active_limit # (default: 20000) Số lượng thư tối đa trong queue hiện hoạt động.
qmqpd_error_delay # How long the Postfix QMQP server will pause before sending a negative reply to the remote QMQP client. 


smtpd_history_flush_threshold #(default: 100) Số dòng tối đa trong lịch sử lệnh của máy chủ Postfix SMTP trước khi nó được xóa khi nhận EHLO, RSET hoặc kết thúc DATA.



* Một số tham số logs send email
```
Dec 08 14:04:05 aamsmail002 postfix-smtp1/smtp[31675]: 673DA3007B0D: to=<xxx@chaos-space.com>, relay=chaos-space.com[112.78.112.174]:25, delay=0.34, delays=0.02/0.01/0.28/0.04, dsn=2.0.0, status=sent (250 2.0.0 xB8545C4067243 Message accepted for delivery)
````

```
# Message delivery time stamps
# delays=a/b/c/d, where
#   a = time before queue manager, including message transmission
#   b = time in queue manager
#   c = connection setup including DNS, HELO and TLS;
#   d = message transmission time.
```
### The postsuper command
```


So, to delete all emails in the queue, we use this command as:

postsuper -d ALL
Also, to remove all mails in the deferred queue, the command is as follows:

postsuper -d ALL deferred
Similarly, we can remove a particular mail from the queue using the command,

postsuper -d mail_id
And, we get the mail queue ID on running the mailq command.

To remove messages from a particular domain.

mailq | grep domain.com | awk {'print $1'} | xargs -I{} postsuper -d {}
```
Refer: 
```
https://www.linuxtopia.org/online_books/mail_systems/postfix_documentation/postconf.5.html#permit_mynetworks
https://www.thesysadmin.rocks/2020/01/13/postfix-sender_dependent_default_transport_maps-per-domain-outgoing-ip/
http://www.postfix.org/postconf.5.html
https://www.oreilly.com/library/view/postfix-the-definitive/0596002122/ch04s05.html
```



