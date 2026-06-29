# Phần 28: Xác thực dữ liệu đầu vào (Model Validations) trong RESTful Web API

Trong kiến trúc RESTful Web API, việc trao đổi dữ liệu giữa ứng dụng khách (client applications) và máy chủ là nền tảng. Khi API của chúng ta cho phép ứng dụng khách gửi dữ liệu (ví dụ: tạo mới hoặc cập nhật tài nguyên), chất lượng và định dạng của dữ liệu đó trở thành yếu tố tối quan trọng. Dữ liệu không hợp lệ không chỉ tiềm ẩn nguy cơ làm hỏng cơ sở dữ liệu bằng "dữ liệu rác" (junk data) mà còn có thể gây ra các ngoại lệ không mong muốn, làm gián đoạn hoạt động của API và ảnh hưởng tiêu cực đến trải nghiệm người dùng.

Chương này đi sâu vào việc triển khai cơ chế xác thực dữ liệu đầu vào (Model Validations) cho các API mà chúng ta đã xây dựng trong dự án thực hành. Chúng ta sẽ khám phá tầm quan trọng của xác thực, các phương pháp triển khai trong ASP.NET Core, và cách thông báo lỗi một cách hiệu quả cho ứng dụng khách bằng mã trạng thái HTTP 400 Bad Request. Mặc dù chúng ta sẽ bắt đầu với phương pháp xác thực thủ công để hiểu rõ cơ chế cốt lõi, chương này cũng sẽ mở rộng kiến thức về các phương pháp tiêu chuẩn và mạnh mẽ hơn như Data Annotations và FluentValidation, giúp bạn xây dựng các API bền vững và dễ bảo trì trong thực tế.

Mục tiêu chính của chương này là:

*   Hiểu rõ tầm quan trọng và vai trò của xác thực dữ liệu đầu vào trong API.
*   Nắm vững cơ chế `ModelState` và cách nó được sử dụng để thu thập và quản lý lỗi xác thực.
*   Triển khai xác thực thủ công cho các Model Request của các Controller `Region`, `Walks` và `WalkDifficulty`.
*   Sử dụng mã trạng thái HTTP `400 Bad Request` để báo hiệu lỗi xác thực cho ứng dụng khách.
*   Hiểu cách kết hợp Dependency Injection và Repository Pattern vào quy trình xác thực, đặc biệt khi cần kiểm tra sự tồn tại của dữ liệu liên quan trong cơ sở dữ liệu.
*   Bổ sung kiến thức về các phương pháp xác thực tiên tiến hơn trong ASP.NET Core như Data Annotations, `IValidatableObject` và FluentValidation.
*   Áp dụng tư duy Vibe Coding và khai thác công cụ Agentic AI như Antigravity IDE để tăng cường hiệu quả trong quá trình phát triển và kiểm thử xác thực.

## 1. Tầm quan trọng cốt lõi của xác thực dữ liệu đầu vào

Xác thực dữ liệu đầu vào là một tuyến phòng thủ thiết yếu, không thể thiếu trong bất kỳ ứng dụng nào, đặc biệt là trong các RESTful API. Nó đảm bảo rằng dữ liệu mà API nhận được tuân thủ các quy tắc đã định trước, từ định dạng cơ bản đến các ràng buộc nghiệp vụ phức tạp.

### 1.1. Tại sao phải xác thực dữ liệu?

Các lý do chính cho việc phải xác thực dữ liệu bao gồm:

*   **Bảo vệ tính toàn vẹn dữ liệu:** Ngăn chặn dữ liệu không hợp lệ, không chính xác hoặc độc hại được lưu trữ trong cơ sở dữ liệu. Điều này đảm bảo rằng dữ liệu luôn đáng tin cậy và không gây ra các vấn đề về logic nghiệp vụ hoặc phân tích sau này.
*   **Ngăn chặn lỗi ứng dụng (Runtime Errors):** Dữ liệu không đúng định dạng có thể dẫn đến các ngoại lệ không mong muốn trong quá trình xử lý. Ví dụ, cố gắng chuyển đổi một chuỗi không phải số thành một số nguyên sẽ gây ra lỗi runtime. Xác thực sớm giúp bắt các lỗi này trước khi chúng gây ra sự cố cho hệ thống.
*   **Cải thiện trải nghiệm người dùng (UX):** Thay vì nhận một lỗi `500 Internal Server Error` chung chung (thường chỉ ra lỗi phía máy chủ) khi gửi dữ liệu sai, ứng dụng khách sẽ nhận được phản hồi `400 Bad Request` với thông tin chi tiết và cụ thể về các trường bị lỗi. Điều này giúp người dùng hoặc các ứng dụng tích hợp dễ dàng hiểu và sửa chữa lỗi của họ.
*   **Tăng cường bảo mật:** Xác thực giúp ngăn chặn một số loại tấn công phổ biến như SQL Injection hoặc Cross-Site Scripting (XSS) bằng cách đảm bảo rằng dữ liệu đầu vào không chứa các ký tự hoặc cấu trúc độc hại. Mặc dù không phải là biện pháp bảo mật duy nhất, nó là một lớp phòng thủ quan trọng.
*   **Tuân thủ logic nghiệp vụ:** Đảm bảo rằng dữ liệu tuân thủ các quy tắc và ràng buộc nghiệp vụ của ứng dụng. Ví dụ: một giá trị phải lớn hơn 0, một địa chỉ email phải có định dạng hợp lệ, hoặc một ngày kết thúc không thể trước ngày bắt đầu.

### 1.2. HTTP 400 Bad Request: Mã trạng thái chuẩn cho lỗi xác thực

Khi một yêu cầu HTTP được gửi đến máy chủ nhưng dữ liệu trong yêu cầu không hợp lệ hoặc không đúng định dạng theo yêu cầu của API, máy chủ sẽ trả về mã trạng thái `400 Bad Request`. Đây là cách tiêu chuẩn và được khuyến nghị để API thông báo về các lỗi xác thực.

**Cơ chế phản hồi:** Khi ASP.NET Core phát hiện lỗi xác thực (thông qua `ModelState`), nó sẽ tự động tạo một phản hồi `400 Bad Request`. Nội dung của phản hồi thường là một đối tượng JSON chứa thông tin chi tiết về các lỗi, bao gồm tên trường bị lỗi và thông báo lỗi tương ứng. Điều này cho phép ứng dụng khách dễ dàng phân tích và hiển thị lỗi cho người dùng cuối.

## 2. Cơ chế `ModelState` và vai trò của `[ApiController]`

Trong ASP.NET Core, `ModelState` là một đối tượng trung tâm để quản lý trạng thái xác thực của dữ liệu đầu vào. Nó là một tập hợp các cặp khóa-giá trị, nơi khóa thường là tên của một thuộc tính trên Model và giá trị là một `ModelStateEntry` chứa thông tin về trạng thái xác thực và bất kỳ lỗi nào liên quan đến thuộc tính đó.

### 2.1. `ModelState` hoạt động như thế nào?

Khi một yêu cầu HTTP đến một Action Method trong Controller, ASP.NET Core Model Binding sẽ cố gắng ánh xạ dữ liệu từ yêu cầu (ví dụ: từ body JSON, query string, form data) vào các tham số của Action Method. Trong quá trình này, `ModelState` được điền với:

*   **Trạng thái ràng buộc (Binding State):** Cho biết liệu dữ liệu có được ràng buộc thành công vào thuộc tính hay không (ví dụ: cố gắng gán chuỗi "abc" vào một thuộc tính `int` sẽ gây lỗi ràng buộc).
*   **Lỗi xác thực (Validation Errors):** Nếu có bất kỳ quy tắc xác thực nào bị vi phạm (dù là tự động từ Data Annotations hay thủ công), các lỗi này sẽ được thêm vào `ModelState`.

Bạn có thể kiểm tra trạng thái tổng thể của `ModelState` bằng thuộc tính `ModelState.IsValid`. Nếu `true`, không có lỗi nào được tìm thấy. Nếu `false`, có ít nhất một lỗi xác thực hoặc lỗi ràng buộc.

Sử dụng `ModelState.AddModelError(key, errorMessage)` là cách để bạn thêm các lỗi xác thực tùy chỉnh vào `ModelState`. `key` thường là tên thuộc tính bị lỗi, giúp ứng dụng khách dễ dàng xác định trường nào cần sửa.

### 2.2. Tác động của thuộc tính `[ApiController]`

Thuộc tính `[ApiController]` được sử dụng trên các Controller trong Web API để kích hoạt một số hành vi tiện lợi theo quy ước API. Một trong những tính năng quan trọng nhất mà nó cung cấp là **tự động phản hồi HTTP 400 Bad Request khi `ModelState` không hợp lệ**.

