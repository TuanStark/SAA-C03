# NGÀY 4: HIGH AVAILABILITY & TRAFFIC ROUTING (ELB, ASG, ROUTE 53, CLOUDFRONT, GLOBAL ACCELERATOR)
*(Tuần 1: Core Infrastructure - Trọng số Domain 2: Resilient Architectures & Domain 3: High-Performing Architectures)*

---

## 1. ELASTIC LOAD BALANCING (ELB): PHÂN LOẠI & SO SÁNH CHUYÊN SÂU

AWS cung cấp 4 loại Load Balancer, trong đó 3 loại chính xuất hiện liên tục trong đề thi SAA-C03 là **ALB (Application Load Balancer)**, **NLB (Network Load Balancer)**, và **GLB (Gateway Load Balancer)** (Classic Load Balancer - CLB đã cũ, AWS khuyến nghị không sử dụng).

```
                             ┌──────────────────────────────────────┐
                             │       ELASTIC LOAD BALANCING         │
                             └──────────────────┬───────────────────┘
                                                │
             ┌──────────────────────────────────┼──────────────────────────────────┐
             ▼                                  ▼                                  ▼
   [ Application Load Balancer ]      [ Network Load Balancer ]          [ Gateway Load Balancer ]
   • Layer 7 (HTTP / HTTPS / gRPC)    • Layer 4 (TCP / UDP / TLS)        • Layer 3 (IP) + GENEVE (Port 6081)
   • Định tuyến thông minh nội dung   • Hiệu năng cực khủng (Millions)   • Triển khai cụm Firewall / IDS
   • Hỗ trợ WebSockets, Redirects     • Độ trễ siêu thấp (Microsecond)   • Định tuyến trong suốt (Transparent)
   • Sticky Sessions, SSL/TLS, WAF    • Cung cấp Static / Elastic IP     • Phối hợp với Route Table & VPC
```

### 1.1. Ma trận so sánh chi tiết: ALB vs NLB vs GLB

| Tiêu chí | Application Load Balancer (ALB) | Network Load Balancer (NLB) | Gateway Load Balancer (GLB) |
| :--- | :--- | :--- | :--- |
| **Tầng mạng (OSI)** | **Layer 7** (Application) | **Layer 4** (Transport) | **Layer 3 Gateway + Layer 4** |
| **Giao thức hỗ trợ** | HTTP, HTTPS, gRPC | TCP, UDP, TLS | IP packets (GENEVE encapsulation) |
| **Tốc độ & Độ trễ** | ~ Hàng chục millisecond | **Độ trễ microsecond**, hàng triệu req/giây | Tùy thuộc vào thiết bị appliance |
| **Khả năng định tuyến** | **Theo nội dung (Path, Host, Query, HTTP Headers, Methods)** | Chỉ dựa trên IP, Port và TCP/UDP | Chuyển toàn bộ traffic qua cụm Appliance kiểm tra |
| **Địa chỉ IP** | **IP động (Dynamic IP)**, chỉ truy cập qua **DNS Name** | **Cung cấp Static IP cố định hoặc Elastic IP cho mỗi AZ** | ENI với Private IP trong subnet |
| **Bảo toàn Client IP** | Ghi đè IP client vào header `X-Forwarded-For` | **Bảo toàn Source IP nguyên bản** của client tới target | Giữ nguyên gói tin gốc bên trong GENEVE wrapper |
| **Loại Target Groups** | EC2, ECS (Fargate/EC2), Lambda, IP addresses | EC2, ECS, IP addresses, **Application Load Balancer** | EC2, IP addresses (Virtual Appliances) |
| **Tích hợp bảo mật** | **AWS WAF**, AWS Shield, ACM (SSL/TLS), OIDC/Cognito | AWS Shield, ACM (TLS Listener) | Tích hợp cụm Firewall bên thứ 3 (Palo Alto, Fortinet) |

---

### 1.2. Application Load Balancer (ALB) - Deep Dive

