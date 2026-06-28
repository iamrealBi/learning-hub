# 📕 Phần 3: Database Design

> **Nguồn**: IBM Module 2 + UMich Module 3-4 + bổ sung kiến thức  
> **Thời lượng ước tính**: 5 giờ

---

## 🎯 Mục Tiêu

- Thiết kế database từ yêu cầu thực tế
- Hiểu và áp dụng Normalization (1NF → BCNF)
- Phân biệt Primary Key, Foreign Key, Composite Key, Surrogate Key
- Thiết kế và implement One-to-Many relationships
- Thiết kế và implement Many-to-Many với Junction Table
- Biết khi nào nên denormalize

---

## 1. Database Design Process

```
Bước 1: Thu thập yêu cầu (Requirements)
    → Hệ thống cần quản lý gì?
    → Dữ liệu nào cần lưu?
    
Bước 2: Thiết kế khái niệm (Conceptual Design)
    → Vẽ ERD: entities, attributes, relationships
    
Bước 3: Thiết kế logic (Logical Design)
    → Chuyển ERD → tables, columns, keys
    → Normalization
    
Bước 4: Thiết kế vật lý (Physical Design)
    → Chọn data types, indexes
    → Viết SQL CREATE TABLE
    
Bước 5: Implement
    → Tạo database, tables
    → Insert sample data
    → Test queries
```

---

## 2. Keys Chi Tiết

### 2.1 Primary Key (PK) — Khóa chính

```sql
-- Natural Key: dùng dữ liệu thật
CREATE TABLE countries (
    country_code CHAR(2) PRIMARY KEY,  -- 'VN', 'US', 'JP'
    name VARCHAR(100) NOT NULL
);

-- Surrogate Key: ID tự tạo (RECOMMENDED) ⭐
CREATE TABLE users (
    id SERIAL PRIMARY KEY,      -- Auto-increment
    email VARCHAR(100) UNIQUE,
    name VARCHAR(100)
);

-- So sánh:
-- Natural Key: có ý nghĩa, nhưng có thể thay đổi
-- Surrogate Key: vô nghĩa, nhưng ỔN ĐỊNH, nhỏ gọn, nhanh hơn
```

### 2.2 Foreign Key (FK) — Khóa ngoại

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    order_date DATE DEFAULT CURRENT_DATE,
    total NUMERIC(10,2),
    
    -- FK: user_id tham chiếu users.id
    FOREIGN KEY (user_id) REFERENCES users(id)
        ON DELETE CASCADE      -- Xóa user → xóa orders
        ON UPDATE CASCADE      -- Đổi user.id → đổi user_id
);

-- ON DELETE options:
-- CASCADE:    Xóa parent → xóa children  (1:N phổ biến)
-- SET NULL:   Xóa parent → children.FK = NULL
-- SET DEFAULT: Xóa parent → children.FK = default value
-- RESTRICT:   KHÔNG cho xóa parent nếu còn children (an toàn nhất)
-- NO ACTION:  Tương tự RESTRICT (default)
```

### 2.3 Composite Key — Khóa kết hợp

```sql
-- Khi 1 cột KHÔNG đủ unique → kết hợp nhiều cột
CREATE TABLE student_courses (
    student_id INT REFERENCES students(id),
    course_id INT REFERENCES courses(id),
    grade CHAR(2),
    enrolled_at DATE DEFAULT CURRENT_DATE,
    
    PRIMARY KEY (student_id, course_id)  -- Composite PK
    -- 1 student + 1 course = unique
);
```

### 2.4 Indexes — Chỉ mục

```sql
-- Index giúp tìm kiếm NHANH hơn (giống mục lục sách)

-- Tạo index
CREATE INDEX idx_employees_email ON employees(email);
CREATE INDEX idx_employees_dept ON employees(department);

-- Composite index (tìm theo nhiều cột)
CREATE INDEX idx_emp_dept_salary ON employees(department, salary);

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Khi nào tạo Index?
-- ✅ Cột thường dùng trong WHERE, JOIN, ORDER BY
-- ✅ Cột có nhiều giá trị DISTINCT
-- ✅ Foreign Key columns
-- ❌ Bảng nhỏ (< 1000 rows)
-- ❌ Cột thường INSERT/UPDATE (index chậm writes)
-- ❌ Cột có ít giá trị distinct (boolean, status)

-- Xem indexes
\di
-- Hoặc:
SELECT * FROM pg_indexes WHERE tablename = 'employees';
```

---

## 3. Normalization

### 3.1 Tại sao cần Normalization?

```
TRƯỚC normalization (flat table):
┌────┬──────┬───────────┬─────────┬──────────┬────────────────┐
│ id │ name │ course    │ teacher │ teacher  │ teacher_email  │
│    │      │           │         │ phone    │                │
├────┼──────┼───────────┼─────────┼──────────┼────────────────┤
│ 1  │ An   │ Math,CS   │ Thầy A  │ 090xxx   │ a@school.com   │
│ 2  │ Bình │ Math      │ Thầy A  │ 090xxx   │ a@school.com   │
│ 3  │ An   │ English   │ Cô B    │ 091xxx   │ b@school.com   │
└────┴──────┴───────────┴─────────┴──────────┴────────────────┘

