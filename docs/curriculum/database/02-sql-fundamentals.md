# 📗 Phần 2: SQL Fundamentals

> **Nguồn**: IBM Module 2 + UMich Module 1-2 + bổ sung kiến thức  
> **Thời lượng ước tính**: 6 giờ

---

## 🎯 Mục Tiêu

- Phân biệt DDL vs DML
- Viết thành thạo CRUD (Create, Read, Update, Delete)
- Sử dụng WHERE, ORDER BY, LIMIT, OFFSET
- Dùng Aggregate Functions: COUNT, SUM, AVG, MIN, MAX
- GROUP BY + HAVING
- Hiểu Data Types trong PostgreSQL

---

## 1. SQL là gì?

```
SQL = Structured Query Language

- Ngôn ngữ TIÊU CHUẨN để giao tiếp với RDBMS
- Ra đời 1970s (IBM), chuẩn ISO/ANSI
- Declarative: nói CẦN GÌ, không nói LÀM THẾ NÀO

Phân loại SQL statements:
┌─────────────────────────────────────────────┐
│ DDL (Data Definition Language)              │
│   CREATE, ALTER, DROP, TRUNCATE             │
│   → Định nghĩa CẤU TRÚC database           │
├─────────────────────────────────────────────┤
│ DML (Data Manipulation Language)            │
│   SELECT, INSERT, UPDATE, DELETE            │
│   → Thao tác DỮ LIỆU                       │
├─────────────────────────────────────────────┤
│ DCL (Data Control Language)                 │
│   GRANT, REVOKE                             │
│   → Quản lý QUYỀN                          │
├─────────────────────────────────────────────┤
│ TCL (Transaction Control Language)          │
│   BEGIN, COMMIT, ROLLBACK, SAVEPOINT        │
│   → Quản lý GIAO DỊCH                      │
└─────────────────────────────────────────────┘
```

---

## 2. Data Types trong PostgreSQL

### 2.1 Numeric Types

```sql
-- Integer types
SMALLINT      -- -32,768 → 32,767 (2 bytes)
INTEGER / INT -- -2.1 tỷ → 2.1 tỷ (4 bytes) ⭐ hay dùng nhất
BIGINT        -- rất lớn (8 bytes)
SERIAL        -- INT + auto-increment ⭐
BIGSERIAL     -- BIGINT + auto-increment

-- Decimal types (chính xác)
DECIMAL(10,2) -- 10 chữ số, 2 sau dấu phẩy
NUMERIC(10,2) -- Tương tự DECIMAL ⭐ dùng cho tiền

-- Floating point (xấp xỉ)
REAL          -- 4 bytes, 6 decimal digits
DOUBLE PRECISION -- 8 bytes, 15 decimal digits
-- ⚠️ KHÔNG dùng cho tiền! Dùng NUMERIC
```

### 2.2 String Types

```sql
CHAR(n)       -- Cố định n ký tự (thêm spaces nếu ngắn hơn)
VARCHAR(n)    -- Tối đa n ký tự ⭐ hay dùng nhất
TEXT          -- Không giới hạn ⭐ PostgreSQL specific
-- Ví dụ:
-- name VARCHAR(100)    → tên người
-- email VARCHAR(255)   → email
-- description TEXT     → mô tả dài
```

### 2.3 Date/Time Types

```sql
DATE          -- Chỉ ngày: '2026-02-22'
TIME          -- Chỉ giờ: '14:30:00'
TIMESTAMP     -- Ngày + giờ: '2026-02-22 14:30:00' ⭐
TIMESTAMPTZ   -- TIMESTAMP + timezone ⭐ nên dùng
INTERVAL      -- Khoảng thời gian: '1 year 2 months'
```

### 2.4 Other Types

```sql
BOOLEAN       -- TRUE, FALSE, NULL
UUID          -- Universally unique identifier
JSON / JSONB  -- JSON data (JSONB = binary, nhanh hơn) ⭐
BYTEA         -- Binary data
ARRAY         -- Mảng: INTEGER[], TEXT[]
```

---

## 3. DDL — Data Definition Language

