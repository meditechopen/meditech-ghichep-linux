# System services

Systemd là một init system mới cho các bản phân phối gần đây của Linux, nó thay thế cho init cũ `/etc/init.d/scripts` . 

Init process là một tiến trình được khởi động lên đầu tiên trong hệ thống Linux. Tức là sau khi bạn chọn hệ điều hành trong menu của Boot Loader. Hệ điều hành bắt đầu được khởi động và tiến trình đầu tiên khởi động lên là init. Nhiệm vụ của init là start và stop các process, services… cần thiết khác.

Có ba kiểu triển khai init system chính trong hệ thống Linux là:

- System V : là phiên bản truyền thống của init system trên nhiều hệ thống Linux.
- Upstart: Được phát triển bởi Canonical vào khoảng năm 2009 và sử dụng trong các phiên bản Ubuntu cũ hơn bản 15.04.
- Systemd: Là một init system được phát triển khoảng năm 2010 và được nhiều Linux distributions sử dụng để thay thế các init system cũ. Ubuntu từ phiên bản 15.04 và Centos từ phiên bản 7 đã sử dụng systemd làm init system mặc định.

Dưới đây là bảng so sánh sysv và systemd

<img src="http://i.imgur.com/2FLLl2j.jpg">

Click vào [đây](http://images.linoxide.com/systemd-vs-sysVinit-cheatsheet.jpg) để xem ảnh gốc. 

Systemd cung cấp nhiều tính năng mạnh mẽ trong việc khởi chạy, kết thúc và quản lí các tiến trình. Dưới đây là một ví dụ về việc cài đặt MineCraft service cho systemd. MineCraft là một trò chơi được lập trình từ ngôn ngữ Java và được phát hành bởi Mojang.

Đầu tiên cần cài đặt game và môi trường để chạy nó:

``` sh
# yum install java-1.8.0-openjdk.x86_64
# which java
/bin/java
# mkdir /root/Minecraft
# cd /root/Minecraft
# wget -O minecraft_server.jar https://s3.amazonaws.com/Minecraft.Download/versions/1.8.6/minecraft_server.1.8.6.jar
# ls -lrt
-rw-r--r--. 1 root root 9780573 May 25 11:47 minecraft_server.jar
-rw-r--r--. 1 root root       2 Jun  1 11:48 whitelist.json
-rw-r--r--. 1 root root     180 Jun  1 12:01 eula.txt
drwxr-xr-x. 2 root root    4096 Jun  1 16:09 logs
-rw-r--r--. 1 root root     785 Jun  1 16:09 server.properties
-rw-r--r--. 1 root root       2 Jun  1 16:09 banned-players.json
-rw-r--r--. 1 root root       2 Jun  1 16:09 banned-ips.json
-rw-r--r--. 1 root root       2 Jun  1 16:09 ops.json
-rw-r--r--. 1 root root     109 Jun  1 16:10 usercache.json
drwxr-xr-x. 8 root root    4096 Jun  1 16:37 world
```

MineCraft server có thể được khởi chạy bằng command line : 

`# java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui`

Ngoài ra, file cấu hình cũng có thể được tạo để khởi chạy, kết thúc và kiểm tra trạng thái của server bằng việc sử dụng `systemctl`

``` sh
# vi /lib/systemd/system/minecraftd.service
[Unit]
Description=Minecraft Server
After=syslog.target network.target

[Service]
Type=simple
WorkingDirectory=/root/Minecraft
ExecStart=/bin/java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui
SuccessExitStatus=143
Restart=on-failure

[Install]
WantedBy=multi-user.target

# systemctl start minecraftd
# systemctl status minecraftd
minecraftd.service - Minecraft Server
   Loaded: loaded (/usr/lib/systemd/system/minecraftd.service; disabled)
   Active: active (running) since Mon 2015-06-01 16:00:12 UTC; 18s ago
 Main PID: 20975 (java)
   CGroup: /system.slice/minecraftd.service
           └─20975 /bin/java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui

# systemctl stop minecraftd
```

Lưu ý rằng `SuccessExitStatus=143` được yêu cầu khi một tiến trình không xử lí được tín hiệu exit đúng cách. Đây là lỗi do lập trình và nó rất phổ biến trong các ứng dụng Java. Để tránh trạng thái lỗi khi kết thúc dịch vụ, `143` cần phải được thêm vào để báo trạng thái exit thành công.

`systemctl` còn có thể được sử dụng để kích hoạt/vô hiệu hóa việc khởi động dịch vụ cùng hệ thống :

``` sh
# systemctl enable minecraftd
ln -s '/usr/lib/systemd/system/minecraftd.service' '/etc/systemd/system/multi-user.target.wants/minecraftd.service'
# systemctl is-enabled minecraftd
enabled
# systemctl disable minecraftd
```

Dưới đây là một ví dụ khác:

``` sh
# cat /etc/systemd/system/redmined.service
[Unit]
Description=Redmine Server
After=syslog.target network.target

[Service]
Type=simple
PermissionsStartOnly=true
WorkingDirectory=/home/redmine/redmine
ExecStartPre=/usr/sbin/iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
ExecStart=/usr/bin/ruby bin/rails server -b 0.0.0.0 -p 8080 webrick -e production
User=redmine
Group=redmine
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=redmined
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```