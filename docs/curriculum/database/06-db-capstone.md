# 🗃️ Phần DB6: Database Capstone (PostgreSQL + Integration)

> **Mục tiêu**: thiết kế và vận hành một hệ dữ liệu đủ thực tế để chứng minh năng lực SQL/Database.  
> **Thời lượng ước tính**: 12-20 giờ.

---

## 🎯 Mục Tiêu

- Thiết kế schema chuẩn hóa đúng mức cho bài toán thực tế.
- Viết query phục vụ nghiệp vụ và báo cáo ở mức production cơ bản.
- Thực hành transaction + isolation với tình huống cạnh tranh dữ liệu.
- Đọc `EXPLAIN ANALYZE` và tối ưu một số truy vấn trọng điểm.
- Tích hợp dữ liệu với ứng dụng C# qua truy vấn an toàn.

---

## 1. Đề Bài: Learning Platform Data Layer

Thiết kế database cho nền tảng học tập với các thực thể chính:

1. `learners`
2. `courses`
3. `modules`
4. `lessons`
5. `enrollments`
6. `lesson_progress`
7. `assessment_attempts`

Mục tiêu nghiệp vụ:

1. Theo dõi tiến độ học theo học viên/khóa.
2. Thống kê completion rate theo module.
3. Phát hiện bài học có tỷ lệ fail cao.
4. Báo cáo active learners theo tuần.

---

## 2. Deliverables Bắt Buộc

1. File schema SQL (`create_tables.sql`).
2. File seed dữ liệu đủ lớn (`seed_data.sql`, tối thiểu 5k rows tổng).
3. Bộ query nghiệp vụ (`queries.sql`).
4. Báo cáo tối ưu (`performance-notes.md`) có before/after.
5. Script rollback/reset môi trường local.

---

## 3. Yêu Cầu Thiết Kế Schema

1. Mỗi bảng có PK rõ ràng.
2. FK đầy đủ cho quan hệ chính.
3. Có `CHECK`/`UNIQUE` cho business rules quan trọng.
4. Có tối thiểu 3 indexes phục vụ truy vấn báo cáo.
5. Tên bảng/cột thống nhất convention (snake_case).

---

## 4. Bộ Query Tối Thiểu Phải Có

### 4.1 CRUD + Validation

1. Tạo enrollment mới, chống duplicate enrollment.
2. Cập nhật progress bài học có kiểm tra phạm vi (0-100).
3. Soft delete hoặc deactivate learner.

### 4.2 Reporting

1. Completion rate theo course và theo module.
2. Top 10 learners theo số giờ học trong 30 ngày.
3. Lessons có tỷ lệ fail cao nhất.
4. Rolling 7-day active learners (window function).

### 4.3 Analytical

1. Dùng `ROW_NUMBER`/`RANK` để xếp hạng.
2. Dùng `LAG`/`LEAD` để so sánh tiến độ tuần này và tuần trước.
3. Dùng CTE cho query nhiều bước.

---

## 5. Transaction Lab (Bắt Buộc)

Mô phỏng tình huống:

1. Hai session cùng cập nhật progress một học viên.
2. Kiểm tra hiện tượng mất cập nhật (lost update).
3. Áp dụng giải pháp:
   - `SELECT ... FOR UPDATE`
   - hoặc optimistic concurrency (version/timestamp).

Viết biên bản ngắn:

1. Hiện tượng quan sát được.
2. Isolation level đang dùng.
3. Cách fix và trade-off.

---

## 6. Performance Lab (Bắt Buộc)

Chọn ít nhất 2 query chậm:

1. Chạy `EXPLAIN (ANALYZE, BUFFERS)`.
2. Ghi lại execution time trước tối ưu.
3. Tối ưu bằng index/query rewrite.
4. Chạy lại và so sánh time sau tối ưu.

---

## 7. Integration Với C#

Tích hợp tối thiểu:

1. Một query read phức tạp trả về DTO.
2. Một transaction write nhiều bảng.
3. Parameterized queries để tránh SQL injection.
4. Logging thời gian query ở tầng app.

---

## 8. Rubric Chấm DB Capstone (100 điểm)

| Hạng mục | Điểm tối đa | Tiêu chí |
|----------|-------------|---------|
| Schema Design | 20 | đúng quan hệ, constraints hợp lý |
| SQL Correctness | 20 | query đúng nghiệp vụ, stable |
| Reporting Depth | 15 | có query tổng hợp và phân tích |
| Transaction Handling | 15 | hiểu và xử lý cạnh tranh dữ liệu |
| Performance Tuning | 20 | có đo đạc và cải thiện rõ |
| Integration & Safety | 10 | tích hợp C# an toàn, có logging |

---

## 9. Checklist Hoàn Thành

1. Schema chạy từ đầu không lỗi.
2. Seed script có thể chạy lặp lại (idempotent hoặc reset rõ ràng).
3. Query outputs kiểm chứng được bằng dữ liệu mẫu.
4. Có bằng chứng tối ưu bằng số liệu.
5. Có kịch bản transaction conflict và cách xử lý.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Viết 3 truy vấn báo cáo cùng một mục tiêu theo 3 cách khác nhau (`JOIN`, `EXISTS`, CTE), so sánh readability và execution time.

**BT2**: Tạo dữ liệu giả lên 100k rows cho `lesson_progress`, đo hiệu năng query top active learners trước/sau khi thêm index.

**BT3**: Mô phỏng race condition bằng 2 transaction song song và ghi lại kết quả theo từng isolation level.

---

## ❓ Câu Hỏi Kiểm Tra

1. Bạn đánh đổi gì giữa normalization và tốc độ query?
2. Vì sao index này hiệu quả cho query này?
3. Khi nào nên chọn `EXISTS` thay cho `JOIN`?
4. Isolation level nào bạn đang dùng và vì sao?
5. Nếu dữ liệu tăng 10x, query nào vỡ trước?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: Describe a database optimization you made.**  
> A: Trình bày query gốc, plan gốc, bottleneck, thay đổi đã áp dụng, và kết quả đo sau tối ưu.

> **Q: How did you handle concurrency issues?**  
> A: Nêu case cụ thể (lost update/race condition), isolation/locking bạn dùng, và lý do chọn.

> **Q: How did you integrate SQL with application code safely?**  
> A: Parameterized queries, transaction scope, timeouts, retry strategy có kiểm soát.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-db.md#db06-db-capstone](./99-answer-key-db.md#db06-db-capstone)
- Bài tập thực hành: [99-answer-key-db.md#db06-db-capstone-exercises](./99-answer-key-db.md#db06-db-capstone-exercises)
- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db06-db-capstone-deep](./97-db-theory-deep-dive.md#db06-db-capstone-deep)

