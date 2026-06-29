# Phần 4: Lớp và Cấu trúc: Tổ chức Dữ liệu và Hành vi trong C#

Trong thế giới lập trình hiện đại, việc xây dựng các ứng dụng phức tạp đòi hỏi nhiều hơn là chỉ sử dụng các kiểu dữ liệu cơ bản như số nguyên hay chuỗi. Chúng ta cần những công cụ mạnh mẽ để mô hình hóa các thực thể trong thế giới thực, tổ chức dữ liệu một cách logic và định nghĩa các hành vi liên quan. Phần này sẽ giới thiệu hai khối xây dựng cơ bản và quan trọng nhất của C# và lập trình hướng đối tượng (OOP): **Lớp (Classes)** và **Cấu trúc (Structs)**.

Bạn sẽ học cách định nghĩa các kiểu dữ liệu tùy chỉnh, tạo ra các "thực thể sống" từ chúng, và quan trọng nhất là nắm bắt sự khác biệt cơ bản giữa chúng về mặt quản lý bộ nhớ – một khái niệm cốt lõi trong C# được gọi là **Kiểu Giá trị (Value Types)** và **Kiểu Tham chiếu (Reference Types)**. Việc hiểu sâu sắc những khái niệm này là chìa khóa để viết mã nguồn không chỉ hiệu quả, dễ bảo trì mà còn vững chắc, tránh được các lỗi liên quan đến bộ nhớ và hành vi không mong muốn.

---

## 1. Giới thiệu: Nhu cầu về các Kiểu Dữ liệu Tùy chỉnh

Hãy tưởng tượng bạn đang xây dựng một ứng dụng quản lý thư viện. Bạn cần lưu trữ thông tin về sách. Nếu chỉ dùng các kiểu dữ liệu nguyên thủy, bạn sẽ có các biến rời rạc như `string tenSach`, `string tacGia`, `int namXuatBan`, `bool daMuon`. Cách này nhanh chóng trở nên cồng kềnh khi bạn có nhiều sách hoặc cần thêm các hành vi như `MuonSach()`, `TraSach()`.

Đây chính là lúc Lớp và Cấu trúc phát huy tác dụng. Chúng cho phép bạn gói gọn tất cả các dữ liệu và hành vi liên quan đến một "Sách" vào một đơn vị logic duy nhất, tạo ra một kiểu dữ liệu mới, tùy chỉnh theo nhu cầu của bạn.

---

```mermaid
flowchart TB
    subgraph CLASS["🔵 Class (Kiểu Tham chiếu)"]
        direction TB
        c1["✅ Hỗ trợ kế thừa"]
        c2["📦 Lưu trên Heap"]
        c3["🔗 Gán = sao chép tham chiếu"]
        c4["🗑️ GC thu gom bộ nhớ"]
        c5["🏗️ Có Destructor/Finalizer"]
    end
    subgraph STRUCT["🟢 Struct (Kiểu Giá trị)"]
        direction TB
        s1["❌ Không kế thừa"]
        s2["📚 Lưu trên Stack"]
        s3["📋 Gán = sao chép toàn bộ giá trị"]
        s4["⚡ Tự giải phóng khi ra scope"]
        s5["🎯 Tối ưu cho dữ liệu nhỏ"]
    end
    style CLASS fill:#e3f2fd,color:#000
    style STRUCT fill:#e8f5e9,color:#000
```
*So sánh tổng quan: Class là kiểu tham chiếu (Heap), Struct là kiểu giá trị (Stack). Chọn Class cho đối tượng phức tạp, Struct cho dữ liệu nhỏ và bất biến.*

```mermaid
flowchart TB
    subgraph CLASS["🔵 Class (Kiểu Tham chiếu)"]
        direction TB
        c1["✅ Hỗ trợ kế thừa"]
        c2["📦 Lưu trên Heap"]
        c3["🔗 Gán = sao chép tham chiếu"]
        c4["🗑️ GC thu gom bộ nhớ"]
        c5["🏗️ Có Destructor/Finalizer"]
    end
    subgraph STRUCT["🟢 Struct (Kiểu Giá trị)"]
        direction TB
        s1["❌ Không kế thừa"]
        s2["📚 Lưu trên Stack"]
        s3["📋 Gán = sao chép toàn bộ giá trị"]
        s4["⚡ Tự giải phóng khi ra scope"]
        s5["🎯 Tối ưu cho dữ liệu nhỏ"]
    end
    style CLASS fill:#e3f2fd,color:#000
    style STRUCT fill:#e8f5e9,color:#000
```
*So sánh tổng quan: Class là kiểu tham chiếu (Heap), Struct là kiểu giá trị (Stack). Chọn Class cho đối tượng phức tạp, Struct cho dữ liệu nhỏ và bất biến.*


## 2. Lớp (Classes) - Nền tảng của Lập trình Hướng đối tượng

Trong C#, các lớp là trái tim của lập trình hướng đối tượng. Chúng cung cấp một cơ chế mạnh mẽ để mô tả các thực thể trong thế giới thực hoặc các khái niệm trừu tượng bằng cách kết hợp dữ liệu (trạng thái) và chức năng (hành vi) có liên quan chặt chẽ thành một đơn vị duy nhất.

### 2.1. Khái niệm: Lớp là Bản thiết kế, Đối tượng là Thực thể

Để dễ hình dung, hãy nghĩ về lớp và đối tượng như bản thiết kế và sản phẩm thực tế:

*   **Lớp (Class):** Là một **bản thiết kế**, một **khuôn mẫu**, hoặc một **kiểu dữ liệu tùy chỉnh**. Nó định nghĩa các đặc điểm chung (dữ liệu) và hành vi chung (chức năng) mà tất cả các thực thể được tạo ra từ bản thiết kế này sẽ có.
    *   **Ví dụ:** Bản thiết kế "Xe Hơi" định nghĩa rằng mọi chiếc xe hơi đều có `Màu`, `Số bánh`, `Tốc độ tối đa` và có thể `Di chuyển`, `Phanh`, `Bấm còi`.
*   **Đối tượng (Object):** Là một **thể hiện (instance)** cụ thể được tạo ra từ một lớp. Mỗi đối tượng là một thực thể độc lập, có các giá trị riêng cho các đặc điểm được định nghĩa trong lớp và có thể thực hiện các hành vi đó.
    *   **Ví dụ:** Từ bản thiết kế "Xe Hơi", chúng ta có thể tạo ra các đối tượng như `myCar` (màu đỏ, 4 bánh, chạy 100km/h) và `yourCar` (màu xanh, 4 bánh, chạy 120km/h). Mỗi đối tượng này là một chiếc xe hơi riêng biệt với các thuộc tính và hành vi riêng.

> [!NOTE]
> Mặc dù trong giao tiếp hàng ngày, "lớp" và "đối tượng" đôi khi được dùng thay thế cho nhau, nhưng về mặt kỹ thuật, việc phân biệt chúng là rất quan trọng. Lớp là ý tưởng trừu tượng, đối tượng là hiện thực cụ thể của ý tưởng đó.

#### Antigravity IDE & Vibe Coding Insights:
Khi Antigravity IDE được yêu cầu tạo một hệ thống, nó sẽ bắt đầu bằng cách "cảm nhận" (vibe) các thực thể chính. Nếu bạn nói "Quản lý thư viện", Antigravity sẽ tự động nhận diện "Sách", "Độc giả", "Thủ thư" là những ứng cử viên sáng giá cho các lớp. Nó sẽ phác thảo các đặc điểm (tên, tuổi, tác giả) và hành vi (mượn, trả, tìm kiếm) cho mỗi lớp, giống như một kiến trúc sư phác thảo bản vẽ ban đầu.

