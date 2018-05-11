## Docker là kiểu ảo hoá sử dụng Container còn có cách gọi khác là "ảo hoá mức hệ điều hành" (operating system virtualization)

### Phân biệt Ảo hóa truyền thống (ảo hóa phần cứng) và ảo hóa mức hệ điều hành

#### Ảo hoá truyền thống 

Như: VMWare, Virtual Box, Parallel... Là kiểu ảo hóa máy chủ sử dụng chính phần cứng của nó: CPU, RAM, DISK ... để ảo hóa mục đích cung cấp phần cứng cho máy ảo sao cho giống như một máy thật bình thường => Phần cứng máy ảo sẽ chiếm thực tế luôn của máy chủ. 

* Ưu điểm: Bất kỳ hệ điều hành máy ảo nào cũng dùng được, không cần quan tâm đến máy chủ dùng hệ điều hành gì

* Nhược điểm: Do hạn chế cấu hình phần cứng máy chủ nên chỉ có thể tạo vài cái máy ảo là hết tài nguyên

#### Ảo hóa container:

Là ảo hóa mức hệ điều hành: Ở đây không ảo hóa trên phần cứng nữa mà ảo hóa container môi trường (đóng gói các môi trường) trên hệ điều hành. Mỗi container chứa đầy đủ các ứng dụng, service kèm theo file nhi phân, thư viện đảm bao cho service đó chạy ổn định, độc lập trên container => Như vậy mỗi Container ở đây được coi như một "máy ảo" mini trên hệ điều hành

* Ưu điểm: Nhẹ: 

1. là ảo hoá nhưng Container lại rất nhẹ. Hệ điều hành chính quản lý các Container ở đây như là môt process của hệ thống. Chỉ mất vài giây để start, stop hay restart một Container, và khi các container ở trạng thái Idle (chờ) chúng gần như không tiêu tốn tài nguyên CPU. Với một máy tính cấu hình thông thường. Nếu chạy máy ảo truyền thống chúng ta chỉ chạy được khoảng vài cái là cùng. Tuy nhiên nếu chạy bằng Container chúng ta có thể chạy vài chục thậm chí đến cả vài trăm cái.

2. Có thể tự tạo một Container từ các template có sẵn, cài đặt môi trường, service, sau đó lưu trạng thái Container lại như là một "image" và triển khai image đến bất kỳ chỗ nào mong muốn

* Nhược điểm: 

Do các Container sử dụng chung kernel với hệ điều hành chủ nên chúng ta chỉ có thể "ảo hoá" được các hệ điều hành mà hệ điều hành chủ hỗ trợ. Nếu hệ điều hành chủ là Linux thì chúng ta chỉ có thể ảo hoá được các hệ điều hành nhân Linux như Lubuntu, OpenSuse, LinuxMint ... chứ không thể tạo được một container Window được.
