# 📗 Module 3: One-To-Many Data Models (Dr. Chuck — PostgreSQL)

> **Nguồn**: Database Design and Basic SQL in PostgreSQL — Week 3  
> **Giảng viên**: Charles Severance (Dr. Chuck) — University of Michigan  
> **Thời lượng ước tính**: 4 giờ

---

## 🎯 Mục Tiêu

- Thiết kế data model từ mock-up UI (chuyển từ spreadsheet → tables)
- Hiểu và loại bỏ vertical replication
- Phân biệt Primary Key, Logical Key, Foreign Key và áp dụng vào thiết kế
- Viết CREATE TABLE với SERIAL, PRIMARY KEY, UNIQUE, REFERENCES, ON DELETE CASCADE
- Insert dữ liệu normalized bằng tay (hiểu trước khi tự động hóa)
- Sử dụng JOIN để ghép dữ liệu từ nhiều bảng
- Hiểu INNER JOIN, CROSS JOIN, và ON clause

---

## 1. Database Design — Nghệ thuật mô hình hóa dữ liệu

### 1.1 Từ UI đến Data Model

```
Quy trình thiết kế database:

┌────────────────────┐
│   UI Mockup        │  ← Designer tạo: có vertical replication
│   (Spreadsheet)    │     (cùng artist/album lặp nhiều lần)
└────────────────────┘
         ↓
┌────────────────────┐
│   Logical Model    │  ← Dev tạo: vẽ hình các bảng + arrows
│   (Diagram)        │     trên whiteboard
└────────────────────┘
         ↓
┌────────────────────┐
│   Physical Model   │  ← Dev tạo: CREATE TABLE + keys + indexes
│   (SQL Schema)     │
└────────────────────┘
         ↓
┌────────────────────┐
│   Reconstruct UI   │  ← JOIN dùng numbers để ghép strings
│   (SELECT + JOIN)  │     trả lại cho user interface
└────────────────────┘
```

### 1.2 Vấn đề: Vertical Replication

```
Spreadsheet có vấn đề:

Track              | Artist      | Album        | Genre
─────────────────────────────────────────────────────────
Black Dog          | Led Zep     | IV           | Rock
Stairway           | Led Zep     | IV           | Rock
About to Rock      | AC/DC      | Who Made Who | Rock
Who Made Who       | AC/DC      | Who Made Who | Rock
 
↑ "Led Zep" lặp 2 lần, "AC/DC" lặp 2 lần
↑ "IV" lặp 2 lần, "Who Made Who" lặp 2 lần
↑ "Rock" lặp 4 lần!

Vấn đề:

1. Typo "Led Zeppelin" → phải sửa KHẮP NƠI
2. 1 triệu records → strings chiếm KHỔNG LỒ disk space
3. Sửa 1 chỗ quên sửa chỗ khác → data KHÔNG NHẤT QUÁN

Giải pháp: COMPRESSION THROUGH CONNECTION
→ Lưu "Led Zeppelin" MỘT LẦN DUY NHẤT
→ Gán số: Led Zeppelin = 1
→ Dùng số 1 THAY CHO string ở mọi nơi
→ Sửa typo → sửa 1 chỗ duy nhất!
```

> 💡 **Dr. Chuck**: "This is compression through connection. The single most important concept in building a data model is: vertical replication of string data is NOT what we want to do."

### 1.3 Quy trình thiết kế — Bước từng bước

```
Bước 1: XÁC ĐỊNH "core entity" (thứ chính mà app tổ chức)
  → Music app: TRACK là core entity
  → Social network: USER là core entity
  → E-commerce: PRODUCT là core entity

Bước 2: ĐẶT core entity ở GIỮA

Bước 3: Với mỗi cột, hỏi:
  "Đây là ATTRIBUTE của entity nào?"
  
  - Numbers (rating, count, length) → attribute trực tiếp (rẻ, không lặp)
  - Strings bị lặp (artist, album) → TẠO TABLE MỚI!

Bước 4: VẼ ARROWS giữa các tables
  → Nhiều track → 1 album (many-to-one)
  → Nhiều album → 1 artist (many-to-one)
  → Nhiều track → 1 genre (many-to-one)
```

