# NGÀY 6: DATABASE SOLUTIONS (RDS MULTI-AZ vs READ REPLICA, AURORA ARCHITECTURE, DYNAMODB, ELASTICACHE)
*(Tuần 1: Core Infrastructure - Trọng số Database, High Availability & Low Latency)*

---

## 1. AMAZON RDS: MULTI-AZ vs READ REPLICA (BẢNG SO SÁNH QUYẾT ĐỊNH)

Đây là cặp bài trùng được kiểm tra nhiều nhất trong toàn bộ bài thi SAA-C03. Rất nhiều kỹ sư nhầm lẫn vai trò giữa khả năng mở rộng (Scaling) và tính sẵn sàng cao (High Availability).

```
          [ RDS MULTI-AZ ]                                     [ RDS READ REPLICA ]
      (Mục đích: HIGH AVAILABILITY)                          (Mục đích: SCALING READS)

        Availability Zone A                                      Availability Zone A
   ┌────────────────────────────┐                           ┌────────────────────────────┐
   │    Primary DB Instance     │                           │    Primary DB Instance     │
   │    (Active Read / Write)   │                           │    (Active Read / Write)   │
   └─────────────┬──────────────┘                           └─────────────┬──────────────┘
                 │                                                        │
                 │ Đồng bộ (Synchronous)                                  │ Bất đồng bộ (Asynchronous)
                 ▼                                                        ▼
        Availability Zone B                                      AZ B / Region khác
   ┌────────────────────────────┐                           ┌────────────────────────────┐
   │    Standby DB Instance     │                           │     Read Replica 1..15     │
   │    (Passive - KHÔNG ĐỌC)   │                           │    (Chỉ phục vụ ĐỌC READ)  │
   └────────────────────────────┘                           └────────────────────────────┘
```

| Tiêu chí | RDS Multi-AZ (Standby) | RDS Read Replica |
| :--- | :--- | :--- |
| **Mục đích chính** | **High Availability (HA) & Disaster Recovery** | **Tăng khả năng chịu tải đọc (Read Scalability)** |
| **Cơ chế sao chép** | **Đồng bộ (Synchronous Replication)** | **Bất đồng bộ (Asynchronous Replication)** |
| **Trạng thái phục vụ** | **Bị động (Passive)**: Không thể kết nối hay đọc/ghi. | **Chủ động (Active)**: Phục vụ các câu truy vấn SELECT/Read. |
| **Điểm kết nối (Endpoint)** | **Chỉ dùng 1 DNS Endpoint duy nhất**. Khi failover, AWS tự trỏ CNAME sang Standby. | **Mỗi Replica có 1 DNS Endpoint riêng biệt** (ứng dụng phải cấu hình trỏ vào). |
| **Khả năng Failover** | **Tự động 100% (Auto-Failover)** trong vòng 60–120 giây khi Primary chết. | **Không tự động failover**. Có thể promote thủ công hoặc qua script thành standalone DB. |
| **Phạm vi địa lý** | Chỉ trong **cùng 1 Region** (qua 2 AZs khác nhau). | Hỗ trợ: **Trong cùng AZ, Khác AZ, và Cross-Region**. |
| **Tác động hiệu năng khi Backup** | Snapshot tự động được chụp từ **Standby Instance**, không gây gián đoạn/lag Primary. | Không dùng để backup thay thế cho Primary. |

> ⚠️ **BẪY ĐỀ THI SAA-C03**:
> 1. *"Cơ sở dữ liệu RDS MySQL bị quá tải CPU do 90% truy vấn là đọc dữ liệu từ người dùng. Làm sao giảm tải cho DB?"*
>    - **Đáp án**: Tạo **RDS Read Replicas** và cấu hình ứng dụng trỏ các câu truy vấn đọc vào endpoint của Replicas (hoặc dùng **ElastiCache**).
>    - *Bẫy*: Chọn Multi-AZ (Multi-AZ là Standby thụ động, **KHÔNG phục vụ đọc**, không giải quyết được vấn đề quá tải CPU do đọc).
> 2. *"Ứng dụng cần RTO thấp và tự động khôi phục khi trung tâm dữ liệu gặp sự cố mà không cần can thiệp thủ công"*:
>    - **Đáp án**: Bật **RDS Multi-AZ**.

