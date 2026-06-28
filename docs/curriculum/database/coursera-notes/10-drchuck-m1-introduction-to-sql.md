# 📗 Module 1: Introduction to SQL (Dr. Chuck — PostgreSQL)

> **Nguồn**: Database Design and Basic SQL in PostgreSQL — Week 1  
> **Giảng viên**: Charles Severance (Dr. Chuck) — University of Michigan  
> **Thời lượng ước tính**: 6 giờ

---

## 🎯 Mục Tiêu

- Hiểu lịch sử và vai trò của relational databases
- Phân biệt relational databases vs flat files
- Hiểu kiến trúc Client-Server của SQL
- Sử dụng `psql` command-line client
- Tạo bảng, insert dữ liệu (CRUD cơ bản)
- Setup môi trường PythonAnywhere hoặc DBeaver

---

## 1. Tại sao học PostgreSQL?

### 1.1 Bối cảnh

```
Vấn đề cốt lõi:
- Dữ liệu quá lớn → không fit vào memory
- Cần truy xuất NHANH trong terabytes dữ liệu
- Cần nhiều người truy cập đồng thời

Giải pháp: Relational Database Management System (RDBMS)
```

### 1.2 Tại sao Postgres mà không phải MySQL?

```
Lịch sử ngắn gọn:
┌─────────────────────────────────────────────────────┐
│ 1. Oracle mua MySQL (2010)                          │
│    → Lo ngại tương lai open source                  │
│                                                     │
│ 2. MariaDB fork ra từ MySQL                         │
│    → Nhưng khó theo kịp MySQL 8 features            │
│                                                     │
│ 3. PostgreSQL = open source THẬT SỰ                 │
│    → Không ai "sở hữu" nó                          │
│    → Features ngang hàng Oracle                     │
│    → Amazon chuyển TOÀN BỘ từ Oracle sang Postgres  │
│                                                     │
│ Kết luận: PostgreSQL là lựa chọn AN TOÀN nhất       │
│ cho tương lai của open-source databases             │
└─────────────────────────────────────────────────────┘
```

### 1.3 So sánh các Database phổ biến

| Database | Loại | Ưu điểm | Nhược điểm |
|----------|------|---------|------------|
| **PostgreSQL** | Open Source | Feature-rich, thật sự free | Ít phổ biến hơn MySQL |
| **MySQL** | Open Source* | Phổ biến nhất | Oracle sở hữu |
| **Oracle** | Commercial | Enterprise, cực mạnh | Đắt, phức tạp |
| **SQL Server** | Commercial | Tích hợp Microsoft | Phí bản quyền |
| **SQLite** | Open Source | Nhẹ, embedded | Không phải server |

> 💡 **Dr. Chuck**: "SQL là ngôn ngữ lập trình YÊU THÍCH nhất của tôi. Nó beautiful vì nó là non-procedural — bạn nói CẦN GÌ, không cần nói LÀM THẾ NÀO."

---

## 2. Lịch sử Relational Databases

### 2.1 Trước khi có Database — Thời đại Magnetic Tape

```
Thời kỳ 1960s-1970s:
┌─────────────────────────────────────────────────────┐
│ Dữ liệu lưu trên BĂNG TỪ (magnetic tape)          │
│                                                     │
│ Vấn đề: Truy cập TUẦN TỰ                          │
│ - Đọc từ đầu đến cuối → mất hàng PHÚT             │
│ - Không thể "nhảy" đến vị trí bất kỳ              │
│                                                     │
│ Giải pháp: Sequential Master Update                 │
│ ┌──────────┐   ┌──────────┐   ┌──────────┐         │
│ │ Old Data │ + │ Sorted   │ → │ New Data │         │
│ │ (Tape)   │   │ Trans.   │   │ (Tape)   │         │
│ └──────────┘   └──────────┘   └──────────┘         │
│                                                     │
│ → Ngân hàng cập nhật số dư VÀO BAN ĐÊM            │
│ → Memory chỉ cần chứa 1 bản ghi tại 1 thời điểm  │
└─────────────────────────────────────────────────────┘
```

