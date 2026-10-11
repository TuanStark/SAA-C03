# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 3: COMPUTE & STORAGE

---

### CÂU 1 (Mức độ: Trung bình - Cost Optimization & Compute Workload)
Một công ty tài chính chạy một ứng dụng xử lý phân tích dữ liệu rủi ro tín dụng vào mỗi tối từ 22:00 đến 04:00 sáng hôm sau. Khối lượng công việc được chia nhỏ thành hàng trăm tác vụ độc lập được điều phối qua một hàng đợi SQS. Nếu một tác vụ bị gián đoạn, nó sẽ được đưa trở lại hàng đợi để worker khác tiếp tục xử lý mà không gây hỏng dữ liệu.

Mô hình mua tính toán (Compute purchasing option) nào sau đây giúp công ty đạt được **chi phí thấp nhất (lowest cost)**?

- **A.** Mua EC2 On-Demand Instances với Auto Scaling Group để tự động tắt sau 04:00 sáng.
- **B.** Mua Standard Reserved Instances cam kết thời hạn 1 năm thanh toán toàn bộ chi phí trả trước (All Upfront).
- **C.** Triển khai một Auto Scaling Group sử dụng **Spot Instances** cho toàn bộ các worker xử lý dữ liệu.
- **D.** Ký hợp đồng Compute Savings Plans thời hạn 3 năm cho toàn bộ số lượng instance chạy tối đa vào ban đêm.

---

### CÂU 2 (Mức độ: Nâng cao - Shared File Storage & Multi-AZ)
Một hệ thống quản lý nội dung số (CMS) chạy trên nền Linux cần được di chuyển từ trung tâm dữ liệu lên AWS. Hệ thống này bao gồm một nhóm các máy chủ web chạy trong một Auto Scaling Group phân bổ đều trên **3 Availability Zones**. Các máy chủ web này cần phải đọc và ghi đồng thời vào một thư mục lưu trữ media chung thông qua giao thức hệ thống tệp chuẩn POSIX.

Giải pháp lưu trữ nào sau đây đáp ứng yêu cầu trên với **độ tin cậy cao và tính sẵn sàng cao (High Availability)**?

- **A.** Tạo một Amazon EBS volume loại `io2`, bật tính năng **EBS Multi-Attach**, và gắn volume này vào tất cả các EC2 instances trong cả 3 AZ.
- **B.** Triển khai một cụm **Amazon EFS** (Elastic File System) và mount hệ thống tệp này vào các EC2 instances trên cả 3 AZ.
- **C.** Tạo một Amazon EBS volume loại `gp3` dung lượng lớn và sử dụng AWS DataSync để đồng bộ dữ liệu liên tục sang các instance ở AZ khác.
- **D.** Triển khai một file server chạy trên EC2 sử dụng EBS `gp3` trong một AZ, và chia sẻ thư mục qua NFS tới các instance ở các AZ khác.

---

### CÂU 3 (Mức độ: Khó - Extreme I/O Performance & Ephemeral Workload)
Một công ty game online chạy một cụm máy chủ xử lý dữ liệu vật lý mô phỏng thời gian thực (physics engine). Ứng dụng này đòi hỏi tốc độ đọc/ghi dữ liệu tạm thời với số lượng I/O cực lớn (**hơn 300,000 IOPS**) và độ trễ cực thấp (microsecond latency). Dữ liệu này chỉ là các phép tính toán tạm thời trong phiên chơi của người dùng, nếu server gặp sự cố thì phiên chơi sẽ được reset và dữ liệu có thể tính toán lại từ đầu.

Giải pháp lưu trữ nào sau đây đáp ứng yêu cầu hiệu năng trên với **chi phí tối ưu nhất**?

- **A.** Sử dụng EC2 instance type có hỗ trợ **Instance Store** (NVMe SSD storage đi kèm trực tiếp với instance).
- **B.** Tạo một Amazon EBS volume loại `io2 Block Express` được cấu hình 300,000 IOPS và gắn vào instance.
- **C.** Sử dụng cụm Amazon EFS ở chế độ Max I/O Performance.
- **D.** Tạo một mảng RAID 0 gồm nhiều volume EBS `gp3` gắn vào instance.

---

