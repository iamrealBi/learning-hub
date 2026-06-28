# Chương 2: Biến, Hằng số và Phạm vi trong C#

## Giới thiệu

Chào mừng bạn đến với Chương 2 của khóa học Lập trình C# và Lập trình Hướng đối tượng (OOP). Trong chương này, chúng ta sẽ xây dựng nền tảng vững chắc bằng cách khám phá các khái niệm cơ bản nhưng vô cùng quan trọng: biến (variables), hằng số (constants) và phạm vi (scope). Việc nắm vững những yếu tố này là điều kiện tiên quyết để bạn có thể viết mã nguồn hiệu quả, dễ đọc và dễ bảo trì.

Chúng ta không chỉ dừng lại ở cú pháp khai báo mà còn đi sâu vào cơ chế hoạt động ngầm của C#, đặc biệt là cách dữ liệu được lưu trữ và quản lý trong bộ nhớ máy tính.

## Mục tiêu của Chương này

Sau khi hoàn thành chương này, bạn sẽ có khả năng:
*   Định nghĩa và sử dụng biến cùng hằng số một cách thành thạo trong C#.
*   Áp dụng các quy tắc và quy ước đặt tên chuẩn của C# để tạo mã nguồn rõ ràng.
*   Phân biệt sâu sắc giữa Kiểu giá trị (Value Type) và Kiểu tham chiếu (Reference Type), bao gồm cơ chế cấp phát bộ nhớ trên Stack và Heap.
*   Hiểu về hiện tượng tràn số (overflowing) và cách phòng tránh hoặc kiểm soát nó bằng từ khóa `checked`.
*   Nắm vững khái niệm phạm vi (scope) và cách nó ảnh hưởng đến vòng đời và khả năng truy cập của biến, hằng số.
*   Thực hành viết mã C# để củng cố kiến thức lý thuyết.

---

## 1. Biến và Hằng số: Nền tảng lưu trữ dữ liệu

Trong mọi ứng dụng, chúng ta cần một cách để lưu trữ, truy xuất và thao tác với dữ liệu. Biến và hằng số chính là hai cơ chế cơ bản nhất để thực hiện điều này.

### 1.1. Biến (Variables)

Một **biến** là một tên (định danh) mà chúng ta đặt cho một vùng nhớ cụ thể trong bộ nhớ máy tính. Vùng nhớ này được dùng để lưu trữ một giá trị, và giá trị đó có thể thay đổi trong suốt quá trình thực thi của chương trình.

Hãy hình dung một biến như một chiếc "hộp" được dán nhãn trong bộ nhớ của máy tính. Bạn có thể đặt một giá trị vào hộp đó, lấy giá trị ra để sử dụng, hoặc thay thế giá trị bên trong bằng một giá trị mới. Nhãn dán trên hộp chính là tên của biến.

**Cú pháp khai báo biến:**

Để khai báo một biến trong C#, bạn cần chỉ định kiểu dữ liệu (data type) của nó, theo sau là tên (định danh - identifier) của biến, và kết thúc bằng dấu chấm phẩy (`;`). Kiểu dữ liệu cho trình biên dịch biết loại giá trị mà biến có thể chứa và lượng bộ nhớ cần thiết để lưu trữ nó.

```csharp
// Khai báo một biến kiểu số nguyên (integer) có tên 'soLuongSanPham'
int soLuongSanPham;

// Khai báo một biến kiểu byte có tên 'tuoiNguoiDung'
byte tuoiNguoiDung;

// Khai báo một biến kiểu số thực (floating-point) có tên 'giaTien'
float giaTien;

// Khai báo một biến kiểu chuỗi (string) có tên 'tenKhachHang'
string tenKhachHang;
```

**Khởi tạo biến:**

Trong C#, bạn có thể gán giá trị cho một biến ngay khi khai báo hoặc gán sau đó. Tuy nhiên, một quy tắc cực kỳ quan trọng là bạn **không thể sử dụng một biến trừ khi nó đã được khởi tạo** (gán giá trị ban đầu). Nếu không, trình biên dịch sẽ báo lỗi. Điều này giúp ngăn chặn việc chương trình hoạt động với các giá trị không xác định, gây ra hành vi không mong muốn.

```csharp
// Khai báo và khởi tạo ngay lập tức
int diemSo = 100;
Console.WriteLine($"Điểm số: {diemSo}"); // Hợp lệ

// Khai báo trước, sau đó khởi tạo
string tenSinhVien;
tenSinhVien = "Nguyen Van A";
Console.WriteLine($"Tên sinh viên: {tenSinhVien}"); // Hợp lệ

// Ví dụ về lỗi biên dịch: Biến chưa được khởi tạo
// int chuaKhoiTao;
// Console.WriteLine(chuaKhoiTao); // Lỗi biên dịch: "Use of unassigned local variable 'chuaKhoiTao'"
```

### 1.2. Hằng số (Constants)

Một **hằng số** là một giá trị mà chúng ta biết tại thời điểm biên dịch và giá trị đó **không thể thay đổi** trong suốt vòng đời của ứng dụng.

**Tại sao chúng ta sử dụng hằng số?**

Việc sử dụng hằng số mang lại nhiều lợi ích quan trọng:
*   **An toàn dữ liệu:** Đảm bảo rằng các giá trị quan trọng không bị thay đổi một cách vô ý trong quá trình thực thi chương trình.
*   **Tính rõ ràng:** Thay vì sử dụng các "số ma thuật" (magic numbers) trực tiếp trong mã (ví dụ: `if (loai == 3)`), việc sử dụng hằng số có tên mô tả (`if (loai == LOAI_SAN_PHAM_DAC_BIET)`) giúp mã dễ đọc và dễ hiểu hơn.
*   **Dễ bảo trì:** Nếu một giá trị cố định cần thay đổi (ví dụ: thuế suất), bạn chỉ cần thay đổi nó ở một vị trí duy nhất (định nghĩa hằng số), thay vì phải tìm kiếm và sửa đổi ở nhiều nơi trong mã.

> [!TIP]
> Hãy tưởng tượng bạn đang xây dựng một ứng dụng tính toán diện tích hình tròn. Giá trị của số Pi (π) là một hằng số toán học xấp xỉ 3.14159. Giá trị này phải luôn cố định. Nếu bạn vô tình thay đổi giá trị của Pi trong chương trình, tất cả các phép tính liên quan đến vòng tròn sẽ sai lệch. Bằng cách khai báo Pi là một hằng số, trình biên dịch sẽ ngăn bạn thay đổi nó, đảm bảo tính đúng đắn của chương trình.

