# 📘 Phần 1: Introduction & C# Fundamentals

> **Nội dung**: Biến, toán tử, điều kiện, vòng lặp, mảng, list, methods, debug  
> **Thời lượng ước tính**: 8–10 giờ

---

## 🎯 Mục Tiêu Phần Này

Sau khi hoàn thành, bạn sẽ:
- Cài đặt và sử dụng Visual Studio / VS Code
- Hiểu quy trình từ source code → executable
- Viết chương trình C# cơ bản với biến, toán tử, điều kiện, vòng lặp
- Sử dụng mảng, list, phương thức
- Debug bằng breakpoint
- Xử lý input từ người dùng

---

## 1. Giới Thiệu & Cài Đặt Môi Trường

### 1.1 Cài đặt Visual Studio Community (Windows)

```
1. Tải Visual Studio Community từ https://visualstudio.microsoft.com
2. Chọn workload: ".NET desktop development"
3. Cài đặt và khởi động
4. Tạo project mới: Console App (.NET 8 trở lên)
```

### 1.2 Cài đặt Visual Studio Code (MacOS/Linux)

```
1. Tải VS Code từ https://code.visualstudio.com
2. Cài .NET SDK từ https://dotnet.microsoft.com/download
3. Cài Extension: "C# Dev Kit" trong VS Code
4. Tạo project: dotnet new console -n MyFirstApp
5. Mở folder và chạy: dotnet run
```

### 1.3 Quy trình biên dịch C#

```
Source Code (.cs)
    ↓ C# Compiler (Roslyn)
Common Intermediate Language (CIL / IL)
    ↓ .NET Runtime (CLR)  
Machine Code (thực thi)
```

> **Lưu ý**: C# là ngôn ngữ **compiled** (biên dịch), không phải interpreted (thông dịch). Code được chuyển thành IL trước, rồi CLR chuyển thành machine code lúc chạy (JIT - Just In Time compilation).

### 1.4 .NET vs .NET Framework

```
.NET Framework (cũ):
  - Chỉ chạy trên Windows
  - Version cuối: 4.8 (không phát triển thêm)

.NET (mới, trước gọi là .NET Core):
  - Cross-platform: Windows, macOS, Linux
  - .NET 5 → 6 → 7 → 8 → 9 (phiên bản mới mỗi năm)
  - Hiệu suất cao hơn, open-source
  - ĐÂY LÀ CÁI BẠN NÊN HỌC ✅
```

### 1.5 Cấu trúc project .NET cơ bản

```
MyApp/
├── MyApp.csproj     ← Project file (config, dependencies)
├── Program.cs       ← Entry point (nơi code chạy đầu tiên)
├── bin/             ← Output (file .dll/.exe sau khi build)
└── obj/             ← Temporary build files
```

```csharp
// Program.cs — Top-level statements (C# 9+)
Console.WriteLine("Hello, World!");

// Trước C# 9, phải viết đầy đủ:
namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
// Main() là entry point — nơi chương trình bắt đầu chạy
```

### ❓ Câu hỏi kiểm tra

1. CIL là gì? Tại sao C# cần biên dịch sang CIL trước?
2. JIT compilation khác AOT compilation như thế nào?
3. Sự khác biệt giữa .NET Framework và .NET (Core)?

### 💼 Câu hỏi phỏng vấn

> **Q: What is the difference between .NET Framework and .NET Core/.NET?**
> A: .NET Framework chỉ chạy trên Windows, version cuối là 4.8. .NET (Core) là cross-platform, open-source, hiệu suất cao hơn, và đang được Microsoft tích cực phát triển.

> **Q: What is the CLR?**
> A: Common Language Runtime — quản lý việc thực thi code .NET: JIT compilation, garbage collection, type safety, exception handling, thread management.

---

## 2. Biến (Variables)

### 2.1 Khái niệm

Biến là **ô nhớ được đặt tên** để lưu trữ dữ liệu. Mỗi biến có:
- **Tên** (identifier)
- **Kiểu dữ liệu** (type) — xác định loại dữ liệu được lưu
- **Giá trị** (value)

### 2.2 Khai báo và gán giá trị

```csharp
// Khai báo (declaration) — chưa có giá trị
int age;

// Gán giá trị (assignment)
age = 25;

// Khai báo + gán cùng lúc (initialization)
int height = 175;
string name = "Nghĩa";
double salary = 1500.50;
bool isStudent = true;

// Khai báo nhiều biến cùng kiểu
int x = 1, y = 2, z = 3;

// ⚠️ Biến PHẢI được gán giá trị trước khi sử dụng
int number;
// Console.WriteLine(number);  // ❌ LỖI! Chưa gán giá trị!
```

### 2.3 Các kiểu dữ liệu cơ bản

| Kiểu | Kích thước | Phạm vi | Ví dụ |
|------|-----------|---------|-------|
| `int` | 4 bytes | -2.1 tỷ → 2.1 tỷ | `int x = 42;` |
| `long` | 8 bytes | ±9.2 × 10¹⁸ | `long big = 9999999999L;` |
| `double` | 8 bytes | ±5.0 × 10⁻³²⁴ → ±1.7 × 10³⁰⁸ | `double pi = 3.14;` |
| `float` | 4 bytes | ±1.5 × 10⁻⁴⁵ → ±3.4 × 10³⁸ | `float f = 3.14f;` |
| `decimal` | 16 bytes | ±7.9 × 10²⁸ (chính xác) | `decimal money = 19.99m;` |
| `bool` | 1 byte | `true` / `false` | `bool ok = true;` |
| `char` | 2 bytes | 1 ký tự Unicode | `char c = 'A';` |
| `string` | varies | chuỗi ký tự | `string s = "hello";` |

**Khi nào dùng kiểu nào?**

```csharp
// int: số nguyên thông thường (tuổi, số lượng, index)
int age = 25;
int count = 100;

// long: số nguyên RẤT LỚN (ID database, timestamp)
long userId = 1234567890123L;  // Chú ý hậu tố L

// double: số thực (khoa học, kỹ thuật, đồ họa)
double distance = 384400.5;    // km to Moon
double pi = 3.14159265358979;

// float: số thực ÍT chính xác (game dev, khi cần tiết kiệm bộ nhớ)
float speed = 3.5f;             // Chú ý hậu tố f

// decimal: TIỀN TỆ (chính xác tuyệt đối!)
decimal price = 19.99m;         // Chú ý hậu tố m
decimal tax = 0.08m;

// ⚠️ ĐỪNG bao giờ dùng double cho tiền!
double money1 = 0.1 + 0.2;  // = 0.30000000000000004 ❌
decimal money2 = 0.1m + 0.2m;  // = 0.3 ✅
```

### 2.4 Type Conversion (Chuyển đổi kiểu)

```csharp
// Implicit conversion (tự động, an toàn — nhỏ → lớn)
int myInt = 42;
long myLong = myInt;      // int → long: OK, không mất dữ liệu
double myDouble = myInt;  // int → double: OK

// Explicit conversion (ép kiểu — lớn → nhỏ, CÓ THỂ mất dữ liệu)
double d = 9.78;
int i = (int)d;           // 9 — mất phần thập phân!

long bigNum = 3000000000L;
int small = (int)bigNum;  // ⚠️ Overflow! Kết quả sai!

// Convert class
string ageStr = "25";
int age = Convert.ToInt32(ageStr);
double price = Convert.ToDouble("19.99");
bool isTrue = Convert.ToBoolean("true");

// Cast vs Convert:
// (int)value — nhanh, nhưng chỉ cho numeric types
// Convert.ToInt32 — linh hoạt, xử lý null (trả về 0)
// int.Parse — cho string, throws exception nếu lỗi
// int.TryParse — cho string, an toàn nhất ✅
```

