# 📕 Phần 3: Exceptions & Error Handling

> **Nội dung**: Exception handling, try-catch-finally, custom exceptions, best practices  
> **Thời lượng ước tính**: 4–5 giờ

---

## 🎯 Mục Tiêu

- Hiểu Exception là gì, Stack Trace đọc ra sao
- Thành thạo try-catch-finally
- Biết khi nào throw, khi nào catch
- Tạo Custom Exception
- Áp dụng exception handling thông minh

---

## 1. Exception Object

### 1.1 Exception là gì?

Exception (ngoại lệ) là **sự kiện xảy ra khi chương trình gặp lỗi runtime**. Khi exception xảy ra, chương trình sẽ dừng ngay lập tức trừ khi được **xử lý** (caught).

```csharp
int[] arr = { 1, 2, 3 };
Console.WriteLine(arr[10]);  // ❌ IndexOutOfRangeException!

int result = 10 / 0;         // ❌ DivideByZeroException!

string s = null;
Console.WriteLine(s.Length);  // ❌ NullReferenceException!
```

### 1.2 Stack Trace

Stack trace cho biết **chuỗi các method call** dẫn đến exception:

```
System.IndexOutOfRangeException: Index was outside the bounds of the array.
   at Program.ProcessData(Int32[] data) in Program.cs:line 15
   at Program.Main(String[] args) in Program.cs:line 8
```

**Đọc từ DƯỚI lên**: Main gọi ProcessData, lỗi ở dòng 15 trong ProcessData.

---

## 2. Try-Catch-Finally

```csharp
try
{
    // Code có thể gây lỗi
    int result = int.Parse(Console.ReadLine());
    Console.WriteLine($"Bạn nhập: {result}");
}
catch (FormatException ex)
{
    // Xử lý khi input không phải số
    Console.WriteLine($"Lỗi format: {ex.Message}");
}
catch (OverflowException ex)
{
    // Xử lý khi số quá lớn
    Console.WriteLine($"Số quá lớn: {ex.Message}");
}
catch (Exception ex)
{
    // Catch-all: bắt MỌI exception (đặt cuối cùng)
    Console.WriteLine($"Lỗi không xác định: {ex.Message}");
}
finally
{
    // LUÔN LUÔN chạy, dù có exception hay không
    // Dùng cho cleanup: đóng file, đóng kết nối DB, giải phóng tài nguyên
    Console.WriteLine("Cleanup hoàn tất");
}
```

---

## 3. Throwing Exceptions

```csharp
// throw: phát sinh exception thủ công
static void SetAge(int age)
{
    if (age < 0)
        throw new ArgumentOutOfRangeException(nameof(age), "Tuổi không thể âm!");
    
    if (age > 150)
        throw new ArgumentOutOfRangeException(nameof(age), "Tuổi không hợp lệ!");
}

// Rethrow: ném lại exception trong catch block
try
{
    DoSomething();
}
catch (Exception ex)
{
    LogError(ex);
    throw;        // ✅ Giữ nguyên stack trace
    // throw ex;  // ❌ Mất stack trace gốc!
}
```

---

## 4. Built-in Exception Types

| Exception | Khi nào xảy ra |
|-----------|----------------|
| `NullReferenceException` | Truy cập member của null |
| `IndexOutOfRangeException` | Index mảng/list ngoài phạm vi |
| `ArgumentNullException` | Truyền null cho parameter không nhận null |
| `ArgumentOutOfRangeException` | Giá trị ngoài phạm vi cho phép |
| `ArgumentException` | Argument không hợp lệ (chung) |
| `InvalidOperationException` | Gọi method khi object ở trạng thái không hợp lệ |
| `FormatException` | Chuỗi không đúng định dạng khi parse |
| `OverflowException` | Kết quả số học vượt phạm vi |
| `StackOverflowException` | Đệ quy vô hạn → hết stack |
| `FileNotFoundException` | File không tồn tại |
| `IOException` | Lỗi I/O chung |
| `NotImplementedException` | Code chưa được implement |
| `NotSupportedException` | Operation không được hỗ trợ |

---

## 5. Custom Exceptions

```csharp
// Tạo exception riêng khi built-in không đủ mô tả
class InsufficientFundsException : Exception
{
    public decimal Balance { get; }
    public decimal Amount { get; }
    
    public InsufficientFundsException(decimal balance, decimal amount)
        : base($"Không đủ tiền. Số dư: {balance:C}, Cần: {amount:C}")
    {
        Balance = balance;
        Amount = amount;
    }
}

// Sử dụng
class BankAccount
{
    private decimal _balance;
    
    public void Withdraw(decimal amount)
    {
        if (amount > _balance)
            throw new InsufficientFundsException(_balance, amount);
        _balance -= amount;
    }
}
```