---

## 2. AMAZON AURORA: KIẾN TRÚC CLOUD-NATIVE RELATIONAL DATABASE

Amazon Aurora là cơ sở dữ liệu quan hệ do AWS thiết kế lại từ đầu để tận dụng tối đa nền tảng đám mây:
- Tương thích hoàn toàn với **MySQL (nhanh hơn gấp 5 lần)** và **PostgreSQL (nhanh hơn gấp 3 lần)**.
- **Tầng lưu trữ phân tán dùng chung (Shared Distributed Storage Engine)**:
  - Tách rời hoàn toàn tầng tính toán (Compute) và tầng lưu trữ (Storage).
  - Tự động sao chép **6 bản sao (6 copies) dữ liệu trải rộng trên 3 Availability Zones** (2 bản sao trên mỗi AZ).
  - Khả năng chịu lỗi: Có thể mất 1 AZ và thêm 1 bản sao nữa (mất 3/6 bản) mà vẫn đọc bình thường; mất 2 bản vẫn ghi bình thường.
  - Bộ nhớ tự động mở rộng theo nhu cầu từ 10 GB lên tới **128 TiB** mà không cần can thiệp.

### 2.1. Các loại Endpoints của cụm Aurora
```
                                   [ CLIENT / APP ]
                                          │
                 ┌────────────────────────┴────────────────────────┐
                 ▼ (Ghi / Write)                                   ▼ (Đọc / Read)
       [ Cluster Endpoint ]                              [ Reader Endpoint ]
         (Writer Endpoint)                                 (Load Balancing tự động)
                 │                                                 │
                 ▼                                  ┌──────────────┴──────────────┐
       ┌───────────────────┐                        ▼                             ▼
       │  Primary Instance │                 ┌───────────────┐             ┌───────────────┐
       │   (Read / Write)  │                 │ Aurora Replica│             │ Aurora Replica│
       └───────────────────┘                 │  (Read-Only)  │             │  (Read-Only)  │
                                             └───────────────┘             └───────────────┘
```
1. **Cluster Endpoint (Writer Endpoint)**: Luôn trỏ tới instance Primary đang giữ quyền Read/Write. Khi failover, endpoint này tự động chuyển sang replica được thăng cấp.
2. **Reader Endpoint**: Tự động cân bằng tải các kết nối đọc (Read queries) trải đều trên tất cả các Aurora Replicas có trong cụm.
3. **Custom Endpoints**: Cho phép nhóm một tập hợp các replicas có cấu hình đặc thù (ví dụ: máy cấu hình RAM lớn để chạy báo cáo phân tích, tách biệt với các replicas phục vụ web).

### 2.2. Aurora Serverless v2 vs Aurora Provisioned
* **Aurora Serverless v2**: Tự động scale compute (CPU/RAM) theo gia số rất nhỏ gọi là **ACUs (Aurora Capacity Units)** chỉ trong vòng **vài phần trăm giây (fractions of a second)**.
* Thích hợp tối ưu cho: Workload có lưu lượng không thể đoán trước, tăng đột biến trong thời gian ngắn (spiky workloads), hoặc các môi trường Dev/Test ban đêm không có người dùng.

### 2.3. Aurora Global Database (Disaster Recovery & Low Latency Toàn cầu)
* Bao gồm **1 Primary Region** (nhận cả Read/Write) và tối đa **5 Secondary Regions** (chỉ nhận Read-Only).
* Sao chép dữ liệu ở tầng lưu trữ chuyên dụng của AWS với độ trễ **dưới 1 giây (typically < 100ms)**.
* **Disaster Recovery**: Có thể thúc đẩy (promote) một Secondary Region thành Primary mới trong vòng **dưới 1 phút** với **RPO < 1s** và **RTO < 1 min**.

---

## 3. AMAZON DYNAMODB: FULLY MANAGED NOSQL DATABASE

