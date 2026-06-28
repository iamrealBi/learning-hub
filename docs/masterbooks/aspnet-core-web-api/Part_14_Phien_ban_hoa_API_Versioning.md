# Bài 14: Phiên bản hóa API (Versioning)

## 1. Giới thiệu tổng quan về Phiên bản hóa API

Trong kỷ nguyên phát triển phần mềm hiện đại, các ứng dụng thường hoạt động như một hệ sinh thái phức tạp, nơi các hệ thống khác nhau giao tiếp thông qua API (Application Programming Interface). Một API thành công không ngừng phát triển để đáp ứng nhu cầu kinh doanh mới, cải thiện hiệu suất, hoặc sửa lỗi. Tuy nhiên, mỗi sự thay đổi, dù nhỏ, cũng có thể phá vỡ khả năng tương thích với các ứng dụng khách (client applications) hiện có, dẫn đến sự gián đoạn dịch vụ và trải nghiệm người dùng tiêu cực.

Kỹ thuật **Phiên bản hóa API (API Versioning)** ra đời để giải quyết triệt để thách thức này. Nó cho phép các nhà phát triển giới thiệu các thay đổi một cách có kiểm soát, đảm bảo rằng các phiên bản API cũ vẫn hoạt động bình thường trong khi các phiên bản mới được triển khai và đưa vào sử dụng. Điều này là nền tảng để xây dựng các hệ thống linh hoạt, bền vững và dễ bảo trì.

Chương này sẽ cung cấp một cái nhìn sâu sắc về phiên bản hóa API trong môi trường ASP.NET Core Web API. Chúng ta sẽ không chỉ tìm hiểu "cách làm" mà còn đi sâu vào "tại sao" và "cơ chế ngầm" (under the hood) của từng phương pháp. Đặc biệt, chúng ta sẽ tập trung vào việc áp dụng các nguyên tắc thiết kế tốt như Dependency Injection (DI), Repository Pattern, Controllers và HTTP Verbs để xây dựng các RESTful Web API mạnh mẽ. Cuối cùng, chúng ta sẽ khám phá cách tích hợp phiên bản hóa với Swagger UI để đảm bảo tài liệu API luôn chính xác và dễ hiểu cho mọi phiên bản.

> [!TIP]
> **Vibe Coding và Antigravity IDE trong phiên bản hóa:**
> Trong một môi trường phát triển hiện đại với sự hỗ trợ của AI như Antigravity IDE, tư duy "Vibe Coding" trở nên cực kỳ mạnh mẽ. Khi bạn bắt đầu cảm nhận "vibe" của một thay đổi có thể là *breaking change* (ví dụ: đổi tên trường dữ liệu quan trọng, thay đổi logic nghiệp vụ cốt lõi), Antigravity sẽ không chỉ đơn thuần thực thi các lệnh. Nó sẽ phân tích ngữ cảnh, lịch sử commit, các phụ thuộc của client (nếu có thông tin) để *hiểu* "vibe" này. Từ đó, Antigravity có thể chủ động đề xuất một chiến lược phiên bản hóa, tự động tạo cấu trúc file, thêm các thuộc tính `[ApiVersion]`, `[MapToApiVersion]`, và thậm chí gợi ý cách refactor các DTO một cách thông minh, giúp bạn tập trung vào logic nghiệp vụ thay vì các tác vụ lặp lại.

## 2. Tại sao cần Phiên bản hóa API? (Mục đích và Tầm quan trọng)

Việc không có chiến lược phiên bản hóa API rõ ràng có thể dẫn đến những hậu quả nghiêm trọng:

*   **Gián đoạn hoạt động của ứng dụng khách (Client Downtime):** Imagine một ứng dụng di động đang dựa vào trường `Name` trong phản hồi API của bạn. Nếu bạn đột ngột đổi tên nó thành `CountryName` mà không có phiên bản hóa, ứng dụng di động đó sẽ ngừng hoạt động ngay lập tức, gây ra lỗi, trải nghiệm người dùng kém và thiệt hại về danh tiếng.
*   **Chi phí bảo trì cao:** Việc phải duy trì khả năng tương thích ngược bằng cách viết các mã "hack" hoặc "if/else" phức tạp để xử lý các phiên bản khác nhau trong cùng một codebase sẽ làm tăng đáng kể độ phức tạp và chi phí bảo trì.
*   **Hạn chế sự đổi mới:** Các nhà phát triển sẽ ngần ngại thực hiện các thay đổi đáng kể vì sợ phá vỡ các ứng dụng khách hiện có, làm chậm quá trình phát triển và đổi mới sản phẩm.
*   **Khó khăn trong quản lý vòng đời API:** Không có phiên bản hóa, việc theo dõi, hỗ trợ, và loại bỏ các phiên bản cũ của API sẽ trở nên vô cùng khó khăn.

Phiên bản hóa API cho phép bạn:

*   **Duy trì khả năng tương thích ngược (Backward Compatibility):** Đây là mục tiêu chính. Nó đảm bảo rằng các ứng dụng khách cũ vẫn có thể hoạt động bình thường ngay cả khi bạn phát hành các phiên bản API mới với những thay đổi lớn. Bạn có thể hỗ trợ nhiều phiên bản API song song trong một khoảng thời gian nhất định.
*   **Giới thiệu các thay đổi đột phá (Breaking Changes) một cách có kiểm soát:** Khi một thay đổi là không thể tránh khỏi và không thể tương thích với các phiên bản cũ (ví dụ: thay đổi cấu trúc dữ liệu, logic nghiệp vụ cơ bản, hoặc luồng xử lý), bạn có thể giới thiệu chúng trong một phiên bản API mới. Các ứng dụng khách có thời gian để chuyển đổi sang phiên bản mới.
*   **Cung cấp sự linh hoạt cho phát triển:** Các nhà phát triển có thể thử nghiệm và triển khai các tính năng mới, tối ưu hóa hiệu suất hoặc tái cấu trúc mã mà không lo lắng về việc phá vỡ các ứng dụng khách hiện có.
*   **Quản lý vòng đời API (API Lifecycle Management):** Phiên bản hóa cho phép bạn theo dõi và hỗ trợ nhiều phiên bản API song song. Bạn có thể chính thức "đánh dấu lỗi thời" (deprecate) một phiên bản cũ, đưa ra thông báo chuyển đổi và cuối cùng "ngừng hỗ trợ" (sunset) phiên bản đó sau một khoảng thời gian nhất định.

> [!NOTE]
> **Thay đổi đột phá (Breaking Change):** Bất kỳ thay đổi nào trong API có thể khiến các ứng dụng khách hiện có ngừng hoạt động hoặc hoạt động không như mong đợi. Ví dụ: đổi tên trường, xóa trường, thay đổi kiểu dữ liệu, thay đổi định dạng URL, thay đổi logic nghiệp vụ cơ bản, thay đổi mã lỗi HTTP.
> **Thay đổi không đột phá (Non-Breaking Change):** Những thay đổi không ảnh hưởng đến các ứng dụng khách hiện có. Ví dụ: thêm một trường mới (với giá trị mặc định hoặc tùy chọn), thêm một endpoint mới, thay đổi thứ tự các trường (nếu client không phụ thuộc vào thứ tự), thêm các giá trị mới vào một enum hiện có.

## 3. Các phương pháp Phiên bản hóa API phổ biến

Có nhiều chiến lược để triển khai phiên bản hóa API, mỗi phương pháp có ưu và nhược điểm riêng. Việc lựa chọn phương pháp phù hợp phụ thuộc vào yêu cầu dự án, mức độ dễ sử dụng và khả năng quản lý.

