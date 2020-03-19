║   		─SSL─
	║	 	NGINX							 PM2
	║		│					React app	  │
	║       	├── react  ───────────────────────────────────────────────┤
	║		│   │							  │	
	╠═══════════════╪═══╧══ Dockerfile-setup-react		 		  │
	║		│							  │	
	║		│ 					First server	  │
	║		├── node   ───────────────────────────────────────────────┤
	║		│   ├─ api/						  │	
	║		│   │							  │
	╠═══════════════╪═══╧══ Dockerfile-setup-node	     			  │
	║		│							  │
	║		│					Second server     │
	║		├── node   ───────────────────────────────────────────────┤
	║		│   ├─ api/						  │	
	║		│   │							  │
	╠═══════════════╪═══╧══ Dockerfile-setup-node	     			  │
	║		│							  │
	║		│					Third server      │
	║       	└── node   ───────────────────────────────────────────────┤
	║			├─ api/						  │	
	║			│						  │	
	╠═══════════════════════╧══ Dockerfile-setup-node	   		  │	
	║									  │ Mobile
	║									  └──────── react-native
	║											│
	╠═══════════════════════════════════════════════════════════════════════════════════════╧═Dockerfile-setup-react
	║   mongo											  	
	║   ├─ run.sh 										  
	║   ├─ set_mongodb_password.sh
	╠═══╧══ Dockerfile-setup-mongo
	║
	╚═ docker-compose.yml

===================================================================================
NGINX
Về cơ bản, NGINX cũng hoạt động tương tự như các web server khác. Khi bạn mở một trang web, trình duyệt của bạn sẽ liên hệ với server chứa website đó. Server sẽ tìm kiếm đúng file yêu cầu của website và gửi về cho bạn. Đây là một trình tự xử lý dữ liệu single – thread, nghĩa là các bước được thực hiện theo một trình tự duy nhất. Mỗi yêu cầu sẽ được tạo một thread riêng.
- Tuy nhiên, NGINX hoạt động theo kiến trúc bất đồng bộ (asynchronous) hướng sự kiện (event driven). Nó cho phép các threads tương đồng được quản lý trong một tiến process. Mỗi process hoạt động sẽ bao gồm các thực thể nhỏ hơn, gọi là worker connections dùng để xử lý tất cả threads.
- Worker connections sẽ gửi các yêu cầu cho worker process, worker process sẽ gửi nó tới master process, và master process sẽ trả lời các yêu cầu đó. Đó là lý do vì sao một worker connection có thể xử lý đến 1024 yêu cầu tương tự nhau. Nhờ vậy, NGINX có thể xử lý hàng ngàn yêu cầu khác nhau cùng một lúc.

0. access_log & error_log
Đây là những tệp tin mà NGINX sẽ sử dụng để log bất kỳ lỗi và số lần truy cập. Các bản ghi này thường được sử dụng để gỡ lỗi hoặc sửa chữa.

1. worker_process
Xác định có bao nhiêu cores của CPU làm việc với Nginx. Nginx sẽ sử dụng một CPU để xử lý các tác vụ của mình. Tùy theo mức độ hoạt động của web server mà chúng ta có thể thay đổi lại thiết lập này.

2. events
NGINX sử dụng mô hình xử lý kết nối dựa trên sự kiện(event) nên các directive được định nghĩa trong context này sẽ ảnh hưởng đến connection processing được chỉ định. Ví dụ ở trên là cấu hình số worker connection mà mỗi worker process có thể xử lý được.

3. worker_connections  
cho biết số lượng connection mà mỗi worker_process có thể xử lý. Mặc định, số lượng connection này được thiết lập là 1024

4. http 
Khi cấu hình Nginx như một web server hoặc reverse proxy, http context sẽ giữ phần lớn cấu hình. Context này sẽ chứa tất cả các directive và những context(block directive) cần thiết khác để xác định cách chương trình sẽ xử lý các kết nối HTTP và HTTPS.

5. include
Chỉ thị include (include /etc/nginx/mime.types) của nginx có vai trò trong việc thêm nội dung từ một file khác vào trong cấu hình nginx. Điều này có nghĩa là bất cứ điều gì được viết trong tập tin mime.types sẽ được hiểu là nó được viết bên trong khối http {}. Điều này cho phép bạn bao gồm một số lượng dài của các chỉ thị trong khối http {} mà không gây lộn xộn lên các tập tin cấu hình chính. Và nó giúp tránh quá nhiều dòng mã cho mục đích dễ đọc

