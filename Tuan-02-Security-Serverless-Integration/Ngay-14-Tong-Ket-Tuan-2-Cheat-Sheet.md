# NGÀY 14: TỔNG KẾT TUẦN 2 - MASTER CHEAT SHEET & KIẾN TRÚC SERVERLESS TOÀN DIỆN
*(Security, Serverless, Integration, Monitoring & Disaster Recovery)*

---

## 1. BẢNG TRA CỨU NHANH CÁC CẶP DỊCH VỤ TUẦN 2 (WEEK 2 EXAM SHOWDOWN)

### 1.1. Security, IAM & Governance (Domain 1: 30%)
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **Permissions Boundary vs SCP** | • **Permissions Boundary**: Gán vào **IAM User/Role**, đặt mức trần quyền hạn tối đa để ngăn **leo thang đặc quyền (Privilege Escalation)** khi ủy quyền tạo Role.<br>• **SCP (Service Control Policy)**: Gán vào **OU/Account** trong Organizations, áp dụng lên cả **Root User của Member Account**, nhưng **KHÔNG áp dụng lên Management Account**. |
| **Cross-Account: Role vs External ID** | • **IAM Role (AssumeRole)**: Cung cấp Temporary Credentials qua STS, không dùng Access Key dài hạn.<br>• **External ID**: Tham số bí mật **bắt buộc** trong Trust Policy khi cấp quyền cho **bên thứ ba (SaaS Vendor)** nhằm ngăn chặn lỗ hổng **The Confused Deputy Problem**. |
| **Secrets Manager vs SSM Parameter Store** | • **Secrets Manager**: Tự động xoay vòng mật khẩu (**Automatic Rotation**) có sẵn cho RDS/Aurora, sinh mật khẩu ngẫu nhiên, có phí.<br>• **SSM Parameter Store**: Quản lý cấu hình dạng cây (`/app/prod/db`), bản Standard **hoàn toàn miễn phí**, không có sẵn tính năng tự xoay vòng mật khẩu. |
| **AWS KMS: Envelope Encryption** | • KMS chỉ mã hóa trực tiếp tệp $\le$ **4 KB** (`kms:Encrypt`).<br>• Với tệp lớn: Dùng `kms:GenerateDataKey` $\rightarrow$ dùng **Plaintext Data Key** mã hóa file $\rightarrow$ xóa RAM $\rightarrow$ lưu **Encrypted Data Key** cạnh file. |
| **WAF vs Shield vs Macie vs GuardDuty** | • **WAF (Layer 7)**: Chặn SQLi, XSS, và **Rate-based rules** (chặn IP gửi > N request/5 phút).<br>• **Shield (Layer 3/4)**: Chống DDoS (Standard miễn phí, Advanced $3,000/tháng + đội phản ứng nhanh SRT).<br>• **Macie**: Dùng ML quét tìm **dữ liệu nhạy cảm PII, thẻ tín dụng** trong **Amazon S3**.<br>• **GuardDuty**: Dùng ML phát hiện **hành vi bất thường, tài khoản bị xâm nhập** (phân tích CloudTrail, VPC Flow Logs, DNS Logs). |

---

### 1.2. Messaging, Decoupling & Streaming (Domain 2 & 3)
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **SQS Standard vs SQS FIFO** | • **Standard**: Băng thông không giới hạn, At-least-once, Best-effort ordering.<br>• **FIFO**: 300 - 3,000 msgs/s, Exactly-once, First-In First-Out.<br>• **MessageGroupId**: Giữ đúng thứ tự trong cùng group, nhưng **cho phép xử lý song song giữa các group khác nhau**.<br>• **MessageDeduplicationId**: Chống gửi trùng lặp trong 5 phút. |
| **SQS Visibility Timeout** | • Mặc định **30 giây**. Nếu tác vụ chạy lâu (1 - 5 phút): Tăng Visibility Timeout hoặc gọi API **`ChangeMessageVisibility`** để gia hạn, tránh bị worker khác lấy xử lý lần 2. |
| **SNS vs EventBridge** | • **SNS**: Pub/Sub đẩy thông báo tới hàng triệu subscriber (Fan-Out tới nhiều SQS queues, SMS, Email).<br>• **EventBridge**: Serverless Event Bus, lọc sự kiện chuyên sâu theo nội dung JSON (**Content-based Filtering**), nhận event từ 100+ AWS services và SaaS bên thứ ba. |
| **Kinesis Data Streams vs Firehose** | • **Data Streams (KDS)**: Real-time (<200ms), quản lý Shards, lưu trữ 1-365 ngày, **hỗ trợ Replay dữ liệu**.<br>• **Firehose**: Fully managed serverless, **Near real-time (60-900s)**, không replay, tự động nạp thẳng vào S3/Redshift và chuyển đổi định dạng Parquet. |

---

### 1.3. Serverless Compute & Orchestration
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **Lambda: Reserved vs Provisioned Concurrency** | • **Reserved**: Giới hạn trần để bảo vệ Database phía sau.<br>• **Provisioned**: Khởi tạo sẵn instance luôn "ấm", **triệt tiêu 100% Cold Start** cho hệ thống cần phản hồi siêu tốc. |
| **Lambda trong VPC** | • Gắn Lambda vào Private Subnet làm **mất kết nối Internet mặc định**.<br>• Muốn vừa đọc RDS trong VPC vừa gọi Internet/Stripe API: Subnet phải có Route trỏ qua **NAT Gateway ở Public Subnet**.<br>• Muốn đọc S3/DynamoDB miễn phí: Dùng **VPC Gateway Endpoint**. |
| **API Gateway: REST vs HTTP vs WebSocket** | • **REST**: Đầy đủ tính năng (WAF, Caching, Usage Plans, API Keys). Timeout tối đa **29 giây**.<br>• **HTTP**: Rẻ hơn 70%, độ trễ cực thấp, OIDC/OAuth 2.0.<br>• **WebSocket**: Kết nối 2 chiều liên tục (Full-duplex) cho Real-time Chat, Live dashboard. |
| **Step Functions: Standard vs Express** | • **Standard**: Chạy tối đa **1 NĂM**, exactly-once, audit log từng bước, hỗ trợ Task Token cho **phê duyệt con người (Human approval)**.<br>• **Express**: Chạy tối đa **5 PHÚT**, at-least-once, tốc độ cực lớn (>100,000 executions/s). |

