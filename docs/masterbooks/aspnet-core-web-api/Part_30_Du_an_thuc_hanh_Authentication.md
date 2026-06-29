# Phần 30: Dự án thực hành: Xác thực & Ủy quyền (Authentication)

## Giới thiệu tổng quan

Trong kỷ nguyên phát triển API hiện đại, việc bảo mật tài nguyên không chỉ là một yêu cầu mà còn là nền tảng của mọi hệ thống đáng tin cậy. Chương này sẽ tập trung vào cơ chế **Xác thực (Authentication)**, bước then chốt đầu tiên để đảm bảo rằng chỉ những người dùng hợp lệ mới có thể truy cập vào các dịch vụ và dữ liệu của bạn. Chúng ta sẽ cùng nhau xây dựng một hệ thống xác thực dựa trên JSON Web Tokens (JWT) cho RESTful Web API sử dụng ASP.NET Core và Entity Framework Core, đồng thời củng cố các kiến thức về Dependency Injection, Repository Pattern, Controllers và HTTP Verbs đã được giới thiệu trong khóa học.

Mục tiêu cụ thể của chương này bao gồm:

*   Nắm vững luồng hoạt động của cơ chế xác thực sử dụng JWT.
*   Thiết lập và cấu hình JWT Authentication một cách bài bản trong ASP.NET Core Web API.
*   Kiểm tra và xác minh hiệu quả của cơ chế xác thực đã triển khai.
*   Tích hợp tư duy "Vibe Coding" và ứng dụng hệ thống Antigravity IDE để tối ưu hóa quá trình phát triển và bảo mật.

