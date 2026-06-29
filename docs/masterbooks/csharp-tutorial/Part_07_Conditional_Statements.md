# Chương 7: Câu lệnh Điều kiện và Luồng Thực thi Chương trình

Trong lập trình, khả năng đưa ra "quyết định" là yếu tố then chốt để xây dựng các ứng dụng linh hoạt và thông minh. Các câu lệnh điều kiện trong C# chính là cơ chế cho phép chương trình của bạn đưa ra những quyết định này, điều khiển luồng thực thi dựa trên các điều kiện cụ thể. Từ việc xác định hành động dựa trên đầu vào của người dùng đến việc quản lý trạng thái phức tạp của hệ thống, câu lệnh điều kiện là nền tảng không thể thiếu.

Chương này sẽ đưa bạn đi sâu vào các cấu trúc điều kiện chính trong C#: `if-else`, toán tử điều kiện (ternary operator), và `switch-case`. Chúng ta sẽ không chỉ học cách sử dụng chúng mà còn hiểu rõ cơ chế hoạt động, các thực hành tốt nhất, và cách tư duy lập trình hiệu quả để viết mã sạch, dễ bảo trì, thậm chí liên hệ với cách các hệ thống AI như Antigravity IDE có thể hỗ trợ bạn trong quá trình này.

## I. Giới thiệu: Điều khiển Luồng Quyết định

Trong một chương trình máy tính, các dòng mã thường được thực thi tuần tự từ trên xuống dưới. Tuy nhiên, trong thực tế, chúng ta hiếm khi muốn chương trình chỉ làm một việc theo một trình tự cố định. Thay vào đó, chương trình cần phải:

*   **Phản ứng với dữ liệu đầu vào:** Nếu người dùng nhập "Admin", cấp quyền quản trị; nếu nhập "Guest", giới hạn quyền truy cập.
*   **Thích ứng với trạng thái hệ thống:** Nếu pin yếu, hiển thị cảnh báo; nếu kết nối mạng bị ngắt, chuyển sang chế độ offline.
*   **Xử lý các trường hợp ngoại lệ:** Nếu dữ liệu không hợp lệ, thông báo lỗi và yêu cầu nhập lại.

Các câu lệnh điều kiện cung cấp cơ chế để chương trình "đánh giá" một tình huống (thông qua một biểu thức logic) và sau đó "chọn" một con đường thực thi phù hợp. Trong C#, ba cấu trúc chính để đạt được điều này là:

1.  **`if-else`**: Cấu trúc linh hoạt nhất, cho phép thực thi một khối mã nếu một điều kiện là `true`, và các khối mã khác nếu điều kiện đó là `false` hoặc các điều kiện khác đúng.
2.  **Toán tử điều kiện (Ternary Operator)**: Một cách viết tắt gọn gàng cho câu lệnh `if-else` đơn giản, thường dùng để gán giá trị có điều kiện.
3.  **`switch-case`**: Được sử dụng để so sánh một biến hoặc biểu thức với nhiều giá trị hằng số rời rạc và thực thi các khối mã tương ứng.

Việc nắm vững các cấu trúc này không chỉ giúp bạn viết mã hoạt động mà còn giúp bạn "Vibe Coding" tốt hơn – tức là cảm nhận được luồng logic, mục đích và ý định của từng phần mã, điều này cực kỳ quan trọng khi bạn làm việc với các công cụ AI như Antigravity IDE để tự động hóa hoặc tối ưu hóa mã nguồn.

## II. Câu lệnh `if-else`: Nền tảng của Quyết định Logic

`if-else` là cấu trúc điều kiện cơ bản nhất và được sử dụng rộng rãi nhất trong C#. Nó cho phép chương trình thực hiện các hành động khác nhau dựa trên kết quả của một biểu thức Boolean.

### 2.1. Cấu trúc cơ bản và Biểu thức Boolean

Cấu trúc của `if-else` bao gồm:

```csharp
if (điều_kiện_1)
{
    // Khối mã này được thực thi nếu điều_kiện_1 là true
}
else if (điều_kiện_2)
{
    // Khối mã này được thực thi nếu điều_kiện_1 là false VÀ điều_kiện_2 là true
}
else
{
    // Khối mã này được thực thi nếu điều_kiện_1 VÀ điều_kiện_2 đều là false
}
```

*   **`điều_kiện`**: Phải là một **biểu thức Boolean**, tức là một biểu thức trả về giá trị `true` hoặc `false`.
*   **`if`**: Luôn là điểm bắt đầu. Nếu `điều_kiện_1` là `true`, khối mã bên trong `if` được thực thi, và toàn bộ cấu trúc `if-else if-else` kết thúc.
*   **`else if`**: Tùy chọn, có thể có nhiều `else if`. Chỉ được kiểm tra nếu tất cả các `điều_kiện` trước đó là `false`. Nếu `điều_kiện_2` là `true`, khối mã bên trong nó được thực thi và cấu trúc kết thúc.
*   **`else`**: Tùy chọn, khối mã này được thực thi nếu tất cả các `điều_kiện` trước đó (trong `if` và `else if`) đều là `false`.

**Về Biến kiểu Boolean (`bool`) và Kiểu Giá trị (Value Type):**

Khi bạn khai báo một biến `bool` (ví dụ: `bool isLoggedIn = true;`), biến này là một **kiểu giá trị (Value Type)**. Điều này có nghĩa là giá trị `true` hoặc `false` được lưu trữ trực tiếp trong vùng nhớ được cấp phát cho biến `isLoggedIn`.

