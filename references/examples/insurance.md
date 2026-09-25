# Ví dụ User Story + AC mẫu - Bảo hiểm

> 5 ví dụ cho bảo hiểm bán online (sức khỏe, xe cơ giới): báo giá, mua hợp đồng, bồi thường, đại lý, tái tục.
>
> ⚠️ **Lưu ý:** các con số (phí, tỷ lệ phí, thời gian chờ, thời gian gia hạn, hạn mức, độ tuổi) chỉ để **minh họa**,
> không phải quy định pháp lý hay số liệu định phí - team thay bằng quy tắc sản phẩm thực tế đã phê duyệt.
> Tên doanh nghiệp được generic hóa: Công ty bảo hiểm X.

---

## Ví dụ 1: Báo giá bảo hiểm sức khỏe theo độ tuổi

### US-QUOTE-001: Nhận báo giá phí bảo hiểm sức khỏe theo ngày sinh người được bảo hiểm

**As a** khách hàng cá nhân chưa có tài khoản, đang tìm hiểu gói "Sức khỏe Cơ bản" trên website của Công ty bảo hiểm X
**I want to** nhận mức phí năm của gói "Sức khỏe Cơ bản" sau khi nhập ngày sinh người được bảo hiểm và ngày bắt đầu hiệu lực
**So that** tôi so sánh được chi phí với ngân sách gia đình trước khi quyết định mua, không phải chờ tư vấn viên gọi lại

**Metadata:** Epic: EP-HEALTH-SALES · Priority: Must

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Không cần tài khoản, không phụ thuộc story mua hợp đồng |
| Negotiable | ✅ | Biểu phí và số nhóm tuổi do phòng sản phẩm quyết định, có thể đổi |
| Valuable | ✅ | Khách tự xem phí; công ty thu được lead có nhu cầu thật |
| Estimable | ✅ | Biểu phí 4 nhóm tuổi đã có bản chính thức |
| Small | ✅ | ~3 ngày dev |
| Testable | ✅ | Có bảng biên tuổi và mức phí cụ thể |

**AC1: Tính phí theo nhóm tuổi - happy path + biên tuổi (Scenario Outline)**

```gherkin
Scenario Outline: Tính phí năm theo tuổi tại ngày bắt đầu hiệu lực
  Given gói "Sức khỏe Cơ bản" áp dụng biểu phí 2027 với 4 nhóm tuổi: 1-17, 18-40, 41-60, 61-65
  When khách hàng yêu cầu báo giá cho người được bảo hiểm <tuoi> tuổi tại ngày bắt đầu hiệu lực
  Then kết quả báo giá là <ket_qua>

  Examples:
    | tuoi | ket_qua                                                                     |
    | 0    | từ chối: "Gói Sức khỏe Cơ bản nhận người được bảo hiểm từ 1 đến 65 tuổi"    |
    | 1    | phí năm 3.200.000 VND                                                       |
    | 17   | phí năm 3.200.000 VND                                                       |
    | 18   | phí năm 2.400.000 VND                                                       |
    | 40   | phí năm 2.400.000 VND                                                       |
    | 41   | phí năm 3.600.000 VND                                                       |
    | 60   | phí năm 3.600.000 VND                                                       |
    | 61   | phí năm 5.800.000 VND                                                       |
    | 65   | phí năm 5.800.000 VND                                                       |
    | 66   | từ chối: "Gói Sức khỏe Cơ bản nhận người được bảo hiểm từ 1 đến 65 tuổi"    |
```

**AC2: Sinh nhật rơi vào giữa ngày yêu cầu và ngày bắt đầu hiệu lực - edge case**
- **Given** ngày hiện tại là 01/03/2027
- **And** khách hàng đã nhập ngày sinh người được bảo hiểm 10/04/1986 (40 tuổi tại ngày hiện tại)
- **And** khách hàng đã chọn ngày bắt đầu hiệu lực 15/04/2027 (41 tuổi tại ngày này)
- **When** khách hàng yêu cầu báo giá
- **Then** phí năm là 3.600.000 VND theo nhóm tuổi 41-60
- **And** báo giá ghi "Tuổi tính phí: 41 (tại ngày bắt đầu hiệu lực 15/04/2027)"

