# Phần 24: Dự án thực hành: API Hành trình (Walks) - Khởi tạo & Đọc dữ liệu

## Giới thiệu Tổng quan

Chào mừng bạn đến với Phần 24 của khóa học, nơi chúng ta sẽ bắt tay vào dự án thực hành thứ hai: xây dựng một RESTful Web API hoàn chỉnh cho thực thể "Hành trình" (Walks). Sau khi đã củng cố kiến thức với API Khu vực (Regions), giờ là lúc áp dụng và mở rộng các kỹ năng đó để tạo ra một API mạnh mẽ và linh hoạt hơn, xử lý các mối quan hệ phức tạp và các chiến lược tải dữ liệu hiệu quả.

Trong phần này, chúng ta sẽ tập trung vào việc thiết lập nền tảng vững chắc cho API Walks và triển khai các chức năng đọc dữ liệu cơ bản. Đây cũng là cơ hội để chúng ta suy ngẫm về **Vibe Coding** – một triết lý lập trình hướng đến việc cảm nhận và hiện thực hóa "cái vibe" (linh hồn, ý định) của hệ thống thông qua cấu trúc mã nguồn và kiến trúc. Khi bạn hình dung một API "tốt," nó mang lại cảm giác gì? Dễ hiểu, dễ mở rộng, đáng tin cậy? Chúng ta sẽ cùng nhau xây dựng API Walks để phản ánh "vibe" đó.

**Mục tiêu chính của Phần này bao gồm:**

*   **Khởi tạo kiến trúc dự án:** Thiết lập Repository Pattern và Dependency Injection (DI) chuyên biệt cho thực thể `Walk`.
*   **Triển khai chức năng đọc dữ liệu:** Xây dựng các endpoint API để lấy danh sách tất cả hành trình và lấy thông tin chi tiết của một hành trình cụ thể theo ID.
*   **Tối ưu hóa truy vấn dữ liệu:** Sử dụng các thuộc tính điều hướng (Navigation Properties) của Entity Framework Core để tải dữ liệu liên quan (như thông tin về Khu vực và Độ khó của Hành trình) một cách hiệu quả thông qua kỹ thuật Eager Loading.
*   **Sử dụng Data Transfer Objects (DTOs) và AutoMapper:** Đảm bảo tách biệt rõ ràng giữa mô hình miền (Domain Model) và mô hình API (API Contract), đồng thời đơn giản hóa quá trình ánh xạ dữ liệu.
*   **Kiểm thử API:** Sử dụng Swagger UI để kiểm tra và xác nhận hoạt động của các endpoint API đã xây dựng, đồng thời áp dụng tư duy "Vibe Testing" để đánh giá chất lượng phản hồi.

Xuyên suốt phần này, chúng ta sẽ không chỉ củng cố sự hiểu biết về Dependency Injection, Repository Pattern, cách thiết kế Controller, và các HTTP Verbs cơ bản, mà còn khám phá sâu hơn về cách Entity Framework Core xử lý các mối quan hệ giữa các thực thể và cách chúng ta có thể tối ưu hóa trải nghiệm phát triển với sự hỗ trợ của các công cụ hiện đại, bao gồm cả tiềm năng của các hệ thống AI Agentic như Antigravity IDE.

## I. Khởi tạo Cấu trúc API Hành trình (Walks API)

Để xây dựng một API bền vững, dễ bảo trì và có "vibe" tốt, chúng ta sẽ bắt đầu bằng việc thiết lập kiến trúc cơ bản cho thực thể `Walk`. Điều này bao gồm việc định nghĩa các giao diện và lớp triển khai cho Repository, cũng như tích hợp chúng vào hệ thống Dependency Injection của ứng dụng.

### 1. Định nghĩa Domain Model `Walk`

Trong kiến trúc Clean Architecture, Domain Model là trái tim của ứng dụng, chứa đựng các khái niệm, dữ liệu và logic nghiệp vụ cốt lõi. Thực thể `Walk` đại diện cho một hành trình cụ thể, với các thuộc tính mô tả nó và các mối quan hệ với các thực thể khác.

> [!NOTE]
> **Domain Model là gì và tầm quan trọng của Navigation Properties?**
>
> **Domain Model** là một biểu diễn trừu tượng của thế giới thực trong bối cảnh nghiệp vụ của ứng dụng. Nó không bị ràng buộc bởi công nghệ cơ sở dữ liệu hay giao diện người dùng. Với `Walk`, các thuộc tính như `Name`, `Description`, `LengthInKm` là những đặc điểm cơ bản.
>
> **Navigation Properties** (Thuộc tính Điều hướng) là một khái niệm mạnh mẽ trong Entity Framework Core, cho phép chúng ta biểu diễn các mối quan hệ giữa các thực thể trong Domain Model. Ví dụ:
> *   `public Guid RegionId { get; set; }` và `public Region Region { get; set; }` tạo thành một mối quan hệ một-nhiều (One-to-Many): một `Region` có thể có nhiều `Walk`, nhưng mỗi `Walk` chỉ thuộc về một `Region`. `RegionId` là khóa ngoại (Foreign Key), còn `Region` là thuộc tính điều hướng.
> *   Tương tự, `WalkDifficultyId` và `WalkDifficulty` thiết lập mối quan hệ giữa `Walk` và `WalkDifficulty`.
>
> Các thuộc tính điều hướng này cho phép chúng ta "điều hướng" từ một thực thể `Walk` để truy cập trực tiếp các đối tượng `Region` và `WalkDifficulty` liên quan, thay vì phải thực hiện các truy vấn join thủ công. Điều này làm cho code trở nên trực quan và dễ đọc hơn, đồng thời là nền tảng cho các kỹ thuật tải dữ liệu liên quan mà chúng ta sẽ khám phá sau.

**Cấu trúc Domain Model `Walk`:**

```csharp
// NZWalks.API/Models/Domain/Walk.cs
using System;

namespace NZWalks.API.Models.Domain
{
    public class Walk
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
        public double LengthInKm { get; set; }
        public string? WalkImageUrl { get; set; } // Có thể null

        // Khóa ngoại (Foreign Keys)
        public Guid RegionId { get; set; }
        public Guid WalkDifficultyId { get; set; }

        // Thuộc tính điều hướng (Navigation Properties)
        public Region Region { get; set; }
        public WalkDifficulty WalkDifficulty { get; set; }
    }
}
```

