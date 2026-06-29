# 📗 Module 4: Many-To-Many Data Models (Dr. Chuck — PostgreSQL)

> **Nguồn**: Database Design and Basic SQL in PostgreSQL — Week 4  
> **Giảng viên**: Charles Severance (Dr. Chuck) — University of Michigan  
> **Thời lượng ước tính**: 3 giờ

---

## 🎯 Mục Tiêu

- Phân biệt one-to-many vs many-to-many relationships
- Nhận biết KHI NÀO cần many-to-many (dấu hiệu: "thêm 1 cột nữa")
- Thiết kế junction table (join table / through table)
- Sử dụng composite PRIMARY KEY
- Insert dữ liệu normalized cho many-to-many
- Viết JOIN qua 3 bảng (entity → junction → entity)
- Model data tại điểm kết nối (role, date, etc.)

---

## 1. From One-To-Many to Many-To-Many

### 1.1 Khi nào One-To-Many KHÔNG đủ?

```
Ví dụ bài trước: 1 Track thuộc 1 Album
  track.album_id → album.id  ✅ OK

Nhưng thực tế:

  - 1 track trên nhiều albums! (original + compilation + soundtrack)
  - 1 album có nhiều artists! (ft., collab, band members)

Dấu hiệu cần many-to-many:
┌────────────────────────────────────────────────────┐
│ Bạn muốn thêm album_id1, album_id2, album_id3... │
│                                                    │
│ → STOP! Đây là nhiều FK cùng loại                 │
│ → Bao nhiêu là đủ? 3? 10? 100?                   │
│ → Không bao giờ đủ! Luôn cần 1 nữa!              │
│ → Tất cả column thừa sẽ NULL → lãng phí!         │
│                                                    │
│ → Giải pháp: MANY-TO-MANY relationship            │
└────────────────────────────────────────────────────┘
```

> 💡 **Dr. Chuck**: "The moment you think 'I need to add another one', that's the moment you realize you need many-to-many."

### 1.2 So sánh One-To-Many vs Many-To-Many

```
ONE-TO-MANY (đã học):
  Nhiều tracks → 1 genre
  
  ┌───────┐    FK     ┌───────┐
  │ Track │ ────────→ │ Genre │
  │ (many)│           │ (one) │
  └───────┘           └───────┘
  
  Chỉ cần 1 FK column trong Track table

MANY-TO-MANY (module này):
  Nhiều students ↔ Nhiều courses
  1 student ở nhiều courses
  1 course có nhiều students

  ┌─────────┐    ┌────────────┐    ┌────────┐
  │ Student │←──→│  Member    │←──→│ Course │
  │         │    │ (junction) │    │        │
  └─────────┘    └────────────┘    └────────┘
  
  Cần 1 TABLE MỚI ở giữa (junction table)!
```

---

## 2. Junction Table — Bảng kết nối

### 2.1 Cấu trúc

```
Junction Table (nhiều tên gọi):
  = Join Table
  = Through Table  
  = Intermediate Table
  = Association Table
  = Bridge Table

Cấu trúc:
┌──────────────────────────────────────────────┐
│ Chia many-to-many thành 2 one-to-many:       │
│                                              │
│  Student ←one-to-many→ Member                │
│  Course  ←one-to-many→ Member                │
│                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│  │ Student  │   │  Member  │   │  Course  │ │
│  │──────────│   │──────────│   │──────────│ │
│  │ id (PK)  │←──│student_id│   │ id (PK)  │ │
│  │ name     │   │course_id │─→ │ title    │ │
│  │ email    │   │ role     │   │          │ │
│  └──────────┘   └──────────┘   └──────────┘ │
│                                              │
│  Member có 2 FK trỏ RA NGOÀI (outward)!     │
└──────────────────────────────────────────────┘
```

### 2.2 Model data tại connection

```
Role = data modeled AT THE CONNECTION

Giải thích:

- Jane KHÔNG phải teacher (globally)
- Jane KHÔNG phải student (globally)
- Jane là teacher TRONG Python course
- Jane là student TRONG SQL course

→ Role thuộc về MỐI QUAN HỆ student-course
→ Không thuộc student, không thuộc course
→ Thuộc junction table!

Tương tự trong thực tế:
┌─────────────────────────────────────────────────┐
│ Discussion Forum:                               │
│   - user_id (FK)                                │
│   - thread_id (FK)                              │
│   - comment_text  ← data at connection!         │
│   - created_at    ← data at connection!         │
│   - upvotes       ← data at connection!         │
│                                                 │
│ Online Course:                                  │
│   - student_id (FK)                             │
│   - course_id (FK)                              │
│   - role (teacher/student) ← at connection!     │
│   - enrollment_date        ← at connection!     │
│   - grade                  ← at connection!     │
└─────────────────────────────────────────────────┘
```

