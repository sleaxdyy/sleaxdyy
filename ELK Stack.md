## Tìm hiểu về ELK Stack
## ELK Stack là gì?

- ELK Stack là tập hợp 3 phần mềm đi chung với nhau, phục vụ cho công việc logging. Ba phần mềm này lần lượt là:

    * Elasticsearch: Cơ sở dữ liệu để lưu trữ, tìm kiếm và query log
    * Logstash: Tiếp nhận log từ nhiều nguồn, sau đó xử lý log và ghi dữ liệu và Elasticsearch
    * Kibana: Giao diện để quản lý, thống kê log. Đọc thông tin từ Elasticsearch
### Cơ chế hoạt động của ELK Stack cũng khá đơn giản:
![image](https://toidicodedao.files.wordpress.com/2018/02/12.png)

Cơ chế hoạt động của ELK Stack
1. Đầu tiên, log sẽ được đưa đến Logstash. (Thông qua nhiều con đường, ví dụ như server gửi UDP request chứa log tới URL của Logstash, hoặc Beat đọc file log và gửi lên Logstash)
2. Logstash sẽ đọc những log này, thêm những thông tin như thời gian, IP, parse dữ liệu từ log (server nào, độ nghiêm trọng, nội dung log) ra, sau đó ghi xuống database là Elasticsearch.
3. Khi muốn xem log, người dùng vào URL của Kibana. Kibana sẽ đọc thông tin log trong Elasticsearch, hiển thị lên giao diện cho người dùng query và xử lý.
### **Cài đặt ELK Stack**
### ***Tạo máy ảo 1 để cài gói ELK***
***Cập nhật server và cài một số gói cần thiết:***
```console
yum update -y && yum install -y vim wget curl
```
1. Disable Firewall
```console
systemctl status firewalld
systemctl disable firewalld
systemctl stop firewalld
```
2. Cài đặt Java
```console
yum -y install java-openjdk-devel java-openjdk
```
***Cài đặt Elasticsearch***
1. Cài đặt public key
```console
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
2. Thêm repo Elasticsearch version 8 để cài đặt
```console
vim /etc/yum.repos.d/elasticsearch.repo
```
- Thêm đoạn này vào:
```console
[elasticsearch-8.x]
name=Elasticsearch repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
3. Thiết lập dung lượng RAM để chạy Elasticsearch 

```console
vim /etc/elasticsearch/jvm.options
```
- Cấu hình lại 2 tham số sau:
```console
-Xms256M
-Xmx512M
```
4. Cấu hình elasticsearch
```console
vi /etc/elasticsearch/elasticsearch.yml
```
- Bỏ ghi chú network.host và http.port và thay bằng địa chỉ IP của máy:
```console
network.host: <IP máy>
http.port: 9200
```
- Đặt elasticsearch là một nút duy nhất
```console
discovery.type: single-node
```
- Đổi các cài đặt security về false
```console
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
xpack.security.http.ssl:
  enabled: false
xpack.security.transport.ssl:
  enabled: false
```
- Thêm ghi chú vào cluster.initial_master_nodes:
```console
#cluster.initial_master_nodes: ["quang"]
```
5. Lưu config và khởi động elasticsearch
```console
systemctl enable elasticsearch
systemctl start elasticsearch
```
***Cài đặt Kibana***
1. Cài đặt Kibana
```console
yum -y install kibana
```
2. Cấu hình Kibana
```console
vi /etc/kibana/kibana.yml
```
- Chỉnh sửa các thông số sau
```
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://<IP máy>:9200"]
```
6. Lưu và khỏi động Kibana
```console
systemctl enable kibana
systemctl start kibana
```
***Cài đặt logstash***
1. Cài đặt Logstash
```console
yum -y install logstash
```
2. Cấi hình logstash
```console
cp /etc/logstash/logstash-sample.conf /etc/logstash/conf.d/logstash.conf
vim /etc/logstash/conf.d/logstash.conf
```
- Cấu hình file như sau
```console
input {
  beats {
    port => 5044
  }
}
output {
 elasticsearch {
  hosts => ["http://192.168.5.31:9200"]
  index => "%{[fields][type]}-%{+yyyy.MM.dd}"
  }
}
```
3. Lưu lại và khởi động logstash
```console
systemctl enable logstash
systemctl start logstash
systemctl status logstash
```
### ***Tạo máy ảo 2 để cài Filebeat và NGINX để test***
***Cài đặt NGINX***
```console
sudo yum install -y epel-release
sudo yum install -y nginx
```
- Chạy NGINX
```console
sudo systemctl start nginx
```
***Cài đặt Filebeat***
1. Thêm Filebeat public signing key trước bằng cách chạy câu lệnh sau:
```
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
2. Các bạn cần tạo mới một tập tin để thêm Filebeat repository:
```console
[filebeat-8.x]
name=Filebeat repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
3. Cài đặt Filebeat
```console
yum install -y filebeat
```
4. Cấu hình Filebeat
```console
filebeat.inputs:
- input_type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  fields:
    type: "nginx.access"
- input_type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  fields:
    type: "nginx.error"

output.logstash:
  hosts: ["192.168.5.31:5044"]
```
5. Lưu và chạy Filebeat
```console
systemctl start Filebeat
```
## Mở web Kibana để kiểm tra log
1. Truy cập vào web tạo ra bởi NGINX để sinh log http://<IP máy 2> 
2. Truy cập vào Kibana theo địa chỉ http://<IP máy 1>:5601
3. Trên thanh tìm kiếm Search 'Data View' để 'create data view'
4. Chọn Create data view => Nhập tên Index => Creat data view
![image](https://cdn.discordapp.com/attachments/473366456092852246/978589106105876510/unknown.png)
![image](https://cdn.discordapp.com/attachments/473366456092852246/978589137609306162/unknown.png)
5. Thanh mở rộng bên phải chọn Discover để xem log được gửi đến
![image](https://cdn.discordapp.com/attachments/473366456092852246/978589164821971015/unknown.png)
![image](https://cdn.discordapp.com/attachments/473366456092852246/978589202621014036/unknown.png)