# Bài 20: Dự án thực hành: API Vùng (Regions) - Đọc dữ liệu & Repository

Trong hành trình xây dựng API RESTful cho dự án "NZWalks", Phần 20 đánh dấu bước khởi đầu quan trọng: phát triển khả năng đọc dữ liệu vùng (Regions). Chúng ta sẽ thiết lập một API Controller để tiếp nhận và xử lý các yêu cầu, đồng thời giới thiệu **Repository Pattern** – một mẫu thiết kế kiến trúc cốt lõi giúp tách biệt logic truy cập dữ liệu khỏi các Controller. Việc áp dụng mẫu thiết kế này không chỉ tăng cường khả năng kiểm thử và bảo trì mã nguồn mà còn thúc đẩy nguyên tắc tách biệt mối quan tâm (separation of concerns) trong kiến trúc ứng dụng. Cuối cùng, chúng ta sẽ kiểm định tính đúng đắn của API thông qua các công cụ như Swagger UI và Postman.

> [!NOTE]
> Khóa học này tập trung vào ASP.NET Core và Entity Framework Core. Các khái niệm trọng tâm bao gồm Dependency Injection, Repository Pattern, Controllers và các HTTP Verbs.

## 1. Xây dựng API Vùng Đầu Tiên: Nền tảng Controller

Mọi API RESTful đều bắt đầu với các Controller – những điểm cuối (endpoints) tiếp nhận yêu cầu từ client và điều phối phản hồi. Chúng ta sẽ bắt đầu bằng việc thiết lập `RegionsController` để quản lý các thao tác liên quan đến dữ liệu vùng.

### 1.1. Tạo và Cấu hình Regions Controller

Trong một dự án ASP.NET Core, các Controller được tổ chức trong thư mục `Controllers`.

1.  **Tạo Controller:**
    *   Trong Solution Explorer, nhấp chuột phải vào thư mục `Controllers`.
    *   Chọn **Add** > **Controller...**.
    *   Chọn mẫu **API Controller - Empty**.
    *   Đặt tên cho Controller là `RegionsController`.

    Sau khi tạo, Visual Studio sẽ tạo tệp `RegionsController.cs` với cấu trúc lớp cơ bản, kế thừa từ `ControllerBase`.

2.  **Cấu hình API Controller với Attributes:**
    Để `RegionsController` hoạt động như một API Controller thực thụ, chúng ta cần áp dụng hai thuộc tính (attributes) quan trọng:

    *   **`[ApiController]`**: Thuộc tính này không chỉ đơn thuần đánh dấu một lớp là API Controller mà còn kích hoạt một loạt các hành vi tiện ích và quy ước cho việc phát triển API. Nó tự động xử lý:
        *   **Model validation error handling**: Khi một yêu cầu đến và dữ liệu đầu vào không hợp lệ theo các `Data Annotation` trong model, `[ApiController]` sẽ tự động trả về phản hồi HTTP 400 Bad Request với chi tiết lỗi, thay vì yêu cầu bạn phải kiểm tra `ModelState.IsValid` thủ công.
        *   **Binding source parameter inference**: Giúp ASP.NET Core suy luận nguồn dữ liệu cho các tham số phương thức hành động (action methods). Ví dụ, nếu tham số là một kiểu phức tạp, nó sẽ được đọc từ body của yêu cầu; nếu là kiểu đơn giản, nó sẽ được đọc từ query string hoặc route.
        *   **Problem details for error statuses**: Tự động định dạng các phản hồi lỗi theo chuẩn RFC 7807 (Problem Details for HTTP APIs), cung cấp thông tin lỗi nhất quán và dễ phân tích.

    *   **`[Route("api/[controller]")]`**: Thuộc tính này xác định mẫu URL mà Controller sẽ phản hồi.
        *   `api/`: Là một tiền tố phổ biến để phân biệt các API endpoint với các trang web truyền thống.
        *   `[controller]`: Là một placeholder sẽ được ASP.NET Core thay thế bằng tên của Controller (bỏ hậu tố "Controller"). Trong trường hợp này, `RegionsController` sẽ trở thành `regions`.
        *   Kết quả là, Controller này sẽ phản hồi các yêu cầu tới URL `/api/regions`.

    **Ví dụ cấu hình:**

    ```csharp
    // Controllers/RegionsController.cs
    using Microsoft.AspNetCore.Mvc;

    namespace NZWalks.API.Controllers
    {
        [ApiController] // Kích hoạt các tính năng API
        [Route("api/[controller]")] // Định tuyến endpoint: /api/regions
        public class RegionsController : ControllerBase // Base class cho API Controller
        {
            // ... (các phương thức hành động sẽ được thêm vào đây)
        }
    }
    ```

    > [!TIP]
    > **Antigravity IDE và Vibe Coding:**
    > Khi làm việc với Antigravity IDE, việc tạo và cấu hình Controller trở nên trực quan hơn. Thay vì các bước thủ công, bạn có thể "vibe" ý định của mình: "Tạo một API Controller tên là `RegionsController` để quản lý các vùng. Nó nên có endpoint `/api/regions` và kế thừa các tính năng chuẩn của API Controller." Antigravity sẽ hiểu ngữ cảnh ASP.NET Core, tự động tạo tệp, thêm `[ApiController]`, `[Route]`, và cấu trúc cơ bản, thậm chí gợi ý namespace và các `using` cần thiết. Đây là một ví dụ điển hình của việc để AI Agent xử lý các tác vụ lặp đi lặp lại, cho phép bạn tập trung vào logic nghiệp vụ cao cấp hơn.

