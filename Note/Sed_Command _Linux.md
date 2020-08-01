## Tìm và thay thế chuỗi ký tự trong một file sử dụng Sed Command trong Linux


Làm cách nào bạn có thể tìm và thay thế một từ, một chuỗi ký tự hay một đoạn text trong một file text trên Linux? Rất đơn giản bằng việc sử dụng sed command, một lệnh cơ bản có sẵn trong các hệ điều hành Linux.

<img src="/Note/img/linux1.png">

### Giới thiệu Sed Command
sed – Unix-like operating system command

+ Viết tắt của “Stream editor”
+ Chức năng: Trình chỉnh sửa luồng để lọc và chuyển đổi văn bản.
+ Cú pháp: sed [OPTION]… {script-only-if-no-other-script} [input-file]…

```
// Tìm và thay thế text
sed -i 's/original/new/g' file.txt

```
### Giải thích:

* sed = Stream EDitor

* -i = in-place (thay thế nội dung ngay trong file gốc)

* Nội dung chuỗi command:

<ul> s = Lệnh thay thế

 <li> original = một từ, một biểu thức chính quy mô tả về đoạn text mà bạn muốn thay thế. </li> 

 <li> new = Đoạn text sẽ thay thế vào. </li>

 <li> g = global (Thay thế tất cả các trường hợp tìm thấy) </li>

 <li> file.txt = Tên file </li>
</ul>







