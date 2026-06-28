# 🧩 Phần 15: Capstone Integration (C# + SQL + Quality)

> **Mục tiêu**: chuyển toàn bộ kiến thức đã học thành một sản phẩm có thể demo, có test, có đo hiệu năng, có dữ liệu thật.  
> **Thời lượng ước tính**: 20-35 giờ.

---

## 🎯 Mục Tiêu

- Thiết kế và triển khai một ứng dụng hoàn chỉnh theo kiến trúc rõ ràng.
- Tích hợp C# business logic với PostgreSQL.
- Có bộ test tự động cho logic chính.
- Chứng minh được code quality, error handling, logging, performance cơ bản.
- Tạo portfolio project đủ tốt để dùng khi phỏng vấn.

---

## ✅ Điều Kiện Bắt Đầu

Bạn nên hoàn thành trước:

1. `01` -> `14` trong track C#.
2. `db/01` -> `db/05` trong track SQL.
3. Tối thiểu 8 bài tập mức trung bình/khó đã tự làm.

---

## 1. Đề Tài Bắt Buộc: Learning Progress Tracker

### 1.1 Mô tả bài toán

Xây dựng ứng dụng quản lý tiến độ học tập:

1. Quản lý khóa học, module, bài học.
2. Ghi nhận tiến độ từng người học.
3. Thống kê thời gian học, tỷ lệ hoàn thành, top khó khăn.
4. Xuất báo cáo theo ngày/tuần/tháng.

### 1.2 Phạm vi chức năng tối thiểu

1. CRUD cho `Learner`, `Course`, `Module`, `Lesson`.
2. Đánh dấu hoàn thành bài học và lưu thời gian học.
3. Tính progress theo `%`.
4. Báo cáo top 5 modules có tỷ lệ fail cao nhất.
5. Import dữ liệu hoạt động từ file CSV (async).

---

## 2. Kiến Trúc Gợi Ý

```text
src/
  LearningTracker.App/          # Console UI hoặc Minimal API
  LearningTracker.Core/         # Entities, Value Objects, Business Rules
  LearningTracker.Infrastructure/ # PostgreSQL, repository, file I/O
tests/
  LearningTracker.Tests/        # Unit tests
```

Nguyên tắc:

1. `Core` không phụ thuộc hạ tầng.
2. `Infrastructure` implement interfaces từ `Core`.
3. `App` orchestrate use cases, không chứa business rules nặng.

---

## 3. Mapping Kiến Thức Theo Module

| Kỹ năng | Bạn phải thể hiện trong capstone |
|--------|----------------------------------|
| OOP + OOP Supplement | Entity/Value Object rõ ràng, invariant chặt |
| Exceptions | Error boundaries + custom exception cho domain |
| Generics + LINQ | Query/filter/transform data hợp lý |
| .NET Internals | `IDisposable`, xử lý file, memory awareness |
| Collections | Chọn cấu trúc dữ liệu phù hợp |
| Projects | Multi-project solution rõ ràng |
| Unit Testing | Test business logic cốt lõi + edge cases |
| Clean Code | Naming, method size, abstraction level |
| Async | Import/export hoặc I/O theo `async/await` |
| SQL | Schema + query báo cáo + tối ưu cơ bản |

---

## 4. Milestones (Gợi Ý 4 Sprint)

### Sprint 1: Domain + Schema

1. Chốt ERD và use cases.
2. Tạo schema SQL.
3. Cài skeleton solution nhiều project.

### Sprint 2: Core Features

1. Implement CRUD + invariants.
2. Kết nối PostgreSQL.
3. Viết test cho luồng chính.

### Sprint 3: Reporting + Async

1. Viết query báo cáo.
2. Import CSV async + cancellation token.
3. Bổ sung logging và error handling.

### Sprint 4: Hardening + Demo

1. Refactor clean code.
2. Benchmark điểm nóng (query hoặc text processing).
3. Hoàn thiện README + script demo.

---

## 5. Yêu Cầu Kỹ Thuật Bắt Buộc

