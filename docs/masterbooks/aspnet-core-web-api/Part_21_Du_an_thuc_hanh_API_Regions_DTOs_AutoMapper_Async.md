# Phần 21: Dự án thực hành: API Vùng (Regions) - DTOs, AutoMapper & Async/Await

Trong hành trình xây dựng RESTful Web API với ASP.NET Core và Entity Framework Core, việc tạo ra một API không chỉ hoạt động mà còn tuân thủ các nguyên tắc thiết kế tốt là yếu tố then chốt. Phần này sẽ đưa dự án API Vùng (Regions) của chúng ta lên một tầm cao mới bằng cách giới thiệu ba kỹ thuật cốt lõi: **Data Transfer Objects (DTOs)**, thư viện **AutoMapper**, và **Lập trình Bất đồng bộ (Async/Await)**.

Mục tiêu chính là nâng cấp API hiện có để đảm bảo tính tách biệt, khả năng bảo trì và hiệu suất vượt trội. Chúng ta sẽ khám phá cách:

*   **DTOs** giúp định hình dữ liệu giao tiếp với client, bảo vệ mô hình miền và tạo ra một "hợp đồng" API ổn định.
*   **AutoMapper** tự động hóa quá trình ánh xạ đối tượng, loại bỏ đáng kể code lặp lại (boilerplate) và tăng tốc độ phát triển.
*   **Async/Await** tối ưu hóa hiệu suất và khả năng mở rộng của ứng dụng bằng cách cho phép các thao tác I/O (như truy vấn cơ sở dữ liệu) diễn ra bất đồng bộ, giải phóng tài nguyên hệ thống.

Xuyên suốt phần này, chúng ta sẽ tiếp tục củng cố sự hiểu biết và ứng dụng các khái niệm nền tảng như Dependency Injection, Repository Pattern, Controllers và HTTP Verbs đã được xây dựng trong các phần trước.

---

## 1. Data Transfer Objects (DTOs): Định hình Hợp đồng API và Tách biệt Mô hình Miền

Khi thiết kế một API, cách dữ liệu được trình bày cho client là một quyết định kiến trúc quan trọng. Việc để lộ trực tiếp các mô hình miền (Domain Models) của ứng dụng thường dẫn đến nhiều vấn đề.

### 1.1. Vấn đề khi để lộ Mô hình Miền trực tiếp ra API

Mô hình miền (Domain Model), ví dụ như lớp `Region` của chúng ta, là những đối tượng cốt lõi đại diện cho các thực thể và logic nghiệp vụ bên trong ứng dụng. Chúng được thiết kế để phục vụ cho các quy tắc kinh doanh nội bộ, thường chứa tất cả các thuộc tính, quan hệ và hành vi cần thiết cho hoạt động của hệ thống.

Việc trả về trực tiếp các đối tượng mô hình miền từ API của bạn có thể gây ra những hệ quả nghiêm trọng:

*   **Thay đổi gây phá vỡ (Breaking Changes):** Mô hình miền thường xuyên phát triển cùng với logic nghiệp vụ. Nếu bạn thay đổi tên một thuộc tính, thêm một thuộc tính mới, hoặc xóa một thuộc tính trong `Region` (ví dụ: thêm trường `LastModifiedBy`, đổi tên `Code` thành `RegionCode`), bất kỳ client nào đang sử dụng API của bạn sẽ bị ảnh hưởng ngay lập tức và cần phải cập nhật theo. Điều này làm giảm tính ổn định của API và gây khó khăn cho việc duy trì các phiên bản API. Các client mong đợi một "hợp đồng" API ổn định và chỉ chấp nhận các thay đổi gây phá vỡ khi có phiên bản API mới được quản lý rõ ràng.
*   **Phơi bày dữ liệu không cần thiết hoặc nhạy cảm:** Mô hình miền có thể chứa các thuộc tính nội bộ (ví dụ: `CreatedDate`, `InternalNotes`), thông tin nhạy cảm (ví dụ: mật khẩu băm của người dùng liên quan, thông tin cấu hình hệ thống), hoặc các thuộc tính điều hướng (Navigation Properties) mà client không cần hoặc không nên truy cập trực tiếp (ví dụ: danh sách `Walks` liên quan đến `Region`). Việc trả về tất cả thông tin này là không cần thiết, tăng kích thước payload không cần thiết và tiềm ẩn rủi ro bảo mật.
*   **Thiếu tách biệt mối quan tâm (Lack of Separation of Concerns):** Mô hình miền được thiết kế để phục vụ logic nghiệp vụ của ứng dụng. Giao diện API (API contract) nên được thiết kế để phục vụ nhu cầu của client. Việc trộn lẫn hai vai trò này làm giảm tính linh hoạt, khả năng bảo trì và tái sử dụng của cả hai phần. Ví dụ, bạn có thể muốn trả về một tên thuộc tính khác cho client (`regionName` thay vì `Name`) hoặc kết hợp dữ liệu từ nhiều mô hình miền khác nhau thành một phản hồi duy nhất.

> [!NOTE]
> Mục tiêu cốt lõi của DTOs là tạo ra một "hợp đồng" (contract) ổn định và rõ ràng giữa API và các client. Hợp đồng này cho phép mô hình miền phát triển độc lập, thay đổi cấu trúc nội bộ mà không làm ảnh hưởng đến giao diện công khai của API, miễn là DTO không thay đổi.

### 1.2. Data Transfer Objects (DTOs) là gì và tại sao cần chúng?

