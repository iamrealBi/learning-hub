# Phần 12: Thao tác với Tệp và Thư mục

## Giới thiệu: Giao tiếp với Hệ thống Tệp

Trong thế giới lập trình hiện đại, khả năng tương tác với hệ thống tệp là một kỹ năng nền tảng và không thể thiếu. Từ các ứng dụng di động lưu trữ cài đặt người dùng, máy chủ web đọc và ghi dữ liệu, cho đến các hệ thống AI thông minh cần truy cập và quản lý thông tin, mọi thứ đều xoay quanh việc thao tác tệp và thư mục. Nắm vững các kỹ thuật này không chỉ giúp bạn xây dựng ứng dụng mạnh mẽ mà còn cung cấp cái nhìn sâu sắc về cách các hệ thống phức tạp, bao gồm cả các công cụ AI lập trình như Antigravity IDE, tương tác với môi trường làm việc của chúng.

Phần này sẽ trang bị cho bạn kiến thức và kỹ năng cần thiết để thực hiện các thao tác cơ bản và nâng cao với tệp và thư mục một cách hiệu quả trong C#. Chúng ta sẽ đi sâu vào không gian tên `System.IO` mạnh mẽ, khám phá sự khác biệt quan trọng về kiến trúc và hiệu suất giữa các lớp cung cấp phương thức tĩnh và các lớp cung cấp phương thức cá thể, cũng như cách xử lý đường dẫn tệp một cách thông minh và an toàn.

Mục tiêu của phần này là giúp bạn:

*   Hiểu và sử dụng không gian tên `System.IO` để tương tác với hệ thống tệp.
*   Nắm vững kiến trúc và cách sử dụng các lớp `File`, `FileInfo`, `Directory`, `DirectoryInfo` và `Path`.
*   Phân biệt sâu sắc cơ chế hoạt động, ưu nhược điểm của phương thức tĩnh và phương thức cá thể, đặc biệt về hiệu suất và quản lý tài nguyên.
*   Thực hiện các thao tác cơ bản như tạo, đọc, ghi, sao chép, di chuyển, xóa tệp và thư mục.
*   Xử lý các đường dẫn tệp một cách linh hoạt, mạnh mẽ và tương thích đa nền tảng.
*   Hiểu vai trò của thao tác tệp trong ngữ cảnh của các hệ thống AI lập trình như Antigravity IDE.

## 1. Không gian tên System.IO: Cửa ngõ đến Hệ thống Tệp và Luồng Dữ liệu

Trong .NET, mọi hoạt động liên quan đến hệ thống tệp và các luồng dữ liệu (Input/Output - I/O) đều được tổ chức và quản lý trong không gian tên `System.IO`. Đây là một thư viện phong phú, cung cấp các khối xây dựng cơ bản để ứng dụng của bạn có thể "giao tiếp" với môi trường bên ngoài, dù đó là đĩa cứng, bộ nhớ, hay mạng lưới.

Trong phần này, chúng ta sẽ tập trung vào các lớp phổ biến và quan trọng nhất cho việc thao tác tệp và thư mục cục bộ: `File`, `FileInfo`, `Directory`, `DirectoryInfo` và `Path`.

### 1.1. Lớp `File` và `FileInfo`: Hai cách tiếp cận thao tác tệp

Cả hai lớp `File` và `FileInfo` đều cung cấp các phương thức để thực hiện các thao tác phổ biến trên tệp như tạo, sao chép, di chuyển, xóa và đọc/ghi nội dung. Tuy nhiên, chúng khác nhau cơ bản về kiến trúc và cách thức cung cấp các phương thức, dẫn đến sự khác biệt đáng kể về hiệu suất và cách sử dụng.

#### 1.1.1. Kiến trúc và cơ chế hoạt động

*   **Lớp `File` (Phương thức tĩnh - Static Methods):**
    *   **Kiến trúc:** `File` là một lớp tĩnh (static class). Mọi phương thức của nó đều là phương thức tĩnh, có nghĩa là bạn gọi chúng trực tiếp thông qua tên lớp mà không cần tạo một đối tượng của lớp `File`.
    *   **Cơ chế:** Mỗi khi bạn gọi một phương thức tĩnh của `File` (ví dụ: `File.Exists(path)`, `File.ReadAllText(path)`), hệ thống sẽ thực hiện một loạt các bước:
        1.  **Phân tích đường dẫn:** Chuỗi đường dẫn (một kiểu tham chiếu - `string`) được truyền vào sẽ được phân tích để xác định vị trí tệp trên đĩa.
        2.  **Kiểm tra bảo mật và quyền truy cập:** Hệ điều hành kiểm tra xem ứng dụng hiện tại có đủ quyền để thực hiện thao tác được yêu cầu trên tệp tại đường dẫn đó hay không.
        3.  **Truy cập hệ thống tệp:** Sau khi các kiểm tra hoàn tất, thao tác thực tế (kiểm tra tồn tại, đọc, ghi, v.v.) mới được thực hiện thông qua các API của hệ điều hành.
    *   **Bộ nhớ:** Các phương thức tĩnh không tạo ra đối tượng trên heap để đại diện cho tệp. Chúng chỉ hoạt động dựa trên chuỗi đường dẫn được cung cấp.

*   **Lớp `FileInfo` (Phương thức cá thể - Instance Methods):**
    *   **Kiến trúc:** `FileInfo` là một lớp thông thường (không tĩnh). Để sử dụng các phương thức của nó, bạn cần tạo một đối tượng `FileInfo` bằng cách truyền đường dẫn của tệp vào hàm tạo (constructor).
    *   **Cơ chế:** Khi bạn tạo một đối tượng `FileInfo` (ví dụ: `new FileInfo(path)`):
        1.  **Phân tích đường dẫn và khởi tạo đối tượng:** Chuỗi đường dẫn được phân tích **một lần** và được lưu trữ bên trong đối tượng `FileInfo` trên heap. Các kiểm tra bảo mật ban đầu có thể được thực hiện hoặc hoãn lại đến khi cần thiết.
        2.  **Đại diện cho tệp:** Đối tượng `FileInfo` trở thành một "đại diện" trong bộ nhớ cho tệp vật lý trên đĩa. Nó có các thuộc tính (ví dụ: `Length`, `CreationTime`, `Exists`) và các phương thức (ví dụ: `Delete()`, `CopyTo()`) để thao tác với tệp mà nó đại diện.
        3.  **Hiệu quả khi thao tác nhiều lần:** Khi bạn gọi các phương thức cá thể hoặc truy cập thuộc tính của đối tượng `FileInfo` này, hệ thống không cần lặp lại việc phân tích đường dẫn và các kiểm tra bảo mật ban đầu (trừ khi cần cập nhật thông tin). Đối tượng đã có ngữ cảnh và thông tin cần thiết.
    *   **Bộ nhớ:** `FileInfo` là một **kiểu tham chiếu (reference type)**. Khi bạn tạo `new FileInfo(path)`, một đối tượng được cấp phát trên **heap**. Biến `fileInfo` sẽ chứa một tham chiếu (địa chỉ bộ nhớ) đến đối tượng này. Các thuộc tính như `Length`, `CreationTime` cũng được lưu trữ trong đối tượng này và có thể được cập nhật bằng phương thức `Refresh()`.

