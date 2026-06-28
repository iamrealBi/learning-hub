# Chương 2: Nền tảng Kiến trúc ASP.NET Core Web API và Nguyên lý RESTful

Chương này đi sâu vào cấu trúc cơ bản của một dự án ASP.NET Core Web API, từ việc khởi tạo đến việc hiểu rõ các khái niệm cốt lõi. Chúng ta sẽ khám phá các nguyên tắc của kiến trúc RESTful, vai trò của HTTP Verbs, cơ chế Định tuyến (Routing), và cách thức hoạt động của Dependency Injection (DI) cùng Middleware trong `Program.cs`. Mục tiêu là xây dựng nền tảng vững chắc, không chỉ để viết code mà còn để "cảm nhận" được luồng hoạt động của một API, một tư duy rất quan trọng khi làm việc với các hệ thống AI Coding như Antigravity IDE.

## 2.1. Giới thiệu về ASP.NET Core Web API và Kiến trúc RESTful

### 2.1.1. ASP.NET Core Web API: Nền tảng Xây dựng Dịch vụ Hiện đại

ASP.NET Core Web API là một framework mã nguồn mở, đa nền tảng và hiệu suất cao của Microsoft, được thiết kế để xây dựng các dịch vụ HTTP. Các dịch vụ này, thường được gọi là Web API, đóng vai trò là cầu nối cung cấp dữ liệu và chức năng cho nhiều loại ứng dụng khách (client) khác nhau, bao gồm ứng dụng web (Single Page Applications như React, Angular, Vue), ứng dụng di động (iOS, Android), ứng dụng desktop, và thậm chí là các hệ thống backend khác.

**Đặc điểm nổi bật của ASP.NET Core Web API:**
*   **Hiệu suất cao:** Được tối ưu hóa cho tốc độ và khả năng mở rộng.
*   **Đa nền tảng:** Có thể chạy trên Windows, macOS và Linux.
*   **Mã nguồn mở:** Cộng đồng lớn và sự phát triển liên tục.
*   **Tích hợp Dependency Injection:** Giúp xây dựng ứng dụng có cấu trúc rõ ràng, dễ kiểm thử và bảo trì.
*   **Middleware Pipeline:** Cung cấp khả năng tùy chỉnh mạnh mẽ luồng xử lý yêu cầu HTTP.

### 2.1.2. REST (Representational State Transfer): Phong cách Kiến trúc cho Web API

REST không phải là một giao thức mà là một phong cách kiến trúc tập hợp các nguyên tắc thiết kế cho các hệ thống phân tán, đặc biệt là các dịch vụ web. Các API tuân thủ các nguyên tắc này được gọi là **RESTful API**. Mục tiêu của REST là tạo ra các API đơn giản, có khả năng mở rộng và dễ sử dụng bằng cách tận dụng các chuẩn mực đã có của giao thức HTTP.

#### Các Nguyên tắc Cốt lõi của REST

1.  **Dựa trên Tài nguyên (Resource-Based):**
    *   Mọi thứ trong REST đều được xem là một "tài nguyên" (resource). Tài nguyên là một khái niệm trừu tượng đại diện cho bất kỳ đối tượng, dữ liệu hoặc dịch vụ nào có thể được truy cập và thao tác thông qua API. Ví dụ: một `Người dùng`, một `Sản phẩm`, một `Đơn hàng`.
    *   Mỗi tài nguyên được xác định duy nhất bởi một **URI (Uniform Resource Identifier)**. URI không chỉ là địa chỉ mà còn là danh tính của tài nguyên.
    *   **Ví dụ:** `/api/products` (tập hợp các sản phẩm), `/api/products/123` (sản phẩm có ID là 123).

2.  **Không trạng thái (Stateless):**
    *   Mỗi yêu cầu từ máy khách đến máy chủ phải chứa tất cả thông tin cần thiết để máy chủ xử lý yêu cầu đó. Máy chủ không được lưu trữ bất kỳ trạng thái phiên (session state) nào của máy khách giữa các yêu cầu.
    *   **Cơ chế ngầm (Under the hood):** Điều này có nghĩa là mỗi yêu cầu là độc lập. Nếu một yêu cầu cần thông tin xác thực, thông tin đó phải được gửi kèm trong *mỗi* yêu cầu. Lợi ích là máy chủ có thể dễ dàng mở rộng theo chiều ngang (scale horizontally) vì bất kỳ máy chủ nào cũng có thể xử lý bất kỳ yêu cầu nào mà không cần biết về các yêu cầu trước đó của máy khách.

3.  **Sử dụng HTTP Verbs tiêu chuẩn:**
    *   REST sử dụng các động từ HTTP (HTTP Verbs hay HTTP Methods) để mô tả loại hành động mà máy khách muốn thực hiện trên tài nguyên. Các động từ này có ý nghĩa ngữ nghĩa rõ ràng.
    *   **Ví dụ:** `GET` (truy xuất), `POST` (tạo mới), `PUT` (cập nhật toàn bộ), `PATCH` (cập nhật một phần), `DELETE` (xóa).

4.  **Giao diện thống nhất (Uniform Interface):**
    *   Nguyên tắc này đề xuất rằng các thành phần trong hệ thống (máy khách và máy chủ) nên tương tác thông qua một giao diện chung và nhất quán. Điều này làm cho việc tương tác giữa các thành phần trở nên đơn giản, dễ dự đoán và giảm thiểu sự ghép nối (coupling).
    *   **Các yếu tố của giao diện thống nhất:**
        *   **Xác định tài nguyên:** Thông qua URI.
        *   **Thao tác tài nguyên thông qua biểu diễn:** Máy khách nhận được một "biểu diễn" (representation) của tài nguyên (ví dụ: JSON, XML) và thao tác với biểu diễn đó.
        *   **Thông điệp tự mô tả:** Mỗi thông điệp (yêu cầu/phản hồi) phải chứa đủ thông tin để người nhận hiểu được cách xử lý nó.
        *   **HATEOAS (Hypermedia As The Engine Of Application State):** Một tài nguyên có thể chứa các liên kết (hyperlinks) đến các tài nguyên hoặc hành động liên quan, giúp máy khách khám phá API động mà không cần biết trước tất cả các URI. (Đây là một nguyên tắc nâng cao hơn, ít khi được áp dụng triệt để trong các API ban đầu).

#### Tầm quan trọng của REST trong bối cảnh AI Coding (Antigravity IDE)

Trong kỷ nguyên AI Coding, việc thiết kế API theo chuẩn RESTful không chỉ giúp con người mà còn hỗ trợ mạnh mẽ cho các hệ thống như Antigravity IDE.
*   **Dễ hiểu và Phân tích cho AI:** Các nguyên tắc rõ ràng của REST (URI cho tài nguyên, HTTP Verbs cho hành động, trạng thái không lưu trữ) cung cấp một cấu trúc ngữ nghĩa mà Antigravity có thể dễ dàng phân tích và hiểu. Thay vì phải "suy luận" ý định của một endpoint, AI có thể dựa vào các quy ước đã biết.
*   **Tự động tạo mã và kiểm thử:** Khi một API tuân thủ REST, Antigravity có thể tự động tạo ra các đoạn mã client để gọi API đó, hoặc thậm chí sinh ra các kịch bản kiểm thử (test cases) một cách chính xác hơn. Ví dụ, khi thấy `GET /api/products/{id}`, Antigravity biết rằng đây là thao tác đọc một sản phẩm cụ thể.
*   **Tương tác linh hoạt:** Giao diện thống nhất giúp Antigravity dễ dàng thích nghi với các API khác nhau mà không cần học một giao diện mới mỗi lần. Nó có thể dự đoán cách một tài nguyên sẽ được thao tác.
*   **Vibe Coding:** Một API có "vibe" tốt là một API mà ý định của nó rõ ràng ngay lập tức. Với Antigravity, điều này có nghĩa là khi AI đọc định nghĩa API của bạn, nó có thể ngay lập tức "cảm nhận" được mục đích của từng endpoint, giúp nó viết code chính xác và hiệu quả hơn.

