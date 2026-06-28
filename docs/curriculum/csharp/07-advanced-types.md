# 📘 Phần 7: Advanced C# Types

> **Nội dung**: Structs, Records, Tuples, Enums, Nullable, Reflection, Attributes  
> **Thời lượng ước tính**: 6–7 giờ

---

## 🎯 Mục Tiêu

- Hiểu Reflection và Attributes
- Phân biệt Struct vs Class, khi nào dùng gì
- Immutable types, Records, "with" expression
- Override Equals, GetHashCode
- Operators overloading
- Nullable types
- Querying APIs

---

## 1. Reflection

```csharp
using System.Reflection;

// Reflection: kiểm tra và thao tác type information lúc RUNTIME
Type type = typeof(string);

Console.WriteLine($"Name: {type.Name}");        // String
Console.WriteLine($"FullName: {type.FullName}");  // System.String
Console.WriteLine($"IsClass: {type.IsClass}");    // True

// Liệt kê methods
foreach (MethodInfo method in type.GetMethods().Take(5))
{
    Console.WriteLine($"  {method.Name}({string.Join(", ", method.GetParameters().Select(p => p.ParameterType.Name))})");
}

// Tạo object bằng reflection
object obj = Activator.CreateInstance(typeof(List<int>));
```

---

## 2. Attributes

```csharp
// Attribute: metadata gắn vào code
[Obsolete("Use NewMethod() instead")]
static void OldMethod() { }

[Serializable]
class MyData
{
    [Required]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; }
    
    [Range(0, 150)]
    public int Age { get; set; }
}

// Custom Attribute
[AttributeUsage(AttributeTargets.Property)]
class MustBeLargerThanAttribute : Attribute
{
    public int Minimum { get; }
    public MustBeLargerThanAttribute(int minimum) => Minimum = minimum;
}

class Product
{
    [MustBeLargerThan(0)]
    public decimal Price { get; set; }
}

// Đọc attribute bằng reflection
var props = typeof(Product).GetProperties();
foreach (var prop in props)
{
    var attr = prop.GetCustomAttribute<MustBeLargerThanAttribute>();
    if (attr != null)
        Console.WriteLine($"{prop.Name} must be > {attr.Minimum}");
}
```

---

## 3. Structs

### 3.1 Struct vs Class

```csharp
// Struct: VALUE type, lưu trên Stack
struct Point
{
    public double X { get; set; }
    public double Y { get; set; }
    
    public Point(double x, double y) => (X, Y) = (x, y);
    
    public double DistanceTo(Point other) =>
        Math.Sqrt(Math.Pow(X - other.X, 2) + Math.Pow(Y - other.Y, 2));
}

Point p1 = new Point(3, 4);
Point p2 = p1;   // COPY giá trị (không chia sẻ reference)
p2.X = 100;
// p1.X vẫn là 3!
```

| | struct | class |
|---|--------|-------|
| Kiểu | Value type | Reference type |
| Bộ nhớ | Stack | Heap |
| Copy | Copy toàn bộ giá trị | Copy reference (con trỏ) |
| Kế thừa | ❌ Không thể | ✅ Có thể |
| Null | ❌ Không thể (trừ `Nullable<T>`) | ✅ Có thể |
| Default constructor | Tự có (all zeros) | Phải tự viết |
| Khi nào dùng | Nhỏ (<16 bytes), immutable, giá trị thuần túy | Mọi trường hợp khác |

### 3.2 Immutable Structs

```csharp
// ✅ NÊN làm struct immutable (không đổi sau khi tạo)
readonly struct Temperature
{
    public double Celsius { get; }
    public double Fahrenheit => Celsius * 9 / 5 + 32;
    
    public Temperature(double celsius) => Celsius = celsius;
    
    // Tạo object MỚI thay vì sửa object cũ
    public Temperature AddDegrees(double degrees) => new(Celsius + degrees);
}
```

### 3.3 "with" Expression (Non-destructive Mutation)

```csharp
readonly struct Point
{
    public double X { get; init; }
    public double Y { get; init; }
}

Point p1 = new Point { X = 3, Y = 4 };
Point p2 = p1 with { X = 10 };  // Tạo bản copy, chỉ đổi X
// p1 = (3, 4), p2 = (10, 4)
```

