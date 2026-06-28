# 📘 Phần 1: Relational Database Concepts

> **Nguồn**: IBM Course Module 1 + bổ sung kiến thức
> **Thời lượng ước tính**: 4 giờ

---

## 🎯 Mục Tiêu

- Hiểu data vs information, database vs DBMS
- Phân biệt Relational vs Non-relational databases
- Nắm vững các Data Models (Hierarchical, Network, Relational)
- Vẽ và đọc Entity-Relationship Diagrams (ERD)
- Hiểu ACID properties
- So sánh PostgreSQL, MySQL, SQL Server, SQLite

---

## 1. Data Fundamentals

### 1.1 Data vs Information

```
Data (Dữ liệu):
  - Các giá trị thô, chưa xử lý
  - Ví dụ: "25", "Nghĩa", "2026-02-22"

Information (Thông tin):
  - Data đã được xử lý, có ý nghĩa
  - Ví dụ: "Nghĩa, 25 tuổi, đăng ký ngày 22/02/2026"

Data → Xử lý → Information → Phân tích → Knowledge → Hành động
```

### 1.2 Database là gì?

```
Database (Cơ sở dữ liệu):
  - Tập hợp dữ liệu có tổ chức
  - Lưu trữ, truy xuất, và quản lý hiệu quả
  - Ví dụ: danh sách sinh viên, đơn hàng, sản phẩm

DBMS (Database Management System):
  - Phần mềm quản lý database
  - Cung cấp interface để CRUD (Create, Read, Update, Delete)
  - Ví dụ: PostgreSQL, MySQL, SQL Server, Oracle, SQLite

RDBMS (Relational DBMS):
  - DBMS dùng relational model (bảng, hàng, cột)
  - Dữ liệu liên kết qua keys
  - Truy vấn bằng SQL
```

---

## 2. Data Models

### 2.1 Hierarchical Model (Mô hình phân cấp)

```
          CEO
         / | \
       VP1 VP2 VP3
      / \     |
    Dir1 Dir2 Dir3

- Cấu trúc cây (tree)
- Mỗi node có 1 parent, nhiều children
- Ví dụ: XML, file system
- ❌ Hạn chế: không hỗ trợ M:N relationships
```

### 2.2 Network Model (Mô hình mạng)

```
    Student1 ──── Course1
       \     \/     /
        \   /  \   /
    Student2 ──── Course2

- Mở rộng hierarchical: node có thể có NHIỀU parents
- Linh hoạt hơn nhưng PHỨC TẠP hơn
- ❌ Hạn chế: khó thay đổi cấu trúc
```

### 2.3 Relational Model (Mô hình quan hệ) ⭐

```
┌──────────────────────────────────┐
│         Students Table           │
├────┬──────────┬─────┬────────────┤
│ id │ name     │ age │ class_id   │
├────┼──────────┼─────┼────────────┤
│ 1  │ Nghĩa   │ 25  │ 101        │
│ 2  │ An      │ 22  │ 102        │
│ 3  │ Bình    │ 23  │ 101        │
└────┴──────────┴─────┴────────────┘
         │ class_id (FK)
         ▼
┌──────────────────────────────────┐
│          Classes Table           │
├─────┬────────────────────────────┤
│ id  │ name                       │
├─────┼────────────────────────────┤
│ 101 │ Computer Science           │
│ 102 │ Mathematics                │
└─────┴────────────────────────────┘

- Dữ liệu tổ chức thành BẢNG (tables/relations)
- Mỗi bảng có HÀNG (rows/tuples) và CỘT (columns/attributes)
- Liên kết giữa bảng qua KEYS
- Truy vấn bằng SQL
- ✅ Chuẩn hóa được, dễ thay đổi cấu trúc
```

### 2.4 So sánh: SQL vs NoSQL

