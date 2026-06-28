# Phần 23: API Vùng (Regions) - Triển khai Thao tác Cập nhật và Xóa

Trong quá trình xây dựng một RESTful Web API hoàn chỉnh, việc quản lý vòng đời của tài nguyên là trọng tâm. Sau khi đã thiết lập các thao tác tạo (Create) và đọc (Read) tài nguyên, chúng ta cần bổ sung khả năng sửa đổi (Update) và loại bỏ (Delete) chúng. Phần này sẽ đi sâu vào việc triển khai hai thao tác quan trọng này cho API Vùng (Regions) của chúng ta, sử dụng ASP.NET Core, Entity Framework Core, Repository Pattern và Dependency Injection.

Mục tiêu chính của phần này là:
*   Nắm vững nguyên tắc và thực hành triển khai thao tác xóa tài nguyên (`DELETE`) trong Repository và Controller.
*   Nắm vững nguyên tắc và thực hành triển khai thao tác cập nhật tài nguyên (`PUT`) trong Repository và Controller.
*   Hiểu rõ cách thiết kế Data Transfer Objects (DTOs) cho yêu cầu cập nhật để đảm bảo tính bảo mật và linh hoạt.
*   Sử dụng Swagger UI để kiểm thử các endpoint API `DELETE` và `PUT`, đồng thời hiểu các mã trạng thái HTTP tương ứng.
*   Củng cố kiến thức về ánh xạ giữa DTO và Domain Model, cũng như vai trò của các HTTP Verbs trong thiết kế RESTful API.

---

## I. Nền tảng RESTful API cho Thao tác Cập nhật và Xóa

Kiến trúc REST (Representational State Transfer) định nghĩa một tập hợp các nguyên tắc để xây dựng các dịch vụ web. Một trong những nguyên tắc cốt lõi là sử dụng các HTTP Verbs tiêu chuẩn để thực hiện các thao tác trên tài nguyên. Đối với việc cập nhật và xóa, chúng ta tập trung vào `DELETE` và `PUT`.

### 1.1. HTTP DELETE: Xóa tài nguyên

HTTP `DELETE` verb được sử dụng để yêu cầu máy chủ xóa một tài nguyên cụ thể được xác định bởi URI.

*   **Tính chất:** `DELETE` được coi là **idempotent**. Điều này có nghĩa là việc gửi cùng một yêu cầu `DELETE` nhiều lần sẽ có cùng một hiệu ứng như khi gửi một lần (tức là tài nguyên sẽ bị xóa một lần duy nhất, các lần gửi sau sẽ không tìm thấy tài nguyên để xóa).
*   **Phản hồi thành công:**
    *   `200 OK`: Nếu yêu cầu thành công và có nội dung trả về (ví dụ: thông tin của tài nguyên vừa bị xóa).
    *   `204 No Content`: Đây là phản hồi phổ biến và được khuyến nghị khi xóa thành công mà không cần trả về bất kỳ nội dung nào trong phần thân phản hồi. Điều này giúp tối ưu băng thông và đơn giản hóa logic phía client.
*   **Phản hồi lỗi:**
    *   `404 Not Found`: Nếu tài nguyên được chỉ định bởi URI không tồn tại trên máy chủ.
    *   `403 Forbidden` hoặc `401 Unauthorized`: Nếu client không có quyền xóa tài nguyên.
    *   `409 Conflict`: Nếu tài nguyên không thể bị xóa do các ràng buộc hoặc phụ thuộc khác (ít phổ biến hơn cho DELETE đơn giản).

### 1.2. HTTP PUT: Cập nhật toàn bộ tài nguyên

HTTP `PUT` verb được sử dụng để thay thế hoàn toàn một tài nguyên hiện có bằng một tài nguyên mới được cung cấp trong phần thân yêu cầu. Nếu tài nguyên không tồn tại, `PUT` có thể được sử dụng để tạo mới (upsert), nhưng trong bối cảnh API của chúng ta, nó sẽ được dùng để cập nhật một tài nguyên đã tồn tại.

*   **Tính chất:** `PUT` cũng được coi là **idempotent**. Nếu bạn gửi cùng một yêu cầu `PUT` với cùng một dữ liệu nhiều lần, kết quả trên máy chủ sẽ luôn giống nhau (tài nguyên sẽ được cập nhật thành trạng thái đó).
*   **Phản hồi thành công:**
    *   `200 OK`: Nếu yêu cầu thành công và có nội dung trả về (thường là tài nguyên đã được cập nhật).
    *   `204 No Content`: Nếu yêu cầu thành công và không có nội dung trả về (ít phổ biến hơn cho `PUT` vì client thường muốn xem tài nguyên đã cập nhật).
    *   `201 Created`: Nếu `PUT` được sử dụng để tạo một tài nguyên mới (upsert).
