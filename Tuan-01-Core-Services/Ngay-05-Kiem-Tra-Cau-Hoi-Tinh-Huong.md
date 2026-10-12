# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 5: S3 STORAGE & DATA PROTECTION

---

### CÂU 1 (Mức độ: Nâng cao - S3 Compliance & Object Lock)
Một tổ chức ngân hàng đầu tư phải tuân thủ quy định tài chính quốc tế (SEC Rule 17a-4). Quy định này yêu cầu toàn bộ dữ liệu lịch sử giao dịch chứng khoán của khách hàng phải được lưu trữ theo mô hình WORM (Write Once, Read Many) trong thời hạn **5 năm**. Trong suốt thời gian này, **tuyệt đối không một ai — kể cả quản trị viên hệ thống hoặc tài khoản AWS Root Account — được phép xóa hoặc sửa đổi các bản ghi này**.

Giải pháp kiến trúc nào sau đây đáp ứng chính xác yêu cầu pháp lý trên với **ít nỗ lực quản trị nhất**?

- **A.** Bật tính năng S3 Versioning và kích hoạt MFA Delete trên bucket thông qua tài khoản Root.
- **B.** Bật tính năng **S3 Object Lock** trên S3 bucket và cấu hình retention mode là **Compliance Mode** với thời hạn lưu giữ là 5 năm.
- **C.** Bật tính năng S3 Object Lock và cấu hình retention mode là **Governance Mode**, sau đó dùng IAM Policy chặn quyền `s3:BypassGovernanceRetention` của tất cả user.
- **D.** Tạo một S3 Lifecycle Rule tự động chuyển dữ liệu sang S3 Glacier Vault và khóa Vault bằng một Vault Lock Policy.

---

### CÂU 2 (Mức độ: Trung bình - Cost Optimization & Unpredictable Access Patterns)
Một công ty phân tích dữ liệu thị trường lưu trữ hàng chục triệu tệp dữ liệu CSV trên Amazon S3 với tổng dung lượng hơn 500 TB. Một số tập dữ liệu được các nhà khoa học dữ liệu truy vấn hàng ngày, một số dữ liệu khác bị lãng quên trong nhiều tháng rồi đột ngột được truy cập dồn dập trong một dự án nghiên cứu bất ngờ. Công ty không thể dự đoán trước được tần suất truy cập của từng tập tin.
Ban giám đốc yêu cầu **tối ưu hóa tối đa chi phí lưu trữ S3** mà **không làm phát sinh thời gian chờ (latency) khi truy xuất dữ liệu** và **không tốn công sức theo dõi, phân loại file thủ công**.

Giải pháp nào sau đây là tối ưu nhất?

- **A.** Thiết lập S3 Lifecycle Rule chuyển toàn bộ các đối tượng sang S3 Standard-IA sau 30 ngày.
- **B.** Cấu hình S3 Lifecycle Rule chuyển toàn bộ dữ liệu sang **S3 Intelligent-Tiering**.
- **C.** Thiết lập S3 Lifecycle Rule chuyển dữ liệu sang S3 Glacier Instant Retrieval sau 60 ngày.
- **D.** Viết một hàm AWS Lambda chạy định kỳ hàng đêm phân tích CloudWatch S3 Metrics để chuyển đổi Storage Class của từng file.

---

### CÂU 3 (Mức độ: Cơ bản/Trung bình - Secure Temporary Access)
Một ứng dụng di động cho phép người dùng chụp ảnh và tải hình đại diện của họ lên một hệ thống lưu trữ S3 bucket riêng tư (private bucket). Do số lượng người dùng lên tới hàng triệu người, công ty không muốn cấp IAM user hay IAM credentials cho các thiết bị di động. 
Đồng thời, công ty muốn người dùng tải ảnh trực tiếp lên S3 từ ứng dụng di động mà **không phải truyền file qua các máy chủ ứng dụng trung gian** để tránh làm nghẽn băng thông server.

Giải pháp nào sau đây đáp ứng được yêu cầu bảo mật và hiệu năng trên?

- **A.** Cấu hình S3 Bucket Policy cho phép quyền `s3:PutObject` công khai từ địa chỉ IP của các thiết bị di động.
- **B.** Máy chủ ứng dụng xác thực người dùng, sau đó tạo một **S3 Presigned URL** có thời hạn ngắn (vài phút) và gửi về cho ứng dụng di động để upload trực tiếp ảnh lên S3.
- **C.** Chuyển đổi S3 bucket sang chế độ Static Website Hosting và cho phép HTTP POST công khai.
- **D.** Tạo một IAM Role có quyền ghi S3 và nhúng trực tiếp AWS Access Key/Secret Key của Role này vào mã nguồn ứng dụng di động.

---

