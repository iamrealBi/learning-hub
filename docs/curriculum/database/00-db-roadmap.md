# 🎯 Database & SQL — Roadmap Tổng Quan (Bản Hoàn Thiện)

> **Mục tiêu**: nắm chắc tư duy dữ liệu, viết SQL vững, thiết kế schema đúng, tối ưu query, và tích hợp được vào ứng dụng C#.  
> **Tổng thời lượng khuyến nghị**: 40-60 giờ (tối thiểu 65% là thực hành query thật).

---

## ✅ Chuẩn Đầu Ra

Sau khi hoàn thành DB track, bạn cần làm được:

1. Thiết kế schema cho bài toán thực tế, chọn keys/constraints phù hợp.
2. Viết SQL chính xác cho nghiệp vụ thật (CRUD, tổng hợp, join nhiều bảng, window functions).
3. Đọc và cải thiện query plan ở mức cơ bản bằng `EXPLAIN ANALYZE`.
4. Hiểu transaction isolation và xử lý các lỗi thường gặp khi đồng thời.
5. Tích hợp database vào C# app với truy vấn an toàn và có test dữ liệu.

---

## 📋 Lộ Trình Học

| # | Module | Nội dung chính | Tài liệu | Giờ gợi ý |
|---|--------|----------------|----------|-----------|
| 1 | Relational Concepts | Data models, ERD, ACID, kiến trúc DB | [01-relational-concepts.md](./01-relational-concepts.md) | 4-6 |
| 2 | SQL Fundamentals | DDL/DML, CRUD, aggregate, GROUP BY/HAVING | [02-sql-fundamentals.md](./02-sql-fundamentals.md) | 6-8 |
| 3 | Database Design | Keys, normalization, relationships, indexes | [03-database-design.md](./03-database-design.md) | 6-8 |
| 4 | JOINs & Subqueries | JOIN patterns, subqueries, CTE, set operations | [04-joins-subqueries.md](./04-joins-subqueries.md) | 6-8 |
| 5 | Advanced PostgreSQL | Views, functions, triggers, window functions, tuning | [05-advanced-postgresql.md](./05-advanced-postgresql.md) | 8-12 |
| 6 | DB Capstone | Thiết kế + dữ liệu mẫu + truy vấn + tối ưu + tích hợp C# | [06-db-capstone.md](./06-db-capstone.md) | 12-20 |

---

## 🧪 Quy Trình Học Mỗi Module

1. Chạy toàn bộ SQL examples bằng tay trong `psql` hoặc DBeaver.
2. Tự tạo dataset mới (ít nhất 200-1000 rows) để kiểm tra query thực tế.
3. Viết lại ít nhất 1 bài theo 2 cách (ví dụ `JOIN` vs `EXISTS`, subquery vs CTE).
4. Ghi lại sai lầm và nguyên nhân sai (NULL logic, duplicate rows, wrong joins).
5. Chốt module bằng mini-lab có output rõ ràng và kiểm chứng được.

---

## 🧱 Quality Gates

1. **Correctness Gate**: query chạy đúng output trên nhiều bộ dữ liệu.
2. **Modeling Gate**: schema có PK/FK/constraints rõ ràng, không dư thừa vô nghĩa.
3. **Performance Gate**: có dùng `EXPLAIN ANALYZE`, biết vì sao query chậm và fix được tối thiểu 1 bottleneck.
4. **Safety Gate**: biết chống SQL injection (parameterized queries) khi tích hợp C#.
5. **Explain Gate**: trình bày được vì sao chọn thiết kế này, không chỉ "chạy được".

---

## ⚠️ Các Chủ Đề Dễ Mất Điểm (Bắt Buộc Ôn Kỹ)

1. `NULL` semantics và three-valued logic (`TRUE/FALSE/UNKNOWN`).
2. Sai khác giữa `WHERE` và `HAVING`, giữa `JOIN` và `EXISTS`.
3. Cardinality explosion khi join nhiều bảng.
4. Transaction anomalies: dirty read, non-repeatable read, phantom read.
5. Index misuse: có index nhưng planner vẫn scan full table do query shape.

---

## 🔗 Tích Hợp Với C#

Sau DB5, đi theo thứ tự:

1. ADO.NET cơ bản (parameterized query, transaction).
2. EF Core cơ bản (entities, migrations, tracking).
3. Tối ưu query ở tầng app (projection, pagination, batching, N+1 avoidance).
4. Logging SQL và đo thời gian query trong môi trường dev.

---

## 🎓 Công Cụ Khuyến Nghị

1. PostgreSQL local + `psql`.
2. DBeaver hoặc pgAdmin để quan sát schema/query plan.
3. `EXPLAIN (ANALYZE, BUFFERS)` cho bài performance.
4. Script seed dữ liệu để test lặp lại được.

---

## 📎 Đáp Án Tổng Hợp

1. DB track answer key: [99-answer-key-db.md](./99-answer-key-db.md)

---

## 📎 Lý Thuyết Đọc Sâu

1. DB deep dive: [97-db-theory-deep-dive.md](./97-db-theory-deep-dive.md)

---

## 📚 Nguồn Bổ Trợ

| Khóa | Nền tảng | Ghi chú |
|------|---------|---------|
| [Introduction to RDBMS](https://www.coursera.org/learn/introduction-to-relational-databases) | IBM | Củng cố nền tảng |
| [Database Design & Basic SQL in PostgreSQL](https://www.coursera.org/learn/database-design-postgresql) | UMich | Tốt cho design + SQL cơ bản |
| [Working with Subqueries in SQL](https://www.coursera.org/projects/working-with-subqueries-in-sql) | Guided Project | Luyện subquery nhanh |

---

## 🚀 Điểm Đích

Hoàn thành DB track khi bạn tự làm xong một hệ thống dữ liệu nhỏ end-to-end:

1. Thiết kế schema hợp lý.
2. Seed dữ liệu đủ lớn để test.
3. Viết được bộ query nghiệp vụ + báo cáo.
4. Có kiểm chứng hiệu năng cơ bản.
5. Kết nối ổn định vào ứng dụng C#.
