# 📗 Module 2: Single Table SQL (Dr. Chuck — PostgreSQL)

> **Nguồn**: Database Design and Basic SQL in PostgreSQL — Week 2  
> **Giảng viên**: Charles Severance (Dr. Chuck) — University of Michigan  
> **Thời lượng ước tính**: 3 giờ

---

## 🎯 Mục Tiêu

- Hiểu sâu các lệnh CRUD trong ngữ cảnh single table
- Nắm rõ Data Types trong PostgreSQL (varchar, text, numeric, date)
- Hiểu concept Primary Key, Logical Key, và Index
- Phân biệt B-Tree Index vs Hash Index
- Import dữ liệu CSV vào PostgreSQL bằng `\copy`
- Viết thành thạo WHERE, ORDER BY, LIKE, LIMIT/OFFSET, COUNT

---

## 1. Xem lại CRUD — Hiểu sâu hơn

### 1.1 INSERT INTO — Implied Contract

```sql
-- Cú pháp INSERT:
INSERT INTO users (name, email) 
VALUES ('Chuck', 'csev@umich.edu');

-- Số cột = Số giá trị (1-to-1 correspondence)
-- 2 cột (name, email) → phải có 2 giá trị
-- Luôn kết thúc bằng dấu chấm phẩy ;
```

### 1.2 DELETE — Implied Loop + IF

```sql
-- DELETE = Loop ngầm + IF statement
DELETE FROM users WHERE email = 'ted@umich.edu';

-- Cách hiểu:
-- FOR mỗi row IN users:
--     IF row.email == 'ted@umich.edu':
--         DELETE row

-- ⚠️ Không có WHERE = XÓA TẤT CẢ!
-- DELETE FROM users;  -- 💀 Xóa sạch bảng!

-- Nhưng database KHÔNG thực sự loop!
-- Với index → tìm trực tiếp, thay đổi 1 mark
-- Vs Sequential Master Update: phải copy TOÀN BỘ → mất hàng giờ
```

### 1.3 UPDATE — Implied Loop + IF + SET

```sql
UPDATE users SET name = 'Charles'
WHERE email = 'csev@umich.edu';

-- Cách hiểu:
-- FOR mỗi row IN users:
--     IF row.email == 'csev@umich.edu':
--         row.name = 'Charles'

-- Có thể update NHIỀU cột:
UPDATE users SET name = 'Charles', email = 'new@umich.edu'
WHERE email = 'csev@umich.edu';

-- ⚠️ Nếu WHERE match NHIỀU rows → update TẤT CẢ matching rows
-- Ví dụ: WHERE email LIKE '%@umich.edu' → update hàng nghìn!

-- Database trả về số rows affected:
-- "UPDATE 1" hoặc "UPDATE 11"
```

### 1.4 SELECT — Truy vấn dữ liệu

```sql
-- Lấy tất cả
SELECT * FROM users;

-- * = tất cả cột
-- Chọn cột cụ thể:
SELECT name, email FROM users;

-- CRUD theo thứ tự:
-- Create  → INSERT INTO
-- Read    → SELECT
-- Update  → UPDATE
-- Delete  → DELETE FROM
```

> 💡 **Dr. Chuck**: "SQL có implied loop ở MỌI câu lệnh. DELETE, UPDATE, SELECT đều có loop ngầm đi qua tất cả rows. WHERE là cách bạn thêm IF statement vào loop đó."

---

## 2. Lọc và Sắp xếp

### 2.1 WHERE — Bộ lọc

```sql
-- Exact match (dùng được INDEX → nhanh!)
SELECT * FROM users WHERE email = 'csev@umich.edu';

-- Comparison operators: =, <>, <, >, <=, >=
```

### 2.2 ORDER BY — Sắp xếp

```sql
-- Tăng dần (mặc định)
SELECT * FROM users ORDER BY email;

-- Giảm dần
SELECT * FROM users ORDER BY email DESC;
```

### 2.3 LIKE — Tìm kiếm Pattern

```sql
-- % = wildcard (bất kỳ chuỗi nào)
SELECT * FROM users WHERE name LIKE '%e%';
-- → Tìm tên có chữ 'e' ở bất kỳ đâu

-- ⚠️ HIỆU NĂNG:
-- LIKE '%text%' → FULL TABLE SCAN (chậm!)
--   Không thể dùng index cho wildcard ở đầu
-- LIKE 'text%'  → Prefix lookup (NHANH!)
--   B-Tree index hỗ trợ prefix lookup
```

