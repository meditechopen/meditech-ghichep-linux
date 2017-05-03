# Các luồng tệp tin

Khi các câu lệnh được thực hiện, mặc định có 3 chuẩn luồng hoặc các bộ mô tả luôn luôn mở để sử dụng :

1. Chuẩn đầu vào hoặc stdin
2. Chuẩn đầu ra hoặc stdout
3. Chuẩn lỗi hoặc stderr

Thông thường, stdin là bàn phím của bạn, stdout và stderr và máy in trong thiết bị đầu cuối của bạn. Thường thì stderr được chuyển đến một tệp tin nhật lý lỗi. Stdin thường được cung cấp bằng các đầu vào trực tiế đến từ một tệp tin hoặc đến từ đầu ra của một lệnh trước đó qua một đường dẫn. Stdout cũng thường được chuyển hướng vào một tệp tin , Vì stderr là nơi lưu các thông báo lỗi được viết ra , nên thường không có gì xảy ra.

Trong Linux, tất cả các tệp tin mở được thể hiện bên trong bằng những gì được gọi là bộ mô tả tệp tin.Đơn giản chỉ cần đặt, những thứ là đại diện bằng các số bắt đầu từ số 0. Stdin là một mô tả 0, stdout là mô tả 1, và stderr là mô tả 2. Thông thường, nếu các tệp khác đã được mở ngoài 3 tệp này, những cái mà được mở theo mặc định, chúng sẽ bắt đầu môt tả lại tệp 3 và tăng lên từ đó.

Chúng ta có thể chuyển hướng 3 luồng tệp tin cơ bản để chúng ta có thể nhận được đầu vào từ một tệp tin hoặc lệnh khác thay vì từ bàn phím, và chúng ta có thể ghi dữ liệu ra và dữ liệu lỗi hoặc gửi chúng đến một đầu vào cho các tập lệnh tiếp theo. Ví dụ, có một chương trình called do_something, nó được độc từ stdin và viết đến stdout và stderr, chúng ta có thể  thay đôi nguồn đầu vào của nó :

```
$ do_something < input-file
```

Nếu bạn muốn gửi đầu ra đến 1 tệp tin , sử dụng như sau :

```
$ do_something > output-file
```

Chúng ta có thể đưa đầu ra của một lệnh hoặc chương trình vào đâù ra khác.

```
$ command1 | command2 | command3
```

# Tìm kiếm tệp

Các tiện ích định vị thực hiện một tìm kiếm thông qua một cơ sở dữ liệu đã được xây dựng trước đó của các tệp tin và thư mục trên hệ thống của bạn, kết nối tất cả với các mục chứa một chuỗi ký tự xác định. Định vị sử dụng cơ sở dữ liệu được tạo ra bởi một chương trình khác, cập nhật cơ sở dữ liệu. Hầu hết các hệ thống Linux chạy tự động mỗi ngày một lần. Tuy nhien, bạn có thể cập nhạt chúng mọi lúc bằng cách chạy updatedb từ các dòng lệnh với quyền root.

```
# yum install -y mlocate
# updatedb
# locate zip
```

Kết quả của việc tìm kiếm tiện ích có thể rất dài. Để có được một danh sách ngăn hơn, chúng ta cần sử dụng chương trinh grep để lọc. Nó sẽ chỉ in ra các dong chưa một hoặc nhiều chuỗi xác định :

```
$ locate zip | grep bin
/usr/bin/gpg-zip
/usr/bin/gunzip
/usr/bin/gzip
/usr/bin/zipdetails
```

Cái này liệt kê tất cả các thử mục "zip" và "bin" trong nên của nó .

Wildcards có thể được dùng để tìm kiếm về tên một tệp tin chứa các kí tự cụ thể.