*   **Phản hồi lỗi:**
    *   `404 Not Found`: Nếu tài nguyên được chỉ định bởi URI để cập nhật không tồn tại.
    *   `400 Bad Request`: Nếu dữ liệu trong phần thân yêu cầu không hợp lệ hoặc thiếu.
    *   `403 Forbidden` hoặc `401 Unauthorized`: Nếu client không có quyền cập nhật tài nguyên.

> [!NOTE]
> **PUT vs. PATCH:**
> *   `PUT` thay thế **toàn bộ** tài nguyên. Mọi thuộc tính của tài nguyên sẽ được cập nhật dựa trên dữ liệu gửi lên. Nếu một thuộc tính không được cung cấp trong yêu cầu `PUT`, nó sẽ bị đặt thành giá trị mặc định hoặc `null` (tùy thuộc vào cách triển khai).
> *   `PATCH` chỉ áp dụng các thay đổi **một phần** cho tài nguyên. Nó chỉ sửa đổi các thuộc tính được chỉ định trong yêu cầu, giữ nguyên các thuộc tính khác.
> Trong phần này, chúng ta sẽ tập trung vào `PUT` để thay thế hoàn toàn tài nguyên, là một kịch bản phổ biến và đơn giản hơn để bắt đầu.

---

## II. Triển khai Chức năng Xóa Vùng (DELETE /api/Regions/{id})

Việc triển khai chức năng xóa một vùng (Region) cụ thể tuân thủ chặt chẽ Repository Pattern và nguyên tắc Dependency Injection, giúp tách biệt logic nghiệp vụ khỏi tầng truy cập dữ liệu và tầng trình bày (Controller).

### 2.1. Định nghĩa Hợp đồng Repository: `IRegionRepository`

Bước đầu tiên là khai báo một phương thức xóa không đồng bộ trong giao diện `IRegionRepository`. Giao diện này đóng vai trò là hợp đồng, định nghĩa những gì Repository có thể làm mà không tiết lộ chi tiết triển khai.

```csharp
// IRegionRepository.cs
namespace NZWalks.API.Repositories
{
    public interface IRegionRepository
    {
        // ... các phương thức khác (GetAllAsync, GetByIdAsync, CreateAsync) ...

        /// <summary>
        /// Xóa một vùng dựa trên ID.
        /// </summary>
        /// <param name="id">ID của vùng cần xóa.</param>
        /// <returns>Đối tượng Region đã xóa nếu tìm thấy, ngược lại là null.</returns>
        Task<Region?> DeleteAsync(Guid id);
    }
}
```

*   **Giải thích:** Phương thức `DeleteAsync` nhận một `Guid id` làm tham số để xác định vùng cần xóa. Nó trả về `Task<Region?>`. Việc trả về `Region?` (Region có thể null) giúp tầng Controller biết được liệu có tìm thấy và xóa được vùng đó hay không, đồng thời cung cấp thông tin về tài nguyên đã xóa nếu cần. Đây là một lựa chọn thiết kế linh hoạt, cho phép client nhận được xác nhận với dữ liệu cụ thể.

### 2.2. Chi tiết Triển khai Logic Xóa: `RegionRepository`

Tiếp theo, chúng ta cung cấp triển khai cụ thể cho phương thức `DeleteAsync` trong lớp `RegionRepository`. Đây là nơi logic tương tác với Entity Framework Core và cơ sở dữ liệu diễn ra.

```csharp
// RegionRepository.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public class RegionRepository : IRegionRepository
    {
        private readonly NZWalksDbContext dbContext;

        // Dependency Injection: NZWalksDbContext được inject vào constructor
        public RegionRepository(NZWalksDbContext dbContext)
        {
            this.dbContext = dbContext;
        }

        // ... các phương thức khác ...

        /// <summary>
        /// Triển khai việc xóa một vùng dựa trên ID.
        /// </summary>
        /// <param name="id">ID của vùng cần xóa.</param>
        /// <returns>Đối tượng Region đã xóa nếu tìm thấy, ngược lại là null.</returns>
        public async Task<Region?> DeleteAsync(Guid id)
        {
            // 1. Tìm kiếm vùng trong cơ sở dữ liệu bằng ID
            // FirstOrDefaultAsync sẽ trả về null nếu không tìm thấy bất kỳ vùng nào khớp.
            // Điều này hiệu quả hơn FindAsync nếu chúng ta không chắc chắn về sự tồn tại của đối tượng.
            var existingRegion = await dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id);

            // 2. Kiểm tra xem vùng có tồn tại không
            if (existingRegion == null)
            {
                return null; // Trả về null nếu không tìm thấy vùng, báo hiệu không thể xóa
            }

            // 3. Xóa vùng khỏi DbContext
            // dbContext.Regions.Remove() chỉ đánh dấu đối tượng là "Deleted" trong Change Tracker
            // của Entity Framework Core. Thay đổi này chưa được lưu vào cơ sở dữ liệu.
            dbContext.Regions.Remove(existingRegion);

            // 4. Lưu các thay đổi vào cơ sở dữ liệu
            // SaveChangesAsync() thực thi các lệnh SQL DELETE tương ứng với các đối tượng
            // đã được đánh dấu là "Deleted" trong DbContext. Đây là thao tác I/O bất đồng bộ.
            await dbContext.SaveChangesAsync();

            // 5. Trả về vùng đã xóa
            // Việc trả về đối tượng đã xóa có thể hữu ích cho client nếu họ muốn hiển thị
            // thông tin của tài nguyên vừa bị loại bỏ.
            return existingRegion;
        }
    }
}
```

