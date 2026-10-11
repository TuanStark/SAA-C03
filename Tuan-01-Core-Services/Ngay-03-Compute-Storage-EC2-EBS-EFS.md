# NGÀY 3: COMPUTE & STORAGE (EC2, PRICING MODELS, EBS, EFS, INSTANCE STORE)
*(Tuần 1: Core Infrastructure - Trọng số Compute, Storage & Cost Optimization)*

---

## 1. MA TRẬN EC2 PRICING MODELS (DOMAIN 4: COST OPTIMIZATION)

Hiểu rõ mô hình giá của EC2 là chìa khóa để xử lý các câu hỏi tối ưu chi phí (Cost-Optimized Architectures chiếm 20% bài thi).

| Mô hình giá | Đặc điểm kỹ thuật | Trường hợp sử dụng tối ưu (Use Cases) | Khả năng gián đoạn |
| :--- | :--- | :--- | :--- |
| **On-Demand** | Trả tiền theo giây/giờ sử dụng thực tế. Không cam kết dài hạn. Giá cao nhất. | Workload ngắn hạn, không dự đoán trước được tải, ứng dụng mới thử nghiệm (Dev/Test). | Không bị gián đoạn. |
| **Spot Instances** | Giảm giá tới **90%** so với On-Demand. Dùng năng lực tính toán dư thừa của AWS. | Xử lý hàng loạt (Batch jobs), Big data (EMR), CI/CD runner, Stateless web tier có thể chịu lỗi. | **Có thể bị thu hồi bất kỳ lúc nào** với thông báo trước **2 phút** (Instance Interruption Notice). |
| **Reserved Instances (RI)** | Cam kết sử dụng **1 năm hoặc 3 năm**. Giảm giá tới **72%**. Trả trước toàn bộ (All Upfront), một phần (Partial) hoặc hàng tháng (No Upfront). | Workload trạng thái ổn định (steady-state), chạy 24/7 (Database, ERP, Core backend). | Không bị gián đoạn. |
| **Savings Plans** | Cam kết mức chi tiêu **$/giờ** (ví dụ: cam kết $10/h) trong **1 hoặc 3 năm**. Linh hoạt hơn RI. | Doanh nghiệp muốn tối ưu chi phí nhưng công nghệ compute có thể thay đổi giữa EC2, Fargate, Lambda. | Không bị gián đoạn. |
| **Dedicated Hosts** | Thuê trọn vẹn **máy chủ vật lý chuyên dụng**. Kiểm soát hoàn toàn placement của instance trên socket/core vật lý. | Tuân thủ quy định bảo mật khắt khe (Compliance) hoặc **giữ license phần mềm theo socket/core (BYOL - Bring Your Own License)**. | Không bị gián đoạn. |
| **Dedicated Instances** | Các instance chạy trên phần cứng vật lý dành riêng cho một khách hàng AWS duy nhất (không chung phần cứng với tenant khác). | Cách ly phần cứng ở mức account, nhưng không kiểm soát socket/core vật lý. | Không bị gián đoạn. |

### Phân biệt 2 loại Savings Plans hay gặp trong đề thi:
1. **Compute Savings Plans**: Linh hoạt nhất. Áp dụng cho **EC2 (bất kể instance family, OS, Region), AWS Fargate, và AWS Lambda**.
2. **EC2 Instance Savings Plans**: Giảm giá sâu hơn Compute Savings Plans, nhưng bị cố định vào một **Instance Family cụ thể trong một Region cụ thể** (ví dụ: c6g ở us-east-1), cho phép đổi OS hoặc size trong family đó.

### 1.3. CHIẾN LƯỢC TOÀN DIỆN: PHỐI HỢP 3 TẦNG TẢI & MIXED INSTANCES POLICY (AWS WELL-ARCHITECTED)

