# NGÀY 1: VPC, SUBNET, ROUTE TABLE, IGW, NAT GATEWAY
*(Tuần 1: Core Infrastructure - Trọng số Networking)*

---

## 1. KIẾN THỨC CỐT LÕI (CORE CONCEPTS & EXAM FOCUS)

### 1.1. VPC & CIDR Block Rules
* **Kích thước VPC**: Giới hạn từ `/16` (65,536 IPs) đến `/28` (16 IPs).
* Khuyến nghị dùng private IP ranges theo chuẩn RFC 1918:
  * `10.0.0.0/8`
  * `172.16.0.0/12`
  * `192.168.0.0/16`
* **VPC CIDR không thể bị thu hẹp hoặc mở rộng trực tiếp** sau khi tạo, nhưng bạn có thể **thêm Secondary CIDR blocks** (IPv4 & IPv6).
* Các VPC hoặc subnet có CIDR trùng lặp (**overlapping**) sẽ **KHÔNG THỂ kết nối** qua VPC Peering, Transit Gateway hoặc AWS Site-to-Site VPN.

### 1.2. 5 IP Addresses bị AWS giữ lại trong mỗi Subnet
Nếu một subnet có CIDR `/24` (256 địa chỉ), thực tế bạn chỉ dùng được **251 địa chỉ**. AWS luôn giữ lại 5 địa chỉ IP đầu và cuối:
1. `x.x.x.0`: Network address.
2. `x.x.x.1`: AWS dành cho **VPC Router**.
3. `x.x.x.2`: AWS dành cho **Amazon DNS (AmazonProvidedDNS / Route 53 Resolver)**.
4. `x.x.x.3`: AWS giữ lại để sử dụng nội bộ trong tương lai.
5. `x.x.x.255`: Network Broadcast address (AWS VPC không hỗ trợ broadcast thực thụ, nhưng vẫn giữ IP này).

> ⚠️ **Bẫy thi SAA**: Đề bài hỏi cần triển khai tối thiểu 200 instances thì chọn Subnet CIDR nào? Nếu chọn `/24` (251 usable) là đủ, nhưng nếu cần 254 instances thì `/24` sẽ **thiếu**, bắt buộc phải dùng `/23`.

---

### 1.3. Subnet Types & Route Tables
* **Public Subnet**: Subnet có Route Table trỏ default route (`0.0.0.0/0`) tới **Internet Gateway (IGW)**. Instance trong public subnet phải có Public IPv4 hoặc Elastic IP để đi ra ngoài internet.
* **Private Subnet**: Subnet có Route Table trỏ `0.0.0.0/0` tới **NAT Gateway / NAT Instance** (nếu cần egress internet) hoặc KHÔNG có route internet nào (Isolated Subnet).
* **Quy tắc Route Table (Longest Prefix Match)**:
  * Khi có nhiều route khớp với destination, AWS sẽ luôn ưu tiên **route có prefix dài nhất (cụ thể nhất)**.
  * Local route (ví dụ `10.0.0.0/16 -> local`) là mặc định, **không thể xóa hoặc chỉnh sửa**.

---

### 1.4. Internet Gateway (IGW) vs Egress-Only Internet Gateway
* **Internet Gateway (IGW)**:
  * Là dịch vụ **Managed, Horizontally Scaled, Redundant & Highly Available** của AWS.
  * Không có nút thắt cổ chai về băng thông (no bandwidth bottleneck).
  * Mỗi VPC chỉ có thể gắn **tối đa 1 IGW**.
  * Cung cấp địa chỉ 1-1 NAT giữa Private IP của instance và Public IPv4 của nó.
* **Egress-Only Internet Gateway**:
  * Chỉ dùng cho **IPv6**.
  * Cho phép các instance IPv6 đi ra ngoài Internet, nhưng chặn hoàn toàn kết nối từ Internet khởi tạo vào trong instance (tương tự chức năng của NAT Gateway nhưng cho IPv6 và hoàn toàn miễn phí).

---

### 1.5. NAT Gateway vs NAT Instance (Cực kỳ hay thi)

| Đặc tính | NAT Gateway | NAT Instance |
| :--- | :--- | :--- |
| **Bản chất** | AWS Managed Service (Serverless, tự động scale) | 1 EC2 Instance chạy AMI Linux |
| **Vị trí đặt** | **Bắt buộc nằm trong Public Subnet** | Nằm trong Public Subnet |
| **Tính sẵn sàng (HA)** | HA trong **1 Availability Zone (AZ)** duy nhất. Muốn Multi-AZ HA phải tạo **1 NAT GW cho mỗi AZ**. | Tự quản lý (phải dùng ASG + script failover) |
| **Băng thông** | Tự động scale từ 5 Gbps đến **45 Gbps** | Phụ thuộc vào Instance Type (T-shirt size) |
| **Bảo mật** | Không dùng Security Group (chỉ quản lý bằng NACL & SG của instances phía sau) | Có gắn Security Group riêng |
| **Cấu hình mạng** | Không cần chỉnh sửa Source/Dest Check | **Bắt buộc tắt Source/Destination Check** trên EC2 |
| **Chi phí** | Phí cố định theo giờ (~$0.045/h) + Phí trên mỗi GB dữ liệu đi qua | Trả theo giờ chạy EC2 + Data Transfer |
| **Public IP** | Bắt buộc gán **Elastic IP (EIP)** khi tạo | Có thể dùng Public IP hoặc Elastic IP |