---

## 3. SQL Implementation

### 3.1 Tạo Tables (từ edges → middle)

```sql
-- ① Tạo Leaf Tables (không có FK):
CREATE TABLE student (
    id SERIAL,
    name VARCHAR(128),
    email VARCHAR(128) UNIQUE,
    PRIMARY KEY(id)
);

CREATE TABLE course (
    id SERIAL,
    title VARCHAR(128) UNIQUE,
    PRIMARY KEY(id)
);

-- ② Tạo Junction Table (có 2 FK):
CREATE TABLE member (
    student_id INTEGER REFERENCES student(id) ON DELETE CASCADE,
    course_id INTEGER REFERENCES course(id) ON DELETE CASCADE,
    role INTEGER,
    PRIMARY KEY(student_id, course_id)  -- ⭐ Composite PK!
);
```

### 3.2 Composite Primary Key — Điểm đặc biệt

```sql
-- KHÔNG dùng id SERIAL ở junction table!
-- Thay vào đó, PRIMARY KEY là COMBINATION:

PRIMARY KEY(student_id, course_id)

-- Nghĩa là:
-- ✅ student_id=1, course_id=1 → OK
-- ✅ student_id=1, course_id=2 → OK  
-- ✅ student_id=2, course_id=1 → OK
-- ❌ student_id=1, course_id=1 → ERROR (đã tồn tại!)

-- → 1 student chỉ ở 1 course TỐI ĐA 1 LẦN
-- → Không cần extra id column!

-- Khi nào CẦN id SERIAL ở junction?
-- → Forum comments: 1 user nhiều comments trên 1 thread
-- → PK: (user_id, thread_id, created_at) hoặc dùng id SERIAL
```

### 3.3 Insert Data

```sql
-- ① Insert vào leaf tables:
INSERT INTO student (name, email) 
VALUES ('Jane', 'jane@tsugi.org');
INSERT INTO student (name, email)
VALUES ('Ed', 'ed@tsugi.org');
INSERT INTO student (name, email)
VALUES ('Sue', 'sue@tsugi.org');

INSERT INTO course (title) VALUES ('Python');
INSERT INTO course (title) VALUES ('SQL');
INSERT INTO course (title) VALUES ('PHP');

-- Xem id đã tạo:
SELECT * FROM student;
--  id | name | email
--   1 | Jane | jane@tsugi.org
--   2 | Ed   | ed@tsugi.org
--   3 | Sue  | sue@tsugi.org

SELECT * FROM course;
--  id | title
--   1 | Python
--   2 | SQL
--   3 | PHP

-- ② Insert connections (nhớ id!):
-- Role: 1 = Teacher, 0 = Student
INSERT INTO member (student_id, course_id, role) VALUES (1, 1, 1);
-- Jane (1) → Python (1) → Teacher (1)

INSERT INTO member (student_id, course_id, role) VALUES (2, 1, 0);
-- Ed (2) → Python (1) → Student (0)

INSERT INTO member (student_id, course_id, role) VALUES (3, 1, 0);
-- Sue (3) → Python (1) → Student (0)

INSERT INTO member (student_id, course_id, role) VALUES (1, 2, 0);
-- Jane (1) → SQL (2) → Student (0)

INSERT INTO member (student_id, course_id, role) VALUES (2, 2, 1);
-- Ed (2) → SQL (2) → Teacher (1)

INSERT INTO member (student_id, course_id, role) VALUES (2, 3, 1);
-- Ed (2) → PHP (3) → Teacher (1)

INSERT INTO member (student_id, course_id, role) VALUES (3, 3, 0);
-- Sue (3) → PHP (3) → Student (0)
```

### 3.4 Xem dữ liệu junction table

```sql
SELECT * FROM member;
--  student_id | course_id | role
-- -----------+-----------+------
--      1     |     1     |   1    (Jane, Python, Teacher)
--      2     |     1     |   0    (Ed, Python, Student)
--      3     |     1     |   0    (Sue, Python, Student)
--      1     |     2     |   0    (Jane, SQL, Student)
--      2     |     2     |   1    (Ed, SQL, Teacher)
--      2     |     3     |   1    (Ed, PHP, Teacher)
--      3     |     3     |   0    (Sue, PHP, Student)

-- → Toàn numbers! Chưa readable → cần JOIN
```

