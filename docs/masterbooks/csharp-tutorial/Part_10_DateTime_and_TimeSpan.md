# Phần 10: Xử lý Ngày và Thời gian trong C#

Trong bất kỳ ứng dụng nào, từ hệ thống ngân hàng đến mạng xã hội, việc quản lý và thao tác với ngày tháng, thời gian là một yêu cầu không thể thiếu. Chúng ta cần ghi lại thời điểm một giao dịch xảy ra, tính toán tuổi, xác định thời hạn, hoặc hiển thị thông tin thời gian theo các định dạng phù hợp với người dùng toàn cầu. Phần này sẽ cung cấp một cái nhìn toàn diện về cách C# và .NET Framework xử lý dữ liệu thời gian, tập trung vào hai kiểu dữ liệu cốt lõi: `DateTime` và `TimeSpan`. Chúng ta sẽ khám phá cơ chế hoạt động, cách khởi tạo, thao tác và định dạng chúng, đồng thời làm rõ các nguyên tắc nền tảng như kiểu giá trị (Value Type) và tính bất biến (Immutability), cùng với các khía cạnh quan trọng của lập trình thời gian thực.

## 1. Nền tảng Quan trọng: Kiểu dữ liệu và Cơ chế bộ nhớ

Trước khi đi sâu vào cú pháp và cách sử dụng `DateTime` và `TimeSpan`, việc nắm vững cách C# quản lý bộ nhớ cho các kiểu dữ liệu là cực kỳ quan trọng. Điều này không chỉ giúp bạn hiểu rõ cách `DateTime` và `TimeSpan` hoạt động, mà còn là nền tảng cho việc viết mã nguồn hiệu quả và không lỗi.

### 1.1. Kiểu Giá trị (`struct`) và Kiểu Tham chiếu (`class`)

Trong C#, các kiểu dữ liệu được chia thành hai loại chính, quyết định cách chúng được cấp phát bộ nhớ và cách chúng được truyền đi trong chương trình:

*   **Kiểu tham chiếu (Reference Types)**:
    *   Được cấp phát trên **Heap**. Heap là một vùng bộ nhớ linh hoạt, lớn hơn, nơi các đối tượng có thể tồn tại lâu hơn phạm vi của phương thức tạo ra chúng.
    *   Biến kiểu tham chiếu không lưu trữ trực tiếp dữ liệu mà lưu trữ một **địa chỉ (tham chiếu)** đến vị trí của dữ liệu trên Heap.
    *   Ví dụ: `class`, `string`, `object`, `array`.
    *   Khi bạn gán một biến kiểu tham chiếu cho một biến khác, cả hai biến sẽ trỏ đến cùng một vị trí dữ liệu trên Heap. Mọi thay đổi thông qua một biến sẽ ảnh hưởng đến biến kia.

*   **Kiểu giá trị (Value Types)**:
    *   Thông thường được cấp phát trên **Stack**. Stack là một vùng bộ nhớ nhỏ hơn, có cấu trúc LIFO (Last-In, First-Out), được sử dụng cho các biến cục bộ và tham số phương thức. Dữ liệu trên Stack tự động bị hủy khi phạm vi của nó kết thúc.
    *   Nếu một kiểu giá trị là một trường (field) bên trong một đối tượng kiểu tham chiếu, nó sẽ được cấp phát cùng với đối tượng đó trên Heap.
    *   Biến kiểu giá trị lưu trữ trực tiếp **giá trị** của dữ liệu.
    *   Ví dụ: `int`, `bool`, `char`, `double`, và các **`struct`** (bao gồm `DateTime`, `TimeSpan`).
    *   Khi bạn gán một biến kiểu giá trị cho một biến khác, một **bản sao hoàn chỉnh** của giá trị sẽ được tạo ra. Hai biến sau đó hoàn toàn độc lập với nhau.

**`DateTime` và `TimeSpan` đều là các `struct`**, nghĩa là chúng là **kiểu giá trị**. Điều này có ý nghĩa sâu sắc:

```csharp
using System;

public class ValueTypeMemoryDemo
{
    public static void Main()
    {
        // Ví dụ với kiểu giá trị (int)
        int a = 10;
        int b = a; // Một bản sao của giá trị 10 được tạo và gán cho b
        Console.WriteLine($"Ban đầu: a = {a}, b = {b}"); // Output: a = 10, b = 10

        a = 20; // Thay đổi giá trị của a
        Console.WriteLine($"Sau khi thay đổi a: a = {a}, b = {b}"); // Output: a = 20, b = 10 (b không bị ảnh hưởng)

        Console.WriteLine("\n--- Với DateTime (cũng là kiểu giá trị) ---");

        DateTime date1 = new DateTime(2023, 1, 1);
        DateTime date2 = date1; // Một bản sao của đối tượng DateTime được tạo và gán cho date2
        Console.WriteLine($"Ban đầu: date1 = {date1}, date2 = {date2}");

        // Thao tác với date1 (sẽ tạo một đối tượng DateTime MỚI do tính bất biến)
        date1 = date1.AddDays(1); // date1 bây giờ trỏ đến một đối tượng mới (2023-01-02)

        Console.WriteLine($"Sau khi thay đổi date1: date1 = {date1}, date2 = {date2}");
        // Output: date1 = 2023-01-02 ..., date2 = 2023-01-01 ...
        // date2 vẫn giữ giá trị gốc vì nó là bản sao độc lập.
    }
}
```

Hiểu được cơ chế sao chép giá trị này là chìa khóa để tránh những lỗi tiềm ẩn khi thao tác với `DateTime` và `TimeSpan`, đặc biệt khi chúng được truyền làm đối số cho các phương thức.

### 1.2. Tính bất biến (Immutability): Thiết kế an toàn và hiệu quả

`DateTime` và `TimeSpan` không chỉ là kiểu giá trị mà còn là các kiểu **bất biến (immutable)**. Tính bất biến có nghĩa là một khi một đối tượng đã được tạo, **giá trị của nó không thể bị thay đổi**. Bất kỳ thao tác nào có vẻ như "sửa đổi" đối tượng (ví dụ: thêm ngày, giờ) thực chất sẽ **trả về một đối tượng mới** với giá trị đã thay đổi, trong khi đối tượng gốc vẫn giữ nguyên giá trị ban đầu của nó trong bộ nhớ.

```csharp
using System;

public class ImmutabilityDemo
{
    public static void Main()
    {
        DateTime originalDate = new DateTime(2023, 10, 27, 10, 0, 0);
        Console.WriteLine($"Ngày gốc (originalDate): {originalDate}");

        // Thêm 1 ngày
        DateTime newDate = originalDate.AddDays(1); // AddDays trả về một đối tượng DateTime MỚI
        Console.WriteLine($"Ngày mới (newDate): {newDate}");
        Console.WriteLine($"Ngày gốc (originalDate) sau AddDays: {originalDate}"); // originalDate KHÔNG THAY ĐỔI

        // Ví dụ về việc "gán lại" biến, chứ không phải sửa đổi đối tượng gốc
        DateTime myDate = DateTime.Now;
        Console.WriteLine($"\nMyDate ban đầu: {myDate}");
        myDate = myDate.AddHours(2); // Tạo đối tượng DateTime MỚI và gán lại cho myDate
        Console.WriteLine($"MyDate sau khi thêm 2 giờ: {myDate}");
        // original object for myDate is now eligible for garbage collection
    }
}
```

