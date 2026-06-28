# 📙 Phần 4: JOINs & Subqueries

> **Nguồn**: UMich Module 3 + Subqueries Guided Project + bổ sung kiến thức  
> **Thời lượng ước tính**: 5 giờ

---

## 🎯 Mục Tiêu

- Thành thạo tất cả loại JOIN: INNER, LEFT, RIGHT, FULL, CROSS, SELF
- Viết subqueries trong WHERE, FROM, SELECT clauses
- Hiểu correlated vs uncorrelated subqueries
- Sử dụng Common Table Expressions (CTEs)
- Dùng set operations: UNION, INTERSECT, EXCEPT

---

## 1. JOIN — Kết hợp dữ liệu từ nhiều bảng

### Setup Data

```sql
-- Bảng mẫu cho tất cả ví dụ JOIN
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    dept_id INT REFERENCES departments(id),
    salary NUMERIC(10,2),
    manager_id INT REFERENCES employees(id)
);

INSERT INTO departments (id, name) VALUES
    (1, 'Engineering'), (2, 'HR'), (3, 'Finance'), (4, 'Marketing');

INSERT INTO employees (id, name, dept_id, salary, manager_id) VALUES
    (1, 'Nghĩa', 1, 80000, NULL),
    (2, 'An', 1, 65000, 1),
    (3, 'Bình', 2, 55000, 1),
    (4, 'Châu', 3, 70000, NULL),
    (5, 'Dũng', 1, 72000, 1),
    (6, 'Em', NULL, 48000, 4),    -- Không có phòng ban!
    (7, 'Giang', 2, 52000, 3);
```

### 1.1 INNER JOIN — Chỉ lấy MATCHING rows

```sql
-- Chỉ trả về employees CÓ department
SELECT e.name, d.name AS department, e.salary
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- Kết quả:
-- Nghĩa  | Engineering | 80000
-- An      | Engineering | 65000
-- Bình    | HR          | 55000
-- Châu    | Finance     | 70000
-- Dũng    | Engineering | 72000
-- Giang   | HR          | 52000
-- ⚠️ Em KHÔNG xuất hiện (dept_id = NULL)
-- ⚠️ Marketing KHÔNG xuất hiện (không có nhân viên)
```

```
Diagram:
┌──────────┐   ┌──────────┐
│Employees │   │Departments│
│  ┌───┐   │   │   ┌───┐  │
│  │ ● │───│───│───│ ● │  │  ← INNER JOIN: chỉ phần giao
│  └───┘   │   │   └───┘  │
│  Em (∅)  │   │ Marketing │  ← Bị loại
└──────────┘   └──────────┘
```

### 1.2 LEFT JOIN — Tất cả rows bên TRÁI

```sql
-- Tất cả employees, kể cả KHÔNG có department
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;

-- Kết quả:
-- Nghĩa  | Engineering
-- An      | Engineering
-- Bình    | HR
-- Châu    | Finance
-- Dũng    | Engineering
-- Em      | NULL          ← Em vẫn xuất hiện!
-- Giang   | HR

-- Use case: Tìm employees KHÔNG có department
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id
WHERE d.id IS NULL;
-- Em
```

### 1.3 RIGHT JOIN — Tất cả rows bên PHẢI

```sql
-- Tất cả departments, kể cả KHÔNG có employee
SELECT e.name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.id;

-- Kết quả:
-- Nghĩa  | Engineering
-- An      | Engineering
-- Dũng    | Engineering
-- Bình    | HR
-- Giang   | HR
-- Châu    | Finance
-- NULL    | Marketing     ← Marketing vẫn xuất hiện!

-- 💡 Mẹo: RIGHT JOIN ít dùng, có thể viết lại bằng LEFT JOIN
-- Đổi thứ tự bảng:
SELECT e.name, d.name AS department
FROM departments d
LEFT JOIN employees e ON e.dept_id = d.id;
```

### 1.4 FULL OUTER JOIN — Tất cả rows CẢ HAI bên

```sql
-- Tất cả employees + tất cả departments
SELECT e.name, d.name AS department
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;

-- Kết quả:
-- Nghĩa  | Engineering
-- An      | Engineering
-- Dũng    | Engineering
-- Bình    | HR
-- Giang   | HR
-- Châu    | Finance
-- Em      | NULL          ← Employee không có dept
-- NULL    | Marketing     ← Dept không có employee
```

### 1.5 CROSS JOIN — Tích Descartes