> [!NOTE]
> Khóa học này sử dụng ASP.NET Core (C#) và Entity Framework Core. Các khái niệm về Dependency Injection, Repository Pattern và Controllers sẽ được áp dụng xuyên suốt quá trình xây dựng API, đặc biệt trong việc tổ chức mã nguồn và xử lý logic nghiệp vụ liên quan đến xác thực.

## 1. Nền tảng bảo mật API: Xác thực & Ủy quyền

Trước khi đi sâu vào chi tiết kỹ thuật, việc hiểu rõ các khái niệm cốt lõi về bảo mật là vô cùng quan trọng.

### 1.1. Xác thực (Authentication) là gì?

**Xác thực** là quá trình kiểm tra và xác minh danh tính của một cá nhân, hệ thống hoặc dịch vụ. Mục đích là để đảm bảo rằng đối tượng đang cố gắng truy cập tài nguyên thực sự là người mà họ tuyên bố. Trong bối cảnh Web API, điều này thường liên quan đến việc người dùng cung cấp thông tin đăng nhập (như tên người dùng và mật khẩu, hoặc mã thông báo) và hệ thống backend kiểm tra tính hợp lệ của thông tin đó, thường là so sánh với dữ liệu được lưu trữ an toàn trong cơ sở dữ liệu.

*Ví dụ thực tế:* Khi bạn đăng nhập vào tài khoản ngân hàng trực tuyến, hệ thống yêu cầu tên đăng nhập và mật khẩu của bạn. Quá trình kiểm tra thông tin này để xác định bạn có phải là chủ tài khoản hợp lệ hay không chính là xác thực.

### 1.2. Ủy quyền (Authorization) là gì?

**Ủy quyền** là quá trình xác định xem một danh tính đã được xác thực có quyền truy cập vào một tài nguyên hoặc thực hiện một hành động cụ thể hay không. Xác thực trả lời câu hỏi "Bạn là ai?", còn ủy quyền trả lời câu hỏi "Bạn có quyền làm gì?".

*Ví dụ thực tế:* Sau khi ngân hàng xác thực bạn là chủ tài khoản (authentication), hệ thống sẽ kiểm tra xem bạn có quyền thực hiện giao dịch chuyển tiền, xem số dư hay thay đổi thông tin cá nhân (authorization) hay không, dựa trên vai trò hoặc quyền hạn được cấp cho tài khoản của bạn.

> [!TIP]
> **Mối quan hệ giữa Authentication và Authorization:**
> Hai khái niệm này luôn đi đôi với nhau và là hai giai đoạn riêng biệt trong quy trình bảo mật. Xác thực luôn phải diễn ra trước ủy quyền. Một người dùng không được xác thực sẽ không bao giờ được phép ủy quyền cho bất kỳ tài nguyên nào.

## 2. JSON Web Tokens (JWT): Cơ chế và Cấu trúc

Để triển khai xác thực trong RESTful API, chúng ta sẽ sử dụng JSON Web Tokens (JWT), một tiêu chuẩn mở (`RFC 7519`) được thiết kế nhỏ gọn, tự chứa (self-contained) và an toàn để truyền tải thông tin giữa các bên dưới dạng đối tượng JSON.

### 2.1. JWT là gì và tại sao lại sử dụng?

JWT là một mã thông báo (token) được sử dụng để xác thực người dùng sau khi họ đã đăng nhập thành công. Thay vì phải gửi lại thông tin đăng nhập trong mỗi yêu cầu, client chỉ cần gửi JWT. API sẽ sử dụng JWT này để xác minh danh tính người dùng mà không cần truy vấn lại cơ sở dữ liệu cho mỗi yêu cầu, giúp giảm tải cho server và cải thiện hiệu suất.

**Ưu điểm của JWT:**

*   **Không trạng thái (Stateless):** Server không cần lưu trữ thông tin session của người dùng. Mọi thông tin cần thiết đều nằm trong token, giúp API dễ dàng mở rộng (scalability).
*   **Bảo mật:** Sử dụng chữ ký số để đảm bảo tính toàn vẹn và xác thực nguồn gốc.
*   **Tự chứa (Self-contained):** Chứa đủ thông tin về người dùng (claims) mà không cần server phải thực hiện các truy vấn bổ sung.
*   **Nhỏ gọn:** Kích thước nhỏ, dễ dàng truyền tải qua HTTP headers.

### 2.2. Cấu trúc của một JWT

Một JWT bao gồm ba phần được mã hóa Base64Url và phân tách bằng dấu chấm (`.`): **Header**, **Payload** và **Signature**.

```
Header.Payload.Signature
```

#### 2.2.1. Header (Tiêu đề)

Header là một đối tượng JSON chứa thông tin về loại token (luôn là `JWT`) và thuật toán mã hóa được sử dụng để ký token. Các thuật toán phổ biến bao gồm HS256 (HMAC SHA256) hoặc RS256 (RSA SHA256).

*Ví dụ Header:*
```json
{
  "alg": "HS256", // Thuật toán ký: HMAC SHA256
  "typ": "JWT"    // Loại token: JSON Web Token
}
```
Sau khi mã hóa Base64Url, header này sẽ trở thành phần đầu tiên của JWT.

#### 2.2.2. Payload (Nội dung / Claims)

Payload cũng là một đối tượng JSON chứa các "claims" (khẳng định) về người dùng và các dữ liệu khác. Claims là những thông tin mà bạn muốn truyền tải về người dùng hoặc phiên làm việc. Có ba loại claims chính:

*   **Registered Claims:** Các claims tiêu chuẩn được định nghĩa bởi JWT, không bắt buộc nhưng được khuyến nghị để đảm bảo khả năng tương thích. Ví dụ:
    *   `iss` (issuer): Thực thể phát hành token.
    *   `exp` (expiration time): Thời gian token hết hạn (dạng Unix timestamp).
    *   `sub` (subject): Chủ thể của token (thường là ID người dùng).
    *   `aud` (audience): Đối tượng mà token này dành cho.
    *   `iat` (issued at): Thời gian token được phát hành.
*   **Public Claims:** Các claims được định nghĩa bởi người dùng JWT, nhưng cần được đăng ký để tránh xung đột tên.
*   **Private Claims:** Các claims tùy chỉnh được định nghĩa cho một ứng dụng cụ thể, không cần đăng ký. Đây thường là nơi bạn đặt các thông tin như `userId`, `username`, `roles`, v.v.

*Ví dụ Payload:*
```json
{
  "sub": "user_123",
  "name": "Nguyen Van A",
  "email": "vana@example.com",
  "roles": ["Admin", "Editor"],
  "exp": 1678886400 // Thời gian hết hạn (ví dụ: 15/03/2023 00:00:00 GMT)
}
```
Sau khi mã hóa Base64Url, payload này sẽ tạo thành phần thứ hai của JWT.

#### 2.2.3. Signature (Chữ ký)

Chữ ký được tạo bằng cách lấy Header và Payload (đã mã hóa Base64Url) và ký chúng bằng một khóa bí mật (secret key) của máy chủ và thuật toán được chỉ định trong Header.

```
Signature = HMACSHA256(
    Base64UrlEncode(Header) + "." +
    Base64UrlEncode(Payload),
    secret_key
)
```
*   **Mục đích:** Chữ ký này đảm bảo tính toàn vẹn của token. Nếu bất kỳ phần nào của Header hoặc Payload bị thay đổi sau khi token được phát hành, chữ ký sẽ không khớp khi được xác minh, và token sẽ bị coi là không hợp lệ. Điều này ngăn chặn việc giả mạo hoặc thay đổi dữ liệu trong token.
*   **Khóa bí mật (`secret_key`):** Đây là yếu tố quan trọng nhất để bảo mật JWT. Nó phải là một chuỗi ngẫu nhiên, đủ dài, và tuyệt đối KHÔNG được tiết lộ. Kẻ tấn công có khóa bí mật có thể tạo ra các JWT giả mạo hợp lệ và truy cập trái phép vào hệ thống.

## 3. Quy trình Xác thực JWT trong RESTful API

Hiểu rõ luồng hoạt động là chìa khóa để triển khai đúng đắn. Hãy xem xét quy trình xác thực JWT điển hình trong một RESTful Web API:

1.  **Đăng nhập (Client -> API):** Người dùng (hoặc ứng dụng client) gửi yêu cầu đăng nhập đến API, cung cấp thông tin định danh (ví dụ: tên người dùng và mật khẩu). Đây thường là một yêu cầu HTTP `POST` đến một endpoint như `/api/auth/login`.

2.  **Xác minh thông tin đăng nhập (API):** API nhận thông tin đăng nhập, sau đó sử dụng các dịch vụ (thường được inject qua Dependency Injection) để kiểm tra tính hợp lệ của thông tin này. Điều này bao gồm việc tra cứu người dùng trong cơ sở dữ liệu (sử dụng Repository Pattern) và xác minh mật khẩu (thường là so sánh hash mật khẩu).

3.  **Tạo JWT (API -> Client):**
    *   Nếu thông tin đăng nhập hợp lệ, API sẽ tạo một mã thông báo JWT. Mã thông báo này chứa các claims về người dùng (ví dụ: ID, vai trò, thời gian hết hạn) và được ký bằng khóa bí mật của máy chủ.
    *   API trả về mã thông báo JWT này cho client, thường là trong phần thân phản hồi của HTTP `200 OK`.

4.  **Lưu trữ JWT (Client):** Client (ứng dụng web, di động, hoặc desktop) nhận và lưu trữ mã thông báo JWT. Các vị trí lưu trữ phổ biến cho ứng dụng web là `Local Storage` hoặc `Session Storage` (cần cân nhắc về bảo mật XSS), hoặc `HTTP-only cookies` (giúp chống XSS nhưng khó sử dụng cho SPA).

5.  **Gửi yêu cầu tài nguyên được bảo vệ (Client -> API):** Khi client muốn truy cập các tài nguyên yêu cầu xác thực, nó sẽ đính kèm mã thông báo JWT vào mỗi yêu cầu HTTP tiếp theo. Theo quy ước, JWT được gửi trong tiêu đề `Authorization` với tiền tố `Bearer`.
    ```
    Authorization: Bearer <your_jwt_token_here>
    ```
    Đây là cách API biết ai đang gửi yêu cầu mà không cần đăng nhập lại.

6.  **Xác thực JWT (API):** API nhận yêu cầu, và middleware xác thực (đã được cấu hình trong `Program.cs`) sẽ tự động trích xuất mã thông báo JWT từ tiêu đề `Authorization`. Sau đó, middleware thực hiện các kiểm tra nghiêm ngặt:
    *   **Tính hợp lệ của chữ ký:** Xác minh chữ ký của token bằng cách sử dụng khóa bí mật đã biết. Nếu chữ ký không khớp, token đã bị giả mạo và không hợp lệ.
    *   **Thời gian hết hạn (`exp`):** Kiểm tra xem token còn hiệu lực hay không.
    *   **Issuer (`iss`) và Audience (`aud`):** Đảm bảo rằng token được phát hành bởi máy chủ hợp lệ và dành cho ứng dụng này.
    *   **Cấu trúc token:** Đảm bảo token có định dạng JWT hợp lệ.
    *   Nếu tất cả các kiểm tra đều vượt qua, middleware sẽ tạo một `ClaimsPrincipal` từ các claims trong token và gán nó vào `HttpContext.User`, xác định danh tính của người dùng hiện tại cho các thành phần tiếp theo trong pipeline (như Controllers).

7.  **Phản hồi tài nguyên (API -> Client):**
    *   **Nếu JWT hợp lệ:** API đã xác định danh tính của người dùng. Tiếp theo, middleware ủy quyền (`UseAuthorization()`) sẽ kiểm tra xem người dùng này có quyền truy cập vào tài nguyên được yêu cầu hay không (dựa trên các vai trò/claims trong `ClaimsPrincipal`). Nếu có quyền, API sẽ tiếp tục xử lý yêu cầu bằng cách gọi action method trong Controller và trả về dữ liệu hoặc thực hiện hành động được yêu cầu.
    *   **Nếu JWT không hợp lệ:** API sẽ từ chối yêu cầu và trả về mã trạng thái HTTP `401 Unauthorized` (không được xác thực) hoặc `403 Forbidden` (không có quyền, nếu token hợp lệ nhưng không đủ quyền).

## 4. Triển khai Xác thực JWT với ASP.NET Core

Bây giờ, chúng ta sẽ bắt tay vào triển khai luồng xác thực JWT vào ứng dụng ASP.NET Core Web API của mình.

### 4.1. Cài đặt các gói NuGet cần thiết

Để tích hợp JWT Authentication vào ASP.NET Core, chúng ta cần cài đặt các gói NuGet sau. Mở cửa sổ "Manage NuGet Packages" cho dự án của bạn (chuột phải vào `Dependencies` -> `Manage NuGet Packages...`) và tìm kiếm, cài đặt:

1.  **`Microsoft.AspNetCore.Authentication.JwtBearer`**: Đây là gói chính, cung cấp các dịch vụ và middleware cần thiết để hỗ trợ xác thực JWT Bearer token trong ASP.NET Core. Nó là cầu nối giữa tiêu chuẩn JWT và pipeline xác thực của ASP.NET Core.
2.  **`Microsoft.IdentityModel.Tokens`**: Chứa các lớp cơ bản để tạo và xác thực các token bảo mật. Gói này cung cấp các thuật toán mã hóa, các lớp `SecurityKey` (như `SymmetricSecurityKey`) và các tham số xác thực token (`TokenValidationParameters`).
3.  **`System.IdentityModel.Tokens.Jwt`**: Cung cấp các lớp để làm việc với JWT, bao gồm việc đọc, ghi và tạo JWT từ phía server.

> [!NOTE]
> **Vibe Coding với Antigravity IDE:**
> Với Antigravity IDE, bạn có thể không cần phải mở cửa sổ NuGet Package Manager theo cách thủ công. Thay vào đó, bạn có thể sử dụng tính năng "Vibe Coding" của Antigravity. Đơn giản chỉ cần "nói" ý định của bạn (ví dụ: "Cài đặt gói NuGet cho JWT Authentication") hoặc gõ lệnh `dotnet add package` trong terminal tích hợp. Antigravity sẽ tự động nhận diện các gói cần thiết và thực thi lệnh thêm gói, thậm chí có thể đề xuất các phiên bản tương thích nhất. Đây là một ví dụ về cách Antigravity tự động hóa các tác vụ lặp đi lặp lại, cho phép bạn tập trung vào logic cốt lõi.

### 4.2. Cấu hình cài đặt JWT trong `appsettings.json`

Để quản lý các thông số cấu hình JWT một cách linh hoạt và an toàn, chúng ta sẽ lưu chúng trong tệp `appsettings.json`. Điều này cho phép dễ dàng thay đổi cấu hình giữa các môi trường (phát triển, thử nghiệm, sản xuất) mà không cần biên dịch lại mã nguồn.

Thêm một mục `JWT` mới vào `appsettings.json` của bạn:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "NZWalksConnectionString": "Server=(localdb)\\mssqllocaldb;Database=NZWalksDb;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "JWT": {
    "Key": "ThisIsMySuperSecretKeyForJWTAuthenticationAndItShouldBeAtLeast32CharactersLongAndRandom", // KHÓA BÍ MẬT: PHẢI LÀ MỘT CHUỖI DÀI, NGẪU NHIÊN VÀ BÍ MẬT
    "Issuer": "https://localhost:7001", // URL của API phát hành token
    "Audience": "https://localhost:7001" // URL của client hoặc API mà token này dành cho
  }
}
```

*   **`Key`**: Đây là **khóa bí mật (secret key)** được sử dụng để ký và xác minh chữ ký của JWT.
    *   **Quan trọng nhất:** Khóa này phải là một chuỗi dài, ngẫu nhiên, và tuyệt đối KHÔNG được chia sẻ. Độ dài khuyến nghị ít nhất 32 ký tự để đảm bảo an toàn với thuật toán HS256 (256 bit).
    *   Nếu kẻ tấn công có khóa này, họ có thể tạo ra các JWT giả mạo hợp lệ và truy cập trái phép vào API của bạn.
    *   Trong môi trường sản xuất, khóa này KHÔNG BAO GIỜ được lưu trữ trực tiếp trong `appsettings.json`. Thay vào đó, nó nên được lưu trữ trong các biến môi trường, Azure Key Vault, AWS Secrets Manager, hoặc các dịch vụ quản lý bí mật an toàn khác. Điều này giúp tách biệt bí mật khỏi mã nguồn và cấu hình.
*   **`Issuer`**: Là URL của ứng dụng hoặc máy chủ phát hành JWT này. Trong trường hợp này, đó là URL của chính API của chúng ta. Middleware xác thực sẽ kiểm tra xem `iss` claim trong token có khớp với giá trị này hay không.
*   **`Audience`**: Là URL của ứng dụng hoặc máy chủ mà JWT này được cấp quyền truy cập. Trong ví dụ này, `Issuer` và `Audience` là giống nhau vì API vừa phát hành token vừa là đối tượng sử dụng token. Middleware xác thực sẽ kiểm tra xem `aud` claim trong token có khớp với giá trị này hay không.

> [!CAUTION]
> **Bảo mật khóa bí mật:**
> Chuỗi ví dụ cho `Key` chỉ mang tính minh họa. Trong thực tế, bạn nên sử dụng một công cụ tạo chuỗi ngẫu nhiên mạnh (ví dụ: `Guid.NewGuid().ToString()` hoặc các thư viện tạo khóa mật mã) để tạo một khóa thực sự ngẫu nhiên và phức tạp.
>
> **Lấy URL chính xác:**
> Để lấy URL chính xác cho `Issuer` và `Audience` của ứng dụng API của bạn, bạn có thể kiểm tra trong tệp `Properties/launchSettings.json`. Tìm hồ sơ khởi chạy bạn đang sử dụng (thường là hồ sơ với tên dự án của bạn) và sao chép giá trị từ `applicationUrl` (ví dụ: `https://localhost:7001`).

