# BỘ CÂU HỎI TÌNH HUỐNG THỰC HÀNH - NGÀY 6: DATABASE SOLUTIONS

---

### CÂU 1 (Mức độ: Trung bình - Phân biệt RDS Multi-AZ vs Read Replica)
Một trang tin tức điện tử lớn chạy cơ sở dữ liệu trên một Amazon RDS MySQL instance. Trong các khung giờ có sự kiện "nóng", số lượng độc giả vào đọc bài tăng vọt khiến mức sử dụng CPU của RDS chạm ngưỡng 100%, dẫn đến hiện tượng nghẽn mạng và người dùng bị tải trang chậm. Phân tích CloudWatch cho thấy **hơn 95% lưu lượng vào database là các câu lệnh đọc (`SELECT`)**, trong khi các thao tác ghi (`INSERT`/`UPDATE`) của phóng viên biên tập bài viết vẫn ở mức rất thấp.

Giải pháp nào sau đây giúp giải quyết triệt để tình trạng quá tải CPU của database với **hiệu quả chi phí và kiến trúc tối ưu nhất**?

- **A.** Chuyển đổi RDS MySQL instance hiện tại sang mô hình **RDS Multi-AZ Deployment**.
- **B.** Tạo các **RDS Read Replicas** và cấu hình ứng dụng web gửi toàn bộ các câu truy vấn đọc (`SELECT`) tới các Endpoint của Read Replicas.
- **C.** Nâng cấp instance type của RDS instance lên kích thước lớn gấp đôi (Scale-up compute).
- **D.** Kích hoạt tính năng RDS Storage Auto Scaling để tự động tăng dung lượng đĩa EBS khi đầy.

---

### CÂU 2 (Mức độ: Nâng cao - Aurora Disaster Recovery Toàn cầu)
Một công ty tài chính đa quốc gia lưu trữ dữ liệu giao dịch cốt lõi trên một cụm **Amazon Aurora MySQL** tại Region Singapore (`ap-southeast-1`). Để đáp ứng tiêu chuẩn an toàn thông tin của ngân hàng trung ương, công ty cần xây dựng phương án Disaster Recovery (DR) sang Region Tokyo (`ap-northeast-1`).
Yêu cầu đặt ra:
1. Độ trễ sao chép dữ liệu giữa 2 Region phải ở mức **dưới 1 giây** (RPO < 1s).
2. Khi thảm họa xảy ra tại Singapore, Region Tokyo phải được thăng cấp (promote) thành cụm nhận ghi chính thức trong thời gian **dưới 1 phút** (RTO < 1 min) mà **không làm mất tính nhất quán của dữ liệu**.

Giải pháp kiến trúc nào sau đây đáp ứng chính xác các chỉ số RPO/RTO trên?

- **A.** Tạo một Amazon Aurora Read Replica liên vùng (Cross-Region Read Replica) tại Region Tokyo.
- **B.** Thiết lập **Amazon Aurora Global Database** với Primary DB đặt tại Singapore và Secondary DB đặt tại Tokyo.
- **C.** Sử dụng AWS Database Migration Service (AWS DMS) để sao chép liên tục dữ liệu từ Singapore sang một cơ sở dữ liệu RDS MySQL tại Tokyo.
- **D.** Cấu hình snapshot tự động cho Aurora mỗi giờ và sử dụng AWS Backup để copy snapshot sang Tokyo.

---

### CÂU 3 (Mức độ: Nâng cao - DynamoDB Microsecond Latency)
Một nền tảng thương mại điện tử sử dụng Amazon DynamoDB để lưu trữ thông tin tồn kho và giá của hàng triệu sản phẩm. Trong ngày hội mua sắm Mega Sale, một số sản phẩm "hot deal" nhận được hàng trăm nghìn lượt truy cập đọc mỗi giây, dẫn đến hiện tượng "hot partition" và người dùng phản hồi rằng thời gian tải thông tin sản phẩm bị chậm. 
Đội ngũ phát triển yêu cầu một giải pháp bộ nhớ đệm (caching) để **giảm độ trễ đọc xuống mức vi-giây (microsecond latency)** cho DynamoDB mà **không phải sửa đổi cấu trúc logic truy vấn phức tạp của ứng dụng**.

