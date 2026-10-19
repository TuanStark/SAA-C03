# BỘ ĐỀ THI TỔNG HỢP 50 CÂU TUẦN 2: SECURITY, SERVERLESS, INTEGRATION & DR
*(Bao quát toàn diện: IAM, Organizations, KMS, Secrets Manager, SQS, SNS, EventBridge, Kinesis, Lambda, API Gateway, Step Functions, CloudWatch, CloudTrail, Config, SSM & Disaster Recovery)*

---

### PHẦN 1: IAM, ORGANIZATIONS & PHÂN QUYỀN (CÂU 1 - 10)

#### CÂU 1
Một quản trị viên gắn một Service Control Policy (SCP) có điều kiện `Deny` cho hành động `ec2:RunInstances` vào một Organizational Unit (OU). Nhóm kỹ sư đăng nhập bằng tài khoản **Root User của một Member Account** nằm trong OU đó để tạo EC2 instance. Điều gì sẽ xảy ra?
- **A.** Lệnh tạo instance thành công vì Root User luôn có quyền lực tối cao vượt qua mọi SCP.
- **B.** Lệnh tạo instance bị từ chối vì **SCP có hiệu lực khống chế toàn bộ người dùng, bao gồm cả Root User của Member Account**.
- **C.** Lệnh tạo instance thành công nhưng một cảnh báo bảo mật được gửi tới Security Hub.
- **D.** SCP chỉ áp dụng nếu tài khoản đó đã bật AWS Shield Advanced.

#### CÂU 2
Một tổ chức muốn trao quyền cho các lập trình viên tự tạo IAM Role cho ứng dụng của họ, nhưng phải đảm bảo các lập trình viên này **không thể tự cấp quyền Administrator hoặc vượt quá mức trần an ninh cho phép (chống Privilege Escalation)**. Giải pháp nào là chuẩn nhất?
- **A.** Sử dụng **Permissions Boundaries** đính kèm điều kiện trong IAM Policy của lập trình viên.
- **B.** Sử dụng AWS Secrets Manager để xoay vòng quyền hạn.
- **C.** Chỉ cho phép lập trình viên tạo IAM User thay vì IAM Role.
- **D.** Cấu hình Amazon Inspector để quét các IAM Role hàng ngày.

#### CÂU 3
Khi tích hợp một phần mềm SaaS của bên thứ ba để tối ưu chi phí hạ tầng AWS thông qua quyền `sts:AssumeRole`, giải pháp nào bắt buộc phải có để ngăn chặn lỗ hổng **The Confused Deputy Problem**?
- **A.** Cung cấp chuỗi **External ID** do bên thứ ba cung cấp vào điều kiện `Condition` của Trust Policy.
- **B.** Gửi email chứa AWS Access Key và Secret Key của tài khoản Root cho bên thứ ba.
- **C.** Yêu cầu bên thứ ba chỉ gọi API trong giờ hành chính.
- **D.** Mở quyền truy cập công khai cho toàn bộ tài nguyên trong tài khoản.

#### CÂU 4
Ứng dụng trên EC2 ở Tài khoản A cần đọc dữ liệu từ S3 bucket ở Tài khoản B. Yêu cầu tuyệt đối **không được lưu trữ Access Key/Secret Key tĩnh trên máy chủ**. Giải pháp tối ưu là gì?
- **A.** Tạo một IAM Role ở Tài khoản B cho phép Tài khoản A assume. Gán một IAM Role (Instance Profile) cho EC2 ở Tài khoản A cho phép gọi **`sts:AssumeRole`** tới Role ở Tài khoản B.
- **B.** Mở quyền public đọc cho S3 bucket ở Tài khoản B.
- **C.** Dùng script Python gửi mật khẩu Root qua giao thức SSH.
- **D.** Tạo kết nối VPC Peering giữa hai tài khoản.

