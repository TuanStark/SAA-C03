# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 10: MESSAGING & DECOUPLING

---

### CÂU 1 (Mức độ: Trung bình - Xử lý sự cố SQS Visibility Timeout)
Một công ty xử lý đồ họa sử dụng một hàng đợi Amazon SQS Standard để điều phối các tác vụ render video cho một nhóm EC2 worker. Mỗi tác vụ render video có dung lượng lớn thường mất trung bình từ **3 đến 5 phút** để hoàn tất. 
Tuy nhiên, trong quá trình chạy thử nghiệm, nhóm phát triển phát hiện ra rằng sau khi một worker nhận một message và bắt đầu render được khoảng 30 giây, một worker khác lại nhận được đúng message đó và bắt đầu render lại từ đầu, dẫn đến việc trùng lặp công việc và lãng phí tài nguyên CPU.

Hành động nào sau đây sẽ giải quyết triệt để vấn đề này với **ít sự thay đổi kiến trúc nhất**?

- **A.** Chuyển đổi hàng đợi từ SQS Standard sang SQS FIFO.
- **B.** Tăng giá trị **Visibility Timeout** của SQS Queue lên mức phù hợp (ví dụ: 6 phút), hoặc cho phép worker liên tục gọi API **`ChangeMessageVisibility`** để gia hạn thời gian trong khi tác vụ render đang diễn ra.
- **C.** Giảm thời gian lưu trữ message (Message Retention Period) xuống dưới 1 phút.
- **D.** Kích hoạt tính năng Long Polling với `WaitTimeSeconds` đặt thành 20 giây.

---

### CÂU 2 (Mức độ: Nâng cao - SQS FIFO Parallel Processing & Message Group ID)
Một sàn giao dịch chứng khoán trực tuyến cần xử lý các lệnh mua và bán cổ phiếu của hàng ngàn khách hàng khác nhau thông qua một hàng đợi **Amazon SQS FIFO**.
Yêu cầu hệ thống:
1. Các lệnh giao dịch của **cùng một khách hàng** bắt buộc phải được xử lý tuần tự theo đúng thứ tự thời gian gửi đến (Strict FIFO).
2. Các lệnh giao dịch của **các khách hàng khác nhau** phải có thể được **xử lý song song đồng thời** bởi nhiều backend worker khác nhau để tối đa hóa thông lượng (Throughput) của hệ thống.

Cấu hình thông số nào sau đây trên message đáp ứng chính xác yêu cầu này?

- **A.** Đặt giá trị **`MessageGroupId`** bằng mã định danh duy nhất của từng khách hàng (`CustomerID`).
- **B.** Đặt giá trị `MessageDeduplicationId` bằng `CustomerID`.
- **C.** Tạo ra hàng ngàn SQS FIFO Queue riêng biệt cho từng khách hàng.
- **D.** Tăng giới hạn kích thước message lên 256 KB.

---

### CÂU 3 (Mức độ: Nâng cao - Decoupling & Fan-Out Architecture)
Một trang web thương mại điện tử cần xử lý sự kiện đặt hàng mới (`OrderCreated`). Khi một đơn hàng được tạo:
- Dịch vụ Thanh toán (Billing) cần nhận thông tin để trừ tiền trong thẻ tín dụng.
- Dịch vụ Quản lý Kho (Inventory) cần nhận thông tin để cập nhật lượng tồn kho.
- Dịch vụ Thông báo (Customer Notification) cần gửi email xác nhận cho khách hàng.

Các dịch vụ này thuộc các nhóm kỹ sư khác nhau quản lý, có tốc độ xử lý khác nhau và cần phải hoạt động hoàn toàn độc lập (Decoupled), sao cho nếu dịch vụ Quản lý Kho bị sập hoặc quá tải thì các dịch vụ Thanh toán và Thông báo vẫn hoạt động bình thường mà không bị mất dữ liệu đơn hàng.

