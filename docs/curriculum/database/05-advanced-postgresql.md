# 📓 Phần 5: Advanced PostgreSQL

> **Nguồn**: IBM + UMich + bổ sung kiến thức  
> **Thời lượng ước tính**: 5 giờ

---

## 🎯 Mục Tiêu

- Tạo và sử dụng Views, Materialized Views
- Viết Stored Functions bằng PL/pgSQL
- Hiểu Transactions và Isolation Levels
- Sử dụng Window Functions cho analytics
- Tạo Triggers cho automation
- Tối ưu query bằng EXPLAIN ANALYZE
- Kết nối PostgreSQL với C# (preview)

---

## 1. Views

### 1.1 View là gì?

```sql
-- View = "stored query" — bảng ẢO, không lưu data

-- Tạo View
CREATE VIEW employee_details AS
SELECT 
    e.id,
    e.name,
    e.salary,
    d.name AS department,
    e.hire_date
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;

-- Dùng View như bảng thường
SELECT * FROM employee_details WHERE department = 'Engineering';
SELECT department, AVG(salary) FROM employee_details GROUP BY department;

-- Ưu điểm:
-- ✅ Đơn giản hóa queries phức tạp
-- ✅ Bảo mật: ẩn columns nhạy cảm
-- ✅ Consistency: 1 nơi sửa, tất cả nơi dùng
```

### 1.2 Updatable Views

```sql
-- Một số views cho phép INSERT/UPDATE/DELETE
CREATE VIEW active_employees AS
SELECT id, name, email, salary
FROM employees
WHERE is_active = TRUE;

-- Insert qua view (nếu đủ điều kiện)
INSERT INTO active_employees (name, email, salary)
VALUES ('Hải', 'hai@co.com', 60000);

-- Update qua view  
UPDATE active_employees SET salary = 65000 WHERE id = 10;

-- WITH CHECK OPTION: ngăn insert/update vi phạm điều kiện view
CREATE VIEW active_employees AS
SELECT * FROM employees WHERE is_active = TRUE
WITH CHECK OPTION;
-- INSERT employees với is_active = FALSE → LỖI!
```

### 1.3 Materialized Views

```sql
-- Materialized View = View + LƯU DATA (cache)
-- Nhanh hơn View thường vì không cần tính lại mỗi query

CREATE MATERIALIZED VIEW monthly_revenue AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;

-- Query (đọc từ cache → NHANH)
SELECT * FROM monthly_revenue;

-- Refresh khi data thay đổi
REFRESH MATERIALIZED VIEW monthly_revenue;

-- Refresh concurrently (không block reads)
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_revenue;
-- ⚠️ Cần UNIQUE index cho CONCURRENTLY

-- Drop
DROP MATERIALIZED VIEW monthly_revenue;

-- Khi nào dùng?
-- ✅ Reports, dashboards (data không cần real-time)
-- ✅ Complex aggregations chạy chậm
-- ❌ Data cần real-time accuracy
```

---

## 2. Stored Functions (PL/pgSQL)

### 2.1 Function cơ bản

```sql
-- Function tính thuế
CREATE OR REPLACE FUNCTION calculate_tax(price NUMERIC, tax_rate NUMERIC DEFAULT 0.1)
RETURNS NUMERIC AS $$
BEGIN
    RETURN ROUND(price * tax_rate, 2);
END;
$$ LANGUAGE plpgsql;

-- Sử dụng
SELECT calculate_tax(100);        -- 10.00
SELECT calculate_tax(100, 0.08);  -- 8.00
SELECT name, price, calculate_tax(price) AS tax FROM products;
```

### 2.2 Function với IF/ELSE

```sql
-- Function xếp loại nhân viên
CREATE OR REPLACE FUNCTION get_employee_level(emp_salary NUMERIC)
RETURNS TEXT AS $$
BEGIN
    IF emp_salary >= 80000 THEN
        RETURN 'Senior';
    ELSIF emp_salary >= 50000 THEN
        RETURN 'Mid';
    ELSIF emp_salary >= 30000 THEN
        RETURN 'Junior';
    ELSE
        RETURN 'Intern';
    END IF;
END;
$$ LANGUAGE plpgsql;

SELECT name, salary, get_employee_level(salary) AS level
FROM employees;
```

