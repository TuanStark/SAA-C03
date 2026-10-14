# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 9: DATA PROTECTION & EDGE SECURITY

---

### CÂU 1 (Mức độ: Nâng cao - Cơ chế Envelope Encryption trong AWS KMS)
Một công ty viễn thông cần mã hóa các tệp sao lưu cơ sở dữ liệu có dung lượng từ 50 GB đến 200 GB trước khi lưu trữ vào Amazon S3. Khi kiến trúc sư đề xuất sử dụng trực tiếp API `kms:Encrypt` của AWS Key Management Service (KMS), nhóm phát triển phản hồi rằng lệnh gọi API bị lỗi từ chối và giải thích rằng KMS không hỗ trợ mã hóa trực tiếp các tệp lớn.

Giải pháp chuẩn xác nào sau đây giải quyết vấn đề mã hóa các tệp dữ liệu lớn bằng **Envelope Encryption (Mã hóa phong bì)**?

- **A.** Nén tệp sao lưu xuống dung lượng dưới 4 KB rồi mới gọi API `kms:Encrypt`.
- **B.** Sử dụng API `kms:GenerateDataKey` để lấy một cặp khóa: Plaintext Data Key và Encrypted Data Key. Ứng dụng dùng Plaintext Data Key để mã hóa tệp dữ liệu cục bộ, sau đó xóa Plaintext Data Key khỏi bộ nhớ RAM và lưu trữ Encrypted Data Key cùng với tệp đã mã hóa.
- **C.** Chuyển sang sử dụng dịch vụ AWS Secrets Manager để lưu trữ toàn bộ nội dung của tệp sao lưu 200 GB.
- **D.** Tải tệp lên Amazon S3 ở chế độ công khai rồi dùng S3 Bucket Policy để tự động mã hóa.

---

### CÂU 2 (Mức độ: Trung bình - Secrets Manager vs SSM Parameter Store)
Một ứng dụng web thương mại điện tử chạy trên EC2 kết nối tới cơ sở dữ liệu Amazon RDS PostgreSQL. Theo tiêu chuẩn bảo mật PCI-DSS, mật khẩu tài khoản quản trị database **bắt buộc phải được thay đổi (rotate) định kỳ 30 ngày một lần**. Công ty yêu cầu việc thay đổi mật khẩu này phải diễn ra **hoàn toàn tự động, không yêu cầu can thiệp thủ công và không làm gián đoạn (downtime)** các kết nối của ứng dụng.

Giải pháp nào sau đây đáp ứng yêu cầu trên với **nỗ lực vận hành ít nhất (least operational effort)**?

- **A.** Lưu trữ mật khẩu trong AWS Systems Manager Parameter Store dưới dạng `SecureString`. Viết một cron job chạy trên EC2 mỗi 30 ngày để gọi API đổi mật khẩu RDS.
- **B.** Lưu trữ thông tin đăng nhập trong **AWS Secrets Manager**, cấu hình tính năng **Automatic Rotation** với chu kỳ 30 ngày và sử dụng hàm AWS Lambda được tích hợp sẵn để tự động cập nhật mật khẩu trên RDS.
- **C.** Lưu trữ mật khẩu trong một tệp cấu hình được mã hóa bằng KMS trên Amazon S3 và thiết lập S3 Lifecycle chuyển đổi tệp mỗi 30 ngày.
- **D.** Tạo một IAM User riêng cho database và cấu hình xoay vòng Access Key bằng AWS Config Rule.

---

### CÂU 3 (Mức độ: Nâng cao - AWS Certificate Manager & CloudFront Distribution)
Một công ty khởi nghiệp triển khai ứng dụng web toàn cầu sử dụng **Amazon CloudFront** phân phối nội dung tĩnh và động từ một Application Load Balancer đặt tại Region Singapore (`ap-southeast-1`). Công ty muốn sử dụng tên miền riêng `https://www.example.com` với chứng chỉ SSL/TLS miễn phí do **AWS Certificate Manager (ACM)** cấp phát và tự động gia hạn hàng năm.
Kỹ sư yêu cầu chứng chỉ ACM tại Region Singapore (`ap-southeast-1`), nhưng khi vào cấu hình CloudFront Distribution thì không thể chọn được chứng chỉ này trong danh sách thả xuống.

Nguyên nhân chính là gì và cách khắc phục?

- **A.** CloudFront không hỗ trợ chứng chỉ SSL miễn phí từ ACM, bắt buộc phải mua chứng chỉ thương mại từ bên thứ ba.
- **B.** Để sử dụng với Amazon CloudFront, chứng chỉ ACM **bắt buộc phải được yêu cầu hoặc nhập vào tại Region `us-east-1` (N. Virginia)**. Kỹ sư cần yêu cầu lại chứng chỉ tại `us-east-1` với phương thức DNS Validation.
- **C.** Kỹ sư phải tải private key của chứng chỉ từ ACM về và cấu hình thủ công vào CloudFront.
- **D.** Cần phải chuyển toàn bộ Application Load Balancer từ Singapore về Region `us-east-1`.

