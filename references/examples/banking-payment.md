# Ví dụ User Story + AC mẫu - Ngân hàng / Thanh toán

> 5 ví dụ cho ngân hàng bán lẻ và ví điện tử tại Việt Nam: chuyển khoản nhanh 24/7,
> mở tài khoản eKYC (minh họa split), thanh toán VietQR, tạm khóa thẻ, tất toán khoản vay.
>
> ⚠️ Các con số (phí, hạn mức, thời gian chờ, hiệu lực mã QR) chỉ để **minh họa**, không phải
> quy định pháp lý - team phải thay bằng business rule thật trước khi dùng. Tên tổ chức đã generic hóa.

---

## Ví dụ 1: Chuyển khoản nhanh liên ngân hàng 24/7

### US-TRANSFER-001: Chuyển khoản nhanh 24/7 tới tài khoản tại ngân hàng khác

**As a** khách hàng cá nhân của Ngân hàng X đã kích hoạt Smart OTP trên ứng dụng di động
**I want to** chuyển tiền tới tài khoản tại ngân hàng khác bằng số tài khoản người nhận
**So that** người nhận có tiền cả vào buổi tối, cuối tuần, ngày lễ mà tôi không phải ra quầy giao dịch

**Metadata:** Epic: EP-TRANSFER - Chuyển tiền · Priority: Must

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Tra cứu tên người nhận là story riêng, đã hoàn thành |
| Negotiable | ✅ | Chưa chốt thời gian chờ tối đa và cách thông báo kết quả cuối |
| Valuable | ✅ | Khách hàng chuyển tiền ngoài giờ; ngân hàng tăng số giao dịch trên kênh số |
| Estimable | ✅ | Luồng chuyển khoản nội bộ đã có, chỉ khác đích đến |
| Small | ✅ | ~5 ngày dev; giao dịch treo giữ trong story vì là rủi ro cốt lõi về tiền |
| Testable | ✅ | Mỗi AC có số dư trước/sau và trạng thái giao dịch cụ thể |

```gherkin
Background:
  Given khách hàng đã đăng nhập ứng dụng Ngân hàng X
  And tài khoản thanh toán ở trạng thái "Hoạt động"
  And tài khoản người nhận 0123456789 tại Ngân hàng Y đã được xác minh tên "NGUYEN VAN B"
```

**AC1: Chuyển khoản thành công - happy path**
- **Given** số dư khả dụng là 10.000.000 VND
- **When** khách hàng xác nhận lệnh chuyển 2.000.000 VND bằng Smart OTP hợp lệ
- **Then** giao dịch có trạng thái "Thành công" trong ≤ 10 giây
- **And** số dư khả dụng còn 8.000.000 VND (phí 0 VND theo biểu phí minh họa)
- **And** khách hàng nhận biên lai gồm mã giao dịch, số tiền, tên người nhận, thời điểm giao dịch

**AC2: Kiểm tra hạn mức giao dịch - boundary**
```gherkin
Scenario Outline: Chặn lệnh chuyển theo hạn mức
  Given số dư khả dụng là 1.500.000.000 VND
  And tổng số tiền đã chuyển đi trong ngày là <da_chuyen> VND
  When khách hàng xác nhận lệnh chuyển <so_tien> VND bằng Smart OTP hợp lệ
  Then kết quả là <ket_qua>
  And số dư khả dụng sau đó là <so_du_sau> VND

  Examples:
    | da_chuyen   | so_tien     | ket_qua                                                        | so_du_sau     |
    | 0           | 0           | từ chối: "Số tiền chuyển phải lớn hơn 0 VND"                   | 1.500.000.000 |
    | 0           | 299.999.999 | "Thành công"                                                   | 1.200.000.001 |
    | 0           | 300.000.000 | "Thành công"                                                   | 1.200.000.000 |
    | 0           | 300.000.001 | từ chối: "Vượt hạn mức 300.000.000 VND/giao dịch"              | 1.500.000.000 |
    | 800.000.000 | 200.000.000 | "Thành công"                                                   | 1.300.000.000 |
    | 800.000.000 | 200.000.001 | từ chối: "Vượt hạn mức ngày. Hạn mức còn lại: 200.000.000 VND" | 1.500.000.000 |
```

