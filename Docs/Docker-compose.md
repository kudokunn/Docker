### Vấn đề: EX: Để chạy bộ  LAMP ta cần: Linux, apache, mysql, php. 

* Với Mysql

  1: Lấy image về: docker pull mysql
  2: Chạy contaier: docker run -e MYSQL_ROOT_PASSWORD=123 MYSQL_DATABASE=abc ... -p 8000:3306 -d mysql
  
* php1:

  1: docker pull php
  2: docker run php [tùy chọn] -d php
  
* Apache:

  1: docker pull httpd
  2: docker run php [tùy chọn] -d httpd
  
Vấn đề:

=> Để chạy LAMP phải chạy 3 contaier bằng tay chưa kể các tham số tùy chọn đi theo thêm sự phức tạp
=> Nếu cần mở rộng thêm sự tùy chọn nào đó thì sẽ rất mệt để gõ lệnh viết lại các tùy chọn cũ và thêm tùy chọn mới

=>=>=> Do đó: Docker-compose giải quyết cái này bằng cách viết một file kịck bản dạng YML sẽ chạy các image thành các container cùng với các tùy chọn tham số cấu hình đi kèm và chạy nó một cách tự động. Hiều là nó làm thay việc chạy nhiều tham số của docker run.

     Giống ansible kể cả dạng cấu trúc chạy theo thư mục, mỗi thư mục là một service chạy

### Ví dụ1 : Tạo file: vim docker-compose.yml. Tham khảo thêm cách viết: https://docs.docker.com/compose/compose-file/compose-file-v2/#healthcheck

    version: '2'

    services:
        mysql:
            image: mysql
            restart: always
            environment:
               - variables.env
            # volumes:
            #     - ./data/mysql:/docker-entrypoint-initdb.d
            
        nginx:
            depends_on:
                - phpfpm
            image: nginx
            ports:
                - "80:80"
            restart: always
            working_dir: /var/www
            volumes:
                - ./wordpress:/var/www/
                - ./conf/nginx/gsviec.conf:/etc/nginx/conf.d/default.conf
            links:
                - phpfpm
     
         phpfpm:
             image: phalconphp/php-apache:ubuntu-16.04
             restart: always
             working_dir: /var/www
             ports:
                 - "8080:80"
             volumes:
                 - ./wordpress:/var/www
             depends_on:
                 - mysql

### Ví dụ 2: 

         version: '2'

        services:
           db:
             image: mysql:5.7
             volumes:
               - ./data:/var/lib/mysql
             restart: always
             environment:
               MYSQL_ROOT_PASSWORD: wordpress # là các tham số cấu hình cho container chạy serivce mysql
               MYSQL_DATABASE: wordpress #
               MYSQL_USER: wordpress #
               MYSQL_PASSWORD: wordpress #

           wordpress:
             depends_on:
               - db
             image: wordpress:latest
             ports:
               - "8000:80"
             restart: always
             environment:
               WORDPRESS_DB_HOST: db:3306 # tham số cấu hình phần cơ sở dữ liệu trong file wp_config.php
               WORDPRESS_DB_PASSWORD: wordpress #.. 
               
Chú giải: 

* version: '2': Chỉ ra phiên bản docker-compose sẽ sử dụng.

* services:: Trong mục services, chỉ ra những services (containers) mà ta sẽ cài đặt. Ở đây, tạo sẽ tạo ra services tương ứng với 2 containers là db và wordpress.

##### Trong services db:

* image: chỉ ra image sẽ được sử dụng để create containers. Ngoài ra, bạn có thể viết dockerfile và khai báo lệnh build để containers sẽ được create từ dockerfile.

* volumes: mount thư mục data trên host (cùng thư mục cha chứa file docker-compose) với thư mục /var/lib/mysql trong container.

* restart: always: Tự động khởi chạy khi container bị shutdown.

* environment: Khai báo các biến môi trường cho container. Cụ thể là thông tin cơ sở dữ liệu.

###### Trong services wordpress:

* depends_on: db: Chỉ ra sự phụ thuộc của services wordpress với services db. Tức là services db phải chạy và tạo ra trước, thì services wordpress mới chạy.

* ports: Forwards the exposed port 80 của container sang port 8000 trên host machine.

* environment: Khai báo các biến môi trường. Sau khi tạo ra db ở container trên, thì sẽ lấy thông tin đấy để cung cấp cho container wordpress (chứa source code).
