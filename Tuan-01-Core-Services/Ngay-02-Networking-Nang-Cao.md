# NGÀY 2: NETWORKING NÂNG CAO (SG vs NACL, ENDPOINTS, PEERING, TGW, HYBRID CONNECTIVITY)
*(Tuần 1: Core Infrastructure - Trọng số Networking nâng cao)*

---

## 1. SO SÁNH CHIẾN LƯỢC: SECURITY GROUP (SG) vs NETWORK ACL (NACL)

Đây là một trong những chủ đề xuất hiện dày đặc nhất trong đề thi SAA-C03 (cả Domain 1 Security lẫn Domain 2 Resilience).

| Tiêu chí | Security Group (SG) | Network ACL (NACL) |
| :--- | :--- | :--- |
| **Phạm vi bảo vệ** | **Instance Level** (gắn vào Elastic Network Interface - ENI của EC2, RDS, ALB...) | **Subnet Level** (gắn vào Subnet, bảo vệ toàn bộ instance bên trong) |
| **Trạng thái (State)** | **Stateful**: Nếu Inbound cho phép, Outbound response tự động được cho phép (và ngược lại), bất kể outbound rule. | **Stateless**: Inbound và Outbound hoàn toàn độc lập. Phải mở cả 2 chiều tường minh. |
| **Loại Rule** | **Chỉ hỗ trợ ALLOW** (Không thể tạo Deny rule) | **Hỗ trợ cả ALLOW và DENY** |
| **Thứ tự đánh giá** | Đánh giá **đồng thời tất cả các rules** trước khi quyết định allow. | Đánh giá **theo số thứ tự từ nhỏ đến lớn** (Rule numbers 1-32766). Khớp rule nào dừng ngay tại rule đó (*). |
| **Xử lý Ephemeral Ports** | Tự động xử lý nhờ cơ chế Stateful. | **Bắt buộc phải mở Inbound/Outbound Ephemeral Ports (1024–65535)** cho traffic phản hồi từ client/server. |
| **Hành vi mặc định (Default)** | Default SG: Chặn mọi Inbound, Mở mọi Outbound. | Default NACL: Mở toàn bộ Inbound và Outbound (`* Allow`). |

> ⚠️ **BẪY THI KINH ĐIỂN CỦA AWS**:
> 1. **"Chặn một địa chỉ IP độc hại cụ thể" (Block a single malicious IP)**:
>    - **Đáp án bắt buộc**: **Network ACL (NACL)** với một **DENY rule** có số thứ tự nhỏ hơn rule Allow (ví dụ: Rule 50 Deny IP x.x.x.x, Rule 100 Allow All).
>    - *Tại sao SG sai?* Security Group **không thể tạo rule DENY**.
> 2. **Client gọi vào web server cổng 80/443 nhưng không nhận được phản hồi**:
>    - Nếu SG đã mở port 80/443, kiểm tra ngay **NACL Outbound Rule** đã mở dải port tạm thời **Ephemeral Ports (TCP 1024–65535)** hay chưa!

---

## 2. VPC ENDPOINTS: GATEWAY ENDPOINT vs INTERFACE ENDPOINT (PRIVATELINK)

VPC Endpoint cho phép kết nối an toàn từ VPC tới các dịch vụ AWS mà **không cần Internet Gateway, NAT Gateway, VPN hay Direct Connect**, traffic hoàn toàn đi trên đường trục nội bộ của AWS (AWS Global Network).

```
                      ┌──────────────────────────────────────┐
                      │            VPC ENDPOINTS             │
                      └──────────────────┬───────────────────┘
                                         │
             ┌───────────────────────────┴───────────────────────────┐
             ▼                                                       ▼
   [ Gateway Endpoint ]                                    [ Interface Endpoint ]
   • Chỉ hỗ trợ: S3 và DynamoDB                            • Hỗ trợ: Hầu hết dịch vụ AWS (>100 services)
   • Cấu hình: Cập nhật ROUTE TABLE                          và AWS S3
   • Không có ENI, không có Private IP                     • Tạo ENI trong Subnet với Private IP
   • CHI PHÍ: HOÀN TOÀN MIỄN PHÍ                           • Dùng AWS PrivateLink
   • Không truy cập được từ On-Premises                    • CHI PHÍ: Tính theo giờ (~$0.01/h) + GB
     (qua VPN/Direct Connect)                              • Truy cập được từ On-Premises (VPN/DX)
                                                           • Hỗ trợ Cross-Region & VPC Peering
```

### Bảng quyết định chọn loại Endpoint cho S3 (Cực kỳ quan trọng)

