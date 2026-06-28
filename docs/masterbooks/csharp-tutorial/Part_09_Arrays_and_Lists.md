# Phần 9: Mảng và Danh sách - Quản lý Dữ liệu Hiệu quả trong C#

Trong thế giới lập trình, việc tổ chức và quản lý một tập hợp các giá trị hoặc đối tượng cùng loại là một yêu cầu cơ bản và thường xuyên. Tưởng tượng bạn cần lưu trữ điểm số của 100 sinh viên, danh sách sản phẩm trong một cửa hàng, hoặc các tác vụ cần thực hiện. Việc khai báo hàng trăm biến riêng lẻ không chỉ cồng kềnh mà còn rất khó quản lý. Đây chính là lúc các cấu trúc dữ liệu tập hợp (collection data structures) phát huy sức mạnh.

Phần này sẽ giới thiệu cho bạn hai trong số các cấu trúc dữ liệu quan trọng nhất trong C# để giải quyết vấn đề này: **Mảng (Arrays)** và **Danh sách (Lists - `List<T>`)**. Chúng ta sẽ đi sâu vào cách khai báo, khởi tạo, truy cập và thao tác với chúng, đồng thời khám phá cơ chế cấp phát bộ nhớ ngầm (under the hood), những điểm khác biệt cốt lõi và trường hợp sử dụng phù hợp cho từng loại. Việc nắm vững mảng và danh sách là nền tảng vững chắc để bạn xây dựng các ứng dụng C# hiệu quả, mạnh mẽ và dễ bảo trì. Đặc biệt, chúng ta sẽ lồng ghép tư duy **Vibe Coding** và cách bạn có thể tận dụng **Antigravity IDE** - hệ thống Agentic AI siêu việt - để nâng cao năng suất và hiểu biết trong quá trình làm việc với các cấu trúc này.

## 1. Giới thiệu về Bộ sưu tập dữ liệu (Data Collections)

Trong C#, khi bạn cần làm việc với nhiều mục dữ liệu có cùng kiểu, thay vì khai báo hàng loạt biến riêng lẻ, bạn sẽ sử dụng các cấu trúc dữ liệu cho phép nhóm chúng lại với nhau. Các cấu trúc này được gọi chung là "bộ sưu tập" (collections). Chúng cung cấp một cách có tổ chức để lưu trữ, truy xuất và thao tác với dữ liệu. Mảng và Danh sách là hai trong số những bộ sưu tập được sử dụng phổ biến nhất và là điểm khởi đầu tuyệt vời để bạn làm quen với khái niệm này.

> [!NOTE]
> Khái niệm "bộ sưu tập" rất rộng trong .NET, bao gồm nhiều lớp khác nhau như `Queue`, `Stack`, `Dictionary`, `HashSet`, v.v. Các lớp này đều nằm trong namespace `System.Collections` hoặc `System.Collections.Generic`. Trong phần này, chúng ta sẽ tập trung vào Mảng và Danh sách, những cấu trúc cơ bản và được sử dụng rộng rãi nhất.

## 2. Mảng (Arrays)

Mảng là một cấu trúc dữ liệu cơ bản cho phép bạn lưu trữ một số lượng **cố định** các biến cùng kiểu dữ liệu. Khi một mảng được tạo, kích thước của nó được xác định và không thể thay đổi sau đó. Điều này làm cho mảng trở thành lựa chọn lý tưởng khi bạn biết trước chính xác số lượng phần tử cần lưu trữ.

### 2.1. Khái niệm và Đặc điểm cốt lõi

*   **Kích thước cố định:** Đây là đặc điểm quan trọng nhất của mảng. Một khi bạn khai báo và cấp phát bộ nhớ cho một mảng với một kích thước nhất định, bạn không thể thêm hoặc bớt phần tử khỏi nó. Nếu bạn cần nhiều hơn hoặc ít hơn, bạn phải tạo một mảng mới và sao chép các phần tử.
*   **Cùng kiểu dữ liệu:** Tất cả các phần tử trong một mảng phải có cùng kiểu dữ liệu. Ví dụ, một mảng `int` chỉ có thể chứa các số nguyên, không thể chứa chuỗi hoặc đối tượng khác.
*   **Truy cập bằng chỉ số:** Các phần tử trong mảng được truy cập thông qua một chỉ số số nguyên, bắt đầu từ `0`. Chỉ số `0` đại diện cho phần tử đầu tiên, chỉ số `1` cho phần tử thứ hai, và cứ thế.
*   **Thứ tự đảm bảo:** Các phần tử được lưu trữ theo một thứ tự xác định và duy trì thứ tự đó.

### 2.1.1. Cơ chế cấp phát bộ nhớ cho Mảng (Under the Hood)

Trong C#, mảng là một **kiểu tham chiếu (reference type)**, ngay cả khi nó chứa các kiểu giá trị (value types) như `int` hay `double`. Điều này có ý nghĩa quan trọng đối với cách bộ nhớ được cấp phát và quản lý:

1.  **Biến mảng trên Stack:** Khi bạn khai báo một biến mảng (ví dụ: `int[] numbers;`), biến `numbers` này thực chất là một **tham chiếu (reference)**, được lưu trữ trên **Stack**. Tham chiếu này ban đầu có giá trị `null` nếu không được khởi tạo.
2.  **Dữ liệu mảng trên Heap:** Khi bạn khởi tạo mảng (ví dụ: `numbers = new int[5];`), một vùng bộ nhớ liên tục có kích thước đủ lớn để chứa 5 phần tử kiểu `int` sẽ được cấp phát trên **Heap**. Địa chỉ của vùng bộ nhớ này sau đó được gán cho biến `numbers` trên Stack.

    *   **Nếu mảng chứa các kiểu giá trị (Value Types) (ví dụ: `int`, `double`, `bool`, `struct`):** Các giá trị thực tế của các phần tử đó sẽ được lưu trữ **trực tiếp** trong vùng bộ nhớ của mảng trên Heap. Ví dụ, trong `int[] numbers = new int[5];`, 5 ô nhớ cho 5 số nguyên sẽ nằm liền kề nhau trên Heap.
    *   **Nếu mảng chứa các kiểu tham chiếu (Reference Types) (ví dụ: `string`, các đối tượng tùy chỉnh như `Product`, `Student`):** Mảng sẽ lưu trữ các **tham chiếu** (con trỏ) tới các đối tượng đó. Bản thân các đối tượng `string` hoặc `Product` sẽ được cấp phát ở các vị trí khác nhau trên Heap. Các tham chiếu này trong mảng sẽ trỏ đến chúng.

