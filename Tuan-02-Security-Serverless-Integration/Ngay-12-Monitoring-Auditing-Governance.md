# NGÀY 12: MONITORING, AUDITING & GOVERNANCE (CLOUDWATCH, CLOUDTRAIL, AWS CONFIG, SYSTEMS MANAGER)
*(Tuần 2: Security, Serverless & Governance - Trọng số Observability & Operational Excellence)*

---

## 1. BỘ BA QUẢN TRỊ & GIÁM SÁT TOÀN DIỆN (OBSERVABILITY TRIO)

Trong đề thi SAA-C03, rất nhiều thí sinh nhầm lẫn giữa **CloudWatch**, **CloudTrail** và **AWS Config**. Hãy ghi nhớ câu "thần chú" sau:
* **CloudWatch**: *"Hệ thống đang chạy như thế nào?"* $\rightarrow$ **Hiệu năng & Vận hành (Metrics, Logs, Alarms)**.
* **CloudTrail**: *"Ai đã làm gì, vào lúc nào, từ IP nào?"* $\rightarrow$ **Kiểm toán hành vi gọi API (Auditing, Who did What & When)**.
* **AWS Config**: *"Tài nguyên đã thay đổi cấu hình ra sao theo thời gian và có tuân thủ quy định không?"* $\rightarrow$ **Lịch sử cấu hình & Tuân thủ (Configuration History & Compliance)**.

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                 MA TRẬN PHÂN BIỆT BỘ BA QUẢN TRỊ                                │
├──────────────────────────────┬──────────────────────────────┬────────────────────────────────────┤
│      Amazon CloudWatch       │        AWS CloudTrail        │             AWS Config             │
├──────────────────────────────┼──────────────────────────────┼────────────────────────────────────┤
│ • CPU, RAM, Disk, Network    │ • Nhật ký gọi API (API calls)│ • Bản ghi thay đổi cấu hình        │
│ • Log files của ứng dụng     │ • User identity, Source IP   │ • Kiểm tra tuân thủ (Rules)        │
│ • Cảnh báo (Alarms) co giãn  │ • Ghi nhận thao tác Console, │ • Lịch sử cấu hình theo thời gian  │
│   Auto Scaling, gửi cảnh báo │   CLI, SDK, AWS Services     │ • Tự động sửa lỗi (Remediation)    │
│ • Trọng tâm: Performance     │ • Trọng tâm: Security Audit  │ • Trọng tâm: Compliance & Timeline │
└──────────────────────────────┴──────────────────────────────┴────────────────────────────────────┘
```

---

## 2. AMAZON CLOUDWATCH CHUYÊN SÂU

### 2.1. CloudWatch Metrics & Unified CloudWatch Agent
* **Default Monitoring**: Thu thập metrics hạ tầng mỗi **5 phút** một lần (miễn phí).
* **Detailed Monitoring**: Thu thập metrics mỗi **1 phút** một lần (có tính phí, dùng khi cần phản ứng nhanh với biến động tải).
* ⚠️ **BẪY ĐỀ THI VỀ CHỈ SỐ BỘ NHỚ RAM (MEMORY METRIC)**:
  - Mặc định, AWS **KHÔNG THỂ thu thập chỉ số % sử dụng RAM (Memory Utilization)** hay dung lượng đĩa trống của EC2, vì RAM là tài nguyên cấp hệ điều hành bên trong máy (Guest OS level), AWS không can thiệp vào trong máy ảo của khách hàng.
  - **Giải pháp**: Cài đặt **Unified CloudWatch Agent** bên trong EC2 instance để thu thập RAM, Swap, Disk space và đẩy log tệp tin về CloudWatch.

### 2.2. CloudWatch Logs & Metric Filters
* **Metric Filters**: Quét qua các dòng log của ứng dụng theo biểu thức (ví dụ: tìm từ khóa `[ERROR]`, `403`, hoặc `HTTP 500`) và chuyển đổi tần suất xuất hiện của từ khóa đó thành một **CloudWatch Custom Metric**.
* Dựa trên Custom Metric đó, bạn có thể tạo một **CloudWatch Alarm** để gửi cảnh báo SNS cho đội vận hành ngay khi có lỗi xuất hiện.

### 2.3. CloudWatch Alarms: 3 Trạng thái & Hành động (Actions)
* **3 trạng thái**: `OK` (Bình thường), `ALARM` (Vượt ngưỡng vi phạm), `INSUFFICIENT_DATA` (Thiếu dữ liệu).
* **Hành động có thể kích hoạt**:
  1. Gửi thông báo tới **Amazon SNS Topic**.
  2. Kích hoạt chính sách co giãn của **Auto Scaling Group**.
  3. Thực hiện hành động quản trị trên EC2: **Stop, Terminate, Reboot, hoặc Recover** (Khôi phục instance sang phần cứng máy chủ vật lý khác khi host cũ bị lỗi mà giữ nguyên IP và EBS volume!).

---

## 3. AWS CLOUDTRAIL: NHẬT KÝ KIỂM TOÁN HÀNH ĐỘNG API

CloudTrail ghi lại toàn bộ dấu vết các thao tác gọi API trên toàn bộ tài khoản AWS (thao tác từ Console, CLI, SDK hay giữa các service với nhau).

### 3.1. Các loại sự kiện trong CloudTrail (Event Types):
1. **Management Events (Mặc định được bật)**:
   - Các thao tác quản lý tài nguyên: Tạo S3 Bucket, Terminate EC2, cấu hình Route Table, gắn IAM Policy.
   - CloudTrail tự động lưu lịch sử 90 ngày miễn phí trong mục **Event History**.
2. **Data Events (Mặc định BỊ TẮT do lưu lượng cực lớn)**:
   - Các thao tác ở mức dữ liệu bên trong tài nguyên: Gọi `s3:GetObject`/`s3:PutObject` trên S3, gọi `Invoke` trên AWS Lambda.
   - Phải bật thủ công và tính phí theo số lượng event.
3. **Insights Events**:
   - Dùng Machine Learning phát hiện các hành vi gọi API bất thường (ví dụ: đột ngột gọi hàng ngàn API IAM hoặc xóa hàng loạt tài nguyên).

### 3.2. Thực hành tốt nhất cho Doanh nghiệp (Enterprise Best Practices):
* **Organization Trail**: Tạo một Trail từ tài khoản Management Account và kích hoạt cho toàn bộ Member Accounts trong AWS Organizations. Toàn bộ log tự động gom về một **S3 Bucket trung tâm nằm tại Log Archive Account**.
* **Log File Integrity Validation (Kiểm tra tính toàn vẹn của Log)**: Bật tính năng này để AWS tự động ký số bằng mã băm SHA-256 và RSA. Giúp chứng minh trước tòa án và cơ quan kiểm toán rằng các file log chưa từng bị hacker sửa đổi hoặc xóa bớt.

---

## 4. AWS CONFIG: QUẢN TRỊ TUÂN THỦ & THEO DÕI THAY ĐỔI CẤU HÌNH

AWS Config hoạt động như một "cuốn sổ ghi chép dòng thời gian" ghi lại trạng thái cấu hình của mọi tài nguyên AWS.

```
        [ TÀI NGUYÊN AWS ] ──► (Thay đổi cấu hình) ──► [ AWS CONFIG RECORDER ]
                                                                 │
                                                                 ▼
                                                       [ ĐÁNH GIÁ CONFIG RULES ]
                                                       (Non-Compliant / Vi phạm)
                                                                 │
                                                                 ▼
                                                  [ AUTOMATIC REMEDIATION ]
                                                  (AWS Systems Manager Automation)
