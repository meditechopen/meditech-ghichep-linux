# 24. Cluster Basics

Clustering là một kiến trúc nhằm đảm bảo nâng cao khả năng sẵn sàng cho các hệ thống mạng máy tính. Clustering cho phép sử dụng nhiều máy chủ kết hợp với nhau tạo thành một cụm có khả năng chịu đựng hay chấp nhận sai sót (fault-tolerant) nhằm nâng cao độ sẵn sàng của hệ thống mạng. Cluster là một hệ thống bao gồm nhiều máy chủ được kết nối với nhau theo dạng song song hay phân tán và được sử dụng như một tài nguyên thống nhất.

Nếu một máy chủ ngừng hoạt động do bị sự cố hoặc để nâng cấp, bảo trì, thì toàn bộ công việc mà máy chủ này đảm nhận sẽ được tự động chuyển sang cho một máy chủ khác (trong cùng một cluster) mà không làm cho hoạt động của hệ thống bị ngắt hay gián đoạn. Quá trình này gọi là “fail-over”; và việc phục hồi tài nguyên của một máy chủ trong hệ thống (cluster) được gọi là “fail-back”. Các máy từ phía client sẽ không thể nhìn thấy node bị lỗi trong hệ thống cluster.

Với Linux, có rất nhiều công cụ cho phép người dùng xây dựng hệ thống có tính sẵn sàng cao và **Peacemaker** chính là tool được sử dụng nhiều nhất. Đối với các hệ thống cluster sử dụng Peacemaker, nó sẽ có các daemon riêng biệt để giám sát từng node, các scripts để quản lí dịch vụ và cả hệ thống quản lí tài nguyên. Sau đây là các thành phần tạo nên cấu trúc của Peacemaker:

1. Cluster Information Base: máy chủ Pacemaker phân phối và đồng bộ thông tin cấu hình cluster và thông tin trạng thái từ Designated Coordinator (DC) của cluster tới tất cả các cluster member khác. DC là một cluster member được chỉ định để lưu trữ trạng của cluster.
2. Cluster Resource Management Daemon: Các tài nguyên cluster được quản lí bởi thành phần này có thể được truy vấn, di chuyển và thay đổi khi cần thiết từ clients. Mỗi một node cũng có một công cụ quản lí tài nguyên riêng và nó sẽ hoạt động như một interface giúp kết nối giữa tài nguyên của node đó với Cluster Resource Management Daemon. 
3. Fencing Manager: thường được triển khai kết hợp với bộ chuyển đổi nguồn cung cấp năng lượng, thành phần này hoạt động như cluster resource trong Peacemaker để xử lý các yêu cầu về hàng rào,  loại bỏ các node bị chết khỏi cluster  để đảm bảo tính toàn vẹn dữ liệu. Pacemaker sử dụng một kỹ thuật có tên là STONITH (Shoot The Other Node In The Head) nhằm ngăn chặn dữ liệu bị hỏng gây ra bởi các node bị lỗi trong cluster không đáp ứng yêu cầu nhưng vẫn truy cập dữ liệu ứng dụng (được gọi là "kịch bản phân chia não").

## Cài đặt hệ thống cluster đơn giản

Pacemaker yêu cầu một chương trình chạy nền có mục đích giao tiếp (gọi là Corosync). Nó tạo ra kết nối giữa các node và mô hình truyền thông bí mật để tạo ra replicated state machines mà Pacemaker có thể chạy trên đó. Corosync có thể được xem như là hệ thống cơ sở kết nối các cluster nodes với nhau, trong khi Pacemaker theo dõi cluster và có hành động trong trường hợp thất bại. Ngoài ra, chúng ta sẽ sử dụng PCS, một giao diện dòng lệnh tương tác với cả Corosync và Pacemaker.

Dưới đây là concepts chung của Linux Clustering

``` sh
                                  |
+----------------------+          |          +----------------------+
| Node01               |          |          | Node02               |
| holly.noverit.com    +----------+----------+ benji.noverit.com    |
| 10.10.10.22          |                     | 10.10.10.24          |
+----------------------+                     +----------------------+
```

Cài đặt, khởi động Peacemaker và PCS trên cả 2 nodes. Bởi vì Corosync phụ thuộc vào Peacemaker nên tốt nhất ta nên cài Peacemaker trước và để hệ thống tự quyết định phiên bản Corosync.

``` sh
[root@holly ~]# yum -y install pacemaker
[root@holly ~]# yum -y install pcs
[root@holly ~]# systemctl start pcsd
[root@holly ~]# systemctl enable pcsd

[root@benji ~]# yum -y install pacemaker
[root@benji ~]# yum -y install pcs
[root@benji ~]# systemctl start pcsd
[root@benji ~]# systemctl enable pcsd
```

