# Bài 11: Xác thực & Ủy quyền (Authentication & Authorization) - Phần 2

## Giới thiệu chuyên sâu về Bảo mật API

Chào mừng bạn đến với Phần 11 của khóa học, nơi chúng ta sẽ tiếp tục hành trình xây dựng các tính năng bảo mật mạnh mẽ cho RESTful Web API của mình bằng ASP.NET Core. Trong chương trước, chúng ta đã đặt nền móng cho hệ thống nhận dạng (Identity System) của ứng dụng. Chương này sẽ tập trung vào việc triển khai các cơ chế xác thực và ủy quyền thiết yếu, đảm bảo rằng chỉ những người dùng hợp lệ mới có thể truy cập các tài nguyên API được bảo vệ, và họ chỉ có thể thực hiện những hành động được phép.

Mục tiêu chính của chương này là trang bị cho bạn kiến thức và kỹ năng để:
*   **Quản lý Danh tính Người dùng:** Tạo các endpoint API cho phép người dùng đăng ký tài khoản mới và đăng nhập an toàn.
*   **Xác thực Phi trạng thái với JWT:** Phát hành và sử dụng JSON Web Token (JWT) để xác thực người dùng sau khi đăng nhập thành công, đồng thời hiểu rõ cơ chế hoạt động "dưới mui xe" của JWT.
*   **Tách biệt Trách nhiệm:** Áp dụng Repository Pattern và Dependency Injection để tổ chức mã nguồn một cách hiệu quả, đặc biệt trong việc tạo và quản lý JWT.
*   **Kiểm soát Quyền truy cập:** Triển khai cơ chế ủy quyền dựa trên vai trò (Role-Based Authorization) để kiểm soát quyền truy cập vào các tài nguyên API một cách chi tiết.
*   **Kiểm thử Hiệu quả:** Cấu hình Swagger UI để hỗ trợ xác thực và ủy quyền, giúp quá trình kiểm thử các API được bảo vệ trở nên trực quan và thuận tiện.

Chúng ta sẽ đi sâu vào các khái niệm cốt lõi, từ việc định nghĩa Data Transfer Objects (DTOs) đến việc tận dụng mạnh mẽ các tính năng tích hợp sẵn của ASP.NET Core Identity và Dependency Injection, luôn hướng tới các phương pháp lập trình tốt nhất.

---

## 1. Xây dựng Controller Xác thực và Cơ chế Đăng ký Người dùng

Để quản lý các hoạt động liên quan đến xác thực như đăng ký và đăng nhập, việc tạo một Controller chuyên biệt là cần thiết. Điều này giúp tách biệt trách nhiệm và giữ cho mã nguồn gọn gàng.

### 1.1. Tạo Auth Controller

Hãy bắt đầu bằng cách tạo một API Controller mới có tên `AuthController.cs` trong thư mục `Controllers`.

```csharp
// Controllers/AuthController.cs
using Microsoft.AspNetCore.Identity; // Cần thiết cho UserManager và IdentityUser
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO; // Giả sử các DTOs sẽ nằm trong namespace này

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")] // Định tuyến cơ bản cho controller
    [ApiController] // Đánh dấu đây là một API Controller
    public class AuthController : ControllerBase
    {
        private readonly UserManager<IdentityUser> userManager; // Dependency Injection cho UserManager

        /// <summary>
        /// Constructor để inject UserManager.
        /// UserManager là một dịch vụ cốt lõi của ASP.NET Core Identity,
        /// chịu trách nhiệm quản lý người dùng (tạo, cập nhật, xóa, kiểm tra mật khẩu, gán vai trò...).
        /// Generic type <IdentityUser> chỉ định loại đối tượng người dùng mà chúng ta đang quản lý.
        /// </summary>
        public AuthController(UserManager<IdentityUser> userManager)
        {
            this.userManager = userManager;
        }

        // Các phương thức hành động (Action Methods) sẽ được thêm vào đây
    }
}
```

> [!NOTE]
> `UserManager<IdentityUser>` là một abstraction mạnh mẽ được cung cấp bởi `Microsoft.AspNetCore.Identity`. Nó giúp bạn tương tác với cơ sở dữ liệu người dùng mà không cần phải viết mã SQL thủ công hoặc xử lý các chi tiết phức tạp như băm mật khẩu. Bằng cách inject nó vào constructor, chúng ta tuân thủ nguyên tắc Dependency Injection, giúp kiểm thử dễ dàng hơn và giảm sự phụ thuộc cứng giữa các thành phần.

### 1.2. Định nghĩa Data Transfer Object (DTO) cho Đăng ký

Trước khi xử lý yêu cầu đăng ký, chúng ta cần một DTO để định hình dữ liệu đầu vào từ client. DTO này giúp tách biệt mô hình dữ liệu của ứng dụng khỏi mô hình truyền tải qua mạng, cho phép kiểm soát chặt chẽ dữ liệu được phép nhận.

```csharp
// Models/DTO/RegisterRequestDTO.cs
using System.ComponentModel.DataAnnotations; // Cần thiết cho các thuộc tính validation

namespace NZWalks.API.Models.DTO
{
    /// <summary>
    /// DTO dùng để nhận thông tin đăng ký người dùng mới từ client.
    /// </summary>
    public class RegisterRequestDTO
    {
        [Required(ErrorMessage = "Tên người dùng là bắt buộc.")] // Đảm bảo trường này không trống
        [DataType(DataType.EmailAddress, ErrorMessage = "Tên người dùng phải là định dạng email hợp lệ.")] // Yêu cầu định dạng email
        public string Username { get; set; }

        [Required(ErrorMessage = "Mật khẩu là bắt buộc.")]
        [DataType(DataType.Password, ErrorMessage = "Mật khẩu không hợp lệ.")] // Chỉ định đây là trường mật khẩu
        public string Password { get; set; }

        /// <summary>
        /// Danh sách các vai trò mà người dùng muốn được gán khi đăng ký.
        /// Ví dụ: ["Reader", "Writer"]
        /// </summary>
        public string[] Roles { get; set; }
    }
}
```