*   **Bộ nhớ Stack:** Các biến kiểu giá trị cục bộ (được khai báo bên trong một phương thức) thường được cấp phát trên **ngăn xếp (Stack)**. Ngăn xếp là một vùng nhớ được quản lý tự động, hoạt động theo cơ chế LIFO (Last-In, First-Out), rất nhanh chóng cho việc cấp phát và giải phóng.
*   **Hiệu suất:** Vì giá trị được lưu trực tiếp, việc truy cập và thao tác với biến kiểu giá trị rất nhanh.
*   **Truyền tham số:** Khi một biến kiểu giá trị được truyền vào một phương thức, một *bản sao* của giá trị đó sẽ được tạo ra. Bất kỳ thay đổi nào đối với bản sao bên trong phương thức sẽ không ảnh hưởng đến biến gốc.

Việc hiểu rằng `bool`, `int`, `float`, `char`, `enum` là các kiểu giá trị là nền tảng quan trọng trong C# và lập trình hướng đối tượng, vì nó khác biệt cơ bản với các kiểu tham chiếu (ví dụ: các đối tượng, chuỗi) sẽ được học sau này.

### 2.2. Toán tử Logic: Kết hợp các Điều kiện

Để tạo ra các điều kiện phức tạp hơn, chúng ta sử dụng các toán tử logic trên các biểu thức Boolean:

*   **`&&` (AND - Phép toán "Và")**: Trả về `true` nếu **cả hai** điều kiện con đều `true`.
    *   Ví dụ: `(hour > 0 && hour < 12)` - `hour` phải lớn hơn 0 VÀ nhỏ hơn 12.
    *   **Nguyên tắc "Short-circuiting"**: Nếu điều kiện đầu tiên của `&&` là `false`, C# sẽ không đánh giá điều kiện thứ hai, vì kết quả cuối cùng chắc chắn là `false`. Điều này giúp tối ưu hiệu suất và ngăn chặn lỗi nếu điều kiện thứ hai phụ thuộc vào điều kiện thứ nhất (ví dụ: kiểm tra `null` trước khi truy cập thuộc tính).
*   **`||` (OR - Phép toán "Hoặc")**: Trả về `true` nếu **ít nhất một** trong hai điều kiện con là `true`.
    *   Ví dụ: `(isWeekend || isHoliday)` - là cuối tuần HOẶC là ngày lễ.
    *   **Nguyên tắc "Short-circuiting"**: Nếu điều kiện đầu tiên của `||` là `true`, C# sẽ không đánh giá điều kiện thứ hai, vì kết quả cuối cùng chắc chắn là `true`.
*   **`!` (NOT - Phép toán "Phủ định")**: Đảo ngược giá trị Boolean của một điều kiện. Nếu điều kiện là `true`, `!` biến nó thành `false`, và ngược lại.
    *   Ví dụ: `(!isLoggedIn)` - nếu KHÔNG đăng nhập.

### 2.3. Khối lệnh, Dấu ngoặc nhọn `{}` và Tính rõ ràng của mã

Trong C#, một khối mã được định nghĩa bởi cặp dấu ngoặc nhọn `{}`. Nếu bạn chỉ có một dòng mã sau `if`, `else if`, hoặc `else`, bạn *có thể* bỏ qua dấu ngoặc nhọn. Tuy nhiên, đây là một thực hành **không được khuyến nghị**.

```csharp
// KHÔNG NÊN làm như thế này, dù vẫn hợp lệ
if (hour < 12)
    Console.WriteLine("Chào buổi sáng!"); // Chỉ dòng này thuộc về if
    Console.WriteLine("Chúc một ngày tốt lành!"); // Dòng này LUÔN được thực thi, không phụ thuộc vào if!

// NÊN làm như thế này
if (hour < 12)
{
    Console.WriteLine("Chào buổi sáng!");
    Console.WriteLine("Chúc một ngày tốt lành!");
}
```

**Lý do nên luôn sử dụng dấu ngoặc nhọn:**

*   **Tránh lỗi logic:** Như ví dụ trên, việc thêm một dòng mã mới mà quên dấu ngoặc nhọn có thể thay đổi hoàn toàn logic của chương trình một cách không mong muốn và rất khó phát hiện.
*   **Tính rõ ràng:** Mã dễ đọc và dễ hiểu hơn, đặc biệt khi người khác đọc mã của bạn hoặc khi bạn xem lại mã sau một thời gian dài.
*   **Tính nhất quán:** Giữ phong cách mã hóa nhất quán trong toàn bộ dự án.

### 2.4. Lồng ghép Câu lệnh `if` và Chiến lược Tái cấu trúc

Bạn có thể đặt một câu lệnh `if-else` bên trong một câu lệnh `if-else` khác. Điều này được gọi là lồng ghép (nesting).

```csharp
if (isAdmin) // Điều kiện 1
{
    if (canEdit) // Điều kiện 2 (lồng ghép)
    {
        Console.WriteLine("Admin có quyền chỉnh sửa.");
    }
    else
    {
        Console.WriteLine("Admin không có quyền chỉnh sửa.");
    }
}
else
{
    Console.WriteLine("Người dùng không phải Admin.");
}
```

> [!CAUTION]
> **Tránh lồng ghép quá sâu!** Nhiều câu lệnh `if` lồng ghép sâu (quá 2-3 cấp) sẽ làm cho mã khó đọc, khó hiểu, khó bảo trì và khó kiểm thử. Đây là một "code smell" (mùi mã) cần tránh.

**Chiến lược Tái cấu trúc (Refactoring) để giảm lồng ghép:**

1.  **Sử dụng toán tử logic (`&&`):**
    ```csharp
    if (isAdmin && canEdit)
    {
        Console.WriteLine("Admin có quyền chỉnh sửa.");
    }
    else if (isAdmin && !canEdit) // Nếu là Admin nhưng không thể chỉnh sửa
    {
        Console.WriteLine("Admin không có quyền chỉnh sửa.");
    }
    else // Nếu không phải Admin
    {
        Console.WriteLine("Người dùng không phải Admin.");
    }
    ```
    Hoặc gọn hơn với tư duy "guard clause":
    ```csharp
    if (!isAdmin) // Guard clause: Thoát sớm nếu không phải admin
    {
        Console.WriteLine("Người dùng không phải Admin.");
        return; // hoặc thực hiện hành động khác và kết thúc phương thức
    }

    // Đến đây, chúng ta biết chắc chắn là Admin
    if (canEdit)
    {
        Console.WriteLine("Admin có quyền chỉnh sửa.");
    }
    else
    {
        Console.WriteLine("Admin không có quyền chỉnh sửa.");
    }
    ```
