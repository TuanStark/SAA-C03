# NGÀY 10: MESSAGING, DECOUPLING & EVENT-DRIVEN (SQS, SNS, EVENTBRIDGE, KINESIS)
*(Tuần 2: Security, Serverless & Integration - Trọng số Resilient & Scalable Architectures)*

---

## 1. AMAZON SQS (SIMPLE QUEUE SERVICE) CHUYÊN SÂU

Amazon SQS là dịch vụ hàng đợi thông điệp (Message Queue) được quản lý hoàn toàn theo mô hình **Kéo (Pull-based)**: Producer đẩy message vào queue, Consumer chủ động poll message về để xử lý.

```
                  ┌────────────────────────────────────────────────────────┐
                  │                 AMAZON SQS QUEUE TYPES                 │
                  └───────────────────────────┬────────────────────────────┘
                                              │
             ┌────────────────────────────────┴────────────────────────────────┐
             ▼                                                                 ▼
    [ SQS Standard Queue ]                                            [ SQS FIFO Queue ]
    • Băng thông: **KHÔNG GIỚI HẠN** (Unlimited)                     • Băng thông: 300 msgs/s (3,000 với batch)
    • Thứ tự: **Best-effort ordering** (có thể đảo thứ tự)           • Thứ tự: **First-In, First-Out (Tuyệt đối)**
    • Phân phối: **At-least-once delivery** (có thể bị trùng lặp)     • Phân phối: **Exactly-once processing**
    • Dùng cho: Tải cực lớn, decoupling chung, worker chịu lỗi         • Tên queue bắt buộc kết thúc bằng: `.fifo`
```

### 1.1. Bốn cơ chế sống còn của SQS trong đề thi SAA-C03:

#### 1. Visibility Timeout (Thời gian ẩn thông điệp)
* Khi Consumer đọc một message, message đó **không bị xóa ngay** mà chuyển sang trạng thái tạm ẩn đối với các Consumer khác trong một khoảng thời gian (Mặc định: **30 giây**, tối đa: 12 giờ).
* Nếu Consumer xử lý xong: Gọi API `DeleteMessage` để xóa vĩnh viễn khỏi hàng đợi.
* ⚠️ **BẪY ĐỀ THI**:
  - *"Một tác vụ xử lý video mất 2 phút nhưng sau 30 giây message lại bị một Consumer khác lấy ra xử lý lần 2, dẫn đến việc render video bị trùng lặp."*
  - **Nguyên nhân**: Quá trình xử lý vượt quá thời gian Visibility Timeout mặc định (30s).
  - **Giải pháp**: 
    1. Tăng Visibility Timeout của Queue lên (ví dụ: 5 phút).
    2. Hoặc Consumer chủ động gọi API **`ChangeMessageVisibility`** để gia hạn thêm thời gian trong khi đang xử lý.

#### 2. Short Polling vs Long Polling
* **Short Polling** (`WaitTimeSeconds = 0`): Trả về kết quả ngay lập tức ngay cả khi queue rỗng $\rightarrow$ Tốn chi phí API requests khổng lồ.
* **Long Polling** (`WaitTimeSeconds = 1 đến 20 giây`): Giữ kết nối tối đa 20 giây để chờ có message mới đến là trả về ngay $\rightarrow$ **Giảm chi phí API requests và giảm độ trễ (latency)**.

#### 3. Dead Letter Queue (DLQ) & Redrive Policy
* Nếu một message bị lỗi khiến Consumer xử lý thất bại nhiều lần: Sau khi vượt ngưỡng **`maxReceiveCount`** (ví dụ: 5 lần), message độc hại (poison pill) sẽ tự động được chuyển sang **Dead Letter Queue (DLQ)**.
* Giúp ngăn chặn việc message lỗi làm nghẽn hàng đợi vĩnh viễn, cho phép kỹ sư phân tích log và debug nguyên nhân lỗi.

#### 4. FIFO Queue: Message Deduplication ID vs Message Group ID
* **Message Deduplication ID**: Dùng để chống trùng lặp trong cửa sổ **5 phút** (theo thuật toán SHA-256 nội dung hoặc chuỗi ID chỉ định).
* **Message Group ID**: 
  - Các messages có **CÙNG Group ID** sẽ được xử lý tuần tự nghiêm ngặt theo đúng thứ tự (Strictly Ordered).
  - Các messages có **KHÁC Group ID** có thể được **xử lý song song đồng thời bởi nhiều Consumer khác nhau**.

---

## 2. AMAZON SNS (SIMPLE NOTIFICATION SERVICE)

Khác với SQS (Pull-based), **Amazon SNS hoạt động theo mô hình Đẩy (Push-based: Pub/Sub)**:
- Một Topic (Chủ đề) có thể nhận thông điệp từ 1 Publisher và **bắn (Fan-out) đồng thời tới hàng ngàn Subscribers**.
- Các loại Subscribers được hỗ trợ: **Amazon SQS, AWS Lambda, Email, SMS, Mobile Push, HTTP/HTTPS endpoints**.