### 2.5 Quy tắc đặt tên biến

```csharp
// ✅ Đúng - camelCase cho biến cục bộ
int studentAge = 20;
string firstName = "John";
bool isValid = true;

// ❌ Sai
int 1stNumber;       // Không bắt đầu bằng số
int student age;     // Không có khoảng trắng
int class;           // Không dùng từ khóa (reserved keyword)
int STUDENT_AGE;     // Đây là style C/C++, không phải C#
```

**Quy ước Clean Code cho tên biến:**
- Dùng **camelCase** cho biến cục bộ: `myVariable`
- Tên phải **có ý nghĩa**: `customerName` thay vì `cn`
- Tên phải **phản ánh mục đích** của biến
- Tránh viết tắt: `numberOfStudents` thay vì `numStd`
- Boolean nên bắt đầu bằng `is`, `has`, `can`: `isActive`, `hasPermission`

### 2.6 Constants (Hằng số)

```csharp
// const: giá trị KHÔNG BAO GIỜ thay đổi (compile-time)
const double PI = 3.14159265358979;
const int MAX_STUDENTS = 30;
const string APP_NAME = "MyApp";

// PI = 4;  // ❌ LỖI! Không thể gán lại const

// readonly: gán 1 lần, tại runtime (trong class)
readonly DateTime createdAt = DateTime.Now;
```

### 2.7 Implicitly typed variables (var)

```csharp
var age = 25;           // Compiler suy ra: int
var name = "Nghĩa";     // Compiler suy ra: string
var price = 19.99;       // Compiler suy ra: double
var items = new List<string>();  // Compiler suy ra: List<string>

// ❌ Không thể dùng var mà không gán giá trị
// var x;  // Lỗi!

// ❌ Không thể đổi kiểu sau khi khai báo
var number = 10;
// number = "text";  // Lỗi! number đã là int

// Khi nào dùng var?
// ✅ Khi kiểu RÕ RÀNG từ vế phải:
var list = new List<string>();  // Rõ ràng là List<string>
var dict = new Dictionary<string, int>();

// ❌ Khi kiểu KHÔNG rõ ràng:
// var result = GetData();  // GetData trả về gì? Không rõ!
// int result = GetData();  // Rõ hơn
```

> **Lưu ý**: `var` là **implicit typing**, KHÔNG phải **dynamic typing**. Kiểu được xác định tại **compile time**, không phải runtime.

### ❓ Câu hỏi kiểm tra

1. Sự khác biệt giữa `int`, `long`, `float`, `double`, `decimal`?
2. Tại sao không nên dùng `double` cho tiền tệ?
3. `var` có phải là dynamic type không? Giải thích.
4. Sự khác nhau giữa implicit conversion và explicit conversion?
5. `const` và `readonly` khác nhau thế nào?

### 💼 Câu hỏi phỏng vấn

> **Q: What is the difference between `float`, `double`, and `decimal`?**
> A: `float` (4 bytes, ~7 digits) và `double` (8 bytes, ~15 digits) là binary floating-point → nhanh nhưng có sai số. `decimal` (16 bytes, 28-29 digits) là decimal floating-point → chậm hơn nhưng chính xác tuyệt đối, dùng cho tài chính.

> **Q: What is the difference between `var` and `dynamic`?**
> A: `var` xác định kiểu tại compile-time (type-safe). `dynamic` xác định kiểu tại runtime (có thể gây runtime error). `var` chỉ là syntactic sugar, `dynamic` thực sự bypass type checking.

> **Q: What happens when you cast a `double` to an `int`?**
> A: Phần thập phân bị CẮT BỎ (truncated), không phải làm tròn. `(int)3.9` = `3`, không phải `4`.

---

## 3. Toán Tử (Operators)

### 3.1 Toán tử số học

```csharp
int a = 10, b = 3;

int sum = a + b;       // 13
int diff = a - b;      // 7
int product = a * b;   // 30
int quotient = a / b;  // 3 (chia nguyên!)
int remainder = a % b; // 1 (phép chia dư - modulo)

// ⚠️ Chia nguyên vs chia thực
double result = 10.0 / 3;  // 3.3333... (chia thực vì có double)
int intResult = 10 / 3;     // 3 (chia nguyên vì cả hai là int)

// ⚠️ Chia cho 0
// int crash = 10 / 0;      // ❌ DivideByZeroException!
double inf = 10.0 / 0;      // Infinity (double không throw exception)
double nan = 0.0 / 0;       // NaN (Not a Number)
```

**Modulo (%) — Ứng dụng thực tế:**

```csharp
// Kiểm tra số chẵn/lẻ
bool isEven = number % 2 == 0;

// Lấy chữ số cuối
int lastDigit = 12345 % 10;  // 5

// Giới hạn giá trị trong phạm vi (wrap around)
int index = counter % array.Length;  // Luôn trong [0, Length-1]

// Kiểm tra chia hết
bool divisibleBy3 = number % 3 == 0;
```

### 3.2 Toán tử gán kết hợp

```csharp
int x = 10;
x += 5;   // x = x + 5  → 15
x -= 3;   // x = x - 3  → 12
x *= 2;   // x = x * 2  → 24
x /= 4;   // x = x / 4  → 6
x %= 5;   // x = x % 5  → 1
```

### 3.3 Increment / Decrement — CHI TIẾT

```csharp
int a = 5;

// Prefix: tăng TRƯỚC, rồi trả về giá trị MỚI
int b = ++a;  // a = 6, b = 6

// Postfix: trả về giá trị CŨ, rồi mới tăng
int c = a++;  // c = 6 (giá trị cũ), a = 7

// Ví dụ dễ nhầm:
int x = 5;
Console.WriteLine(x++);  // In ra 5, SAU ĐÓ x = 6
Console.WriteLine(++x);  // x = 7, In ra 7

// ⚠️ TRÁNH dùng trong biểu thức phức tạp:
// int result = x++ + ++x;  // Khó đọc, dễ sai!
```

### 3.4 Toán tử so sánh

```csharp
int a = 5, b = 10;

bool eq = (a == b);    // false (bằng nhau?)
bool neq = (a != b);   // true  (khác nhau?)
bool gt = (a > b);     // false (lớn hơn?)
bool lt = (a < b);     // true  (nhỏ hơn?)
bool gte = (a >= b);   // false (lớn hơn hoặc bằng?)
bool lte = (a <= b);   // true  (nhỏ hơn hoặc bằng?)

// ⚠️ == vs = (lỗi phổ biến nhất!)
// if (x = 5)   // ❌ Gán! Không phải so sánh!
// if (x == 5)  // ✅ So sánh
```

### 3.5 Toán tử logic

```csharp
bool a = true, b = false;

bool and = a && b;     // false (VÀ - cả hai phải true)
bool or = a || b;      // true  (HOẶC - ít nhất 1 true)
bool not = !a;          // false (PHỦ ĐỊNH - đảo ngược)

// Short-circuit evaluation (đánh giá ngắn mạch)
// && : nếu vế trái false → KHÔNG kiểm tra vế phải
// || : nếu vế trái true → KHÔNG kiểm tra vế phải

// Ứng dụng: tránh null exception
string name = null;
if (name != null && name.Length > 0)  // An toàn! 
{
    // Nếu name == null, vế phải KHÔNG được gọi
    // → không bị NullReferenceException
}
```

**Bảng chân trị (Truth Table):**

```
 A     B     A && B   A || B   !A
───────────────────────────────────
true  true   true     true     false
true  false  false    true     false
false true   false    true     true
false false  false    false    true
```

