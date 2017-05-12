# 22. Linux Virtual Networking

Với việc sử dụng libvirt và linux virtual bridge, việc tạo và quản lí mạng ảo trên hệ điều hành Linux đã trở nên dễ dàng hơn rất nhiều. Xem thêm về Libvirt Virtual Networking [tại đây](http://wiki.libvirt.org/page/VirtualNetworking)

Cài đặt, khởi động và kích hoạt libvirt:

``` sh
# yum install libvirt
# systemctl start libvirtd
# systemctl enable libvirtd
```

Libvirt hỗ trợ những loại mạng ảo dưới đây:

1. NAT mode
2. Routed mode
3. Isolated mode
4. Bridged mode

## NAT mode

Khi libvirt được cài lên server, nó sẽ có thiết lập mặc định là NAT mode. Switch ảo được thiết lập với chế độ này sẽ được dùng bởi các VM để giao tiếp với mạng bên ngoài thông qua card vật lí của máy chủ. Cấu hình được lưu tại file xml `/etc/libvirt/qemu/networks/default.xml` . Bridge có tên `vibr0` đồng thời cũng sẽ được tạo ra trên host vật lí.

``` sh
# cat /etc/libvirt/qemu/networks/default.xml
<network>
  <name>default</name>
  <uuid>f6514662-26e5-431e-8544-fd216bed9312</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:53:d4:ee'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>

# ifconfig virbr0
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:53:d4:ee  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# ping 192.168.122.1
PING 192.168.122.1 (192.168.122.1) 56(84) bytes of data.
64 bytes from 192.168.122.1: icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from 192.168.122.1: icmp_seq=2 ttl=64 time=0.035 ms
^C
```

Câu lệnh `virsh` được dùng để cấu hình và kiểm tra mạng ảo

``` sh
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
```

Trong trường hợp không thấy có mạng default, người dùng có thể reload và kích hoạt lại file cấu hình

``` sh
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------

# virsh net-define /etc/libvirt/qemu/networks/default.xml
Network default defined from /etc/libvirt/qemu/networks/default.xml
# virsh net-start default
Network default started
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
```

Khi mà chế độ mạng ảo mặc định của libvirt đang chạy, bạn sẽ thấy 1 bridge độc lập. Thiết bị này không được gán bởi bất kì cổng mạng vật lí nào, nó sử dụng NAT và forward trực tiếp ra bên ngoài mạng thật. Đừng gán bất kì card mạng nào cho bridge này

``` sh
# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
```

Libvirt sẽ add rules vào iptables để cho phép traffic từ các máy ảo. Nó cũng sẽ kích hoạt chế độ `ip_forward` . Tuy nhiên một vài ứng dụng có thể sẽ tắt nó đi, tốt nhất là bạn nên thêm dòng sau để kiểm tra trước khi thực hiện tiến trình:

``` sh
# cat /etc/sysctl.conf
...
net.ipv4.ip_forward=1
...
```

Tạo máy ảo (KVM) và gán nó vào mạng ảo

``` sh
# yum install virt-install
# virt-install \
--name VM1 \
--ram 2048 \
--vcpu 2 \
--network network=default \
--graphics none \
--os-type linux \
--disk path=/data/vm-images/vm1.img,size=10 \
--location /tmp/ubuntu-14.04.1-server-amd64.iso \
--extra-args 'console=ttyS0,115200n8 serial'
```

Trên host vật lí, bạn sẽ thấy có thêm một interface là `vnet0`, đây là một phần của linux bridge và nó cũng chỉ vừa mới được định nghĩa

``` sh
#ifconfig vnet0
vnet0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc54:ff:fe6d:3355  prefixlen 64  scopeid 0x20<link>
        ether fe:54:00:6d:33:55  txqueuelen 500  (Ethernet)
        RX packets 117  bytes 55089 (53.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6958  bytes 372399 (363.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
                                                        vnet0
```

Đăng nhập vào máy ảo và kiểm tra khả năng truy cập mạng ra mạng bên ngoài

``` sh
# virsh list --all
 Id    Name                           State
----------------------------------------------------
 4     VM1                            running

# virsh console VM1
Connected to domain VM1
Ubuntu 14.04.2 LTS VM1 ttyS0
VM1 login: caldera
Password:

caldera@VM1:~$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 52:54:00:6d:33:55
          inet addr:192.168.122.91  Bcast:192.168.122.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe6d:3355/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:94 errors:0 dropped:0 overruns:0 frame:0
          TX packets:119 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:14491 (14.4 KB)  TX bytes:55473 (55.4 KB)

caldera@VM1:~$ netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.122.1   0.0.0.0         UG        0 0          0 eth0
192.168.122.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
caldera@VM1:~$

caldera@VM1:~$ ping github.com
PING github.com (192.30.252.128) 56(84) bytes of data.
64 bytes from github.com (192.30.252.128): icmp_seq=1 ttl=50 time=100 ms
64 bytes from github.com (192.30.252.128): icmp_seq=1 ttl=50 time=100 ms
...
```

Nếu bạn tạo thêm một máy ảo, thêm 1 interface (vmnet1) sẽ được thiết lập

``` sh
# virt-install \
--name VM2 \
--ram 2048 \
--vcpu 2 \
--network network=default \
--graphics none \
--os-type linux \
--disk path=/data/vm-images/vm1.img,size=10 \
--location /tmp/ubuntu-14.04.1-server-amd64.iso \
--extra-args 'console=ttyS0,115200n8 serial'

# virsh list --all
 Id    Name                           State
----------------------------------------------------
 2     VM1                            running
 3     VM2                            running

# ifconfig vnet1
vnet1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc54:ff:feb7:1ab9  prefixlen 64  scopeid 0x20<link>
        ether fe:54:00:b7:1a:b9  txqueuelen 500  (Ethernet)
        RX packets 48  bytes 7264 (7.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 367  bytes 22146 (21.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
                                                        vnet0
                                                        vnet1
```

Đăng nhập vào VM 2 và kiểm tra kết nối tới máy VM1 cùng với kết nối ra bên ngoài internet

``` sh
# virsh console VM2
Connected to domain VM2
VM1 login: caldera
caldera@VM2:~$
caldera@VM2:~$ netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.122.1   0.0.0.0         UG        0 0          0 eth0
192.168.122.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
caldera@VM2:~$ tracepath cisco.com
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.122.1                                         0.265ms
 1:  192.168.122.1                                         0.255ms
 2:  10.10.10.1                                            1.952ms
     ...

caldera@VM2:~$ tracepath  192.168.122.91 (VM1)
 1?: [LOCALHOST]                                         pmtu 1500
 1:  VM1                                                   0.660ms reached
 1:  VM1                                                   0.396ms reached
     Resume: pmtu 1500 hops 1 back 1
caldera@VM2:~$
```

Trên máy chủ vật lí, kiểm tra lại trạng thái của bridge

``` sh
# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
                                                        vnet0
                                                        vnet1
```

## Bridged Mode

Để thiết lập một virtual network mới sử dụng bridged mode, trước tiên hãy vô hiệu hóa chế độ NAT mặc định

``` sh
# virsh net-destroy default
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
```

Tạo file xml mới cho bridge network với tên gọi mybridge0

``` sh
# cp /etc/libvirt/qemu/networks/default.xml /etc/libvirt/qemu/networks/bridged_network.xml
# virsh net-define /etc/libvirt/qemu/networks/bridged_network.xml
Network bridged_network defined from /etc/libvirt/qemu/networks/bridged_network.xml
# virsh net-edit bridged_network
# cat /etc/libvirt/qemu/networks/bridged_network.xml
<network>
  <name>bridged_network</name>
  <uuid>6579db1f-ca49-4e51-a116-c1342cd5d47a</uuid>
  <forward mode='bridge'/>
  <bridge name='mybridge0'/>
</network>

# virsh net-start bridged_network
Network bridged_network started

# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 bridged_network      active     yes           yes
```

Tạo một bridge mới với tên gọi mybridge0

``` sh
# brctl addbr mybridge0
# brctl show
bridge name     bridge id               STP enabled     interfaces
mybridge0       8000.0024810d318d       no
```

Chỉnh sửa file scripts của bridge bên trên. Các thông số thường được lấy từ card mạng thật

``` sh 
# vi /etc/sysconfig/network-scripts/ifcfg-mybridge0
DEVICE=mybridge0
TYPE=Bridge
BOOTPROTO=none
IPADDR=10.10.10.98
PREFIX=24
GATEWAY=10.10.10.1
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
ONBOOT=yes
DELAY=0
NM_CONTROLLED=no
```

Thay đổi card mạng thật (xóa ip, netmask, dns, gateway) để biến nó trở thành unumbered interface

``` sh
# vi /etc/sysconfig/network-scripts/ifcfg-enp0s25
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=enp0s25
UUID=48d129a3-89df-4f8b-9b99-5e3518edc111
ONBOOT=yes
HWADDR=00:24:81:0D:3F:8D
BRIDGE=mybridge0
NM_CONTROLLED=no
IPV4_FAILURE_FATAL=no
```

Gán card mạng thật cho bridge và reset lại dịch vụ

``` sh
# brctl addif mybridge0 enp0s25
# brctl show
bridge name     bridge id               STP enabled     interfaces
mybridge0       8000.0024810d318d       no              enp0s25

# systemctl restart network
# ifconfig mybridge0
mybridge0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.98  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 fe80::224:81ff:fe0d:318d  prefixlen 64  scopeid 0x20<link>
        ether 00:24:81:0d:3f:8d  txqueuelen 0  (Ethernet)
        RX packets 232483  bytes 612684572 (584.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 212561  bytes 17767763 (16.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Tạo máy ảo sử dụng bridged network

``` sh
# virt-install \
--name VM3 \
--ram 2048 \
--vcpu 2 \
--network network=bridged_network \
--graphics none \
--os-type linux \
--disk path=/data/vm-images/vm3.img,size=10 \
--location /tmp/ubuntu-14.04.1-server-amd64.iso \
--extra-args 'console=ttyS0,115200n8 serial'

# virsh list-all
 Id    Name                           State
----------------------------------------------------
 6     VM3                            running
 -     VM1                            shut off
 -     VM2                            shut off
```

Đăng nhập vào máy ảo và kiểm tra kết nối bằng  cùng một interface với host vật lí

``` sh
ubuntu login: caldera
Password:
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.13.0-55-generic x86_64)
...
caldera@ubuntu:~$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 52:54:00:33:cc:01
          inet addr:10.10.10.106  Bcast:10.10.10.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe33:cc01/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:81 errors:0 dropped:0 overruns:0 frame:0
          TX packets:30 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:8247 (8.2 KB)  TX bytes:3057 (3.0 KB)

