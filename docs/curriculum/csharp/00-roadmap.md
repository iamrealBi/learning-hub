# 🎯 C# Masterclass — Roadmap Tổng Quan (Bản Hoàn Thiện)

> **Mục tiêu**: học C# bài bản theo hướng đi làm thực tế, cân bằng giữa lý thuyết, code tay, debug, testing và thiết kế.  
> **Tổng thời lượng khuyến nghị**: 120-160 giờ (ít nhất 60% thời gian cho thực hành).

---

## ✅ Chuẩn Đầu Ra (Definition of Done)

Sau khi hoàn thành toàn bộ roadmap, bạn cần làm được:

1. Viết và tổ chức được ứng dụng C# nhiều project (App/Core/Tests), dùng OOP đúng ngữ cảnh.
2. Tự thiết kế và triển khai luồng xử lý dữ liệu từ input -> validate -> business logic -> output.
3. Xử lý lỗi có chủ đích: throw/catch đúng tầng, log được lỗi và giữ stack trace chuẩn.
4. Sử dụng thành thạo `Generics`, `LINQ`, `Collections`, `async/await`, `Events`.
5. Viết unit test có chất lượng (AAA, parameterized tests, mocking, edge-case coverage).
6. Tự đánh giá code theo Clean Code + SOLID + hiệu năng cơ bản.
7. Thiết kế schema quan hệ, viết SQL từ cơ bản đến nâng cao, đọc được query plan cơ bản.
8. Hoàn thành ít nhất 2 capstone tích hợp C# + SQL (chi tiết ở cuối file).

---

## 🎓 Quy Trình Học Mỗi Module (Bắt Buộc)

1. **Đọc khái niệm**: hiểu mục tiêu, thuật ngữ, bối cảnh dùng.
2. **Chạy lại ví dụ**: gõ lại code bằng tay, không copy-paste.
3. **Biến thể hóa**: tự sửa ví dụ theo 2-3 kịch bản mới.
4. **Làm bài tập**: ưu tiên bài mức trung bình/khó trước khi qua bài mới.
5. **Tự giảng lại**: giải thích bằng lời 5-10 phút như đang dạy người khác.
6. **Retrospective**: ghi lại lỗi sai, điều chưa chắc, câu hỏi mở để ôn lại.

> Nếu chưa tự giải thích được "vì sao làm vậy", xem như chưa xong module.

---

## 📋 Lộ Trình Học Chính

| # | Module | Trọng tâm | Tài liệu | Giờ gợi ý |
|---|--------|-----------|----------|-----------|
| 1 | C# Fundamentals | Biến, điều kiện, vòng lặp, mảng/list, methods, debug | [01-fundamentals.md](./01-fundamentals.md) | 10-14 |
| 2 | OOP Core | Class, object, constructor, property, encapsulation | [02-oop.md](./02-oop.md) | 8-12 |
| 3 | OOP Supplement | Deep OOP thinking, modeling, anti-patterns | [02b-oop-supplement.md](./02b-oop-supplement.md) | 5-8 |
| 4 | Exceptions | Lỗi runtime, custom exceptions, error boundaries | [03-exceptions.md](./03-exceptions.md) | 4-6 |
| 5 | Generics + Delegates | Generic types/methods, constraints, strategies | [04-generics.md](./04-generics.md) | 5-7 |
| 6 | LINQ | Query mindset, deferred execution, transformations | [05-linq.md](./05-linq.md) | 4-6 |
| 7 | .NET Internals | Stack/heap, GC, IDisposable, memory patterns | [06-dotnet-internals.md](./06-dotnet-internals.md) | 5-8 |
| 8 | Advanced Types | Struct/record/nullable/reflection/operators | [07-advanced-types.md](./07-advanced-types.md) | 5-8 |
| 9 | Collections | Interfaces, data structures, iterators, Big-O | [08-collections.md](./08-collections.md) | 5-7 |
| 10 | Projects + Strings | Solution structure, assembly, text processing | [09-projects-strings.md](./09-projects-strings.md) | 4-6 |
| 11 | Numerics + Events | Numeric safety, delegates/events, observer model | [10-numerics-events.md](./10-numerics-events.md) | 4-6 |
| 12 | Unit Testing | NUnit, mocks, test design, anti-patterns test | [11-unit-testing.md](./11-unit-testing.md) | 5-8 |
| 13 | Clean Code | Naming, abstraction levels, composition, refactoring | [12-clean-code.md](./12-clean-code.md) | 4-7 |
| 14 | Async & Multithreading | Task/TPL, cancellation, sync primitives | [13-async.md](./13-async.md) | 6-9 |
| 15 | C# Evolution | C# 11/12+ features, migration mindset | [14-evolution.md](./14-evolution.md) | 2-4 |
| 16 | Capstone | Tích hợp toàn bộ kiến thức vào sản phẩm hoàn chỉnh | [15-capstone-integration.md](./15-capstone-integration.md) | 20-35 |

