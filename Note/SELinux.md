### Cài đặt thêm 1 số package 
`yum -y install policycoreutils policycoreutils-python selinux-policy selinuxpolicy-targeted libselinux-utils setroubleshoot-server setools setoolsconsole mcstrans 
`
- 02 gói dịch vụ để test SELinux:

```
 yum install httpd vsftpd 
 systemctl start httpd 
 systemctl start vsftpd 
 ```
 ### SELinux Mode

* Có 3 chế độ SELinux  đó là:

+ Enforcing: Ở chế độ này, SELinux áp các policy trên toàn hệ thống và đảm bảo các truy nhập bới các user và proceess không được chứng thực sẽ bị từ chối. Các truy cập bị từ chối sẽ được ghi vào Log.

+ Permissive: Ở chế độ này, SELinux không từ chối bất kì truy nhập nào, đây là chế độ tốt nhất để test các SELinux policy.

+ Disabled: Chế độ này thì SELinux bị disabled hoàn toàn và không ghi lại log 

File cấu hình của SELinux là /etc/selinux/config. Ta sẽ enable SELinux bằng setup SELINUX=permissive trong file này và reboot lại.

SELINUXTYPE trong file config cũng có 3 tùy chọn:

+ Targerted: Đây là giá trị mặc định cho phép customize và tinh chỉnh các policy

+ Minimum: Ở chế độ này, chỉ các process được chọn mới được bảo vệ

+ MLS: Multi Level Security là một chế độ bảo vệ mở rộng và cần thêm các để cài đặt nó

Ta sẽ để SELINUXTYPE ở giá trị mặc định và SELinux ở chế độ “permissive”, vì chỉ ở chế độ này các lỗi và truy cập bị chặn mới được ghi vào log. Việc theo dõi liên tục log để xác định lỗi là một thao tác không thể bỏ qua:

`grep 'SELinux' /var/log/messages`

```
[root@controller ~]# grep 'SELinux' /var/log/messages
Mar 23 15:47:05 localhost kernel: SELinux:  Initializing.
Mar 23 15:47:06 localhost systemd[1]: Successfully loaded SELinux policy in 87.031ms.
Mar 23 15:47:07 localhost systemd: Starting Migrate local SELinux policy changes from the old store structure to the new structure...
Mar 23 15:47:07 localhost systemd: Started Migrate local SELinux policy changes from the old store structure to the new structure.
Mar 23 15:50:32 localhost kernel: SELinux:  Context system_u:unconfined_r:sandbox_t:s0-s0:c0.c1023 became invalid (unmapped).
Mar 23 15:50:33 localhost kernel: SELinux:  Context unconfined_u:unconfined_r:sandbox_t:s0-s0:c0.c1023 became invalid (unmapped).
Mar 23 15:52:16 localhost kernel: SELinux:  Initializing.
Mar 23 15:52:18 localhost kernel: SELinux:  Class bpf not defined in policy.
Mar 23 15:52:18 localhost kernel: SELinux: the above unknown classes and permissions will be allowed
Mar 23 15:52:18 localhost systemd[1]: Successfully loaded SELinux policy in 103.628ms.
Mar 23 15:57:12 zabbix kernel: SELinux:  Initializing.
Mar 23 15:57:14 zabbix kernel: SELinux:  Class bpf not defined in policy.
Mar 23 15:57:14 zabbix kernel: SELinux: the above unknown classes and permissions will be allowed
Mar 23 15:57:14 zabbix systemd[1]: Successfully loaded SELinux policy in 122.172ms.
Mar 23 16:06:32 zabbix kernel: SELinux:  Class bpf not defined in policy.
Mar 23 16:06:32 zabbix kernel: SELinux: the above unknown classes and permissions will be allowed
May 31 22:34:55 zabbix kernel: SELinux:  Initializing.
May 31 22:34:59 zabbix kernel: SELinux:  Class bpf not defined in policy.
May 31 22:34:59 zabbix kernel: SELinux: the above unknown classes and permissions will be allowed
May 31 22:34:59 zabbix systemd[1]: Successfully loaded SELinux policy in 182.023ms.
May 31 22:36:56 controller kernel: SELinux:  Initializing.
May 31 22:36:58 controller kernel: SELinux:  Class bpf not defined in policy.
May 31 22:36:58 controller kernel: SELinux: the above unknown classes and permissions will be allowed
May 31 22:36:58 controller systemd[1]: Successfully loaded SELinux policy in 88.710ms.
May 31 22:46:22 controller kernel: SELinux:  Class bpf not defined in policy.
May 31 22:46:22 controller kernel: SELinux: the above unknown classes and permissions will be allowed
[root@controller ~]#
```
* SELinux Policy