### 2.4 LIMIT & OFFSET — Phân trang

```sql
-- Lấy 2 records đầu tiên
SELECT * FROM users ORDER BY email LIMIT 2;

-- Bỏ qua 1, lấy 2 tiếp theo
SELECT * FROM users ORDER BY email LIMIT 2 OFFSET 1;

-- ⚠️ OFFSET bắt đầu từ 0 (giống Python list)
-- Phân trang hiệu quả:
--   Trang 1: LIMIT 25 OFFSET 0
--   Trang 2: LIMIT 25 OFFSET 25
--   Trang N: LIMIT 25 OFFSET (N-1)*25

-- Database xử lý phân trang HIỆU QUẢ HƠN
-- SO VỚI select tất cả rồi code bỏ bớt
```

### 2.5 COUNT — Đếm hiệu quả

```sql
-- Đếm tất cả rows
SELECT COUNT(*) FROM users;

-- Đếm với điều kiện
SELECT COUNT(*) FROM users WHERE email = 'csev@umich.edu';

-- COUNT hiệu quả hơn:
-- SELECT * FROM users → code đếm (kéo TẤT CẢ data qua network)
-- SELECT COUNT(*) FROM users → chỉ trả về 1 số
-- Database có thể biết COUNT mà không đọc actual data
```

---

## 3. Data Types trong PostgreSQL

### 3.1 Text Types

```
┌─────────────────────────────────────────────────────────┐
│ Text Types trong PostgreSQL                             │
├────────────┬────────────────────────────────────────────┤
│ VARCHAR(n) │ Biến đổi, tối đa n ký tự ⭐ hay dùng nhất│
│            │ - Lưu trữ: count + actual data            │
│            │ - Hiệu quả khi length thay đổi nhiều      │
│            │ - Ví dụ: name VARCHAR(128)                 │
├────────────┼────────────────────────────────────────────┤
│ CHAR(n)    │ Cố định đúng n ký tự                      │
│            │ - Thêm spaces nếu ngắn hơn n              │
│            │ - Tốt cho: GUID (64 ký tự), hash codes    │
│            │ - Database "yêu" CHAR vì predict được size │
│            │ - Dùng khi CHẮC CHẮN length cố định       │
├────────────┼────────────────────────────────────────────┤
│ TEXT       │ Không giới hạn chiều dài                   │
│            │ - Tốt cho: blog posts, comments, web pages │
│            │ - ⚠️ KHÔNG nên dùng cho ORDER BY, WHERE   │
│            │ - CÓ THỂ dùng cho LIKE (full table scan)  │
│            │ - PostgreSQL chỉ có 1 loại TEXT (đơn giản!)│
├────────────┼────────────────────────────────────────────┤
│ BYTEA      │ Binary data (không có character set)       │
│            │ - Lưu: images, blob data                   │
│            │ - KHÔNG sort, KHÔNG index trên BYTEA       │
└────────────┴────────────────────────────────────────────┘
```

### 3.2 Character Sets — Tại sao quan trọng?

```
VARCHAR(100) KHÔNG có nghĩa 100 bytes!

Character Set:
- ASCII/Latin: 1 byte per ký tự (127 ký tự)
- UTF-8: 1-4 bytes per ký tự
  - Tiếng Việt: 2-3 bytes mỗi ký tự
  - Tiếng Trung/Nhật: 3 bytes mỗi ký tự
  
→ VARCHAR(100) có thể chiếm đến 400 bytes!
→ Database xử lý SORTING khác nhau cho character sets khác nhau
→ CHAR, VARCHAR, TEXT: CÓ character set
→ BYTEA: KHÔNG CÓ character set (raw binary)
```

### 3.3 Numeric Types

```sql
-- Integer Types
SMALLINT         -- 2 bytes: ±32,768
INTEGER          -- 4 bytes: ±2.1 tỷ ⭐ phổ biến nhất
BIGINT           -- 8 bytes: rất lớn

-- Floating Point (XẤP XỈ — sai số!)
REAL             -- 32-bit, 7 chữ số chính xác
                 -- Tốt cho: nhiệt độ trung bình, ước lượng
DOUBLE PRECISION -- 64-bit, 15 chữ số chính xác
                 -- Tốt cho: tính toán khoa học, simulation

-- ⚠️ CẢ HAI ĐỀU KHÔNG TỐT CHO TIỀN!
-- Lý do: phân số nhị phân không biểu diễn chính xác 1/100
-- Ví dụ: $0.01 KHÔNG chính xác trong REAL/DOUBLE

-- Exact Numeric (CHÍNH XÁC — dùng cho tiền!)
NUMERIC(14, 2)   -- 14 digits tổng, 2 digits sau dấu phẩy
                 -- ⭐ LUÔN dùng cho tiền!
```