#### A. Các quy tắc định tuyến nội dung (Advanced Routing Rules)
ALB cho phép kiểm tra chi tiết gói tin HTTP/HTTPS ở Layer 7 để chuyển tiếp đến các **Target Group** khác nhau:
1. **Path-based routing**: Định tuyến dựa trên đường dẫn URL (ví dụ: `/api/*` trỏ về API Target Group, `/static/*` trỏ về Static Target Group).
2. **Host-based routing**: Định tuyến dựa trên domain header (ví dụ: `app.company.com` sang Target Group A, `admin.company.com` sang Target Group B).
3. **HTTP Header & Query String routing**: Định tuyến dựa theo tham số truy vấn hoặc header đặc thù (ví dụ: `?version=v2`, `User-Agent: Mobile`).
4. **HTTP Request Method routing**: Định tuyến theo phương thức `GET`, `POST`, `DELETE`, `PUT`.

#### B. Sticky Sessions (Session Affinity)
* **Khái niệm**: Đảm bảo toàn bộ request từ một người dùng cụ thể trong suốt phiên làm việc luôn được chuyển đến **cùng một target backend** duy nhất.
* **Cơ chế hoạt động**: Sử dụng **Cookie**.
  * **Duration-based cookie (do ALB sinh ra)**: Tên cookie `AWSALB`, có thời hạn từ 1 giây đến 7 ngày do bạn cấu hình.
  * **Application-based cookie (do ứng dụng backend sinh ra)**: ALB dựa vào cookie tùy chỉnh của ứng dụng để gắn phiên.
* ⚠️ **Lưu ý kiến trúc**: Sticky sessions có thể gây ra hiện tượng mất cân bằng tải (unbalanced load) giữa các instance backend.

#### C. SSL/TLS Termination & SNI (Server Name Indication)
* **SSL Termination**: ALB giải mã HTTPS tại Load Balancer, giúp giảm tải CPU cho các máy chủ EC2 phía sau. Traffic từ ALB về backend có thể là HTTP nội bộ.
* **SNI (Server Name Indication)**: Cho phép ALB lưu trữ **nhiều chứng chỉ SSL/TLS** trên cùng một HTTPS Listener (Port 443). Khi client gửi request, hostname trong TLS handshake giúp ALB tự động chọn chứng chỉ phù hợp với website mà client muốn truy cập.

#### D. Các HTTP Headers đặc thù cần nhớ trong đề thi
* `X-Forwarded-For`: Chứa địa chỉ **IP gốc thực sự của Client** (do ALB thay thế Source IP của gói tin bằng Private IP của ALB).
* `X-Forwarded-Proto`: Cho biết giao thức client đã sử dụng để kết nối tới ALB (`http` hoặc `https`).
* `X-Forwarded-Port`: Cổng kết nối client đã gửi request tới (ví dụ: `80` hoặc `443`).

---

### 1.3. Network Load Balancer (NLB) - Deep Dive

* **Hiệu năng cực đại**: Xử lý hàng chục triệu request mỗi giây với độ trễ siêu thấp (sub-millisecond latency).
* **Đặc tính Static IP / Elastic IP**:
  * Mỗi AZ mà NLB kích hoạt sẽ được cấp **1 địa chỉ IP tĩnh cố định (Static IP)** hoặc bạn có thể gán **Elastic IP**.
  * 👉 *Tình huống đề thi*: Ứng dụng yêu cầu khách hàng doanh nghiệp phải **whitelist danh sách IP cố định trên tường lửa (Firewall whitelist)** $\rightarrow$ **Bắt buộc chọn NLB** (ALB không thể làm được vì ALB chỉ có DNS name và IP thay đổi liên tục).
* **Bảo toàn Client Source IP (Preserve Source IP)**:
  * Khi client gửi TCP packet, NLB không thay đổi Source IP. Instance backend sẽ nhận được trực tiếp IP thật của client ở tầng socket mà không cần phụ thuộc vào HTTP header.
* **Hỗ trợ AWS PrivateLink**:
  * NLB là thành phần bắt buộc để làm front-end cho một **VPC Endpoint Service**, cho phép chia sẻ dịch vụ private tới hàng nghìn VPC của khách hàng khác mà không cần peering hay routing.

---

### 1.4. Gateway Load Balancer (GLB) - Deep Dive

