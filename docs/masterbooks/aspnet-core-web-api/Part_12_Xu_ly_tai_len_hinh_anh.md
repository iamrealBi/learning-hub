# Bài 12: Xử lý Tải lên Hình ảnh với RESTful API trong ASP.NET Core

Trong kỷ nguyên phát triển web hiện đại, khả năng quản lý và tải lên hình ảnh là một chức năng không thể thiếu đối với phần lớn các ứng dụng. Từ việc cung cấp ảnh hồ sơ người dùng, hình ảnh sản phẩm cho các nền tảng thương mại điện tử, đến quản lý nội dung đa phương tiện, việc triển khai một cơ chế xử lý tải lên hình ảnh hiệu quả, an toàn và có cấu trúc rõ ràng là tối quan trọng. Chương này sẽ hướng dẫn bạn xây dựng một RESTful Web API mạnh mẽ bằng ASP.NET Core kết hợp với Entity Framework Core để quản lý quy trình tải lên hình ảnh. Chúng ta sẽ tập trung vào các nguyên tắc kiến trúc cốt lõi như Dependency Injection, Repository Pattern, thiết kế Controller và việc sử dụng đúng các HTTP Verbs.

Mục tiêu học tập của chương này là trang bị cho bạn kiến thức và kỹ năng thực tế để:

*   **Thiết kế Mô hình Miền (Domain Model)** để biểu diễn và lưu trữ metadata của hình ảnh.
*   **Cập nhật DbContext** và sử dụng **Entity Framework Core Migrations** để tạo cấu trúc bảng cơ sở dữ liệu tương ứng.
*   **Xây dựng API Controller** và **Phương thức Hành động (Action Method)** để xử lý các yêu cầu tải lên hình ảnh từ client.
*   **Triển khai xác thực phía máy chủ** toàn diện cho các tệp tải lên (kiểm tra loại tệp, kích thước, v.v.).
*   **Áp dụng Repository Pattern** để tách biệt logic lưu trữ tệp vật lý và quản lý dữ liệu cơ sở dữ liệu.
*   **Lưu trữ hình ảnh vật lý** vào hệ thống tệp cục bộ.
*   **Lưu thông tin metadata** của hình ảnh vào cơ sở dữ liệu.
*   **Cấu hình ASP.NET Core Web API** để phục vụ các tệp tĩnh, cho phép truy cập hình ảnh đã tải lên thông qua URL công khai.

Chúng ta sẽ tiếp cận từng bước một, đảm bảo sự hiểu biết sâu sắc về mọi khía cạnh kỹ thuật và kiến trúc liên quan.

## 1. Thiết kế Mô hình Miền cho Hình ảnh

Bước nền tảng trong việc xử lý tải lên hình ảnh là định nghĩa cách chúng ta sẽ biểu diễn thông tin về hình ảnh trong logic nghiệp vụ và cơ sở dữ liệu của ứng dụng. Chúng ta sẽ tạo một mô hình miền `Image` để lưu trữ các thuộc tính quan trọng liên quan đến hình ảnh, chứ không phải bản thân dữ liệu nhị phân của hình ảnh.

> [!NOTE]
> **Mô hình Miền (Domain Model)** là một biểu diễn trừu tượng của các khái niệm trong thế giới thực mà ứng dụng của bạn đang xử lý. Nó không chỉ chứa dữ liệu mà còn cả logic nghiệp vụ liên quan đến các khái niệm đó. Trong bối cảnh của Entity Framework Core, các mô hình miền thường được ánh xạ trực tiếp tới các bảng trong cơ sở dữ liệu, đóng vai trò là xương sống cho Persistent Layer.

Tạo một lớp mới có tên `Image.cs` trong thư mục `Models/Domain` của dự án.

```csharp
// Models/Domain/Image.cs
using System.ComponentModel.DataAnnotations.Schema; // Cần thiết cho [NotMapped]
using Microsoft.AspNetCore.Http; // Cần thiết cho IFormFile

namespace NZWalks.API.Models.Domain
{
    /// <summary>
    /// Đại diện cho một thực thể hình ảnh trong miền ứng dụng, lưu trữ metadata và đường dẫn vật lý.
    /// </summary>
    public class Image
    {
        /// <summary>
        /// Định danh duy nhất cho hình ảnh (Khóa chính).
        /// </summary>
        public Guid Id { get; set; }

        /// <summary>
        /// Tệp hình ảnh thực tế được tải lên từ client.
        /// Thuộc tính này KHÔNG được ánh xạ vào cơ sở dữ liệu; nó chỉ được sử dụng trong quá trình xử lý tải lên.
        /// </summary>
        [NotMapped]
        public IFormFile File { get; set; }

        /// <summary>
        /// Tên của tệp hình ảnh (ví dụ: "my_profile_pic").
        /// </summary>
        public string FileName { get; set; }

        /// <summary>
        /// Mô tả tùy chọn cho hình ảnh.
        /// </summary>
        public string? FileDescription { get; set; }

        /// <summary>
        /// Phần mở rộng của tệp (ví dụ: ".jpg", ".png").
        /// </summary>
        public string FileExtension { get; set; }

        /// <summary>
        /// Kích thước của tệp tính bằng byte.
        /// </summary>
        public long FileSizeInBytes { get; set; }

        /// <summary>
        /// Đường dẫn URL công khai để truy cập hình ảnh sau khi tải lên.
        /// </summary>
        public string FilePath { get; set; }
    }
}
```

**Giải thích các Thuộc tính quan trọng:**

*   **`Id` (Guid)**: Khóa chính, định danh duy nhất cho mỗi bản ghi hình ảnh. Sử dụng `Guid` giúp tránh xung đột ID trong môi trường phân tán và không yêu cầu cơ sở dữ liệu phải tự động tăng.
*   **`IFormFile File` và `[NotMapped]`**:
    *   **`IFormFile`**: Đây là một interface cốt lõi trong ASP.NET Core, được thiết kế để biểu diễn một tệp được gửi từ một biểu mẫu HTML (thường là qua `multipart/form-data`). Nó cung cấp các thuộc tính tiện lợi như `FileName` (tên gốc của tệp), `Length` (kích thước tệp tính bằng byte), và các phương thức để đọc nội dung tệp (ví dụ: `CopyToAsync` để sao chép vào một `Stream`). Trong quá trình tải lên, ASP.NET Core runtime sẽ tự động ánh xạ dữ liệu tệp từ yêu cầu HTTP vào thuộc tính `IFormFile` này.
    *   **`[NotMapped]` Attribute**: Thuộc tính này thuộc namespace `System.ComponentModel.DataAnnotations.Schema`. Khi áp dụng cho một thuộc tính trong mô hình miền của Entity Framework Core, nó chỉ thị rõ ràng rằng thuộc tính này không nên được ánh xạ tới một cột trong cơ sở dữ liệu.
        *   **Lý do**: Chúng ta không lưu trữ toàn bộ dữ liệu nhị phân của tệp hình ảnh trực tiếp trong cơ sở dữ liệu (thường không phải là phương pháp tốt cho hiệu suất và quản lý dữ liệu lớn - BLOBs). Thay vào đó, chúng ta sẽ lưu tệp vật lý vào hệ thống tệp của máy chủ và chỉ lưu trữ đường dẫn (`FilePath`) đến tệp đó trong cơ sở dữ liệu. Thuộc tính `File` chỉ tồn tại trong bộ nhớ ứng dụng trong suốt quá trình xử lý yêu cầu tải lên.
*   **`FileName`, `FileDescription`, `FileExtension`, `FileSizeInBytes`**: Các thuộc tính này lưu trữ metadata quan trọng về hình ảnh, hữu ích cho việc hiển thị, tìm kiếm và quản lý.
*   **`FilePath`**: Đây là thuộc tính then chốt. Sau khi tệp hình ảnh được lưu vật lý, thuộc tính này sẽ chứa URL công khai mà qua đó client có thể truy cập hình ảnh.

