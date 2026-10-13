# BỘ ĐỀ THI TỔNG HỢP 40 CÂU TUẦN 1: CORE INFRASTRUCTURE
*(Bao quát toàn diện: Networking, Compute, Storage, Database, Routing, High Availability & Cost Optimization)*

---

### PHẦN 1: NETWORKING & HYBRID CLOUD (CÂU 1 - 8)

#### CÂU 1
Một công ty cần thiết kế một VPC mới cho hệ thống microservices. Dự kiến hệ thống có 4 subnets trải đều trên 2 Availability Zones. Mỗi subnet cần chứa tối thiểu 120 EC2 instances. Cấu hình CIDR block nào sau đây cho mỗi Subnet là nhỏ nhất có thể đáp ứng yêu cầu?
- **A.** `/26`
- **B.** `/25`
- **C.** `/24`
- **D.** `/23`

#### CÂU 2
Một ứng dụng web chạy trên EC2 trong một Private Subnet cần tải các gói cập nhật từ Internet công cộng. Giải pháp nào sau đây vừa đảm bảo tính bảo mật vừa có **chi phí vận hành tối thiểu (least operational overhead)**?
- **A.** Đặt EC2 trong Public Subnet và gán Elastic IP.
- **B.** Triển khai một NAT Gateway trong Public Subnet và cấu hình Route Table của Private Subnet trỏ `0.0.0.0/0` về NAT Gateway.
- **C.** Triển khai một EC2 instance chạy hệ điều hành Amazon Linux cấu hình iptables làm NAT Instance.
- **D.** Tạo một Internet Gateway thứ hai gắn trực tiếp vào Private Subnet.

#### CÂU 3
Bộ phận an ninh phát hiện một dải địa chỉ IP (`198.51.100.0/24`) đang thực hiện quét cổng (port scanning) liên tục vào cụm máy chủ web EC2. Hành động nào sau đây sẽ chặn dải IP này với tốc độ nhanh nhất và hiệu quả nhất?
- **A.** Tạo Inbound Deny rule trên Security Group của các EC2 instances.
- **B.** Tạo Outbound Deny rule trên Security Group của các EC2 instances.
- **C.** Tạo Inbound Deny rule cho `198.51.100.0/24` trên Network ACL (NACL) của subnet với số thứ tự rule nhỏ hơn rule Allow.
- **D.** Gỡ bỏ Internet Gateway khỏi VPC.

#### CÂU 4
Một công ty có máy chủ on-premises kết nối tới AWS qua **AWS Direct Connect**. Ứng dụng on-premises cần ghi dữ liệu dung lượng lớn trực tiếp vào một Amazon S3 bucket mà không đi qua Internet công cộng. Giải pháp VPC Endpoint nào sau đây là bắt buộc?
- **A.** S3 Gateway Endpoint
- **B.** S3 Interface Endpoint (AWS PrivateLink)
- **C.** Network Load Balancer kết hợp S3 Gateway Endpoint
- **D.** Internet Gateway kết hợp NAT Gateway

#### CÂU 5
Một doanh nghiệp quản lý hơn 50 VPCs và một kết nối AWS Direct Connect tới trung tâm dữ liệu on-premises. Doanh nghiệp cần kết nối toàn bộ các VPCs này với nhau và với on-premises theo mô hình tập trung, hỗ trợ phân đoạn mạng (Dev vs Prod). Giải pháp nào tối ưu nhất?
- **A.** Thiết lập mạng lưới Full-Mesh VPC Peering giữa tất cả các VPC.
- **B.** Triển khai **AWS Transit Gateway (TGW)** và gắn (attach) các VPC cùng kết nối Direct Connect vào Transit Gateway.
- **C.** Sử dụng AWS Site-to-Site VPN tạo full-mesh kết nối giữa các VPC.
- **D.** Triển khai một cụm EC2 proxy Nginx trong một VPC trung tâm.

#### CÂU 6
Một ngân hàng yêu cầu kết nối chuyên dụng có độ trễ cực thấp giữa on-premises và AWS VPC. Đồng thời, toàn bộ dữ liệu truyền trên kết nối này **bắt buộc phải được mã hóa IPsec**. Giải pháp nào sau đây đáp ứng được yêu cầu?
- **A.** Chỉ sử dụng đường truyền vật lý AWS Direct Connect tiêu chuẩn.
- **B.** Thiết lập **AWS Site-to-Site VPN chạy trên nền kết nối AWS Direct Connect**.
- **C.** Sử dụng AWS Snowball Edge để truyền dữ liệu định kỳ.
- **D.** Kích hoạt tính năng VPC Flow Logs để mã hóa gói tin mạng.

