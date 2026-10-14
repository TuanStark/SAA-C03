# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 8: IAM, ORGANIZATIONS & MULTI-ACCOUNT

---

### CÂU 1 (Mức độ: Trung bình - Bản chất Service Control Policy & Root User)
Một tập đoàn đa quốc gia sử dụng **AWS Organizations** để quản lý 50 tài khoản AWS. Để đảm bảo an toàn thông tin, Giám đốc An ninh mạng (CISO) yêu cầu phải ngăn chặn tuyệt đối việc bất kỳ ai có thể xóa bảng nhật ký kiểm toán **AWS CloudTrail** trong tất cả các Member Accounts của tổ chức. 
Một kỹ sư tạo một Service Control Policy (SCP) chứa điều kiện Deny đối với hành động `cloudtrail:DeleteTrail` và `cloudtrail:StopLogging`, sau đó gắn SCP này vào Root của Organization.

Điều gì sau đây mô tả chính xác phạm vi hiệu lực của SCP này?

- **A.** SCP sẽ chặn các IAM User và IAM Role thông thường, nhưng tài khoản Root User của Member Accounts vẫn có thể xóa CloudTrail.
- **B.** SCP sẽ chặn toàn bộ người dùng trong các Member Accounts, **bao gồm cả tài khoản Root User của các Member Accounts**, nhưng **KHÔNG áp dụng** lên tài khoản Management Account (Master Account).
- **C.** SCP sẽ tự động áp dụng lên toàn bộ các tài khoản trong Organization bao gồm cả Management Account.
- **D.** SCP chỉ có hiệu lực cảnh báo trên CloudWatch, muốn chặn thực tế phải cấu hình IAM Policy trong từng Member Account.

---

### CÂU 2 (Mức độ: Nâng cao - Permissions Boundaries & Chống leo thang đặc quyền)
Một công ty công nghệ muốn trao quyền tự chủ cho các kỹ sư phát triển phần mềm (Developers). Các kỹ sư này cần được phép tự tạo các IAM Roles mới và đính kèm IAM Policies vào các roles đó để phục vụ cho các ứng dụng AWS Lambda và Amazon EC2 mà họ đang xây dựng. 
Tuy nhiên, nhóm Quản trị An ninh yêu cầu:
1. Các kỹ sư không được phép tự cấp cho mình hoặc cho các roles mới tạo quyền quản trị tối cao (`AdministratorAccess`).
2. Các kỹ sư không thể leo thang đặc quyền (Privilege Escalation) để vượt qua các hạn mức an ninh cho phép của công ty.

Giải pháp kiến trúc IAM nào sau đây đáp ứng chính xác yêu cầu trên?

- **A.** Sử dụng AWS Organizations SCP gắn vào tài khoản của nhóm Developers.
- **B.** Thiết lập một **Permissions Boundary** định nghĩa các quyền hạn tối đa được phép, và đính kèm điều kiện trong IAM Policy của Developer bắt buộc họ phải gán Permissions Boundary này vào bất kỳ IAM Role nào mà họ tạo ra.
- **C.** Cấu hình Amazon GuardDuty để quét các IAM Role mới tạo và tự động xóa nếu có quyền Admin.
- **D.** Tạo một IAM Group dành cho Developers và chỉ cấp quyền `iam:CreateRole` ở chế độ Read-Only.

---

### CÂU 3 (Mức độ: Khó - Third-party SaaS Access & The Confused Deputy Problem)
Một công ty tài chính muốn sử dụng một dịch vụ SaaS của bên thứ ba (Third-party Vendor) để tự động phân tích chi phí và tối ưu hóa tài nguyên AWS trong tài khoản của mình. Dịch vụ SaaS này cũng phục vụ hàng ngàn khách hàng doanh nghiệp khác trên cùng một nền tảng của họ.
Để tích hợp, công ty cần tạo một IAM Role trong tài khoản của mình cho phép tài khoản AWS của bên thứ ba thực hiện lệnh `sts:AssumeRole`. 

Để bảo vệ công ty khỏi lỗ hổng bảo mật **"Người đại diện bối rối" (The Confused Deputy Problem)** — trong đó một khách hàng xấu khác của bên thứ ba cố tình lừa phần mềm SaaS truy cập trái phép vào tài khoản của công ty bạn — Solutions Architect bắt buộc phải cấu hình điều gì?

- **A.** Yêu cầu bên thứ ba đăng nhập bằng tài khoản AWS Root Account của họ khi gọi API.
- **B.** Tạo một IAM User với Access Key/Secret Key dài hạn và gửi file cấu hình qua email cho bên thứ ba.
- **C.** Yêu cầu bên thứ ba cung cấp một mã định danh duy nhất (**External ID**) và thêm điều kiện kiểm tra chuỗi `sts:ExternalId` này vào **Trust Policy** của IAM Role.
- **D.** Giới hạn địa chỉ IP nguồn (Source IP) trong Policy trỏ về dải IP của bên thứ ba.

---