### 1.4 Ví dụ: Music Database Data Model

```
Từ columns: Track, Artist, Album, Genre, Rating, Count, Length

     ┌──────────────┐
     │   ARTIST     │ ← 1 artist có NHIỀU albums
     │ • id (PK)    │
     │ * name (LK)  │
     └──────────────┘
           ↑
     ┌──────────────┐
     │   ALBUM      │ ← 1 album có NHIỀU tracks
     │ • id (PK)    │
     │ * title (LK) │
     │ ∘ artist_id  │─→ FK trỏ đến artist.id
     └──────────────┘
           ↑
     ┌──────────────┐        ┌──────────────┐
     │   TRACK      │        │   GENRE      │
     │ • id (PK)    │        │ • id (PK)    │
     │ * title (LK) │        │ * name (LK)  │
     │   rating     │        └──────────────┘
     │   len        │              ↑
     │   count      │              │
     │ ∘ album_id   │─→ FK         │
     │ ∘ genre_id   │──────────────┘ FK
     └──────────────┘

Legend:
  • = Primary Key (SERIAL, auto-increment)

  * = Logical Key (UNIQUE)
  ∘ = Foreign Key (INTEGER REFERENCES)
```

### 1.5 Genre thuộc về đâu?

```
Genre có thể gắn vào:

1. TRACK  → đổi genre 1 track KHÔNG ảnh hưởng track khác ✅
2. ALBUM  → đổi genre album → TẤT CẢ tracks trong album đổi
3. ARTIST → đổi genre artist → TẤT CẢ tracks của artist đổi

→ Thường chọn TRACK vì linh hoạt nhất!
→ Nhưng tùy vào business requirement
→ Đây là lúc THẢO LUẬN với team trong phòng whiteboard
```

---

## 2. Keys trong thiết kế Multi-Table

### 2.1 Naming Convention (Quy ước đặt tên)

```sql
-- Convention mà Dr. Chuck sử dụng:

-- Primary Key: luôn tên "id"
id SERIAL PRIMARY KEY

-- Logical Key: tên gọi thực tế, thêm UNIQUE
name VARCHAR(128) UNIQUE
title VARCHAR(128) UNIQUE
email VARCHAR(128) UNIQUE

-- Foreign Key: [tên_bảng]_id
artist_id INTEGER REFERENCES artist(id)
album_id INTEGER REFERENCES album(id)
genre_id INTEGER REFERENCES genre(id)

-- ⚠️ MỖI tổ chức có convention KHÁC NHAU
-- → Khi vào công ty mới, KHÔNG nói "thầy tôi dạy khác"
-- → HỌC convention của họ và FOLLOW
-- → Consistency > "đúng sai"
```

### 2.2 UNIQUE có thể là combination

```sql
-- Một cột UNIQUE:
name VARCHAR(128) UNIQUE  -- Chỉ 1 "Led Zeppelin" 

-- Combination UNIQUE (đặc biệt):
-- Có thể có nhiều track tên "Moonlight" (albums khác nhau!)
-- Nhưng 1 album chỉ có 1 "Moonlight"
CREATE TABLE track (
    ...
    UNIQUE(title, album_id)  -- Combination unique
);

-- Nếu bạn put UNIQUE chỉ trên title:
-- → Toàn bộ hệ thống chỉ có 1 "Moonlight" → quá restricted!
```

---

## 3. Tạo Tables — SQL thật

### 3.1 Tạo từ leaves → center