Trong bài thi SAA-C03 và thực tế vận hành enterprise, một hệ thống lớn không bao giờ chỉ dùng một mô hình giá duy nhất. AWS Well-Architected Framework khuyến nghị chia tải của hệ thống thành **3 tầng kiến trúc** để tối ưu hóa chi phí đến mức tối đa:

```text
Số lượng EC2
   ▲
40 │                                   ╭───╮ (Tầng 3: Flash Sale / Batch Jobs / SQS Workers)
   │                                  ╭╯   ╰╮ ──► DÙNG SPOT INSTANCES (Tiết kiệm tới 90%)
20 │                  ╭───────────────╯     ╰───────────────╮ (Tầng 2: Giờ cao điểm ban ngày)
   │                 ╭╯                                     ╰╮ ──► DÙNG ON-DEMAND (Linh hoạt bật/tắt)
 4 │ ───────────────────────────────────────────────────────── (Tầng 1: Tải nền thấp nhất lúc 3h sáng)
   └─────────────────────────────────────────────────────────────► ──► DÙNG SAVINGS PLANS / RI (Rẻ hơn 40-72%)
     00:00          08:00            12:00                  23:59
```

#### 1. Tầng 1: Tải nền (Baseline / Steady-state Load)
* **Bản chất**: Lượng tài nguyên tối thiểu bắt buộc phải duy trì 24/7/365 (ví dụ: tối thiểu 4 instances để duy trì heartbeat, kết nối database, xử lý phiên đêm).
* **Mô hình mua**: **Savings Plans** hoặc **Standard Reserved Instances (RI)** cam kết 1 hoặc 3 năm.
* **Lý do**: Vì các máy này chắc chắn không bao giờ tắt, việc cam kết dài hạn giúp AWS giảm giá sâu từ **40% đến 72%** so với giá niêm yết.

#### 2. Tầng 2: Tải đỉnh ban ngày có thể dự đoán (Predictable Peak Load)
* **Bản chất**: Lượng truy cập tăng cao trong giờ hành chính (từ 8h sáng đến 18h tối cần thêm 16 instances, hết giờ thì tắt).
* **Mô hình mua**: **On-Demand Instances**.
* **Lý do**: Các instances này chỉ chạy từ 8 đến 10 tiếng mỗi ngày rồi tắt. Nếu mua cam kết dài hạn (RI/Savings Plans) cho cả 20 máy thì vào ban đêm khi 16 máy này tắt, bạn vẫn bị tính tiền cam kết $\rightarrow$ gây lãng phí lớn. Dùng On-Demand bật lúc nào trả tiền lúc đó, tắt đi là hết tiền.

#### 3. Tầng 3: Tải xử lý ngầm hoặc Flash Sale đột biến (Stateless / Burst / Batch Load)
* **Bản chất**: 
  - Đột biến ngắn hạn (Flash Sale 12h trưa) hoặc các batch worker xử lý hàng đợi SQS, render video, tính toán dữ liệu ban đêm.
  - **Đặc tính kỹ thuật bắt buộc**: Phải là **Stateless** (phiên người dùng lưu ở ElastiCache Redis/DynamoDB) hoặc **Fault-Tolerant** (nhận việc từ SQS, nếu máy chết thì task tự động quay lại SQS cho máy khác xử lý).
* **Mô hình mua**: **Spot Instances**.
* **Lý do**: Tiết kiệm chi phí lên đến **90%**. Việc AWS có thể thu hồi máy bất ngờ (thông báo trước 2 phút) hoàn toàn không ảnh hưởng đến tính toàn vẹn của hệ thống.

---

### 1.4. TÍNH NĂNG "MIXED INSTANCES POLICY" TRONG AUTO SCALING GROUP (ASG)

Để triển khai chiến lược 3 tầng trên vào thực tế chỉ với **1 Auto Scaling Group duy nhất**, AWS cung cấp tính năng **Mixed Instances Policy**:

1. **Cấu hình Base Capacity (Tải nền vững chắc)**:
   - Khai báo cố định: `Set On-Demand Base Capacity = 2 instances`.
   - Đảm bảo ASG luôn duy trì tối thiểu 2 máy On-Demand kiên cố (hưởng giá rẻ từ Savings Plans ở Tầng 1).
2. **Cấu hình Scale-out Ratio (Tỷ lệ phân bổ khi mở rộng)**:
   - Khai báo: `On-Demand Percentage Above Base = 20%` (đồng nghĩa `80% Spot`).
   - Khi có tải lớn ập đến cần scale thêm 10 instances: ASG sẽ tự động bật **2 máy On-Demand** và **8 máy Spot**.
3. **Cấu hình Spot Allocation Strategy (Chiến lược dự phòng Spot)**:
   - Khai báo nhiều Instance Types tương đương nhau (ví dụ: `c5.large`, `c5a.large` chip AMD, `c6g.large` chip Graviton).
   - Chọn chiến lược: `price-capacity-optimized` $\rightarrow$ AWS sẽ tự động chọn các pool Spot vừa có giá rẻ nhất vừa có dung lượng dồi dào nhất, giảm thiểu tối đa nguy cơ bị thu hồi máy.

---

## 2. PHÂN LOẠI STORAGE: EBS vs INSTANCE STORE vs EFS

```
                             ┌───────────────────────────────────┐
                             │       AWS STORAGE CHO COMPUTE     │
                             └─────────────────┬─────────────────┘
                                               │
             ┌─────────────────────────────────┼─────────────────────────────────┐
             ▼                                 ▼                                 ▼
   [ Instance Store ]                     [ Amazon EBS ]                    [ Amazon EFS ]
   • Gắn vật lý vào host                 • Network Drive qua mạng          • Network File System (NFSv4)
   • Tốc độ cực cao (Triệu IOPS)         • Khóa trong 1 AZ                 • Multi-AZ, POSIX compliant
   • Dữ liệu TẠM THỜI (Ephemeral)        • Tách rời vòng đời với EC2       • Chia sẻ đồng thời hàng trăm EC2
   • Mất dữ liệu khi Stop/Terminate      • Hỗ trợ Snapshot lưu trên S3     • Linux-only (Windows dùng FSx)
```

### 2.1. Instance Store (Ephemeral Storage)
* **Bản chất**: Ổ cứng SSD/NVMe được gắn vật lý trực tiếp vào máy chủ phần cứng đang chạy EC2 instance.
* **Hiệu năng**: IOPS cực lớn, độ trễ sub-millisecond, không bị giới hạn băng thông qua network.
* ⚠️ **ĐẶC TÍNH SỐNG CÒN**:
  - Dữ liệu **KHÔNG BỊ MẤT** khi EC2 **Reboot**.
  - Dữ liệu **SẼ BỊ MẤT HOÀN TOÀN** khi EC2 bị **Stop (dừng)** hoặc **Terminate (xóa)** hoặc khi phần cứng vật lý gặp sự cố.
* **Trường hợp sử dụng đề thi**: Làm bộ nhớ đệm (Cache), buffer, file tạm (scratch data), dữ liệu replicate ở tầng ứng dụng (ví dụ: Hadoop HDFS, Cassandra/Kafka replicas nơi tầng app tự lo việc dự phòng).

### 2.2. Amazon Elastic Block Store (EBS)
* **Bản chất**: Block storage ảo kết nối qua mạng (Network Drive). Độc lập với vòng đời của EC2 (khi xóa EC2, volume EBS có thể được giữ lại nếu bỏ chọn *Delete on Termination*).
* **Ràng buộc AZ**: EBS Volume chỉ tồn tại trong **1 Availability Zone (AZ)** duy nhất.
  - *Muốn di chuyển EBS sang AZ khác*: Bắt buộc phải **Tạo Snapshot** $\rightarrow$ Snapshot lưu ngầm trên S3 $\rightarrow$ Dùng Snapshot đó tạo Volume mới ở AZ đích.