**Lợi ích của tính bất biến:**

*   **An toàn luồng (Thread-safety)**: Vì giá trị không bao giờ thay đổi, nhiều luồng có thể truy cập cùng một đối tượng `DateTime` hoặc `TimeSpan` mà không cần cơ chế khóa, loại bỏ nguy cơ tranh chấp dữ liệu.
*   **Tính nhất quán**: Đảm bảo rằng giá trị của một đối tượng sẽ không bị thay đổi một cách bất ngờ ở các phần khác của chương trình, giúp việc dự đoán hành vi mã nguồn dễ dàng hơn.
*   **Dễ dàng gỡ lỗi**: Việc theo dõi trạng thái chương trình trở nên đơn giản hơn vì bạn không cần lo lắng về việc giá trị bị thay đổi tại các điểm khác nhau.
*   **Hiệu suất (trong một số trường hợp)**: Đối với các đối tượng nhỏ như `DateTime` và `TimeSpan`, chi phí tạo một đối tượng mới thường là không đáng kể và được bù đ đắp bởi các lợi ích khác.

## 2. Kiểu `DateTime`: Đại diện cho một Khoảnh khắc Cụ thể

Kiểu `DateTime` (thuộc namespace `System`) được sử dụng để đại diện cho một điểm thời gian cụ thể, bao gồm ngày và giờ, từ năm 0001 đến năm 9999.

### 2.1. Khởi tạo đối tượng `DateTime`

Có nhiều cách để tạo một đối tượng `DateTime`, tùy thuộc vào nhu cầu của bạn.

#### 2.1.1. Sử dụng Constructor (`new DateTime(...)`)

Bạn có thể sử dụng toán tử `new` với các constructor khác nhau của `DateTime` để chỉ định các thành phần thời gian:

```csharp
using System;

public class DateTimeConstructorExamples
{
    public static void Main()
    {
        // Khởi tạo chỉ với năm, tháng, ngày (thời gian mặc định là 00:00:00)
        DateTime date1 = new DateTime(2015, 1, 1); // 1 tháng 1 năm 2015, 00:00:00
        Console.WriteLine($"Ngày khởi tạo: {date1}");

        // Khởi tạo với năm, tháng, ngày, giờ, phút, giây
        DateTime date2 = new DateTime(2023, 10, 27, 14, 30, 0); // 27 tháng 10 năm 2023, 14:30:00
        Console.WriteLine($"Ngày giờ cụ thể: {date2}");

        // Khởi tạo với năm, tháng, ngày, giờ, phút, giây, mili giây
        DateTime date3 = new DateTime(2023, 10, 27, 14, 30, 0, 500); // 27 tháng 10 năm 2023, 14:30:00.500
        Console.WriteLine($"Ngày giờ với mili giây: {date3}");

        // Constructor đầy đủ nhất: năm, tháng, ngày, giờ, phút, giây, mili giây, và DateTimeKind
        // DateTimeKind sẽ được giải thích ở phần nâng cao.
        DateTime date4 = new DateTime(2023, 10, 27, 14, 30, 0, 500, DateTimeKind.Local);
        Console.WriteLine($"Ngày giờ với Kind: {date4}");
    }
}
```

#### 2.1.2. Sử dụng các thuộc tính tĩnh (`static properties`)

`DateTime` cung cấp các thuộc tính tĩnh tiện lợi để lấy thời gian hiện tại của hệ thống:

*   **`DateTime.Now`**: Trả về một đối tượng `DateTime` đại diện cho ngày và giờ hiện tại của hệ thống **theo múi giờ địa phương (local time zone)**.
*   **`DateTime.Today`**: Trả về một đối tượng `DateTime` đại diện cho ngày hiện tại của hệ thống (theo múi giờ địa phương), với thời gian được đặt về 00:00:00.
*   **`DateTime.UtcNow`**: Trả về một đối tượng `DateTime` đại diện cho ngày và giờ hiện tại theo **Giờ Phối hợp Quốc tế (Coordinated Universal Time - UTC)**. Đây là lựa chọn tốt nhất cho các ứng dụng cần đồng bộ hóa thời gian trên toàn cầu hoặc lưu trữ dữ liệu thời gian trong cơ sở dữ liệu.

```csharp
using System;

public class DateTimeStaticProperties
{
    public static void Main()
    {
        DateTime currentLocalTime = DateTime.Now;
        Console.WriteLine($"Thời gian hiện tại (DateTime.Now - Local): {currentLocalTime}");

        DateTime currentDateOnly = DateTime.Today;
        Console.WriteLine($"Chỉ ngày hiện tại (DateTime.Today - Local): {currentDateOnly}");

        DateTime currentUtcTime = DateTime.UtcNow;
        Console.WriteLine($"Thời gian hiện tại (DateTime.UtcNow - UTC): {currentUtcTime}");
    }
}
```

> [!TIP]
> **Khi nào sử dụng `DateTime.Now` và `DateTime.UtcNow`?**
> *   Sử dụng `DateTime.Now` khi bạn cần hiển thị thời gian cho người dùng cuối trong múi giờ của họ, hoặc khi bạn đang làm việc với các hệ thống chỉ hoạt động cục bộ.
> *   **Luôn ưu tiên sử dụng `DateTime.UtcNow`** khi lưu trữ thời gian vào cơ sở dữ liệu, truyền thời gian giữa các hệ thống, hoặc thực hiện các phép tính logic liên quan đến thời gian mà không bị ảnh hưởng bởi múi giờ của máy chủ. Sau đó, chuyển đổi sang múi giờ địa phương khi hiển thị cho người dùng.

### 2.2. Truy cập các thành phần của `DateTime`

Sau khi có một đối tượng `DateTime`, bạn có thể dễ dàng truy cập các thành phần riêng lẻ của nó thông qua các thuộc tính:

```csharp
using System;

public class DateTimeComponents
{
    public static void Main()
    {
        DateTime now = DateTime.Now;

        Console.WriteLine($"Năm: {now.Year}");
        Console.WriteLine($"Tháng: {now.Month}");
        Console.WriteLine($"Ngày: {now.Day}");
        Console.WriteLine($"Giờ (24h): {now.Hour}");
        Console.WriteLine($"Phút: {now.Minute}");
        Console.WriteLine($"Giây: {now.Second}");
        Console.WriteLine($"Mili giây: {now.Millisecond}");
        Console.WriteLine($"Ngày trong tuần: {now.DayOfWeek}"); // Enum DayOfWeek (Sunday, Monday, ...)
        Console.WriteLine($"Ngày trong năm: {now.DayOfYear}");
        Console.WriteLine($"Loại thời gian (Kind): {now.Kind}"); // Local, Utc, Unspecified
        Console.WriteLine($"Số Ticks: {now.Ticks}"); // Số chu kỳ 100 nano giây kể từ 0001-01-01 00:00:00.000
    }
}
```