**Data Transfer Object (DTO)** là một mô hình đơn giản, thuần túy chứa dữ liệu, được thiết kế đặc biệt để truyền dữ liệu giữa các lớp trong ứng dụng hoặc giữa các hệ thống (ví dụ: giữa API và client). Trong ngữ cảnh của RESTful API, DTO là định dạng dữ liệu mà API sẽ gửi đi (response DTO) hoặc nhận về (request DTO) từ client.

**Mục đích và lợi ích chính của DTOs:**

*   **Tách biệt (Decoupling):** DTOs tách rời mô hình miền khỏi giao diện API công khai. Mọi thay đổi trong mô hình miền sẽ không tự động ảnh hưởng đến client, miễn là DTO vẫn giữ nguyên.
*   **Định hình dữ liệu (Data Shaping):** Cho phép bạn tùy chỉnh chính xác các thuộc tính mà client cần. Bạn có thể:
    *   Chọn lọc các thuộc tính cần thiết (ví dụ: chỉ trả về `Id`, `Name`, `Code`, không trả về `Walks`).
    *   Đổi tên thuộc tính để phù hợp hơn với client (ví dụ: `RegionCode` thay vì `Code`).
    *   Tổng hợp dữ liệu từ nhiều mô hình miền khác nhau vào một DTO duy nhất.
    *   Ẩn đi các chi tiết triển khai nội bộ hoặc thông tin nhạy cảm.
*   **Bảo mật (Security):** Bằng cách chỉ phơi bày những dữ liệu mà client cần, bạn giảm thiểu bề mặt tấn công và nguy cơ rò rỉ thông tin nhạy cảm.
*   **Tính ổn định (Stability):** Client tương tác với DTO. Điều này tạo ra một hợp đồng API ổn định, giảm thiểu các thay đổi gây phá vỡ và giúp việc quản lý phiên bản API dễ dàng hơn.
*   **Giảm kích thước payload:** Chỉ gửi những dữ liệu cần thiết giúp giảm kích thước phản hồi, cải thiện hiệu suất mạng.

### 1.3. Triển khai DTOs trong ASP.NET Core

Để triển khai DTOs, chúng ta sẽ tạo một cấu trúc thư mục rõ ràng và định nghĩa các lớp DTO tương ứng.

#### 1.3.1. Cấu trúc thư mục

Trong dự án của bạn, hãy tạo một thư mục mới để chứa các DTO. Thông thường, nó được đặt trong `Models` hoặc một thư mục riêng biệt như `DTOs`. Chúng ta sẽ sử dụng `Models/DTOs` để nhóm chúng với các mô hình khác.

```
NZWalks.API/
├── Controllers/
├── Data/
├── Models/
│   ├── Domain/       // Chứa Domain Models (ví dụ: Region.cs)
│   └── DTOs/         // Chứa Data Transfer Objects (ví dụ: RegionDto.cs)
├── Profiles/
├── Repositories/
└── Program.cs
```

#### 1.3.2. Định nghĩa `RegionDto`

Tạo một lớp mới `RegionDto.cs` bên trong thư mục `Models/DTOs`. Lớp này sẽ chứa các thuộc tính mà chúng ta muốn phơi bày ra bên ngoài cho client.

**`Models/DTOs/RegionDto.cs`**
```csharp
namespace NZWalks.API.Models.DTOs
{
    // RegionDto là mô hình dữ liệu sẽ được trả về cho client.
    // Nó chỉ chứa các thuộc tính cần thiết và an toàn để phơi bày.
    public class RegionDto
    {
        public Guid Id { get; set; }
        public string Code { get; set; }
        public string Name { get; set; }
        public string? RegionImageUrl { get; set; } // Sử dụng '?' cho thuộc tính có thể null
    }
}
```

Để so sánh, đây là mô hình miền `Region` giả định của chúng ta:
**`Models/Domain/Region.cs`**
```csharp
namespace NZWalks.API.Models.Domain
{
    public class Region
    {
        public Guid Id { get; set; }
        public string Code { get; set; }
        public string Name { get; set; }
        public string? RegionImageUrl { get; set; }

        // Navigation Property: Đây là thuộc tính nội bộ của Domain Model,
        // thường không nên phơi bày trực tiếp ra API để tránh vòng lặp tham chiếu
        // và dữ liệu không cần thiết.
        public IEnumerable<Walk> Walks { get; set; }
    }
}
```
Bạn có thể thấy rõ ràng rằng `RegionDto` đã loại bỏ thuộc tính `Walks`, chỉ giữ lại những thông tin cốt lõi mà client cần khi truy vấn danh sách các vùng.

#### 1.3.3. Áp dụng DTOs trong Controller (Ánh xạ Thủ công)

Ban đầu, để hiểu rõ cơ chế, chúng ta sẽ thực hiện việc chuyển đổi từ `Region` (mô hình miền) sang `RegionDto` (mô hình giao tiếp) một cách thủ công trong controller.

