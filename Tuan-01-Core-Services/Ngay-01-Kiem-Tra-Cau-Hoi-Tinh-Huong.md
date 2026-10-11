# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 1: VPC & NETWORKING NỀN TẢNG

---

### CÂU 1 (Mức độ: Cơ bản - Khả năng nhận diện kiến trúc Subnet & Route Table)
Một công ty đang triển khai ứng dụng web hai tầng (two-tier) trên AWS gồm các web server và database server. Theo chính sách an ninh, database server không được phép kết nối trực tiếp từ Internet nhưng phải có khả năng tải các bản vá bảo mật từ Internet. 

Kiến trúc nào sau đây đáp ứng yêu cầu với **chi phí vận hành tối thiểu (least operational overhead)**?

- **A.** Đặt database server trong public subnet, dùng Security Group để chặn toàn bộ inbound traffic từ 0.0.0.0/0 và mở outbound tới 0.0.0.0/0.
- **B.** Đặt database server trong private subnet. Tạo một NAT Gateway trong public subnet, và cập nhật Route Table của private subnet để route `0.0.0.0/0` tới NAT Gateway.
- **C.** Đặt database server trong private subnet. Dựng một EC2 instance cấu hình iptables làm NAT Instance trong public subnet, và route `0.0.0.0/0` tới ENI của NAT Instance.
- **D.** Đặt database server trong private subnet. Gắn một Internet Gateway thứ hai vào VPC và cấu hình Route Table riêng cho database server.

---

### CÂU 2 (Mức độ: Trung bình - High Availability & Fault Tolerance)
Một giải pháp trên AWS có các backend worker chạy trên EC2 đặt tại 2 Availability Zone (AZ-a và AZ-b) trong các private subnets. Các worker này liên tục gọi các bên API thứ ba bên ngoài Internet để xử lý giao dịch tài chính. Hiện tại, kiến trúc sư chỉ tạo **1 NAT Gateway duy nhất** đặt tại Public Subnet của AZ-a, và trỏ route `0.0.0.0/0` của cả 2 private subnet về NAT Gateway này.

Trong một sự cố hiếm hoi, AZ-a bị mất điện toàn phần dẫn đến gián đoạn dịch vụ của worker tại AZ-b. Giải pháp nào sau đây giúp khắc phục vấn đề này để đảm bảo **tính sẵn sàng cao nhất (highest availability)** và **giảm chi phí data transfer**?

- **A.** Tạo thêm 1 NAT Gateway thứ hai đặt tại Public Subnet của AZ-b. Cập nhật Route Table của Private Subnet ở AZ-b trỏ `0.0.0.0/0` tới NAT Gateway ở AZ-b.
- **B.** Tạo một Auto Scaling Group cho NAT Gateway để tự động scale sang AZ-b khi AZ-a sập.
- **C.** Gán thêm một Elastic IP thứ hai vào NAT Gateway hiện tại để dự phòng failover.
- **D.** Chuyển các worker sang Public Subnet và gán Elastic IP cho từng worker.

---

### CÂU 3 (Mức độ: Trung bình - Phân tích địa chỉ IP & Subnet Sizing)
Một Solutions Architect cần thiết kế một VPC mới cho một cụm Kubernetes tự quản lý trên EC2. Dự kiến hệ thống có:
- Tối thiểu 500 EC2 instances phân bổ đều trên 3 Availability Zones.
- Mỗi AZ cần một subnet riêng.

Cấu hình CIDR block nào cho mỗi Subnet sau đây là **nhỏ nhất** có thể chứa đủ số lượng instance yêu cầu cho mỗi AZ?

- **A.** `/25`
- **B.** `/24`
- **C.** `/23`
- **D.** `/22`

---

### CÂU 4 (Mức độ: Nâng cao - Xử lý sự cố mạng & Routing)
Một kỹ sư tạo một EC2 instance trong một Private Subnet. Instance này không thể kết nối tới Internet dù đã có NAT Gateway hoạt động trong VPC. Khi kiểm tra cấu hình:
1. Private Subnet Route Table: `0.0.0.0/0` -> `nat-0123456789abcdef0`.
2. Public Subnet nơi đặt NAT Gateway: Route Table chỉ có route `10.0.0.0/16` -> `local`.
3. NAT Gateway đã được gán Elastic IP hợp lệ.

Nguyên nhân chính khiến EC2 không ra được Internet là gì?

- **A.** Security Group của NAT Gateway đang chặn outbound traffic.
- **B.** Public Subnet chứa NAT Gateway thiếu route `0.0.0.0/0` trỏ tới Internet Gateway (IGW).
- **C.** Tính năng Source/Destination Check trên NAT Gateway chưa được tắt.
- **D.** EC2 instance chưa được gán Public IPv4 address.

---

### CÂU 5 (Mức độ: Trung bình - IPv6 Egress Architecture)
Một công ty viễn thông triển khai ứng dụng IoT sử dụng địa chỉ IPv6 trên các EC2 instances trong một VPC. Các cảm biến IoT cần gửi dữ liệu telemetry ra các server ngoài Internet, nhưng công ty yêu cầu tuyệt đối **không được cho phép bất kỳ kết nối nào từ Internet chủ động truy cập vào các instance này**.

Giải pháp kiến trúc nào tối ưu nhất?