### 2.3. Thao tác với `DateTime` (Các phương thức `Add`)

Như đã thảo luận ở phần 1.2, `DateTime` là kiểu bất biến. Điều này có nghĩa là bạn không thể thay đổi một đối tượng `DateTime` hiện có. Thay vào đó, các phương thức thao tác (như thêm hoặc bớt thời gian) sẽ luôn trả về một đối tượng `DateTime` *mới* với giá trị đã được điều chỉnh.

Các phương thức phổ biến để thêm hoặc bớt thời gian:

*   `AddDays(double value)`
*   `AddHours(double value)`
*   `AddMinutes(double value)`
*   `AddSeconds(double value)`
*   `AddMilliseconds(double value)`
*   `AddMonths(int value)`
*   `AddYears(int value)`
*   `AddTicks(long value)`

Bạn có thể sử dụng giá trị dương để tiến lên trong thời gian và giá trị âm để lùi lại.

```csharp
using System;

public class DateTimeManipulation
{
    public static void Main()
    {
        DateTime originalDate = new DateTime(2023, 10, 27, 10, 0, 0);
        Console.WriteLine($"Ngày gốc: {originalDate}");

        // Thêm 1 ngày
        DateTime tomorrow = originalDate.AddDays(1);
        Console.WriteLine($"Ngày mai: {tomorrow}"); // originalDate không thay đổi

        // Bớt 2 giờ
        DateTime twoHoursAgo = originalDate.AddHours(-2);
        Console.WriteLine($"Hai giờ trước: {twoHoursAgo}");

        // Thêm 3 tháng
        DateTime threeMonthsLater = originalDate.AddMonths(3);
        Console.WriteLine($"Ba tháng sau: {threeMonthsLater}");

        // Thêm 1 năm và 5 ngày
        DateTime futureDate = originalDate.AddYears(1).AddDays(5);
        Console.WriteLine($"Một năm và 5 ngày sau: {futureDate}");
    }
}
```

### 2.4. Định dạng `DateTime` thành chuỗi và Chuyển đổi từ chuỗi

Việc hiển thị `DateTime` cho người dùng thường yêu cầu định dạng cụ thể. `DateTime` cung cấp phương thức `ToString()` với nhiều overload khác nhau để thực hiện việc này, cũng như các phương thức để phân tích chuỗi thành `DateTime`.

#### 2.4.1. Định dạng chuẩn (Standard Format Specifiers)

`DateTime` có một số phương thức tiện lợi để định dạng nhanh theo văn hóa hiện tại của hệ thống:

*   **`ToShortDateString()`**: Chỉ hiển thị phần ngày (ví dụ: `27/10/2023` hoặc `10/27/2023`).
*   **`ToLongDateString()`**: Hiển thị phần ngày đầy đủ (ví dụ: `Thứ Sáu, 27 Tháng Mười, 2023` hoặc `Friday, October 27, 2023`).
*   **`ToShortTimeString()`**: Chỉ hiển thị phần thời gian ngắn (ví dụ: `2:30 CH` hoặc `2:30 PM`).
*   **`ToLongTimeString()`**: Hiển thị phần thời gian đầy đủ (ví dụ: `2:30:00 CH` hoặc `2:30:00 PM`).
*   **`ToString()`**: Không có đối số sẽ sử dụng định dạng mặc định của văn hóa hiện tại.
*   **`ToUniversalTime()`**: Chuyển đổi `DateTime` hiện tại sang UTC.
*   **`ToLocalTime()`**: Chuyển đổi `DateTime` hiện tại sang múi giờ địa phương.

```csharp
using System;
using System.Globalization; // Để sử dụng CultureInfo

public class DateTimeFormatting
{
    public static void Main()
    {
        DateTime now = DateTime.Now;
        Console.WriteLine($"Thời gian hiện tại: {now}");

        Console.WriteLine($"\n--- Định dạng chuẩn ---");
        Console.WriteLine($"Ngày ngắn: {now.ToShortDateString()}");
        Console.WriteLine($"Ngày dài: {now.ToLongDateString()}");
        Console.WriteLine($"Thời gian ngắn: {now.ToShortTimeString()}");
        Console.WriteLine($"Thời gian dài: {now.ToLongTimeString()}");

        // Ví dụ với chuyển đổi múi giờ
        DateTime utcNow = DateTime.UtcNow;
        Console.WriteLine($"\nUTC Time: {utcNow}");
        Console.WriteLine($"UTC sang Local: {utcNow.ToLocalTime()}");

        DateTime localNow = DateTime.Now;
        Console.WriteLine($"Local Time: {localNow}");
        Console.WriteLine($"Local sang UTC: {localNow.ToUniversalTime()}");
    }
}
```

#### 2.4.2. Định dạng tùy chỉnh (Custom Format Specifiers)

Để có sự kiểm soát hoàn toàn về định dạng, bạn có thể truyền một chuỗi định dạng tùy chỉnh vào phương thức `ToString()`. Các ký tự định dạng này có thể kết hợp với nhau và với các ký tự thông thường.

| Ký tự | Mô tả                                       | Ví dụ (với `2023-10-27 14:35:08.500`) |
| :---- | :------------------------------------------ | :------------------------------------ |
| `d`   | Ngày (1-31)                                 | `27`                                  |
| `dd`  | Ngày (01-31)                                | `27`                                  |
| `M`   | Tháng (1-12)                                | `10`                                  |
| `MM`  | Tháng (01-12)                               | `10`                                  |
| `MMM` | Tên viết tắt của tháng (ví dụ: Oct)        | `Thg 10` (phụ thuộc CultureInfo)      |
| `MMMM`| Tên đầy đủ của tháng (ví dụ: October)      | `Tháng Mười` (phụ thuộc CultureInfo) |
| `y`   | Năm (1-2 chữ số)                            | `23`                                  |
| `yy`  | Năm (2 chữ số)                              | `23`                                  |
| `yyyy`| Năm (4 chữ số)                              | `2023`                                |
| `h`   | Giờ (1-12, định dạng 12 giờ)                | `2`                                   |
| `hh`  | Giờ (01-12, định dạng 12 giờ)               | `02`                                  |
| `H`   | Giờ (0-23, định dạng 24 giờ)                | `14`                                  |
| `HH`  | Giờ (00-23, định dạng 24 giờ)               | `14`                                  |
| `m`   | Phút (0-59)                                 | `35`                                  |
| `mm`  | Phút (00-59)                                | `35`                                  |
| `s`   | Giây (0-59)                                 | `8`                                   |
| `ss`  | Giây (00-59)                                | `08`                                  |
| `f`   | Giây thập phân (mili giây, 1 chữ số)        | `5` (từ .500)                         |
| `ff`  | Giây thập phân (mili giây, 2 chữ số)        | `50` (từ .500)                        |
| `fff` | Giây thập phân (mili giây, 3 chữ số)        | `500` (từ .500)                       |
| `t`   | Ký tự đầu tiên của AM/PM (ví dụ: A hoặc P) | `C` (chiều)                           |
| `tt`  | AM/PM (ví dụ: AM hoặc PM)                   | `CH` (chiều)                          |
| `K`   | Thông tin múi giờ                          | `+07:00`                              |
| `zzz` | Offset múi giờ                             | `+07:00`                              |
| `O`   | Định dạng ISO 8601 (Round-trip)             | `2023-10-27T14:35:08.5000000+07:00`   |
| `s`   | Định dạng chuẩn để truyền trong XML/JSON   | `2023-10-27T14:35:08`                 |

