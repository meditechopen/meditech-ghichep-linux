# Sao lưu dữ liệu

Lệnh rsync được sử dụng để đồng bộ hóa toàn bộ cây thư mục. Về cơ bản, nó sao chép tập tin như là lệnh cp làm. Ngoài ra, rsync kiểm tra nếu tập tin đang được sao chép đã tồn tại. Nếu tập tin tồn tại và không có thay đổi trong kích thước hoặc thời gian sửa đổi, rsync sẽ không tạo bản sao để tránh lãng phí thời gian. Hơn nữa, vì rsync chỉ sao chép các phần của các tệp tin đã thay đổi , vì thế tốc độ sao chép của nó rất nhanh

Rsync rất hiệu quả khi sao chép đệ quy một cây thư mục, vì nó chỉ sao chép những thứ khác biệt.Khi muốn đồng bộ một cách nguyên bản với cây thư mục gốc, sử dụng tùy chọn rsync-r để sao chép đệ quy tất cả các thư mục và các tệp tin như cây thư mục gốc :

```
# rsync -ravzh project_ABC /data/backups
sending incremental file list
project_ABC/
project_ABC/file1.txt
project_ABC/file2.txt
project_ABC/file3.txt
project_ABC/file4.txt

sent 636 bytes  received 92 bytes  1.46K bytes/sec
total size is 452  speedup is 0.62
```

# Nén dữ liệu

Dữ liệu tệp thường được nén để tiết kiệm không gian đĩa và giảm thời gian cần để truyền tệp qua mạng. Linux sử dụng một số phương pháp để thực hiện việc nén này.

| Câu lệnh | Cách dung |
|-----------|-------|
| gzip | Tiện ích nén Linux được sử dụng nhiều nhất |
| bzip2 | Tạo ra các tệp có dung lượng nhỏ hơn gzip |
| xz | Tiện ích nén không gian hiệu quả nhất được sử dụng trong Linux. Nó hiện được sử dụng bởi kernel.org để lưu trữ lưu trữ của hạt nhân Linux |
| zip | Thường phải kiểm tra và giải nén lưu trữ từ các hệ điều hành khác |

Những kỹ thuật này khác nhau về hiệu quả nén (dung lượng sau nén) và nén trong bao lâu : Nói chung các kỹ thuật hiệu quả hơn sẽ mất thời gian hơn. Thời gian nén không thay đổi nhiều theo các phương pháp hác nhau .

# Lưu trữ dữ liệu

Lệnh tar cho phép bạn tạo hoặc trích xuất các tệp tin từ một tệp lưu trữ, thường được gọi là tarball. Đồng thời bạn có thể tùy ý nén trong khi tạo kho lưu trữ, và giải nén trong khi trích xuất nội dung của nó.

Dưới đây là một số ví dụ về việc sử dụng tar:

| Câu lệnh | Cách dùng |
|------|-----------|
| tar xvf mydir.tar | Trích xuất tất cả các tệp tin trong mydir.tar vào thư mục mydir|
| tar zcvf mydir.tar.gz mydir | Tạo tệp lưu trữ và nén bằng gzip|
| tar jcvf mydir.tar.bz2 myd | Tạo kho lưu trữ và nén với bz2|
|  tar xvf mydir.tar.gz | Trích xuất tất cả các tệp tin vào mydir.tar.gz vào trong thư mục mydir |
| tar cvf mydir.tar | Hiển thị nội dung vào trong thư mục mydir |

# Sao chép đĩa

Lệnh dd rất hữu ích cho việc tạo bản sao của không gian đĩa gốc. Ví dụ, để sao lưu Master Boot Record (MBR) (sector 512 byte đầu tiên trên đĩa có chứa một bảng mô tả các phân vùng trên đĩa đó), sử dụng:

```
# dd if=/dev/sda of=sda.mbr bs=512 count=1
```

Để sử dụng dd để tạo một bản sao của đĩa này sang đĩa khác, xóa tất cả mọi thứ đã tồn tại trên đĩa thứ hai, sử dụng:

```
# dd if=/dev/sda of=/dev/sdb
```

Một bản sao của thiết bị đĩa đầu tiên được tạo ra trên thiết bị đĩa thứ hai.


```
# dd if=/dev/sdb of=./backup.img
```

Tháo Thẻ CF, lắp mới và tạo một bản sao mới

```
# dd if=./backup.img of=/dev/sdc
```
