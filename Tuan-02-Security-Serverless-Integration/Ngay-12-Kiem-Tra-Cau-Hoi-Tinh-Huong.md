# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 12: MONITORING, AUDITING & GOVERNANCE

---

### CÂU 1 (Mức độ: Cơ bản/Trung bình - Giám sát chỉ số RAM trên EC2)
Một công ty chạy một ứng dụng phân tích dữ liệu lớn trên một cụm EC2 instances. Nhóm vận hành phát hiện các tiến trình xử lý dữ liệu thường xuyên bị crash do tràn bộ nhớ RAM (Out Of Memory - OOM). Khi kiểm tra trang giám sát Amazon CloudWatch mặc định của các instances, nhóm chỉ nhìn thấy các chỉ số về CPU Utilization, Network In/Out và Disk Read/Write, nhưng **hoàn toàn không thấy chỉ số phần trăm sử dụng bộ nhớ RAM (Memory Utilization)**.

Giải pháp nào sau đây giúp thu thập và hiển thị chỉ số RAM của EC2 trên CloudWatch?

- **A.** Bật tính năng CloudWatch Detailed Monitoring trên từng EC2 instance.
- **B.** Nâng cấp instance type của EC2 lên cấu hình bộ nhớ RAM lớn hơn.
- **C.** Cài đặt và cấu hình **Unified CloudWatch Agent** bên trong hệ điều hành của các EC2 instances để thu thập và đẩy các chỉ số cấp độ hệ điều hành (RAM, Disk space, Swap) về CloudWatch.
- **D.** Tạo một AWS Config Rule để tự động theo dõi dung lượng bộ nhớ RAM của máy ảo.

---

### CÂU 2 (Mức độ: Nâng cao - Quản trị máy chủ EC2 an toàn không cần Bastion Host)
Một doanh nghiệp tài chính triển khai một cụm máy chủ EC2 quan trọng nằm hoàn toàn trong các **Private Subnets** của VPC (không có Public IP và không có kết nối Internet Gateway). Theo tiêu chuẩn bảo mật của ngân hàng:
1. Tuyệt đối không được mở cổng TCP 22 (SSH) hay cổng 3389 (RDP) trên các Security Group.
2. Không được sử dụng máy chủ nhảy trung gian (Bastion Hosts / Jump Boxes).
3. Các kỹ sư quản trị hệ thống vẫn phải có khả năng truy cập dòng lệnh (Shell command) an toàn để cấu hình máy chủ khi cần thiết, và toàn bộ các câu lệnh thực thi phải được ghi nhật ký kiểm toán đầy đủ.

Giải pháp nào sau đây đáp ứng trọn vẹn toàn bộ các yêu cầu an ninh trên?

- **A.** Cấu hình AWS Site-to-Site VPN từ văn phòng vào VPC và mở cổng 22 cho dải IP của văn phòng.
- **B.** Cài đặt SSM Agent và đính kèm IAM Role có chính sách `AmazonSSMManagedInstanceCore` cho các EC2 instances. Cấu hình **VPC Endpoints cho AWS Systems Manager**, và sử dụng tính năng **SSM Session Manager** để truy cập shell trực tiếp từ AWS Management Console hoặc AWS CLI.
- **C.** Gán Elastic IP tạm thời cho từng instance mỗi khi kỹ sư cần đăng nhập và gỡ bỏ ngay sau khi hoàn thành.
- **D.** Sử dụng AWS Cloud9 IDE để viết script đẩy code tự động lên máy chủ.

---

### CÂU 3 (Mức độ: Nâng cao - Truy vết hành vi đọc dữ liệu nhạy cảm trên S3)
Một tổ chức y tế lưu trữ các hồ sơ bệnh án nhạy cảm trên một Amazon S3 bucket. Một ngày nọ, ban giám đốc nghi ngờ có một cựu nhân viên đã tải xuống trái phép các tệp tin chứa thông tin bệnh nhân trước khi nghỉ việc. Ban an ninh mạng mở **AWS CloudTrail Event History** để kiểm tra nhưng **không tìm thấy bất kỳ bản ghi nào về việc tải tệp (`s3:GetObject`)** từ bucket đó, mặc dù họ nhìn thấy các sự kiện tạo bucket và thay đổi bucket policy.

Nguyên nhân chính là gì và tổ chức cần cấu hình gì để ghi lại toàn bộ các thao tác tải tệp trong tương lai?

- **A.** CloudTrail không hỗ trợ theo dõi dịch vụ Amazon S3. Cần chuyển dữ liệu sang Amazon EFS.
- **B.** Thao tác `s3:GetObject` thuộc nhóm **Data Events** trong CloudTrail. Mặc định CloudTrail chỉ ghi lại các sự kiện quản lý (**Management Events**). Tổ chức cần cấu hình Trail để **kích hoạt ghi nhận Data Events cho S3 bucket** đó.
- **C.** CloudTrail Event History chỉ lưu trữ 7 ngày, các sự kiện cũ đã bị xóa tự động.
- **D.** Cần phải nâng cấp tài khoản AWS lên gói Enterprise Support thì CloudTrail mới ghi lại thao tác đọc file.

