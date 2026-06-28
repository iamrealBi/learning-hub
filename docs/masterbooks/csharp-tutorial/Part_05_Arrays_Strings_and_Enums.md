# Phần 5: Mảng, Chuỗi và Kiểu liệt kê

Trong lập trình C#, việc tổ chức và quản lý dữ liệu hiệu quả là nền tảng để xây dựng các ứng dụng mạnh mẽ, linh hoạt và dễ bảo trì. Phần này sẽ đi sâu vào ba cấu trúc dữ liệu cơ bản nhưng cực kỳ quan trọng: Mảng (Arrays), Chuỗi (Strings) và Kiểu liệt kê (Enums). Chúng ta sẽ khám phá cách khai báo, khởi tạo, truy cập và thao tác với chúng, đồng thời tập trung vào cơ chế hoạt động "dưới vỏ bọc" (under the hood), đặc biệt là liên quan đến việc cấp phát bộ nhớ và sự khác biệt giữa kiểu giá trị (Value Type) và kiểu tham chiếu (Reference Type). Mục tiêu là giúp bạn không chỉ biết cách sử dụng mà còn hiểu rõ bản chất của những công cụ này để lưu trữ các tập hợp dữ liệu, làm việc với văn bản và định nghĩa các tập hợp hằng số một cách rõ ràng và có cấu trúc.

## I. Mảng (Arrays)

Mảng là một trong những cấu trúc dữ liệu cơ bản và được sử dụng rộng rãi nhất trong lập trình. Nó cho phép bạn lưu trữ một tập hợp các biến có cùng kiểu dữ liệu dưới một tên duy nhất, được sắp xếp liên tiếp trong bộ nhớ.

### 1. Mảng là gì?

Một mảng trong C# là một tập hợp có thứ tự các phần tử có cùng kiểu dữ liệu, được lưu trữ trong một khối bộ nhớ liên tục. Mỗi phần tử trong mảng có một vị trí duy nhất được xác định bởi một chỉ mục số nguyên.

*   **Chỉ mục (Index):** Trong C#, mảng được lập chỉ mục từ 0 (zero-indexed). Điều này có nghĩa là phần tử đầu tiên của mảng có chỉ mục `0`, phần tử thứ hai có chỉ mục `1`, và cứ thế tiếp tục cho đến phần tử cuối cùng có chỉ mục là `kích thước - 1`.
*   **Kích thước cố định:** Một đặc điểm quan trọng của mảng trong C# là kích thước của chúng được xác định tại thời điểm khởi tạo và không thể thay đổi sau đó. Nếu bạn cần một tập hợp dữ liệu có kích thước thay đổi linh hoạt, các cấu trúc dữ liệu khác như `List<T>` sẽ phù hợp hơn (sẽ được giới thiệu trong các phần sau).

### 2. Khai báo và Khởi tạo Mảng

Để sử dụng một mảng, bạn cần khai báo nó và cấp phát bộ nhớ cho các phần tử của nó.

#### a. Khai báo một biến mảng

Khai báo một biến mảng chỉ định rằng biến đó sẽ *tham chiếu* đến một mảng thuộc một kiểu dữ liệu cụ thể.

```csharp
KieuDuLieu[] tenMang;
```

Ví dụ:

```csharp
int[] numbers;        // Khai báo một biến sẽ tham chiếu đến mảng các số nguyên
string[] names;       // Khai báo một biến sẽ tham chiếu đến mảng các chuỗi
bool[] flags;         // Khai báo một biến sẽ tham chiếu đến mảng các giá trị boolean
```

Lưu ý rằng tại thời điểm này, biến `tenMang` chỉ là một tham chiếu `null` (chưa trỏ đến bất kỳ mảng nào trong bộ nhớ).

#### b. Khởi tạo (cấp phát bộ nhớ) cho mảng

Sau khi khai báo, bạn phải cấp phát bộ nhớ cho mảng bằng toán tử `new`. Đây là lúc bạn chỉ định kích thước cố định của mảng.

```csharp
tenMang = new KieuDuLieu[kichThuoc];
```

Ví dụ:

```csharp
int[] numbers = new int[3]; // Khai báo và khởi tạo một mảng số nguyên có 3 phần tử.
                            // Bộ nhớ cho 3 số nguyên được cấp phát trên Heap.
```

> [!TIP]
> Bạn có thể sử dụng từ khóa `var` để trình biên dịch tự động suy luận kiểu dữ liệu của mảng, giúp mã nguồn ngắn gọn hơn:
> ```csharp
> var numbers = new int[3]; // Tương đương với int[] numbers = new int[3];
> ```

#### c. Khởi tạo mảng với các giá trị ban đầu

Nếu bạn biết trước các giá trị muốn lưu trữ trong mảng, bạn có thể khởi tạo chúng ngay trong quá trình khai báo bằng cú pháp khởi tạo đối tượng (object initializer):

```csharp
KieuDuLieu[] tenMang = new KieuDuLieu[] { giaTri1, giaTri2, ..., giaTriN };
```

Hoặc viết gọn hơn (trình biên dịch tự động suy luận kiểu và kích thước):

```csharp
KieuDuLieu[] tenMang = { giaTri1, giaTri2, ..., giaTriN };
```

Ví dụ:

```csharp
string[] names = new string[] { "Jack", "John", "Mary" };
// Hoặc
string[] names = { "Jack", "John", "Mary" }; // Mảng này có kích thước 3
```

### 3. Cơ chế cấp phát bộ nhớ: Mảng là Kiểu Tham chiếu (Reference Type)

Đây là một điểm cực kỳ quan trọng cần nắm vững. Trong C#, mảng không phải là một kiểu giá trị (value type) mà là một **kiểu tham chiếu (reference type)**.

