# Phần 26: Hoàn Thiện API Hành Trình (Walks): Cập Nhật, Xóa & Quản Lý Mã Nguồn Với Git

Trong chương này, chúng ta sẽ tập trung vào việc hoàn thiện các thao tác cơ bản của RESTful API cho tài nguyên "Hành trình" (Walks) mà chúng ta đang xây dựng bằng ASP.NET Core, C# và Entity Framework Core. Cụ thể, chúng ta sẽ triển khai chức năng cập nhật và xóa một hành trình hiện có, sau đó kiểm thử kỹ lưỡng bằng Swagger UI. Đồng thời, chúng ta sẽ tích hợp quy trình quản lý phiên bản mã nguồn bằng Git, đảm bảo dự án được duy trì một cách chuyên nghiệp và có khả năng phục hồi.

Chúng ta sẽ tiếp tục củng cố các nguyên tắc thiết kế tốt như Dependency Injection và Repository Pattern, hiểu sâu hơn về cách Entity Framework Core tương tác với cơ sở dữ liệu, và áp dụng đúng các HTTP Verbs để xây dựng một API thực sự tuân thủ REST. Đặc biệt, chúng ta sẽ khám phá cách tư duy Vibe Coding có thể được áp dụng với một hệ thống AI mạnh mẽ như Antigravity IDE để tối ưu hóa quy trình phát triển, từ lập kế hoạch đến triển khai và kiểm thử.

## 1. Nền Tảng RESTful và Mô Hình Thiết Kế cho Thao Tác Dữ Liệu

Trước khi đi sâu vào mã nguồn, hãy cùng củng cố lại các khái niệm quan trọng định hình cách chúng ta xây dựng API:

*   **RESTful Principles**:
    *   **Resource Identification**: Mỗi "Hành trình" được xác định duy nhất bằng một URI (ví dụ: `/api/Walks/{id}`).
    *   **Statelessness**: Mỗi yêu cầu từ client đến server phải chứa tất cả thông tin cần thiết để server xử lý yêu cầu đó. Server không lưu trữ trạng thái của client giữa các yêu cầu.
    *   **Standardized Interface (HTTP Verbs)**: Sử dụng các phương thức HTTP tiêu chuẩn (`GET`, `POST`, `PUT`, `DELETE`) để chỉ định hành động muốn thực hiện trên tài nguyên.
        *   `PUT`: Được sử dụng để cập nhật toàn bộ tài nguyên, hoặc tạo mới nếu tài nguyên không tồn tại (Idempotent).
        *   `DELETE`: Được sử dụng để xóa một tài nguyên cụ thể.
*   **Dependency Injection (DI)**: Là một kỹ thuật thiết kế phần mềm giúp giảm sự phụ thuộc cứng giữa các thành phần. Trong ASP.NET Core, DI được sử dụng rộng rãi để cung cấp các dịch vụ (như Repository, Mapper) cho các Controller hoặc các lớp khác mà không yêu cầu chúng tự khởi tạo các dịch vụ đó. Điều này giúp mã nguồn dễ kiểm thử hơn, linh hoạt hơn và dễ bảo trì.
*   **Repository Pattern**: Cung cấp một lớp trừu tượng giữa logic nghiệp vụ và lớp truy cập dữ liệu. Thay vì Controller gọi trực tiếp Entity Framework Core `DbContext`, nó sẽ gọi một phương thức trên interface `IWalkRepository`. Điều này giúp:
    *   **Tách biệt mối quan tâm (Separation of Concerns)**: Controller không cần biết chi tiết về cách dữ liệu được lưu trữ hay truy vấn.
    *   **Khả năng kiểm thử (Testability)**: Dễ dàng mock (giả lập) `IWalkRepository` trong các bài kiểm thử đơn vị cho Controller.
    *   **Linh hoạt**: Có thể thay đổi công nghệ cơ sở dữ liệu mà không ảnh hưởng đến Controller.
*   **Data Transfer Objects (DTOs)**: Là các đối tượng đơn giản được sử dụng để truyền dữ liệu giữa các lớp hoặc giữa các hệ thống. Trong API, DTOs được sử dụng để:
    *   **Định nghĩa hợp đồng API**: Kiểm soát chính xác dữ liệu nào được gửi đến và nhận từ client.
    *   **Bảo mật**: Ngăn chặn việc phơi bày các thuộc tính nhạy cảm của đối tượng miền ra bên ngoài.
    *   **Tối ưu hóa**: Chỉ gửi những dữ liệu cần thiết, giảm tải mạng.
    *   **Xác thực**: Dễ dàng áp dụng các quy tắc xác thực riêng biệt cho dữ liệu đầu vào.

## 2. Triển Khai Chức Năng Cập Nhật Hành Trình (Update Walk)

Chức năng cập nhật (Update) cho phép người dùng thay đổi thông tin của một hành trình đã tồn tại. Chúng ta sẽ thực hiện điều này một cách có cấu trúc qua ba lớp chính: Giao diện Repository, Lớp Repository và Controller.

### 2.1. Định Nghĩa Phương Thức Cập Nhật trong `IWalkRepository`

Bước đầu tiên là khai báo hợp đồng cho chức năng cập nhật trong giao diện `IWalkRepository`. Điều này đảm bảo rằng bất kỳ lớp nào triển khai giao diện này đều phải cung cấp logic cho việc cập nhật hành trình.

