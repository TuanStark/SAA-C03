# NGÀY 15: CHIẾN THUẬT LÀM ĐỀ THI 65 CÂU (130 PHÚT) & MA TRẬN PHÂN TÍCH LỖI
*(Tuần 3: Luyện đề chuyên sâu theo 4 Domain - Chuyển dịch từ "Biết AWS" sang "Đỗ chứng chỉ 850+")*

---

## 1. TỔNG QUAN CẤU TRÚC ĐỀ THI SAA-C03 CHUẨN QUỐC TẾ

* **Tổng số câu**: **65 câu hỏi trắc nghiệm** (gồm câu 1 đáp án và câu chọn 2/3 đáp án đúng).
* **Thời gian làm bài**: **130 phút** (Trung bình **2 phút / câu**).
* **Cơ chế tính điểm**:
  - Thang điểm: từ **100 đến 1000 điểm**. Điểm đỗ chính thức là **720/1000**.
  - Trong 65 câu có **50 câu được tính điểm** và **15 câu thử nghiệm (Unscored questions)** của AWS để đánh giá đề thi mới. Bạn không thể biết câu nào được tính điểm, nên phải làm cẩn thận toàn bộ 65 câu.
* **Tỷ trọng điểm theo 4 Domain chuẩn AWS Exam Guide**:
  1. **Domain 1**: Design Secure Architectures — **30%** (~20 câu)
  2. **Domain 2**: Design Resilient Architectures — **26%** (~17 câu)
  3. **Domain 3**: Design High-Performing Architectures — **24%** (~15 câu)
  4. **Domain 4**: Design Cost-Optimized Architectures — **20%** (~13 câu)

---

## 2. CHIẾN THUẬT PHÂN BỔ THỜI GIAN "3 LƯỢT" (THE 3-PASS STRATEGY)

Để không bao giờ bị thiếu giờ và tránh căng thẳng tâm lý khi gặp các câu hỏi tình huống dài 5-7 dòng:

```
    ┌─────────────────────────────────────────────────────────────────────────────┐
    │                      CHIẾN THUẬT LÀM BÀI 3 LƯỢT (130 PHÚT)                  │
    ├──────────────────────┬──────────────────────┬───────────────────────────────┤
    │  LƯỢT 1 (60 Phút)    │  LƯỢT 2 (50 Phút)    │      LƯỢT 3 (20 Phút)         │
    │  Giải quyết câu dễ   │  Giải quyết câu khó  │      Rà soát câu Flag         │
    ├──────────────────────┼──────────────────────┼───────────────────────────────┤
    │ • Làm ngay các câu   │ • Đọc kỹ các bài     │ • Kiểm tra lại các câu        │
    │   ngắn, câu định nghĩa│   toán kiến trúc dài │   đã đánh dấu (Flag for review)│
    │ • Bắt từ khóa chốt   │ • Dùng phương pháp   │ • Tuyệt đối KHÔNG ĐỔI đáp án   │
    │ • Câu nào phân vân   │   loại trừ (2/4)     │   trừ khi tìm thấy bằng chứng │
    │   $\rightarrow$ FLAG │ • Chốt đáp án tối ưu │   chắc chắn mình đã đọc sót đề│
    └──────────────────────┴──────────────────────┴───────────────────────────────┘
```

### Quy trình 4 bước bóc tách câu hỏi tình huống dài:
1. **Bước 1: Đọc câu hỏi cuối cùng trước**: Xác định ngay mục tiêu: Đề bài yêu cầu *"Most cost-effective"*, *"Least operational overhead"*, *"Highest availability"*, hay *"Near real-time"*?
2. **Bước 2: Tìm các ràng buộc bắt buộc (Hard Constraints)** trong thân bài:
   - Ví dụ: Không được dùng Internet công cộng? RTO dưới 15 phút? Lưu trữ chuẩn POSIX Linux? Không lưu Access Key dài hạn?
