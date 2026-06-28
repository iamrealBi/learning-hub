# 📘 C# Answer Key (Câu Hỏi + Bài Tập)

> Tài liệu này gom đáp án gợi ý cho toàn bộ track C#. Mục tiêu là giúp tự kiểm tra hiểu biết, không thay thế việc tự code.

---

## Cách Dùng

1. Tự làm câu hỏi/bài tập trước.
2. Chỉ mở đáp án khi đã thử ít nhất 20-30 phút.
3. Với bài code, so sánh theo tiêu chí: đúng logic, xử lý edge-case, đọc dễ, test được.

---

<a id="p01-fundamentals"></a>
## P01 Fundamentals - Đáp Án Câu Hỏi

1. C# compile sang IL/CIL để runtime (.NET CLR) có thể JIT thành machine code trên nhiều nền tảng.
2. `JIT` biên dịch lúc chạy; `AOT` biên dịch trước khi chạy.
3. `decimal` cho tiền tệ; `double/float` cho số thực kỹ thuật.
4. `TryParse` an toàn cho input user; `Parse` ném exception khi sai format.
5. `for` khi cần index; `foreach` khi chỉ duyệt phần tử.
6. `Array` kích thước cố định; `List<T>` co giãn động.
7. `const` là compile-time constant; `readonly` gán ở ctor/runtime init.
8. `break` thoát loop/switch; `continue` bỏ qua iteration hiện tại; `return` thoát method.

<a id="p01-fundamentals-exercises"></a>
### P01 Fundamentals - Đáp Án Bài Tập

- `BT1 Swap`
```csharp
int a = 5, b = 10;
(a, b) = (b, a);
Console.WriteLine($"a={a}, b={b}");
```
- `BT1 (không tuple, tránh biến tạm)`
```csharp
int a = 5, b = 10;
a = a + b;
b = a - b;
a = a - b;
```
- Lưu ý: cách cộng/trừ có rủi ro overflow với số lớn.
- `BT2 FizzBuzz`: kiểm tra `% 15` trước `% 3` và `% 5`.
- `BT4 IsPrime`: trả `false` cho `n < 2`, chỉ duyệt đến `sqrt(n)`.
- `BT8 BubbleSort`: dừng sớm nếu không có lần swap nào trong 1 pass.

---

<a id="p02-oop"></a>
## P02 OOP - Đáp Án Câu Hỏi

1. Class là blueprint; object là instance cụ thể.
2. Encapsulation: bảo vệ state qua `private` field + method/property có validate.
3. `virtual/override` cho runtime polymorphism; `new` là method hiding.
4. C# không hỗ trợ multiple inheritance cho class; dùng nhiều interfaces.
5. Interface là contract; abstract class có thể chứa state + implementation.
6. DIP: high-level module phụ thuộc abstraction thay vì concrete class.

<a id="p02-oop-exercises"></a>
### P02 OOP - Đáp Án Bài Tập

- `Shape Calculator`: dùng abstract `Shape` + `Area()/Perimeter()` override.
- `Payment System`: inject `IPaymentProcessor` qua ctor, không `new` processor trong service.
- `Bank Account`: enforce rule rút tiền trong method `Withdraw`, không để set balance trực tiếp.
- `Plugin System`: `PluginManager` quản lý `List<IPlugin>` và chạy tuần tự qua `RunAll()`.

---

<a id="p02b-oop-supplement"></a>
## P02b OOP Supplement - Đáp Án Câu Hỏi

1. Anemic model: object chỉ chứa data, logic dồn sang service.
2. Entity so sánh theo identity; Value Object so sánh theo value.
3. Invariant đặt tại ctor/factory và các method đổi state.
4. Composition tốt khi behavior cần thay runtime hoặc tránh hierarchy sâu.
5. Primitive obsession gây code khó hiểu và validate phân tán.

<a id="p02b-oop-supplement-exercises"></a>
### P02b OOP Supplement - Đáp Án Bài Tập