> [!TIP]
> **Vibe Coding & Antigravity IDE trong việc định nghĩa Domain Model:**
>
> Khi định nghĩa `Walk` Domain Model, tư duy Vibe Coding giúp chúng ta tập trung vào *ý nghĩa nghiệp vụ* của một hành trình. Điều gì thực sự cần thiết để mô tả một `Walk`? Những mối quan hệ nào nó có với các thực thể khác?
>
> Với một AI Agentic như Antigravity IDE, bạn có thể "truyền vibe" này một cách hiệu quả:
> *   "Antigravity, tạo một Domain Model tên `Walk` với các thuộc tính `Id`, `Name`, `Description`, `LengthInKm`, `WalkImageUrl`. Nó có mối quan hệ một-nhiều với `Region` và một-một với `WalkDifficulty`. Hãy tạo cả khóa ngoại và thuộc tính điều hướng."
> *   Antigravity sẽ không chỉ tạo file `Walk.cs` mà còn có thể gợi ý các mối quan hệ trong `Region.cs` và `WalkDifficulty.cs` (nếu cần), đảm bảo tính nhất quán của Domain Model. Nó tự động hiểu rằng `Region` và `WalkDifficulty` cần được đưa vào `using` statement và có thể kiểm tra xem các thực thể đó đã tồn tại chưa.

### 2. Triển khai Repository Pattern cho `Walk`

Repository Pattern là một mẫu thiết kế nền tảng trong các ứng dụng có kiến trúc Clean Architecture. Nó không chỉ là việc "trừu tượng hóa truy cập dữ liệu" mà còn là một cam kết về việc tách biệt rõ ràng giữa logic nghiệp vụ và chi tiết về lưu trữ dữ liệu.

#### 2.1. Tầm quan trọng của Repository Pattern

Hãy hình dung lớp nghiệp vụ của bạn cần lấy danh sách các hành trình. Nếu nó trực tiếp gọi `_dbContext.Walks.ToList()`, nó sẽ bị ràng buộc chặt chẽ với Entity Framework Core và `NZWalksDbContext`. Điều gì xảy ra nếu bạn muốn chuyển sang một cơ sở dữ liệu NoSQL, hoặc một dịch vụ lưu trữ đám mây khác? Bạn sẽ phải thay đổi logic ở mọi nơi truy cập dữ liệu.

Repository Pattern giải quyết vấn đề này bằng cách:
*   **Trừu tượng hóa Data Access Layer (DAL):** Lớp nghiệp vụ không cần biết dữ liệu được lưu trữ ở đâu hay lấy ra như thế nào (SQL, NoSQL, API bên ngoài). Nó chỉ tương tác với một giao diện `IWalkRepository`.
*   **Tăng khả năng kiểm thử (Testability):** Bạn có thể dễ dàng tạo các phiên bản giả (mock) của `IWalkRepository` để kiểm thử logic nghiệp vụ mà không cần một cơ sở dữ liệu thực sự. Điều này đặc biệt quan trọng cho Unit Tests.
*   **Dễ bảo trì và mở rộng (Maintainability & Extensibility):** Khi có thay đổi về công nghệ lưu trữ dữ liệu, bạn chỉ cần thay đổi lớp triển khai Repository (`WalkRepository`), không ảnh hưởng đến Controller hay các lớp nghiệp vụ khác.
*   **Đảm bảo tính nhất quán:** Repository có thể đảm bảo rằng tất cả các truy vấn dữ liệu cho một thực thể tuân theo các quy tắc nghiệp vụ hoặc tối ưu hóa nhất định.

#### 2.2. Định nghĩa Interface `IWalkRepository`

Giao diện là "hợp đồng" mà lớp Repository phải tuân thủ. Nó định nghĩa các thao tác mà bất kỳ ai muốn tương tác với dữ liệu `Walk` đều có thể thực hiện.

```csharp
// NZWalks.API/Repositories/IWalkRepository.cs
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public interface IWalkRepository
    {
        Task<IEnumerable<Walk>> GetAllAsync(); // Lấy tất cả các hành trình
        Task<Walk?> GetByIdAsync(Guid id);     // Lấy một hành trình theo ID
        // Các phương thức khác sẽ được thêm sau (ví dụ: Add, Update, Delete)
    }
}
```

#### 2.3. Triển khai Lớp `WalkRepository`

Lớp `WalkRepository` hiện thực hóa "hợp đồng" `IWalkRepository` bằng cách sử dụng `NZWalksDbContext` để tương tác với cơ sở dữ liệu. Để `WalkRepository` có thể truy cập `NZWalksDbContext`, chúng ta sử dụng kỹ thuật **Constructor Injection** – một hình thức của Dependency Injection.

```csharp
// NZWalks.API/Repositories/WalkRepository.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public class WalkRepository : IWalkRepository
    {
        private readonly NZWalksDbContext _dbContext;

        // Constructor Injection: NZWalksDbContext được "inject" vào đây bởi DI Container
        public WalkRepository(NZWalksDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task<IEnumerable<Walk>> GetAllAsync()
        {
            // Logic lấy tất cả hành trình sẽ được triển khai chi tiết ở phần sau
            return await _dbContext.Walks.ToListAsync();
        }

        public async Task<Walk?> GetByIdAsync(Guid id)
        {
            // Logic lấy hành trình theo ID sẽ được triển khai chi tiết ở phần sau
            return await _dbContext.Walks.FirstOrDefaultAsync(x => x.Id == id);
        }
    }
}
```

#### 2.4. Đăng ký Repository vào Dependency Injection Container

Để các thành phần khác trong ứng dụng (như Controller) có thể sử dụng `IWalkRepository` mà không cần biết đến lớp triển khai cụ thể (`WalkRepository`), chúng ta cần đăng ký ánh xạ này vào container Dependency Injection (DI) của ASP.NET Core.