### 2.2. Định nghĩa Lớp và các Thành viên

Để tạo một lớp trong C#, chúng ta sử dụng từ khóa `class`.

#### 2.2.1. Cú pháp khai báo Lớp

```csharp
// Cú pháp cơ bản của một lớp
public class TenLop
{
    // Các thành viên của lớp:
    // - Trường (Fields): Lưu trữ dữ liệu/trạng thái
    // - Thuộc tính (Properties): Cung cấp quyền truy cập có kiểm soát đến dữ liệu
    // - Phương thức (Methods): Định nghĩa hành vi/chức năng
    // - Hàm tạo (Constructors): Khởi tạo đối tượng
    // - ... và nhiều thành viên khác
}
```

#### 2.2.2. Các Thành viên của Lớp: Trường (Fields) và Phương thức (Methods)

Thành viên của một lớp bao gồm dữ liệu và các hàm thực hiện hành động:

*   **Trường (Fields):** Là các biến lưu trữ dữ liệu hoặc trạng thái của đối tượng. Chúng thường được khai báo là `private` để đảm bảo tính đóng gói (encapsulation).

    ```csharp
    public class Person
    {
        public string TenDau;  // Trường để lưu tên đầu
        public string TenCuoi; // Trường để lưu tên cuối
        public int Tuoi;       // Trường để lưu tuổi
    }
    ```
    > [!NOTE]
    > Khi một trường không được gán giá trị khởi tạo, C# sẽ tự động gán giá trị mặc định: `0` cho kiểu số, `false` cho `bool`, và `null` cho kiểu tham chiếu (`string` là một kiểu tham chiếu).

*   **Phương thức (Methods):** Là các hàm (chức năng) thực hiện một hành động hoặc một phép tính nào đó.

    ```csharp
    public class Person
    {
        public string TenDau;
        public string TenCuoi;
        public int Tuoi;

        // Phương thức không trả về giá trị (void) và không nhận tham số
        public void GioiThieu()
        {
            // Từ khóa 'this' tham chiếu đến đối tượng hiện tại
            Console.WriteLine($"Tên tôi là {this.TenDau} {this.TenCuoi}. Tôi {this.Tuoi} tuổi.");
        }

        // Phương thức trả về một giá trị (int) và nhận hai tham số
        public int TinhTuoiSauSoNam(int soNam)
        {
            return this.Tuoi + soNam;
        }
    }
    ```
    *   `void`: Là kiểu trả về của phương thức. `void` có nghĩa là phương thức này không trả về bất kỳ giá trị nào.
    *   Dấu ngoặc đơn `()` sau tên phương thức: Chứa danh sách các tham số mà phương thức nhận vào (nếu có).
    *   `this`: Là một từ khóa đặc biệt trong C# dùng để tham chiếu đến thể hiện hiện tại của lớp. Nó giúp phân biệt giữa các biến cục bộ hoặc tham số với các trường của lớp khi chúng có cùng tên, hoặc đơn giản là để chỉ rõ bạn đang truy cập một thành viên của đối tượng hiện tại.

#### 2.2.3. Công cụ sửa đổi quyền truy cập (Access Modifiers)

Các công cụ sửa đổi quyền truy cập (`public`, `private`, `protected`, `internal`, v.v.) xác định phạm vi mà lớp hoặc thành viên của lớp có thể được truy cập.

*   **`public`:** Thành viên có thể được truy cập từ bất kỳ đâu. Thường dùng cho các phương thức và thuộc tính mà bạn muốn công khai.
*   **`private`:** Thành viên chỉ có thể được truy cập từ bên trong cùng một lớp. Đây là công cụ sửa đổi mặc định nếu bạn không chỉ định gì, và thường được dùng cho các trường để bảo vệ dữ liệu.
*   **`protected`:** Thành viên có thể được truy cập từ bên trong cùng một lớp hoặc từ các lớp con kế thừa từ nó. (Sẽ được tìm hiểu sâu hơn trong phần về Kế thừa).

> [!TIP]
> Trong khóa học cơ bản này, chúng ta sẽ chủ yếu sử dụng `public` cho các thành viên để dễ dàng truy cập và minh họa. Tuy nhiên, trong thực tế, `private` là lựa chọn ưu tiên cho các trường để đảm bảo tính đóng gói, và các thuộc tính (properties) sẽ được sử dụng để cung cấp quyền truy cập có kiểm soát.

### 2.3. Tạo và Sử dụng Đối tượng: Khởi tạo và Tương tác

Để sử dụng một lớp, chúng ta cần tạo một đối tượng (thể hiện) của lớp đó.

#### 2.3.1. Toán tử `new` và Hàm tạo (Constructors)

1.  **Khai báo biến kiểu lớp:** `Person john;`
2.  **Cấp phát bộ nhớ và Khởi tạo đối tượng:** Trong C#, các lớp là **kiểu tham chiếu**. Điều này có nghĩa là chúng ta cần cấp phát bộ nhớ một cách rõ ràng cho đối tượng trên **Heap** bằng toán tử `new`. Toán tử `new` không chỉ cấp phát bộ nhớ mà còn gọi một **hàm tạo (constructor)** để khởi tạo đối tượng.

    *   **Hàm tạo (Constructor):** Là một phương thức đặc biệt được gọi tự động khi một đối tượng của lớp được tạo ra. Nó có cùng tên với lớp và không có kiểu trả về (kể cả `void`). Mục đích chính là để khởi tạo trạng thái ban đầu của đối tượng.

        ```csharp
        public class Person
        {
            public string TenDau;
            public string TenCuoi;
            public int Tuoi;

            // Hàm tạo mặc định (không tham số) - C# tự tạo nếu không có hàm tạo nào khác
            public Person()
            {
                // Có thể đặt các giá trị mặc định ở đây
                TenDau = "Chưa biết";
                TenCuoi = "Chưa biết";
                Tuoi = 0;
                Console.WriteLine("Một đối tượng Person đã được tạo!");
            }

            // Hàm tạo có tham số
            public Person(string tenDau, string tenCuoi, int tuoi)
            {
                TenDau = tenDau;
                TenCuoi = tenCuoi;
                Tuoi = tuoi;
                Console.WriteLine($"Đối tượng {tenDau} {tenCuoi} đã được tạo với tuổi {tuoi}!");
            }

            public void GioiThieu()
            {
                Console.WriteLine($"Tên tôi là {TenDau} {TenCuoi}. Tôi {Tuoi} tuổi.");
            }
        }
        ```