*   **Cơ chế ngầm (Under the Hood):**
    *   `FirstOrDefaultAsync(x => x.Id == id)`: Đây là cách EF Core tìm kiếm một bản ghi trong cơ sở dữ liệu. Nó sẽ tạo một câu lệnh SQL `SELECT TOP 1 * FROM Regions WHERE Id = @id` và thực thi nó. Nếu không tìm thấy, nó trả về `null` thay vì ném ngoại lệ.
    *   `dbContext.Regions.Remove(existingRegion)`: Khi bạn gọi phương thức này, Entity Framework Core sẽ không ngay lập tức xóa bản ghi khỏi cơ sở dữ liệu. Thay vào đó, nó theo dõi đối tượng `existingRegion` trong `Change Tracker` và đánh dấu trạng thái của nó là `Deleted`.
    *   `await dbContext.SaveChangesAsync()`: Đây là thời điểm thực sự các thay đổi được áp dụng vào cơ sở dữ liệu. EF Core sẽ kiểm tra tất cả các đối tượng trong `Change Tracker` có trạng thái `Added`, `Modified`, hoặc `Deleted` và tạo ra các câu lệnh SQL `INSERT`, `UPDATE`, `DELETE` tương ứng, sau đó thực thi chúng trong một transaction để đảm bảo tính toàn vẹn dữ liệu.

### 2.3. Xây dựng Endpoint API Xóa: `RegionsController`

Cuối cùng, chúng ta tạo một endpoint API trong `RegionsController` để xử lý các yêu cầu HTTP DELETE. Endpoint này sẽ tiếp nhận ID của vùng cần xóa từ URL.

```csharp
// RegionsController.cs
using AutoMapper; // Sẽ được cấu hình và sử dụng sau này
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")] // Định nghĩa route cơ sở cho Controller này
    [ApiController] // Đánh dấu đây là một Controller API
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository regionRepository;
        private readonly IMapper mapper; // Được inject nếu sử dụng AutoMapper

        // Dependency Injection: IRegionRepository và IMapper được inject vào constructor
        public RegionsController(IRegionRepository regionRepository, IMapper mapper)
        {
            this.regionRepository = regionRepository;
            this.mapper = mapper;
        }

        // ... các phương thức Controller khác ...

        /// <summary>
        /// Xóa một vùng cụ thể dựa trên ID.
        /// </summary>
        /// <param name="id">ID của vùng cần xóa (lấy từ Route).</param>
        /// <returns>Thông tin vùng đã xóa hoặc 404 Not Found nếu không tìm thấy.</returns>
        [HttpDelete] // Chỉ định rằng phương thức này xử lý yêu cầu HTTP DELETE
        [Route("{id:Guid}")] // Định nghĩa route cho endpoint này, với tham số ID là GUID
        public async Task<IActionResult> DeleteRegionAsync([FromRoute] Guid id)
        {
            // 1. Gọi phương thức DeleteAsync từ Repository để xóa vùng
            // Controller không cần biết chi tiết logic xóa, chỉ cần gọi Repository.
            var deletedRegionDomain = await regionRepository.DeleteAsync(id);

            // 2. Kiểm tra nếu vùng không tồn tại (Repository trả về null)
            if (deletedRegionDomain == null)
            {
                // Trả về mã trạng thái HTTP 404 Not Found
                return NotFound();
            }

            // 3. Chuyển đổi Domain Model thành DTO để trả về cho Client
            // Mặc dù tài nguyên đã bị xóa, việc trả về DTO của tài nguyên đó trong phản hồi
            // 200 OK có thể hữu ích để client xác nhận và cập nhật UI.
            // Ở đây đang ánh xạ thủ công, sau này sẽ dùng AutoMapper.
            var regionDto = new RegionDto
            {
                Id = deletedRegionDomain.Id,
                Code = deletedRegionDomain.Code,
                Name = deletedRegionDomain.Name,
                Area = deletedRegionDomain.Area,
                Lat = deletedRegionDomain.Lat,
                Long = deletedRegionDomain.Long,
                Population = deletedRegionDomain.Population
            };

            // 4. Trả về mã trạng thái HTTP 200 OK cùng với thông tin vùng đã xóa
            return Ok(regionDto);
        }
    }
}
```

