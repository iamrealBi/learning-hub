# Phần 8: Xác thực dữ liệu đầu vào (Model Validations)

Trong thế giới phát triển RESTful Web API với ASP.NET Core và Entity Framework Core, việc đảm bảo rằng dữ liệu mà client gửi lên server là hợp lệ, nhất quán và an toàn là một yếu tố then chốt. Phần này sẽ đưa chúng ta đi sâu vào cơ chế xác thực dữ liệu đầu vào (Model Validations) của ASP.NET Core, từ những khái niệm cơ bản về Data Annotations cho đến các kỹ thuật tối ưu hóa mã nguồn bằng Custom Action Filters. Mục tiêu là trang bị cho bạn kiến thức để xây dựng các API không chỉ mạnh mẽ và đáng tin cậy mà còn có khả năng cung cấp phản hồi rõ ràng, giúp client dễ dàng điều chỉnh và sử dụng API một cách hiệu quả.

## 1. Tầm quan trọng cốt lõi của xác thực dữ liệu trong kiến trúc Web API

Khi thiết kế và triển khai một RESTful Web API, chúng ta đang tạo ra một "hợp đồng" giao tiếp giữa server và các ứng dụng client đa dạng (web, di động, desktop). Các client này gửi yêu cầu để thực hiện các thao tác CRUD (Create, Read, Update, Delete) trên tài nguyên của hệ thống. Dữ liệu mà client gửi lên, đặc biệt trong các yêu cầu `POST` (tạo mới) và `PUT` (cập nhật), cần phải được kiểm tra kỹ lưỡng trước khi được xử lý bởi logic nghiệp vụ hoặc lưu trữ vào cơ sở dữ liệu.

### 1.1. Tại sao xác thực dữ liệu là không thể thiếu?

1.  **Đảm bảo tính toàn vẹn và nhất quán của dữ liệu:** Đây là mục tiêu hàng đầu. Xác thực ngăn chặn việc lưu trữ dữ liệu không hợp lệ (ví dụ: một chuỗi rỗng cho trường bắt buộc, một số âm cho giá trị không thể âm) vào cơ sở dữ liệu. Dữ liệu sạch sẽ giúp hệ thống hoạt động chính xác và đáng tin cậy.
2.  **Ngăn ngừa lỗi ứng dụng và ngoại lệ:** Dữ liệu không hợp lệ có thể gây ra các lỗi không mong muốn trong logic nghiệp vụ (ví dụ: chia cho 0, truy cập chỉ mục ngoài mảng) hoặc khi tương tác với cơ sở dữ liệu (ví dụ: vi phạm ràng buộc khóa ngoại, kiểu dữ liệu không khớp). Xác thực sớm giúp bắt lỗi trước khi chúng gây ra sự cố nghiêm trọng.
3.  **Tăng cường bảo mật hệ thống:** Xác thực dữ liệu là lớp phòng thủ đầu tiên chống lại nhiều loại tấn công phổ biến. Mặc dù Entity Framework Core đã giúp giảm thiểu rủi ro SQL Injection, nhưng các lỗ hổng khác như Cross-Site Scripting (XSS) (nếu dữ liệu không được làm sạch trước khi hiển thị lại), hoặc các lỗi logic nghiệp vụ do đầu vào độc hại vẫn có thể xảy ra. Xác thực giúp đảm bảo chỉ dữ liệu an toàn mới được xử lý.
4.  **Cung cấp phản hồi rõ ràng và cải thiện trải nghiệm người dùng:** Khi một yêu cầu chứa dữ liệu không hợp lệ, API không nên chỉ đơn thuần trả về lỗi server (như `500 Internal Server Error`). Thay vào đó, nó cần phản hồi bằng một mã trạng thái HTTP thích hợp (`400 Bad Request`) kèm theo thông tin chi tiết về các lỗi đã xảy ra. Điều này giúp client hiểu rõ vấn đề, sửa chữa dữ liệu và gửi lại yêu cầu một cách hiệu quả.

### 1.2. Cơ chế xác thực trong ASP.NET Core và phản hồi HTTP 400

Trong ASP.NET Core, khi một yêu cầu HTTP POST hoặc PUT đến một action method nhận dữ liệu, dữ liệu từ request body (thường là JSON) sẽ được ánh xạ (Model Binding) vào một đối tượng C# (thường được gọi là Model hoặc Data Transfer Object - DTO). Sau quá trình Model Binding, ASP.NET Core sẽ tự động thực hiện quá trình xác thực trên đối tượng DTO này dựa trên các quy tắc đã định nghĩa.

Nếu dữ liệu không hợp lệ, chúng ta sẽ phản hồi lại client bằng HTTP Status Code `400 Bad Request`. Mã `400` là phản hồi tiêu chuẩn cho biết server không thể xử lý yêu cầu do "client error" (lỗi từ phía client), thường là do cú pháp yêu cầu không đúng hoặc dữ liệu không hợp lệ. Kèm theo mã này, server sẽ gửi một payload JSON chứa chi tiết các lỗi xác thực, giúp client dễ dàng gỡ lỗi.

> [!NOTE]
> **Tư duy Vibe Coding và Antigravity IDE trong thiết kế Validation:**
>
> Với một hệ thống Agentic AI như Antigravity IDE, bạn có thể áp dụng tư duy Vibe Coding ngay từ giai đoạn thiết kế validation. Thay vì chỉ liệt kê các trường và kiểu dữ liệu, bạn có thể mô tả "vibe" của dữ liệu mong muốn:
>
> *   "Mã khu vực phải là 3 ký tự viết hoa."
> *   "Tên chuyến đi không được quá 100 ký tự và không được để trống."
> *   "Độ dài chuyến đi phải là một số dương, không quá 50 km."
>
> Antigravity, với khả năng hiểu ngữ cảnh và lập kế hoạch, có thể tự động gợi ý các Data Annotations phù hợp hoặc thậm chí tạo ra các DTO với cấu trúc validation ban đầu dựa trên mô tả ngôn ngữ tự nhiên này. Nó có thể phân tích các ràng buộc từ lược đồ cơ sở dữ liệu (nếu có) hoặc các quy tắc nghiệp vụ đã được định nghĩa ở nơi khác để đảm bảo tính nhất quán. Điều này giúp tăng tốc quá trình thiết kế DTO và validation, đồng thời giảm thiểu lỗi do thiếu sót ban đầu.