## 2.2. Khởi tạo Dự án ASP.NET Core Web API

Việc tạo một dự án mới là bước đầu tiên để đặt nền móng cho RESTful API của bạn.

### 2.2.1. Hướng dẫn Tạo Dự án Mới với Visual Studio

1.  **Mở Visual Studio:** Khởi động Visual Studio (phiên bản 2022 trở lên được khuyến nghị).
2.  **Tạo dự án mới:** Trên màn hình khởi động, chọn **"Create a new project"**.
3.  **Tìm kiếm mẫu:**
    *   Trong hộp thoại "Create a new project", sử dụng các bộ lọc để thu hẹp kết quả:
        *   **Language:** `C#`
        *   **Platform:** `All platforms`
        *   **Project type:** `Web`
    *   Tìm và chọn mẫu **"ASP.NET Core Web API"**. Mô tả của mẫu này thường là "A template for creating ASP.NET Core applications with an example controller for a RESTful HTTP service."
    *   Nhấp vào **"Next"**.
4.  **Cấu hình dự án:**
    *   **Project name:** Đặt tên cho dự án API của bạn. Ví dụ: `NZWalks.API`.
    *   **Location:** Chọn thư mục mà bạn muốn lưu trữ dự án.
    *   **Solution name:** Visual Studio sẽ tự động đề xuất tên giải pháp dựa trên tên dự án. Đối với các dự án lớn hơn, một giải pháp (solution) có thể chứa nhiều dự án con. Ví dụ, bạn có thể đặt tên giải pháp là `NZWalks` để chứa `NZWalks.API` và các dự án khác sau này.
    *   Nhấp vào **"Next"**.

### 2.2.2. Cấu hình Dự án Ban đầu và Lựa chọn Phiên bản .NET

Hộp thoại "Additional information" cho phép bạn cấu hình các tùy chọn quan trọng cho dự án:

*   **Framework:** Chọn phiên bản .NET mà bạn muốn dự án nhắm tới. Luôn ưu tiên chọn phiên bản hỗ trợ dài hạn (LTS) mới nhất để đảm bảo ổn định và nhận được hỗ trợ lâu dài. Tại thời điểm hiện tại, **.NET 8 (Long Term Support)** là lựa chọn được khuyến nghị.
*   **Authentication type:**
    *   Đối với khóa học này và các dự án thực hành ban đầu, chọn **"None"**. Điều này cho phép bạn tự triển khai hoặc tích hợp các cơ chế xác thực sau này, giúp hiểu rõ hơn về cách chúng hoạt động.
*   **Configure for HTTPS:** Giữ mặc định là đã chọn. HTTPS là tiêu chuẩn bảo mật cho truyền thông web.
*   **Enable Docker:** Bỏ chọn (hoặc giữ nguyên nếu bạn muốn khám phá Docker sớm).
*   **Use controllers (uncheck to use minimal APIs):** Giữ mặc định là đã chọn. Chúng ta sẽ tập trung vào kiến trúc Controller truyền thống. Minimal APIs là một phong cách mới hơn, phù hợp cho các API đơn giản hơn.
*   **Do not use top-level statements:** Bỏ chọn (để sử dụng cú pháp top-level statements hiện đại trong `Program.cs`).
*   Nhấp vào **"Create"**.

Visual Studio sẽ tạo ra một dự án ASP.NET Core Web API cơ bản, sẵn sàng cho bạn khám phá.

## 2.3. Giải phẫu Cấu trúc Dự án ASP.NET Core Web API

Khi một dự án ASP.NET Core Web API được tạo, Visual Studio thiết lập một cấu trúc tệp và thư mục tiêu chuẩn. Việc hiểu rõ vai trò của từng thành phần là chìa khóa để quản lý và phát triển ứng dụng một cách hiệu quả.

### 2.3.1. Tổng quan về Giải pháp (`.sln`) và Dự án (`.csproj`)

*   **Giải pháp (`.sln` - Solution File):** Đây là một tệp văn bản có cấu trúc XML, hoạt động như một container logic để tổ chức một hoặc nhiều dự án liên quan trong Visual Studio. Một giải pháp có thể chứa nhiều dự án (ví dụ: một dự án API, một dự án giao diện người dùng web, một dự án thư viện lớp chứa logic nghiệp vụ dùng chung, một dự án kiểm thử). Nó giúp quản lý các mối quan hệ và thứ tự xây dựng giữa các dự án.