```csharp
// Interfaces/IWalkRepository.cs

namespace NZWalks.API.Repositories
{
    public interface IWalkRepository
    {
        // ... các phương thức khác (Get All, Get By Id, Add) ...

        /// <summary>
        /// Cập nhật thông tin của một hành trình (Walk) dựa trên ID.
        /// </summary>
        /// <param name="id">ID của hành trình cần cập nhật.</param>
        /// <param name="walkDomainModel">Đối tượng miền Walk chứa thông tin mới.</param>
        /// <returns>Đối tượng Walk đã được cập nhật, hoặc null nếu không tìm thấy.</returns>
        Task<Walk?> UpdateWalkAsync(Guid id, Walk walkDomainModel);
    }
}
```

> **Giải thích**:
> *   Phương thức `UpdateWalkAsync` nhận vào hai tham số: `id` (kiểu `Guid`) để xác định hành trình cần cập nhật và `walkDomainModel` (kiểu `Walk`) chứa dữ liệu mới sẽ được áp dụng.
> *   Kiểu trả về là `Task<Walk?>` cho thấy đây là một thao tác bất đồng bộ và có thể trả về `null` nếu không tìm thấy hành trình với `id` đã cho. Việc sử dụng `Task` là tiêu chuẩn trong ASP.NET Core để đảm bảo hiệu suất, đặc biệt là với các thao tác I/O như truy vấn cơ sở dữ liệu.

### 2.2. Triển Khai Logic Cập Nhật trong `WalkRepository`

Bây giờ, chúng ta sẽ cung cấp logic cụ thể cho phương thức `UpdateWalkAsync` trong lớp `WalkRepository`. Đây là nơi Entity Framework Core sẽ thực hiện tương tác với cơ sở dữ liệu.

```csharp
// Repositories/WalkRepository.cs

using Microsoft.EntityFrameworkCore; // Quan trọng: Đảm bảo có using này
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

        // ... các triển khai phương thức khác ...

        public async Task<Walk?> UpdateWalkAsync(Guid id, Walk walkDomainModel)
        {
            // 1. Tìm kiếm hành trình hiện có trong cơ sở dữ liệu
            // FindAsync là phương thức hiệu quả để tìm kiếm theo khóa chính.
            // Nó cũng kiểm tra xem thực thể đã có trong DbContext chưa trước khi truy vấn DB.
            var existingWalk = await _dbContext.Walks.FindAsync(id);

            // 2. Xử lý trường hợp không tìm thấy hành trình
            if (existingWalk == null)
            {
                return null; // Trả về null để báo hiệu không tìm thấy
            }

            // 3. Cập nhật các thuộc tính của hành trình hiện có
            // Quan trọng: Chúng ta cập nhật các thuộc tính của 'existingWalk'
            // chứ không phải gán 'walkDomainModel' trực tiếp.
            // Entity Framework Core sẽ theo dõi các thay đổi trên 'existingWalk'.
            existingWalk.Name = walkDomainModel.Name;
            existingWalk.Length = walkDomainModel.Length;
            existingWalk.RegionId = walkDomainModel.RegionId;
            existingWalk.DifficultyId = walkDomainModel.DifficultyId;
            // Không cần cập nhật ID vì đó là khóa chính và không thay đổi.

            // 4. Lưu các thay đổi vào cơ sở dữ liệu
            // SaveChangesAsync() sẽ tạo câu lệnh SQL UPDATE dựa trên các thuộc tính
            // đã thay đổi của 'existingWalk' và thực thi nó trên DB.
            await _dbContext.SaveChangesAsync();

            // 5. Trả về đối tượng hành trình đã được cập nhật
            return existingWalk;
        }
    }
}
```

> **Cơ chế ngầm (Under the Hood) của Entity Framework Core khi Cập nhật**:
> Khi bạn lấy một thực thể (ví dụ: `existingWalk`) từ `DbContext` bằng các phương thức như `FindAsync`, `FirstOrDefaultAsync`, hoặc `Where().ToListAsync()`, Entity Framework Core bắt đầu "theo dõi" thực thể đó. Mọi thay đổi bạn thực hiện trên các thuộc tính của `existingWalk` sẽ được `DbContext` ghi nhận.
> Khi bạn gọi `_dbContext.SaveChangesAsync()`, EF Core sẽ quét tất cả các thực thể đang được theo dõi để tìm ra những thay đổi. Nếu nó phát hiện một thực thể có trạng thái `Modified` (đã thay đổi), nó sẽ tạo ra một câu lệnh `UPDATE` SQL tương ứng và thực thi nó trên cơ sở dữ liệu.
> `FindAsync(id)` là lựa chọn tốt nhất để tìm kiếm theo khóa chính vì nó tối ưu: nó sẽ kiểm tra bộ nhớ đệm của `DbContext` trước, nếu tìm thấy thực thể đã được theo dõi, nó sẽ trả về ngay mà không cần truy vấn cơ sở dữ liệu.