**Minh họa (ý tưởng):**

```
Stack:
+-------------------+
| numbers (ref) ----|-----> (Địa chỉ trên Heap)
+-------------------+

Heap:
+-------------------------------------------------------------+
| [int] [int] [int] [int] [int] (5 số nguyên liền kề)         |  <-- Nếu là int[]
+-------------------------------------------------------------+
| [ref] [ref] [ref] [ref] [ref] (5 tham chiếu)                |  <-- Nếu là string[] hoặc Product[]
|   |     |     |     |     |                               |
|   V     V     V     V     V                               |
| "Hello" "World" obj1  obj2  null (Các đối tượng string/Product rải rác trên Heap)
+-------------------------------------------------------------+
```

Việc hiểu rõ cơ chế này rất quan trọng để tránh các lỗi như `NullReferenceException` (khi biến mảng chưa trỏ đến vùng nhớ nào trên Heap) hoặc hiểu cách sao chép mảng hoạt động (chỉ sao chép tham chiếu nếu mảng chứa kiểu tham chiếu).

### 2.2. Mảng một chiều (Single-dimensional Arrays)

Đây là loại mảng cơ bản nhất, thường được sử dụng để lưu trữ một danh sách các mục.

**Cú pháp khai báo và khởi tạo:**

```csharp
using System; // Cần thiết cho Console.WriteLine

public class SingleDimensionalArrayExample
{
    public static void Main(string[] args)
    {
        // 1. Khai báo một mảng số nguyên có 5 phần tử, chưa gán giá trị cụ thể.
        // Các phần tử sẽ được khởi tạo với giá trị mặc định của kiểu (0 cho int).
        int[] numbers = new int[5];
        Console.WriteLine($"Mảng 'numbers' ban đầu: {string.Join(", ", numbers)}"); // Output: 0, 0, 0, 0, 0

        // 2. Khởi tạo và gán giá trị ban đầu ngay lập tức.
        // Kích thước mảng sẽ được xác định tự động dựa trên số lượng phần tử cung cấp.
        int[] initialNumbers = new int[] { 1, 2, 3, 4, 5 };
        Console.WriteLine($"Mảng 'initialNumbers': {string.Join(", ", initialNumbers)}"); // Output: 1, 2, 3, 4, 5

        // 3. Cú pháp viết tắt khi khởi tạo và gán giá trị ngay lập tức.
        // Không cần từ khóa 'new int[]' nếu khai báo và khởi tạo cùng dòng.
        string[] names = { "Alice", "Bob", "Charlie" };
        Console.WriteLine($"Mảng 'names': {string.Join(", ", names)}"); // Output: Alice, Bob, Charlie

        // 4. Sử dụng từ khóa 'var' để C# tự suy luận kiểu.
        // Chỉ dùng khi khởi tạo ngay lập tức.
        var anotherNumbers = new int[] { 100, 200, 300 };
        Console.WriteLine($"Mảng 'anotherNumbers': {string.Join(", ", anotherNumbers)}"); // Output: 100, 200, 300

        // Hoặc ngắn gọn hơn nữa, C# suy luận cả 'new int[]'
        var yetAnotherNumbers = new[] { 1, 2, 3 }; // Kiểu int[] được suy luận
        Console.WriteLine($"Mảng 'yetAnotherNumbers': {string.Join(", ", yetAnotherNumbers)}"); // Output: 1, 2, 3
    }
}
```

**Truy cập phần tử:**

Các phần tử được truy cập bằng chỉ số trong dấu ngoặc vuông `[]`. Chỉ số đầu tiên là `0`.

```csharp
Console.WriteLine(initialNumbers[0]); // Output: 1 (phần tử đầu tiên)
Console.WriteLine(initialNumbers[4]); // Output: 5 (phần tử cuối cùng)

// Gán lại giá trị cho một phần tử
initialNumbers[2] = 99; // Thay đổi giá trị của phần tử thứ 3
Console.WriteLine(initialNumbers[2]); // Output: 99
```

> [!CAUTION]
> Truy cập một chỉ số nằm ngoài phạm vi của mảng (ví dụ: `initialNumbers[5]` hoặc `initialNumbers[-1]`) sẽ gây ra lỗi `IndexOutOfRangeException` trong thời gian chạy. Đây là một lỗi phổ biến, đặc biệt khi lặp qua mảng.

### 2.3. Mảng đa chiều (Multi-dimensional Arrays)

Mảng đa chiều được sử dụng để biểu diễn các cấu trúc dữ liệu có nhiều hơn một chiều, như ma trận, bảng, hoặc không gian 3D. Trong C#, chúng ta có hai loại mảng đa chiều chính: mảng hình chữ nhật (rectangular arrays) và mảng jagged (jagged arrays).

#### 2.3.1. Mảng hình chữ nhật (Rectangular Arrays)

Mảng hình chữ nhật là loại mảng đa chiều mà mỗi "hàng" có cùng số lượng "cột" (hoặc mỗi chiều có cùng kích thước). Chúng tạo thành một hình chữ nhật (hoặc khối lập phương cho 3 chiều trở lên) hoàn chỉnh và được lưu trữ trong một khối bộ nhớ liên tục.

**Cú pháp khai báo và khởi tạo:**

Các chiều được phân tách bằng dấu phẩy bên trong một cặp dấu ngoặc vuông `[,]`.

```csharp
// Mảng 2 chiều (ma trận 3 hàng, 5 cột)
// Tất cả các phần tử sẽ được khởi tạo về 0 (giá trị mặc định của int)
int[,] matrix = new int[3, 5];

// Khởi tạo và gán giá trị ban đầu
// Kích thước được suy luận từ dữ liệu cung cấp: 3 hàng, 3 cột
int[,] prefilledMatrix = new int[,]
{
    { 1, 2, 3 }, // Hàng 0
    { 4, 5, 6 }, // Hàng 1
    { 7, 8, 9 }  // Hàng 2
};

// Mảng 3 chiều (ví dụ: không gian 3x3x3)
int[,,] cube = new int[3, 3, 3];
```

**Truy cập phần tử:**

Truy cập bằng cách cung cấp chỉ số cho mỗi chiều, phân tách bằng dấu phẩy.