Kiến trúc nào sau đây là chuẩn mực nhất theo AWS Well-Architected Framework?

- **A.** Ứng dụng đặt hàng gửi trực tiếp HTTP POST request tới API của cả 3 dịch vụ cùng một lúc.
- **B.** Ứng dụng đặt hàng đẩy thông điệp vào một **Amazon SNS Topic**. Tạo **3 hàng đợi Amazon SQS độc lập** đăng ký làm Subscriber của SNS Topic đó (Fan-Out pattern), mỗi dịch vụ backend sẽ kéo (pull) dữ liệu từ SQS queue riêng của mình để xử lý.
- **C.** Sử dụng một Amazon SQS Standard Queue duy nhất và cấu hình cho cả 3 dịch vụ cùng đọc chung từ queue này.
- **D.** Sử dụng AWS Step Functions để gọi đồng bộ (Synchronous) lần lượt từng dịch vụ.

---

### CÂU 4 (Mức độ: Trung bình - Streaming Data Ingestion: Streams vs Firehose)
Một công ty viễn thông cần thu thập hàng triệu bản ghi nhật ký cuộc gọi (Call Detail Records - CDR) mỗi giây từ các trạm phát sóng di động. Dữ liệu này cần được gom lô (batching), tự động chuyển đổi sang định dạng Apache Parquet, và nạp trực tiếp vào **Amazon S3** và **Amazon Redshift** để phục vụ báo cáo phân tích kinh doanh hàng giờ. Độ trễ yêu cầu trong khoảng từ **1 đến 5 phút (Near real-time)** và công ty muốn một giải pháp **Serverless hoàn toàn (không phải quản lý hạ tầng hay phân bổ Shard)**.

Giải pháp nào sau đây đáp ứng tốt nhất với chi phí và công sức quản trị tối thiểu?

- **A.** Amazon Kinesis Data Streams kết hợp với ứng dụng chạy trên cụm EC2 Auto Scaling.
- **B.** **Amazon Kinesis Data Firehose** tích hợp với một hàm AWS Lambda để chuyển đổi dữ liệu sang định dạng Parquet trước khi lưu vào S3 và nạp vào Redshift.
- **C.** Amazon SQS Standard kết hợp với tác vụ AWS Glue chạy định kỳ 1 phút/lần.
- **D.** AWS Database Migration Service (AWS DMS) với replication instance loại Multi-AZ.

---

### CÂU 5 (Mức độ: Nâng cao - Event-Driven Architecture với EventBridge)
Một doanh nghiệp sử dụng một phần mềm quản lý quan hệ khách hàng dạng SaaS (như Zendesk hoặc Salesforce). Doanh nghiệp muốn xây dựng một hệ sinh thái hướng sự kiện (Event-Driven) trong đó:
- Khi một khiếu nại của khách hàng (Support Ticket) được tạo trên phần mềm SaaS với độ ưu tiên là `"High"`, hệ thống AWS phải tự động kích hoạt một luồng xử lý trên **AWS Step Functions**.
- Các khiếu nại có độ ưu tiên thấp hoặc bình thường sẽ bị bỏ qua và không kích hoạt luồng này.
- Toàn bộ kiến trúc phải được quản lý tập trung và dễ dàng mở rộng thêm các mục tiêu xử lý khác trong tương lai.

Dịch vụ nào của AWS cung cấp giải pháp tích hợp tối ưu nhất cho kịch bản này?