---

## 4. Equals & GetHashCode

### 4.1 Override Equals

```csharp
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public override bool Equals(object? obj)
    {
        if (obj is Person other)
        {
            return Name == other.Name && Age == other.Age;
        }
        return false;
    }
    
    // Implement IEquatable<T> cho type-safe comparison
    public bool Equals(Person? other)
    {
        if (other is null) return false;
        return Name == other.Name && Age == other.Age;
    }
    
    // PHẢI override GetHashCode khi override Equals!
    public override int GetHashCode()
    {
        return HashCode.Combine(Name, Age);
    }
    
    // Override == operator
    public static bool operator ==(Person? a, Person? b)
    {
        if (ReferenceEquals(a, b)) return true;
        if (a is null || b is null) return false;
        return a.Equals(b);
    }
    
    public static bool operator !=(Person? a, Person? b) => !(a == b);
}
```

### 4.2 Hash Functions

```csharp
// GetHashCode trả về int đại diện cho object
// Quy tắc:
// 1. Nếu a.Equals(b) → a.GetHashCode() == b.GetHashCode()
// 2. Ngược lại KHÔNG nhất thiết đúng (hash collision)
// 3. Dùng trong Dictionary, HashSet để tìm nhanh

var dict = new Dictionary<Person, string>();
var p = new Person { Name = "A", Age = 25 };
dict[p] = "info";

// Dictionary dùng GetHashCode() để tìm bucket
// rồi dùng Equals() để xác nhận chính xác
```

---

## 5. Operator Overloading

```csharp
struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency) =>
        (Amount, Currency) = (amount, currency);
    
    // Overload +
    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
        return new Money(a.Amount + b.Amount, a.Currency);
    }
    
    // Overload -
    public static Money operator -(Money a, Money b) =>
        new Money(a.Amount - b.Amount, a.Currency);
    
    // Implicit conversion: Money → decimal
    public static implicit operator decimal(Money m) => m.Amount;
    
    // Explicit conversion: decimal → Money
    public static explicit operator Money(decimal amount) => new(amount, "USD");
    
    public override string ToString() => $"{Amount:F2} {Currency}";
}

Money a = new(100, "USD");
Money b = new(50, "USD");
Money c = a + b;  // 150.00 USD
decimal value = c; // 150 (implicit)
Money d = (Money)200m; // 200.00 USD (explicit)
```

---

## 6. Records (C# 9+)

```csharp
// Record: class đặc biệt tự động có Equals, GetHashCode, ToString, "with"
record Person(string Name, int Age);

Person p1 = new("Nghĩa", 25);
Person p2 = new("Nghĩa", 25);

Console.WriteLine(p1 == p2);       // true (value equality!)
Console.WriteLine(p1);             // Person { Name = Nghĩa, Age = 25 }
Console.WriteLine(p1.GetHashCode() == p2.GetHashCode()); // true

Person p3 = p1 with { Age = 26 }; // Non-destructive mutation
// p1 = Person { Name = Nghĩa, Age = 25 }  (KHÔNG đổi)
// p3 = Person { Name = Nghĩa, Age = 26 }

// Record struct (C# 10+)
record struct Point(double X, double Y);
```

**Record vs Class vs Struct:**

| | class | struct | record (class) | record struct |
|---|-------|--------|----------------|---------------|
| Kiểu | Reference | Value | Reference | Value |
| Equality | Reference | Value | **Value** ✨ | **Value** |
| Immutable | Manual | Manual | **Mặc định** ✨ | **Mặc định** |
| `with` | ❌ | ❌ | ✅ | ✅ |
| ToString | Type name | Type name | **Pretty print** ✨ | **Pretty print** |

---

## 7. Nullable Types

### 7.1 Nullable Value Types

```csharp
int? nullableInt = null;   // Nullable<int>
double? temperature = null;

if (nullableInt.HasValue)
    Console.WriteLine(nullableInt.Value);

// Null-coalescing
int result = nullableInt ?? 0;           // 0 nếu null
int result2 = nullableInt.GetValueOrDefault(); // 0 nếu null

// Null-coalescing assignment
nullableInt ??= 42;  // Gán 42 CHỈ KHI null
```