```csharp
Console.WriteLine(prefilledMatrix[0, 0]); // Output: 1 (Hàng 0, Cột 0)
Console.WriteLine(prefilledMatrix[1, 2]); // Output: 6 (Hàng 1, Cột 2)

// Gán giá trị
prefilledMatrix[2, 1] = 100; // Hàng 2, Cột 1
Console.WriteLine(prefilledMatrix[2, 1]); // Output: 100
```

#### 2.3.2. Mảng Jagged (Jagged Arrays - "Mảng của các mảng")

Mảng jagged là một mảng mà các phần tử của nó là các mảng khác nhau, và các mảng con này có thể có độ dài khác nhau. Chúng được gọi là "jagged" (răng cưa) vì các hàng không nhất thiết phải có cùng độ dài, tạo ra một hình dạng không đều. Về mặt bộ nhớ, mảng jagged là một mảng các **tham chiếu** đến các mảng một chiều riêng biệt trên Heap.

**Cú pháp khai báo và khởi tạo:**

Sử dụng nhiều cặp dấu ngoặc vuông `[][]`. Cặp dấu ngoặc vuông đầu tiên chỉ định kích thước của mảng cấp cao nhất (số lượng "hàng" hoặc mảng con), và mỗi phần tử của mảng cấp cao nhất là một mảng con riêng biệt.

```csharp
// Khai báo một mảng jagged có 3 hàng (mảng con), nhưng chưa định nghĩa kích thước của từng mảng con.
// Các phần tử của jaggedArray[0], jaggedArray[1], jaggedArray[2] ban đầu sẽ là null.
int[][] jaggedArray = new int[3][];

// Khởi tạo từng mảng con với kích thước khác nhau
jaggedArray[0] = new int[4] { 1, 2, 3, 4 };      // Hàng 0 có 4 phần tử
jaggedArray[1] = new int[2] { 5, 6 };          // Hàng 1 có 2 phần tử
jaggedArray[2] = new int[3] { 7, 8, 9 };      // Hàng 2 có 3 phần tử

// Khởi tạo và gán giá trị ban đầu ngay lập tức
int[][] anotherJaggedArray = new int[][]
{
    new int[] { 10, 20 },
    new int[] { 30, 40, 50 },
    new int[] { 60 }
};
```

**Truy cập phần tử:**

Truy cập bằng cách sử dụng các cặp dấu ngoặc vuông riêng biệt cho từng chiều.

```csharp
Console.WriteLine(jaggedArray[0][0]); // Output: 1 (Hàng 0, phần tử đầu tiên)
Console.WriteLine(jaggedArray[1][1]); // Output: 6 (Hàng 1, phần tử thứ hai)
Console.WriteLine(jaggedArray[2][2]); // Output: 9 (Hàng 2, phần tử thứ ba)
```

#### 2.3.3. So sánh Mảng hình chữ nhật và Mảng Jagged

| Đặc điểm          | Mảng hình chữ nhật (`int[,]`)                                  | Mảng Jagged (`int[][]`)                                       |
| :---------------- | :------------------------------------------------------------- | :-------------------------------------------------------------- |
| **Cấu trúc**      | Một khối bộ nhớ liên tục trên Heap. Mỗi hàng có cùng số cột.   | Một mảng các tham chiếu trên Heap, mỗi tham chiếu trỏ đến một mảng một chiều riêng biệt trên Heap. Mỗi mảng con có thể có độ dài khác nhau. |
| **Cú pháp**       | Một cặp `[]` với các chiều phân tách bằng dấu phẩy: `new int[rows, cols]` | Nhiều cặp `[]` riêng biệt: `new int[rows][]`                    |
| **Truy cập**      | `array[row, col]`                                              | `array[row][col]`                                               |
| **Bộ nhớ**        | Hiệu quả hơn một chút nếu tất cả các hàng có cùng kích thước, vì không cần lưu trữ các tham chiếu riêng lẻ cho từng hàng. | Có thể sử dụng nhiều bộ nhớ hơn một chút do phải lưu trữ một mảng các tham chiếu tới các mảng con. |
| **Hiệu suất**     | CLR (Common Language Runtime) có thể tối ưu hóa truy cập cho mảng hình chữ nhật. Tuy nhiên, việc truy cập phần tử thường đòi hỏi tính toán chỉ số phức tạp hơn (row * num_cols + col). | Thường nhanh hơn một chút cho các ma trận lớn vì mỗi mảng con là một mảng một chiều tiêu chuẩn, và việc truy cập `array[row][col]` chỉ đơn giản là hai lần truy cập mảng một chiều. |
| **Sử dụng**       | Phù hợp cho các ma trận có kích thước cố định, hình dạng đều (ví dụ: bảng tính, lưới trò chơi). | Phù hợp cho dữ liệu có cấu trúc không đều, hoặc khi bạn cần linh hoạt về kích thước của từng "hàng" (ví dụ: danh sách các học sinh và điểm số của họ cho các môn học khác nhau). |

> [!TIP]
> Trong thực tế, sự khác biệt về hiệu suất giữa mảng hình chữ nhật và mảng jagged thường rất nhỏ đối với hầu hết các ứng dụng. Đừng quá bận tâm về điều này trừ khi bạn đang làm việc với các hệ thống yêu cầu hiệu suất cực cao và đã thực hiện đo lường (profiling) để xác định điểm nghẽn. Hãy chọn loại mảng nào làm cho mã của bạn dễ đọc, dễ bảo trì và phù hợp với cấu trúc dữ liệu tự nhiên của vấn đề.

### 2.4. Các phương thức và thuộc tính hữu ích với `System.Array`

Tất cả các mảng trong C# đều kế thừa từ lớp cơ sở trừu tượng `System.Array`. Lớp này cung cấp nhiều thuộc tính và phương thức hữu ích để thao tác với mảng.

> [!NOTE]
> `System.Array` là một lớp **tham chiếu** và là lớp cơ sở cho tất cả các loại mảng trong C#. Nó cung cấp các chức năng chung cho việc tạo, thao tác, tìm kiếm và sắp xếp các mảng.

**Ví dụ minh họa:**

