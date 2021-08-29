### Note for work

```
if condition;
then
    commands;
fi
```


```
if condition;
then
    commands_01;
else
    commands_02;
fi
```

```
if condition;
then
    commands;
else if condition;
then
    commands;
else
    commands;
fi
```

```
if [ condition ];
then
    commands;
fi
```

```
if [ condition ];
then
    commands;
else
    commands;
fi
```

```
-eq => Bằng nhau (Equal)
-ne => Không bằng nhau (Not equal)
-lt => Nhỏ hơn (Less than)
-gt => Lớn hơn (Greater than)
-le => Nhỏ hơn hoặc bằng (Less or equal)
-ge => Lớn hơn hoặc bằng (Greater or equal)
```


* So sánh chuỗi

```
Sử dụng các cấu trúc sau để thực hiện việc so sánh chuỗi trong Bash shell

[[ $str1 = $str2 ]] hoặc [[ $str1 == $str2 ]]
Trả về TRUE nếu 2 biến str1 và str2 có nội dung giống nhau. FALSE nếu ngược lại.

[[ $str1 != $str2 ]]
Trả về TRUE nếu 2 biến str1 và str2 có nội dung giống nhau. FALSE nếu ngược lại.

[[ $str1 > $str2 ]]
Trả về TRUE nếu srt1 lớn hơn str2 tính theo bảng chữ cái. FALSE nếu ngược lại.
```

```
[[ $str1 < $str2 ]]
Trả về TRUE nếu srt1 nhỏ hơn str2 tính theo bảng chữ cái. FALSE nếu ngược lại.

[[ -z $str1 ]]
Trả về TRUE nếu $str1 là 1 chuỗi rỗng. FALSE nếu ngược lại

[[ -n $str1 ]]
Trả về TRUE nếu $str1 là 1 chuỗi khác rỗng. FALSE nếu ngược lại.
```


* Kiểm tra hệ thống tập tin

```
Sử dụng các phép toán kiểm tra hệ thống tập tin dưới đây bên trong cấu trúc kiểm tra [] ở phần 1.

[ -f $file_var]
Trả về TRUE nếu file_var là 1 tập tin.

[ -x $var ]
Trả về TRUE nếu var là tập tin và có quyền thực thi (executable)

[ -d $var ]
Trả về TRUE nếu var là 1 thư mục.

[ -e $var ]
Trả về TRUE nếu var tồn tại

[ -w $var ]
Trả về TRUE nếu var là 1 tập tin và có quyền ghi (writable)

[ -r $var ]
Trả về TRUE nếu var là 1 tập tin và có quyền đọc (readable)

[ -L $var ]
Trả về TRUE nếu var là 1 liên kết mềm (symlink)
```

```
str1="Not empty "
str2=""
if [[ -n $str1 ]] && [[ -z $str2 ]];
then
    echo TRUE
fi
```

```
str1="Not empty "
if [[ -n $str1 ]] || [[ -z $str1 ]];
then
    echo TRUE
fi
```





























