# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 4: HIGH AVAILABILITY & TRAFFIC ROUTING

---

### CÂU 1 (Mức độ: Dễ - Trung bình - Load Balancer & Firewall Whitelisting)
Một công ty cung cấp dịch vụ B2B đang triển khai một cổng thanh toán trực tuyến chạy trên các EC2 instances trong một Auto Scaling Group trải dài trên 3 Availability Zones. Khách hàng doanh nghiệp của công ty có chính sách an ninh mạng nghiêm ngặt: Tường lửa (corporate firewall) của họ chỉ cho phép lưu lượng truy cập đi ra (outbound) tới các **địa chỉ IPv4 tĩnh cố định (Fixed Static IPs)** đã được phê duyệt trong danh sách trắng (whitelist), và họ từ chối định tuyến dựa trên tên miền DNS có IP thay đổi liên tục.

Solutions Architect nên lựa chọn giải pháp cân bằng tải nào sau đây để đáp ứng yêu cầu trên?

- **A.** Triển khai một Application Load Balancer (ALB) trong Public Subnet và gán 3 địa chỉ Elastic IP trực tiếp vào cấu hình Network Interfaces của ALB.
- **B.** Triển khai một **Network Load Balancer (NLB)** trên 3 Availability Zones và gán một địa chỉ **Elastic IP cố định** cho mỗi Availability Zone.
- **C.** Triển khai một Application Load Balancer (ALB) kết hợp với AWS WAF, sau đó cấp phát danh sách CNAME tĩnh của ALB cho khách hàng.
- **D.** Triển khai một Gateway Load Balancer (GLB) kết hợp với AWS Transit Gateway để chia sẻ dải IP private của VPC tới khách hàng.

---

### CÂU 2 (Mức độ: Trung bình - Auto Scaling & Application Health Monitoring)
Một ứng dụng web thương mại điện tử chạy trên các máy chủ EC2 nằm trong một Auto Scaling Group (ASG) đứng phía sau một Application Load Balancer (ALB). Đôi khi, tiến trình web server (Nginx/NodeJS) bên trong instance bị lỗi đơ ứng dụng (hang/crash) và trả về mã lỗi `HTTP 502 Bad Gateway` cho người dùng. Tuy nhiên, hệ điều hành Linux và phần cứng máy chủ EC2 vẫn hoạt động bình thường, dẫn đến việc Auto Scaling Group không tự động phát hiện lỗi và không thay thế các instance bị hỏng này. Quản trị viên hiện đang phải kiểm tra và xóa máy chủ lỗi thủ công.

Giải pháp nào sau đây giúp hệ thống **tự động phục hồi (self-healing)** khi gặp lỗi trên với ít công sức quản trị nhất?

- **A.** Viết một kịch bản Cron job chạy định kỳ mỗi 5 phút trên từng EC2 instance để kiểm tra tiến trình web và tự khởi động lại máy chủ nếu tiến trình chết.
- **B.** Tạo một CloudWatch Alarm theo dõi chỉ số `CPUUtilization`, nếu CPU xuống dưới 5% trong 10 phút thì gửi lệnh qua AWS Systems Manager để restart instance.
- **C.** Chuyển đổi cấu hình **Health Check Type của Auto Scaling Group từ "EC2" sang "ELB"**, đồng thời cấu hình đường dẫn HTTP Health Check chuẩn xác trên Target Group của ALB.
- **D.** Thiết lập một ASG Lifecycle Hook ở trạng thái `autoscaling:EC2_INSTANCE_TERMINATING` để chuyển hướng traffic sang các instance khác.

---

### CÂU 3 (Mức độ: Nâng cao - Non-HTTP Protocols & Fast Global Failover)
Một công ty trò chơi điện tử (eSports) phát triển một tựa game thi đấu trực tuyến nhiều người chơi thời gian thực. Giao thức mạng phục vụ các trận đấu đòi hỏi giao tiếp hai chiều thời gian thực qua **giao thức UDP** nhằm đảm bảo tốc độ và độ trễ cực thấp. Người chơi phân bố trên toàn thế giới và kết nối tới hai cụm máy chủ game đặt tại hai AWS Regions: `us-east-1` (Bắc Mỹ) và `eu-west-1` (Châu Âu). Ngoài ra, hệ thống yêu cầu:
1. Cung cấp địa chỉ IP tĩnh duy nhất trên toàn cầu để tích hợp vào ứng dụng game client.
2. Có khả năng tự động chuyển hướng toàn bộ người chơi sang Region còn lại trong **dưới 1 phút** nếu một Region gặp sự cố thảm họa.

