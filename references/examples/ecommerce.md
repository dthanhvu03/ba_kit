# Ví dụ User Story + AC mẫu - Thương mại điện tử

> 5 ví dụ cho sàn thương mại điện tử (marketplace) Việt Nam: đặt hàng COD, voucher của shop,
> seller đổi giá sản phẩm, đổi trả hàng (demo split story) và đánh giá sản phẩm.
>
> ⚠️ Các con số (giá, phí vận chuyển, hạn mức COD, thời hạn đổi trả, hạn đánh giá) chỉ mang tính
> minh họa - thay bằng business rule thực tế của team trước khi dùng.
>
> Tên thương hiệu được generic hóa: Sàn X, Đơn vị vận chuyển X, shop "Thời trang A", shop "Gia dụng B".

---

## Ví dụ 1: Đặt hàng thanh toán khi nhận hàng (COD)

### US-ORDER-001: Đặt hàng COD từ giỏ hàng nhiều sản phẩm

**As a** người mua đã đăng nhập Sàn X, đã lưu địa chỉ nhận hàng mặc định tại TP.HCM
**I want to** đặt 1 đơn hàng thanh toán khi nhận hàng cho các sản phẩm đang chọn trong giỏ
**So that** kiểm tra được hàng trước khi trả tiền, không chịu rủi ro trả trước cho shop lần đầu mua

**Metadata:** Epic: Đặt hàng · Priority: Must · Estimate: ~3 ngày

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Giỏ hàng, sổ địa chỉ đã có (story riêng); không phụ thuộc voucher |
| Negotiable | ✅ | Hạn mức COD, vùng hỗ trợ thu hộ còn thảo luận với PO |
| Valuable | ✅ | Người mua chưa có thẻ/ví vẫn mua được; sàn tăng tỷ lệ chuyển đổi |
| Estimable | ✅ | Luồng tạo đơn tương tự đơn thanh toán online |
| Small | ✅ | Giới hạn giỏ hàng 1 shop, ~3 ngày dev |
| Testable | ✅ | Tổng tiền, tồn kho, trạng thái đơn đều đo được |

**AC1: Đặt hàng COD thành công - happy path**
- **Given** giỏ hàng của người mua đang chọn các sản phẩm sau của shop "Thời trang A":

  ```
  | sản phẩm       | số lượng | đơn giá (VND) | tồn kho |
  | Áo thun basic  | 2        | 150.000       | 20      |
  | Quần jean slim | 1        | 450.000       | 5       |
  ```
- **And** phí vận chuyển tới địa chỉ mặc định là 30.000 VND
- **When** người mua đặt hàng với phương thức "Thanh toán khi nhận hàng"
- **Then** hệ thống tạo 1 đơn hàng trạng thái "Chờ xác nhận" với tổng thanh toán 780.000 VND (750.000 tiền hàng + 30.000 phí vận chuyển)
- **And** tồn kho còn: "Áo thun basic" 18, "Quần jean slim" 4
- **And** 2 dòng sản phẩm đã đặt được xóa khỏi giỏ hàng
- **And** người mua nhận thông báo "Đặt hàng thành công. Vui lòng chuẩn bị 780.000 VND khi nhận hàng"

**AC2: Race condition - 2 người mua cùng đặt sản phẩm cuối cùng**
- **Given** "Quần jean slim" còn tồn kho 1
- **And** giỏ hàng của người mua A và người mua B cùng chứa 1 "Quần jean slim"
- **And** đơn của người mua B chứa sản phẩm này được ghi nhận trước đơn của người mua A 1 giây
- **When** người mua A đặt hàng COD cho giỏ hàng đó
- **Then** hệ thống từ chối đơn của người mua A với thông báo "Sản phẩm 'Quần jean slim' vừa hết hàng. Vui lòng cập nhật giỏ hàng"
- **And** không tạo đơn hàng cho người mua A
- **And** tồn kho "Quần jean slim" là 0, không xuống số âm
- **And** "Quần jean slim" trong giỏ của người mua A chuyển sang trạng thái "Hết hàng"

