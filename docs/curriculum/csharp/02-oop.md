# 📗 Phần 2: Object-Oriented Programming (OOP)

> **Nội dung**: Classes, Properties, Inheritance, Polymorphism, Interfaces, SOLID  
> **Thời lượng ước tính**: 12–15 giờ

---

## 🎯 Mục Tiêu

- Hiểu 4 trụ cột OOP: Abstraction, Encapsulation, Inheritance, Polymorphism
- Tạo class, property, constructor
- Sử dụng inheritance, virtual/override, abstract, sealed
- Hiểu và implement interfaces
- Áp dụng SRP, DIP, Template Method Pattern

> 📌 Sau khi học xong file này, học tiếp [02b-oop-supplement.md](./02b-oop-supplement.md) để đi sâu vào domain modeling, invariants và anti-patterns.

---

## 🧠 Tư Duy OOP Trước Khi Viết Code

Nhiều người học OOP theo cách "nhớ cú pháp class/interface", nhưng khi làm dự án thật lại rối vì không biết mô hình hóa nghiệp vụ.  
Cách học đúng là đi theo thứ tự:

1. **Bài toán nghiệp vụ**: hệ thống cần làm gì?
2. **Trạng thái quan trọng**: dữ liệu nào phải luôn hợp lệ?
3. **Hành vi nghiệp vụ**: object nào chịu trách nhiệm thay đổi trạng thái đó?
4. **Biên giao tiếp**: phần nào nên đi qua abstraction/interface?

Nếu bạn chỉ có class với nhiều `get; set;` và logic nằm hết trong service/controller, thì đó chưa phải OOP mạnh.

---

## ⚖️ OOP Không Phải Lúc Nào Cũng Tối Ưu

OOP rất tốt cho bài toán nghiệp vụ có trạng thái và hành vi rõ.  
Nhưng với bài toán thuần transform dữ liệu đơn giản, over-design OOP sẽ làm code nặng.

**Khi nên dùng OOP mạnh tay:**
- Bài toán có nhiều rule nghiệp vụ, nhiều trạng thái.
- Cần mở rộng behavior theo loại (payment methods, plugins, policies).
- Cần test unit theo từng business behavior.

**Khi nên giữ đơn giản:**
- Logic tuyến tính, ít trạng thái, ít biến thể.
- Script xử lý dữ liệu ngắn.
- Không có nhu cầu mở rộng hierarchy/polymorphism.

---

## 🧭 Cách Đọc Ví Dụ Trong File Này

Mỗi ví dụ nên được tự kiểm tra theo 4 câu hỏi:

1. Object này đại diện cho cái gì trong nghiệp vụ?
2. Rule nào được bảo vệ bên trong object?
3. Nếu requirement thay đổi, sửa ở đâu là hợp lý nhất?
4. Code hiện tại dễ test hay cần refactor để test được?

> Nếu không trả lời được 4 câu trên, bạn mới hiểu cú pháp, chưa hiểu thiết kế.

---

## 1. Tại Sao Cần OOP?

### 1.1 Vấn đề của code thủ tục

```csharp
// ❌ Code thủ tục - khó bảo trì, lặp lại nhiều
string dogName = "Rex";
int dogAge = 5;
string dogBreed = "Labrador";

string catName = "Mimi";
int catAge = 3;
// → Nếu có 100 con vật? Sẽ cần 300 biến!
```

### 1.2 Giải pháp: OOP

```csharp
// ✅ OOP - gọn, tái sử dụng, dễ mở rộng
class Dog
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Breed { get; set; }
}

Dog rex = new Dog { Name = "Rex", Age = 5, Breed = "Labrador" };
Dog buddy = new Dog { Name = "Buddy", Age = 3, Breed = "Poodle" };
```

---

## 2. Bốn Trụ Cột OOP

```
┌──────────────────────────────────────────────┐
│              OOP (4 pillars)                 │
├───────────┬───────────┬───────────┬──────────┤
│Abstraction│Encapsul.  │Inheritance│Polymorp. │
│Trừu tượng │Đóng gói   │Kế thừa   │Đa hình   │
│Ẩn chi tiết│Bảo vệ data│Tái sử dụng│Nhiều hình│
│phức tạp   │bên trong  │code      │thái      │
└───────────┴───────────┴───────────┴──────────┘
```

### 2.1 Diễn giải thực tế từng trụ cột

- **Abstraction**: chỉ lộ ra API cần dùng, giấu cách làm chi tiết.
- **Encapsulation**: object tự bảo vệ state và rule của chính nó.
- **Inheritance**: tái sử dụng behavior khi quan hệ thật sự là `is-a`.
- **Polymorphism**: code gọi qua base/interface nhưng runtime chọn đúng implementation.

Ví dụ thực tế:
- Thanh toán online: `IPaymentProcessor` là abstraction.
- `Order.Submit()` tự kiểm tra trạng thái trước khi chuyển sang submitted là encapsulation.
- `Shape` -> `Circle/Rectangle` là inheritance.
- `List<Shape>` gọi `Area()` ra kết quả khác nhau là polymorphism.

