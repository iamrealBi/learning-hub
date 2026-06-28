# 📚 C# Theory Deep Dive

> Tài liệu bổ sung chiều sâu lý thuyết cho toàn bộ C# track.  
> Mục tiêu: giúp bạn hiểu **vì sao**, **trade-off**, và **khi nào áp dụng** chứ không chỉ chạy được code.

---

## Cách Dùng

1. Học file module chính trước.
2. Mở phần deep dive tương ứng khi cần hiểu bản chất hoặc quyết định thiết kế.
3. Sau khi đọc, quay lại module chính và refactor một ví dụ theo tư duy mới.

---

<a id="p01-fundamentals-deep"></a>
## P01 Fundamentals - Deep Dive

### Mental Model Cốt Lõi

- Program là chuỗi trạng thái thay đổi theo thời gian.
- Biến không chỉ là "chỗ chứa dữ liệu", mà còn là ranh giới của scope và lifetime.
- Kiểu dữ liệu là hợp đồng giúp compiler bảo vệ bạn khỏi lỗi logic sớm.

### Sai Lầm Phổ Biến

1. Ép kiểu tùy tiện mà không nghĩ về mất dữ liệu/overflow.
2. Dùng exception cho input user thay vì `TryParse`.
3. Chọn loop theo thói quen thay vì theo nhu cầu (`index`, mutate, readability).

### Quy Tắc Thực Chiến

1. Input từ user/file/network luôn coi là không tin cậy.
2. So sánh số thực `double/float` bằng epsilon, không so sánh `==` trực tiếp.
3. Mọi loop cần xác định rõ điều kiện dừng và biên.

---

<a id="p02-oop-deep"></a>
## P02 OOP - Deep Dive

### OOP Là Tổ Chức Trách Nhiệm, Không Chỉ Là Cú Pháp

- Class tốt mô hình hóa một khái niệm nghiệp vụ có trạng thái + hành vi.
- Method tốt bảo vệ invariant của object.
- Access modifiers là công cụ bảo vệ, không phải trang trí.

### Heuristics Khi Thiết Kế

1. Nếu phải đọc nhiều class mới hiểu 1 hành vi, thiết kế đang quá phân mảnh.
2. Nếu sửa một rule mà phải chỉnh nhiều file, boundaries đang sai.
3. Nếu class có quá nhiều `set`, khả năng cao đang thiếu encapsulation.

### Inheritance vs Composition

- Inheritance mạnh khi có quan hệ `is-a` rõ ràng và base contract ổn định.
- Composition mạnh khi cần linh hoạt hành vi và giảm coupling dài hạn.

### Dấu Hiệu Thiết Kế Đang Đi Sai

1. Nhiều class "Manager/Helper/Util" chứa logic hỗn tạp.
2. Object domain không có method nghiệp vụ, chỉ có DTO-style properties.
3. Dùng `new` trực tiếp trong business logic thay vì inject abstraction.

---

<a id="p02b-oop-supplement-deep"></a>
## P02b OOP Supplement - Deep Dive

### Entity, Value Object, Service

- Entity có identity xuyên thời gian.
- Value Object đại diện ý nghĩa của dữ liệu, nên immutable.
- Service điều phối nhiều object, không chiếm logic thuộc về entity/value object.

### Invariant-Driven Design

1. Không cho object tồn tại ở trạng thái invalid.
2. Validate ở constructor/factory và method đổi state.
3. Đừng để external code sửa state quan trọng không kiểm soát.

---

<a id="p03-exceptions-deep"></a>
## P03 Exceptions - Deep Dive

### Error Taxonomy

- **Expected failure**: có thể xảy ra thường xuyên (input sai) -> dùng `TryXxx`.
- **Unexpected failure**: vi phạm assumptions hoặc lỗi môi trường -> exception.

### Nguyên Tắc Kiến Trúc

1. Throw sớm ở boundary của module khi invariant bị phá.
2. Catch ở nơi có thể xử lý hoặc chuyển đổi context có ý nghĩa.
3. Không nuốt lỗi im lặng.

### Rethrow Đúng Cách

- Trong `catch`, ưu tiên `throw;` để giữ stack trace gốc.

---

<a id="p04-generics-deep"></a>
## P04 Generics - Deep Dive

### Vì Sao Generics Quan Trọng

- Type safety ở compile-time.
- Tránh boxing/unboxing không cần thiết.
- Reuse logic mà vẫn giữ semantics theo type cụ thể.

### Constraints Là Công Cụ Thiết Kế

- Constraint không chỉ để compile pass; nó thể hiện điều kiện tối thiểu của algorithm.
- Ví dụ `where T : IComparable<T>` nói rõ hàm cần khả năng so sánh.

---

<a id="p05-linq-deep"></a>
## P05 LINQ - Deep Dive

### Declarative Thinking

- LINQ mô tả "muốn gì" thay vì "lặp như thế nào".
- Chất lượng LINQ nằm ở pipeline rõ ràng, không nằm ở viết ngắn.

### Deferred Execution Trade-off

- Ưu điểm: lazy, tiết kiệm tài nguyên.
- Rủi ro: query re-run nhiều lần ngoài ý muốn, gây bug hiệu năng.

### Quy Tắc

