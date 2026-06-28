# Phần 3: Chuyển đổi kiểu dữ liệu và Toán tử

Phần này tập trung vào hai khái niệm cốt lõi trong lập trình C#: **Chuyển đổi kiểu dữ liệu** và **Toán tử**. Việc nắm vững cách C# xử lý các kiểu dữ liệu khác nhau và cách các toán tử thực hiện các phép tính là nền tảng để viết mã chính xác, hiệu quả và dễ bảo trì.

Chúng ta sẽ phân tích các phương pháp chuyển đổi kiểu dữ liệu, từ những chuyển đổi ngầm định an toàn đến những chuyển đổi rõ ràng đòi hỏi sự thận trọng, và cách thức xử lý các kiểu dữ liệu không tương thích. Tiếp theo, chúng ta sẽ khảo sát các loại toán tử mà C# cung cấp, bao gồm toán tử số học, so sánh, gán, logic và bitwise, cùng với quy tắc ưu tiên và cách ứng dụng chúng. Cuối cùng, vai trò của bình luận trong việc cải thiện khả năng đọc và bảo trì mã nguồn cũng sẽ được làm rõ.

Mục tiêu của phần này là trang bị cho người học kiến thức vững chắc để thao tác với dữ liệu và thực hiện các phép tính một cách tự tin, tạo cơ sở vững chắc cho việc tiếp thu các khái niệm lập trình phức tạp hơn.

## 1. Chuyển đổi kiểu dữ liệu (Type Conversion)

Trong C#, một ngôn ngữ kiểu mạnh (strongly-typed), mỗi biến được gán một kiểu dữ liệu cụ thể khi khai báo và kiểu đó không thay đổi trong suốt vòng đời của biến. Tuy nhiên, việc chuyển đổi giữa các kiểu dữ liệu là một thao tác phổ biến và cần thiết. C# cung cấp ba cơ chế chuyển đổi chính: chuyển đổi ngầm định, chuyển đổi rõ ràng (ép kiểu), và chuyển đổi giữa các kiểu không tương thích.

### 1.1. Chuyển đổi kiểu ngầm định (Implicit Type Conversion)

Chuyển đổi kiểu ngầm định xảy ra khi trình biên dịch (compiler) có thể tự động chuyển đổi một kiểu dữ liệu sang một kiểu dữ liệu khác mà không có bất kỳ rủi ro mất mát dữ liệu nào. Điều này thường áp dụng khi chuyển đổi từ một kiểu dữ liệu có phạm vi giá trị "nhỏ hơn" sang một kiểu dữ liệu "lớn hơn", nghĩa là kiểu đích có khả năng chứa tất cả các giá trị mà kiểu nguồn có thể biểu diễn.

> [!NOTE]
> Chuyển đổi ngầm định chỉ được thực hiện khi:
> 1.  Các kiểu dữ liệu có tính tương thích.
> 2.  Không có khả năng mất dữ liệu trong quá trình chuyển đổi.

**Cơ chế cấp phát bộ nhớ và hoạt động:**
Khi một kiểu giá trị (Value Type) được chuyển đổi ngầm định sang một kiểu giá trị khác có kích thước lớn hơn, C# thực hiện việc sao chép giá trị từ biến nguồn sang biến đích. Kiểu đích, với không gian bộ nhớ lớn hơn, sẽ được "mở rộng" bằng cách đệm thêm các bit 0 vào các vị trí bit cao hơn để lấp đầy không gian còn lại. Ví dụ, khi một giá trị `byte` (chiếm 1 byte) được gán cho một biến `int` (chiếm 4 byte), giá trị 8 bit của `byte` sẽ được sao chép và mở rộng thành 32 bit trong biến `int`, với 24 bit cao nhất được điền bằng 0. Quá trình này đảm bảo không có mất mát dữ liệu.

**Ví dụ:**

```csharp
// Chuyển đổi từ byte sang int
byte b = 1;      // Biến 'b' lưu trữ giá trị 1 (00000001) trên stack, chiếm 1 byte.
int i = b;       // Biến 'i' lưu trữ giá trị 1 (00000000 00000000 00000000 00000001) trên stack, chiếm 4 byte.
                 // Giá trị của 'b' được sao chép vào 'i' và mở rộng an toàn.
                 // Không có mất mát dữ liệu.

Console.WriteLine($"byte b: {b}"); // Output: byte b: 1
Console.WriteLine($"int i: {i}");   // Output: int i: 1

// Chuyển đổi từ int sang float
int numInt = 100; // Biến 'numInt' lưu trữ 100 (dạng nhị phân 32 bit) trên stack.
float numFloat = numInt; // Biến 'numFloat' lưu trữ 100 (dạng dấu phẩy động 32 bit) trên stack.
                         // Mặc dù cả 'int' và 'float' đều chiếm 4 byte, 'float' có thể biểu diễn một phạm vi giá trị rộng hơn
                         // và lưu trữ số nguyên chính xác trong phạm vi của nó.
                         // Việc chuyển đổi này an toàn và không mất dữ liệu.

Console.WriteLine($"int numInt: {numInt}");     // Output: int numInt: 100
Console.WriteLine($"float numFloat: {numFloat}"); // Output: float numFloat: 100.0 (Lưu ý: float luôn có phần thập phân)

// Một số chuyển đổi ngầm định phổ biến khác:
short s = 50;
long l = s; // short (2 byte) sang long (8 byte)
double d = numFloat; // float (4 byte) sang double (8 byte)
```