### 3.6 Ternary Operator (Toán tử ba ngôi)

```csharp
// condition ? valueIfTrue : valueIfFalse
int age = 20;
string status = age >= 18 ? "Người lớn" : "Trẻ em";

// Tương đương:
string status2;
if (age >= 18)
    status2 = "Người lớn";
else
    status2 = "Trẻ em";

// Dùng trong interpolation
Console.WriteLine($"Bạn {(age >= 18 ? "đủ" : "chưa đủ")} tuổi bầu cử");

// ⚠️ ĐỪNG lồng ternary quá sâu — khó đọc!
// ❌ string s = a ? b ? "x" : "y" : c ? "z" : "w";
// ✅ Dùng if-else cho logic phức tạp
```

### 3.7 Null-related Operators (sẽ học sâu ở phần sau)

```csharp
// ?? (null-coalescing): trả về vế trái nếu != null, ngược lại vế phải
string name = null;
string displayName = name ?? "Anonymous";  // "Anonymous"

// ??= (null-coalescing assignment): gán CHỈ KHI null
name ??= "Default";  // name = "Default" (vì name là null)

// ?. (null-conditional): gọi method CHỈ KHI không null
int? length = name?.Length;  // null nếu name == null
```

### ❓ Câu hỏi kiểm tra

1. `10 / 3` trong C# cho kết quả bao nhiêu? Tại sao?
2. `10.0 / 0` cho kết quả gì? `10 / 0` thì sao?
3. `++x` và `x++` khác nhau thế nào?
4. Short-circuit evaluation là gì? Cho ví dụ.
5. `5 % 3` = ? `7 % 2` = ? `-7 % 2` = ?

### 💼 Câu hỏi phỏng vấn

> **Q: What is the difference between `==` and `.Equals()` in C#?**
> A: `==` so sánh reference (cho reference types) hoặc value (cho value types) mặc định. `.Equals()` so sánh value. Nhưng `string` override `==` để so sánh value. Classes có thể override cả hai.

> **Q: What is short-circuit evaluation?**
> A: Với `&&`, nếu vế trái false thì vế phải không được đánh giá. Với `||`, nếu vế trái true thì vế phải không được đánh giá. Giúp tối ưu hiệu suất và tránh lỗi runtime.

---

## 4. Nhập Xuất Dữ Liệu (User Input/Output)

### 4.1 Console Output

```csharp
Console.Write("Hello ");         // Không xuống dòng
Console.WriteLine("World!");     // Có xuống dòng
Console.WriteLine();             // Chỉ xuống dòng

// Escape characters
Console.WriteLine("Line 1\nLine 2");     // \n = xuống dòng
Console.WriteLine("Tab\there");           // \t = tab
Console.WriteLine("He said \"Hi\"");      // \" = dấu ngoặc kép
Console.WriteLine("Path: C:\\Users");     // \\ = backslash

// Verbatim string (tránh escape)
Console.WriteLine(@"Path: C:\Users\Documents");  // Không cần \\

// Màu console
Console.ForegroundColor = ConsoleColor.Green;
Console.WriteLine("Thành công!");
Console.ResetColor();
```

### 4.2 Console Input

```csharp
Console.Write("Nhập tên: ");
string name = Console.ReadLine();  // Đọc CHUỖI từ người dùng

Console.Write("Nhập tuổi: ");
string ageText = Console.ReadLine();
int age = int.Parse(ageText);  // Chuyển chuỗi → số nguyên

// ⚠️ int.Parse sẽ THROW EXCEPTION nếu chuỗi không phải số!

// ✅ An toàn hơn: TryParse
Console.Write("Nhập số: ");
if (int.TryParse(Console.ReadLine(), out int number))
{
    Console.WriteLine($"Số bạn nhập: {number}");
}
else
{
    Console.WriteLine("Đó không phải là số!");
}

// Đọc 1 phím (không cần Enter)
ConsoleKeyInfo key = Console.ReadKey();
Console.WriteLine($"\nBạn bấm: {key.KeyChar}");
```

### 4.3 String Interpolation — Chi tiết

```csharp
string name = "Nghĩa";
int age = 25;

// Cách 1: Nối chuỗi (concatenation) — TRÁNH dùng
string msg1 = "Tên: " + name + ", Tuổi: " + age;

// Cách 2: String interpolation — NÊN dùng ✅
string msg2 = $"Tên: {name}, Tuổi: {age}";

// Cách 3: String.Format
string msg3 = string.Format("Tên: {0}, Tuổi: {1}", name, age);

// Biểu thức trong {}
Console.WriteLine($"Năm sinh: {2026 - age}");
Console.WriteLine($"Tên viết hoa: {name.ToUpper()}");
Console.WriteLine($"Chẵn? {(age % 2 == 0 ? "Có" : "Không")}");

// Formatting
double pi = 3.14159265;
Console.WriteLine($"Pi ≈ {pi:F2}");          // 3.14 (2 decimal places)
Console.WriteLine($"Tiền: {1234.5:C}");      // $1,234.50 (currency)
Console.WriteLine($"Phần trăm: {0.85:P0}");  // 85% (percent)
Console.WriteLine($"Pad: {42:D5}");           // 00042 (padded)

// Alignment
Console.WriteLine($"{"Tên",-15}{"Tuổi",5}");   // Left & right align
Console.WriteLine($"{"Nghĩa",-15}{25,5}");
Console.WriteLine($"{"Mai",-15}{22,5}");
```

### ❓ Câu hỏi kiểm tra

1. `Console.Write` vs `Console.WriteLine` — khác nhau thế nào?
2. Làm sao in dấu `"` trong chuỗi?
3. `@"C:\path"` và `"C:\\path"` — cái nào dễ đọc hơn? Khi nào dùng?
4. `int.Parse` vs `int.TryParse` — nên dùng cái nào? Tại sao?

---

## 5. Boolean & Câu Lệnh Điều Kiện

### 5.1 if / else if / else

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("Xuất sắc (A)");
}
else if (score >= 80)
{
    Console.WriteLine("Giỏi (B)");    // ← Chạy dòng này
}
else if (score >= 70)
{
    Console.WriteLine("Khá (C)");
}
else if (score >= 60)
{
    Console.WriteLine("Trung bình (D)");
}
else
{
    Console.WriteLine("Không đạt (F)");
}

// ⚠️ Thứ tự QUAN TRỌNG!
// ❌ Sai thứ tự:
if (score >= 60) { /* ... */ }      // Mọi score >= 60 vào đây!
else if (score >= 90) { /* ... */ }  // KHÔNG BAO GIỜ chạy!
```

### 5.2 Scope (Phạm vi)

```csharp
if (true)
{
    int x = 10;  // x chỉ tồn tại trong block này
    Console.WriteLine(x);  // OK
}
// Console.WriteLine(x);  // ❌ LỖI! x không tồn tại ở đây

// Nếu cần dùng biến SAU if:
int result;  // Khai báo TRƯỚC if
if (condition)
    result = 1;
else
    result = 2;
Console.WriteLine(result);  // OK
```

### 5.3 Switch Statement

```csharp
string dayOfWeek = "Monday";

switch (dayOfWeek)
{
    case "Monday":
    case "Tuesday":
    case "Wednesday":
    case "Thursday":
    case "Friday":
        Console.WriteLine("Ngày làm việc");
        break;  // BẮT BUỘC phải có break!
    case "Saturday":
    case "Sunday":
        Console.WriteLine("Cuối tuần");
        break;
    default:
        Console.WriteLine("Không hợp lệ");
        break;
}
```

### 5.4 Switch Expression (C# 8+, QUAN TRỌNG)

```csharp
// Gọn hơn switch statement rất nhiều
string dayType = dayOfWeek switch
{
    "Monday" or "Tuesday" or "Wednesday" or "Thursday" or "Friday" 
        => "Ngày làm việc",
    "Saturday" or "Sunday" 
        => "Cuối tuần",
    _ => "Không hợp lệ"  // _ là default (discard pattern)
};