**Cơ chế ngầm (Under the Hood):**
Khi một Controller được đánh dấu bằng `[ApiController]`, ASP.NET Core sẽ tự động thêm một bộ lọc (filter) vào pipeline xử lý yêu cầu. Bộ lọc này sẽ kiểm tra `ModelState.IsValid` *trước khi Action Method của bạn được thực thi*.

*   **Nếu `ModelState.IsValid` là `false`:** Bộ lọc sẽ tự động tạo và trả về một `ValidationProblemDetails` object (theo chuẩn RFC 7807) với mã trạng thái `400 Bad Request`. Điều này có nghĩa là bạn không cần phải viết `if (!ModelState.IsValid) { return BadRequest(ModelState); }` thủ công ở đầu mỗi Action Method khi sử dụng Data Annotations.
*   **Nếu `ModelState.IsValid` là `true`:** Yêu cầu sẽ tiếp tục được xử lý bởi Action Method của bạn.

Trong phần này, vì chúng ta đang học về xác thực thủ công, chúng ta vẫn sẽ tự kiểm tra `ModelState.IsValid` và trả về `BadRequest(ModelState)` để hiểu rõ cơ chế. Tuy nhiên, hãy nhớ rằng `[ApiController]` tự động hóa bước này cho các phương pháp xác thực tích hợp sẵn.

## 3. Các phương pháp xác thực trong ASP.NET Core (Tổng quan)

ASP.NET Core cung cấp nhiều cách để triển khai xác thực dữ liệu đầu vào, từ đơn giản đến rất linh hoạt. Việc lựa chọn phương pháp phù hợp phụ thuộc vào độ phức tạp của logic xác thực và yêu cầu của dự án.

### 3.1. Xác thực bằng Data Annotations

Data Annotations là các thuộc tính (attributes) được đặt trực tiếp trên các thuộc tính của Model (thường là DTOs - Data Transfer Objects). Đây là cách đơn giản và phổ biến nhất để thêm xác thực cho các Model trong ASP.NET Core.

**Ưu điểm:**

*   **Đơn giản, dễ sử dụng:** Dễ dàng thêm các quy tắc xác thực cơ bản như `[Required]`, `[StringLength]`, `[Range]`, `[RegularExpression]`.
*   **Gắn liền với Model:** Logic xác thực nằm ngay cạnh định nghĩa thuộc tính, dễ dàng nhìn thấy.
*   **Tích hợp sẵn:** Hoạt động liền mạch với Model Binding và `[ApiController]`.

**Nhược điểm:**

*   **Hạn chế cho logic phức tạp:** Khó thực hiện các xác thực liên quan đến nhiều thuộc tính (`cross-property validation`) hoặc xác thực yêu cầu truy cập cơ sở dữ liệu/dịch vụ khác.
*   **Tái sử dụng:** Khó tái sử dụng logic xác thực cho các Model khác nhau.
*   **Gây "ô nhiễm" Model:** Đôi khi làm cho các DTO trở nên cồng kềnh với nhiều thuộc tính xác thực.

**Ví dụ:**

```csharp
// Models/DTOs/AddRegionRequest.cs
using System.ComponentModel.DataAnnotations;

namespace NZWalks.API.Models.DTOs
{
    public class AddRegionRequest
    {
        [Required(ErrorMessage = "Mã khu vực là bắt buộc.")]
        [StringLength(5, MinimumLength = 2, ErrorMessage = "Mã khu vực phải có từ 2 đến 5 ký tự.")]
        [RegularExpression(@"^[A-Z]{2,5}$", ErrorMessage = "Mã khu vực phải là chữ cái in hoa.")]
        public string Code { get; set; }

        [Required(ErrorMessage = "Tên khu vực là bắt buộc.")]
        [StringLength(100, MinimumLength = 3, ErrorMessage = "Tên khu vực phải có từ 3 đến 100 ký tự.")]
        public string Name { get; set; }

        [Range(0.01, double.MaxValue, ErrorMessage = "Diện tích phải lớn hơn 0.")]
        public double Area { get; set; }

        [Range(-90.0, 90.0, ErrorMessage = "Vĩ độ phải nằm trong khoảng từ -90 đến 90.")]
        public double Lat { get; set; }

        [Range(-180.0, 180.0, ErrorMessage = "Kinh độ phải nằm trong khoảng từ -180 đến 180.")]
        public double Long { get; set; }

        [Range(0, long.MaxValue, ErrorMessage = "Dân số không thể là số âm.")]
        public long Population { get; set; }
    }
}
```

> [!TIP]
> **Khi nào sử dụng Data Annotations?**
> *   Đối với các xác thực đơn giản, độc lập với ngữ cảnh và chỉ liên quan đến một thuộc tính duy nhất.
> *   Khi bạn muốn giữ logic xác thực gần với định nghĩa Model và không có các phụ thuộc phức tạp (ví dụ: không cần truy cập cơ sở dữ liệu để xác thực).
> *   Đối với các API nhỏ và vừa, nơi sự đơn giản được ưu tiên.

### 3.2. Xác thực bằng `IValidatableObject`

Để xử lý các xác thực phức tạp hơn mà Data Annotations đơn thuần không thể giải quyết, chẳng hạn như xác thực liên quan đến nhiều thuộc tính của cùng một đối tượng (cross-property validation) hoặc các ràng buộc nghiệp vụ có điều kiện, bạn có thể triển khai giao diện `IValidatableObject` trên Model của mình.

**Cơ chế ngầm:** Khi Model Binding hoàn tất và các Data Annotations đã được kiểm tra, ASP.NET Core sẽ gọi phương thức `Validate` của `IValidatableObject` nếu Model của bạn triển khai giao diện này.

**Ưu điểm:**

*   **Xác thực liên thuộc tính:** Dễ dàng so sánh hoặc kết hợp giá trị của nhiều thuộc tính để đưa ra quyết định xác thực.
*   **Logic nghiệp vụ tùy chỉnh:** Cho phép viết logic xác thực phức tạp hơn bằng code C# thuần túy.

**Nhược điểm:**

*   **Vẫn nằm trong Model:** Logic xác thực vẫn nằm trong Model, có thể làm Model trở nên cồng kềnh nếu logic quá phức tạp.
*   **Không hỗ trợ DI:** Không thể tiêm các dịch vụ (như Repository) trực tiếp vào phương thức `Validate` một cách dễ dàng, giới hạn khả năng xác thực dựa trên dữ liệu bên ngoài.

**Ví dụ:**

```csharp
// Models/DTOs/AddWalkRequest.cs
using System.ComponentModel.DataAnnotations;
using System.Collections.Generic;

namespace NZWalks.API.Models.DTOs
{
    public class AddWalkRequest : IValidatableObject
    {
        [Required]
        [StringLength(100, MinimumLength = 3)]
        public string Name { get; set; }

        [Range(0.01, double.MaxValue)]
        public double Length { get; set; }

        public Guid RegionId { get; set; }
        public Guid WalkDifficultyId { get; set; }

        public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
        {
            // Ví dụ: Xác thực liên thuộc tính
            if (Name == "Khó khăn" && Length < 5)
            {
                yield return new ValidationResult(
                    "Độ dài phải ít nhất 5 nếu tên là 'Khó khăn'.",
                    new[] { nameof(Length) });
            }

            // Ví dụ: Xác thực có điều kiện
            // Giả sử có một thuộc tính IsLoop. Nếu IsLoop là true, Length phải là số chẵn.
            // if (IsLoop && Length % 2 != 0)
            // {
            //     yield return new ValidationResult(
            //         "Độ dài phải là số chẵn cho các tuyến đường vòng lặp.",
            //         new[] { nameof(Length) });
            // }

            // Lưu ý: Xác thực sự tồn tại của RegionId/WalkDifficultyId trong DB không thể làm trực tiếp ở đây
            // vì IValidatableObject không hỗ trợ Dependency Injection dễ dàng.
        }
    }
}
```

### 3.3. Xác thực bằng FluentValidation

FluentValidation là một thư viện xác thực của bên thứ ba, cung cấp một cách tiếp cận mạnh mẽ và linh hoạt hơn nhiều so với Data Annotations và `IValidatableObject`. Nó cho phép bạn định nghĩa các quy tắc xác thực trong các lớp riêng biệt (Validator classes), tách biệt hoàn toàn logic xác thực khỏi Model.

**Ưu điểm:**