| Ký tự đại diện | Kết quả|
|-------|------|
| ? | Phù hợp với bất kỳ kí tự đơn |
| * | Phù hợp với bất kì chuỗi kí tự |
| [set] | Phù hợp với bất kì kí tự nào có trong bộ kí tự |
| [!set] | Phù hợp với bất kì kí tự nào không có trong bộ kí tự |

Find là chương trình cực kì hữu ích và thường được sử dụng trong cuộc sống hằng ngày của một quản trị viên Linux. Nó thực hiện lại các cây hệ thống từ bất kì thư mục cụ thể (hoặc tập hợp các thư mục ) và định vị các tệp phù hợp với các điều kiện quy định. Mặc định luôn là thư mục làm việc hiện tại .

```
$ find /var -name *.log
/var/log/audit/audit.log
/var/log/tuned/tuned.log
/var/log/anaconda/anaconda.log
/var/log/anaconda/anaconda.program.log
/var/log/anaconda/anaconda.packaging.log
/var/log/anaconda/anaconda.storage.log
```

Khi không có đối số được đưa ra, find sẽ lên danh sách tất cả các tệp tin trong thư mục hiện tại và tất cả các thư mục con của nó.

Tìm kiếm thư mục có tên "gcc" :

```
$ find /usr -name gcc
```

Chỉ tìm kiếm các thư mục có tên "gcc" :

```
$ find /usr -type d -name gcc
```

Chỉ tìm kiếm các tệp thông thương có tên "test1"

```
$ find /usr -type f -name test1
```

Một cách sử dụng khác rất hay của find là có thể chạy các lệnh trên tệp tin phù hợp với tiêu chí tìm kiếm của bạn. Để tìm và xóa tất cả các tệp tin kết thúc bằng .swp :

```
$ find -name "*.swp" -exec rm {} ’;’
$ find -name "*.swp" -ok rm {} \;
```