**`Controllers/RegionsController.cs` (trước khi dùng AutoMapper)**
```csharp
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTOs;
using NZWalks.API.Repositories;
using System.Collections.Generic; // Thêm namespace này cho List

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository regionRepository;

        // Dependency Injection: Controller nhận IRegionRepository qua constructor
        public RegionsController(IRegionRepository regionRepository)
        {
            this.regionRepository = regionRepository;
        }

        // GET ALL REGIONS (Mapping thủ công từ Domain Model sang DTO)
        [HttpGet]
        public IActionResult GetAllRegions()
        {
            // 1. Lấy Domain Models từ Repository
            var regionsDomain = regionRepository.GetAll();

            // 2. Chuyển đổi (Mapping) thủ công Domain Models sang DTOs
            var regionsDto = new List<RegionDto>();
            foreach (var regionDomain in regionsDomain)
            {
                regionsDto.Add(new RegionDto
                {
                    Id = regionDomain.Id,
                    Code = regionDomain.Code,
                    Name = regionDomain.Name,
                    RegionImageUrl = regionDomain.RegionImageUrl
                });
            }

            // 3. Trả về DTOs qua Ok (HTTP 200 OK)
            return Ok(regionsDto);
        }
    }
}
```
Sau khi chạy ứng dụng và kiểm tra API qua Swagger, bạn sẽ thấy dữ liệu trả về đã được định hình bởi `RegionDto`. Mặc dù cách này giải quyết được vấn đề tách biệt, nhưng việc ánh xạ thủ công rất dài dòng, lặp lại và dễ gây lỗi khi có nhiều thuộc tính hoặc nhiều DTO. Đây chính là lúc AutoMapper phát huy tác dụng.

---

## 2. AutoMapper: Tự động hóa Ánh xạ Đối tượng hiệu quả

Như đã thấy ở trên, việc chuyển đổi dữ liệu giữa các lớp (ví dụ: từ `Region` sang `RegionDto`) là một công việc lặp đi lặp lại (boilerplate code) và tốn thời gian. Khi số lượng mô hình và DTO tăng lên, việc quản lý các ánh xạ thủ công trở nên không khả thi. AutoMapper ra đời để giải quyết vấn đề này.

### 2.1. Hạn chế của Ánh xạ Thủ công

Việc ánh xạ thủ công có những nhược điểm rõ rệt:

*   **Code lặp lại (Boilerplate Code):** Bạn phải viết code chuyển đổi cho từng thuộc tính của từng cặp đối tượng. Điều này làm phình to code, đặc biệt trong các controller, làm giảm tính dễ đọc và tập trung vào logic nghiệp vụ chính.
*   **Dễ gây lỗi (Error-prone):** Rất dễ quên ánh xạ một thuộc tính, ánh xạ sai kiểu dữ liệu, hoặc gán nhầm thuộc tính, dẫn đến lỗi runtime hoặc dữ liệu không chính xác.
*   **Khó bảo trì (Hard to Maintain):** Khi mô hình miền hoặc DTO thay đổi (ví dụ: thêm, xóa, đổi tên thuộc tính), bạn phải tìm và cập nhật tất cả các đoạn code ánh xạ thủ công liên quan. Điều này tốn thời gian và dễ bỏ sót, gây ra lỗi tiềm ẩn.
*   **Giảm hiệu suất phát triển:** Thời gian dành cho việc viết và kiểm tra code ánh xạ có thể được dùng cho các tính năng quan trọng hơn.

### 2.2. AutoMapper là gì?

**AutoMapper** là một thư viện mã nguồn mở phổ biến trong .NET, được thiết kế để giải quyết vấn đề ánh xạ đối tượng-đối tượng một cách hiệu quả. Nó hoạt động dựa trên nguyên tắc "quy ước qua cấu hình" (convention over configuration), nghĩa là nó sẽ tự động ánh xạ các thuộc tính có tên và kiểu dữ liệu giống nhau giữa đối tượng nguồn (source) và đối tượng đích (destination). Bạn chỉ cần cấu hình các quy tắc ánh xạ cho những trường hợp đặc biệt hoặc khi tên/kiểu dữ liệu không khớp.

**Cách AutoMapper hoạt động (Under the Hood):**
Khi bạn cấu hình một ánh xạ (ví dụ: `CreateMap<Source, Destination>()`), AutoMapper không thực hiện ánh xạ ngay lập tức. Thay vào đó, nó xây dựng một biểu đồ biểu thức (expression tree) và sau đó biên dịch biểu đồ này thành một hàm ánh xạ hiệu quả. Hàm này sau đó được lưu vào bộ nhớ đệm và tái sử dụng cho các lần ánh xạ sau, mang lại hiệu suất gần như ánh xạ thủ công nhưng với code tối giản hơn rất nhiều.

**Lợi ích của AutoMapper:**

*   **Giảm code boilerplate:** Tự động hóa hầu hết các tác vụ ánh xạ, giúp code sạch hơn, gọn gàng hơn và dễ đọc hơn.
*   **Tăng hiệu suất phát triển:** Tiết kiệm thời gian đáng kể trong việc viết code ánh xạ, cho phép tập trung vào logic nghiệp vụ.
*   **Dễ bảo trì:** Các quy tắc ánh xạ được tập trung trong các Profile, dễ dàng quản lý và cập nhật khi có thay đổi trong mô hình.
*   **Tính linh hoạt:** Hỗ trợ nhiều kịch bản ánh xạ phức tạp, bao gồm ánh xạ lồng nhau, ánh xạ từ nhiều nguồn, và ánh xạ ngược.

### 2.3. Cài đặt AutoMapper

Để sử dụng AutoMapper, chúng ta cần cài đặt hai gói NuGet vào dự án của mình:

1.  **`AutoMapper`**: Gói core của thư viện, chứa các chức năng ánh xạ chính.
2.  **`AutoMapper.Extensions.Microsoft.DependencyInjection`**: Gói này cung cấp các tiện ích mở rộng để tích hợp AutoMapper một cách liền mạch với hệ thống Dependency Injection (DI) của ASP.NET Core, giúp việc cấu hình và sử dụng trở nên dễ dàng hơn.

Bạn có thể cài đặt chúng thông qua NuGet Package Manager Console:

```bash
Install-Package AutoMapper
Install-Package AutoMapper.Extensions.Microsoft.DependencyInjection
```

Hoặc sử dụng giao diện người dùng NuGet Package Manager trong Visual Studio.

### 2.4. Cấu hình AutoMapper với Profile

AutoMapper sử dụng khái niệm **Profile** để định nghĩa và nhóm các quy tắc ánh xạ. Mỗi Profile là một lớp kế thừa từ `AutoMapper.Profile`, và trong constructor của nó, chúng ta sẽ khai báo các ánh xạ giữa các cặp đối tượng.

#### 2.4.1. Tạo thư mục `Profiles`

Tạo một thư mục mới có tên `Profiles` trong thư mục gốc của dự án để chứa tất cả các Profile ánh xạ.

```
NZWalks.API/
├── ...
├── Models/
├── Profiles/         // Thư mục mới cho AutoMapper Profiles
│   └── RegionProfile.cs
├── ...
```

#### 2.4.2. Định nghĩa `RegionProfile`

Tạo một lớp mới `RegionProfile.cs` bên trong thư mục `Profiles`.

**`Profiles/RegionProfile.cs`**
```csharp
using AutoMapper; // Thêm namespace AutoMapper
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTOs;

namespace NZWalks.API.Profiles
{
    // RegionProfile kế thừa từ AutoMapper.Profile để định nghĩa các ánh xạ
    public class RegionProfile : Profile
    {
        public RegionProfile()
        {
            // Định nghĩa ánh xạ từ Domain Model (Region) sang DTO (RegionDto)
            // AutoMapper sẽ tự động ánh xạ các thuộc tính có tên và kiểu dữ liệu giống nhau.
            CreateMap<Region, RegionDto>()
                .ReverseMap(); // .ReverseMap() là một tiện ích, cho phép AutoMapper
                              // tự động tạo ánh xạ ngược lại từ RegionDto sang Region.
                              // Điều này hữu ích khi bạn muốn ánh xạ cả hai chiều
                              // và các thuộc tính tương đồng.

            // Ví dụ về cấu hình ánh xạ chi tiết hơn (nếu cần):
            // Nếu bạn có một thuộc tính trong DTO tên là "RegionName" nhưng trong Domain Model
            // lại là "Name", bạn sẽ cấu hình như sau:
            // CreateMap<Region, RegionDto>()
            //     .ForMember(dest => dest.RegionName, opt => opt.MapFrom(src => src.Name))
            //     .ReverseMap();
            
            // Hoặc bỏ qua một thuộc tính:
            // CreateMap<Region, RegionDto>()
            //     .ForMember(dest => dest.SomeInternalProperty, opt => opt.Ignore());
        }
    }
}
```
Trong ví dụ này, `CreateMap<Region, RegionDto>()` khai báo rằng chúng ta muốn ánh xạ từ đối tượng `Region` (nguồn) sang `RegionDto` (đích). Phương thức `.ReverseMap()` là một tiện ích giúp AutoMapper tự động tạo ánh xạ ngược lại từ `RegionDto` sang `Region`, giả sử các thuộc tính cũng khớp tên. Đối với các trường hợp phức tạp hơn, bạn có thể sử dụng `ForMember()` để tùy chỉnh ánh xạ cho từng thuộc tính cụ thể.

### 2.5. Đăng ký AutoMapper vào Dependency Injection

Sau khi đã tạo Profile, chúng ta cần đăng ký AutoMapper vào hệ thống Dependency Injection của ASP.NET Core để ứng dụng có thể phát hiện và sử dụng các Profile đã định nghĩa.

Trong tệp `Program.cs`, thêm dòng cấu hình sau (thường là sau khi đã thêm các repository hoặc các dịch vụ khác):

