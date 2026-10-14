# NGÀY 8: IAM, ORGANIZATIONS & MULTI-ACCOUNT (SCP, CROSS-ACCOUNT ROLES, STS, PERMISSIONS BOUNDARIES)
*(Tuần 2: Security, Serverless & Disaster Recovery - Trọng số Domain 1: Design Secure Architectures 30%)*

---

## 1. QUY TẮC ĐÁNH GIÁ POLICY TRONG AWS (POLICY EVALUATION LOGIC)

Domain 1 (Bảo mật) chiếm tới **30% tổng số điểm** của bài thi SAA-C03. Nền tảng cốt lõi của bảo mật AWS nằm ở thứ tự ưu tiên khi đánh giá các IAM Policy:

```
                                  [ REQUEST GỬI ĐẾN ]
                                           │
                                           ▼
                     Có bất kỳ EXPLICIT DENY nào không?
                     (Từ SCP, Permissions Boundary, IAM Policy, Resource Policy)
                                     /           \
                               (Có) /             \ (Không)
                                   ▼               ▼
                           [ BỊ CHẶN DENY ]    Có EXPLICIT ALLOW nào không?
                                                /                \
                                          (Có) /                  \ (Không)
                                              ▼                    ▼
                                     [ ĐƯỢC PHÉP ALLOW ]   [ DEFAULT DENY ]
```

### Nguyên tắc vàng:
1. **Mặc định là từ chối (Default Deny)**: Bất kỳ thao tác nào chưa được cấp quyền tường minh đều bị từ chối.
2. **Từ chối tuyệt đối luôn thắng (Explicit Deny trumps Allow)**: Nếu có một dòng `"Effect": "Deny"`, nó sẽ ghi đè và vô hiệu hóa tất cả các dòng `"Effect": "Allow"`, bất kể policy đó được gán ở đâu.
3. **Giao thoa quyền (Intersection)**: Nếu một User bị ràng buộc bởi cả **Identity Policy**, **Permissions Boundary**, và **SCP**, quyền thực tế User đó có được là **phần giao (Intersection)** của cả 3 loại policy này.

---

## 2. CÁC LOẠI POLICIES & PERMISSIONS BOUNDARIES

### 2.1. Phân biệt Identity-based Policy vs Resource-based Policy

| Tiêu chí | Identity-based Policy | Resource-based Policy |
| :--- | :--- | :--- |
| **Đối tượng gắn vào** | Gắn vào IAM User, Group, hoặc IAM Role | Gắn trực tiếp vào **Tài nguyên (Resource)**: S3 Bucket, KMS Key, SQS Queue, Lambda, Secrets Manager |
| **Phần tử `Principal`** | **KHÔNG CÓ** (vì bản thân policy đã gắn vào danh tính) | **BẮT BUỘC PHẢI CÓ** (để chỉ định ai/tài khoản nào được phép truy cập) |
| **Truy cập Cross-Account** | Cần cả 2 bên: Account A cho phép User AssumeRole, Account B cấp quyền cho Role. | **Chỉ cần Resource Policy ở Account đích cho phép trực tiếp Account nguồn** (ngoại lệ: KMS và IAM Role vẫn cần cả 2). |

### 2.2. Permissions Boundaries (Ranh giới phân quyền)
* **Vấn đề thực tế**: Doanh nghiệp muốn cho phép các kỹ sư DevOps tự tạo IAM Role/User cho ứng dụng của họ, nhưng sợ họ tự cấp quyền `AdministratorAccess` cho chính mình (lỗ hổng **Privilege Escalation - Leo thang đặc quyền**).
* **Giải pháp của AWS**: Gán một **Permissions Boundary** lên IAM User/Role đó.
* **Bản chất**: Permissions Boundary **KHÔNG CẤP QUYỀN**, nó chỉ đóng vai trò như một "chiếc lồng bảo vệ" đặt ra **mức trần quyền hạn tối đa (Maximum allowable permissions)** mà identity đó có thể có được.
* 👉 *Từ khóa đề thi*: *"Allow developers to create IAM roles for Lambda, but prevent privilege escalation to Administrator"* $\rightarrow$ **Dùng Permissions Boundaries**.

---

