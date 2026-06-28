# Phần 13: Ghi nhật ký & Xử lý ngoại lệ toàn cục

## Giới thiệu: Nâng cao Khả năng Quan sát và Phục hồi cho RESTful API

Trong thế giới phát triển phần mềm hiện đại, đặc biệt là với các hệ thống phân tán như RESTful Web API, việc đảm bảo ứng dụng không chỉ hoạt động đúng mà còn có thể được theo dõi, chẩn đoán và phục hồi một cách hiệu quả là yếu tố then chốt. Chương này sẽ tập trung vào hai trụ cột quan trọng giúp đạt được mục tiêu này: **Ghi nhật ký (Logging)** và **Xử lý ngoại lệ toàn cục (Global Exception Handling)**.

Ghi nhật ký là đôi mắt của nhà phát triển trong môi trường runtime. Nó cung cấp luồng thông tin liên tục về các sự kiện, trạng thái và hành vi của ứng dụng, từ đó giúp gỡ lỗi, giám sát hiệu suất, và phân tích các vấn đề tiềm ẩn. Chúng ta sẽ khám phá Serilog, một thư viện ghi nhật ký cấu trúc (structured logging) mạnh mẽ, để không chỉ ghi lại các sự kiện mà còn thu thập dữ liệu có ý nghĩa, dễ dàng truy vấn và phân tích.

Bên cạnh đó, không một ứng dụng nào hoàn hảo và ngoại lệ là điều khó tránh khỏi. Cách chúng ta xử lý các lỗi không mong muốn quyết định độ tin cậy và trải nghiệm người dùng của API. Thay vì phân tán logic xử lý lỗi khắp mã nguồn bằng các khối `try-catch` lặp lại, chúng ta sẽ xây dựng một cơ chế xử lý ngoại lệ toàn cục bằng Middleware tùy chỉnh. Phương pháp này không chỉ tinh gọn mã nguồn mà còn đảm bảo mọi lỗi đều được ghi lại một cách nhất quán và phản hồi về client theo một định dạng chuẩn, an toàn thông tin.

Mục tiêu chính của chương này là trang bị cho bạn kiến thức và kỹ năng thực tiễn để:
*   Nắm vững vai trò và lợi ích của ghi nhật ký cấu trúc trong phát triển API.
*   Cấu hình và sử dụng Serilog để ghi nhật ký ra nhiều đích đến (sinks) khác nhau như console và tệp.
*   Hiểu và áp dụng các cấp độ ghi nhật ký để kiểm soát luồng thông tin.
*   Đánh giá tầm quan trọng của xử lý ngoại lệ toàn cục và nhược điểm của các phương pháp cục bộ.
*   Xây dựng và tích hợp một Middleware xử lý ngoại lệ toàn cục tùy chỉnh vào pipeline của ASP.NET Core.
*   Thiết kế phản hồi lỗi nhất quán, an toàn thông tin cho client, đồng thời ghi lại chi tiết lỗi cho mục đích gỡ lỗi và phân tích.

Chúng ta sẽ tiếp cận các kỹ thuật này trong bối cảnh kiến trúc ASP.NET Core Web API, tận dụng triệt để Dependency Injection và các nguyên tắc thiết kế phần mềm tốt để xây dựng một hệ thống mạnh mẽ và dễ bảo trì.

---

## 1. Ghi nhật ký chuyên sâu với Serilog trong ASP.NET Core

Ghi nhật ký là một công cụ chẩn đoán không thể thiếu, cung cấp cái nhìn sâu sắc về hoạt động bên trong của ứng dụng. Nó giúp nhà phát triển hiểu điều gì đang xảy ra, khi nào và tại sao, đặc biệt hữu ích trong các môi trường sản xuất nơi việc gỡ lỗi trực tiếp là không khả thi.

### 1.1. Nền tảng ghi nhật ký trong .NET Core

ASP.NET Core cung cấp một hệ thống ghi nhật ký tích hợp sẵn, linh hoạt và có khả năng mở rộng. Hệ thống này được xây dựng xung quanh các giao diện chính:
*   `ILogger`: Giao diện chính để ghi nhật ký các thông báo.
*   `ILogger<T>`: Một phiên bản cụ thể của `ILogger` được tham số hóa với kiểu `T`, thường là tên của lớp đang ghi nhật ký. Điều này giúp hệ thống ghi nhật ký biết được nguồn gốc của thông báo, hữu ích cho việc lọc và phân tích.
*   `ILoggerFactory`: Dùng để tạo các thể hiện của `ILogger`.
*   `ILoggingBuilder`: Cung cấp các phương thức mở rộng để thêm các nhà cung cấp ghi nhật ký (logging providers) khác nhau.

**Cơ chế hoạt động "Under the Hood":**
Khi bạn inject `ILogger<T>` vào một constructor, Dependency Injection container của ASP.NET Core sẽ sử dụng `ILoggerFactory` để tạo ra một thể hiện của `ILogger<T>`. `ILoggerFactory` này đã được cấu hình với một hoặc nhiều **nhà cung cấp ghi nhật ký (logging providers)**. Mỗi nhà cung cấp chịu trách nhiệm đẩy thông báo nhật ký đến một đích đến cụ thể (ví dụ: console, debug window, EventSource). Hệ thống này cho phép bạn dễ dàng hoán đổi hoặc kết hợp các nhà cung cấp khác nhau mà không cần thay đổi code gọi `ILogger`.

#### Các cấp độ ghi nhật ký chuẩn

Để quản lý lượng thông tin được ghi, hầu hết các hệ thống ghi nhật ký đều phân loại thông báo theo mức độ nghiêm trọng. Việc này cho phép bạn cấu hình để chỉ hiển thị những thông tin phù hợp với ngữ cảnh (ví dụ: chi tiết cao trong môi trường phát triển, chỉ lỗi nghiêm trọng trong môi trường sản xuất). Các cấp độ phổ biến, theo thứ tự từ ít nghiêm trọng nhất đến nghiêm trọng nhất, bao gồm:

*   **Trace (Theo dõi):** Thông tin rất chi tiết về luồng thực thi, thường chỉ dùng cho gỡ lỗi chuyên sâu nội bộ. Có thể chứa dữ liệu nhạy cảm.
*   **Debug (Gỡ lỗi):** Thông tin gỡ lỗi và chẩn đoán hữu ích cho nhà phát triển trong quá trình phát triển.
*   **Information (Thông tin):** Thông tin chung về luồng hoạt động của ứng dụng, các sự kiện quan trọng (ví dụ: "Yêu cầu HTTP đã hoàn thành", "Người dùng {UserId} đăng nhập").
*   **Warning (Cảnh báo):** Một sự kiện bất thường hoặc không mong muốn đã xảy ra nhưng không làm gián đoạn hoạt động chính của ứng dụng (ví dụ: "Dữ liệu đầu vào không hợp lệ, đã sử dụng giá trị mặc định").
*   **Error (Lỗi):** Một lỗi đã xảy ra và làm gián đoạn một phần chức năng của ứng dụng (ví dụ: "Không thể kết nối đến dịch vụ {ServiceName}").
*   **Critical (Nghiêm trọng):** Một lỗi nghiêm trọng đã xảy ra, làm cho ứng dụng hoặc một thành phần quan trọng không thể hoạt động (ví dụ: "Ứng dụng bị sập", "Lỗi bộ nhớ không thể phục hồi").

