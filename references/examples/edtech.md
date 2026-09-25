# Ví dụ User Story + AC mẫu - EdTech / Đào tạo trực tuyến

> 7 ví dụ cho sản phẩm giáo dục trực tuyến: thanh toán khóa học, mentor,
> theo dõi tiến độ, chứng chỉ, live class, doanh nghiệp đối tác, diễn đàn.
>
> Nội dung gốc: **Phúc NT** · BA Zone · Digital School. Bản này đã chỉnh lại
> cho khớp `checklists/quality-checklist.md` (When 1 hành động, 1 AC = 1
> scenario, có INVEST/Notes/Readiness) và generic hóa tên thương hiệu
> (Platform X, Ví X...).
>
> Số liệu (giá, thời gian, ngưỡng điểm) là minh họa - thay bằng rule thật của team.

---

## Ví dụ 1: Thanh toán khóa học

### US-ENROLL-001: Thanh toán khóa học bằng ví điện tử

**As a** học viên đã có tài khoản Platform X và đã liên kết Ví X
**I want to** thanh toán khóa học "Business Analysis Fundamentals" bằng ví
**So that** được truy cập nội dung học ngay mà không chờ xác nhận thủ công

*Epic: Enrollment · Priority: Must*

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Liên kết ví là story riêng, đã xong |
| Negotiable | ✅ | Chưa cố định cổng thanh toán |
| Valuable | ✅ | Học viên học ngay, nền tảng ghi nhận doanh thu |
| Estimable | ✅ | Luồng tương tự story thanh toán trước |
| Small | ✅ | ~3 ngày dev |
| Testable | ✅ | Số tiền, trạng thái đo được |

**AC1: Thanh toán thành công - happy path**
- **Given** khóa "BA Fundamentals" giá 1.500.000 VND còn slot
- **And** số dư Ví X của học viên là 2.000.000 VND
- **When** học viên xác nhận thanh toán bằng Ví X
- **Then** ví bị trừ 1.500.000 VND trong vòng 5 giây
- **And** học viên được enroll với trạng thái "Enrolled"
- **And** khóa học xuất hiện trong mục "Khóa học của tôi"
- **And** học viên nhận email xác nhận kèm link truy cập

**AC2: Số dư ví không đủ**
- **Given** số dư Ví X là 800.000 VND, giá khóa học 1.500.000 VND
- **When** học viên xác nhận thanh toán bằng Ví X
- **Then** hệ thống hiển thị "Số dư không đủ. Bạn cần thêm 700.000 VND"
- **And** đưa ra 2 lựa chọn "Nạp tiền vào ví" và "Chọn phương thức khác"
- **And** không trừ tiền, không tạo enrollment

**AC3: Mất kết nối khi giao dịch đang xử lý**
- **Given** học viên đã xác nhận thanh toán và giao dịch đang ở trạng thái "Đang xử lý"
- **When** kết nối mạng của học viên bị ngắt trước khi nhận kết quả
- **Then** sau khi có mạng lại, hệ thống hiển thị kết quả cuối cùng của giao dịch ("Thành công" hoặc "Thất bại")
- **And** ví chỉ bị trừ tối đa 1 lần cho giao dịch đó

**Notes**
- Dependencies: story liên kết Ví X
- Assumptions: Ví X trả trạng thái giao dịch khi được hỏi lại
- Open Questions: Hoàn tiền nếu khóa hết slot sau khi đã trừ tiền?

**Readiness (DoR):** ⚠️ Cần PO trả lời Open Question về hoàn tiền

---

## Ví dụ 2: Đặt lịch 1-on-1 với Mentor

### US-MENTOR-BOOK-001: Đặt lịch tư vấn 1-on-1 với mentor

**As a** học viên đang theo học chương trình BA Advanced, còn quota tư vấn trong gói tháng
**I want to** đặt lịch tư vấn 1-on-1 với mentor có chuyên môn phù hợp
**So that** nhận được hướng dẫn cụ thể cho bài tập thực hành mà không phải chờ buổi học chung