- **A.** Amazon SNS Topic với Filter Policy lọc theo header HTTP.
- **B.** **Amazon EventBridge (Partner Event Bus)** kết hợp với một EventBridge Rule có **Content-based Event Pattern** để lọc các sự kiện có thuộc tính `{"priority": ["High"]}` và trỏ mục tiêu tới AWS Step Functions.
- **C.** Amazon Simple Queue Service (SQS) kết hợp với một cron job trên CloudWatch Events.
- **D.** Triển khai một cụm EC2 proxy Nginx liên tục thăm dò (polling) API của phần mềm SaaS.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **B**
- **Tại sao đúng**: 
  - Đây là câu hỏi kinh điển về cơ chế **Visibility Timeout** của Amazon SQS:
    - Mặc định Visibility Timeout của một SQS queue là **30 giây**. Khi một worker lấy message ra để xử lý, message đó bị ẩn đi trong 30 giây.
    - Nếu tác vụ render video kéo dài từ 3 đến 5 phút, thì sau giây thứ 30, message sẽ tự động xuất hiện trở lại trong queue $\rightarrow$ các worker khác tưởng là message mới và lấy ra xử lý lần 2.
    - **Cách giải quyết tối ưu**:
      1. Tăng cấu hình Visibility Timeout của Queue lên lớn hơn thời gian xử lý tối đa (ví dụ: 6 phút).
      2. Hoặc trong quá trình chạy, worker định kỳ gọi API **`ChangeMessageVisibility`** để gia hạn thêm thời gian (Heartbeat mechanism) nếu tác vụ chưa kết thúc.
- **Phân tích bẫy**:
  - *Phương án A*: SQS FIFO cũng có cơ chế Visibility Timeout mặc định 30 giây y hệt như Standard Queue, không giải quyết được vấn đề nếu không tăng timeout.
  - *Phương án C*: Message Retention Period là thời gian lưu message trong queue trước khi bị hủy (mặc định 4 ngày), giảm xuống 1 phút làm mất message nếu worker chưa kịp đọc.
  - *Phương án D*: Long Polling chỉ tối ưu thời gian chờ khi *lấy message* về, không ảnh hưởng đến thời gian xử lý message sau khi đã lấy ra.

---

### Đáp án Câu 2: **A**
- **Tại sao đúng**: 
  - Trong **Amazon SQS FIFO**, thông số **`MessageGroupId`** được thiết kế chính xác cho bài toán này:
    - **Tính tuần tự (Ordering)**: Tất cả các messages có cùng một `MessageGroupId` sẽ được xử lý tuần tự nghiêm ngặt theo thứ tự First-In, First-Out (chỉ một consumer được xử lý group đó tại một thời điểm). Bằng cách đặt `MessageGroupId = CustomerID`, các lệnh mua/bán của *cùng một khách hàng* luôn được khớp lệnh đúng thứ tự.
    - **Tính song song (Concurrency/Scale)**: Các messages có `MessageGroupId` khác nhau (khách hàng khác nhau) hoàn toàn độc lập và có thể được phân phối cho **nhiều worker khác nhau xử lý song song cùng một lúc**.
- **Phân tích bẫy**:
  - *Phương án B*: `MessageDeduplicationId` dùng để chống gửi trùng message trong vòng 5 phút (nếu 2 message có cùng ID này thì message thứ hai bị hủy bỏ), không dùng để nhóm luồng xử lý song song.
  - *Phương án C*: Tạo hàng ngàn queue là một kiến trúc tồi tệ (Anti-pattern), cực kỳ khó quản lý và tốn chi phí vận hành.
  - *Phương án D*: Tăng kích thước message không liên quan đến cơ chế xử lý tuần tự hay song song.

---

### Đáp án Câu 3: **B**
- **Tại sao đúng**: 
  - **Mô hình SNS + SQS Fan-Out** là mô hình chuẩn mực nhất của AWS để thực hiện kiến trúc Pub/Sub phân tán (Decoupling):
    - Đơn hàng được publish vào **1 SNS Topic duy nhất**.
    - Mỗi hệ thống backend (Billing, Inventory, Notification) có một **SQS Queue riêng** đăng ký nhận tin từ Topic đó.
    - Khi có đơn hàng mới, SNS tự động nhân bản (fan-out) thông điệp vào cả 3 hàng đợi cùng lúc.
    - **Tính độc lập (Fault Tolerance)**: Nếu dịch vụ Quản lý Kho bị sập hoặc xử lý chậm, các tin nhắn vẫn nằm an toàn trong SQS của Kho và chờ worker phục hồi để xử lý tiếp, trong khi dịch vụ Thanh toán và Thông báo vẫn rút tin từ SQS của riêng họ và chạy bình thường mà không bị ảnh hưởng.
