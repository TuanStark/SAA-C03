# NGÀY 11: SERVERLESS COMPUTE & ORCHESTRATION (LAMBDA, API GATEWAY, STEP FUNCTIONS)
*(Tuần 2: Security, Serverless & Integration - Trọng số Serverless & Modern Architectures)*

---

## 1. AWS LAMBDA CHUYÊN SÂU

AWS Lambda là dịch vụ tính toán không máy chủ (Serverless Compute) chạy mã nguồn theo mô hình hướng sự kiện (Event-driven) mà không cần cấp phát hay quản lý máy chủ.

```
                           ┌──────────────────────────────────────────┐
                           │          AWS LAMBDA ARCHITECTURE         │
                           └────────────────────┬─────────────────────┘
                                                │
                 ┌──────────────────────────────┼──────────────────────────────┐
                 ▼                              ▼                              ▼
      [ Synchronous (Đồng bộ) ]      [ Asynchronous (Bất đồng bộ) ]  [ Event Source Mapping (Poll) ]
      • API Gateway, ALB, SDK        • S3 Events, SNS, EventBridge   • SQS, Kinesis, DynamoDB Streams
      • Client chờ kết quả trả về    • Lambda tự đưa vào hàng đợi    • Lambda tự poll theo batch
      • Lỗi xử lý bởi Client         • Tự động Retry 2 lần (3 tổng)  • Gặp lỗi: Tự retry cả batch
      • Timeout tối đa 29s (API GW)  • Có thể gắn **DLQ / Destination** (Bisect on error, split batch)
```

### 1.1. Các giới hạn tài nguyên bắt buộc phải nhớ trong đề thi (Limits):
* **Thời gian thực thi tối đa (Execution Timeout)**: **15 phút (900 giây)**.
  - 👉 *Nếu tác vụ kéo dài hơn 15 phút*: Bắt buộc chuyển sang **AWS Fargate / ECS**, **AWS Batch**, hoặc chia nhỏ luồng bằng **AWS Step Functions**.
* **Bộ nhớ (Memory)**: Từ **128 MB đến 10,240 MB (10 GB)**.
  - CPU và băng thông mạng tỷ lệ thuận với Memory. Tại mức **1,769 MB**, Lambda được cấp trọn vẹn **1 vCPU**.
* **Bộ nhớ tạm thời (`/tmp`)**: Từ 512 MB lên tới **10 GB**.
* **Kích thước gói triển khai (Package Size)**: 50 MB (nén zip), 250 MB (giải nén), hoặc lên tới **10 GB nếu đóng gói dạng Container Image** (lưu trên Amazon ECR).

### 1.2. Concurrency & Khắc phục Cold Start
* **Reserved Concurrency**:
  - Dành riêng một lượng tài nguyên đồng thời cố định cho một hàm (ví dụ: cấp 200/1000 concurrency của account).
  - Ngăn không cho hàm này ngốn hết hạn mức của toàn bộ account, đồng thời giới hạn tốc độ để không làm quá tải database phía sau (Tránh lỗi connection pool).
* **Provisioned Concurrency**:
  - Khởi tạo sẵn một số lượng instances của hàm luôn ở trạng thái "ấm" (Warm).
  - **Triệt tiêu hoàn toàn độ trễ khởi động lạnh (Zero Cold Start)**, phản hồi trong vài mili-giây. Phù hợp cho API thanh toán hoặc hệ thống tài chính yêu cầu độ trễ cực thấp.
* **Tối ưu hóa mã nguồn (Execution Context Reuse)**:
  - Khởi tạo các kết nối Database client, AWS SDK client **BÊN NGOÀI hàm handler** để tái sử dụng qua các lần gọi tiếp theo, không tạo mới connection mỗi lần request.

### 1.3. Lambda trong VPC (VPC Networking)
* Mặc định Lambda chạy trong VPC bảo mật do AWS quản lý, có sẵn kết nối Internet công cộng nhưng **không thể truy cập tài nguyên Private trong VPC của bạn (như RDS, ElastiCache)**.
* **Khi gắn Lambda vào VPC của bạn**:
  - Cần cung cấp: **VPC ID, Subnet IDs, và Security Group**.
  - AWS sử dụng công nghệ Hyperplane ENI để chia sẻ card mạng (không gây delay cold start).
  - ⚠️ **BẪY ĐỀ THI SỐNG CÒN**:
    - Ngay khi bạn gắn Lambda vào một Private Subnet trong VPC, **Lambda sẽ MẤT TOÀN BỘ KẾT NỐI INTERNET MẶC ĐỊNH**.
    - Nếu Lambda vừa cần đọc RDS trong VPC, vừa cần gọi API bên ngoài Internet (hoặc gửi thông báo qua Slack/Stripe): **Subnet của Lambda phải có Route trỏ qua NAT Gateway đặt ở Public Subnet!**
    - Nếu Lambda chỉ cần đọc S3 hoặc DynamoDB: Tạo **VPC Gateway Endpoint** (hoàn toàn miễn phí, không cần NAT Gateway).

