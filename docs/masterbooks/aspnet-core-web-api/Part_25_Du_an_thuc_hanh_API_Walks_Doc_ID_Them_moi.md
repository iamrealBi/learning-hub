# Phần 25: Xây dựng RESTful API cho Hành trình (Walks) - Đọc Chi tiết và Thêm mới

Trong hành trình xây dựng API với ASP.NET Core, việc quản lý tài nguyên là cốt lõi. Phần này sẽ tập trung vào hai thao tác cơ bản nhưng cực kỳ quan trọng: đọc thông tin chi tiết của một tài nguyên duy nhất (dựa trên ID) và thêm mới một tài nguyên vào hệ thống. Chúng ta sẽ áp dụng triệt để các nguyên tắc thiết kế RESTful, kết hợp với Dependency Injection, Repository Pattern, và sức mạnh của Entity Framework Core để tạo ra các endpoint mạnh mẽ, dễ bảo trì và mở rộng.

> [!NOTE]
> Mục tiêu chính là củng cố kiến thức về cách thiết kế và triển khai các hoạt động `GET by ID` và `POST (Create)` cho tài nguyên `Walk` (Hành trình), đồng thời đào sâu vào các cơ chế hoạt động, lợi ích của các kiến trúc và công nghệ được sử dụng.

## 1. Đọc Thông Tin Một Hành Trình Cụ Thể (GET by ID)

Chức năng `GET by ID` là một trong những yêu cầu phổ biến nhất trong các API RESTful, cho phép client truy xuất toàn bộ thông tin chi tiết của một tài nguyên duy nhất thông qua định danh duy nhất của nó – trong trường hợp này là `WalkId`.

### 1.1. Thiết kế và Cập nhật Repository cho GET by ID

Để duy trì tính tách biệt giữa logic nghiệp vụ và logic truy cập dữ liệu, chúng ta sẽ thực hiện thao tác này thông qua Repository Pattern.

#### 1.1.1. Định nghĩa phương thức trong Interface `IWalkRepository`

Mọi thay đổi về khả năng của Repository cần được khai báo rõ ràng trong interface `IWalkRepository`. Điều này đảm bảo tính nhất quán, dễ kiểm thử và tuân thủ nguyên tắc "Liskov Substitution Principle" (LSP) trong SOLID.

```csharp
// IWalkRepository.cs
namespace NZWalks.API.Repositories
{
    public interface IWalkRepository
    {
        // Phương thức hiện có để lấy tất cả hành trình với các tùy chọn lọc, sắp xếp, phân trang.
        Task<IEnumerable<Walk>> GetAllAsync(
            string? filterOn = null, string? filterQuery = null,
            string? sortBy = null, bool isAscending = true,
            int pageNumber = 1, int pageSize = 1000);

        // [MỚI] Phương thức để lấy một hành trình cụ thể theo ID.
        // Trả về Walk? để chỉ ra rằng có thể không tìm thấy (null).
        Task<Walk?> GetByIdAsync(Guid id);

        // [SẼ THÊM SAU] Phương thức để thêm mới một hành trình.
        Task<Walk> CreateAsync(Walk walk);
        // ... các phương thức khác (Update, Delete)
    }
}
```

> [!AI Perspective: Antigravity IDE Insight]
> Khi người dùng "vibe" (ý định) muốn "lấy một đối tượng Walk theo ID", Antigravity IDE sẽ tự động phân tích ngữ cảnh, nhận diện `Walk` là một Domain Model, và đề xuất signature `Task<Walk?> GetByIdAsync(Guid id);` trong `IWalkRepository`. Nó hiểu rằng `Guid` là kiểu dữ liệu phổ biến cho ID và `Walk?` là cần thiết để xử lý trường hợp không tìm thấy.

#### 1.1.2. Triển khai phương thức `GetByIdAsync` trong `WalkRepository`

Logic truy vấn cơ sở dữ liệu thực tế sẽ nằm trong lớp triển khai `WalkRepository`. Đây là nơi Entity Framework Core được sử dụng để tương tác với database.

```csharp
// WalkRepository.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public class WalkRepository : IWalkRepository
    {
        private readonly NZWalksDbContext dbContext;

        public WalkRepository(NZWalksDbContext dbContext)
        {
            this.dbContext = dbContext;
        }

        public async Task<Walk?> GetByIdAsync(Guid id)
        {
            // Sử dụng FirstOrDefaultAsync để tìm một hành trình duy nhất dựa trên ID.
            // Nếu không tìm thấy, nó sẽ trả về null.
            // .Include() để tải các đối tượng liên quan (Area và Difficulty) cùng lúc.
            // Điều này là "eager loading" và giúp tránh vấn đề N+1 query.
            return await dbContext.Walks
                .Include(nameof(Difficulty)) // Bao gồm thông tin Difficulty
                .Include(nameof(Area))       // Bao gồm thông tin Area
                .FirstOrDefaultAsync(x => x.Id == id);
        }

        // ... các phương thức khác
    }
}
```