*   **Tách biệt logic:** Logic xác thực hoàn toàn tách rời khỏi Model, giúp Model sạch sẽ và dễ đọc hơn.
*   **Linh hoạt và mạnh mẽ:** Cung cấp API fluent, dễ đọc để định nghĩa các quy tắc phức tạp, có điều kiện, hoặc liên thuộc tính.
*   **Hỗ trợ Dependency Injection:** Các lớp Validator có thể nhận các dịch vụ thông qua DI (ví dụ: Repository để kiểm tra sự tồn tại trong DB), cho phép xác thực dựa trên dữ liệu bên ngoài hoặc trạng thái ứng dụng.
*   **Dễ kiểm thử:** Các Validator có thể được kiểm thử độc lập mà không cần phải khởi tạo toàn bộ Controller hoặc Model.
*   **Tái sử dụng cao:** Có thể tạo các quy tắc xác thực chung và tái sử dụng chúng.

**Nhược điểm:**

*   **Thêm phụ thuộc:** Cần cài đặt gói NuGet và cấu hình thêm một chút trong `Program.cs`.
*   **Độ phức tạp ban đầu:** Có thể cảm thấy phức tạp hơn Data Annotations cho các trường hợp đơn giản nhất.

**Ví dụ:**

```csharp
// Validators/AddRegionRequestValidator.cs
using FluentValidation;
using NZWalks.API.Models.DTOs;
using NZWalks.API.Repositories; // Ví dụ nếu cần inject Repository

namespace NZWalks.API.Validators
{
    public class AddRegionRequestValidator : AbstractValidator<AddRegionRequest>
    {
        // Có thể inject Repository ở đây nếu cần kiểm tra sự tồn tại của Code (ví dụ Code phải là duy nhất)
        // private readonly IRegionRepository regionRepository;
        // public AddRegionRequestValidator(IRegionRepository regionRepository)
        // {
        //     this.regionRepository = regionRepository;
        //     // ... các RuleFor ...
        //     // RuleFor(x => x.Code).MustAsync(BeUniqueCode).WithMessage("Mã khu vực đã tồn tại.");
        // }

        public AddRegionRequestValidator()
        {
            RuleFor(x => x.Code)
                .NotEmpty().WithMessage("Mã khu vực không được để trống.")
                .Length(2, 5).WithMessage("Mã khu vực phải có từ 2 đến 5 ký tự.")
                .Matches("^[A-Z]{2,5}$").WithMessage("Mã khu vực phải là chữ cái in hoa.");

            RuleFor(x => x.Name)
                .NotEmpty().WithMessage("Tên khu vực không được để trống.")
                .Length(3, 100).WithMessage("Tên khu vực phải có từ 3 đến 100 ký tự.");

            RuleFor(x => x.Area).GreaterThan(0).WithMessage("Diện tích phải lớn hơn 0.");
            RuleFor(x => x.Lat).InclusiveBetween(-90, 90).WithMessage("Vĩ độ phải nằm trong khoảng từ -90 đến 90.");
            RuleFor(x => x.Long).InclusiveBetween(-180, 180).WithMessage("Kinh độ phải nằm trong khoảng từ -180 đến 180.");
            RuleFor(x => x.Population).GreaterThanOrEqualTo(0).WithMessage("Dân số không thể là số âm.");
        }

        // private async Task<bool> BeUniqueCode(string code, CancellationToken cancellationToken)
        // {
        //     // Logic kiểm tra trong DB
        //     var existingRegion = await regionRepository.GetByCodeAsync(code);
        //     return existingRegion == null;
        // }
    }
}
```

> [!TIP]
> **Khi nào sử dụng FluentValidation?**
> *   Đối với các ứng dụng lớn, phức tạp với nhiều logic xác thực.
> *   Khi bạn cần tách biệt logic xác thực khỏi Model để code sạch và dễ bảo trì.
> *   Khi bạn cần các xác thực có điều kiện, phức tạp hoặc cần truy cập các dịch vụ khác (như Repository để kiểm tra sự tồn tại hoặc tính duy nhất của dữ liệu).
> *   Để kiểm thử xác thực dễ dàng hơn.

### 3.4. Xác thực thủ công (Manual Validation)

Phương pháp này liên quan đến việc viết các câu lệnh `if-else` và gọi `ModelState.AddModelError()` trực tiếp trong Controller hoặc trong các phương thức trợ giúp riêng tư. Mặc dù không phải là phương pháp được khuyến nghị cho các dự án thực tế do tính lặp lại và khó bảo trì, nó cực kỳ hữu ích để **hiểu rõ cơ chế hoạt động của `ModelState` và cách ASP.NET Core xử lý lỗi xác thực**.

**Ưu điểm:**

*   **Kiểm soát hoàn toàn:** Bạn có toàn quyền kiểm soát từng quy tắc xác thực và cách lỗi được thêm vào `ModelState`.
*   **Hiểu biết sâu sắc:** Giúp bạn hiểu rõ cách `ModelState` hoạt động, điều này rất quan trọng ngay cả khi bạn chuyển sang các phương pháp tự động hơn.

**Nhược điểm:**

*   **Lặp lại code (boilerplate):** Cần viết nhiều code lặp lại cho mỗi thuộc tính, mỗi Action Method.
*   **Khó bảo trì:** Khi có nhiều quy tắc hoặc thay đổi yêu cầu, việc cập nhật nhiều vị trí có thể dễ gây lỗi.
*   **Kết hợp chặt chẽ:** Logic xác thực bị gắn chặt vào Controller, khó tái sử dụng.
*   **Khó kiểm thử:** Việc kiểm thử các phương thức xác thực này thường yêu cầu khởi tạo Controller.

> [!NOTE]
> **Trong phần này, chúng ta sẽ tập trung vào phương pháp xác thực thủ công** để hiểu rõ cơ chế cốt lõi của `ModelState` và cách xử lý lỗi. Tuy nhiên, hãy ghi nhớ rằng Data Annotations và FluentValidation là các lựa chọn tốt hơn cho các dự án thực tế, giúp code sạch hơn và dễ bảo trì hơn.

## 4. Dự án thực hành: Triển khai xác thực thủ công

Chúng ta sẽ áp dụng phương pháp xác thực thủ công cho các Controller `Region`, `Walks` và `WalkDifficulty`. Điều này sẽ giúp bạn hình dung rõ ràng cách `ModelState` được điền và cách phản hồi lỗi được trả về.

### 4.1. Xác thực `RegionsController`

Controller `Region` là điểm khởi đầu lý tưởng vì nó có các thuộc tính cơ bản như `Code`, `Name`, `Area`, `Lat`, `Long`, và `Population` cần được xác thực. Chúng ta sẽ triển khai xác thực cho các phương thức `AddRegion` (POST) và `UpdateRegion` (PUT). Các phương thức `Get` và `Delete` thường không yêu cầu xác thực dữ liệu đầu vào phức tạp vì chúng chủ yếu làm việc với `ID` (thường là `Guid`), và việc kiểm tra `ID` hợp lệ thường được xử lý thông qua định tuyến hoặc tìm kiếm trong cơ sở dữ liệu.

#### 4.1.1. Xác thực `AddRegionRequest` Model (POST /Regions)

Phương thức `AddRegionAsync` nhận một đối tượng `AddRegionRequest`. Chúng ta cần đảm bảo các thuộc tính của nó hợp lệ trước khi tạo tài nguyên mới.