- Refactor từ data model sang behavior-rich model bằng cách:
  - chặn setter state quan trọng,
  - thêm method nghiệp vụ có validate,
  - viết test cho invariant.
- `EmailAddress` value object: validate format, immutable.
- Chuyển `switch discount` thành strategy classes (`IPricingPolicy`).

---

<a id="p03-exceptions"></a>
## P03 Exceptions - Đáp Án Câu Hỏi

1. `throw;` giữ stack trace gốc; `throw ex;` làm mất stack trace ban đầu.
2. Không dùng exception cho control flow bình thường; ưu tiên `TryXxx`.
3. Chỉ catch khi có thể xử lý/khôi phục hoặc thêm context rồi rethrow.
4. Custom exception kế thừa trực tiếp `Exception`.
5. `finally` gần như luôn chạy, trừ một số tình huống process chết bất thường.

<a id="p03-exceptions-exercises"></a>
### P03 Exceptions - Đáp Án Bài Tập

- `SafeDivide`: return `null` khi mẫu số bằng 0.
- `RetryAsync`: loop với `maxRetries`, `try/catch`, delay backoff đơn giản.
- CSV parser: bắt lỗi từng dòng, log số dòng, tiếp tục xử lý.

---

<a id="p04-generics"></a>
## P04 Generics - Đáp Án Câu Hỏi

1. Generics giảm lặp code và tăng type safety.
2. Constraint giới hạn kiểu hợp lệ cho `T` (`where T : class/new()/IComparable<T>`...).
3. `Func` có return, `Action` không return.
4. Lambda là cú pháp ngắn cho delegate.
5. `Dictionary<TKey, TValue>` tối ưu lookup trung bình O(1).

<a id="p04-generics-exercises"></a>
### P04 Generics - Đáp Án Bài Tập

- `Generic repository/service`: tách interface chung, implement theo type.
- `Strategy pattern`: inject strategy qua constructor, không `if/switch` dày đặc.
- `Decorator`: bọc behavior (logging/cache) mà không sửa class gốc.

---

<a id="p05-linq"></a>
## P05 LINQ - Đáp Án Câu Hỏi

1. LINQ là truy vấn trên object collections (`IEnumerable<T>`, `IQueryable<T>`).
2. Deferred execution: query chỉ chạy khi enumerate.
3. `Any` kiểm tra tồn tại, `All` kiểm tra mọi phần tử, `Count` đếm.
4. `First` lỗi nếu rỗng; `FirstOrDefault` trả default.
5. `Select` projection, `Where` filtering, `GroupBy` grouping.

<a id="p05-linq-exercises"></a>
### P05 LINQ - Đáp Án Bài Tập

- Ưu tiên pipeline rõ ràng: `Where -> Select -> OrderBy -> ToList`.
- Tránh enumerate nhiều lần cùng query nếu nguồn tốn kém; materialize khi cần.
- Khi nhóm dữ liệu, model output rõ (`new { Key, Count, Avg }` hoặc DTO riêng).

---

<a id="p06-dotnet-internals"></a>
## P06 .NET Internals - Đáp Án Câu Hỏi

1. Stack cho call frames và data nhỏ sống ngắn; heap cho object sống động.
2. Boxing: value type -> object; unboxing cần cast ngược.
3. GC theo generations (0,1,2) để tối ưu thu gom.
4. `IDisposable` để dọn unmanaged resources đúng lúc.
5. `using` đảm bảo `Dispose()` được gọi tự động.

<a id="p06-dotnet-internals-exercises"></a>
### P06 .NET Internals - Đáp Án Bài Tập

- CSV processing: stream từng dòng, không read all nếu file lớn.
- Dùng `using` cho `FileStream/StreamReader`.
- Hạn chế boxing trong vòng lặp lớn (tránh cast qua `object` không cần thiết).

---