*   **Biến mảng trên Stack, dữ liệu mảng trên Heap:**
    *   Khi bạn khai báo `int[] numbers;`, biến `numbers` được tạo trên vùng nhớ **Stack**. Tuy nhiên, nó không lưu trữ trực tiếp các số nguyên. Thay vào đó, nó sẽ lưu trữ một *địa chỉ bộ nhớ* (một tham chiếu) đến nơi dữ liệu mảng thực sự được lưu trữ.
    *   Khi bạn gọi `numbers = new int[3];`, toán tử `new` sẽ cấp phát một khối bộ nhớ liên tục trên vùng nhớ **Heap** đủ lớn để chứa 3 phần tử `int`. Sau đó, địa chỉ của khối bộ nhớ này sẽ được gán cho biến `numbers` trên Stack.
    *   Mỗi phần tử trong mảng trên Heap sẽ được tự động gán giá trị mặc định của kiểu dữ liệu tương ứng (ví dụ: `0` cho `int`, `false` cho `bool`, `null` cho kiểu tham chiếu như `string`).

    ```
    // Minh họa cấp phát bộ nhớ cho int[] numbers = new int[3];
    //
    // Stack:
    // +-----------------+
    // | numbers (ref)   | ----> (Địa chỉ trên Heap)
    // +-----------------+
    //
    // Heap:
    // +-----------------+
    // | numbers[0] = 0  |
    // | numbers[1] = 0  |
    // | numbers[2] = 0  |
    // +-----------------+
    ```

*   **`System.Array`:** Đằng sau hậu trường, mỗi mảng trong C# là một thể hiện của lớp `System.Array`. Điều này có nghĩa là mảng không chỉ là một khối dữ liệu mà còn là một đối tượng, cho phép nó có các thuộc tính (như `Length` để lấy kích thước) và các phương thức (như `Sort`, `Clear`, v.v.).

*   **Ý nghĩa của Kiểu Tham chiếu:**
    *   **Sao chép tham chiếu:** Khi bạn gán một mảng cho một biến mảng khác (ví dụ: `int[] otherNumbers = numbers;`), bạn đang sao chép *tham chiếu* (địa chỉ bộ nhớ), không phải toàn bộ dữ liệu của mảng. Cả hai biến `numbers` và `otherNumbers` sẽ trỏ đến cùng một mảng trên Heap. Mọi thay đổi thông qua một biến sẽ ảnh hưởng đến mảng mà biến kia cũng đang tham chiếu.
    *   **Garbage Collection (GC):** Bộ nhớ của mảng trên Heap sẽ được tự động thu hồi bởi Garbage Collector khi không còn bất kỳ tham chiếu nào trỏ tới nó.

### 4. Truy cập và Thao tác với phần tử Mảng

Các phần tử trong mảng được truy cập bằng cách sử dụng chỉ mục (index) đặt trong dấu ngoặc vuông `[]`.

```csharp
int[] numbers = new int[3];
numbers[0] = 1; // Gán giá trị 1 cho phần tử đầu tiên (chỉ mục 0)
numbers[1] = 5; // Gán giá trị 5 cho phần tử thứ hai (chỉ mục 1)
// numbers[2] vẫn là giá trị mặc định (0)

Console.WriteLine(numbers[0]); // In ra giá trị của phần tử đầu tiên (1)
Console.WriteLine(numbers[1]); // In ra giá trị của phần tử thứ hai (5)
Console.WriteLine(numbers[2]); // In ra giá trị của phần tử thứ ba (0)
```

*   **Thuộc tính `Length`:** Để lấy kích thước của mảng, bạn sử dụng thuộc tính `Length`.
    ```csharp
    string[] names = { "Alice", "Bob" };
    Console.WriteLine($"Kích thước mảng: {names.Length}"); // Output: 2
    ```
*   **Lỗi `IndexOutOfRangeException`:** Cố gắng truy cập một phần tử bằng chỉ mục nằm ngoài phạm vi hợp lệ (`0` đến `Length - 1`) sẽ gây ra lỗi thời gian chạy `IndexOutOfRangeException`.

### 5. Ví dụ minh họa về Mảng

```csharp
using System;

class ArrayExample
{
    static void Main(string[] args)
    {
        Console.WriteLine("--- Ví dụ về Mảng trong C# ---");

        // 1. Khai báo và khởi tạo một mảng số nguyên với kích thước cố định (5 phần tử)
        // Các phần tử sẽ được khởi tạo mặc định là 0 (vì int là kiểu số nguyên)
        int[] scores = new int[5];
        Console.WriteLine($"\nKích thước mảng 'scores': {scores.Length}");
        Console.WriteLine($"Giá trị mặc định của scores[0]: {scores[0]}"); // Output: 0

        // Gán giá trị cho các phần tử
        scores[0] = 85;
        scores[1] = 92;
        scores[2] = 78;
        // scores[3] và scores[4] vẫn giữ giá trị mặc định là 0

        Console.WriteLine($"Giá trị scores[0] sau khi gán: {scores[0]}"); // Output: 85
        Console.WriteLine($"Giá trị scores[3] (mặc định): {scores[3]}");   // Output: 0

        // 2. Khai báo và khởi tạo một mảng boolean
        // Các phần tử sẽ được khởi tạo mặc định là false
        bool[] isActive = new bool[2];
        Console.WriteLine($"\nGiá trị mặc định của isActive[0]: {isActive[0]}"); // Output: False
        isActive[0] = true;
        Console.WriteLine($"Giá trị isActive[0] sau khi gán: {isActive[0]}"); // Output: True

        // 3. Khai báo và khởi tạo một mảng chuỗi sử dụng cú pháp khởi tạo đối tượng
        string[] students = { "Alice", "Bob", "Charlie" };
        Console.WriteLine($"\nKích thước mảng 'students': {students.Length}");
        Console.WriteLine($"Học sinh đầu tiên: {students[0]}"); // Output: Alice

        // 4. Minh họa kiểu tham chiếu của mảng
        int[] originalArray = { 10, 20, 30 };
        int[] copiedArray = originalArray; // Sao chép tham chiếu, không phải dữ liệu

        Console.WriteLine("\n--- Minh họa Kiểu Tham chiếu ---");
        Console.WriteLine($"originalArray[0] ban đầu: {originalArray[0]}"); // Output: 10
        Console.WriteLine($"copiedArray[0] ban đầu: {copiedArray[0]}");     // Output: 10

        copiedArray[0] = 100; // Thay đổi qua 'copiedArray'
        Console.WriteLine($"originalArray[0] sau khi thay đổi qua copiedArray: {originalArray[0]}"); // Output: 100 (thay đổi!)
        Console.WriteLine($"copiedArray[0] sau khi thay đổi: {copiedArray[0]}");     // Output: 100

        // 5. Vòng lặp để duyệt qua mảng
        Console.WriteLine("\n--- Duyệt qua mảng 'students' ---");
        for (int i = 0; i < students.Length; i++)
        {
            Console.WriteLine($"students[{i}]: {students[i]}");
        }

        Console.WriteLine("\n--- Duyệt qua mảng 'scores' bằng foreach ---");
        foreach (int score in scores)
        {
            Console.WriteLine($"Điểm số: {score}");
        }

        // 6. Thử truy cập ngoài chỉ mục (sẽ gây lỗi IndexOutOfRangeException nếu không comment)
        // Console.WriteLine(students[3]); // Lỗi! Chỉ mục hợp lệ là 0, 1, 2
    }
}
```