```
                                  [ DỊCH VỤ ĐẶT HÀNG (PUBLISHER) ]
                                                 │
                                                 ▼
                                     ┌───────────────────────┐
                                     │   Amazon SNS Topic    │
                                     │    "Order-Created"    │
                                     └───────────┬───────────┘
                                                 │
             ┌───────────────────────────────────┼───────────────────────────────────┐
             │ (Fan-out)                         │ (Fan-out)                         │ (Fan-out)
             ▼                                   ▼                                   ▼
    ┌─────────────────┐                 ┌─────────────────┐                 ┌─────────────────┐
    │ SQS: Xử lý kho  │                 │ SQS: Thanh toán │                 │ SQS: Gửi email  │
    └────────┬────────┘                 └────────┬────────┘                 └────────┬────────┘
             ▼                                   ▼                                   ▼
    [ Worker Inventory ]                 [ Worker Billing ]                  [ Worker Email ]
```

### Kiến trúc kinh điển: SNS + SQS Fan-Out Pattern
* Khi một sự kiện xảy ra cần nhiều hệ thống độc lập cùng xử lý:
  - Publisher gửi event vào **1 SNS Topic duy nhất**.
  - Mỗi hệ thống backend gắn một **SQS Queue riêng làm subscriber** của Topic đó.
  - **Lợi ích**: Hoàn toàn tách rời (Decoupled), mỗi hệ thống tự xử lý theo tốc độ riêng của mình, nếu một worker sập thì message vẫn nằm an toàn trong SQS của nó mà không làm ảnh hưởng các hệ thống khác.
* **Message Filtering Policy**: Gán bộ lọc JSON trên từng SQS subscription để chỉ nhận đúng loại message mong muốn (ví dụ: chỉ nhận đơn hàng `"order_type": "vip"`).

---

## 3. AMAZON EVENTBRIDGE (TIỀN THÂN LÀ CLOUDWATCH EVENTS)

Amazon EventBridge là một **Serverless Event Bus** mạnh mẽ dùng để kết nối các ứng dụng thông qua dữ liệu sự kiện:

| Tiêu chí | Amazon SNS | Amazon EventBridge |
| :--- | :--- | :--- |
| **Mô hình** | Pub/Sub Topic thuần túy (Đẩy thông báo) | Event Bus trung tâm (Định tuyến sự kiện dựa trên nội dung) |
| **Nguồn sự kiện (Sources)** | Ứng dụng tự đẩy message qua SDK | **Hơn 100 dịch vụ AWS** (S3, EC2, IAM...), SaaS đối tác (DataDog, Shopify, Auth0), và custom app |
| **Bộ lọc sự kiện** | JSON Attribute Filtering đơn giản | **Content-based Filtering nâng cao** (kiểm tra trường sâu trong JSON payload) |
| **Khả năng tích hợp** | Giới hạn (SQS, Lambda, Email, HTTP) | Hỗ trợ **hơn 20 AWS targets** (Step Functions, Kinesis, ECS, Batch...) |
| **Tính năng cao cấp** | FIFO Topic, SMS/Mobile Push | **Schema Registry**, **EventBridge Pipes**, **EventBridge Scheduler** (Cron/Rate với timezone) |

> 💡 **QUY TẮC PHÂN BIỆT ĐỀ THI**:
> - *"Cần gửi thông báo dạng Push đồng thời tới nhiều SQS queues, SMS và Email"* $\rightarrow$ **Amazon SNS**.
> - *"Cần lắng nghe sự kiện thay đổi trạng thái của tài nguyên AWS (EC2 Stop, S3 Upload, IAM policy change) hoặc sự kiện từ SaaS đối tác rồi kích hoạt Step Functions/Lambda"* $\rightarrow$ **Amazon EventBridge**.

---

## 4. AMAZON KINESIS: NỀN TẢNG STREAMING DATA REAL-TIME

Khi đề bài xuất hiện từ khóa **"Streaming Data"**, **"Real-time Analytics"**, **"Clickstream"**, hoặc **"IoT Telemetry"**, đáp án 90% sẽ nằm trong hệ sinh thái Kinesis:

```
                                  ┌───────────────────────────────────┐
                                  │      HỆ SINH THÁI KINESIS         │
                                  └─────────────────┬─────────────────┘
                                                    │
                 ┌──────────────────────────────────┼──────────────────────────────────┐
                 ▼                                  ▼                                  ▼
   [ Kinesis Data Streams (KDS) ]      [ Kinesis Data Firehose ]          [ Kinesis Data Analytics ]
   • Băng thông tùy chỉnh qua Shards   • **Near real-time (60s - 900s)**  • Phân tích dữ liệu stream
   • Khả năng lưu trữ: 1 đến 365 ngày  • Tự động load dữ liệu vào:          bằng chuẩn **SQL hoặc Apache
   • Cho phép **ĐỌC LẠI (Replay)**       S3, Redshift, OpenSearch, Splunk   Flink** thời gian thực.
   • Xử lý bằng Lambda hoặc KCL app    • **KHÔNG lưu trữ, KHÔNG replay**  • Phát hiện bất thường (Anomaly).
```