#### 1.1.2. So sánh hiệu suất và ứng dụng thực tế

**1. Ưu điểm của `File` (phương thức tĩnh):**

*   **Tiện lợi cho thao tác đơn lẻ:** Nếu bạn chỉ cần thực hiện một hoặc hai thao tác trên một tệp (ví dụ: kiểm tra tồn tại rồi đọc nội dung một lần), việc gọi trực tiếp `File.Exists()` hoặc `File.ReadAllText()` sẽ nhanh chóng và dễ dàng hơn vì bạn không cần phải tạo và quản lý một đối tượng `FileInfo`.
*   **Không tạo đối tượng:** Tránh được chi phí nhỏ của việc cấp phát đối tượng trên heap.

**2. Nhược điểm của `File` (phương thức tĩnh):**

*   **Hiệu suất kém hơn khi thực hiện nhiều thao tác:** Mỗi khi bạn gọi một phương thức tĩnh của `File`, hệ điều hành sẽ thực hiện đầy đủ các bước phân tích đường dẫn và kiểm tra bảo mật/quyền truy cập. Nếu bạn thực hiện hàng chục hoặc hàng trăm thao tác trên cùng một tệp, những kiểm tra lặp đi lặp lại này có thể tạo ra chi phí đáng kể, ảnh hưởng đến hiệu suất của ứng dụng.

**3. Ưu điểm của `FileInfo` (phương thức cá thể):**

*   **Hiệu quả cho nhiều thao tác:** Khi bạn tạo một đối tượng `FileInfo`, các kiểm tra bảo mật ban đầu và việc thu thập thông tin về tệp chỉ được thực hiện **một lần** trong quá trình khởi tạo đối tượng (hoặc khi truy cập thuộc tính lần đầu). Sau đó, tất cả các phương thức cá thể được gọi trên đối tượng `FileInfo` đó sẽ hoạt động hiệu quả hơn vì chúng đã có ngữ cảnh và quyền truy cập đã được xác thực, giảm thiểu các cuộc gọi lặp lại đến hệ điều hành.
*   **Cung cấp thuộc tính:** `FileInfo` cung cấp các thuộc tính (ví dụ: `Length`, `CreationTime`, `LastWriteTime`, `IsReadOnly`) giúp bạn dễ dàng truy cập thông tin về tệp mà không cần gọi các phương thức riêng biệt.
*   **Khả năng cập nhật trạng thái:** Nếu tệp trên đĩa thay đổi sau khi đối tượng `FileInfo` được tạo, bạn có thể gọi phương thức `Refresh()` để cập nhật các thuộc tính của đối tượng với trạng thái hiện tại của tệp.

**4. Nhược điểm của `FileInfo` (phương thức cá thể):**

*   **Phải tạo đối tượng:** Đối với các thao tác đơn giản, việc tạo một đối tượng `FileInfo` trên heap có thể là một chi phí nhỏ không cần thiết.

> [!TIP]
> **Quy tắc vàng khi lựa chọn:**
> *   Sử dụng lớp **`File`** khi bạn chỉ cần thực hiện **một hoặc một vài thao tác độc lập** trên một tệp.
> *   Sử dụng lớp **`FileInfo`** khi bạn cần thực hiện **nhiều thao tác liên tiếp** trên cùng một tệp để tối ưu hiệu suất và tận dụng các thuộc tính.

### 1.2. Lớp `Directory` và `DirectoryInfo`: Thao tác với Thư mục

Tương tự như `File` và `FileInfo`, các lớp `Directory` và `DirectoryInfo` cũng hoạt động theo cặp với các phương thức tĩnh và cá thể để thao tác với thư mục:

*   **Lớp `Directory`**: Cung cấp các **phương thức tĩnh** để làm việc với các thư mục (ví dụ: tạo, xóa, kiểm tra sự tồn tại, lấy danh sách tệp/thư mục con).
*   **Lớp `DirectoryInfo`**: Cung cấp các **phương thức cá thể** cho các thao tác tương tự. Bạn cần tạo một đối tượng `DirectoryInfo` (một kiểu tham chiếu được cấp phát trên heap) để sử dụng chúng.

Nguyên tắc về hiệu suất, cơ chế cấp phát bộ nhớ (kiểu tham chiếu trên heap cho `DirectoryInfo`), và tiện lợi khi lựa chọn giữa phương thức tĩnh và cá thể cũng hoàn toàn tương tự như đối với `File` và `FileInfo`. `DirectoryInfo` cũng có phương thức `Refresh()` để cập nhật các thuộc tính của nó nếu thư mục trên đĩa thay đổi.

### 1.3. Lớp `Path`: Xử lý đường dẫn thông minh

Lớp `Path` là một lớp tĩnh (static class) cung cấp các phương thức tiện ích để xử lý các chuỗi đường dẫn tệp hoặc thư mục. Thay vì phải tự mình xử lý chuỗi (ví dụ: tìm vị trí dấu chấm để lấy phần mở rộng), lớp `Path` cung cấp các phương thức đơn giản, đáng tin cậy và **tương thích đa nền tảng** để trích xuất các thành phần khác nhau của một đường dẫn hoặc kết hợp chúng lại một cách an toàn.