---

## 6. Smart Usage of Exceptions

### 6.1 Khi nào THROW

```csharp
// ✅ Throw khi: Điều kiện tiên quyết bị vi phạm
public void ProcessOrder(Order order)
{
    if (order is null) throw new ArgumentNullException(nameof(order));
    if (order.Items.Count == 0) throw new InvalidOperationException("Order must have items");
}

// ❌ ĐỪNG dùng exception cho control flow bình thường
// Bad:
try { int.Parse(input); } catch { /* không phải số */ }
// Good:
if (int.TryParse(input, out int result)) { /* là số */ }
```

### 6.2 Khi nào CATCH

```csharp
// ✅ Catch khi bạn có thể XỬ LÝ hoặc KHÔI PHỤC
try
{
    var data = File.ReadAllText("config.json");
}
catch (FileNotFoundException)
{
    // Dùng giá trị mặc định thay thế
    CreateDefaultConfig();
}

// ❌ ĐỪNG catch rồi để trống (swallow exception)
// try { ... } catch { }  // Lỗi bị nuốt im lặng!

// ❌ ĐỪNG catch Exception chung chung ở mọi nơi
// Chỉ catch Exception ở global level (top of call stack)
```

### 6.3 Exception Filters

```csharp
try
{
    await httpClient.GetAsync(url);
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    Console.WriteLine("Trang không tồn tại");
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.ServiceUnavailable)
{
    Console.WriteLine("Server đang bận, thử lại sau");
}
```

---

## 7. Recursive Methods & StackOverflow

```csharp
// Đệ quy: method gọi chính nó
static int Factorial(int n)
{
    if (n <= 1) return 1;        // Base case (điều kiện dừng)
    return n * Factorial(n - 1);  // Recursive case
}

// ⚠️ StackOverflowException nếu THIẾU base case
static void InfiniteRecursion()
{
    InfiniteRecursion();  // ❌ Không bao giờ dừng!
}
```

---

## ❓ Câu Hỏi Kiểm Tra

1. Exception là gì? Khác error thế nào?
2. Stack trace đọc từ hướng nào? Tại sao?
3. `try-catch-finally` — `finally` có LUÔN chạy không? Kể cả khi có `return` trong `try`?
4. `throw` vs `throw ex` — khác nhau thế nào? Cái nào nên dùng?
5. Tại sao KHÔNG nên `catch (Exception ex) { }` (catch rồi bỏ trống)?
6. Khi nào nên throw exception, khi nào dùng `TryParse`/return code?
7. Custom exception nên kế thừa từ class nào?
8. Exception filters (`when`) có gì hay hơn dùng `if` trong catch?
9. `StackOverflowException` có catch được không? Tại sao?
10. `using` statement liên quan gì đến exception handling?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between `throw` and `throw ex`?**
> A: `throw` giữ nguyên stack trace gốc — biết lỗi bắt đầu từ đâu. `throw ex` reset stack trace — mất thông tin debug quan trọng. LUÔN dùng `throw;` trong catch block.

> **Q: When should you create custom exceptions?**
> A: Khi cần truyền domain-specific information (ví dụ: Balance, Amount). Khi cần phân biệt error types cho caller. Khi built-in exceptions không mô tả đúng lỗi. Kế thừa từ `Exception`.

> **Q: What is the difference between `Exception` and `ApplicationException`?**
> A: `ApplicationException` ban đầu thiết kế để phân biệt user code vs framework. Nhưng thực tế ít ai dùng. Microsoft khuyên kế thừa trực tiếp từ `Exception`.

> **Q: Explain exception filters (`when` keyword).**
> A: `catch (HttpException ex) when (ex.StatusCode == 404)` — chỉ catch KHI điều kiện đúng. Không cuộn (unwind) stack, hiệu quả hơn catch-rethrow. Dùng khi muốn catch exception có điều kiện.

> **Q: Does `finally` always execute?**
> A: Gần như luôn — kể cả khi có `return` trong try. Ngoại lệ: `Environment.FailFast()`, unhandled StackOverflowException, hoặc process bị kill.

> **Q: Should you use exceptions for control flow?**
> A: KHÔNG. Exceptions tốn hiệu suất (stack unwinding). Dùng pattern TryXxx (TryParse, TryGetValue) cho expected failures. Chỉ throw cho truly exceptional situations.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Viết method `SafeDivide(int a, int b)` trả về `int?`, return null nếu chia cho 0.