> [!TIP]
> Các thuộc tính `[Required]`, `[DataType]`, v.v., là một phần của **Model Validation** trong ASP.NET Core. Khi một yêu cầu HTTP đến, ASP.NET Core sẽ tự động kiểm tra xem `RegisterRequestDTO` có tuân thủ các quy tắc này hay không. Nếu không, `ModelState.IsValid` sẽ là `false` và bạn có thể trả về lỗi `BadRequest` với thông tin chi tiết về lỗi validation mà không cần viết mã kiểm tra thủ công. Điều này giúp giảm boilerplate code và tăng cường bảo mật.

### 1.3. Triển khai Phương thức Đăng ký (Register Action Method)

Phương thức `Register` sẽ xử lý yêu cầu `HTTP POST` để tạo người dùng mới.

```csharp
// Controllers/AuthController.cs (tiếp tục)
// ... (các phần trên)

        /// <summary>
        /// Đăng ký người dùng mới vào hệ thống.
        /// </summary>
        /// <param name="registerRequestDto">Thông tin đăng ký bao gồm tên người dùng (email), mật khẩu và các vai trò.</param>
        /// <returns>HTTP 200 OK nếu đăng ký thành công, HTTP 400 Bad Request nếu có lỗi.</returns>
        [HttpPost]
        [Route("Register")]
        public async Task<IActionResult> Register([FromBody] RegisterRequestDTO registerRequestDto)
        {
            // Kiểm tra Model Validation tự động của ASP.NET Core
            // Nếu DTO không hợp lệ (ví dụ: thiếu trường bắt buộc), ASP.NET Core sẽ tự động trả về 400 Bad Request
            // trước khi mã này được thực thi, trừ khi chúng ta tắt nó đi.
            // Tuy nhiên, việc kiểm tra thủ công giúp chúng ta có thể tùy chỉnh thông báo lỗi.
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            // 1. Tạo đối tượng IdentityUser từ DTO
            // UserName và Email thường được gán cùng một giá trị (email) trong nhiều ứng dụng.
            var identityUser = new IdentityUser
            {
                UserName = registerRequestDto.Username,
                Email = registerRequestDto.Username // Sử dụng email làm tên người dùng
            };

            // 2. Sử dụng UserManager để tạo người dùng và băm mật khẩu
            // CreateAsync sẽ tự động xử lý việc băm mật khẩu an toàn và lưu vào cơ sở dữ liệu.
            var identityResult = await userManager.CreateAsync(identityUser, registerRequestDto.Password);

            // 3. Xử lý kết quả tạo người dùng
            if (identityResult.Succeeded)
            {
                // 4. Gán vai trò cho người dùng nếu có vai trò được cung cấp
                if (registerRequestDto.Roles != null && registerRequestDto.Roles.Any())
                {
                    // AddToRolesAsync sẽ gán các vai trò đã tồn tại cho người dùng.
                    // Nếu vai trò không tồn tại, IdentityResult sẽ báo lỗi.
                    identityResult = await userManager.AddToRolesAsync(identityUser, registerRequestDto.Roles);

                    if (identityResult.Succeeded)
                    {
                        return Ok("User was registered successfully! Please Login.");
                    }
                    else
                    {
                        // Xử lý lỗi khi gán vai trò
                        return BadRequest(identityResult.Errors);
                    }
                }
                return Ok("User was registered successfully! Please Login.");
            }

            // 5. Trả về lỗi nếu có vấn đề trong quá trình tạo người dùng (ví dụ: tên người dùng đã tồn tại)
            return BadRequest(identityResult.Errors); // identityResult.Errors chứa danh sách lỗi chi tiết
        }
    }
}
```

> [!NOTE]
> *   `UserManager.CreateAsync(user, password)` không chỉ tạo người dùng mà còn tự động băm (hash) mật khẩu một cách an toàn trước khi lưu vào cơ sở dữ liệu. Điều này là cực kỳ quan trọng để bảo vệ thông tin người dùng.
> *   `IdentityResult` là một đối tượng chứa thông tin về kết quả của một thao tác Identity, bao gồm một thuộc tính `Succeeded` (bool) và một danh sách `Errors` (IEnumerable<IdentityError>) nếu thao tác thất bại. Việc kiểm tra `Succeeded` và trả về `identityResult.Errors` giúp client biết chính xác vấn đề là gì.

---

## 2. Triển khai Cơ chế Đăng nhập và Khái niệm Xác thực Phi trạng thái

Sau khi người dùng có thể đăng ký, bước tiếp theo là cho phép họ đăng nhập vào hệ thống. Phương thức đăng nhập sẽ xác thực thông tin đăng nhập và, quan trọng hơn, sẽ phát hành một JSON Web Token (JWT) để duy trì trạng thái xác thực của người dùng trong các yêu cầu tiếp theo.

### 2.1. Định nghĩa Data Transfer Object (DTO) cho Đăng nhập

Tương tự như đăng ký, chúng ta cần một DTO đơn giản để nhận thông tin đăng nhập từ client.

```csharp
// Models/DTO/LoginRequestDTO.cs
using System.ComponentModel.DataAnnotations;

namespace NZWalks.API.Models.DTO
{
    /// <summary>
    /// DTO dùng để nhận thông tin đăng nhập từ client.
    /// </summary>
    public class LoginRequestDTO
    {
        [Required(ErrorMessage = "Tên người dùng là bắt buộc.")]
        [DataType(DataType.EmailAddress, ErrorMessage = "Tên người dùng phải là định dạng email hợp lệ.")]
        public string Username { get; set; }

        [Required(ErrorMessage = "Mật khẩu là bắt buộc.")]
        [DataType(DataType.Password, ErrorMessage = "Mật khẩu không hợp lệ.")]
        public string Password { get; set; }
    }
}
```

### 2.2. Triển khai Phương thức Đăng nhập (Login Action Method)

Phương thức `Login` sẽ là một `HTTP POST`. Nó sẽ tìm kiếm người dùng bằng tên người dùng (email) và sau đó kiểm tra mật khẩu đã cung cấp.

