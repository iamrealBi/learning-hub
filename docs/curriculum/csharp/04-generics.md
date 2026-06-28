# 📙 Phần 4: Generic Types & Advanced Methods

> **Nội dung**: Generics, Constraints, Delegates, Lambda, Func/Action/Predicate  
> **Thời lượng ước tính**: 5–6 giờ

---

## 🎯 Mục Tiêu

- Hiểu Generic types và tại sao cần chúng
- Tạo generic class, method, và sử dụng type constraints
- Thành thạo Func, Action, Lambda expressions
- Dictionary và cách sử dụng
- Strategy design pattern & Decorator pattern

---

## 1. Generic Types

### 1.1 Vấn đề: Code lặp lại cho từng kiểu

```csharp
// ❌ Phải viết riêng cho mỗi kiểu
class IntList { private int[] _items; /* ... */ }
class StringList { private string[] _items; /* ... */ }
class DoubleList { private double[] _items; /* ... */ }

// ✅ Generic: viết MỘT LẦN, dùng cho MỌI kiểu
class MyList<T>
{
    private T[] _items;
    private int _count;
    
    public MyList(int capacity = 4)
    {
        _items = new T[capacity];
    }
    
    public void Add(T item)
    {
        if (_count >= _items.Length)
        {
            // Mở rộng mảng (giống cách List<T> hoạt động)
            T[] newArray = new T[_items.Length * 2];
            Array.Copy(_items, newArray, _count);
            _items = newArray;
        }
        _items[_count++] = item;
    }
    
    public T Get(int index) => _items[index];
    public int Count => _count;
}

// Sử dụng
MyList<int> numbers = new MyList<int>();
numbers.Add(42);

MyList<string> names = new MyList<string>();
names.Add("Nghĩa");
```

### 1.2 T là gì?

`T` là **Type Parameter** — một placeholder cho kiểu dữ liệu thực tế:
- `T` → Convention, viết tắt của "Type"
- `TKey, TValue` → Cho dictionary/map
- `TResult` → Cho kiểu trả về
- Có thể dùng tên bất kỳ: `<TAnimal>`, `<TElement>`

---

## 2. Generic Methods

```csharp
// Generic method: method dùng type parameter
static T Max<T>(T a, T b) where T : IComparable<T>
{
    return a.CompareTo(b) > 0 ? a : b;
}

static void Swap<T>(ref T a, ref T b)
{
    T temp = a;
    a = b;
    b = temp;
}

// Sử dụng
int maxInt = Max(5, 10);           // T = int (inferred)
string maxStr = Max("abc", "xyz"); // T = string

int x = 1, y = 2;
Swap(ref x, ref y);  // x=2, y=1
```

---

## 3. Type Constraints

```csharp
// where T : struct        → T phải là value type
// where T : class         → T phải là reference type
// where T : new()         → T phải có constructor không tham số
// where T : BaseClass     → T phải kế thừa từ BaseClass
// where T : IInterface    → T phải implement IInterface
// where T : IComparable<T>→ T phải so sánh được
// where T : notnull        → T không được null

class Repository<T> where T : class, new()
{
    public T Create()
    {
        return new T();  // Chỉ hợp lệ nhờ constraint new()
    }
}

// C# 11: Generic math (INumber<T>)
static T Sum<T>(T[] numbers) where T : System.Numerics.INumber<T>
{
    T total = T.Zero;
    foreach (T n in numbers)
        total += n;
    return total;
}
```

---

## 4. Tuples

```csharp
// Tuple: nhóm nhiều giá trị mà không cần tạo class
(string Name, int Age) person = ("Nghĩa", 25);
Console.WriteLine(person.Name);   // "Nghĩa"
Console.WriteLine(person.Age);    // 25

// Trả về nhiều giá trị từ method
static (int Min, int Max, double Average) GetStats(int[] numbers)
{
    return (numbers.Min(), numbers.Max(), numbers.Average());
}

var stats = GetStats(new[] { 1, 5, 3, 9, 2 });
Console.WriteLine($"Min={stats.Min}, Max={stats.Max}, Avg={stats.Average:F1}");

// Deconstruction
var (min, max, avg) = GetStats(new[] { 1, 5, 3, 9, 2 });
```