---

## 3. Class & Object

### 3.1 Khai báo class

```csharp
class Dog
{
    // Fields (trường) - lưu dữ liệu
    private string _name;
    private int _age;
    
    // Constructor - khởi tạo object
    public Dog(string name, int age)
    {
        _name = name;
        _age = age;
    }
    
    // Method - hành vi
    public void Bark()
    {
        Console.WriteLine($"{_name} says: Woof!");
    }
    
    // Property - truy cập dữ liệu an toàn
    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }
}
```

### 3.2 Tạo Object (Instance)

```csharp
Dog rex = new Dog("Rex", 5);     // Tạo object
rex.Bark();                       // Gọi method
Console.WriteLine(rex.Name);      // Truy cập property
```

### 3.3 Class vs Object

| Class | Object |
|-------|--------|
| **Bản thiết kế** (blueprint) | **Thực thể** (instance) |
| Chỉ có 1 định nghĩa | Có thể tạo nhiều thể hiện |
| Mô tả cấu trúc | Chứa dữ liệu thực |
| `class Dog { }` | `new Dog("Rex", 5)` |

---

## 4. Encapsulation (Đóng gói)

### 4.1 Data Hiding

```csharp
class BankAccount
{
    // ❌ public field - ai cũng sửa được → nguy hiểm
    // public decimal balance;
    
    // ✅ private field + public method kiểm soát
    private decimal _balance;
    
    public decimal Balance => _balance;  // Chỉ đọc
    
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Số tiền phải dương!");
        _balance += amount;
    }
    
    public bool Withdraw(decimal amount)
    {
        if (amount > _balance) return false;
        _balance -= amount;
        return true;
    }
}
```

### 4.2 Access Modifiers

| Modifier | Phạm vi truy cập |
|----------|-------------------|
| `public` | Mọi nơi |
| `private` | Chỉ trong class hiện tại |
| `protected` | Class hiện tại + class con |
| `internal` | Cùng assembly (project) |
| `protected internal` | Cùng assembly HOẶC class con |
| `private protected` | Class con TRONG CÙNG assembly |

---

## 5. Constructor

### 5.1 Constructor cơ bản

```csharp
class Person
{
    public string Name { get; }
    public int Age { get; }
    
    // Constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}
```

### 5.2 Constructor Overloading

```csharp
class Rectangle
{
    public double Width { get; }
    public double Height { get; }
    
    // Constructor 1: Hình chữ nhật
    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }
    
    // Constructor 2: Hình vuông (gọi constructor 1)
    public Rectangle(double side) : this(side, side)
    {
    }
    
    // Constructor 3: Mặc định
    public Rectangle() : this(1, 1)
    {
    }
}
```

### 5.3 Validation trong Constructor

```csharp
class Temperature
{
    public double Celsius { get; }
    
    public Temperature(double celsius)
    {
        if (celsius < -273.15)
            throw new ArgumentOutOfRangeException(
                nameof(celsius), 
                "Nhiệt độ không thể thấp hơn -273.15°C (0K)!"
            );
        Celsius = celsius;
    }
}
```

---

## 6. Properties

### 6.1 Full Property

```csharp
class Student
{
    private string _name;
    
    public string Name
    {
        get { return _name; }
        set 
        {
            if (string.IsNullOrWhiteSpace(value))
                throw new ArgumentException("Tên không được trống!");
            _name = value;
        }
    }
}
```

### 6.2 Auto-Implemented Property

```csharp
class Product
{
    public string Name { get; set; }      // Đọc + Ghi
    public decimal Price { get; set; }
    public int Id { get; private set; }   // Đọc public, ghi chỉ trong class
    public DateTime Created { get; }      // Chỉ đọc (gán trong constructor)
}
```

### 6.3 Computed Property

```csharp
class Circle
{
    public double Radius { get; set; }
    
    // Property tính toán - không lưu trữ, tính lại mỗi lần gọi
    public double Area => Math.PI * Radius * Radius;
    public double Circumference => 2 * Math.PI * Radius;
}
```

### 6.4 Object Initializer

```csharp
// Thay vì:
var product = new Product();
product.Name = "Laptop";
product.Price = 999.99m;

// Dùng object initializer:
var product = new Product
{
    Name = "Laptop",
    Price = 999.99m
};
```

---

## 7. Static

### 7.1 Static Method

```csharp
class MathHelper
{
    // Static method - gọi TRỰC TIẾP qua class, không cần object
    public static int Max(int a, int b) => a > b ? a : b;
    public static double ToRadians(double degrees) => degrees * Math.PI / 180;
}

// Gọi:
int max = MathHelper.Max(5, 10);    // Không cần new MathHelper()
```

### 7.2 Static Class

