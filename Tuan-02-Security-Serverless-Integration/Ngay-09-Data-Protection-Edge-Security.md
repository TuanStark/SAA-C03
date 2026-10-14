# NGÀY 9: DATA PROTECTION & EDGE SECURITY (KMS, SECRETS MANAGER, PARAMETER STORE, ACM, WAF, SHIELD)
*(Tuần 2: Security, Serverless & Disaster Recovery - Trọng số Domain 1: Design Secure Architectures)*

---

## 1. AWS KMS (KEY MANAGEMENT SERVICE) CHUYÊN SÂU

AWS KMS là dịch vụ quản lý khóa mã hóa tập trung chuẩn FIPS 140-2 Level 2 (hoặc Level 3 đối với CloudHSM). 

```
                                  ┌───────────────────────────────────┐
                                  │           AWS KMS KEYS            │
                                  └─────────────────┬─────────────────┘
                                                    │
                 ┌──────────────────────────────────┼──────────────────────────────────┐
                 ▼                                  ▼                                  ▼
      [ AWS Owned Keys ]                   [ AWS Managed Keys ]               [ Customer Managed Keys (CMK) ]
      • Do AWS nội bộ sở hữu               • Mặc định tạo bởi service:        • Do khách hàng tự tạo & quản lý
      • Hoàn toàn miễn phí                   `aws/s3`, `aws/ebs`, `aws/rds`   • $1 / tháng / khóa + phí API
      • Không thấy trên Console            • Thấy được trên KMS Console       • Toàn quyền sửa Key Policy
      • Không xuất hiện trong CloudTrail   • Tự động xoay khóa mỗi 1 năm      • Bật/tắt xoay khóa tự động
                                           • KHÔNG THỂ sửa Key Policy         • Hỗ trợ Cross-Account Sharing
```

### 1.1. Khái niệm cốt lõi: Envelope Encryption (Mã hóa phong bì)
* **Vấn đề**: AWS KMS API chỉ có khả năng mã hóa trực tiếp dữ liệu có dung lượng tối đa **4 KB** (vì lý do hiệu năng và bảo mật của phần cứng HSM).
* **Giải pháp Envelope Encryption**:
  1. Ứng dụng gọi KMS API `GenerateDataKey`.
  2. KMS trả về 2 phiên bản của **Data Key**:
     - **Plaintext Data Key** (Khóa thô nằm trong RAM).
     - **Encrypted Data Key** (Khóa đã được mã hóa bởi KMS CMK).
  3. Ứng dụng dùng *Plaintext Data Key* để mã hóa file dữ liệu lớn (file hàng GB/TB).
  4. Sau khi mã hóa xong, ứng dụng **xóa ngay lập tức Plaintext Data Key khỏi RAM**.
  5. Đính kèm *Encrypted Data Key* ngay bên cạnh file mã hóa (Ciphertext) để lưu trữ.
* 👉 *Khi cần giải mã (Decrypt)*: Ứng dụng gửi *Encrypted Data Key* lên KMS $\rightarrow$ KMS trả về *Plaintext Data Key* $\rightarrow$ Ứng dụng giải mã file.

### 1.2. KMS Key Policies & Multi-Region Keys
* **Key Policy**: Là Resource-based Policy **BẮT BUỘC** trên mỗi CMK. Nếu một khóa CMK bị xóa mất quyền trong Key Policy thì khóa đó sẽ trở thành "mồ côi" và không ai có thể can thiệp được.
  - *Mặc định*: Key Policy cấp toàn quyền cho tài khoản Root của AWS Account sở hữu, cho phép quản trị viên dùng IAM Policy để phân quyền tiếp.
* **KMS Multi-Region Keys**:
  - Cho phép sao chép một khóa Primary sang các Region khác với **cùng Key ID và Key Material**.
  - Cho phép mã hóa ở Region A và giải mã trực tiếp ở Region B **mà không cần gọi API xuyên vùng (No Cross-Region API calls)**.

---

## 2. SECRETS MANAGER vs SSM PARAMETER STORE (SO SÁNH QUYẾT ĐỊNH)

Đây là câu hỏi phân loại cực kỳ phổ biến trong đề thi SAA-C03:

| Tiêu chí | AWS Secrets Manager | AWS Systems Manager Parameter Store |
| :--- | :--- | :--- |
| **Mục đích chính** | Lưu trữ **Bí mật nhạy cảm (Secrets)**: Mật khẩu Database, API Keys, OAuth tokens. | Lưu trữ **Cấu hình hệ thống (Configuration)** và bí mật dạng chuỗi key-value. |
| **Xoay vòng tự động (Automatic Rotation)** | **TỰ ĐỘNG 100% (Out-of-the-box)**: Tích hợp sẵn với RDS, Aurora, DocumentDB qua Lambda template. | **KHÔNG CÓ SẴN**: Phải tự viết EventBridge rule + Lambda tùy biến rất phức tạp. |
| **Sinh mật khẩu ngẫu nhiên** | Có sẵn tính năng tự sinh chuỗi password phức tạp khi xoay vòng. | Không hỗ trợ sinh mật khẩu. |
| **Chia sẻ Cross-Account** | **Hỗ trợ bản địa** thông qua Resource Policy trên Secret. | Khó khăn hơn (chỉ hỗ trợ chia sẻ qua RAM cho Advanced tier). |
| **Chi phí** | **Có phí**: $0.40 / secret / tháng + $0.05 cho mỗi 10,000 API calls. | **Standard Tier: HOÀN TOÀN MIỄN PHÍ** (tối đa 10,000 parameters, dung lượng 4KB/param). |
| **Mã hóa** | Bắt buộc mã hóa bằng KMS. | Hỗ trợ 3 kiểu: `String`, `StringList`, và `SecureString` (mã hóa qua KMS). |

> 💡 **QUY TẮC BẮT TỪ KHÓA ĐỀ THI**:
> - *"Cần lưu trữ mật khẩu cơ sở dữ liệu và **tự động xoay vòng mật khẩu (automatically rotate secrets)** định kỳ 30 ngày một lần mà không gây downtime"* $\rightarrow$ **AWS Secrets Manager**.
> - *"Cần lưu trữ các biến cấu hình ứng dụng, license keys với **chi phí thấp nhất (lowest cost / free)**"* $\rightarrow$ **SSM Parameter Store (Standard Tier)**.

---

## 3. AWS CERTIFICATE MANAGER (ACM)

* **Chức năng**: Cấp phát, quản lý và gia hạn tự động chứng chỉ SSL/TLS miễn phí cho các dịch vụ AWS.
* **Tích hợp**: Cung cấp SSL/TLS endpoints trực tiếp cho **ALB, CloudFront, API Gateway**.
* **Cơ chế xác thực (Validation)**:
  - **DNS Validation (Khuyên dùng)**: Tạo bản ghi CNAME trên Route 53. Tự động gia hạn chứng chỉ hàng năm mà không cần can thiệp.
  - **Email Validation**: Gửi email xác nhận tới chủ sở hữu tên miền.
* ⚠️ **HAI BẪY ĐỀ THI KINH ĐIỂN VỀ ACM**:
  1. **Chứng chỉ SSL cho CloudFront Distribution**:
     - Bắt buộc phải được tạo hoặc import tại Region **`us-east-1` (N. Virginia)**. Nếu tạo ở Region khác, CloudFront sẽ không nhìn thấy chứng chỉ này.
  2. **Cài chứng chỉ ACM lên máy chủ EC2**:
     - Bạn **KHÔNG THỂ tải private key** của chứng chỉ Public ACM về để cài vào máy chủ web Nginx/Apache trên EC2.
     - **Giải pháp**: Đặt một **Application Load Balancer (ALB)** phía trước EC2 và cài chứng chỉ ACM lên ALB (TLS Termination).

---

## 4. EDGE SECURITY: AWS WAF vs AWS SHIELD vs COMPLIANCE TOOLS