### 3.1. Phiên bản hóa dựa trên URL (Path Versioning)

Đây là phương pháp phổ biến và trực quan nhất. Số phiên bản được nhúng trực tiếp vào đường dẫn URL của API.

**Cơ chế:** Server đọc số phiên bản từ một đoạn của URL và định tuyến yêu cầu đến phiên bản API tương ứng.

**Ví dụ:**
*   `GET /api/v1/countries`
*   `GET /api/v2/countries`

> [!TIP]
> **Ưu điểm:**
> *   **Rõ ràng và dễ hiểu:** Số phiên bản hiển thị rõ ràng trong URL, giúp dễ dàng kiểm tra bằng trình duyệt hoặc công cụ như Postman.
> *   **Dễ dàng kiểm thử:** Chỉ cần thay đổi URL để kiểm thử các phiên bản khác nhau.
> *   **Thân thiện với cache:** Các URL khác nhau có thể được lưu cache độc lập mà không gây xung đột.
> **Nhược điểm:**
> *   **Thay đổi URL cơ bản:** Khi phiên bản thay đổi, URL thay đổi, có thể làm cho một số client thấy bất tiện.
> *   **Có thể không hoàn toàn RESTful:** Một số nhà phê bình cho rằng việc nhúng phiên bản vào URL vi phạm nguyên tắc "Resource has one URI" của REST, nhưng đây là một tranh luận phổ biến và trong thực tế, nó được chấp nhận rộng rãi.

### 3.2. Phiên bản hóa dựa trên Tham số truy vấn (Query Parameter Versioning)

Số phiên bản được truyền dưới dạng tham số trong chuỗi truy vấn của URL.

**Cơ chế:** Server kiểm tra các tham số truy vấn trong URL để tìm khóa chứa thông tin phiên bản.

**Ví dụ:**
*   `GET /api/countries?api-version=1.0`
*   `GET /api/countries?api-version=2.0`

> [!TIP]
> **Ưu điểm:**
> *   **URL tài nguyên cơ bản không thay đổi:** `GET /api/countries` vẫn là URI của tài nguyên, chỉ có tham số truy vấn thay đổi.
> *   **Dễ dàng thay đổi phiên bản:** Client chỉ cần sửa đổi tham số truy vấn.
> **Nhược điểm:**
> *   **Dễ bị bỏ qua hoặc quên:** Client có thể quên thêm tham số `api-version`, dẫn đến việc sử dụng phiên bản mặc định không mong muốn.
> *   **Khó khăn trong việc lưu trữ cache:** Nếu tham số truy vấn không được xử lý đúng cách, các hệ thống cache có thể coi `?api-version=1.0` và `?api-version=2.0` là cùng một tài nguyên, dẫn đến trả về dữ liệu sai phiên bản.
> *   **Không hoàn toàn RESTful:** Tương tự như Path Versioning, việc sử dụng tham số truy vấn để định nghĩa phiên bản của tài nguyên có thể bị coi là không hoàn toàn phù hợp với nguyên tắc REST.

### 3.3. Phiên bản hóa dựa trên Tiêu đề (Header Versioning)

Số phiên bản được truyền trong một tiêu đề HTTP tùy chỉnh.

**Cơ chế:** Server đọc một tiêu đề HTTP cụ thể (ví dụ: `X-API-Version`) từ yêu cầu và định tuyến đến phiên bản API tương ứng.

**Ví dụ:**
*   `GET /api/countries`
    *   `Headers: X-API-Version: 1.0`
*   `GET /api/countries`
    *   `Headers: X-API-Version: 2.0`

> [!TIP]
> **Ưu điểm:**
> *   **URL không thay đổi:** Tài nguyên có một URI duy nhất, phù hợp với nguyên tắc REST hơn.
> *   **Thân thiện với cache:** Vì URL không thay đổi, việc lưu trữ cache hiệu quả hơn.
> *   **Ẩn thông tin phiên bản khỏi URL:** Giúp URL gọn gàng hơn.
> **Nhược điểm:**
> *   **Khó kiểm tra trực tiếp bằng trình duyệt:** Cần sử dụng các công cụ như Postman, curl, hoặc phần mềm client để thêm tiêu đề.
> *   **Yêu cầu cấu hình thêm trong các ứng dụng khách:** Client phải biết thêm một tiêu đề tùy chỉnh để thêm vào mỗi yêu cầu.

### 3.4. Phiên bản hóa dựa trên Kiểu Media (Media Type Versioning / Accept Header)

Sử dụng tiêu đề HTTP `Accept` để chỉ định phiên bản API mong muốn, thường kết hợp với một kiểu media tùy chỉnh.

**Cơ chế:** Client gửi một tiêu đề `Accept` với một kiểu media tùy chỉnh bao gồm thông tin phiên bản (ví dụ: `application/vnd.myapi.v1+json`). Server phân tích tiêu đề này để trả về phản hồi phù hợp.

**Ví dụ:**
*   `GET /api/countries`
    *   `Headers: Accept: application/vnd.myapi.v1+json`
*   `GET /api/countries`
    *   `Headers: Accept: application/vnd.myapi.v2+json`

> [!TIP]
> **Ưu điểm:**
> *   **Theo tiêu chuẩn HTTP:** Phương pháp này được coi là RESTful nhất vì nó sử dụng cơ chế đàm phán nội dung (content negotiation) của HTTP.
> *   **Linh hoạt:** Cho phép đàm phán cả định dạng dữ liệu (JSON, XML) và phiên bản API trong cùng một tiêu đề.
> **Nhược điểm:**
> *   **Phức tạp hơn để triển khai và kiểm tra:** Client phải tạo các tiêu đề `Accept` phức tạp hơn.
> *   **Ít được sử dụng rộng rãi hơn:** Do độ phức tạp, nhiều nhà phát triển ưu tiên các phương pháp đơn giản hơn.

Trong phần này, chúng ta sẽ tập trung vào phiên bản hóa dựa trên URL (Path Versioning) và có thể kết hợp với Tham số truy vấn (Query Parameter Versioning) khi sử dụng gói NuGet, vì đây là những phương pháp phổ biến và được khuyến nghị trong nhiều dự án ASP.NET Core.

## 4. Chuẩn bị môi trường và Cấu trúc dự án cơ bản

Để minh họa các khái niệm về phiên bản hóa, chúng ta sẽ xây dựng một Web API đơn giản trong ASP.NET Core. Giả sử chúng ta đã có một dự án ASP.NET Core Web API cơ bản với một `CountryController` cung cấp danh sách các quốc gia.

**Cấu trúc dự án ban đầu:**

```
YourSolutionName/
├── YourProjectName/
│   ├── Controllers/
│   │   └── CountryController.cs
│   ├── Models/
│   │   ├── Country.cs
│   │   └── CountryDto.cs
│   ├── Data/
│   │   └── CountryData.cs
│   ├── Program.cs
│   └── appsettings.json
└── YourSolutionName.sln
```

### 4.1. Các thành phần chính

1.  **`Country.cs` (Domain Model):** Đại diện cho một quốc gia trong miền nghiệp vụ của ứng dụng. Đây là mô hình lõi và thường ít khi thay đổi đột phá.

    ```csharp
    // Models/Country.cs
    namespace YourProjectName.Models
    {
        public class Country
        {
            public int Id { get; set; }
            public string Name { get; set; } = string.Empty;
        }
    }
    ```

