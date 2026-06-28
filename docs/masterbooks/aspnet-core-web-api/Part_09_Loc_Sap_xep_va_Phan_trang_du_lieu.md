# Phần 9: Lọc, Sắp xếp và Phân trang dữ liệu trong RESTful Web API

## 1. Giới thiệu: Nhu cầu thao tác dữ liệu nâng cao

Trong kỷ nguyên của dữ liệu lớn và các ứng dụng phân tán, việc truy xuất thông tin từ một API không chỉ đơn thuần là "lấy tất cả". Các hệ thống client hiện đại đòi hỏi khả năng tinh chỉnh dữ liệu nhận được để tối ưu hóa hiệu suất, giảm tải mạng và cải thiện trải nghiệm người dùng. Điều này dẫn đến sự cần thiết của ba kỹ thuật cốt lõi trong phát triển Web API: **Lọc (Filtering)**, **Sắp xếp (Sorting)** và **Phân trang (Pagination)**.

Chương này sẽ đi sâu vào cách triển khai các kỹ thuật này một cách hiệu quả trong ASP.NET Core Web API sử dụng Entity Framework Core, tuân thủ chặt chẽ Repository Pattern và nguyên tắc Dependency Injection. Chúng ta sẽ khám phá cách các tham số truy vấn (query parameters) được sử dụng để điều khiển luồng dữ liệu, và tầm quan trọng sống còn của `IQueryable` trong việc tối ưu hóa hiệu suất truy vấn cơ sở dữ liệu. Đồng thời, chúng ta sẽ liên hệ với cách tiếp cận "Vibe Coding" và sức mạnh của "Antigravity IDE" trong việc tự động hóa và tinh chỉnh quá trình phát triển các tính năng này.

## 2. Chuẩn bị dữ liệu mẫu (Seeding Data)

Để thực hành và kiểm thử các tính năng lọc, sắp xếp, và phân trang một cách hiệu quả, việc có một tập dữ liệu đủ lớn và nhất quán là điều kiện tiên quyết. Quá trình "seeding data" (gieo hạt dữ liệu) đảm bảo rằng môi trường phát triển và kiểm thử của bạn luôn có một nền tảng dữ liệu đáng tin cậy.

### 2.1. Tầm quan trọng của Seeding Data

*   **Tính nhất quán:** Đảm bảo mọi nhà phát triển và môi trường triển khai đều làm việc trên cùng một bộ dữ liệu cơ bản, giúp dễ dàng tái hiện lỗi và kiểm tra tính năng.
*   **Kiểm thử:** Cung cấp đủ dữ liệu để kiểm tra các kịch bản phức tạp của lọc, sắp xếp và phân trang, đặc biệt là các trường hợp biên.
*   **Phát triển nhanh:** Giúp ứng dụng hoạt động ngay lập tức mà không cần nhập dữ liệu thủ công, tăng tốc quá trình phát triển ban đầu.
*   **Tái tạo:** Dễ dàng "reset" cơ sở dữ liệu về trạng thái ban đầu để kiểm thử các thay đổi mới.

### 2.2. Kịch bản Seeding SQL

Trong khóa học này, chúng ta sẽ sử dụng một script SQL để thêm dữ liệu mẫu vào cơ sở dữ liệu SQL Server. Phương pháp này đơn giản, trực tiếp và hiệu quả cho môi trường phát triển. Script sẽ thực hiện các bước sau:

1.  **Xóa dữ liệu hiện có:** Đảm bảo cơ sở dữ liệu sạch trước khi thêm dữ liệu mới để tránh trùng lặp và xung đột khóa chính.
2.  **Chèn dữ liệu `Difficulties`:** Các cấp độ khó cho các chuyến đi bộ.
3.  **Chèn dữ liệu `Regions`:** Các khu vực địa lý.
4.  **Chèn dữ liệu `Walks`:** Các chuyến đi bộ thực tế, liên kết với `Difficulties` và `Regions`.

**Ví dụ Script SQL (minh họa):**