> [!NOTE]
> **Dependency Injection (DI) và Cơ chế hoạt động:**
>
> **Dependency Injection** là một nguyên tắc thiết kế giúp giảm sự phụ thuộc cứng giữa các thành phần phần mềm. Thay vì một lớp tự tạo ra các đối tượng phụ thuộc của nó (ví dụ: `new WalkRepository(new NZWalksDbContext())`), nó sẽ "nhận" các đối tượng đó từ bên ngoài, thường là thông qua constructor (Constructor Injection) hoặc qua thuộc tính (Property Injection).
>
> **DI Container (hay IoC Container)** là một framework chịu trách nhiệm quản lý vòng đời của các đối tượng và tiêm các phụ thuộc vào chúng. Khi bạn yêu cầu một dịch vụ (ví dụ: `IWalkRepository`), container sẽ:
> 1.  Tìm xem `IWalkRepository` được đăng ký với lớp triển khai nào (`WalkRepository`).
> 2.  Kiểm tra các phụ thuộc của `WalkRepository` (ví dụ: `NZWalksDbContext`).
> 3.  Nếu `NZWalksDbContext` chưa có, nó sẽ tạo một thể hiện của `NZWalksDbContext` (dựa trên cấu hình lifetime của nó).
> 4.  Tạo một thể hiện của `WalkRepository`, truyền `NZWalksDbContext` vào constructor của nó.
> 5.  Trả về thể hiện `WalkRepository` đã tạo cho bạn.
>
> **Lifetime của Service:** Khi đăng ký dịch vụ vào DI container, bạn cần chỉ định vòng đời (lifetime) của dịch vụ đó, quyết định khi nào một thể hiện mới được tạo ra:
> *   `AddTransient()`: Một thể hiện mới được tạo *mỗi khi* dịch vụ được yêu cầu. Phù hợp cho các dịch vụ nhẹ, không trạng thái, hoặc những dịch vụ cần một thể hiện độc lập mỗi lần sử dụng.
> *   `AddScoped()`: Một thể hiện được tạo *một lần cho mỗi phạm vi yêu cầu HTTP*. Nó được chia sẻ trong toàn bộ yêu cầu HTTP hiện tại. Đây là lựa chọn phổ biến nhất cho `DbContext` và Repository, vì nó đảm bảo rằng tất cả các hoạt động cơ sở dữ liệu trong cùng một yêu cầu HTTP sử dụng cùng một ngữ cảnh, giúp quản lý giao dịch và trạng thái dữ liệu hiệu quả.
> *   `AddSingleton()`: Một thể hiện duy nhất được tạo ra khi ứng dụng khởi động lần đầu và được sử dụng lại trong suốt vòng đời của ứng dụng. Phù hợp cho các dịch vụ không trạng thái, có trạng thái toàn cục, hoặc tài nguyên đắt tiền cần chia sẻ.
>
> Trong trường hợp của Repository, `AddScoped` là lựa chọn tối ưu vì nó đồng bộ hóa các hoạt động database trong một yêu cầu HTTP duy nhất, tránh các vấn đề về quản lý trạng thái hoặc xung đột dữ liệu.

```csharp
// NZWalks.API/Program.cs (một phần)
using NZWalks.API.Data;
using NZWalks.API.Mappings;
using NZWalks.API.Repositories;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Thêm các dịch vụ vào container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Cấu hình DbContext với AddScoped mặc định
builder.Services.AddDbContext<NZWalksDbContext>(options =>
options.UseSqlServer(builder.Configuration.GetConnectionString("NZWalksConnectionString")));

// Đăng ký Repository cho Region (đã có từ các phần trước)
builder.Services.AddScoped<IRegionRepository, SQLRegionRepository>();

// Đăng ký Repository cho Walk
builder.Services.AddScoped<IWalkRepository, WalkRepository>();

// Đăng ký AutoMapper (sẽ giải thích chi tiết sau)
builder.Services.AddAutoMapper(typeof(AutoMapperProfiles)); // Đăng ký tất cả Profiles trong assembly này

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
> **Antigravity IDE và Tự động hóa DI:**
>
> Antigravity IDE, với khả năng tự chạy script và đọc/ghi file, có thể tự động hóa đáng kể quá trình đăng ký DI.
> *   **Gợi ý thông minh:** Khi bạn tạo `IWalkRepository` và `WalkRepository`, Antigravity có thể tự động nhận diện và gợi ý dòng `builder.Services.AddScoped<IWalkRepository, WalkRepository>();` để thêm vào `Program.cs`.
> *   **Thực thi lệnh:** Bạn có thể ra lệnh: "Antigravity, đăng ký `WalkRepository` như là một `IWalkRepository` với lifetime `Scoped` trong `Program.cs`." Antigravity sẽ tìm đúng vị trí và thêm mã cần thiết, đảm bảo cấu hình chính xác và nhất quán.
> *   **Kiểm tra lỗi:** Nó thậm chí có thể kiểm tra xem bạn đã đăng ký tất cả các phụ thuộc cần thiết chưa hoặc có sử dụng lifetime phù hợp không, giúp bạn duy trì "vibe" của một ứng dụng được cấu hình đúng đắn.

## II. Triển khai Chức năng Đọc Dữ liệu (Read Operations)

Sau khi đã thiết lập cấu trúc cơ bản và đăng ký các dịch vụ, chúng ta sẽ tập trung vào việc triển khai hai chức năng đọc dữ liệu chính: lấy tất cả hành trình và lấy một hành trình theo ID. Để làm điều này một cách hiệu quả và chuyên nghiệp, chúng ta sẽ giới thiệu khái niệm Data Transfer Objects (DTOs) và thư viện AutoMapper.

### 1. Thiết lập Data Transfer Objects (DTOs)

#### 1.1. Tại sao cần DTOs? Tách biệt Domain Model và API Contract

Nếu bạn sử dụng trực tiếp Domain Model (`Walk`) để trả về dữ liệu qua API, bạn có thể gặp phải nhiều vấn đề:

1.  **Tiết lộ thông tin không cần thiết/nhạy cảm:** Domain Model có thể chứa các thuộc tính nội bộ, logic nghiệp vụ phức tạp, hoặc các trường nhạy cảm (ví dụ: mật khẩu hash, trường audit nội bộ) mà client không cần hoặc không nên thấy.
2.  **Vấn đề Over-fetching/Under-fetching:** Client có thể nhận quá nhiều dữ liệu (Over-fetching) hoặc không đủ dữ liệu (Under-fetching) nếu Domain Model không khớp chính xác với nhu cầu của API.
3.  **Vòng lặp tham chiếu (Circular References):** Trong các mối quan hệ hai chiều phức tạp (ví dụ: `Region` có danh sách `Walks`, và mỗi `Walk` lại có thuộc tính `Region`), việc serialize Domain Model trực tiếp có thể gây ra lỗi vòng lặp khi chuyển đổi sang JSON.
4.  **Phân tách mối quan tâm:** Domain Model nên tập trung vào logic nghiệp vụ. DTOs tập trung vào định nghĩa "API Contract" – tức là dữ liệu mà API cam kết sẽ cung cấp hoặc nhận từ client. Khi Domain Model thay đổi (ví dụ: thêm một thuộc tính mới), API Contract (DTO) không nhất thiết phải thay đổi, giúp duy trì tính ổn định của API.
5.  **Tối ưu hóa hiệu suất:** Giảm lượng dữ liệu truyền qua mạng bằng cách chỉ gửi những gì cần thiết.

**Data Transfer Object (DTO)** là một mẫu thiết kế đơn giản: một đối tượng thuần túy chỉ chứa dữ liệu, không có logic nghiệp vụ. Nó được sử dụng để truyền dữ liệu giữa các lớp hoặc giữa API và client, đảm bảo rằng chỉ những thông tin cần thiết mới được hiển thị.

> [!TIP]
> **Vibe Coding trong thiết kế DTOs:**
>
> Khi thiết kế DTOs, tư duy Vibe Coding giúp chúng ta đặt mình vào vị trí của client. "Cái vibe" mà client mong muốn khi gọi API này là gì?
> *   Khi lấy danh sách `Walks`, client có cần toàn bộ chi tiết của `Region` hay chỉ `Name` và `Code` của nó?
> *   Có trường nào trong Domain Model là nội bộ hoặc nhạy cảm mà client không nên thấy không?
>
> Bằng cách này, chúng ta định hình DTOs để chúng phản ánh chính xác "vibe" của API Contract, mang lại trải nghiệm tốt nhất cho người dùng API.

#### 1.2. Định nghĩa các DTOs cho `Walk`, `Region`, và `WalkDifficulty`

Chúng ta sẽ tạo các DTOs tương ứng, chỉ bao gồm các thuộc tính cần thiết cho client. Lưu ý rằng `WalkDto` sẽ chứa các DTOs lồng nhau cho `Region` và `WalkDifficulty`, phá vỡ mọi vòng lặp tham chiếu tiềm ẩn và cung cấp một cấu trúc dữ liệu rõ ràng.

```csharp
// NZWalks.API/Models/DTO/WalkDto.cs
using System;