```sql
-- Mỗi employee × mỗi department (TẤT CẢ tổ hợp)
SELECT e.name, d.name
FROM employees e
CROSS JOIN departments d;

-- 7 employees × 4 departments = 28 rows!
-- ⚠️ Cẩn thận với bảng lớn!

-- Use case: tạo tất cả combo sizes × colors
SELECT s.name AS size, c.name AS color
FROM sizes s CROSS JOIN colors c;
```

### 1.6 SELF JOIN — Join với CHÍNH mình

```sql
-- Tìm employee và manager
SELECT 
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees e AS m ON e.manager_id = m.id;
-- (Dùng employees cho CẢ HAI phía)

-- Kết quả:
-- An      | Nghĩa    ← An báo cáo Nghĩa
-- Bình    | Nghĩa
-- Dũng    | Nghĩa
-- Em      | Châu     ← Em báo cáo Châu
-- Giang   | Bình
-- Nghĩa   | NULL     ← Nghĩa không có manager (top)
-- Châu    | NULL
```

### 1.7 JOIN nhiều bảng

```sql
-- E-Commerce: Order → Customer + Product
SELECT 
    c.name AS customer,
    o.order_date,
    p.name AS product,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
ORDER BY o.order_date DESC;
```

### 1.8 Tóm Tắt JOINs

```
                  LEFT TABLE    RIGHT TABLE
                  ┌─────────┐   ┌─────────┐
INNER JOIN:       │    ■■■■■│■■■│■■■■■    │   Chỉ matching
LEFT JOIN:        │■■■■■■■■■│■■■│■■■■■    │   Tất cả trái + matching phải
RIGHT JOIN:       │    ■■■■■│■■■│■■■■■■■■■│   Matching trái + tất cả phải
FULL OUTER JOIN:  │■■■■■■■■■│■■■│■■■■■■■■■│   Tất cả cả hai
CROSS JOIN:       Mỗi row trái × mỗi row phải (tích Descartes)
SELF JOIN:        Bảng join với chính nó
```

---

## 2. Subqueries — Truy vấn lồng nhau

### 2.1 Subquery trong WHERE

```sql
-- Employees có lương > trung bình
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Employees trong department 'Engineering'
SELECT name, salary
FROM employees
WHERE dept_id = (SELECT id FROM departments WHERE name = 'Engineering');

-- IN: employees trong nhiều departments
SELECT name, salary
FROM employees
WHERE dept_id IN (
    SELECT id FROM departments WHERE name IN ('Engineering', 'HR')
);

-- NOT IN: employees KHÔNG thuộc department nào có avg salary > 60k
SELECT name
FROM employees
WHERE dept_id NOT IN (
    SELECT dept_id FROM employees
    GROUP BY dept_id
    HAVING AVG(salary) > 60000
);
```

### 2.2 EXISTS / NOT EXISTS

```sql
-- Departments CÓ nhân viên (hiệu quả hơn IN)
SELECT d.name
FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e WHERE e.dept_id = d.id
);

-- Departments KHÔNG CÓ nhân viên
SELECT d.name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1 FROM employees e WHERE e.dept_id = d.id
);
-- → Marketing

-- EXISTS vs IN:
-- EXISTS: dừng sớm khi tìm thấy 1 match → NHANH hơn cho bảng lớn
-- IN: tính toàn bộ subquery → chậm hơn nếu subquery return nhiều rows
```

### 2.3 ALL / ANY

```sql
-- Salary > ALL (tất cả) salary trong HR
-- → Lương lớn hơn MỌI người trong HR
SELECT name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary FROM employees WHERE dept_id = 2
);

-- Salary > ANY (bất kỳ) salary trong HR
-- → Lương lớn hơn ÍT NHẤT 1 người trong HR
SELECT name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary FROM employees WHERE dept_id = 2
);
```

### 2.4 Subquery trong FROM (Derived Table)

```sql
-- Lấy phòng ban có avg salary cao nhất
SELECT * FROM (
    SELECT dept_id, ROUND(AVG(salary), 2) AS avg_salary
    FROM employees
    WHERE dept_id IS NOT NULL
    GROUP BY dept_id
) AS dept_avg
WHERE avg_salary = (
    SELECT MAX(avg_salary) FROM (
        SELECT AVG(salary) AS avg_salary
        FROM employees
        WHERE dept_id IS NOT NULL
        GROUP BY dept_id
    ) AS max_avg
);

-- Đơn giản hơn: top department
SELECT dept_id, ROUND(AVG(salary), 2) AS avg_salary
FROM employees
WHERE dept_id IS NOT NULL
GROUP BY dept_id
ORDER BY avg_salary DESC
LIMIT 1;
```

### 2.5 Subquery trong SELECT (Scalar Subquery)