### 4.3. Đăng ký dịch vụ xác thực trong `Program.cs`

Tệp `Program.cs` là nơi chúng ta cấu hình các dịch vụ (services) và middleware cho ứng dụng ASP.NET Core. Đây là trái tim của quá trình thiết lập.

#### 4.3.1. Đăng ký các dịch vụ xác thực

Trong phần đăng ký dịch vụ của `Program.cs` (trước `builder.Build()`), thêm cấu hình xác thực JWT:

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;
// ... (các using statements khác)

var builder = WebApplication.CreateBuilder(args);

// ... (các dịch vụ khác như DbContext, Repository, AutoMapper, v.v. được đăng ký qua Dependency Injection)

// Thêm dịch vụ xác thực JWT
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            // Cấu hình các tham số để xác thực JWT đến
            // 1. Xác thực Issuer (người phát hành token)
            ValidateIssuer = true,
            // 2. Xác thực Audience (đối tượng nhận token)
            ValidateAudience = true,
            // 3. Xác thực thời gian sống của token (hết hạn)
            ValidateLifetime = true,
            // 4. Xác thực khóa ký của token
            ValidateIssuerSigningKey = true,

            // Thiết lập các giá trị hợp lệ từ cấu hình appsettings.json
            ValidIssuer = builder.Configuration["JWT:Issuer"],
            ValidAudience = builder.Configuration["JWT:Audience"],
            // Thiết lập khóa ký của Issuer, sử dụng SymmetricSecurityKey từ khóa bí mật
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["JWT:Key"]))
        };
    });

