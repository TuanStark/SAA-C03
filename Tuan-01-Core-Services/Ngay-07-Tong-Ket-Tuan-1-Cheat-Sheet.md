# NGÀY 7: TỔNG KẾT TUẦN 1 - MASTER CHEAT SHEET & KIẾN TRÚC TOÀN DIỆN
*(Core Infrastructure: Networking, Compute, Storage, Database)*

---

## 1. BẢNG TRA CỨU NHANH CÁC CẶP DỊCH VỤ DỄ NHẦM LẪN (EXAM SHOWDOWN)

### 1.1. Networking & Hybrid Connectivity
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **SG vs NACL** | • **SG**: Instance level, Stateful, chỉ có ALLOW, đánh giá tất cả rules.<br>• **NACL**: Subnet level, Stateless, có cả ALLOW & **DENY** (dùng để chặn IP cụ thể), đánh giá theo số thứ tự (Rule numbers). Cần mở Ephemeral ports (1024-65535) cho outbound. |
| **Gateway Endpoint vs Interface Endpoint** | • **Gateway**: Chỉ S3 & DynamoDB, cấu hình qua Route Table, **HOÀN TOÀN MIỄN PHÍ**, không hỗ trợ on-premises qua DX/VPN.<br>• **Interface (PrivateLink)**: Cho hầu hết services (bao gồm S3), tạo ENI có Private IP, tính phí theo giờ + GB, **hỗ trợ truy cập từ On-Premises qua Direct Connect/VPN**. |
| **VPC Peering vs Transit Gateway** | • **Peering**: 1:1, không transitive, không tốn phí duy trì, phù hợp ít VPC.<br>• **Transit Gateway**: Hub-and-Spoke, hỗ trợ **Transitive routing**, kết nối hàng ngàn VPC và On-premises qua VPN/DX, hỗ trợ Multicast và Route Table segmentation (Dev vs Prod). |
| **Direct Connect vs Site-to-Site VPN** | • **Direct Connect**: Kênh vật lý riêng, băng thông 1-100 Gbps, độ trễ thấp nhất quán, **KHÔNG mã hóa mặc định** (cần IPsec VPN over DX hoặc MACsec nếu yêu cầu mã hóa).<br>• **VPN**: Chạy qua Public Internet, mã hóa IPsec, tối đa 1.25 Gbps/tunnel, triển khai nhanh. |

---

### 1.2. Compute & Storage
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **Savings Plans vs Reserved Instances (RI)** | • **Compute Savings Plans**: Linh hoạt nhất (áp dụng cho EC2 mọi family/region/OS + Fargate + Lambda).<br>• **EC2 Instance Savings Plans / Standard RI**: Giảm giá sâu hơn nhưng khóa vào Instance family cụ thể trong 1 Region. |
| **Instance Store vs EBS** | • **Instance Store**: Ổ cứng vật lý gắn trực tiếp vào host, IOPS cực cao (hàng trăm ngàn), dữ liệu tạm thời (**MẤT khi Stop/Terminate**, giữ khi Reboot).<br>• **EBS**: Network drive, độc lập vòng đời với EC2, AZ-locked, hỗ trợ Snapshot. |
| **EBS gp3 vs io2 Block Express** | • **gp3**: General purpose, điều chỉnh độc lập IOPS (tới 16,000) và Throughput mà không cần tăng dung lượng đĩa.<br>• **io2 Block Express**: Sub-millisecond latency, tới 256,000 IOPS, 99.999% durability, hỗ trợ **Multi-Attach (trong cùng 1 AZ)**. |
| **EFS vs EBS Multi-Attach vs FSx** | • **EFS**: Hệ thống file chia sẻ Linux chuẩn POSIX/NFS, **Multi-AZ** (hàng trăm EC2 ở nhiều AZ mount đồng thời).<br>• **EBS Multi-Attach**: Gắn 1 volume vào nhiều EC2 nhưng **CHỈ TRONG CÙNG 1 AZ**.<br>• **FSx for Windows**: File share dùng giao thức SMB, tích hợp Active Directory, hỗ trợ Windows.<br>• **FSx for Lustre**: Tệp phân tán siêu tốc cho HPC/Machine Learning, tích hợp trực tiếp 2 chiều với S3. |
| **Placement Groups** | • **Cluster**: Cùng 1 rack trong 1 AZ $\rightarrow$ Low latency, High throughput (HPC).<br>• **Spread**: Mỗi instance 1 rack riêng biệt, tối đa 7 instances/AZ $\rightarrow$ HA tối đa.<br>• **Partition**: Chia thành các phân vùng rack, tối đa 7 partitions/AZ $\rightarrow$ Hadoop, Kafka, Cassandra. |