*   **Giải thích chi tiết:**
    *   `[HttpDelete]`: Attribute này chỉ định rằng phương thức `DeleteRegionAsync` sẽ xử lý các yêu cầu HTTP `DELETE`.
    *   `[Route("{id:Guid}")]`: Attribute này mở rộng route cơ sở của controller. Nó định nghĩa một tham số `id` sẽ được lấy từ URL. Cú pháp `:Guid` là một ràng buộc route (route constraint), đảm bảo rằng giá trị của `id` phải là một `Guid` hợp lệ. Nếu không, yêu cầu sẽ không khớp với route này.
    *   `[FromRoute] Guid id`: Attribute `[FromRoute]` hướng dẫn ASP.NET Core lấy giá trị cho tham số `id` từ các tham số trong URL route.
    *   `IActionResult`: Đây là kiểu trả về phổ biến cho các phương thức Controller trong ASP.NET Core API, cho phép trả về các mã trạng thái HTTP khác nhau (ví dụ: `Ok()`, `NotFound()`, `BadRequest()`).
    *   **Dependency Injection:** `IRegionRepository` được inject vào constructor của `RegionsController`. Điều này có nghĩa là controller không cần biết cách tạo ra một `RegionRepository` cụ thể; nó chỉ yêu cầu một đối tượng triển khai `IRegionRepository`, và hệ thống DI sẽ cung cấp nó. Điều này giúp tăng tính mô-đun và khả năng kiểm thử.

### 2.4. Kiểm thử Chức năng Xóa với Swagger UI

Sau khi triển khai, hãy chạy ứng dụng và truy cập Swagger UI (thường là `https://localhost:port/swagger`).

1.  **Lấy ID của một vùng:**
    *   Mở rộng endpoint `GET /api/Regions`.
    *   Nhấp vào "Try it out" và sau đó "Execute".
    *   Trong phần "Responses", sao chép `Id` (một giá trị GUID) của một vùng bạn muốn xóa.
2.  **Thực hiện yêu cầu DELETE:**
    *   Mở rộng endpoint `DELETE /api/Regions/{id}`.
    *   Nhấp vào "Try it out".
    *   Dán `Id` đã sao chép vào trường `id`.
    *   Nhấp vào "Execute".
3.  **Kiểm tra kết quả:**
    *   **Trường hợp thành công:** Bạn sẽ nhận được `200 OK` cùng với thông tin của vùng đã bị xóa trong phần "Response body".
    *   **Trường hợp thất bại (tài nguyên không tồn tại):** Cố gắng xóa cùng một `Id` lần nữa. Bạn sẽ nhận được `404 Not Found` vì tài nguyên đó đã không còn tồn tại.
    *   **Xác nhận:** Gọi lại `GET /api/Regions` để kiểm tra xem vùng đó có còn trong danh sách hay không.

---

## III. Triển khai Chức năng Cập nhật Vùng (PUT /api/Regions/{id})

Tương tự như chức năng xóa, việc cập nhật một vùng cũng tuân theo Repository Pattern. Tuy nhiên, thao tác cập nhật đòi hỏi một DTO riêng biệt cho dữ liệu đầu vào để kiểm soát tốt hơn các trường được phép sửa đổi.

### 3.1. Thiết kế Data Transfer Object (DTO) cho Cập nhật: `UpdateRegionRequestDto`

Khi cập nhật một tài nguyên, chúng ta không nên trực tiếp sử dụng Domain Model làm dữ liệu đầu vào từ client. Thay vào đó, chúng ta sử dụng Data Transfer Object (DTO) để kiểm soát dữ liệu nào được phép cập nhật và để tách biệt Domain Model khỏi các yêu cầu API.