---

## 5. Func, Action, Lambda

### 5.1 Func và Action

```csharp
// Func<TResult>: hàm trả về giá trị
// Func<T1, TResult>: hàm nhận T1, trả về TResult
// Func<T1, T2, TResult>: hàm nhận T1+T2, trả về TResult

Func<int, int, int> add = (a, b) => a + b;
Func<string, bool> isLong = s => s.Length > 10;
Func<int> getRandom = () => new Random().Next();

int sum = add(3, 5);     // 8
bool long_ = isLong("Hi"); // false

// Action: hàm KHÔNG trả về giá trị (void)
Action<string> print = message => Console.WriteLine(message);
Action<string, int> repeat = (msg, times) =>
{
    for (int i = 0; i < times; i++)
        Console.WriteLine(msg);
};

print("Hello!");
repeat("Hi!", 3);
```

### 5.2 Lambda Expressions

```csharp
// Lambda: cách viết ngắn gọn cho anonymous method

// Expression lambda (1 biểu thức)
Func<int, int> square = x => x * x;

// Statement lambda (nhiều dòng)
Func<int, string> classify = n =>
{
    if (n > 0) return "Positive";
    if (n < 0) return "Negative";
    return "Zero";
};

// Lambda truyền vào method khác
List<int> numbers = new() { 5, 2, 8, 1, 9, 3 };
numbers.Sort((a, b) => a.CompareTo(b));  // Sắp xếp tăng dần
```

### 5.3 Delegates (nền tảng của Func/Action)

```csharp
// Delegate: kiểu dữ liệu đại diện cho method
delegate int MathOperation(int a, int b);

MathOperation op = (a, b) => a + b;
Console.WriteLine(op(3, 5));  // 8

op = (a, b) => a * b;         // Đổi sang phép nhân
Console.WriteLine(op(3, 5));  // 15

// Func/Action chính là generic delegates có sẵn trong .NET
// Func<int, int, int> ≡ delegate int Func(int, int)
```

---

## 6. Dictionary

```csharp
// Dictionary<TKey, TValue>: ánh xạ key → value
Dictionary<string, int> ages = new()
{
    ["Nghĩa"] = 25,
    ["An"] = 30,
    ["Bình"] = 22
};

// Thêm/sửa
ages["Chi"] = 28;        // Thêm mới
ages["Nghĩa"] = 26;      // Sửa giá trị

// Đọc
int nghiaAge = ages["Nghĩa"];  // 26

// Kiểm tra key tồn tại
if (ages.ContainsKey("An"))
    Console.WriteLine(ages["An"]);

// An toàn hơn với TryGetValue
if (ages.TryGetValue("Mai", out int maiAge))
    Console.WriteLine(maiAge);
else
    Console.WriteLine("Không tìm thấy Mai");

// Duyệt
foreach (KeyValuePair<string, int> pair in ages)
{
    Console.WriteLine($"{pair.Key}: {pair.Value} tuổi");
}

// Hoặc dùng deconstruction
foreach (var (name, age) in ages)
{
    Console.WriteLine($"{name}: {age} tuổi");
}
```

---

## 7. Strategy Design Pattern