```sql
-- Đảm bảo bạn đang sử dụng đúng cơ sở dữ liệu
USE [network.DB];
GO

-- Xóa dữ liệu hiện có (quan trọng để tránh trùng lặp và đảm bảo dữ liệu nhất quán)
-- Thứ tự DELETE phải tuân thủ ràng buộc khóa ngoại (foreign key constraints)
-- Xóa bảng con trước, bảng cha sau.
DELETE FROM [Walks];
DELETE FROM [Difficulties];
DELETE FROM [Regions];
GO

-- Chèn dữ liệu mẫu cho Difficulties
INSERT INTO [Difficulties] ([Id], [Name]) VALUES
(NEWID(), 'Easy'),
(NEWID(), 'Medium'),
(NEWID(), 'Hard');
GO

-- Chèn dữ liệu mẫu cho Regions
INSERT INTO [Regions] ([Id], [Name], [Code], [RegionImageUrl]) VALUES
(NEWID(), 'Northland', 'NTL', 'https://example.com/northland.jpg'),
(NEWID(), 'Auckland', 'AUK', 'https://example.com/auckland.jpg'),
(NEWID(), 'Wellington', 'WLG', 'https://example.com/wellington.jpg'),
(NEWID(), 'Canterbury', 'CAN', 'https://example.com/canterbury.jpg'),
(NEWID(), 'Otago', 'OTA', 'https://example.com/otago.jpg'),
(NEWID(), 'Fiordland', 'FIO', 'https://example.com/fiordland.jpg');
GO

-- Chèn dữ liệu mẫu cho Walks (ví dụ 15 bản ghi)
-- Lấy Id của Difficulties và Regions đã chèn
DECLARE @easyId UNIQUEIDENTIFIER = (SELECT Id FROM [Difficulties] WHERE Name = 'Easy');
DECLARE @mediumId UNIQUEIDENTIFIER = (SELECT Id FROM [Difficulties] WHERE Name = 'Medium');
DECLARE @hardId UNIQUEIDENTIFIER = (SELECT Id FROM [Difficulties] WHERE Name = 'Hard');

DECLARE @aucklandId UNIQUEIDENTIFIER = (SELECT Id FROM [Regions] WHERE Name = 'Auckland');
DECLARE @wellingtonId UNIQUEIDENTIFIER = (SELECT Id FROM [Regions] WHERE Name = 'Wellington');
DECLARE @fiordlandId UNIQUEIDENTIFIER = (SELECT Id FROM [Regions] WHERE Name = 'Fiordland');
DECLARE @canterburyId UNIQUEIDENTIFIER = (SELECT Id FROM [Regions] WHERE Name = 'Canterbury');
DECLARE @otagoId UNIQUEIDENTIFIER = (SELECT Id FROM [Regions] WHERE Name = 'Otago');
DECLARE @northlandId UNIQUEIDENTIFIER = (SELECT Id FROM [Regions] WHERE Name = 'Northland');

INSERT INTO [Walks] ([Id], [Name], [Description], [LengthInKm], [WalkImageUrl], [DifficultyId], [RegionId]) VALUES
(NEWID(), 'Botanic Garden Walk', 'A pleasant walk through the botanic gardens.', 2.5, 'https://example.com/botanic.jpg', @easyId, @wellingtonId),
(NEWID(), 'Mount Victoria Loop', 'A scenic loop around Mount Victoria.', 3.0, 'https://example.com/mtvictoria.jpg', @mediumId, @wellingtonId),
(NEWID(), 'Makara Beach Walk', 'Coastal walk with stunning views.', 5.0, 'https://example.com/makarabeach.jpg', @mediumId, @wellingtonId),
(NEWID(), 'Kepler Track', 'Famous multi-day great walk.', 60.0, 'https://example.com/kepler.jpg', @hardId, @fiordlandId),
(NEWID(), 'Routeburn Track', 'Another iconic great walk.', 32.0, 'https://example.com/routeburn.jpg', @hardId, @fiordlandId),
(NEWID(), 'Tongariro Alpine Crossing', 'One of the best day hikes in the world.', 19.4, 'https://example.com/tongariro.jpg', @hardId, @northlandId),
(NEWID(), 'Abel Tasman Coast Track', 'Coastal beauty in the north.', 60.0, 'https://example.com/abeltasman.jpg', @mediumId, @northlandId),
(NEWID(), 'Hooker Valley Track', 'Glacier views in Aoraki/Mount Cook.', 10.0, 'https://example.com/hookervalley.jpg', @easyId, @canterburyId),
(NEWID(), 'Roys Peak Track', 'Iconic Wanaka viewpoint.', 16.0, 'https://example.com/royspeak.jpg', @hardId, @otagoId),
(NEWID(), 'Queenstown Trail', 'Various trails around Queenstown.', 12.0, 'https://example.com/queenstown.jpg', @easyId, @otagoId),
(NEWID(), 'White Pine Bush Walk', 'Short bush walk near Masterton.', 1.5, 'https://example.com/whitepine.jpg', @easyId, @wellingtonId),
(NEWID(), 'Boulder Bank Walk', 'Unique walk on a natural rock formation.', 10.0, 'https://example.com/boulderbank.jpg', @mediumId, @northlandId),
(NEWID(), 'Coast to Coast Walkway', 'Urban walk across Auckland.', 16.0, 'https://example.com/coasttocoast.jpg', @mediumId, @aucklandId),
(NEWID(), 'Rangitoto Summit Track', 'Volcanic island hike.', 7.0, 'https://example.com/rangitoto.jpg', @mediumId, @aucklandId),
(NEWID(), 'Waiheke Island Walk', 'Coastal and vineyard walks.', 8.0, 'https://example.com/waiheke.jpg', @easyId, @aucklandId);
GO
```

> [!TIP]
> Đảm bảo tên cơ sở dữ liệu (`network.DB`) và tên bảng (`Regions`, `Difficulties`, `Walks`) trong script khớp với cấu hình của bạn. Nếu bạn đã làm theo các bài giảng trước, chúng sẽ khớp chính xác. Nếu không, hãy điều chỉnh script cho phù hợp.

### 2.3. Antigravity IDE & Vibe Coding: Hỗ trợ Seeding

Với "Antigravity IDE", quá trình seeding dữ liệu có thể trở nên mượt mà và tự động hơn. Thay vì chạy script SQL thủ công, bạn có thể áp dụng tư duy "Vibe Coding" để giao phó nhiệm vụ này cho Antigravity:

*   **Mô tả ý định:** Bạn chỉ cần mô tả ý định của mình, ví dụ: "Tôi cần seed dữ liệu cho các bảng `Regions`, `Difficulties` và `Walks` với khoảng 15 bản ghi cho `Walks`, đảm bảo dữ liệu liên kết và có tính đa dạng để kiểm thử lọc/sắp xếp/phân trang."
*   **Antigravity thực thi:** Antigravity IDE, với khả năng hiểu ngữ cảnh (Entity Framework Core, SQL Server), sẽ tự động:
    *   Phân tích cấu trúc cơ sở dữ liệu của bạn.
    *   Tạo ra script SQL seeding tương tự hoặc sử dụng các tính năng seeding của EF Core (nếu bạn yêu cầu).
    *   Thậm chí có thể tự động chạy script này hoặc hướng dẫn bạn cách chạy thông qua các lệnh `dotnet ef database update` với seeding logic nếu bạn đã cấu hình trong `DbContext`.
    *   Kiểm tra kết quả để đảm bảo dữ liệu đã được thêm thành công.

Điều này giúp bạn tập trung vào logic nghiệp vụ chính của API mà không mất thời gian vào các tác vụ cấu hình lặp đi lặp lại.

## 3. Lọc dữ liệu (Filtering)

Lọc là một tính năng không thể thiếu trong bất kỳ ứng dụng nào, cho phép người dùng thu hẹp một tập dữ liệu lớn thành một tập con nhỏ hơn, có liên quan hơn dựa trên các tiêu chí cụ thể. Trong RESTful API, việc này thường được thực hiện thông qua các tham số truy vấn trên endpoint `GET`.

### 3.1. Khái niệm và Nguyên lý hoạt động

**Lọc dữ liệu** là quá trình chọn lọc các bản ghi từ một tập dữ liệu gốc mà thỏa mãn một hoặc nhiều điều kiện nhất định. Ví dụ, trong danh sách các chuyến đi bộ, người dùng có thể muốn xem "tất cả các chuyến đi có tên chứa từ 'Track'" hoặc "tất cả các chuyến đi có độ khó là 'Easy'".

**Tầm quan trọng:**

*   **Hiệu suất:** Giảm đáng kể lượng dữ liệu được tải từ cơ sở dữ liệu, truyền qua mạng và xử lý trên client, cải thiện thời gian phản hồi API.
*   **Trải nghiệm người dùng (UX):** Giúp người dùng dễ dàng tìm kiếm thông tin mong muốn trong các tập dữ liệu lớn, tăng tính hữu dụng của ứng dụng.
*   **Tải trọng máy chủ:** Giảm tài nguyên (CPU, RAM) cần thiết trên máy chủ cơ sở dữ liệu và máy chủ ứng dụng bằng cách chỉ truy xuất những gì cần thiết.

