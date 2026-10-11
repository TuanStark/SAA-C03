# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 2: NETWORKING NÂNG CAO

---

### CÂU 1 (Mức độ: Trung bình - Security Group vs Network ACL)
Bộ phận Security của một công ty phát hiện một địa chỉ IP công cộng cụ thể (`203.0.113.50`) đang thực hiện tấn công brute-force liên tục vào cổng 22 (SSH) của các máy chủ web EC2 đặt trong một Public Subnet. Các máy chủ này được gắn một Security Group cho phép port 80, 443 từ mọi nơi (`0.0.0.0/0`) và port 22 từ mạng văn phòng công ty.

Hành động nào sau đây sẽ ngăn chặn ngay lập tức địa chỉ IP độc hại này với **thời gian xử lý nhanh nhất và ít ảnh hưởng nhất**?

- **A.** Thêm một Outbound Deny rule cho IP `203.0.113.50` trên Security Group của các EC2 instances.
- **B.** Thêm một Inbound Deny rule cho IP `203.0.113.50` trên Security Group của các EC2 instances.
- **C.** Thêm một Inbound Deny rule cho IP `203.0.113.50/32` trên Network ACL (NACL) liên kết với Public Subnet, với số thứ tự rule nhỏ hơn rule Allow All hiện tại.
- **D.** Tạo một Application Load Balancer phía trước các EC2 instances và cấu hình Listener Rule để drop traffic từ IP `203.0.113.50`.

---

### CÂU 2 (Mức độ: Nâng cao - VPC Endpoint & Hybrid Connectivity)
Một tập đoàn y tế có trung tâm dữ liệu On-Premises kết nối tới AWS thông qua đường truyền vật lý **AWS Direct Connect**. Ứng dụng quản lý hồ sơ bệnh án chạy tại on-premise cần upload các file ảnh chụp X-quang dung lượng lớn trực tiếp lên một **Amazon S3 bucket** trong AWS Region.
Chính sách an ninh yêu cầu:
1. Toàn bộ lưu lượng truyền tải dữ liệu KHÔNG ĐƯỢC đi qua mạng Internet công cộng.
2. Dữ liệu phải đi qua kết nối riêng tư (private connection) có sẵn của Direct Connect.
3. **Chi phí vận hành và độ phức tạp cấu hình phải ở mức tối thiểu**.

Giải pháp kiến trúc nào sau đây đáp ứng chính xác các yêu cầu trên?

- **A.** Tạo một **S3 Gateway Endpoint** trong VPC. Cấu hình DNS on-premises để trỏ các truy vấn S3 bucket về VPC Gateway Endpoint.
- **B.** Tạo một **S3 Interface Endpoint (AWS PrivateLink)** trong một subnet của VPC. Cấu hình ứng dụng on-premises hoặc DNS để gửi request tới Private IP hoặc private DNS name của Interface Endpoint này.
- **C.** Thiết lập một cụm EC2 proxy (Squid proxy) trong Public Subnet của VPC kết nối ra Internet qua Internet Gateway, và cấu hình on-premises gửi file qua proxy này.
- **D.** Tạo một AWS Storage Gateway dạng File Gateway trên nền tảng VMware on-premises và kích hoạt Public VIF trên đường Direct Connect.

---

### CÂU 3 (Mức độ: Nâng cao - Multi-VPC Architecture & Transitive Routing)
Một doanh nghiệp sở hữu 15 VPCs phục vụ các môi trường khác nhau (Dev, Test, Staging, Production). Doanh nghiệp vừa tạo thêm một **Shared Services VPC** chứa hệ thống Active Directory, Git Server và Logging tập trung. 
Yêu cầu đặt ra:
- Cả 15 VPCs nói trên đều cần truy cập hai chiều tới Shared Services VPC.
- Các VPCs Dev và Test tuyệt đối không được phép thấy hoặc gửi traffic tới các VPCs Production.
- Doanh nghiệp dự kiến sẽ tăng số lượng VPCs lên hơn 50 VPCs trong năm tới và sẽ kết nối thêm với mạng On-Premises.

Giải pháp kiến trúc mạng nào mang lại khả năng **mở rộng cao nhất (highest scalability)** và **dễ quản lý nhất (least operational complexity)**?

- **A.** Thiết lập Full-Mesh VPC Peering giữa tất cả 16 VPCs với nhau. Sử dụng Security Groups để chặn kết nối giữa Dev/Test và Production.
- **B.** Thiết lập 15 kết nối VPC Peering từ 15 VPCs tới Shared Services VPC.
- **C.** Triển khai **AWS Transit Gateway (TGW)**. Đính kèm (attach) tất cả 16 VPCs vào Transit Gateway. Sử dụng **nhiều Transit Gateway Route Tables** để cô lập (segment) traffic giữa Dev/Test và Production.
- **D.** Triển khai một cụm EC2 chạy phần mềm định tuyến mã nguồn mở (Software VPN/Router) trong Shared Services VPC và nối tất cả các VPC còn lại qua IPsec tunnels.