### 3.1 CREATE TABLE

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    department VARCHAR(50) DEFAULT 'General',
    salary NUMERIC(10,2) CHECK (salary >= 0),
    hire_date DATE DEFAULT CURRENT_DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Tạo bảng với Foreign Key
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    manager_id INT REFERENCES employees(id)
);
```

### 3.2 ALTER TABLE

```sql
-- Thêm cột
ALTER TABLE employees ADD COLUMN phone VARCHAR(20);

-- Xóa cột
ALTER TABLE employees DROP COLUMN phone;

-- Đổi kiểu dữ liệu
ALTER TABLE employees ALTER COLUMN salary TYPE DECIMAL(12,2);

-- Thêm constraint
ALTER TABLE employees ADD CONSTRAINT chk_salary CHECK (salary >= 0);

-- Đổi tên cột
ALTER TABLE employees RENAME COLUMN first_name TO fname;

-- Đổi tên bảng
ALTER TABLE employees RENAME TO staff;

-- Set NOT NULL
ALTER TABLE employees ALTER COLUMN email SET NOT NULL;

-- Remove NOT NULL
ALTER TABLE employees ALTER COLUMN email DROP NOT NULL;
```

### 3.3 DROP & TRUNCATE

```sql
-- Xóa bảng HOÀN TOÀN (structure + data)
DROP TABLE employees;

-- Xóa chỉ khi exist (không lỗi nếu không tồn tại)
DROP TABLE IF EXISTS employees;

-- Xóa bảng và tất cả bảng REFER tới nó
DROP TABLE departments CASCADE;

-- TRUNCATE: Xóa TẤT CẢ data nhưng GIỮ cấu trúc
TRUNCATE TABLE employees;

-- TRUNCATE vs DELETE:
-- TRUNCATE: nhanh hơn, reset auto-increment, KHÔNG WHERE
-- DELETE:   chậm hơn, giữ auto-increment, CÓ WHERE
```

---

## 4. DML — Data Manipulation Language

### 4.1 INSERT

```sql
-- Insert 1 row
INSERT INTO employees (first_name, last_name, email, salary)
VALUES ('Nghĩa', 'Nguyễn', 'nghia@company.com', 50000);

-- Insert nhiều rows
INSERT INTO employees (first_name, last_name, email, salary)
VALUES 
    ('An', 'Trần', 'an@company.com', 45000),
    ('Bình', 'Lê', 'binh@company.com', 55000),
    ('Châu', 'Phạm', 'chau@company.com', 48000);

-- Insert + trả về data vừa insert
INSERT INTO employees (first_name, last_name, email, salary)
VALUES ('Dũng', 'Võ', 'dung@company.com', 52000)
RETURNING id, first_name, email;

-- Insert từ SELECT (copy data)
INSERT INTO employees_backup
SELECT * FROM employees WHERE is_active = TRUE;
```

### 4.2 SELECT

```sql
-- Tất cả cột
SELECT * FROM employees;

-- Chọn cột cụ thể
SELECT first_name, last_name, salary FROM employees;

-- Alias (đặt tên khác)
SELECT 
    first_name AS "Tên",
    last_name AS "Họ",
    salary AS "Lương",
    salary * 12 AS "Lương năm"
FROM employees;

-- DISTINCT (loại bỏ trùng lặp)
SELECT DISTINCT department FROM employees;

-- Biểu thức
SELECT 
    first_name || ' ' || last_name AS full_name,  -- Ghép chuỗi
    salary * 1.1 AS new_salary,                     -- Tính toán
    UPPER(email) AS email_upper                     -- Function
FROM employees;
```

### 4.3 WHERE — Lọc dữ liệu

```sql
-- So sánh
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE department = 'IT';
SELECT * FROM employees WHERE is_active = TRUE;

-- AND, OR, NOT
SELECT * FROM employees 
WHERE department = 'IT' AND salary > 50000;

SELECT * FROM employees 
WHERE department = 'IT' OR department = 'HR';

SELECT * FROM employees 
WHERE NOT is_active;