2.  **`CountryDto.cs` (Data Transfer Object - DTO):** Đối tượng được sử dụng để truyền dữ liệu giữa API và ứng dụng khách. Việc sử dụng DTO là một thực hành tốt để tách biệt mô hình miền khỏi định dạng phản hồi API, giúp kiểm soát chính xác dữ liệu nào được hiển thị ra bên ngoài.

    ```csharp
    // Models/CountryDto.cs
    namespace YourProjectName.Models
    {
        // Đây là DTO ban đầu, chỉ có ID và Name
        public class CountryDto
        {
            public int Id { get; set; }
            public string Name { get; set; } = string.Empty;
        }
    }
    ```

3.  **`CountryData.cs` (Static Data - Giả lập Repository):** Để đơn giản hóa, chúng ta sẽ sử dụng dữ liệu tĩnh thay vì kết nối cơ sở dữ liệu. Lớp này đóng vai trò như một kho lưu trữ (repository) đơn giản để cung cấp dữ liệu quốc gia.

    > [!NOTE]
    > **Vai trò của Repository Pattern:** Trong một ứng dụng thực tế, `CountryData` sẽ là một lớp triển khai `ICountryRepository` (interface) và tương tác với Entity Framework Core để truy vấn dữ liệu từ cơ sở dữ liệu. Việc sử dụng Repository Pattern giúp trừu tượng hóa lớp truy cập dữ liệu, làm cho Controller không cần quan tâm đến chi tiết lưu trữ dữ liệu, dễ dàng thay đổi nguồn dữ liệu và kiểm thử (unit testing) hơn. Ở đây, chúng ta chỉ giả lập một kho dữ liệu tĩnh để tập trung vào vấn đề phiên bản hóa.

    ```csharp
    // Data/CountryData.cs
    using YourProjectName.Models;
    using System.Collections.Generic;
    using System.Linq;

    namespace YourProjectName.Data
    {
        public static class CountryData // Trong thực tế sẽ là một service được inject
        {
            private static List<Country> _countries = new List<Country>
            {
                new Country { Id = 1, Name = "United States" },
                new Country { Id = 2, Name = "Canada" },
                new Country { Id = 3, Name = "Mexico" },
                new Country { Id = 4, Name = "Brazil" },
                new Country { Id = 5, Name = "Argentina" },
                new Country { Id = 6, Name = "United Kingdom" },
                new Country { Id = 7, Name = "France" },
                new Country { Id = 8, Name = "Germany" },
                new Country { Id = 9, Name = "Japan" },
                new Country { Id = 10, Name = "Australia" }
            };

            public static List<Country> GetCountries()
            {
                return _countries;
            }

            public static Country? GetCountryById(int id)
            {
                return _countries.FirstOrDefault(c => c.Id == id);
            }
        }
    }
    ```

4.  **`CountryController.cs` (Controller - Phiên bản ban đầu):**

    > [!NOTE]
    > **Controllers và HTTP Verbs:** `CountryController` là một ví dụ điển hình về một API Controller trong ASP.NET Core. Nó sử dụng thuộc tính `[Route("api/[controller]")]` để định tuyến yêu cầu đến `/api/country` và `[ApiController]` để kích hoạt các tính năng tiện ích của API như tự động validate model. Phương thức `GetCountries()` sử dụng `[HttpGet]` để chỉ định nó phản hồi các yêu cầu GET, và `[ProducesResponseType(200, Type = typeof(IEnumerable<CountryDto>))]` để tài liệu hóa loại phản hồi thành công.

    ```csharp
    // Controllers/CountryController.cs
    using Microsoft.AspNetCore.Mvc;
    using YourProjectName.Data;
    using YourProjectName.Models;
    using System.Collections.Generic;
    using System.Linq;

    namespace YourProjectName.Controllers
    {
        [Route("api/[controller]")]
        [ApiController]
        public class CountryController : ControllerBase
        {
            [HttpGet]
            [ProducesResponseType(200, Type = typeof(IEnumerable<CountryDto>))]
            public IActionResult GetCountries()
            {
                var countries = CountryData.GetCountries();
                var countryDtos = countries.Select(c => new CountryDto { Id = c.Id, Name = c.Name }).ToList();
                return Ok(countryDtos);
            }
        }
    }
    ```

Khi chạy ứng dụng này, bạn sẽ có một API tại `/api/country` trả về danh sách các quốc gia với `Id` và `Name`.

## 5. Triển khai phiên bản hóa thủ công bằng cấu trúc thư mục

Phương pháp này thể hiện rõ ràng ý tưởng về việc duy trì nhiều phiên bản API bằng cách tổ chức mã nguồn. Khi có một thay đổi đột phá, chúng ta sẽ tạo một phiên bản mới trong một thư mục riêng biệt. Mặc dù trực quan, phương pháp này thường không được khuyến nghị cho các dự án lớn vì nó dẫn đến việc sao chép mã và khó quản lý.

Giả sử chúng ta muốn thay đổi thuộc tính `Name` thành `CountryName` trong phản hồi API. Đây là một **thay đổi đột phá**. Để hỗ trợ cả khách hàng cũ (mong đợi `Name`) và khách hàng mới (mong đợi `CountryName`), chúng ta sẽ tạo phiên bản `V2`.

### 5.1. Các bước thực hiện

1.  **Tạo các thư mục phiên bản:**
    Trong thư mục gốc của dự án, tạo hai thư mục mới: `V1` và `V2`. Bên trong mỗi thư mục này, tạo một thư mục `Controllers`.

    ```
    YourProjectName/
    ├── V1/
    │   └── Controllers/
    ├── V2/
    │   └── Controllers/
    ├── Controllers/ (Sẽ xóa hoặc đổi tên sau)
    ├── Models/
    ├── Data/
    └── ...
    ```

2.  **Di chuyển và sao chép Controller:**
    *   Di chuyển `CountryController.cs` hiện có vào thư mục `V1/Controllers`.
    *   Sao chép `CountryController.cs` từ `V1/Controllers` sang `V2/Controllers`.
    *   (Tùy chọn) Xóa `CountryController.cs` khỏi thư mục `Controllers` gốc để tránh xung đột.

    Sau bước này, bạn sẽ có:

    ```
    YourProjectName/
    ├── V1/
    │   └── Controllers/
    │       └── CountryController.cs
    ├── V2/
    │   └── Controllers/
    │       └── CountryController.cs
    ├── Models/
    │   ├── Country.cs
    │   ├── CountryDto.cs (Sẽ được chỉnh sửa/tạo mới)
    ├── Data/
    └── ...
    ```

3.  **Điều chỉnh DTO cho từng phiên bản:**
    Để phản ánh sự thay đổi đột phá, chúng ta cần hai DTO khác nhau. Đổi tên `CountryDto.cs` thành `CountryDtoV1.cs` và tạo `CountryDtoV2.cs`.

    ```csharp
    // Models/CountryDtoV1.cs (Đổi tên từ CountryDto.cs)
    namespace YourProjectName.Models
    {
        public class CountryDtoV1
        {
            public int Id { get; set; }
            public string Name { get; set; } = string.Empty;
        }
    }

    // Models/CountryDtoV2.cs (Tạo mới)
    namespace YourProjectName.Models
    {
        public class CountryDtoV2
        {
            public int Id { get; set; }
            public string CountryName { get; set; } = string.Empty; // Thay đổi đột phá
        }
    }
    ```