##### Giải thích cơ chế Eager Loading và vấn đề N+1

*   **`FirstOrDefaultAsync(x => x.Id == id)`**: Đây là phương thức truy vấn LINQ to Entities, được Entity Framework Core dịch sang câu lệnh SQL `SELECT TOP 1 ... WHERE Id = @id`. Nó hiệu quả hơn `SingleOrDefaultAsync` khi bạn chỉ cần *một* kết quả phù hợp đầu tiên, không cần kiểm tra tính duy nhất trên toàn bộ tập hợp.
*   **`Include(nameof(Difficulty)).Include(nameof(Area))`**: Đây là kỹ thuật **Eager Loading** (tải trước). Trong Domain Model `Walk`, chúng ta có các thuộc tính điều hướng (navigation properties) là `Difficulty` và `Area`. Mặc định, Entity Framework Core sẽ không tải các đối tượng liên quan này khi bạn truy vấn `Walk`. Nếu bạn cố gắng truy cập `walk.Difficulty.Name` mà không có `.Include()`, nó có thể gây ra lỗi (nếu `Difficulty` là `null`) hoặc tệ hơn, gây ra vấn đề **N+1 Query**.
    *   **Vấn đề N+1 Query**: Nếu bạn có một danh sách N `Walk` và muốn hiển thị `Difficulty` và `Area` cho mỗi `Walk`, Entity Framework Core (với lazy loading hoặc không có eager loading) sẽ thực hiện 1 truy vấn để lấy danh sách N `Walk`, sau đó N truy vấn riêng biệt cho `Difficulty` và N truy vấn riêng biệt cho `Area` (tổng cộng 1 + 2N truy vấn). Điều này cực kỳ kém hiệu quả.
    *   **Giải pháp Eager Loading**: Bằng cách sử dụng `.Include()`, Entity Framework Core sẽ tạo một câu lệnh SQL `JOIN` duy nhất để tải `Walk` cùng với `Difficulty` và `Area` liên quan chỉ trong một truy vấn duy nhất, giúp tối ưu hóa đáng kể hiệu suất.

> [!AI Perspective: Antigravity IDE Insight]
> Khi Antigravity IDE được yêu cầu triển khai `GetByIdAsync`, nó sẽ ngay lập tức nhận ra các mối quan hệ (navigation properties) của `Walk` (`Difficulty`, `Area`) từ `NZWalksDbContext` hoặc định nghĩa Domain Model. Dựa trên "vibe" của việc trả về "thông tin chi tiết", nó sẽ tự động thêm các `.Include()` cần thiết để tránh vấn đề N+1 query, thể hiện khả năng dự đoán và áp dụng best practices mà không cần hướng dẫn tường minh.

### 1.2. Xây dựng Endpoint GET theo ID trong Controller

Sau khi Repository đã sẵn sàng, chúng ta sẽ tạo một action method trong `WalksController` để tiếp nhận và xử lý các yêu cầu HTTP GET đến `/api/walks/{id}`.

#### 1.2.1. Cập nhật `WalksController`

```csharp
// WalksController.cs
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")] // Route cơ sở: /api/Walks
    [ApiController]             // Chỉ ra đây là một API Controller
    public class WalksController : ControllerBase
    {
        private readonly IMapper mapper;
        private readonly IWalkRepository walkRepository;

        // Dependency Injection: IMapper và IWalkRepository được inject vào constructor
        public WalksController(IMapper mapper, IWalkRepository walkRepository)
        {
            this.mapper = mapper;
            this.walkRepository = walkRepository;
        }

        // GET Single Walk By ID
        // HTTP GET request to: /api/Walks/{id}
        [HttpGet]
        [Route("{id:Guid}")] // Định nghĩa route cụ thể cho phương thức này, yêu cầu ID là kiểu Guid
        public async Task<IActionResult> GetWalkByIdAsync([FromRoute] Guid id) // Lấy ID từ route
        {
            // 1. Gọi Repository để lấy đối tượng Domain Model.
            // Repository trả về Walk? (có thể null).
            var walkDomainModel = await walkRepository.GetByIdAsync(id);

            // 2. Xử lý trường hợp không tìm thấy tài nguyên.
            if (walkDomainModel == null)
            {
                return NotFound(); // Trả về HTTP 404 Not Found
            }

            // 3. Chuyển đổi Domain Model sang DTO (Data Transfer Object).
            // DTO giúp định hình dữ liệu trả về cho client, ẩn đi các chi tiết nội bộ.
            var walkDto = mapper.Map<WalkDto>(walkDomainModel);

            // 4. Trả về HTTP 200 OK cùng với DTO.
            return Ok(walkDto);
        }

        // ... các phương thức khác
    }
}
```