---

## 🗄️ Database & SQL Track (Song Song)

| # | Module | Trọng tâm | Tài liệu | Giờ gợi ý |
|---|--------|-----------|----------|-----------|
| DB1 | Relational Concepts | Mô hình dữ liệu, ERD, ACID, kiến trúc DB | [db/01-relational-concepts.md](db/01-relational-concepts.md) | 4-6 |
| DB2 | SQL Fundamentals | DDL/DML, CRUD, aggregate, GROUP BY/HAVING | [db/02-sql-fundamentals.md](db/02-sql-fundamentals.md) | 6-8 |
| DB3 | Database Design | Keys, normalization, constraints, denormalization | [db/03-database-design.md](db/03-database-design.md) | 5-7 |
| DB4 | JOINs + Subqueries | JOIN patterns, CTE, set operations | [db/04-joins-subqueries.md](db/04-joins-subqueries.md) | 5-7 |
| DB5 | Advanced PostgreSQL | Views, transactions, window functions, tuning | [db/05-advanced-postgresql.md](db/05-advanced-postgresql.md) | 6-9 |
| DB6 | DB Capstone | Thiết kế + tối ưu + tích hợp vào ứng dụng C# | [db/06-db-capstone.md](db/06-db-capstone.md) | 12-20 |

Xem tổng quan DB tại [db/00-db-roadmap.md](db/00-db-roadmap.md).

---

## 🌐 ASP.NET Core Track (Sau khi xong C# + DB cơ bản)

| # | Module | Trọng tâm | Tài liệu | Giờ gợi ý |
|---|--------|-----------|----------|-----------|
| WEB0 | .NET Overview | .NET ecosystem, SDK, CLI, project setup | [aspnet/00-dotnet-overview.md](aspnet/00-dotnet-overview.md) | 3-5 |
| WEB1 | ASP.NET Introduction | MVC, Razor Pages, middleware, routing | [aspnet/01-introduction.md](aspnet/01-introduction.md) | 6-8 |
| WEB2 | Web APIs | REST, controllers, DTOs, model binding, validation | [aspnet/02-web-apis.md](aspnet/02-web-apis.md) | 6-8 |
| WEB3 | Entity Framework Core | Code-First, migrations, LINQ→SQL, relationships | [aspnet/03-ef-core.md](aspnet/03-ef-core.md) | 8-12 |
| WEB4 | Authentication & Security | Identity, JWT, OAuth, authorization | [aspnet/04-auth.md](aspnet/04-auth.md) | 8-12 |

---

## 📚 Ghi Chú Bài Giảng (Coursera)

Ghi chú chi tiết từ bài giảng Dr. Chuck (PostgreSQL) — bổ trợ cho DB track:

| File | Nội dung |
|------|----------|
| [Dr. Chuck M1](coursera-notes/10-drchuck-m1-introduction-to-sql.md) | Lịch sử SQL, CRUD, psql, setup |
| [Dr. Chuck M2](coursera-notes/11-drchuck-m2-single-table-sql.md) | Data types, keys, indexes, CSV |
| [Dr. Chuck M3](coursera-notes/12-drchuck-m3-one-to-many.md) | One-to-Many, JOINs, ON DELETE |
| [Dr. Chuck M4](coursera-notes/13-drchuck-m4-many-to-many.md) | Many-to-Many, junction tables |