---

### CÂU 4 (Mức độ: Khó - Hybrid Connectivity & Disaster Recovery)
Một ngân hàng yêu cầu kết nối giữa Data Center chính và AWS VPC. Lưu lượng truyền dữ liệu trung bình là 2 Gbps với yêu cầu độ trễ thấp và ổn định. Ngân hàng đồng thời yêu cầu một **kênh kết nối dự phòng (backup connection)** tự động failover nếu kênh chính bị đứt cáp, với tiêu chí **tối ưu hóa chi phí nhất có thể (most cost-effective)**.

Kiến trúc nào sau đây là tối ưu nhất?

- **A.** Thiết lập 2 đường AWS Direct Connect độc lập 10 Gbps từ 2 nhà cung cấp dịch vụ viễn thông khác nhau kết nối tới 2 Direct Connect Locations.
- **B.** Thiết lập 1 đường kết nối **AWS Direct Connect (Private VIF)** làm kênh chính, và 1 kết nối **AWS Site-to-Site VPN qua Internet** làm kênh dự phòng (backup). Cấu hình định tuyến động **BGP** trên cả hai đường.
- **C.** Thiết lập 2 kết nối AWS Site-to-Site VPN qua Internet và kích hoạt ECMP (Equal-Cost Multi-Path) để gộp băng thông.
- **D.** Thiết lập 1 đường AWS Direct Connect và cấu hình AWS DataSync để sao lưu dữ liệu tự động định kỳ sang S3.

---

### CÂU 5 (Mức độ: Trung bình - Xử lý sự cố Network ACL & Ephemeral Ports)
Một kỹ sư mạng tạo một custom Network ACL mới và liên kết nó với một Public Subnet chứa web server (EC2). Kỹ sư này cấu hình Inbound Rules cho NACL như sau:
- **Rule 100**: Protocol TCP, Port Range 80, Source: 0.0.0.0/0, ALLOW
- **Rule 110**: Protocol TCP, Port Range 443, Source: 0.0.0.0/0, ALLOW
- **Rule \***: ALL Traffic, DENY

Security Group của EC2 đã mở port 80 và 443 Inbound từ `0.0.0.0/0`, và mở All Traffic Outbound.
Tuy nhiên, khi người dùng từ ngoài Internet duyệt web thì trình duyệt báo lỗi **Connection Timed Out**.

Nguyên nhân chính là gì và cách khắc phục?

- **A.** Security Group của EC2 thiếu Outbound rule cho port 80 và 443.
- **B.** Network ACL là **stateless**, cần phải thêm một **Outbound Rule trên NACL** cho phép traffic gửi về các cổng tạm thời (**Ephemeral Ports: TCP 1024–65535**) tới `0.0.0.0/0`.
- **C.** Network ACL cần phải đổi Rule 100 và Rule 110 thành Stateful bằng cách bật cờ ESTABLISHED.
- **D.** Cần phải gán Elastic IP trực tiếp cho Network ACL.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **C**
- **Tại sao đúng**: 
  - **Security Group KHÔNG CÓ khả năng tạo DENY rule** (nó chỉ hỗ trợ Allow). Do đó phương án A và B bị loại ngay lập tức.
  - **Network ACL (NACL)** hỗ trợ cả Allow và Deny rule. NACL đánh giá rule từ số nhỏ đến số lớn. Đặt Deny rule với rule number nhỏ hơn rule allow-all (ví dụ: Rule 50 Deny IP `203.0.113.50/32`) sẽ chặn ngay lập tức gói tin tại ranh giới Subnet trước khi nó chạm tới EC2 instance.
- **Phân tích bẫy**:
  - *Phương án D*: ALB chỉ xử lý HTTP/HTTPS (Layer 7), không thể chặn traffic SSH (Layer 4/Port 22) trừ khi kết hợp với AWS WAF, nhưng WAF cũng chỉ áp dụng cho Web Application traffic (Layer 7), không bảo vệ cổng SSH.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - Đây là câu hỏi kinh điển phân biệt **S3 Gateway Endpoint vs S3 Interface Endpoint**.
  - **S3 Gateway Endpoint** hoạt động bằng cách add Prefix List vào **Route Table của VPC**. Các máy chủ On-Premises kết nối qua Direct Connect hoặc VPN **KHÔNG THỂ** sử dụng Gateway Endpoint vì Route Table của VPC không thể định tuyến traffic từ bên ngoài router on-premise vào Gateway Endpoint.
  - **S3 Interface Endpoint (PrivateLink)** tạo ra các Elastic Network Interfaces (ENI) với **Private IPv4 cục bộ** ngay trong subnet của VPC. Máy chủ on-premise hoàn toàn có thể gửi request trực tiếp tới Private IP này thông qua đường Direct Connect chuyên dụng mà không cần chạm tới Internet.