### 3.2. Thiết kế API Endpoint (Controller)

Theo nguyên tắc của RESTful API, các thao tác truy vấn dữ liệu (GET) với các tiêu chí lọc nên được biểu diễn thông qua các tham số truy vấn (query parameters) trong URL. Chúng ta sẽ mở rộng phương thức `GetAllWalks` trong `WalksController` để chấp nhận hai tham số lọc:

*   `filterOn`: Tên cột mà client muốn áp dụng bộ lọc (ví dụ: "Name", "Description").
*   `filterQuery`: Giá trị hoặc từ khóa mà client muốn tìm kiếm trong cột đã chọn.

```csharp
// Controllers/WalksController.cs
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO; // Giả sử có DTO cho Walks
using NZWalks.API.Repositories; // Giả sử có IWalkRepository
using AutoMapper; // Sử dụng AutoMapper cho việc chuyển đổi DTO

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class WalksController : ControllerBase
    {
        private readonly IWalkRepository walkRepository;
        private readonly IMapper mapper; // Thêm IMapper

        public WalksController(IWalkRepository walkRepository, IMapper mapper) // Inject IMapper
        {
            this.walkRepository = walkRepository;
            this.mapper = mapper;
        }

        [HttpGet]
        public async Task<IActionResult> GetAllWalks(
            [FromQuery] string? filterOn, 
            [FromQuery] string? filterQuery)
        {
            // Logic để chuyển các tham số này đến repository sẽ được thêm sau
            var walksDomain = await walkRepository.GetAllAsync(filterOn, filterQuery);

            // Chuyển đổi Domain Model sang DTO bằng AutoMapper để gọn gàng hơn
            var walksDto = mapper.Map<List<WalkDto>>(walksDomain);
            
            return Ok(walksDto);
        }
        // ... các phương thức khác
    }
}
```

> [!NOTE]
> Các tham số `filterOn` và `filterQuery` được đánh dấu là `string?` (nullable string) vì chúng là tùy chọn. Người dùng không nhất thiết phải luôn lọc dữ liệu. Việc sử dụng `AutoMapper` giúp giảm bớt mã boilerplate khi chuyển đổi từ Domain Model sang DTO.

#### 3.2.2. Antigravity IDE & Vibe Coding: Thiết kế Controller

Khi sử dụng Antigravity IDE với "Vibe Coding", bạn không cần phải gõ từng dòng code thủ công. Bạn có thể mô tả ý định:

*   "Tôi muốn thêm khả năng lọc vào endpoint `GET /api/Walks`. Người dùng có thể lọc theo `Name` hoặc `Description` bằng cách truyền `filterOn` và `filterQuery` làm tham số truy vấn."

Antigravity IDE sẽ:
1.  Phân tích phương thức `GetAllWalks` hiện có.
2.  Tự động thêm các tham số `[FromQuery] string? filterOn` và `[FromQuery] string? filterQuery` vào chữ ký phương thức.
3.  Cập nhật lời gọi `walkRepository.GetAllAsync` để truyền các tham số mới.
4.  Nếu bạn chưa sử dụng `AutoMapper`, Antigravity có thể đề xuất và tích hợp nó để làm cho việc chuyển đổi DTO trở nên gọn gàng hơn.

Điều này cho phép bạn tập trung vào thiết kế API cấp cao thay vì chi tiết triển khai cú pháp.

### 3.3. Cập nhật Repository Interface

Để tuân thủ Repository Pattern và nguyên tắc Dependency Injection, chúng ta cần cập nhật giao diện `IWalkRepository`. Điều này tạo ra một "hợp đồng" rõ ràng về khả năng lọc dữ liệu mà bất kỳ triển khai nào của `IWalkRepository` cũng phải tuân thủ.

```csharp
// Repositories/IWalkRepository.cs
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public interface IWalkRepository
    {
        Task<List<Walk>> GetAllAsync(
            string? filterOn = null, 
            string? filterQuery = null);
        // ... các phương thức khác
    }
}
```

Các tham số được gán giá trị mặc định là `null` để chúng vẫn là tùy chọn.

#### 3.3.2. Antigravity IDE & Vibe Coding: Cập nhật Interface

Với Antigravity, việc cập nhật interface trở nên đơn giản:

*   "Cập nhật `IWalkRepository` để phương thức `GetAllAsync` chấp nhận các tham số lọc `filterOn` và `filterQuery`."

Antigravity sẽ tự động mở file `IWalkRepository.cs` và thêm các tham số vào chữ ký phương thức, đảm bảo tính nhất quán với Controller.

### 3.4. Triển khai Logic lọc trong Repository

Đây là nơi logic lọc thực tế được áp dụng. Điều cực kỳ quan trọng là phải sử dụng `IQueryable` để đảm bảo các thao tác lọc được dịch thành truy vấn SQL và thực thi trên cơ sở dữ liệu, thay vì tải toàn bộ dữ liệu vào bộ nhớ rồi mới lọc.

```csharp
// Repositories/SqlWalkRepository.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public class SqlWalkRepository : IWalkRepository
    {
        private readonly NZWalksDbContext dbContext;

        public SqlWalkRepository(NZWalksDbContext dbContext)
        {
            this.dbContext = dbContext;
        }

        public async Task<List<Walk>> GetAllAsync(
            string? filterOn = null, 
            string? filterQuery = null)
        {
            // Bước 1: Khởi tạo IQueryable để xây dựng truy vấn
            // Sử dụng Include() để eager load các đối tượng liên quan (Difficulty, Region)
            // tránh vấn đề N+1 query.
            var walks = dbContext.Walks
                .Include(x => x.Difficulty)
                .Include(x => x.Region)
                .AsQueryable(); // Quan trọng: Chuyển đổi sang IQueryable để xây dựng truy vấn động

            // Bước 2: Áp dụng Lọc (Filtering)
            if (!string.IsNullOrWhiteSpace(filterOn) && !string.IsNullOrWhiteSpace(filterQuery))
            {
                // Sử dụng StringComparison.OrdinalIgnoreCase để so sánh không phân biệt chữ hoa/thường
                if (filterOn.Equals("Name", StringComparison.OrdinalIgnoreCase))
                {
                    walks = walks.Where(x => x.Name != null && x.Name.Contains(filterQuery, StringComparison.OrdinalIgnoreCase));
                }
                else if (filterOn.Equals("Description", StringComparison.OrdinalIgnoreCase))
                {
                    walks = walks.Where(x => x.Description != null && x.Description.Contains(filterQuery, StringComparison.OrdinalIgnoreCase));
                }
                // Có thể mở rộng để lọc theo các cột khác như DifficultyName, RegionName
                // Tuy nhiên, việc lọc trên các thuộc tính điều hướng (navigation properties)
                // cần xử lý cẩn thận hơn để tránh lỗi nếu đối tượng liên quan là null.
                // Ví dụ:
                // else if (filterOn.Equals("DifficultyName", StringComparison.OrdinalIgnoreCase))
                // {
                //    walks = walks.Where(x => x.Difficulty != null && x.Difficulty.Name.Contains(filterQuery, StringComparison.OrdinalIgnoreCase));
                // }
            }

            // Bước cuối: Thực thi truy vấn và trả về kết quả
            // .ToListAsync() sẽ kích hoạt việc dịch IQueryable sang SQL và thực thi trên DB.
            return await walks.ToListAsync();
        }
        // ... các phương thức khác
    }
}
```