```csharp
// UpdateRegionRequestDto.cs
using System.ComponentModel.DataAnnotations; // Để thêm Validation Attributes

namespace NZWalks.API.Models.DTO
{
    public class UpdateRegionRequestDto
    {
        // ID không cần có ở đây vì nó sẽ được lấy từ URL route
        // Các thuộc tính mà client có thể cập nhật
        [Required] // Đảm bảo trường này không được để trống
        [MinLength(3, ErrorMessage = "Code phải có ít nhất 3 ký tự")]
        [MaxLength(3, ErrorMessage = "Code không được vượt quá 3 ký tự")]
        public string Code { get; set; }

        [Required]
        [MaxLength(100, ErrorMessage = "Tên không được vượt quá 100 ký tự")]
        public string Name { get; set; }

        [Range(0, 1000000, ErrorMessage = "Diện tích phải nằm trong khoảng 0 đến 1,000,000")]
        public double Area { get; set; }

        [Range(-90, 90, ErrorMessage = "Vĩ độ phải nằm trong khoảng -90 đến 90")]
        public double Lat { get; set; }

        [Range(-180, 180, ErrorMessage = "Kinh độ phải nằm trong khoảng -180 đến 180")]
        public double Long { get; set; }

        [Range(0, 1000000000, ErrorMessage = "Dân số phải nằm trong khoảng 0 đến 1,000,000,000")]
        public long Population { get; set; }
    }
}
```

*   **Giải thích về DTO và Validation:**
    *   **Tách biệt và Bảo mật:** `UpdateRegionRequestDto` chỉ chứa các thuộc tính mà client được phép sửa đổi. Điều này ngăn chặn việc client vô tình hoặc cố ý cập nhật các trường nhạy cảm hoặc không mong muốn (ví dụ: `Id` là khóa chính, hoặc các trường chỉ đọc khác). `Id` của tài nguyên được lấy từ URL route, không phải từ request body.
    *   **Linh hoạt:** Cho phép định dạng yêu cầu cập nhật khác với cấu trúc Domain Model nội bộ.
    *   **Validation (Xác thực):** Các thuộc tính như `[Required]`, `[MinLength]`, `[MaxLength]`, `[Range]` là các Data Annotation được sử dụng để xác thực dữ liệu đầu vào. ASP.NET Core tự động kiểm tra các ràng buộc này khi model được binding từ request body. Nếu dữ liệu không hợp lệ, `ModelState.IsValid` trong Controller sẽ là `false`, và chúng ta có thể trả về `400 Bad Request` với thông tin lỗi. (Chúng ta sẽ bổ sung logic kiểm tra `ModelState.IsValid` vào Controller sau).

### 3.2. Định nghĩa Hợp đồng Repository: `IRegionRepository`

Tiếp theo, chúng ta khai báo phương thức cập nhật không đồng bộ trong giao diện `IRegionRepository`.

```csharp
// IRegionRepository.cs
namespace NZWalks.API.Repositories
{
    public interface IRegionRepository
    {
        // ... các phương thức khác ...

        /// <summary>
        /// Cập nhật thông tin của một vùng dựa trên ID.
        /// </summary>
        /// <param name="id">ID của vùng cần cập nhật.</param>
        /// <param name="region">Đối tượng Region chứa dữ liệu mới để cập nhật.</param>
        /// <returns>Đối tượng Region đã cập nhật nếu tìm thấy, ngược lại là null.</returns>
        Task<Region?> UpdateAsync(Guid id, Region region);
    }
}
```

*   **Giải thích:** Phương thức `UpdateAsync` nhận `Guid id` (để tìm vùng) và một đối tượng `Region` (chứa dữ liệu mới để áp dụng). Nó trả về `Task<Region?>`, tương tự như `DeleteAsync`, để báo hiệu kết quả cập nhật.

### 3.3. Chi tiết Triển khai Logic Cập nhật: `RegionRepository`

Bây giờ, chúng ta sẽ triển khai logic cập nhật trong lớp `RegionRepository`. Logic này bao gồm: tìm kiếm vùng theo ID, nếu tìm thấy thì cập nhật các thuộc tính của nó bằng dữ liệu mới và lưu các thay đổi.

```csharp
// RegionRepository.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;

namespace NZWalks.API.Repositories
{
    public class RegionRepository : IRegionRepository
    {
        private readonly NZWalksDbContext dbContext;

        public RegionRepository(NZWalksDbContext dbContext)
        {
            this.dbContext = dbContext;
        }

        // ... các phương thức khác ...

        /// <summary>
        /// Triển khai việc cập nhật một vùng dựa trên ID.
        /// </summary>
        /// <param name="id">ID của vùng cần cập nhật.</param>
        /// <param name="region">Đối tượng Region chứa dữ liệu mới.</param>
        /// <returns>Đối tượng Region đã cập nhật nếu tìm thấy, ngược lại là null.</returns>
        public async Task<Region?> UpdateAsync(Guid id, Region region)
        {
            // 1. Tìm kiếm vùng hiện có trong cơ sở dữ liệu
            var existingRegion = await dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id);

            // 2. Kiểm tra xem vùng có tồn tại không
            if (existingRegion == null)
            {
                return null; // Trả về null nếu không tìm thấy vùng
            }

            // 3. Cập nhật các thuộc tính của vùng hiện có bằng dữ liệu mới từ tham số 'region'
            // Đảm bảo không cập nhật ID vì nó là khóa chính và không thay đổi.
            existingRegion.Code = region.Code;
            existingRegion.Name = region.Name;
            existingRegion.Area = region.Area;
            existingRegion.Lat = region.Lat;
            existingRegion.Long = region.Long;
            existingRegion.Population = region.Population;

            // 4. Lưu các thay đổi vào cơ sở dữ liệu
            // Entity Framework Core sẽ tự động phát hiện các thuộc tính đã thay đổi
            // trên existingRegion và tạo câu lệnh SQL UPDATE tương ứng.
            await dbContext.SaveChangesAsync();

            // 5. Trả về vùng đã cập nhật
            return existingRegion;
        }
    }
}
```

