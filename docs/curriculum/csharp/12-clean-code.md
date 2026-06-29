# 📘 Phần 12: Clean Code

> **Nội dung**: Naming, SOLID, DRY, code smells, refactoring, composition  
> **Thời lượng ước tính**: 5–6 giờ

---

## 🎯 Mục Tiêu

- Hiểu tầm quan trọng của clean code
- Đặt tên biến/method/class đúng chuẩn
- Viết method ngắn gọn, đúng trách nhiệm
- Viết comment đúng cách
- Composition over Inheritance
- Khi nào dùng static methods

---

## 1. Tầm Quan Trọng

```
Code is read MORE than it is written.
→ Code chủ yếu được ĐỌC chứ không phải viết.
→ Clean code = đầu tư cho tương lai.

Tech Debt (Nợ kỹ thuật):

- Code xấu = "vay nợ" — nhanh bây giờ, trả lãi mãi sau
- Càng nhiều nợ → càng chậm phát triển
- Fix bug → tạo bug mới → vòng xoáy
```

---

## 2. Naming — Đặt Tên

### 2.1 Nguyên tắc

```csharp
// ✅ GOOD: Tên có ý nghĩa, tự giải thích
int daysSinceLastLogin = 30;
bool isEligibleForDiscount = true;
List<Customer> activeCustomers = GetActiveCustomers();

// ❌ BAD: Tên vô nghĩa
int d = 30;          // d là gì?
bool flag = true;    // flag gì?
var list = GetData(); // data gì?

// ❌ BAD: Viết tắt
int numCst = 5;        // → numberOfCustomers
string addr = "HN";   // → address
bool chk = true;       // → isChecked
```

### 2.2 Conventions

| Loại | Convention | Ví dụ |
|------|-----------|-------|
| Local variable | camelCase | `customerAge` |
| Method parameter | camelCase | `firstName` |
| Private field | _camelCase | `_connectionString` |
| Public property | PascalCase | `FirstName` |
| Method | PascalCase | `CalculateTotal()` |
| Class / Struct | PascalCase | `CustomerService` |
| Interface | IPascalCase | `IRepository` |
| Constant | PascalCase | `MaxRetryCount` |
| Enum | PascalCase | `OrderStatus` |

### 2.3 Tên xấu thường gặp

```csharp
// ❌ Meaningless words
class DataManager { }    // "Manager" → quá chung
class ProcessorHelper { } // "Helper" → quá chung
class InfoHandler { }     // "Handler", "Info" → vô nghĩa

// ❌ Overspecific
int numberOfCustomersWhoPlacedOrdersInTheLast30Days;
// → recentOrderCustomerCount

// ❌ Hungarian notation (lỗi thời)
string strName;     // → name
int intAge;         // → age
bool bIsValid;      // → isValid

// ❌ Confusing
Thread thread;      // Biến thread trong Thread class → nhầm!
int count2;         // count1 ở đâu?
```

### 2.4 The Boy Scout Rule

> "Leave the code cleaner than you found it."
> Mỗi lần đọc code, cải thiện 1 tên biến/method. Từ từ code sẽ sạch hơn.

---

## 3. Methods — Phương Thức

### 3.1 Thông tin phương thức tốt

```csharp
// ✅ Ít parameters (tối đa 3, lý tưởng 0-2)
void SendEmail(string to, string subject, string body);

// ❌ Quá nhiều parameters
void CreateUser(string name, int age, string email, string phone, 
                string address, string city, string country, bool isActive);

// ✅ Giải pháp: Bundle parameters vào object
void CreateUser(UserCreationRequest request);
```

### 3.2 Tránh boolean parameters

```csharp
// ❌ Boolean parameter che giấu ý nghĩa
void Print(string text, bool uppercase);
Print("hello", true);   // true là gì?

// ✅ Tách thành 2 methods rõ ràng
void Print(string text);
void PrintUpperCase(string text);

// Hoặc dùng enum
enum PrintFormat { Normal, UpperCase, LowerCase }
void Print(string text, PrintFormat format);
```

### 3.3 Small methods, one job

```csharp
// ❌ Method làm quá nhiều việc
void ProcessOrder(Order order)
{
    // Validate order (10 dòng)
    // Calculate total (15 dòng)
    // Apply discount (10 dòng)
    // Save to database (5 dòng)
    // Send confirmation email (10 dòng)
    // Update inventory (8 dòng)
    // Log the action (5 dòng)
}

// ✅ Mỗi method = 1 job
void ProcessOrder(Order order)
{
    ValidateOrder(order);
    decimal total = CalculateTotal(order);
    total = ApplyDiscount(total, order.Customer);
    SaveOrder(order, total);
    SendConfirmation(order);
    UpdateInventory(order.Items);
    LogOrderProcessed(order);
}

// Mỗi method private ở trên = 5-15 dòng, rõ ràng, dễ test
```

