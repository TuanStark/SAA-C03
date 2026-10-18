# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 13: HIGH AVAILABILITY & DISASTER RECOVERY

---

### CÂU 1 (Mức độ: Trung bình/Nâng cao - Nhận diện chiến lược Disaster Recovery)
Một doanh nghiệp vận hành một cổng thông tin nhân sự trên AWS tại Region `us-east-1`. Doanh nghiệp cần xây dựng phương án khắc phục thảm họa (Disaster Recovery) sang Region `us-west-2` với tiêu chí:
- Dữ liệu cơ sở dữ liệu phải được đồng bộ liên tục giữa 2 Region để đảm bảo RPO tính bằng giây.
- Tại Region dự phòng, doanh nghiệp duy trì **một phiên bản thu nhỏ (scaled-down version)** của hệ thống gồm 2 EC2 instances nhỏ luôn chạy 24/7 để tiếp nhận lưu lượng kiểm thử nội bộ.
- Khi thảm họa xảy ra tại `us-east-1`, hệ thống tại `us-west-2` sẽ được tự động scale-out tăng số lượng instances lên mức phục vụ toàn bộ người dùng và chuyển hướng DNS trong vòng **dưới 10 phút**.

Chiến lược Disaster Recovery nào sau đây đang được triển khai?

- **A.** Backup & Restore
- **B.** Pilot Light
- **C.** **Warm Standby**
- **D.** Multi-Site Active/Active

---

### CÂU 2 (Mức độ: Nâng cao - Tối ưu hóa chi phí cho Chiến lược Pilot Light)
Một công ty tài chính có hệ thống thanh toán cốt lõi chạy trên 30 EC2 instances và Amazon RDS Multi-AZ tại Region Frankfurt. Ban giám đốc yêu cầu thiết lập giải pháp Disaster Recovery sang Region Ireland với yêu cầu:
1. **RPO không quá 5 phút** (không được mất mát nhiều dữ liệu giao dịch).
2. **RTO chấp nhận được trong khoảng 15 đến 30 phút**.
3. **Chi phí hàng tháng cho tài nguyên compute tại Region dự phòng phải ở mức tối thiểu tuyệt đối (gần như bằng 0)** khi chưa xảy ra thảm họa.

Kiến trúc sư nên đề xuất giải pháp nào để tối ưu chi phí theo mô hình **Pilot Light**?

- **A.** Chạy 30 EC2 instances ở Region Ireland ở chế độ On-Demand nhưng tắt nguồn mỗi đêm.
- **B.** Thiết lập một **Amazon RDS Cross-Region Read Replica** tại Region Ireland. Tạo một **Auto Scaling Group tại Ireland với Desired Capacity = 0** dựa trên Launch Template chứa AMI mới nhất. Khi có sự cố, promote Read Replica thành Primary và tăng Desired Capacity của ASG lên 30.
- **C.** Sử dụng AWS Backup để chụp snapshot RDS mỗi 6 giờ và copy sang Ireland.
- **D.** Triển khai giải pháp Multi-Site Active/Active với Route 53 Weighted Routing 50/50.

---

### CÂU 3 (Mức độ: Nâng cao - Replicate On-Premises sang AWS với AWS DRS)
Một tập đoàn sở hữu hàng trăm máy chủ vật lý và máy ảo VMware tại trung tâm dữ liệu On-Premises. Tập đoàn muốn thiết lập giải pháp khôi phục thảm họa (Disaster Recovery) lên AWS với mục tiêu:
1. Sao chép liên tục dữ liệu cấp độ khối (Block-level replication) từ On-Premises lên AWS.
2. Giảm thiểu tối đa chi phí hạ tầng trên AWS trong điều kiện vận hành bình thường (không muốn trả tiền cho hàng trăm máy chủ EC2 cấu hình cao chạy nhàn rỗi).
3. Đạt chỉ số RTO trong vòng vài phút khi trung tâm dữ liệu On-Premises bị mất điện hoặc hỏng hóc.

Dịch vụ nào của AWS được thiết kế tối ưu nhất cho kịch bản này?

- **A.** AWS Database Migration Service (AWS DMS)
- **B.** AWS DataSync
- **C.** **AWS Elastic Disaster Recovery (AWS DRS)**
- **D.** AWS Snowball Edge

---

### CÂU 4 (Mức độ: Nâng cao - Chống Ransomware & Bảo vệ Bản sao lưu với Vault Lock)
Một tổ chức y tế sử dụng **AWS Backup** để quản lý việc sao lưu tự động các cơ sở dữ liệu và ổ đĩa EBS lưu trữ hồ sơ bệnh án sang một tài khoản sao lưu độc lập (Backup Account). Để phòng chống nguy cơ tấn công của mã độc tống tiền (Ransomware) hoặc trường hợp tài khoản quản trị viên bị lộ thông tin đăng nhập:
Tổ chức yêu cầu: **Tuyệt đối không một ai — kể cả quản trị viên hệ thống hoặc tài khoản AWS Root Account — có thể xóa, rút ngắn thời hạn retention, hoặc can thiệp vào các bản sao lưu đã được tạo trong két lưu trữ (Backup Vault) trong thời hạn 3 năm**.