```
                                      AWS TRANSIT GATEWAY / ROUTE TABLE
                                                     │
                                 ┌───────────────────┴───────────────────┐
                                 ▼                                       ▼
                       [ Người dùng Internet ]                  [ VPC Ứng Dụng ]
                                 │                                       ▲
                                 ▼ (Định tuyến trong suốt)               │ (Sau khi quét an toàn)
                    ┌───────────────────────────┐                        │
                    │ Gateway Load Balancer     │                        │
                    └─────────────┬─────────────┘                        │
                                  │ (Giao thức GENEVE Port 6081)         │
                                  ▼                                      │
                   ┌─────────────────────────────┐                       │
                   │ Cụm EC2 3rd Party Firewalls │───────────────────────┘
                   │ (Palo Alto, Checkpoint...)  │
                   └─────────────────────────────┘
```

* **Mục đích**: Triển khai, nhân rộng quy mô và quản lý một nhóm các thiết bị ảo của bên thứ ba (Third-party Virtual Appliances) như: Firewalls, IDS/IPS, Deep Packet Inspection systems.
* **Nguyên lý hoạt động**:
  * Kết hợp tính năng của một **Transparent Network Gateway** (điểm đón traffic vào/ra duy nhất) và **Load Balancer** (phân phối tải tới các firewall EC2).
  * Sử dụng giao thức **GENEVE trên Port 6081** để đóng gói toàn bộ gói tin IP gốc kèm theo metadata. Sau khi firewall quét xong, gói tin được trả lại GLB để đi tiếp tới đích mà không làm thay đổi header mạng.

---

### 1.5. Các tính năng cốt lõi chung của ELB

| Tính năng | Đặc tính kỹ thuật | Hành vi mặc định theo loại ELB |
| :--- | :--- | :--- |
| **Cross-Zone Load Balancing** | Cho phép load balancer phân phối traffic đều cho tất cả các target trong **mọi AZ**, thay vì chỉ trong AZ mà node đó cư ngụ. | • **ALB**: Mặc định **BẬT (Always ON)**, miễn phí data transfer.<br>• **NLB**: Mặc định **TẮT**, nếu bật sẽ tính phí Inter-AZ Data Transfer. |
| **Deregistration Delay (Connection Draining)** | Thời gian chờ (từ 1 đến 3600 giây, mặc định **300 giây**) để các kết nối hiện tại (in-flight requests) hoàn thành trước khi gỡ target bị unhealth hoặc bị terminate. | Áp dụng cho cả ALB và NLB. |
| **Health Checks** | Load Balancer gửi gói ping định kỳ (HTTP/HTTPS/TCP) tới endpoint kiểm tra sức khỏe của target. Nếu target fail ngưỡng `UnhealthyThreshold`, ELB sẽ **dừng gửi traffic mới** tới target đó. | Tự động chuyển hướng traffic sang các target còn khỏe mạnh mà không làm gián đoạn dịch vụ. |

---

## 2. AUTO SCALING GROUP (ASG): CO GIÃN TỰ ĐỘNG & ĐỘ SẴN SÀNG CAO

Auto Scaling Group tự động duy trì tính sẵn sàng của ứng dụng bằng cách bổ sung hoặc gỡ bỏ các EC2 instances dựa theo nhu cầu thực tế và theo dõi tình trạng sức khỏe của máy chủ.

```text
              Min Capacity = 2        Desired Capacity = 4         Max Capacity = 8
           ┌──────────────────────┬────────────────────────────┬──────────────────────┐
           │   Instance 1 (AZ-A)  │     Instance 3 (AZ-A)      │   [ Dự phòng mở rộng │
           │   Instance 2 (AZ-B)  │     Instance 4 (AZ-B)      │    khi có tải cao ]  │
           └──────────────────────┴────────────────────────────┴──────────────────────┘
              (Tải tối thiểu luôn     (Số lượng instance           (Giới hạn tối đa
               duy trì)                đang chạy thực tế)           ngăn vượt ngân sách)
```