Giải pháp kiến trúc nào sau đây đáp ứng toàn bộ các yêu cầu trên?

- **A.** Triển khai **Amazon CloudFront** với Origin là cụm máy chủ game tại 2 Regions, kích hoạt CloudFront Origin Failover.
- **B.** Cấu hình **Amazon Route 53 với Geolocation Routing Policy**, kết hợp kiểm tra Route 53 Health Checks trỏ tới các Elastic IP của máy chủ game.
- **C.** Triển khai **AWS Global Accelerator** cung cấp **2 địa chỉ Static Anycast IP**, gắn các Network Load Balancer (NLB) ở cả hai Region làm endpoint và bật tính năng Health Checking liên tục.
- **D.** Thiết lập Route 53 với Latency-based Routing trỏ tới hai Application Load Balancers (ALB) đặt tại hai Region.

---

### CÂU 4 (Mức độ: Nâng cao - Route 53 Routing & Zone Apex Restrictions)
Một doanh nghiệp muốn triển khai thử nghiệm một phiên bản giao diện website mới (Phiên bản B). Họ muốn điều hướng chính xác **20% lưu lượng truy cập của người dùng** vào phiên bản B đang chạy trên cụm ALB mới, và duy trì **80% lưu lượng còn lại** ở phiên bản A đang chạy trên cụm ALB cũ. Yêu cầu đặc biệt là việc điều hướng này phải áp dụng trực tiếp cho **tên miền gốc (Zone Apex / Naked Domain)** của công ty là `example.com`.

Cấu hình DNS nào trên Amazon Route 53 đáp ứng đúng yêu cầu kỹ thuật trên?

- **A.** Tạo hai bản ghi **CNAME** cho tên miền `example.com` sử dụng chính sách **Weighted Routing**, gán trọng số 80 cho ALB phiên bản A và 20 cho ALB phiên bản B.
- **B.** Tạo hai bản ghi **A record (kích hoạt ALIAS = Yes)** cho tên miền `example.com` sử dụng chính sách **Weighted Routing**, gán trọng số 80 trỏ tới ALB phiên bản A và trọng số 20 trỏ tới ALB phiên bản B.
- **C.** Sử dụng Route 53 **Failover Routing Policy** với Primary trỏ tới ALB A và Secondary trỏ tới ALB B.
- **D.** Tạo một bản ghi Simple Routing cho `example.com` chứa đồng thời địa chỉ IP của cả hai ALB.

---

### CÂU 5 (Mức độ: Trung bình - Web Session Persistence & Zero Code Change)
Một website bán hàng trực tuyến đang chạy trên một nhóm các EC2 instances trong một Auto Scaling Group nằm sau một Application Load Balancer. Hiện tại, ứng dụng đang lưu trữ thông tin giỏ hàng và phiên đăng nhập của người dùng trực tiếp trong **bộ nhớ RAM cục bộ (in-memory session)** của từng instance. Khách hàng phàn nàn rằng khi họ duyệt qua các danh mục hàng hóa khác nhau, các mặt hàng vừa thêm vào giỏ hàng thường xuyên bị biến mất một cách ngẫu nhiên. Nhóm phát triển đang chuẩn bị kế hoạch chuyển dữ liệu session sang lưu tại Amazon ElastiCache Redis, nhưng cần một giải pháp khắc phục ngay lập tức mà **không cần chỉnh sửa mã nguồn ứng dụng (Zero Code Change)**.

Solutions Architect nên khuyến nghị giải pháp tạm thời nào?