<a id="p07-advanced-types"></a>
## P07 Advanced Types - Đáp Án Câu Hỏi

1. Reflection dùng để inspect metadata/type runtime.
2. Attribute gắn metadata cho type/member để đọc bằng reflection.
3. Struct là value type, phù hợp object nhỏ, immutable.
4. Override `Equals/GetHashCode` để so sánh value semantics chuẩn.
5. Record hỗ trợ value-based equality và cú pháp gọn.

<a id="p07-advanced-types-exercises"></a>
### P07 Advanced Types - Đáp Án Bài Tập

- Nếu type dùng làm key trong `Dictionary/HashSet`, bắt buộc hash/equality nhất quán.
- Với nullable reference types, annotate rõ `?` và xử lý null flow.
- Chỉ overloading operator khi ngữ nghĩa tự nhiên (e.g. vector/money cùng currency).

---

<a id="p08-collections"></a>
## P08 Collections - Đáp Án Câu Hỏi

1. `IEnumerable<T>` chỉ đảm bảo iterate.
2. `ICollection<T>` thêm count + add/remove.
3. `IList<T>` thêm truy cập theo index.
4. `HashSet<T>` lookup nhanh O(1) trung bình, không giữ thứ tự.
5. `Queue` FIFO, `Stack` LIFO, `LinkedList` chèn/xóa node nhanh khi đã có node ref.

<a id="p08-collections-exercises"></a>
### P08 Collections - Đáp Án Bài Tập

- Chọn cấu trúc theo thao tác chính thay vì theo thói quen.
- Custom iterator: dùng `yield return` để lazy generate.
- Với API read-only, expose `IReadOnlyCollection<T>`/`IReadOnlyList<T>`.

---

<a id="p09-projects-strings"></a>
## P09 Projects & Strings - Đáp Án Câu Hỏi

1. Solution chứa nhiều projects; mỗi project build thành assembly.
2. Debug ưu tiên khả năng debug; Release ưu tiên optimization.
3. `String` immutable; nối nhiều trong loop nên dùng `StringBuilder`.
4. `CultureInfo.InvariantCulture` dùng cho persist/serialization.
5. NuGet là package manager của .NET.

<a id="p09-projects-strings-exercises"></a>
### P09 Projects & Strings - Đáp Án Bài Tập

- So sánh performance string concat vs `StringBuilder` bằng `Stopwatch`.
- `ReverseWords`: split, reverse array, join.
- `IsAnagram`: normalize + sort chars hoặc đếm tần suất ký tự.

---

<a id="p10-numerics-events"></a>
## P10 Numerics & Events - Đáp Án Câu Hỏi

1. `0.1 + 0.2 != 0.3` vì binary floating-point representation.
2. `decimal` cho tiền, `double` cho tính toán khoa học.
3. `checked` ném `OverflowException`; `unchecked` wrap-around.
4. Event là delegate có encapsulation tốt hơn (ngoài class không tự raise được).
5. Event leak xảy ra khi publisher sống lâu giữ ref đến subscriber.

<a id="p10-numerics-events-exercises"></a>
### P10 Numerics & Events - Đáp Án Bài Tập

- `TemperatureMonitor`: raise event khi delta > ngưỡng.
- `Permission flags`: dùng enum `[Flags]`, `|` để cấp quyền, `&` để kiểm tra.
- Luôn unsubscribe ở `Dispose()` hoặc vòng đời kết thúc.

---

<a id="p11-unit-testing"></a>
## P11 Unit Testing - Đáp Án Câu Hỏi

1. Unit test kiểm tra hành vi đơn vị nhỏ, độc lập.
2. AAA: Arrange, Act, Assert.
3. Test name nên mô tả điều kiện + kỳ vọng.
4. Mock dùng khi dependency ngoài làm test chậm/khó kiểm soát.
5. Test tốt phải deterministic, nhanh, và độc lập.