```csharp
using System;
using System.Globalization;

public class CustomDateTimeFormatting
{
    public static void Main()
    {
        DateTime dt = new DateTime(2023, 10, 27, 14, 35, 8, 500, DateTimeKind.Local); // 2:35:08.500 PM

        Console.WriteLine($"\n--- Định dạng tùy chỉnh ---");
        Console.WriteLine($"yyyy-MM-dd HH:mm:ss.fff: {dt.ToString("yyyy-MM-dd HH:mm:ss.fff")}");
        Console.WriteLine($"dd/MM/yyyy: {dt.ToString("dd/MM/yyyy")}");
        Console.WriteLine($"MMMM dd, yyyy (hh:mm tt): {dt.ToString("MMMM dd, yyyy (hh:mm tt)")}");
        Console.WriteLine($"ISO 8601 (O): {dt.ToString("O")}"); // Định dạng chuẩn ISO 8601, bảo toàn Kind
        Console.WriteLine($"XML (s): {dt.ToString("s")}"); // Định dạng chuẩn để truyền trong XML/JSON
        Console.WriteLine($"Ngày và giờ với múi giờ: {dt.ToString("yyyy-MM-dd HH:mm:ss K")}");
        Console.WriteLine($"Ngày và giờ với offset: {dt.ToString("yyyy-MM-dd HH:mm:ss zzz")}");

        // Ví dụ định dạng theo CultureInfo cụ thể
        CultureInfo usCulture = new CultureInfo("en-US");
        CultureInfo vnCulture = new CultureInfo("vi-VN");

        Console.WriteLine($"\n--- Định dạng theo CultureInfo ---");
        Console.WriteLine($"US: {dt.ToString("D", usCulture)}"); // Long date format for US
        Console.WriteLine($"VN: {dt.ToString("D", vnCulture)}"); // Long date format for VN
        Console.WriteLine($"US Custom: {dt.ToString("MMMM dd, yyyy", usCulture)}");
        Console.WriteLine($"VN Custom: {dt.ToString("dd MMMM yyyy", vnCulture)}");
    }
}
```

> [!NOTE]
> Để xem danh sách đầy đủ các ký tự định dạng chuẩn và tùy chỉnh, bạn có thể tham khảo tài liệu của Microsoft về "Standard date and time format strings" và "Custom date and time format strings" trên MSDN. Việc sử dụng `CultureInfo` cho phép ứng dụng của bạn hiển thị ngày giờ phù hợp với ngôn ngữ và khu vực của người dùng.

#### 2.4.3. Chuyển đổi từ chuỗi sang `DateTime` (`Parse`/`TryParse`)

Để chuyển đổi một chuỗi biểu diễn ngày giờ thành một đối tượng `DateTime`, bạn có thể sử dụng các phương thức tĩnh:

*   **`DateTime.Parse(string s)`**: Chuyển đổi chuỗi thành `DateTime`. Sẽ ném ra `FormatException` nếu chuỗi không hợp lệ.
*   **`DateTime.TryParse(string s, out DateTime result)`**: Cố gắng chuyển đổi chuỗi thành `DateTime`. Trả về `true` nếu thành công, `false` nếu thất bại. An toàn hơn khi xử lý đầu vào không đáng tin cậy.
*   **`DateTime.ParseExact(string s, string format, IFormatProvider provider)`**: Phân tích chuỗi theo một định dạng *chính xác* đã cho.
*   **`DateTime.TryParseExact(string s, string format, IFormatProvider provider, DateTimeStyles style, out DateTime result)`**: Phiên bản an toàn của `ParseExact`.

```csharp
using System;
using System.Globalization;

public class DateTimeParsing
{
    public static void Main()
    {
        string dateString1 = "2023-10-27 14:30:00";
        DateTime parsedDate1 = DateTime.Parse(dateString1);
        Console.WriteLine($"Parsed '{dateString1}': {parsedDate1}");

        string dateString2 = "12/25/2023"; // Định dạng có thể khác nhau tùy theo CultureInfo
        // Sử dụng TryParse để an toàn hơn
        if (DateTime.TryParse(dateString2, out DateTime parsedDate2))
        {
            Console.WriteLine($"Parsed '{dateString2}': {parsedDate2}");
        }
        else
        {
            Console.WriteLine($"Không thể phân tích chuỗi '{dateString2}'.");
        }

        // ParseExact để đảm bảo định dạng
        string dateString3 = "27-10-2023 10:00 AM";
        string format = "dd-MM-yyyy hh:mm tt";
        if (DateTime.TryParseExact(dateString3, format, CultureInfo.InvariantCulture, DateTimeStyles.None, out DateTime parsedDate3))
        {
            Console.WriteLine($"Parsed Exact '{dateString3}': {parsedDate3}");
        }
        else
        {
            Console.WriteLine($"Không thể phân tích chuỗi '{dateString3}' với định dạng '{format}'.");
        }
    }
}
```

> [!TIP]
> **Tư duy Vibe Coding với Antigravity IDE (Cho `DateTime`):**
>
> Khi bạn đang làm việc trong Antigravity IDE, thay vì phải nhớ chính xác các ký tự định dạng tùy chỉnh (`yyyy-MM-dd HH:mm:ss.fff`) hay các tùy chọn `CultureInfo`, bạn có thể sử dụng "Vibe Coding" để biểu đạt ý định của mình.
>
> *   **Ví dụ:** Thay vì gõ `DateTime.Now.ToString("dd/MM/yyyy HH:mm")`, bạn có thể chỉ cần "yêu cầu" Antigravity: "Hiển thị ngày giờ hiện tại theo định dạng ngày-tháng-năm và giờ-phút 24h."
> *   **Antigravity phản hồi:** Hệ thống Antigravity, với khả năng hiểu ngữ cảnh và lập kế hoạch, có thể tự động tạo ra mã C# tương ứng, thậm chí đề xuất các tùy chọn `CultureInfo` nếu nó nhận thấy người dùng đang ở một khu vực cụ thể hoặc muốn định dạng cho một đối tượng người dùng khác.
> *   **Lợi ích:** Điều này giúp bạn tập trung vào *kết quả mong muốn* hơn là *cú pháp cụ thể*, đặc biệt khi đối mặt với sự phức tạp của múi giờ (`DateTimeKind`) hoặc định dạng đa ngôn ngữ. Hãy thử hỏi Antigravity: "Làm thế nào để lưu trữ thời gian hiện tại vào database một cách an toàn để sau này có thể hiển thị chính xác cho người dùng ở bất kỳ múi giờ nào?" Antigravity có thể gợi ý sử dụng `DateTime.UtcNow` và giải thích lý do, thay vì bạn phải tự tìm kiếm.