**AC3: Ngày bắt đầu hiệu lực ngoài khoảng cho phép - negative path**
- **Given** ngày hiện tại là 01/03/2027
- **And** khách hàng đã chọn ngày bắt đầu hiệu lực 01/07/2027 (122 ngày sau ngày hiện tại)
- **When** khách hàng yêu cầu báo giá
- **Then** hệ thống từ chối với thông báo "Ngày bắt đầu hiệu lực phải từ 02/03/2027 đến 30/05/2027"
- **And** không có báo giá nào được tạo

**Notes:**
- Dependencies: Biểu phí 2027 gói "Sức khỏe Cơ bản" được phòng sản phẩm phê duyệt
- Assumptions: Tuổi tính theo số năm tròn tại ngày bắt đầu hiệu lực; khoảng chọn ngày bắt đầu là 1-90 ngày từ ngày hiện tại
- Open Questions: Báo giá có cần lưu lại để khách mở lại sau không? (nếu có → story riêng)

**Readiness (DoR):** ✅ Sẵn sàng

---

## Ví dụ 2: Mua bảo hiểm vật chất xe ô tô online (minh họa split story)

**❌ Story gốc (chưa đạt INVEST - Small, Estimable):**

> **Tiêu đề:** "Mua bảo hiểm vật chất xe ô tô và thanh toán online"
> **As a** chủ xe ô tô muốn mua bảo hiểm online
> **I want to** khai báo xe, nhận báo giá, thanh toán bằng thẻ, ví điện tử hoặc QR, rồi nhận giấy chứng nhận điện tử
> **So that** mua được bảo hiểm xe mà không cần gặp nhân viên
>
> Dev ước lượng ~12 ngày, bản nháp có 15 AC, tiêu đề chứa "và".

**Đề xuất split: Story gốc → Epic EP-MOTOR-01**

- **Chẩn đoán:** Compound - gộp 3 bước nghiệp vụ đã hiểu rõ (báo giá → thanh toán → cấp giấy chứng nhận) và 3 phương thức thanh toán; không có ẩn số kỹ thuật nên không cần Spike
- **Pattern áp dụng:** Bước workflow + Cách thay thế (mỗi phương thức thanh toán 1 story)

| Story con | Giá trị riêng | Ưu tiên gợi ý |
|-----------|---------------|---------------|
| US-MOTOR-001: Nhận báo giá bảo hiểm vật chất xe ô tô từ thông tin xe | Chủ xe biết phí trước khi mua; sales có lead kèm thông tin xe | Must |
| US-MOTOR-002: Thanh toán phí bảo hiểm xe bằng thẻ ATM nội địa | Thu phí online qua phương thức phổ biến nhất | Must |
| US-MOTOR-003: Nhận giấy chứng nhận bảo hiểm điện tử qua email | Chủ xe có chứng từ xuất trình khi cần | Must |
| US-MOTOR-004: Thanh toán phí bảo hiểm xe bằng ví điện tử | Tăng tỷ lệ hoàn tất với khách trẻ | Should |
| US-MOTOR-005: Thanh toán phí bảo hiểm xe bằng mã QR ngân hàng | Thêm lựa chọn khi khách không có thẻ | Could |

Story nào giao được trước mà vẫn có giá trị: **US-MOTOR-001** (báo giá tự đứng được, sales gọi chốt đơn offline trong lúc chờ các story thanh toán).

### US-MOTOR-001: Nhận báo giá bảo hiểm vật chất xe ô tô từ thông tin xe

**As a** chủ xe ô tô cá nhân đã có tài khoản trên ứng dụng của Công ty bảo hiểm X
**I want to** nhận báo giá phí bảo hiểm vật chất 1 năm sau khi khai báo biển số, năm sản xuất và giá trị xe
**So that** tôi biết số tiền phải trả trước khi quyết định mua, không phải hẹn nhân viên đến giám định với xe còn mới