#### 1.2.2. Phân tích Chi tiết Endpoint và Vai trò của DTO

*   **`[HttpGet]` và `[Route("{id:Guid}")]`**:
    *   `[HttpGet]`: Chỉ định rằng phương thức này sẽ xử lý các yêu cầu HTTP GET.
    *   `[Route("{id:Guid}")]`: Định nghĩa một phân đoạn trong URL là `id` và ràng buộc kiểu dữ liệu của nó phải là `Guid`. Điều này ngăn chặn các yêu cầu với ID không hợp lệ đến được action method. Route hoàn chỉnh sẽ là `/api/Walks/{id}`.
*   **`[FromRoute] Guid id`**: Thuộc tính `[FromRoute]` hướng dẫn ASP.NET Core Model Binder lấy giá trị của tham số `id` trực tiếp từ URL segment đã định nghĩa.
*   **`async Task<IActionResult>`**:
    *   `async/await`: Cho phép phương thức thực hiện các hoạt động I/O (như truy vấn cơ sở dữ liệu) một cách bất đồng bộ, giải phóng thread của server để xử lý các yêu cầu khác trong khi chờ đợi, cải thiện khả năng mở rộng của ứng dụng.
    *   `IActionResult`: Một interface linh hoạt trong ASP.NET Core, cho phép action method trả về nhiều loại phản hồi HTTP khác nhau (ví dụ: `Ok`, `NotFound`, `BadRequest`, `Created`).
*   **`walkRepository.GetByIdAsync(id)`**: Sử dụng Dependency Injection, `WalksController` không cần biết chi tiết về cách dữ liệu được truy vấn, nó chỉ gọi phương thức đã được trừu tượng hóa trong `IWalkRepository`. Đây là minh chứng cho lợi ích của Repository Pattern.
*   **`NotFound()`**: Phương thức này tạo ra một phản hồi HTTP `404 Not Found`. Đây là mã trạng thái chuẩn trong RESTful API để chỉ ra rằng tài nguyên mà client yêu cầu không tồn tại trên máy chủ.
*   **`mapper.Map<WalkDto>(walkDomainModel)`**:
    *   **Data Transfer Objects (DTOs)**: DTO là các đối tượng đơn giản được sử dụng để truyền dữ liệu giữa các lớp hoặc giữa các ứng dụng. Trong ngữ cảnh API, chúng đóng vai trò là "hợp đồng" (contract) giữa API và client.
    *   **Lợi ích của DTOs**:
        *   **Tùy chỉnh dữ liệu**: Chỉ trả về những trường dữ liệu mà client cần, tránh gửi thừa thông tin.
        *   **Bảo mật**: Ngăn chặn việc lộ các trường nhạy cảm hoặc không cần thiết của Domain Model ra bên ngoài.
        *   **Phiên bản API**: Dễ dàng thay đổi cấu trúc phản hồi mà không ảnh hưởng đến Domain Model hoặc logic nghiệp vụ cốt lõi.
        *   **Tránh các tham chiếu vòng**: Đặc biệt hữu ích khi Domain Model có các mối quan hệ phức tạp, có thể gây ra lỗi tuần hoàn khi serialize.
    *   AutoMapper (được inject thông qua `IMapper`) tự động ánh xạ các thuộc tính từ `Walk` Domain Model sang `WalkDto` dựa trên cấu hình đã thiết lập.
*   **`Ok(walkDto)`**: Nếu tìm thấy hành trình, chúng ta trả về phản hồi HTTP `200 OK` cùng với đối tượng `WalkDto` đã được chuyển đổi.

### 1.3. Kiểm thử Endpoint GET theo ID với Swagger

Swagger UI là một công cụ tuyệt vời để kiểm thử và tài liệu hóa API của bạn.

#### 1.3.1. Quy trình kiểm thử

