# 📕 Phần 10: Numeric Types & Events

> **Nội dung**: Binary, bitwise operators, delegates, events, Observer pattern  
> **Thời lượng ước tính**: 4–5 giờ

---

## 🎯 Mục Tiêu

- Nắm chắc biểu diễn số trong máy tính (binary, signed/unsigned, overflow).
- Chọn đúng kiểu số (`int`, `long`, `double`, `decimal`) theo ngữ cảnh.
- Sử dụng được bitwise operators cho các bài toán flags và low-level logic.
- Hiểu mô hình publisher/subscriber và viết event đúng chuẩn C#.
- Tránh memory leak do event subscriptions và biết cách cleanup an toàn.

---

## PART A: Numeric Types

### 1. Binary Number System

```
Decimal:  42
Binary:   101010

Conversion Decimal → Binary:
42 ÷ 2 = 21 remainder 0
21 ÷ 2 = 10 remainder 1
10 ÷ 2 = 5  remainder 0
5  ÷ 2 = 2  remainder 1
2  ÷ 2 = 1  remainder 0
1  ÷ 2 = 0  remainder 1
→ Read bottop up: 101010

Bits:  N bits → max value = 2^N - 1
8 bits  (byte)  → 0 to 255
16 bits (short) → 0 to 65,535
32 bits (int)   → 0 to 4,294,967,295 (unsigned)
```

### 2. Integer Types Summary

| Type | Size | Range | Suffix |
|------|------|-------|--------|
| `byte` | 1B | 0–255 | — |
| `sbyte` | 1B | -128–127 | — |
| `short` | 2B | ±32K | — |
| `ushort` | 2B | 0–65K | — |
| `int` | 4B | ±2.1B | — |
| `uint` | 4B | 0–4.3B | `u` / `U` |
| `long` | 8B | ±9.2×10¹⁸ | `L` |
| `ulong` | 8B | 0–1.8×10¹⁹ | `UL` |

### 3. Numeric Overflow

```csharp
// Silent overflow (mặc định)
int maxInt = int.MaxValue;  // 2,147,483,647
int overflow = maxInt + 1;   // -2,147,483,648 (wrap around!)

// checked: throw OverflowException
checked
{
    int result = int.MaxValue + 1;  // ❌ OverflowException!
}

// unchecked (mặc định): quay vòng im lặng
unchecked
{
    int result = int.MaxValue + 1;  // -2,147,483,648
}
```

### 4. Floating-Point Numbers

```csharp
// double & float: IEEE 754 binary floating-point
// ⚠️ KHÔNG chính xác với số thập phân!
double a = 0.1 + 0.2;
Console.WriteLine(a == 0.3);      // false!
Console.WriteLine(a);              // 0.30000000000000004

// So sánh đúng cách:
bool equal = Math.Abs(a - 0.3) < 1e-10;  // true

// decimal: chính xác cho tiền tệ
decimal price = 0.1m + 0.2m;
Console.WriteLine(price == 0.3m);  // true!

// Khi nào dùng gì?
// float/double: khoa học, đồ họa (nhanh, chấp nhận sai số nhỏ)
// decimal: tài chính, tiền tệ, kế toán (chính xác tuyệt đối)
```

### 5. Bitwise Operators (AND, OR, XOR, NOT, SHIFT)

```csharp
// Biểu diễn nhị phân của 10 và 6
// 10 = 1010
//  6 = 0110
int a = 10;
int b = 6;

int andResult = a & b;  // 0010 = 2
int orResult = a | b;   // 1110 = 14
int xorResult = a ^ b;  // 1100 = 12
int notA = ~a;          // đảo bit (two's complement)

int leftShift = a << 1;   // 10100 = 20
int rightShift = a >> 1;  // 0101 = 5

Console.WriteLine($"a & b = {andResult}");
Console.WriteLine($"a | b = {orResult}");
Console.WriteLine($"a ^ b = {xorResult}");
Console.WriteLine($"~a = {notA}");
Console.WriteLine($"a << 1 = {leftShift}, a >> 1 = {rightShift}");

// Ứng dụng flags enum
[Flags]
enum Permission
{
    None = 0,
    Read = 1 << 0,   // 0001
    Write = 1 << 1,  // 0010
    Execute = 1 << 2 // 0100
}

Permission p = Permission.Read | Permission.Write;
bool canWrite = (p & Permission.Write) != 0; // true
```