> **Vibe Coding với Antigravity IDE - Triển khai Repository**:
> Với Antigravity IDE, quy trình này có thể được tối ưu hóa đáng kể.
> 1.  **Ý định (Intent)**: Bạn có thể bắt đầu bằng một câu lệnh tự nhiên trong Antigravity: "Implement `UpdateWalkAsync` in `WalkRepository` to find a walk by ID, update its properties from the provided domain model, and save changes. Return null if not found."
> 2.  **Lập kế hoạch của Agent**: Antigravity sẽ tự động phân tích:
>     *   Cần truy cập `_dbContext.Walks`.
>     *   Phương thức tìm kiếm theo ID: `FindAsync` là phù hợp.
>     *   Cần kiểm tra `null` sau khi tìm kiếm.
>     *   Cần gán các thuộc tính từ `walkDomainModel` vào `existingWalk`.
>     *   Cần gọi `_dbContext.SaveChangesAsync()`.
> 3.  **Tự động tạo mã**: Antigravity sẽ tự động tạo ra phần lớn mã trên, sử dụng các kiến thức về EF Core và cấu trúc dự án hiện có (nó có thể đọc `NZWalksDbContext` và `Walk` domain model).
> 4.  **Tối ưu và gợi ý**: Nó có thể gợi ý `using Microsoft.EntityFrameworkCore;` nếu thiếu, hoặc đề xuất cách xử lý các trường hợp cạnh tranh (concurrency) nếu cần (mặc dù không nằm trong yêu cầu này).
> Vibe Coding ở đây là bạn cung cấp "vibe" (ý định cấp cao), và Antigravity thực hiện các bước chi tiết, giúp bạn tập trung vào kiến trúc và luồng nghiệp vụ thay vì cú pháp.

### 2.3. Xây Dựng DTO Yêu Cầu Cập Nhật (`UpdateWalkRequestDto`)

Để duy trì sự tách biệt giữa mô hình miền và hợp đồng API, chúng ta tạo một DTO riêng cho yêu cầu cập nhật. Điều này cho phép chúng ta kiểm soát chính xác những dữ liệu nào client có thể gửi lên.

```csharp
// Models/DTO/UpdateWalkRequestDto.cs

using System.ComponentModel.DataAnnotations; // Để thêm xác thực

namespace NZWalks.API.Models.DTO
{
    public class UpdateWalkRequestDto
    {
        [Required(ErrorMessage = "Tên hành trình là bắt buộc.")]
        [MaxLength(100, ErrorMessage = "Tên hành trình không được vượt quá 100 ký tự.")]
        public string Name { get; set; }

        [Required(ErrorMessage = "Độ dài hành trình là bắt buộc.")]
        [Range(0, double.MaxValue, ErrorMessage = "Độ dài hành trình phải là một số dương.")]
        public double Length { get; set; }

        [Required(ErrorMessage = "ID vùng là bắt buộc.")]
        public Guid RegionId { get; set; }

        [Required(ErrorMessage = "ID độ khó là bắt buộc.")]
        public Guid DifficultyId { get; set; }
    }
}
```

> **Lưu ý về DTO và Xác thực**:
> *   `UpdateWalkRequestDto` thường có các thuộc tính tương tự như `AddWalkRequestDto` vì cả hai đều đại diện cho dữ liệu đầu vào từ client.
> *   Việc thêm các thuộc tính xác thực (Data Annotations) như `[Required]` và `[MaxLength]` trực tiếp vào DTO là một thực hành tốt. ASP.NET Core sẽ tự động kiểm tra các quy tắc này khi dữ liệu được ràng buộc từ yêu cầu HTTP, và trả về phản hồi `HTTP 400 Bad Request` nếu xác thực thất bại. Điều này giúp bảo vệ logic nghiệp vụ khỏi dữ liệu đầu vào không hợp lệ.

### 2.4. Triển Khai Phương Thức Cập Nhật trong `WalksController`

Cuối cùng, chúng ta sẽ thêm một phương thức hành động (Action Method) vào `WalksController` để xử lý các yêu cầu HTTP PUT từ client.

```csharp
// Controllers/WalksController.cs

using AutoMapper; // Quan trọng: Đảm bảo có using này
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
        private readonly IWalkRepository _walkRepository;
        private readonly IMapper _mapper; // Khai báo AutoMapper

        // Dependency Injection cho IWalkRepository và IMapper
        public WalksController(IWalkRepository walkRepository, IMapper mapper)
        {
            _walkRepository = walkRepository;
            _mapper = mapper;
        }

        // ... các phương thức hành động khác ...

        /// <summary>
        /// Cập nhật thông tin của một hành trình dựa trên ID.
        /// </summary>
        /// <param name="id">ID của hành trình cần cập nhật (từ URL).</param>
        /// <param name="updateWalkRequestDto">Dữ liệu cập nhật từ client (từ Body).</param>
        /// <returns>Thông tin hành trình đã được cập nhật hoặc NotFound nếu không tìm thấy.</returns>
        [HttpPut("{id}")] // HTTP PUT verb với tham số ID trong URL
        public async Task<IActionResult> UpdateWalkAsync([FromRoute] Guid id, [FromBody] UpdateWalkRequestDto updateWalkRequestDto)
        {
            // 0. Kiểm tra trạng thái mô hình (Model State) cho xác thực DTO
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState); // Trả về HTTP 400 Bad Request với lỗi xác thực
            }

            // 1. Chuyển đổi DTO yêu cầu thành đối tượng miền (Domain Model)
            // AutoMapper giúp tự động ánh xạ các thuộc tính từ DTO sang Domain Model.
            var walkDomainModel = _mapper.Map<Walk>(updateWalkRequestDto);

            // 2. Gọi phương thức cập nhật từ Repository
            var updatedWalkDomainModel = await _walkRepository.UpdateWalkAsync(id, walkDomainModel);

            // 3. Xử lý trường hợp không tìm thấy hành trình
            if (updatedWalkDomainModel == null)
            {
                return NotFound(); // Trả về HTTP 404 Not Found
            }

            // 4. Chuyển đổi đối tượng miền đã cập nhật thành DTO phản hồi
            // Sử dụng AutoMapper để ánh xạ ngược lại từ Domain Model sang WalkDto
            // để trả về cho client. Đảm bảo các thuộc tính liên quan (Region, Difficulty)
            // cũng được ánh xạ nếu cần (nếu chúng đã được eager loaded trong Repository).
            var walkDto = _mapper.Map<WalkDto>(updatedWalkDomainModel);

            // 5. Trả về phản hồi HTTP 200 OK với dữ liệu hành trình đã cập nhật
            return Ok(walkDto);
        }
    }
}
```