```csharp
// Static class: KHÔNG THỂ tạo object, chỉ chứa static members
static class Converter
{
    public static double CelsiusToFahrenheit(double c) => c * 9 / 5 + 32;
    public static double KgToPounds(double kg) => kg * 2.20462;
}
```

### 7.3 Static Field & Property

```csharp
class User
{
    private static int _nextId = 0;  // Chia sẻ giữa TẤT CẢ objects
    
    public int Id { get; }
    public string Name { get; }
    
    public User(string name)
    {
        Id = ++_nextId;  // Tự tăng cho mỗi user mới
        Name = name;
    }
    
    public static int TotalUsers => _nextId;
}
```

### 7.4 Static Constructor

```csharp
class Config
{
    public static string ConnectionString { get; }
    
    // Static constructor: chạy MỘT LẦN DUY NHẤT khi class được dùng lần đầu
    static Config()
    {
        ConnectionString = File.ReadAllText("config.txt");
    }
}
```

---

## 8. Readonly & Const

```csharp
class PhysicsConstants
{
    // const: phải biết giá trị LÚC COMPILE, không đổi TUYỆT ĐỐI
    public const double PI = 3.14159265358979;
    public const int MAX_SIZE = 100;
    
    // readonly: gán giá trị trong constructor hoặc khai báo, không đổi sau đó
    public readonly DateTime CreatedAt;
    
    public PhysicsConstants()
    {
        CreatedAt = DateTime.Now;  // OK - gán trong constructor
    }
}

// const vs readonly
// const: compile-time, phải là literal/known value
// readonly: runtime, có thể tính toán
```

---

## 9. Expression-bodied Members

```csharp
class Person
{
    public string FirstName { get; }
    public string LastName { get; }
    
    // Expression-bodied constructor
    public Person(string first, string last) => (FirstName, LastName) = (first, last);
    
    // Expression-bodied method
    public string FullName() => $"{FirstName} {LastName}";
    
    // Expression-bodied property
    public string Initials => $"{FirstName[0]}{LastName[0]}";
    
    // Expression-bodied ToString
    public override string ToString() => FullName();
}
```

---

## 10. "this" Keyword

```csharp
class Point
{
    private int x, y;
    
    public Point(int x, int y)
    {
        this.x = x;  // this.x = field, x = parameter
        this.y = y;
    }
    
    // Trả về chính object hiện tại (method chaining)
    public Point MoveX(int dx)
    {
        this.x += dx;
        return this;  // Trả về chính mình
    }
}
```

---

## 11. Single Responsibility Principle (SRP)

### 11.1 Nguyên lý

> **Mỗi class chỉ nên có MỘT lý do để thay đổi.**

```csharp
// ❌ Vi phạm SRP: quá nhiều trách nhiệm
class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }
    
    public decimal CalculateTax() { /* ... */ }     // Tính thuế
    public void SaveToDatabase() { /* ... */ }       // Lưu DB
    public string GenerateReport() { /* ... */ }     // Tạo báo cáo
    public void SendEmail() { /* ... */ }             // Gửi email
}

// ✅ Tuân thủ SRP: tách ra các class riêng
class Employee { public string Name { get; set; } public decimal Salary { get; set; } }
class TaxCalculator { public decimal Calculate(Employee e) { /*...*/ } }
class EmployeeRepository { public void Save(Employee e) { /*...*/ } }
class ReportGenerator { public string Generate(Employee e) { /*...*/ } }
class EmailService { public void Send(string to, string body) { /*...*/ } }
```

---

## 12. Inheritance (Kế thừa)

### 12.1 Cơ bản

```csharp
// Base class (lớp cha)
class Animal
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public void Eat() => Console.WriteLine($"{Name} is eating");
}

// Derived class (lớp con) - kế thừa từ Animal
class Dog : Animal
{
    public string Breed { get; set; }
    
    public void Bark() => Console.WriteLine($"{Name} says: Woof!");
}

class Cat : Animal
{
    public bool IsIndoor { get; set; }
    
    public void Meow() => Console.WriteLine($"{Name} says: Meow!");
}

// Sử dụng
Dog rex = new Dog { Name = "Rex", Age = 5, Breed = "Lab" };
rex.Eat();   // Kế thừa từ Animal
rex.Bark();  // Riêng của Dog
```

### 12.2 Protected Access Modifier

```csharp
class Animal
{
    protected int _energyLevel = 100;  // Chỉ Animal và class con truy cập được
    
    public void Eat()
    {
        _energyLevel += 10;
    }
}

class Dog : Animal
{
    public void Run()
    {
        _energyLevel -= 20;  // ✅ OK - truy cập protected member
    }
}

// Dog dog = new Dog();
// dog._energyLevel;  // ❌ Lỗi! protected không truy cập từ bên ngoài
```

### 12.3 "base" Keyword

```csharp
class Animal
{
    public string Name { get; }
    
    public Animal(string name)
    {
        Name = name;
    }
}

class Dog : Animal
{
    public string Breed { get; }
    
    // Gọi constructor của base class
    public Dog(string name, string breed) : base(name)
    {
        Breed = breed;
    }
}
```

