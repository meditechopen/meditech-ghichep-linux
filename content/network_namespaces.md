# 23. Linux Network Namespaces

Thông thường, một bản cài đặt Linux sẽ chia sẻ chung tập hợp các network interfaces và các bản ghi trên bảng định tuyến. Ta có thể chỉnh sửa bảng định tuyến sử dụng các chính sách định tuyến, tuy nhiên về căn bản thì điều đó không thay đổi thực tế là các network interfaces và các bảng định tuyến vẫn chia sẻ chung khi xét trên toàn bộ hệ điều hành. 
Linux network namespaces được đưa ra để giải quyết vấn đề đó. Với linux namespaces, ta có thể có các máy ảo tách biệt nhau về network interfaces cũng như bảng định tuyến khi mà các máy ảo này vận hành trên các namespaces khác nhau. Mỗi network namespaces có bản định tuyến riêng, các thiết lập iptables riêng cung cấp cơ chế NAT và lọc đối với các máy ảo thuộc namespace đó. Linux network namespaces cũng cung cấp thêm khả năng để chạy các tiến trình riêng biệt trong nội bộ mỗi namespace. 

## Các cấu trúc cơ bản trong namespace

Trong Linux, chỉ user root mới có thể chỉnh sửa các file cấu hình network.

Tạo mới một network namespace

``` sh
[root@centos-01 ~]# ip netns add Blue
[root@centos-01 ~]# ip netns list
Blue
[root@centos-01 network-scripts]# ll /var/run/netns/
total 0
-r--r--r-- 1 root root 0 Feb 11 16:29 Blue
```

Mỗi một network namespace sẽ có riêng địa chỉ loopback, bảng định tuyến và cả iptables nữa.

``` sh
[root@centos-01 ~]# ip netns exec Blue ip addr list
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

Chắc chắn rằng bạn đã bật nó lên trước khi tiến hành các thao tác tiếp theo với network namespace

``` sh
[root@centos-01 ~]# ip netns exec Blue ip link set dev lo up
[root@centos-01 ~]# ip netns exec Blue ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Network namespace còn cung cấp khả năng chạy các tiến trình trên nó. Ví dụ dưới đây chạy một chương trình bash trên namespace Blue

``` sh
[root@centos-01 ~]# ip netns exec Blue bash
[root@centos-01 ~]# ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[root@centos-01 ~]# netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
[root@centos-01 ~]# exit
```

Để xóa 1 namespace

``` sh
[root@centos-01 ~]# ip netns add Yellow
[root@centos-01 ~]# ip netns list
Yellow
Blue
[root@centos-01 ~]# ip netns delete Yellow
[root@centos-01 ~]# ip netns list
Blue
```

## Gán các cổng cho network namespaces

Để kết nối network namespace với mạng bên ngoài cần gán các virtual interface vào "default" hoặc "global" namespace, nơi mà card mạng thật được gán vào. Để làm được điều này, trước tiên ta cần tạo ra 2 virtual interfaces là `veha` và `vehb` 

``` sh
[root@centos-01 ~]# ip link add vetha type veth peer name vethb
```

Gán `vehb` vào namespace Blue

``` sh
[root@centos-01 ~]# ip link set vethb netns Blue
[root@centos-01 ~]# ip netns exec Blue ip link set dev vethb up
[root@centos-01 ~]# ip netns exec Blue ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
vethb: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 7e:e4:29:bc:9c:67  txqueuelen 1000  (Ethernet)
```

`veha` vẫn đang nằm ở namespace global

``` sh
[root@centos-01 ~]# ip link set dev vetha up
[root@centos-01 ~]# ifconfig
ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.21  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 fe80::20c:29ff:fe1e:6bf1  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:1e:6b:f1  txqueuelen 1000  (Ethernet)
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
vetha: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::e899:ceff:fef6:3010  prefixlen 64  scopeid 0x20<link>
        ether ea:99:ce:f6:30:10  txqueuelen 1000  (Ethernet)
```

Cấu hình virtual interface nằm tại namespace global 

``` sh
[root@centos-01 ~]# ip addr add 192.168.100.1/24 dev vetha
[root@centos-01 ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    100    0        0 ens32
10.10.10.0      0.0.0.0         255.255.255.0   U     100    0        0 ens32
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 vetha
[root@centos-01 ~]#
```

và trong namespace Blue

``` sh
[root@centos-01 ~]# ip netns exec Blue ip addr add 192.168.100.2/24 dev vethb
[root@centos-01 ~]# ip netns exec Blue route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 vethb
[root@centos-01 ~]#
```

Cả hai namespace giờ đây đã kết nối với nhau qua virtual interface

``` sh
[root@centos-01 ~]# ping 192.168.100.2
PING 192.168.100.2 (192.168.100.2) 56(84) bytes of data.
64 bytes from 192.168.100.2: icmp_seq=1 ttl=64 time=0.041 ms
64 bytes from 192.168.100.2: icmp_seq=2 ttl=64 time=0.029 ms
^C
[root@centos-01 ~]# ip netns exec Blue ping 192.168.100.1
PING 192.168.100.1 (192.168.100.1) 56(84) bytes of data.
64 bytes from 192.168.100.1: icmp_seq=1 ttl=64 time=0.034 ms
64 bytes from 192.168.100.1: icmp_seq=2 ttl=64 time=0.039 ms
^C
```

Tuy vậy chúng được định tuyến hoàn toàn tách biệt nhau

``` sh
[root@centos-01 ~]# ip netns exec Blue ping 10.10.10.1
connect: Network is unreachable
[root@centos-01 ~]#
[root@centos-01 ~]# ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.545 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.369 ms
^C
```