*   **Cơ chế ngầm (Under the Hood):**
    *   `existingRegion.Code = region.Code;` và các dòng tương tự: Khi bạn gán giá trị mới cho các thuộc tính của `existingRegion` (mà đã được theo dõi bởi `DbContext`), Entity Framework Core sẽ tự động đánh dấu đối tượng này là `Modified` trong `Change Tracker`.
    *   `await dbContext.SaveChangesAsync()`: Khi phương thức này được gọi, EF Core sẽ kiểm tra các thuộc tính nào của `existingRegion` đã thay đổi so với trạng thái ban đầu khi nó được truy vấn từ cơ sở dữ liệu. Sau đó, nó sẽ tạo một câu lệnh SQL `UPDATE` chỉ bao gồm các cột đã thay đổi để cập nhật bản ghi trong cơ sở dữ liệu. Điều này hiệu quả hơn là cập nhật tất cả các cột.

### 3.4. Xây dựng Endpoint API Cập nhật: `RegionsController`

Bây giờ, chúng ta sẽ tạo một endpoint API trong `RegionsController` để xử lý các yêu cầu HTTP PUT. Endpoint này sẽ nhận ID của vùng từ URL và dữ liệu cập nhật từ phần thân yêu cầu.

```csharp
// RegionsController.cs
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository regionRepository;
        private readonly IMapper mapper;

        public RegionsController(IRegionRepository regionRepository, IMapper mapper)
        {
            this.regionRepository = regionRepository;
            this.mapper = mapper;
        }

        // ... các phương thức Controller khác ...

        /// <summary>
        /// Cập nhật thông tin của một vùng cụ thể dựa trên ID.
        /// </summary>
        /// <param name="id">ID của vùng cần cập nhật (lấy từ Route).</param>
        /// <param name="updateRegionRequestDto">Dữ liệu cập nhật vùng (lấy từ Body).</param>
        /// <returns>Thông tin vùng đã cập nhật hoặc 404 Not Found nếu không tìm thấy.</returns>
        [HttpPut] // Chỉ định rằng phương thức này xử lý yêu cầu HTTP PUT
        [Route("{id:Guid}")] // Định nghĩa route với tham số ID là GUID
        public async Task<IActionResult> UpdateRegionAsync(
            [FromRoute] Guid id, // Lấy ID từ URL route
            [FromBody] UpdateRegionRequestDto updateRegionRequestDto) // Lấy dữ liệu cập nhật từ request body
        {
            // Bước 1: Kiểm tra tính hợp lệ của dữ liệu đầu vào (Validation)
            // ModelState.IsValid sẽ là false nếu các Data Annotations trong UpdateRegionRequestDto không hợp lệ
            if (!ModelState.IsValid)
            {
                // Trả về 400 Bad Request với thông tin lỗi validation
                return BadRequest(ModelState);
            }

            // 2. Chuyển đổi DTO yêu cầu cập nhật thành Domain Model
            // Controller chịu trách nhiệm chuyển đổi dữ liệu từ tầng trình bày sang tầng nghiệp vụ.
            // Hiện tại đang ánh xạ thủ công, sau này sẽ dùng AutoMapper.
            var regionDomainModel = new Region
            {
                // ID của Domain Model sẽ được lấy từ Route, không phải từ DTO.
                // Các thuộc tính khác được lấy từ DTO.
                Code = updateRegionRequestDto.Code,
                Name = updateRegionRequestDto.Name,
                Area = updateRegionRequestDto.Area,
                Lat = updateRegionRequestDto.Lat,
                Long = updateRegionRequestDto.Long,
                Population = updateRegionRequestDto.Population
            };

            // 3. Gọi phương thức UpdateAsync từ Repository để cập nhật vùng
            var updatedRegionDomain = await regionRepository.UpdateAsync(id, regionDomainModel);

            // 4. Kiểm tra nếu vùng không tồn tại (Repository trả về null)
            if (updatedRegionDomain == null)
            {
                return NotFound(); // Trả về 404 Not Found
            }

            // 5. Chuyển đổi Domain Model đã cập nhật thành DTO để trả về cho Client
            // Đảm bảo phản hồi nhất quán và chỉ chứa các thuộc tính mong muốn.
            // Hiện tại đang ánh xạ thủ công, sau này sẽ dùng AutoMapper.
            var regionDto = new RegionDto
            {
                Id = updatedRegionDomain.Id,
                Code = updatedRegionDomain.Code,
                Name = updatedRegionDomain.Name,
                Area = updatedRegionDomain.Area,
                Lat = updatedRegionDomain.Lat,
                Long = updatedRegionDomain.Long,
                Population = updatedRegionDomain.Population
            };

            // 6. Trả về 200 OK cùng với thông tin vùng đã cập nhật
            return Ok(regionDto);
        }
    }
}
```

