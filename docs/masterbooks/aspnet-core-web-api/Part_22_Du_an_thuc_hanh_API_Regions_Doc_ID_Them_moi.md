# Chương 22: Dự án thực hành API Vùng (Regions) – Nâng cao: Đọc theo ID và Thêm mới

Trong hành trình xây dựng RESTful Web API với ASP.NET Core và Entity Framework Core, chúng ta đã thiết lập nền tảng và triển khai phương thức lấy tất cả các vùng. Chương này sẽ đưa ứng dụng của chúng ta lên một tầm cao mới bằng cách bổ sung hai chức năng cốt lõi của một API quản lý tài nguyên: đọc thông tin chi tiết của một vùng dựa trên định danh (ID) và khả năng thêm một vùng mới vào hệ thống.

Chúng ta sẽ tập trung sâu vào việc áp dụng các nguyên tắc thiết kế phần mềm hiện đại như Dependency Injection (DI) và Repository Pattern. Điều này không chỉ giúp mã nguồn trở nên sạch sẽ, dễ bảo trì và kiểm thử mà còn thúc đẩy tính module hóa cao. Đồng thời, việc sử dụng chính xác các HTTP Verbs (GET, POST) và quản lý mã trạng thái HTTP sẽ được nhấn mạnh, đảm bảo API của chúng ta tuân thủ nghiêm ngặt các chuẩn mực của kiến trúc RESTful.

## I. Nền tảng RESTful API: Nguyên tắc và Phản hồi

Trước khi đi sâu vào mã nguồn, việc củng cố lại các khái niệm cơ bản về RESTful API và các hoạt động CRUD (Create, Read, Update, Delete) là rất quan trọng để đảm bảo chúng ta có một cái nhìn tổng thể và đúng đắn về cách thức xây dựng API hiệu quả.

### 1. Kiến trúc RESTful API: Hơn cả một giao thức

REST (Representational State Transfer) không phải là một giao thức mà là một *phong cách kiến trúc* cho các hệ thống phân tán, đặc biệt là Web API. Một API được coi là RESTful khi nó tuân thủ các nguyên tắc thiết kế sau:

*   **Sử dụng tài nguyên (Resources):** Mọi thứ trong API đều được coi là một tài nguyên. Mỗi tài nguyên có một URI (Uniform Resource Identifier) duy nhất để định danh.
    *   **Giải thích sâu sắc:** Tài nguyên là một khái niệm trừu tượng. Ví dụ, `/api/regions` đại diện cho tập hợp tất cả các tài nguyên vùng, trong khi `/api/regions/{id}` đại diện cho một tài nguyên vùng cụ thể. URI phải mô tả tài nguyên, không mô tả hành động. Việc này giúp client có thể dễ dàng đoán được các URI và tăng tính khám phá của API.
*   **Sử dụng các phương thức HTTP (HTTP Verbs):** Các hành động trên tài nguyên được thực hiện thông qua các phương thức HTTP tiêu chuẩn, mỗi phương thức mang một ngữ nghĩa cụ thể:
    *   **GET:** Lấy dữ liệu từ máy chủ. Đây là phương thức an toàn (safe) và bất biến (idempotent), nghĩa là nó không gây ra thay đổi trạng thái trên server và việc gọi nhiều lần sẽ trả về cùng một kết quả.
    *   **POST:** Gửi dữ liệu mới để tạo tài nguyên trên máy chủ. Phương thức này không an toàn và không bất biến, mỗi lần gọi có thể tạo ra một tài nguyên mới.
    *   **PUT:** Cập nhật toàn bộ tài nguyên hiện có. Phương thức này không an toàn nhưng bất biến (gửi cùng một yêu cầu nhiều lần sẽ đưa tài nguyên về cùng một trạng thái).
    *   **PATCH:** Cập nhật một phần tài nguyên hiện có. Tương tự PUT, nhưng chỉ áp dụng các thay đổi một phần.
    *   **DELETE:** Xóa tài nguyên. Phương thức này không an toàn và bất biến (sau lần xóa đầu tiên, các lần gọi tiếp theo sẽ vẫn cố gắng xóa tài nguyên không tồn tại, nhưng trạng thái cuối cùng vẫn là "đã xóa").
*   **Stateless (Không trạng thái):** Mỗi yêu cầu từ client đến server phải chứa tất cả thông tin cần thiết để server hiểu và xử lý yêu cầu đó. Server không lưu trữ bất kỳ trạng thái nào của client giữa các yêu cầu.
    *   **Giải thích sâu sắc:** Điều này có nghĩa là mỗi yêu cầu là độc lập. Server không dựa vào "session" hay "context" từ các yêu cầu trước đó. Lợi ích là API dễ dàng mở rộng theo chiều ngang (horizontal scaling) vì bất kỳ server nào cũng có thể xử lý bất kỳ yêu cầu nào. Các thông tin như xác thực (Authentication) hoặc ủy quyền (Authorization) thường được truyền qua header (ví dụ: `Authorization: Bearer <token>`).