**Metadata:** Epic: EP-MOTOR-01 · Priority: Must

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Không cần story thanh toán để có giá trị |
| Negotiable | ✅ | Tỷ lệ phí, ngưỡng chênh lệch giá trị xe có thể điều chỉnh |
| Valuable | ✅ | Khách tự báo giá, giảm tải cho nhân viên kinh doanh |
| Estimable | ⚠️ | Cần xác nhận nguồn giá tham khảo theo dòng xe |
| Small | ✅ | ~3-4 ngày dev sau khi tách |
| Testable | ✅ | Có giá trị xe, tỷ lệ phí, message cụ thể |

**AC1: Báo giá thành công cho xe đến 3 năm sử dụng - happy path**
- **Given** ngày hiện tại là 31/03/2027
- **And** khách hàng đã khai báo xe 5 chỗ biển số 30A-123.45, năm sản xuất 2024, giá trị 800.000.000 VND
- **And** giá tham khảo của dòng xe là 820.000.000 VND, tỷ lệ phí cho xe đến 3 năm sử dụng là 1,5%/năm
- **When** khách hàng yêu cầu báo giá
- **Then** phí bảo hiểm vật chất 1 năm là 12.000.000 VND
- **And** báo giá BG-XCG-2027-000321 có trạng thái "Còn hiệu lực" đến ngày 15/04/2027

**AC2: Xe trên 10 năm sử dụng - edge case**
- **Given** khách hàng đã khai báo xe biển số 51G-678.90, năm sản xuất 2016 (11 năm sử dụng tại năm 2027)
- **When** khách hàng yêu cầu báo giá
- **Then** hệ thống không đưa ra mức phí
- **And** hiển thị thông báo "Xe trên 10 năm sử dụng cần giám định trực tiếp trước khi báo giá. Nhân viên sẽ liên hệ bạn trong 1 ngày làm việc"
- **And** yêu cầu giám định được tạo với trạng thái "Chờ liên hệ"

**AC3: Giá trị khai báo chênh lệch quá 10% so với giá tham khảo - negative path**
- **Given** giá tham khảo của dòng xe là 820.000.000 VND
- **And** khách hàng đã khai báo giá trị xe 950.000.000 VND
- **When** khách hàng yêu cầu báo giá
- **Then** hệ thống từ chối với thông báo "Giá trị xe cần nằm trong khoảng 738.000.000 - 902.000.000 VND (±10% giá tham khảo)"
- **And** không có báo giá nào được tạo

**Notes:**
- Dependencies: Bảng giá tham khảo theo dòng xe, năm sản xuất (phòng sản phẩm cung cấp)
- Assumptions: Số năm sử dụng = năm hiện tại - năm sản xuất; báo giá có hiệu lực 15 ngày
- Open Questions: Xe kinh doanh vận tải có dùng chung tỷ lệ phí không, hay tách story riêng?

**Readiness (DoR):** ⚠️ Còn thiếu: nguồn giá tham khảo theo dòng xe chưa được chốt

---

## Ví dụ 3: Yêu cầu bồi thường có chứng từ

### US-CLAIM-001: Gửi yêu cầu bồi thường ngoại trú kèm chứng từ điện tử

**As a** khách hàng có hợp đồng bảo hiểm sức khỏe đang hiệu lực với quyền lợi điều trị ngoại trú, đã đăng nhập ứng dụng
**I want to** gửi yêu cầu bồi thường chi phí khám ngoại trú kèm ảnh chụp chứng từ
**So that** tôi được xét bồi thường mà không phải gửi hồ sơ giấy qua bưu điện hoặc đến văn phòng