#### CÂU 5
Một doanh nghiệp muốn cấm tạo tài nguyên bên ngoài 2 Region được phê duyệt thông qua SCP của AWS Organizations. Tuy nhiên, khi áp dụng policy, người dùng bị mất quyền truy cập vào IAM và Route 53. Cách khắc phục chuẩn xác là gì?
- **A.** Sử dụng phần tử **`NotAction`** trong SCP để loại trừ các Global Services (IAM, Route 53, CloudFront, STS) khỏi lệnh Deny Region.
- **B.** Bật tính năng AWS Control Tower để ghi đè SCP.
- **C.** Xóa bỏ hoàn toàn SCP và kiểm tra thủ công hàng ngày.
- **D.** Chuyển toàn bộ các Member Accounts sang một tổ chức mới.

#### CÂU 6
Đặc tính nào sau đây mô tả đúng về tài khoản **Management Account (Master Account)** trong AWS Organizations?
- **A.** Management Account bị kiểm soát nghiêm ngặt bởi toàn bộ các SCP gắn tại Root.
- **B.** **Management Account hoàn toàn MIỄN NHIỄM với mọi Service Control Policies (SCP)**.
- **C.** Management Account không thể xem hóa đơn hợp nhất (Consolidated Billing).
- **D.** Management Account không thể mời các tài khoản khác tham gia tổ chức.

#### CÂU 7
Khi một yêu cầu (Request) được gửi tới AWS, nếu một Identity-based Policy có dòng `Allow`, nhưng Resource-based Policy có dòng `Explicit Deny`, kết quả đánh giá cuối cùng sẽ là gì?
- **A.** Được phép truy cập (Allow).
- **B.** **Bị từ chối truy cập tuyệt đối (Explicit Deny trumps Allow)**.
- **C.** Yêu cầu được đưa vào hàng đợi chờ quản trị viên duyệt.
- **D.** Hệ thống hỏi mã xác thực 2 bước MFA.

#### CÂU 8
Dịch vụ nào cung cấp cổng đăng nhập tập trung (Single Sign-On) cho phép người dùng truy cập nhiều tài khoản AWS thông qua tài khoản Microsoft Active Directory doanh nghiệp hiện có?
- **A.** AWS Directory Service
- **B.** **AWS IAM Identity Center (tiền thân là AWS SSO)**
- **C.** AWS Secrets Manager
- **D.** Amazon Cognito Identity Pools

#### CÂU 9
Một ứng dụng di động cần cho phép hàng triệu người dùng đăng nhập bằng tài khoản Google hoặc Facebook để tải ảnh lên S3 mà không cần tạo IAM User cho từng người. API nào của AWS STS nên được sử dụng ngầm?
- **A.** `sts:AssumeRole`
- **B.** **`sts:AssumeRoleWithWebIdentity`** (thông qua Amazon Cognito)
- **C.** `sts:GetCallerIdentity`
- **D.** `sts:DecodeAuthorizationMessage`

#### CÂU 10
Để thiết lập một môi trường Multi-Account chuẩn tự động hóa theo AWS Best Practices kèm theo các rào chắn phòng ngừa (Preventive Guardrails qua SCP) và phát hiện (Detective Guardrails qua Config), dịch vụ nào nên được dùng?
- **A.** **AWS Control Tower**
- **B.** AWS Cloud9
- **C.** AWS CodePipeline
- **D.** AWS DataSync

---

### PHẦN 2: BẢO MẬT DỮ LIỆU & AN NINH BIÊN (CÂU 11 - 18)

#### CÂU 11
Tại sao kỹ thuật **Envelope Encryption** lại bắt buộc phải được sử dụng khi mã hóa các tệp tin có dung lượng lớn (hàng trăm MB đến GB) bằng AWS KMS?
- **A.** Vì KMS không hỗ trợ thuật toán AES-256.
- **B.** Vì API `kms:Encrypt` của AWS KMS **bị giới hạn phần cứng, chỉ cho phép mã hóa trực tiếp dữ liệu tối đa 4 KB**.
- **C.** Vì Envelope Encryption giúp giảm 100% dung lượng tệp tin.
- **D.** Vì S3 chỉ chấp nhận các tệp đã được zip.