---

### 1.4. Monitoring, Auditing & Disaster Recovery
| Cặp so sánh | Điểm khác biệt mấu chốt để chọn trong đề thi |
| :--- | :--- |
| **CloudWatch vs CloudTrail vs Config** | • **CloudWatch**: Giám sát hiệu năng (RAM cần Unified Agent, Metric Filters đếm lỗi HTTP 500).<br>• **CloudTrail**: Kiểm toán API (Data Events của S3 mặc định tắt, cần bật để biết ai tải file).<br>• **AWS Config**: Quản trị tuân thủ, phát hiện vi phạm và tự động sửa qua **SSM Automation**. |
| **SSM Session Manager** | • Mở shell vào EC2 an toàn trực tiếp từ Console/CLI **không cần mở port 22 (SSH), không cần Public IP, không cần Bastion Host**. |
| **4 Chiến lược DR (RTO / RPO)** | 1. **Backup & Restore**: Rẻ nhất ($), RTO/RPO tính bằng Giờ/Ngày.<br>2. **Pilot Light**: RPO vài giây, compute tắt hoặc ASG `Desired=0` ($0 compute khi bình thường), RTO 10-30 phút.<br>3. **Warm Standby**: Một bản sao thu nhỏ chạy 24/7 ở Region phụ, RTO < 10 phút.<br>4. **Multi-Site Active/Active**: Chạy 100% công suất đa vùng 24/7, RTO/RPO gần bằng 0 (Zero Downtime). |
| **AWS Backup Vault Lock** | • Chế độ **Compliance Mode**: Khóa bất biến WORM, **ngay cả tài khoản Root Account cũng không thể xóa bản sao lưu** trước ngày hết hạn (chống tuyệt đối Ransomware). |

---

## 2. LAB KIẾN TRÚC TOÀN DIỆN TUẦN 2: EVENT-DRIVEN SERVERLESS ENTERPRISE

```
                            [ NGƯỜI DÙNG TOÀN CẦU ]
                                       │
                                       ▼ (HTTPS)
    ┌────────────────────────── TẦNG AN NINH BIÊN (EDGE) ──────────────────────────┐
    │   Amazon CloudFront + AWS WAF (Rate Limiting, SQLi) + AWS Shield Standard    │
    └──────────────────────────────────┬───────────────────────────────────────────┘
                                       │
                                       ▼
    ┌─────────────────────────── TẦNG API GATEWAY ──────────────────────────────────┐
    │   Amazon API Gateway (REST API / HTTP API)                                   │
    │   • Chứng chỉ SSL quản lý bởi ACM                                            │
    │   • Tiếp nhận request đặt hàng $\rightarrow$ Kích hoạt Lambda Dispatcher     │
    └──────────────────────────────────┬───────────────────────────────────────────┘
                                       │
                                       │ (Trả về ngay: HTTP 202 Accepted + TaskID)
                                       ▼
    ┌────────────────────────── TẦNG ĐIỀU PHỐI & BUFFER ───────────────────────────┐
    │   Amazon SNS Topic ("OrderEvents")                                           │
    │   • Fan-Out sang: SQS Payments (FIFO) + SQS Inventory + EventBridge          │
    └──────────────┬───────────────────────────────────────────────┬───────────────┘
                   │                                               │
                   ▼ (Batching pull)                               ▼ (Event-driven)
    ┌───────────────────────────────┐               ┌───────────────────────────────┐
    │ AWS Lambda Worker (VPC)       │               │ AWS Step Functions (Standard) │
    │ • Provisioned Concurrency     │               │ • Điều phối luồng giao hàng   │
    │ • Lấy DB pass từ Secrets Mgr  │               │ • Chờ phê duyệt (Task Token)  │
    └──────────────┬────────────────┘               └──────────────┬────────────────┘
                   │                                               │
                   ▼ (Private subnet qua NAT GW)                   ▼
    ┌───────────────────────────────┐               ┌───────────────────────────────┐
    │ Amazon Aurora Serverless v2   │               │ Amazon DynamoDB               │
    │ (ACID Transactions lưu Order) │               │ (Lưu trạng thái giao hàng)    │
    └──────────────┬────────────────┘               └──────────────┬────────────────┘
                   │                                               │
                   ▼                                               ▼
    ┌──────────────────────────────────────────────────────────────────────────────┐
    │                     TẦNG QUẢN TRỊ, GIÁM SÁT & SAO LƯU                        │
    │ • CloudWatch Logs & Metrics Filters (Đếm lỗi HTTP 500)                       │
    │ • AWS CloudTrail (Ghi vết Data Events đọc/ghi dữ liệu)                       │
    │ • AWS Config + SSM Automation (Tự động đóng quyền S3/EBS nếu vi phạm)        │
    │ • AWS Backup (Vault Lock Compliance Mode chống Ransomware)                   │
    └──────────────────────────────────────────────────────────────────────────────┘
```