// ... (builder.Services.AddControllers(), AddSwaggerGen(), v.v.)

var app = builder.Build();

// ... (cấu hình middleware khác)
```

**Giải thích chi tiết về cấu hình:**

*   `builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)`:
    *   Đây là phương thức mở rộng để đăng ký dịch vụ xác thực vào bộ chứa Dependency Injection của ASP.NET Core.
    *   `JwtBearerDefaults.AuthenticationScheme` là một hằng số chuỗi ("Bearer") được định nghĩa sẵn, chỉ định rằng lược đồ xác thực mặc định cho ứng dụng này sẽ là JWT Bearer token. Khi một yêu cầu đến, middleware sẽ tìm kiếm token trong tiêu đề `Authorization: Bearer <token>`.
*   `.AddJwtBearer(options => { ... })`:
    *   Phương thức này cho phép chúng ta cấu hình các tùy chọn cụ thể cho JWT Bearer authentication. Đối tượng `options` chứa các thuộc tính để tùy chỉnh hành vi của middleware.
*   `options.TokenValidationParameters = new TokenValidationParameters { ... }`:
    *   Đây là đối tượng trung tâm, chứa tất cả các tham số mà middleware xác thực sẽ sử dụng để kiểm tra tính hợp lệ của một JWT đến.
    *   `ValidateIssuer = true`: Khi được đặt là `true`, middleware yêu cầu rằng JWT phải có claim `iss` (issuer), và giá trị của claim này phải khớp với giá trị được cung cấp trong `ValidIssuer`. Điều này ngăn chặn việc chấp nhận token được phát hành bởi các nguồn không đáng tin cậy.
    *   `ValidateAudience = true`: Tương tự, middleware yêu cầu JWT phải có claim `aud` (audience), và giá trị này phải khớp với `ValidAudience`. Điều này đảm bảo rằng token chỉ được sử dụng bởi ứng dụng hoặc dịch vụ mà nó được cấp quyền.
    *   `ValidateLifetime = true`: Middleware sẽ kiểm tra các claim `nbf` (not before) và `exp` (expiration time) của token. Nếu token được sử dụng trước thời gian `nbf` hoặc sau thời gian `exp`, nó sẽ bị coi là không hợp lệ. Điều này ngăn chặn các cuộc tấn công phát lại (replay attacks) với các token đã hết hạn hoặc chưa có hiệu lực.
    *   `ValidateIssuerSigningKey = true`: Đây là kiểm tra quan trọng nhất về bảo mật. Middleware sẽ sử dụng `IssuerSigningKey` được cấu hình để giải mã và xác minh chữ ký của token. Nếu chữ ký không khớp, token đã bị giả mạo hoặc thay đổi trong quá trình truyền tải, và sẽ bị từ chối.
    *   `ValidIssuer`, `ValidAudience`: Các giá trị này được đọc từ phần `JWT` trong `appsettings.json` bằng cách sử dụng `builder.Configuration["JWT:Issuer"]` và `builder.Configuration["JWT:Audience"]`. `builder.Configuration` cung cấp quyền truy cập vào cấu hình ứng dụng.
    *   `IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["JWT:Key"]))`:
        *   Đây là phần cốt lõi để xác minh chữ ký. Chúng ta tạo một `SymmetricSecurityKey` từ khóa bí mật (`JWT:Key`) đã lưu trong `appsettings.json`.
        *   `Encoding.UTF8.GetBytes()` chuyển chuỗi khóa (string) thành mảng byte, định dạng cần thiết cho `SymmetricSecurityKey`.
        *   `SymmetricSecurityKey` được sử dụng khi cả việc ký và xác minh token đều sử dụng cùng một khóa bí mật (khóa đối xứng). Đây là phương pháp phổ biến và hiệu quả cho các ứng dụng API tự phát hành và xác thực token.

> [!NOTE]
> **Tích hợp sâu sắc với Dependency Injection:**
> Việc sử dụng `builder.Configuration["JWT:Key"]` là một ví dụ về cách cấu hình được inject vào ứng dụng. Trong các phần sau của khóa học, bạn sẽ thấy cách các dịch vụ (Service, Repository) được đăng ký và inject vào Controller thông qua Dependency Injection. Cơ chế xác thực này cũng là một dịch vụ được cấu hình và sử dụng trong toàn bộ ứng dụng thông qua DI.

#### 4.3.2. Thêm middleware xác thực vào pipeline xử lý yêu cầu

Sau khi đã đăng ký các dịch vụ, chúng ta cần thêm middleware xác thực vào pipeline xử lý yêu cầu của ASP.NET Core. Điều này đảm bảo rằng mỗi yêu cầu đi qua hệ thống sẽ được kiểm tra xác thực trước khi đến các bộ điều khiển và logic nghiệp vụ.

Trong phần cấu hình middleware của `Program.cs` (sau `app.UseHttpsRedirection()` và trước `app.UseAuthorization()`), thêm các dòng sau:

```csharp
// ... (các cấu hình middleware khác)

