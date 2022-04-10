### Crontab

* Command line

```
crontab -e: tạo hoặc chỉnh sửa file crontab 
crontab -l: hiển thị file crontab 
crontab -r: xóa file crontab
```
* Cài đặt crontab

```
yum install cronie -y
service crond start
chkconfig crond on
```

* Cấu trúc của crontab

```
*     *     *     *     *     command to be executed
-     -     -     -     -
|     |     |     |     |
|     |     |     |     +----- day of week (0 - 6) (Sunday=0)
|     |     |     +------- month (1 - 12)
|     |     +--------- day of month (1 - 31)
|     +----------- hour (0 - 23)
+------------- min (0 - 59)
``` 
Ex:

```
0,30 * * * * command  # Chạy script 30 phút 1 lần:
0,15,30,45 * * * * command # Chạy script 15 phút 1 lần
0 3 * * * command # Chạy script vào 3 giờ sáng mỗi ngày
* * * * * sh /etc/backup.sh # chạy mỗi phút một lần
0 0 * * * sh /etc/backup.sh # chạy 24h một lần
>/dev/null 2>&1 # Disable email
```