**Cú pháp khai báo hằng số:**

Để khai báo một hằng số, bạn sử dụng từ khóa `const`, theo sau là kiểu dữ liệu, tên định danh và phải **khởi tạo nó ngay lập tức** với một giá trị.

```csharp
// Khai báo hằng số Pi
const float PI = 3.14159f; // Chú ý hậu tố 'f' cho kiểu float

// Khai báo hằng số số ngày trong tuần
const int SO_NGAY_TRONG_TUAN = 7;

// Khai báo hằng số chuỗi cho tên ứng dụng
const string TEN_UNG_DUNG = "Quản Lý Học Sinh";

Console.WriteLine($"Hằng số Pi: {PI}");
Console.WriteLine($"Số ngày trong tuần: {SO_NGAY_TRONG_TUAN}");

// Lỗi: Không thể gán giá trị cho hằng số sau khi khai báo
// PI = 3.14f; // Lỗi biên dịch: "The left-hand side of an assignment must be a variable, property or indexer"
```

> [!NOTE]
> Khác với biến, bạn bắt buộc phải gán giá trị cho một hằng số ngay tại thời điểm khai báo. Bạn không thể khai báo một hằng số mà không gán giá trị cho nó.

### 1.3. Quy tắc và Quy ước đặt tên (Identifiers and Naming Conventions)

**Định danh (Identifier)** là tên bạn đặt cho biến, hằng số, phương thức, lớp, v.v. Có một số quy tắc bạn phải tuân thủ và các quy ước tốt nhất để mã của bạn dễ đọc hơn.

**Quy tắc bắt buộc:**
*   Định danh phải bắt đầu bằng một chữ cái (a-z, A-Z), dấu gạch dưới (`_`), hoặc ký tự `@`. Nó không thể bắt đầu bằng một số (ví dụ: `1stName` là sai, `firstName` là đúng).
*   Định danh không thể chứa khoảng trắng (ví dụ: `first Name` là sai, `firstName` là đúng).
*   Định danh không thể là một từ khóa dành riêng của C# (ví dụ: `class`, `int`, `void`, `public`).
    *   Nếu bạn thực sự cần sử dụng một từ trùng với từ khóa C# (ví dụ, tên biến là `class`), bạn có thể tiền tố nó bằng ký hiệu `@` (ví dụ: `@class`). Tuy nhiên, điều này không được khuyến khích và chỉ nên dùng trong những trường hợp đặc biệt (ví dụ, khi tương tác với thư viện từ ngôn ngữ khác).
*   C# là ngôn ngữ **phân biệt chữ hoa chữ thường** (case-sensitive). Điều này có nghĩa là `myVariable`, `MyVariable`, và `MYVARIABLE` được coi là ba định danh khác nhau.

**Quy ước đặt tên (Naming Conventions) trong C#:**

Các quy ước này không bắt buộc về mặt kỹ thuật nhưng được cộng đồng C# và Microsoft khuyến khích mạnh mẽ. Việc tuân thủ giúp mã nguồn dễ đọc, dễ hiểu và dễ bảo trì, đặc biệt khi làm việc trong các dự án nhóm.

*   **`camelCase` (Kiểu lạc đà):**
    *   Chữ cái đầu tiên của từ đầu tiên là chữ thường, và chữ cái đầu tiên của mỗi từ tiếp theo là chữ hoa.
    *   **Áp dụng cho:** Biến cục bộ (local variables) và tham số phương thức (method parameters).
    *   **Ví dụ:** `firstName`, `totalPrice`, `employeeId`, `inputMessage`.

*   **`PascalCase` (Kiểu Pascal):**
    *   Chữ cái đầu tiên của mỗi từ (bao gồm cả từ đầu tiên) đều là chữ hoa.
    *   **Áp dụng cho:** Hằng số (constants), tên lớp (class names), tên phương thức (method names), thuộc tính (properties), tên enum, tên namespace.
    *   **Ví dụ:** `Pi`, `NumberOfDaysInWeek`, `CalculateArea`, `MyClass`, `GetStudentName`.

*   **Tránh ký hiệu Hungarian (Hungarian Notation):**
    *   Đây là một quy ước đặt tên tiền tố kiểu dữ liệu vào tên biến (ví dụ: `strName` cho biến chuỗi, `intCount` cho biến số nguyên, `bIsActive` cho biến boolean).
    *   **Lý do tránh trong C#:** Với tính năng IntelliSense mạnh mẽ của Visual Studio và hệ thống kiểu tĩnh của C#, việc thêm tiền tố kiểu dữ liệu là không cần thiết. Nó làm cho mã trở nên dài dòng, khó đọc hơn và không mang lại lợi ích đáng kể.

## 2. Các kiểu dữ liệu cơ bản và Cơ chế cấp phát bộ nhớ

Mỗi biến trong C# phải có một kiểu dữ liệu. Kiểu dữ liệu xác định loại giá trị mà biến có thể lưu trữ, phạm vi giá trị hợp lệ, và lượng bộ nhớ mà nó chiếm dụng.

### 2.1. Giới thiệu về Kiểu dữ liệu

Kiểu dữ liệu là một khía cạnh cốt lõi của bất kỳ ngôn ngữ lập trình nào. Nó giúp trình biên dịch hiểu cách xử lý dữ liệu và cách cấp phát bộ nhớ. C# là một ngôn ngữ có kiểu dữ liệu mạnh (strongly typed), nghĩa là bạn phải khai báo kiểu dữ liệu cho biến và trình biên dịch sẽ kiểm tra tính nhất quán của kiểu trong suốt quá trình biên dịch.

### 2.2. Kiểu Giá trị (Value Types) và Kiểu Tham chiếu (Reference Types)

Để hiểu sâu hơn về cách dữ liệu được lưu trữ và thao tác trong C#, chúng ta cần phân biệt rõ ràng giữa kiểu giá trị và kiểu tham chiếu. Đây là một khái niệm nền tảng, ảnh hưởng đến hiệu suất và cách bạn quản lý dữ liệu.

#### 2.2.1. Kiểu Giá trị (Value Types)