---

## 4. JOIN qua Junction Table

### 4.1 Reconstruct readable data

```sql
-- JOIN Through junction table:
SELECT student.name, member.role, course.title
FROM student 
    JOIN member ON member.student_id = student.id
    JOIN course ON member.course_id = course.id
ORDER BY course.title, member.role DESC, student.name;

-- Output:
--  name | role |  title
-- ------+------+--------
--  Ed   |   1  | PHP      ← Teacher
--  Sue  |   0  | PHP      ← Student
--  Jane |   1  | Python   ← Teacher
--  Ed   |   0  | Python   ← Student
--  Sue  |   0  | Python   ← Student
--  Ed   |   1  | SQL      ← Teacher
--  Jane |   0  | SQL      ← Student
```

### 4.2 Giải thích pattern JOIN

```sql
-- Pattern cho many-to-many JOIN:
SELECT 
    left_table.column,
    junction_table.column,
    right_table.column
FROM left_table
    JOIN junction ON junction.left_id = left_table.id
    JOIN right_table ON junction.right_id = right_table.id;

-- Dr. Chuck thích đặt junction ở GIỮA:
-- FROM student → JOIN member → JOIN course
-- Đọc: "Đi từ student, QUA member, ĐẾN course"

-- ORDER BY:
-- course.title              → nhóm theo khóa học
-- member.role DESC          → teacher (1) trước student (0)
-- student.name              → ABC trong cùng nhóm
```

---

## 5. Tổng kết Pattern

### 5.1 One-To-Many vs Many-To-Many

```
┌──────────────────────────────────────────────────────┐
│              ONE-TO-MANY                             │
│                                                      │
│  Sử dụng khi: 1 parent có nhiều children            │
│  Ví dụ: 1 genre → nhiều tracks                      │
│  Cấu trúc: FK column trong child table              │
│  Tables: 2 (parent + child)                         │
│                                                      │
│  ┌───────┐  FK    ┌───────┐                         │
│  │ Child │ ─────→ │Parent │                         │
│  └───────┘        └───────┘                         │
├──────────────────────────────────────────────────────┤
│              MANY-TO-MANY                            │
│                                                      │
│  Sử dụng khi: cả 2 bên đều "nhiều"                 │
│  Ví dụ: students ↔ courses                          │
│  Cấu trúc: Junction table với 2 FK                  │
│  Tables: 3 (left + junction + right)                │
│                                                      │
│  ┌──────┐   ┌──────────┐   ┌───────┐               │
│  │ Left │←──│ Junction │──→│ Right │               │
│  └──────┘   └──────────┘   └───────┘               │
│                                                      │
│  Junction có thể model DATA tại connection          │
│  (role, date, grade, comment_text...)               │
└──────────────────────────────────────────────────────┘
```

### 5.2 Bài học quan trọng nhất

```
Tại sao học tất cả những này?

1. Spreadsheet = KHÔNG scalable
   - 100 rows: OK
   - 1 million rows: CHẬM
   - 100 million rows: KHÔNG HOẠT ĐỘNG

2. Database = scalable
   - Integer keys + indexes → tìm trong milliseconds
   - Dù 100 hay 100 million rows

3. Design TRƯỚC, code SAU
   - Vẽ hình trên whiteboard
   - Xác định entities và relationships
   - Tạo tables với proper keys
   - INSERT normalized data
   - JOIN để reconstruct UI

"You just learned the most important 60% of SQL."
— Dr. Chuck

Tiếp theo: Intermediate PostgreSQL

- Aggregate functions (GROUP BY, HAVING)
- Subqueries
- Stored procedures
- Text operations & regular expressions
- Advanced indexing
```

---

## 6. Tổng kết toàn khoá — SQL Overview

```
4 modules đã học:

Module 1: Introduction to SQL
  → Lịch sử, architecture, CRUD cơ bản, psql setup

Module 2: Single Table SQL  
  → Data types, keys, indexes (B-Tree/Hash)
  → WHERE, ORDER BY, LIKE, LIMIT, COUNT

Module 3: One-To-Many
  → Database design, normalization
  → SERIAL, FOREIGN KEY, ON DELETE CASCADE
  → INNER JOIN, CROSS JOIN

Module 4: Many-To-Many (module này)
  → Junction tables, composite PRIMARY KEY
  → Data at connection point
  → Multi-table JOIN through junction

Kỹ năng đạt được:
✅ Thiết kế data model từ requirements
✅ Viết CRUD operations
✅ Tạo normalized multi-table schemas
✅ JOIN data across tables
✅ Hiểu indexes và performance concepts
```

