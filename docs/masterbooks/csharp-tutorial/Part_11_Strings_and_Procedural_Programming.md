# Bài 11: Làm việc với Chuỗi và Lập trình thủ tục

Trong chương này, chúng ta sẽ khám phá hai trụ cột cơ bản nhưng vô cùng quan trọng trong lập trình C#: kiểu dữ liệu chuỗi (string) và mô hình lập trình thủ tục (procedural programming). Chuỗi là kiểu dữ liệu không thể thiếu để xử lý văn bản, dữ liệu người dùng và thông tin hiển thị trong hầu hết các ứng dụng. Tuy nhiên, cách C# quản lý và thao tác với chuỗi có những đặc thù riêng biệt, đặc biệt là về cơ chế cấp phát bộ nhớ và tính bất biến, mà việc hiểu rõ chúng là chìa khóa để viết mã hiệu quả. Chúng ta sẽ tìm hiểu sâu về các phương thức thao tác chuỗi, khám phá `StringBuilder` như một giải pháp tối ưu cho các tình huống cần chỉnh sửa chuỗi liên tục, và đặt nền móng vững chắc cho việc tổ chức mã nguồn thông qua lập trình thủ tục – một bước chuẩn bị cần thiết trước khi đi sâu vào lập trình hướng đối tượng.

## 1. Hiểu về Chuỗi (String) trong C#

Trong C#, từ khóa `string` không phải là một kiểu dữ liệu nguyên thủy (value type) như `int` hay `bool`. Thay vào đó, `string` là một bí danh (alias) cho lớp `System.String` trong .NET Framework. Điều này có nghĩa là, khi bạn khai báo một biến `string`, bạn đang làm việc với một **đối tượng** của lớp `System.String`, và nó là một **kiểu tham chiếu (reference type)**.

### 1.1 Khái niệm cơ bản về Chuỗi: Tính bất biến và cơ chế bộ nhớ

Để hiểu sâu về cách chuỗi hoạt động trong C#, chúng ta cần nắm vững hai khái niệm cốt lõi: **kiểu tham chiếu** và **tính bất biến**.

#### 1.1.1 Chuỗi là Kiểu tham chiếu (Reference Type)

Khi bạn khai báo một biến `string`, biến đó không trực tiếp lưu trữ giá trị chuỗi mà lưu trữ một **tham chiếu (reference)** đến một đối tượng `String` thực sự nằm trên **Heap** (vùng bộ nhớ động).

*   **Stack:** Chứa các biến cục bộ và tham chiếu.
*   **Heap:** Chứa các đối tượng thực sự.

**Ví dụ:**
```csharp
string s1 = "Hello";
string s2 = s1; // s2 cũng tham chiếu đến đối tượng "Hello"
```
Trong trường hợp này, cả `s1` và `s2` đều là các biến trên Stack, và chúng cùng trỏ (tham chiếu) đến một đối tượng `String` duy nhất chứa giá trị "Hello" trên Heap.

#### 1.1.2 Tính bất biến (Immutability) của Chuỗi

Đây là đặc điểm quan trọng nhất của `System.String`. **Một khi một đối tượng chuỗi đã được tạo trên Heap, nội dung của nó không thể thay đổi.**

Bất kỳ thao tác nào dường như "thay đổi" một chuỗi (như nối, thay thế, cắt tỉa) thực chất đều tạo ra một đối tượng chuỗi *mới* trên Heap và gán tham chiếu của biến sang đối tượng mới này. Đối tượng chuỗi cũ vẫn tồn tại trên Heap nhưng không còn được tham chiếu bởi biến đó nữa, và sẽ trở thành "rác" để Bộ thu gom rác (Garbage Collector - GC) thu hồi.

**Cơ chế cấp phát bộ nhớ minh họa:**

Hãy xem xét ví dụ sau:
```csharp
string message = "Xin chào";
message = message + " thế giới";
message = message.ToUpper();
```

1.  `string message = "Xin chào";`
    *   Một đối tượng `String` chứa "Xin chào" được tạo trên Heap.
    *   Biến `message` trên Stack lưu trữ tham chiếu đến đối tượng này.
    ```
    Stack: [message] ----> Heap: ["Xin chào"] (Địa chỉ A)
    ```

2.  `message = message + " thế giới";`
    *   C# không sửa đổi đối tượng "Xin chào" tại Địa chỉ A.
    *   Thay vào đó, một đối tượng `String` *mới* chứa "Xin chào thế giới" được tạo trên Heap (tại Địa chỉ B).
    *   Biến `message` trên Stack được cập nhật để tham chiếu đến đối tượng mới này.
    *   Đối tượng "Xin chào" tại Địa chỉ A không còn được `message` tham chiếu nữa và trở thành ứng cử viên cho GC.
    ```
    Stack: [message] ----> Heap: ["Xin chào thế giới"] (Địa chỉ B)
                        Heap: ["Xin chào"] (Địa chỉ A - chờ GC)
    ```

3.  `message = message.ToUpper();`
    *   Tương tự, một đối tượng `String` *mới* chứa "XIN CHÀO THẾ GIỚI" được tạo trên Heap (tại Địa chỉ C).
    *   Biến `message` trên Stack được cập nhật để tham chiếu đến đối tượng mới này.
    *   Đối tượng "Xin chào thế giới" tại Địa chỉ B không còn được `message` tham chiếu nữa và trở thành ứng cử viên cho GC.
    ```
    Stack: [message] ----> Heap: ["XIN CHÀO THẾ GIỚI"] (Địa chỉ C)
                        Heap: ["Xin chào thế giới"] (Địa chỉ B - chờ GC)
                        Heap: ["Xin chào"] (Địa chỉ A - chờ GC)
    ```

#### 1.1.3 Hậu quả của tính bất biến

Đối với các thao tác chuỗi thường xuyên (ví dụ: trong một vòng lặp), việc tạo ra nhiều đối tượng chuỗi mới có thể gây ra gánh nặng đáng kể về hiệu suất và tăng áp lực lên Bộ thu gom rác. Điều này dẫn đến:
*   **Tiêu tốn bộ nhớ:** Nhiều đối tượng tạm thời được tạo và sau đó bị loại bỏ.
*   **Tiêu tốn CPU:** Thời gian xử lý bị lãng phí cho việc cấp phát bộ nhớ, sao chép dữ liệu, và thu gom rác.

Chính vì lý do này, C# cung cấp `StringBuilder` cho các tình huống cần thao tác chuỗi liên tục, như chúng ta sẽ tìm hiểu ở phần sau.

### 1.2 Các phương thức thao tác chuỗi cơ bản

Lớp `String` cung cấp một bộ sưu tập phong phú các phương thức để xử lý và thao tác chuỗi một cách hiệu quả.

#### 1.2.1 Định dạng và làm sạch chuỗi

Các phương thức này giúp chuẩn hóa và loại bỏ dữ liệu không mong muốn.