### 1.2. Triển khai Phương thức `GetAllRegions` (Dữ liệu tĩnh)

Để kiểm tra nhanh hoạt động của Controller, chúng ta sẽ tạo một phương thức trả về dữ liệu tĩnh.

*   **`[HttpGet]`**: Thuộc tính này đánh dấu phương thức là một HTTP GET endpoint. Khi không có tham số route nào được chỉ định cho `[HttpGet]`, nó sẽ ánh xạ tới đường dẫn gốc của Controller (ví dụ: `/api/regions`).
*   **`public IActionResult GetAllRegions()`**:
    *   Phương thức này sẽ xử lý yêu cầu GET.
    *   `IActionResult` là một giao diện rất mạnh mẽ trong ASP.NET Core, cho phép phương thức hành động trả về nhiều loại phản hồi HTTP khác nhau (ví dụ: `Ok` cho 200 OK, `NotFound` cho 404 Not Found, `BadRequest` cho 400 Bad Request, `Created` cho 201 Created, v.v.). Nó giúp trừu tượng hóa việc tạo phản hồi HTTP, cho phép bạn tập trung vào dữ liệu cần trả về.

**Mã nguồn ban đầu với dữ liệu tĩnh:**

```csharp
// Controllers/RegionsController.cs
using Microsoft.AspNetCore.Mvc;
using NZWalks.Models.Domain; // Đảm bảo đã import namespace của lớp Region
using System; // Để sử dụng Guid.NewGuid()
using System.Collections.Generic; // Để sử dụng List

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class RegionsController : ControllerBase
    {
        // Phương thức để lấy tất cả các vùng
        [HttpGet] // Phản hồi yêu cầu HTTP GET
        public IActionResult GetAllRegions()
        {
            // Khởi tạo một danh sách các đối tượng Region với dữ liệu tĩnh
            var regions = new List<Region>
            {
                new Region
                {
                    Id = Guid.NewGuid(), // Tạo GUID duy nhất cho mỗi vùng
                    Name = "Wellington",
                    Code = "WLG",
                    RegionImageUrl = "https://example.com/wlg.jpg" // Ví dụ URL hình ảnh
                },
                new Region
                {
                    Id = Guid.NewGuid(),
                    Name = "Auckland",
                    Code = "AKL",
                    RegionImageUrl = "https://example.com/akl.jpg" // Ví dụ URL hình ảnh
                }
            };

            // Trả về phản hồi HTTP 200 OK cùng với danh sách các vùng dưới dạng JSON
            return Ok(regions);
        }
    }
}
```

## 2. Kiểm thử API: Xác thực Chức năng Cơ bản

Sau khi đã thiết lập Controller và phương thức `GetAllRegions` với dữ liệu tĩnh, bước tiếp theo là xác nhận rằng API hoạt động như mong đợi.

### 2.1. Chạy Ứng dụng

1.  **Khởi động ứng dụng:** Trong Visual Studio, nhấn `Ctrl+F5` (Start Without Debugging) hoặc nút "Run".
2.  Ứng dụng sẽ được biên dịch và chạy. Nếu bạn đã cấu hình Swagger UI, trình duyệt sẽ tự động mở và điều hướng đến giao diện Swagger.