*Epic: Mentoring · Priority: Should*

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Lịch trống của mentor đã có (story riêng) |
| Negotiable | ✅ | Chưa chốt công cụ họp trực tuyến |
| Valuable | ✅ | Tăng tỷ lệ hoàn thành bài tập |
| Estimable | ✅ | |
| Small | ✅ | ~3 ngày dev |
| Testable | ✅ | |

**AC1: Đặt lịch thành công**
- **Given** mentor Trần B có slot trống 20:00 ngày 15/06/2026
- **And** học viên còn 1 session trong quota tháng
- **When** học viên xác nhận đặt slot đó kèm chủ đề cần tư vấn
- **Then** hệ thống tạo booking với mã dạng MNT20260615001
- **And** gửi link họp trực tuyến cho học viên và mentor
- **And** quota tháng của học viên còn 0 session
- **And** lên lịch nhắc qua email và app notification lúc 19:00 ngày 15/06/2026

**AC2: Slot vừa bị học viên khác đặt trước**
- **Given** học viên đang xem slot 20:00 ngày 15/06 ở trạng thái "Trống"
- **And** học viên khác đã đặt slot này 3 giây trước
- **When** học viên xác nhận đặt slot đó
- **Then** hệ thống hiển thị "Slot này vừa được đặt. Vui lòng chọn slot khác"
- **And** danh sách slot trống được tải lại, không còn slot 20:00 ngày 15/06
- **And** không trừ quota, không tạo booking

**AC3: Mentor hủy buổi tư vấn**
- **Given** học viên có booking thành công với mentor vào 20:00 ngày 15/06
- **When** mentor hủy buổi tư vấn lúc 10:00 ngày 15/06 (trước 10 tiếng)
- **Then** quota tháng của học viên được hoàn lại 1 session
- **And** học viên nhận thông báo kèm lý do hủy và 3 slot thay thế trong 7 ngày tới
- **And** lượt hủy được ghi vào hồ sơ mentor

**Notes**
- Open Questions: Mentor hủy dưới 6 tiếng trước giờ hẹn thì xử lý thế nào?

**Readiness (DoR):** ✅ Sẵn sàng

---

## Ví dụ 3: Theo dõi tiến độ học

### US-PROGRESS-001: Xem dashboard tiến độ học cá nhân

**As a** học viên đang học song song nhiều khóa trên Platform X
**I want to** xem tiến độ học của từng khóa trên một màn hình
**So that** biết mình đang ở đâu trong lộ trình và ưu tiên phần chưa hoàn thành

*Epic: Learning Experience · Priority: Should*

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | |
| Negotiable | ✅ | Chưa chốt cách trình bày biểu đồ |
| Valuable | ✅ | Giảm bỏ học giữa chừng |
| Estimable | ✅ | |
| Small | ✅ | |
| Testable | ✅ | % và số bài đo được |

**AC1: Hiển thị tiến độ đúng**
- **Given** học viên đang học "BA Fundamentals" (12/20 bài) và "Agile for BA" (2/10 bài)
- **When** học viên mở mục "Tiến độ học của tôi"
- **Then** hệ thống hiển thị "BA Fundamentals: 60% - còn 8 bài" và "Agile for BA: 20% - còn 8 bài"
- **And** mỗi khóa có tên bài học tiếp theo cần làm

**AC2: Khóa học chưa bắt đầu**
- **Given** học viên đã mua khóa "SQL for BA" nhưng chưa học bài nào
- **When** học viên mở mục "Tiến độ học của tôi"
- **Then** khóa "SQL for BA" hiển thị "0% hoàn thành" kèm nút "Bắt đầu học"

**AC3: Đồng bộ tiến độ học offline**
- **Given** học viên đã hoàn thành 3 bài khi không có mạng, tiến độ đã lưu trên server là 50%
- **When** thiết bị của học viên kết nối lại internet
- **Then** trong vòng 60 giây tiến độ được cập nhật lên 65%
- **And** hiển thị thông báo "Tiến độ đã được cập nhật"