### 6. Antigravity IDE và Vibe Coding cho Mảng

Với Antigravity IDE, việc làm việc với mảng có thể trở nên hiệu quả và trực quan hơn thông qua tư duy Vibe Coding. Thay vì phải nhớ chính xác cú pháp hoặc các phương thức cụ thể, bạn có thể mô tả *ý định* của mình.

*   **Khởi tạo mảng nhanh chóng:**
    *   **Vibe:** "Tạo một mảng số nguyên tên `temperatures` có 7 phần tử, khởi tạo tất cả bằng 0."
    *   **Antigravity:** Sẽ tự động tạo `int[] temperatures = new int[7];`
    *   **Vibe:** "Tạo một mảng chuỗi tên `weekdays` với các giá trị 'Thứ Hai', 'Thứ Ba', ..., 'Chủ Nhật'."
    *   **Antigravity:** Sẽ tạo `string[] weekdays = { "Thứ Hai", "Thứ Ba", "Thứ Tư", "Thứ Năm", "Thứ Sáu", "Thứ Bảy", "Chủ Nhật" };`
*   **Thao tác với mảng:**
    *   **Vibe:** "Duyệt qua mảng `productPrices` và in từng giá trị."
    *   **Antigravity:** Sẽ tạo một vòng lặp `foreach` hoặc `for` phù hợp.
    *   **Vibe:** "Tìm giá trị lớn nhất trong mảng `sensorReadings`."
    *   **Antigravity:** Có thể đề xuất hoặc tự động tạo mã dùng `sensorReadings.Max()` (từ LINQ, nếu phù hợp) hoặc một vòng lặp thủ công.
*   **Trực quan hóa và gỡ lỗi:** Antigravity có thể cung cấp giao diện để trực quan hóa nội dung của mảng trong quá trình gỡ lỗi, giúp bạn dễ dàng kiểm tra các phần tử và hiểu cách dữ liệu thay đổi.
*   **Xử lý lỗi:** Khi bạn cố gắng truy cập chỉ mục không hợp lệ, Antigravity có thể cảnh báo sớm hoặc đề xuất các biện pháp xử lý ngoại lệ (như kiểm tra `i < array.Length`) để tránh `IndexOutOfRangeException`.
*   **Refactoring:** Nếu bạn có một đoạn mã sử dụng mảng không hiệu quả, Antigravity có thể đề xuất cách refactor, ví dụ như sử dụng `Array.Copy()` thay vì vòng lặp thủ công để sao chép mảng, hoặc gợi ý chuyển sang `List<T>` nếu phát hiện mảng cần thay đổi kích thước thường xuyên.

Tư duy Vibe Coding giúp bạn tập trung vào "cái gì" bạn muốn đạt được, thay vì "làm thế nào" để viết cú pháp chính xác, để Antigravity xử lý chi tiết kỹ thuật.

---

## II. Chuỗi (Strings)

Chuỗi là một trong những kiểu dữ liệu được sử dụng phổ biến nhất trong lập trình, dùng để biểu diễn và thao tác với văn bản. Trong C#, chuỗi có những đặc điểm độc đáo và quan trọng cần được hiểu rõ.

### 1. Chuỗi là gì?

Trong C#, một chuỗi là một chuỗi các ký tự Unicode. Chuỗi được khai báo bằng từ khóa `string` và được bao quanh bởi dấu ngoặc kép (`"`).

*   **`string` là `System.String`:** Từ khóa `string` trong C# là một bí danh (alias) cho lớp `System.String` trong .NET Framework. Điều này có nghĩa là `string` và `System.String` là hoàn toàn giống nhau. Vì `System.String` là một lớp, chuỗi là một **kiểu tham chiếu (reference type)**, giống như mảng.

### 2. Cơ chế cấp phát bộ nhớ và Tính bất biến của Chuỗi (String Immutability)

Tính bất biến (immutability) là khái niệm quan trọng nhất khi làm việc với chuỗi trong C#.

*   **Kiểu Tham chiếu trên Heap:** Giống như mảng, các đối tượng `string` được cấp phát trên vùng nhớ **Heap**. Biến `string` trên Stack sẽ lưu trữ tham chiếu đến đối tượng chuỗi trên Heap.

*   **Tính bất biến (Immutable):** Một khi một đối tượng `string` đã được tạo, bạn **không thể thay đổi nội dung của nó**. Bất kỳ thao tác nào dường như "sửa đổi" chuỗi (như nối chuỗi, thay thế ký tự, cắt chuỗi) thực chất đều tạo ra một đối tượng `string` *mới* trong bộ nhớ và trả về tham chiếu đến đối tượng mới đó. Chuỗi ban đầu vẫn tồn tại trên Heap (cho đến khi Garbage Collector thu hồi nếu không còn tham chiếu nào đến nó).

    ```csharp
    string message = "Hello"; // 1. Tạo đối tượng chuỗi "Hello" trên Heap.
                              //    Biến 'message' trỏ đến "Hello".

    message = message + " World"; // 2. KHÔNG thay đổi "Hello".
                                  //    Tạo đối tượng chuỗi " World" trên Heap.
                                  //    Tạo đối tượng chuỗi "Hello World" MỚI trên Heap
                                  //    bằng cách kết hợp "Hello" và " World".
                                  // 3. Biến 'message' bây giờ trỏ đến đối tượng "Hello World" MỚI.
                                  //    Đối tượng "Hello" ban đầu vẫn tồn tại nhưng không còn được
                                  //    tham chiếu bởi biến 'message' (sẽ bị GC thu hồi sau này).

    // Minh họa trên bộ nhớ:
    // Stack:
    // +-----------------+
    // | message (ref)   | ----> (Địa chỉ của "Hello World" trên Heap)
    // +-----------------+
    //
    // Heap:
    // +-----------------+
    // | "Hello"         | (Không còn được tham chiếu, chờ GC)
    // +-----------------+
    // | " World"        | (Có thể là một literal, hoặc chờ GC)
    // +-----------------+
    // | "Hello World"   | (Được tham chiếu bởi 'message')
    // +-----------------+
    ```