4.  **Cập nhật `CountryController` cho V1:**
    *   Mở `YourProjectName/V1/Controllers/CountryController.cs`.
    *   Cập nhật `namespace` để phản ánh vị trí mới (ví dụ: `YourProjectName.V1.Controllers`).
    *   Cập nhật `Route` attribute để chỉ định phiên bản (`[Route("api/v1/[controller]")]`).
    *   Sử dụng `CountryDtoV1`.

    ```csharp
    // V1/Controllers/CountryController.cs
    using Microsoft.AspNetCore.Mvc;
    using YourProjectName.Data;
    using YourProjectName.Models;
    using System.Collections.Generic;
    using System.Linq;

    namespace YourProjectName.V1.Controllers // Namespace thay đổi
    {
        [Route("api/v1/[controller]")] // Route thay đổi để chỉ định phiên bản
        [ApiController]
        public class CountryController : ControllerBase
        {
            [HttpGet]
            [ProducesResponseType(200, Type = typeof(IEnumerable<CountryDtoV1>))]
            public IActionResult GetCountries()
            {
                var countries = CountryData.GetCountries();
                // Ánh xạ sang CountryDtoV1
                var countryDtos = countries.Select(c => new CountryDtoV1 { Id = c.Id, Name = c.Name }).ToList();
                return Ok(countryDtos);
            }
        }
    }
    ```

5.  **Cập nhật `CountryController` cho V2:**
    *   Mở `YourProjectName/V2/Controllers/CountryController.cs`.
    *   Cập nhật `namespace` (ví dụ: `YourProjectName.V2.Controllers`).
    *   Cập nhật `Route` attribute (`[Route("api/v2/[controller]")]`).
    *   Sử dụng `CountryDtoV2` và ánh xạ thuộc tính `Name` của model thành `CountryName` của DTO.

    ```csharp
    // V2/Controllers/CountryController.cs
    using Microsoft.AspNetCore.Mvc;
    using YourProjectName.Data;
    using YourProjectName.Models;
    using System.Collections.Generic;
    using System.Linq;

    namespace YourProjectName.V2.Controllers // Namespace thay đổi
    {
        [Route("api/v2/[controller]")] // Route thay đổi để chỉ định phiên bản
        [ApiController]
        public class CountryController : ControllerBase
        {
            [HttpGet]
            [ProducesResponseType(200, Type = typeof(IEnumerable<CountryDtoV2>))]
            public IActionResult GetCountries()
            {
                var countries = CountryData.GetCountries();
                // Ánh xạ sang CountryDtoV2 với CountryName
                var countryDtos = countries.Select(c => new CountryDtoV2 { Id = c.Id, CountryName = c.Name }).ToList();
                return Ok(countryDtos);
            }
        }
    }
    ```

Bây giờ, khi bạn chạy ứng dụng, bạn sẽ có hai endpoint hoạt động độc lập:
*   `GET /api/v1/country`: Trả về danh sách quốc gia với `Id` và `Name`.
*   `GET /api/v2/country`: Trả về danh sách quốc gia với `Id` và `CountryName`.

> [!NOTE]
> Phương pháp thủ công này rất trực quan cho các dự án nhỏ hoặc khi chỉ có một vài thay đổi. Tuy nhiên, nó nhanh chóng trở nên cồng kềnh khi số lượng phiên bản và bộ điều khiển tăng lên. Nó yêu cầu sao chép và duy trì nhiều file, dễ dẫn đến lỗi, vi phạm nguyên tắc DRY (Don't Repeat Yourself) và khó quản lý.

> [!TIP]
> **Antigravity IDE và phiên bản hóa thủ công:**
> Nếu bạn quyết định theo đuổi "vibe" của phiên bản hóa thủ công, Antigravity IDE có thể giúp bạn tự động hóa nhiều tác vụ lặp lại. Với khả năng tự chạy script ngầm và đọc/ghi file, Antigravity có thể:
> 1.  **Tạo cấu trúc thư mục:** Tự động tạo các thư mục `V1/Controllers`, `V2/Controllers`.
> 2.  **Sao chép và di chuyển file:** Di chuyển controller gốc vào `V1` và sao chép sang `V2`.
> 3.  **Refactor tự động:** Dựa trên hướng dẫn của bạn (ví dụ: "chuyển `Name` thành `CountryName` trong V2"), Antigravity có thể tự động:
>     *   Đổi tên `CountryDto.cs` thành `CountryDtoV1.cs`.
>     *   Tạo `CountryDtoV2.cs` với thuộc tính mới.
>     *   Cập nhật `namespace` và `[Route]` trong cả hai controller.
>     *   Chỉnh sửa logic ánh xạ trong `V2/Controllers/CountryController.cs` để sử dụng `CountryDtoV2`.
> Điều này minh họa cách Antigravity, với khả năng lập kế hoạch và thực thi chi tiết, có thể biến một quy trình thủ công tẻ nhạt thành một tác vụ tự động hiệu quả.

## 6. Phương pháp khuyến nghị: Phiên bản hóa bằng gói NuGet

Để đơn giản hóa việc quản lý phiên bản, ASP.NET Core cung cấp gói NuGet `Microsoft.AspNetCore.Mvc.Versioning` mạnh mẽ. Phương pháp này được ưa chuộng hơn vì nó giảm thiểu việc sao chép mã, tập trung cấu hình phiên bản, và cho phép quản lý nhiều phiên bản trong cùng một controller file.

Chúng ta sẽ hoàn nguyên các thay đổi thủ công và quay lại cấu trúc dự án ban đầu với một `CountryController` duy nhất.

### 6.1. Cài đặt gói NuGet

Sử dụng gói `Microsoft.AspNetCore.Mvc.Versioning` để thêm khả năng phiên bản hóa vào các controller và action của bạn.

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
```

Hoặc thông qua NuGet Package Manager trong Visual Studio.

> [!TIP]
> **Antigravity IDE và cài đặt gói:**
> Với Antigravity IDE, bạn có thể chỉ cần đưa ra một "vibe" như "Tôi muốn thêm phiên bản hóa API vào dự án này." Antigravity sẽ tự động nhận diện gói NuGet cần thiết, thực thi lệnh `dotnet add package`, và bắt đầu quá trình cấu hình tiếp theo.

### 6.2. Cấu hình trong `Program.cs`

Sau khi cài đặt gói, bạn cần cấu hình dịch vụ phiên bản hóa trong file `Program.cs`.

```csharp
// Program.cs
using Microsoft.AspNetCore.Mvc; // Cần thiết cho ApiVersion
using Microsoft.AspNetCore.Mvc.Versioning; // Cần thiết cho ApiVersionReader

var builder = WebApplication.CreateBuilder(args);

// Thêm dịch vụ Controllers
builder.Services.AddControllers();

// Cấu hình dịch vụ phiên bản hóa API
builder.Services.AddApiVersioning(options =>
{
    // 1. DefaultApiVersion: Phiên bản mặc định được sử dụng khi không có phiên bản nào được chỉ định rõ ràng
    options.DefaultApiVersion = new ApiVersion(1, 0);

    // 2. AssumeDefaultVersionWhenUnspecified: Nếu không có phiên bản nào được chỉ định trong yêu cầu,
    //    hệ thống sẽ mặc định sử dụng DefaultApiVersion. Nếu là false, yêu cầu không có phiên bản sẽ gây lỗi.
    options.AssumeDefaultVersionWhenUnspecified = true;

    // 3. ReportApiVersions: Báo cáo các phiên bản API được hỗ trợ bởi controller trong tiêu đề phản hồi HTTP.
    //    Ví dụ: api-supported-versions: 1.0, 2.0
    //    api-deprecated-versions: (nếu có)
    options.ReportApiVersions = true;

    // 4. ApiVersionReader: Cấu hình cách API Versioning đọc thông tin phiên bản từ yêu cầu HTTP.
    //    ApiVersionReader.Combine cho phép bạn chỉ định nhiều phương thức đọc.
    //    Thứ tự trong Combine quan trọng: nó sẽ thử từng phương thức theo thứ tự cho đến khi tìm thấy phiên bản.
    options.ApiVersionReader = ApiVersionReader.Combine(
        // Đọc từ query parameter: ?api-version=1.0
        new QueryStringApiVersionReader("api-version"),
        // Đọc từ tiêu đề HTTP tùy chỉnh: X-Api-Version: 1.0
        new HeaderApiVersionReader("X-Api-Version"),
        // Đọc từ URL segment: /v1/ (ví dụ: api/v{version:apiVersion}/[controller])
        new UrlSegmentApiVersionReader()
    );
});

// Cấu hình Swagger/OpenAPI (sẽ được cấu hình chi tiết hơn sau)
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(); // Sẽ cần chỉnh sửa để hiển thị đúng các phiên bản
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers(); // Kích hoạt định tuyến cho các controller

app.Run();
```

> [!NOTE]
> **Cơ chế `ApiVersionReader`:**
> `Microsoft.AspNetCore.Mvc.Versioning` sử dụng `IApiVersionReader` để trích xuất thông tin phiên bản từ `HttpRequest`. Khi bạn sử dụng `ApiVersionReader.Combine`, hệ thống sẽ duyệt qua danh sách các reader theo thứ tự. Reader đầu tiên tìm thấy thông tin phiên bản hợp lệ sẽ được sử dụng. Nếu không có reader nào tìm thấy phiên bản hoặc có xung đột (ví dụ: phiên bản trong query string khác với phiên bản trong header), hệ thống sẽ trả về lỗi `400 Bad Request` hoặc `404 Not Found` tùy thuộc vào cấu hình.

### 6.3. Áp dụng Versioning lên Controller và Action Methods

Bây giờ, chúng ta sẽ điều chỉnh `CountryController` để hỗ trợ nhiều phiên bản trong cùng một file, tận dụng các thuộc tính từ gói NuGet.

**Cập nhật DTOs:** Chúng ta vẫn cần `CountryDtoV1` và `CountryDtoV2` như đã định nghĩa ở phần thủ công. Đảm bảo chúng nằm trong thư mục `Models`.

```csharp
// Models/CountryDtoV1.cs
namespace YourProjectName.Models
{
    public class CountryDtoV1
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
    }
}