```sql
-- ⚠️ THỨ TỰ TẠO QUAN TRỌNG!
-- Tạo tables BÊN NGOÀI trước (tables mà FK trỏ đến)
-- Vì FK REFERENCES cần table đích đã tồn tại

-- 1. Tạo Artist (leaf — không FK nào)
CREATE TABLE artist (
    id SERIAL,
    name VARCHAR(128) UNIQUE,
    PRIMARY KEY(id)
);

-- 2. Tạo Album (FK trỏ đến artist)
CREATE TABLE album (
    id SERIAL,
    title VARCHAR(128) UNIQUE,
    artist_id INTEGER REFERENCES artist(id) ON DELETE CASCADE,
    PRIMARY KEY(id)
);

-- 3. Tạo Genre (leaf — không FK nào)
CREATE TABLE genre (
    id SERIAL,
    name VARCHAR(128) UNIQUE,
    PRIMARY KEY(id)
);

-- 4. Tạo Track (FK trỏ đến album + genre)
CREATE TABLE track (
    id SERIAL,
    title VARCHAR(128),
    len INTEGER,
    rating INTEGER,
    count INTEGER,
    album_id INTEGER REFERENCES album(id) ON DELETE CASCADE,
    genre_id INTEGER REFERENCES genre(id) ON DELETE CASCADE,
    UNIQUE(title, album_id),
    PRIMARY KEY(id)
);
```

### 3.2 SERIAL — Magic của PostgreSQL

```sql
-- SERIAL tương đương viết tay:
-- id INTEGER NOT NULL DEFAULT nextval('artist_id_seq')
-- + tự tạo sequence 'artist_id_seq'

-- Trong MySQL: AUTO_INCREMENT
-- Trong SQLite: AUTOINCREMENT  
-- Trong PostgreSQL: SERIAL ⭐ (gọn nhất!)

-- Bạn KHÔNG cần insert id:
INSERT INTO artist (name) VALUES ('Led Zeppelin');
-- → id tự động = 1

INSERT INTO artist (name) VALUES ('AC/DC');
-- → id tự động = 2
```

### 3.3 ON DELETE CASCADE — Dọn dẹp tự động

```
ON DELETE CASCADE: Khi xóa parent → tự xóa children

Ví dụ: DELETE FROM genre WHERE name = 'Metal';

TRƯỚC:
  genre: {1: Rock, 2: Metal}
  track: {1: ..genre_id=1, 2: ..genre_id=2, 
          3: ..genre_id=2, 4: ..genre_id=1}

SAU DELETE Metal:
  genre: {1: Rock}
  track: {1: ..genre_id=1, 4: ..genre_id=1}
  → Track 2 và 3 (genre_id=2) tự động BỊ XÓA!

Alternatives:
┌─────────────────┬────────────────────────────────────┐
│ ON DELETE...     │ Hành vi                            │
├─────────────────┼────────────────────────────────────┤
│ CASCADE         │ Xóa parent → xóa luôn children    │
│                 │ ⭐ Dr. Chuck recommend             │
├─────────────────┼────────────────────────────────────┤
│ RESTRICT        │ Xóa parent → ERROR nếu có children│
│                 │ Bảo vệ khỏi xóa nhầm             │
├─────────────────┼────────────────────────────────────┤
│ SET NULL        │ Xóa parent → children FK = NULL    │
│                 │ Cần: column INTEGER NULL (cho phép)│
└─────────────────┴────────────────────────────────────┘
```

> 💡 **Dr. Chuck**: "Khi SQL báo lỗi, đừng nghĩ nó mean. Bạn đã BẢO nó enforce rules on you. Nó cứu bạn khỏi mistakes!"

---

## 4. Insert dữ liệu Normalized

### 4.1 Insert từ ngoài vào trong