*   **Định nghĩa:** Kiểu giá trị trực tiếp chứa dữ liệu của chúng. Khi một biến kiểu giá trị được khai báo, một vùng nhớ đủ lớn để chứa giá trị đó sẽ được cấp phát.
*   **Cơ chế lưu trữ trên Stack:**
    *   Các biến kiểu giá trị thường được lưu trữ trên **Stack** (ngăn xếp). Stack là một vùng bộ nhớ được quản lý tự động, nhanh và hiệu quả, hoạt động theo nguyên tắc LIFO (Last-In, First-Out).
    *   Khi một hàm được gọi, các biến cục bộ kiểu giá trị và tham số của hàm đó sẽ được đẩy vào Stack. Khi hàm kết thúc, các biến này sẽ tự động bị loại bỏ khỏi Stack, giải phóng bộ nhớ.
    *   Stack lý tưởng cho các dữ liệu có kích thước cố định và vòng đời ngắn.
*   **Ví dụ minh họa và cơ chế sao chép:**
    *   Khi bạn gán một biến kiểu giá trị cho một biến khác, một **bản sao hoàn chỉnh** của giá trị được tạo ra. Hai biến sau đó hoạt động độc lập với nhau.

```csharp
int soGoc = 10; // Biến soGoc (value type) được lưu trên Stack, chứa giá trị 10
int soBanSao = soGoc; // Một bản sao của giá trị 10 được tạo ra và gán cho soBanSao
                      // soBanSao cũng được lưu trên Stack và chứa giá trị 10

Console.WriteLine($"Ban đầu: soGoc = {soGoc}, soBanSao = {soBanSao}"); // Output: Ban đầu: soGoc = 10, soBanSao = 10

soGoc = 20; // Thay đổi giá trị của soGoc không ảnh hưởng đến soBanSao

Console.WriteLine($"Sau thay đổi: soGoc = {soGoc}, soBanSao = {soBanSao}"); // Output: Sau thay đổi: soGoc = 20, soBanSao = 10
```
Trong ví dụ trên, `soGoc` và `soBanSao` là hai vùng nhớ riêng biệt trên Stack, mỗi vùng chứa một bản sao của giá trị.

*   **Các kiểu dữ liệu là Value Types:**
    *   Các kiểu số nguyên (`byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`).
    *   Các kiểu số thực (`float`, `double`, `decimal`).
    *   Ký tự (`char`).
    *   Boolean (`bool`).
    *   Kiểu cấu trúc (`struct`).
    *   Kiểu liệt kê (`enum`).

#### 2.2.2. Kiểu Tham chiếu (Reference Types)

*   **Định nghĩa:** Kiểu tham chiếu không trực tiếp chứa dữ liệu. Thay vào đó, chúng lưu trữ một **tham chiếu** (một địa chỉ bộ nhớ) đến nơi dữ liệu thực tế (đối tượng) được lưu trữ.
*   **Cơ chế lưu trữ trên Heap và Stack:**
    *   Dữ liệu thực tế (đối tượng) của kiểu tham chiếu được lưu trữ trên **Heap** (vùng nhớ động). Heap là một vùng bộ nhớ linh hoạt hơn, được sử dụng cho các đối tượng có kích thước thay đổi và vòng đời dài hơn.
    *   Biến kiểu tham chiếu bản thân nó (chứa địa chỉ) được lưu trữ trên Stack.
    *   **Ví dụ:** Khi bạn khai báo `string ten = "Alice";`, biến `ten` trên Stack sẽ chứa địa chỉ của chuỗi "Alice" trên Heap.
*   **Ví dụ minh họa và cơ chế sao chép:**
    *   Khi bạn gán một biến kiểu tham chiếu cho một biến khác, chỉ có địa chỉ tham chiếu được sao chép. Cả hai biến sau đó sẽ **cùng trỏ đến một đối tượng** trên Heap. Thay đổi dữ liệu thông qua một biến sẽ ảnh hưởng đến dữ liệu mà biến kia đang tham chiếu.

```csharp
class NhanVien // Một class là kiểu tham chiếu
{
    public string Ten { get; set; }
    public NhanVien(string ten) { Ten = ten; }
}

NhanVien nv1 = new NhanVien("Duong"); // 1. nv1 (trên Stack) chứa địa chỉ của đối tượng NhanVien("Duong") (trên Heap)
NhanVien nv2 = nv1;                   // 2. nv2 (trên Stack) sao chép địa chỉ từ nv1.
                                      //    Cả nv1 và nv2 đều trỏ đến CÙNG một đối tượng NhanVien("Duong") trên Heap.

Console.WriteLine($"Ban đầu: nv1.Ten = {nv1.Ten}, nv2.Ten = {nv2.Ten}"); // Output: Ban đầu: nv1.Ten = Duong, nv2.Ten = Duong

nv1.Ten = "Minh"; // Thay đổi thuộc tính Ten thông qua nv1. Điều này thay đổi đối tượng trên Heap.

Console.WriteLine($"Sau thay đổi: nv1.Ten = {nv1.Ten}, nv2.Ten = {nv2.Ten}"); // Output: Sau thay đổi: nv1.Ten = Minh, nv2.Ten = Minh
```
Trong ví dụ này, `nv1` và `nv2` là hai biến trên Stack, nhưng chúng cùng trỏ đến **một** đối tượng `NhanVien` trên Heap.

*   **Vai trò của Garbage Collector:**
    *   Bộ sưu tập rác (Garbage Collector - GC) của .NET là một phần quan trọng trong việc quản lý bộ nhớ Heap. Khi không còn biến nào tham chiếu đến một đối tượng trên Heap, GC sẽ tự động giải phóng vùng nhớ đó. Điều này giúp lập trình viên không cần phải lo lắng về việc quản lý bộ nhớ thủ công, giảm thiểu lỗi rò rỉ bộ nhớ.

*   **Các kiểu dữ liệu là Reference Types:**
    *   Chuỗi (`string`).
    *   Các lớp (`class`).
    *   Mảng (`array`).
    *   Giao diện (`interface`).
    *   Ủy quyền (`delegate`).
    *   Kiểu `object`.

#### 2.2.3. So sánh Value Type và Reference Type