// Models/CountryDtoV2.cs
namespace YourProjectName.Models
{
    public class CountryDtoV2
    {
        public int Id { get; set; }
        public string CountryName { get; set; } = string.Empty;
    }
}
```

**Cập nhật `CountryController.cs`:**

```csharp
// Controllers/CountryController.cs
using Microsoft.AspNetCore.Mvc;
using YourProjectName.Data;
using YourProjectName.Models;
using System.Collections.Generic;
using System.Linq;

namespace YourProjectName.Controllers
{
    // [ApiVersion("X.Y")]: Xác định các phiên bản mà controller này hỗ trợ.
    // Một controller có thể hỗ trợ nhiều phiên bản.
    [ApiVersion("1.0")]
    [ApiVersion("2.0")]

    // [Route("api/v{version:apiVersion}/[controller]")]:
    // Định tuyến theo URL segment cho phiên bản.
    // {version:apiVersion} là một ràng buộc tuyến đường đặc biệt do gói Versioning cung cấp.
    // Nó cho phép hệ thống đọc phiên bản từ URL (ví dụ: /api/v1/country).
    // Nếu bạn muốn hỗ trợ query parameter hoặc header, bạn có thể bỏ phần v{version:apiVersion}
    // khỏi Route và dựa vào ApiVersionReader đã cấu hình trong Program.cs.
    // Tuy nhiên, việc có phiên bản trong URL là rõ ràng và được khuyến nghị.
    [Route("api/v{version:apiVersion}/[controller]")]
    [ApiController]
    public class CountryController : ControllerBase
    {
        // Phương thức này ánh xạ tới API phiên bản 1.0
        [HttpGet]
        // [MapToApiVersion("X.Y")]: Chỉ định phiên bản cụ thể cho action method này.
        // Một action method chỉ có thể thuộc về một phiên bản duy nhất.
        [MapToApiVersion("1.0")]
        [ProducesResponseType(200, Type = typeof(IEnumerable<CountryDtoV1>))]
        public IActionResult GetCountriesV1()
        {
            var countries = CountryData.GetCountries();
            var countryDtos = countries.Select(c => new CountryDtoV1 { Id = c.Id, Name = c.Name }).ToList();
            return Ok(countryDtos);
        }

        // Phương thức này ánh xạ tới API phiên bản 2.0
        [HttpGet]
        [MapToApiVersion("2.0")]
        [ProducesResponseType(200, Type = typeof(IEnumerable<CountryDtoV2>))]
        public IActionResult GetCountriesV2()
        {
            var countries = CountryData.GetCountries();
            var countryDtos = countries.Select(c => new CountryDtoV2 { Id = c.Id, CountryName = c.Name }).ToList();
            return Ok(countryDtos);
        }
    }
}
```

> [!TIP]
> **Cơ chế định tuyến với `v{version:apiVersion}`:**
> Khi một yêu cầu đến (ví dụ: `/api/v1/country`), ràng buộc tuyến đường `v{version:apiVersion}` sẽ được kích hoạt. `UrlSegmentApiVersionReader` (mà chúng ta đã cấu hình trong `Program.cs`) sẽ trích xuất `1.0` từ URL. Sau đó, hệ thống định tuyến của ASP.NET Core sẽ tìm kiếm một action method nào có thuộc tính `[MapToApiVersion("1.0")]` và khớp với HTTP Verb (GET) và tên controller. Điều này cho phép nhiều action methods với cùng tên và HTTP Verb tồn tại trong cùng một controller, miễn là chúng được ánh xạ tới các phiên bản API khác nhau.

> [!NOTE]
> **Xung đột định tuyến:**
> Nếu bạn có hai action methods có cùng HTTP Verb và cùng `[MapToApiVersion]` trong cùng một controller, hoặc nếu một action method không có `[MapToApiVersion]` nhưng controller có `[ApiVersion]` chung, hệ thống có thể gặp xung đột hoặc hành vi không mong muốn. Luôn đảm bảo mỗi action method có một ánh xạ phiên bản rõ ràng và duy nhất trong ngữ cảnh của controller.

> [!TIP]
> **Antigravity IDE và Controller Versioning:**
> Với "vibe" là "phiên bản hóa controller này sang V2 với thay đổi X, Y", Antigravity có thể:
> 1.  **Phân tích code hiện có:** Xác định các action methods và DTOs.
> 2.  **Đề xuất cấu trúc DTO mới:** Dựa trên thay đổi bạn muốn (ví dụ: `Name` thành `CountryName`), nó sẽ tạo `CountryDtoV2.cs`.
> 3.  **Refactor controller:**
>     *   Thêm `[ApiVersion("1.0")]` và `[ApiVersion("2.0")]` vào controller.
>     *   Thêm `[Route("api/v{version:apiVersion}/[controller]")]`.
>     *   Đổi tên `GetCountries()` thành `GetCountriesV1()` và thêm `[MapToApiVersion("1.0")]`.
>     *   Tạo một phương thức `GetCountriesV2()` mới, thêm `[MapToApiVersion("2.0")]`, và cập nhật logic ánh xạ để sử dụng `CountryDtoV2`.
> Antigravity không chỉ thực hiện các thay đổi này mà còn *giải thích* lý do và cách thức, giúp học viên hiểu sâu hơn về quy trình.

### 6.4. Kiểm tra bằng Postman / cURL

Với cấu hình trên, bạn có thể kiểm tra API bằng Postman, cURL hoặc trình duyệt:

*   **Truy cập phiên bản 1.0 (Path Versioning):**
    *   `GET http://localhost:<port>/api/v1/country`
    *   Sẽ trả về: `[{"id":1,"name":"United States"}, ...]`