**AC3: Đơn vượt hạn mức COD - negative**
- **Given** tổng thanh toán của giỏ hàng là 5.200.000 VND
- **And** hạn mức COD của Sàn X là 5.000.000 VND/đơn
- **When** người mua đặt hàng với phương thức "Thanh toán khi nhận hàng"
- **Then** hệ thống từ chối với thông báo "Đơn trên 5.000.000 VND không hỗ trợ thanh toán khi nhận hàng. Vui lòng chọn thanh toán online"
- **And** không tạo đơn hàng, tồn kho không thay đổi

**AC4: Địa chỉ ngoài vùng thu hộ - negative**
- **Given** địa chỉ nhận hàng mặc định thuộc "Xã đảo Y", nơi Đơn vị vận chuyển X không nhận thu hộ
- **When** người mua đặt hàng với phương thức "Thanh toán khi nhận hàng"
- **Then** hệ thống từ chối với thông báo "Địa chỉ này chưa hỗ trợ thanh toán khi nhận hàng. Vui lòng chọn thanh toán online hoặc đổi địa chỉ"
- **And** không tạo đơn hàng

**Notes:**
- **Dependencies:** Story giỏ hàng, sổ địa chỉ; bảng phí vận chuyển và danh sách vùng thu hộ của Đơn vị vận chuyển X
- **Assumptions:** Giỏ hàng chỉ gồm sản phẩm của 1 shop (đơn nhiều shop là story riêng); tồn kho bị trừ ngay khi tạo đơn, không chờ seller xác nhận
- **Open Questions:** Hạn mức COD áp theo từng đơn hay theo tổng đơn COD đang mở của 1 người mua? Người mua có lịch sử từ chối nhận hàng có bị chặn COD không?

**Readiness (DoR):** ⚠️ Còn thiếu: PO chốt hạn mức COD và danh sách vùng thu hộ chính thức

---

## Ví dụ 2: Áp voucher của shop

### US-VOUCHER-001: Áp voucher của shop cho đơn đang thanh toán

**As a** người mua đã đăng nhập, đang ở bước thanh toán đơn của shop "Thời trang A" và đã lưu voucher "GIAM50K" của shop
**I want to** áp voucher "GIAM50K" vào đơn hàng
**So that** trả ít hơn 50.000 VND cho đơn, còn shop tăng giá trị đơn trung bình nhờ điều kiện đơn tối thiểu

**Metadata:** Epic: Khuyến mãi · Priority: Should · Estimate: ~2 ngày

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ⚠️ | Cần voucher do seller tạo (US-SELLER-VOUCHER-001); có thể dùng voucher mẫu để test độc lập |
| Negotiable | ✅ | Cách hiển thị số tiền còn thiếu, thứ tự gợi ý voucher còn thảo luận |
| Valuable | ✅ | Người mua được giảm giá; shop kích cầu đơn lớn hơn |
| Estimable | ✅ | Điều kiện voucher đã chốt: đơn tối thiểu, hạn dùng, số lượt |
| Small | ✅ | Chỉ voucher của shop, chưa gồm voucher sàn, ~2 ngày |
| Testable | ✅ | Có bảng biên giá trị cụ thể |

Điều kiện voucher "GIAM50K" (minh họa): giảm 50.000 VND cho đơn có tiền hàng từ 300.000 VND (chưa gồm phí vận chuyển), hạn dùng đến 23:59 30/09/2026, tổng 1.000 lượt, mỗi người mua dùng 1 lần.

**AC1: Áp voucher thành công - happy path**
- **Given** đơn có tiền hàng 350.000 VND, phí vận chuyển 30.000 VND
- **And** người mua chưa từng dùng voucher "GIAM50K"
- **When** người mua áp voucher "GIAM50K"
- **Then** tổng thanh toán của đơn là 330.000 VND (350.000 − 50.000 + 30.000)
- **And** đơn hiển thị dòng "Voucher của shop: −50.000 VND"

**AC2: Điều kiện áp voucher theo biên giá trị - Scenario Outline**