> **Giải thích chi tiết Controller Action**:
> *   **`[HttpPut("{id}")]`**: Đây là một thuộc tính định tuyến (routing attribute) chỉ định rằng phương thức này sẽ xử lý các yêu cầu HTTP `PUT` đến URL `/api/Walks/{id}`, nơi `{id}` là một tham số động (path parameter).
> *   **`[FromRoute] Guid id`**: Thuộc tính ràng buộc mô hình (model binding attribute) này hướng dẫn ASP.NET Core lấy giá trị cho tham số `id` từ phân đoạn URL.
> *   **`[FromBody] UpdateWalkRequestDto updateWalkRequestDto`**: Thuộc tính này chỉ định rằng giá trị cho tham số `updateWalkRequestDto` sẽ được đọc từ phần thân (body) của yêu cầu HTTP. ASP.NET Core sẽ tự động deserialize JSON trong body thành đối tượng `UpdateWalkRequestDto`.
> *   **`if (!ModelState.IsValid)`**: Đây là kiểm tra bắt buộc sau khi model binding. Nếu bất kỳ quy tắc xác thực nào trên `UpdateWalkRequestDto` bị vi phạm, `ModelState.IsValid` sẽ là `false`, và chúng ta trả về `HTTP 400 Bad Request` cùng với thông tin lỗi chi tiết.
> *   **`_mapper.Map<Walk>(updateWalkRequestDto)`**: AutoMapper (được inject qua `IMapper`) tự động chuyển đổi `UpdateWalkRequestDto` thành đối tượng `Walk` (domain model). Điều này giúp tránh việc gán thủ công từng thuộc tính, làm cho mã gọn gàng và dễ bảo trì hơn.
> *   **`return NotFound()`**: Nếu `_walkRepository.UpdateWalkAsync` trả về `null` (nghĩa là không tìm thấy hành trình), chúng ta trả về phản hồi HTTP 404 Not Found, đây là mã trạng thái tiêu chuẩn cho tài nguyên không tồn tại.
> *   **`return Ok(walkDto)`**: Nếu cập nhật thành công, chúng ta trả về `HTTP 200 OK` cùng với `WalkDto` của hành trình đã cập nhật. Việc trả về DTO đảm bảo rằng client nhận được một phản hồi nhất quán và chỉ chứa các thông tin công khai.

> **Vibe Coding với Antigravity IDE - Triển khai Controller**:
> 1.  **Ý định**: "Create an `HttpPut` endpoint in `WalksController` to update a walk by its ID. It should take `UpdateWalkRequestDto` from the body, map it to a domain model, call the repository, handle `NotFound`, map the result back to `WalkDto`, and return `Ok`. Add `ModelState` validation."
> 2.  **Lập kế hoạch của Agent**:
>     *   Xác định HTTP verb và route: `[HttpPut("{id}")]`.
>     *   Xác định tham số đầu vào: `[FromRoute] Guid id`, `[FromBody] UpdateWalkRequestDto`.
>     *   Kiểm tra `ModelState.IsValid`.
>     *   Sử dụng `_mapper` để chuyển đổi DTO sang domain model.
>     *   Gọi `_walkRepository.UpdateWalkAsync`.
>     *   Kiểm tra kết quả `null` và trả về `NotFound`.
>     *   Sử dụng `_mapper` để chuyển đổi domain model sang DTO phản hồi.
>     *   Trả về `Ok` với DTO.
> 3.  **Tự động tạo mã**: Antigravity sẽ tạo ra cấu trúc cơ bản của phương thức, bao gồm các thuộc tính định tuyến, tham số, và các bước gọi `_mapper` và `_walkRepository`. Nó có thể tự động thêm kiểm tra `ModelState.IsValid` như một thực hành tốt.
> 4.  **Học viên tinh chỉnh**: Bạn có thể xem lại mã, đảm bảo các tên biến và logic phản ánh chính xác ý định của bạn, hoặc thêm các tùy chỉnh nhỏ nếu Antigravity chưa nắm bắt được hết ngữ cảnh đặc biệt.

### 2.5. Kiểm Thử Chức Năng Cập Nhật với Swagger UI

Sau khi triển khai, kiểm thử là bước không thể thiếu để đảm bảo chức năng hoạt động đúng như mong đợi. Swagger UI cung cấp một giao diện thuận tiện để thực hiện điều này.

1.  **Chạy ứng dụng**: Khởi động ứng dụng ASP.NET Core của bạn (thường bằng `dotnet run` hoặc F5 trong Visual Studio). Swagger UI sẽ tự động mở trong trình duyệt.
2.  **Lấy ID của một hành trình hiện có**:
    *   Mở rộng endpoint `GET /api/Walks`.
    *   Nhấp vào "Try it out" và sau đó "Execute".
    *   Trong phần "Response body", sao chép `Id` của một hành trình bất kỳ mà bạn muốn cập nhật (ví dụ: hành trình "Pinnacles Walk"). Lưu ý các giá trị `RegionId` và `DifficultyId` hiện có.