1. Nếu nguồn data thay đổi theo thời gian, cân nhắc materialize (`ToList`) khi cần snapshot.
2. Tránh side effects trong `Select/Where`.

---

<a id="p06-dotnet-internals-deep"></a>
## P06 .NET Internals - Deep Dive

### Stack, Heap, Lifetime

- Câu hỏi quan trọng không chỉ là "nằm ở đâu", mà là "sống bao lâu" và "ai giữ reference".

### GC Và Thiết Kế

- GC không miễn phí; allocation quá nhiều trong hot path sẽ tạo áp lực.
- Tối ưu đúng chỗ: đo trước khi tối ưu.

### Resource Management

- Managed memory != unmanaged resource.
- File handles, sockets, database connections vẫn cần `Dispose`.

---

<a id="p07-advanced-types-deep"></a>
## P07 Advanced Types - Deep Dive

### Equality Semantics

- Value semantics cần `Equals` + `GetHashCode` nhất quán.
- Khi dùng làm key trong hash collections, hash contract là bắt buộc.

### Struct Design

- Dùng struct cho object nhỏ, immutable, value-centric.
- Tránh struct lớn mutable vì copy cost và bug khó đoán.

---

<a id="p08-collections-deep"></a>
## P08 Collections - Deep Dive

### Data Structure First, API Second

- Chọn cấu trúc theo thao tác chủ đạo:
  - lookup -> hash-based
  - ordered traversal -> list/tree
  - FIFO/LIFO -> queue/stack

### Complexity Literacy

- Cần phân biệt worst-case và amortized.
- Big-O là khung tư duy, không thay thế benchmark thực tế.

---

<a id="p09-projects-strings-deep"></a>
## P09 Projects & Strings - Deep Dive

### Solution Architecture

- Tách project theo trách nhiệm:
  - Core (domain/use-cases)
  - Infrastructure (IO/DB/External)
  - App (composition root/UI/API)

### String Handling

- String immutable giúp safety và predictability.
- Kết hợp `StringBuilder` + culture-aware formatting trong data-heavy workflows.

---

<a id="p10-numerics-events-deep"></a>
## P10 Numerics & Events - Deep Dive

### Numeric Correctness

- Chọn kiểu số là quyết định nghiệp vụ:
  - tiền tệ -> `decimal`
  - scientific/graphics -> `double`

### Event Architecture

- Event giúp decoupling theo mô hình publish-subscribe.
- Cần kiểm soát lifecycle subscription để tránh memory leaks.

---

<a id="p11-unit-testing-deep"></a>
## P11 Unit Testing - Deep Dive

### Test Value Pyramid

1. Behavioral correctness.
2. Regression safety.
3. Documentation của business rules.

### Test Smells

- Test phụ thuộc thời gian thực, random, external IO.
- Test assert quá chung chung, khó xác định lỗi.

### Nguyên Tắc

- Một test nên thất bại vì một lý do chính.
- Tên test phải mô tả condition + expected outcome.

---

<a id="p12-clean-code-deep"></a>
## P12 Clean Code - Deep Dive

### Clean Code Là Tối Ưu Chi Phí Đổi Thay

- Code "chạy được" hôm nay không đủ nếu team phải bảo trì dài hạn.
- Chất lượng nằm ở khả năng thay đổi nhanh mà không phá behavior.

### Practical Refactoring Loop

1. Viết test bảo vệ behavior.
2. Refactor bước nhỏ.
3. Chạy lại test.
4. Lặp đến khi structure rõ ràng.

---

<a id="p13-async-deep"></a>
## P13 Async - Deep Dive

### Async Mental Model

- Async nhằm giải phóng thread khỏi việc chờ I/O.
- Không phải cứ async là chạy song song.

### Fault/Cancellation Model

- Bắt lỗi đúng tầng, propagate hợp lý.
- Cancellation là một phần của contract, không phải add-on sau cùng.

### Production Heuristics

1. Tránh `async void` trừ event handlers.
2. Tránh `.Result/.Wait()` trên code async.
3. Giới hạn concurrency có chủ đích.

---

<a id="p14-evolution-deep"></a>
## P14 C# Evolution - Deep Dive

### Feature Adoption Strategy

- Không áp dụng feature mới chỉ vì "mới".
- Đánh giá theo tiêu chí:
  - giảm bug?
  - tăng readability?
  - team có maintain nổi không?

### Migration Risk

- Mismatch SDK/TFM/IDE.
- Code style không đồng nhất khi refactor dở dang.

---

<a id="p15-capstone-deep"></a>
## P15 Capstone - Deep Dive

### Outcome Quan Trọng Nhất

- Chứng minh được khả năng biến requirement thành design, implementation, test, và vận hành có số liệu.

### Review Checklist Trước Khi Demo

1. Business rules nào đang được enforce ở đâu?
2. Test nào bảo vệ bug nghiêm trọng nhất?
3. Query/logic nào là bottleneck và có dữ liệu đo chưa?
4. Kiến trúc có ranh giới rõ giữa domain và infrastructure chưa?

---

## Kết Luận

- Mỗi module đều có thể học bằng cú pháp.
- Nhưng để "đi làm được", bạn phải hiểu design rationale và trade-off.
- Hãy dùng tài liệu này như lớp thứ hai: sau cú pháp là tư duy.