#### CÂU 7
Một kỹ sư tạo một custom Network ACL cho Public Subnet. Inbound mở cổng 80 và 443 từ `0.0.0.0/0`. Outbound để mặc định là Deny All. Người dùng từ Internet truy cập website thì bị lỗi Connection Timed Out. Nguyên nhân là gì?
- **A.** Security Group của EC2 đang chặn cổng 80.
- **B.** Network ACL là stateless, thiếu Outbound rule cho phép **Ephemeral Ports (TCP 1024-65535)** gửi phản hồi về client.
- **C.** Cần phải gắn Elastic IP vào Network ACL.
- **D.** Subnet chưa được gắn Internet Gateway.

#### CÂU 8
Ứng dụng trong VPC A muốn truy cập tài nguyên trong VPC B qua VPC Peering. Tuy nhiên, VPC A có dải CIDR là `10.0.0.0/16` và VPC B cũng có dải CIDR là `10.0.0.0/16`. Điều gì sẽ xảy ra khi thiết lập VPC Peering?
- **A.** AWS tự động dịch NAT giữa hai VPC.
- **B.** VPC Peering sẽ được thiết lập thành công nhưng chỉ hỗ trợ giao thức UDP.
- **C.** AWS sẽ từ chối tạo kết nối VPC Peering vì hai VPC có dải địa chỉ IP trùng lặp (**Overlapping CIDR**).
- **D.** Chỉ có các instance có IP lẻ mới kết nối được với nhau.

---

### PHẦN 2: COMPUTE & STORAGE (CÂU 9 - 16)

#### CÂU 9
Một công ty cần chạy một tác vụ phân tích gen sinh học quy mô lớn kéo dài trong 6 giờ vào cuối tuần. Tác vụ có thể chia nhỏ thành hàng ngàn tiến trình độc lập, dữ liệu được checkpoint liên tục lên S3, nếu một máy tính bị dừng đột ngột thì tiến trình đó có thể chạy lại từ checkpoint gần nhất. Mô hình EC2 nào mang lại **chi phí thấp nhất**?
- **A.** On-Demand Instances
- **B.** Standard Reserved Instances (1 năm)
- **C.** **Spot Instances**
- **D.** Dedicated Hosts

#### CÂU 10
Một ứng dụng doanh nghiệp chạy cơ sở dữ liệu Oracle đòi hỏi giấy phép phần mềm tính tiền nghiêm ngặt dựa trên **số lượng socket và core CPU vật lý**. Đồng thời, doanh nghiệp phải đảm bảo không chia sẻ phần cứng máy chủ với bất kỳ ai. Giải pháp EC2 nào là bắt buộc?
- **A.** EC2 Dedicated Instances
- **B.** **EC2 Dedicated Hosts**
- **C.** EC2 Reserved Instances
- **D.** EC2 Spot Fleet

#### CÂU 11
Một cụm tính toán hiệu năng cao (HPC) gồm nhiều EC2 instances yêu cầu giao tiếp mạng với nhau ở độ trễ mạng thấp nhất có thể (sub-millisecond) và băng thông tối đa (lên tới 100 Gbps). Cấu hình Placement Group nào nên được sử dụng?
- **A.** **Cluster Placement Group** trong 1 AZ duy nhất
- **B.** Spread Placement Group trải trên 3 AZs
- **C.** Partition Placement Group
- **D.** Auto Placement Group

#### CÂU 12
Để ngăn chặn các lỗ hổng bảo mật dạng SSRF (Server-Side Request Forgery) cố tình đánh cắp thông tin xác thực IAM Role tạm thời từ dịch vụ EC2 Instance Metadata Service, Solutions Architect nên khuyến nghị điều gì?
- **A.** Tắt hoàn toàn card mạng ENI của instance.
- **B.** Bắt buộc sử dụng **IMDSv2** bằng cách thiết lập tham số `HttpTokens=required`.
- **C.** Đổi địa chỉ IP `169.254.169.254` sang một địa chỉ IP công cộng khác.
- **D.** Sử dụng Security Group chặn toàn bộ traffic gửi tới port 80.