#### 2.3.2. Truy cập thành viên và Gọi phương thức

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        // 1. Tạo một đối tượng Person bằng hàm tạo mặc định
        Person john = new Person(); // Gọi hàm tạo Person()
        john.TenDau = "John";
        john.TenCuoi = "Smith";
        john.Tuoi = 30;
        john.GioiThieu(); // Output: Tên tôi là John Smith. Tôi 30 tuổi.

        // 2. Tạo một đối tượng Person bằng hàm tạo có tham số
        Person mary = new Person("Mary", "Jane", 25); // Gọi hàm tạo Person(string, string, int)
        mary.GioiThieu(); // Output: Tên tôi là Mary Jane. Tôi 25 tuổi.

        int tuoiSau5Nam = mary.TinhTuoiSauSoNam(5);
        Console.WriteLine($"{mary.TenDau} sẽ {tuoiSau5Nam} tuổi sau 5 năm."); // Output: Mary sẽ 30 tuổi sau 5 năm.

        // Sử dụng 'var' để C# tự suy luận kiểu (thường dùng trong thực tế)
        var scott = new Person("Scott", "Tiger", 40);
        scott.GioiThieu();
    }
}
```

> [!IMPORTANT]
> Không giống như các ngôn ngữ như C++ nơi bạn phải tự giải phóng bộ nhớ, C# có một cơ chế gọi là **Thu gom rác (Garbage Collection - GC)**. CLR (Common Language Runtime) sẽ tự động theo dõi và giải phóng bộ nhớ của các đối tượng trên Heap không còn được sử dụng, giúp lập trình viên không phải lo lắng về việc quản lý bộ nhớ thủ công.

#### Antigravity IDE & Vibe Coding Insights:
Khi bạn yêu cầu Antigravity tạo một lớp `Person`, nó không chỉ tạo ra các trường mà còn tự động gợi ý hoặc tạo các hàm tạo khác nhau (mặc định, có tham số) để giúp bạn dễ dàng khởi tạo đối tượng. Nó "hiểu" rằng việc khởi tạo là một phần thiết yếu của vòng đời đối tượng. Nếu bạn thay đổi một trường từ `public` sang `private`, Antigravity sẽ tự động gợi ý tạo một `Property` tương ứng để duy trì quyền truy cập có kiểm soát, thể hiện tư duy về tính đóng gói.

### 2.4. Thành viên Tĩnh (Static Members): Thuộc về Lớp, không phải Đối tượng

Một thành viên (trường, phương thức, thuộc tính, hàm tạo) được đánh dấu bằng từ khóa `static` được gọi là thành viên tĩnh.

*   **Thuộc về Lớp:** Thành viên tĩnh thuộc về chính lớp đó, không phải bất kỳ đối tượng cụ thể nào của lớp. Bạn không cần tạo đối tượng để truy cập chúng.
*   **Một phiên bản duy nhất:** Chỉ có một phiên bản của thành viên tĩnh tồn tại trong bộ nhớ, bất kể bạn tạo bao nhiêu đối tượng của lớp đó (hoặc không tạo đối tượng nào).
*   **Truy cập trực tiếp:** Bạn truy cập thành viên tĩnh trực tiếp thông qua tên lớp, không cần tạo đối tượng.

```csharp
public class Calculator
{
    // Trường tĩnh: đếm số phép cộng đã thực hiện bởi TẤT CẢ các đối tượng hoặc các lần gọi tĩnh.
    public static int SoPhepCongDaThucHien = 0;

    // Phương thức tĩnh: thực hiện phép cộng
    public static int Cong(int a, int b)
    {
        SoPhepCongDaThucHien++; // Tăng biến đếm mỗi khi phương thức tĩnh này được gọi
        return a + b;
    }

    // Phương thức không tĩnh (instance method): cần đối tượng để gọi
    public int Tru(int a, int b)
    {
        return a - b;
    }
}
```

**Cách sử dụng thành viên tĩnh:**

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        // Gọi phương thức tĩnh trực tiếp từ tên lớp
        int ketQua1 = Calculator.Cong(10, 20);
        Console.WriteLine($"Kết quả 1: {ketQua1}"); // Output: Kết quả 1: 30

        int ketQua2 = Calculator.Cong(5, 7);
        Console.WriteLine($"Kết quả 2: {ketQua2}"); // Output: Kết quả 2: 12

        // Truy cập trường tĩnh trực tiếp từ tên lớp
        Console.WriteLine($"Tổng số phép cộng đã thực hiện: {Calculator.SoPhepCongDaThucHien}"); // Output: Tổng số phép cộng đã thực hiện: 2

        // Để gọi phương thức không tĩnh, cần tạo đối tượng
        Calculator myCalc = new Calculator();
        int ketQuaTru = myCalc.Tru(20, 5);
        Console.WriteLine($"Kết quả trừ: {ketQuaTru}"); // Output: Kết quả trừ: 15

        // KHÔNG THỂ truy cập thành viên tĩnh từ đối tượng
        // myCalc.Cong(1, 1); // Lỗi biên dịch: "An object reference is required for the non-static field, method, or property 'Calculator.Cong(int, int)'"
    }
}
```

**Khi nào sử dụng thành viên tĩnh?**

*   Khi bạn muốn biểu diễn một khái niệm mà chỉ có một phiên bản duy nhất tồn tại trong bộ nhớ và thuộc về chính lớp đó, không phải các đối tượng cụ thể.
*   **Ví dụ phổ biến:**
    *   Phương thức `Main` trong lớp `Program` của mọi ứng dụng C# đều là tĩnh, vì nó là điểm khởi đầu duy nhất của chương trình.
    *   Lớp `Console` với các phương thức như `WriteLine`, `ReadLine` là các phương thức tĩnh, vì chỉ có một bảng điều khiển duy nhất cho ứng dụng để tương tác.
    *   `Math.PI` (trường tĩnh) hoặc `Math.Sqrt()` (phương thức tĩnh): Các hằng số và hàm toán học không cần một "đối tượng Math" để hoạt động.
    *   `DateTime.Now`: Trả về thời điểm hiện tại. Bạn không cần tạo một đối tượng `DateTime` để hỏi "bây giờ là mấy giờ?".

### 2.5. Encapsulation (Tính đóng gói) và Thuộc tính (Properties)

Encapsulation (Tính đóng gói) là một trong bốn trụ cột của OOP. Nó liên quan đến việc ẩn dữ liệu bên trong một lớp và chỉ cung cấp một giao diện công khai để truy cập hoặc sửa đổi dữ liệu đó một cách có kiểm soát.

#### 2.5.1. Tại sao cần Encapsulation?

Giả sử bạn có lớp `Person` với trường `Tuoi` (tuổi) là `public`.

```csharp
public class Person
{
    public string TenDau;
    public string TenCuoi;
    public int Tuoi; // Public field

    // ...
}

// Trong Main:
Person john = new Person();
john.Tuoi = -5; // KHÔNG HỢP LỆ! Tuổi không thể âm.
```
Việc cho phép truy cập trực tiếp vào các trường `public` có thể dẫn đến trạng thái dữ liệu không hợp lệ. Encapsulation giúp ngăn chặn điều này.

#### 2.5.2. Giới thiệu Properties (Thuộc tính)

Trong C#, **thuộc tính (properties)** là một cách để triển khai tính đóng gói. Chúng trông giống như các trường nhưng hoạt động giống như các phương thức, cho phép bạn đọc (`get`) và/hoặc ghi (`set`) giá trị của một trường một cách có kiểm soát.