### 2.1. Cấu hình nền tảng: Launch Template vs Launch Configuration
* **Launch Configuration**: Công nghệ cũ (Legacy), không hỗ trợ versioning, không hỗ trợ mua hỗn hợp On-Demand và Spot, không thể sửa đổi sau khi tạo. Đã bị AWS **Deprecate**.
* **Launch Template (Bắt buộc dùng trong đề thi)**:
  * Hỗ trợ **Versioning** (quản lý phiên bản cấu hình v1, v2...).
  * Cho phép chỉ định nhiều Instance Types và cấu hình **Mixed Instances Policy** (kết hợp On-Demand và Spot).
  * Hỗ trợ tính năng T2/T3 Unlimited, Provisioning Rules, IAM Role, User Data script.

---

### 2.2. Các chính sách co giãn (Scaling Policies)

| Loại Policy | Cơ chế hoạt động | Kịch bản sử dụng trong đề thi |
| :--- | :--- | :--- |
| **Target Tracking Scaling** | Bạn chỉ định một chỉ số mục tiêu cụ thể, ASG tự động tính toán tăng/giảm để duy trì chỉ số đó.<br>Ví dụ: *Duy trì `ASGAverageCPUUtilization = 50%`* hoặc *`ALBRequestCountPerTarget = 1000`*. | **Cách đơn giản và tối ưu nhất** khi muốn giữ hiệu năng ổn định mà không cần tự cấu hình từng ngưỡng CloudWatch Alarm phức tạp. |
| **Step Scaling** | Mở rộng theo từng bậc dựa trên mức độ vi phạm CloudWatch Alarm.<br>Ví dụ: CPU > 60% thêm 2 máy, CPU > 80% thêm 4 máy, CPU < 40% giảm 2 máy. | Phản ứng nhanh với sự thay đổi tải đột ngột, xử lý theo từng cấp độ tải khẩn cấp. |
| **Simple Scaling** | Kích hoạt thêm/bớt N máy khi có cảnh báo CloudWatch. Phải chờ qua thời gian làm nguội (**Cooldown Period**) mới được thực hiện hành động scaling tiếp theo. | Dạng scaling truyền thống, phản ứng chậm hơn Step Scaling. |
| **Scheduled Scaling** | Lên lịch co giãn dựa theo thời gian biểu biết trước.<br>Ví dụ: *Scale out lên 10 instances lúc 8:00 sáng Thứ Hai, scale in về 2 instances lúc 18:00 chiều*. | Ứng dụng có mô hình truy cập tuần hoàn rõ rệt, sự kiện khuyến mãi, giờ làm việc hành chính. |
| **Predictive Scaling** | Sử dụng **Machine Learning** phân tích dữ liệu lịch sử tải của các tuần trước để dự báo tải trong 24-48 giờ tới và **scale đón đầu trước khi tải ập đến**. | Hệ thống mất nhiều thời gian để khởi động (warm up) EC2 và có quy luật tải lặp lại hàng tuần. |

---

### 2.3. Các cơ chế vận hành chuyên sâu của ASG

#### A. Scaling Cooldown Period
* **Bản chất**: Khoảng thời gian tạm nghỉ (mặc định **300 giây**) sau khi một hành động co giãn (Scale-in hoặc Scale-out) hoàn tất.
* **Mục đích sống còn**: Trong thời gian cooldown, ASG sẽ **tạm dừng các yêu cầu scaling khác** để cho phép các instance mới khởi chạy hoàn tất, nạp cache và tiếp nhận tải thực tế. Ngăn chặn hiện tượng **Scaling Thrashing** (bật tắt máy chủ liên tục ngoài tầm kiểm soát).

#### B. ASG Lifecycle Hooks
Cho phép tạm dừng quá trình khởi tạo hoặc hủy instance để thực hiện các tác vụ tùy biến trước khi instance chính thức phục vụ hoặc bị xóa bỏ:

```text
EC2 Launch ──► [ Pending ] ──► [ Pending:Wait ] ──► [ InService ]
                                (Lifecycle Hook: Cài đặt phần mềm, nạp warm cache)

EC2 Terminate ──► [ Terminating ] ──► [ Terminating:Wait ] ──► [ Terminated ]
                                      (Lifecycle Hook: Đẩy log lên S3, dọn dẹp kết nối)
```