#### CÂU 13
Một hệ sinh thái máy chủ web chạy trên Linux trải đều trên **3 Availability Zones** cần đọc và ghi đồng thời vào một thư mục chia sẻ tệp tin chung tương thích chuẩn POSIX. Dịch vụ lưu trữ nào đáp ứng hoàn hảo yêu cầu này?
- **A.** Amazon EBS với tính năng Multi-Attach
- **B.** **Amazon EFS** (Elastic File System)
- **C.** Amazon FSx for Windows File Server
- **D.** Amazon S3 gắn qua giao thức iSCSI

#### CÂU 14
Một ứng dụng đồ họa 3D thời gian thực cần ghi hàng trăm gigabyte dữ liệu tạm thời (scratch data) với tốc độ I/O cực cao hàng trăm ngàn IOPS và độ trễ vi-giây. Dữ liệu này không cần lưu lại khi tắt máy. Giải pháp lưu trữ nào có chi phí tối ưu nhất và hiệu năng cao nhất?
- **A.** Amazon EBS loại `io2 Block Express`
- **B.** **EC2 Instance Store** (NVMe SSD gắn vật lý)
- **C.** Amazon EFS ở chế độ Provisioned Throughput
- **D.** Amazon S3 Standard kết hợp S3 Transfer Acceleration

#### CÂU 15
Một cơ sở dữ liệu quan trọng yêu cầu hiệu năng lưu trữ block storage đạt mức ổn định **80,000 IOPS** với độ trễ dưới 1 mili-giây. Loại EBS volume nào sau đây là lựa chọn duy nhất đáp ứng được chỉ số này?
- **A.** General Purpose SSD (`gp2`)
- **B.** General Purpose SSD (`gp3`)
- **C.** **Provisioned IOPS SSD (`io2` / `io2 Block Express`)**
- **D.** Throughput Optimized HDD (`st1`)

#### CÂU 16
Một công ty muốn chia sẻ một bản sao lưu EBS Snapshot đã được mã hóa sang một tài khoản AWS của đối tác kiểm toán độc lập. Điều kiện kỹ thuật bắt buộc để có thể chia sẻ được Snapshot này là gì?
- **A.** Snapshot phải được mã hóa bằng khóa mặc định của AWS (`aws/ebs`).
- **B.** Snapshot phải được mã hóa bằng **Customer Managed Key (CMK)** và cấp quyền cho tài khoản đối tác trong KMS Key Policy.
- **C.** Snapshot phải được chuyển sang định dạng ZIP rồi upload lên S3 công khai.
- **D.** Tài khoản đối tác phải có quyền Root trên tài khoản nguồn.

---

### PHẦN 3: HIGH AVAILABILITY, LOAD BALANCING & ROUTING (CÂU 17 - 24)

#### CÂU 17
Một hệ thống microservices yêu cầu định tuyến lưu lượng truy cập dựa trên đường dẫn URL (Path-based: `/orders`, `/users`), hỗ trợ HTTPS và tích hợp trực tiếp với AWS WAF. Loại Load Balancer nào phù hợp nhất?
- **A.** Network Load Balancer (NLB)
- **B.** **Application Load Balancer (ALB)**
- **C.** Gateway Load Balancer (GLB)
- **D.** Classic Load Balancer (CLB)

#### CÂU 18
Một ứng dụng tài chính yêu cầu định tuyến HTTPS theo URL path nhưng các chi nhánh ngân hàng đối tác bắt buộc phải có **2 địa chỉ Static Public IP cố định** để whitelist tường lửa. Giải pháp kiến trúc nào tối ưu nhất?
- **A.** Gán 2 Elastic IP trực tiếp vào cấu hình của Application Load Balancer.
- **B.** Triển khai **AWS Global Accelerator** cung cấp 2 Static Anycast IPs đứng phía trước một Application Load Balancer.
- **C.** Sử dụng Network Load Balancer chuyển tiếp traffic dựa trên URL path.
- **D.** Sử dụng Route 53 Weighted Routing với các bản ghi CNAME.

#### CÂU 19
Một ứng dụng game bắn súng thời gian thực sử dụng giao thức **UDP**, yêu cầu độ trễ mạng thấp nhất quán trên toàn cầu và khả năng failover giữa hai Region Singapore và Frankfurt trong vòng **dưới 1 phút**. Giải pháp tối ưu là gì?
- **A.** Amazon CloudFront Distribution với tính năng Origin Shield.
- **B.** **AWS Global Accelerator** kết hợp với Endpoint Groups tại hai Region.
- **C.** Amazon Route 53 Geolocation Routing.
- **D.** Application Load Balancer với giao thức gRPC.

