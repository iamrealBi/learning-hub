# 📓 Phần 5: LINQ

> **Nội dung**: LINQ methods, deferred execution, query vs method syntax  
> **Thời lượng ước tính**: 4–5 giờ

---

## 🎯 Mục Tiêu

- Hiểu LINQ là gì và tại sao nó quan trọng
- Thành thạo các LINQ method phổ biến
- Hiểu deferred execution (thực thi trễ)
- Chaining methods hiệu quả

---

## 1. LINQ là gì?

**LINQ** = **L**anguage **IN**tegrated **Q**uery — truy vấn dữ liệu trực tiếp trong C#.

```csharp
using System.Linq;  // Namespace chứa LINQ

List<int> numbers = new() { 5, 2, 8, 1, 9, 3, 7, 4, 6 };

// ❌ Không có LINQ: viết vòng lặp
List<int> evenNumbers = new();
foreach (int n in numbers)
    if (n % 2 == 0)
        evenNumbers.Add(n);
evenNumbers.Sort();

// ✅ Có LINQ: gọn và rõ ràng
var result = numbers
    .Where(n => n % 2 == 0)
    .OrderBy(n => n)
    .ToList();
// [2, 4, 6, 8]
```

### LINQ hoạt động trên IEnumerable<T>

```csharp
// LINQ là extension methods trên IEnumerable<T>
// → Dùng được trên: Array, List, Dictionary, HashSet, v.v.
int[] array = { 1, 2, 3 };
List<string> list = new() { "a", "b" };
Dictionary<string, int> dict = new() { ["x"] = 1 };

// Tất cả đều dùng LINQ được!
array.Where(x => x > 1);
list.Any(s => s == "a");
dict.Select(kvp => kvp.Key);
```

---

## 2. Deferred Execution (Thực thi trễ)

```csharp
List<int> numbers = new() { 1, 2, 3, 4, 5 };

// LINQ query CHƯA thực thi ngay!
var query = numbers.Where(n => n > 3);  // Chỉ "ghi nhận" query

numbers.Add(6);  // Thêm phần tử SAU khi tạo query

// Query thực thi KHI duyệt kết quả
foreach (int n in query)  // Lúc này mới chạy filter
    Console.Write($"{n} ");  // 4 5 6 (bao gồm 6 vừa thêm!)

// Force immediate execution:
var list = numbers.Where(n => n > 3).ToList();  // Thực thi ngay
var array = numbers.Where(n => n > 3).ToArray(); // Thực thi ngay
int count = numbers.Count(n => n > 3);            // Thực thi ngay
```

---

## 3. Các LINQ Methods Quan Trọng

### 3.1 Filtering: Where

```csharp
List<Person> people = GetPeople();

var adults = people.Where(p => p.Age >= 18);
var vietnamPeople = people.Where(p => p.Country == "Vietnam");

// Multiple conditions
var targets = people.Where(p => p.Age >= 18 && p.Age <= 30 && p.IsActive);
```

### 3.2 Checking: Any, All, Contains

```csharp
List<int> numbers = new() { 1, 2, 3, 4, 5 };

// Any: có BẤT KỲ phần tử nào thỏa điều kiện?
bool hasEven = numbers.Any(n => n % 2 == 0);    // true
bool isEmpty = !numbers.Any();                     // false (có phần tử)

// All: TẤT CẢ phần tử đều thỏa điều kiện?
bool allPositive = numbers.All(n => n > 0);       // true
bool allEven = numbers.All(n => n % 2 == 0);     // false

// Contains: có chứa giá trị cụ thể?
bool has3 = numbers.Contains(3);                   // true
```

### 3.3 Counting: Count

```csharp
int total = numbers.Count;                       // Property (nhanh)
int count = numbers.Count();                     // LINQ method
int evenCount = numbers.Count(n => n % 2 == 0); // Đếm có điều kiện
```