1. Có ít nhất 1 custom exception chứa domain context.
2. Có tối thiểu 30 unit tests cho business layer.
3. Không hardcode connection string trong source code.
4. Ít nhất 3 truy vấn SQL có `JOIN` và 2 truy vấn có aggregate/window.
5. Có ít nhất 1 tác vụ async thật (file I/O hoặc network).
6. Có logging đủ để trace một request end-to-end.

---

## 6. Rubric Chấm Capstone (100 điểm)

| Hạng mục | Điểm tối đa | Tiêu chí |
|----------|-------------|---------|
| Domain Modeling | 20 | đúng thực thể, invariants, không anemic |
| Code Quality | 15 | clean code, structure, readability |
| Testing | 20 | coverage logic trọng yếu, test edge cases |
| SQL + Data Design | 20 | schema hợp lý, query chính xác và rõ |
| Reliability | 10 | error handling, logging, resource cleanup |
| Performance | 10 | có đo và tối ưu ít nhất 1 bottleneck |
| Demo/Docs | 5 | hướng dẫn chạy và trình bày rõ |

---

## 7. Deliverables Phải Có

1. Source code đầy đủ.
2. Script SQL tạo schema + seed data.
3. Test suite chạy được.
4. README gồm:
   - kiến trúc,
   - cách chạy local,
   - quyết định thiết kế quan trọng,
   - known limitations.
5. Demo script 5-10 phút (văn bản hoặc video ngắn).

---

## 8. Failure Modes Thường Gặp

1. Đẩy business logic vào UI layer.
2. DB schema thay đổi liên tục nhưng không có migration discipline.
3. Test chỉ happy-path, thiếu cases lỗi.
4. Async dùng sai (fire-and-forget không kiểm soát).
5. Nhiều class nhưng boundaries mơ hồ.

---

## 9. Stretch Goals (Tùy Chọn)

1. Thêm authorization role-based.
2. Thêm caching có invalidation policy.
3. Thêm dashboard đơn giản (web UI hoặc export charts).
4. Đóng gói Docker cho app + DB local.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Viết sequence diagram cho luồng "Mark lesson completed" từ app -> core -> DB -> response.

**BT2**: Thiết kế 5 test cases edge cho nghiệp vụ enrollment (trùng khóa, khóa đóng, dữ liệu thiếu...).

**BT3**: Tạo một bug có chủ đích trong query báo cáo, rồi dùng test + logging để tìm và sửa.

---

## ❓ Câu Hỏi Kiểm Tra

1. Domain rule nào là quan trọng nhất và bạn bảo vệ nó bằng cách nào?
2. Tại sao bạn chọn kiến trúc hiện tại?
3. Query nào là đắt nhất và bạn đã tối ưu ra sao?
4. Nếu tăng dữ liệu gấp 100 lần, bottleneck đầu tiên nằm ở đâu?
5. Test nào bảo vệ được bug nghiêm trọng nhất?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: Tell me about a project where you applied OOP and SQL together.**  
> A: Trình bày domain, schema, use cases, trade-offs, lỗi gặp phải, và cách bạn validate bằng tests.

> **Q: What was the hardest design decision?**  
> A: Chọn một trade-off thật (ví dụ normalized vs denormalized, sync vs async) và giải thích vì sao.

> **Q: How did you ensure code quality?**  
> A: Dùng tests, refactoring loops, logging, code review checklist, và đo performance có số liệu.

---

## ✅ Definition of Completion

Bạn hoàn thành capstone khi:

1. Dự án chạy ổn định trên máy mới theo README.
2. Test pass và bảo vệ được luồng nghiệp vụ trọng yếu.
3. Có thể demo live mà không phụ thuộc "chạy may mắn".
4. Trả lời được câu hỏi "vì sao thiết kế như vậy" ở từng phần.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p15-capstone](./99-answer-key-csharp.md#p15-capstone)
- Bài tập thực hành: [99-answer-key-csharp.md#p15-capstone-exercises](./99-answer-key-csharp.md#p15-capstone-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p15-capstone-deep](./97-csharp-theory-deep-dive.md#p15-capstone-deep)