#### 3.4.1. Sức mạnh của IQueryable trong Entity Framework Core

Đây là khái niệm then chốt để tối ưu hóa hiệu suất:

*   **`IQueryable` vs `IEnumerable`:**
    *   **`IEnumerable`**: Là một interface cơ bản trong .NET cho phép lặp qua một tập hợp. Khi bạn áp dụng các thao tác LINQ (như `Where`, `OrderBy`) trên `IEnumerable`, các thao tác này sẽ được thực hiện *trong bộ nhớ ứng dụng*. Nếu bạn lấy `dbContext.Walks.ToList().Where(...)`, toàn bộ bảng `Walks` sẽ được tải từ cơ sở dữ liệu vào ứng dụng của bạn, sau đó mới lọc. Điều này cực kỳ kém hiệu quả cho tập dữ liệu lớn.
    *   **`IQueryable`**: Mở rộng `IEnumerable` bằng cách thêm khả năng xây dựng truy vấn biểu thức. Khi bạn áp dụng các thao tác LINQ trên `IQueryable`, các thao tác này không được thực thi ngay lập tức. Thay vào đó, chúng được "dịch" thành các câu lệnh SQL tương ứng (ví dụ: `WHERE`, `ORDER BY`, `OFFSET`, `FETCH NEXT`) và chỉ được thực thi trên cơ sở dữ liệu khi bạn gọi một phương thức kích hoạt thực thi truy vấn (như `ToList()`, `FirstOrDefault()`, `Count()`).
*   **Tối ưu hóa Truy vấn:** Bằng cách giữ lại `IQueryable` càng lâu càng tốt (trước khi gọi `.ToListAsync()`), Entity Framework Core có thể xây dựng một truy vấn SQL phức tạp duy nhất, gửi nó đến cơ sở dữ liệu và chỉ tải về những dữ liệu đã được lọc, sắp xếp và phân trang. Điều này giảm thiểu lưu lượng mạng, giảm tải cho máy chủ ứng dụng và tận dụng tối đa khả năng xử lý của cơ sở dữ liệu.

#### 3.4.2. Cơ chế lọc và tối ưu hóa truy vấn SQL

Khi bạn viết `walks.Where(x => x.Name.Contains(filterQuery, StringComparison.OrdinalIgnoreCase))`, Entity Framework Core sẽ dịch nó thành một mệnh đề `WHERE` trong SQL, thường sử dụng toán tử `LIKE` hoặc hàm `CONTAINS`:

```sql
SELECT [w].[Id], [w].[Description], [w].[DifficultyId], ...
FROM [Walks] AS [w]
LEFT JOIN [Difficulties] AS [d] ON [w].[DifficultyId] = [d].[Id]
LEFT JOIN [Regions] AS [r] ON [w].[RegionId] = [r].[Id]
WHERE [w].[Name] LIKE N'%<filterQuery>%' COLLATE Latin1_General_CI_AS; -- Ví dụ cho SQL Server
```

Việc này đảm bảo rằng việc lọc được thực hiện ở cấp độ cơ sở dữ liệu, nơi nó hiệu quả nhất.

#### 3.4.3. Antigravity IDE & Vibe Coding: Triển khai Logic lọc

Khi đến phần triển khai logic lọc, Antigravity IDE có thể là một trợ thủ đắc lực:

*   **Mô tả ý định:** "Trong `SqlWalkRepository`, tôi cần triển khai logic lọc cho `GetAllAsync`. Nếu `filterOn` là 'Name', hãy lọc các chuyến đi có `Name` chứa `filterQuery` (không phân biệt chữ hoa/thường). Tương tự cho `Description`."
*   **Antigravity thực thi:**
    *   Antigravity sẽ tự động thêm khối `if (!string.IsNullOrWhiteSpace(filterOn) && !string.IsNullOrWhiteSpace(filterQuery))`
    *   Nó sẽ tạo ra các mệnh đề `if (filterOn.Equals("Name", ...))` và `else if (filterOn.Equals("Description", ...))`.
    *   Quan trọng nhất, Antigravity sẽ sử dụng `IQueryable` và các phương thức LINQ (`.Where()`, `.Contains()`) một cách chính xác, đảm bảo rằng truy vấn được xây dựng một cách tối ưu và được dịch sang SQL hiệu quả.
    *   Nó có thể gợi ý các trường hợp biên như kiểm tra `x.Name != null` trước khi gọi `Contains` để tránh `NullReferenceException` nếu cột có thể chứa giá trị null.

Antigravity giúp bạn viết code đúng cách ngay từ đầu, giảm thiểu lỗi và tối ưu hóa hiệu suất mà không cần phải nhớ chi tiết cú pháp hoặc cơ chế EF Core.

### 3.5. Kiểm thử chức năng lọc với Swagger UI

Sau khi triển khai, bạn có thể kiểm thử chức năng lọc bằng Swagger UI:

1.  Chạy ứng dụng ASP.NET Core của bạn.
2.  Truy cập URL của Swagger UI (thường là `https://localhost:<port>/swagger`).
3.  Tìm đến endpoint `GET /api/Walks`.
4.  Trong phần "Try it out", nhấp vào "Execute" để xem tất cả các chuyến đi bộ.
5.  Nhập giá trị cho `filterOn` (ví dụ: "Name") và `filterQuery` (ví dụ: "Track").
6.  Nhấn "Execute" và quan sát kết quả đã lọc. Bạn sẽ chỉ thấy các chuyến đi bộ có tên chứa từ "Track".
7.  Thử với `filterOn=Description` và `filterQuery=stunning` để xem các chuyến đi có mô tả chứa từ "stunning".

## 4. Sắp xếp dữ liệu (Sorting)