### 3.4 Ordering: OrderBy, ThenBy

```csharp
List<Person> people = GetPeople();

// Sắp xếp tăng dần
var byAge = people.OrderBy(p => p.Age);

// Sắp xếp giảm dần
var byAgeDesc = people.OrderByDescending(p => p.Age);

// Sắp xếp nhiều tiêu chí
var sorted = people
    .OrderBy(p => p.LastName)
    .ThenBy(p => p.FirstName)
    .ThenByDescending(p => p.Age);
```

### 3.5 Selecting: First, Last, Single

```csharp
List<int> numbers = new() { 5, 2, 8, 1, 9 };

int first = numbers.First();                    // 5
int firstEven = numbers.First(n => n % 2 == 0); // 2
int last = numbers.Last();                       // 9

// OrDefault: trả về default nếu không tìm thấy (không throw exception)
int? firstNeg = numbers.FirstOrDefault(n => n < 0);  // 0 (default of int)

// Single: CHỈ MỘT phần tử thỏa mãn (throw nếu 0 hoặc >1)
// int single = numbers.Single(n => n == 8);  // 8
```

### 3.6 Projection: Select

```csharp
List<Person> people = GetPeople();

// Chuyển đổi từ kiểu này sang kiểu khác
var names = people.Select(p => p.Name);                     // IEnumerable<string>
var infos = people.Select(p => $"{p.Name} ({p.Age} tuổi)"); // IEnumerable<string>

// Select với anonymous type
var dtos = people.Select(p => new
{
    FullName = $"{p.FirstName} {p.LastName}",
    IsAdult = p.Age >= 18
});

foreach (var dto in dtos)
    Console.WriteLine($"{dto.FullName}: {dto.IsAdult}");
```

### 3.7 Removing duplicates: Distinct

```csharp
List<int> numbers = new() { 1, 2, 2, 3, 3, 3, 4 };
var unique = numbers.Distinct();  // 1, 2, 3, 4

// DistinctBy (C# 6+)
var uniqueByAge = people.DistinctBy(p => p.Age);
```

### 3.8 Aggregation: Sum, Average, Min, Max

```csharp
List<int> numbers = new() { 10, 20, 30, 40, 50 };

int sum = numbers.Sum();              // 150
double avg = numbers.Average();        // 30.0
int min = numbers.Min();              // 10
int max = numbers.Max();              // 50

// Với selector
double avgAge = people.Average(p => p.Age);
decimal totalSalary = people.Sum(p => p.Salary);
Person oldest = people.MaxBy(p => p.Age);
```

### 3.9 Method Chaining (Chuỗi phương thức)

```csharp
// Power of LINQ: chain nhiều operations
var result = people
    .Where(p => p.Age >= 18)              // Lọc người lớn
    .Where(p => p.Country == "Vietnam")   // Lọc quốc gia
    .OrderByDescending(p => p.Salary)     // Sắp xếp theo lương
    .Select(p => new                       // Chuyển đổi shape
    {
        Name = p.Name,
        Salary = p.Salary
    })
    .Take(10)                              // Lấy 10 người đầu
    .ToList();                             // Thực thi & lưu kết quả
```

---

## 4. Tổng Hợp LINQ Methods

