# 📔 Phần 6: .NET Under the Hood

> **Nội dung**: CIL, CLR, Stack vs Heap, GC, Boxing, IDisposable, Span<T>  
> **Thời lượng ước tính**: 5–6 giờ

---

## 🎯 Mục Tiêu

- Hiểu kiến trúc .NET (CIL, CLR)
- Phân biệt Stack vs Heap, Value types vs Reference types
- Hiểu Boxing/Unboxing và chi phí hiệu suất
- Nắm vững Garbage Collector
- Memory leaks và cách phòng tránh
- IDisposable, Finalizer, và `using` statement

---

## 1. .NET Architecture

```
┌─────────────────────────────────────────┐
│            Your C# Code (.cs)           │
├─────────────────────────────────────────┤
│        C# Compiler (Roslyn)             │
│     Biên dịch → Common IL (CIL)        │
├─────────────────────────────────────────┤
│     .NET Assembly (.dll / .exe)         │
│     Chứa CIL + Metadata                │
├─────────────────────────────────────────┤
│     Common Language Runtime (CLR)       │
│  ┌─────────┬───────┬──────────────┐     │
│  │   JIT   │  GC   │ Type Safety  │     │
│  │Compiler │       │ Verification │     │
│  └─────────┴───────┴──────────────┘     │
├─────────────────────────────────────────┤
│          Machine Code (CPU)             │
└─────────────────────────────────────────┘
```

- **CIL** (Common Intermediate Language): Mã trung gian, giống bytecode của Java
- **CLR** (Common Language Runtime): Máy ảo chạy CIL, quản lý bộ nhớ
- **JIT** (Just-In-Time): Biên dịch CIL → Machine code lúc chạy

---

## 2. Stack vs Heap

```
┌─────────────────┐     ┌─────────────────────┐
│     STACK        │     │        HEAP          │
│  (Ngăn xếp)     │     │    (Đống nhớ)        │
├─────────────────┤     ├─────────────────────┤
│ - Tự động       │     │ - GC quản lý         │
│ - LIFO (vào sau │     │ - Phân bổ & giải     │
│   ra trước)     │     │   phóng linh hoạt    │
│ - Rất NHANH     │     │ - Chậm hơn Stack     │
│ - Kích thước nhỏ│     │ - Kích thước lớn     │
│ - Value types   │     │ - Reference types     │
│ - Local vars    │     │ - Objects, arrays     │
│ - Method params │     │ - Strings             │
└─────────────────┘     └─────────────────────┘
```

### 2.1 Value Types (lưu trên Stack)

```csharp
int a = 42;       // Stack: [a = 42]
int b = a;        // Stack: [a = 42, b = 42]  ← COPY giá trị
b = 100;         // Stack: [a = 42, b = 100] ← a KHÔNG đổi

// Value types: int, double, bool, char, struct, enum, decimal
```

### 2.2 Reference Types (lưu trên Heap)

```csharp
int[] arr1 = { 1, 2, 3 };  // Stack: [arr1 → ●]  Heap: [1, 2, 3]
int[] arr2 = arr1;           // Stack: [arr2 → ●]  cùng trỏ đến Heap!
arr2[0] = 999;               // Heap: [999, 2, 3]

Console.WriteLine(arr1[0]);  // 999! ← arr1 cũng bị ảnh hưởng!

// Reference types: class, string, array, interface, delegate, object
```

### 2.3 "ref" Keyword

```csharp
// Mặc định: value types truyền BY VALUE (copy)
static void Double(int x)
{
    x *= 2;  // Chỉ thay đổi bản copy
}

int num = 5;
Double(num);
Console.WriteLine(num);  // Vẫn 5!

// ref: truyền BY REFERENCE (trỏ đến biến gốc)
static void DoubleRef(ref int x)
{
    x *= 2;  // Thay đổi biến GỐC
}

DoubleRef(ref num);
Console.WriteLine(num);  // 10!
```

---

## 3. Boxing & Unboxing