Giải pháp nào sau đây là tối ưu nhất?

- **A.** Triển khai cụm **Amazon DynamoDB Accelerator (DAX)** phía trước bảng DynamoDB và trỏ SDK của ứng dụng tới DAX cluster.
- **B.** Triển khai một cụm Amazon ElastiCache for Memcached và viết lại mã nguồn ứng dụng để kiểm tra cache trước khi truy vấn DynamoDB.
- **C.** Bật tính năng DynamoDB Global Tables để nhân bản bảng sang 3 Region khác nhằm chia tải đọc.
- **D.** Tăng số lượng Read Capacity Units (RCU) của bảng DynamoDB lên 100,000 RCU ở chế độ Provisioned Mode.

---

### CÂU 4 (Mức độ: Trung bình - ElastiCache Engine Selection)
Một công ty game online đang phát triển tính năng **Bảng xếp hạng thời gian thực (Real-time Leaderboard)** cho các game thủ trên toàn thế giới. Bảng xếp hạng này liên tục cập nhật thứ tự điểm số của hàng triệu người chơi và hiển thị Top 100 game thủ xuất sắc nhất.
Yêu cầu kỹ thuật:
1. Dữ liệu xếp hạng phải được lưu trữ trong bộ nhớ đệm (in-memory) để truy xuất tức thì với độ trễ sub-millisecond.
2. Dịch vụ bộ nhớ đệm phải hỗ trợ cấu trúc dữ liệu sắp xếp theo điểm số (**Sorted Sets**).
3. Hệ thống phải có khả năng tự động khôi phục (Auto-Failover) khi xảy ra sự cố phần cứng trên cụm máy chủ đệm.

Dịch vụ nào sau đây đáp ứng toàn bộ các yêu cầu trên?

- **A.** Amazon ElastiCache for Memcached.
- **B.** **Amazon ElastiCache for Redis** (kích hoạt chế độ Multi-AZ với Auto-Failover).
- **C.** Amazon DynamoDB với chế độ Eventually Consistent Reads.
- **D.** Amazon RDS for PostgreSQL với bảng lưu trữ Unlogged Tables.

---

### CÂU 5 (Mức độ: Nâng cao - Multi-Region Active-Active Database)
Một ứng dụng đặt vé máy bay toàn cầu cần cho phép khách hàng tại cả Bắc Mỹ và Châu Âu có thể tìm kiếm, đặt vé và sửa đổi thông tin vé máy bay với **độ trễ đọc và ghi cực thấp (low-latency local reads and writes)** tại khu vực địa lý của họ. 
Hệ thống yêu cầu dữ liệu đặt vé phải được sao chép hai chiều liên tục giữa các Region để đảm bảo tính sẵn sàng cao, nếu toàn bộ một AWS Region gặp sự cố thì khách hàng vẫn có thể tiếp tục đặt vé ở Region còn lại mà không bị gián đoạn.

Giải pháp cơ sở dữ liệu nào sau đây là phù hợp nhất?

- **A.** Amazon RDS PostgreSQL Multi-AZ Deployment kết hợp với Cross-Region Read Replicas.
- **B.** **Amazon DynamoDB Global Tables** với bảng dữ liệu được tạo ở cả hai Region Bắc Mỹ và Châu Âu.
- **C.** Amazon Aurora MySQL Cluster với cấu hình Multi-Master tại Region Bắc Mỹ.
- **D.** Amazon DocumentDB (tương thích MongoDB) với tính năng Global Clusters.

---

---

## ĐÁP ÁN & PHÂN TÍCH CHUYÊN SÂU DÀNH CHO ARCHITECT