*   **Client-Server Architecture:** Tách biệt rõ ràng trách nhiệm giữa client (giao diện người dùng, logic trình bày) và server (logic nghiệp vụ, quản lý dữ liệu).
    *   **Lợi ích:** Cho phép phát triển độc lập client và server, tăng tính linh hoạt và khả năng tái sử dụng.
*   **Layered System (Hệ thống phân lớp):** Hệ thống có thể được tổ chức thành các lớp (ví dụ: lớp trình bày, lớp nghiệp vụ, lớp truy cập dữ liệu) mà không ảnh hưởng đến client. Client chỉ cần tương tác với lớp ngoài cùng.

### 2. Mã trạng thái HTTP: Ngôn ngữ của API

Mã trạng thái HTTP là một phần không thể thiếu trong phản hồi từ server, đóng vai trò như một "ngôn ngữ" cho client biết kết quả của yêu cầu. Việc sử dụng đúng mã trạng thái là cực kỳ quan trọng để xây dựng một API dễ hiểu và dễ sử dụng. Mã trạng thái được chia thành 5 loại chính:

*   **1xx (Informational):** Yêu cầu đã được nhận và quá trình đang tiếp tục. (Ít gặp trong API thông thường).
*   **2xx (Success):** Yêu cầu đã được nhận, hiểu và chấp nhận thành công.
    *   **200 OK:** Yêu cầu thành công. Đây là mã mặc định cho các yêu cầu GET, PUT, PATCH hoặc DELETE thành công.
    *   **201 Created:** Tài nguyên mới đã được tạo thành công. Luôn được sử dụng cho các yêu cầu POST tạo tài nguyên mới. Phản hồi này thường bao gồm đối tượng tài nguyên mới và một header `Location` trỏ đến URI của tài nguyên đó.
    *   **204 No Content:** Yêu cầu thành công nhưng không có nội dung nào để trả về trong phần thân phản hồi (thường dùng cho PUT hoặc DELETE khi client không cần biết trạng thái của tài nguyên đã thay đổi).
*   **3xx (Redirection):** Cần thực hiện hành động bổ sung để hoàn tất yêu cầu. (Ví dụ: 301 Moved Permanently).
*   **4xx (Client Error):** Yêu cầu chứa cú pháp không chính xác hoặc không thể được thực hiện.
    *   **400 Bad Request:** Yêu cầu không hợp lệ. Thường xảy ra khi dữ liệu đầu vào không đúng định dạng, thiếu trường bắt buộc hoặc không vượt qua được các quy tắc nghiệp vụ.
    *   **401 Unauthorized:** Client chưa được xác thực (chưa đăng nhập).
    *   **403 Forbidden:** Client đã được xác thực nhưng không có quyền truy cập vào tài nguyên.
    *   **404 Not Found:** Tài nguyên được yêu cầu không tồn tại trên server.
*   **5xx (Server Error):** Server không thể thực hiện yêu cầu hợp lệ do một lỗi nội bộ.
    *   **500 Internal Server Error:** Lỗi không mong muốn xảy ra trên server. Đây là mã lỗi chung khi server gặp sự cố không xác định.

> [!TIP]
> **Cơ chế ngầm của ASP.NET Core:** Các phương thức như `Ok()`, `NotFound()`, `CreatedAtAction()`, `BadRequest()` trong `ControllerBase` (mà `RegionsController` kế thừa) là các phương thức tiện ích giúp bạn dễ dàng trả về `IActionResult` với mã trạng thái HTTP và dữ liệu phù hợp, mà không cần phải tự thiết lập `HttpResponse.StatusCode` một cách thủ công.

## II. Kiến trúc ứng dụng hiện đại: Dependency Injection và Repository Pattern

Trong ASP.NET Core, Dependency Injection (DI) là một tính năng cốt lõi, thúc đẩy việc xây dựng các ứng dụng có tính mô đun cao, dễ bảo trì và kiểm thử. Kết hợp với Repository Pattern, chúng ta có thể tách biệt logic truy cập dữ liệu khỏi các lớp nghiệp vụ và controller, tạo ra một kiến trúc sạch sẽ và mạnh mẽ.

### 1. Dependency Injection (DI) trong ASP.NET Core: Nền tảng của sự linh hoạt

Dependency Injection là một kỹ thuật triển khai nguyên tắc Inversion of Control (IoC), trong đó một đối tượng không tự tạo ra các đối tượng mà nó phụ thuộc (dependencies). Thay vào đó, các đối tượng phụ thuộc này sẽ được "tiêm" (injected) vào nó từ bên ngoài, thường là thông qua constructor.

> [!TIP]
> **Lợi ích cốt lõi của DI:**
> *   **Giảm khớp nối (Loose Coupling):** Các lớp không cần biết cách tạo ra các phụ thuộc của chúng. Chúng chỉ quan tâm đến việc sử dụng các interface được cung cấp. Điều này giúp thay đổi một triển khai mà không ảnh hưởng đến các lớp sử dụng nó.
> *   **Tăng khả năng kiểm thử (Testability):** Đây là lợi ích lớn nhất. Trong quá trình kiểm thử đơn vị (unit testing), bạn có thể dễ dàng thay thế các phụ thuộc thực tế bằng các đối tượng giả (mock objects) hoặc đối tượng giả lập (stub objects) để cô lập logic cần kiểm thử.
> *   **Tăng khả năng mở rộng (Extensibility):** Dễ dàng thêm hoặc thay đổi triển khai cho một interface mà không cần sửa đổi các lớp đang sử dụng interface đó.
> *   **Quản lý vòng đời (Lifecycle Management):** ASP.NET Core IoC Container (Service Provider) quản lý vòng đời của các đối tượng (Transient, Scoped, Singleton), giảm thiểu lỗi và tối ưu hóa tài nguyên.