---

## 13. Polymorphism (Đa hình)

### 13.1 Virtual & Override

```csharp
class Shape
{
    // virtual: CÓ THỂ bị override bởi class con
    public virtual double Area() => 0;
    
    public virtual string Describe() => "I am a shape";
}

class Circle : Shape
{
    public double Radius { get; set; }
    
    // override: thay thế implementation của base class
    public override double Area() => Math.PI * Radius * Radius;
    
    public override string Describe() => $"Circle r={Radius}";
}

class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public override double Area() => Width * Height;
    
    public override string Describe() => $"Rectangle {Width}x{Height}";
}

// Polymorphism in action!
Shape[] shapes = { new Circle { Radius = 5 }, new Rectangle { Width = 4, Height = 6 } };

foreach (Shape shape in shapes)
{
    // Gọi method → runtime quyết định method NÀO chạy dựa trên TYPE THỰC
    Console.WriteLine($"{shape.Describe()}: Area = {shape.Area():F2}");
}
// Output:
// Circle r=5: Area = 78.54
// Rectangle 4x6: Area = 24.00
```

### 13.2 System.Object và ToString

```csharp
// MỌI class trong C# đều kế thừa từ System.Object
// Object có sẵn: ToString(), Equals(), GetHashCode()

class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    // Override ToString cho hiển thị đẹp hơn
    public override string ToString() => $"{Name} (tuổi {Age})";
}

Person p = new Person { Name = "Nghĩa", Age = 25 };
Console.WriteLine(p);  // Tự gọi ToString() → "Nghĩa (tuổi 25)"
```

---

## 14. Upcasting & Downcasting

### 14.1 Implicit (Upcasting) — Con → Cha

```csharp
Dog rex = new Dog("Rex", "Lab");
Animal animal = rex;   // ✅ Implicit - Dog IS AN Animal (tự động)
// animal chỉ truy cập được members của Animal, không thấy Dog.Bark()
```

### 14.2 Explicit (Downcasting) — Cha → Con

```csharp
Animal animal = new Dog("Rex", "Lab");

// Cách 1: Cast trực tiếp (nguy hiểm)
Dog dog = (Dog)animal;  // ✅ OK vì animal thực sự là Dog

// Cách 2: "as" operator (an toàn)
Dog? maybeDog = animal as Dog;  // null nếu không phải Dog
if (maybeDog != null)
{
    maybeDog.Bark();
}

// Cách 3: "is" operator + pattern matching (NÊN dùng)
if (animal is Dog d)
{
    d.Bark();  // d đã được cast sẵn
}
```

### 14.3 Null

```csharp
string name = null;  // Reference types có thể là null

// Kiểm tra null
if (name != null)
{
    Console.WriteLine(name.Length);
}

// Null-conditional operator
int? length = name?.Length;  // null nếu name là null

// Null-coalescing operator
string displayName = name ?? "Unknown";  // "Unknown" nếu name là null
```

---

## 15. Abstract Classes & Methods

### 15.1 Abstract Class

```csharp
// Abstract class: KHÔNG THỂ tạo object trực tiếp
abstract class Shape
{
    public string Color { get; set; }
    
    // Abstract method: KHÔNG CÓ body, class con BẮT BUỘC phải override
    public abstract double Area();
    public abstract double Perimeter();
    
    // Non-abstract method: có body, class con KHÔNG BẮT BUỘC override
    public void PrintInfo()
    {
        Console.WriteLine($"Shape: {GetType().Name}, Color: {Color}");
        Console.WriteLine($"Area: {Area():F2}, Perimeter: {Perimeter():F2}");
    }
}

class Circle : Shape
{
    public double Radius { get; set; }
    
    public override double Area() => Math.PI * Radius * Radius;
    public override double Perimeter() => 2 * Math.PI * Radius;
}

// Shape s = new Shape();  // ❌ Lỗi! Không thể tạo abstract class
Circle c = new Circle { Radius = 5, Color = "Red" };
c.PrintInfo();  // OK
```

### 15.2 Abstract vs Virtual

| | abstract | virtual |
|---|---------|---------|
| Có body | ❌ Không | ✅ Có |
| Class con bắt buộc override | ✅ Bắt buộc | ❌ Tùy chọn |
| Chỉ trong abstract class | ✅ | ❌ Class thường cũng được |

---

## 16. Sealed Classes & Methods

```csharp
sealed class FinalClass
{
    // Không class nào được kế thừa from FinalClass
}

// class Child : FinalClass { }  // ❌ Lỗi!

class Parent
{
    public virtual void Method() { }
}

class Child : Parent
{
    // sealed override: ngăn class cháu override tiếp
    public sealed override void Method() { }
}
```

---

## 17. Extension Methods

