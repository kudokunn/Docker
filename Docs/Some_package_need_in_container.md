## Chú ý: Với một container thì Hệ điều hành linux kèm theo nó là rất mininal của minimal, nên có những lệnh rất cần thiết để làm việc trong container mà nó lại thiếu cần cài thêm cho nó

 ## 1. Cập nhật repo cho docker:

Họ Ubuntu: apt-get update

Họ Centos: yum repolist

## 2. Kiểm tra ip của một container: hostname -i
hoặc cài package: 

- Ubuntu: apt-get install net-tools

- Centos: yum install net-tools

## 3. Lệnh ping: 

- Ubuntu: apt-get install iputils-ping

- Centos: yum install iputils

## 4. Lệnh vim: 

- Ubuntu: apt-get install vim

- Centos: apt-get install vim 