> 🎬 **Trivia**: Có nhiều bộ phim kể về việc hack lỗi floating-point trong hệ thống ngân hàng — gom các phần lẻ cent từ hàng triệu giao dịch!

### 3.4 Date/Time Types

```sql
-- Date Types
DATE             -- Chỉ ngày
TIME             -- Chỉ giờ  
TIMESTAMP        -- Ngày + giờ (64-bit) ⭐

-- ⚠️ Vấn đề Y2038:
-- Hệ thống cũ dùng 32-bit Unix time (giây từ 1/1/1970)
-- 32-bit chỉ đếm được đến năm 2038!
-- PostgreSQL đã chuyển sang 64-bit → an toàn đến ~300,000 AD

-- Function thường dùng:
SELECT NOW();           -- Thời gian hiện tại
SELECT CURRENT_DATE;    -- Ngày hiện tại
```

---

## 4. Keys — Chìa khóa của Database

### 4.1 Ba loại Key

```
┌─────────────────────────────────────────────────────────────┐
│                    3 Loại Keys                              │
├──────────────┬──────────────────────────────────────────────┤
│ PRIMARY KEY  │ "Handle" cho mỗi row                        │
│ (id)         │ - Auto-increment integer (SERIAL)           │
│              │ - Tự động gán: 1, 2, 3, 4...               │
│              │ - KHÔNG BAO GIỜ thay đổi                    │
│              │ - Nội bộ database, user không cần biết       │
│              │ - Có INDEX siêu nhanh (Hash)                │
├──────────────┼──────────────────────────────────────────────┤
│ LOGICAL KEY  │ "Tên gọi" để thế giới bên ngoài tìm row    │
│ (email,      │ - Email, username, title, slug...           │
│  title...)   │ - UNIQUE constraint                         │
│              │ - CÓ THỂ thay đổi (hiếm)                   │
│              │ - Có INDEX B-Tree (sort, prefix search)     │
├──────────────┼──────────────────────────────────────────────┤
│ FOREIGN KEY  │ Integer trong bảng NÀY trỏ đến row bảng    │
│ (artist_id)  │ bảng KHÁC                                  │
│              │ - Tên convention: [table]_id                │
│              │ - Integer → matching siêu nhanh             │
│              │ - Là "đầu mũi tên" trong data model        │
└──────────────┴──────────────────────────────────────────────┘
```

### 4.2 Tại sao không dùng Email làm Primary Key?

```
Lý do dùng INTEGER thay vì STRING làm PRIMARY KEY:

1. PERFORMANCE: 
   - Integer comparison: CPU instruction đơn 
   - String comparison: loop qua từng ký tự

2. STORAGE:
   - Integer: 4 bytes (cố định)
   - Email: 5-100 bytes (biến đổi)
   - Foreign key replicate KHẮP NƠI → tiết kiệm rất nhiều

3. STABILITY:
   - Email CÓ THỂ thay đổi (kết hôn, đổi tên, đổi công ty)
   - Primary key: thay đổi 1 chỗ
   - Logical key: search lại bằng tên mới

4. UNIVERSALITY:
   - Mọi database tối ưu integer index
   - GUID (string dài) kém hiệu quả hơn integer
   
→ Kết luận: LUÔN dùng SERIAL integer cho PRIMARY KEY!
```

> 💡 **Dr. Chuck**: "Có database experts nói dùng email làm primary key cũng được. Họ SAI. Bên dưới database vẫn phải map string thành number. Tốt hơn là bạn tự quản lý number đó."

---

## 5. Indexes — Tốc độ của Database

### 5.1 Tại sao cần Index?

```
Bài toán: Tìm 1 user trong 500 triệu users (Twitter)

Không có Index:
  → Đọc TẤT CẢ data từ đầu đến cuối
  → Terabytes data → PHÚT để tìm
  → = Full Table Scan = Sequential scan

Có Index:
  → Dùng "lối tắt" nhảy thẳng đến đúng chỗ
  → Nhiều TB data → MILLISECONDS để tìm

Index = Dữ liệu THÊM trên disk
  → Nói "build index" = bảo database lưu thêm metadata
  → Trade-off: dùng thêm disk space, INSERT chậm hơn một chút
  → Nhưng SELECT nhanh hơn RẤT NHIỀU
```