*   `ToLower()`: Chuyển đổi tất cả các ký tự trong chuỗi thành chữ thường.
*   `ToUpper()`: Chuyển đổi tất cả các ký tự trong chuỗi thành chữ hoa.
*   `Trim()`: Loại bỏ tất cả các khoảng trắng (bao gồm space, tab, newline, v.v.) ở đầu và cuối chuỗi. Đây là phương thức cực kỳ quan trọng khi xử lý dữ liệu nhập từ người dùng để đảm bảo tính nhất quán.
*   `TrimStart()`: Loại bỏ khoảng trắng ở đầu chuỗi.
*   `TrimEnd()`: Loại bỏ khoảng trắng ở cuối chuỗi.

```csharp
string rawInput = "  mosh HAMEDANI \t\n ";

Console.WriteLine($"Gốc: '{rawInput}'");
Console.WriteLine($"Trim: '{rawInput.Trim()}'"); // Loại bỏ khoảng trắng ở đầu và cuối
Console.WriteLine($"ToLower: '{rawInput.ToLower()}'"); // Chuyển sang chữ thường
Console.WriteLine($"ToUpper: '{rawInput.ToUpper()}'"); // Chuyển sang chữ hoa
Console.WriteLine($"TrimStart: '{rawInput.TrimStart()}'"); // Chỉ loại bỏ khoảng trắng ở đầu
Console.WriteLine($"TrimEnd: '{rawInput.TrimEnd()}'");   // Chỉ loại bỏ khoảng trắng ở cuối
```

#### 1.2.2 Tìm kiếm và kiểm tra trong chuỗi

Các phương thức này giúp xác định vị trí hoặc sự tồn tại của ký tự/chuỗi con.

*   `IndexOf(char value)` / `IndexOf(string value)`: Trả về chỉ mục (index) của lần xuất hiện đầu tiên của một ký tự hoặc chuỗi con trong chuỗi. Chỉ mục bắt đầu từ 0. Nếu không tìm thấy, trả về -1.
*   `LastIndexOf(char value)` / `LastIndexOf(string value)`: Trả về chỉ mục của lần xuất hiện cuối cùng của một ký tự hoặc chuỗi con.
*   `Contains(string value)`: Trả về `true` nếu chuỗi chứa chuỗi con được chỉ định, ngược lại trả về `false`.
*   `StartsWith(string value)`: Trả về `true` nếu chuỗi bắt đầu bằng chuỗi con được chỉ định.
*   `EndsWith(string value)`: Trả về `true` nếu chuỗi kết thúc bằng chuỗi con được chỉ định.

```csharp
string sentence = "Hôm nay trời đẹp, hôm nay tôi vui.";

Console.WriteLine($"Index của 'nay': {sentence.IndexOf("nay")}");       // Tìm thấy 'nay' đầu tiên (tại index 4)
Console.WriteLine($"LastIndex của 'nay': {sentence.LastIndexOf("nay")}"); // Tìm thấy 'nay' cuối cùng (tại index 21)
Console.WriteLine($"Index của 'x': {sentence.IndexOf('x')}");         // Không tìm thấy, trả về -1

Console.WriteLine($"Contains 'trời': {sentence.Contains("trời")}");     // True
Console.WriteLine($"Starts with 'Hôm': {sentence.StartsWith("Hôm")}"); // True
Console.WriteLine($"Ends with 'vui.': {sentence.EndsWith("vui.")}");   // True
```

#### 1.2.3 Trích xuất chuỗi con

*   `Substring(int startIndex)`: Trích xuất một chuỗi con bắt đầu từ `startIndex` đến cuối chuỗi.
*   `Substring(int startIndex, int length)`: Trích xuất một chuỗi con bắt đầu từ `startIndex` với độ dài là `length` ký tự.

```csharp
string fullName = "Mosh Hamedani";
int spaceIndex = fullName.IndexOf(' '); // Tìm vị trí của khoảng trắng

string firstName = fullName.Substring(0, spaceIndex);      // "Mosh" (từ index 0, dài spaceIndex ký tự)
string lastName = fullName.Substring(spaceIndex + 1);      // "Hamedani" (từ sau khoảng trắng đến hết)

Console.WriteLine($"Tên: {firstName}");
Console.WriteLine($"Họ: {lastName}");

string email = "user@example.com";
string domain = email.Substring(email.IndexOf('@') + 1); // "example.com"
Console.WriteLine($"Tên miền: {domain}");
```

#### 1.2.4 Thay thế ký tự/chuỗi con

*   `Replace(char oldChar, char newChar)`: Thay thế tất cả các lần xuất hiện của một ký tự bằng một ký tự khác.
*   `Replace(string oldValue, string newValue)`: Thay thế tất cả các lần xuất hiện của một chuỗi con bằng một chuỗi con khác.

```csharp
string text = "Xin chào thế giới.";
string newText = text.Replace("thế giới", "bạn"); // "Xin chào bạn."

Console.WriteLine($"Chuỗi gốc: {text}");
Console.WriteLine($"Chuỗi mới: {newText}");

string priceWithDashes = "123-456-789";
string cleanPrice = priceWithDashes.Replace("-", ""); // "123456789"
Console.WriteLine($"Giá sạch: {cleanPrice}");

string commaText = "apple,banana,orange";
string spaceText = commaText.Replace(',', ' '); // "apple banana orange"
Console.WriteLine($"Chuỗi có khoảng trắng: {spaceText}");
```

#### 1.2.5 Kiểm tra tính hợp lệ của chuỗi

Các phương thức tĩnh này rất hữu ích khi xác thực đầu vào từ người dùng hoặc kiểm tra dữ liệu.

*   `string.IsNullOrEmpty(string value)`: Kiểm tra xem chuỗi có phải là `null` hoặc chuỗi rỗng (`""`) hay không. Trả về `true` nếu là một trong hai trường hợp này.
*   `string.IsNullOrWhiteSpace(string value)`: Kiểm tra xem chuỗi có phải là `null`, chuỗi rỗng (`""`) hoặc chỉ chứa các ký tự khoảng trắng (space, tab, newline, v.v.) hay không. Trả về `true` nếu là một trong các trường hợp này.

```csharp
string name1 = null;
string name2 = "";
string name3 = "   "; // Chỉ chứa khoảng trắng
string name4 = "Mosh";
string name5 = "  \t\n  "; // Chỉ chứa khoảng trắng và tab/newline

Console.WriteLine($"IsNullOrEmpty(name1): {string.IsNullOrEmpty(name1)}"); // True
Console.WriteLine($"IsNullOrEmpty(name2): {string.IsNullOrEmpty(name2)}"); // True
Console.WriteLine($"IsNullOrEmpty(name3): {string.IsNullOrEmpty(name3)}"); // False (chỉ có khoảng trắng, không rỗng)
Console.WriteLine($"IsNullOrWhiteSpace(name3): {string.IsNullOrWhiteSpace(name3)}"); // True
Console.WriteLine($"IsNullOrWhiteSpace(name5): {string.IsNullOrWhiteSpace(name5)}"); // True
Console.WriteLine($"IsNullOrEmpty(name4): {string.IsNullOrEmpty(name4)}"); // False
Console.WriteLine($"IsNullOrWhiteSpace(name4): {string.IsNullOrWhiteSpace(name4)}"); // False
```