-- BETWEEN (khoảng)
SELECT * FROM employees 
WHERE salary BETWEEN 40000 AND 60000;
-- Tương đương: salary >= 40000 AND salary <= 60000

-- IN (danh sách giá trị)
SELECT * FROM employees 
WHERE department IN ('IT', 'HR', 'Finance');
-- Tương đương: department = 'IT' OR department = 'HR' OR ...

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE phone IS NULL;
SELECT * FROM employees WHERE phone IS NOT NULL;

-- LIKE (pattern matching)
SELECT * FROM employees WHERE first_name LIKE 'N%';     -- Bắt đầu bằng N
SELECT * FROM employees WHERE email LIKE '%@gmail.com';  -- Kết thúc bằng @gmail.com
SELECT * FROM employees WHERE last_name LIKE '_g%';      -- Ký tự 2 là 'g'

-- ILIKE (case-insensitive LIKE — PostgreSQL specific)
SELECT * FROM employees WHERE first_name ILIKE 'ngh%';
```

### 4.4 ORDER BY — Sắp xếp

```sql
-- Sắp xếp tăng dần (default)
SELECT * FROM employees ORDER BY salary;
SELECT * FROM employees ORDER BY salary ASC;

-- Sắp xếp giảm dần
SELECT * FROM employees ORDER BY salary DESC;

-- Nhiều cột
SELECT * FROM employees 
ORDER BY department ASC, salary DESC;

-- Sắp xếp theo biểu thức
SELECT first_name, salary, salary * 12 AS annual
FROM employees 
ORDER BY annual DESC;

-- Sắp xếp theo vị trí cột (KHÔNG khuyến khích)
SELECT first_name, salary FROM employees ORDER BY 2 DESC;
```

### 4.5 LIMIT & OFFSET — Phân trang

```sql
-- Top 5 lương cao nhất
SELECT * FROM employees ORDER BY salary DESC LIMIT 5;

-- Phân trang: Trang 1 (10 items/page)
SELECT * FROM employees ORDER BY id LIMIT 10 OFFSET 0;

-- Trang 2
SELECT * FROM employees ORDER BY id LIMIT 10 OFFSET 10;

-- Trang N (0-indexed)
-- OFFSET = (page_number - 1) * page_size
-- LIMIT  = page_size
```

### 4.6 UPDATE

```sql
-- Update 1 cột
UPDATE employees SET salary = 55000 WHERE id = 1;

-- Update nhiều cột
UPDATE employees 
SET salary = 60000, department = 'Engineering'
WHERE id = 1;

-- Update với expression
UPDATE employees 
SET salary = salary * 1.10  -- Tăng 10%
WHERE department = 'IT';

-- Update + RETURNING
UPDATE employees 
SET salary = salary * 1.05 
WHERE department = 'HR'
RETURNING id, first_name, salary;

-- ⚠️ CẢNH BÁO: UPDATE không có WHERE → update TẤT CẢ rows!
UPDATE employees SET salary = 0;  -- 💀 Đừng bao giờ làm thế!
```

### 4.7 DELETE

```sql
-- Delete với điều kiện
DELETE FROM employees WHERE id = 5;

-- Delete nhiều rows
DELETE FROM employees WHERE department = 'Temp';

-- Delete + RETURNING
DELETE FROM employees WHERE is_active = FALSE
RETURNING id, first_name;

-- ⚠️ CẢNH BÁO: DELETE không có WHERE → xóa TẤT CẢ rows!
DELETE FROM employees;  -- 💀 Xóa sạch bảng!

-- DELETE vs TRUNCATE:
-- DELETE:   chậm hơn, log từng row, WHERE được, trigger chạy
-- TRUNCATE: nhanh hơn, không log row, không WHERE, không trigger
```

---

## 5. Aggregate Functions

```sql
-- COUNT: Đếm số rows
SELECT COUNT(*) FROM employees;                    -- Tất cả rows (kể cả NULL)
SELECT COUNT(phone) FROM employees;                -- Chỉ đếm non-NULL
SELECT COUNT(DISTINCT department) FROM employees;  -- Đếm unique values

