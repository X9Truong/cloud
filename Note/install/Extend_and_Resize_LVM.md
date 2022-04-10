## How to Extend and Resize LVM Partition in Linux

Logical Volume Manager (LVM) cho phép Linux kernel quản lý các large  disk lớn một cách hiệu quả. Điều này cho phép người dùng tạo các phân vùng từ nhiều hơn một đĩa và cho phép họ mở rộng filesystem size online trong vòng vài giây.

Trong bài viết này, chúng ta sẽ thấy, làm thế nào chúng ta có thể mở rộng và thay đổi kích thước phân vùng LVM mà không mất dữ liệu.

Nếu bạn đang sử dụng LVM trên Linux, đây là các bước để mở rộng phân vùng LVM online mà không mất dữ liệu.

Lưu ý - các lệnh này phải được chạy với quyền root.

### 1. Extend LVM using existing disk

Trong trường hợp này, chúng ta có một phân vùng chưa sử dụng trên disk lưu trữ hiện có và chúng ta muốn sử dụng nó để mở rộng một phân vùng LVM khác.

* Kiểm tra thông tin volume groups 

Đầu tiên, chúng ta sẽ sử dụng lệnh vgdisplay để xem thông tin volume groups.

```
[root@devstack ~]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <99.00 GiB
  PE Size               4.00 MiB
  Total PE              25343
  Alloc PE / Size       25342 / 98.99 GiB
  Free  PE / Size       1 / 4.00 GiB
  VG UUID               LeB4JO-LbOJ-1Ej2-TYOz-A00U-O1SN-vA7Q5P

[root@devstack ~]#
```
Chúng ta có thể thấy dung lượng còn trống `Free  PE / Size       1 / 4.00 GiB` 

Để xem thông tin chi tiết về volume groups thêm option `-v` vào sau lệnh `vgdisplay`

`vgdisplay -v`

```
[root@devstack ~]# vgdisplay -v
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <99.00 GiB
  PE Size               4.00 MiB
  Total PE              25343
  Alloc PE / Size       25342 / 98.99 GiB
  Free  PE / Size       1 / 4.00 GiB
  VG UUID               LeB4JO-LbOJ-1Ej2-TYOz-A00U-O1SN-vA7Q5P
```
* Extend storage

/dev/