## 3. Kiểu `TimeSpan`: Đại diện cho một Khoảng thời gian

Trong khi `DateTime` đại diện cho một điểm thời gian cụ thể, `TimeSpan` (cũng thuộc namespace `System`) đại diện cho một khoảng thời gian hoặc một khoảng chênh lệch giữa hai thời điểm. Ví dụ: 1 giờ 30 phút, 5 ngày, hoặc 10 mili giây. Nó có thể biểu thị cả khoảng thời gian dương và âm.

### 3.1. Khởi tạo đối tượng `TimeSpan`

Có nhiều cách để tạo một đối tượng `TimeSpan`:

#### 3.1.1. Sử dụng Constructor (`new TimeSpan(...)`)

Bạn có thể sử dụng toán tử `new` với các constructor của `TimeSpan` để chỉ định số ngày, giờ, phút, giây, mili giây:

```csharp
using System;

public class TimeSpanConstructorExamples
{
    public static void Main()
    {
        // Khởi tạo với giờ, phút, giây
        TimeSpan duration1 = new TimeSpan(1, 2, 3); // 1 giờ, 2 phút, 3 giây
        Console.WriteLine($"Khoảng thời gian 1 (HH:mm:ss): {duration1}");

        // Khởi tạo với ngày, giờ, phút, giây
        TimeSpan duration2 = new TimeSpan(2, 10, 30, 0); // 2 ngày, 10 giờ, 30 phút, 0 giây
        Console.WriteLine($"Khoảng thời gian 2 (d.HH:mm:ss): {duration2}");

        // Khởi tạo với ngày, giờ, phút, giây, mili giây
        TimeSpan duration3 = new TimeSpan(0, 0, 0, 0, 500); // 500 mili giây
        Console.WriteLine($"Khoảng thời gian 3 (mili giây): {duration3}");

        // Khởi tạo chỉ với số ticks (1 tick = 100 nano giây)
        TimeSpan duration4 = new TimeSpan(123456789); // Khoảng 12.3 giây
        Console.WriteLine($"Khoảng thời gian 4 (từ ticks): {duration4}");
    }
}
```

#### 3.1.2. Sử dụng các phương thức tĩnh (`static methods`)

Để dễ đọc và rõ ràng hơn, `TimeSpan` cung cấp các phương thức tĩnh để tạo một khoảng thời gian từ một đơn vị cụ thể:

*   `TimeSpan.FromDays(double value)`
*   `TimeSpan.FromHours(double value)`
*   `TimeSpan.FromMinutes(double value)`
*   `TimeSpan.FromSeconds(double value)`
*   `TimeSpan.FromMilliseconds(double value)`
*   `TimeSpan.FromTicks(long value)`

```csharp
using System;

public class TimeSpanStaticMethods
{
    public static void Main()
    {
        TimeSpan oneHour = TimeSpan.FromHours(1);
        Console.WriteLine($"Một giờ: {oneHour}");

        TimeSpan fiveMinutes = TimeSpan.FromMinutes(5);
        Console.WriteLine($"Năm phút: {fiveMinutes}");

        TimeSpan tenSeconds = TimeSpan.FromSeconds(10);
        Console.WriteLine($"Mười giây: {tenSeconds}");

        TimeSpan halfDay = TimeSpan.FromDays(0.5); // Nửa ngày
        Console.WriteLine($"Nửa ngày: {halfDay}");

        // Kết hợp các phương thức tĩnh cho tính toán phức tạp hơn
        TimeSpan totalTime = TimeSpan.FromHours(1) + TimeSpan.FromMinutes(30) + TimeSpan.FromSeconds(15);
        Console.WriteLine($"Tổng thời gian: {totalTime}");
    }
}
```

#### 3.1.3. Trừ hai đối tượng `DateTime`

Khi bạn trừ một đối tượng `DateTime` này cho một đối tượng `DateTime` khác, kết quả sẽ là một đối tượng `TimeSpan` biểu thị khoảng thời gian giữa hai thời điểm đó.

```csharp
using System;

public class TimeSpanFromDateTimeSubtraction
{
    public static void Main()
    {
        DateTime startDate = new DateTime(2023, 10, 27, 9, 0, 0);
        DateTime endDate = new DateTime(2023, 10, 27, 11, 30, 0);

        TimeSpan difference = endDate - startDate; // Phép trừ giữa hai DateTime trả về TimeSpan
        Console.WriteLine($"Thời gian làm việc: {difference}"); // Output: 02:30:00 (2 giờ 30 phút)

        DateTime now = DateTime.Now;
        DateTime future = now.AddDays(5).AddHours(3);
        TimeSpan timeUntilFuture = future - now;
        Console.WriteLine($"Thời gian đến tương lai: {timeUntilFuture}");

        // Khoảng thời gian âm
        TimeSpan pastDifference = startDate - endDate;
        Console.WriteLine($"Khoảng thời gian âm: {pastDifference}"); // Output: -02:30:00
    }
}
```

### 3.2. Truy cập các thành phần của `TimeSpan`

`TimeSpan` có hai loại thuộc tính để truy cập các thành phần:

*   **Các thuộc tính thành phần đơn lẻ (`Days`, `Hours`, `Minutes`, `Seconds`, `Milliseconds`)**: Trả về giá trị của thành phần đó trong khoảng thời gian, không tính đến các đơn vị lớn hơn hoặc nhỏ hơn. Ví dụ, nếu `TimeSpan` là "1 ngày 2 giờ", thì thuộc tính `Hours` sẽ trả về `2`.
*   **Các thuộc tính tổng (`TotalDays`, `TotalHours`, `TotalMinutes`, `TotalSeconds`, `TotalMilliseconds`)**: Trả về tổng giá trị của khoảng thời gian được biểu diễn bằng đơn vị đó, bao gồm cả các đơn vị lớn hơn. Ví dụ, nếu `TimeSpan` là "1 ngày 2 giờ", thì `TotalHours` sẽ trả về `26.0` (24 giờ từ 1 ngày + 2 giờ). Các thuộc tính này trả về `double` để bao gồm cả phần thập phân.

