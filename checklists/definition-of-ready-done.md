# Definition of Ready & Definition of Done cho User Story

> Dùng để kiểm tra story **đã sẵn sàng đưa vào sprint** (DoR) và gợi ý
> những gì **không** nên viết vào AC vì thuộc DoD.
>
> Tổng hợp từ: *Definition of Ready & Definition of Done* (J. Barjis, SAFe SPC),
> *Writing Good User Stories* (E. Worts), *Scrum Basics - User Stories*,
> *Checklist - Backlog Refinement* (ZXM) - xem `docs/DOCUMENTS/AGILE/`.

---

## 1. Phân biệt DoR - AC - DoD

| | Definition of Ready | Acceptance Criteria | Definition of Done |
|---|---|---|---|
| **Phạm vi** | Chung cho mọi story | Riêng cho **1** story | Chung cho mọi story |
| **Trả lời** | Story đủ rõ để kéo vào sprint chưa? | Làm **đúng thứ** cần làm chưa? (functionality) | Làm **đúng cách** chưa? (quality) |
| **Ai quyết** | Team | Product Owner / khách hàng | Team |
| **Ví dụ** | "Có AC kiểm chứng được" | "Then số dư giảm 1.000.000 VND" | "Code đã review, test pass" |

Story chỉ **Done** khi thỏa **cả AC lẫn DoD**. Kết quả phải nhị phân:
Done / Not Done - không có "gần xong", "99%".

---

## 2. DoR checklist cho User Story

Skill tự chạy checklist này ở cuối output (mục **Readiness**):

- [ ] Viết đúng format As a / I want to / So that
- [ ] Đạt INVEST, **không có dependency bên ngoài chưa giải quyết**
- [ ] Có AC kiểm chứng được một cách khách quan (pass/fail rõ ràng)
- [ ] Đủ nhỏ để xong trong 1 sprint (thường 2-3 ngày công)
- [ ] Không còn Open Question **chặn** việc estimate
- [ ] Đã xác định assumption và dependency (ghi trong Notes)
- [ ] Có mockup/wireframe nếu story có giao diện mới *(nếu team yêu cầu)*
- [ ] Có yêu cầu hiệu năng nếu story nhạy cảm về thời gian phản hồi *(nếu áp dụng)*

Các mục cần **team xác nhận**, skill không tự đánh dấu được:
- [ ] Đã được cả team estimate
- [ ] Đã trao đổi với cả team (không phải chỉ BA/PO biết)

**Cách trình bày trong output:**
```
### Readiness (DoR)
✅ Sẵn sàng cho refinement | ⚠️ Còn thiếu: <liệt kê> | ❌ Chưa sẵn sàng: <lý do>
```

---

## 3. DoD mẫu (tham khảo, KHÔNG viết vào AC)

- Thỏa toàn bộ Acceptance Criteria
- Code đã review
- Unit test / regression test đã cập nhật và pass
- Không còn defect mức must-fix
- Tài liệu người dùng đã cập nhật (nếu cần)
- Đã demo cho stakeholder
- Product Owner đã chấp nhận

Nếu user yêu cầu viết những mục này vào AC → giải thích chúng thuộc DoD
và đề xuất đưa vào DoD của team.

---

## 4. Anti-pattern: DoR biến thành "cửa ải Waterfall"

DoR là **hướng dẫn**, không phải cổng chặn cứng. Dấu hiệu sai:
- Team không kéo được story nào vào sprint vì "chưa đủ 100% DoR"
- DoR đòi hoàn tất thiết kế chi tiết, sign-off nhiều cấp trước khi bắt đầu
- Mọi chi tiết phải chốt trước → mất tính **Negotiable**

→ Chỉ một số mục là **bắt buộc** (AC + estimate), phần còn lại là **mong muốn**.
Khi skill đánh giá ⚠️ Readiness, nêu rõ mục nào bắt buộc, mục nào có thể
bổ sung trong sprint.

---

## 5. Backlog sâu dần (DEEP) - mức chi tiết theo độ ưu tiên

- **D**etailed appropriately - chi tiết vừa đủ, top backlog chi tiết nhất
- **E**mergent - backlog thay đổi khi có thông tin mới
- **E**stimated - item đã có estimate
- **P**rioritized - xếp hạng theo giá trị

Gợi ý phân bổ: ~20% đầu backlog là story chi tiết (đủ DoR), ~20% tiếp theo
chi tiết vừa, ~60% còn lại vẫn là Epic. **Không cần** viết AC đầy đủ cho
story ở cuối backlog - chỉ cần tiêu đề + value.

---
*Tài liệu bổ sung cho skill user-story-ac-writer (bản gốc: Phúc NT · BA Zone · Digital School)*