| Đặc điểm           | Kiểu Giá trị (Value Types)                               | Kiểu Tham chiếu (Reference Types)                                 |
| :----------------- | :------------------------------------------------------- | :---------------------------------------------------------------- |
| **Lưu trữ dữ liệu** | Trực tiếp chứa dữ liệu.                                  | Chứa địa chỉ (tham chiếu) đến dữ liệu thực tế.                   |
| **Vị trí bộ nhớ**  | Thường trên **Stack**.                                  | Biến trên **Stack**, dữ liệu thực tế trên **Heap**.               |
| **Cơ chế gán**     | Sao chép **giá trị** (tạo bản sao độc lập).             | Sao chép **tham chiếu** (cùng trỏ đến một đối tượng).             |
| **Giải phóng bộ nhớ** | Tự động khi thoát khỏi phạm vi (trên Stack).             | Được quản lý bởi Garbage Collector (trên Heap).                   |
| **Hiệu suất**      | Nhanh hơn khi truy cập và gán (do dữ liệu trực tiếp, trên Stack). | Chậm hơn một chút (cần theo địa chỉ để truy cập dữ liệu, trên Heap). |
| **Ví dụ**          | `int`, `float`, `bool`, `char`, `struct`, `enum`.        | `string`, `class`, `array`, `interface`, `delegate`.              |

> [!IMPORTANT]
> Hiểu rõ sự khác biệt giữa Value Type và Reference Type là cực kỳ quan trọng để tránh các lỗi tiềm ẩn liên quan đến việc sao chép và chia sẻ dữ liệu, đặc biệt khi làm việc với các đối tượng phức tạp và cấu trúc dữ liệu.

### 2.3. Các Kiểu Dữ liệu Nguyên thủy Phổ biến (Value Types)

Dưới đây là bảng tổng hợp các kiểu dữ liệu nguyên thủy (Value Types) thường dùng trong C#, cùng với kích thước và phạm vi giá trị của chúng. Việc chọn kiểu dữ liệu phù hợp giúp tiết kiệm bộ nhớ và tránh lỗi tràn số.

| Loại dữ liệu | Từ khóa C# | Kiểu .NET Framework | Kích thước (Byte) | Phạm vi giá trị                                    | Ghi chú                                         |
| :----------- | :--------- | :----------------- | :---------------- | :------------------------------------------------- | :---------------------------------------------- |
| **Số nguyên có dấu** |            |                    |                   |                                                    | Có thể chứa số âm và số dương.                   |
| Byte có dấu  | `sbyte`    | `System.SByte`     | 1                 | -128 đến 127                                       |                                                 |
| Số nguyên nhỏ | `short`    | `System.Int16`     | 2                 | -32,768 đến 32,767                                |                                                 |
| Số nguyên    | `int`      | `System.Int32`     | 4                 | -2,147,483,648 đến 2,147,483,647                  | **Kiểu mặc định cho các số nguyên.**             |
| Số nguyên lớn | `long`     | `System.Int64`     | 8                 | -9,223,372,036,854,775,808 đến 9,223,372,036,854,775,807 |                                                 |
| **Số nguyên không dấu** |            |                    |                   |                                                    | Chỉ chứa số không âm.                             |
| Byte         | `byte`     | `System.Byte`      | 1                 | 0 đến 255                                          |                                                 |
| Số nguyên nhỏ không dấu | `ushort`   | `System.UInt16`    | 2                 | 0 đến 65,535                                       |                                                 |
| Số nguyên không dấu | `uint`     | `System.UInt32`    | 4                 | 0 đến 4,294,967,295                               |                                                 |
| Số nguyên lớn không dấu | `ulong`    | `System.UInt64`    | 8                 | 0 đến 18,446,744,073,709,551,615                  |                                                 |
| **Số thực**  |            |                    |                   |                                                    |                                                 |
| Số thực đơn  | `float`    | `System.Single`    | 4                 | ±1.5 × 10^-45 đến ±3.4 × 10^38 (7 chữ số thập phân) | Cần hậu tố `f` hoặc `F` (ví dụ: `1.23f`).      |
| Số thực kép  | `double`   | `System.Double`    | 8                 | ±5.0 × 10^-324 đến ±1.7 × 10^308 (15-16 chữ số thập phân) | **Kiểu mặc định cho các số thực.**              |
| Số thập phân | `decimal`  | `System.Decimal`   | 16                | ±1.0 × 10^-28 đến ±7.9 × 10^28 (28-29 chữ số thập phân) | Cần hậu tố `m` hoặc `M` (ví dụ: `1.23m`). Chính xác cao, dùng cho tài chính. |
| **Ký tự**    |            |                    |                   |                                                    |                                                 |
| Ký tự        | `char`     | `System.Char`      | 2                 | Một ký tự Unicode duy nhất (U+0000 đến U+FFFF)    | Sử dụng dấu nháy đơn `'` (ví dụ: `'A'`).         |
| **Boolean**  |            |                    |                   |                                                    |                                                 |
| Logic        | `bool`     | `System.Boolean`   | 1                 | `true` hoặc `false`                                | Từ khóa `true` và `false` là chữ thường.        |

### 2.4. Kiểu Chuỗi (`string`) - Một Kiểu Tham chiếu Đặc biệt

Mặc dù `string` (`System.String`) là một kiểu tham chiếu, nó có một số hành vi đặc biệt khiến nó đôi khi giống như một kiểu giá trị, đặc biệt là tính **bất biến (immutability)**.

*   **Bất biến là gì?** Khi bạn tạo một đối tượng `string`, giá trị của nó không thể thay đổi sau đó. Bất kỳ thao tác nào dường như "thay đổi" một chuỗi (ví dụ: nối chuỗi, thay thế ký tự) thực chất đều tạo ra một đối tượng `string` mới trên Heap với giá trị đã sửa đổi, và biến ban đầu sẽ trỏ đến đối tượng mới đó (hoặc một biến mới được tạo ra).
*   **Ví dụ:**

```csharp
string s1 = "Hello"; // s1 trỏ đến chuỗi "Hello" trên Heap
string s2 = s1;      // s2 cũng trỏ đến chuỗi "Hello" trên Heap

Console.WriteLine($"Ban đầu: s1 = {s1}, s2 = {s2}"); // Output: Ban đầu: s1 = Hello, s2 = Hello

s1 = s1 + " World";  // KHÔNG thay đổi chuỗi "Hello" gốc.
                     // Thay vào đó, một chuỗi mới "Hello World" được tạo trên Heap,
                     // và s1 bây giờ trỏ đến chuỗi mới này.
                     // Chuỗi "Hello" gốc vẫn còn trên Heap (cho đến khi GC dọn dẹp nếu không còn tham chiếu nào khác).

Console.WriteLine($"Sau thay đổi: s1 = {s1}, s2 = {s2}"); // Output: Sau thay đổi: s1 = Hello World, s2 = Hello
```
Trong ví dụ trên, `s2` vẫn trỏ đến đối tượng chuỗi "Hello" ban đầu, chứng minh rằng `s1 = s1 + " World"` đã tạo ra một đối tượng chuỗi mới thay vì sửa đổi đối tượng cũ.