// Pattern matching trong switch
object obj = 42;
string description = obj switch
{
    int n when n > 0 => $"Số dương: {n}",
    int n when n < 0 => $"Số âm: {n}",
    int => "Số 0",
    string s => $"Chuỗi: {s}",
    null => "Null",
    _ => "Không xác định"
};
```

### ❓ Câu hỏi kiểm tra

1. Nếu quên `break` trong switch, chuyện gì xảy ra?
2. Switch expression và switch statement khác nhau thế nào?
3. `_` trong switch expression nghĩa là gì?

### 💼 Câu hỏi phỏng vấn

> **Q: Can a switch statement work with `string` in C#?**
> A: Có. C# switch hỗ trợ `int`, `string`, `char`, `bool`, `enum`, và bất kỳ type nào có `Equals()`. Khác với C/C++ chỉ hỗ trợ integral types.

---

## 6. Phương Thức (Methods)

### 6.1 Anatomy of a Method

```csharp
//   access   return   method name   parameters
//   modifier type
     static   int      Add           (int a, int b)
{
    return a + b;   // method body
}
```

### 6.2 Void vs Non-void Methods

```csharp
// Void: KHÔNG trả về giá trị
static void PrintGreeting(string name)
{
    Console.WriteLine($"Xin chào, {name}!");
    // return; ← tùy chọn, dùng để thoát sớm
}

// Non-void: PHẢI trả về giá trị
static int Add(int a, int b)
{
    return a + b;  // BẮT BUỘC có return
}

static double CalculateArea(double radius)
{
    return Math.PI * radius * radius;
}

// Sử dụng
PrintGreeting("Nghĩa");             // Gọi void method
int sum = Add(3, 5);                  // Lưu kết quả
double area = CalculateArea(5);       // area = 78.54...
Console.WriteLine(Add(10, 20));       // Dùng trực tiếp: 30
```

### 6.3 Parameters — Chi tiết

```csharp
// Optional parameters (giá trị mặc định)
static void Greet(string name, string greeting = "Xin chào")
{
    Console.WriteLine($"{greeting}, {name}!");
}
Greet("Nghĩa");                // "Xin chào, Nghĩa!"
Greet("Nghĩa", "Hello");      // "Hello, Nghĩa!"

// Named parameters
static void CreateUser(string name, int age, string city = "HN")
{
    Console.WriteLine($"{name}, {age}, {city}");
}
CreateUser(age: 25, name: "Nghĩa");  // Đổi thứ tự bằng named params

// out parameter: method trả về NHIỀU giá trị
static bool TryDivide(int a, int b, out int result)
{
    if (b == 0) { result = 0; return false; }
    result = a / b;
    return true;
}

if (TryDivide(10, 3, out int quotient))
    Console.WriteLine($"Kết quả: {quotient}");  // 3
```

### 6.4 Method Overloading

```csharp
// CÙNG TÊN, KHÁC PARAMETERS
static int Add(int a, int b) => a + b;
static double Add(double a, double b) => a + b;
static int Add(int a, int b, int c) => a + b + c;

// Compiler tự tìm method phù hợp dựa trên arguments
int r1 = Add(3, 5);          // Gọi Add(int, int)
double r2 = Add(3.0, 5.0);   // Gọi Add(double, double)
int r3 = Add(1, 2, 3);       // Gọi Add(int, int, int)
```

### 6.5 Expression-bodied Methods (C# 6+)

```csharp
// Khi method chỉ có 1 expression
static int Add(int a, int b) => a + b;
static bool IsEven(int n) => n % 2 == 0;
static string GetGreeting(string name) => $"Hello, {name}!";

// Tương đương:
static int Add(int a, int b)
{
    return a + b;
}
```

### 6.6 Parsing: Chuyển đổi chuỗi sang số

```csharp
// int.Parse — throws exception nếu thất bại
int number = int.Parse("123");      // OK → 123
// int bad = int.Parse("abc");      // ❌ FormatException!
// int bad2 = int.Parse("");        // ❌ FormatException!
// int bad3 = int.Parse(null);      // ❌ ArgumentNullException!

// int.TryParse — trả về bool, AN TOÀN hơn ✅
bool success = int.TryParse("123", out int result);
// success = true, result = 123

bool fail = int.TryParse("abc", out int result2);
// fail = false, result2 = 0 (default)

// Pattern thường dùng:
Console.Write("Nhập số: ");
string input = Console.ReadLine();
if (int.TryParse(input, out int userNumber))
    Console.WriteLine($"Bạn nhập: {userNumber}");
else
    Console.WriteLine("Đó không phải là số!");

// Các kiểu khác cũng có Parse/TryParse:
double.TryParse("3.14", out double pi);
bool.TryParse("true", out bool isOk);
DateTime.TryParse("2026-02-21", out DateTime date);
```

### ❓ Câu hỏi kiểm tra

1. Method overloading là gì? Compiler chọn overload nào?
2. Optional parameter PHẢI đặt ở đâu trong danh sách?
3. `out` parameter khác `return` thế nào? Khi nào dùng?
4. `=>` (expression-bodied) dùng khi nào?

### 💼 Câu hỏi phỏng vấn

> **Q: What is method overloading vs method overriding?**
> A: Overloading = cùng tên, KHÁC parameters, trong CÙNG class. Overriding = cùng tên, CÙNG parameters, trong class CON kế thừa (dùng `virtual`/`override`).

> **Q: Can you overload methods that differ only in return type?**
> A: KHÔNG. Overloading yêu cầu khác parameter list. Khác return type không đủ.

## 7. Vòng Lặp (Loops)

### 7.1 While Loop

```csharp
// Lặp KHI điều kiện còn đúng
int count = 0;
while (count < 5)
{
    Console.WriteLine($"Lần lặp: {count}");
    count++;  // QUAN TRỌNG: phải thay đổi điều kiện!
}
// Output: 0, 1, 2, 3, 4

// ⚠️ Vòng lặp vô hạn — cẩn thận!
// while (true) { ... }  // Chạy mãi mãi trừ khi break

// Ví dụ thực tế: Validation loop
int number;
while (true)
{
    Console.Write("Nhập số dương: ");
    if (int.TryParse(Console.ReadLine(), out number) && number > 0)
        break;  // Thoát khi input hợp lệ
    Console.WriteLine("Không hợp lệ! Thử lại.");
}
```

### 7.2 Do...While Loop

```csharp
// Thực hiện ÍT NHẤT 1 LẦN, sau đó kiểm tra điều kiện
string input;
do
{
    Console.Write("Nhập 'exit' để thoát: ");
    input = Console.ReadLine();
    Console.WriteLine($"Bạn nhập: {input}");
} while (input != "exit");

// Khác biệt while vs do-while:
// while: kiểm tra TRƯỚC → có thể không chạy lần nào
// do-while: kiểm tra SAU → luôn chạy ÍT NHẤT 1 lần
```

**So sánh:**

```
while:                          do-while:
┌─────────────┐                 ┌─────────────┐
│ Kiểm tra    │─false─→ END     │ Thực hiện   │
│ điều kiện   │                  │ body        │
└──────┬──────┘                  └──────┬──────┘
       │ true                           │
