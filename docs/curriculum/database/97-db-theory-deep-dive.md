# 📚 DB Theory Deep Dive

> Tài liệu bổ sung chiều sâu cho DB track: từ mô hình dữ liệu đến tối ưu truy vấn và vận hành thực tế.

---

## Cách Dùng

1. Học file module chính trước.
2. Dùng deep dive để hiểu trade-off và tránh lỗi thiết kế phổ biến.
3. Áp dụng ngay vào mini-lab của module đang học.

---

<a id="db01-relational-concepts-deep"></a>
## DB01 Relational Concepts - Deep Dive

### Core Mental Model

- Dữ liệu cần được mô hình theo ngữ nghĩa nghiệp vụ, không theo giao diện tạm thời.
- ERD không phải tài liệu trang trí; nó là hợp đồng giữa business và engineering.

### Common Failure

1. Thiếu định nghĩa rõ entity boundary.
2. Dùng một bảng quá lớn để chứa nhiều khái niệm.
3. Không định nghĩa cardinality nên join/report sai từ đầu.

---

<a id="db02-sql-fundamentals-deep"></a>
## DB02 SQL Fundamentals - Deep Dive

### Query Thinking

- SQL là declarative: bạn mô tả tập kết quả mong muốn.
- Đọc query theo logic engine, không theo thứ tự viết.

### Mistakes To Avoid

1. Dùng `SELECT *` trong query production/report chính.
2. Bỏ qua `NULL` semantics.
3. Dùng `OFFSET` lớn mà không có chiến lược pagination tốt.

---

<a id="db03-database-design-deep"></a>
## DB03 Database Design - Deep Dive

### Normalization Là Công Cụ, Không Là Mục Đích

- Normalize để giảm redundancy và anomaly.
- Denormalize có chủ đích khi read-heavy và có đo hiệu năng rõ.

### Design Heuristics

1. Bắt đầu từ use-cases query quan trọng.
2. Chỉ số hóa theo pattern truy vấn thật.
3. Constraints phải encode business rules tối thiểu.

---

<a id="db04-joins-subqueries-deep"></a>
## DB04 JOINs & Subqueries - Deep Dive

### Cardinality Literacy

- Sai cardinality là nguồn lỗi report phổ biến nhất.
- Trước khi join, phải biết một-một, một-nhiều, hay nhiều-nhiều.

### Practical Rules

1. Khi chỉ cần kiểm tra tồn tại, ưu tiên `EXISTS`.
2. Sau join nhiều bảng, luôn tự kiểm duplicate rows.
3. CTE giúp readability; không mặc định nhanh hơn.

---

<a id="db05-advanced-postgresql-deep"></a>
## DB05 Advanced PostgreSQL - Deep Dive

### Performance Method

1. Xác định query chậm.
2. Đo bằng `EXPLAIN ANALYZE`.
3. Chỉnh index/query shape.
4. Đo lại và lưu evidence.

### Transaction Correctness

- Dữ liệu đúng quan trọng hơn query nhanh.
- Chọn isolation level theo rủi ro nghiệp vụ, không chọn theo cảm giác.

### Operational Cautions

1. Trigger mạnh nhưng dễ tạo logic ẩn.
2. Function giúp tái dùng nhưng cần versioning discipline.

---

<a id="db06-db-capstone-deep"></a>
## DB06 DB Capstone - Deep Dive

### What Makes Capstone Strong

1. Schema phản ánh đúng domain.
2. Query phục vụ use-cases thật, không demo toy.
3. Có performance notes trước/sau.
4. Có concurrency scenario tái hiện được.

### Reviewer Questions You Must Answer

1. Vì sao bảng này cần index này?
2. Vì sao chọn `JOIN` thay vì `EXISTS` ở query này?
3. Nếu dữ liệu tăng 10x, bottleneck đầu tiên là gì?
4. Cách rollback khi migration lỗi?

---

## Kết Luận

- DB tốt không chỉ là "query đúng".
- DB tốt là query đúng + model đúng + vận hành an toàn + tối ưu có số liệu.
