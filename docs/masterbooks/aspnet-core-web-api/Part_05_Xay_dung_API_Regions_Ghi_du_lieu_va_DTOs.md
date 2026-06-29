# Phần 5: Xây dựng API cho Vùng (Regions) - Ghi dữ liệu & DTOs

Trong kỷ nguyên phát triển phần mềm hiện đại, việc xây dựng các API RESTful mạnh mẽ, dễ bảo trì và an toàn là yếu tố then chốt. Phần này sẽ đưa chúng ta đi sâu vào việc mở rộng API RESTful hiện có để hỗ trợ các hoạt động ghi dữ liệu (Create, Update, Delete) cho tài nguyên "Vùng" (Region). Trọng tâm chính là giới thiệu và sử dụng các Đối tượng Truyền Dữ liệu (DTOs) một cách hiệu quả, đảm bảo sự tách biệt mối quan tâm, bảo mật và khả năng bảo trì cho ứng dụng. Chúng ta sẽ khám phá cách DTOs khác biệt với các mô hình miền (Domain Models) và cách áp dụng chúng vào cả các phương thức đọc (GET) đã có, cũng như xây dựng các phương thức hành động (Action Methods) mới cho việc tạo, cập nhật và xóa dữ liệu sử dụng ASP.NET Core, Entity Framework Core, và các động từ HTTP chuẩn. Đặc biệt, chúng ta sẽ lồng ghép tư duy "Vibe Coding" và cách một hệ thống Agentic AI như Antigravity IDE có thể hỗ trợ bạn trong suốt quá trình này.

## 1. Nền tảng thiết kế API: DTOs và Mô hình Miền

Trước khi đi sâu vào triển khai, việc nắm vững các khái niệm cốt lõi về DTOs và Mô hình Miền là cực kỳ quan trọng để xây dựng một kiến trúc ứng dụng vững chắc.

### 1.1. Mô hình Miền (Domain Models) - Trái tim của nghiệp vụ

Mô hình miền là các đối tượng đại diện cho dữ liệu và hành vi cốt lõi của miền nghiệp vụ trong ứng dụng của bạn. Chúng là "ngôn ngữ chung" mà các nhà phát triển và chuyên gia nghiệp vụ sử dụng để mô tả hệ thống. Trong ngữ cảnh của ASP.NET Core kết hợp với Entity Framework Core (EF Core), các mô hình miền thường đóng vai trò là các thực thể (Entities) được EF Core sử dụng để tương tác trực tiếp với cơ sở dữ liệu. Chúng chứa tất cả các thuộc tính cần thiết để lưu trữ và thao tác dữ liệu ở cấp độ cơ sở dữ liệu, đồng thời có thể chứa các quy tắc nghiệp vụ hoặc phương thức liên quan đến dữ liệu đó.

> [!NOTE]
> `DbContext` của EF Core chỉ nhận biết và làm việc với các đối tượng mô hình miền này. Khi bạn truy vấn dữ liệu, EF Core sẽ tạo ra các đối tượng mô hình miền từ các hàng trong cơ sở dữ liệu. Khi bạn lưu dữ liệu, EF Core sẽ phân tích các thay đổi trên các đối tượng mô hình miền này để tạo ra các câu lệnh SQL `INSERT`, `UPDATE`, `DELETE` phù hợp. Cơ chế này được gọi là **Change Tracking** (Theo dõi thay đổi), một tính năng cốt lõi của EF Core giúp đơn giản hóa việc tương tác với cơ sở dữ liệu.

**Ví dụ về Mô hình Miền `Region`:**

```csharp
// Models/Region.cs
namespace MyApi.Models
{
    public class Region
    {
        public Guid Id { get; set; } // Khóa chính duy nhất cho mỗi vùng
        public string Code { get; set; } // Mã vùng (ví dụ: "NZ-AKL" cho Auckland, New Zealand)
        public string Name { get; set; } // Tên vùng (ví dụ: "Auckland")
        public string? RegionImageUrl { get; set; } // URL hình ảnh minh họa cho vùng, có thể null
    }
}
```

### 1.2. DTO (Data Transfer Object) - Giao diện của API

DTO, hay Đối tượng Truyền Dữ liệu, là các đối tượng đơn giản được thiết kế đặc biệt để truyền dữ liệu giữa các lớp hoặc ranh giới khác nhau của ứng dụng, đặc biệt là giữa API của bạn và ứng dụng khách (client). Chúng thường là một tập hợp con hoặc một phiên bản được định hình lại của các thuộc tính của mô hình miền.

> [!TIP]
> Hãy hình dung DTO như một "hợp đồng" (contract) công khai giữa API của bạn và ứng dụng khách. Nó định nghĩa chính xác những dữ liệu nào được gửi đi hoặc nhận vào qua mạng, không nhất thiết phải trùng khớp hoàn toàn với cấu trúc dữ liệu nội bộ (Domain Model) của bạn. DTO là cách bạn kiểm soát "bộ mặt" của API.

**Mục đích chính của DTOs:**

*   **Truyền dữ liệu qua mạng:** Gửi dữ liệu từ API tới ứng dụng khách (client) hoặc ngược lại một cách tối ưu và an toàn.
*   **Truyền dữ liệu giữa các lớp ứng dụng:** Ví dụ, từ lớp dịch vụ đến lớp trình bày hoặc ngược lại, giữ cho các lớp này không phụ thuộc trực tiếp vào mô hình miền.

### 1.3. Tại sao cần DTOs? Lợi ích và sự tách biệt

Mặc dù DTOs có thể trông giống hệt mô hình miền, nhưng chúng phục vụ các mục đích khác nhau và mang lại nhiều lợi ích chiến lược:

*   **Tách biệt Mối quan tâm (Separation of Concerns - SoC):** Đây là lợi ích quan trọng nhất.
    *   **Mô hình miền:** Gắn liền với lược đồ cơ sở dữ liệu, logic nghiệp vụ, và các quy tắc ràng buộc dữ liệu nội bộ. Nó là nguồn sự thật duy nhất cho dữ liệu cốt lõi.
    *   **DTOs:** Được thiết kế để phù hợp với yêu cầu của giao diện API hoặc lớp trình bày. Chúng có thể được tùy chỉnh để chỉ hiển thị những gì cần thiết cho client, hoặc nhận những gì client được phép gửi.
    *   Sự tách biệt này đảm bảo rằng các thay đổi trong cơ sở dữ liệu hoặc logic nghiệp vụ (ví dụ: đổi tên cột, thêm thuộc tính nội bộ) không trực tiếp phá vỡ giao diện API của bạn hoặc buộc bạn phải cập nhật tất cả các ứng dụng khách.

*   **Hiệu suất (Performance):**
    *   DTOs cho phép bạn chỉ truy xuất và gửi những dữ liệu cần thiết qua mạng.
    *   **Ví dụ:** Nếu client chỉ cần tên và mã vùng, bạn có thể tạo một DTO chỉ chứa thuộc tính `Name` và `Code`, tránh việc gửi toàn bộ dữ liệu của mô hình miền `Region` (bao gồm `Id`, `RegionImageUrl` nếu không cần thiết). Điều này giảm tải mạng và thời gian xử lý ở cả phía máy chủ và máy khách.

*   **Bảo mật (Security):**
    *   Bằng cách hạn chế lượng dữ liệu được hiển thị hoặc cho phép cập nhật, DTOs giúp cải thiện bảo mật.
    *   Bạn có thể ẩn các thuộc tính nhạy cảm (ví dụ: mật khẩu, thông tin nội bộ) hoặc không cần thiết của mô hình miền khỏi ứng dụng khách. Điều này ngăn chặn các cuộc tấn công "over-posting" (khi client gửi dữ liệu cho các thuộc tính mà họ không được phép thay đổi).

*   **Lập phiên bản API (API Versioning):**
    *   DTOs có thể được lập phiên bản độc lập với mô hình miền. Điều này cho phép bạn cập nhật API (ví dụ: thêm hoặc xóa trường trong phản hồi) mà không làm hỏng các ứng dụng khách hiện có đang sử dụng phiên bản DTO cũ hơn. Bạn có thể duy trì nhiều phiên bản DTO cho các phiên bản API khác nhau.

*   **Kiểm soát dữ liệu đầu vào/đầu ra:**
    *   DTOs cho các hoạt động `POST` hoặc `PUT` (`AddRegionRequestDto`, `UpdateRegionRequestDto`) giúp bạn định nghĩa chính xác những gì client *được phép gửi* để tạo hoặc cập nhật tài nguyên. Điều này khác biệt với DTO dùng để *hiển thị* dữ liệu (`RegionDto`).

**Quy trình tương tác với DTOs:**

1.  **Khi nhận yêu cầu từ Client (POST, PUT):** Client gửi DTO (ví dụ: `AddRegionRequestDto`) → API nhận DTO → **Chuyển đổi DTO sang Mô hình Miền** → Lưu Mô hình Miền vào Cơ sở dữ liệu.
2.  **Khi gửi phản hồi cho Client (GET, POST, PUT, DELETE):** API truy xuất Mô hình Miền từ Cơ sở dữ liệu → **Chuyển đổi Mô hình Miền sang DTO** (ví dụ: `RegionDto`) → Gửi DTO cho Client.

---

#### Tư duy Vibe Coding với DTOs và Antigravity IDE

Khi làm việc với Antigravity IDE, việc thiết kế DTOs trở nên trực quan hơn. Bạn có thể bắt đầu bằng cách mô tả yêu cầu nghiệp vụ hoặc giao diện API mong muốn bằng ngôn ngữ tự nhiên.

**Ví dụ:**

*   **Prompt cho Antigravity:** "Tôi cần một DTO để hiển thị thông tin vùng. Nó nên có Id, Code, Name và RegionImageUrl."
    *   *Antigravity sẽ:* Tạo `RegionDto.cs` với các thuộc tính tương ứng.
*   **Prompt cho Antigravity:** "Thiết kế DTO để tạo vùng mới. Client chỉ được phép cung cấp Code, Name và RegionImageUrl. Id sẽ được tạo tự động bởi server."
    *   *Antigravity sẽ:* Tạo `AddRegionRequestDto.cs` mà không có thuộc tính `Id`.
*   **Prompt cho Antigravity:** "Khi cập nhật vùng, client có thể thay đổi Code, Name và RegionImageUrl. Id sẽ được cung cấp qua URL."
    *   *Antigravity sẽ:* Tạo `UpdateRegionRequestDto.cs` tương tự `AddRegionRequestDto`.

Antigravity không chỉ viết code mà còn giúp bạn *lý giải* các lựa chọn thiết kế DTO, so sánh với Domain Model và nhấn mạnh các lợi ích (bảo mật, hiệu suất). Nó khuyến khích bạn suy nghĩ về "hợp đồng" API trước khi đi sâu vào logic nghiệp vụ.

---

## 2. Nâng cấp API Đọc (GET) với DTOs

Trong các bài giảng trước, các phương thức GET của chúng ta có thể đã trả về trực tiếp các đối tượng mô hình miền `Region`. Bây giờ, chúng ta sẽ refactor chúng để sử dụng DTOs, tuân thủ các phương pháp hay nhất đã học.