```csharp
// Controllers/RegionsController.cs

using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTOs;
using NZWalks.API.Repositories;
using System.Threading.Tasks;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository regionRepository;

        public RegionsController(IRegionRepository regionRepository)
        {
            this.regionRepository = regionRepository;
        }

        // ... Các phương thức GetRegionAsync, GetAllRegionsAsync, DeleteRegionAsync ...

        [HttpPost]
        public async Task<IActionResult> AddRegionAsync(AddRegionRequest addRegionRequest)
        {
            // 1. Xác thực yêu cầu đầu vào
            if (!await ValidateAddRegionRequestAsync(addRegionRequest))
            {
                // Nếu xác thực thất bại, ModelState đã chứa các lỗi.
                // Trả về BadRequest cùng với ModelState để client biết chi tiết lỗi.
                // Đối tượng ModelState tự động được serialize thành JSON,
                // cung cấp một dictionary các lỗi.
                return BadRequest(ModelState);
            }

            // --- Logic chuyển đổi DTO sang Domain Model và thêm vào DB ---
            // (Phần này đã được xây dựng trong các bài học trước)
            var regionDomain = new Models.Domain.Region
            {
                Code = addRegionRequest.Code,
                Name = addRegionRequest.Name,
                Area = addRegionRequest.Area,
                Lat = addRegionRequest.Lat,
                Long = addRegionRequest.Long,
                Population = addRegionRequest.Population
            };

            // Tạo vùng sử dụng Repository
            regionDomain = await regionRepository.AddAsync(regionDomain);

            // Chuyển đổi Domain Model thành DTO để trả về
            var regionDTO = new Models.DTOs.Region
            {
                Id = regionDomain.Id,
                Code = regionDomain.Code,
                Name = regionDomain.Name,
                Area = regionDomain.Area,
                Lat = regionDomain.Lat,
                Long = regionDomain.Long,
                Population = regionDomain.Population
            };

            return CreatedAtAction(nameof(GetRegionAsync), new { id = regionDTO.Id }, regionDTO);
        }

        #region Private methods for validation
        /// <summary>
        /// Thực hiện xác thực thủ công cho đối tượng AddRegionRequest.
        /// Các lỗi sẽ được thêm vào ModelState.
        /// </summary>
        /// <param name="addRegionRequest">Đối tượng yêu cầu thêm khu vực.</param>
        /// <returns>True nếu hợp lệ, False nếu có lỗi.</returns>
        private async Task<bool> ValidateAddRegionRequestAsync(AddRegionRequest addRegionRequest)
        {
            // Bước 1: Kiểm tra đối tượng yêu cầu có null không.
            // Đây là kiểm tra cơ bản nhất, nếu body rỗng hoặc không phân tích được.
            if (addRegionRequest == null)
            {
                ModelState.AddModelError(nameof(addRegionRequest),
                    "Dữ liệu khu vực là bắt buộc. Vui lòng cung cấp thông tin.");
                return false; // Không cần kiểm tra thêm nếu đối tượng đã null
            }

            // Bước 2: Kiểm tra từng thuộc tính
            // Ví dụ: Kiểm tra thuộc tính Code
            if (string.IsNullOrWhiteSpace(addRegionRequest.Code))
            {
                ModelState.AddModelError(nameof(addRegionRequest.Code),
                    $"{nameof(addRegionRequest.Code)} không được để trống.");
            }
            else if (addRegionRequest.Code.Length < 2 || addRegionRequest.Code.Length > 5)
            {
                ModelState.AddModelError(nameof(addRegionRequest.Code),
                    $"{nameof(addRegionRequest.Code)} phải có từ 2 đến 5 ký tự.");
            }
            // Có thể thêm kiểm tra Regex cho Code, ví dụ: chỉ cho phép chữ cái in hoa
            // else if (!System.Text.RegularExpressions.Regex.IsMatch(addRegionRequest.Code, @"^[A-Z]{2,5}$"))
            // {
            //     ModelState.AddModelError(nameof(addRegionRequest.Code),
            //         $"{nameof(addRegionRequest.Code)} phải là chữ cái in hoa.");
            // }

            // Thêm một ví dụ kiểm tra nghiệp vụ: Code phải là duy nhất.
            // Đây là lúc Repository Pattern và Dependency Injection phát huy tác dụng.
            // Chúng ta sẽ cần một phương thức trong IRegionRepository để kiểm tra sự tồn tại của Code.
            // Giả sử có: bool IsCodeExistAsync(string code);
            // var existingRegion = await regionRepository.GetByCodeAsync(addRegionRequest.Code); // Giả định phương thức này tồn tại
            // if (existingRegion != null)
            // {
            //     ModelState.AddModelError(nameof(addRegionRequest.Code),
            //         "Mã khu vực đã tồn tại. Vui lòng chọn mã khác.");
            // }

            // Kiểm tra thuộc tính Name
            if (string.IsNullOrWhiteSpace(addRegionRequest.Name))
            {
                ModelState.AddModelError(nameof(addRegionRequest.Name),
                    $"{nameof(addRegionRequest.Name)} không được để trống.");
            }
            else if (addRegionRequest.Name.Length < 3 || addRegionRequest.Name.Length > 100)
            {
                ModelState.AddModelError(nameof(addRegionRequest.Name),
                    $"{nameof(addRegionRequest.Name)} phải có từ 3 đến 100 ký tự.");
            }

            // Kiểm tra thuộc tính Area
            if (addRegionRequest.Area <= 0)
            {
                ModelState.AddModelError(nameof(addRegionRequest.Area),
                    $"{nameof(addRegionRequest.Area)} phải lớn hơn 0.");
            }

            // Kiểm tra thuộc tính Lat (Vĩ độ)
            // Vĩ độ hợp lệ nằm trong khoảng từ -90 đến 90.
            if (addRegionRequest.Lat < -90 || addRegionRequest.Lat > 90)
            {
                ModelState.AddModelError(nameof(addRegionRequest.Lat),
                    $"{nameof(addRegionRequest.Lat)} phải nằm trong khoảng từ -90 đến 90.");
            }

            // Kiểm tra thuộc tính Long (Kinh độ)
            // Kinh độ hợp lệ nằm trong khoảng từ -180 đến 180.
            if (addRegionRequest.Long < -180 || addRegionRequest.Long > 180)
            {
                ModelState.AddModelError(nameof(addRegionRequest.Long),
                    $"{nameof(addRegionRequest.Long)} phải nằm trong khoảng từ -180 đến 180.");
            }

            // Kiểm tra thuộc tính Population
            if (addRegionRequest.Population < 0)
            {
                ModelState.AddModelError(nameof(addRegionRequest.Population),
                    $"{nameof(addRegionRequest.Population)} không thể là số âm.");
            }

            // Trả về true nếu không có lỗi nào được thêm vào ModelState, false nếu có.
            return ModelState.ErrorCount == 0;
        }
        #endregion
    }
}
```

#### 4.1.2. Xác thực `UpdateRegionRequest` Model (PUT /Regions/{id})

Phương thức `UpdateRegionAsync` tương tự như `AddRegionAsync` nhưng nhận thêm `id` của khu vực cần cập nhật. Logic xác thực cho `UpdateRegionRequest` sẽ rất giống với `AddRegionRequest` vì chúng có các thuộc tính tương tự.

```csharp
// Controllers/RegionsController.cs (tiếp theo)

// ... Các phương thức AddRegionAsync ...

        [HttpPut]
        [Route("{id:Guid}")]
        public async Task<IActionResult> UpdateRegionAsync([FromRoute] Guid id, [FromBody] UpdateRegionRequest updateRegionRequest)
        {
            // 1. Xác thực yêu cầu đầu vào
            if (!await ValidateUpdateRegionRequestAsync(updateRegionRequest))
            {
                return BadRequest(ModelState);
            }

            // --- Logic chuyển đổi DTO sang Domain Model và cập nhật DB ---
            // (Phần này đã được xây dựng trong các bài học trước)
            var regionDomain = new Models.Domain.Region
            {
                Code = updateRegionRequest.Code,
                Name = updateRegionRequest.Name,
                Area = updateRegionRequest.Area,
                Lat = updateRegionRequest.Lat,
                Long = updateRegionRequest.Long,
                Population = updateRegionRequest.Population
            };

            // Cập nhật vùng sử dụng Repository
            regionDomain = await regionRepository.UpdateAsync(id, regionDomain);

            if (regionDomain == null)
            {
                // Nếu không tìm thấy Region với ID đã cho, trả về 404 Not Found
                return NotFound();
            }

            // Chuyển đổi Domain Model thành DTO để trả về
            var regionDTO = new Models.DTOs.Region
            {
                Id = regionDomain.Id,
                Code = regionDomain.Code,
                Name = regionDomain.Name,
                Area = regionDomain.Area,
                Lat = regionDomain.Lat,
                Long = regionDomain.Long,
                Population = regionDomain.Population
            };

            return Ok(regionDTO);
        }

        #region Private methods for validation (tiếp theo)
        /// <summary>
        /// Thực hiện xác thực thủ công cho đối tượng UpdateRegionRequest.
        /// Các lỗi sẽ được thêm vào ModelState.
        /// </summary>
        /// <param name="updateRegionRequest">Đối tượng yêu cầu cập nhật khu vực.</param>
        /// <returns>True nếu hợp lệ, False nếu có lỗi.</returns>
        private async Task<bool> ValidateUpdateRegionRequestAsync(UpdateRegionRequest updateRegionRequest)
        {
            if (updateRegionRequest == null)
            {
                ModelState.AddModelError(nameof(updateRegionRequest),
                    "Dữ liệu khu vực là bắt buộc. Vui lòng cung cấp thông tin.");
                return false;
            }

            // Các kiểm tra tương tự như AddRegionRequest
            if (string.IsNullOrWhiteSpace(updateRegionRequest.Code))
            {
                ModelState.AddModelError(nameof(updateRegionRequest.Code),
                    $"{nameof(updateRegionRequest.Code)} không được để trống.");
            }
            else if (updateRegionRequest.Code.Length < 2 || updateRegionRequest.Code.Length > 5)
            {
                ModelState.AddModelError(nameof(updateRegionRequest.Code),
                    $"{nameof(updateRegionRequest.Code)} phải có từ 2 đến 5 ký tự.");
            }

            if (string.IsNullOrWhiteSpace(updateRegionRequest.Name))
            {
                ModelState.AddModelError(nameof(updateRegionRequest.Name),
                    $"{nameof(updateRegionRequest.Name)} không được để trống.");
            }
            else if (updateRegionRequest.Name.Length < 3 || updateRegionRequest.Name.Length > 100)
            {
                ModelState.AddModelError(nameof(updateRegionRequest.Name),
                    $"{nameof(updateRegionRequest.Name)} phải có từ 3 đến 100 ký tự.");
            }

            if (updateRegionRequest.Area <= 0)
            {
                ModelState.AddModelError(nameof(updateRegionRequest.Area),
                    $"{nameof(updateRegionRequest.Area)} phải lớn hơn 0.");
            }

            if (updateRegionRequest.Lat < -90 || updateRegionRequest.Lat > 90)
            {
                ModelState.AddModelError(nameof(updateRegionRequest.Lat),
                    $"{nameof(updateRegionRequest.Lat)} phải nằm trong khoảng từ -90 đến 90.");
            }

            if (updateRegionRequest.Long < -180 || updateRegionRequest.Long > 180)
            {
                ModelState.AddModelError(nameof(updateRegionRequest.Long),
                    $"{nameof(updateRegionRequest.Long)} phải nằm trong khoảng từ -180 đến 180.");
            }

            if (updateRegionRequest.Population < 0)
            {
                ModelState.AddModelError(nameof(updateRegionRequest.Population),
                    $"{nameof(updateRegionRequest.Population)} không thể là số âm.");
            }

            // Lưu ý: Đối với Update, việc kiểm tra tính duy nhất của Code cần phức tạp hơn một chút.
            // Bạn cần đảm bảo Code mới không trùng với các Region khác NGOẠI TRỪ chính Region đang được cập nhật.
            // Điều này thường được xử lý tốt hơn với FluentValidation hoặc logic nghiệp vụ trong Repository.

            return ModelState.ErrorCount == 0;
        }
        #endregion
    }
}
```