```csharp
using System; // Cần thiết cho Console.WriteLine và string.Join
// using System.Linq; // Không cần thiết cho string.Join(IEnumerable<T>)

public class ArrayUtilities
{
    public static void Main(string[] args)
    {
        Console.WriteLine("--- Thao tác với Mảng ---");

        // Khởi tạo một mảng
        var numbers = new int[] { 3, 9, 2, 14, 6, 0 };
        Console.WriteLine("Mảng ban đầu: " + string.Join(", ", numbers));

        // 1. Thuộc tính Length: Trả về số lượng phần tử trong mảng (instance member)
        Console.WriteLine($"\nLength: {numbers.Length}"); // Output: 6

        // 2. Phương thức Array.IndexOf(): Tìm vị trí của một phần tử (static method)
        //    Trả về chỉ số của phần tử đầu tiên tìm thấy, hoặc -1 nếu không tìm thấy.
        int indexOf9 = Array.IndexOf(numbers, 9);
        Console.WriteLine($"Index of 9: {indexOf9}"); // Output: 1 (chỉ số của số 9)

        int indexOf100 = Array.IndexOf(numbers, 100);
        Console.WriteLine($"Index of 100: {indexOf100}"); // Output: -1 (không tìm thấy)

        // 3. Phương thức Array.Clear(): Đặt một phạm vi phần tử về giá trị mặc định (static method)
        //    (0 cho int, false cho bool, null cho kiểu tham chiếu)
        Console.WriteLine("\nSử dụng Array.Clear(numbers, 0, 2):");
        Array.Clear(numbers, 0, 2); // Xóa 2 phần tử đầu tiên (tại chỉ số 0 và 1)
        Console.WriteLine("Mảng sau khi Clear: " + string.Join(", ", numbers)); // Output: 0, 0, 2, 14, 6, 0

        // 4. Phương thức Array.Copy(): Sao chép một phần tử từ mảng này sang mảng khác (static method)
        var anotherNumbers = new int[3];
        Array.Copy(numbers, anotherNumbers, 3); // Sao chép 3 phần tử đầu tiên từ 'numbers' sang 'anotherNumbers'
        Console.WriteLine("\n'anotherNumbers' sau khi Copy từ 'numbers': " + string.Join(", ", anotherNumbers)); // Output: 0, 0, 2

        // Khôi phục mảng gốc để sắp xếp và đảo ngược
        numbers = new int[] { 3, 9, 2, 14, 6, 0 };
        Console.WriteLine("\nMảng được khôi phục: " + string.Join(", ", numbers));

        // 5. Phương thức Array.Sort(): Sắp xếp các phần tử trong mảng theo thứ tự tăng dần (static method)
        Array.Sort(numbers);
        Console.WriteLine("Mảng sau khi Sort: " + string.Join(", ", numbers)); // Output: 0, 2, 3, 6, 9, 14

        // 6. Phương thức Array.Reverse(): Đảo ngược thứ tự các phần tử trong mảng (static method)
        Array.Reverse(numbers);
        Console.WriteLine("Mảng sau khi Reverse: " + string.Join(", ", numbers)); // Output: 14, 9, 6, 3, 2, 0
    }
}
```

> [!TIP]
> **Phương thức tĩnh (Static Methods) và Thành viên thể hiện (Instance Members):**
>
> *   **Thành viên thể hiện (Instance Members):** Là các thuộc tính hoặc phương thức được truy cập thông qua một **đối tượng cụ thể** của lớp. Ví dụ: `numbers.Length` (thuộc tính `Length` thuộc về đối tượng `numbers` cụ thể).
> *   **Phương thức tĩnh (Static Methods):** Là các phương thức được truy cập trực tiếp thông qua **tên của lớp**, không cần tạo đối tượng. Ví dụ: `Array.IndexOf()`, `Array.Clear()`, `Array.Sort()`. Các phương thức này thường thực hiện các thao tác chung trên dữ liệu được truyền vào như một tham số.
>
> Khi bạn thấy một phương thức được gọi trực tiếp trên tên lớp (ví dụ `Array.Sort`), đó là một phương thức tĩnh. Nếu nó được gọi trên một biến đối tượng (ví dụ `myObject.DoSomething()`), đó là một phương thức thể hiện. Bạn có thể tra cứu tài liệu MSDN (Microsoft Developer Network) để xem một thành viên là tĩnh hay thể hiện (thường có ký hiệu 'S' màu đỏ cho phương thức tĩnh).

### 2.5. Tư duy Vibe Coding và Antigravity IDE với Mảng

Khi làm việc với mảng, tư duy **Vibe Coding** khuyến khích bạn cảm nhận và hiểu rõ các giới hạn cố định của chúng. Bạn cần "vibe" được rằng kích thước sẽ không thay đổi, và mọi thao tác thêm/bớt sẽ đòi hỏi việc tạo mảng mới.

**Antigravity IDE** có thể trở thành một trợ thủ đắc lực trong việc này:

*   **Tự động sinh mã khởi tạo:** Khi bạn cần một mảng với kích thước và giá trị ban đầu, Antigravity có thể tự động sinh ra cú pháp khởi tạo phù hợp, bao gồm cả cú pháp viết tắt. Ví dụ, bạn chỉ cần nói "tạo một mảng số nguyên từ 1 đến 10", Antigravity sẽ viết `int[] numbers = { 1, 2, ..., 10 };`.
*   **Gợi ý phương thức `System.Array`:** Nếu bạn đang cần sắp xếp, tìm kiếm, hoặc sao chép mảng, Antigravity có thể gợi ý ngay các phương thức tĩnh phù hợp từ `System.Array` (như `Array.Sort`, `Array.IndexOf`), giúp bạn viết mã nhanh và chính xác hơn mà không cần nhớ tên chính xác.
*   **Phân tích lỗi `IndexOutOfRangeException`:** Antigravity có thể phân tích các vòng lặp hoặc truy cập mảng của bạn và cảnh báo sớm về các trường hợp có thể dẫn đến `IndexOutOfRangeException`, thậm chí đề xuất các cách sửa lỗi như kiểm tra `array.Length`.
*   **Refactor mã nguồn:** Nếu bạn có một mảng và nhận ra mình cần thêm/bớt phần tử thường xuyên, Antigravity có thể đề xuất và hỗ trợ bạn refactor mã nguồn từ việc sử dụng mảng sang `List<T>`, giúp bạn chuyển đổi mượt mà hơn.
*   **Vibe Coding để tối ưu:** Khi bạn làm việc với dữ liệu hiệu suất cao và biết rõ kích thước, Vibe Coding với Antigravity có thể giúp bạn tối ưu bằng cách tận dụng mảng cho tốc độ truy cập trực tiếp và tránh chi phí quản lý bộ nhớ động. Antigravity có thể giúp bạn viết các vòng lặp `for` hiệu quả để duyệt mảng.