> [!TIP]
> Luôn ưu tiên sử dụng `string.IsNullOrWhiteSpace()` khi kiểm tra đầu vào của người dùng. Một chuỗi chỉ chứa khoảng trắng thường được coi là không hợp lệ trong hầu hết các tình huống nhập liệu, và `IsNullOrWhiteSpace()` xử lý trường hợp này một cách chính xác.

#### 1.2.6 Tách và Nối chuỗi

*   `Split(char[] separator)` / `Split(string[] separator)`: Tách một chuỗi thành một mảng các chuỗi con dựa trên một hoặc nhiều ký tự/chuỗi phân tách. Có thể cung cấp các tùy chọn để loại bỏ các phần tử rỗng.
*   `string.Join(string separator, IEnumerable<string> values)`: Phương thức tĩnh, nối các phần tử của một mảng hoặc danh sách các chuỗi thành một chuỗi duy nhất, sử dụng một chuỗi phân tách được chỉ định.

```csharp
string fullName = "Mosh Hamedani";
string[] names = fullName.Split(' '); // Tách bằng khoảng trắng

Console.WriteLine($"Phần tên: {names[0]}"); // Mosh
Console.WriteLine($"Phần họ: {names[1]}"); // Hamedani

string emailAddresses = "user1@example.com;user2@domain.net;user3@test.org";
// Tách bằng dấu chấm phẩy, loại bỏ các mục rỗng nếu có nhiều dấu phân cách liên tiếp
string[] emails = emailAddresses.Split(';', StringSplitOptions.RemoveEmptyEntries);
foreach (var email in emails)
{
    Console.WriteLine($"- {email}");
}

string[] words = { "Hôm", "nay", "trời", "đẹp" };
string joinedSentence = string.Join(" ", words); // Nối các từ bằng khoảng trắng
Console.WriteLine($"Câu nối: {joinedSentence}"); // "Hôm nay trời đẹp"

List<int> numbers = new List<int> { 10, 20, 30 };
// Chuyển List<int> sang List<string> để dùng Join
string joinedNumbers = string.Join(", ", numbers.Select(n => n.ToString())); // Dùng LINQ Select để chuyển đổi
Console.WriteLine($"Số nối: {joinedNumbers}"); // "10, 20, 30"
```

### 1.3 Chuyển đổi giữa Chuỗi và Số

Trong các ứng dụng thực tế, việc chuyển đổi giữa chuỗi (thường là đầu vào người dùng) và các kiểu số là rất phổ biến.

#### 1.3.1 Từ Chuỗi sang Số

Có ba phương pháp chính để chuyển đổi chuỗi sang số, mỗi phương pháp có cách xử lý lỗi khác nhau:

*   `int.Parse(string s)`: Chuyển đổi biểu diễn chuỗi của một số thành một số nguyên 32-bit.
    *   **Ưu điểm:** Đơn giản, trực tiếp.
    *   **Nhược điểm:** Nếu chuỗi không phải là định dạng số hợp lệ hoặc là `null`, nó sẽ ném ra một ngoại lệ (`FormatException` hoặc `ArgumentNullException`). Điều này có thể làm dừng chương trình nếu không được xử lý bằng `try-catch`.

*   `Convert.ToInt32(string value)`: Tương tự như `Parse`, nhưng linh hoạt hơn một chút.
    *   **Ưu điểm:** Nếu `value` là `null` hoặc chuỗi rỗng (`""`), nó sẽ trả về giá trị mặc định của kiểu số (ví dụ: 0 cho `int`) thay vì ném ngoại lệ.
    *   **Nhược điểm:** Vẫn ném `FormatException` nếu chuỗi không phải là số hợp lệ (ví dụ: "abc", "10.5").

*   `int.TryParse(string s, out int result)`: Đây là phương pháp **được khuyến nghị** nhất khi bạn không chắc chắn về tính hợp lệ của chuỗi đầu vào, đặc biệt là từ người dùng.
    *   **Ưu điểm:** Không bao giờ ném ngoại lệ. Nó trả về `true` nếu chuyển đổi thành công và gán giá trị vào biến `result`, ngược lại trả về `false` và gán 0 vào `result`.
    *   **Nhược điểm:** Cần một biến `out` và kiểm tra giá trị trả về.

```csharp
string strAge = "25";
int age = int.Parse(strAge); // age = 25
Console.WriteLine($"int.Parse(\"25\"): {age}");

string strPriceDecimal = "10.99";
// int price = int.Parse(strPriceDecimal); // Lỗi: FormatException vì có dấu thập phân
// int price = Convert.ToInt32(strPriceDecimal); // Lỗi: FormatException

string strNull = null;
// int num1 = int.Parse(strNull); // Lỗi: ArgumentNullException
int num2 = Convert.ToInt32(strNull); // num2 = 0
Console.WriteLine($"Convert.ToInt32(null): {num2}");

string strInvalid = "abc";
// int num3 = int.Parse(strInvalid); // Lỗi: FormatException
// int num4 = Convert.ToInt32(strInvalid); // Lỗi: FormatException

// Sử dụng TryParse (phương pháp tốt nhất cho đầu vào người dùng)
Console.Write("Nhập một số: ");
string userInput = Console.ReadLine();
int parsedNumber;
if (int.TryParse(userInput, out parsedNumber))
{
    Console.WriteLine($"Bạn đã nhập số: {parsedNumber}");
}
else
{
    Console.WriteLine($"'{userInput}' không phải là định dạng số hợp lệ.");
}

// TryParse với chuỗi không hợp lệ
string testStr = "123x";
if (int.TryParse(testStr, out int numResult))
{
    Console.WriteLine($"TryParse(\"123x\"): Thành công, kết quả = {numResult}");
}
else
{
    Console.WriteLine($"TryParse(\"123x\"): Thất bại, kết quả = {numResult} (giá trị mặc định)");
}
```

> [!NOTE]
> Luôn ưu tiên `TryParse` khi xử lý đầu vào không đáng tin cậy (ví dụ: từ người dùng, từ file, từ mạng) để tránh các ngoại lệ không mong muốn và làm cho chương trình của bạn mạnh mẽ hơn.

#### 1.3.2 Từ Số sang Chuỗi

*   `ToString()`: Mọi đối tượng trong C# đều có phương thức `ToString()`, trả về biểu diễn chuỗi của đối tượng đó. Đối với các kiểu số, nó sẽ trả về chuỗi số.
*   `ToString(string format)`: Cho phép bạn định dạng số thành chuỗi theo các quy tắc định dạng cụ thể (chuỗi định dạng chuẩn hoặc tùy chỉnh).