### 2.1. Tạo DTO hiển thị (`RegionDto`)

Đầu tiên, hãy tạo một thư mục `DTOs` trong dự án của bạn (nếu chưa có) và định nghĩa DTO cho việc hiển thị thông tin Vùng.

```csharp
// DTOs/RegionDto.cs
namespace MyApi.DTOs
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

> [!NOTE]
> Trong ví dụ này, `RegionDto` có tất cả các thuộc tính của `Region` domain model. Tuy nhiên, bạn hoàn toàn có thể chỉ bao gồm một tập hợp con các thuộc tính nếu yêu cầu nghiệp vụ hoặc mục đích hiển thị cho phép. Ví dụ, nếu client chỉ cần `Name` và `Code`, bạn có thể bỏ `Id` và `RegionImageUrl` khỏi `RegionDto` để tối ưu hóa phản hồi.

### 2.2. Cập nhật phương thức `GetAllRegions`

Phương thức này sẽ truy xuất tất cả các vùng từ cơ sở dữ liệu dưới dạng mô hình miền, sau đó ánh xạ chúng sang danh sách `RegionDto` trước khi gửi phản hồi.

```csharp
// Controllers/RegionsController.cs
using MyApi.DTOs;
using MyApi.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using MyApi.Data; // Giả định DbContext của bạn nằm ở đây

namespace MyApi.Controllers
{
    [Route("api/[controller]")]
    [ApiController] // Tự động xử lý validation và trả về 400 Bad Request
    public class RegionsController : ControllerBase
    {
        private readonly ApplicationDbContext _dbContext; // Giả sử bạn đã inject DbContext

        public RegionsController(ApplicationDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        [HttpGet]
        public async Task<IActionResult> GetAllRegions()
        {
            // 1. Lấy dữ liệu từ cơ sở dữ liệu (Mô hình Miền)
            // EF Core truy vấn và tạo các đối tượng Region
            var regionsDomain = await _dbContext.Regions.ToListAsync();

            // 2. Ánh xạ Mô hình Miền sang DTO
            // Đây là bước quan trọng để chuyển đổi dữ liệu nội bộ sang định dạng công khai của API
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

            // 3. Trả về DTO cho client
            // Phản hồi 200 OK kèm theo danh sách DTO
            return Ok(regionsDto);
        }

        // ... các phương thức khác
    }
}
```

### 2.3. Cập nhật phương thức `GetRegionById`

Tương tự, phương thức này sẽ tìm một vùng theo ID, và nếu tìm thấy, nó sẽ ánh xạ mô hình miền sang `RegionDto` trước khi trả về.

```csharp
// Controllers/RegionsController.cs (tiếp tục)
// ...

        [HttpGet]
        [Route("{id:Guid}")] // Định nghĩa route với tham số ID kiểu Guid
        public async Task<IActionResult> GetRegionById([FromRoute] Guid id)
        {
            // 1. Lấy dữ liệu từ cơ sở dữ liệu (Mô hình Miền) dựa trên ID
            var regionDomain = await _dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id);

            // Kiểm tra xem vùng có tồn tại không
            if (regionDomain == null)
            {
                // Trả về 404 Not Found nếu không tìm thấy tài nguyên
                return NotFound();
            }

            // 2. Ánh xạ Mô hình Miền sang DTO
            var regionDto = new RegionDto
            {
                Id = regionDomain.Id,
                Code = regionDomain.Code,
                Name = regionDomain.Name,
                RegionImageUrl = regionDomain.RegionImageUrl
            };

            // 3. Trả về DTO cho client
            // Phản hồi 200 OK kèm theo DTO của tài nguyên tìm thấy
            return Ok(regionDto);
        }

// ...
```

> [!TIP]
> **Tối ưu hóa ánh xạ với AutoMapper:**
> Việc ánh xạ thủ công (manual mapping) như trên có thể trở nên tẻ nhạt, dễ gây lỗi và khó bảo trì khi số lượng thuộc tính hoặc DTO tăng lên. Trong các dự án thực tế, bạn nên xem xét sử dụng thư viện ánh xạ đối tượng như **AutoMapper**.
>
> AutoMapper giúp tự động hóa quá trình ánh xạ bằng cách định nghĩa các quy tắc ánh xạ một lần (ví dụ: `CreateMap<Region, RegionDto>()`). Sau đó, bạn chỉ cần gọi `_mapper.Map<List<RegionDto>>(regionsDomain)` hoặc `_mapper.Map<RegionDto>(regionDomain)` để thực hiện ánh xạ, giảm thiểu mã lặp lại và tăng cường khả năng bảo trì.
>
> **Tư duy Vibe Coding: Antigravity và Refactoring DTOs**
> Bạn có thể yêu cầu Antigravity IDE thực hiện refactor này.
> *   **Prompt:** "Refactor `GetAllRegions` và `GetRegionById` để sử dụng `RegionDto`. Antigravity, bạn có thể tạo giúp tôi AutoMapper profile và tích hợp nó không?"
> *   *Antigravity sẽ:*
>     1.  Tạo lớp `MappingProfiles.cs` kế thừa từ `Profile` của AutoMapper.
>     2.  Thêm `CreateMap<Region, RegionDto>();` vào profile.
>     3.  Hướng dẫn bạn cách đăng ký AutoMapper trong `Program.cs` hoặc `Startup.cs`.
>     4.  Chỉnh sửa `RegionsController` để inject `IMapper` và sử dụng nó để ánh xạ thay vì code thủ công.
>
> Điều này minh họa cách Antigravity không chỉ viết code mà còn giúp bạn áp dụng các phương pháp hay nhất và tích hợp thư viện một cách hiệu quả.

## 3. Xây dựng API Tạo Vùng Mới (HTTP POST)

Để tạo một tài nguyên mới, chúng ta sẽ sử dụng động từ HTTP `POST`. Phương thức này sẽ nhận một DTO chứa dữ liệu cần thiết từ client, chuyển đổi nó thành mô hình miền, lưu vào cơ sở dữ liệu và trả về phản hồi thích hợp.

### 3.1. Thiết kế `AddRegionRequestDto`

DTO này sẽ chỉ chứa các thuộc tính mà client được phép cung cấp khi tạo một vùng mới. ID sẽ được tạo tự động bởi ứng dụng (thường là ở phía máy chủ), vì vậy nó không cần có trong DTO này.

```csharp
// DTOs/AddRegionRequestDto.cs
using System.ComponentModel.DataAnnotations; // Để sử dụng các thuộc tính Validation