**`Program.cs`**
```csharp
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Repositories;
using AutoMapper; // Thêm namespace AutoMapper

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Cấu hình Database Context với SQL Server
builder.Services.AddDbContext<NZWalksDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("NZWalksConnectionString"));
});

// Đăng ký Repository
builder.Services.AddScoped<IRegionRepository, SQLRegionRepository>();

// Đăng ký AutoMapper vào hệ thống Dependency Injection
// AutoMapper sẽ quét tất cả các Profile trong assembly hiện tại
// (assembly chứa lớp Program) và đăng ký chúng.
builder.Services.AddAutoMapper(typeof(Program).Assembly);

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
`builder.Services.AddAutoMapper(typeof(Program).Assembly)` là phương thức mở rộng từ gói `AutoMapper.Extensions.Microsoft.DependencyInjection`. Nó sẽ tự động tìm tất cả các lớp kế thừa từ `Profile` trong assembly chứa lớp `Program` (thường là assembly của dự án hiện tại) và đăng ký chúng vào DI container. Điều này giúp bạn dễ dàng quản lý nhiều Profile mà không cần đăng ký từng Profile một cách thủ công.

### 2.6. Sử dụng AutoMapper trong Controller

Cuối cùng, chúng ta sẽ cập nhật `RegionsController` để sử dụng `IMapper` đã được inject thông qua Dependency Injection.

1.  **Inject `IMapper`:** Thêm `IMapper` vào constructor của `RegionsController`.
2.  **Sử dụng `_mapper.Map<DestinationType>(source)`:** Thay thế đoạn code ánh xạ thủ công bằng một dòng code sử dụng phương thức `Map()` của `IMapper`.

**`Controllers/RegionsController.cs` (với AutoMapper)**
```csharp
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTOs;
using NZWalks.API.Repositories;
using AutoMapper; // Thêm namespace AutoMapper

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository regionRepository;
        private readonly IMapper mapper; // Khai báo biến IMapper để sử dụng AutoMapper

        // Inject IRegionRepository và IMapper qua constructor
        public RegionsController(IRegionRepository regionRepository, IMapper mapper)
        {
            this.regionRepository = regionRepository;
            this.mapper = mapper; // Gán IMapper được inject
        }

        // GET ALL REGIONS (Sử dụng AutoMapper để ánh xạ)
        [HttpGet]
        public IActionResult GetAllRegions()
        {
            // 1. Lấy Domain Models từ Repository
            var regionsDomain = regionRepository.GetAll();

            // 2. Chuyển đổi Domain Models sang DTOs bằng AutoMapper
            // AutoMapper có thể tự động ánh xạ danh sách các đối tượng từ nguồn sang đích.
            var regionsDto = mapper.Map<List<RegionDto>>(regionsDomain);

            // 3. Trả về DTOs
            return Ok(regionsDto);
        }
    }
}
```
Như bạn có thể thấy, toàn bộ logic ánh xạ thủ công đã được thay thế bằng một dòng code đơn giản: `mapper.Map<List<RegionDto>>(regionsDomain);`. Điều này làm cho code của controller gọn gàng hơn, dễ đọc và dễ bảo trì hơn rất nhiều, đồng thời tập trung vào vai trò chính của nó là xử lý yêu cầu HTTP và điều phối dữ liệu.

---

## 3. Lập trình Bất đồng bộ (Async/Await): Nâng cao Hiệu suất và Khả năng mở rộng của API

Trong các ứng dụng web hiện đại, đặc biệt là các API phải xử lý hàng ngàn yêu cầu đồng thời, khả năng phản hồi nhanh và hiệu quả là cực kỳ quan trọng. Lập trình bất đồng bộ (Asynchronous Programming) với `async/await` trong C# là một công cụ mạnh mẽ để đạt được điều này, đặc biệt khi ứng dụng phải thực hiện các thao tác I/O tốn thời gian như truy cập cơ sở dữ liệu, gọi API bên ngoài, hoặc đọc/ghi file.

### 3.1. Mã Đồng bộ (Synchronous Code) và những hạn chế

**Mã đồng bộ (Synchronous code)** thực hiện các tác vụ một cách tuần tự, từng bước một. Khi một tác vụ bắt đầu, luồng (thread) hiện tại sẽ bị chặn (block) cho đến khi tác vụ đó hoàn thành hoàn toàn. Chỉ sau khi tác vụ đầu tiên kết thúc, tác vụ tiếp theo mới có thể bắt đầu.

**Cơ chế hoạt động (Under the Hood):**
Trong môi trường ASP.NET Core, khi một yêu cầu HTTP đến, một luồng từ Thread Pool được lấy ra để xử lý yêu cầu đó. Nếu phương thức xử lý yêu cầu (Action Method) gọi một tác vụ I/O đồng bộ (ví dụ: `regionRepository.GetAll()`), luồng đó sẽ bị "đóng băng" và không thể thực hiện bất kỳ công việc nào khác cho đến khi tác vụ I/O hoàn tất.

**Ví dụ trong ngữ cảnh API (mã đồng bộ):**
Hãy tưởng tượng một nhà hàng chỉ có một đầu bếp (luồng). Khi một khách hàng (yêu cầu HTTP) đặt món (gọi API `GetAllRegions()`), đầu bếp bắt đầu chuẩn bị món ăn (gọi `regionRepository.GetAll()`). Nếu món ăn cần một nguyên liệu phải lấy từ kho rất xa (truy vấn CSDL mất thời gian), đầu bếp phải đi đến kho, tìm nguyên liệu, mang về, và chỉ khi đó mới tiếp tục nấu. Trong suốt thời gian đầu bếp đi đến kho và quay về, không có món ăn nào khác được chuẩn bị, và các khách hàng khác phải chờ đợi.

**Hạn chế của mã đồng bộ:**

*   **Giảm khả năng phản hồi (Responsiveness):** Nếu một tác vụ I/O mất nhiều thời gian (ví dụ: 500ms để truy vấn CSDL), luồng sẽ bị chặn trong suốt thời gian đó. Điều này khiến ứng dụng không thể xử lý các yêu cầu khác trong thời gian chờ đợi, dẫn đến độ trễ cao và trải nghiệm người dùng kém.
*   **Lãng phí tài nguyên (Resource Wastage):** Các luồng bị chặn vẫn chiếm giữ tài nguyên hệ thống (bộ nhớ, CPU context) nhưng không thực hiện công việc hữu ích nào. Chúng chỉ đơn giản là chờ đợi. Điều này dẫn đến việc sử dụng tài nguyên kém hiệu quả.
*   **Kém khả năng mở rộng (Poor Scalability):** Khi số lượng yêu cầu đồng thời tăng lên, Thread Pool có thể nhanh chóng cạn kiệt luồng bị chặn. Tình trạng này được gọi là "thread starvation" và làm giảm nghiêm trọng khả năng đáp ứng của ứng dụng. Các yêu cầu mới sẽ phải chờ đợi luồng rảnh, hoặc tệ hơn là bị từ chối.

### 3.2. Mã Bất đồng bộ (Asynchronous Code) với `async/await`

**Mã bất đồng bộ (Asynchronous code)** cho phép các tác vụ bắt đầu mà không cần chờ tác vụ trước đó hoàn thành. Thay vì chặn luồng, một tác vụ bất đồng bộ sẽ "giải phóng" luồng để luồng đó có thể xử lý các công việc khác. Khi tác vụ bất đồng bộ hoàn thành, nó sẽ thông báo cho hệ thống biết để tiếp tục xử lý kết quả. Trong C#, điều này được thực hiện thông qua các từ khóa `async` và `await`.

#### 3.2.1. Cơ chế hoạt động của `async/await` (Under the Hood)

1.  **`async`**: Từ khóa này được đặt trước một phương thức để đánh dấu rằng phương thức đó có thể chứa các toán tử `await`. Một phương thức `async` luôn trả về một `Task` (nếu không có giá trị trả về) hoặc `Task<T>` (nếu trả về giá trị kiểu `T`). `Task` là một đối tượng đại diện cho một hoạt động có thể hoàn thành trong tương lai.
2.  **`await`**: Từ khóa này được sử dụng bên trong một phương thức `async` để tạm dừng việc thực thi của phương thức đó cho đến khi một `Task` hoàn thành.
    *   Khi `await` được gọi trên một `Task` (ví dụ: `await dbContext.Regions.ToListAsync();`), luồng hiện tại sẽ được giải phóng và trở về Thread Pool. Nó có thể được sử dụng để xử lý một yêu cầu HTTP khác hoặc thực hiện các công việc khác.
    *   Tác vụ I/O (ví dụ: truy vấn CSDL) sẽ tiếp tục chạy ở chế độ nền (không chặn luồng managed của .NET). Hệ điều hành sẽ thông báo khi tác vụ I/O hoàn thành thông qua I/O Completion Port (IOCP).
    *   Khi tác vụ được `await` hoàn thành, một luồng từ Thread Pool (có thể là luồng ban đầu hoặc một luồng mới) sẽ được lấy ra để tiếp tục thực thi phần còn lại của phương thức `async` từ điểm dừng `await`.
    *   Nếu tác vụ đã hoàn thành ngay lập tức (ví dụ: dữ liệu đã có trong bộ nhớ cache), `await` sẽ không giải phóng luồng mà tiếp tục thực thi ngay lập tức, tránh overhead không cần thiết.

**Ví dụ trong ngữ cảnh API (với `async/await`):**
Trở lại ví dụ nhà hàng: Bây giờ, đầu bếp (luồng) không đi đến kho xa nữa. Khi món ăn cần nguyên liệu từ kho, đầu bếp sẽ đưa một "phiếu yêu cầu" cho một người giao hàng (hệ thống I/O) và nói "Khi nào có nguyên liệu này, hãy mang nó đến đây và tôi sẽ tiếp tục làm món này". Sau đó, đầu bếp không đứng chờ mà quay sang chuẩn bị món khác cho khách hàng khác. Khi người giao hàng mang nguyên liệu đến, đầu bếp (hoặc một đầu bếp khác rảnh) sẽ lấy nguyên liệu và tiếp tục làm món ăn đã bị tạm dừng.

#### 3.2.2. Lợi ích của mã bất đồng bộ

*   **Tăng khả năng phản hồi (Increased Responsiveness):** Ứng dụng có thể xử lý nhiều yêu cầu đồng thời mà không bị chặn, mang lại trải nghiệm người dùng tốt hơn và độ trễ thấp hơn.
*   **Tối ưu hóa tài nguyên (Optimized Resource Utilization):** Các luồng không bị chặn chờ đợi mà được giải phóng để thực hiện công việc khác, giúp sử dụng hiệu quả hơn các luồng trong Thread Pool. Điều này có nghĩa là với cùng một lượng tài nguyên phần cứng, ứng dụng có thể xử lý nhiều yêu cầu hơn.
*   **Cải thiện khả năng mở rộng (Improved Scalability):** Do tài nguyên luồng được sử dụng hiệu quả hơn, ứng dụng có thể xử lý số lượng yêu cầu lớn hơn với cùng một lượng tài nguyên phần cứng, hoặc xử lý cùng số lượng yêu cầu với ít tài nguyên hơn.
*   **Đặc biệt hữu ích cho I/O-bound operations:** Hầu hết các thao tác trong Web API (truy cập CSDL, gọi dịch vụ ngoài, đọc/ghi file) là I/O-bound (chờ đợi dữ liệu), nơi `async/await` mang lại lợi ích lớn nhất. Đối với các tác vụ CPU-bound (tính toán nặng), `async/await` không trực tiếp cải thiện hiệu suất mà chỉ giúp giải phóng luồng UI/Request; để xử lý CPU-bound hiệu quả, bạn cần sử dụng `Task.Run()` để đẩy công việc sang một luồng riêng biệt.

#### 3.2.3. Nguyên tắc "Async All The Way Down"

Để đạt được lợi ích tối đa của lập trình bất đồng bộ, bạn nên áp dụng `async/await` từ lớp ngoài cùng (Controller) xuống lớp thấp nhất (Repository/Database access) trong chuỗi gọi.

**Tại sao "Async All The Way Down" lại quan trọng?**
Nếu bạn gọi một phương thức bất đồng bộ (`GetAllAsync()`) từ một phương thức đồng bộ (`GetAll()`), bạn sẽ phải gọi `GetAllAsync().Result` hoặc `GetAllAsync().Wait()`. Điều này sẽ chặn luồng hiện tại cho đến khi tác vụ bất đồng bộ hoàn thành, gây ra các vấn đề tương tự như mã đồng bộ, và thậm chí có thể dẫn đến **deadlock** trong một số trường hợp (ví dụ: trong môi trường UI có `SynchronizationContext`). Bằng cách làm cho toàn bộ chuỗi cuộc gọi bất đồng bộ, bạn đảm bảo rằng luồng không bao giờ bị chặn không cần thiết, tối đa hóa hiệu quả của `async/await`.

### 3.3. Áp dụng Async/Await trong dự án NZWalks

Chúng ta sẽ áp dụng `async/await` vào API của mình, bắt đầu từ lớp Repository (nơi tương tác trực tiếp với cơ sở dữ liệu) và sau đó lan truyền lên Controller.

#### 3.3.1. Sửa đổi Repository Layer

1.  **Cập nhật Interface `IRegionRepository`:**
    Thay đổi kiểu trả về của phương thức `GetAll()` để nó trả về một `Task<List<Region>>`. Đồng thời, theo quy ước trong C#, chúng ta sẽ thêm hậu tố `Async` vào tên phương thức để chỉ rõ nó là bất đồng bộ.

    **`Repositories/IRegionRepository.cs`**
    ```csharp
    using NZWalks.API.Models.Domain;
    using System.Collections.Generic;
    using System.Threading.Tasks; // Cần thiết cho Task

    namespace NZWalks.API.Repositories
    {
        public interface IRegionRepository
        {
            // Thay đổi kiểu trả về thành Task<List<Region>> và thêm hậu tố Async
            Task<List<Region>> GetAllAsync();
        }
    }
    ```

2.  **Cập nhật Implementation `SQLRegionRepository`:**
    *   Thêm từ khóa `async` vào định nghĩa phương thức.
    *   Sử dụng từ khóa `await` khi gọi các phương thức bất đồng bộ của Entity Framework Core (ví dụ: `ToListAsync()`).
    *   Đảm bảo phương thức trả về `Task<List<Region>>`.

    **`Repositories/SQLRegionRepository.cs`**
    ```csharp
    using Microsoft.EntityFrameworkCore; // Cần thiết cho ToListAsync()
    using NZWalks.API.Data;
    using NZWalks.API.Models.Domain;
    using System.Collections.Generic;
    using System.Threading.Tasks;

    namespace NZWalks.API.Repositories
    {
        public class SQLRegionRepository : IRegionRepository
        {
            private readonly NZWalksDbContext dbContext;

            public SQLRegionRepository(NZWalksDbContext dbContext)
            {
                this.dbContext = dbContext;
            }

            // Đánh dấu phương thức là async và sử dụng await cho thao tác I/O
            public async Task<List<Region>> GetAllAsync()
            {
                // ToListAsync() là một phương thức mở rộng bất đồng bộ từ Entity Framework Core.
                // Khi await được gọi, luồng hiện tại sẽ được giải phóng.
                // Truy vấn CSDL sẽ chạy ở chế độ nền.
                // Khi kết quả có, luồng sẽ được tiếp tục để trả về List<Region>.
                return await dbContext.Regions.ToListAsync();
            }
        }
    }
    ```
    `ToListAsync()` là một phương thức mở rộng từ Entity Framework Core, nó sẽ thực thi truy vấn cơ sở dữ liệu một cách bất đồng bộ và trả về kết quả dưới dạng `Task<List<Region>>`. Bằng cách `await` nó, chúng ta đảm bảo rằng luồng không bị chặn trong khi chờ đợi CSDL phản hồi.

#### 3.3.2. Sửa đổi Controller Layer

Sau khi Repository đã bất đồng bộ, chúng ta cần cập nhật Controller để gọi phương thức bất đồng bộ này và cũng tự nó trở thành bất đồng bộ theo nguyên tắc "Async All The Way Down".

1.  **Cập nhật phương thức action `GetAllRegions`:**
    *   Thêm từ khóa `async` vào định nghĩa phương thức.
    *   Thay đổi kiểu trả về của phương thức action từ `IActionResult` thành `Task<IActionResult>`.
    *   Sử dụng từ khóa `await` khi gọi phương thức repository `GetAllAsync()`.

    **`Controllers/RegionsController.cs` (với Async/Await và AutoMapper)**
    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using NZWalks.API.Models.Domain;
    using NZWalks.API.Models.DTOs;
    using NZWalks.API.Repositories;
    using AutoMapper;
    using System.Collections.Generic;
    using System.Threading.Tasks; // Cần thiết cho Task

    namespace NZWalks.API.Controllers
    {
        [ApiController]
        [Route("api/[controller]")]
        public class RegionsController : ControllerBase
        {
            private readonly IRegionRepository regionRepository;
            private readonly IMapper mapper;

            public RegionsController(IRegionRepository regionRepository, IMapper mapper)
            {
                this.regionRepository = regionRepository;
                this.mapper = mapper;
            }

            // GET ALL REGIONS (Sử dụng Async/Await và AutoMapper)
            [HttpGet]
            // Đánh dấu phương thức là async và trả về Task<IActionResult>
            // Theo quy ước, thêm hậu tố Async vào tên phương thức
            public async Task<IActionResult> GetAllAsync()
            {
                // 1. Gọi phương thức bất đồng bộ từ Repository và await kết quả.
                // Luồng HTTP sẽ được giải phóng trong thời gian chờ CSDL.
                var regionsDomain = await regionRepository.GetAllAsync();

                // 2. Chuyển đổi Domain Models sang DTOs bằng AutoMapper.
                // Thao tác ánh xạ này thường là CPU-bound và khá nhanh,
                // nên không cần await ở đây trừ khi bạn có một hàm MapAsync.
                var regionsDto = mapper.Map<List<RegionDto>>(regionsDomain);

                // 3. Trả về DTOs qua Ok (HTTP 200 OK)
                return Ok(regionsDto);
            }
        }
    }
    ```
    Bây giờ, khi một yêu cầu đến `GetAllAsync()`, khi nó gọi `await regionRepository.GetAllAsync()`, luồng HTTP sẽ được giải phóng để xử lý các yêu cầu khác. Khi dữ liệu từ cơ sở dữ liệu sẵn sàng, phương thức `GetAllAsync()` sẽ tiếp tục thực thi từ điểm dừng, ánh xạ dữ liệu và trả về kết quả. Điều này giúp API của chúng ta phản hồi nhanh hơn và có khả năng mở rộng tốt hơn dưới tải nặng, tối ưu hóa việc sử dụng tài nguyên máy chủ.