---

## PART B: Events

### 1. Observer Design Pattern

```
Subject (Publisher)  ←  Observer (Subscriber)
  Khi trạng thái       Được thông báo
  thay đổi...          và phản ứng

Ví dụ: YouTube Channel (Subject) → Subscribers (Observers)

- Channel đăng video mới → TẤT CẢ subscribers nhận thông báo
```

### 2. Events in C#

```csharp
// Publisher: class phát sinh event
class StockPrice
{
    private decimal _price;
    
    // Khai báo event
    public event EventHandler<PriceChangedEventArgs> PriceChanged;
    
    public decimal Price
    {
        get => _price;
        set
        {
            decimal oldPrice = _price;
            _price = value;
            
            // Raise event (phát sự kiện)
            PriceChanged?.Invoke(this, new PriceChangedEventArgs(oldPrice, value));
        }
    }
}

// Custom EventArgs
class PriceChangedEventArgs : EventArgs
{
    public decimal OldPrice { get; }
    public decimal NewPrice { get; }
    public decimal Change => NewPrice - OldPrice;
    
    public PriceChangedEventArgs(decimal old, decimal @new) =>
        (OldPrice, NewPrice) = (old, @new);
}

// Subscriber: class lắng nghe event
class PriceAlert
{
    public void OnPriceChanged(object sender, PriceChangedEventArgs e)
    {
        if (Math.Abs(e.Change) > 5)
            Console.WriteLine($"⚠️ Biến động lớn: {e.Change:+0.00;-0.00}");
    }
}

// Kết nối
var stock = new StockPrice();
var alert = new PriceAlert();

stock.PriceChanged += alert.OnPriceChanged;  // Subscribe
stock.Price = 100;  // → Trigger event
stock.Price = 110;  // → ⚠️ Biến động lớn: +10.00

stock.PriceChanged -= alert.OnPriceChanged;  // Unsubscribe
```

### 3. Event vs Delegate

```csharp
// ❌ delegate field: bên ngoài có thể gọi trực tiếp
public Action<string> OnMessage;
// Bất kỳ ai cũng gọi: obj.OnMessage("hack!");

// ✅ event: chỉ class chứa mới có thể raise
public event Action<string> OnMessage;
// Bên ngoài chỉ có thể += và -=
// obj.OnMessage("hack!");  // ❌ Lỗi compile!
```

### 4. Memory Leaks từ Events

```csharp
// ⚠️ Event giữ reference đến subscriber → subscriber không bị GC dọn!
class LongLivedPublisher
{
    public event EventHandler SomethingHappened;
}

class ShortLivedSubscriber
{
    public ShortLivedSubscriber(LongLivedPublisher publisher)
    {
        publisher.SomethingHappened += OnSomething;
        // ❌ Quên unsubscribe → ShortLivedSubscriber KHÔNG BAO GIỜ bị GC dọn!
    }
    
    private void OnSomething(object s, EventArgs e) { }
}

// ✅ Giải pháp: luôn unsubscribe khi không cần nữa
// publisher.SomethingHappened -= OnSomething;
```

### 5. Windows Forms Basics

```csharp
// Windows Forms: Framework để tạo desktop GUI
using System.Windows.Forms;

public class MainForm : Form
{
    private Button _button;
    private TextBox _textBox;
    private Label _label;
    
    public MainForm()
    {
        _button = new Button { Text = "Click me", Location = new(10, 10) };
        _textBox = new TextBox { Location = new(10, 50), Width = 200 };
        _label = new Label { Location = new(10, 80), AutoSize = true };
        
        _button.Click += (s, e) =>
        {
            _label.Text = $"Bạn nhập: {_textBox.Text}";
        };
        
        Controls.Add(_button);
        Controls.Add(_textBox);
        Controls.Add(_label);
    }
}

// Entry point
[STAThread]
static void Main()
{
    Application.EnableVisualStyles();
    Application.Run(new MainForm());
}
```

