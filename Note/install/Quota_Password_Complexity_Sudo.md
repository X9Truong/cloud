### 1. QUOTA

Quota cho phép điều chỉnh dung lượng đĩa được sử dụng trên Linux filesystem đến từng user và group. Mặc định trong Centos 7, quota không hoạt động, vì vị ta phải active nó trước. Câu lệnh kiểm tra như sau:

`mount | grep ' / '`

Nếu kết quả trả về như này :

`/dev/sda2 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)`

Có nghĩa là chưa active rất rõ ràng rồi. Vì thế ta cần điều chỉnh lại bằng cách thêm vào `file /etc/default/grub` ở thông số `GRUB_CMDLINE_LINUX` dòng sau` ” rootflags=usrquota,grpquota “`

Khi đó ta sẽ có được như sau:

`GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/swap vconsole.font=latarcyrheb-sun16 rd.lvm.lv=centos/root crashkernel=auto vconsole.keymap=us rhgb quiet rootflags=usrquota,grpquota"`

Một khi đã đụng chạm đến GRUB, ta phải rất cẩn thận và rebuid lại như sau để có thể active những thông số mới:

```
cp /boot/grub2/grub.cfg /boot/grub2/grub.cfg.original

grub2-mkconfig -o /boot/grub2/grub.cfg

reboot
```

Sau đó check lại bằng lệnh `mount | grep ' / '`ta được như sau là tạm ổn:

`/dev/sda2 on / type xfs (rw,relatime,seclabel,attr2,inode64,usrquota,grpquota)`

Như vậy quota đã được active, giờ ta cần có một tool để phân bổ quota cho các user, group. Đây là lúc ta cần cài thêm gói quota:

`yum -y install quota`

Kiểm tra lại :

`repquota -as`

<img src="/Note/img/1.jpg">

Như hình trên ta thấy có 2 cách để set quota cho 1 user hay Group:

+ Set theo số lượng file

Cả 2 cách đều sử dụng soft limit và hard limit. Trong đó soft limit là giới hạn cảnh báo khi chạm đến, còn hard limit là giới hạn không được phép vượt qua.

Để thay đổi quota cho 1 user hoặc group ta dùng lệnh sau:


`edquota -u username/group`

Sau đó thay đổi các thông số như đã mô tả ở trên.

Nếu muốn set quota trên một phân vùng riêng biệt khác ngoài /, chẳng hạn phân vùng /var, ta add thêm usrquota và grpquota vào file /etc/fstab như sau:

```
UUID=7aa4a465-953b-4f4e-b60d-a2157699b0f8 /var ext4 defaults,usrquota,grpquota 1 1
UUID=54712ad1-d2a9-4ead-8d95-4d9de5828a58 /tmp ext4 defaults 1 2
UUID=8ba72a4b-11b8-40e5-89ac-4d0eb2c9c809 swap swap defaults 0 0
```

Sau đó remount lại filesystem này là hoạt động:

```
mount -o remount /var
quotacheck -avugm
quotaon -avug
```


Đến đây, ta có thể thấy việc set lại quota cho các user trên hệ thống Linux Centos hoàn toàn không có gì phức tạp, các công cụ quản trị hosting như Cpanel hay DirectAdmin cũng chỉ chạy những lệnh như trên mà không hề phải có kỹ thuật gì cao siêu, đặc biệt.


### 2. PASSWORD COMPLEXITY

Password Complexity là việc cấu hình đặt các mật khẩu theo các quy tắc phức tạp để password trở nên mạnh hơn nhờ quy đinh về chữ hoa, chữ thường, ký tự đăc biệt ….hoặc cấu hình sao cho password chỉ được sử dụng lại một vài lần, mục đích làm cho mật khẩu luôn ở trạng thái an toàn nhất có thể. Password Complexity sử dụng cơ chế xác thực PAM module. Ta có các thành phần như sau:


```
Buộc sử dụng chữ in hoa:                  ucredit=-X, X là số lượng chữ in hoa trong password.
Buộc sử dụng chữ thường:                lcredit=-X, X là số lượng chữ thường trong password.
Buộc sử dụng số:                               dcredit=-X, X là số lượng sốtrong password
Buộc sử dụng ký tự đăc biệt:             ocredit=-X, X là số lượng ký tự trong password. Ví dụ cụ thể như sau:
password requisite pam_cracklib.so try_first_pass retry=3 type=ucredit=-2 lcredit=-2 dcredit=-2 ocredit=-2
Không cho phép sử dụng lại password: remember=X, X là số lần password đã bị từ chối , ví du:
password sufficient pam_unix.so sha512 shadow nullok try_first_pass use_authtok remember=5
```

Ta sẽ thiết lập một rule như sau: user login failed sau 3 lần sẽ bị locked và chỉ system admin mới có thể unlock.

Trong file /etc/pam.d/system-auth:

`account required pam_tally2.so`

Trong file  /etc/pam.d/login:

`auth required pam_tally2.so file=/var/log/tallylog deny=3 no_magic_root unlock_time=300`

Để xem login failure, dùng lệnh sau:

`pam_tally2 –user=<User Name>`

Để reset login failure và enable user được phép login trở lại: 

`pam_tally2 –user=<User Name> --reset`


### 3. SUDOERS

Phân chia quyền hạn của các user là một trong những đặc điểm chính của Linux. Root là account cao nhất của hệ thống, các account khác luôn bị giới hạn trong phạm vi của riêng nó. Chính vì thế, sudo ra đời cho phép các account thông thường có thể thực thi các lệnh như root thực hiện. “sudo” khác hoàn toàn “su” khi nó không yêu cầu password, trong khi “su” thì có yêu cầu.

Cấu hình của sudo nằm trong file /etc/sudoers hoặc dùng trực tiếp lệnh visudo. Ta thấy trong file này có những điều rất thú vị như sau:

```
Alias cho một set các lệnh:
Cmnd_Alias NETWORKING = /sbin/route, /sbin/ifconfig, /bin/ping, /sbin/dhclient, /usr/bin/net, /sbin/iptables, /usr/bin/rfcomm, /usr/bin/wvdial, /sbin/iwconfig, /sbin/mii-tool
Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig
```

Sử dụng các alias trên để gán quyền cho một group, chẳng hạn group netadmin như sau sẽ có toàn quyền sử dụng các lệnh trong NETWORKING

`%netadmin ALL = NETWORKING`

Nếu muốn 1 user có thể access tất cả các command:

`admin ALL=(ALL) ALL`

Hoặc tạo các User trong từng Group:


```
User_Alias GROUPONE = admin1, admin11, admin 12
User_Alias GROUPTWO = admin2, admin22, admin 23
User_Alias GROUPTHREE = admin3, admin33, admin34
```


Nếu muốn user thực hiện quyền root mà không phải gõ password:

`GROUPONE ALL = NOPASSWD: /usr/bin/updatedb`

Hoặc phức tạp hơn thì:

`GROUPTWO ALL = NOPASSWD: /usr/bin/updatedb, PASSWD: /bin/kill`

Trong trường hợp này, user có thể sử dụng lệnh updatedb mà không cần password nhưng với lệnh kill thì vẫn cần.