### 4.2. Xác thực `WalksController`: Liên kết với Repository và DI

Controller `Walks` giới thiệu một yếu tố phức tạp hơn: các thuộc tính `RegionId` và `WalkDifficultyId`. Các ID này không chỉ phải là `Guid` hợp lệ (điều này thường được xử lý bởi Model Binding) mà còn phải **tồn tại** trong cơ sở dữ liệu để đảm bảo tính toàn vẹn tham chiếu (referential integrity). Để kiểm tra sự tồn tại này, chúng ta cần truy cập vào các Repository tương ứng (`IRegionRepository` và `IWalkDifficultyRepository`). Điều này đòi hỏi phải sử dụng Dependency Injection để đưa các Repository này vào Controller.

#### 4.2.1. Chuẩn bị Dependency Injection

Đảm bảo rằng `IRegionRepository` và `IWalkDifficultyRepository` đã được đăng ký trong `Program.cs` (hoặc `Startup.cs`) và được tiêm vào constructor của `WalksController`.

```csharp
// Program.cs (Ví dụ)
// ...
builder.Services.AddScoped<IRegionRepository, RegionRepository>();
builder.Services.AddScoped<IWalkDifficultyRepository, WalkDifficultyRepository>();
builder.Services.AddScoped<IWalkRepository, WalkRepository>();
// ...
```

#### 4.2.2. Xác thực `AddWalkRequest` Model (POST /Walks)

Phương thức `AddWalkAsync` nhận một đối tượng `AddWalkRequest`. Ngoài việc xác thực các thuộc tính cơ bản như `Name` và `Length`, chúng ta cần kiểm tra `RegionId` và `WalkDifficultyId` để đảm bảo chúng trỏ đến các thực thể hiện có trong cơ sở dữ liệu.

```csharp
// Controllers/WalksController.cs

using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTOs;
using NZWalks.API.Repositories;
using System;
using System.Threading.Tasks;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WalksController : ControllerBase
    {
        private readonly IWalkRepository walkRepository;
        private readonly IRegionRepository regionRepository; // Được inject
        private readonly IWalkDifficultyRepository walkDifficultyRepository; // Được inject

        public WalksController(IWalkRepository walkRepository,
            IRegionRepository regionRepository,
            IWalkDifficultyRepository walkDifficultyRepository)
        {
            this.walkRepository = walkRepository;
            this.regionRepository = regionRepository;
            this.walkDifficultyRepository = walkDifficultyRepository;
        }

        // ... Các phương thức GetWalkAsync, GetAllWalksAsync, DeleteWalkAsync ...

        [HttpPost]
        public async Task<IActionResult> AddWalkAsync([FromBody] AddWalkRequest addWalkRequest)
        {
            // 1. Xác thực yêu cầu đầu vào
            if (!await ValidateAddWalkRequestAsync(addWalkRequest))
            {
                return BadRequest(ModelState);
            }

            // --- Logic chuyển đổi DTO sang Domain Model và thêm vào DB ---
            var walkDomain = new Models.Domain.Walk
            {
                Name = addWalkRequest.Name,
                Length = addWalkRequest.Length,
                RegionId = addWalkRequest.RegionId,
                WalkDifficultyId = addWalkRequest.WalkDifficultyId
            };

            // Tạo Walk sử dụng Repository
            walkDomain = await walkRepository.AddAsync(walkDomain);

            // Chuyển đổi Domain Model thành DTO để trả về
            // Lưu ý: Để có thông tin Region/WalkDifficulty trong DTO trả về,
            // Repository.AddAsync cần include các navigation properties hoặc bạn cần fetch lại.
            // Ví dụ này giả định walkRepository.AddAsync trả về walkDomain đã có sẵn Region và WalkDifficulty.
            // Nếu không, bạn có thể cần một AutoMapper profile hoặc fetch lại từ DB.
            var walkDTO = new Models.DTOs.Walk
            {
                Id = walkDomain.Id,
                Name = walkDomain.Name,
                Length = walkDomain.Length,
                RegionId = walkDomain.RegionId,
                WalkDifficultyId = walkDomain.WalkDifficultyId,
                // Đảm bảo rằng walkDomain.Region và walkDomain.WalkDifficulty không null trước khi truy cập
                Region = walkDomain.Region != null ? new Models.DTOs.Region
                {
                    Id = walkDomain.Region.Id,
                    Code = walkDomain.Region.Code,
                    Name = walkDomain.Region.Name,
                    Area = walkDomain.Region.Area,
                    Lat = walkDomain.Region.Lat,
                    Long = walkDomain.Region.Long,
                    Population = walkDomain.Region.Population
                } : null,
                WalkDifficulty = walkDomain.WalkDifficulty != null ? new Models.DTOs.WalkDifficulty
                {
                    Id = walkDomain.WalkDifficulty.Id,
                    Code = walkDomain.WalkDifficulty.Code
                } : null
            };

            return CreatedAtAction(nameof(GetWalkAsync), new { id = walkDTO.Id }, walkDTO);
        }

        #region Private methods for validation
        /// <summary>
        /// Thực hiện xác thực thủ công cho đối tượng AddWalkRequest, bao gồm kiểm tra sự tồn tại của các ID liên quan.
        /// Các lỗi sẽ được thêm vào ModelState.
        /// </summary>
        /// <param name="addWalkRequest">Đối tượng yêu cầu thêm Walk.</param>
        /// <returns>True nếu hợp lệ, False nếu có lỗi.</returns>
        private async Task<bool> ValidateAddWalkRequestAsync(AddWalkRequest addWalkRequest)
        {
            if (addWalkRequest == null)
            {
                ModelState.AddModelError(nameof(addWalkRequest),
                    "Dữ liệu Walk là bắt buộc. Vui lòng cung cấp thông tin.");
                return false;
            }

            if (string.IsNullOrWhiteSpace(addWalkRequest.Name))
            {
                ModelState.AddModelError(nameof(addWalkRequest.Name),
                    $"{nameof(addWalkRequest.Name)} không được để trống.");
            }
            else if (addWalkRequest.Name.Length < 3 || addWalkRequest.Name.Length > 100)
            {
                ModelState.AddModelError(nameof(addWalkRequest.Name),
                    $"{nameof(addWalkRequest.Name)} phải có từ 3 đến 100 ký tự.");
            }

            if (addWalkRequest.Length <= 0)
            {
                ModelState.AddModelError(nameof(addWalkRequest.Length),
                    $"{nameof(addWalkRequest.Length)} phải lớn hơn 0.");
            }

            // --- Kiểm tra sự tồn tại của RegionId trong cơ sở dữ liệu ---
            // Đây là lúc Dependency Injection và Repository Pattern phát huy tác dụng.
            // Chúng ta sử dụng 'regionRepository' đã được inject vào constructor.
            var region = await regionRepository.GetAsync(addWalkRequest.RegionId);
            if (region == null)
            {
                ModelState.AddModelError(nameof(addWalkRequest.RegionId),
                    $"ID Khu vực '{addWalkRequest.RegionId}' không hợp lệ hoặc không tồn tại.");
            }

            // --- Kiểm tra sự tồn tại của WalkDifficultyId trong cơ sở dữ liệu ---
            // Tương tự, sử dụng 'walkDifficultyRepository' đã được inject.
            var walkDifficulty = await walkDifficultyRepository.GetAsync(addWalkRequest.WalkDifficultyId);
            if (walkDifficulty == null)
            {
                ModelState.AddModelError(nameof(addWalkRequest.WalkDifficultyId),
                    $"ID Độ khó đi bộ '{addWalkRequest.WalkDifficultyId}' không hợp lệ hoặc không tồn tại.");
            }

            return ModelState.ErrorCount == 0;
        }
        #endregion
    }
}
```