*   **Giải thích chi tiết:**
    *   `[HttpPut]`: Attribute này chỉ định rằng phương thức `UpdateRegionAsync` sẽ xử lý các yêu cầu HTTP `PUT`.
    *   `[FromBody] UpdateRegionRequestDto updateRegionRequestDto`: Attribute `[FromBody]` hướng dẫn ASP.NET Core lấy dữ liệu cho tham số `updateRegionRequestDto` từ phần thân (body) của yêu cầu HTTP. ASP.NET Core sẽ tự động deserializes JSON (hoặc XML) trong body thành một đối tượng `UpdateRegionRequestDto`.
    *   **Validation (`ModelState.IsValid`):** Việc kiểm tra `ModelState.IsValid` là cực kỳ quan trọng. Nó kiểm tra xem dữ liệu được gửi trong `updateRegionRequestDto` có tuân thủ các Data Annotations (`[Required]`, `[Range]`, v.v.) đã định nghĩa hay không. Nếu không hợp lệ, `BadRequest(ModelState)` sẽ trả về mã trạng thái `400 Bad Request` cùng với chi tiết lỗi, giúp client hiểu rõ vấn đề.
    *   **Mapping DTO to Domain Model:** Trước khi gọi Repository, dữ liệu từ `UpdateRegionRequestDto` phải được ánh xạ sang `Region` Domain Model. Controller không nên trực tiếp truyền DTO vào Repository vì Repository nên làm việc với Domain Model.

*   **Khuyến nghị: Sử dụng AutoMapper**
    Trong các dự án thực tế, việc ánh xạ thủ công giữa DTO và Domain Model (như `var regionDomainModel = new Region { ... }` và `var regionDto = new RegionDto { ... }`) có thể trở nên tẻ nhạt, dễ mắc lỗi và khó bảo trì khi số lượng thuộc tính tăng lên.
    **AutoMapper** là một thư viện phổ biến giúp tự động hóa quá trình ánh xạ này. Thay vì viết từng dòng gán, bạn chỉ cần định nghĩa các profile ánh xạ một lần, sau đó sử dụng `_mapper.Map<DestinationType>(sourceObject)` để thực hiện ánh xạ.
    Mặc dù ở đây chúng ta đang thực hiện ánh xạ thủ công theo nội dung bài giảng để làm rõ cơ chế, việc tìm hiểu và áp dụng AutoMapper là một khuyến nghị mạnh mẽ cho các dự án lớn hơn, giúp giảm thiểu mã lặp lại (boilerplate code) và tăng cường khả năng bảo trì.

### 3.5. Kiểm thử Chức năng Cập nhật với Swagger UI

Sau khi triển khai, hãy chạy ứng dụng và truy cập Swagger UI.

1.  **Lấy ID và dữ liệu hiện có của một vùng:**
    *   Sử dụng endpoint `GET /api/Regions` để lấy danh sách các vùng.
    *   Chọn một vùng bạn muốn cập nhật và sao chép `Id` của nó cùng với các dữ liệu hiện có (Code, Name, Area, v.v.).
2.  **Thực hiện yêu cầu PUT:**
    *   Mở rộng endpoint `PUT /api/Regions/{id}`.
    *   Nhấp vào "Try it out".
    *   Dán `Id` đã sao chép vào trường `id` trong URL.
    *   Trong phần "Request body", dán dữ liệu JSON của vùng bạn muốn cập nhật.
        *   **QUAN TRỌNG:** Đảm bảo **loại bỏ trường `Id`** khỏi JSON body vì `UpdateRegionRequestDto` không chứa nó.
        *   Thay đổi một vài giá trị (ví dụ: `Code`, `Name`, `Population`) trong JSON body.
        *   Ví dụ JSON body:
            ```json
            {
              "code": "AUCK",
              "name": "Auckland Updated",
              "area": 12345.67,
              "lat": -36.8485,
              "long": 174.7633,
              "population": 1650000
            }
            ```
    *   Nhấp vào "Execute".