```sql
-- BƯỚC 1: Insert vào tables NGOÀI (không có FK):
INSERT INTO artist (name) VALUES ('Led Zeppelin');
INSERT INTO artist (name) VALUES ('AC/DC');

SELECT * FROM artist;
--  id |    name
-- ----+--------------
--   1 | Led Zeppelin
--   2 | AC/DC

-- BƯỚC 2: Insert vào tables CÓ FK (nhớ số id!):
INSERT INTO album (title, artist_id) VALUES ('IV', 1);
-- IV thuộc Led Zeppelin (id=1)
INSERT INTO album (title, artist_id) VALUES ('Who Made Who', 2);
-- Who Made Who thuộc AC/DC (id=2)

-- BƯỚC 3: Insert genre:
INSERT INTO genre (name) VALUES ('Rock');
INSERT INTO genre (name) VALUES ('Metal');

-- BƯỚC 4: Insert track (cần nhớ album_id VÀ genre_id):
INSERT INTO track (title, rating, len, count, album_id, genre_id)
VALUES ('Black Dog', 5, 297, 0, 2, 1);
-- album_id=2 (Who Made Who? KHÔNG! Cần check lại!)
-- → Đây là lúc dễ SAI nếu làm bằng tay

INSERT INTO track (title, rating, len, count, album_id, genre_id)
VALUES ('Stairway', 5, 482, 0, 1, 1);
-- album_id=1 (IV), genre_id=1 (Rock)
```

> ⚠️ **Ghi chú bến giấy!** Khi insert bằng tay, bạn cần GHI NHỚ atau ghi ra giấy id của mỗi row. Sau này sẽ học cách tự động hóa bằng sub-SELECTs.

### 4.2 UNIQUE constraint bảo vệ bạn

```sql
-- Cố insert genre 'Rock' lần 2:
INSERT INTO genre (name) VALUES ('Rock');
-- ERROR: duplicate key violates unique constraint

-- Đây KHÔNG phải bug!
-- Database ĐANG BẢO VỆ bạn khỏi nhập trùng!
-- UNIQUE enforcement = bạn yêu cầu database check cho bạn
```

---

## 5. JOIN — Ghép dữ liệu từ nhiều bảng

### 5.1 Tại sao cần JOIN?

```
Dữ liệu trong database: toàn NUMBERS (compressed)

Track: {title: "Black Dog", album_id: 1, genre_id: 1}
Album: {id: 1, title: "IV", artist_id: 1}
Artist: {id: 1, name: "Led Zeppelin"}
Genre: {id: 1, name: "Rock"}

User muốn thấy: 
  "Black Dog | Led Zeppelin | IV | Rock"

→ JOIN = RECONSTRUCT strings từ numbers
→ Decompression at the last moment!
```

### 5.2 INNER JOIN — Ghép matching rows

```sql
-- JOIN 2 bảng: Album + Artist
SELECT album.title, artist.name
FROM album
JOIN artist ON album.artist_id = artist.id;

-- Output:
--    title     |    name
-- -------------+--------------
--  IV          | Led Zeppelin
--  Who Made Who| AC/DC

-- Giải thích:
-- FROM album          → bắt đầu từ album
-- JOIN artist         → nối thêm artist  
-- ON album.artist_id  → field FK trong album
--  = artist.id        → field PK trong artist
-- → Chỉ lấy rows mà FK MATCH PK
```

### 5.3 CROSS JOIN — Hiểu cơ chế JOIN

```sql
-- CROSS JOIN = TẤT CẢ combinations (không có ON)
SELECT track.title, genre.name
FROM track CROSS JOIN genre;

-- Nếu track có 4 rows, genre có 2 rows:
-- → Output: 4 × 2 = 8 rows (MỌI tổ hợp!)

-- CROSS JOIN cho thấy:
-- INNER JOIN = CROSS JOIN + FILTER (ON clause)
-- ON clause = "WHERE khác" lọc bỏ rows không match

-- ⚠️ CROSS JOIN rất EXPENSIVE!
-- 1 triệu × 1 triệu = 1 TRILLION rows
-- ĐỪNG dùng trong production!
```

### 5.4 Multi-Table JOIN — Ghép 4 bảng

```sql
-- JOIN across 4 tables:
SELECT track.title, artist.name, album.title, genre.name
FROM track
    JOIN genre  ON track.genre_id  = genre.id
    JOIN album  ON track.album_id  = album.id
    JOIN artist ON album.artist_id = artist.id;

-- Output:
-- title        | name         | title        | name
-- -------------+--------------+--------------+------
-- Black Dog    | Led Zeppelin | IV           | Rock
-- Stairway     | Led Zeppelin | IV           | Rock
-- About to Rock| AC/DC        | Who Made Who | Rock
-- Who Made Who | AC/DC        | Who Made Who | Rock

-- Pattern nhận ra:
-- 1. Mỗi JOIN cần 1 ON clause
-- 2. ON clause luôn là FK = PK
-- 3. Convention giúp đọc nhanh:
--    track.genre_id = genre.id
--    [table].[target_table]_id = [target_table].id
```

