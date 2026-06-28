# 📓 Phần 9: Projects, Assemblies & Strings

> **Nội dung**: Solutions, Assemblies, NuGet, string manipulation, StringBuilder  
> **Thời lượng ước tính**: 4–5 giờ

---

## 🎯 Mục Tiêu

- Hiểu rõ quan hệ giữa `solution`, `project`, `assembly` trong .NET.
- Tổ chức được mã nguồn nhiều project theo hướng dễ mở rộng và dễ test.
- Quản lý dependencies bằng NuGet và hiểu vai trò của `.csproj`.
- Làm chủ xử lý chuỗi: hiệu năng, định dạng, encoding, culture.
- Biết chọn `string` hay `StringBuilder` dựa trên bối cảnh thực tế.

---

## PART A: Projects, Assemblies, Solutions

### 1. Project & Solution Structure

```
MySolution.sln                 ← Solution file
├── MyApp/                     ← Console App project
│   ├── MyApp.csproj           ← Project file
│   ├── Program.cs
│   └── Models/
├── MyApp.Core/                ← Class Library project
│   ├── MyApp.Core.csproj
│   └── Services/
└── MyApp.Tests/               ← Test project
    ├── MyApp.Tests.csproj
    └── ServiceTests.cs
```

- **Solution (.sln)**: nhóm các projects liên quan
- **Project (.csproj)**: đơn vị build, mỗi project tạo ra 1 assembly
- **Assembly (.dll / .exe)**: file nhị phân chứa CIL + metadata

### 2. Debug vs Release Build

| Đặc điểm | Debug | Release |
|-----------|-------|---------|
| Optimization | ❌ Tắt | ✅ Bật |
| Debug symbols | ✅ Đầy đủ | ❌ Ít/Không |
| Tốc độ | Chậm hơn | **Nhanh hơn** |
| Kích thước | Lớn hơn | Nhỏ hơn |
| Dùng khi | **Phát triển** | **Triển khai** |

### 3. Access Modifiers Summary

```
┌──────────────────────────────────────────────────────┐
│ Modifier          │ Same │ Same │ Sub  │ Sub   │ Any │
│                   │Class │Asm.  │Class │Class  │     │
│                   │      │      │Same  │Diff   │     │
│                   │      │      │Asm.  │Asm.   │     │
├───────────────────┼──────┼──────┼──────┼───────┼─────┤
│ public            │  ✅  │  ✅  │  ✅  │  ✅   │ ✅  │
│ internal          │  ✅  │  ✅  │  ✅  │  ❌   │ ❌  │
│ protected internal│  ✅  │  ✅  │  ✅  │  ✅   │ ❌  │
│ protected         │  ✅  │  ❌  │  ✅  │  ✅   │ ❌  │
│ private protected │  ✅  │  ❌  │  ✅  │  ❌   │ ❌  │
│ private           │  ✅  │  ❌  │  ❌  │  ❌   │ ❌  │
└──────────────────────────────────────────────────────┘
```

### 4. NuGet

```bash
# NuGet: package manager cho .NET (giống npm/pip)
dotnet add package Newtonsoft.Json          # Thêm package
dotnet add package Serilog --version 3.0.0  # Version cụ thể
dotnet restore                               # Restore packages
dotnet list package                          # Liệt kê packages
```

### 5. .csproj File

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
    <ProjectReference Include="..\MyApp.Core\MyApp.Core.csproj" />
  </ItemGroup>
</Project>
```

---

## PART B: Strings

### 1. Character Encoding

```csharp
// Mỗi char trong C# = 2 bytes (UTF-16)
char a = 'A';           // Unicode: U+0041
char vn = 'Ă';          // Unicode: U+0102

// Encoding conversion
byte[] utf8Bytes = Encoding.UTF8.GetBytes("Xin chào");
string str = Encoding.UTF8.GetString(utf8Bytes);

byte[] win1252 = Encoding.GetEncoding(1252).GetBytes("Straße");
```

### 2. String Immutability (Bất biến)

```csharp
string s = "Hello";
s = s + " World";   // KHÔNG sửa "Hello" — tạo string MỚI
// "Hello" cũ vẫn còn trong bộ nhớ, chờ GC dọn

// ⚠️ Nối chuỗi trong vòng lặp = GẤP ĐÔI bộ nhớ mỗi lần!
string result = "";
for (int i = 0; i < 10000; i++)
    result += i.ToString();  // ❌ Tạo 10000 string objects!
```

### 3. StringBuilder

```csharp
// StringBuilder: chuỗi có thể sửa đổi (mutable)
var sb = new StringBuilder();

for (int i = 0; i < 10000; i++)
    sb.Append(i);  // ✅ Sửa buffer hiện có, KHÔNG tạo string mới

string result = sb.ToString();

// StringBuilder methods
sb.Clear();               // Xóa hết
sb.Append("Hello");       // Thêm cuối
sb.AppendLine(" World");  // Thêm + xuống dòng
sb.Insert(5, ",");        // Chèn tại vị trí
sb.Replace("Hello", "Hi"); // Thay thế
sb.Remove(0, 3);          // Xóa từ vị trí
```

### 4. String Interning

```csharp
// .NET tự động intern literal strings
string a = "Hello";
string b = "Hello";
Console.WriteLine(ReferenceEquals(a, b));  // true! (cùng 1 object)

