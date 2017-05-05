## Vị trí các ứng dụng
Tùy thuộc vào sự phân bố cụ thể , các chương trình và các gói phần mềm có thể cài đặt trong các thư mục khác nhau . Nói chung , các chương trình thực thi nên ở trong các thư mục sau đây :

```
/bin
/usr/bin
/sbin
/usr/sbin
/opt.
```

Một cách để định vị các chương trình là sử dụng những tiện ích . Ví dụ, để tìm ra chính xác nơi các chương trình nằm trên file hệ thống :

```
$ which diff
/usr/bin/diff
```

Nếu không tìm thấy chương trình, whereis là một lựa chọn tốt bởi vì nó tìm các gói trong một phạm vi rộng hơn thư mục hệ thống.

```
$ whereis diff
diff: /usr/bin/diff /usr/share/man/man1/diff.1.gz
```

# Truy cập thư mục

Những câu lệnh sau rất hữu ích cho việc điều hướng thư mục :

|Câu lệnh|Kết quả|
|-------|-----------|
|cd 	|Di chuyển đến thư mục home|
|cd ..|Di chuyển đến thư mục cha|
|cd - |Di chuyển đến thư mục trước đó|
|cd /	|Di chuyển đến thư mục root (/) |

# Khám phá tệp tin hệ thống

Lệnh tree là một cách hay để có một cái nhìn về cây thư mục hệ thống. Các câu lệnh sau có thể giúp khám phá tệp tin hệ thống :

|Câu lệnh|Kết quả|
|-------|-----------|
|ls 	  |Liệt kê nội dung của thư mục làm việc hiện tại|
|ls –a  |Liệt kê tất cả các tệp tin và thư mục ẩn|
|tree   |Hiển thị chế độ xem dạng cây của các tệp tin hệ thống|
|tree -d|Chỉ cần liệt kê các thư mục và bỏ liệt kê tên tệp tin|

# Liên kết cứng và tượng trưng

Lệnh ln có thể được dử dụng để tạo các liên kết cứng và liên kết mềm, còn được biết là các liên kết tượng trưng hoặc symlink. Hai loại liên kết này rất phổ biến trên hệ đều hành dựa trên UNIX.

Giả sử răng file1.txt đã tồn tại. Tạo một liên kết cứng tên là file2.txt bằng lệnh :

```
# ln file1.txt file2.txt
```

Lưu ý rằng hiện tại 2 tệp tin đã tồn tại, kiểm tra kỹ hơn thấy điều này không đúng :

```
# ls -l file*
-rw-r--r--. 2 root root 604 Feb 16 11:49 file1.txt
-rw-r--r--. 2 root root 604 Feb 16 11:49 file2.txt
# ls -li file*
134415251 -rw-r--r--. 2 root root 604 Feb 16 11:49 file1.txt
134415251 -rw-r--r--. 2 root root 604 Feb 16 11:49 file2.txt
```

Tùy chọn -i sẽ in ra số i-node trong cột đầu tiên, cái này là đối số duy nhất của mỗi tệp tin. Trường này cả hai tệp tin đều giống nhau. Chỉ có thể xảy ra trường hợp có 1 tệp tin nhưng lại có nhiều liên kết đến tệp tin đó nhưng đầu ra lại có 2 bản :

```
# ln file1.txt file3.txt
# ls -li file*
134415251 -rw-r--r--. 3 root root 604 Feb 16 11:49 file1.txt
134415251 -rw-r--r--. 3 root root 604 Feb 16 11:49 file2.txt
134415251 -rw-r--r--. 3 root root 604 Feb 16 11:49 file3.txt
```

Thay đổi file3.txt có nghĩa là thay đổi cùng đối tượng với tên là file1.txt,file2.txt và file3.txt.

Các liên kết tượng trưng hoặc mềm được tạo ra với tùy chọn -s như :

```
# ln -s file1.txt file4.txt
# ls -li file*
134415251 -rw-r--r--. 3 root root 644 Feb 16 11:59 file1.txt
134415251 -rw-r--r--. 3 root root 644 Feb 16 11:59 file2.txt
134415251 -rw-r--r--. 3 root root 644 Feb 16 11:59 file3.txt
134415252 lrwxrwxrwx. 1 root root   9 Feb 16 11:59 file4.txt -> file1.txt
```

file4.txt không còn như những file khác, và điểm thấy rõ nhất là nó trỏ đến file1 và có số i-node khác biệt. Các liên kết tượng trưng không chiếm thêm dung lượng của ổ đĩa.Điều này rất thuận tiện vì chúng ta có thể  đến nhiều nơi khác nhau bằng cách tạo các liên kết tượng trưng từ thu mục bất kì đến thư mục không mà không làm tăng dung lượng thư mục đó.

Không giống như các liên kết cứng, liên kết mềm có thể trỏ đến các tệp tin khác nhau trong hệ thống (hoặc phân vùng) cho dù tệp tin đó có hay không.

Liên kết cứng rất hữu ích và tiết kiệm không gian, nhưng phải cẩn thận trong việc sử dụng,đôi khi cần phải tinh tế. Nếu bạn muốn xóa bỏ file1.txt hoặc file2.txt trong ví dụ, đối tượng inode vẫn còn tồn tại, điều này là không mong muốn vì nó có thể dẫn đến các báo lỗi sai khi bạn tạo lại 1 tệp tin có tên đó. Nếu bạn chỉnh sửa một trong các tệp tin, kết quả sẽ phụ thuộc và trình soạn thảo văn bản của bạn. Hầu hết các công cụ biên tập như vi và gedit sẽ giữ liên kết theo mặc định nhưng có thể sửa đổi một trong những tên có thể làm hỏng liên kết và tạo ra 2 đối tượng.