### 2.3 Function trả về TABLE

```sql
-- Function trả về employees theo department
CREATE OR REPLACE FUNCTION get_department_employees(dept_name TEXT)
RETURNS TABLE (
    employee_name VARCHAR,
    employee_salary NUMERIC,
    employee_level TEXT
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        e.name,
        e.salary,
        get_employee_level(e.salary)
    FROM employees e
    JOIN departments d ON e.dept_id = d.id
    WHERE d.name = dept_name
    ORDER BY e.salary DESC;
END;
$$ LANGUAGE plpgsql;

-- Sử dụng
SELECT * FROM get_department_employees('Engineering');
```

### 2.4 Function với Variables và Loops

```sql
-- Function tính tổng lương phòng ban
CREATE OR REPLACE FUNCTION department_payroll_report()
RETURNS TABLE (
    dept_name VARCHAR,
    total_salary NUMERIC,
    avg_salary NUMERIC,
    emp_count BIGINT
) AS $$
DECLARE
    dept RECORD;
BEGIN
    FOR dept IN SELECT id, name FROM departments ORDER BY name
    LOOP
        dept_name := dept.name;
        
        SELECT 
            COALESCE(SUM(salary), 0),
            COALESCE(ROUND(AVG(salary), 2), 0),
            COUNT(*)
        INTO total_salary, avg_salary, emp_count
        FROM employees
        WHERE dept_id = dept.id;
        
        RETURN NEXT;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

SELECT * FROM department_payroll_report();
```

---

## 3. Triggers

### 3.1 Trigger là gì?

```
Trigger = function TỰ ĐỘNG chạy khi có sự kiện (INSERT/UPDATE/DELETE)

Timing:
  BEFORE: chạy TRƯỚC thao tác → có thể THAY ĐỔI data
  AFTER:  chạy SAU thao tác → logging, sync
  INSTEAD OF: thay thế thao tác (dùng cho views)

Level:
  FOR EACH ROW:       chạy cho MỖI row affected
  FOR EACH STATEMENT: chạy 1 lần cho cả statement
```

### 3.2 Ví dụ Triggers

```sql
-- 1. Auto-update updated_at timestamp
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_employees_updated
    BEFORE UPDATE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION update_timestamp();

-- Khi UPDATE employee → updated_at tự động cập nhật!

-- 2. Audit log
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name TEXT,
    action TEXT,
    old_data JSONB,
    new_data JSONB,
    changed_at TIMESTAMPTZ DEFAULT NOW(),
    changed_by TEXT DEFAULT CURRENT_USER
);

CREATE OR REPLACE FUNCTION log_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF (TG_OP = 'DELETE') THEN
        INSERT INTO audit_log (table_name, action, old_data)
        VALUES (TG_TABLE_NAME, 'DELETE', to_jsonb(OLD));
        RETURN OLD;
    ELSIF (TG_OP = 'UPDATE') THEN
        INSERT INTO audit_log (table_name, action, old_data, new_data)
        VALUES (TG_TABLE_NAME, 'UPDATE', to_jsonb(OLD), to_jsonb(NEW));
        RETURN NEW;
    ELSIF (TG_OP = 'INSERT') THEN
        INSERT INTO audit_log (table_name, action, new_data)
        VALUES (TG_TABLE_NAME, 'INSERT', to_jsonb(NEW));
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_employees_audit
    AFTER INSERT OR UPDATE OR DELETE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION log_changes();

-- 3. Validate data (BEFORE INSERT)
CREATE OR REPLACE FUNCTION validate_salary()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.salary < 0 THEN
        RAISE EXCEPTION 'Salary cannot be negative: %', NEW.salary;
    END IF;
    IF NEW.salary > 1000000 THEN
        RAISE WARNING 'Very high salary: %', NEW.salary;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_validate_salary
    BEFORE INSERT OR UPDATE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION validate_salary();
```

---