```gherkin
Scenario Outline: Áp voucher "GIAM50K" theo đơn tối thiểu, hạn dùng, số lượt
  Given người mua chưa từng dùng voucher "GIAM50K"
  And đơn có tiền hàng <tien_hang> VND
  And thời điểm áp mã là <thoi_diem>
  And voucher còn <luot_con> lượt
  When người mua áp voucher "GIAM50K"
  Then số tiền được giảm là <giam> VND
  And hệ thống hiển thị thông báo <thong_bao>

  Examples:
    | tien_hang | thoi_diem        | luot_con | giam   | thong_bao                                       |
    | 299.000   | 20:00 30/09/2026 | 120      | 0      | "Mua thêm 1.000 VND để dùng voucher này"        |
    | 300.000   | 20:00 30/09/2026 | 120      | 50.000 | "Đã áp dụng voucher GIAM50K"                    |
    | 300.000   | 23:59 30/09/2026 | 120      | 50.000 | "Đã áp dụng voucher GIAM50K"                    |
    | 300.000   | 00:00 01/10/2026 | 120      | 0      | "Voucher đã hết hạn sử dụng"                    |
    | 300.000   | 20:00 30/09/2026 | 1        | 50.000 | "Đã áp dụng voucher GIAM50K"                    |
    | 300.000   | 20:00 30/09/2026 | 0        | 0      | "Voucher đã hết lượt sử dụng"                   |
```

**AC3: Người mua đã dùng voucher trước đó - negative**
- **Given** người mua đã dùng "GIAM50K" cho đơn DH0005 ở trạng thái "Hoàn thành"
- **And** đơn hiện tại có tiền hàng 400.000 VND, phí vận chuyển 30.000 VND
- **When** người mua áp voucher "GIAM50K"
- **Then** hệ thống từ chối với thông báo "Bạn đã sử dụng voucher GIAM50K (tối đa 1 lần/người mua)"
- **And** tổng thanh toán giữ nguyên 430.000 VND

**AC4: Giảm số lượng làm đơn rớt dưới ngưỡng - edge case**
- **Given** đơn gồm 2 "Áo thun basic" × 150.000 VND và 1 "Mũ lưỡi trai" × 50.000 VND, phí vận chuyển 30.000 VND
- **And** voucher "GIAM50K" đang được áp vào đơn
- **When** người mua giảm số lượng "Áo thun basic" từ 2 xuống 1
- **Then** voucher "GIAM50K" được gỡ khỏi đơn với thông báo "Đơn chưa đạt tối thiểu 300.000 VND. Voucher GIAM50K đã được gỡ"
- **And** tổng thanh toán là 230.000 VND (200.000 tiền hàng + 30.000 phí vận chuyển)

**Notes:**
- **Dependencies:** US-SELLER-VOUCHER-001 (seller tạo voucher); story lưu voucher vào ví voucher
- **Assumptions:** Lượt voucher chỉ bị trừ khi đơn đặt thành công, không trừ lúc áp mã; mốc hạn dùng tính theo giờ Việt Nam (GMT+7)
- **Open Questions:** Đơn bị hủy có hoàn lại lượt voucher không? Có cho cộng dồn voucher shop với voucher của Sàn X không?

**Readiness (DoR):** ✅ Sẵn sàng - câu hỏi hoàn lượt thuộc story hủy đơn, không chặn story này

---

## Ví dụ 3: Seller đổi giá sản phẩm

### US-SELLER-PRICE-001: Seller đổi giá bán sản phẩm của shop mình

**As a** chủ shop "Thời trang A" đã được Sàn X duyệt gian hàng, có sản phẩm ở trạng thái "Đang bán"
**I want to** đổi giá bán của 1 sản phẩm trong shop của mình
**So that** theo kịp giá nhập mới mà không phải ẩn sản phẩm rồi đăng lại, tránh mất lượt bán và đánh giá đã tích lũy