## 2. Data Annotations: Nền tảng xác thực Model trong ASP.NET Core

ASP.NET Core cung cấp một cơ chế xác thực tích hợp sẵn, dựa trên các thuộc tính (attributes) được gọi là Data Annotations. Đây là cách phổ biến và đơn giản nhất để thêm các quy tắc xác thực vào các Data Transfer Objects (DTOs) của bạn.

### 2.1. Data Annotations là gì và cơ chế hoạt động

**Data Annotations** là các thuộc tính được định nghĩa trong namespace `System.ComponentModel.DataAnnotations`. Chúng cho phép bạn khai báo các quy tắc xác thực trực tiếp trên các thuộc tính của lớp Model hoặc DTO.

**Cơ chế hoạt động (Under the Hood):**

1.  **Model Binding:** Khi một yêu cầu HTTP `POST` hoặc `PUT` đến một action method (ví dụ: `Create(AddAreaRequestDto dto)`), ASP.NET Core sẽ cố gắng ánh xạ dữ liệu từ request body (thường là JSON) vào đối tượng `AddAreaRequestDto`. Quá trình này được gọi là Model Binding.
2.  **Discovery và Execution:** Sau khi đối tượng DTO được khởi tạo và các thuộc tính của nó được điền đầy đủ từ dữ liệu đầu vào, Model Binding pipeline sẽ quét (discover) tất cả các Data Annotations được áp dụng trên các thuộc tính của DTO.
3.  **Populating `ModelState`:** Đối với mỗi Data Annotation, một quy tắc xác thực sẽ được thực thi. Nếu một quy tắc bị vi phạm, một lỗi xác thực sẽ được thêm vào đối tượng `ModelState`. `ModelState` là một từ điển chứa trạng thái xác thực của toàn bộ model, bao gồm các lỗi cụ thể cho từng thuộc tính.
4.  **`ModelState.IsValid`:** Sau khi tất cả các Data Annotations đã được kiểm tra, thuộc tính `ModelState.IsValid` sẽ phản ánh tổng thể liệu model có hợp lệ hay không. Nếu có bất kỳ lỗi nào được thêm vào `ModelState`, `IsValid` sẽ là `false`.

> [!TIP]
> **Data Transfer Objects (DTOs) và Repository Pattern:**
>
> Trong kiến trúc Web API, DTOs là các đối tượng đơn giản được sử dụng để truyền dữ liệu giữa client và server. Việc áp dụng validation trên DTOs thay vì Domain Models mang lại nhiều lợi ích:
>
> *   **Tách biệt mối quan tâm (Separation of Concerns):** Domain Models nên tập trung vào logic nghiệp vụ cốt lõi và trạng thái của miền, trong khi DTOs tập trung vào hình dạng dữ liệu cần thiết cho giao tiếp API.
> *   **Ngăn chặn Over-posting/Under-posting:** DTOs cho phép bạn chỉ tiếp xúc các trường dữ liệu mà client được phép gửi hoặc nhận, tránh việc client vô tình hoặc cố ý thay đổi các trường không mong muốn trong Domain Model.
> *   **Quy tắc xác thực linh hoạt:** Quy tắc xác thực cho việc tạo mới (Create) có thể khác với cập nhật (Update), và DTOs cho phép bạn định nghĩa các tập hợp quy tắc riêng biệt cho từng ngữ cảnh.
>
> Với Repository Pattern, dữ liệu từ DTO được xác thực trước, sau đó được ánh xạ sang Domain Model (thường bằng AutoMapper) và cuối cùng được truyền đến Repository để tương tác với cơ sở dữ liệu. Điều này đảm bảo rằng Repository luôn nhận được Domain Model hợp lệ.

### 2.2. Các Data Annotations phổ biến và cách áp dụng trên DTOs

Dưới đây là một số Data Annotations thường dùng để xác thực dữ liệu:

*   `[Required]`: Đánh dấu thuộc tính là bắt buộc. Giá trị không được là `null` hoặc chuỗi rỗng.
*   `[MinLength(length)]`: Giá trị chuỗi phải có độ dài tối thiểu là `length`.
*   `[MaxLength(length)]`: Giá trị chuỗi phải có độ dài tối đa là `length`.
*   `[StringLength(maximumLength, MinimumLength = minimumLength)]`: Đặt cả độ dài tối thiểu và tối đa cho chuỗi.
*   `[Range(minimum, maximum)]`: Giá trị số phải nằm trong khoảng từ `minimum` đến `maximum`.
*   `[EmailAddress]`: Kiểm tra định dạng email hợp lệ.
*   `[Url]`: Kiểm tra định dạng URL hợp lệ.
*   `[Phone]`: Kiểm tra định dạng số điện thoại hợp lệ.
*   `[RegularExpression("pattern")]`: Cho phép sử dụng biểu thức chính quy (regex) để xác thực chuỗi.

Bạn có thể tùy chỉnh thông báo lỗi bằng cách thêm thuộc tính `ErrorMessage` vào Data Annotation. Ví dụ: `[Required(ErrorMessage = "Trường '{0}' là bắt buộc.")]`. `{0}` sẽ được thay thế bằng tên của thuộc tính.