**Cơ chế ngầm của DI trong ASP.NET Core:**
ASP.NET Core sử dụng một IoC Container (còn gọi là Service Provider) được tích hợp sẵn. Khi bạn đăng ký các dịch vụ (service) trong phương thức `ConfigureServices` (hoặc `Program.cs` đối với .NET 6+), bạn đang hướng dẫn container cách tạo và cung cấp các đối tượng khi chúng được yêu cầu.

Ví dụ:
```csharp
// Trong Program.cs hoặc Startup.cs
builder.Services.AddScoped<IRegionRepository, RegionRepository>();
builder.Services.AddAutoMapper(typeof(Program).Assembly); // Đăng ký AutoMapper
```
Khi `RegionsController` được khởi tạo:
```csharp
public class RegionsController : ControllerBase
{
    private readonly IRegionRepository _regionRepository;
    private readonly IMapper _mapper;

    // IoC Container sẽ tự động tìm IRegionRepository và IMapper đã đăng ký
    // và tiêm các đối tượng RegionRepository và Mapper vào constructor này.
    public RegionsController(IRegionRepository regionRepository, IMapper mapper)
    {
        _regionRepository = regionRepository;
        _mapper = mapper;
    }
    // ...
}
```
Container sẽ kiểm tra các phụ thuộc trong constructor của `RegionsController`, tìm kiếm các đăng ký phù hợp, tạo ra các đối tượng `RegionRepository` và `Mapper` (hoặc tái sử dụng nếu vòng đời là Singleton/Scoped) và tiêm chúng vào `RegionsController`.

### 2. Repository Pattern: Trừu tượng hóa lớp truy cập dữ liệu

Repository Pattern là một mẫu thiết kế giúp trừu tượng hóa lớp truy cập dữ liệu. Thay vì các controller hoặc lớp nghiệp vụ trực tiếp tương tác với Entity Framework Core (hoặc bất kỳ ORM nào khác), chúng sẽ tương tác với một interface repository.

> [!NOTE]
> **Vai trò và lợi ích của Repository Pattern:**
> *   **Tách biệt mối quan tâm (Separation of Concerns):** Logic truy cập dữ liệu (cách thức lưu trữ và truy xuất) được tách biệt hoàn toàn khỏi logic nghiệp vụ (cách thức xử lý dữ liệu). Controller chỉ cần biết *cần dữ liệu gì*, không cần biết *dữ liệu được lấy như thế nào*.
> *   **Dễ dàng thay đổi công nghệ cơ sở dữ liệu:** Nếu trong tương lai bạn muốn chuyển từ SQL Server sang MongoDB hoặc một hệ thống lưu trữ khác, bạn chỉ cần thay đổi triển khai của repository (`RegionRepository`) mà không cần sửa đổi `RegionsController` hoặc các lớp nghiệp vụ khác.
> *   **Tăng khả năng kiểm thử:** Giống như DI, Repository Pattern giúp dễ dàng tạo các triển khai giả (mock implementations) của repository cho mục đích kiểm thử. Bạn có thể kiểm thử logic của controller mà không cần kết nối đến cơ sở dữ liệu thực.
> *   **Đảm bảo tính nhất quán:** Tất cả các thao tác truy cập dữ liệu cho một tài nguyên cụ thể (ví dụ: `Region`) được tập trung tại một nơi, giúp đảm bảo tính nhất quán trong cách thức truy xuất và lưu trữ.

Chúng ta sẽ có một interface `IRegionRepository` định nghĩa các hoạt động truy cập dữ liệu và một lớp `RegionRepository` để triển khai các hoạt động đó bằng Entity Framework Core. Điều này tạo ra một "hàng rào" giữa domain logic và data access logic.

## III. Triển khai chức năng Đọc Vùng theo ID (GET /api/regions/{id})

Để cho phép client lấy thông tin chi tiết của một vùng cụ thể, chúng ta sẽ triển khai một endpoint GET nhận ID của vùng làm tham số.

### 1. Cập nhật Domain Model và DTOs

Trước khi đi vào Repository, hãy đảm bảo chúng ta có các định nghĩa cơ bản cho Domain Model và Data Transfer Object (DTO) phản hồi.

**Domain Model:** `Region` là đại diện cho một đối tượng vùng trong tầng nghiệp vụ và ánh xạ trực tiếp tới bảng trong cơ sở dữ liệu.