*   **Dự án (`.csproj` - C# Project File):** Đây cũng là một tệp XML, định nghĩa các thuộc tính, tham chiếu, và cấu hình xây dựng cho một dự án C# cụ thể. Bạn có thể xem nội dung tệp này bằng cách nhấp đúp vào tên dự án trong Solution Explorer.
    *   **`<TargetFramework>`:** Chỉ định phiên bản .NET mà dự án của bạn nhắm tới (ví dụ: `net8.0`).
    *   **`<Nullable>`:** Kích hoạt hoặc vô hiệu hóa các kiểu tham chiếu có thể null (Nullable Reference Types) ở cấp độ dự án, giúp phát hiện lỗi null reference tại thời điểm biên dịch.
    *   **`<ImplicitUsings>`:** Kích hoạt tính năng tự động thêm các chỉ thị `using` phổ biến cho các namespace cơ bản, giảm thiểu số lượng `using` bạn phải viết thủ công trong mỗi tệp.
    *   **`<ItemGroup>`:** Chứa các tham chiếu đến các gói NuGet (thư viện bên ngoài) hoặc các dự án khác trong cùng giải pháp mà dự án này phụ thuộc.

### 2.3.2. Quản lý Cấu hình: `appsettings.json` và `launchSettings.json`

Cấu hình là một phần thiết yếu của bất kỳ ứng dụng nào, cho phép ứng dụng hoạt động linh hoạt trong các môi trường khác nhau mà không cần thay đổi mã nguồn.

*   **`launchSettings.json`:**
    *   **Vị trí:** Nằm trong thư mục `Properties`.
    *   **Mục đích:** Chứa các cấu hình liên quan đến việc khởi chạy ứng dụng trong môi trường phát triển (debug). Nó không được triển khai lên môi trường production.
    *   **Nội dung:** Định nghĩa các "profile" khởi chạy khác nhau (ví dụ: HTTP, HTTPS, IIS Express). Mỗi profile chỉ định các thiết lập như:
        *   `commandName`: Loại máy chủ web (ví dụ: `Project` cho Kestrel, `IISExpress`).
        *   `launchBrowser`: Có mở trình duyệt khi khởi chạy không.
        *   `applicationUrl`: Các URL (bao gồm số cổng) mà ứng dụng sẽ lắng nghe khi chạy.
        *   `environmentVariables`: Các biến môi trường cụ thể cho profile đó.
    *   **Lưu ý quan trọng:** Các số cổng trong `applicationUrl` của bạn sẽ khác nhau giữa các máy tính. Luôn kiểm tra tệp `launchSettings.json` trong dự án của bạn để biết chính xác các URL và cổng khi debug.
    *   **Cơ chế ngầm:** Khi bạn nhấn nút "Run" trong Visual Studio, nó sẽ đọc `launchSettings.json` để biết cách cấu hình môi trường và khởi động ứng dụng của bạn.

*   **`appsettings.json`:**
    *   **Vị trí:** Nằm ở thư mục gốc của dự án.
    *   **Mục đích:** Lưu trữ các cài đặt cấu hình cho ứng dụng của bạn, chẳng hạn như:
        *   **Chuỗi kết nối cơ sở dữ liệu:** `ConnectionStrings` để ứng dụng kết nối với database.
        *   **Cấp độ nhật ký (Log levels):** Cấu hình mức độ chi tiết của các thông báo log (ví dụ: `Information`, `Warning`, `Error`).
        *   Các biến cấu hình khác có thể thay đổi tùy theo môi trường (phát triển, staging, production).
    *   **Cơ chế ngầm:** ASP.NET Core có một hệ thống cấu hình mạnh mẽ, có thể đọc dữ liệu từ nhiều nguồn (tệp JSON, biến môi trường, dòng lệnh, Azure Key Vault). Các tệp `appsettings.{EnvironmentName}.json` (ví dụ: `appsettings.Development.json`, `appsettings.Production.json`) cho phép bạn ghi đè các giá trị cấu hình tùy theo môi trường. Hệ thống sẽ tự động tải tệp phù hợp dựa trên biến môi trường `ASPNETCORE_ENVIRONMENT`.

### 2.3.3. `Program.cs`: Trái tim của Ứng dụng

`Program.cs` là điểm vào (entry point) của mọi ứng dụng ASP.NET Core. Khi ứng dụng khởi chạy, mã trong tệp này được thực thi đầu tiên. Nó chịu trách nhiệm cho việc cấu hình các dịch vụ (services) cần thiết và định nghĩa chuỗi các Middleware sẽ xử lý các yêu cầu HTTP. Với cú pháp top-level statements hiện đại của C#, tệp này trở nên gọn gàng và dễ đọc hơn rất nhiều.

```csharp
// 1. Khởi tạo WebApplicationBuilder
var builder = WebApplication.CreateBuilder(args);

// 2. Cấu hình Dịch vụ (Service Configuration) - builder.Services
// Đăng ký các dịch vụ vào container Dependency Injection của ứng dụng
builder.Services.AddControllers(); // Thêm hỗ trợ cho các Controller API
builder.Services.AddEndpointsApiExplorer(); // Hỗ trợ khám phá API cho Swagger
builder.Services.AddSwaggerGen(); // Thêm Swagger/OpenAPI để tài liệu hóa và kiểm thử API

// 3. Xây dựng ứng dụng
var app = builder.Build();

// 4. Cấu hình HTTP Request Pipeline (Middleware) - app
// Định nghĩa chuỗi các Middleware sẽ xử lý mỗi yêu cầu HTTP đến
if (app.Environment.IsDevelopment()) // Chỉ sử dụng trong môi trường phát triển
{
    app.UseSwagger(); // Kích hoạt Swagger middleware
    app.UseSwaggerUI(); // Kích hoạt Swagger UI
}

app.UseHttpsRedirection(); // Chuyển hướng yêu cầu HTTP sang HTTPS để bảo mật

app.UseAuthorization(); // Cấu hình xác thực và ủy quyền

app.MapControllers(); // Ánh xạ các yêu cầu đến các Controller

// 5. Chạy ứng dụng
app.Run();
```

Trong `Program.cs`, có hai phần chính:

#### 2.3.3.1. Cấu hình Dịch vụ (Service Configuration) và Dependency Injection (DI)

Phần `builder.Services` là nơi bạn đăng ký các dịch vụ (services) vào container Dependency Injection (DI) của ứng dụng.

**Dependency Injection (DI): Tại sao và Làm thế nào?**
*   **Tại sao cần DI?** DI là một kỹ thuật thiết kế phần mềm giúp giảm sự ghép nối (loose coupling) giữa các thành phần trong ứng dụng. Thay vì một đối tượng tự tạo ra hoặc tìm kiếm các đối tượng mà nó cần (gọi là "phụ thuộc"), các đối tượng đó sẽ được "tiêm" (inject) vào nó từ bên ngoài thông qua constructor, property hoặc method.
    *   **Lợi ích:**
        *   **Dễ kiểm thử (Testability):** Có thể dễ dàng thay thế các phụ thuộc thực tế bằng các đối tượng giả (mock) hoặc giả lập (stub) trong các bài kiểm thử đơn vị (unit tests).
        *   **Dễ bảo trì và mở rộng:** Các thành phần độc lập hơn, dễ dàng thay đổi hoặc thêm mới mà không ảnh hưởng đến các phần khác của hệ thống.
        *   **Tái sử dụng mã:** Các dịch vụ có thể được tái sử dụng ở nhiều nơi khác nhau.
        *   **Tuân thủ SOLID Principles:** Đặc biệt là Dependency Inversion Principle.
*   **Cơ chế ngầm (Under the hood):** ASP.NET Core có một DI container tích hợp sẵn. Khi bạn gọi `builder.Services.Add...()`, bạn đang "đăng ký" một dịch vụ (ví dụ: một interface và một implementation cụ thể) vào container. Khi một thành phần khác cần dịch vụ đó (ví dụ: thông qua constructor injection), DI container sẽ tạo ra một instance của implementation đã đăng ký và "tiêm" nó vào thành phần đó.

**Các loại Lifetime của dịch vụ trong DI:**
Khi đăng ký một dịch vụ, bạn phải chỉ định lifetime (vòng đời) của nó, quyết định khi nào và bao nhiêu lần một instance của dịch vụ sẽ được tạo.

*   **`Singleton`:**
    *   **Cơ chế:** Một instance duy nhất của dịch vụ được tạo ra và sử dụng trong **toàn bộ vòng đời của ứng dụng**.
    *   **Khi sử dụng:** Phù hợp cho các dịch vụ không trạng thái (stateless) hoặc các dịch vụ cần chia sẻ trạng thái chung (ví dụ: cấu hình, logger, cache).
    *   **Đăng ký:** `builder.Services.AddSingleton<IMyService, MyService>();`
    *   **Analogy:** Giống như một cái bút duy nhất được dùng chung cho tất cả mọi người trong một văn phòng, mọi người đều dùng cùng một cây bút đó.

*   **`Scoped`:**
    *   **Cơ chế:** Một instance của dịch vụ được tạo ra cho **mỗi yêu cầu HTTP** và được tái sử dụng trong suốt yêu cầu đó.
    *   **Khi sử dụng:** Phù hợp cho các dịch vụ xử lý dữ liệu liên quan đến một yêu cầu cụ thể (ví dụ: database context trong Entity Framework Core, các dịch vụ liên quan đến người dùng hiện tại).
    *   **Đăng ký:** `builder.Services.AddScoped<IMyService, MyService>();`
    *   **Analogy:** Giống như mỗi khách hàng vào nhà hàng được phát một bộ dao dĩa riêng, và họ dùng bộ đó cho đến khi rời đi. Mỗi yêu cầu HTTP là một "khách hàng" mới.

*   **`Transient`:**
    *   **Cơ chế:** Một instance mới của dịch vụ được tạo ra **mỗi khi dịch vụ được yêu cầu**.
    *   **Khi sử dụng:** Phù hợp cho các dịch vụ nhẹ, không trạng thái hoặc các dịch vụ cần một instance mới cho mỗi lần sử dụng (ví dụ: các dịch vụ tiện ích, các đối tượng cần được khởi tạo với trạng thái riêng biệt mỗi lần).
    *   **Đăng ký:** `builder.Services.AddTransient<IMyService, MyService>();`
    *   **Analogy:** Giống như một cây diêm, mỗi lần bạn cần lửa, bạn lấy một cây diêm mới.

**DI và Antigravity IDE:**
Việc sử dụng Dependency Injection một cách nhất quán mang lại lợi ích lớn cho các hệ thống AI Coding như Antigravity IDE.
*   **Phân tích mã nguồn rõ ràng:** Antigravity có thể dễ dàng phân tích biểu đồ phụ thuộc (dependency graph) của ứng dụng, hiểu rõ các mối quan hệ giữa các thành phần.
*   **Tự động tạo Mock/Stub:** Khi cần tạo các bài kiểm thử đơn vị, Antigravity có thể tự động sinh ra các đối tượng mock hoặc stub cho các phụ thuộc, vì nó biết cách các dịch vụ được "tiêm" vào.
*   **Đề xuất tái cấu trúc (Refactoring):** Nếu Antigravity phát hiện các phụ thuộc không hợp lý hoặc các vi phạm nguyên tắc SOLID, nó có thể đề xuất các cải tiến về kiến trúc.
*   **Vibe Coding:** Một codebase được thiết kế với DI tốt có một "vibe" của sự rõ ràng và có tổ chức. Antigravity "cảm nhận" được điều này, giúp nó làm việc hiệu quả hơn, ít mắc lỗi hơn khi tự động hóa các tác vụ lập trình.

#### 2.3.3.2. Cấu hình HTTP Request Pipeline (Middleware)

Phần `app` là nơi bạn định nghĩa chuỗi các Middleware sẽ xử lý mỗi yêu cầu HTTP đến.

**Middleware: Cơ chế Chuỗi Trách nhiệm**
*   **Cơ chế ngầm (Under the hood):** Middleware là một phần mềm được tích hợp vào pipeline của ứng dụng để xử lý các yêu cầu và phản hồi HTTP. Mỗi Middleware có thể thực hiện một tác vụ cụ thể (ví dụ: ghi log, xác thực, xử lý lỗi, nén dữ liệu) trước khi chuyển yêu cầu cho Middleware tiếp theo trong chuỗi, hoặc nó có thể tạo phản hồi và kết thúc chuỗi.
*   **Thứ tự quan trọng:** Middleware được thực thi theo thứ tự mà chúng được thêm vào trong `Program.cs`. Thứ tự này rất quan trọng vì mỗi Middleware có thể ảnh hưởng đến cách yêu cầu được xử lý bởi các Middleware sau đó. Ví dụ, `UseAuthentication()` phải đứng trước `UseAuthorization()`.

**Các Middleware thiết yếu:**

*   **`if (app.Environment.IsDevelopment()) { app.UseSwagger(); app.UseSwaggerUI(); }`:**
    *   `UseSwagger()`: Kích hoạt Swagger middleware, cho phép ứng dụng tạo ra các tài liệu API theo chuẩn OpenAPI.
    *   `UseSwaggerUI()`: Kích hoạt giao diện người dùng Swagger, cung cấp một trang web tương tác để khám phá và kiểm thử API của bạn.
    *   **Lưu ý:** Thường chỉ được kích hoạt trong môi trường phát triển để tránh phơi bày tài liệu API công khai trên production.

*   **`app.UseHttpsRedirection();`:**
    *   Đảm bảo rằng tất cả các yêu cầu HTTP không bảo mật được tự động chuyển hướng sang HTTPS. Điều này rất quan trọng cho bảo mật.

*   **`app.UseAuthorization();`:**
    *   Xử lý các quy tắc ủy quyền (authorization), xác định xem người dùng hiện tại có quyền truy cập vào một tài nguyên hoặc hành động cụ thể hay không. (Lưu ý: Xác thực `UseAuthentication()` thường được thêm trước `UseAuthorization()` nếu bạn có cấu hình xác thực).

*   **`app.MapControllers();`:**
    *   Đây là Middleware cuối cùng trong chuỗi xử lý định tuyến. Nó ánh xạ các yêu cầu đến các Controller và Action Methods phù hợp dựa trên cấu hình định tuyến của bạn. Nếu không có `MapControllers()`, các yêu cầu API sẽ không được chuyển đến Controller của bạn.

**Tầm quan trọng của Pipeline đối với khả năng kiểm soát của AI:**
Một pipeline Middleware được cấu trúc rõ ràng giúp Antigravity IDE hiểu được luồng xử lý của một yêu cầu. Điều này cho phép AI:
*   **Đề xuất chèn Middleware:** Nếu bạn quên một Middleware quan trọng (ví dụ: xác thực), Antigravity có thể gợi ý thêm nó vào đúng vị trí.
*   **Phân tích hiệu suất:** AI có thể phân tích các Middleware để xác định các điểm nghẽn hiệu suất tiềm năng.
*   **Kiểm tra bảo mật:** Bằng cách hiểu thứ tự của các Middleware bảo mật, Antigravity có thể giúp xác định các lỗ hổng (ví dụ: ủy quyền được thực hiện trước xác thực).

#### 2.3.4. Thư mục `Controllers`

Đây là thư mục trung tâm nơi bạn sẽ tạo các Controller cho API của mình.
*   Mỗi Controller là một lớp C# kế thừa từ `ControllerBase` (hoặc `Controller` nếu bạn xây dựng cả API và View).
*   Mỗi Controller chứa một hoặc nhiều phương thức hành động (Action Methods) được thiết kế để xử lý các yêu cầu HTTP đến các tài nguyên cụ thể.
*   Các Controller chịu trách nhiệm nhận yêu cầu, gọi logic nghiệp vụ (thường thông qua các dịch vụ đã được inject), và trả về phản hồi HTTP.

### 2.3.5. Thư mục `Models` (Thường được thêm thủ công)

Mặc dù không được tạo mặc định, thư mục `Models` là một quy ước phổ biến để chứa các lớp đại diện cho dữ liệu của ứng dụng (ví dụ: `Product`, `User`, `Order`). Các lớp này được sử dụng trong các Action Method để ràng buộc dữ liệu từ yêu cầu HTTP hoặc để định dạng dữ liệu trả về.

## 2.4. HTTP Verbs: Ngôn ngữ của Tương tác API

HTTP Verbs (hoặc HTTP Methods) là xương sống ngữ nghĩa của các API RESTful. Chúng định nghĩa loại hành động mà máy khách muốn thực hiện trên một tài nguyên. Việc sử dụng đúng Verb giúp API dễ hiểu, dễ sử dụng và tuân thủ các chuẩn mực web.

### 2.4.1. Các HTTP Verb Cơ bản và Mục đích Sử dụng

*   **`GET` (Truy xuất/Đọc):**
    *   **Mục đích:** Yêu cầu lấy một hoặc nhiều biểu diễn của tài nguyên từ máy chủ.
    *   **Tính chất:** `Idempotent` (gửi nhiều yêu cầu `GET` liên tiếp với cùng một URI sẽ tạo ra cùng một kết quả trên máy chủ mà không có tác dụng phụ) và `Safe` (không thay đổi trạng thái tài nguyên trên máy chủ).
    *   **Ví dụ URI:**
        *   `GET /api/employees`: Lấy danh sách tất cả nhân viên.
        *   `GET /api/employees/{id}`: Lấy thông tin một nhân viên cụ thể theo ID.
    *   **Phản hồi thành công phổ biến:** `200 OK` (với dữ liệu tài nguyên trong phần thân).

*   **`POST` (Tạo mới):**
    *   **Mục đích:** Gửi dữ liệu đến máy chủ để tạo một tài nguyên mới dưới một tài nguyên mẹ.
    *   **Tính chất:** Không `Idempotent` (gửi nhiều yêu cầu `POST` giống nhau có thể tạo ra nhiều tài nguyên mới).
    *   **Ví dụ URI:**
        *   `POST /api/employees`: Tạo một nhân viên mới (dữ liệu nhân viên được gửi trong body).
    *   **Phản hồi thành công phổ biến:** `201 Created` (với URI của tài nguyên mới được tạo trong header `Location` và dữ liệu tài nguyên trong body).

*   **`PUT` (Cập nhật toàn bộ/Thay thế):**
    *   **Mục đích:** Cập nhật toàn bộ một tài nguyên hiện có trên máy chủ. Nếu tài nguyên không tồn tại, `PUT` có thể tạo mới nó (tùy thuộc vào cách triển khai, nhưng tốt nhất nên dùng `POST` để tạo mới).
    *   **Tính chất:** `Idempotent` (gửi nhiều yêu cầu `PUT` giống nhau sẽ chỉ khiến tài nguyên đạt đến cùng một trạng thái cuối cùng).
    *   **Ví dụ URI:**
        *   `PUT /api/employees/{id}`: Cập nhật thông tin toàn bộ nhân viên có ID `{id}` (dữ liệu cập nhật được gửi trong body).
    *   **Phản hồi thành công phổ biến:** `200 OK` (với dữ liệu tài nguyên đã cập nhật) hoặc `204 No Content` (nếu không cần trả về nội dung).

*   **`DELETE` (Xóa):**
    *   **Mục đích:** Xóa một tài nguyên cụ thể khỏi máy chủ.
    *   **Tính chất:** `Idempotent` (xóa một tài nguyên nhiều lần sẽ không gây ra lỗi nếu tài nguyên đã bị xóa ở lần đầu).
    *   **Ví dụ URI:**
        *   `DELETE /api/employees/{id}`: Xóa nhân viên có ID `{id}`.
    *   **Phản hồi thành công phổ biến:** `200 OK` (nếu có nội dung phản hồi), `202 Accepted` (nếu yêu cầu được chấp nhận nhưng việc xóa chưa hoàn tất), hoặc `204 No Content` (phổ biến nhất, không có nội dung phản hồi).

*   **`PATCH` (Cập nhật một phần):**
    *   **Mục đích:** Cập nhật một phần (một hoặc nhiều trường) của tài nguyên hiện có trên máy chủ.
    *   **Tính chất:** Không `Idempotent` một cách tự nhiên (phụ thuộc vào cách triển khai logic cập nhật một phần).
    *   **Ví dụ URI:**
        *   `PATCH /api/employees/{id}`: Cập nhật chỉ tên hoặc email của nhân viên (chỉ gửi các trường cần cập nhật trong body).
    *   **Phản hồi thành công phổ biến:** `200 OK` hoặc `204 No Content`.

### 2.4.2. Ví dụ Thực tế và Quy ước RESTful

Việc sử dụng đúng HTTP Verbs là một phần quan trọng của việc tuân thủ quy ước RESTful.
*   **Tài nguyên:** `Album`
*   `GET /albums`: Lấy tất cả album.
*   `GET /albums/5`: Lấy album có ID là 5.
*   `POST /albums`: Tạo album mới (thông tin album trong request body).
*   `PUT /albums/5`: Cập nhật toàn bộ album có ID là 5 (thông tin album đầy đủ trong request body).
*   `DELETE /albums/5`: Xóa album có ID là 5.
*   `PATCH /albums/5`: Cập nhật một số trường của album có ID là 5 (chỉ các trường cần cập nhật trong request body).

### 2.4.3. Tư duy Vibe Coding với HTTP Verbs: Thiết kế API trực quan cho cả người và AI

Trong Vibe Coding, chúng ta không chỉ viết code mà còn "cảm nhận" được ý nghĩa của nó. Với HTTP Verbs, điều này có nghĩa là:
*   **Sự rõ ràng ngay lập tức:** Khi Antigravity IDE hoặc một developer khác nhìn vào một endpoint như `DELETE /api/users/{id}`, họ ngay lập tức "cảm nhận" được ý định là xóa một người dùng. Không cần đọc tài liệu dài dòng.
*   **Giảm thiểu lỗi:** Việc sử dụng đúng Verb giúp tránh hiểu lầm và giảm thiểu lỗi trong quá trình tích hợp. Nếu bạn dùng `POST` để xóa, đó là một "bad vibe" vì nó đi ngược lại quy ước.
*   **Dễ dàng tự động hóa:** Đối với Antigravity, sự nhất quán trong việc sử dụng Verbs cho phép nó dễ dàng suy luận các hành động và tự động tạo ra các yêu cầu client hoặc kiểm thử chính xác. AI có thể "đọc vị" được API nhanh chóng hơn.
*   **Tăng cường khả năng khám phá:** Một API tuân thủ Verbs chuẩn mực sẽ dễ dàng được khám phá bởi các công cụ như Swagger, và từ đó, Antigravity có thể dễ dàng khai thác thông tin này để hỗ trợ bạn.

## 2.5. Định tuyến (Routing) trong ASP.NET Core Web API: Ánh xạ Yêu cầu

Định tuyến là cơ chế cốt lõi trong ASP.NET Core Web API, chịu trách nhiệm khớp các yêu cầu HTTP đến với các phương thức hành động (Action Methods) thích hợp trong các Controller. Điều này cho phép bạn định nghĩa các URL rõ ràng và cấu trúc API một cách logic.

### 2.5.1. Cơ chế Hoạt động của Định tuyến và Vai trò của Attribute Routing

1.  **Yêu cầu đến:** Một yêu cầu HTTP (ví dụ: `GET https://localhost:7081/api/products/2`) được gửi đến ứng dụng.
2.  **Middleware Định tuyến:** Middleware định tuyến (`app.MapControllers()` trong `Program.cs`) bắt đầu phân tích URL và HTTP Verb của yêu cầu.
3.  **Khớp mẫu (Pattern Matching):** Hệ thống định tuyến so sánh URL đến với các mẫu đường dẫn (route templates) đã được định nghĩa trên các Controller và Action Methods.
4.  **Xác định Controller và Action:** Khi một mẫu khớp, hệ thống sẽ xác định Controller và Action Method tương ứng.
5.  **Ràng buộc tham số (Parameter Binding):** Nếu URL chứa các giá trị động (ví dụ: `{id}`), hệ thống định tuyến sẽ trích xuất các giá trị này và tự động chuyển đổi (bind) chúng thành các tham số đầu vào của Action Method.
6.  **Thực thi Action Method:** Action Method được gọi để xử lý logic nghiệp vụ và trả về `IActionResult`.

**Attribute Routing:**
Trong ASP.NET Core Web API, chúng ta chủ yếu sử dụng **Attribute Routing**, nơi các mẫu đường dẫn được định nghĩa trực tiếp trên các Controller và Action Methods bằng cách sử dụng các thuộc tính (attributes). Điều này giúp định tuyến trở nên rõ ràng và gần với Action Method mà nó xử lý.

### 2.5.2. Các Thuộc tính Định tuyến Chính

*   **`[ApiController]`:**
    *   **Vị trí:** Đặt ở cấp độ lớp (class level) của Controller.
    *   **Mục đích:** Kích hoạt một loạt các hành vi tiện lợi dành riêng cho Web API, bao gồm:
        *   **Attribute Routing:** Bắt buộc sử dụng định tuyến dựa trên thuộc tính.
        *   **Automatic HTTP 400 responses:** Tự động trả về `HTTP 400 Bad Request` khi lỗi model binding (ví dụ: dữ liệu gửi lên không đúng định dạng).
        *   **Binding source parameter inference:** Tự động suy luận nguồn dữ liệu cho tham số (từ phần thân yêu cầu `[FromBody]`, chuỗi truy vấn `[FromQuery]`, hoặc đường dẫn `[FromRoute]`).
        *   **Validations:** Tích hợp kiểm tra `ModelState` tự động.

*   **`[Route("template")]`:**
    *   **Vị trí:** Có thể đặt ở cấp độ Controller hoặc Action Method.
    *   **Mục đích:** Định nghĩa mẫu đường dẫn cho Controller hoặc Action Method.
    *   **Ví dụ:**
        *   `[Route("api/[controller]")]`: Một mẫu phổ biến ở cấp độ Controller. `[controller]` là một placeholder sẽ được thay thế bằng tên của Controller (bỏ đi hậu tố `Controller`). Ví dụ: `ProductsController` sẽ trở thành `/api/products`.
        *   `[Route("api/mycustomroute")]`: Một đường dẫn cố định.

*   **`[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`, `[HttpPatch]`:**
    *   **Vị trí:** Đặt trên các Action Methods.
    *   **Mục đích:** Chỉ định HTTP Verb mà phương thức đó sẽ xử lý. Bạn cũng có thể cung cấp một mẫu đường dẫn cụ thể cho Action Method, mẫu này sẽ được nối thêm vào mẫu đường dẫn của Controller.
    *   **Ví dụ:**
        *   `[HttpGet]`: Xử lý yêu cầu `GET` đến đường dẫn cơ sở của Controller.
        *   `[HttpGet("{id}")]`: Xử lý yêu cầu `GET` với một tham số `id` từ URL (ví dụ: `/api/products/123`).
        *   `[HttpPost("create")]`: Xử lý yêu cầu `POST` đến `/api/products/create` (nếu Controller có `[Route("api/[controller]")]`).

### 2.5.3. Xây dựng Controller Hoàn chỉnh với Định tuyến và HTTP Verbs

Hãy xem một ví dụ về Controller với các Action Methods minh họa cách định tuyến hoạt động. Chúng ta sẽ sử dụng một danh sách sản phẩm giả lập trong bộ nhớ để đơn giản hóa.

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Linq;

namespace NZWalks.API.Controllers
{
    // Định nghĩa một lớp Product đơn giản để minh họa
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }

    // [ApiController] kích hoạt các tính năng API như attribute routing, automatic model validation, v.v.
    [ApiController]
    // [Route] định nghĩa mẫu đường dẫn cơ sở cho controller này.
    // "api/[controller]" sẽ trở thành "api/products" vì tên lớp là ProductsController.
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        // Danh sách sản phẩm giả lập trong bộ nhớ
        private static List<Product> _products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 1200 },
            new Product { Id = 2, Name = "Mouse", Price = 25 },
            new Product { Id = 3, Name = "Keyboard", Price = 75 }
        };

        /// <summary>
        /// Lấy danh sách tất cả sản phẩm.
        /// GET: api/products
        /// </summary>
        /// <returns>Danh sách các sản phẩm.</returns>
        [HttpGet] // Xử lý yêu cầu HTTP GET không có tham số
        public IActionResult GetProducts()
        {
            // Trả về danh sách tất cả sản phẩm với HTTP 200 OK
            return Ok(_products); 
        }

        /// <summary>
        /// Lấy một sản phẩm cụ thể theo ID.
        /// GET: api/products/{id}
        /// </summary>
        /// <param name="id">ID của sản phẩm cần lấy.</param>
        /// <returns>Sản phẩm nếu tìm thấy, ngược lại là 404 Not Found.</returns>
        [HttpGet("{id}")] // "{id}" là một placeholder cho giá trị động từ URL
        public IActionResult GetProductById([FromRoute] int id) // [FromRoute] là tùy chọn nhưng giúp làm rõ nguồn tham số
        {
            var product = _products.FirstOrDefault(p => p.Id == id);
            if (product == null)
            {
                // Trả về HTTP 404 Not Found nếu không tìm thấy sản phẩm
                return NotFound($"Sản phẩm với ID {id} không tìm thấy."); 
            }
            // Trả về sản phẩm cụ thể với HTTP 200 OK
            return Ok(product); 
        }

        /// <summary>
        /// Tạo một sản phẩm mới.
        /// POST: api/products
        /// </summary>
        /// <param name="newProduct">Đối tượng sản phẩm mới từ request body.</param>
        /// <returns>Sản phẩm đã tạo với HTTP 201 Created.</returns>
        [HttpPost] // Xử lý yêu cầu HTTP POST để tạo sản phẩm mới
        // [FromBody] chỉ định rằng dữ liệu sản phẩm sẽ được lấy từ phần thân yêu cầu (JSON)
        public IActionResult CreateProduct([FromBody] Product newProduct)
        {
            // Kiểm tra tính hợp lệ của Model (được thực hiện tự động nhờ [ApiController])
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState); // Trả về lỗi 400 với chi tiết lỗi
            }
            
            // Gán ID mới (thực tế sẽ do database quản lý)
            newProduct.Id = _products.Any() ? _products.Max(p => p.Id) + 1 : 1; 
            _products.Add(newProduct);

            // Trả về HTTP 201 Created cùng với URI của tài nguyên mới và đối tượng đã tạo
            // nameof(GetProductById) giúp tránh hardcode tên action
            return CreatedAtAction(nameof(GetProductById), new { id = newProduct.Id }, newProduct);
        }

        /// <summary>
        /// Cập nhật toàn bộ một sản phẩm hiện có.
        /// PUT: api/products/{id}
        /// </summary>
        /// <param name="id">ID của sản phẩm cần cập nhật.</param>
        /// <param name="updatedProduct">Đối tượng sản phẩm với thông tin cập nhật từ request body.</param>
        /// <returns>204 No Content nếu thành công, 404 nếu không tìm thấy, 400 nếu dữ liệu không hợp lệ.</returns>
        [HttpPut("{id}")] // Xử lý yêu cầu HTTP PUT để cập nhật sản phẩm hiện có
        public IActionResult UpdateProduct(int id, [FromBody] Product updatedProduct)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            var existingProduct = _products.FirstOrDefault(p => p.Id == id);
            if (existingProduct == null)
            {
                return NotFound($"Sản phẩm với ID {id} không tìm thấy.");
            }
            // Đảm bảo ID trong URI khớp với ID trong body (quy tắc tốt)
            if (id != updatedProduct.Id) 
            {
                return BadRequest("ID trong đường dẫn và ID trong dữ liệu không khớp.");
            }

            existingProduct.Name = updatedProduct.Name;
            existingProduct.Price = updatedProduct.Price;
            // Trả về HTTP 204 No Content cho yêu cầu cập nhật thành công mà không cần trả về dữ liệu
            return NoContent(); 
        }

        /// <summary>
        /// Xóa một sản phẩm.
        /// DELETE: api/products/{id}
        /// </summary>
        /// <param name="id">ID của sản phẩm cần xóa.</param>
        /// <returns>204 No Content nếu thành công, 404 nếu không tìm thấy.</returns>
        [HttpDelete("{id}")] // Xử lý yêu cầu HTTP DELETE để xóa sản phẩm
        public IActionResult DeleteProduct(int id)
        {
            var productToRemove = _products.FirstOrDefault(p => p.Id == id);
            if (productToRemove == null)
            {
                return NotFound($"Sản phẩm với ID {id} không tìm thấy.");
            }
            _products.Remove(productToRemove);
            // Trả về HTTP 204 No Content cho yêu cầu xóa thành công
            return NoContent(); 
        }
    }
}
```

**Giải thích bổ sung về `IActionResult` và Mã trạng thái HTTP:**
*   **`IActionResult`:** Là một interface trong ASP.NET Core MVC/API, đại diện cho kết quả của một Action Method. Nó cho phép Controller trả về nhiều loại phản hồi khác nhau (JSON, XML, file, mã trạng thái HTTP).
*   **Các phương thức trợ giúp phổ biến của `ControllerBase`:**
    *   `Ok(object value)`: Trả về `HTTP 200 OK` với dữ liệu trong body.
    *   `NotFound(object value)`: Trả về `HTTP 404 Not Found` với thông báo lỗi.
    *   `BadRequest(object value)`: Trả về `HTTP 400 Bad Request` với thông báo lỗi (thường dùng khi dữ liệu đầu vào không hợp lệ).
    *   `CreatedAtAction(string actionName, object routeValues, object value)`: Trả về `HTTP 201 Created` khi tạo tài nguyên thành công, kèm theo URI của tài nguyên mới và dữ liệu của nó.
    *   `NoContent()`: Trả về `HTTP 204 No Content` khi yêu cầu thành công nhưng không có nội dung phản hồi (ví dụ: cập nhật hoặc xóa).

Ví dụ trên minh họa cách các thuộc tính định tuyến và HTTP Verbs được sử dụng để định nghĩa rõ ràng các API endpoint. Điều này không chỉ giúp các nhà phát triển dễ dàng hiểu và sử dụng API mà còn cung cấp một "bản đồ" rõ ràng cho Antigravity IDE để phân tích và tương tác với hệ thống của bạn.

## 2.6. Vận hành và Kiểm thử API: Từ Phát triển đến Tương tác

Sau khi đã xây dựng API, việc chạy và kiểm thử là bước không thể thiếu để đảm bảo mọi thứ hoạt động đúng như mong đợi và tương tác chính xác với các client.

### 2.6.1. Xây dựng và Khởi chạy Ứng dụng

1.  **Xây dựng Giải pháp (Build Solution):** Trong Visual Studio, nhấn `Ctrl + Shift + B` hoặc chọn `Build` > `Build Solution`. Bước này sẽ biên dịch mã nguồn, kiểm tra lỗi cú pháp và tạo ra các tệp thực thi.
2.  **Chạy Ứng dụng:** Nhấn nút `Play` màu xanh lá cây trên thanh công cụ của Visual Studio (thường hiển thị tên dự án của bạn, ví dụ: `NZWalks.API`).

Khi ứng dụng chạy, Visual Studio sẽ khởi động máy chủ web Kestrel (máy chủ mặc định của ASP.NET Core) và mở một trình duyệt web. Trình duyệt này sẽ tự động điều hướng đến giao diện **Swagger UI**, nhờ các Middleware `app.UseSwagger()` và `app.UseSwaggerUI()` mà chúng ta đã cấu hình trong `Program.cs`.

### 2.6.2. Sử dụng Swagger UI: Công cụ Khám phá và Kiểm thử Tích hợp

**Swagger UI** (dựa trên tiêu chuẩn OpenAPI Specification) là một công cụ mạnh mẽ và phổ biến để tài liệu hóa các API và cung cấp một giao diện tương tác thân thiện với người dùng để khám phá và kiểm thử các API của bạn trực tiếp từ trình duyệt. Nó được tích hợp sẵn trong các dự án ASP.NET Core Web API mới.

1.  **Giao diện Swagger:** Khi API của bạn chạy, trình duyệt sẽ hiển thị trang Swagger UI. Bạn sẽ thấy danh sách các API endpoint được nhóm theo Controller (ví dụ: `Products`, `WeatherForecast`).
    *   `WeatherForecastController` là một Controller mẫu được tạo sẵn bởi template dự án.
2.  **Khám phá Endpoint:**
    *   Nhấp vào một endpoint (ví dụ: `Products`) để mở rộng nó và xem các phương thức HTTP có sẵn (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`).
    *   Mỗi phương thức sẽ hiển thị thông tin chi tiết:
        *   **Parameters:** Các tham số cần thiết (từ path, query, hoặc request body).
        *   **Request body schema:** Cấu trúc dữ liệu JSON/XML mong đợi cho các yêu cầu `POST`/`PUT`.
        *   **Responses:** Các mã trạng thái HTTP có thể có (ví dụ: `200`, `201`, `400`, `404`) và cấu trúc phản hồi.