### 2.2. Kiểm thử với Swagger UI

Swagger UI là một công cụ tuyệt vời để khám phá và kiểm thử API của bạn ngay trong trình duyệt.

1.  **Tìm Endpoint:** Trong Swagger UI, bạn sẽ thấy `RegionsController` mới và phương thức `GET /api/Regions` được liệt kê.
2.  **Mở rộng và Thực thi:**
    *   Nhấp vào `GET /api/Regions` để mở rộng chi tiết của endpoint.
    *   Nhấp vào nút **Try it out**.
    *   Nhấp vào nút **Execute**.
3.  **Xem Phản hồi:** Bạn sẽ thấy phản hồi HTTP 200 OK và phần "Response body" sẽ hiển thị dữ liệu JSON của hai vùng (Wellington và Auckland) mà chúng ta đã định nghĩa tĩnh.

### 2.3. Kiểm thử với Postman hoặc Trình duyệt

Bạn cũng có thể kiểm thử API bằng cách sử dụng các công cụ khác:

1.  **Lấy URL:** Sao chép URL của API (ví dụ: `https://localhost:7000/api/regions`) từ Swagger UI hoặc thanh địa chỉ của trình duyệt khi ứng dụng đang chạy.
2.  **Gửi yêu cầu:**
    *   **Trong trình duyệt:** Dán URL vào một tab trình duyệt mới và nhấn Enter. Trình duyệt sẽ hiển thị phản hồi JSON.
    *   **Trong Postman:** Tạo một yêu cầu GET mới, dán URL vào trường địa chỉ và nhấn "Send".
3.  **Xác nhận:** Phản hồi nhận được sẽ là dữ liệu JSON tương tự như trong Swagger UI.

> [!NOTE]
> **Antigravity IDE và Kiểm thử:**
> Antigravity có thể tự động tạo ra các tập lệnh kiểm thử. Bạn có thể "vibe" cho Antigravity: "Sau khi tạo `RegionsController` với phương thức `GetAllRegions` trả về dữ liệu tĩnh, hãy tạo một yêu cầu GET đến `/api/regions` và hiển thị phản hồi." Antigravity có thể thực thi yêu cầu HTTP ngầm, phân tích JSON và thậm chí so sánh với một mẫu phản hồi mong đợi, giúp bạn xác nhận chức năng ngay lập tức mà không cần chuyển đổi công cụ.

## 3. Kiến trúc Cốt lõi: Hiểu sâu về Repository Pattern

Mặc dù việc sử dụng dữ liệu tĩnh rất hữu ích cho việc kiểm thử nhanh, nhưng trong một ứng dụng thực tế, Controller cần tương tác với cơ sở dữ liệu. Việc cho phép Controller gọi trực tiếp `DbContext` (Entity Framework Core) để truy vấn dữ liệu là một thực hành không được khuyến khích.

### 3.1. Các vấn đề khi Controller truy cập `DbContext` trực tiếp

1.  **Phụ thuộc chặt chẽ (Tight Coupling):**
    *   Controller sẽ bị ràng buộc trực tiếp với cách triển khai truy cập dữ liệu cụ thể (ví dụ: Entity Framework Core). Điều này có nghĩa là nếu bạn quyết định thay đổi công nghệ truy cập dữ liệu (ví dụ: chuyển từ EF Core sang Dapper, hoặc sử dụng một cơ sở dữ liệu NoSQL), bạn sẽ phải sửa đổi *tất cả* các Controller và các lớp khác trực tiếp sử dụng `DbContext`.
    *   Điều này vi phạm nguyên tắc "Depend upon abstractions, not concretions" (Phụ thuộc vào trừu tượng, không phải vào triển khai cụ thể).

2.  **Khó kiểm thử (Difficult to Test):**
    *   Khi Controller trực tiếp gọi `DbContext`, việc viết unit test cho Controller trở nên phức tạp. Bạn sẽ cần một cơ sở dữ liệu thực sự đang chạy hoặc một `DbContext` giả (mock) phức tạp để kiểm thử, điều này làm chậm quá trình kiểm thử và tăng độ phức tạp của môi trường kiểm thử.
    *   Unit test lý tưởng nên kiểm thử một đơn vị mã độc lập, không phụ thuộc vào tài nguyên bên ngoài như cơ sở dữ liệu.

