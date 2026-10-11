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

> 💡 **Chiến lược phối hợp Compute tối ưu chi phí theo AWS Well-Architected Framework**:
> - **Lớp tải nền (Baseline steady-state load)**: Dùng **Savings Plans** hoặc **Reserved Instances** (Chạy 24/7 với chi phí rẻ nhất).
> - **Lớp tải tăng đột biến theo mùa vụ hoặc có thể dự đoán trước**: Dùng **On-Demand Instances**.
> - **Lớp xử lý nền bất đồng bộ, tính toán song song, stateless có thể chạy lại nếu lỗi**: Dùng **Spot Instances** kết hợp Auto Scaling Group với **Mixed Instances Policy** (kết hợp On-Demand làm baseline và Spot để scale-out).

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