> [!NOTE]
> **Client-side Validation vs. Server-side Validation:**
>
> Điều quan trọng cần nhớ là Data Annotations có thể được sử dụng để hỗ trợ cả client-side validation (ví dụ: trong ASP.NET Core MVC với Tag Helpers hoặc các thư viện JavaScript tương thích) và server-side validation. Tuy nhiên, **server-side validation là bắt buộc và không thể bỏ qua**. Client-side validation chỉ nhằm mục đích cải thiện trải nghiệm người dùng bằng cách cung cấp phản hồi tức thì và giảm tải cho server. Bất kỳ client-side validation nào cũng có thể dễ dàng bị bỏ qua hoặc vô hiệu hóa bởi người dùng có ý đồ xấu, do đó, bạn **LUÔN PHẢI** thực hiện xác thực ở phía server để đảm bảo tính toàn vẹn và bảo mật dữ liệu.

#### 2.2.1. Ví dụ Code: `Area` DTOs

Hãy áp dụng các Data Annotations cho `AddAreaRequestDto` và `UpdateAreaRequestDto`.

```csharp
using System.ComponentModel.DataAnnotations; // Quan trọng: Đừng quên namespace này

namespace NZWalks.API.Models.DTO
{
    public class AddAreaRequestDto
    {
        [Required(ErrorMessage = "Mã khu vực là bắt buộc.")]
        [MinLength(3, ErrorMessage = "Mã khu vực phải có ít nhất 3 ký tự.")]
        [MaxLength(3, ErrorMessage = "Mã khu vực phải có tối đa 3 ký tự.")]
        // Hoặc có thể dùng [StringLength(3, MinimumLength = 3, ErrorMessage = "Mã khu vực phải có đúng 3 ký tự.")]
        public string Code { get; set; }

        [Required(ErrorMessage = "Tên khu vực là bắt buộc.")]
        [MaxLength(100, ErrorMessage = "Tên khu vực không được vượt quá 100 ký tự.")]
        public string Name { get; set; }

        // Url hình ảnh là tùy chọn, không cần [Required]
        // Tuy nhiên, nếu có, bạn có thể thêm [Url(ErrorMessage = "Định dạng URL hình ảnh không hợp lệ.")]
        public string? AreaImageUrl { get; set; }
    }
}
```

Đối với `UpdateAreaRequestDto`, các quy tắc xác thực thường tương tự như `AddAreaRequestDto` vì chúng đại diện cho các trường dữ liệu cơ bản của một khu vực.

```csharp
using System.ComponentModel.DataAnnotations;

namespace NZWalks.API.Models.DTO
{
    public class UpdateAreaRequestDto
    {
        [Required(ErrorMessage = "Mã khu vực là bắt buộc.")]
        [MinLength(3, ErrorMessage = "Mã khu vực phải có ít nhất 3 ký tự.")]
        [MaxLength(3, ErrorMessage = "Mã khu vực phải có tối đa 3 ký tự.")]
        public string Code { get; set; }

        [Required(ErrorMessage = "Tên khu vực là bắt buộc.")]
        [MaxLength(100, ErrorMessage = "Tên khu vực không được vượt quá 100 ký tự.")]
        public string Name { get; set; }

        public string? AreaImageUrl { get; set; }
    }
}
```

#### 2.2.2. Ví dụ Code: `Walk` DTOs

Tương tự, chúng ta áp dụng Data Annotations cho `AddWalkRequestDto` và `UpdateWalkRequestDto`.

```csharp
using System.ComponentModel.DataAnnotations;

namespace NZWalks.API.Models.DTO
{
    public class AddWalkRequestDto
    {
        [Required(ErrorMessage = "Tên chuyến đi là bắt buộc.")]
        [MaxLength(100, ErrorMessage = "Tên chuyến đi không được vượt quá 100 ký tự.")]
        public string Name { get; set; }

        [Required(ErrorMessage = "Mô tả chuyến đi là bắt buộc.")]
        [MinLength(10, ErrorMessage = "Mô tả phải có ít nhất 10 ký tự.")]
        [MaxLength(1000, ErrorMessage = "Mô tả không được vượt quá 1000 ký tự.")]
        public string Description { get; set; }

        [Required(ErrorMessage = "Độ dài là bắt buộc.")]
        [Range(0.01, 50.0, ErrorMessage = "Độ dài phải nằm trong khoảng từ 0.01 đến 50 km.")] // Ví dụ: độ dài tính bằng km, không thể là 0
        public double LengthInKm { get; set; }

        public string? WalkImageUrl { get; set; }

        [Required(ErrorMessage = "ID độ khó là bắt buộc.")]
        public Guid DifficultyId { get; set; }

        [Required(ErrorMessage = "ID khu vực là bắt buộc.")]
        public Guid RegionId { get; set; }
    }
}
```

Và `UpdateWalkRequestDto` có cấu trúc tương tự:

```csharp
using System.ComponentModel.DataAnnotations;

namespace NZWalks.API.Models.DTO
{
    public class UpdateWalkRequestDto
    {
        [Required(ErrorMessage = "Tên chuyến đi là bắt buộc.")]
        [MaxLength(100, ErrorMessage = "Tên chuyến đi không được vượt quá 100 ký tự.")]
        public string Name { get; set; }

        [Required(ErrorMessage = "Mô tả chuyến đi là bắt buộc.")]
        [MinLength(10, ErrorMessage = "Mô tả phải có ít nhất 10 ký tự.")]
        [MaxLength(1000, ErrorMessage = "Mô tả không được vượt quá 1000 ký tự.")]
        public string Description { get; set; }

        [Required(ErrorMessage = "Độ dài là bắt buộc.")]
        [Range(0.01, 50.0, ErrorMessage = "Độ dài phải nằm trong khoảng từ 0.01 đến 50 km.")]
        public double LengthInKm { get; set; }

        public string? WalkImageUrl { get; set; }

        [Required(ErrorMessage = "ID độ khó là bắt buộc.")]
        public Guid DifficultyId { get; set; }

        [Required(ErrorMessage = "ID khu vực là bắt buộc.")]
        public Guid RegionId { get; set; }
    }
}
```

## 3. Xử lý kết quả xác thực trong Controller: `ModelState` và `BadRequest`