### 1.2. Serilog: Giải pháp ghi nhật ký có cấu trúc mạnh mẽ

Mặc dù hệ thống ghi nhật ký mặc định của .NET Core là đủ cho nhiều trường hợp, Serilog nổi lên như một lựa chọn ưu việt của bên thứ ba, đặc biệt khi yêu cầu khả năng ghi nhật ký nâng cao.

**Ưu điểm vượt trội của Serilog:**

*   **Ghi nhật ký có cấu trúc (Structured Logging):** Đây là ưu điểm lớn nhất. Thay vì chỉ ghi các chuỗi văn bản đơn thuần, Serilog cho phép bạn ghi các đối tượng dữ liệu dưới dạng các thuộc tính (properties) có cấu trúc. Khi được lưu trữ trong các định dạng như JSON, Elasticsearch, hoặc cơ sở dữ liệu, các thuộc tính này có thể được tìm kiếm, lọc, tổng hợp và phân tích một cách hiệu quả hơn rất nhiều so với văn bản thuần túy. Ví dụ, thay vì `_logger.LogInformation("Người dùng 123 đăng nhập.")`, bạn có thể ghi `_logger.LogInformation("Người dùng {UserId} đăng nhập.", userId);`, và `UserId` sẽ trở thành một thuộc tính riêng biệt trong log.
*   **Đa dạng Sinks (Đích đến):** Serilog hỗ trợ một hệ sinh thái phong phú các "sinks" (tương tự như logging providers) để lưu trữ nhật ký. Ngoài console và tệp, bạn có thể dễ dàng cấu hình Serilog để ghi nhật ký vào cơ sở dữ liệu (SQL Server, MongoDB), các dịch vụ đám mây (Azure Application Insights, AWS CloudWatch), các hệ thống quản lý nhật ký tập trung (Elasticsearch, Seq, Splunk), v.v.
*   **Cấu hình linh hoạt:** Serilog có thể được cấu hình thông qua mã nguồn hoặc thông qua tệp cấu hình (ví dụ: `appsettings.json`), cho phép điều chỉnh hành vi ghi nhật ký mà không cần biên dịch lại ứng dụng.
*   **Hiệu suất cao:** Được thiết kế để có hiệu suất tốt, phù hợp với các ứng dụng có tải cao.

> [!NOTE]
> Trong khóa học này, chúng ta sẽ tích hợp Serilog để tận dụng khả năng ghi nhật ký có cấu trúc và hệ sinh thái sinks phong phú của nó, mang lại lợi ích lâu dài cho việc gỡ lỗi và giám sát ứng dụng.

### 1.3. Cài đặt và Cấu hình Serilog

Để tích hợp Serilog vào ứng dụng ASP.NET Core Web API, chúng ta cần cài đặt các gói NuGet và cấu hình chúng trong `Program.cs`.

#### 1.3.1. Cài đặt các gói NuGet cần thiết

Sử dụng NuGet Package Manager hoặc .NET CLI để cài đặt các gói sau vào dự án của bạn:

1.  **`Serilog`**: Gói cốt lõi của Serilog, cung cấp các API ghi nhật ký cơ bản.
2.  **`Serilog.AspNetCore`**: Gói tích hợp Serilog với hệ thống `ILogger` của ASP.NET Core, cho phép Serilog hoạt động như một nhà cung cấp ghi nhật ký.
3.  **`Serilog.Sinks.Console`**: Cho phép Serilog ghi nhật ký ra cửa sổ giao diện điều khiển (Console).
4.  **`Serilog.Sinks.File`**: Cho phép Serilog ghi nhật ký ra tệp văn bản.

```bash
dotnet add package Serilog
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

#### 1.3.2. Cấu hình Serilog trong `Program.cs`

Sau khi cài đặt gói, chúng ta sẽ cấu hình Serilog trong tệp `Program.cs`. Đây là nơi chúng ta định nghĩa cách Serilog sẽ hoạt động, bao gồm nơi nó sẽ ghi nhật ký và mức độ chi tiết.

```csharp
using Serilog; // Đảm bảo có câu lệnh using này

var builder = WebApplication.CreateBuilder(args);

// --- Cấu hình Serilog ---
// Bước 1: Tạo một đối tượng LoggerConfiguration.
// Đây là nơi chúng ta định nghĩa các quy tắc ghi nhật ký.
var logger = new LoggerConfiguration()
    // Bước 2: Thiết lập mức ghi nhật ký tối thiểu.
    // Mọi thông báo có cấp độ thấp hơn (Trace, Debug) sẽ bị bỏ qua.
    // Các cấp độ cao hơn (Warning, Error, Critical) sẽ luôn được ghi.
    // Ví dụ: .MinimumLevel.Debug() sẽ ghi cả Debug và các cấp cao hơn.
    .MinimumLevel.Information() 
    
    // Bước 3: Định cấu hình đích đến (sink) cho nhật ký.
    // Ghi nhật ký ra Console. Serilog sẽ tự động định dạng đẹp mắt.
    .WriteTo.Console()
    
    // Ghi nhật ký ra tệp văn bản.
    // "Logs/nz-walks-log-.txt" định nghĩa đường dẫn và tiền tố tên tệp.
    // "{Date}" là một placeholder sẽ được Serilog thay thế bằng ngày hiện tại.
    // Ví dụ: nz-walks-log-20231027.txt
    // RollingInterval.Day nghĩa là một tệp nhật ký mới sẽ được tạo mỗi ngày.
    // Các tùy chọn khác: .Minute, .Hour, .Month, .Year, .Infinite.
    .WriteTo.File("Logs/nz-walks-log-.txt", RollingInterval.Day)
    
    // Bước 4: Hoàn tất cấu hình và tạo Logger.
    .CreateLogger();

// Bước 5: Tích hợp Serilog vào hệ thống ghi nhật ký của ASP.NET Core.
// Xóa tất cả các nhà cung cấp ghi nhật ký mặc định của ASP.NET Core.
builder.Logging.ClearProviders();
// Thêm Serilog làm nhà cung cấp ghi nhật ký chính.
// Điều này cho phép chúng ta tiếp tục sử dụng ILogger<T> tiêu chuẩn của .NET Core,
// nhưng Serilog sẽ là engine xử lý việc ghi nhật ký bên dưới.
builder.Logging.AddSerilog(logger);
// --- Kết thúc cấu hình Serilog ---