#### CÂU 12
Ứng dụng trên EC2 cần kết nối cơ sở dữ liệu RDS MySQL. Yêu cầu đặt ra: Mật khẩu database phải được **tự động xoay vòng (Automatic Rotation) mỗi 30 ngày một lần mà không gây downtime**. Dịch vụ nào là lựa chọn tối ưu nhất?
- **A.** AWS Systems Manager Parameter Store Standard Tier
- **B.** **AWS Secrets Manager**
- **C.** Amazon S3 với Object Lock
- **D.** AWS Key Management Service (KMS)

#### CÂU 13
Một công ty cần chứng chỉ SSL/TLS miễn phí từ AWS Certificate Manager (ACM) để gắn vào **Amazon CloudFront Distribution**. Chứng chỉ này bắt buộc phải được yêu cầu tại Region nào?
- **A.** Cùng Region với máy chủ Origin.
- **B.** **Region `us-east-1` (N. Virginia)**.
- **C.** Region `eu-west-1` (Ireland).
- **D.** Bất kỳ Region nào hỗ trợ ACM.

#### CÂU 14
Website bán vé máy bay liên tục bị các bot tự động gửi hơn 5,000 requests/phút từ các địa chỉ IP lạ để quét giá vé làm nghẽn máy chủ ALB. Tính năng nào của **AWS WAF** giúp phát hiện và tự động chặn các IP này tại biên?
- **A.** SQL Injection Match Rule
- **B.** **Rate-based Rule**
- **C.** Geo-blocking Rule
- **D.** Cross-Site Scripting Rule

#### CÂU 15
Để bảo vệ hệ thống trước các cuộc tấn công từ chối dịch vụ (DDoS) quy mô lớn ở Layer 3 và Layer 4 với sự hỗ trợ 24/7 của đội phản ứng nhanh **DDoS Response Team (SRT)** và bảo hiểm chi phí co giãn đột biến, doanh nghiệp nên đăng ký gói dịch vụ nào?
- **A.** AWS Shield Standard
- **B.** **AWS Shield Advanced**
- **C.** AWS WAF Managed Rules
- **D.** Amazon Inspector

#### CÂU 16
Một ngân hàng lưu trữ hàng triệu tài liệu hợp đồng và ảnh chụp giấy tờ tùy thân trên Amazon S3. Dịch vụ nào sử dụng Machine Learning để **tự động quét và lập danh mục các dữ liệu chứa thông tin cá nhân (PII) và số thẻ tín dụng** trong các S3 bucket?
- **A.** Amazon GuardDuty
- **B.** **Amazon Macie**
- **C.** AWS CloudTrail
- **D.** Amazon Comprehend Medical

#### CÂU 17
Dịch vụ nào sử dụng Machine Learning để liên tục phân tích VPC Flow Logs, CloudTrail Management Events và DNS Logs nhằm **phát hiện các hành vi bất thường, tài khoản bị xâm nhập hoặc EC2 đào coin**?
- **A.** **Amazon GuardDuty**
- **B.** Amazon Macie
- **C.** AWS Config
- **D.** AWS Trusted Advisor

#### CÂU 18
Sự khác biệt cốt lõi về chi phí giữa **SSM Parameter Store (Standard)** và **AWS Secrets Manager** là gì?
- **A.** Parameter Store tính phí theo giây; Secrets Manager tính phí theo giờ.
- **B.** **Parameter Store Standard hoàn toàn miễn phí**; còn Secrets Manager tính phí cố định $0.40/secret/tháng + phí API calls.
- **C.** Cả hai đều hoàn toàn miễn phí.
- **D.** Secrets Manager chỉ có phí khi sử dụng dung lượng trên 1 TB.

---

### PHẦN 3: MESSAGING, DECOUPLING & STREAMING (CÂU 19 - 28)