## 3. Danh sách (Lists - `System.Collections.Generic.List<T>`)

Trong nhiều trường hợp thực tế, bạn không biết trước chính xác số lượng phần tử cần lưu trữ, hoặc số lượng đó có thể thay đổi liên tục trong quá trình chạy chương trình. Mảng với kích thước cố định sẽ không phù hợp. Đây là lúc **Danh sách** (cụ thể là `System.Collections.Generic.List<T>`) phát huy tác dụng. `List<T>` là một bộ sưu tập động, có thể thay đổi kích thước linh hoạt trong quá trình chạy chương trình.

### 3.1. Khái niệm và Đặc điểm cốt lõi

*   **Kích thước động:** Điểm khác biệt lớn nhất so với mảng. Bạn có thể thêm hoặc xóa phần tử, và danh sách sẽ tự động điều chỉnh kích thước bên trong để chứa các phần tử đó.
*   **Kiểu Generic (`<T>`):** `List<T>` là một kiểu generic. `T` là một "tham số kiểu" (type parameter) và bạn phải chỉ định kiểu dữ liệu cụ thể mà danh sách sẽ chứa khi khai báo (ví dụ: `List<int>`, `List<string>`, `List<Product>`). Điều này giúp đảm bảo an toàn kiểu (type safety) - bạn không thể thêm một `string` vào `List<int>` - và hiệu suất tốt hơn vì C# không cần thực hiện các thao tác kiểm tra kiểu hoặc boxing/unboxing trong thời gian chạy.
    > [!NOTE]
    > Generics là một chủ đề mạnh mẽ trong C# cho phép bạn viết các lớp và phương thức hoạt động với bất kỳ kiểu dữ liệu nào mà vẫn giữ được tính an toàn kiểu. Chúng ta sẽ tìm hiểu sâu hơn về Generics trong các phần nâng cao của khóa học này.
*   **Tham chiếu (Reference Type):** Tương tự như mảng, `List<T>` cũng là một kiểu tham chiếu. Một biến `List<T>` là một con trỏ tới một đối tượng `List<T>` trên Heap.

### 3.1.1. Cơ chế cấp phát bộ nhớ và Thay đổi kích thước (Under the Hood)

Để đạt được khả năng thay đổi kích thước động, `List<T>` bên trong sử dụng một **mảng một chiều** làm kho lưu trữ chính. Đây là cách nó hoạt động:

1.  **Mảng nội bộ:** Khi bạn tạo một `List<T>`, nó sẽ khởi tạo một mảng nội bộ (ví dụ `_items`) với một `Capacity` (dung lượng) ban đầu nhỏ (thường là 0 hoặc 4).
2.  **`Count` vs. `Capacity`:**
    *   `Count`: Số lượng phần tử thực tế hiện có trong danh sách.
    *   `Capacity`: Kích thước hiện tại của mảng nội bộ. Đây là số lượng phần tử tối đa mà danh sách có thể chứa mà không cần thay đổi kích thước mảng nội bộ.
3.  **Thay đổi kích thước (Resizing):**
    *   Khi bạn `Add` thêm một phần tử và `Count` đạt đến `Capacity` của mảng nội bộ, `List<T>` sẽ tự động thực hiện một thao tác thay đổi kích thước.
    *   Quá trình này bao gồm:
        *   Tạo một mảng mới có kích thước lớn hơn (thường gấp đôi `Capacity` hiện tại).
        *   Sao chép tất cả các phần tử hiện có từ mảng cũ sang mảng mới.
        *   Cập nhật tham chiếu của `List<T>` để trỏ đến mảng mới.
        *   Mảng cũ (không còn được tham chiếu) sẽ trở thành đối tượng đủ điều kiện để Bộ thu gom rác (Garbage Collector) thu hồi.
    *   Thao tác sao chép này có thể tốn kém về hiệu suất (O(N) - tỷ lệ với số lượng phần tử N) nếu nó xảy ra thường xuyên. Tuy nhiên, nhờ chiến lược gấp đôi kích thước, chi phí này được phân bổ đều theo thời gian, dẫn đến hiệu suất thêm phần tử trung bình là O(1) (amortized constant time).
4.  **Giảm kích thước:** `List<T>` không tự động giảm kích thước mảng nội bộ khi bạn xóa phần tử. `Capacity` chỉ giảm khi bạn gọi phương thức `TrimExcess()` hoặc gán một `List<T>` mới.

**Minh họa (ý tưởng):**

```
Ban đầu:
Stack: listRef --> Heap: List<int> Object { Count: 0, Capacity: 0, _items: null }

Add(1):
Stack: listRef --> Heap: List<int> Object { Count: 1, Capacity: 4, _items: [1, 0, 0, 0] }

Add(2), Add(3), Add(4):
Stack: listRef --> Heap: List<int> Object { Count: 4, Capacity: 4, _items: [1, 2, 3, 4] }

Add(5) (Capacity đã đầy, cần Resize):
1. Tạo mảng mới: `new_items = new int[8]`
2. Sao chép: `new_items = [1, 2, 3, 4, 0, 0, 0, 0]`
3. Cập nhật tham chiếu: `_items` giờ trỏ đến `new_items`
4. Mảng cũ `[1,2,3,4]` bị GC thu hồi.
Stack: listRef --> Heap: List<int> Object { Count: 5, Capacity: 8, _items: [1, 2, 3, 4, 5, 0, 0, 0] }
```

Hiểu được cơ chế này giúp bạn đưa ra quyết định tốt hơn, ví dụ như khởi tạo `List<T>` với `Capacity` ban đầu nếu bạn biết trước số lượng phần tử tối thiểu để tránh các lần thay đổi kích thước không cần thiết.

### 3.2. Cú pháp khai báo và khởi tạo

Để sử dụng `List<T>`, bạn cần thêm `using System.Collections.Generic;` vào đầu tệp mã nguồn của mình.