---

### 1.3. Load Balancing & Traffic Routing
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **ALB vs NLB** | • **ALB**: Layer 7 (HTTP/HTTPS), định tuyến theo Path/Host/Header, tích hợp WAF, **KHÔNG CÓ Static IP**.<br>• **NLB**: Layer 4 (TCP/UDP), hàng triệu RPS, độ trễ cực thấp, **CÓ Static IP / Elastic IP** cho mỗi AZ, giữ nguyên IP nguồn client. |
| **Global Accelerator vs CloudFront** | • **CloudFront**: CDN Layer 7, **CÓ CACHING**, tối ưu cho web tĩnh/động, hình ảnh, video.<br>• **Global Accelerator**: Bộ gia tốc mạng Layer 4, **KHÔNG CACHING**, cấp 2 Static Anycast IPs, chạy trên cáp quang riêng của AWS, tối ưu cho game UDP, IoT, và failover đa vùng < 1 phút. |
| **Route 53 CNAME vs ALIAS** | • **CNAME**: Hostname sang Hostname, **KHÔNG DÙNG ĐƯỢC cho Apex Domain (`example.com`)**.<br>• **ALIAS**: Đặc quyền của AWS, trỏ được Apex Domain sang tài nguyên AWS (ALB, CloudFront...), tự cập nhật IP và **miễn phí truy vấn DNS**. |
| **Route 53 Geolocation vs Geoproximity** | • **Geolocation**: Định tuyến theo vị trí pháp lý/quốc gia của User (tuân thủ luật pháp/GDPR).<br>• **Geoproximity**: Định tuyến theo khoảng cách vật lý + tham số **Bias** để mở rộng/thu hẹp vùng. |

---

### 1.4. Amazon S3 & Data Protection
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **S3 Storage Classes** | • **Standard-IA**: Dữ liệu ít dùng nhưng khi cần phải có ngay lập tức (phí retrieval, tối thiểu 30 ngày).<br>• **Intelligent-Tiering**: Mẫu truy cập không xác định hoặc thay đổi liên tục, không tốn phí retrieval.<br>• **Glacier Flexible**: Lưu trữ 3-5h (Expedited 1-5m, Bulk 5-12h).<br>• **Glacier Deep Archive**: Rẻ nhất toàn AWS, lưu trữ 7-10 năm (Standard 12h, Bulk 48h). |
| **S3 Object Lock: Compliance vs Governance** | • **Compliance Mode**: Tuyệt đối không ai (kể cả AWS Root Account) có thể xóa hay rút ngắn thời hạn retention.<br>• **Governance Mode**: Có thể được bypass bởi người dùng có quyền `s3:BypassGovernanceRetention`. |
| **SSE-S3 vs SSE-KMS vs SSE-C** | • **SSE-S3**: AWS quản lý khóa AES-256 hoàn toàn (mặc định cho mọi bucket).<br>• **SSE-KMS**: Khóa do KMS quản lý, ghi vết audit log trên CloudTrail, có nguy cơ KMS Quota Throttling $\rightarrow$ bật **S3 Bucket Keys** để giảm 99% request KMS.<br>• **SSE-C**: Khách hàng tự giữ và gửi key qua HTTPS mỗi request. |

---

### 1.5. Database Solutions
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **RDS Multi-AZ vs Read Replica** | • **Multi-AZ**: Phục vụ **High Availability & Disaster Recovery** (Đồng bộ, Standby passive KHÔNG đọc được, 1 endpoint tự động failover).<br>• **Read Replica**: Phục vụ **Scaling lưu lượng đọc** (Bất đồng bộ, Active phục vụ đọc, mỗi replica có endpoint riêng, hỗ trợ Cross-Region). |
| **Aurora Architecture** | • Storage tách rời: 6 bản sao trên 3 AZs (quorums 4/6 ghi, 3/6 đọc), tự mở rộng tới 128 TiB.<br>• **Cluster Endpoint**: Luôn trỏ vào Primary Writer.<br>• **Reader Endpoint**: Tự động cân bằng tải trên toàn bộ Replicas.<br>• **Global Database**: Sao chép liên vùng độ trễ < 1s ở tầng storage, RTO < 1 min, RPO < 1s. |
| **DynamoDB: DAX vs Streams vs Global Tables** | • **DAX**: In-memory cache kéo độ trễ đọc từ milliseconds xuống **MICROSECONDS**.<br>• **Streams**: Changelog 24h kích hoạt AWS Lambda (Event-driven).<br>• **Global Tables**: **Active-Active đa vùng**, đọc và ghi độ trễ thấp trên toàn cầu. |
| **ElastiCache Redis vs Memcached** | • **Redis**: Hỗ trợ Sorted Sets (cho Leaderboard), Multi-AZ Auto-Failover, Session Store, lưu trữ đĩa (Persistence).<br>• **Memcached**: Thuần in-memory, Multi-threaded, không có failover. |

