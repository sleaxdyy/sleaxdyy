ss## Khái niệm

Nginx là 1 máy chủ reverse proxy mã nguồn mở cho các giao thức HTTP, HTTPS, SMTP, POP3 và IMAP, cũng như là 1 máy chủ cân bằng tải (load balancer), HTTP cache và web. Dự án Nginx được bắt đầu với việc tập trung vào tính đồng thời cao, hiệu năng cao và sử dụng tài nguyên thấp và được phát triển bởi Igor Sysoev vào nằm 2002, được phân phối ra công chúng lần đầu vào nằm 2004.
## Authentication Nginx
*Để tạo cặp tên người dùng-mật khẩu, hãy sử dụng tiện ích tạo tệp mật khẩu, ví dụ: apache2-utils hoặc httpd-tools*
1. Xác minh rằng apache2-utils (Debian, Ubuntu) hoặc httpd-tools (RHEL / CentOS / Oracle Linux) đã được cài đặt
2. Tạo tệp mật khẩu và người dùng đầu tiên. Chạy tiện ích htpasswd với -c (để tạo tệp mới), tên đường dẫn tệp làm đối số đầu tiên và tên người dùng làm đối số thứ hai: 
```console
sudo htpasswd -c /etc/nginx/htpasswd.users user1
```
3. Tạo các cặp mật khẩu người dùng bổ sung. Bỏ qua -c vì tệp đã tồn tại:
```console
sudo htpasswd /etc/apache2/.htpasswd user2
```
- Nhấn Enter và nhập mật khẩu cho user1 tại lời nhắc.
4. Tiếp theo, chúng tôi sẽ tạo tệp cấu hình nginx:
```console
sudo vi /etc/nginx/conf.d/kibana.conf
```
```console
server {
    listen       9000;
    auth_basic "Restricted";
    auth_basic_user_file /etc/nginx/htpasswd.users;
    
    location / {
           
            proxy_pass http://192.168.0.108:5601;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
    }
}
```
- auth_basic: Cho phép xác thực tên người dùng và mật khẩu bằng giao thức “HTTP Basic Authentication”
- auth_basic_user_file: đường dẫn tới địa chỉ lưu tên người dùng và mật khẩu
- $host Biến này tham chiêu đến tên host của yêu cầu gốc, "Host" header của yêu cầu từ client , hoặc tên server khớp với yêu cầu. 
- Header X-Real-IP là địa chỉ IP của client để proxy server ra quyết định xử lí hoặc ghi log thế nào với yêu cầu

## Cấu hình Http trong Nginx

**http** : được khai báo ở phần đầu của tập tin cấu hình. Nó cho phép chúng ta định nghĩa các chỉ thị và các khối từ tất cả các module liên quan đến khía cạnh HTTP của Nginx. Khối chỉ thị này có thể được khai báo nhiều lần trong tập tin cấu hình, và các giá trị chỉ thị được chèn trong khối http sau sẽ ghi đè lên các chỉ thị nằm trong khối http trước đó

**server** : khối này cho phép chúng ta khai báo 1 website. Nói cách khác, 1 website cụ thể (được nhận diện bởi 1 hoặc nhiều hostname) được thừa nhận bới Nginx và nhận cấu hình của chính nó. Khối này chỉ có thể được dùng bên trong khối http.

**location** : cho phép chúng ta định nghĩa 1 nhóm các thiết lập được áp dụng cho 1 vị trí cụ thể trên website (thể hiện qua URL của website đó). Khối location có thể được dùng bên trong 1 khối server hoặc nằm chồng bên trong 1 khối location khác.

## Các chỉ thị
**listen** :
- Sử dụng trong khối : server
- Chỉ rõ địa chỉ IP và/hoặc port được dùng bởi socket phục vụ website. Các website thường được phục vụ trên port 80 (giá trị mặc định) qua HTTP, hoặc 443 qua HTTPS.
- Cú pháp: listen [```address```] [```:port```] [```additional options```]; 
- Các tùy chọn bổ sung:
    - ```default``` hoặc ```default_server``` : Chỉ rõ khối server này được dùng như website mặc định cho bất kỳ yêu cầu nhận được tại địa chỉ IP và port được chỉ rõ.
    - ```ssl```: Chỉ rõ website sẽ sử dụng SSL.
    - Các tùy chọn khác liên quan đến các lời gọi hệ thống bind và listen gồm: ```backlog=num, rcvbuf=size, sndbuf=size, accept_filter=filter, deferred, setfib=number, và bind```.
* Ví dụ :
    ```console
        listen 192.168.1.1:80;
        listen 127.0.0.1;
        listen 80 default;
        listen 443 ssl;
    ```
**server_name:**
- Sử dụng trong khối : server
- Đăng ký 1 hoặc nhiều hostname cho khối server. Khi Nginx nhận 1 yêu cầu HTTP, nó so sánh giá trị Host trong phần header của yêu cầu với tất cả các khối server đang có. Khối server đầu tiên khớp với hostname này sẽ được chọn.
- Nếu không có khối server nào khớp với hostname trên, Nginx chọn khối server đầu tiên khớp với các thông số của chỉ thị listen (ví dụ như listen *:80 sẽ bắt tất cả các yêu cầu nhận được trên port 80), ưu tiên khối đầu tiên có tùy chọn mặc định được cho phép trên chỉ thị listen.
- Cú pháp: ```server_name hostname1 [hostname2…];```
- Ví dụ :
```console
        server_name www.acb.com;
        server_name www.abc.com abc.com;
        server_name *.website.com; # nhận tất cả các domain có đuôi là .website.com
        server_name .website.com; # Kết hợp cả *.website.com và website.com
        server_name *.website.*;
        server_name ~^\.example\.com$;
```
- Lưu ý rằng chúng ta có thể sử dụng chuỗi rỗng như 1 giá trị của chỉ thị để bắt tất cả các yêu cầu không có giá trị Host trong phần header, nhưng chỉ sau ít nhất 1 tên thông thường (hoặc “_”)
```console
                server_name abc.com “”;
                server_name _ “”;
```

**satisfy** :

- Sử dụng trong khối : ```location```
- Chỉ thị satisfy định nghĩa việc khách hàng được yêu cầu đáp ứng tất cả các điều kiện truy cập (satisfy all) hoặc ít nhất 1 điều kiện (satisfy any) để là khách hàng hợp lệ:
```connsole
        location /admin/ {
          allow 192.168.1.0/24;
          deny all;
          auth_basic “Authentication required”;
          auth_basic_user_file conf/htpasswd;
        }
```
- Trong ví dụ trên, có 2 điều kiện cho khách hàng để có thể truy cập:

- Qua chỉ thị allow và deny, chúng ta chỉ cho phép các khách hàng có địa chỉ IP cục bộ, tất cả các khách hàng khác sẽ bị từ chối.
- Qua chỉ thị auth_basic và auth_basic_user_file, chúng ta chỉ cho phép các khách hàng cung cấp 1 username và password hợp lệ
Với satisfy all, khách hàng phải thoả mãn cả 2 điều kiện trên.

- Với satisfy any, khách hàng chỉ cần thoả mãn 1 trong 2 điều kiện trên

- Cú pháp: ```satisfy any | all;```

- Giá trị mặc định: all