```csharp
// Boxing: Value type → Object (Stack → Heap)
int number = 42;
object boxed = number;  // ⚠️ Boxing! Copy value lên Heap, bọc trong object

// Unboxing: Object → Value type (Heap → Stack)
int unboxed = (int)boxed;  // ⚠️ Unboxing! Copy từ Heap về Stack

// ⚠️ CHI PHÍ HIỆU SUẤT:
// Boxing:   ~20x chậm hơn gán bình thường
// Unboxing: ~4x chậm hơn gán bình thường
// + Tạo rác trên Heap → GC phải dọn

// Boxing ẩn (hidden boxing) - NÊN TRÁNH:
ArrayList list = new ArrayList();  // Chứa object
list.Add(42);     // ⚠️ Boxing! int → object
list.Add(3.14);   // ⚠️ Boxing! double → object

// ✅ Dùng generic thay thế:
List<int> genericList = new List<int>();
genericList.Add(42);  // KHÔNG boxing!
```

---

## 4. Garbage Collector (GC)

### 4.1 Mark-and-Sweep Algorithm

```
1. MARK:  GC đi từ "GC Roots" (static vars, local vars, ...)
          và đánh dấu tất cả objects ĐANG ĐƯỢC THAM CHIẾU
          
2. SWEEP: Mọi object KHÔNG được đánh dấu → giải phóng bộ nhớ

3. COMPACT: Dồn các object còn lại lại gần nhau 
            (giải quyết memory fragmentation)
```

### 4.2 Generations

```
┌───────────────┬──────────────┬──────────────┐
│ Generation 0  │ Generation 1 │ Generation 2 │
│ (Mới nhất)    │ (Trung bình) │ (Lâu nhất)   │
│ GC thường xuyên│GC ít hơn    │ GC hiếm khi  │
│ Objects mới   │ Sống sót G0  │ Sống sót G1  │
└───────────────┴──────────────┴──────────────┘

Giả thuyết: Đa số object có tuổi thọ NGẮN
→ GC tập trung dọn Gen 0 (nhanh, hiệu quả)
→ Object sống sót nhiều lần GC → thăng cấp Generation
```

### 4.3 Memory Leaks trong C#

```csharp
// Dù có GC, C# VẪN CÓ THỂ bị memory leak!

// 1. Event handlers không unsubscribe
button.Click += OnClick;  // ← Giữ reference → GC không dọn được

// 2. Static collections giữ reference
static List<object> cache = new();  // Không bao giờ bị GC dọn!

// 3. Không dispose unmanaged resources
var stream = new FileStream("file.txt", FileMode.Open);
// Quên Close/Dispose → File bị lock!
```

---

## 5. IDisposable & using

### 5.1 Vấn đề: Unmanaged Resources

GC chỉ quản lý **managed memory** (Heap). Các tài nguyên khác (file handles, DB connections, network sockets) cần **giải phóng thủ công**.

### 5.2 IDisposable Pattern

```csharp
class DatabaseConnection : IDisposable
{
    private SqlConnection _connection;
    private bool _disposed = false;
    
    public DatabaseConnection(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
    }
    
    public void Dispose()
    {
        if (!_disposed)
        {
            _connection?.Close();
            _connection?.Dispose();
            _disposed = true;
        }
    }
}
```

### 5.3 using Statement (TỰ ĐỘNG Dispose)

```csharp
// using statement: gọi Dispose() tự động khi ra khỏi block
using (StreamWriter writer = new StreamWriter("output.txt"))
{
    writer.WriteLine("Hello!");
}  // ← Dispose() được gọi tự động ở đây

// using declaration (C# 8+): ngắn gọn hơn
using StreamReader reader = new StreamReader("input.txt");
string content = reader.ReadToEnd();
// ← Dispose() gọi khi ra khỏi scope (cuối method)
```

### 5.4 Finalizer (Destructor)

```csharp
class ResourceHolder
{
    ~ResourceHolder()  // Finalizer: GC gọi TRƯỚC khi dọn object
    {
        // Cleanup code
        // ⚠️ KHÔNG nên dùng trừ khi thực sự cần
        // vì: chậm, không đảm bảo thời điểm, tốn tài nguyên
    }
}
```

---

## 6. Reading/Writing Files

