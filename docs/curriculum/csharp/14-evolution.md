# 📓 Phần 14: C# Evolution (C# 11/12 Features)

> **Nội dung**: Raw strings, list patterns, primary constructors, collection expressions  
> **Thời lượng ước tính**: 1–2 giờ

---

## 🎯 Mục Tiêu

- Nắm vững các tính năng mới trong C# 11 và C# 12
- Biết khi nào áp dụng chúng vào dự án thực tế

---

## 1. C# 11 — Raw String Literals

```csharp
// Trước C# 11: string chứa dấu ngoặc, xuống dòng phải escape
string json = "{\n  \"name\": \"Nghĩa\",\n  \"age\": 25\n}";
string path = "C:\\Users\\Documents\\file.txt";

// ✅ C# 11: Raw string literals (""" ... """)
string json = """
    {
        "name": "Nghĩa",
        "age": 25
    }
    """;

string path = """C:\Users\Documents\file.txt""";

// Interpolation trong raw string: dùng $$ với {{}}
string name = "Nghĩa";
string template = $$"""
    {
        "name": "{{name}}",
        "greeting": "Hello {world}"
    }
    """;
// {{name}} được interpolate, {world} giữ nguyên
```

### Khi nào dùng?
- JSON, XML, HTML templates
- SQL queries
- Regex patterns
- Bất kỳ string nào cần nhiều dấu ngoặc/escape

---

## 2. C# 11 — Required Members

```csharp
// Trước C# 11: không bắt buộc gán property khi dùng object initializer
class Person
{
    public string Name { get; set; }  // Có thể quên gán!
    public int Age { get; set; }
}
var p = new Person();  // Name = null → bugs!

// ✅ C# 11: required keyword
class Person
{
    public required string Name { get; set; }
    public required int Age { get; set; }
    public string? Email { get; set; }  // Không required → tùy chọn
}

// var p = new Person();  // ❌ Lỗi compile! Thiếu required members!
var p = new Person { Name = "Nghĩa", Age = 25 };  // ✅ OK
var p2 = new Person { Name = "Nghĩa", Age = 25, Email = "n@test.com" };  // ✅ OK
```

### Required + Constructor

```csharp
class Product
{
    public required string Name { get; set; }
    public required decimal Price { get; set; }
    
    // Constructor vẫn hoạt động bình thường
    [SetsRequiredMembers]  // Nói compiler: constructor này SET required members
    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
    
    public Product() { }  // Khi dùng constructor này → BẮT BUỘC gán required members
}

var p1 = new Product("Laptop", 999);  // ✅ OK nhờ [SetsRequiredMembers]
var p2 = new Product { Name = "Phone", Price = 499 };  // ✅ OK, gán đủ required
```

---

## 3. C# 12 — Primary Constructors for Classes

```csharp
// Trước C# 12: constructor + field assignment = nhiều boilerplate
class UserService
{
    private readonly IDatabase _db;
    private readonly ILogger _logger;
    
    public UserService(IDatabase db, ILogger logger)
    {
        _db = db;
        _logger = logger;
    }
    
    public void CreateUser(string name)
    {
        _logger.Log($"Creating user: {name}");
        _db.Save(new User(name));
    }
}

// ✅ C# 12: Primary constructor (gọn hơn nhiều!)
class UserService(IDatabase db, ILogger logger)
{
    public void CreateUser(string name)
    {
        logger.Log($"Creating user: {name}");  // Dùng trực tiếp parameter
        db.Save(new User(name));
    }
}

// ⚠️ Lưu ý: Primary constructor parameters là MUTABLE
// Nếu cần immutable, gán vào private readonly field:
class UserService(IDatabase db, ILogger logger)
{
    private readonly IDatabase _db = db;
    private readonly ILogger _logger = logger;
    
    public void CreateUser(string name)
    {
        _logger.Log($"Creating: {name}");
        _db.Save(new User(name));
    }
}
```

### Primary Constructors for Structs

```csharp
// Structs cũng dùng được primary constructor
struct Point(double x, double y)
{
    public double X { get; } = x;
    public double Y { get; } = y;
    
    public double DistanceTo(Point other) =>
        Math.Sqrt(Math.Pow(X - other.X, 2) + Math.Pow(Y - other.Y, 2));
}
```

---

## 4. C# 12 — Collection Expressions

```csharp
// Trước C# 12: nhiều cách khai báo collection
int[] arr1 = new int[] { 1, 2, 3 };
int[] arr2 = new[] { 1, 2, 3 };
List<int> list = new List<int> { 1, 2, 3 };
Span<int> span = stackalloc int[] { 1, 2, 3 };

// ✅ C# 12: Collection expression — cú pháp thống nhất []
int[] arr = [1, 2, 3];
List<int> list = [1, 2, 3];
Span<int> span = [1, 2, 3];
ImmutableArray<int> immutable = [1, 2, 3];

// Empty collection
int[] empty = [];
List<string> emptyList = [];

// Spread operator (..)
int[] first = [1, 2, 3];
int[] second = [4, 5, 6];
int[] combined = [..first, ..second];  // [1, 2, 3, 4, 5, 6]

// Kết hợp spread với elements
int[] withExtra = [0, ..first, 99, ..second, 100];
// [0, 1, 2, 3, 99, 4, 5, 6, 100]

// Dùng trong method call
PrintNumbers([1, 2, 3, 4, 5]);

static void PrintNumbers(IEnumerable<int> numbers)
{
    foreach (int n in numbers) Console.Write($"{n} ");
}
```