{} là nơi giữ những thứ sẽ được điền đầy đủ tên tệp tin kết quả từ biểu thức find, và lệnh trược sẽ được chạy riêng biệt.  Lưu ý rằng bạn có thể kết thúc câu lệnh bằng `;` hoặc `\` . Cả 2 kiểu đều ổn. Loại thứ 2 hoạt động giống như loại 1 ngoại trừ việc sẽ nhắc bạn cho phép trước khi thực hiện lệnh. Điều này làm cho nó trở thành một cách để kiểm tra kết quả trước khi bạn thực hiện bất kì câu lệnh tiềm ẩn nguy hiểm nào.

ôi khi trường hợp bạn muốn tìm các tập tin theo các thuộc tính như khi chúng được tạo ra, được sử dụng cuối cùng,.. hoặc dựa trên kích thước của chúng. Cả 2 đều dễ thực hiện .

```
$ find / -ctime 3
```

Ở đây -ctime là inode siêu dữ liệu (tức là quyền sở hữu tệp, các quyền hạn,...) thay đổi cuối cùng. nó thường xảy ra , nhưng không nhất thiết là khi tệp tin được tạo ra lần đầu tiên. Bạn có thể tìm kiếm lần truy cập, lần đọc hay lần sửa cuối cùng với tham số -mtime. Số là số ngày và có thể được biểu diễn dười dạng giá trị số (n) , +n nghĩa là lơn hơn số đó, và -n nghĩa là bé hơn số đó.

Tìm kiếm dựa trên kích cỡ :

```
$ find / -size +10M
```

Để tìm kiếm tệp có kích thước lơn hơn 10MB :

# Quản lý tệp tin  :

Sử dụng các tiện ích sau để xem têp  :

| Câu lệnh | Cách dùng |
|------------|-----------|
| cat | Sử dụng để xem các tệp không dài |
| tac | Sử dụng để nhìn vào một tệp ngược, bắt đầu với dòng cuối cùng |
| less | Được sử dụng để xem các tệp tin lớn hơn vì có phân trang, nó tạm dừng ở mỗi màn hình của văn bản, cung cấp khả năng cuộn lai, cho phe tìm kiếm và điều hướng trong tệp |
| tail | Sử dụng để in 10 dòng cuối cùng của tệp theo mặc định, bạn có thể thay đổi bằng -n 15 để xem 15 dòng cuối |
| head | Ngược lại với tail, nó in 10 dòng đầu theo mặc định |

Câu lệnh touch thường được sử dụng để thiết lập  hoặc cập nhật quyền truy cập ,thay đổi, sửa đổi thời gian của các tệp tin. Theo mặc định nó đặt lại một tem thời gian của tập tin để phù hợp với thời gian hiện tại.

Tuy nhiên bạn cũng có thể tạo ra một tệp tin rỗng bằng các sử dụng touch :

```
$ touch <filename>
```

Điều này thường được thực hiện để tạo ra một tệp tin trống và để sử dụng cho các mục đích sau đó. Tùy chọn -t cho phép bạn đóng đầu thời gian ngày giờ của tập tin, để đặt nhãn thời gian vào một teeo tun cụ thể :

```
$ touch -t 03201600 <filename>
```

Lệnh mkdir dùng để tạo một thư mục. Loại bỏ 1 thư mục bằng rmkdir . Thư mục phải trống hoặc nó sẽ lỗi.

```
# mkdir ./test
# rmdir ./test
#
# mkdir ./test
# mkdir ./test/inside
# rmdir ./test
rmdir: failed to remove ‘test’: Directory not empty
# rm -rf ./test
# ls ./test
ls: cannot access ./test: No such file or directory
```

# So sánh tệp tin

Lệnh diff dùng để so sánh các tệp tin và thư mục :

```
$ cat file1.txt
Amor, ch'a nullo amato amar perdona,
Mi prese del costui piacer si forte,
Che, come vedi, ancor non m'abbandona.
$
$  cat file2.txt
amor, ch'a nullo amato amar perdona,
mi prese del costui piacer si forte,
che, come vedi, ancor non m'abbandona.
$
$ diff  file1.txt file2.txt
< Amor, ch'a nullo amato amar perdona,
< Mi prese del costui piacer si forte,
< Che, come vedi, ancor non m'abbandona.
---
> amor, ch'a nullo amato amar perdona,
> mi prese del costui piacer si forte,
> che, come vedi, ancor non m'abbandona.
$
$ diff -c file1.txt file2.txt
*** file1.txt   2015-02-17 16:10:03.781804799 +0100
--- file2.txt   2015-02-17 16:13:41.059088459 +0100
***************
! Amor, ch'a nullo amato amar perdona,
! Mi prese del costui piacer si forte,
! Che, come vedi, ancor non m'abbandona.
--- 1,3 ----
! amor, ch'a nullo amato amar perdona,
! mi prese del costui piacer si forte,
! che, come vedi, ancor non m'abbandona.
$
$  diff -i file1.txt file2.txt
$
```

# Tiện ích tệp tin

Trong Linux , phần mở rộng của tệp thường không phân loại theo cách nó có thể xảy ra trong hệ điều hành khác . Không thể giả định răng một tệp tin có tên file.txt là một tệp văn bản hoặc một chương trình thực thi. Trong Linux, một tệp tin nói chung có ý nghĩa hơn với người sử dụng hệ thống so với chính hệ thống. Trên thực tế, hầu hết các  ứng dụng trực tiếp kiểm tra nội dung của một tệp tin để xem loại đối tượng đó là gì chứ không phải dựa vào phần mở rộng. Bản chất của một tệp tin có thể được đưa ra dưới dạng đối số, nó kiểm tra các nội dung và đặc điểm nhất định để xác định liệu các tệp là văn bản thuần túy, các thư viện chia sẻ, các chương trình thực thi, các tập lệnh hoặc cái gì khác.

```
$ file /etc/resolv.conf
/etc/resolv.conf: ASCII text
```
