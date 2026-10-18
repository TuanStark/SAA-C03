# NGÀY 13: HIGH AVAILABILITY & DISASTER RECOVERY (RTO/RPO, 4 CHIẾN LƯỢC DR, AWS BACKUP, AWS DRS)
*(Tuần 2: Security, Serverless & Resilience - Trọng số Domain 2: Design Resilient Architectures 26%)*

---

## 1. KHÁI NIỆM NỀN TẢNG: RTO & RPO

Trong bài thi SAA-C03, các câu hỏi về Phục hồi sau thảm họa (Disaster Recovery - DR) luôn xoay quanh 2 chỉ số thời gian sống còn:

```
    [ LẦN BACKUP GẦN NHẤT ]                 [ THẢM HỌA XẢY RA ]             [ HỆ THỐNG PHỤC HỒI ]
               │                                     │                                │
               ├────────────────── RPO ──────────────┤                                │
               │          (Data Loss / Mất dữ liệu)  │                                │
               │                                     ├────────────── RTO ─────────────┤
               │                                     │       (Downtime / Thời gian sập)
               ▼                                     ▼                                ▼
              10:00                                 12:00                            12:30
```

1. **RPO (Recovery Point Objective - Mục tiêu điểm phục hồi)**:
   - Khoảng thời gian tối đa chấp nhận được giữa thời điểm sao lưu dữ liệu gần nhất và thời điểm thảm họa xảy ra.
   - 👉 Câu hỏi cốt lõi: *"Doanh nghiệp chấp nhận bị **MẤT BAO NHIÊU DỮ LIỆU**?"*
   - *Ví dụ*: RPO = 1 giờ $\rightarrow$ Nếu thảm họa xảy ra, tối đa 1 giờ dữ liệu phát sinh gần nhất có thể bị mất.
2. **RTO (Recovery Time Objective - Mục tiêu thời gian phục hồi)**:
   - Khoảng thời gian tối đa chấp nhận được để khôi phục hệ thống từ lúc xảy ra sự cố cho đến khi người dùng có thể truy cập lại bình thường.
   - 👉 Câu hỏi cốt lõi: *"Hệ thống được phép **SẬP TRONG BAO LÂU (Downtime)**?"*
   - *Ví dụ*: RTO = 15 phút $\rightarrow$ Sau khi thảm họa xảy ra, trong vòng 15 phút hệ thống phải chạy lại.

---

## 2. BỐN CHIẾN LƯỢC DISASTER RECOVERY KINH ĐIỂN CỦA AWS

Thứ tự sắp xếp từ **Chi phí thấp nhất / RTO-RPO dài nhất** đến **Chi phí đắt nhất / RTO-RPO gần bằng 0**:

```
                       ┌────────────────────────────────────────────────────────┐
                       │          4 CHIẾN LƯỢC DISASTER RECOVERY (DR)           │
                       └───────────────────────────┬────────────────────────────┘
                                                   │
             ┌─────────────────────┬───────────────┴───────────────┬─────────────────────┐
             ▼                     ▼                               ▼                     ▼
     [ Backup & Restore ]   [ Pilot Light ]                [ Warm Standby ]       [ Multi-Site Active-Active ]
     • Chi phí: Rẻ nhất ($) • Chi phí: Thấp ($$)           • Chi phí: Vừa ($$$)   • Chi phí: Đắt nhất ($$$$)
     • RTO: Vài Giờ/Ngày    • RTO: Mười mấy Phút           • RTO: Vài Phút        • RTO: Gần như bằng 0 (Real-time)
     • RPO: Vài Giờ         • RPO: Vài Giây/Phút           • RPO: Vài Giây        • RPO: Gần như bằng 0 (Zero loss)
```

### Bảng so sánh chi tiết 4 chiến lược:

| Chiến lược DR | Bản chất dữ liệu (Data) | Bản chất tính toán (Compute) | RTO | RPO | Chi phí |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Backup & Restore** | Định kỳ sao lưu dữ liệu sang S3/Glacier ở Region khác (qua AWS Backup hoặc Snapshot replication). | **KHÔNG CÓ GÌ CHẠY**. Khi có thảm họa, dùng CloudFormation/Terraform dựng lại toàn bộ EC2, RDS từ đầu. | Hàng Giờ $\rightarrow$ Hàng Ngày | Hàng Giờ | **Rẻ nhất ($)** |
| **2. Pilot Light** | Dữ liệu cốt lõi được **sao chép liên tục (Replicated continuously)** sang Region dự phòng (ví dụ: Aurora Global DB, RDS Cross-Region Read Replica). | **Hạ tầng compute tắt hoặc ở quy mô 0** (Chỉ lưu sẵn AMIs, Launch Templates). Khi có sự cố: Promote DB lên Primary và bật Auto Scaling Group scale máy từ 0 lên N. | 10 – 30 Phút | Vài Giây – Vài Phút | **Thấp ($$)** |
| **3. Warm Standby** | Dữ liệu được đồng bộ liên tục thời gian thực. | Một **bản sao thu nhỏ (scaled-down version)** của hệ thống **LUÔN LUÔN CHẠY 24/7** ở Region dự phòng (ví dụ: chạy 2 máy EC2 nhỏ thay vì 20 máy lớn). Khi có sự cố: Route 53 chuyển hướng traffic và ASG scale-out lên đủ tải. | Dưới 10 Phút | Vài Giây | **Trung bình ($$$)** |
| **4. Multi-Site Active/Active** | Dữ liệu được sao chép 2 chiều thời gian thực (DynamoDB Global Tables, Aurora Global DB). | Hệ thống chạy **đầy đủ 100% công suất 24/7** đồng thời ở cả 2 hoặc nhiều Region. Cả 2 Region cùng phục vụ người dùng. | **Gần như bằng 0 (Tức thì)** | **Gần như bằng 0** | **Đắt nhất ($$$$)** |

> 💡 **BÍ KÍP BẮT TỪ KHÓA ĐỀ THI**:
> - *"Lowest cost, can tolerate hours of downtime and data loss"* $\rightarrow$ **Backup & Restore**.
> - *"Core data is continuously replicated, compute is launched only when disaster occurs"* $\rightarrow$ **Pilot Light**.
> - *"A minimal scaled-down version of the system runs 24/7 in secondary region, scale up when primary fails"* $\rightarrow$ **Warm Standby**.
> - *"Zero downtime, zero data loss, traffic served simultaneously from multiple regions"* $\rightarrow$ **Multi-Site Active/Active**.

---

## 3. CÁC DỊCH VỤ DỰ PHÒNG & SAO LƯU CHUYÊN BIỆT

### 3.1. AWS Backup (Quản trị sao lưu tập trung)
* **Phạm vi bảo vệ**: EC2, EBS, EFS, Amazon RDS, Aurora, DynamoDB, Amazon S3, AWS Storage Gateway.
* **Tính năng doanh nghiệp**:
  - **Cross-Region & Cross-Account Backup**: Tự động copy bản sao lưu sang một Region khác và sang một **AWS Account độc lập chuyên lưu trữ log/backup (Log Archive Account)** để chống ransomware.
  - **AWS Backup Vault Lock**: Khóa két sao lưu theo chuẩn WORM (Write Once, Read Many). Một khi đã bật Vault Lock ở chế độ Compliance, **tuyệt đối không ai (kể cả AWS Root Account)** có thể xóa bản sao lưu trước ngày hết hạn retention. Chống triệt để việc hacker chiếm quyền admin rồi xóa sạch backup.

### 3.2. AWS Elastic Disaster Recovery (AWS DRS - Tiền thân là CloudEndure)
* **Bản chất**: Dịch vụ sao chép khối cấp độ đĩa (Block-level continuous replication) từ máy chủ vật lý On-Premises, VMware, hoặc EC2 sang AWS.
* **Cơ chế tiết kiệm chi phí độc đáo (Staging Area)**:
  - AWS DRS liên tục ghi dữ liệu thay đổi vào một vùng đệm chi phí thấp (**Low-cost Staging Area**: gồm các ổ đĩa EBS gp3 giá rẻ và EC2 kích thước nhỏ `t3.small`).
  - Toàn bộ máy chủ sản xuất đắt tiền **KHÔNG CẦN CHẠY**.
  - Khi có thảm họa (Disaster Recovery Failover): AWS DRS tự động khởi chạy các máy chủ EC2 cấu hình lớn đúng chuẩn sản xuất trong vòng **vài phút**.