1.  **Chạy ứng dụng:** Khởi động ứng dụng ASP.NET Core. Swagger UI sẽ tự động mở trong trình duyệt.
2.  **Mở rộng endpoint GET `/api/Walks/{id}`:** Trong Swagger UI, tìm mục `Walks` và mở rộng phần `GET /api/Walks/{id}`.
3.  **Lấy một Walk ID hợp lệ:**
    *   Để có ID, bạn có thể gọi endpoint `GET /api/Walks` (lấy tất cả hành trình) trước đó.
    *   Sao chép một `Id` từ bất kỳ hành trình nào trong kết quả.
4.  **Dán ID và thực thi:**
    *   Nhấp vào "Try it out".
    *   Dán `Id` đã sao chép vào trường `id` trong phần "Parameters".
    *   Nhấp vào nút "Execute".

#### 1.3.2. Phân tích kết quả phản hồi

*   **Phản hồi thành công (200 OK):**
    *   Bạn sẽ thấy HTTP Status Code là `200 OK`.
    *   Phần "Response body" sẽ chứa thông tin chi tiết của hành trình tương ứng với ID đã nhập, bao gồm cả thông tin `Area` và `Difficulty` liên quan (nhờ `.Include()` trong repository và ánh xạ của AutoMapper).
*   **Phản hồi không tìm thấy (404 Not Found):**
    *   Nếu bạn nhập một ID không tồn tại (ví dụ: một GUID ngẫu nhiên không có trong DB), bạn sẽ nhận được phản hồi `404 Not Found` và "Response body" thường sẽ trống hoặc chứa thông báo lỗi mặc định.

> [!AI Perspective: Antigravity IDE Insight]
> Antigravity IDE không chỉ dừng lại ở việc tạo mã. Sau khi phát triển `GetWalkByIdAsync`, nó sẽ tự động tạo các kịch bản kiểm thử tích hợp. Nó có thể sử dụng một "subagent trình duyệt" để tương tác với Swagger UI, hoặc tự động gửi các yêu cầu HTTP `GET` với ID hợp lệ và không hợp lệ, sau đó phân tích các mã trạng thái (`200 OK`, `404 Not Found`) và cấu trúc phản hồi để đảm bảo endpoint hoạt động đúng như mong đợi.

## 2. Thêm Mới Một Hành Trình (POST)

Chức năng `POST` cho phép client gửi dữ liệu đến máy chủ để tạo một tài nguyên mới. Đây là một thao tác thay đổi trạng thái quan trọng trong bất kỳ API nào.

### 2.1. Thiết kế DTO cho Yêu cầu Thêm Hành Trình (AddWalkRequestDto)

Khi client muốn tạo một hành trình mới, họ sẽ gửi dữ liệu cần thiết. Chúng ta cần một DTO riêng biệt cho yêu cầu đầu vào này, thường được gọi là Request DTO hoặc Create DTO, để kiểm soát chặt chẽ dữ liệu được phép nhận.

#### 2.1.1. Mục đích và cấu trúc

*   `AddWalkRequestDto` chỉ chứa các thông tin mà client cần cung cấp để tạo một hành trình mới.
*   Nó **không** bao gồm `Id`, vì ID sẽ được tạo tự động bởi máy chủ.
*   Các thuộc tính như `Area` và `Difficulty` (là các đối tượng navigation trong Domain Model) cũng không có mặt. Thay vào đó, chúng ta chỉ cần `AreaId` và `DifficultyId` để thiết lập mối quan hệ. Điều này giúp client dễ dàng gửi dữ liệu hơn.

```csharp
// AddWalkRequestDto.cs
namespace NZWalks.API.Models.DTO
{
    public class AddWalkRequestDto
    {
        // Các thuộc tính cần thiết để tạo một hành trình mới
        public string Name { get; set; }
        public string Description { get; set; } // Mô tả hành trình
        public double LengthInKm { get; set; }
        public string? WalkImageUrl { get; set; } // Đường dẫn hình ảnh, có thể null

        // ID của các tài nguyên liên quan mà client cần cung cấp
        public Guid DifficultyId { get; set; }
        public Guid AreaId { get; set; }
    }
}
```

### 2.2. Cập nhật Repository để Thêm Hành Trình

Logic thêm mới vào cơ sở dữ liệu sẽ được đóng gói trong Repository, tương tự như các thao tác khác.

#### 2.2.1. Định nghĩa phương thức trong Interface `IWalkRepository`