**AC3: Số dư không đủ - negative**
- **Given** số dư khả dụng là 1.000.000 VND
- **When** khách hàng xác nhận lệnh chuyển 1.500.000 VND bằng Smart OTP hợp lệ
- **Then** hệ thống từ chối với thông báo "Số dư khả dụng không đủ để thực hiện giao dịch"
- **And** số dư khả dụng vẫn là 1.000.000 VND
- **And** không phát sinh giao dịch ghi nợ trên tài khoản

**AC4: Ngân hàng nhận không phản hồi - timeout**
- **Given** khách hàng đã xác nhận lệnh chuyển 2.000.000 VND lúc 22:15:00 khi số dư khả dụng là 10.000.000 VND
- **And** chưa có kết quả từ Ngân hàng Y
- **When** thời gian chờ kết quả đạt 60 giây
- **Then** giao dịch có trạng thái "Đang xử lý"
- **And** 2.000.000 VND được tạm giữ, số dư khả dụng còn 8.000.000 VND
- **And** khách hàng thấy thông báo "Giao dịch đang được xử lý. Vui lòng không chuyển lại. Kết quả sẽ được gửi trong vòng 30 phút"

**AC5: Gửi lại lệnh đang xử lý - chống trừ tiền 2 lần**
- **Given** lệnh chuyển FT2609250001 số tiền 2.000.000 VND đang ở trạng thái "Đang xử lý"
- **And** số dư khả dụng là 8.000.000 VND, trong đó 2.000.000 VND đang tạm giữ cho lệnh này
- **When** khách hàng gửi lại lệnh chuyển FT2609250001
- **Then** hệ thống không tạo giao dịch mới
- **And** khách hàng thấy thông báo "Lệnh chuyển FT2609250001 đang được xử lý. Vui lòng chờ kết quả"
- **And** số dư khả dụng vẫn là 8.000.000 VND

**Notes:**
- **Dependencies:** US-TRANSFER-002 (tra soát, trả kết quả cuối cho giao dịch "Đang xử lý") làm sau, không chặn story này
- **Assumptions:** Hạn mức ngày tính theo tổng lệnh chuyển đi từ 00:00 đến 23:59 giờ Việt Nam; luồng nhập sai Smart OTP thuộc story xác thực chung
- **Open Questions:** Giao dịch "Đang xử lý" có bị tính vào hạn mức ngày không? Có cảnh báo khi khách hàng tạo lệnh **mới** trùng người nhận và số tiền trong 5 phút không?

**Readiness (DoR):** ⚠️ Còn thiếu: PO chốt câu hỏi hạn mức ngày cho giao dịch "Đang xử lý" (cần chốt trước sprint planning vì có thể thêm dòng Examples cho AC2)

---

## Ví dụ 2: Mở tài khoản online bằng eKYC (minh họa split story)

**❌ Story gốc (quá to, chưa gán ID):**
> **As a** người chưa có tài khoản tại Ngân hàng X
> **I want to** mở tài khoản online, nhận thẻ ghi nợ, đăng ký Smart OTP, liên kết Ví X ngay trên ứng dụng
> **So that** dùng được đầy đủ dịch vụ ngân hàng mà không ra quầy

Dấu hiệu: 4 mục tiêu trong 1 câu "I want to", dev ước lượng > 10 ngày, AC dự kiến > 15 scenario.

### Đề xuất split: Story gốc "Mở tài khoản online trọn gói"
Chẩn đoán: **Compound** - gộp 4 story đã hiểu rõ nghiệp vụ (mở tài khoản, kích hoạt Smart OTP, phát hành thẻ, liên kết ví); không có ẩn số lớn cần Spike
Pattern áp dụng: **Bước workflow** + **Dữ liệu - lõi tối thiểu**