### 1.2. Chuyển đổi kiểu rõ ràng (Explicit Type Conversion / Casting)

Chuyển đổi kiểu rõ ràng, hay còn gọi là ép kiểu (casting), là bắt buộc khi có khả năng xảy ra mất mát dữ liệu trong quá trình chuyển đổi. Điều này thường diễn ra khi bạn cố gắng chuyển đổi từ một kiểu dữ liệu "lớn hơn" sang một kiểu dữ liệu "nhỏ hơn" (ví dụ: từ `int` sang `byte`), hoặc giữa các kiểu mà trình biên dịch không thể đảm bảo an toàn.

> [!WARNING]
> Khi trình biên dịch phát hiện nguy cơ mất dữ liệu, nó sẽ không cho phép chuyển đổi ngầm định. Bạn phải sử dụng cú pháp ép kiểu rõ ràng để "thông báo" cho trình biên dịch rằng bạn đã nhận thức được rủi ro và chấp nhận việc chuyển đổi.

**Cú pháp:**
Để thực hiện ép kiểu rõ ràng, bạn đặt tên kiểu đích trong dấu ngoặc đơn `()` ngay trước biểu thức hoặc biến nguồn:
`(kiểu_đích) biểu_thức_nguồn`

**Cơ chế cấp phát bộ nhớ và hoạt động:**
Khi ép kiểu một kiểu giá trị lớn hơn sang một kiểu giá trị nhỏ hơn, C# sẽ cố gắng "cắt bớt" giá trị để phù hợp với kiểu đích. Nếu giá trị nguồn vượt quá phạm vi của kiểu đích, các bit cao hơn (most significant bits) sẽ bị loại bỏ. Điều này dẫn đến việc mất mát dữ liệu và có thể tạo ra kết quả không mong muốn. Trong trường hợp ép kiểu số nguyên lớn hơn sang số nguyên nhỏ hơn (ví dụ: `int` sang `byte`), C# thực hiện một phép toán tương tự như phép modulo. Cụ thể, nó lấy phần dư của giá trị nguồn khi chia cho `(Max_Value_of_Target_Type + 1)`.

**Ví dụ:**

```csharp
// Chuyển đổi từ int sang byte (có khả năng mất dữ liệu)
int i = 1;
// byte b = i; // Lỗi biên dịch: Cannot implicitly convert type 'int' to 'byte'. An explicit conversion exists (are you missing a cast?)

byte b = (byte)i; // Ép kiểu rõ ràng: int sang byte
Console.WriteLine($"int i: {i}");   // Output: int i: 1
Console.WriteLine($"byte b: {b}");   // Output: byte b: 1 (Không mất dữ liệu vì 1 nằm trong phạm vi của byte: 0-255)

// Ví dụ về mất mát dữ liệu (Data Loss)
i = 300; // Giá trị 300 (decimal) vượt quá phạm vi của kiểu byte (0-255).
         // Biểu diễn nhị phân của 300 là: 00000001 00101100 (32-bit int)
         // (để đơn giản, ta chỉ quan tâm 9 bit cuối: 1 00101100)
b = (byte)i; // Khi ép kiểu sang byte (8 bit), bit cao nhất (bit thứ 9) sẽ bị loại bỏ.
             // Phần còn lại là: 00101100 (binary) = 44 (decimal).
             // Đây chính là kết quả của phép toán 300 % 256 = 44.

Console.WriteLine($"int i (300): {i}");       // Output: int i (300): 300
Console.WriteLine($"byte b (sau cast): {b}"); // Output: byte b (sau cast): 44 (Dữ liệu bị mất hoàn toàn)

// Chuyển đổi từ float sang int
float f = 1234.567f; // Biến 'f' lưu trữ giá trị dấu phẩy động 1234.567 trên stack.
int j = (int)f;      // Ép kiểu rõ ràng: float sang int. Phần thập phân sẽ bị cắt bỏ (truncation), không làm tròn.
                     // Biến 'j' lưu trữ giá trị số nguyên 1234 trên stack.

Console.WriteLine($"float f: {f}"); // Output: float f: 1234.567
Console.WriteLine($"int j: {j}");   // Output: int j: 1234 (Phần thập phân .567 đã bị mất)
```

### 1.3. Chuyển đổi giữa các kiểu không tương thích

Trong nhiều trường hợp, các kiểu dữ liệu hoàn toàn không tương thích với nhau (ví dụ: `string` và `int`), và bạn không thể sử dụng ép kiểu rõ ràng thông thường. Để chuyển đổi giữa các kiểu này, C# cung cấp các cơ chế chuyên biệt, chủ yếu thông qua lớp `System.Convert` hoặc các phương thức `Parse()`/`TryParse()` của các kiểu dữ liệu.

**Cơ chế cấp phát bộ nhớ và hoạt động:**
Khi chuyển đổi giữa các kiểu không tương thích như `string` (một Kiểu tham chiếu - Reference Type, lưu trữ địa chỉ của dữ liệu trên heap) và `int` (một Kiểu giá trị - Value Type, lưu trữ trực tiếp giá trị trên stack), không có sự sao chép trực tiếp các bit hay cắt bớt giá trị. Thay vào đó, các phương thức như `Convert.ToInt32()` hoặc `int.Parse()` sẽ đọc *nội dung* của chuỗi, *diễn giải* nó như một số, và *tạo ra một giá trị `int` mới* dựa trên nội dung đó. Quá trình này không thay đổi đối tượng `string` ban đầu trên heap mà tạo ra một thực thể `int` hoàn toàn mới trên stack.