> [!NOTE]
> **Dependency Injection (DI) và Repository Pattern trong xác thực:**
> Trong ví dụ trên, chúng ta đã inject `IRegionRepository` và `IWalkDifficultyRepository` vào constructor của `WalksController`. Điều này cho phép phương thức xác thực (`ValidateAddWalkRequestAsync`) truy cập vào cơ sở dữ liệu thông qua các Repository để kiểm tra xem `RegionId` và `WalkDifficultyId` được cung cấp có thực sự tồn tại hay không. Đây là một ví dụ điển hình về việc kết hợp các mẫu thiết kế để xây dựng một API mạnh mẽ, linh hoạt và đảm bảo tính toàn vẹn dữ liệu. Việc xác thực này là **xác thực nghiệp vụ (business validation)**, không chỉ là xác thực định dạng.

#### 4.2.3. Xác thực `UpdateWalkRequest` Model (PUT /Walks/{id})

Tương tự như `AddWalkRequest`, phương thức `UpdateWalkAsync` cũng yêu cầu xác thực các thuộc tính cơ bản và kiểm tra sự tồn tại của `RegionId` và `WalkDifficultyId`.

```csharp
// Controllers/WalksController.cs (tiếp theo)

// ... Các phương thức AddWalkAsync ...

        [HttpPut]
        [Route("{id:Guid}")]
        public async Task<IActionResult> UpdateWalkAsync([FromRoute] Guid id, [FromBody] UpdateWalkRequest updateWalkRequest)
        {
            // 1. Xác thực yêu cầu đầu vào
            if (!await ValidateUpdateWalkRequestAsync(updateWalkRequest))
            {
                return BadRequest(ModelState);
            }

            // --- Logic chuyển đổi DTO sang Domain Model và cập nhật DB ---
            var walkDomain = new Models.Domain.Walk
            {
                Name = updateWalkRequest.Name,
                Length = updateWalkRequest.Length,
                RegionId = updateWalkRequest.RegionId,
                WalkDifficultyId = updateWalkRequest.WalkDifficultyId
            };

            // Cập nhật Walk sử dụng Repository
            walkDomain = await walkRepository.UpdateAsync(id, walkDomain);

            if (walkDomain == null)
            {
                return NotFound();
            }

            // Chuyển đổi Domain Model thành DTO để trả về
            var walkDTO = new Models.DTOs.Walk
            {
                Id = walkDomain.Id,
                Name = walkDomain.Name,
                Length = walkDomain.Length,
                RegionId = walkDomain.RegionId,
                WalkDifficultyId = walkDomain.WalkDifficultyId,
                Region = walkDomain.Region != null ? new Models.DTOs.Region
                {
                    Id = walkDomain.Region.Id,
                    Code = walkDomain.Region.Code,
                    Name = walkDomain.Region.Name,
                    Area = walkDomain.Region.Area,
                    Lat = walkDomain.Region.Lat,
                    Long = walkDomain.Region.Long,
                    Population = walkDomain.Region.Population
                } : null,
                WalkDifficulty = walkDomain.WalkDifficulty != null ? new Models.DTOs.WalkDifficulty
                {
                    Id = walkDomain.WalkDifficulty.Id,
                    Code = walkDomain.WalkDifficulty.Code
                } : null
            };

            return Ok(walkDTO);
        }

        #region Private methods for validation (tiếp theo)
        /// <summary>
        /// Thực hiện xác thực thủ công cho đối tượng UpdateWalkRequest, bao gồm kiểm tra sự tồn tại của các ID liên quan.
        /// Các lỗi sẽ được thêm vào ModelState.
        /// </summary>
        /// <param name="updateWalkRequest">Đối tượng yêu cầu cập nhật Walk.</param>
        /// <returns>True nếu hợp lệ, False nếu có lỗi.</returns>
        private async Task<bool> ValidateUpdateWalkRequestAsync(UpdateWalkRequest updateWalkRequest)
        {
            if (updateWalkRequest == null)
            {
                ModelState.AddModelError(nameof(updateWalkRequest),
                    "Dữ liệu Walk là bắt buộc. Vui lòng cung cấp thông tin.");
                return false;
            }

            if (string.IsNullOrWhiteSpace(updateWalkRequest.Name))
            {
                ModelState.AddModelError(nameof(updateWalkRequest.Name),
                    $"{nameof(updateWalkRequest.Name)} không được để trống.");
            }
            else if (updateWalkRequest.Name.Length < 3 || updateWalkRequest.Name.Length > 100)
            {
                ModelState.AddModelError(nameof(updateWalkRequest.Name),
                    $"{nameof(updateWalkRequest.Name)} phải có từ 3 đến 100 ký tự.");
            }

            if (updateWalkRequest.Length <= 0)
            {
                ModelState.AddModelError(nameof(updateWalkRequest.Length),
                    $"{nameof(updateWalkRequest.Length)} phải lớn hơn 0.");
            }

            // Kiểm tra sự tồn tại của RegionId trong cơ sở dữ liệu
            var region = await regionRepository.GetAsync(updateWalkRequest.RegionId);
            if (region == null)
            {
                ModelState.AddModelError(nameof(updateWalkRequest.RegionId),
                    $"ID Khu vực '{updateWalkRequest.RegionId}' không hợp lệ hoặc không tồn tại.");
            }

            // Kiểm tra sự tồn tại của WalkDifficultyId trong cơ sở dữ liệu
            var walkDifficulty = await walkDifficultyRepository.GetAsync(updateWalkRequest.WalkDifficultyId);
            if (walkDifficulty == null)
            {
                ModelState.AddModelError(nameof(updateWalkRequest.WalkDifficultyId),
                    $"ID Độ khó đi bộ '{updateWalkRequest.WalkDifficultyId}' không hợp lệ hoặc không tồn tại.");
            }

            return ModelState.ErrorCount == 0;
        }
        #endregion
    }
}
```

### 4.3. Xác thực `WalkDifficultiesController`

Controller `WalkDifficulty` có cấu trúc đơn giản hơn, chỉ với một thuộc tính `Code` cần xác thực. Chúng ta sẽ áp dụng cùng một kỹ thuật xác thực thủ công.

#### 4.3.1. Xác thực `AddWalkDifficultyRequest` Model (POST /WalkDifficulties)

```csharp
// Controllers/WalkDifficultiesController.cs

using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTOs;
using NZWalks.API.Repositories;
using System.Threading.Tasks;

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WalkDifficultiesController : ControllerBase
    {
        private readonly IWalkDifficultyRepository walkDifficultyRepository;

        public WalkDifficultiesController(IWalkDifficultyRepository walkDifficultyRepository)
        {
            this.walkDifficultyRepository = walkDifficultyRepository;
        }

        // ... Các phương thức GetWalkDifficultyById, GetAllWalkDifficulties, DeleteWalkDifficulty ...

        [HttpPost]
        public async Task<IActionResult> AddWalkDifficultyAsync(
            [FromBody] AddWalkDifficultyRequest addWalkDifficultyRequest)
        {
            // 1. Xác thực yêu cầu đầu vào
            if (!ValidateAddWalkDifficultyRequest(addWalkDifficultyRequest))
            {
                return BadRequest(ModelState);
            }

            // --- Logic chuyển đổi DTO sang Domain Model và thêm vào DB ---
            var walkDifficultyDomain = new Models.Domain.WalkDifficulty
            {
                Code = addWalkDifficultyRequest.Code
            };

            walkDifficultyDomain = await walkDifficultyRepository.AddAsync(walkDifficultyDomain);

            // Chuyển đổi Domain Model thành DTO để trả về
            var walkDifficultyDTO = new Models.DTOs.WalkDifficulty
            {
                Id = walkDifficultyDomain.Id,
                Code = walkDifficultyDomain.Code
            };

            return CreatedAtAction(nameof(GetWalkDifficultyById), new { id = walkDifficultyDTO.Id }, walkDifficultyDTO);
        }

        #region Private methods for validation
        /// <summary>
        /// Thực hiện xác thực thủ công cho đối tượng AddWalkDifficultyRequest.
        /// Các lỗi sẽ được thêm vào ModelState.
        /// </summary>
        /// <param name="addWalkDifficultyRequest">Đối tượng yêu cầu thêm độ khó đi bộ.</param>
        /// <returns>True nếu hợp lệ, False nếu có lỗi.</returns>
        private bool ValidateAddWalkDifficultyRequest(AddWalkDifficultyRequest addWalkDifficultyRequest)
        {
            if (addWalkDifficultyRequest == null)
            {
                ModelState.AddModelError(nameof(addWalkDifficultyRequest),
                    "Dữ liệu độ khó đi bộ là bắt buộc. Vui lòng cung cấp thông tin.");
                return false;
            }

            if (string.IsNullOrWhiteSpace(addWalkDifficultyRequest.Code))
            {
                ModelState.AddModelError(nameof(addWalkDifficultyRequest.Code),
                    $"{nameof(addWalkDifficultyRequest.Code)} không được để trống.");
            }
            else if (addWalkDifficultyRequest.Code.Length < 2 || addWalkDifficultyRequest.Code.Length > 10) // Ví dụ thêm ràng buộc độ dài
            {
                ModelState.AddModelError(nameof(addWalkDifficultyRequest.Code),
                    $"{nameof(addWalkDifficultyRequest.Code)} phải có từ 2 đến 10 ký tự.");
            }
            // Thêm các kiểm tra khác cho Code nếu cần (ví dụ: định dạng, tính duy nhất)

            return ModelState.ErrorCount == 0;
        }
        #endregion
    }
}
```