**Metadata:** Epic: EP-CLAIM-ONLINE · Priority: Must

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Story dừng ở trạng thái "Đã tiếp nhận"; thẩm định là US-CLAIM-002 |
| Negotiable | ✅ | Danh sách chứng từ bắt buộc có thể đổi theo sản phẩm |
| Valuable | ✅ | Khách tiết kiệm thời gian; công ty giảm chi phí xử lý hồ sơ giấy |
| Estimable | ✅ | Quy tắc thời gian chờ đã có trong quy tắc sản phẩm |
| Small | ⚠️ | ~5 ngày; nếu thêm tự đọc số liệu hóa đơn thì tách story riêng |
| Testable | ✅ | Có ngày biên thời gian chờ, message cụ thể |

**Background** (áp dụng cho AC1-AC4):

```gherkin
Background:
  Given khách hàng Trần Thị B đã đăng nhập ứng dụng của Công ty bảo hiểm X
  And hợp đồng SK-2027-000123 ở trạng thái "Đang hiệu lực", ngày bắt đầu hiệu lực 01/03/2027
  And hợp đồng có quyền lợi điều trị ngoại trú
```

**AC1: Gửi hồ sơ đủ chứng từ - happy path**
- **Given** hồ sơ có ngày khám 15/05/2027, loại điều trị "Bệnh thông thường", số tiền yêu cầu 1.850.000 VND
- **And** hồ sơ đính kèm đủ 3 chứng từ bắt buộc: phiếu khám, đơn thuốc, hóa đơn tài chính
- **When** khách hàng gửi yêu cầu bồi thường
- **Then** hồ sơ BT-2027-004567 được tạo với trạng thái "Đã tiếp nhận"
- **And** khách hàng nhận thông báo "Hồ sơ BT-2027-004567 đã được tiếp nhận. Kết quả thẩm định dự kiến trong 5 ngày làm việc"

**AC2: Ngày khám so với thời gian chờ - edge case (Scenario Outline)**

```gherkin
Scenario Outline: Kiểm tra thời gian chờ theo loại điều trị
  Given hồ sơ đính kèm đủ 3 chứng từ bắt buộc
  And loại điều trị khai báo là "<loai_dieu_tri>" với ngày khám <ngay_kham>
  When khách hàng gửi yêu cầu bồi thường
  Then kết quả là <ket_qua>

  Examples:
    | loai_dieu_tri     | ngay_kham                 | ket_qua                                                                               |
    | Tai nạn           | 02/03/2027 (ngày thứ 2)   | tiếp nhận, trạng thái "Đã tiếp nhận"                                                  |
    | Bệnh thông thường | 30/03/2027 (ngày thứ 30)  | từ chối: "Ngày khám 30/03/2027 nằm trong thời gian chờ 30 ngày của bệnh thông thường" |
    | Bệnh thông thường | 31/03/2027 (ngày thứ 31)  | tiếp nhận, trạng thái "Đã tiếp nhận"                                                  |
    | Bệnh đặc biệt     | 27/08/2027 (ngày thứ 180) | từ chối: "Ngày khám 27/08/2027 nằm trong thời gian chờ 180 ngày của bệnh đặc biệt"    |
    | Bệnh đặc biệt     | 28/08/2027 (ngày thứ 181) | tiếp nhận, trạng thái "Đã tiếp nhận"                                                  |
```

**AC3: Thiếu chứng từ bắt buộc - negative path**
- **Given** hồ sơ có ngày khám 15/05/2027, đính kèm phiếu khám và đơn thuốc, chưa có hóa đơn tài chính
- **When** khách hàng gửi yêu cầu bồi thường
- **Then** hệ thống từ chối với thông báo "Thiếu chứng từ bắt buộc: Hóa đơn tài chính"
- **And** hồ sơ giữ trạng thái "Nháp" cùng 2 chứng từ đã tải lên

**AC4: Hóa đơn đã dùng cho hồ sơ khác - negative path**
- **Given** hóa đơn tài chính số 0001234 đã thuộc hồ sơ BT-2027-003210
- **When** khách hàng gửi yêu cầu bồi thường mới kèm hóa đơn số 0001234
- **Then** hệ thống từ chối với thông báo "Hóa đơn 0001234 đã được sử dụng trong hồ sơ BT-2027-003210"
- **And** không có hồ sơ mới nào được tạo