3.  **Kiểm tra kết quả:**
    *   **Trường hợp thành công:** Bạn sẽ nhận được `200 OK` cùng với thông tin của vùng đã được cập nhật trong phần "Response body". Bạn sẽ thấy các giá trị đã thay đổi trong phản hồi.
    *   **Trường hợp thất bại (tài nguyên không tồn tại):** Cung cấp một `Id` không tồn tại trong URL. Bạn sẽ nhận được `404 Not Found`.
    *   **Trường hợp thất bại (dữ liệu không hợp lệ):** Cung cấp dữ liệu JSON không hợp lệ (ví dụ: `Code` quá dài, `Name` trống). Bạn sẽ nhận được `400 Bad Request` cùng với chi tiết lỗi validation.
    *   **Xác nhận:** Gọi lại `GET /api/Regions` và kiểm tra xem vùng đó đã được cập nhật với các giá trị mới trong danh sách hay chưa.

---

## IV. Quản lý Mã nguồn với Git

Sau khi hoàn thành việc triển khai và kiểm thử các chức năng xóa và cập nhật, việc lưu trữ các thay đổi vào hệ thống kiểm soát phiên bản (Git) là bước không thể thiếu trong quy trình phát triển chuyên nghiệp.

1.  **Thêm các tệp đã thay đổi vào staging area:**
    ```bash
    git add .
    ```
    Lệnh này sẽ thêm tất cả các tệp mới hoặc đã sửa đổi vào staging area, chuẩn bị cho việc commit.
2.  **Tạo một commit với thông điệp rõ ràng:**
    ```bash
    git commit -m "feat(regions): Implement DELETE and PUT operations for Regions API including DTOs and validation"
    ```
    Thông điệp commit nên mô tả ngắn gọn và chính xác những thay đổi bạn đã thực hiện. Việc sử dụng tiền tố như `feat` (feature) giúp phân loại loại thay đổi.
3.  **Đẩy các thay đổi đã commit lên repository từ xa:**
    ```bash
    git push
    ```
    Lệnh này sẽ đẩy các commit của bạn từ local repository lên GitHub, GitLab hoặc repository từ xa khác, đảm bảo công việc của bạn được lưu trữ an toàn, có thể theo dõi và chia sẻ với những người khác trong nhóm phát triển.

---

## V. Tóm tắt Chương

Trong Phần 23 này, chúng ta đã hoàn tất việc xây dựng các thao tác quản lý vòng đời tài nguyên cho API Vùng (Regions) bằng cách triển khai chức năng cập nhật và xóa.

*   Chúng ta đã định nghĩa và triển khai phương thức `DeleteAsync` trong `IRegionRepository` và `RegionRepository`, hiểu rõ cơ chế của Entity Framework Core khi xóa dữ liệu.
*   Chúng ta đã tạo một endpoint `[HttpDelete]` trong `RegionsController` để xử lý các yêu cầu xóa, trả về `200 OK` (với tài nguyên đã xóa) hoặc `404 Not Found`, và nắm vững cách sử dụng `[FromRoute]` và `[Route("{id:Guid}")]`.
*   Chúng ta đã thiết kế `UpdateRegionRequestDto` làm DTO đầu vào cho yêu cầu cập nhật, nhấn mạnh tầm quan trọng của nó trong việc tách biệt Domain Model, tăng cường bảo mật và cho phép validation dữ liệu đầu vào.
*   Chúng ta đã định nghĩa và triển khai phương thức `UpdateAsync` trong `IRegionRepository` và `RegionRepository`, tìm hiểu cách EF Core theo dõi và cập nhật các thuộc tính đã thay đổi.
*   Chúng ta đã tạo một endpoint `[HttpPut]` trong `RegionsController` để xử lý các yêu cầu cập nhật, thực hiện ánh xạ từ DTO sang Domain Model, gọi Repository và trả về `200 OK` (với tài nguyên đã cập nhật) hoặc `404 Not Found`, đồng thời tích hợp kiểm tra `ModelState.IsValid` để xử lý validation.
*   Chúng ta đã sử dụng Swagger UI để kiểm thử thành công cả hai chức năng xóa và cập nhật, xác nhận rằng các HTTP Verbs và phản hồi trạng thái hoạt động đúng như mong đợi.
*   Cuối cùng, chúng ta đã lưu trữ các thay đổi vào Git, đảm bảo tiến độ công việc được ghi lại và bảo toàn.

Việc hoàn thành các thao tác CRUD (Create, Read, Update, Delete) là một cột mốc quan trọng, tạo nền tảng vững chắc cho việc phát triển các tính năng phức tạp hơn trong API của chúng ta.

<!-- REVIEWED_BY_AGENT -->