### 2.2 Disk Drives thay đổi mọi thứ

```
Disk Drive vs Tape:
- Random access: nhảy đến BẤT KỲ vị trí trong milliseconds
- Seek time + rotational delay thay vì minutes
- SSD còn nhanh hơn: không cần seek/rotation

→ Có thể CẬP NHẬT NGAY LẬP TỨC thay vì đợi đến đêm
→ Nhưng cần SOFTWARE thông minh để tận dụng random access
→ Từ đây → RELATIONAL DATABASE ra đời
```

### 2.3 SQL ra đời — NIST và tiêu chuẩn hóa

```
Vấn đề thập niên 70s:
- IBM có database riêng
- Burroughs có database riêng
- Mỗi vendor có cách khác nhau
- Đổi vendor = viết lại code!

Giải pháp: NIST (National Institute of Standards and Technology)
- Không ÉP cách xây database
- CHỈ YÊU CẦU chuẩn giao tiếp chung
- → SQL = Structured Query Language ra đời

Tại sao SQL đặc biệt:
┌──────────────────────────────────────────────┐
│ NON-PROCEDURAL (Khai báo, không thủ tục)     │
│                                              │
│ Procedural:     "Rẽ trái, đi thẳng,         │
│                  rẽ phải, dừng lại"          │
│                                              │
│ SQL:            "Đưa tôi đến đó"            │
│                 Database tự tìm              │
│                 ĐƯỜNG ĐI TỐI ƯU nhất!       │
│                                              │
│ → Mỗi vendor thi nhau tối ưu bên trong      │
│ → Developers chỉ cần biết SQL               │
└──────────────────────────────────────────────┘
```

> 🎓 **Elizabeth Fong (NIST)**: "Timing is everything. Standardize quá sớm → giết innovation. Quá muộn → quá nhiều lựa chọn. SQL là một trong những success stories."

### 2.4 CRUD — Bốn thao tác cơ bản

```
C.R.U.D = Core operations của MỌI database:

┌──────────┬──────────────────┬──────────────────┐
│ Chữ cái  │ Hành động        │ SQL Command      │
├──────────┼──────────────────┼──────────────────┤
│ C        │ Create (Tạo)     │ INSERT INTO      │
│ R        │ Read (Đọc)       │ SELECT           │
│ U        │ Update (Sửa)     │ UPDATE           │
│ D        │ Delete (Xóa)     │ DELETE FROM      │
└──────────┴──────────────────┴──────────────────┘
```

### 2.5 Thuật ngữ: Lý thuyết vs Thực hành

```
Lý thuyết (Theory)    vs    Thực hành (Practice)
─────────────────────        ──────────────────
Relation               =    Table (Bảng)
Tuple                  =    Row (Dòng)
Attribute              =    Column (Cột)

→ Đừng sợ khi đọc tài liệu thấy thuật ngữ lạ!
→ Spreadsheet analogy: 
  - Tabs dưới cùng = Tables
  - Hàng đầu tiên  = Schema (metadata)
  - Các hàng còn lại = Data rows
```

---

## 3. Kiến trúc Client-Server

### 3.1 Mô hình hoạt động

```
┌──────────────────┐                    ┌──────────────────┐
│     CLIENT       │  SQL Commands      │     SERVER       │
│   (psql, app)    │ ─────────────────→ │   (PostgreSQL)   │
│                  │                    │                  │
│  Nhập SQL        │ ←───────────────── │  ⭐ Magic:       │
│  Hiển thị kết quả│  Results/Data      │  - Indexes       │
│                  │                    │  - Caching       │
└──────────────────┘                    │  - Optimization  │
                                        │  - Disk I/O      │
Các loại Client:                        └──────────────────┘
1. psql (command line) ⭐ recommend
2. DBeaver (desktop app)
3. pgAdmin (web-based — hạn chế với shared server)
```

### 3.2 Kết nối Database

```bash
# Thông tin kết nối:
Host:     pg.pg4e.com
Port:     5432
Database: pg4e_8675309
User:     pg4e_8675309
Password: pg4e_p_422398745954

# Kết nối bằng psql:
psql -h pg.pg4e.com -p 5432 -U pg4e_8675309 pg4e_8675309
# → Nhập password khi được hỏi
```

