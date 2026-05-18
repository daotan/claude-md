# ET Task — Estimate Time & Phân tích Requirement

## Mục tiêu
Phân tích requirement ở trên, clarify ambiguity, và đưa ra estimate chi tiết cho task.
Thực hiện đầy đủ 5 bước bên dưới ngay bây giờ, không hỏi lại user.

---

## Context mặc định của team
- **Platforms:** Magento 2.4.x | Shopify | WordPress
- **Stack:** PHP 8.1 | MySQL 8.0 | Redis 7.0
- **Frontend:** LESS/SCSS, RequireJS, KnockoutJS (Magento) | Liquid (Shopify) | Gutenberg (WP)
- **Conventions:** PSR-12 (PHP) | BEM (CSS) | Repository pattern (không dùng raw SQL)
- **Rules:** Không sửa core — override qua plugin/preference/hook
- **After deploy:** `bin/magento setup:di:compile && bin/magento cache:flush` (Magento)

---

## Các bước thực hiện

### Bước 1 — Đọc & tóm tắt requirement
Đọc toàn bộ requirement, sau đó tóm tắt lại bằng bullet points theo cấu trúc:
- **Mục tiêu chính:** [1 câu mô tả task làm gì]
- **Các tính năng cụ thể:**
  - [Feature 1]
  - [Feature 2]
- **Out of scope** (nếu xác định được): [những gì KHÔNG làm trong task này]

### Bước 2 — Liệt kê câu hỏi cần clarify
Xác định tất cả điểm còn mơ hồ, ambiguous, hoặc thiếu thông tin. Output theo format:

| # | Câu hỏi | Lý do cần clarify | Mức độ ưu tiên |
|---|---------|-------------------|----------------|
| 1 | [Câu hỏi cụ thể?] | [Ảnh hưởng gì nếu không rõ] | 🔴 Blocker / 🟡 Important / 🟢 Nice to know |

### Bước 3 — Xác định scope kỹ thuật
- **Files/Modules cần tạo mới:** [list]
- **Files/Modules cần sửa:** [list]
- **Database changes:** Migration cần không? Bảng nào?
- **Third-party integration:** Có gọi API bên ngoài không?
- **Dependencies:** Task này phụ thuộc vào task/module nào khác?

### Bước 4 — Breakdown Estimate

Estimate theo từng phần, đơn vị: **giờ (h)**

| Phần việc | Mô tả cụ thể | Estimate (h) | Ghi chú |
|-----------|-------------|:------------:|---------|
| Phân tích & design | Đọc code, lên approach, confirm với team | ? | |
| Backend | PHP, XML, DB migration | ? | |
| Frontend | Template, LESS/JS | ? | |
| Testing | Unit test, self-test | ? | |
| **Subtotal** | | **?** | |
| **Buffer 15%** | Cho unforeseen issues | **?** | |
| **Tổng ET** | | **?** | |

### Bước 5 — Nêu Risk kỹ thuật
Liệt kê các risk có thể xảy ra và mức độ ảnh hưởng:

| Risk | Khả năng xảy ra | Ảnh hưởng | Cách giảm thiểu |
|------|:--------------:|:---------:|-----------------|
| [Mô tả risk] | Cao / Trung / Thấp | Cao / Trung / Thấp | [Giải pháp dự phòng] |

---

## Output cuối cùng
Sau khi hoàn thành 5 bước, đưa ra tóm tắt:

```
TỔNG KẾT ET
════════════════════════════════
Task: [Tên task]
Platform: [Magento / WordPress]

Câu hỏi cần clarify: [X] câu — cần confirm trước khi bắt đầu
Scope: [X] files cần tạo | [X] files cần sửa

Estimate:
  Subtotal : X giờ
  Buffer   : X giờ (15%)
  Tổng ET  : X giờ (~X ngày làm việc)

Risk chính: [1-2 dòng tóm tắt risk quan trọng nhất]
════════════════════════════════
```