*   **Truy cập phiên bản 2.0 (Path Versioning):**
    *   `GET http://localhost:<port>/api/v2/country`
    *   Sẽ trả về: `[{"id":1,"countryName":"United States"}, ...]`
*   **Truy cập phiên bản 1.0 (Query Parameter Versioning):**
    *   Nếu bạn thay đổi `[Route]` trên controller thành `[Route("api/[controller]")]` (bỏ `v{version:apiVersion}`) và chỉ dùng query string:
    *   `GET http://localhost:<port>/api/country?api-version=1.0`
    *   Sẽ trả về: `[{"id":1,"name":"United States"}, ...]`
*   **Truy cập phiên bản 2.0 (Query Parameter Versioning):**
    *   `GET http://localhost:<port>/api/country?api-version=2.0`
    *   Sẽ trả về: `[{"id":1,"countryName":"United States"}, ...]`
*   **Truy cập phiên bản 1.0 (Header Versioning):**
    *   Nếu bạn thay đổi `[Route]` trên controller thành `[Route("api/[controller]")]` và gửi tiêu đề:
    *   `GET http://localhost:<port>/api/country`
    *   `Headers: X-Api-Version: 1.0`
    *   Sẽ trả về: `[{"id":1,"name":"United States"}, ...]`

> [!NOTE]
> Khi sử dụng `[Route("api/v{version:apiVersion}/[controller]")]`, việc không chỉ định `v{version:apiVersion}` trong URL sẽ dẫn đến lỗi `404 Not Found`, vì tuyến đường yêu cầu phải có segment phiên bản. Điều này đảm bảo rằng các ứng dụng khách luôn phải rõ ràng về phiên bản API mà họ đang gọi. Nếu `options.AssumeDefaultVersionWhenUnspecified = true` được bật và bạn không sử dụng `v{version:apiVersion}` trong route, yêu cầu không có phiên bản sẽ tự động chuyển đến phiên bản mặc định.

## 7. Tích hợp và Khắc phục hiển thị phiên bản hóa trong Swagger UI

Mặc dù API của chúng ta đã được phiên bản hóa thành công, Swagger UI mặc định có thể không hiển thị tất cả các phiên bản API một cách chính xác hoặc có thể gặp lỗi. Điều này là do Swagger cần thêm thông tin để hiểu cách các phiên bản API được cấu trúc và cách nhóm chúng.

### 7.1. Cài đặt gói NuGet cho API Explorer

Để Swagger có thể khám phá và hiển thị đúng các phiên bản API, chúng ta cần gói `Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer`. Gói này cung cấp các dịch vụ cần thiết để trích xuất siêu dữ liệu về các API đã được phiên bản hóa.

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer
```

Hoặc thông qua NuGet Package Manager trong Visual Studio.

### 7.2. Cấu hình `VersionedApiExplorer` và `SwaggerGen` trong `Program.cs`

Sau khi cài đặt gói, hãy thêm dịch vụ `VersionedApiExplorer` và cấu hình `SwaggerGen` để làm việc với nó trong `Program.cs`.

```csharp
// Program.cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Versioning;
using Microsoft.AspNetCore.Mvc.ApiExplorer; // Cần thiết cho IApiVersionDescriptionProvider, AddVersionedApiExplorer
using Microsoft.Extensions.Options; // Cần thiết cho IConfigureOptions
using Swashbuckle.AspNetCore.SwaggerGen; // Cần thiết cho SwaggerGenOptions
using YourProjectName.Configuration; // Namespace cho ConfigureSwaggerOptions của chúng ta

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Cấu hình dịch vụ phiên bản hóa API (như đã làm ở phần 6.2)
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("X-Api-Version"),
        new UrlSegmentApiVersionReader()
    );
});

// Thêm dịch vụ khám phá API được phiên bản hóa.
// Gói Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer cung cấp dịch vụ này.
builder.Services.AddVersionedApiExplorer(options =>
{
    // Định dạng tên nhóm cho các phiên bản API trong Swagger UI.
    // 'v' - tiền tố, VVV - phiên bản chính (major), phụ (minor) và patch (nếu có).
    // Ví dụ: 'v'1.0, 'v'2.0
    options.GroupNameFormat = "'v'VVV";

    // Thay thế tham số phiên bản trong URL của Swagger UI.
    // Điều này đảm bảo rằng các URL được tạo trong Swagger UI sẽ có định dạng đúng
    // (ví dụ: /api/v1/country thay vì /api/{version}/country)
    options.SubstituteApiVersionInUrl = true;
});

builder.Services.AddEndpointsApiExplorer();

// Cấu hình SwaggerGen để sử dụng các tùy chọn phiên bản hóa.
// Chúng ta sử dụng Dependency Injection để cấu hình SwaggerGenOptions
// bằng cách đăng ký một IConfigureOptions<SwaggerGenOptions> tùy chỉnh.
// Lớp ConfigureSwaggerOptions sẽ được tạo ở bước tiếp theo.
builder.Services.AddTransient<IConfigureOptions<SwaggerGenOptions>, ConfigureSwaggerOptions>();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    // Lấy IApiVersionDescriptionProvider từ ServiceProvider.
    // Provider này chứa thông tin về tất cả các phiên bản API đã được khám phá.
    var apiVersionDescriptionProvider = app.Services.GetRequiredService<IApiVersionDescriptionProvider>();

    app.UseSwaggerUI(options =>
    {
        // Tạo một endpoint Swagger cho mỗi phiên bản API đã khám phá.
        // Điều này sẽ tạo ra các mục trong dropdown selector của Swagger UI.
        foreach (var description in apiVersionDescriptionProvider.ApiVersionDescriptions)
        {
            options.SwaggerEndpoint($"/swagger/{description.GroupName}/swagger.json",
                                    description.GroupName.ToUpperInvariant());
        }
    });
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

> [!NOTE]
> **Cơ chế `IApiVersionDescriptionProvider`:**
> `IApiVersionDescriptionProvider` là một dịch vụ được cung cấp bởi gói `Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer`. Nó có nhiệm vụ thu thập tất cả thông tin về các phiên bản API đã được định nghĩa trong ứng dụng của bạn (thông qua các thuộc tính `[ApiVersion]`). Khi bạn gọi `AddVersionedApiExplorer()`, dịch vụ này sẽ được đăng ký và điền dữ liệu. Sau đó, nó có thể được inject vào các dịch vụ khác (như `ConfigureSwaggerOptions`) để tạo tài liệu Swagger động cho từng phiên bản.