#### CÂU 20
Một công ty muốn người dùng khi gõ tên miền gốc (Zone Apex: `example.com` - không có www) sẽ được trỏ trực tiếp đến Application Load Balancer mà không tốn chi phí truy vấn DNS. Cấu hình Route 53 nào đúng?
- **A.** Bản ghi CNAME trỏ tới DNS name của ALB.
- **B.** Bản ghi **A (ALIAS)** trỏ trực tiếp tới Application Load Balancer.
- **C.** Bản ghi MX trỏ tới IP của ALB.
- **D.** Bản ghi PTR trỏ ngược về tên miền.

#### CÂU 21
Để tuân thủ luật bảo vệ dữ liệu người dùng tại Châu Âu (GDPR), một công ty yêu cầu toàn bộ traffic từ các nước thuộc Liên minh Châu Âu phải được chuyển tới cụm máy chủ tại Ireland, còn các khu vực khác chuyển tới Virginia. Chính sách Route 53 nào nên dùng?
- **A.** Latency-based Routing
- **B.** **Geolocation Routing**
- **C.** Geoproximity Routing
- **D.** Multi-Value Answer Routing

#### CÂU 22
Trong một đợt scale-in (giảm tải), Auto Scaling Group chuẩn bị tắt một EC2 instance. Quản trị viên cần đảm bảo instance này có đủ thời gian để tải các tệp log chưa kịp đồng bộ lên Amazon S3 trước khi bị xóa vĩnh viễn. Tính năng nào của ASG hỗ trợ việc này?
- **A.** Target Tracking Scaling Policy
- **B.** **ASG Lifecycle Hook (`EC2_INSTANCE_TERMINATING`)**
- **C.** Cooldown Period
- **D.** Instance Warmup

#### CÂU 23
Một công ty muốn kiểm tra phiên bản mới của website bằng cách điều hướng **10% lượng người dùng** vào phiên bản mới (Canary deployment) và 90% người dùng vào phiên bản hiện tại. Chính sách định tuyến Route 53 nào đáp ứng yêu cầu?
- **A.** Failover Routing
- **B.** **Weighted Routing** (Tỷ trọng 10/90)
- **C.** Latency Routing
- **D.** Simple Routing

#### CÂU 24
Sự khác biệt cốt lõi giữa Amazon CloudFront và AWS Global Accelerator là gì?
- **A.** CloudFront không hỗ trợ HTTPS, Global Accelerator hỗ trợ HTTPS.
- **B.** **CloudFront có Caching nội dung tại Edge**, còn **Global Accelerator không Caching** mà gia tốc gói tin TCP/UDP qua mạng trục riêng của AWS.
- **C.** Global Accelerator chỉ dùng được cho S3.
- **D.** CloudFront cung cấp 2 địa chỉ Static IP tĩnh, Global Accelerator cung cấp DNS name động.

---

### PHẦN 4: AMAZON S3 & DATA PROTECTION (CÂU 25 - 32)

#### CÂU 25
Một công ty có 100 TB dữ liệu hồ sơ y tế có tần suất truy cập thay đổi thất thường và không thể dự đoán trước. Công ty muốn tối ưu hóa chi phí lưu trữ tối đa nhưng tuyệt đối **không được làm chậm thời gian truy xuất khi cần và không tốn phí truy xuất dữ liệu (no retrieval fee)**. Storage class nào tối ưu nhất?
- **A.** S3 Standard-IA
- **B.** **S3 Intelligent-Tiering**
- **C.** S3 Glacier Flexible Retrieval
- **D.** S3 One Zone-IA

#### CÂU 26
Theo quy định pháp lý ngành chứng khoán (SEC Rule 17a-4), dữ liệu giao dịch phải được lưu trữ dạng WORM (Write Once, Read Many) trong 7 năm và **ngay cả tài khoản Root Account cũng không được phép xóa hoặc sửa đổi** trong thời gian này. Tính năng nào đáp ứng?
- **A.** S3 Versioning kết hợp MFA Delete
- **B.** **S3 Object Lock ở chế độ Compliance Mode**
- **C.** S3 Object Lock ở chế độ Governance Mode
- **D.** S3 Bucket Policy với quyền Read-Only

