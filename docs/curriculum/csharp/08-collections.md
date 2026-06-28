# 📒 Phần 8: Collections

> **Nội dung**: IEnumerable, Dictionary, HashSet, yield, Big O, custom collections  
> **Thời lượng ước tính**: 5–6 giờ

---

## 🎯 Mục Tiêu

- Hiểu IEnumerable, ICollection, IList interfaces
- Implement IEnumerable<T> cho custom collection
- Indexers, Readonly collections
- Big O Notation và hiệu suất các cấu trúc dữ liệu
- HashSet, Queue, Stack
- yield keyword và iterators
- Interface Segregation Principle

---

## 1. IEnumerable<T> — Nền Tảng

### 1.1 Vai trò

```csharp
// IEnumerable<T>: cho phép duyệt qua collection bằng foreach
// MỌI collection trong .NET đều implement IEnumerable<T>

public interface IEnumerable<T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}

public interface IEnumerator<T>
{
    T Current { get; }        // Phần tử hiện tại
    bool MoveNext();          // Chuyển sang phần tử tiếp theo
    void Reset();             // Quay về đầu
}
```

### 1.2 Implement IEnumerable<T>

```csharp
class NumberRange : IEnumerable<int>
{
    private readonly int _start;
    private readonly int _end;
    
    public NumberRange(int start, int end) => (_start, _end) = (start, end);
    
    public IEnumerator<int> GetEnumerator()
    {
        for (int i = _start; i <= _end; i++)
            yield return i;  // Trả về từng phần tử
    }
    
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

// Sử dụng
var range = new NumberRange(1, 10);
foreach (int n in range)
    Console.Write($"{n} ");  // 1 2 3 4 5 6 7 8 9 10

// Dùng LINQ trên custom collection!
var evens = range.Where(n => n % 2 == 0);
```

---

## 2. Collection Interfaces Hierarchy

```
IEnumerable<T>        ← foreach
    └─ ICollection<T> ← Count, Add, Remove, Contains
        └─ IList<T>   ← indexer [i], Insert, RemoveAt
```

### 2.1 Implicit vs Explicit Interface Implementation

```csharp
interface IAnimal { void Speak(); }
interface IRobot { void Speak(); }

class Hybrid : IAnimal, IRobot
{
    // Implicit: dùng cho cả hai
    public void Speak() => Console.WriteLine("I'm both!");
    
    // Explicit: chỉ truy cập qua interface
    void IAnimal.Speak() => Console.WriteLine("Animal speaks");
    void IRobot.Speak() => Console.WriteLine("Robot speaks");
}

Hybrid h = new();
h.Speak();                    // "I'm both!" (implicit)
((IAnimal)h).Speak();         // "Animal speaks" (explicit)
((IRobot)h).Speak();          // "Robot speaks" (explicit)
```

---

## 3. Indexers

```csharp
class Sentence
{
    private readonly string[] _words;
    
    public Sentence(string text) => _words = text.Split(' ');
    
    // Indexer: truy cập object như array
    public string this[int index]
    {
        get => _words[index];
        set => _words[index] = value;
    }
    
    public int Length => _words.Length;
    public override string ToString() => string.Join(' ', _words);
}

var sentence = new Sentence("Hello World From CSharp");
Console.WriteLine(sentence[0]);  // "Hello"
sentence[1] = "Beautiful";
Console.WriteLine(sentence);     // "Hello Beautiful From CSharp"
```

---

## 4. Readonly Collections

```csharp
List<int> list = new() { 1, 2, 3 };

// ReadOnlyCollection: không thể Add/Remove/Set
IReadOnlyList<int> readOnly = list.AsReadOnly();
// readOnly.Add(4);  // ❌ Lỗi compile!
// readOnly[0] = 10; // ❌ Lỗi compile!

// ⚠️ Nhưng sửa list gốc VẪN ảnh hưởng readOnly!
list.Add(4);
Console.WriteLine(readOnly.Count); // 4

// ImmutableList (System.Collections.Immutable): hoàn toàn bất biến
```

---

## 5. Interface Segregation Principle (ISP)

> **Client không nên bị buộc phụ thuộc vào methods mà nó không sử dụng.**