**Metadata:** Epic: Seller quản lý sản phẩm · Priority: Must · Estimate: ~2 ngày

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Đăng sản phẩm đã có (story riêng); không phụ thuộc story tồn kho |
| Negotiable | ✅ | Biên độ đổi giá tối đa, số lần đổi/ngày còn thảo luận |
| Valuable | ✅ | Seller giữ được lịch sử bán; người mua thấy giá đúng |
| Estimable | ✅ | Phạm vi 1 trường giá, không gồm giá theo phân loại |
| Small | ✅ | ~2 ngày dev |
| Testable | ✅ | Giá trước/sau, trạng thái đơn đo được |

**Background** (áp dụng cho AC1-AC5):

```gherkin
Background:
  Given seller đã đăng nhập Kênh người bán của Sàn X bằng tài khoản shop "Thời trang A"
  And shop "Thời trang A" ở trạng thái "Đang hoạt động"
  And sản phẩm "Áo thun basic" của shop "Thời trang A" đang bán với giá 150.000 VND
```

**AC1: Đổi giá thành công - happy path**
- **Given** "Áo thun basic" không tham gia chương trình khuyến mãi nào đang diễn ra
- **When** seller đổi giá "Áo thun basic" thành 135.000 VND
- **Then** trang sản phẩm hiển thị giá 135.000 VND cho người mua trong vòng 5 phút
- **And** lịch sử giá của sản phẩm có thêm 1 dòng: giá cũ 150.000 VND, giá mới 135.000 VND, thời điểm đổi, tài khoản thực hiện

**AC2: Đổi giá không ảnh hưởng đơn đã đặt - tác động lên dữ liệu có sẵn**
- **Given** đơn DH0001 gồm 2 "Áo thun basic" × 150.000 VND đang ở trạng thái "Chờ lấy hàng"
- **And** giỏ hàng của người mua C đang có 1 "Áo thun basic"
- **When** seller đổi giá "Áo thun basic" thành 180.000 VND
- **Then** đơn DH0001 giữ nguyên đơn giá 150.000 VND, tiền hàng 300.000 VND
- **And** giỏ hàng của người mua C hiển thị giá 180.000 VND kèm thông báo "Giá sản phẩm đã thay đổi từ 150.000 VND thành 180.000 VND"

**AC3: Giá mới dưới mức tối thiểu - validation**
- **Given** giá bán tối thiểu trên Sàn X là 1.000 VND
- **When** seller đổi giá "Áo thun basic" thành 500 VND
- **Then** hệ thống từ chối với thông báo "Giá bán tối thiểu là 1.000 VND"
- **And** giá "Áo thun basic" giữ nguyên 150.000 VND

**AC4: Sản phẩm đang trong Flash Sale - edge case**
- **Given** "Áo thun basic" đang tham gia Flash Sale của Sàn X từ 12:00 đến 14:00 ngày 25/09/2026 với giá 99.000 VND
- **And** thời điểm hiện tại là 13:00 25/09/2026
- **When** seller đổi giá "Áo thun basic" thành 140.000 VND
- **Then** hệ thống từ chối với thông báo "Không thể đổi giá khi sản phẩm đang tham gia Flash Sale (kết thúc lúc 14:00)"
- **And** giá gốc giữ nguyên 150.000 VND, giá Flash Sale giữ nguyên 99.000 VND

**AC5: Không sửa được sản phẩm của shop khác - phân quyền âm**
- **Given** sản phẩm "Nồi chiên 5L" thuộc shop "Gia dụng B", giá 1.290.000 VND
- **When** seller shop "Thời trang A" đổi giá "Nồi chiên 5L" thành 990.000 VND
- **Then** hệ thống từ chối với thông báo "Bạn không có quyền chỉnh sửa sản phẩm này"
- **And** giá "Nồi chiên 5L" giữ nguyên 1.290.000 VND

**Notes:**
- **Dependencies:** Story đăng sản phẩm; lịch Flash Sale do Sàn X quản lý
- **Assumptions:** Đơn đã đặt giữ giá tại thời điểm đặt; giỏ hàng lấy giá mới nhất
- **Open Questions:** Có giới hạn biên độ tăng giá (vd: > 50%) để chống "tăng giá rồi giảm ảo" trước đợt sale không? Nhân viên shop (tài khoản phụ) có được đổi giá không?