**Notes:**
- Dependencies: Quy tắc thời gian chờ theo loại điều trị của sản phẩm sức khỏe
- Assumptions: Ngày bắt đầu hiệu lực tính là ngày thứ 1; thời gian chờ không áp dụng cho tai nạn
- Open Questions: Hồ sơ trong thời gian chờ nên bị từ chối ngay khi gửi, hay vẫn tiếp nhận để thẩm định viên kết luận?

**Readiness (DoR):** ⚠️ Còn thiếu: PO chốt cách xử lý hồ sơ nằm trong thời gian chờ (Open Question)

---

## Ví dụ 4: Đại lý theo dõi hợp đồng sắp đến hạn tái tục

### US-AGENT-001: Xem danh sách hợp đồng sắp đến hạn tái tục của khách hàng do mình quản lý

**As a** đại lý bảo hiểm cá nhân của Công ty bảo hiểm X có mã đại lý đang hoạt động, quản lý khoảng 150 khách hàng
**I want to** xem danh sách hợp đồng của khách hàng mình sẽ hết hạn trong 30 ngày tới
**So that** tôi liên hệ nhắc tái tục trước ngày hết hạn, giảm số hợp đồng mất hiệu lực, giữ được hoa hồng tái tục

**Metadata:** Epic: EP-AGENT-PORTAL · Priority: Should

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Dùng dữ liệu hợp đồng và phân công đại lý đã có |
| Negotiable | ✅ | Khoảng 30 ngày, cột hiển thị có thể thảo luận |
| Valuable | ✅ | Đại lý chủ động chăm sóc; công ty tăng tỷ lệ tái tục |
| Estimable | ✅ | Quy tắc phân quyền theo danh mục đại lý đã có |
| Small | ✅ | ~3 ngày dev |
| Testable | ✅ | Có ngày biên 30/31, message phân quyền cụ thể |

**Background** (áp dụng cho AC1-AC4):

```gherkin
Background:
  Given đại lý Lê Văn C (mã DL-0456) đã đăng nhập cổng đại lý
  And mã đại lý DL-0456 ở trạng thái "Đang hoạt động"
  And ngày hiện tại là 01/06/2027
```

**AC1: Danh sách hợp đồng trong 30 ngày tới - happy path**
- **Given** DL-0456 quản lý 5 hợp đồng hết hạn lần lượt ngày 05/06, 20/06, 01/07, 02/07, 20/07/2027
- **When** đại lý xem danh sách "Sắp đến hạn tái tục"
- **Then** danh sách có đúng 3 hợp đồng hết hạn 05/06, 20/06, 01/07/2027, xếp theo ngày hết hạn gần nhất trước
- **And** mỗi hợp đồng hiển thị tên khách hàng, số hợp đồng, sản phẩm, ngày hết hạn, phí tái tục dự kiến
- **And** 2 hợp đồng hết hạn 02/07/2027 (ngày thứ 31) và 20/07/2027 không có trong danh sách

**AC2: Không có hợp đồng nào đến hạn - edge case**
- **Given** DL-0456 không có hợp đồng nào hết hạn từ 02/06/2027 đến 01/07/2027
- **When** đại lý xem danh sách "Sắp đến hạn tái tục"
- **Then** hệ thống hiển thị "Không có hợp đồng nào đến hạn tái tục trong 30 ngày tới"

**AC3: Khách hàng đã chuyển sang đại lý khác - edge case**
- **Given** khách hàng Hoàng Văn E có hợp đồng hết hạn 15/06/2027, đã được chuyển từ DL-0456 sang DL-0789 ngày 25/05/2027
- **When** đại lý DL-0456 xem danh sách "Sắp đến hạn tái tục"
- **Then** danh sách không có hợp đồng của Hoàng Văn E

