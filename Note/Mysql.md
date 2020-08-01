Lệnh trong MySQL
Kết nối vào MySQL 
file mysql.exe trong thư mục Bin của MySQL trên Windows hoặc file mysql trong thư mục  bin của MySQL trên Linux.  
              mysql -h 192.168.0.1 -u test -ptest test_db
Lệnh trên sẽ kết nối vào MySQL ở địa chỉ 192.168.0.1 với username là test, mật mã là test và sử dụng CSDL có tên là  test_db. Nếu không có tham số -h 192.168.0.1, mysql sẽ mặc định kết  nối vào server localhost.
Nếu bạn không muốn cung cấp mật mã trong câu lệnh kết nối thì bạn chỉ cần cung cấp tham số -p, mysql sẽ nhắc bạn nhập vào mật mã sau. 
VD:   mysql -h localhost -u myuser -p mydb
Nếu bạn không cung cấp tên của CSDL cần sử dụng, thì mặc định sau khi kết nối  sẽ không có CSDL nào được mở ra để bạn sử dụng cả. VD:
      mysql -u root -p
Sau khi kết nối thành công, để thoát khỏi chế độ dòng lệnh của mysql và trở về hệ điều hành, bạn có thể nhấn Ctrl-C, hoặc \q và Enter.
Chọn CSDL để làm việc  bằng câu lệnh USE tên_CSDL:
USE mysql; (cuối câu lệnh SQL bạn nhớ thêm dấu chấm phảy ;)
Liệt kê danh sách các CSDL trong hệ thống bằng lệnh:
SHOW DATABASES;
Hiển thị thông tin về table
Sau khi kết nối và chọn CSDL để làm việc xong, bạn có thể liệt kê danh sách các table trong CSDL bằng lệnh:  SHOW TABLES;
Bạn cũng có thể liệt kê thông tin chi tiết về table bằng lệnh SHOW TABLE STATUS.
Lệnh SHOW TABLE STATUS; không tham số sẽ liệt kê thông tin về tất cả các table có trong CSDL hiện tại.
Lệnh SHOW TABLE STATUS FROM db_name; sẽ liệt kê thông tin về tất cả các table trong CSDL có tên là db_name.
Lệnh SHOW TABLE STATUS FROM db_name LIKE 'tbl_name'; sẽ liệt kê thông tin về table có tên là tbl_name trong CSDL db_name;
và lệnh SHOW TABLE STATUS LIKE 'tbl_name'; sẽ liệt kê thông tin về table có tên là tbl_name trong CSDL hiện tại.
Lệnh SHOW TABLE STATUS sẽ trả về các thông tin sau:
Name: tên của table.
Engine: kiểu của table (VD: InnoDB, MYISAM...).
Version: phiên bản của table.
Row_format: kiểu dòng của table (Fixed, Dynamic hoặc Compressed).
Rows: số lượng dòng có trong table (lưu ý: với một số kiểu table  như InnoDB số lượng dòng chỉ là ước lượng chứ không chính xác, bạn cần dùng  lệnh SELECT COUNT(*) FROM tbl_name; để lấy chính xác số 
dòng).
Avg_row_length: kích thước/độ dài trung bình của một dòng trong  table.
Data_length: kích thước của file lưu trữ table.
Max_data_length: kích thước tối đa của file lưu trữ table.
Index_length: kích thước của file index.
Data_free: dung lượng được cấp phát nhưng chưa được sử dụng trong file.
Auto_increment: giá trị tiếp theo của cột có kiểu AUTO_INCREMENT trong table.
Create_time: thời điểm table được tạo.
Update_time: thời điểm table được cập nhật lần cuối.
Check_time: thời điểm table được kiểm tra lần cuối.
Collation: lưu trữ thông tin về charset sử dụng trong table.
Checksum: giá trị kiểm tra checksum của table.
Create_options: lưu trữ các thông tin của lệnh CREATE TABLE khi tạo table.
Comment: thông tin chú thích về table.
Và cuối cùng, với lệnh SHOW CREATE TABLE tên_table, MySQL sẽ trả về cho bạn câu lệnh SQL dùng để tạo ra table đó. VD câu lệnh: SHOW CREATE TABLE db;
Tạo CSDL
Để tạo CSDL mới, lệnh CREATE DATABASE: 
CREATE DATABASE db_name; lệnh sẽ tạo 1 CSDL mới có tên là db_name. Bạn cũng có thể xoá  bỏ một CSDL với lệnh DROP DATABASE:
DROP DATABASE db_name;
Tuy nhiên, CSDL mà đã bị xoá rồi thì không thể khôi phục lại được, cho nên bạn hãy cẩn thận khi dùng lệnh DROP DATABASE.
Và bạn cũng có thể xem lại cú pháp câu lệnh CREATE DATABASE của 1 CSDL đã có sẵn bằng lệnh SHOW CREATE DATABASE:
SHOW CREATE DATABASE db_name;
Tạo user account
Để tạo account mới và gán quyền truy cập vào MySQL server cho account đó, ta dùng lệnh GRANT. Cú pháp đơn giản của lệnh GRANT như sau:
     GRANT quyền ON tên_csdl TO tên_user IDENTIFIED BY 'mật_mã';