> [!NOTE]
> **Dependency Injection (DI) và `IConfigureOptions<SwaggerGenOptions>`:**
> Trong ASP.NET Core, `IConfigureOptions<TOptions>` là một mẫu thiết kế mạnh mẽ để cấu hình các tùy chọn (options) một cách có tổ chức và linh hoạt thông qua DI. Thay vì cấu hình trực tiếp `SwaggerGenOptions` trong `Program.cs`, chúng ta tạo một lớp `ConfigureSwaggerOptions` riêng biệt. Lớp này sẽ được DI container khởi tạo và gọi phương thức `Configure` của nó để áp dụng các cài đặt cho `SwaggerGenOptions`. Điều này giúp tách biệt các mối quan tâm và làm cho `Program.cs` gọn gàng hơn.

### 7.3. Tạo lớp `ConfigureSwaggerOptions`

Để quản lý cấu hình Swagger một cách sạch sẽ và tận dụng Dependency Injection, chúng ta sẽ tạo một lớp riêng biệt `ConfigureSwaggerOptions.cs`. Lớp này sẽ kế thừa từ `IConfigureOptions<SwaggerGenOptions>` và sử dụng `IApiVersionDescriptionProvider` để tự động tạo tài liệu cho từng phiên bản API.

Tạo một file mới `ConfigureSwaggerOptions.cs` trong thư mục `Configuration` của dự án (hoặc thư mục gốc nếu bạn muốn đơn giản).

```csharp
// Configuration/ConfigureSwaggerOptions.cs
using Microsoft.AspNetCore.Mvc.ApiExplorer;
using Microsoft.Extensions.Options;
using Microsoft.OpenApi.Models;
using Swashbuckle.AspNetCore.SwaggerGen;

namespace YourProjectName.Configuration // Đảm bảo namespace này khớp với nơi bạn đã using trong Program.cs
{
    // Lớp này cấu hình các tùy chọn cho SwaggerGen để hỗ trợ phiên bản hóa.
    public class ConfigureSwaggerOptions : IConfigureOptions<SwaggerGenOptions>
    {
        private readonly IApiVersionDescriptionProvider _provider;

        // Constructor inject IApiVersionDescriptionProvider.
        // DI container sẽ tự động cung cấp thể hiện của IApiVersionDescriptionProvider.
        public ConfigureSwaggerOptions(IApiVersionDescriptionProvider provider)
        {
            _provider = provider;
        }

        // Phương thức này được gọi bởi hệ thống DI để cấu hình SwaggerGenOptions.
        public void Configure(SwaggerGenOptions options)
        {
            // Thêm một Swagger document (OpenApiInfo) cho mỗi phiên bản API đã được khám phá.
            foreach (var description in _provider.ApiVersionDescriptions)
            {
                options.SwaggerDoc(description.GroupName, CreateInfoForApiVersion(description));
            }
        }

        // Phương thức trợ giúp để tạo đối tượng OpenApiInfo cho từng phiên bản API.
        private OpenApiInfo CreateInfoForApiVersion(ApiVersionDescription description)
        {
            var info = new OpenApiInfo()
            {
                Title = "Your Versioned API", // Tên API của bạn
                Version = description.ApiVersion.ToString(), // Phiên bản API (ví dụ: 1.0, 2.0)
                Description = "A sample API with versioning built with ASP.NET Core." // Mô tả API
            };

            // Nếu phiên bản API bị đánh dấu là lỗi thời (deprecated), thêm thông báo vào mô tả.
            if (description.IsDeprecated)
            {
                info.Description += " This API version has been deprecated.";
            }

            return info;
        }
    }
}
```

Với các thay đổi này, khi bạn chạy ứng dụng, Swagger UI sẽ hiển thị một danh sách thả xuống (dropdown) ở góc trên bên phải, cho phép bạn chọn giữa các phiên bản API khác nhau (ví dụ: `v1`, `v2`). Mỗi lựa chọn sẽ hiển thị tài liệu cho phiên bản API tương ứng.

> [!TIP]
> **Antigravity IDE và Swagger Integration:**
> Khi bạn muốn tích hợp Swagger với phiên bản hóa, Antigravity IDE có thể là một trợ thủ đắc lực. Với "vibe" là "kích hoạt Swagger cho các phiên bản API," Antigravity sẽ:
> 1.  **Kiểm tra phụ thuộc:** Đảm bảo gói `Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer` đã được cài đặt. Nếu chưa, nó sẽ tự động thêm.
> 2.  **Cấu hình `Program.cs`:** Tự động thêm `AddVersionedApiExplorer` và dòng `AddTransient<IConfigureOptions<SwaggerGenOptions>, ConfigureSwaggerOptions>()`.
> 3.  **Tạo `ConfigureSwaggerOptions.cs`:** Tự động tạo file này với nội dung chuẩn, bao gồm việc inject `IApiVersionDescriptionProvider` và logic tạo `OpenApiInfo` cho từng phiên bản.
> 4.  **Cập nhật `UseSwaggerUI`:** Chỉnh sửa vòng lặp để tạo các endpoint Swagger cho từng phiên bản.
> 5.  **Giải thích:** Cung cấp giải thích chi tiết về từng phần cấu hình, giúp bạn hiểu rõ cơ chế hoạt động của Swagger với phiên bản hóa.
> Khả năng của Antigravity trong việc tự động hóa các tác vụ cấu hình phức tạp này giúp tiết kiệm thời gian và giảm thiểu lỗi, cho phép bạn tập trung vào việc xây dựng logic nghiệp vụ chính.

## 8. Tổng kết và Thực hành tốt nhất

Phiên bản hóa API là một khía cạnh không thể thiếu của việc phát triển API hiện đại, giúp duy trì sự ổn định, khả năng mở rộng và quản lý hiệu quả vòng đời của hệ thống.

### 8.1. Khi nào nên phiên bản hóa?

*   **Thay đổi đột phá (Breaking Change):** Đây là lý do chính. Bất cứ khi nào bạn thực hiện một thay đổi có thể phá vỡ các ứng dụng khách hiện có, bạn cần một phiên bản API mới.
*   **Giới thiệu các tính năng mới quan trọng:** Đôi khi, một tập hợp các tính năng mới lớn có thể biện minh cho một phiên bản mới, ngay cả khi không có thay đổi đột phá trực tiếp, để nhóm các tính năng đó lại với nhau.
*   **Hỗ trợ các ứng dụng khách khác nhau:** Các client cũ (ví dụ: ứng dụng di động đã phát hành) có thể yêu cầu phiên bản API cũ, trong khi các client mới có thể tận dụng các tính năng mới nhất.
*   **Tái cấu trúc lớn:** Khi kiến trúc nội bộ của API thay đổi đáng kể, việc tạo một phiên bản mới có thể là cách tốt nhất để phản ánh sự thay đổi này ra bên ngoài.

### 8.2. Chọn chiến lược phiên bản hóa phù hợp

*   **Dựa trên URL (Path Versioning):** Rõ ràng, dễ hiểu, dễ kiểm thử, thân thiện với cache. Được khuyến nghị cho hầu hết các trường hợp vì tính minh bạch và đơn giản của nó. Đây là phương pháp chúng ta đã triển khai với gói NuGet.
*   **Dựa trên Query Parameter:** Đơn giản để thay đổi phiên bản từ phía client, nhưng có thể dễ bị bỏ qua và có vấn đề với cache. Phù hợp cho các trường hợp mà URL tài nguyên cơ bản phải cố định.
*   **Dựa trên Header/Media Type:** Tinh vi hơn, phù hợp với nguyên tắc RESTful hơn, URL không thay đổi, giúp duy trì khả năng lưu trữ cache tốt hơn. Tuy nhiên, phức tạp hơn cho việc kiểm thử và tài liệu hóa. Thường được sử dụng trong các hệ thống đòi hỏi độ chính xác cao về REST.

