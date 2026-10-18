# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 11: SERVERLESS & ORCHESTRATION

---

### CÂU 1 (Mức độ: Nâng cao - API Gateway Timeout & Asynchronous Pattern)
Một ứng dụng web cho phép người dùng tải lên danh sách hóa đơn và kích hoạt quy trình tạo báo cáo thuế tổng hợp. Tác vụ tạo báo cáo thuế chạy trên một hàm AWS Lambda và mất trung bình **từ 45 giây đến 2 phút** để hoàn thành. Khi tích hợp trực tiếp qua **Amazon API Gateway** đồng bộ, người dùng liên tục nhận được mã lỗi **HTTP 504 Gateway Timeout**.

Giải pháp kiến trúc nào sau đây khắc phục triệt để lỗi timeout này theo đúng chuẩn thiết kế Serverless?

- **A.** Tăng thời gian Integration Timeout của Amazon API Gateway lên 5 phút trong bảng điều khiển.
- **B.** Tách quy trình thành kiến trúc bất đồng bộ (Asynchronous): Hàm Lambda đầu tiên nhận request, tạo một mã tác vụ (`TaskID`), gửi thông tin vào một **Amazon SQS Queue**, và trả về ngay lập tức mã **HTTP 202 Accepted** cho người dùng. Một hàm Lambda thứ hai sẽ đọc từ SQS để xử lý tạo báo cáo và lưu kết quả vào Amazon S3.
- **C.** Chuyển đổi API Gateway từ REST API sang HTTP API để tăng gấp đôi thời gian timeout.
- **D.** Tăng dung lượng bộ nhớ RAM của Lambda lên 10 GB để ép tác vụ chạy xong trong 10 giây.

---

### CÂU 2 (Mức độ: Nâng cao - Lambda VPC Networking & Internet Egress)
Một công ty tài chính phát triển một hàm AWS Lambda để tự động thanh toán nợ cho khách hàng. Hàm Lambda này cần:
1. Kết nối vào một cơ sở dữ liệu **Amazon RDS PostgreSQL** nằm trong một **Private Subnet** của VPC.
2. Gọi ra một cổng thanh toán điện tử (Payment Gateway) của bên thứ ba nằm ngoài **Internet công cộng** để xác thực giao dịch thẻ.

Sau khi kỹ sư cấu hình gắn hàm Lambda vào Private Subnet của VPC để kết nối với database RDS, hàm Lambda kết nối database thành công nhưng **hoàn toàn mất kết nối và báo lỗi timeout khi gọi ra cổng thanh toán ngoài Internet**.

Nguyên nhân chính là gì và cách khắc phục chuẩn xác nhất là gì?

- **A.** Cổng thanh toán bên thứ ba chặn IP của AWS. Cần chuyển Lambda sang một tài khoản AWS khác.
- **B.** Khi được gắn vào VPC, Lambda mất toàn bộ quyền truy cập Internet mặc định. Kỹ sư cần triển khai một **NAT Gateway trong Public Subnet**, và cập nhật Route Table của Private Subnet (nơi gắn Lambda) trỏ `0.0.0.0/0` về NAT Gateway này.
- **C.** Lambda trong VPC không bao giờ có thể đi ra Internet. Bắt buộc phải di chuyển RDS PostgreSQL ra Public Subnet.
- **D.** Kỹ sư chỉ cần gắn Elastic IP trực tiếp vào cấu hình card mạng của hàm Lambda.

---

### CÂU 3 (Mức độ: Trung bình/Nâng cao - Triệt tiêu Cold Start cho Hệ thống Tài chính)
Một ngân hàng số cung cấp API thanh toán mã QR trực tuyến thông qua cụm Amazon API Gateway và AWS Lambda. Trong các giờ cao điểm ngẫu nhiên, một số khách hàng phàn nàn rằng giao dịch thanh toán đầu tiên bị trễ từ 3 đến 5 giây do hiện tượng **Khởi động lạnh (Cold Start)** khi Lambda phải nạp lại container và khởi tạo môi trường thực thi.
Ban giám đốc yêu cầu loại bỏ hoàn toàn độ trễ khởi động lạnh này để đảm bảo 100% các lệnh thanh toán đều phản hồi với **độ trễ siêu thấp ổn định ở mức hai chữ số mili-giây (double-digit milliseconds)**.

Tính năng nào của AWS Lambda giải quyết chính xác yêu cầu này?

- **A.** Cấu hình **Reserved Concurrency** cho hàm Lambda.
- **B.** Kích hoạt tính năng **Provisioned Concurrency** cho hàm Lambda với số lượng instances ước tính trước.
- **C.** Tăng dung lượng đĩa tạm `/tmp` của Lambda lên 10 GB.
- **D.** Chuyển mã nguồn Lambda từ ngôn ngữ Python sang Node.js.