┌──────┴──────┐                  ┌──────┴──────┐
│ Thực hiện   │                  │ Kiểm tra    │─false─→ END
│ body        │                  │ điều kiện   │
└──────┬──────┘                  └──────┬──────┘
       │                                │ true
       └─── quay lại kiểm tra           └─── quay lại body
```

### 7.3 For Loop

```csharp
// for (khởi tạo; điều kiện; bước nhảy)
for (int i = 0; i < 10; i++)
{
    Console.Write($"{i} ");  // 0 1 2 3 4 5 6 7 8 9
}

// Đếm ngược
for (int i = 10; i >= 1; i--)
{
    Console.Write($"{i} ");  // 10 9 8 7 6 5 4 3 2 1
}

// Bước nhảy 2 (số chẵn)
for (int i = 0; i <= 20; i += 2)
{
    Console.Write($"{i} ");  // 0 2 4 6 8 10 12 14 16 18 20
}

// Khi nào dùng for vs while?
// for: biết trước SỐ LẦN lặp
// while: lặp đến khi ĐIỀU KIỆN thay đổi (không biết trước số lần)
```

### 7.4 Foreach Loop

```csharp
string[] fruits = { "Táo", "Cam", "Chuối", "Xoài" };

// foreach: duyệt qua collection MÀ KHÔNG CẦN INDEX
foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}

// ⚠️ KHÔNG THỂ sửa collection trong foreach!
// foreach (string fruit in fruits)
// {
//     fruits[0] = "Bưởi";  // ❌ Không thể sửa array trong foreach!
// }

// Khi nào dùng for vs foreach?
// for: cần INDEX (vị trí phần tử)
// foreach: chỉ cần VALUE (giá trị phần tử) — gọn hơn, an toàn hơn
```

### 7.5 Break & Continue

```csharp
// break: THOÁT khỏi vòng lặp ngay lập tức
for (int i = 0; i < 100; i++)
{
    if (i == 5) break;       
    Console.Write($"{i} ");   // 0 1 2 3 4
}

// continue: BỎ QUA lần lặp hiện tại, chuyển sang lần tiếp
for (int i = 0; i < 10; i++)
{
    if (i % 2 == 0) continue; // Bỏ qua số chẵn
    Console.Write($"{i} ");    // 1 3 5 7 9
}

// Ví dụ thực tế: Tìm phần tử trong mảng
int[] numbers = { 3, 7, 2, 9, 4, 6, 8 };
int target = 9;
int foundIndex = -1;

for (int i = 0; i < numbers.Length; i++)
{
    if (numbers[i] == target)
    {
        foundIndex = i;
        break;  // Tìm thấy → dừng luôn, không cần duyệt tiếp
    }
}
Console.WriteLine(foundIndex == -1 ? "Không tìm thấy" : $"Tìm thấy tại index {foundIndex}");
```

### 7.6 Nested Loops (Vòng lặp lồng nhau)

```csharp
// Bảng cửu chương
for (int i = 2; i <= 9; i++)
{
    Console.WriteLine($"--- Bảng {i} ---");
    for (int j = 1; j <= 10; j++)
    {
        Console.WriteLine($"  {i} x {j} = {i * j}");
    }
}

// Vẽ hình tam giác
// *
// **
// ***
// ****
// *****
for (int i = 1; i <= 5; i++)
{
    for (int j = 0; j < i; j++)
        Console.Write("*");
    Console.WriteLine();
}

// ⚠️ break trong nested loop chỉ thoát VÒNG LẶP TRONG
for (int i = 0; i < 3; i++)
{
    for (int j = 0; j < 3; j++)
    {
        if (j == 1) break;  // Chỉ thoát vòng for j
        Console.Write($"({i},{j}) ");
    }
}
// Output: (0,0) (1,0) (2,0)
```

### 7.7 Hiệu suất & Common Mistakes

```csharp
// ❌ Tính toán lặp trong điều kiện
for (int i = 0; i < list.Count; i++) // list.Count gọi mỗi lần!

// ✅ Cache giá trị
int count = list.Count;
for (int i = 0; i < count; i++)

// ❌ Off-by-one error (sai 1 đơn vị) — BUG phổ biến nhất!
int[] arr = { 1, 2, 3, 4, 5 };
// for (int i = 0; i <= arr.Length; i++)  // ❌ i = 5 → IndexOutOfRange!
for (int i = 0; i < arr.Length; i++)     // ✅ i chạy từ 0 đến 4

// ❌ Infinite loop (vòng lặp vô hạn)
// int i = 0;
// while (i < 10) { Console.WriteLine(i); }  // Quên i++!
```

### ❓ Câu hỏi kiểm tra

1. `while` và `do-while` khác nhau thế nào? Cho ví dụ khi PHẢI dùng `do-while`.
2. `break` trong nested loop ảnh hưởng vòng lặp nào?
3. `for` vs `foreach` — khi nào dùng cái nào?
4. Off-by-one error là gì? Làm sao tránh?
5. Viết vòng lặp in ra: `5 4 3 2 1 Liftoff!`

### 💼 Câu hỏi phỏng vấn

> **Q: What is the difference between `for` and `foreach`?**
> A: `for` dùng index, có thể sửa collection, linh hoạt hơn. `foreach` dùng iterator, gọn hơn, an toàn hơn (read-only), và hoạt động với bất kỳ `IEnumerable`. Ưu tiên `foreach` trừ khi cần index.

> **Q: How do you break out of a nested loop?**
> A: `break` chỉ thoát vòng lặp gần nhất. Để thoát toàn bộ: (1) dùng biến flag `bool found = true`, (2) dùng `goto` (không khuyến khích), (3) extract thành method riêng rồi `return`.

---

## 8. Mảng (Arrays)

### 8.1 Khai báo và sử dụng

```csharp
// Khai báo mảng
int[] numbers = new int[5];          // 5 phần tử, mặc định = 0
string[] names = new string[3];      // 3 phần tử, mặc định = null
bool[] flags = new bool[4];          // 4 phần tử, mặc định = false

// Khai báo + khởi tạo
int[] scores = { 90, 85, 78, 92, 88 };
int[] scores2 = new int[] { 90, 85, 78, 92, 88 };
int[] scores3 = new int[5] { 90, 85, 78, 92, 88 };

// Truy cập (index bắt đầu từ 0!)
int first = scores[0];    // 90
int last = scores[4];     // 88
scores[2] = 100;          // Gán giá trị mới

// Thuộc tính Length
int size = scores.Length;  // 5

// ⚠️ IndexOutOfRangeException — lỗi runtime phổ biến!
// scores[5] = 0;  // ❌ Index 5 không tồn tại! (max index = 4)
// scores[-1] = 0; // ❌ Negative index không hợp lệ!
```

### 8.2 Duyệt mảng

```csharp
string[] fruits = { "Táo", "Cam", "Chuối", "Xoài" };

// Cách 1: for loop (khi cần index)
for (int i = 0; i < fruits.Length; i++)
{
    Console.WriteLine($"[{i}] {fruits[i]}");
}

// Cách 2: foreach loop (khi chỉ cần giá trị)
foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}

// Cách 3: Index from end (C# 8+)
string lastFruit = fruits[^1];   // "Xoài" (phần tử cuối)
string secondLast = fruits[^2];  // "Chuối"

// Cách 4: Range (C# 8+)
string[] middle = fruits[1..3];  // {"Cam", "Chuối"} (index 1 đến 2)
string[] firstTwo = fruits[..2]; // {"Táo", "Cam"}
string[] lastTwo = fruits[^2..]; // {"Chuối", "Xoài"}
```

### 8.3 Array Methods

```csharp
int[] numbers = { 5, 3, 8, 1, 9, 2, 7 };