- **Phân tích bẫy**:
  - *Phương án A*: Gọi HTTP POST đồng bộ tạo ra sự phụ thuộc chặt chẽ (Tight Coupling). Nếu một dịch vụ bị chậm hoặc sập, toàn bộ tiến trình đặt hàng của người dùng sẽ bị lỗi hoặc timeout.
  - *Phương án C*: Nếu 3 dịch vụ đọc chung 1 SQS queue, một message khi bị dịch vụ Billing lấy ra sẽ bị xóa và 2 dịch vụ còn lại sẽ không nhận được dữ liệu (SQS là điểm-tới-điểm, không phải Pub/Sub).
  - *Phương án D*: Step Functions đồng bộ sẽ giữ kết nối và thất bại nếu một thành phần sập.

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**: 
  - Đề bài có các từ khóa nhận diện: **"Gom lô nạp trực tiếp vào S3 và Redshift"**, **"Độ trễ 1 đến 5 phút (Near real-time)"**, và **"Serverless hoàn toàn (không quản lý Shards)"**.
  - **Amazon Kinesis Data Firehose**:
    - Là dịch vụ fully managed 100% serverless, tự động co giãn theo dung lượng dữ liệu mà không cần người dùng cấp phát Shards.
    - Tự động gom lô (batching) theo thời gian (60 - 900 giây) hoặc theo kích thước (1 - 128 MB).
    - Hỗ trợ tích hợp sẵn tính năng gọi hàm **AWS Lambda** để chuyển đổi định dạng dữ liệu (ví dụ: chuyển từ JSON sang columnar format như **Apache Parquet hoặc ORC**) trước khi ghi trực tiếp vào S3 hoặc Redshift.
- **Phân tích bẫy**:
  - *Phương án A*: Kinesis Data Streams yêu cầu bạn phải quản lý số lượng Shards (hoặc On-Demand mode) và phải tự viết/duy trì ứng dụng Consumer trên EC2 để đọc stream và ghi vào S3/Redshift $\rightarrow$ tốn nhiều công sức quản trị.
  - *Phương án C*: SQS kết hợp Glue chạy định kỳ 1 phút/lần không tối ưu cho streaming và chi phí Glue job rất cao.
  - *Phương án D*: AWS DMS dùng cho di chuyển dữ liệu cơ sở dữ liệu (Database Migration), không dùng để thu thập log streaming từ trạm viễn thông.

---

### Đáp án Câu 5: **B**
- **Tại sao đúng**: 
  - **Amazon EventBridge (Partner Event Bus)** là giải pháp chuẩn mực để tích hợp các sự kiện từ ứng dụng SaaS của bên thứ ba (Zendesk, Salesforce, Datadog...) vào hệ sinh thái AWS.
  - **Content-based Filtering**: EventBridge cho phép viết các quy tắc (Rules) phân tích sâu vào nội dung JSON payload của sự kiện (ví dụ: chỉ bắt các event có `{"detail": {"priority": ["High"]}}`) để định tuyến chính xác tới mục tiêu là **AWS Step Functions**.
- **Phân tích bẫy**:
  - *Phương án A*: Amazon SNS Topic không hỗ trợ Partner Event Bus từ bên thứ 3 và khả năng lọc nội dung JSON payload bên trong không mạnh mẽ và linh hoạt bằng EventBridge.
  - *Phương án C*: SQS không thể lọc theo nội dung payload một cách tự động và không có cơ chế nhận event từ SaaS bên thứ ba.
  - *Phương án D*: Tự dựng EC2 polling API vi phạm nguyên tắc thiết kế hướng sự kiện (Event-driven) và tốn chi phí vận hành.