#### 1.3.1. Sử dụng lớp `System.Convert`

Lớp `System.Convert` (thuộc namespace `System`) cung cấp một bộ sưu tập các phương thức tĩnh mạnh mẽ để chuyển đổi giữa hầu hết các kiểu dữ liệu cơ bản trong C#. Các phương thức này thường có dạng `Convert.ToKiểuDữLiệu()`.

**Ví dụ:**

```csharp
string strNumber = "1234"; // strNumber là một kiểu tham chiếu, trỏ đến chuỗi "1234" trên heap.

// Chuyển đổi từ string sang int
int convertedInt = Convert.ToInt32(strNumber); // Phương thức đọc chuỗi, phân tích và tạo một int mới trên stack.
Console.WriteLine($"string strNumber: {strNumber}");     // Output: string strNumber: 1234
Console.WriteLine($"int convertedInt: {convertedInt}"); // Output: int convertedInt: 1234

// Chuyển đổi từ string sang byte (có thể gây lỗi nếu giá trị vượt quá phạm vi)
string strByte = "123";
byte convertedByte = Convert.ToByte(strByte);
Console.WriteLine($"string strByte: {strByte}");         // Output: string strByte: 123
Console.WriteLine($"byte convertedByte: {convertedByte}"); // Output: byte convertedByte: 123

// Chuyển đổi từ string sang bool
string strBool = "true";
bool convertedBool = Convert.ToBoolean(strBool); // "true", "True", "TRUE" (hoặc "false", "False", "FALSE")
Console.WriteLine($"string strBool: {strBool}");         // Output: string strBool: true
Console.WriteLine($"bool convertedBool: {convertedBool}"); // Output: bool convertedBool: True

// Chuyển đổi từ int sang string
int numToString = 456;
string stringFromNum = Convert.ToString(numToString); // Tạo một chuỗi mới trên heap từ giá trị số.
Console.WriteLine($"int numToString: {numToString}");     // Output: int numToString: 456
Console.WriteLine($"string stringFromNum: {stringFromNum}"); // Output: string stringFromNum: 456
```

> [!NOTE]
> Các kiểu như `Int32`, `Int16`, `Int64` trong .NET Framework là tên đầy đủ của các kiểu dữ liệu nguyên thủy tương ứng trong C#:
> *   `byte`: `Byte` (1 byte, 8 bit)
> *   `short`: `Int16` (2 byte, 16 bit)
> *   `int`: `Int32` (4 byte, 32 bit)
> *   `long`: `Int64` (8 byte, 64 bit)
> Việc sử dụng `Convert.ToInt32()` tương đương với việc chuyển đổi sang kiểu `int` trong C#.

#### 1.3.2. Sử dụng phương thức `Parse()`

Hầu hết các kiểu dữ liệu nguyên thủy trong C# (như `int`, `long`, `float`, `double`, `bool`, v.v.) đều cung cấp một phương thức tĩnh `Parse()` nhận một chuỗi và cố gắng chuyển đổi nó sang kiểu tương ứng.

**Ví dụ:**

```csharp
string strValue = "5678";
int parsedInt = int.Parse(strValue); // int.Parse() là một phương thức tiện lợi để chuyển đổi chuỗi sang int.
Console.WriteLine($"string strValue: {strValue}"); // Output: string strValue: 5678
Console.WriteLine($"int parsedInt: {parsedInt}");   // Output: int parsedInt: 5678

string strFloat = "98.76";
float parsedFloat = float.Parse(strFloat);
Console.WriteLine($"string strFloat: {strFloat}");   // Output: string strFloat: 98.76
Console.WriteLine($"float parsedFloat: {parsedFloat}"); // Output: float parsedFloat: 98.76

string strBoolParse = "True";
bool parsedBool = bool.Parse(strBoolParse);
Console.WriteLine($"string strBoolParse: {strBoolParse}"); // Output: string strBoolParse: True
Console.WriteLine($"bool parsedBool: {parsedBool}");       // Output: bool parsedBool: True
```

#### 1.3.3. Xử lý lỗi với `try-catch` và `TryParse()`

Khi chuyển đổi một chuỗi sang kiểu số hoặc Boolean, có nguy cơ xảy ra lỗi nếu chuỗi không phải là một định dạng hợp lệ hoặc giá trị số quá lớn/nhỏ so với kiểu đích. Trong C#, những lỗi này được gọi là **ngoại lệ (exceptions)**, và nếu không được xử lý, chúng sẽ làm ứng dụng của bạn bị treo (crash).

**a. Xử lý ngoại lệ với `try-catch`:**
Khối `try-catch` là một cơ chế mạnh mẽ để xử lý các ngoại lệ một cách duyên dáng, cho phép ứng dụng của bạn tiếp tục chạy ngay cả khi có lỗi.