| Method | Mô tả | Trả về |
|--------|--------|--------|
| `Where` | Lọc | `IEnumerable<T>` |
| `Select` | Chuyển đổi | `IEnumerable<TResult>` |
| `OrderBy` / `OrderByDescending` | Sắp xếp | `IOrderedEnumerable<T>` |
| `ThenBy` / `ThenByDescending` | Sắp xếp phụ | `IOrderedEnumerable<T>` |
| `First` / `FirstOrDefault` | Phần tử đầu | `T` |
| `Last` / `LastOrDefault` | Phần tử cuối | `T` |
| `Single` / `SingleOrDefault` | Phần tử duy nhất | `T` |
| `Any` | Có thỏa mãn? | `bool` |
| `All` | Tất cả thỏa mãn? | `bool` |
| `Contains` | Có chứa? | `bool` |
| `Count` | Đếm | `int` |
| `Sum` / `Average` / `Min` / `Max` | Tổng hợp số | number |
| `Distinct` | Loại trùng | `IEnumerable<T>` |
| `Take` / `Skip` | Lấy/Bỏ N phần tử | `IEnumerable<T>` |
| `ToList` / `ToArray` | Convert | `List<T>` / `T[]` |
| `ToDictionary` | Convert | `Dictionary<TKey, TValue>` |
| `GroupBy` | Nhóm | `IEnumerable<IGrouping>` |
| `SelectMany` | Flatten | `IEnumerable<TResult>` |
| `Zip` | Ghép cặp | `IEnumerable<(T1, T2)>` |
| `Aggregate` | Reduce | `T` |

---

## 🧪 Coding Exercises

| # | Bài | LINQ Methods |
|---|-----|-------------|
| 37 | Any & All | `Any`, `All` |
| 38 | Count & Contains | `Count`, `Contains` |
| 39 | OrderBy, First & Last | `OrderBy`, `First`, `Last` |
| 40 | Where & Distinct | `Where`, `Distinct` |
| 41 | Select & Average | `Select`, `Average` |

---

## ❓ Câu Hỏi Kiểm Tra

1. LINQ là gì? Viết tắt của cái gì?
2. Deferred execution là gì? Cho ví dụ khi nó gây bất ngờ.
3. `FirstOrDefault` vs `First` — khi nào dùng cái nào?
4. `Select` vs `SelectMany` — khác nhau thế nào?
5. `Any()` vs `Count() > 0` — cái nào nên dùng? Tại sao?
6. `.Count` (property) vs `.Count()` (method) — khác nhau?
7. `ToList()` gọi TRƯỚC hay SAU các method chain? Tại sao quan trọng?
8. `OrderBy` có stable sort không? `ThenBy` dùng khi nào?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is deferred execution in LINQ?**
> A: LINQ query KHÔNG chạy ngay khi khai báo. Chỉ chạy khi enumerate (foreach, ToList, Count...). Lợi ích: tiết kiệm memory, chỉ tính khi cần. `ToList()`, `ToArray()`, `Count()`, `First()` trigger immediate execution.

> **Q: What is the difference between `IEnumerable` and `IQueryable`?**
> A: `IEnumerable<T>`: xử lý in-memory (LINQ to Objects). `IQueryable<T>`: xử lý remote (LINQ to SQL/EF) — query được translate sang SQL. IQueryable kế thừa IEnumerable. Dùng IQueryable cho database để tận dụng server-side filtering.

> **Q: What is the difference between `Select` and `SelectMany`?**
> A: `Select` map 1:1 (mỗi item → 1 result). `SelectMany` map 1:N và flatten (mỗi item → nhiều results, gộp thành 1 list). Ví dụ: `customers.SelectMany(c => c.Orders)` trả về tất cả orders từ tất cả customers.

> **Q: What is the difference between `First` and `Single`?**
> A: `First` trả về phần tử ĐẦU TIÊN (có thể có nhiều). `Single` yêu cầu CHỈ CÓ 1 phần tử thỏa mãn — throw nếu 0 hoặc >1. Dùng Single khi business logic đảm bảo uniqueness.

> **Q: Why should you prefer `Any()` over `Count() > 0`?**
> A: `Any()` dừng ngay khi tìm thấy 1 phần tử → O(1) best case. `Count()` phải duyệt TOÀN BỘ collection → O(n). Cho big data, Any() nhanh hơn rất nhiều.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Cho `List<string> words`, viết LINQ: tìm tất cả từ dài hơn 5 ký tự, uppercase, sorted alphabetically.