## 3. IAM ROLES & AWS STS (SECURITY TOKEN SERVICE)

Tuyệt đối không lưu Access Key / Secret Key dài hạn trong mã nguồn hoặc máy chủ EC2. Thay vào đó, toàn bộ workload phải sử dụng **IAM Roles** thông qua **AWS STS**.

```
                        ┌─────────────────────────────────────────┐
                        │          AWS STS (TOKEN SERVICE)        │
                        └────────────────────┬────────────────────┘
                                             │
             ┌───────────────────────────────┼───────────────────────────────┐
             ▼                               ▼                               ▼
     [ AssumeRole ]             [ AssumeRoleWithWebIdentity ]     [ AssumeRoleWithSAML ]
   • Dùng cho: Cross-Account,   • Dùng cho: Ứng dụng di động,    • Dùng cho: Đăng nhập tập trung
     EC2 Instance Profile,        Web app đăng nhập qua Google,     Doanh nghiệp (Enterprise SSO)
     Lambda Execution Role.       Facebook, Amazon, OIDC/Cognito.   kết nối với Active Directory / Okta.
   • Cấp: Temporary Credentials • Không cần quản lý user trong    • Chuẩn SAML 2.0.
     (Access Key, Secret Key,     IAM của AWS.
      Session Token 15m - 12h).
```

### Bẫy bảo mật: The "Confused Deputy" Problem & External ID
* **Kịch bản**: Công ty bạn thuê một bên thứ ba (SaaS Vendor) để giám sát tài nguyên AWS. Bạn tạo một IAM Role trong tài khoản của bạn để bên thứ 3 AssumeRole vào.
* **Lỗ hổng (Confused Deputy)**: Một kẻ xấu (khách hàng khác của bên thứ 3) biết được ARN Role của công ty bạn, lừa bên thứ 3 gửi request AssumeRole vào tài khoản của bạn để đánh cắp dữ liệu.
* **Giải pháp bắt buộc**: Bên thứ 3 phải cung cấp một chuỗi định danh duy nhất gọi là **External ID**. Khi bạn cấu hình Trust Policy của Role, bạn thêm điều kiện:
```json
"Condition": {
  "StringEquals": {
    "sts:ExternalId": "unique-client-id-12345"
  }
}
```
* 👉 *Từ khóa đề thi*: *"Third-party SaaS vendor accessing AWS account"*, *"Prevent confused deputy problem"* $\rightarrow$ **Sử dụng External ID trong `sts:AssumeRole`**.

---

## 4. AWS ORGANIZATIONS & SERVICE CONTROL POLICIES (SCP)

AWS Organizations cho phép quản lý tập trung hàng trăm tài khoản AWS theo cấu trúc hình cây gồm **Root**, các **Organizational Units (OUs)**, và các **Member Accounts**.

```
                           [ MANAGEMENT ACCOUNT (MASTER) ]
                                          │
                                       [ ROOT ] ──► (SCP Level 1)
                                          │
                      ┌───────────────────┴───────────────────┐
                      ▼                                       ▼
             [ OU: Production ] ──► (SCP Level 2)       [ OU: Non-Production ]
                      │                                       │
             ┌────────┴────────┐                             [ Account: Dev ]
             ▼                 ▼
      [ Account: Prod-1 ] [ Account: Prod-2 ]
```

### 4.1. Đặc tính sống còn của Service Control Policies (SCP):
1. **SCP KHÔNG CẤP QUYỀN (DO NOT GRANT PERMISSIONS)**:
   - SCP chỉ hoạt động như một bộ lọc biên (Guardrails). Nếu SCP không cho phép một action, thì dù User bên trong Account có quyền Administrator cũng **KHÔNG THỂ thực hiện action đó**.
2. **Áp dụng lên cả ROOT USER của Member Account**:
   - Ngay cả tài khoản Root của một Member Account cũng bị khống chế hoàn toàn bởi SCP của Organization.
3. **NGOẠI LỆ DUY NHẤT**:
   - **SCP KHÔNG BAO GIỜ áp dụng lên Management Account (Master Account)**. Management Account luôn miễn nhiễm với SCP.