2.  **Tách thành các phương thức nhỏ hơn:** Chia nhỏ logic thành các hàm riêng biệt, mỗi hàm xử lý một phần nhỏ của quyết định.
3.  **Sử dụng `switch-case` (nếu phù hợp):** Nếu các điều kiện dựa trên cùng một biến với nhiều giá trị rời rạc.

**Antigravity IDE và Tái cấu trúc:**
Một hệ thống như Antigravity IDE, với khả năng hiểu "Vibe Coding" và lập kế hoạch tự động, có thể đóng vai trò quan trọng trong việc này. Khi bạn viết mã có `if` lồng ghép sâu, Antigravity có thể:

*   **Phát hiện "code smell":** Tự động cảnh báo về độ phức tạp của mã.
*   **Đề xuất tái cấu trúc:** Gợi ý các cách để làm phẳng cấu trúc `if` (ví dụ: chuyển đổi thành toán tử logic, guard clauses, hoặc thậm chí `switch` biểu thức hiện đại).
*   **Thực hiện tái cấu trúc tự động:** Với quyền hạn phù hợp, Antigravity có thể tự động áp dụng các mẫu tái cấu trúc đã học để cải thiện chất lượng mã của bạn, giúp bạn tập trung vào logic nghiệp vụ thay vì cấu trúc mã.

### 2.5. Ví dụ minh họa: Lời chào theo thời gian

```csharp
using System;

public class ConditionalStatements
{
    public static void Main(string[] args)
    {
        // Khai báo một biến kiểu int (kiểu giá trị) để lưu giờ hiện tại theo định dạng 24 giờ.
        // Biến 'hour' được lưu trữ trực tiếp trên Stack.
        int hour = 10; 

        Console.WriteLine($"--- Kiểm tra với giờ: {hour} ---");
        // Kiểm tra điều kiện để đưa ra lời chào phù hợp
        if (hour > 0 && hour < 12) // Nếu giờ lớn hơn 0 VÀ nhỏ hơn 12 (ví dụ: từ 1h sáng đến 11h sáng)
        {
            Console.WriteLine("Chào buổi sáng!"); 
        }
        else if (hour >= 12 && hour < 18) // Nếu không phải buổi sáng VÀ giờ lớn hơn hoặc bằng 12 VÀ nhỏ hơn 18 (ví dụ: từ 12h trưa đến 5h chiều)
        {
            Console.WriteLine("Chào buổi chiều!"); 
        }
        else // Nếu tất cả các điều kiện trên đều sai (ví dụ: từ 6h chiều đến 12h đêm, hoặc 0h)
        {
            Console.WriteLine("Chào buổi tối!"); 
        }

        // Thay đổi giá trị của hour để kiểm tra các trường hợp khác
        hour = 14; 
        Console.WriteLine($"\n--- Kiểm tra với giờ: {hour} ---");
        if (hour > 0 && hour < 12) 
        {
            Console.WriteLine("Chào buổi sáng!"); 
        }
        else if (hour >= 12 && hour < 18) 
        {
            Console.WriteLine("Chào buổi chiều!"); 
        }
        else 
        {
            Console.WriteLine("Chào buổi tối!"); 
        }
        
        hour = 20; 
        Console.WriteLine($"\n--- Kiểm tra với giờ: {hour} ---");
        if (hour > 0 && hour < 12) 
        {
            Console.WriteLine("Chào buổi sáng!"); 
        }
        else if (hour >= 12 && hour < 18) 
        {
            Console.WriteLine("Chào buổi chiều!"); 
        }
        else 
        {
            Console.WriteLine("Chào buổi tối!"); 
        }

        hour = 0; // Thử một trường hợp biên
        Console.WriteLine($"\n--- Kiểm tra với giờ: {hour} ---");
        if (hour > 0 && hour < 12) 
        {
            Console.WriteLine("Chào buổi sáng!"); 
        }
        else if (hour >= 12 && hour < 18) 
        {
            Console.WriteLine("Chào buổi chiều!"); 
        }
        else 
        {
            Console.WriteLine("Chào buổi tối!"); // Sẽ in ra "Chào buổi tối!"
        }
    }
}
```
**Kết quả đầu ra:**
```
--- Kiểm tra với giờ: 10 ---
Chào buổi sáng!

--- Kiểm tra với giờ: 14 ---
Chào buổi chiều!

--- Kiểm tra với giờ: 20 ---
Chào buổi tối!

--- Kiểm tra với giờ: 0 ---
Chào buổi tối!
```

## III. Toán tử Điều kiện (Ternary Operator): Quyết định Gọn gàng

Toán tử điều kiện, hay còn gọi là toán tử ba ngôi (ternary operator), là một cách viết tắt cho câu lệnh `if-else` khi bạn cần gán một giá trị cho một biến dựa trên một điều kiện đơn giản. Nó là một **biểu thức** (expression), không phải một câu lệnh (statement), nghĩa là nó luôn trả về một giá trị. Điều này giúp mã nguồn ngắn gọn và dễ đọc hơn đối với các trường hợp cụ thể.

### 3.1. Cú pháp và Ứng dụng

Cú pháp của toán tử điều kiện là:

```csharp
biến = (điều_kiện) ? giá_trị_nếu_đúng : giá_trị_nếu_sai;
```
*   `điều_kiện`: Một biểu thức Boolean.
*   `giá_trị_nếu_đúng`: Giá trị sẽ được gán cho `biến` nếu `điều_kiện` là `true`.
*   `giá_trị_nếu_sai`: Giá trị sẽ được gán cho `biến` nếu `điều_kiện` là `false`.