// Sắp xếp
Array.Sort(numbers);     // { 1, 2, 3, 5, 7, 8, 9 }

// Đảo ngược
Array.Reverse(numbers);  // { 9, 8, 7, 5, 3, 2, 1 }

// Tìm kiếm
int index = Array.IndexOf(numbers, 5);  // Trả về index, hoặc -1

// Kiểm tra tồn tại
bool exists = Array.Exists(numbers, n => n > 8);  // true

// Copy
int[] copy = new int[numbers.Length];
Array.Copy(numbers, copy, numbers.Length);

// Fill (lấp đầy)
int[] zeros = new int[10];
Array.Fill(zeros, 42);  // Tất cả = 42

// Min, Max (cần using System.Linq)
int max = numbers.Max();  // 9
int min = numbers.Min();  // 1
int sum = numbers.Sum();  // 35
double avg = numbers.Average();  // 5.0
```

### 8.4 Mảng đa chiều

```csharp
// 2D Array (ma trận)
int[,] matrix = {
    { 1, 2, 3 },
    { 4, 5, 6 },
    { 7, 8, 9 }
};

int center = matrix[1, 1];  // 5
int rows = matrix.GetLength(0);  // 3
int cols = matrix.GetLength(1);  // 3

// Duyệt 2D
for (int row = 0; row < rows; row++)
{
    for (int col = 0; col < cols; col++)
        Console.Write($"{matrix[row, col],4}");
    Console.WriteLine();
}

// Jagged Array (mảng của mảng — mỗi hàng có thể khác chiều dài)
int[][] jagged = new int[3][];
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5 };
jagged[2] = new int[] { 6 };
// jagged[1][2] = 5
```

### ❓ Câu hỏi kiểm tra

1. Mảng có thể thay đổi kích thước sau khi tạo không?
2. `fruits[^1]` nghĩa là gì?
3. `int[,]` vs `int[][]` — khác nhau thế nào?
4. Default value của `int[]`, `string[]`, `bool[]` là gì?
5. Viết code tìm phần tử lớn nhất trong mảng KHÔNG dùng `.Max()`.

### 💼 Câu hỏi phỏng vấn

> **Q: What is the difference between `Array` and `List<T>` in C#?**
> A: Array có kích thước cố định, truy cập nhanh hơn, ít overhead. List<T> có kích thước động, hỗ trợ Add/Remove, dễ dùng hơn. Nội bộ List<T> dùng Array và tự mở rộng khi đầy (capacity doubles).

> **Q: What is the difference between a jagged array and a multi-dimensional array?**
> A: Jagged (`int[][]`): mảng của mảng, mỗi hàng có thể khác length, lưu trên heap riêng biệt. Multi-dimensional (`int[,]`): 1 block memory liên tục, tất cả hàng cùng length. Jagged thường nhanh hơn do cách .NET tối ưu.

---

## 9. List (Danh sách)

### 9.1 Khác biệt List vs Array

| Đặc điểm | Array | List |
|-----------|-------|------|
| Kích thước | **Cố định** | **Động** (tự mở rộng) |
| Khai báo | `int[] arr = new int[5]` | `List<int> list = new()` |
| Thêm | Không thể | `list.Add(item)` |
| Xóa | Không thể | `list.Remove(item)` |
| Hiệu suất | Nhanh hơn (ít overhead) | Linh hoạt hơn |
| Khi nào | Biết trước số lượng | Số lượng thay đổi |

### 9.2 Các thao tác đầy đủ

```csharp
// Khai báo
List<string> names = new List<string>();
List<string> names2 = new() { "An", "Bình", "Chi" };  // C# 9+
List<int> numbers = [1, 2, 3, 4, 5];  // C# 12+

// === THÊM ===
names.Add("Nghĩa");                           // Thêm cuối
names.AddRange(new[] { "Mai", "Lan" });        // Thêm nhiều
names.Insert(0, "Hùng");                       // Chèn tại vị trí
names.InsertRange(2, new[] { "Đức", "Hà" });  // Chèn nhiều

// === TRUY CẬP ===
string first = names[0];
string last = names[^1];  // C# 8+

// === SỬA ===
names[0] = "Hải";

// === XÓA ===
names.Remove("Lan");          // Xóa theo giá trị (phần tử ĐẦU TIÊN)
names.RemoveAt(0);            // Xóa theo index
names.RemoveAll(n => n.Length < 3);  // Xóa theo điều kiện
names.RemoveRange(0, 2);      // Xóa 2 phần tử từ index 0
names.Clear();                 // Xóa TẤT CẢ

// === KIỂM TRA ===
bool has = names.Contains("Mai");  // true/false
int idx = names.IndexOf("Mai");     // index hoặc -1
int lastIdx = names.LastIndexOf("Mai");

// === ĐẾM ===
int count = names.Count;  // (không phải Length!)

// === SẮP XẾP ===
names.Sort();                    // A-Z
names.Sort((a, b) => b.CompareTo(a));  // Z-A
names.Reverse();                 // Đảo ngược

// === CHUYỂN ĐỔI ===
string[] array = names.ToArray();
List<string> copy = new(names);  // Shallow copy

// === DUYỆT ===
foreach (string name in names)
    Console.WriteLine(name);

// Với index
for (int i = 0; i < names.Count; i++)
    Console.WriteLine($"[{i}] {names[i]}");

// ForEach method
names.ForEach(name => Console.WriteLine(name));
```

### 9.3 List Internal — Hiểu sâu

```csharp
// List<T> nội bộ dùng Array
// Khi array đầy → tạo array MỚI gấp đôi capacity → copy sang

var list = new List<int>();
Console.WriteLine($"Count: {list.Count}, Capacity: {list.Capacity}");
// Count: 0, Capacity: 0

list.Add(1);
Console.WriteLine($"Count: {list.Count}, Capacity: {list.Capacity}");
// Count: 1, Capacity: 4 (tự allocate)

for (int i = 2; i <= 5; i++) list.Add(i);
Console.WriteLine($"Count: {list.Count}, Capacity: {list.Capacity}");
// Count: 5, Capacity: 8 (doubled!)

// ✅ Nếu biết trước số lượng → chỉ định capacity để tránh resize
var efficient = new List<int>(1000);  // Allocate 1 lần cho 1000 items
```

### ❓ Câu hỏi kiểm tra

1. `Count` vs `Capacity` trong List khác nhau thế nào?
2. Khi List đầy và thêm phần tử mới, chuyện gì xảy ra bên trong?
3. `Remove("item")` xóa TẤT CẢ hay chỉ phần tử ĐẦU TIÊN?
4. Làm sao sao chép (clone) 1 List?

---

## 10. Debug với Breakpoints

### 10.1 Cách đặt breakpoint

```
1. Click vào lề trái (gutter) cạnh số dòng → chấm đỏ 🔴
2. Hoặc đặt con trỏ vào dòng → F9
3. Chạy Debug: F5
4. Chạy KHÔNG debug: Ctrl+F5
```

### 10.2 Các công cụ debug

| Phím | Chức năng | Mô tả |
|------|-----------|-------|
| **F5** | Continue | Chạy tiếp đến breakpoint kế |
| **F10** | Step Over | Chạy dòng hiện tại (không vào method) |
| **F11** | Step Into | Đi VÀO method đang gọi |
| **Shift+F11** | Step Out | Thoát ra khỏi method hiện tại |
| **Shift+F5** | Stop | Dừng debug |

### 10.3 Watch, Locals, Conditional Breakpoints

```
Locals window: Xem tất cả biến cục bộ
Watch window:  Thêm biểu thức tùy ý để theo dõi
Hover:         Di chuột lên biến → xem giá trị