Tính năng nào sau đây đáp ứng chính xác yêu cầu trên?

- **A.** Kích hoạt S3 Versioning trên các bucket sao lưu.
- **B.** Kích hoạt **AWS Backup Vault Lock** ở chế độ **Compliance Mode** trên Backup Vault.
- **C.** Kích hoạt AWS Backup Vault Lock ở chế độ Governance Mode.
- **D.** Tạo một IAM Policy gắn vào tất cả user với điều kiện Deny hành động `backup:DeleteRecoveryPoint`.

---

### CÂU 5 (Mức độ: Khó - Yêu cầu RTO & RPO gần bằng 0)
Một sàn giao dịch tiền điện tử quốc tế yêu cầu hệ thống phải duy trì tính liên tục của hoạt động kinh doanh ở mức cao nhất:
- Tuyệt đối không chấp nhận bất kỳ khoảng thời gian gián đoạn dịch vụ nào (**Zero Downtime / RTO gần như bằng 0**).
- Tuyệt đối không chấp nhận mất mát dữ liệu giao dịch (**Zero Data Loss / RPO gần như bằng 0**).
- Người dùng ở Châu Á và Châu Mỹ phải được phục vụ đồng thời bởi các cụm máy chủ đặt tại khu vực địa lý của họ để đạt độ trễ thấp nhất.

Kiến trúc sư nên lựa chọn chiến lược nào sau đây?

- **A.** Chiến lược Pilot Light sử dụng Route 53 Failover Routing.
- **B.** Chiến lược Warm Standby với Amazon Aurora Global Database.
- **C.** Chiến lược **Multi-Site Active/Active** sử dụng **Amazon DynamoDB Global Tables** phân bổ trên 2 Region kết hợp với cụm compute chạy đủ tải 24/7 ở cả 2 Region và điều phối bằng **Route 53 Geolocation / Latency-based Routing**.
- **D.** Chiến lược Backup & Restore sử dụng AWS Elastic Disaster Recovery.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **C**
- **Tại sao đúng**: 
  - Đề bài mô tả rõ ràng: **"duy trì một phiên bản thu nhỏ (scaled-down version) gồm 2 EC2 instances nhỏ luôn chạy 24/7 ở Region dự phòng"**, và dữ liệu được replicate liên tục.
  - Đây chính là định nghĩa chuẩn mực của **Warm Standby**:
    - Khác với Pilot Light (ở Pilot Light, hạ tầng compute tắt hoàn toàn hoặc số instance = 0).
    - Ở Warm Standby, một hệ thống thu nhỏ luôn hoạt động 24/7, có thể nhận một phần tải nhỏ (như test nội bộ), và khi có sự cố, hệ thống chỉ cần scale-out các instances hiện có lên đủ tải lớn trong vòng vài phút.
- **Phân tích bẫy**:
  - *Phương án A*: Backup & Restore không có máy nào chạy sẵn và RTO tính bằng giờ/ngày.
  - *Phương án B*: Pilot Light không chạy các máy EC2 thu nhỏ 24/7 (compute tắt hoàn toàn).
  - *Phương án D*: Multi-Site Active/Active chạy 100% công suất ở cả 2 Region, không phải phiên bản thu nhỏ.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - Yêu cầu đề bài: **Chiến lược Pilot Light**, **RPO < 5 phút**, **RTO 15-30 phút**, và **chi phí compute dự phòng hàng tháng gần như bằng 0**.
  - Mô hình Pilot Light chuẩn AWS:
    - **Tầng dữ liệu (Data)**: Duy trì một **RDS Cross-Region Read Replica** (hoặc Aurora Global DB). Dữ liệu được đồng bộ liên tục $\rightarrow$ thỏa mãn RPO tính bằng giây/phút. Chi phí chỉ tốn cho 1 database replica nhỏ.
    - **Tầng tính toán (Compute)**: Cấu hình sẵn một **Auto Scaling Group với `Desired = 0` và `Min = 0`** $\rightarrow$ **Hoàn toàn KHÔNG tốn chi phí chạy EC2 hàng tháng ($0)**.
    - Khi có thảm họa: Promote Read Replica thành Master DB (mất 2-3 phút) và tăng Desired của ASG từ 0 lên 30 (mất 5-10 phút) $\rightarrow$ Hoàn tất khôi phục trong vòng dưới 15-20 phút (đáp ứng RTO).