-- SUM: Tổng
SELECT SUM(salary) FROM employees;
SELECT SUM(salary) FROM employees WHERE department = 'IT';

-- AVG: Trung bình
SELECT AVG(salary) FROM employees;
SELECT ROUND(AVG(salary), 2) AS avg_salary FROM employees;  -- Làm tròn

-- MIN, MAX
SELECT MIN(salary) AS lowest, MAX(salary) AS highest FROM employees;
SELECT MIN(hire_date) AS earliest, MAX(hire_date) AS latest FROM employees;

-- Kết hợp nhiều aggregates
SELECT 
    COUNT(*) AS total_employees,
    ROUND(AVG(salary), 2) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    SUM(salary) AS total_payroll
FROM employees
WHERE is_active = TRUE;
```

---

## 6. GROUP BY & HAVING

### 6.1 GROUP BY

```sql
-- Đếm nhân viên theo phòng ban
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;

-- Lương trung bình theo phòng ban
SELECT 
    department,
    COUNT(*) AS count,
    ROUND(AVG(salary), 2) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;

-- GROUP BY nhiều cột
SELECT department, is_active, COUNT(*)
FROM employees
GROUP BY department, is_active;
```

### 6.2 HAVING — Lọc SAU GROUP BY

```sql
-- Phòng ban có > 5 nhân viên
SELECT department, COUNT(*) AS count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

-- Phòng ban có lương TB > 50,000
SELECT department, ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000
ORDER BY avg_salary DESC;
```

### 6.3 WHERE vs HAVING

```sql
-- WHERE: lọc TRƯỚC khi group
-- HAVING: lọc SAU khi group

-- Thứ tự thực hiện:
-- FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE is_active = TRUE          -- 1. Lọc chỉ active employees
GROUP BY department              -- 2. Nhóm theo department
HAVING AVG(salary) > 50000      -- 3. Lọc groups có avg > 50k
ORDER BY avg_salary DESC        -- 4. Sắp xếp
LIMIT 5;                        -- 5. Giới hạn kết quả
```

---

## 7. Useful SQL Functions

### 7.1 String Functions

```sql
-- Ghép chuỗi
SELECT first_name || ' ' || last_name AS full_name FROM employees;
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- Chuyển đổi
SELECT UPPER('hello');        -- 'HELLO'
SELECT LOWER('HELLO');        -- 'hello'
SELECT INITCAP('hello world'); -- 'Hello World'

-- Cắt/tìm
SELECT LENGTH('hello');       -- 5
SELECT SUBSTRING('hello world' FROM 1 FOR 5);  -- 'hello'
SELECT POSITION('world' IN 'hello world');      -- 7
SELECT REPLACE('hello world', 'world', 'SQL');  -- 'hello SQL'

-- Trim
SELECT TRIM('  hello  ');     -- 'hello'
SELECT LTRIM('  hello');      -- 'hello'
SELECT RTRIM('hello  ');      -- 'hello'

-- LEFT, RIGHT
SELECT LEFT('PostgreSQL', 8);  -- 'PostgreS'
SELECT RIGHT('PostgreSQL', 3); -- 'SQL'
```

### 7.2 Date/Time Functions

```sql
-- Ngày hiện tại
SELECT CURRENT_DATE;          -- '2026-02-22'
SELECT CURRENT_TIMESTAMP;     -- '2026-02-22 10:30:00+07'
SELECT NOW();                 -- Tương tự CURRENT_TIMESTAMP

-- Trích xuất
SELECT EXTRACT(YEAR FROM hire_date) AS year FROM employees;
SELECT EXTRACT(MONTH FROM hire_date) AS month FROM employees;
SELECT EXTRACT(DOW FROM hire_date) AS day_of_week FROM employees;

-- Tính toán ngày
SELECT hire_date + INTERVAL '1 year' FROM employees;
SELECT hire_date + 30 FROM employees;  -- + 30 ngày
SELECT AGE(NOW(), hire_date) FROM employees;  -- Khoảng cách

