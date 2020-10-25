### Install NFS

```
NFS Server IP address: 192.168.30.128
NFS Client IP address: 192.168.30.129
```
### Server NFS

- Để cài đặt NFS Server các bạn cài package nfs-utils

`sudo yum install nfs-utils -y`

– Tạo thư mục chia sẻ tài nguyên trên server

```
mkdir -p /var/nfs/share
chown nfsnobody:nfsnobody /var/nfs/share
chmod -R 755 /var/nfs/share
```

– Thiết lập quyền truy cập (client truy cập được từ một IP cụ thể của Client), cập nhật quyền này vào file /etc/exports

```
/var/nfs/share    192.168.30.129(rw,sync,no_root_squash,no_all_squash)
/home            192.168.30.129(rw,sync,no_root_squash,no_all_squash)
```

- Start nfs

```
systemctl start rpcbind nfs-server
systemctl enable rpcbind nfs-server
```

- Test port

`rpcinfo -p`

### Client NFS

`yum install nfs-utils`

```
systemctl start rpcbind nfs-server
systemctl enable rpcbind nfs-server
```


```
mkdir -p /mnt/nfs/share-data
mount -t nfs 192.168.30.128:/share-data /mnt/nfs/share-data/
```


Để NFS share luôn được mount kể cả khi khởi động lại, ta làm như sau:
```
vi /etc/fstab
192.168.30.128:/home    /mnt/nfs/home   nfs defaults 0 0
192.168.30.128:/var/nfs/share    /mnt/nfs/var/share-data   nfs defaults 0 0
```


- Các file cấu hình quan trọng của NFS


```
/etc/exports : Đây là file config chính của NFS, chứa thông tin danh sách các file và thư mục chia sẻ trên NFS Server.
/etc/fstab : Để tự động mount một thư mục NFS trên hệ thống của bạn trong trượng reboot.
/etc/sysconfig/nfs : File config của NFS để quản lý port đang lắng nghe của rpc và các service khác.
```

- Một số command hay dùng

```
showmount -e : Hiển thị thư mục Share trên hệ thống của bạn
showmount -e <server-ip or hostname>: Hiển thị danh sách thư mục share trên một Remote Server
showmount -d : Liệt kê danh sách các thư mục còn
exportfs -v : Hiển thị danh sách các file chia sẻ và các options trên server
exportfs -a : Exports toàn bộ thư mục share trong /etc/exports
exportfs -u : Unexports toàn bộ thư mục share trong /etc/exports
exportfs -r : Refresh sau khi đã chỉnh sửa /etc/exports
```