DynamoDB là cơ sở dữ liệu NoSQL dạng Key-Value và Document có khả năng mở rộng không giới hạn với độ trễ ổn định ở mức **vài mili-giây (single-digit millisecond latency)**.

### 3.1. Các khái niệm cốt lõi:
* **Partition Key (PK - Hash Key)**: Dùng để phân bổ dữ liệu vào các ổ đĩa vật lý (Partitions) qua hàm băm.
* **Sort Key (SK - Range Key)**: Dùng để sắp xếp dữ liệu bên trong cùng một Partition.
* **Read Consistency**:
  * **Eventually Consistent Read** (Mặc định): Dữ liệu đọc có thể trễ vài mili-giây so với lần ghi gần nhất. Tiêu tốn ít tài nguyên nhất (**1/2 RCU** cho 4KB).
  * **Strongly Consistent Read**: Đảm bảo luôn đọc được dữ liệu mới nhất. Tiêu tốn gấp đôi tài nguyên (**1 RCU** cho 4KB).

### 3.2. Ba "Vũ khí bí mật" của DynamoDB trong đề thi SAA-C03:

#### 1. DynamoDB Accelerator (DAX)
* Là cụm **In-Memory Cache chuyên dụng cho DynamoDB**.
* Kéo giảm độ trễ đọc từ **Mili-giây (milliseconds) xuống Vi-giây (MICROSECONDS)**.
* Là bộ nhớ đệm trong suốt (**Transparent Cache**): Ứng dụng không cần viết lại logic code, chỉ cần đổi endpoint của SDK sang DAX cluster.
* 👉 *Từ khóa đề thi*: *"Microsecond latency for DynamoDB reads"*, *"Reduce read load on hot keys"*.

#### 2. DynamoDB Streams
* Ghi lại nhật ký theo thứ tự thời gian của mọi thay đổi ở mức item (Insert, Update, Delete) trong vòng **24 giờ**.
* Tích hợp hoàn hảo với **AWS Lambda** để tạo kiến trúc hướng sự kiện (Event-driven):
  * *Ví dụ*: Người dùng tạo tài khoản mới trong DynamoDB $\rightarrow$ Stream bắn event $\rightarrow$ Kích hoạt Lambda gửi email chào mừng và kích hoạt quy trình tích điểm.

#### 3. DynamoDB Global Tables (Active-Active Multi-Region)
* Cho phép đọc và ghi (**Multi-Region Read & Write**) đồng thời ở nhiều AWS Regions trên toàn cầu.
* Dữ liệu tự động đồng bộ 2 chiều (Fully managed two-way replication) giữa các Regions.
* 👉 *Từ khóa đề thi*: *"Active-Active multi-region database"*, *"Global low-latency read and write"*.

#### 4. Time to Live (TTL)
* Tự động xóa các bản ghi sau khi đến thời gian hết hạn (Timestamp dạng Unix epoch) mà **HOÀN TOÀN MIỄN PHÍ, KHÔNG TỐN WCU (Write Capacity Units)**.
* Lý tưởng cho: Quản lý giỏ hàng tạm, session đăng nhập, lịch sử log 30 ngày.

---

## 4. AMAZON ELASTICACHE: REDIS OSS vs MEMCACHED

ElastiCache là dịch vụ In-Memory Cache giúp giảm tải truy vấn cho cơ sở dữ liệu quan hệ và đem lại tốc độ phản hồi sub-millisecond.