namespace MyApi.DTOs
{
    public class AddRegionRequestDto
    {
        [Required(ErrorMessage = "Mã vùng là bắt buộc.")] // Đảm bảo trường này không null
        [StringLength(3, ErrorMessage = "Mã vùng phải có đúng 3 ký tự.")] // Ví dụ: "NZL", "USA"
        public string Code { get; set; }

        [Required(ErrorMessage = "Tên vùng là bắt buộc.")]
        [MaxLength(100, ErrorMessage = "Tên vùng không được vượt quá 100 ký tự.")]
        public string Name { get; set; }

        public string? RegionImageUrl { get; set; } // Có thể null
    }
}
```

> [!NOTE]
> **Validation dữ liệu đầu vào:**
> Các thuộc tính như `[Required]`, `[StringLength]`, `[MaxLength]` là các Data Annotations giúp bạn định nghĩa các quy tắc kiểm tra tính hợp lệ cho dữ liệu nhận từ client.
>
> Khi bạn sử dụng `[ApiController]` trên controller, ASP.NET Core sẽ tự động thực hiện kiểm tra `ModelState` cho các DTO nhận được từ `[FromBody]`. Nếu dữ liệu không hợp lệ (ví dụ: `Code` bị thiếu hoặc không đúng độ dài), framework sẽ tự động trả về phản hồi `400 Bad Request` với chi tiết lỗi mà không cần bạn phải viết code kiểm tra thủ công. Điều này giúp giảm đáng kể mã lặp lại và tăng tính nhất quán.

### 3.2. Triển khai phương thức `CreateRegion` trong Controller

```csharp
// Controllers/RegionsController.cs (tiếp tục)
// ...

        [HttpPost]
        public async Task<IActionResult> CreateRegion([FromBody] AddRegionRequestDto addRegionRequestDto)
        {
            // [ApiController] tự động kiểm tra ModelState.IsValid.
            // Nếu không hợp lệ, nó sẽ trả về 400 Bad Request.
            // Do đó, không cần kiểm tra if (!ModelState.IsValid) ở đây.

            // 1. Ánh xạ DTO sang Mô hình Miền
            // Chuyển đổi dữ liệu từ "hợp đồng" API sang cấu trúc dữ liệu nội bộ
            var regionDomainModel = new Region
            {
                Id = Guid.NewGuid(), // Tạo ID mới duy nhất cho vùng
                Code = addRegionRequestDto.Code,
                Name = addRegionRequestDto.Name,
                RegionImageUrl = addRegionRequestDto.RegionImageUrl
            };

            // 2. Sử dụng DbContext để thêm Mô hình Miền vào cơ sở dữ liệu
            // _dbContext.Regions.AddAsync() thêm thực thể vào bộ theo dõi thay đổi của EF Core
            // Nó chưa ghi vào DB ngay lập tức.
            await _dbContext.Regions.AddAsync(regionDomainModel);
            
            // _dbContext.SaveChangesAsync() thực thi các thay đổi đang chờ xử lý vào cơ sở dữ liệu.
            // Đây là bước BẮT BUỘC để dữ liệu được ghi vào DB.
            // Nó cũng xử lý các giao dịch (transactions) cơ sở dữ liệu.
            await _dbContext.SaveChangesAsync();

            // 3. Ánh xạ Mô hình Miền đã tạo trở lại DTO để trả về client
            // Điều này đảm bảo client nhận được ID đã tạo và bất kỳ dữ liệu nào khác đã được xử lý bởi server.
            var regionDto = new RegionDto
            {
                Id = regionDomainModel.Id,
                Code = regionDomainModel.Code,
                Name = regionDomainModel.Name,
                RegionImageUrl = regionDomainModel.RegionImageUrl
            };

            // 4. Trả về 201 CreatedAtAction
            // CreatedAtAction là phản hồi tiêu chuẩn cho các hoạt động tạo tài nguyên thành công.
            // Nó tạo phản hồi HTTP 201 (Created)
            // Nó cũng thêm một header 'Location' trỏ đến URI của tài nguyên mới được tạo,
            // giúp client dễ dàng truy cập tài nguyên đó ngay lập tức.
            // Tham số đầu tiên: Tên Action để lấy URI (ví dụ: "GetRegionById")
            // Tham số thứ hai: Đối tượng ẩn danh chứa các giá trị route (ví dụ: new { id = regionDto.Id })
            // Tham số thứ ba: Đối tượng DTO được tạo để trả về trong body
            return CreatedAtAction(nameof(GetRegionById), new { id = regionDto.Id }, regionDto);
        }