Vấn đề:
❌ Redundancy: Thầy A info lặp 2 lần
❌ Update anomaly: Đổi phone Thầy A → phải sửa NHIỀU rows
❌ Delete anomaly: Xóa student cuối → mất luôn teacher info
❌ Insert anomaly: Thêm teacher mới → phải có student
```

### 3.2 First Normal Form (1NF)

```
Quy tắc:
1. Mỗi ô chỉ chứa 1 giá trị (atomic)
2. Mỗi row là duy nhất (có PK)
3. Không có repeating groups
```

```sql
-- ❌ Vi phạm 1NF: 1 ô chứa nhiều giá trị
-- courses = "Math,CS" → KHÔNG atomic!

-- ✅ 1NF: Tách thành nhiều rows
CREATE TABLE student_courses (
    id SERIAL PRIMARY KEY,
    student_name VARCHAR(100),
    course VARCHAR(100),        -- 1 giá trị/ô
    teacher VARCHAR(100),
    teacher_phone VARCHAR(20),
    teacher_email VARCHAR(100)
);
-- An + Math = 1 row, An + CS = 1 row
```

### 3.3 Second Normal Form (2NF)

```
Quy tắc:
1. Đã đạt 1NF
2. Mọi non-key column phải phụ thuộc vào TOÀN BỘ PK
   (loại bỏ partial dependency)
```

```sql
-- ❌ Vi phạm 2NF (composite PK: student_id + course_id):
-- student_name chỉ phụ thuộc student_id (partial dependency)

-- ✅ 2NF: Tách bảng
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE enrollments (
    student_id INT REFERENCES students(id),
    course_id INT REFERENCES courses(id),
    grade CHAR(2),
    PRIMARY KEY (student_id, course_id)
    -- Grade phụ thuộc CẢ student + course → OK!
);
```

### 3.4 Third Normal Form (3NF)

```
Quy tắc:
1. Đã đạt 2NF
2. Không có transitive dependency
   (non-key column KHÔNG phụ thuộc vào non-key column khác)
```

```sql
-- ❌ Vi phạm 3NF:
-- teacher_phone phụ thuộc teacher (không phải PK)
-- teacher → teacher_phone (transitive dependency)

-- ✅ 3NF: Tách teacher thành bảng riêng
CREATE TABLE teachers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(100)
);

CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    teacher_id INT REFERENCES teachers(id)
);
```

### 3.5 Boyce-Codd Normal Form (BCNF)

```
Quy tắc:
1. Đã đạt 3NF
2. Mọi functional dependency phải có superkey bên trái
   (stricter 3NF)

Hiếm khi cần trong thực tế. 3NF thường là đủ.
```

### 3.6 Tổng Kết Normalization

```
1NF: Atomic values, no repeating groups
     → Mỗi ô chứa 1 giá trị

2NF: 1NF + No partial dependencies
     → Non-key columns phụ thuộc TOÀN BỘ PK

3NF: 2NF + No transitive dependencies
     → Non-key columns KHÔNG phụ thuộc non-key column

Trong thực tế:
✅ Luôn đạt 3NF cho database thiết kế mới
⚠️ Chỉ denormalize khi CÓ LÝ DO (performance)
```

---

## 4. Relationships

### 4.1 One-to-One (1:1)

```sql
-- Ví dụ: User ← 1:1 → Profile
-- Tách khi: bảng quá lớn, hoặc data ít khi cần cùng lúc

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL
);

CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    user_id INT UNIQUE REFERENCES users(id),  -- UNIQUE = 1:1
    full_name VARCHAR(100),
    bio TEXT,
    avatar_url VARCHAR(255)
);
```

### 4.2 One-to-Many (1:N) ⭐ Phổ biến nhất

```sql
-- 1 department có N employees
-- FK nằm ở bên "Many"

CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    dept_id INT REFERENCES departments(id),  -- FK ở bên "Many"
    salary NUMERIC(10,2)
);

-- Truy vấn: nhân viên và phòng ban
SELECT e.name, d.name AS department
FROM employees e
JOIN departments d ON e.dept_id = d.id;
```

### 4.3 Many-to-Many (M:N)

```sql
-- Students <<→>> Courses: Cần JUNCTION TABLE

CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    credits INT DEFAULT 3
);