| Tình huống thực tế trong đề thi | Giải pháp Endpoint tối ưu | Lý do |
| :--- | :--- | :--- |
| EC2 trong Private Subnet cần đọc/ghi S3 với **chi phí thấp nhất** (*most cost-effective*) | **Gateway Endpoint for S3** | Hoàn toàn miễn phí, cấu hình qua Route Table. |
| Máy chủ On-Premises cần kết nối private an toàn tới S3 qua **Direct Connect hoặc VPN** | **Interface Endpoint for S3** | Gateway Endpoint không thể định tuyến traffic từ ngoài VPC (on-prem) vào. Phải dùng Interface Endpoint vì nó có Private IP nội bộ trong Subnet. |
| Ứng dụng trong VPC A cần truy cập S3 bucket nhưng đi qua **VPC Peering từ VPC B** | **Interface Endpoint for S3** | Gateway Endpoint không hỗ trợ kết nối transitive qua VPC Peering. |

---

## 3. VPC PEERING vs AWS TRANSIT GATEWAY (TGW)

```
       Mô hình Full-Mesh VPC Peering               Mô hình Hub-and-Spoke Transit Gateway
             (Khó quản lý khi N tăng)                      (Chuẩn Enterprise)

                   [VPC A]                                        [VPC A]
                  /   |   \                                          │
                 /    |    \                                         │
            [VPC B]───┼───[VPC C]                       [VPC B]───[Transit]───[VPC C]
                 \    |    /                                    [Gateway]
                  \   |   /                                          │
                   [VPC D]                                        [VPC D]
         Công thức kết nối: N*(N-1)/2                                │
         (Ví dụ: 10 VPCs cần 45 peerings)                 [Direct Connect / VPN]
```

### Bảng so sánh chuyên sâu:

| Tiêu chí | VPC Peering | AWS Transit Gateway (TGW) |
| :--- | :--- | :--- |
| **Mô hình** | Điểm - Điểm (Point-to-Point 1:1) | Hub-and-Spoke (Tập trung) |
| **Transitive Routing** | **KHÔNG HỖ TRỢ** (A nối B, B nối C thì A **không thể** đi qua B để sang C). | **HỖ TRỢ ĐẦY ĐỦ** (A có thể routing sang B, C, on-prem qua TGW). |
| **Quy mô** | Thích hợp cho ít VPC (2 - 5 VPCs). Cồng kềnh khi hệ thống lớn. | Hỗ trợ hàng ngàn VPCs và kết nối Hybrid on-premise. |
| **Hỗ trợ On-Premises** | Không nối trực tiếp được với VPN hay Direct Connect. | Hỗ trợ nối trực tiếp Site-to-Site VPN và Direct Connect Gateway. |
| **Chi phí** | **Không có phí duy trì** (Chỉ trả tiền Data Transfer). | Có phí duy trì theo giờ cho mỗi VPC attachment (~$0.05/attachment/h) + phí Data Transfer. |
| **Tính năng đặc biệt** | Hỗ trợ Inter-Region Peering. | Hỗ trợ **IP Multicast**, chia sẻ qua **AWS RAM**, nhiều Route Tables riêng biệt (Network Segmentation: Dev/Prod/Shared). |

---

## 4. KẾT NỐI HYBRID: AWS SITE-TO-SITE VPN vs DIRECT CONNECT (DX)

### 4.1. AWS Site-to-Site VPN
* **Đặc điểm**: Chạy qua **Public Internet**, được mã hóa bằng đường hầm IPsec (AES-256).
* **Thành phần**:
  * AWS side: **Virtual Private Gateway (VGW)** gắn vào 1 VPC, hoặc **Transit Gateway (TGW)** cho nhiều VPC.
  * On-Premise side: **Customer Gateway (CGW)** (thiết bị vật lý hoặc phần mềm).
* **Băng thông**: Tối đa **1.25 Gbps** cho mỗi tunnel (AWS cung cấp 2 tunnels dự phòng cho mỗi VPN connection).
* Có thể dùng **ECMP (Equal-Cost Multi-Path)** với Transit Gateway để ghép nhiều tunnels nhằm tăng băng thông tổng thể (> 1.25 Gbps).
* Ưu điểm: Triển khai nhanh trong vài phút, chi phí thấp.

### 4.2. AWS Direct Connect (DX)
* **Đặc điểm**: Kết nối vật lý **chuyên dụng (Dedicated Physical Link)** từ On-Premises trực tiếp vào AWS qua đối tác viễn thông (Direct Connect Location).
* **Băng thông**: 1 Gbps, 10 Gbps, 100 Gbps (hoặc sub-1Gbps qua đối tác APN).
* Ưu điểm: Băng thông cực lớn, độ trễ nhất quán (low jitter, predictable latency), giảm chi phí data transfer cho lưu lượng khổng lồ.
* Nhược điểm: Mất từ vài tuần đến hàng tháng để kéo cáp vật lý.
* ⚠️ **LƯU Ý BẢO MẬT ĐỀ THI**:
  - Direct Connect **KHÔNG MÃ HÓA MẶC ĐỊNH** (traffic chạy raw qua dedicated link).
  - Nếu đề bài yêu cầu *"Consistent dedicated bandwidth AND IPsec encryption in transit"*:
    $\rightarrow$ **Giải pháp**: Thiết lập **AWS Site-to-Site VPN chạy trên nền Direct Connect** (Public VIF hoặc Transit VIF), hoặc dùng **MACsec (IEEE 802.1AE)** trên đường link 10Gbps/100Gbps.

