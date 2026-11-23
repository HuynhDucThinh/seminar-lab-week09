# Báo cáo lab Nhóm 09

**Họ tên:** Huỳnh Đức Thịnh · **MSSV:** 23120199 · **JDK đã dùng** (kết quả `java -version`): `openjdk version "21.0.13.1" 2026-11-17 LTS`

## Bài 1: Giá trị biên

| Ngày trễ | Hạng | Số tiền bạn điền | Cách tính |
|---:|---|---:|---|
| 24 | NORMAL | 48000 | 24 × 2.000 = 48.000đ (ngay dưới mức trần 50.000đ) |
| 25 | NORMAL | 50000 | 25 × 2.000 = 50.000đ (đúng tại mức trần 50.000đ) |
| 26 | NORMAL | 50000 | 26 × 2.000 = 52.000đ → áp mức trần tối đa 50.000đ |
| 2 | VIP | 0 | Nằm trong 2 ngày ân hạn đầu nên không tính phí (0đ) |
| 3 | VIP | 1000 | (3 − 2) × 1.000 = 1.000đ (ngày đầu tiên sau thời gian ân hạn) |
| 51 | VIP | 49000 | (51 − 2) × 1.000 = 49.000đ (ngay dưới mức trần 50.000đ) |
| 52 | VIP | 50000 | (52 − 2) × 1.000 = 50.000đ (đúng tại mức trần 50.000đ) |
| 53 | VIP | 50000 | (53 − 2) × 1.000 = 51.000đ → áp mức trần tối đa 50.000đ |

**Vì sao chọn các ngày này mà không chọn ngày 10?**
Các ngày `24, 25, 26` (NORMAL) và `2, 3, 51, 52, 53` (VIP) được chọn theo kỹ thuật **Phân tích giá trị biên (Boundary Value Analysis – BVA)**. Lỗi logic thường xuất hiện ngay tại các điểm chuyển trạng thái (nhầm `<` với `<=`, tính lệch 1 ngày ân hạn hoặc mức trần): biên ân hạn 2–3 ngày của hạng VIP, và biên chạm mức trần 50.000đ ở 25 ngày (NORMAL) / 52 ngày (VIP). Trong khi đó, ngày 10 nằm ở giữa miền tương đương nên không nhạy cảm để phát hiện các lỗi lệch 1 đơn vị (off-by-one) tại biên.

## Bài 2a: Coverage

| | Branch coverage của `LoanService` |
|---|---|
| Trước khi viết test | 66% (ratio `0.66`, 4/6 nhánh) |
| Sau khi viết 2 test | 100% (ratio `1.00`, 6/6 nhánh) |

**Test `daDatHanMuc` của bạn dùng `countActiveByMemberId` trả về giá trị nào? Vì sao chọn giá trị đó?**
Ban đầu ở bước 2a, mình thử giá trị `4L` để cả hai nhánh của câu lệnh `if (loans.countActiveByMemberId(memberId) > 3)` đều được phủ và đạt 100% branch coverage (vì test sẵn có đã phủ nhánh `2L`). Tuy nhiên, sau khi chạy PIT và đối chiếu lại đặc tả (*"đang mượn 3 cuốn thì không được mượn thêm"*), mình sửa về đúng giá trị biên **`3L`** để kiểm tra chính xác giới hạn nghiệp vụ.

## Bài 2b: Mutation testing

**Lần chạy PIT đầu tiên:** có bao nhiêu mutant sống sót (SURVIVED)? Nằm ở dòng nào, là loại mutant gì?
Có **1 mutant sống sót (`SURVIVED`)** trên tổng số 9 mutant (8/9 bị diệt – 89%). Mutant nằm ở **dòng 28** của `LoanService.java`, thuộc loại **`ConditionalsBoundaryMutator`** (`changed conditional boundary`, đột biến toán tử `>` thành `>=` trong điều kiện `loans.countActiveByMemberId(memberId) > 3`).

**Lỗi cài sẵn nằm ở đâu? Bạn phát hiện ra nó bằng cách nào?**
Lỗi cài sẵn nằm ở **dòng 28 của `src/main/java/vn/edu/seminar/loan/LoanService.java`**:
```java
if (loans.countActiveByMemberId(memberId) > 3)
```
Phát hiện nhờ PIT báo mutant `ConditionalsBoundaryMutator` sống sót tại dòng này: test với `2L` (cho mượn) và `4L` (từ chối) không phân biệt được `> 3` với `>= 3`. Khi đối chiếu đặc tả (*"Mỗi thành viên được mượn tối đa 3 cuốn: đang mượn 3 cuốn thì không được mượn thêm"*), điều kiện `> 3` bị sai off-by-one vì vẫn cho phép thành viên đang mượn 3 cuốn mượn thêm cuốn thứ 4.

**Bạn đã sửa test và code như thế nào?**
- Trong `src/test/java/vn/edu/seminar/loan/LoanServiceTest.java`: sửa test `daDatHanMuc()` để `loans.countActiveByMemberId(1)` trả về đúng giá trị biên `3L` thay vì `4L`.
- Trong `src/main/java/vn/edu/seminar/loan/LoanService.java` (dòng 28): sửa `if (loans.countActiveByMemberId(memberId) > 3)` thành `if (loans.countActiveByMemberId(memberId) >= 3)`.

**Kết quả PIT sau khi sửa** (Killed bao nhiêu / tổng bao nhiêu):
**Killed 9 / 9 (100%)**, 0 mutant `SURVIVED`, Test strength **100%**, Line Coverage **17/17 (100%)**.

## Câu hỏi tổng kết
**Coverage 100% có nghĩa là code không còn lỗi không? Hãy giải thích dựa trên những gì bạn thấy trong lab.**
**Không.** Coverage 100% chỉ cho biết mọi dòng lệnh và nhánh điều kiện đã được thực thi qua ít nhất một lần khi chạy test, chứ không đảm bảo test đã kiểm tra đúng các giá trị biên hay đúng đặc tả nghiệp vụ. Trong lab này, dù `LoanService` đạt 100% branch coverage với `countActiveByMemberId` là `2L` và `4L`, lỗi off-by-one `> 3` (thay vì `>= 3`) vẫn hoàn toàn lọt qua JaCoCo và chỉ bị phát hiện khi chạy mutation testing (PIT) kết hợp đối chiếu giá trị biên `3L` theo đặc tả.
