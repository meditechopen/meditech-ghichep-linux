# Hệ thống Quản lý Gói

Các phần cốt lõi của bản phân phối Linux và hầu hết các phần mềm bổ sung được cài đặt thông qua Hệ thống Quản lý Gói. Mỗi gói chứa các tập tin và các hướng dẫn khác cần thiết để làm cho một thành phần phần mềm hoạt động trên hệ thống.Các gói có thể phụ thuộc lẫn nhau . Có 2 loại nhóm bao gồm các gói quản lý :  Những thứ dựa trên dpkg và những thứ sử dụng rpm như quản lý cấp thấp .

Hai hệ thống này không tương thích, nhưng cung cấp các tính năng tương tự ở mức độ rộng.

# Hệ thống Quản lý Gói

|Công cụ cấp cao | Công cụ cấp thấp | Họ |
|-------------|-------------|-----------|
| apt-get | dpkg | Debian |
|zypper | rpm | SUSE |
|yum |rpm | RedHat|

Cả hai hệ thống quản lý gói cung cấp hai cấp công cụ: một công cụ cấp thấp (như dpkg hoặc rpm), chú ý đến các chi tiết của việc giải nén các gói riêng lẻ, chạy các tập lệnh, cài đặt phần mềm chính xác, trong khi một công cụ cấp cao (như Apt-get, yum, hoặc zypper) làm việc với các nhóm gói, tải các gói từ nhà cung cấp, và tính toán các sự phụ thuộc. Hầu hết người dùng chỉ cần làm việc với công cụ cấp cao, sẽ cần gọi công cụ cấp thấp nếu cần. Theo dõi phụ thuộc là một tính năng đặc biệt quan trọng của công cụ cấp cao vì nó xử lý các chi tiết về việc tìm và cài đặt từng phụ thuộc cho bạn. Hãy cẩn thận, tuy nhiên, như cài đặt một gói duy nhất có thể dẫn đến hàng chục hoặc thậm chí hàng trăm gói phụ thuộc đang được cài đặt.

| Hoạt động | RPM | Debian |
|--------------|-------|-----------|
|Cài đặt một gói |rpm –i foo.rpm |dpkg --install foo.deb |
|Cài đặt một gói với phụ thuộc từ kho lưu trữ |yum install foo |apt-get install foo |
|Loại bỏ một gói |rpm –e foo.rpm |dpkg --remove foo.deb|
|Loại bỏ một gói và phụ thuộc bằng cách sử dụng kho |yum remove foo |apt-get remove foo |
|Cập nhật gói lên phiên bản mới hơn|rpm –U foo.rpm |dpkg --install foo.deb|
|Cập nhật gói sử dụng kho lưu trữ và giải quyết các sự phụ thuộc |yum update foo |apt-get upgrade foo |
|yum update foo |yum update foo |apt-get dist-upgrade |
|Hiển thị tất cả gói đã cài đặt |yum list installed |dpkg --list |
|Nhận thông tin về một gói cài đặt bao gồm các tệp rpm |rpm –qil foo |rpm –qil foo |
|Nhận thông tin về một gói cài đặt bao gồm các tệp |rpm –qil foo |dpkg --listfiles foo |
|Hiển thị gói có sẵn với "foo" trong tên |yum list foo |apt-cache search foo |
|Hiển thị tất cả các gói có sẵn |yum list| apt-cache dumpavail|
|Hiển thị gói một tập tin thuộc về|rpm –qf file |dpkg --search file |