#### CÂU 27
Một công ty muốn đảm bảo toàn bộ dữ liệu truyền tải giữa client và S3 bucket **bắt buộc phải qua giao thức mã hóa HTTPS/TLS**, mọi kết nối HTTP không an toàn phải bị chặn. Điều kiện Condition nào trong Bucket Policy thực hiện được việc này?
- **A.** `"Effect": "Deny"`, Condition: `{"Bool": {"aws:SecureTransport": "false"}}`
- **B.** `"Effect": "Allow"`, Condition: `{"Bool": {"aws:SecureTransport": "false"}}`
- **C.** `"Effect": "Deny"`, Condition: `{"StringEquals": {"s3:x-amz-acl": "public-read"}}`
- **D.** Bật tính năng SSE-S3 trên bucket.

#### CÂU 28
Ứng dụng di động cần cho phép người dùng tải trực tiếp video lên một S3 bucket riêng tư mà không cần tạo IAM user hay nhúng Access Key vào ứng dụng. Giải pháp nào an toàn và chuẩn kiến trúc nhất?
- **A.** Mở quyền public write cho toàn bộ S3 bucket.
- **B.** Backend server xác thực người dùng rồi sinh ra một **S3 Presigned URL** có thời hạn ngắn để ứng dụng upload file.
- **C.** Tạo một IAM Role công khai không cần mật khẩu.
- **D.** Lưu trữ video tạm thời trên GitHub rồi đồng bộ sang S3.

#### CÂU 29
Khi kích hoạt tính năng Cross-Region Replication (CRR) trên một S3 bucket có sẵn 20 TB dữ liệu cũ, các file mới upload đều được replicate sang bucket đích, nhưng **toàn bộ 20 TB dữ liệu cũ không xuất hiện ở bucket đích**. Giải pháp nào giải quyết vấn đề với nỗ lực tối thiểu?
- **A.** Sử dụng tính năng **S3 Batch Replication** để thực hiện replicate một lần toàn bộ các đối tượng có sẵn.
- **B.** Tắt và bật lại S3 Versioning trên bucket nguồn.
- **C.** Dùng script Python tải về máy local rồi upload sang bucket đích.
- **D.** Yêu cầu AWS Support can thiệp thủ công.

#### CÂU 30
Để lưu trữ các bản ghi log hệ thống trong thời gian 10 năm với **chi phí lưu trữ rẻ nhất có thể trên toàn bộ nền tảng AWS**, và thời gian chờ lấy lại dữ liệu có thể chấp nhận từ 12 đến 48 giờ. Storage class nào nên được chọn?
- **A.** S3 Standard-IA
- **B.** S3 Glacier Flexible Retrieval
- **C.** **S3 Glacier Deep Archive**
- **D.** S3 One Zone-IA

#### CÂU 31
Một hệ thống xử lý hàng trăm ngàn lượt đọc file mỗi giây từ S3 bucket được mã hóa bằng **SSE-KMS**. Hệ thống bắt đầu gặp lỗi HTTP 429 (KMS ThrottlingException). Giải pháp nào khắc phục lỗi này mà không cần nâng hạn mức KMS quota?
- **A.** Chuyển toàn bộ dữ liệu sang SSE-C.
- **B.** Bật tính năng **S3 Bucket Keys** trên bucket để giảm tới 99% số lượng request gọi sang AWS KMS.
- **C.** Tạo 10 khóa KMS khác nhau và xoay vòng ngẫu nhiên.
- **D.** Chuyển sang dùng Amazon EBS.

#### CÂU 32
Điều kiện tiên quyết bắt buộc để có thể kích hoạt tính năng S3 Cross-Region Replication (CRR) giữa hai bucket là gì?
- **A.** Cả hai bucket phải nằm trong cùng một AWS Account.
- **B.** **Cả bucket nguồn và bucket đích bắt buộc phải BẬT tính năng Versioning**.
- **C.** Phải có đường truyền AWS Direct Connect giữa 2 Region.
- **D.** Cả hai bucket phải được cấu hình Static Website Hosting.

---

### PHẦN 5: DATABASE SOLUTIONS (CÂU 33 - 40)