```csharp
// Strategy: đóng gói thuật toán vào object, cho phép swap lúc runtime

// Thay vì if/else dài:
// ❌ Vi phạm Open-Closed Principle
class Sorter
{
    public void Sort(List<int> list, string algorithm)
    {
        if (algorithm == "bubble") { /* ... */ }
        else if (algorithm == "quick") { /* ... */ }
        else if (algorithm == "merge") { /* ... */ }
        // Thêm thuật toán mới → phải SỬA class này!
    }
}

// ✅ Strategy pattern: mỗi thuật toán là 1 object riêng
interface ISortStrategy<T>
{
    void Sort(List<T> list);
}

class BubbleSort<T> : ISortStrategy<T> where T : IComparable<T>
{
    public void Sort(List<T> list) { /* bubble sort */ }
}

class QuickSort<T> : ISortStrategy<T> where T : IComparable<T>
{
    public void Sort(List<T> list) { /* quick sort */ }
}

class Sorter<T> where T : IComparable<T>
{
    private readonly ISortStrategy<T> _strategy;
    
    public Sorter(ISortStrategy<T> strategy) => _strategy = strategy;
    
    public void Sort(List<T> list) => _strategy.Sort(list);
}

// Dùng Func thay vì interface (đơn giản hơn):
static List<T> Filter<T>(List<T> items, Func<T, bool> predicate)
{
    List<T> result = new();
    foreach (T item in items)
        if (predicate(item))
            result.Add(item);
    return result;
}

var adults = Filter(people, p => p.Age >= 18);
var vips = Filter(people, p => p.TotalSpent > 1000);
```

---

## 8. Open-Closed Principle (OCP)

> **Open for extension, Closed for modification.**  
> Mở rộng chức năng bằng cách THÊM code mới, KHÔNG SỬA code cũ.

```csharp
// ✅ OCP: Thêm strategy mới không cần sửa Sorter
class MergeSort<T> : ISortStrategy<T> where T : IComparable<T>
{
    public void Sort(List<T> list) { /* merge sort */ }
}
// Sorter không cần thay đổi gì!
```

---

## 9. Decorator Design Pattern

```csharp
// Decorator: Thêm behavior cho object mà không sửa class gốc

interface IDataSource
{
    string Read();
    void Write(string data);
}

class FileDataSource : IDataSource
{
    private readonly string _path;
    public FileDataSource(string path) => _path = path;
    public string Read() => File.ReadAllText(_path);
    public void Write(string data) => File.WriteAllText(_path, data);
}

// Decorator: thêm encryption
class EncryptionDecorator : IDataSource
{
    private readonly IDataSource _inner;
    public EncryptionDecorator(IDataSource inner) => _inner = inner;
    
    public string Read() => Decrypt(_inner.Read());
    public void Write(string data) => _inner.Write(Encrypt(data));
    
    private string Encrypt(string data) => Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(data));
    private string Decrypt(string data) => System.Text.Encoding.UTF8.GetString(Convert.FromBase64String(data));
}

// Decorator: thêm compression
class CompressionDecorator : IDataSource
{
    private readonly IDataSource _inner;
    public CompressionDecorator(IDataSource inner) => _inner = inner;
    
    public string Read() => Decompress(_inner.Read());
    public void Write(string data) => _inner.Write(Compress(data));
    
    // ...
}

// Composing decorators (xếp chồng):
IDataSource source = new FileDataSource("data.txt");
source = new EncryptionDecorator(source);    // Thêm mã hóa
source = new CompressionDecorator(source);   // Thêm nén

source.Write("Hello!");  // Nén → Mã hóa → Ghi file
string data = source.Read();  // Đọc file → Giải mã → Giải nén
```

---

## 10. Caching

```csharp
// Cache: lưu kết quả tính toán để tái sử dụng

class SimpleCache<TKey, TValue>
{
    private readonly Dictionary<TKey, TValue> _cache = new();
    
    public TValue GetOrAdd(TKey key, Func<TKey, TValue> factory)
    {
        if (!_cache.ContainsKey(key))
        {
            _cache[key] = factory(key);
        }
        return _cache[key];
    }
}

// Dùng:
var cache = new SimpleCache<string, int>();
int length = cache.GetOrAdd("hello", s => s.Length);  // Tính lần đầu
int same = cache.GetOrAdd("hello", s => s.Length);    // Lấy từ cache
```

---

## 🧪 Coding Exercises