```sql
-- Mỗi employee kèm avg salary của department mình
SELECT 
    e.name,
    e.salary,
    (SELECT ROUND(AVG(salary), 2) 
     FROM employees e2 
     WHERE e2.dept_id = e.dept_id) AS dept_avg_salary,
    e.salary - (SELECT ROUND(AVG(salary), 2) 
                FROM employees e2 
                WHERE e2.dept_id = e.dept_id) AS diff_from_avg
FROM employees e
WHERE e.dept_id IS NOT NULL;
```

### 2.6 Correlated vs Uncorrelated Subqueries

```sql
-- UNCORRELATED: subquery chạy 1 lần, KHÔNG phụ thuộc outer query
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);  -- 1 lần

-- CORRELATED: subquery chạy MỖI ROW, phụ thuộc outer query
SELECT name, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary) FROM employees e2 
    WHERE e2.dept_id = e1.dept_id  -- Phụ thuộc e1!
);
-- → Employees có lương > avg department MỦA MÌNH

-- ⚠️ Correlated subquery CHẬM hơn (N lần chạy subquery)
-- → Nên dùng JOIN hoặc CTE khi có thể
```

---

## 3. Common Table Expressions (CTEs)

### 3.1 CTE Cơ bản

```sql
-- CTE = "named result set" — dễ đọc hơn subquery
WITH dept_stats AS (
    SELECT 
        dept_id,
        COUNT(*) AS emp_count,
        ROUND(AVG(salary), 2) AS avg_salary,
        SUM(salary) AS total_salary
    FROM employees
    WHERE dept_id IS NOT NULL
    GROUP BY dept_id
)
SELECT d.name, ds.emp_count, ds.avg_salary, ds.total_salary
FROM dept_stats ds
JOIN departments d ON ds.dept_id = d.id
ORDER BY ds.avg_salary DESC;
```

### 3.2 Multiple CTEs

```sql
WITH 
    dept_avg AS (
        SELECT dept_id, AVG(salary) AS avg_salary
        FROM employees
        WHERE dept_id IS NOT NULL
        GROUP BY dept_id
    ),
    high_earners AS (
        SELECT e.*, da.avg_salary AS dept_avg
        FROM employees e
        JOIN dept_avg da ON e.dept_id = da.dept_id
        WHERE e.salary > da.avg_salary
    )
SELECT he.name, he.salary, ROUND(he.dept_avg, 2) AS dept_avg, d.name AS dept
FROM high_earners he
JOIN departments d ON he.dept_id = d.id;
```

### 3.3 Recursive CTE

```sql
-- Hierarchy: Employee → Manager chain
WITH RECURSIVE emp_hierarchy AS (
    -- Base case: top-level (no manager)
    SELECT id, name, manager_id, 1 AS level, 
           name::TEXT AS path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees với manager
    SELECT e.id, e.name, e.manager_id, h.level + 1,
           h.path || ' > ' || e.name
    FROM employees e
    JOIN emp_hierarchy h ON e.manager_id = h.id
)
SELECT name, level, path FROM emp_hierarchy ORDER BY path;

-- Kết quả:
-- Nghĩa  | 1 | Nghĩa
-- An      | 2 | Nghĩa > An
-- Bình    | 2 | Nghĩa > Bình
-- Giang   | 3 | Nghĩa > Bình > Giang
-- Dũng    | 2 | Nghĩa > Dũng
-- Châu    | 1 | Châu
-- Em      | 2 | Châu > Em

-- Use cases:
-- Organizational charts
-- Category trees (Electronics > Phones > iPhone)
-- Bill of Materials
-- File system paths
```

---

## 4. Set Operations

### 4.1 UNION / UNION ALL

```sql
-- UNION: kết hợp kết quả, LOẠI trùng lặp
SELECT name, email FROM customers
UNION
SELECT name, email FROM suppliers;

-- UNION ALL: kết hợp kết quả, GIỮ trùng lặp (NHANH hơn)
SELECT name FROM customers
UNION ALL
SELECT name FROM suppliers;

-- UNION ALL + nguồn gốc
SELECT name, 'customer' AS type FROM customers
UNION ALL
SELECT name, 'supplier' AS type FROM suppliers;
```

### 4.2 INTERSECT

```sql
-- Chỉ lấy rows có trong CẢ HAI queries
SELECT name FROM customers
INTERSECT
SELECT name FROM suppliers;
-- → Người vừa là customer vừa là supplier
```

### 4.3 EXCEPT