> 💡 **Quy tắc vàng thiết kế HA Egress**:
> - Nếu ứng dụng Multi-AZ yêu cầu khả năng phục hồi cao (Resilience): **Mỗi AZ có 1 NAT Gateway riêng đặt tại Public Subnet của AZ đó**.
> - Nếu AZ-a sập, AZ-b vẫn còn NAT Gateway của nó để ra internet. Ngoài ra còn tránh **chi phí Cross-AZ Data Transfer** không đáng có.

---

## 2. LAB KIẾN TRÚC: MULTI-TIER HIGH AVAILABILITY ARCHITECTURE

```
                                  INTERNET
                                     │
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
│   │            │ (forward traffic) │ (egress) │  │            │ (forward traffic) │ (egress) │  │
│   │            ▼                   │          │  │            ▼                   │          │  │
│   │  [Private App Subnet 1 - 10.0.10.0/24]   │  │  [Private App Subnet 2 - 10.0.20.0/24]   │  │
│   │  ┌─────────────────────────────────────┐  │  │  ┌─────────────────────────────────────┐  │  │
│   │  │ Auto Scaling Group - EC2 App Tier   │  │  │  │ Auto Scaling Group - EC2 App Tier   │  │  │
│   │  └──────────────────┬──────────────────┘  │  │  └──────────────────┬──────────────────┘  │  │
│   │                     │                     │  │                     │                     │  │
│   │                     ▼ (db connection)     │  │                     ▼ (db connection)     │  │
│   │  [Private DB Subnet 1 - 10.0.100.0/24]    │  │  [Private DB Subnet 2 - 10.0.200.0/24]   │  │
│   │  ┌─────────────────────────────────────┐  │  │  ┌─────────────────────────────────────┐  │  │
│   │  │ RDS Primary (Active Read/Write)     ├──┼──┼─►│ RDS Standby (Synchronous Replica)   │  │  │
│   │  └─────────────────────────────────────┘  │  │  └─────────────────────────────────────┘  │  │
│   │                                           │  │                                           │  │
│   └───────────────────────────────────────────┘  └───────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

### Câu hỏi phản xạ kiến trúc:
1. **Tại sao ALB cần đặt ở Public Subnets trên cả 2 AZ?**
   - ALB cần tối thiểu 2 Public Subnet thuộc 2 AZ khác nhau để phân giải IP công khai và routing traffic vào các target ở Private Subnets.
2. **Tại sao không nên để EC2 App Tier ở AZ-B dùng chung NAT Gateway ở AZ-A?**
   - Nếu AZ-A gặp sự cố hạ tầng, NAT Gateway A chết $\rightarrow$ App Tier ở AZ-B mất kết nối ra ngoài (patching, payment gateway, third-party API).
   - Traffic từ AZ-B sang AZ-A chịu thêm chi phí Data Transfer liên vùng (**Cross-AZ Data Transfer Cost**).
3. **Database Subnet Group yêu cầu gì?**
   - Phải chứa tối thiểu 2 subnet ở 2 AZ khác nhau trong cùng Region để hỗ trợ Multi-AZ failover.

---

## 3. TỪ KHÓA BẪY & CHIẾN LƯỢC CHỌN ĐÁP ÁN (EXAM PATTERNS)

| Cụm từ trong câu hỏi | Lời giải đáp án của AWS | Cạm bẫy cần tránh |
| :--- | :--- | :--- |
| **"Egress-only internet access for private IPv4 instances"** | **NAT Gateway** | Chọn nhầm IGW (vì IGW cho phép cả 2 chiều vào/ra). |
| **"Egress-only internet access for IPv6"** | **Egress-Only Internet Gateway** | NAT Gateway không hỗ trợ dịch IPv6 sang IPv6. |
| **"High availability for outbound internet"** | **1 NAT Gateway cho mỗi AZ** | Dùng 1 NAT Gateway duy nhất cho toàn VPC (Single Point of Failure). |
| **"Least operational effort"** | **NAT Gateway** | Chọn NAT Instance (phải tự maintain, patch, script HA). |
| **"Custom port forwarding or proxy script on NAT"** | **NAT Instance** | NAT Gateway không cho phép SSH vào tùy biến cấu hình hệ thống. |