---

### CÂU 4 (Mức độ: Trung bình/Nâng cao - Chống tấn công Brute-force & Lớp ứng dụng)
Một website bán vé hòa nhạc trực tuyến nhận thấy vào những ngày mở bán vé, hệ thống liên tục bị tấn công quét vé tự động (bot scraping) và tấn công brute-force đăng nhập từ hàng ngàn địa chỉ IP khác nhau. Mỗi địa chỉ IP gửi hơn 3,000 requests trong vòng 5 phút, làm quá tải cụm máy chủ Application Load Balancer (ALB).

Giải pháp nào sau đây có thể **tự động phát hiện và chặn tạm thời các địa chỉ IP có hành vi gửi request bất thường này tại biên mạng (Edge)** trước khi lưu lượng chạm vào ALB?

- **A.** Bật tính năng AWS Shield Standard trên Route 53.
- **B.** Tạo một Web ACL trên **AWS WAF** gắn vào ALB hoặc CloudFront, cấu hình một **Rate-based rule** với ngưỡng giới hạn (ví dụ: tối đa 2,000 requests trong 5 phút) để tự động block IP vi phạm.
- **C.** Sử dụng Security Group của ALB để chặn từng địa chỉ IP gửi request nhiều.
- **D.** Tăng dung lượng Auto Scaling Group của EC2 lên gấp 5 lần để hấp thụ lưu lượng tấn công.

---

### CÂU 5 (Mức độ: Trung bình - Nhận diện dữ liệu nhạy cảm PII cho Compliance)
Một công ty bảo hiểm nhân thọ lưu trữ hàng triệu tài liệu hợp đồng, ảnh chụp chứng minh nhân dân/căn cước công dân và thông tin thẻ tín dụng của khách hàng trên nhiều **Amazon S3 buckets**. Để tuân thủ quy định bảo mật dữ liệu, kiểm toán viên yêu cầu công ty phải rà soát toàn bộ các bucket này để **phát hiện và lập danh mục chính xác các vị trí đang chứa thông tin nhận dạng cá nhân (PII - Personally Identifiable Information)** và cảnh báo nếu có bucket bị mở công khai.

Dịch vụ nào của AWS được thiết kế chuyên biệt để tự động hóa hoàn toàn tác vụ này?

- **A.** Amazon GuardDuty
- **B.** AWS CloudTrail
- **C.** **Amazon Macie**
- **D.** Amazon Inspector

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **B**
- **Tại sao đúng**: 
  - AWS KMS chỉ cho phép mã hóa trực tiếp dữ liệu có kích thước **tối đa 4 KB** thông qua API `kms:Encrypt`.
  - Để mã hóa các tệp dung lượng lớn (hàng trăm MB đến GB), AWS sử dụng kỹ thuật **Envelope Encryption (Mã hóa phong bì)**:
    1. Gọi `kms:GenerateDataKey` để KMS sinh ra một cặp khóa dữ liệu (Data Key).
    2. Sử dụng **Plaintext Data Key** để mã hóa khối dữ liệu lớn cục bộ bằng thuật toán mã hóa đối xứng (như AES-GCM 256-bit).
    3. Hủy ngay lập tức Plaintext Data Key khỏi bộ nhớ RAM của máy chủ để chống lộ khóa.
    4. Lưu **Encrypted Data Key** (đã được khóa KMS CMK bảo vệ) ngay cạnh tệp dữ liệu đã mã hóa.
- **Phân tích bẫy**:
  - *Phương án A*: Tệp sao lưu 50GB không thể nén xuống dưới 4KB.
  - *Phương án C*: Secrets Manager chỉ lưu trữ chuỗi bí mật nhỏ (tối đa 64 KB), không thể chứa file 200 GB.
  - *Phương án D*: Đưa tệp lên S3 công khai gây lộ dữ liệu nghiêm trọng.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - Đề bài yêu cầu: **Xoay vòng mật khẩu database định kỳ 30 ngày hoàn toàn tự động**, **không downtime** và **ít nỗ lực quản trị nhất**.
  - **AWS Secrets Manager** là dịch vụ sinh ra để giải quyết bài toán này:
    - Tích hợp sẵn tính năng **Automatic Rotation** với Amazon RDS, Aurora.
    - AWS cung cấp sẵn template hàm AWS Lambda thực hiện việc: Đổi mật khẩu trên RDS $\rightarrow$ Cập nhật giá trị mới vào Secrets Manager $\rightarrow$ Giữ phiên bản mật khẩu cũ tạm thời để ứng dụng chuyển tiếp mượt mà không bị ngắt kết nối đột ngột.