```csharp
using System;

public class TimeSpanComponents
{
    public static void Main()
    {
        TimeSpan duration = new TimeSpan(1, 2, 30, 45, 500); // 1 ngày, 2 giờ, 30 phút, 45 giây, 500 mili giây

        Console.WriteLine($"Khoảng thời gian: {duration}");

        Console.WriteLine($"\n--- Các thành phần đơn lẻ ---");
        Console.WriteLine($"Ngày: {duration.Days}");       // Output: 1
        Console.WriteLine($"Giờ: {duration.Hours}");       // Output: 2
        Console.WriteLine($"Phút: {duration.Minutes}");    // Output: 30
        Console.WriteLine($"Giây: {duration.Seconds}");    // Output: 45
        Console.WriteLine($"Mili giây: {duration.Milliseconds}"); // Output: 500

        Console.WriteLine($"\n--- Các thành phần tổng ---");
        Console.WriteLine($"Tổng số ngày: {duration.TotalDays}");       // Output: 1.09071759259259
        Console.WriteLine($"Tổng số giờ: {duration.TotalHours}");       // Output: 26.1793055555556
        Console.WriteLine($"Tổng số phút: {duration.TotalMinutes}");    // Output: 1570.75833333333
        Console.WriteLine($"Tổng số giây: {duration.TotalSeconds}");    // Output: 94245.5
        Console.WriteLine($"Tổng số mili giây: {duration.TotalMilliseconds}"); // Output: 94245500
        Console.WriteLine($"Tổng số Ticks: {duration.Ticks}"); // Output: 942455000000
    }
}
```

### 3.3. Thao tác với `TimeSpan`

Tương tự `DateTime`, `TimeSpan` cũng là kiểu bất biến. Các phương thức thao tác sẽ trả về một đối tượng `TimeSpan` mới.

Các phương thức chính là `Add()` và `Subtract()`, chúng nhận một đối tượng `TimeSpan` khác làm đối số. Bạn cũng có thể sử dụng các toán tử `+` và `-` trực tiếp.

```csharp
using System;

public class TimeSpanManipulation
{
    public static void Main()
    {
        TimeSpan originalDuration = new TimeSpan(1, 2, 3); // 1 giờ, 2 phút, 3 giây
        Console.WriteLine($"Khoảng thời gian gốc: {originalDuration}");

        // Thêm 8 phút bằng phương thức Add
        TimeSpan eightMinutes = TimeSpan.FromMinutes(8);
        TimeSpan newDurationAdd = originalDuration.Add(eightMinutes);
        Console.WriteLine($"Sau khi thêm 8 phút (Add): {newDurationAdd}"); // 1 giờ, 10 phút, 3 giây

        // Bớt 2 phút bằng toán tử -
        TimeSpan twoMinutes = TimeSpan.FromMinutes(2);
        TimeSpan newDurationSubtract = originalDuration - twoMinutes; // Sử dụng toán tử trừ
        Console.WriteLine($"Sau khi bớt 2 phút (-): {newDurationSubtract}"); // 1 giờ, 0 phút, 3 giây

        // Nhân một TimeSpan với một số (ví dụ: gấp đôi thời gian)
        TimeSpan doubledDuration = originalDuration.Multiply(2);
        Console.WriteLine($"Thời gian gấp đôi: {doubledDuration}"); // 2 giờ, 4 phút, 6 giây

        // Chia một TimeSpan cho một số (ví dụ: một nửa thời gian)
        TimeSpan halfDuration = originalDuration.Divide(2);
        Console.WriteLine($"Thời gian một nửa: {halfDuration}"); // 0 giờ, 31 phút, 1.5 giây
    }
}
```

### 3.4. Chuyển đổi `TimeSpan` sang và từ chuỗi

#### 3.4.1. `ToString()`

Phương thức `ToString()` của `TimeSpan` sẽ chuyển đổi khoảng thời gian thành một chuỗi theo định dạng chuẩn (ví dụ: `d.hh:mm:ss.ff`). Khi bạn sử dụng `Console.WriteLine()` với một đối tượng `TimeSpan`, phương thức `ToString()` này sẽ được gọi ngầm định.

```csharp
using System;

public class TimeSpanToString
{
    public static void Main()
    {
        TimeSpan duration1 = new TimeSpan(1, 2, 30, 45); // 1 ngày, 2 giờ, 30 phút, 45 giây
        Console.WriteLine($"TimeSpan mặc định: {duration1}"); // Tự động gọi ToString()
        Console.WriteLine($"TimeSpan với ToString() rõ ràng: {duration1.ToString()}");

        TimeSpan duration2 = TimeSpan.FromHours(5.5); // 5 giờ 30 phút
        Console.WriteLine($"TimeSpan 5.5 giờ: {duration2}");

        TimeSpan negativeDuration = TimeSpan.FromMinutes(-90); // -1 giờ 30 phút
        Console.WriteLine($"TimeSpan âm: {negativeDuration}");

        // Định dạng tùy chỉnh cho TimeSpan (ít phổ biến hơn DateTime)
        // Cần sử dụng chuỗi định dạng như "g", "G" hoặc định dạng tùy chỉnh thủ công.
        // "g": General short format (e.g., 1:02:03)
        // "G": General long format (e.g., 1:02:03.4560000)
        Console.WriteLine($"TimeSpan format 'g': {duration1.ToString("g")}");
        Console.WriteLine($"TimeSpan format 'G': {duration1.ToString("G")}");
    }
}
```

#### 3.4.2. `TimeSpan.Parse()` và `TimeSpan.TryParse()`

Để chuyển đổi một chuỗi biểu diễn khoảng thời gian thành một đối tượng `TimeSpan`, bạn có thể sử dụng phương thức tĩnh `TimeSpan.Parse()`. Chuỗi đầu vào phải tuân theo một định dạng `TimeSpan` hợp lệ (ví dụ: `[d.]hh:mm:ss[.ff]`).

```csharp
using System;

public class TimeSpanParse
{
    public static void Main()
    {
        string timeString1 = "1:02:03"; // 1 giờ, 2 phút, 3 giây
        TimeSpan parsedTime1 = TimeSpan.Parse(timeString1);
        Console.WriteLine($"Parsed '{timeString1}': {parsedTime1}");

        string timeString2 = "2.10:30:00"; // 2 ngày, 10 giờ, 30 phút, 0 giây
        TimeSpan parsedTime2 = TimeSpan.Parse(timeString2);
        Console.WriteLine($"Parsed '{timeString2}': {parsedTime2}");

        string timeString3 = "-00:45:00"; // Khoảng thời gian âm
        TimeSpan parsedTime3 = TimeSpan.Parse(timeString3);
        Console.WriteLine($"Parsed '{timeString3}': {parsedTime3}");

        // Xử lý lỗi với TryParse
        string invalidTimeString = "invalid time string";
        if (TimeSpan.TryParse(invalidTimeString, out TimeSpan parsedTime4))
        {
            Console.WriteLine($"Parsed '{invalidTimeString}': {parsedTime4}");
        }
        else
        {
            Console.WriteLine($"Không thể phân tích chuỗi '{invalidTimeString}' thành TimeSpan.");
        }

        // TryParseExact cho định dạng cụ thể
        string customTimeString = "01h 30m 15s";
        string format = "hh\\h mm\\m ss\\s"; // Dùng \\ để escape ký tự 'h', 'm', 's'
        if (TimeSpan.TryParseExact(customTimeString, format, CultureInfo.InvariantCulture, out TimeSpan parsedTime5))
        {
            Console.WriteLine($"Parsed Exact '{customTimeString}': {parsedTime5}");
        }
        else
        {
            Console.WriteLine($"Không thể phân tích chuỗi '{customTimeString}' với định dạng '{format}'.");
        }
    }
}
```