### CÂU 4 (Mức độ: Trung bình - Compliance & Software Licensing)
Một ngân hàng cần di chuyển một ứng dụng cơ sở dữ liệu quan trọng lên AWS EC2. Ứng dụng này sử dụng giấy phép phần mềm độc quyền (proprietary software license) của bên thứ ba, và điều khoản cấp phép quy định nghiêm ngặt rằng:
1. License được tính tiền theo **số lượng socket và số core vật lý (physical cores)** của máy chủ phần cứng.
2. Ngân hàng phải chứng minh được rằng máy chủ này không chia sẻ phần cứng vật lý với bất kỳ khách hàng nào khác của AWS.

Solutions Architect nên khuyến nghị phương án nào sau đây?

- **A.** Khởi chạy EC2 instances trong một Default Multi-Tenant VPC với cấu hình Reserved Instances.
- **B.** Khởi chạy EC2 instances sử dụng **Dedicated Hosts**.
- **C.** Khởi chạy EC2 instances với tenancy được đặt thành **Dedicated Instances**.
- **D.** Sử dụng AWS Organizations Service Control Policies (SCP) để cấm các account khác truy cập vào cụm máy chủ.

---

### CÂU 5 (Mức độ: Trung bình - EBS Architecture & AZ Migration)
Một ứng dụng thương mại điện tử có database đang chạy trên một EC2 instance nằm tại `us-east-1a` gắn với một EBS volume chứa dữ liệu giao dịch. Do nhu cầu tái cấu trúc mạng, nhóm hạ tầng cần chuyển toàn bộ cơ sở dữ liệu và dữ liệu trên EBS volume này sang chạy tại một Subnet mới nằm ở `us-east-1b`.

Quy trình nào sau đây thực hiện việc chuyển đổi an toàn và đúng nguyên lý kỹ thuật của AWS nhất?

- **A.** Tháo rời (detach) EBS volume từ instance ở `us-east-1a`, sau đó gắn (attach) trực tiếp volume này vào một instance mới ở `us-east-1b`.
- **B.** Tạo một bản **Snapshot** của EBS volume ở `us-east-1a`. Tại `us-east-1b`, tạo một **EBS volume mới từ Snapshot** này, sau đó gắn vào instance mới ở `us-east-1b`.
- **C.** Sử dụng AWS KMS để tạo một bản sao mã hóa của EBS volume và chỉ định đích đến là `us-east-1b`.
- **D.** Bật tính năng EBS Multi-AZ trên volume hiện tại, sau đó thực hiện lệnh failover sang `us-east-1b`.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **C**
- **Tại sao đúng**: 
  - Đề bài có 3 từ khóa then chốt: **"các tác vụ độc lập qua hàng đợi SQS"**, **"nếu gián đoạn thì worker khác xử lý lại"** (nghĩa là workload này hoàn toàn chịu lỗi - fault-tolerant và stateless), và **"chi phí thấp nhất (lowest cost)"**.
  - **Spot Instances** mang lại mức chiết khấu lớn nhất trong toàn bộ hệ sinh thái AWS (lên tới **90%** so với On-Demand). Khi một Spot Instance bị thu hồi, message trong SQS sẽ tự động xuất hiện lại sau Visibility Timeout để instance khác xử lý, không gây mất mát dữ liệu.
- **Phân tích bẫy**:
  - *Phương án A*: On-Demand dù có ASG tắt sau 04:00 sáng vẫn có đơn giá theo giờ rất cao so với Spot.
  - *Phương án B & D*: Reserved Instances và Savings Plans yêu cầu cam kết trả tiền 24/7 trong 1-3 năm. Trong khi workload này chỉ chạy 6 tiếng/ngày (22h đến 4h), nếu mua RI/Savings Plans cho toàn bộ tải tối đa thì 18 tiếng còn lại trong ngày bạn vẫn phải trả tiền mà không sử dụng, gây lãng phí ngân sách nghiêm trọng.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - Yêu cầu đề bài: **Linux**, **chuẩn POSIX**, **chia sẻ đọc/ghi đồng thời**, và trải rộng trên **3 Availability Zones**.
  - **Amazon EFS** là giải pháp chuẩn mực duy nhất đáp ứng trọn vẹn: là dịch vụ tệp mạng chia sẻ cấp độ Region (Multi-AZ), hỗ trợ hàng trăm EC2 instances ở các AZ khác nhau mount và thao tác file cùng lúc qua giao thức NFSv4.