```csharp
// Models/Domain/Region.cs
using System;

namespace NZWalks.API.Models.Domain
{
    public class Region
    {
        public Guid Id { get; set; }
        public string Code { get; set; }
        public string Name { get; set; }
        public string? RegionImageUrl { get; set; } // Có thể null
    }
}
```

**Response DTO:** `RegionDto` là đối tượng sẽ được gửi về cho client. Nó giúp che giấu cấu trúc nội bộ của Domain Model và chỉ cung cấp những thông tin cần thiết.

```csharp
// Models/DTOs/RegionDto.cs
using System;

namespace NZWalks.API.Models.DTOs
{
    public class RegionDto
    {
        public Guid Id { get; set; }
        public string Code { get; set; }
        public string Name { get; set; }
        public string? RegionImageUrl { get; set; }
    }
}
```

### 2. Định nghĩa và Triển khai Repository cho việc đọc theo ID

#### a. Interface `IRegionRepository`

Đầu tiên, chúng ta cần định nghĩa phương thức `GetRegionByIdAsync` trong interface `IRegionRepository`. Phương thức này sẽ nhận một `Guid` làm ID và trả về một đối tượng `Region` (domain model) nếu tìm thấy, hoặc `null` nếu không tìm thấy.

```csharp
// Repositories/Interfaces/IRegionRepository.cs
using NZWalks.API.Models.Domain; // Đảm bảo đúng namespace cho Region

namespace NZWalks.API.Repositories.Interfaces
{
    public interface IRegionRepository
    {
        Task<IEnumerable<Region>> GetAllAsync();
        Task<Region?> GetRegionByIdAsync(Guid id); // Phương thức mới để lấy vùng theo ID
    }
}
```

> [!NOTE]
> *   `Task<Region?>`: Kiểu trả về `Task` cho biết đây là một hoạt động bất đồng bộ. Dấu `?` sau `Region` là cú pháp C# 8.0 cho phép `Region` có thể là `null` (Nullable Reference Types), giúp biên dịch viên cảnh báo nếu bạn không xử lý trường hợp `null`.
> *   Việc sử dụng `async` và `await` là cần thiết cho các hoạt động I/O (Input/Output) như truy vấn cơ sở dữ liệu. Nó giúp ứng dụng không bị chặn (block) luồng chính trong khi chờ đợi kết quả từ DB, từ đó cải thiện khả năng phản hồi và hiệu suất tổng thể của server.

#### b. Triển khai `RegionRepository`

Tiếp theo, chúng ta sẽ triển khai phương thức `GetRegionByIdAsync` trong lớp `RegionRepository`. Tại đây, chúng ta sẽ sử dụng Entity Framework Core để truy vấn cơ sở dữ liệu.

```csharp
// Repositories/RegionRepository.cs
using Microsoft.EntityFrameworkCore; // Cần thiết cho FirstOrDefaultAsync
using NZWalks.API.Data; // Giả định NZWalksDbContext nằm trong namespace này
using NZWalks.API.Models.Domain;
using NZWalks.API.Repositories.Interfaces;

namespace NZWalks.API.Repositories
{
    public class RegionRepository : IRegionRepository
    {
        private readonly NZWalksDbContext _dbContext;

        public RegionRepository(NZWalksDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task<IEnumerable<Region>> GetAllAsync()
        {
            return await _dbContext.Regions.ToListAsync();
        }

        public async Task<Region?> GetRegionByIdAsync(Guid id)
        {
            // Sử dụng FirstOrDefaultAsync để tìm một vùng dựa trên ID
            // Nếu không tìm thấy bất kỳ vùng nào khớp, nó sẽ trả về null.
            // Điều này hiệu quả hơn so với việc gọi ToList() rồi lọc trong bộ nhớ.
            return await _dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id);
        }
    }
}
```

> [!TIP]
> **Cơ chế ngầm của `FirstOrDefaultAsync()`:**
> *   `FirstOrDefaultAsync()` là một phương thức mở rộng của Entity Framework Core. Khi được gọi, EF Core sẽ dịch biểu thức lambda (`x => x.Id == id`) thành một câu lệnh SQL `WHERE` tương ứng và thực thi nó trên cơ sở dữ liệu.
> *   Nó được tối ưu hóa để chỉ tìm kiếm và trả về *một* bản ghi đầu tiên thỏa mãn điều kiện, hoặc `null` nếu không tìm thấy.
> *   **So sánh với các phương thức khác:**
    *   `SingleOrDefaultAsync()`: Tương tự `FirstOrDefaultAsync` nhưng sẽ ném ngoại lệ nếu tìm thấy nhiều hơn một bản ghi thỏa mãn điều kiện. Chỉ nên dùng khi bạn *chắc chắn* rằng chỉ có một hoặc không có bản ghi nào khớp.
    *   `Find()`: Đây là một phương thức khác của `DbSet` chỉ dùng để tìm kiếm theo khóa chính (primary key). Nó sẽ tìm trong bộ nhớ cục bộ (tracking cache) trước, sau đó mới truy vấn DB nếu không tìm thấy. Thường nhanh hơn nếu đối tượng đã được tải. Tuy nhiên, `FirstOrDefaultAsync` linh hoạt hơn vì có thể dùng với bất kỳ điều kiện lọc nào.