```csharp
// Controllers/AuthController.cs (tiếp tục)
// ... (các phần trên)

        [HttpPost]
        [Route("Login")]
        public async Task<IActionResult> Login([FromBody] LoginRequestDTO loginRequestDto)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            // 1. Tìm người dùng bằng email
            // FindByEmailAsync hiệu quả hơn FindByNameAsync nếu bạn dùng email làm tên người dùng.
            var user = await userManager.FindByEmailAsync(loginRequestDto.Username);

            // 2. Kiểm tra nếu người dùng tồn tại và mật khẩu chính xác
            if (user != null)
            {
                // CheckPasswordAsync sẽ kiểm tra mật khẩu đã cung cấp với mật khẩu băm trong DB.
                var checkPasswordResult = await userManager.CheckPasswordAsync(user, loginRequestDto.Password);

                if (checkPasswordResult)
                {
                    // 3. Nếu xác thực thành công, TẠO TOKEN (sẽ triển khai ở phần sau)
                    // Hiện tại, chỉ trả về thành công với một placeholder.
                    return Ok("Login successful! Token will be generated here."); // Placeholder
                }
            }

            // 4. Trả về lỗi nếu tên người dùng hoặc mật khẩu không chính xác.
            // Thông báo lỗi chung chung là một biện pháp bảo mật để tránh lộ thông tin người dùng.
            return BadRequest("Username or password incorrect!");
        }
    }
}
```

> [!TIP]
> Việc trả về một thông báo lỗi chung chung như "Username or password incorrect!" thay vì chỉ rõ "Username not found" hoặc "Incorrect password" là một biện pháp bảo mật quan trọng. Nó ngăn chặn kẻ tấn công biết được liệu một tên người dùng cụ thể có tồn tại trong hệ thống hay không, từ đó làm giảm khả năng thực hiện các cuộc tấn công vét cạn (brute-force) hoặc liệt kê tài khoản.

---

## 3. Phát triển Cơ chế Phát hành JSON Web Token (JWT)

Sau khi người dùng được xác thực thành công, chúng ta cần cung cấp cho họ một cơ chế để chứng minh danh tính của mình trong các yêu cầu API tiếp theo mà không cần gửi lại tên người dùng và mật khẩu mỗi lần. JSON Web Token (JWT) là giải pháp tiêu chuẩn cho xác thực phi trạng thái (stateless authentication).

### 3.1. JSON Web Token (JWT) là gì và Tại sao lại sử dụng?

> [!NOTE]
> **JSON Web Token (JWT)** (phát âm là "jot") là một tiêu chuẩn mở (RFC 7519) định nghĩa một cách nhỏ gọn và an toàn về URL để truyền tải thông tin an toàn giữa các bên dưới dạng một đối tượng JSON. Thông tin này có thể được xác minh và tin cậy vì nó được ký số.
>
> **Cấu trúc của JWT:** Một JWT luôn gồm ba phần, được phân tách bằng dấu chấm (`.`):
> 1.  **Header:** Là một đối tượng JSON mã hóa Base64Url, chứa loại token (thường là "JWT") và thuật toán mã hóa được sử dụng để ký (ví dụ: HS256, RS256).
>    ```json
>    {
>      "alg": "HS256",
>      "typ": "JWT"
>    }
>    ```
> 2.  **Payload (Claims):** Là một đối tượng JSON mã hóa Base64Url, chứa các "claims" (khẳng định) về người dùng và các thuộc tính bổ sung. Có ba loại claims:
>    *   **Registered claims:** Các claims chuẩn được định nghĩa bởi JWT specification (ví dụ: `iss` (issuer), `exp` (expiration time), `sub` (subject), `aud` (audience)).
>    *   **Public claims:** Các claims tùy chỉnh nhưng được đăng ký công khai để tránh xung đột.
>    *   **Private claims:** Các claims tùy chỉnh được sử dụng giữa các bên đồng ý và không cần đăng ký.
>    Ví dụ:
>    ```json
>    {
>      "sub": "1234567890",
>      "name": "John Doe",
>      "admin": true,
>      "email": "john.doe@example.com",
>      "roles": ["Reader", "Writer"],
>      "exp": 1678886400 // Thời gian hết hạn (Unix timestamp)
>    }
>    ```
> 3.  **Signature:** Được tạo bằng cách lấy Header đã mã hóa, Payload đã mã hóa, một khóa bí mật (secret key) và thuật toán được chỉ định trong Header.
>    `HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret_key)`
>    Chữ ký này đảm bảo tính toàn vẹn của token – bất kỳ thay đổi nào đối với Header hoặc Payload sẽ làm cho chữ ký không hợp lệ, và server sẽ từ chối token.
>
> **Tại sao sử dụng JWT?**
> *   **Xác thực Phi trạng thái (Stateless Authentication):** Server không cần lưu trữ trạng thái phiên của người dùng. Token chứa tất cả thông tin cần thiết về người dùng và các quyền hạn của họ. Điều này giúp giảm tải cho server và loại bỏ nhu cầu duy trì phiên (session).
> *   **Khả năng mở rộng (Scalability):** Dễ dàng mở rộng các dịch vụ API vì không có trạng thái phiên cụ thể trên server. Bất kỳ server nào cũng có thể xác minh token độc lập.
> *   **Bảo mật:** Chữ ký số đảm bảo token không bị giả mạo. Server có thể tin tưởng vào thông tin trong token miễn là chữ ký hợp lệ.
> *   **Tương thích đa nền tảng (Cross-Domain/Microservices):** JWT có thể được sử dụng dễ dàng trong các kiến trúc microservices hoặc khi API và frontend ở các domain khác nhau, vì nó là một tiêu chuẩn độc lập.

### 3.2. Thiết lập Repository để Phát hành Token

Để duy trì nguyên tắc tách biệt trách nhiệm (Separation of Concerns) và làm cho mã nguồn dễ quản lý, chúng ta sẽ tạo một Repository riêng biệt để xử lý việc tạo JWT.

#### 3.2.1. Tạo ITokenRepository Interface

```csharp
// Repositories/ITokenRepository.cs
using Microsoft.AspNetCore.Identity; // Cần thiết cho IdentityUser

namespace NZWalks.API.Repositories
{
    /// <summary>
    /// Interface định nghĩa các phương thức để tạo JWT Token.
    /// </summary>
    public interface ITokenRepository
    {
        /// <summary>
        /// Tạo một JWT Token cho người dùng đã xác thực.
        /// </summary>
        /// <param name="user">Đối tượng IdentityUser của người dùng.</param>
        /// <param name="roles">Danh sách các vai trò của người dùng.</param>
        /// <returns>Chuỗi JWT Token đã được tạo.</returns>
        string CreateJwtToken(IdentityUser user, List<string> roles);
    }
}
```

