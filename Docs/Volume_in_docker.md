
Dữ liệu khi chạy trong container thì sẽ bị mất nếu tắt contaier đó  
Nếu một tiến trình khác cần đến dữ liệu thì rất khó để lấy dữ liệu từ container ra bên ngoài
Docker cung cấp 3 cách khác nhau để có thể chia sẻ dữ liệu (mount data) từ Docker host tới container đó là:

volumes
bind mounts
tmpfs mounts

=> volumes thường luôn là cách được lựa chọn sử dụng nhiều nhất đối với mọi trường hợp
Sự khác biệt giữa volumes, bind mounts và tmpfs mounts chỉ đơn giản là khác nhau về vị trí lưu trữ dữ liệu trên Docker host.

 Volumes được lưu trữ như một phần của filesystem trên host cài docker và được quản lý bởi Docker. Nó nằm trong thư mục xuất hiện trong /var/lib/docker/volumes trên Linux

bind mounts cho phép lưu trữ bất cứ đâu trong host system.

tmpfs mounts cho phép lưu trữ tạm thời dữ liệu vào bộ nhớ của Docker host, không bao giờ ghi vào filesystem của Docker host.