// Thêm dịch vụ vào bộ chứa.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Cấu hình đường ống xử lý yêu cầu HTTP.
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
> **Tạo thư mục `Logs`:** Để Serilog có thể ghi nhật ký ra tệp, bạn nên tạo một thư mục có tên `Logs` (hoặc bất kỳ tên nào bạn cấu hình) ở thư mục gốc của dự án. Nếu thư mục không tồn tại, Serilog sẽ tự động tạo nó, nhưng việc tạo trước sẽ giúp bạn kiểm soát cấu trúc thư mục tốt hơn.

**Giải thích cấu hình chi tiết:**
*   `LoggerConfiguration()`: Khởi tạo đối tượng cấu hình Serilog.
*   `.MinimumLevel.Information()`: Đặt ngưỡng ghi nhật ký. Chỉ các thông báo có mức độ `Information` trở lên (gồm `Information`, `Warning`, `Error`, `Critical`) mới được xử lý. Các cấp độ thấp hơn như `Trace` và `Debug` sẽ bị bỏ qua, giúp giảm thiểu lượng dữ liệu nhật ký không cần thiết trong môi trường production.
*   `.WriteTo.Console()`: Đăng ký một sink để ghi nhật ký ra console. Serilog có một định dạng mặc định khá dễ đọc cho console.
*   `.WriteTo.File("Logs/nz-walks-log-.txt", RollingInterval.Day)`: Đăng ký một sink để ghi nhật ký vào tệp.
    *   `"Logs/nz-walks-log-.txt"`: Đường dẫn tương đối đến thư mục `Logs` và tiền tố tên tệp. Serilog sẽ tự động thêm ngày vào tên tệp.
    *   `RollingInterval.Day`: Chỉ định rằng một tệp nhật ký mới sẽ được tạo mỗi ngày. Điều này giúp quản lý kích thước tệp và dễ dàng lưu trữ/xóa các tệp cũ.
*   `.CreateLogger()`: Hoàn tất quá trình cấu hình và tạo ra một thể hiện của `ILogger` của Serilog.
*   `builder.Logging.ClearProviders()`: Xóa bỏ tất cả các nhà cung cấp ghi nhật ký mặc định mà ASP.NET Core tự động thêm vào (như ConsoleLogger, DebugLogger). Điều này quan trọng để tránh việc ghi nhật ký trùng lặp và đảm bảo Serilog là nhà cung cấp duy nhất.
*   `builder.Logging.AddSerilog(logger)`: Đăng ký Serilog làm nhà cung cấp ghi nhật ký chính cho hệ thống `ILogger` của ASP.NET Core. Từ giờ, mọi yêu cầu `ILogger<T>` sẽ được Serilog xử lý.

### 1.4. Sử dụng Serilog trong Controllers

Sau khi cấu hình Serilog, bạn có thể sử dụng nó trong bất kỳ lớp nào của ứng dụng thông qua Dependency Injection (DI) của ASP.NET Core.

#### 1.4.1. Inject `ILogger<T>` vào Controller

ASP.NET Core sử dụng giao diện `ILogger<T>` để cung cấp khả năng ghi nhật ký. `T` thường là tên của lớp mà bạn đang ghi nhật ký, giúp Serilog (hoặc bất kỳ nhà cung cấp ghi nhật ký nào) biết nguồn gốc của thông báo.

Ví dụ, trong một Controller như `AreaController`:

```csharp
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain; 
using System.Text.Json; // Để serialize đối tượng thành JSON khi ghi log

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("[controller]")] 
    public class AreaController : ControllerBase
    {
        private readonly ILogger<AreaController> _logger; // Khai báo ILogger
        // Giả sử có thêm các dịch vụ khác như IMapper, IAreaRepository

        public AreaController(ILogger<AreaController> logger /*, IMapper mapper, IAreaRepository areaRepository */)
        {
            _logger = logger;
            // _mapper = mapper;
            // _areaRepository = areaRepository;
        }

        [HttpGet]
        public IActionResult GetAllAreas()
        {
            // --- Ghi nhật ký thông tin khi phương thức bắt đầu ---
            // Sử dụng ghi nhật ký có cấu trúc: EndpointName là một thuộc tính riêng biệt.
            _logger.LogInformation("Endpoint {EndpointName} was invoked.", nameof(GetAllAreas));

            try
            {
                // Giả định truy vấn cơ sở dữ liệu hoặc logic nghiệp vụ
                var areas = new List<Area> // Dữ liệu giả định
                {
                    new Area { Id = Guid.NewGuid(), Code = "A1", Name = "Area One", AreaImageUrl = null },
                    new Area { Id = Guid.NewGuid(), Code = "A2", Name = "Area Two", AreaImageUrl = "https://example.com/a2.jpg" }
                };
                // var areas = _areaRepository.GetAll(); // Sử dụng repository thực tế

                // --- Ghi nhật ký dữ liệu trả về với cấu trúc ---
                // Serilog có thể ghi các đối tượng có cấu trúc.
                // Để dễ đọc trong log file, ta có thể serialize đối tượng thành JSON.
                _logger.LogInformation("Finished {EndpointName} request. Returned {AreaCount} areas: {AreasData}", 
                                       nameof(GetAllAreas), areas.Count, JsonSerializer.Serialize(areas));

                return Ok(areas);
            }
            catch (Exception ex)
            {
                // --- Ghi nhật ký lỗi và ngoại lệ ---
                // LogError có thể nhận đối tượng Exception trực tiếp để ghi stack trace chi tiết.
                // Thêm một ErrorId duy nhất để dễ dàng theo dõi lỗi trong hệ thống log.
                var errorId = Guid.NewGuid();
                _logger.LogError(ex, "ErrorId: {ErrorId} - An error occurred while fetching all areas in {EndpointName}.", 
                                 errorId, nameof(GetAllAreas));
                
                // Trong môi trường production, bạn KHÔNG NÊN trả về chi tiết lỗi cho client.
                return StatusCode(StatusCodes.Status500InternalServerError, new { ErrorId = errorId, Message = "An unexpected error occurred." });
            }
        }

        [HttpGet("{id:Guid}")]
        public IActionResult GetAreaById(Guid id)
        {
            // LogDebug chỉ hiển thị nếu MinimumLevel được đặt là Debug hoặc thấp hơn
            _logger.LogDebug("Attempting to retrieve area by ID: {AreaId}", id);

            if (id == Guid.Empty)
            {
                _logger.LogWarning("Attempted to retrieve area with an empty GUID. Client IP: {ClientIp}", HttpContext.Connection.RemoteIpAddress);
                return BadRequest("Area ID cannot be empty.");
            }

            // ... (Logic lấy Area theo ID)
            // Giả định không tìm thấy
            if (id == Guid.Parse("00000000-0000-0000-0000-000000000001")) // Giả lập ID không tồn tại
            {
                 _logger.LogInformation("Area with ID {AreaId} not found.", id);
                 return NotFound();
            }

            _logger.LogInformation("Successfully retrieved area with ID: {AreaId}", id);
            return Ok(new Area { Id = id, Code = "A3", Name = "Area Three", AreaImageUrl = null });
        }
    }
}
```