**Readiness (DoR):** ✅ Sẵn sàng

---

## Ví dụ 4: Đổi trả hàng - demo split story

### ❌ Story gốc (chưa đạt INVEST)

> **As a** người mua trên Sàn X
> **I want to** quản lý đổi trả và hoàn tiền đơn hàng
> **So that** tôi có thể đổi trả và hoàn tiền

Vấn đề: persona chung chung; goal "quản lý" là story mở, chứa "và"; So that lặp lại goal; gộp 3 vai trò (người mua, seller, CSKH Sàn X) cùng 2 cách hoàn tiền (online, COD); dev ước lượng > 10 ngày.

### Đề xuất split: Story gốc "Đổi trả hàng"
Chẩn đoán: **Compound** - nghiệp vụ đổi trả đã hiểu rõ, chỉ là gộp nhiều bước workflow và nhiều vai trò
Pattern áp dụng: **Bước workflow** + **Persona** + **Cách thay thế** (phương thức hoàn tiền)

| Story con | Giá trị riêng | Ưu tiên gợi ý |
|-----------|---------------|---------------|
| US-RETURN-001: Người mua gửi yêu cầu trả hàng cho đơn đã giao | Người mua có kênh chính thức báo lỗi, CSKH có dữ liệu để xử lý | Must |
| US-RETURN-002: Seller phản hồi yêu cầu trả hàng | Seller tự xử lý trong 48 giờ, giảm tải CSKH | Must |
| US-RETURN-003: Hoàn tiền về phương thức thanh toán online ban đầu | Người mua nhận lại tiền không cần thao tác thêm | Must |
| US-RETURN-004: Hoàn tiền đơn COD vào ví Sàn X | Mở rộng đổi trả cho đơn COD | Should |
| US-RETURN-005: Người mua khiếu nại lên Sàn X khi seller từ chối | Có trọng tài khi hai bên bất đồng | Should |

Story nào giao được trước mà vẫn có giá trị: **US-RETURN-001** (giai đoạn đầu CSKH xử lý thủ công các bước sau).

### US-RETURN-001: Người mua gửi yêu cầu trả hàng cho đơn đã giao

**As a** người mua có đơn hàng thanh toán online ở trạng thái "Đã giao" chưa quá 7 ngày
**I want to** gửi yêu cầu trả hàng kèm lý do cùng ảnh bằng chứng cho sản phẩm trong đơn
**So that** có căn cứ để được hoàn tiền mà không phải tự thương lượng qua tin nhắn với shop

**Metadata:** Epic: Đổi trả hàng · Priority: Must · Estimate: ~3 ngày

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Không cần US-RETURN-002..005; CSKH xử lý thủ công phần sau |
| Negotiable | ✅ | Danh sách lý do trả hàng, số ảnh tối đa còn thảo luận |
| Valuable | ✅ | Người mua có kênh chính thức; sàn có dữ liệu tranh chấp |
| Estimable | ✅ | Phạm vi: 1 đơn, 1 yêu cầu, chưa gồm hoàn tiền |
| Small | ✅ | ~3 ngày dev |
| Testable | ✅ | Mốc thời hạn, trạng thái yêu cầu, thông báo cụ thể |

**Background** (áp dụng cho AC1-AC4):

```gherkin
Background:
  Given người mua đã đăng nhập Sàn X
  And đơn DH0002 của người mua chuyển sang "Đã giao" lúc 10:00 20/09/2026
  And đơn DH0002 gồm 1 "Quần jean slim" giá 450.000 VND, đã thanh toán online
```