```csharp
string numberString = "12345678901234567890"; // Chuỗi này quá lớn để chuyển đổi sang int
int i;
try
{
    i = int.Parse(numberString); // Thao tác này sẽ ném OverflowException
    Console.WriteLine($"Chuyển đổi thành công: {i}");
}
catch (OverflowException) // Bắt ngoại lệ khi giá trị quá lớn hoặc quá nhỏ cho kiểu đích
{
    Console.WriteLine($"Lỗi: Giá trị '{numberString}' quá lớn hoặc quá nhỏ để chuyển đổi sang int.");
}
catch (FormatException) // Bắt ngoại lệ khi chuỗi không đúng định dạng số
{
    Console.WriteLine($"Lỗi: Chuỗi '{numberString}' không phải là định dạng số hợp lệ.");
}
catch (Exception ex) // Bắt các loại ngoại lệ khác (nên cụ thể hơn nếu có thể)
{
    Console.WriteLine($"Một lỗi không xác định đã xảy ra: {ex.Message}");
}

string invalidString = "abc";
try
{
    i = int.Parse(invalidString); // Thao tác này sẽ ném FormatException
    Console.WriteLine($"Chuyển đổi thành công: {i}");
}
catch (FormatException)
{
    Console.WriteLine($"Lỗi: Chuỗi '{invalidString}' không phải là định dạng số hợp lệ.");
}
```

> [!NOTE]
> `try-catch` là một kỹ thuật quản lý lỗi thiết yếu, giúp ứng dụng của bạn ổn định và cung cấp phản hồi hữu ích cho người dùng. Chúng ta sẽ khám phá sâu hơn về ngoại lệ và `try-catch` trong các phần nâng cao của khóa học.

**b. Sử dụng `TryParse()` để chuyển đổi an toàn hơn:**
`TryParse()` là phương thức được khuyến nghị để chuyển đổi chuỗi sang số hoặc Boolean khi bạn không chắc chắn về tính hợp lệ của chuỗi đầu vào. Thay vì ném ngoại lệ, `TryParse()` trả về một giá trị `bool` (`true` nếu chuyển đổi thành công, `false` nếu thất bại) và trả về giá trị đã chuyển đổi thông qua một tham số `out`.

```csharp
string s1 = "123";
int result1;
// int.TryParse() cố gắng chuyển đổi s1. Nếu thành công, nó trả về true và gán giá trị cho result1.
if (int.TryParse(s1, out result1))
{
    Console.WriteLine($"'{s1}' chuyển đổi thành int: {result1}"); // Output: '123' chuyển đổi thành int: 123
}
else
{
    Console.WriteLine($"Không thể chuyển đổi '{s1}' thành int.");
}

string s2 = "hello";
int result2;
if (int.TryParse(s2, out result2)) // Thất bại, trả về false, result2 sẽ có giá trị mặc định là 0.
{
    Console.WriteLine($"'{s2}' chuyển đổi thành int: {result2}");
}
else
{
    Console.WriteLine($"Không thể chuyển đổi '{s2}' thành int. (Giá trị mặc định của result2: {result2})");
    // Output: Không thể chuyển đổi 'hello' thành int. (Giá trị mặc định của result2: 0)
}
```

> [!TIP]
> Luôn ưu tiên sử dụng `TryParse()` thay vì `Parse()` hoặc `Convert.ToKiểu()` khi bạn không chắc chắn về định dạng hoặc phạm vi của chuỗi đầu vào. Điều này giúp mã của bạn mạnh mẽ hơn, tránh các lỗi thời gian chạy không mong muốn và cải thiện trải nghiệm người dùng bằng cách xử lý lỗi một cách duyên dáng.

## 2. Toán tử (Operators)

Toán tử là các ký hiệu đặc biệt trong C# được sử dụng để thực hiện các thao tác trên một hoặc nhiều toán hạng (operand). C# cung cấp một tập hợp phong phú các toán tử, được phân loại thành các nhóm chính: số học, so sánh, gán, logic và bitwise.

### 2.1. Toán tử số học (Arithmetic Operators)

Toán tử số học được sử dụng để thực hiện các phép tính toán học cơ bản trên các giá trị số.

| Toán tử | Mô tả                                  | Ví dụ           | Kết quả |
| :------- | :------------------------------------- | :-------------- | :------ |
| `+`      | Cộng                                   | `10 + 3`        | `13`    |
| `-`      | Trừ                                    | `10 - 3`        | `7`     |
| `*`      | Nhân                                   | `10 * 3`        | `30`    |
| `/`      | Chia                                   | `10 / 3` (int)  | `3`     |
|          |                                        | `10.0 / 3.0` (float/double) | `3.333...` |
| `%`      | Chia lấy phần dư (modulo)              | `10 % 3`        | `1`     |
| `++`     | Tăng (tăng giá trị biến lên 1)         | `a++` hoặc `++a` |         |
| `--`     | Giảm (giảm giá trị biến đi 1)          | `a--` hoặc `--a` |         |

**Ví dụ:**

```csharp
int a = 10;
int b = 3;

Console.WriteLine($"a + b = {a + b}"); // Output: a + b = 13
Console.WriteLine($"a - b = {a - b}"); // Output: a - b = 7
Console.WriteLine($"a * b = {a * b}"); // Output: a * b = 30

// Chia số nguyên: Khi cả hai toán hạng đều là số nguyên, kết quả sẽ là phần nguyên.
Console.WriteLine($"a / b = {a / b}"); // Output: a / b = 3

// Để có kết quả thập phân, ít nhất một trong các toán hạng phải là kiểu dấu phẩy động.
// Có thể ép kiểu một trong hai toán hạng để đạt được điều này.
Console.WriteLine($"a / (float)b = {a / (float)b}"); // Output: a / (float)b = 3.3333333
Console.WriteLine($"(double)a / b = {(double)a / b}"); // Output: (double)a / b = 3.3333333333333335

Console.WriteLine($"a % b = {a % b}"); // Output: a % b = 1 (10 chia 3 được 3 dư 1)
```

#### 2.1.1. Toán tử tăng/giảm (`++`, `--`): Tiền tố và Hậu tố