-- JUNCTION TABLE (bảng trung gian)
CREATE TABLE enrollments (
    student_id INT REFERENCES students(id) ON DELETE CASCADE,
    course_id INT REFERENCES courses(id) ON DELETE CASCADE,
    enrolled_at DATE DEFAULT CURRENT_DATE,
    grade DECIMAL(3,1),
    
    PRIMARY KEY (student_id, course_id)
);

-- Insert
INSERT INTO enrollments (student_id, course_id)
VALUES (1, 1), (1, 2), (2, 1), (3, 2), (3, 3);

-- Query: Student và courses
SELECT s.name, c.title, e.grade
FROM enrollments e
JOIN students s ON e.student_id = s.id
JOIN courses c ON e.course_id = c.id
ORDER BY s.name;

-- Đếm courses per student
SELECT s.name, COUNT(c.id) AS course_count
FROM students s
LEFT JOIN enrollments e ON s.id = e.student_id
LEFT JOIN courses c ON e.course_id = c.id
GROUP BY s.name;
```

---

## 5. Denormalization

### Khi nào denormalize?

```
Normalization:
  ✅ Giảm redundancy
  ✅ Data consistency
  ❌ Nhiều JOINs → chậm hơn

Denormalization:
  ✅ Read queries NHANH hơn (ít JOINs)
  ❌ Data có thể inconsistent
  ❌ Storage tăng

Denormalize khi:
1. Read >> Write (analytics, reporting)
2. Performance là ưu tiên #1
3. Data ít thay đổi
4. Có caching layer

Ví dụ thực tế:
- Lưu total_amount trực tiếp trong orders (thay vì tính từ order_items)
- Lưu product_name trong order_items (snapshot lúc mua)
- Lưu comment_count trong posts (thay vì COUNT mỗi query)
```

```sql
-- Normalized: phải tính mỗi query
SELECT p.title, COUNT(c.id) AS comment_count
FROM posts p
LEFT JOIN comments c ON p.id = c.post_id
GROUP BY p.id;

-- Denormalized: đọc trực tiếp
-- (Cập nhật bằng trigger hoặc application logic)
SELECT title, comment_count FROM posts;
```

---

## 6. Complete Design Example — E-Commerce

### 6.1 Requirements

```
1. Quản lý khách hàng (customers): tên, email, địa chỉ
2. Sản phẩm (products): tên, giá, danh mục, tồn kho
3. Đơn hàng (orders): khách hàng, ngày, tổng tiền, trạng thái
4. Chi tiết đơn hàng (order_items): sản phẩm, số lượng, đơn giá
5. Danh mục (categories): phân loại sản phẩm
6. Tags M:N products
```

### 6.2 ERD

```
┌───────────────┐                    ┌───────────────┐
│  Categories   │ 1:N                │    Tags       │
│───────────────│←───────┐           │───────────────│
│ PK: id        │        │           │ PK: id        │
│ name          │        │           │ name          │
└───────────────┘        │           └───────┬───────┘
                         │                   │ M:N
                  ┌──────┴────────┐   ┌──────┴───────┐
                  │   Products    │───│ Product_Tags  │
                  │──────────────│   │──────────────│
                  │ PK: id       │   │ FK: prod_id  │
                  │ name         │   │ FK: tag_id   │
                  │ price        │   └──────────────┘
                  │ FK: cat_id   │
                  │ stock        │
                  └──────┬───────┘
                         │ N:1
┌───────────────┐ 1:N  ┌──────────────┐
│  Customers    │─────>│   Orders     │ 1:N  ┌──────────────┐
│───────────────│      │──────────────│─────>│ Order_Items   │
│ PK: id        │      │ PK: id       │      │──────────────│
│ name          │      │ FK: cust_id  │      │ FK: order_id │
│ email         │      │ order_date   │      │ FK: prod_id  │
│ address       │      │ status       │      │ quantity     │
└───────────────┘      │ total        │      │ unit_price   │
                       └──────────────┘      └──────────────┘
```

### 6.3 SQL Implementation

```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    price NUMERIC(10,2) NOT NULL CHECK (price > 0),
    category_id INT REFERENCES categories(id),
    stock INT DEFAULT 0 CHECK (stock >= 0),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE product_tags (
    product_id INT REFERENCES products(id) ON DELETE CASCADE,
    tag_id INT REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (product_id, tag_id)
);

CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    address TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL REFERENCES customers(id),
    order_date TIMESTAMPTZ DEFAULT NOW(),
    status VARCHAR(20) DEFAULT 'pending' 
        CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    total NUMERIC(10,2)
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT NOT NULL REFERENCES products(id),
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL,
    UNIQUE (order_id, product_id)
);