### 7.2 Nullable Reference Types (C# 8+)

```csharp
#nullable enable

string nonNull = "Hello";    // KHÔNG được null
string? canBeNull = null;    // CÓ THỂ null

// Compiler WARNING nếu:
// string bad = null;          // ⚠️ Warning!
// int len = canBeNull.Length; // ⚠️ Warning! Có thể NullReferenceException

// An toàn:
int len = canBeNull?.Length ?? 0;  // 0 nếu null

// Null-forgiving operator
string definitelyNotNull = canBeNull!;  // "Tôi biết nó không null"
```

---

## 8. Querying APIs

```csharp
using System.Net.Http;
using System.Text.Json;

// Gọi REST API
using HttpClient client = new();
string json = await client.GetStringAsync("https://swapi.dev/api/planets/");

// Deserialize
var result = JsonSerializer.Deserialize<ApiResponse>(json);

// DTO (Data Transfer Object
class PlanetDto
{
    [JsonPropertyName("name")]
    public string Name { get; set; }
    
    [JsonPropertyName("population")]
    public string Population { get; set; }
    
    [JsonPropertyName("diameter")]
    public string Diameter { get; set; }
}

// Chuyển DTO → Domain model
record Planet(string Name, long? Population, int? Diameter);

Planet ToDomain(PlanetDto dto) => new(
    dto.Name,
    long.TryParse(dto.Population, out long pop) ? pop : null,
    int.TryParse(dto.Diameter, out int dia) ? dia : null
);
```

---

## 🧪 Coding Exercises

| # | Bài | Kiến thức |
|---|-----|-----------|
| 44 | Attributes - MustBeLargerThanAttribute | Custom attributes |
| 45 | Immutable struct - Time | Readonly struct |
| 46 | Equals - FullName class | Override Equals |
| 47 | Operators overloading - Time structs | Operator overloading |
| 48 | GetHashCode - Time struct | HashCode |
| 49 | Records - GpsCoordinates | Record types |

---

## ❓ Câu Hỏi Kiểm Tra

1. Reflection là gì? Sử dụng khi nào?
2. Attribute gắn lên code để làm gì? Cho 3 built-in attributes.
3. Struct vs Class — 5 điểm khác nhau chính?
4. Tại sao struct NÊN immutable?
5. `Equals` vs `==` — mặc định khác nhau thế nào cho class vs struct?
6. Tại sao PHẢI override `GetHashCode` khi override `Equals`?
7. Record tự động có gì mà class không có?
8. `int?` khác `int` thế nào? Nó là gì bên dưới?
9. `?.` và `??` operators dùng khi nào?
10. Implicit vs Explicit conversion — khác nhau?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between struct and class?**
> A: Struct: value type (Stack), copy by value, no inheritance, no null, default constructor. Class: reference type (Heap), copy by reference, supports inheritance. Dùng struct cho small, immutable value objects (<16 bytes).

> **Q: What are Records in C#?**
> A: Record = immutable reference type (C# 9) hoặc value type (C# 10). Tự động có: value equality, ToString, `with` expression, deconstruction. Lý tưởng cho DTOs, domain values, immutable data.

> **Q: Why must you override GetHashCode when you override Equals?**
> A: Dictionary/HashSet dùng GetHashCode() để hash bucket → nếu 2 objects Equal mà hash khác nhau → tìm không thấy key! Quy tắc: `a.Equals(b)` → `a.GetHashCode() == b.GetHashCode()`.

> **Q: What is Reflection and when would you use it?**
> A: Reflection cho phép kiểm tra/thao tác types lúc runtime. Dùng cho: serialization, DI containers, ORM mapping, attribute processing. Trade-off: chậm hơn compile-time, mất type safety.

> **Q: What is the null-conditional operator (?.) and null-coalescing operator (??)? **
> A: `?.` short-circuit nếu null: `person?.Address?.City` trả về null nếu bất kỳ phần nào null. `??` cung cấp default: `name ?? "Unknown"`. `??=` gán nếu null: `x ??= 42`.

---

## 🏋️ Bài Tập Thực Hành
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency) =>
        (Amount, Currency) = (amount, currency);
    
    // Overload +
    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
        return new Money(a.Amount + b.Amount, a.Currency);
    }
    
    // Overload -
    public static Money operator -(Money a, Money b) =>
        new Money(a.Amount - b.Amount, a.Currency);
    
    // Implicit conversion: Money → decimal
    public static implicit operator decimal(Money m) => m.Amount;
    
    // Explicit conversion: decimal → Money
    public static explicit operator Money(decimal amount) => new(amount, "USD");
    
    public override string ToString() => $"{Amount:F2} {Currency}";
}