3. **Bước 3: Loại bỏ các đáp án sai nguyên lý hoặc vi phạm ràng buộc**: Thường sẽ có 2 đáp án sai rõ rệt (ví dụ: gán Elastic IP cho ALB, hoặc chọn S3 Standard cho dữ liệu lưu 7 năm).
4. **Bước 4: So sánh 2 đáp án khả thi còn lại**: Chọn đáp án nào bám sát từ khóa ở Bước 1 nhất (ví dụ: giữa một giải pháp tự code script EC2 rẻ hơn và một dịch vụ AWS Managed, nếu đề hỏi *"least operational effort"* thì luôn chọn **AWS Managed Service**).

---

## 3. MA TRẬN ĐÁNH GIÁ ĐIỂM SỐ & CHẨN ĐOÁN THEO 4 DOMAIN

Sau khi làm xong đề 65 câu hôm nay, hãy điền kết quả vào bảng này để xác định chính xác điểm yếu:

| Domain | Trọng số đề thi | Số câu trong đề | Số câu làm đúng | Tỷ lệ % đạt | Đánh giá mức độ |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Domain 1: Secure Architectures** | 30% | 20 câu | ... / 20 | ... % | $\ge$ 80%: Đạt \| < 80%: Cần củng cố Ngày 16 |
| **Domain 2: Resilient Architectures**| 26% | 17 câu | ... / 17 | ... % | $\ge$ 80%: Đạt \| < 80%: Cần củng cố Ngày 17 |
| **Domain 3: High-Performing** | 24% | 15 câu | ... / 15 | ... % | $\ge$ 80%: Đạt \| < 80%: Cần củng cố Ngày 18 |
| **Domain 4: Cost-Optimized** | 20% | 13 câu | ... / 13 | ... % | $\ge$ 80%: Đạt \| < 80%: Cần củng cố Ngày 19 |
| **TỔNG CỘNG** | **100%** | **65 câu** | **... / 65** | **... %** | **Mục tiêu chẩn đoán: $\ge$ 52/65 (80%+)** |

---

## 4. MA TRẬN 4 NGUYÊN NHÂN LỖI SAI (ROOT CAUSE LOGGING)

Khi phân tích các câu làm sai hoặc phân vân, hãy phân loại ngay vào 1 trong 4 nhóm nguyên nhân sau trong file `Error-Log-Template.md`:

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                   4 NHÓM NGUYÊN NHÂN LỖI SAI                                     │
├──────────────────────────────┬──────────────────────────────┬────────────────────────────────────┤
│ 1. Thiếu kiến thức ngầm      │ Không biết NAT GW chỉ HA     │ Ôn lại cơ chế ngầm và các giới hạn │
│    (Internals & Limits)      │ trong 1 AZ, không biết S3    │ kỹ thuật cứng của dịch vụ.         │
│                              │ Object Lock cần Versioning...│                                    │
├──────────────────────────────┼──────────────────────────────┼────────────────────────────────────┤
│ 2. Nhầm lẫn cặp dịch vụ      │ RDS Multi-AZ vs Read Replica,│ Đọc lại bảng Master Cheat Sheet ở  │
│    tương đồng                │ CloudFront vs Global Accel,  │ Ngày 7 và Ngày 14 để phân biệt     │
│                              │ Secrets Mgr vs Parameter St. │ sự khác biệt cốt lõi.              │
├──────────────────────────────┼──────────────────────────────┼────────────────────────────────────┤
│ 3. Đọc sót từ khóa ràng buộc │ Đề yêu cầu "Least overhead"  │ Tập thói quen gạch chân từ khóa:   │
│    (Missed Constraints)      │ nhưng chọn tự dựng EC2;      │ "least operational overhead",      │
│                              │ hoặc yêu cầu "lowest cost"...│ "near real-time", "no downtime"... │
├──────────────────────────────┼──────────────────────────────┼────────────────────────────────────┤
│ 4. Tư duy On-Premise         │ Cố gắng cấu hình Cluster     │ Luôn ưu tiên Serverless và Fully   │
│    (The Curse of Experience) │ phức tạp bằng bash script    │ Managed Services của AWS khi đề bài│
│                              │ thay vì dùng dịch vụ có sẵn. │ không yêu cầu tùy biến OS.         │
└──────────────────────────────┴──────────────────────────────┴────────────────────────────────────┘
```
