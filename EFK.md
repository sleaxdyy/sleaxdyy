### **Cơ chế**
![image](https://cdn.discordapp.com/attachments/473366456092852246/930383312499916810/efk.png)
1.	Đầu tiên, log sẽ được đưa đến Fluentd. (Ví dụ như log access server nginx/apache, log do develop setting trong source php/java vv. miễn là có ghi ra file log).
2.	Fluentd sẽ đọc những log này, thêm những thông tin như thời gian, IP, parse dữ liệu từ log (server nào, độ nghiêm trọng, nội dung log) ra, sau đó ghi xuống database là Elasticsearch.
3.	Khi muốn xem log, người dùng vào URL của Kibana. Kibana sẽ đọc thông tin log trong Elasticsearch, hiển thị lên giao diện cho người dùng query và xử lý.
4.	Log có nhiều loại (tag), do developer định nghĩa chẳng hạn access_log error_log, peformance_log, api_log. Khi Fluentd đọc log và phân loại từng loại log rồi gửi đến Elasticsearch, ở giao diện Kibana chúng ta chỉ cần add một số filter như access_log chẳng hạn và query.
## **CÀI ĐẶT EFK TRÊN CENTOS 7**
***Tài liệu tham khảo:***
- [https://blog.vietnamlab.vn/quan-ly-log-voi-logstash-elasticsearch-kibana/](https://blog.vietnamlab.vn/quan-ly-log-voi-logstash-elasticsearch-kibana/)
- [https://kifarunix.com/setup-kibana-elasticsearch-and-fluentd-on-centos-8/#configure-fluentd-aggregator-centos-8](https://kifarunix.com/setup-kibana-elasticsearch-and-fluentd-on-centos-8/#configure-fluentd-aggregator-centos-8)
- [https://docs.fluentd.org/input/tail](https://docs.fluentd.org/input/tail)
### **TẠO ÍT NHẤT 2 MÁY CENTOS:**
***Centos cài EFK nhận logs : 192.168.5.82***

***Centos cài fluentd td-agent gửi logs: 192.168.5.88***
### **CÀI ĐẶT ELASTICSEARCH:**
1. Tạo repo Elasticsearch Stack trên Centos 7:
cat > /etc/yum.repos.d/elasticstack.repo << EOL 
[elasticstack] 
name=Elastic repository for 7.x packages 
baseurl=https://artifacts.elastic.co/packages/7.x/yum 
gpgcheck=1 
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch 
enabled=1 
autorefresh=1 
type=rpm-md 
EOL
```console 
cat > /etc/yum.repos.d/elasticstack.repo << EOL 
[elasticstack] 
name=Elastic repository for 7.x packages 
baseurl=https://artifacts.elastic.co/packages/7.x/yum 
gpgcheck=1 
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch 
enabled=1 
autorefresh=1 
type=rpm-md 
EOL
```
2. Install Elasticsearch trên Centos7 từ repo
```console
yum install elasticsearch
```
3. Cấu hình Elasticsearch
```console 
vi /etc/elasticsearch/elasticsearch.yml
```
- Bỏ ghi chú và thay bằng địa chỉ IP của máy Centos7 đang sử dụng
```
#network.host: 192.168.0.1
#network.host: 192.168.5.82
```
- Cấu hình ES là 1 single node
``` 
discovery.type: single-node 
```
4. Khởi động và bật cho phép ES chạy khi khởi động hệ thống
```console
systemctl daemon-reload
systemctl enable --now elasticsearch
systemctl start elasticsearch
```
5. Xác mình ES đang chạy như mong đợi
```console
curl -XGET 192.168.5.82:9200
```
*Để sử dụng lệnh curl*

```console 
yum install curl
``` 
### **CÀI ĐẶT KIBANA:**
***Install Kibana***
```console
yum install kibana
```
6. Cấu hình Kibana
```console
vim /etc/kibana/kibana.yml
```
7. Bỏ ghi chú 3 dòng dưới và thay địa chỉ IP giống với ES
```
#server.port: 5601
#server.host: "localhost"
#elasticsearch.hosts: ["http://localhost:9200"]
```
```
server.port: 5601
server.host: "192.168.5.82"
elasticsearch.hosts: ["http://192.168.5.82:9200"]
```
8. Chạy và cho phép bật Kibana khi hệ thống khởi động
```console
systemctl enable --now kibana
systemctl start kibana
```
9. Mở Kibana Port trên FirewallD
```console
firewall-cmd --add-port=5601/tcp --permanent
firewall-cmd –reload
```
10. Truy cập giao diện Kibana bằng trình duyệt : 
http://192.168.5.82:5601

### **CÀI ĐẶT VÀ CẤU HÌNH FLUENTD TRÊN CENTOS7**
11. Cài đặt và cấu hình fluentd để thu thập logs vào ES (sử dụng td-agent)
```console
curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent4.sh | sh
```

12.	Chạy và bật chạy Fluentd td-agent khi khỏi động
```console
systemctl enable --now td-agent
systemctl start td-agent
```
13.	Cài đặt kết nối Fluentd Elasticsearch
```console
td-agent-gem install fluent-plugin-elasticsearch
```
14.	Cài đặt secure_forward kết nối đầu ra Fluentd gửi dữ liệu an toàn
td-agent-gem install fluent-plugin-secure-forward

### **CẤU HÌNH FLUENTD THU THẬP LOGS(192.168.5.82)**
```console
vi /etc/td-agent/td-agent.conf
```
```console
<match *.**> 
@type elasticsearch 
host 192.168.5.82 
port 9200 
logstash_format true 
logstash_prefix fluentd 
enable_ilm true 
index_date_pattern "now/m{yyyy.mm}" 
flush_interval 10s 
</match> 
<source> 
@type forward 
port 24224 
bind 192.168.5.82 
</source>
```
15.	Mở cổng 24224 trên firewall
```console
firewall-cmd --add-port=24224/{tcp,udp} --permanent
firewall-cmd --reload
```
16.	Khởi dộng lại td-agent:
```console
systemctl restart td-agent
```

### **CÀI ĐẶT FLUENTD ĐỂ GỬI CÁC LOGS(192.168.5.88)**
```console
curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent4.sh | sh
```
17.	Cấu hình:
```console
vi /etc/td-agent/td-agent.conf
```
```console
<match *.**> 
@type forward 
send_timeout 60s 
recover_wait 10s 
hard_timeout 60s 
<server> 
name log_mgr 
host 192.168.5.88 
port 24224 
weight 60 
</server> 
</match> 
<source> 
@type tail 
path /var/log/secure 
pos_file /var/log/td-agent/secure.pos 
tag ssh.auth 
<parse> 
@type syslog 
</parse> 
</source>
```
18. Cấp quyền truy cập đọc Fluentd vào logs xác thực hoặc bất kỳ log nào đang được thu thập.
```console
ls -alh /var/log/secure
chmod og+r /var/log/secure
```
19.	Khởi động và cho phép Fluentd Forwarder chạy khi khởi động hệ thống
```console
systemctl enable --now td-agent
systemctl start td-agent
```
20.	Đọc logs
```console
tail -f /var/log/td-agent/td-agent.log
```
21.	Kiểm tra xem có dữ liệu nào đang nhận trên cổng 24224 không
```console
tcpdump -i enp0s8 -nn dst port 24224
```
22.	Sau đó, hãy kiểm tra xem chỉ mục Elasticsearch của bạn đã được tạo chưa
(logstash_prefix [Chỉ mục])
```console
curl -XGET http://192.168.5.88:9200/_cat/indices?v
```
## **TẠO CHỈ MỤC FLUENT KIBANA**
23.	Mở trình duyệt và truy cập vào: http://192.168.5.82:5601
24.	Search: Kibana -> IndexPattern -> Create IndexPattern-> Nhập tên chỉ mục
 ![imange](https://cdn.discordapp.com/attachments/473366456092852246/930383312864813066/efk.png)
25.	Chọn @timestamp -> Creata IndexPattern

***XEM DỮ LIỆU TRÊN KIBANA***

26.	Chọn Discover trong tab mở rộng ở bên trái
 ![image](https://cdn.discordapp.com/attachments/473366456092852246/930383313309425694/efk1.png)