### 3. Xây dựng Endpoint trong `RegionsController`

Sau khi Repository đã sẵn sàng, chúng ta sẽ tạo một action method trong `RegionsController` để xử lý các yêu cầu GET cho một vùng cụ thể.

```csharp
// Controllers/RegionsController.cs
using AutoMapper; // Cần thiết để ánh xạ Domain Model sang DTO
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTOs; // Cần thiết cho DTOs
using NZWalks.API.Repositories.Interfaces; // Cần thiết cho IRegionRepository

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")] // Định tuyến cơ bản cho controller
    [ApiController] // Đánh dấu đây là một API controller, cung cấp các tính năng tiện ích như Model Validation tự động
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository _regionRepository;
        private readonly IMapper _mapper; // Tiêm AutoMapper thông qua DI

        // Dependency Injection thông qua constructor
        public RegionsController(IRegionRepository regionRepository, IMapper mapper)
        {
            _regionRepository = regionRepository;
            _mapper = mapper;
        }

        // Endpoint hiện có: Lấy tất cả các vùng
        [HttpGet]
        public async Task<IActionResult> GetAllAsync()
        {
            var regionsDomain = await _regionRepository.GetAllAsync();
            var regionsDto = _mapper.Map<List<RegionDto>>(regionsDomain);
            return Ok(regionsDto); // Trả về 200 OK
        }

        // Endpoint mới: Lấy vùng theo ID
        [HttpGet("{id:guid}")] // Định tuyến với tham số ID kiểu GUID
        public async Task<IActionResult> GetRegionByIdAsync([FromRoute] Guid id)
        {
            // 1. Gọi Repository để lấy vùng từ cơ sở dữ liệu
            var regionDomain = await _regionRepository.GetRegionByIdAsync(id);

            // 2. Xử lý trường hợp không tìm thấy vùng
            if (regionDomain == null)
            {
                return NotFound(); // Trả về 404 Not Found
            }

            // 3. Ánh xạ Domain Model sang DTO phản hồi
            // AutoMapper sẽ tự động chuyển đổi các thuộc tính tương ứng
            var regionDto = _mapper.Map<RegionDto>(regionDomain);

            // 4. Trả về 200 OK với DTO của vùng tìm thấy
            return Ok(regionDto);
        }
    }
}
```

> [!NOTE]
> *   `[HttpGet("{id:guid}")]`: Đây là một thuộc tính định tuyến mạnh mẽ. `"{id:guid}"` không chỉ khai báo rằng action method này nhận một tham số `id` từ URL, mà còn thêm một ràng buộc (route constraint) rằng `id` phải là một GUID hợp lệ. Nếu client gửi một giá trị không phải GUID, yêu cầu sẽ bị từ chối ngay từ tầng định tuyến (routing) mà không cần đi vào logic của action method, và trả về `400 Bad Request`.
> *   `[FromRoute] Guid id`: Thuộc tính `[FromRoute]` chỉ định rõ ràng rằng giá trị của tham số `id` sẽ được lấy từ các biến trong URL route. Mặc định, ASP.NET Core có thể suy luận điều này, nhưng việc chỉ định rõ ràng giúp mã dễ đọc và duy trì hơn.
> *   `_mapper.Map<RegionDto>(regionDomain)`: AutoMapper là một thư viện phổ biến giúp tự động ánh xạ các thuộc tính giữa các đối tượng có cấu trúc tương tự (ví dụ: từ `Region` sang `RegionDto`). Để sử dụng AutoMapper, bạn cần cấu hình một `Profile` trong ứng dụng của mình để định nghĩa các ánh xạ.

### 4. Kiểm thử với Swagger UI

Sau khi triển khai, bạn có thể chạy ứng dụng và sử dụng Swagger UI để kiểm thử phương thức `GetRegionByIdAsync`.

1.  **Chạy ứng dụng:** Mở trình duyệt và truy cập `https://localhost:<port>/swagger`.
2.  **Tìm endpoint:** Bạn sẽ thấy một endpoint mới được liệt kê dưới mục `RegionsController`: `GET /api/Regions/{id}`.
3.  **Kiểm thử trường hợp hợp lệ (200 OK):**
    *   Sử dụng endpoint `GET /api/Regions` để lấy danh sách tất cả các vùng và sao chép một `Id` hợp lệ từ phản hồi.
    *   Dán `Id` đó vào trường `id` của endpoint `GET /api/Regions/{id}` và nhấn "Execute".
    *   **Kết quả mong đợi:** Bạn sẽ nhận được phản hồi với mã trạng thái `200 OK` và chi tiết của vùng đó trong phần thân phản hồi.
4.  **Kiểm thử trường hợp không tìm thấy (404 Not Found):**
    *   Nhập một `Id` có định dạng GUID hợp lệ nhưng không tồn tại trong cơ sở dữ liệu (ví dụ: `00000000-0000-0000-0000-000000000000`).
    *   Nhấn "Execute".
    *   **Kết quả mong đợi:** Bạn sẽ nhận được phản hồi với mã trạng thái `404 Not Found` và phần thân phản hồi trống.