Toán tử `++` (tăng) và `--` (giảm) có thể được sử dụng dưới hai dạng: tiền tố (prefix, `++a` hoặc `--a`) hoặc hậu tố (postfix, `a++` hoặc `a--`). Sự khác biệt quan trọng nằm ở thời điểm giá trị của biến được cập nhật trong một biểu thức.

*   **Hậu tố (`a++`, `a--`):** Giá trị *hiện tại* của biến được sử dụng trong biểu thức trước, sau đó biến mới được tăng hoặc giảm.
*   **Tiền tố (`++a`, `--a`):** Biến được tăng hoặc giảm *trước*, sau đó giá trị *mới* của biến được sử dụng trong biểu thức.

**Ví dụ:**

```csharp
int x = 1;
int y = x++; // Hậu tố:
             // 1. Giá trị hiện tại của x (1) được gán cho y. (y = 1)
             // 2. Sau đó, x tăng lên 2. (x = 2)
Console.WriteLine($"Hậu tố: x = {x}, y = {y}"); // Output: Hậu tố: x = 2, y = 1

int p = 1;
int q = ++p; // Tiền tố:
             // 1. p tăng lên 2. (p = 2)
             // 2. Sau đó, giá trị mới của p (2) được gán cho q. (q = 2)
Console.WriteLine($"Tiền tố: p = {p}, q = {q}"); // Output: Tiền tố: p = 2, q = 2
```

#### 2.1.2. Độ ưu tiên của toán tử (Operator Precedence)

Khi có nhiều toán tử trong một biểu thức, C# tuân theo một bộ quy tắc về độ ưu tiên để xác định thứ tự thực hiện các phép toán. Ví dụ, phép nhân và chia có độ ưu tiên cao hơn phép cộng và trừ. Các toán tử có độ ưu tiên cao hơn sẽ được thực hiện trước.

**Thứ tự ưu tiên cơ bản (từ cao đến thấp):**
1.  Dấu ngoặc đơn `()`
2.  Toán tử một ngôi (ví dụ: `++`, `--`, `!`, `-` (âm))
3.  Nhân, Chia, Modulo (`*`, `/`, `%`)
4.  Cộng, Trừ (`+`, `-`)
5.  Toán tử dịch bit (`<<`, `>>`)
6.  Toán tử so sánh (`<`, `>`, `<=`, `>=`, `==`, `!=`)
7.  Toán tử logic bitwise (`&`, `^`, `|`)
8.  Toán tử logic điều kiện (`&&`, `||`)
9.  Toán tử gán (`=`, `+=`, `-=`, v.v.)

**Ví dụ:**

```csharp
int num1 = 1 + 2 * 3; // Độ ưu tiên: phép nhân (*) cao hơn phép cộng (+).
                     // Thực hiện: 1 + (2 * 3) = 1 + 6 = 7
Console.WriteLine($"1 + 2 * 3 = {num1}"); // Output: 7

int num2 = (1 + 2) * 3; // Dấu ngoặc đơn () có độ ưu tiên cao nhất, thay đổi thứ tự thực hiện.
                     // Thực hiện: (1 + 2) * 3 = 3 * 3 = 9
Console.WriteLine($"(1 + 2) * 3 = {num2}"); // Output: 9
```

> [!TIP]
> Khi bạn không chắc chắn về độ ưu tiên của các toán tử hoặc muốn làm cho mã dễ đọc và dễ hiểu hơn, hãy luôn sử dụng dấu ngoặc đơn `()` để nhóm các phép toán và làm rõ ý định của bạn.

### 2.2. Toán tử so sánh (Comparison Operators)

Toán tử so sánh được sử dụng để so sánh hai giá trị và luôn trả về một giá trị `bool` (`true` hoặc `false`). Chúng là nền tảng cho các câu lệnh điều kiện và vòng lặp.

| Toán tử | Mô tả                       | Ví dụ           |
| :------- | :-------------------------- | :-------------- |
| `==`     | Bằng (kiểm tra sự bằng nhau về giá trị) | `a == b`        |
| `!=`     | Không bằng                  | `a != b`        |
| `>`      | Lớn hơn                     | `a > b`         |
| `<`      | Nhỏ hơn                     | `a < b`         |
| `>=`     | Lớn hơn hoặc bằng           | `a >= b`        |
| `<=`     | Nhỏ hơn hoặc bằng           | `a <= b`        |

**Ví dụ:**

```csharp
int x = 5;
int y = 10;
int z = 5;

Console.WriteLine($"x == y: {x == y}"); // Output: x == y: False (5 không bằng 10)
Console.WriteLine($"x == z: {x == z}"); // Output: x == z: True (5 bằng 5)
Console.WriteLine($"x != y: {x != y}"); // Output: x != y: True (5 không bằng 10)
Console.WriteLine($"x > y: {x > y}");   // Output: x > y: False (5 không lớn hơn 10)
Console.WriteLine($"x < y: {x < y}");   // Output: x < y: True (5 nhỏ hơn 10)
Console.WriteLine($"x >= z: {x >= z}"); // Output: x >= z: True (5 lớn hơn hoặc bằng 5)

string str1 = "hello";
string str2 = "Hello";
string str3 = "hello";
Console.WriteLine($"str1 == str2: {str1 == str2}"); // Output: False (phân biệt chữ hoa/thường)
Console.WriteLine($"str1 == str3: {str1 == str3}"); // Output: True
```

### 2.3. Toán tử gán (Assignment Operators)