```csharp
public class Person
{
    private string _tenDau; // Trường private (backing field)
    private string _tenCuoi;
    private int _tuoi;

    // Thuộc tính TenDau: chỉ đọc (read-only)
    public string TenDau
    {
        get { return _tenDau; }
        // set { _tenDau = value; } // Không có setter thì là read-only
    }

    // Thuộc tính TenCuoi: đọc và ghi với logic kiểm tra
    public string TenCuoi
    {
        get { return _tenCuoi; }
        set
        {
            if (string.IsNullOrWhiteSpace(value))
            {
                Console.WriteLine("Tên cuối không được để trống!");
                // Hoặc ném ra ngoại lệ: throw new ArgumentException("Tên cuối không được để trống.");
            }
            else
            {
                _tenCuoi = value;
            }
        }
    }

    // Thuộc tính Tuoi: đọc và ghi với logic kiểm tra
    public int Tuoi
    {
        get { return _tuoi; }
        set
        {
            if (value >= 0 && value <= 150) // Giả sử tuổi hợp lệ từ 0 đến 150
            {
                _tuoi = value;
            }
            else
            {
                Console.WriteLine("Tuổi không hợp lệ!");
            }
        }
    }

    // Auto-implemented Property (Thuộc tính tự động):
    // C# tự động tạo backing field private cho bạn. Thường dùng khi không cần logic đặc biệt.
    public string Email { get; set; }

    public Person(string tenDau, string tenCuoi, int tuoi)
    {
        // Gán qua properties để kích hoạt logic kiểm tra (nếu có)
        TenDau = tenDau; // Lưu ý: Nếu TenDau chỉ có get, phải gán vào _tenDau trực tiếp hoặc qua constructor khác
        _tenDau = tenDau; // Gán trực tiếp vào backing field vì TenDau chỉ có get
        TenCuoi = tenCuoi;
        Tuoi = tuoi;
    }

    public void GioiThieu()
    {
        Console.WriteLine($"Tên tôi là {TenDau} {TenCuoi}. Tôi {Tuoi} tuổi. Email: {Email}");
    }
}

// Trong Main:
public class Program
{
    public static void Main(string[] args)
    {
        Person alice = new Person("Alice", "Wonderland", 28);
        alice.Email = "alice@example.com";
        alice.GioiThieu(); // Output: Tên tôi là Alice Wonderland. Tôi 28 tuổi. Email: alice@example.com

        alice.Tuoi = -10; // Output: Tuổi không hợp lệ! (Giá trị tuổi vẫn là 28)
        alice.TenCuoi = ""; // Output: Tên cuối không được để trống! (Giá trị tên cuối vẫn là "Wonderland")

        Console.WriteLine($"Tuổi của Alice: {alice.Tuoi}"); // Output: Tuổi của Alice: 28
    }
}
```
Properties là cách chuẩn để quản lý dữ liệu trong các lớp C#, thúc đẩy tính đóng gói và làm cho mã nguồn an toàn, dễ bảo trì hơn.

### 2.6. Tổ chức mã nguồn: Namespaces và Tệp

Trong các ứng dụng thực tế, bạn sẽ có hàng chục, thậm chí hàng nghìn lớp. Việc đặt tất cả các lớp vào một tệp duy nhất sẽ khiến mã nguồn trở nên lộn xộn và khó quản lý. C# cung cấp các cơ chế để tổ chức mã nguồn một cách hiệu quả:

*   **Mỗi lớp một tệp:** Theo quy ước, mỗi lớp C# nên được đặt trong một tệp `.cs` riêng biệt, với tên tệp trùng với tên lớp (ví dụ: `Person.cs` cho lớp `Person`). Điều này giúp dễ dàng tìm kiếm, quản lý và cộng tác.
*   **Namespaces (Không gian tên):** Là một cách để nhóm các lớp, cấu trúc, giao diện và các kiểu khác có liên quan với nhau. Namespaces giúp tránh xung đột tên khi bạn sử dụng các thư viện khác nhau hoặc làm việc trong các dự án lớn.

    ```csharp
    // Tệp: Models/Person.cs
    namespace MyAwesomeApp.Models // Định nghĩa namespace cho lớp Person
    {
        public class Person
        {
            public string TenDau { get; set; }
            public string TenCuoi { get; set; }
            public int Tuoi { get; set; }

            public Person(string tenDau, string tenCuoi, int tuoi)
            {
                TenDau = tenDau;
                TenCuoi = tenCuoi;
                Tuoi = tuoi;
            }
            public void GioiThieu()
            {
                Console.WriteLine($"Tên tôi là {TenDau} {TenCuoi}.");
            }
        }
    }

    // Tệp: Services/Calculator.cs
    namespace MyAwesomeApp.Services // Định nghĩa namespace cho lớp Calculator
    {
        public class Calculator
        {
            public static int Cong(int a, int b)
            {
                return a + b;
            }
        }
    }
    ```

*   **Sử dụng `using` directive:** Để sử dụng các lớp từ một namespace khác, bạn có thể khai báo đầy đủ tên lớp (ví dụ: `MyAwesomeApp.Models.Person`) hoặc sử dụng từ khóa `using` ở đầu tệp để "nhập" namespace đó.

    ```csharp
    // Tệp: Program.cs
    using System; // Namespace mặc định cho Console.WriteLine
    using MyAwesomeApp.Models;    // Nhập namespace chứa lớp Person
    using MyAwesomeApp.Services;  // Nhập namespace chứa lớp Calculator

    namespace MyAwesomeApp // Namespace của ứng dụng chính
    {
        public class Program
        {
            public static void Main(string[] args)
            {
                Person john = new Person("John", "Smith", 30); // Không cần MyAwesomeApp.Models.Person
                john.GioiThieu();

                int ketQua = Calculator.Cong(10, 5); // Không cần MyAwesomeApp.Services.Calculator
                Console.WriteLine($"Kết quả phép cộng: {ketQua}");
            }
        }
    }
    ```
    > [!TIP]
    > Trong các môi trường phát triển tích hợp (IDE) như Visual Studio, bạn có thể dễ dàng tạo tệp mới cho lớp hoặc tự động thêm các câu lệnh `using` cần thiết. Việc tổ chức này giúp mã nguồn rõ ràng, dễ đọc và dễ quản lý khi dự án phát triển.

#### Antigravity IDE & Vibe Coding Insights:
Antigravity IDE không chỉ viết mã, mà còn là một kiến trúc sư. Khi bạn yêu cầu nó xây dựng một ứng dụng, nó sẽ tự động tổ chức các lớp vào các namespace hợp lý (ví dụ: `MyAwesomeApp.Models`, `MyAwesomeApp.Services`, `MyAwesomeApp.Controllers`) và đặt mỗi lớp vào một tệp riêng. Nó sẽ "vibe" được cấu trúc dự án tốt nhất để tăng tính module hóa và dễ bảo trì. Nếu bạn quên `using` một namespace, Antigravity sẽ tự động thêm nó, giúp bạn tập trung vào logic nghiệp vụ thay vì các chi tiết cú pháp.

---

## 3. Quản lý Bộ nhớ trong C#: Stack, Heap và Garbage Collection

Một trong những khía cạnh quan trọng nhất cần hiểu trong C# là cách bộ nhớ được quản lý, đặc biệt là sự khác biệt giữa Kiểu Giá trị và Kiểu Tham chiếu. Việc nắm vững cơ chế này là nền tảng để viết mã hiệu quả, tránh các lỗi logic tiềm ẩn và hiểu sâu hơn về ngôn ngữ.

### 3.1. Kiến trúc Bộ nhớ: Stack (Ngăn xếp) và Heap (Vùng nhớ động)

Khi một chương trình C# chạy, nó sử dụng hai khu vực chính trong bộ nhớ để lưu trữ dữ liệu:

*   **Stack (Ngăn xếp):**
    *   Là một vùng bộ nhớ được tổ chức theo cơ chế **LIFO (Last-In, First-Out)** – phần tử nào vào sau sẽ được lấy ra trước, giống như một chồng đĩa.
    *   **Tốc độ:** Việc cấp phát và giải phóng bộ nhớ trên Stack diễn ra rất nhanh chóng và hiệu quả.
    *   **Nội dung lưu trữ:**
        *   Các biến cục bộ (local variables) trong một phương thức.
        *   Các tham số của phương thức.
        *   Địa chỉ trả về của các phương thức.
        *   **Toàn bộ dữ liệu của các kiểu giá trị (Value Types).**
    *   **Thời gian sống:** Dữ liệu trên Stack có thời gian sống ngắn, chỉ tồn tại trong phạm vi của phương thức hoặc khối mã mà nó được khai báo. Khi phương thức kết thúc, toàn bộ khung ngăn xếp (stack frame) của phương thức đó sẽ bị xóa.

*   **Heap (Vùng nhớ động):**
    *   Là một vùng bộ nhớ linh hoạt hơn, được tổ chức không theo thứ tự cụ thể, giống như một "đống" các đối tượng.
    *   **Tốc độ:** Việc cấp phát và giải phóng bộ nhớ trên Heap chậm hơn so với Stack.
    *   **Nội dung lưu trữ:**
        *   **Dữ liệu thực tế của các đối tượng kiểu tham chiếu (Reference Types).**
    *   **Thời gian sống:** Dữ liệu trên Heap có thể có thời gian sống dài hơn, tồn tại cho đến khi không còn bất kỳ tham chiếu nào trỏ đến nó và được Garbage Collector (GC) thu hồi.
    *   **Quản lý:** Việc giải phóng bộ nhớ trên Heap được quản lý tự động bởi Garbage Collector của .NET.

**Minh họa đơn giản:**

```
+----------------+  <-- Stack (LIFO)
| main() {       |
|   int x = 10;  |  <-- 'x' (Value Type) lưu giá trị 10 trực tiếp trên Stack
|   Person p;    |  <-- 'p' (Reference Type biến) lưu địa chỉ trên Stack
|   p = new Person();| <-- Đối tượng Person thực sự được tạo trên Heap
|   ...          |
| }              |
+----------------+
|                |
|                |
+----------------+  <-- Heap (Không theo thứ tự)
| Person Object  |  <-- Dữ liệu của Person (TenDau, TenCuoi, Tuoi)
|   TenDau: "John" |
|   TenCuoi: "Smith" |
|   Tuoi: 30     |
+----------------+
```

### 3.2. Kiểu Giá trị (Value Types): Lưu trữ trực tiếp

*   **Định nghĩa:** Kiểu giá trị trực tiếp lưu trữ giá trị của chúng trong vùng bộ nhớ mà chúng được khai báo.
*   **Lưu trữ:**
    *   Thông thường, chúng được lưu trữ trên **Stack**.
    *   Tuy nhiên, nếu một kiểu giá trị là một trường của một kiểu tham chiếu, nó sẽ được lưu trữ cùng với đối tượng kiểu tham chiếu đó trên **Heap**.
*   **Ví dụ:**
    *   Các kiểu nguyên thủy: `int`, `bool`, `float`, `double`, `char`, `decimal`.
    *   `enum` (kiểu liệt kê).
    *   **`struct` (cấu trúc)** – đây là kiểu giá trị tùy chỉnh quan trọng nhất.

#### 3.2.1. Cơ chế gán và Sao chép giá trị

Khi bạn gán một kiểu giá trị này cho một kiểu giá trị khác, một **bản sao đầy đủ của giá trị** sẽ được tạo ra. Hai biến sau đó hoàn toàn độc lập với nhau trong bộ nhớ.

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        int a = 10; // 'a' là một biến kiểu giá trị, lưu trữ giá trị 10 trực tiếp trên Stack.
                    // Bộ nhớ: [Stack] a: 10

        int b = a;  // 'b' cũng là một biến kiểu giá trị. Một BẢN SAO của giá trị của 'a' (là 10) được tạo ra và gán cho 'b'.
                    // 'a' và 'b' là hai vùng nhớ riêng biệt với giá trị 10.
                    // Bộ nhớ: [Stack] a: 10, b: 10

        b = 20;     // Thay đổi giá trị của 'b' thành 20. Điều này chỉ ảnh hưởng đến vùng nhớ của 'b'.
                    // Bộ nhớ: [Stack] a: 10, b: 20

        Console.WriteLine($"a: {a}, b: {b}"); // Output: a: 10, b: 20
    }
}
```
> [!NOTE]
> Việc sao chép giá trị đảm bảo rằng mỗi biến có bản sao dữ liệu riêng, loại bỏ các tác dụng phụ không mong muốn khi một biến được sửa đổi. Đây là hành vi mà bạn mong đợi từ các "giá trị" như số hoặc ngày tháng.

### 3.3. Kiểu Tham chiếu (Reference Types): Lưu trữ địa chỉ

*   **Định nghĩa:** Kiểu tham chiếu không trực tiếp lưu trữ dữ liệu của đối tượng. Thay vào đó, chúng lưu trữ một **tham chiếu** (thực chất là một địa chỉ bộ nhớ) đến nơi đối tượng thực sự được lưu trữ trên Heap.
*   **Lưu trữ:**
    *   Biến tham chiếu (chứa địa chỉ) được lưu trữ trên **Stack**.
    *   Dữ liệu thực tế của đối tượng được lưu trữ trên **Heap**.
*   **Ví dụ:**
    *   **`class` (lớp)** – đây là kiểu tham chiếu tùy chỉnh quan trọng nhất.
    *   `string` (chuỗi) – mặc dù có hành vi bất biến giống kiểu giá trị trong nhiều trường hợp, nhưng về bản chất là kiểu tham chiếu.
    *   `array` (mảng).
    *   `interface` (giao diện).
    *   `delegate` (ủy quyền).

#### 3.3.1. Cơ chế gán và Sao chép tham chiếu

Khi bạn gán một kiểu tham chiếu này cho một kiểu tham chiếu khác, **chỉ có địa chỉ tham chiếu được sao chép**. Cả hai biến sau đó sẽ trỏ đến **cùng một đối tượng** trên Heap.

```csharp
public class MyClass
{
    public int Value;
}

