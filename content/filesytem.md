# Cấu trúc tệp tin hệ thống

Trên nhiều hệ thống, kể cả Linux, hệ thống tập tin được cấu trúc giống như một cái cây. Và thường được miêu tả ngược lại, và bắt đầu từ thư mục gốc, sau đó đến các thư mục con và thư mục con bên trong,giữa các thư mục được ngăn bằng dấu /.

Tiêu chuẩn Phân cấp Hệ thống tập tin (FHS) đã phát triển vượt trội so với các tiêu chuẩn lịch sử từ các phiên bản đầu tiên của UNIX. FHS cung cấp cho các nhà phát triển và quản trị hệ thống Linux một cấu trúc tiêu chuẩn thư mục cho hệ thống tập tin, cung cấp tính nhất quán giữa các hệ thống và phân phối. Linux hỗ trợ các loại hệ thống tập tin khác nhau được tạo ra cho Linux, cùng với các hệ thống tập tin tương thích từ các hệ điều hành khác nhau. Nhiều hệ thống tệp cũ hơn cũng được hỗ trợ. Một số ví dụ về các loại hệ tập tin mà Linux hỗ trợ là:

1. ext3, ext4, btrfs, xfs(Hệ thống tập tin Linux gốc)
2. vfat, ntfs, hfs(Hệ thống tập tin từ các hệ điều hành khác)

Mỗi hệ thống tập tin nằm trên một phân vùng đĩa cứng. Phân vùng giúp tổ chức các nội dung của đĩa theo loại chứa dữ liệu và nó được sử dụng như thế nào. Ví dụ, các chương trình quan trọng cần thiết để chạy hệ thống thường được lưu trữ trên một phân vùng riêng biệt so với tệp chứa các tệp do người dùng thông thường sở hữu. Ngoài ra, các tệp tạm tạo và xóa trong quá trình hoạt động bình thường của Linux thường nằm trên một phân vùng riêng; Theo cách này, sử dụng tất cả các không gian có sẵn trên một phân vùng cụ thể có thể không gây ảnh hưởng đến hoạt động bình thường của hệ thống.

Trước khi bắt đầu sử dụng một hệ thống tập tin, bạn cần gắn nó vào cây hệ thống tập tin tại một điểm kết nối. Đây chỉ đơn giản là một thư mục (có thể hoặc không có sản phẩm nào) nơi hệ thống tập tin được gắn vào (gắn kết). Đôi khi bạn cần phải tạo thư mục nếu nó chưa tồn tại. Nếu bạn gắn kết một hệ thống tập tin vào một thư mục không rỗng.trước đây các nội dung của thư mục đó được giấu và không thể truy cập cho đến khi hệ thống tập tin được ngắt kết nối. Do đó điểm gắn kết thường là các thư mục trống.

Lệnh mount được sử dụng để đính kèm một hệ thống tập tin nào đó bên trong cây hệ thống tập tin. Đối số bao gồm nút thiết bị và điểm kết nối.

```
$ mount /dev/sda5 /mnt
```

Điều này sẽ đính kèm hệ thống tập tin chứa trong phân vùng đĩa liên kết với nút thiết bị /dev/sda5, vào cây hệ thống tập tin tại điểm gắn kết /mnt. Lưu ý rằng trừ khi hệ thống được cấu hình khác chỉ có người dùng root có quyền chạy mount. Nếu bạn muốn nó được tự động có sẵn mỗi khi hệ thống khởi động, bạn cần phải chỉnh sửa tập tin /etc/fstab cho phù hợp. là tên viết tắt của Bảng Hệ thống tệp tin. Nhìn vào tập tin này sẽ cho bạn thấy cấu hình của tất cả các hệ thống tập tin cấu hình sẵn.

Lệnh umount được sử dụng để tách hệ thống tập tin khỏi điểm kết nối.

```
$ umount /mnt
```

