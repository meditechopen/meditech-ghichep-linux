# Text commands

Linux cung cấp sẵn một vài các tiện ích cho việc chỉnh sửa văn bản

1. `cat` và `echo` dùng để hiển thị nội dung văn bản
2. `sed` và `awk` dùng để chỉnh sửa file
3. `grep` để tìm kiếm đặc biệt

## Hiển thị nội dung

- Câu lệnh `cat` (concatenate) thường được dùng để đọc, in và xem nội dung của file. Bên cạnh đó, người ta cũng sử dụng câu lệnh `tac` để xem nội dung theo thứ tự ngược lại.

- Để "in" nội dung vài file văn bản, ta dùng câu lệnh `cat > file.txt`, sau khi điền nội dung, sử dụng `Ctrl D` để lưu lại. 

Lưu ý nếu file chưa tồn tại, một file mới sẽ được tạo ra. Trong trường hợp file đã tồn tại và có nội dung, toàn bộ nội dung sẽ bị thay thế.

``` sh
$ cat > myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
Ctrl-D
$ cat myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
$
$ tac myfile.txt
Michele Laforca
Antonio Esposito
Mario Rossi
```

- Câu lệnh `echo` dùng để hiển thị văn bản

``` sh
$ echo myfile.txt
myfile.txt
]$echo HOME
HOME
$ echo $HOME
/home/ec2-user
```

## Chỉnh sửa nội dung file

- Câu lệnh `sed` (stream editor) là một công cụ xử lí văn bản rất hữu ích. Nó xử lí văn bản theo dòng dữ liệu. Những sữ liệu đầu vào sẽ được đưa vào không gian chỉnh sửa. Toàn bộ những thay đổi sẽ chỉ được áp dụng lên dữ liệu ở nơi này và nội dung cuối cùng sẽ được xuất ra file đầu ra.

Tùy chọn `/s` để thay thế

``` sh
$ sed s/Mario/Saverio/ myfile.txt
Saverio Rossi
Antonio Esposito
Michele Laforca
$ cat myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
```

Tuy nhiên sự thay đổi này không được áp dụng lên file nếu người dùng không khai báo đầu ra.

```
$ sed s/Mario/Saverio/ myfile.txt >  myfile2.txt
$ cat myfile2.txt
Saverio Rossi
Antonio Esposito
Michele Laforca
```