3.  **Sử dụng endpoint Cập nhật**:
    *   Cuộn xuống endpoint `PUT /api/Walks/{id}`.
    *   Nhấp vào "Try it out".
    *   **Điền ID**: Dán `Id` bạn đã sao chép vào trường `id` trong phần "Parameters".
    *   **Điền Request Body**: Trong phần "Request body", thay đổi các giá trị cho `name`, `length`, `regionId`, `difficultyId` theo ý muốn.
        *   Ví dụ: Thay đổi `Name` thành "Pinnacles Walk - Updated", `Length` từ 6 thành 5.5.
        *   Đảm bảo `RegionId` và `DifficultyId` là các GUID hợp lệ và tồn tại trong cơ sở dữ liệu của bạn để tránh lỗi khóa ngoại.
    *   Nhấp vào "Execute".
4.  **Kiểm tra kết quả**:
    *   Bạn sẽ nhận được phản hồi **HTTP 200 OK** nếu cập nhật thành công.
    *   Phần "Response body" sẽ hiển thị thông tin chi tiết của hành trình sau khi cập nhật, phản ánh những thay đổi bạn vừa gửi.
    *   Để xác nhận lại, bạn có thể gọi lại `GET /api/Walks` hoặc `GET /api/Walks/{id}` (với ID của hành trình đã cập nhật) để xem hành trình đó đã được cập nhật vĩnh viễn trong cơ sở dữ liệu hay chưa.

## 3. Triển Khai Chức Năng Xóa Hành Trình (Delete Walk)

Chức năng xóa (Delete) cho phép người dùng loại bỏ một hành trình khỏi hệ thống. Tương tự như cập nhật, chúng ta sẽ triển khai nó qua các lớp Repository và Controller.

### 3.1. Định Nghĩa Phương Thức Xóa trong `IWalkRepository`

Thêm định nghĩa phương thức xóa vào giao diện `IWalkRepository`. Phương thức này chỉ cần ID của hành trình cần xóa.

```csharp
// Interfaces/IWalkRepository.cs

namespace NZWalks.API.Repositories
{
    public interface IWalkRepository
    {
        // ... các phương thức khác ...

        /// <summary>
        /// Xóa một hành trình (Walk) dựa trên ID.
        /// </summary>
        /// <param name="id">ID của hành trình cần xóa.</param>
        /// <returns>Đối tượng Walk đã bị xóa, hoặc null nếu không tìm thấy.</returns>
        Task<Walk?> DeleteWalkAsync(Guid id);
    }
}
```

> **Giải thích**:
> *   Phương thức `DeleteWalkAsync` chỉ cần `id` để xác định tài nguyên cần xóa.
> *   Kiểu trả về `Task<Walk?>` cho phép chúng ta trả về đối tượng `Walk` đã bị xóa (có thể hữu ích cho client để xác nhận hoặc hoàn tác) hoặc `null` nếu không tìm thấy.

### 3.2. Triển Khai Logic Xóa trong `WalkRepository`

Trong lớp `WalkRepository`, chúng ta sẽ triển khai logic để tìm và xóa hành trình khỏi cơ sở dữ liệu.

```csharp
// Repositories/WalkRepository.cs

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

        // ... các triển khai phương thức khác ...

        public async Task<Walk?> DeleteWalkAsync(Guid id)
        {
            // 1. Tìm kiếm hành trình hiện có trong cơ sở dữ liệu
            var existingWalk = await _dbContext.Walks.FindAsync(id);

            // 2. Xử lý trường hợp không tìm thấy hành trình
            if (existingWalk == null)
            {
                return null;
            }

            // 3. Xóa hành trình khỏi DbContext
            // Phương thức Remove() đánh dấu thực thể là 'Deleted' trong DbContext.
            _dbContext.Walks.Remove(existingWalk);

            // 4. Lưu các thay đổi vào cơ sở dữ liệu
            // SaveChangesAsync() sẽ tạo câu lệnh SQL DELETE và thực thi nó trên DB.
            await _dbContext.SaveChangesAsync();

            // 5. Trả về đối tượng hành trình đã bị xóa
            // Việc trả về đối tượng đã xóa có thể hữu ích cho client để xác nhận.
            return existingWalk;
        }
    }
}
```

> **Cơ chế ngầm (Under the Hood) của Entity Framework Core khi Xóa**:
> Tương tự như cập nhật, khi bạn gọi `_dbContext.Walks.Remove(existingWalk)`, Entity Framework Core sẽ đánh dấu thực thể `existingWalk` là `Deleted`. Khi `_dbContext.SaveChangesAsync()` được gọi, EF Core sẽ tạo ra một câu lệnh `DELETE` SQL tương ứng và thực thi nó trên cơ sở dữ liệu, loại bỏ bản ghi đó.