---

## 2. AMAZON API GATEWAY: REST vs HTTP vs WEBSOCKET

API Gateway là cổng tiếp nhận và quản lý API quy mô lớn, chịu tải hàng trăm ngàn request/giây.

| Tiêu chí | REST API | HTTP API | WebSocket API |
| :--- | :--- | :--- | :--- |
| **Giao thức** | HTTP, HTTPS (RESTful) | HTTP, HTTPS (RESTful) | **WebSocket (Hai chiều - Stateful)** |
| **Độ trễ & Chi phí** | Chi phí chuẩn, tính năng phong phú | **Rẻ hơn 70%, độ trễ cực thấp** | Tính phí theo số phút kết nối + tin nhắn |
| **Tính năng cao cấp** | • Tích hợp **AWS WAF**<br>• Caching API response<br>• Request/Response Transform (VTL)<br>• Usage Plans & API Keys | • CORS out-of-the-box<br>• Tích hợp OpenID Connect & OAuth 2.0<br>• Thích hợp làm proxy trực tiếp cho Lambda | Hỗ trợ chat real-time, bảng điện tử, multiplayer game |
| **Endpoint Types** | Edge-Optimized, Regional, Private | Regional | Regional |

### Các loại Endpoint của REST API:
1. **Edge-Optimized**: Tự động định tuyến qua mạng lưới **Amazon CloudFront Edge Locations** toàn cầu trước khi về API Gateway. Dành cho client phân tán toàn cầu.
2. **Regional**: Dành cho các client trong cùng một Region, hoặc khi bạn muốn tự đặt một CloudFront Distribution riêng biệt phía trước để tùy biến sâu.
3. **Private Endpoint**: **CHỈ CÓ THỂ TRUY CẬP TỪ BÊN TRONG VPC** thông qua một Interface VPC Endpoint (`aws.sapi`). Hoàn toàn cách ly khỏi Internet công cộng.

---

## 3. AWS STEP FUNCTIONS: SERVERLESS ORCHESTRATION

AWS Step Functions là dịch vụ điều phối luồng công việc (State Machine) trực quan, cho phép ghép nối nhiều dịch vụ AWS (Lambda, ECS, SQS, SNS, DynamoDB) thành một quy trình nghiệp vụ hoàn chỉnh.

```
                           ┌──────────────────────────────────────────┐
                           │          AWS STEP FUNCTIONS TYPES        │
                           └────────────────────┬─────────────────────┘
                                                │
                 ┌──────────────────────────────┴──────────────────────────────┐
                 ▼                                                             ▼
     [ Standard Workflows ]                                        [ Express Workflows ]
     • Thời gian chạy: **Lên tới 1 NĂM**                          • Thời gian chạy: **Tối đa 5 PHÚT**
     • Cơ chế thực thi: **Exactly-once**                          • Cơ chế thực thi: **At-least-once**
     • Tốc độ: Tối đa 2,000 executions/giây                       • Tốc độ: **Hơn 100,000 executions/giây**
     • Trực quan hóa: Debug từng bước trên Console                • Dùng cho: Xử lý dữ liệu IoT, streaming,
     • Dùng cho: Quy trình nghiệp vụ dài, phê duyệt con người,      chuyển đổi dữ liệu tốc độ cực cao
       xử lý đơn hàng, điều phối microservices phức tạp
```

### Các trạng thái cốt lõi trong State Machine (States):
* **Task State**: Thực hiện một công việc (gọi Lambda, chạy ECS Task, ghi DynamoDB).
* **Choice State**: Rẽ nhánh điều kiện logic (`if-else` dựa trên JSON input).
* **Parallel State**: Thực thi song song nhiều nhánh công việc độc lập cùng lúc.
* **Map State**: Lặp qua một danh sách các phần tử (Iterate) và xử lý đồng thời.
* **Wait State**: Tạm dừng luồng trong một khoảng thời gian (ví dụ: chờ 2 giờ hoặc chờ đến ngày chỉ định).
* **Error Handling**: Hỗ trợ khối `Retry` (tự động thử lại với Exponential Backoff) và `Catch` (chuyển sang bước xử lý lỗi khi thất bại).