```csharp
using System;
using System.Collections.Generic; // Quan trọng để sử dụng List<T>

public class ListExample
{
    public static void Main(string[] args)
    {
        // 1. Khai báo và khởi tạo một danh sách số nguyên rỗng
        List<int> numbers = new List<int>();
        Console.WriteLine($"Danh sách 'numbers' ban đầu (rỗng): Count = {numbers.Count}");

        // 2. Khởi tạo với các giá trị ban đầu (sử dụng cú pháp khởi tạo đối tượng)
        List<string> names = new List<string>() { "Alice", "Bob", "Charlie" };
        Console.WriteLine($"Danh sách 'names': {string.Join(", ", names)}");

        // 3. Sử dụng 'var' để C# tự suy luận kiểu
        var scores = new List<double>() { 9.5, 8.0, 7.25 };
        Console.WriteLine($"Danh sách 'scores': {string.Join(", ", scores)}");

        // 4. Khởi tạo với một Capacity ban đầu để tối ưu hiệu suất nếu biết trước số lượng gần đúng
        List<int> largeList = new List<int>(100); // Khởi tạo mảng nội bộ có dung lượng 100
        Console.WriteLine($"Danh sách 'largeList' ban đầu: Count = {largeList.Count}, Capacity = {largeList.Capacity}");
    }
}
```

### 3.3. Các phương thức và thuộc tính hữu ích với `List<T>`

`List<T>` cung cấp một tập hợp phong phú các phương thức và thuộc tính để thêm, xóa, tìm kiếm và thao tác với các phần tử, giúp công việc quản lý bộ sưu tập trở nên dễ dàng hơn nhiều so với mảng thuần túy.

**Ví dụ minh họa:**

```csharp
using System;
using System.Collections.Generic; // Cần thiết để sử dụng List<T>
// using System.Linq; // string.Join cho IEnumerable<T> không cần System.Linq

public class ListUtilities
{
    public static void Main(string[] args)
    {
        Console.WriteLine("--- Thao tác với Danh sách ---");

        // Khởi tạo một danh sách với các giá trị ban đầu
        var numbers = new List<int>() { 1, 2, 3, 4 };
        Console.WriteLine("Danh sách ban đầu: " + string.Join(", ", numbers));

        // 1. Phương thức Add(): Thêm một phần tử vào cuối danh sách
        numbers.Add(1); // Thêm số 1 một lần nữa
        numbers.Add(5);
        Console.WriteLine($"\nSau khi Add(1) và Add(5): {string.Join(", ", numbers)}"); // Output: 1, 2, 3, 4, 1, 5

        // 2. Phương thức AddRange(): Thêm một bộ sưu tập (mảng hoặc danh sách khác) vào danh sách
        var moreNumbers = new int[] { 8, 5, 6, 7 };
        numbers.AddRange(moreNumbers);
        Console.WriteLine($"Sau khi AddRange(moreNumbers): {string.Join(", ", numbers)}"); // Output: 1, 2, 3, 4, 1, 5, 8, 5, 6, 7

        // 3. Thuộc tính Count: Trả về số lượng phần tử hiện có trong danh sách
        Console.WriteLine($"\nCount: {numbers.Count}"); // Output: 10
        // Thuộc tính Capacity: Kích thước của mảng nội bộ
        Console.WriteLine($"Capacity: {numbers.Capacity}"); // Sẽ là một số >= Count (ví dụ 16 nếu bắt đầu từ 4 và resize 2 lần)

        // 4. Phương thức IndexOf(): Tìm chỉ số của phần tử đầu tiên tìm thấy
        Console.WriteLine($"\nIndex of 1 (first occurrence): {numbers.IndexOf(1)}"); // Output: 0

        // 5. Phương thức LastIndexOf(): Tìm chỉ số của phần tử cuối cùng tìm thấy
        Console.WriteLine($"Last Index of 1: {numbers.LastIndexOf(1)}"); // Output: 4
        Console.WriteLine($"Last Index of 5: {numbers.LastIndexOf(5)}"); // Output: 7

        // 6. Phương thức Contains(): Kiểm tra xem danh sách có chứa phần tử nào đó không
        Console.WriteLine($"Contains 100? {numbers.Contains(100)}"); // Output: False
        Console.WriteLine($"Contains 8? {numbers.Contains(8)}"); // Output: True

        // 7. Phương thức Remove(): Xóa phần tử ĐẦU TIÊN tìm thấy
        numbers.Remove(1); // Xóa số 1 đầu tiên
        Console.WriteLine($"\nSau khi Remove(1): {string.Join(", ", numbers)}"); // Output: 2, 3, 4, 1, 5, 8, 5, 6, 7

        // 8. Phương thức RemoveAt(index): Xóa phần tử tại chỉ số cụ thể
        numbers.RemoveAt(0); // Xóa phần tử đầu tiên (số 2)
        Console.WriteLine($"Sau khi RemoveAt(0): {string.Join(", ", numbers)}"); // Output: 3, 4, 1, 5, 8, 5, 6, 7

        // 9. Phương thức Insert(index, item): Chèn một phần tử vào vị trí chỉ định
        numbers.Insert(0, 100); // Chèn 100 vào đầu danh sách
        Console.WriteLine($"Sau khi Insert(0, 100): {string.Join(", ", numbers)}"); // Output: 100, 3, 4, 1, 5, 8, 5, 6, 7

        // 10. Phương thức Sort(): Sắp xếp các phần tử trong danh sách
        numbers.Sort();
        Console.WriteLine($"Sau khi Sort(): {string.Join(", ", numbers)}"); // Output: 1, 3, 4, 5, 5, 6, 7, 8, 100

        // 11. Phương thức Reverse(): Đảo ngược thứ tự các phần tử
        numbers.Reverse();
        Console.WriteLine($"Sau khi Reverse(): {string.Join(", ", numbers)}"); // Output: 100, 8, 7, 6, 5, 5, 4, 3, 1

        Console.WriteLine("\n--- Xóa tất cả các số 5 khỏi danh sách ---");
        // Thêm lại số 5 để demo RemoveAll
        numbers.Add(5);
        numbers.Add(5);
        Console.WriteLine($"Danh sách với số 5: {string.Join(", ", numbers)}");

        numbers.RemoveAll(n => n == 5); // Xóa tất cả các phần tử có giá trị là 5
        Console.WriteLine($"Sau khi RemoveAll(n => n == 5): {string.Join(", ", numbers)}"); // Output: 100, 8, 7, 6, 4, 3, 1

        // 12. Phương thức Clear(): Xóa tất cả các phần tử khỏi danh sách
        numbers.Clear();
        Console.WriteLine($"\nSau khi Clear(): {string.Join(", ", numbers)}"); // Output: (trống)
        Console.WriteLine($"Count sau khi Clear(): {numbers.Count}"); // Output: 0
    }
}
```

