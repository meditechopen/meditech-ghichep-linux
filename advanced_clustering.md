# 25. Advanced Clustering

Linux clustering cung cấp rất nhiều các kĩ thuật để sử dụng hầu hết các thẻ loại cluster. Ở phần [Cluster Basic](), ta đã cài đặt hệ thống Active/Standby đơn giản. Ở phần này, chúng ta sẽ mở rộng hệ thống cũ để nó trở thành Active/Active cluster.

Ở hệ thống Active/Standby, stanby node gần như không có bất cứ nhiệm vụ gì. Vì vậy không có bất cứ dữ liệu gì được chia sẻ chung giữa cả hai, cũng bởi thế không có bất cứ sự nguy hiểm gì về việc hỏng dữ liệu. Node thứ 2 hoàn toàn có thể trở thành active node và tham gia vào các tác vụ để góp phần tăng hiệu suất của hệ thống. Để làm được điều này, chúng ta sẽ khiến HTTP Server chạy trên cả hai nodes và cài đặt Load Balancer trên cả hai node.

<img src="https://raw.githubusercontent.com/kalise/Linux-Tutorial/master/img/active-active-cluster.jpg">

Xóa bỏ HTTP Server resource trên cluster

``` sh
[root@benji ~]# pcs resource delete HTTPServer
Attempting to stop: HTTPServer...Stopped
```

Thêm lại HTTP Server resource và thay đổi kiểu

``` sh
[root@benji ~]# pcs resource create httpd systemd:httpd \
> configfile=/etc/httpd/conf/httpd.conf \
> op monitor interval=30s clone
```

Việc thay đổi kiểu từ `ocf:heartbeat:apache` sang `systemd:httpd` nhằm mục đích biến HTTP Server trở thành system daemon. Nó cho phép server chạy trên cả 2 node ở cùng một thời điểm. Lưu ý rằng dịch vụ vẫn sẽ được quản lí bởi Peacemaker chứ không phải systemd.

Trên cả hai nodes, cài đặt Load Balancer. Ta sẽ sử dụng HAProxy

``` sh
[root@benji ~]# yum install haproxy -y
```

Kiểm tra để chắc chắn rằng cấu hình trên cả hai nodes là giống nhau

``` sh
[root@benji ~]# vi /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats
#---------------------------------------------------------------------
# Common defaults
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
#---------------------------------------------------------------------
# Listen configuration
#---------------------------------------------------------------------
listen apache
    bind 10.10.10.23:80 transparent #bind to the Virtual IP
    mode http
    option http-server-close
    option forwardfor
    balance roundrobin
    server holly 10.10.10.22:80 check
    server benji 10.10.10.24:80 check
```

HAProxy sẽ kết hợp Virtual IP address và sau đó chuyển tiếp request từ clients tới HTTPServer theo giải thuật Round Robin. Để tránh sự xung đột giữa HTTPServer và HAProxy, ta cần cấu hình file config 

``` sh
[root@benji ~]# vi /etc/httpd/conf/httpd.conf
...
Listen 10.10.10.24:80
...
[root@holly ~]# vi /etc/httpd/conf/httpd.conf
...
Listen 10.10.10.22:80
...
```

Vì ta cần Load Balancer chạy trên cả hai nodes để điều khiển client request nên phải thêm HAProxy resource vào cluster

``` sh
[root@benji ~]# pcs resource create haproxy systemd:haproxy \
> op monitor interval=15s clone
```

Khởi động lại cluster 

``` sh
[root@benji ~]# pcs cluster start --all
holly: Starting Cluster...
benji: Starting Cluster...
```

Và check trạng thái

``` sh
[root@benji ~]# pcs status
Cluster name: mycluster
Last updated: Mon Jul 18 01:06:01 2016 
Stack: corosync
Current DC: holly.b-cloud.it (version 1.1.13-10.el7_2.2-44eb2dd) - partition with quorum
2 nodes and 5 resources configured
Online: [ benji holly ]
Full list of resources:
    VIP-10.10.10.23        (ocf::heartbeat:IPaddr2):       Started benji
    Clone Set: httpd-clone [httpd]
         Started: [ benji holly ]
    Clone Set: haproxy-clone [haproxy]
         Started: [ benji holly ]
    PCSD Status:
      holly: Online
      benji: Online
    Daemon Status:
      corosync: active/enabled
      pacemaker: active/enabled
      pcsd: active/enabled
```