Sắp xếp là một yêu cầu phổ biến khác cho phép người dùng xem dữ liệu theo một thứ tự cụ thể, chẳng hạn như theo tên tăng dần hoặc theo độ dài giảm dần.

### 4.1. Khái niệm và Lợi ích

**Sắp xếp dữ liệu** là quá trình sắp xếp các bản ghi trong một tập dữ liệu theo thứ tự tăng dần (ascending) hoặc giảm dần (descending) dựa trên giá trị của một hoặc nhiều cột.

**Tầm quan trọng:**

*   **Trình bày dữ liệu:** Giúp người dùng dễ dàng đọc và hiểu dữ liệu bằng cách hiển thị chúng theo một thứ tự hợp lý, chẳng hạn như theo thứ tự bảng chữ cái, theo ngày hoặc theo một số liệu quan trọng.
*   **Phân tích:** Hỗ trợ phân tích dữ liệu bằng cách nhóm các mục tương tự lại với nhau hoặc làm nổi bật các giá trị cao nhất/thấp nhất.
*   **Điều hướng:** Cải thiện khả năng điều hướng trong các danh sách dài, giúp người dùng nhanh chóng tìm thấy các mục cụ thể.

### 4.2. Thiết kế API Endpoint (Controller)

Chúng ta sẽ bổ sung hai tham số truy vấn vào phương thức `GetAllWalks` trong Controller để hỗ trợ sắp xếp:

*   `sortBy`: Tên cột mà client muốn sắp xếp (ví dụ: "Name", "LengthInKm").
*   `isAscending`: Một giá trị boolean cho biết thứ tự sắp xếp là tăng dần (`true`) hay giảm dần (`false`). Mặc định là `true` (tăng dần).

```csharp
// Controllers/WalksController.cs
// ... (các using và constructor như trên)

        [HttpGet]
        public async Task<IActionResult> GetAllWalks(
            [FromQuery] string? filterOn, 
            [FromQuery] string? filterQuery,
            [FromQuery] string? sortBy, // Tham số sắp xếp
            [FromQuery] bool? isAscending = true) // Tham số thứ tự sắp xếp, mặc định tăng dần
        {
            var walksDomain = await walkRepository.GetAllAsync(
                filterOn, 
                filterQuery,
                sortBy,
                isAscending); // Truyền các tham số mới đến repository

            var walksDto = mapper.Map<List<WalkDto>>(walksDomain);
            
            return Ok(walksDto);
        }
        // ... các phương thức khác
```

#### 4.2.2. Antigravity IDE & Vibe Coding: Thiết kế Controller cho Sắp xếp

Tương tự như lọc, bạn có thể hướng dẫn Antigravity IDE:

*   "Bổ sung khả năng sắp xếp vào endpoint `GET /api/Walks`. Cho phép sắp xếp theo `Name` hoặc `LengthInKm`, với tùy chọn tăng dần/giảm dần (`isAscending` mặc định là `true`)."

Antigravity sẽ tự động thêm `[FromQuery] string? sortBy` và `[FromQuery] bool? isAscending = true` vào chữ ký phương thức `GetAllWalks` và cập nhật lời gọi `walkRepository.GetAllAsync`.

### 4.3. Cập nhật Repository Interface

Cập nhật giao diện `IWalkRepository` để thêm các tham số sắp xếp vào "hợp đồng" của Repository.

```csharp
// Repositories/IWalkRepository.cs
// ... (using như trên)

    public interface IWalkRepository
    {
        Task<List<Walk>> GetAllAsync(
            string? filterOn = null, 
            string? filterQuery = null,
            string? sortBy = null, // Tham số sắp xếp
            bool? isAscending = true); // Tham số thứ tự sắp xếp
        // ... các phương thức khác
    }
```

### 4.4. Triển khai Logic sắp xếp trong Repository

Logic sắp xếp sẽ được thêm vào sau logic lọc, tiếp tục làm việc trên đối tượng `IQueryable<Walk>` để tối ưu hóa truy vấn.

```csharp
// Repositories/SqlWalkRepository.cs
// ... (các using và constructor như trên)

        public async Task<List<Walk>> GetAllAsync(
            string? filterOn = null, 
            string? filterQuery = null,
            string? sortBy = null, // Tham số sắp xếp
            bool? isAscending = true) // Tham số thứ tự sắp xếp
        {
            var walks = dbContext.Walks
                .Include(x => x.Difficulty)
                .Include(x => x.Region)
                .AsQueryable();

            // Bước 2: Áp dụng Lọc (Filtering) (như đã triển khai ở trên)
            if (!string.IsNullOrWhiteSpace(filterOn) && !string.IsNullOrWhiteSpace(filterQuery))
            {
                if (filterOn.Equals("Name", StringComparison.OrdinalIgnoreCase))
                {
                    walks = walks.Where(x => x.Name != null && x.Name.Contains(filterQuery, StringComparison.OrdinalIgnoreCase));
                }
                else if (filterOn.Equals("Description", StringComparison.OrdinalIgnoreCase))
                {
                    walks = walks.Where(x => x.Description != null && x.Description.Contains(filterQuery, StringComparison.OrdinalIgnoreCase));
                }
            }

            // Bước 3: Áp dụng Sắp xếp (Sorting)
            // Quan trọng: Sắp xếp phải được áp dụng sau khi lọc để sắp xếp trên tập dữ liệu đã được thu hẹp.
            if (!string.IsNullOrWhiteSpace(sortBy))
            {
                if (sortBy.Equals("Name", StringComparison.OrdinalIgnoreCase))
                {
                    walks = (isAscending == true) ? walks.OrderBy(x => x.Name) : walks.OrderByDescending(x => x.Name);
                }
                else if (sortBy.Equals("LengthInKm", StringComparison.OrdinalIgnoreCase))
                {
                    walks = (isAscending == true) ? walks.OrderBy(x => x.LengthInKm) : walks.OrderByDescending(x => x.LengthInKm);
                }
                // Có thể mở rộng để sắp xếp theo các cột khác, ví dụ: Difficulty.Name, Region.Name
                // Lưu ý: Sắp xếp trên các thuộc tính điều hướng cũng cần kiểm tra null nếu có thể.
                // else if (sortBy.Equals("DifficultyName", StringComparison.OrdinalIgnoreCase))
                // {
                //    walks = (isAscending == true) ? walks.OrderBy(x => x.Difficulty.Name) : walks.OrderByDescending(x => x.Difficulty.Name);
                // }
            }

            // Bước cuối: Thực thi truy vấn và trả về kết quả
            return await walks.ToListAsync();
        }
        // ... các phương thức khác
    }
}
```