Các chuỗi định dạng chuẩn phổ biến:
*   `C` (Currency): Định dạng tiền tệ. Ví dụ: `1234.56.ToString("C")` -> "$1,234.56" (tùy thuộc vào văn hóa hệ thống). `C0` sẽ không có số thập phân và làm tròn.
*   `D` (Decimal): Định dạng số thập phân. `D` theo sau bởi một số chỉ định số chữ số tối thiểu, thêm số 0 ở đầu nếu cần. Ví dụ: `5.ToString("D3")` -> "005".
*   `E` (Exponential): Định dạng số mũ.
*   `F` (Fixed-point): Định dạng số dấu phẩy động cố định. `F` theo sau bởi một số chỉ định số chữ số thập phân. Ví dụ: `123.456.ToString("F2")` -> "123.46".
*   `N` (Number): Định dạng số với dấu phân cách hàng nghìn và số chữ số thập phân mặc định.
*   `P` (Percent): Định dạng phần trăm.
*   `X` (Hexadecimal): Định dạng số thập lục phân. `X` theo sau bởi một số chỉ định số chữ số tối thiểu, thêm số 0 ở đầu nếu cần.

```csharp
int number = 12345;
Console.WriteLine($"Số sang chuỗi: {number.ToString()}");      // "12345"
Console.WriteLine($"Số với D5: {number.ToString("D5")}");     // "012345"

double price = 29.954;
Console.WriteLine($"Giá tiền (C): {price.ToString("C")}");    // Ví dụ: "$29.95" (tùy thuộc vùng miền)
Console.WriteLine($"Giá tiền (C0): {price.ToString("C0")}");  // Ví dụ: "$30" (làm tròn)
Console.WriteLine($"Giá tiền (F2): {price.ToString("F2")}");  // "29.95"
Console.WriteLine($"Giá tiền (N1): {price.ToString("N1")}");  // "29.9"

int hexNumber = 255;
Console.WriteLine($"Hex (X): {hexNumber.ToString("X")}");     // "FF"
Console.WriteLine($"Hex (X4): {hexNumber.ToString("X4")}");   // "00FF"

double percentage = 0.75;
Console.WriteLine($"Phần trăm (P): {percentage.ToString("P")}"); // "75.00 %"
```

### 1.4 Ví dụ tổng hợp: Tóm tắt văn bản

Hãy cùng giải quyết một bài toán thực tế: tóm tắt một đoạn văn bản dài thành một đoạn ngắn hơn, nhưng đảm bảo không cắt ngang từ. Đây là một kỹ thuật phổ biến trên các blog hoặc trang tin tức.

**Mục tiêu:**
1.  Nếu văn bản đủ ngắn, hiển thị toàn bộ.
2.  Nếu văn bản dài, cắt bớt thành độ dài tối đa cho phép, nhưng chỉ cắt ở ranh giới từ. Thêm dấu "..." ở cuối.

```csharp
using System;
using System.Collections.Generic;
// using System.Linq; // Không cần thiết cho logic chính trong ví dụ này

public class StringUtilities
{
    /// <summary>
    /// Tóm tắt một đoạn văn bản dài thành một chuỗi ngắn hơn, không cắt ngang từ.
    /// </summary>
    /// <param name="text">Đoạn văn bản cần tóm tắt.</param>
    /// <param name="maxLength">Độ dài tối đa mong muốn của chuỗi tóm tắt (mặc định là 20).</param>
    /// <returns>Chuỗi tóm tắt.</returns>
    public static string SummarizeText(string text, int maxLength = 20)
    {
        // 1. Xử lý trường hợp chuỗi rỗng, null hoặc chỉ có khoảng trắng
        if (string.IsNullOrWhiteSpace(text))
            return "";

        // 2. Nếu độ dài văn bản nhỏ hơn hoặc bằng độ dài tối đa, trả về nguyên bản
        if (text.Length <= maxLength)
            return text;

        // 3. Tách văn bản thành các từ
        // string.Split() trả về một mảng các chuỗi.
        // StringSplitOptions.RemoveEmptyEntries đảm bảo không có chuỗi rỗng nếu có nhiều khoảng trắng liên tiếp.
        string[] words = text.Split(' ', StringSplitOptions.RemoveEmptyEntries);

        // 4. Lặp qua các từ và xây dựng chuỗi tóm tắt
        int totalCharacters = 0;
        var summaryWords = new List<string>(); // Danh sách để lưu trữ các từ trong bản tóm tắt

        foreach (var word in words)
        {
            // Cộng độ dài của từ và 1 khoảng trắng (tưởng tượng) sau mỗi từ
            // Nếu thêm từ này vào mà vượt quá maxLength, thì không thêm nữa
            if (totalCharacters + word.Length + 1 > maxLength && summaryWords.Count > 0)
                break; // Dừng lại nếu từ tiếp theo làm tổng độ dài vượt quá ngưỡng

            totalCharacters += word.Length + 1; // Cập nhật tổng độ dài đã bao gồm từ và khoảng trắng
            summaryWords.Add(word);             // Thêm từ vào danh sách tóm tắt
        }

        // Nếu không có từ nào được thêm vào (ví dụ maxLength rất nhỏ và từ đầu tiên quá dài)
        if (summaryWords.Count == 0)
        {
            // Trả về một phần của từ đầu tiên hoặc rỗng tùy theo yêu cầu
            // Ở đây, chúng ta sẽ trả về phần đầu của từ đầu tiên nếu nó quá dài
            // Hoặc có thể cân nhắc trả về "" hoặc chỉ "..."
            return words[0].Substring(0, Math.Min(words[0].Length, maxLength - 3)) + "...";
        }

        // 5. Nối các từ trong danh sách tóm tắt lại và thêm dấu "..."
        // string.Join() nối các phần tử của một mảng/IEnumerable thành một chuỗi
        return string.Join(" ", summaryWords) + "...";
    }
}

public class Program
{
    public static void Main(string[] args)
    {
        string longText = "Đây là một đoạn văn bản thực sự rất dài, được sử dụng để minh họa cách chúng ta có thể tóm tắt nó một cách thông minh.";
        string shortText = "Văn bản ngắn.";
        string veryLongWordText = "SiêuDàiSiêuDàiSiêuDàiSiêuDàiSiêuDàiSiêuDàiSiêuDài";

        Console.WriteLine("--- Tóm tắt văn bản ---");
        
        // Sử dụng giá trị maxLength mặc định là 20
        string summary1 = StringUtilities.SummarizeText(longText); 
        Console.WriteLine($"Tóm tắt 1 (max 20): {summary1}"); // "Đây là một đoạn văn bản..."

        // Chỉ định maxLength là 50
        string summary2 = StringUtilities.SummarizeText(longText, 50); 
        Console.WriteLine($"Tóm tắt 2 (max 50): {summary2}"); // "Đây là một đoạn văn bản thực sự rất dài, được sử dụng..."

        string summary3 = StringUtilities.SummarizeText(shortText);
        Console.WriteLine($"Tóm tắt 3 (max 20): {summary3}"); // "Văn bản ngắn."

        string emptyText = "";
        string summary4 = StringUtilities.SummarizeText(emptyText);
        Console.WriteLine($"Tóm tắt 4 (rỗng): '{summary4}'"); // ''

        string summary5 = StringUtilities.SummarizeText(veryLongWordText, 10);
        Console.WriteLine($"Tóm tắt 5 (từ dài): {summary5}"); // "SiêuDàiSi..." (đã điều chỉnh logic để xử lý tốt hơn)
    }
}
```