**Lưu ý quan trọng:** Hai giá trị (`giá_trị_nếu_đúng` và `giá_trị_nếu_sai`) phải có kiểu dữ liệu tương thích hoặc cùng kiểu dữ liệu, vì kết quả của biểu thức này sẽ được gán cho `biến`.

### 3.2. Ví dụ minh họa: Giá sản phẩm có điều kiện

```csharp
using System;

public class TernaryOperator
{
    public static void Main(string[] args)
    {
        // Khai báo một biến kiểu bool (kiểu giá trị) để xác định khách hàng vàng.
        // Biến 'isGoldCustomer' cũng được lưu trữ trực tiếp trên Stack.
        bool isGoldCustomer = true; 
        float price; // Khai báo biến price kiểu float (kiểu giá trị)

        Console.WriteLine($"--- Kiểm tra với isGoldCustomer = {isGoldCustomer} ---");
        // Sử dụng if-else truyền thống để gán giá
        if (isGoldCustomer)
        {
            price = 19.95f; // Giá cho khách hàng vàng
        }
        else
        {
            price = 29.95f; // Giá thông thường
        }
        Console.WriteLine($"Giá (if-else): ${price}");

        // Tái cấu trúc mã trên bằng cách sử dụng toán tử điều kiện
        // Nếu isGoldCustomer là true, gán 19.95f cho price; ngược lại, gán 29.95f.
        price = (isGoldCustomer) ? 19.95f : 29.95f;
        Console.WriteLine($"Giá (Ternary Operator): ${price}");

        // Thay đổi isGoldCustomer thành false để kiểm tra trường hợp ngược lại
        isGoldCustomer = false;
        Console.WriteLine($"\n--- Kiểm tra với isGoldCustomer = {isGoldCustomer} ---");
        price = (isGoldCustomer) ? 19.95f : 29.95f;
        Console.WriteLine($"Giá (Ternary Operator - không phải khách vàng): ${price}");

        // Một ví dụ khác: gán chuỗi thông báo
        string statusMessage = (isGoldCustomer) ? "Bạn là khách hàng vàng!" : "Bạn là khách hàng tiêu chuẩn.";
        Console.WriteLine($"\nThông báo: {statusMessage}");
    }
}
```
**Kết quả đầu ra:**
```
--- Kiểm tra với isGoldCustomer = True ---
Giá (if-else): $19.95
Giá (Ternary Operator): $19.95

--- Kiểm tra với isGoldCustomer = False ---
Giá (Ternary Operator - không phải khách vàng): $29.95

Thông báo: Bạn là khách hàng tiêu chuẩn.
```

> [!TIP]
> Toán tử điều kiện rất hữu ích cho các phép gán đơn giản. Tuy nhiên, đối với các logic phức tạp hơn hoặc khi bạn cần thực hiện nhiều hành động khác nhau (không chỉ gán giá trị), hãy ưu tiên `if-else` để giữ cho mã dễ đọc và bảo trì. Lạm dụng toán tử điều kiện cho logic phức tạp có thể làm giảm khả năng đọc của mã.

## IV. Câu lệnh `switch-case`: Phân loại và Xử lý Đa dạng

Câu lệnh `switch-case` cung cấp một cách hiệu quả để xử lý các tình huống mà bạn cần so sánh một biến hoặc biểu thức với nhiều giá trị hằng số rời rạc khác nhau. Nó thường giúp mã nguồn rõ ràng và có tổ chức hơn so với một chuỗi dài các câu lệnh `else if` khi bạn đang kiểm tra cùng một biến.

### 4.1. Cấu trúc cơ bản và Nguyên tắc `break`

Cú pháp của `switch-case` là:

```csharp
switch (biểu_thức_đánh_giá)
{
    case giá_trị_1:
        // Khối mã này được thực thi nếu biểu_thức_đánh_giá == giá_trị_1
        break; // Bắt buộc phải có break để thoát khỏi switch
    case giá_trị_2:
        // Khối mã này được thực thi nếu biểu_thức_đánh_giá == giá_trị_2
        break;
    // ... có thể có nhiều case khác
    default:
        // Khối mã này được thực thi nếu biểu_thức_đánh_giá không khớp với bất kỳ giá_trị nào
        break; // Nên có break ở default để nhất quán
}
```
*   `biểu_thức_đánh_giá`: Biến hoặc biểu thức sẽ được so sánh. Các kiểu dữ liệu được hỗ trợ bao gồm `char`, `string`, `bool`, các kiểu số nguyên (`int`, `byte`, `long`, v.v.), và `enum`.
*   `case giá_trị`: Một nhãn `case` định nghĩa một giá trị cụ thể để so sánh với `biểu_thức_đánh_giá`.
*   `break;`: Sau mỗi khối `case` (trừ khi có fall-through có chủ đích, xem phần sau), bạn **phải** sử dụng `break` để thoát khỏi câu lệnh `switch`. Nếu không có `break`, mã sẽ "rơi xuống" (fall-through) `case` tiếp theo, điều này thường gây ra lỗi logic và bị cấm bởi trình biên dịch C# (khác với C++ hoặc Java).
*   `default:`: Tùy chọn, được thực thi nếu `biểu_thức_đánh_giá` không khớp với bất kỳ `case` nào. Nó tương tự như khối `else` trong `if-else`.

### 4.2. `enum` (Kiểu liệt kê): Quản lý Các giá trị rời rạc hiệu quả

`enum` (enumeration - kiểu liệt kê) là một kiểu dữ liệu đặc biệt trong C# cho phép bạn định nghĩa một tập hợp các hằng số có tên. `enum` là một **kiểu giá trị (Value Type)**, tương tự như `int` hay `bool`. Mặc định, các thành phần của `enum` được gán giá trị số nguyên bắt đầu từ 0.