Sau khi đã áp dụng các Data Annotations vào DTO, bước tiếp theo là kiểm tra xem dữ liệu đầu vào có hợp lệ hay không bên trong các action methods của Controller. ASP.NET Core cung cấp thuộc tính `ModelState` trong `ControllerBase` (mà `Controller` kế thừa) để thực hiện việc này.

### 3.1. `ModelState`: Trạng thái xác thực của Model

`ModelState` là một thuộc tính kiểu `ModelStateDictionary` có sẵn trong tất cả các Controller kế thừa từ `ControllerBase`. Nó đóng vai trò là một từ điển chứa trạng thái xác thực của đối tượng model được bind từ request.

Các thành phần chính của `ModelState`:

*   **`IsValid`:** Một thuộc tính boolean quan trọng, cho biết liệu toàn bộ model có hợp lệ theo tất cả các quy tắc đã định nghĩa (Data Annotations) hay không. Nếu `true`, không có lỗi nào được tìm thấy. Nếu `false`, có ít nhất một lỗi.
*   **Các mục lỗi:** `ModelState` chứa một tập hợp các `ModelStateEntry`, mỗi entry đại diện cho trạng thái của một thuộc tính hoặc của toàn bộ model. Mỗi `ModelStateEntry` có thể chứa một danh sách các `ModelError` mô tả lỗi cụ thể.

`ModelState` được điền tự động bởi Model Binding pipeline. Khi dữ liệu từ request được cố gắng ánh xạ vào DTO, ASP.NET Core sẽ kiểm tra các Data Annotations. Bất kỳ lỗi nào phát hiện được sẽ được ghi vào `ModelState`.

### 3.2. Triển khai kiểm tra `ModelState.IsValid` trong Controller

Sau khi Model Binding hoàn tất, trước khi thực thi bất kỳ logic nghiệp vụ nào, chúng ta cần kiểm tra `ModelState.IsValid`. Nếu nó là `false`, chúng ta sẽ trả về phản hồi `HTTP 400 Bad Request` cùng với chi tiết các lỗi từ `ModelState`.

```csharp
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class AreasController : ControllerBase
    {
        private readonly IAreaRepository areaRepository;
        private readonly IMapper mapper;

        // Dependency Injection: Controller nhận các dịch vụ qua constructor
        public AreasController(IAreaRepository areaRepository, IMapper mapper)
        {
            this.areaRepository = areaRepository;
            this.mapper = mapper;
        }

        // ... Các phương thức GET khác ...

        [HttpPost] // HTTP Verb: POST để tạo tài nguyên mới
        public async Task<IActionResult> Create([FromBody] AddAreaRequestDto addAreaRequestDto)
        {
            // Bước 1: Kiểm tra trạng thái Model sau khi Model Binding
            if (!ModelState.IsValid)
            {
                // Nếu không hợp lệ, trả về lỗi 400 Bad Request kèm chi tiết lỗi
                return BadRequest(ModelState);
            }

            // Bước 2: Chuyển đổi DTO sang Domain Model (sử dụng AutoMapper)
            var areaDomainModel = mapper.Map<Models.Domain.Area>(addAreaRequestDto);

            // Bước 3: Sử dụng Repository để thêm vào DB
            // Repository Pattern: Tách biệt logic truy cập dữ liệu
            areaDomainModel = await areaRepository.CreateAsync(areaDomainModel);

            // Bước 4: Chuyển đổi Domain Model trở lại DTO để trả về client
            var areaDto = mapper.Map<Models.DTO.AreaDto>(areaDomainModel);

            // Bước 5: Trả về kết quả HTTP 201 Created với URI của tài nguyên mới
            return CreatedAtAction(nameof(GetById), new { id = areaDto.Id }, areaDto);
        }

        [HttpPut] // HTTP Verb: PUT để cập nhật tài nguyên hiện có
        [Route("{id:Guid}")]
        public async Task<IActionResult> Update([FromRoute] Guid id, [FromBody] UpdateAreaRequestDto updateAreaRequestDto)
        {
            // Bước 1: Kiểm tra trạng thái Model sau khi Model Binding
            if (!ModelState.IsValid)
            {
                // Nếu không hợp lệ, trả về lỗi 400 Bad Request kèm chi tiết lỗi
                return BadRequest(ModelState);
            }

            // Bước 2: Chuyển đổi DTO sang Domain Model
            var areaDomainModel = mapper.Map<Models.Domain.Area>(updateAreaRequestDto);

            // Bước 3: Sử dụng Repository để cập nhật
            areaDomainModel = await areaRepository.UpdateAsync(id, areaDomainModel);

            // Bước 4: Kiểm tra xem tài nguyên có tồn tại không
            if (areaDomainModel == null)
            {
                return NotFound(); // Trả về HTTP 404 Not Found nếu không tìm thấy
            }

            // Bước 5: Chuyển đổi Domain Model trở lại DTO để trả về client
            var areaDto = mapper.Map<Models.DTO.AreaDto>(areaDomainModel);

            // Bước 6: Trả về kết quả HTTP 200 OK
            return Ok(areaDto);
        }

        // ... Các phương thức DELETE khác ...
    }
}
```

> [!NOTE]
> Khi bạn gọi `return BadRequest(ModelState);`, ASP.NET Core sẽ tự động serialize nội dung của `ModelState` thành một phản hồi JSON. Phản hồi này thường có cấu trúc như sau:
>
> ```json
> {
>   "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
>   "title": "One or more validation errors occurred.",
>   "status": 400,
>   "traceId": "|<some-id>|",
>   "errors": {
>     "Code": [
>       "Mã khu vực phải có ít nhất 3 ký tự.",
>       "Mã khu vực phải có tối đa 3 ký tự."
>     ],
>     "Name": [
>       "Tên khu vực là bắt buộc."
>     ]
>   }
> }
> ```
> Cấu trúc này rất hữu ích cho các client để hiển thị thông báo lỗi cho người dùng cuối.