// ...
```

> [!NOTE]
> *   **Động từ HTTP `POST`:** Được sử dụng để tạo tài nguyên mới. Nó **không an toàn** (thay đổi trạng thái server) và **không idempotent** (gửi cùng một yêu cầu nhiều lần có thể tạo ra nhiều tài nguyên giống hệt nhau).
> *   `[FromBody]` chỉ ra rằng tham số `addRegionRequestDto` sẽ được đọc từ phần thân của yêu cầu HTTP (thường là JSON).
> *   `Guid.NewGuid()` được sử dụng để tạo một ID duy nhất cho vùng mới. Đây là cách phổ biến khi sử dụng `Guid` làm khóa chính.

---

#### Tư duy Vibe Coding: Tạo API POST với Antigravity IDE

Khi bạn cần tạo một API `POST`, Antigravity có thể là một trợ thủ đắc lực.

**Ví dụ:**

*   **Prompt:** "Antigravity, tôi muốn tạo một endpoint `POST /api/regions` để thêm một vùng mới. Dữ liệu đầu vào sẽ là `AddRegionRequestDto` (Code, Name, RegionImageUrl). Endpoint này phải tạo một `Guid` mới cho Id, lưu vào cơ sở dữ liệu và trả về `201 Created` cùng với DTO của vùng đã tạo."
*   *Antigravity sẽ:*
    1.  Đề xuất hoặc tạo `AddRegionRequestDto` (nếu chưa có), bao gồm các Data Annotations để validation.
    2.  Tạo phương thức `CreateRegion` trong `RegionsController` với `[HttpPost]`, `[FromBody]`.
    3.  Viết logic ánh xạ từ DTO sang Domain Model, tạo `Guid.NewGuid()`.
    4.  Sử dụng `_dbContext.Regions.AddAsync()` và `_dbContext.SaveChangesAsync()`.
    5.  Ánh xạ ngược lại Domain Model sang `RegionDto` và trả về `CreatedAtAction`.
    6.  Antigravity cũng có thể nhắc nhở bạn về tầm quan trọng của `[ApiController]` để kích hoạt validation tự động.

Với Antigravity, bạn có thể tập trung vào "ý định" của mình, để AI xử lý các chi tiết triển khai tiêu chuẩn, giúp bạn tiết kiệm thời gian và giảm thiểu lỗi.

---

## 4. Xây dựng API Cập nhật Vùng (HTTP PUT)

Để cập nhật một tài nguyên hiện có, chúng ta sử dụng động từ HTTP `PUT`. Phương thức này sẽ nhận ID của tài nguyên cần cập nhật từ URL và dữ liệu cập nhật từ phần thân yêu cầu.

### 4.1. Thiết kế `UpdateRegionRequestDto`

DTO này sẽ chứa các thuộc tính mà client được phép cập nhật. ID của vùng sẽ được lấy từ URL chứ không phải từ DTO này.

```csharp
// DTOs/UpdateRegionRequestDto.cs
using System.ComponentModel.DataAnnotations;

namespace MyApi.DTOs
{
    public class UpdateRegionRequestDto
    {
        [Required(ErrorMessage = "Mã vùng là bắt buộc.")]
        [StringLength(3, ErrorMessage = "Mã vùng phải có đúng 3 ký tự.")]
        public string Code { get; set; }

        [Required(ErrorMessage = "Tên vùng là bắt buộc.")]
        [MaxLength(100, ErrorMessage = "Tên vùng không được vượt quá 100 ký tự.")]
        public string Name { get; set; }

        public string? RegionImageUrl { get; set; }
    }
}
```

> [!NOTE]
> Tương tự như `AddRegionRequestDto`, bạn có thể tùy chỉnh các thuộc tính trong `UpdateRegionRequestDto` tùy theo yêu cầu nghiệp vụ. Ví dụ, nếu bạn không muốn cho phép client thay đổi `Code` sau khi vùng đã được tạo, bạn có thể bỏ thuộc tính đó khỏi DTO này. Các Data Annotations cũng được áp dụng tương tự để validation.

### 4.2. Triển khai phương thức `UpdateRegion` trong Controller

```csharp
// Controllers/RegionsController.cs (tiếp tục)
// ...

        [HttpPut]
        [Route("{id:Guid}")] // Định nghĩa route với tham số ID kiểu Guid
        public async Task<IActionResult> UpdateRegion([FromRoute] Guid id, 
                                                      [FromBody] UpdateRegionRequestDto updateRegionRequestDto)
        {
            // [ApiController] tự động kiểm tra ModelState.IsValid.

            // 1. Tìm Mô hình Miền hiện có trong cơ sở dữ liệu
            // EF Core sẽ theo dõi thực thể này sau khi nó được truy xuất.
            var regionDomainModel = await _dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id);

            // Kiểm tra xem vùng có tồn tại không
            if (regionDomainModel == null)
            {
                // Trả về 404 Not Found nếu không tìm thấy tài nguyên để cập nhật
                return NotFound();
            }

            // 2. Cập nhật các thuộc tính của Mô hình Miền từ DTO
            // Khi bạn sửa đổi các thuộc tính của regionDomainModel,
            // EF Core Change Tracker sẽ nhận biết các thay đổi này.
            regionDomainModel.Code = updateRegionRequestDto.Code;
            regionDomainModel.Name = updateRegionRequestDto.Name;
            regionDomainModel.RegionImageUrl = updateRegionRequestDto.RegionImageUrl;

            // 3. Lưu các thay đổi vào cơ sở dữ liệu
            // DbContext.SaveChangesAsync() sẽ tạo và thực thi câu lệnh UPDATE SQL
            // dựa trên các thay đổi mà EF Core đã theo dõi.
            await _dbContext.SaveChangesAsync();

            // 4. Ánh xạ Mô hình Miền đã cập nhật trở lại DTO để trả về client
            // Client nhận được phiên bản mới nhất của tài nguyên.
            var regionDto = new RegionDto
            {
                Id = regionDomainModel.Id,
                Code = regionDomainModel.Code,
                Name = regionDomainModel.Name,
                RegionImageUrl = regionDomainModel.RegionImageUrl
            };

            // 5. Trả về 200 OK với DTO đã cập nhật
            // Phản hồi 200 OK là tiêu chuẩn cho cập nhật thành công.
            return Ok(regionDto);
        }