3.  **Kiểm thử một Phương thức GET (ví dụ: `GET /api/products`):**
    *   Mở rộng phương thức `GET /api/products`.
    *   Nhấp vào nút **"Try it out"**.
    *   Nhấp vào nút **"Execute"**.
    *   Swagger sẽ gửi một yêu cầu HTTP đến API của bạn và hiển thị phản hồi, bao gồm:
        *   **Request URL:** URL được gọi.
        *   **Response body:** Dữ liệu JSON được trả về.
        *   **Response code:** Mã trạng thái HTTP (ví dụ: `200 OK`).
        *   **Response headers:** Các header HTTP của phản hồi.

#### Tạo và Kiểm thử Controller Tùy chỉnh trong Swagger

Để minh họa cách tạo một endpoint tùy chỉnh và kiểm thử nó, chúng ta sẽ tạo một `StudentsController` đơn giản.

1.  **Tạo một Controller mới:**
    *   Dừng ứng dụng đang chạy trong Visual Studio.
    *   Trong Solution Explorer, nhấp chuột phải vào thư mục `Controllers`.
    *   Chọn `Add` > `Controller...`.
    *   Trong cửa sổ "Add New Scaffolded Item", chọn `API` từ danh sách bên trái.
    *   Chọn `API Controller - Empty` và nhấp `Add`.
    *   Đặt tên cho Controller của bạn, ví dụ: `StudentsController`. (Luôn kết thúc bằng hậu tố `Controller`).
    *   Nhấp `Add`.