**AC1: Gửi yêu cầu trả hàng thành công - happy path**
- **Given** thời điểm hiện tại là 09:00 25/09/2026
- **When** người mua gửi yêu cầu trả hàng cho DH0002 với lý do "Giao sai kích cỡ" kèm 2 ảnh sản phẩm
- **Then** hệ thống tạo yêu cầu TH0001 ở trạng thái "Chờ seller phản hồi"
- **And** đơn DH0002 chuyển sang trạng thái "Đang xử lý trả hàng"
- **And** người mua nhận thông báo "Đã gửi yêu cầu trả hàng TH0001. Shop sẽ phản hồi trước 09:00 27/09/2026"

**AC2: Gửi yêu cầu sau thời hạn 7 ngày - edge case**
- **Given** thời điểm hiện tại là 10:01 27/09/2026 (quá hạn 1 phút)
- **When** người mua gửi yêu cầu trả hàng cho DH0002
- **Then** hệ thống từ chối với thông báo "Đơn hàng đã quá thời hạn trả hàng 7 ngày (hết hạn lúc 10:00 27/09/2026)"
- **And** đơn DH0002 giữ trạng thái "Đã giao"

**AC3: Thiếu ảnh bằng chứng cho lý do sản phẩm lỗi - negative**
- **Given** thời điểm hiện tại là 09:00 25/09/2026
- **When** người mua gửi yêu cầu trả hàng với lý do "Sản phẩm lỗi" không kèm ảnh hay video
- **Then** hệ thống từ chối với thông báo "Vui lòng tải lên ít nhất 1 ảnh hoặc video cho lý do 'Sản phẩm lỗi'"
- **And** không tạo yêu cầu trả hàng, đơn DH0002 giữ trạng thái "Đã giao"

**AC4: Đơn đã có yêu cầu đang xử lý - negative**
- **Given** yêu cầu TH0001 của đơn DH0002 đang ở trạng thái "Chờ seller phản hồi"
- **When** người mua gửi thêm 1 yêu cầu trả hàng cho DH0002
- **Then** hệ thống từ chối với thông báo "Đơn hàng đã có yêu cầu trả hàng TH0001 đang xử lý"
- **And** đơn DH0002 vẫn chỉ có 1 yêu cầu trả hàng

**Notes:**
- **Dependencies:** Trạng thái "Đã giao" do Đơn vị vận chuyển X cập nhật (đã có); quy trình CSKH xử lý thủ công trong lúc chờ US-RETURN-002
- **Assumptions:** Thời hạn 7 ngày tính từ mốc "Đã giao"; mỗi đơn có tối đa 1 yêu cầu trả hàng đang mở
- **Open Questions:** Đơn nhiều sản phẩm có cho trả từng sản phẩm không? Lý do "Không còn nhu cầu" có được chấp nhận không, ai chịu phí vận chuyển trả hàng?

**Readiness (DoR):** ⚠️ Còn thiếu: PO chốt danh sách lý do trả hàng, bên chịu phí vận chuyển trả hàng

---

## Ví dụ 5: Đánh giá sản phẩm

### US-REVIEW-001: Đánh giá sản phẩm trong đơn đã hoàn thành

**As a** người mua có đơn hàng ở trạng thái "Hoàn thành" chưa quá 30 ngày
**I want to** chấm sao kèm nhận xét cho sản phẩm trong đơn đó
**So that** người mua sau có thông tin từ người đã thực sự mua, shop nhận phản hồi để cải thiện chất lượng

**Metadata:** Epic: Đánh giá sản phẩm · Priority: Should · Estimate: ~2 ngày

**INVEST Self-check:**

| Tiêu chí | ✅/⚠️ | Ghi chú |
|----------|------|---------|
| Independent | ✅ | Chỉ cần trạng thái đơn "Hoàn thành" đã có |
| Negotiable | ✅ | Đánh giá kèm ảnh/video, sửa đánh giá để story sau |
| Valuable | ✅ | Tăng niềm tin người mua, giảm đánh giá ảo |
| Estimable | ✅ | Phạm vi: sao + nhận xét văn bản |
| Small | ✅ | ~2 ngày dev |
| Testable | ✅ | Điểm trung bình, số đánh giá, thông báo cụ thể |