### 5.2 B-Tree Index

```
B-Tree (Balanced Tree / Binary Tree):

         ┌──────────────────────┐
         │   Index Block 1      │
         │  A-M → Block 2       │
         │  N-Z → Block 3       │
         └──────────────────────┘
              ↙            ↘
┌──────────────────┐  ┌──────────────────┐
│  Index Block 2   │  │  Index Block 3   │
│  A-F → Data 1    │  │  N-R → Data 4    │
│  G-M → Data 2    │  │  S-Z → Data 5    │
└──────────────────┘  └──────────────────┘
     ↓        ↓           ↓        ↓
  Data 1   Data 2      Data 4   Data 5

Hiệu quả:
  1 triệu records → ~6 lần đọc disk (log₂ 1M ≈ 20, nhưng fan-out lớn)
  10 triệu records → ~7 lần đọc disk

Tốt cho:
  ✅ Exact match (WHERE email = 'x')
  ✅ Sorting (ORDER BY)  
  ✅ Range lookup (WHERE salary BETWEEN 50K AND 100K)
  ✅ Prefix lookup (WHERE name LIKE 'Chuck%')
  
Không tốt cho:
  ❌ Suffix/infix (WHERE name LIKE '%uck')
```

### 5.3 Hash Index

```
Hash Index:

Input: "csev@umich.edu"
  ↓ Hash Function (MD5, SHA256...)
  ↓ Tính toán → số (ví dụ: 847291)
  ↓ 847291 mod 1000000 = 847291
  ↓ Nhảy thẳng đến vị trí 847291
  ↓ → Tìm được record!

Hiệu quả:
  1 triệu records → 1-2 lần đọc disk!
  CÒN NHANH HƠN B-Tree cho exact match!

Tốt cho:
  ✅ Exact match ONLY (WHERE id = 42)
  ✅ Primary key lookups
  ✅ GUID lookups

Không tốt cho:
  ❌ Sorting (data không có thứ tự)
  ❌ Range queries
  ❌ Prefix matching
```

### 5.4 Khi nào Database chọn loại Index nào?

```sql
-- PRIMARY KEY → thường dùng Hash (exact match cực nhanh)
id SERIAL PRIMARY KEY

-- UNIQUE trên string → thường dùng B-Tree (sort + prefix)
email VARCHAR(128) UNIQUE

-- Bạn KHÔNG cần tự chọn! Database tự quyết!
-- "Hey database, you're smarter than I am.
--  You got a lot of PhDs that work on this."
-- — Dr. Chuck
```

---

## 6. Import dữ liệu CSV

### 6.1 Tải file CSV

```bash
# Trên PythonAnywhere hoặc Linux:
wget https://www.pg4e.com/tools/sql/library.csv

# Hoặc:
curl -o library.csv https://www.pg4e.com/tools/sql/library.csv

# Xem nội dung:
cat library.csv
# → title,artist,album,count,rating,len
# → Another One Bites the Dust,Queen,Greatest Hits,55,100,217
```

### 6.2 Tạo bảng và Import

```sql
-- Tạo bảng với cấu trúc khớp CSV:
CREATE TABLE track_raw (
    title TEXT,
    artist TEXT,
    album TEXT,
    count INTEGER,
    rating INTEGER,
    len INTEGER
);

-- Import bằng lệnh \copy (psql command, KHÔNG phải SQL):
\copy track_raw(title, artist, album, count, rating, len) 
FROM 'library.csv' WITH DELIMITER ',' CSV;

-- Kiểm tra:
SELECT COUNT(*) FROM track_raw;
-- → 296

-- Xem một số rows:
SELECT * FROM track_raw ORDER BY title LIMIT 5;
```