caldera@ubuntu:~$ ping github.com
PING github.com (192.30.252.131) 56(84) bytes of data.
64 bytes from github.com (192.30.252.131): icmp_seq=1 ttl=50 time=101 ms
64 bytes from github.com (192.30.252.131): icmp_seq=2 ttl=50 time=100 ms
^C
caldera@ubuntu:~$ netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.10.10.1      0.0.0.0         UG        0 0          0 eth0
10.10.10.0      0.0.0.0         255.255.255.0   U         0 0          0 eth0
```

Cuối cùng, kiểm tra interface mới `vmnet0` được tạo và trở thành một phần của bridge.

``` sh 
# ifconfig vnet0
vnet0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc54:ff:fe33:cc01  prefixlen 64  scopeid 0x20<link>
        ether fe:54:00:33:cc:01  txqueuelen 500  (Ethernet)
        RX packets 1556  bytes 147144 (143.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3892  bytes 3763246 (3.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# brctl show
bridge name     bridge id               STP enabled     interfaces
mybridge0       8000.0024810d318d       no              enp0s25
                                                        vnet0
```

Lưu ý rằng các loại virtual network khác nhau có thể được tạo ra trên cùng 1 host vật lý

``` sh
# virsh net-start default
Network default started
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 bridged_network      active     yes           yes
 default              active     yes           yes
 
# virsh start VM1
Domain VM1 started

# virsh start VM2
Domain VM2 started

# virsh list --all
 Id    Name                           State
----------------------------------------------------
 6     VM3                            running
 7     VM1                            running
 8     VM2                            running
```

## Routed Mode

Routed mode giống như một biến thể của chế độ NAT. Ở chế độ này, `vibr0` sẽ hoạt động như 1 routing interface mà không dùng NAT. DHCP có thể có hoặc không tùy thuộc vào yêu cầu.

Tạo môi trường cho routed virtual networking

``` sh
# cat /etc/libvirt/qemu/networks/router.xml
<network>
  <name>router</name>
  <uuid>186ffd11-ce2d-452a-9970-baf07b723de8</uuid>
  <forward mode='route'/>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:56:e6:21'/>
  <ip address='192.168.132.1' netmask='255.255.255.0'>
  </ip>
</network>

# virsh net-define /etc/libvirt/qemu/networks/router.xml
Network router defined from /etc/libvirt/qemu/networks/router.xml

# virsh net-edit router
Network router XML configuration not changed.

# virsh net-start router
Network router started

# virsh net-autostart router
Network router marked as autostarted

# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
 router               active     yes           yes

# ifconfig virbr1
virbr1: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.132.1  netmask 255.255.255.0  broadcast 192.168.132.255
        ether 52:54:00:46:e6:21  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# brctl show virbr1
bridge name     bridge id               STP enabled     interfaces
virbr1          8000.52540046e621       yes             virbr1-nic

# netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.10.10.1      0.0.0.0         UG        0 0          0 enp0s25
10.10.10.0      0.0.0.0         255.255.255.0   U         0 0          0 enp0s25
192.168.122.0   0.0.0.0         255.255.255.0   U         0 0          0 virbr0
192.168.132.0   0.0.0.0         255.255.255.0   U         0 0          0 virbr1
```

VM bắt buộc phải được cài với IP tĩnh và default gateway cũng phải trùng nhau. Vì thế cơ chế NAT sẽ không được sử dụng.

## Spawning multiple hosts

Linux virtual network có thể sinh ra ở nhiều máy chủ. Cùng xây dựng mạng ảo trên 2 host vật lí, mỗi host có một vài VMs chạy trên đó. Chỉ có duy nhất 1 host có interface kết nối với internet, đó là NAT gateway cho cả hai hosts. Hai host kết nối với nhau thông qua một interface thứ 2.

``` sh
 caldera02            caldera03
----------            ----------
|   VM1  |            |   VM1  |
|   VM2  |============|   VM2  |
----------            ----------
    ||
 Internet
```

Ở host có gateway, kích hoạt virtual network mặc định dưới chế độ NAT

``` sh
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes

# cat /etc/libvirt/qemu/networks/default.xml
<network>
  <name>default</name>
  <uuid>f6514662-26e5-431e-8544-fd216bed9312</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:53:d4:ee'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

Tạo và khởi chạy 2 máy ảo rồi kiểm tra trạng thái của bridge

``` sh
# virsh start VM1
# virsh start VM2
# virsh list --all
 Id    Name                           State
----------------------------------------------------
 16    VM1                            running
 17    VM2                            running
 
# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             virbr0-nic
                                                        vnet2
                                                        vnet3
```

Trên host thứ 2, kích hoạt interface thứ 2 dưới chế độ isolated (các máy chỉ có thể giao tiếp với nhau, chế độ này còn được gọi là private bridge)

``` sh
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes

cat /etc/libvirt/qemu/networks/default.xml
<network>
  <name>default</name>
  <uuid>e8849d67-243a-4b0f-83ea-bc2b447580cf</uuid>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:22:1b:5f'/>
</network>
```

Ở chế độ isolated, file xml sẽ không có `<forward mode=''/>`. Bên cạnh đó, việc không có `<ip address='' netmask=''>` đồng nghĩa với việc interface này sẽ không nhận bất cứ địa chỉ IP nào.

Tạo và khởi chạy 2 máy ảo

``` sh
# virsh start VM1
# virsh start VM2
# virsh list --all
 Id    Name                           State
----------------------------------------------------
 16    VM1                            running
 17    VM2                            running

# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.c46e1f0099cd       yes             virbr0-nic
                                                        vnet0
                                                        vnet1
```

Giờ là lúc gán hai host vật lí với một card mạng thật có tên `ens2` . `ens2` sẽ được loại bỏ một số thông tin để trở thành `unumbered` và gán vào bridge

``` sh
# cat /etc/sysconfig/network-scripts/ifcfg-ens2
TYPE=Ethernet
BOOTPROTO=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=ens2
UUID=452ee132-35a6-4c36-915b-91b85902f2b9
ONBOOT=yes
NM_CONTROLLED=no

#  brctl addif virbr0 ens2
#  brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540053d4ee       yes             ens2
                                                        virbr0-nic
                                                        vnet2
                                                        vnet3
```

Làm tương tự với host còn lại

``` sh
# cat /etc/sysconfig/network-scripts/ifcfg-ens2
TYPE=Ethernet
BOOTPROTO=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=ens2
UUID=1de92d8b-1c83-4bf5-be22-171f65b1b66b
ONBOOT=yes
NM_CONTROLLED=no

#  brctl addif virbr0 ens2
# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.c46e1f0099cd       yes             ens2
                                                        vnet0
                                                        vnet1
```

Trên host calendar03, đăng nhập vào máy ảo và kiểm tra xem đã nhận IP từ host calendar02 chưa đồng thời kiểm tra kết nối tới internet

``` sh
# virsh console VM1
Connected to domain VM1
Escape character is ^]

caldera@ubuntu:~$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 52:54:00:36:5a:aa
          inet addr:192.168.122.101  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4975 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3031 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:6741689 (6.7 MB)  TX bytes:368948 (368.9 KB)

caldera@ubuntu:~$ tracepath cisco.com
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.122.1                                         0.686ms
 1:  192.168.122.1                                         0.670ms
 2:  10.10.10.1                                            4.425ms
```