#### 1.4.2. Các phương thức ghi nhật ký và ghi nhật ký có cấu trúc

`ILogger<T>` cung cấp các phương thức tiện lợi để ghi nhật ký ở các cấp độ khác nhau. Quan trọng hơn, Serilog cho phép bạn truyền các đối số dưới dạng các thuộc tính để tận dụng ghi nhật ký có cấu trúc:

*   `_logger.LogTrace("...");`
*   `_logger.LogDebug("...");`
*   `_logger.LogInformation("Endpoint {EndpointName} completed in {ElapsedMs} ms.", nameof(GetAllAreas), stopwatch.ElapsedMilliseconds);`
    *   Ở đây, `EndpointName` và `ElapsedMs` sẽ trở thành các trường riêng biệt trong log, giúp bạn dễ dàng tìm kiếm tất cả các log từ một endpoint cụ thể hoặc lọc theo thời gian thực thi.
*   `_logger.LogWarning("...");`
*   `_logger.LogError(exception, "ErrorId: {ErrorId} - An error occurred: {Message}", errorId, exception.Message);`
    *   **Quan trọng:** Luôn truyền đối tượng `Exception` vào phương thức `LogError` hoặc `LogCritical`. Điều này cho phép Serilog thu thập thông tin chi tiết như stack trace, giúp việc gỡ lỗi hiệu quả hơn nhiều.

#### 1.4.3. Kiểm tra kết quả ghi nhật ký

Khi bạn chạy ứng dụng và gọi các API, bạn sẽ thấy các thông báo nhật ký xuất hiện trong:
*   **Cửa sổ Console** (trong Visual Studio, đây là cửa sổ Output hoặc cửa sổ Terminal nếu bạn chạy từ dòng lệnh).
*   **Tệp văn bản** trong thư mục `Logs` mà bạn đã cấu hình.

Hãy thử thay đổi `MinimumLevel` trong `Program.cs` thành `Warning` và chạy lại. Bạn sẽ thấy các thông báo `Information` và `Debug` không còn xuất hiện trong nhật ký, chỉ còn `Warning`, `Error` và `Critical`. Điều này minh họa cách các cấp độ ghi nhật ký giúp bạn kiểm soát lượng thông tin được ghi.

### 1.5. Tối ưu hóa ghi nhật ký và Vai trò của AI trong Vibe Coding

Ghi nhật ký không chỉ là việc ném các chuỗi văn bản vào một tệp. Với Serilog và tư duy "Vibe Coding", chúng ta có thể nâng tầm việc ghi nhật ký thành một công cụ chẩn đoán mạnh mẽ.

**Vibe Coding và Ghi nhật ký:**
"Vibe Coding" khuyến khích nhà phát triển không chỉ viết code hoạt động mà còn code *thể hiện ý định* và *dễ hiểu*. Trong ngữ cảnh ghi nhật ký, điều này có nghĩa là:
*   **Ghi nhật ký có ý nghĩa:** Mỗi thông báo log nên trả lời câu hỏi "Điều gì đang xảy ra ở đây?" và "Tại sao nó lại quan trọng?". Tránh các log chung chung không cung cấp ngữ cảnh.
*   **Ghi nhật ký ngữ cảnh:** Sử dụng ghi nhật ký có cấu trúc để thêm các thuộc tính quan trọng (ví dụ: `UserId`, `RequestId`, `TransactionId`, `EndpointName`) vào mọi thông báo. Điều này giúp bạn tái tạo "câu chuyện" của một yêu cầu hoặc một sự kiện cụ thể qua nhiều log entry khác nhau.
*   **Phân biệt cấp độ:** Sử dụng đúng cấp độ ghi nhật ký để phản ánh mức độ nghiêm trọng và tầm quan trọng của sự kiện. Điều này là cốt lõi để lọc hiệu quả.

**Antigravity IDE và Hỗ trợ Ghi nhật ký:**
Một hệ thống Agentic AI như Antigravity IDE có thể đóng vai trò cực kỳ quan trọng trong việc nâng cao chất lượng và hiệu quả của việc ghi nhật ký:

*   **Tự động tạo cấu hình Serilog:** Dựa trên yêu cầu của bạn (ví dụ: "ghi nhật ký ra console và tệp, mức độ Information, lưu log 7 ngày"), Antigravity có thể tự động tạo hoặc điều chỉnh cấu hình Serilog trong `Program.cs` và `appsettings.json`.
*   **Đề xuất vị trí và nội dung Log:** Khi bạn viết một phương thức, Antigravity có thể phân tích luồng logic và đề xuất các vị trí chiến lược để thêm các câu lệnh `_logger.LogInformation`, `_logger.LogDebug`, hoặc `_logger.LogWarning`, kèm theo các thuộc tính có cấu trúc phù hợp (ví dụ: "Thêm log ở đây để theo dõi giá trị của `inputParameter` sau khi xác thực").
*   **Đảm bảo ghi nhật ký có cấu trúc:** Antigravity có thể quét code và cảnh báo nếu bạn đang sử dụng ghi nhật ký chuỗi thuần túy ở những nơi cần ghi nhật ký có cấu trúc, thậm chí tự động chuyển đổi chúng sang định dạng Serilog với các placeholder `{}`.
*   **Phân tích nhật ký và cảnh báo:** Sau khi ứng dụng chạy, Antigravity có thể được kết nối với hệ thống log (ví dụ: đọc tệp log) để:
    *   Phát hiện các mẫu lỗi lặp lại.
    *   Cảnh báo về các cảnh báo (warnings) thường xuyên xảy ra.
    *   Đề xuất các điểm có thể tối ưu hóa hoặc nơi cần thêm log chi tiết hơn để hiểu rõ hành vi ứng dụng.
    *   Tự động liên kết `ErrorId` từ phản hồi API với chi tiết stack trace trong log file, giúp quá trình gỡ lỗi trở nên nhanh chóng và chính xác.
*   **Tạo báo cáo tổng hợp:** Antigravity có thể tổng hợp các log quan trọng, tạo ra các báo cáo tóm tắt về hiệu suất, lỗi, và các sự kiện đáng chú ý, giúp bạn có cái nhìn tổng quan về sức khỏe của API.

Bằng cách kết hợp tư duy Vibe Coding và sức mạnh của Antigravity IDE, việc ghi nhật ký sẽ không còn là một công việc thủ công tẻ nhạt mà trở thành một phần không thể thiếu, được tự động hóa và thông minh hóa trong quy trình phát triển.