Tính bất biến của chuỗi có nhiều lợi ích, bao gồm an toàn trong môi trường đa luồng và đơn giản hóa việc xử lý chuỗi.

### 2.5. Từ khóa `var` (Implicitly Typed Local Variables)

Trong C#, bạn có thể sử dụng từ khóa `var` để khai báo một biến cục bộ mà không cần chỉ định rõ ràng kiểu dữ liệu của nó. Trình biên dịch C# sẽ tự động suy luận kiểu dữ liệu dựa trên giá trị được gán cho biến tại thời điểm khai báo.

```csharp
var soLuong = 10;       // Trình biên dịch suy luận là int
var ten = "Alice";      // Trình biên dịch suy luận là string
var giaTri = 12.34f;    // Trình biên dịch suy luận là float (do có hậu tố 'f')
var isSuccess = true;   // Trình biên dịch suy luận là bool
var danhSach = new List<string>(); // Suy luận là List<string>
```

> [!TIP]
> **Lưu ý quan trọng khi sử dụng `var`:**
> *   `var` chỉ có thể được sử dụng cho các biến cục bộ (bên trong phương thức, vòng lặp, khối lệnh).
> *   Biến phải được khởi tạo ngay lập tức khi khai báo `var`. Nếu không có giá trị khởi tạo, trình biên dịch không thể suy luận kiểu.
> *   Kiểu dữ liệu được suy luận tại thời điểm biên dịch và không thể thay đổi sau đó. `var` không phải là một kiểu `dynamic`; nó chỉ là một cách viết tắt.
> *   Trình biên dịch mặc định suy luận các số nguyên không có hậu tố là `int` và số thực không có hậu tố là `double`. Ví dụ: `var num = 10;` sẽ là `int`, `var price = 10.5;` sẽ là `double`. Nếu bạn muốn `byte` hoặc `float`, bạn phải khai báo rõ ràng hoặc thêm hậu tố: `byte myByte = 10;` hoặc `var myFloat = 10.5f;`.
>
> **Khi nào nên dùng và không nên dùng `var`?**
> *   **Nên dùng:** Khi kiểu dữ liệu rõ ràng từ biểu thức gán (ví dụ: `var result = SomeMethodThatReturnsComplexType();` hoặc `var item = new MyComplexObject();`). Nó giúp mã ngắn gọn và dễ đọc hơn khi tên kiểu dài.
> *   **Không nên dùng:** Khi kiểu dữ liệu không rõ ràng từ ngữ cảnh hoặc khi việc khai báo kiểu tường minh làm tăng tính dễ đọc của mã. Mục tiêu là sự rõ ràng, không phải sự ngắn gọn bằng mọi giá.

## 3. Tràn số (Overflowing) và Kiểm soát

Hiện tượng tràn số xảy ra khi bạn cố gắng lưu trữ một giá trị vượt quá phạm vi mà kiểu dữ liệu của biến có thể chứa. Điều này có thể dẫn đến kết quả tính toán sai lệch hoặc hành vi không mong muốn.

### 3.1. Hiện tượng Tràn số

Mỗi kiểu dữ liệu số có một phạm vi giá trị nhất định. Khi bạn thực hiện một phép toán mà kết quả vượt ra ngoài phạm vi đó, hiện tượng tràn số sẽ xảy ra.

**Ví dụ về tràn số với kiểu `byte`:**

Kiểu `byte` có thể lưu trữ giá trị từ 0 đến 255. Nếu bạn cố gắng lưu trữ 256 vào một biến `byte`, điều gì sẽ xảy ra?

```csharp
byte number = 255;
Console.WriteLine($"Giá trị ban đầu: {number}"); // Output: 255

// Mặc định trong C#, các phép toán số học trên kiểu số nguyên là 'unchecked'.
// Khi một kiểu số nguyên bị tràn trong ngữ cảnh unchecked, giá trị sẽ "cuộn" trở lại từ đầu phạm vi của nó.
// 255 + 1 = 256. Vì byte chỉ đến 255, nó sẽ cuộn về 0.
number = (byte)(number + 1); // Ép kiểu về byte là cần thiết cho phép cộng.
Console.WriteLine($"Sau khi cộng 1 (unchecked): {number}"); // Output: 0 (đã tràn số)

// Tương tự, nếu giảm quá giới hạn dưới
byte anotherNumber = 0;
anotherNumber = (byte)(anotherNumber - 1); // 0 - 1 = -1. Cuộn về 255.
Console.WriteLine($"Sau khi trừ 1 (unchecked): {anotherNumber}"); // Output: 255
```

Hành vi "cuộn" này được gọi là hành vi **không kiểm tra tràn** (unchecked overflow). Nó có thể hữu ích trong một số trường hợp cụ thể (ví dụ: thuật toán hash), nhưng thường là không mong muốn và có thể dẫn đến lỗi logic nghiêm trọng trong các ứng dụng thông thường.

### 3.2. Kiểm soát Tràn số với `checked` và `unchecked`

C# cung cấp các từ khóa `checked` và `unchecked` để kiểm soát hành vi tràn số.

*   **`unchecked` (Mặc định):**
    *   Đây là hành vi mặc định cho hầu hết các phép toán số học trong C#. Khi tràn số xảy ra, giá trị sẽ "cuộn" theo vòng tròn như ví dụ trên, mà không gây ra lỗi.
    *   Bạn có thể tường minh sử dụng từ khóa `unchecked` cho một khối mã hoặc một biểu thức, nhưng thường không cần thiết vì nó là mặc định.

*   **`checked` (Kiểm tra tràn số):**
    *   Nếu bạn muốn ngăn chặn việc tràn số một cách im lặng và thay vào đó, muốn chương trình báo lỗi khi tràn xảy ra, bạn có thể sử dụng từ khóa `checked`.
    *   Khi một thao tác số học bên trong khối `checked` gây ra tràn số, C# sẽ ném ra một `OverflowException`. Điều này cho phép bạn bắt và xử lý ngoại lệ một cách có kiểm soát.

