# NGÀY 5: S3 STORAGE & DATA PROTECTION (STORAGE CLASSES, LIFECYCLE, REPLICATION, ENCRYPTION, BUCKET POLICIES)
*(Tuần 1: Core Infrastructure - Trọng số Storage, Security & Cost Optimization)*

---

## 1. MA TRẬN S3 STORAGE CLASSES: CHI PHÍ & THỜI GIAN TRUY XUẤT

Amazon S3 có độ bền cam kết lên tới **99.999999999% (11 con số 9)** trên tất cả các storage classes (trừ One Zone-IA bị mất nếu AZ chứa nó bị phá hủy). Hiểu rõ các thông số sau là chìa khóa xử lý các câu hỏi tối ưu chi phí:

| Storage Class | Availability | AZ Phân bổ | Thời gian truy xuất | Thời gian lưu tối thiểu | Phí truy xuất (Retrieval Fee) | Trường hợp sử dụng tối ưu trong đề thi |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **S3 Standard** | 99.99% | $\ge$ 3 AZs | Milliseconds | Không có | **Không** | Dữ liệu truy cập thường xuyên, website static assets, big data analytics. |
| **S3 Intelligent-Tiering** | 99.9% | $\ge$ 3 AZs | Milliseconds | Không có | **Không** | **Mẫu truy cập không xác định hoặc thay đổi liên tục**. Tự động chuyển tier, không tốn operational overhead. |
| **S3 Standard-IA** | 99.9% | $\ge$ 3 AZs | Milliseconds | **30 ngày** | **Có ($/GB)** | Dữ liệu ít truy cập nhưng **khi cần phải có ngay lập tức** (Disaster Recovery backup, báo cáo tài chính). |
| **S3 One Zone-IA** | 99.5% | **1 AZ duy nhất** | Milliseconds | **30 ngày** | **Có ($/GB)** | Dữ liệu backup phụ **có thể tạo lại được** nếu mất, bản sao thứ cấp, thumbnails ảnh. Tiết kiệm 20% so với Standard-IA. |
| **S3 Glacier Instant Retrieval** | 99.9% | $\ge$ 3 AZs | **Milliseconds** | **90 ngày** | **Có ($/GB)** | Lưu trữ lưu trữ y tế, hồ sơ bảo hiểm truy cập 1 lần mỗi quý nhưng cần xem ngay lập tức. |
| **S3 Glacier Flexible Retrieval** *(trước đây là Glacier)* | 99.99% | $\ge$ 3 AZs | • Expedited: 1-5 phút<br>• Standard: 3-5 giờ<br>• Bulk: 5-12 giờ | **90 ngày** | **Có ($/GB)** | Dữ liệu lưu trữ dài hạn (Archive), sao lưu định kỳ hàng tuần/tháng không cần truy xuất tức thì. |
| **S3 Glacier Deep Archive** | 99.99% | $\ge$ 3 AZs | • Standard: 12 giờ<br>• Bulk: 48 giờ | **180 ngày** | **Có ($/GB)** | **Chi phí rẻ nhất toàn bộ AWS**. Lưu trữ hồ sơ pháp lý, ngân hàng, y tế 7 - 10 năm theo luật định. |

```
                       ┌────────────────────────────────────────────────────────┐
                       │          TỐI ƯU CHI PHÍ LƯU TRỮ VỚI AMAZON S3          │
                       └───────────────────────────┬────────────────────────────┘
                                                   │
             ┌─────────────────────────────────────┼─────────────────────────────────────┐
             ▼                                     ▼                                     ▼
     [ TẦNG TRUY XUẤT NHANH ]              [ TẦNG LƯU TRỮ ARCHIVE ]              [ TRUY CẬP KHÔNG ĐOÁN ĐƯỢC ]
     • S3 Standard (thường xuyên)          • S3 Glacier Flexible (3-5h)          • **S3 Intelligent-Tiering**
     • S3 Standard-IA (ít dùng)            • S3 Glacier Deep Archive (12-48h)      Tự động di chuyển dữ liệu
     • S3 Glacier Instant (vài ms)           Chi phí rẻ nhất cho compliance        giữa các tầng, không tính
       Dành cho dữ liệu cần đọc ngay!        lưu trữ từ 7 đến 10 năm.              phí retrieval.
```

---

## 2. S3 LIFECYCLE MANAGEMENT (QUẢN LÝ VÒNG ĐỜI DỮ LIỆU)

S3 Lifecycle Rules cho phép tự động hóa quá trình tối ưu chi phí và dọn dẹp dữ liệu:

### 2.1. Hai loại hành động chính (Actions):
1. **Transition Actions**: Định nghĩa thời điểm chuyển đổi dữ liệu từ storage class đắt sang storage class rẻ hơn:
   - *Ví dụ*: Đẩy từ `Standard` $\rightarrow$ `Standard-IA` (sau 30 ngày) $\rightarrow$ `Glacier Flexible Retrieval` (sau 90 ngày) $\rightarrow$ `Glacier Deep Archive` (sau 180 ngày).
2. **Expiration Actions**: Định nghĩa thời điểm xóa vĩnh viễn dữ liệu:
   - Xóa các đối tượng cũ sau 365 ngày.
   - Xóa các phiên bản cũ (**Noncurrent Versions**) sau 60 ngày.
   - **Xóa các Delete Markers hết hạn** (Expired Object Delete Markers).
   - **Hủy các Multipart Upload chưa hoàn thành** (*Abort incomplete multipart uploads after 7 days*) $\rightarrow$ **Mẹo tiết kiệm chi phí cực lớn trong bài thi**.

> ⚠️ **QUY TẮC RÀNG BUỘC VÒNG ĐỜI BẮT BUỘC TRONG ĐỀ THI**:
> - Đối tượng phải nằm ở **S3 Standard tối thiểu 30 ngày** trước khi có thể chuyển sang **S3 Standard-IA** hoặc **S3 One Zone-IA**.
> - Không thể chuyển trực tiếp từ Standard-IA sang One Zone-IA.

---

## 3. BẢO MẬT & BẢO VỆ DỮ LIỆU: VERSIONING, OBJECT LOCK, ENCRYPTION

### 3.1. S3 Versioning & MFA Delete
* **Versioning**: Bảo vệ chống lại việc vô tình xóa hoặc ghi đè file:
  - Khi xóa file bằng lệnh `DeleteObject`, AWS chỉ tạo một **Delete Marker** che file đó đi. Người dùng có thể xóa Delete Marker để phục hồi file nguyên vẹn.
  - Chỉ có thể **Suspend** (tạm dừng) Versioning, **KHÔNG THỂ TẮT (Disable)** một khi đã bật trên bucket.
* **MFA Delete**:
  - Bắt buộc phải có mã xác thực 2 bước (MFA) từ tài khoản **Root** khi muốn: (1) Xóa vĩnh viễn một Version ID cụ thể, hoặc (2) Tắt tính năng Versioning của bucket.
  - Chỉ cấu hình được qua **AWS CLI** hoặc API, không hỗ trợ trên giao diện web Console.

### 3.2. S3 Object Lock (Chuẩn WORM - Write Once, Read Many)
Bắt buộc bật Versioning để dùng Object Lock. Chống ghi đè hoặc xóa dữ liệu trong một khoảng thời gian xác định.

* **Retention Modes**:
  1. **Governance Mode**: Bảo vệ file khỏi hầu hết user, nhưng những user có IAM Permission đặc biệt (`s3:BypassGovernanceRetention`) vẫn có thể xóa hoặc sửa file.
  2. **Compliance Mode**: **TUYỆT ĐỐI KHÔNG AI CÓ THỂ XÓA HOẶC THAY ĐỔI FILE**, kể cả tài khoản **AWS Root Account** trong suốt thời gian retention. Thường dùng để tuân thủ các quy định tài chính khắt khe (SEC Rule 17a-4).
* **Legal Hold**: Khóa đối tượng vô thời hạn cho tới khi lệnh Legal Hold được gỡ bỏ thủ công.

### 3.3. S3 Encryption at Rest (Mã hóa lưu trữ)
Mặc định hiện tại, toàn bộ bucket mới tạo đều được tự động bật mã hóa **SSE-S3**.

| Phương thức mã hóa | Quản lý khóa | Header yêu cầu | Audit Trail & Quyền kiểm soát |
| :--- | :--- | :--- | :--- |
| **SSE-S3** | AWS quản lý hoàn toàn khóa AES-256 | `x-amz-server-side-encryption: AES256` | Miễn phí, không theo dõi được ai decrypt |
| **SSE-KMS** | Khóa nằm trong **AWS KMS** (KMS CMK) | `x-amz-server-side-encryption: aws:kms` | **Ghi nhận log CloudTrail khi khóa được gọi**, hỗ trợ xoay khóa tự động (Key rotation). |
| **SSE-C** | Khách hàng tự sinh và giữ khóa, gửi kèm mỗi request qua HTTPS | Gửi key trong HTTP Header | AWS không lưu khóa; mất khóa là mất vĩnh viễn dữ liệu. |
| **Client-Side** | Ứng dụng tự mã hóa dữ liệu trước khi upload lên S3 | Tùy biến trên code app | Mã hóa hoàn toàn từ phía máy khách |