```csharp
// Đọc file
string content = File.ReadAllText("data.txt");
string[] lines = File.ReadAllLines("data.txt");

// Ghi file
File.WriteAllText("output.txt", "Hello World");
File.WriteAllLines("output.txt", new[] { "Line 1", "Line 2" });

// StreamWriter / StreamReader (cho file lớn)
using var writer = new StreamWriter("log.txt", append: true);
writer.WriteLine($"[{DateTime.Now}] Event occurred");

using var reader = new StreamReader("data.csv");
string line;
while ((line = reader.ReadLine()) != null)
{
    Console.WriteLine(line);
}
```

---

## 7. CSV Files

```csharp
// Đọc CSV đơn giản
string[] lines = File.ReadAllLines("data.csv");

// Bỏ qua header (dòng đầu)
for (int i = 1; i < lines.Length; i++)
{
    string[] columns = lines[i].Split(',');
    string name = columns[0];
    int age = int.Parse(columns[1]);
    Console.WriteLine($"{name} is {age} years old");
}
```

---

## ❓ Câu Hỏi Kiểm Tra

1. CIL là gì? JIT là gì? Mối quan hệ giữa chúng?
2. Stack vs Heap — value types lưu ở đâu? Reference types?
3. Boxing/Unboxing là gì? Tại sao gây mất hiệu suất?
4. GC có mấy generations? Tại sao phân chia thế?
5. Trong C# có Memory Leak không? Cho 3 ví dụ.
6. `using` statement làm gì bên trong? Liên quan interface nào?
7. `ref` vs `out` vs `in` parameter — khác nhau thế nào?
8. `File.ReadAllText` vs `StreamReader` — khi nào dùng cái nào?
9. Finalizer là gì? Tại sao KHÔNG nên dùng?
10. `using declaration` (C# 8) khác `using statement` thế nào?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between Stack and Heap?**
> A: **Stack**: LIFO, tự động allocate/deallocate, nhanh, chứa value types + references. **Heap**: GC quản lý, chậm hơn, chứa objects/arrays/strings. Local `int x = 5` trên Stack, `new Person()` trên Heap.

> **Q: Explain Boxing and Unboxing. Why is it bad?**
> A: **Boxing**: value type → object (Stack → Heap, bọc trong object). **Unboxing**: ngược lại. Chi phí: allocate heap memory, copy data, GC pressure. Dùng `List<int>` thay `ArrayList` để tránh.

> **Q: How does Garbage Collection work in .NET?**
> A: Mark-and-Sweep: đánh dấu objects reachable từ GC roots, giải phóng unreachable. 3 generations (0=most frequent, 2=least). Ephemeral segment cho Gen 0/1. Large Object Heap cho objects >85KB.

> **Q: What is the Dispose pattern? When should you implement it?**
> A: Implement `IDisposable` khi class sở hữu unmanaged resources (file handles, DB connections, sockets). `using` statement đảm bảo Dispose() gọi kể cả khi exception. Pattern: guard `_disposed` flag.

> **Q: What is the difference between `ref`, `out`, and `in` parameters?**
> A: `ref`: truyền reference, PHẢI initialize trước. `out`: truyền reference, KHÔNG cần initialize, PHẢI gán trong method. `in`: readonly reference, không thể sửa. `ref` cho read-write, `out` cho output, `in` cho performance (avoid copy).

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Viết chương trình đếm Boxing/Unboxing: tạo vòng lặp 1 triệu additions dùng `ArrayList` vs `List<int>`, đo thời gian.

**BT2**: Implement `IDisposable` cho class `TempFileManager` — tạo file tạm, auto-delete khi Dispose.

**BT3**: Viết CSV parser: đọc file CSV, trả về `List<Dictionary<string, string>>` (header → value cho mỗi row).

---

## 📚 Bổ Sung: High-Performance Memory Types

### Span\<T\> — Stack-allocated "view" vào memory

```csharp
// Span<T> = "slice" vào array/memory KHÔNG copy data
// Stack-only → không boxing, không GC pressure

int[] array = { 1, 2, 3, 4, 5, 6, 7, 8 };

Span<int> slice = array.AsSpan(2, 4);  // [3, 4, 5, 6]
// ⚠️ slice CHIA SẺ memory với array — không copy!

slice[0] = 99;  // array[2] cũng = 99!

// Dùng trong string parsing (KHÔNG tạo string mới)
ReadOnlySpan<char> text = "Hello, World!".AsSpan();
ReadOnlySpan<char> hello = text[..5];   // "Hello" — KHÔNG allocate!
ReadOnlySpan<char> world = text[7..12]; // "World" — KHÔNG allocate!

// stackalloc + Span (không heap allocation)
Span<int> buffer = stackalloc int[100];
buffer[0] = 42;

// Hạn chế: Span KHÔNG lưu trên heap
// ❌ Không dùng trong async methods
// ❌ Không làm field trong class
// → Dùng Memory<T> nếu cần
```

### Memory\<T\> — Heap-safe version of Span

```csharp
// Memory<T> = giống Span nhưng CÓ THỂ lưu trên Heap
// → Dùng được trong async methods, class fields

int[] array = { 1, 2, 3, 4, 5 };
Memory<int> memory = array.AsMemory(1, 3);  // [2, 3, 4]

// Chuyển thành Span khi cần xử lý nhanh
Span<int> span = memory.Span;
span[0] = 99;

// Async-friendly:
async Task ProcessAsync(Memory<byte> data)
{
    // Span<byte> span = data.Span;  // OK trong sync context
    await Task.Delay(100);
    // Dùng data.Span sau await
}
```

### ArrayPool\<T\> — Tái sử dụng arrays

```csharp
using System.Buffers;

// ArrayPool cho "mượn" array thay vì allocate mới
// Giảm GC pressure cho operations cần nhiều temp arrays

var pool = ArrayPool<byte>.Shared;

byte[] buffer = pool.Rent(1024);  // Mượn (có thể > 1024!)
try
{
    // Dùng buffer...
    buffer[0] = 42;
    
    // ⚠️ Chỉ dùng bytes[0..1024], KHÔNG dùng buffer.Length
    // vì buffer.Length có thể > 1024
}
finally
{
    pool.Return(buffer);  // TRẢ LẠI cho pool (không GC)
}

// Use case: HTTP server xử lý nhiều requests
// Mỗi request mượn buffer → xử lý → trả lại
// Thay vì allocate/GC mỗi request → performance tốt hơn nhiều
```

---

## ❓ Câu Hỏi Kiểm Tra Bổ Sung (Nâng Cao)

1. Stack và Heap khác nhau thế nào? Cái nào nhanh hơn?
2. Value type lưu ở đâu? Reference type lưu ở đâu?
3. Boxing là gì? Tại sao nên tránh?
4. GC có 3 generations — tại sao?
5. `using` statement giải quyết vấn đề gì?

---

## 💼 Câu Hỏi Phỏng Vấn Bổ Sung (Nâng Cao)

> **Q: Explain the Garbage Collector in .NET.**
> A: GC tự động quản lý memory trên Heap. Dùng Mark-and-Sweep: đánh dấu objects đang được reference, xóa objects không reference. Có 3 generations (0, 1, 2): Gen 0 collect thường xuyên (objects mới), Gen 2 hiếm khi (objects lâu đời). Compact memory sau khi sweep.

> **Q: Can memory leaks occur in C#?**
> A: CÓ! Dù có GC. Nguyên nhân: (1) Event handlers không unsubscribe, (2) Static collections giữ reference, (3) Unmanaged resources không Dispose, (4) Large Object Heap fragmentation, (5) Closures capture variables unintentionally.

> **Q: What is the `IDisposable` pattern? When do you need it?**
> A: Dùng khi class giữ unmanaged resources (file handles, DB connections, network sockets). Implement Dispose() để giải phóng. Dùng `using` statement để auto-dispose. Best practice: implement cả Dispose pattern + finalizer.

> **Q: What is the difference between `Finalize` and `Dispose`?**
> A: `Dispose()`: gọi thủ công (hoặc qua `using`), deterministic, nhanh. `Finalizer (~ClassName)`: GC gọi, non-deterministic, chậm (object qua 2 GC cycles). Nên dùng Dispose, dùng Finalizer chỉ là safety net.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p06-dotnet-internals](./99-answer-key-csharp.md#p06-dotnet-internals)
- Bài tập thực hành: [99-answer-key-csharp.md#p06-dotnet-internals-exercises](./99-answer-key-csharp.md#p06-dotnet-internals-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p06-dotnet-internals-deep](./97-csharp-theory-deep-dive.md#p06-dotnet-internals-deep)