| Tiêu chí | ElastiCache Redis OSS (hoặc Valkey) | ElastiCache Memcached |
| :--- | :--- | :--- |
| **Cấu trúc dữ liệu** | Hỗ trợ phong phú: String, Hash, List, Set, Sorted Set, Bitmaps, Geospatial. | Chỉ hỗ trợ dạng **Key-Value đơn giản** (chuỗi string/object). |
| **Kiến trúc luồng (Threading)** | Single-threaded | **Multi-threaded** (Tận dụng tối đa CPU nhiều core). |
| **Tính sẵn sàng cao (HA)** | **Hỗ trợ Multi-AZ với Auto-Failover** và Read Replicas. | Không có Multi-AZ, không có failover. Chết node nào mất dữ liệu node đó. |
| **Lưu trữ bền vững (Persistence)**| Hỗ trợ ghi dữ liệu xuống đĩa (AOF, RDB Snapshots). | **Thuần túy trong RAM (Pure In-Memory)**. Restart máy là mất trắng. |
| **Tính năng cao cấp** | Pub/Sub, Redis Cluster (Sharding), Backup & Restore. | Không hỗ trợ Pub/Sub hay clustering phức tạp. |
| **Trường hợp sử dụng đề thi** | • **Bảng xếp hạng game (Leaderboards)** (Sorted Sets).<br>• Lưu trữ phiên đăng nhập (**User Session Store**).<br>• Dữ liệu cần độ bền vững và tự động failover. | • Bộ nhớ đệm tệp tin/HTML tĩnh đơn giản.<br>• Mở rộng scale theo số core CPU. |

---

## 5. MA TRẬN CHỌN DATABASE THEO YÊU CẦU ĐỀ BÀI (DATABASE DECISION MATRIX)

| Yêu cầu đặc thù trong đề thi | Giải pháp Database tối ưu của AWS |
| :--- | :--- |
| Quan hệ truyền thống (SQL), ACID, tải trung bình, tự động failover | **RDS Multi-AZ** |
| SQL quan hệ, tải đọc cực lớn, cần tự động failover nhanh (<30s), scale lưu trữ tự động lên 128TB | **Amazon Aurora** (kết hợp Reader Endpoint) |
| SQL quan hệ, tải thất thường, biến động đột ngột hoặc các môi trường dev/test | **Aurora Serverless v2** |
| SQL quan hệ, cần Disaster Recovery đa vùng với RPO < 1s, RTO < 1 min | **Aurora Global Database** |
| NoSQL Key-Value, độ trễ vài mili-giây, scale không giới hạn, serverless | **Amazon DynamoDB** |
| Đọc DynamoDB với độ trễ **microsecond** (vi-giây), xử lý các bản ghi quá hot | **DynamoDB Accelerator (DAX)** |
| Ứng dụng đọc/ghi phân tán đồng thời trên toàn cầu (Active-Active) | **DynamoDB Global Tables** |
| Bảng xếp hạng game (Leaderboards), Session Store có Multi-AZ failover | **ElastiCache Redis** |
| Đồ thị mạng xã hội, phòng chống gian lận tài chính (Graph Database) | **Amazon Neptune** |
| Dữ liệu chuỗi thời gian IoT, giám sát máy móc công nghiệp (Time-series) | **Amazon Timestream** |
| Sổ cái kế toán, kiểm toán tài chính bất biến không thể sửa (Ledger) | **Amazon QLDB** |

---

## 6. LAB KIẾN TRÚC: HỆ THỐNG CƠ SỞ DỮ LIỆU THƯƠNG MẠI ĐIỆN TỬ HIỆU NĂNG CAO

```
                                  [ WEB / APP SERVERS ]
                                             │
             ┌───────────────────────────────┼───────────────────────────────┐
             │ (1) User Session              │ (2) Giỏ hàng &                │ (3) Giao dịch đặt hàng &
             │     & Leaderboard             │     Catalog sản phẩm          │     Báo cáo tài chính
             ▼                               ▼                               ▼
    ┌─────────────────┐             ┌─────────────────┐             ┌─────────────────────────┐
    │   ElastiCache   │             │ Amazon DynamoDB │             │      Amazon Aurora      │
    │      Redis      │             │ (Single-digit   │             │   (ACID Transactions)   │
    │ (Sub-millisecond│             │  ms latency)    │             │                         │
    │  Session Store) │             └────────┬────────┘             │ • Writer: Primary DB    │
    └─────────────────┘                      │                      │ • Reader: 3 Replicas    │
                                             │ (Microsecond cache)  │ • Storage: 6 bản/3 AZs  │
                                             ▼                      └─────────────────────────┘
                                    ┌─────────────────┐
                                    │  DynamoDB DAX   │
                                    │ (In-memory Read)│
                                    └─────────────────┘
```