| Story con | Giá trị riêng | Ưu tiên gợi ý |
|-----------|---------------|---------------|
| US-EKYC-001: Mở tài khoản thanh toán online bằng CCCD gắn chip | Khách hàng có số tài khoản để nhận tiền trong ngày | Must |
| US-EKYC-002: Kích hoạt Smart OTP cho tài khoản mới mở | Khách hàng tự xác thực lệnh chuyển tiền đi | Must |
| US-EKYC-003: Phát hành thẻ ghi nợ phi vật lý cho tài khoản mới | Thanh toán online bằng thẻ trong ngày mở tài khoản | Should |
| US-EKYC-004: Liên kết tài khoản mới với Ví X | Nạp tiền vào ví không cần chuyển khoản thủ công | Could |

Story nào giao được trước mà vẫn có giá trị: **US-EKYC-001** (tài khoản nhận được tiền dù chưa có Smart OTP hay thẻ)

### US-EKYC-001: Mở tài khoản thanh toán online bằng CCCD gắn chip

**As a** người từ 18 tuổi trở lên, có CCCD gắn chip còn hạn, chưa là khách hàng của Ngân hàng X
**I want to** mở tài khoản thanh toán bằng cách xác thực chip CCCD và khuôn mặt trên ứng dụng
**So that** nhận được lương, tiền chuyển đến trong ngày đăng ký mà không phải xếp hàng tại quầy

**Metadata:** Epic: EP-ONBOARD - Mở tài khoản số · Priority: Must

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Có giá trị khi chưa có Smart OTP, thẻ, liên kết ví |
| Negotiable | ✅ | Chưa chốt số lần thử khuôn mặt, thời gian tạm chặn |
| Valuable | ✅ | Khách hàng có tài khoản trong ngày; ngân hàng giảm tải quầy |
| Estimable | ✅ | Đã dùng đọc chip CCCD ở luồng cập nhật hồ sơ |
| Small | ✅ | ~4 ngày dev sau khi tách 3 story con còn lại |
| Testable | ✅ | Trạng thái hồ sơ, thông báo, thời gian tạm chặn đều cụ thể |

**AC1: Mở tài khoản thành công - happy path**
- **Given** chip CCCD của khách hàng Trần Thị C đọc được, CCCD còn hạn đến 12/2035
- **And** ảnh khuôn mặt chụp trực tiếp khớp với ảnh trong chip CCCD
- **When** khách hàng gửi yêu cầu mở tài khoản
- **Then** tài khoản thanh toán mới được tạo với trạng thái "Hoạt động"
- **And** khách hàng thấy số tài khoản mới kèm thông báo "Tài khoản của bạn đã sẵn sàng nhận tiền"
- **And** tài khoản có hạn mức chuyển đi 100.000.000 VND/ngày dành cho tài khoản mở online
- **And** khách hàng nhận SMS xác nhận mở tài khoản tới số điện thoại đã đăng ký

**AC2: Khuôn mặt không khớp lần thứ 3 - edge case**
- **Given** khách hàng đã xác thực khuôn mặt không khớp 2 lần trong phiên hiện tại
- **When** lần xác thực khuôn mặt thứ 3 cũng không khớp
- **Then** phiên mở tài khoản dừng với thông báo "Không xác thực được khuôn mặt. Vui lòng thử lại sau 24 giờ hoặc đến quầy giao dịch"
- **And** số CCCD này bị tạm chặn mở tài khoản online trong 24 giờ
- **And** không có tài khoản nào được tạo

**AC3: CCCD đã là khách hàng hiện hữu - negative**
- **Given** số CCCD đã gắn với một hồ sơ khách hàng của Ngân hàng X
- **When** người dùng gửi yêu cầu mở tài khoản bằng CCCD đó
- **Then** hệ thống từ chối với thông báo "Số CCCD đã được đăng ký tại Ngân hàng X. Vui lòng đăng nhập bằng tài khoản hiện có"
- **And** không tạo hồ sơ khách hàng mới

**AC4: CCCD đã hết hạn - negative**
- **Given** chip CCCD ghi ngày hết hạn 20/09/2026, ngày thực hiện là 25/09/2026
- **When** khách hàng gửi yêu cầu mở tài khoản
- **Then** hệ thống từ chối với thông báo "CCCD đã hết hạn. Vui lòng dùng CCCD còn hiệu lực"
- **And** không có tài khoản nào được tạo