---

## ❓ Câu Hỏi Kiểm Tra

1. `0.1 + 0.2 != 0.3` trong C# — tại sao?
2. `float` vs `double` vs `decimal` — khi nào dùng cái nào?
3. `checked` và `unchecked` blocks làm gì?
4. Event là gì? Khác delegate thế nào?
5. Observer pattern giải quyết vấn đề gì?
6. `event` keyword ngăn chặn điều gì mà delegate không ngăn?
7. Tại sao event có thể gây memory leak?
8. `EventHandler<T>` — T phải kế thừa từ class nào?
9. `?.Invoke()` dùng để làm gì khi raise event?
10. Signed vs Unsigned integer — khi nào dùng unsigned?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between `float`, `double`, and `decimal`?**
> A: `float` (4B, ~7 digits), `double` (8B, ~15 digits): binary floating-point, nhanh, có sai số. `decimal` (16B, ~28 digits): base-10, chính xác, chậm 10x. Dùng decimal cho tiền tệ, double cho khoa học.

> **Q: What is the Observer pattern? How do events implement it?**
> A: Publisher (Subject) thông báo Subscribers (Observers) khi state thay đổi. C# event = built-in Observer. Publisher khai báo `event`, subscribers dùng `+=` để subscribe. Decoupling: publisher không biết subscriber là ai.

> **Q: How can events cause memory leaks?**
> A: Event giữ strong reference đến subscriber. Nếu publisher tồn tại lâu hơn subscriber → subscriber không bị GC dọn. Fix: luôn `-=` trong Dispose(), hoặc dùng `WeakEventPattern`, hoặc dùng `IDisposable`.

> **Q: What is numeric overflow and how to handle it?**
> A: Khi kết quả vượt range (int.MaxValue + 1 → wrap around thành negative). Mặc định C# im lặng (unchecked). Dùng `checked { }` block để throw OverflowException. Hoặc dùng type lớn hơn (long, decimal).

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Implement `TemperatureMonitor` với event `TemperatureChanged`. Raise khi nhiệt độ thay đổi >5°C. 2 subscribers: Logger và AlertSystem.

**BT2**: Viết chương trình convert giữa decimal, binary, hex. Input: "42" → Output: "Binary: 101010, Hex: 2A".

**BT3**: Tạo `EventBus` class — generic pub/sub system: `Subscribe<TEvent>(Action<TEvent>)` và `Publish<TEvent>(TEvent)`.

**BT4**: Implement `EventAggregator` — publisher/subscriber pattern cho loosely-coupled communication.

**BT5**: Dùng `enum` + bitwise để xây dựng hệ thống phân quyền (`Read`, `Write`, `Delete`, `Admin`) với các hàm `Grant`, `Revoke`, `HasPermission`.

---

## ✅ Checklist Hoàn Thành Module

1. Tự giải thích được vì sao overflow xảy ra và khi nào cần `checked`.
2. Viết và đọc thành thạo bitwise expressions trên flags enum.
3. Xây được event flow có subscribe/unsubscribe đúng vòng đời object.
4. Chứng minh được (bằng ví dụ) memory leak do event và cách fix.
5. Hoàn thành ít nhất 1 bài numeric + 1 bài event có test hoặc demo output rõ ràng.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p10-numerics-events](./99-answer-key-csharp.md#p10-numerics-events)
- Bài tập thực hành: [99-answer-key-csharp.md#p10-numerics-events-exercises](./99-answer-key-csharp.md#p10-numerics-events-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p10-numerics-events-deep](./97-csharp-theory-deep-dive.md#p10-numerics-events-deep)