## 4. Transactions

### 4.1 Transaction Basics

```sql
-- Transaction = nhóm operations thực hiện ATOMIC

-- Chuyển tiền: phải cả 2 thành công hoặc cả 2 thất bại
BEGIN;  -- hoặc BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 500000 WHERE id = 1;
UPDATE accounts SET balance = balance + 500000 WHERE id = 2;

-- Kiểm tra
SELECT id, balance FROM accounts WHERE id IN (1, 2);

COMMIT;    -- ✅ Xác nhận tất cả thay đổi
-- ROLLBACK; -- ❌ Huỷ tất cả thay đổi
```

### 4.2 SAVEPOINT

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100000 WHERE id = 1;
SAVEPOINT sp1;

UPDATE accounts SET balance = balance - 200000 WHERE id = 1;
-- Oops, sai rồi!

ROLLBACK TO SAVEPOINT sp1;  -- Chỉ huỷ từ sp1, giữ trước đó

-- Balance đã trừ 100k (trước sp1), chưa trừ 200k (sau sp1)
COMMIT;
```

### 4.3 Isolation Levels

```sql
-- 4 mức cô lập (từ thấp → cao):

-- 1. READ UNCOMMITTED (PostgreSQL không hỗ trợ, tương đương READ COMMITTED)
-- Có thể đọc data chưa commit → "dirty read"

-- 2. READ COMMITTED (default PostgreSQL) ⭐
-- Chỉ đọc data đã COMMIT
-- Mỗi statement thấy snapshot mới
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 3. REPEATABLE READ
-- Snapshot cố định từ đầu transaction
-- Không bị phantom reads
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- 4. SERIALIZABLE (cao nhất — an toàn nhất)
-- Transactions chạy như TUẦN TỰ
-- Chậm nhất nhưng không bao giờ sai
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

```
Vấn đề:                    RC    RR    SERIAL
─────────────────────────────────────────────
Dirty Read                  ❌    ❌      ❌
Non-repeatable Read         ✅    ❌      ❌
Phantom Read                ✅    ❌      ❌
Serialization Anomaly       ✅    ✅      ❌

RC = READ COMMITTED, RR = REPEATABLE READ
✅ = có thể xảy ra, ❌ = không xảy ra
```

---

## 5. Window Functions

### 5.1 Window Functions là gì?

```sql
-- Window function = tính toán trên "cửa sổ" rows LIÊN QUAN
-- Khác aggregate: KHÔNG gom rows lại (giữ từng row)

-- So sánh:
-- Aggregate: SELECT dept_id, AVG(salary) GROUP BY dept_id  → 1 row/dept
-- Window:    SELECT name, salary, AVG(salary) OVER(PARTITION BY dept_id)  → giữ mỗi row!
```

### 5.2 ROW_NUMBER, RANK, DENSE_RANK

```sql
-- ROW_NUMBER: số thứ tự liên tục (1, 2, 3, 4...)
SELECT 
    name, salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;

-- RANK: xếp hạng, SKIP nếu trùng (1, 2, 2, 4...)
SELECT 
    name, salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- DENSE_RANK: xếp hạng, KHÔNG skip (1, 2, 2, 3...)
SELECT 
    name, salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- So sánh:
-- Salary: 80k, 72k, 72k, 65k
-- ROW_NUMBER: 1, 2, 3, 4
-- RANK:       1, 2, 2, 4  (skip 3!)
-- DENSE_RANK: 1, 2, 2, 3  (không skip)
```

### 5.3 PARTITION BY

```sql
-- Xếp hạng TRONG MỖI phòng ban
SELECT 
    name, dept_id, salary,
    ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS dept_rank
FROM employees;

-- Top 2 lương cao nhất MỖI phòng ban
WITH ranked AS (
    SELECT 
        e.name, d.name AS dept, e.salary,
        ROW_NUMBER() OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS rn
    FROM employees e
    JOIN departments d ON e.dept_id = d.id
)
SELECT name, dept, salary FROM ranked WHERE rn <= 2;
```

### 5.4 LAG, LEAD