---

## 2. Xử lý ngoại lệ toàn cục hiệu quả trong ASP.NET Core Web API

Ngoại lệ là một phần không thể tránh khỏi của quá trình phát triển phần mềm. Cách chúng ta quản lý và phản hồi các lỗi này có tác động trực tiếp đến độ tin cậy, bảo mật và trải nghiệm người dùng của API.

### 2.1. Nhược điểm của `try-catch` cục bộ và Sự cần thiết của xử lý toàn cục

Một phương pháp phổ biến nhưng kém hiệu quả để xử lý lỗi là sử dụng các khối `try-catch` trong từng phương thức hành động (action method) của Controller.

```csharp
// Ví dụ về try-catch cục bộ
[HttpGet]
public IActionResult GetAllWalks()
{
    try
    {
        // Giả lập một lỗi
        // throw new Exception("Lỗi kết nối cơ sở dữ liệu khi lấy danh sách Walks.");
        // var walks = _walkRepository.GetAll();
        // return Ok(_mapper.Map<List<WalkDto>>(walks));
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "An error occurred in GetAllWalks.");
        // Trả về phản hồi lỗi tùy chỉnh
        return StatusCode(StatusCodes.Status500InternalServerError, new { Message = "Đã xảy ra sự cố không mong muốn." });
    }
}
```

Cách tiếp cận này, mặc dù hoạt động, nhưng mang lại nhiều nhược điểm đáng kể:

*   **Lặp lại mã (Code Duplication):** Logic xử lý lỗi (ghi nhật ký, tạo phản hồi) sẽ phải được viết lại trong *mỗi* phương thức hành động có khả năng gây lỗi. Điều này làm cho mã nguồn trở nên dài dòng, khó đọc và khó hiểu.
*   **Khó bảo trì:** Nếu bạn cần thay đổi cách ghi nhật ký lỗi, định dạng phản hồi lỗi, hoặc thêm logic xử lý lỗi mới, bạn sẽ phải cập nhật ở rất nhiều nơi trong codebase.
*   **Phản hồi không nhất quán:** Các nhà phát triển khác nhau (hoặc ngay cả cùng một nhà phát triển ở các thời điểm khác nhau) có thể tạo ra các định dạng lỗi không đồng nhất cho client. Điều này gây khó khăn cho client trong việc xử lý và hiển thị lỗi.
*   **Dễ bỏ sót lỗi:** Nếu một nhà phát triển quên thêm `try-catch` vào một phương thức, một ngoại lệ chưa được xử lý có thể gây ra lỗi máy chủ mặc định của ASP.NET Core (thường là một trang lỗi HTML hoặc thông báo lỗi không thân thiện), không được ghi nhật ký đúng cách và có thể tiết lộ thông tin nhạy cảm.

**Lợi ích của xử lý ngoại lệ toàn cục:**

Xử lý ngoại lệ toàn cục giải quyết tất cả các vấn đề trên bằng cách tập trung logic xử lý lỗi vào một điểm duy nhất trong ứng dụng. Khi một ngoại lệ chưa được xử lý xảy ra ở bất kỳ đâu trong quá trình xử lý yêu cầu, nó sẽ được "bắt" bởi cơ chế toàn cục này.

Các lợi ích chính bao gồm:
*   **Tập trung logic:** Toàn bộ logic ghi nhật ký lỗi và tạo phản hồi lỗi nằm ở một vị trí duy nhất, dễ dàng quản lý.
*   **Phản hồi nhất quán:** Đảm bảo rằng mọi phản hồi lỗi từ API đều tuân theo một định dạng chuẩn, giúp client dễ dàng xử lý và hiển thị thông báo.
*   **Đơn giản hóa mã nguồn:** Các Controller trở nên gọn gàng hơn, chỉ tập trung vào logic nghiệp vụ mà không bị xen lẫn bởi các khối `try-catch` lặp đi lặp lại.
*   **Dễ bảo trì:** Mọi thay đổi đối với logic xử lý lỗi chỉ cần thực hiện ở một nơi duy nhất.
*   **Ghi nhật ký đầy đủ:** Mọi ngoại lệ đều được ghi nhật ký một cách tự động và chi tiết, kể cả những ngoại lệ mà nhà phát triển có thể đã bỏ sót `try-catch` cục bộ.
*   **Bảo mật:** Đảm bảo rằng các thông tin nhạy cảm về lỗi nội bộ không bị rò rỉ ra bên ngoài cho client.

Trong ASP.NET Core, cách tốt nhất để triển khai xử lý ngoại lệ toàn cục là sử dụng **Middleware tùy chỉnh**.

### 2.2. Middleware trong ASP.NET Core và Cơ chế hoạt động

**Middleware** là một thành phần quan trọng trong kiến trúc ASP.NET Core. Nó là một phần mềm được sắp xếp trong một chuỗi xử lý yêu cầu HTTP (request pipeline). Mỗi middleware có thể:
1.  Thực hiện logic trước khi chuyển yêu cầu cho middleware tiếp theo.
2.  Chuyển yêu cầu cho middleware tiếp theo trong chuỗi bằng cách gọi `await _next(httpContext)`.
3.  Thực hiện logic sau khi middleware tiếp theo (hoặc toàn bộ chuỗi) đã hoàn thành xử lý và trả về phản hồi.
4.  Ngắn mạch chuỗi bằng cách tự tạo và trả về phản hồi, ngăn chặn các middleware còn lại trong chuỗi được thực thi.

**Cơ chế hoạt động "Under the Hood":**
Khi một yêu cầu HTTP đến ứng dụng ASP.NET Core, nó sẽ đi qua từng middleware trong pipeline theo thứ tự mà chúng được đăng ký trong `Program.cs`. Mỗi middleware nhận một `HttpContext` và một `RequestDelegate` (`_next`) trỏ đến middleware kế tiếp.
Middleware xử lý ngoại lệ toàn cục sẽ được đặt ở vị trí đầu chuỗi. Điều này rất quan trọng vì nó cho phép middleware này "bao bọc" toàn bộ phần còn lại của pipeline. Nếu bất kỳ middleware hoặc Controller nào *sau* nó ném ra một ngoại lệ chưa được xử lý, ngoại lệ đó sẽ được `try-catch` của middleware xử lý ngoại lệ bắt giữ.

### 2.3. Xây dựng Middleware Xử lý ngoại lệ toàn cục tùy chỉnh

Chúng ta sẽ tạo một lớp Middleware mới để xử lý các ngoại lệ chưa được xử lý.

#### 2.3.1. Tạo thư mục và lớp Middleware

1.  Trong thư mục gốc của dự án, tạo một thư mục mới có tên là `Middlewares`.
2.  Trong thư mục `Middlewares`, thêm một lớp mới có tên `ExceptionHandlerMiddleware.cs`.

#### 2.3.2. Mã nguồn `ExceptionHandlerMiddleware.cs`