---

### CÂU 4 (Mức độ: Nâng cao - Tự động phát hiện và khắc phục vi phạm tuân thủ)
Một tập đoàn yêu cầu tất cả các ổ đĩa **Amazon EBS volumes** được tạo ra trong toàn bộ tài khoản AWS **bắt buộc phải được mã hóa tĩnh (Encrypted at rest)**. Nếu một kỹ sư vô tình hoặc cố ý tạo một volume EBS chưa được mã hóa:
1. Hệ thống phải tự động phát hiện trạng thái không tuân thủ này.
2. Hệ thống phải tự động kích hoạt quy trình khắc phục mà không cần sự can thiệp thủ công của con người.

Kiến trúc nào sau đây đáp ứng chính xác yêu cầu này theo đúng chuẩn AWS Governance?

- **A.** Thiết lập một **AWS Config Rule** (ví dụ: `encrypted-volumes`) để liên tục theo dõi trạng thái mã hóa của các EBS volumes. Cấu hình tính năng **Automatic Remediation** trên Config Rule liên kết với một **AWS Systems Manager Automation Document** để tự động xử lý các volume không tuân thủ.
- **B.** Viết một cron job trên một EC2 instance quét các volume mỗi giờ và gọi API mã hóa.
- **C.** Sử dụng Amazon Inspector để quét các ổ đĩa định kỳ hàng tuần.
- **D.** Tạo một CloudWatch Metric Alarm dựa trên dung lượng ổ đĩa để phát hiện ổ đĩa chưa mã hóa.

---

### CÂU 5 (Mức độ: Trung bình - Giám sát lỗi ứng dụng từ Log Files)
Một ứng dụng web chạy trên EC2 ghi toàn bộ log truy cập và lỗi vào tệp tin `/var/log/application.log` trên ổ cứng máy chủ. Đội ngũ kỹ thuật muốn nhận được thông báo qua email ngay lập tức nếu tỷ lệ lỗi máy chủ nội bộ (**lỗi HTTP 500**) xuất hiện **nhiều hơn 50 lần trong vòng 5 phút**.

Giải pháp kiến trúc nào sau đây là tối ưu nhất?

- **A.** Cài đặt Unified CloudWatch Agent trên EC2 để đẩy tệp log lên **CloudWatch Logs**. Tạo một **Metric Filter** trên Log Group để đếm các dòng log chứa mã `HTTP 500` và chuyển đổi thành một Custom Metric. Thiết lập một **CloudWatch Alarm** trên Custom Metric này với ngưỡng > 50 trong 5 phút và kích hoạt thông báo gửi tới một **Amazon SNS Topic**.
- **B.** Sử dụng Amazon Athena để quét liên tục tệp log trên EC2 mỗi phút một lần.
- **C.** Tạo một AWS Lambda function chạy bằng CloudWatch Events mỗi 10 giây để đọc trực tiếp file log qua SSH.
- **D.** Kích hoạt tính năng VPC Flow Logs để phân tích mã lỗi HTTP của các gói tin mạng.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **C**
- **Tại sao đúng**: 
  - Mô hình chia sẻ trách nhiệm (Shared Responsibility Model) của AWS quy định: Bộ nhớ RAM, dung lượng ổ đĩa phân vùng trong OS, tiến trình đang chạy thuộc về tầng **Khách hàng quản lý (Guest OS Level)**.
  - AWS Hypervisor bên ngoài chỉ có thể đo đạc các thông số phần cứng ảo hóa bên ngoài: CPU Utilization, Network I/O, Disk I/O (ở mức EBS controller).
  - Để giám sát RAM, giải pháp duy nhất là cài đặt **Unified CloudWatch Agent** (hoặc custom script) vào bên trong hệ điều hành để đọc thông số từ kernel OS và đẩy về CloudWatch dưới dạng **Custom Metrics**.
- **Phân tích bẫy**:
  - *Phương án A*: Detailed Monitoring chỉ giúp tăng tần suất thu thập các metric mặc định từ 5 phút xuống 1 phút, **KHÔNG BỔ SUNG thêm chỉ số RAM**.
  - *Phương án B*: Nâng cấp RAM không giải quyết được việc giám sát chỉ số.
  - *Phương án D*: AWS Config theo dõi cấu hình tài nguyên (ví dụ: máy loại gì, gắn subnet nào), không theo dõi chỉ số hiệu năng thời gian thực.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - **AWS Systems Manager Session Manager** là chuẩn mực an ninh hiện đại nhất của AWS để quản trị máy chủ EC2 mà **không cần mở cổng SSH (port 22) hay RDP (port 3389)**.
  - Cơ chế hoạt động:
    - Máy chủ chạy **SSM Agent** và có IAM Role phù hợp (`AmazonSSMManagedInstanceCore`).
    - Giao tiếp giữa máy chủ và AWS Systems Manager diễn ra an toàn qua kênh mã hóa HTTPS (port 443) đi qua **Interface VPC Endpoints** (PrivateLink) $\rightarrow$ Instance nằm hoàn toàn trong Private Subnet không cần Public IP và không cần Internet Gateway vẫn kết nối được.
    - Kỹ sư mở terminal trực tiếp từ Console/CLI, toàn bộ phiên làm việc được ghi log kiểm toán đầy đủ vào **CloudWatch Logs** và **S3**.
