### Vấn đề cập đến Bridge Network trong Docker:

1. Mặc định docker sẽ dùng loại mạng bridge này, thực tế docker hỗ trợ 3 loại card mạng: bridge, host, only.

2. Khi cài đặt docker service thì sẽ tạo card mạng docker0 trên hosts (máy cài đặt docker) => khi khởi chạy một container trên docker trong host thì một interface mới đại diện cho container đó sẽ hiện ra 

![](/image/1.PNG)

3. Trong container đó cũng tạo ra một card mạng cho nó (chưa biết kiểm tra ra sao -_-) nhưng có thể xem ip của nó qua: 

         docker inspect <id container or container's name> | grep IPAddr
         note 1: Lần lượt mỗi container được tạo ra sẽ dùng ip kế tiếp trong dải 172.17.0.0/16 Với 16 bist dành cho host thì số host có thể có tối đa là 2^16 - 2 container trong một host.
         note 2: và tất cả containter sẽ nhận IP 172.17.0.1 là gateway 
         
4. Do các container đều dùng host làm gateway nên bridge network cho phép:

       Các container trên cùng host giao tiếp với nhau được
       Các container có thể gửi packet ra bên ngoài host được.
       
5. Khi start service docker thì docker đã tạo ra một số rules để đảm bảo là đủ để container có thể sử dụng host làm gateway để truy cập ra internet. Không có các rule này thì docker vẫn chạy. Ví dụ: 

        *nat
         :PREROUTING ACCEPT [67:4224]
         :INPUT ACCEPT [3:1248]
         :OUTPUT ACCEPT [1:160]
         :POSTROUTING ACCEPT [1:160]
         :DOCKER - [0:0]
         -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
         -A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
         -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
         -A DOCKER -i docker0 -j RETURN
         COMMIT


         *filter
         :INPUT ACCEPT [121:23360]
         :FORWARD ACCEPT [0:0]
         :OUTPUT ACCEPT [48:6144]
         :DOCKER - [0:0]
         :DOCKER-ISOLATION - [0:0]
         -A FORWARD -j DOCKER-ISOLATION
         -A FORWARD -o docker0 -j DOCKER
         -A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
         -A FORWARD -i docker0 ! -o docker0 -j ACCEPT
         -A FORWARD -i docker0 -o docker0 -j ACCEPT
         -A DOCKER-ISOLATION -j RETURN
         COMMIT
         
Giải thích: 

-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
ý nghĩa: cho biết là thêm một rule vào POSTROUTING chain của nat table của iptables, tất cả các packet có source ip là 172.17.0.0/16 không đi đến interface docker0 sẽ được MASQUERADE nghĩa là nếu package này sẽ được sửa đổi source IP sao cho giống out interface của host mà nó đi qua.

6.
Để từ outside world truy cập dịch vụ bên trong container, bạn phải expose service trong container. Thực chất là một kiểu port forwarding. Ví dụ: [root@localhost Docker_Centos6]# docker run -it -p 81:80 image_name => Ở đây port trong container 80 sẽ được map đến 81 trong host.

+ Nhưng port đã được map nhưng dịch vụ trong container phải được start thì mới dùng đươc.

+ Khi stop container, các rule phục vụ cho map port này cũng tự động được gỡ bỏ.

+ Trường hợp nếu khi run bạn quên không map port thì chỉ cần tự bổ sung iptables rule cho DNAT là cũng map port được thôi. <Hơi khó hiểu >