#### Bảng so sánh các loại EBS Volume (Cực kỳ hay hỏi):

| Loại Volume | API Name | Đơn vị đo hiệu năng | Trường hợp sử dụng tối ưu trong đề thi |
| :--- | :--- | :--- | :--- |
| **General Purpose SSD** | **gp3** (khuyên dùng) / gp2 | IOPS & Throughput mở rộng độc lập | Ổ đĩa hệ điều hành (Boot volume), môi trường Dev/Test, database tải vừa. |
| **Provisioned IOPS SSD** | **io2** / io2 Block Express | Lên tới 256,000 IOPS, 99.999% durability | Cơ sở dữ liệu trọng yếu (Mission-critical OLTP: Oracle, SAP HANA, SQL Server, PostgreSQL) cần hiệu năng ổn định sub-millisecond. **Hỗ trợ Multi-Attach**. |
| **Throughput Optimized HDD** | **st1** | MB/s (Throughput) | Dữ liệu truy cập tuần tự dung lượng lớn (Big Data, Data Warehouses, Log processing). **KHÔNG THỂ làm boot volume**. |
| **Cold HDD** | **sc1** | MB/s (Throughput) | Dữ liệu ít truy cập, lưu trữ chi phí rẻ nhất cho block storage. **KHÔNG THỂ làm boot volume**. |

> ⚠️ **Tính năng EBS Multi-Attach**:
> - Chỉ hỗ trợ trên dòng **io1 / io2**.
> - Cho phép gắn 1 volume EBS đồng thời vào **tối đa 16 EC2 instances trong CÙNG 1 AZ**.
> - Ứng dụng phải dùng hệ thống tệp cluster-aware (như GFS2, Oracle RAC). **Không tự động xử lý xung đột ghi ở cấp ứng dụng**.

### 2.3. Amazon Elastic File System (EFS)
* **Bản chất**: Hệ thống tệp chia sẻ dạng Network File System (NFSv4), tương thích chuẩn POSIX.
* **Đặc tính High Availability**: Là dịch vụ **Multi-AZ** (dữ liệu tự động replicate qua nhiều AZ trong 1 Region).
* **Khả năng chia sẻ**: Cho phép **hàng trăm/nghìn EC2 instances ở các AZ khác nhau truy cập đồng thời** (read/write đồng thời).
* **Hệ điều hành hỗ trợ**: **Chỉ hỗ trợ Linux** (Nếu đề bài yêu cầu chia sẻ file dùng SMB protocol cho Windows instances $\rightarrow$ Đáp án là **Amazon FSx for Windows File Server**).
* **Tối ưu chi phí với EFS**:
  - Dùng **EFS Lifecycle Management** để tự động chuyển các file không truy cập sau N ngày sang **EFS Infrequent Access (EFS-IA)** hoặc **EFS Archive** (giảm chi phí tới 90%).

---

## 3. LAB KIẾN TRÚC: CHỌN COMPUTE & STORAGE CHO WORKLOAD CO DÃN

### Tình huống Lab:
Một sàn thương mại điện tử chạy ứng dụng xử lý ảnh và video sản phẩm do người bán upload:
1. Người bán upload ảnh gốc lên hệ thống.
2. Một nhóm EC2 instances sẽ render và resize ảnh thành 5 kích thước khác nhau.
3. Quá trình render cần ghi dữ liệu tạm cực nhanh để xử lý file nặng.
4. Ảnh sau khi hoàn tất phải được lưu vào một nơi mà toàn bộ cụm web server ở nhiều AZ có thể đọc trực tiếp qua đường dẫn thư mục chuẩn Linux.
5. Số lượng ảnh upload tăng vọt vào khung giờ vàng (12h trưa và 20h tối), nhưng ban đêm hầu như rất ít.

