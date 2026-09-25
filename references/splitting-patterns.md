# Split User Story - Pattern chi tiết

> Mở rộng phần "Khi nào cần split" trong `SKILL.md`.
>
> Tổng hợp từ: Mike Cohn - *User Stories Applied* (Ch.2, Ch.7),
> *User Story Splitting Cheat Sheet* (The Master Channel),
> *Scrum Basics - User Stories*, *Story Mapping Playbook*,
> *Everything You Wanted to Know About User Stories* - xem `docs/DOCUMENTS/AGILE/SCRUM/`.

---

## 1. Chẩn đoán trước khi split: Compound hay Complex?

| Loại | Dấu hiệu | Cách xử lý |
|------|----------|------------|
| **Compound** - "quá to" | Gộp nhiều story nhỏ đã hiểu rõ (CRUD, nhiều loại dữ liệu, nhiều bước) | Tách thành nhiều story chức năng, PO xếp ưu tiên từng cái |
| **Complex** - "chưa đủ hiểu" | Nhiều ẩn số kỹ thuật/nghiệp vụ, dev không estimate được | Tách **Spike** (nghiên cứu, có timebox) + story chức năng làm **sau** spike |

**3 lý do story không estimate được** (Cohn):
1. Thiếu hiểu biết nghiệp vụ → nói chuyện với khách hàng/PO, không phải split
2. Thiếu hiểu biết kỹ thuật → Spike
3. Story quá to → split

Spike: luôn có **timebox**, ưu tiên đặt ở sprint **trước** story chức năng,
có AC (câu hỏi nào phải trả lời được), và "So that" nói rõ sẽ dùng kết quả
nghiên cứu để làm gì. Nhiều spike cùng lúc = dấu hiệu chưa hiểu vấn đề.

---

## 2. Nguyên tắc vàng: cắt dọc ("slice the cake")

Mỗi story con phải đi **xuyên suốt** từ giao diện → nghiệp vụ → dữ liệu và
tự mang giá trị. **Không** cắt theo tầng kỹ thuật.

❌ Story 1: "Làm form nhập hồ sơ" / Story 2: "Lưu hồ sơ vào DB"
✅ Story 1: "Nộp hồ sơ với thông tin cơ bản" / Story 2: "Nộp hồ sơ với đầy đủ thông tin"

Mỗi story nên **"đóng"** (closed): kết thúc bằng một mục tiêu người dùng
cảm thấy đã hoàn thành. "Quản lý tin tuyển dụng" là story mở → tách thành
"xem hồ sơ ứng tuyển cho 1 tin", "đổi ngày hết hạn tin", "xóa hồ sơ không phù hợp".

---

## 3. Bảng pattern split

| # | Pattern | Khi nào dùng | Ví dụ (story gốc → story con) |
|---|---------|--------------|-------------------------------|
| 1 | **Thao tác (CRUD)** | Story chứa "quản lý" | Quản lý giá SP → Thêm giá / Xem giá / Sửa giá / Xóa giá |
| 2 | **Bước workflow** | Quy trình nhiều bước | Đăng SP mới → Nhập chi tiết / Duyệt chi tiết / Xuất bản |
| 3 | **Cách thay thế** | Nhiều cách đạt cùng kết quả | Thanh toán thẻ, Apple Pay, QR → mỗi phương thức 1 story; hoặc 1 loại thẻ trước, thêm loại khác sau |
| 4 | **Business rule** | Nhiều rule/validation phức tạp | Nhập địa chỉ → Nhập tự do trước / Gợi ý tên đường sau / Gợi ý tỉnh thành sau |
| 5 | **Dữ liệu - lõi tối thiểu** | Form nhiều nhóm thông tin | Đăng ký → Chỉ email + mật khẩu trước; thông tin cá nhân, tùy chọn marketing sau |
| 6 | **Hoãn yêu cầu phi chức năng** | NFR khó (hiệu năng, bảo mật nâng cao) | Tìm kiếm → "Tìm được" trước / "Kết quả < 500ms" sau |
| 7 | **Theo AC / path phổ biến** | AC quá nhiều nhánh | Thanh toán thẻ (path chính) trước; điểm thưởng, voucher (edge) sau |
| 8 | **Persona** | Nhiều vai trò trong 1 story | Tách theo từng vai trò |
| 9 | **Nền tảng** | Web / Mobile / API khác biệt lớn | Mỗi nền tảng 1 story |
| 10 | **Spike** | Story complex | Spike nghiên cứu + story triển khai |

Gợi ý tên story để chừa chỗ mở rộng: "Thanh toán **[bằng thẻ ghi nợ]**".

---

## 4. Đừng split quá tay

- Story quá nhỏ (vài giờ) → **gộp** lại: "nhập ngày cho từng mục" quá vụn,
  gộp thành "thêm / sửa / xóa mục".
- Các việc lặt vặt (bug nhỏ, chỉnh UI) → gộp thành 1 story có tên, cỡ nửa ngày
  đến vài ngày.
- Story con phụ thuộc lẫn nhau (chỉ làm được khi đi cùng story khác) →
  gộp lại, hoặc cắt theo hướng khác.

---

## 5. Ngưỡng kích thước

- Đặt ngưỡng cho team (vd: ≤ 3 ngày công) - story vượt ngưỡng → split, thu
  nhỏ phạm vi, hoặc loại bỏ.
- Quy tắc **1-2-3**: 1 story, tối đa 2 người, tối đa 3 ngày.
- Story cho 1-2 sprint tới phải nhỏ và rõ; story xa hơn có thể vẫn là Epic
  với estimate thô.

---

## 6. Cách trình bày khi đề xuất split

```
### Đề xuất split: US-XXX
Chẩn đoán: Compound / Complex - <lý do>
Pattern áp dụng: <tên pattern>

| Story con | Giá trị riêng | Ưu tiên gợi ý |
|-----------|---------------|---------------|
| US-XXX-a: ... | ... | Must |
| US-XXX-b: ... | ... | Should |

Story nào giao được trước mà vẫn có giá trị: US-XXX-a
```

---
*Tài liệu bổ sung cho skill user-story-ac-writer (bản gốc: Phúc NT · BA Zone · Digital School)*