Bên cạnh đó, người dùng cũng có thể sử dụng tùy chọn `-i` để insert luôn vào file hiện tại. Xem thêm về các tùy chọn của câu lệnh `sed` [tại đây](http://www.grymoire.com/Unix/Sed.html)

- Câu lệnh `awk` được dùng để trích xuất đồng thời in ra màn hình những nội dung đặc biệt trong file.  Đây là công cụ rất hữu ích cho việc thao tác với các tệp dữ liệu, truy xuất và xử lí văn bản.

``` sh
$ awk '{ print $0 }' myfile.txt
Saverio Rossi
Antonio Esposito
Michele Laforca
$ awk '{ print $1 }' myfile.txt
Saverio
Antonio
Michele
$ awk '{ print $2 }' myfile.txt
Rossi
Esposito
Laforca
```

Xem thêm về câu lệnh `awk` [tại đây](https://www.tutorialspoint.com/unix_commands/awk.htm)

## Thao tác với file

- Câu lệnh `sort` được dùng để thay đổi thứ tự các dòng trong 1 file văn bản theo thứ tự tăng dần hoặc giảm dần. Nó cũng đi kèm với các tùy chọn. Ví dụ `-r` để hiển thị kết quả ngược lại.

``` sh
# cat myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
# sort myfile.txt
Antonio Esposito
Mario Rossi
Michele Laforca
# sort -r myfile.txt
Michele Laforca
Mario Rossi
Antonio Esposito
```

Xem thêm về câu lệnh `sort` [tại đây](http://www.tutorialspoint.com/unix_commands/sort.htm).

- Câu lệnh `uniq` được dùng để loại bỏ những dòng trùng lặp.

``` sh
# cat myfile.txt
Mario Rossi
Antonio Esposito
Michele Laforca
Antonio Esposito
# sort myfile.txt | uniq
Antonio Esposito
Mario Rossi
Michele Laforca
# sort myfile.txt | uniq -c
      2 Antonio Esposito
      1 Mario Rossi
      1 Michele Laforca
```

- Câu lệnh `paste` được dùng để kết hợp các trường từ các file khác nhau

``` sh
# cat names.txt
Mario Rossi
Antonio Esposito
Michele Laforca
Antonio Esposito
[root@caldera01 ~]# cat ages.txt
34
46
29
46
[root@caldera01 ~]# paste names.txt ages.txt
Mario Rossi     34
Antonio Esposito        46
Michele Laforca 29
Antonio Esposito        46
```

- Câu lệnh `join` được dùng để kết hợp 2 file có 1 trường trùng lặp

``` sh
# cat names.txt
01 Mario Rossi
02 Antonio Esposito
03 Michele Laforca
04 Antonio Esposito
# cat ages.txt
01 34
02 46
03 29
04 46
# join names.txt ages.txt
01 Mario Rossi 34
02 Antonio Esposito 46
03 Michele Laforca 29
04 Antonio Esposito 46
```

- Câu lệnh `grep` thường được sử dụng với mục đích tìm kiếm. Nó được dùng với "regular expressions" giúp ích hơn cho việc scan văn bản.

``` sh
# grep Ant* names.txt
02 Antonio Esposito
04 Antonio Esposito
```

- Câu lệnh `tr` được dùng để chuyển đổi những kí tự chỉ định hoặc xóa chúng

``` sh
# cat names.txt
01 Mario Rossi
02 Antonio Esposito
03 Michele Laforca
04 Antonio Esposito
# cat names.txt | tr a-z A-Z
01 MARIO ROSSI
02 ANTONIO ESPOSITO
03 MICHELE LAFORCA
04 ANTONIO ESPOSITO
```

- Câu lệnh `tee` lấy output từ bất kì câu lệnh nào, hiện thỉ ra màn hình và in nó ra 1 file riêng biệt

``` sh
# ls -l | tee list.txt
total 32
-rw-r--r--. 1 root root   24 Mar  3 14:42 ages.txt
-rw-------. 1 root root 1883 Jan 21 20:53 anaconda-ks.cfg
-rw-r--r--. 1 root root   74 Mar  3 14:42 names.txt
-rwxr--r--. 1 root root  102 Feb 21 16:47 script.sh
-rw-r--r--. 1 root root   74 Mar  3 14:52 tr
[root@caldera01 ~]# cat list.txt
total 32
-rw-r--r--. 1 root root   24 Mar  3 14:42 ages.txt
-rw-------. 1 root root 1883 Jan 21 20:53 anaconda-ks.cfg
-rw-r--r--. 1 root root   74 Mar  3 14:42 names.txt
-rwxr--r--. 1 root root  102 Feb 21 16:47 script.sh
-rw-r--r--. 1 root root   74 Mar  3 14:52 tr
```

- Câu lệnh `wc` (word count) được dùng để đếm số dòng, số từ và kí tự của 1 file hoặc nhiều file khác nhau

``` sh
# cat names.txt
01 Mario Rossi
02 Antonio Esposito
03 Michele Laforca
04 Antonio Esposito
[root@caldera01 ~]# wc -l names.txt
4 names.txt
[root@caldera01 ~]# wc -c names.txt
74 names.txt
[root@caldera01 ~]# wc -w names.txt
12 names.txt
```

- Câu lệnh `cut` được dùng để thao tác với các file có định dạng "cột". Mỗi một cột mặc định sẽ được phân cách với nhau bởi dấu tab. Thay vào đó, người dùng cũng có thể đưa ra các tùy chọn khác nhau để phân tách các cột thay cho sự lựa chọn mặc định. 

Ở ví dụ dưới đây, người dùng định nghĩa các cột được phân tách bởi dấu cách bằng tùy chọn `-d` , ngoài ra, người dùng cũng đưa thêm tùy chọn `-f` để lấy chính xác cột mà mình mong muốn.

``` sh
# cut -d" " -f1 names.txt
01
02
03
04
# cut -d" " -f2 names.txt
Mario
Antonio
Michele
Antonio
# cut -d" " -f3 names.txt
Rossi
Esposito
Laforca
Esposito
```

- Câu lệnh `head` sẽ lấy ra những dòng đầu trong 1 file (mặc định là 10 dòng). Người dùng có thể thêm tùy chọn để thiết lập số dòng lấy ra.

``` sh
# head -n 2 names.txt
01 Mario Rossi
02 Antonio Esposito
```

- Ngược lại, câu lệnh `tail` sẽ lấy ra một vài dòng cuối cùng của file (mặc định là 10).

``` sh
# tail -n 2 names.txt
03 Michele Laforca
04 Antonio Esposito
#
# tail -f -n3 /var/log/messages
Mar  3 14:38:59 caldera01 systemd: Started Session 35 of user root.
Mar  3 15:01:01 caldera01 systemd: Starting Session 36 of user root.
Mar  3 15:01:01 caldera01 systemd: Started Session 36 of user root.
```