```csharp
// ❌ Vi phạm ISP: interface quá lớn
interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

class Robot : IWorker
{
    public void Work() { /* OK */ }
    public void Eat() { /* Không hợp lý cho robot! */ }
    public void Sleep() { /* Không hợp lý! */ }
}

// ✅ Tuân thủ ISP: tách interface nhỏ
interface IWorkable { void Work(); }
interface IFeedable { void Eat(); }
interface ISleepable { void Sleep(); }

class Human : IWorkable, IFeedable, ISleepable { /* Implement tất cả */ }
class Robot : IWorkable { /* Chỉ implement Work */ }
```

---

## 6. Big O Notation

```
Complexity      Name           Example
─────────────────────────────────────────────
O(1)            Constant       Dictionary lookup
O(log n)        Logarithmic    Binary search
O(n)            Linear         Linear search, List.Contains
O(n log n)      Linearithmic   Sorting (QuickSort, MergeSort)
O(n²)           Quadratic      Nested loops, Bubble sort
O(2ⁿ)           Exponential    Brute force, subset generation
```

### Performance Comparison

| Operation | Array/List | LinkedList | Dictionary | HashSet |
|-----------|-----------|------------|------------|---------|
| Access by index | **O(1)** | O(n) | — | — |
| Search | O(n) | O(n) | **O(1)** | **O(1)** |
| Insert at end | **O(1)** amortized | **O(1)** | **O(1)** | **O(1)** |
| Insert at start | O(n) | **O(1)** | — | — |
| Remove | O(n) | **O(1)** | **O(1)** | **O(1)** |

---

## 7. Data Structures

### 7.1 HashSet

```csharp
// HashSet: collection KHÔNG trùng lặp, lookup O(1)
HashSet<string> tags = new() { "csharp", "dotnet", "programming" };

tags.Add("csharp");       // Không thêm (đã có)
tags.Add("linq");          // Thêm thành công

bool has = tags.Contains("dotnet");  // O(1)!

// Set operations
HashSet<int> a = new() { 1, 2, 3, 4 };
HashSet<int> b = new() { 3, 4, 5, 6 };

a.UnionWith(b);       // a = {1, 2, 3, 4, 5, 6}
a.IntersectWith(b);   // a = {3, 4}
a.ExceptWith(b);      // a = {1, 2}
```

### 7.2 Queue (Hàng đợi - FIFO)

```csharp
// FIFO: First In, First Out
Queue<string> queue = new();

queue.Enqueue("Task A");  // Thêm vào cuối
queue.Enqueue("Task B");
queue.Enqueue("Task C");

string next = queue.Dequeue();  // "Task A" (lấy từ đầu)
string peek = queue.Peek();     // "Task B" (xem nhưng không lấy)
```

### 7.3 Stack (Ngăn xếp - LIFO)

```csharp
// LIFO: Last In, First Out
Stack<string> history = new();

history.Push("Page 1");   // Thêm lên đỉnh
history.Push("Page 2");
history.Push("Page 3");

string back = history.Pop();   // "Page 3" (lấy từ đỉnh)
string top = history.Peek();   // "Page 2" (xem đỉnh)
```

### 7.4 Linked List

```csharp
LinkedList<string> list = new();

var node = list.AddLast("B");
list.AddFirst("A");
list.AddAfter(node, "C");
// A → B → C

list.Remove("B");
// A → C
```

---

## 8. "params" Keyword

```csharp
// params: nhận số lượng arguments không xác định
static int Sum(params int[] numbers)
{
    int total = 0;
    foreach (int n in numbers)
        total += n;
    return total;
}

int result = Sum(1, 2, 3);           // 6
int result2 = Sum(1, 2, 3, 4, 5);   // 15
int result3 = Sum();                  // 0
```

---

## 9. yield Statement & Iterators

### 9.1 yield return

```csharp
// yield return: trả về từng phần tử MỘT CÁCH LƯỜI BIẾNG (lazy)
static IEnumerable<int> EvenNumbers(int max)
{
    for (int i = 0; i <= max; i += 2)
    {
        yield return i;  // Trả về i, TẠM DỪNG tại đây
        // Lần tiếp theo gọi MoveNext(), tiếp tục từ đây
    }
}

// Chỉ tính toán KHI CẦN!
foreach (int n in EvenNumbers(1000000))
{
    if (n > 10) break;  // Chỉ tính đến 10, KHÔNG tính 1 triệu số
}
```

### 9.2 yield break