## 2. Tối ưu hóa thao tác chuỗi với StringBuilder

Như đã phân tích ở trên, tính bất biến của `string` dẫn đến việc tạo ra nhiều đối tượng chuỗi tạm thời trên Heap khi thực hiện các thao tác chỉnh sửa chuỗi liên tục. Điều này có thể gây ra vấn đề hiệu suất đáng kể trong các ứng dụng cần xây dựng chuỗi động hoặc xử lý lượng lớn văn bản. Để giải quyết vấn đề này, C# cung cấp lớp `System.Text.StringBuilder`.

### 2.1 Hạn chế của Chuỗi bất biến trong các thao tác lặp lại

Hãy tưởng tượng bạn cần xây dựng một chuỗi rất dài bằng cách nối thêm các phần nhỏ trong một vòng lặp, ví dụ như tạo một báo cáo từ hàng nghìn dòng dữ liệu:

```csharp
string report = "Báo cáo doanh thu:\n";
for (int i = 0; i < 1000; i++)
{
    report += $" - Dòng dữ liệu {i}: Giá trị {i * 100}\n"; // Mỗi lần += tạo ra một đối tượng string mới
}
// report = "Báo cáo doanh thu:\n - Dòng dữ liệu 0: Giá trị 0\n - Dòng dữ liệu 1: Giá trị 100\n ..."
```
Trong ví dụ trên, mỗi lần thực hiện phép nối chuỗi (`+=`), .NET sẽ:
1.  Cấp phát một vùng nhớ mới trên Heap đủ lớn để chứa chuỗi kết quả.
2.  Sao chép nội dung của chuỗi `report` cũ vào vùng nhớ mới.
3.  Sao chép nội dung của chuỗi mới được nối thêm vào vùng nhớ mới.
4.  Cập nhật biến `report` để trỏ đến vùng nhớ mới.
5.  Đối tượng chuỗi cũ trở thành rác và chờ GC thu hồi.

Nếu vòng lặp chạy hàng nghìn lần, hàng nghìn đối tượng `string` sẽ được tạo ra và sau đó bị loại bỏ, gây lãng phí bộ nhớ và CPU cho việc cấp phát, sao chép và thu gom rác. Điều này đặc biệt nghiêm trọng khi các chuỗi trở nên rất dài, vì việc sao chép dữ liệu sẽ tốn kém hơn.

### 2.2 Giới thiệu StringBuilder: Giải pháp Khả biến (Mutable)

`StringBuilder` là một lớp được thiết kế đặc biệt để thao tác chuỗi hiệu quả. Không giống như `string`, `StringBuilder` là **khả biến (mutable)**. Điều này có nghĩa là khi bạn thêm, chèn, xóa hoặc thay thế các ký tự, `StringBuilder` sẽ sửa đổi nội dung của nó *tại chỗ* trong bộ nhớ mà không cần tạo đối tượng mới cho mỗi thao tác.

`StringBuilder` duy trì một **bộ đệm nội bộ (internal buffer)**, thường là một mảng ký tự. Khi bạn thêm dữ liệu, nó sẽ ghi vào bộ đệm này. Nếu bộ đệm hiện tại không đủ chỗ, `StringBuilder` sẽ tự động cấp phát một bộ đệm lớn hơn (thường là gấp đôi kích thước hiện tại), sao chép dữ liệu cũ sang bộ đệm mới, và tiếp tục ghi. Mặc dù việc mở rộng bộ đệm vẫn tốn kém, nhưng nó xảy ra ít thường xuyên hơn nhiều so với việc tạo một đối tượng `string` mới cho mỗi thao tác, dẫn đến hiệu suất tổng thể tốt hơn đáng kể.

> [!NOTE]
> `StringBuilder` được tối ưu hóa cho **thao tác chuỗi** (thêm, chèn, xóa, thay thế), không phải **tìm kiếm chuỗi**. Do đó, nó không cung cấp các phương thức như `IndexOf()`, `LastIndexOf()`, `Contains()`, `StartsWith()`, v.v. Nếu bạn cần các chức năng tìm kiếm hoặc các phương thức thao tác chuỗi phức tạp khác, bạn cần chuyển đổi `StringBuilder` thành `string` bằng phương thức `ToString()` và sau đó sử dụng các phương thức của lớp `String`.

### 2.3 Các phương thức chính của StringBuilder

`StringBuilder` cung cấp các phương thức tương tự như `String` nhưng hoạt động trên chính đối tượng hiện tại, trả về `StringBuilder` để cho phép xâu chuỗi (chaining) các lệnh gọi:

*   `Append(value)`: Thêm giá trị vào cuối `StringBuilder`. Có nhiều overload cho phép thêm các kiểu dữ liệu khác nhau (string, char, int, bool, v.v.).
*   `AppendLine(value)`: Thêm giá trị vào cuối `StringBuilder` và sau đó thêm một ký tự xuống dòng (`\r\n`).
*   `Insert(int startIndex, value)`: Chèn giá trị vào vị trí `startIndex` trong `StringBuilder`.
*   `Remove(int startIndex, int length)`: Xóa một số `length` ký tự bắt đầu từ `startIndex`.
*   `Replace(string oldValue, string newValue)`: Thay thế tất cả các lần xuất hiện của `oldValue` bằng `newValue`.
*   `Clear()`: Xóa sạch nội dung của `StringBuilder`, đặt độ dài về 0.
*   `ToString()`: Chuyển đổi nội dung của `StringBuilder` thành một đối tượng `string` thông thường. Đây là bước cuối cùng khi bạn muốn sử dụng chuỗi đã xây dựng.

> [!TIP]
> Hầu hết các phương thức của `StringBuilder` (như `Append`, `Insert`, `Remove`, `Replace`) đều trả về chính đối tượng `StringBuilder` hiện tại. Điều này cho phép bạn **xâu chuỗi (chain)** các lệnh gọi phương thức lại với nhau, làm cho mã gọn gàng và dễ đọc hơn.

### 2.4 Ví dụ minh họa StringBuilder

Hãy xem cách `StringBuilder` hoạt động trong thực tế và cách chúng ta có thể xâu chuỗi các phương thức.