namespace NZWalks.API.Models.DTO
{
    public class WalkDto
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
        public double LengthInKm { get; set; }
        public string? WalkImageUrl { get; set; }

        // DTOs cho các thuộc tính điều hướng (lồng nhau)
        public RegionDto Region { get; set; }
        public WalkDifficultyDto WalkDifficulty { get; set; }
    }
}

// NZWalks.API/Models/DTO/RegionDto.cs
using System;

namespace NZWalks.API.Models.DTO
{
    public class RegionDto
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public string Code { get; set; }
        public string? RegionImageUrl { get; set; }
    }
}

// NZWalks.API/Models/DTO/WalkDifficultyDto.cs
using System;

namespace NZWalks.API.Models.DTO
{
    public class WalkDifficultyDto
    {
        public Guid Id { get; set; }
        public string Code { get; set; }
    }
}
```

### 2. Cấu hình AutoMapper cho ánh xạ dữ liệu

Sau khi định nghĩa DTOs, chúng ta cần một cách để chuyển đổi dữ liệu từ Domain Model sang DTOs (và ngược lại). Thay vì viết mã sao chép thuộc tính thủ công, chúng ta sẽ sử dụng AutoMapper.

#### 2.1. AutoMapper: Tự động hóa ánh xạ đối tượng

**AutoMapper** là một thư viện phổ biến giúp đơn giản hóa việc ánh xạ giữa các đối tượng có cấu trúc tương tự nhưng khác kiểu.
*   **Lợi ích:**
    *   **Giảm Boilerplate Code:** Không cần viết hàng tá dòng mã để sao chép từng thuộc tính.
    *   **Dễ bảo trì:** Khi Domain Model hoặc DTO thay đổi, bạn chỉ cần cập nhật cấu hình ánh xạ thay vì sửa nhiều dòng code sao chép rải rác.
    *   **Tăng năng suất:** Tăng tốc độ phát triển bằng cách tự động hóa một tác vụ lặp đi lặp lại.
*   **Cơ chế hoạt động:** AutoMapper sử dụng reflection để so sánh các thuộc tính có tên tương tự giữa hai kiểu đối tượng và tự động ánh xạ chúng. Bạn có thể tùy chỉnh các quy tắc ánh xạ cho các trường hợp đặc biệt (ví dụ: tên thuộc tính khác nhau, chuyển đổi kiểu).
*   **Khi nào không nên dùng:** Nếu logic ánh xạ quá phức tạp, bao gồm nhiều phép tính hoặc điều kiện, việc viết mã ánh xạ thủ công có thể rõ ràng và dễ kiểm soát hơn. Tuy nhiên, với các trường hợp ánh xạ 1-1 đơn giản, AutoMapper là một công cụ mạnh mẽ.

#### 2.2. Định nghĩa AutoMapper Profile

Để sử dụng AutoMapper, chúng ta tạo một lớp kế thừa từ `Profile` và định nghĩa các quy tắc ánh xạ trong constructor của nó.

```csharp
// NZWalks.API/Mappings/WalksProfile.cs
using AutoMapper;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTO;