| Đặc điểm | SQL (Relational) | NoSQL (Non-Relational) |
|-----------|-----------------|----------------------|
| Cấu trúc | Bảng cố định (schema) | Linh hoạt (schemaless) |
| Ngôn ngữ | SQL | Khác nhau tùy DB |
| ACID | ✅ Đầy đủ | ❌ Thường chỉ eventual consistency |
| Quan hệ | ✅ Foreign keys, JOINs | ❌ Embedded/Reference |
| Scale | Vertical (scale up) | Horizontal (scale out) |
| Ví dụ | PostgreSQL, MySQL | MongoDB, Redis, Cassandra |
| Dùng khi | Dữ liệu có cấu trúc rõ ràng | Big data, real-time, flexible schema |

---

## 3. Relational Model Concepts

### 3.1 Thuật ngữ

```
Table / Relation:    Bảng chứa dữ liệu
Row / Tuple:         Một bản ghi (1 hàng)
Column / Attribute:  Một trường dữ liệu (1 cột)
Schema:              Cấu trúc/blueprint của database
Domain:              Tập giá trị hợp lệ cho 1 cột
Degree:              Số cột trong bảng
Cardinality:         Số hàng trong bảng
```

### 3.2 Keys (Khóa)

```sql
-- Primary Key (PK): Định danh DUY NHẤT mỗi hàng
-- KHÔNG trùng, KHÔNG NULL
CREATE TABLE students (
    id SERIAL PRIMARY KEY,        -- PK: auto-increment
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE     -- UNIQUE ≠ PK (có thể nhiều UNIQUE)
);

-- Foreign Key (FK): Tham chiếu đến PK của bảng khác
-- Tạo LIÊN KẾT giữa các bảng
CREATE TABLE enrollments (
    id SERIAL PRIMARY KEY,
    student_id INT REFERENCES students(id),  -- FK → students.id
    course_id INT REFERENCES courses(id),    -- FK → courses.id
    enrolled_at DATE DEFAULT CURRENT_DATE
);

-- Composite Key: PK gồm NHIỀU cột
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)  -- 2 cột kết hợp = PK
);
```

### 3.3 Constraints (Ràng buộc)

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,                         -- PK
    name VARCHAR(200) NOT NULL,                    -- Bắt buộc có giá trị
    price DECIMAL(10,2) CHECK (price > 0),         -- Phải > 0
    category VARCHAR(50) DEFAULT 'Uncategorized',  -- Giá trị mặc định
    sku VARCHAR(20) UNIQUE,                        -- Không trùng lặp
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tóm tắt Constraints:
-- NOT NULL:    Không cho phép NULL
-- UNIQUE:     Giá trị không trùng lặp
-- PRIMARY KEY: NOT NULL + UNIQUE (1 bảng 1 PK)
-- FOREIGN KEY: Tham chiếu đến PK bảng khác
-- CHECK:       Điều kiện phải thỏa mãn
-- DEFAULT:     Giá trị mặc định khi không chỉ định
```

---

## 4. Entity-Relationship Diagram (ERD)

### 4.1 Thành phần ERD

```
┌─────────────┐
│   Entity     │  → Hình chữ nhật (bảng)
│  (Student)   │
└─────────────┘

╱ Attribute ╲   → Hình oval (cột)
╲  (name)   ╱

◇ Relationship ◇ → Hình thoi (quan hệ)
  (enrolls_in)
```

### 4.2 Cardinality (Số lượng quan hệ)

```
One-to-One (1:1):
  Person ──── Passport
  Mỗi người có ĐÚNG 1 hộ chiếu

One-to-Many (1:N):
  Department ──<< Employee
  1 phòng ban có NHIỀU nhân viên
  1 nhân viên thuộc 1 phòng ban

Many-to-Many (M:N):
  Student >>──<< Course
  1 sinh viên học NHIỀU khóa
  1 khóa có NHIỀU sinh viên
  → Cần JUNCTION TABLE (bảng trung gian)