### 3.4 Levels of Abstraction

```csharp
// ✅ Cùng level abstraction trong 1 method
void MakeCoffee()
{
    BoilWater();
    GrindBeans();
    PourWater();
    AddMilk();
}

// ❌ Mix levels
void MakeCoffee()
{
    BoilWater();                    // High-level
    beans = File.ReadAllText(".."); // Low-level detail!
    PourWater();                    // High-level
    milk.Temperature = 65;          // Low-level detail!
}
```

---

## 4. Comments

### 4.1 Worst Comments (TRÁNH)

```csharp
// ❌ Paraphrasing code
int age = 25; // Set age to 25

// ❌ Redundant
/// <summary>
/// Gets the name
/// </summary>
public string Name { get; set; }

// ❌ Commented-out code (dùng Git!)
// var oldResult = CalculateOld(data);
// if (oldResult > 100) { ... }

// ❌ Closing brace comments
if (condition)
{
    // ...
} // end if  ← Nếu cần comment này, method quá dài!

// ❌ TODO without action (vĩnh viễn)
// TODO: fix this later
```

### 4.2 Good Comments (NÊN)

```csharp
// ✅ WHY, not WHAT
// Using retry because the API is flaky during peak hours
await RetryAsync(3, () => apiClient.SendAsync(request));

// ✅ WARNING
// ⚠️ This method is NOT thread-safe. Use lock if calling from multiple threads.
public void UpdateCache(string key, object value)

// ✅ Legal / License
// Copyright (c) 2026 Company. All rights reserved.

// ✅ Regex explanation
// Matches: "2026-02-21" (YYYY-MM-DD format)
var datePattern = @"\d{4}-\d{2}-\d{2}";

// ✅ XML docs for PUBLIC API
/// <summary>
/// Calculates compound interest over a period.
/// </summary>
/// <param name="principal">Initial investment amount.</param>
/// <param name="rate">Annual interest rate (e.g., 0.05 for 5%).</param>
/// <param name="years">Number of years.</param>
/// <returns>Total amount after the period.</returns>
public decimal CalculateCompoundInterest(decimal principal, double rate, int years)
```

---

## 5. Static Methods — Khi Nào Dùng

```csharp
// ✅ Static: pure functions (output chỉ phụ thuộc input)
static class MathUtils
{
    public static double ToRadians(double degrees) => degrees * Math.PI / 180;
    public static int Factorial(int n) => n <= 1 ? 1 : n * Factorial(n - 1);
}

// ❌ Static public methods → khó test, khó mock
public static class EmailService
{
    public static void Send(string to, string body)
    {
        // Code gửi email thật → không thể mock trong tests!
    }
}

// ✅ Private static methods: OK (implementation detail)
class OrderProcessor
{
    public decimal CalculateTotal(List<Item> items)
    {
        return items.Sum(i => CalculateItemPrice(i));
    }
    
    private static decimal CalculateItemPrice(Item item) =>
        item.Price * item.Quantity * (1 - item.Discount);
}
```

---

## 6. Composition over Inheritance

```csharp
// ❌ Inheritance: cứng nhắc, khó thay đổi
class Animal { public virtual void Move() { } }
class Bird : Animal { public override void Move() { /* fly */ } }
class Penguin : Bird { public override void Move() { /* walk! */ } }
// Penguin IS A Bird nhưng KHÔNG bay → thiết kế sai!

// ✅ Composition: linh hoạt, dễ thay đổi
interface IMovement { void Move(); }
class FlyMovement : IMovement { public void Move() { /* fly */ } }
class WalkMovement : IMovement { public void Move() { /* walk */ } }
class SwimMovement : IMovement { public void Move() { /* swim */ } }

class Animal
{
    private readonly IMovement _movement;
    public Animal(IMovement movement) => _movement = movement;
    public void Move() => _movement.Move();
}

var eagle = new Animal(new FlyMovement());
var penguin = new Animal(new WalkMovement());  // ✅ Dễ dàng!
var duck = new Animal(new SwimMovement());

// Thậm chí có thể đổi behavior lúc runtime!
```

### Lợi ích:

| | Inheritance | Composition |
|---|-----------|-------------|
| Coupling | Tight (chặt) | **Loose (lỏng)** |
| Flexibility | Cố định lúc compile | **Đổi lúc runtime** |
| Testing | Khó mock base class | **Dễ mock interface** |
| Single Responsibility | Thường vi phạm | **Tuân thủ** |

---

## 7. Principle of Least Surprise

> Code nên hoạt động **đúng như tên gọi gợi ý**, không làm điều bất ngờ.

```csharp
// ❌ Surprise! Method tên "Get" lại xóa data
public List<Order> GetOrders()
{
    var orders = _db.FetchAll();
    _db.DeleteAll();  // ⚠️ Side effect bất ngờ!
    return orders;
}

// ✅ Predictable
public List<Order> GetOrders() => _db.FetchAll();
public void ClearOrders() => _db.DeleteAll();
```

---

## ❓ Câu Hỏi Kiểm Tra

1. "Code được ĐỌC nhiều hơn VIẾT" — ý nghĩa thực tế?
2. Tech debt là gì? Hậu quả?
3. Naming convention C#: PascalCase dùng cho gì? camelCase?
4. Tại sao tránh boolean parameters?
5. "Method nên làm 1 việc" — SRP cho methods nghĩa gì?
6. Levels of Abstraction — tại sao trộn lẫn là sai?
7. Comment tốt vs comment xấu — cho ví dụ mỗi loại.
8. Static methods — khi nào OK, khi nào KHÔNG?
9. Composition over Inheritance — tại sao?
10. Boy Scout Rule là gì?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is Clean Code?**
> A: Code dễ đọc, dễ hiểu, dễ thay đổi. Đặt tên rõ ràng, methods ngắn gọn, ít dependencies. "Clean code reads like well-written prose" (Robert C. Martin). Đầu tư clean code = tiết kiệm thời gian lâu dài.

> **Q: What is Composition over Inheritance?**
> A: Ưu tiên compose behavior qua interfaces thay vì kế thừa class. Inheritance = tight coupling, cố định compile-time. Composition = loose coupling, đổi runtime. Ví dụ: `IMovement` interface thay vì Bird → Penguin hierarchy.

> **Q: What is Technical Debt?**
> A: "Nợ" do code xấu — nhanh hôm nay nhưng trả giá mai sau. Symptoms: hard to change, bugs khi sửa, slow development. Giảm bằng: refactoring, code review, testing, clean code practices.

> **Q: What are SOLID principles?**
> A: **S**RP: 1 class = 1 trách nhiệm. **O**CP: mở rộng, không sửa. **L**SP: derived class thay thế base. **I**SP: interface nhỏ, focused. **D**IP: depend on abstractions. Tất cả nhằm tạo code maintainable.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Refactor code xấu: method 100 dòng ProcessOrder → tách thành 5-7 methods nhỏ, đặt tên rõ ràng.

**BT2**: Refactor inheritance hierarchy → composition: Animal > Bird > Penguin thành interface-based design.

**BT3**: Review code: tìm 10 code smells trong sample code (đặt tên xấu, methods dài, comments thừa).

**BT4**: Refactor class theo SRP:

```csharp
class UserManager
{
    public void Register(string email, string pwd) { /* validate, hash, save DB, send email */ }
    public string GenerateReport() { /* query all users, format PDF */ }
    public void BackupDatabase() { /* dump DB to file */ }
}
```

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p12-clean-code](./99-answer-key-csharp.md#p12-clean-code)
- Bài tập thực hành: [99-answer-key-csharp.md#p12-clean-code-exercises](./99-answer-key-csharp.md#p12-clean-code-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p12-clean-code-deep](./97-csharp-theory-deep-dive.md#p12-clean-code-deep)

---

## 8. SOLID Deep Dive — Ví Dụ Chi Tiết

### 8.1 Liskov Substitution Principle (LSP)

**Nguyên tắc:** Mọi derived class phải có thể **thay thế** base class mà không phá vỡ logic.

```csharp
// ✅ TUÂN THỦ LSP
void FeedAnimal(Animal animal)
{
    animal.MakeSound();  // Hoạt động đúng dù là Dog, Cat, hay bất kỳ Animal nào
}

// ❌ VI PHẠM LSP
public class Bird : Animal
{
    public override void MakeSound()
    {
        throw new NotSupportedException("Loài chim này không biết kêu");
        // → vi phạm LSP! Người gọi không mong đợi exception!
    }
}
```

### 8.2 Interface Segregation Principle (ISP)

**Nguyên tắc:** Client KHÔNG nên bị ép phụ thuộc vào interface mà nó KHÔNG sử dụng.

```csharp
// ❌ TỆ — God Interface
public interface IDocumentService
{
    void ReadDocument(string path);
    void WriteDocument(string path, string content);
    void ValidateDocument(string path);
    void PrintDocument(string path);
    void EmailDocument(string path, string recipient);
}

// ✅ TỐT — Interfaces nhỏ, tập trung
public interface IDocumentReader { string ReadDocument(string path); }
public interface IDocumentWriter { void WriteDocument(string path, string content); }
public interface IDocumentValidator { bool ValidateDocument(string path); }

// Class chỉ implement những gì nó CẦN
public class DocumentViewer : IDocumentReader { /* chỉ đọc */ }
public class DocumentEditor : IDocumentReader, IDocumentWriter, IDocumentValidator { /* đọc + ghi + validate */ }
```

### 8.3 Dependency Inversion Principle (DIP)

**Nguyên tắc:** Module cấp cao KHÔNG phụ thuộc vào module cấp thấp. Cả hai phụ thuộc vào **abstractions**.

```csharp
// ❌ TỆ — Gắn cứng vào implementations
public class CustomerService
{
    private SqlConnection _connection;    // Gắn cứng SQL Server!
    private SmtpClient _emailClient;      // Gắn cứng SMTP!
}

// ✅ TỐT — Phụ thuộc vào abstractions
public interface ICustomerRepository
{
    Task<Customer> GetByIdAsync(int id);
    Task SaveAsync(Customer customer);
}

public class CustomerService
{
    private readonly ICustomerRepository _repository;
    private readonly INotificationService _notifications;

    public CustomerService(ICustomerRepository repository,
                          INotificationService notifications)
    {
        _repository = repository;
        _notifications = notifications;
    }
}
```

> 💡 **Lợi ích:** Dễ test (mock interfaces), dễ swap implementation (SQL → MongoDB), business logic sạch.

**Ví dụ thực tế:** Công ty chuyển từ Oracle → PostgreSQL trong 3 tháng. Không có abstraction → sửa hàng ngàn class → 12+ tháng. Có DIP/Interface → chỉ swap implementation → hoàn thành **vài ngày**.

---

## 9. Design Patterns Cơ Bản

### 9.1 Singleton — Chỉ Một Instance

```csharp
public class Database
{
    private static Database _instance;
    private Database() { } // Private constructor!

    public static Database GetInstance()
    {
        if (_instance == null) _instance = new Database();
        return _instance;
    }
}

// Thread-safe version:
private static readonly Lazy<Database> _lazy =
    new Lazy<Database>(() => new Database());
public static Database Instance => _lazy.Value;
```

### 9.2 Factory — Tạo Object Có Tổ Chức

```csharp
public interface INotification { void Send(string message); }

public class NotificationFactory
{
    public static INotification CreateNotification(string channel)
    {
        return channel.ToLower() switch
        {
            "email" => new EmailNotification(),
            "sms"   => new SmsNotification(),
            "push"  => new PushNotification(),
            _       => throw new ArgumentException($"Unknown: {channel}")
        };
    }
}

INotification n = NotificationFactory.CreateNotification("email");
n.Send("Đơn hàng đã xác nhận!");
```

### 9.3 Observer — Thông Báo Tự Động

```csharp
public interface IObserver { void Update(double temperature); }

public class WeatherStation
{
    private List<IObserver> _observers = new();
    private double _temperature;

    public void RegisterObserver(IObserver o) => _observers.Add(o);

    public void SetTemperature(double temp)
    {
        _temperature = temp;
        foreach (var o in _observers) o.Update(_temperature);
    }
}
```

### 9.4 Adapter — Kết Nối Incompatible Interfaces

```csharp
public class SquarePegAdapter : IRound
{
    private readonly SquarePeg _squarePeg;
    public SquarePegAdapter(SquarePeg sp) => _squarePeg = sp;
    public double GetRadius() => _squarePeg.Width * Math.Sqrt(2) / 2;
}
```

| Pattern | Nhóm | Giải quyết | Ví dụ |
|---|---|---|---|
| **Singleton** | Creational | Chỉ 1 instance | Logger, DB connection |
| **Factory** | Creational | Tập trung tạo object | Notification system |
| **Observer** | Behavioral | Thông báo khi thay đổi | Event system, UI |
| **Adapter** | Structural | Kết nối interfaces | Legacy integration |

- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p12-clean-code-deep](./97-csharp-theory-deep-dive.md#p12-clean-code-deep)