| # | Bài | Kiến thức |
|---|-----|-----------|
| 31 | Generic types - Pair class | Tạo generic class |
| 32 | Generic methods - SwapTupleItems | Generic method |
| 33 | Type constraints & IComparable - SortedList | Constraints |
| 34 | Basics of Funcs and Actions | Func, Action |
| 35 | Basics of lambda expressions | Lambda |
| 36 | Dictionaries - FindMaxWeights | Dictionary |

---

## ❓ Câu Hỏi Kiểm Tra

1. Generic type `T` giải quyết vấn đề gì so với dùng `object`?
2. `where T : class, new()` nghĩa là gì?
3. `Func<int, string>` nhận gì, trả về gì?
4. `Action<string>` khác `Func<string>` thế nào?
5. Lambda expression là gì? Khác anonymous method thế nào?
6. Delegate là gì? Quan hệ với Func/Action?
7. Dictionary lookup có Big O bao nhiêu? Tại sao?
8. Strategy pattern giải quyết vấn đề gì?
9. Decorator pattern khác inheritance thế nào?
10. OCP (Open-Closed Principle) nghĩa là gì? Cho ví dụ.

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between `Func` and `Action`?**
> A: `Func<T, TResult>` có return value — tham số cuối là return type. `Action<T>` KHÔNG có return value (void). Cả hai là delegate types. `Predicate<T>` tương đương `Func<T, bool>`.

> **Q: What are generic constraints? Name the common types.**
> A: `where T : struct` (value type), `where T : class` (reference type), `where T : new()` (parameterless constructor), `where T : IComparable` (implement interface), `where T : BaseClass` (inherit). Có thể kết hợp nhiều constraints.

> **Q: What is covariance and contravariance?**
> A: **Covariance** (`out T`): dùng derived type thay base type (IEnumerable<Dog> → IEnumerable<Animal>). **Contravariance** (`in T`): ngược lại (Action<Animal> → Action<Dog>). Chỉ cho interfaces và delegates.

> **Q: Explain the Strategy Pattern.**
> A: Định nghĩa family of algorithms, đóng gói mỗi cái vào class riêng qua interface, cho phép swap runtime. Tuân thủ OCP — thêm algorithm mới không sửa code cũ. Ví dụ: ISortStrategy với BubbleSort, QuickSort.

> **Q: Explain the Decorator Pattern.**
> A: Thêm behavior cho object bằng cách "bọc" nó trong decorator class, cả 2 implement cùng interface. Có thể xếp chồng nhiều decorators. Linh hoạt hơn inheritance vì compose được runtime.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Generic method `T FindMax<T>(List<T> items) where T : IComparable<T>`.

**BT2**: Generic class `Pair<T1, T2>` lưu 2 giá trị. Method `Swap()` trả về `Pair<T2, T1>`.

**BT3**: Implement `Pipeline<T>` nhận `List<Func<T, T>>`, method `Execute(T input)` chạy qua tất cả functions tuần tự.

**BT4**: Implement word frequency counter dùng Dictionary — nhập chuỗi, đếm số lần xuất hiện mỗi từ.

---

## 📚 Bổ Sung: Covariance & Contravariance Code Examples