#### 3.3.1. Cảnh báo quan trọng: Không sửa đổi bộ sưu tập trong vòng lặp `foreach`!

Đây là một trong những lỗi phổ biến nhất khi làm việc với `List<T>` và các bộ sưu tập khác.

> [!CAUTION]
> Khi bạn lặp qua một bộ sưu tập bằng vòng lặp `foreach` và cố gắng thêm hoặc xóa phần tử khỏi bộ sưu tập đó bên trong vòng lặp, C# sẽ ném ra một `InvalidOperationException` với thông báo "Collection was modified; enumeration operation may not execute." (Bộ sưu tập đã được sửa đổi; thao tác liệt kê không thể thực thi).
>
> Lý do là `foreach` dựa vào một "enumerator" để duyệt qua các phần tử. Khi bộ sưu tập bị sửa đổi, enumerator trở nên không hợp lệ, dẫn đến lỗi.

**Đoạn mã GÂY LỖI:**

```csharp
/*
var numbers = new List<int>() { 1, 2, 3, 4, 5 };
foreach (var number in numbers)
{
    if (number == 5)
    {
        numbers.Remove(number); // LỖI: InvalidOperationException
    }
}
*/
```

**Các cách đúng để sửa đổi bộ sưu tập khi đang lặp:**

1.  **Sử dụng vòng lặp `for` (lặp ngược từ cuối về đầu):** Đây là cách an toàn và hiệu quả khi bạn cần xóa nhiều phần tử dựa trên điều kiện. Việc lặp ngược đảm bảo rằng việc xóa một phần tử không ảnh hưởng đến chỉ số của các phần tử mà bạn sẽ duyệt tiếp theo.

    ```csharp
    var numbersToModify = new List<int>() { 1, 5, 2, 5, 3, 5, 4 };
    Console.WriteLine($"\nTrước khi xóa số 5 (dùng for): {string.Join(", ", numbersToModify)}");
    for (int i = numbersToModify.Count - 1; i >= 0; i--)
    {
        if (numbersToModify[i] == 5)
        {
            numbersToModify.RemoveAt(i); // Xóa phần tử tại chỉ số i
        }
    }
    Console.WriteLine($"Sau khi xóa số 5 (dùng for): {string.Join(", ", numbersToModify)}");
    // Output: 1, 2, 3, 4
    ```

2.  **Sử dụng phương thức `RemoveAll()`:** Đây là cách hiệu quả và an toàn nhất khi bạn muốn xóa tất cả các phần tử thỏa mãn một điều kiện nhất định.

    ```csharp
    var numbersToModify2 = new List<int>() { 1, 5, 2, 5, 3, 5, 4 };
    Console.WriteLine($"\nTrước khi xóa số 5 (dùng RemoveAll): {string.Join(", ", numbersToModify2)}");
    numbersToModify2.RemoveAll(n => n == 5); // Xóa tất cả các phần tử có giá trị là 5
    Console.WriteLine($"Sau khi xóa số 5 (dùng RemoveAll): {string.Join(", ", numbersToModify2)}");
    // Output: 1, 2, 3, 4
    ```

3.  **Tạo một danh sách tạm thời:** Nếu bạn cần phức tạp hơn, bạn có thể tạo một danh sách tạm thời chứa các phần tử cần xóa/thêm, hoặc chứa các phần tử *còn lại*, sau đó gán lại hoặc thực hiện các thao tác sau vòng lặp `foreach`.

    ```csharp
    var originalList = new List<string>() { "Apple", "Banana", "Orange", "Grape", "Pineapple" };
    var itemsToRemove = new List<string>();

    foreach (var item in originalList)
    {
        if (item.Contains("apple", StringComparison.OrdinalIgnoreCase)) // Tìm các mục có chứa "apple"
        {
            itemsToRemove.Add(item);
        }
    }

    foreach (var item in itemsToRemove)
    {
        originalList.Remove(item);
    }
    Console.WriteLine($"\nDanh sách sau khi xóa các mục chứa 'apple': {string.Join(", ", originalList)}");
    // Output: Banana, Grape
    ```

### 3.4. Tư duy Vibe Coding và Antigravity IDE với Danh sách

Với `List<T>`, tư duy **Vibe Coding** khuyến khích bạn chấp nhận sự linh hoạt và khả năng mở rộng. Bạn "vibe" được rằng dữ liệu có thể thay đổi, và `List<T>` sẽ xử lý phần lớn công việc quản lý bộ nhớ cho bạn.

**Antigravity IDE** trở nên vô cùng mạnh mẽ khi làm việc với `List<T>`:

*   **Tự động hoàn thành và gợi ý phương thức:** Khi bạn gõ `myList.`, Antigravity sẽ hiển thị và giải thích các phương thức hữu ích như `Add`, `Remove`, `IndexOf`, `Sort`, `Clear`, v.v., giúp bạn nhanh chóng tìm thấy chức năng mình cần.
*   **Hỗ trợ refactor thông minh:** Nếu Antigravity nhận thấy bạn đang sử dụng một mảng nhưng lại liên tục tạo mảng mới để thêm/bớt phần tử, nó có thể đề xuất tự động refactor mã của bạn để sử dụng `List<T>`, giảm thiểu mã lặp và cải thiện hiệu suất.
*   **Cảnh báo và sửa lỗi `InvalidOperationException`:** Khi bạn cố gắng sửa đổi `List<T>` bên trong vòng lặp `foreach`, Antigravity có thể cảnh báo bạn về `InvalidOperationException` tiềm ẩn và tự động đề xuất các giải pháp đúng đắn, như chuyển sang vòng lặp `for` lặp ngược hoặc sử dụng `RemoveAll()`.
*   **Tối ưu hóa `Capacity`:** Nếu bạn đang thêm rất nhiều phần tử vào một `List<T>` mà không chỉ định `Capacity` ban đầu, Antigravity có thể phân tích và đề xuất bạn khởi tạo `List<T>` với một `Capacity` ước tính, giúp giảm thiểu các thao tác cấp phát lại bộ nhớ tốn kém.
*   **Vibe Coding để thiết kế linh hoạt:** Khi bạn không chắc chắn về kích thước dữ liệu, hãy "vibe" rằng `List<T>` là lựa chọn mặc định. Antigravity có thể giúp bạn nhanh chóng triển khai các tính năng thêm/sửa/xóa mà không phải lo lắng về việc quản lý bộ nhớ cấp thấp.