* **`autoscaling:EC2_INSTANCE_LAUNCHING`**: Instance dừng ở trạng thái `Pending:Wait`. Bạn có thể chạy script kiểm tra, nạp warm database cache. Sau khi hoàn thành gọi lệnh `CompleteLifecycleAction` để đưa vào `InService`.
* **`autoscaling:EC2_INSTANCE_TERMINATING`**: Instance dừng ở trạng thái `Terminating:Wait`. Bạn có thể gửi notification (SNS), lấy file log cuối cùng đẩy lên S3 hoặc drain transaction.

#### C. ASG Termination Policy (Thứ tự gỡ bỏ instance khi Scale-in)
Khi ASG cần giảm bớt số lượng instance, AWS thực hiện theo thuật toán mặc định (**Default Termination Policy**):
1. **Bước 1**: Tìm xem **Availability Zone (AZ) nào đang có nhiều instance nhất** và có ít nhất 1 instance không được bảo vệ khỏi scale-in.
2. **Bước 2**: Trong AZ đó, tìm instance sử dụng **Launch Template / Launch Configuration phiên bản cũ nhất**.
3. **Bước 3**: Nếu vẫn trùng nhau, chọn instance **gần chạm đến giờ tính phí tiếp theo nhất** (tiết kiệm chi phí).
4. **Bước 4**: Nếu vẫn trùng nhau, chọn ngẫu nhiên.

#### D. Health Check: EC2 Status Check vs ELB Health Check
* **EC2 Status Check (Mặc định)**: ASG chỉ coi instance là Unhealthy nếu phần cứng vật lý hoặc OS bị treo (System/Instance status check fail). **Nếu Web Server (Nginx/Apache) bị crash và trả về lỗi 502/503**, EC2 Status Check vẫn báo Healthy $\rightarrow$ **Người dùng vẫn gặp lỗi**.
* **ELB Health Check**:
  * Khi gắn ASG vào ELB, bạn phải cấu hình: `Health Check Type = ELB`.
  * Nếu target trả về mã lỗi HTTP không hợp lệ trên health check path, ELB sẽ báo Unhealthy. **ASG lập tức nhận biết và tự động terminate instance lỗi này để tạo một instance mới thay thế**.

---

## 3. AMAZON ROUTE 53: DNS CHUYÊN SÂU & CHIẾN LƯỢC ĐỊNH TUYẾN

Amazon Route 53 là dịch vụ DNS (Domain Name System) có độ sẵn sàng cao và khả năng scale vô hạn với cam kết SLA **100% Availability**.

### 3.1. Các loại bản ghi quan trọng: CNAME vs ALIAS

| Tiêu chí | CNAME Record | ALIAS Record (Độc quyền AWS) |
| :--- | :--- | :--- |
| **Bản chất** | Chuẩn DNS quốc tế. Trỏ một hostname này sang một hostname khác (ví dụ: `www.example.com -> myapp.elb.amazonaws.com`). | Tính năng mở rộng của Route 53. Trỏ trực tiếp tên miền tới một tài nguyên AWS (ALB, CloudFront, S3 Bucket, API Gateway). |
| **Zone Apex (Root domain)** | **KHÔNG HỖ TRỢ** trên root domain (ví dụ: không thể tạo CNAME cho `example.com`). | **HỖ TRỢ TOÀN DIỆN** cho Zone Apex (`example.com`). |
| **Chi phí truy vấn DNS** | Trả phí cho mỗi lần truy vấn DNS tiêu chuẩn. | **HOÀN TOÀN MIỄN PHÍ** khi truy vấn tới các tài nguyên AWS được hỗ trợ Alias. |
| **Nhận biết thay đổi IP** | Client phải truy vấn 2 lần (Query CNAME $\rightarrow$ Query A record của target). | Route 53 tự động cập nhật IP của tài nguyên AWS và trả về trực tiếp IP cho client trong 1 truy vấn duy nhất. |