Lệnh df -Th sẽ hiển thị thông tin về các hệ thống tập tin được gắn kết bao gồm số liệu thống kê loại và cách sử dụng về không gian hiện có và không gian sẵn có.

```
# df -Th
Filesystem                 Type      Size  Used Avail Use% Mounted on
/dev/mapper/os-root        xfs        50G  2.0G   48G   4% /
devtmpfs                   devtmpfs  1.8G     0  1.8G   0% /dev
tmpfs                      tmpfs     1.9G  4.0K  1.9G   1% /dev/shm
tmpfs                      tmpfs     1.9G  8.6M  1.8G   1% /run
tmpfs                      tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/swift01-zone01 xfs        49G   33M   49G   1% /srv/node/z1d1
/dev/mapper/swift02-zone02 xfs        49G   33M   49G   1% /srv/node/z2d1
/dev/sda1                  xfs       497M  167M  331M  34% /boot
/dev/mapper/os-data        xfs        20G  261M   20G   2% /data
```

# Thư mục home

Trong bất kỳ hệ thống UNIX, mỗi người dùng có thư mục chính của riêng mình, thường được đặt dưới /home. Thư mục root / trên các hệ thống Linux mới không được nhiều hơn thư mục gốc của root. Thư mục /home thường được gắn kết như là một hệ thống tập tin riêng biệt trên phân vùng riêng của nó.

# Các thư mục nhị phân

Thư mục /bin chứa các chương trình thực thi, các lệnh cần thiết được sử dụng trong chế độ người dùng đơn và các lệnh cần thiết được yêu cầu bởi tất cả người dùng hệ thống, chẳng hạn như ps, ls, cp. Các lệnh không cần thiết cho hệ thống trong chế độ người dùng đơn được đặt trong thư mục /usr/bin, trong khi thư mục /sbin được sử dụng cho các nhị phân thiết yếu liên quan đến quản trị hệ thống, chẳng hạn như ifconfig và shutdown. Ngoài ra còn có thư mục /usr/sbin cho các chương trình quản trị hệ thống ít cần thiết. Tất cả các thư mục nhị phân nằm dưới phân vùng root. Đôi khi /usr là một hệ thống tập tin riêng biệt mà có thể không có sẵn trong chế độ người dùng đơn. Đó là lý do tại sao các lệnh thiết yếu đã được tách ra khỏi các lệnh không cần thiết. Tuy nhiên, trong một số hệ thống Linux mới, sự phân biệt này được coi là đã lỗi thời, và /usr/bin và /bin thực sự chỉ được liên kết với nhau như là /usr/sbin và /sbin.

# Thư mục thiết bị

Thư mục /dev chứa các nút thiết bị, một loại tệp giả sử dụng phần lớn các thiết bị phần cứng và phần mềm, ngoại trừ các thiết bị mạng. Thư mục này rỗng trên phân vùng đĩa khi nó không được gắn kết nhưng chứa các mục được tạo ra bởi hệ thống udev, tạo ra và quản lý các nút thiết bị trên Linux, tạo chúng tự động khi các thiết bị được tìm thấy. Thư mục /dev chứa các mục như:

```
/dev/sda1
/dev/lp1
/dev/dvd1
```

# Thư mục biến

Thư mục var /var chứa các tệp dự kiến ​​sẽ thay đổi về kích thước và nội dung khi hệ thống đang chạy (var là viết tắt của biến) chẳng hạn như các mục trong các thư mục sau:
<ul>
<li>Tệp nhật ký hệ thống
 : /var/log </li>
 <li>Các gói tin: /var/lib</li>
 <li>Hàng đợi in : /var/spool</li>
 <li>Nhãn tệp tin : var/tmp</li>
 <li>Thư mục chính của FTP: /var/ftp</li>
 <li>Thư mục Máy chủ Web: /var/www</li>
</ul>

Thư mục var /var có thể được đặt trong phân vùng riêng của nó để phát triển các tập tin có thể được cung cấp và kích thước tập tin mà không ảnh hưởng đến hệ thống.