#### CÂU 19
Một tác vụ xử lý ảnh trên EC2 worker mất trung bình 2 phút để hoàn thành. Tuy nhiên, chỉ sau 30 giây kể từ khi worker nhận tin từ SQS Standard, một worker khác lại nhận được đúng tin nhắn đó và bắt đầu xử lý lại. Cần điều chỉnh thông số nào?
- **A.** Tăng **Visibility Timeout** của SQS Queue lên hơn 2 phút (hoặc gọi API `ChangeMessageVisibility`).
- **B.** Giảm Message Retention Period xuống 30 giây.
- **C.** Bật tính năng Short Polling.
- **D.** Chuyển đổi queue sang Dead Letter Queue.

#### CÂU 20
Trong một hàng đợi **Amazon SQS FIFO**, nếu muốn các đơn hàng của *cùng một khách hàng* được xử lý đúng thứ tự nghiêm ngặt, nhưng các đơn hàng của *các khách hàng khác nhau* có thể được **xử lý song song đồng thời** bởi nhiều worker, ta cần cấu hình thông số nào?
- **A.** Đặt **`MessageGroupId`** bằng `CustomerID`.
- **B.** Đặt `MessageDeduplicationId` bằng `CustomerID`.
- **C.** Đặt tên Queue kết thúc bằng `.fifo`.
- **D.** Tăng Delay Seconds lên 15 phút.

#### CÂU 21
Khi một sự kiện thanh toán thành công xảy ra, thông tin cần được gửi đồng thời tới: (1) Hệ thống xuất hóa đơn, (2) Hệ thống kho vận, (3) Hệ thống chăm sóc khách hàng. Mỗi hệ thống có tốc độ xử lý khác nhau và cần hoạt động độc lập (Decoupled). Kiến trúc chuẩn là gì?
- **A.** Gọi HTTP POST đồng bộ tới cả 3 hệ thống.
- **B.** Đẩy event vào **1 Amazon SNS Topic**, và cấu hình **3 Amazon SQS Queues độc lập đăng ký (Subscribe)** vào Topic đó (Mô hình Fan-Out).
- **C.** Cho cả 3 hệ thống đọc chung một SQS Standard Queue duy nhất.
- **D.** Sử dụng AWS Batch để xử lý tuần tự.

#### CÂU 22
Một sàn thương mại điện tử cần thu thập hàng triệu sự kiện click chuột (Clickstream) mỗi giây và nạp thẳng vào **Amazon S3** và **Amazon Redshift** với độ trễ từ 1 đến 5 phút (Near real-time). Doanh nghiệp muốn giải pháp Serverless 100% không phải quản lý Shards. Dịch vụ nào tối ưu nhất?
- **A.** Amazon Kinesis Data Streams
- **B.** **Amazon Kinesis Data Firehose**
- **C.** Amazon SQS Standard
- **D.** AWS Glue ETL Job

#### CÂU 23
Sự khác biệt cốt lõi giữa **Kinesis Data Streams (KDS)** và **Kinesis Data Firehose** là gì?
- **A.** KDS không hỗ trợ mã hóa KMS; Firehose hỗ trợ mã hóa.
- **B.** **KDS lưu trữ dữ liệu từ 1 đến 365 ngày và cho phép Replay dữ liệu**; còn **Firehose không lưu trữ và không hỗ trợ Replay**.
- **C.** Firehose có độ trễ mili-giây thời gian thực; KDS có độ trễ hàng giờ.
- **D.** KDS là dịch vụ Serverless hoàn toàn; Firehose yêu cầu quản lý Shards.

#### CÂU 24
Một hệ thống muốn tự động kích hoạt một luồng xử lý trên AWS Step Functions khi có một sự kiện thay đổi trạng thái từ phần mềm SaaS của bên thứ ba (như Salesforce hoặc Zendesk) với nội dung JSON thỏa mãn điều kiện nhất định. Dịch vụ nào đóng vai trò Event Bus trung tâm?
- **A.** Amazon SNS Topic
- **B.** **Amazon EventBridge (Partner Event Bus)**
- **C.** Amazon SQS FIFO
- **D.** AWS AppSync