> [!TIP]
> Luôn sử dụng `TimeSpan.TryParse()` hoặc `DateTime.TryParse()` (hoặc `TryParseExact()`) khi phân tích chuỗi đầu vào từ người dùng hoặc từ nguồn bên ngoài (file, database, API) để tránh ngoại lệ `FormatException` nếu chuỗi không hợp lệ. Điều này giúp ứng dụng của bạn mạnh mẽ và ổn định hơn.

## 4. Mối quan hệ và Tương tác giữa `DateTime` và `TimeSpan`

`DateTime` và `TimeSpan` thường được sử dụng cùng nhau để thực hiện các phép tính liên quan đến thời gian, tạo nên một hệ thống mạnh mẽ và linh hoạt.

### 4.1. Tính toán khoảng thời gian giữa hai `DateTime`

Như đã thấy, phép trừ hai đối tượng `DateTime` sẽ trả về một `TimeSpan`, biểu thị khoảng thời gian chênh lệch giữa hai thời điểm đó.

```csharp
using System;

public class DateTimeTimeSpanInteraction
{
    public static void Main()
    {
        DateTime eventStart = new DateTime(2023, 11, 1, 9, 0, 0); // 1/11/2023 9:00 AM
        DateTime eventEnd = new DateTime(2023, 11, 1, 17, 30, 0); // 1/11/2023 5:30 PM

        TimeSpan eventDuration = eventEnd - eventStart; // Kết quả là một TimeSpan
        Console.WriteLine($"Sự kiện kéo dài: {eventDuration}"); // Output: 08:30:00 (8 giờ 30 phút)

        // Tính tuổi
        DateTime birthDate = new DateTime(1990, 5, 15);
        TimeSpan ageDuration = DateTime.Now - birthDate;
        Console.WriteLine($"Tuổi (dưới dạng TimeSpan): {ageDuration.TotalDays / 365.25:F2} năm"); // Chia cho 365.25 để tính năm nhuận
    }
}
```

### 4.2. Thêm hoặc bớt `TimeSpan` vào `DateTime`

Bạn có thể cộng hoặc trừ một `TimeSpan` vào một `DateTime` để nhận được một `DateTime` mới. Đây là cách chính để tính toán một thời điểm trong tương lai hoặc quá khứ.

```csharp
using System;

public class DateTimeTimeSpanArithmetic
{
    public static void Main()
    {
        DateTime baseDate = new DateTime(2023, 1, 1, 12, 0, 0); // 1/1/2023 12:00 PM
        Console.WriteLine($"Ngày gốc: {baseDate}");

        TimeSpan delay = TimeSpan.FromDays(7) + TimeSpan.FromHours(3); // 7 ngày 3 giờ
        DateTime futureDate = baseDate + delay; // Cộng TimeSpan vào DateTime
        Console.WriteLine($"Ngày trong tương lai (7 ngày 3 giờ sau): {futureDate}");

        TimeSpan pastDelay = TimeSpan.FromHours(5).AddMinutes(30); // 5 giờ 30 phút
        DateTime pastDate = baseDate - pastDelay; // Trừ TimeSpan khỏi DateTime
        Console.WriteLine($"Ngày trong quá khứ (5 giờ 30 phút trước): {pastDate}");

        // Sử dụng các phương thức Add của DateTime (thực chất cũng nhận TimeSpan ngầm)
        DateTime anotherFutureDate = baseDate.Add(delay);
        Console.WriteLine($"Ngày trong tương lai (Add method): {anotherFutureDate}");
    }
}
```

## 5. Các Khía cạnh Nâng cao và Lưu ý Quan trọng

### 5.1. `DateTimeKind` và Vấn đề Múi giờ

`DateTimeKind` là một thuộc tính quan trọng của `DateTime` cho biết loại thời gian mà đối tượng `DateTime` biểu diễn:

*   **`DateTimeKind.Unspecified`**: Không rõ múi giờ nào. Đây là giá trị mặc định nếu bạn không chỉ định khi tạo `DateTime` bằng constructor không có `DateTimeKind`.
*   **`DateTimeKind.Local`**: Thời gian trong múi giờ địa phương của hệ thống. `DateTime.Now` trả về loại này.
*   **`DateTimeKind.Utc`**: Thời gian theo Giờ Phối hợp Quốc tế (UTC). `DateTime.UtcNow` trả về loại này.

Việc hiểu và sử dụng `DateTimeKind` đúng cách là cực kỳ quan trọng để tránh các lỗi liên quan đến múi giờ trong các ứng dụng phân tán hoặc toàn cầu.

```csharp
using System;

public class DateTimeKindDemo
{
    public static void Main()
    {
        DateTime localTime = DateTime.Now;
        DateTime utcTime = DateTime.UtcNow;
        DateTime unspecifiedTime = new DateTime(2023, 11, 1, 10, 0, 0); // Mặc định là Unspecified

        Console.WriteLine($"Local Time: {localTime} (Kind: {localTime.Kind})");
        Console.WriteLine($"UTC Time: {utcTime} (Kind: {utcTime.Kind})");
        Console.WriteLine($"Unspecified Time: {unspecifiedTime} (Kind: {unspecifiedTime.Kind})");

        // Chuyển đổi giữa các Kind
        Console.WriteLine($"\nLocal sang UTC: {localTime.ToUniversalTime()} (Kind: {localTime.ToUniversalTime().Kind})");
        Console.WriteLine($"UTC sang Local: {utcTime.ToLocalTime()} (Kind: {utcTime.ToLocalTime().Kind})");

        // Cẩn trọng khi chuyển đổi Unspecified: nó sẽ coi như là Local và chuyển đổi
        Console.WriteLine($"Unspecified sang UTC: {unspecifiedTime.ToUniversalTime()} (Kind: {unspecifiedTime.ToUniversalTime().Kind})");
        // Điều này có thể gây ra lỗi nếu unspecifiedTime thực chất là UTC
    }
}
```

> [!CAUTION]
> **Thực hành tốt nhất về Múi giờ:**
> *   **Luôn lưu trữ thời gian dưới dạng UTC** trong cơ sở dữ liệu hoặc khi truyền dữ liệu giữa các hệ thống. Điều này loại bỏ sự mơ hồ về múi giờ.
> *   **Chỉ chuyển đổi sang múi giờ địa phương** khi hiển thị cho người dùng cuối.
> *   Sử dụng `DateTimeOffset` cho các tình huống phức tạp hơn yêu cầu lưu trữ cả thời điểm và offset múi giờ cụ thể. (Đây là một kiểu dữ liệu nâng cao hơn `DateTime` và nằm ngoài phạm vi khóa học cơ bản này, nhưng đáng để biết).