*   **Tại sao chuỗi bất biến?**
    *   **Hiệu suất và tối ưu hóa:** C# sử dụng "string interning" để lưu trữ các chuỗi literal giống nhau chỉ một lần trong bộ nhớ, tiết kiệm không gian. Nếu chuỗi có thể thay đổi, điều này sẽ không thể thực hiện được.
    *   **An toàn luồng (Thread Safety):** Chuỗi bất biến an toàn khi được truy cập bởi nhiều luồng cùng lúc mà không cần cơ chế khóa.
    *   **An toàn dữ liệu:** Chuỗi thường được dùng làm khóa trong các bộ sưu tập (như `Dictionary`). Nếu chuỗi có thể thay đổi, mã băm (hash code) của nó cũng thay đổi, làm mất hiệu lực của khóa.

*   **Khi nào cần thay đổi chuỗi hiệu quả?** Nếu bạn cần thực hiện nhiều thao tác sửa đổi chuỗi liên tiếp (nối, chèn, xóa), việc tạo ra quá nhiều đối tượng chuỗi mới có thể gây lãng phí tài nguyên và ảnh hưởng hiệu suất. Trong trường hợp này, bạn nên sử dụng lớp `System.Text.StringBuilder` (sẽ được giới thiệu trong các phần nâng cao hơn) để xây dựng chuỗi hiệu quả.

### 3. Các cách tạo và thao tác với Chuỗi

#### a. Sử dụng chuỗi ký tự (String Literals)

Cách phổ biến nhất là gán trực tiếp một chuỗi ký tự cho một biến:

```csharp
string message = "Hello, C#!";
string firstName = "John";
string lastName = "Doe";
```

#### b. Nối chuỗi (String Concatenation)

Bạn có thể kết hợp nhiều chuỗi lại với nhau bằng toán tử cộng (`+`):

```csharp
string fullName = firstName + " " + lastName; // "John Doe"
```

#### c. Định dạng chuỗi (String Formatting)

*   **`string.Format()`:** Phương thức tĩnh `string.Format()` cho phép bạn tạo chuỗi từ một mẫu định dạng và các đối số.
    ```csharp
    string formattedName = string.Format("Tên đầy đủ: {0} {1}", firstName, lastName);
    // Output: "Tên đầy đủ: John Doe"
    ```
*   **String Interpolation (`$` - Hiện đại và được khuyến nghị):** Từ C# 6.0 trở đi, string interpolation là cách được khuyến nghị để định dạng chuỗi, giúp mã nguồn ngắn gọn và dễ đọc hơn. Bạn đặt tiền tố `$` trước chuỗi và có thể nhúng các biểu thức C# trực tiếp vào bên trong dấu ngoặc nhọn `{}`.
    ```csharp
    string age = "30";
    string interpolatedGreeting = $"Chào mừng, {firstName} {lastName}! Bạn {age} tuổi.";
    // Output: "Chào mừng, John Doe! Bạn 30 tuổi."
    ```

#### d. Nối các phần tử mảng (String.Join)

Phương thức tĩnh `string.Join()` cho phép bạn kết hợp tất cả các phần tử của một mảng (hoặc bất kỳ `IEnumerable<T>` nào) thành một chuỗi duy nhất, sử dụng một dấu phân cách đã chỉ định.

```csharp
string[] names = { "Jack", "John", "Mary" };
string allNames = string.Join(", ", names); // "Jack, John, Mary"

int[] numbers = { 1, 2, 3 };
string numbersJoined = string.Join("-", numbers); // "1-2-3"
```

#### e. Ký tự thoát (Escape Characters)

Trong C#, một số ký tự có ý nghĩa đặc biệt và cần được "thoát" bằng dấu gạch chéo ngược (`\`) để được hiểu là ký tự thông thường hoặc để biểu diễn các ký tự không in được.

