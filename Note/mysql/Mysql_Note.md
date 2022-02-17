### Tối ưu truy vấn cơ sở dữ liệu cần biết

```
* Cơ sở dữ liệu là nơi lưu trữ dữ liệu.
* Những thao tác tới cơ sở dữ liệu gồm truy vấn (select), thêm (insert), sửa (update), xoá (delete) dữ liệu. Khi dữ liệu nhỏ, tốc độ thực hiện gần như là tức thì, khi dữ liệu đủ lớn thì tốc độ thực hiện trở thành một trở ngại đối với dự án. Đôi khi bạn có thể mất hàng giờ để thực hiện một thao tác bất kỳ tới cơ sở dữ liệu.
```

### 1. Hạ tầng

- Do cơ sở dữ liệu lưu trữ trên ổ cứng nên việc nâng cấp hạ tầng như chuyển ổ cứng thường thành SSD, kết nối mạng giữa server và client phải đảm bảo, tăng bộ nhớ, chia nhỏ thành nhiều CSDL rồi thực hiện tổng hợp lại... Cách thức này chỉ phù hợp trong một thời gian vì hạ tầng luôn bị giới hạn chưa kể chi phí cũng đội lên rất nhiều.

### 2. Giới hạn kết quả trả về

- Chỉ trả về những trường được sử dụng (tránh sử dụng Select * ...). Hạn chế số bản ghi trả về.

### 3. Đánh chỉ mục (Index)

- Chỉ mục là bảng tra cứu đặc biệt mà Database Search Engine có thể sử dụng để tăng thời gian và hiệu suất truy vấn dữ liệu. Một lưu ý nhỏ là: index làm tăng hiệu năng của lệnh `SELECT` nhưng lại làm giảm hiệu năng của lệnh `INSERT`, `UPDATE` và `DELETE`. Chỉ nên index những trường có kiểu dữ liệu số. Những kiểu dữ liệu khác nếu không phải là đặc biệt, hoặc ko phải tìm kiếm nhiều thì ko nên index.

Ví dụ: bạn có bảng Users có trường `Username` là `nvarchar(200)`. Trong ứng dụng việc truy vấn theo `Username` là rất nhiều nên bạn có thể đánh `index` cho trường `Username` đó.

- Các trường được index không nên thao tác với một số phép toán phủ định như: “IS NULL”, “!=”, “NOT”, “NOT IN”, “NOT LIKE”... Vì vậy một trường được tạo ra và được xác định index thì nên là khác NULL hoặc có giá trị mặc định.

- Hạn chế sử dụng phép toán so sánh 2 lần như “>=”, “<=” với những trường đánh index (bản chất của phép toán này là phép toán OR).

Ex: 

`Select UserId, Username From Users Where Amount >= 18`

Câu lệnh trên tương ứng với câu lệnh sau:

`Select UserId, Username From Users Where Amount > 18 Or Amount = 18`

### 4. Sử dụng từ khoá Like phải hợp lý

Khi sử dụng `LIKE`, không nên sử dụng ký tự `%, *` đặt ở phía trước giá trị tìm kiếm.

`Select UserId, Username, FirstName From Users Where FirstName Like '%Hello';`

Bạn hãy thay thế bằng câu lệnh sau:

`Select UserId, Username, FirstName From Users Where FirstName Like 'Hello%';`

Hoặc trong trường hợp bắt buộc thì nên sử dụng FULL TEXT search như sau:

`Select UserId, Username, FirstName From Users Where CONTAINS(FirstName, 'Hello');`

### 5. Không nên sử dụng hàm thao tác trực tiếp đến các column

Chúng ta hãy xem ví dụ dưới đây - xác định danh sách user có tuổi lớn hơn 18:

`Select UserId, Username, DateOfBirth From Users Where DATEDIFF(YY, DateOfBirth, GETDATE()) > 18`

Hàm DATEDIFF đã tác động trực tiếp tới trường DateOfBirth. Chúng ta có thể thay thế thành câu lệnh như sau:

`Select UserId, Username, DateOfBirth From Users Where DateOfBirth < DATEADD(YY, -18, GETDATE())`

### 6. UNION vs UNION ALL

Khi sử dụng UNION, Mssql sẽ thực hiện sắp xếp, lọc và loại bỏ các bản ghi trùng. Sử dụng UNION không khác gì bạn đang sử dụng SELECT DISTINCT. Nếu chúng ta xác định việc hợp dữ liệu giữa các nguồn với nhau mà không có bản ghi trùng thì việc sử dụng UNION sẽ không hợp lý.

Trong trường hợp này, bạn nên sử dụng UNION ALL.
 
### 7. Distinct, Order By
Hạn chế sử dụng 2 từ khóa này. Hai thao tác này chiếm phần lớn thời gian truy vấn vì phải thực hiện lọc bản ghi trùng và sắp xếp các bản ghi.

### 8. EXISTS vs COUNT; IN vs EXISTS và IN vs BETWEEN

Khi bạn muốn xác định một bản ghi có tồn tại không hãy chọn EXISTS thay vì COUNT hoặc IN.

Giữa IN và BETWEEN hãy chọn BETWEEN.

### 9. Đếm số bản ghi

Trong trường hợp sử dụng SELECT COUNT(*) để xác định số bản ghi thì thay bằng `Select rows FROM sysindexes WHERE id = OBJECT_ID(‘table_name’)`.

### 10. Sử dụng store procedure (SP)
Đối với những thao tác được thực hiện thường xuyên và phức tạp, bạn nên sử dụng SP để giảm lượng dữ liệu truyền đến máy chủ (thay vì bạn phải gửi câu lệnh sql dài bạn chỉ cần gửi đi tên sp và danh sách tham số).

### 11. Tránh dùng CURSOR
Cursor không khác một vòng lặp thao tác tới từng bản ghi. Bản ghi đó sẽ bị lock cho tới khi được xử lý xong. Khi có thao tác tác động tới dữ liệu của cursor sẽ gây ra lỗi. Trong nhiều trường hợp sử dụng bảng temp để thay thế cho cursor. Nếu có thể hãy dùng biến table trong Mssql (từ phiên bản 2008 đến nay) thay thế cho bảng temp.

### 12. Tối ưu hóa bảng

Câu lệnh OPTIMIZE TABLE thực hiện một số chức năng như chống phân mảnh, sắp xếp các chỉ mục của các bảng. OPTIMIZE TABLE thực hiện các tác vụ sau:

- Tạo một bảng tạm
- Xóa bản gốc sau khi tối ưu hóa nó (Thu nhỏ các trang dữ liệu, Thu hẹp các trang chỉ mục, Tính toán thống kê chỉ mục mới)
- Cuối cùng, đổi tên bảng tạm thành tên ban đầu.

query_cache_type: Xác định chế độ hoạt động của query cache
0 hoặc OFF: Không lưu trữ các kết quả hoặc truy xuất các kết quả được lưu trữ
1 hoặc ON: Cho phép lưu trữ trừ các câu lệnh bắt đầu với SELECT SQL_NO_CACHE.
2 hoặc DEMAND: Chỉ lưu trữ các kết quả bắt đầu SELECT SQL_CACHE.
query_cache_limit: Xác định kích thước lớn nhất cho một kết quả có thể lưu trong cache
query_cache_size: Xác định dung lượng bộ nhớ phân phối cho các Query cache. Giá trị ngầm định của biến là 0, có nghĩa query caching không được bật
VD: Đặt lại lại giá trị cho biến
```
mysql> SET GLOBAL query_cache_size = 40000;

Query OK, 0 rows affected, 1 warning (0.00 sec)
```




