# 🎯 Tổng Hợp Câu Hỏi & Đáp Án — Ôn Phỏng Vấn C# .NET
> Click vào **💡 Đáp án** để mở/đóng câu trả lời

---

## 📘 Part 1: Fundamentals

**1. CIL là gì? Tại sao C# cần biên dịch sang CIL trước?**
<details><summary>💡 Đáp án</summary>

**CIL (Common Intermediate Language)** là mã trung gian. C# → CIL → Machine Code (bởi JIT). CIL là **platform-independent** — cùng 1 file .dll chạy trên Windows, Linux, macOS. CLR trên mỗi OS sẽ JIT compile CIL thành machine code phù hợp.
```
C# source → [Roslyn Compiler] → CIL (.dll) → [JIT/CLR] → Machine Code
```
</details>

**2. JIT vs AOT compilation?**
<details><summary>💡 Đáp án</summary>

| | JIT (Just-In-Time) | AOT (Ahead-Of-Time) |
|---|---|---|
| **Khi nào?** | Runtime (method gọi lần đầu) | Build time |
| **Startup** | Chậm (compile lần đầu) | Nhanh (đã compile sẵn) |
| **Tối ưu** | Theo hardware thực tế | Generic |
| **Dùng cho** | Server apps | Mobile, IoT, serverless |
</details>

**3. .NET Framework vs .NET (Core)?**
<details><summary>💡 Đáp án</summary>

.NET Framework: Windows only, closed-source, version cuối 4.8 (ngừng phát triển).
.NET (Core → .NET 5+): cross-platform, open-source, hiệu suất cao hơn (Kestrel, async), .NET 8 LTS. **Tương lai của .NET.**
</details>

**4. CLR là gì?**
<details><summary>💡 Đáp án</summary>

**Common Language Runtime** — quản lý: JIT Compilation, **Garbage Collection** (tự thu hồi bộ nhớ), Type Safety, Exception Handling, Thread Management, Security.
</details>

**5. Value type vs Reference type?**
<details><summary>💡 Đáp án</summary>

Value types (int, struct, enum) → **Stack**, copy giá trị, so giá trị. Reference types (class, string, array) → **Heap** (reference trên Stack), copy tham chiếu, so tham chiếu.
```csharp
int a = 5; int b = a; b = 10;       // a vẫn = 5 (copy giá trị)
int[] x = {1}; int[] y = x; y[0]=9; // x[0] = 9! (cùng object)
```
</details>

**6. `var` vs `dynamic`?**
<details><summary>💡 Đáp án</summary>

`var` = **compile-time** inference (compiler biết kiểu, có IntelliSense, lỗi compile-time).
`dynamic` = **runtime** (compiler KHÔNG kiểm tra, crash runtime nếu sai). Luôn ưu tiên `var`.
</details>

**7. `const` vs `readonly`?**
<details><summary>💡 Đáp án</summary>

`const`: compile-time, tự static, chỉ primitive/string, **inline** vào code. `readonly`: runtime, gán trong constructor, mọi kiểu. VD: `const double PI = 3.14;` vs `readonly DateTime Start = DateTime.Now;`
</details>

**8. `out` vs `ref`?**
<details><summary>💡 Đáp án</summary>

`ref`: biến PHẢI khởi tạo trước, method có thể sửa. `out`: không cần khởi tạo, method BẮT BUỘC gán giá trị. VD: `int.TryParse("123", out int result)` — trả 2 giá trị (bool + int).
</details>

**9. Stack vs Heap?**
<details><summary>💡 Đáp án</summary>

Stack: value types, nhanh (LIFO), ~1MB/thread, tự dọn khi hết scope.
Heap: reference types, chậm hơn, GC quản lý, chia sẻ giữa threads, lớn (GB).
</details>

**10. Short-circuit evaluation?**
<details><summary>💡 Đáp án</summary>

`&&`: vế trái FALSE → bỏ qua vế phải. `||`: vế trái TRUE → bỏ qua vế phải.
Quan trọng: `if (list != null && list.Count > 0)` → tránh NullReferenceException!
</details>

---

## 📗 Part 2: OOP

**11. 4 pillars of OOP?**
<details><summary>💡 Đáp án</summary>

