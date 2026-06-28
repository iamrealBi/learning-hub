# 📙 DB Answer Key (Câu Hỏi + Bài Tập)

> Tài liệu đáp án gợi ý cho DB track. Tập trung vào tư duy query đúng và thiết kế schema hợp lý.

---

## Cách Dùng

1. Tự viết SQL trước, chạy trên dữ liệu mẫu.
2. So output với expected result.
3. Sau đó mới đối chiếu đáp án để tối ưu.

---

<a id="db01-relational-concepts"></a>
## DB01 Relational Concepts - Đáp Án Câu Hỏi

1. Dữ liệu thô chưa có ngữ cảnh; thông tin là dữ liệu đã xử lý có ý nghĩa.
2. RDBMS tổ chức dữ liệu thành bảng với quan hệ qua keys.
3. ACID: Atomicity, Consistency, Isolation, Durability.
4. ERD mô tả entity, attributes, relationships và cardinality.
5. SQL (quan hệ) mạnh consistency; NoSQL linh hoạt schema và scale theo kiểu dữ liệu phù hợp.

<a id="db01-relational-concepts-exercises"></a>
### DB01 Relational Concepts - Đáp Án Bài Tập

- Vẽ ERD trước khi viết SQL schema.
- Xác định PK/FK ngay từ đầu để tránh orphan rows.
- Với quan hệ M:N, luôn cần bảng trung gian.

---

<a id="db02-sql-fundamentals"></a>
## DB02 SQL Fundamentals - Đáp Án Câu Hỏi

1. DDL: tạo/sửa cấu trúc (`CREATE/ALTER/DROP`); DML: thao tác dữ liệu (`SELECT/INSERT/UPDATE/DELETE`).
2. `WHERE` lọc trước khi group; `HAVING` lọc sau `GROUP BY`.
3. `COUNT(*)` đếm mọi dòng; `COUNT(col)` bỏ qua `NULL`.
4. `TRUNCATE` xóa toàn bộ nhanh; `DELETE` xóa có điều kiện.
5. Thứ tự thực thi điển hình: `FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT`.

<a id="db02-sql-fundamentals-exercises"></a>
### DB02 SQL Fundamentals - Đáp Án Bài Tập

- Với báo cáo grade band/status band, dùng `CASE WHEN`.
- Phân trang: `ORDER BY` bắt buộc trước `LIMIT/OFFSET`.
- Khi tính trung bình/tổng, xác định rõ có loại trừ `NULL` hay không.

---

<a id="db03-database-design"></a>
## DB03 Database Design - Đáp Án Câu Hỏi

1. PK định danh duy nhất; FK đảm bảo toàn vẹn tham chiếu.
2. 1NF: cột atomic; 2NF: không phụ thuộc một phần vào PK ghép; 3NF: không phụ thuộc bắc cầu.
3. Denormalization dùng khi cần tốc độ đọc và đã có bài toán hiệu năng thực tế.
4. Index tăng tốc đọc nhưng làm write chậm hơn và tốn storage.

<a id="db03-database-design-exercises"></a>
### DB03 Database Design - Đáp Án Bài Tập

- Thiết kế bắt đầu từ use cases query quan trọng.
- Dùng `UNIQUE` cho natural key cần chống trùng (email, code...).
- Chỉ thêm index sau khi xác định pattern truy vấn chính.

---

<a id="db04-joins-subqueries"></a>
## DB04 JOINs & Subqueries - Đáp Án Câu Hỏi

1. `INNER JOIN` lấy phần giao; `LEFT JOIN` giữ toàn bộ bảng trái.
2. `EXISTS` thường hiệu quả cho kiểm tra tồn tại hơn `IN` ở một số pattern lớn.
3. Correlated subquery chạy phụ thuộc từng dòng ngoài.
4. CTE giúp query nhiều bước dễ đọc và bảo trì.

<a id="db04-joins-subqueries-exercises"></a>
### DB04 JOINs & Subqueries - Đáp Án Bài Tập

- Khi bị duplicate rows sau join, kiểm tra cardinality từng quan hệ.
- Anti-join chuẩn: `LEFT JOIN ... WHERE right.id IS NULL` hoặc `NOT EXISTS`.
- Recursive CTE cần điều kiện dừng rõ để tránh vòng lặp vô hạn.

---

<a id="db05-advanced-postgresql"></a>
## DB05 Advanced PostgreSQL - Đáp Án Câu Hỏi

1. View là query ảo; materialized view lưu dữ liệu thật.
2. Trigger dùng cho rule tự động nhưng cần tránh logic quá nặng trong DB.
3. Isolation levels cân bằng giữa đúng đắn và concurrency.
4. Window functions tính toán theo tập dòng liên quan mà vẫn giữ từng dòng.
5. `EXPLAIN ANALYZE` cho plan thực thi thực tế (kèm timing).

<a id="db05-advanced-postgresql-exercises"></a>
### DB05 Advanced PostgreSQL - Đáp Án Bài Tập

- Ranking theo phòng ban: `PARTITION BY department ORDER BY salary DESC`.
- So sánh kỳ trước/kỳ này: `LAG()`/`LEAD()`.
- Tối ưu query: đo trước/sau khi thêm index hoặc rewrite condition.

---

<a id="db06-db-capstone"></a>
## DB06 DB Capstone - Đáp Án Câu Hỏi

1. Trade-off normalization vs performance phải dựa trên workload thực tế.
2. Concurrency issue cần tái hiện được (ít nhất 2 sessions) rồi mới chọn fix.
3. Query optimization phải có số liệu đo (`EXPLAIN ANALYZE`) trước/sau.
4. Tích hợp C# cần parameterized query để chống SQL injection.

<a id="db06-db-capstone-exercises"></a>
### DB06 DB Capstone - Đáp Án Bài Tập

- Viết 3 biến thể query (`JOIN`, `EXISTS`, CTE`) rồi đo cost/time để chọn.
- Tạo dataset lớn để kiểm thử index thực sự có hiệu lực.
- Ghi lại runbook reset DB để tái lập môi trường nhanh.

---

## Ghi Chú

1. Đáp án SQL tốt luôn đi cùng dữ liệu test và output kiểm chứng.
2. Nếu query khác đáp án nhưng đúng kết quả, rõ ràng và nhanh hơn, vẫn là đáp án tốt.