#### CÂU 25
Để giảm thiểu tối đa chi phí gọi API và tránh lãng phí CPU khi hàng đợi SQS đang rỗng, nhóm kỹ sư nên cấu hình thuộc tính nào của SQS?
- **A.** Short Polling (`WaitTimeSeconds = 0`)
- **B.** **Long Polling (`WaitTimeSeconds = 1 đến 20 giây`)**
- **C.** Tăng Visibility Timeout lên 12 giờ
- **D.** Bật tính năng Dead Letter Queue

#### CÂU 26
Khi một worker xử lý một thông điệp từ SQS bị crash liên tục 5 lần khiến thông điệp đó không thể hoàn thành, tính năng nào giúp chuyển thông điệp lỗi này sang một hàng đợi riêng để cách ly và phân tích sự cố?
- **A.** Priority Queue
- **B.** **Dead Letter Queue (DLQ) với Redrive Policy**
- **C.** Delay Queue
- **D.** SQS Temporary Queue

#### CÂU 27
Giới hạn kích thước tối đa của một thông điệp (Message Size) trên Amazon SQS là bao nhiêu, và giải pháp nào cho phép gửi các thông điệp có kích thước lên tới 2 GB?
- **A.** Tối đa 64 KB; dùng EBS để mở rộng.
- **B.** Tối đa **256 KB**; dùng thư viện **Amazon SQS Extended Client Library for Java kết hợp Amazon S3** để lưu payload lớn.
- **C.** Tối đa 1 MB; dùng DynamoDB Streams.
- **D.** Tối đa 10 MB; dùng Kinesis Data Streams.

#### CÂU 28
Tính năng nào của Amazon SNS cho phép các subscriber chỉ nhận được các thông điệp phù hợp với tiêu chí của họ (ví dụ: chỉ nhận đơn hàng có giá trị > $1000) mà không cần nhận toàn bộ thông điệp của Topic?
- **A.** **SNS Message Filtering Policy**
- **B.** SNS Dead Letter Queue
- **C.** SNS Deduplication Scope
- **D.** SNS Fan-Out Policy

---

### PHẦN 4: SERVERLESS COMPUTE & ORCHESTRATION (CÂU 29 - 36)

#### CÂU 29
Thời gian thực thi tối đa (Maximum Execution Timeout) của một hàm **AWS Lambda** là bao lâu?
- **A.** 29 giây
- **B.** 5 phút
- **C.** **15 phút (900 giây)**
- **D.** 1 giờ

#### CÂU 30
Một tác vụ xử lý chuyển đổi video dung lượng lớn mất trung bình **45 phút** để hoàn thành. Doanh nghiệp muốn chạy tác vụ này theo kiến trúc không máy chủ (Serverless) dựa trên container mà không phải quản lý cụm EC2. Dịch vụ nào là lựa chọn chuẩn xác?
- **A.** AWS Lambda với bộ nhớ 10 GB
- **B.** **AWS Fargate (Amazon ECS)**
- **C.** Amazon API Gateway
- **D.** AWS Step Functions Express Workflow

#### CÂU 31
Một API thanh toán trực tuyến sử dụng AWS Lambda cần phản hồi với độ trễ cực thấp ổn định và **không được phép xảy ra hiện tượng Khởi động lạnh (Zero Cold Start)** trong giờ cao điểm. Tính năng nào của Lambda giải quyết vấn đề này?
- **A.** Reserved Concurrency
- **B.** **Provisioned Concurrency**
- **C.** Tăng dung lượng đĩa `/tmp`
- **D.** Gắn Lambda vào VPC

#### CÂU 32
Khi một hàm AWS Lambda được cấu hình gắn vào một **Private Subnet trong VPC** để truy cập cơ sở dữ liệu nội bộ RDS, điều gì sẽ xảy ra với kết nối Internet của hàm Lambda đó?
- **A.** Lambda tự động sử dụng card mạng của AWS để ra Internet bình thường.
- **B.** **Lambda mất toàn bộ kết nối Internet mặc định; muốn đi ra Internet (gọi API ngoài), Private Subnet đó bắt buộc phải có route trỏ qua NAT Gateway ở Public Subnet**.
- **C.** Lambda sẽ bị lỗi không thể thực thi được bất kỳ dòng mã nào.
- **D.** AWS tự động cấp Elastic IP miễn phí cho hàm Lambda.