```csharp
static IEnumerable<string> ReadUntilEmpty()
{
    while (true)
    {
        string line = Console.ReadLine();
        if (string.IsNullOrEmpty(line))
            yield break;  // Kết thúc iterator
        yield return line;
    }
}
```

### 9.3 Lợi ích của yield

```
1. Lazy evaluation: chỉ tính khi cần
2. Memory efficient: không tạo collection trung gian
3. Infinite sequences: có thể tạo dãy vô hạn
4. Pipeline: kết hợp với LINQ rất hiệu quả
```

```csharp
// Fibonacci vô hạn
static IEnumerable<long> Fibonacci()
{
    long a = 0, b = 1;
    while (true)
    {
        yield return a;
        (a, b) = (b, a + b);
    }
}

// Lấy 10 số Fibonacci đầu tiên
var first10 = Fibonacci().Take(10);  // 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```

---

## 🧪 Coding Exercises

| # | Bài | Kiến thức |
|---|-----|-----------|
| 50 | Custom indexer - PairOfArrays | Indexer, ValueTuples |
| 51 | HashSet - CreateUnion | HashSet operations |
| 52 | params - StackContainsAny | params keyword |
| 53 | yield - GetAllAfterLastNullReversed | yield statement |

---

## ❓ Câu Hỏi Kiểm Tra

1. `IEnumerable<T>` vs `ICollection<T>` vs `IList<T>` — mỗi cái thêm gì?
2. `yield return` hoạt động thế nào bên trong?
3. `HashSet<T>` khác `List<T>` thế nào? Khi nào dùng?
4. Queue vs Stack — FIFO vs LIFO nghĩa là gì?
5. Indexer là gì? Khai báo bằng keyword gì?
6. Big O: `Dictionary.ContainsKey` vs `List.Contains` — cái nào nhanh hơn?
7. `ReadOnlyCollection` có thực sự immutable không? Tại sao?
8. `params` keyword dùng khi nào?
9. ISP (Interface Segregation Principle) nghĩa là gì?
10. `yield break` vs `return` trong iterator — khác nhau?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between `IEnumerable` and `ICollection`?**
> A: `IEnumerable<T>` chỉ có `GetEnumerator()` — cho phép foreach. `ICollection<T>` thêm `Count`, `Add`, `Remove`, `Contains`. `IList<T>` thêm indexer `[i]`, `Insert`, `RemoveAt`. Chọn interface NHỎ NHẤT đủ dùng (ISP).

> **Q: What is `yield return` and how does it work?**
> A: `yield return` tạo state machine — method "tạm dừng" và trả về 1 giá trị, tiếp tục từ chỗ dừng khi `MoveNext()` gọi. Lazy evaluation: chỉ tính khi cần. Không tạo collection trung gian → tiết kiệm memory.

> **Q: When would you use HashSet over List?**
> A: Khi cần: (1) đảm bảo phần tử unique, (2) lookup nhanh O(1) thay vì O(n), (3) set operations (Union, Intersect, Except). Trade-off: không có index, không có thứ tự.

> **Q: Explain Big O notation with examples.**
> A: Đo performance theo kích thước input. O(1): Dictionary lookup. O(log n): binary search. O(n): linear search. O(n log n): sorting. O(n²): nested loops. Chọn algorithm/collection phù hợp dựa trên Big O.

> **Q: What is the Interface Segregation Principle?**
> A: Client không nên bị buộc phụ thuộc methods không dùng. Thay vì 1 interface lớn → tách thành nhiều interface nhỏ, focused. Ví dụ: `IWorkable`, `IFeedable` thay vì `IWorker` có cả `Work()`, `Eat()`, `Sleep()`.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Implement `CustomStack<T>` dùng array bên trong, với methods: Push, Pop, Peek, Count, IsEmpty.

**BT2**: Viết extension method `IEnumerable<T>.ForEach(Action<T>)` dùng yield.

**BT3**: Implement `LRU Cache<TKey, TValue>` (Least Recently Used) — capacity giới hạn, tự xóa phần tử ít dùng nhất.

**BT4**: Dùng HashSet để tìm phần tử trùng lặp đầu tiên trong mảng.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p08-collections](./99-answer-key-csharp.md#p08-collections)
- Bài tập thực hành: [99-answer-key-csharp.md#p08-collections-exercises](./99-answer-key-csharp.md#p08-collections-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p08-collections-deep](./97-csharp-theory-deep-dive.md#p08-collections-deep)