> [!TIP]
> **Vibe Coding với Antigravity IDE**: Khi thiết kế mô hình miền, bạn có thể áp dụng tư duy "Vibe Coding" với Antigravity IDE. Thay vì gõ từng dòng mã, bạn có thể mô tả ý định của mình bằng ngôn ngữ tự nhiên: "Tạo một mô hình `Image` để lưu trữ thông tin về hình ảnh. Nó cần một ID, tên tệp, mô tả (tùy chọn), phần mở rộng, kích thước và đường dẫn URL. Quan trọng là, nó cũng cần một thuộc tính để nhận tệp được tải lên nhưng không lưu vào cơ sở dữ liệu."
>
> Antigravity IDE, với khả năng hiểu ngữ cảnh lập trình và mục tiêu cuối cùng, có thể tự động tạo cấu trúc lớp `Image` này, bao gồm việc thêm `using` statements, thuộc tính `[NotMapped]` cho `IFormFile`, và lựa chọn kiểu dữ liệu phù hợp. Nó giúp bạn tập trung vào *những gì* cần xây dựng hơn là *cách* xây dựng chi tiết, tăng tốc độ phát triển và giảm thiểu lỗi cú pháp.

## 2. Cập nhật DbContext và Thực hiện Migrations

Sau khi định nghĩa mô hình miền `Image`, bước tiếp theo là thông báo cho Entity Framework Core (EF Core) biết về mô hình này để nó có thể tạo một bảng tương ứng trong cơ sở dữ liệu. Điều này được thực hiện bằng cách thêm một thuộc tính `DbSet<Image>` vào lớp `ApplicationDbContext` của chúng ta.

> [!NOTE]
> **`DbContext`** là một lớp trung tâm trong EF Core, đóng vai trò là cầu nối giữa các mô hình miền của bạn và cơ sở dữ liệu. Nó quản lý các phiên làm việc với cơ sở dữ liệu, bao gồm việc theo dõi các thay đổi, truy vấn, và lưu dữ liệu.
>
> **`DbSet<TEntity>`** đại diện cho một tập hợp các thực thể của một kiểu cụ thể trong `DbContext`. Mỗi thuộc tính `DbSet` trong `DbContext` thường ánh xạ tới một bảng trong cơ sở dữ liệu, cho phép bạn thực hiện các thao tác CRUD (Create, Read, Update, Delete) trên các thực thể đó.

Mở lớp `ApplicationDbContext.cs` (thường nằm trong thư mục `Data`) và thêm `DbSet<Image>`:

```csharp
// Data/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Models.Domain; // Đảm bảo include namespace của mô hình Image

namespace NZWalks.API.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> dbContextOptions) : base(dbContextOptions)
        {
        }

        // ... các DbSet khác (ví dụ: public DbSet<Walk> Walks { get; set; }) ...

        /// <summary>
        /// Đại diện cho tập hợp các hình ảnh trong cơ sở dữ liệu.
        /// </summary>
        public DbSet<Image> Images { get; set; }
    }
}
```

### 2.1. Thực hiện Entity Framework Core Migrations

Sau khi cập nhật `DbContext`, chúng ta cần tạo một migration mới và cập nhật cơ sở dữ liệu để EF Core tạo bảng `Images` dựa trên mô hình `Image` của chúng ta.

> [!TIP]
> **Entity Framework Core Migrations** là một tính năng mạnh mẽ cho phép bạn quản lý và tiến hóa lược đồ cơ sở dữ liệu của mình theo thời gian khi các mô hình miền trong ứng dụng thay đổi. Thay vì phải viết SQL thủ công, bạn có thể tạo các tập lệnh migration để tự động áp dụng các thay đổi lược đồ một cách có kiểm soát và theo dõi lịch sử thay đổi.

Mở **Package Manager Console** (trong Visual Studio: `Tools` > `NuGet Package Manager` > `Package Manager Console`) hoặc sử dụng cửa sổ Terminal trong VS Code và chạy các lệnh sau:

1.  **Thêm Migration mới:**
    ```powershell
    Add-Migration "Adding Image Table" -Context ApplicationDbContext
    ```
    *   Lệnh này sẽ quét các mô hình trong `ApplicationDbContext` của bạn, so sánh với trạng thái lược đồ cơ sở dữ liệu gần nhất (được ghi trong thư mục `Migrations` và tệp `[DbContext]ModelSnapshot.cs`).
    *   Nó sẽ tạo một tập tin migration mới trong thư mục `Migrations` (ví dụ: `20231027100000_AddingImageTable.cs`), chứa mã C# để tạo bảng `Images` và các cột tương ứng.
    *   Chúng ta chỉ định `-Context ApplicationDbContext` để đảm bảo lệnh áp dụng cho đúng `DbContext` nếu ứng dụng có nhiều hơn một.

2.  **Cập nhật cơ sở dữ liệu:**
    ```powershell
    Update-Database -Context ApplicationDbContext
    ```
    *   Lệnh này sẽ đọc các migration chưa được áp dụng vào cơ sở dữ liệu của bạn và thực thi chúng.
    *   Trong trường hợp này, nó sẽ tạo bảng `Images` với các cột tương ứng với các thuộc tính của mô hình `Image` (trừ thuộc tính `[NotMapped] File`) trong cơ sở dữ liệu.

Sau khi các lệnh này chạy thành công, bạn có thể kiểm tra cơ sở dữ liệu của mình (sử dụng SQL Server Management Studio, Azure Data Studio, hoặc SQL Server Object Explorer trong Visual Studio) để xác minh rằng bảng `Images` đã được tạo với cấu trúc cột chính xác.

> [!TIP]
> **Vibe Coding với Antigravity IDE (Migrations)**: Antigravity IDE có thể đơn giản hóa quy trình migrations. Sau khi bạn chỉnh sửa mô hình miền và lưu tệp, Antigravity có thể tự động phát hiện các thay đổi lược đồ. Thay vì phải nhớ và gõ các lệnh `Add-Migration` và `Update-Database`, bạn có thể nhận được một gợi ý trực tiếp trong IDE: "Changes detected in `Image` model. Do you want to generate a new migration and apply it to the database?" Một cú nhấp chuột hoặc một lệnh ngôn ngữ tự nhiên "Apply database changes" có thể thực hiện toàn bộ quy trình, bao gồm việc chạy các lệnh ngầm và hiển thị kết quả.

## 3. Xây dựng API Controller và Action Method cho Tải lên Hình ảnh

Với mô hình miền và bảng cơ sở dữ liệu đã sẵn sàng, bước tiếp theo là tạo một API Controller để xử lý các yêu cầu HTTP từ client liên quan đến việc tải lên hình ảnh. Controller này sẽ là điểm vào cho các tương tác của client.

### 3.1. Tạo Controller mới

Tạo một Controller mới có tên `ImagesController.cs` trong thư mục `Controllers`:

```csharp
// Controllers/ImagesController.cs
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO; // Sẽ tạo DTO này sau
using NZWalks.API.Repositories; // Sẽ tạo Repository này sau

namespace NZWalks.API.Controllers
{
    /// <summary>
    /// Controller để xử lý các yêu cầu liên quan đến hình ảnh.
    /// </summary>
    [Route("api/[controller]")] // Định nghĩa tuyến đường cơ sở cho Controller
    [ApiController] // Kích hoạt các tính năng tiện lợi cho API Controller
    public class ImagesController : ControllerBase
    {
        private readonly IImageRepository _imageRepository; // Dependency Injection cho Image Repository

        /// <summary>
        /// Constructor để inject IImageRepository.
        /// </summary>
        /// <param name="imageRepository">Thực thể của IImageRepository.</param>
        public ImagesController(IImageRepository imageRepository)
        {
            _imageRepository = imageRepository;
        }

        // Phương thức hành động Upload sẽ được thêm vào đây trong phần tiếp theo.
    }
}
```