**Notes:**
- **Dependencies:** Không có dependency chặn; US-EKYC-002 cần story này xong trước
- **Assumptions:** Thiết bị của khách hàng đọc được chip CCCD; hạn mức 100.000.000 VND/ngày là số minh họa
- **Open Questions:** Có mở tài khoản online cho người từ 15 đến dưới 18 tuổi không? Pháp chế xác nhận hạn mức áp dụng cho tài khoản mở online?

**Readiness (DoR):** ⚠️ Còn thiếu: Pháp chế xác nhận hạn mức ở AC1 (có thể bổ sung trong sprint, không chặn estimate)

---

## Ví dụ 3: Thanh toán tại điểm bán bằng VietQR

### US-QRPAY-001: Thanh toán đơn hàng tại cửa hàng bằng mã VietQR động trên Ví X

**As a** người dùng Ví X đã định danh eKYC, đang ở quầy thu ngân của cửa hàng chấp nhận VietQR
**I want to** quét mã VietQR động của đơn hàng để thanh toán đúng số tiền cửa hàng yêu cầu
**So that** không cần mang tiền mặt, không nhập tay số tiền nên không chuyển nhầm cho cửa hàng

**Metadata:** Epic: EP-QR - Thanh toán tại điểm bán · Priority: Must

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Mã QR tĩnh (khách tự nhập số tiền) là story riêng |
| Negotiable | ✅ | Thời gian hiệu lực mã QR còn thảo luận với cửa hàng đối tác |
| Valuable | ✅ | Người dùng thanh toán không tiền mặt; cửa hàng nhận đúng số tiền |
| Estimable | ✅ | Đọc mã VietQR đã có ở luồng chuyển khoản |
| Small | ✅ | ~3 ngày dev |
| Testable | ✅ | Số dư ví, trạng thái đơn hàng, thông báo đều cụ thể |

```gherkin
Background:
  Given người dùng đã đăng nhập Ví X
  And Cửa hàng Z hiển thị mã VietQR động cho đơn hàng HD-0925-017 trị giá 185.000 VND
  And mã QR được tạo lúc 14:00:00, hết hiệu lực từ 14:10:00
```

**AC1: Thanh toán thành công - happy path**
- **Given** số dư Ví X là 500.000 VND
- **And** đơn hàng HD-0925-017 ở trạng thái "Chờ thanh toán"
- **When** người dùng xác nhận thanh toán mã QR lúc 14:03:00
- **Then** giao dịch có trạng thái "Thành công", số dư Ví X còn 315.000 VND
- **And** đơn hàng HD-0925-017 chuyển sang trạng thái "Đã thanh toán"
- **And** người dùng thấy thông báo "Thanh toán thành công 185.000 VND cho Cửa hàng Z"

**AC2: Hiệu lực mã QR - boundary**
```gherkin
Scenario Outline: Chấp nhận mã QR theo thời điểm xác nhận
  Given số dư Ví X là 500.000 VND
  And đơn hàng HD-0925-017 ở trạng thái "Chờ thanh toán"
  When người dùng xác nhận thanh toán mã QR lúc <thoi_diem>
  Then kết quả là <ket_qua>
  And số dư Ví X sau đó là <so_du_sau> VND

  Examples:
    | thoi_diem | ket_qua                                                           | so_du_sau |
    | 14:09:59  | "Thành công"                                                      | 315.000   |
    | 14:10:00  | từ chối: "Mã QR đã hết hạn. Vui lòng yêu cầu cửa hàng tạo mã mới" | 500.000   |
```

**AC3: Quét lại mã của đơn hàng đã thanh toán - chống thanh toán trùng**
- **Given** đơn hàng HD-0925-017 đã ở trạng thái "Đã thanh toán" lúc 14:03:00
- **And** số dư Ví X là 315.000 VND
- **When** người dùng xác nhận thanh toán lại mã QR của đơn hàng này lúc 14:04:00
- **Then** hệ thống từ chối với thông báo "Đơn hàng này đã được thanh toán lúc 14:03. Bạn không bị trừ tiền lần nữa"
- **And** số dư Ví X vẫn là 315.000 VND