Policy là một set các rules định nghĩa mối quan hệ tương tác, các tính năng bảo mật và quyền truy cập cho các đối tượng sau:

+ User: Tất cả các user thông thường được định nghĩa bới 1 hoặc nhiều SELinux user. Rất chú ý ở đây là SELinux user hoàn toàn khác một user thông thường trên Linux.

+ Role: Như là những filter định nghĩa user nào có thể truy nhập vào một process. Nó giống như cánh cổng giữa user và process. Một user chỉ có thể chạy một process cụ thể nếu role gán cho nó và user có quyền truy nhập vào role. SELinux dựa trên Role Based Access Control (RBAC).

+ Subject và Object: Subject giống như một procees hoặc một chương trinh và object là bất kỳ cái gì có thể hoạt động trên đó như một file, port , thư mục ….Các hoạt động được thực hiện bới một subject trên một object phụ thuộc vào quyền của subject.

+ Domain: Giống như wrapper quanh Subject, nó cho process biết cái nó có thể hoặc không thể làm. Chẳng hạn, domain sẽ định nghĩa thư mục, folder, file, ports và một process có thể truy nhập. Domain có liên quan đến Subject trong SELinux.

+ Type: Một file context được gọi là type. Chẳng hạn, context của một file mô tả nó chỉ có thể truy nhập vào process web server nội bộ hay đến bất cứ process nào nằm trong thư mục khác hoặc một SELinux user cụ thể nào là owner của file.Type liên quan đến object trong SELinux.
Trong SELinux, policy định nghĩa các rules cho các user truy nhập vào roles, roles truy nhập vào domain và domain truy nhập vào type.

Có 3 dạng truy nhập chính trong SELinux được định nghĩa bới SELINUXTYPE trong file config `/etc/selinux/config` là :  Type Enforcement (TE), Role-Based Access Control (RBAC) và Multi-Level Security (MLS)

Targeted policy được sử dụng mặc định trong SELinux. Ở đây, SELinux không thay thế DAC policy truyền thống của Linux, thay vào đó, nếu một việc truy nhập vào một file bị cấm bởi DAC, SELinux sẽ bị bỏ qua.

SELinux policy được load trong một định dạng module giống như những kernel module trong bộ nhớ và có thể xem được bằng lệnh sau:

```
[root@controller ~]# semodule -l | more
abrt    1.4.1
accountsd       1.1.0
acct    1.6.0
afs     1.9.0
aiccu   1.1.0
aide    1.7.1
ajaxterm        1.0.0
alsa    1.12.2
amanda  1.15.0
amtu    1.3.0
anaconda        1.7.0
antivirus       1.0.0
apache  2.7.2
apcupsd 1.9.0
apm     1.12.0
application     1.2.0
arpwatch        1.11.0
asterisk        1.12.1
auditadm        2.2.0
authconfig      1.0.0
authlogin       2.5.1
automount       1.14.1
.....
````
Lệnh semodule được sử dụng để cài đặt, xóa, reload, upgrade, enable và disable các module SELinux policy. Các file module là các file binary nằm trong thư mục /etc/selinux/targeted/modules/active/modules với đuôi .pp và chúng có liên quan đến các ứng dụng khác nhau trong Linux. Các policy module này được kết hợp vào một policy đang hoạt động và sau được load vào bộ nhớ. Policy này được đặt trong thư mục /etc/selinux/targeted/policy. Ta không thể thay đổi trực tiếp các rules này nhưng có thể quản lý được nhờ việc sử dụng lệnh semanage boolean như sau:

`semanage boolean -l | less`

<img src="/Note/img/2.png">

Trạng thái đối với ftpd services bằng câu lệnh sau:

`semanage boolean -l | grep ftpd`

<img src="/Note/img/2.JPG">

Từ trên ta có thể thấy, FTP user không có quyền full access, ftpd service cũng không thể chuyển sang chế độ passive mode. 

```
# getsebool ftpd_full_access
ftpd_full_access --> off
# getsebool ftpd_use_passive_mode
ftpd_use_passive_mode --> off
```
<img src="/Note/img/3.JPG">

Ta cần Enable các thông số này như sau:

```
setsebool -P ftpd_full_access on
setsebool -P ftpd_use_passive_mode on
```
Lúc này các tính năng trên sẽ hoạt động trơn tru.


 Context trong SELinux là một tập hợp các thông tin liên quan đến bảo mật giúp SELinux xác định các policy kiểm soát truy nhập. Trong Linux, mọi thứ đều có security context nhưng security context khác nhau cho những loại object khác nhau. Ta hiển thị SELinux file context cho một file bằng lệnh sau:

`ls -alZ /var/*`


Ta được như sau:

`drwx------. apache apache system_u:object_r:httpd_cache_t:s0 httpd`

<img src="/Note/img/4.png">


Có 4 phần phân cách nhau bới dấu “:”

+ Phần đầu là SELinux user context, được chỉ ra là system_u ở đây là một SELinux user.

+ Phần thứ hai là SELinux roles với object_r

+ Phần thứ ba quan trọng nhất là cái định nghĩa loại file (type of context) và chúng ta hiểu rằng nó thuộc về thư mục của một user

+ Phần thứ tư (s0) cho biết tính chất của file và nó làm việc với MLS

Bây giờ ta sẽ xem SELinux process context sử dụng httpd ta đã cài trước đó.


`ps -efZ | grep httpd`

<img src="/Note/img/4.JPG">


Security context ở đây là:

`system_u:system_r:httpd_t:s0`

Với  users kí hiệu bởi _u, roles kí hiệu bởi _r, và types kí hiệu bởi _t. 

### SELinux users

SELinux user được định nghĩa trong policy được load vào bộ nhớ lúc khởi động và chỉ có một vài user này. Các user thông thường trên Linux sẽ được map vào các SELinux user. Để xem các mapping này, ta dùng lệnh sau:

`semanage login -l`

<img src="/Note/img/5.JPG">

Ở đây ta thấy chỉ có user thông thường và root được map vào unconfined_u trong khi đó process và các services được map vào system_u SELinux user. Để xem toàn bộ SELinux user đang có trên hệ thống, ta dùng lệnh sau:

`semanage user -l`

<img src="/Note/img/6.JPG">

Các user ở đây bao gồm:

 + guest_u: User này không truy nhập vào X Windows system hay netwoking và cũng không thể thực hiện lệnh su hoặc sudo

 + xguest_u: User này có quyền truy nhập vào GUI và network thông qua duy nhất browser

 + user_u: User này có quyền truy nhập vào GUI và network nhưng không sử dụng được su hoặc sudo

 + staff_u:  Giống user_u nhưng có thể sử dụng được su hoặc sudo

 + sysadmin_u: User này gần như có quyền truy nhập tất tả như root.

 + system_u: User này dành cho các system services và không được map vào các account thông thường.

Ứng dụng mớ lý thuyết trên, ta sẽ thực hiện một vài policy trên hệ thống như sau:

1.  Giới hạn sử dụng su hoặc sudo

Bằng cách map 1 user vào một SELinux user đã có sẵn, chẳng hạn admin  như ví dụ sau, ta đã hạn chế admin sử dụng lệnh su hoặc sudo

`semanage login -a -s user_u admin`

2.  Giới hạn quyền thực thi scripts


Với policy này thì ngoài việc map vào 1 SELinux user không có quyền chạy scripts, ta còn phải giới hạn quyền thực thi scripts trong thư mục của chính user đó

 `semanage login -a -s guest_u admin`
 
Kiểm tra thuộc tính sau:

 `getsebool allow_guest_exec_content`
 
Nếu là ON thì ta cần OFF ngay nó như sau:

 `setsebool allow_guest_exec_content off` 

3. Giới hạn truy nhập vào dịch vụ

Giả sử user admin có quyền dùng sudo để thực thi các tác vụ như stop/start services như httpd. Ta sẽ phải giới hạn việc truy cập này như sau:

`semanage login -a -s user_u admin`

Sau đó ta verify lại bằng cách chạy lệnh seinfo như sau:

`seinfo -uuser_u -x`

<img src="/Note/img/7.JPG">

Kết quả trên cho thấy roles user_u có thể access vào. Thêm 1 bước nữa để xem domain nào user_r được cho phép 

`seinfo -ruser_r -x `

<img src="/Note/img/8.JPG">

Ta xem role có thể access  vào domain httpd_t bằng lệnh sau:

 `seinfo -ruser_r -x | grep httpd_t`
 
Không giá trị nào trả về, điều đó có nghĩa là role user_r không được phép đi vào khu vực domain httpd_t và do đó nó không thể start các httpd process hay các daemon.
















