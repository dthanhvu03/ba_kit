# Story Smells & Guidelines - Dùng cho Mode B (Refine / Review)

> Khi user nhờ review/refine US có sẵn, đối chiếu với danh mục này để chỉ
> ra lỗi cụ thể theo tên, kèm cách sửa.
>
> Tổng hợp từ: Mike Cohn - *User Stories Applied* (Ch.2, 3, 4, 6, 7, 12, 14, 16),
> *Everything You Wanted to Know About User Stories*, *Story Mapping Playbook*,
> *Writing Effective User Stories* - xem `docs/DOCUMENTS/AGILE/SCRUM/`.

---

## 1. Danh mục Story Smells

| Smell | Triệu chứng | Cách sửa |
|-------|-------------|----------|
| **Persona chung chung** | "As a user", "As a customer" | Chỉ rõ vai trò + trạng thái ("khách hàng đã xác thực eKYC") |
| **Viết cho số nhiều** | "Khách hàng có thể xóa hồ sơ" | Viết cho **1 người**: "Khách hàng có thể xóa hồ sơ **của mình**" (lộ ra quyền sở hữu dữ liệu) |
| **Câu bị động** | "Hồ sơ được tải lên bởi..." | Chủ động: "Ứng viên tải lên hồ sơ" |
| **Story kỹ thuật** | "As a developer, I want connection pool..." | Viết lại theo lợi ích người dùng: "Tối đa 50 người dùng đồng thời với license 5 user" |
| **"The system shall..."** | Danh sách yêu cầu kiểu SRS | Chuyển về mục tiêu người dùng; hỏi "Ai dùng, dùng thế nào, vì sao?" |
| **Chi tiết UI quá sớm** | "Trên trang Chi tiết công việc, bấm nút..." | "Khi xem chi tiết một công việc, ứng viên có thể..." |
| **Quá nhiều chi tiết** | Story dài hơn cuộc trao đổi; spec viết trước quá xa | Giữ story ngắn, chi tiết đưa vào AC hoặc Notes/Open Questions |
| **Goldplating** | Thêm tính năng không ai yêu cầu | Tách thành story mới để PO xếp ưu tiên |
| **Story phụ thuộc lẫn nhau** | Chỉ làm được khi kéo kèm story khác | Gộp, hoặc cắt lại theo lát dọc (xem `splitting-patterns.md`) |
| **Story quá nhỏ** | Estimate thay đổi tùy thứ tự làm | Gộp để lập kế hoạch |
| **Story "mở"** | "Quản lý...", không có điểm kết thúc | Tách thành các story "đóng" có mục tiêu hoàn thành rõ |
| **So that khó viết** | Không nói được vì sao cần | Story cần nghĩ lại, có thể không đáng làm |
| **AC = DoD** | AC chứa "đã review code", "đã test" | Chuyển sang Definition of Done |
| **PO không xếp ưu tiên được** | Story quá to hoặc giá trị không rõ | Làm nhỏ lại; viết lại cho rõ giá trị |

---

## 2. Guidelines khi viết story (Cohn)

1. **Bắt đầu từ goal story**: với mỗi vai trò, liệt kê các mục tiêu lớn rồi mới tách nhỏ.
2. **Kích thước theo tầm nhìn**: story cho 1-2 sprint tới phải nhỏ; story xa hơn giữ ở dạng Epic.
3. **Không phải thứ gì cũng là story**: UI guideline, thỏa thuận interface giữa hệ thống,
   data dictionary → dùng tài liệu phù hợp, link từ story.
4. **Story là lời hẹn trò chuyện**: mặt trước ghi story + câu hỏi mở; chi tiết đã
   thống nhất thành AC. Viết quá chi tiết tạo "độ chính xác giả".
5. **Ràng buộc (Constraint)**: yêu cầu phi chức năng áp dụng lâu dài (hiệu năng,
   dung lượng, độ chính xác) → ghi riêng nhãn *Constraint*, phải đo được
   ("≤ 2 giây cho 95% request", không phải "không phải chờ lâu").

---

## 3. 4 câu hỏi tìm AC còn thiếu (Cohn, Ch.6)

1. Dev còn cần biết thêm điều gì?
2. Mình đang ngầm giả định gì về cách làm?
3. Có trường hợp nào hệ thống phải hành xử khác đi không?
4. Điều gì có thể sai?

Thêm các mẫu AC hay bị bỏ sót:
- **Phân quyền âm**: "người không phải Admin không thể thêm sản phẩm"
- **Dữ liệu bắt buộc** bị thiếu thì hiển thị/ẩn gì
- **Tác động lên dữ liệu có sẵn**: "đổi giá không ảnh hưởng đơn đã đặt"

Dừng thêm AC khi AC mới **không làm rõ thêm ý định**; không lặp các case tương
đương (thẻ Visa hết hạn vs MasterCard hết hạn); chi tiết vụn (ngày không hợp lệ)
để QA/unit test lo.

---

## 4. Câu hỏi làm rõ persona (Cohn, Ch.3-4)

Khi persona chưa rõ, hỏi user về:
- Tần suất sử dụng, hiểu biết nghiệp vụ, mức thành thạo công nghệ
- Mục tiêu: tiện nhanh hay trải nghiệm đầy đủ
- Người dùng này **tiếp theo sẽ làm gì**? Có thể **nhầm** ở đâu? Điều gì làm họ **bối rối**?
  Họ cần **thêm thông tin** gì?

Vai trò luôn là **1 người** (không phải "công ty"). Dùng câu hỏi mở, không
gợi ý đáp án ("Cần hiệu năng thế nào?" thay vì "Tìm kiếm có cần nhanh không?").

---

## 5. Story cho bug

- Viết dạng: "As a <vai trò>, I want to <làm X> **without** <lỗi Y>"
- Kèm bước tái hiện, kết quả mong đợi, kết quả thực tế → chuyển thành AC
- Nhiều bug nhỏ cùng khu vực → gộp 1 story

---
*Tài liệu bổ sung cho skill user-story-ac-writer (bản gốc: Phúc NT · BA Zone · Digital School)*