1. **Abstraction**: ẩn chi tiết phức tạp, chỉ expose interface đơn giản (VD: `Car.Start()` — không cần biết engine hoạt động thế nào)
2. **Encapsulation**: bảo vệ data nội bộ qua access modifiers (private field + public property)
3. **Inheritance**: tái sử dụng code từ base class (`Dog : Animal`)
4. **Polymorphism**: cùng method, behavior khác nhau (`animal.Speak()` → "Woof" hoặc "Meow" tùy actual type)
</details>

**12. Abstract class vs Interface?**
<details><summary>💡 Đáp án</summary>

| | Abstract Class | Interface |
|---|---|---|
| **Implementation** | Có thể có code | Chỉ "hợp đồng" (C# 8+ có default) |
| **Fields** | ✅ Có | ❌ Không |
| **Constructor** | ✅ Có | ❌ Không |
| **Kế thừa** | 1 class | Nhiều interfaces |
| **Dùng khi** | IS-A + shared code | CAN-DO behavior |

VD: `Animal` (abstract) — có shared `Eat()`. `ISwimmable` (interface) — Dog và Fish đều swim nhưng khác nhau.
</details>

**13. `virtual` vs `abstract` method?**
<details><summary>💡 Đáp án</summary>

`virtual`: CÓ implementation mặc định, derived CÓ THỂ override.
`abstract`: KHÔNG CÓ implementation, derived BẮT BUỘC override. Chỉ trong abstract class.
</details>

**14. Method hiding (`new`) vs overriding?**
<details><summary>💡 Đáp án</summary>

`override` (virtual/override) = **runtime polymorphism** — gọi method theo ACTUAL type.
`new` (hiding) = **compile-time** — gọi method theo DECLARED type.
```csharp
Animal a = new Dog();
a.Speak(); // override → "Woof!" (actual type)
           // new → "..." (declared type Animal)
```
Override = đúng 99%. Hiding = anti-pattern, tránh dùng.
</details>

**15. SOLID principles?**
<details><summary>💡 Đáp án</summary>

- **S — Single Responsibility**: mỗi class 1 lý do thay đổi. VD: `UserService` KHÔNG nên vừa validate, vừa send email
- **O — Open/Closed**: mở cho mở rộng (thêm class mới), đóng cho sửa đổi (không sửa code cũ)
- **L — Liskov Substitution**: derived class thay thế base mà không gây lỗi. VD: `Penguin : Bird` nhưng throw `CannotFly()` → vi phạm!
- **I — Interface Segregation**: interface nhỏ, chuyên biệt. Không ép client implement method không dùng
- **D — Dependency Inversion**: depend on abstractions (interface), không on implementations (concrete class)
</details>

**16. Dependency Injection là gì? 3 loại?**
<details><summary>💡 Đáp án</summary>

DI = truyền dependency từ **bên ngoài** vào (thay vì `new` bên trong).

1. **Constructor injection** (phổ biến nhất): `public TodoController(IRepository repo)`
2. **Property injection**: `public ILogger Logger { get; set; }`
3. **Method injection**: `public void Process(IValidator validator)`

Lợi ích: loose coupling, dễ test (mock), dễ swap implementation.
</details>

**17. `sealed` class dùng khi nào?**
<details><summary>💡 Đáp án</summary>

Ngăn kế thừa. Dùng khi: (1) Security — không cho override behavior, (2) Performance — JIT tối ưu hơn, (3) Design — `string` là sealed. Thể hiện ý đồ "class này KHÔNG nên được kế thừa".
</details>

**18. Extension method là gì?**
<details><summary>💡 Đáp án</summary>

Static method "giả vờ" là instance method: `public static bool IsNullOrEmpty(this string s)`. Gọi: `myString.IsNullOrEmpty()`. Không override được (compile-time binding). LINQ toàn bộ là extension methods trên `IEnumerable<T>`.
</details>

**19. Shallow copy vs Deep copy?**
<details><summary>💡 Đáp án</summary>

**Shallow**: copy reference → 2 objects chia sẻ nested objects (thay đổi 1 → ảnh hưởng cả 2).
**Deep**: copy TẤT CẢ nested objects → hoàn toàn độc lập. `MemberwiseClone()` = shallow. Deep = serialize/deserialize hoặc manual copy.
</details>

**20. `is` vs `as` operators?**
<details><summary>💡 Đáp án</summary>

`is`: kiểm tra type + pattern match: `if (obj is Dog d) { d.Bark(); }` → true/false + biến mới.
`as`: cast, trả null nếu fail: `Dog d = obj as Dog;` → chỉ reference types. Ưu tiên `is`.
</details>

---

## 📕 Part 3: Exceptions & Error Handling

**21. `throw` vs `throw ex` — khác nhau?**
<details><summary>💡 Đáp án</summary>

`throw;` giữ nguyên stack trace gốc. `throw ex;` reset stack trace → mất thông tin bug ở đâu. **Luôn dùng `throw;`** trong catch block.
</details>

**22. `finally` block luôn chạy?**
<details><summary>💡 Đáp án</summary>

Gần như luôn — kể cả khi có `return` trong try/catch. Ngoại lệ: `Environment.FailFast()`, power off, stack overflow. Dùng cho cleanup: đóng file, connection, release resources.
</details>

**23. Custom exception — khi nào tạo?**
<details><summary>💡 Đáp án</summary>

Khi built-in exceptions không diễn tả đủ ý nghĩa. VD: `InsufficientFundsException`, `OrderNotFoundException`. Luôn kế thừa `Exception`, thêm properties hữu ích, implement 3 constructors chuẩn.
</details>

---

## 📙 Part 4: Generics & Delegates

**24. Generics là gì? Tại sao cần?**
<details><summary>💡 Đáp án</summary>

Generics = code hoạt động với **nhiều kiểu dữ liệu** mà vẫn type-safe. `List<T>` thay vì `ArrayList` (boxing/unboxing). Lợi ích: type safety (compile-time check), performance (no boxing), reusability.
</details>

**25. `Func<T>` vs `Action<T>` vs `Predicate<T>`?**
<details><summary>💡 Đáp án</summary>

| Delegate | Return | VD |
|---|---|---|
| `Action<T>` | void | `Action<string> log = s => Console.WriteLine(s);` |
| `Func<T, TResult>` | TResult | `Func<int, int, int> add = (a,b) => a+b;` |
| `Predicate<T>` | bool | `Predicate<int> isEven = n => n%2==0;` |
</details>

**26. Covariance vs Contravariance?**
<details><summary>💡 Đáp án</summary>

**Covariance** (`out T`): `IEnumerable<Dog>` gán cho `IEnumerable<Animal>` ✅ (con → cha, output).
**Contravariance** (`in T`): `Action<Animal>` gán cho `Action<Dog>` ✅ (cha → con, input).
Nhớ: **out = output = con→cha, in = input = cha→con**.
</details>

---

## 📒 Part 5: LINQ

**27. LINQ là gì? Query syntax vs Method syntax?**
<details><summary>💡 Đáp án</summary>

LINQ = **Language Integrated Query** — truy vấn dữ liệu bằng C# syntax.
Query syntax: `from x in list where x > 5 select x` (giống SQL).
Method syntax: `list.Where(x => x > 5)` (dùng extension methods).
Method syntax phổ biến hơn vì flexible, chain được.
</details>

**28. Deferred execution là gì?**
<details><summary>💡 Đáp án</summary>

LINQ query KHÔNG chạy ngay khi khai báo. Chỉ chạy khi **enumerate** (foreach, ToList, Count...).
```csharp
var query = list.Where(x => x > 5); // CHƯA chạy!
var result = query.ToList();         // BÂY GIỜ mới chạy
```
Lợi ích: compose queries, lazy evaluation. Rủi ro: query chạy nhiều lần nếu enumerate nhiều lần.
</details>

**29. `IEnumerable<T>` vs `IQueryable<T>`?**
<details><summary>💡 Đáp án</summary>

| | IEnumerable\<T> | IQueryable\<T> |
|---|---|---|
| **Xử lý** | In-memory (C#) | Translated (SQL, API...) |
| **Filter** | Client-side | Server-side ✅ |
| **Dùng cho** | Collections, files | EF Core, databases |

⚠️ `IEnumerable` + `.Where()` = load TẤT CẢ rồi filter → chậm!
`IQueryable` + `.Where()` = tạo SQL WHERE → nhanh!
</details>

---

## 📓 Part 6: .NET Internals & Memory

**30. Garbage Collection hoạt động thế nào?**
<details><summary>💡 Đáp án</summary>

GC tự động thu hồi objects trên Heap không còn ai reference. 3 generations:

- **Gen 0**: objects mới (collect thường xuyên, nhanh)
- **Gen 1**: objects sống qua Gen 0 (buffer)
- **Gen 2**: objects sống lâu (collect hiếm, chậm)

GC KHÔNG deterministic — không biết chính xác khi nào chạy. Dùng `IDisposable` + `using` cho unmanaged resources.
</details>

**31. `IDisposable` và `using` statement?**
<details><summary>💡 Đáp án</summary>

`IDisposable` = pattern giải phóng **unmanaged resources** (file handles, DB connections, network).
`using` = tự gọi `Dispose()` khi ra khỏi scope:
```csharp
using var conn = new SqlConnection(connStr); // Tự Dispose() khi hết scope
```
</details>

**32. Boxing vs Unboxing?**
<details><summary>💡 Đáp án</summary>

**Boxing**: value type → object (stack → heap, tạo wrapper). `object o = 42;`
**Unboxing**: object → value type (heap → stack, unwrap). `int n = (int)o;`
⚠️ Tốn performance! Generics (`List<int>` thay `ArrayList`) tránh boxing.
</details>

---

## 📔 Part 7: Async/Await

**33. `async`/`await` hoạt động thế nào?**
<details><summary>💡 Đáp án</summary>

`async` đánh dấu method có thể chạy bất đồng bộ. `await` "tạm dừng" method, trả control cho caller, tiếp tục khi Task hoàn thành. **KHÔNG tạo thread mới** — dùng I/O completion ports.
```csharp
async Task<string> GetDataAsync()
{
    var data = await httpClient.GetStringAsync(url); // Thread FREE trong lúc chờ!
    return data; // Tiếp tục SAU KHI có kết quả
}
```
</details>

**34. `Task` vs `Task<T>` vs `void`?**
<details><summary>💡 Đáp án</summary>

| Return type | Khi nào | Await được? |
|---|---|---|
| `Task<T>` | Trả giá trị | ✅ |
| `Task` | Không trả giá trị | ✅ |
| `async void` | ❌ Event handlers only | ❌ KHÔNG! |

⚠️ `async void` → exceptions KHÔNG bắt được → crash app. Luôn return `Task`.
</details>

**35. Deadlock với async — tránh thế nào?**
<details><summary>💡 Đáp án</summary>

Deadlock xảy ra khi `.Result` hoặc `.Wait()` block synchronization context.
```csharp
// ❌ DEADLOCK
var result = GetDataAsync().Result;  // Block thread → deadlock!

// ✅ ĐÚNG
var result = await GetDataAsync();   // Không block
```
Quy tắc: **async all the way** — không mix sync/async. `.ConfigureAwait(false)` trong library code.
</details>

---

## 📘 Part 8: Clean Code & Design Patterns

**36. SOLID chi tiết — cho ví dụ LSP, ISP, DIP?**
<details><summary>💡 Đáp án</summary>

**LSP**: `FeedAnimal(Animal a) { a.Eat(); }` — mọi derived class phải hoạt động đúng. `Penguin.Fly()` throw exception → vi phạm!

**ISP**: Không ép client implement method không dùng. Tách `IDocumentService` (5 methods) → `IDocumentReader`, `IDocumentWriter`, `IDocumentValidator`.

**DIP**: `CustomerService` phụ thuộc `ICustomerRepository` (interface), KHÔNG phụ thuộc `SqlCustomerRepository` (concrete). Đổi SQL → MongoDB = chỉ swap implementation.
</details>

**37. Singleton pattern?**
<details><summary>💡 Đáp án</summary>

Đảm bảo chỉ **1 instance** trong toàn app. Private constructor, static property.
Thread-safe: `private static readonly Lazy<T> _lazy = new(() => new T()); public static T Instance => _lazy.Value;`
Dùng cho: Logger, DB connection pool, Configuration.
</details>

**38. Factory pattern?**
<details><summary>💡 Đáp án</summary>

Tập trung logic tạo objects. Client không cần biết concrete class.
```csharp
INotification n = NotificationFactory.Create("email"); // Trả EmailNotification
INotification n = NotificationFactory.Create("sms");   // Trả SmsNotification
```
Lợi ích: OCP (thêm loại mới không sửa client code), encapsulate creation logic.
</details>

---

## 🗄️ Part 9: Database

**39. Normalization — 1NF, 2NF, 3NF?**
<details><summary>💡 Đáp án</summary>

**1NF**: Mỗi cell chỉ 1 giá trị (atomic). Không có `skills = "C#, Java"`.
**2NF**: 1NF + không partial dependency. Mọi non-key column phụ thuộc TOÀN BỘ primary key.
**3NF**: 2NF + không transitive dependency. Non-key column CHỈ phụ thuộc primary key, không qua column khác.
</details>

**40. Index là gì? Khi nào KHÔNG dùng?**
<details><summary>💡 Đáp án</summary>

Index = B-Tree data structure giúp tìm kiếm O(log n) thay vì O(n).
**Tạo cho**: WHERE/JOIN/ORDER BY columns, Foreign Keys.
**KHÔNG tạo cho**: bảng nhỏ (<1000 rows), cột ít distinct values (boolean), cột write-heavy.
Trade-off: **nhanh reads, chậm writes** (vì phải cập nhật index).
</details>

**41. Transaction và ACID?**
<details><summary>💡 Đáp án</summary>

Transaction = nhóm operations thực hiện **all-or-nothing**.

- **A — Atomicity**: tất cả hoặc không gì cả
- **C — Consistency**: DB luôn trong trạng thái hợp lệ
- **I — Isolation**: transactions không ảnh hưởng lẫn nhau
- **D — Durability**: sau COMMIT, data KHÔNG mất dù crash

VD: chuyển tiền: trừ A + cộng B = 1 transaction. Fail giữa chừng → ROLLBACK cả hai.
</details>

**42. SQL JOIN types?**
<details><summary>💡 Đáp án</summary>

| JOIN | Kết quả |
|---|---|
| **INNER** | Chỉ rows match CẢ HAI bảng |
| **LEFT** | TẤT CẢ rows bảng trái + match bảng phải (NULL nếu không match) |
| **RIGHT** | Ngược LEFT |
| **FULL** | TẤT CẢ rows cả hai bảng |
| **CROSS** | Mọi tổ hợp (cartesian product) |
</details>

---

## 🌐 Part 10: ASP.NET Core

**43. Middleware pipeline là gì?**
<details><summary>💡 Đáp án</summary>

Mỗi HTTP request đi qua chuỗi middleware THEO THỨ TỰ. Mỗi middleware có thể: xử lý request, gọi next, hoặc short-circuit.
Thứ tự quan trọng: `UseAuthentication()` TRƯỚC `UseAuthorization()` (phải biết user là AI trước khi kiểm tra QUYỀN).
</details>

**44. Dependency Injection trong ASP.NET Core?**
<details><summary>💡 Đáp án</summary>

DI container built-in. Đăng ký service: `builder.Services.AddScoped<IRepository, SqlRepository>();`
3 lifetimes: **Transient** (mới mỗi lần inject), **Scoped** (1 per request), **Singleton** (1 per app).
Controller nhận qua constructor: `public TodoController(IRepository repo)` — framework tự inject.
</details>

**45. MVC pattern trong ASP.NET?**
<details><summary>💡 Đáp án</summary>

**Model**: data + business rules (TodoItem, Category).
**View**: UI render (Razor .cshtml).
**Controller**: nhận request, xử lý logic, chọn View trả về.
Flow: Browser → Routing → Controller → Model → View → HTML → Browser.
</details>

**46. Entity Framework Core — Code-First là gì?**
<details><summary>💡 Đáp án</summary>

Viết C# classes (Models) TRƯỚC → EF Core tạo DB schema từ đó qua Migrations.
```
Write Models → dotnet ef migrations add → dotnet ef database update → DB created!
```
Ngược lại Database-First: DB đã tồn tại → scaffold C# classes từ DB.
Code-First tốt cho: greenfield projects, version control schema.
</details>
