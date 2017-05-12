# Bản phát hành Linux và thông tin hệ thống

Người quản trị hệ thống Linux cần lấy thông tin từ hệ thống. Dưới đây là một số lệnh hữu ích :

```
# cat /etc/*release
CentOS Linux release 7.0.1406 (Core)
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"
CentOS Linux release 7.0.1406 (Core)
```

Kernel version

```
# uname -r
3.10.0-123.13.2.el7.x86_64
```

Thông tin bộ nhớ

```
# head /proc/meminfo
MemTotal:        3776748 kB
MemFree:         2230496 kB
MemAvailable:    2782088 kB
Buffers:            1452 kB
Cached:           652196 kB
SwapCached:            0 kB
Active:          1069616 kB
Inactive:         193056 kB
Active(anon):     609504 kB
Inactive(anon):     8304 kB
```

Tệp tin hệ thống

```
# df -h
Filesystem         Dimens. Usati Disp. Uso% Montato su
/dev/sda1          12G     6,2G  4,9G  56%      /
/dev/small-db02    5,9G    2,6G  3,0G  46%      /db02
/dev/small-db01    5,0G    3,6G  1,2G  77%      /db01
/dev/small-db05    7,8G    1,2G  6,2G  17%      /db05
/dev/small-db03    39G     5,4G   32G  15%      /db03                      
/dev/small-db04    30G     2,5G   26G   9%      /db04
```

Đếm số lượng CPU

```
# cat /proc/cpuinfo | grep model | uniq -c
      2 model name      : Intel(R) Core(TM)2 Duo CPU     E8500  @ 3.16GHz
```

Hệ thống tập tin proc

Hệ thống tập tin /proc chưa các tập tin ảo chỉ tồn tại trong bô nhớ. Hệ thống này chứa các tập tin và thư mục bắt chước cấu trúc của hạt nhân và thông tin cấu hình .Nó không chứa các tệp tin thực những thôn tin thời gian chạy hệ thống (vd : bộ nhớ ,thiết bị được gắn, cấu hình phần cứng ,.... ) Một số tệp tin quan trọng trong /proc là :

```
/proc/cpuinfo
/proc/interrupts
/proc/meminfo
/proc/mounts
/proc/partitions
/proc/version
/proc/<process-id-#>
/proc/sys
```

Hệ thống tập tin /proc rất hữu ích, bởi vì nó chủ báo cáo những thông tin cần thiết và không cần lưu trữ trên đĩa.

# Tên máy chủ

Tên máy chủ xác định máy trong miền

```
# cat /etc/hostname
```

Đặt tên máy chủ mới

```
# hostname NEW_NAME
```

Điều này sẽ thiết lập tên máy chủ trong hệ thống thành tên mới là NEW_NAME . Thao tác này sẽ hoạt động ngay và giữ nguyên cho đến khi hệ thống khởi động lại. Trên hệ thông Debian , sử dụng tập tin /etc/hostname để đọc tên máy trong hệ thống lúc khởi động và thiết lập nó bằng /init script /etc/init.d/hostname.sh . Tên máy chủ được lưu lại trên file /etc/hostname sẽ lưu khi khởi động lại và sẽ được sử dụng như script chúng ta đã dùng.


Trên hệ thống dựa trên RedHat, sử dụng tiện ích hostnamectl để lấy và đặt tên máy.

```
# hostnamectl status
   Static hostname: caldera01
         Icon name: computer-desktop
           Chassis: desktop
        Machine ID: <machineId>
           Boot ID: <bootId>
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-123.13.2.el7.x86_64
      Architecture: x86_64
```