#### 4.3.2. Xác thực `UpdateWalkDifficultyRequest` Model (PUT /WalkDifficulties/{id})

```csharp
// Controllers/WalkDifficultiesController.cs (tiếp theo)

// ... Các phương thức AddWalkDifficultyAsync ...

        [HttpPut]
        [Route("{id:Guid}")]
        public async Task<IActionResult> UpdateWalkDifficultyAsync(
            [FromRoute] Guid id,
            [FromBody] UpdateWalkDifficultyRequest updateWalkDifficultyRequest)
        {
            // 1. Xác thực yêu cầu đầu vào
            if (!ValidateUpdateWalkDifficultyRequest(updateWalkDifficultyRequest))
            {
                return BadRequest(ModelState);
            }

            // --- Logic chuyển đổi DTO sang Domain Model và cập nhật DB ---
            var walkDifficultyDomain = new Models.Domain.WalkDifficulty
            {
                Code = updateWalkDifficultyRequest.Code
            };

            walkDifficultyDomain = await walkDifficultyRepository.UpdateAsync(id, walkDifficultyDomain);

            if (walkDifficultyDomain == null)
            {
                return NotFound();
            }

            // Chuyển đổi Domain Model thành DTO để trả về
            var walkDifficultyDTO = new Models.DTOs.WalkDifficulty
            {
                Id = walkDifficultyDomain.Id,
                Code = walkDifficultyDomain.Code
            };

            return Ok(walkDifficultyDTO);
        }

        #region Private methods for validation (tiếp theo)
        /// <summary>
        /// Thực hiện xác thực thủ công cho đối tượng UpdateWalkDifficultyRequest.
        /// Các lỗi sẽ được thêm vào ModelState.
        /// </summary>
        /// <param name="updateWalkDifficultyRequest">Đối tượng yêu cầu cập nhật độ khó đi bộ.</param>
        /// <returns>True nếu hợp lệ, False nếu có lỗi.</returns>
        private bool ValidateUpdateWalkDifficultyRequest(UpdateWalkDifficultyRequest updateWalkDifficultyRequest)
        {
            if (updateWalkDifficultyRequest == null)
            {
                ModelState.AddModelError(nameof(updateWalkDifficultyRequest),
                    "Dữ liệu độ khó đi bộ là bắt buộc. Vui lòng cung cấp thông tin.");
                return false;
            }

            if (string.IsNullOrWhiteSpace(updateWalkDifficultyRequest.Code))
            {
                ModelState.AddModelError(nameof(updateWalkDifficultyRequest.Code),
                    $"{nameof(updateWalkDifficultyRequest.Code)} không được để trống.");
            }
            else if (updateWalkDifficultyRequest.Code.Length < 2 || updateWalkDifficultyRequest.Code.Length > 10)
            {
                ModelState.AddModelError(nameof(updateWalkDifficultyRequest.Code),
                    $"{nameof(updateWalkDifficultyRequest.Code)} phải có từ 2 đến 10 ký tự.");
            }

            return ModelState.ErrorCount == 0;
        }
        #endregion
    }
}
```

## 5. Kiểm thử xác thực với Swagger UI

Sau khi triển khai các cơ chế xác thực, chúng ta có thể kiểm thử chúng một cách dễ dàng bằng cách sử dụng Swagger UI.

1.  **Chạy ứng dụng:** Khởi chạy ứng dụng ASP.NET Core của bạn (ví dụ: nhấn `F5` trong Visual Studio). Swagger UI sẽ tự động mở trong trình duyệt.
2.  **Chọn điểm cuối (Endpoint) cần kiểm thử:** Ví dụ, chọn điểm cuối `POST /Regions`.
3.  **Thử `Try it out`:** Nhấp vào nút "Try it out".
4.  **Sửa đổi dữ liệu để gây lỗi:**
    *   **Để trống trường `Code` hoặc `Name`:** Xóa giá trị hiện có.
    *   **Đặt `Area` là `0` hoặc một số âm:** Ví dụ: `-10`.
    *   **Đặt `Population` là một số âm:** Ví dụ: `-100`.
    *   **Đối với `Walks` (POST /Walks):** Cung cấp một `RegionId` hoặc `WalkDifficultyId` là một `Guid` không tồn tại trong cơ sở dữ liệu của bạn (ví dụ: `00000000-0000-0000-0000-000000000000`).
5.  **Nhấp `Execute`:** Gửi yêu cầu.

**Kết quả mong đợi:**
Bạn sẽ nhận được phản hồi với mã trạng thái `HTTP 400 Bad Request`. Phần nội dung (response body) của phản hồi sẽ là một đối tượng JSON chứa chi tiết các lỗi xác thực từ `ModelState`.

**Ví dụ phản hồi lỗi cho `Regions`:**

```json
{
  "Code": [
    "Mã khu vực không được để trống.",
    "Mã khu vực phải có từ 2 đến 5 ký tự."
  ],
  "Area": [
    "Diện tích phải lớn hơn 0."
  ],
  "Population": [
    "Dân số không thể là số âm."
  ],
  "Lat": [
    "Vĩ độ phải nằm trong khoảng từ -90 đến 90."
  ]
}
```

**Ví dụ phản hồi lỗi cho `Walks` (khi ID không tồn tại):**

```json
{
  "RegionId": [
    "ID Khu vực '00000000-0000-0000-0000-000000000000' không hợp lệ hoặc không tồn tại."
  ],
  "WalkDifficultyId": [
    "ID Độ khó đi bộ '00000000-0000-0000-0000-000000000000' không hợp lệ hoặc không tồn tại."
  ]
}
```

Phản hồi này cung cấp thông tin rõ ràng và có cấu trúc cho ứng dụng khách về những gì đã sai, cho phép họ sửa lỗi và gửi lại yêu cầu hợp lệ.

## 6. Tối ưu hóa quy trình với Antigravity IDE và Vibe Coding

Trong một môi trường phát triển hiện đại với các công cụ Agentic AI như Antigravity IDE, việc triển khai và quản lý xác thực có thể được tối ưu hóa đáng kể. Antigravity, với khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file, và lập kế hoạch tự động, là một trợ thủ đắc lực.

### 6.1. Vibe Coding cho Validations: Tư duy định hướng ý định

Vibe Coding khuyến khích chúng ta tập trung vào *ý định* và *kết quả mong muốn*, thay vì sa lầy vào chi tiết triển khai ban đầu. Đối với xác thực:

1.  **Xác định ý định:** "Tôi muốn `Code` của `Region` phải là chữ cái in hoa, dài từ 2 đến 5 ký tự và không được trùng lặp."
2.  **Để Antigravity trợ giúp:** Thay vì viết thủ công từng `if` cho `Code`, bạn có thể mô tả ý định này cho Antigravity. Nó có thể gợi ý ngay việc sử dụng Data Annotations hoặc FluentValidation, và thậm chí tạo ra các `RuleFor` hoặc `[Attribute]` tương ứng.
3.  **Tập trung vào Business Logic:** Khi đến phần xác thực `RegionId` và `WalkDifficultyId` trong `WalksController` (những phần cần truy cập DB), Vibe Coding giúp bạn nhận ra rằng đây là "xác thực nghiệp vụ" cần Repository. Bạn có thể yêu cầu Antigravity tạo một phương thức kiểm tra sự tồn tại trong Repository, sau đó tạo quy tắc xác thực gọi phương thức đó.
4.  **Phản hồi nhanh và lặp lại:** Khi kiểm thử với Swagger và nhận lỗi, Antigravity có thể giúp phân tích lỗi từ response JSON, đề xuất sửa chữa trong code C# của bạn, hoặc thậm chí tự động tạo một test case mới để xác nhận lỗi đã được khắc phục.