- **Phân tích bẫy**:
  - *Phương án A*: Mở cổng 22 qua VPN vi phạm yêu cầu 1 (tuyệt đối không mở port 22 trên SG).
  - *Phương án C*: Gán Elastic IP đưa máy chủ ra Internet công cộng là rủi ro bảo mật cực lớn đối với hệ thống tài chính.
  - *Phương án D*: Cloud9 không phải là giải pháp quản trị shell an toàn không cần mở port cho cụm máy chủ private.

---

### Đáp án Câu 3: **B**
- **Tại sao đúng**: 
  - Đây là một điểm phân biệt cực kỳ quan trọng trong đề thi về **AWS CloudTrail**:
    - **Management Events (Sự kiện quản lý)**: Ghi lại các thao tác cấu hình tài nguyên (như `CreateBucket`, `PutBucketPolicy`, `DeleteBucket`). Được tự động ghi lại miễn phí trong 90 ngày (Event History).
    - **Data Events (Sự kiện dữ liệu)**: Ghi lại các thao tác thao túng dữ liệu bên trong tài nguyên (như `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`, hoặc Lambda `Invoke`). Do khối lượng các sự kiện này là hàng triệu/tỷ requests, AWS **MẶC ĐỊNH TẮT DATA EVENTS** để tránh phát sinh chi phí khổng lồ cho khách hàng.
    - Muốn theo dõi ai đã download tệp tin nào từ S3, quản trị viên bắt buộc phải tạo một Trail và **bật tính năng ghi Data Events cho S3 bucket** đó.
- **Phân tích bẫy**:
  - *Phương án A*: CloudTrail hỗ trợ S3 rất mạnh mẽ.
  - *Phương án C*: Event History lưu 90 ngày chứ không phải 7 ngày.
  - *Phương án D*: Data Events là tính năng cấu hình có sẵn cho mọi tài khoản, không yêu cầu gói Enterprise Support.

---

### Đáp án Câu 4: **A**
- **Tại sao đúng**: 
  - **AWS Config** là dịch vụ sinh ra để quản lý sự tuân thủ cấu hình (Compliance Management):
    - Cung cấp sẵn các AWS Managed Rules như `encrypted-volumes` để liên tục đối soát xem có volume nào chưa mã hóa hay không.
    - Tính năng **Automatic Remediation**: Khi phát hiện tài nguyên vi phạm (Non-Compliant), AWS Config tự động kích hoạt một **AWS Systems Manager Automation Document (SSM Runbook)** để thực hiện hành động khắc phục tức thì (ví dụ: tạo snapshot, mã hóa và thông báo cho người quản trị).
- **Phân tích bẫy**:
  - *Phương án B*: Tự viết cron job tốn công vận hành và không mang tính chất chuẩn mực governance của AWS.
  - *Phương án C*: Amazon Inspector dùng để quét lỗ hổng bảo mật phần mềm (CVE) và phơi nhiễm mạng, không dùng để kiểm tra mã hóa EBS.
  - *Phương án D*: CloudWatch Alarms theo dõi hiệu năng (metrics), không thể kiểm tra thuộc tính cấu hình mã hóa của volume.

---

### Đáp án Câu 5: **A**
- **Tại sao đúng**: 
  - Chuỗi liên kết dịch vụ chuẩn mực để phát hiện lỗi từ tệp log ứng dụng:
    1. **Unified CloudWatch Agent**: Thu thập các dòng log từ tệp `/var/log/application.log` trên EC2 và gửi lên **Amazon CloudWatch Logs**.
    2. **CloudWatch Metric Filter**: Quét qua các dòng log và đếm số lượng xuất hiện của chuỗi `HTTP 500`, xuất ra một **Custom Metric** (ví dụ: `HTTP500ErrorCount`).
    3. **CloudWatch Alarm**: Giám sát Custom Metric này, nếu số lượng vượt quá 50 trong vòng 5 phút $\rightarrow$ chuyển trạng thái sang `ALARM`.
    4. **Amazon SNS Topic**: Nhận tín hiệu từ Alarm và gửi email/SMS tức thì cho đội kỹ sư trực on-call.
- **Phân tích bẫy**:
  - *Phương án B*: Amazon Athena dùng để truy vấn dữ liệu lớn trên S3 (OLAP), không dùng để cảnh báo thời gian thực từng phút từ file log trên EC2.
  - *Phương án C*: Dùng Lambda chạy SSH mỗi 10 giây là một thiết kế tồi (Anti-pattern), tốn chi phí và kém bảo mật.
  - *Phương án D*: VPC Flow Logs chỉ ghi lại thông tin Header Layer 4 (Source IP, Dest IP, Port, Protocol, ACCEPT/REJECT), **HOÀN TOÀN KHÔNG ĐỌC ĐƯỢC HTTP STATUS CODES Ở LAYER 7**.