**AC4: Số dư ví không đủ - negative**
- **Given** số dư Ví X là 100.000 VND
- **When** người dùng xác nhận thanh toán mã QR lúc 14:03:00
- **Then** hệ thống từ chối với thông báo "Số dư ví không đủ. Cần nạp thêm 85.000 VND"
- **And** đơn hàng HD-0925-017 vẫn ở trạng thái "Chờ thanh toán"
- **And** số dư Ví X vẫn là 100.000 VND

**Notes:**
- **Dependencies:** Cửa hàng đối tác đã tạo được mã VietQR động (phía cửa hàng, ngoài phạm vi story)
- **Assumptions:** Số tiền trong mã QR động do cửa hàng quyết định, người dùng không sửa được; hiệu lực 10 phút là số minh họa
- **Open Questions:** Ví chưa định danh có được thanh toán QR không, với hạn mức bao nhiêu?

**Readiness (DoR):** ✅ Sẵn sàng cho refinement

---

## Ví dụ 4: Tạm khóa thẻ ghi nợ

### US-CARD-001: Tạm khóa thẻ ghi nợ trên ứng dụng

**As a** chủ thẻ ghi nợ chính của Ngân hàng X đang nghi ngờ thẻ vật lý bị thất lạc
**I want to** tạm khóa thẻ ghi nợ của mình trên ứng dụng
**So that** không phát sinh giao dịch gian lận trong lúc tìm lại thẻ, mà không phải gọi tổng đài hay hủy thẻ vĩnh viễn

**Metadata:** Epic: EP-CARD - Quản lý thẻ · Priority: Must

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Mở khóa thẻ là story riêng (US-CARD-002) |
| Negotiable | ✅ | Chưa chốt kênh thông báo khi giao dịch bị từ chối |
| Valuable | ✅ | Chủ thẻ tự chặn rủi ro; giảm cuộc gọi tổng đài báo mất thẻ |
| Estimable | ✅ | Trạng thái "Tạm khóa" đã có ở hệ thống thẻ, chỉ thêm kênh ứng dụng |
| Small | ✅ | ~3 ngày dev |
| Testable | ✅ | Có trạng thái thẻ trước/sau, số tiền giao dịch cụ thể |

**AC1: Tạm khóa thẻ thành công - happy path**
- **Given** chủ thẻ chính đã đăng nhập ứng dụng
- **And** thẻ ghi nợ đuôi 4821 đang ở trạng thái "Hoạt động"
- **When** chủ thẻ xác nhận tạm khóa thẻ 4821 bằng Smart OTP hợp lệ lúc 09:00
- **Then** thẻ 4821 chuyển sang trạng thái "Tạm khóa"
- **And** chủ thẻ nhận thông báo "Thẻ đuôi 4821 đã được tạm khóa lúc 09:00 25/09/2026"

**AC2: Giao dịch mới trên thẻ đang tạm khóa - edge case**
- **Given** thẻ 4821 đang ở trạng thái "Tạm khóa"
- **And** số dư tài khoản gắn với thẻ là 5.000.000 VND
- **When** một giao dịch thanh toán 850.000 VND bằng thẻ 4821 được gửi từ máy POS
- **Then** giao dịch bị từ chối với lý do "Thẻ đang tạm khóa"
- **And** số dư tài khoản vẫn là 5.000.000 VND
- **And** chủ thẻ nhận thông báo "Giao dịch 850.000 VND bị từ chối do thẻ đang tạm khóa"

**AC3: Giao dịch đã cấp phép trước khi khóa - tác động lên dữ liệu có sẵn**
- **Given** lúc 08:30 thẻ 4821 được cấp phép giao dịch khách sạn 1.200.000 VND, số tiền này đang tạm giữ
- **And** thẻ 4821 chuyển sang trạng thái "Tạm khóa" lúc 09:00
- **When** khách sạn gửi yêu cầu quyết toán giao dịch 1.200.000 VND
- **Then** giao dịch được ghi nợ 1.200.000 VND từ khoản đang tạm giữ
- **And** thẻ 4821 vẫn ở trạng thái "Tạm khóa"