3.  **Vi phạm nguyên tắc trách nhiệm đơn nhất (Single Responsibility Principle - SRP):**
    *   SRP nói rằng một lớp chỉ nên có một lý do để thay đổi. Controller có trách nhiệm xử lý yêu cầu HTTP, điều phối logic nghiệp vụ và trả về phản hồi. Việc quản lý chi tiết truy cập dữ liệu (tạo truy vấn, xử lý kết nối, v.v.) là một trách nhiệm khác.
    *   Khi Controller làm cả hai việc, nó có hai lý do để thay đổi, làm cho mã khó hiểu, khó bảo trì và dễ xảy ra lỗi hơn.

### 3.2. Repository Pattern là gì?

Repository Pattern là một mẫu thiết kế kiến trúc giúp giải quyết các vấn đề trên bằng cách trừu tượng hóa lớp truy cập dữ liệu. Nó đóng vai trò như một lớp trung gian giữa logic nghiệp vụ (ví dụ: Controller) và lớp truy cập dữ liệu (ví dụ: Entity Framework Core `DbContext`).

**Cấu trúc cơ bản:**

1.  **Interface của Repository (e.g., `IRegionRepository`)**:
    *   Đây là "hợp đồng" định nghĩa các thao tác dữ liệu mà Repository sẽ cung cấp (ví dụ: `GetAll`, `GetById`, `Add`, `Update`, `Delete`).
    *   Controller sẽ chỉ phụ thuộc vào *interface* này, không phải lớp triển khai cụ thể. Điều này mang lại sự linh hoạt cao.

2.  **Lớp triển khai Repository (e.g., `RegionRepository`)**:
    *   Chứa logic cụ thể để tương tác với cơ sở dữ liệu (sử dụng `DbContext` của Entity Framework Core, Dapper, SQL thuần, v.v.).
    *   Triển khai các phương thức được định nghĩa trong Interface.

3.  **Dependency Injection (DI)**:
    *   Là một kỹ thuật thiết kế phần mềm mà các đối tượng (dependencies) được cung cấp cho một đối tượng khác, thay vì đối tượng tự tạo ra hoặc tìm kiếm các dependencies của nó.
    *   Trong ASP.NET Core, DI là một phần cốt lõi, được sử dụng để "tiêm" (inject) lớp triển khai Repository vào Controller thông qua constructor.
    *   Controller không cần biết Repository được triển khai như thế nào, nó chỉ cần biết nó có thể gọi các phương thức trên Interface.

### 3.3. Lợi ích của Repository Pattern

*   **Tách biệt quan tâm (Separation of Concerns):**
    *   Controller chỉ tập trung vào logic API (xử lý yêu cầu, định dạng phản hồi).
    *   Repository chỉ tập trung vào logic truy cập dữ liệu (truy vấn, lưu trữ).
    *   Điều này giúp mã nguồn dễ hiểu, dễ quản lý hơn.

*   **Khả năng kiểm thử (Testability):**
    *   Dễ dàng mock (giả lập) Interface Repository trong unit test cho Controller. Bạn có thể tạo một đối tượng giả mạo `IRegionRepository` trả về dữ liệu kiểm thử mà không cần tương tác với cơ sở dữ liệu thực.
    *   Điều này cho phép unit test chạy nhanh chóng và độc lập.

*   **Linh hoạt (Flexibility):**
    *   Có thể thay đổi công nghệ truy cập dữ liệu (ví dụ: từ EF Core sang Dapper) bằng cách tạo một lớp triển khai Repository mới mà không ảnh hưởng đến Controller hoặc các lớp nghiệp vụ khác. Chỉ cần thay đổi cấu hình DI.

*   **Tái sử dụng mã (Code Reusability):**
    *   Logic truy cập dữ liệu có thể được đóng gói và tái sử dụng trên nhiều Controller hoặc dịch vụ khác nhau.