Bạn sẽ cần lặp lại logic kiểm tra `ModelState.IsValid` này cho tất cả các endpoint `POST` và `PUT` trong các controller khác, ví dụ như `WalksController`. Tuy nhiên, việc lặp lại này dẫn đến mã trùng lặp, và chúng ta sẽ giải quyết nó trong phần tiếp theo.

## 4. Tối ưu hóa xác thực với Custom Action Filter: Loại bỏ mã trùng lặp

Việc lặp lại đoạn mã `if (!ModelState.IsValid) { return BadRequest(ModelState); }` trong mỗi action method là một sự trùng lặp mã (boilerplate code) rõ ràng và vi phạm nguyên tắc DRY (Don't Repeat Yourself - Đừng lặp lại chính mình). Để giải quyết vấn đề này một cách thanh lịch và hiệu quả trong ASP.NET Core, chúng ta sẽ sử dụng một kỹ thuật nâng cao: **Custom Action Filter**.

### 4.1. Vấn đề của việc kiểm tra lặp lại `ModelState.IsValid`

Trong phần trước, chúng ta đã thấy rằng mỗi khi tạo một action method mới nhận dữ liệu đầu vào (như `Create` hoặc `Update`), chúng ta đều phải thêm câu lệnh kiểm tra `ModelState.IsValid`. Điều này làm cho code của controller trở nên dài dòng, khó đọc và khó bảo trì. Nếu có bất kỳ thay đổi nào trong cách xử lý lỗi xác thực (ví dụ: muốn ghi log lỗi trước khi trả về, hoặc muốn định dạng phản hồi lỗi khác), chúng ta sẽ phải cập nhật ở nhiều nơi, tiềm ẩn nguy cơ sai sót.

### 4.2. Giới thiệu Action Filters trong ASP.NET Core

**Action Filters** là một loại Filter trong ASP.NET Core cho phép bạn thực thi logic tùy chỉnh trước hoặc sau khi một action method được thực thi. Chúng là một phần quan trọng của pipeline xử lý yêu cầu và cung cấp một cách mạnh mẽ để thêm hành vi xuyên suốt (cross-cutting concerns) vào ứng dụng của bạn mà không làm thay đổi logic nghiệp vụ cốt lõi trong controller.

Các loại filters khác nhau trong ASP.NET Core hoạt động tại các giai đoạn khác nhau của pipeline:

*   **Authorization Filters:** Chạy trước mọi thứ khác để xác định xem người dùng có quyền truy cập tài nguyên hay không.
*   **Resource Filters:** Chạy sau Authorization Filters và trước Model Binding.
*   **Action Filters:** Chạy trước và sau khi action method được thực thi. Đây là loại chúng ta sẽ sử dụng để kiểm tra `ModelState`.
*   **Exception Filters:** Xử lý các ngoại lệ không được xử lý trong pipeline.
*   **Result Filters:** Chạy trước và sau khi action result được thực thi.

Để tạo một Custom Action Filter, chúng ta sẽ tạo một lớp kế thừa từ `ActionFilterAttribute` và ghi đè phương thức `OnActionExecuting`.

### 4.3. Xây dựng Custom `ValidateModelAttribute`

Chúng ta sẽ tạo một lớp Action Filter có tên `ValidateModelAttribute` để tự động kiểm tra `ModelState.IsValid` trước khi action method được gọi.

1.  **Tạo thư mục `CustomActionFilters`:**
    Trong thư mục gốc của dự án API (ví dụ: `NZWalks.API`), tạo một thư mục mới có tên `CustomActionFilters`.

2.  **Tạo lớp `ValidateModelAttribute`:**
    Bên trong thư mục `CustomActionFilters`, tạo một lớp mới có tên `ValidateModelAttribute.cs`.

    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.AspNetCore.Mvc.Filters; // Quan trọng: Đừng quên namespace này

    namespace NZWalks.API.CustomActionFilters
    {
        /// <summary>
        /// Action Filter tùy chỉnh để tự động kiểm tra ModelState.IsValid
        /// và trả về BadRequest nếu model không hợp lệ.
        /// </summary>
        public class ValidateModelAttribute : ActionFilterAttribute
        {
            /// <summary>
            /// Phương thức này được gọi trước khi action method được thực thi.
            /// </summary>
            /// <param name="context">Ngữ cảnh thực thi action.</param>
            public override void OnActionExecuting(ActionExecutingContext context)
            {
                // Kiểm tra xem ModelState có hợp lệ hay không
                if (!context.ModelState.IsValid)
                {
                    // Nếu không hợp lệ, chúng ta thiết lập kết quả của context thành BadRequestObjectResult.
                    // Điều này sẽ ngăn action method gốc được thực thi (short-circuiting the pipeline)
                    // và trả về lỗi 400 Bad Request cùng với chi tiết lỗi từ ModelState.
                    context.Result = new BadRequestObjectResult(context.ModelState);
                }
            }
        }
    }
    ```

    **Giải thích chi tiết:**
    *   `ActionFilterAttribute`: Đây là lớp cơ sở để tạo Action Filter. Kế thừa từ nó cho phép lớp của chúng ta được sử dụng như một thuộc tính (`[ValidateModel]`).
    *   `OnActionExecuting(ActionExecutingContext context)`: Phương thức này là điểm mấu chốt. Nó được gọi **trước** khi action method mà filter được áp dụng lên đó được thực thi.
    *   `context.ModelState.IsValid`: Thuộc tính này cho phép chúng ta truy cập vào `ModelState` của request hiện tại để kiểm tra tính hợp lệ của dữ liệu đầu vào.
    *   `context.Result = new BadRequestObjectResult(context.ModelState);`: Đây là phần quan trọng nhất. Nếu `ModelState` không hợp lệ, chúng ta gán một `BadRequestObjectResult` vào thuộc tính `context.Result`. Khi `context.Result` được gán, ASP.NET Core sẽ hiểu rằng action đã được xử lý và kết quả này nên được trả về client ngay lập tức. Điều này có nghĩa là action method gốc (ví dụ: `Create` hoặc `Update`) sẽ **không bao giờ được gọi**, giúp chúng ta tiết kiệm tài nguyên và giữ cho logic nghiệp vụ chỉ xử lý dữ liệu hợp lệ.

> [!NOTE]
> **Antigravity IDE và Vibe Coding trong việc tạo Custom Action Filter:**
>
> Antigravity IDE, với khả năng lập kế hoạch và thực thi script ngầm, có thể hỗ trợ bạn tạo và áp dụng `ValidateModelAttribute` một cách tự động.
>
> 1.  **Phát hiện trùng lặp:** Antigravity có thể phân tích mã nguồn của bạn, nhận diện các đoạn `if (!ModelState.IsValid)` lặp lại trong nhiều controller.
> 2.  **Đề xuất giải pháp:** Dựa trên nguyên tắc DRY, nó có thể đề xuất tạo một Custom Action Filter.
> 3.  **Tự động tạo mã:** Với một lệnh "Generate ValidateModel filter" hoặc đơn giản là diễn đạt ý định bằng ngôn ngữ tự nhiên, Antigravity có thể tự động tạo file `ValidateModelAttribute.cs` với nội dung tương tự như trên.
> 4.  **Refactor tự động:** Không chỉ tạo, Antigravity có thể tự động đi qua tất cả các controller và thay thế các đoạn mã kiểm tra `ModelState.IsValid` bằng việc thêm thuộc tính `[ValidateModel]` vào các action method thích hợp.
>
> Điều này minh họa cách Antigravity giúp bạn áp dụng Vibe Coding: bạn tập trung vào "loại bỏ mã trùng lặp để xác thực model", và Antigravity đảm nhận "cách thức" thực hiện điều đó thông qua việc tạo và áp dụng Action Filter.

### 4.4. Áp dụng Custom Action Filter vào Controller

Bây giờ, thay vì viết `if (!ModelState.IsValid)` trong mỗi action method, chúng ta chỉ cần áp dụng thuộc tính `[ValidateModel]` lên các action methods cần xác thực.

```csharp
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.CustomActionFilters; // Quan trọng: Nhập namespace của Custom Action Filter
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class AreasController : ControllerBase
    {
        private readonly IAreaRepository areaRepository;
        private readonly IMapper mapper;

        // Dependency Injection: Controller nhận các dịch vụ qua constructor
        public AreasController(IAreaRepository areaRepository, IMapper mapper)
        {
            this.areaRepository = areaRepository;
            this.mapper = mapper;
        }

        // ... Các phương thức GET khác ...

        [HttpPost]
        [ValidateModel] // Áp dụng Custom Action Filter ở đây!
        public async Task<IActionResult> Create([FromBody] AddAreaRequestDto addAreaRequestDto)
        {
            // KHÔNG CẦN kiểm tra if (!ModelState.IsValid) nữa!
            // Logic này đã được xử lý bởi ValidateModelAttribute trước khi phương thức này được gọi.
            // Nếu model không hợp lệ, filter đã trả về BadRequest và phương thức này sẽ không được thực thi.

            // Chuyển đổi DTO sang Domain Model
            var areaDomainModel = mapper.Map<Models.Domain.Area>(addAreaRequestDto);

            // Sử dụng Repository để thêm vào DB
            areaDomainModel = await areaRepository.CreateAsync(areaDomainModel);

            // Chuyển đổi Domain Model trở lại DTO để trả về client
            var areaDto = mapper.Map<Models.DTO.AreaDto>(areaDomainModel);

            return CreatedAtAction(nameof(GetById), new { id = areaDto.Id }, areaDto);
        }

        [HttpPut]
        [Route("{id:Guid}")]
        [ValidateModel] // Áp dụng Custom Action Filter ở đây!
        public async Task<IActionResult> Update([FromRoute] Guid id, [FromBody] UpdateAreaRequestDto updateAreaRequestDto)
        {
            // KHÔNG CẦN kiểm tra if (!ModelState.IsValid) nữa!
            // Logic này đã được xử lý bởi ValidateModelAttribute.

            // Chuyển đổi DTO sang Domain Model
            var areaDomainModel = mapper.Map<Models.Domain.Area>(updateAreaRequestDto);

            // Sử dụng Repository để cập nhật
            areaDomainModel = await areaRepository.UpdateAsync(id, areaDomainModel);

            if (areaDomainModel == null)
            {
                return NotFound();
            }

            // Chuyển đổi Domain Model trở lại DTO để trả về client
            var areaDto = mapper.Map<Models.DTO.AreaDto>(areaDomainModel);

            return Ok(areaDto);
        }

        // ... Các phương thức DELETE khác ...
    }
}
```

Tương tự, bạn sẽ áp dụng `[ValidateModel]` cho các phương thức `Create` và `Update` trong `WalksController` hoặc bất kỳ controller nào khác có các action method nhận dữ liệu đầu vào cần xác thực.

> [!TIP]
> **Lợi ích của Custom Action Filter:**
>
> *   **Giảm trùng lặp code (DRY):** Loại bỏ việc kiểm tra `ModelState.IsValid` lặp đi lặp lại trong mỗi action method.
> *   **Tăng tính dễ đọc và gọn gàng:** Controller methods trở nên gọn gàng hơn, tập trung hoàn toàn vào logic nghiệp vụ mà không bị xen kẽ bởi logic xác thực.
> *   **Dễ bảo trì:** Mọi thay đổi về cách xử lý lỗi xác thực có thể được thực hiện ở một nơi duy nhất (trong `ValidateModelAttribute`), giúp việc cập nhật và sửa lỗi trở nên dễ dàng hơn.
> *   **Khả năng tái sử dụng:** `ValidateModelAttribute` có thể được sử dụng trên bất kỳ action method nào trong toàn bộ ứng dụng, thậm chí có thể được đăng ký toàn cục nếu muốn áp dụng cho tất cả các action.

## 5. Các kỹ thuật xác thực nâng cao và trường hợp đặc biệt

Mặc dù Data Annotations và Custom Action Filters là những công cụ mạnh mẽ và đủ dùng cho hầu hết các trường hợp, đôi khi bạn cần các giải pháp linh hoạt hơn cho các kịch bản xác thực phức tạp.

### 5.1. Custom Validation Attributes: Khi Data Annotations không đủ

Các Data Annotations có sẵn rất tiện lợi, nhưng chúng có những giới hạn. Ví dụ, chúng không thể dễ dàng xử lý các quy tắc xác thực liên quan đến nhiều thuộc tính (ví dụ: `EndDate` phải lớn hơn `StartDate`), hoặc các quy tắc nghiệp vụ rất đặc thù (ví dụ: một mã sản phẩm phải có định dạng "ABC-123-X").

Trong những trường hợp này, bạn có thể tạo Custom Validation Attributes bằng cách kế thừa từ lớp `ValidationAttribute` và ghi đè phương thức `IsValid`.

```csharp
using System.ComponentModel.DataAnnotations;