2.  **Thêm Action Methods vào Controller:**
    Mở tệp `StudentsController.cs` và thêm mã sau:

    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using System.Collections.Generic;
    using System.Linq; // Thêm using này cho FirstOrDefault

    namespace NZWalks.API.Controllers
    {
        public class Student
        {
            public int Id { get; set; }
            public string Name { get; set; }
            public string Major { get; set; }
        }

        [ApiController]
        [Route("api/[controller]")] // Định tuyến cơ bản: api/students
        public class StudentsController : ControllerBase
        {
            private static List<Student> _students = new List<Student>
            {
                new Student { Id = 1, Name = "Alice", Major = "Computer Science" },
                new Student { Id = 2, Name = "Bob", Major = "Mathematics" },
                new Student { Id = 3, Name = "Charlie", Major = "Physics" }
            };

            /// <summary>
            /// Lấy danh sách tất cả sinh viên.
            /// GET: api/students
            /// </summary>
            [HttpGet]
            public IActionResult GetAllStudents()
            {
                return Ok(_students);
            }

            /// <summary>
            /// Lấy sinh viên theo ID.
            /// GET: api/students/{id}
            /// </summary>
            /// <param name="id">ID của sinh viên.</param>
            [HttpGet("{id}")]
            public IActionResult GetStudentById([FromRoute] int id)
            {
                var student = _students.FirstOrDefault(s => s.Id == id);
                if (student == null)
                {
                    return NotFound($"Sinh viên với ID {id} không tìm thấy.");
                }
                return Ok(student);
            }

            /// <summary>
            /// Tạo sinh viên mới.
            /// POST: api/students
            /// </summary>
            /// <param name="newStudent">Thông tin sinh viên mới.</param>
            [HttpPost]
            public IActionResult CreateStudent([FromBody] Student newStudent)
            {
                if (!ModelState.IsValid || newStudent == null)
                {
                    return BadRequest(ModelState);
                }
                newStudent.Id = _students.Any() ? _students.Max(s => s.Id) + 1 : 1;
                _students.Add(newStudent);
                return CreatedAtAction(nameof(GetStudentById), new { id = newStudent.Id }, newStudent);
            }
        }
    }
    ```
3.  **Chạy lại API và Kiểm thử trong Swagger:**
    *   Chạy lại ứng dụng của bạn.
    *   Trong Swagger UI, bạn sẽ thấy một endpoint mới là `Students`.
    *   Mở rộng phương thức `GET /api/students`, nhấp **"Try it out"** > **"Execute"** để lấy danh sách sinh viên.
    *   Mở rộng phương thức `GET /api/students/{id}`, nhập một ID (ví dụ: `1`) vào trường tham số và nhấn "Execute".
    *   Mở rộng phương thức `POST /api/students`, nhấp **"Try it out"**, sửa đổi JSON mẫu trong trường "Request body" (ví dụ: `{ "name": "Eve", "major": "Art" }`), sau đó nhấn "Execute". Bạn sẽ nhận được `201 Created`. Sau đó thử `GET /api/students` để xem sinh viên mới.

### 2.6.3. Postman: Sức mạnh Kiểm thử API Chuyên nghiệp

**Postman** là một công cụ client REST API phổ biến và mạnh mẽ hơn Swagger UI, cho phép bạn tạo, quản lý và kiểm thử các yêu cầu HTTP phức tạp. Nó cung cấp nhiều tính năng như lưu trữ các bộ sưu tập yêu cầu, kiểm thử tự động, và chia sẻ.

1.  **Tải xuống và cài đặt Postman:** Truy cập [postman.com](https://www.postman.com/) và tải xuống ứng dụng.
2.  **Tạo yêu cầu mới:**
    *   Mở Postman và nhấp vào nút `+` để tạo một tab yêu cầu mới.
    *   **Chọn HTTP Verb:** Từ dropdown menu, chọn `GET`.
    *   **Nhập URL:** Nhập URL của endpoint API của bạn. Ví dụ: `https://localhost:7081/api/students` (nhớ thay thế cổng bằng cổng của dự án bạn từ `launchSettings.json`).
    *   **Gửi yêu cầu:** Nhấp vào nút **"Send"**.