> [!IMPORTANT]
> **Thứ tự của các thao tác rất quan trọng:** Bạn nên **Lọc** trước, sau đó **Sắp xếp**, và cuối cùng là **Phân trang**. Điều này đảm bảo bạn sắp xếp trên tập dữ liệu đã lọc và phân trang trên tập dữ liệu đã được sắp xếp, mang lại kết quả chính xác và hiệu quả.

#### 4.4.2. Antigravity IDE & Vibe Coding: Triển khai Logic sắp xếp

Với Antigravity, bạn có thể mô tả:

*   "Tiếp tục từ logic lọc, thêm logic sắp xếp. Nếu `sortBy` là 'Name', sắp xếp theo tên. Nếu là 'LengthInKm', sắp xếp theo độ dài. Sử dụng `isAscending` để xác định thứ tự."

Antigravity sẽ:
1.  Chèn khối `if (!string.IsNullOrWhiteSpace(sortBy))` sau khối lọc.
2.  Tạo ra các mệnh đề `if (sortBy.Equals("Name", ...))` và `else if (sortBy.Equals("LengthInKm", ...))`.
3.  Sử dụng toán tử ternary (`condition ? valueIfTrue : valueIfFalse`) để chọn giữa `OrderBy()` và `OrderByDescending()` một cách gọn gàng, tận dụng `IQueryable` để đảm bảo truy vấn SQL được tối ưu với mệnh đề `ORDER BY`.

### 4.5. Kiểm thử chức năng sắp xếp với Swagger UI

1.  Chạy ứng dụng.
2.  Truy cập endpoint `GET /api/Walks`.
3.  Trong phần "Try it out", bạn có thể:
    *   Sắp xếp theo tên tăng dần: `sortBy=Name&isAscending=true`
    *   Sắp xếp theo độ dài giảm dần: `sortBy=LengthInKm&isAscending=false`
    *   Kết hợp lọc và sắp xếp: `filterOn=Name&filterQuery=Track&sortBy=LengthInKm&isAscending=true`
4.  Nhấn "Execute" và quan sát kết quả.

## 5. Phân trang dữ liệu (Pagination)

Phân trang là một tính năng cực kỳ quan trọng đối với các API xử lý tập dữ liệu lớn. Thay vì trả về hàng ngàn hoặc hàng triệu bản ghi cùng lúc (có thể gây quá tải cho cả máy chủ và client), phân trang cho phép chia dữ liệu thành các "trang" nhỏ hơn, dễ quản lý hơn.

### 5.1. Khái niệm và Tầm quan trọng chiến lược

**Phân trang dữ liệu** là kỹ thuật giới hạn số lượng bản ghi được trả về trong một phản hồi API bằng cách chia tập dữ liệu thành các trang có kích thước cố định. Đây là một chiến lược thiết yếu để đảm bảo khả năng mở rộng và hiệu suất của API.

**Tầm quan trọng chiến lược:**

*   **Hiệu suất API và máy chủ:** Giảm đáng kể thời gian phản hồi bằng cách chỉ tải một lượng nhỏ dữ liệu mỗi lần. Điều này giảm tải cho cơ sở dữ liệu, giảm mức tiêu thụ bộ nhớ và CPU trên máy chủ ứng dụng.
*   **Khả năng mở rộng:** Cho phép API xử lý hiệu quả các tập dữ liệu ngày càng lớn mà không làm suy giảm hiệu suất.
*   **Trải nghiệm người dùng (UX) trên Client:** Client có thể hiển thị dữ liệu theo từng phần, tải thêm khi người dùng cuộn hoặc yêu cầu trang tiếp theo, mang lại trải nghiệm mượt mà và phản hồi nhanh hơn.
*   **Kiểm soát tài nguyên:** Ngăn chặn các yêu cầu "lấy tất cả" có thể làm sập hệ thống khi dữ liệu tăng trưởng không kiểm soát.

### 5.2. Thiết kế API Endpoint (Controller)

Chúng ta sẽ thêm hai tham số truy vấn vào phương thức `GetAllWalks` trong Controller để hỗ trợ phân trang:

*   `pageNumber`: Số trang mà client muốn truy xuất (mặc định là 1).
*   `pageSize`: Số lượng bản ghi trên mỗi trang (mặc định là 10 hoặc 20 là hợp lý hơn trong thực tế so với 1000).

```csharp
// Controllers/WalksController.cs
// ... (các using và constructor như trên)

        [HttpGet]
        public async Task<IActionResult> GetAllWalks(
            [FromQuery] string? filterOn, 
            [FromQuery] string? filterQuery,
            [FromQuery] string? sortBy, 
            [FromQuery] bool? isAscending = true,
            [FromQuery] int pageNumber = 1, // Số trang, mặc định là trang đầu tiên
            [FromQuery] int pageSize = 10) // Kích thước trang, mặc định 10 bản ghi/trang
        {
            // Đảm bảo pageSize không vượt quá giới hạn hợp lý và pageNumber là dương
            if (pageSize < 1 || pageSize > 100) pageSize = 10; // Giới hạn pageSize từ 1 đến 100
            if (pageNumber < 1) pageNumber = 1;

            var walksDomain = await walkRepository.GetAllAsync(
                filterOn, 
                filterQuery,
                sortBy,
                isAscending,
                pageNumber, // Truyền các tham số mới đến repository
                pageSize);

            var walksDto = mapper.Map<List<WalkDto>>(walksDomain);
            
            return Ok(walksDto);
        }
        // ... các phương thức khác
```

> [!TIP]
> **Chọn `pageSize` mặc định:** Mặc dù một số ví dụ có thể sử dụng `1000` làm mặc định, trong thực tế, bạn nên chọn một giá trị `pageSize` mặc định hợp lý (ví dụ: 10, 20, 50) để tránh tải quá nhiều dữ liệu nếu client không chỉ định. Việc giới hạn `pageSize` tối đa cũng là một biện pháp bảo vệ API khỏi các yêu cầu quá lớn.

#### 5.2.2. Antigravity IDE & Vibe Coding: Thiết kế Controller cho Phân trang

Với Antigravity IDE, bạn có thể chỉ định:

*   "Thêm khả năng phân trang vào endpoint `GET /api/Walks`. Cho phép người dùng chỉ định `pageNumber` (mặc định 1) và `pageSize` (mặc định 10). Đảm bảo `pageSize` không vượt quá 100 và `pageNumber` là số dương."

Antigravity sẽ:
1.  Thêm `[FromQuery] int pageNumber = 1` và `[FromQuery] int pageSize = 10` vào chữ ký phương thức.
2.  Chèn logic kiểm tra và điều chỉnh giá trị `pageSize` và `pageNumber` để bảo vệ API.
3.  Cập nhật lời gọi `walkRepository.GetAllAsync` để truyền các tham số phân trang.

