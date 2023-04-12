# Hướng dẫn cài đặt Netdata trên CentOS 6 offline

Để cài đặt Netdata trên CentOS 6 offline, bạn có thể làm theo các bước sau:

## Bước 1: Tải xuống Netdata

Truy cập vào trang web chính thức của Netdata (https://my-netdata.io/), sau đó tải xuống bản phù hợp với hệ điều hành CentOS 6 của bạn. Bạn có thể tải về bằng lệnh wget trên máy tính khác và di chuyển file tải về sang máy đích qua USB hoặc một phương tiện lưu trữ khác.

## Bước 2: Cài đặt các gói yêu cầu cho Netdata

Trong khi cài đặt Netdata, bạn cần phải cài đặt một số gói phụ thuộc. Thực hiện lệnh sau để cài đặt các gói này:

```
sudo yum install zlib-devel uuid-devel libmnl gcc make git autoconf autogen automake pkgconfig curl which
```

## Bước 3: Giải nén và cài đặt Netdata

Sau khi tải xuống Netdata, bạn giải nén nó bằng lệnh sau:

```
tar -xvf netdata-x.x.x.tar.gz
```

Chú ý: Thay thế `x.x.x` bằng phiên bản của Netdata bạn đã tải xuống.

Sau khi giải nén, bạn tiến hành cài đặt Netdata:

```
cd netdata-x.x.x/
sudo ./netdata-installer.sh --dont-wait
```

Lưu ý rằng quá trình cài đặt sẽ mất một chút thời gian để hoàn tất và sau đó bạn đã cài đặt xong Netdata.

## Bước 4: Khởi động dịch vụ

Sau khi cài đặt, hãy khởi động danh sách dịch vụ bằng lệnh sau:

```
sudo service netdata start
```

Nếu muốn dịch vụ được khởi động cùng với hệ thống, bạn có thể sử dụng lệnh:

```
sudo chkconfig netdata on
```

## Bước 5: Truy cập vào giao diện Netdata

Mở trình duyệt web và truy cập vào địa chỉ `http://localhost:19999` để xem giao diện Netdata. Bạn có thể thay đổi cổng này trong tệp cấu hình nếu cần thiết.

Hy vọng hướng dẫn này sẽ giúp bạn cài đặt thành công Netdata trên CentOS 6 offline!