**BT2**: Cho `List<Student>` (Name, Grade, Score), viết LINQ: group by Grade, tính Average Score mỗi group.

**BT3**: Implement `Where`, `Select`, `Any` methods KHÔNG dùng LINQ (dùng `yield return`).

**BT4**: Viết LINQ query từ list Products: top 5 sản phẩm đắt nhất theo category, format output đẹp.

---

## 📚 Bổ Sung: Advanced LINQ Methods

### Aggregate (Reduce)

```csharp
// Aggregate = reduce: gộp collection thành 1 giá trị
List<int> numbers = new() { 1, 2, 3, 4, 5 };

// Tổng (tương đương Sum)
int sum = numbers.Aggregate((acc, n) => acc + n);  // 15

// Tích
int product = numbers.Aggregate((acc, n) => acc * n);  // 120

// Với seed (giá trị khởi tạo)
string sentence = new[] { "Hello", "World" }
    .Aggregate("Greeting:", (acc, word) => $"{acc} {word}");
// "Greeting: Hello World"

// Thực tế: ghép chuỗi với separator
string csv = names.Aggregate((a, b) => $"{a},{b}");
// Tốt hơn: string.Join(",", names)
```

### GroupBy Chi Tiết

```csharp
var students = new List<Student>
{
    new("An", "A", 9.0), new("Bình", "B", 7.5),
    new("Châu", "A", 8.5), new("Dũng", "B", 8.0)
};

// GroupBy: nhóm theo tiêu chí
var groups = students
    .GroupBy(s => s.Grade)
    .Select(g => new
    {
        Grade = g.Key,           // Giá trị nhóm ("A", "B")
        Count = g.Count(),
        AvgScore = g.Average(s => s.Score),
        Students = g.ToList()
    });

foreach (var g in groups)
    Console.WriteLine($"Grade {g.Grade}: {g.Count} students, avg = {g.AvgScore:F1}");
```

### Join & GroupJoin

```csharp
// JOIN: similar SQL INNER JOIN
var result = students
    .Join(classes,
        student => student.ClassId,   // Key từ students
        cls => cls.Id,                 // Key từ classes
        (student, cls) => new         // Kết hợp
        {
            student.Name,
            ClassName = cls.Name
        });

// GROUPJOIN: similar SQL LEFT JOIN + GROUP
var classWithStudents = classes
    .GroupJoin(students,
        cls => cls.Id,
        student => student.ClassId,
        (cls, studs) => new
        {
            Class = cls.Name,
            Students = studs.ToList(),
            Count = studs.Count()
        });
```

### Query Syntax vs Method Syntax

```csharp
// Method Syntax (Lambda) ⭐ Phổ biến hơn trong C#
var result1 = students
    .Where(s => s.Age >= 18)
    .OrderBy(s => s.Name)
    .Select(s => s.Name);

// Query Syntax (SQL-like)
var result2 = from s in students
              where s.Age >= 18
              orderby s.Name
              select s.Name;

// Cả hai tạo ra KẾT QUẢ GIỐNG NHAU
// Method Syntax: linh hoạt hơn, có nhiều methods hơn
// Query Syntax: dễ đọc hơn cho JOIN, GROUP BY phức tạp
```

---

## ❓ Câu Hỏi Kiểm Tra Bổ Sung (Nâng Cao)

1. LINQ là gì? Viết tắt của cái gì?
2. Deferred execution (lazy) vs Immediate execution — khác nhau?
3. `FirstOrDefault` vs `First` — khi nào dùng cái nào?
4. `Select` vs `SelectMany` — khác nhau thế nào?
5. Method syntax vs Query syntax — cái nào nên dùng?

---

## 💼 Câu Hỏi Phỏng Vấn Bổ Sung (Nâng Cao)