### 5.3. Cập nhật Repository Interface

Cập nhật giao diện `IWalkRepository` để thêm các tham số phân trang.

```csharp
// Repositories/IWalkRepository.cs
// ... (using như trên)

    public interface IWalkRepository
    {
        Task<List<Walk>> GetAllAsync(
            string? filterOn = null, 
            string? filterQuery = null,
            string? sortBy = null, 
            bool? isAscending = true,
            int pageNumber = 1, // Số trang
            int pageSize = 10); // Kích thước trang
        // ... các phương thức khác
    }
```

### 5.4. Triển khai Logic phân trang trong Repository

Logic phân trang sẽ là thao tác cuối cùng được áp dụng trên đối tượng `IQueryable<Walk>` trước khi thực thi truy vấn.

```csharp
// Repositories/SqlWalkRepository.cs
// ... (các using và constructor như trên)

        public async Task<List<Walk>> GetAllAsync(
            string? filterOn = null, 
            string? filterQuery = null,
            string? sortBy = null, 
            bool? isAscending = true,
            int pageNumber = 1, // Số trang
            int pageSize = 10) // Kích thước trang
        {
            var walks = dbContext.Walks
                .Include(x => x.Difficulty)
                .Include(x => x.Region)
                .AsQueryable();

            // Bước 2: Áp dụng Lọc (Filtering)
            if (!string.IsNullOrWhiteSpace(filterOn) && !string.IsNullOrWhiteSpace(filterQuery))
            {
                if (filterOn.Equals("Name", StringComparison.OrdinalIgnoreCase))
                {
                    walks = walks.Where(x => x.Name != null && x.Name.Contains(filterQuery, StringComparison.OrdinalIgnoreCase));
                }
                else if (filterOn.Equals("Description", StringComparison.OrdinalIgnoreCase))
                {
                    walks = walks.Where(x => x.Description != null && x.Description.Contains(filterQuery, StringComparison.OrdinalIgnoreCase));
                }
            }

            // Bước 3: Áp dụng Sắp xếp (Sorting)
            if (!string.IsNullOrWhiteSpace(sortBy))
            {
                if (sortBy.Equals("Name", StringComparison.OrdinalIgnoreCase))
                {
                    walks = (isAscending == true) ? walks.OrderBy(x => x.Name) : walks.OrderByDescending(x => x.Name);
                }
                else if (sortBy.Equals("LengthInKm", StringComparison.OrdinalIgnoreCase))
                {
                    walks = (isAscending == true) ? walks.OrderBy(x => x.LengthInKm) : walks.OrderByDescending(x => x.LengthInKm);
                }
            }

            // Bước 4: Áp dụng Phân trang (Pagination)
            // Quan trọng: Phân trang phải được áp dụng sau khi lọc và sắp xếp.
            var skipResults = (pageNumber - 1) * pageSize;
            walks = walks.Skip(skipResults).Take(pageSize);

            // Bước cuối: Thực thi truy vấn và trả về kết quả
            // .ToListAsync() sẽ kích hoạt việc dịch IQueryable sang SQL và thực thi trên DB.
            return await walks.ToListAsync();
        }
        // ... các phương thức khác
    }
}
```

#### 5.4.1. Sử dụng Skip và Take với IQueryable

Các phương thức `Skip()` và `Take()` của LINQ là công cụ chính để triển khai phân trang.

*   `Skip(n)`: Bỏ qua `n` bản ghi đầu tiên của tập dữ liệu.
*   `Take(m)`: Lấy `m` bản ghi tiếp theo sau khi đã bỏ qua.

**Công thức phân trang:**

`skipResults = (pageNumber - 1) * pageSize;`

*   **Ví dụ 1:** `pageNumber = 1, pageSize = 5`
    *   `skipResults = (1 - 1) * 5 = 0`.
    *   `walks.Skip(0).Take(5)`: Bỏ qua 0 bản ghi, lấy 5 bản ghi đầu tiên.
*   **Ví dụ 2:** `pageNumber = 2, pageSize = 5`
    *   `skipResults = (2 - 1) * 5 = 5`.
    *   `walks.Skip(5).Take(5)`: Bỏ qua 5 bản ghi, lấy 5 bản ghi tiếp theo (tức là từ bản ghi thứ 6 đến 10).
*   **Ví dụ 3:** `pageNumber = 3, pageSize = 5`
    *   `skipResults = (3 - 1) * 5 = 10`.
    *   `walks.Skip(10).Take(5)`: Bỏ qua 10 bản ghi, lấy 5 bản ghi tiếp theo (tức là từ bản ghi thứ 11 đến 15).

#### 5.4.2. Cơ chế OFFSET/FETCH NEXT trong SQL

Khi `Skip()` và `Take()` được áp dụng trên `IQueryable`, Entity Framework Core sẽ dịch chúng thành các câu lệnh SQL tương ứng, thường là sử dụng mệnh đề `OFFSET` và `FETCH NEXT` (hoặc `LIMIT` và `OFFSET` trong PostgreSQL/MySQL):

```sql
SELECT [w].[Id], [w].[Description], [w].[DifficultyId], ...
FROM [Walks] AS [w]
LEFT JOIN [Difficulties] AS [d] ON [w].[DifficultyId] = [d].[Id]
LEFT JOIN [Regions] AS [r] ON [w].[RegionId] = [r].[Id]
-- Các mệnh đề WHERE và ORDER BY (nếu có) sẽ ở đây
ORDER BY [w].[Name] -- Cần có ORDER BY để OFFSET/FETCH NEXT hoạt động đúng
OFFSET <skipResults> ROWS FETCH NEXT <pageSize> ROWS ONLY;
```

Cơ chế này cực kỳ hiệu quả vì toàn bộ quá trình phân trang được xử lý trực tiếp bởi cơ sở dữ liệu, đảm bảo chỉ những bản ghi cần thiết cho trang hiện tại mới được truy xuất.

#### 5.4.3. Antigravity IDE & Vibe Coding: Triển khai Logic phân trang

Với Antigravity IDE, bạn có thể đơn giản hóa việc triển khai phân trang:

*   **Mô tả ý định:** "Sau khi lọc và sắp xếp, áp dụng phân trang bằng cách sử dụng `pageNumber` và `pageSize`. Tính toán số lượng bản ghi cần bỏ qua và sử dụng `Skip()` và `Take()`."