> [!TIP]
> **Antigravity IDE và Vibe Coding Repository Pattern:**
> Đây là một lĩnh vực mà Antigravity thực sự tỏa sáng. Thay vì phải nhớ cấu trúc thư mục, cú pháp interface, lớp triển khai, và cách inject `DbContext`, bạn có thể "vibe" cho Antigravity: "Tôi muốn triển khai Repository Pattern cho `Region` entity. Nó cần một interface `IRegionRepository` với phương thức `GetAll()` và một lớp triển khai `RegionRepository` sử dụng `NZWalksDbContext`."
> Antigravity, với khả năng hiểu sâu về các mẫu thiết kế và ngữ cảnh ASP.NET Core/EF Core, sẽ:
> 1.  Tạo thư mục `Repositories`.
> 2.  Tạo tệp `IRegionRepository.cs` với interface đã định nghĩa.
> 3.  Tạo tệp `RegionRepository.cs`, triển khai interface và inject `NZWalksDbContext` vào constructor.
> 4.  Thậm chí, nó có thể gợi ý hoặc tự động thêm `using` statements và cấu hình Dependency Injection trong `Program.cs`.
> Điều này giúp bạn tập trung vào *ý tưởng* kiến trúc thay vì chi tiết triển khai, tăng tốc độ phát triển và giảm thiểu lỗi cú pháp.

## 4. Triển khai Repository Pattern cho Vùng (Regions)

Bây giờ, chúng ta sẽ áp dụng Repository Pattern vào dự án NZWalks.

### 4.1. Cấu trúc thư mục

Tạo một thư mục mới có tên `Repositories` trong dự án của bạn để chứa các interface và lớp triển khai Repository.

### 4.2. Định nghĩa Interface `IRegionRepository`

Tạo một interface mới trong thư mục `Repositories` có tên `IRegionRepository.cs`.

```csharp
// Repositories/IRegionRepository.cs
using NZWalks.Models.Domain; // Import namespace của lớp Region
using System.Collections.Generic; // Để sử dụng IEnumerable

namespace NZWalks.API.Repositories
{
    public interface IRegionRepository
    {
        // Định nghĩa phương thức để lấy tất cả các vùng
        // Trả về một tập hợp các đối tượng Region
        IEnumerable<Region> GetAll();
    }
}
```

> [!NOTE]
> `IEnumerable<Region>` được sử dụng ở đây vì nó biểu thị một tập hợp có thể lặp lại, nhưng không tiết lộ cách thức dữ liệu được lưu trữ hoặc truy cập. Nó là một hình thức trừu tượng nhẹ, phù hợp cho việc trả về danh sách các đối tượng.

### 4.3. Triển khai `RegionRepository`

Tạo một lớp mới trong thư mục `Repositories` có tên `RegionRepository.cs`. Lớp này sẽ triển khai `IRegionRepository` và tương tác với `NZWalksDbContext` để lấy dữ liệu từ cơ sở dữ liệu.

```csharp
// Repositories/RegionRepository.cs
using NZWalks.API.Data; // Import namespace của DbContext (nơi NZWalksDbContext được định nghĩa)
using NZWalks.Models.Domain;
using System.Collections.Generic;
using System.Linq; // Để sử dụng ToList()

namespace NZWalks.API.Repositories
{
    public class RegionRepository : IRegionRepository
    {
        private readonly NZWalksDbContext _dbContext; // Field để lưu trữ DbContext

        // Constructor để inject NZWalksDbContext
        // ASP.NET Core DI sẽ tự động cung cấp một thể hiện của NZWalksDbContext
        public RegionRepository(NZWalksDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        // Triển khai phương thức GetAll từ IRegionRepository
        public IEnumerable<Region> GetAll()
        {
            // Truy vấn dữ liệu từ bảng Regions trong cơ sở dữ liệu
            // _dbContext.Regions đại diện cho DbSet<Region> đã được cấu hình trong DbContext
            // ToList() thực thi truy vấn SQL và trả về dữ liệu dưới dạng List<Region>
            return _dbContext.Regions.ToList();
        }
    }
}
```

> [!NOTE]
> Trong ví dụ này, chúng ta đang sử dụng phương thức `ToList()` đồng bộ. Trong các ứng dụng thực tế, đặc biệt là với API, việc sử dụng các phương thức bất đồng bộ (`ToListAsync()`) là một best practice để tránh chặn luồng xử lý (thread blocking) và cải thiện khả năng mở rộng (scalability) của ứng dụng. Chúng ta sẽ tìm hiểu về `async/await` và cách áp dụng chúng trong các phần sau.

### 4.4. Đăng ký Dependency Injection (DI)

Để ASP.NET Core biết cách cung cấp một thể hiện của `RegionRepository` khi `IRegionRepository` được yêu cầu, chúng ta cần đăng ký ánh xạ này trong bộ sưu tập dịch vụ của ứng dụng. Điều này được thực hiện trong tệp `Program.cs`.