> **Q: What is deferred execution in LINQ?**
> A: LINQ query KHÔNG chạy ngay khi khai báo. Chỉ chạy khi enumerate (foreach, ToList, Count...). Lợi ích: tiết kiệm memory, chỉ tính khi cần. `ToList()`, `ToArray()`, `Count()` trigger immediate execution.

> **Q: What is the difference between `IEnumerable` and `IQueryable`?**
> A: `IEnumerable<T>`: xử lý in-memory (LINQ to Objects). `IQueryable<T>`: xử lý remote (LINQ to SQL/EF) — query được translate sang SQL. IQueryable kế thừa IEnumerable. Dùng IQueryable cho database queries để tận dụng server-side filtering.

> **Q: What is the difference between `Select` and `SelectMany`?**
> A: `Select` map 1:1 (mỗi item → 1 result). `SelectMany` map 1:N và flatten (mỗi item → nhiều results, gộp thành 1 list). Ví dụ: `customers.SelectMany(c => c.Orders)` trả về tất cả orders từ tất cả customers.

---

## 🏋️ Bài Tập Bổ Sung (Nâng Cao)

**BT1**: Cho `List<string> words`, viết LINQ: tìm tất cả từ dài hơn 5 ký tự, uppercase, sorted alphabetically.

**BT2**: Cho `List<Student>` (Name, Grade, Score), viết LINQ: group by Grade, tính Average Score mỗi group.

**BT3**: Implement `Where`, `Select`, `Any` methods KHÔNG dùng LINQ (dùng yield return).

| `Distinct` | Loại trùng | `IEnumerable<T>` |
| `Take` / `Skip` | Lấy/Bỏ N phần tử | `IEnumerable<T>` |
| `ToList` / `ToArray` | Convert | `List<T>` / `T[]` |
| `ToDictionary` | Convert | `Dictionary<TKey, TValue>` |
| `GroupBy` | Nhóm | `IEnumerable<IGrouping>` |
| `SelectMany` | Flatten | `IEnumerable<TResult>` |
| `Zip` | Ghép cặp | `IEnumerable<(T1, T2)>` |
| `Aggregate` | Reduce | `T` |

---

## 🧪 Coding Exercises

| # | Bài | LINQ Methods |
|---|-----|-------------|
| 37 | Any & All | `Any`, `All` |
| 38 | Count & Contains | `Count`, `Contains` |
| 39 | OrderBy, First & Last | `OrderBy`, `First`, `Last` |
| 40 | Where & Distinct | `Where`, `Distinct` |
| 41 | Select & Average | `Select`, `Average` |

---

## ❓ Câu Hỏi Kiểm Tra

1. LINQ là gì? Viết tắt của cái gì?
2. Deferred execution là gì? Cho ví dụ khi nó gây bất ngờ.
3. `FirstOrDefault` vs `First` — khi nào dùng cái nào?
4. `Select` vs `SelectMany` — khác nhau thế nào?
5. `Any()` vs `Count() > 0` — cái nào nên dùng? Tại sao?
6. `.Count` (property) vs `.Count()` (method) — khác nhau?
7. `ToList()` gọi TRƯỚC hay SAU các method chain? Tại sao quan trọng?
8. `OrderBy` có stable sort không? `ThenBy` dùng khi nào?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is deferred execution in LINQ?**
> A: LINQ query KHÔNG chạy ngay khi khai báo. Chỉ chạy khi enumerate (foreach, ToList, Count...). Lợi ích: tiết kiệm memory, chỉ tính khi cần. `ToList()`, `ToArray()`, `Count()`, `First()` trigger immediate execution.

> **Q: What is the difference between `IEnumerable` and `IQueryable`?**
> A: `IEnumerable<T>`: xử lý in-memory (LINQ to Objects). `IQueryable<T>`: xử lý remote (LINQ to SQL/EF) — query được translate sang SQL. IQueryable kế thừa IEnumerable. Dùng IQueryable cho database để tận dụng server-side filtering.