Peacemaker cần sự giao tiếp giữa các nodes, vì vậy cần enable port trên từng node, mặc định đó là port 2224 sử dụng TCP. Nếu không, bạn cũng có thể tắt firewall trên cả 2 nodes đi.

``` sh
[root@holly ~]# systemctl stop firewalld
[root@holly ~]# systemctl disable firewalld
[root@benji ~]# systemctl stop firewalld
[root@benji ~]# systemctl disable firewalld
```

PCS sẽ tạo ra user `hacluster` trong quá trình cài đặt, user này không có mật khẩu. Ta cần phải đặt mật khẩu cho nó trên cả 2 máy chủ. Điều này sẽ cho phép PCS có thể thực hiện một số tác vụ như đồng bộ cấu hình Corosync cũng như start và stop cluster.

``` sh
[root@holly ~]# passwd hacluster
Changing password for user hacluster
[root@benji ~]# passwd hacluster
Changing password for user hacluster
```

Sử dụng cùng 1 password cho cả hai máy chủ. Mật khẩu này sẽ được dùng để cấu hình trong các bước tiếp theo. Lưu ý rằng đây chỉ là tài khoản của PCS và nó sẽ không thể dùng để đăng nhập vào máy chủ.

``` sh
[root@holly ~]# pcs cluster auth holly benji
Username: hacluster
Password:
holly: Authorized
benji: Authorized
```

Với những node giống nhau, thực hiện việc đồng bộ hóa cấu hình Corosync

``` sh
[root@holly ~]# pcs cluster setup --name mycluster holly benji
Shutting down pacemaker/corosync services...
Redirecting to /bin/systemctl stop  pacemaker.service
Redirecting to /bin/systemctl stop  corosync.service
Killing any remaining services...
Removing all cluster configuration files...
holly: Succeeded
benji: Succeeded
Synchronizing pcsd certificates on nodes holly, benji...
benji: Success
holly: Success
Restaring pcsd on the nodes in order to reload the certificates...
benji: Success
holly: Success
```

Nó sẽ generate ra một file cấu hình đặt tại `/etc/corosync/corosync.conf`

``` sh
[root@holly ~]# cat /etc/corosync/corosync.conf
totem {
    version: 2
    secauth: off
    cluster_name: mycluster
    transport: udpu
}
nodelist {
    node {
        ring0_addr: holly
        nodeid: 1
    }
    node {
        ring0_addr: benji
        nodeid: 2
    }
}
quorum {
    provider: corosync_votequorum
    two_node: 1
}
logging {
    to_logfile: yes
    logfile: /var/log/cluster/corosync.log
    to_syslog: yes
}
```

Kích hoạt và khởi động cluster cùng hệ thống

``` sh
[root@holly ~]# pcs cluster start --all
benji: Starting Cluster...
holly: Starting Cluster...

[root@holly ~]# pcs cluster enable --all
holly: Cluster Enabled
benji: Cluster Enabled
```

Kiểm tra trạng thái của cluster

``` sh
[root@holly ~]# pcs status
Cluster name: mycluster
WARNING: no stonith devices and stonith-enabled is not false
Last updated: Sat Jul 16 17:20:14 2016
Stack: corosync
Current DC: holly (version 1.1.13-10.el7_2.2-44eb2dd) - partition with quorum
2 nodes and 0 resources configured
Online: [ benji holly ]
Full list of resources: -
PCSD Status:
    holly: Online
    benji: Online
Daemon Status:
    corosync: active/enabled
    pacemaker: active/enabled
    pcsd: active/enabled
```

Một vài thông tin đáng lưu ý trong ví dụ trên:

1. Designated Coordinator (DC) là node mà ta cấu hình cluster
2. Chỉ có duy nhất 2 node và không có nơi lưu dữ liệu
3. Tên của cluster là "my cluster"
4. Các ứng dụng chạy nền: corosync, pacemaker và pcsd đều đã được active và enable
5. Fencing (stonith) đã được enable tuy nhiên không có fencing devices nào được cấu hình

Để chắc chắn rằng cả hai nodes đã nằm trong cluster

``` sh
[root@holly ~]# pcs status corosync
Membership information
----------------------
    Nodeid      Votes Name
         1          1 holly (local)
         2          1 benji
[root@holly ~]#
```

Bởi vì cluster trong ví dụ này không quản lí tài nguyên chung, nên sẽ không có bất cứ mối nguy hại nào tới dữ liệu, ta có thể disable fencing

`[root@holly ~]# pcs property set stonith-enabled=false`

Ta cũng có thể disable Cluster quorum bởi nó không có tác dụng trong một cluster chỉ có 2 nodes.

`[root@holly ~]# pcs property set no-quorum-policy=ignore`

Để xem thông tin của cluster

``` sh
[root@benji ~]# pcs property list
Cluster Properties:
 cluster-infrastructure: corosync
 cluster-name: mycluster
 dc-version: 1.1.13-10.el7_2.2-44eb2dd
 have-watchdog: false
 no-quorum-policy: ignore
 stonith-enabled: false
```