Sử dụng phương thức `AddScoped()` để đăng ký dịch vụ.

*   **`AddScoped<TService, TImplementation>()`**:
    *   Tạo một thể hiện mới của dịch vụ (`TImplementation`) cho *mỗi yêu cầu HTTP*.
    *   Thể hiện này được chia sẻ trong toàn bộ phạm vi của yêu cầu đó. Điều này có nghĩa là nếu trong cùng một yêu cầu HTTP, nhiều thành phần yêu cầu `IRegionRepository`, chúng sẽ nhận được cùng một thể hiện của `RegionRepository`.
    *   Đây là lựa chọn phổ biến và phù hợp nhất cho các dịch vụ truy cập dữ liệu như Repository, nơi bạn muốn một `DbContext` (cũng thường được đăng ký là `Scoped`) được chia sẻ trong suốt một yêu cầu nhưng không tồn tại giữa các yêu cầu khác nhau.

*   **`AddTransient<TService, TImplementation>()`**:
    *   Tạo một thể hiện mới của dịch vụ mỗi khi nó được *yêu cầu*.
    *   Nếu trong cùng một yêu cầu HTTP, hai thành phần yêu cầu `IRegionRepository`, chúng sẽ nhận được hai thể hiện riêng biệt của `RegionRepository`.
    *   Thường được sử dụng cho các dịch vụ nhẹ, không trạng thái.

*   **`AddSingleton<TService, TImplementation>()`**:
    *   Tạo một thể hiện duy nhất của dịch vụ khi ứng dụng khởi động lần đầu và tái sử dụng nó trong suốt vòng đời của ứng dụng.
    *   Tất cả các yêu cầu HTTP và tất cả các thành phần sẽ nhận được cùng một thể hiện của dịch vụ.
    *   Thường được sử dụng cho các dịch vụ có trạng thái toàn cục, các logger, hoặc các cấu hình không thay đổi.

**Cập nhật `Program.cs`:**

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Repositories; // Import namespace của Repositories

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Đăng ký DbContext với Dependency Injection
builder.Services.AddDbContext<NZWalksDbContext>(options =>
{
    // Cấu hình để sử dụng SQL Server và lấy chuỗi kết nối từ cấu hình ứng dụng
    options.UseSqlServer(builder.Configuration.GetConnectionString("NZWalksConnectionString"));
});

// Đăng ký Repository Pattern cho Regions
// Khi IRegionRepository được yêu cầu, ASP.NET Core sẽ cung cấp một thể hiện của RegionRepository
// Lifetime là Scoped: Một thể hiện cho mỗi yêu cầu HTTP
builder.Services.AddScoped<IRegionRepository, RegionRepository>();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

> [!TIP]
> **Antigravity IDE và Cấu hình DI:**
> Với Antigravity, bạn có thể "vibe" một cách trừu tượng hơn: "Đăng ký `IRegionRepository` với `RegionRepository` để Dependency Injection có thể cung cấp nó cho các Controller." Antigravity sẽ tự động xác định vị trí `Program.cs`, thêm dòng `builder.Services.AddScoped<IRegionRepository, RegionRepository>();` và đảm bảo các `using` statements cần thiết được thêm vào. Nó hiểu rằng `RegionRepository` phụ thuộc vào `NZWalksDbContext` (cũng là một dịch vụ `Scoped`), do đó `AddScoped` là lựa chọn thích hợp để tránh các vấn đề về vòng đời (capturing lifetime mismatches).

### 4.5. Cập nhật `RegionsController`

Cuối cùng, chúng ta sẽ sửa đổi `RegionsController` để sử dụng `IRegionRepository` đã được inject, thay vì dữ liệu tĩnh.

```csharp
// Controllers/RegionsController.cs
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Repositories; // Import namespace của Repositories
using NZWalks.Models.Domain;
using System.Collections.Generic;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository _regionRepository; // Field để lưu trữ Repository

        // Constructor để inject IRegionRepository
        // ASP.NET Core DI sẽ tự động cung cấp một thể hiện của IRegionRepository (là RegionRepository)
        public RegionsController(IRegionRepository regionRepository)
        {
            _regionRepository = regionRepository;
        }

        [HttpGet]
        public IActionResult GetAllRegions()
        {
            // Sử dụng Repository để lấy tất cả các vùng từ cơ sở dữ liệu
            // Controller không cần biết chi tiết về cách dữ liệu được truy vấn
            var regions = _regionRepository.GetAll();

            // Trả về phản hồi HTTP 200 OK cùng với danh sách các vùng
            return Ok(regions);
        }
    }
}
```