> **Q: What is the difference between `Select` and `SelectMany`?**
> A: `Select` map 1:1 (mỗi item → 1 result). `SelectMany` map 1:N và flatten (mỗi item → nhiều results, gộp thành 1 list). Ví dụ: `customers.SelectMany(c => c.Orders)` trả về tất cả orders từ tất cả customers.

> **Q: What is the difference between `First` and `Single`?**
> A: `First` trả về phần tử ĐẦU TIÊN (có thể có nhiều). `Single` yêu cầu CHỈ CÓ 1 phần tử thỏa mãn — throw nếu 0 hoặc >1. Dùng Single khi business logic đảm bảo uniqueness.

> **Q: Why should you prefer `Any()` over `Count() > 0`?**
> A: `Any()` dừng ngay khi tìm thấy 1 phần tử → O(1) best case. `Count()` phải duyệt TOÀN BỘ collection → O(n). Cho big data, Any() nhanh hơn rất nhiều.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Cho `List<string> words`, viết LINQ: tìm tất cả từ dài hơn 5 ký tự, uppercase, sorted alphabetically.

**BT2**: Cho `List<Student>` (Name, Grade, Score), viết LINQ: group by Grade, tính Average Score mỗi group.

**BT3**: Implement `Where`, `Select`, `Any` methods KHÔNG dùng LINQ (dùng `yield return`).

**BT4**: Viết LINQ query từ list Products: top 5 sản phẩm đắt nhất theo category, format output đẹp.

---

## 📚 Bổ Sung: Advanced LINQ Methods

### Aggregate (Reduce)

```csharp
// Aggregate = reduce: gộp collection thành 1 giá trị
List<int> numbers = new() { 1, 2, 3, 4, 5 };

// Tổng (tương đương Sum)
int sum = numbers.Aggregate((acc, n) => acc + n);  // 15

// Tích
int product = numbers.Aggregate((acc, n) => acc * n);  // 120

// Với seed (giá trị khởi tạo)
string sentence = new[] { "Hello", "World" }
    .Aggregate("Greeting:", (acc, word) => $"{acc} {word}");
// "Greeting: Hello World"

// Thực tế: ghép chuỗi với separator
string csv = names.Aggregate((a, b) => $"{a},{b}");
// Tốt hơn: string.Join(",", names)
```

### GroupBy Chi Tiết

```csharp
var students = new List<Student>
{
    new("An", "A", 9.0), new("Bình", "B", 7.5),
    new("Châu", "A", 8.5), new("Dũng", "B", 8.0)
};

// GroupBy: nhóm theo tiêu chí
var groups = students
    .GroupBy(s => s.Grade)
    .Select(g => new
    {
        Grade = g.Key,           // Giá trị nhóm ("A", "B")
        Count = g.Count(),
        AvgScore = g.Average(s => s.Score),
        Students = g.ToList()
    });

foreach (var g in groups)
    Console.WriteLine($"Grade {g.Grade}: {g.Count} students, avg = {g.AvgScore:F1}");
```

### Join & GroupJoin

```csharp
// JOIN: similar SQL INNER JOIN
var result = students
    .Join(classes,
        student => student.ClassId,   // Key từ students
        cls => cls.Id,                 // Key từ classes
        (student, cls) => new         // Kết hợp
        {
            student.Name,
            ClassName = cls.Name
        });

// GROUPJOIN: similar SQL LEFT JOIN + GROUP
var classWithStudents = classes
    .GroupJoin(students,
        cls => cls.Id,
        student => student.ClassId,
        (cls, studs) => new
        {
            Class = cls.Name,
            Students = studs.ToList(),
            Count = studs.Count()
        });
```

### Query Syntax vs Method Syntax