```sql
-- LAG: giá trị row TRƯỚC
-- LEAD: giá trị row SAU

SELECT 
    name, hire_date, salary,
    LAG(salary) OVER (ORDER BY hire_date) AS prev_salary,
    LEAD(salary) OVER (ORDER BY hire_date) AS next_salary,
    salary - LAG(salary) OVER (ORDER BY hire_date) AS salary_change
FROM employees
ORDER BY hire_date;

-- Use case: So sánh doanh thu tháng này vs tháng trước
SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month,
    ROUND((revenue - LAG(revenue) OVER (ORDER BY month)) / 
          LAG(revenue) OVER (ORDER BY month) * 100, 1) AS growth_pct
FROM monthly_revenue;
```

### 5.5 SUM, AVG, COUNT over Window

```sql
-- Running total (tổng cộng dồn)
SELECT 
    name, salary,
    SUM(salary) OVER (ORDER BY hire_date) AS running_total
FROM employees;

-- Running average
SELECT 
    name, salary,
    ROUND(AVG(salary) OVER (ORDER BY hire_date), 2) AS running_avg
FROM employees;

-- % of department total
SELECT 
    name, dept_id, salary,
    SUM(salary) OVER (PARTITION BY dept_id) AS dept_total,
    ROUND(salary * 100.0 / SUM(salary) OVER (PARTITION BY dept_id), 1) AS pct_of_dept
FROM employees;

-- Moving average (3 rows)
SELECT 
    order_date, amount,
    ROUND(AVG(amount) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS moving_avg_3
FROM orders;
```

### 5.6 FIRST_VALUE, LAST_VALUE, NTH_VALUE

```sql
-- Highest salary per department
SELECT 
    name, dept_id, salary,
    FIRST_VALUE(name) OVER (PARTITION BY dept_id ORDER BY salary DESC) AS top_earner,
    FIRST_VALUE(salary) OVER (PARTITION BY dept_id ORDER BY salary DESC) AS top_salary
FROM employees;
```

---

## 6. Query Optimization

### 6.1 EXPLAIN ANALYZE

```sql
-- EXPLAIN: hiển thị query plan (không chạy query)
EXPLAIN SELECT * FROM employees WHERE dept_id = 1;

-- EXPLAIN ANALYZE: chạy query + hiển thị thời gian thực
EXPLAIN ANALYZE SELECT * FROM employees WHERE dept_id = 1;

-- Output:
-- Seq Scan on employees  (cost=0.00..1.09 rows=3 width=...)
--   Filter: (dept_id = 1)
--   Rows Removed by Filter: 4
--   Planning Time: 0.1 ms
--   Execution Time: 0.05 ms
```

### 6.2 Đọc Query Plan

```sql
-- Seq Scan:   Quét TOÀN BỘ bảng (chậm với bảng lớn)
-- Index Scan: Dùng index → NHANH ⭐
-- Hash Join:  JOIN bằng hash table
-- Nested Loop: JOIN bằng nested loop (tốt cho bảng nhỏ)

-- Tạo index để tối ưu
CREATE INDEX idx_emp_dept ON employees(dept_id);

-- Chạy lại → Index Scan thay vì Seq Scan
EXPLAIN ANALYZE SELECT * FROM employees WHERE dept_id = 1;
```

### 6.3 Tips Tối Ưu Query

```sql
-- 1. Dùng SELECT cụ thể thay vì SELECT *
SELECT name, salary FROM employees;  -- ✅
SELECT * FROM employees;              -- ❌

-- 2. Tạo index cho cột WHERE/JOIN/ORDER BY
CREATE INDEX idx_orders_date ON orders(order_date);

-- 3. LIMIT khi không cần tất cả data
SELECT * FROM orders ORDER BY order_date DESC LIMIT 10;

-- 4. EXISTS thay vì IN cho correlated queries
WHERE EXISTS (...) thay vì WHERE id IN (SELECT ...)

-- 5. Tránh function trên indexed column
WHERE LOWER(name) = 'nghĩa'  -- ❌ Index không dùng được
WHERE name = 'Nghĩa'          -- ✅

-- 6. Composite index cho multi-column WHERE
CREATE INDEX idx_emp_dept_salary ON employees(dept_id, salary);
-- ⚠️ Thứ tự cột QUAN TRỌNG! (leftmost prefix rule)
```