**Notes**
- Assumptions: App lưu được tiến độ offline trên thiết bị

**Readiness (DoR):** ✅ Sẵn sàng

---

## Ví dụ 4: Cấp chứng chỉ hoàn thành

### US-CERT-001: Nhận chứng chỉ hoàn thành khóa học

**As a** học viên đã hoàn thành 100% bài học của một khóa
**I want to** nhận chứng chỉ có tên tôi, tên khóa và mã xác thực
**So that** có bằng chứng để đưa vào hồ sơ nghề nghiệp và hồ sơ xin việc

*Epic: Certification · Priority: Must*

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | |
| Negotiable | ✅ | Mẫu chứng chỉ do team design quyết |
| Valuable | ✅ | Lý do chính học viên mua khóa |
| Estimable | ✅ | |
| Small | ✅ | |
| Testable | ✅ | |

**AC1: Cấp chứng chỉ khi đủ điều kiện**
- **Given** học viên hoàn thành 20/20 bài khóa "BA Fundamentals"
- **And** điểm bài kiểm tra cuối khóa là 70%
- **When** hệ thống ghi nhận kết quả bài kiểm tra
- **Then** chứng chỉ PDF được tạo với mã xác thực dạng CERT-BAF-2026-00123
- **And** học viên nhận email kèm chứng chỉ trong vòng 5 phút
- **And** chứng chỉ xuất hiện trong mục "Thành tích của tôi"

**AC2: Điểm chưa đạt**
- **Given** học viên hoàn thành 20/20 bài, điểm kiểm tra cuối khóa là 55%
- **When** hệ thống ghi nhận kết quả bài kiểm tra
- **Then** không tạo chứng chỉ
- **And** hiển thị "Bạn cần đạt tối thiểu 70% để nhận chứng chỉ. Điểm hiện tại: 55%"
- **And** nút "Làm lại bài kiểm tra" chỉ bật sau 24 giờ

**AC3: Xác thực mã chứng chỉ hợp lệ**
- **Given** chứng chỉ CERT-BAF-2026-00123 đã được cấp cho học viên Nguyễn Văn A
- **When** nhà tuyển dụng tra mã này trên trang xác thực chứng chỉ của Platform X
- **Then** hệ thống hiển thị tên học viên, tên khóa, ngày cấp và điểm

**AC4: Xác thực mã không tồn tại**
- **Given** mã CERT-BAF-2026-99999 không có trong hệ thống
- **When** nhà tuyển dụng tra mã này trên trang xác thực
- **Then** hệ thống hiển thị "Mã chứng chỉ không hợp lệ"

**Readiness (DoR):** ✅ Sẵn sàng

---

## Ví dụ 5: Live Class & Q&A

### US-LIVE-001: Gửi câu hỏi trong buổi live class

**As a** học viên đang tham dự một buổi live class
**I want to** gửi câu hỏi bằng văn bản trong khi giảng viên đang trình bày
**So that** không làm gián đoạn bài giảng nhưng vẫn được giải đáp ở phần Q&A cuối buổi

*Epic: Live Learning · Priority: Should*

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | |
| Negotiable | ✅ | |
| Valuable | ✅ | |
| Estimable | ⚠️ | Cần thống nhất quy tắc nhận diện câu hỏi trùng (AC3) |
| Small | ✅ | |
| Testable | ✅ | |

**Background**
```gherkin
Background:
  Given buổi live class "Agile for BA - Buổi 3" đang ở trạng thái "Live"
  And học viên đã tham gia buổi học
```

**AC1: Gửi câu hỏi thành công**
- **When** học viên gửi câu hỏi dài 120 ký tự
- **Then** câu hỏi xuất hiện trong hàng đợi Q&A của giảng viên
- **And** học viên thấy trạng thái "Câu hỏi đã gửi - Chờ giải đáp"
- **And** học viên khác thấy nút "Tôi cũng có câu hỏi này (+1)" trên câu hỏi đó