### 4.3. Các loại Virtual Interface (VIF) của Direct Connect:
1. **Private VIF**: Kết nối trực tiếp vào 1 VPC duy nhất (qua VGW).
2. **Public VIF**: Kết nối tới các dịch vụ public của AWS (S3, DynamoDB, SNS...) mà không cần qua Internet công cộng.
3. **Transit VIF**: Kết nối trực tiếp tới **AWS Transit Gateway** để vươn tới hàng trăm VPC.

### 4.4. Mô hình Dự phòng Failover (Direct Connect + VPN Backup)
* **Yêu cầu đề thi**: *"High bandwidth, low latency as primary, with a cost-effective backup for disaster recovery"*.
* **Kiến trúc**:
  - Đường chính (Primary): **AWS Direct Connect**.
  - Đường phụ (Backup): **AWS Site-to-Site VPN**.
  - Cơ chế failover: Dùng giao thức định tuyến động **BGP (Border Gateway Protocol)**. AWS sẽ tự động ưu tiên đường Direct Connect vì có đường đi ngắn hơn. Khi dùng BGP, có thể áp dụng **AS-Path Prepending** trên đường VPN để biến nó thành backup thụ động (Standby).

---

## 5. LAB KIẾN TRÚC: THIẾT KẾ PRIVATE ACCESS TỚI S3 & ĐƯỜNG ĐI TRAFFIC

### Tình huống Lab:
Một doanh nghiệp tài chính yêu cầu:
1. EC2 App trong Private Subnet cần upload báo cáo tài chính hàng ngày lên S3 bucket.
2. Tuyệt đối không để traffic đi ra ngoài Internet công cộng, không được dùng NAT Gateway (để tiết kiệm chi phí data transfer).
3. Đảm bảo chỉ các EC2 trong VPC này mới được quyền ghi vào S3 bucket đó, chặn mọi truy cập từ bên ngoài.

```
┌──────────────────────────────── VPC (10.0.0.0/16) ────────────────────────────────┐
│                                                                                   │
│   [ Private Subnet: 10.0.10.0/24 ]                                                │
│   ┌──────────────────────────────┐                                                │
│   │ EC2 Application Instance     │                                                │
│   │ (IAM Role: S3-PutObject)     │                                                │
│   └──────────────┬───────────────┘                                                │
│                  │                                                                │
│                  │ (1) Gửi request: s3.ap-southeast-1.amazonaws.com               │
│                  ▼                                                                │
│   [ Route Table của Private Subnet ]                                              │
│   ┌─────────────────────────────────────────────────────────┐                     │
│   │ Destination          │ Target                           │                     │
│   │ 10.0.0.0/16          │ local                            │                     │
│   │ pl-63a5400a (S3 CIDR)│ vpce-0123456789abcdef0 (Gateway) ├───┐                 │
│   └─────────────────────────────────────────────────────────┘   │                 │
│                                                                 │                 │
└─────────────────────────────────────────────────────────────────┼─────────────────┘
                                                                  │ (2) Traffic đi qua
                                                                  │ AWS Private Backbone
                                                                  ▼ (Hoàn toàn FREE)
                                                     ┌────────────────────────────┐
                                                     │      Amazon S3 Bucket      │
                                                     │    "finance-reports-prod"  │
                                                     └────────────────────────────┘
```

### Chính sách bảo mật 2 lớp bắt buộc (Defense in Depth):
1. **Endpoint Policy (trên Gateway Endpoint)**: Giới hạn Endpoint này CHỈ được phép gửi request tới đúng bucket `finance-reports-prod`:
```json
{
  "Statement": [
    {
      "Sid": "RestrictToSpecificBucket",
      "Principal": "*",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::finance-reports-prod/*"
    }
  ]
}
```

2. **Bucket Policy (trên S3 Bucket)**: Từ chối mọi request nếu KHÔNG đi qua Gateway Endpoint ID (`vpce-xxx`) của VPC:
```json
{
  "Statement": [
    {
      "Sid": "DenyNonVPCEndpointTraffic",
      "Principal": "*",
      "Action": "s3:*",
      "Effect": "Deny",
      "Resource": [
        "arn:aws:s3:::finance-reports-prod",
        "arn:aws:s3:::finance-reports-prod/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "vpce-0123456789abcdef0"
        }
      }
    }
  ]
}
```