```

### 4.3 Ví dụ ERD — Hệ thống E-Commerce

```
┌──────────────┐     1:N     ┌──────────────┐
│   Customer   │────────────>│    Order      │
│──────────────│             │──────────────│
│ PK: id       │             │ PK: id       │
│ name         │             │ FK: cust_id  │
│ email        │             │ order_date   │
│ phone        │             │ total        │
└──────────────┘             └──────┬───────┘
                                    │ 1:N
                                    ▼
                             ┌──────────────┐     N:1     ┌──────────────┐
                             │ Order_Items   │────────────>│   Product    │
                             │──────────────│             │──────────────│
                             │ FK: order_id │             │ PK: id       │
                             │ FK: prod_id  │             │ name         │
                             │ quantity     │             │ price        │
                             │ unit_price   │             │ category     │
                             └──────────────┘             └──────────────┘
```

---

## 5. ACID Properties

```
A - Atomicity (Tính nguyên tử):
    Giao dịch hoặc THÀNH CÔNG HOÀN TOÀN hoặc THẤT BẠI HOÀN TOÀN.
    Ví dụ: Chuyển tiền — trừ A VÀ cộng B, không chỉ làm 1 nửa.

C - Consistency (Tính nhất quán):
    Database luôn ở trạng thái hợp lệ trước VÀ sau giao dịch.
    Ví dụ: Tổng tiền trước và sau chuyển khoản phải BẰNG NHAU.

I - Isolation (Tính cô lập):
    Các giao dịch đồng thời KHÔNG ảnh hưởng nhau.
    Ví dụ: 2 người cùng mua hàng cuối cùng → chỉ 1 người thành công.

D - Durability (Tính bền vững):
    Sau khi commit, dữ liệu KHÔNG BỊ MẤT kể cả khi crash.
    Ví dụ: Sau khi chuyển tiền thành công → mất điện → tiền vẫn đúng.
```

```sql
-- Ví dụ ACID: Chuyển 1 triệu từ A sang B
BEGIN;  -- Bắt đầu giao dịch

UPDATE accounts SET balance = balance - 1000000
WHERE id = 1;  -- Trừ tiền A

UPDATE accounts SET balance = balance + 1000000
WHERE id = 2;  -- Cộng tiền B

COMMIT;  -- Xác nhận → cả 2 lệnh được thực hiện

-- Nếu lỗi ở giữa:
ROLLBACK;  -- Huỷ tất cả → A và B giữ nguyên
```

---

## 6. Database Architecture

### 6.1 Client-Server Model

```
┌────────────┐     SQL Query     ┌─────────────────┐
│   Client    │ ────────────────> │   Database       │
│ (App, Tool) │ <──────────────── │   Server         │
│             │     Result Set    │  (PostgreSQL)    │
└────────────┘                    └─────────────────┘

