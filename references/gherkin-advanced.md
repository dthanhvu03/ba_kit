# Gherkin nâng cao - Viết AC cho scenario phức tạp

> Bổ sung cho `templates/ac-template.md`. Dùng khi AC có nhiều bộ dữ liệu,
> nhiều AC chung tiền điều kiện, hoặc cần quy tắc chặt cho từng mệnh đề.
>
> Tổng hợp từ: *Getting it Right: Defining Acceptance Criteria so there are
> No Surprises* (Midgley & Evans, IIBA Cleveland), *Writing Effective User
> Stories*, *Writing User Stories Workshop*, *Sprint Backlog Specified by
> Example* (R. Jocham) - xem `docs/DOCUMENTS/AGILE/SCRUM/`.

---

## 1. Quy tắc cho từng mệnh đề

### GIVEN - bối cảnh ban đầu
| NÊN | KHÔNG NÊN |
|-----|-----------|
| Thể hiện ý định nghiệp vụ | Mô tả thao tác kỹ thuật / việc của dev |
| Chỉ nêu bối cảnh **cần thiết** cho scenario | Kể thừa bối cảnh không liên quan |
| Là **trạng thái có sẵn** (đã, đang, còn...) | Là một **hành động** ("user bấm...") |
| Dùng `And` khi có nhiều hơn 1 tiền điều kiện | |

### WHEN - sự kiện xảy ra
| NÊN | KHÔNG NÊN |
|-----|-----------|
| Mô tả **cái gì** xảy ra | Mô tả **cách làm** (bấm nút A → chọn B → nhập C) |
| **Đúng 1 hành động / sự kiện** | Dùng `And` sau When |
| Chính là hành động đang được kiểm tra | |

❌ `When khách hàng bấm "Mua" → chọn ví → nhập PIN`
✅ `When khách hàng xác nhận thanh toán bằng ví`

### THEN - kết quả quan sát được
| NÊN | KHÔNG NÊN |
|-----|-----------|
| Mô tả hệ thống **phải làm gì** | Mô tả việc người dùng làm |
| Mô tả **kết quả nghiệp vụ** | Mô tả chi tiết implementation |
| Chỉ kiểm tra kết quả **liên quan tới hành động ở When** | |
| Dùng `And` khi có nhiều kết quả quan sát được | |

### SCENARIO - toàn bộ AC
| NÊN | KHÔNG NÊN |
|-----|-----------|
| Có **ví dụ cụ thể** (số tiền, số lượng, trạng thái) | Chép lại nguyên văn business rule |
| Nói hệ thống làm gì | Hướng dẫn cách dùng hệ thống |
| Mô tả chức năng nghiệp vụ | Mô tả thiết kế phần mềm |

---

## 2. Background - tiền điều kiện dùng chung

Khi **≥ 3 AC** lặp lại cùng một Given, gom vào `Background` đặt trước các AC.

```gherkin
Background:
  Given khách hàng đã đăng nhập
  And tài khoản ở trạng thái "Active"

AC1: Chuyển khoản thành công
  Given số dư là 5.000.000 VND
  When khách hàng chuyển 1.000.000 VND tới tài khoản hợp lệ
  Then số dư còn 4.000.000 VND
  And giao dịch có trạng thái "Thành công"

AC2: Số dư không đủ
  Given số dư là 500.000 VND
  When khách hàng chuyển 1.000.000 VND
  Then hệ thống từ chối với thông báo "Số dư không đủ"
  And số dư không thay đổi
```

Chỉ đưa vào Background những gì **mọi** AC trong story đều cần.

---

## 3. Scenario Outline + Examples - nhiều bộ dữ liệu, 1 logic

Khi nhiều AC có **cùng cấu trúc**, chỉ khác dữ liệu (thường gặp: ngưỡng,
biên giá trị, bảng phí, phân quyền), dùng Scenario Outline thay vì viết
lặp nhiều AC.

```gherkin
Scenario Outline: Tính phí chuyển khoản theo số tiền
  Given khách hàng thuộc hạng <hang>
  When khách hàng chuyển <so_tien> VND
  Then phí giao dịch là <phi> VND

  Examples:
    | hang     | so_tien     | phi    |
    | Standard | 499.999     | 0      |
    | Standard | 500.000     | 5.500  |
    | Gold     | 500.000     | 0      |
    | Standard | 0           | từ chối: "Số tiền phải lớn hơn 0" |
```

**Mẹo chọn dòng Examples** (Specification by Example):
- Mỗi **biên** lấy 2 giá trị: ngay dưới và đúng ngưỡng (499.999 / 500.000)
- Thêm giá trị **không hợp lệ**: 0, số âm, quá giới hạn
- Mỗi dòng phải có **kết quả mong đợi cụ thể**, kể cả dòng lỗi

Một Scenario Outline được tính là **1 AC**, không làm story bị tính là "quá nhiều AC".

---

## 4. Data table - nhiều điều kiện trong 1 bước

Khi Given có nhiều thuộc tính của cùng một đối tượng, dùng bảng thay vì
nhiều dòng `And`:

```gherkin
Given đơn hàng có thông tin sau:
  | sản phẩm | số lượng | đơn giá  |
  | Áo thun  | 2        | 150.000  |
  | Quần jean| 1        | 450.000  |
When khách hàng áp mã giảm giá "SALE10"
Then tổng thanh toán là 675.000 VND
```

---

## 5. AC nên có và KHÔNG nên có

**AC nên bao quát:**
- Kịch bản negative (lỗi, vi phạm rule, không đủ quyền)
- Yêu cầu phi chức năng **riêng của story này** (vd: "kết quả trả về trong ≤ 2 giây")
- Ảnh hưởng của story tới tính năng khác (nếu có)
- Vai trò nào được / không được thực hiện hành động

**AC KHÔNG chứa các mục thuộc Definition of Done** (áp dụng cho mọi story):
- ❌ "Code đã được review"
- ❌ "Đã chạy unit test / regression test"
- ❌ "Đã deploy lên môi trường staging"
- ❌ "Không còn bug nghiêm trọng"

→ Những mục này đưa vào DoD của team (xem `checklists/definition-of-ready-done.md`).

**Tránh từ định lượng tuyệt đối** trong AC nếu không kiểm chứng được:
`tất cả`, `bất kỳ`, `mọi`, `luôn luôn`, `không bao giờ`, `v.v.`
→ Thay bằng danh sách/giới hạn cụ thể.

---

## 6. Mức độ chi tiết phù hợp

Không có công thức cố định. Checklist tự hỏi:
1. Team (dev, QA) đã thảo luận story này chưa?
2. Đã hiểu đúng story card (As a / I want / So that) chưa?
3. Có đủ cả scenario **positive** và **negative**?
4. Có scenario **alternate** (cách khác đạt cùng kết quả) và **exception**?
5. Chi tiết nào không thuộc AC (UI spec, business rule đầy đủ, NFR chung)
   → đưa sang tài liệu khác và **link tới**, không nhồi vào AC.
6. Có 1 ví dụ cụ thể với nhiều giá trị (Scenario Outline) cho phần tính toán?

---
*Tài liệu bổ sung cho skill user-story-ac-writer (bản gốc: Phúc NT · BA Zone · Digital School)*