### 6.2. Antigravity's Role trong sinh mã và tinh chỉnh Validations

Antigravity IDE có thể biến quá trình phát triển xác thực từ một công việc lặp lại thành một tác vụ hiệu quả hơn nhiều:

*   **Tạo boilerplate Data Annotations:** Khi bạn khai báo một DTO mới, bạn có thể yêu cầu Antigravity tự động thêm các Data Annotations cơ bản (`[Required]`, `[StringLength]`, `[Range]`) dựa trên kiểu dữ liệu và tên thuộc tính. Ví dụ: "Thêm các Data Annotations cơ bản cho `AddRegionRequest`." Antigravity có thể phân tích cấu trúc DTO và thêm các thuộc tính phù hợp.
*   **Chuyển đổi từ thủ công sang FluentValidation:** Sau khi hiểu cơ chế xác thực thủ công, bạn có thể hướng dẫn Antigravity refactor các phương thức `Validate...RequestAsync` thủ công thành các lớp FluentValidation `AbstractValidator<T>`. "Antigravity, hãy chuyển đổi logic xác thực thủ công trong `ValidateAddRegionRequestAsync` thành một `AddRegionRequestValidator` sử dụng FluentValidation." Antigravity có thể tự động tạo file `AddRegionRequestValidator.cs`, di chuyển logic, và đăng ký validator trong `Program.cs`.
*   **Hỗ trợ xác thực nghiệp vụ:** Khi bạn cần kiểm tra tính duy nhất của `Code` hoặc sự tồn tại của `ID` trong DB, bạn có thể yêu cầu Antigravity: "Trong `AddRegionRequestValidator`, thêm một quy tắc để đảm bảo `Code` là duy nhất, sử dụng `IRegionRepository`." Antigravity có thể:
    *   Nhận ra cần inject `IRegionRepository` vào constructor của validator.
    *   Tạo một `RuleFor(x => x.Code).MustAsync(...)` và một phương thức `MustAsync` gọi `regionRepository.GetByCodeAsync`.
    *   Thậm chí đề xuất thêm phương thức `GetByCodeAsync` vào `IRegionRepository` và triển khai nó.

### 6.3. Kiểm thử và gỡ lỗi tự động với Antigravity

Antigravity không chỉ giúp viết code mà còn hỗ trợ mạnh mẽ trong việc kiểm thử và gỡ lỗi:

*   **Tạo test cases tự động:** Dựa trên các quy tắc xác thực đã định nghĩa (dù là thủ công, Data Annotations hay FluentValidation), Antigravity có thể tự động sinh ra các test cases cho API của bạn. Ví dụ: "Tạo các test cases cho `POST /Regions` để kiểm tra tất cả các quy tắc xác thực đã được triển khai." Antigravity có thể tạo ra các request body không hợp lệ (Code rỗng, Area âm, v.v.) và mong đợi phản hồi `400 Bad Request`.
*   **Mô phỏng yêu cầu và phân tích phản hồi:** Bạn có thể yêu cầu Antigravity "Gửi một yêu cầu `POST /Walks` với `RegionId` không tồn tại và phân tích phản hồi." Antigravity sẽ:
    *   Sử dụng subagent để gửi yêu cầu đến API đang chạy.
    *   Đọc phản hồi, kiểm tra mã trạng thái HTTP (mong đợi 400).
    *   Phân tích cấu trúc JSON của `ModelState` để xác nhận các thông báo lỗi chính xác được trả về.
    *   Nếu có lỗi, nó có thể đề xuất các điểm trong code cần điều chỉnh.
*   **Chạy script gỡ lỗi ngầm:** Khi bạn gặp một lỗi xác thực khó hiểu, Antigravity có thể chạy các script gỡ lỗi, thêm breakpoint tạm thời, hoặc trích xuất log từ ứng dụng đang chạy để giúp bạn pinpoint vấn đề.

Sử dụng Antigravity IDE không chỉ giúp bạn viết code xác thực nhanh hơn mà còn đảm bảo chúng mạnh mẽ hơn, ít lỗi hơn và dễ bảo trì hơn, bằng cách chuyển đổi quá trình phát triển thành một vòng lặp phản hồi nhanh chóng và định hướng ý định.

## 7. Lưu trữ thay đổi vào Git

Sau khi hoàn tất việc triển khai và kiểm thử xác thực, việc lưu trữ các thay đổi của bạn vào hệ thống kiểm soát phiên bản (Version Control System) như Git là bước cuối cùng quan trọng.

```bash
# Thêm tất cả các tệp đã thay đổi vào staging area
git add .

# Tạo commit với thông báo rõ ràng về các thay đổi
git commit -m "feat: Implement manual model validations for Regions, Walks, and WalkDifficulties controllers"

# Đẩy các thay đổi lên remote repository (ví dụ: GitHub, Azure DevOps)
git push
```

Việc lưu trữ thường xuyên giúp bạn theo dõi lịch sử phát triển, dễ dàng quay lại các phiên bản trước và cộng tác hiệu quả với nhóm.

## Tóm tắt Chương

Chương này đã trang bị cho bạn kiến thức và kỹ năng cần thiết để triển khai xác thực dữ liệu đầu vào trong các RESTful Web API bằng ASP.NET Core.

*   **Tầm quan trọng của xác thực:** Chúng ta đã hiểu rõ xác thực là tuyến phòng thủ đầu tiên chống lại dữ liệu không hợp lệ, bảo vệ tính toàn vẹn dữ liệu, ngăn chặn lỗi ứng dụng, cải thiện trải nghiệm người dùng và tăng cường bảo mật.
*   **HTTP 400 Bad Request:** Đây là mã trạng thái chuẩn để API thông báo cho ứng dụng khách về lỗi xác thực dữ liệu đầu vào.
*   **`ModelState`:** Là cơ chế cốt lõi trong ASP.NET Core để thu thập và quản lý lỗi xác thực, cho phép chúng ta thêm các lỗi tùy chỉnh bằng `ModelState.AddModelError(key, message)`.
*   **Các phương pháp xác thực:**
    *   **Data Annotations:** Đơn giản, gắn liền với Model, phù hợp cho xác thực thuộc tính cơ bản.
    *   **`IValidatableObject`:** Cho phép xác thực liên thuộc tính và logic nghiệp vụ phức tạp hơn trong Model.
    *   **FluentValidation:** Mạnh mẽ, linh hoạt, tách biệt logic xác thực khỏi Model, hỗ trợ Dependency Injection, lý tưởng cho các ứng dụng lớn và phức tạp.
    *   **Xác thực thủ công:** Được sử dụng trong chương này để hiểu sâu sắc cơ chế `ModelState`, mặc dù ít được khuyến nghị cho các dự án thực tế do tính lặp lại.
*   **Triển khai thực hành:**
    *   **`RegionsController`**: Đã triển khai xác thực thủ công cho `Code`, `Name`, `Area`, `Lat`, `Long`, `Population` trong `AddRegionRequest` và `UpdateRegionRequest`.
    *   **`WalksController`**: Đã triển khai xác thực cho `Name`, `Length` và đặc biệt là kiểm tra sự tồn tại của `RegionId` và `WalkDifficultyId` trong cơ sở dữ liệu bằng cách sử dụng **Dependency Injection** và **Repository Pattern**.
    *   **`WalkDifficultiesController`**: Đã triển khai xác thực thủ công cho thuộc tính `Code`.
*   **Kiểm thử với Swagger:** Chúng ta đã học cách sử dụng Swagger UI để kiểm thử các cơ chế xác thực và xác nhận rằng API trả về `400 Bad Request` với các thông báo lỗi chi tiết khi dữ liệu không hợp lệ.
*   **Tối ưu hóa với Antigravity IDE và Vibe Coding:** Đã khám phá cách tận dụng sức mạnh của Agentic AI để định hướng ý định, sinh mã, tinh chỉnh và tự động hóa kiểm thử xác thực, giúp quy trình phát triển nhanh hơn và hiệu quả hơn.

Việc triển khai xác thực dữ liệu đầu vào là một bước quan trọng để xây dựng các RESTful API mạnh mẽ, đáng tin cậy và thân thiện với người dùng, đồng thời chuẩn bị cho việc tích hợp các công cụ AI để nâng cao năng suất lập trình của bạn.

<!-- REVIEWED_BY_AGENT -->