```csharp
// Định nghĩa một enum cho các mùa
public enum Season
{
    Spring, // Mặc định có giá trị 0
    Summer, // Mặc định có giá trị 1
    Autumn, // Mặc định có giá trị 2
    Winter  // Mặc định có giá trị 3
}
```
**Lợi ích của `enum` trong `switch-case`:**

*   **Tính đọc hiểu:** Thay vì so sánh với các số "ma thuật" (magic numbers) như `0`, `1`, `2`, `3`, bạn so sánh với `Season.Spring`, `Season.Summer`, v.v., giúp mã dễ hiểu hơn rất nhiều.
*   **An toàn kiểu dữ liệu:** Trình biên dịch sẽ kiểm tra xem bạn có đang sử dụng một giá trị `Season` hợp lệ hay không, giảm thiểu lỗi runtime.
*   **Dễ bảo trì:** Nếu bạn cần thêm một mùa mới, bạn chỉ cần thêm nó vào `enum`, và trình biên dịch sẽ cảnh báo bạn ở những nơi cần cập nhật `switch-case`.
*   **Bộ nhớ Stack:** Giống như các kiểu giá trị khác, một biến `enum` cục bộ cũng được lưu trữ trực tiếp trên ngăn xếp (Stack), mang lại hiệu suất cao.

### 4.3. Fall-through có chủ đích và Multi-case

C# yêu cầu mỗi `case` phải kết thúc bằng `break`, `return`, `goto`, hoặc `throw` để tránh fall-through không mong muốn. Tuy nhiên, bạn có thể tạo ra fall-through có chủ đích bằng cách đặt nhiều nhãn `case` liên tiếp mà không có mã giữa chúng. Điều này có nghĩa là nếu bất kỳ `case` nào trong số đó khớp, cùng một khối mã sẽ được thực thi.

```csharp
switch (season)
{
    case Season.Autumn:
    case Season.Summer: // Nếu là Autumn HOẶC Summer, thực thi khối mã này
        Console.WriteLine("Chúng tôi có khuyến mãi đặc biệt!");
        break;
    // ... các case khác
}
```

### 4.4. `switch` Biểu thức và Pattern Matching (C# hiện đại)

Với C# 8.0 trở lên, `switch` đã được mở rộng đáng kể với **`switch` biểu thức** (switch expression) và **pattern matching** (khớp mẫu). Điều này cho phép bạn viết các câu lệnh `switch` ngắn gọn hơn nhiều, đặc biệt khi mục tiêu là trả về một giá trị duy nhất dựa trên điều kiện, và cũng cho phép kiểm tra nhiều hơn là chỉ các giá trị hằng số.

**`switch` biểu thức:**
Thay vì một câu lệnh, `switch` có thể được sử dụng như một biểu thức để trả về một giá trị.

```csharp
// Ví dụ với enum Season
public enum Season { Spring, Summer, Autumn, Winter }

public static string GetSeasonMessage(Season season)
{
    string message = season switch
    {
        Season.Spring => "Mùa xuân - mùa của sự khởi đầu!",
        Season.Summer => "Chúng tôi có khuyến mãi đặc biệt!",
        Season.Autumn => "Mùa thu - lá vàng rơi!",
        Season.Winter => "Mùa đông - thời gian để nghỉ ngơi.",
        _ => "Mùa không xác định." // _ là discard pattern, tương đương với default
    };
    return message;
}
```
Đây là cách viết rất gọn gàng và thường được ưu tiên trong mã C# hiện đại khi bạn cần gán một giá trị có điều kiện.

**Pattern Matching (`is`, `when`):**
C# còn cho phép bạn sử dụng `switch` để khớp với các kiểu dữ liệu khác nhau, kiểm tra thuộc tính, hoặc thậm chí các điều kiện phức tạp hơn bằng cách sử dụng `is` và `when`.

```csharp
public static string DescribeObject(object obj)
{
    string description = obj switch
    {
        int i when i > 100 => $"Số nguyên lớn: {i}", // Khớp int VÀ có điều kiện i > 100
        int i => $"Số nguyên: {i}", // Khớp int
        string s when s.Length > 5 => $"Chuỗi dài: {s}", // Khớp string VÀ có điều kiện độ dài
        string s => $"Chuỗi ngắn: {s}", // Khớp string
        null => "Đối tượng null", // Khớp null
        _ => "Đối tượng không xác định" // Mọi thứ còn lại
    };
    return description;
}
```
**Antigravity IDE và `switch` hiện đại:**
Antigravity IDE, với khả năng phân tích mã và "Vibe Coding", sẽ nhận biết khi bạn có một chuỗi `if-else if` dài kiểm tra cùng một biến và có thể đề xuất chuyển đổi nó thành một `switch` biểu thức hoặc sử dụng pattern matching để làm cho mã gọn gàng và dễ đọc hơn. Nó giúp bạn áp dụng các tính năng ngôn ngữ hiện đại một cách tự nhiên, cải thiện chất lượng mã mà không cần bạn phải nhớ tất cả các cú pháp mới.

### 4.5. Ví dụ minh họa: Khuyến mãi theo mùa và Phân loại