app.UseHttpsRedirection();

// Thêm middleware xác thực
// Đây là nơi HttpContext.User được gán một ClaimsPrincipal nếu token hợp lệ
app.UseAuthentication();

// Thêm middleware ủy quyền (sẽ được khám phá chi tiết sau)
// Middleware này sử dụng HttpContext.User đã được xác thực bởi UseAuthentication
app.UseAuthorization();

app.MapControllers(); // Định tuyến các yêu cầu đến Controller

app.Run();
```

> [!TIP]
> **Thứ tự của Middleware là Tối quan trọng:**
> 1.  `app.UseAuthentication()` phải được gọi **TRƯỚC** `app.UseAuthorization()`.
>     *   Middleware xác thực (`UseAuthentication`) có nhiệm vụ kiểm tra token (nếu có), xác định danh tính của người dùng và tạo ra một đối tượng `ClaimsPrincipal` (đại diện cho người dùng đã xác thực, chứa các claims như ID, vai trò). Đối tượng `ClaimsPrincipal` này sau đó được gán vào `HttpContext.User`.
> 2.  Middleware ủy quyền (`app.UseAuthorization()`) sau đó sẽ sử dụng danh tính đã được xác thực này (từ `HttpContext.User`) để kiểm tra xem người dùng có quyền truy cập vào tài nguyên được yêu cầu hay không, dựa trên các thuộc tính `[Authorize]` trên Controller hoặc action method.
> Nếu thứ tự bị đảo ngược, middleware ủy quyền sẽ chạy trước khi danh tính người dùng được xác định, dẫn đến việc ủy quyền không thể hoạt động đúng cách và có thể từ chối mọi yêu cầu hoặc chấp nhận sai.

## 5. Kiểm tra Cơ chế Xác thực: Phản hồi 401 Unauthorized

Sau khi đã thiết lập xác thực, chúng ta cần kiểm tra xem nó có hoạt động như mong đợi hay không. Chúng ta sẽ sử dụng thuộc tính `[Authorize]` trên một bộ điều khiển và cố gắng truy cập nó mà không cung cấp JWT hợp lệ.

### 5.1. Áp dụng thuộc tính `[Authorize]`

Thuộc tính `[Authorize]` là một filter trong ASP.NET Core cho phép bạn chỉ định rằng một bộ điều khiển (Controller) hoặc một hành động (Action Method) cụ thể yêu cầu xác thực.

Hãy thêm thuộc tính này vào bộ điều khiển `RegionsController.cs` của chúng ta:

```csharp
using Microsoft.AspNetCore.Authorization; // Đảm bảo có using này
using Microsoft.AspNetCore.Mvc;
// ... (các using statements khác)