```csharp
// Extension method: "thêm" method cho type có sẵn mà KHÔNG sửa source code

static class StringExtensions
{
    // "this" keyword trước parameter đầu tiên
    public static bool IsNullOrEmpty(this string str)
    {
        return string.IsNullOrEmpty(str);
    }
    
    public static string Reverse(this string str)
    {
        char[] chars = str.ToCharArray();
        Array.Reverse(chars);
        return new string(chars);
    }
    
    public static int WordCount(this string str)
    {
        return str.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
    }
}

// Sử dụng như method bình thường!
string name = "Hello World";
bool empty = name.IsNullOrEmpty();    // false
string rev = name.Reverse();          // "dlroW olleH"
int words = name.WordCount();          // 2
```

---

## 18. Interfaces

### 18.1 Khai báo và implement

```csharp
// Interface: "hợp đồng" - định nghĩa PHẢI có gì, KHÔNG định nghĩa cách làm
interface IMovable
{
    void Move(int dx, int dy);
    double Speed { get; }
}

interface IDrawable
{
    void Draw();
}

// Implement nhiều interfaces (C# không cho phép đa kế thừa class)
class Player : IMovable, IDrawable
{
    public double X { get; set; }
    public double Y { get; set; }
    
    public double Speed => 5.0;
    
    public void Move(int dx, int dy)
    {
        X += dx * Speed;
        Y += dy * Speed;
    }
    
    public void Draw()
    {
        Console.WriteLine($"Drawing player at ({X}, {Y})");
    }
}
```

### 18.2 Interface vs Abstract Class