```

### 4.1. Cơ chế hoạt động:
* **Timeline Cấu hình**: Cho phép xem lại lịch sử chi tiết: *"Vào lúc 14:00 ngày hôm qua, Security Group này đã bị ai mở thêm cổng 22 ra Internet?"*.
* **Config Rules (Quy tắc tuân thủ)**:
  - *Managed Rules có sẵn*: `s3-bucket-public-read-prohibited` (phát hiện bucket bị mở public), `encrypted-volumes` (kiểm tra EBS chưa mã hóa), `restricted-ssh` (phát hiện mở port 22 cho 0.0.0.0/0).
  - *Custom Rules*: Viết bằng hàm AWS Lambda để kiểm tra các chính sách nội bộ tùy biến.
* **Tự động khắc phục (Remediation Actions)**:
  - Khi phát hiện một tài nguyên vi phạm quy tắc (Non-Compliant), AWS Config có thể tự động kích hoạt một **AWS Systems Manager Automation Document** để tự sửa lỗi (ví dụ: tự động thu hồi quyền Public của S3 bucket ngay lập tức).

---

## 5. AWS SYSTEMS MANAGER (SSM) CHUYÊN SÂU

AWS Systems Manager là trung tâm quản trị và vận hành tập trung cho các cụm máy chủ EC2 và máy chủ On-Premises (thông qua **SSM Agent** và IAM Role `AmazonSSMManagedInstanceCore`).

### 4 tính năng "vàng" xuất hiện liên tục trong đề thi:
1. **SSM Session Manager (Thay thế hoàn toàn SSH / Bastion Host)**:
   - Cho phép mở phiên làm việc dòng lệnh (Shell/PowerShell) an toàn vào EC2 trực tiếp từ trình duyệt web hoặc AWS CLI **MÀ KHÔNG CẦN**:
     - Mở cổng 22 (SSH) hay cổng 3389 (RDP) trên Security Group.
     - Dựng máy chủ nhảy trung gian (Bastion Hosts).
     - Gán Public IP cho instance (máy nằm hoàn toàn trong Private Subnet vẫn kết nối được qua VPC Endpoints).
   - Toàn bộ câu lệnh gõ trong phiên đều được ghi log vào **CloudWatch Logs** và **S3**.
2. **SSM Run Command**:
   - Thực thi các lệnh hoặc script đồng loạt trên hàng trăm máy chủ EC2 cùng lúc mà không cần đăng nhập vào từng máy.
3. **SSM Patch Manager**:
   - Tự động hóa quét và cài đặt các bản vá lỗi hệ điều hành và bảo mật (OS & Security Patches) định kỳ theo các bộ tiêu chuẩn **Patch Baselines**.
4. **SSM Parameter Store**:
   - Lưu trữ biến môi trường và mật khẩu có cấu trúc dạng cây phân tầng (đã học ở Ngày 9).

---

## 6. LAB KIẾN TRÚC: HỆ THỐNG GIÁM SÁT & TỰ ĐỘNG KHẮC PHỤC AN NINH (EVENT-DRIVEN REMEDIATION)

```
                            [ KỸ SƯ / HACKER MỞ PUBLIC S3 BUCKET ]
                                             │
                                             ▼
                             ┌───────────────────────────────┐
                             │       AWS CloudTrail          │ (Ghi lại: Ai gọi API `PutBucketAcl`)
                             └───────────────┬───────────────┘
                                             │
                                             ▼
                             ┌───────────────────────────────┐
                             │          AWS Config           │
                             │ (Rule: `s3-bucket-public-`    │
                             │        `read-prohibited`)     │
                             └───────────────┬───────────────┘
                                             │ (Phát hiện: NON-COMPLIANT)
                                             ▼
                             ┌───────────────────────────────┐
                             │    Amazon EventBridge Rule    │
                             └───────┬───────────────┬───────┘
                                     │               │
             ┌───────────────────────┘               └───────────────────────┐
             ▼ (Tự động khắc phục)                                           ▼ (Gửi cảnh báo)
    ┌─────────────────────────────────┐                             ┌─────────────────────────────────┐
    │ AWS Systems Manager Automation  │                             │ Amazon SNS Topic                │
    │ (Runbook: Đóng quyền Public S3) │                             │ (Gửi SMS / Email cho Security)  │
    └─────────────────────────────────┘                             └─────────────────────────────────┘
```