- **Phân tích bẫy**:
  - *Phương án A*: Sai hoàn toàn vì Gateway Endpoint không thể tiếp nhận traffic định tuyến từ on-premises qua DX/VPN.
  - *Phương án C*: Dùng proxy EC2 public làm traffic đi ra Internet, vi phạm yêu cầu 1 và 2.
  - *Phương án D*: Triển khai Storage Gateway tăng chi phí vận hành và không phải là giải pháp direct upload tối giản cho ứng dụng hiện có.

---

### Đáp án Câu 3: **C**
- **Tại sao đúng**: 
  - Khi số lượng VPC tăng lên hàng chục hoặc hàng trăm, mô hình VPC Peering sẽ bùng nổ theo cấp số nhân và trở thành thảm họa vận hành (Operational nightmare).
  - **AWS Transit Gateway (TGW)** hoạt động như một Cloud Router trung tâm (Hub-and-Spoke).
  - TGW hỗ trợ tính năng **Multiple Route Tables (Network Segmentation)**: Có thể tạo một Route Table cho Dev/Test (chỉ trỏ route về Shared Services và không có route sang Prod) và một Route Table riêng cho Prod (chỉ trỏ về Shared Services và không có route sang Dev/Test). Điều này ngăn chặn hoàn toàn kết nối giữa Dev và Prod ở mức Layer 3.
- **Phân tích bẫy**:
  - *Phương án A*: Full-Mesh 16 VPCs cần $16 \times 15 / 2 = 120$ peering connections, không thể mở rộng và tốn chi phí quản lý cực lớn.
  - *Phương án B*: Chỉ giải quyết được kết nối tới Shared Services, nhưng khi sau này cần mở rộng kết nối On-Premise thì VPC Peering không hỗ trợ transitive routing (on-premise không thể đi xuyên qua Shared Services để vào 15 VPC kia).

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**: 
  - Yêu cầu đề bài có 2 trọng tâm: **Băng thông cao ổn định (Primary)** và **Tối ưu chi phí nhất (Most cost-effective backup)**.
  - Phương án kinh điển chuẩn AWS Architecture: **Direct Connect làm kênh chính, và Site-to-Site VPN làm kênh dự phòng**.
  - Kênh VPN chạy trên đường Internet công cộng nên chi phí duy trì chỉ là phí cố định hàng giờ rất rẻ (~$0.05/giờ) + phí transfer khi xảy ra sự cố.
  - Khi cả 2 kết nối cùng quảng bá dải mạng qua **BGP**, AWS sẽ tự động ưu tiên đường Direct Connect vì AS-Path ngắn hơn hoặc độ ưu tiên định tuyến cao hơn.
- **Phân tích bẫy**:
  - *Phương án A*: Dựng 2 đường Direct Connect đáp ứng tính sẵn sàng tối đa (Maximum Resiliency), nhưng **chi phí rất cao**, vi phạm ràng buộc *"most cost-effective"*.
  - *Phương án C*: VPN chỉ đạt tối đa 1.25 Gbps/tunnel, chạy qua Internet công cộng nên độ trễ dao động (high jitter), không đáp ứng yêu cầu lưu lượng 2 Gbps ổn định của ngân hàng.
  - *Phương án D*: DataSync chỉ dùng để đồng bộ file/data, không phải là kết nối mạng (network connectivity) cho toàn bộ hệ thống.

---

### Đáp án Câu 5: **B**
- **Tại sao đúng**: 
  - **Network ACL là STATELESS**. Nghĩa là khi client từ Internet gửi request vào cổng 80/443 (được phép bởi Inbound Rule 100 & 110), web server muốn gửi response trả về cho client thì gói tin phản hồi sẽ đi ra ngoài bằng các cổng ngẫu nhiên do client mở (gọi là **Ephemeral Ports**, thường là TCP từ **1024 đến 65535**).
  - Do NACL mới tạo chỉ có Inbound rule mở 80/443 và Outbound mặc định là DENY ALL, nên toàn bộ gói tin response trả về bị NACL Outbound chặn lại $\rightarrow$ Client bị Timeout.
  - **Cách khắc phục**: Thêm một Outbound Rule trên NACL: Protocol TCP, Port Range `1024-65535`, Destination: `0.0.0.0/0`, ALLOW.
- **Phân tích bẫy**:
  - *Phương án A*: Security Group mặc định đã mở All Outbound và SG là stateful nên không phải lỗi ở SG.
  - *Phương án C*: NACL trong AWS là stateless theo thiết kế hạ tầng, không thể cấu hình thành stateful.
  - *Phương án D*: NACL là tài nguyên logic gắn vào Subnet, không thể và không cần gán Elastic IP.