namespace NZWalks.API.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    [Authorize] // Yêu cầu xác thực cho tất cả các endpoint trong Controller này
    public class RegionsController : ControllerBase
    {
        // Constructor và các dịch vụ được inject qua DI (ví dụ: IRegionRepository, IMapper)
        private readonly IRegionRepository regionRepository;
        private readonly IMapper mapper;

        public RegionsController(IRegionRepository regionRepository, IMapper mapper)
        {
            this.regionRepository = regionRepository;
            this.mapper = mapper;
        }

        // Các action methods: GetAll, GetById, Add, Update, Delete
        // Tất cả đều yêu cầu xác thực do [Authorize] ở cấp Controller
        [HttpGet]
        public async Task<IActionResult> GetAll()
        {
            // Logic để lấy tất cả các vùng
            var regionsDomain = await regionRepository.GetAllAsync();
            var regionsDto = mapper.Map<List<RegionDto>>(regionsDomain);
            return Ok(regionsDto);
        }

        // ... (các action methods khác như GetById, Add, Update, Delete)
        // Ví dụ:
        // [HttpGet]
        // [Route("{id:Guid}")]
        // public async Task<IActionResult> GetById([FromRoute] Guid id) { ... }
    }
}
```

Bằng cách đặt `[Authorize]` ở cấp độ lớp `RegionsController`, chúng ta đang thông báo rằng **TẤT CẢ** các điểm cuối (endpoints) bên trong bộ điều khiển này (ví dụ: `GET /api/regions`, `POST /api/regions`, v.v.) đều yêu cầu một JWT hợp lệ để truy cập. Nếu không có JWT hoặc JWT không hợp lệ, yêu cầu sẽ bị từ chối ngay lập tức bởi middleware xác thực.

### 5.2. Quan sát truy cập trái phép (Mã trạng thái 401)

Bây giờ, hãy chạy ứng dụng API của bạn và thử truy cập một điểm cuối của `RegionsController` thông qua Swagger UI hoặc một công cụ như Postman, mà không cung cấp bất kỳ token xác thực nào.

1.  Khởi động ứng dụng API của bạn (thường là `dotnet run` hoặc F5 trong Visual Studio).
2.  Mở Swagger UI trong trình duyệt của bạn (thường là `https://localhost:7xxx/swagger`).
3.  Tìm đến `Regions` Controller trong danh sách các API.
4.  Chọn bất kỳ hành động nào, ví dụ: `GET /api/Regions` (GetAllRegions).
5.  Nhấp vào "Try it out" và sau đó "Execute".

> [!NOTE]
> Bạn sẽ nhận được một phản hồi HTTP với mã trạng thái `401 Unauthorized`.

**Giải thích về phản hồi `401 Unauthorized`:**

