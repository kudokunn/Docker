## Nhớ: Docker chỉ tạo ra môi trường nhanh chóng, xong còn lại là vẫn vào container cấu hình như một service gì đấy bình thường

Sử dụng 2 image docker: mysql và wordpress

### Bước 1: Pull images về

    $ sudo docker pull mysql

    $ sudo docker pull wordpress
    
### Bước 2: Tạo liên kết giữa hai docker bằng việc tạo một built-in dns riêng.

    $ sudo docker network create my-net
    
    $ sudo docker network ls ### chú ý cái thứ 3 là my-net đấy có ip là 127.0.0.11
    
    NETWORK ID          NAME                DRIVER
    716f591e185a        bridge              bridge              
    4b0041303d6d        host                host                
    7239bb9e0255        my-net              bridge              
    016cf6ec1791        none                null
    
### Bước 3: Chạy image để tạo container:

 * Với mysql: 
   
        $ sudo docker run -it --name=mysqlwp --net my-net -e MYSQL_ROOT_PASSWORD=pass -d mysql 
        $ sudo docker exec -it mysqlwp bash
        $ hostname -i
        172.18.0.2
        # --net my-net: cần liên kết với network my-net đã tạo trước 
        # Tham số -e MYSQL_ROOT_PASSWORD=pass: cài thêm trước tham số pass_root vào mysql, đỡ công vào setup pass root 
        
 * Với wordpress:
 
        $ sudo docker run -it --name wordpress --net my-net -p 8080:80 -d wordpress 
        $ sudo docker exec -it wordpress bash
        $ sudo hostname-i
        172.18.0.3
        # --net my-net: cần liên kết với network my-net thông qua nó mà ping đến cái kia, nó được coi như là nameserver để cài nó
 
 * Test ping thử:
 
    + Từ contanter wordpress
    
          # ping mys
          PING mysqlwp (172.18.0.2): 56 data bytes
          64 bytes from 172.18.0.2: icmp_seq=0 ttl=64 time=0.161 ms
        
    + Từ container mysqlwp    
    
          # ping wordpress
          PING redis1 (172.18.0.3): 56 data bytes
          64 bytes from 172.18.0.3: icmp_seq=0 ttl=64 time=0.168 ms
        


    
