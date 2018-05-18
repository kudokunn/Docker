
#### Nhiều lúc cần các containter trong docker liên kết với nhau thì các container cần cùng một host và sử dụng bridge network

* Cách duy nhất giải quyết là dùng name, vừa dễ lại có thể map động nên khi IP thay đổi cũng không lo mất kết nối giữa các dịch vụ chạy trên các container khác nhau. Phương thức thực hiện của docker lại hơi khác nhau trong hai trường hợp: 

 + Sử dụng default bridge network 
 
 1. Để kết nối được các container qua name thì chỉ có cách dùng dns hoặc /etc/hosts
 
 1.1 Xem file hosts trong container 
 
 ![](/image/2.PNG)
 
 Cái 172.17.0.2	79f6f55e2eff là cho chính nó <localhost> không có thông tin để phân giải name sang IP cho container khác. Hosname thì không định nghĩa riêng khi chạy docker run -h thì nó lấy ID của containter là hostname
 
 1.2 => Muốn xem liên kết qua default bridge network thì làm:  
 
 Dùng giải pháp --link:

Giả sử có mô hình 
web -> redis -> db

Ở đây từ container web phải connect được đến redis và đến db.  Sau đó link theo tên, phải đảm bảo container được link phải tồn tại. Do đó phải run theo thư tự db -> redis -> web

        docker run -itd --name=db -e MYSQL_ROOT_PASSWORD=pass mysql:latest
        docker run -itd --name=redis redis:latest
        docker run -itd --name=web --link=redis --link=db nginx:latest
        docker exec -it web sh
        # ping redis
        PING redis (172.17.0.3): 56 data bytes
        64 bytes from 172.17.0.3: icmp_seq=0 ttl=64 time=0.245 ms
        ...
        # ping db
        PING db (172.17.0.2): 56 data bytes
        64 bytes from 172.17.0.2: icmp_seq=0 ttl=64 time=0.126 ms
 
 Lý do web container liên lạc được db và redis qua name là nhờ docker engine đã tự set thông tin link vào /etc/hosts của web container:

      # cat /etc/hosts

      172.17.0.3  redis 21cdcee10183
      172.17.0.2  db 158aed2b8df9

Link này chỉ link một chiều, từ db và redis bạn không thể ping đến web được. Nói chung dùng link thì rất bất tiện, phải định nghĩa rõ chiều kết nối, phải đảm bảo start container theo đúng thứ tự. Phương thức kết nối qua link đã bị xem là quá khư. Hiện tại nó chỉ còn được dùng để đảm bảo thứ tự khỏi chạy của các container

 + Sử dụng user-defined bridge network
 
 2. Trường hợp này docker sẽ có sẵn một built-in dns, bạn không cần thực hiện thao tác link qua link lại giữa các container nữa. 

2.1 Mỗi một user-defined network này sẽ có một built-in dns riêng.

      docker network create my-net
      docker network ls
      NETWORK ID          NAME                DRIVER
      716f591e185a        bridge              bridge              
      4b0041303d6d        host                host                
      7239bb9e0255        my-net              bridge              
      016cf6ec1791        none                null

      docker run -itd --name=web1 --net my-net nginx:latest
      docker run -itd --net my-net --name=redis1 redis:latest
      docker run -itd --name=db1 --net my-net -e MYSQL_ROOT_PASSWORD=pass mysql:latest

      docker exec -it web1 sh
      # ping db1
      PING db1 (172.18.0.4): 56 data bytes
      64 bytes from 172.18.0.4: icmp_seq=0 ttl=64 time=0.161 ms
      # ping redis1
      PING redis1 (172.18.0.3): 56 data bytes
      64 bytes from 172.18.0.3: icmp_seq=0 ttl=64 time=0.168 ms
      
 Và do không dùng link mà qua một dns built-in nên các container có thể kết nối lẫn nhau, từ db1 bạn có thể ping được redis1 và web1

      cat /etc/resolv.conf
      nameserver 127.0.0.11
      options ndots:0

Nếu trong default bridge network, cố tính set nameserver 127.0.0.11 cũng không active được built-in dns này. :))