```csharp
using System;

// Định nghĩa kiểu liệt kê (enum) Season
// Enum là một kiểu giá trị, giúp định nghĩa một tập hợp các hằng số có tên.
// Các biến kiểu Season cục bộ sẽ được lưu trữ trên Stack.
public enum Season
{
    Spring,
    Summer,
    Autumn,
    Winter,
    Unknown // Thêm một trường hợp để kiểm tra default
}

public class SwitchCaseExample
{
    public static void Main(string[] args)
    {
        // Khai báo một biến kiểu Season và gán giá trị
        var season = Season.Autumn; 

        Console.WriteLine($"\n--- Kiểm tra với mùa: {season} ---");
        // Sử dụng câu lệnh switch để kiểm tra giá trị của biến season
        switch (season)
        {
            case Season.Spring:
                Console.WriteLine("Mùa xuân - mùa của sự khởi đầu!");
                break; 

            case Season.Summer:
            case Season.Autumn: // Ví dụ về fall-through có chủ đích: nếu là Hè HOẶC Thu, thực thi cùng một khối mã
                Console.WriteLine("Chúng tôi có khuyến mãi đặc biệt!");
                Console.WriteLine("Thời tiết đẹp để đi chơi!");
                break; 

            case Season.Winter:
                Console.WriteLine("Mùa đông - thời gian để nghỉ ngơi.");
                break;

            default: // Nếu giá trị của season không khớp với bất kỳ case nào ở trên
                Console.WriteLine("Tôi không hiểu mùa này.");
                break;
        }

        // Kiểm tra với mùa Hè
        season = Season.Summer;
        Console.WriteLine($"\n--- Kiểm tra với mùa: {season} ---");
        switch (season)
        {
            case Season.Spring:
                Console.WriteLine("Mùa xuân - mùa của sự khởi đầu!");
                break;
            case Season.Summer:
            case Season.Autumn:
                Console.WriteLine("Chúng tôi có khuyến mãi đặc biệt!");
                Console.WriteLine("Thời tiết đẹp để đi chơi!");
                break;
            case Season.Winter:
                Console.WriteLine("Mùa đông - thời gian để nghỉ ngơi.");
                break;
            default:
                Console.WriteLine("Tôi không hiểu mùa này.");
                break;
        }

        // Kiểm tra với mùa Đông
        season = Season.Winter;
        Console.WriteLine($"\n--- Kiểm tra với mùa: {season} ---");
        switch (season)
        {
            case Season.Spring:
                Console.WriteLine("Mùa xuân - mùa của sự khởi đầu!");
                break;
            case Season.Summer:
            case Season.Autumn:
                Console.WriteLine("Chúng tôi có khuyến mãi đặc biệt!");
                Console.WriteLine("Thời tiết đẹp để đi chơi!");
                break;
            case Season.Winter:
                Console.WriteLine("Mùa đông - thời gian để nghỉ ngơi.");
                break;
            default:
                Console.WriteLine("Tôi không hiểu mùa này.");
                break;
        }

        // Kiểm tra với mùa không xác định
        season = Season.Unknown;
        Console.WriteLine($"\n--- Kiểm tra với mùa: {season} ---");
        switch (season)
        {
            case Season.Spring:
                Console.WriteLine("Mùa xuân - mùa của sự khởi đầu!");
                break;
            case Season.Summer:
            case Season.Autumn:
                Console.WriteLine("Chúng tôi có khuyến mãi đặc biệt!");
                Console.WriteLine("Thời tiết đẹp để đi chơi!");
                break;
            case Season.Winter:
                Console.WriteLine("Mùa đông - thời gian để nghỉ ngơi.");
                break;
            default:
                Console.WriteLine("Tôi không hiểu mùa này."); // Sẽ thực thi
                break;
        }

        // --- Sử dụng switch biểu thức (C# 8.0+) ---
        Console.WriteLine("\n--- Sử dụng switch biểu thức (C# 8.0+) ---");
        season = Season.Spring;
        string message = season switch
        {
            Season.Spring => "Mùa xuân đã đến!",
            Season.Summer => "Nắng vàng rực rỡ!",
            Season.Autumn => "Lá vàng rơi!",
            Season.Winter => "Tuyết trắng phủ đầy!",
            _ => "Mùa gì đây?" // Discard pattern, tương đương default
        };
        Console.WriteLine($"Thông báo cho mùa {season}: {message}");

        season = Season.Unknown;
        message = season switch
        {
            Season.Spring => "Mùa xuân đã đến!",
            Season.Summer => "Nắng vàng rực rỡ!",
            Season.Autumn => "Lá vàng rơi!",
            Season.Winter => "Tuyết trắng phủ đầy!",
            _ => "Mùa gì đây?"
        };
        Console.WriteLine($"Thông báo cho mùa {season}: {message}");
    }
}
```
**Kết quả đầu ra:**
```
--- Kiểm tra với mùa: Autumn ---
Chúng tôi có khuyến mãi đặc biệt!
Thời tiết đẹp để đi chơi!

--- Kiểm tra với mùa: Summer ---
Chúng tôi có khuyến mãi đặc biệt!
Thời tiết đẹp để đi chơi!

--- Kiểm tra với mùa: Winter ---
Mùa đông - thời gian để nghỉ ngơi.

--- Kiểm tra với mùa: Unknown ---
Tôi không hiểu mùa này.

--- Sử dụng switch biểu thức (C# 8.0+) ---
Thông báo cho mùa Spring: Mùa xuân đã đến!
Thông báo cho mùa Unknown: Mùa gì đây?
```

## V. Lựa chọn tối ưu: `if-else` vs. `switch-case` vs. Ternary Operator

Việc chọn cấu trúc điều kiện phù hợp là một phần quan trọng của "Vibe Coding" và viết mã chất lượng cao.

*   **Sử dụng `if-else` khi:**
    *   Bạn cần kiểm tra các điều kiện Boolean phức tạp, sử dụng nhiều toán tử logic (`&&`, `||`, `!`) để kết hợp các biểu thức khác nhau.
    *   Bạn cần kiểm tra các phạm vi giá trị (ví dụ: `x > 10 && x < 20`).
    *   Bạn đang so sánh các biến thuộc các kiểu dữ liệu khác nhau trong cùng một chuỗi điều kiện.
    *   Bạn có ít điều kiện cần kiểm tra (1-3 điều kiện).
    *   Mỗi nhánh điều kiện thực hiện một logic phức tạp hoặc nhiều hành động.