---

## 🧭 Lưu Ý Về Phiên Bản

- Luôn kiểm tra `TargetFramework` và version compiler trước khi dùng tính năng mới.
- Nếu code không compile, xác nhận:
  - SDK đã đủ mới (`dotnet --version`)
  - project dùng đúng `TargetFramework` (ví dụ `net8.0`, `net9.0`)
  - IDE đã bật language version phù hợp

---

## 📋 Tổng Hợp C# Versions

| Version | .NET | Tính năng nổi bật |
|---------|------|-------------------|
| C# 8 | .NET Core 3.0 | Nullable reference types, switch expressions, using declarations |
| C# 9 | .NET 5 | Records, init-only properties, top-level statements |
| C# 10 | .NET 6 | Global usings, file-scoped namespaces |
| C# 11 | .NET 7 | **Raw string literals**, **required members**, list patterns |
| C# 12 | .NET 8 | **Primary constructors**, **collection expressions**, alias any type |
| C# 13 | .NET 9 | params collections, lock object, escape char \e |

---

## 🎉 Chúc Mừng!

Bạn đã hoàn thành toàn bộ giáo trình **Ultimate C# Masterclass**! 

### Tiếp theo:
- 🔧 Xây dựng dự án thực tế (ASP.NET Core Web API, Blazor, MAUI)
- 📚 Học thêm: Entity Framework Core, Dependency Injection containers
- 🏗️ Tìm hiểu Clean Architecture, Domain-Driven Design
- 🧪 Viết test cho dự án thật
- 👥 Contribute vào open-source C# projects

---

## ❓ Câu Hỏi Kiểm Tra

1. Raw string literals (`"""`) giải quyết vấn đề gì?
2. `required` keyword dùng khi nào? Khác `init` thế nào?
3. `[SetsRequiredMembers]` attribute dùng để làm gì?
4. Primary constructor (C# 12) giảm boilerplate gì?
5. Primary constructor parameters có mutable không? Cách fix?
6. Collection expression `[]` thay thế cú pháp nào?
7. Spread operator `..` trong collection expressions dùng thế nào?
8. C# 9 Records vs C# 11 Required Members — dùng cái nào khi nào?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What are the most important C# features in recent versions?**
> A: C# 9: Records (immutable data types). C# 10: Global usings, file-scoped namespaces. C# 11: Raw string literals, required members. C# 12: Primary constructors, collection expressions. Trend: giảm boilerplate, tăng safety.

> **Q: What is the `required` keyword in C# 11?**
> A: Bắt buộc property phải được gán khi tạo object (compile-time check). Khác init: init cho phép bỏ qua, required KHÔNG cho phép. Kết hợp `[SetsRequiredMembers]` cho constructors mà set tất cả required members.

> **Q: What are Primary Constructors for classes (C# 12)?**
> A: Khai báo constructor parameters trực tiếp trên class definition: `class Foo(IService svc)`. Giảm boilerplate: không cần field + constructor body. ⚠️ Parameters mutable → cần gán vào `readonly` field nếu muốn immutable.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Refactor một class cũ đang nhiều boilerplate constructor sang primary constructor, nhưng vẫn đảm bảo immutability.

**BT2**: Viết module cấu hình JSON bằng raw string literals, dùng interpolation để thay đổi môi trường (`dev`, `staging`, `prod`).

**BT3**: Chuyển tất cả danh sách khởi tạo kiểu cũ sang collection expressions và kiểm tra code readability trước/sau.

**BT4**: Tạo class có `required` members, thêm constructor với `[SetsRequiredMembers]`, rồi chứng minh compile-time safety bằng ví dụ lỗi.

**BT5**: Viết tài liệu ngắn "trước/sau" cho 5 tính năng mới bạn thấy giá trị nhất và lý do chọn.

---

## ✅ Checklist Hoàn Thành Module

1. Biết chính xác khi nào dùng raw string, required members, primary constructors, collection expressions.
2. Refactor được code cũ sang style mới mà không làm giảm readability.
3. Hiểu rủi ro migration (version mismatch, mutable parameters).
4. Hoàn thành tối thiểu 3 bài tập thực hành và trình bày được trade-off.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p14-evolution](./99-answer-key-csharp.md#p14-evolution)
- Bài tập thực hành: [99-answer-key-csharp.md#p14-evolution-exercises](./99-answer-key-csharp.md#p14-evolution-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p14-evolution-deep](./97-csharp-theory-deep-dive.md#p14-evolution-deep)