Quyền có dạng như sau:
ALL PRIVILEGES: tất cả mọi quyền.
Các quyền sau đây có thể được kết hợp với nhau, phân cách bằng dấu phảy (,):
ALTER: cho phép user sử dụng lệnh ALTER TABLE.
CREATE: cho phép user sử dụng lệnh CREATE TABLE.
CREATE TEMPORARY TABLES: cho phép user sử dụng lệnh CREATE TEMPORARY TABLE.
CREATE VIEW: cho phép user sử dụng lệnh CREATE VIEW.
DELETE: cho phép user sử dụng lệnh DELETE.
DROP: cho phép user sử dụng lệnh DROP TABLE.
FILE: cho phép user sử dụng lệnh SELECT ... INTO OUTFILE và LOAD DATE INFILE.
INDEX: cho phép user sử dụng lệnh CREATE INDEX và DROP INDEX.
INSERT: cho phép user sử dụng lệnh INSERT.
LOCK TABLES: cho phép user sử dụng lệnh LOCK 
TABLES trên những table nào user có quyền SELECT.
PROCESS: cho phép user sử dụng lệnh SHOW FULL PROCESSLIST.
RELOAD: cho phép user sử dụng lệnh FLUSH.
SELECT: cho phép user sử dụng lệnh SELECT.
SHOW DATABASES: khi user sử dụng lệnh SHOW 
DATABASES, danh sach của toàn bộ các CSDL trong hệ thống.
SHOW VIEW: cho phép user sử dụng lệnh SHOW CREATE VIEW.
UPDATE: cho phép user sử dụng lệnh UPDATE.
Ngoài ra một quyền đặt biệc là USAGE sẽ gán toàn bộ các quyền của user là "không được phép". Quyền này thường được gán cho các account không có quyền global trên hệ thống. Thường sau khi gán quyền USAGE, quản trị viên của server sẽ tiếp tục gán một vài quyền nhất định cho account trên một số CSDL nhất định.

 Tên_CSDL có dạng như sau:
*.*: toàn bộ các CSDL và table trên hệ thống.
db_name.*: giới hạn quyền trên CSDL có tên là 
db_name và các table của CSDL này.
db_name.tbl_name: giới hạn quyền trên table có 
tên tbl_name trên CSDL có tên db_name.

Tên_user có dạng như sau:
'nbthanh'@'localhost': account với username  nbthanh có thể kết nối từ localhost.
'nbthanh'@'%': account với username nbthanh có thể kết nối từ bất kỳ nơi nào trừ localhost.
Nếu trong câu lệnh GRANT không có phần IDENTIFIED BY 'mật_mã' thì 
account sẽ được tạo với mật mã là rỗng.
Ví dụ:
+ GRANT ALL PRIVILEGES ON *.* TO 'nbthanh'@'localhost' IDENTIFIED BY 'my_password'; Lệnh trên sẽ tạo 1 account có username là nbthanh với mật mã là my_password, được phép kết nối từ localhost và có mọi quyền trên tất cả các CSDL có trên hệ thống.
+ GRANT SELECT ON *.* TO 'nbthanh'@'%' IDENTIFIED BY 'my_password2'; Lệnh trên sẽ tạo 1 account có username là nbthanh với mật mã là my_password2, được phép kết nối từ bất cứ nơi đâu ngoại trừ localhost, nhưng chỉ có quyền đọc (SELECT) trên tất cả các CSDL có trên hệ 
thống.
+ Đôi khi bạn sẽ có nhu cầu đổi mật mã cho 1 account mà không muốn thay đổi lại các quyền hiện có của account đó, MySQL cung cấp lệnh SET PASSWORD để thực hiện thao tác này:
+ SET PASSWORD FOR 'nbthanh'@'localhost' = PASSWORD('new_password'); Lệnh trên sẽ đổi mật mã của account nbthanh, kết nối từ localhost, thành new_password.
+ SET PASSWORD FOR 'nbthanh'@'%' = PASSWORD('new_password2');Lệnh trên sẽ đổi mật mã của account nbthanh, kết nối từ bất cứ nơi đâu ngoại trừ localhost, thành new_password2.
+ Và cuối cùng, để xoá account ra khỏi hệ thống, MySQL cung cấp cho ta lệnh DROP USER:
+ DROP USER 'nbthanh'@'localhost'; Xoá account có username là nbthanh, kết nối từ localhost, ra khỏi hệ thống (lệnh này không xoá các account khác như 'nbthanh'@'%').
+ DROP USER 'nbthanh'@'%'; Xoá account có username là nbthanh, kết nối từ bất cứ nơi đâu ngoài trừ localhost, ra khỏi hệ thống (lệnh này không xoá các account khác như 'nbthanh'@'localhost').
Lưu ý: Đôi khi bạn cần phải thực hiện lệnh FLUSH PRIVILEGES;  sau khi thực hiện các thao tác thêm/xoá/thay đổi account thì các thao tác mới có  hiệu lực.