- **A.** Thay thế Application Load Balancer bằng Network Load Balancer để tận dụng tính năng giữ nguyên Client IP gốc.
- **B.** Kích hoạt tính năng **Sticky Sessions (Session Affinity)** trên Target Group của Application Load Balancer với thời gian hết hạn cookie phù hợp.
- **C.** Tăng thời gian Cooldown Period của Auto Scaling Group lên 3600 giây để ngăn các instance bị gỡ bỏ.
- **D.** Chuyển đổi toàn bộ các máy chủ EC2 sang sử dụng loại instance có hỗ trợ EBS Multi-Attach để chia sẻ chung bộ nhớ RAM.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **B**
- **Tại sao đúng**:
  - Đề bài đưa ra yêu cầu cốt lõi: Khách hàng cần **địa chỉ IPv4 tĩnh cố định (Fixed Static IPs)** để cấu hình trên tường lửa doanh nghiệp (Firewall Whitelisting).
  - **Network Load Balancer (NLB)** hoạt động ở Layer 4, có đặc tính kỹ thuật quan trọng là cung cấp một địa chỉ IP tĩnh duy nhất cho mỗi Availability Zone (và cho phép người dùng gán trực tiếp **Elastic IP cố định** của mình vào từng AZ).
- **Phân tích bẫy**:
  - *Phương án A*: **Application Load Balancer (ALB) KHÔNG THỂ gán Elastic IP**. ALB chỉ cung cấp một tên miền DNS (ví dụ: `my-alb-1234.us-east-1.elb.amazonaws.com`) và các địa chỉ IP thực tế của các node ALB sẽ liên tục thay đổi linh hoạt theo nhu cầu co giãn của AWS.
  - *Phương án C*: CNAME vẫn là định tuyến tên miền, khách hàng đã nêu rõ là tường lửa của họ từ chối định tuyến DNS và bắt buộc cần danh sách IP tĩnh.
  - *Phương án D*: Gateway Load Balancer dùng cho các thiết bị ảo bảo mật (Virtual Appliances), không phải giải pháp tiếp nhận traffic từ Internet công cộng của khách hàng.

---

### Đáp án Câu 2: **C**
- **Tại sao đúng**:
  - Mặc định, Auto Scaling Group chỉ sử dụng **EC2 Status Checks**. Cơ chế này chỉ kiểm tra tính toàn vẹn của phần cứng (Hardware) và hệ điều hành (OS). Khi tiến trình web server (Nginx/NodeJS) bị treo hoặc sinh mã lỗi 502, hệ điều hành vẫn đang chạy nên EC2 status checks vẫn báo `OK / Healthy`.
  - Trong khi đó, Target Group của **Application Load Balancer (ALB)** thực hiện HTTP Health Check định kỳ tới endpoint của ứng dụng (ví dụ: `GET /health`). Khi web server crash, ALB lập tức phát hiện target bị `Unhealthy` và ngừng điều phối traffic đến đó.
  - Bằng cách đổi **Health Check Type của ASG sang "ELB"**, ASG sẽ lắng nghe trạng thái sức khỏe từ ALB. Khi ALB báo một instance Unhealthy, ASG sẽ lập tức **tự động terminate instance lỗi và khởi tạo instance mới để thay thế**, khôi phục tính sẵn sàng của ứng dụng hoàn toàn tự động.
- **Phân tích bẫy**:
  - *Phương án A*: Tự viết script và cấu hình cron job gây tốn công sức bảo trì, không chuẩn theo AWS Well-Architected Framework và không tận dụng được cơ chế có sẵn của ASG.
  - *Phương án B*: Khi web server bị treo, CPU có thể ở mức bình thường hoặc bất kỳ giá trị nào, việc dùng ngưỡng CPU < 5% không phản ánh đúng lỗi ứng dụng.
  - *Phương án D*: Lifecycle Hook dùng để tạm dừng tiến trình xóa/bật instance nhằm thực hiện tác vụ tùy biến, không giải quyết việc phát hiện lỗi crash của web server.

---

### Đáp án Câu 3: **C**
- **Tại sao đúng**:
  - Các từ khóa kỹ thuật then chốt: **Giao thức UDP**, **độ trễ siêu thấp**, **IP tĩnh toàn cầu**, và **tự động failover giữa các Region trong dưới 1 phút**.
  - **AWS Global Accelerator** là dịch vụ hoàn hảo đáp ứng trọn vẹn:
    1. Cung cấp **2 địa chỉ Static Anycast IPv4** đóng vai trò là điểm tiếp nhận duy nhất trên toàn cầu (không bao giờ thay đổi).
    2. Hỗ trợ đầy đủ **cả TCP và UDP** (trong khi CloudFront không hỗ trợ UDP).
    3. Lưu lượng từ client lập tức đi vào các Edge Location của AWS và được dẫn truyền trên mạng cáp quang riêng của AWS (Global Private Backbone Network) tới thẳng Region đích, loại bỏ tình trạng mất gói và jitter của mạng Internet công cộng.
    4. Tích hợp liên tục với Health Checks ở từng Region, tự động chuyển hướng traffic sang Region còn lại trong vòng **dưới 30 - 60 giây**.