Money a = new(100, "USD");
Money b = new(50, "USD");
Money c = a + b;  // 150.00 USD
decimal value = c; // 150 (implicit)
Money d = (Money)200m; // 200.00 USD (explicit)
```

---

## 6. Records (C# 9+)

```csharp
// Record: class đặc biệt tự động có Equals, GetHashCode, ToString, "with"
record Person(string Name, int Age);

Person p1 = new("Nghĩa", 25);
Person p2 = new("Nghĩa", 25);

Console.WriteLine(p1 == p2);       // true (value equality!)
Console.WriteLine(p1);             // Person { Name = Nghĩa, Age = 25 }
Console.WriteLine(p1.GetHashCode() == p2.GetHashCode()); // true

Person p3 = p1 with { Age = 26 }; // Non-destructive mutation
// p1 = Person { Name = Nghĩa, Age = 25 }  (KHÔNG đổi)
// p3 = Person { Name = Nghĩa, Age = 26 }

// Record struct (C# 10+)
record struct Point(double X, double Y);
```

**Record vs Class vs Struct:**

| | class | struct | record (class) | record struct |
|---|-------|--------|----------------|---------------|
| Kiểu | Reference | Value | Reference | Value |
| Equality | Reference | Value | **Value** ✨ | **Value** |
| Immutable | Manual | Manual | **Mặc định** ✨ | **Mặc định** |
| `with` | ❌ | ❌ | ✅ | ✅ |
| ToString | Type name | Type name | **Pretty print** ✨ | **Pretty print** |

---

## 7. Nullable Types

### 7.1 Nullable Value Types

```csharp
int? nullableInt = null;   // Nullable<int>
double? temperature = null;

if (nullableInt.HasValue)
    Console.WriteLine(nullableInt.Value);

// Null-coalescing
int result = nullableInt ?? 0;           // 0 nếu null
int result2 = nullableInt.GetValueOrDefault(); // 0 nếu null

// Null-coalescing assignment
nullableInt ??= 42;  // Gán 42 CHỈ KHI null
```

### 7.2 Nullable Reference Types (C# 8+)

```csharp
#nullable enable

string nonNull = "Hello";    // KHÔNG được null
string? canBeNull = null;    // CÓ THỂ null

// Compiler WARNING nếu:
// string bad = null;          // ⚠️ Warning!
// int len = canBeNull.Length; // ⚠️ Warning! Có thể NullReferenceException

// An toàn:
int len = canBeNull?.Length ?? 0;  // 0 nếu null

// Null-forgiving operator
string definitelyNotNull = canBeNull!;  // "Tôi biết nó không null"
```

---

## 8. Querying APIs

```csharp
using System.Net.Http;
using System.Text.Json;

// Gọi REST API
using HttpClient client = new();
string json = await client.GetStringAsync("https://swapi.dev/api/planets/");

// Deserialize
var result = JsonSerializer.Deserialize<ApiResponse>(json);

// DTO (Data Transfer Object
class PlanetDto
{
    [JsonPropertyName("name")]
    public string Name { get; set; }
    
    [JsonPropertyName("population")]
    public string Population { get; set; }
    
    [JsonPropertyName("diameter")]
    public string Diameter { get; set; }
}

// Chuyển DTO → Domain model
record Planet(string Name, long? Population, int? Diameter);