```csharp
using System.Net; // Để sử dụng HttpStatusCode
using System.Text.Json; // Để serialize đối tượng lỗi thành JSON
using Serilog; // Đảm bảo có câu lệnh using này
using NZWalks.API.Models.DTO; // Giả sử bạn có một DTO cho phản hồi lỗi

namespace NZWalks.API.Middlewares
{
    // Middleware này được thiết kế để bắt và xử lý tất cả các ngoại lệ chưa được xử lý
    // phát sinh trong quá trình xử lý yêu cầu HTTP.
    public class ExceptionHandlerMiddleware
    {
        private readonly RequestDelegate _next; // Tham chiếu đến middleware tiếp theo trong pipeline
        private readonly ILogger<ExceptionHandlerMiddleware> _logger; // Logger để ghi lại lỗi

        // Constructor để inject RequestDelegate và ILogger thông qua Dependency Injection.
        public ExceptionHandlerMiddleware(RequestDelegate next, ILogger<ExceptionHandlerMiddleware> logger)
        {
            _next = next;
            _logger = logger;
        }

        // Phương thức InvokeAsync là trái tim của Middleware.
        // Nó được gọi cho mỗi yêu cầu HTTP đi qua middleware này.
        public async Task InvokeAsync(HttpContext httpContext)
        {
            try
            {
                // Cố gắng gọi middleware tiếp theo trong pipeline.
                // Nếu không có ngoại lệ nào xảy ra, yêu cầu sẽ được xử lý bình thường.
                await _next(httpContext);
            }
            catch (Exception ex)
            {
                // --- Một ngoại lệ chưa được xử lý đã xảy ra! ---
                // Bước 1: Tạo một ID lỗi duy nhất để theo dõi.
                // ID này sẽ được ghi vào log và trả về cho client, giúp client báo cáo lỗi
                // và bạn dễ dàng tìm kiếm chi tiết lỗi trong log server.
                var errorId = Guid.NewGuid(); 

                // Bước 2: Ghi lại ngoại lệ chi tiết vào Serilog.
                // Truyền đối tượng 'ex' để Serilog có thể ghi stack trace đầy đủ.
                // Sử dụng ghi nhật ký có cấu trúc để thêm ErrorId và Message vào log.
                _logger.LogError(ex, "ErrorId: {ErrorId} - An unexpected error occurred: {Message}", errorId, ex.Message);

                // --- Bước 3: Chuẩn bị phản hồi lỗi cho client ---
                // Đặt mã trạng thái HTTP của phản hồi thành 500 Internal Server Error.
                httpContext.Response.StatusCode = (int)HttpStatusCode.InternalServerError; 
                // Đặt loại nội dung của phản hồi để client biết rằng nó sẽ nhận được JSON.
                httpContext.Response.ContentType = "application/json"; 

                // Tạo một đối tượng DTO chứa thông tin lỗi an toàn để gửi về client.
                // Quan trọng: KHÔNG NÊN trả về chi tiết ngoại lệ nhạy cảm (như stack trace đầy đủ
                // hoặc thông báo lỗi nội bộ) cho client trong môi trường production vì lý do bảo mật.
                // Thay vào đó, cung cấp một thông báo chung chung và sử dụng ErrorId để tra cứu chi tiết.
                var errorResponse = new ErrorResponseDto 
                {
                    Id = errorId,
                    Message = "Đã xảy ra lỗi không mong muốn. Vui lòng thử lại sau."
                    // Trong môi trường phát triển (debug), bạn CÓ THỂ thêm chi tiết lỗi để gỡ lỗi dễ hơn:
                    // Details = ex.Message,
                    // StackTrace = ex.StackTrace
                };

                // Ghi đối tượng lỗi dưới dạng JSON vào phản hồi HTTP.
                await httpContext.Response.WriteAsJsonAsync(errorResponse);
            }
        }
    }
}
```

Để `ErrorResponseDto` hoạt động, bạn cần định nghĩa nó. Tạo một thư mục `Models/DTO` nếu chưa có và thêm lớp sau:

```csharp
// Models/DTO/ErrorResponseDto.cs
namespace NZWalks.API.Models.DTO
{
    public class ErrorResponseDto
    {
        public Guid Id { get; set; } // ID lỗi duy nhất để theo dõi
        public string Message { get; set; } = string.Empty; // Thông báo lỗi chung chung
        
        // Các thuộc tính tùy chọn chỉ nên được bật trong môi trường phát triển (development)
        // để tránh rò rỉ thông tin nhạy cảm trong production.
        // public string? Details { get; set; } 
        // public string? StackTrace { get; set; }
    }
}
```

**Giải thích chi tiết mã nguồn Middleware:**
*   `RequestDelegate _next`: Đây là một delegate đại diện cho middleware tiếp theo trong chuỗi xử lý yêu cầu. Khi `_next(httpContext)` được gọi, yêu cầu sẽ được chuyển đến middleware kế tiếp.
*   `ILogger<ExceptionHandlerMiddleware> _logger`: Được inject thông qua DI để ghi lại các ngoại lệ xảy ra trong ứng dụng. Việc sử dụng `ILogger<ExceptionHandlerMiddleware>` giúp Serilog biết nguồn gốc của thông báo log.
*   `InvokeAsync(HttpContext httpContext)`: Đây là phương thức bắt buộc mà mọi middleware phải triển khai.
    *   **Khối `try-catch`:** Bao quanh `await _next(httpContext)`. Điều này đảm bảo rằng bất kỳ ngoại lệ nào phát sinh từ các middleware hoặc Controller *sau* `ExceptionHandlerMiddleware` trong pipeline đều sẽ được bắt.
    *   `var errorId = Guid.NewGuid();`: Tạo một định danh duy nhất cho mỗi lỗi. ID này cực kỳ hữu ích để client có thể báo cáo lỗi (ví dụ: "Lỗi Id: abc-123") và bạn có thể dễ dàng tìm thấy chi tiết lỗi tương ứng trong log file của server.
    *   `_logger.LogError(...)`: Ghi lại ngoại lệ chi tiết vào Serilog. Việc truyền đối tượng `ex` là rất quan trọng vì nó cho phép Serilog ghi lại toàn bộ stack trace, giúp xác định chính xác vị trí và nguyên nhân của lỗi.
    *   `httpContext.Response.StatusCode = (int)HttpStatusCode.InternalServerError;`: Đặt mã trạng thái HTTP của phản hồi thành `500 Internal Server Error`, đây là mã tiêu chuẩn cho các lỗi không mong muốn phía máy chủ.
    *   `httpContext.Response.ContentType = "application/json";`: Đặt loại nội dung của phản hồi để client biết rằng nó sẽ nhận được dữ liệu JSON.
    *   `var errorResponse = new ErrorResponseDto { ... };`: Tạo một đối tượng DTO chứa thông tin lỗi. Điều quan trọng là chỉ cung cấp một thông báo chung chung cho client trong môi trường production (`Message = "Đã xảy ra lỗi không mong muốn..."`) để tránh rò rỉ thông tin nhạy cảm về cấu trúc nội bộ của ứng dụng.
    *   `await httpContext.Response.WriteAsJsonAsync(errorResponse);`: Ghi đối tượng `errorResponse` đã được serialize thành JSON vào body của phản hồi HTTP.