### 3.3 Superuser vs Normal User

```sql
-- Đăng nhập superuser (postgres)
-- Prompt: postgres=#  (dấu # = superuser)

-- Tạo user mới
CREATE USER pg4e WITH PASSWORD 'secret';

-- Tạo database
CREATE DATABASE people WITH OWNER pg4e;

-- Thoát superuser session
\q

-- Đăng nhập bằng user thường
psql people pg4e
-- Prompt: people=>  (dấu > = normal user)

-- ⚠️ NGUYÊN TẮC: Dùng superuser CÀO ÍT CÀG TỐT
-- Tạo user + database xong → THOÁT superuser
-- Làm mọi thứ bằng normal user
```

---

## 4. Các lệnh psql quan trọng

```bash
# Lệnh psql (bắt đầu bằng \, KHÔNG phải SQL)
\l          # List tất cả databases
\dt         # List tất cả tables
\d+ users   # Show schema chi tiết của table users
\q          # Thoát psql
\i file.sql # Chạy SQL từ file
\?          # Help psql commands

# Chú ý: 
# - Lệnh \... là lệnh psql (client-specific)
# - Lệnh SQL (SELECT, INSERT...) là universal
```

---

## 5. Tạo Table đầu tiên

### 5.1 CREATE TABLE

```sql
-- Syntax cơ bản:
CREATE TABLE users (
    name VARCHAR(128),
    email VARCHAR(128)
);

-- Giải thích:
-- users         → tên bảng
-- name          → tên cột
-- VARCHAR(128)  → kiểu dữ liệu: chuỗi tối đa 128 ký tự
-- → Đây là CONTRACT với database!

-- Kiểm tra:
\dt
-- → Thấy bảng users trong danh sách

\d+ users
-- → Xem schema chi tiết
```

> ⚠️ **Quan trọng**: VARCHAR(128) nghĩa là bạn HỨA không bao giờ lưu quá 128 ký tự. Database tin bạn và tối ưu dựa trên "hợp đồng" này. Nếu vi phạm → ERROR!

### 5.2 INSERT INTO

```sql
-- Insert một row:
INSERT INTO users (name, email) 
VALUES ('Chuck', 'csev@umich.edu');

INSERT INTO users (name, email) 
VALUES ('Colleen', 'cvl@umich.edu');

INSERT INTO users (name, email) 
VALUES ('Ted', 'ted@umich.edu');

INSERT INTO users (name, email) 
VALUES ('Sally', 'sally@umich.edu');
```

### 5.3 SELECT (Read)

```sql
-- Đọc tất cả
SELECT * FROM users;

-- Output:
--   name   |     email
-- ---------+------------------
--  Chuck   | csev@umich.edu
--  Colleen | cvl@umich.edu
--  Ted     | ted@umich.edu
--  Sally   | sally@umich.edu
```

### 5.4 DELETE

```sql
-- ⚠️ LUÔN có WHERE clause!
DELETE FROM users WHERE email = 'ted@umich.edu';

-- Không có WHERE → XÓA TẤT CẢ rows!
-- DELETE FROM users;  -- 💀 NGUY HIỂM!

-- Cách hiểu: Loop ngầm + IF
-- "Duyệt qua TẤT CẢ rows, NẾU email = ... thì xóa"
-- Database KHÔNG thực sự loop — nó dùng index!
```

### 5.5 UPDATE

```sql
-- Update với WHERE
UPDATE users SET name = 'Charles'
WHERE email = 'csev@umich.edu';

-- ⚠️ Không có WHERE → update TẤT CẢ rows!
-- UPDATE users SET name = 'Unknown';  -- 💀

-- Có thể update nhiều cột:
-- UPDATE users SET name = 'Charles', email = 'new@umich.edu'
-- WHERE email = 'csev@umich.edu';
```

---

## 6. Sắp xếp và Lọc dữ liệu

### 6.1 ORDER BY