```csharp
// IWalkRepository.cs
namespace NZWalks.API.Repositories
{
    public interface IWalkRepository
    {
        // ... các phương thức khác
        // [MỚI] Phương thức để thêm một đối tượng Walk mới vào cơ sở dữ liệu.
        // Trả về đối tượng Walk đã được thêm (có ID sau khi lưu).
        Task<Walk> CreateAsync(Walk walk);
    }
}
```

#### 2.2.2. Triển khai phương thức `CreateAsync` trong `WalkRepository`

```csharp
// WalkRepository.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public class WalkRepository : IWalkRepository
    {
        private readonly NZWalksDbContext dbContext;

        public WalkRepository(NZWalksDbContext dbContext)
        {
            this.dbContext = dbContext;
        }

        public async Task<Walk> CreateAsync(Walk walk)
        {
            // [QUAN TRỌNG] Tự động gán một ID mới và duy nhất cho hành trình trước khi thêm vào DB.
            // Chúng ta không nên tin tưởng client cung cấp ID duy nhất.
            walk.Id = Guid.NewGuid();

            // Thêm đối tượng Walk vào DbSet. Tại đây, Entity Framework Core bắt đầu theo dõi đối tượng.
            // Dữ liệu *chưa* được lưu vào cơ sở dữ liệu.
            await dbContext.Walks.AddAsync(walk);

            // Lưu các thay đổi từ ngữ cảnh theo dõi vào cơ sở dữ liệu.
            // SaveChangesAsync() là nơi thực sự tương tác với DB để chèn dữ liệu.
            await dbContext.SaveChangesAsync();

            // Trả về đối tượng Walk đã được thêm (nay đã có ID được gán).
            return walk;
        }

        // ... các phương thức khác
    }
}
```

##### Cơ chế tạo ID và lưu trữ dữ liệu với Entity Framework Core

*   **`walk.Id = Guid.NewGuid()`**: Đây là một bước bảo mật và đảm bảo tính toàn vẹn dữ liệu quan trọng. Thay vì dựa vào client để cung cấp một ID (có thể trùng lặp hoặc không hợp lệ), chúng ta tự động tạo một `Guid` mới và duy nhất trên server cho mỗi hành trình mới. `Guid` (Globally Unique Identifier) là một chuỗi 128-bit được thiết kế để đảm bảo tính duy nhất trên toàn cầu mà không cần một cơ quan trung ương cấp phát.
*   **`await dbContext.Walks.AddAsync(walk)`**: Phương thức này thêm đối tượng `Walk` vào ngữ cảnh theo dõi (change tracker) của Entity Framework Core. EF Core sẽ đánh dấu đối tượng này là "Added" nhưng chưa thực hiện bất kỳ thay đổi nào với cơ sở dữ liệu vật lý.
*   **`await dbContext.SaveChangesAsync()`**: Đây là phương thức thực sự gửi các thay đổi (thêm, sửa, xóa) đã được theo dõi từ ngữ cảnh xuống cơ sở dữ liệu. EF Core sẽ tạo ra câu lệnh SQL `INSERT` tương ứng và thực thi nó. Sau khi `SaveChangesAsync` hoàn tất, đối tượng `walk` trong bộ nhớ sẽ được cập nhật với bất kỳ giá trị nào được tạo tự động bởi database (ví dụ: nếu bạn cấu hình ID tự tăng, nhưng ở đây chúng ta tự tạo `Guid`).

> [!AI Perspective: Antigravity IDE Insight]
> Đối với `CreateAsync`, Antigravity IDE sẽ nhận ra rằng đây là một thao tác tạo mới và cần một ID duy nhất. Nó sẽ tự động thêm `walk.Id = Guid.NewGuid();` như một best practice để đảm bảo tính duy nhất và không phụ thuộc vào client. Hơn nữa, nó hiểu quy trình của Entity Framework Core: `AddAsync` để theo dõi và `SaveChangesAsync` để lưu trữ, tự động tạo ra chuỗi lệnh này một cách chính xác.

### 2.3. Xây dựng Endpoint POST để Thêm Hành Trình

Bây giờ, chúng ta sẽ tạo một action method trong `WalksController` để xử lý các yêu cầu HTTP POST đến `/api/walks`.

#### 2.3.1. Cập nhật `WalksController`