## 5. Kiểm thử API với Dữ liệu từ Cơ sở dữ liệu

Bây giờ, khi bạn chạy ứng dụng và truy cập lại API `GET /api/Regions` thông qua Swagger UI, Postman hoặc trình duyệt, bạn sẽ thấy rằng dữ liệu trả về không còn là dữ liệu tĩnh nữa. Thay vào đó, API sẽ truy vấn cơ sở dữ liệu thông qua Repository và trả về danh sách các vùng mà bạn đã seed vào cơ sở dữ liệu trước đó.

Điều này xác nhận rằng:

*   Repository Pattern đã được triển khai thành công, tách biệt logic truy cập dữ liệu.
*   Dependency Injection đã hoạt động đúng cách, cung cấp `RegionRepository` cho `RegionsController`.
*   API của bạn hiện đang đọc dữ liệu trực tiếp từ cơ sở dữ liệu, chứng minh sự chuyển đổi từ mock data sang dữ liệu thực tế.

> [!TIP]
> **Antigravity IDE và Kiểm thử Dữ liệu Thực:**
> Sau khi bạn đã cập nhật `RegionsController` để sử dụng Repository, Antigravity có thể tự động chạy lại các kiểm thử API trước đó. Nó sẽ nhận ra rằng phản hồi JSON đã thay đổi từ dữ liệu tĩnh sang dữ liệu từ cơ sở dữ liệu. Bạn có thể "vibe" cho Antigravity: "Xác nhận rằng endpoint `/api/regions` hiện đang trả về dữ liệu từ cơ sở dữ liệu, không phải dữ liệu tĩnh." Antigravity có thể thực thi yêu cầu, phân tích cấu trúc và nội dung JSON, và thậm chí so sánh với dữ liệu dự kiến hoặc các bản ghi trong cơ sở dữ liệu (nếu có quyền truy cập), cung cấp phản hồi ngay lập tức về tính đúng đắn của API.

## Tóm tắt Phần 20

Trong phần này, chúng ta đã đặt nền móng vững chắc cho API Vùng (Regions) của dự án NZWalks, tích hợp thành công với cơ sở dữ liệu và áp dụng các nguyên tắc thiết kế quan trọng:

*   **Thiết lập `RegionsController`**: Tạo điểm đầu vào cho các yêu cầu HTTP liên quan đến vùng, cấu hình định tuyến và các thuộc tính API.
*   **Phương thức `GetAllRegions`**: Ban đầu được triển khai với dữ liệu tĩnh để kiểm thử nhanh, sau đó được nâng cấp để truy vấn dữ liệu thực.
*   **Kiểm thử API cơ bản**: Sử dụng Swagger UI và Postman để xác nhận chức năng của API.
*   **Hiểu sâu về Repository Pattern**: Nắm vững tầm quan trọng của việc tách biệt logic truy cập dữ liệu khỏi Controller, giải quyết các vấn đề về phụ thuộc, kiểm thử và nguyên tắc trách nhiệm đơn nhất.
*   **Triển khai Repository cho Regions**: Tạo `IRegionRepository` và `RegionRepository` để trừu tượng hóa việc truy cập cơ sở dữ liệu.
*   **Cấu hình Dependency Injection**: Đăng ký Repository với ASP.NET Core DI bằng `builder.Services.AddScoped()`, đảm bảo các thành phần nhận được các thể hiện dịch vụ phù hợp.
*   **Cập nhật Controller**: Thay thế logic dữ liệu tĩnh bằng việc gọi phương thức từ Repository đã được inject.
*   **Xác nhận cuối cùng**: Kiểm thử API để đảm bảo nó trả về dữ liệu thực tế từ cơ sở dữ liệu.

Phần tiếp theo sẽ tiếp tục phát triển API của chúng ta bằng cách triển khai các thao tác tạo mới và cập nhật dữ liệu vùng, đồng thời giới thiệu khái niệm DTO (Data Transfer Object) để tối ưu hóa việc truyền tải dữ liệu.

<!-- REVIEWED_BY_AGENT -->