**Giải thích các Thuộc tính quan trọng:**

*   **`[Route("api/[controller]")]`**: Đây là một thuộc tính định tuyến (routing attribute) định nghĩa tuyến đường cơ sở cho Controller này. `[controller]` là một placeholder sẽ được thay thế bằng tên của Controller (trừ hậu tố `Controller`), tức là `Images`. Do đó, tuyến đường cơ sở sẽ là `/api/Images`.
*   **`[ApiController]`**: Thuộc tính này là một công cụ mạnh mẽ trong ASP.NET Core MVC, giúp kích hoạt một loạt các tính năng tiện lợi và quy ước cho các API Controller, bao gồm:
    *   **Xác thực mô hình tự động (Automatic Model Validation)**: Khi `ModelState.IsValid` là `false` do lỗi xác thực từ các thuộc tính `[Required]`, `[StringLength]`, v.v., `[ApiController]` sẽ tự động trả về phản hồi `400 Bad Request` với chi tiết lỗi, mà không cần bạn phải kiểm tra `if (!ModelState.IsValid)` thủ công trong mọi action method.
    *   **Suy luận nguồn tham số (Parameter Source Inference)**: Giúp tự động xác định nơi lấy dữ liệu cho các tham số của action method (ví dụ: từ body, query string, route).
    *   **Xử lý lỗi nâng cao**: Tự động trả về các kiểu `ProblemDetails` theo chuẩn RFC 7807 cho các lỗi HTTP cụ thể.
*   **`ControllerBase`**: Đây là lớp cơ sở cho các API Controller không có giao diện người dùng (views). Nó cung cấp quyền truy cập vào các tính năng cốt lõi của MVC như `ModelState`, `HttpContext`, và các phương thức `IActionResult` (ví dụ: `Ok()`, `BadRequest()`).
*   **Dependency Injection (`IImageRepository _imageRepository`)**: Chúng ta khai báo một trường `_imageRepository` kiểu `IImageRepository` và khởi tạo nó thông qua constructor. Đây là một ví dụ điển hình của Dependency Injection, nơi Controller không tự tạo `IImageRepository` mà nhận nó từ bên ngoài (thông qua DI container), giúp tăng tính mô-đun và khả năng kiểm thử.

### 3.2. Định nghĩa Data Transfer Object (DTO) cho Yêu cầu Tải lên

Để duy trì nguyên tắc tách biệt các mối quan tâm (separation of concerns) và kiểm soát chặt chẽ dữ liệu được trao đổi với client, chúng ta sẽ tạo một DTO (`ImageUploadRequestDto`) riêng biệt cho yêu cầu tải lên hình ảnh. Điều này giúp chúng ta định hình chính xác dữ liệu mà client có thể gửi, tránh các vấn đề bảo mật như over-posting (khi client gửi nhiều dữ liệu hơn dự kiến và có thể ghi đè các thuộc tính nhạy cảm).

> [!NOTE]
> **Data Transfer Object (DTO)** là một đối tượng dùng để truyền dữ liệu giữa các lớp hoặc giữa các quá trình. Trong ngữ cảnh của API, DTO thường được sử dụng để định hình dữ liệu mà client gửi đến (Request DTO) hoặc nhận từ API (Response DTO), khác với mô hình miền nội bộ của ứng dụng. DTO giúp:
> 1.  **Tách biệt**: Giữ mô hình miền không bị ảnh hưởng bởi các yêu cầu hoặc phản hồi cụ thể của API.
> 2.  **Bảo mật**: Chỉ lộ ra những thuộc tính cần thiết, ngăn chặn over-posting.
> 3.  **Linh hoạt**: Cho phép thay đổi API mà không ảnh hưởng đến mô hình miền.
> 4.  **Xác thực**: Cung cấp một bề mặt riêng để áp dụng các quy tắc xác thực cho dữ liệu đầu vào.

Tạo một lớp mới có tên `ImageUploadRequestDto.cs` trong thư mục `Models/DTO`:

```csharp
// Models/DTO/ImageUploadRequestDto.cs
using System.ComponentModel.DataAnnotations; // Cần thiết cho [Required]
using Microsoft.AspNetCore.Http; // Cần thiết cho IFormFile

namespace NZWalks.API.Models.DTO
{
    /// <summary>
    /// DTO đại diện cho yêu cầu tải lên hình ảnh từ client.
    /// </summary>
    public class ImageUploadRequestDto
    {
        /// <summary>
        /// Tệp hình ảnh được gửi từ client. Bắt buộc.
        /// </summary>
        [Required(ErrorMessage = "Please select an image file.")]
        public IFormFile File { get; set; }

        /// <summary>
        /// Tên mong muốn cho tệp hình ảnh. Bắt buộc.
        /// </summary>
        [Required(ErrorMessage = "File name is required.")]
        public string FileName { get; set; }

        /// <summary>
        /// Mô tả tùy chọn cho hình ảnh.
        /// </summary>
        public string? FileDescription { get; set; }
    }
}
```

**Giải thích các Thuộc tính:**

*   **`File` (IFormFile)**: Đây là nơi tệp hình ảnh thực tế sẽ được nhận từ yêu cầu HTTP. Thuộc tính `[Required]` đảm bảo rằng client phải cung cấp một tệp.
*   **`FileName` (string)**: Tên mà người dùng muốn đặt cho tệp. Thuộc tính `[Required]` đảm bảo tên tệp được cung cấp.
*   **`FileDescription` (string?)**: Mô tả tùy chọn cho hình ảnh. Dấu `?` chỉ ra rằng thuộc tính này có thể là null.

### 3.3. Triển khai Phương thức Hành động `Upload`

Bây giờ, hãy thêm phương thức `Upload` vào `ImagesController`. Phương thức này sẽ xử lý các yêu cầu HTTP POST để tải lên hình ảnh, bao gồm xác thực, chuyển đổi dữ liệu và ủy quyền cho Repository.