5.  **Kiểm thử trường hợp ID không hợp lệ (400 Bad Request):**
    *   Nhập một giá trị không phải GUID (ví dụ: `123` hoặc `abc`).
    *   **Kết quả mong đợi:** Swagger sẽ hiển thị lỗi định dạng ngay lập tức do ràng buộc `:guid` trong route. Nếu ràng buộc này không tồn tại, API sẽ trả về `400 Bad Request` do lỗi chuyển đổi kiểu dữ liệu.

## IV. Triển khai chức năng Thêm Vùng mới (POST /api/regions)

Để cho phép client thêm một vùng mới vào hệ thống, chúng ta sẽ triển khai một endpoint POST.

### 1. Thiết kế DTO cho yêu cầu thêm mới: `AddRegionRequestDto`

Khi client muốn tạo một tài nguyên mới, họ không nên và không cần cung cấp ID, vì ID thường được tạo ra bởi server hoặc cơ sở dữ liệu. Do đó, chúng ta cần một DTO riêng biệt cho yêu cầu thêm mới, không bao gồm trường ID.

```csharp
// Models/DTOs/AddRegionRequestDto.cs
using System.ComponentModel.DataAnnotations; // Để thêm các thuộc tính validation (tùy chọn)

namespace NZWalks.API.Models.DTOs
{
    public class AddRegionRequestDto
    {
        [Required(ErrorMessage = "Mã vùng là bắt buộc.")] // Ví dụ về Validation
        [MinLength(3, ErrorMessage = "Mã vùng phải có ít nhất 3 ký tự.")]
        [MaxLength(3, ErrorMessage = "Mã vùng chỉ có thể có tối đa 3 ký tự.")]
        public string Code { get; set; }

        [Required(ErrorMessage = "Tên vùng là bắt buộc.")]
        [MaxLength(100, ErrorMessage = "Tên vùng không được vượt quá 100 ký tự.")]
        public string Name { get; set; }

        public string? RegionImageUrl { get; set; }
    }
}
```

> [!NOTE]
> *   DTO này chỉ chứa các thông tin mà client cần gửi để tạo một vùng mới. Trường `Id` được loại bỏ để đảm bảo rằng ID sẽ được tạo tự động bởi server, tránh xung đột hoặc các ID không mong muốn do client cung cấp.
> *   Các thuộc tính `[Required]`, `[MinLength]`, `[MaxLength]` là các Data Annotation được ASP.NET Core tự động xử lý (Model Validation). Nếu dữ liệu gửi lên không thỏa mãn các điều kiện này, `ApiController` sẽ tự động trả về `400 Bad Request` mà không cần bạn viết logic kiểm tra thủ công trong action method. Điều này giúp giữ cho controller của bạn gọn gàng và tập trung vào logic nghiệp vụ.

### 2. Định nghĩa và Triển khai Repository cho việc thêm mới

#### a. Interface `IRegionRepository`

Thêm phương thức `AddRegionAsync` vào `IRegionRepository`. Phương thức này sẽ nhận một đối tượng `Region` (domain model) và trả về đối tượng `Region` đã được thêm vào cơ sở dữ liệu (lúc này đã có ID).

```csharp
// Repositories/Interfaces/IRegionRepository.cs
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories.Interfaces
{
    public interface IRegionRepository
    {
        Task<IEnumerable<Region>> GetAllAsync();
        Task<Region?> GetRegionByIdAsync(Guid id);
        Task<Region> AddRegionAsync(Region region); // Phương thức mới để thêm vùng
    }
}
```

#### b. Triển khai `RegionRepository`

Trong lớp `RegionRepository`, chúng ta sẽ triển khai logic để thêm một vùng mới vào cơ sở dữ liệu.

```csharp
// Repositories/RegionRepository.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;
using NZWalks.API.Repositories.Interfaces;

namespace NZWalks.API.Repositories
{
    public class RegionRepository : IRegionRepository
    {
        private readonly NZWalksDbContext _dbContext;

        public RegionRepository(NZWalksDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task<IEnumerable<Region>> GetAllAsync()
        {
            return await _dbContext.Regions.ToListAsync();
        }

        public async Task<Region?> GetRegionByIdAsync(Guid id)
        {
            return await _dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id);
        }

        public async Task<Region> AddRegionAsync(Region region)
        {
            // Gán một ID mới (GUID) cho vùng trước khi thêm vào DB.
            // Điều này đảm bảo ID được tạo bởi server, không phải client,
            // và là duy nhất.
            region.Id = Guid.NewGuid();

            // Thêm vùng vào ngữ cảnh của Entity Framework.
            // Tại bước này, đối tượng Region được đánh dấu là "Added".
            await _dbContext.Regions.AddAsync(region);

            // Lưu các thay đổi đã được đánh dấu vào cơ sở dữ liệu.
            // Đây là lúc câu lệnh INSERT INTO được thực thi.
            await _dbContext.SaveChangesAsync();

            // Trả về đối tượng vùng đã được thêm (lúc này đã có ID thực tế từ DB)
            return region;
        }
    }
}
```

