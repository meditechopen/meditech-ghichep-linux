# Cấp phép tệp tin

Trong Linux và các hệ điều hành UNIX khác, mọi tệp đều được liên kết với người dùng là chủ sở hữu. Mỗi tập tin cũng được liên kết với một nhóm có quan tâm đến tập tin và một số quyền, hoặc cho phép: đọc, viết và thực thi.

| Câu lệnh | Kết quả|
|-----------|--------|
| chown | Dùng để thay đổi chủ sở hữu của một tệp tin hoặc thư mục |
| chgrp | Dùng để thay đổi nhóm sở hữu |
| chmod | Được sử dụng để thay đổi các quyền trên tệp|

Các tập tin có ba loại quyền: read (r), write (w), execute (x). Chúng thường được biểu diễn theo thứ tự sau rwx. Các quyền này ảnh hưởng đến ba nhóm chủ sở hữu: người dùng (u), nhóm (g) và những người khác (o). Kết quả là, bạn có ba nhóm ba quyền sau:

| rwx: | rwx: |rwx |
|-------|-----|--------|
|u: | g: | o |

Có một số cách khác nhau để sử dụng lệnh chmod. Ví dụ, để cho chủ sở hữu thực hiện sự cho phép:

```
$ ls -l test1
-rw-rw-r-- 1 joy caldera 1601 Mar 9 15:04 test1
$ chmod u+x test1
$ ls -l test1
-rwxrw-r-- 1 joy caldera 1601 Mar 9 15:04 test1
```

Loại cú pháp này có thể khó gõ và ghi nhớ, do đó, một người thường sử dụng một phép viết tắt cho phép bạn thiết lập tất cả các điều khoản trong một bước. Điều này được thực hiện với một thuật toán đơn giản, và một chữ số duy nhất đủ để xác định tất cả ba bit quyền cho mỗi thực thể. Chữ số này là tổng của:

- 4 : được phép đọc .
- 2 : được phép ghi.
- 1 :được phép thực thi.

Như vậy 7 có nghĩa là đọc + viết + thực thi, 6 có nghĩa là đọc + viết, và 5 có nghĩa là đọc + thực thi.

Khi bạn áp dụng lệnh này vào lệnh chmod, bạn phải đưa ra ba chữ số cho mỗi mức độ tự do, như trong

```
$ chmod 755 test1
$ ls -l test1
-rwxr-xr-x 1 joy caldera 1601 Mar 9 15:04 test1
```

Quyền sở hữu nhóm được thay đổi bằng cách sử dụng lệnh chgrp

```
# ll /home/mina/myfile.txt
-rw-rw-r--. 1 mina caldera 679 Feb 19 16:51 /home/mina/myfile.txt
# chgrp root /home/mina/myfile.txt
# ll /home/mina/myfile.txt
-rw-rw-r--. 1 mina root 679 Feb 19 16:51 /home/mina/myfile.txt
```