Planet ToDomain(PlanetDto dto) => new(
    dto.Name,
    long.TryParse(dto.Population, out long pop) ? pop : null,
    int.TryParse(dto.Diameter, out int dia) ? dia : null
);
```

---

## 🧪 Coding Exercises

| # | Bài | Kiến thức |
|---|-----|-----------|
| 44 | Attributes - MustBeLargerThanAttribute | Custom attributes |
| 45 | Immutable struct - Time | Readonly struct |
| 46 | Equals - FullName class | Override Equals |
| 47 | Operators overloading - Time structs | Operator overloading |
| 48 | GetHashCode - Time struct | HashCode |
| 49 | Records - GpsCoordinates | Record types |

---

## ❓ Câu Hỏi Kiểm Tra

1. Reflection là gì? Sử dụng khi nào?
2. Attribute gắn lên code để làm gì? Cho 3 built-in attributes.
3. Struct vs Class — 5 điểm khác nhau chính?
4. Tại sao struct NÊN immutable?
5. `Equals` vs `==` — mặc định khác nhau thế nào cho class vs struct?
6. Tại sao PHẢI override `GetHashCode` khi override `Equals`?
7. Record tự động có gì mà class không có?
8. `int?` khác `int` thế nào? Nó là gì bên dưới?
9. `?.` và `??` operators dùng khi nào?
10. Implicit vs Explicit conversion — khác nhau?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between struct and class?**
> A: Struct: value type (Stack), copy by value, no inheritance, no null, default constructor. Class: reference type (Heap), copy by reference, supports inheritance. Dùng struct cho small, immutable value objects (<16 bytes).

> **Q: What are Records in C#?**
> A: Record = immutable reference type (C# 9) hoặc value type (C# 10). Tự động có: value equality, ToString, `with` expression, deconstruction. Lý tưởng cho DTOs, domain values, immutable data.

> **Q: Why must you override GetHashCode when you override Equals?**
> A: Dictionary/HashSet dùng GetHashCode() để hash bucket → nếu 2 objects Equal mà hash khác nhau → tìm không thấy key! Quy tắc: `a.Equals(b)` → `a.GetHashCode() == b.GetHashCode()`.

> **Q: What is Reflection and when would you use it?**
> A: Reflection cho phép kiểm tra/thao tác types lúc runtime. Dùng cho: serialization, DI containers, ORM mapping, attribute processing. Trade-off: chậm hơn compile-time, mất type safety.

> **Q: What is the null-conditional operator (?.) and null-coalescing operator (??)? **
> A: `?.` short-circuit nếu null: `person?.Address?.City` trả về null nếu bất kỳ phần nào null. `??` cung cấp default: `name ?? "Unknown"`. `??=` gán nếu null: `x ??= 42`.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Tạo `readonly struct Money` (Amount, Currency) với operator +, -, ==, !=, implicit/explicit conversions.

**BT2**: Viết validator dùng custom attributes: `[Required]`, `[Range(min, max)]`. Dùng reflection để kiểm tra.

**BT3**: Tạo record `Student(string Name, double GPA)`. Implement `IComparable<Student>` sort theo GPA. Test với List.Sort().

---

## 9. Indexers

**Indexer** cho phép truy cập object **như mảng** bằng toán tử `[]`.

```csharp
public class StringCollection
{
    private string[] strArray = new string[10];

    // Indexer — dùng this[]
    public string this[int index]
    {
        get
        {
            if (index < 0 || index >= strArray.Length)
                throw new ArgumentOutOfRangeException();
            return strArray[index];
        }
        set
        {
            if (index < 0 || index >= strArray.Length)
                throw new IndexOutOfRangeException();
            strArray[index] = value;
        }
    }
}

// Sử dụng
StringCollection collection = new StringCollection();
collection[0] = "Hello";    // SET: dùng [] như mảng
collection[1] = "World";
Console.WriteLine(collection[0]); // GET: "Hello"
```

| | **Property** | **Indexer** |
|---|---|---|
| Khai báo | Tên cụ thể | `this[]` keyword |
| Truy cập | `.Name` | `[0]` |
| Static | ✅ Có thể | ❌ Không |

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p07-advanced-types](./99-answer-key-csharp.md#p07-advanced-types)
- Bài tập thực hành: [99-answer-key-csharp.md#p07-advanced-types-exercises](./99-answer-key-csharp.md#p07-advanced-types-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p07-advanced-types-deep](./97-csharp-theory-deep-dive.md#p07-advanced-types-deep)