#### CÂU 33
Một cơ sở dữ liệu Amazon RDS PostgreSQL bị quá tải CPU nghiêm trọng do 90% lưu lượng truy cập là các câu lệnh đọc báo cáo (`SELECT`). Giải pháp nào tối ưu nhất để giảm tải cho database chính?
- **A.** Kích hoạt tính năng RDS Multi-AZ Deployment.
- **B.** Tạo các **RDS Read Replicas** và cấu hình ứng dụng trỏ các câu truy vấn đọc vào Endpoint của Read Replicas.
- **C.** Bật tính năng Multi-Region VPC Peering.
- **D.** Chuyển đổi storage từ gp3 sang st1.

#### CÂU 34
Sự khác biệt cốt lõi về mặt chức năng giữa RDS Multi-AZ Standby và RDS Read Replica là gì?
- **A.** Multi-AZ phục vụ đọc; Read Replica phục vụ ghi.
- **B.** **Multi-AZ phục vụ High Availability (Standby thụ động KHÔNG thể đọc)**; **Read Replica phục vụ mở rộng khả năng ĐỌC**.
- **C.** Read Replica sao chép đồng bộ; Multi-AZ sao chép bất đồng bộ.
- **D.** Multi-AZ hỗ trợ đa vùng (cross-region); Read Replica chỉ hỗ trợ trong 1 AZ.

#### CÂU 35
Một ứng dụng yêu cầu cơ sở dữ liệu quan hệ MySQL có khả năng tự động chịu lỗi (khôi phục sau thảm họa) đa vùng (Disaster Recovery) với chỉ số **RPO < 1 giây** và **RTO < 1 phút**. Giải pháp nào là tốt nhất?
- **A.** RDS MySQL với Cross-Region Read Replica.
- **B.** **Amazon Aurora Global Database**.
- **C.** Tự động snapshot RDS mỗi ngày và copy sang Region khác.
- **D.** DynamoDB với tính năng Streams.

#### CÂU 36
Một cụm cơ sở dữ liệu Amazon Aurora có 1 Primary Writer và 3 Aurora Replicas. Nhóm phát triển muốn ứng dụng tự động cân bằng tải các câu truy vấn đọc (Read queries) trải đều qua 3 Replicas này mà không cần quản lý danh sách IP. Nhóm nên dùng endpoint nào?
- **A.** Cluster Endpoint
- **B.** **Reader Endpoint**
- **C.** Instance Endpoint của Writer
- **D.** Gateway Endpoint

#### CÂU 37
Một nền tảng thương mại điện tử cần giảm độ trễ đọc của các mặt hàng "hot sale" trên Amazon DynamoDB từ mức mili-giây xuống mức **vi-giây (microsecond latency)** mà **không cần viết lại logic code kiểm tra cache của ứng dụng**. Giải pháp tối ưu là gì?
- **A.** Triển khai **Amazon DynamoDB Accelerator (DAX)**.
- **B.** Triển khai Amazon ElastiCache for Memcached.
- **C.** Tăng gấp đôi Read Capacity Units (RCU).
- **D.** Bật tính năng DynamoDB Global Tables.

#### CÂU 38
Một ứng dụng yêu cầu cơ sở dữ liệu NoSQL phân tán toàn cầu cho phép người dùng tại cả Tokyo và London đều có thể **đọc và ghi dữ liệu tại chỗ với độ trễ thấp (Active-Active Multi-Region read and write)**. Dịch vụ nào đáp ứng hoàn hảo?
- **A.** Amazon RDS Multi-AZ với Read Replicas.
- **B.** **Amazon DynamoDB Global Tables**.
- **C.** Amazon Aurora Serverless v2.
- **D.** Amazon Redshift.

#### CÂU 39
Một công ty trò chơi trực tuyến cần xây dựng tính năng **Bảng xếp hạng thời gian thực (Real-time Leaderboard)** xếp hạng hàng triệu người chơi theo điểm số cao nhất, yêu cầu cấu hình **Multi-AZ Auto-Failover** và hỗ trợ kiểu dữ liệu **Sorted Sets**. Dịch vụ nào tối ưu nhất?
- **A.** Amazon ElastiCache for Memcached
- **B.** **Amazon ElastiCache for Redis**
- **C.** Amazon RDS SQL Server
- **D.** Amazon S3 Glacier

#### CÂU 40
Một trang web muốn tự động xóa các sản phẩm trong giỏ hàng tạm thời của khách hàng sau 7 ngày không hoạt động mà **không phải tốn chi phí Write Capacity Units (WCU)** và không cần chạy batch job quét database định kỳ. Tính năng nào của DynamoDB thực hiện được việc này?
- **A.** DynamoDB Streams
- **B.** **DynamoDB Time to Live (TTL)**
- **C.** DynamoDB Transactions
- **D.** DynamoDB Point-in-Time Recovery (PITR)