Cluster đang chạy với virtual IP trên Benji node

``` sh
[root@benji ~]#  netstat -tupln | grep 80
tcp        0      0 10.10.10.23:80          0.0.0.0:*               LISTEN      9251/haproxy
tcp        0      0 10.10.10.24:80          0.0.0.0:*               LISTEN      1729/httpd
[root@benji ~]# ip addr show ens32
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:20:d2:dd brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.24/24 brd 10.10.10.255 scope global ens32
       valid_lft forever preferred_lft forever
    inet 10.10.10.23/32 brd 10.10.10.255 scope global ens32
       valid_lft forever preferred_lft forever

[root@holly ~]# netstat -tupln | grep 80
tcp        0      0 10.10.10.23:80          0.0.0.0:*               LISTEN      18467/haproxy
tcp        0      0 10.10.10.22:80          0.0.0.0:*               LISTEN      18444/httpd
udp6       0      0 fe80::20c:29ff:fe77:123 :::*                                623/ntpd
[root@holly ~]#  ip addr show ens32
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:77:68:56 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.22/24 brd 10.10.10.255 scope global ens32
       valid_lft forever preferred_lft forever
```

Kiểm tra dịch vụ được khởi động với systemd

``` sh
[root@benji ~]# systemctl status haproxy
● haproxy.service - Cluster Controlled haproxy
   Loaded: loaded (/usr/lib/systemd/system/haproxy.service; disabled; vendor preset: disabled)
  Drop-In: /run/systemd/system/haproxy.service.d
           └─50-pacemaker.conf
   Active: active (running) since Mon 2016-07-18 01:05:55 CEST; 5min ago
 Main PID: 1748 (haproxy-systemd)
   CGroup: /system.slice/haproxy.service
           ├─1748 /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
           ├─1749 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
           └─1750 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
Jul 18 01:05:55 benji systemd[1]: Started Cluster Controlled haproxy.
Jul 18 01:05:55 benji systemd[1]: Starting Cluster Controlled haproxy...
```

``` sh
[root@benji ~]# systemctl status httpd
● httpd.service - Cluster Controlled httpd
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
  Drop-In: /run/systemd/system/httpd.service.d
           └─50-pacemaker.conf
   Active: active (running) since Mon 2016-07-18 01:05:53 CEST; 6min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1729 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─1729 /usr/sbin/httpd -DFOREGROUND
           └─1734 /usr/sbin/httpd -DFOREGROUND
Jul 18 01:05:52 benji systemd[1]: Starting Cluster Controlled httpd...
Jul 18 01:05:53 benji systemd[1]: Started Cluster Controlled httpd.
```

Thiết lập thứ tự khởi động virtual IP trước và sau đó tới các dịch vụ khác.

``` sh
[root@benji ~]# pcs constraint order VIP-10.10.10.23 then haproxy-clone
Adding VIP-10.10.10.23 haproxy-clone (kind: Mandatory) (Options: first-action=start then-action=start)
[root@benji ~]# pcs constraint order httpd-clone then haproxy-clone
Adding httpd-clone haproxy-clone (kind: Mandatory) (Options: first-action=start then-action=start)
[root@benji ~]# pcs constraint colocation add VIP-10.10.10.23 with haproxy-clone
[root@benji ~]# pcs constraint
Location Constraints:
Ordering Constraints:
  start VIP-10.10.10.23 then start haproxy-clone (kind:Mandatory)
  start httpd-clone then start haproxy-clone (kind:Mandatory)
Colocation Constraints:
  VIP-10.10.10.23 with haproxy-clone (score:INFINITY)
```

Cluster đã sẵn sàng

``` sh
[stack@director ~]$ curl http://10.10.10.23
Hello Holly
[stack@director ~]$ curl http://10.10.10.23
Hello Benji
[stack@director ~]$ curl http://10.10.10.23
Hello Holly
```