> [!TIP]
> **Cơ chế ngầm của Entity Framework Core khi thêm mới:**
> *   `region.Id = Guid.NewGuid();`: Việc tạo GUID trên server là một thực hành tốt. Nó đảm bảo tính duy nhất và ngăn chặn client gửi ID trùng lặp hoặc ID không mong muốn. Nếu bạn sử dụng Identity Column (auto-increment) trong SQL Server, bạn sẽ không cần gán ID thủ công, EF Core sẽ tự động lấy ID sau `SaveChanges`. Với GUID, việc gán trước là cần thiết.
> *   `_dbContext.Regions.AddAsync(region)`: Phương thức này không ngay lập tức ghi dữ liệu vào cơ sở dữ liệu. Thay vào đó, nó "đánh dấu" đối tượng `region` trong ngữ cảnh thay đổi của Entity Framework (Change Tracker) với trạng thái `Added`.
> *   `_dbContext.SaveChangesAsync()`: Đây là lệnh thực sự gửi các thay đổi đã được đánh dấu (bao gồm các đối tượng `Added`, `Modified`, `Deleted`) đến cơ sở dữ liệu dưới dạng các câu lệnh SQL tương ứng (INSERT, UPDATE, DELETE). Nếu có nhiều thay đổi được đánh dấu, `SaveChangesAsync` sẽ cố gắng nhóm chúng lại thành một giao dịch duy nhất để đảm bảo tính toàn vẹn dữ liệu.

### 3. Xây dựng Endpoint trong `RegionsController`

Cuối cùng, chúng ta sẽ thêm một action method vào `RegionsController` để xử lý các yêu cầu POST để thêm vùng mới.

```csharp
// Controllers/RegionsController.cs
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTOs;
using NZWalks.API.Repositories.Interfaces;

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository _regionRepository;
        private readonly IMapper _mapper;

        public RegionsController(IRegionRepository regionRepository, IMapper mapper)
        {
            _regionRepository = regionRepository;
            _mapper = mapper;
        }

        [HttpGet]
        public async Task<IActionResult> GetAllAsync()
        {
            var regionsDomain = await _regionRepository.GetAllAsync();
            var regionsDto = _mapper.Map<List<RegionDto>>(regionsDomain);
            return Ok(regionsDto);
        }

        [HttpGet("{id:guid}")]
        public async Task<IActionResult> GetRegionByIdAsync([FromRoute] Guid id)
        {
            var regionDomain = await _regionRepository.GetRegionByIdAsync(id);
            if (regionDomain == null)
            {
                return NotFound();
            }
            var regionDto = _mapper.Map<RegionDto>(regionDomain);
            return Ok(regionDto);
        }

        // Phương thức mới: Thêm vùng mới
        [HttpPost] // Sử dụng HTTP POST để tạo tài nguyên mới
        public async Task<IActionResult> AddRegionAsync([FromBody] AddRegionRequestDto addRegionRequestDto)
        {
            // Tự động kiểm tra Model Validation nhờ [ApiController]
            // Nếu ModelState không hợp lệ, ASP.NET Core sẽ tự động trả về BadRequest (400)
            // với chi tiết lỗi mà không cần if (!ModelState.IsValid)

            // 1. Ánh xạ DTO yêu cầu sang Domain Model
            // AutoMapper giúp chuyển đổi AddRegionRequestDto sang Region (domain model)
            var regionDomainModel = _mapper.Map<Region>(addRegionRequestDto);

            // 2. Gọi Repository để thêm vùng vào cơ sở dữ liệu
            regionDomainModel = await _regionRepository.AddRegionAsync(regionDomainModel);

            // 3. Ánh xạ Domain Model đã thêm (lúc này đã có ID được tạo) sang DTO phản hồi
            var regionResponseDto = _mapper.Map<RegionDto>(regionDomainModel);

            // 4. Trả về 201 Created với Location header
            // CreatedAtAction tạo một phản hồi HTTP 201 (Created)
            // Nó cũng thêm một header 'Location' trỏ đến URL của tài nguyên mới được tạo
            return CreatedAtAction(
                nameof(GetRegionByIdAsync), // Tên action method để lấy tài nguyên này
                new { id = regionResponseDto.Id }, // Tham số route cho action method GetRegionByIdAsync
                regionResponseDto); // Đối tượng DTO của tài nguyên mới được tạo
        }
    }
}
```