```csharp
// Controllers/ImagesController.cs (tiếp tục)
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;
using NZWalks.API.Models.Domain; // Cần thiết cho mô hình Image
using System.IO; // Cần thiết cho Path

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ImagesController : ControllerBase
    {
        private readonly IImageRepository _imageRepository;
        // Định nghĩa các phần mở rộng tệp được phép và kích thước tối đa
        private readonly string[] _allowedExtensions = { ".jpg", ".jpeg", ".png", ".gif" };
        private readonly long _maxFileSize = 10 * 1024 * 1024; // 10 MB

        public ImagesController(IImageRepository imageRepository)
        {
            _imageRepository = imageRepository;
        }

        /// <summary>
        /// Xử lý yêu cầu tải lên hình ảnh mới.
        /// </summary>
        /// <param name="request">DTO chứa tệp hình ảnh và metadata.</param>
        /// <returns>Đối tượng Image đã được lưu hoặc lỗi xác thực.</returns>
        // POST: /api/Images/Upload
        [HttpPost]
        [Route("Upload")]
        public async Task<IActionResult> Upload([FromForm] ImageUploadRequestDto request)
        {
            // 1. Thực hiện xác thực phía máy chủ tùy chỉnh
            ValidateFileUpload(request);

            // Kiểm tra ModelState sau khi xác thực tùy chỉnh và xác thực DTO ([Required])
            if (ModelState.IsValid)
            {
                // 2. Chuyển đổi DTO sang Mô hình Miền
                var imageDomainModel = new Image
                {
                    File = request.File,
                    FileName = request.FileName,
                    FileDescription = request.FileDescription,
                    FileExtension = Path.GetExtension(request.File.FileName).ToLowerInvariant(), // Lấy phần mở rộng chuẩn hóa
                    FileSizeInBytes = request.File.Length
                    // FilePath sẽ được điền bởi Repository sau khi lưu vật lý
                };

                // 3. Sử dụng Repository để tải lên hình ảnh (lưu vào hệ thống tệp và DB)
                // Repository sẽ trả về mô hình miền đã được cập nhật với FilePath
                imageDomainModel = await _imageRepository.Upload(imageDomainModel);

                // 4. Trả về phản hồi thành công
                // Có thể chuyển đổi lại thành DTO phản hồi nếu cần, nhưng ở đây trả về Domain Model đơn giản
                return Ok(imageDomainModel);
            }

            // 5. Trả về lỗi nếu ModelState không hợp lệ (do [ApiController] hoặc ValidateFileUpload)
            return BadRequest(ModelState);
        }

        /// <summary>
        /// Phương thức riêng tư để thực hiện xác thực phía máy chủ cho tệp tải lên.
        /// </summary>
        /// <param name="request">DTO yêu cầu tải lên hình ảnh.</param>
        private void ValidateFileUpload(ImageUploadRequestDto request)
        {
            // Kiểm tra nếu tệp không được cung cấp (mặc dù [Required] trên DTO đã xử lý)
            if (request.File == null)
            {
                ModelState.AddModelError("file", "No file uploaded.");
                return;
            }

            // Xác thực phần mở rộng tệp
            var fileExtension = Path.GetExtension(request.File.FileName).ToLowerInvariant();
            if (!_allowedExtensions.Contains(fileExtension))
            {
                ModelState.AddModelError("file", $"Unsupported file extension. Only {string.Join(", ", _allowedExtensions)} are allowed.");
            }

            // Xác thực kích thước tệp
            if (request.File.Length > _maxFileSize)
            {
                ModelState.AddModelError("file", $"File size is too large, please upload a file less than {_maxFileSize / (1024 * 1024)}MB.");
            }
        }
    }
}
```

**Giải thích chi tiết các thành phần:**

*   **HTTP Verb và Routing (`[HttpPost]`, `[Route("Upload")]`)**:
    *   `[HttpPost]`: Chỉ định rằng phương thức `Upload` sẽ xử lý các yêu cầu HTTP POST. Đây là HTTP Verb phù hợp cho thao tác tạo tài nguyên mới trên máy chủ.
    *   `[Route("Upload")]`: Kết hợp với `[Route("api/[controller]")]` trên lớp Controller, tuyến đường đầy đủ để truy cập phương thức này sẽ là `/api/Images/Upload`.
*   **Tham số Yêu cầu (`[FromForm] ImageUploadRequestDto request`)**:
    *   `[FromForm]`: Thuộc tính này chỉ thị cho ASP.NET Core rằng dữ liệu cho tham số `request` sẽ đến từ dữ liệu biểu mẫu HTTP (`multipart/form-data`). Đây là cách tiêu chuẩn để gửi tệp và các trường văn bản khác trong một yêu cầu HTTP duy nhất.
*   **Xác thực phía máy chủ (`ValidateFileUpload` và `ModelState`)**:
    *   **Tầm quan trọng**: Việc xác thực phía máy chủ là bắt buộc và không thể bỏ qua, ngay cả khi bạn có xác thực phía client. Client-side validation chỉ giúp cải thiện trải nghiệm người dùng, nhưng server-side validation là tuyến phòng thủ cuối cùng chống lại dữ liệu không hợp lệ hoặc độc hại.
    *   **Logic**:
        *   Chúng ta định nghĩa một mảng `_allowedExtensions` và `_maxFileSize` để kiểm soát các quy tắc tải lên.
        *   `Path.GetExtension(request.File.FileName).ToLowerInvariant()`: Lấy phần mở rộng của tệp từ tên gốc và chuyển nó thành chữ thường không phân biệt văn hóa để so sánh nhất quán. `ToLowerInvariant()` tốt hơn `ToLower()` vì nó đảm bảo hành vi nhất quán trên các môi trường ngôn ngữ khác nhau.
        *   `ModelState.AddModelError("file", "...");`: Nếu phát hiện lỗi (phần mở rộng không hợp lệ hoặc kích thước quá lớn), chúng ta thêm một lỗi vào `ModelState`. Key `"file"` là tên của thuộc tính trong DTO mà lỗi liên quan.
    *   **`ModelState.IsValid`**: Thuộc tính này sẽ là `false` nếu:
        *   Bất kỳ thuộc tính `[Required]` nào trên `ImageUploadRequestDto` không được đáp ứng.
        *   Bất kỳ lỗi xác thực tùy chỉnh nào được thêm vào bởi phương thức `ValidateFileUpload` của chúng ta.
    *   Nếu `ModelState.IsValid` là `false`, phương thức sẽ trả về `BadRequest(ModelState)`, cung cấp chi tiết lỗi cho client.
*   **Chuyển đổi DTO sang Mô hình Miền**:
    *   Nếu xác thực thành công, chúng ta tạo một đối tượng `Image` (mô hình miền) từ `ImageUploadRequestDto`.
    *   Các thuộc tính như `FileExtension` và `FileSizeInBytes` được trích xuất trực tiếp từ `request.File`.
    *   `FilePath` chưa được điền ở đây vì nó sẽ được xác định sau khi tệp được lưu vật lý.
*   **Sử dụng Repository (`_imageRepository.Upload`)**:
    *   Chúng ta ủy quyền toàn bộ logic lưu trữ hình ảnh (cả tệp vật lý và metadata vào DB) cho `_imageRepository`. Điều này tuân thủ Repository Pattern, giữ cho Controller nhẹ nhàng và tập trung vào việc xử lý yêu cầu HTTP.
    *   Repository sẽ trả về đối tượng `Image` đã được cập nhật, bao gồm `FilePath` đã được tạo.
*   **Trả về phản hồi (`Ok(imageDomainModel)`)**:
    *   Nếu tải lên và lưu trữ thành công, chúng ta trả về `Ok(imageDomainModel)` (HTTP 200 OK) cùng với đối tượng `Image` đã được lưu, bao gồm `Id` và `FilePath` đầy đủ, để client có thể sử dụng ngay lập tức.

> [!TIP]
> **Vibe Coding với Antigravity IDE (Controller & Validation)**: Để tạo `ImagesController` và phương thức `Upload` với Antigravity, bạn có thể bắt đầu với ý định: "Tạo một API Controller để xử lý việc tải lên hình ảnh. Yêu cầu cần có tệp hình ảnh, tên tệp và mô tả tùy chọn. Chỉ cho phép các định dạng JPG, PNG, GIF và kích thước tối đa 10MB."
>
> Antigravity sẽ:
> 1.  Tạo cấu trúc Controller, thêm `[Route]` và `[ApiController]`.
> 2.  Tự động tạo `ImageUploadRequestDto` với các thuộc tính `[Required]` phù hợp.
> 3.  Tạo phương thức `Upload` với `[HttpPost]` và `[FromForm]`.
> 4.  Thậm chí có thể tự động tạo phương thức `ValidateFileUpload` với logic kiểm tra phần mở rộng và kích thước, dựa trên các ràng buộc bạn đã mô tả.
> 5.  Đảm bảo rằng DTO được chuyển đổi thành mô hình miền trước khi gọi Repository.
>
> Điều này giúp bạn nhanh chóng có được một khung làm việc chuẩn mực, tập trung vào các quy tắc nghiệp vụ hơn là chi tiết triển khai.

## 4. Triển khai Repository Pattern cho Tải lên Hình ảnh