| Ký tự thoát | Ý nghĩa          |
| :---------- | :--------------- |
| `\n`        | Dòng mới (Newline) |
| `\t`        | Tab ngang        |
| `\\`        | Dấu gạch chéo ngược (`\`) |
| `\"`        | Dấu ngoặc kép (`"`) |
| `\'`        | Dấu nháy đơn (`'`) |

Ví dụ:

```csharp
string path = "C:\\Program Files\\My App";   // Để có dấu \ cần dùng \\
string quote = "Anh ấy nói: \"Xin chào!\""; // Để có dấu " cần dùng \"
string multiLine = "Dòng 1\nDòng 2";        // Xuống dòng
```

#### f. Chuỗi nguyên văn (Verbatim Strings)

Chuỗi nguyên văn được khai báo bằng cách thêm tiền tố `@` trước dấu ngoặc kép. Chúng rất hữu ích khi bạn muốn chuỗi được hiểu theo đúng nghĩa đen, không xử lý các ký tự thoát.

*   Các ký tự gạch chéo ngược (`\`) được hiểu là ký tự thông thường.
*   Chuỗi có thể kéo dài qua nhiều dòng.
*   Để bao gồm dấu ngoặc kép trong chuỗi nguyên văn, bạn phải lặp lại dấu ngoặc kép hai lần (`""`).

Ví dụ:

```csharp
string verbatimPath = @"C:\Program Files\My App"; // Không cần \\
string multiLineText = @"Đây là dòng đầu tiên.
Đây là dòng thứ hai."; // Chuỗi kéo dài qua nhiều dòng
string quoteInVerbatim = @"Anh ấy nói: ""Xin chào!"""; // Để có " cần dùng ""
```

### 4. Các phương thức chuỗi hữu ích

Lớp `string` (hay `System.String`) cung cấp rất nhiều phương thức tiện ích để thao tác với chuỗi. Một số phương thức phổ biến bao gồm:

*   `Length`: Thuộc tính trả về độ dài của chuỗi.
*   `Substring(startIndex, length)`: Trả về một chuỗi con từ vị trí bắt đầu với độ dài nhất định.
*   `Replace(oldChar, newChar)`: Thay thế tất cả các lần xuất hiện của một ký tự/chuỗi bằng ký tự/chuỗi khác.
*   `Contains(value)`: Kiểm tra xem chuỗi có chứa một chuỗi con cụ thể hay không.
*   `IndexOf(value)`: Trả về chỉ mục của lần xuất hiện đầu tiên của một ký tự/chuỗi con.
*   `ToUpper()`, `ToLower()`: Chuyển đổi chuỗi sang chữ hoa hoặc chữ thường.
*   `Trim()`, `TrimStart()`, `TrimEnd()`: Xóa khoảng trắng ở đầu và/hoặc cuối chuỗi.
*   `Split(separator)`: Chia chuỗi thành một mảng các chuỗi con dựa trên một dấu phân cách.

### 5. Ví dụ minh họa về Chuỗi

```csharp
using System;

class StringExample
{
    static void Main(string[] args)
    {
        Console.WriteLine("--- Ví dụ về Chuỗi trong C# ---");

        // 1. Tạo chuỗi bằng literal và String Interpolation
        string firstName = "Alice";
        string lastName = "Smith";
        int age = 25;
        string greeting = $"Chào mừng, {firstName} {lastName}! Bạn {age} tuổi.";
        Console.WriteLine(greeting);

        // 2. Nối chuỗi (Concatenation)
        string profession = "Developer";
        string info = firstName + " là một " + profession + ".";
        Console.WriteLine(info);

        // 3. Nối các phần tử mảng (String.Join)
        string[] tags = { "C#", "Programming", "Tutorial" };
        string joinedTags = string.Join(", ", tags);
        Console.WriteLine($"Các thẻ liên quan: {joinedTags}");

        // 4. Ký tự thoát (Escape Characters)
        string filePath = "Tài liệu: C:\\Projects\\MyCode\\program.cs";
        Console.WriteLine(filePath);
        string dialog = "Cô ấy hỏi: \"Bạn có muốn học C# không?\"\nAnh ấy trả lời: \"Tất nhiên!\"";
        Console.WriteLine(dialog);

        // 5. Chuỗi nguyên văn (Verbatim Strings)
        string verbatimFilePath = @"Tài liệu: C:\Projects\MyCode\program.cs";
        Console.WriteLine(verbatimFilePath);
        string longDescription = @"Đây là một đoạn mô tả dài
bao gồm nhiều dòng
và không cần ký tự thoát cho dấu gạch chéo ngược.
Để có dấu "" trong chuỗi nguyên văn, ta dùng hai dấu "" liên tiếp.";
        Console.WriteLine(longDescription);

        // 6. Minh họa tính bất biến (Immutability)
        string originalString = "C# Programming";
        Console.WriteLine($"\nChuỗi gốc: {originalString}"); // Output: C# Programming

        string modifiedString = originalString.Replace("C#", "DotNet"); // Tạo chuỗi mới
        Console.WriteLine($"Chuỗi sau khi Replace: {modifiedString}"); // Output: DotNet Programming
        Console.WriteLine($"Chuỗi gốc vẫn là: {originalString}");     // Output: C# Programming (không thay đổi)

        // 7. Các phương thức chuỗi hữu ích
        Console.WriteLine($"\nĐộ dài của chuỗi '{originalString}': {originalString.Length}"); // Output: 14
        Console.WriteLine($"Chuỗi con (từ index 3, độ dài 5): {originalString.Substring(3, 5)}"); // Output: Progr
        Console.WriteLine($"Chứa 'Program'?: {originalString.Contains("Program")}"); // Output: True
        Console.WriteLine($"Chữ hoa: {originalString.ToUpper()}"); // Output: C# PROGRAMMING
        Console.WriteLine($"Chuỗi đã cắt khoảng trắng: {"  Hello World  ".Trim()}"); // Output: Hello World
    }
}
```

### 6. Antigravity IDE và Vibe Coding cho Chuỗi

Antigravity IDE có thể là một trợ thủ đắc lực trong việc xử lý chuỗi, đặc biệt khi bạn cần thực hiện các thao tác phức tạp hoặc muốn đảm bảo tính đúng đắn của định dạng.

*   **Tạo chuỗi thông minh:**
    *   **Vibe:** "Tạo một thông báo chào mừng bao gồm tên người dùng `userName`, tuổi `userAge` và thành phố `userCity`."
    *   **Antigravity:** Sẽ tự động tạo mã sử dụng string interpolation: `string welcomeMessage = $"Chào mừng, {userName}! Bạn {userAge} tuổi và sống ở {userCity}.";`
*   **Chuyển đổi định dạng:**
    *   **Vibe:** "Cho một đường dẫn file `C:\Users\Documents\report.pdf`, biến nó thành chuỗi nguyên văn."
    *   **Antigravity:** `string verbatimPath = @"C:\Users\Documents\report.pdf";`
    *   **Vibe:** "Chuyển đổi chuỗi `product_name` thành `ProductName` (PascalCase)."
    *   **Antigravity:** Có thể đề xuất và tạo mã sử dụng các phương thức `Split`, `ToUpper`, `Join` để đạt được kết quả mong muốn.
*   **Trích xuất và phân tích chuỗi:**
    *   **Vibe:** "Từ chuỗi `Log: User 'Alice' logged in at 2023-10-27 10:30:00`, trích xuất tên người dùng và thời gian đăng nhập."
    *   **Antigravity:** Có thể gợi ý sử dụng `Substring`, `IndexOf`, `Split` hoặc thậm chí `Regex` (nếu bạn đã học) để phân tích chuỗi.
*   **Tối ưu hóa hiệu suất:** Khi bạn có nhiều thao tác nối chuỗi trong một vòng lặp, Antigravity có thể cảnh báo về vấn đề hiệu suất do tính bất biến của chuỗi và đề xuất sử dụng `StringBuilder`.
*   **Kiểm tra và xác thực:** Antigravity có thể giúp tạo các đoạn mã kiểm tra chuỗi (ví dụ: `string.IsNullOrEmpty`, `string.IsNullOrWhiteSpace`) hoặc xác thực định dạng chuỗi bằng biểu thức chính quy (regular expressions) nếu bạn mô tả ý định của mình.

Với Vibe Coding, bạn có thể diễn đạt yêu cầu về thao tác chuỗi bằng ngôn ngữ tự nhiên, và Antigravity sẽ chuyển đổi ý định đó thành mã C# hiệu quả, giúp bạn tiết kiệm thời gian và tránh lỗi cú pháp.

---

## III. Kiểu liệt kê (Enums)

Kiểu liệt kê, hay `enum`, là một kiểu giá trị đặc biệt cho phép bạn định nghĩa một tập hợp các hằng số được đặt tên. Chúng giúp cải thiện khả năng đọc, bảo trì và tính an toàn của mã nguồn bằng cách thay thế các "số ma thuật" (magic numbers) bằng các tên có ý nghĩa rõ ràng.

### 1. Enum là gì?

Một `enum` (viết tắt của enumeration) là một kiểu dữ liệu đại diện cho một tập hợp các giá trị hằng số có liên quan chặt chẽ với nhau. Thay vì sử dụng các số nguyên trực tiếp (ví dụ: `1` cho "Standard", `2` cho "Express"), bạn có thể định nghĩa một `enum` với các tên rõ ràng hơn.

Ví dụ, thay vì dùng `int shippingMethod = 1;`, bạn có thể dùng `ShippingMethod shippingMethod = ShippingMethod.Standard;`. Điều này làm cho mã nguồn dễ hiểu hơn rất nhiều.

> [!TIP]
> Sử dụng `enum` khi bạn có một tập hợp các hằng số hữu hạn và cố định, có ý nghĩa ngữ cảnh trong ứng dụng của bạn (ví dụ: trạng thái đơn hàng, ngày trong tuần, cấp độ ưu tiên).

### 2. Khai báo Enum

Enum được khai báo bằng từ khóa `enum`. Vì enum là một kiểu dữ liệu mới, nó thường được định nghĩa ở cấp độ không gian tên (namespace) hoặc bên trong một lớp, chứ không phải bên trong một phương thức.

```csharp
enum TenEnum
{
    ThanhVien1,
    ThanhVien2,
    // ...
}
```

Ví dụ:

```csharp
namespace MyApp
{
    enum ShippingMethod
    {
        Standard,
        Express,
        Overnight
    }

    class OrderProcessor
    {
        // ...
    }
}
```

### 3. Giá trị và Kiểu cơ sở của thành viên Enum

*   **Giá trị mặc định:** Mặc định, thành viên đầu tiên của enum được gán giá trị `0`, và các thành viên tiếp theo sẽ tự động tăng lên `1`.
    *   `Standard` = 0
    *   `Express` = 1
    *   `Overnight` = 2
*   **Gán giá trị tường minh:** Bạn có thể gán giá trị số nguyên cụ thể cho từng thành viên. Điều này rất hữu ích khi bạn muốn các giá trị enum tương ứng với các ID trong cơ sở dữ liệu hoặc các giá trị từ một hệ thống bên ngoài. Nếu bạn chỉ định giá trị cho một thành viên, các thành viên tiếp theo sẽ tự động tăng lên từ giá trị đó (trừ khi bạn gán tường minh cho chúng).

    ```csharp
    enum ShippingMethod
    {
        Standard = 1,  // Giá trị là 1
        Express = 2,   // Giá trị là 2
        Overnight = 3  // Giá trị là 3
    }
    ```

    > [!IMPORTANT]
    > **Thực hành tốt nhất:** Luôn đặt giá trị tường minh cho các thành viên enum. Điều này đảm bảo rằng các giá trị không bị thay đổi nếu bạn thêm hoặc xóa các thành viên trong tương lai, giúp duy trì tính nhất quán, đặc biệt khi enum được lưu trữ hoặc trao đổi với các hệ thống khác (ví dụ: cơ sở dữ liệu, API).

*   **Kiểu cơ sở (Underlying Type):** Mặc định, kiểu cơ sở của một enum là `int`. Tuy nhiên, bạn có thể chỉ định một kiểu số nguyên khác như `byte`, `sbyte`, `short`, `ushort`, `uint`, `long`, hoặc `ulong` để tối ưu hóa bộ nhớ hoặc phù hợp với yêu cầu dữ liệu.

    ```csharp
    enum ShippingMethod : byte // Kiểu cơ sở là byte
    {
        Standard = 1,
        Express = 2,
        Overnight = 3
    }
    ```

### 4. Cơ chế cấp phát bộ nhớ: Enum là Kiểu Giá trị (Value Type)

Không giống như mảng và chuỗi, `enum` là một **kiểu giá trị (value type)**.

*   **Lưu trữ trực tiếp giá trị:** Khi bạn khai báo một biến kiểu enum (ví dụ: `ShippingMethod myMethod;`), biến `myMethod` sẽ lưu trữ *trực tiếp giá trị số nguyên* của thành viên enum mà nó đại diện.
    *   Nếu `myMethod` là một biến cục bộ, giá trị của nó sẽ được lưu trữ trên **Stack**.
    *   Nếu `myMethod` là một trường (field) trong một đối tượng, giá trị của nó sẽ được lưu trữ trực tiếp bên trong đối tượng đó trên **Heap** (inline).

    ```csharp
    // Minh họa cấp phát bộ nhớ cho ShippingMethod myShippingMethod = ShippingMethod.Express;
    // (Giả sử Express = 2)
    //
    // Stack:
    // +---------------------+
    // | myShippingMethod = 2|
    // +---------------------+
    ```

*   **Ý nghĩa của Kiểu Giá trị:**
    *   **Sao chép giá trị:** Khi bạn gán một enum cho một biến enum khác (ví dụ: `ShippingMethod anotherMethod = myShippingMethod;`), bạn đang sao chép *giá trị* (số 2), không phải tham chiếu. Hai biến sẽ độc lập với nhau.
    *   **Không cần Garbage Collection:** Vì dữ liệu được lưu trữ trực tiếp, không có đối tượng riêng biệt trên Heap, nên Garbage Collector không cần quản lý bộ nhớ của các biến enum.

### 5. Sử dụng Enum

Bạn sử dụng enum giống như bất kỳ kiểu dữ liệu nào khác để khai báo biến và truy cập các thành viên của nó bằng ký hiệu dấu chấm.

```csharp
ShippingMethod myShippingMethod = ShippingMethod.Express;
Console.WriteLine(myShippingMethod); // Output: Express
```

Enum thường được sử dụng trong các câu lệnh điều kiện như `switch` để xử lý các trường hợp khác nhau một cách rõ ràng và an toàn kiểu.

```csharp
switch (myShippingMethod)
{
    case ShippingMethod.Standard:
        Console.WriteLine("Phương thức vận chuyển tiêu chuẩn.");
        break;
    case ShippingMethod.Express:
        Console.WriteLine("Phương thức vận chuyển nhanh.");
        break;
    case ShippingMethod.Overnight:
        Console.WriteLine("Phương thức vận chuyển qua đêm.");
        break;
    default:
        Console.WriteLine("Phương thức vận chuyển không xác định.");
        break;
}
```

### 6. Chuyển đổi giữa Enum, Số nguyên và Chuỗi

Bạn thường xuyên cần chuyển đổi giữa các giá trị enum và các kiểu dữ liệu khác.

#### a. Chuyển đổi Enum sang Số nguyên (Enum to Integer)

Bạn có thể ép kiểu tường minh (explicit cast) một giá trị enum sang kiểu số nguyên tương ứng.

```csharp
ShippingMethod method = ShippingMethod.Express; // Giả sử Express = 2
int methodId = (int)method;                     // methodId sẽ là 2
Console.WriteLine($"Giá trị số của Express: {methodId}");
```

#### b. Chuyển đổi Số nguyên sang Enum (Integer to Enum)

Ngược lại, bạn cũng có thể ép kiểu một số nguyên sang kiểu enum. Hãy cẩn thận vì nếu số nguyên không khớp với bất kỳ giá trị enum nào, nó vẫn sẽ được chuyển đổi thành một giá trị enum không hợp lệ mà không gây lỗi (cho đến khi bạn sử dụng giá trị đó).

```csharp
int methodIdFromDb = 3; // Giả sử nhận được từ cơ sở dữ liệu (Overnight = 3)
ShippingMethod method = (ShippingMethod)methodIdFromDb; // method sẽ là Overnight
Console.WriteLine($"Kiểu vận chuyển từ ID 3: {method}");

int invalidId = 99; // Không có thành viên nào có giá trị 99
ShippingMethod invalidMethod = (ShippingMethod)invalidId;
Console.WriteLine($"Kiểu vận chuyển từ ID 99: {invalidMethod}"); // Output: 99 (không phải tên thành viên)
```

#### c. Chuyển đổi Enum sang Chuỗi (Enum to String)

Mọi đối tượng trong C# đều có phương thức `ToString()`. Gọi phương thức này trên một giá trị enum sẽ trả về tên của thành viên enum dưới dạng chuỗi. `Console.WriteLine()` cũng tự động gọi `ToString()` khi in một enum.

```csharp
ShippingMethod method = ShippingMethod.Express;
string methodName = method.ToString(); // methodName sẽ là "Express"
Console.WriteLine($"Tên của kiểu vận chuyển: {methodName}");
```

#### d. Chuyển đổi Chuỗi sang Enum (String to Enum - Parsing)

Chuyển đổi một chuỗi thành một giá trị enum được gọi là **phân tích cú pháp (parsing)**. Lớp tĩnh `System.Enum` cung cấp các phương thức `Parse()` và `TryParse()` để thực hiện việc này.

*   **`Enum.Parse()`:** Phương thức này sẽ ném một ngoại lệ (`ArgumentException`) nếu chuỗi không khớp với bất kỳ tên thành viên enum nào.

    ```csharp
    // Cú pháp: (TenEnum)Enum.Parse(typeof(TenEnum), "TenThanhVien");
    string methodNameFromInput = "Overnight";
    try
    {
        ShippingMethod method = (ShippingMethod)Enum.Parse(typeof(ShippingMethod), methodNameFromInput);
        Console.WriteLine($"Phân tích chuỗi '{methodNameFromInput}' thành enum: {method}");
    }
    catch (ArgumentException)
    {
        Console.WriteLine($"Lỗi: '{methodNameFromInput}' không phải là một phương thức vận chuyển hợp lệ.");
    }
    ```

*   **`Enum.TryParse()` (Được khuyến nghị cho an toàn):** Phương thức này không ném ngoại lệ mà trả về `true` nếu phân tích cú pháp thành công và `false` nếu thất bại. Giá trị enum được trả về qua tham số `out`. Đây là cách an toàn hơn để chuyển đổi chuỗi sang enum khi bạn không chắc chắn về tính hợp lệ của chuỗi đầu vào.

    ```csharp
    string validMethodString = "Standard";
    ShippingMethod parsedMethod;
    if (Enum.TryParse(validMethodString, out parsedMethod))
    {
        Console.WriteLine($"Phân tích chuỗi '{validMethodString}' thành enum thành công: {parsedMethod}");
    }
    else
    {
        Console.WriteLine($"Lỗi: Chuỗi '{validMethodString}' không phải là một phương thức vận chuyển hợp lệ.");
    }

    string invalidMethodString = "InvalidMethod";
    if (Enum.TryParse(invalidMethodString, out parsedMethod))
    {
        Console.WriteLine($"Phân tích chuỗi '{invalidMethodString}' thành enum thành công: {parsedMethod}");
    }
    else
    {
        Console.WriteLine($"Lỗi: Chuỗi '{invalidMethodString}' không phải là một phương thức vận chuyển hợp lệ.");
    }
    ```

### 7. Ví dụ minh họa về Enum

```csharp
using System;

// Khai báo enum ở cấp độ namespace
// Sử dụng byte làm kiểu cơ sở và gán giá trị tường minh
enum OrderStatus : byte
{
    Pending = 1,
    Processing = 2,
    Shipped = 3,
    Delivered = 4,
    Cancelled = 5
}

class EnumExample
{
    static void Main(string[] args)
    {
        Console.WriteLine("--- Ví dụ về Kiểu liệt kê (Enum) trong C# ---");

        // 1. Sử dụng enum
        OrderStatus currentStatus = OrderStatus.Processing;
        Console.WriteLine($"\nTrạng thái đơn hàng hiện tại: {currentStatus}"); // Output: Processing

        // 2. Chuyển đổi Enum sang Số nguyên
        int processingValue = (int)OrderStatus.Processing;
        Console.WriteLine($"Giá trị số của Processing: {processingValue}"); // Output: 2

        // 3. Chuyển đổi Số nguyên sang Enum
        byte statusIdFromDb = 4; // Giả sử ID nhận được từ cơ sở dữ liệu (Delivered = 4)
        OrderStatus statusFromId = (OrderStatus)statusIdFromDb;
        Console.WriteLine($"Trạng thái đơn hàng cho ID {statusIdFromDb}: {statusFromId}"); // Output: Delivered

        // 4. Chuyển đổi Enum sang Chuỗi
        string pendingStatusName = OrderStatus.Pending.ToString();
        Console.WriteLine($"Tên chuỗi của Pending: {pendingStatusName}"); // Output: Pending

        // 5. Sử dụng enum trong câu lệnh switch
        Console.WriteLine("\n--- Xử lý trạng thái đơn hàng ---");
        switch (currentStatus)
        {
            case OrderStatus.Pending:
                Console.WriteLine("Đơn hàng đang chờ xử lý.");
                break;
            case OrderStatus.Processing:
                Console.WriteLine("Đơn hàng đang được xử lý.");
                break;
            case OrderStatus.Shipped:
                Console.WriteLine("Đơn hàng đã được vận chuyển.");
                break;
            case OrderStatus.Delivered:
                Console.WriteLine("Đơn hàng đã được giao thành công.");
                break;
            case OrderStatus.Cancelled:
                Console.WriteLine("Đơn hàng đã bị hủy.");
                break;
            default:
                Console.WriteLine("Trạng thái đơn hàng không xác định.");
                break;
        }

        // 6. Chuyển đổi Chuỗi sang Enum (Parsing an toàn với TryParse)
        Console.WriteLine("\n--- Chuyển đổi Chuỗi sang Enum ---");
        string inputStatusString1 = "Shipped";
        OrderStatus parsedStatus;
        if (Enum.TryParse(inputStatusString1, out parsedStatus))
        {
            Console.WriteLine($"Phân tích chuỗi '{inputStatusString1}' thành enum: {parsedStatus}"); // Output: Shipped
        }
        else
        {
            Console.WriteLine($"Lỗi: Chuỗi '{inputStatusString1}' không phải là trạng thái đơn hàng hợp lệ.");
        }

        string inputStatusString2 = "Returned"; // Chuỗi không hợp lệ
        if (Enum.TryParse(inputStatusString2, out parsedStatus))
        {
            Console.WriteLine($"Phân tích chuỗi '{inputStatusString2}' thành enum: {parsedStatus}");
        }
        else
        {
            Console.WriteLine($"Lỗi: Chuỗi '{inputStatusString2}' không phải là trạng thái đơn hàng hợp lệ."); // Output: Lỗi...
        }
    }
}
```

### 8. Antigravity IDE và Vibe Coding cho Enum

Antigravity IDE có thể biến việc định nghĩa và sử dụng `enum` trở nên dễ dàng và ít lỗi hơn, đặc biệt khi bạn cần quản lý nhiều tập hợp hằng số.

*   **Định nghĩa Enum tự động:**
    *   **Vibe:** "Tạo một enum tên `TrafficLightColor` với các giá trị Đỏ, Vàng, Xanh, gán chúng từ 1 đến 3."
    *   **Antigravity:** Sẽ tạo:
        ```csharp
        enum TrafficLightColor
        {
            Red = 1,
            Yellow = 2,
            Green = 3
        }
        ```
    *   **Vibe:** "Tôi cần một enum `UserRole` với các vai trò Admin, Editor, Viewer. Đảm bảo kiểu cơ sở là `byte`."
    *   **Antigravity:** Sẽ tạo `enum UserRole : byte { Admin, Editor, Viewer }`.
*   **Tạo mã sử dụng Enum:**
    *   **Vibe:** "Viết một hàm kiểm tra vai trò người dùng `userRole` và in ra thông báo chào mừng khác nhau cho Admin, Editor, Viewer."
    *   **Antigravity:** Sẽ tạo một cấu trúc `switch` statement hoàn chỉnh với các `case` cho từng thành viên của `UserRole`.
*   **Chuyển đổi và xử lý an toàn:**
    *   **Vibe:** "Cho một chuỗi đầu vào `statusString`, chuyển đổi nó thành `OrderStatus` một cách an toàn."
    *   **Antigravity:** Sẽ đề xuất và tạo mã sử dụng `Enum.TryParse()`, bao gồm cả logic xử lý khi chuyển đổi thất bại.
    *   **Vibe:** "Hiển thị tất cả các giá trị của enum `OrderStatus` và các giá trị số tương ứng của chúng."
    *   **Antigravity:** Có thể tạo mã sử dụng `Enum.GetValues(typeof(OrderStatus))` và vòng lặp để in ra từng giá trị.
*   **Gỡ lỗi và kiểm tra:** Antigravity có thể giúp bạn kiểm tra các giá trị enum trong quá trình gỡ lỗi hoặc tự động tạo các unit test cơ bản để đảm bảo các chuyển đổi enum hoạt động chính xác.

Với Vibe Coding trong Antigravity, bạn có thể tập trung vào ý nghĩa ngữ nghĩa của các hằng số và cách chúng được sử dụng, thay vì phải lo lắng về cú pháp chi tiết của `enum`, chuyển đổi kiểu hay xử lý lỗi parsing.

## Tóm tắt Phần

*   **Mảng (Arrays):**
    *   Là tập hợp các biến cùng kiểu dữ liệu, kích thước cố định, được lập chỉ mục từ 0.
    *   Là **kiểu tham chiếu**, biến mảng lưu trữ địa chỉ trên Stack, dữ liệu mảng được cấp phát trên Heap.
    *   Các phần tử được khởi tạo với giá trị mặc định của kiểu dữ liệu.
    *   Gán mảng sao chép tham chiếu, không phải dữ liệu.
*   **Chuỗi (Strings):**
    *   Là chuỗi các ký tự Unicode, `string` là bí danh của `System.String`.
    *   Là **kiểu tham chiếu** và **bất biến** (immutable) – mọi thao tác "sửa đổi" đều tạo ra một đối tượng chuỗi mới trên Heap.
    *   Hỗ trợ các ký tự thoát (`\n`, `\t`, `\\`, `\"`) và chuỗi nguyên văn (`@""`) để xử lý văn bản phức tạp.
    *   Có nhiều phương thức tiện ích để nối, định dạng (đặc biệt là string interpolation `$`) và thao tác.
*   **Kiểu liệt kê (Enums):**
    *   Là **kiểu giá trị** định nghĩa một tập hợp các hằng số được đặt tên, giúp mã nguồn dễ đọc và bảo trì hơn.
    *   Mặc định, thành viên đầu tiên có giá trị 0, các thành viên tiếp theo tăng dần 1. Nên gán giá trị tường minh.
    *   Kiểu cơ sở mặc định là `int`, có thể thay đổi (`byte`, `short`, v.v.).
    *   Có thể chuyển đổi giữa enum, số nguyên và chuỗi thông qua ép kiểu tường minh hoặc phương thức `Enum.Parse()`/`Enum.TryParse()` (nên dùng `TryParse()` để an toàn).
*   **Antigravity IDE & Vibe Coding:** Tận dụng khả năng của Antigravity để mô tả ý định của bạn bằng ngôn ngữ tự nhiên. Antigravity sẽ giúp bạn tạo, thao tác và quản lý mảng, chuỗi và enum một cách hiệu quả, tự động hóa các tác vụ lặp lại và đảm bảo tính đúng đắn của mã nguồn.

<!-- REVIEWED_BY_AGENT -->