4. **Quy tắc kế thừa (Inheritance)**:
   - Mặc định khi tạo AWS Organizations, policy `FullAWSAccess` được gắn vào Root và tất cả các OUs.
   - Để chặn một dịch vụ (ví dụ cấm dùng dịch vụ ngoài us-east-1): Tạo một SCP có `Effect: Deny` và gắn vào Root hoặc OU mong muốn.

### 4.2. Các kịch bản SCP kinh điển trong bài thi SAA-C03:

#### Kịch bản 1: Giới hạn chỉ cho phép tạo tài nguyên ở các Region chỉ định (Data Residency)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllOutsideApprovedRegions",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "organizations:*",
        "route53:*",
        "cloudfront:*",
        "waf:*",
        "sts:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "ap-southeast-1",
            "us-east-1"
          ]
        }
      }
    }
  ]
}
```
*(Lưu ý: Phải loại trừ các dịch vụ Global như IAM, Route 53, CloudFront bằng `NotAction` nếu không hệ thống sẽ bị lỗi).*

#### Kịch bản 2: Ngăn chặn bất kỳ ai xóa CloudTrail hoặc tắt bảo mật
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PreventCloudTrailDisabling",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 5. IAM IDENTITY CENTER (AWS SSO) & AWS CONTROL TOWER

* **AWS IAM Identity Center (tiền thân là AWS SSO)**:
  - Cổng đăng nhập tập trung (Single Sign-On) cho người dùng truy cập vào nhiều tài khoản AWS và các ứng dụng doanh nghiệp (Salesforce, Office 365).
  - Tích hợp trực tiếp với Microsoft Active Directory, Okta, hoặc cơ sở dữ liệu người dùng nội bộ.
  - Phân quyền thông qua các **Permission Sets** (tương đương IAM Roles được gán tự động cho các tài khoản).
* **AWS Control Tower**:
  - Dịch vụ thiết lập môi trường Multi-Account chuẩn tự động theo AWS Best Practices (Landing Zone).
  - Cung cấp sẵn các **Guardrails**:
    - **Preventive Guardrails**: Thực thi bằng **SCPs** (chặn ngay lập tức các hành vi vi phạm).
    - **Detective Guardrails**: Thực thi bằng **AWS Config** và **AWS CloudTrail** (phát hiện và cảnh báo vi phạm).

---

## 6. LAB KIẾN TRÚC: CROSS-ACCOUNT WORKLOAD ACCESS S3 AN TOÀN TUYỆT ĐỐI

```
    ┌──────────── ACCOUNT A (APP / WORKLOAD: 111111111111) ────────────┐
    │                                                                  │
    │   [ EC2 Instance ]                                               │
    │   • Gắn Instance Profile: Role-App                               │
    │   • IAM Policy: Cho phép `sts:AssumeRole` tới Role-S3-Reader     │
    │                                                                  │
    └──────────────────────────────────┬───────────────────────────────┘
                                       │
                                       │ (1) sts:AssumeRole
                                       │     (Nhận Temporary Credentials)
                                       ▼
    ┌──────────── ACCOUNT B (DATA LAKE / S3: 222222222222) ────────────┐
    │                                                                  │
    │   [ IAM Role: Role-S3-Reader ]                                   │
    │   • Trust Policy (Ai được assume):                               │
    │     Principal: `arn:aws:iam::111111111111:role/Role-App`         │
    │   • Permissions Policy:                                          │
    │     `s3:GetObject` trên `arn:aws:s3:::corporate-data-lake/*`     │
    │                                                                  │
    │                                  │ (2) Đọc dữ liệu an toàn       │
    │                                  ▼                               │
    │   [ Amazon S3 Bucket: "corporate-data-lake" ]                    │
    │                                                                  │
    └──────────────────────────────────────────────────────────────────┘
```

### Ưu điểm vượt trội:
1. **Không có bất kỳ Hardcoded Access Keys nào** trên EC2 hay trong mã nguồn.
2. Dữ liệu xác thực có thời hạn ngắn (Temporary credentials) do AWS STS tự động cấp phát và xoay vòng.
3. Toàn bộ hoạt động AssumeRole và đọc S3 đều được ghi vết đầy đủ trên **AWS CloudTrail** của cả 2 tài khoản.