> **Vibe Coding với Antigravity IDE - Triển khai Xóa Repository**:
> 1.  **Ý định**: "Implement `DeleteWalkAsync` in `WalkRepository` to find and remove a walk by ID, then save changes. Return the deleted walk or null if not found."
> 2.  **Lập kế hoạch của Agent**:
>     *   Tìm kiếm bằng `FindAsync`.
>     *   Kiểm tra `null`.
>     *   Sử dụng `_dbContext.Walks.Remove()`.
>     *   Gọi `_dbContext.SaveChangesAsync()`.
> 3.  **Tự động tạo mã**: Antigravity sẽ tạo mã tương tự như trên.
> 4.  **Lợi ích**: Giúp developer tập trung vào việc đảm bảo các ràng buộc khóa ngoại (ví dụ: một Walk không thể bị xóa nếu có các thực thể khác đang tham chiếu đến nó, trừ khi cấu hình CASCADE DELETE) thay vì viết boilerplate code.

### 3.3. Triển Khai Phương Thức Xóa trong `WalksController`

Phương thức hành động trong `WalksController` sẽ xử lý các yêu cầu HTTP DELETE. Nó chỉ cần ID của hành trình từ URL.

```csharp
// Controllers/WalksController.cs

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
        private readonly IWalkRepository _walkRepository;
        private readonly IMapper _mapper;

        public WalksController(IWalkRepository walkRepository, IMapper mapper)
        {
            _walkRepository = walkRepository;
            _mapper = mapper;
        }

        // ... các phương thức hành động khác ...

        /// <summary>
        /// Xóa một hành trình dựa trên ID.
        /// </summary>
        /// <param name="id">ID của hành trình cần xóa.</param>
        /// <returns>Thông tin hành trình đã bị xóa hoặc NotFound nếu không tìm thấy.</returns>
        [HttpDelete("{id}")] // HTTP DELETE verb với tham số ID trong URL
        public async Task<IActionResult> DeleteWalkAsync([FromRoute] Guid id)
        {
            // 1. Gọi phương thức xóa từ Repository
            var deletedWalkDomainModel = await _walkRepository.DeleteWalkAsync(id);

            // 2. Xử lý trường hợp không tìm thấy hành trình
            if (deletedWalkDomainModel == null)
            {
                return NotFound(); // Trả về HTTP 404 Not Found
            }

            // 3. Chuyển đổi đối tượng miền đã xóa thành DTO phản hồi
            // Việc trả về DTO của đối tượng đã xóa là một thực hành tốt,
            // giúp client xác nhận tài nguyên nào đã bị loại bỏ.
            var walkDto = _mapper.Map<WalkDto>(deletedWalkDomainModel);

            // 4. Trả về phản hồi HTTP 200 OK với dữ liệu hành trình đã bị xóa
            return Ok(walkDto);
        }
    }
}
```

> **Giải thích Controller Action**:
> *   **`[HttpDelete("{id}")]`**: Đây là HTTP Verb tiêu chuẩn cho thao tác xóa tài nguyên. Nó chỉ định rằng phương thức này sẽ xử lý các yêu cầu `DELETE` đến `/api/Walks/{id}`.
> *   **`[FromRoute] Guid id`**: Lấy ID từ URL.
> *   **`return NotFound()`**: Nếu Repository trả về `null`, báo hiệu không tìm thấy hành trình để xóa, chúng ta trả về `HTTP 404 Not Found`.
> *   **`return Ok(walkDto)`**: Nếu xóa thành công, trả về `HTTP 200 OK` cùng với `WalkDto` của hành trình vừa bị xóa.

> **Vibe Coding với Antigravity IDE - Triển khai Xóa Controller**:
> 1.  **Ý định**: "Create an `HttpDelete` endpoint in `WalksController` to delete a walk by its ID. Call the repository, handle `NotFound`, map the result to `WalkDto`, and return `Ok`."
> 2.  **Lập kế hoạch của Agent**: Tương tự như `HttpPut`, Antigravity sẽ xác định verb, route, tham số, và các bước gọi repository, xử lý lỗi và ánh xạ DTO.
> 3.  **Tự động tạo mã**: Antigravity sẽ cung cấp skeleton code.
> 4.  **Tư duy Vibe Coding**: Bạn có thể chỉ định "trả về đối tượng đã xóa" hoặc "chỉ trả về 204 No Content nếu xóa thành công" tùy theo yêu cầu cụ thể của API. Antigravity sẽ điều chỉnh mã cho phù hợp, cho phép bạn tập trung vào quyết định thiết kế API thay vì cú pháp.

### 3.4. Kiểm Thử Chức Năng Xóa với Swagger UI

Kiểm thử chức năng xóa là rất quan trọng để đảm bảo rằng dữ liệu được loại bỏ đúng cách và API xử lý các trường hợp không tìm thấy tài nguyên một cách chính xác.

1.  **Chạy ứng dụng**: Khởi động ứng dụng.
2.  **Lấy ID của một hành trình để xóa**:
    *   Mở endpoint `GET /api/Walks`.
    *   Lấy `Id` của một hành trình mà bạn *chắc chắn muốn xóa*. (Lưu ý: Thao tác này là vĩnh viễn trong cơ sở dữ liệu).
3.  **Sử dụng endpoint Xóa**:
    *   Cuộn xuống endpoint `DELETE /api/Walks/{id}`.
    *   Nhấp vào "Try it out".
    *   **Điền ID**: Dán `Id` bạn đã sao chép vào trường `id`.
    *   Nhấp vào "Execute".
4.  **Kiểm tra kết quả xóa thành công**:
    *   Bạn sẽ nhận được phản hồi **HTTP 200 OK** nếu xóa thành công.
    *   Phần "Response body" sẽ hiển thị thông tin chi tiết của hành trình đã bị xóa.