namespace NZWalks.API.Models.DTO.CustomValidations
{
    /// <summary>
    /// Thuộc tính xác thực tùy chỉnh để đảm bảo một giá trị số là số chẵn.
    /// </summary>
    public class MustBeEvenAttribute : ValidationAttribute
    {
        public MustBeEvenAttribute()
        {
            // Thiết lập thông báo lỗi mặc định
            ErrorMessage = "Giá trị '{0}' phải là số chẵn.";
        }

        /// <summary>
        /// Ghi đè phương thức IsValid để chứa logic xác thực tùy chỉnh.
        /// </summary>
        /// <param name="value">Giá trị của thuộc tính đang được xác thực.</param>
        /// <param name="validationContext">Ngữ cảnh xác thực, chứa thông tin về đối tượng.</param>
        /// <returns>ValidationResult.Success nếu hợp lệ, ngược lại là ValidationResult với lỗi.</returns>
        protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
        {
            // Kiểm tra nếu giá trị là null, coi là hợp lệ (nếu cần Required, hãy dùng [Required] riêng)
            if (value == null)
            {
                return ValidationResult.Success;
            }

            // Cố gắng chuyển đổi giá trị sang kiểu int
            if (value is int intValue)
            {
                if (intValue % 2 == 0)
                {
                    return ValidationResult.Success; // Hợp lệ
                }
            }
            // Nếu không phải int hoặc không chẵn
            return new ValidationResult(FormatErrorMessage(validationContext.DisplayName));
        }
    }
}