#### CÂU 33
Giới hạn thời gian chờ phản hồi tối đa (Maximum Integration Timeout) của **Amazon API Gateway** là bao nhiêu?
- **A.** **29 giây**
- **B.** 60 giây
- **C.** 15 phút
- **D.** Không giới hạn

#### CÂU 34
Một tác vụ tạo báo cáo tài chính mất 2 phút để xử lý. Nếu người dùng gọi qua API Gateway đồng bộ, họ sẽ bị lỗi HTTP 504 Gateway Timeout. Mô hình thiết kế chuẩn Serverless để xử lý trường hợp này là gì?
- **A.** Nâng cấp API Gateway lên gói Enterprise.
- **B.** **Mô hình Asynchronous Decoupled**: API Gateway kích hoạt Lambda đẩy task vào SQS và **trả về ngay lập tức HTTP 202 Accepted** kèm `TaskID` cho client; một Lambda worker chạy ngầm xử lý task trong 2 phút và ghi kết quả vào S3.
- **C.** Chuyển đổi mã nguồn sang chạy trên DynamoDB Streams.
- **D.** Giảm dung lượng file báo cáo xuống dưới 1 KB.

#### CÂU 35
Sự khác biệt cốt lõi giữa **AWS Step Functions Standard Workflows** và **Express Workflows** là gì?
- **A.** Standard chạy tối đa 5 phút; Express chạy tối đa 1 năm.
- **B.** **Standard chạy tối đa 1 năm (hỗ trợ Exactly-once và chờ người duyệt qua Task Token)**; còn **Express chạy tối đa 5 phút (tốc độ > 100,000 executions/s)**.
- **C.** Express có tính năng visual debug từng bước trên console; Standard thì không.
- **D.** Express chỉ hỗ trợ chạy trên máy chủ On-Premises.

#### CÂU 36
Loại API nào của Amazon API Gateway cung cấp khả năng duy trì kết nối hai chiều (Full-Duplex Stateful) lý tưởng cho ứng dụng Live Chat thời gian thực?
- **A.** REST API Edge-Optimized
- **B.** HTTP API
- **C.** **WebSocket API**
- **D.** Private REST API

---

### PHẦN 5: MONITORING, AUDITING & GOVERNANCE (CÂU 37 - 44)

#### CÂU 37
Tại sao trang giám sát mặc định của Amazon CloudWatch trên một EC2 instance không hiển thị chỉ số **% sử dụng bộ nhớ RAM (Memory Utilization)**?
- **A.** Vì AWS quên chưa bật tính năng này.
- **B.** **Vì bộ nhớ RAM thuộc quyền quản lý của hệ điều hành máy khách (Guest OS Level)**, phần cứng ảo hóa bên ngoài của AWS không thể đọc được dữ liệu bên trong RAM nếu không cài **Unified CloudWatch Agent**.
- **C.** Vì instance đó chưa được gắn Elastic IP.
- **D.** Vì chỉ có các instance dòng C-type (Compute Optimized) mới có RAM.

#### CÂU 38
Một kỹ sư cần mở phiên làm việc dòng lệnh (Shell) vào một máy chủ EC2 nằm hoàn toàn trong **Private Subnet** (không có Public IP) mà **không cần mở cổng 22 (SSH) trên Security Group và không dùng Bastion Host**. Giải pháp nào là chuẩn nhất?
- **A.** Sử dụng **AWS Systems Manager Session Manager** thông qua Interface VPC Endpoints.
- **B.** Gán Elastic IP tạm thời vào instance.
- **C.** Cấu hình CloudWatch Logs Insights để gửi lệnh shell.
- **D.** Đặt máy chủ vào vị trí DMZ của VPC.