// Runtime strings KHÔNG được intern tự động
string c = new string("Hello".ToCharArray());
Console.WriteLine(ReferenceEquals(a, c));  // false (khác object)

string d = string.Intern(c);
Console.WriteLine(ReferenceEquals(a, d));  // true (đã intern)
```

### 5. String Formatting

```csharp
// Format specifiers
decimal price = 1234.567m;
Console.WriteLine($"{price:C}");      // $1,234.57 (currency)
Console.WriteLine($"{price:F2}");     // 1234.57 (fixed point)
Console.WriteLine($"{price:N0}");     // 1,235 (number with commas)
Console.WriteLine($"{price:P1}");     // 123,456.7% (percent)

DateTime now = DateTime.Now;
Console.WriteLine($"{now:yyyy-MM-dd}");         // 2026-02-21
Console.WriteLine($"{now:dd/MM/yyyy HH:mm}");   // 21/02/2026 17:30

int num = 42;
Console.WriteLine($"{num:D5}");      // 00042 (padded)
Console.WriteLine($"{num:X}");       // 2A (hex)
Console.WriteLine($"{num,10}");      // "        42" (alignment)
Console.WriteLine($"{num,-10}|");    // "42        |" (left align)
```

### 6. Culture-specific Formatting

```csharp
using System.Globalization;

decimal price = 1234.56m;

// Specific culture
Console.WriteLine(price.ToString("C", CultureInfo.GetCultureInfo("vi-VN")));
// 1.234,56 ₫

Console.WriteLine(price.ToString("C", CultureInfo.GetCultureInfo("en-US")));
// $1,234.56

Console.WriteLine(price.ToString("C", CultureInfo.GetCultureInfo("de-DE")));
// 1.234,56 €

// Invariant culture (cho serialization/file output)
string saved = price.ToString(CultureInfo.InvariantCulture);  // "1234.56"
decimal loaded = decimal.Parse(saved, CultureInfo.InvariantCulture);
```

---

## ❓ Câu Hỏi Kiểm Tra

1. Solution vs Project vs Assembly — mối quan hệ?
2. Debug vs Release build — khác nhau thế nào?
3. `internal` access modifier — ai có thể truy cập?
4. NuGet là gì? Tương đương gì trong npm/pip?
5. String immutability nghĩa là gì? Tại sao lại thiết kế vậy?
6. `StringBuilder` giải quyết vấn đề gì?
7. String interning là gì? Khi nào nó xảy ra tự động?
8. `CultureInfo.InvariantCulture` dùng khi nào?
9. `$"{price:C}"` in ra gì? Phụ thuộc vào cái gì?
10. `protected internal` vs `private protected` — khác nhau?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: Why are strings immutable in C#?**
> A: Thread-safe (không cần lock), string interning (tiết kiệm memory), security (hash không đổi), và reliability (phương thức nhận string đảm bảo không bị sửa). Trade-off: nối chuỗi nhiều → dùng StringBuilder.

> **Q: What is the difference between `string` and `StringBuilder`?**
> A: `string` immutable — mỗi thay đổi tạo object mới. `StringBuilder` mutable — sửa buffer nội bộ. Dùng StringBuilder khi nối chuỗi trong loop (>5-10 lần). String cho concatenation đơn giản.

> **Q: What is `string.Intern()` and string interning?**
> A: .NET pool literal strings — 2 strings cùng value trỏ cùng 1 object. `string.Intern()` cho phép thêm runtime strings vào pool. Tiết kiệm memory khi có nhiều duplicate strings.

> **Q: Explain access modifiers in C#.**
> A: `public`: mọi nơi. `private`: chỉ class. `protected`: class + derived. `internal`: cùng assembly. `protected internal`: cùng assembly HOẶC derived. `private protected`: cùng assembly VÀ derived.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Tạo solution 3 projects: App (console), Core (library), Tests. App reference Core. Chạy tests.

**BT2**: So sánh performance: string concatenation vs StringBuilder cho 100,000 iterations. Đo bằng `Stopwatch`.

**BT3**: Viết method format tiền tệ cho 5 quốc gia khác nhau dùng `CultureInfo`.

**BT4**: Viết method `string ReverseWords(string sentence)` — đảo thứ tự từ. "Hello World" → "World Hello".

**BT5**: Viết method `bool IsAnagram(string a, string b)` — kiểm tra 2 chuỗi có phải anagram. "listen" và "silent" → true.

**BT6**: Implement `CaesarCipher(string text, int shift)` — mã hóa Caesar.

---

## ✅ Checklist Hoàn Thành Module

1. Tự tạo mới solution nhiều project mà không cần xem lại hướng dẫn.
2. Giải thích được vì sao project nào nên là `Class Library`, project nào nên là `Executable`.
3. Benchmark được `string` vs `StringBuilder` với dữ liệu lớn và rút ra kết luận có số liệu.
4. Format/parse được dữ liệu theo `CultureInfo` khác nhau mà không lỗi locale.
5. Hoàn thành ít nhất 3 bài tập thực hành, trong đó có 1 bài xử lý text dài.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p09-projects-strings](./99-answer-key-csharp.md#p09-projects-strings)
- Bài tập thực hành: [99-answer-key-csharp.md#p09-projects-strings-exercises](./99-answer-key-csharp.md#p09-projects-strings-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p09-projects-strings-deep](./97-csharp-theory-deep-dive.md#p09-projects-strings-deep)