```csharp
Console.WriteLine("\n--- Minh họa kiểm soát Tràn số với 'checked' ---");
try
{
    checked // Bật kiểm tra tràn số cho khối mã này
    {
        byte maxByteValue = 255;
        Console.WriteLine($"Giá trị byte lớn nhất: {maxByteValue}");

        // Thao tác này sẽ gây tràn số và ném ra OverflowException
        byte checkedOverflowResult = (byte)(maxByteValue + 1);
        Console.WriteLine($"255 + 1 (checked byte): {checkedOverflowResult}"); // Dòng này sẽ không được thực thi
    }
}
catch (OverflowException ex)
{
    // Một ngoại lệ OverflowException sẽ được ném ra và được bắt ở đây
    Console.WriteLine($"Lỗi khi kiểm tra tràn số: {ex.Message}"); // Output: Lỗi khi kiểm tra tràn số: Arithmetic operation resulted in an overflow.
}

try
{
    checked // Kiểm tra tràn số cho một biểu thức cụ thể
    {
        int largeInt = int.MaxValue; // 2,147,483,647
        int result = largeInt + 1; // Sẽ ném OverflowException
        Console.WriteLine($"Kết quả: {result}");
    }
}
catch (OverflowException ex)
{
    Console.WriteLine($"Lỗi: {ex.Message}"); // Output: Lỗi: Arithmetic operation resulted in an overflow.
}
```

> [!NOTE]
> Việc xử lý ngoại lệ (`try-catch`) là một chủ đề nâng cao hơn, nhưng hiện tại, điều quan trọng là bạn biết rằng `checked` sẽ khiến chương trình của bạn "sập" (nếu không được xử lý) thay vì tiếp tục với một giá trị sai.

> [!TIP]
> **Thực tiễn tốt nhất để tránh tràn số:**
> 1.  **Chọn kiểu dữ liệu phù hợp:** Cách tốt nhất để tránh tràn số là chọn một kiểu dữ liệu có phạm vi đủ lớn ngay từ đầu. Ví dụ, nếu bạn không chắc chắn giá trị sẽ vượt quá 255, hãy sử dụng `int` thay vì `byte`.
> 2.  **Sử dụng `checked` khi cần thiết:** Đối với các phép tính quan trọng mà bạn muốn đảm bảo không có tràn số ngầm định (ví dụ: tính toán tài chính, chỉ số mảng), hãy sử dụng `checked` để buộc trình biên dịch kiểm tra và ném ngoại lệ.
> 3.  **Kiểm tra giá trị đầu vào:** Luôn kiểm tra các giá trị đầu vào từ người dùng hoặc từ các nguồn bên ngoài để đảm bảo chúng nằm trong phạm vi mong đợi trước khi thực hiện các phép toán.

## 4. Phạm vi (Scope) của Biến và Hằng số

**Phạm vi** của một biến hoặc hằng số là vùng trong mã chương trình mà biến hoặc hằng số đó có thể được truy cập và có ý nghĩa. Trong C#, phạm vi được xác định chủ yếu bởi các khối mã (code blocks), được biểu thị bằng cặp dấu ngoặc nhọn `{}`.

### 4.1. Định nghĩa Phạm vi

Mỗi khi bạn mở một cặp dấu ngoặc nhọn `{` và đóng nó bằng `}`, bạn đang định nghĩa một khối mã mới, và khối mã này tạo ra một phạm vi mới. Ví dụ: thân của một phương thức, thân của một câu lệnh `if`, một vòng lặp `for`, `while`, hoặc một khối `try-catch` đều là các khối mã.

### 4.2. Quy tắc Phạm vi

*   **Khả năng truy cập:** Một biến hoặc hằng số chỉ có thể được truy cập từ bên trong khối mã mà nó được khai báo, hoặc từ bất kỳ khối mã con nào nằm bên trong khối đó.
*   **Không thể truy cập bên ngoài:** Bạn không thể truy cập một biến hoặc hằng số từ bên ngoài khối mã nơi nó được định nghĩa.
*   **Vòng đời biến và giải phóng bộ nhớ:**
    *   Các biến cục bộ (local variables) kiểu giá trị được khai báo trong một phạm vi sẽ được cấp phát trên Stack khi phạm vi đó được khởi tạo.
    *   Khi quá trình thực thi rời khỏi phạm vi đó (ví dụ: kết thúc một phương thức hoặc một vòng lặp), các biến này sẽ tự động bị hủy khỏi Stack, giải phóng bộ nhớ. Điều này giúp quản lý tài nguyên bộ nhớ hiệu quả.
    *   Đối với biến kiểu tham chiếu, biến trên Stack (chứa địa chỉ) cũng bị hủy, và nếu không còn tham chiếu nào khác đến đối tượng trên Heap, đối tượng đó sẽ trở thành đối tượng đủ điều kiện để Garbage Collector dọn dẹp.

### 4.3. Ví dụ Minh họa Phạm vi

```csharp
class Program
{
    // Hằng số cấp lớp có phạm vi toàn bộ lớp Program
    const string APP_VERSION = "1.0.0";

    static void Main(string[] args)
    {
        // Khối 1: Phạm vi của phương thức Main
        // Biến 'a' được khai báo trong khối Main, có thể truy cập trong toàn bộ Main
        int a = 10;
        Console.WriteLine($"Trong Main (trước if): a = {a}, Version = {APP_VERSION}"); // APP_VERSION có thể truy cập

        if (a > 5)
        {
            // Khối 2: Phạm vi của câu lệnh if
            // Biến 'b' được khai báo trong khối if, chỉ tồn tại trong khối này
            int b = 20;
            Console.WriteLine($"Trong khối if: a = {a}, b = {b}"); // 'a' (từ phạm vi cha) và 'b' đều có thể truy cập
        } // Khi thoát khỏi khối if, biến 'b' bị hủy

        // Console.WriteLine($"Ngoài khối if: b = {b}"); // Lỗi biên dịch: 'b' không thể truy cập ở đây
                                                      // vì nó nằm ngoài phạm vi khai báo của nó (khối if)

        for (int i = 0; i < 2; i++)
        {
            // Khối 3: Phạm vi của vòng lặp for
            // Biến 'i' và 'loopMessage' được khai báo trong khối for, chỉ tồn tại trong vòng lặp
            string loopMessage = $"Lặp thứ {i}";
            Console.WriteLine($"Trong vòng lặp: {loopMessage}, a = {a}, i = {i}"); // 'a' (từ phạm vi cha), 'loopMessage', 'i' đều có thể truy cập
        } // Khi vòng lặp kết thúc, biến 'i' và 'loopMessage' bị hủy

        // Console.WriteLine($"Ngoài vòng lặp: loopMessage = {loopMessage}"); // Lỗi biên dịch: 'loopMessage' không thể truy cập
        // Console.WriteLine($"Ngoài vòng lặp: i = {i}");                   // Lỗi biên dịch: 'i' không thể truy cập

        Console.WriteLine($"Trong Main (cuối): a = {a}"); // 'a' vẫn có thể truy cập
    } // Khi phương thức Main kết thúc, biến 'a' bị hủy
}
```

