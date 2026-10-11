# NHẬT KÝ LỖI SAI (ERROR LOG & WEAK SPOTS)

> **Mục đích**: Ghi lại các câu làm sai hoặc còn phân vân khi luyện đề. Mỗi cuối tuần (Ngày 7, 14, 21, 28), đọc lại bảng này và làm lại các câu đã sai để triệt tiêu lỗ hổng kiến thức.

---

### BẢNG THEO DÕI CÂU SAI

| Ngày | Câu số / Nguồn đề | Chủ đề / Service | Lý do sai (Thiếu kiến thức / Nhầm service / Đọc thiếu keyword / Bị bẫy) | Từ khóa chốt của đề | Nguyên tắc / Kiến thức cần nhớ để không sai lại | Đã làm lại đúng chưa? |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| *Ví dụ* | *Câu 2 - Ngày 1* | *NAT Gateway HA* | *Nhầm tính chất managed của NAT Gateway* | *"Highest availability", "2 AZs"* | *NAT Gateway chỉ HA trong 1 AZ; muốn Multi-AZ HA phải tạo 1 NAT GW cho mỗi AZ.* | [x] |
| | | | | | | [ ] |
| | | | | | | [ ] |
| | | | | | | [ ] |
| | | | | | | [ ] |
| | | | | | | [ ] |
| | | | | | | [ ] |
| | | | | | | [ ] |
| | | | | | | [ ] |

---

### 4 DẠNG LỖI KINH ĐIỂN CẦN NHẬN DIỆN:
1. **Thiếu kiến thức ngầm (Gaps in AWS internals)**:
   - Ví dụ: Không biết NAT Gateway bị giới hạn trong 1 AZ, không biết AWS giữ 5 IP trong mỗi subnet, không biết Gateway Endpoint là miễn phí.
2. **Nhầm cặp dịch vụ tương đồng (Confusing similar services)**:
   - SQS vs SNS vs Kinesis; Global Accelerator vs CloudFront; EFS vs EBS Multi-Attach; Secrets Manager vs SSM Parameter Store.
3. **Bỏ sót ràng buộc bắt buộc (Overlooking constraints)**:
   - Đề bài yêu cầu *"least operational overhead"* nhưng chọn phương án rẻ hơn mà phải tự code / quản trị EC2.
   - Đề bài yêu cầu *"most cost-effective"* nhưng chọn giải pháp xịn nhất, đắt tiền nhất.
   - Đề bài yêu cầu *"near real-time"* nhưng chọn giải pháp chạy batch job 1 giờ/lần.
4. **Tư duy theo kinh nghiệm On-Premise (Curse of Experience)**:
   - Cố gắng cấu hình Active-Passive cluster bằng HAProxy / Corosync thay vì dùng Route 53 Failover + ALB.