#### CÂU 39
Để truy vết xem **ai đã tải xuống một tệp tin nhạy cảm cụ thể (`s3:GetObject`) từ một S3 bucket**, quản trị viên cần bật tính năng nào trong AWS CloudTrail?
- **A.** Management Events
- **B.** **Data Events cho S3 bucket đó**
- **C.** Insights Events
- **D.** CloudTrail Log File Integrity

#### CÂU 40
Tính năng nào của AWS CloudTrail giúp chứng minh trước kiểm toán viên rằng **các tệp log lịch sử gọi API chưa từng bị sửa đổi, chèn thêm hoặc xóa bớt** sau khi được lưu trữ vào S3?
- **A.** CloudTrail Event History
- **B.** **CloudTrail Log File Integrity Validation**
- **C.** S3 Intelligent-Tiering
- **D.** AWS Key Management Service (KMS) Default Key

#### CÂU 41
Một công ty muốn liên tục theo dõi và kiểm tra xem có bất kỳ Security Group nào trong toàn bộ tài khoản bị mở cổng 22 (SSH) cho toàn thế giới (`0.0.0.0/0`) hay không, và nếu có thì **tự động sửa lỗi (Remediation)** đóng cổng đó lại ngay lập tức. Dịch vụ nào thực hiện việc này?
- **A.** Amazon CloudWatch Alarms
- **B.** **AWS Config (Config Rules) kết hợp AWS Systems Manager Automation Document**
- **C.** Amazon GuardDuty
- **D.** AWS Trusted Advisor

#### CÂU 42
Đội ngũ DevOps muốn nhận được cảnh báo qua email ngay lập tức nếu chuỗi ký tự `[ERROR]` hoặc mã lỗi `HTTP 500` xuất hiện nhiều hơn 100 lần trong 5 phút bên trong tệp log ứng dụng trên EC2. Chuỗi giải pháp chuẩn mực là gì?
- **A.** Unified CloudWatch Agent $\rightarrow$ **CloudWatch Logs $\rightarrow$ Metric Filter $\rightarrow$ CloudWatch Alarm $\rightarrow$ Amazon SNS Topic**.
- **B.** CloudTrail $\rightarrow$ S3 Glacier $\rightarrow$ SNS Topic.
- **C.** VPC Flow Logs $\rightarrow$ Kinesis Firehose $\rightarrow$ SNS Topic.
- **D.** AWS Config $\rightarrow$ Systems Manager $\rightarrow$ SES.

#### CÂU 43
Hành động nào sau đây **KHÔNG THỂ** được kích hoạt trực tiếp từ một **Amazon CloudWatch Alarm**?
- **A.** Gửi thông báo tới một Amazon SNS Topic.
- **B.** Kích hoạt chính sách co giãn của Auto Scaling Group.
- **C.** Thực hiện hành động Reboot, Stop, Terminate hoặc Recover một EC2 instance.
- **D.** **Tự động xóa một S3 Bucket đang chứa dữ liệu**.

#### CÂU 44
Tính năng nào của AWS Systems Manager cho phép chạy các script bash hoặc PowerShell đồng loạt trên **hàng trăm máy chủ EC2 cùng một lúc** mà không cần mở kết nối SSH vào từng máy?
- **A.** SSM Session Manager
- **B.** **SSM Run Command**
- **C.** SSM Parameter Store
- **D.** SSM Inventory

---

### PHẦN 6: DISASTER RECOVERY & RESILIENCE (CÂU 45 - 50)

#### CÂU 45
Chỉ số **RPO (Recovery Point Objective)** trong kế hoạch khắc phục thảm họa định nghĩa điều gì?
- **A.** Thời gian tối đa hệ thống được phép ngừng hoạt động (Downtime).
- **B.** **Khoảng thời gian tối đa dữ liệu có thể bị mất mát tính từ lần sao lưu gần nhất**.
- **C.** Chi phí tối đa để mua sắm máy chủ mới.
- **D.** Số lượng nhân viên cần thiết để khôi phục hệ thống.