namespace NZWalks.API.Mappings
{
    public class WalksProfile : Profile
    {
        public WalksProfile()
        {
            // Ánh xạ từ Domain Model sang DTO
            // .ReverseMap() cho phép ánh xạ ngược lại (từ DTO sang Domain Model), hữu ích cho các thao tác ghi dữ liệu.
            CreateMap<Walk, WalkDto>().ReverseMap();
            CreateMap<WalkDifficulty, WalkDifficultyDto>().ReverseMap();
            CreateMap<Region, RegionDto>().ReverseMap();
            // AutoMapper sẽ tự động ánh xạ các thuộc tính lồng nhau (ví dụ: Walk.Region sang WalkDto.Region)
            // miễn là các profile cho Region và WalkDifficulty cũng đã được định nghĩa.
        }
    }
}

// NZWalks.API/Mappings/AutoMapperProfiles.cs (để gom tất cả profiles)
// Lớp này chỉ dùng để định danh assembly chứa các AutoMapper profiles
// và được truyền vào builder.Services.AddAutoMapper(typeof(AutoMapperProfiles));
namespace NZWalks.API.Mappings
{
    public class AutoMapperProfiles : Profile
    {
        public AutoMapperProfiles()
        {
            // Các profile cụ thể sẽ được định nghĩa ở các file riêng biệt như WalksProfile, RegionsProfile
            // AutoMapper sẽ tự động quét và đăng ký tất cả các Profile trong assembly này.
        }
    }
}
```

> [!TIP]
> **Antigravity IDE và AutoMapper:**
>
> Antigravity IDE có thể là một trợ thủ đắc lực trong việc cấu hình AutoMapper:
> *   **Tự động tạo Profile:** "Antigravity, tạo một AutoMapper Profile để ánh xạ `Walk` Domain Model sang `WalkDto` và bao gồm `ReverseMap()`."
> *   **Gợi ý ánh xạ phức tạp:** Nếu có các thuộc tính tên khác nhau hoặc cần logic chuyển đổi đặc biệt, Antigravity có thể gợi ý các phương thức `ForMember` để tùy chỉnh ánh xạ.
> *   **Đảm bảo đăng ký:** "Antigravity, đảm bảo tất cả các AutoMapper Profiles trong dự án được đăng ký chính xác trong `Program.cs`." Antigravity sẽ kiểm tra và thêm dòng `builder.Services.AddAutoMapper(typeof(AutoMapperProfiles));` nếu thiếu.

### 3. Xây dựng WalksController

Controller là nơi API tiếp nhận các yêu cầu HTTP, điều phối các hoạt động nghiệp vụ và trả về phản hồi. `WalksController` sẽ chứa các endpoint cho thực thể `Walk`.

#### 3.1. Vai trò của Controller trong RESTful API

*   **Tiếp nhận yêu cầu:** Lắng nghe các yêu cầu HTTP đến các URL cụ thể, được định nghĩa bằng thuộc tính `[Route]` và các HTTP Verbs (như `[HttpGet]`, `[HttpPost]`).
*   **Xử lý yêu cầu:** Điều phối các hoạt động, thường là gọi các dịch vụ (như Repository) để thực hiện logic nghiệp vụ hoặc truy vấn dữ liệu.
*   **Trả về phản hồi:** Định dạng dữ liệu thành phản hồi HTTP (ví dụ: JSON) và trả về cho client cùng với mã trạng thái HTTP thích hợp (ví dụ: `200 OK`, `404 Not Found`, `201 Created`).
*   `[ApiController]`: Thuộc tính này tự động thêm các tính năng hữu ích như kiểm tra ràng buộc mô hình, suy luận nguồn tham số (inferring parameter binding sources), và tự động trả về lỗi `400 Bad Request` cho các lỗi validation.
*   `[Route("api/[controller]")]`: Định nghĩa route cơ bản cho controller. `[controller]` là một placeholder sẽ được thay thế bằng tên của controller (ví dụ: `Walks` cho `WalksController`), tạo ra route `/api/walks`.

Chúng ta sẽ sử dụng Constructor Injection để đưa `IWalkRepository` (để truy cập dữ liệu) và `IMapper` (để ánh xạ Domain Model sang DTO) vào `WalksController`.

```csharp
// NZWalks.API/Controllers/WalksController.cs
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")] // Định nghĩa route cơ bản: /api/walks
    public class WalksController : ControllerBase
    {
        private readonly IWalkRepository _walkRepository;
        private readonly IMapper _mapper;

        public WalksController(IWalkRepository walkRepository, IMapper mapper)
        {
            _walkRepository = walkRepository;
            _mapper = mapper;
        }

        // Endpoint GET /api/walks
        [HttpGet]
        public async Task<IActionResult> GetAllWalks()
        {
            // Logic sẽ được thêm vào sau
            return Ok();
        }

        // ... các endpoint khác
    }
}
```

#### 3.2. Triển khai Endpoint `GetAllWalks`

Bây giờ chúng ta có thể hoàn thiện phương thức `GetAllWalks` trong `WalksController` để lấy dữ liệu từ Repository, ánh xạ sang DTO và trả về.

```csharp
// NZWalks.API/Controllers/WalksController.cs (đã cập nhật)
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class WalksController : ControllerBase
    {
        private readonly IWalkRepository _walkRepository;
        private readonly IMapper _mapper;

        public WalksController(IWalkRepository walkRepository, IMapper mapper)
        {
            _walkRepository = walkRepository;
            _mapper = mapper;
        }

        // Endpoint GET /api/walks
        [HttpGet]
        public async Task<IActionResult> GetAllWalks()
        {
            // 1. Lấy tất cả các hành trình từ Repository (Domain Models)
            var walksDomain = await _walkRepository.GetAllAsync();

            // 2. Ánh xạ danh sách Domain Models sang danh sách DTOs bằng AutoMapper
            var walksDto = _mapper.Map<List<WalkDto>>(walksDomain);

            // 3. Trả về kết quả 200 OK với danh sách DTOs
            return Ok(walksDto);
        }

        // ... các endpoint khác
    }
}
```

### 4. Tối ưu hóa truy vấn với Navigation Properties (Eager Loading)

Khi chúng ta gọi `_dbContext.Walks.ToListAsync()` trong `WalkRepository`, Entity Framework Core theo mặc định sẽ chỉ tải các thuộc tính của chính thực thể `Walk`. Các thuộc tính điều hướng như `Region` và `WalkDifficulty` sẽ không được tải cùng lúc, dẫn đến việc chúng có giá trị `null` trong Domain Model và sau đó trong DTO trả về. Để khắc phục điều này và đảm bảo dữ liệu đầy đủ, chúng ta cần sử dụng kỹ thuật "Eager Loading" thông qua phương thức `.Include()`.

#### 4.1. Hiểu sâu về Navigation Properties và Các chiến lược tải dữ liệu liên quan trong EF Core

Navigation Properties cho phép bạn điều hướng qua các mối quan hệ giữa các thực thể trong Domain Model của bạn. Ví dụ, từ một `Walk`, bạn có thể truy cập `Region` liên quan của nó thông qua `walk.Region`.

**Các chiến lược tải dữ liệu liên quan trong EF Core:**

1.  **Lazy Loading (Tải lười biếng):**
    *   **Cơ chế:** Dữ liệu liên quan được tải tự động từ cơ sở dữ liệu *khi* thuộc tính điều hướng được truy cập lần đầu.
    *   **Ưu điểm:** Chỉ tải những gì thực sự cần thiết, tránh tải quá nhiều dữ liệu không dùng đến.
    *   **Nhược điểm:**
        *   Có thể dẫn đến vấn đề **"N+1 query problem"**: Nếu bạn có một danh sách N đối tượng `Walk` và truy cập `walk.Region` trong một vòng lặp, EF Core sẽ thực hiện N truy vấn riêng biệt cho `Region` (ngoài truy vấn ban đầu cho `Walks`), dẫn đến hiệu suất rất kém.
        *   Yêu cầu các thuộc tính điều hướng phải là `virtual` và cài đặt gói `Microsoft.EntityFrameworkCore.Proxies`.
    *   **Ví dụ (có thể gây N+1):**
        ```csharp
        var walks = _dbContext.Walks.ToList(); // Chỉ tải Walks
        foreach (var walk in walks)
        {
            Console.WriteLine(walk.Region.Name); // Mỗi lần truy cập walk.Region sẽ gây ra một query mới
        }
        ```

2.  **Eager Loading (Tải sớm):**
    *   **Cơ chế:** Dữ liệu liên quan được tải cùng lúc với thực thể chính trong *một truy vấn duy nhất* (thường là một JOIN SQL).
    *   **Ưu điểm:**
        *   Hiệu quả hơn khi bạn biết chắc mình sẽ cần dữ liệu liên quan.
        *   Tránh vấn đề N+1 query.
        *   Không yêu cầu `virtual` properties hay cài đặt proxy.
    *   **Nhược điểm:** Có thể tải quá nhiều dữ liệu nếu bạn không cần tất cả các mối quan hệ (dẫn đến các JOIN không cần thiết và dữ liệu thừa).
    *   **Cách thực hiện:** Sử dụng phương thức `.Include()` và `.ThenInclude()` trên `IQueryable`.
    *   **Ví dụ:**
        ```csharp
        var walks = _dbContext.Walks
                               .Include(x => x.Region)
                               .Include(x => x.WalkDifficulty)
                               .ToList(); // Tải Walks, Region và WalkDifficulty trong MỘT truy vấn
        ```

3.  **Explicit Loading (Tải rõ ràng):**
    *   **Cơ chế:** Dữ liệu liên quan được tải rõ ràng *sau khi* thực thể chính đã được tải, sử dụng các phương thức `Load()` hoặc `Collection()`/`Reference()` của `Entry`.
    *   **Ưu điểm:** Kiểm soát chính xác thời điểm tải dữ liệu liên quan, hữu ích trong các kịch bản đặc biệt khi bạn cần tải động dựa trên logic nghiệp vụ.
    *   **Nhược điểm:** Yêu cầu nhiều mã hơn, có thể tạo ra nhiều truy vấn nếu không cẩn thận.

**Trong trường hợp của chúng ta**, khi trả về `WalkDto`, chúng ta muốn hiển thị thông tin chi tiết về `Region` và `WalkDifficulty` cùng với `Walk`. Do đó, **Eager Loading** là lựa chọn phù hợp nhất vì nó hiệu quả và đảm bảo dữ liệu đầy đủ trong một truy vấn duy nhất.

> [!TIP]
> **Vibe Coding & Antigravity IDE trong Eager Loading:**
>
> Vibe Coding khuyến khích chúng ta suy nghĩ về "vibe" của dữ liệu mà API trả về. Một `WalkDto` mà có `Region` và `WalkDifficulty` là `null` sẽ có "vibe" không hoàn chỉnh.
>
> Antigravity IDE có thể giúp chúng ta hiện thực hóa "vibe" của dữ liệu đầy đủ:
> *   "Antigravity, cập nhật phương thức `GetAllAsync` trong `WalkRepository` để bao gồm các thuộc tính điều hướng `Region` và `WalkDifficulty`."
> *   Antigravity sẽ phân tích `Walk` Domain Model, nhận diện các thuộc tính điều hướng, và tự động thêm `.Include(x => x.Region).Include(x => x.WalkDifficulty)` vào truy vấn, đảm bảo rằng dữ liệu liên quan được tải đúng cách. Nó có thể thậm chí cảnh báo nếu DTO của bạn yêu cầu dữ liệu mà Repository chưa tải, giúp bạn duy trì "vibe" của API hoàn chỉnh.

#### 4.2. Cập nhật Repository với `.Include()` để tải dữ liệu liên quan

Chúng ta sẽ cập nhật phương thức `GetAllAsync` (và sau này là `GetByIdAsync`) trong `WalkRepository` để bao gồm `Region` và `WalkDifficulty` khi truy vấn các `Walk` từ cơ sở dữ liệu.

```csharp
// NZWalks.API/Repositories/WalkRepository.cs (đã cập nhật với .Include())
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public class WalkRepository : IWalkRepository
    {
        private readonly NZWalksDbContext _dbContext;

        public WalkRepository(NZWalksDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task<IEnumerable<Walk>> GetAllAsync()
        {
            // Sử dụng .Include() để tải dữ liệu liên quan của Region và WalkDifficulty
            // EF Core sẽ tạo một truy vấn SQL với JOIN để lấy tất cả dữ liệu này trong một lần.
            return await _dbContext.Walks
                .Include(x => x.Region)         // Tải thông tin Region liên quan
                .Include(x => x.WalkDifficulty) // Tải thông tin WalkDifficulty liên quan
                .ToListAsync();
        }

        public async Task<Walk?> GetByIdAsync(Guid id)
        {
            // Logic lấy hành trình theo ID cũng sẽ được cập nhật tương tự
            // để đảm bảo Region và WalkDifficulty được tải cùng lúc.
            return await _dbContext.Walks
                .Include(x => x.Region)
                .Include(x => x.WalkDifficulty)
                .FirstOrDefaultAsync(x => x.Id == id);
        }
    }
}
```

Với thay đổi này, khi `GetAllWalks` được gọi, `Region` và `WalkDifficulty` của mỗi `Walk` sẽ được tải cùng lúc trong một truy vấn duy nhất, và AutoMapper sẽ có thể ánh xạ chúng sang `RegionDto` và `WalkDifficultyDto` tương ứng trong `WalkDto`.

### 5. Triển khai Endpoint `GetWalkById`

Tương tự như `GetAllWalks`, chúng ta sẽ triển khai chức năng lấy một hành trình cụ thể dựa trên ID của nó.

#### 5.1. Cập nhật phương thức `GetByIdAsync` trong Repository

Phương thức `GetByIdAsync` trong `WalkRepository` sẽ truy vấn cơ sở dữ liệu để tìm một `Walk` duy nhất có ID khớp. Chúng ta đã cập nhật nó với `.Include()` ở bước trên để đảm bảo dữ liệu liên quan được tải.

#### 5.2. Triển khai Endpoint `GetWalkById` trong Controller

Endpoint này sẽ nhận một ID từ URL, gọi Repository để tìm hành trình, xử lý trường hợp không tìm thấy, và trả về DTO tương ứng.

```csharp
// NZWalks.API/Controllers/WalksController.cs (đã cập nhật)
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class WalksController : ControllerBase
    {
        private readonly IWalkRepository _walkRepository;
        private readonly IMapper _mapper;

        public WalksController(IWalkRepository walkRepository, IMapper mapper)
        {
            _walkRepository = walkRepository;
            _mapper = mapper;
        }

        // Endpoint GET /api/walks
        [HttpGet]
        public async Task<IActionResult> GetAllWalks()
        {
            var walksDomain = await _walkRepository.GetAllAsync();
            var walksDto = _mapper.Map<List<WalkDto>>(walksDomain);
            return Ok(walksDto);
        }

        // Endpoint GET /api/walks/{id}
        // id:guid chỉ định rằng tham số id phải là kiểu GUID, giúp định tuyến chính xác và validation tự động.
        [HttpGet("{id:guid}")]
        public async Task<IActionResult> GetWalkById([FromRoute] Guid id)
        {
            // 1. Lấy hành trình từ Repository (Domain Model)
            var walkDomain = await _walkRepository.GetByIdAsync(id);

            // 2. Xử lý trường hợp không tìm thấy
            if (walkDomain == null)
            {
                return NotFound(); // Trả về 404 Not Found
            }

            // 3. Ánh xạ Domain Model sang DTO
            var walkDto = _mapper.Map<WalkDto>(walkDomain);

            // 4. Trả về kết quả 200 OK với DTO
            return Ok(walkDto);
        }
    }
}
```

## III. Kiểm thử API bằng Swagger UI và tư duy Vibe Testing

Sau khi triển khai các endpoint, việc kiểm thử là bước quan trọng để xác nhận chúng hoạt động đúng như mong đợi. Swagger UI là một công cụ tuyệt vời được tích hợp sẵn trong ASP.NET Core để khám phá và kiểm thử API của bạn. Đồng thời, chúng ta sẽ áp dụng "Vibe Testing" – không chỉ kiểm tra chức năng, mà còn cảm nhận "vibe" của phản hồi API. Liệu phản hồi có dễ hiểu, đầy đủ, và đúng như mong đợi của client không?

### 1. Kiểm thử `GetAllWalks`

1.  **Chạy ứng dụng:** Khởi động ứng dụng của bạn (thường bằng cách nhấn F5 hoặc `dotnet run`).
2.  **Truy cập Swagger UI:** Mở trình duyệt và điều hướng đến URL của Swagger UI (thường là `https://localhost:PORT/swagger`).
3.  **Tìm endpoint:** Trong danh sách các API, bạn sẽ thấy mục `Walks` và một phương thức `GET /api/Walks`.
4.  **Thực hiện yêu cầu:**
    *   Nhấp vào `GET /api/Walks`.
    *   Nhấp vào nút `Try it out`.
    *   Nhấp vào nút `Execute`.