**AC4: Chủ thẻ phụ tạm khóa thẻ chính - phân quyền (negative)**
- **Given** người dùng đang đăng nhập là chủ thẻ phụ đuôi 7730, phát hành theo thẻ chính 4821
- **And** thẻ 4821 đang ở trạng thái "Hoạt động"
- **When** người dùng này yêu cầu tạm khóa thẻ 4821
- **Then** hệ thống từ chối với thông báo "Chỉ chủ thẻ chính được tạm khóa thẻ này"
- **And** thẻ 4821 vẫn ở trạng thái "Hoạt động"

**Notes:**
- **Dependencies:** US-CARD-002 (mở khóa thẻ) nên vào cùng release để chủ thẻ không phải gọi tổng đài khi tìm lại thẻ
- **Assumptions:** Tạm khóa chỉ chặn giao dịch **mới**; giao dịch đã cấp phép trước thời điểm khóa vẫn được quyết toán
- **Open Questions:** Đăng ký thanh toán định kỳ bằng thẻ (hóa đơn điện, gói cước) có bị chặn khi thẻ tạm khóa không? Chủ thẻ chính khóa thẻ chính thì thẻ phụ 7730 có bị khóa theo không?

**Readiness (DoR):** ⚠️ Còn thiếu: PO trả lời câu hỏi thanh toán định kỳ (bắt buộc, có thể sinh thêm 1 AC)

---

## Ví dụ 5: Tất toán khoản vay tiêu dùng trước hạn

### US-LOAN-001: Tất toán khoản vay tiêu dùng trước hạn trên ứng dụng

**As a** khách hàng đang có khoản vay tiêu dùng tín chấp tại Ngân hàng X, chưa có kỳ trả nợ quá hạn
**I want to** tất toán toàn bộ dư nợ trước hạn bằng tiền trong tài khoản thanh toán
**So that** không phải trả lãi cho các kỳ còn lại, chấm dứt nghĩa vụ trả góp hàng tháng

**Metadata:** Epic: EP-LOAN - Khoản vay số · Priority: Should

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Trả trước một phần dư nợ là story riêng (US-LOAN-002) |
| Negotiable | ✅ | Biểu phí trả trước hạn do khối tín dụng chốt, story chỉ áp dụng |
| Valuable | ✅ | Khách hàng tiết kiệm lãi; ngân hàng giảm cuộc gọi, giấy tờ tại quầy |
| Estimable | ✅ | Công thức lãi đến ngày tất toán đã có ở màn hình quầy |
| Small | ✅ | ~4 ngày dev |
| Testable | ✅ | Có số tiền gốc, lãi, phí, số dư trước/sau cụ thể |

**AC1: Tất toán thành công - happy path**
- **Given** khoản vay LN-2025-0715 có dư nợ gốc 50.000.000 VND, đã giải ngân được 14 tháng
- **And** lãi tính đến ngày 25/09/2026 là 312.000 VND
- **And** số dư tài khoản thanh toán là 60.000.000 VND
- **When** khách hàng xác nhận tất toán khoản vay LN-2025-0715 bằng Smart OTP hợp lệ
- **Then** tài khoản bị trích 51.312.000 VND (gốc 50.000.000 + lãi 312.000 + phí trả trước hạn 1.000.000)
- **And** khoản vay chuyển sang trạng thái "Đã tất toán", số dư tài khoản còn 8.688.000 VND
- **And** khách hàng nhận thông báo "Khoản vay LN-2025-0715 đã được tất toán ngày 25/09/2026"