Antigravity sẽ:
1.  Chèn khối logic phân trang sau khối sắp xếp.
2.  Tự động tính toán `skipResults = (pageNumber - 1) * pageSize;`.
3.  Áp dụng `walks = walks.Skip(skipResults).Take(pageSize);` một cách chính xác.
4.  Có thể nhắc nhở bạn rằng `ORDER BY` là bắt buộc khi sử dụng `OFFSET/FETCH NEXT` trong SQL và đảm bảo rằng logic sắp xếp đã được áp dụng trước đó.

### 5.5. Kiểm thử chức năng phân trang với Swagger UI

1.  Chạy ứng dụng.
2.  Truy cập endpoint `GET /api/Walks`.
3.  Trong phần "Try it out", bạn có thể:
    *   Chỉ lấy 5 bản ghi đầu tiên: `pageNumber=1&pageSize=5`
    *   Lấy 5 bản ghi tiếp theo: `pageNumber=2&pageSize=5`
    *   Kết hợp lọc, sắp xếp và phân trang: `filterOn=Name&filterQuery=Track&sortBy=LengthInKm&isAscending=true&pageNumber=1&pageSize=2`
4.  Nhấn "Execute" và quan sát kết quả. Bạn sẽ thấy số lượng bản ghi được trả về chính xác theo `pageSize` và `pageNumber` yêu cầu.

### 5.6. Bổ sung: Thông tin phân trang (Metadata)

Một API phân trang hoàn chỉnh thường không chỉ trả về dữ liệu của trang hiện tại mà còn cung cấp "metadata" (siêu dữ liệu) về tổng số bản ghi, tổng số trang, trang hiện tại, v.v. Điều này giúp client xây dựng các điều khiển phân trang (như nút "Trang trước", "Trang sau", hiển thị số trang).

Bạn có thể thêm metadata bằng cách:

1.  **Đếm tổng số bản ghi:** Trước khi áp dụng `Skip()` và `Take()`, hãy gọi `await walks.CountAsync()` để lấy tổng số bản ghi sau khi đã lọc và sắp xếp.
2.  **Tạo một DTO chứa Metadata:** Tạo một DTO mới (ví dụ: `PagedResponseDto<T>`) chứa `List<T> Data` và các thuộc tính metadata như `TotalCount`, `PageSize`, `CurrentPage`, `TotalPages`.
3.  **Trả về PagedResponseDto:** Trong Controller, đóng gói `walksDto` và metadata vào `PagedResponseDto` trước khi trả về.

```csharp
// Ví dụ trong Controller (không phải Repository)
// ...
        [HttpGet]
        public async Task<IActionResult> GetAllWalks(
            // ... các tham số lọc, sắp xếp, phân trang
            [FromQuery] int pageNumber = 1, 
            [FromQuery] int pageSize = 10)
        {
            // ... (xử lý validation pageNumber, pageSize)

            // Lấy tổng số bản ghi TRƯỚC KHI áp dụng Skip/Take
            var totalCount = await walkRepository.CountAllAsync(filterOn, filterQuery); // Cần thêm phương thức CountAllAsync vào repo

            var walksDomain = await walkRepository.GetAllAsync(
                filterOn, filterQuery, sortBy, isAscending, pageNumber, pageSize);

            var walksDto = mapper.Map<List<WalkDto>>(walksDomain);

            var totalPages = (int)Math.Ceiling((double)totalCount / pageSize);

            var pagedResponse = new PagedResponseDto<WalkDto> // Giả sử có DTO này
            {
                Data = walksDto,
                TotalCount = totalCount,
                PageSize = pageSize,
                CurrentPage = pageNumber,
                TotalPages = totalPages
            };
            
            return Ok(pagedResponse);
        }
// ...
```

Việc này giúp client có đầy đủ thông tin để quản lý trạng thái phân trang.

## 6. Tổng kết và Tối ưu hóa luồng dữ liệu

Chương này đã trang bị cho bạn các kỹ thuật mạnh mẽ để quản lý và truy xuất dữ liệu một cách hiệu quả trong RESTful Web API với ASP.NET Core và Entity Framework Core:

*   **Chuẩn bị dữ liệu mẫu (Seeding Data):** Nền tảng cho mọi quá trình kiểm thử và phát triển. Antigravity IDE có thể tự động hóa việc này.
*   **Lọc dữ liệu (Filtering):** Cho phép client thu hẹp tập dữ liệu bằng các tham số truy vấn (`filterOn`, `filterQuery`), giúp cải thiện hiệu suất và trải nghiệm người dùng.
*   **Sắp xếp dữ liệu (Sorting):** Cung cấp khả năng sắp xếp dữ liệu theo thứ tự mong muốn (`sortBy`, `isAscending`), tăng tính dễ đọc và khả năng phân tích.
*   **Phân trang dữ liệu (Pagination):** Giải pháp tối ưu cho việc xử lý tập dữ liệu lớn bằng cách chia nhỏ thành các trang (`pageNumber`, `pageSize`), đảm bảo khả năng mở rộng và giảm tải.

**Điểm mấu chốt để tối ưu hóa:**

*   **`IQueryable` là chìa khóa:** Luôn sử dụng `IQueryable` và các phương thức LINQ của nó (`Where()`, `OrderBy()`, `Skip()`, `Take()`) để đảm bảo rằng các thao tác này được dịch thành các truy vấn SQL hiệu quả và thực thi trên cơ sở dữ liệu, thay vì tải toàn bộ dữ liệu vào bộ nhớ ứng dụng.
*   **Thứ tự các thao tác:** Luôn áp dụng các thao tác theo thứ tự **Lọc -> Sắp xếp -> Phân trang** để đảm bảo tính đúng đắn của kết quả.
*   **Repository Pattern & Dependency Injection:** Các kiến trúc này giúp tách biệt logic truy cập dữ liệu khỏi Controller, làm cho mã nguồn dễ bảo trì, kiểm thử và mở rộng hơn.
*   **Antigravity IDE & Vibe Coding:** Bằng cách mô tả ý định của bạn, Antigravity IDE có thể tự động sinh mã, refactor và tối ưu hóa các phần này, giúp bạn tập trung vào "vibe" của giải pháp thay vì các chi tiết triển khai lặp đi lặp lại. Đây là một minh chứng rõ ràng về cách các công cụ AI hiện đại có thể nâng cao năng suất và chất lượng mã nguồn trong phát triển phần mềm.

Với các kỹ thuật này, API của bạn không chỉ mạnh mẽ mà còn hiệu quả và thân thiện với người dùng, sẵn sàng xử lý các yêu cầu dữ liệu phức tạp trong các ứng dụng thực tế.

<!-- REVIEWED_BY_AGENT -->
