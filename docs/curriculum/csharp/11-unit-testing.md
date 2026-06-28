# 📗 Phần 11: Unit Testing

> **Nội dung**: NUnit, AAA pattern, Mocking, TDD, test best practices  
> **Thời lượng ước tính**: 5–6 giờ

---

## 🎯 Mục Tiêu

- Hiểu tại sao cần unit test
- Viết test với NUnit (AAA pattern)
- TestCase, TestCaseSource (parameterized tests)
- Mocking với Moq
- Clean code trong tests
- Dependency Injection cho testability

---

## 1. Unit Test Fundamentals

### 1.1 Cài đặt

```bash
dotnet new nunit -n MyApp.Tests
dotnet add reference ../MyApp/MyApp.csproj
dotnet add package Moq           # Mocking framework
```

### 1.2 First Test

```csharp
using NUnit.Framework;

[TestFixture]
public class CalculatorTests
{
    [Test]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        // Arrange (Chuẩn bị)
        var calculator = new Calculator();
        
        // Act (Thực hiện)
        int result = calculator.Add(3, 5);
        
        // Assert (Kiểm tra)
        Assert.That(result, Is.EqualTo(8));
    }
}
```

### 1.3 Naming Convention

```
MethodName_Scenario_ExpectedBehavior

Ví dụ:
Add_TwoPositiveNumbers_ReturnsSum
Withdraw_AmountGreaterThanBalance_ThrowsException
IsAdult_AgeIs17_ReturnsFalse
```

---

## 2. AAA Pattern

```csharp
[Test]
public void Divide_ByZero_ThrowsDivideByZeroException()
{
    // Arrange
    var calc = new Calculator();
    
    // Act & Assert (cho exceptions)
    Assert.Throws<DivideByZeroException>(() => calc.Divide(10, 0));
}
```

---

## 3. Parameterized Tests

### 3.1 TestCase

```csharp
[TestCase(1, 1, 2)]
[TestCase(0, 0, 0)]
[TestCase(-1, 1, 0)]
[TestCase(100, 200, 300)]
public void Add_VariousInputs_ReturnsExpected(int a, int b, int expected)
{
    var calc = new Calculator();
    Assert.That(calc.Add(a, b), Is.EqualTo(expected));
}
```

### 3.2 TestCaseSource

```csharp
private static IEnumerable<TestCaseData> DivisionCases()
{
    yield return new TestCaseData(10, 2).Returns(5);
    yield return new TestCaseData(9, 3).Returns(3);
    yield return new TestCaseData(7, 2).Returns(3);  // Integer division
}

[TestCaseSource(nameof(DivisionCases))]
public int Divide_ValidInputs_ReturnsQuotient(int a, int b)
{
    return new Calculator().Divide(a, b);
}
```

---

## 4. Common Assertions

```csharp
// Equality
Assert.That(result, Is.EqualTo(expected));
Assert.That(result, Is.Not.EqualTo(wrong));

// Boolean
Assert.That(isValid, Is.True);
Assert.That(isEmpty, Is.False);

// Null
Assert.That(obj, Is.Null);
Assert.That(obj, Is.Not.Null);

// Collections
Assert.That(list, Has.Count.EqualTo(3));
Assert.That(list, Contains.Item("hello"));
Assert.That(list, Is.Empty);
Assert.That(list, Is.Ordered);

// Strings
Assert.That(str, Does.Contain("hello"));
Assert.That(str, Does.StartWith("Hi"));
Assert.That(str, Is.EqualTo("HELLO").IgnoreCase);

// Exceptions
Assert.Throws<ArgumentException>(() => Method(badArg));
var ex = Assert.Throws<InvalidOperationException>(() => Method());
Assert.That(ex.Message, Does.Contain("invalid"));

// Range
Assert.That(value, Is.InRange(1, 100));
Assert.That(value, Is.GreaterThan(0));
```

---

## 5. Mocking with Moq

### 5.1 Tại sao cần Mock?

```csharp
// Vấn đề: class phụ thuộc vào external service
class OrderService
{
    private readonly IEmailService _emailService;
    private readonly IDatabase _db;
    
    public OrderService(IEmailService email, IDatabase db)
    {
        _emailService = email;
        _db = db;
    }
    
    public bool PlaceOrder(Order order)
    {
        _db.Save(order);
        _emailService.Send(order.CustomerEmail, "Order confirmed!");
        return true;
    }
}

// Test KHÔNG NÊN gửi email thật hay ghi DB thật!
// → Dùng Mock: object giả lập
```

### 5.2 Tạo Mock

```csharp
using Moq;

[Test]
public void PlaceOrder_ValidOrder_SavesAndSendsEmail()
{
    // Arrange
    var mockDb = new Mock<IDatabase>();
    var mockEmail = new Mock<IEmailService>();
    
    // Setup mock behavior
    mockDb.Setup(db => db.Save(It.IsAny<Order>())).Returns(true);
    
    var service = new OrderService(mockEmail.Object, mockDb.Object);
    var order = new Order { CustomerEmail = "test@test.com" };
    
    // Act
    bool result = service.PlaceOrder(order);
    
    // Assert
    Assert.That(result, Is.True);
    
    // Verify mock interactions
    mockDb.Verify(db => db.Save(order), Times.Once);
    mockEmail.Verify(e => e.Send("test@test.com", It.IsAny<string>()), Times.Once);
}
```