- **Phân tích bẫy**:
  - *Phương án A (Bẫy rất tinh vi)*: Tính năng **EBS Multi-Attach** chỉ cho phép gắn đồng thời vào nhiều EC2 instances trong **CÙNG 1 AVAILABILITY ZONE**. EBS volume là tài nguyên bị khóa theo AZ (AZ-locked), không thể gắn vào instance ở AZ khác.
  - *Phương án C*: Dùng DataSync để đồng bộ dữ liệu liên tục không phải là giải pháp hệ thống tệp chia sẻ đồng thời thời gian thực (shared concurrent file system) cho web tier.
  - *Phương án D*: Tự dựng EC2 NFS server tạo ra điểm thắt cổ chai và là một **Single Point of Failure (SPOF)**, nếu AZ đó sập thì toàn bộ các AZ khác đều mất kết nối file storage.

---

### Đáp án Câu 3: **A**
- **Tại sao đúng**: 
  - Yêu cầu: **IOPS cực lớn (> 300,000)**, **độ trễ microsecond**, **dữ liệu tính toán tạm thời** có thể reset và tính toán lại nếu mất máy chủ, và **chi phí tối ưu nhất**.
  - **Instance Store** (NVMe SSD đi kèm trực tiếp với phần cứng máy chủ) cung cấp hiệu năng I/O vượt trội hơn bất kỳ dịch vụ network storage nào, đạt hàng trăm nghìn đến hàng triệu IOPS mà **hoàn toàn không tốn thêm chi phí lưu trữ riêng** (chi phí đã nằm trong giá thuê instance). Nhược điểm mất dữ liệu khi stop/terminate hoàn toàn không ảnh hưởng đến loại workload này.
- **Phân tích bẫy**:
  - *Phương án B*: `io2 Block Express` có thể đạt tới 256,000 IOPS (vẫn chưa đạt mốc >300,000 IOPS nếu chỉ dùng 1 volume đơn lẻ) và chi phí Provisioned IOPS cho mức này là cực kỳ đắt đỏ, vi phạm yêu cầu tối ưu chi phí.
  - *Phương án C*: Amazon EFS có độ trễ ở mức millisecond (cao hơn nhiều so với microsecond của NVMe local) và không phù hợp cho tác vụ render vật lý thời gian thực.
  - *Phương án D*: RAID 0 nhiều EBS `gp3` bị giới hạn bởi băng thông kết nối mạng của EC2 (EBS-Optimized bandwidth limits) và vẫn không thể nhanh bằng bus PCI-e/NVMe của Instance Store.

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**: 
  - Đây là câu hỏi phân biệt cốt lõi giữa **Dedicated Hosts** và **Dedicated Instances**:
    - **Dedicated Hosts**: Cung cấp khả năng hiển thị và kiểm soát đầy đủ đối với **socket và physical cores** của máy chủ vật lý. Khách hàng có thể cấu hình số lượng core, đảm bảo tuân thủ điều kiện cấp phép phần mềm theo socket/core vật lý (Bring Your Own License - BYOL).
    - **Dedicated Instances**: Chỉ đảm bảo rằng các instance chạy trên phần cứng vật lý không chia sẻ với tài khoản khác, nhưng **KHÔNG cho phép người dùng kiểm soát socket hoặc core vật lý**, không hỗ trợ ràng buộc license theo core.
- **Phân tích bẫy**:
  - *Phương án A*: Default tenancy là môi trường Multi-tenant (chia sẻ phần cứng với khách hàng khác), vi phạm điều kiện 2.
  - *Phương án D*: SCP là chính sách phân quyền tổ chức AWS Organizations, hoàn toàn không liên quan đến phần cứng vật lý máy chủ.

---

### Đáp án Câu 5: **B**
- **Tại sao đúng**: 
  - **EBS Volume là một AZ-locked resource** (chỉ tồn tại và gắn được vào instance trong cùng 1 AZ).
  - Quy trình chuẩn của AWS để chuyển EBS volume sang AZ khác hoặc Region khác:
    1. Tạo một bản **Snapshot** của EBS volume (Snapshot được lưu trữ phân tán trên Amazon S3 ở cấp độ Region).
    2. Sử dụng bản Snapshot đó để **Create Volume mới**, tại bước này chỉ định Availability Zone đích là `us-east-1b`.
    3. Gắn volume mới tạo vào EC2 instance tại `us-east-1b`.
- **Phân tích bẫy**:
  - *Phương án A*: Không thể gắn trực tiếp một EBS volume ở AZ này vào instance ở AZ khác.
  - *Phương án C*: AWS KMS chỉ quản lý khóa mã hóa, không phải là công cụ di chuyển dữ liệu mạng.
  - *Phương án D*: AWS **không có khái niệm hay tính năng nào gọi là "EBS Multi-AZ"** (chỉ có RDS Multi-AZ).