Để duy trì kiến trúc ứng dụng sạch sẽ, dễ bảo trì và khả năng kiểm thử cao, chúng ta sẽ áp dụng Repository Pattern. Pattern này tạo ra một lớp trừu tượng giữa tầng nghiệp vụ (Controller) và tầng truy cập dữ liệu (Entity Framework Core và hệ thống tệp), giúp tách biệt logic lưu trữ khỏi logic điều khiển.

> [!TIP]
> **Repository Pattern** là một design pattern giúp tạo ra một lớp trừu tượng giữa tầng nghiệp vụ và tầng truy cập dữ liệu. Nó cung cấp một tập hợp các phương thức để thực hiện các thao tác CRUD (Create, Read, Update, Delete) trên các thực thể, mà không cần tầng nghiệp vụ phải biết chi tiết về cách dữ liệu được lưu trữ, nơi lưu trữ (cơ sở dữ liệu, hệ thống tệp, dịch vụ đám mây), hoặc công nghệ truy cập dữ liệu cụ thể (EF Core, Dapper, ADO.NET). Lợi ích chính bao gồm:
> *   **Tách biệt các mối quan tâm**: Controller không cần biết chi tiết về việc lưu tệp hay tương tác với DB.
> *   **Khả năng kiểm thử**: Dễ dàng mock (giả lập) Repository trong các bài kiểm thử đơn vị cho Controller.
> *   **Tính linh hoạt**: Có thể thay đổi công nghệ lưu trữ dữ liệu hoặc vị trí lưu tệp mà không ảnh hưởng đến logic nghiệp vụ.

### 4.1. Định nghĩa Interface `IImageRepository`

Đầu tiên, chúng ta định nghĩa một interface cho `Image Repository`. Interface này sẽ mô tả các phương thức mà Repository phải cung cấp, thiết lập một "hợp đồng" mà các triển khai cụ thể phải tuân theo.

Tạo một interface mới có tên `IImageRepository.cs` trong thư mục `Repositories`:

```csharp
// Repositories/IImageRepository.cs
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    /// <summary>
    /// Interface cho Image Repository, định nghĩa các thao tác liên quan đến quản lý hình ảnh.
    /// </summary>
    public interface IImageRepository
    {
        /// <summary>
        /// Tải lên và lưu trữ một hình ảnh (cả tệp vật lý và metadata).
        /// </summary>
        /// <param name="image">Đối tượng Image chứa tệp và metadata cần lưu.</param>
        /// <returns>Đối tượng Image đã được cập nhật với FilePath sau khi lưu.</returns>
        Task<Image> Upload(Image image);
    }
}
```

Phương thức `Upload` nhận một đối tượng `Image` (mô hình miền) và trả về đối tượng `Image` sau khi xử lý (ví dụ: với thuộc tính `FilePath` đã được điền đầy đủ). Sử dụng `Task<Image>` cho thấy đây là một thao tác không đồng bộ.

### 4.2. Triển khai Local Image Repository

Tiếp theo, chúng ta sẽ triển khai interface `IImageRepository` bằng một lớp cụ thể, `LocalImageRepository`. Lớp này sẽ chịu trách nhiệm chính trong việc lưu trữ tệp hình ảnh vào hệ thống tệp cục bộ của máy chủ và lưu metadata hình ảnh vào cơ sở dữ liệu thông qua Entity Framework Core.

Tạo một lớp mới có tên `LocalImageRepository.cs` trong thư mục `Repositories`:

```csharp
// Repositories/LocalImageRepository.cs
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;
using Microsoft.AspNetCore.Hosting; // Cần thiết cho IWebHostEnvironment
using Microsoft.AspNetCore.Http; // Cần thiết cho IHttpContextAccessor
using System.IO; // Cần thiết cho Path, FileStream

namespace NZWalks.API.Repositories
{
    /// <summary>
    /// Triển khai IImageRepository để lưu trữ hình ảnh vào hệ thống tệp cục bộ
    /// và metadata vào cơ sở dữ liệu.
    /// </summary>
    public class LocalImageRepository : IImageRepository
    {
        private readonly IWebHostEnvironment _webHostEnvironment;
        private readonly IHttpContextAccessor _httpContextAccessor;
        private readonly ApplicationDbContext _dbContext;

        /// <summary>
        /// Constructor để inject các dependency cần thiết.
        /// </summary>
        /// <param name="webHostEnvironment">Cung cấp thông tin về môi trường lưu trữ web.</param>
        /// <param name="httpContextAccessor">Cung cấp quyền truy cập vào HttpContext hiện tại.</param>
        /// <param name="dbContext">DbContext để tương tác với cơ sở dữ liệu.</param>
        public LocalImageRepository(IWebHostEnvironment webHostEnvironment,
            IHttpContextAccessor httpContextAccessor,
            ApplicationDbContext dbContext)
        {
            _webHostEnvironment = webHostEnvironment;
            _httpContextAccessor = httpContextAccessor;
            _dbContext = dbContext;
        }

        /// <summary>
        /// Triển khai phương thức Upload để lưu tệp hình ảnh và metadata.
        /// </summary>
        /// <param name="image">Đối tượng Image chứa tệp và metadata.</param>
        /// <returns>Đối tượng Image đã được cập nhật với FilePath.</returns>
        public async Task<Image> Upload(Image image)
        {
            // 1. Chuẩn bị thư mục lưu trữ cục bộ
            // Ví dụ: C:\Projects\NZWalks.API\Images\
            var uploadFolderPath = Path.Combine(_webHostEnvironment.ContentRootPath, "Images");

            // Đảm bảo thư mục "Images" tồn tại, nếu không thì tạo mới.
            if (!Directory.Exists(uploadFolderPath))
            {
                Directory.CreateDirectory(uploadFolderPath);
            }

            // 2. Xây dựng đường dẫn tệp cục bộ đầy đủ
            // Ví dụ: C:\Projects\NZWalks.API\Images\my_profile_pic.jpg
            var localFilePath = Path.Combine(uploadFolderPath, $"{image.FileName}{image.FileExtension}");

            // 3. Tải lên (ghi) tệp hình ảnh vật lý vào đường dẫn cục bộ
            // Sử dụng "using" để đảm bảo FileStream được đóng và giải phóng tài nguyên.
            using (var stream = new FileStream(localFilePath, FileMode.Create))
            {
                await image.File.CopyToAsync(stream);
            }

            // 4. Xây dựng đường dẫn URL công khai để client có thể truy cập hình ảnh
            // Ví dụ: https://localhost:1234/Images/my_profile_pic.jpg
            // Scheme (http/https), Host (localhost:port), PathBase (nếu có), và đường dẫn ảo /Images
            var urlFilePath = $"{_httpContextAccessor.HttpContext.Request.Scheme}://{_httpContextAccessor.HttpContext.Request.Host}{_httpContextAccessor.HttpContext.Request.PathBase}/Images/{image.FileName}{image.FileExtension}";

            // Cập nhật thuộc tính FilePath trong mô hình miền
            image.FilePath = urlFilePath;

            // 5. Thêm metadata hình ảnh vào cơ sở dữ liệu
            await _dbContext.Images.AddAsync(image);
            await _dbContext.SaveChangesAsync();

            // 6. Trả về mô hình Image đã được cập nhật
            return image;
        }
    }
}
```

**Giải thích chi tiết các thành phần:**

*   **Dependency Injection**: Constructor của `LocalImageRepository` nhận các dependencies sau thông qua DI:
    *   **`IWebHostEnvironment _webHostEnvironment`**: Interface này cung cấp thông tin về môi trường lưu trữ web của ứng dụng. Cụ thể, chúng ta sử dụng `_webHostEnvironment.ContentRootPath` để lấy đường dẫn tuyệt đối đến thư mục gốc của ứng dụng. Điều này đảm bảo rằng chúng ta có thể xây dựng đường dẫn tệp vật lý một cách linh hoạt, không phụ thuộc vào thư mục làm việc hiện tại.
    *   **`IHttpContextAccessor _httpContextAccessor`**: Interface này cung cấp quyền truy cập vào `HttpContext` hiện tại của yêu cầu HTTP. Điều này rất quan trọng để xây dựng URL công khai đầy đủ cho hình ảnh, vì nó cho phép chúng ta lấy thông tin như scheme (`http` hoặc `https`), host (tên máy chủ và cổng), và đường dẫn cơ sở của yêu cầu hiện tại.
    *   **`ApplicationDbContext _dbContext`**: Instance của DbContext của chúng ta, được sử dụng để tương tác với cơ sở dữ liệu và lưu trữ metadata hình ảnh.