public class Program
{
    public static void Main(string[] args)
    {
        MyClass obj1 = new MyClass { Value = 10 }; // 'obj1' là biến tham chiếu trên Stack, trỏ đến đối tượng {Value=10} trên Heap.
                                                  // Bộ nhớ: [Stack] obj1 -> (Địa chỉ X)
                                                  //          [Heap] (Địa chỉ X): { Value: 10 }

        MyClass obj2 = obj1;                       // 'obj2' cũng là biến tham chiếu trên Stack.
                                                  // Nó nhận BẢN SAO của địa chỉ mà 'obj1' đang trỏ tới (Địa chỉ X).
                                                  // Bây giờ, cả 'obj1' và 'obj2' CÙNG trỏ đến một đối tượng DUY NHẤT trên Heap.
                                                  // Bộ nhớ: [Stack] obj1 -> (Địa chỉ X), obj2 -> (Địa chỉ X)
                                                  //          [Heap] (Địa chỉ X): { Value: 10 }

        obj2.Value = 20;                           // Thay đổi Value thông qua 'obj2'. Vì cả 'obj1' và 'obj2' trỏ đến cùng đối tượng,
                                                  // việc thay đổi này sẽ ảnh hưởng đến đối tượng mà 'obj1' cũng đang trỏ tới.
                                                  // Bộ nhớ: [Stack] obj1 -> (Địa chỉ X), obj2 -> (Địa chỉ X)
                                                  //          [Heap] (Địa chỉ X): { Value: 20 }

        Console.WriteLine($"obj1.Value: {obj1.Value}, obj2.Value: {obj2.Value}"); // Output: obj1.Value: 20, obj2.Value: 20
    }
}
```
> [!IMPORTANT]
> Hiểu rõ sự khác biệt này là cực kỳ quan trọng để tránh các lỗi logic khi bạn làm việc với các đối tượng, đặc biệt là khi truyền chúng qua các phương thức hoặc gán cho nhiều biến. Nếu bạn muốn một bản sao độc lập của một đối tượng kiểu tham chiếu, bạn cần tự thực hiện việc sao chép (gọi là "cloning" hoặc "deep copy").

#### Trường hợp đặc biệt: `string`

`string` trong C# là một kiểu tham chiếu, nhưng nó lại có tính **bất biến (immutable)**. Điều này có nghĩa là một khi một đối tượng `string` được tạo ra, giá trị của nó không thể thay đổi. Mỗi khi bạn "thay đổi" một chuỗi (ví dụ: `str = str + "abc"`), thực chất C# tạo ra một đối tượng `string` MỚI trên Heap và gán tham chiếu đến đối tượng mới đó cho biến. Hành vi này khiến `string` thường được cảm nhận và sử dụng như một kiểu giá trị.

```csharp
string s1 = "Hello"; // s1 trỏ đến "Hello" trên Heap
string s2 = s1;      // s2 cũng trỏ đến "Hello" trên Heap (cùng đối tượng)

s1 = "World";        // KHÔNG thay đổi đối tượng "Hello". Thay vào đó, một đối tượng "World" MỚI được tạo,
                     // và s1 bây giờ trỏ đến "World". s2 vẫn trỏ đến "Hello".
Console.WriteLine($"s1: {s1}, s2: {s2}"); // Output: s1: World, s2: Hello
```
Đây là lý do tại sao `string` dường như hoạt động giống kiểu giá trị, dù bản chất là kiểu tham chiếu.

### 3.4. Toán tử `new` và Cơ chế Thu gom rác (Garbage Collection)

*   **Toán tử `new`:** Khi bạn sử dụng `new` với một kiểu tham chiếu (ví dụ: `new Person()`), bạn đang yêu cầu Common Language Runtime (CLR) cấp phát một vùng bộ nhớ mới trên **Heap** cho đối tượng đó và trả về địa chỉ của nó. Địa chỉ này sau đó được lưu trữ trong biến tham chiếu trên Stack.
*   **Garbage Collection (Thu gom rác):** C# sử dụng một cơ chế tự động gọi là Garbage Collector (GC) để quản lý bộ nhớ Heap.
    *   GC liên tục theo dõi các đối tượng trên Heap. Khi một đối tượng không còn được bất kỳ biến tham chiếu nào trỏ tới (nghĩa là nó không còn "có thể truy cập được" bởi chương trình), GC sẽ tự động phát hiện và giải phóng vùng bộ nhớ đó.
    *   Điều này giúp ngăn chặn các lỗi rò rỉ bộ nhớ (memory leaks) phổ biến trong các ngôn ngữ quản lý bộ nhớ thủ công và giảm gánh nặng quản lý bộ nhớ cho lập trình viên, cho phép họ tập trung hơn vào logic nghiệp vụ.

#### Antigravity IDE & Vibe Coding Insights:
Antigravity IDE hiểu sâu về quản lý bộ nhớ. Khi nó tạo ra các lớp, nó sẽ tự động xem xét liệu việc sử dụng `new` có cần thiết hay không, và biết rằng Garbage Collector sẽ xử lý việc giải phóng bộ nhớ. Trong những tác vụ tối ưu hóa hiệu suất cao, Antigravity thậm chí có thể phân tích luồng dữ liệu để nhận diện các đối tượng không còn được sử dụng sớm hơn, hoặc gợi ý sử dụng các kỹ thuật quản lý bộ nhớ nâng cao nếu phát hiện rò rỉ tiềm ẩn (dù điều này thường nằm ngoài scope của khóa học cơ bản).

---

## 4. Cấu trúc (Structs) - Kiểu Giá trị tùy chỉnh

Bên cạnh lớp, C# còn cung cấp một kiểu dữ liệu tùy chỉnh khác có tên là cấu trúc (`struct`). Về mặt cú pháp, cấu trúc rất giống với lớp, nhưng chúng có những điểm khác biệt cốt lõi, đặc biệt là về cách chúng được quản lý trong bộ nhớ.

### 4.1. Khái niệm và Cú pháp

*   **Khái niệm:** Cấu trúc cũng là một bản thiết kế kết hợp các trường, thuộc tính và phương thức liên quan với nhau, tương tự như lớp. Tuy nhiên, cấu trúc được thiết kế cho các kiểu dữ liệu nhỏ, nhẹ, thường đại diện cho một giá trị đơn lẻ và có hành vi sao chép giá trị.
*   **Cú pháp:** Sự khác biệt duy nhất trong cú pháp là chúng ta sử dụng từ khóa `struct` thay vì `class`.

```csharp
public struct Point
{
    public int X { get; set; } // Thuộc tính để lưu tọa độ X
    public int Y { get; set; } // Thuộc tính để lưu tọa độ Y

    // Cấu trúc có thể có phương thức
    public void HienThiToaDo()
    {
        Console.WriteLine($"Tọa độ: ({X}, {Y})");
    }

    // Cấu trúc có thể có hàm tạo (constructor) nhưng PHẢI có tham số.
    // Cấu trúc KHÔNG THỂ có hàm tạo mặc định (không tham số) do người dùng định nghĩa.
    // Trình biên dịch C# LUÔN tự động cung cấp một hàm tạo mặc định không tham số cho structs,
    // hàm tạo này sẽ khởi tạo tất cả các trường về giá trị mặc định của chúng (0, false, null).
    public Point(int x, int y)
    {
        X = x; // Phải gán giá trị cho tất cả các trường/thuộc tính trong constructor có tham số
        Y = y;
    }
}
```

### 4.2. Sự khác biệt cốt lõi: Structs là Kiểu Giá trị

Sự khác biệt quan trọng nhất giữa lớp và cấu trúc nằm ở bản chất của chúng:

| Đặc điểm               | Lớp (Class)                                 | Cấu trúc (Struct)                               |
| :--------------------- | :------------------------------------------ | :---------------------------------------------- |
| **Bản chất kiểu**      | **Kiểu Tham chiếu (Reference Type)**        | **Kiểu Giá trị (Value Type)**                   |
| **Vị trí lưu trữ**     | Đối tượng trên **Heap**, biến tham chiếu trên **Stack** | Thường trên **Stack** (hoặc nằm trong Heap nếu là trường của kiểu tham chiếu) |
| **Cơ chế gán**         | Sao chép **tham chiếu** (địa chỉ bộ nhớ)    | Sao chép **toàn bộ giá trị**                    |
| **Toán tử `new`**      | Bắt buộc để cấp phát bộ nhớ trên Heap      | Không bắt buộc (có thể khởi tạo mà không dùng `new`, sẽ gán giá trị mặc định) |
| **Giá trị `null`**     | Có thể là `null` (không trỏ đến đối tượng nào) | Không thể là `null` (trừ khi dùng `Nullable<T>`) |
| **Kế thừa**            | Có thể kế thừa từ một lớp khác, chỉ kế thừa đơn | Không thể kế thừa từ struct/class khác (chỉ có thể implement interface) |
| **Kích thước**         | Thường lớn hơn, có thể chứa nhiều dữ liệu phức tạp | Thường nhỏ, nhẹ (khuyến nghị dưới 16 byte để tối ưu hiệu suất) |
| **Garbage Collection** | Bị quản lý bởi GC khi không còn được tham chiếu | Không bị GC quản lý trực tiếp (bộ nhớ được giải phóng khi ra khỏi phạm vi) |

**Ví dụ minh họa sự khác biệt về gán giá trị:**

```csharp
public class MyClass
{
    public int Value;
    public MyClass(int val) { Value = val; }
}

