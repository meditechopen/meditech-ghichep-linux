Bash shell programming

Shell là một trình thông dịch dòng lệnh cái mà cung cấp giao diện người dùng dưới dạng cửa sổ terminal. Nó cũng có thể được sử dụng để chạy các mã scrips,kể cả trong những phiên không tương tác mà không cần cửa sổ terminal,giống như các câu lệnh được gõ trực tiếp.

```
#!/bin/bash
find /usr/lib -name "*.c" -ls
```

Dòng đầu tiên của script , nó bắt đầu bằng `#!/bin/bash` chứa đầy đủ các đường dẫn của câu lệnh thông dịch mà sẽ được dùng trên tệp tin . Các lệnh thông dịch được giao nhiệm vụ cùng với thực thi các câu lệnh mà theo nó trong các script. Thông thường các thông dịch thường sử dụng :

```
/usr/bin/perl
/bin/bash
/bin/csh
/bin/tcsh
/bin/ksh
/usr/bin/python
/bin/sh
```

script không chỉ giới hạn các thông dịch shell . Nó cũng có thể được sử dụng cho các Python scripts.

```
# ll script
-rwxr--r--. 1 root root 55 Mar  3 15:22 script
# cat script
#!/usr/bin/python
print "Welcome to the Python script"
# ./script
Welcome to the Python script
```

 Các script cũng có thể được tương tác :

 ```
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

Tất cả các Shell script tạo ra một giá trị trả về  khi thực hiện kết thúc.Giá trị có thể được cài đặt cùng với câu lệnh `exit`. Giá trị trả về cho phép một tiến trình để giám sát trạng thái exit của các tiến trình thường xuyên khác trong một mối quan hệ cha-con. Điều này xác định các tiến trình này chấm dứt như thế nào và thực hiện các bước cần thiết, có thể thành công hoặc thất bại. Theo quy ước, thành công được trả về giá trị 0, và thất bại được trả về một giá trị khác 0. Giá trị trả về luôn được lưu trữ trong biến môi trường `$?`.

```
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

Basic sytax

Các script yêu cầu bạn phải tuân thủ các cú pháp theo các ngôn ngữ chuẩn. Các tuy tắc mô tả làm thể nào để xác định các biến và làm thế nào để xây dựng và định dạng các câu lệnh được phép,... Bảng liệt kê một số cách sử dụng các ký tự đặc biệt trong các bash script.


|Character|Description|
|---------|-----------|
|#|Được dùng để  comment, ngoại trừ khi sử dụng với \#, hoặc như #! khi bắt đầu một script|
|\\| Được suwrr dụng ở cuối dòng để cho biết dòng tiếp theo|
|;|Được sử dụng để giải thích sau những câu lệnh mới|
|$|Chỉ ra những cái sau đó là một biến|

Đôi khi bạn có thể muốn nhóm nhiều câu lệnh trên một dòng. Ký tự dấu chấm phẩy được dùng để tách biệt các lệnh này và thực hiện chúng theo thứ tự như thể chúng được gõ trên các dòng riêng biệt.

Ba câu lệnh trong ví dụ say sẽ thực thi ngay cả khi các lệnh trước đó thực hiện không thành công :

```
$ make ; make install ; make clean
```

Tuy nhiên , bạn có thể muốn hủy bỏ những lệnh tiếp theo nếu một trong những chúng bị lỗi . Bạn có thể làm điều này bằng toán tử `and` :

```
$ make && make install && make clean
```

Nếu câu lệnh đầu tiên bị lỗi thì câu lệnh tiếp theo sẽ không bao giờ được thực hiện. Một sàng lọc cuối cùng là toán tử hoặc :

```
$ cat file1 || cat file2 || cat file3
```

Trong lựa chọn này, bạn sẽ tiến hành cho đến khi một cái gì đó thành công và sau đó bạn sẽ ngừng thực hiện các bước tiếp theo.

Các hàm

Một hàm là các khối mã thực thi một tập các thao tác. Các chức năng rất hữu ích cho việc thực hiện  các thủ tục nhiều lần với các biến đầu vào khác nhau.Các hàm cũng thường gọi là chương trình con. Sử dụng các hàm trong các script đòi hỏi 2 bước :

 1. Khai báo hàm
 2. Gọi hàm

Khai báo hàm đòi hỏi một cái tên, cái mà được sử dụng để gọi nó. Cú pháp thích hợp là :

```
function_name () {
   command...
}
```

Ví dụ, hàm sau đây được đặt tên là hàm hiển thị :

```
display () {
       echo "This is a sample function"
    }
```

Hàm có thể chỉ cần yêu cầu và có nhiều câu lệnh. Một khi được định nghĩa, hàm có thể được gọi nhiều lần nếu cần thiết. Trong ví dụ đầy đủ hiển thị trong hình, chúng tôi cũng hiển thị một sàng lọc thường được sử dụng : làm thế nào để chuyển một đối số đến hàm. Đầu tiên, thứ 2 ,..., đối số thứ n có thể được gọi là $1,$2,...$n . Tên script được gọi là $0, Tất cả các tham số được gọi là $* và tổng tất cả các tham số là $#.

```
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

# Thay thế lệnh

Bạn có thể cần phải thay thế kết qủa của một lệnh như là một phần của lệnh khác. Nó có thể được thực hiện theo 2 đường : 

1. Bằng cách kèm các lệnh bên trong backticks(`)
2. Bằng cách kèm theo các lệnh bên trong $(`)
