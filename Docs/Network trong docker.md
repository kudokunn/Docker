Vấn đề cập đến Bridge Network trong Docker:
1. Mặc định docker sẽ dùng loại mạng bridge này, thực tế docker hỗ trợ 3 loại card mạng: bridge, host, only.
2. Khi cài đặt docker service thì sẽ tạo card mạng docker0 trên hosts (máy cài đặt docker) => khi khởi chạy một container trên docker trong host thì một interface mới đại diện cho container đó sẽ hiện ra 


         docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
          link/ether 02:42:f1:59:1a:46 brd ff:ff:ff:ff:ff:ff
          inet 172.17.0.1/16 scope global docker0
             valid_lft forever preferred_lft forever
          inet6 fe80::42:f1ff:fe59:1a46/64 scope link 
             valid_lft forever preferred_lft forever
         vethc4c9915@if91: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP 
           link/ether de:bf:41:5b:54:f6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
           inet6 fe80::dcbf:41ff:fe5b:54f6/64 scope link 
             valid_lft forever preferred_lft forever