### CÂU 4 (Mức độ: Nâng cao - Cross-Account Access Không dùng Access Key)
Một công ty có hai tài khoản AWS:
- **Tài khoản A (Workload Account)**: Chạy một ứng dụng xử lý dữ liệu trên Amazon EC2.
- **Tài khoản B (Data Lake Account)**: Chứa một Amazon S3 bucket lưu trữ các báo cáo tài chính bảo mật.

Ứng dụng trên EC2 ở Tài khoản A cần đọc dữ liệu định kỳ từ S3 bucket ở Tài khoản B. Chính sách an ninh yêu cầu **tuyệt đối không được lưu trữ bất kỳ Access Key / Secret Key dài hạn nào** trên EC2 instance hay trong code của ứng dụng.

Kiến trúc phân quyền nào sau đây là chuẩn mực nhất theo AWS Best Practices?

- **A.** Tạo một IAM User trong Tài khoản B có quyền đọc S3, sinh Access Key/Secret Key rồi cấu hình vào file `~/.aws/credentials` trên EC2 của Tài khoản A.
- **B.** Mở S3 Bucket Policy ở Tài khoản B cho phép truy cập công khai nhưng giới hạn bằng VPC ID của Tài khoản A.
- **C.** Tạo một **IAM Role trong Tài khoản B** có quyền đọc S3 bucket và cấu hình Trust Policy cho phép Tài khoản A assume. Tạo một **IAM Role (Instance Profile) gắn vào EC2 ở Tài khoản A** cho phép gọi API `sts:AssumeRole` tới Role ở Tài khoản B.
- **D.** Thiết lập kết nối AWS Direct Connect giữa Tài khoản A và Tài khoản B.

---

### CÂU 5 (Mức độ: Trung bình/Nâng cao - Data Residency & Multi-Region SCP)
Để tuân thủ luật an ninh mạng quốc gia, một doanh nghiệp tại Singapore yêu cầu các tài khoản thành viên trong **AWS Organizations** chỉ được phép tạo và vận hành tài nguyên trong 2 Region được phê duyệt là: `ap-southeast-1` (Singapore) và `us-east-1` (N. Virginia). Tuyệt đối cấm tạo tài nguyên ở các Region khác.
Tuy nhiên, quản trị viên nhận thấy rằng khi áp dụng một SCP Deny đơn giản theo điều kiện `aws:RequestedRegion`, các kỹ sư không thể đăng nhập vào AWS Management Console, không tạo được IAM Users và Route 53 bị ngừng hoạt động.

Nguyên nhân chính là gì và giải pháp khắc phục chuẩn xác nhất là gì?

- **A.** Phải nâng cấp gói AWS Support lên Enterprise Support thì SCP mới hỗ trợ lọc Region.
- **B.** Các dịch vụ như IAM, Route 53, CloudFront, WAF, STS là **Global Services** (không gắn với một Region cụ thể). SCP Deny bắt buộc phải sử dụng phần tử **`NotAction`** để loại trừ các dịch vụ Global này khỏi lệnh Deny.
- **C.** Không thể dùng SCP để chặn Region, bắt buộc phải dùng Security Group trên từng VPC.
- **D.** Cần phải tạo một Organization riêng cho từng Region.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **B**
- **Tại sao đúng**: 
  - Đây là đặc tính kỹ thuật căn bản nhưng cực kỳ quan trọng của **Service Control Policies (SCP)**:
    1. **SCP kiểm soát cả tài khoản Root của Member Account**: Người dùng đăng nhập bằng root email/password của bất kỳ Member Account nào cũng **bị chặn tuyệt đối** nếu hành vi đó bị SCP Deny. Điều này đảm bảo tính quản trị tập trung của tập đoàn.
    2. **SCP KHÔNG BAO GIỜ áp dụng lên Management Account (Master Account)**: Management Account là tài khoản sở hữu toàn bộ Organization, có quyền lực tối cao để chỉnh sửa và phục hồi hệ thống khi có sự cố, nên AWS thiết kế để SCP không khống chế tài khoản này.
- **Phân tích bẫy**:
  - *Phương án A*: Sai vì SCP có hiệu lực kiểm soát cả Root User của Member Accounts.
  - *Phương án C*: Sai vì SCP không áp dụng lên Management Account.
  - *Phương án D*: SCP là chính sách rào chắn thực thi chặn cứng (Preventive Guardrail), không phải là công cụ cảnh báo.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - **Permissions Boundaries** là tính năng cao cấp của IAM được sinh ra để giải quyết chính xác bài toán **Ủy quyền quản trị (Delegated Administration)** mà không lo leo thang đặc quyền:
    - Nhóm bảo mật tạo một Policy định nghĩa ranh giới tối đa (ví dụ: chỉ được thao tác S3, DynamoDB, CloudWatch, cấm Admin).
    - Cấp quyền cho Developer tạo Role nhưng thêm điều kiện: `"Condition": {"StringEquals": {"iam:PermissionsBoundary": "arn:aws:iam::123:policy/DevBoundary"}}`.
    - Khi đó, Developer có thể tạo bao nhiêu role tùy thích nhưng bắt buộc phải gắn Boundary này vào $\rightarrow$ Các role mới tạo vĩnh viễn không thể vượt qua mức trần quyền hạn đó.