### 5.3 Mock Setup

```csharp
// Return specific value
mock.Setup(x => x.GetPrice("apple")).Returns(1.99m);

// Return based on input
mock.Setup(x => x.GetPrice(It.IsAny<string>()))
    .Returns((string name) => name.Length * 0.5m);

// Throw exception
mock.Setup(x => x.GetPrice("banned"))
    .Throws<InvalidOperationException>();

// Callback
mock.Setup(x => x.Save(It.IsAny<Order>()))
    .Callback<Order>(o => Console.WriteLine($"Saving {o.Id}"))
    .Returns(true);
```

### 5.4 Verify

```csharp
mock.Verify(x => x.Method(), Times.Once);
mock.Verify(x => x.Method(), Times.Never);
mock.Verify(x => x.Method(), Times.Exactly(3));
mock.Verify(x => x.Method(), Times.AtLeastOnce);
mock.Verify(x => x.Method(It.Is<int>(n => n > 0)), Times.Once);
```

---

## 6. Testing Best Practices

```csharp
// ✅ Test ONE thing per test
// ✅ Tests should be INDEPENDENT (không phụ thuộc thứ tự)
// ✅ Tests should be FAST
// ✅ Tests should be READABLE

// Common setup
[TestFixture]
public class CalculatorTests
{
    private Calculator _sut;  // System Under Test
    
    [SetUp]
    public void Setup()
    {
        _sut = new Calculator();
    }
    
    [TearDown]
    public void Teardown()
    {
        // Cleanup after each test
    }
}

// ❌ Don't test private methods (test via public API)
// ❌ Don't test framework code (e.g., List.Add works)
// ❌ Don't make tests that pass only on your machine
```

---

## 7. Untestable Code

```csharp
// ❌ Hard to test: concrete dependency
class ReportGenerator
{
    public string Generate()
    {
        var data = new SqlDatabase().GetData();  // ← Can't mock!
        return FormatReport(data);
    }
}

// ✅ Testable: depend on abstraction
class ReportGenerator
{
    private readonly IDatabase _db;
    public ReportGenerator(IDatabase db) => _db = db;
    
    public string Generate()
    {
        var data = _db.GetData();  // ← Can mock via DI!
        return FormatReport(data);
    }
}

// ❌ Hard to test: static method calls
class PriceCalculator
{
    public decimal Calculate(decimal price)
    {
        decimal tax = TaxService.GetTaxRate();  // ← Static, can't mock!
        return price * (1 + tax);
    }
}
```

---

## ❓ Câu Hỏi Kiểm Tra

1. AAA pattern là gì? (Arrange, Act, Assert)
2. `[TestCase]` vs `[TestCaseSource]` — khi nào dùng cái nào?
3. Tại sao cần mock? Cho ví dụ.
4. `mock.Setup()` vs `mock.Verify()` — khác nhau?
5. Naming convention test method: `MethodName_Scenario_Expected` nghĩa gì?
6. `[SetUp]` và `[TearDown]` dùng khi nào?
7. Tại sao KHÔNG nên test private methods?
8. `Assert.Throws<T>` vs `try-catch` trong test — cái nào nên dùng?
9. SUT là gì?
10. Code untestable thường có đặc điểm gì?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the AAA pattern in unit testing?**
> A: **Arrange**: chuẩn bị data, mock, SUT. **Act**: gọi method cần test. **Assert**: kiểm tra kết quả. Mỗi test NÊN có đúng 1 Act và 1 Assert (hoặc ít assertions liên quan). Tách biệt 3 phần rõ ràng.

> **Q: What is mocking? Why do you need it?**
> A: Mock = object giả lập thay thế dependency thật (DB, API, email). Cần vì: (1) tránh side effects thật, (2) test nhanh hơn, (3) kiểm soát behavior, (4) verify interactions. Dùng Moq framework.

> **Q: What is the difference between unit test and integration test?**
> A: **Unit test**: test 1 unit (method/class) RIÊNG BIỆT, mock dependencies, nhanh. **Integration test**: test nhiều components CÙNG NHAU, dùng dependencies thật (DB, API), chậm hơn. Tỷ lệ: nhiều unit tests, ít integration tests.

> **Q: How do you make code testable?**
> A: (1) Depend on abstractions (interfaces), (2) Constructor injection (DI), (3) Tránh static methods cho external dependencies, (4) Small focused methods, (5) Avoid tight coupling. Đây là DIP (Dependency Inversion Principle).

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Viết `Calculator` class và 10 test cases cho methods Add, Subtract, Divide (bao gồm edge cases: overflow, divide by zero).

**BT2**: Test `BankAccount.Withdraw()` dùng Moq cho `INotificationService`. Verify gửi notification khi balance < 100.

**BT3**: Refactor untestable code: class `WeatherReport` gọi API trực tiếp → inject `IWeatherApi`. Viết tests với mock.

**BT4**: Viết tests cho `PasswordValidator` (min length, uppercase, lowercase, digit, special char). Dùng TestCase parameterized.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p11-unit-testing](./99-answer-key-csharp.md#p11-unit-testing)
- Bài tập thực hành: [99-answer-key-csharp.md#p11-unit-testing-exercises](./99-answer-key-csharp.md#p11-unit-testing-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p11-unit-testing-deep](./97-csharp-theory-deep-dive.md#p11-unit-testing-deep)