---

## 7. Kết Nối với C# (Preview)

### 7.1 ADO.NET (Low-level)

```csharp
// NuGet: Npgsql
using Npgsql;

var connString = "Host=localhost;Database=mydb;Username=postgres;Password=secret";

await using var conn = new NpgsqlConnection(connString);
await conn.OpenAsync();

// Query
await using var cmd = new NpgsqlCommand(
    "SELECT name, salary FROM employees WHERE dept_id = @deptId", conn);
cmd.Parameters.AddWithValue("deptId", 1);  // Parameterized → safe from SQL injection!

await using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    Console.WriteLine($"{reader.GetString(0)}: {reader.GetDecimal(1):C}");
}
```

### 7.2 Entity Framework Core (High-level) ⭐

```csharp
// NuGet: Npgsql.EntityFrameworkCore.PostgreSQL

// Models
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Salary { get; set; }
    public int? DeptId { get; set; }
    public Department? Department { get; set; }
}

public class Department
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public List<Employee> Employees { get; set; } = new();
}

// DbContext
public class AppDbContext : DbContext
{
    public DbSet<Employee> Employees => Set<Employee>();
    public DbSet<Department> Departments => Set<Department>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseNpgsql("Host=localhost;Database=mydb;Username=postgres;Password=secret");
}

// CRUD
using var db = new AppDbContext();

// Read (LINQ → SQL!)
var engineers = db.Employees
    .Include(e => e.Department)
    .Where(e => e.Department!.Name == "Engineering")
    .OrderByDescending(e => e.Salary)
    .ToList();

// Tương đương SQL:
// SELECT e.* FROM employees e
// JOIN departments d ON e.dept_id = d.id
// WHERE d.name = 'Engineering'
// ORDER BY e.salary DESC

// Create
db.Employees.Add(new Employee { Name = "Hải", Salary = 60000, DeptId = 1 });
await db.SaveChangesAsync();

// Update
var emp = db.Employees.Find(1);
emp!.Salary = 85000;
await db.SaveChangesAsync();

// Delete
db.Employees.Remove(emp);
await db.SaveChangesAsync();
```

---

## ❓ Câu Hỏi Kiểm Tra

1. View vs Materialized View — khác nhau?
2. Khi nào dùng Materialized View?
3. BEFORE vs AFTER trigger — dùng khi nào?
4. Transaction SAVEPOINT dùng để làm gì?
5. 4 Isolation Levels — kể từ thấp → cao.
6. ROW_NUMBER vs RANK vs DENSE_RANK — cho ví dụ kết quả.
7. LAG và LEAD dùng khi nào?
8. EXPLAIN ANALYZE cho thấy gì?
9. Seq Scan vs Index Scan — cái nào nhanh hơn?
10. SQL injection là gì? Cách phòng tránh?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What are Views in SQL?**
> A: View = stored query, bảng ảo. Không lưu data (tính mỗi query). Dùng cho: simplify queries, security (ẩn columns), consistency. Materialized View lưu data + cần REFRESH. Một số views cho phép INSERT/UPDATE.

> **Q: What are Window Functions?**
> A: Functions tính trên set rows liên quan NHƯNG giữ từng row (khác aggregate). ROW_NUMBER, RANK, DENSE_RANK cho ranking. LAG/LEAD cho comparison. SUM/AVG OVER cho running totals. PARTITION BY chia groups.

> **Q: What are database transactions?**
> A: Nhóm operations thực hiện atomic (all-or-nothing). BEGIN → operations → COMMIT/ROLLBACK. Đảm bảo ACID. Isolation levels kiểm soát concurrency: READ COMMITTED (default), REPEATABLE READ, SERIALIZABLE.