```csharp
using System;
using System.Text; // Cần thêm namespace này để sử dụng StringBuilder

public class Program
{
    public static void Main(string[] args)
    {
        // 1. Khởi tạo StringBuilder
        // Có thể truyền chuỗi ban đầu hoặc chỉ định dung lượng ban đầu
        var builder = new StringBuilder("Hello World");
        Console.WriteLine($"Khởi tạo: '{builder}' (Length: {builder.Length}, Capacity: {builder.Capacity})");

        // 2. Xâu chuỗi các thao tác (Method Chaining)
        builder
            .Append('-', 10)         // Nối 10 dấu gạch ngang
            .AppendLine()            // Thêm một dòng mới
            .Append("Header")        // Nối "Header"
            .AppendLine()            // Thêm một dòng mới
            .Append('-', 10)         // Nối 10 dấu gạch ngang
            .Replace('-', '+')       // Thay thế tất cả '-' bằng '+'
            .Remove(0, 10)           // Xóa 10 ký tự đầu tiên
            .Insert(0, new string('=', 10)); // Chèn 10 dấu '=' vào đầu

        Console.WriteLine("\n--- Sau khi xâu chuỗi các thao tác ---");
        Console.WriteLine(builder);
        Console.WriteLine($"(Length: {builder.Length}, Capacity: {builder.Capacity})");

        // 3. Truy cập ký tự riêng lẻ (giống như string, nhưng có thể sửa đổi)
        // Lưu ý: Việc sửa đổi ký tự riêng lẻ không cần tạo đối tượng mới
        builder[0] = 'X'; 
        Console.WriteLine($"\nKý tự đầu tiên sau khi sửa: {builder[0]}");
        Console.WriteLine($"Chuỗi sau khi sửa ký tự đầu: {builder}");

        // 4. Chuyển StringBuilder thành string khi hoàn tất
        string finalString = builder.ToString();
        Console.WriteLine($"\n--- Chuỗi cuối cùng (kiểu string) ---");
        Console.WriteLine(finalString);
        Console.WriteLine($"(Kiểu: {finalString.GetType().Name})");


        // Một ví dụ khác về xây dựng chuỗi phức tạp (ví dụ email)
        Console.WriteLine("\n--- Tạo Email bằng StringBuilder ---");
        StringBuilder emailBuilder = new StringBuilder();
        emailBuilder.Append("Kính gửi ");
        emailBuilder.Append("Khách hàng");
        emailBuilder.AppendLine(","); // Thêm dấu phẩy và xuống dòng
        emailBuilder.AppendLine("Chúng tôi xin thông báo về một cập nhật quan trọng.");
        emailBuilder.AppendLine("Vui lòng truy cập trang web của chúng tôi để biết thêm chi tiết.");
        emailBuilder.AppendLine("Trân trọng,");
        emailBuilder.Append("Đội ngũ Hỗ trợ.");

        Console.WriteLine(emailBuilder.ToString());
    }
}
```

## 3. Lập trình thủ tục (Procedural Programming)

Lập trình thủ tục là một trong những mô hình lập trình cơ bản và lâu đời nhất. Trong mô hình này, chương trình được cấu trúc xung quanh các lời gọi thủ tục (procedures), còn được gọi là hàm (functions) hoặc phương thức (methods). Cho đến nay, bạn có thể đã viết hầu hết mã của mình trực tiếp trong phương thức `Main()`. Tuy nhiên, khi ứng dụng phát triển, việc viết tất cả mã vào một phương thức duy nhất sẽ nhanh chóng trở nên khó quản lý, khó đọc và khó bảo trì.

### 3.1 Khái niệm và tầm quan trọng

**Lập trình thủ tục** tập trung vào việc chia nhỏ một chương trình lớn thành các khối mã nhỏ hơn, mỗi khối có một nhiệm vụ cụ thể. Các khối này được gọi là thủ tục hoặc phương thức. Dữ liệu thường được truyền giữa các thủ tục thông qua tham số.

**Tầm quan trọng của Lập trình thủ tục:**