3.  **Xem phản hồi:** Postman sẽ hiển thị phản hồi từ API của bạn, bao gồm mã trạng thái HTTP, thời gian phản hồi, và phần thân phản hồi (body) được định dạng.
4.  **Kiểm thử POST/PUT/DELETE:**
    *   Chọn Verb tương ứng.
    *   Nhập URL.
    *   Đối với `POST`/`PUT`, chọn tab `Body`, chọn `raw` và `JSON` từ dropdown, sau đó nhập dữ liệu JSON vào textarea.
    *   Nhấn **"Send"**.

### 2.6.4. Tối ưu hóa Quy trình Kiểm thử với Antigravity IDE (AI Coding Integration)

Cả Swagger UI và Postman đều là công cụ tuyệt vời, nhưng khi kết hợp với một hệ thống AI Coding như Antigravity IDE, quy trình kiểm thử và phát triển có thể đạt đến một cấp độ mới:

*   **Tự động tạo Test Cases từ Swagger/OpenAPI:** Antigravity có khả năng đọc và hiểu các tệp định nghĩa OpenAPI (JSON/YAML) được tạo bởi Swagger. Từ đó, nó có thể tự động sinh ra:
    *   **Unit tests** cho từng Action Method của Controller.
    *   **Integration tests** để kiểm tra luồng làm việc giữa nhiều endpoint.
    *   **Client-side code** (ví dụ: các hàm JavaScript/TypeScript) để gọi API của bạn.
    Điều này giúp giảm đáng kể thời gian viết test thủ công và đảm bảo độ phủ test cao.