---

---

## BẢNG ĐÁP ÁN NHANH & TỔNG KẾT MA TRẬN ĐIỂM

| Câu | Đáp án | Chủ đề cốt lõi | Câu | Đáp án | Chủ đề cốt lõi |
| :---: | :---: | :--- | :---: | :---: | :--- |
| **1** | **B** | Subnet Sizing (/25 = 123 usable IPs cho 120 EC2) | **21** | **B** | Route 53 Geolocation (Tuân thủ luật pháp GDPR) |
| **2** | **B** | NAT Gateway trong Public Subnet (Egress private) | **22** | **B** | ASG Lifecycle Hook (`EC2_INSTANCE_TERMINATING`) |
| **3** | **C** | Network ACL Deny rule (SG không có Deny) | **23** | **B** | Route 53 Weighted Routing (Canary deployment) |
| **4** | **B** | S3 Interface Endpoint (PrivateLink từ on-premises qua DX)| **24** | **B** | CloudFront (CDN Caching) vs Global Accelerator (Layer 4) |
| **5** | **B** | AWS Transit Gateway (Hub-and-Spoke 50+ VPCs) | **25** | **B** | S3 Intelligent-Tiering (Truy cập không đoán được, no fee) |
| **6** | **B** | VPN over Direct Connect (Mã hóa IPsec trên đường vật lý)| **26** | **B** | S3 Object Lock Compliance Mode (WORM không ai xóa được) |
| **7** | **B** | NACL Stateless - Cần mở Ephemeral Ports (1024-65535) | **27** | **A** | Bucket Policy Deny `aws:SecureTransport: false` (HTTPS) |
| **8** | **C** | Overlapping CIDR không thể Peering | **28** | **B** | S3 Presigned URLs (Upload an toàn không cần IAM) |
| **9** | **C** | Spot Instances (Batch checkpointable rẻ nhất 90%) | **29** | **A** | S3 Batch Replication (Replicate dữ liệu cũ có sẵn) |
| **10** | **B** | EC2 Dedicated Hosts (Kiểm soát socket/core vật lý BYOL)| **30** | **C** | S3 Glacier Deep Archive (Rẻ nhất toàn bộ AWS) |
| **11** | **A** | Cluster Placement Group (Sub-millisecond latency 100Gbps)| **31** | **B** | S3 Bucket Keys (Giảm 99% request gọi sang KMS) |
| **12** | **B** | Enforce IMDSv2 (`HttpTokens=required` chống SSRF) | **32** | **B** | S3 Versioning bắt buộc bật cả 2 bucket khi dùng CRR |
| **13** | **B** | Amazon EFS (Multi-AZ Linux POSIX chia sẻ đồng thời) | **33** | **B** | RDS Read Replicas (Giảm tải CPU 90% truy vấn đọc SELECT)|
| **14** | **B** | EC2 Instance Store (NVMe local tốc độ đỉnh cao, free) | **34** | **B** | Multi-AZ (HA Standby không đọc) vs Read Replica (Read) |
| **15** | **C** | EBS `io2 Block Express` (Đạt tới 256,000 IOPS) | **35** | **B** | Aurora Global Database (DR đa vùng RPO < 1s, RTO < 1m) |
| **16** | **B** | Share Snapshot mã hóa cần Customer Managed Key (CMK)| **36** | **B** | Aurora Reader Endpoint (Cân bằng tải đọc tự động) |
| **17** | **B** | Application Load Balancer (Layer 7 path-based routing) | **37** | **A** | DynamoDB DAX (Microsecond read latency, transparent) |
| **18** | **B** | Global Accelerator + ALB (2 Static Anycast IPs) | **38** | **B** | DynamoDB Global Tables (Active-Active multi-region R/W)|
| **19** | **B** | Global Accelerator (UDP gaming, failover đa vùng < 1m) | **39** | **B** | ElastiCache Redis (Sorted Sets cho Leaderboard, HA) |
| **20** | **B** | Route 53 A ALIAS Record (Trỏ Zone Apex miễn phí) | **40** | **B** | DynamoDB Time to Live (Tự động xóa data không tốn WCU) |