5.  **Phân tích kết quả (Vibe Testing):**
    *   Bạn sẽ nhận được phản hồi HTTP `200 OK`. Đây là "vibe" của thành công.
    *   Trong phần "Response body", bạn sẽ thấy một mảng JSON chứa danh sách các hành trình.
    *   **Kiểm tra "vibe" dữ liệu:** Quan trọng nhất, hãy kiểm tra từng đối tượng `Walk` để đảm bảo rằng các thuộc tính `Region` và `WalkDifficulty` không còn là `null` mà chứa thông tin chi tiết về khu vực và độ khó tương ứng. Nếu chúng đầy đủ, "vibe" của dữ liệu là hoàn chỉnh và hữu ích cho client, xác nhận rằng `.Include()` đã hoạt động hiệu quả. Nếu chúng `null`, "vibe" của API là thiếu sót, cần điều chỉnh lại.

### 2. Kiểm thử `GetWalkById`

1.  **Truy cập Swagger UI:** Tương tự như trên.
2.  **Tìm endpoint:** Trong mục `Walks`, bạn sẽ thấy phương thức `GET /api/Walks/{id}`.
3.  **Thực hiện yêu cầu (trường hợp thành công):**
    *   Để có một `Id` hợp lệ, bạn có thể lấy từ kết quả của `GetAllWalks` hoặc từ cơ sở dữ liệu của bạn.
    *   Nhấp vào `GET /api/Walks/{id}`.
    *   Nhấp vào nút `Try it out`.
    *   Nhập `Id` bạn đã chọn vào trường `id`.
    *   Nhấp vào nút `Execute`.
    *   **Phân tích kết quả (Vibe Testing):** Bạn sẽ nhận được phản hồi HTTP `200 OK` và một đối tượng JSON chứa thông tin chi tiết của hành trình đó, bao gồm `Region` và `WalkDifficulty` đầy đủ. Đây là "vibe" của việc cung cấp chính xác thông tin chi tiết mà client yêu cầu.