```
                        ┌─────────────────────────────────────────┐
                        │          AN NINH BIÊN (EDGE SECURITY)   │
                        └────────────────────┬────────────────────┘
                                             │
             ┌───────────────────────────────┴───────────────────────────────┐
             ▼                                                               ▼
     [ AWS WAF (Layer 7) ]                                          [ AWS Shield (Layer 3/4) ]
   • Bảo vệ: ALB, CloudFront, API Gateway, AppSync                 • Chống tấn công từ chối dịch vụ (DDoS)
   • Chặn: SQL Injection, XSS, HTTP flood                          • **Shield Standard**: Miễn phí, tự động
   • Lọc: IP Blacklist/Whitelist, Geo-blocking                       chặn SYN/UDP floods, reflection.
   • **Rate-based rules**: Chặn IP gửi > N request/5 phút         • **Shield Advanced**: $3,000/tháng, có đội
     (chống quét web, brute-force, web scraping)                     DDoS Response Team (SRT), bồi hoàn phí.
```

### 4.1. Phân biệt nhanh các công cụ An ninh & Tuân thủ:
* **AWS WAF**: Tường lửa tầng ứng dụng (Layer 7), kiểm tra HTTP Body, Headers, Cookies, lọc mã độc SQLi/XSS, giới hạn tần suất request (Rate Limiting).
* **AWS Shield Standard vs Advanced**:
  - *Standard*: Bật tự động 100% cho mọi khách hàng AWS, bảo vệ chống DDoS tầng mạng/vận chuyển (SYN Flood, UDP Reflection).
  - *Advanced*: Dịch vụ bảo vệ chuyên biệt chống DDoS quy mô lớn, có quyền truy cập 24/7 tới nhóm phản ứng nhanh **AWS SRT (Shield Response Team)** và hỗ trợ tài chính bảo vệ chi phí co giãn đột biến do tấn công gây ra.
* **Amazon GuardDuty**: Dịch vụ phát hiện mối đe dọa thông minh (Intelligent Threat Detection) sử dụng Machine Learning phân tích CloudTrail Events, VPC Flow Logs, DNS Logs, EKS Audit Logs (ví dụ: phát hiện EC2 đào coin, tài khoản bị xâm nhập).
* **Amazon Macie**: Dịch vụ bảo vệ dữ liệu sử dụng Machine Learning để **quét và nhận diện dữ liệu nhạy cảm (PII - Personally Identifiable Information, số thẻ tín dụng)** lưu trữ trong **Amazon S3**.

---

## 5. LAB KIẾN TRÚC: HỆ THỐNG AN NINH ĐA LỚP (DEFENSE IN DEPTH)

```
                            [ NGƯỜI DÙNG INTERNET ]
                                       │
                                       ▼ (HTTPS)
    ┌────────────────────────── TẦNG AN NINH BIÊN (EDGE) ──────────────────────────┐
    │                                                                              │
    │   [ Amazon CloudFront ] + [ AWS Shield Standard (Anti-DDoS Layer 3/4) ]      │
    │             │                                                                │
    │             ▼                                                                │
    │   [ AWS WAF (Web Application Firewall) ]                                     │
    │   • Rule 1: Chặn SQL Injection & Cross-Site Scripting (XSS)                  │
    │   • Rule 2: Rate-based rule (Chặn IP gửi > 2000 request/5 phút)              │
    │   • Rule 3: Geo-match (Chặn traffic từ các quốc gia trong danh sách đen)     │
    │                                                                              │
    └──────────────────────────────────┬───────────────────────────────────────────┘
                                       │
                                       ▼ (Traffic sạch)
    ┌─────────────────────────── PUBLIC SUBNET ────────────────────────────────────┐
    │                                                                              │
    │   [ Application Load Balancer (ALB) ]                                        │
    │   • Chứng chỉ SSL/TLS quản lý tự động bởi **AWS Certificate Manager (ACM)**  │
    │                                                                              │
    └──────────────────────────────────┬───────────────────────────────────────────┘
                                       │
                                       ▼
    ┌─────────────────────────── PRIVATE SUBNET ───────────────────────────────────┐
    │                                                                              │
    │   [ EC2 Web/API App Tier ]                                                   │
    │   • Gắn IAM Role (Instance Profile)                                          │
    │   • Đọc DB password tự động từ **AWS Secrets Manager**                       │
    │                                                                              │
    │                                  │ (Truy vấn an toàn)                        │
    │                                  ▼                                           │
    │   [ Amazon Aurora DB (Mã hóa tĩnh bằng KMS CMK) ]                            │
    │   • **Secrets Manager Lambda**: Tự động xoay password DB mỗi 30 ngày        │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘
```