```
                    ┌────────────────────────────────────────────────────────┐
                    │               SÀN THƯƠNG MẠI ĐIỆN TỬ                   │
                    └───────────────────────────┬────────────────────────────┘
                                                │
                 ┌──────────────────────────────┴──────────────────────────────┐
                 ▼                                                             ▼
     [ TẦNG XỬ LÝ ẢNH (WORKERS) ]                                  [ TẦNG WEB SERVER ĐỌC ẢNH ]
  • Compute: Auto Scaling Group kết hợp                         • Compute: EC2 chạy trong ASG
    Mixed Instances:                                              Multi-AZ (Baseline = Savings Plans).
    - 20% On-Demand (đảm bảo không gián đoạn)                   • Storage: Mount chung **Amazon EFS**
    - 80% Spot Instances (giảm 90% chi phí).                      (Multi-AZ, hỗ trợ hàng trăm EC2
  • Storage tạm khi render: **Instance Store**                     đọc/ghi đồng thời qua NFS).
    (độ trễ cực thấp, tốc độ ghi hàng GB/s).                   • Tối ưu chi phí: Bật EFS Lifecycle
    Nếu instance bị thu hồi $\rightarrow$ worker khác              chuyển ảnh cũ sau 30 ngày sang
    nhận job từ hàng đợi xử lý lại.                               **EFS Infrequent Access (IA)**.
```

---

## 4. TỪ KHÓA BẪY & CHIẾN LƯỢC CHỌN ĐÁP ÁN (EXAM PATTERNS)

| Cụm từ khóa trong đề thi | Giải pháp cốt lõi của AWS | Cạm bẫy cần tránh |
| :--- | :--- | :--- |
| **"Lowest cost for fault-tolerant / batch processing"** | **Spot Instances** | Chọn On-Demand vì sợ bị terminate (đề đã ghi rõ fault-tolerant). |
| **"Shared storage across multiple instances in multiple AZs (Linux)"** | **Amazon EFS** | Chọn EBS Multi-Attach (EBS chỉ hỗ trợ Multi-Attach trong **cùng 1 AZ**). |
| **"Shared SMB storage for Windows applications"** | **Amazon FSx for Windows** | Chọn Amazon EFS (EFS chỉ hỗ trợ Linux NFS). |
| **"Highest IOPS, ultra-low latency, temporary cache/scratch"** | **Instance Store** | Chọn Provisioned IOPS EBS (EBS vẫn đi qua mạng, đắt hơn và không nhanh bằng Instance Store). |
| **"Mission-critical relational DB needing > 64,000 IOPS"** | **EBS io2 Block Express** | Chọn gp3 (gp3 tối đa 16,000 IOPS, không đáp ứng được yêu cầu). |
| **"Bring Your Own License (BYOL) per socket/core"** | **EC2 Dedicated Hosts** | Chọn Dedicated Instances (không quản lý được số lượng socket/core vật lý). |
| **"Prevent SSRF attacks when fetching IAM credentials from EC2"** | **Enforce IMDSv2** | Chặn IP 169.254.169.254 (làm EC2 mất quyền lấy credentials hợp lệ). |
| **"Lowest network latency & highest packet throughput between EC2s"** | **Cluster Placement Group** | Chọn Spread Placement Group (Spread dùng cho HA, phân tán rack vật lý). |
| **"App takes a long time to warm up cache on startup, needs fast resume"** | **EC2 Hibernation** | Tăng kích thước instance type (không giải quyết được thời gian load cache ban đầu). |

---

## 5. CÁC CHỦ ĐỀ CHUYÊN SÂU NÂNG CAO THƯỜNG GẶP TRONG ĐỀ THI

### 5.1. EC2 Placement Groups (Chiến lược định vị máy chủ vật lý)
* **Cluster Placement Group**:
  - Gộp các instances vào cùng một rack phần cứng trong **cùng 1 Availability Zone**.
  - Đạt độ trễ mạng cực thấp (sub-millisecond) và thông lượng mạng tối đa (10 – 100 Gbps).
  - *Use Case*: High Performance Computing (HPC), Big Data tính toán phân tán chặt chẽ.
  - *Rủi ro*: Nếu rack phần cứng gặp sự cố, toàn bộ cụm instances sẽ chết.