```csharp
// WalksController.cs
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class WalksController : ControllerBase
    {
        private readonly IMapper mapper;
        private readonly IWalkRepository walkRepository;

        public WalksController(IMapper mapper, IWalkRepository walkRepository)
        {
            this.mapper = mapper;
            this.walkRepository = walkRepository;
        }

        // POST To Create New Walk
        // HTTP POST request to: /api/Walks
        [HttpPost] // Chỉ định rằng phương thức này xử lý yêu cầu HTTP POST
        public async Task<IActionResult> CreateAsync([FromBody] AddWalkRequestDto addWalkRequestDto)
        {
            // 1. Chuyển đổi DTO nhận được từ client sang Domain Model.
            // Repository chỉ làm việc với Domain Model.
            var walkDomainModel = mapper.Map<Walk>(addWalkRequestDto);

            // 2. Gọi Repository để thêm Walk vào cơ sở dữ liệu.
            // Repository sẽ trả về Domain Model đã được tạo (có ID).
            walkDomainModel = await walkRepository.CreateAsync(walkDomainModel);

            // 3. Chuyển đổi Domain Model đã được tạo trở lại DTO để trả về cho client.
            // Client cần đối tượng DTO với ID mới được gán.
            var walkDto = mapper.Map<WalkDto>(walkDomainModel);

            // 4. Trả về 201 Created cùng với DTO và Location Header.
            // CreatedAtAction là phản hồi tiêu chuẩn cho HTTP POST khi tạo tài nguyên thành công.
            // Nó trả về mã trạng thái 201 Created và một Location header trỏ đến URL của tài nguyên mới.
            return CreatedAtAction(nameof(GetWalkByIdAsync), new { id = walkDto.Id }, walkDto);
        }

        // ... các phương thức khác
    }
}
```

#### 2.3.2. Phân tích Chi tiết Endpoint và HTTP 201 Created

*   **`[HttpPost]`**: Thuộc tính này cho biết phương thức sẽ xử lý các yêu cầu HTTP POST. Vì không có `[Route]` cụ thể, nó sẽ sử dụng route mặc định của controller: `/api/Walks`.
*   **`[FromBody] AddWalkRequestDto addWalkRequestDto`**: Thuộc tính `[FromBody]` hướng dẫn ASP.NET Core Model Binder lấy dữ liệu từ phần thân (body) của yêu cầu HTTP và deserialize nó thành đối tượng `AddWalkRequestDto`. Điều này thường áp dụng cho dữ liệu JSON hoặc XML.
*   **`mapper.Map<Walk>(addWalkRequestDto)`**: Chuyển đổi `AddWalkRequestDto` nhận được từ client thành đối tượng `Walk` Domain Model. Đây là bước quan trọng để ánh xạ dữ liệu đầu vào không chứa ID thành đối tượng Domain Model đầy đủ để Repository có thể xử lý.
*   **`await walkRepository.CreateAsync(walkDomainModel)`**: Gọi phương thức `CreateAsync` trong repository để thêm hành trình vào cơ sở dữ liệu.
*   **`mapper.Map<WalkDto>(walkDomainModel)`**: Sau khi hành trình được tạo và có ID, chúng ta chuyển đổi lại đối tượng `Walk` Domain Model (đã có ID) thành `WalkDto` để trả về cho client. `WalkDto` là DTO phản hồi, có thể khác với `AddWalkRequestDto`.
*   **`CreatedAtAction(nameof(GetWalkByIdAsync), new { id = walkDto.Id }, walkDto)`**: Đây là một phản hồi chuẩn và được khuyến nghị cho yêu cầu POST thành công trong RESTful API.
    *   Nó trả về mã trạng thái HTTP **201 Created**. Mã này thể hiện chính xác rằng một tài nguyên mới đã được tạo thành công trên máy chủ.
    *   Nó bao gồm một **`Location` header** trong phản hồi, trỏ đến URL của tài nguyên mới được tạo (ví dụ: `https://localhost:PORT/api/Walks/GUID_CUA_WALK_MOI`). Điều này rất quan trọng vì nó cho phép client dễ dàng "khám phá" và truy cập tài nguyên vừa tạo mà không cần phải tự xây dựng URL.
    *   `nameof(GetWalkByIdAsync)`: Chỉ định tên của action method có thể được sử dụng để truy xuất tài nguyên vừa tạo.
    *   `new { id = walkDto.Id }`: Cung cấp các tham số route cần thiết cho action `GetWalkByIdAsync` (trong trường hợp này là `id`).
    *   `walkDto`: Đối tượng DTO của hành trình vừa tạo, được trả về trong phần thân của phản hồi.