---

### CÂU 4 (Mức độ: Nâng cao - Serverless Workflow Orchestration)
Một chuỗi cung ứng logistics cần xây dựng một quy trình xử lý đơn hàng xuất nhập khẩu phức tạp. Quy trình bao gồm:
1. Kiểm tra tồn kho và đặt trước hàng hóa (mất vài giây).
2. Gửi yêu cầu phê duyệt thông quan hải quan tới nhân viên kiểm soát. Quá trình phê duyệt này có thể kéo dài **từ 2 đến 7 ngày** chờ con người nhấn nút xác nhận trên cổng thông tin.
3. Sau khi được duyệt, gọi tiếp các dịch vụ đóng gói và vận chuyển.
4. Nếu bị từ chối hoặc hết hạn 7 ngày, tự động hoàn trả hàng tồn kho và hủy đơn.

Dịch vụ nào của AWS là lựa chọn tối ưu nhất để điều phối luồng quy trình nghiệp vụ kéo dài nhiều ngày này?

- **A.** Amazon SQS FIFO Queue kết hợp với Dead Letter Queue.
- **B.** **AWS Step Functions (Standard Workflow)** sử dụng Task Token callback để tạm dừng quy trình chờ phản hồi từ con người.
- **C.** Một hàm AWS Lambda chạy liên tục với vòng lặp `while` kiểm tra trạng thái database mỗi phút.
- **D.** AWS Step Functions (Express Workflow) với cấu hình Timeout 14 ngày.

---

### CÂU 5 (Mức độ: Cơ bản/Trung bình - Lựa chọn loại API Gateway)
Một nhóm phát triển đang xây dựng tính năng **Chat trực tuyến thời gian thực (Real-time Live Chat)** giữa người mua và người bán trên một sàn thương mại điện tử. Ứng dụng yêu cầu máy chủ có thể chủ động đẩy tin nhắn mới về cho trình duyệt của người dùng ngay khi có tin nhắn đến mà máy khách không cần phải liên tục gửi request thăm dò (polling).

Loại API nào trên Amazon API Gateway nên được sử dụng?

- **A.** REST API ở chế độ Edge-Optimized.
- **B.** HTTP API với tính năng CORS.
- **C.** **WebSocket API**.
- **D.** Private REST API kết hợp với VPC Endpoint.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **B**
- **Tại sao đúng**: 
  - **Giới hạn cứng của Amazon API Gateway**: Thời gian chờ phản hồi tối đa (Maximum Integration Timeout) của API Gateway là **29 giây** (và không thể tăng thêm). Do đó, bất kỳ tác vụ nào mất từ 45 giây đến 2 phút sẽ luôn bị lỗi **HTTP 504 Gateway Timeout** nếu chạy đồng bộ.
  - **Mô hình kiến trúc bất đồng bộ (Asynchronous Decoupled Pattern)** là giải pháp chuẩn mực của AWS:
    1. Client gửi request $\rightarrow$ Lambda Dispatcher tiếp nhận, đẩy thông tin vào SQS và **trả về ngay lập tức HTTP 202 Accepted** kèm `TaskID` trong vòng chưa đầy 1 giây.
    2. Client nhận phản hồi ngay, không bị treo kết nối và không bị timeout.
    3. Một Lambda Worker độc lập ở phía sau sẽ rút message từ SQS để chạy tác vụ render trong 2 phút (Lambda hỗ trợ timeout lên tới 15 phút) rồi lưu file kết quả vào S3.
- **Phân tích bẫy**:
  - *Phương án A*: Giới hạn 29s của API Gateway là giới hạn cứng (hard limit), không thể cấu hình tăng lên 5 phút trên console.
  - *Phương án C*: HTTP API thậm chí có giới hạn timeout tối đa là 30 giây, không giải quyết được tác vụ 2 phút.
  - *Phương án D*: Tăng RAM có thể giúp code chạy nhanh hơn nhưng không thể đảm bảo chắc chắn tác vụ nặng luôn kết thúc dưới 29 giây.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - Đây là một **bẫy mạng (Networking Trap) kinh điển** của AWS Lambda:
    - Mặc định, Lambda chạy trong VPC công cộng của AWS và có đường truyền Internet riêng.
    - Khi bạn cấu hình Lambda kết nối vào một VPC cá nhân (để truy cập tài nguyên private như RDS, ElastiCache), AWS sẽ cấp phát các card mạng ảo (Hyperplane ENI) bên trong các Subnet bạn chọn.
    - **Hậu quả**: Toàn bộ kết nối Internet mặc định của Lambda bị ngắt hoàn toàn. Các ENI của Lambda chỉ có Private IP, không có Public IP.
    - **Cách giải quyết**: Muốn Lambda trong Private Subnet vừa đọc được RDS vừa đi ra Internet để gọi Payment Gateway: Private Subnet đó bắt buộc phải có một Route trỏ `0.0.0.0/0` tới một **NAT Gateway** (hoặc NAT Instance) nằm ở Public Subnet của VPC.