```sql
-- Tăng dần (mặc định)
SELECT * FROM users ORDER BY email;

-- Giảm dần
SELECT * FROM users ORDER BY email DESC;
```

### 6.2 WHERE với LIKE (Wildcards)

```sql
-- % = match bất kỳ chuỗi nào
SELECT * FROM users WHERE name LIKE '%e%';
-- → Tìm tên có chữ 'e' bất kỳ đâu

-- ⚠️ LIKE '%text%' → FULL TABLE SCAN (chậm!)
-- Vì không thể dùng index cho wildcard ở đầu
-- LIKE 'text%' → CÓ THỂ dùng index (prefix lookup)
```

### 6.3 LIMIT và OFFSET (Phân trang)

```sql
-- Lấy 2 rows đầu tiên
SELECT * FROM users ORDER BY email LIMIT 2;

-- Bỏ qua 1 row, lấy 2 row tiếp theo
SELECT * FROM users ORDER BY email LIMIT 2 OFFSET 1;

-- Phân trang: Page N (0-indexed offset)
-- OFFSET = (page_number) * page_size
-- LIMIT  = page_size

-- ⚠️ OFFSET bắt đầu từ 0 (giống Python list)
```

### 6.4 COUNT

```sql
-- Đếm tất cả rows
SELECT COUNT(*) FROM users;

-- Đếm với điều kiện
SELECT COUNT(*) FROM users WHERE email = 'csev@umich.edu';

-- COUNT hiệu quả hơn SELECT rồi đếm bằng code
-- Vì database có thể biết count MÀ KHÔNG đọc dữ liệu
```

---

## 7. Setup môi trường

### 7.1 PythonAnywhere (Khuyến nghị — Free)

```
Bước 1: Đăng ký tài khoản miễn phí tại www.pythonanywhere.com
Bước 2: Mở Console → Start Bash console
Bước 3: psql đã được cài sẵn!

Lệnh Linux cơ bản:
  pwd        → hiện thư mục hiện tại
  ls         → liệt kê files
  cd folder  → vào thư mục
  cd ~       → về home directory
  cat file   → xem nội dung file
  clear      → xóa màn hình
  Tab        → auto-complete tên file
  ↑↓         → lịch sử lệnh đã gõ

Download file:
  wget URL          → tải file từ internet
  curl -o file URL  → tương tự wget
```

### 7.2 DBeaver (Desktop — Optional)

```
- Free, open source
- Download: https://dbeaver.io/
- Hỗ trợ nhiều loại database
- Tạo connection: nhập Host, Database, User, Password
- ⚠️ pgAdmin KHÔNG hoạt động tốt với shared server
  (lỗi 'no access to pg_roles')
```

### 7.3 Cài đặt trên máy cá nhân (Optional)

```bash
# Cài psql (command-line client):
# MacOS: brew install libpq
# Linux: apt-get install postgresql-client
# Windows: https://www.postgresql.org/download/

# Cài psycopg2 (Python connector — dùng sau):
pip3 install psycopg2

# Kiểm tra:
python3
>>> import psycopg2  # Không lỗi = OK
```

---

## 8. Cách Assignments hoạt động

```
Quy trình:
┌──────────────────┐    SQL     ┌──────────────────┐
│   Bạn (Client)   │ ────────→ │  Database Server  │
│                  │           │  (pg.pg4e.com)    │
│  psql / DBeaver  │           │                  │
└──────────────────┘           └──────────────────┘
         ↑                              ↑
         │                              │
         │  Check Answer                │
         │ ─────────────────────→ ┌──────────────┐
         │                        │  Autograder  │
         └──────────── Grade ←── │  (pg4e.com)  │
                                  └──────────────┘

1. Bạn nhận Host/Port/Database/User/Password
2. Kết nối từ BẤT KỲ client nào có internet
3. Chạy SQL commands để hoàn thành assignment
4. Autograder kết nối vào DATABASE CỦA BẠN
5. Kiểm tra xem bạn đã làm đúng chưa
6. Gửi điểm về gradebook
```

---

## ❓ Câu Hỏi Kiểm Tra