**AC2: Câu hỏi vượt giới hạn ký tự**
- **When** học viên gửi câu hỏi dài 501 ký tự
- **Then** hệ thống không gửi và hiển thị "Câu hỏi tối đa 500 ký tự"

**AC3: Câu hỏi trùng với câu đã có**
- **Given** hàng đợi đã có câu hỏi được đánh giá là trùng nội dung theo quy tắc so khớp đã thống nhất
- **When** học viên gửi câu hỏi đó
- **Then** hệ thống hiển thị "Câu hỏi tương tự đã được gửi. Bạn có muốn +1 cho câu hỏi này?"
- **And** chưa tạo câu hỏi mới trong hàng đợi

**AC4: Buổi học đã kết thúc**
- **Given** buổi live class chuyển sang trạng thái "Ended"
- **When** học viên gửi câu hỏi
- **Then** hệ thống hiển thị "Buổi học đã kết thúc. Xem lại recording hoặc đặt câu hỏi trong diễn đàn khóa học"
- **And** hiển thị link tới diễn đàn của khóa học

*(AC4 ghi đè Given của Background về trạng thái buổi học.)*

**Notes**
- Open Questions: Quy tắc thế nào là "trùng nội dung"? (cần spike nhỏ hoặc PO quyết)

**Readiness (DoR):** ⚠️ Cần chốt quy tắc câu hỏi trùng trước khi estimate AC3

---

## Ví dụ 6: Doanh nghiệp đối tác cấp phát khóa học

### US-B2B-ENROLL-001: Cấp phát khóa học cho nhân viên bằng file Excel

**As a** HR Manager của doanh nghiệp đối tác đã ký hợp đồng license
**I want to** cấp phát khóa học cho danh sách nhân viên bằng cách upload file Excel
**So that** không phải enroll từng người khi số lượng nhân viên lớn (> 20 người)

*Epic: B2B · Priority: Must*

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Template Excel đã chốt |
| Negotiable | ✅ | |
| Valuable | ✅ | Giảm thao tác thủ công cho khách B2B |
| Estimable | ✅ | |
| Small | ⚠️ | ~5 ngày, sát ngưỡng - cân nhắc tách AC2 thành story riêng |
| Testable | ✅ | |

**AC1: Upload file hợp lệ**
- **Given** tài khoản doanh nghiệp còn 50 license chưa kích hoạt
- **And** file Excel đúng template có 50 nhân viên hợp lệ
- **When** HR Manager xác nhận cấp phát
- **Then** 50 nhân viên được enroll trong vòng 2 phút
- **And** mỗi nhân viên nhận email chào mừng kèm thông tin đăng nhập
- **And** hiển thị báo cáo "50/50 thành công"
- **And** số license chưa kích hoạt còn 0

**AC2: File có dòng không hợp lệ**
- **Given** file Excel có 50 dòng, 3 dòng sai định dạng email và 2 dòng trùng email
- **When** HR Manager upload file
- **Then** màn hình preview đánh dấu 5 dòng lỗi kèm mô tả lỗi của từng dòng
- **And** HR Manager có 2 lựa chọn "Cấp phát 45 dòng hợp lệ" hoặc "Tải file khác"

**AC3: Không đủ license**
- **Given** tài khoản doanh nghiệp còn 30 license, file upload có 50 nhân viên hợp lệ
- **When** HR Manager xác nhận cấp phát
- **Then** hệ thống hiển thị "Bạn cần thêm 20 license. Liên hệ Platform X để mua thêm"
- **And** không enroll nhân viên nào, số license vẫn là 30
- **And** hiển thị link tới trang nâng cấp gói doanh nghiệp