> ⚠️ **BẪY THI CỰC QUAN TRỌNG**: Nếu đề thi yêu cầu cấu hình DNS cho tên miền gốc (Naked domain / Zone Apex - ví dụ `company.com`) trỏ tới một **Application Load Balancer** hoặc **CloudFront Distribution**:
> - **Đáp án đúng duy nhất**: Tạo một **A record với tính năng ALIAS = Yes** trỏ tới ALB/CloudFront.
> - *Không bao giờ được chọn CNAME cho Zone Apex*.

---

### 3.2. Bảng quyết định 7 chính sách định tuyến (Route 53 Routing Policies)

```
                            ┌────────────────────────────────────┐
                            │    ROUTE 53 ROUTING POLICIES       │
                            └─────────────────┬──────────────────┘
                                              │
         ┌──────────────────┬─────────────────┼─────────────────┬──────────────────┐
         ▼                  ▼                 ▼                 ▼                  ▼
    [ Simple ]        [ Weighted ]       [ Latency ]       [ Failover ]     [ Geolocation ]
   Một hoặc nhiều     Phân chia % lưu    Tự động chọn      Active-Passive    Theo vị trí địa
   IP không check     lượng (A/B Test    Region có ping    Dự phòng sự cố    lý người dùng
   sức khỏe target    Blue-Green dep)    thấp nhất         DR (Multi-Region) (Châu lục, nước)
```

| Routing Policy | Cơ chế kỹ thuật | Trường hợp sử dụng tối ưu trong bài thi SAA |
| :--- | :--- | :--- |
| **Simple Routing** | Trả về một hoặc một danh sách địa chỉ IP ngẫu nhiên. Không hỗ trợ Health Check nâng cao. | Cấu hình DNS cơ bản cho một tài nguyên đơn lẻ, không yêu cầu dự phòng phức tạp. |
| **Weighted Routing** | Gán trọng số (ví dụ: 80% traffic vào hệ thống v1, 20% traffic vào hệ thống v2). | **A/B Testing**, kiểm thử tính năng mới, triển khai an toàn theo kiểu Canary hoặc Blue-Green deployment. |
| **Latency-based Routing** | Route 53 đo lường độ trễ mạng giữa client và các AWS Regions, sau đó **chuyển hướng client tới Region có latency thấp nhất**. | Hệ thống phục vụ toàn cầu yêu cầu **tốc độ truy cập nhanh nhất (lowest latency)** cho người dùng ở các khu vực khác nhau. |
| **Failover Routing** | Cấu hình theo mô hình **Active-Passive Disaster Recovery**: Toàn bộ traffic đổ về Primary. Khi Health Check của Primary fail, Route 53 tự động chuyển toàn bộ traffic sang Secondary (Disaster Recovery site / Static error page trên S3). | Xây dựng kiến trúc khắc phục thảm họa (Disaster Recovery) với thời gian downtime tối thiểu. |
| **Geolocation Routing** | Định tuyến dựa trên **vị trí địa lý thực tế của người dùng** (theo Châu lục, Quốc gia, hoặc Bang tại Mỹ). | Phân phối nội dung theo bản quyền ngôn ngữ (User Pháp thấy giao diện tiếng Pháp), tuân thủ luật bảo vệ dữ liệu (GDPR). |
| **Geoproximity Routing** | Định tuyến dựa trên khoảng cách địa lý giữa người dùng và tài nguyên AWS, kết hợp tham số **Bias** để mở rộng hoặc thu hẹp vùng phủ sóng. | Muốn dịch chuyển ranh giới traffic linh hoạt giữa các Data Center. (Bắt buộc dùng **Route 53 Traffic Flow**). |
| **Multi-Value Answer** | Trả về tối đa **8 bản ghi IP khỏe mạnh (Healthy Records)** cho client kèm theo kiểm tra Health Check. Client sẽ tự chọn IP để kết nối. | Cung cấp cân bằng tải DNS đơn giản ở phía Client (Client-side load balancing) có kèm dự phòng lỗi. |

---

## 4. AMAZON CLOUDFRONT vs AWS GLOBAL ACCELERATOR: PHÂN BIỆT ĐỈNH CAO