// Cách sử dụng trong DTO:
// public class MyDto
// {
//     [Required]
//     [MustBeEven(ErrorMessage = "Số lượng sản phẩm phải là một số chẵn.")]
//     public int Quantity { get; set; }
// }
```

> [!NOTE]
> **Antigravity IDE và Vibe Coding trong việc tạo Custom Validation Attributes:**
>
> Khi đối mặt với các quy tắc nghiệp vụ phức tạp, bạn có thể tận dụng Antigravity IDE để:
>
> 1.  **Diễn đạt quy tắc:** Mô tả quy tắc xác thực bằng ngôn ngữ tự nhiên: "Trường `EndDate` phải sau `StartDate`", hoặc "Tên người dùng không được chứa ký tự đặc biệt trừ gạch dưới và dấu chấm."
> 2.  **Antigravity lập kế hoạch:** Hệ thống Agentic AI có thể phân tích yêu cầu, nhận ra rằng Data Annotations sẵn có không đủ, và đề xuất tạo một Custom Validation Attribute.
> 3.  **Tự động tạo logic:** Antigravity có thể tự động tạo cấu trúc của `ValidationAttribute` và thậm chí viết logic bên trong phương thức `IsValid` dựa trên mô tả của bạn. Nó có thể gọi subagent để tìm kiếm các ví dụ regex hoặc logic so sánh ngày tháng nếu cần.
> 4.  **Tích hợp vào DTO:** Sau khi tạo, Antigravity có thể hướng dẫn bạn hoặc tự động áp dụng thuộc tính này vào DTO tương ứng.
>
> Cách tiếp cận này cho phép bạn "vibe code" các quy tắc phức tạp mà không cần sa đà vào chi tiết cú pháp ban đầu, để Antigravity xử lý việc chuyển đổi từ ý định sang mã nguồn thực thi.

### 5.2. FluentValidation: Giải pháp mạnh mẽ cho xác thực phức tạp

Đối với các dự án lớn hơn, có nhiều DTO, hoặc có yêu cầu xác thực cực kỳ phức tạp (ví dụ: xác thực phụ thuộc vào trạng thái cơ sở dữ liệu, xác thực có điều kiện phức tạp, hoặc muốn tách biệt hoàn toàn logic xác thực khỏi DTO), nhiều nhà phát triển lựa chọn thư viện bên thứ ba như **FluentValidation**.

**Ưu điểm của FluentValidation:**

*   **Tách biệt mối quan tâm:** Logic xác thực được đặt trong các lớp riêng biệt (validators), tách rời hoàn toàn khỏi DTO. Điều này giúp DTO sạch sẽ hơn và validator dễ kiểm thử hơn.
*   **API dễ đọc (Fluent API):** Cung cấp một cú pháp rất dễ đọc và mạnh mẽ để định nghĩa các quy tắc xác thực.
*   **Hỗ trợ các quy tắc phức tạp:** Cho phép định nghĩa các quy tắc liên quan đến nhiều thuộc tính, quy tắc có điều kiện (ví dụ: chỉ xác thực trường X nếu trường Y có giá trị Z), hoặc thậm chí gọi đến dịch vụ bên ngoài để xác thực.
*   **Tích hợp tốt với ASP.NET Core:** Có các gói NuGet để tích hợp FluentValidation vào pipeline xác thực của ASP.NET Core, cho phép nó hoạt động liền mạch với `ModelState`.

**Ví dụ khái niệm về FluentValidation:**

```csharp
// 1. DTO đơn giản, không có Data Annotations
public class AddProductRequestDto
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int Quantity { get; set; }
}