**AC4: Người không phải HR Manager**
- **Given** người dùng có vai trò "Nhân viên" trong tài khoản doanh nghiệp
- **When** người dùng mở chức năng cấp phát khóa học
- **Then** hệ thống hiển thị "Bạn không có quyền thực hiện chức năng này"

**Readiness (DoR):** ✅ Sẵn sàng (theo dõi Small khi estimate)

---

## Ví dụ 7: Báo cáo vi phạm trong diễn đàn

### US-FORUM-REPORT-001: Báo cáo nội dung vi phạm trong diễn đàn

**As a** thành viên đang tham gia thảo luận trong diễn đàn khóa học
**I want to** báo cáo bài viết hoặc bình luận vi phạm quy tắc cộng đồng
**So that** nội dung vi phạm được xử lý sớm, giữ chất lượng thảo luận

*Epic: Community · Priority: Should*

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | |
| Negotiable | ✅ | Danh sách lý do báo cáo có thể đổi |
| Valuable | ✅ | |
| Estimable | ✅ | |
| Small | ⚠️ | 5 AC, gồm cả luồng Moderator - có thể tách AC3-AC5 thành story "Xử lý báo cáo" |
| Testable | ✅ | |

**AC1: Gửi báo cáo thành công**
- **Given** thành viên đang xem một bài viết chứa spam
- **When** thành viên gửi báo cáo với lý do "Spam"
- **Then** hệ thống tạo ticket với mã dạng RPT20260506001
- **And** hiển thị "Báo cáo của bạn đã được ghi nhận"
- **And** ticket được giao cho Moderator đang trực ca

**AC2: Tạm ẩn bài viết bị báo cáo nhiều**
- **Given** bài viết đã nhận 4 báo cáo trong 1 giờ qua
- **When** thành viên thứ 5 gửi báo cáo cho bài viết đó
- **Then** bài viết chuyển sang trạng thái "Tạm ẩn - chờ duyệt"

**AC3: Moderator xác nhận vi phạm**
- **Given** ticket báo cáo đang ở trạng thái "Đang xem xét"
- **When** Moderator kết luận "Vi phạm"
- **Then** người báo cáo nhận notification "Cảm ơn bạn đã góp phần xây dựng cộng đồng" kèm lý do quyết định

**AC4: Moderator kết luận không vi phạm**
- **Given** ticket báo cáo đang ở trạng thái "Đang xem xét"
- **When** Moderator kết luận "Không vi phạm"
- **Then** người báo cáo nhận notification kết quả kèm lý do quyết định
- **And** bài viết (nếu đang tạm ẩn) được hiển thị lại

**AC5: Quá hạn xử lý 48 giờ**
- **Given** ticket báo cáo đã tạo 48 giờ và chưa được xử lý
- **When** hệ thống chạy kiểm tra SLA định kỳ
- **Then** ticket được chuyển lên Community Manager
- **And** người báo cáo nhận thông báo xin lỗi, cam kết xử lý trong 12 giờ tiếp theo

**Readiness (DoR):** ⚠️ Cân nhắc split trước khi đưa vào sprint

---

## Patterns rút ra

1. **Persona có trạng thái**: "học viên còn quota tư vấn", "HR Manager đã ký hợp đồng license"
2. **Then có số liệu và message cụ thể**: 1.500.000 VND, "50/50 thành công", thông báo trong ngoặc kép
3. **Mỗi nhánh 1 AC**: mã hợp lệ và mã không hợp lệ là 2 AC riêng (Ví dụ 4)
4. **Background** khi mọi AC chung bối cảnh (Ví dụ 5)
5. **Phân quyền âm** là 1 AC riêng (Ví dụ 6 - AC4)
6. **Edge case đặc thù EdTech**: tranh slot mentor, hết license, học offline, câu hỏi trùng
7. **INVEST ⚠️ không phải lỗi**: ghi rõ lý do và hướng xử lý (Ví dụ 6, 7)

---
*Nội dung gốc: Phúc NT · BA Zone · Digital School - chỉnh sửa để khớp checklist của skill*
