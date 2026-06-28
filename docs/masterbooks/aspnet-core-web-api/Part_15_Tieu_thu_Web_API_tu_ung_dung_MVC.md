# Phần 15: Tiêu thụ Web API từ ứng dụng MVC

## Giới thiệu tổng quan

Trong kỷ nguyên phát triển phần mềm hiện đại, việc xây dựng các ứng dụng có khả năng giao tiếp và trao đổi dữ liệu với các hệ thống khác là một yêu cầu không thể thiếu. Các ứng dụng web, đặc biệt là các ứng dụng frontend như ASP.NET Core MVC, thường cần tiêu thụ dữ liệu từ các nguồn bên ngoài hoặc các dịch vụ backend chuyên biệt để mở rộng chức năng và cung cấp trải nghiệm người dùng phong phú. Một trong những phương pháp phổ biến và hiệu quả nhất để thực hiện điều này là thông qua việc tiêu thụ các Web API theo kiến trúc REST (Representational State Transfer).

Chương này sẽ hướng dẫn bạn cách xây dựng một ứng dụng web ASP.NET Core MVC có khả năng tiêu thụ một RESTful Web API. Chúng ta sẽ sử dụng API đã được xây dựng trong các phần trước của khóa học (sử dụng ASP.NET Core, C# và Entity Framework Core) làm nguồn dữ liệu. Mục tiêu là giúp bạn hiểu rõ từ cơ bản đến nâng cao về cách một ứng dụng frontend (MVC) tương tác với một backend API để thực hiện các thao tác CRUD (Create, Read, Update, Delete) trên các tài nguyên.

Chúng ta sẽ đi sâu vào các khía cạnh kỹ thuật quan trọng, bao gồm:
*   **Nền tảng Web API và RESTful**: Hiểu các nguyên tắc cơ bản của REST và cách HTTP Verbs được sử dụng.
*   **Thiết lập môi trường**: Tạo và cấu hình ứng dụng ASP.NET Core MVC làm client.
*   **Cơ chế giao tiếp hiệu quả**: Sử dụng `HttpClient` và `IHttpClientFactory` một cách đúng đắn, cùng với việc xây dựng một lớp dịch vụ (Service Layer) để trừu tượng hóa các cuộc gọi API.
*   **Thực hiện các thao tác CRUD**: Hướng dẫn chi tiết cách gửi các yêu cầu `GET`, `POST`, `PUT`, và `DELETE` để tương tác với tài nguyên.
*   **Xử lý dữ liệu và lỗi**: Cách tuần tự hóa/giải tuần tự hóa JSON, quản lý trạng thái ứng dụng, và xử lý các lỗi tiềm ẩn.
*   **Tư duy Vibe Coding và Antigravity IDE**: Khám phá cách tiếp cận "Vibe Coding" có thể được áp dụng để xây dựng các tương tác API một cách trực quan, và cách một hệ thống AI như Antigravity IDE có thể hỗ trợ và tăng tốc quá trình này.

Xuyên suốt chương này, bạn sẽ được hướng dẫn từng bước với các ví dụ code minh họa chi tiết, giúp bạn nắm vững kiến thức và áp dụng vào các dự án thực tế một cách khoa học và chỉn chu.

## I. Nền tảng Web API và RESTful

Trước khi đi sâu vào việc tiêu thụ API, việc hiểu rõ các nguyên tắc nền tảng của Web API và kiến trúc RESTful là rất quan trọng.

### 1. Kiến trúc REST (Representational State Transfer)

REST không phải là một giao thức mà là một phong cách kiến trúc phần mềm (architectural style) cho các hệ thống siêu phương tiện phân tán. Nó định nghĩa một tập hợp các nguyên tắc thiết kế để tạo ra các dịch vụ web nhẹ, có khả năng mở rộng và dễ bảo trì.

Các nguyên tắc cốt lõi của REST bao gồm:

*   **Client-Server**: Tách biệt giao diện người dùng (client) khỏi lưu trữ dữ liệu (server). Điều này cho phép phát triển độc lập client và server, cải thiện tính di động của giao diện người dùng trên nhiều nền tảng và tăng khả năng mở rộng của server. *Ví dụ: Ứng dụng di động của bạn (client) giao tiếp với API trên đám mây (server) mà không cần biết cách dữ liệu được lưu trữ.*
*   **Stateless (Không trạng thái)**: Mỗi yêu cầu từ client đến server phải chứa tất cả thông tin cần thiết để server hiểu và xử lý yêu cầu. Server không lưu trữ bất kỳ trạng thái nào của client giữa các yêu cầu. Điều này đơn giản hóa thiết kế server, cải thiện khả năng mở rộng và khả năng phục hồi. *Ví dụ: Mỗi lần bạn gửi yêu cầu để lấy danh sách sản phẩm, yêu cầu đó phải độc lập, không dựa vào thông tin của yêu cầu trước đó.*
*   **Cacheable (Có thể lưu trữ cache)**: Phản hồi từ server có thể được đánh dấu là có thể lưu trữ cache hoặc không. Điều này giúp cải thiện hiệu suất bằng cách giảm thiểu các yêu cầu lặp lại đến server cho cùng một tài nguyên. *Ví dụ: Trình duyệt của bạn có thể lưu trữ phản hồi từ API cho một danh sách sản phẩm ít thay đổi, giảm tải cho server.*
*   **Layered System (Hệ thống phân lớp)**: Client có thể không biết liệu nó có đang kết nối trực tiếp với server cuối cùng hay một lớp trung gian nào đó (ví dụ: proxy, load balancer, API Gateway). Điều này giúp cải thiện tính linh hoạt, khả năng mở rộng và bảo mật. *Ví dụ: Yêu cầu của bạn có thể đi qua một tường lửa hoặc một dịch vụ cân bằng tải trước khi đến API thực tế.*
*   **Uniform Interface (Giao diện đồng nhất)**: Đây là nguyên tắc cốt lõi của REST, đảm bảo sự đơn giản và tách biệt kiến trúc. Nó bao gồm:
    *   **Identification of resources (Nhận diện tài nguyên)**: Tài nguyên được nhận diện bằng URI (Uniform Resource Identifier) duy nhất. *Ví dụ: `/api/regions` cho danh sách vùng, `/api/regions/123` cho một vùng cụ thể.*
    *   **Manipulation of resources through representations (Thao tác tài nguyên thông qua biểu diễn)**: Client tương tác với tài nguyên bằng cách gửi và nhận các biểu diễn của tài nguyên (thường là JSON hoặc XML). Bạn không thao tác trực tiếp với cơ sở dữ liệu mà là với "biểu diễn" của dữ liệu.
    *   **Self-descriptive messages (Thông báo tự mô tả)**: Mỗi thông báo chứa đủ thông tin để client hiểu cách xử lý nó (ví dụ: thông qua các HTTP headers như `Content-Type`).
    *   **Hypermedia as the Engine of Application State (HATEOAS)**: Client tương tác với ứng dụng hoàn toàn thông qua siêu phương tiện động được cung cấp bởi server (ví dụ: API trả về dữ liệu kèm theo các liên kết đến các hành động liên quan). *Mặc dù đây là một phần quan trọng của REST, nhiều Web API hiện đại không tuân thủ hoàn toàn HATEOAS để đơn giản hóa, nhưng việc hiểu nó là quan trọng.*

### 2. HTTP Verbs (Phương thức HTTP) và CRUD

Các phương thức HTTP là nền tảng cho việc tương tác với tài nguyên trong một RESTful API. Mỗi phương thức tương ứng với một thao tác CRUD cơ bản và có ngữ nghĩa riêng:

| Phương thức HTTP | Thao tác CRUD | Mô tả                                                                                                | Tính an toàn (Safe) | Tính Idempotent |
| :--------------- | :------------ | :--------------------------------------------------------------------------------------------------- | :----------------- | :-------------- |
| `GET`            | Read          | Lấy dữ liệu từ server. Không thay đổi trạng thái server.                                          | Có                 | Có              |
| `POST`           | Create        | Tạo tài nguyên mới trên server. Có thể thay đổi trạng thái server và không Idempotent (gửi nhiều lần có thể tạo nhiều tài nguyên). | Không              | Không           |
| `PUT`            | Update        | Cập nhật toàn bộ tài nguyên đã có trên server. Nếu tài nguyên không tồn tại, có thể tạo mới (thay thế). Idempotent (gửi nhiều lần có cùng kết quả). | Không              | Có              |
| `PATCH`          | Update        | Cập nhật một phần tài nguyên trên server. Idempotent.                                            | Không              | Có              |
| `DELETE`         | Delete        | Xóa tài nguyên khỏi server. Idempotent.                                                            | Không              | Có              |

*   **Tính an toàn (Safe)**: Một phương thức là "an toàn" nếu nó không gây ra bất kỳ tác dụng phụ nào trên server (chỉ đọc dữ liệu). `GET` là phương thức an toàn duy nhất.
*   **Tính Idempotent**: Một phương thức là "idempotent" nếu việc thực hiện nó nhiều lần với cùng một tham số sẽ cho cùng một kết quả như khi thực hiện một lần. `GET`, `PUT`, `DELETE`, `PATCH` là idempotent. `POST` thì không.

Trong chương này, chúng ta sẽ tập trung vào `GET`, `POST`, `PUT` và `DELETE` để thực hiện các thao tác CRUD đầy đủ.

### 3. Định dạng dữ liệu và Trao đổi

Trong các Web API hiện đại, **JSON (JavaScript Object Notation)** là định dạng dữ liệu được sử dụng phổ biến nhất để trao đổi thông tin giữa client và server. JSON có cấu trúc nhẹ, dễ đọc bởi con người và dễ phân tích cú pháp bởi máy.

Trong .NET Core, thư viện `System.Text.Json` là lựa chọn mặc định và được khuyến nghị để tuần tự hóa (serialize) và giải tuần tự hóa (deserialize) JSON do hiệu suất cao và tích hợp sâu.

## II. Chuẩn bị môi trường ứng dụng MVC Client

Để tiêu thụ Web API, chúng ta cần một ứng dụng client. Trong trường hợp này, chúng ta sẽ tạo một ứng dụng ASP.NET Core MVC mới trong cùng một `Solution` với dự án Web API đã có.

### 1. Tạo dự án ASP.NET Core MVC

1.  **Mở Visual Studio**: Đảm bảo bạn đang ở trong `Solution` chứa dự án Web API của mình (ví dụ: `NZWalks`).
2.  **Thêm dự án mới**: Nhấp chuột phải vào `Solution` trong `Solution Explorer`, chọn `Add` -> `New Project...`.
3.  **Chọn mẫu dự án**:
    *   Tìm kiếm "ASP.NET Core Web App (Model-View-Controller)".
    *   Chọn mẫu này và nhấp `Next`.
    *   > [!TIP]
        > Khóa học này tập trung vào MVC. Nếu bạn muốn sử dụng Razor Pages, hãy chọn "ASP.NET Core Web App" (không có MVC trong tên).
4.  **Đặt tên dự án**: Đặt tên dự án là `NZWalks.UI` (hoặc tên tương tự như `[TênGiảiPháp].UI`). Nhấp `Next`.
5.  **Cấu hình framework**: Chọn phiên bản .NET Core mới nhất (ví dụ: .NET 8.0). Đảm bảo `Authentication Type` là `None` cho mục đích demo này. Nhấp `Create`.

Visual Studio sẽ tạo một dự án MVC cơ bản với cấu trúc thư mục chuẩn (Controllers, Views, Models, wwwroot, v.v.).

### 2. Cấu hình chạy đa dự án

Để ứng dụng MVC có thể giao tiếp với Web API, cả hai dự án cần chạy đồng thời.

1.  **Nhấp chuột phải vào `Solution`** trong `Solution Explorer`, chọn `Properties`.
2.  Trong cửa sổ `Solution Property Pages`, chọn `Startup Project` ở bên trái.
3.  Chọn `Multiple startup projects`.
4.  Trong danh sách các dự án, tìm dự án Web API (`NZWalks.API`) và dự án UI (`NZWalks.UI`). Đặt `Action` của cả hai thành `Start`.
5.  Nhấp `Apply`, sau đó `OK`.

Bây giờ, khi bạn chạy `Solution` (bằng cách nhấp vào nút `Start` hoặc nhấn `F5`), cả ứng dụng API và ứng dụng MVC sẽ khởi động trong các cửa sổ trình duyệt riêng biệt.

### 3. Lưu ý về Bảo mật (Authorization)

> [!NOTE]
> Để đơn giản hóa quá trình demo và tập trung vào cơ chế tiêu thụ API, trong dự án API (`NZWalks.API`), bạn nên tạm thời bỏ ghi chú (comment out) thuộc tính `[Authorize]` trên các controller hoặc phương thức hành động mà bạn muốn tiêu thụ.
>
> **Trong một ứng dụng thực tế, việc này là bắt buộc và rất quan trọng.** Khi triển khai sản phẩm, bạn sẽ cần:
> 1.  **Tái kích hoạt `[Authorize]`**: Đảm bảo các endpoint API được bảo vệ.
> 2.  **Triển khai cơ chế xác thực/ủy quyền**: Ví dụ: sử dụng JWT (JSON Web Tokens), OAuth 2.0 với một Identity Server (như IdentityServer4 hoặc Duende IdentityServer) để cấp và xác thực token.
> 3.  **Ứng dụng MVC client sẽ cần**:
    *   Một trang đăng nhập để người dùng lấy JWT token từ Identity Server.
    *   Lưu trữ token an toàn (ví dụ: trong cookie http-only).
    *   Đính kèm token này vào mỗi yêu cầu HTTP gửi đến API thông qua `Authorization` header (`Bearer [token]`). `HttpClientFactory` có thể được cấu hình để tự động thêm header này.

## III. Cơ chế giao tiếp API: `HttpClient` và `IHttpClientFactory`

Để ứng dụng MVC gửi yêu cầu HTTP đến Web API, chúng ta sẽ sử dụng lớp `HttpClient` của .NET. Tuy nhiên, việc sử dụng `HttpClient` cần được thực hiện đúng cách để tránh các vấn đề về hiệu suất và tài nguyên.

### 1. Vấn đề với `new HttpClient()`

Việc khởi tạo một `HttpClient` mới cho mỗi yêu cầu HTTP (`new HttpClient()`) là một sai lầm phổ biến và có thể dẫn đến các vấn đề nghiêm trọng trong các ứng dụng có tải cao:

*   **Socket Exhaustion**: Mỗi `HttpClient` mới tạo ra một kết nối TCP mới. Nếu bạn tạo quá nhiều `HttpClient` trong một thời gian ngắn, bạn có thể nhanh chóng cạn kiệt số lượng socket khả dụng trên hệ điều hành, dẫn đến lỗi "Unable to connect" hoặc "Only one usage of each socket address (protocol/network address/port) is normally permitted".
*   **DNS Caching Issues**: `HttpClient` không giải quyết lại DNS cho mỗi yêu cầu. Nếu một `HttpClient` tồn tại quá lâu và địa chỉ IP của dịch vụ API thay đổi (ví dụ: do cân bằng tải hoặc triển khai mới), `HttpClient` cũ sẽ tiếp tục cố gắng kết nối đến địa chỉ IP cũ không còn tồn tại, gây ra lỗi kết nối.
*   **Overhead**: Việc tạo và hủy các đối tượng `HttpClient` và các kết nối TCP cơ bản liên tục gây ra chi phí tài nguyên và làm giảm hiệu suất tổng thể.

### 2. Giải pháp: `IHttpClientFactory`

`IHttpClientFactory` được giới thiệu trong .NET Core 2.1 để giải quyết triệt để các vấn đề trên. Nó hoạt động như một factory để tạo và quản lý các thể hiện `HttpClient`, đồng thời quản lý vòng đời của các `HttpMessageHandler` cơ bản (thành phần chịu trách nhiệm cho việc gửi yêu cầu thực tế) bằng cách pooling chúng.

**Lợi ích của `IHttpClientFactory`**:
*   **Quản lý vòng đời hiệu quả**: Tái sử dụng `HttpMessageHandler` để giảm socket exhaustion và cải thiện hiệu suất.
*   **DNS Refresh**: Các handler được quản lý bởi factory có thời gian sống (lifetime) nhất định, đảm bảo DNS được giải quyết lại định kỳ, tránh các vấn đề DNS cũ.
*   **Cấu hình tập trung**: Cho phép cấu hình các `HttpClient` khác nhau cho các mục đích khác nhau (ví dụ: đặt `BaseAddress`, `Timeout`, hoặc thêm các `DelegatingHandler` cho logging, caching, retry policies).
*   **Tích hợp Dependency Injection**: Dễ dàng inject vào các dịch vụ và controller.

> [!IMPORTANT]
> Luôn sử dụng `IHttpClientFactory` để tạo các thể hiện `HttpClient` thay vì trực tiếp khởi tạo `new HttpClient()`.

### 3. Cấu hình `IHttpClientFactory` với Strong Typing

Để sử dụng `IHttpClientFactory`, bạn cần đăng ký nó vào hệ thống Dependency Injection của ASP.NET Core. Chúng ta cũng sẽ cấu hình URL API một cách mạnh mẽ (strongly typed) thông qua `appsettings.json` và `IOptions`.

#### a. Thêm cấu hình API vào `appsettings.json`

1.  Mở `appsettings.json` trong dự án `NZWalks.UI`.
2.  Thêm cấu hình sau:

    ```json
    {
      "Logging": {
        "LogLevel": {
          "Default": "Information",
          "Microsoft.AspNetCore": "Warning"
        }
      },
      "AllowedHosts": "*",
      "NZWalksApi": { // Tên một phần cấu hình cụ thể cho API của bạn
        "BaseUrl": "https://localhost:7001/" // Thay thế bằng URL API của bạn
      }
    }
    ```

#### b. Tạo lớp cấu hình strongly typed

1.  Trong dự án `NZWalks.UI`, tạo một thư mục mới có tên `Configuration` (hoặc `Settings`).
2.  Tạo một lớp mới tên `NZWalksApiSettings.cs` bên trong thư mục `Configuration`.

    ```csharp
    // NZWalks.UI/Configuration/NZWalksApiSettings.cs
    namespace NZWalks.UI.Configuration
    {
        public class NZWalksApiSettings
        {
            public const string SectionName = "NZWalksApi"; // Tên phần trong appsettings.json
            public string BaseUrl { get; set; } = string.Empty;
        }
    }
    ```

#### c. Đăng ký `IHttpClientFactory` và cấu hình trong `Program.cs`

1.  Mở `Program.cs` trong dự án `NZWalks.UI`.
2.  Thêm các `using` cần thiết và cấu hình như sau:

    ```csharp
    // Program.cs trong dự án NZWalks.UI
    using NZWalks.UI.Configuration; // Để dùng NZWalksApiSettings
    using Microsoft.Extensions.Options; // Để dùng IOptions

    var builder = WebApplication.CreateBuilder(args);

    // Add services to the container.
    builder.Services.AddControllersWithViews();

    // 1. Đăng ký phần cấu hình NZWalksApiSettings
    builder.Services.Configure<NZWalksApiSettings>(builder.Configuration.GetSection(NZWalksApiSettings.SectionName));

    // 2. Đăng ký HttpClient với BaseAddress được cấu hình từ appsettings
    builder.Services.AddHttpClient("NZWalksApi", (serviceProvider, client) =>
    {
        // Lấy cấu hình NZWalksApiSettings thông qua IOptions
        var apiSettings = serviceProvider.GetRequiredService<IOptions<NZWalksApiSettings>>().Value;
        client.BaseAddress = new Uri(apiSettings.BaseUrl);
        // Có thể thêm các cấu hình khác như Timeout
        // client.Timeout = TimeSpan.FromSeconds(30);
    });

    var app = builder.Build();
    // ... (phần còn lại của Program.cs)
    ```

### 4. Xây dựng lớp Service chuyên biệt cho API (Best Practice)

Để giữ cho Controller gọn gàng (thin controllers) và tách biệt logic giao tiếp API khỏi logic hiển thị, chúng ta sẽ tạo một lớp dịch vụ (Service Layer) chuyên biệt để xử lý tất cả các cuộc gọi đến `NZWalks.API`. Điều này tương tự như Repository Pattern, nhưng dành cho các dịch vụ bên ngoài.

#### a. Tạo thư mục `Services` và Interface

1.  Trong dự án `NZWalks.UI`, tạo một thư mục mới có tên `Services`.
2.  Trong thư mục `Services`, tạo một thư mục con tên `Interfaces`.
3.  Tạo interface `IRegionService.cs` trong `Services/Interfaces`:

    ```csharp
    // NZWalks.UI/Services/Interfaces/IRegionService.cs
    using NZWalks.UI.Models.DTO; // Đảm bảo đã tạo DTOs ở bước sau

    namespace NZWalks.UI.Services.Interfaces
    {
        public interface IRegionService
        {
            Task<List<RegionDto>?> GetAllRegionsAsync();
            Task<RegionDto?> GetRegionByIdAsync(Guid id);
            Task<RegionDto?> AddRegionAsync(AddRegionRequestDto addRegionRequestDto);
            Task<RegionDto?> UpdateRegionAsync(Guid id, UpdateRegionRequestDto updateRegionRequestDto);
            Task<bool> DeleteRegionAsync(Guid id);
        }
    }
    ```

#### b. Tạo lớp triển khai Service

1.  Trong thư mục `Services`, tạo lớp `RegionService.cs`:

    ```csharp
    // NZWalks.UI/Services/RegionService.cs
    using NZWalks.UI.Services.Interfaces;
    using NZWalks.UI.Models.DTO;
    using System.Text.Json;
    using System.Text;
    using Microsoft.Extensions.Options;
    using NZWalks.UI.Configuration;

    namespace NZWalks.UI.Services
    {
        public class RegionService : IRegionService
        {
            private readonly IHttpClientFactory _httpClientFactory;
            private readonly string _apiBaseUrl;
            private readonly ILogger<RegionService> _logger; // Thêm logger để ghi lỗi

            public RegionService(IHttpClientFactory httpClientFactory, 
                                 IOptions<NZWalksApiSettings> apiSettings,
                                 ILogger<RegionService> logger)
            {
                _httpClientFactory = httpClientFactory;
                _apiBaseUrl = apiSettings.Value.BaseUrl;
                _logger = logger;
            }

            private HttpClient CreateClient()
            {
                // Tạo một HttpClient đã được cấu hình sẵn với BaseAddress
                return _httpClientFactory.CreateClient("NZWalksApi");
            }

            public async Task<List<RegionDto>?> GetAllRegionsAsync()
            {
                try
                {
                    var client = CreateClient();
                    var httpResponseMessage = await client.GetAsync("api/regions"); // Chỉ cần đường dẫn tương đối

                    httpResponseMessage.EnsureSuccessStatusCode(); // Ném ngoại lệ nếu mã trạng thái không thành công

                    return await httpResponseMessage.Content.ReadFromJsonAsync<List<RegionDto>>();
                }
                catch (HttpRequestException ex)
                {
                    _logger.LogError(ex, "Lỗi HTTP khi lấy tất cả khu vực.");
                    // Xử lý lỗi cụ thể hơn, ví dụ: kiểm tra ex.StatusCode
                    return null;
                }
                catch (JsonException ex)
                {
                    _logger.LogError(ex, "Lỗi giải tuần tự hóa JSON khi lấy tất cả khu vực.");
                    return null;
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Đã xảy ra lỗi không mong muốn khi lấy tất cả khu vực.");
                    return null;
                }
            }

            public async Task<RegionDto?> GetRegionByIdAsync(Guid id)
            {
                try
                {
                    var client = CreateClient();
                    var httpResponseMessage = await client.GetAsync($"api/regions/{id}");

                    httpResponseMessage.EnsureSuccessStatusCode();

                    return await httpResponseMessage.Content.ReadFromJsonAsync<RegionDto>();
                }
                catch (HttpRequestException ex)
                {
                    _logger.LogError(ex, "Lỗi HTTP khi lấy khu vực với ID: {RegionId}.", id);
                    return null;
                }
                catch (JsonException ex)
                {
                    _logger.LogError(ex, "Lỗi giải tuần tự hóa JSON khi lấy khu vực với ID: {RegionId}.", id);
                    return null;
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Đã xảy ra lỗi không mong muốn khi lấy khu vực với ID: {RegionId}.", id);
                    return null;
                }
            }

            public async Task<RegionDto?> AddRegionAsync(AddRegionRequestDto addRegionRequestDto)
            {
                try
                {
                    var client = CreateClient();
                    var jsonContent = JsonSerializer.Serialize(addRegionRequestDto);
                    var httpContent = new StringContent(jsonContent, Encoding.UTF8, "application/json");

                    var httpResponseMessage = await client.PostAsync("api/regions", httpContent);

                    httpResponseMessage.EnsureSuccessStatusCode();

                    return await httpResponseMessage.Content.ReadFromJsonAsync<RegionDto>();
                }
                catch (HttpRequestException ex)
                {
                    _logger.LogError(ex, "Lỗi HTTP khi thêm khu vực mới.");
                    return null;
                }
                catch (JsonException ex)
                {
                    _logger.LogError(ex, "Lỗi giải tuần tự hóa JSON phản hồi khi thêm khu vực mới.");
                    return null;
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Đã xảy ra lỗi không mong muốn khi thêm khu vực mới.");
                    return null;
                }
            }

            public async Task<RegionDto?> UpdateRegionAsync(Guid id, UpdateRegionRequestDto updateRegionRequestDto)
            {
                try
                {
                    var client = CreateClient();
                    var jsonContent = JsonSerializer.Serialize(updateRegionRequestDto);
                    var httpContent = new StringContent(jsonContent, Encoding.UTF8, "application/json");

                    var httpResponseMessage = await client.PutAsync($"api/regions/{id}", httpContent);

                    httpResponseMessage.EnsureSuccessStatusCode();

                    // API có thể trả về đối tượng đã cập nhật hoặc chỉ mã trạng thái 204 No Content
                    if (httpResponseMessage.StatusCode == System.Net.HttpStatusCode.NoContent)
                    {
                        // Nếu API trả về NoContent, bạn có thể trả về DTO đã gửi đi hoặc null
                        // Tùy thuộc vào yêu cầu, có thể cần GET lại để lấy dữ liệu mới nhất
                        return new RegionDto // Trả về một DTO tạm thời nếu API không trả về nội dung
                        {
                            Id = id,
                            Code = updateRegionRequestDto.Code,
                            Name = updateRegionRequestDto.Name,
                            RegionImageUrl = updateRegionRequestDto.RegionImageUrl
                        };
                    }
                    return await httpResponseMessage.Content.ReadFromJsonAsync<RegionDto>();
                }
                catch (HttpRequestException ex)
                {
                    _logger.LogError(ex, "Lỗi HTTP khi cập nhật khu vực với ID: {RegionId}.", id);
                    return null;
                }
                catch (JsonException ex)
                {
                    _logger.LogError(ex, "Lỗi giải tuần tự hóa JSON phản hồi khi cập nhật khu vực với ID: {RegionId}.", id);
                    return null;
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Đã xảy ra lỗi không mong muốn khi cập nhật khu vực với ID: {RegionId}.", id);
                    return null;
                }
            }

            public async Task<bool> DeleteRegionAsync(Guid id)
            {
                try
                {
                    var client = CreateClient();
                    var httpResponseMessage = await client.DeleteAsync($"api/regions/{id}");

                    httpResponseMessage.EnsureSuccessStatusCode();
                    return true;
                }
                catch (HttpRequestException ex)
                {
                    _logger.LogError(ex, "Lỗi HTTP khi xóa khu vực với ID: {RegionId}.", id);
                    return false;
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Đã xảy ra lỗi không mong muốn khi xóa khu vực với ID: {RegionId}.", id);
                    return false;
                }
            }
        }
    }
    ```

#### c. Đăng ký Service vào Dependency Injection

1.  Trong `Program.cs`, thêm dòng sau sau `builder.Services.AddHttpClient(...)`:

    ```csharp
    // Program.cs trong dự án NZWalks.UI
    // ...
    builder.Services.AddHttpClient("NZWalksApi", (serviceProvider, client) =>
    {
        var apiSettings = serviceProvider.GetRequiredService<IOptions<NZWalksApiSettings>>().Value;
        client.BaseAddress = new Uri(apiSettings.BaseUrl);
    });

    // Đăng ký RegionService vào Dependency Injection
    builder.Services.AddScoped<IRegionService, RegionService>();

    var app = builder.Build();
    // ...
    ```
    Bây giờ, `RegionService` đã sẵn sàng để được inject vào các Controller hoặc các dịch vụ khác.

### 5. Inject `IRegionService` vào Controller

Sau khi đăng ký `IRegionService`, bạn có thể inject nó vào constructor của Controller.

```csharp
// Trong một Controller (ví dụ: RegionsController.cs)
using Microsoft.AspNetCore.Mvc;
using NZWalks.UI.Services.Interfaces; // Namespace cho IRegionService
// ... các using khác

namespace NZWalks.UI.Controllers
{
    public class RegionsController : Controller
    {
        private readonly IRegionService _regionService;

        // Constructor để inject IRegionService
        public RegionsController(IRegionService regionService)
        {
            _regionService = regionService;
        }

        // ... Các phương thức hành động (Action Methods) sẽ ở đây
    }
}
```

## IV. Tiêu thụ dữ liệu (GET)

Chúng ta sẽ bắt đầu bằng cách lấy dữ liệu từ API và hiển thị nó trong ứng dụng MVC.

### 1. Lấy tất cả tài nguyên (GET All)

Mục tiêu là hiển thị danh sách tất cả các khu vực từ API trong một bảng trên trang web.

#### a. Tạo DTO (Data Transfer Object) cho UI

Mặc dù API của bạn đã có DTO (ví dụ: `RegionDto` trong dự án API), bạn nên tạo một DTO tương tự trong dự án UI của mình. Điều này giúp duy trì sự tách biệt giữa các tầng, cho phép UI có các thuộc tính riêng hoặc xử lý các trường hợp khi API bên ngoài không khớp hoàn toàn với các mô hình nội bộ của bạn.

1.  Trong dự án `NZWalks.UI`, tạo một thư mục mới có tên `DTO` bên trong thư mục `Models`.
2.  Tạo một lớp mới tên `RegionDto.cs` bên trong thư mục `Models/DTO`.
3.  Thêm các thuộc tính sau, khớp với DTO mà API trả về:

    ```csharp
    // NZWalks.UI/Models/DTO/RegionDto.cs
    using System.ComponentModel.DataAnnotations;

    namespace NZWalks.UI.Models.DTO
    {
        public class RegionDto
        {
            public Guid Id { get; set; }
            
            [Required(ErrorMessage = "Mã vùng là bắt buộc.")]
            [MinLength(2, ErrorMessage = "Mã vùng phải có ít nhất 2 ký tự.")]
            [MaxLength(3, ErrorMessage = "Mã vùng không được quá 3 ký tự.")]
            public string Code { get; set; } = string.Empty;

            [Required(ErrorMessage = "Tên vùng là bắt buộc.")]
            [MaxLength(100, ErrorMessage = "Tên vùng không được quá 100 ký tự.")]
            public string Name { get; set; } = string.Empty;
            
            public string? RegionImageUrl { get; set; } // Có thể null
        }
    }
    ```
    > [!NOTE]
    > Tôi đã thêm Data Annotations vào `RegionDto` trong UI. Điều này sẽ giúp MVC thực hiện validation phía server trước khi cố gắng gửi dữ liệu lên API, và cũng có thể được dùng cho validation phía client với `asp-validation-for`.

#### b. Tạo Controller và Action Method

1.  Trong thư mục `Controllers` của dự án `NZWalks.UI`, nhấp chuột phải, chọn `Add` -> `Controller...`.
2.  Chọn `MVC Controller - Empty` và nhấp `Add`.
3.  Đặt tên Controller là `RegionsController.cs`.

    ```csharp
    // NZWalks.UI/Controllers/RegionsController.cs
    using Microsoft.AspNetCore.Mvc;
    using NZWalks.UI.Models.DTO; // Để dùng RegionDto
    using NZWalks.UI.Services.Interfaces; // Để inject IRegionService

    namespace NZWalks.UI.Controllers
    {
        public class RegionsController : Controller
        {
            private readonly IRegionService _regionService;

            public RegionsController(IRegionService regionService)
            {
                _regionService = regionService;
            }

            [HttpGet]
            public async Task<IActionResult> Index()
            {
                List<RegionDto> regions = new List<RegionDto>();

                // Gọi dịch vụ để lấy dữ liệu
                regions = (await _regionService.GetAllRegionsAsync())?.ToList() ?? new List<RegionDto>();
                
                if (regions.Count == 0 && TempData["ErrorMessage"] == null)
                {
                    TempData["ErrorMessage"] = "Không thể tải dữ liệu khu vực. Vui lòng thử lại sau.";
                }

                // Truyền danh sách vùng đến View
                return View(regions);
            }
        }
    }
    ```

#### c. Tạo View để hiển thị dữ liệu

1.  Trong thư mục `Views/Regions` (tạo nếu chưa có), nhấp chuột phải, chọn `Add` -> `View` -> `Razor View - Empty`.
2.  Đặt tên View là `Index.cshtml`.
3.  Thêm code sau vào `Index.cshtml` để hiển thị danh sách vùng trong một bảng:

    ```html
    @model List<NZWalks.UI.Models.DTO.RegionDto>

    @{
        ViewData["Title"] = "Regions";
    }

    <h1 class="mt-3">Regions</h1>

    @if (TempData["SuccessMessage"] != null)
    {
        <div class="alert alert-success" role="alert">
            @TempData["SuccessMessage"]
        </div>
    }

    @if (TempData["ErrorMessage"] != null)
    {
        <div class="alert alert-danger" role="alert">
            @TempData["ErrorMessage"]
        </div>
    }

    <div class="d-flex justify-content-end">
        <a href="/Regions/Add" class="btn btn-primary">Add Region</a>
    </div>

    @if (Model != null && Model.Any())
    {
        <table class="table table-bordered mt-3">
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Code</th>
                    <th>Name</th>
                    <th>Image URL</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                @foreach (var region in Model)
                {
                    <tr>
                        <td>@region.Id</td>
                        <td>@region.Code</td>
                        <td>@region.Name</td>
                        <td>
                            @if (!string.IsNullOrWhiteSpace(region.RegionImageUrl))
                            {
                                <img src="@region.RegionImageUrl" style="max-width: 100px; max-height: 50px;" alt="Region Image" />
                            }
                            else
                            {
                                <span>No Image</span>
                            }
                        </td>
                        <td>
                            <a href="/Regions/Edit/@region.Id" class="btn btn-light">Edit</a>
                        </td>
                    </tr>
                }
            </tbody>
        </table>
    }
    else
    {
        <p class="mt-3">No regions found.</p>
    }
    ```

> [!NOTE]
> Để truy cập trang `Index` của `RegionsController`, bạn có thể thêm một liên kết vào menu điều hướng chính của ứng dụng (`_Layout.cshtml` trong `Views/Shared`):
>
> ```html
> <li class="nav-item">
>     <a class="nav-link text-dark" asp-area="" asp-controller="Regions" asp-action="Index">Regions</a>
> </li>
> ```

### 2. Lấy một tài nguyên cụ thể (GET Single) - Chuẩn bị cho chỉnh sửa

Để chỉnh sửa một khu vực, trước tiên chúng ta cần lấy thông tin chi tiết của khu vực đó từ API và điền vào một biểu mẫu.

#### a. Thêm nút "Edit" vào danh sách vùng

Chúng ta đã thêm nút "Edit" vào `Index.cshtml` ở bước trên. Nút này sẽ điều hướng đến `"/Regions/Edit/@region.Id"`.

#### b. Tạo Action Method `Edit` (GET)

1.  Trong `RegionsController.cs`, thêm phương thức `Edit` (HTTP GET) để nhận `Id` của khu vực:

    ```csharp
    // NZWalks.UI/Controllers/RegionsController.cs
    // ... (trong RegionsController)

    [HttpGet]
    public async Task<IActionResult> Edit(Guid id)
    {
        var regionDto = await _regionService.GetRegionByIdAsync(id);

        if (regionDto == null)
        {
            TempData["ErrorMessage"] = "Không tìm thấy khu vực bạn muốn chỉnh sửa.";
            return RedirectToAction("Index");
        }

        // Truyền DTO đến View
        return View(regionDto);
    }
    ```

#### c. Tạo View `Edit.cshtml`

1.  Trong thư mục `Views/Regions`, nhấp chuột phải, chọn `Add` -> `View` -> `Razor View - Empty`.
2.  Đặt tên View là `Edit.cshtml`.
3.  Thêm code sau để hiển thị form chỉnh sửa, điền sẵn dữ liệu đã lấy từ API:

    ```html
    @model NZWalks.UI.Models.DTO.RegionDto

    @{
        ViewData["Title"] = "Edit Region";
    }

    <h1 class="mt-3">Edit Region</h1>

    @if (TempData["SuccessMessage"] != null)
    {
        <div class="alert alert-success" role="alert">
            @TempData["SuccessMessage"]
        </div>
    }

    @if (TempData["ErrorMessage"] != null)
    {
        <div class="alert alert-danger" role="alert">
            @TempData["ErrorMessage"]
        </div>
    }

    @if (Model != null)
    {
        <form method="post" action="/Regions/Edit">
            <div class="mt-3">
                <label for="Id" class="form-label">Id</label>
                <input type="text" class="form-control" id="Id" asp-for="Id" readonly />
            </div>

            <div class="mt-3">
                <label for="Code" class="form-label">Code</label>
                <input type="text" class="form-control" id="Code" asp-for="Code" />
                <span asp-validation-for="Code" class="text-danger"></span>
            </div>

            <div class="mt-3">
                <label for="Name" class="form-label">Name</label>
                <input type="text" class="form-control" id="Name" asp-for="Name" />
                <span asp-validation-for="Name" class="text-danger"></span>
            </div>

            <div class="mt-3">
                <label for="RegionImageUrl" class="form-label">Region Image URL</label>
                <input type="text" class="form-control" id="RegionImageUrl" asp-for="RegionImageUrl" />
                <span asp-validation-for="RegionImageUrl" class="text-danger"></span>
            </div>

            <div class="mt-3 d-flex justify-content-between">
                <button type="submit" class="btn btn-primary">Save</button>
                <button type="submit" class="btn btn-danger" 
                        asp-controller="Regions" asp-action="Delete" asp-route-id="@Model.Id">Delete</button>
            </div>
        </form>
    }
    ```

## V. Tạo tài nguyên mới (POST)

Để tạo một khu vực mới, chúng ta cần một biểu mẫu để người dùng nhập thông tin và sau đó gửi dữ liệu này đến API.

### 1. Tạo ViewModel và Request DTO cho việc thêm mới

Chúng ta sẽ tạo một ViewModel riêng cho form thêm mới và một Request DTO để gửi đến API. ViewModel giúp xử lý validation và hiển thị lỗi trên UI, trong khi Request DTO là đối tượng mà API mong đợi.

1.  Trong thư mục `Models` của dự án `NZWalks.UI`, tạo một lớp mới tên `AddRegionViewModel.cs`.

    ```csharp
    // NZWalks.UI/Models/AddRegionViewModel.cs
    using System.ComponentModel.DataAnnotations;

    namespace NZWalks.UI.Models
    {
        public class AddRegionViewModel
        {
            [Required(ErrorMessage = "Mã vùng là bắt buộc.")]
            [MinLength(2, ErrorMessage = "Mã vùng phải có ít nhất 2 ký tự.")]
            [MaxLength(3, ErrorMessage = "Mã vùng không được quá 3 ký tự.")]
            public string Code { get; set; } = string.Empty;

            [Required(ErrorMessage = "Tên vùng là bắt buộc.")]
            [MaxLength(100, ErrorMessage = "Tên vùng không được quá 100 ký tự.")]
            public string Name { get; set; } = string.Empty;

            public string? RegionImageUrl { get; set; }
        }
    }
    ```

2.  Trong thư mục `Models/DTO` của dự án `NZWalks.UI`, tạo một lớp mới tên `AddRegionRequestDto.cs`. (Lưu ý: API của bạn có thể đã có lớp này, nhưng việc tạo một bản sao trong UI giúp tách biệt và kiểm soát).

    ```csharp
    // NZWalks.UI/Models/DTO/AddRegionRequestDto.cs
    namespace NZWalks.UI.Models.DTO
    {
        public class AddRegionRequestDto
        {
            public string Code { get; set; } = string.Empty;
            public string Name { get; set; } = string.Empty;
            public string? RegionImageUrl { get; set; }
        }
    }
    ```

### 2. Tạo Action Method `Add` (GET và POST)

1.  Trong `RegionsController.cs`, thêm phương thức `Add` (HTTP GET) để hiển thị form và phương thức `Add` (HTTP POST) để xử lý dữ liệu gửi lên.

    ```csharp
    // NZWalks.UI/Controllers/RegionsController.cs
    // ... (trong RegionsController)
    using NZWalks.UI.Models; // Để dùng AddRegionViewModel

    [HttpGet]
    public IActionResult Add()
    {
        return View();
    }

    [HttpPost]
    public async Task<IActionResult> Add(AddRegionViewModel addRegionViewModel)
    {
        if (!ModelState.IsValid)
        {
            // Nếu có lỗi validation từ ViewModel, trả về view với lỗi
            TempData["ErrorMessage"] = "Vui lòng kiểm tra lại thông tin nhập liệu.";
            return View(addRegionViewModel); 
        }

        // Chuyển đổi ViewModel sang DTO mà API mong đợi
        var addRegionRequestDto = new AddRegionRequestDto
        {
            Code = addRegionViewModel.Code,
            Name = addRegionViewModel.Name,
            RegionImageUrl = addRegionViewModel.RegionImageUrl
        };

        var response = await _regionService.AddRegionAsync(addRegionRequestDto);

        if (response != null)
        {
            TempData["SuccessMessage"] = "Khu vực đã được thêm thành công!";
            return RedirectToAction("Index"); // Chuyển hướng về trang danh sách
        }
        else
        {
            TempData["ErrorMessage"] = "Không thể thêm khu vực. Vui lòng thử lại.";
            return View(addRegionViewModel); // Trả về view với dữ liệu đã nhập nếu có lỗi
        }
    }
    ```

### 3. Tạo View `Add.cshtml`

1.  Trong thư mục `Views/Regions`, nhấp chuột phải, chọn `Add` -> `View` -> `Razor View - Empty`.
2.  Đặt tên View là `Add.cshtml`.
3.  Thêm code sau để tạo form thêm khu vực mới:

    ```html
    @model NZWalks.UI.Models.AddRegionViewModel

    @{
        ViewData["Title"] = "Add New Region";
    }

    <h1 class="mt-3">Add New Region</h1>

    @if (TempData["ErrorMessage"] != null)
    {
        <div class="alert alert-danger" role="alert">
            @TempData["ErrorMessage"]
        </div>
    }

    <form method="post">
        <div class="mt-3">
            <label for="Code" class="form-label">Region Code</label>
            <input type="text" class="form-control" id="Code" asp-for="Code" />
            <span asp-validation-for="Code" class="text-danger"></span>
        </div>

        <div class="mt-3">
            <label for="Name" class="form-label">Region Name</label>
            <input type="text" class="form-control" id="Name" asp-for="Name" />
            <span asp-validation-for="Name" class="text-danger"></span>
        </div>

        <div class="mt-3">
            <label for="RegionImageUrl" class="form-label">Region Image URL</label>
            <input type="text" class="form-control" id="RegionImageUrl" asp-for="RegionImageUrl" />
            <span asp-validation-for="RegionImageUrl" class="text-danger"></span>
        </div>

        <div class="mt-3">
            <button type="submit" class="btn btn-primary">Save</button>
        </div>
    </form>
    ```

## VI. Cập nhật tài nguyên (PUT)

Để cập nhật một khu vực, chúng ta sẽ sử dụng form `Edit.cshtml` đã điền sẵn dữ liệu. Khi người dùng thay đổi thông tin và nhấp "Save", dữ liệu sẽ được gửi đến API thông qua yêu cầu HTTP PUT.

### 1. Tạo Request DTO cho việc cập nhật

Tương tự như thêm mới, chúng ta tạo một DTO riêng cho yêu cầu cập nhật.

1.  Trong thư mục `Models/DTO` của dự án `NZWalks.UI`, tạo một lớp mới tên `UpdateRegionRequestDto.cs`.

    ```csharp
    // NZWalks.UI/Models/DTO/UpdateRegionRequestDto.cs
    namespace NZWalks.UI.Models.DTO
    {
        public class UpdateRegionRequestDto
        {
            public string Code { get; set; } = string.Empty;
            public string Name { get; set; } = string.Empty;
            public string? RegionImageUrl { get; set; }
        }
    }
    ```

### 2. Cập nhật Action Method `Edit` (POST)

1.  Trong `RegionsController.cs`, thêm phương thức `Edit` (HTTP POST) để xử lý dữ liệu cập nhật:

    ```csharp
    // NZWalks.UI/Controllers/RegionsController.cs
    // ... (trong RegionsController)
    using NZWalks.UI.Models.DTO; // Để dùng UpdateRegionRequestDto

    [HttpPost]
    public async Task<IActionResult> Edit(RegionDto request) // Nhận RegionDto từ form
    {
        if (!ModelState.IsValid)
        {
            // Nếu có lỗi validation từ Data Annotations trên RegionDto, trả về view với lỗi
            TempData["ErrorMessage"] = "Vui lòng kiểm tra lại thông tin cập nhật.";
            return View(request);
        }

        // Chuyển đổi RegionDto sang UpdateRegionRequestDto mà API mong đợi
        var updateRegionRequestDto = new UpdateRegionRequestDto
        {
            Code = request.Code,
            Name = request.Name,
            RegionImageUrl = request.RegionImageUrl
        };

        var response = await _regionService.UpdateRegionAsync(request.Id, updateRegionRequestDto);

        if (response != null)
        {
            TempData["SuccessMessage"] = "Khu vực đã được cập nhật thành công!";
            return RedirectToAction("Index"); // Chuyển hướng về trang danh sách
        }
        else
        {
            TempData["ErrorMessage"] = "Không thể cập nhật khu vực. Vui lòng thử lại.";
            return View(request); // Trả về view với dữ liệu đã nhập nếu có lỗi
        }
    }
    ```

> [!TIP]
> **Sử dụng `asp-for` và `asp-route-id`**:
> Trong `Edit.cshtml`, chúng ta sử dụng `asp-for="Id"` để ràng buộc trường `Id` với thuộc tính `Id` của `RegionDto`. Khi form được submit, toàn bộ `RegionDto` sẽ được gửi lên phương thức `Edit` (POST). ASP.NET Core MVC Model Binding sẽ tự động ánh xạ các trường trong form HTML sang các thuộc tính tương ứng của đối tượng `RegionDto` được truyền vào phương thức `Edit`.

## VII. Xóa tài nguyên (DELETE)

Để xóa một khu vực, chúng ta sẽ thêm một nút "Delete" vào trang chỉnh sửa. Khi nhấp vào, yêu cầu HTTP DELETE sẽ được gửi đến API.

### 1. Thêm nút "Delete" vào trang chỉnh sửa

Chúng ta đã thêm nút "Delete" vào `Edit.cshtml` ở bước trên:

```html
<button type="submit" class="btn btn-danger" 
        asp-controller="Regions" asp-action="Delete" asp-route-id="@Model.Id">Delete</button>
```

> [!NOTE]
> Để nút "Delete" hoạt động đúng cách trong một ứng dụng MVC truyền thống (sử dụng form POST), chúng ta định tuyến yêu cầu POST của nút này đến phương thức `Delete` trong `RegionsController`, đồng thời truyền `Id` qua route data (`asp-route-id`).
>
> **Lưu ý về RESTful DELETE**: Theo đúng ngữ nghĩa REST, thao tác xóa nên được thực hiện bằng phương thức HTTP `DELETE`. Trong môi trường trình duyệt, việc gửi yêu cầu `DELETE` trực tiếp từ form HTML không được hỗ trợ. Các cách tiếp cận phổ biến để thực hiện HTTP `DELETE` là:
> 1.  **Sử dụng JavaScript/AJAX**: Gửi yêu cầu `DELETE` không đồng bộ từ client.
> 2.  **Sử dụng form POST với trường ẩn `_method`**: Một số framework hỗ trợ "method spoofing" để biến yêu cầu POST thành DELETE.
> 3.  **Như ví dụ hiện tại**: Sử dụng một yêu cầu `POST` đến một endpoint MVC có tên `Delete` và bên trong đó, gọi API backend bằng `HttpClient.DeleteAsync()`. Cách này đơn giản và phổ biến trong các ứng dụng MVC.

### 2. Tạo Action Method `Delete` (POST)

1.  Trong `RegionsController.cs`, thêm phương thức `Delete` (HTTP POST) để xử lý yêu cầu xóa:

    ```csharp
    // NZWalks.UI/Controllers/RegionsController.cs
    // ... (trong RegionsController)

    [HttpPost]
    public async Task<IActionResult> Delete(RegionDto request) // Nhận RegionDto từ form (chỉ cần Id)
    {
        var isDeleted = await _regionService.DeleteRegionAsync(request.Id);

        if (isDeleted)
        {
            TempData["SuccessMessage"] = "Khu vực đã được xóa thành công!";
            return RedirectToAction("Index"); // Chuyển hướng về trang danh sách
        }
        else
        {
            TempData["ErrorMessage"] = "Không thể xóa khu vực. Vui lòng thử lại.";
            // Nếu có lỗi, có thể chuyển hướng lại trang chỉnh sửa hoặc hiển thị lỗi
            return RedirectToAction("Edit", new { id = request.Id }); 
        }
    }
    ```

> [!CAUTION]
> **Xử lý lỗi và xác nhận xóa**:
> Trong ứng dụng thực tế, bạn nên có một hộp thoại xác nhận (confirmation dialog) trước khi thực hiện thao tác xóa để tránh xóa nhầm dữ liệu. Điều này thường được thực hiện bằng JavaScript để hiển thị một modal hoặc alert. Ngoài ra, việc xử lý lỗi cần chi tiết hơn, ví dụ như log lỗi và hiển thị thông báo cụ thể cho người dùng.

## VIII. Vibe Coding và Antigravity IDE trong phát triển ứng dụng Client

Trong quá trình xây dựng ứng dụng MVC client tiêu thụ API này, bạn không chỉ học về cú pháp và cấu trúc mà còn cần phát triển tư duy để giải quyết vấn đề một cách hiệu quả. Đây chính là lúc "Vibe Coding" và một hệ thống AI như "Antigravity IDE" phát huy sức mạnh.

### 1. Tư duy Vibe Coding khi tiêu thụ API

Vibe Coding là việc hiểu được "linh hồn" của vấn đề, nắm bắt được ý định và luồng dữ liệu, thay vì chỉ tập trung vào từng dòng code riêng lẻ. Khi tiêu thụ API, Vibe Coding giúp bạn:

*   **Hiểu rõ "Hợp đồng" API**: Trước khi viết bất kỳ dòng code nào, hãy hình dung rõ ràng API đang cung cấp gì (các endpoint, phương thức HTTP, cấu trúc Request/Response DTO, các mã trạng thái lỗi). Bạn "cảm nhận" được API sẽ phản hồi như thế nào trong từng trường hợp.
*   **Phác thảo luồng dữ liệu**: Từ khi người dùng nhập liệu trên form, dữ liệu sẽ đi qua ViewModel, được chuyển đổi thành Request DTO, gửi đến API, API xử lý và trả về Response DTO, sau đó Response DTO được ánh xạ lại để hiển thị trên View. Việc hình dung toàn bộ chu trình này giúp bạn thiết kế các lớp và phương thức một cách logic.
*   **Dự đoán và xử lý lỗi**: Vibe Coding giúp bạn "cảm nhận" các điểm có thể xảy ra lỗi (mất kết nối mạng, API không khả dụng, dữ liệu không hợp lệ, lỗi server 5xx) và chủ động thiết kế các cơ chế `try-catch` và thông báo lỗi thân thiện cho người dùng.
*   **Thiết kế cấu trúc module**: Nhận ra rằng các cuộc gọi API nên được trừu tượng hóa vào một lớp dịch vụ riêng biệt (`RegionService`) để giữ cho Controller gọn gàng và dễ bảo trì.

### 2. Antigravity IDE: Trợ thủ đắc lực cho Vibe Coding

Antigravity IDE, với khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file, và lập kế hoạch tự động, là một công cụ lý tưởng để hiện thực hóa tư duy Vibe Coding trong việc phát triển ứng dụng client tiêu thụ API:

*   **Phân tích và Lập kế hoạch tự động**:
    *   **Đọc Swagger/OpenAPI**: Antigravity có thể sử dụng subagent trình duyệt để truy cập Swagger UI của `NZWalks.API` (ví dụ: `https://localhost:7001/swagger/index.html`). Nó sẽ tự động phân tích các endpoint, phương thức HTTP, và cấu trúc Request/Response DTO.
    *   **Đề xuất cấu trúc dự án**: Dựa trên phân tích, Antigravity có thể đề xuất tạo các thư mục `Models/DTO`, `Models`, `Configuration`, `Services/Interfaces`, `Services` trong dự án `NZWalks.UI`.
    *   **Lập kế hoạch triển khai**: "Để tiêu thụ API này, tôi cần: 1. Tạo DTOs tương ứng. 2. Cấu hình `IHttpClientFactory` với `BaseUrl`. 3. Xây dựng `RegionService` để đóng gói logic API. 4. Tạo `RegionsController` và các Action Methods. 5. Thiết kế các Views hiển thị và tương tác." Antigravity sẽ tự động tạo ra một kế hoạch chi tiết như vậy.

*   **Tạo mã tự động (Code Scaffolding) thông minh**:
    *   **Sinh DTOs**: Dựa trên schema JSON từ Swagger, Antigravity có thể tự động sinh ra các lớp `RegionDto`, `AddRegionRequestDto`, `UpdateRegionRequestDto` trong thư mục `Models/DTO` với các thuộc tính chính xác.
    *   **Khởi tạo Service Layer**: Antigravity có thể tạo file `IRegionService.cs` và `RegionService.cs` với các phương thức CRUD cơ bản (ví dụ: `GetAllRegionsAsync`, `GetRegionByIdAsync`, v.v.) và boilerplate `HttpClient` calls, bao gồm `try-catch` và logging.
    *   **Tạo Controller và Action Methods**: Sinh ra `RegionsController.cs` với constructor inject `IRegionService` và các Action Method rỗng `Index`, `Add`, `Edit`, `Delete` đã được định tuyến sẵn.
    *   **Khởi tạo Views**: Tạo các file `.cshtml` cơ bản (`Index.cshtml`, `Add.cshtml`, `Edit.cshtml`) với các thẻ HTML cần thiết và `asp-for` helpers để ràng buộc với ViewModel/DTO.

*   **Kiểm thử và Gỡ lỗi tự động**:
    *   **Kiểm tra API Endpoint**: Antigravity có thể tự chạy script ngầm để gửi các yêu cầu `curl` hoặc `HttpClient` đến các endpoint API đang chạy (`NZWalks.API`) để xác minh chúng hoạt động đúng, trả về dữ liệu mong muốn, và xử lý lỗi.
    *   **Phân tích phản hồi JSON**: Nếu API trả về JSON không đúng định dạng hoặc lỗi, Antigravity có thể phân tích và chỉ ra vấn đề, thậm chí đề xuất sửa đổi DTO trong `NZWalks.UI` hoặc cấu trúc phản hồi của API.
    *   **Đề xuất xử lý lỗi**: Khi phát hiện `HttpRequestException` hoặc `JsonException` trong quá trình chạy thử nghiệm hoặc phân tích mã, Antigravity có thể đề xuất các khối `try-catch` chi tiết hơn, ghi log lỗi, và hiển thị thông báo thân thiện cho người dùng thông qua `TempData`.

*   **Tối ưu hóa và Đảm bảo Best Practices**:
    *   **Đề xuất `IHttpClientFactory`**: Nếu bạn vô tình sử dụng `new HttpClient()`, Antigravity sẽ tự động cảnh báo và đề xuất refactor để sử dụng `IHttpClientFactory` với các cấu hình tối ưu.
    *   **Quản lý cấu hình**: Antigravity có thể tự động thêm `NZWalksApiSettings` vào `appsettings.json`, tạo lớp `NZWalksApiSettings.cs` và cập nhật `Program.cs` để sử dụng `IOptions<NZWalksApiSettings>`, đảm bảo cấu hình strongly typed.
    *   **Cải thiện Validation**: Nhắc nhở thêm Data Annotations vào ViewModels/DTOs trong UI để có validation phía client và server hiệu quả.

Bằng cách áp dụng tư duy Vibe Coding và tận dụng sức mạnh của Antigravity IDE, bạn có thể xây dựng các ứng dụng client tương tác API một cách nhanh chóng, chính xác và tuân thủ các best practices, biến quá trình phát triển trở nên hiệu quả và ít lỗi hơn.

## Tóm tắt Chương

Trong chương này, chúng ta đã khám phá chi tiết cách xây dựng một ứng dụng ASP.NET Core MVC có khả năng tiêu thụ một RESTful Web API. Dưới đây là những điểm chính đã được trình bày:

*   **Nền tảng vững chắc**: Nắm vững các nguyên tắc cơ bản của kiến trúc REST và cách các phương thức HTTP (GET, POST, PUT, DELETE) tương ứng với các thao tác CRUD trên tài nguyên.
*   **Thiết lập môi trường hiệu quả**: Hướng dẫn tạo một dự án ASP.NET Core MVC mới, cấu hình để chạy đồng thời với dự án Web API, và nhấn mạnh tầm quan trọng của việc xử lý bảo mật trong ứng dụng thực tế.
*   **Cơ chế giao tiếp tối ưu**:
    *   Hiểu rõ vấn đề của việc khởi tạo `HttpClient` trực tiếp và tầm quan trọng của `IHttpClientFactory` trong việc quản lý tài nguyên và kết nối.
    *   Cấu hình `IHttpClientFactory` sử dụng `IOptions` và cấu hình strongly typed từ `appsettings.json`.
    *   **Thực hành tốt nhất**: Xây dựng một lớp dịch vụ (`RegionService`) để trừu tượng hóa các cuộc gọi API, giúp Controller gọn gàng và dễ bảo trì, đồng thời đăng ký nó vào Dependency Injection.
*   **Thực hiện đầy đủ các thao tác CRUD**:
    *   **Tiêu thụ dữ liệu (GET)**: Lấy danh sách tất cả các tài nguyên và thông tin chi tiết của một tài nguyên cụ thể, hiển thị chúng trên View.
    *   **Tạo tài nguyên mới (POST)**: Xây dựng form nhập liệu, sử dụng ViewModel và Request DTO, sau đó gửi dữ liệu JSON đến API.
    *   **Cập nhật tài nguyên (PUT)**: Sử dụng form chỉnh sửa đã có, gửi DTO đã cập nhật đến API.
    *   **Xóa tài nguyên (DELETE)**: Thêm nút xóa và gửi yêu cầu DELETE đến API (thông qua POST trong MVC).
*   **Xử lý dữ liệu và lỗi**: Cách tuần tự hóa/giải tuần tự hóa JSON bằng `System.Text.Json`, kiểm tra `EnsureSuccessStatusCode()`, và sử dụng khối `try-catch` trong lớp dịch vụ để xử lý các lỗi HTTP hoặc lỗi giải tuần tự hóa JSON, kèm theo ghi log chi tiết.
*   **Tư duy Vibe Coding và Antigravity IDE**: Khám phá cách Vibe Coding giúp bạn hình dung toàn bộ luồng tương tác API và cách Antigravity IDE có thể hỗ trợ mạnh mẽ từ việc phân tích API, tạo mã tự động, kiểm thử, gỡ lỗi cho đến việc đảm bảo các best practices được áp dụng.

Qua chương này, bạn đã có một nền tảng vững chắc để xây dựng các ứng dụng ASP.NET Core MVC có khả năng tương tác mạnh mẽ với các RESTful Web API, mở ra cánh cửa cho việc phát triển các ứng dụng phân tán và hiện đại một cách chuyên nghiệp và hiệu quả.

<!-- REVIEWED_BY_AGENT -->