*   **Tạo và Thực thi Yêu cầu API Thông minh:** Thay vì bạn phải nhập URL và body thủ công vào Postman, bạn có thể mô tả ý định của mình cho Antigravity (ví dụ: "tạo một sinh viên mới với tên 'Emma' và chuyên ngành 'Chemistry'"). Antigravity sẽ:
    *   Tìm endpoint `POST /api/students`.
    *   Tạo request body JSON phù hợp.
    *   Thực thi yêu cầu và hiển thị kết quả.
    *   Thậm chí tự động kiểm tra mã trạng thái phản hồi để xác nhận thành công.

*   **Tự động kiểm tra API:** Antigravity có thể được cấu hình để chạy các bộ kiểm thử API định kỳ hoặc sau mỗi lần thay đổi mã, giúp phát hiện sớm các lỗi hồi quy (regression bugs). Nó có thể so sánh phản hồi hiện tại với phản hồi mong đợi và báo cáo sự khác biệt.

*   **Vibe Coding: Phản hồi tức thì và vòng lặp phát triển nhanh:**
    *   Với Antigravity, việc kiểm thử trở thành một phần không thể thiếu của quá trình thiết kế và phát triển. Bạn có thể thay đổi một Controller, yêu cầu Antigravity chạy các test liên quan ngay lập tức, và nhận phản hồi về "vibe" của thay đổi đó – liệu nó có hoạt động đúng không, có phá vỡ điều gì không, có tuân thủ các quy ước không.
    *   Antigravity giúp bạn duy trì một "vibe" tốt cho codebase bằng cách liên tục xác thực hành vi của API, cho phép bạn tập trung vào logic nghiệp vụ mà không lo lắng về các lỗi ẩn.
    *   Sử dụng Antigravity, bạn có thể "cảm nhận" được API của mình có đang hoạt động một cách mạnh mẽ và đáng tin cậy hay không, thông qua khả năng tự động hóa kiểm thử và phân tích phản hồi của nó.