*   **Xây dựng đường dẫn tệp cục bộ (`localFilePath`)**:
    *   `Path.Combine()`: Đây là một phương thức tiện ích từ `System.IO` giúp kết hợp các phần của đường dẫn tệp một cách an toàn và tương thích với các hệ điều hành khác nhau (tự động xử lý dấu gạch chéo `/` hoặc `\`).
    *   `_webHostEnvironment.ContentRootPath`: Lấy đường dẫn gốc của dự án (ví dụ: `C:\Projects\MyWebApp`).
    *   Chúng ta tạo một thư mục con tên là `Images` trong thư mục gốc của dự án để lưu trữ tất cả các tệp hình ảnh được tải lên. Logic `Directory.Exists` và `Directory.CreateDirectory` đảm bảo thư mục này tồn tại.
    *   `$"{image.FileName}{image.FileExtension}"`: Kết hợp tên tệp và phần mở rộng để tạo tên tệp duy nhất.
*   **Lưu trữ hình ảnh vật lý (`FileStream`, `CopyToAsync`)**:
    *   `using (var stream = new FileStream(localFilePath, FileMode.Create))`: Mở một luồng tệp để ghi dữ liệu vào `localFilePath`. `FileMode.Create` sẽ tạo một tệp mới hoặc ghi đè nếu tệp đã tồn tại. Khối `using` đảm bảo rằng luồng tệp sẽ được đóng và tài nguyên hệ thống sẽ được giải phóng một cách an toàn ngay cả khi có lỗi xảy ra.
    *   `await image.File.CopyToAsync(stream);`: Đây là phương thức không đồng bộ để sao chép nội dung của `IFormFile` (tệp được tải lên) vào luồng tệp cục bộ. Việc sử dụng `async/await` giúp ứng dụng không bị chặn trong khi tệp đang được ghi, cải thiện khả năng mở rộng.
*   **Xây dựng đường dẫn URL công khai (`urlFilePath`)**:
    *   Đây là URL mà client sẽ sử dụng để truy cập hình ảnh sau khi nó đã được lưu trữ. Nó được xây dựng một cách động dựa trên yêu cầu HTTP hiện tại:
        *   `_httpContextAccessor.HttpContext.Request.Scheme`: Giao thức (ví dụ: `http` hoặc `https`).
        *   `_httpContextAccessor.HttpContext.Request.Host`: Tên máy chủ và cổng (ví dụ: `localhost:1234` hoặc `example.com`).
        *   `_httpContextAccessor.HttpContext.Request.PathBase`: Đường dẫn cơ sở của ứng dụng nếu nó được triển khai dưới một đường dẫn ảo (ví dụ: `/myapp`).
        *   `/Images/`: Đây là đường dẫn ảo mà chúng ta sẽ cấu hình trong `Program.cs` để phục vụ các tệp tĩnh từ thư mục `Images` vật lý.
        *   `image.FileName}{image.FileExtension}`: Tên tệp và phần mở rộng của hình ảnh.
    *   `image.FilePath = urlFilePath;`: Cập nhật thuộc tính `FilePath` của mô hình miền `Image` với URL công khai này.
*   **Lưu metadata vào cơ sở dữ liệu**:
    *   `await _dbContext.Images.AddAsync(image);`: Thêm đối tượng `Image` (hiện đã có `FilePath`) vào `DbSet<Image>` trong `DbContext`.
    *   `await _dbContext.SaveChangesAsync();`: Lưu các thay đổi vào cơ sở dữ liệu. Entity Framework Core sẽ tự động tạo một bản ghi mới trong bảng `Images` với tất cả các thuộc tính đã được điền.

> [!TIP]
> **Vibe Coding với Antigravity IDE (Repository)**: Khi triển khai Repository, bạn có thể nói với Antigravity IDE: "Tôi cần một Repository để lưu hình ảnh. Nó phải lưu tệp vật lý vào thư mục 'Images' trong thư mục gốc của ứng dụng và lưu metadata (bao gồm URL truy cập) vào cơ sở dữ liệu."
>
> Antigravity sẽ:
> 1.  Tự động inject các dependency cần thiết như `IWebHostEnvironment`, `IHttpContextAccessor`, và `ApplicationDbContext`.
> 2.  Xây dựng logic để kiểm tra và tạo thư mục `Images`.
> 3.  Tạo mã để xử lý `FileStream` và `CopyToAsync`.
> 4.  Xây dựng logic phức tạp để tạo `urlFilePath` bằng cách sử dụng `IHttpContextAccessor`.
> 5.  Viết mã để tương tác với `DbContext` để lưu metadata.
>
> Antigravity không chỉ viết mã mà còn giải thích *tại sao* các dependency đó cần thiết và *cách* chúng được sử dụng, giúp học viên hiểu sâu hơn về cơ chế ngầm.

### 4.3. Cấu hình Dependency Injection

Để ứng dụng có thể sử dụng `IImageRepository` và `LocalImageRepository`, chúng ta cần đăng ký chúng vào hệ thống Dependency Injection (DI) của ASP.NET Core. Ngoài ra, `IHttpContextAccessor` cũng là một dịch vụ cần được đăng ký.

> [!NOTE]
> **Dependency Injection (DI)** là một kỹ thuật trong lập trình mà trong đó một đối tượng (hoặc hàm) nhận các đối tượng mà nó phụ thuộc vào (các dependency) từ bên ngoài, thay vì tự tạo ra chúng. ASP.NET Core có một container DI tích hợp sẵn, nơi các dịch vụ (dependencies) được đăng ký và giải quyết tự động khi cần. Điều này thúc đẩy tính mô-đun, khả năng kiểm thử và khả năng mở rộng của ứng dụng.

Mở tệp `Program.cs` và thêm các dòng cấu hình DI sau:

```csharp
// Program.cs

// ... (các cấu hình builder.Services khác) ...

// Đăng ký HttpContextAccessor vào DI container.
// Cần thiết cho LocalImageRepository để xây dựng URL công khai.
builder.Services.AddHttpContextAccessor();

// Đăng ký Repository Pattern cho Image Upload.
// Khi IImageRepository được yêu cầu, một instance của LocalImageRepository sẽ được cung cấp.
// AddScoped: Một instance mới được tạo cho mỗi yêu cầu HTTP.
builder.Services.AddScoped<IImageRepository, LocalImageRepository>();

// ... (các cấu hình app.Use... khác) ...

var app = builder.Build();