Clients:
  - psql (command line)
  - pgAdmin (GUI)
  - DBeaver (IDE)
  - Application code (C#, Python, etc.)
```

### 6.2 Tiered Architecture

```
┌─────────────────┐
│  Presentation   │  ← UI / API (ASP.NET, React)
├─────────────────┤
│  Business Logic  │  ← C# Services, Validation
├─────────────────┤
│  Data Access     │  ← Entity Framework / ADO.NET
├─────────────────┤
│  Database        │  ← PostgreSQL, MySQL
└─────────────────┘
```

---

## 7. RDBMS Comparison

| Đặc điểm | PostgreSQL | MySQL | SQL Server | SQLite |
|-----------|-----------|-------|------------|--------|
| **License** | Open Source | Open Source (Oracle) | Commercial (MS) | Public Domain |
| **ACID** | ✅ Full | ✅ (InnoDB) | ✅ Full | ✅ Full |
| **JSON support** | ✅ Xuất sắc | ✅ Tốt | ✅ Tốt | ❌ Giới hạn |
| **Full-text search** | ✅ Built-in | ✅ Built-in | ✅ Built-in | ❌ Extension |
| **Window functions** | ✅ Full | ✅ MySQL 8+ | ✅ Full | ✅ SQLite 3.25+ |
| **Extensions** | ✅ rất phong phú | ❌ Ít | ❌ Ít | ❌ Ít |
| **Performance** | Xuất sắc (complex queries) | Nhanh (simple reads) | Tốt (enterprise) | Nhanh (embedded) |
| **Dùng khi** | Web apps, analytics | Web apps (LAMP) | Enterprise (.NET) | Mobile, embedded, dev |
| **C# Integration** | ✅ Npgsql / EF Core | ✅ MySqlConnector | ✅ Native | ✅ Microsoft.Data.Sqlite |

### Chọn RDBMS nào?

```
PostgreSQL  → Mặc định tốt nhất cho hầu hết projects
              Mạnh: extensions, JSON, full-text search, CTE
              
MySQL       → Web apps đơn giản, WordPress, legacy systems
              Mạnh: tốc độ đọc, phổ biến, hosting hỗ trợ rộng

SQL Server  → Enterprise .NET, tích hợp Azure
              Mạnh: tooling, SSMS, BI integration

SQLite      → Mobile apps, embedded, prototyping, testing
              Mạnh: zero-config, đơn giản, file-based
```

---

## 8. PostgreSQL Setup

### 8.1 Cài đặt

```bash
# Windows: Download từ https://www.postgresql.org/download/
# Hoặc dùng Chocolatey:
choco install postgresql

# macOS:
brew install postgresql@16

# Linux (Ubuntu/Debian):
sudo apt install postgresql postgresql-contrib
```

### 8.2 Kết nối bằng psql

```bash
# Kết nối local
psql -U postgres

# Kết nối với database cụ thể
psql -U postgres -d mydb

# Các lệnh psql thường dùng
\l          -- Liệt kê databases
\c mydb     -- Chuyển sang database mydb
\dt         -- Liệt kê tables
\d students -- Mô tả cấu trúc table students
\q          -- Thoát
\?          -- Help
```

### 8.3 Tạo Database đầu tiên

```sql
-- Tạo database
CREATE DATABASE learning_db;

-- Kết nối
\c learning_db

-- Tạo bảng đầu tiên
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT CHECK (age >= 16 AND age <= 100),
    enrolled_at DATE DEFAULT CURRENT_DATE
);

-- Chèn dữ liệu
INSERT INTO students (name, email, age)
VALUES 
    ('Nghĩa', 'nghia@example.com', 25),
    ('An', 'an@example.com', 22),
    ('Bình', 'binh@example.com', 23);