### CÂU 4 (Mức độ: Nâng cao - Encryption in Transit & Bucket Policy)
Một công ty y tế lưu trữ hồ sơ bệnh án điện tử (EHR) trên Amazon S3. Theo tiêu chuẩn bảo mật HIPAA, tất cả dữ liệu khi truyền tải qua mạng (In-Transit) giữa client và S3 bucket **bắt buộc phải được mã hóa bằng giao thức HTTPS (TLS)**. Mọi yêu cầu kết nối không an toàn qua giao thức HTTP thuần túy đều phải bị từ chối ngay lập tức.

Chính sách S3 Bucket Policy nào sau đây thực thi được quy tắc này?

- **A.** Tạo một Bucket Policy với `Effect: Allow`, `Action: s3:*`, và điều kiện `Condition: {"Bool": {"aws:SecureTransport": "true"}}`.
- **B.** Tạo một Bucket Policy với `Effect: Deny`, `Principal: *`, `Action: s3:*`, và điều kiện `Condition: {"Bool": {"aws:SecureTransport": "false"}}`.
- **C.** Bật tính năng AWS KMS (SSE-KMS) mặc định trên bucket và chọn một Customer Managed Key.
- **D.** Tạo một Network ACL cho S3 bucket chặn toàn bộ traffic đến cổng 80 (HTTP).

---

### CÂU 5 (Mức độ: Nâng cao - Cross-Region Replication & Existing Objects)
Một doanh nghiệp muốn cấu hình tính năng **Cross-Region Replication (CRR)** từ một S3 bucket chính ở `us-east-1` sang một bucket dự phòng ở `eu-west-1` để phục vụ Disaster Recovery. S3 bucket chính hiện đang chứa hơn 10 TB dữ liệu đã được tải lên từ nhiều năm trước.
Sau khi bật Versioning trên cả 2 bucket và cấu hình replication rule thành công, kỹ sư nhận thấy các file mới upload lên bucket chính đều được replicate sang bucket phụ, nhưng **toàn bộ 10 TB dữ liệu cũ trước đây không hề xuất hiện ở bucket đích**.

Giải pháp nào sau đây giúp đồng bộ toàn bộ 10 TB dữ liệu cũ sang bucket đích với chi phí và công sức tối ưu nhất?

- **A.** Viết một script Python sử dụng AWS SDK (Boto3) chạy trên một cụm EC2 lớn để quét và tải từng file từ bucket nguồn sang bucket đích.
- **B.** Tắt và bật lại tính năng S3 Versioning trên bucket nguồn để ép AWS đánh chỉ mục lại toàn bộ file.
- **C.** Sử dụng tính năng **S3 Batch Replication** để thực hiện đồng bộ một lần toàn bộ các đối tượng hiện có (existing objects) sang bucket đích.
- **D.** Tải toàn bộ 10 TB dữ liệu về on-premises qua AWS Direct Connect rồi upload ngược lại lên bucket ở `eu-west-1`.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **B**
- **Tại sao đúng**: 
  - Yêu cầu đề bài: **Mô hình WORM**, **tuân thủ quy định tài chính (SEC Rule 17a-4)**, và **tuyệt đối không ai (kể cả Root account) được phép xóa trong 5 năm**.
  - **S3 Object Lock ở chế độ Compliance Mode** được thiết kế chính xác cho quy định này. Khi bật Compliance Mode:
    - Không một IAM User, Role hay ngay cả tài khoản gốc AWS Root Account nào có thể xóa hoặc ghi đè đối tượng hay rút ngắn thời gian retention cho đến khi hết hạn.
- **Phân tích bẫy**:
  - *Phương án A*: MFA Delete chỉ yêu cầu mã OTP khi xóa, tài khoản Root có MFA vẫn có thể xóa file, không đáp ứng được yêu cầu cấm tuyệt đối mọi người xóa.
  - *Phương án C*: **Governance Mode** cho phép những người có quyền `s3:BypassGovernanceRetention` (hoặc tài khoản Root) xóa được file, vi phạm yêu cầu đề bài.
  - *Phương án D*: S3 Glacier Vault Lock có thể khóa policy, nhưng nếu dữ liệu đang nằm ở S3 thông thường thì Object Lock là tính năng native của S3, ít nỗ lực quản trị hơn nhiều so với việc migrate sang Glacier Vault.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - Đề bài có 3 ràng buộc then chốt: **"mẫu truy cập không dự đoán được (unpredictable access patterns)"**, **"không làm phát sinh độ trễ truy xuất (no retrieval latency)"**, và **"không tốn công phân loại thủ công (least operational overhead)"**.
  - **S3 Intelligent-Tiering** được sinh ra dành riêng cho kịch bản này:
    - Tự động theo dõi các mẫu truy cập của từng đối tượng.
    - Tự động di chuyển đối tượng giữa 2 tầng truy cập thường xuyên (Frequent) và ít truy cập (Infrequent) dựa trên thời gian không có request (sau 30 ngày).
    - **Hoàn toàn KHÔNG tính phí truy xuất (No retrieval fees)** và giữ nguyên độ trễ ở mức **milliseconds**.
