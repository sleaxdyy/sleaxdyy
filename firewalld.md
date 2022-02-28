## **FIREWALLD**
## **Khái niệm**

- FirewallD là giải pháp tường lửa mạnh mẽ, toàn diện được cài đặt mặc định trên CentOS/RHEL 7, nhằm thay thế Iptables với những khác biệt cơ bản:

- FirewallD sử dụng “zones” và “services” thay vì “chain” và “rules” trong Iptables.

- FirewallD quản lý các quy tắc được thiết lập tự động, có tác dụng ngay lập tức mà không làm mất đi các kết nối và session hiện có.
## **Các khái niệm cơ bản**
### **1. Zone**
- Trong FirewallD, zone là một nhóm các quy tắc nhằm chỉ ra những luồng dữ liệu được cho phép, dựa trên mức độ tin tưởng của điểm xuất phát luồng dữ liệu đó trong hệ thống mạng. Để sử dụng, bạn có thể lựa chọn zone mặc định, thiết lập các quy tắc trong zone hay chỉ định network interface để quy định hành vi được cho phép.

- ```drop :``` ít tin cậy nhất – toàn bộ các kết nối đến sẽ bị từ chối mà không phản hồi , chỉ cho phép duy nhất kết nối đi ra .

- ```block :``` tương tự như drop nhưng các kết nối đến bị từ chối và phản hồi bằng tin nhắn từ icmp-host-prohibited ( hoặc icmp6-adm-prohibited ).

- ```public :``` đại diện cho mạng công cộng , không đáng tin cậy . Các máy tính/services khác không được tin tưởng trong hệ thống nhưng vẫn cho phép các kết nối đến trên cơ sở chọn từng trường hợp cụ thể .

- ```external :``` hệ thống mạng bên ngoài trong trường hợp bạn sử dụng tường lửa làm gateway, được cấu hình giả lập NAT để giữ bảo mật mạng nội bộ mà vẫn có thể truy cập.

- ```internal :``` đối lập với external zone , sử dụng cho phần nội bộ của gateway. Các máy tính/services thuộc zone này thì khá đáng tin cậy .

- ```dmz :``` sử dụng cho các máy tính/service trong khu vực DMZ ( Demilitarized ) – cách ly không cho phép truy cập vào phần còn lại của hệ thống mạng , chỉ cho phép một số kết nối đến nhất định .

- ```work :``` sử dụng trong công việc, tin tưởng hầu hết các máy tính và một vài services được cho phép hoạt động.

- ```home :``` môi trường gia đình – tin tưởng hầu hết các máy tính khác và thêm một vài services được cho phép hoạt động .

- ```trusted :``` đáng tin cậy nhất – tin tưởng toàn bộ thiết bị trong hệ thống.

### **2. Hiệu lực các quy tắc**
- ```Runtime ( mặc định ) :``` có tác dụng ngay lập tức , mất hiệu lực khi reboot hệ thống .

- ```Permanent :``` không áp dụng cho hệ thống đang chạy, cần reload mới có hiệu lực , tác dụng vĩnh viễn cả khi reboot hệ thống .
## **Cài đặt cấu hình firewalld**
### **1. Thiết lập các zone**
- Liệt kê tất cả các zone trong hệ thống :

```console 
firewall-cmd --get-zones
```

- Kiểm tra zone mặc định :

```console
firewall-cmd --get-default-zone
```

- Kiểm tra zone active ( được sử dụng bởi card mạng )

- Vì FirewallD chưa được thiết lập bất kỳ quy tắc nào nên zone mặc định cũng đồng thời là zone duy nhất được kích hoạt , điều khiển mọi luồng dữ liệu .

```console
firewall-cmd --get-active-zones
```

- Thay đổi default zone ( vd thành home ) :

```console
firewall-cmd --set-default-zone=home
```

### **2. Thiết lập các rule**

- Liệt kê toàn bộ các rule của các zones :

```console
firewall-cmd --list-all-zones
```

- Liệt kê toàn bộ các rule trong default zone và active zone :

```console
firewall-cmd --list-all
```

- Liệt kê toàn bộ các quy tắc trong một zone cụ thể , ví dụ home :

```console
firewall-cmd --zone=home --list-all
```

- Liệt kê danh sách services/port được cho phép trong zone cụ thể :
```console
firewall-cmd --zone=public --list-services
```

```console
firewall-cmd --zone=public --list-ports
```
### **3. Thiết lập service**
Xác định các services trên hệ thống :

```console
firewall-cmd --get-services
```
- Thông tin về services được lưu trữ tại /usr/lib/firewalld/services

- Thiết lập cho phép services trên firewalld , sử dụng --add-service :

```console
firewall-cmd --zone=public --add-service=http
```
```console
firewall-cmd --zone=public --add-service=http --permanent
```

- Thực hiện kiểm tra xem service đã được cho phép chưa :

```console
firewall-cmd --zone=public --list-services
```

- Vô hiệu hóa services trên firewalld , sử dụng --remove-service :

```console
firewall-cmd --zone=public --remove-service=http
```

```console
firewall-cmd --zone=public --remove-service=http --permanent
```
### **4. Thiết lập cho port**

- Mở port với tham số --add-port :

```console
firewall-cmd --zone=public --add-port=9999/tcp
```

```console
firewall-cmd --zone=public --add-port=9999/tcp --permanent
```

- Mở 1 dải port :

```console
firewall-cmd --zone=public --add-port=4990-5000/tcp
```

```console
firewall-cmd --zone=public --add-port=4990-5000/tcp --permanent
```

- Kiểm tra lại các port đã mở :

```console
firewall-cmd --zone=public --list-ports
```

- Đóng port với tham số --remove-port :
```console
firewall-cmd --zone=public --remove-port=9999/tcp
```
```console
firewall-cmd --zone=public --remove-port=9999/tcp --permanent
```
### **5. Cấu hình nâng cao**
### Tạo Zone riêng

- Mặc dù, các zone có sẵn là quá đủ với nhu cầu sử dụng , vẫn có thể tạo lập zone của riêng mình để mô tả rõ ràng hơn về các chức năng của chúng .

- VD : Có thể tạo riêng một zone cho webserver publicweb hay một zone cấu hình riêng cho DNS trong mạng nội bộ private DNS . Cần thiết lập permanent khi thêm một zone :

```console
firewall-cmd --permanent --new-zone=publicweb
```
```console
firewall-cmd --permanent --new-zone=privateDNS
```
```console
firewall-cmd --reload
```
- Kiểm tra lại :
```console
firewall-cmd --get-zones
```

- Khi đã có zone thiết lập riêng , có thể cấu hình như các zone thông thường : thiết lập mặc định , thêm quy tắc :

```console
firewall-cmd --zone=publicweb --add-service=ssh --permanent

firewall-cmd --zone=publicweb --add-service=http --permanent

firewall-cmd --zone=publicweb --add-service=https --permanent
```
### Định nghĩa services riêng trên FirewallD
- Khi có một service mới thêm vào hệ thống , có 2 phương án :

    - Mở port của service đó trên FirewallD

    - Tự định nghĩa service đó trên FirewallD

( VD : có thể tự định nghĩa port cho Admin Port là 9999 )

- Tạo file định nghĩa riêng từ file chuẩn ban đầu :

```console
cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/admin.xml
```
- Chỉnh sửa để định nghĩa servies trên FirewallD :

```console
vi /etc/firewalld/services/admin.xml
```

- Lưu lại và khởi động lại FirewallD :

```console
firewall-cmd --reload
```
- Kiểm tra lại danh sách các services :

```console
firewall-cmd --get-services
```