> ⚠️ `\copy` là lệnh **psql** (bắt đầu bằng `\`), KHÔNG phải SQL. Nếu dùng DBeaver, cần dùng chức năng Import của DBeaver thay thế.

---

## 7. Tổng kết: SQL là ngôn ngữ ĐẸP

```
Tại sao Dr. Chuck yêu SQL:

1. ĐƠN GIẢN: Chỉ có CRUD + WHERE + ORDER BY + LIMIT
   → Dễ hơn NHIỀU so với viết chương trình thông thường

2. DECLARATIVE: Nói CẦN GÌ, không nói LÀM THẾ NÀO
   → Database tự tìm cách tối ưu

3. ABSTRACTION: Giấu toàn bộ complexity
   → Indexes, B-Trees, Hash, disk I/O → database lo
   → Developer chỉ cần viết SQL đúng

4. HIỆU NĂNG: 50 terabytes, login < 0.25 giây
   → Không có ngôn ngữ nào khác làm được

"This is the core of SQL. It's beautiful. It's simple.
 It's easier than learning how to write programs."
 — Dr. Chuck
```

---

## ❓ Câu Hỏi Kiểm Tra

1. DELETE, UPDATE, SELECT đều có "implied loop" — giải thích.
2. VARCHAR(128) vs CHAR(64) vs TEXT — khi nào dùng cái nào?
3. Tại sao REAL/DOUBLE PRECISION không dùng cho tiền? Dùng gì thay thế?
4. So sánh B-Tree vs Hash Index: mỗi loại tốt cho gì?
5. Primary Key vs Logical Key vs Foreign Key — cho ví dụ mỗi loại.
6. Tại sao `LIKE '%abc%'` gây full table scan? `LIKE 'abc%'` thì sao?
7. `LIMIT 10 OFFSET 20` lấy rows nào?
8. SERIAL trong PostgreSQL tương đương lệnh gì ở các database khác?
9. `\copy` khác gì lệnh SQL `COPY`?
10. Tại sao nên dùng integer auto-increment làm PRIMARY KEY thay vì email?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What are the different types of indexes? When would you use each?**  
> A: **B-Tree**: phổ biến nhất, tốt cho exact match, range queries, sorting, và prefix lookups. Postgres mặc định dùng B-Tree. **Hash**: chỉ tốt cho exact match, nhưng nhanh hơn B-Tree cho exact match. Thường dùng cho primary key lookups. Thực tế, database tự chọn loại index tối ưu.

> **Q: What is the difference between VARCHAR, CHAR, and TEXT?**  
> A: **VARCHAR(n)**: variable-length string tối đa n ký tự, lưu trữ hiệu quả. **CHAR(n)**: fixed-length, thêm padding spaces, tốt cho data luôn cùng length (GUIDs, hash codes). **TEXT**: unlimited length, tốt cho big content (blog posts), không nên dùng cho indexing/sorting.

> **Q: Why should you use integer auto-increment for primary keys?**  
> A: Integer comparison là single CPU instruction (cực nhanh), chiếm 4 bytes cố định (tiết kiệm storage), ổn định (không bao giờ thay đổi), và mọi database đều tối ưu integer indexes. String keys (email) thay đổi được, chiếm nhiều space hơn, và comparison chậm hơn.

> **Q: What is a full table scan and when does it happen?**  
> A: Full table scan xảy ra khi database phải đọc TẤT CẢ rows trong bảng. Xảy ra khi: không có index trên cột trong WHERE clause, dùng LIKE với wildcard ở đầu ('%abc'), hoặc query không thể tận dụng index nào. Rất chậm với bảng lớn.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Tạo bảng `movies (title VARCHAR(200), director VARCHAR(100), year INTEGER, rating NUMERIC(3,1), genre VARCHAR(50))`. Insert 10 phim. Viết query: (a) top 5 rating cao nhất, (b) phim năm 2020+, (c) đếm theo genre, (d) tìm phim có "The" trong tên.

**BT2**: Download `library.csv` vào PythonAnywhere. Import bằng `\copy`. Viết queries: (a) tổng số tracks, (b) tracks có rating > 80, (c) top 10 tracks nghe nhiều nhất, (d) tracks của artist "Queen".

**BT3**: Tạo bảng có `id SERIAL PRIMARY KEY` và `email VARCHAR(128) UNIQUE`. Thử insert email trùng → xem lỗi gì xảy ra. Giải thích tại sao UNIQUE constraint hữu ích.

---

## ✅ Checklist Hoàn Thành Module 2

1. Giải thích được "implied loop" trong SQL.
2. Chọn đúng data type cho từng loại dữ liệu.
3. Phân biệt được 3 loại key và khi nào dùng.
4. Hiểu B-Tree vs Hash Index ở level concept.
5. Import thành công CSV vào PostgreSQL bằng `\copy`.
6. Viết thành thạo SELECT với WHERE, ORDER BY, LIKE, LIMIT, COUNT.
7. Hoàn thành tất cả assignments trên Coursera.

---

## 📎 Tham Khảo

- Đáp Án: [99-answer-key-db.md#drchuck-m2](./99-answer-key-db.md#drchuck-m2)
- Lý thuyết sâu: [97-db-theory-deep-dive.md#drchuck-m2-deep](./97-db-theory-deep-dive.md#drchuck-m2-deep)