*   **Sử dụng `switch-case` khi:**
    *   Bạn cần so sánh một biến hoặc biểu thức duy nhất với nhiều giá trị hằng số rời rạc (số nguyên, `char`, `string`, `enum`).
    *   Khi có nhiều `else if` kiểm tra cùng một biến, `switch-case` thường mang lại mã nguồn dễ đọc và có cấu trúc hơn.
    *   Bạn muốn tận dụng tính năng fall-through có chủ đích cho các trường hợp cụ thể.
    *   Đặc biệt hiệu quả với `enum` để quản lý các lựa chọn có giới hạn.
    *   Với C# 8.0+, `switch` biểu thức và pattern matching cung cấp các cách mạnh mẽ và ngắn gọn hơn để xử lý các kịch bản này.

*   **Sử dụng Toán tử Điều kiện (Ternary Operator) khi:**
    *   Bạn cần gán một giá trị cho một biến dựa trên một điều kiện Boolean đơn giản.
    *   Mục tiêu là làm ngắn gọn mã nguồn cho các phép gán có điều kiện, giữ cho dòng mã đơn giản và dễ hiểu.
    *   Tránh sử dụng nó cho các biểu thức phức tạp hoặc khi cần thực hiện nhiều tác vụ phụ.

**Antigravity IDE và Hỗ trợ lựa chọn:**
Một hệ thống AI như Antigravity IDE không chỉ tuân theo các quy tắc này mà còn có thể hỗ trợ bạn trong việc đưa ra quyết định. Nó có thể:

*   **Phân tích ngữ cảnh:** Dựa trên kiểu dữ liệu của biến, số lượng điều kiện, và độ phức tạp của logic, Antigravity có thể đề xuất cấu trúc điều kiện tối ưu.
*   **Chuyển đổi mã:** Nếu bạn bắt đầu với một chuỗi `if-else if` dài, Antigravity có thể gợi ý và thậm chí tự động chuyển đổi nó thành một `switch-case` hoặc `switch` biểu thức gọn gàng hơn, giúp bạn "Vibe Coding" hiệu quả hơn với cấu trúc mã sạch.

## VI. Vibe Coding và Antigravity IDE: Tư duy Lập trình Điều kiện hiệu quả

"Vibe Coding" là một triết lý lập trình nhấn mạnh việc hiểu sâu sắc *ý định* và *luồng logic* của mã, thay vì chỉ tập trung vào cú pháp. Đối với các câu lệnh điều kiện, Vibe Coding có nghĩa là:

1.  **Hiểu rõ Quyết định:** Bạn đang cố gắng đưa ra quyết định gì? Các yếu tố nào ảnh hưởng đến quyết định đó?
2.  **Xác định Kết quả:** Với mỗi nhánh của quyết định, kết quả mong muốn là gì?
3.  **Dự đoán Lỗi:** Những trường hợp nào có thể xảy ra ngoài mong đợi? Làm thế nào để xử lý chúng?

**Antigravity IDE và Vibe Coding trong Điều kiện:**

Antigravity IDE, với khả năng Agentic AI của nó, được thiết kế để hỗ trợ Vibe Coding một cách mạnh mẽ, đặc biệt trong việc xử lý các câu lệnh điều kiện:

*   **Hiểu Ý Định (Intent Understanding):** Khi bạn mô tả một vấn đề hoặc bắt đầu viết một `if-else` hoặc `switch`, Antigravity không chỉ nhìn vào từ khóa mà còn cố gắng hiểu *mục đích* đằng sau logic điều kiện của bạn. Ví dụ, nếu bạn muốn "xác thực người dùng", nó sẽ biết rằng cần kiểm tra `username`, `password`, `isLockedOut`, v.v., và có thể gợi ý các điều kiện phù hợp.
*   **Tạo mã nguồn từ Pseudocode/Mô tả:** Bạn có thể cung cấp cho Antigravity một mô tả bằng ngôn ngữ tự nhiên hoặc pseudocode về các quyết định cần đưa ra. Antigravity sẽ tự động dịch chúng thành các câu lệnh `if-else`, `switch-case` hoặc toán tử điều kiện C# chuẩn, tuân thủ các thực hành tốt nhất.
    *   **Ví dụ:** "Nếu nhiệt độ dưới 0, báo 'Đóng băng'. Nếu từ 0 đến 10, báo 'Lạnh'. Nếu từ 11 đến 25, báo 'Mát mẻ'. Còn lại, báo 'Nóng'." Antigravity sẽ tạo ra một chuỗi `if-else if-else` hoặc `switch` biểu thức phù hợp.
*   **Tái cấu trúc và Tối ưu hóa Tự động:** Như đã đề cập, Antigravity có thể phát hiện các "code smell" trong các câu lệnh điều kiện (như lồng ghép sâu) và tự động đề xuất hoặc thực hiện tái cấu trúc để cải thiện tính đọc và hiệu suất. Nó có thể chuyển đổi một `if-else if` dài thành `switch-case` khi phù hợp, hoặc đơn giản hóa các điều kiện phức tạp.
*   **Kiểm thử Điều kiện (Implicit Testing):** Antigravity có thể tự động tạo ra các trường hợp kiểm thử cho các nhánh điều kiện khác nhau, giúp bạn đảm bảo rằng tất cả các luồng logic đã được bao phủ và hoạt động đúng như mong đợi. Điều này giảm thiểu lỗi và tăng cường độ tin cậy của mã.
*   **Giải thích Mã (Code Explanation):** Nếu bạn gặp một khối mã điều kiện phức tạp, Antigravity có thể giải thích từng phần của nó, bao gồm cách các điều kiện được đánh giá và tại sao một nhánh cụ thể được chọn. Điều này giúp bạn học hỏi và gỡ lỗi hiệu quả hơn.