Cả hai dịch vụ đều sử dụng mạng lưới hạ tầng toàn cầu (Global Edge Network) gồm hàng trăm trạm PoP (Point of Presence) của AWS để tăng tốc độ truy cập cho người dùng trên toàn thế giới, nhưng **bản chất hoạt động hoàn toàn khác nhau**.

```
                         ┌──────────────────────────────────────────────┐
                         │         AWS GLOBAL NETWORK SOLUTIONS         │
                         └──────────────────────┬───────────────────────┘
                                                │
                 ┌──────────────────────────────┴──────────────────────────────┐
                 ▼                                                             ▼
     [ Amazon CloudFront (CDN) ]                                  [ AWS Global Accelerator ]
     • CÓ BỘ NHỚ ĐỆM (Content Caching)                            • HOÀN TOÀN KHÔNG CACHING
     • Tối ưu cho HTTP / HTTPS                                    • Tối ưu cho TCP & UDP (Game, VoIP, IoT)
     • Edge Locations lưu trữ nội dung tĩnh/động                  • Cung cấp 2 địa chỉ Static Anycast IP cố định
     • Bảo vệ S3 qua Origin Access Control (OAC)                  • Đi vào mạng cáp quang riêng của AWS từ Edge
     • Xử lý logic tại biên: CloudFront Functions / Lambda@Edge   • Failover cực nhanh giữa các Region (< 1 phút)
```

### 4.1. Bảng so sánh toàn diện: CloudFront vs Global Accelerator

| Tiêu chí so sánh | Amazon CloudFront | AWS Global Accelerator |
| :--- | :--- | :--- |
| **Tính năng Caching** | **CÓ (Đây là dịch vụ CDN cốt lõi)**: Lưu trữ bản cache của ảnh, video, HTML, API response tại Edge. | **KHÔNG CÓ CACHING**: Không lưu trữ bất kỳ dữ liệu nào. Chỉ tăng tốc độ truyền tải gói tin qua mạng cáp quang AWS. |
| **Giao thức hỗ trợ** | Chỉ hỗ trợ giao thức web: **HTTP, HTTPS, WebSockets**. | Hỗ trợ cả **TCP, UDP** (bất kỳ giao thức tầng 4 nào) và HTTP/HTTPS. |
| **Địa chỉ IP cung cấp** | **DNS Name** (`*.cloudfront.net`). Các Edge Location có IP động thay đổi liên tục. | **Cung cấp 2 địa chỉ Static Anycast IP duy nhất và cố định** trên toàn cầu. |
| **Xử lý Logic tại Biên** | **Có**: Hỗ trợ **CloudFront Functions** (JS siêu nhẹ) và **Lambda@Edge** (NodeJS/Python phức tạp). | **Không**: Không hỗ trợ chạy code serverless tại Edge. |
| **Khả năng Failover** | Có thể cấu hình Origin Groups để failover giữa 2 origin HTTP/S3. | **Failover cực nhanh (dưới 30 giây)** giữa các endpoint ở nhiều AWS Region khác nhau nhờ Anycast IP và Health Checks liên tục. |
| **Trường hợp sử dụng điển hình (Use Cases)** | • Phân phối video/ảnh/website tĩnh từ **S3**.<br>• Giảm tải request cho EC2/ALB bằng cách cache HTTP API.<br>• Chặn tấn công DDoS ở tầng Web (WAF, Shield). | • Ứng dụng chơi game thời gian thực (Gaming UDP).<br>• Dịch vụ gọi thoại/video (VoIP).<br>• Thiết bị IoT giao tiếp qua TCP/UDP.<br>• Ứng dụng doanh nghiệp cần **IP tĩnh cố định để whitelist**. |

---

### 4.2. Amazon CloudFront - Deep Dive

#### A. Nguồn cấp dữ liệu (Origins) & Cơ chế bảo mật Origin
* **Origins được hỗ trợ**:
  * **Amazon S3 Bucket**: Phổ biến nhất. Dùng để lưu trữ assets tĩnh.
  * **Custom Origin**: Bất kỳ máy chủ web công khai nào hỗ trợ HTTP/HTTPS (ALB, EC2 Instance, API Gateway, máy chủ On-Premises).