- **Phân tích bẫy**:
  - *Phương án A*: **Amazon CloudFront KHÔNG hỗ trợ giao thức UDP** (CloudFront chỉ hỗ trợ HTTP, HTTPS và WebSockets).
  - *Phương án B & D*: Route 53 phụ thuộc vào thời gian lưu bộ nhớ đệm DNS (**DNS TTL**) của các nhà mạng Internet (ISP). Khi Region gặp sự cố, nhiều client vẫn giữ DNS cache cũ trong vài phút đến vài giờ, không thể đạt được thời gian chuyển hướng dưới 1 phút. Ngoài ra ALB trong phương án D cũng không hỗ trợ UDP.

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**:
  - Câu hỏi kiểm tra 2 kiến thức trọng tâm:
    1. **Quy tắc DNS cho Zone Apex (Root Domain)**: Theo chuẩn kỹ thuật DNS quốc tế (RFC 1034), **không thể tạo bản ghi CNAME cho tên miền gốc (Zone Apex)** như `example.com`. Route 53 giải quyết vấn đề này bằng bản ghi **ALIAS** (bản quyền của AWS), cho phép tạo bản ghi kiểu **A record với ALIAS = Yes** trỏ thẳng tới các tài nguyên AWS như Application Load Balancer.
    2. **Chính sách phân chia lưu lượng**: **Weighted Routing Policy** cho phép cấu hình phân bổ lưu lượng theo tỷ lệ phần trăm (ở đây là trọng số 80 cho ALB A và 20 cho ALB B).
- **Phân tích bẫy**:
  - *Phương án A*: Bẫy kinh điển! Tạo bản ghi CNAME cho `example.com` là sai quy chuẩn DNS và Route 53 sẽ báo lỗi không cho phép tạo CNAME trên Zone Apex.
  - *Phương án C*: Failover Routing chỉ dùng cho mô hình Active-Passive (100% traffic đổ vào Primary, chỉ chuyển sang Secondary khi Primary chết hoàn toàn), không thể chia tỷ lệ 80/20.
  - *Phương án D*: Simple Routing trả về danh sách IP ngẫu nhiên, không thể kiểm soát tỷ lệ 80% và 20%.

---

### Đáp án Câu 5: **B**
- **Tại sao đúng**:
  - Nguyên nhân: Do ứng dụng lưu trữ session trong RAM của từng EC2 instance cục bộ, khi người dùng gửi request tiếp theo, nếu ALB chuyển request đó sang một EC2 instance khác thì instance mới này không có dữ liệu session trước đó, khiến giỏ hàng bị mất.
  - Giải pháp tức thì không sửa code: Bật tính năng **Sticky Sessions (Session Affinity)** trên Target Group của Application Load Balancer. ALB sẽ phát hành một cookie (mặc định tên là `AWSALB`) trả về cho trình duyệt. Các request tiếp theo từ trình duyệt này sẽ luôn được ALB định tuyến về đúng EC2 instance ban đầu, đảm bảo session trong RAM được duy trì.
- **Phân tích bẫy**:
  - *Phương án A*: NLB giữ Source IP nhưng không duy trì phiên làm việc ứng dụng ở tầng HTTP cookie, và client có thể đổi IP mạng (ví dụ chuyển từ Wifi sang 4G).
  - *Phương án C*: Tăng Cooldown Period chỉ hoãn việc tắt mở instance của ASG, hoàn toàn không giải quyết việc ALB cân bằng tải request qua lại giữa các máy đang chạy.
  - *Phương án D*: EBS Multi-Attach là công nghệ lưu trữ block storage trên ổ cứng trong cùng 1 AZ (cho cluster file system), hoàn toàn không thể và không dùng để chia sẻ bộ nhớ RAM giữa các máy chủ.