---

## 🧪 Quality Gates (Không Qua Gate Thì Không Qua Module)

1. **Code Gate**: hoàn thành tối thiểu 3 bài thực hành/module, có edge-case tests cho bài chính.
2. **Explain Gate**: trả lời được 80% câu hỏi kiểm tra mà không nhìn tài liệu.
3. **Debug Gate**: tự tìm và fix ít nhất 1 bug logic + 1 bug runtime bằng debugger/log.
4. **Refactor Gate**: cải tiến lại một đoạn code cũ theo Clean Code/SOLID.
5. **Performance Gate**: đo tối thiểu một điểm nóng (time/memory) bằng số liệu trước/sau.

---

## 📌 Phần Còn Thiếu Nếu Muốn "Đi Làm Thực Chiến"

Các phần dưới đây nên tìm hiểu thêm sau khi xong các track chính:

1. Observability (structured logging, metrics, tracing, correlation id).
2. Security nâng cao (OWASP, secret management, input sanitization).
3. CI/CD + DevOps cơ bản (Docker, GitHub Actions/Azure DevOps, deployment).
4. Kiến trúc hệ thống (Clean Architecture, modular monolith, messaging patterns).

---

## 🚀 Capstone Bắt Buộc

1. **Capstone ứng dụng C#**: [15-capstone-integration.md](./15-capstone-integration.md)
2. **Capstone cơ sở dữ liệu**: [db/06-db-capstone.md](db/06-db-capstone.md)

Mục tiêu capstone: không chỉ "chạy được", mà phải chứng minh được tư duy thiết kế, kiểm thử, vận hành và tối ưu.

---

## 📎 Đáp Án Tổng Hợp

1. C# track: [99-answer-key-csharp.md](./99-answer-key-csharp.md)
2. DB track: [db/99-answer-key-db.md](db/99-answer-key-db.md)

---

## 📎 Lý Thuyết Đọc Sâu

1. C# deep dive: [97-csharp-theory-deep-dive.md](./97-csharp-theory-deep-dive.md)
2. DB deep dive: [db/97-db-theory-deep-dive.md](db/97-db-theory-deep-dive.md)

---

## 📅 Kế Hoạch Học Gợi Ý (16 Tuần)

| Tuần | Nội dung chính | Output bắt buộc |
|------|----------------|-----------------|
| 1-2 | Fundamentals | 2 mini apps + 1 buổi debug report |
| 3-4 | OOP + OOP Supplement | 1 app OOP hoàn chỉnh + UML đơn giản |
| 5 | Exceptions + Generics | 1 module xử lý lỗi + test các nhánh lỗi |
| 6 | LINQ + Collections | 1 bài xử lý dữ liệu lớn + benchmark |
| 7 | .NET Internals + Advanced Types | 1 báo cáo memory/performance ngắn |
| 8 | Projects + Strings + Numerics | 1 tool xử lý text thực tế |
| 9-10 | Unit Testing + Clean Code | refactor code cũ + nâng coverage |
| 11-12 | Async/Multithreading + C# Evolution | 1 app async có cancellation/retry |
| 13-14 | DB1 -> DB5 | schema + truy vấn nghiệp vụ + tối ưu query |
| 15-16 | Capstone (C# + DB) | dự án tích hợp + demo + retrospective |

---

## 🧭 Cách Dùng Tài Liệu Để "Thấu Hiểu"

1. Đọc ít, code nhiều: tỷ lệ 30/70.
2. Mỗi khái niệm phải có phản ví dụ (counter-example).
3. Mỗi tuần phải có một bản "what I misunderstood".
4. Không học theo cảm giác "đã hiểu"; chỉ chốt khi code/test/debug được.
5. Cứ 2 tuần quay lại làm lại 1 bài cũ bằng cách khác để kiểm tra độ hiểu sâu.