Toán tử gán được sử dụng để gán một giá trị cho một biến. Ngoài toán tử gán cơ bản `=`, C# còn cung cấp các toán tử gán kết hợp để thực hiện một phép toán và gán kết quả một cách ngắn gọn.

| Toán tử | Mô tả                                  | Ví dụ           | Tương đương với |
| :------- | :------------------------------------- | :-------------- | :-------------- |
| `=`      | Gán giá trị                            | `a = 10`        |                 |
| `+=`     | Cộng và gán                            | `a += 5`        | `a = a + 5`     |
| `-=`     | Trừ và gán                             | `a -= 2`        | `a = a - 2`     |
| `*=`     | Nhân và gán                            | `a *= 3`        | `a = a * 3`     |
| `/=`     | Chia và gán                            | `a /= 4`        | `a = a / 4`     |
| `%=`     | Modulo và gán                          | `a %= 2`        | `a = a % 2`     |

**Ví dụ:**

```csharp
int myVar = 10;
Console.WriteLine($"myVar ban đầu: {myVar}"); // Output: myVar ban đầu: 10

myVar += 5; // myVar = myVar + 5; => 15
Console.WriteLine($"myVar sau += 5: {myVar}"); // Output: myVar sau += 5: 15

myVar *= 2; // myVar = myVar * 2; => 30
Console.WriteLine($"myVar sau *= 2: {myVar}"); // Output: myVar sau *= 2: 30

myVar /= 3; // myVar = myVar / 3; => 10 (chia số nguyên)
Console.WriteLine($"myVar sau /= 3: {myVar}"); // Output: myVar sau /= 3: 10

myVar %= 3; // myVar = myVar % 3; => 10 % 3 = 1
Console.WriteLine($"myVar sau %= 3: {myVar}"); // Output: myVar sau %= 3: 1
```

### 2.4. Toán tử logic (Logical Operators)

Toán tử logic được sử dụng để kết hợp hoặc đảo ngược các biểu thức Boolean (`true` hoặc `false`). Chúng thường được sử dụng để xây dựng các điều kiện phức tạp trong các câu lệnh điều khiển luồng như `if`, `while`, và `for`.

| Toán tử | Mô tả                                  | Ví dụ           |
| :------- | :------------------------------------- | :-------------- |
| `&&`     | AND logic (cả hai điều kiện đều đúng) | `true && false` |
| `||`     | OR logic (ít nhất một điều kiện đúng) | `true || false` |
| `!`      | NOT logic (đảo ngược giá trị Boolean) | `!true`         |

**Ví dụ:**

```csharp
bool isSunny = true;
bool isWarm = false;

// Toán tử AND (&&): Chỉ trả về true nếu TẤT CẢ các điều kiện là true.
Console.WriteLine($"isSunny && isWarm: {isSunny && isWarm}"); // Output: False (vì isWarm là false)

// Toán tử OR (||): Trả về true nếu ÍT NHẤT MỘT trong các điều kiện là true.
Console.WriteLine($"isSunny || isWarm: {isSunny || isWarm}"); // Output: True (vì isSunny là true)

// Toán tử NOT (!): Đảo ngược giá trị Boolean.
Console.WriteLine($"!isSunny: {!isSunny}"); // Output: False (đảo ngược true thành false)

// Kết hợp các toán tử để tạo điều kiện phức tạp
int age = 25;
bool hasLicense = true;
bool isTeenager = (age >= 13) && (age <= 19); // Điều kiện tuổi teen
bool canDrive = (age >= 18) && hasLicense;   // Điều kiện để có thể lái xe

Console.WriteLine($"Có phải tuổi teen: {isTeenager}"); // Output: Có phải tuổi teen: False
Console.WriteLine($"Có thể lái xe: {canDrive}");     // Output: Có thể lái xe: True
```

> [!TIP]
> **Ngắt mạch (Short-circuiting):**
> Các toán tử logic `&&` và `||` trong C# áp dụng cơ chế ngắt mạch, giúp tối ưu hiệu suất và ngăn ngừa lỗi thời gian chạy.
> *   Với toán tử `&&` (AND): Nếu toán hạng đầu tiên được đánh giá là `false`, toán hạng thứ hai sẽ *không bao giờ* được đánh giá, vì kết quả cuối cùng của biểu thức chắc chắn là `false`.
> *   Với toán tử `||` (OR): Nếu toán hạng đầu tiên được đánh giá là `true`, toán hạng thứ hai sẽ *không bao giờ* được đánh giá, vì kết quả cuối cùng của biểu thức chắc chắn là `true`.
> Tính năng này rất hữu ích, ví dụ, khi bạn cần kiểm tra xem một đối tượng có `null` trước khi truy cập các thuộc tính của nó: `if (myObject != null && myObject.IsValid())`. Nếu `myObject` là `null`, `myObject.IsValid()` sẽ không được gọi, tránh được lỗi `NullReferenceException`.

### 2.5. Toán tử Bitwise (Bitwise Operators)

Toán tử bitwise thực hiện các phép toán trực tiếp trên từng bit của các số nguyên. Chúng thường được sử dụng trong các tình huống lập trình cấp thấp, ví dụ như thao tác với cờ bit, mã hóa/giải mã đơn giản, hoặc làm việc với dữ liệu nhị phân.

