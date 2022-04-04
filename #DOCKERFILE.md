#DOCKERFILE
RUN : Để thực thi một câu lệnh nào đó trong quá trình build images.

CMD : Để thực thi một câu lệnh trong quá trình bật container.

Mỗi Dockerfile chỉ có một câu lệnh CMD, nếu như có nhiều hơn một câu lệnh CMD thì chỉ có câu lệnh CMD cuối cùng được sử dụng.

Một câu hỏi đặt ra là nếu tôi muốn khởi động nhiều ứng dụng khi start container thì sao, lúc đó hay nghĩ tới ENTRYPOINT

ENTRYPOINT : Để thực thi một số câu lệnh trong quá trình start container, những câu lệnh này sẽ được viết trong file .sh.

EXPOSE : Container sẽ lắng nghe trên các cổng mạng được chỉ định khi chạy

ADD : Copy file, thư mục, remote file thêm chúng vào filesystem của image.

COPY : Copy file, thư mục từ host machine vào image. Có thể sử dụng url cho tập tin cần copy.

WORKDIR : Định nghĩa directory cho CMD

VOLUME : Mount thư mục từ máy host vào container.

#DOCKER COMPOSE
image : Chỉ định image để khởi động container, ở đây ta dùng image có sẵn như đã nói ở mục 4.

☑ container_name : Chỉ định tên container tùy chỉnh, thay vì tên mặc định.

☑ restart : Giá trị mặc định là no, còn nếu bạn đặt là always thì container sẽ khởi động lại nếu mã thoát cho biết lỗi không thành công.

☑ environment : Thêm các biến môi trường

☑ volumes : Chia sẻ dữ liệu giữa container (máy ảo) và host (máy thật) hoặc giữa các container với nhau.