public struct MyStruct
{
    public int Value;
    public MyStruct(int val) { Value = val; }
}

public class Program
{
    public static void Main(string[] args)
    {
        Console.WriteLine("--- Với Class (Kiểu Tham chiếu) ---");
        MyClass classObj1 = new MyClass(10); // Tạo đối tượng trên Heap, obj1 trỏ đến nó
        MyClass classObj2 = classObj1;       // classObj2 sao chép tham chiếu của obj1. Cả hai trỏ đến CÙNG đối tượng.

        classObj2.Value = 20; // Thay đổi qua classObj2 ảnh hưởng đến đối tượng mà classObj1 cũng trỏ tới
        Console.WriteLine($"Class - classObj1.Value: {classObj1.Value}, classObj2.Value: {classObj2.Value}");
        // Output: Class - classObj1.Value: 20, classObj2.Value: 20

        Console.WriteLine("\n--- Với Struct (Kiểu Giá trị) ---");
        MyStruct structObj1 = new MyStruct(10); // structObj1 lưu trữ giá trị 10 trực tiếp
        MyStruct structObj2 = structObj1;       // structObj2 tạo một BẢN SAO độc lập của structObj1 (giá trị 10).

        structObj2.Value = 20; // Thay đổi qua structObj2 KHÔNG ảnh hưởng đến structObj1
        Console.WriteLine($"Struct - structObj1.Value: {structObj1.Value}, structObj2.Value: {structObj2.Value}");
        // Output: Struct - structObj1.Value: 10, structObj2.Value: 20

        // Khởi tạo struct không dùng new (fields sẽ có giá trị mặc định)
        MyStruct defaultStruct;
        // defaultStruct.Value = 5; // Lỗi: "Use of unassigned local variable 'defaultStruct'"
                                 // Phải gán giá trị cho tất cả các trường trước khi sử dụng nếu không dùng new
        defaultStruct.Value = 5; // Ok sau khi gán
        Console.WriteLine($"Default struct value: {defaultStruct.Value}"); // Output: Default struct value: 5
    }
}
```

### 4.3. Khi nào nên sử dụng Structs (và khi nào không)

Trong thực tế, đại đa số các kiểu dữ liệu tùy chỉnh bạn tạo sẽ là lớp (`class`). Tuy nhiên, cấu trúc (`struct`) có những trường hợp sử dụng cụ thể mà ở đó chúng vượt trội:

*   **Đối tượng nhỏ, nhẹ:** Sử dụng cấu trúc khi bạn muốn định nghĩa một đối tượng nhỏ gọn, thường có kích thước dưới 16 byte và không chứa nhiều trường kiểu tham chiếu. Ví dụ điển hình là các điểm tọa độ (`Point` với X, Y), màu sắc (`Color` với R, G, B, A), hoặc các cặp giá trị đơn giản.
*   **Hành vi sao chép giá trị:** Khi bạn muốn mỗi biến có một bản sao độc lập của dữ liệu, thay vì cùng trỏ đến một đối tượng. Điều này phù hợp với các "giá trị" như số hoặc ngày tháng.
*   **Cải thiện hiệu suất:** Khi bạn cần tạo hàng nghìn hoặc hàng chục nghìn đối tượng nhỏ trong các vòng lặp dày đặc. Việc sử dụng cấu trúc có thể hiệu quả hơn vì chúng được lưu trữ trên Stack (hoặc trực tiếp trong Heap nếu là trường của một lớp), giảm gánh nặng cho Garbage Collector và có thể cải thiện khả năng truy cập bộ nhớ (cache locality).
*   **Kiểu dữ liệu bất biến (Immutable types):** Cấu trúc thường được sử dụng cho các kiểu dữ liệu mà giá trị của chúng không thay đổi sau khi được tạo. Điều này giúp đơn giản hóa logic và tránh các lỗi tác dụng phụ.

> [!TIP]
> **Nguyên tắc chung:** Nếu bạn không chắc chắn nên sử dụng lớp hay cấu trúc, hãy bắt đầu với lớp. Lớp linh hoạt hơn, hỗ trợ đầy đủ các tính năng OOP như kế thừa và là lựa chọn mặc định trong hầu hết các tình huống. Chỉ xem xét `struct` khi bạn có yêu cầu cụ thể về hiệu suất, kích thước nhỏ gọn và hành vi sao chép giá trị.

### 4.4. Hạn chế và cân nhắc của Cấu trúc

*   **Không thể là `null`:** Một biến cấu trúc luôn phải chứa một giá trị. Để có một cấu trúc có thể `null`, bạn phải sử dụng `Nullable<T>` (ví dụ: `Point? p = null;`).
*   **Không hỗ trợ kế thừa:** Cấu trúc không thể kế thừa từ một cấu trúc hoặc lớp khác, giới hạn khả năng mở rộng và tái sử dụng mã thông qua đa hình.
*   **Có thể gây hiệu suất kém nếu sử dụng sai:** Nếu một cấu trúc quá lớn (ví dụ: chứa nhiều trường hoặc các trường là kiểu tham chiếu) hoặc được truyền qua các phương thức dưới dạng tham số giá trị quá thường xuyên, việc sao chép toàn bộ dữ liệu có thể tốn kém và làm giảm hiệu suất, đôi khi còn tệ hơn cả sử dụng `class`.
*   **Không thể có hàm tạo mặc định do người dùng định nghĩa:** Bạn không thể viết một hàm tạo không tham số cho `struct` của mình; C# luôn cung cấp một hàm tạo mặc định sẽ khởi tạo tất cả các trường về giá trị mặc định.

#### Antigravity IDE & Vibe Coding Insights:
Antigravity IDE sẽ không chỉ tạo mã mà còn "tư vấn" cho bạn. Khi bạn định nghĩa một kiểu dữ liệu nhỏ như `Point` hoặc `Color`, Antigravity có thể gợi ý sử dụng `struct` thay vì `class`, giải thích những lợi ích về hiệu suất và ngữ nghĩa sao chép giá trị. Ngược lại, nếu bạn cố gắng tạo một `struct` quá lớn hoặc có hành vi phức tạp, Antigravity có thể cảnh báo và đề xuất chuyển sang `class`, cùng với lý do về hiệu suất và tính linh hoạt. Nó giúp bạn "vibe" được lựa chọn thiết kế phù hợp nhất.

---

## 5. Tổng kết: Lớp và Cấu trúc - Lựa chọn đúng cho từng trường hợp

*   **Lớp (Class):**
    *   Là bản thiết kế cho các đối tượng phức tạp, kết hợp dữ liệu (trường/thuộc tính) và hành vi (phương thức).
    *   Là **Kiểu Tham chiếu (Reference Type)**: Biến lưu trữ địa chỉ của đối tượng trên **Heap**.
    *   Được cấp phát bộ nhớ bằng toán tử `new` trên Heap, bộ nhớ được giải phóng tự động bởi **Garbage Collector**.
    *   Hỗ trợ kế thừa, đa hình và tính đóng gói (thông qua properties).
    *   Là lựa chọn mặc định và linh hoạt nhất cho hầu hết các kiểu dữ liệu tùy chỉnh.
*   **Cấu trúc (Struct):**
    *   Tương tự lớp nhưng được thiết kế cho các kiểu dữ liệu nhỏ, nhẹ, đại diện cho một giá trị.
    *   Là **Kiểu Giá trị (Value Type)**: Trực tiếp lưu trữ giá trị của chúng trên **Stack** (hoặc nằm trong Heap nếu là trường của một lớp).
    *   Khi gán, tạo một bản **sao đầy đủ của giá trị**.
    *   Không thể là `null` (trừ khi dùng `Nullable<T>`), không hỗ trợ kế thừa.
    *   Thích hợp cho các đối tượng nhỏ (dưới 16 byte), bất biến hoặc khi cần hành vi sao chép giá trị rõ ràng, có thể cải thiện hiệu suất trong các kịch bản cụ thể.
*   **Kiểu Giá trị vs. Kiểu Tham chiếu:**
    *   Sự khác biệt cốt lõi nằm ở cách chúng được lưu trữ trong bộ nhớ (Stack vs. Heap) và cách chúng hoạt động khi được gán (sao chép giá trị vs. sao chép tham chiếu).
    *   Hiểu rõ sự khác biệt này là rất quan trọng để tránh các lỗi logic, quản lý bộ nhớ hiệu quả và đưa ra quyết định thiết kế đúng đắn.
*   **Thành viên Tĩnh (Static Members):**
    *   Thuộc về chính lớp, không phải đối tượng cụ thể.
    *   Chỉ có một phiên bản duy nhất trong bộ nhớ.
    *   Truy cập trực tiếp qua tên lớp (ví dụ: `Console.WriteLine()`).
    *   Sử dụng cho các chức năng tiện ích hoặc dữ liệu toàn cục, không liên quan đến một thể hiện cụ thể.
*   **Tổ chức mã nguồn:**
    *   Sử dụng tệp `.cs` riêng cho mỗi lớp/cấu trúc.
    *   Sử dụng **Namespaces** để nhóm các kiểu liên quan và tránh xung đột tên.
    *   Sử dụng từ khóa `using` để nhập các Namespaces cần thiết.

---

## 6. Antigravity IDE và Vibe Coding: Áp dụng tư duy thiết kế

Hệ thống Antigravity IDE mà bạn đang sử dụng là một ví dụ điển hình về việc AI có thể hỗ trợ lập trình viên ở cấp độ cao. Để tận dụng tối đa sức mạnh của nó, bạn cần "vibe" (cảm nhận) được các nguyên tắc thiết kế cơ bản mà chúng ta vừa học.

### 6.1. Hiểu "Vibe" của dữ liệu: Lớp hay Cấu trúc?

Khi bạn phác thảo một ý tưởng cho Antigravity, nó sẽ cố gắng hiểu "ý định" của bạn.

*   Nếu bạn mô tả một thực thể phức tạp, có vòng đời dài, có thể cần kế thừa hoặc thay đổi trạng thái thường xuyên (ví dụ: `Customer`, `Order`, `Product`), Antigravity sẽ "vibe" rằng đây nên là một **lớp**. Nó sẽ tự động tạo ra một `class` với các thuộc tính và phương thức phù hợp.
*   Nếu bạn mô tả một tập hợp dữ liệu nhỏ, đơn giản, đại diện cho một giá trị, và bạn muốn hành vi sao chép giá trị (ví dụ: `Point`, `Size`, `Temperature`), Antigravity có thể gợi ý hoặc tự động tạo ra một **cấu trúc**. Nó hiểu rằng việc này có thể cải thiện hiệu suất và rõ ràng về ngữ nghĩa.

Bằng cách hiểu sự khác biệt giữa lớp và cấu trúc, bạn có thể đưa ra yêu cầu rõ ràng hơn cho Antigravity, hoặc hiểu lý do tại sao nó lại tạo ra một `class` thay vì `struct` (hoặc ngược lại) trong một kịch bản nhất định.

### 6.2. Tổ chức mã nguồn và Namespaces với Antigravity

Antigravity IDE được thiết kế để duy trì một codebase có tổ chức.

*   Khi bạn tạo một lớp mới, Antigravity sẽ tự động đặt nó vào một tệp riêng và gợi ý một `namespace` phù hợp dựa trên cấu trúc dự án hiện có.
*   Nếu bạn di chuyển một lớp hoặc đổi tên namespace, Antigravity sẽ tự động cập nhật các câu lệnh `using` và các tham chiếu khác trong toàn bộ dự án, đảm bảo tính nhất quán.
*   Bạn có thể hướng dẫn Antigravity tạo các module hoặc thư viện mới, và nó sẽ tạo cấu trúc namespace tương ứng, giúp bạn quản lý các dự án lớn một cách hiệu quả.

### 6.3. Thiết kế thành viên và Tương tác đối tượng

*   **Tính đóng gói (Encapsulation):** Khi bạn yêu cầu Antigravity tạo các trường dữ liệu cho một lớp, nó thường sẽ tạo chúng dưới dạng `private fields` và cung cấp `public properties` tương ứng. Đây là cách Antigravity "vibe" được nguyên tắc đóng gói mà không cần bạn phải chỉ định rõ ràng.
*   **Hàm tạo (Constructors):** Antigravity có thể tự động tạo ra các hàm tạo (có tham số hoặc không) dựa trên các trường của lớp, giúp bạn khởi tạo đối tượng dễ dàng.
*   **Thành viên tĩnh (Static Members):** Nếu bạn yêu cầu một chức năng tiện ích hoặc một biến toàn cục cho một lớp, Antigravity có thể tạo ra các phương thức hoặc trường tĩnh, nhận ra rằng chúng thuộc về lớp chứ không phải đối tượng cụ thể.

### 6.4. Tối ưu hóa bộ nhớ và hiệu suất (Dành cho Antigravity)

Mặc dù Antigravity xử lý nhiều chi tiết cấp thấp, nhưng việc bạn hiểu về Stack, Heap và Garbage Collection giúp bạn:

*   **Đánh giá gợi ý của Antigravity:** Khi Antigravity đưa ra một lựa chọn thiết kế có thể ảnh hưởng đến hiệu suất (ví dụ: đề xuất sử dụng `struct` thay vì `class`), bạn có thể hiểu được căn nguyên của gợi ý đó.
*   **Cung cấp ngữ cảnh tốt hơn:** Nếu bạn đang làm việc trên một phần mã nguồn cực kỳ nhạy cảm về hiệu suất, bạn có thể giải thích rõ ràng hơn cho Antigravity về các ràng buộc bộ nhớ, cho phép nó tạo ra mã tối ưu hơn (ví dụ: tránh cấp phát đối tượng không cần thiết trên Heap).

Bằng cách "vibe" được các nguyên tắc cơ bản của C# và OOP, bạn không chỉ trở thành một lập trình viên giỏi hơn mà còn là một người cộng tác hiệu quả hơn với Antigravity IDE, hướng dẫn nó tạo ra mã nguồn chất lượng cao, đúng với ý định và tối ưu nhất cho ứng dụng của bạn.

<!-- REVIEWED_BY_AGENT -->
