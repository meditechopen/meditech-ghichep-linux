# Trao đổi bộ nhớ Linux

Linux chia RAM vào các vùng nhớ được gọi là trang. Swapping là quá trình trong đó một trang bộ nhớ được sao chép vào trong một không gian được cấu hình sẵn trên đĩa cứng, được gọi là swap, nơi cung cấp bộ nhớ.Kết hợp kích thước của bộ nhớ và trao đổi không gian vật lý thành dung lượng bộ nhớ ảo có sẵn. Trao đổi là cần thiết vì có 2 lý do :
1. Thứ nhất, khi hệ thống đòi hỏi nhiều bộ nhớ hơn bộ nhớ có sẵn, kernel sẽ di chuyển các trang ít được sử dụng trong không gian trao đổi và cấp việc sử dụng bô nhớ RAM cho các ứng dụng (tiến trình) đòi hỏi bộ nhớ.
2. Thứ hai, một số lượng đáng kể các trang được sử dụng bởi một ứng dụng chỉ được sử dụng trong giai đoạn khởi động của nó và sau đó không bao giờ được sử dụng.

Hệ thống này sau đó có thể dử dụng hoán đổi trên các trang và để giải phóng bộ nhớ cho các ứng dụng khác  hoặc thậm chí cho bộ nhớ cache . Tuy nhiên , trao đổi bộ nhớ có một nhược điểm. So với bộ nhớ RAM , ổ đĩa chậm hơn rất nhiều. Tốc độ bộ nhớ được đo bằng nano giây, trong khi ổ đĩa tính bằng mili giây,ổ đĩa có tốc độ truy cập chậm hơn hàng chục, hàng ngàn lần so với RAM. Trao đổi xảy ra càng nhiều thì hệ thống càng chậm.Đôi khi trao đổi nhiều quá tạo ra quá tải , bởi vì có một tình huống đặc biệt,một trang được đưa vào trao đổi, sau đó nhanh chóng đưa đến RAM ,điều này diễn ra liên tục. Trong trường hợp như vậy, các tranh chấp trong hệ thống để tìm ra bộ nhớ trống và duy trì các ứng dụng khác nhau chạy cùng lúc liên tục diễn ra. Trong trường hợp này chỉ cần bổ sung thêm RAM để ổn định hệ thống.

```
# swapon -s
Filename                                Type            Size    Used    Priority
/dev/dm-0                               partition       4079612 0       -1
```

Mỗi hàng liệt kê một phân vùng swap riêng biệt được sử dụng bởi hệ thống. Một đặc thù của swap trên Linux đó là cho dù kết nối 2 hoặc nhiều hơn không gian hoán đổi ( tốt nhất là trên 2 thiết bị khác nhau ) với cùng các ưu tiên như nhau, linux chia hoạt động trao đổi của nó giữa chúng. Điều này tăng hiệu suất đáng kể. Để thêm một phân vùng hệ thống swap cho hệ thống của bạn, nhưng trước tiên bạn cần phải chuẩn bị nó.

# Thêm vùng trao đổi dưới dạng tệp

```
dd if=/dev/zero of=/var/swapfile bs=1M count=2048
chmod 600 /var/swapfile
mkswap /var/swapfile
echo /var/swapfile none swap defaults 0 0 | sudo tee -a /etc/fstab
swapon -a
```