> [!TIP]
> Hiểu rõ phạm vi giúp bạn:
> *   **Quản lý tài nguyên bộ nhớ:** Các biến không còn cần thiết sẽ tự động bị loại bỏ, ngăn chặn lãng phí bộ nhớ.
> *   **Tránh xung đột tên:** Bạn có thể sử dụng cùng một tên biến trong các phạm vi khác nhau mà không gây nhầm lẫn.
> *   **Tăng tính module hóa:** Các khối mã có thể hoạt động độc lập hơn mà không cần lo lắng về các biến bên ngoài.

---

## 5. Ví dụ Thực hành Tổng hợp

Hãy cùng xem một ví dụ C# đầy đủ để minh họa tất cả các khái niệm đã học trong phần này một cách mạch lạc.

```csharp
using System; // Cần thiết để sử dụng Console và các kiểu dữ liệu cơ bản

// Định nghĩa một lớp (class) là kiểu tham chiếu
class ThongTinNguoiDung
{
    public string Ten { get; set; }
    public int Tuoi { get; set; }

    public ThongTinNguoiDung(string ten, int tuoi)
    {
        Ten = ten;
        Tuoi = tuoi;
    }

    public void HienThiThongTin()
    {
        Console.WriteLine($"  Tên: {Ten}, Tuổi: {Tuoi}");
    }
}

class ChuongBienHangSoPhamVi
{
    // Khai báo hằng số cấp lớp (PascalCase)
    const float TY_GIA_THUE = 0.10f; // 10% thuế
    const int SO_LUONG_TOI_DA_SAN_PHAM = 1000;
    const string TEN_CONG_TY = "ABC Solutions";

    static void Main(string[] args)
    {
        Console.WriteLine("--- Bắt đầu Chương 2: Biến, Hằng số và Phạm vi ---");
        Console.WriteLine($"Tên công ty: {TEN_CONG_TY}\n");

        // 1. Khai báo và khởi tạo biến (Value Types)
        // Sử dụng camelCase cho biến cục bộ
        int soLuongHangTonKho = 500; // int là kiểu mặc định cho số nguyên
        Console.WriteLine($"1. Hàng tồn kho ban đầu: {soLuongHangTonKho}");

        float giaDonVi = 12.50f; // Cần hậu tố 'f' cho float
        Console.WriteLine($"   Giá đơn vị: {giaDonVi}");

        decimal tongDoanhThu = 0.00m; // Cần hậu tố 'm' cho decimal (chính xác cao cho tài chính)
        Console.WriteLine($"   Tổng doanh thu ban đầu: {tongDoanhThu}");

        char maPhanLoai = 'A'; // Ký tự dùng dấu nháy đơn
        Console.WriteLine($"   Mã phân loại: {maPhanLoai}");

        bool sanPhamDangHoatDong = true;
        Console.WriteLine($"   Sản phẩm đang hoạt động: {sanPhamDangHoatDong}\n");

        // 2. Sử dụng từ khóa 'var'
        var tenSanPham = "Laptop XYZ"; // Suy luận là string
        var soSerial = 123456789L;     // Suy luận là long do hậu tố 'L'
        var tyLeGiamGia = 0.15;        // Suy luận là double (mặc định cho số thực không hậu tố)
        Console.WriteLine($"2. Tên sản phẩm: {tenSanPham}, Serial: {soSerial}, Giảm giá: {tyLeGiamGia * 100}%");

        // 3. Khai báo và khởi tạo biến (Reference Type: class)
        ThongTinNguoiDung nguoiDung1 = new ThongTinNguoiDung("Nguyen Thi B", 25);
        Console.Write("3. Người dùng 1: ");
        nguoiDung1.HienThiThongTin();

        // Minh họa sao chép tham chiếu
        ThongTinNguoiDung nguoiDung2 = nguoiDung1; // nv2 và nv1 cùng trỏ đến MỘT đối tượng trên Heap
        Console.Write("   Người dùng 2 (ban đầu): ");
        nguoiDung2.HienThiThongTin();

        nguoiDung1.Tuoi = 26; // Thay đổi qua nguoiDung1 sẽ ảnh hưởng đến đối tượng mà nguoiDung2 đang trỏ tới
        Console.Write("   Người dùng 1 (sau đổi tuổi): ");
        nguoiDung1.HienThiThongTin();
        Console.Write("   Người dùng 2 (sau đổi tuổi của nguoiDung1): ");
        nguoiDung2.HienThiThongTin(); // Tuổi của nguoiDung2 cũng thay đổi
        Console.WriteLine();

        // 4. Hằng số
        Console.WriteLine($"4. Tỷ giá thuế: {TY_GIA_THUE * 100}%");
        Console.WriteLine($"   Số lượng sản phẩm tối đa: {SO_LUONG_TOI_DA_SAN_PHAM}");
        // TY_GIA_THUE = 0.12f; // Lỗi biên dịch: Không thể thay đổi giá trị hằng số

        // 5. Tràn số (Overflowing)
        Console.WriteLine("\n--- 5. Minh họa Tràn số ---");
        byte soLuongNho = 250;
        Console.WriteLine($"   Giá trị byte ban đầu: {soLuongNho}");

        // Unchecked (mặc định): Giá trị cuộn về
        soLuongNho = (byte)(soLuongNho + 10); // 250 + 10 = 260. Sẽ cuộn về 260 - 256 = 4.
        Console.WriteLine($"   (Unchecked) 250 + 10 = {soLuongNho}"); // Output: 4

        // Checked: Ném ngoại lệ
        try
        {
            checked
            {
                byte kiemTraSoLuong = 250;
                Console.WriteLine($"   Kiểm tra số lượng ban đầu: {kiemTraSoLuong}");
                kiemTraSoLuong = (byte)(kiemTraSoLuong + 10); // Sẽ ném OverflowException
                Console.WriteLine($"   (Checked) 250 + 10 = {kiemTraSoLuong}");
            }
        }
        catch (OverflowException ex)
        {
            Console.WriteLine($"   Lỗi khi kiểm tra tràn số: {ex.Message}");
        }
        Console.WriteLine();

        // 6. Phạm vi (Scope)
        Console.WriteLine("--- 6. Minh họa Phạm vi ---");
        int tongSoHang = 100; // Biến trong phạm vi Main
        Console.WriteLine($"   Trong Main: tongSoHang = {tongSoHang}");

        if (tongSoHang > 50)
        {
            // Bắt đầu một khối phạm vi mới
            int soLuongDatHang = 20; // Biến trong phạm vi if
            Console.WriteLine($"   Trong khối if: tongSoHang = {tongSoHang}, soLuongDatHang = {soLuongDatHang}");
            tongSoHang -= soLuongDatHang; // Thay đổi tongSoHang
        } // soLuongDatHang bị hủy khi thoát khỏi khối if

        // Console.WriteLine(soLuongDatHang); // Lỗi biên dịch: soLuongDatHang nằm ngoài phạm vi

        for (int i = 0; i < 3; i++)
        {
            // Biến 'i' và 'thongBao' chỉ tồn tại trong phạm vi vòng lặp for
            string thongBao = $"Đang xử lý đơn hàng thứ {i + 1}";
            Console.WriteLine($"   Trong vòng lặp: {thongBao}, tongSoHang = {tongSoHang}");
        }
        // Console.WriteLine(i); // Lỗi biên dịch: i nằm ngoài phạm vi
        // Console.WriteLine(thongBao); // Lỗi biên dịch: thongBao nằm ngoài phạm vi

        Console.WriteLine($"   Ngoài các khối con: tongSoHang = {tongSoHang}"); // tongSoHang vẫn khả dụng và đã được cập nhật
        Console.WriteLine();

        // 7. Minh họa các giá trị Min/Max của kiểu dữ liệu
        Console.WriteLine("--- 7. Phạm vi giá trị của các kiểu ---");
        Console.WriteLine($"   Byte: Min = {byte.MinValue}, Max = {byte.MaxValue}");
        Console.WriteLine($"   Int: Min = {int.MinValue}, Max = {int.MaxValue}");
        Console.WriteLine($"   Float: Min = {float.MinValue}, Max = {float.MaxValue}");
        Console.WriteLine($"   Decimal: Min = {decimal.MinValue}, Max = {decimal.MaxValue}");
        Console.WriteLine($"   Char: Min = {(int)char.MinValue} ({char.MinValue}), Max = {(int)char.MaxValue} ({char.MaxValue})");

        Console.WriteLine("\n--- Kết thúc Chương 2 ---");
    }
}
```