**AC4: Đại lý xem hợp đồng của khách hàng thuộc đại lý khác - negative path (phân quyền)**
- **Given** hợp đồng SK-2027-000789 của khách hàng Phạm Thị D thuộc danh mục đại lý DL-0789
- **When** đại lý DL-0456 yêu cầu xem chi tiết hợp đồng SK-2027-000789
- **Then** hệ thống từ chối với thông báo "Bạn không có quyền xem hợp đồng này"
- **And** không hiển thị tên, số điện thoại, phí hoặc quyền lợi của Phạm Thị D
- **And** lượt truy cập bị từ chối được ghi vào nhật ký truy cập với mã DL-0456 và thời điểm yêu cầu

**Notes:**
- Dependencies: Dữ liệu phân công khách hàng - đại lý được cập nhật khi chuyển giao danh mục
- Assumptions: "30 ngày tới" tính từ ngày hôm sau đến hết ngày thứ 30 (02/06 - 01/07/2027)
- Open Questions: Trưởng nhóm đại lý có được xem danh sách của đại lý trong nhóm không? (nếu có → story riêng theo persona)

**Readiness (DoR):** ✅ Sẵn sàng

---

## Ví dụ 5: Tái tục hợp đồng sức khỏe online

### US-RENEW-001: Tái tục hợp đồng bảo hiểm sức khỏe cá nhân online

**As a** khách hàng có hợp đồng "Sức khỏe Toàn diện" đến kỳ tái tục (còn ≤ 30 ngày đến hạn hoặc đang trong 30 ngày gia hạn)
**I want to** thanh toán phí tái tục trên ứng dụng để gia hạn hợp đồng thêm 1 năm
**So that** tôi được bảo vệ liên tục, không phải chịu lại thời gian chờ như khi mua hợp đồng mới

**Metadata:** Epic: EP-RETENTION · Priority: Must

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ⚠️ | Cần story thanh toán thẻ (US-PAY-001) đã có; không cần story khác |
| Negotiable | ✅ | Độ dài thời gian gia hạn, khoảng mở tái tục có thể thảo luận |
| Valuable | ✅ | Khách không gián đoạn quyền lợi; công ty giữ doanh thu tái tục |
| Estimable | ✅ | Quy tắc gia hạn đã có trong quy tắc sản phẩm |
| Small | ✅ | ~4 ngày dev |
| Testable | ✅ | Có ngày biên, mức phí, trạng thái cụ thể |

**Background** (áp dụng cho AC1-AC4):

```gherkin
Background:
  Given hợp đồng SK-2027-000456 của khách hàng Nguyễn Văn F hiệu lực từ 01/07/2027 đến 30/06/2028
  And phí kỳ hiện tại là 5.200.000 VND
```

**AC1: Tái tục trước ngày hết hạn - happy path**
- **Given** ngày hiện tại là 10/06/2028
- **And** phí tái tục kỳ 01/07/2028 - 30/06/2029 là 5.800.000 VND theo biểu phí 2028
- **When** khách hàng thanh toán thành công 5.800.000 VND phí tái tục
- **Then** hợp đồng kỳ mới SK-2028-000456 được tạo với hiệu lực 01/07/2028 - 30/06/2029, trạng thái "Chờ hiệu lực"
- **And** thời gian chờ không áp dụng lại cho quyền lợi đã có ở kỳ trước
- **And** khách hàng nhận thông báo "Hợp đồng SK-2027-000456 đã được tái tục đến 30/06/2029"

**AC2: Ngày thanh toán so với khoảng tái tục - edge case + negative (Scenario Outline)**

```gherkin
Scenario Outline: Tái tục theo khoảng mở 30 ngày trước hạn và 30 ngày gia hạn
  Given khoảng tái tục từ 31/05/2028 đến 30/07/2028
  When khách hàng thanh toán phí tái tục vào ngày <ngay_thanh_toan>
  Then kết quả là <ket_qua>

  Examples:
    | ngay_thanh_toan                  | ket_qua                                                                                            |
    | 30/05/2028 (31 ngày trước hạn)   | từ chối: "Hợp đồng SK-2027-000456 được tái tục từ ngày 31/05/2028", không thu phí                  |
    | 31/05/2028 (30 ngày trước hạn)   | tái tục thành công, kỳ mới hiệu lực từ 01/07/2028                                                  |
    | 30/07/2028 (ngày thứ 30 gia hạn) | tái tục thành công, kỳ mới hiệu lực liên tục từ 01/07/2028                                         |
    | 31/07/2028 (ngày thứ 31 gia hạn) | từ chối: "Hợp đồng SK-2027-000456 đã quá thời hạn tái tục. Vui lòng mua hợp đồng mới", không thu phí |
```