-- Indexes
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_order_items_order ON order_items(order_id);
```

---

## ❓ Câu Hỏi Kiểm Tra

1. Primary Key vs Surrogate Key — ưu nhược?
2. ON DELETE CASCADE vs RESTRICT — khi nào dùng?
3. 1NF quy tắc gì? Cho ví dụ vi phạm.
4. 2NF giải quyết vấn đề gì? (partial dependency)
5. 3NF giải quyết vấn đề gì? (transitive dependency)
6. Junction table dùng khi nào? Cấu trúc?
7. Khi nào tạo Index? Khi nào KHÔNG?
8. Denormalization — ưu nhược điểm?
9. Composite Key khác Composite Index thế nào?
10. 1:1 relationship implement thế nào bằng SQL?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is normalization and what are the normal forms?**
> A: Normalization = giảm redundancy, đảm bảo consistency. 1NF: atomic values. 2NF: no partial dependencies. 3NF: no transitive dependencies. BCNF: stricter 3NF. Thực tế thiết kế đến 3NF. Denormalize vì performance.

> **Q: What is the difference between PRIMARY KEY and UNIQUE constraint?**
> A: PRIMARY KEY: 1 per table, NOT NULL, identifies row. UNIQUE: nhiều per table, cho phép NULL (1 lần), đảm bảo unique. PK = UNIQUE + NOT NULL. FK chỉ reference PK (hoặc UNIQUE).

> **Q: How do you implement a Many-to-Many relationship?**
> A: Tạo Junction Table (bridge table) với 2 FKs tham chiếu 2 bảng. Composite PK gồm cả 2 FKs. Junction table có thể có thêm attributes (grade, enrolled_at). Ví dụ: students ↔ enrollments ↔ courses.

> **Q: What are database indexes and when should you use them?**
> A: Index = data structure giúp tìm kiếm nhanh (B-tree). Tạo cho: WHERE/JOIN/ORDER BY columns, FKs. KHÔNG tạo cho: bảng nhỏ, cột ít distinct, cột write-heavy. Trade-off: nhanh reads, chậm writes.

> **Q: When would you denormalize a database?**
> A: Khi read performance quan trọng hơn write consistency. Use cases: dashboards, reports, analytics, caching. Ví dụ: lưu calculated totals, duplicated names. Trade-off: faster reads, data inconsistency risk.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Thiết kế database cho hệ thống blog: Users, Posts, Comments, Tags (M:N). Vẽ ERD, viết SQL, normalize đến 3NF.

**BT2**: Implement E-Commerce schema ở trên. Insert sample data. Viết queries: doanh thu theo tháng, top products, customers bỏ giỏ hàng.

**BT3**: Cho flat table: `orders(id, customer_name, customer_email, product_name, product_price, quantity)`. Normalize đến 3NF — tách thành bao nhiêu bảng? Vẽ ERD.

**BT4**: Thiết kế database cho app quản lý lớp học: Teachers, Students, Classes, Attendance (M:N), Grades. Tạo indexes phù hợp.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-db.md#db03-database-design](./99-answer-key-db.md#db03-database-design)
- Bài tập thực hành: [99-answer-key-db.md#db03-database-design-exercises](./99-answer-key-db.md#db03-database-design-exercises)
- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db03-database-design-deep](./97-db-theory-deep-dive.md#db03-database-design-deep)

---

## 📚 Quy Trình Thiết Kế Database — 3 Giai Đoạn

```
┌──────────────────────────────────────┐
│ 📋 GĐ 1: REQUIREMENTS ANALYSIS      │
│  → Thu thập yêu cầu từ stakeholders │
│  → Xác định objects + relationships  │
│  → Output: requirements document     │
└─────────────────┬────────────────────┘
                  ↓
┌──────────────────────────────────────┐
│ 🧩 GĐ 2: LOGICAL DESIGN            │
│  → Objects → Entities               │
│  → Characteristics → Attributes     │
│  → Normalize (1NF → 2NF → 3NF)    │
│  → KHÔNG chỉ định data types       │
└─────────────────┬────────────────────┘
                  ↓
┌──────────────────────────────────────┐
│ 🏗️ GĐ 3: PHYSICAL DESIGN           │
│  → Entities → Tables                │
│  → Attributes → Typed Columns       │
│  → PKs, FKs, Indexes, Constraints  │
│  → ERD → SQL Script!               │
└──────────────────────────────────────┘
```

### Best Practices

```
1️⃣ Normalize đến 3NF cho OLTP
2️⃣ Denormalize cho OLAP/Analytics
3️⃣ Naming conventions nhất quán
4️⃣ Tạo ERD TRƯỚC KHI viết SQL
5️⃣ SERIAL PRIMARY KEY cho auto IDs
6️⃣ Test design với sample data TRƯỚC production
7️⃣ Đầu tư thời gian cho Requirements Analysis!
```

- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db03-database-design-deep](./97-db-theory-deep-dive.md#db03-database-design-deep)

