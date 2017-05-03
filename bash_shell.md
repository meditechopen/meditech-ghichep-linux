# Bash shell programming

- Shell là trình biên dịch của Linux, nó cung cấp giao diện cho người dùng. Mỗi lệnh bạn gõ ra sẽ được Shell diễn dịch rồi chuyển tới nhân Linux. Nói một cách dễ hiểu Shell là bộ diễn dịch ngôn ngữ lệnh. Ngoài ra nó cũng được dùng để chạy các scripts ngay cả với những phiên làm việc không có cửa sổ giao diện, như thể các câu lệnh đã được gõ vào trực tiếp.

``` sh
#!/bin/bash
find /usr/lib -name "*.c" -ls
```

- Câu lệnh đầu tiên của 1 script đó là `#!/bin/bash` , chứa đường dẫn đầy đủ của trình biên dịch được sử dụng trong file. Trình biên dịch này có nhiệm vụ thực hiện những câu lệnh được đưa ra trong script. Các trình biên dịch thường được sử dụng bao gồm:

``` sh
/usr/bin/perl
/bin/bash
/bin/csh
/bin/tcsh
/bin/ksh
/usr/bin/python
/bin/sh
```

- Việc tạo các script không chỉ giới hạn trong việc dành cho trình biên dịch shell. Nó còn có thể được sử dụng trong các scripts của Python.

``` sh
# ll script
-rwxr--r--. 1 root root 55 Mar  3 15:22 script
# cat script
#!/usr/bin/python
print "Welcome to the Python script"
# ./script
Welcome to the Python script
```

- Các scripts cũng có thể tương tác với người dùng.

```sh 
# cat script.sh
#!/bin/bash
   # Interactive reading of variables
   echo "ENTER YOUR NAME"
   read sname
   # Display of variable values
   echo "WELCOME "$sname"!"
# ./script.sh
ENTER YOUR NAME
Adriano
WELCOME Adriano!
```

- Tất cả các scripts đều trả về một giá trị khi kết thúc việc thực hiện các lệnh. Giá trị trả về này có thể được thiết lập thành câu lệnh `exit`. Giá trị trả về cho phép tiến trình thực hiện giám sát trạng thái thoát ra của các tiến trình khác theo mối quan hệ cha-con. Điều này sẽ giúp xác định tiến trình đã kết thúc như thế nào và thực hiện các hành động cần thiết, phụ thuộc vào việc thành công hay thất bại của tiến trình. Theo quy ước, giá trị báo hiệu thành công là `0` và thất bại là 1 con số nào đó khác 0. Giá trị trả về sẽ được lưu tại biến `$?`

``` sh
# cat names.txt
01 Mario Rossi
02 Antonio Esposito
03 Michele Laforca
04 Antonio Esposito
# echo $?
0
# cat names
cat: names: No such file or directory
# echo $?
1
```

## Cú pháp cơ bản

- Các script yêu cầu người dùng phải tuân theo các cú pháp ngôn ngữ chuẩn. Các quy tắc mô tả làm thế nào để định nghĩa các biến và làm sao để xây dựng câu lệnh theo đúng format. Bảng dưới đây sẽ liệt kê một vài kí tự đặc biệt được sử dụng trong bash scripts:

| Character | Description                              |
| --------- | ---------------------------------------- |
| #         | Được dùng để add comment, trừ trường hợp dùng \#, hoặc #! khi bắt đầu một script |
| \\        | Dùng ở cuối dòng để xác định việc tiếp tục ở dòng tiếp theo |
| ;         | Dùng để biên dịch những lệnh tiếp theo   |
| $         | Dùng để chỉ ra biến                      |

- Đối với những dòng có nhiều câu lệnh, kí tự `;` sẽ được dùng để tách những câu lệnh này ra và chúng sẽ được chạy theo thứ tự giống như khi chúng được  nhập thành nhiều dòng khác nhau.
- Ba câu lệnh dưới đây sẽ được thực hiện ngay cả khi những câu lệnh trước đó thất bại:

``` sh
$ make ; make install ; make clean
```

- Trong trường hợp bạn muốn hủy bỏ các câu lệnh phía sau nếu nó thất bại, hãy sử dụng `&&` :

``` sh
$ make && make install && make clean
```

- Nếu bạn muốn câu lệnh phía sau chỉ chạy được nếu câu lệnh trước đó thành công thì hãy sử dụng `||` :

``` sh
$ cat file1 || cat file2 || cat file3
```

## Hàm functions

- Function là một khối mã lệnh thực thi nhiều thao tác khác nhau. Nó rất hữu ích cho việc thực hiện các tiến trình lặp đi lặp lại nhiều lần với các biến đầu vào khác nhau. Functions cũng được gọi là một chương trình con. Việc sử dụng function trong script bao gồm 2 bước:

1. Khai báo function
2. Gọi function

- Khi khai báo function, người dùng phải đặt tên để sau này gọi function theo tên đó :

``` sh 
function_name () {
       command...
    }
```

- Ví dụ một function hiển thị ra màn hình :

``` sh
display () {
       echo "This is a sample function"
    }
```

- Function có thể chứa rất nhiều câu lệnh. Một khi đã được khai báo, nó có thể được gọi ra rất nhiều lần sau này khi cần thiết. Ví dụ dưới đây sẽ chỉ ra làm thế nào để đưa các tham số vào trong một function. Các tham số theo thứ tự sẽ được thay thế bằng `$1, $2, $3, ..., $n` . Tên của script là `$0` . Tất cả các tham số là `$*` , tổng số dòng của các argument là `$#`

``` sh 
# cat script.sh
#!/bin/bash
echo The name of this program is: $0
echo The first argument passed from the command line is: $1
echo The second argument passed from the command line is: $2
echo The third argument passed from the command line is: $3
echo All of the arguments passed from the command line are : $*
echo All done with $0
exit 0
#
# ./script.sh A B C
The name of this program is: ./script.sh
The first argument passed from the command line is: A
The second argument passed from the command line is: B
The third argument passed from the command line is: C
All of the arguments passed from the command line are : A B C
All done with ./script.sh
```

## Lệnh thay thế

- Bạn đôi khi sẽ phải dùng kết quả của một câu lệnh nào đó trong một câu lệnh khác. Có 2 cách để làm được điều này:

1. Kết thúc câu lệnh inner bằng (`)
2. Kết thúc câu lệnh inner bằng $()

- Bất kể là phương thức nào thì câu lệnh inner cũng sẽ được chạy trong một môi trường shell mới. Đầu ra của shell sẽ được chèn vào nơi mà bạn thực hiện lệnh thay thế. Hầu hết các lệnh đều thực hiện được theo cách này. Lưu ý rằng phương pháp thứ 2 sẽ cho phép thực hiện các lệnh lồng nhau.

``` sh 
# cat ./count.sh
#!/bin/bash
echo "The " $1 " contains " $(wc -l < $1) " lines."
echo $?
# ./count.sh /var/log/messages
The  /var/log/messages  contains  114  lines.
0
```

## Câu lệnh if

- Câu lệnh if là cấu trúc cơ bản rất hữu dụng mà hầu hết các ngôn ngữ lập trình đều có. Khi sử dụng câu lệnh if, hành động tiếp theo được thực hiện sẽ phụ thuộc vào giá trị của một chuỗi các điều kiện cụ thể ví dụ như:

*. So sánh số hoặc chuỗi *. *.Giá trị trả về (0 là thành công) *. *. Sự tồn tại và quyền của file *. 

Cú pháp của câu lệnh if là :

`if TEST-COMMANDS; then CONSEQUENT-COMMANDS; fi`

Đưới đây là cú pháp hay được sử dụng:

``` sh
if condition
then
       statements
else
       statements
fi
``` 

Chương trình dưới đây sẽ check các câu lệnh của file, nếu nó tồn tại, màn hình sẽ hiển thị thông báo

``` sh
#!/bin/bash
if [ -f $1 ]
then
    echo "The " $1 " contains " $(wc -l < $1) " lines.";
    echo $?
fi
# ./count.sh /etc/passwd
The  /etc/passwd  contains  35  lines.
0
```

Bảng dưới đây mô tả các tùy chọn dùng để check file

|Option|Action|
|------|------|
|-e file|	Check xem file có tồn tại không.|
|-d file|	Check xem file có phải là một thư mục hay không.|
|-f file|	Check xem file có thường được sử dụng không.|
|-s file|	Check xem file có thích thước khác 0 hay không.|
|-g file|	Check xem file có tập sgid không.|
|-u file|	Check xem file có tập suid không.|
|-r file|	Check xem file có thể đọc được hay không.|
|-w file|	Check xem file có thể viết được hay không.|
|-x file|	Check xem file có thể thực thi (execute) được hay không|

Bạn cũng có thể dùng if để so sánh các chuỗi với nhau :

``` sh
if [ string1 == string2 ]
then
   ACTION
fi
```

Hoặc so sánh các số:

``` sh
if [ exp1 OPERATOR exp2 ]
then
   ACTION
fi
```

Một số các tùy chọn so sánh

|Option|Action|
|------|------|
|-eq|Bằng nhau|
|-ne|Không bằng nhau|
|-gt|Lớn hơn|
|-lt|Nhỏ hơn|
|-ge|Lớn hơn hoặc bằng|
|-le|Nhỏ hơn hoặc bằng|