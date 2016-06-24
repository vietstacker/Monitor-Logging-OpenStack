#Mục lục


## I. Log trong Linux
Một trong những điều khiến GNU/Linux trở thành một hệ điều hành tuyệt vời đó là các thông tin về mọi thứ diễn ra trong hệ thống, hay những ứng dụng
 chạy trên nó sẽ luôn được ghi lại. Mỗi khi có sự cố xảy ra - và thực sự nó rất thường xuyên xảy ra, thì những thông tin này trở nên vô cùng quý giá trong việc 
khắc phục. Các thông tin trên thường được ghi lại dưới dạng văn bản, được gọi là cái file log.

- Tại sao ứng dụng của tôi không thể khởi động lên được?
- Có bao nhiêu người đã SSH vào server của tôi trong vòng 3 ngày qua?
- Ai đó hôm qua đã VPN vào hệ thống và lấy đi 1 tài liệu rất quan trọng, tôi có thể kiểm tra được đó là ai không?
- Tôi có thể thống kê được số VM được tạo, được xóa trên hệ thống cloud của tôi không?
...
Cả trăm câu hỏi như vậy được đặt ra, và đều có chung một câu trả lời : 

Hãy xem các file log !!!

Các file log sẽ cho bạn tất cả các thông tin cần thiết, miễn là bạn biết được file log cần xem nằm ở đâu.

Các file log được ghi dưới dạng văn bản, vậy nên bạn có thể vô cùng dễ dàng dùng các kỹ thuật đọc và tìm kiếm văn bản trong Linux để đọc log như 
``find``, ``awk``, ``grep``... 

Hệ thống với quy mô 1 vài chục server?. Ok, fine! Bạn vẫn có thể dễ dàng dễ dàng SSH vào từng server, kiểm tra các file log. Nhưng với hệ thống
lên tới vài trăm, vài nghìn máy chủ thì sao? Bạn không thể lúc nào cũng biết chính xác được mình cần tìm đến file log nào. 

Các giải pháp log tập trung bắt đầu ra đời. Các server, bằng những cách khác nhau, sẽ đẩy các file log tại máy local tập trung về một máy log 
server. Các giải pháp log tập trung không chỉ giúp người quản trị có thể quản lý log của các máy client một cách dễ dàng hơn. Mà còn giúp người 
quản trị khai thác tối đa được lợi ích từ các file log.

**Mô hình log tập trung**

![NOTE7](images/i7.png)


Dựa vào các trải nghiệm khi làm việc với các giải pháp log tập trung khác nhau, cũng như do đặc trưng và yêu cầu của hệ thống OpenStack đặt ra,
 tôi xin được giới thiệu giải pháp log tập trung đang được tôi tin dùng, đó là Graylog.

## II. Giới thiệu về Graylog
###1. Giới thiệu chung
 Graylog là phần mềm mã nguồn mở quản lý log tập trung, bắt đầu phát triển vào 2010 bởi Lennart Koopman (người Đức) với tên Graylog2. Vào tháng 
 2 năm 2014, phiên bản Graylog2 version 0.20.0 chính thức được phát hành. Chỉ chưa đến 1 năm sau, tháng 1 năm 2015, phiên bản Graylog 1.0 Beta
ra đời, đổi tên từ Graylog2 thành Graylog, và Graylog Inc được thành lập. 

 Từ version 1.0, Graylog đã trải qua 5 phiên bản, version mới nhất là Graylog 2.0.3, phiên bản ổn định là Graylog 1.3.
 
###2. Đặc điểm nổi bật
- Việc triển khai và cài đặt kha dễ dàng.
- Graylog có thể nhận log từ rất nhiều khác nhau : log của các server Linux, Window, thiết bị mạng như router, switch, firewall, các thiết bị 
lưu trữ như CEPH.
- Sử dụng công cụ chuyên dụng để tìm kiếm là Elastisearch, giúp việc tìm kiếm các bản tin log đuọc dễ dàng, nhanh chóng và chính xác.
- Phân tích các số liệu các được từ các file log thành dạng số liệu, biểu đồ thống kê, tổng hợp lại trong các dashboard.
- Cơ chế cảnh báo qua Email, Slack.
- Khả năng tích hợp mạnh mẽ với các phần mềm khác như sử dụng cơ chế plugin, bạn có thể dễ dàng tích hợp Graylog với các LDAP, Graphite/Ganglia, 
Logstash, NetFlow..

Đặc biệt, không chỉ tự mình thu thập log, từ phiên bản 2.0, Graylog có thể đóng vai trò quản lý và trung gian. Với các server có sẵn có chế thu
thập log như nx-log, hay logstash. Bạn có thể thu thập log từ chính các cơ chế có sẵn này. Graylog còn có thể làm trung gian, đẩy log thu được 
tới bên thứ 3 sử dụng.

###3. Cấu trúc
![NOTE4](images/i4.png)
Graylog có 4 thành phần chính :
- Graylog Server:  Nhận, xử lý các bản tin và truyền thông với các thành phần khác – Cần CPU. 
- Elasticsearch:	 Công cụ lưu trữ, tìm kiếm dữ liệu - tất cả phụ thuộc vào tốc độ I/O, Cần RAM. 
- MongoDB:	 	 Lưu trữ metadata ( file cấu hình…). Chỉ cần cấu hình thấp.
- Web Interface: 	 Cung cấp giao diện cho người dùng.

Từ phiên bản Graylog 2.0, thành phần web-interface đuọc tích hợp cùng với graylog-server.

###4. Mô hình logical
Dựa vào document của Graylog, có 2 mô hình logical được khuyến cáo sử dụng. 

Mô hình Minimum setup. Cả 4 thành phần của Graylog được cấu hình trên cùng 1 con server.

![NOTE5](images/i5.png)

Mô hình Bigger production setup. Các thành phần của Graylog được tách riêng, và ta có thể triển khai các cơ chế như Load Balancer, HA, Cluster
cho từng thành phần.

![NOTE6](images/i6.png)

##III. Triển khai Graylog trên mô hình OpenStack Mitaka

Tôi sẽ triển khai phiên bản Graylog ổn định nhất là version 1.3

###1. Cấu hình máy Graylog Server
OS: Ubuntu Server 14.04 64 bit
RAM: 4GB
CPU: 2x2
NIC1: eth1: 172.16.69.0/24, gateway 172.16.69.1 (sử dụng card NAT hoặc Bridge VMware Workstation)
HDD: +60GB
###2. Chạy script
#### Mô hình 
![Graylogmodel](images/i3.png)

####2.1 : Cài Graylog Server

 - Với Graylog Server
 
 ```sh
 wget https://github.com/manhdinh/ghichep-graylog/blob/master/graylog/graylog-scripts/graylog-server.sh
 ```

 
 #####Một số *lưu ý* khi chạy script:
 
 - Nhập password cho admin khi đăng nhập vào Web-interface
 
 ![NOTE1](images/i1.png)

 - Ấn phím *ENTER* để tiếp tục
 
 ![NOTE2](images/i2.png)
 
####2.2. Hướng dẫn thu thập log của 2 node controller và compute

#####Step1 : 

- Đăng nhập WEB Interface của Graylog : http://IP-Graylogserver:9000
 
 ![NOTE8](images/i8.png)

#####Step 2 : 
- Tạo Input trên Graylog. Input cũng giống như 1 địa chỉ, giúp các máy Client tìm đến và giao tiếp với Graylog server.

 ![NOTE9](images/i9.png)
 ![NOTE10](images/i10.png)
 ![NOTE11](images/i11.png)
 ![NOTE12](images/i12.png)