```csharp
// Method Syntax (Lambda) ⭐ Phổ biến hơn trong C#
var result1 = students
    .Where(s => s.Age >= 18)
    .OrderBy(s => s.Name)
    .Select(s => s.Name);

// Query Syntax (SQL-like)
var result2 = from s in students
              where s.Age >= 18
              orderby s.Name
              select s.Name;

// Cả hai tạo ra KẾT QUẢ GIỐNG NHAU
// Method Syntax: linh hoạt hơn, có nhiều methods hơn
// Query Syntax: dễ đọc hơn cho JOIN, GROUP BY phức tạp
```

---

## ❓ Câu Hỏi Kiểm Tra Bổ Sung (Nâng Cao)

1. LINQ là gì? Viết tắt của cái gì?
2. Deferred execution (lazy) vs Immediate execution — khác nhau?
3. `FirstOrDefault` vs `First` — khi nào dùng cái nào?
4. `Select` vs `SelectMany` — khác nhau thế nào?
5. Method syntax vs Query syntax — cái nào nên dùng?

---

## 💼 Câu Hỏi Phỏng Vấn Bổ Sung (Nâng Cao)

> **Q: What is deferred execution in LINQ?**
> A: LINQ query KHÔNG chạy ngay khi khai báo. Chỉ chạy khi enumerate (foreach, ToList, Count...). Lợi ích: tiết kiệm memory, chỉ tính khi cần. `ToList()`, `ToArray()`, `Count()` trigger immediate execution.

> **Q: What is the difference between `IEnumerable` and `IQueryable`?**
> A: `IEnumerable<T>`: xử lý in-memory (LINQ to Objects). `IQueryable<T>`: xử lý remote (LINQ to SQL/EF) — query được translate sang SQL. IQueryable kế thừa IEnumerable. Dùng IQueryable cho database queries để tận dụng server-side filtering.

> **Q: What is the difference between `Select` and `SelectMany`?**
> A: `Select` map 1:1 (mỗi item → 1 result). `SelectMany` map 1:N và flatten (mỗi item → nhiều results, gộp thành 1 list). Ví dụ: `customers.SelectMany(c => c.Orders)` trả về tất cả orders từ tất cả customers.

---

## 🏋️ Bài Tập Bổ Sung (Nâng Cao)

**BT1**: Cho `List<string> words`, viết LINQ: tìm tất cả từ dài hơn 5 ký tự, uppercase, sorted alphabetically.

**BT2**: Cho `List<Student>` (Name, Grade, Score), viết LINQ: group by Grade, tính Average Score mỗi group.

**BT3**: Implement `Where`, `Select`, `Any` methods KHÔNG dùng LINQ (dùng yield return).

---

## 📚 Bổ Sung: Expression Trees

**Expression Tree** biểu diễn code dưới dạng **cấu trúc dữ liệu cây**, cho phép phân tích và biến đổi code tại runtime.

```csharp
using System.Linq.Expressions;

// Tạo expression tree: x => x * x
ParameterExpression x = Expression.Parameter(typeof(int), "x");
BinaryExpression body = Expression.Multiply(x, x);
Expression<Func<int, int>> square = Expression.Lambda<Func<int, int>>(body, x);

// Compile và chạy
Func<int, int> compiled = square.Compile();
Console.WriteLine(compiled(5)); // 25

// Phân tích cấu trúc
Console.WriteLine(square);           // x => (x * x)
Console.WriteLine(square.Body);      // (x * x)
Console.WriteLine(square.Body.NodeType); // Multiply
```

> 💡 **Ứng dụng thực tế:** EF Core dùng Expression Trees để chuyển LINQ queries thành SQL. Khi bạn viết `.Where(e => e.Age > 18)`, EF Core phân tích expression tree để tạo `WHERE Age > 18`.
---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p05-linq](./99-answer-key-csharp.md#p05-linq)
- Bài tập thực hành: [99-answer-key-csharp.md#p05-linq-exercises](./99-answer-key-csharp.md#p05-linq-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p05-linq-deep](./97-csharp-theory-deep-dive.md#p05-linq-deep)