<a id="p11-unit-testing-exercises"></a>
### P11 Unit Testing - Đáp Án Bài Tập

- Mỗi rule nghiệp vụ có ít nhất 1 happy path + 1 invalid path.
- Với boundary values, test min/max/off-by-one.
- Verify interaction khi business rule yêu cầu gọi dependency.

---

<a id="p12-clean-code"></a>
## P12 Clean Code - Đáp Án Câu Hỏi

1. Tên biến/hàm nên mô tả ý nghĩa nghiệp vụ.
2. Method nhỏ, một nhiệm vụ, abstraction level nhất quán.
3. Comment chỉ nên giải thích "vì sao", không lặp lại "cái gì".
4. Composition thường linh hoạt hơn inheritance trong thay đổi dài hạn.
5. Principle of Least Surprise: API hành xử đúng kỳ vọng trực giác.

<a id="p12-clean-code-exercises"></a>
### P12 Clean Code - Đáp Án Bài Tập

- Refactor theo vòng nhỏ: đổi tên -> tách method -> chạy test -> lặp lại.
- Thay boolean parameter bằng enum hoặc method chuyên biệt.
- Loại bỏ duplicated logic bằng extraction hoặc strategy.

---

<a id="p13-async"></a>
## P13 Async - Đáp Án Câu Hỏi

1. Async không đồng nghĩa multithreading; async chủ yếu tránh block thread.
2. `Task.WhenAll` cho chạy song song I/O nhiều tác vụ.
3. `CancellationToken` là chuẩn hủy tác vụ hợp tác.
4. Exception async nằm trong `Task`, cần `await` để thấy rõ.
5. `HttpClient` nên tái sử dụng thay vì tạo mới liên tục.

<a id="p13-async-exercises"></a>
### P13 Async - Đáp Án Bài Tập

- `RetryAsync`: retry có giới hạn + delay + cancel support.
- Producer/consumer: có thể dùng `Channel<T>` cho luồng bất đồng bộ.
- Tránh `.Result`/`.Wait()` trong code async để giảm deadlock risk.

---

<a id="p14-evolution"></a>
## P14 C# Evolution - Đáp Án Câu Hỏi

1. Raw string literals giảm escape noise khi viết JSON/SQL/template.
2. `required` ép compile-time initialization cho members quan trọng.
3. `[SetsRequiredMembers]` báo compiler ctor này đã set đủ required members.
4. Primary constructor giảm boilerplate constructor injection.
5. Collection expressions `[]` giúp khởi tạo collection ngắn gọn, nhất quán.

<a id="p14-evolution-exercises"></a>
### P14 C# Evolution - Đáp Án Bài Tập

- Refactor dần từng class sang feature mới, tránh sửa hàng loạt một lần.
- Với primary constructor, nếu cần immutability thì map sang `readonly` field.
- Kiểm tra version SDK/TFM trước khi dùng tính năng mới.

---

<a id="p15-capstone"></a>
## P15 Capstone - Đáp Án Câu Hỏi

1. Thiết kế tốt phải giải thích được trade-off (không chỉ “code chạy”).
2. Test phải bảo vệ luồng nghiệp vụ quan trọng và các nhánh lỗi chính.
3. Query/report phải có số liệu kiểm chứng, không đoán hiệu năng.
4. Logging phải đủ để trace flow chính từ request đến dữ liệu.

<a id="p15-capstone-exercises"></a>
### P15 Capstone - Đáp Án Bài Tập

- Sequence diagram: thể hiện rõ app -> domain -> infra -> db -> response.
- Edge tests enrollment: duplicate, invalid state, closed course, null input.
- Performance report: có before/after và nguyên nhân cải thiện.

---

## Ghi Chú Chất Lượng

1. Đáp án ở đây là “gợi ý chuẩn”, không phải đáp án duy nhất.
2. Nếu bạn đưa được cách giải khác và chứng minh hợp lý bằng test/benchmark, cách đó vẫn đúng.