*   **`401 Unauthorized`**: Mã trạng thái HTTP này có nghĩa là yêu cầu của bạn thiếu thông tin xác thực hợp lệ cho tài nguyên mục tiêu. API đang nói rằng "Tôi không biết bạn là ai, vui lòng cung cấp thông tin xác thực hợp lệ."
*   **Cơ chế chặn ở tầng Middleware:** Điều quan trọng là yêu cầu này bị chặn **NGAY LẬP TỨC** bởi middleware xác thực (`app.UseAuthentication()`), TRƯỚC KHI nó kịp chạm đến logic bên trong action method (`GetAll()` trong `RegionsController`). Bạn có thể đặt một breakpoint ở dòng đầu tiên của phương thức `GetAll()` và sẽ thấy rằng breakpoint đó không bao giờ được kích hoạt khi bạn nhận được `401`. Điều này cho thấy cơ chế bảo mật đã hoạt động hiệu quả ở tầng middleware, ngăn chặn truy cập trái phép vào logic nghiệp vụ của bạn và giảm thiểu rủi ro.
*   Nếu bạn thử các hành động khác như `POST /api/Regions` (thêm một vùng mới), bạn cũng sẽ nhận được `401 Unauthorized`, bất kể dữ liệu bạn gửi có hợp lệ hay không. Điều này nhấn mạnh rằng xác thực là lớp bảo vệ đầu tiên và quan trọng nhất.

### 5.3. Các bước tiếp theo

Việc nhận được mã trạng thái `401 Unauthorized` là dấu hiệu cho thấy cơ chế xác thực của chúng ta đã được thiết lập thành công và đang hoạt động đúng cách. Bước tiếp theo là tạo một cơ chế để người dùng có thể đăng ký và đăng nhập để nhận được JWT hợp lệ. Sau đó, chúng ta sẽ hướng dẫn cách đính kèm JWT này vào các yêu cầu để truy cập thành công các tài nguyên được bảo vệ.

## 6. Tối ưu hóa quy trình với Antigravity IDE và Vibe Coding

Trong khóa học này, bạn đang trực tiếp sử dụng hệ thống Antigravity IDE, một môi trường Agentic AI siêu việt được thiết kế để nâng cao năng suất và chất lượng mã hóa. Hãy cùng khám phá cách áp dụng tư duy "Vibe Coding" và các khả năng của Antigravity để tối ưu hóa quá trình triển khai và kiểm thử xác thực JWT.

### 6.1. Vibe Coding là gì trong bối cảnh Antigravity IDE?

"Vibe Coding" là một phương pháp lập trình trực quan, có định hướng ý định, nơi lập trình viên truyền đạt "ý tưởng" hoặc "mục tiêu" của mình cho AI (Antigravity), và AI sẽ chủ động phân tích, lập kế hoạch, đề xuất, hoặc thậm chí tự động tạo và thực thi mã để đạt được mục tiêu đó. Nó không chỉ là tự động hoàn thành mã, mà là một quá trình cộng tác sâu sắc, nơi Antigravity hiểu "cảm nhận" (vibe) của bạn về giải pháp.

Trong ngữ cảnh thiết lập JWT Authentication:

*   Thay vì nhớ từng dòng cấu hình `TokenValidationParameters`, bạn có thể "vibe" cho Antigravity: "Tôi muốn thiết lập JWT Authentication với validation cho issuer, audience, lifetime và signing key."
*   Antigravity sẽ tự động gợi ý, hoặc thậm chí tạo ra khối mã `AddJwtBearer` đầy đủ với các tham số cần thiết, lấy giá trị từ `appsettings.json`.

### 6.2. Ứng dụng Antigravity IDE vào triển khai JWT Authentication

1.  **Tạo Khóa Bí mật An toàn (`appsettings.json`):**
    *   **Thử thách:** Tạo một khóa bí mật đủ dài và ngẫu nhiên là rất quan trọng. Việc tự gõ một chuỗi dài dễ dẫn đến lỗi hoặc độ phức tạp thấp.
    *   **Antigravity Solution (Vibe Coding):** Bạn có thể chỉ định: "Antigravity, hãy tạo một khóa bí mật an toàn 32 ký tự cho JWT trong `appsettings.json` và chèn nó vào mục `JWT:Key`."
    *   **Cơ chế Antigravity:** Antigravity có thể tự chạy một script ngầm (`Guid.NewGuid().ToString()` hoặc một thuật toán tạo khóa mạnh hơn), đọc tệp `appsettings.json`, cập nhật giá trị và lưu lại. Nó có thể thậm chí cảnh báo bạn về tầm quan trọng của việc không chia sẻ khóa này trong môi trường sản xuất.

2.  **Scaffolding Cấu hình `Program.cs`:**
    *   **Thử thách:** Việc nhớ tất cả các `using` statements, các tùy chọn `TokenValidationParameters` và thứ tự middleware có thể tốn thời gian và dễ mắc lỗi.
    *   **Antigravity Solution (Vibe Coding):** "Antigravity, tích hợp JWT Bearer Authentication vào `Program.cs` của tôi. Đảm bảo validate tất cả các tham số cơ bản (issuer, audience, lifetime, signing key) và sử dụng cấu hình từ `appsettings.json`."
    *   **Cơ chế Antigravity:** Antigravity sẽ:
        *   Phân tích `Program.cs` hiện có.
        *   Tự động thêm các `using` statements cần thiết (`Microsoft.AspNetCore.Authentication.JwtBearer`, `Microsoft.IdentityModel.Tokens`, `System.Text`).
        *   Chèn khối mã `AddAuthentication` và `AddJwtBearer` vào đúng vị trí trong phần cấu hình dịch vụ (`builder.Services`).
        *   Chèn `app.UseAuthentication()` và `app.UseAuthorization()` vào đúng vị trí trong pipeline middleware, đảm bảo tuân thủ thứ tự.
        *   Nó có thể thậm chí đề xuất các cải tiến hoặc cảnh báo nếu bạn đã có một cấu hình xác thực xung đột.