#### 3.2.2. Triển khai TokenRepository

Lớp `TokenRepository` sẽ chứa tất cả logic cần thiết để xây dựng một JWT hoàn chỉnh. Nó sẽ cần truy cập vào cấu hình ứng dụng (`appsettings.json`) để lấy khóa bí mật và các thông tin JWT khác.

```csharp
// Repositories/TokenRepository.cs
using Microsoft.AspNetCore.Identity;
using Microsoft.IdentityModel.Tokens; // Đối với SymmetricSecurityKey, SigningCredentials
using System.IdentityModel.Tokens.Jwt; // Đối với JwtSecurityToken, JwtSecurityTokenHandler
using System.Security.Claims; // Đối với Claim, ClaimTypes
using System.Text; // Đối với Encoding
using Microsoft.Extensions.Configuration; // Đối với IConfiguration

namespace NZWalks.API.Repositories
{
    public class TokenRepository : ITokenRepository
    {
        private readonly IConfiguration configuration;

        /// <summary>
        /// Constructor để inject IConfiguration, cho phép truy cập cài đặt ứng dụng.
        /// </summary>
        public TokenRepository(IConfiguration configuration)
        {
            this.configuration = configuration;
        }

        public string CreateJwtToken(IdentityUser user, List<string> roles)
        {
            // 1. Tạo danh sách Claims (Payload)
            // Claims là các thông tin về người dùng hoặc các quyền hạn.
            var claims = new List<Claim>
            {
                // Thêm email của người dùng làm ClaimTypes.Email
                new Claim(ClaimTypes.Email, user.Email)
            };

            // Thêm từng vai trò của người dùng vào danh sách Claims dưới dạng ClaimTypes.Role
            foreach (var role in roles)
            {
                claims.Add(new Claim(ClaimTypes.Role, role));
            }

            // 2. Lấy khóa bí mật từ cấu hình ứng dụng (appsettings.json)
            // Khóa này được sử dụng để ký token, đảm bảo tính toàn vẹn.
            var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(configuration["Jwt:Key"]));

            // 3. Tạo thông tin xác thực ký (SigningCredentials)
            // Đây là cách token sẽ được ký, sử dụng khóa bí mật và thuật toán băm HMACSHA256.
            var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

            // 4. Tạo đối tượng JWT Token
            var token = new JwtSecurityToken(
                issuer: configuration["Jwt:Issuer"],       // Người phát hành token
                audience: configuration["Jwt:Audience"],   // Đối tượng nhận token
                claims: claims,                            // Danh sách claims chứa thông tin người dùng và vai trò
                expires: DateTime.Now.AddMinutes(15),      // Thời gian hết hạn của token (ví dụ: 15 phút từ bây giờ)
                signingCredentials: credentials);          // Thông tin ký token

            // 5. Serialize token thành chuỗi JWT cuối cùng
            return new JwtSecurityTokenHandler().WriteToken(token);
        }
    }
}
```

#### 3.2.3. Cấu hình JWT trong appsettings.json

Bạn cần thêm một phần `Jwt` vào `appsettings.json` để lưu trữ các thông tin cấu hình JWT như khóa bí mật, nhà phát hành (Issuer) và đối tượng (Audience). Các giá trị này sẽ được `TokenRepository` sử dụng.

```json
// appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "NZWalksConnectionString": "Server=localhost;Database=NZWalksDb;Trusted_Connection=True;TrustServerCertificate=True",
    "NZWalksAuthConnectionString": "Server=localhost;Database=NZWalksAuthDb;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Jwt": {
    "Key": "ThisIsAVeryStrongAndSecureKeyForJWTAuthenticationThatIsAtLeast32BytesLong", // KHÓA NÀY PHẢI DÀI VÀ PHỨC TẠP
    "Issuer": "https://localhost:7001", // URL của API của bạn (ví dụ: https://yourdomain.com)
    "Audience": "https://localhost:7001" // URL của client hoặc API của bạn (ví dụ: https://yourclient.com hoặc https://yourdomain.com)
  }
}
```

> [!WARNING]
> **Bảo mật của `Jwt:Key` là tối quan trọng!**
> *   Khóa này phải đủ dài (ít nhất 16-32 byte cho HS256) và phức tạp, không dễ đoán.
> *   Trong môi trường sản phẩm, **TUYỆT ĐỐI KHÔNG** lưu khóa bí mật trực tiếp trong `appsettings.json`. Thay vào đó, hãy sử dụng các phương pháp quản lý bí mật an toàn hơn như:
    *   **Biến môi trường (Environment Variables):** Lý tưởng cho các môi trường triển khai.
    *   **Azure Key Vault / AWS Secrets Manager / HashiCorp Vault:** Các dịch vụ quản lý bí mật chuyên dụng.
    *   **User Secrets:** Dành cho môi trường phát triển cục bộ để tránh đẩy bí mật lên source control.
> *   **`Issuer` và `Audience`**: `Issuer` là bên phát hành token (API của bạn). `Audience` là bên mà token được cấp cho (có thể là client hoặc chính API của bạn). Việc cấu hình đúng giúp tăng cường bảo mật bằng cách đảm bảo token chỉ được chấp nhận bởi các bên dự kiến.

---

## 4. Tích hợp TokenRepository và Hoàn thiện Chức năng Phát hành Token

Bây giờ chúng ta đã có logic để tạo JWT, chúng ta cần tích hợp nó vào hệ thống Dependency Injection (DI) của ASP.NET Core và sử dụng nó trong phương thức `Login` của `AuthController`.

### 4.1. Đăng ký TokenRepository vào DI Container

Trong tệp `Program.cs`, hãy đăng ký `ITokenRepository` và `TokenRepository` vào DI Container của ASP.NET Core. Điều này cho phép ASP.NET Core tự động inject `TokenRepository` khi có yêu cầu `ITokenRepository`.

```csharp
// Program.cs
// ... (các using khác)
using NZWalks.API.Repositories; // Đảm bảo namespace này được thêm vào

var builder = WebApplication.CreateBuilder(args);

// ... (các dịch vụ khác đã có)

// Đăng ký Repository cho Token vào DI Container
// AddScoped: Một instance của TokenRepository sẽ được tạo cho mỗi yêu cầu HTTP
// và được chia sẻ trong suốt vòng đời của yêu cầu đó.
builder.Services.AddScoped<ITokenRepository, TokenRepository>();

var app = builder.Build();

// ... (các middleware khác)
```