6. default_type
Định nghĩa kiểu MIME mặc định. Khi Nginx dc yêu cầu một file, định dạng file dc khớp với kiểu dc khai báo trong lock types để trả về kiểu MIME phù hợp trong trường Content-type trong HTTP response header. Nếu định dạng file không khớp với bất kì kiểu MIME nào, thì giá trị mặc định của chỉ thị default_type sẽ được sử dụng.

7. resolver
- Xác định tên verver (name server) dc sử dụng để Nginx chuyển hostname thành địa chỉ IP và ngược lại. (resulve: # use local DNS)
- Chỉ rõ các máy chủ phân giải tên miền (DNS) được sử dụng bởi Nginx để phân giải hostname của các địa chỉ IP và ngược lại. Các kết quả truy vấn DNS được lưu trong bộ nhớ cache trong 1 thời gian, hoặc được chỉ rõ bởi giá trị TTL (Time-to-live) của máy chủ DNS, hoặc được chỉ rõ bởi giá trị thời gian cho đối số hợp lệ

8. upstream backend
- định nghĩa một cụm mà bạn có thể yêu cầu proxy. Nó thường được sử dụng để xác định cụm máy chủ web để cân bằng tải hoặc cụm máy chủ ứng dụng để định tuyến / cân bằng tải.
- Ở đây backend1 và backend2 chính là server name của 2 máy chủ web, ta có thể thay bằng địa chỉ IP tương ứng.
- Để truyền các request từ người dùng vào một group các server, tên của group được truyền vào với directive proxy_pass (hoặc fastcgi_pass, memcached_pass, uwsgi_pass, scgi_pass tùy thuộc vào giao thức). Trong bài viết này, virtual server chạy trên NGINX sẽ truyền tất cả các request tới backend upstream server

8.1. upstream
Ngữ cảnh upstream định nghĩa một pool của các server cái NGINX sẽ ủy quyền các request tới. Sau khi chúng ta tạo một khối upstream và định nghĩa một server bên trong nó chúng có thể tham chiếu nó bằng tên bên trong các khối location. Thêm nữa, một ngữ cảnh upstream có thể có nhiều server được gán trong nó vì rằng NGINX sẽ làm một vài load balancing khi ủy quyền các request.

8.2. Thuật toán cân bằng tải round-robin, least_conn, least_time, ip_hash, ...
- round-robin: Các request lần lượt được đẩy về 2 server backend1 và backend2 theo tỉ lệ dựa trên server weights, ở đây là 1:1

8.3 Bảo toàn session người dùng
Hãy thử tưởng tượng bạn có một ứng dụng yêu cầu đăng nhập, nếu khi đăng nhập, session lưu trên Backend 1, sau một hồi request lại được chuyển tới Backend 2, trạng thái đăng nhập bị mất, hẳn là người dùng sẽ vô cùng nản. 
- NGINX PLUS có cung cấp sticky directive, giúp NGINX tracks user sessions và đưa họ tới đúng upstream server.
- Dùng ip_hash làm phương thức cân bằng tải
+ Hash được sinh từ 3 chỉ số đầu của một IP, do đó tất cả IP trong cùng C-class network sẽ đc điều hướng tới cùng một backend.
+ Tất cả user phía sau một NAT sẽ truy cập vào cùng một backend.
+ Nếu ta thêm mới một backend, toàn bộ hash sẽ thay đổi, đương nhiên session sẽ mất.

9.  sendfile
cho phép send file.
giá trị mặc định sendfile: off

10. keepalive_timeout
- Vì các kết nối dạng keepalive được mở trong thời gian nhất định, những kết nối này cũng tốn một lượng tài nguyên. Vì thế ta cần cấu hình thời gian timeout của các kết nối này một cách hợp lý dựa trên ứng dụng/website và traffic của ứng dụng. Tùy theo cấu hình mà có thể tăng/giảm hiệu năng của ứng dụng khi gặp tải cao.
- keepalive_timeout time1[time2]
giá trị mặc định : 75. Tham số thứ 2 được đưa vào giá trị Keep-alive của HTTP response header để báo cho trình duyệt biết tự đóng kết nối trước khoảng thời gian time out do server chỉ định

11. client_max_body_size
Nó là kích thước tối đa của dữ liệu yêu cầu từ client. Nếu kích thước này bị vượt qua, Nginx trả về 1 lỗi HTTP 413 Request entity too large. Thiết lập này đặc biệt quan trọng nếu chúng ta cho phép người dùng tải các tập tin lên máy chủ qua HTTP

12. server
Ngữ cảnh server định nghĩa một server ảo để xử lý các request từ client của bạn. Bạn có thể có nhiều khối server, và NGINX sẽ chọn một trong số chúng dựa trên các chỉ thị listen và server_name.
- Trong một khối server, chúng ta định nghĩa nhiều ngữ cảnh location được sử dụng để quyết định cách xử lý các request từ client. Bất cứ khi nào một request đến, NGINX sẽ thử khớp URI tới một trong số các định nghĩa location và xử lý nó cho phù hợp

13. listen
- Chỉ rõ địa chỉ IP và/hoặc port được dùng bởi socket phục vụ website. Các website thường được phục vụ trên port 80 (giá trị mặc định) qua HTTP, hoặc 443 qua HTTPS.
- Cú pháp: listen [address] [:port] [additional options];
- dditional options: 
+ ssl: Chỉ rõ website sẽ sử dụng SSL.

14. server_name 
- Đăng ký 1 hoặc nhiều hostname cho khối server. Khi Nginx nhận 1 yêu cầu HTTP, nó so sánh giá trị Host trong phần header của yêu cầu với tất cả các khối server đang có. Khối server đầu tiên khớp với hostname này sẽ được chọn.
- Nếu không có khối server nào khớp với hostname trên, Nginx chọn khối server đầu tiên khớp với các thông số của chỉ thị listen (ví dụ như listen *:80 sẽ bắt tất cả các yêu cầu nhận được trên port 80), ưu tiên khối đầu tiên có tùy chọn mặc định được cho phép trên chỉ thị listen.

15. ssl
SSL là viết tắt của từ Secure Sockets Layer. Đây là một tiêu chuẩn an ninh công nghệ toàn cầu tạo ra một liên kết được mã hóa giữa máy chủ web và trình duyệt. Liên kết này đảm bảo tất cả các dữ liệu trao đổi giữa máy chủ web và trình duyệt luôn được bảo mật và an toàn. 

SSL đảm bảo rằng tất cả các dữ liệu được truyền giữa các máy chủ web và các trình duyệt được mang tính riêng tư, tách rời. SSL là một chuẩn công nghiệp được sử dụng bởi hàng triệu trang web trong việc bảo vệ các giao dịch trực tuyến với khách hàng của họ.
HTTP -> HTTPS


16. location
- Sau khi đã chọn được server block nào sẽ tiếp nhận request này thì nginx sẽ tiếp tục phân tích URI của request để tìm ra hướng xử lí của request dựa vào các block location có syntax như sau:
location optional_modifier location_match {
    . . .
}
- optional_modifier: bạn có thể tạm hiểu nó là kiểu so sánh để tìm ra để đối chiếu với location_match. Có mấy loại option như sau:
+ (none): Nếu không khai báo gì thì NGINX sẽ hiểu là tất cả các request có URI bắt đầu bằng phần location_match sẽ được chuyển cho location block này xử lí.
+ = : Khai báo này chỉ ra rằng URI phải có chính xác giống như location_match (giống như so sánh string bình thường).
+ ~ : Sử dụng regular expression cho các URI
+ ~* : Sử dụng regular expression cho các URI cho phép pass cả chữ hoa và chữ thường

16.1. index directive
index direct nằm bên trong location luôn được nginx trỏ tới đầu tiên khi xử lí điều hướng request. Định nghĩa trang mặc định mà Nginx sẽ phục vụ nếu không có tên tập tin được chỉ rõ trong yêu cầu (nói cách khác, trang chỉ mục). Chúng ta có thể chỉ rõ nhiều tên tập tin và tập tin đầu tiên được tìm thấy sẽ được sử dụng. Nếu không có tập tin cụ thể nào được tìm thấy, Nginx sẽ hoặc là cố gắng phát sinh 1 chỉ mục tự động của các tập tin
location = / {
    root html;
    index index.html;
}
- Nginx sẽ đọc root directive để xác định thư mục chứa trang client yêu cầu. Thứ tự các trang được ưu tiên sẽ được khai báo trong index directive.
Nếu không tìm được nội dung mà client yêu cầu, nginx sẽ điều hướng sang location context khác và thông báo lỗi cho người dùng.


16.2  error_page directive
chỉ thị khi không tìm thấy file tham chiếu.
location / {
    error_page 404 = @fallback;
}

location @fallback {
    proxy_pass http://backend;
}

17. Có nhiều chỉ thị quan trọng có thể được sử dụng dưới ngữ cảnh location
- try_files sẽ cố gắng phục vụ các tệp tin tĩnh được tìm thấy trong thư mục được trỏ tới bởi chỉ thị gốc.
- proxy_pass sẽ gửi request tới một proxy server cụ thể.
- rewrite sẽ viết lại URI tới dựa trên một regular expression để một khối location có thể xử lý nó.

I. INSTALL by WSL
1. Open windowns powershell and enter:
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux

2. Search on your windows:
Turn Windows features on or off

3. Enable: Windows Subsystem for Linux

4. Enable: Vitual machine platforms

5. Download linux (wsl) on Microsoft store
I choise Ubuntu18.04

6. Create UNIX account

7. Open wsl and enter:
sudo apt-get update

sudo apt-get upgrade

sudo add-apt-repository ppa:nginx/stable

sudo apt-get update

sudo apt-get install -y nginx

note: Your nginx will install at C:\Users\<Username>\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs\etc\nginx

8. Start nginx by:
sudo service nginx start 

nginx will start default at: localhost:80

note: Some command nginx for wsl:

+ sudo systemctl start nginx 
sudo systemctl stop nginx 
sudo systemctl restart nginx

+ sudo service nginx start
sudo service nginx stop
sudo service nginx restart

+ sudo /etc/init.d/nginx start
sudo /etc/init.d/nginx stop
sudo /etc/init.d/nginx restart

9. Let's play

10. Unistall nginx by wsl
- Removes all but config files: 
sudo apt-get remove nginx nginx-common

- Removes everything:
sudo apt-get purge nginx nginx-common

- After using any of the above commands, use this in order to remove dependencies used by nginx which are no longer required:
sudo apt-get autoremove


II. Nginx for windowns
1. Download at: http://nginx.org/en/download.html
choose nginx/windowns-<version>

2. Unpack and use nginx by the ways double click nginx.exe

3. Some example for the drive C: root directory:
cd c:\
unzip nginx-1.17.9.zip
cd nginx-1.17.9
start nginx

- Run the tasklist command-line utility to see nginx processes:
tasklist /fi "imagename eq nginx.exe"

Image Name           PID Session Name     Session#    Mem Usage
=============== ======== ============== ========== ============
nginx.exe            652 Console                 0      2 780 K
nginx.exe           1332 Console                 0      3 112 K

- If nginx does not start, look for the reason in the error log file:
logs\error.log

- If an error page is displayed instead of the expected page, also look for the reason in the logs\error.log file.

- nginx/Windows runs as a standard console application (not a service), and it can be managed using the following commands:
+ nginx -s stop 		fast shutdown
+ nginx -s quit	    	graceful shutdown
+ nginx -s reload		changing configuration, starting new worker processes with a new configuration, graceful shutdown of old worker processes
+ nginx -s reopen		re-opening log files

4. Let's play


III. Nginx in docker


===================================================================================
PM2
Tự động chạy ứng dụng Nodejs, tự động restart khi lỗi. PM2 còn kèm theo nhiều plugin với nhiều chức năng như: cân bằng tải, update source no zero down time (cập nhật mã nguồn mà không làm tắt server), scale (tự động mở rộng khả năng chịu tải), tự động cập nhật source code khi Git được update, …

- Quản lý các process, bao gồm tự động restart app khi bị chết hoặc reboot hệ thống.
- Giám sát ứng dụng
- Khai báo cấu hình qua JSON file
- Quản lý log
- Cluster mode
- Chạy các kịch bản lệnh cho hệ thống
- Seamless updates
- Cho phép tích hợp các module cho hệ thống


// Khởi động lại ứng dụng thông qua app name
pm2 restart <app_name>
 
// Tải lại lại ứng dụng thông qua app name
pm2 reload <app_name>
 
// Dừng ứng dụng thông qua app name
pm2 stop <app_name>
 
// Xóa ứng dụng thông qua app name
pm2 delete <app_name>
 
// Khởi động lại ứng dụng thông qua app name
pm2 restart <app_name>
 
// Tải lại lại ứng dụng thông qua app name
pm2 reload <app_name>
 
// Dừng ứng dụng thông qua app name
pm2 stop <app_name>
 
// Xóa tất cả ứng dụng đang chạy của Pm2
pm2 kill

// Xem nhật ký (logs) trong PM2
pm2 logs