> [!TIP]
> **Tại sao lại dùng `CreatedAtAction` thay vì `Ok` cho POST?**
>
> *   **Tính ngữ nghĩa RESTful:** Mã trạng thái `201 Created` thể hiện chính xác rằng một tài nguyên mới đã được tạo thành công trên máy chủ. `200 OK` thường dùng cho các yêu cầu `GET` hoặc `PUT` thành công khi không có tài nguyên mới nào được tạo.
> *   **Khả năng khám phá tài nguyên:** `Location` header là một phần quan trọng của phản hồi `201 Created`. Nó cung cấp cho client URL đầy đủ để truy cập tài nguyên mới, giúp client dễ dàng tương tác tiếp với tài nguyên đó mà không cần phải tự xây dựng URL.

### 2.4. Kiểm thử Endpoint POST với Swagger

#### 2.4.1. Chuẩn bị dữ liệu và thực thi

1.  **Chạy ứng dụng:** Khởi động ứng dụng ASP.NET Core. Swagger UI sẽ tự động mở.
2.  **Mở rộng endpoint POST `/api/Walks`:** Trong Swagger UI, tìm mục `Walks` và mở rộng phần `POST /api/Walks`.
3.  **Chuẩn bị dữ liệu yêu cầu:**
    *   Nhấp vào "Try it out".
    *   Bạn cần cung cấp một đối tượng JSON cho `AddWalkRequestDto` trong phần "Request body".
    *   **Lưu ý quan trọng:** Để tạo một hành trình, bạn cần cung cấp `AreaId` và `DifficultyId` hợp lệ.
        *   Bạn có thể lấy các `AreaId` bằng cách gọi `GET /api/Areas`.
        *   Bạn có thể lấy các `DifficultyId` bằng cách gọi `GET /api/Difficulties`.
    *   Ví dụ JSON mẫu:
        ```json
        {
          "name": "Pinnacles Walk",
          "description": "A challenging climb to the Pinnacles Hut with stunning views.",
          "lengthInKm": 6.5,
          "walkImageUrl": "https://example.com/pinnacles-walk.jpg",
          "difficultyId": "00000000-0000-0000-0000-000000000002", // Thay bằng ID của "Medium"
          "areaId": "00000000-0000-0000-0000-000000000001"    // Thay bằng ID của "Waikato"
        }
        ```
        (Hãy thay thế các `Guid` mẫu bằng `Guid` thực tế từ cơ sở dữ liệu của bạn).
4.  **Thực thi yêu cầu:** Nhấp vào nút "Execute".

#### 2.4.2. Phân tích kết quả và xác nhận

*   **Phản hồi thành công (201 Created):**
    *   Bạn sẽ thấy HTTP Status Code là `201 Created`.
    *   Phần "Response body" sẽ chứa đối tượng `WalkDto` của hành trình vừa được tạo, bao gồm cả `Id` mới được gán.
    *   Kiểm tra "Response headers", bạn sẽ thấy một `Location` header trỏ đến URL của tài nguyên mới (ví dụ: `https://localhost:PORT/api/Walks/GUID_CUA_WALK_MOI`).
*   **Xác nhận thêm:**
    *   Bạn có thể gọi lại `GET /api/Walks` để xem danh sách tất cả các hành trình và xác nhận rằng hành trình mới đã được thêm vào.
    *   Sử dụng URL từ `Location` header, bạn có thể gọi `GET /api/Walks/{id_moi_tao}` để lấy thông tin chi tiết của hành trình vừa tạo, bao gồm cả các thông tin `Area` và `Difficulty` đầy đủ (nhờ endpoint `GetWalkByIdAsync` của chúng ta đã có `.Include()`).

> [!AI Perspective: Antigravity IDE Insight]
> Đối với việc kiểm thử `POST`, Antigravity IDE sẽ không chỉ thực thi yêu cầu mà còn thông minh hơn. Nó sẽ:
> 1.  **Lập kế hoạch:** Trước khi gửi `POST`, nó sẽ sử dụng "subagent trình duyệt" hoặc script để gọi `GET /api/Areas` và `GET /api/Difficulties` để lấy các `Guid` hợp lệ.
> 2.  **Tạo Request Body:** Tự động điền một `AddWalkRequestDto` mẫu với các `Guid` hợp lệ đã lấy được.
> 3.  **Thực thi và Phân tích:** Gửi yêu cầu `POST`, kiểm tra `201 Created` và `Location` header.
> 4.  **Xác nhận:** Ngay lập tức sử dụng `Location` header để thực hiện một `GET` request đến tài nguyên vừa tạo, đảm bảo rằng dữ liệu đã được lưu trữ chính xác và có thể truy xuất được.
> 5.  **Dọn dẹp (tùy chọn):** Trong một môi trường kiểm thử, nó có thể tự động gửi yêu cầu `DELETE` để dọn dẹp tài nguyên đã tạo.
> Đây là minh chứng cho khả năng lập kế hoạch, gọi các subagent, và thực hiện chuỗi hành động phức tạp của một hệ thống Agentic AI như Antigravity.