- **Phân tích bẫy**:
  - *Phương án A*: SCP áp dụng ở cấp độ Account/OU, không thể đi sâu vào việc kiểm soát logic tạo IAM role của một nhóm user cụ thể bên trong tài khoản.
  - *Phương án C*: GuardDuty chỉ phát hiện sau khi sự việc đã rồi (Detective), không ngăn chặn được hành vi tạo quyền Admin ngay từ đầu.
  - *Phương án D*: Đặt ở chế độ Read-Only thì Developer không thể tạo được role cho Lambda/EC2 chạy ứng dụng.

---

### Đáp án Câu 3: **C**
- **Tại sao đúng**: 
  - **The Confused Deputy Problem (Vấn đề người đại diện bối rối)** xảy ra khi một bên thứ 3 (Deputy) có quyền truy cập vào tài nguyên của bạn, bị một đối tượng khác lợi dụng quyền đó để tác động lên tài nguyên của bạn.
  - Giải pháp chuẩn được AWS khuyến nghị và đưa vào đề thi:
    - Bên thứ 3 cung cấp cho bạn một chuỗi bí mật định danh tài khoản của bạn trên hệ thống của họ: **External ID**.
    - Bạn cấu hình Trust Policy của IAM Role trong tài khoản của mình:
      `"Condition": {"StringEquals": {"sts:ExternalId": "my-secret-vendor-id"}}`
    - Khi bên thứ 3 gửi request `sts:AssumeRole`, SDK của họ bắt buộc phải truyền kèm External ID này. Kẻ xấu không thể biết được chuỗi External ID này nên không thể lợi dụng bên thứ ba để tấn công vào tài khoản của bạn.
- **Phân tích bẫy**:
  - *Phương án A & B*: Vi phạm nghiêm trọng các quy tắc an toàn bảo mật căn bản của AWS (chia sẻ Root hoặc lưu Access Key dài hạn).
  - *Phương án D*: Hệ thống SaaS của bên thứ 3 thường chạy trên các cụm máy chủ tự động co giãn với dải IP động hoặc chạy trên nền tảng Serverless, lọc theo IP không ngăn chặn được kẻ xấu cùng là khách hàng của SaaS đó.

---

### Đáp án Câu 4: **C**
- **Tại sao đúng**: 
  - Đây là mô hình kiến trúc phân quyền Cross-Account chuẩn nhất của AWS Well-Architected Framework:
    1. **Tài khoản B (Chứa tài nguyên)**: Tạo một **IAM Role (Role-S3-Reader)** gắn quyền đọc S3 bucket. Trong Trust Policy của Role này, khai báo Principal cho phép IAM Role của EC2 ở Tài khoản A được phép assume.
    2. **Tài khoản A (Chứa compute)**: Gán một **IAM Role (Instance Profile)** cho EC2. Role này có quyền gọi `sts:AssumeRole` tới ARN của Role-S3-Reader ở Tài khoản B.
    3. EC2 SDK sẽ tự động gọi AWS STS để lấy **Temporary Security Credentials** (có thời hạn từ 15 phút đến 12 giờ) để đọc dữ liệu S3 mà **không cần lưu trữ bất kỳ một Access Key tĩnh nào**.
- **Phân tích bẫy**:
  - *Phương án A*: Lưu Access Key/Secret Key dài hạn trên server là anti-pattern nguy hiểm, bị trừ điểm trong đề thi.
  - *Phương án B*: Mở quyền public S3 là vi phạm nghiêm trọng chính sách bảo mật dữ liệu tài chính.
  - *Phương án D*: Direct Connect là kết nối hạ tầng mạng (Layer 3), không giải quyết được bài toán ủy quyền IAM (Layer 7).

---

### Đáp án Câu 5: **B**
- **Tại sao đúng**: 
  - AWS có một số dịch vụ cốt lõi không thuộc về bất kỳ một Region cụ thể nào mà hoạt động trên phạm vi toàn cầu (**Global Services**):
    - **IAM** (Quản lý User, Role, Policy)
    - **AWS STS** (Xác thực đăng nhập và cấp token)
    - **Amazon Route 53** (Hệ thống phân giải DNS toàn cầu)
    - **Amazon CloudFront / AWS WAF**
  - Khi bạn viết SCP Deny toàn bộ các Region khác với `Condition: {"StringNotEquals": {"aws:RequestedRegion": ["ap-southeast-1", "us-east-1"]}}`, các request gọi vào IAM hay STS sẽ bị hiểu là nằm ngoài 2 Region trên $\rightarrow$ dẫn đến **tất cả người dùng bị khóa tài khoản, không thể đăng nhập Console và không quản lý được hạ tầng**.
  - **Cách khắc phục bắt buộc**: Dùng phần tử **`NotAction`** trong SCP để loại trừ các dịch vụ Global: `"NotAction": ["iam:*", "organizations:*", "route53:*", "cloudfront:*", "waf:*", "sts:*"]`.
- **Phân tích bẫy**:
  - *Phương án A & C & D*: Đều đưa ra các lý do kỹ thuật sai nguyên lý hoạt động của AWS Organizations.