-- Truy vấn
SELECT * FROM students;
```

---

## ❓ Câu Hỏi Kiểm Tra

1. Database khác DBMS thế nào?
2. Relational model tổ chức dữ liệu thế nào? (Table, Row, Column)
3. Primary Key vs Foreign Key — vai trò?
4. Kể 3 loại Data Models — ưu nhược điểm?
5. ACID là gì? Giải thích từng chữ.
6. SQL vs NoSQL — khi nào dùng cái nào?
7. ERD có mấy thành phần chính?
8. 1:1, 1:N, M:N — cho ví dụ thực tế mỗi loại.
9. PostgreSQL khác MySQL ở điểm nào?
10. Constraint nào kiểm tra GIÁ TRỊ, constraint nào kiểm tra SỰ TỒN TẠI?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is a Relational Database?**
> A: Database tổ chức dữ liệu thành tables (relations). Tables có rows (records) và columns (attributes). Tables liên kết qua foreign keys. Truy vấn bằng SQL. Tuân thủ ACID. Ví dụ: PostgreSQL, MySQL, SQL Server.

> **Q: What are the ACID properties?**
> A: **Atomicity**: all-or-nothing. **Consistency**: valid state before/after. **Isolation**: concurrent transactions don't interfere. **Durability**: committed data survives crashes. Đảm bảo reliability cho banking, e-commerce.

> **Q: What is the difference between SQL and NoSQL?**
> A: SQL: structured schema, tables, ACID, vertical scaling. NoSQL: flexible schema, documents/key-value/graph, eventual consistency, horizontal scaling. Chọn SQL cho data integrity (banking), NoSQL cho scalability (social media feeds).

> **Q: What is normalization? Why is it important?**
> A: Quá trình tổ chức dữ liệu để giảm redundancy và dependency. Tách data vào multiple tables liên kết bằng keys. Đảm bảo consistency, tiết kiệm storage, tránh update anomalies. Trade-off: nhiều JOINs hơn.

> **Q: What is an Entity-Relationship Diagram?**
> A: Visual representation của database structure. Entities = tables, Attributes = columns, Relationships = how tables connect (1:1, 1:N, M:N). Dùng ở design phase trước khi viết SQL.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Vẽ ERD cho hệ thống thư viện: Books, Authors, Members, Loans. Xác định quan hệ 1:N và M:N.

**BT2**: Cài PostgreSQL, tạo database `bookstore`, tạo bảng `books` với PK, constraints (price > 0, title NOT NULL). Insert 5 books.

**BT3**: So sánh PostgreSQL vs MySQL cho 3 use cases: blog, banking app, IoT sensor data.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-db.md#db01-relational-concepts](./99-answer-key-db.md#db01-relational-concepts)
- Bài tập thực hành: [99-answer-key-db.md#db01-relational-concepts-exercises](./99-answer-key-db.md#db01-relational-concepts-exercises)
- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db01-relational-concepts-deep](./97-db-theory-deep-dive.md#db01-relational-concepts-deep)

---

## 📚 Lịch Sử Databases — Từ Băng Từ Đến SQL

### Trước khi có Database — Thời đại Magnetic Tape (1960s)

```
Dữ liệu lưu trên BĂNG TỪ (magnetic tape)
→ Truy cập TUẦN TỰ: đọc từ đầu đến cuối → mất hàng PHÚT
→ Giải pháp: Sequential Master Update
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ Old Data │ + │ Sorted   │ → │ New Data │
  │ (Tape)   │   │ Trans.   │   │ (Tape)   │
  └──────────┘   └──────────┘   └──────────┘
→ Ngân hàng cập nhật số dư VÀO BAN ĐÊM
```

### Disk Drives → Random Access → Database ra đời

```
Disk Drive: nhảy đến BẤT KỲ vị trí trong milliseconds
→ Cập nhật NGAY LẬP TỨC thay vì đợi đến đêm
→ Cần SOFTWARE thông minh → RELATIONAL DATABASE ra đời
```

### NIST và SQL — Chuẩn hóa giao tiếp

```
Thập niên 70s: Mỗi vendor có database riêng → đổi vendor = viết lại code!
NIST chỉ yêu cầu CHUẨN GIAO TIẾP → SQL ra đời

SQL = NON-PROCEDURAL (Khai báo):
  Procedural: "Rẽ trái, đi thẳng, rẽ phải, dừng lại"
  SQL:        "Đưa tôi đến đó" → Database tự tìm ĐƯỜNG TỐI ƯU
```

> 💡 **Dr. Chuck**: "SQL là ngôn ngữ yêu thích nhất của tôi. Beautiful vì nó là non-procedural."

---

## 📚 Setup PostgreSQL

### PythonAnywhere (Free)
```
1. Đăng ký: www.pythonanywhere.com (miễn phí)
2. Mở Console → Start Bash → psql đã cài sẵn!
```

### DBeaver (Desktop, Free)
```
1. Download: https://dbeaver.io/
2. Hỗ trợ PostgreSQL, MySQL, Oracle, SQLite
3. ⚠️ pgAdmin KHÔNG hoạt động tốt với shared server
```

### Cài trên máy cá nhân
```bash
# MacOS: brew install postgresql
# Linux: sudo apt-get install postgresql postgresql-client
# Windows: https://www.postgresql.org/download/windows/
```

- Đọc sâu lý thuyết: [97-db-theory-deep-dive.md#db01-relational-concepts-deep](./97-db-theory-deep-dive.md#db01-relational-concepts-deep)