* 👉 *Từ khóa đề thi*: *"Migrate or replicate on-premises servers to AWS for disaster recovery with minimal cost and RTO of minutes"* $\rightarrow$ **AWS Elastic Disaster Recovery (AWS DRS)**.

---

## 4. MA TRẬN CHỌN CHIẾN LƯỢC DR THEO NGÂN SÁCH & RTO/RPO

```
      RTO/RPO
        ▲
Hàng Ngày │  [ Backup & Restore ]
         │  (Rẻ nhất, chấp nhận gián đoạn)
         │
Hàng Giờ │
         │         [ Pilot Light ]
15 Phút  │         (Core DB replicate, Compute bật khi cần)
         │
         │                  [ Warm Standby ]
 Vài Phút│                  (Bản sao thu nhỏ chạy 24/7)
         │
         │                           [ Multi-Site Active-Active ]
       0 └───────────────────────────(Không downtime, cực kỳ tốn kém)────► Chi phí ($)
          Rẻ                                                         Rất đắt
```

---

## 5. LAB KIẾN TRÚC: THIẾT KẾ DR TIẾT KIỆM CHI PHÍ CHO HỆ THỐNG TÀI CHÍNH

### Kịch bản Lab:
Một ngân hàng thương mại yêu cầu:
1. Ứng dụng ngân hàng lõi chạy trên EC2 và cơ sở dữ liệu Amazon Aurora MySQL tại Region Singapore.
2. Xây dựng phương án DR sang Region Tokyo với ngân sách hạn chế: **RPO phải dưới 1 phút (không được mất dữ liệu giao dịch)**, nhưng **RTO có thể chấp nhận trong vòng 15–20 phút**.
3. Không muốn tốn chi phí chạy hàng chục máy chủ EC2 nhàn rỗi ở Tokyo 24/7.

```
    ┌────────────────────── REGION CHÍNH: SINGAPORE (PRIMARY) ──────────────────────┐
    │                                                                               │
    │   [ Route 53 (Failover Routing) ] ──► Trỏ vào Primary ALB                     │
    │                                                                               │
    │   [ Application Load Balancer ]                                               │
    │              │                                                                │
    │              ▼                                                                │
    │   [ Auto Scaling Group: 20 x c5.2xlarge Instances (Chạy 24/7) ]               │
    │              │                                                                │
    │              ▼                                                                │
    │   [ Amazon Aurora Primary (Writer) ] ─────────────────────────┐               │
    │                                                               │               │
    └───────────────────────────────────────────────────────────────┼───────────────┘
                                                                    │
                                                                    │ Aurora Global Database
                                                                    │ (Sao chép liên tục RPO < 1s)
                                                                    ▼
    ┌────────────────────── REGION DỰ PHÒNG: TOKYO (PILOT LIGHT) ───────────────────┐
    │                                                                               │
    │   [ Route 53 (Failover Standby) ]                                             │
    │                                                                               │
    │   [ Application Load Balancer ]                                               │
    │              │                                                                │
    │              ▼                                                                │
    │   [ Auto Scaling Group: Min=0, Desired=0 (KHÔNG CHẠY MÁY NÀO - $0 COMPUTE) ]  │
    │     (Có sẵn Launch Template trỏ vào AMI đã replicate sang Tokyo)              │
    │                                                                               │
    │   [ Amazon Aurora Secondary (Read-Only) ] ◄───────────────────┘               │
    │                                                                               │
    └───────────────────────────────────────────────────────────────────────────────┘
```

### Quy trình kích hoạt Failover khi Singapore sập:
1. **Bước 1**: Nhấn nút **Promote** Aurora Secondary ở Tokyo lên thành Primary Writer mới (mất < 1 phút, dữ liệu nguyên vẹn vì RPO < 1s).
2. **Bước 2**: Cập nhật Auto Scaling Group ở Tokyo từ `Desired=0` lên `Desired=20` (ASG bật 20 máy c5.2xlarge trong vòng 3–5 phút).
3. **Bước 3**: Route 53 phát hiện Health Check Singapore chết $\rightarrow$ tự động điều hướng toàn bộ traffic người dùng sang ALB ở Tokyo.
4. **Kết quả**: Thỏa mãn RTO < 15 phút, RPO < 1 giây mà **tiết kiệm 100% chi phí chạy máy chủ EC2 dự phòng hàng tháng**!