### Đáp án Câu 1: **B**
- **Tại sao đúng**: 
  - Đề bài xác định rõ: **CPU quá tải do hơn 95% lưu lượng là lệnh đọc (`SELECT`)**.
  - **RDS Read Replicas** được thiết kế chuyên biệt để giải quyết bài toán **Scale-out lưu lượng đọc**. Bạn có thể tạo tối đa 15 Read Replicas (chạy bất đồng bộ) và phân phối các kết nối đọc của ứng dụng sang các Replicas này, giải phóng hoàn toàn gánh nặng CPU cho Primary instance để chuyên tâm xử lý các lệnh ghi (`INSERT`/`UPDATE`).
- **Phân tích bẫy**:
  - *Phương án A (Bẫy kinh điển)*: **RDS Multi-AZ KHÔNG DÙNG ĐỂ TĂNG HIỆU NĂNG ĐỌC**. Standby instance trong Multi-AZ là một máy thụ động (passive/idle), không có endpoint cho ứng dụng kết nối vào đọc, nó chỉ có nhiệm vụ duy nhất là sẵn sàng failover khi Primary chết.
  - *Phương án C*: Scale-up CPU giải quyết tạm thời nhưng chi phí rất đắt (Vertical scaling luôn chạm trần vật lý) và không phải là kiến trúc tối ưu theo AWS Well-Architected.
  - *Phương án D*: Đề bài bị nghẽn CPU chứ không phải hết dung lượng ổ đĩa, tăng storage không giải quyết được vấn đề.

---

### Đáp án Câu 2: **B**
- **Tại sao đúng**: 
  - Yêu cầu đặt ra cực kỳ khắt khe: **RPO < 1s** và **RTO < 1 min** giữa 2 Region cách nhau hàng ngàn km.
  - **Amazon Aurora Global Database** là giải pháp duy nhất của AWS thỏa mãn các thông số này:
    - Sao chép ở cấp độ hạ tầng lưu trữ phân tán chuyên dụng (Storage layer replication) không tốn CPU của instance, độ trễ sao chép thông thường **dưới 1 giây**.
    - Khi Region chính sập, tính năng **Cross-Region Failover / Promote** của Global Database cho phép biến Secondary Region thành Primary mới chỉ trong **vòng dưới 1 phút (thậm chí dưới 30 giây)** mà không bị mất dữ liệu (zero data loss).
- **Phân tích bẫy**:
  - *Phương án A*: Cross-Region Read Replica thông thường của Aurora/RDS dựa trên cơ chế replication logic truyền thống (MySQL binlog/PostgreSQL WAL) có độ trễ lớn hơn và thời gian thăng cấp (promote) thường mất vài phút đến hàng chục phút, không cam kết RTO < 1 min.
  - *Phương án C*: AWS DMS dùng cho di chuyển dữ liệu (Migration), không tối ưu cho kiến trúc HA/DR liên vùng của Aurora.
  - *Phương án D*: Backup/Snapshot hàng giờ thì khi có sự cố, dữ liệu của 59 phút gần nhất sẽ bị mất trắng $\rightarrow$ RPO lên tới 1 giờ, vi phạm chỉ số RPO < 1s.

---

### Đáp án Câu 3: **A**
- **Tại sao đúng**: 
  - Đề bài có 2 từ khóa chốt: **"giảm độ trễ đọc xuống mức vi-giây (microsecond latency)"** và **"không phải sửa đổi cấu trúc logic truy vấn phức tạp của ứng dụng (transparent cache)"**.
  - **DynamoDB Accelerator (DAX)** là cụm bộ nhớ đệm in-memory được thiết kế riêng cho DynamoDB:
    - Cung cấp tốc độ đọc ở cấp độ **microsecond** (nhanh gấp 10 lần mức millisecond của DynamoDB gốc).
    - Là bộ nhớ đệm tương thích 100% với DynamoDB API: Lập trình viên chỉ cần trỏ SDK vào DAX endpoint thay vì DynamoDB endpoint, không cần viết lại logic kiểm tra cache (như `get_from_cache`, `set_cache`).