> [!NOTE]
> Các phương thức đăng ký dịch vụ trong DI (`AddScoped`, `AddTransient`, `AddSingleton`) xác định vòng đời (lifetime) của instance dịch vụ:
> *   `AddScoped`: Một instance được tạo mỗi khi một phạm vi (scope) mới được tạo. Trong ứng dụng web, một scope mới thường được tạo cho mỗi yêu cầu HTTP.
> *   `AddTransient`: Một instance mới được tạo mỗi khi nó được yêu cầu.
> *   `AddSingleton`: Chỉ một instance duy nhất được tạo và được tái sử dụng trong suốt vòng đời của ứng dụng.
> Chọn lifetime phù hợp là quan trọng để quản lý tài nguyên và trạng thái. `AddScoped` thường là lựa chọn tốt cho các repository trong ứng dụng web.

### 4.2. Inject TokenRepository vào AuthController

Bây giờ, hãy inject `ITokenRepository` vào `AuthController` thông qua constructor.

```csharp
// Controllers/AuthController.cs (cập nhật)
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories; // Thêm namespace này

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class AuthController : ControllerBase
    {
        private readonly UserManager<IdentityUser> userManager;
        private readonly ITokenRepository tokenRepository; // Khai báo ITokenRepository

        public AuthController(UserManager<IdentityUser> userManager, ITokenRepository tokenRepository) // Inject ITokenRepository
        {
            this.userManager = userManager;
            this.tokenRepository = tokenRepository;
        }

        // ... (phương thức Register)
```

### 4.3. Cập nhật Phương thức Login để Tạo và Trả về Token

Cuối cùng, chúng ta sẽ cập nhật phương thức `Login` để gọi `CreateJwtToken` từ `TokenRepository` và trả về token trong một DTO phản hồi.

#### 4.3.1. Tạo LoginResponseDTO

```csharp
// Models/DTO/LoginResponseDTO.cs
namespace NZWalks.API.Models.DTO
{
    /// <summary>
    /// DTO dùng để trả về JWT Token sau khi đăng nhập thành công.
    /// </summary>
    public class LoginResponseDTO
    {
        public string JwtToken { get; set; }
    }
}
```

#### 4.3.2. Cập nhật phương thức Login

```csharp
// Controllers/AuthController.cs (cập nhật phương thức Login)
// ... (các phần trên)

        [HttpPost]
        [Route("Login")]
        public async Task<IActionResult> Login([FromBody] LoginRequestDTO loginRequestDto)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            var user = await userManager.FindByEmailAsync(loginRequestDto.Username);

            if (user != null)
            {
                var checkPasswordResult = await userManager.CheckPasswordAsync(user, loginRequestDto.Password);

                if (checkPasswordResult)
                {
                    // Lấy tất cả các vai trò mà người dùng này được gán
                    var roles = await userManager.GetRolesAsync(user);

                    // Tạo JWT Token bằng cách sử dụng TokenRepository
                    var jwtToken = tokenRepository.CreateJwtToken(user, roles.ToList());

                    // Trả về token trong LoginResponseDTO
                    var response = new LoginResponseDTO
                    {
                        JwtToken = jwtToken
                    };

                    return Ok(response);
                }
            }

            return BadRequest("Username or password incorrect!");
        }
    }
}
```

Bây giờ, khi người dùng đăng nhập thành công, API sẽ trả về một JWT hợp lệ. JWT này là "chìa khóa" mà client sẽ sử dụng để truy cập các tài nguyên được bảo vệ trong các yêu cầu tiếp theo.

---

## 5. Ủy quyền Dựa trên Vai trò (Role-Based Authorization)

Sau khi người dùng được xác thực và có một JWT hợp lệ, bước tiếp theo là kiểm soát quyền truy cập của họ vào các tài nguyên cụ thể dựa trên vai trò của họ. Đây là khái niệm ủy quyền (Authorization).

### 5.1. Phân biệt Xác thực (Authentication) và Ủy quyền (Authorization)

> [!NOTE]
> Việc hiểu rõ sự khác biệt giữa hai khái niệm này là rất quan trọng trong bảo mật ứng dụng:
> *   **Xác thực (Authentication): "Bạn là ai?"**
>    *   Là quá trình xác minh danh tính của người dùng. Nó trả lời câu hỏi "Bạn có phải là người bạn nói bạn là ai không?".
>    *   Ví dụ: Đăng nhập bằng tên người dùng và mật khẩu, sử dụng mã OTP, vân tay, hoặc nhận dạng khuôn mặt để chứng minh danh tính. Sau khi xác thực thành công, người dùng được cung cấp một bằng chứng (như JWT) để sử dụng trong các yêu cầu tiếp theo.
> *   **Ủy quyền (Authorization): "Bạn được phép làm gì?"**
>    *   Là quá trình xác định quyền truy cập của người dùng đã xác thực vào một tài nguyên hoặc hành động cụ thể. Nó trả lời câu hỏi "Người dùng này có quyền thực hiện hành động này không?".
>    *   Ví dụ: Người dùng có vai trò "Reader" chỉ được xem dữ liệu, trong khi người dùng có vai trò "Writer" được phép thêm, sửa, xóa dữ liệu. Ủy quyền luôn diễn ra sau khi xác thực thành công.

### 5.2. Triển khai Ủy quyền Dựa trên Vai trò với thuộc tính `[Authorize]`

ASP.NET Core cung cấp thuộc tính `[Authorize]` để dễ dàng triển khai ủy quyền. Chúng ta có thể áp dụng thuộc tính này ở cấp controller hoặc cấp action method để chỉ định các vai trò cần thiết để truy cập.

Giả sử chúng ta có một `AreaController` với các phương thức CRUD (Create, Read, Update, Delete) cho một tài nguyên "Area".