---

## 4. Lưu Thay đổi vào Hệ thống Kiểm soát Phiên bản (Git Commit)

Sau khi hoàn thành các cải tiến quan trọng này cho API của chúng ta, việc lưu các thay đổi vào hệ thống kiểm soát phiên bản Git là bước không thể thiếu. Việc commit thường xuyên giúp bạn theo dõi tiến độ, dễ dàng quay lại các phiên bản trước khi cần, và cộng tác hiệu quả hơn với nhóm phát triển.

Để lưu các thay đổi:

1.  **Mở cửa sổ Git Changes** trong Visual Studio hoặc sử dụng dòng lệnh Git trong terminal.
2.  **Đánh dấu tất cả các tệp đã thay đổi** (Stage Changes) bao gồm các tệp DTO (`RegionDto.cs`), Profile (`RegionProfile.cs`), Repository (`IRegionRepository.cs`, `SQLRegionRepository.cs`), và Controller (`RegionsController.cs`), cùng với tệp cấu hình (`Program.cs`).
3.  **Viết một thông điệp commit rõ ràng và súc tích** tóm tắt các thay đổi đã thực hiện. Một thông điệp commit tốt nên bắt đầu bằng một động từ thể hiện hành động, ví dụ: "feat: Implement DTOs, AutoMapper, and Async/Await for Regions API".
4.  **Thực hiện Commit** để lưu các thay đổi vào kho lưu trữ cục bộ của bạn.
5.  Sau đó, **Push** các thay đổi này lên kho lưu trữ từ xa (ví dụ: GitHub, Azure DevOps) nếu bạn đang làm việc trong một dự án được chia sẻ hoặc muốn sao lưu công việc của mình.