> [!TIP]
> **Vibe Coding với Antigravity IDE (Vấn đề Múi giờ):**
>
> Vấn đề múi giờ là một trong những điểm phức tạp nhất khi xử lý ngày giờ. Đây là nơi tư duy "Vibe Coding" và sự hỗ trợ của Antigravity IDE thực sự tỏa sáng.
>
> *   **Yêu cầu Vibe Coding:** Thay vì phải nhớ `DateTimeKind.Utc` hay `ToUniversalTime()`, bạn có thể chỉ cần nói với Antigravity: "Tôi muốn lưu thời điểm hiện tại vào database. Đảm bảo nó không bị ảnh hưởng bởi múi giờ của server."
> *   **Antigravity phản hồi:** Antigravity, với vai trò là một Agentic AI, sẽ hiểu ý định "không bị ảnh hưởng bởi múi giờ" và đề xuất sử dụng `DateTime.UtcNow`. Nó có thể giải thích lý do, đưa ra ví dụ về cách chuyển đổi ngược lại khi hiển thị, và thậm chí tự động tạo một đoạn mã mẫu.
> *   **Antigravity cho việc gỡ lỗi:** Nếu bạn có một bug liên quan đến thời gian, bạn có thể hỏi Antigravity: "Dòng code này đang sử dụng `DateTime.Now`, nhưng người dùng ở múi giờ khác đang thấy sai giờ. Có vấn đề gì không?" Antigravity có thể phân tích mã của bạn, nhận diện `DateTime.Now` là nguồn gốc tiềm năng của vấn đề và đề xuất các giải pháp dựa trên `DateTime.UtcNow` hoặc `DateTimeOffset`.
> *   **Bài học cho học viên:** Hãy tập thói quen *diễn đạt vấn đề* hoặc *ý định* của bạn một cách rõ ràng cho Antigravity, thay vì chỉ tìm kiếm cú pháp. Điều này sẽ giúp bạn phát triển khả năng tư duy hệ thống và giải quyết vấn đề hiệu quả hơn.

### 5.2. So sánh `DateTime` và `TimeSpan`

Cả `DateTime` và `TimeSpan` đều hỗ trợ các toán tử so sánh (`==`, `!=`, `>`, `<`, `>=`, `<=`) và phương thức `CompareTo()` để xác định thứ tự của chúng.

```csharp
using System;

public class DateTimeTimeSpanComparison
{
    public static void Main()
    {
        // So sánh DateTime
        DateTime dateA = new DateTime(2023, 1, 1);
        DateTime dateB = new DateTime(2023, 1, 1, 12, 0, 0); // Cùng ngày, nhưng khác giờ
        DateTime dateC = new DateTime(2023, 1, 2);

        Console.WriteLine($"\nSo sánh DateTime:");
        Console.WriteLine($"dateA == dateB: {dateA == dateB}"); // False (khác giờ)
        Console.WriteLine($"dateA < dateB: {dateA < dateB}");   // True
        Console.WriteLine($"dateC > dateB: {dateC > dateB}");   // True

        // So sánh TimeSpan
        TimeSpan duration1 = TimeSpan.FromHours(2);
        TimeSpan duration2 = TimeSpan.FromMinutes(120); // Cũng là 2 giờ
        TimeSpan duration3 = TimeSpan.FromHours(1.5);

        Console.WriteLine($"\nSo sánh TimeSpan:");
        Console.WriteLine($"duration1 == duration2: {duration1 == duration2}"); // True
        Console.WriteLine($"duration1 > duration3: {duration1 > duration3}");   // True
        Console.WriteLine($"duration3 < duration2: {duration3 < duration2}");   // True

        // Sử dụng CompareTo
        int comparisonResult = dateA.CompareTo(dateC);
        if (comparisonResult < 0)
        {
            Console.WriteLine($"\n{dateA} diễn ra trước {dateC}");
        }
        else if (comparisonResult > 0)
        {
            Console.WriteLine($"{dateA} diễn ra sau {dateC}");
        }
        else
        {
            Console.WriteLine($"{dateA} và {dateC} là cùng một thời điểm");
        }
    }
}
```

## Tóm tắt Phần

*   **`DateTime`** và **`TimeSpan`** là hai kiểu dữ liệu cốt lõi trong C# để xử lý ngày và thời gian.
*   **`DateTime`** đại diện cho một điểm thời gian cụ thể (ví dụ: 27/10/2023 14:30:00).
    *   Có thể khởi tạo bằng `new DateTime(...)`, `DateTime.Now` (local), `DateTime.UtcNow` (UTC), `DateTime.Today`.
    *   Các thuộc tính như `Year`, `Month`, `Hour`, `DayOfWeek`, `Kind` cho phép truy cập các thành phần và thông tin múi giờ.
    *   Các phương thức `AddDays()`, `AddHours()`, v.v., trả về một đối tượng `DateTime` *mới* đã được điều chỉnh do tính bất biến.
    *   Có thể định dạng thành chuỗi bằng `ToString()` với các định dạng chuẩn hoặc tùy chỉnh, và phân tích từ chuỗi bằng `Parse()`/`TryParse()`.
*   **`TimeSpan`** đại diện cho một khoảng thời gian hoặc một khoảng chênh lệch giữa hai thời điểm (ví dụ: 2 giờ 30 phút).
    *   Có thể khởi tạo bằng `new TimeSpan(...)`, `TimeSpan.FromHours(...)`, hoặc từ phép trừ hai `DateTime`.
    *   Các thuộc tính `Days`, `Hours` cung cấp các thành phần riêng lẻ; `TotalDays`, `TotalHours` cung cấp tổng giá trị dưới dạng số thực.
    *   Các phương thức `Add()` và `Subtract()` (hoặc toán tử `+`, `-`) trả về một đối tượng `TimeSpan` *mới*.
    *   Có thể chuyển đổi sang chuỗi bằng `ToString()` và từ chuỗi bằng `TimeSpan.Parse()`/`TryParse()`.
*   Cả **`DateTime`** và **`TimeSpan`** đều là **kiểu giá trị (`struct`)** và **bất biến (immutable)**. Điều này có nghĩa là chúng được sao chép khi gán hoặc truyền, và các phương thức thao tác luôn trả về một đối tượng mới thay vì sửa đổi đối tượng hiện có, đảm bảo tính nhất quán và an toàn luồng.
*   Hai kiểu này thường được sử dụng cùng nhau để tính toán khoảng thời gian (`DateTime` - `DateTime` = `TimeSpan`) hoặc điều chỉnh thời điểm (`DateTime` + `TimeSpan` = `DateTime`).
*   Hiểu rõ **`DateTimeKind`** và luôn cân nhắc việc sử dụng **UTC** cho logic nghiệp vụ và lưu trữ là chìa khóa để xây dựng các ứng dụng mạnh mẽ, không lỗi múi giờ.
*   Hãy tận dụng khả năng của **Antigravity IDE** và tư duy **Vibe Coding** để diễn đạt ý định của bạn một cách rõ ràng, cho phép AI hỗ trợ bạn trong việc lựa chọn kiểu dữ liệu, định dạng và xử lý múi giờ một cách thông minh và hiệu quả.

<!-- REVIEWED_BY_AGENT -->