5.  **Kiểm tra trường hợp tài nguyên đã bị xóa (Not Found)**:
    *   Thử thực hiện lại yêu cầu `DELETE /api/Walks/{id}` với cùng một `Id` mà bạn vừa xóa.
    *   Lần này, bạn sẽ nhận được phản hồi **HTTP 404 Not Found**, xác nhận rằng hành trình đó không còn tồn tại trong cơ sở dữ liệu.
    *   Bạn cũng có thể kiểm tra bằng cách gọi lại `GET /api/Walks` để xác nhận rằng hành trình đó không còn xuất hiện trong danh sách.

## 4. Quản Lý Phiên Bản Mã Nguồn với Git và Quy Trình Phát Triển Hiệu Quả

Sau khi hoàn tất việc triển khai và kiểm thử các tính năng quan trọng, việc lưu trữ các thay đổi vào hệ thống kiểm soát phiên bản Git là bước cuối cùng và cực kỳ quan trọng trong bất kỳ quy trình phát triển phần mềm chuyên nghiệp nào.

### 4.1. Tầm Quan Trọng của Git trong Phát Triển Hiện Đại

**Git** là một hệ thống kiểm soát phiên bản phân tán (Distributed Version Control System - DVCS) được sử dụng rộng rãi nhất hiện nay. Nó là xương sống cho việc quản lý mã nguồn, đặc biệt trong môi trường làm việc nhóm.

**Lợi ích chính của Git**:
*   **Lịch sử thay đổi minh bạch**: Ghi lại mọi thay đổi, ai đã thay đổi, khi nào và tại sao. Bạn có thể xem lại toàn bộ lịch sử phát triển của dự án.
*   **Phục hồi dễ dàng**: Dễ dàng quay lại các phiên bản trước của mã nguồn, phục hồi các tệp bị xóa hoặc sửa đổi sai.
*   **Hợp tác hiệu quả**: Cho phép nhiều nhà phát triển làm việc trên cùng một dự án mà không xung đột, thông qua cơ chế phân nhánh (branching) và hợp nhất (merging).
*   **Phân nhánh (Branching)**: Hỗ trợ phát triển các tính năng mới, sửa lỗi, hoặc thử nghiệm các ý tưởng độc lập mà không ảnh hưởng đến mã nguồn chính (thường là nhánh `main` hoặc `master`).
*   **Sao lưu**: Kho lưu trữ từ xa (remote repository) như GitHub, GitLab, Azure DevOps đóng vai trò là bản sao lưu an toàn cho mã nguồn của bạn.

### 4.2. Lưu Các Thay Đổi Vào Git

Để lưu trữ các thay đổi của chúng ta vào Git, chúng ta sẽ thực hiện các bước cơ bản: thêm các tệp đã thay đổi vào staging area, tạo một commit và đẩy (push) các thay đổi lên kho lưu trữ từ xa.

1.  **Staging các thay đổi**:
    Trước khi commit, bạn cần cho Git biết những tệp nào bạn muốn đưa vào commit tiếp theo. Đây gọi là "staging" các thay đổi.
    ```bash
    git add .
    ```
    Lệnh này thêm tất cả các tệp đã thay đổi hoặc mới tạo vào staging area. Nếu bạn chỉ muốn thêm một số tệp cụ thể, bạn có thể chỉ định đường dẫn của chúng (ví dụ: `git add Repositories/WalkRepository.cs`).
2.  **Tạo Commit**:
    Thực hiện commit với một thông điệp rõ ràng, mô tả những gì đã được thay đổi trong phiên làm việc này. Thông điệp commit là một phần quan trọng của lịch sử dự án.
    ```bash
    git commit -m "feat: Implement Update and Delete operations for Walks API"
    ```
    > **Mẹo về Thông điệp Commit**:
    > *   **Ngắn gọn và súc tích**: Dòng đầu tiên (subject) không quá 50-72 ký tự.
    > *   **Mô tả rõ ràng**: Giải thích `what` và `why` của thay đổi.
    > *   **Sử dụng tiền tố**: Các tiền tố như `feat:` (feature), `fix:` (bug fix), `docs:` (documentation), `refactor:` (refactoring code) giúp phân loại loại thay đổi. Điều này đặc biệt hữu ích cho việc tạo nhật ký thay đổi tự động.
3.  **Đẩy lên Kho lưu trữ từ xa (Push)**:
    Sau khi commit, đẩy các thay đổi từ kho lưu trữ cục bộ của bạn lên kho lưu trữ từ xa (ví dụ: GitHub, GitLab) để chia sẻ với nhóm và sao lưu mã nguồn.
    ```bash
    git push origin main
    ```
    (Giả sử nhánh chính của bạn là `main`. Nếu bạn đang làm việc trên một nhánh tính năng, bạn sẽ đẩy lên nhánh đó, ví dụ: `git push origin feature/walks-crud`).

### 4.3. Tối ưu hóa Quy Trình với Antigravity IDE và Vibe Coding

Antigravity IDE, với khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file và lập kế hoạch tự động, có thể biến các bước Git này thành một phần liền mạch của quy trình Vibe Coding của bạn:

1.  **Tự động đề xuất Commit**: Sau khi bạn hoàn thành một tác vụ (ví dụ: "Implement Update/Delete for Walks"), Antigravity có thể tự động quét các tệp đã thay đổi, so sánh chúng với các tệp gốc và đề xuất một thông điệp commit dựa trên ngữ cảnh của tác vụ bạn đang làm việc.
    *   **Ví dụ**: "Antigravity, tôi đã hoàn thành chức năng cập nhật và xóa hành trình. Hãy tạo một commit phù hợp." Antigravity có thể phản hồi: "Tôi đã phát hiện các thay đổi trong `WalkRepository.cs`, `WalksController.cs`, `IWalkRepository.cs`, và `UpdateWalkRequestDto.cs`. Tôi đề xuất thông điệp commit: `feat: Implement Update and Delete for Walks API`. Bạn có muốn tôi thực hiện `git add .` và `git commit` không?"
2.  **Tự động hóa Staging và Commit**: Với sự xác nhận của bạn, Antigravity có thể tự động thực hiện `git add .` và `git commit -m "..."`, giúp bạn tiết kiệm thời gian gõ lệnh và đảm bảo tính nhất quán của thông điệp commit.
3.  **Quản lý Nhánh (Branching)**: Antigravity có thể giúp bạn tạo nhánh mới, chuyển đổi giữa các nhánh, và thậm chí đề xuất các lệnh `git merge` hoặc `git rebase` khi đến lúc hợp nhất các thay đổi, dựa trên lịch sử dự án và các quy tắc làm việc nhóm.
4.  **Tự động Push**: Sau một loạt các commit cục bộ, Antigravity có thể nhắc bạn đẩy các thay đổi lên kho lưu trữ từ xa, hoặc bạn có thể thiết lập để nó tự động push sau mỗi commit nếu bạn làm việc trên một nhánh cá nhân.

Vibe Coding với Antigravity không chỉ là việc tạo mã, mà còn là việc tối ưu hóa toàn bộ vòng đời phát triển, từ lập kế hoạch, viết code, kiểm thử, đến quản lý phiên bản. Nó cho phép bạn duy trì "dòng chảy" của công việc, giảm thiểu sự gián đoạn và tập trung vào các quyết định kiến trúc và nghiệp vụ cấp cao.

## Tóm Tắt Phần 26

Trong Phần này, chúng ta đã hoàn thiện các thao tác CRUD cốt lõi cho API Hành trình (Walks) với các điểm chính sau:

*   **Cập Nhật Hành Trình (UpdateWalk)**:
    *   Đã định nghĩa phương thức `UpdateWalkAsync` trong `IWalkRepository` để thiết lập hợp đồng.
    *   Đã triển khai logic cập nhật trong `WalkRepository`, sử dụng `_dbContext.Walks.FindAsync()` để tìm kiếm và `_dbContext.SaveChangesAsync()` để ghi nhận các thay đổi được theo dõi bởi Entity Framework Core.
    *   Đã tạo `UpdateWalkRequestDto` với các thuộc tính xác thực để định nghĩa hợp đồng API đầu vào.
    *   Đã triển khai phương thức `UpdateWalkAsync` trong `WalksController` sử dụng HTTP PUT, tích hợp AutoMapper để chuyển đổi DTO sang đối tượng miền và ngược lại, xử lý xác thực `ModelState` và các trường hợp không tìm thấy tài nguyên (HTTP 404 Not Found).
    *   Đã kiểm thử thành công chức năng cập nhật bằng Swagger UI, xác nhận phản hồi HTTP 200 OK và sự thay đổi dữ liệu trong cơ sở dữ liệu.
*   **Xóa Hành Trình (DeleteWalk)**:
    *   Đã định nghĩa phương thức `DeleteWalkAsync` trong `IWalkRepository`.
    *   Đã triển khai logic xóa trong `WalkRepository`, sử dụng `_dbContext.Walks.Remove()` để đánh dấu thực thể cần xóa và `_dbContext.SaveChangesAsync()` để thực thi lệnh xóa trên cơ sở dữ liệu.
    *   Đã triển khai phương thức `DeleteWalkAsync` trong `WalksController` sử dụng HTTP DELETE, xử lý các trường hợp không tìm thấy tài nguyên và trả về HTTP 200 OK cùng với thông tin của hành trình đã bị xóa.
    *   Đã kiểm thử thành công chức năng xóa bằng Swagger UI, xác nhận phản hồi HTTP 200 OK cho lần xóa đầu tiên và HTTP 404 Not Found cho các yêu cầu xóa lặp lại.
*   **Quản Lý Phiên Bản với Git và Antigravity IDE**:
    *   Đã hiểu tầm quan trọng của Git trong việc theo dõi lịch sử, phục hồi và hợp tác.
    *   Đã thực hiện các lệnh Git cơ bản (`git add`, `git commit`, `git push`) để lưu trữ an toàn các thay đổi mã nguồn.
    *   Đã liên hệ cách Antigravity IDE và tư duy Vibe Coding có thể tự động hóa và tối ưu hóa quy trình quản lý Git, giúp developer tập trung vào logic nghiệp vụ chính.

Với việc hoàn thành các thao tác CRUD cơ bản, API Hành trình của chúng ta đã trở nên đầy đủ chức năng hơn. Trong các Phần tiếp theo, chúng ta sẽ tiếp tục xây dựng trên nền tảng này để bổ sung các tính năng nâng cao khác như xác thực, phân quyền, phân trang, và lọc dữ liệu, đồng thời liên tục tìm cách tối ưu hóa quy trình phát triển với sự hỗ trợ của các công cụ AI tiên tiến.

<!-- REVIEWED_BY_AGENT -->