## 4. Khi nào sử dụng Mảng và Danh sách? (So sánh chuyên sâu)

Việc lựa chọn giữa mảng và `List<T>` phụ thuộc vào yêu cầu cụ thể của ứng dụng. Dưới đây là bảng so sánh chi tiết và các hướng dẫn sử dụng:

| Đặc điểm / Trường hợp sử dụng | Mảng (Arrays)                                    | Danh sách (`List<T>`)                                                                |
| :---------------------------- | :----------------------------------------------- | :----------------------------------------------------------------------------------- |
| **Kích thước**                | Kích thước cố định, không thay đổi được sau khi khởi tạo. | Kích thước động, có thể thêm/bớt phần tử linh hoạt trong thời gian chạy.            |
| **Hiệu suất (thêm/xóa)**      | **Kém:** Để thêm/xóa, bạn phải tạo mảng mới và sao chép. Thường là O(N). | **Tốt (trung bình):** `Add` là O(1) trung bình (amortized). `Insert`/`Remove` ở giữa danh sách là O(N) do phải dịch chuyển phần tử. |
| **Hiệu suất (truy cập)**      | **Tuyệt vời:** Truy cập phần tử bằng chỉ số là O(1) (thời gian hằng số) do bộ nhớ liên tục. | **Tuyệt vời:** Truy cập phần tử bằng chỉ số là O(1) do sử dụng mảng nội bộ.         |
| **Bộ nhớ**                    | Sử dụng bộ nhớ hiệu quả hơn một chút nếu kích thước biết trước, không có chi phí overhead cho quản lý động. | Sử dụng nhiều bộ nhớ hơn một chút để quản lý kích thước động (mảng nội bộ có `Capacity` lớn hơn `Count`). |
| **Khởi tạo**                  | Đơn giản khi biết trước kích thước và giá trị.   | Đơn giản, linh hoạt, có thể khởi tạo rỗng hoặc với các giá trị ban đầu.             |
| **Các thao tác**              | Ít phương thức tích hợp sẵn, cần sử dụng `System.Array` cho các thao tác chung (Sort, Copy, IndexOf). | Nhiều phương thức tiện lợi để thêm, xóa, tìm kiếm, sắp xếp, v.v. (Add, Remove, Contains, Sort, Clear). |
| **Khuyến nghị sử dụng**       | 1. Khi bạn **chắc chắn** về số lượng phần tử và nó **không thay đổi** trong suốt vòng đời của mảng. <br> 2. Các tình huống yêu cầu hiệu suất cực cao và kiểm soát bộ nhớ chặt chẽ (ví dụ: xử lý hình ảnh, dữ liệu số lớn). <br> 3. Tương tác với các API hoặc thư viện cũ yêu cầu mảng. <br> 4. Tạo các mảng tạm thời với kích thước nhỏ, biết trước. | 1. Trong **hầu hết các trường hợp** khi bạn cần một bộ sưu tập linh hoạt, động. <br> 2. Khi bạn không biết trước số lượng phần tử cần lưu trữ. <br> 3. Khi bạn cần thường xuyên thêm hoặc xóa phần tử. <br> 4. Sử dụng làm kiểu dữ liệu mặc định cho các danh sách đối tượng. |

> [!TIP]
> Trong lập trình C# hiện đại, **`List<T>` là lựa chọn ưu tiên và phổ biến hơn nhiều** so với mảng (có thể lên đến 99% các trường hợp). `List<T>` cung cấp sự linh hoạt, an toàn kiểu và các tính năng tiện lợi mà mảng không có, với chi phí hiệu suất thường không đáng kể đối với hầu hết các ứng dụng.
>
> Tuy nhiên, bạn vẫn cần hiểu và biết cách làm việc với mảng, đặc biệt khi làm việc với các thư viện bên thứ ba, mã nguồn cũ (legacy code), hoặc các tình huống cực kỳ cần tối ưu hóa hiệu suất khi kích thước dữ liệu đã biết rõ và không thay đổi.

## 5. Tóm tắt Phần

*   **Mảng (Arrays)** là các cấu trúc dữ liệu có kích thước **cố định** dùng để lưu trữ các phần tử cùng kiểu. Chúng là kiểu tham chiếu, với dữ liệu mảng được cấp phát trên Heap.
*   C# hỗ trợ mảng một chiều, mảng hình chữ nhật (đa chiều, kích thước đều, bộ nhớ liên tục), và mảng jagged (đa chiều, mảng của các mảng, mỗi mảng con có thể có kích thước khác nhau).
*   Tất cả các mảng C# đều kế thừa từ lớp `System.Array`, cung cấp các phương thức tĩnh hữu ích như `IndexOf`, `Clear`, `Copy`, `Sort`, `Reverse`.
*   **Danh sách (`List<T>`)** là các cấu trúc dữ liệu **động**, có thể thay đổi kích thước linh hoạt, dùng để lưu trữ các phần tử cùng kiểu thông qua Generics (`<T>`). Chúng cũng là kiểu tham chiếu và quản lý một mảng nội bộ, tự động thay đổi kích thước khi cần thiết.
*   `List<T>` cung cấp nhiều phương thức tiện lợi như `Add`, `AddRange`, `Remove`, `IndexOf`, `Count`, `Clear`.
*   **Không được sửa đổi** một `List<T>` (thêm/xóa phần tử) khi đang lặp qua nó bằng vòng lặp `foreach` để tránh lỗi `InvalidOperationException`. Hãy sử dụng vòng lặp `for` lặp ngược hoặc các phương thức như `RemoveAll()`.
*   Trong hầu hết các tình huống thực tế, **`List<T>` là lựa chọn ưu tiên** do tính linh hoạt của nó. Mảng được sử dụng khi kích thước bộ sưu tập đã biết trước và không thay đổi, hoặc khi tối ưu hiệu suất là cực kỳ quan trọng.
*   **Tư duy Vibe Coding** giúp bạn cảm nhận được đặc tính cố định của mảng và sự linh hoạt của danh sách. **Antigravity IDE** là công cụ mạnh mẽ hỗ trợ bạn trong việc sinh mã, refactor, cảnh báo lỗi và tối ưu hóa khi làm việc với cả hai cấu trúc dữ liệu này, nâng cao năng suất và độ chính xác trong lập trình C#.

<!-- REVIEWED_BY_AGENT -->