> 💡 **Dr. Chuck**: "JOIN RECONSTRUCTS vertical replication. Database lưu compressed (numbers), JOIN decompresses (strings) at the last moment. After that, it's still efficient in the database."

### 5.5 Thêm cột debug để xem ON clause hoạt động

```sql
-- Xem thêm FK và PK để hiểu cơ chế:
SELECT album.title, album.artist_id, artist.id, artist.name
FROM album
JOIN artist ON album.artist_id = artist.id;

-- Output:
--    title     | artist_id | id |    name
-- -------------+-----------+----+--------------
--  IV          |     1     |  1 | Led Zeppelin   ← match!
--  Who Made Who|     2     |  2 | AC/DC          ← match!

-- Thấy rõ: artist_id (FK) = id (PK) → đó là ON clause!
```

---

## 6. ON DELETE CASCADE — Demo thực tế

```sql
-- Xem track ban đầu: 4 rows
SELECT * FROM track;
-- id | title        | ... | genre_id
-- 1  | Black Dog    | ... | 1 (Rock)
-- 2  | Stairway     | ... | 1 (Rock)
-- 3  | About to Rock| ... | 1 (Rock)
-- 4  | Who Made Who | ... | 2 (Metal)

-- Xóa genre "Metal":
DELETE FROM genre WHERE name = 'Metal';

-- Kiểm tra genre:
SELECT * FROM genre;
-- id | name
-- 1  | Rock
-- → Metal đã bị xóa

-- Kiểm tra track:
SELECT * FROM track;
-- id | title        | ... | genre_id
-- 1  | Black Dog    | ... | 1
-- 2  | Stairway     | ... | 1
-- 3  | About to Rock| ... | 1
-- → "Who Made Who" (genre_id=2) tự động bị xóa!
-- → ON DELETE CASCADE hoạt động!
```

---

## 7. Tổng kết One-To-Many Pattern

```
Pattern tổng quát:

1. VẼ HÌNH trước, code sau
   → Whiteboard, giấy, draw.io, dbdiagram.io

2. LOẠI BỎ vertical replication
   → String lặp > 2 lần → tách thành table riêng

3. THÊM keys:
   → Mỗi table: id SERIAL PRIMARY KEY
   → String chính: UNIQUE (logical key)  
   → Mũi tên: [table]_id INTEGER REFERENCES

4. TẠO TABLE từ ngoài vào trong
   → Tạo tables không có FK trước

5. INSERT DATA từ ngoài vào trong
   → Insert vào leaf tables trước, nhớ id

6. JOIN để reconstruct
   → JOIN + ON clause = ghép FK với PK

Pattern SQL (copy-paste nhanh):
┌──────────────────────────────────────────────┐
│ CREATE TABLE parent (                        │
│     id SERIAL,                               │
│     name VARCHAR(128) UNIQUE,                │
│     PRIMARY KEY(id)                          │
│ );                                           │
├──────────────────────────────────────────────┤
│ CREATE TABLE child (                         │
│     id SERIAL,                               │
│     title VARCHAR(128),                      │
│     parent_id INTEGER REFERENCES parent(id)  │
│         ON DELETE CASCADE,                   │
│     PRIMARY KEY(id)                          │
│ );                                           │
├──────────────────────────────────────────────┤
│ SELECT child.title, parent.name              │
│ FROM child                                   │
│ JOIN parent ON child.parent_id = parent.id;  │
└──────────────────────────────────────────────┘
```

---

## ❓ Câu Hỏi Kiểm Tra