// ... (các middleware khác) ...
```

**Giải thích các cấu hình DI:**

*   **`builder.Services.AddHttpContextAccessor()`**:
    *   Đăng ký dịch vụ `IHttpContextAccessor` vào container DI.
    *   `IHttpContextAccessor` là một dịch vụ đặc biệt cho phép bạn truy cập `HttpContext` hiện tại từ bất kỳ nơi nào trong ứng dụng mà không cần phải truyền nó qua chuỗi tham số.
    *   Nó cần thiết cho `LocalImageRepository` để có thể truy cập thông tin về yêu cầu HTTP hiện tại (scheme, host) để xây dựng URL công khai cho hình ảnh.
*   **`builder.Services.AddScoped<IImageRepository, LocalImageRepository>()`**:
    *   Đây là dòng đăng ký chính cho Repository Pattern của chúng ta.
    *   Nó nói với container DI rằng khi bất kỳ thành phần nào (ví dụ: `ImagesController`) yêu cầu một instance của `IImageRepository`, hãy cung cấp một instance của `LocalImageRepository`.
    *   **`AddScoped`**: Đây là một trong ba lifetime options chính cho các dịch vụ trong ASP.NET Core DI:
        *   **Scoped**: Một instance mới của dịch vụ được tạo *cho mỗi yêu cầu HTTP*. Instance đó sẽ được tái sử dụng trong suốt vòng đời của yêu cầu đó. Đây là lựa chọn phổ biến cho các dịch vụ liên quan đến yêu cầu (như `DbContext` và các Repository) vì nó đảm bảo tính nhất quán của dữ liệu trong một yêu cầu và giải phóng tài nguyên sau khi yêu cầu kết thúc.
        *   `AddTransient`: Một instance mới được tạo *mỗi khi dịch vụ được yêu cầu*. Phù hợp cho các dịch vụ nhẹ, không trạng thái.
        *   `AddSingleton`: Một instance duy nhất của dịch vụ được tạo *khi ứng dụng khởi động lần đầu tiên* và được tái sử dụng trong suốt vòng đời của ứng dụng. Phù hợp cho các dịch vụ có trạng thái toàn cầu hoặc chi phí khởi tạo cao.

> [!TIP]
> **Vibe Coding với Antigravity IDE (Dependency Injection)**: Với Antigravity IDE, việc cấu hình DI trở nên gần như tự động. Khi bạn tạo `IImageRepository` và `LocalImageRepository`, Antigravity có thể nhận ra rằng `LocalImageRepository` là một triển khai của `IImageRepository` và rằng `LocalImageRepository` phụ thuộc vào `IWebHostEnvironment`, `IHttpContextAccessor` và `ApplicationDbContext`.
>
> Antigravity có thể tự động đề xuất và thêm các dòng cấu hình DI cần thiết vào `Program.cs`, bao gồm `AddHttpContextAccessor()` và `AddScoped<IImageRepository, LocalImageRepository>()`. Nó thậm chí có thể giải thích lý do chọn `AddScoped` dựa trên các nguyên tắc thiết kế phổ biến cho các Repository trong ứng dụng web. Điều này giúp học viên tránh quên cấu hình DI – một lỗi phổ biến khi làm việc thủ công.

## 5. Phục vụ Tệp Tĩnh trong ASP.NET Core Web API

Theo mặc định, một ứng dụng ASP.NET Core Web API được cấu hình để xử lý các yêu cầu API, chứ không phải để phục vụ các tệp tĩnh (như hình ảnh, CSS, JavaScript). Để client có thể truy cập hình ảnh đã tải lên thông qua `FilePath` (URL công khai) mà chúng ta đã tạo trong Repository, chúng ta cần cấu hình API để phục vụ các tệp tĩnh từ thư mục `Images` vật lý.

Mở tệp `Program.cs` và thêm middleware `UseStaticFiles` vào pipeline xử lý yêu cầu HTTP. Điều quan trọng là vị trí của middleware này trong pipeline.

```csharp
// Program.cs

// ... (các cấu hình middleware khác) ...

// Ví dụ: app.UseHttpsRedirection();
// Ví dụ: app.UseRouting();

app.UseAuthentication(); // Nếu ứng dụng có xác thực
app.UseAuthorization();  // Nếu ứng dụng có phân quyền

// Cấu hình để phục vụ các tệp tĩnh từ thư mục "Images"
// (Đảm bảo thư mục "Images" tồn tại trong thư mục gốc của dự án hoặc được tạo bởi Repository)
app.UseStaticFiles(new StaticFileOptions
{
    // Chỉ định nhà cung cấp tệp vật lý: lấy tệp từ thư mục "Images" bên trong ContentRootPath
    FileProvider = new PhysicalFileProvider(Path.Combine(builder.Environment.ContentRootPath, "Images")),
    // Định nghĩa đường dẫn yêu cầu ảo: bất kỳ yêu cầu nào đến /Images/{tên_tệp} sẽ được ánh xạ tới thư mục vật lý
    RequestPath = "/Images"
});

app.MapControllers(); // Middleware để ánh xạ các yêu cầu HTTP tới các Controller

app.Run();
```

**Giải thích chi tiết:**

*   **`app.UseStaticFiles()`**: Middleware này kích hoạt khả năng phục vụ các tệp tĩnh trong ứng dụng ASP.NET Core. Khi một yêu cầu HTTP đến, middleware này sẽ kiểm tra xem yêu cầu có khớp với một tệp tĩnh được cấu hình hay không.
*   **`new StaticFileOptions`**: Đối tượng này cho phép chúng ta tùy chỉnh cách các tệp tĩnh được phục vụ:
    *   **`FileProvider = new PhysicalFileProvider(Path.Combine(builder.Environment.ContentRootPath, "Images"))`**:
        *   `FileProvider`: Chỉ định nơi middleware `StaticFiles` nên tìm kiếm các tệp vật lý.
        *   `PhysicalFileProvider`: Một triển khai của `IFileProvider` tìm kiếm các tệp trên hệ thống tệp vật lý của máy chủ.
        *   `Path.Combine(builder.Environment.ContentRootPath, "Images")`: Xây dựng đường dẫn tuyệt đối đến thư mục `Images` trong thư mục gốc của dự án. Điều này có nghĩa là các tệp tĩnh sẽ được lấy từ thư mục này.
    *   **`RequestPath = "/Images"`**:
        *   `RequestPath`: Định nghĩa đường dẫn yêu cầu ảo (URL path prefix). Điều này có nghĩa là bất kỳ yêu cầu HTTP nào bắt đầu bằng `/Images/` (ví dụ: `https://localhost:port/Images/my_profile_pic.jpg`) sẽ được middleware `StaticFiles` xử lý. Middleware sẽ tìm kiếm `my_profile_pic.jpg` trong thư mục vật lý được chỉ định bởi `FileProvider` (`[ContentRootPath]/Images`).
*   **Lưu ý về thứ tự Middleware**:
    *   Thứ tự của middleware trong `Program.cs` là cực kỳ quan trọng vì chúng được thực thi theo thứ tự chúng được thêm vào pipeline.
    *   `UseStaticFiles` nên được đặt ở vị trí hợp lý. Thông thường, nó được đặt **sau** các middleware liên quan đến xác thực và phân quyền (`UseAuthentication()`, `UseAuthorization()`) nếu bạn muốn các tệp tĩnh có thể truy cập công khai mà không cần xác thực. Nếu bạn muốn bảo vệ các tệp tĩnh bằng xác thực/phân quyền, bạn sẽ đặt nó *trước* các middleware đó hoặc triển khai logic phân quyền tùy chỉnh.
    *   Nó phải được đặt **trước** `app.MapControllers()` để đảm bảo rằng yêu cầu đến tệp tĩnh được xử lý bởi `UseStaticFiles` trước khi nó được chuyển đến các API Controller. Nếu không, `MapControllers` có thể cố gắng định tuyến yêu cầu tệp tĩnh tới một Controller không tồn tại, dẫn đến lỗi 404.

Sau khi thực hiện thay đổi này, khi bạn truy cập một URL như `https://localhost:port/Images/yourimage.jpg`, ASP.NET Core sẽ tìm kiếm `yourimage.jpg` trong thư mục `Images` vật lý của dự án và phục vụ nó trực tiếp cho client.