```sql
-- Rows trong query 1 NHƯNG KHÔNG trong query 2
SELECT name FROM customers
EXCEPT
SELECT name FROM suppliers;
-- → Customers mà KHÔNG phải supplier
```

---

## 5. Advanced JOIN Patterns

### 5.1 Anti-JOIN (Tìm NOT matching)

```sql
-- Cách 1: LEFT JOIN + IS NULL (⭐ recommended)
SELECT d.name
FROM departments d
LEFT JOIN employees e ON d.id = e.dept_id
WHERE e.id IS NULL;

-- Cách 2: NOT EXISTS
SELECT d.name FROM departments d
WHERE NOT EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.id);

-- Cách 3: NOT IN (⚠️ cẩn thận với NULL!)
SELECT d.name FROM departments d
WHERE d.id NOT IN (SELECT dept_id FROM employees WHERE dept_id IS NOT NULL);
```

### 5.2 Semi-JOIN (Tìm matching, không lấy data bên kia)

```sql
-- Departments có nhân viên (chỉ lấy dept info)
SELECT d.name FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.id);

-- Tương đương DISTINCT:
SELECT DISTINCT d.name
FROM departments d
JOIN employees e ON d.id = e.dept_id;
```

---

## ❓ Câu Hỏi Kiểm Tra

1. INNER JOIN vs LEFT JOIN — khác nhau cơ bản?
2. RIGHT JOIN có cần thiết không? Tại sao?
3. CROSS JOIN tr return bao nhiêu rows? (A rows × B rows)
4. SELF JOIN dùng khi nào? Cho ví dụ.
5. Correlated subquery khác uncorrelated thế nào?
6. EXISTS vs IN — khi nào dùng EXISTS? Tại sao nhanh hơn?
7. CTE khác subquery thế nào? Ưu điểm?
8. Recursive CTE — 2 phần bắt buộc?
9. UNION vs UNION ALL — khi nào dùng ALL?
10. Anti-JOIN pattern là gì? 3 cách viết?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What are the different types of JOINs in SQL?**
> A: INNER JOIN: only matching rows. LEFT JOIN: all left + matching right. RIGHT JOIN: matching left + all right. FULL OUTER: all both sides. CROSS JOIN: cartesian product. SELF JOIN: table with itself. Hay dùng nhất: INNER + LEFT.

> **Q: What is a subquery? When would you use one?**
> A: Query lồng trong query khác. Dùng trong WHERE (filter), FROM (derived table), SELECT (scalar). Correlated subquery phụ thuộc outer query. Uncorrelated chạy 1 lần. Trade-off: readability vs performance. CTE thường tốt hơn.

> **Q: What is a CTE and when would you use it?**
> A: Common Table Expression — named temporary result. `WITH name AS (...)`. Ưu điểm: dễ đọc, tái sử dụng, recursive support. Dùng cho: complex queries, hierarchical data, step-by-step logic. Không persist — chỉ trong 1 query.

> **Q: Explain correlated vs uncorrelated subqueries.**
> A: Uncorrelated: independent, chạy 1 lần. Correlated: phụ thuộc outer row, chạy N lần — O(N × M). Ví dụ: `WHERE salary > (SELECT AVG(salary) FROM emp WHERE dept_id = outer.dept_id)`. Prefer JOINs/CTEs cho performance.

> **Q: How do you find rows in one table but not in another?**
> A: Anti-JOIN pattern. 3 cách: (1) LEFT JOIN + WHERE IS NULL ⭐ nhanh nhất, (2) NOT EXISTS, (3) NOT IN (cẩn thận NULLs). Ví dụ: departments không có employees.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Dùng E-Commerce schema (phần 3). Viết: (a) customers chưa bao giờ order, (b) products chưa bán lần nào, (c) top 5 products bán nhiều nhất + revenue.

**BT2**: Viết recursive CTE hiển thị category tree: Electronics > Phones > iPhone > iPhone 15. (Tự tạo categories table có parent_id.)

**BT3**: Viết query dùng CTE: Tìm employees có lương > avg department mình VÀ > avg toàn công ty. So sánh với subquery version.

**BT4**: Viết 3 subquery exercises: (a) subquery trong WHERE với IN, (b) subquery trong FROM tính rank, (c) correlated subquery tìm second highest salary per department.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-db.md#db04-joins-subqueries](./99-answer-key-db.md#db04-joins-subqueries)
- Bài tập thực hành: [99-answer-key-db.md#db04-joins-subqueries-exercises](./99-answer-key-db.md#db04-joins-subqueries-exercises)
- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db04-joins-subqueries-deep](./97-db-theory-deep-dive.md#db04-joins-subqueries-deep)