---

## ❓ Câu Hỏi Kiểm Tra

1. Dấu hiệu nào cho thấy cần many-to-many thay vì one-to-many?
2. Junction table là gì? Có mấy FK? FK trỏ đi đâu?
3. `PRIMARY KEY(student_id, course_id)` — giải thích composite PK.
4. Tại sao junction table thường KHÔNG cần `id SERIAL`?
5. "Data at connection point" nghĩa là gì? Cho 3 ví dụ.
6. Viết JOIN cho many-to-many: book ↔ author qua book_author.
7. Tại sao tạo leaf tables TRƯỚC junction table?
8. ON DELETE CASCADE trong member table: xóa student → gì xảy ra?
9. Khi nào junction table CẦN `id SERIAL`? (hint: forum comments)
10. Tổng kết: "the most important 60% of SQL" bao gồm những gì?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is a many-to-many relationship? How do you implement it in SQL?**  
> A: Many-to-many xảy ra khi cả hai bên có thể liên kết với nhiều entities bên kia (students ↔ courses). Implement bằng junction table (bridge table) chứa 2 foreign keys trỏ đến 2 bảng chính. Primary key of junction table thường là composite key kết hợp 2 FK. Junction table có thể model additional data tại connection point (role, date, etc.).

> **Q: What is a composite primary key?**  
> A: Composite primary key = primary key gồm 2+ columns. Ví dụ: `PRIMARY KEY(student_id, course_id)` đảm bảo combination duy nhất — 1 student chỉ xuất hiện 1 lần per course. Không cần auto-increment id riêng nếu combination đã đủ unique.

> **Q: When would you use a junction table vs adding more foreign key columns?**  
> A: LUÔN dùng junction table khi relationship là many-to-many. Thêm FK columns (album_id1, album_id2, album_id3...) là anti-pattern vì: (1) không biết cần bao nhiêu columns, (2) hầu hết sẽ NULL → lãng phí, (3) query phức tạp (phải check TẤT CẢ columns), (4) không scalable.

> **Q: Can you explain the difference between one-to-many and many-to-many with examples?**  
> A: **One-to-many**: 1 department có nhiều employees, nhưng 1 employee chỉ thuộc 1 department. Implement: FK `dept_id` trong employee table. **Many-to-many**: 1 student enrolled nhiều courses, 1 course có nhiều students. Implement: junction table `enrollment(student_id, course_id, grade)` với 2 FK.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Tạo hệ thống Book ↔ Author (many-to-many). 1 book nhiều authors, 1 author nhiều books. Junction table `book_author` có thêm `contribution_role` (author, editor, translator). Insert 5 books, 5 authors, 10+ connections. Viết JOIN show book.title, author.name, contribution_role.

**BT2**: Tạo hệ thống Movie ↔ Actor (many-to-many). Junction table `cast` có `role_name` (character name trong phim). Insert data cho ít nhất 3 phim. Viết query: (a) tất cả diễn viên trong phim X, (b) tất cả phim của diễn viên Y.

**BT3**: Mở rộng Music Database từ Module 3: thay đổi Track ↔ Album thành many-to-many (1 track có thể ở nhiều albums). Tạo junction table `track_album`. Migrate data từ old FK sang junction table.

**BT4**: Thiết kế COMPLETE data model cho hệ thống "Online Course Platform" bao gồm: Users, Courses, Enrollments (role: student/instructor), Assignments, Submissions, Grades. Xác định one-to-many và many-to-many relationships. Vẽ diagram và viết CREATE TABLE statements.

---

## ✅ Checklist Hoàn Thành Module 4

1. Nhận biết KHI NÀO cần many-to-many (dấu hiệu "thêm 1 cột nữa").
2. Thiết kế junction table với 2 FK + composite PK.
3. Model data tại connection point (role, date, etc.).
4. Insert normalized data cho many-to-many.
5. Viết JOIN qua junction table (3-table JOIN).
6. Phân biệt rõ one-to-many vs many-to-many.
7. Thiết kế data model HOÀN CHỈNH cho bất kỳ ứng dụng nào.
8. Hoàn thành tất cả assignments trên Coursera.

---

## 📎 Tham Khảo

- Đáp Án: [99-answer-key-db.md#drchuck-m4](./99-answer-key-db.md#drchuck-m4)
- Lý thuyết sâu: [97-db-theory-deep-dive.md#drchuck-m4-deep](./97-db-theory-deep-dive.md#drchuck-m4-deep)