1. Tại sao PostgreSQL ngày càng phổ biến hơn MySQL? Kể 3 lý do.
2. Sequential Master Update hoạt động thế nào? Tại sao nó cần dữ liệu SORTED?
3. SQL là gì? Tại sao gọi là "non-procedural"? Cho ví dụ so sánh.
4. CRUD viết tắt của gì? SQL command tương ứng?
5. Phân biệt Relation/Tuple/Attribute với Table/Row/Column.
6. Client-Server trong PostgreSQL hoạt động thế nào?
7. Superuser prompt (`#`) khác gì normal user prompt (`>`)? Khi nào dùng?
8. `DELETE FROM users` (không WHERE) sẽ xảy ra gì?
9. `LIKE '%e%'` tại sao chậm hơn `WHERE email = 'x@y.com'`?
10. `LIMIT 5 OFFSET 10` lấy rows nào? (0-indexed hay 1-indexed?)

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: Why PostgreSQL over MySQL?**  
> A: PostgreSQL là truly open-source (không bị vendor nào sở hữu), feature-rich ngang Oracle, được Amazon/Apple/Instagram sử dụng. MySQL bị Oracle mua năm 2010, tạo lo ngại về tương lai open-source.

> **Q: Explain the client-server architecture of PostgreSQL.**  
> A: PostgreSQL hoạt động theo mô hình client-server. Server process (postmaster) quản lý database files, chấp nhận connections từ clients (psql, DBeaver, applications). Clients gửi SQL commands, server xử lý và trả kết quả. Client và server có thể chạy trên cùng máy hoặc khác máy qua network.

> **Q: What is CRUD?**  
> A: CRUD = Create (INSERT), Read (SELECT), Update (UPDATE), Delete (DELETE). Đây là 4 thao tác cơ bản mà mọi data storage system cần hỗ trợ. SQL cung cấp cú pháp khai báo (declarative) cho mỗi thao tác.

> **Q: What is the difference between SQL and procedural languages?**  
> A: SQL là declarative/non-procedural — bạn mô tả KẾT QUẢ mong muốn (WHAT), không cần viết CÁCH thực hiện (HOW). Database optimizer tự quyết execution plan tối ưu. Ngược lại, procedural languages (Python, C#) yêu cầu bạn viết từng bước cụ thể.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Kết nối PostgreSQL qua PythonAnywhere. Tạo bảng `students` với cột `name VARCHAR(100)` và `grade CHAR(1)`. Insert 5 sinh viên. SELECT tất cả, rồi xóa 1 sinh viên.

**BT2**: Tạo bảng `products (name VARCHAR(100), price INTEGER, category VARCHAR(50))`. Insert 10 sản phẩm. Viết query: (a) sắp theo giá giảm dần, (b) tìm sản phẩm có tên chứa chữ "Pro", (c) top 3 đắt nhất, (d) đếm tổng sản phẩm.

**BT3**: Thực hành Linux trên PythonAnywhere: (a) tạo file `homework.sql` chứa các lệnh SQL, (b) chạy file bằng `\i homework.sql` trong psql.

---

## ✅ Checklist Hoàn Thành Module 1

1. Giải thích được tại sao cần database thay vì đọc file tuần tự.
2. Kết nối thành công PostgreSQL từ psql hoặc DBeaver.
3. Tạo bảng, insert dữ liệu, SELECT, UPDATE, DELETE thành thạo.
4. Sử dụng WHERE, ORDER BY, LIMIT, OFFSET.
5. Phân biệt lệnh psql (`\dt`, `\q`) vs SQL (`SELECT`, `INSERT`).
6. Hiểu kiến trúc Client-Server và vai trò của superuser.
7. Hoàn thành tất cả assignments trên Coursera.

---

## 📎 Tham Khảo

- Đáp Án: [99-answer-key-db.md#drchuck-m1](./99-answer-key-db.md#drchuck-m1)
- Lý thuyết sâu: [97-db-theory-deep-dive.md#drchuck-m1-deep](./97-db-theory-deep-dive.md#drchuck-m1-deep)
