### Nội dung cơ bản

* Dữ liệu khi chạy trong container thì sẽ bị mất nếu tắt contaier đó  

* Nếu một tiến trình khác cần đến dữ liệu thì rất khó để lấy dữ liệu từ container ra bên ngoài

* Docker cung cấp 3 cách khác nhau để có thể chia sẻ dữ liệu (mount data) từ Docker host tới container đó là:

       volumes

       bind mounts

       tmpfs mounts

=> Volumes thường luôn là cách được lựa chọn sử dụng nhiều nhất đối với mọi trường hợp

#### 1. Sự khác biệt giữa volumes, bind mounts và tmpfs mounts chỉ đơn giản là khác nhau về vị trí lưu trữ dữ liệu trên Docker host.

* Volumes được lưu trữ như một phần của filesystem trên host cài docker và được quản lý bởi Docker. Nó nằm trong thư mục xuất hiện trong /var/lib/docker/volumes trên Linux

* Bind mounts cho phép lưu trữ bất cứ đâu trong host system.

* Tmpfs mounts cho phép lưu trữ tạm thời dữ liệu vào bộ nhớ của Docker host, không bao giờ ghi vào filesystem của Docker host.

#### 2. Đặc điểm của Volumes:

* Chia sẻ dữ liệu với nhiều containers đang chạy. Dữ liệu yêu cầu phải tồn tại kể cả khi dừng hoặc loại bỏ containers.

* Khi muốn lưu trữ dữ liệu containers trên remote hosts, cloud thay vì Docker host.

* Khi có nhu cầu sao lưu, backup hoặc migrate dữ liệu tới Docker host khác thì volumes là một sự lựa tốt. Ta cần phải dừng containers sử dụng volumes sau đó thực hiện backup tại đường dẫn /var/lib/docker/volumes/<volume-name>

#### 3. Đặc điểm của bind mounts

* Sử dụng bind mounts thì một file hoặc một thư mục trên Docker host sẽ được mount tới containers với đường dẫn đầy đủ.

* Chia sẻ các file cấu hình từ Docker host tới containers.

* Kiểm soát được các thay đổi của containers đối với filesystem trên Docker host. Do khi sử dụng bind mounts, containers có thể trực tiếp thay đổi filesystem trên Docker host.

#### 4. Đặc điểm  tmpfs mount

* tmpfs mounts được sử dụng trong các trường hợp ta không muốn dữ liệu tồn tại trên Docker host hay containers vì lý do bảo mật hoặc đảm bảo hiệu suất của containers khi ghi một lượng lớn dữ liệu một cách không liên tục.


#### 5.Cú pháp sử dụng:

Note: bind mounts và volumes đều có thể được mount vào container khi sử dụng flag -v hoặc --volume nhưng cú pháp sử dụng có một chút khác nhau. Đối với tmpfs mounts có thể sử dụng flag --tmpfs. Tuy nhiên, từ bản Docker 17.06 trở đi, chúng ta được khuyến cáo dùng flag --mount cho cả 3 cách, để cú pháp câu lệnh minh bạch hơn.

* Sự khác nhau giữa --volume, -v và --mount là về cách khai báo các giá trị:

       --volume, -v các giá trị cách nhau bởi :. theo dạng source:target. Ví dụ: -v myvol2:/app

       --mount khai báo giá trị theo dạng key=values. Ví dụ: --mount type=xxx source=myvol2,target=/app. Trong đó source có thể thay thế bằng src, target có thể thay thế bằng destination hoặc dst.


* Chú ý: Khi sử dụng volumes cho services thì chỉ --mount mới có thể sử dụng (Cái này hơi khó hiểu)

##### 6. Hướng dẫn sử dụng Volume:

6.1 Lợi ích của nó: 

* Volumes dễ dàng backups, migrate hơn so với bind mounts.

* Có thể quản lý volumes sử dụng Docker CLI hoặc Docker API.

* Volumes làm việc trên cả Linux containers và Winodws containers.

* Volumes an toàn hơn khi sử dụng để chi sẻ giữa nhiều containers.

6.2 Sử dụng:

* Để tạo một volume, ta sử dụng câu lệnh:

     docker volume create my-vol
     
* Khi khởi chạy containers với volume chưa có (tồn tại) thì Docker sẽ tự động tạo ra volume với tên được khai báo hoặc với một tên ngẫu nhiên và đảm bảo tên ngẫu nhiên này là duy nhất. Ví dụ:

       docker run -d \
      -it \
      --name devtest \
      --mount type=volume,source=myvol2,target=/app \
      nginx:latest
    
câu lệnh trên sẽ thực hiện mount volume myvol2 tới thư mục /app trong container devtest.

* Nếu muốn mount volume với chế độ readonly, ta có thể thêm vào trong --mount. Với ví dụ trên có thể là làm như sau:

      --mount source=myvol2,target=/app,readonly