1. "Vertical replication" là gì? Tại sao nó xấu?
2. Từ spreadsheet sau, tách thành bao nhiêu tables? `Student | Course | Professor | Grade`
3. Tại sao phải tạo tables theo thứ tự "từ ngoài vào trong"?
4. SERIAL trong PostgreSQL tương đương gì trong MySQL?
5. `ON DELETE CASCADE` vs `ON DELETE RESTRICT` — khác nhau thế nào?
6. Viết JOIN cho 3 bảng: student JOIN enrollment ON ... JOIN course ON ...
7. CROSS JOIN produces bao nhiêu rows nếu table A có 100 rows, table B có 50 rows?
8. `UNIQUE(title, album_id)` khác gì `title VARCHAR(128) UNIQUE`?
9. Tại sao Dr. Chuck nói "don't argue with the UI designer"?
10. "Compression through connection" nghĩa là gì?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is database normalization?**  
> A: Normalization là quá trình tổ chức dữ liệu để loại bỏ redundancy. Core idea: mỗi piece of string data chỉ lưu MỘT LẦN DUY NHẤT trong database, thay thế bằng integer references (foreign keys). Điều này giảm storage, tăng consistency (sửa 1 chỗ = sửa everywhere), và tăng performance.

> **Q: Explain primary key, foreign key, and logical key.**  
> A: **Primary key**: unique integer identifier cho mỗi row (auto-increment), dùng nội bộ database, không bao giờ thay đổi. **Foreign key**: integer trong table này trỏ đến primary key của table khác, tạo relationship. **Logical key**: field mà thế giới bên ngoài dùng để tìm row (email, username), có thể thay đổi.

> **Q: What is a JOIN and how does it work?**  
> A: JOIN combines rows từ 2+ tables dựa trên related columns. INNER JOIN chỉ trả về rows matching ON condition. Internally, conceptually giống CROSS JOIN (all combinations) + filter (ON clause chỉ giữ matches). Pattern: `FROM table_a JOIN table_b ON table_a.foreign_key = table_b.id`.

> **Q: What does ON DELETE CASCADE mean?**  
> A: Khi delete parent row, tự động delete tất cả child rows có foreign key trỏ đến parent đó. Giúp maintain referential integrity. Alternatives: RESTRICT (block delete if children exist), SET NULL (set FK to NULL instead of deleting children).

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Thiết kế data model cho hệ thống Library: `Book (title, author, publisher, genre, year, pages)`. Xác định vertical replication, tách tables, vẽ diagram, viết CREATE TABLE statements.

**BT2**: Tạo music database (artist, album, genre, track) theo bài giảng. Insert ít nhất 3 artists, 5 albums, 3 genres, 10 tracks. Viết JOIN ghép tất cả 4 bảng.

**BT3**: Thử xóa một genre có tracks liên quan. Quan sát ON DELETE CASCADE. Sau đó tạo lại tables với ON DELETE RESTRICT, thử xóa genre tương tự → so sánh kết quả.

**BT4**: Tạo hệ thống đơn giản: `Department (name)` và `Employee (name, salary, dept_id)`. Insert data, viết JOIN show employee.name, employee.salary, department.name.

---

## ✅ Checklist Hoàn Thành Module 3

1. Có thể nhìn spreadsheet và xác định NGAY cần bao nhiêu tables.
2. Vẽ được data model diagram (boxes + arrows).
3. Viết CREATE TABLE với SERIAL, PRIMARY KEY, FOREIGN KEY, UNIQUE.
4. Tạo tables đúng THỨ TỰ (leaves first).
5. INSERT normalized data bằng tay (biết track id numbers).
6. Viết INNER JOIN cho 2, 3, 4 bảng.
7. Giải thích được ON DELETE CASCADE vs RESTRICT vs SET NULL.
8. Hoàn thành tất cả assignments trên Coursera.

---

## 📎 Tham Khảo

- Đáp Án: [99-answer-key-db.md#drchuck-m3](./99-answer-key-db.md#drchuck-m3)
- Lý thuyết sâu: [97-db-theory-deep-dive.md#drchuck-m3-deep](./97-db-theory-deep-dive.md#drchuck-m3-deep)
