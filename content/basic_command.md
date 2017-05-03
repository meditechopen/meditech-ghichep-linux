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

Giả sử răng file1.txt đã tồn tại. Một liên kết cứng, được gọi là file2.txt được tạo ra bằng lệnh :

```
# ln file1.txt file2.txt
```

Lưu ý rằng hiện tại 2 tệp tin đã tồn tại, một kiểm tra chặt chẽ hơn cho thấy rằng điều này không đúng :

```
# ls -l file*
-rw-r--r--. 2 root root 604 Feb 16 11:49 file1.txt
-rw-r--r--. 2 root root 604 Feb 16 11:49 file2.txt
# ls -li file*
134415251 -rw-r--r--. 2 root root 604 Feb 16 11:49 file1.txt
134415251 -rw-r--r--. 2 root root 604 Feb 16 11:49 file2.txt
```

Tùy chọn -i sẽ in ra trong cột đầu tiên số i-node , cái mà là một số lượng duy nhất cho mỗi đối tượng tệp tin. Trường này giống nhau cho cả hai tệp tin. Những gì xảy ra ở đây là nó chỉ là một tệp tin nhưng nó có nhiều hơn một liên kết với nó, như được chỉ ra bởi 2 xuất bản trong đầu ra :

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

Thông báo file4.txt không còn xuất hiện như một tệp tin thường, và điểm rõ ràng đến file1 và có một số onide khác nhau. Các liên kết tượng trưng không chiếm thêm dung lượng trên hệ thống.Điều này rất thuận tiện vì chúng ta có thể chỉ đến những nơi khác nhau. Một cách dễ hơn để tạo ra một phím tắt từ thư mục chính đến thư mục gốc là tạo các liên kết tượng trưng.

Không giống như các liên kết cứng, liên kết mềm có thể trỏ đến các đối tượng trên các file hệ thống khác nhau (hoặc phân vùng) co thể không có thể hiện hoặc thậm chí không tồn tại. Trong trường hơp này liên kết không trỏ đến đối tượng hiện tại có sẵn hoặc đối tượng hiện tại.

Liên kết cứng rất hữu ích và tiết kiệm không gian, nhưng bạn phải cẩn thận trong việc sử dụng,đôi khi cần phải tinh tế. Đối với 1 thứ nếu bạn muốn xóa bỏ file1.txt hoặc file2.txt trong ví dụ, đối tượng inode vẫn còn tồn tại, điều này là không mong muốn vì nó có thể dẫn đến các báo lỗi sai khi bạn tạo lại 1 tệp tin có tên đó. Nếu bạn chỉnh sửa một trong các tệp tin, những gì xảy ra sẽ phụ thuộc và trình soạn thảo văn bản của bạn. Hầu hết các công cụ biên tập như vi và gedit sẽ giữ liên kết theo mặc định nhưng xóc thể sửa đổi một trong những tên có thể làm hỏng liên kết và tạo ra 2 đối tượng.