// 2. Lớp Validator riêng biệt
using FluentValidation;

public class AddProductRequestDtoValidator : AbstractValidator<AddProductRequestDto>
{
    public AddProductRequestDtoValidator()
    {
        RuleFor(product => product.Name)
            .NotEmpty().WithMessage("Tên sản phẩm là bắt buộc.")
            .Length(3, 100).WithMessage("Tên sản phẩm phải từ 3 đến 100 ký tự.");

        RuleFor(product => product.Price)
            .GreaterThan(0).WithMessage("Giá sản phẩm phải lớn hơn 0.")
            .ScalePrecision(2, 10).WithMessage("Giá sản phẩm có tối đa 2 chữ số thập phân và 10 chữ số tổng cộng.");

        RuleFor(product => product.Quantity)
            .GreaterThanOrEqualTo(0).WithMessage("Số lượng phải là số không âm.");

        // Ví dụ quy tắc phức tạp hơn: Quantity phải là số chẵn nếu Price > 100
        RuleFor(product => product.Quantity)
            .Must(quantity => quantity % 2 == 0)
            .When(product => product.Price > 100)
            .WithMessage("Số lượng phải là số chẵn nếu giá sản phẩm lớn hơn 100.");
    }
}
```

> [!NOTE]
> **Antigravity IDE và FluentValidation:**
>
> Antigravity có thể là một công cụ cực kỳ mạnh mẽ khi làm việc với FluentValidation:
>
> 1.  **Phân tích yêu cầu:** Bạn có thể cung cấp cho Antigravity các yêu cầu nghiệp vụ phức tạp bằng ngôn ngữ tự nhiên.
> 2.  **Tạo Validator:** Antigravity có thể tự động tạo lớp validator cho DTO của bạn, điền vào các quy tắc FluentValidation dựa trên phân tích yêu cầu và thậm chí tự động đăng ký validator vào hệ thống Dependency Injection của ASP.NET Core.
> 3.  **Tối ưu hóa quy tắc:** Đối với các quy tắc xác thực phụ thuộc vào cơ sở dữ liệu (ví dụ: kiểm tra tính duy nhất của một trường), Antigravity có thể lập kế hoạch để tạo các phương thức kiểm tra trong repository và tích hợp chúng vào validator.
> 4.  **Refactor:** Nếu bạn đang chuyển từ Data Annotations sang FluentValidation, Antigravity có thể thực hiện quá trình refactor bằng cách đọc các Data Annotations hiện có và chuyển đổi chúng thành các quy tắc FluentValidation tương ứng, sau đó loại bỏ các Data Annotations khỏi DTO gốc.

## Tóm tắt Phần 8: Xác thực dữ liệu đầu vào (Model Validations)

*   **Tầm quan trọng cốt lõi:** Xác thực dữ liệu đầu vào là yếu tố then chốt để đảm bảo tính toàn vẹn, bảo mật, ổn định của Web API và cung cấp phản hồi rõ ràng cho client.
*   **HTTP 400 Bad Request:** Là mã trạng thái HTTP tiêu chuẩn được sử dụng để phản hồi khi dữ liệu đầu vào không hợp lệ, kèm theo chi tiết lỗi.
*   **Data Annotations:** Cơ chế xác thực tích hợp sẵn trong ASP.NET Core, sử dụng các thuộc tính (`[Required]`, `[MinLength]`, `[Range]`, v.v.) trực tiếp trên các thuộc tính của DTO.
*   **DTOs (Data Transfer Objects):** Nên áp dụng xác thực trên DTOs thay vì Domain Models để tách biệt mối quan tâm, kiểm soát dữ liệu đầu vào/đầu ra và giữ Domain Models sạch sẽ.
*   **Cơ chế `ModelState`:** Một từ điển trong `ControllerBase` chứa trạng thái xác thực của model sau Model Binding. Thuộc tính `ModelState.IsValid` cho biết model có hợp lệ hay không.
*   **`BadRequest(ModelState)`:** Phương thức trả về phản hồi HTTP 400 Bad Request kèm theo chi tiết các lỗi xác thực từ `ModelState`, được tự động serialize thành JSON.
*   **Custom Action Filter (`ValidateModelAttribute`):** Kỹ thuật nâng cao để tối ưu hóa mã nguồn, loại bỏ việc kiểm tra `if (!ModelState.IsValid)` lặp lại trong mỗi action method. Nó sử dụng `ActionFilterAttribute` và ghi đè `OnActionExecuting` để chặn và xử lý yêu cầu trước khi action method được gọi.
*   **Server-side Validation là bắt buộc:** Luôn phải thực hiện xác thực ở phía server, ngay cả khi đã có client-side validation, để đảm bảo tính toàn vẹn và bảo mật của dữ liệu.
*   **Các phương pháp mở rộng:**
    *   **Custom Validation Attributes:** Tạo các thuộc tính xác thực tùy chỉnh bằng cách kế thừa `ValidationAttribute` cho các quy tắc phức tạp hoặc liên quan đến nhiều thuộc tính.
    *   **FluentValidation:** Thư viện bên thứ ba mạnh mẽ, cung cấp API dễ đọc và khả năng định nghĩa các quy tắc xác thực phức tạp trong các lớp validator riêng biệt, tăng tính dễ đọc và khả năng kiểm thử.
*   **Vibe Coding với Antigravity IDE:** Tận dụng Antigravity để gợi ý, tự động tạo và refactor mã validation (Data Annotations, Custom Attributes, FluentValidation) dựa trên mô tả ngôn ngữ tự nhiên và nguyên tắc thiết kế, giúp tăng tốc phát triển và duy trì mã sạch.

Bằng cách nắm vững các kỹ thuật xác thực này, bạn có thể xây dựng các Web API mạnh mẽ, an toàn và dễ sử dụng, là nền tảng vững chắc cho mọi ứng dụng hiện đại.

<!-- REVIEWED_BY_AGENT -->