| | Interface | Abstract Class |
|---|-----------|---------------|
| Đa kế thừa | ✅ Implement nhiều | ❌ Chỉ 1 class cha |
| Constructor | ❌ Không | ✅ Có |
| Fields | ❌ Không | ✅ Có |
| Default methods (C# 8+) | ✅ Có | ✅ Có |
| Khi nào dùng | **Can-do** (có thể làm gì) | **Is-a** (thuộc loại gì) |

---

## 19. JSON Serialization

```csharp
using System.Text.Json;

class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// Serialize: Object → JSON string
Person p = new Person { Name = "Nghĩa", Age = 25 };
string json = JsonSerializer.Serialize(p);
// {"Name":"Nghĩa","Age":25}

// Deserialize: JSON string → Object
Person p2 = JsonSerializer.Deserialize<Person>(json);

// Pretty print
string prettyJson = JsonSerializer.Serialize(p, new JsonSerializerOptions
{
    WriteIndented = true
});
```

---

## 20. Dependency Inversion & Dependency Injection

### 20.1 Nguyên lý

> **Module cấp cao KHÔNG nên phụ thuộc vào module cấp thấp. CẢ HAI nên phụ thuộc vào abstraction (interface).**

```csharp
// ❌ Vi phạm DIP: UserService phụ thuộc trực tiếp vào SqlDatabase
class UserService
{
    private SqlDatabase _db = new SqlDatabase();
    public void Save(User user) => _db.Insert(user);
}

// ✅ Tuân thủ DIP: phụ thuộc vào interface
interface IDatabase
{
    void Insert(object item);
}

class SqlDatabase : IDatabase { /* ... */ }
class MongoDatabase : IDatabase { /* ... */ }

class UserService
{
    private readonly IDatabase _db;
    
    // Dependency Injection: nhận dependency từ bên ngoài
    public UserService(IDatabase db)
    {
        _db = db;
    }
    
    public void Save(User user) => _db.Insert(user);
}

// Dùng:
var service = new UserService(new SqlDatabase());     // Dùng SQL
var service2 = new UserService(new MongoDatabase());  // Dùng Mongo
```

---

## 🚨 Các Lỗi OOP Thường Gặp (Và Cách Tránh)

1. **God Class**: một class làm quá nhiều việc.
- Dấu hiệu: class dài, nhiều method không liên quan chặt.
- Cách tránh: tách responsibility theo SRP.

2. **Inheritance sai ngữ nghĩa**:
- Dấu hiệu: class con override để "tắt bớt" behavior class cha.
- Cách tránh: ưu tiên composition nếu quan hệ không thật sự `is-a`.

3. **Anemic Model**:
- Dấu hiệu: object chỉ có data, business rule nằm rải rác.
- Cách tránh: đưa logic nghiệp vụ vào method của object sở hữu state.

4. **Lạm dụng `public set`**:
- Dấu hiệu: bất kỳ chỗ nào cũng sửa được state quan trọng.
- Cách tránh: dùng `private set`, method có validate, hoặc immutable model.

5. **Interface quá to**:
- Dấu hiệu: class implement nhiều method không cần dùng.
- Cách tránh: tách nhỏ interface theo ISP.

---

## ✅ Rubric Tự Đánh Giá Phần OOP

Sau khi học xong file này, bạn nên tự check:

1. Có phân biệt rõ `is-a` và `has-a` khi thiết kế?
2. Có biết khi nào dùng abstract class vs interface theo context?
3. Có thể giải thích vì sao chọn composition hoặc inheritance cho một case cụ thể?
4. Có thể viết 1 module nhỏ có DI + unit test mà không phụ thuộc concrete class?
5. Có thể refactor một class vi phạm SRP thành thiết kế rõ trách nhiệm hơn?

---

## 21. Enums

```csharp
// Enum: tập hợp các hằng số có tên
enum Season
{
    Spring,    // 0
    Summer,    // 1
    Autumn,    // 2
    Winter     // 3
}

enum HttpStatus
{
    OK = 200,
    NotFound = 404,
    ServerError = 500
}

// Sử dụng
Season current = Season.Summer;

switch (current)
{
    case Season.Spring:
        Console.WriteLine("Mùa xuân");
        break;
    case Season.Summer:
        Console.WriteLine("Mùa hè");
        break;
    // ...
}

// Chuyển đổi
int value = (int)Season.Summer;        // 1
Season s = (Season)2;                   // Autumn
string name = Season.Spring.ToString(); // "Spring"
```

---

## 📝 Assignment: Cookies Cookbook

**Kỹ năng**: Interfaces, DI, JSON, Template Method Pattern, File I/O

Xây dựng ứng dụng tạo công thức nấu ăn:
1. Hiển thị danh sách nguyên liệu
2. Người dùng chọn nguyên liệu
3. Lưu/đọc công thức từ file (TXT hoặc JSON)

**Design Patterns áp dụng**:
- **Template Method**: class base định nghĩa khung đọc/ghi file, class con triển khai chi tiết
- **DIP**: dùng interface cho file reader/writer

---

## 🧪 Coding Exercises Tổng Hợp

| # | Bài | Kiến thức |
|---|-----|-----------|
| 15 | HotelBooking class | Class, Constructor |
| 16 | Triangle class | Encapsulation, Validation |
| 17 | Dog class | Optional parameters |
| 18 | Properties of Order class | Properties |
| 19 | Computed properties - DailyAccountState | Computed property |
| 20 | Static classes - NumberToDayOfWeekTranslator | Static |
| 21 | string.Split and string.Join | String methods |
| 22 | Inheritance & Overriding - Animals | Virtual/Override |
| 23 | Virtual methods - StringsProcessor | Polymorphism |
| 24 | "is" operator - NumericTypesDescriber | Type checking |
| 25 | Abstract methods - Shapes | Abstract |
| 26 | Extension methods - List extensions | Extension methods |
| 27 | Interfaces - Number transformations | Interfaces |

---

## ❓ Câu Hỏi Kiểm Tra

### Section 3: Basics of OOP

1. Class vs Object — khác nhau thế nào? Cho ví dụ cụ thể.
2. Field và Property khác nhau thế nào? Tại sao nên dùng Property?
3. Constructor có thể trả về giá trị không? Tại sao?
4. `private`, `protected`, `internal` — khi nào dùng cái nào?
5. Static method có thể truy cập instance member không? Tại sao?
6. `const` vs `readonly` — khác nhau thế nào? Cho ví dụ.
7. SRP là gì? Cho ví dụ class vi phạm SRP và cách fix.
8. Object initializer `{ }` hoạt động thế nào bên trong?
9. `this` keyword dùng khi nào? Cho 3 use cases.
10. Extension method là gì? Tại sao không thể override?

### Section 4: Inheritance & Polymorphism

11. C# có hỗ trợ đa kế thừa (multiple inheritance) không? Tại sao?
12. `virtual` vs `abstract` method — khác nhau thế nào?
13. Nếu không đánh dấu method là `virtual`, class con có override được không?
14. `new` keyword khi hiding method — khác `override` thế nào?
15. Upcasting và Downcasting — khi nào cần? Rủi ro?
16. `is` vs `as` — khi nào dùng cái nào?
17. Interface vs Abstract class — chọn cái nào? Cho kịch bản cụ thể.
18. DIP là gì? DI (Dependency Injection) khác DIP thế nào?
19. `sealed` class dùng khi nào?
20. Enum có thể có method không?

---

## 💼 Câu Hỏi Phỏng Vấn

### Cơ bản

> **Q1: What are the four pillars of OOP?**
> A: **Abstraction** (ẩn chi tiết phức tạp, chỉ hiện interface đơn giản), **Encapsulation** (bảo vệ dữ liệu nội bộ qua access modifiers), **Inheritance** (tái sử dụng code từ class cha), **Polymorphism** (cùng method, hành vi khác nhau dựa trên type thực).

> **Q2: What is the difference between `class` and `struct` in C#?**
> A: `class` là reference type (heap), hỗ trợ kế thừa, nullable. `struct` là value type (stack), KHÔNG kế thừa, copy by value, hiệu suất hơn cho dữ liệu nhỏ. Dùng struct khi: < 16 bytes, immutable, giá trị thuần túy.

> **Q3: What is the difference between an `abstract class` and an `interface`?**
> A: Abstract class: chứa implementation + fields + constructor, chỉ kế thừa 1 class. Interface: chỉ "hợp đồng", implement nhiều interfaces. Dùng abstract class khi có shared code, interface khi định nghĩa behavior cần implement.

> **Q4: Can you instantiate an abstract class?**
> A: KHÔNG trực tiếp. Phải tạo instance qua concrete class kế thừa từ nó.

> **Q5: What is method hiding (`new` keyword) vs method overriding?**
> A: Override (virtual/override): runtime polymorphism — gọi method dựa trên ACTUAL type. Hiding (new): compile-time — gọi method dựa trên DECLARED type. Override = hầu như luôn đúng, hiding = hiếm khi cần.

### Trung bình

> **Q6: What is the difference between `is` and `as` operators?**
> A: `is` kiểm tra type và trả về bool (hoặc pattern match). `as` cố cast và trả về null nếu thất bại (chỉ cho reference types). Ưu tiên `is` với pattern matching: `if (obj is Dog d)`.

> **Q7: Explain SOLID principles.**
> A:
> - **S** — Single Responsibility: mỗi class 1 trách nhiệm
> - **O** — Open/Closed: mở cho mở rộng, đóng cho sửa đổi
> - **L** — Liskov Substitution: class con dùng thay class cha không gây lỗi
> - **I** — Interface Segregation: interface nhỏ, chuyên biệt
> - **D** — Dependency Inversion: phụ thuộc vào abstraction, không vào implementation

> **Q8: What is Dependency Injection? What are the 3 types?**
> A: DI là pattern truyền dependency từ bên ngoài vào (thay vì tự tạo bên trong).
> 1. **Constructor injection** (phổ biến nhất): nhận qua constructor
> 2. **Property injection**: gán qua property
> 3. **Method injection**: truyền qua method parameter

> **Q9: What is the difference between shallow copy and deep copy?**
> A: **Shallow copy**: copy reference → 2 objects chia sẻ cùng nested objects. **Deep copy**: copy TẤT CẢ nested objects → hoàn toàn độc lập. MemberwiseClone() tạo shallow copy. Deep copy cần implement ICloneable hoặc serialize/deserialize.

> **Q10: Can a C# class inherit from multiple classes?**
> A: KHÔNG. C# chỉ hỗ trợ **single inheritance** cho classes (tránh diamond problem). Nhưng CÓ THỂ implement nhiều **interfaces**.

### Nâng cao

> **Q11: What is the diamond problem and how does C# solve it?**
> A: Diamond problem xảy ra khi class kế thừa 2 classes có cùng method → nhập nhằng method nào được gọi. C# giải quyết bằng cách chỉ cho phép single class inheritance + multiple interface implementation.

> **Q12: What is the Template Method Pattern?**
> A: Abstract class định nghĩa "khung" algorithm (các bước), class con override các bước cụ thể. Skeleton algorithm cố định, chi tiết linh hoạt. Ví dụ: abstract class `FileProcessor` có `Process()` gọi `Read()` → `Transform()` → `Write()`, class con override Read/Transform/Write.

> **Q13: When should you use composition over inheritance?**
> A: Khi quan hệ là HAS-A thay vì IS-A. Khi cần flexibility (thay đổi behavior runtime). Khi inheritance hierarchy quá sâu. Favor composition vì nó loose coupling, dễ test, dễ thay đổi.

---

## 🏋️ Bài Tập Thực Hành

### Mức Dễ ⭐

**BT1: Shape Calculator** — Tạo abstract class `Shape` với abstract method `Area()` và `Perimeter()`. Implement `Circle`, `Rectangle`, `Triangle`. Dùng polymorphism: `List<Shape>` tính tổng diện tích.

**BT2: Student Manager** — Class `Student` (Name, Grade, Score). Tạo List<Student>, thêm 5 sinh viên, in ra danh sách sorted theo Score giảm dần.

**BT3: Enum Practice** — Enum `TrafficLight` (Red, Yellow, Green). Viết method nhận TrafficLight, trả về string mô tả hành động ("Dừng", "Chuẩn bị", "Đi").

### Mức Trung Bình ⭐⭐

**BT4: Payment System** — Implement `IPaymentProcessor` interface với 3 classes: `CreditCardPayment`, `PayPalPayment`, `CryptoPayment`. Service nhận `IPaymentProcessor` qua constructor.

**BT5: Bank Account System** — Abstract class `Account` (Balance, Deposit, Withdraw). Implement `SavingsAccount` (lãi suất, không rút quá 1 triệu/lần) và `CheckingAccount` (phí rút 5000đ). Override ToString.

**BT6: Animal Hierarchy** — Interface `IFeedable`, `ISpeakable`, `ITrainable`. Class `Dog` implement cả 3, `Cat` implement 2, `Fish` implement 1. Sử dụng polymorphism để quản lý List<IFeedable>.

**BT7: Plugin System** — Interface `IPlugin` (Name, Execute). Tạo 3 plugins: `LoggerPlugin`, `TimerPlugin`, `NotificationPlugin`. Class `PluginManager` lưu List<IPlugin>, method `RunAll()`.

### Mức Khó ⭐⭐⭐

**BT8: JSON File System** — Interface `IDataStore<T>` (Save, Load, Delete). Implement `JsonFileStore<T>` lưu/đọc JSON file. Test với class `Contact` (Name, Phone, Email).

**BT9: Builder Pattern** — Implement Builder pattern cho class `Pizza` (Size, Crust, Toppings, ExtraCheese).

```csharp
var pizza = new PizzaBuilder()
    .SetSize("Large")
    .SetCrust("Thin")
    .AddTopping("Pepperoni")
    .AddTopping("Mushrooms")
    .WithExtraCheese()
    .Build();
```

**BT10: Command Pattern** — Interface `ICommand` (Execute, Undo). Implement `AddTextCommand`, `DeleteTextCommand`, `ReplaceTextCommand`. Class `TextEditor` với undo/redo history.

---

## 📋 Tổng Hợp Interview Questions — OOP

| # | Câu hỏi | Chủ đề | Mức độ |
|---|---------|--------|--------|
| 1 | What are the 4 pillars of OOP? | OOP | ⭐ |
| 2 | Class vs Struct? | Types | ⭐ |
| 3 | Abstract class vs Interface? | Abstraction | ⭐ |
| 4 | Virtual vs Abstract vs Override vs New? | Polymorphism | ⭐⭐ |
| 5 | What is Encapsulation? Why important? | Encapsulation | ⭐ |
| 6 | Access modifiers in C#? | Encapsulation | ⭐ |
| 7 | What is SOLID? Explain each. | Principles | ⭐⭐ |
| 8 | What is Dependency Injection? 3 types? | DI | ⭐⭐ |
| 9 | `is` vs `as` operator? | Casting | ⭐ |
| 10 | Shallow copy vs Deep copy? | Objects | ⭐⭐ |
| 11 | Can C# have multiple inheritance? | Inheritance | ⭐ |
| 12 | Template Method Pattern? | Patterns | ⭐⭐ |
| 13 | Composition vs Inheritance? | Design | ⭐⭐⭐ |
| 14 | Diamond problem? | Inheritance | ⭐⭐⭐ |
| 15 | Sealed class — when to use? | Inheritance | ⭐⭐ |

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p02-oop](./99-answer-key-csharp.md#p02-oop)
- Bài tập thực hành: [99-answer-key-csharp.md#p02-oop-exercises](./99-answer-key-csharp.md#p02-oop-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p02-oop-deep](./97-csharp-theory-deep-dive.md#p02-oop-deep)

---

## 22. Method Hiding vs Overriding (Chi Tiết)

Đây là khái niệm **dễ nhầm lẫn nhất** trong C# OOP:

```csharp
public class Base
{
    public virtual void Display() => Console.WriteLine("Base.Display()");
    public void ShowInfo() => Console.WriteLine("Base.ShowInfo()");
}

// OVERRIDING — Đa hình thực sự
public class OverrideChild : Base
{
    public override void Display() => Console.WriteLine("OverrideChild.Display()");
}

// HIDING — Che giấu method (KHÔNG phải đa hình)
public class HidingChild : Base
{
    public new void ShowInfo() => Console.WriteLine("HidingChild.ShowInfo()");
}
```

**Sự khác biệt quan trọng:**
```csharp
// OVERRIDING — Luôn gọi đúng kiểu THỰC TẾ
Base obj1 = new OverrideChild();
obj1.Display();  // → "OverrideChild.Display()" ✅ Đa hình!

// HIDING — Gọi theo kiểu REFERENCE!
Base obj2 = new HidingChild();
obj2.ShowInfo();           // → "Base.ShowInfo()" ⚠️ Gọi Base!
((HidingChild)obj2).ShowInfo(); // → "HidingChild.ShowInfo()" ← cần cast
```

| | Method Hiding (`new`) | Method Overriding (`override`) |
|---|---|---|
| **Polymorphic?** | ❌ Theo reference type | ✅ Theo actual type |
| **Binding** | Compile-time | **Runtime** |

> ⚠️ **Quy tắc vàng:** Gần như luôn dùng `override` thay vì `new`. Method hiding phá vỡ polymorphism.

---

## 23. Virtual Method Table (vtable)

Khi gọi virtual method, **CLR** tra cứu **vtable** để tìm đúng implementation:

```
Khi gọi: mediaFile.Play()

1. CLR xem kiểu THỰC TẾ của object (không phải reference)
2. Tra cứu vtable:
   ┌─────────────┬──────────────────────┐
   │ Object Type │ Play() points to     │
   ├─────────────┼──────────────────────┤
   │ AudioFile   │ AudioFile.Play()     │
   │ VideoFile   │ VideoFile.Play()     │
   │ StreamingF. │ StreamingFile.Play() │
   └─────────────┴──────────────────────┘
3. Gọi implementation tương ứng

⚡ Overhead: ~ 1 nanosecond (không đáng kể)
```