### So sánh chuyên sâu Kinesis Data Streams vs Firehose:

| Đặc tính | Kinesis Data Streams (KDS) | Kinesis Data Firehose |
| :--- | :--- | :--- |
| **Mức độ quản lý** | Bạn quản lý Shards (Provisioned) hoặc On-Demand. | **Fully Managed Serverless 100%**, tự động scale. |
| **Độ trễ (Latency)** | **Real-time (< 200 milliseconds)**. | **Near real-time** (chờ đệm từ **60 đến 900 giây** hoặc đạt 1–128 MB). |
| **Khả năng Replay dữ liệu** | **CÓ**: Lưu giữ dữ liệu từ 24h đến 365 ngày. Nhiều consumer có thể đọc lại. | **KHÔNG**: Đẩy thẳng vào đích đến, không có bộ nhớ đệm lưu trữ dài hạn. |
| **Biến đổi dữ liệu (Transform)** | Tự viết mã nguồn Consumer (Lambda / EC2). | Tích hợp trực tiếp **AWS Lambda** để chuyển đổi định dạng (JSON sang Parquet/ORC). |

---

## 5. BẢNG TỔNG KẾT QUYẾT ĐỊNH (INTEGRATION DECISION MATRIX)

| Tình huống thực tế trong đề thi | Dịch vụ tối ưu | Lý do kỹ thuật |
| :--- | :--- | :--- |
| Đệm tải bất đồng bộ (Buffer), chống nghẽn cho backend worker, hỗ trợ retry | **Amazon SQS Standard** | Lưu message tối đa 14 ngày, scale không giới hạn. |
| Giao dịch ngân hàng hoặc đặt vé máy bay yêu cầu **đúng thứ tự và không trùng lặp** | **Amazon SQS FIFO** | Exactly-once delivery + First-In First-Out. |
| Một sự kiện xảy ra cần gửi tới nhiều hệ thống backend khác nhau cùng lúc | **SNS + SQS Fan-Out** | Tách rời publisher và các consumer độc lập. |
| Tự động thực hiện hành động khi EC2 đổi trạng thái hoặc S3 upload tệp mới | **Amazon EventBridge** | Bộ lọc sự kiện native từ toàn bộ AWS services. |
| Thu thập hàng triệu bản ghi clickstream/giây và nạp trực tiếp vào Amazon Redshift/S3 | **Kinesis Data Firehose** | Managed, tự động gom lô (batching) và chuyển đổi Parquet. |
| Xử lý phân tích video, telemetry xe tự hành thời gian thực độ trễ < 200ms và cần đọc lại | **Kinesis Data Streams** | Hỗ trợ lưu trữ theo Shard và replay dữ liệu. |

---

## 6. LAB KIẾN TRÚC: HỆ THỐNG XỬ LÝ ĐƠN HÀNG CHỊU TẢI ĐỘT BIẾN (BURST PROTECTION)

```
                            [ NGƯỜI DÙNG ĐẶT HÀNG (FLASH SALE) ]
                                             │
                                             ▼
                             ┌───────────────────────────────┐
                             │    API Gateway + AWS Lambda   │
                             └───────────────┬───────────────┘
                                             │
                                             ▼ (Publish Order Created)
                             ┌───────────────────────────────┐
                             │       Amazon SNS Topic        │
                             └───────┬───────────────┬───────┘
                                     │ (Fan-Out)     │ (Fan-Out)
             ┌───────────────────────┘               └───────────────────────┐
             ▼                                                               ▼
    ┌─────────────────────────────────┐                             ┌─────────────────────────────────┐
    │ SQS FIFO Queue: "Payments.fifo" │                             │ SQS Standard: "Analytics-Queue" │
    │ • MessageGroupId = CustomerID   │                             │ • Băng thông không giới hạn     │
    │ • Đảm bảo thanh toán đúng thứ tự│                             │ • Xử lý phân tích hành vi       │
    └────────┬────────────────────────┘                             └────────┬────────────────────────┘
             │                                                               │
             ▼ (Pull messages)                                               ▼ (Batch read)
    ┌─────────────────────────────────┐                             ┌─────────────────────────────────┐
    │ Auto Scaling Group (EC2)        │                             │ Kinesis Data Firehose           │
    │ • Scale dựa trên CloudWatch:    │                             │ • Gom lô chuyển đổi sang Parquet│
    │   `ApproximateNumberOfMessages` │                             │ • Lưu trữ lâu dài trên S3       │
    └────────┬────────────────────────┘                             └─────────────────────────────────┘
             │ (Nếu xử lý lỗi > 5 lần)
             ▼
    ┌─────────────────────────────────┐
    │ SQS Dead Letter Queue (DLQ)     │
    │ • Kích hoạt CloudWatch Alarm    │
    │ • Bắn thông báo SNS cho On-call │
    └─────────────────────────────────┘
```
