Chức năng:  Dockerfile chứa tập hợp các lệnh để docker có thể đọc hiểu và thực hiện để đóng gói thành một image theo yêu cầu người dùng
Cấu trúc của Dockerfile

FROM <tên image thừa kế>
RUN <command chạy>
CMD 
EXPOSE

### Các command trong dockerfile

1. FROM

Dùng để chỉ ra image được build từ đâu (từ image gốc nào)

    FROM ubuntu

    hoặc có thể chỉ rõ tag của image gốc

    FROM ubuntu14.04:lastest
    
2. RUN

Dùng để chạy một lệnh nào đó khi build image, nghĩa là ban đầu nó sẽ chạy images gốc tạo container và chạy những lệnh này trong container gốc và xong thì nó nén lại thành images mới

    FROM ubuntu
    RUN apt-get update
    RUN apt-get install curl -y

3. CMD

Định nghĩa lệnh sẽ chạy ngay khi tạo container từ images mới này (images từ dockerfile)

          FROM ubuntu
          RUN apt-get update
          RUN apt-get install wget -y
          CMD curl ifconfig.io

4. MAINTAINER
   
Dùng để đặt tên giả của images.

      MAINTAINER <name>
      
5. COPY: 

Copy một file từ host machine tới docker image. Chỉ thị COPY, copy file, thư mục (src) và thêm chúng vào filesystem của container (dest). 

      COPY <src>... <dest>
      COPY ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
      
6. ENV: 

Định nghĩa các biến môi trường 

      COPY nginx.conf /etc/nginx/defaults 
      
7. WORKDIR: Định nghĩa directory cho CMD
 
8. VOLUME: 
 
 Cho phép truy cập / liên kết thư mục giữa các container và máy chủ (host machine)

9. EXPOSE

Thông báo cho Docker rằng container sẽ lắng nghe trên các cổng được chỉ định khi chạy. Lưu ý là cái này chỉ để khai báo container sẽ chạy port này, ví dụ là 8080. nếu là images nginx thì container sẽ chạy ngay 2 port là 80 mặc định và 8080 khai báo. Chú ý, nó không pubish port ra ngoài hosts, vì vậy nếu gõ http://IP:8080 thì sẽ không hiện banner nginx. 

      EXPOSE 8080 

Ví dụ: 


      #Download base image ubuntu 16.04
      FROM ubuntu:16.04

      # Update Software repository
      RUN apt-get update

      # Install nginx, php-fpm and supervisord from ubuntu repository
      RUN apt-get install -y nginx php7.0-fpm supervisor && \
          rm -rf /var/lib/apt/lists/*

      #Define the ENV variable
      ENV nginx_vhost /etc/nginx/sites-available/default
      ENV php_conf /etc/php/7.0/fpm/php.ini
      ENV nginx_conf /etc/nginx/nginx.conf
      ENV supervisor_conf /etc/supervisor/supervisord.conf

      # Enable php-fpm on nginx virtualhost configuration
      COPY default ${nginx_vhost}
      RUN sed -i -e 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' ${php_conf} && \
          echo "\ndaemon off;" >> ${nginx_conf}

      #Copy supervisor configuration
      COPY supervisord.conf ${supervisor_conf}

      RUN mkdir -p /run/php && \
          chown -R www-data:www-data /var/www/html && \
          chown -R www-data:www-data /run/php

      # Volume configuration
      VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]
      # Đường dẫn trên là trên container, còn trên hosts nó lưu tại /var/lib/docker/volumes 

      # Configure Services and Port
      COPY start.sh /start.sh
      CMD ["./start.sh"]

      EXPOSE 80 443

Tạo file cho việc COPY

      $ touch default
      $ touch supervisord.conf
      $ touch start.sh





      
      
      
      
      
### Chạy lệnh tạo image: Docker build -t <image:tag> .