3.  **Áp dụng Thuộc tính `[Authorize]`:**
    *   **Thử thách:** Nhớ áp dụng `[Authorize]` cho tất cả các Controller hoặc Action cần bảo vệ.
    *   **Antigravity Solution:** "Antigravity, áp dụng thuộc tính `[Authorize]` cho tất cả các Controller trong thư mục `Controllers` trừ `AuthController` (sẽ tạo sau)."
    *   **Cơ chế Antigravity:** Antigravity có thể duyệt qua các tệp Controller, tự động thêm `using Microsoft.AspNetCore.Authorization;` và `[Authorize]` vào đầu mỗi lớp Controller được chỉ định.

4.  **Kiểm thử Tự động và Xác minh `401 Unauthorized`:**
    *   **Thử thách:** Kiểm tra thủ công bằng Swagger UI hoặc Postman cho mỗi endpoint là lặp đi lặp lại.
    *   **Antigravity Solution (Agentic Testing):** "Antigravity, chạy ứng dụng API và sau đó sử dụng một subagent trình duyệt (hoặc Postman agent) để gửi yêu cầu GET đến `/api/regions` mà không có token xác thực. Xác minh rằng tôi nhận được phản hồi `401 Unauthorized`."
    *   **Cơ chế Antigravity:**
        *   Antigravity sẽ khởi động ứng dụng API của bạn.
        *   Nó sẽ kích hoạt một subagent (ví dụ: một phiên Postman hoặc một công cụ HTTP client nội bộ).
        *   Subagent này sẽ gửi yêu cầu HTTP `GET` đến `https://localhost:7xxx/api/regions` (Antigravity biết URL của ứng dụng của bạn).
        *   Antigravity sẽ phân tích phản hồi HTTP, kiểm tra mã trạng thái.
        *   Nếu mã trạng thái là `401`, nó sẽ báo cáo thành công. Nếu không, nó sẽ báo cáo lỗi và cung cấp thông tin chi tiết để gỡ lỗi.
        *   Điều này giúp bạn nhanh chóng xác nhận rằng cơ chế bảo mật đã được triển khai đúng cách mà không cần tương tác thủ công.

Bằng cách tận dụng Antigravity IDE và tư duy Vibe Coding, bạn không chỉ tiết kiệm thời gian mà còn giảm thiểu lỗi, đảm bảo rằng các cấu hình bảo mật quan trọng được triển khai một cách chính xác và nhất quán. Antigravity hoạt động như một chuyên gia lập trình cấp senior luôn sẵn sàng hỗ trợ bạn trong mọi bước của quá trình phát triển.

## Tóm tắt Phần

*   **Xác thực (Authentication)** là quá trình kiểm tra danh tính người dùng ("Bạn là ai?"), trong khi **Ủy quyền (Authorization)** xác định quyền truy cập của họ ("Bạn có quyền làm gì?").
*   **JSON Web Tokens (JWT)** là một tiêu chuẩn an toàn, nhỏ gọn để truyền thông tin giữa các bên, bao gồm ba phần: Header, Payload (Claims) và Signature.
*   Chữ ký của JWT (được tạo bằng khóa bí mật) đảm bảo tính toàn vẹn và xác thực nguồn gốc của token.
*   Quy trình xác thực JWT bao gồm đăng nhập, tạo token, lưu trữ token, gửi token cùng các yêu cầu (qua tiêu đề `Authorization: Bearer <token>`) và xác minh token bởi API.
*   Để thiết lập JWT Authentication trong ASP.NET Core, cần cài đặt các gói NuGet: `Microsoft.AspNetCore.Authentication.JwtBearer`, `Microsoft.IdentityModel.Tokens`, `System.IdentityModel.Tokens.Jwt`.
*   Cấu hình JWT (Key, Issuer, Audience) được lưu trữ trong `appsettings.json` để dễ quản lý, trong đó `Key` là yếu tố bí mật quan trọng nhất cần được bảo vệ.
*   Trong `Program.cs`, dịch vụ xác thực được đăng ký bằng `builder.Services.AddAuthentication().AddJwtBearer()` và cấu hình `TokenValidationParameters` để xác minh các khía cạnh của token (Issuer, Audience, Lifetime, Signing Key).
*   Middleware xác thực (`app.UseAuthentication()`) và ủy quyền (`app.UseAuthorization()`) phải được thêm vào pipeline xử lý yêu cầu, với `UseAuthentication()` đứng trước `UseAuthorization()` để đảm bảo `ClaimsPrincipal` được thiết lập trước khi ủy quyền diễn ra.
*   Thuộc tính `[Authorize]` được sử dụng trên Controllers hoặc Action Methods để yêu cầu xác thực.
*   Khi truy cập tài nguyên được bảo vệ bằng `[Authorize]` mà không có JWT hợp lệ, API sẽ trả về mã trạng thái HTTP `401 Unauthorized`, chặn yêu cầu trước khi nó đến logic nghiệp vụ.
*   **Antigravity IDE và Vibe Coding** có thể được ứng dụng để tự động hóa việc tạo khóa bí mật, scaffolding cấu hình JWT trong `Program.cs`, áp dụng thuộc tính `[Authorize]` và tự động kiểm thử cơ chế xác thực, giúp tăng hiệu quả và độ chính xác trong quá trình phát triển.

<!-- REVIEWED_BY_AGENT -->