**AC1: Gửi đánh giá thành công - happy path**
- **Given** đơn DH0003 của người mua chuyển sang "Hoàn thành" lúc 08:00 23/09/2026, gồm 1 "Áo thun basic" chưa được đánh giá
- **And** "Áo thun basic" đang có 9 đánh giá, điểm trung bình 4,0
- **When** người mua gửi đánh giá 5 sao kèm nhận xét "Vải dày, đúng size"
- **Then** đánh giá xuất hiện trên trang sản phẩm với nhãn "Đã mua hàng" trong vòng 5 phút
- **And** "Áo thun basic" có 10 đánh giá, điểm trung bình 4,1

**AC2: Đánh giá sau hạn 30 ngày - edge case**
- **Given** đơn DH0004 của người mua chuyển sang "Hoàn thành" lúc 08:00 23/08/2026
- **And** thời điểm hiện tại là 08:01 22/09/2026 (quá hạn 1 phút)
- **When** người mua gửi đánh giá cho sản phẩm trong DH0004
- **Then** hệ thống từ chối với thông báo "Đã quá hạn đánh giá 30 ngày kể từ khi đơn hoàn thành (hết hạn lúc 08:00 22/09/2026)"

**AC3: Người chưa mua sản phẩm - phân quyền âm**
- **Given** người mua không có đơn "Hoàn thành" nào chứa "Áo thun basic"
- **And** "Áo thun basic" đang có 9 đánh giá
- **When** người mua gửi đánh giá 1 sao cho "Áo thun basic"
- **Then** hệ thống từ chối với thông báo "Chỉ người đã mua sản phẩm này mới có thể đánh giá"
- **And** "Áo thun basic" vẫn có 9 đánh giá

**AC4: Nhận xét chứa số điện thoại - negative**
- **Given** đơn DH0003 của người mua ở trạng thái "Hoàn thành" từ 08:00 23/09/2026, "Áo thun basic" chưa được đánh giá
- **When** người mua gửi đánh giá 4 sao kèm nhận xét "Cần mua sỉ gọi 0900 000 000"
- **Then** hệ thống từ chối với thông báo "Nhận xét không được chứa số điện thoại hoặc đường link"
- **And** "Áo thun basic" trong DH0003 vẫn ở trạng thái "Chưa đánh giá"

**Notes:**
- **Dependencies:** Trạng thái đơn "Hoàn thành" (đã có)
- **Assumptions:** Mỗi sản phẩm trong 1 đơn được đánh giá 1 lần; điểm trung bình làm tròn 1 chữ số thập phân
- **Open Questions:** Có thưởng xu cho người mua khi đánh giá không? Seller có được phản hồi công khai dưới đánh giá không (story riêng)?

**Readiness (DoR):** ✅ Sẵn sàng

---

## Patterns rút ra

1. **Tiền ghi kèm công thức**: Then nêu tổng thanh toán cùng các thành phần (tiền hàng − voucher + phí vận chuyển) để QA đối chiếu từng con số.
2. **Tồn kho là dữ liệu tranh chấp**: story nào trừ tồn kho cũng cần 1 AC race condition cho sản phẩm cuối cùng, kèm kết quả "tồn kho không xuống số âm".
3. **Khuyến mãi dùng Scenario Outline**: mỗi biên lấy 2 giá trị - ngay dưới/đúng ngưỡng đơn tối thiểu, phút cuối/phút đầu sau hạn, còn 1/còn 0 lượt.
4. **Marketplace nhiều seller cần phân quyền âm theo quyền sở hữu**: seller shop A không sửa được sản phẩm shop B; người chưa mua không đánh giá được.
5. **Đổi dữ liệu gốc phải có AC tác động lên dữ liệu có sẵn**: đơn đã đặt giữ giá chốt, giỏ hàng nhận giá mới kèm thông báo.
6. **Luồng nhiều vai trò (đổi trả, hoàn tiền) split theo bước workflow + persona**: giao story người mua gửi yêu cầu trước, bước sau tạm xử lý thủ công.

---

*Tài liệu bổ sung cho skill user-story-ac-writer (bản gốc: Phúc NT · BA Zone · Digital School)*