| Toán tử | Mô tả                                  | Ví dụ           |
| :------- | :------------------------------------- | :-------------- |
| `&`      | Bitwise AND                            | `a & b`         |
| `|`      | Bitwise OR                             | `a | b`         |
| `^`      | Bitwise XOR (OR loại trừ)              | `a ^ b`         |
| `~`      | Bitwise NOT (bổ sung bit)              | `~a`            |
| `<<`     | Dịch bit sang trái                     | `a << 2`        |
| `>>`     | Dịch bit sang phải                     | `a >> 2`        |

**Ví dụ:**

```csharp
byte x = 5;  // Biểu diễn nhị phân: 00000101
byte y = 3;  // Biểu diễn nhị phân: 00000011

// Bitwise AND (&)
// 00000101 (x)
// 00000011 (y)
// --------
// 00000001 (Kết quả: 1)
Console.WriteLine($"x & y = {x & y}"); // Output: x & y = 1

// Bitwise OR (|)
// 00000101 (x)
// 00000011 (y)
// --------
// 00000111 (Kết quả: 7)
Console.WriteLine($"x | y = {x | y}"); // Output: x | y = 7

// Dịch bit sang trái (<<)
// 00000101 (x) << 1 bit
// --------
// 00001010 (Kết quả: 10)
Console.WriteLine($"x << 1 = {x << 1}"); // Output: x << 1 = 10
```

> [!NOTE]
> Điều quan trọng là phân biệt toán tử bitwise (`&`, `|`) với toán tử logic (`&&`, `||`). Toán tử bitwise hoạt động trên từng bit riêng lẻ của các số nguyên, trong khi toán tử logic hoạt động trên các giá trị Boolean (`true`/`false`).
>
> Mặc dù các toán tử bitwise là một phần của C#, việc tìm hiểu chi tiết về chúng thường nằm ngoài phạm vi của một khóa học C# cơ bản, chủ yếu dành cho các ứng dụng đòi hỏi thao tác cấp thấp. Tuy nhiên, việc nhận biết sự tồn tại và mục đích cơ bản của chúng là rất quan trọng.

## 3. Bình luận (Comments)

Bình luận là những đoạn văn bản mà lập trình viên thêm vào mã nguồn để giải thích, ghi chú hoặc làm rõ một phần mã. Trình biên dịch C# sẽ bỏ qua hoàn toàn các bình luận; chúng không ảnh hưởng đến hoạt động của chương trình mà chỉ phục vụ mục đích cải thiện khả năng đọc, hiểu và bảo trì mã cho con người.

Trong C#, có hai cách chính để viết bình luận:

### 3.1. Bình luận một dòng (Single-line Comments)

Bình luận một dòng bắt đầu bằng hai dấu gạch chéo `//`. Mọi văn bản sau `//` trên cùng một dòng sẽ được coi là bình luận.

```csharp
int counter = 0; // Khởi tạo biến đếm về 0.
counter++;       // Tăng giá trị của biến đếm lên 1.
Console.WriteLine(counter); // In giá trị hiện tại của biến đếm ra màn hình console.
```

### 3.2. Bình luận nhiều dòng (Multi-line Comments)

Bình luận nhiều dòng bắt đầu bằng `/*` và kết thúc bằng `*/`. Mọi văn bản nằm giữa hai ký hiệu này sẽ được coi là bình luận, kể cả khi nó trải dài qua nhiều dòng.

```csharp
/*
 * Đoạn mã này tính toán tổng chi phí của một đơn hàng
 * bao gồm giá sản phẩm, thuế và phí vận chuyển.
 * Đảm bảo các biến productPrice, taxRate và shippingCost
 * đã được khởi tạo trước khi sử dụng.
 */
double productPrice = 150.75;
double taxRate = 0.08; // 8% thuế
double shippingCost = 10.00;

double totalCost = productPrice * (1 + taxRate) + shippingCost;
Console.WriteLine($"Tổng chi phí đơn hàng: {totalCost:C}"); // Định dạng tiền tệ
```

> [!NOTE]
> Mặc dù kiểu `/* ... */` là kiểu bình luận truyền thống và hiệu quả cho các khối văn bản lớn, trong C# hiện đại, việc sử dụng `//` cho từng dòng, ngay cả với các bình luận dài, đã trở nên phổ biến hơn do sự hỗ trợ tốt của các IDE trong việc tự động thêm `//` khi xuống dòng.

### 3.3. Thực hành tốt nhất khi viết bình luận

Viết bình luận hiệu quả là một kỹ năng quan trọng trong lập trình. Dưới đây là một số nguyên tắc thực hành tốt nhất:

*   **Giữ bình luận ở mức tối thiểu:** Mã của bạn nên tự giải thích càng nhiều càng tốt (self-documenting code). Nếu mã được viết rõ ràng, sử dụng tên biến và hàm có ý nghĩa, và có cấu trúc hợp lý, bạn sẽ cần ít bình luận hơn.
*   **Giải thích LÝ DO, không phải CÁI GÌ:** Tránh bình luận giải thích những gì mã đang làm (ví dụ: `// Tăng biến lên 1`). Thay vào đó, hãy giải thích *tại sao* bạn lại viết mã theo cách đó, những giả định đằng sau nó, những ràng buộc hoặc những vấn đề mà nó giải quyết.
    *   **Kém:** `// Khởi tạo i bằng 0`
    *   **Tốt hơn:** `// Sử dụng 'i' làm chỉ mục bắt đầu từ 0 để duyệt qua tất cả các phần tử của mảng, tuân thủ quy ước chỉ mục 0-based.`