**AC2: Phí trả nợ trước hạn theo thời gian đã vay - boundary**
```gherkin
Scenario Outline: Tính phí trả nợ trước hạn
  Given khoản vay có dư nợ gốc 50.000.000 VND
  And khoản vay đã giải ngân được <so_thang> tháng
  When khách hàng yêu cầu xem số tiền cần trả để tất toán
  Then kết quả là <ket_qua>

  Examples:
    | so_thang | ket_qua                                                             |
    | 0        | từ chối: "Chỉ được tất toán sau tối thiểu 1 tháng kể từ ngày giải ngân" |
    | 11       | phí trả trước hạn 1.500.000 VND (3% dư nợ gốc)                      |
    | 12       | phí trả trước hạn 1.000.000 VND (2% dư nợ gốc)                      |
    | 23       | phí trả trước hạn 1.000.000 VND (2% dư nợ gốc)                      |
    | 24       | phí trả trước hạn 0 VND                                             |
```

**AC3: Số dư không đủ để tất toán - negative**
- **Given** khoản vay LN-2025-0715 cần 51.312.000 VND để tất toán
- **And** số dư tài khoản thanh toán là 40.000.000 VND
- **When** khách hàng xác nhận tất toán khoản vay LN-2025-0715 bằng Smart OTP hợp lệ
- **Then** hệ thống từ chối với thông báo "Số dư không đủ. Cần thêm 11.312.000 VND để tất toán"
- **And** khoản vay vẫn ở trạng thái "Đang vay" với dư nợ gốc 50.000.000 VND
- **And** số dư tài khoản vẫn là 40.000.000 VND

**AC4: Hủy lịch trích nợ tự động - tác động lên dữ liệu có sẵn**
- **Given** khoản vay LN-2025-0715 kỳ hạn 36 tháng còn 22 lệnh trích nợ tự động chưa đến hạn, gần nhất là kỳ 15/10/2026 (4.850.000 VND)
- **When** khoản vay LN-2025-0715 chuyển sang trạng thái "Đã tất toán"
- **Then** 22 lệnh trích nợ tự động từ kỳ 15/10/2026 trở đi chuyển sang trạng thái "Đã hủy"
- **And** lịch trả nợ của khoản vay hiển thị "0 kỳ còn lại"

**Notes:**
- **Dependencies:** Biểu phí trả nợ trước hạn đã được khối tín dụng phê duyệt (số trong AC2 là minh họa)
- **Assumptions:** Khoản vay có kỳ quá hạn phải tất toán tại quầy, ngoài phạm vi story này
- **Open Questions:** Tất toán đúng ngày đến hạn một kỳ (lệnh trích tự động đã chạy sáng cùng ngày) thì số tiền tất toán tính trên dư nợ nào?

**Readiness (DoR):** ✅ Sẵn sàng cho refinement

---

## Patterns rút ra

1. **Tiền phải "khép sổ" trong từng AC**: nêu số dư trước/sau, kể cả AC bị từ chối ("số dư vẫn là ...") để QA chứng minh được "không trừ tiền".
2. **Timeout ≠ thất bại**: giao dịch treo có trạng thái "Đang xử lý", tiền được tạm giữ; gửi lại cùng lệnh không tạo giao dịch thứ hai (idempotency); kết quả cuối do story tra soát / đối soát riêng xử lý.
3. **Hạn mức, phí, hiệu lực mã → Scenario Outline**: mỗi ngưỡng lấy giá trị ngay dưới và đúng ngưỡng, thêm 0 và vượt ngưỡng; mỗi dòng có kết quả cụ thể.
4. **OTP / sinh trắc gói gọn trong When**: "xác nhận ... bằng Smart OTP hợp lệ", không liệt kê từng bước nhập mã; luồng nhập sai OTP là story xác thực chung.
5. **Dữ liệu có sẵn hay bị quên**: giao dịch đã cấp phép trước khi khóa thẻ, lệnh trích nợ tự động sau tất toán, đơn hàng đã thanh toán khi quét lại mã QR.
6. **Phân quyền theo quan hệ sở hữu**: chủ thẻ chính / thẻ phụ, ví đã / chưa định danh, khách hàng mới / hiện hữu - viết AC negative cho người **không** được làm.

---
*Tài liệu bổ sung cho skill user-story-ac-writer (bản gốc: Phúc NT · BA Zone · Digital School)*