-- Format
SELECT TO_CHAR(hire_date, 'DD/MM/YYYY') FROM employees;
SELECT TO_CHAR(NOW(), 'HH24:MI:SS') AS current_time;
```

### 7.3 Conditional: CASE

```sql
-- CASE WHEN — tương tự switch/if-else
SELECT 
    first_name,
    salary,
    CASE 
        WHEN salary >= 80000 THEN 'Senior'
        WHEN salary >= 50000 THEN 'Mid'
        WHEN salary >= 30000 THEN 'Junior'
        ELSE 'Intern'
    END AS level
FROM employees;

-- CASE với GROUP BY
SELECT 
    CASE 
        WHEN salary >= 60000 THEN 'High'
        WHEN salary >= 40000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_band,
    COUNT(*) AS count
FROM employees
GROUP BY salary_band;
```

### 7.4 COALESCE & NULLIF

```sql
-- COALESCE: trả về giá trị NON-NULL đầu tiên
SELECT COALESCE(phone, email, 'No contact') AS contact 
FROM employees;

-- NULLIF: trả về NULL nếu 2 giá trị bằng nhau
SELECT NULLIF(department, 'Unknown') FROM employees;
-- Nếu department = 'Unknown' → NULL, ngược lại giữ nguyên

-- Tránh chia cho 0
SELECT total / NULLIF(count, 0) AS average FROM stats;
```

---

## 8. Practical Examples

### 8.1 Employee Management System

```sql
-- Setup
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    dept_id INT REFERENCES departments(id),
    salary NUMERIC(10,2) CHECK (salary > 0),
    hire_date DATE DEFAULT CURRENT_DATE,
    is_active BOOLEAN DEFAULT TRUE
);

-- Insert data
INSERT INTO departments (name) 
VALUES ('Engineering'), ('HR'), ('Finance'), ('Marketing');

INSERT INTO employees (first_name, last_name, email, dept_id, salary, hire_date)
VALUES
    ('Nghĩa', 'Nguyễn', 'nghia@co.com', 1, 75000, '2023-01-15'),
    ('An', 'Trần', 'an@co.com', 1, 65000, '2023-06-01'),
    ('Bình', 'Lê', 'binh@co.com', 2, 55000, '2024-03-10'),
    ('Châu', 'Phạm', 'chau@co.com', 3, 70000, '2022-11-20'),
    ('Dũng', 'Võ', 'dung@co.com', 1, 80000, '2021-08-05'),
    ('Em', 'Hoàng', 'em@co.com', 4, 48000, '2025-01-10'),
    ('Giang', 'Đỗ', 'giang@co.com', 2, 52000, '2024-07-15'),
    ('Hải', 'Bùi', 'hai@co.com', 3, 72000, '2023-04-22');

-- Queries thực tế:

-- 1. Top 3 lương cao nhất
SELECT first_name, last_name, salary
FROM employees ORDER BY salary DESC LIMIT 3;

-- 2. Nhân viên Engineering, sắp theo thâm niên
SELECT first_name, hire_date, salary
FROM employees 
WHERE dept_id = 1
ORDER BY hire_date ASC;

-- 3. Thống kê theo phòng ban
SELECT 
    d.name AS department,
    COUNT(e.id) AS employees,
    ROUND(AVG(e.salary), 0) AS avg_salary
FROM employees e
JOIN departments d ON e.dept_id = d.id
GROUP BY d.name
ORDER BY avg_salary DESC;