> **Q: What is the difference between a stored procedure and a function?**
> A: Function: RETURNS giá trị, dùng trong SELECT/WHERE, ideally no side effects. Procedure: KHÔNG bắt buộc return, có thể có side effects (INSERT/UPDATE), gọi bằng CALL. PostgreSQL: dùng functions nhiều hơn.

> **Q: How do you optimize a slow SQL query?**
> A: (1) EXPLAIN ANALYZE xem plan. (2) Tạo indexes phù hợp. (3) SELECT cột cần thay vì *. (4) LIMIT data. (5) EXISTS thay IN. (6) Composite indexes. (7) Materialized Views cho reports. (8) Tránh functions trên indexed columns.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Tạo View `order_summary` hiển thị: customer name, order date, total items, total amount. Tạo Materialized View cho monthly revenue report.

**BT2**: Viết trigger: audit log cho bảng products — ghi lại mọi INSERT/UPDATE/DELETE vào bảng `product_changes`.

**BT3**: Dùng Window Functions: (a) rank employees by salary per department, (b) running total revenue by month, (c) growth % so với tháng trước dùng LAG.

**BT4**: Tạo stored function `transfer_money(from_id, to_id, amount)` dùng transaction. Handle insufficient balance bằng RAISE EXCEPTION.

**BT5**: Dùng EXPLAIN ANALYZE so sánh query VỚI và KHÔNG CÓ index. Measure thời gian chênh lệch trên bảng 10,000+ rows.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-db.md#db05-advanced-postgresql](./99-answer-key-db.md#db05-advanced-postgresql)
- Bài tập thực hành: [99-answer-key-db.md#db05-advanced-postgresql-exercises](./99-answer-key-db.md#db05-advanced-postgresql-exercises)
- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db05-advanced-postgresql-deep](./97-db-theory-deep-dive.md#db05-advanced-postgresql-deep)

---

## 8. So Sánh MySQL vs PostgreSQL

| Tiêu chí | MySQL 🐬 | PostgreSQL 🐘 |
|----------|---------|--------------| 
| **License** | GPL (Oracle sở hữu) | PostgreSQL License (100% free) |
| **Kiểu** | RDBMS | Object-RDBMS |
| **CLI** | `mysql` | `psql` |
| **GUI chính** | Workbench, phpMyAdmin | pgAdmin |
| **Auto-increment** | `AUTO_INCREMENT` | `SERIAL` |
| **Backup** | `mysqldump` | `pg_dump` |
| **Views** | Có ✅ | Có + **Materialized Views** ✅ |
| **JSON** | Cơ bản | Nâng cao (JSONB) |
| **GIS** | Hạn chế | PostGIS (cực mạnh) |
| **Phù hợp** | Web apps, e-commerce | Analytics, GIS, enterprise |

---

## 9. B-Tree vs Hash Index (Internals)

```
B-Tree Index (DEFAULT):
┌────────────────────────────────────────┐
│         Root: M                        │
│        /     \                         │
│    A-L        N-Z                      │
│   /  \       /  \                      │
│ A-F  G-L   N-S  T-Z                   │
│ Tìm "csev" → 3 bước thay vì scan!    │
└────────────────────────────────────────┘

Hash Index:
Input: "csev@umich.edu" → Hash → slot 47 → Found! O(1)
```

| | B-Tree Index | Hash Index |
|---|---|---|
| `=` | ✅ O(log n) | ✅ O(1) ⚡ |
| `<`, `>`, `≤` | ✅ O(log n) | ❌ |
| `LIKE 'a%'` | ✅ Prefix match | ❌ |
| `ORDER BY` | ✅ Pre-sorted | ❌ |

> → B-Tree linh hoạt hơn → **DEFAULT** index type

---

## 10. Backup & Restore

```bash
# PostgreSQL
pg_dump mydb > backup.sql       # Backup
psql mydb < backup.sql          # Restore
# pgAdmin: Right-click DB → Backup/Restore

# MySQL
mysqldump -u root mydb > backup.sql
mysql -u root mydb < backup.sql
SOURCE backup.sql;  # Tại MySQL prompt
```

- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db05-advanced-postgresql-deep](./97-db-theory-deep-dive.md#db05-advanced-postgresql-deep)