4.  **Thực hiện yêu cầu (trường hợp không tìm thấy):**
    *   Nhập một `Id` không tồn tại (ví dụ: `00000000-0000-0000-0000-000000000000`).
    *   Nhấp vào `Execute`.
    *   **Phân tích kết quả (Vibe Testing):** Bạn sẽ nhận được phản hồi HTTP `404 Not Found`. Đây là "vibe" của việc xử lý lỗi một cách rõ ràng và chuyên nghiệp, thông báo cho client rằng tài nguyên không tồn tại.

> [!TIP]
> **Antigravity IDE và Tự động hóa kiểm thử (Vibe Testing):**
>
> Antigravity IDE có khả năng tự chạy các script ngầm và gọi subagent trình duyệt, khiến nó trở thành một công cụ mạnh mẽ cho việc tự động hóa và "Vibe Testing":
> *   **Tự động tạo request:** "Antigravity, tạo một request GET đến `/api/walks` và hiển thị JSON response."
> *   **Kiểm tra nội dung phản hồi:** "Antigravity, sau khi gọi `/api/walks`, hãy kiểm tra xem mỗi `WalkDto` trong danh sách có thuộc tính `Region.Name` và `WalkDifficulty.Code` không phải là null không." Antigravity có thể viết và chạy một script kiểm tra JSON response, xác nhận "vibe" của dữ liệu đầy đủ.
> *   **Kiểm tra trường hợp lỗi:** "Antigravity, gửi một request GET đến `/api/walks/{invalid_guid}` và xác nhận rằng HTTP status code là 404."
>
> Bằng cách này, Antigravity không chỉ giúp bạn thực hiện các bước kiểm thử thủ công nhanh hơn mà còn có thể tự động xác nhận "vibe" mong muốn của API dựa trên các yêu cầu của bạn, giải phóng bạn để tập trung vào các khía cạnh thiết kế và nghiệp vụ phức tạp hơn.