## 3. Tóm tắt và Bài học chính

Phần này đã trang bị cho chúng ta những kiến thức và kỹ năng cần thiết để xây dựng hai trong số các thao tác CRUD cơ bản nhất cho một API RESTful: đọc chi tiết một tài nguyên và thêm mới một tài nguyên.

*   **Đọc tài nguyên theo ID (`GET by ID`)**:
    *   Đã triển khai phương thức `GetByIdAsync` trong `IWalkRepository` và `WalkRepository` để truy vấn một `Walk` Domain Model duy nhất.
    *   Hiểu rõ tầm quan trọng của `.Include()` trong Entity Framework Core để thực hiện **eager loading** các đối tượng liên quan (`Area`, `Difficulty`) và tránh **vấn đề N+1 query**.
    *   Đã tạo endpoint `[HttpGet("{id:Guid}")]` trong `WalksController` để xử lý yêu cầu `GET /api/Walks/{id}`.
    *   Sử dụng `[FromRoute]` để lấy ID từ URL và AutoMapper để chuyển đổi Domain Model sang `WalkDto` (DTO phản hồi).
    *   Trả về `200 OK` nếu tìm thấy và `404 Not Found` nếu không tìm thấy, tuân thủ tiêu chuẩn HTTP.
    *   Kiểm thử thành công bằng Swagger UI.
*   **Thêm mới tài nguyên (`POST`)**:
    *   Đã thiết kế `AddWalkRequestDto` làm DTO cho yêu cầu đầu vào, chỉ chứa các thông tin cần thiết từ client và không bao gồm `Id`.
    *   Đã triển khai phương thức `CreateAsync` trong `IWalkRepository` và `WalkRepository`.
    *   Trong `WalkRepository.CreateAsync`, chúng ta tự động gán `Guid.NewGuid()` cho `Id` của hành trình mới trên server để đảm bảo tính duy nhất và bảo mật, sau đó sử dụng `dbContext.Walks.AddAsync()` và `dbContext.SaveChangesAsync()` để lưu vào cơ sở dữ liệu.
    *   Đã tạo endpoint `[HttpPost]` trong `WalksController` để xử lý yêu cầu `POST /api/Walks`.
    *   Sử dụng `[FromBody]` để lấy dữ liệu từ body của yêu cầu và AutoMapper để chuyển đổi `AddWalkRequestDto` sang `Walk` Domain Model.
    *   Trả về `201 Created` bằng `CreatedAtAction`, bao gồm `WalkDto` của tài nguyên vừa tạo và một `Location` header trỏ đến URL của tài nguyên đó, tuân thủ nghiêm ngặt tiêu chuẩn RESTful.
    *   Kiểm thử thành công bằng Swagger UI, bao gồm việc sử dụng `AreaId` và `DifficultyId` hợp lệ.
*   **Các nguyên tắc quan trọng được áp dụng và củng cố**:
    *   **Dependency Injection**: Để inject `IMapper` và `IWalkRepository` vào `WalksController`, thúc đẩy tính linh hoạt và dễ kiểm thử.
    *   **Repository Pattern**: Tách biệt logic truy cập dữ liệu khỏi logic nghiệp vụ của Controller, cải thiện cấu trúc và khả năng bảo trì.
    *   **DTOs**: Sử dụng DTO cho cả yêu cầu và phản hồi để kiểm soát định dạng dữ liệu, bảo mật và phiên bản hóa API.
    *   **HTTP Verbs và Status Codes**: Sử dụng `GET` với `200 OK`/`404 Not Found` và `POST` với `201 Created` một cách chính xác theo tiêu chuẩn RESTful.
    *   **Asynchronous Programming (`async`/`await`)**: Để cải thiện khả năng mở rộng và hiệu suất của API bằng cách xử lý các hoạt động I/O một cách hiệu quả.
    *   **Entity Framework Core**: Hiểu cách tương tác với cơ sở dữ liệu, quản lý các mối quan hệ và tối ưu hóa truy vấn.

Với những kiến thức này, bạn đã có thể tự tin xây dựng các chức năng cơ bản cho một API RESTful, đặt nền móng vững chắc cho việc phát triển các tính năng phức tạp hơn trong tương lai.

<!-- REVIEWED_BY_AGENT -->