```csharp
// Controllers/AreaController.cs (ví dụ)
using Microsoft.AspNetCore.Authorization; // Cần thiết cho thuộc tính [Authorize]
using Microsoft.AspNetCore.Mvc;
// ... (các using khác cho DTOs và Repository của Area)

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    // [Authorize] // Áp dụng ở cấp controller để bảo vệ TẤT CẢ các action method bên trong.
                  // Nếu không có tham số Roles, chỉ cần người dùng được xác thực là đủ.
    public class AreaController : ControllerBase
    {
        // ... (constructor và các dependency khác, ví dụ: IAreaRepository)

        /// <summary>
        /// Lấy tất cả các vùng. Yêu cầu vai trò "Reader" hoặc "Writer".
        /// </summary>
        [HttpGet]
        [Authorize(Roles = "Reader,Writer")] // Người dùng phải có vai trò "Reader" HOẶC "Writer"
        public async Task<IActionResult> GetAllAreas()
        {
            // Logic lấy tất cả vùng từ repository
            // Ví dụ: var areas = await areaRepository.GetAllAsync();
            return Ok(/* danh sách vùng */);
        }

        /// <summary>
        /// Lấy một vùng theo ID. Yêu cầu vai trò "Reader" hoặc "Writer".
        /// </summary>
        [HttpGet]
        [Route("{id:Guid}")]
        [Authorize(Roles = "Reader,Writer")]
        public async Task<IActionResult> GetAreaById([FromRoute] Guid id)
        {
            // Logic lấy vùng theo ID
            // Ví dụ: var area = await areaRepository.GetByIdAsync(id);
            return Ok(/* vùng */);
        }

        /// <summary>
        /// Tạo vùng mới. Chỉ người dùng có vai trò "Writer" mới có thể thực hiện.
        /// </summary>
        [HttpPost]
        [Authorize(Roles = "Writer")] // Người dùng phải có vai trò "Writer"
        public async Task<IActionResult> CreateArea([FromBody] AddAreaRequestDto addAreaRequestDto)
        {
            // Logic tạo vùng
            return CreatedAtAction(nameof(GetAreaById), new { id = /* ID của vùng mới */ }, /* vùng mới */);
        }

        /// <summary>
        /// Cập nhật thông tin vùng. Chỉ người dùng có vai trò "Writer" mới có thể thực hiện.
        /// </summary>
        [HttpPut]
        [Route("{id:Guid}")]
        [Authorize(Roles = "Writer")]
        public async Task<IActionResult> UpdateArea([FromRoute] Guid id, [FromBody] UpdateAreaRequestDto updateAreaRequestDto)
        {
            // Logic cập nhật vùng
            return Ok(/* vùng đã cập nhật */);
        }

        /// <summary>
        /// Xóa một vùng. Chỉ người dùng có vai trò "Writer" mới có thể thực hiện.
        /// </summary>
        [HttpDelete]
        [Route("{id:Guid}")]
        [Authorize(Roles = "Writer")]
        public async Task<IActionResult> DeleteArea([FromRoute] Guid id)
        {
            // Logic xóa vùng
            return Ok(/* vùng đã xóa */);
        }
    }
}
```

> [!TIP]
> *   **`[Authorize]` không có tham số:** Yêu cầu người dùng phải được xác thực (có JWT hợp lệ), nhưng không yêu cầu bất kỳ vai trò cụ thể nào.
> *   **`[Authorize(Roles = "Role1")]`:** Yêu cầu người dùng phải có vai trò "Role1".
> *   **`[Authorize(Roles = "Role1,Role2")]`:** Yêu cầu người dùng phải có vai trò "Role1" **HOẶC** "Role2".
> *   **Yêu cầu nhiều vai trò cùng lúc (AND logic):** Nếu bạn muốn yêu cầu người dùng phải có **cả** "Role1" **VÀ** "Role2", bạn có thể áp dụng nhiều thuộc tính `[Authorize]` hoặc sử dụng Policy-Based Authorization (một cơ chế ủy quyền mạnh mẽ và linh hoạt hơn, thường được sử dụng cho các kịch bản phức tạp hơn, sẽ được đề cập trong các chương nâng cao).
> *   **Thứ tự ưu tiên:** Thuộc tính `[AllowAnonymous]` (cho phép truy cập mà không cần xác thực) sẽ ưu tiên hơn `[Authorize]`. Nếu bạn áp dụng `[Authorize]` ở cấp controller, bạn có thể sử dụng `[AllowAnonymous]` trên một action method cụ thể để bỏ qua yêu cầu ủy quyền cho action đó.

### 5.3. Kiểm thử Ủy quyền với Postman

Để kiểm thử cơ chế ủy quyền dựa trên vai trò, bạn sẽ cần thực hiện các bước sau:

1.  **Đăng ký người dùng:**
    *   Sử dụng endpoint `/api/Auth/Register` để đăng ký một người dùng với vai trò "Reader" (ví dụ: `reader@example.com`, mật khẩu `Reader@123`).
    *   Đăng ký một người dùng khác với vai trò "Writer" (ví dụ: `writer@example.com`, mật khẩu `Writer@123`).
    *   Đăng ký một người dùng không có vai trò nào (ví dụ: `user@example.com`, mật khẩu `User@123`).
2.  **Đăng nhập và lấy JWT:**
    *   Sử dụng endpoint `/api/Auth/Login` với thông tin đăng nhập của từng người dùng để lấy JWT tương ứng. Lưu các JWT này lại.
3.  **Gửi yêu cầu đến các endpoint được bảo vệ trong Postman:**
    *   Chọn một endpoint được bảo vệ, ví dụ: `GET /api/Area`.
    *   Trong tab `Authorization` của Postman, chọn loại `Bearer Token`.
    *   Dán JWT bạn nhận được sau khi đăng nhập vào trường `Token`.

    **Các trường hợp kiểm thử:**
    *   **Không có token hoặc token không hợp lệ:** Gửi yêu cầu đến `GET /api/Area` mà không có header `Authorization` hoặc với một token bị sửa đổi.
        *   **Phản hồi dự kiến:** `401 Unauthorized`.
    *   **Token hợp lệ nhưng người dùng không có vai trò cần thiết:** Gửi yêu cầu đến `POST /api/Area` (yêu cầu vai trò "Writer") bằng JWT của người dùng "Reader".
        *   **Phản hồi dự kiến:** `403 Forbidden`.
    *   **Token hợp lệ và người dùng có vai trò cần thiết:**
        *   Gửi yêu cầu đến `GET /api/Area` bằng JWT của người dùng "Reader" hoặc "Writer".
            *   **Phản hồi dự kiến:** `200 OK` với dữ liệu.
        *   Gửi yêu cầu đến `POST /api/Area` bằng JWT của người dùng "Writer".
            *   **Phản hồi dự kiến:** `201 Created` hoặc `200 OK` (tùy thuộc vào logic tạo).