* **Bảo vệ an toàn cho S3 Origin**:
  * ⚠️ Không bao giờ mở public bucket S3 cho internet.
  * Bắt buộc sử dụng **Origin Access Control (OAC)**: Tính năng bảo mật hiện đại nhất của AWS (thay thế cho tính năng cũ OAI - Origin Access Identity).
  * OAC sử dụng AWS IAM Service Principal `cloudfront.amazonaws.com` trong S3 Bucket Policy, đảm bảo người dùng **chỉ có thể đọc S3 thông qua CloudFront**.

#### B. So sánh Edge Serverless: CloudFront Functions vs Lambda@Edge

| Tiêu chí | CloudFront Functions | Lambda@Edge |
| :--- | :--- | :--- |
| **Ngôn ngữ hỗ trợ** | Chỉ hỗ trợ **JavaScript (ECMAScript 5.1+)** | **Node.js** và **Python** |
| **Vị trí chạy** | Trực tiếp tại **225+ Edge Locations (Viewer side)** | Tại **Regional Edge Caches** |
| **Thời gian thực thi** | Dưới **1 millisecond** (Scale hàng triệu req/s) | Lên tới 5 - 10 giây (Tốn tài nguyên hơn) |
| **Truy cập Body & Network** | **KHÔNG**: Không thể truy cập request body, không thể gọi API ra mạng ngoài. | **CÓ**: Có thể đọc/ghi request body, truy cập internet và các service AWS khác. |
| **Use cases đề thi** | Chuẩn hóa URL, rewrite/redirect URL, kiểm tra/thêm HTTP headers, xác thực JWT đơn giản. | Biến đổi ảnh động (resize on the fly), tương tác database từ Edge, xử lý logic phức tạp. |

---

## 5. MA TRẬN TỔNG HỢP KIẾN TRÚC TOÀN CẦU (WELL-ARCHITECTED CHEAT SHEET)

Khi gặp các câu hỏi kết hợp nhiều thành phần trong đề thi SAA-C03, áp dụng ngay bảng tra cứu nhanh sau:

| Yêu cầu của bài toán | Sự kết hợp dịch vụ tối ưu (Best Practice) |
| :--- | :--- |
| **Website tĩnh (Static Web) có người dùng toàn cầu, yêu cầu chi phí rẻ nhất và bảo mật cao** | **Route 53 (Alias A record)** $\rightarrow$ **CloudFront (bật OAC)** $\rightarrow$ **Amazon S3 Bucket (Private, khóa toàn bộ public access)**. |
| **Hệ thống Web động (Dynamic Web) chịu tải cao, cần tự phục hồi khi có sự cố AZ** | **Route 53 (Latency/Geolocation)** $\rightarrow$ **CloudFront (Cache static + Bypass dynamic)** $\rightarrow$ **ALB (Multi-AZ)** $\rightarrow$ **ASG (EC2 trong Private Subnets, bật ELB Health Check)**. |
| **Ứng dụng trò chơi (Gaming) hoặc VoIP chạy giao thức UDP, cần độ trễ thấp nhất toàn cầu** | **Route 53** $\rightarrow$ **AWS Global Accelerator (2 Static Anycast IPs)** $\rightarrow$ **Network Load Balancer (NLB)** $\rightarrow$ **EC2 Instances**. |
| **Hệ thống yêu cầu khách hàng doanh nghiệp phải Whitelist địa chỉ IP tĩnh trên tường lửa** | **AWS Global Accelerator** (nếu cần scale toàn cầu/multi-region) HOẶC **Network Load Balancer với Elastic IP** (trong 1 Region đơn lẻ). *(Không bao giờ chọn ALB)*. |
| **Triển khai cụm tường lửa (Firewall Appliance) để kiểm tra toàn bộ traffic mạng vào/ra VPC** | **Gateway Load Balancer (GLB)** kết hợp **AWS Transit Gateway** và giao thức **GENEVE**. |
| **Mô hình Khắc phục thảm họa (Disaster Recovery) Active-Passive giữa 2 AWS Regions** | **Route 53 Failover Routing Policy** kết hợp **Route 53 Health Checks** giám sát Primary Region. |