- **Phân tích bẫy**:
  - *Phương án A*: Lỗi xảy ra do thiếu đường định tuyến (routing) từ VPC ra Internet, không phải do bên thứ ba chặn.
  - *Phương án C*: Đưa RDS ra Public Subnet là hành vi cực kỳ nguy hiểm, vi phạm nghiêm trọng an toàn cơ sở dữ liệu tài chính.
  - *Phương án D*: Không thể gán Elastic IP trực tiếp cho hàm Lambda hay card mạng Hyperplane ENI của Lambda.

---

### Đáp án Câu 3: **B**
- **Tại sao đúng**: 
  - Hiện tượng **Cold Start** xảy ra khi Lambda phải khởi tạo môi trường thực thi mới (tải container image, nạp runtime, chạy code khởi tạo bên ngoài handler).
  - **Provisioned Concurrency** là giải pháp của AWS sinh ra để giải quyết triệt để vấn đề này:
    - Nó phân bổ và khởi tạo trước một số lượng instances của hàm Lambda, giữ chúng luôn ở trạng thái "ấm" (Warm & Initialized).
    - Khi có request ập đến, hàm xử lý ngay lập tức với thời gian phản hồi chỉ vài mili-giây (**Zero Cold Start**).
- **Phân tích bẫy**:
  - *Phương án A*: **Reserved Concurrency** chỉ đảm bảo số lượng hàm tối đa được chạy đồng thời (Concurrency limit), nó **KHÔNG giữ các instances ở trạng thái ấm**, do đó các request mới vẫn bị dính Cold Start như thường.
  - *Phương án C*: Tăng dung lượng `/tmp` không ảnh hưởng đến thời gian nạp container.
  - *Phương án D*: Đổi ngôn ngữ có thể giảm nhẹ vài trăm mili-giây nhưng không thể loại bỏ hoàn toàn Cold Start.

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**: 
  - Yêu cầu đề bài: Quy trình nghiệp vụ phức tạp, kéo dài **nhiều ngày (2 đến 7 ngày)** và có bước **chờ con người duyệt (Human approval)**.
  - **AWS Step Functions (Standard Workflow)**:
    - Cho phép một quy trình State Machine duy trì trạng thái thực thi lên tới **1 năm (365 ngày)**.
    - Cung cấp cơ chế **Task Token (Callback Pattern)**: Step Functions tạm dừng luồng tại một bước, gửi email/thông báo kèm một mã Token duy nhất cho người quản lý. Luồng sẽ ngủ đông mà không tốn chi phí compute cho đến khi người quản lý nhấn nút "Duyệt" trên giao diện web (gọi API `SendTaskSuccess` kèm Token) để Step Functions thức dậy và tiếp tục chạy các bước tiếp theo.
- **Phân tích bẫy**:
  - *Phương án A*: SQS Queue có thời gian lưu message tối đa là 14 ngày nhưng không có khả năng điều phối phân nhánh logic phức tạp (State machine).
  - *Phương án C*: Cho Lambda chạy vòng lặp chờ là bất khả thi vì Lambda chỉ sống tối đa 15 phút.
  - *Phương án D*: **Step Functions Express Workflow chỉ chạy tối đa 5 phút**, không thể chạy 7 ngày.

---

### Đáp án Câu 5: **C**
- **Tại sao đúng**: 
  - Ứng dụng chat trực tuyến thời gian thực đòi hỏi kết nối **hai chiều (Full-Duplex)** và có lưu trạng thái (Stateful) để server có thể chủ động đẩy (Push) tin nhắn về client ngay khi có tin mới mà client không cần gửi request hỏi thăm (No Polling).
  - **Amazon API Gateway WebSocket API** được thiết kế riêng cho mục đích này: Giữ kết nối socket mở liên tục giữa trình duyệt và API Gateway, tự động tích hợp với các backend service như AWS Lambda hay DynamoDB.
- **Phân tích bẫy**:
  - *Phương án A & B*: REST API và HTTP API hoạt động theo mô hình Stateless Request-Response (Client phải chủ động gửi request thì Server mới trả lời được), không hỗ trợ Server-Push tự nhiên cho Live Chat.
  - *Phương án D*: Private REST API chỉ dùng cho kết nối nội bộ trong VPC, không phải giao thức WebSocket.