> [!NOTE]
> *   **`401 Unauthorized`:** Có nghĩa là bạn chưa được xác thực (chưa chứng minh bạn là ai) hoặc token của bạn không hợp lệ (hết hạn, bị giả mạo).
> *   **`403 Forbidden`:** Có nghĩa là bạn đã được xác thực (hệ thống biết bạn là ai và token của bạn hợp lệ), nhưng bạn không có quyền truy cập vào tài nguyên hoặc thực hiện hành động đó.

---

## 6. Tích hợp Ủy quyền vào Swagger UI để Kiểm thử Thuận tiện

Mặc dù Postman là một công cụ kiểm thử API tuyệt vời, việc có thể kiểm thử các API được bảo vệ trực tiếp từ Swagger UI sẽ tiện lợi hơn nhiều trong quá trình phát triển. Chúng ta cần cấu hình Swagger để cho phép người dùng nhập JWT và tự động gửi nó trong các yêu cầu đến các endpoint được bảo vệ.

### 6.1. Cấu hình SwaggerGen để Hỗ trợ JWT Bearer Authentication

Bạn cần mở rộng cấu hình `AddSwaggerGen` trong `Program.cs` để thêm định nghĩa bảo mật cho JWT Bearer. Điều này sẽ thêm một tùy chọn "Authorize" vào Swagger UI.

```csharp
// Program.cs
// ... (các using khác)
using Microsoft.OpenApi.Models; // Thêm namespace này cho OpenApiInfo, OpenApiSecurityScheme, v.v.
using Microsoft.AspNetCore.Authentication.JwtBearer; // Cần thiết cho JwtBearerDefaults.AuthenticationScheme

var builder = WebApplication.CreateBuilder(args);

// ... (các dịch vụ khác)

// Cấu hình Swagger/OpenAPI
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "NZ Walks API", Version = "v1" });

    // 1. Thêm định nghĩa bảo mật cho JWT Bearer
    // Điều này cho Swagger biết cách người dùng có thể xác thực bằng JWT.
    options.AddSecurityDefinition(JwtBearerDefaults.AuthenticationScheme, new OpenApiSecurityScheme
    {
        Name = "Authorization", // Tên của header HTTP mà token sẽ được gửi đi
        Description = "Vui lòng nhập chuỗi Bearer Authorization theo định dạng sau: `Bearer {Generated-JWT-Token}`",
        In = ParameterLocation.Header, // Token sẽ được gửi trong Header của yêu cầu
        Type = SecuritySchemeType.ApiKey, // Loại bảo mật là API Key
        Scheme = JwtBearerDefaults.AuthenticationScheme // Tên scheme là "Bearer"
    });

    // 2. Thêm yêu cầu bảo mật vào tất cả các endpoint của Swagger
    // Điều này cho Swagger biết rằng các endpoint của bạn yêu cầu cơ chế bảo mật đã định nghĩa ở trên.
    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = JwtBearerDefaults.AuthenticationScheme
                },
                Scheme = "oauth2", // Scheme này thường được dùng với OAuth2/OpenID Connect, nhưng cũng phù hợp với JWT Bearer
                Name = JwtBearerDefaults.AuthenticationScheme,
                In = ParameterLocation.Header
            },
            new List<string>() // Danh sách các scope yêu cầu (để trống nếu không sử dụng OAuth2 scopes)
        }
    });
});

var app = builder.Build();

// ... (các middleware khác)
```

> [!NOTE]
> *   `options.AddSecurityDefinition`: Định nghĩa một cơ chế bảo mật mới trong tài liệu OpenAPI. Ở đây, chúng ta định nghĩa cách Swagger sẽ hiển thị và chấp nhận token `Bearer`. `JwtBearerDefaults.AuthenticationScheme` là một hằng số có giá trị "Bearer".
> *   `options.AddSecurityRequirement`: Áp dụng cơ chế bảo mật đã định nghĩa cho các endpoint trong tài liệu Swagger. Điều này sẽ khiến Swagger UI hiển thị biểu tượng ổ khóa bên cạnh các endpoint được bảo vệ và thêm trường nhập token vào giao diện.

### 6.2. Cấu hình Middleware Xác thực JWT Bearer

Để API của bạn thực sự xử lý JWT từ các yêu cầu đến, bạn cần thêm middleware xác thực JWT Bearer vào pipeline yêu cầu trong `Program.cs`.

```csharp
// Program.cs (tiếp tục)
// ... (các using khác)
using Microsoft.AspNetCore.Authentication.JwtBearer; // Cần thiết cho JwtBearerDefaults
using Microsoft.IdentityModel.Tokens; // Cần thiết cho SymmetricSecurityKey
using System.Text; // Cần thiết cho Encoding

var builder = WebApplication.CreateBuilder(args);

// ... (các dịch vụ khác)

// Thêm dịch vụ xác thực JWT Bearer
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,          // Xác thực Issuer của token
            ValidateAudience = true,        // Xác thực Audience của token
            ValidateLifetime = true,        // Xác thực thời gian hết hạn của token
            ValidateIssuerSigningKey = true, // Xác thực khóa ký của token
            ValidIssuer = builder.Configuration["Jwt:Issuer"],       // Issuer hợp lệ
            ValidAudience = builder.Configuration["Jwt:Audience"],   // Audience hợp lệ
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"])) // Khóa ký
        };
    });

// Thêm dịch vụ Ủy quyền
builder.Services.AddAuthorization();

var app = builder.Build();

// ... (các middleware khác)

// Đảm bảo các middleware xác thực và ủy quyền được đặt ĐÚNG THỨ TỰ
// Authenticate phải đến trước Authorize
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers(); // Đảm bảo MapControllers nằm sau UseAuthorization

app.Run();
```