Conditional Breakpoint:
  - Right-click breakpoint → Conditions
  - Nhập điều kiện: i == 50 → chỉ dừng khi i = 50
  - Rất hữu ích khi debug vòng lặp lớn!

Hit Count:
  - Breakpoint chỉ active sau N lần chạy qua
```

### 💼 Câu hỏi phỏng vấn

> **Q: How do you debug a problem that only occurs in production?**
> A: Logging (Serilog, NLog), Application Insights (monitoring), Remote debugging, Memory dumps, và Structured logging với correlation IDs.

---

## 11. Char & Comments

### Char

```csharp
char letter = 'A';      // Dùng dấu nháy đơn ''
char digit = '7';
char newline = '\n';

// Kiểm tra
char.IsLetter('A');       // true
char.IsDigit('7');        // true
char.IsWhiteSpace(' ');   // true
char.IsUpper('A');        // true
char.IsLower('a');        // true
char.IsPunctuation('!');  // true

// Chuyển đổi
char.ToLower('A');  // 'a'
char.ToUpper('z');  // 'Z'

// Unicode
char vi = 'Ă';           // Vietnamese character
int code = (int)'A';     // 65 (ASCII/Unicode)
char fromCode = (char)65; // 'A'
```

### Comments — Best Practices

```csharp
// Comment một dòng

/* Comment
   nhiều
   dòng */

/// <summary>
/// XML Documentation — cho public APIs
/// </summary>
/// <param name="name">Tên người dùng</param>
/// <returns>Lời chào</returns>
static string Greet(string name) => $"Xin chào, {name}!";

// ✅ GOOD: giải thích TẠI SAO
// Using retry because the payment API is unreliable during peak hours
await RetryAsync(3, () => ProcessPayment(order));

// ❌ BAD: giải thích CÁI GÌ (code đã nói rồi)
// Increment counter by 1
counter++;

// ❌ BAD: commented-out code → dùng Git!
// var old = CalculateOld(data);
```

---

## 📝 Assignments

### Assignment 1: Simple Calculator

```csharp
Console.Write("Nhập số thứ nhất: ");
int a = int.Parse(Console.ReadLine());

Console.Write("Nhập phép tính (+, -, *, /): ");
string op = Console.ReadLine();

Console.Write("Nhập số thứ hai: ");
int b = int.Parse(Console.ReadLine());

string result = op switch
{
    "+" => $"{a} + {b} = {a + b}",
    "-" => $"{a} - {b} = {a - b}",
    "*" => $"{a} * {b} = {a * b}",
    "/" when b != 0 => $"{a} / {b} = {(double)a / b:F2}",
    "/" => "Lỗi: Không thể chia cho 0!",
    _ => $"Phép tính '{op}' không hợp lệ"
};

Console.WriteLine(result);
```

### Assignment 2: TODO List

```csharp
List<string> todos = new();
bool running = true;

while (running)
{
    Console.WriteLine("\n--- TODO LIST ---");
    Console.WriteLine("[S]how  [A]dd  [R]emove  [Q]uit");
    
    string choice = Console.ReadLine().ToUpper();
    switch (choice)
    {
        case "S": ShowTodos(todos); break;
        case "A": AddTodo(todos); break;
        case "R": RemoveTodo(todos); break;
        case "Q": running = false; break;
        default: Console.WriteLine("Lựa chọn không hợp lệ!"); break;
    }
}

static void ShowTodos(List<string> todos)
{
    if (todos.Count == 0) { Console.WriteLine("Danh sách trống!"); return; }
    for (int i = 0; i < todos.Count; i++)
        Console.WriteLine($"  [{i + 1}] {todos[i]}");
}

static void AddTodo(List<string> todos)
{
    Console.Write("Nhập TODO: ");
    string todo = Console.ReadLine();
    if (!string.IsNullOrWhiteSpace(todo))
    {
        todos.Add(todo);
        Console.WriteLine("Đã thêm!");
    }
}

static void RemoveTodo(List<string> todos)
{
    ShowTodos(todos);
    Console.Write("Nhập số cần xóa: ");
    if (int.TryParse(Console.ReadLine(), out int idx) && idx >= 1 && idx <= todos.Count)
    {
        string removed = todos[idx - 1];
        todos.RemoveAt(idx - 1);
        Console.WriteLine($"Đã xóa: {removed}");
    }
    else Console.WriteLine("Số không hợp lệ!");
}
```

---

## 📚 Bổ Sung Nhanh: Tuples (Bonus)

> Phần này là bonus ngắn để dùng cho một số bài tập. Tuples sẽ được học kỹ hơn ở [04-generics.md](./04-generics.md).

```csharp
// Tuple: nhóm nhiều giá trị trong một biến
(int, int) pair = (5, 10);
Console.WriteLine($"a={pair.Item1}, b={pair.Item2}");

// Deconstruction: tách tuple thành biến riêng
(int a, int b) = (5, 10);