Sử dụng lớp `Path` là cách tốt nhất để tránh các lỗi xử lý chuỗi thủ công, đặc biệt khi ứng dụng của bạn có thể chạy trên các hệ điều hành khác nhau (ví dụ: Windows sử dụng dấu gạch chéo ngược `\` còn Linux/macOS sử dụng dấu gạch chéo thuận `/`).

## 2. Thao tác với Tệp (File Operations)

Phần này sẽ đi sâu vào các ví dụ thực tế về cách sử dụng `File` và `FileInfo` để thực hiện các thao tác phổ biến với tệp.

> [!NOTE]
> Trong các ví dụ dưới đây, chúng ta sẽ sử dụng **chuỗi ký tự nguyên văn (verbatim string literal)** bằng cách thêm `@` vào đầu chuỗi đường dẫn. Điều này giúp tránh việc phải sử dụng dấu gạch chéo ngược kép `\\` để thoát ký tự `\`, làm cho đường dẫn dễ đọc hơn. Ví dụ: thay vì `"C:\\temp\\myfile.txt"`, chúng ta viết `@"C:\temp\myfile.txt"`.

```csharp
using System;
using System.IO; // Không gian tên chính cho thao tác tệp và thư mục

namespace FileOperations
{
    class Program
    {
        static void Main(string[] args)
        {
            // Định nghĩa đường dẫn cơ sở cho các tệp demo
            string basePath = @"C:\Temp\FileDemo"; // Sử dụng một thư mục riêng để tránh xung đột
            string sourceFilePath = Path.Combine(basePath, "MyFile.txt");
            string destinationFilePath = Path.Combine(basePath, "MyFile.copy.txt");
            string movedFilePath = Path.Combine(basePath, "MyFile.moved.txt");
            string content = "Đây là nội dung của tệp demo.\nThêm một dòng nữa.";

            // Đảm bảo thư mục cơ sở tồn tại và sạch sẽ trước khi chạy các demo
            if (Directory.Exists(basePath))
            {
                Directory.Delete(basePath, true); // Xóa đệ quy nếu tồn tại
            }
            Directory.CreateDirectory(basePath); // Tạo lại thư mục
            Console.WriteLine($"Đã chuẩn bị thư mục demo: {basePath}");

            Console.WriteLine("\n--- Thao tác với Tệp sử dụng lớp File (phương thức tĩnh) ---");
            PerformStaticFileOperations(sourceFilePath, destinationFilePath, movedFilePath, content);

            // Dọn dẹp lại cho demo FileInfo
            if (Directory.Exists(basePath))
            {
                Directory.Delete(basePath, true);
            }
            Directory.CreateDirectory(basePath);
            Console.WriteLine($"\nĐã chuẩn bị lại thư mục demo cho FileInfo: {basePath}");

            Console.WriteLine("\n--- Thao tác với Tệp sử dụng lớp FileInfo (phương thức cá thể) ---");
            PerformInstanceFileOperations(sourceFilePath, destinationFilePath, movedFilePath, content);

            // Dọn dẹp cuối cùng
            if (Directory.Exists(basePath))
            {
                Directory.Delete(basePath, true);
                Console.WriteLine($"\nĐã dọn dẹp thư mục demo: {basePath}");
            }
        }

        static void PerformStaticFileOperations(string sourcePath, string destinationPath, string newPathForMove, string content)
        {
            try
            {
                // 1. Tạo tệp và ghi nội dung
                // File.WriteAllText: Ghi toàn bộ chuỗi vào tệp. Nếu tệp không tồn tại, nó sẽ được tạo. Nếu tồn tại, nó sẽ bị ghi đè.
                File.WriteAllText(sourcePath, content);
                Console.WriteLine($"[File] Tạo và ghi nội dung vào tệp: {sourcePath}");

                // 2. Kiểm tra sự tồn tại của tệp
                if (File.Exists(sourcePath)) // Mỗi lần gọi Exists, hệ thống lại kiểm tra quyền và đường dẫn
                {
                    Console.WriteLine($"[File] Tệp tồn tại: {sourcePath}");
                }
                else
                {
                    Console.WriteLine($"[File] Tệp không tồn tại: {sourcePath}");
                }

                // 3. Sao chép tệp
                // Tham số thứ ba (true) cho phép ghi đè tệp đích nếu nó đã tồn tại.
                File.Copy(sourcePath, destinationPath, true); // Mỗi lần Copy, hệ thống lại kiểm tra quyền và đường dẫn
                Console.WriteLine($"[File] Sao chép tệp từ '{sourcePath}' sang '{destinationPath}'");

                // 4. Đọc tất cả nội dung văn bản từ tệp
                string readContent = File.ReadAllText(sourcePath); // Mỗi lần ReadAllText, hệ thống lại kiểm tra quyền và đường dẫn
                Console.WriteLine($"[File] Nội dung của tệp '{sourcePath}':\n{readContent}");

                // 5. Di chuyển tệp
                // Di chuyển tệp từ vị trí cũ sang vị trí mới. Nếu tệp đích đã tồn tại, sẽ gây lỗi IOException.
                // Do đó, cần xóa tệp đích nếu nó tồn tại trước khi di chuyển.
                if (File.Exists(newPathForMove))
                {
                    File.Delete(newPathForMove);
                }
                File.Move(sourcePath, newPathForMove); // Mỗi lần Move, hệ thống lại kiểm tra quyền và đường dẫn
                Console.WriteLine($"[File] Di chuyển tệp từ '{sourcePath}' sang '{newPathForMove}'");

                // 6. Xóa tệp
                File.Delete(destinationPath); // Xóa tệp đã sao chép
                Console.WriteLine($"[File] Xóa tệp: {destinationPath}");

                File.Delete(newPathForMove); // Xóa tệp đã di chuyển để dọn dẹp môi trường demo
                Console.WriteLine($"[File] Xóa tệp đã di chuyển: {newPathForMove}");
            }
            catch (Exception ex)
            {
                // Xử lý các ngoại lệ thường gặp:
                // - IOException: Tệp đang được sử dụng, đường dẫn không hợp lệ, đĩa đầy, v.v.
                // - UnauthorizedAccessException: Không có quyền truy cập.
                // - FileNotFoundException: Tệp nguồn không tìm thấy.
                // - DirectoryNotFoundException: Thư mục đích không tìm thấy.
                Console.WriteLine($"[File] Lỗi khi thao tác tệp tĩnh: {ex.Message}");
            }
        }

        static void PerformInstanceFileOperations(string sourcePath, string destinationPath, string newPathForMove, string content)
        {
            try
            {
                // Để sử dụng FileInfo, trước tiên cần đảm bảo tệp gốc tồn tại
                File.WriteAllText(sourcePath, content);
                Console.WriteLine($"[FileInfo] Tạo và ghi nội dung vào tệp (để khởi tạo FileInfo): {sourcePath}");

                // Tạo đối tượng FileInfo: Đối tượng này được cấp phát trên heap và đại diện cho tệp.
                // Các kiểm tra ban đầu và phân tích đường dẫn xảy ra ở đây.
                FileInfo fileInfo = new FileInfo(sourcePath);
                Console.WriteLine($"[FileInfo] Tạo đối tượng FileInfo cho: {sourcePath}");

                // 1. Kiểm tra sự tồn tại của tệp và truy cập các thuộc tính
                // Các thuộc tính như Exists, Length, CreationTime được truy cập từ đối tượng đã tạo,
                // không cần lặp lại phân tích đường dẫn và kiểm tra quyền như File.Exists().
                if (fileInfo.Exists)
                {
                    Console.WriteLine($"[FileInfo] Tệp tồn tại: {fileInfo.FullName}");
                    Console.WriteLine($"[FileInfo] Kích thước tệp: {fileInfo.Length} bytes");
                    Console.WriteLine($"[FileInfo] Thời gian tạo: {fileInfo.CreationTime}");
                    Console.WriteLine($"[FileInfo] Thư mục chứa tệp: {fileInfo.DirectoryName}");
                }
                else
                {
                    Console.WriteLine($"[FileInfo] Tệp không tồn tại: {fileInfo.FullName}");
                }

                // Cập nhật thông tin nếu tệp đã thay đổi bên ngoài
                // Ví dụ: nếu một tiến trình khác sửa đổi tệp sau khi fileInfo được tạo.
                // fileInfo.Refresh(); 

                // 2. Sao chép tệp
                // Phương thức CopyTo của FileInfo yêu cầu tệp đích không được tồn tại hoặc phải xử lý ghi đè thủ công
                // (có overload CopyTo(string destFileName, bool overwrite) từ .NET 5 trở lên).
                // Trong ví dụ này, chúng ta sẽ xóa tệp đích nếu nó tồn tại trước.
                FileInfo destinationFileInfo = new FileInfo(destinationPath);
                if (destinationFileInfo.Exists)
                {
                    destinationFileInfo.Delete();
                    Console.WriteLine($"[FileInfo] Đã xóa tệp đích cũ: {destinationFileInfo.FullName}");
                }
                fileInfo.CopyTo(destinationPath); // Sao chép bằng phương thức cá thể
                Console.WriteLine($"[FileInfo] Sao chép tệp từ '{fileInfo.FullName}' sang '{destinationPath}'");

                // 3. Di chuyển tệp
                // Tương tự, MoveTo sẽ gây lỗi nếu tệp đích đã tồn tại.
                FileInfo newPathFileInfo = new FileInfo(newPathForMove);
                if (newPathFileInfo.Exists)
                {
                    newPathFileInfo.Delete();
                    Console.WriteLine($"[FileInfo] Đã xóa tệp di chuyển cũ: {newPathFileInfo.FullName}");
                }
                fileInfo.MoveTo(newPathForMove); // Di chuyển bằng phương thức cá thể
                Console.WriteLine($"[FileInfo] Di chuyển tệp từ '{fileInfo.FullName}' sang '{newPathForMove}'");

                // QUAN TRỌNG: Đối tượng fileInfo ban đầu vẫn trỏ đến đường dẫn cũ.
                // Sau khi MoveTo, tệp không còn ở vị trí đó nữa. Cần tạo lại hoặc cập nhật đối tượng.
                fileInfo = new FileInfo(newPathForMove); 
                Console.WriteLine($"[FileInfo] Cập nhật đối tượng FileInfo tới đường dẫn mới: {fileInfo.FullName}");

                // 4. Mở tệp để đọc nội dung (phương pháp cơ bản với Stream)
                // FileInfo không có phương thức ReadAllText() trực tiếp như lớp File.
                // Thay vào đó, nó cung cấp các phương thức để làm việc với Stream (luồng dữ liệu).
                // Sử dụng khối lệnh "using" là CÁCH CHUẨN để đảm bảo Stream được đóng và tài nguyên được giải phóng đúng cách.
                string readContent = "";
                if (fileInfo.Exists)
                {
                    using (StreamReader sr = fileInfo.OpenText()) // Mở tệp dưới dạng StreamReader
                    {
                        readContent = sr.ReadToEnd();
                    }
                    Console.WriteLine($"[FileInfo] Nội dung của tệp '{fileInfo.FullName}' (đọc qua FileInfo.OpenText()):\n{readContent}");
                }
                
                // 5. Xóa tệp
                fileInfo.Delete(); // Xóa tệp đã di chuyển
                Console.WriteLine($"[FileInfo] Xóa tệp: {fileInfo.FullName}");

                // Xóa tệp sao chép nếu còn tồn tại
                // Lưu ý: destinationFileInfo vẫn trỏ đến đường dẫn cũ (destinationPath).
                // Các thao tác tiếp theo trên destinationFileInfo sẽ ảnh hưởng đến tệp tại destinationPath.
                if (destinationFileInfo.Exists) // Kiểm tra lại vì có thể đã bị xóa trong quá trình di chuyển
                {
                    destinationFileInfo.Delete();
                    Console.WriteLine($"[FileInfo] Xóa tệp sao chép: {destinationFileInfo.FullName}");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"[FileInfo] Lỗi khi thao tác tệp cá thể: {ex.Message}");
            }
        }
    }
}
```

> [!NOTE]
> **Quản lý tài nguyên với `using` statement:**
> Trong ví dụ trên, khi sử dụng `FileInfo.OpenText()` để đọc tệp, chúng ta sử dụng khối lệnh `using`. Đây là một cấu trúc ngôn ngữ rất quan trọng trong C# khi làm việc với các tài nguyên hệ thống **không được quản lý (unmanaged resources)** như tệp (file streams), kết nối mạng, hoặc các handle của hệ điều hành.
>
> *   **Cơ chế:** Các lớp quản lý tài nguyên này thường triển khai giao diện `IDisposable`. Khối `using` đảm bảo rằng phương thức `Dispose()` của đối tượng sẽ được gọi tự động ngay sau khi khối `using` kết thúc, hoặc khi có lỗi xảy ra.
> *   **Lợi ích:** Điều này ngăn chặn tình trạng rò rỉ tài nguyên (resource leak), nơi các tệp bị khóa hoặc bộ nhớ không được giải phóng, gây ra các vấn đề về hiệu suất và ổn định cho ứng dụng. Luôn sử dụng `using` khi làm việc với các đối tượng `Stream` (như `FileStream`, `StreamReader`, `StreamWriter`), `SqlConnection`, v.v.

## 3. Thao tác với Thư mục (Directory Operations)

Tương tự như tệp, chúng ta có thể làm việc với thư mục bằng cách sử dụng các lớp `Directory` (phương thức tĩnh) và `DirectoryInfo` (phương thức cá thể). Các nguyên tắc về hiệu suất và quản lý bộ nhớ cũng được áp dụng tương tự.

```csharp
using System;
using System.IO;
using System.Linq; // Cần thiết cho phương thức .ToList() nếu muốn chuyển đổi mảng sang List

namespace DirectoryOperations
{
    class Program
    {
        static void Main(string[] args)
        {
            string baseDirectory = @"C:\Temp\DirectoryDemo"; // Thư mục demo
            string subDirectory1 = Path.Combine(baseDirectory, "SubDir1");
            string subDirectory2 = Path.Combine(baseDirectory, "SubDir2");
            string filePath1 = Path.Combine(subDirectory1, "file1.txt");
            string filePath2 = Path.Combine(subDirectory2, "image.jpg");

            // Dọn dẹp trước khi chạy để đảm bảo trạng thái sạch
            if (Directory.Exists(baseDirectory))
            {
                Directory.Delete(baseDirectory, true); // true để xóa đệ quy tất cả nội dung
                Console.WriteLine($"Đã xóa thư mục gốc cũ: {baseDirectory}");
            }

            Console.WriteLine("\n--- Thao tác với Thư mục sử dụng lớp Directory (phương thức tĩnh) ---");
            PerformStaticDirectoryOperations(baseDirectory, subDirectory1, subDirectory2, filePath1, filePath2);

            // Dọn dẹp lại cho demo DirectoryInfo
            if (Directory.Exists(baseDirectory))
            {
                Directory.Delete(baseDirectory, true);
            }
            Console.WriteLine("\n--- Thao tác với Thư mục sử dụng lớp DirectoryInfo (phương thức cá thể) ---");
            PerformInstanceDirectoryOperations(baseDirectory, subDirectory1, subDirectory2, filePath1, filePath2);

            // Dọn dẹp cuối cùng
            if (Directory.Exists(baseDirectory))
            {
                Directory.Delete(baseDirectory, true);
                Console.WriteLine($"\nĐã dọn dẹp thư mục demo: {baseDirectory}");
            }
        }

        static void PerformStaticDirectoryOperations(string baseDir, string subDir1, string subDir2, string file1, string file2)
        {
            try
            {
                // 1. Tạo thư mục
                Directory.CreateDirectory(baseDir); // Tạo thư mục gốc
                Directory.CreateDirectory(subDir1); // Tạo thư mục con 1
                Directory.CreateDirectory(subDir2); // Tạo thư mục con 2
                Console.WriteLine($"[Directory] Đã tạo các thư mục: {baseDir}, {subDir1}, {subDir2}");

                // Tạo một số tệp bên trong các thư mục con để minh họa
                File.WriteAllText(file1, "Nội dung tệp 1");
                File.WriteAllText(file2, "Nội dung tệp 2");
                Console.WriteLine($"[Directory] Đã tạo các tệp: {file1}, {file2}");

                // 2. Kiểm tra sự tồn tại của thư mục
                if (Directory.Exists(baseDir))
                {
                    Console.WriteLine($"[Directory] Thư mục tồn tại: {baseDir}");
                }

                // 3. Lấy danh sách các tệp trong một thư mục
                Console.WriteLine($"\n[Directory] Các tệp trong '{subDir1}':");
                string[] filesInSubDir1 = Directory.GetFiles(subDir1); // Chỉ lấy tệp trong thư mục hiện tại
                foreach (string f in filesInSubDir1)
                {
                    Console.WriteLine($"- {Path.GetFileName(f)}");
                }

                // 4. Lấy danh sách các tệp với bộ lọc và tìm kiếm đệ quy
                Console.WriteLine($"\n[Directory] Các tệp *.txt trong '{baseDir}' (bao gồm thư mục con):");
                // SearchOption.AllDirectories: Tìm kiếm trong thư mục hiện tại và tất cả các thư mục con.
                // SearchOption.TopDirectoryOnly: Chỉ tìm kiếm trong thư mục hiện tại.
                string[] allTxtFiles = Directory.GetFiles(baseDir, "*.txt", SearchOption.AllDirectories);
                foreach (string f in allTxtFiles)
                {
                    Console.WriteLine($"- {Path.GetFileName(f)} (tại: {Path.GetDirectoryName(f)})");
                }
                
                // 5. Lấy danh sách các thư mục con
                Console.WriteLine($"\n[Directory] Các thư mục con trực tiếp trong '{baseDir}':");
                string[] subDirectories = Directory.GetDirectories(baseDir);
                foreach (string dir in subDirectories)
                {
                    Console.WriteLine($"- {Path.GetFileName(dir)}");
                }

                // 6. Xóa thư mục
                // Directory.Delete(path) chỉ xóa được thư mục rỗng.
                // Nếu muốn xóa thư mục không rỗng, phải dùng overload Directory.Delete(path, true).
                File.Delete(file2); // Xóa tệp trước khi xóa thư mục chứa nó
                Directory.Delete(subDir2); // Xóa thư mục rỗng
                Console.WriteLine($"[Directory] Đã xóa thư mục rỗng: {subDir2}");

                // Xóa thư mục gốc và tất cả nội dung của nó (bao gồm cả tệp và thư mục con)
                Directory.Delete(baseDir, true); // true để xóa đệ quy tất cả nội dung
                Console.WriteLine($"[Directory] Đã xóa thư mục gốc và tất cả nội dung: {baseDir}");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"[Directory] Lỗi khi thao tác thư mục tĩnh: {ex.Message}");
            }
        }

        static void PerformInstanceDirectoryOperations(string baseDir, string subDir1, string subDir2, string file1, string file2)
        {
            try
            {
                // 1. Tạo thư mục
                // Directory.CreateDirectory trả về một đối tượng DirectoryInfo, rất tiện lợi.
                DirectoryInfo baseDirInfo = Directory.CreateDirectory(baseDir); 
                // Có thể tạo thư mục con trực tiếp từ đối tượng DirectoryInfo.
                DirectoryInfo subDir1Info = baseDirInfo.CreateSubdirectory("SubDir1");
                DirectoryInfo subDir2Info = baseDirInfo.CreateSubdirectory("SubDir2");
                Console.WriteLine($"[DirectoryInfo] Đã tạo các thư mục: {baseDirInfo.FullName}, {subDir1Info.FullName}, {subDir2Info.FullName}");

                // Tạo một số tệp bên trong các thư mục con
                File.WriteAllText(file1, "Nội dung tệp 1");
                File.WriteAllText(file2, "Nội dung tệp 2");
                Console.WriteLine($"[DirectoryInfo] Đã tạo các tệp: {file1}, {file2}");

                // 2. Kiểm tra sự tồn tại của thư mục và truy cập các thuộc tính
                if (baseDirInfo.Exists)
                {
                    Console.WriteLine($"[DirectoryInfo] Thư mục tồn tại: {baseDirInfo.FullName}");
                    Console.WriteLine($"[DirectoryInfo] Thời gian tạo: {baseDirInfo.CreationTime}");
                    Console.WriteLine($"[DirectoryInfo] Thư mục cha: {baseDirInfo.Parent?.FullName ?? "Không có"}");
                }

                // 3. Lấy danh sách các tệp trong một thư mục
                Console.WriteLine($"\n[DirectoryInfo] Các tệp trong '{subDir1Info.FullName}':");
                FileInfo[] filesInSubDir1 = subDir1Info.GetFiles(); // Trả về mảng FileInfo
                foreach (FileInfo f in filesInSubDir1)
                {
                    Console.WriteLine($"- {f.Name} (Kích thước: {f.Length} bytes)");
                }

                // 4. Lấy danh sách các tệp với bộ lọc và tìm kiếm đệ quy
                Console.WriteLine($"\n[DirectoryInfo] Các tệp *.txt trong '{baseDirInfo.FullName}' (bao gồm thư mục con):");
                FileInfo[] allTxtFiles = baseDirInfo.GetFiles("*.txt", SearchOption.AllDirectories);
                foreach (FileInfo f in allTxtFiles)
                {
                    Console.WriteLine($"- {f.Name} (tại: {f.DirectoryName})");
                }

                // 5. Lấy danh sách các thư mục con
                Console.WriteLine($"\n[DirectoryInfo] Các thư mục con trực tiếp trong '{baseDirInfo.FullName}':");
                DirectoryInfo[] subDirectories = baseDirInfo.GetDirectories(); // Trả về mảng DirectoryInfo
                foreach (DirectoryInfo dir in subDirectories)
                {
                    Console.WriteLine($"- {dir.Name} (Thời gian tạo: {dir.CreationTime})");
                }

                // 6. Xóa thư mục
                // FileInfo cũng có phương thức Delete
                FileInfo file2Info = new FileInfo(file2);
                if (file2Info.Exists) file2Info.Delete(); // Xóa tệp trước khi xóa thư mục chứa nó
                
                subDir2Info.Delete(); // Xóa thư mục rỗng
                Console.WriteLine($"[DirectoryInfo] Đã xóa thư mục rỗng: {subDir2Info.FullName}");

                // Xóa thư mục gốc và tất cả nội dung của nó
                baseDirInfo.Delete(true); // true để xóa đệ quy tất cả nội dung
                Console.WriteLine($"[DirectoryInfo] Đã xóa thư mục gốc và tất cả nội dung: {baseDirInfo.FullName}");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"[DirectoryInfo] Lỗi khi thao tác thư mục cá thể: {ex.Message}");
            }
        }
    }
}
```

> [!TIP]
> **Enum `SearchOption`**: Khi tìm kiếm tệp hoặc thư mục, bạn có thể sử dụng `SearchOption` để chỉ định cách thức tìm kiếm.
> *   `SearchOption.TopDirectoryOnly`: Chỉ tìm kiếm trong thư mục hiện tại.
> *   `SearchOption.AllDirectories`: Tìm kiếm trong thư mục hiện tại và tất cả các thư mục con của nó (tìm kiếm đệ quy).

## 4. Làm việc với Đường dẫn (Path Operations)

Lớp `Path` là một lớp tĩnh rất hữu ích để phân tích và xây dựng các đường dẫn tệp hoặc thư mục một cách dễ dàng, đáng tin cậy và quan trọng nhất là **tương thích với các hệ điều hành khác nhau**. Điều này giúp mã của bạn hoạt động chính xác trên Windows, Linux, macOS mà không cần quan tâm đến sự khác biệt về ký tự phân tách đường dẫn (`\` hay `/`).

```csharp
using System;
using System.IO;

