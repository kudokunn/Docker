- Xem các trợ giúp về các lệnh trong docker: docker --help
- Xem các option lệnh trong docker: docker <command> --help
Ex: docker run --help
- Xem cac image hiện có: docker images
- Kéo image: docker pull <tên image:version>
Ex: docker pull ubuntu:14.04 or docker pull ubuntu=ubuntu:latest
- Tìm kiếm image: docker seacher <tên images> 
Ex: docker search nginx
- Xóa image: docker rmi <tên image>
- Chạy một image để tạo container: docker run [option] <tên image>
[option]: -d: chạy background, 
          -it bash: chạy vào hẳn container để thao tác trong container, 
          -e: thêm các biến môi trường
          -v: đồng bộ data từ host đến container: docker run -v ./:/minh/hah -it <image> bash
          ... more: docker run --help
- Bật sẵn sàng một container: docker create [option] <tên image>

#### Cấu trúc chuẩn cho khi chạy images tạo container: docker run --name <name_container> 

#### [-p xx:yy | change port yy container ra port host sẽ đọc là xx] 

#### [-v /xx:/yy | xx->yy | mount /xx vào /yy trong container ]

#### -d nginx