---

## Tóm tắt Chương 2: Nền tảng Kiến trúc ASP.NET Core Web API và Nguyên lý RESTful

*   **Giới thiệu ASP.NET Core Web API & RESTful Service:** ASP.NET Core là framework hiệu suất cao, đa nền tảng để xây dựng dịch vụ HTTP. REST là phong cách kiến trúc dựa trên tài nguyên, không trạng thái, sử dụng HTTP Verbs tiêu chuẩn, và giao diện thống nhất để thiết kế các API dễ hiểu, dễ mở rộng. Các nguyên tắc REST rất quan trọng cho khả năng phân tích và tự động hóa của các hệ thống AI Coding như Antigravity IDE.
*   **Tạo Dự án Mới:** Hướng dẫn từng bước tạo dự án ASP.NET Core Web API bằng Visual Studio, tập trung vào việc chọn mẫu, cấu hình tên dự án, giải pháp, phiên bản .NET (ưu tiên .NET 8 LTS) và loại xác thực (None).
*   **Giải phẫu Cấu trúc Dự án:**
    *   **`.sln` & `.csproj`:** Tổ chức giải pháp và định nghĩa thuộc tính dự án.
    *   **`launchSettings.json`:** Cấu hình khởi chạy trong môi trường phát triển (bao gồm các URL và số cổng).
    *   **`appsettings.json`:** Lưu trữ cấu hình ứng dụng (chuỗi kết nối, log levels) có thể thay đổi theo môi trường.
    *   **`Program.cs`:** Điểm vào của ứng dụng, nơi cấu hình chính diễn ra:
        *   **Cấu hình Dịch vụ (Service Configuration):** Đăng ký các dịch vụ vào container **Dependency Injection (DI)** với các lifetime `Singleton`, `Scoped`, `Transient`. DI giúp giảm ghép nối, tăng khả năng kiểm thử và bảo trì, đồng thời giúp AI dễ dàng phân tích cấu trúc ứng dụng.
        *   **Cấu hình HTTP Request Pipeline:** Định nghĩa chuỗi các **Middleware** (ví dụ: Swagger, HTTPS Redirection, Authorization, MapControllers) sẽ xử lý yêu cầu HTTP. Thứ tự của Middleware là rất quan trọng.
    *   **`Controllers`:** Thư mục chứa các Controller định nghĩa các API endpoint.
    *   **`Models`:** Thư mục (thường được thêm thủ công) chứa các lớp đại diện cho dữ liệu.
*   **HTTP Verbs:** Các động từ HTTP (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`) là ngôn ngữ tiêu chuẩn để mô tả các hành động trên tài nguyên. Mỗi verb có một mục đích cụ thể và tính chất riêng (Idempotent, Safe). Việc sử dụng đúng verb tạo ra một "vibe" rõ ràng cho API, giúp cả con người và AI dễ dàng hiểu và tương tác.
*   **Định tuyến (Routing):** Cơ chế ánh xạ yêu cầu HTTP đến Controller và Action Method phù hợp. Sử dụng **Attribute Routing** với các thuộc tính như `[ApiController]`, `[Route]`, và các thuộc tính HTTP Verb (`[HttpGet]`, `[HttpPost]`, v.v.) để định nghĩa các đường dẫn API. Đã cung cấp ví dụ `ProductsController` hoàn chỉnh.
*   **Vận hành & Kiểm thử API:**
    *   Xây dựng và chạy ứng dụng từ Visual Studio.
    *   Sử dụng **Swagger UI** (tích hợp sẵn) để khám phá tài liệu API và kiểm thử các endpoint một cách tương tác.
    *   Sử dụng **Postman** làm công cụ kiểm thử API chuyên nghiệp hơn.
    *   **Tối ưu hóa với Antigravity IDE:** Nhấn mạnh cách Antigravity có thể tận dụng các định nghĩa API (từ Swagger) và quy ước RESTful để tự động tạo test cases, thực thi yêu cầu thông minh, và cung cấp phản hồi nhanh chóng, hỗ trợ tư duy **Vibe Coding** trong quá trình phát triển API.

<!-- REVIEWED_BY_AGENT -->