- **A.** Triển khai một NAT Gateway trong public subnet và cấu hình routing IPv6 qua NAT Gateway.
- **B.** Đặt các instance trong private subnet và triển khai **Egress-Only Internet Gateway**. Cấu hình Route Table trỏ `::/0` về Egress-Only Internet Gateway.
- **C.** Gắn Internet Gateway (IGW) thông thường vào VPC và dùng Network ACL để chặn toàn bộ Inbound IPv6 traffic từ mọi nguồn (`::/0`).
- **D.** Sử dụng AWS PrivateLink để kết nối các IoT instances với các máy chủ bên ngoài.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **B**
- **Tại sao đúng**: 
  - Đặt DB trong Private Subnet đảm bảo không ai từ Internet truy cập được.
  - Dùng **NAT Gateway** đáp ứng trọn vẹn yêu cầu outbound tải patch, đồng thời là managed service của AWS $\rightarrow$ thỏa mãn điều kiện *"least operational overhead"*.
- **Phân tích bẫy**:
  - *Phương án A*: Vi phạm an ninh nguyên tắc vì đặt DB trực tiếp ở Public Subnet. Chỉ cần một cấu hình sai ở Security Group là DB bị lộ ra ngoài Internet.
  - *Phương án C*: NAT Instance chạy trên EC2 đòi hỏi phải tự vá lỗi hệ điều hành, cấu hình script, quản lý failover $\rightarrow$ tốn operational overhead.
  - *Phương án D*: Một VPC **chỉ có thể gắn tối đa 1 Internet Gateway**. Việc nói "gắn IGW thứ hai" là sai nguyên lý AWS.

---

### Đáp án Câu 2: **A**
- **Tại sao đúng**: 
  - NAT Gateway là dịch vụ **AZ-redundant (chỉ có tính dự phòng trong 1 AZ duy nhất)**, KHÔNG tự động failover sang AZ khác.
  - Thiết kế chuẩn High Availability (HA) theo AWS Well-Architected Framework: **Mỗi AZ có 1 Public Subnet chứa 1 NAT Gateway riêng**. Private Subnet ở AZ nào sẽ route outbound internet qua NAT Gateway của chính AZ đó.
  - Điều này giải quyết triệt để 2 vấn đề:
    1. **Resilience**: AZ-a sập thì NAT Gateway B ở AZ-b vẫn hoạt động bình thường.
    2. **Cost**: Tránh phát sinh chi phí Data Transfer liên vùng (Cross-AZ data transfer: $0.01/GB mỗi chiều).
- **Phân tích bẫy**:
  - *Phương án B*: NAT Gateway là dịch vụ managed, **không nằm trong Auto Scaling Group**.
  - *Phương án C*: Gán thêm EIP không giúp tạo HA qua vùng khác.
  - *Phương án D*: Vi phạm nguyên tắc bảo mật khi đưa worker ra public subnet.

---

### Đáp án Câu 3: **B**
- **Tại sao đúng**:
  - Yêu cầu: Tối thiểu 500 instances trên 3 AZ $\rightarrow$ Mỗi AZ cần chứa: $500 / 3 \approx 167$ instances.
  - Tính toán số IP usable trên AWS (trừ đi 5 IP reserved của AWS):
    - Subnet `/25`: $2^{(32-25)} = 128$ IPs $\rightarrow$ Khả dụng: $128 - 5 = 123$ IPs (**KHÔNG ĐỦ** cho 167 instances).
    - Subnet `/24`: $2^{(32-24)} = 256$ IPs $\rightarrow$ Khả dụng: $256 - 5 = 251$ IPs (**ĐỦ** cho 167 instances).
  - Do đề bài hỏi CIDR block **nhỏ nhất** có thể chứa đủ, nên `/24` là đáp án chính xác nhất.
- **Phân tích bẫy**:
  - `/23` và `/22` chứa đủ nhưng không phải là block *nhỏ nhất* (tốn dải IP thừa).

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**:
  - Một subnet được coi là **Public Subnet** khi và chỉ khi Route Table của nó có route: `0.0.0.0/0 -> igw-xxxx`.
  - Ở đây, subnet chứa NAT Gateway chỉ có route `10.0.0.0/16 -> local`. Điều này biến nó thành Private Subnet. Dù NAT Gateway có Elastic IP, traffic từ NAT Gateway không có đường ra ngoài Internet vì thiếu route tới IGW.
- **Phân tích bẫy**:
  - *Phương án A*: NAT Gateway **không có Security Group** (chỉ có Elastic IP và gắn vào Subnet).
  - *Phương án C*: Tắt Source/Destination Check chỉ áp dụng cho **NAT Instance** (EC2), không áp dụng cho NAT Gateway.
  - *Phương án D*: EC2 trong Private Subnet tuyệt đối không cần Public IP khi đã đi qua NAT Gateway.

---

### Đáp án Câu 5: **B**
- **Tại sao đúng**:
  - **Egress-Only Internet Gateway** được thiết kế chuyên biệt cho IPv6 để thực hiện chức năng tương tự NAT: cho phép instance khởi tạo kết nối ra ngoài (egress), nhưng chặn toàn bộ chiều ngược lại từ ngoài Internet khởi tạo vào (prevent inbound).
  - Đây là stateful gateway dành riêng cho IPv6, hoàn toàn miễn phí và không cần cấu hình NAT phức tạp.
- **Phân tích bẫy**:
  - *Phương án A*: NAT Gateway hiện tại **không hỗ trợ dịch địa chỉ từ IPv6 sang IPv6 trên Internet**.
  - *Phương án C*: Dùng IGW kết hợp NACL chặn Inbound traffic có rủi ro bị cấu hình sai, và NACL là stateless $\rightarrow$ nếu chặn inbound thì các response packet trả về từ internet cũng có nguy cơ bị ảnh hưởng nếu không cấu hình mở ephemeral ports chính xác.
  - *Phương án D*: PrivateLink dùng để kết nối private giữa các VPC hoặc service trên AWS, không dùng để duyệt Internet công cộng chung.