- **Phân tích bẫy**:
  - *Phương án A*: Chạy 30 instances dù tắt ban đêm vẫn tốn chi phí rất lớn ban ngày, vi phạm ràng buộc chi phí compute tối thiểu.
  - *Phương án C*: Snapshot 6 giờ/lần làm RPO lên tới 6 giờ, vi phạm yêu cầu RPO < 5 phút.
  - *Phương án D*: Multi-Site Active/Active có chi phí đắt đỏ nhất toàn bộ các phương án.

---

### Đáp án Câu 3: **C**
- **Tại sao đúng**: 
  - **AWS Elastic Disaster Recovery (AWS DRS)** là giải pháp chuyên dụng của AWS để sao chép dự phòng thảm họa từ On-Premises (hoặc giữa các Cloud) lên AWS:
    - Cơ chế sao chép khối liên tục (Continuous block-level replication) đạt RPO tính bằng giây.
    - **Tối ưu chi phí bằng Staging Area**: Trong điều kiện bình thường, AWS DRS chỉ ghi dữ liệu vào các ổ đĩa EBS gp3 giá rẻ và vài EC2 rất nhỏ (`t3.small`) $\rightarrow$ Chi phí duy trì cực thấp.
    - Khi có sự cố thảm họa: DRS tự động chuyển đổi định dạng và khởi chạy hàng loạt các máy chủ EC2 cấu hình lớn đúng chuẩn môi trường sản xuất chỉ trong vòng **vài phút (RTO vài phút)**.
- **Phân tích bẫy**:
  - *Phương án A*: AWS DMS chỉ dùng để di chuyển dữ liệu của database, không sao chép được toàn bộ hệ điều hành và ổ đĩa máy chủ.
  - *Phương án B*: AWS DataSync dùng để đồng bộ file/object theo lịch trình, không phải giải pháp tự động failover cho máy chủ vật lý.
  - *Phương án D*: Snowball Edge là thiết bị vận chuyển vật lý, thời gian vận chuyển mất nhiều ngày.

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**: 
  - **AWS Backup Vault Lock ở chế độ Compliance Mode** là giải pháp tối thượng để phòng chống mã độc tống tiền (Ransomware) và xóa dữ liệu có chủ đích từ nội bộ:
    - Khi đã bật Compliance Mode và qua thời gian ân hạn (grace period), **KHÔNG AI — kể cả tài khoản AWS Root Account hay quản trị viên cấp cao nhất — CÓ THỂ XÓA HOẶC RÚT NGẮN THỜI HẠN RETENTION CỦA BẢN SAO LƯU**.
    - Đảm bảo dữ liệu sao lưu luôn tồn tại nguyên vẹn trong 3 năm để phục hồi khi hệ thống chính bị tấn công.
- **Phân tích bẫy**:
  - *Phương án A*: S3 Versioning chỉ áp dụng cho file trong S3, không bảo vệ được bản sao lưu tập trung của EBS, RDS, EFS trong AWS Backup.
  - *Phương án C*: **Governance Mode** cho phép người dùng có quyền đặc biệt (hoặc tài khoản Root) xóa được bản sao lưu, không đáp ứng yêu cầu cấm tuyệt đối tài khoản Root.
  - *Phương án D*: Tài khoản Root không bị ảnh hưởng bởi IAM Policy và có thể tự gỡ IAM Policy để xóa dữ liệu.

---

### Đáp án Câu 5: **C**
- **Tại sao đúng**: 
  - Yêu cầu đề bài: **Zero Downtime (RTO gần bằng 0)**, **Zero Data Loss (RPO gần bằng 0)**, và **phục vụ đồng thời người dùng ở cả 2 khu vực địa lý**.
  - Đây là đặc điểm nhận diện độc quyền của chiến lược **Multi-Site Active/Active**:
    - **Cơ sở dữ liệu**: Sử dụng **Amazon DynamoDB Global Tables** (hoặc Aurora Global DB) cho phép đọc và ghi đa chiều đồng thời ở cả 2 Region với độ trễ thấp.
    - **Compute**: Cụm máy chủ chạy đầy đủ 100% công suất 24/7 ở cả 2 Region.
    - **Traffic Routing**: Sử dụng Route 53 Latency-based Routing hoặc Geolocation Routing để phân phối người dùng vào Region gần nhất. Nếu 1 Region bị sập hoàn toàn, Route 53 tự động dẫn 100% traffic sang Region còn lại mà không có bất kỳ downtime nào.
- **Phân tích bẫy**:
  - *Phương án A & B*: Pilot Light và Warm Standby đều là mô hình Active-Passive, khi xảy ra sự cố vẫn phải mất từ vài phút đến 15 phút để failover, không thể đạt Zero Downtime.
  - *Phương án D*: Backup & Restore có RTO tính bằng giờ/ngày.
