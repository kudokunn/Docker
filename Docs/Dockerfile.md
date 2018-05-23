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

Ví dụ Dockerfile chạy service nginx và php-fpm container ubuntu


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

* File default

        touch defaults
        
            server {
            listen 80 default_server;
            listen [::]:80 default_server;

            root /var/www/html;
            index index.html index.htm index.nginx-debian.html;

            server_name localhost;

            error_log /var/log/nginx/error.log;
            access_log /var/log/nginx/access.log;

            location / {
                try_files $uri $uri/ =404;
            }

            location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.0-fpm.sock;
            }

            # deny access to .htaccess files, if Apache's document root
            # concurs with nginx's one
            #
            #location ~ /\.ht {
            #    deny all;
            #}
        }

File supervisord

            
      touch supervisord.conf

            [unix_http_server]
            file=/dev/shm/supervisor.sock   ; (the path to the socket file)

            [supervisord]
            logfile=/var/log/supervisord.log ; (main log file;default $CWD/supervisord.log)
            logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
            logfile_backups=10           ; (num of main logfile rotation backups;default 10)
            loglevel=info                ; (log level;default info; others: debug,warn,trace)
            pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
            nodaemon=false               ; (start in foreground if true;default false)
            minfds=1024                  ; (min. avail startup file descriptors;default 1024)
            minprocs=200                 ; (min. avail process descriptors;default 200)
            user=root             ;

            ; the below section must remain in the config file for RPC
            ; (supervisorctl/web interface) to work, additional interfaces may be
            ; added by defining them in separate rpcinterface: sections
            [rpcinterface:supervisor]
            supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

            [supervisorctl]
            serverurl=unix:///dev/shm/supervisor.sock ; use a unix:// URL  for a unix socket

            ; The [include] section can just contain the "files" setting.  This
            ; setting can list multiple files (separated by whitespace or
            ; newlines).  It can also contain wildcards.  The filenames are
            ; interpreted as relative to this file.  Included files *cannot*
            ; include files themselves.

            [include]
            files = /etc/supervisor/conf.d/*.conf


            [program:php-fpm7.0]
            command=/usr/sbin/php-fpm7.0 -F
            numprocs=1
            autostart=true
            autorestart=true

            [program:nginx]
            command=/usr/sbin/nginx
            numprocs=1
            autostart=true
            autorestart=true

File script

            touch start.sh

                #!/bin/sh

                /usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf


Thêm vào Dockerfile lệnh để gán quyền chạy cho file start.sh
    
                RUN chmod +x start.sh     
      
### Chạy lệnh tạo image: Docker build -t <image:tag> .

### Chạy tạo container từ images mới: 

            docker run -d -v /webroot:/var/www/html -p 9000:80 --name nginx-php nginx_image
            
Bây giờ thư mục /webroot được tạo trên hosts, ta có thể tạo file index.html để nó tự đồng bộ về container và chạy

            vim index.html
            
            <h1>Nginx and PHP-FPM 7 inside Docker Container</h1>