### 2.4. Tích hợp Middleware vào Request Pipeline

Để Middleware xử lý ngoại lệ hoạt động, bạn cần thêm nó vào pipeline xử lý yêu cầu HTTP trong tệp `Program.cs`.

```csharp
// Program.cs (sau phần cấu hình Serilog và các dịch vụ khác)
using NZWalks.API.Middlewares; // Đảm bảo có câu lệnh using này

var builder = WebApplication.CreateBuilder(args);

// ... (Cấu hình Serilog và các dịch vụ khác) ...

var app = builder.Build();

// --- Tích hợp Middleware xử lý ngoại lệ toàn cục ---
// RẤT QUAN TRỌNG: Middleware này PHẢI được đặt ở ĐẦU pipeline
// (trước tất cả các middleware khác có khả năng gây lỗi như UseRouting, UseAuthorization, MapControllers)
// để nó có thể bắt tất cả các ngoại lệ phát sinh từ các middleware và controller sau nó.
app.UseMiddleware<ExceptionHandlerMiddleware>();
// --- Kết thúc tích hợp Middleware ---

// Cấu hình đường ống xử lý yêu cầu HTTP.
// Các middleware khác (Swagger, HttpsRedirection, Authorization, v.v.)
// sẽ được đặt SAU ExceptionHandlerMiddleware.
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
> **Vị trí Middleware là tối quan trọng!**
> Đảm bảo rằng `app.UseMiddleware<ExceptionHandlerMiddleware>()` được đặt **trước** tất cả các middleware khác có khả năng gây lỗi (như `app.UseRouting()`, `app.UseAuthorization()`, `app.MapControllers()`). Nếu bạn đặt nó sau, các ngoại lệ từ các middleware trước đó sẽ không được bắt bởi `ExceptionHandlerMiddleware` của bạn.

### 2.5. Kiểm tra Xử lý ngoại lệ toàn cục

Bây giờ, hãy loại bỏ các khối `try-catch` cục bộ khỏi Controller của bạn và mô phỏng một ngoại lệ để xem Middleware hoạt động như thế nào.

```csharp
// Trong một Controller bất kỳ, ví dụ: WalksController.cs
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain; 
using NZWalks.API.Models.DTO; 
using Serilog; 

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class WalksController : ControllerBase
    {
        private readonly ILogger<WalksController> _logger;
        // private readonly IWalkRepository _walkRepository;
        // private readonly IMapper _mapper;

        public WalksController(ILogger<WalksController> logger /*, IWalkRepository walkRepository, IMapper mapper */)
        {
            _logger = logger;
            // _walkRepository = walkRepository;
            // _mapper = mapper;
        }

        [HttpGet]
        public IActionResult GetAllWalks()
        {
            _logger.LogInformation("GetAllWalks action method was invoked.");

            // --- Tạo một ngoại lệ giả định để kiểm tra Middleware ---
            // Ngoại lệ này sẽ không được xử lý cục bộ và sẽ được ExceptionHandlerMiddleware bắt.
            throw new Exception("Đây là một ngoại lệ tùy chỉnh được ném từ GetAllWalks để kiểm tra Middleware.");
            // --- Kết thúc ngoại lệ giả định ---

            // Logic thực tế để lấy danh sách Walks
            // var walksDomain = _walkRepository.GetAll();
            // var walksDto = _mapper.Map<List<WalkDto>>(walksDomain);
            // _logger.LogInformation($"Finished GetAllWalks request with data: {JsonSerializer.Serialize(walksDto)}");
            // return Ok(walksDto);
        }

        // Các phương thức khác...
    }
}
```

Khi bạn chạy ứng dụng và gọi endpoint `/api/Walks` (hoặc bất kỳ endpoint nào mà bạn đã thêm ngoại lệ), bạn sẽ thấy:
*   **Phản hồi từ API:** Client sẽ nhận được một phản hồi HTTP `500 Internal Server Error` với một payload JSON có định dạng như `ErrorResponseDto` (chứa một `Id` duy nhất và thông báo lỗi chung chung, ví dụ: `{"id": "...", "message": "Đã xảy ra lỗi không mong muốn. Vui lòng thử lại sau."}`).
*   **Nhật ký Server:** Trong cửa sổ console và tệp nhật ký, bạn sẽ thấy một thông báo `LogError` được tạo bởi `ExceptionHandlerMiddleware`, bao gồm `ErrorId` duy nhất, thông báo lỗi và stack trace đầy đủ của ngoại lệ.

Điều này chứng tỏ rằng Middleware xử lý ngoại lệ toàn cục đã hoạt động thành công: nó đã bắt được ngoại lệ, ghi lại thông tin chi tiết cho nhà phát triển, và trả về một phản hồi lỗi nhất quán và an toàn cho client.

### 2.6. Nâng cao xử lý ngoại lệ và Hỗ trợ từ Antigravity IDE

Xử lý ngoại lệ không chỉ là việc bắt lỗi mà còn là việc thiết kế các phản hồi lỗi có ý nghĩa và khả năng chẩn đoán cao.

**Vibe Coding và Xử lý ngoại lệ:**
Tư duy Vibe Coding trong xử lý ngoại lệ tập trung vào việc:
*   **Dự đoán thất bại:** Trước khi viết code, hãy nghĩ về những gì có thể sai và cách hệ thống nên phản ứng. Điều này giúp bạn thiết kế các loại ngoại lệ tùy chỉnh (`CustomException`) hoặc mã lỗi cụ thể.
*   **Thông báo lỗi rõ ràng (cho nhà phát triển):** Đảm bảo log lỗi cung cấp đủ ngữ cảnh và chi tiết để gỡ lỗi nhanh chóng.
*   **Thông báo lỗi thân thiện (cho người dùng):** Phản hồi API nên hướng dẫn người dùng hoặc chỉ ra vấn đề một cách chung chung, không tiết lộ chi tiết kỹ thuật.
*   **Đồng nhất hóa:** Đảm bảo rằng mọi loại lỗi đều được xử lý theo một cách nhất quán về mặt cấu trúc phản hồi.

**Antigravity IDE và Hỗ trợ Xử lý ngoại lệ:**
Hệ thống Agentic AI như Antigravity IDE có thể cung cấp hỗ trợ đáng kể trong việc xây dựng và duy trì một hệ thống xử lý ngoại lệ mạnh mẽ:

*   **Tạo Middleware tự động:** Dựa trên các yêu cầu về định dạng phản hồi lỗi (ví dụ: "cần `ErrorId`, `Message` và chỉ `Details` trong môi trường Dev"), Antigravity có thể tự động tạo scaffolding cho `ExceptionHandlerMiddleware` và `ErrorResponseDto`.
*   **Phân tích và đề xuất xử lý lỗi:** Khi bạn viết code có thể ném ngoại lệ (ví dụ: tương tác với cơ sở dữ liệu, gọi API bên ngoài), Antigravity có thể phân tích và đề xuất:
    *   Sử dụng các loại ngoại lệ cụ thể (ví dụ: `NotFoundException`, `ValidationException`).
    *   Thêm các khối `try-catch` cục bộ cho các trường hợp đặc biệt cần xử lý ngay lập tức (ví dụ: roll back transaction) trước khi ngoại lệ được đẩy lên Middleware toàn cục.
    *   Tự động thêm log `_logger.LogError` với `ErrorId` và stack trace khi một ngoại lệ được bắt.
*   **Tích hợp với hệ thống chẩn đoán:** Antigravity có thể kết nối trực tiếp với các hệ thống giám sát và quản lý log (như Seq, Elasticsearch) để:
    *   Tự động báo cáo lỗi nghiêm trọng.
    *   Phân tích nguyên nhân gốc rễ (Root Cause Analysis) bằng cách đối chiếu `ErrorId` với các log liên quan khác trong một khoảng thời gian nhất định.
    *   Đề xuất các giải pháp khắc phục dựa trên dữ liệu lỗi lịch sử và kiến thức về các mẫu lỗi phổ biến.
*   **Đảm bảo tuân thủ bảo mật:** Antigravity có thể kiểm tra các phản hồi lỗi để đảm bảo rằng không có thông tin nhạy cảm nào bị rò rỉ ra bên ngoài trong môi trường production, tuân thủ các nguyên tắc bảo mật.
*   **Kiểm thử lỗi tự động:** Có thể tạo các kịch bản kiểm thử tự động để kích hoạt các ngoại lệ và xác minh rằng `ExceptionHandlerMiddleware` hoạt động đúng cách và phản hồi lỗi theo định dạng mong muốn.

Bằng cách tận dụng Antigravity IDE, quá trình thiết kế và triển khai xử lý ngoại lệ trở nên thông minh hơn, ít tốn công sức hơn, và đảm bảo một API có khả năng phục hồi cao hơn.

---

## Tóm tắt Chương

Trong chương này, chúng ta đã đi sâu vào hai chủ đề quan trọng để xây dựng các RESTful Web API mạnh mẽ, dễ bảo trì và có khả năng phục hồi cao trong ASP.NET Core: **Ghi nhật ký** và **Xử lý ngoại lệ toàn cục**.

Chúng ta đã bắt đầu với **Ghi nhật ký (Logging)**, hiểu rõ vai trò thiết yếu của nó trong việc theo dõi, gỡ lỗi và phân tích hành vi ứng dụng.
*   Chúng ta đã khám phá nền tảng ghi nhật ký tích hợp của .NET Core với `ILogger` và `ILoggerFactory`, cùng với các cấp độ ghi nhật ký chuẩn (`Trace`, `Debug`, `Information`, `Warning`, `Error`, `Critical`).
*   Tiếp theo, chúng ta đã tích hợp thư viện **Serilog** mạnh mẽ, nổi bật với khả năng ghi nhật ký có cấu trúc (structured logging), cho phép thu thập và phân tích dữ liệu log hiệu quả hơn. Chúng ta đã cài đặt các gói NuGet cần thiết (`Serilog`, `Serilog.AspNetCore`, `Serilog.Sinks.Console`, `Serilog.Sinks.File`) và cấu hình Serilog trong `Program.cs` để ghi nhật ký ra cả **Console** và **Tệp văn bản**, đồng thời kiểm soát mức ghi nhật ký tối thiểu bằng `MinimumLevel`.
*   Chúng ta cũng đã học cách sử dụng **Dependency Injection** để inject `ILogger<T>` vào các Controller và ghi lại các sự kiện, thông tin, cảnh báo, và đặc biệt là ngoại lệ một cách chi tiết bằng các phương thức như `LogInformation`, `LogWarning`, `LogError`, tận dụng tối đa các thuộc tính có cấu trúc.
*   Cuối cùng, chúng ta đã liên hệ việc tối ưu hóa ghi nhật ký với tư duy **Vibe Coding** và vai trò hỗ trợ của **Antigravity IDE** trong việc tự động tạo cấu hình, đề xuất nội dung log, đảm bảo cấu trúc và phân tích nhật ký.

Phần thứ hai của chương tập trung vào **Xử lý ngoại lệ toàn cục (Global Exception Handling)**, một yếu tố then chốt để tạo ra các API ổn định với phản hồi lỗi nhất quán.
*   Chúng ta đã phân tích nhược điểm của việc sử dụng các khối `try-catch` cục bộ rải rác trong mã nguồn, dẫn đến sự lặp lại mã, khó bảo trì và phản hồi không nhất quán.
*   Để khắc phục, chúng ta đã học cách xây dựng một **Middleware tùy chỉnh** (`ExceptionHandlerMiddleware`) để tập trung toàn bộ logic xử lý ngoại lệ vào một vị trí duy nhất.
*   Trong Middleware này, chúng ta đã ghi lại các ngoại lệ chi tiết bằng Serilog (bao gồm một `ErrorId` duy nhất để dễ dàng đối chiếu) và tạo ra một phản hồi lỗi **nhất quán và an toàn thông tin** cho client (sử dụng `ErrorResponseDto`), tránh rò rỉ chi tiết nhạy cảm.
*   Việc tích hợp Middleware này vào **HTTP Request Pipeline** của ASP.NET Core trong `Program.cs`, với vị trí đặt ở đầu pipeline, là rất quan trọng để đảm bảo nó có thể bắt tất cả các ngoại lệ chưa được xử lý từ các thành phần sau nó.
*   Chúng ta cũng đã thảo luận về việc nâng cao xử lý ngoại lệ thông qua **Vibe Coding** để dự đoán thất bại và thiết kế thông báo lỗi, cùng với cách **Antigravity IDE** có thể hỗ trợ tự động tạo middleware, phân tích lỗi, và đảm bảo tuân thủ bảo mật.

Bằng cách triển khai ghi nhật ký toàn diện và xử lý ngoại lệ toàn cục, bạn đã trang bị cho API của mình khả năng tự phục hồi, dễ dàng gỡ lỗi, giám sát hiệu quả và cung cấp trải nghiệm người dùng chuyên nghiệp hơn, đồng thời tận dụng các công cụ AI hiện đại để tối ưu hóa quy trình phát triển.

<!-- REVIEWED_BY_AGENT -->