---

## Tóm tắt Chương 2: Biến, Hằng số và Phạm vi

*   **Biến (Variables):** Là vùng nhớ được đặt tên để lưu trữ dữ liệu có thể thay đổi trong quá trình thực thi chương trình. Cần được khai báo với kiểu dữ liệu và khởi tạo trước khi sử dụng để tránh lỗi biên dịch.
*   **Hằng số (Constants):** Là giá trị cố định, không thể thay đổi sau khi được khởi tạo. Được sử dụng để tăng cường tính an toàn, rõ ràng và dễ bảo trì cho các giá trị không đổi trong ứng dụng. Phải được khởi tạo ngay lập tức khi khai báo.
*   **Quy tắc và Quy ước đặt tên (Identifiers & Naming Conventions):**
    *   **Quy tắc bắt buộc:** Tên không bắt đầu bằng số, không chứa khoảng trắng, không trùng từ khóa C#. C# là ngôn ngữ phân biệt chữ hoa chữ thường.
    *   **Quy ước:** `camelCase` cho biến cục bộ và tham số; `PascalCase` cho hằng số, tên lớp, phương thức, thuộc tính. Tránh ký hiệu Hungarian.
*   **Kiểu dữ liệu (Data Types):** Xác định loại giá trị và lượng bộ nhớ mà biến có thể lưu trữ.
    *   **Kiểu Giá trị (Value Types):** Trực tiếp chứa dữ liệu, được lưu trữ trên **Stack**. Khi gán, tạo ra một bản sao giá trị độc lập. Ví dụ: `int`, `float`, `bool`, `char`, `struct`, `enum`.
    *   **Kiểu Tham chiếu (Reference Types):** Chứa địa chỉ (tham chiếu) đến dữ liệu thực tế, được lưu trữ trên **Heap**. Biến tham chiếu trên Stack, đối tượng trên Heap. Khi gán, chỉ sao chép địa chỉ tham chiếu, hai biến cùng trỏ đến một đối tượng. Ví dụ: `string`, `class`, `array`.
    *   **`string`:** Là một kiểu tham chiếu nhưng có tính chất bất biến.
*   **`var` keyword:** Cho phép trình biên dịch tự động suy luận kiểu dữ liệu của biến cục bộ dựa trên giá trị khởi tạo. Tiện lợi nhưng cần sử dụng hợp lý để duy trì tính rõ ràng của mã.
*   **Tràn số (Overflowing):** Xảy ra khi một giá trị vượt quá phạm vi của kiểu dữ liệu. Mặc định C# không kiểm tra tràn (`unchecked`), dẫn đến giá trị "cuộn" lại.
*   **`checked` keyword:** Dùng để bật kiểm tra tràn số. Nếu tràn xảy ra trong khối `checked`, một `OverflowException` sẽ được ném ra, cho phép xử lý lỗi một cách có kiểm soát.
*   **Phạm vi (Scope):** Là vùng trong mã mà một biến hoặc hằng số có thể được truy cập. Được xác định bởi các khối mã (`{}`), biến chỉ có thể truy cập trong khối nó được khai báo và các khối con của nó. Phạm vi ảnh hưởng đến vòng đời của biến và việc giải phóng bộ nhớ.

<!-- REVIEWED_BY_AGENT -->