**AC3: Biểu phí mới không ảnh hưởng kỳ hợp đồng đang hiệu lực - side effect**
- **Given** biểu phí 2028 tăng phí gói "Sức khỏe Toàn diện" cho nhóm tuổi của Nguyễn Văn F lên 5.800.000 VND
- **And** hạn mức ngoại trú còn lại của kỳ hiện tại là 4.000.000 VND
- **When** biểu phí 2028 có hiệu lực vào ngày 01/01/2028
- **Then** hợp đồng SK-2027-000456 giữ nguyên phí 5.200.000 VND, ngày hết hạn 30/06/2028
- **And** hạn mức ngoại trú còn lại vẫn là 4.000.000 VND
- **And** phí tái tục hiển thị cho kỳ 01/07/2028 - 30/06/2029 là 5.800.000 VND

**AC4: Thanh toán phí tái tục thất bại - negative path**
- **Given** ngày hiện tại là 15/07/2028 (trong thời gian gia hạn)
- **When** ngân hàng từ chối giao dịch thanh toán 5.800.000 VND phí tái tục
- **Then** không có hợp đồng kỳ mới nào được tạo, hợp đồng SK-2027-000456 ở trạng thái "Chờ tái tục"
- **And** khách hàng nhận thông báo "Thanh toán không thành công. Hợp đồng chưa được tái tục, hạn cuối tái tục: 30/07/2028"

**Notes:**
- Dependencies: US-PAY-001 (thanh toán thẻ); biểu phí 2028 được phê duyệt trước 01/01/2028
- Assumptions: Khoảng tái tục mở 30 ngày trước ngày hết hạn và kéo dài 30 ngày gia hạn sau ngày hết hạn
- Open Questions: Sự kiện bảo hiểm xảy ra trong thời gian gia hạn, trước khi khách đóng phí, có được bồi thường không? (cần Pháp chế xác nhận)

**Readiness (DoR):** ⚠️ Còn thiếu: Pháp chế xác nhận quyền lợi trong thời gian gia hạn

---

## Patterns rút ra

1. **Phí theo ngưỡng → Scenario Outline với cặp biên** (17/18, 65/66); nêu rõ tuổi tính tại ngày bắt đầu hiệu lực, không phải ngày yêu cầu.
2. **Thời gian chờ, thời gian gia hạn là nguồn edge case chính**: ghi quy ước đếm ngày (ngày bắt đầu hiệu lực = ngày thứ 1) trong Assumptions, rồi kiểm tra cặp ngày N / N+1.
3. **Tác động lên dữ liệu có sẵn**: biểu phí mới chỉ áp dụng cho kỳ tái tục hoặc hợp đồng mới; AC khẳng định phí, ngày hết hạn, hạn mức còn lại của hợp đồng đang hiệu lực không đổi.
4. **Phân quyền theo danh mục đại lý**: AC âm nêu cả message từ chối lẫn dữ liệu không được lộ (tên, số điện thoại, phí), kèm ghi nhận lượt truy cập bị từ chối.
5. **Tách tiếp nhận khỏi thẩm định**: story bồi thường dừng ở trạng thái "Đã tiếp nhận"; thẩm định, chi trả là story riêng để giữ story Small.
6. **Mua online nhiều bước → split theo workflow + phương thức thanh toán**: báo giá là lát đầu tiên tự có giá trị, sales dùng được ngay khi chưa có thanh toán online.

---
*Tài liệu bổ sung cho skill user-story-ac-writer (bản gốc: Phúc NT · BA Zone · Digital School)*