- **Phân tích bẫy**:
  - *Phương án A*: Chuyển sang Standard-IA sẽ bị tính phí truy xuất dữ liệu ($/GB retrieval fee). Khi các nhà khoa học dữ liệu đột ngột truy vấn 500 TB, chi phí truy xuất sẽ tăng vọt cực kỳ đắt đỏ.
  - *Phương án C*: Glacier Instant Retrieval có phí truy xuất cao hơn nhiều so với Standard-IA, không tối ưu cho dữ liệu truy cập thất thường dung lượng lớn.
  - *Phương án D*: Tự viết Lambda gây tốn chi phí vận hành, phức tạp và không tối ưu bằng dịch vụ managed có sẵn của AWS.

---

### Đáp án Câu 3: **B**
- **Tại sao đúng**: 
  - **S3 Presigned URL** là giải pháp chuẩn kiến trúc AWS cho phép người dùng không có tài khoản AWS (như mobile app users) thực hiện thao tác upload (`PUT`) hoặc download (`GET`) trực tiếp với S3 bucket riêng tư một cách an toàn.
  - Luồng hoạt động:
    1. Mobile App gửi yêu cầu upload lên Server (API Gateway / EC2).
    2. Server kiểm tra quyền người dùng, sinh ra một Presigned URL có thời hạn (ví dụ: hết hạn sau 5 phút).
    3. Mobile App dùng Presigned URL này để upload trực tiếp file lên S3 qua HTTPS.
    4. Giảm tải hoàn toàn băng thông mạng và CPU cho cụm server backend.
- **Phân tích bẫy**:
  - *Phương án A*: Mở quyền công khai theo IP di động là bất khả thi vì IP mạng 4G/5G thay đổi liên tục, và gây lộ bucket ra ngoài Internet.
  - *Phương án C*: Biến bucket thành website công khai làm lộ toàn bộ dữ liệu người dùng khác.
  - *Phương án D*: Nhúng cứng Access Key/Secret Key vào app mobile là lỗi bảo mật nghiêm trọng (bị decompile và đánh cắp credentials).

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**: 
  - Cơ chế đánh giá Policy của AWS luôn tuân thủ nguyên tắc: **Explicit Deny luôn ghi đè mọi Allow**.
  - Để chặn hoàn toàn mọi traffic HTTP không an toàn, ta dùng một Bucket Policy với `Effect: Deny` kết hợp với điều kiện biến toàn cục:
    `"Condition": {"Bool": {"aws:SecureTransport": "false"}}`
  - Nếu kết nối gửi vào S3 không sử dụng giao thức HTTPS/TLS, giá trị `aws:SecureTransport` sẽ là `false` $\rightarrow$ Khớp với điều kiện Deny $\rightarrow$ Request bị từ chối ngay lập tức với mã lỗi HTTP 403 Forbidden.
- **Phân tích bẫy**:
  - *Phương án A*: Dùng `Effect: Allow` không có tác dụng chặn triệt để, vì nếu có một IAM Policy khác cho phép truy cập thì request HTTP vẫn có thể lọt qua.
  - *Phương án C*: SSE-KMS là mã hóa lúc lưu trữ tĩnh (Encryption at Rest), hoàn toàn không liên quan đến mã hóa đường truyền mạng (Encryption in Transit).
  - *Phương án D*: S3 là dịch vụ Regional Public Endpoint, không nằm trong VPC Subnet nên không có Network ACL gắn vào S3 bucket.

---

### Đáp án Câu 5: **C**
- **Tại sao đúng**: 
  - Theo nguyên lý của AWS, tính năng **S3 Replication (cả CRR và SRR) chỉ tự động replicate các đối tượng MỚI được upload hoặc thay đổi sau khi replication rule được bật**. Nó KHÔNG tự động replicate dữ liệu lịch sử đã tồn tại từ trước.
  - Trước đây kỹ sư phải tự viết script, nhưng AWS đã ra mắt tính năng **S3 Batch Replication**:
    - Cho phép tạo một Batch Operations job chỉ với vài cú click trên Console hoặc 1 lệnh CLI.
    - Tự động replicate toàn bộ các đối tượng có sẵn (existing objects) sang bucket đích mà không cần dựng thêm hạ tầng EC2 hay code script.
- **Phân tích bẫy**:
  - *Phương án A*: Tự viết script và dựng EC2 tốn kém thời gian, chi phí compute và phức tạp hơn rất nhiều so với S3 Batch Replication có sẵn.
  - *Phương án B*: Tắt và bật lại Versioning không kích hoạt replication cho dữ liệu cũ.
  - *Phương án D*: Tải 10 TB về on-premise rồi đẩy lại lên AWS gây lãng phí băng thông, thời gian và chi phí data transfer không cần thiết.