- **Phân tích bẫy**:
  - *Phương án B*: ElastiCache Memcached đòi hỏi phải viết lại toàn bộ mã nguồn ứng dụng để quản lý logic caching (Cache-Aside pattern), vi phạm ràng buộc đề bài.
  - *Phương án C*: Global Tables dùng cho đa vùng, không kéo độ trễ từ millisecond xuống microsecond.
  - *Phương án D*: Tăng RCU chỉ giúp xử lý nhiều request hơn nhưng độ trễ vẫn là **mili-giây (single-digit milliseconds)**, không thể đạt mức **vi-giây (microsecond)**, đồng thời rất tốn kém chi phí nếu gặp hot partition.

---

### Đáp án Câu 4: **B**
- **Tại sao đúng**: 
  - Yêu cầu đề bài: **Bảng xếp hạng (Leaderboard)**, **cấu trúc dữ liệu có sắp xếp (Sorted Sets)**, và **tự động khôi phục (Auto-Failover)**.
  - **Amazon ElastiCache for Redis**:
    - Hỗ trợ kiểu dữ liệu bản địa **Sorted Sets (ZSET)**: Cho phép lưu trữ cặp `(Score, Member)` và tự động sắp xếp theo điểm số với độ phức tạp $O(\log N)$, cực kỳ lý tưởng cho Leaderboard.
    - Hỗ trợ kiến trúc **Multi-AZ với tính năng Auto-Failover** tự động thăng cấp Replica thành Primary khi node chính bị hỏng phần cứng.
- **Phân tích bẫy**:
  - *Phương án A*: **Memcached** chỉ là bộ đệm Key-Value chuỗi đơn giản, **KHÔNG hỗ trợ Sorted Sets**, và **KHÔNG có tính năng Multi-AZ Auto-Failover**.
  - *Phương án C*: DynamoDB không tối ưu bằng in-memory Redis cho bài toán liên tục cập nhật và sort real-time hàng triệu bản ghi mỗi giây.
  - *Phương án D*: RDS PostgreSQL lưu trữ trên ổ đĩa, không phải là in-memory cache sub-millisecond cho bảng xếp hạng game trực tuyến.

---

### Đáp án Câu 5: **B**
- **Tại sao đúng**: 
  - Đề bài yêu cầu: **Đọc VÀ GHI phân tán tại chỗ với độ trễ thấp (Active-Active local reads and writes)** tại cả 2 Region Bắc Mỹ và Châu Âu, kết hợp sao chép 2 chiều.
  - **Amazon DynamoDB Global Tables** là giải pháp Multi-Region, Multi-Active (Active-Active) được quản lý hoàn toàn của AWS:
    - Cho phép khách hàng ở Bắc Mỹ đọc/ghi vào bảng ở Bắc Mỹ, khách hàng Châu Âu đọc/ghi vào bảng ở Châu Âu.
    - Dữ liệu được đồng bộ 2 chiều tự động ngầm giữa các Region.
    - Khi 1 Region sập, ứng dụng tự động chuyển hướng traffic sang Region còn lại mà không cần bất kỳ bước promote hay failover phức tạp nào.
- **Phân tích bẫy**:
  - *Phương án A*: RDS Read Replica chỉ hỗ trợ **ĐỌC (Read-Only)** ở Region thứ 2, mọi lệnh ghi vẫn phải gửi về Region chính ở Bắc Mỹ $\rightarrow$ Không đáp ứng yêu cầu local writes độ trễ thấp tại Châu Âu.
  - *Phương án C*: Aurora Multi-Master chỉ hoạt động trong **cùng 1 Region** (giữa các AZs), không hỗ trợ Multi-Master xuyên lục địa qua nhiều Regions.
  - *Phương án D*: DocumentDB Global Clusters là kiến trúc Single-Master (chỉ 1 Region nhận ghi, các Region khác chỉ đọc).