### 8.3. Quản lý DTOs và Models trong các phiên bản

*   **Sử dụng DTOs riêng biệt cho mỗi phiên bản:** Đây là một thực hành tốt nhất. Đối với mỗi phiên bản API có thay đổi đột phá về cấu trúc dữ liệu, hãy tạo một DTO riêng (ví dụ: `CountryDtoV1`, `CountryDtoV2`). Điều này đảm bảo rằng mỗi phiên bản API trả về một cấu trúc dữ liệu rõ ràng, nhất quán và độc lập với các phiên bản khác.
*   **Ánh xạ (Mapping) hiệu quả:** Sử dụng các thư viện ánh xạ như [AutoMapper](https://automapper.org/) hoặc ánh xạ thủ công để chuyển đổi giữa Domain Model (ví dụ: `Country.cs`) và các DTO phiên bản khác nhau. AutoMapper giúp giảm thiểu boilerplate code cho việc ánh xạ.
*   **Domain Model ổn định:** Thường thì Domain Model (ví dụ: `Country.cs`) nên được giữ ổn định và độc lập với các phiên bản API càng nhiều càng tốt. Các thay đổi đối với Domain Model cần được quản lý cẩn thận, có thể yêu cầu phiên bản hóa cơ sở dữ liệu (database versioning) song song.

### 8.4. Tài liệu hóa API versions

*   **Swagger/OpenAPI là bắt buộc:** Luôn tích hợp Swagger để tự động tạo tài liệu cho tất cả các phiên bản API của bạn. Điều này giúp các nhà phát triển ứng dụng khách dễ dàng hiểu, khám phá và sử dụng API của bạn.
*   **Mô tả rõ ràng:** Đảm bảo rằng tài liệu Swagger của bạn cung cấp mô tả rõ ràng về các thay đổi giữa các phiên bản và bất kỳ phiên bản nào bị lỗi thời (deprecated). Sử dụng thuộc tính `[Obsolete]` trong C# hoặc đánh dấu `IsDeprecated` trong Swagger để cảnh báo client về các phiên bản sắp ngừng hỗ trợ.

### 8.5. Quản lý vòng đời và Deprecation (Đánh dấu lỗi thời)

*   **Không phiên bản hóa quá mức:** Không phải mọi thay đổi nhỏ đều cần một phiên bản API mới. Chỉ phiên bản hóa khi có một thay đổi đột phá thực sự. Việc có quá nhiều phiên bản có thể làm tăng độ phức tạp trong việc duy trì và gây khó khăn cho client.
*   **Thông báo Deprecation rõ ràng:** Khi bạn quyết định ngừng hỗ trợ một phiên bản API cũ, hãy thông báo rõ ràng cho các ứng dụng khách thông qua tài liệu, email, hoặc tiêu đề HTTP (`Deprecation` header).
*   **Cung cấp thời gian chuyển đổi:** Cho phép các ứng dụng khách một khoảng thời gian hợp lý (ví dụ: 6-12 tháng) để chuyển đổi sang phiên bản API mới trước khi bạn chính thức ngừng hỗ trợ (sunsetting) phiên bản cũ.
*   **Giám sát việc sử dụng:** Sử dụng các công cụ giám sát API để theo dõi lưu lượng truy cập của từng phiên bản. Điều này giúp bạn xác định khi nào một phiên bản cũ không còn được sử dụng và có thể an toàn để loại bỏ.

> [!TIP]
> **Tư duy Vibe Coding và Antigravity IDE trong quản lý vòng đời API:**
> Antigravity IDE, với khả năng Agentic AI, có thể đóng vai trò như một "kiến trúc sư phụ" trong việc quản lý vòng đời API.
> 1.  **Phân tích tác động:** Khi bạn thực hiện một thay đổi, Antigravity có thể phân tích "vibe" của thay đổi đó (ví dụ: "thay đổi này ảnh hưởng đến nhiều client cũ") và tự động đề xuất tạo một phiên bản API mới, đồng thời đánh dấu phiên bản cũ là `deprecated` trong code và Swagger.
> 2.  **Lập kế hoạch Deprecation:** Dựa trên các chính sách đã định nghĩa hoặc dữ liệu sử dụng API (nếu được tích hợp), Antigravity có thể đề xuất một lịch trình deprecation và sunsetting, tạo các task nhắc nhở cho đội phát triển.
> 3.  **Tạo thông báo tự động:** Khi một phiên bản bị deprecated, nó có thể tự động tạo các thông báo trong tài liệu Swagger và thậm chí gợi ý nội dung email thông báo cho các đối tác.
> 4.  **Hỗ trợ loại bỏ phiên bản cũ:** Khi đến thời điểm sunset, Antigravity có thể hướng dẫn bạn cách loại bỏ code của phiên bản cũ một cách an toàn, đảm bảo không có phụ thuộc nào bị bỏ sót.
> Khả năng này biến Antigravity thành một công cụ không chỉ viết code mà còn hỗ trợ các quyết định chiến lược và quản lý dự án phức tạp.

## 9. Tóm tắt Chương

Chương này đã trang bị cho bạn kiến thức và kỹ năng cần thiết để triển khai phiên bản hóa API một cách chuyên nghiệp trong ASP.NET Core:

*   **Mục đích của phiên bản hóa API:** Quản lý các thay đổi đột phá, duy trì khả năng tương thích ngược và cho phép API phát triển một cách linh hoạt.
*   **Các phương pháp phiên bản hóa phổ biến:** Dựa trên URL (path), tham số truy vấn (query parameter), tiêu đề (header) và kiểu media (media type), cùng với ưu nhược điểm của từng phương pháp.
*   **Phiên bản hóa thủ công:** Một phương pháp trực quan nhưng không hiệu quả cho các dự án lớn, thường dẫn đến sao chép mã.
*   **Phiên bản hóa bằng gói NuGet (`Microsoft.AspNetCore.Mvc.Versioning`):** Phương pháp được khuyến nghị, cho phép quản lý các phiên bản trong cùng một controller file bằng cách sử dụng các thuộc tính `[ApiVersion]` và `[MapToApiVersion]`.
*   **Cấu hình `AddApiVersioning`:** Thiết lập phiên bản mặc định, cách đọc phiên bản từ yêu cầu (URL, query string, header) thông qua `ApiVersionReader`.
*   **Khắc phục Swagger UI (`Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer`):** Cần gói này để Swagger có thể khám phá và hiển thị đúng tất cả các phiên bản API.
*   **Cấu hình `AddVersionedApiExplorer` và `ConfigureSwaggerOptions`:** Sử dụng `IApiVersionDescriptionProvider` và Dependency Injection để tự động tạo tài liệu Swagger cho mỗi phiên bản API.
*   **Thực hành tốt nhất:** Phiên bản hóa khi có thay đổi đột phá, chọn chiến lược phù hợp, quản lý DTOs rõ ràng, và tài liệu hóa đầy đủ bằng Swagger.
*   **Áp dụng tư duy Vibe Coding và Antigravity IDE:** Cách một hệ thống AI mạnh mẽ có thể hỗ trợ và tự động hóa các tác vụ liên quan đến phiên bản hóa, từ việc nhận diện breaking changes đến quản lý vòng đời API.

Với những kiến thức này, bạn có thể tự tin xây dựng và duy trì các RESTful Web API mạnh mẽ, linh hoạt và dễ bảo trì, sẵn sàng cho sự phát triển không ngừng của ứng dụng.

<!-- REVIEWED_BY_AGENT -->