---

## 4. BẢNG TỔNG KẾT QUYẾT ĐỊNH (SERVERLESS DECISION MATRIX)

| Tình huống thực tế trong đề thi | Giải pháp tối ưu | Lý do kỹ thuật |
| :--- | :--- | :--- |
| Xử lý backend nhanh gọn, serverless, không cần duy trì kết nối liên tục | **API Gateway + AWS Lambda** | Tự động scale từ 0 lên hàng ngàn requests, chỉ trả tiền khi chạy. |
| Ứng dụng chat real-time hoặc bảng báo giá chứng khoán cập nhật liên tục | **API Gateway WebSocket API** | Duy trì kết nối hai chiều (Full-Duplex) với client. |
| Tác vụ xử lý ảnh hoặc báo cáo kéo dài **hơn 15 phút** | **AWS Fargate / ECS** hoặc **AWS Batch** | Lambda bị giới hạn cứng tối đa 15 phút. |
| Điều phối quy trình thanh toán có bước chờ duyệt của con người (Human approval) hoặc kéo dài nhiều ngày | **Step Functions (Standard Workflow)** | Hỗ trợ lưu trạng thái lên tới 1 năm, có tích hợp Task Token callback. |
| Lambda trong Private Subnet cần kết nối database RDS nội bộ và gọi bên ngoài Internet | **Lambda gắn VPC + NAT Gateway trong Public Subnet** | Gắn VPC làm mất Internet mặc định, cần NAT GW để ra ngoài. |
| Lambda đọc dữ liệu S3/DynamoDB trong Private Subnet với chi phí rẻ nhất | **VPC Gateway Endpoint** | Hoàn toàn miễn phí, không tốn tiền NAT Gateway. |

---

## 5. LAB KIẾN TRÚC: ASYNCHRONOUS DECOUPLED API FOR LONG-RUNNING TASKS

### Tình huống Lab:
Một ứng dụng web cần tạo báo cáo PDF phân tích tài chính dung lượng lớn. Tác vụ tạo PDF mất từ **1 đến 3 phút**. Nếu client gọi trực tiếp qua API Gateway đồng bộ, kết nối sẽ bị lỗi **HTTP 504 Gateway Timeout** (vì API Gateway có giới hạn timeout cứng là **29 giây**).

```
                            [ NGƯỜI DÙNG / FRONTEND ]
                                       │
                                       │ (1) POST /reports (Yêu cầu tạo báo cáo)
                                       ▼
                       ┌───────────────────────────────┐
                       │   Amazon API Gateway (HTTP)   │
                       └───────────────┬───────────────┘
                                       │
                                       │ (2) Chuyển request
                                       ▼
                       ┌───────────────────────────────┐
                       │   AWS Lambda (Dispatcher)     │
                       └───────┬───────────────┬───────┘
                               │               │
      (3) Đẩy task vào hàng đợi│               │ (4) Trả về ngay lập tức:
                               ▼               │     **HTTP 202 Accepted**
               ┌───────────────────────────────┤     `{"reportId": "xyz123", "status": "PENDING"}`
               │    Amazon SQS Standard Queue  │
               └───────────────┬───────────────┘
                               │
                               │ (5) Pull task xử lý ngầm (1 - 3 phút)
                               ▼
               ┌───────────────────────────────┐
               │    AWS Lambda (Worker)        │
               │    (Timeout: 10 phút, 2GB RAM)│
               └───────────────┬───────────────┘
                               │
                               │ (6) Ghi file hoàn tất
                               ▼
               ┌───────────────────────────────┐
               │    Amazon S3 Bucket (Storage) │
               └───────────────────────────────┘
```

### Luồng xử lý phi đồng bộ chuẩn Enterprise:
1. Client gửi yêu cầu tạo báo cáo tới API Gateway.
2. Lambda Dispatcher nhận yêu cầu, sinh một mã `ReportID`, ghi task vào SQS queue, và **trả về ngay lập tức HTTP 202 Accepted** kèm `ReportID` cho client trong vòng chưa đầy 1 giây.
3. Client không bị treo kết nối và không bị timeout.
4. Lambda Worker ở phía sau rút message từ SQS và thực hiện render file PDF trong 3 phút, sau đó lưu file vào Amazon S3 và cập nhật trạng thái trong DynamoDB thành `COMPLETED`.
5. Client định kỳ kiểm tra trạng thái (`GET /reports/xyz123`) hoặc nhận thông báo qua WebSocket/Email khi file đã sẵn sàng để tải về.