# Thư mục cấu hình hệ thống

Thư mục /etc là nơi chứa tập tin cấu hình hệ thống. Nó không chứa các chương trình nhị phân, mặc dù có một số tập lệnh thực thi. Ví dụ, tập tin resolv.conf nói với hệ thống nơi để đi trên mạng để có được tên máy chủ để ánh xạ địa chỉ IP (DNS). Các tệp tin như passwd, shadow và nhóm để quản lý tài khoản người dùng được tìm thấy trong thư mục /etc. Các kịch bản cấp độ hệ thống được tìm thấy trong các thư mục con của /etc. Ví dụ, /etc/rc2.d chứa liên kết đến các kịch bản để vào và rời khỏi mức độ chạy 2. Một số bản phân phối Linux mở rộng nội dung của /etc. Ví dụ, Red Hat thêm thư mục con /etc/sysconfig chứa nhiều tệp cấu hình hơn.

# Thư mục khởi động

Thư mục /boot chứa vài tập tin cần thiết để khởi động hệ thống. Đối với mỗi hạt nhân thay thế được cài đặt trên hệ thống có bốn tập tin:

 - Vmlinuz là hạt nhân Linux, bắt buộc để khởi động.
 - Initramfs là hệ thống tập tin ram ban đầu, được yêu cầu để khởi động
- Config is :  tập tin cấu hình hạt nhân, chỉ được sử dụng để gỡ lỗi
- System.map chứa bảng ký hiệu hạt nhân, chỉ được sử dụng để gỡ lỗi
- Mỗi tập tin này có một phiên bản hạt nhân được nối vào tên của nó.

# Thư mục thư viện

Thư viện /lib chứa các thư viện (mã chung chung được chia sẻ bởi các ứng dụng và cần thiết để chúng chạy) cho các chương trình thiết yếu trong thư mục /bin và /sbin. Hầu hết trong số này là những gì được gọi là thư viện được nạp động (còn được gọi là thư viện chia sẻ hoặc Shared Objects (SO)). Trên một số bản phân phối Linux, có thư mục a/lib64 có chứa các thư viện 64-bit, trong khi /lib chứa các phiên bản 32-bit. Các mô-đun hạt nhân (mã hạt nhân, thường là trình điều khiển thiết bị, có thể được nạp và giải phóng mà không cần khởi động lại hệ thống) được đặt trong /lib/modules /.

# Thư mục bổ sung

| Đường dẫn | Cách dùng |
|------------|--------------|
|/opt |Gói phần mềm ứng dụng tùy chọn |
|/sys	 |Hệ thống tập tin giả cho thông tin về hệ thống và phần cứng. Có thể được sử dụng để thay đổi các thông số hệ thống và cho mục đích gỡ lỗi. |
|/srv | |
|/tmp |Tệp tin tạm thời; Trên một số phân phối các tập tin này sẽ bị xóa trên một khởi động lại |
|/media |Nó thường nằm ở nơi mà các phương tiện di động, chẳng hạn như đĩa CD, DVD và ổ đĩa USB được gắn kết. Trừ khi cấu hình ngăn cấm nó, Linux tự động gắn các phương tiện di động trong thư mục này khi chúng được phát hiện. |
|/usr |Ứng dụng đa người dùng, tiện ích và dữ liệu |
|/usr/include |Tập tin tiêu đề được sử dụng để biên dịch ứng dụng |
|/usr/lib |Thư viện cho các chương trình nhị phân |
|/usr/lib64 |Thư viện 64bit cho các chương trình nhị phân |
|/usr/share |Dữ liệu chia sẻ được sử dụng bởi các ứng dụng, nói chung là kiến ​​trúc độc lập |
|/usr/src |Mã nguồn, thường là cho nhân Linux|
|/usr/local |Dữ liệu và các chương trình cụ thể cho máy cục bộ. |

# Bảng thư mục hệ thống