> [!WARNING]
> **Thứ tự của Middleware là CỰC KỲ QUAN TRỌNG:**
> *   `app.UseAuthentication()` phải được gọi **trước** `app.UseAuthorization()`.
> *   Nếu không, các thông tin xác thực (như danh tính người dùng và vai trò) sẽ không được thiết lập trong ngữ cảnh HTTP trước khi ủy quyền được kiểm tra, dẫn đến việc `[Authorize]` không hoạt động đúng.
> *   `app.MapControllers()` (hoặc `app.UseEndpoints()`) phải được gọi **sau** `app.UseAuthorization()`.

### 6.3. Sử dụng Ủy quyền trong Swagger UI

Sau khi cấu hình, khi bạn chạy ứng dụng và truy cập Swagger UI (thường là `https://localhost:port/swagger`), bạn sẽ thấy:
1.  Một nút "Authorize" (hoặc biểu tượng ổ khóa) ở góc trên bên phải của giao diện Swagger.
2.  Biểu tượng ổ khóa nhỏ bên cạnh mỗi endpoint được bảo vệ bởi `[Authorize]`.

**Các bước để kiểm thử với Swagger UI:**
1.  **Đăng nhập và lấy JWT:**
    *   Sử dụng endpoint `/api/Auth/Login` trong Swagger UI để đăng nhập với một người dùng (ví dụ: `writer@example.com`, mật khẩu `Writer@123`).
    *   Sao chép chuỗi JWT từ phản hồi.
2.  **Nhập JWT vào Swagger UI:**
    *   Nhấp vào nút "Authorize" (hoặc biểu tượng ổ khóa lớn).
    *   Trong cửa sổ bật lên, bạn sẽ thấy một trường để nhập token.
    *   Nhập JWT của bạn vào trường đó, theo định dạng **`Bearer <JWT_Token>`** (ví dụ: `Bearer eyJhbGciOiJIUzI1Ni...`).
    *   Nhấp vào "Authorize" hoặc "Login".
3.  **Kiểm thử các API được bảo vệ:**
    *   Bây giờ, khi bạn thực hiện các yêu cầu đến các endpoint được bảo vệ (những endpoint có biểu tượng ổ khóa), Swagger UI sẽ tự động thêm header `Authorization` với JWT bạn đã cung cấp.
    *   Bạn có thể "Logout" (hoặc "Clear") để xóa token và kiểm tra các trường hợp không có quyền truy cập hoặc không được xác thực.

---

## Tóm tắt Phần 11: Xác thực & Ủy quyền (Authentication & Authorization) - Phần 2

Trong chương này, chúng ta đã đi qua các bước quan trọng để triển khai một hệ thống xác thực và ủy quyền đầy đủ chức năng và an toàn cho RESTful Web API của mình:

*   **Thiết lập Auth Controller:** Tạo `AuthController` để tập trung các logic liên quan đến xác thực và ủy quyền.
*   **Phương thức Đăng ký (Register):**
    *   Định nghĩa `RegisterRequestDTO` với các thuộc tính validation để nhận thông tin người dùng mới (username/email, password, roles).
    *   Sử dụng `UserManager<IdentityUser>` để tạo người dùng một cách an toàn (bao gồm băm mật khẩu) trong cơ sở dữ liệu nhận dạng và gán vai trò.
*   **Phương thức Đăng nhập (Login):**
    *   Định nghĩa `LoginRequestDTO` cho thông tin đăng nhập.
    *   Sử dụng `UserManager<IdentityUser>` để tìm người dùng và kiểm tra mật khẩu.
*   **Phát triển JWT Token:**
    *   Hiểu sâu sắc về cấu trúc (Header, Payload, Signature) và lợi ích của JSON Web Token (JWT) trong xác thực phi trạng thái, khả năng mở rộng và bảo mật.
    *   Triển khai `ITokenRepository` và `TokenRepository` để tạo JWT, bao gồm việc xây dựng `Claims` (thông tin người dùng và vai trò), sử dụng `SymmetricSecurityKey` từ `appsettings.json` để ký token, và tạo `JwtSecurityToken` với các thông tin quan trọng như `Issuer`, `Audience`, và thời gian hết hạn (`expires`).
    *   Cấu hình các tham số JWT (`Key`, `Issuer`, `Audience`) trong `appsettings.json` và nhấn mạnh các biện pháp bảo mật cho khóa bí mật.
*   **Tích hợp Token Repository:**
    *   Đăng ký `ITokenRepository` và `TokenRepository` vào hệ thống Dependency Injection của ASP.NET Core với lifetime `Scoped`.
    *   Inject `ITokenRepository` vào `AuthController` và sử dụng nó trong phương thức `Login` để phát hành JWT sau khi xác thực thành công, trả về token trong `LoginResponseDTO`.
*   **Ủy quyền Dựa trên Vai trò (Role-Based Authorization):**
    *   Phân biệt rõ ràng giữa xác thực ("Bạn là ai?") và ủy quyền ("Bạn được phép làm gì?").
    *   Sử dụng thuộc tính `[Authorize(Roles = "RoleName")]` trên các phương thức hành động hoặc toàn bộ controller để kiểm soát quyền truy cập dựa trên vai trò của người dùng.
    *   Hướng dẫn kiểm thử các endpoint được bảo vệ bằng Postman, quan sát và hiểu ý nghĩa của các phản hồi `401 Unauthorized` và `403 Forbidden`.
*   **Tích hợp Ủy quyền vào Swagger UI:**
    *   Cấu hình `AddSwaggerGen` trong `Program.cs` để thêm định nghĩa bảo mật cho JWT Bearer, cho phép Swagger UI hiển thị giao diện nhập token.
    *   Cấu hình middleware `AddAuthentication().AddJwtBearer()` và `AddAuthorization()` trong `Program.cs` để xử lý và xác thực JWT từ các yêu cầu đến, đồng thời đảm bảo thứ tự middleware chính xác (`UseAuthentication` trước `UseAuthorization`).
    *   Hướng dẫn cách sử dụng nút "Authorize" trong Swagger UI để nhập JWT và kiểm thử các API được xác thực và ủy quyền một cách tiện lợi.

Với những kiến thức và triển khai này, bạn đã xây dựng được một hệ thống xác thực và ủy quyền mạnh mẽ, linh hoạt, sẵn sàng bảo vệ các tài nguyên API của mình khỏi truy cập trái phép.

<!-- REVIEWED_BY_AGENT -->