#### CÂU 46
Chiến lược Disaster Recovery nào sau đây có **chi phí thấp nhất**, nhưng chấp nhận thời gian ngừng hoạt động (RTO) và mất dữ liệu (RPO) kéo dài từ vài giờ đến vài ngày?
- **A.** Pilot Light
- **B.** Warm Standby
- **C.** **Backup & Restore**
- **D.** Multi-Site Active/Active

#### CÂU 47
Trong chiến lược **Pilot Light**, hạ tầng tính toán (Compute - EC2 instances) tại Region dự phòng thường ở trạng thái nào khi hệ thống chính đang vận hành bình thường?
- **A.** Chạy 100% công suất phục vụ người dùng.
- **B.** Chạy một phiên bản thu nhỏ 24/7 gồm các máy nhỏ.
- **C.** **Hoàn toàn tắt hoặc ở mức 0 (Auto Scaling Group có `Desired = 0`)**, chỉ có dữ liệu cốt lõi (Database) được sao chép liên tục; khi có thảm họa mới bật máy lên.
- **D.** Chạy ngầm trong tài khoản của đối tác.

#### CÂU 48
Chiến lược Disaster Recovery nào duy trì một **phiên bản thu nhỏ (scaled-down version)** của toàn bộ hệ thống luôn chạy 24/7 ở Region dự phòng và sẵn sàng scale-out đủ tải trong vòng vài phút khi Region chính sập?
- **A.** Backup & Restore
- **B.** Pilot Light
- **C.** **Warm Standby**
- **D.** Multi-Site Active/Active

#### CÂU 49
Để ngăn chặn triệt để nguy cơ **mã độc tống tiền (Ransomware)** hoặc kẻ xấu chiếm quyền tài khoản Root rồi xóa sạch các bản sao lưu trong **AWS Backup**, quản trị viên nên kích hoạt tính năng nào?
- **A.** S3 Lifecycle Policy
- **B.** **AWS Backup Vault Lock ở chế độ Compliance Mode**
- **C.** IAM Policy với quyền Deny All
- **D.** AWS Shield Standard

#### CÂU 50
Dịch vụ nào của AWS cung cấp khả năng **sao chép khối liên tục (Block-level continuous replication)** từ máy chủ On-Premises sang một vùng đệm chi phí thấp (**Low-cost Staging Area**) trên AWS, và tự động khởi chạy máy chủ sản xuất EC2 trong vài phút khi có thảm họa?
- **A.** AWS Database Migration Service (AWS DMS)
- **B.** **AWS Elastic Disaster Recovery (AWS DRS)**
- **C.** AWS DataSync
- **D.** AWS Snowmobile

---

---

## BẢNG ĐÁP ÁN NHANH TOÀN BỘ 50 CÂU

| Câu | Đáp án | Câu | Đáp án | Câu | Đáp án | Câu | Đáp án | Câu | Đáp án |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **1** | **B** | **11** | **B** | **21** | **B** | **31** | **B** | **41** | **B** |
| **2** | **A** | **12** | **B** | **22** | **B** | **32** | **B** | **42** | **A** |
| **3** | **A** | **13** | **B** | **23** | **B** | **33** | **A** | **43** | **D** |
| **4** | **A** | **14** | **B** | **24** | **B** | **34** | **B** | **44** | **B** |
| **5** | **A** | **15** | **B** | **25** | **B** | **35** | **B** | **45** | **B** |
| **6** | **B** | **16** | **B** | **26** | **B** | **36** | **C** | **46** | **C** |
| **7** | **B** | **17** | **A** | **27** | **B** | **37** | **B** | **47** | **C** |
| **8** | **B** | **18** | **B** | **28** | **A** | **38** | **A** | **48** | **C** |
| **9** | **B** | **19** | **A** | **29** | **C** | **39** | **B** | **49** | **B** |
| **10** | **A** | **20** | **A** | **30** | **B** | **40** | **B** | **50** | **B** |