> ⚠️ **Bẫy hiệu năng của SSE-KMS**:
> - Khi tải hàng nghìn object/giây từ S3 được mã hóa bằng SSE-KMS, mỗi lần đọc file S3 phải gọi API `kms:Decrypt`. 
> - Nếu vượt ngưỡng quota của KMS (ví dụ 5,500 - 30,000 requests/s tùy Region), ứng dụng sẽ bị lỗi **KMS Throttling (HTTP 429)**.
> - **Giải pháp**: Bật tính năng **S3 Bucket Keys** (giảm tới 99% số lượng request gọi sang KMS).

---

## 4. S3 REPLICATION: CROSS-REGION (CRR) vs SAME-REGION (SRR)

* **Điều kiện tiên quyết**: Cả Bucket nguồn và Bucket đích **BẮT BUỘC PHẢI BẬT VERSIONING**.
* **Đặc tính cơ chế**:
  - Tự động replicate các object mới upload sau khi kích hoạt replication.
  - **Không tự động replicate các object có sẵn từ trước** (muốn replicate dữ liệu cũ phải dùng **S3 Batch Replication**).
  - Có thể replicate giữa **2 AWS Accounts khác nhau** (đổi chủ sở hữu object sang Destination Account: *Change Object Ownership*).
  - Replicate đối tượng mã hóa bằng KMS: Cần cấp quyền giải mã khóa nguồn và mã hóa lại bằng khóa KMS ở Region đích.

---

## 5. KIỂM SOÁT TRUY CẬP: BUCKET POLICIES, PRESIGNED URLS & ACCESS POINTS

### 5.1. S3 Bucket Policy (Resource-based Policy)
Dùng để kiểm soát quyền ở cấp độ toàn bộ bucket:
- **Bắt buộc truyền tải qua HTTPS (In-Transit Encryption)**:
```json
{
  "Statement": [
    {
      "Sid": "EnforceTLSRequestsOnly",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-secure-bucket",
        "arn:aws:s3:::my-secure-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

### 5.2. S3 Presigned URLs
* Cho phép người dùng hoặc hệ thống bên ngoài upload/download một file cụ thể **trong một khoảng thời gian giới hạn** mà **không cần cấp IAM credentials** cho họ.
* URL được ký bằng chính quyền IAM của người/dịch vụ tạo ra URL đó.

### 5.3. S3 Multi-Region Access Points (MRAP)
* Cung cấp một Global Endpoint duy nhất (dạng hostname toàn cầu) đứng trước nhiều S3 Buckets ở các Region khác nhau.
* Tự động định tuyến request của client qua mạng trục AWS tới bucket gần nhất với độ trễ thấp nhất.

---

## 6. LAB KIẾN TRÚC: HỆ THỐNG LƯU TRỮ TUÂN THỦ TÀI CHÍNH (WORM & DISASTER RECOVERY)

```
                            [ ỨNG DỤNG TÀI CHÍNH ]
                                       │
                                       │ Upload hóa đơn/giao dịch
                                       ▼
    ┌────────────────────────── REGION CHÍNH (SINGAPORE) ──────────────────────────┐
    │                                                                              │
    │   [ Primary S3 Bucket ]                                                      │
    │   • Versioning: ENABLED                                                      │
    │   • Mã hóa: SSE-KMS (Audit log qua CloudTrail) + S3 Bucket Keys              │
    │   • S3 Object Lock: COMPLIANCE MODE (3 năm không ai được xóa)                │
    │   • Bucket Policy: Deny nếu `aws:SecureTransport` là false                   │
    │   • Lifecycle Rule:                                                          │
    │     - Ngày 1 - 30:     S3 Standard                                           │
    │     - Ngày 31 - 90:    S3 Standard-IA                                        │
    │     - Ngày 91 - 365:   S3 Glacier Flexible Retrieval                         │
    │     - Sau 365 ngày:    S3 Glacier Deep Archive (Lưu 7 năm)                   │
    │                                                                              │
    └──────────────────────────────────┬───────────────────────────────────────────┘
                                       │
                                       │ Cross-Region Replication (CRR)
                                       │ (Đổi quyền sở hữu sang DR Account)
                                       ▼
    ┌────────────────────────── REGION DỰ PHÒNG (TOKYO) ──────────────────────────┐
    │                                                                              │
    │   [ Disaster Recovery S3 Bucket (Secondary Account) ]                        │
    │   • Versioning: ENABLED                                                      │
    │   • S3 Object Lock: COMPLIANCE MODE                                          │
    │   • Storage Class ngay khi sang: S3 Glacier Instant Retrieval                │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘
```
