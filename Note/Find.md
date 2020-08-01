### find 

Các tùy chọn hay dùng: 

 

-name: Tìm kiếm theo tên file/folder 

-type: Theo kiểu, f: Folder; d: Directory 

-mtime N: Thời gian tạo/chỉnh sửa. N: là số ngày. 

-exec: Thực thi gì đó với các file tìm được 

-perm: Tìm theo quyền truy cập (RWX) 

Tham khảo thêm: man find 

 

 

Tìm file theo dung lượng lớn hơn 100M 

find . -type f -size +100M -exec ls -sh {} \; 

Chú ý: Nếu muốn tìm dung lượng nhỏ hơn -100M 

 

Tìm file nào đó trong thư mục 

find ./folder -print | grep -e '<tên-file>' 
 

Tìm file nào đó có quyền thực thi 

find ./folder -executable 
 

Tìm các file mới được tạo sau 1 ngày 

find / -atime -1 
 

Tìm các file có dung lượng > 100M và mới tạo sau 1 ngày 

find / -type f -size +100M -atime -1 -exec du -sh {} \; 
 

Xóa các file được tạo sau 15 ngày 

find /data/logs/ -type f -mtime +15 -exec rm -rf {} \; 
 

Tìm các file được tạo trong vòng 15 ngày trở lại 

find /data/logs/ -type f -mtime -15 -exec ls -l {} \; 
 

Đếm số file trong thư mục 

find . -type f | wc -l 
 

Đếm sub-directory trong thư mục 

find . -type d | wc -l 
 

Đếm tổng số file trong thư mục 

find . -print| wc -l 
 

Tìm các file có quyền khác với 644 và thay đổi 

find /tmp -type f ! -perm 644 -exec chmod 644 {} \; 



 

https://cloudcraft.info/lap-trinh-bash-shell-co-ban-phan-1/  

 
 

https://viblo.asia/p/gioi-thieu-cac-cong-cu-so-sanh-files-diff-danh-cho-linuxubuntu-tot-nhat-3P0lPO0pZox  