// Swap nhanh bằng tuple assignment (C# 7+)
(a, b) = (b, a); // a=10, b=5
```

**Khi nào dùng?**
- Gộp nhanh nhiều giá trị trả về từ method.
- Viết code ngắn gọn trong bài tập nhỏ.
- Với domain phức tạp, vẫn nên ưu tiên tạo class/record rõ nghĩa.

---

## 🏋️ Bài Tập Thực Hành

### Mức Dễ ⭐

**BT1: Swap hai số** — Hoán đổi giá trị 2 biến KHÔNG dùng biến tạm.

```csharp
int a = 5, b = 10;
// Sau khi swap: a = 10, b = 5
// Hint: dùng phép cộng/trừ hoặc tuple assignment (xem mục bonus phía trên)
```

**BT2: FizzBuzz** — In từ 1 đến 100:
- Chia hết cho 3: in "Fizz"
- Chia hết cho 5: in "Buzz"  
- Chia hết cho cả 3 và 5: in "FizzBuzz"
- Còn lại: in số đó

**BT3: Đảo ngược chuỗi** — Nhập 1 chuỗi, in ra chuỗi đảo ngược.

### Mức Trung Bình ⭐⭐

**BT4: Tìm số nguyên tố** — Viết method `bool IsPrime(int n)` kiểm tra n có phải số nguyên tố.

**BT5: Palindrome** — Kiểm tra 1 chuỗi có phải palindrome (đọc xuôi = đọc ngược). VD: "madam", "racecar".

**BT6: Dãy Fibonacci** — In N số Fibonacci đầu tiên. VD: N=10 → 0, 1, 1, 2, 3, 5, 8, 13, 21, 34.

**BT7: Đếm ký tự** — Nhập 1 chuỗi, đếm số lượng nguyên âm, phụ âm, số, và ký tự đặc biệt.

### Mức Khó ⭐⭐⭐

**BT8: Sắp xếp mảng** — Viết Bubble Sort: sắp xếp mảng int từ nhỏ đến lớn KHÔNG dùng `Array.Sort`.

```csharp
static void BubbleSort(int[] arr)
{
    // Hint: So sánh 2 phần tử liên tiếp, swap nếu sai thứ tự
    // Lặp lại cho đến khi không còn swap nào
}
```

**BT9: Number Guessing Game** — Máy random 1 số 1-100, người chơi đoán:
- Sau mỗi lần đoán: "Cao hơn!" hoặc "Thấp hơn!"
- Đếm số lần đoán
- Hiển thị "Chúc mừng! Đoán đúng sau N lần"

**BT10: Student Grade Calculator** — Nhập N sinh viên (tên, điểm), hiển thị:
- Danh sách sorted theo điểm giảm dần
- Điểm trung bình
- Sinh viên điểm cao nhất/thấp nhất
- Số sinh viên đạt/không đạt (>= 5.0)

---

## ✅ Checklist Hoàn Thành Fundamentals

1. Tự viết một console app từ đầu (không copy) có input, validate, xử lý logic, output rõ ràng.
2. Giải thích được khác biệt giữa `value type` và `reference type` bằng ví dụ chạy thật.
3. Dùng thành thạo `if/switch`, `for/while/foreach` và biết khi nào chọn từng loại.
4. Viết được methods có `return`, `out`, `ref`, overloading trong tình huống hợp lý.
5. Xử lý an toàn user input bằng `TryParse` thay vì crash app.
6. Dùng debugger tìm được ít nhất 2 bug (1 bug logic, 1 bug runtime).
7. Hoàn thành ít nhất 6 bài tập, trong đó có tối thiểu 2 bài mức khó.

## ❓ Câu Hỏi Kiểm Tra & Phỏng Vấn (Click để xem đáp án)

**1. CIL là gì? Tại sao C# cần biên dịch sang CIL trước?**

<details><summary>💡 Đáp án</summary>

**CIL (Common Intermediate Language)** là mã trung gian. C# → CIL → Machine Code (bởi JIT). CIL là **platform-independent** — cùng 1 file .dll chạy được trên Windows, Linux, macOS. CLR trên mỗi OS sẽ JIT compile CIL thành machine code phù hợp.

```
C# source → [Roslyn Compiler] → CIL (.dll) → [JIT/CLR] → Machine Code
```
</details>

**2. JIT compilation khác AOT compilation như thế nào?**

<details><summary>💡 Đáp án</summary>

| | JIT (Just-In-Time) | AOT (Ahead-Of-Time) |
|---|---|---|
| **Khi nào?** | Runtime (method gọi lần đầu) | Build time (trước khi chạy) |
| **Startup** | Chậm hơn | Nhanh hơn |
| **Tối ưu** | Theo hardware thực tế | Generic |
| **Dùng khi** | Server apps | Mobile, IoT, serverless |
</details>

**3. .NET Framework vs .NET (Core)?**

<details><summary>💡 Đáp án</summary>

| | .NET Framework | .NET (Core/.NET 5+) |
|---|---|---|
| **Platform** | Windows only | Cross-platform ✅ |
| **Open-source** | ❌ | ✅ |
| **Performance** | Chậm hơn | Nhanh hơn |
| **Version cuối** | 4.8 (ngừng) | .NET 8 LTS (tương lai) |
</details>

**4. CLR là gì? Làm những gì?**

<details><summary>💡 Đáp án</summary>

**CLR (Common Language Runtime)** quản lý: JIT Compilation (CIL → machine code), Garbage Collection (thu hồi bộ nhớ), Type Safety, Exception Handling, Thread Management, Security.
</details>

**5. Value type vs Reference type?**

<details><summary>💡 Đáp án</summary>

| | Value Type | Reference Type |
|---|---|---|
| **Lưu ở** | Stack | Heap (reference trên Stack) |
| **Copy** | Copy giá trị | Copy tham chiếu (cùng object) |
| **VD** | `int, double, bool, struct, enum` | `class, string, array` |

```csharp
int a = 5; int b = a; b = 10;       // a vẫn = 5
int[] x = {1,2}; int[] y = x; y[0] = 99; // x[0] = 99!
```
</details>

**6. `float` vs `double` vs `decimal`?**

<details><summary>💡 Đáp án</summary>

`float` (4B, ~7 digits) → game/graphics. `double` (8B, ~15 digits) → khoa học. `decimal` (16B, ~28 digits) → **tiền tệ**. ⚠️ `0.1 + 0.2 != 0.3` với float/double!
</details>

**7. `var` vs `dynamic`?**

<details><summary>💡 Đáp án</summary>

`var` = compile-time (compiler suy luận, có IntelliSense). `dynamic` = runtime (không kiểm tra, crash nếu sai). Luôn ưu tiên `var`.
</details>

**8. `==` vs `.Equals()`?**

<details><summary>💡 Đáp án</summary>

Value types: giống nhau (so giá trị). Reference types: `==` so tham chiếu, `Equals()` có thể override so giá trị. `string` override cả hai → so giá trị.
</details>

**9. `const` vs `readonly`?**

<details><summary>💡 Đáp án</summary>

`const` = compile-time, tự static, chỉ primitive/string. `readonly` = runtime (gán trong constructor), mọi kiểu. VD: `const double PI = 3.14;` vs `readonly DateTime Start = DateTime.Now;`
</details>

**10. `out` vs `ref`?**

<details><summary>💡 Đáp án</summary>

`ref`: biến PHẢI khởi tạo trước, method có thể sửa. `out`: không cần khởi tạo, method BẮT BUỘC gán. VD: `int.TryParse("123", out int result)`
</details>

**11. Stack vs Heap?**

<details><summary>💡 Đáp án</summary>

Stack: value types, nhanh, LIFO, ~1MB/thread, tự dọn. Heap: objects, chậm hơn, GC quản lý, chia sẻ giữa threads, lớn (GB).
</details>

**12. `for` vs `foreach`?**

<details><summary>💡 Đáp án</summary>

`for`: cần index, sửa collection, bước nhảy tùy ý. `foreach`: chỉ đọc, sạch hơn, an toàn hơn. ⚠️ Không sửa collection trong foreach!
</details>

**13. Method overloading vs overriding?**

<details><summary>💡 Đáp án</summary>

Overloading: cùng class, cùng tên, KHÁC params (compile-time). Overriding: derived class, `virtual` + `override`, cùng signature (runtime/polymorphism).
</details>

**14. `Array` vs `List<T>`?**

<details><summary>💡 Đáp án</summary>

Array: kích thước cố định, nhanh hơn. List: tự mở rộng, `.Add()/.Remove()`, dùng hầu hết trường hợp.
</details>

**15. Short-circuit evaluation?**

<details><summary>💡 Đáp án</summary>

`&&`: vế trái FALSE → bỏ qua vế phải. `||`: vế trái TRUE → bỏ qua vế phải. Quan trọng: `if (list != null && list.Count > 0)` → tránh NullReferenceException!
</details>


| # | Câu hỏi | Chủ đề |
|---|---------|--------|
| 1 | .NET Framework vs .NET Core/.NET? | Platform |
| 2 | What is CLR? What does it do? | Runtime |
| 3 | Value type vs Reference type? | Types |
| 4 | `float` vs `double` vs `decimal`? | Numeric |
| 5 | `var` vs `dynamic`? | Typing |
| 6 | `==` vs `.Equals()`? | Comparison |
| 7 | Implicit vs Explicit conversion? | Casting |
| 8 | `const` vs `readonly`? | Constants |
| 9 | Method overloading vs overriding? | Methods |
| 10 | `Array` vs `List<T>`? | Collections |
| 11 | Stack vs Heap? | Memory |
| 12 | `for` vs `foreach`? | Loops |
| 13 | `break` vs `continue` vs `return`? | Flow |
| 14 | `out` vs `ref` parameter? | Parameters |
| 15 | What is short-circuit evaluation? | Logic |

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p01-fundamentals](./99-answer-key-csharp.md#p01-fundamentals)
- Bài tập thực hành: [99-answer-key-csharp.md#p01-fundamentals-exercises](./99-answer-key-csharp.md#p01-fundamentals-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p01-fundamentals-deep](./97-csharp-theory-deep-dive.md#p01-fundamentals-deep)