- **Phân tích bẫy**:
  - *Phương án A*: SSM Parameter Store không có sẵn tính năng tự động xoay vòng mật khẩu out-of-the-box (phải tự viết code và tự duy trì cron job phức tạp), vi phạm ràng buộc *"least operational effort"*.
  - *Phương án C*: S3 Lifecycle chỉ chuyển storage class hoặc xóa file, không thể cập nhật mật khẩu trong database.
  - *Phương án D*: IAM User Access Key không dùng để xác thực kết nối PostgreSQL database.

---

### Đáp án Câu 3: **B**
- **Tại sao đúng**: 
  - Đây là một **quy tắc kiến trúc bắt buộc (Hard Requirement)** của AWS:
    - **Amazon CloudFront** là một dịch vụ Global Edge. Để phân phối chứng chỉ SSL/TLS tới toàn bộ mạng lưới Edge Locations trên toàn cầu, CloudFront yêu cầu chứng chỉ từ AWS Certificate Manager (ACM) **BẮT BUỘC PHẢI NẰM TẠI REGION `us-east-1` (N. Virginia)**.
    - Nếu bạn tạo chứng chỉ ACM tại bất kỳ Region nào khác (như `ap-southeast-1`, `eu-west-1`), CloudFront Distribution sẽ không hiển thị chứng chỉ đó để gán vào Custom Domain Name.
    - Phương thức **DNS Validation** là phương thức tối ưu nhất vì nó cho phép ACM tự động gia hạn chứng chỉ trước khi hết hạn thông qua bản ghi CNAME trên Route 53.
- **Phân tích bẫy**:
  - *Phương án A*: CloudFront tích hợp hoàn hảo và miễn phí với ACM.
  - *Phương án C*: Public ACM không cho phép trích xuất Private Key.
  - *Phương án D*: ALB có thể đặt ở bất kỳ Region nào (Singapore), chỉ có chứng chỉ của CloudFront mới cần nằm ở `us-east-1`.

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**: 
  - **AWS WAF (Web Application Firewall)** hoạt động ở Tầng ứng dụng (Layer 7).
  - Tính năng **Rate-based Rules** của WAF cho phép theo dõi số lượng web requests từ từng địa chỉ IP nguồn trong một cửa sổ trượt thời gian 5 phút. Nếu một IP vượt quá ngưỡng cho phép (ví dụ: > 2,000 requests/5 phút), WAF sẽ tự động chặn (Block) địa chỉ IP đó ngay tại CloudFront/ALB, ngăn chặn triệt để tấn công brute-force và web scraping mà không làm ảnh hưởng tới server backend.
- **Phân tích bẫy**:
  - *Phương án A*: AWS Shield Standard chỉ bảo vệ ở Layer 3 và Layer 4 (SYN Flood, UDP Reflection), không phân tích được số lượng HTTP requests ở Layer 7.
  - *Phương án C*: Security Group chỉ hỗ trợ ALLOW, không hỗ trợ chặn theo IP động và không thể tự động nhận diện tần suất request.
  - *Phương án D*: Scale-up EC2 làm tăng chi phí hạ tầng đột biến mà không ngăn chặn được kẻ tấn công.

---

### Đáp án Câu 5: **C**
- **Tại sao đúng**: 
  - **Amazon Macie** là dịch vụ bảo mật chuyên dụng sử dụng **Machine Learning và Pattern Matching** để tự động quét các đối tượng trong **Amazon S3** nhằm:
    - Phát hiện các dữ liệu nhạy cảm: Thông tin nhận dạng cá nhân (PII như tên, số CMT/CCCD, hộ chiếu), số thẻ tín dụng, hồ sơ y tế, khóa bí mật (Secret keys).
    - Đánh giá trạng thái an ninh của S3 buckets (ví dụ: cảnh báo các bucket đang bị cấu hình mở công khai hoặc chưa được mã hóa).
- **Phân tích bẫy**:
  - *Phương án A*: **Amazon GuardDuty** là dịch vụ giám sát thông minh phát hiện các hành vi bất thường và mã độc trên tài khoản (như EC2 bị chiếm quyền điều khiển để đào coin, tấn công brute-force SSH), GuardDuty không quét nội dung tệp tin trong S3 để tìm PII.
  - *Phương án B*: CloudTrail chỉ ghi lại nhật ký gọi API, không đọc nội dung dữ liệu bên trong file.
  - *Phương án D*: Amazon Inspector dùng để quét lỗ hổng bảo mật của hệ điều hành trên EC2, Container images trong ECR, và mã nguồn Lambda.
