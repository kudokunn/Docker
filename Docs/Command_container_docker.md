- xem các contaier đang chạy: docker ps
- Xem các container đang chạy và đã chạy: docekr ps -a
- Xóa một container đã hoặc đang chạy: docker rm -f <ID của container> hoặc docker rm -f <name container>
Ex: docker rm -f abcd12342 hoặc docker rm -f mysqlwp
- Start hoặc dừng một container: docker start/stop <ID container>
- Dừng tất cả container đang chạy: docker stop $(docker ps -qa)
- Xóa tất cả các container đang và đã chạy: docker rm$(docker ps -qa)
- Xem logs của một docker đang chạy: docker logs <ID container>
- Chạy command trong container đang chạy: docker exec -it <tên container hoặc ID container đang chạy> bash

Lỗi khi rm môt container: docker run <ID container>

[root@template-centos7 ~]# docker rm graphite -f
Error response from daemon: driver "devicemapper" failed to remove root filesystem for e5e42ae7002f86a44db4d1378e395135949dd59e415b62bac1485e607e972e0c: remove /var/lib/docker/devicemapper/mnt/c4d4523bd3d9aa218d7da2cd0118d4976ab4bcae9c5642033763511fed372d69: device or resource busy

* Cách fix: Do chính service nào đó trên hosts chạy docker container có đang dùng đến hay liên quan gì đó đến thư mục /var/lib/docker/devicemapper/mnt/c4d4523bd3d9aa218d7da2cd0118d4976ab4bcae9c5642033763511fed372d69, cần tìm ra pid của service đó đang dùng thư mục này kill nó xong mới rm container đó được. => Như vậy

* Cách tìm: Dùng lệnh: 

[root@template-centos7 mnt]# grep c4d4523bd3d9aa218d7da2cd0118d4976ab4bcae9c5642033763511fed372d69 /proc/*/mountinfo

    /proc/32298/mountinfo:249 242 253:3 / /var/lib/docker/devicemapper/mnt/c4d4523bd3d9aa218d7da2cd0118d4976ab4bcae9c5642033763511fed372d69       rw,relatime shared:212 - xfs /dev/mapper/docker-253:0-202371570-c4d4523bd3d9aa218d7da2cd0118d4976ab4bcae9c5642033763511fed372d69            rw,nouuid,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota

=> Pid là 32298

* Xem pid này của service nào: 

        [root@template-centos7 mnt]# ps -p 32298 -o comm=
        mysqld
        
* Status serivce có chạy, xong hãy kill pid đó: kill -9 32298 và xóa container đó. Xong phải restart lại service mysqld ngay lâp tức để nó khởi động, cần xem nếu serivce đó quá quan trọng không được tắt thì chịu, không xóa được container đó.