```csharp
// === COVARIANCE (out T) — derived → base ===
// IEnumerable<out T> là covariant

class Animal { public string Name { get; set; } = ""; }
class Dog : Animal { public string Breed { get; set; } = ""; }

IEnumerable<Dog> dogs = new List<Dog>
{
    new Dog { Name = "Rex", Breed = "German Shepherd" },
    new Dog { Name = "Max", Breed = "Golden Retriever" }
};

// ✅ Covariance: IEnumerable<Dog> → IEnumerable<Animal>
IEnumerable<Animal> animals = dogs;  // OK! Dog IS-A Animal
foreach (Animal a in animals)
    Console.WriteLine(a.Name);

// ❌ KHÔNG thể ngược lại:
// IEnumerable<Animal> animals2 = new List<Animal>();
// IEnumerable<Dog> dogs2 = animals2;  // Compile error!

// === CONTRAVARIANCE (in T) — base → derived ===
// Action<in T> là contravariant

Action<Animal> printAnimal = a => Console.WriteLine($"Animal: {a.Name}");

// ✅ Contravariance: Action<Animal> → Action<Dog>
Action<Dog> printDog = printAnimal;  // OK! Xử lý Animal thì xử lý Dog cũng được
printDog(new Dog { Name = "Rex" });  // "Animal: Rex"

// === Custom covariant interface ===
interface IProducer<out T>  // out = chỉ OUTPUT T
{
    T Produce();
    // void Consume(T item);  // ❌ Không thể dùng T ở input position!
}

interface IConsumer<in T>  // in = chỉ INPUT T
{
    void Consume(T item);
    // T Get();  // ❌ Không thể dùng T ở output position!
}
```

### `default` Keyword

```csharp
// default trả về giá trị mặc định của type
int num = default;        // 0
bool flag = default;      // false
string? str = default;    // null
DateTime dt = default;    // 0001-01-01

// Trong generic methods:
T GetDefaultValue<T>()
{
    return default(T)!;   // hoặc default! (C# 7.1+)
}

// Dùng trong switch (C# 7+):
string Describe<T>(T value) => value switch
{
    null => "null",
    0 => "zero",
    "" => "empty string",
    _ => value.ToString() ?? "unknown"
};
```

---

## ❓ Câu Hỏi Kiểm Tra Bổ Sung (Nâng Cao)

1. Generic type `T` giải quyết vấn đề gì so với dùng `object`?
2. Generic constraint `where T : class` nghĩa là gì?
3. `Func<int, string>` nghĩa là gì? Nhận và trả về gì?
4. `Action<string>` khác `Func<string>` thế nào?
5. Lambda expression là gì? Cho 3 ví dụ.

---

## 💼 Câu Hỏi Phỏng Vấn Bổ Sung (Nâng Cao)

> **Q: What is the difference between `Func` and `Action`?**
> A: `Func<T, TResult>` có return value (tham số cuối là return type). `Action<T>` KHÔNG có return value (void). Cả hai là delegate types. `Predicate<T>` là `Func<T, bool>` chuyên biệt.

> **Q: What are generic constraints? Name the types.**
> A: `where T : struct` (value type), `where T : class` (reference type), `where T : new()` (parameterless constructor), `where T : IComparable` (implement interface), `where T : BaseClass` (inherit from class), `where T : notnull` (non-nullable).

> **Q: What is covariance and contravariance?**
> A: **Covariance** (`out T`): cho phép dùng derived type thay base type (IEnumerable<Dog> → IEnumerable<Animal>). **Contravariance** (`in T`): ngược lại (Action<Animal> → Action<Dog>). Chỉ áp dụng cho interfaces và delegates.

> **Q: Explain the Strategy Pattern.**
> A: Định nghĩa family of algorithms, đóng gói mỗi cái vào class riêng, và làm cho chúng có thể thay thế nhau. Client chọn strategy lúc runtime qua DI. Ví dụ: ISortingStrategy với BubbleSort, QuickSort implementations.

---

## 🏋️ Bài Tập Bổ Sung (Nâng Cao)

**BT1**: Generic method `T FindMax<T>(List<T> items) where T : IComparable<T>` — tìm phần tử lớn nhất.

**BT2**: Generic class `Pair<T1, T2>` lưu 2 giá trị. Method `Swap()` đổi chỗ.

**BT3**: Implement `Pipeline<T>` nhận List<Func<T, T>>, method `Execute(T input)` chạy qua tất cả functions tuần tự.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p04-generics](./99-answer-key-csharp.md#p04-generics)
- Bài tập thực hành: [99-answer-key-csharp.md#p04-generics-exercises](./99-answer-key-csharp.md#p04-generics-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p04-generics-deep](./97-csharp-theory-deep-dive.md#p04-generics-deep)

