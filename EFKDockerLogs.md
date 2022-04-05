## **Cài đặt EFK với Docker**
## - Tạo Image Fluentd
1. Tạo thư mục mới để chứa tài nguyên Fluentd Docker và di chuyển vào thư mục đó
```console
mkdir ~/fluentd-docker && cd ~/fluentd-docker
```
2. Tạo Dockerfile
```console
FROM ruby:3.1.1
RUN apt-get update
RUN gem install fluentd -v "~>0.12.3"
RUN mkdir /etc/fluent
RUN apt-get install -y libcurl4-gnutls-dev make
RUN /usr/local/bin/gem install fluent-plugin-elasticsearch
ADD fluent.conf /etc/fluent/
ENTRYPOINT ["/usr/local/bundle/bin/fluentd", "-c", "/etc/fluent/fluent.conf"]
```
3. Tạo fluent.conf
```console
<source>
  type tail
  read_from_head true
  path /var/lib/docker/containers/*/*-json.log
  pos_file /var/log/fluentd-docker.pos
  time_format %Y-%m-%dT%H:%M:%S
  timezone +07:00
  tag docker.*
  format json
</source>
# Using filter to add container IDs to each event
<filter docker.var.lib.docker.containers.*.*.log>
  type record_transformer
  <record>
    container_id ${tag_parts[5]}
  </record>
</filter>

<match docker.var.lib.docker.containers.*.*.log>
  type elasticsearch
  logstash_format true
  host "#{ENV['ES_PORT_9200_TCP_ADDR']}" # dynamically configured to use Docker's link feature
  port 9200
  flush_interval 5s
</match>
```
4. Build Dockerfile
```console
docker build -t fluentd-es .
```
5. Kiểm tra Image
```console
docker image ls
```
## - Cài đặt Elasticsearch với Docker
1. Để sử dụng Image phải tăng giá trị `max_map_count` trên máy chủ chứa Docker
```console
sudo sysctl -w vm.max_map_count=262144
```
2. Thực hiện lệnh tải xuống Image và chạy Container
```console
docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "xpack.security.enabled=false" elasticsearch:8.1.1
```
## - Cài đặt Kibana với Docker
- Thực hiện lệnh tải xuống Image và chạy Container
```console
docker run -d -p 5601:5601 --name kibana kibana:8.1.1
```
## - Kiểm tra 
1. Truy cập web theo đường dẫn `http://<your-ip>:5601`
2. Chọn Config Manually
3. Dán địa chỉ kết nối với Elasticsearch `http://<your-ip>:9200`
4. Truy cập vào container Kibana để lấy `kibana-verification-code`
```console
docker exec -it kibana bash
bin/kibana-verification-code
```
5. Tìm từ khóa `Data Views` trên thanh tìm kiếm
6. Create data view và tìm chỉ mục theo tên đã đặt
7. Chọn tab Discover để xem logs
## - Thay đổi múi giờ container
1. Truy cập vào container cần thay đổi
```console
docker exec -u 0 -it nginx bash
```
2. Xóa múi giờ đã được đặt từ trước
```console
rm -rf /etc/localtime
```
3. Đặt múi giờ cho container
```console
ln -s /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
```
4. Kiểm tra
```console
date
```