Tốt hơn hết hãy tắt cluster trước khi tắt hệ thống.

Tắt cluster ở trên 1 node

``` sh
[root@benji ~]# pcs cluster stop
Stopping Cluster (pacemaker)... Stopping Cluster (corosync)...
[root@benji ~]# pcs cluster status
Error: cluster is not currently running on this node
```

Tắt cluster trên tất cả các nodes

``` sh
[root@holly ~]# pcs cluster stop --all
holly: Stopping Cluster (pacemaker)...
benji: Stopping Cluster (pacemaker)...
benji: Stopping Cluster (corosync)...
holly: Stopping Cluster (corosync)...
[root@holly ~]#
```

## Gán resource cho cluster

Cài đặt và cấu hình HTTP server cho cả hai node. Lưu ý: không cần phải kích hoạt dịch vụ

``` sh
[root@benji ~]# yum install -y httpd
[root@benji ~]# echo "Hello Benji" > /var/www/html/index.html
[root@holly ~]# yum install -y httpd
[root@holly ~]# echo "Hello Holly" > /var/www/html/index.html
```

Add HTTP server thành resource của cluster

``` sh
[root@benji ~]# pcs resource create HTTPServer ocf:heartbeat:apache \
> configfile=/etc/httpd/conf/httpd.conf \
> op monitor interval=1min
```

Tên của resource là HTTPServer thuộc kiểu `ocf:heartbeat:apache` . Kiểu này sẽ cho cluster biết script nào được dùng, chủ của script và nó thuộc chuẩn nào. Câu lệnh này cũng cho Peacemaker biết "sức khỏe" hiện tại.

Thêm địa chỉ ip ảo như là một resource thứ 2 của cluster. Địa chỉ IP này sẽ được sử dụng bởi clients của cluster để truy cập tới HTTP server.

``` sh
[root@benji ~]# pcs resource create VirtualIP ocf:heartbeat:IPaddr2 \
> ip=10.10.10.23 \
> cidr_netmask=24 \
> op monitor interval=30s
```

Tên của resource là `VirtualIP` thuộc kiểu `ocf:heartbeat:IPaddr2`. Câu lệnh khiến Peacemaker kiểm tra "sức khỏe" của dịch vụ sau mỗi 30 giây. Virtual IP resource kết hợp địa chỉ IP được chỉ định trong câu lệnh trên với interface của chính các node sở hữu virtual IP resources. Virtual IP này sẽ truyền từ node này sang node khác tùy thuộc vào trạng thái của từng node:

``` sh
[root@benji ~]# ip addr show ens32
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:20:d2:dd brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.24/24 brd 10.10.10.255 scope global ens32
       valid_lft forever preferred_lft forever
    inet 10.10.10.23/24 brd 10.10.10.255 scope global secondary ens32
       valid_lft forever preferred_lft forever
       
[root@holly ~]# ip addr show ens32
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:77:68:56 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.22/24 brd 10.10.10.255 scope global ens32
       valid_lft forever preferred_lft forever
```

Cấu hình HTTPServer  và VirtualIP luôn nằm trên các node giống nhau

``` sh
[root@benji ~]# pcs constraint colocation add HTTPServer with VirtualIP
```

Cấu hình khởi động virtual IP trước HTTPServer. 

``` sh
[root@holly ~]# pcs constraint order VirtualIP then HTTPServer
Adding VirtualIP HTTPServer (kind: Mandatory) (Options: first-action=start then-action=start)
```

Xem trạng thái của cả hai resources

``` sh
[root@benji ~]# pcs status resources
 VirtualIP      (ocf::heartbeat:IPaddr2):       Started by benji
 HTTPServer     (ocf::heartbeat:apache):        Started by holly
```

và sự phụ thuộc lẫn nhau giữa các resource

``` sh
[root@holly ~]# pcs constraint
Location Constraints:
Ordering Constraints:
  start VirtualIP then start HTTPServer (kind:Mandatory)
Colocation Constraints:
  HTTPServer with VirtualIP (score:INFINITY)
```

Giờ đây ta có thể truy cập vào HTTPServer từ web client theo Virtual IP address 10.10.10.23

``` sh
[stack@director ~]$ curl http://10.10.10.23
Hello Benji
```

Để test cluster failover, dừng node đang active bằng tay

``` sh
[root@benji html]# pcs cluster stop
Stopping Cluster (pacemaker)... Stopping Cluster (corosync)...
```

và chắc chắn rằng resource sẽ được chuyển sang node khác

``` sh
[stack@director ~]$ curl http://10.10.10.23
Hello Holly
```

Truy cập giao diện quản lí cluster

``` sh
https://<primary_node_ip>:2224
```

Lưu ý: dùng tài khoản và mật khẩu của `hacluster` để đăng nhập.