---

## 2. LAB KIẾN TRÚC TOÀN DIỆN TUẦN 1: ENTERPRISE 3-TIER HIGH AVAILABILITY ARCHITECTURE

```
                                  [ NGƯỜI DÙNG TOÀN CẦU ]
                                             │
                                     ┌───────▼───────┐
                                     │   Route 53    │ (Geolocation / Latency / ALIAS Record)
                                     └───────┬───────┘
                                             │
                             ┌───────────────▼───────────────┐
                             │       Amazon CloudFront       │ (CDN Caching + AWS WAF)
                             └───────────────┬───────────────┘
                                             │ (HTTPS)
                                     ┌───────▼───────┐
                                     │      IGW      │
                                     └───────┬───────┘
                                             │
┌──────────────────────────── VPC 10.0.0.0/16 ───────────────────────────┐
│                                                                        │
│   ┌──────────── Availability Zone A ──────────┐  ┌──────────── Availability Zone B ──────────┐  │
│   │                                           │  │                                           │  │
│   │  [Public Subnet 1 - 10.0.1.0/24]          │  │  [Public Subnet 2 - 10.0.2.0/24]          │  │
│   │  ┌──────────────────┐ ┌────────────────┐  │  │  ┌──────────────────┐ ┌────────────────┐  │  │
│   │  │   ALB Node A     │ │  NAT Gateway A │  │  │  │   ALB Node B     │ │  NAT Gateway B │  │  │
│   │  └─────────┬────────┘ └────────▲───────┘  │  │  └─────────┬────────┘ └────────▲───────┘  │  │
│   │            │                   │          │  │            │                   │          │  │
│   │            │                   │ (egress) │  │            │                   │ (egress) │  │
│   │            ▼                   │          │  │            ▼                   │          │  │
│   │  [Private App Subnet 1 - 10.0.10.0/24]   │  │  [Private App Subnet 2 - 10.0.20.0/24]   │  │
│   │  ┌─────────────────────────────────────┐  │  │  ┌─────────────────────────────────────┐  │  │
│   │  │ Auto Scaling Group - EC2 App Tier   │  │  │  │ Auto Scaling Group - EC2 App Tier   │  │  │
│   │  │ • Target Tracking: CPU 50%          │  │  │  │ • Mixed Instances: Base On-Demand   │  │  │
│   │  │ • Mount: Amazon EFS (Multi-AZ)      │  │  │  │   + Scale-out 80% Spot              │  │  │
│   │  └─────────┬───────────────────┬───────┘  │  │  └─────────┬───────────────────┬───────┘  │  │
│   │            │                   │          │  │            │                   │          │  │
│   │            │ (cache)           │ (read/write)│            │ (cache)           │ (read/write)│
│   │            ▼                   ▼          │  │            ▼                   ▼          │  │
│   │  ┌──────────────────┐ ┌────────────────┐  │  │  ┌──────────────────┐ ┌────────────────┐  │  │
│   │  │ ElastiCache Redis│ │ Aurora Writer  ├──┼──┼─►│ ElastiCache Standby│ Aurora Reader   │  │  │
│   │  │ (Primary Node)   │ │ (Primary DB)   │  │  │  │ (Replica Node)   │ (Auto-Scaling)   │  │  │
│   │  └──────────────────┘ └────────┬───────┘  │  │  └──────────────────┘ └────────▲───────┘  │  │
│   │                                │          │  │                                 │         │  │
│   │                                └──────────┼──┼─────────────────────────────────┘         │  │
│   │                                           │  │                                           │  │
│   │  [Private DB / Storage Subnets]           │  │  [Private DB / Storage Subnets]           │  │
│   │  ┌─────────────────────────────────────┐  │  │  ┌─────────────────────────────────────┐  │  │
│   │  │ S3 Gateway Endpoint (FREE)          ├──┼──┼──┤ DynamoDB Gateway Endpoint (FREE)     │  │  │
│   │  └─────────────────────────────────────┘  │  │  └─────────────────────────────────────┘  │  │
│   └───────────────────────────────────────────┘  └───────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```