-- 4. Tăng lương 5% cho ai làm > 2 năm
UPDATE employees 
SET salary = salary * 1.05
WHERE hire_date < CURRENT_DATE - INTERVAL '2 years'
RETURNING first_name, salary;
```

---

## ❓ Câu Hỏi Kiểm Tra

1. DDL vs DML — cho 2 ví dụ mỗi loại.
2. VARCHAR(100) vs TEXT — khi nào dùng cái nào?
3. SERIAL làm gì? Tương đương lệnh gì?
4. INSERT nhiều rows trong 1 lệnh — syntax?
5. WHERE vs HAVING — khác nhau thế nào?
6. LIKE '%abc%' tìm gì? 'a_c' tìm gì?
7. COUNT(*) vs COUNT(column) — khác nhau?
8. TRUNCATE vs DELETE — 3 điểm khác?
9. ORDER BY 2 DESC nghĩa gì?
10. COALESCE dùng khi nào? NULLIF dùng khi nào?
11. Thứ tự thực hiện SQL: FROM → ? → ? → ? → ? → ?
12. CASE WHEN trong SQL tương tự gì trong C#?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between DDL and DML?**
> A: **DDL** (Data Definition Language): CREATE, ALTER, DROP, TRUNCATE — thay đổi STRUCTURE. **DML** (Data Manipulation Language): SELECT, INSERT, UPDATE, DELETE — thay đổi DATA. DDL tự động commit, DML có thể rollback.

> **Q: What is the difference between DELETE and TRUNCATE?**
> A: DELETE: DML, lọc WHERE, log từng row, trigger chạy, chậm. TRUNCATE: DDL, xóa ALL, không log row, reset auto-increment, nhanh. Dùng DELETE khi cần chọn lọc, TRUNCATE khi xóa toàn bộ bảng.

> **Q: What is the difference between WHERE and HAVING?**
> A: WHERE lọc ROWS trước grouping. HAVING lọc GROUPS sau grouping. WHERE không dùng aggregate functions, HAVING dùng được. Thứ tự: FROM → WHERE → GROUP BY → HAVING.

> **Q: What are aggregate functions? Give examples.**
> A: Functions tính trên tập hợp rows: COUNT (đếm), SUM (tổng), AVG (trung bình), MIN (nhỏ nhất), MAX (lớn nhất). Thường dùng với GROUP BY. COUNT(*) đếm tất cả rows, COUNT(col) bỏ qua NULL.

> **Q: What is the order of SQL query execution?**
> A: FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT/OFFSET. Lưu ý: SELECT alias KHÔNG dùng được trong WHERE (vì WHERE chạy trước SELECT).

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Tạo database `school_db`. Tạo bảng `students` (id, name, age, grade, enrollment_date). Insert 10 students. Viết query: (a) top 5 điểm cao nhất, (b) trung bình điểm, (c) đếm theo grade band.

**BT2**: Dùng employee data ở trên. Viết query: (a) nhân viên hire trong 2024, (b) phòng ban nào có avg salary > 60k, (c) phân trang trang 2 (5 items/page).

**BT3**: Viết 5 queries dùng CASE WHEN để phân loại employees thành Junior/Mid/Senior/Lead dựa trên salary + thâm niên.

**BT4**: Tạo bảng `orders` (id, customer_name, amount, order_date, status). Insert 20 orders. Viết: (a) doanh thu theo tháng, (b) top 3 khách hàng chi nhiều nhất, (c) % đơn hàng theo status.

---

## ✅ Checklist Hoàn Thành SQL Fundamentals

1. Tự tạo schema mới với `PRIMARY KEY`, `UNIQUE`, `CHECK`, `DEFAULT` và giải thích vì sao chọn từng ràng buộc.
2. Viết được trọn bộ CRUD có điều kiện lọc và sắp xếp rõ ràng.
3. Dùng đúng `WHERE`, `GROUP BY`, `HAVING`, không nhầm thứ tự xử lý query.
4. Viết được query báo cáo có `CASE`, `COALESCE`, aggregate functions.
5. Xử lý được dữ liệu `NULL` mà không gây sai kết quả.
6. Hoàn thành ít nhất 10 queries tự nghĩ thêm ngoài ví dụ trong tài liệu.
7. Trình bày được query theo ngôn ngữ nghiệp vụ (không chỉ đọc syntax).

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-db.md#db02-sql-fundamentals](./99-answer-key-db.md#db02-sql-fundamentals)
- Bài tập thực hành: [99-answer-key-db.md#db02-sql-fundamentals-exercises](./99-answer-key-db.md#db02-sql-fundamentals-exercises)
- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db02-sql-fundamentals-deep](./97-db-theory-deep-dive.md#db02-sql-fundamentals-deep)