*   **Tránh bình luận thừa thãi:** Bình luận thừa thãi dễ bị lỗi thời khi mã thay đổi. Vì bình luận không được biên dịch, không có cách nào để xác thực chúng. Bình luận lỗi thời có thể gây nhầm lẫn và khó hiểu hơn là giúp ích.
*   **Sử dụng bình luận để cảnh báo hoặc ghi chú tạm thời:** Đôi khi, bình luận có thể được dùng để cảnh báo về một hành vi không mong muốn, một giải pháp tạm thời (workaround), một phần mã cần được tối ưu hóa trong tương lai (`// TODO: Cần tối ưu hiệu suất tại đây`), hoặc một lỗi đã biết (`// BUG: Lỗi này cần được sửa trong phiên bản tiếp theo`).
*   **Cập nhật bình luận:** Nếu bạn thay đổi mã, hãy đảm bảo rằng các bình luận liên quan cũng được cập nhật để phản ánh sự thay đổi đó.

> [!TIP]
> Mục tiêu cuối cùng là viết mã dễ hiểu mà không cần quá nhiều bình luận. Bình luận nên là phần bổ sung có giá trị, không phải là sự thay thế cho mã rõ ràng và có cấu trúc tốt.

## Tóm tắt Phần 3: Chuyển đổi kiểu dữ liệu và Toán tử

Phần 3 đã cung cấp cái nhìn sâu sắc về các khía cạnh cơ bản nhưng quan trọng của việc thao tác dữ liệu và thực hiện các phép tính trong C#. Các điểm chính cần nắm vững bao gồm:

*   **C# là ngôn ngữ kiểu mạnh:** Mỗi biến có một kiểu dữ liệu cố định, yêu cầu chuyển đổi rõ ràng khi thay đổi kiểu.
*   **Chuyển đổi kiểu dữ liệu (Type Conversion):**
    *   **Ngầm định:** Tự động, an toàn, không mất dữ liệu (ví dụ: `byte` sang `int`). Cơ chế: sao chép giá trị và đệm bit 0.
    *   **Rõ ràng (Casting):** Yêu cầu ép kiểu thủ công khi có khả năng mất dữ liệu (ví dụ: `int` sang `byte`, `float` sang `int`). Cơ chế: cắt bớt bit cao hoặc phần thập phân, có thể dẫn đến kết quả không mong muốn.
    *   **Giữa các kiểu không tương thích:** Sử dụng `System.Convert` (ví dụ: `Convert.ToInt32()`) hoặc phương thức `Parse()` (ví dụ: `int.Parse()`) để chuyển đổi `string` sang các kiểu số. Cơ chế: đọc nội dung chuỗi, phân tích và tạo giá trị mới; không có sự sao chép bit trực tiếp.
    *   **Xử lý lỗi:** Sử dụng khối `try-catch` để bắt và quản lý ngoại lệ (`OverflowException`, `FormatException`) hoặc phương thức `TryParse()` để chuyển đổi an toàn hơn mà không ném ngoại lệ, đặc biệt khi đầu vào không đáng tin cậy.
*   **Toán tử (Operators):**
    *   **Số học:** `+`, `-`, `*`, `/`, `%` cho các phép tính cơ bản. Lưu ý chia số nguyên.
    *   **Tăng/Giảm (`++`, `--`):** Phân biệt giữa tiền tố (thay đổi rồi dùng) và hậu tố (dùng rồi thay đổi) trong biểu thức.
    *   **Độ ưu tiên của toán tử:** Quy tắc xác định thứ tự thực hiện phép toán; sử dụng dấu ngoặc đơn `()` để ghi đè độ ưu tiên mặc định.
    *   **So sánh:** `==`, `!=`, `>`, `<`, `>=`, `<=` để so sánh giá trị và luôn trả về `bool`.
    *   **Gán:** `=` và các toán tử gán kết hợp (`+=`, `-=`, `*=`, `/=`, `%=`) để gán giá trị cho biến một cách ngắn gọn.
    *   **Logic:** `&&` (AND), `||` (OR), `!` (NOT) để kết hợp hoặc đảo ngược các biểu thức Boolean. Ghi nhớ tính năng ngắt mạch (short-circuiting) để tối ưu hiệu suất và ngăn ngừa lỗi.
    *   **Bitwise:** `&`, `|`, `^`, `~`, `<<`, `>>` để thao tác trên từng bit của số nguyên (thường dùng trong lập trình cấp thấp), phân biệt với toán tử logic.
*   **Bình luận (Comments):**
    *   Sử dụng `//` cho bình luận một dòng và `/* ... */` cho bình luận nhiều dòng.
    *   Thực hành tốt nhất: ưu tiên mã tự giải thích, giải thích *lý do* thay vì *cái gì*, giữ bình luận tối thiểu và cập nhật chúng.

Với kiến thức vững chắc về chuyển đổi kiểu dữ liệu và toán tử, bạn đã có thể xây dựng các biểu thức, thực hiện các phép tính và thao tác cơ bản với dữ liệu trong chương trình C#. Đây là nền tảng thiết yếu để tiếp tục khám phá các cấu trúc dữ liệu và logic phức tạp hơn. Trong phần tiếp theo, chúng ta sẽ mở rộng sang các kiểu dữ liệu không nguyên thủy như lớp, cấu trúc, chuỗi và mảng, cùng với khái niệm quan trọng về Kiểu giá trị và Kiểu tham chiếu, mở ra khả năng làm việc với các hệ thống phức tạp hơn. Hẹn gặp lại bạn ở phần tiếp theo!

<!-- REVIEWED_BY_AGENT -->