> [!TIP]
> **Vibe Coding với Antigravity IDE (Static Files)**: Cấu hình `UseStaticFiles` đôi khi có thể gây nhầm lẫn về đường dẫn vật lý và đường dẫn ảo cũng như thứ tự middleware. Với Antigravity, bạn có thể đơn giản hóa bằng cách mô tả ý định: "Tôi muốn phục vụ các hình ảnh từ thư mục 'Images' trong dự án của tôi thông qua URL `/Images`."
>
> Antigravity, với kiến thức về cấu trúc dự án và pipeline middleware của ASP.NET Core, có thể tự động:
> 1.  Xác định vị trí tối ưu để chèn `app.UseStaticFiles` trong `Program.cs`.
> 2.  Tạo đối tượng `StaticFileOptions` với `FileProvider` và `RequestPath` chính xác, dựa trên mô tả của bạn.
> 3.  Giải thích ngắn gọn về lý do chọn vị trí đó trong pipeline middleware, tăng cường hiểu biết của học viên về kiến trúc ứng dụng.

## 6. Kiểm tra và Xác minh Chức năng Tải lên Hình ảnh

Bây giờ tất cả các thành phần đã được thiết lập và cấu hình, chúng ta có thể chạy ứng dụng và kiểm tra chức năng tải lên hình ảnh bằng Swagger UI (nếu được bật) hoặc một công cụ kiểm thử API như Postman.

1.  **Chạy ứng dụng**: Khởi động ứng dụng ASP.NET Core của bạn (ví dụ: nhấn F5 trong Visual Studio hoặc `dotnet run` từ terminal). Swagger UI sẽ tự động mở trong trình duyệt nếu được cấu hình.
2.  **Truy cập điểm cuối `Upload`**: Trong Swagger UI, tìm điểm cuối `POST /api/Images/Upload`.
3.  **Thử nghiệm với dữ liệu hợp lệ**:
    *   Nhấp vào "Try it out".
    *   Trong trường `File`, nhấp vào "Choose File" và chọn một tệp hình ảnh hợp lệ (ví dụ: `.jpg`, `.png`, `.gif`) có kích thước nhỏ hơn 10MB.
    *   Trong trường `FileName`, nhập một tên tệp mong muốn (ví dụ: "MyNewProfilePic").
    *   (Tùy chọn) Nhập `FileDescription`.
    *   Nhấp vào "Execute".

**Kết quả mong đợi:**

*   **Phản hồi thành công (200 OK)**: Nếu tải lên thành công, bạn sẽ nhận được phản hồi HTTP 200 OK. Body của phản hồi sẽ chứa đối tượng `Image` đã được lưu, bao gồm `Id` và `FilePath` đầy đủ (ví dụ: `https://localhost:port/Images/MyNewProfilePic.jpg`).
*   **Kiểm tra tệp vật lý**: Kiểm tra thư mục `Images` trong thư mục gốc của dự án của bạn. Bạn sẽ thấy tệp hình ảnh đã tải lên được lưu ở đó với tên và phần mở rộng đã chỉ định.
*   **Kiểm tra cơ sở dữ liệu**: Kiểm tra bảng `Images` trong cơ sở dữ liệu của bạn. Bạn sẽ thấy một hàng dữ liệu mới chứa metadata của hình ảnh, bao gồm `FilePath` đã được lưu.
*   **Truy cập hình ảnh qua URL**: Sao chép `FilePath` từ phản hồi API hoặc từ cơ sở dữ liệu. Dán URL này vào trình duyệt của bạn. Bạn sẽ thấy hình ảnh được hiển thị trực tiếp. Nếu không, hãy kiểm tra lại cấu hình `UseStaticFiles` trong `Program.cs` và đảm bảo đường dẫn vật lý/ảo khớp.

**Kiểm tra xác thực (dữ liệu không hợp lệ):**

*   **Tệp không phải hình ảnh**: Thử tải lên một tệp không phải hình ảnh (ví dụ: `.txt`, `.pdf`).
*   **Tệp quá lớn**: Thử tải lên một tệp hình ảnh có kích thước lớn hơn 10MB.
*   **Thiếu trường bắt buộc**: Thử gửi yêu cầu mà không chọn tệp hoặc không cung cấp `FileName`.

**Kết quả mong đợi cho xác thực:**

*   Bạn sẽ nhận được phản hồi `400 Bad Request` với các thông báo lỗi xác thực chi tiết trong body phản hồi (ví dụ: "Unsupported file extension.", "File size is too large.", "File name is required.").

---

## Tóm tắt Chương 12: Xử lý Tải lên Hình ảnh

Chương này đã trang bị cho bạn một giải pháp toàn diện để xử lý tải lên hình ảnh trong ứng dụng ASP.NET Core Web API, tuân thủ các nguyên tắc thiết kế tốt và dễ dàng mở rộng trong tương lai. Các điểm chính bao gồm:

*   **Mô hình Miền `Image`**: Chúng ta đã thiết kế một mô hình `Image` để lưu trữ metadata hình ảnh, sử dụng `[NotMapped]` cho `IFormFile File` để tránh lưu trữ dữ liệu nhị phân trực tiếp trong cơ sở dữ liệu.
*   **Cập nhật DbContext và Migrations**: Đã thêm `DbSet<Image>` vào `ApplicationDbContext` và sử dụng Entity Framework Core Migrations để tạo bảng `Images` trong cơ sở dữ liệu.
*   **API Controller và DTO**:
    *   Tạo `ImagesController` với `[ApiController]` và `[Route]` để xử lý các yêu cầu HTTP POST.
    *   Định nghĩa `ImageUploadRequestDto` làm DTO đầu vào, tách biệt yêu cầu client khỏi mô hình miền và tạo một bề mặt rõ ràng cho xác thực.
    *   Sử dụng `[FromForm]` trong phương thức `Upload` để nhận dữ liệu từ biểu mẫu HTTP.
*   **Xác thực phía máy chủ**: Triển khai logic xác thực tùy chỉnh trong phương thức `ValidateFileUpload` để kiểm tra phần mở rộng tệp và kích thước tệp, sử dụng `ModelState.AddModelError` để báo cáo lỗi.
*   **Repository Pattern**:
    *   Định nghĩa interface `IImageRepository` để trừu tượng hóa logic lưu trữ.
    *   Triển khai `LocalImageRepository` chịu trách nhiệm lưu trữ tệp hình ảnh vật lý vào hệ thống tệp cục bộ (`Images` folder) và lưu metadata (bao gồm URL công khai) vào cơ sở dữ liệu thông qua `ApplicationDbContext`.
    *   Sử dụng `IWebHostEnvironment` để lấy đường dẫn gốc của ứng dụng và `IHttpContextAccessor` để xây dựng URL công khai.
*   **Dependency Injection**: Đã cấu hình `Program.cs` để đăng ký `IHttpContextAccessor` và `IImageRepository` (với triển khai `LocalImageRepository`) vào container DI của ASP.NET Core, sử dụng `AddScoped` cho lifetime phù hợp.
*   **Phục vụ Tệp Tĩnh**: Cấu hình middleware `app.UseStaticFiles` trong `Program.cs` để cho phép ASP.NET Core Web API phục vụ các tệp từ thư mục `Images` vật lý thông qua đường dẫn yêu cầu ảo `/Images`.
*   **Vibe Coding và Antigravity IDE**: Xuyên suốt chương, chúng ta đã liên hệ cách một hệ thống AI như Antigravity IDE có thể hỗ trợ từng bước trong quá trình này, từ việc tạo mô hình, migrations, controller, validation, đến cấu hình DI và static files, giúp học viên tập trung vào ý định và kiến trúc thay vì chi tiết triển khai thủ công.

Sau chương này, bạn đã nắm vững một giải pháp hoàn chỉnh và tuân thủ các nguyên tắc thiết kế tốt để xử lý tải lên hình ảnh, đặt nền tảng vững chắc cho các tính năng đa phương tiện trong ứng dụng của bạn.

<!-- REVIEWED_BY_AGENT -->