---

## Tóm tắt Phần 21

Trong phần này, chúng ta đã thực hiện một loạt các cải tiến kiến trúc quan trọng cho RESTful API Vùng của mình, giúp nó tuân thủ các phương pháp hay nhất trong phát triển API hiện đại, tăng cường khả năng bảo trì và hiệu suất:

*   **Data Transfer Objects (DTOs):** Chúng ta đã hiểu sâu sắc lý do tại sao việc phơi bày mô hình miền trực tiếp là một rủi ro và cách tạo `RegionDto` để định hình dữ liệu giao tiếp với client. DTOs là một "hợp đồng" API ổn định, giúp tách biệt các mối quan tâm, bảo vệ mô hình miền khỏi các thay đổi gây phá vỡ và kiểm soát chính xác dữ liệu được phơi bày.
*   **AutoMapper:** Để giải quyết vấn đề ánh xạ đối tượng thủ công lặp lại và dễ gây lỗi, chúng ta đã cài đặt và cấu hình thư viện AutoMapper. Bằng cách tạo một `RegionProfile` và đăng ký nó vào Dependency Injection, chúng ta có thể ánh xạ giữa `Region` và `RegionDto` chỉ bằng một dòng code, giúp code của controller gọn gàng, dễ đọc và dễ bảo trì hơn rất nhiều.
*   **Lập trình Bất đồng bộ (Async/Await):** Chúng ta đã khám phá sự khác biệt cơ bản giữa code đồng bộ và bất đồng bộ, hiểu được cơ chế hoạt động "under the hood" của `async/await` và những lợi ích to lớn của nó trong việc cải thiện hiệu suất và khả năng mở rộng của API, đặc biệt với các thao tác I/O-bound như truy vấn cơ sở dữ liệu. Chúng ta đã áp dụng `async/await` từ lớp Repository lên Controller, tuân thủ nguyên tắc "Async All The Way Down" để tối ưu hóa việc sử dụng tài nguyên hệ thống.

Những kỹ thuật này là nền tảng không thể thiếu cho việc xây dựng các RESTful API hiệu quả, có khả năng mở rộng và dễ bảo trì trong môi trường ASP.NET Core, chuẩn bị cho các tính năng phức tạp hơn trong tương lai.

<!-- REVIEWED_BY_AGENT -->