**BT2**: Tạo custom exception `InsufficientFundsException` cho BankAccount. Bao gồm properties: Amount, Balance, Shortfall.

**BT3**: Viết program đọc file CSV, parse mỗi dòng thành object. Log tất cả dòng lỗi (với số dòng) mà KHÔNG dừng chương trình.

**BT4**: Implement retry pattern: method `RetryAsync(int maxRetries, Func<Task> action)` thử lại N lần nếu gặp exception.

---

## 📚 Bổ Sung: Advanced Exception Patterns

### ExceptionDispatchInfo — Rethrow giữ Stack Trace

```csharp
using System.Runtime.ExceptionServices;

// Vấn đề: throw ex; MẤT stack trace gốc
// Giải pháp: ExceptionDispatchInfo giữ nguyên stack trace

ExceptionDispatchInfo? capturedException = null;

try
{
    await SomeAsyncOperation();
}
catch (Exception ex)
{
    capturedException = ExceptionDispatchInfo.Capture(ex);
}

// Rethrow ở nơi khác, giữ nguyên stack trace
if (capturedException != null)
    capturedException.Throw();  // Stack trace gốc được giữ!
```

### Global Exception Handling

```csharp
// 1. Console apps
AppDomain.CurrentDomain.UnhandledException += (sender, e) =>
{
    var ex = (Exception)e.ExceptionObject;
    Console.WriteLine($"Fatal: {ex.Message}");
    // Log, cleanup, etc.
};

// 2. ASP.NET Core — Middleware
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var exception = context.Features.Get<IExceptionHandlerFeature>()?.Error;
        await context.Response.WriteAsJsonAsync(new { error = exception?.Message });
    });
});

// 3. Task unobserved exceptions
TaskScheduler.UnobservedTaskException += (sender, e) =>
{
    Console.WriteLine($"Unobserved: {e.Exception.Message}");
    e.SetObserved();  // Tránh crash app
};
```

---

## ❓ Câu Hỏi Kiểm Tra Bổ Sung (Nâng Cao)

1. `try-catch-finally` — `finally` luôn chạy? Kể cả khi có `return`?
2. Tại sao KHÔNG nên `catch (Exception ex)` mà bỏ trống body?
3. Custom exception nên kế thừa từ class nào?
4. `throw` vs `throw ex` — khác nhau thế nào?
5. Khi nào nên throw exception, khi nào trả về error code?

---

## 💼 Câu Hỏi Phỏng Vấn Bổ Sung (Nâng Cao)

> **Q: What is the difference between `throw` and `throw ex`?**
> A: `throw` giữ nguyên stack trace gốc — biết lỗi bắt đầu từ đâu. `throw ex` reset stack trace — mất thông tin debug quan trọng. LUÔN dùng `throw;` trong catch.

> **Q: When should you create custom exceptions?**
> A: Khi cần truyền domain-specific information. Khi cần phân biệt error types cho caller. Khi built-in exceptions không mô tả đúng lỗi. Kế thừa từ `Exception` hoặc specialized exception.

> **Q: What is the difference between `Exception` and `ApplicationException`?**
> A: `ApplicationException` ban đầu design để phân biệt user code vs framework exceptions. Nhưng thực tế ít ai dùng. Microsoft khuyên kế thừa trực tiếp từ `Exception`.

> **Q: Explain exception filters (`when` keyword).**
> A: `catch (HttpException ex) when (ex.StatusCode == 404)` — chỉ catch exception KHI điều kiện đúng. Không cuộn stack trace, hiệu quả hơn catch-rethrow.

---

## 🏋️ Bài Tập Bổ Sung (Nâng Cao)

**BT1**: Viết method `SafeDivide(int a, int b)` trả về `int?`, return null nếu chia cho 0 (thay vì throw).

**BT2**: Tạo custom exception `InsufficientFundsException` cho BankAccount. Include Amount, Balance, Shortfall.

**BT3**: Viết program đọc file CSV, parse mỗi dòng thành object. Log tất cả dòng lỗi (với số dòng) mà KHÔNG dừng chương trình.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p03-exceptions](./99-answer-key-csharp.md#p03-exceptions)
- Bài tập thực hành: [99-answer-key-csharp.md#p03-exceptions-exercises](./99-answer-key-csharp.md#p03-exceptions-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p03-exceptions-deep](./97-csharp-theory-deep-dive.md#p03-exceptions-deep)