Bằng cách áp dụng tư duy Vibe Coding và tận dụng sức mạnh của Antigravity IDE, bạn không chỉ viết được mã C# chính xác mà còn tạo ra mã chất lượng cao, dễ hiểu, dễ bảo trì và mở rộng.

## VII. Các Thực hành Tốt (Best Practices) cho Câu lệnh Điều kiện

Để viết mã điều kiện hiệu quả, dễ đọc và dễ bảo trì, hãy tuân thủ các nguyên tắc sau:

1.  **Luôn sử dụng dấu ngoặc nhọn `{}`:** Ngay cả khi chỉ có một dòng mã trong khối `if`, `else if`, `else`, hoặc `case`. Điều này ngăn chặn các lỗi tiềm ẩn khi bạn hoặc người khác thêm dòng mã sau này và cải thiện tính rõ ràng.
2.  **Tránh lồng ghép `if` quá sâu:** Nếu bạn thấy mình lồng ghép `if` quá 2-3 cấp, hãy xem xét tái cấu trúc mã.
    *   **Sử dụng Guard Clauses (Mệnh đề bảo vệ):** Kiểm tra các điều kiện lỗi hoặc trường hợp đặc biệt ở đầu một phương thức và thoát sớm (sử dụng `return`, `throw`) để làm phẳng logic chính.
    *   **Kết hợp điều kiện:** Sử dụng toán tử logic (`&&`, `||`) để kết hợp nhiều điều kiện vào một `if` duy nhất.
    *   **Tách phương thức:** Chia nhỏ logic phức tạp thành các phương thức nhỏ hơn, mỗi phương thức xử lý một phần trách nhiệm cụ thể.
3.  **Đảm bảo điều kiện rõ ràng và dễ hiểu:**
    *   Đặt tên biến và phương thức hợp lý, dễ hiểu.
    *   Viết các biểu thức điều kiện sao cho dễ đọc nhất có thể, tránh các phép toán phức tạp trực tiếp trong điều kiện.
    *   Tránh các giá trị "ma thuật" (magic numbers/strings) bằng cách sử dụng hằng số (`const`) hoặc `enum`.
4.  **Ưu tiên `switch-case` khi phù hợp:** Đối với việc kiểm tra một biến với nhiều giá trị hằng số rời rạc, `switch-case` (hoặc `switch` biểu thức trong C# hiện đại) thường cung cấp cấu trúc rõ ràng và dễ đọc hơn so với một chuỗi dài `else if`.
5.  **Sử dụng `enum` cho các giá trị rời rạc:** Khi làm việc với một tập hợp các lựa chọn cố định, `enum` kết hợp với `switch-case` là một cách tiếp cận mạnh mẽ, giúp tăng cường tính an toàn kiểu dữ liệu, tính đọc hiểu và dễ bảo trì.
6.  **Luôn bao gồm `default` trong `switch-case`:** Trừ khi bạn chắc chắn rằng tất cả các trường hợp có thể đã được xử lý bởi các `case` khác, việc có một `default` sẽ giúp bắt các giá trị không mong muốn và ngăn chặn các lỗi tiềm ẩn hoặc hành vi không xác định. Đối với `switch` biểu thức, sử dụng `_` (discard pattern) cho trường hợp mặc định.
7.  **Sử dụng toán tử điều kiện một cách có chọn lọc:** Chỉ dùng cho các phép gán đơn giản, một dòng. Tránh lạm dụng cho logic phức tạp.
8.  **Độ bao phủ kiểm thử:** Đảm bảo rằng các kiểm thử đơn vị (unit tests) của bạn bao phủ tất cả các nhánh điều kiện quan trọng để xác minh rằng mã hoạt động như mong đợi trong mọi tình huống.

## VIII. Tóm tắt Chương

*   **Câu lệnh điều kiện** là công cụ cơ bản để kiểm soát luồng chương trình, cho phép chương trình đưa ra quyết định dựa trên các biểu thức Boolean.
*   **`if-else`** là cấu trúc linh hoạt nhất, cho phép thực thi các khối mã khác nhau dựa trên các biểu thức Boolean, sử dụng **toán tử logic** (`&&`, `||`, `!`) để kết hợp hoặc sửa đổi các điều kiện.
*   **Kiểu giá trị (Value Type)** như `int`, `bool`, `enum` lưu trữ dữ liệu trực tiếp trên Stack, mang lại hiệu suất cao và được truyền theo giá trị (bản sao).
*   **Toán tử điều kiện (`? :`)** là một cú pháp viết tắt cho các phép gán có điều kiện đơn giản, giúp mã gọn gàng hơn.
*   **`switch-case`** được sử dụng để so sánh một biến hoặc biểu thức với nhiều giá trị hằng số rời rạc, thường rõ ràng hơn `if-else if` khi có nhiều lựa chọn.
*   **`enum`** (kiểu giá trị) rất hữu ích khi làm việc với `switch-case` để định nghĩa các tập hợp hằng số có tên, cải thiện tính đọc và an toàn của mã.
*   **`break`** là bắt buộc trong mỗi `case` của `switch` (trừ fall-through có chủ đích) để ngăn chặn việc rơi xuống `case` tiếp theo.
*   **`switch` biểu thức và Pattern Matching** (C# 8.0+) mở rộng khả năng của `switch`, cho phép viết mã điều kiện gọn gàng và mạnh mẽ hơn.
*   **Vibe Coding** và sự hỗ trợ của các công cụ AI như **Antigravity IDE** giúp bạn hiểu sâu sắc ý định của mã, tối ưu hóa cấu trúc điều kiện và tự động tái cấu trúc để duy trì mã sạch và dễ bảo trì.
*   **Thực hành tốt nhất** bao gồm việc sử dụng dấu ngoặc nhọn nhất quán, tránh lồng ghép quá sâu, và chọn cấu trúc điều kiện phù hợp nhất cho từng tình huống để duy trì mã sạch và dễ bảo trì.

<!-- REVIEWED_BY_AGENT -->