## Tóm tắt Phần

Trong phần này, chúng ta đã xây dựng một nền tảng vững chắc cho API Hành trình (Walks) của mình, bao gồm các kiến thức và kỹ thuật quan trọng sau:

*   **Domain Model `Walk`:** Đã định nghĩa thực thể `Walk` với các thuộc tính cơ bản và đặc biệt là các **Navigation Properties** để biểu diễn mối quan hệ với `Region` và `WalkDifficulty`.
*   **Repository Pattern:** Đã định nghĩa và triển khai `IWalkRepository` và `WalkRepository` để trừu tượng hóa lớp truy cập dữ liệu, giúp mã nguồn sạch hơn, dễ kiểm thử và dễ bảo trì.
*   **Dependency Injection (DI):** Đã đăng ký `IWalkRepository` vào DI container của ASP.NET Core với `AddScoped`, cho phép các Controller sử dụng Repository mà không cần quan tâm đến cách nó được tạo ra. Chúng ta cũng đã hiểu sâu hơn về các lifetime của dịch vụ (`AddScoped`, `AddTransient`, `AddSingleton`) và lý do chọn `AddScoped` cho Repository và `DbContext`.
*   **Data Transfer Objects (DTOs):** Đã định nghĩa `WalkDto`, `RegionDto`, và `WalkDifficultyDto` để tách biệt Domain Model khỏi API Contract, kiểm soát dữ liệu được hiển thị cho client, tránh các vấn đề như over-fetching và vòng lặp tham chiếu.
*   **AutoMapper:** Đã sử dụng AutoMapper để tự động hóa quá trình ánh xạ giữa Domain Model và DTO, giảm thiểu boilerplate code và tăng năng suất.
*   **Controllers và HTTP Verbs:** Đã tạo `WalksController` với các thuộc tính `[ApiController]` và `[Route]`, và triển khai các endpoint `GET` (`GetAllWalks`, `GetWalkById`) để lấy dữ liệu.
*   **Entity Framework Core Navigation Properties và Eager Loading:** Đã hiểu sâu về Navigation Properties và ba chiến lược tải dữ liệu liên quan (Lazy, Eager, Explicit Loading). Chúng ta đã áp dụng phương thức `.Include()` trong Repository để thực hiện Eager Loading, tải dữ liệu `Region` và `WalkDifficulty` cùng lúc với thực thể `Walk` trong một truy vấn duy nhất, tránh vấn đề N+1 query và đảm bảo dữ liệu đầy đủ trong phản hồi API.
*   **Kiểm thử API và Vibe Testing:** Đã sử dụng Swagger UI để kiểm thử các endpoint `GetAllWalks` và `GetWalkById`, xác nhận hoạt động chính xác và phản hồi dữ liệu đầy đủ. Đồng thời, chúng ta đã áp dụng tư duy "Vibe Testing" để đánh giá chất lượng và sự hoàn chỉnh của phản hồi API từ góc độ của client.
*   **Antigravity IDE và Vibe Coding:** Xuyên suốt phần này, chúng ta đã liên hệ cách các nguyên tắc của Vibe Coding có thể được áp dụng trong thiết kế kiến trúc và cách một hệ thống AI Agentic như Antigravity IDE có thể hỗ trợ tự động hóa, gợi ý thông minh và kiểm thử, giúp hiện thực hóa "vibe" của một hệ thống chất lượng cao.

Những kiến thức này là nền tảng quan trọng để tiếp tục phát triển các chức năng ghi dữ liệu (thêm, cập nhật, xóa) cho API Hành trình trong các phần tiếp theo, nơi chúng ta sẽ tiếp tục áp dụng và mở rộng các nguyên tắc thiết kế tốt nhất.

<!-- REVIEWED_BY_AGENT -->