> [!NOTE]
> *   `[FromBody] AddRegionRequestDto addRegionRequestDto`: Thuộc tính `[FromBody]` chỉ định rằng dữ liệu cho đối tượng `addRegionRequestDto` sẽ được lấy từ phần thân của yêu cầu HTTP (thường là JSON). ASP.NET Core tự động deserialized (chuyển đổi từ JSON sang đối tượng C#) dữ liệu này.
> *   `CreatedAtAction()`: Đây là một phương thức tiện ích quan trọng của `ControllerBase` khi tạo tài nguyên mới. Nó thực hiện hai việc chính:
    1.  Trả về mã trạng thái HTTP `201 Created`, báo hiệu thành công việc tạo tài nguyên mới.
    2.  Thêm một header `Location` vào phản hồi HTTP. Header này chứa URL đầy đủ của tài nguyên mới được tạo (ví dụ: `https://localhost:<port>/api/Regions/{new-guid}`). Điều này là một phần quan trọng của nguyên tắc HATEOAS (Hypermedia as the Engine of Application State) trong RESTful API, cho phép client dễ dàng truy cập tài nguyên đó ngay lập tức mà không cần phải tự xây dựng URL.
*   `nameof(GetRegionByIdAsync)`: Sử dụng `nameof()` thay vì chuỗi cứng `"GetRegionByIdAsync"` giúp tránh lỗi khi bạn đổi tên action method trong tương lai.

### 4. Kiểm thử với Swagger UI

Sau khi triển khai, hãy chạy ứng dụng và sử dụng Swagger UI để kiểm thử phương thức `AddRegionAsync`.

1.  **Chạy ứng dụng:** Mở trình duyệt và truy cập `https://localhost:<port>/swagger`.
2.  **Tìm endpoint:** Bạn sẽ thấy một endpoint mới: `POST /api/Regions`.
3.  **Thử nghiệm thành công (201 Created):**
    *   Mở rộng endpoint `POST /api/Regions` và nhấp vào "Try it out".
    *   Trong phần "Request body", cung cấp dữ liệu cho vùng mới (không bao gồm `id`):
        ```json
        {
          "code": "WLG",
          "name": "Wellington",
          "regionImageUrl": "https://example.com/wellington.jpg"
        }
        ```
    *   Nhấn "Execute".
    *   **Kết quả mong đợi:** Bạn sẽ nhận được phản hồi với mã trạng thái `201 Created`. Phản hồi cũng sẽ bao gồm đối tượng vùng đã tạo (với ID được tạo tự động) và một header `Location` trỏ đến URL của vùng mới này (ví dụ: `https://localhost:<port>/api/Regions/{new-guid}`).
4.  **Thử nghiệm thất bại (400 Bad Request) do Validation:**
    *   Thử gửi một request body thiếu trường `Code` hoặc `Name`, hoặc `Code` không đúng độ dài (ví dụ: `{"name": "Invalid Region"}`).
    *   **Kết quả mong đợi:** Bạn sẽ nhận được phản hồi với mã trạng thái `400 Bad Request` và một chi tiết lỗi mô tả các trường nào không hợp lệ.

> [!TIP]
> Header `Location` rất hữu ích cho client. Nó cho phép client ngay lập tức biết và truy cập tài nguyên mới được tạo mà không cần phải xây dựng URL theo cách thủ công. Điều này làm cho API trở nên tự mô tả và dễ sử dụng hơn.

## V. Tổng kết chương

Trong chương này, chúng ta đã mở rộng đáng kể khả năng của API Vùng bằng cách triển khai hai hoạt động quan trọng: đọc vùng theo ID và thêm vùng mới.

*   Chúng ta đã cập nhật `IRegionRepository` và `RegionRepository` để định nghĩa và triển khai các phương thức `GetRegionByIdAsync` và `AddRegionAsync`, tận dụng sức mạnh của Entity Framework Core cho việc truy cập dữ liệu bất đồng bộ hiệu quả.
*   Trong `RegionsController`, chúng ta đã tạo các action method tương ứng, sử dụng `[HttpGet("{id:guid}")]` cho việc đọc theo ID và `[HttpPost]` cho việc thêm mới. Các thuộc tính định tuyến và ràng buộc đã được giải thích chi tiết, cùng với việc sử dụng `[FromRoute]` và `[FromBody]` để binding dữ liệu.
*   Việc sử dụng Data Transfer Objects (DTOs) như `AddRegionRequestDto` và `RegionDto` đã được nhấn mạnh để tách biệt model domain khỏi cấu trúc dữ liệu gửi/nhận qua API, đồng thời sử dụng AutoMapper để đơn giản hóa quá trình ánh xạ giữa các đối tượng.
*   Chúng ta đã học cách xử lý các tình huống khác nhau bằng cách trả về các mã trạng thái HTTP phù hợp: `200 OK` cho yêu cầu thành công, `404 Not Found` khi tài nguyên không tồn tại, và `201 Created` kèm theo header `Location` khi tài nguyên mới được tạo, tuân thủ chặt chẽ nguyên tắc RESTful.
*   Cuối cùng, chúng ta đã sử dụng Swagger UI để kiểm thử chi tiết từng phương thức, đảm bảo chúng hoạt động đúng như mong đợi trong các kịch bản khác nhau, bao gồm cả việc kiểm tra Model Validation tự động.

Các kiến thức về Dependency Injection, Repository Pattern, Controllers, HTTP Verbs và DTOs đã được áp dụng một cách nhất quán, giúp xây dựng một API mạnh mẽ, dễ bảo trì và mở rộng. Trong các chương tiếp theo, chúng ta sẽ tiếp tục hoàn thiện API Vùng bằng cách triển khai các chức năng cập nhật và xóa, đồng thời đi sâu vào các khía cạnh nâng cao hơn của ASP.NET Core API.

<!-- REVIEWED_BY_AGENT -->