namespace PathOperations
{
    class Program
    {
        static void Main(string[] args)
        {
            string filePath = @"C:\Users\Admin\Documents\Projects\MyProject\Program.cs";
            string directoryPath = @"C:\Users\Admin\Downloads";
            string relativePath = @"..\..\Data\config.json";

            Console.WriteLine($"Đường dẫn gốc: {filePath}");

            // 1. Lấy phần mở rộng của tệp
            string extension = Path.GetExtension(filePath);
            Console.WriteLine($"Phần mở rộng: {extension}"); // Output: .cs

            // 2. Lấy tên tệp (bao gồm phần mở rộng)
            string fileName = Path.GetFileName(filePath);
            Console.WriteLine($"Tên tệp: {fileName}"); // Output: Program.cs

            // 3. Lấy tên tệp (không có phần mở rộng)
            string fileNameWithoutExtension = Path.GetFileNameWithoutExtension(filePath);
            Console.WriteLine($"Tên tệp không mở rộng: {fileNameWithoutExtension}"); // Output: Program

            // 4. Lấy tên thư mục chứa tệp
            string directoryName = Path.GetDirectoryName(filePath);
            Console.WriteLine($"Thư mục chứa tệp: {directoryName}"); // Output: C:\Users\Admin\Documents\Projects\MyProject

            // 5. Kết hợp các thành phần đường dẫn một cách an toàn
            // Path.Combine tự động xử lý các dấu gạch chéo ngược/thuận và thêm chúng nếu cần.
            string combinedPath = Path.Combine(directoryPath, "NewFolder", "report.pdf");
            Console.WriteLine($"Đường dẫn kết hợp: {combinedPath}"); // Output: C:\Users\Admin\Downloads\NewFolder\report.pdf

            // 6. Kiểm tra xem đường dẫn có phải là đường dẫn tuyệt đối (rooted) hay không
            bool isAbsolute = Path.IsPathRooted(filePath);
            Console.WriteLine($"Đường dẫn '{filePath}' là tuyệt đối: {isAbsolute}"); // Output: True
            Console.WriteLine($"Đường dẫn '{relativePath}' là tuyệt đối: {Path.IsPathRooted(relativePath)}"); // Output: False

            // 7. Lấy đường dẫn đến thư mục tạm thời của người dùng hiện tại
            string tempPath = Path.GetTempPath();
            Console.WriteLine($"Thư mục tạm thời của hệ thống: {tempPath}"); // Output: (ví dụ: C:\Users\Admin\AppData\Local\Temp\)

            // 8. Lấy tên tệp tạm thời duy nhất
            string tempFileName = Path.GetTempFileName(); // Phương thức này tạo một tệp rỗng trên đĩa
            Console.WriteLine($"Tên tệp tạm thời duy nhất: {tempFileName}"); // Output: (ví dụ: C:\Users\Admin\AppData\Local\Temp\tmpA893.tmp)
            // Cần xóa tệp này sau khi sử dụng để dọn dẹp hệ thống
            File.Delete(tempFileName);
            Console.WriteLine($"Đã xóa tệp tạm thời: {tempFileName}");

            // 9. Thay đổi phần mở rộng của tệp
            string newExtensionPath = Path.ChangeExtension(filePath, ".txt");
            Console.WriteLine($"Đường dẫn với phần mở rộng mới: {newExtensionPath}"); // Output: C:\Users\Admin\Documents\Projects\MyProject\Program.txt

            // 10. Lấy ký tự phân tách đường dẫn của hệ điều hành hiện tại
            Console.WriteLine($"Ký tự phân tách đường dẫn của hệ điều hành: '{Path.DirectorySeparatorChar}'"); // Output: '\' trên Windows, '/' trên Linux
        }
    }
}
```

> [!TIP]
> Luôn sử dụng `Path.Combine()` để kết hợp các phần của một đường dẫn thay vì nối chuỗi thủ công. Điều này giúp tránh các vấn đề về dấu gạch chéo ngược/thuận trên các hệ điều hành khác nhau và đảm bảo đường dẫn được định dạng chính xác.

## 5. Thao tác Tệp và Thư mục trong Ngữ cảnh Antigravity IDE và Vibe Coding

Hiểu rõ cách C# tương tác với hệ thống tệp không chỉ là kỹ năng cơ bản để xây dựng ứng dụng mà còn là chìa khóa để nắm bắt cách các hệ thống AI lập trình tiên tiến như Antigravity IDE hoạt động và tương tác với môi trường của chúng.

Antigravity IDE, với khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file và lập kế hoạch tự động, thực chất đang sử dụng các cơ chế thao tác tệp và thư mục tương tự (dù có thể được triển khai bằng các ngôn ngữ khác như Python hoặc thông qua các API cấp cao hơn). Đối với Antigravity, hệ thống tệp chính là "thế giới vật lý" mà nó có thể "chạm" và "thao tác" trong quá trình lập trình.

Hãy cùng xem xét cách tư duy **Vibe Coding** – tập trung vào ý định và ngữ cảnh – có thể áp dụng khi bạn tương tác với Antigravity thông qua lăng kính của các thao tác tệp:

1.  **Antigravity "Đọc" Dự án của Bạn (File.ReadAllText, Directory.GetFiles, FileInfo.OpenText):**
    *   Khi bạn yêu cầu Antigravity "phân tích cấu trúc dự án," nó không hề "ma thuật." Thay vào đó, nó sẽ sử dụng các phương thức tương đương `Directory.GetDirectories` và `Directory.GetFiles` (với `SearchOption.AllDirectories`) để liệt kê tất cả các thư mục và tệp.
    *   Để "hiểu" nội dung của một tệp `.cs` hoặc `.json`, Antigravity sẽ thực hiện một thao tác tương đương `File.ReadAllText()` hoặc `FileInfo.OpenText()` để đọc nội dung tệp vào bộ nhớ của nó.
    *   **Vibe Coding Insight:** Khi bạn "vibes" Antigravity để hiểu một phần code, bạn đang ngầm chỉ dẫn nó sử dụng các thao tác đọc tệp. Nếu Antigravity gặp lỗi, bạn có thể "vibes" rằng có thể có vấn đề với quyền truy cập tệp (`UnauthorizedAccessException`) hoặc tệp không tồn tại (`FileNotFoundException`).

2.  **Antigravity "Tạo" và "Sửa đổi" Code (File.WriteAllText, FileInfo.Create, FileInfo.AppendText):**
    *   Khi Antigravity "tạo một tệp class mới" hoặc "cập nhật một phương thức hiện có," nó đang thực hiện các thao tác ghi tệp.
    *   "Tạo một tệp mới" đồng nghĩa với việc nó sẽ sử dụng `File.WriteAllText()` (hoặc tương đương) để tạo tệp và ghi nội dung ban đầu.
    *   "Sửa đổi một tệp" có thể liên quan đến việc đọc toàn bộ nội dung, chỉnh sửa trong bộ nhớ, sau đó ghi lại toàn bộ nội dung mới. Đối với các tệp log, nó có thể sử dụng `File.AppendAllText()` để thêm nội dung vào cuối tệp.
    *   **Vibe Coding Insight:** Khi bạn yêu cầu Antigravity "thêm một tính năng," bạn đang "vibes" nó để tạo hoặc sửa đổi các tệp. Nếu Antigravity không thể lưu thay đổi, bạn có thể suy luận rằng có thể có vấn đề về quyền ghi hoặc tệp đang bị khóa bởi một tiến trình khác (`IOException`).

3.  **Antigravity "Tổ chức lại" Dự án (Directory.CreateDirectory, File.Move, FileInfo.MoveTo, Path.Combine):**
    *   Nếu bạn yêu cầu Antigravity "di chuyển tất cả các tệp ViewModel vào thư mục `ViewModels`," nó sẽ sử dụng `Directory.CreateDirectory()` để tạo thư mục nếu chưa có, và sau đó dùng `File.Move()` hoặc `FileInfo.MoveTo()` cho từng tệp.
    *   Việc xác định đường dẫn đích cho các tệp này chắc chắn sẽ liên quan đến `Path.Combine()` để xây dựng đường dẫn mới một cách an toàn.
    *   **Vibe Coding Insight:** "Vibes" về cấu trúc và tổ chức dự án sẽ kích hoạt các thao tác di chuyển và tạo thư mục. Nếu Antigravity "làm mất" tệp hoặc không thể tạo thư mục, bạn có thể "vibes" rằng có lỗi trong việc xây dựng đường dẫn (`Path` class) hoặc quyền tạo thư mục.

**Áp dụng Tư duy Vibe Coding vào Antigravity:**

*   **Hiểu Rõ Ý Định:** Khi bạn đưa ra một yêu cầu cho Antigravity, hãy nghĩ về những thao tác tệp cơ bản mà nó sẽ cần thực hiện. Ví dụ: "Tạo một dịch vụ mới" = > `Directory.CreateDirectory` (nếu cần), `File.WriteAllText` (tạo file service), `File.WriteAllText` (tạo interface service).
*   **Dự Đoán Hành Vi:** Khi Antigravity báo cáo một hành động, hãy hình dung các phương thức `System.IO` nào có thể đã được sử dụng. Điều này giúp bạn hiểu sâu hơn về logic của AI.
*   **Gỡ Lỗi Hiệu Quả Hơn:** Nếu Antigravity gặp lỗi trong quá trình thực hiện một nhiệm vụ, kiến thức về các ngoại lệ `System.IO` (như `UnauthorizedAccessException`, `IOException`, `FileNotFoundException`) sẽ giúp bạn nhanh chóng xác định nguyên nhân gốc rễ: liệu có phải là vấn đề về quyền, tệp bị khóa, hay đường dẫn không hợp lệ.
*   **Hướng Dẫn Chính Xác:** Thay vì chỉ nói "sửa lỗi này," bạn có thể nói "hãy kiểm tra xem tệp cấu hình `appsettings.json` có tồn tại ở đường dẫn X không và đọc nội dung của nó." Điều này cung cấp cho Antigravity các bước cụ thể hơn, tương ứng với các thao tác `System.IO` mà nó có thể thực hiện.

Tóm lại, File I/O là ngôn ngữ cơ bản mà mọi ứng dụng, bao gồm cả các agent AI, sử dụng để tương tác với thế giới dữ liệu bên ngoài. Nắm vững nó không chỉ giúp bạn viết code tốt hơn mà còn giúp bạn trở thành một "người hướng dẫn" hiệu quả hơn cho các công cụ AI lập trình trong tương lai.

## Tóm tắt Phần

*   **Không gian tên `System.IO`** là trung tâm cho mọi hoạt động I/O trong .NET, chứa các lớp để thao tác với tệp và thư mục.
*   **Lớp `File` và `FileInfo`** được sử dụng để làm việc với tệp.
    *   `File` cung cấp các **phương thức tĩnh** tiện lợi cho các thao tác đơn lẻ, nhưng có thể kém hiệu quả hơn do lặp lại kiểm tra bảo mật và phân tích đường dẫn.
    *   `FileInfo` cung cấp các **phương thức cá thể**, hiệu quả hơn cho nhiều thao tác liên tiếp trên cùng một tệp vì nó đại diện cho tệp dưới dạng một đối tượng trên heap và có thể lưu trữ thông tin đã được kiểm tra ban đầu.
*   **Lớp `Directory` và `DirectoryInfo`** được sử dụng để làm việc với thư mục, với sự khác biệt về phương thức tĩnh và cá thể tương tự như `File` và `FileInfo`.
*   **Sự khác biệt giữa phương thức tĩnh và cá thể** nằm ở kiến trúc (lớp tĩnh vs. đối tượng trên heap), hiệu suất (kiểm tra lặp lại vs. kiểm tra một lần) và cách quản lý trạng thái (không trạng thái vs. có trạng thái và thuộc tính).
*   **Lớp `Path`** là một lớp tĩnh mạnh mẽ giúp phân tích, xây dựng và làm việc với các chuỗi đường dẫn tệp hoặc thư mục một cách dễ dàng, đáng tin cậy và tương thích đa nền tảng, tránh việc xử lý chuỗi thủ công. Luôn sử dụng `Path.Combine()` để kết hợp đường dẫn.
*   **Xử lý lỗi (Exception Handling)** với `try-catch` là cực kỳ quan trọng khi làm việc với hệ thống tệp, vì các lỗi như `FileNotFoundException`, `UnauthorizedAccessException`, `IOException` là phổ biến và cần được quản lý để ứng dụng ổn định.
*   **Sử dụng `using` statement** là cách tốt nhất để đảm bảo các tài nguyên hệ thống không được quản lý như luồng tệp (file streams) được giải phóng (gọi `Dispose()`) đúng cách và tự động sau khi sử dụng, ngăn chặn rò rỉ tài nguyên.
*   Hiểu rõ các thao tác tệp giúp bạn không chỉ viết ứng dụng tốt hơn mà còn cung cấp cái nhìn sâu sắc về cách các hệ thống AI lập trình như Antigravity IDE tương tác với môi trường của chúng, cho phép bạn áp dụng tư duy **Vibe Coding** để định hướng và gỡ lỗi AI một cách hiệu quả hơn.

<!-- REVIEWED_BY_AGENT -->