* **Spread Placement Group**:
  - Phân tán nghiêm ngặt mỗi instance trên một giá đỡ phần cứng (rack) độc lập riêng biệt.
  - Tối đa **7 instances trên mỗi AZ**. Có thể trải dài qua nhiều AZ trong cùng Region.
  - *Use Case*: Ứng dụng trọng yếu có số lượng máy ít nhưng cần cách ly lỗi phần cứng tối đa.
* **Partition Placement Group**:
  - Chia cụm thành nhiều phân vùng (partitions/racks), mỗi phân vùng chứa nhiều instances. Các phân vùng không dùng chung rack.
  - Tối đa **7 partitions trên mỗi AZ**.
  - *Use Case*: Hadoop HDFS, Apache Cassandra, Apache Kafka (mỗi partition đại diện cho 1 failure domain).

---

### 5.2. EC2 Instance Metadata Service: IMDSv1 vs IMDSv2
* **Địa chỉ truy cập**: `http://169.254.169.254/latest/meta-data/`
* **Vấn đề an ninh**: Kẻ tấn công lợi dụng lỗi **SSRF (Server-Side Request Forgery)** trên ứng dụng web để gửi HTTP GET tới địa chỉ metadata nhằm lấy IAM Role temporary credentials.
* **Giải pháp IMDSv2**: Bắt buộc sử dụng mã thông báo phiên (**Session Token**):
  1. Lấy token: `TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")`
  2. Dùng token lấy dữ liệu: `curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/`
* Để bảo vệ EC2: Bật tính năng **Require IMDSv2** (`HttpTokens=required`).

---

### 5.3. Quy trình mã hóa EBS Volume & Chia sẻ Cross-Account
1. **Mã hóa một Volume chưa mã hóa**:
   - Snapshot volume cũ $\rightarrow$ Copy Snapshot (tích chọn **Encrypt** với KMS Key) $\rightarrow$ Tạo Volume mới từ Encrypted Snapshot $\rightarrow$ Gắn vào EC2.
2. **Chia sẻ Encrypted Snapshot sang Account khác**:
   - Snapshot mã hóa bằng default key `aws/ebs` **KHÔNG THỂ** chia sẻ ra ngoài account.
   - Bắt buộc phải mã hóa bằng **Customer Managed Key (CMK)** $\rightarrow$ Thêm quyền truy cập khóa KMS cho Account đích trong KMS Key Policy $\rightarrow$ Chia sẻ Snapshot.

---

### 5.4. Hệ sinh thái Amazon FSx (So sánh toàn diện)
* **FSx for Windows File Server**: Hỗ trợ giao thức SMB, NTFS, tích hợp Microsoft Active Directory, DFS Namespaces. Hỗ trợ Multi-AZ.
* **FSx for Lustre**: Hệ thống tệp hiệu năng cực cao dành cho HPC, AI/Machine Learning, kết nối trực tiếp hai chiều với Amazon S3.
* **FSx for NetApp ONTAP**: Hỗ trợ đồng thời cả NFS và SMB, hỗ trợ tính năng Deduplication, Snapshot của NetApp để migrate trực tiếp hạ tầng NAS on-premises lên AWS.

---

### 5.5. EC2 Hibernation (Ngủ đông)
* Dump toàn bộ dữ liệu trên RAM xuống **Root EBS Volume** trước khi tắt máy.
* Khi start lại, nạp RAM trực tiếp từ EBS $\rightarrow$ Ứng dụng khôi phục tức thì mà không cần load lại bộ nhớ đệm hay boot lại toàn bộ tiến trình.
* **Yêu cầu kỹ thuật**: Root volume phải là **EBS, phải được mã hóa (Encrypted)**, và dung lượng đủ chứa toàn bộ RAM.