*   **Tái sử dụng mã (Code Reusability):** Một thủ tục được định nghĩa một lần có thể được gọi nhiều lần từ các phần khác nhau của chương trình, tránh việc lặp lại mã (Don't Repeat Yourself - DRY).
*   **Dễ đọc và bảo trì (Readability and Maintainability):** Khi mã được chia thành các phần nhỏ, rõ ràng, mỗi phần tập trung vào một nhiệm vụ cụ thể, nó trở nên dễ hiểu hơn. Việc sửa lỗi hoặc thêm tính năng mới vào một thủ tục nhỏ sẽ dễ dàng hơn nhiều so với việc chỉnh sửa một khối mã lớn.
*   **Quản lý độ phức tạp (Complexity Management):** Lập trình thủ tục giúp xử lý các hệ thống lớn bằng cách chia chúng thành các thành phần nhỏ hơn, độc lập hơn. Thay vì phải hiểu toàn bộ hệ thống cùng một lúc, lập trình viên có thể tập trung vào từng thủ tục riêng lẻ.
*   **Tạo sự trừu tượng (Abstraction):** Các thủ tục cho phép bạn "trừu tượng hóa" các chi tiết thực thi phức tạp thành một tên gọi đơn giản, giúp người gọi không cần biết cách thủ tục đó hoạt động bên trong mà chỉ cần biết nó làm gì.

Trong khi lập trình hướng đối tượng (OOP) là một mô hình tiên tiến hơn dựa trên các đối tượng và tương tác giữa chúng, thì việc nắm vững lập trình thủ tục là nền tảng vững chắc để chuyển sang OOP. Các phương thức trong C# chính là hiện thân của các thủ tục này.

### 3.2 Quy tắc vàng: Tách biệt logic kinh doanh và I/O

Một nguyên tắc thiết yếu trong lập trình thủ tục (và cả OOP sau này) là **tách biệt logic kinh doanh (business logic) khỏi các thao tác nhập/xuất (Input/Output - I/O)**.

*   **Logic kinh doanh (Business Logic):** Là phần mã thực hiện các tính toán, xử lý dữ liệu, hoặc áp dụng các quy tắc nghiệp vụ cốt lõi của ứng dụng. Ví dụ: đảo ngược chuỗi, tìm số duy nhất, tóm tắt văn bản, tính toán thuế, xác thực dữ liệu. Logic này độc lập với cách dữ liệu được lấy vào hoặc hiển thị ra.
*   **Thao tác I/O (Input/Output):** Là phần mã tương tác với thế giới bên ngoài chương trình. Ví dụ: `Console.ReadLine()`, `Console.WriteLine()`, đọc/ghi file, gửi/nhận dữ liệu qua mạng, tương tác với cơ sở dữ liệu.

**Tại sao phải tách biệt?**

1.  **Tái sử dụng (Reusability):** Logic kinh doanh thường là trái tim của ứng dụng. Bằng cách đóng gói nó vào các phương thức riêng biệt, bạn có thể tái sử dụng logic đó trên nhiều giao diện người dùng khác nhau (ứng dụng console, ứng dụng web, ứng dụng di động, API) mà không cần viết lại. Chỉ phần I/O là cần thay đổi để phù hợp với giao diện cụ thể.
2.  **Kiểm thử (Testability):** Các phương thức chứa logic kinh doanh thuần túy (không có I/O) rất dễ dàng để viết kiểm thử đơn vị (unit tests). Bạn chỉ cần cung cấp đầu vào và kiểm tra đầu ra, không cần mô phỏng các tương tác phức tạp với console, file hay mạng.
3.  **Bảo trì (Maintainability):** Khi có thay đổi trong cách hiển thị dữ liệu (I/O) hoặc cách tính toán dữ liệu (logic kinh doanh), bạn chỉ cần sửa đổi một phần nhỏ của mã mà không ảnh hưởng đến phần còn lại.
4.  **Dễ hiểu (Readability):** Mã trở nên rõ ràng hơn khi các nhiệm vụ được phân tách. Phương thức `Main()` sẽ trở thành một "điều phối viên" gọi các phương thức khác, thay vì chứa toàn bộ logic.

### 3.3 Cấu trúc phương thức trong C#

Một phương thức trong C# thường có cấu trúc như sau:

```csharp
<Access Modifier> static <Return Type> <Method Name>(<Parameter Type> <Parameter Name>, ...)
{
    // Logic của phương thức
    // ...
    return <value>; // Nếu Return Type khác void
}
```

*   **`<Access Modifier>`**: Xác định phạm vi truy cập của phương thức. Các từ khóa phổ biến:
    *   `public`: Có thể được truy cập từ bất kỳ đâu.
    *   `private`: Chỉ có thể được truy cập trong cùng một lớp.
    *   `protected`, `internal`, `protected internal`, `private protected`: Sẽ được học chi tiết hơn trong các chương về OOP.
    Trong phần này, chúng ta sẽ chủ yếu sử dụng `public` để có thể gọi từ `Main()`.

*   **`static`**: Từ khóa này có nghĩa là phương thức thuộc về lớp (class) chứ không phải một thể hiện (instance) cụ thể của lớp. Bởi vì phương thức `Main()` là `static`, bất kỳ phương thức nào được gọi trực tiếp từ `Main()` cũng phải là `static`. Chúng ta sẽ tìm hiểu sâu hơn về `static` và non-`static` khi học về OOP.

*   **`<Return Type>`**: Kiểu dữ liệu mà phương thức sẽ trả về sau khi hoàn thành công việc của nó (ví dụ: `string`, `int`, `List<int>`).
    *   Nếu phương thức không trả về giá trị nào (chỉ thực hiện một hành động), kiểu trả về sẽ là `void`.
    *   Nếu kiểu trả về khác `void`, phương thức phải có câu lệnh `return <value>;` tương ứng.

*   **`<Method Name>`**: Tên của phương thức. Nên đặt tên có ý nghĩa, sử dụng PascalCase, và phản ánh chức năng của phương thức (ví dụ: `ReverseName`, `GetUniqueNumbers`, `CalculateTotalPrice`).

*   **`(<Parameter Type> <Parameter Name>, ...)`**: Danh sách các tham số. Đây là các giá trị đầu vào mà phương thức cần để thực hiện công việc của nó.
    *   Mỗi tham số có một kiểu dữ liệu và một tên.
    *   **Truyền tham số theo giá trị (Pass by Value):** Đối với các kiểu giá trị (như `int`, `bool`, `char`, `struct`), một bản sao của giá trị được truyền vào phương thức. Mọi thay đổi bên trong phương thức sẽ không ảnh hưởng đến biến gốc bên ngoài.
    *   **Truyền tham số theo tham chiếu (Pass by Reference):** Đối với các kiểu tham chiếu (như `string`, `List<int>`, các đối tượng), một bản sao của *tham chiếu* (địa chỉ bộ nhớ) được truyền vào phương thức. Điều này có nghĩa là phương thức có thể truy cập và thay đổi đối tượng mà tham chiếu đó trỏ tới. Tuy nhiên, nếu bạn gán một đối tượng mới cho tham số bên trong phương thức, biến gốc bên ngoài vẫn sẽ trỏ đến đối tượng cũ (trừ khi dùng từ khóa `ref` hoặc `out`, sẽ được học sau).

### 3.4 Ví dụ tái cấu trúc mã (Refactoring)

Hãy xem xét việc tái cấu trúc (refactoring) một số ví dụ trước đây bằng cách trích xuất logic vào các phương thức riêng biệt, tuân thủ nguyên tắc tách biệt logic và I/O.

#### 3.4.1 Ví dụ 1: Đảo ngược tên

Mã ban đầu (tất cả trong `Main()`, không có tách biệt):

```csharp
// public static void Main(string[] args)
// {
//     Console.Write("Nhập tên của bạn: ");
//     string name = Console.ReadLine(); // Ví dụ: "Mosh"
//
//     char[] array = new char[name.Length];
//     for (var i = 0; i < name.Length; i++)
//         array[i] = name[name.Length - 1 - i];
//     string reversed = new string(array);
//
//     Console.WriteLine(reversed); // "hsoM"
// }
```

Tái cấu trúc thành một phương thức `ReverseString`:

```csharp
using System;
// using System.Linq; // Không cần thiết cho phương thức ReverseString cơ bản này

public class Program
{
    public static void Main(string[] args)
    {
        // Phần I/O: Lấy đầu vào từ người dùng
        Console.Write("Nhập tên của bạn: ");
        string name = Console.ReadLine();

        // Phần Logic kinh doanh: Gọi phương thức để thực hiện logic đảo ngược
        string reversedName = ReverseString(name);

        // Phần I/O: Hiển thị kết quả
        Console.WriteLine($"Tên đảo ngược: {reversedName}");
    }

    /// <summary>
    /// Đảo ngược một chuỗi.
    /// </summary>
    /// <param name="inputString">Chuỗi cần đảo ngược.</param>
    /// <returns>Chuỗi đã đảo ngược.</returns>
    public static string ReverseString(string inputString)
    {
        // Logic kinh doanh: Đảm bảo tính toán đúng, không liên quan đến cách hiển thị
        if (string.IsNullOrEmpty(inputString))
            return "";

        char[] array = new char[inputString.Length];
        for (var i = 0; i < inputString.Length; i++)
        {
            // Lấy ký tự từ cuối chuỗi gốc và đặt vào đầu chuỗi mới
            array[i] = inputString[inputString.Length - 1 - i];
        }
        return new string(array);
        
        // Cách ngắn gọn hơn với LINQ (sẽ học sau):
        // return new string(inputString.Reverse().ToArray());
    }
}
```

Trong ví dụ này:
*   `Main()` chỉ chịu trách nhiệm lấy đầu vào và hiển thị kết quả (I/O).
*   `ReverseString()` chịu trách nhiệm hoàn toàn về logic đảo ngược chuỗi (logic kinh doanh). Phương thức này có thể được gọi từ bất kỳ đâu cần đảo ngược một chuỗi mà không cần biết chuỗi đó đến từ đâu hay sẽ được sử dụng như thế nào.

#### 3.4.2 Ví dụ 2: Lấy các số duy nhất từ một danh sách

Mã ban đầu (tất cả trong `Main()`, giả định có một `List<int> numbers`):

```csharp
// public static void Main(string[] args)
// {
//     var numbers = new List<int> { 1, 2, 2, 3, 4, 4, 5 };
//     var uniques = new List<int>();
//     foreach (var number in numbers)
//     {
//         if (!uniques.Contains(number))
//             uniques.Add(number);
//     }
//     foreach (var unique in uniques)
//         Console.WriteLine(unique);
// }
```

Tái cấu trúc thành một phương thức `GetUniqueNumbers`:

```csharp
using System;
using System.Collections.Generic;
using System.Linq; // Cần cho List<int>.Distinct().ToList() nếu sử dụng LINQ

public class Program
{
    public static void Main(string[] args)
    {
        // Phần I/O (hoặc giả định đầu vào): Tạo danh sách số
        var numbers = new List<int> { 1, 2, 2, 3, 4, 4, 5, 1, 6 };

        // Phần Logic kinh doanh: Gọi phương thức để lấy danh sách số duy nhất
        List<int> uniqueNumbers = GetUniqueNumbers(numbers);

        // Phần I/O: Hiển thị kết quả
        Console.WriteLine("--- Các số duy nhất ---");
        foreach (var number in uniqueNumbers)
        {
            Console.WriteLine(number);
        }
    }

    /// <summary>
    /// Trích xuất các số duy nhất từ một danh sách các số nguyên.
    /// </summary>
    /// <param name="numbers">Danh sách các số nguyên.</param>
    /// <returns>Danh sách các số nguyên duy nhất.</returns>
    public static List<int> GetUniqueNumbers(List<int> numbers)
    {
        // Logic kinh doanh: Xử lý danh sách để tìm các giá trị duy nhất
        if (numbers == null)
            return new List<int>(); // Trả về danh sách rỗng nếu đầu vào là null để tránh lỗi

        var uniques = new List<int>();
        foreach (var number in numbers)
        {
            if (!uniques.Contains(number)) // Kiểm tra xem số đã có trong danh sách duy nhất chưa
                uniques.Add(number);      // Nếu chưa, thêm vào
        }
        return uniques;
        
        // Cách ngắn gọn hơn với LINQ (sẽ học sau):
        // return numbers.Distinct().ToList();
    }
}
```
Trong ví dụ này, logic tìm kiếm số duy nhất được đóng gói trong phương thức `GetUniqueNumbers`. Điều này giúp nó có thể tái sử dụng ở bất cứ đâu bạn cần lọc các số duy nhất từ một danh sách, đồng thời làm cho `Main()` gọn gàng và dễ đọc hơn.

#### 3.4.3 Ví dụ 3: Tóm tắt văn bản (Di chuyển sang lớp tiện ích)

Ví dụ tóm tắt văn bản từ phần 1.4 đã minh họa việc tách logic vào một phương thức riêng (`SummarizeText`). Hơn nữa, chúng ta đã đặt phương thức này vào một lớp `StringUtilities` riêng biệt. Đây là một bước tiến quan trọng trong lập trình thủ tục và tổ chức mã:

*   **Tách lớp:** Thay vì để mọi thứ trong lớp `Program`, chúng ta tạo các lớp tiện ích (utility classes) để nhóm các phương thức có liên quan. Lớp `StringUtilities` là nơi hợp lý để chứa các phương thức xử lý chuỗi. Điều này giúp mã được tổ chức theo chức năng, dễ dàng tìm kiếm và sử dụng lại.
*   **Phạm vi truy cập `public`:** Để phương thức `SummarizeText` có thể được gọi từ `Main()` (hoặc bất kỳ lớp nào khác), nó cần được khai báo là `public`.
*   **Phương thức `static`:** Vì `SummarizeText` không cần truy cập bất kỳ dữ liệu cụ thể nào của một đối tượng `StringUtilities` (nó chỉ hoạt động trên các tham số được truyền vào), nó được khai báo là `static`. Điều này cho phép chúng ta gọi nó trực tiếp bằng tên lớp (`StringUtilities.SummarizeText(...)`) mà không cần tạo một đối tượng `StringUtilities`.

Việc này minh họa cách các phương thức và lớp có thể được sử dụng để xây dựng một kiến trúc chương trình mô-đun, dễ quản lý và mở rộng hơn.

## Tóm tắt Phần 11

*   **Chuỗi (String) trong C# là một kiểu tham chiếu và bất biến:** Mọi thao tác "chỉnh sửa" chuỗi thực chất đều tạo ra một đối tượng chuỗi mới trên Heap và cập nhật tham chiếu của biến. Điều này có thể gây ra vấn đề về hiệu suất và bộ nhớ nếu thực hiện quá nhiều lần trong các vòng lặp hoặc thao tác chuỗi phức tạp.
*   **Các phương thức `String` hữu ích:** Bao gồm `Trim()`, `ToLower()`, `ToUpper()`, `IndexOf()`, `LastIndexOf()`, `Contains()`, `StartsWith()`, `EndsWith()`, `Substring()`, `Replace()`, `Split()`, `string.Join()`, `string.IsNullOrEmpty()`, `string.IsNullOrWhiteSpace()`.
*   **Chuyển đổi Chuỗi và Số:**
    *   Sử dụng `int.Parse()` hoặc `Convert.ToInt32()` để chuyển từ chuỗi sang số (cẩn thận với ngoại lệ `FormatException` hoặc `ArgumentNullException`).
    *   **Ưu tiên `int.TryParse()`** khi xử lý đầu vào không đáng tin cậy, vì nó không ném ngoại lệ và cho phép xử lý lỗi một cách duyên dáng.
    *   Sử dụng `ToString()` (với các định dạng như `C`, `D`, `F`, `X`, `P`) để chuyển từ số sang chuỗi và định dạng theo yêu cầu.
*   **`StringBuilder` cho các thao tác chuỗi thường xuyên:** `StringBuilder` là một lớp khả biến (mutable) được tối ưu hóa để thêm, chèn, xóa, thay thế ký tự mà không tạo đối tượng mới cho mỗi lần thay đổi. Nó nên được sử dụng khi cần thực hiện nhiều thao tác chỉnh sửa chuỗi liên tục để tránh lãng phí tài nguyên.
*   **`StringBuilder` không có các phương thức tìm kiếm:** Để tìm kiếm trong `StringBuilder`, bạn cần chuyển nó thành `string` bằng `ToString()`.
*   **Lập trình thủ tục:** Là mô hình lập trình dựa trên việc chia nhỏ chương trình thành các phương thức (hàm, thủ tục) nhỏ hơn, có nhiệm vụ cụ thể.
*   **Lợi ích của lập trình thủ tục:** Tăng khả năng tái sử dụng, dễ đọc, dễ bảo trì và quản lý độ phức tạp của mã. Nó là nền tảng quan trọng cho OOP.
*   **Quy tắc tách biệt logic và I/O:** Luôn cố gắng tách logic nghiệp vụ của ứng dụng khỏi các thao tác nhập/xuất để tăng tính linh hoạt, khả năng kiểm thử và khả năng tái sử dụng của mã.
*   **Cấu trúc phương thức:** Hiểu về `Access Modifier` (`public`, `private`), từ khóa `static`, kiểu trả về, tên phương thức và tham số khi tạo các phương thức riêng. Nắm vững cách các tham số được truyền (theo giá trị hoặc tham chiếu).

<!-- REVIEWED_BY_AGENT -->