// ...
```

> [!NOTE]
> *   **Động từ HTTP `PUT`:** Được sử dụng để cập nhật toàn bộ tài nguyên. Nó **không an toàn** (thay đổi trạng thái server) nhưng **idempotent**. Idempotent có nghĩa là việc gửi cùng một yêu cầu `PUT` nhiều lần với cùng một dữ liệu sẽ tạo ra cùng một kết quả trên server (chỉ một lần cập nhật, không tạo ra các bản sao).
> *   `[FromRoute]` lấy `id` từ URL path.
> *   `[FromBody]` lấy `updateRegionRequestDto` từ phần thân của yêu cầu.
> *   **Cơ chế cập nhật của EF Core:** Khi bạn truy xuất một thực thể bằng `FirstOrDefaultAsync()`, EF Core bắt đầu theo dõi nó. Mọi thay đổi bạn thực hiện trên các thuộc tính của thực thể đó sẽ được EF Core ghi nhận. Khi bạn gọi `SaveChangesAsync()`, EF Core sẽ tự động tạo và thực thi câu lệnh `UPDATE` SQL phù hợp chỉ với những thuộc tính đã thay đổi.
> *   **HTTP PATCH vs. PUT:** `PUT` thường dùng để thay thế hoàn toàn một tài nguyên. Nếu bạn chỉ muốn cập nhật một phần nhỏ của tài nguyên (ví dụ: chỉ thay đổi `Name` mà không ảnh hưởng đến các trường khác), `PATCH` là động từ phù hợp hơn. `PATCH` yêu cầu một cách tiếp cận khác để xử lý dữ liệu đầu vào (ví dụ: JSON Patch hoặc DTO chỉ chứa các trường tùy chọn).

---

#### Tư duy Vibe Coding: Cập nhật API PUT với Antigravity IDE

Để tạo hoặc chỉnh sửa một API `PUT`, bạn có thể tận dụng Antigravity để đẩy nhanh quá trình.

**Ví dụ:**

*   **Prompt:** "Antigravity, tôi cần một endpoint `PUT /api/regions/{id}` để cập nhật thông tin vùng. ID sẽ đến từ URL, và dữ liệu cập nhật (Code, Name, RegionImageUrl) sẽ đến từ `UpdateRegionRequestDto`. Endpoint này phải tìm vùng theo ID, cập nhật các thuộc tính và trả về `200 OK` với DTO của vùng đã cập nhật. Đừng quên xử lý trường hợp vùng không tồn tại."
*   *Antigravity sẽ:*
    1.  Đề xuất hoặc tạo `UpdateRegionRequestDto` (nếu chưa có), bao gồm validation.
    2.  Tạo phương thức `UpdateRegion` trong `RegionsController` với `[HttpPut]`, `[FromRoute]`, `[FromBody]`.
    3.  Viết logic để tìm vùng bằng `FirstOrDefaultAsync()`.
    4.  Thêm kiểm tra `null` và trả về `NotFound()` nếu vùng không tồn tại.
    5.  Cập nhật các thuộc tính của `regionDomainModel` từ DTO.
    6.  Gọi `_dbContext.SaveChangesAsync()`.
    7.  Ánh xạ ngược lại Domain Model sang `RegionDto` và trả về `Ok()`.

Antigravity giúp bạn đảm bảo rằng tất cả các trường hợp Edge Case (như tài nguyên không tồn tại) được xử lý đúng cách và tuân thủ tiêu chuẩn REST.

---

## 5. Xây dựng API Xóa Vùng (HTTP DELETE)

Để xóa một tài nguyên, chúng ta sử dụng động từ HTTP `DELETE`. Phương thức này chỉ cần ID của tài nguyên cần xóa.

### 5.1. Triển khai phương thức `DeleteRegion` trong Controller

```csharp
// Controllers/RegionsController.cs (tiếp tục)
// ...

        [HttpDelete]
        [Route("{id:Guid}")] // Định nghĩa route với tham số ID kiểu Guid
        public async Task<IActionResult> DeleteRegion([FromRoute] Guid id)
        {
            // 1. Tìm Mô hình Miền hiện có trong cơ sở dữ liệu
            var regionDomainModel = await _dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id);

            // Kiểm tra xem vùng có tồn tại không
            if (regionDomainModel == null)
            {
                // Trả về 404 Not Found nếu không tìm thấy tài nguyên để xóa
                return NotFound();
            }

            // 2. Sử dụng DbContext để xóa Mô hình Miền
            // _dbContext.Regions.Remove() đánh dấu thực thể này để xóa khỏi cơ sở dữ liệu.
            // Nó chưa thực thi câu lệnh DELETE ngay lập tức.
            _dbContext.Regions.Remove(regionDomainModel);
            
            // _dbContext.SaveChangesAsync() thực thi câu lệnh DELETE SQL trong cơ sở dữ liệu.
            await _dbContext.SaveChangesAsync();

            // 3. Ánh xạ Mô hình Miền đã xóa trở lại DTO để trả về client (tùy chọn)
            // Việc trả về đối tượng đã xóa có thể hữu ích để client xác nhận
            // hoặc bạn có thể chỉ trả về 204 No Content.
            var regionDto = new RegionDto
            {
                Id = regionDomainModel.Id,
                Code = regionDomainModel.Code,
                Name = regionDomainModel.Name,
                RegionImageUrl = regionDomainModel.RegionImageUrl
            };

            // 4. Trả về 200 OK với DTO của đối tượng đã xóa hoặc 204 No Content
            // 200 OK: Xóa thành công, kèm theo dữ liệu của đối tượng đã xóa.
            // 204 No Content: Xóa thành công, nhưng không có nội dung nào được trả về trong phản hồi.
            // Cả hai đều là các phản hồi hợp lệ theo quy ước REST.
            return Ok(regionDto); // Hoặc return NoContent(); cho 204
        }
    }
}
```

> [!NOTE]
> *   **Động từ HTTP `DELETE`:** Được sử dụng để xóa một tài nguyên cụ thể. Nó **không an toàn** (thay đổi trạng thái server) và **idempotent** (xóa một tài nguyên nhiều lần với cùng ID sẽ vẫn dẫn đến cùng một trạng thái: tài nguyên đó không còn tồn tại).
> *   `_dbContext.Regions.Remove(regionDomainModel);` đánh dấu thực thể trong bộ theo dõi thay đổi của EF Core là cần được xóa.
> *   `SaveChangesAsync()` thực thi câu lệnh `DELETE` trong cơ sở dữ liệu.

---

#### Tư duy Vibe Coding: Xóa API DELETE với Antigravity IDE

Antigravity có thể giúp bạn tạo các endpoint `DELETE` một cách nhanh chóng và chính xác.

**Ví dụ:**

*   **Prompt:** "Antigravity, tạo cho tôi một endpoint `DELETE /api/regions/{id}` để xóa một vùng theo ID. Nếu vùng không tồn tại, trả về `404 Not Found`. Nếu xóa thành công, trả về `200 OK` với DTO của vùng đã xóa."
*   *Antigravity sẽ:*
    1.  Tạo phương thức `DeleteRegion` trong `RegionsController` với `[HttpDelete]` và `[FromRoute]`.
    2.  Viết logic để tìm vùng bằng `FirstOrDefaultAsync()`.
    3.  Thêm kiểm tra `null` và trả về `NotFound()` nếu vùng không tồn tại.
    4.  Sử dụng `_dbContext.Regions.Remove()` và `_dbContext.SaveChangesAsync()`.
    5.  Ánh xạ Domain Model đã xóa sang `RegionDto` và trả về `Ok()`.
    6.  Antigravity cũng có thể gợi ý về lựa chọn giữa `200 OK` và `204 No Content` và giải thích khi nào nên sử dụng từng loại.

Với Antigravity, bạn có thể nhanh chóng xây dựng các API CRUD đầy đủ chức năng, tuân thủ tiêu chuẩn và có khả năng xử lý lỗi.

---

## 6. Tóm tắt và Hướng tới tương lai

Trong phần này, chúng ta đã xây dựng một nền tảng vững chắc cho việc xử lý các hoạt động ghi dữ liệu (CRUD) trong API RESTful của mình, đồng thời áp dụng các phương pháp hay nhất.

*   **DTOs và Mô hình Miền:** Chúng ta đã hiểu sự khác biệt cốt lõi giữa DTOs (Data Transfer Objects) và Mô hình Miền (Domain Models), nhận ra tầm quan trọng của DTOs trong việc tách biệt mối quan tâm, cải thiện hiệu suất, bảo mật và khả năng lập phiên bản API.
*   **Áp dụng DTOs cho các Hoạt động Đọc:** Chúng ta đã refactor các phương thức `GET` (`GetAllRegions`, `GetRegionById`) để trả về `RegionDto` thay vì trực tiếp Mô hình Miền, thông qua quá trình ánh xạ (thủ công hoặc có thể dùng AutoMapper).
*   **Xây dựng API Tạo Vùng (HTTP POST):** Đã tạo `AddRegionRequestDto` (kèm validation) để nhận dữ liệu từ client. Triển khai phương thức `CreateRegion` sử dụng `[HttpPost]`, `[FromBody]`, `DbContext.AddAsync()`, `DbContext.SaveChangesAsync()`. Sử dụng `CreatedAtAction` để trả về phản hồi `201 Created` cùng với header `Location`.
*   **Xây dựng API Cập nhật Vùng (HTTP PUT):** Đã tạo `UpdateRegionRequestDto` (kèm validation) để nhận dữ liệu cập nhật. Triển khai phương thức `UpdateRegion` sử dụng `[HttpPut]`, `[FromRoute]`, `[FromBody]`. Tìm kiếm thực thể hiện có, cập nhật các thuộc tính và gọi `DbContext.SaveChangesAsync()`. Trả về `200 OK` với DTO đã cập nhật.
*   **Xây dựng API Xóa Vùng (HTTP DELETE):** Triển khai phương thức `DeleteRegion` sử dụng `[HttpDelete]`, `[FromRoute]`. Tìm kiếm thực thể và sử dụng `DbContext.Remove()` cùng với `DbContext.SaveChangesAsync()`. Trả về `200 OK` (có thể kèm DTO đã xóa) hoặc `204 No Content`.
*   **Tư duy Vibe Coding với Antigravity IDE:** Chúng ta đã thấy cách Antigravity có thể hỗ trợ từng bước trong quá trình phát triển API, từ thiết kế DTO đến triển khai các phương thức CRUD, bao gồm cả việc áp dụng validation và xử lý phản hồi HTTP chuẩn.

### Hướng tới tương lai: Repository Pattern và Dependency Injection

Mặc dù việc sử dụng `DbContext` trực tiếp trong Controller là một cách tiếp cận nhanh chóng để bắt đầu, nhưng trong các dự án lớn hơn, nó có thể dẫn đến các vấn đề về khả năng kiểm thử, tách biệt mối quan tâm và tái sử dụng mã. Đây là lúc **Repository Pattern** phát huy tác dụng.

**Repository Pattern là gì và tại sao cần?**
Repository Pattern tạo ra một lớp trừu tượng (abstraction layer) giữa tầng nghiệp vụ (Controller/Service) và tầng truy cập dữ liệu (EF Core `DbContext`). Thay vì controller gọi trực tiếp `_dbContext.Regions.AddAsync()`, nó sẽ gọi `_regionRepository.AddAsync()`.

**Lợi ích:**

*   **Khả năng kiểm thử (Testability):** Controller trở nên độc lập với `DbContext`. Bạn có thể dễ dàng tạo các mock `IRegionRepository` cho các bài kiểm thử đơn vị mà không cần database thực.
*   **Tách biệt Mối quan tâm (SoC):** Controller chỉ quan tâm đến logic API, không quan tâm đến chi tiết triển khai truy cập dữ liệu. Logic truy cập dữ liệu được đóng gói trong Repository.
*   **Khả năng tái sử dụng (Reusability):** Logic truy cập dữ liệu có thể được tái sử dụng qua nhiều Controller hoặc Service.
*   **Linh hoạt trong thay đổi công nghệ:** Nếu bạn quyết định thay đổi công nghệ cơ sở dữ liệu hoặc ORM (ví dụ: từ EF Core sang Dapper), bạn chỉ cần thay đổi việc triển khai Repository mà không ảnh hưởng đến Controller.

**Cách tích hợp (Tổng quan):**

1.  **Định nghĩa Interface Repository:**
    ```csharp
    // Repositories/IRegionRepository.cs
    public interface IRegionRepository
    {
        Task<List<Region>> GetAllAsync();
        Task<Region?> GetByIdAsync(Guid id);
        Task<Region> CreateAsync(Region region);
        Task<Region?> UpdateAsync(Guid id, Region region);
        Task<Region?> DeleteAsync(Guid id);
    }
    ```
2.  **Triển khai Repository:**
    ```csharp
    // Repositories/RegionRepository.cs
    public class RegionRepository : IRegionRepository
    {
        private readonly ApplicationDbContext _dbContext;
        public RegionRepository(ApplicationDbContext dbContext)
        {
            _dbContext = dbContext;
        }
        // Triển khai các phương thức GetAllAsync, GetByIdAsync, v.v. sử dụng _dbContext
        // Ví dụ:
        public async Task<List<Region>> GetAllAsync()
        {
            return await _dbContext.Regions.ToListAsync();
        }
        // ... các phương thức CRUD khác
    }
    ```
3.  **Đăng ký với Dependency Injection:**
    Trong `Program.cs` (hoặc `Startup.cs`), bạn đăng ký Repository để nó có thể được inject vào Controller:
    ```csharp
    builder.Services.AddScoped<IRegionRepository, RegionRepository>();
    ```
4.  **Sử dụng trong Controller:**
    ```csharp
    // Controllers/RegionsController.cs
    // ...
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository _regionRepository; // Inject interface
        // private readonly IMapper _mapper; // (Nếu dùng AutoMapper)

        public RegionsController(IRegionRepository regionRepository /*, IMapper mapper */)
        {
            _regionRepository = regionRepository;
            // _mapper = mapper;
        }

        [HttpGet]
        public async Task<IActionResult> GetAllRegions()
        {
            var regionsDomain = await _regionRepository.GetAllAsync();
            // var regionsDto = _mapper.Map<List<RegionDto>>(regionsDomain); // Dùng AutoMapper
            // ... ánh xạ thủ công nếu không dùng AutoMapper
            return Ok(regionsDto);
        }
        // ... các phương thức CRUD khác cũng gọi _regionRepository thay vì _dbContext
    }
    ```

**Tư duy Vibe Coding tổng thể với Antigravity IDE:**
Antigravity không chỉ là một công cụ sinh mã. Nó là một đối tác AI giúp bạn *thiết kế* và *lý giải* các quyết định kiến trúc.

*   **Thiết kế kiến trúc:** Bạn có thể hỏi Antigravity: "Tôi nên áp dụng Repository Pattern cho tài nguyên Vùng không? Giải thích ưu và nhược điểm."
*   **Sinh mã Repository:** Sau khi quyết định, bạn có thể yêu cầu: "Antigravity, tạo cho tôi `IRegionRepository` và triển khai `RegionRepository` với các phương thức CRUD cơ bản sử dụng `ApplicationDbContext`."
*   **Refactor Controller:** "Bây giờ, hãy refactor `RegionsController` để sử dụng `IRegionRepository` đã tạo."
*   **Giải thích cơ chế:** "Giải thích cách Dependency Injection hoạt động khi tôi inject `IRegionRepository` vào Controller."

Antigravity IDE với khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file, và lập kế hoạch tự động sẽ giúp bạn biến các ý tưởng kiến trúc phức tạp thành mã nguồn thực tế một cách hiệu quả, đồng thời cung cấp các giải thích sâu sắc để củng cố kiến thức của bạn. Với những kiến thức này, bạn đã có thể xây dựng một API RESTful đầy đủ chức năng, tuân thủ các nguyên tắc thiết kế tốt và dễ dàng mở rộng trong tương lai.

<!-- REVIEWED_BY_AGENT -->
