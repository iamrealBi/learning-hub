# Chương 10: Bảo Mật API với Xác Thực & Ủy Quyền (Phần 1)

Chào mừng bạn đến với chương 10, nơi chúng ta sẽ đặt nền móng vững chắc cho tính bảo mật của RESTful Web API bằng ASP.NET Core. Trong các chương trước, chúng ta đã xây dựng kiến trúc cốt lõi của API, từ việc định nghĩa các điểm cuối, xác thực mô hình dữ liệu (model validation), đến việc triển khai Repository Pattern kết hợp với Dependency Injection để quản lý dữ liệu hiệu quả. Giờ đây, để đưa API của chúng ta vào môi trường thực tế một cách an toàn, việc hiểu và triển khai các cơ chế **Xác thực (Authentication)** và **Ủy quyền (Authorization)** là điều không thể thiếu.

Mục tiêu của chương này là trang bị cho bạn kiến thức và kỹ năng ban đầu để thiết lập một hệ thống bảo mật cơ bản nhưng mạnh mẽ cho API của mình. Cụ thể, chúng ta sẽ:

*   Phân biệt rõ ràng giữa Xác thực và Ủy quyền, và hiểu tầm quan trọng của chúng đối với bảo mật API.
*   Tìm hiểu sâu về luồng xác thực dựa trên **JSON Web Token (JWT)**, một tiêu chuẩn phổ biến cho việc truyền tải thông tin an toàn giữa các bên.
*   Thiết lập môi trường ứng dụng ASP.NET Core để hỗ trợ xác thực JWT, bao gồm cài đặt các gói NuGet cần thiết và cấu hình trong `appsettings.json` và `Program.cs`.
*   Tích hợp **ASP.NET Core Identity** để quản lý người dùng và vai trò, bao gồm việc tạo `IdentityDbContext` riêng biệt, cấu hình chuỗi kết nối, gieo dữ liệu vai trò (seeding roles), và thực hiện các thao tác di chuyển (migrations) Entity Framework Core để tạo cơ sở dữ liệu Identity.
*   Hiểu cách bảo vệ các điểm cuối API bằng thuộc tính `[Authorize]` và cách ứng dụng phản hồi khi người dùng chưa được xác thực truy cập.

Xuyên suốt chương này, chúng ta sẽ tập trung vào các khái niệm cốt lõi, cách triển khai thực tế bằng C# và ASP.NET Core, đồng thời nhấn mạnh vai trò của Dependency Injection trong việc quản lý các dịch vụ bảo mật và cách các thành phần khác nhau tương tác trong kiến trúc ứng dụng.

---

## 1. Nền Tảng Bảo Mật API: Xác Thực và Ủy Quyền

Trước khi đi sâu vào kỹ thuật, điều quan trọng là phải có một sự hiểu biết rõ ràng và phân biệt chính xác giữa hai khái niệm nền tảng này, vì chúng là hai trụ cột của bất kỳ hệ thống bảo mật ứng dụng nào.

### 1.1. Xác Thực (Authentication): "Bạn là ai?"

**Xác thực** là quá trình xác minh danh tính của một người dùng, một hệ thống hoặc một ứng dụng khách (client). Mục tiêu chính là trả lời câu hỏi: "Bạn có thực sự là người bạn tuyên bố không?".

Thông thường, quá trình này liên quan đến việc người dùng cung cấp một số thông tin chứng minh danh tính, chẳng hạn như:

*   **Tên người dùng và mật khẩu:** Phổ biến nhất.
*   **Chứng chỉ số (Digital Certificates):** Dùng cho xác thực máy chủ hoặc ứng dụng.
*   **Mã OTP (One-Time Password):** Thường kết hợp với các phương thức khác (xác thực đa yếu tố).
*   **Token:** Như JWT mà chúng ta sẽ tìm hiểu.

**Ví dụ thực tế:** Hãy hình dung việc xác thực giống như khi bạn xuất trình Chứng minh nhân dân/Căn cước công dân của mình cho nhân viên bảo vệ tại cổng vào một tòa nhà. Mục đích là để chứng minh bạn là một cá nhân đã biết và được phép có mặt ở đó.

**Tại sao API cần Xác thực?**
Nếu một API không có xác thực, bất kỳ ai biết URL của các điểm cuối đều có thể truy cập, đọc, sửa đổi hoặc thậm chí xóa dữ liệu. Điều này tạo ra rủi ro bảo mật nghiêm trọng, đặc biệt đối với dữ liệu nhạy cảm. Bằng cách triển khai xác thực, chúng ta đảm bảo rằng chỉ những người dùng hoặc ứng dụng khách đã được xác minh danh tính mới có thể tương tác với API, bảo vệ dữ liệu quý giá khỏi truy cập trái phép.

### 1.2. Ủy Quyền (Authorization): "Bạn được làm gì?"

**Ủy quyền** là quá trình xác định xem một người dùng (đã được xác thực thành công) có được phép thực hiện một hành động cụ thể hoặc truy cập một tài nguyên cụ thể hay không. Mục tiêu chính là trả lời câu hỏi: "Bạn có quyền thực hiện hành động này không?".

Ủy quyền thường dựa trên các yếu tố như:

*   **Vai trò (Roles):** Ví dụ: "Quản trị viên", "Người đọc", "Người viết".
*   **Chính sách (Policies):** Các quy tắc phức tạp hơn dựa trên nhiều yêu cầu (claims) hoặc điều kiện.
*   **Yêu cầu (Claims):** Các mảnh thông tin về người dùng, ví dụ: "tuổi trên 18", "thuộc phòng ban Marketing".

**Ví dụ thực tế:** Sau khi bạn đã xuất trình CCCD và được xác thực để vào tòa nhà, ủy quyền sẽ xác định bạn được phép đi vào những tầng nào, những phòng nào, hoặc sử dụng những thiết bị nào bên trong. Một người có vai trò "Khách" chỉ có thể vào sảnh chờ, trong khi "Nhân viên" có thể vào văn phòng của họ.

**Tại sao API cần Ủy quyền?**
Ngay cả khi người dùng đã được xác thực, không phải tất cả họ đều có quyền thực hiện mọi hành động. Ví dụ, một người dùng thông thường có thể xem danh sách sản phẩm, nhưng chỉ người dùng có vai trò "Quản trị viên" mới có thể thêm, sửa đổi hoặc xóa sản phẩm. Ủy quyền giúp chúng ta kiểm soát chi tiết quyền truy cập, đảm bảo rằng mỗi người dùng chỉ có thể thực hiện những hành động mà họ được phép, duy trì tính toàn vẹn và bảo mật của ứng dụng.

### 1.3. Mối Quan Hệ Cộng Sinh: 401 Unauthorized vs. 403 Forbidden

Xác thực và ủy quyền hoạt động song song để tạo nên một hệ thống bảo mật hoàn chỉnh. Xác thực là bước đầu tiên để biết "ai" đang truy cập, còn ủy quyền là bước thứ hai để biết "người đó được phép làm gì". Không có một trong hai, API của bạn sẽ không an toàn.

Khi một yêu cầu đến API, quy trình kiểm tra diễn ra như sau:

1.  **Xác thực:** API kiểm tra xem yêu cầu có đính kèm thông tin xác thực hợp lệ hay không (ví dụ: một JWT).
    *   Nếu không có thông tin xác thực hoặc thông tin không hợp lệ: API trả về mã trạng thái **`401 Unauthorized`**. Điều này có nghĩa là "Tôi không biết bạn là ai, hãy cung cấp thông tin xác thực hợp lệ."
2.  **Ủy quyền:** Nếu người dùng đã được xác thực, API sẽ kiểm tra xem người dùng đó có quyền thực hiện hành động được yêu cầu trên tài nguyên đó hay không.
    *   Nếu người dùng không có quyền: API trả về mã trạng thái **`403 Forbidden`**. Điều này có nghĩa là "Tôi biết bạn là ai, nhưng bạn không có quyền làm điều đó."

Hiểu rõ sự khác biệt giữa `401` và `403` là rất quan trọng để cung cấp phản hồi chính xác cho ứng dụng khách và để khắc phục sự cố bảo mật.

---

## 2. JSON Web Token (JWT): Tiêu Chuẩn Xác Thực Phi Trạng Thái

Trong các ứng dụng web hiện đại, đặc biệt là RESTful API, **JSON Web Token (JWT)** là một lựa chọn phổ biến cho xác thực phi trạng thái (stateless authentication). JWT là một tiêu chuẩn mở (RFC 7519) định nghĩa một cách nhỏ gọn và khép kín để truyền thông tin an toàn giữa các bên dưới dạng một đối tượng JSON.

### 2.1. Cấu Trúc JWT: Header, Payload, Signature

Một JWT là một chuỗi mã hóa, thường được sử dụng để xác thực và trao đổi thông tin giữa client và server. Nó bao gồm ba phần được phân tách bằng dấu chấm (`.`):

`Header.Payload.Signature`

**Ví dụ về JWT:**
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`

Hãy phân tích từng phần:

1.  **Header (Tiêu đề):**
    *   Là một đối tượng JSON chứa thông tin về loại token (JWT) và thuật toán mã hóa được sử dụng để ký token (ví dụ: `HS256`, `RS256`).
    *   Sau đó được mã hóa Base64Url.
    *   **Ví dụ (decoded):**
        ```json
        {
          "alg": "HS256", // Thuật toán ký: HMAC-SHA256
          "typ": "JWT"    // Loại token: JSON Web Token
        }
        ```

2.  **Payload (Tải trọng):**
    *   Là một đối tượng JSON chứa các **yêu cầu (claims)**. Các claims là các mảnh thông tin về người dùng và các siêu dữ liệu khác.
    *   Có ba loại claims:
        *   **Registered Claims:** Các claims được định nghĩa trước bởi JWT specification để đảm bảo tính tương thích (ví dụ: `iss` (issuer - nhà phát hành), `exp` (expiration time - thời gian hết hạn), `sub` (subject - chủ thể), `aud` (audience - đối tượng), `nbf` (not before), `iat` (issued at)).
        *   **Public Claims:** Có thể được định nghĩa bởi bất kỳ ai nhưng cần được đăng ký để tránh xung đột.
        *   **Private Claims:** Các claims tùy chỉnh được tạo ra để chia sẻ thông tin giữa các bên đồng ý, không cần đăng ký. Ví dụ: `roles`, `userId`.
    *   Sau đó được mã hóa Base64Url.
    *   **Ví dụ (decoded):**
        ```json
        {
          "sub": "1234567890",
          "name": "John Doe",
          "iat": 1516239022, // Issued At (thời gian tạo token)
          "exp": 1516242622, // Expiration Time (thời gian hết hạn)
          "roles": ["Reader", "Writer"]
        }
        ```
    > [!CAUTION]
    > JWT chỉ đảm bảo tính toàn vẹn của dữ liệu (không bị thay đổi) và danh tính của người gửi (server tạo token), chứ không mã hóa thông tin bên trong Payload. Do đó, **TUYỆT ĐỐI KHÔNG lưu trữ thông tin nhạy cảm (như mật khẩu, thông tin cá nhân bí mật) trong Payload của JWT**, vì bất kỳ ai cũng có thể giải mã Base64Url để đọc nội dung của nó.

3.  **Signature (Chữ ký):**
    *   Được tạo bằng cách lấy Base64Url mã hóa của Header và Payload, kết hợp với một khóa bí mật (secret key) và thuật toán mã hóa đã chỉ định trong Header.
    *   Ví dụ, với thuật toán HS256, chữ ký được tạo như sau:
        `HMACSHA256( Base64UrlEncode(header) + "." + Base64UrlEncode(payload), secret_key)`

    *   Chữ ký này là phần quan trọng nhất, đảm bảo tính toàn vẹn của token. Nếu bất kỳ phần nào của Header hoặc Payload bị thay đổi sau khi token được tạo, chữ ký sẽ không hợp lệ khi server xác minh, và token sẽ bị từ chối. Điều này giúp ngăn chặn việc giả mạo token.

### 2.2. Luồng Xác Thực JWT Toàn Diện

Luồng xác thực JWT cung cấp một cơ chế hiệu quả và phi trạng thái để quản lý phiên người dùng:

1.  **Người dùng Đăng nhập:** Người dùng truy cập ứng dụng khách (ví dụ: một trang web SPA, ứng dụng di động) và nhập thông tin đăng nhập (tên người dùng và mật khẩu) vào biểu mẫu.
2.  **Ứng dụng khách Gửi thông tin đăng nhập:** Ứng dụng khách gửi thông tin đăng nhập này đến API (thường là qua một điểm cuối `/login` hoặc `/register` sử dụng HTTP POST).
3.  **API Xác thực và Tạo JWT:**
    *   API nhận thông tin đăng nhập, kiểm tra tính hợp lệ (ví dụ: so khớp tên người dùng và mật khẩu với dữ liệu trong cơ sở dữ liệu Identity).
    *   Nếu thông tin hợp lệ, API sẽ tạo một JWT. Token này chứa các claims về người dùng (ví dụ: ID, vai trò, thời gian hết hạn) và được ký bằng khóa bí mật của server.
    *   API gửi JWT này trở lại ứng dụng khách trong phản hồi HTTP (thường là trong body JSON).
4.  **Ứng dụng khách Lưu trữ và Gửi JWT:**
    *   Ứng dụng khách nhận JWT và lưu trữ nó (thường là trong bộ nhớ cục bộ (LocalStorage), SessionStorage hoặc cookie, tùy thuộc vào loại ứng dụng và yêu cầu bảo mật).
    *   Đối với các yêu cầu HTTP tiếp theo đến các điểm cuối được bảo vệ của API, ứng dụng khách sẽ đính kèm JWT này vào tiêu đề `Authorization` dưới dạng `Bearer Token` (ví dụ: `Authorization: Bearer <your_jwt_token>`).
5.  **API Xác minh JWT và Cấp quyền truy cập:**
    *   Khi nhận được một yêu cầu có đính kèm JWT, API sẽ trích xuất token.
    *   API xác minh chữ ký của token bằng cách sử dụng khóa bí mật đã biết.
    *   API kiểm tra các claims khác như thời gian hết hạn (`exp`), nhà phát hành (`iss`), và đối tượng (`aud`).
    *   Nếu chữ ký hợp lệ và token chưa hết hạn, API sẽ coi người dùng là đã được xác thực và trích xuất thông tin từ Payload (ví dụ: vai trò của người dùng).
    *   API sau đó sử dụng thông tin trong Payload để thực hiện ủy quyền, tức là kiểm tra xem người dùng có quyền truy cập tài nguyên hoặc thực hiện hành động được yêu cầu hay không.
    *   Nếu xác minh thành công và được ủy quyền, API sẽ trả về dữ liệu hoặc thực hiện hành động. Nếu token không hợp lệ, thiếu, hoặc người dùng không được ủy quyền, API sẽ từ chối yêu cầu (thường là lỗi `401 Unauthorized` hoặc `403 Forbidden`).

**Ưu điểm của JWT:**

*   **Phi trạng thái (Stateless):** Server không cần lưu trữ thông tin phiên của người dùng. Mỗi yêu cầu chứa tất cả thông tin cần thiết để xác thực và ủy quyền, giúp API dễ dàng mở rộng theo chiều ngang (horizontal scaling).
*   **Hiệu quả:** Kích thước nhỏ gọn, dễ dàng truyền tải qua HTTP header.
*   **Tính di động:** Có thể được sử dụng trên nhiều nền tảng và dịch vụ.

**Nhược điểm của JWT:**

*   **Không có khả năng thu hồi ngay lập tức:** Một khi JWT đã được phát hành, nó vẫn hợp lệ cho đến khi hết hạn. Nếu bạn muốn thu hồi một token ngay lập tức (ví dụ: khi người dùng thay đổi mật khẩu hoặc bị cấm), bạn cần triển khai một cơ chế danh sách đen (blacklist) hoặc sử dụng refresh token.
*   **Kích thước:** Mặc dù nhỏ gọn, nhưng nếu chứa quá nhiều claims, kích thước của JWT có thể tăng lên và ảnh hưởng đến hiệu suất.
*   **Bảo mật Payload:** Như đã đề cập, thông tin trong Payload không được mã hóa, chỉ được ký.

---

## 3. Tích Hợp ASP.NET Core Identity: Quản Lý Người Dùng & Vai Trò

Để quản lý người dùng, mật khẩu, và vai trò một cách mạnh mẽ và an toàn, chúng ta sẽ sử dụng **ASP.NET Core Identity**. Đây là một hệ thống thành viên được thiết kế để xây dựng các tính năng đăng nhập, đăng ký, xác nhận email, đặt lại mật khẩu và quản lý vai trò.

### 3.1. Giới Thiệu ASP.NET Core Identity

ASP.NET Core Identity cung cấp một API toàn diện để quản lý thông tin người dùng, bao gồm:

*   **Lưu trữ người dùng và vai trò:** Sử dụng Entity Framework Core để lưu trữ dữ liệu vào cơ sở dữ liệu.
*   **Quản lý mật khẩu:** Hashing mật khẩu an toàn, đổi mật khẩu.
*   **Xác nhận tài khoản:** Email, số điện thoại.
*   **Xác thực đa yếu tố (MFA).**
*   **Quản lý phiên đăng nhập.**

Trong ngữ cảnh của một RESTful API, chúng ta sẽ sử dụng Identity để quản lý cơ sở dữ liệu người dùng và vai trò, nhưng sẽ không sử dụng các UI mặc định của nó. Thay vào đó, chúng ta sẽ xây dựng các điểm cuối API tùy chỉnh để người dùng đăng ký và đăng nhập, trả về JWT.

### 3.2. Cấu Hình Chuỗi Kết Nối cho Identity Database

Để Identity có thể lưu trữ dữ liệu của nó, chúng ta cần một cơ sở dữ liệu riêng. Việc tách biệt cơ sở dữ liệu Identity khỏi cơ sở dữ liệu chính của ứng dụng (ví dụ: `NZWalksDb`) là một thực hành tốt, giúp quản lý và bảo mật dữ liệu hiệu quả hơn.

Trong tệp `appsettings.json`, chúng ta đã thêm một chuỗi kết nối mới có tên `NZWalksAuthConnectionString`:

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
    "NZWalksConnectionString": "Server=(localdb)\\mssqllocaldb;Database=NZWalksDb;Trusted_Connection=True;MultipleActiveResultSets=true",
    // Chuỗi kết nối mới cho cơ sở dữ liệu Identity
    "NZWalksAuthConnectionString": "Server=(localdb)\\mssqllocaldb;Database=NZWalksAuthDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "JWT": {
    "Key": "Đây là một chuỗi khóa bí mật rất dài và phức tạp, hãy tạo một chuỗi ngẫu nhiên có độ dài lớn hơn 32 ký tự để đảm bảo an toàn",
    "Issuer": "https://localhost:7119", // Hoặc URL của ứng dụng/API của bạn
    "Audience": "https://localhost:7119" // Hoặc URL của ứng dụng/API của bạn
  }
}
```

Chuỗi kết nối này sẽ trỏ đến một cơ sở dữ liệu SQL Server mới có tên `NZWalksAuthDb`, chuyên dùng để lưu trữ dữ liệu liên quan đến Identity (người dùng, vai trò, claims, v.v.).

### 3.3. Định Nghĩa IdentityDbContext Riêng Biệt

Chúng ta sẽ tạo một lớp `DbContext` riêng biệt kế thừa từ `IdentityDbContext` để quản lý các bảng của Identity. Điều này giúp tách biệt dữ liệu chính của ứng dụng (`NZWalksDb`) khỏi dữ liệu xác thực (`NZWalksAuthDb`), tăng cường tính module hóa và dễ quản lý.

Tạo một thư mục `Data` (nếu chưa có) và thêm một lớp mới tên là `NZWalksAuthDbContext.cs`:

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace NZWalks.API.Data
{
    // Kế thừa từ IdentityDbContext, sử dụng IdentityUser làm kiểu người dùng mặc định
    // IdentityDbContext cung cấp các DbSet cho IdentityUser, IdentityRole và các bảng liên quan
    public class NZWalksAuthDbContext : IdentityDbContext
    {
        // Hàm tạo phải nhận DbContextOptions<T> để Dependency Injection có thể phân biệt
        // giữa các DbContext khi có nhiều hơn một trong ứng dụng.
        public NZWalksAuthDbContext(DbContextOptions<NZWalksAuthDbContext> options) : base(options)
        {
        }

        // Phương thức này được ghi đè để cấu hình mô hình dữ liệu, bao gồm việc gieo dữ liệu (seeding)
        protected override void OnModelCreating(ModelBuilder builder)
        {
            // Gọi phương thức OnModelCreating của lớp cơ sở để IdentityDbContext tự tạo các bảng mặc định
            base.OnModelCreating(builder);

            // Gieo dữ liệu Vai trò (Roles) vào cơ sở dữ liệu
            // Đây là các vai trò ban đầu mà ứng dụng sẽ sử dụng
            var readerRoleId = "0a9e7f8d-c1b2-4a3d-9e6f-1b2c3d4e5f6a"; // GUID ngẫu nhiên cho Reader Role
            var writerRoleId = "1b2c3d4e-5f6a-7b8c-9d0e-1f2a3b4c5d6e"; // GUID ngẫu nhiên cho Writer Role

            var roles = new List<IdentityRole>
            {
                new IdentityRole
                {
                    Id = readerRoleId,
                    ConcurrencyStamp = readerRoleId, // Dùng cho xử lý đồng thời, giúp tránh xung đột khi nhiều người sửa đổi cùng một lúc
                    Name = "Reader",
                    NormalizedName = "READER" // Tên chuẩn hóa (thường là chữ hoa) để tìm kiếm và so sánh không phân biệt chữ hoa/thường hiệu quả hơn
                },
                new IdentityRole
                {
                    Id = writerRoleId,
                    ConcurrencyStamp = writerRoleId,
                    Name = "Writer",
                    NormalizedName = "WRITER"
                }
            };

            // Thêm các vai trò này vào bảng AspNetRoles trong cơ sở dữ liệu khi chạy migration
            builder.Entity<IdentityRole>().HasData(roles);
        }
    }
}
```

**Giải thích chi tiết:**

*   **`IdentityDbContext`**: Đây là một phiên bản đặc biệt của `DbContext` được thiết kế để làm việc với ASP.NET Core Identity. Nó đã định nghĩa sẵn các `DbSet` cho `IdentityUser`, `IdentityRole`, `IdentityUserClaim`, `IdentityUserRole`, `IdentityUserLogin`, `IdentityRoleClaim`, và `IdentityUserToken`, giúp bạn không cần tự định nghĩa các bảng này.
*   **`DbContextOptions<NZWalksAuthDbContext> options`**: Khi có nhiều `DbContext` trong một ứng dụng, bạn cần chỉ định rõ ràng kiểu `DbContext` cho `DbContextOptions` trong hàm tạo. Điều này giúp hệ thống Dependency Injection của ASP.NET Core biết chính xác phiên bản `DbContext` nào cần tiêm khi có nhiều lựa chọn, tránh lỗi mơ hồ.
*   **`OnModelCreating(ModelBuilder builder)`**: Phương thức này được ghi đè để tùy chỉnh cách Entity Framework Core xây dựng mô hình cơ sở dữ liệu. Chúng ta sử dụng nó ở đây để gieo dữ liệu (seed data) cho các vai trò ban đầu.
*   **Gieo dữ liệu Vai trò (Seeding Roles)**:
    *   Chúng ta tạo một danh sách các đối tượng `IdentityRole`.
    *   Mỗi vai trò có một `Id` (GUID duy nhất), `ConcurrencyStamp` (một giá trị được sử dụng bởi Entity Framework Core để xử lý các kịch bản đồng thời, giúp phát hiện và ngăn chặn các xung đột khi nhiều người dùng cố gắng cập nhật cùng một bản ghi), `Name` (tên hiển thị của vai trò), và `NormalizedName` (một phiên bản chuẩn hóa của `Name`, thường là chữ hoa, được Identity sử dụng để tìm kiếm và so sánh vai trò một cách hiệu quả và không phân biệt chữ hoa/thường).
    *   `builder.Entity<IdentityRole>().HasData(roles);` yêu cầu Entity Framework Core thêm các vai trò này vào bảng `AspNetRoles` khi quá trình di chuyển (migration) được áp dụng vào cơ sở dữ liệu. Điều này đảm bảo rằng các vai trò cơ bản luôn có sẵn ngay từ đầu.

### 3.4. Áp Dụng Entity Framework Core Migrations

Sau khi đã định nghĩa `NZWalksAuthDbContext` và cấu hình gieo dữ liệu vai trò, chúng ta cần tạo và áp dụng các di chuyển (migrations) để Entity Framework Core tạo cơ sở dữ liệu `NZWalksAuthDb` cùng với tất cả các bảng Identity và dữ liệu vai trò đã gieo.

> [!AI_VIBE]
> Với Antigravity IDE, bạn không cần mở Package Manager Console thủ công. Bạn có thể áp dụng tư duy Vibe Coding bằng cách mô tả ý định của mình: "Tôi muốn tạo migration ban đầu cho `NZWalksAuthDbContext` và sau đó áp dụng nó vào cơ sở dữ liệu." Antigravity sẽ hiểu và thực thi các lệnh sau đây một cách tự động, giúp bạn tập trung vào luồng công việc mà không bị phân tâm bởi cú pháp lệnh.

1.  **Mở Package Manager Console (PMC)** trong Visual Studio (Tools -> NuGet Package Manager -> Package Manager Console).
2.  **Thêm Migration:** Chạy lệnh sau để tạo một migration mới. **Lưu ý quan trọng:** Bạn phải chỉ định `DbContext` nào sẽ được sử dụng bằng tham số `-Context` vì chúng ta có nhiều `DbContext` trong dự án.

    ```bash
    Add-Migration "CreateAuthDatabase" -Context NZWalksAuthDbContext
    ```

    Lệnh này sẽ tạo một thư mục `Migrations` mới trong dự án của bạn (nếu chưa có) và một tệp migration chứa mã C# để tạo các bảng Identity như `AspNetUsers`, `AspNetRoles`, `AspNetUserRoles`, v.v., trong cơ sở dữ liệu `NZWalksAuthDb`.

3.  **Cập nhật Database:** Sau khi migration đã được tạo, chạy lệnh sau để áp dụng nó vào cơ sở dữ liệu.

    ```bash
    Update-Database -Context NZWalksAuthDbContext
    ```

    Lệnh này sẽ tạo cơ sở dữ liệu `NZWalksAuthDb` (nếu chưa tồn tại) và các bảng Identity trong SQL Server. Nó cũng sẽ chèn hai vai trò "Reader" và "Writer" vào bảng `AspNetRoles` nhờ vào cấu hình `HasData` trong `OnModelCreating`.

> [!TIP]
> Bạn có thể kiểm tra cơ sở dữ liệu SQL Server của mình bằng SQL Server Management Studio hoặc Server Explorer trong Visual Studio để xác nhận rằng `NZWalksAuthDb` đã được tạo và chứa các bảng Identity cùng với dữ liệu vai trò trong bảng `AspNetRoles`.

---

## 4. Thiết Lập Xác Thực JWT trong ASP.NET Core API

Để triển khai xác thực JWT, chúng ta cần cài đặt các gói NuGet cần thiết, cấu hình thông tin JWT trong `appsettings.json`, và đăng ký các dịch vụ xác thực cùng middleware trong `Program.cs`.

### 4.1. Cài Đặt Các Gói NuGet Cần Thiết

Mở cửa sổ Package Manager Console hoặc sử dụng giao diện người dùng NuGet Package Manager, sau đó cài đặt các gói sau vào dự án Web API của bạn:

```bash
Install-Package Microsoft.AspNetCore.Authentication.JwtBearer
Install-Package Microsoft.AspNetCore.Identity.EntityFrameworkCore
Install-Package Microsoft.AspNetCore.Identity.UI
Install-Package Microsoft.IdentityModel.Tokens
Install-Package System.IdentityModel.Tokens.Jwt
```

**Giải thích các gói:**

*   `Microsoft.AspNetCore.Authentication.JwtBearer`: Cung cấp middleware để xử lý xác thực JWT Bearer token.
*   `Microsoft.AspNetCore.Identity.EntityFrameworkCore`: Cung cấp các lớp Entity Framework Core để lưu trữ dữ liệu Identity vào cơ sở dữ liệu.
*   `Microsoft.AspNetCore.Identity.UI`: Mặc dù chúng ta không sử dụng giao diện người dùng (UI) mặc định của Identity trong API thuần túy, gói này thường được cài đặt cùng với `Identity.EntityFrameworkCore` và cung cấp các thành phần cốt lõi cho Identity.
*   `Microsoft.IdentityModel.Tokens`: Cung cấp các lớp và giao diện cơ bản để làm việc với các token bảo mật, bao gồm việc tạo và xác thực token.
*   `System.IdentityModel.Tokens.Jwt`: Cung cấp các lớp để tạo, đọc và xác thực JSON Web Tokens.

### 4.2. Cấu Hình Thông Số JWT trong `appsettings.json`

Chúng ta cần lưu trữ các thông tin cấu hình cho JWT như khóa bí mật, nhà phát hành (issuer) và đối tượng (audience) trong tệp `appsettings.json`. Điều này giúp dễ dàng quản lý và thay đổi cấu hình mà không cần biên dịch lại mã nguồn.

Thêm một đối tượng `JWT` mới vào `appsettings.json` của bạn:

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
    "NZWalksConnectionString": "Server=(localdb)\\mssqllocaldb;Database=NZWalksDb;Trusted_Connection=True;MultipleActiveResultSets=true",
    "NZWalksAuthConnectionString": "Server=(localdb)\\mssqllocaldb;Database=NZWalksAuthDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "JWT": {
    "Key": "DayLaMotChuoiKhoaBiMatRatDaiVaPhucTapMaBanNenTaoNgauNhienVaCoDoDaiLonHon32KyTuDeDamBaoAnToanTuyetDoiChoHeThongCuaBan", // Chuỗi khóa bí mật
    "Issuer": "https://localhost:7119", // URL của ứng dụng/API của bạn (phải khớp với URL thực tế)
    "Audience": "https://localhost:7119" // URL của ứng dụng/API mà JWT này được cấp quyền truy cập
  }
}
```

*   **Key (Khóa):** Đây là khóa bí mật được sử dụng để ký và xác minh JWT. Nó phải là một chuỗi dài, ngẫu nhiên và mạnh mẽ (thường là ít nhất 32 ký tự để đảm bảo an toàn với HS256). Tuyệt đối không chia sẻ khóa này.
*   **Issuer (Nhà phát hành):** URL của máy chủ hoặc ứng dụng phát hành JWT. Khi API nhận được JWT, nó sẽ kiểm tra xem `iss` claim trong token có khớp với giá trị `Issuer` đã cấu hình hay không.
*   **Audience (Đối tượng):** URL của ứng dụng hoặc tài nguyên mà JWT này được cấp quyền truy cập. Tương tự, API sẽ kiểm tra `aud` claim trong token.

> [!CAUTION]
> Trong môi trường sản phẩm, khóa bí mật (Key) **TUYỆT ĐỐI KHÔNG** nên được lưu trữ trực tiếp trong `appsettings.json`. Thay vào đó, hãy sử dụng các biến môi trường (Environment Variables), Azure Key Vault, AWS Secrets Manager, hoặc các giải pháp quản lý bí mật an toàn khác. Điều này ngăn chặn việc khóa bị lộ nếu mã nguồn bị truy cập trái phép.

> [!AI_VIBE]
> Khi cấu hình các thông số nhạy cảm như khóa JWT, bạn có thể hỏi Antigravity IDE: "Làm thế nào để tôi lưu trữ khóa JWT một cách an toàn hơn trong môi trường sản phẩm thay vì trong `appsettings.json`?" Antigravity sẽ cung cấp hướng dẫn chi tiết về việc sử dụng biến môi trường hoặc các dịch vụ quản lý bí mật đám mây, thậm chí có thể đề xuất các đoạn mã cấu hình cần thiết để đọc chúng trong `Program.cs`.

### 4.3. Đăng Ký Dịch Vụ Identity và JWT Bearer trong `Program.cs`

Tệp `Program.cs` là nơi chúng ta cấu hình các dịch vụ (Dependency Injection) và middleware của ứng dụng. Đây là trung tâm để thiết lập toàn bộ hệ thống xác thực.

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using NZWalks.API.Data;
using NZWalks.API.Mappings;
using NZWalks.API.Repositories;
using System.Text; // Cần thiết cho Encoding.UTF8.GetBytes

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
// builder.Services.AddLocalization(); // Bỏ ghi chú nếu cần hỗ trợ đa ngôn ngữ

// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Cấu hình CORS (Cross-Origin Resource Sharing)
// Cho phép các ứng dụng khách từ các nguồn khác truy cập API
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.AllowAnyOrigin() // Cho phép bất kỳ nguồn gốc nào (chỉ nên dùng cho phát triển)
              .AllowAnyHeader()  // Cho phép bất kỳ tiêu đề nào
              .AllowAnyMethod(); // Cho phép bất kỳ phương thức HTTP nào (GET, POST, PUT, DELETE)
    });
});

// Đăng ký DbContext cho dữ liệu chính của ứng dụng (NZWalksDb)
builder.Services.AddDbContext<NZWalksDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("NZWalksConnectionString"));
});

// Đăng ký DbContext cho Identity (Xác thực & Ủy quyền) (NZWalksAuthDb)
// Đảm bảo rằng mỗi DbContext được cấu hình với chuỗi kết nối riêng
builder.Services.AddDbContext<NZWalksAuthDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("NZWalksAuthConnectionString"));
});

// Đăng ký các Repository Pattern vào Dependency Injection container
// Điều này giúp tách biệt logic truy cập dữ liệu khỏi các controller
builder.Services.AddScoped<IRegionRepository, SQLRegionRepository>();
builder.Services.AddScoped<IWalkRepository, SQLWalkRepository>();
// ITokenRepository sẽ được tạo trong chương tiếp theo để quản lý JWT
builder.Services.AddScoped<ITokenRepository, TokenRepository>(); 

// Đăng ký AutoMapper để tự động ánh xạ giữa các Domain Model và DTO
builder.Services.AddAutoMapper(typeof(AutoMapperProfiles));

// ------------------------------------------------------------------------------------------
// CẤU HÌNH ASP.NET CORE IDENTITY VÀ XÁC THỰC JWT
// Đây là phần cốt lõi của việc thiết lập bảo mật
// ------------------------------------------------------------------------------------------

// 1. Cấu hình ASP.NET Core Identity
// AddIdentityCore đăng ký các dịch vụ cốt lõi của Identity mà không bao gồm UI
builder.Services.AddIdentityCore<IdentityUser>() // Sử dụng IdentityUser mặc định của ASP.NET Core
    .AddRoles<IdentityRole>() // Thêm hỗ trợ cho các vai trò (Roles)
    .AddTokenProvider<DataProtectorTokenProvider<IdentityUser>>("NZWalks") // Thêm nhà cung cấp token cho các mục đích như reset mật khẩu, xác nhận email
    .AddEntityFrameworkStores<NZWalksAuthDbContext>() // Chỉ định DbContext mà Identity sẽ sử dụng để lưu trữ dữ liệu người dùng và vai trò
    .AddDefaultTokenProviders(); // Thêm các nhà cung cấp token mặc định khác của Identity

// Cấu hình các tùy chọn cho Identity (ví dụ: yêu cầu mật khẩu)
builder.Services.Configure<IdentityOptions>(options =>
{
    // Cấu hình yêu cầu về mật khẩu để đơn giản hóa trong quá trình học
    // Trong môi trường sản phẩm, hãy đặt các yêu cầu mạnh mẽ hơn
    options.Password.RequireDigit = false;          // Không yêu cầu chữ số
    options.Password.RequireLowercase = false;      // Không yêu cầu chữ thường
    options.Password.RequireNonAlphanumeric = false; // Không yêu cầu ký tự đặc biệt
    options.Password.RequireUppercase = false;      // Không yêu cầu chữ hoa
    options.Password.RequiredLength = 6;            // Độ dài tối thiểu là 6 ký tự
    options.Password.RequiredUniqueChars = 1;       // Số ký tự độc đáo tối thiểu
});

// 2. Cấu hình Xác thực JWT Bearer
// Đăng ký dịch vụ xác thực và chỉ định lược đồ mặc định là JWT Bearer
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        // TokenValidationParameters chứa các tham số mà API sẽ sử dụng để xác thực JWT đến
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true, // Yêu cầu xác thực nhà phát hành (Issuer) của token
            ValidateAudience = true, // Yêu cầu xác thực đối tượng (Audience) của token
            ValidateLifetime = true, // Yêu cầu xác thực thời gian sống (hết hạn) của token
            ValidateIssuerSigningKey = true, // Yêu cầu xác thực khóa ký (Signature) của token

            // Lấy nhà phát hành hợp lệ từ cấu hình appsettings.json
            ValidIssuer = builder.Configuration["JWT:Issuer"], 
            // Lấy đối tượng hợp lệ từ cấu hình appsettings.json
            ValidAudience = builder.Configuration["JWT:Audience"], 
            // Lấy khóa bí mật từ cấu hình appsettings.json và chuyển đổi thành SymmetricSecurityKey
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["JWT:Key"])) 
        };
    });

// ------------------------------------------------------------------------------------------

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// ------------------------------------------------------------------------------------------
// THÊM MIDDLEWARE XÁC THỰC VÀ ỦY QUYỀN VÀO PIPELINE
// Thứ tự của các middleware này là RẤT QUAN TRỌNG
// ------------------------------------------------------------------------------------------

app.UseAuthentication(); // Middleware này phải được gọi TRƯỚC UseAuthorization
app.UseAuthorization();  // Middleware này dựa vào kết quả của UseAuthentication

// ------------------------------------------------------------------------------------------

app.MapControllers();

app.Run();
```

**Giải thích chi tiết các bước cấu hình trong `Program.cs`:**

1.  **`AddIdentityCore<IdentityUser>()`**: Đăng ký các dịch vụ cốt lõi của ASP.NET Core Identity vào hệ thống Dependency Injection. `IdentityUser` là lớp người dùng mặc định mà Identity sử dụng. `AddIdentityCore` được chọn thay vì `AddIdentity` vì nó chỉ thêm các dịch vụ Identity cốt lõi mà không bao gồm các dịch vụ liên quan đến UI (như Razor Pages), phù hợp cho API thuần túy.
2.  **`.AddRoles<IdentityRole>()`**: Mở rộng Identity để thêm hỗ trợ cho các vai trò (roles). Điều này cho phép chúng ta gán các vai trò như "Reader" hoặc "Writer" cho người dùng, phục vụ cho việc ủy quyền dựa trên vai trò.
3.  **`.AddTokenProvider<DataProtectorTokenProvider<IdentityUser>>("NZWalks")`**: Đăng ký một nhà cung cấp token. Identity sử dụng các nhà cung cấp token để tạo ra các token dùng cho các mục đích như xác nhận email, đặt lại mật khẩu. "NZWalks" là tên định danh cho nhà cung cấp này. `DataProtectorTokenProvider` sử dụng hệ thống bảo vệ dữ liệu của ASP.NET Core để tạo các token an toàn.
4.  **`.AddEntityFrameworkStores<NZWalksAuthDbContext>()`**: Chỉ định rằng ASP.NET Core Identity sẽ sử dụng Entity Framework Core để lưu trữ dữ liệu người dùng và vai trò vào cơ sở dữ liệu thông qua `NZWalksAuthDbContext` mà chúng ta đã tạo. Đây là cầu nối giữa Identity và cơ sở dữ liệu.
5.  **`.AddDefaultTokenProviders()`**: Đăng ký các nhà cung cấp token mặc định khác do Identity cung cấp, đảm bảo rằng tất cả các tính năng liên quan đến token (ví dụ: cho xác nhận tài khoản) đều hoạt động.
6.  **`builder.Services.Configure<IdentityOptions>(options => { ... });`**: Cho phép chúng ta tùy chỉnh các quy tắc và yêu cầu cho mật khẩu của người dùng, chẳng hạn như độ dài tối thiểu, yêu cầu chữ hoa/chữ thường, chữ số, ký tự đặc biệt. Trong ví dụ này, chúng ta đã nới lỏng các yêu cầu để dễ dàng hơn trong quá trình phát triển và học tập. **Trong môi trường sản phẩm, hãy luôn đặt các yêu cầu mật khẩu mạnh mẽ.**
7.  **`AddAuthentication(JwtBearerDefaults.AuthenticationScheme)`**: Đăng ký dịch vụ xác thực vào DI container và chỉ định rằng lược đồ xác thực mặc định cho ứng dụng là JWT Bearer. Điều này cho ASP.NET Core biết cách xử lý các yêu cầu xác thực.
8.  **`.AddJwtBearer(options => { ... });`**: Cấu hình các tùy chọn cụ thể cho xác thực JWT Bearer.
    *   **`TokenValidationParameters`**: Đây là đối tượng quan trọng chứa các tham số mà API sẽ sử dụng để xác thực JWT đến.
        *   `ValidateIssuer`, `ValidateAudience`, `ValidateLifetime`, `ValidateIssuerSigningKey`: Đặt tất cả thành `true` là một thực hành bảo mật tốt để đảm bảo token được xác thực một cách chặt chẽ trên tất cả các khía cạnh quan trọng.
        *   `ValidIssuer`, `ValidAudience`: Lấy giá trị từ `appsettings.json` để so sánh với các giá trị `iss` và `aud` trong token. Nếu không khớp, token sẽ bị từ chối.
        *   `IssuerSigningKey`: Khóa bí mật được sử dụng để xác minh chữ ký của token. Nó phải giống với khóa đã dùng để tạo token. Chúng ta sử dụng `Encoding.UTF8.GetBytes` để chuyển đổi chuỗi khóa từ `appsettings.json` thành mảng byte, vì khóa ký phải ở định dạng byte.
9.  **`app.UseAuthentication();`**: Thêm middleware xác thực vào pipeline yêu cầu HTTP. Middleware này sẽ kiểm tra mọi yêu cầu đến để xem có JWT hợp lệ hay không. Nếu có, nó sẽ giải mã token, xác minh chữ ký và các claims, sau đó tạo một `ClaimsPrincipal` để đại diện cho người dùng đã xác thực. **Điều quan trọng là nó phải được gọi trước `app.UseAuthorization();`**.
10. **`app.UseAuthorization();`**: Thêm middleware ủy quyền vào pipeline. Middleware này sẽ kiểm tra xem người dùng (đã được xác thực bởi `UseAuthentication`) có quyền truy cập tài nguyên được yêu cầu hay không, dựa trên các vai trò hoặc chính sách được định nghĩa bởi thuộc tính `[Authorize]` trên controller hoặc action.

> [!AI_VIBE]
> Khi đối mặt với một đoạn cấu hình dài như trong `Program.cs`, bạn có thể sử dụng Antigravity IDE để "vibe" với mã. Hãy chọn một phần cấu hình (ví dụ: `options.TokenValidationParameters = new TokenValidationParameters { ... }`) và hỏi Antigravity: "Giải thích từng thuộc tính trong `TokenValidationParameters` và tại sao chúng lại quan trọng cho bảo mật JWT?" Antigravity sẽ cung cấp một phân tích sâu sắc, giúp bạn hiểu rõ cơ chế hoạt động ngầm của từng tùy chọn.

### 4.4. Thêm Middleware Xác Thực & Ủy Quyền vào Pipeline

Thứ tự của các middleware trong pipeline xử lý yêu cầu HTTP của ASP.NET Core là rất quan trọng. `UseAuthentication` phải luôn được gọi trước `UseAuthorization`.

```csharp
// ...
app.UseHttpsRedirection();

// ------------------------------------------------------------------------------------------
// THÊM MIDDLEWARE XÁC THỰC VÀ ỦY QUYỀN VÀO PIPELINE
// ------------------------------------------------------------------------------------------

app.UseAuthentication(); // PHẢI ĐẶT TRƯỚC UseAuthorization
app.UseAuthorization();

// ------------------------------------------------------------------------------------------

app.MapControllers();

app.Run();
```

*   `app.UseAuthentication()`: Đây là middleware chịu trách nhiệm đọc và xác minh thông tin xác thực (ví dụ: JWT) từ tiêu đề yêu cầu. Nếu token hợp lệ, nó sẽ tạo một `ClaimsPrincipal` và gán nó vào `HttpContext.User`, đại diện cho người dùng đã được xác thực.
*   `app.UseAuthorization()`: Middleware này sẽ kiểm tra `HttpContext.User` (được thiết lập bởi `UseAuthentication`) để xác định xem người dùng có quyền truy cập vào tài nguyên được yêu cầu hay không, dựa trên các quy tắc ủy quyền (ví dụ: thuộc tính `[Authorize]`).

Nếu bạn đảo ngược thứ tự, `UseAuthorization` sẽ chạy trước khi `UseAuthentication` có cơ hội xác định danh tính người dùng, dẫn đến việc tất cả các yêu cầu đều bị từ chối truy cập vì không có người dùng nào được xác thực.

---

## 5. Kiểm Tra Khả Năng Bảo Vệ Điểm Cuối API

Sau khi đã cấu hình xác thực và ủy quyền, chúng ta có thể thử bảo vệ một điểm cuối API và xem cách ứng dụng phản hồi khi một yêu cầu không có token hợp lệ.

### 5.1. Bảo Vệ Controller bằng thuộc tính `[Authorize]`

Để bảo vệ một controller hoặc một phương thức hành động cụ thể, chúng ta chỉ cần thêm thuộc tính `[Authorize]` lên trên lớp controller hoặc phương thức đó.

Ví dụ, để bảo vệ `RegionsController`, chúng ta sẽ sửa đổi nó như sau:

```csharp
using Microsoft.AspNetCore.Authorization; // Đảm bảo có using này
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;
using AutoMapper;

namespace NZWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    [Authorize] // Thêm thuộc tính này để bảo vệ toàn bộ controller
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository regionRepository;
        private readonly IMapper mapper;

        public RegionsController(IRegionRepository regionRepository, IMapper mapper)
        {
            this.regionRepository = regionRepository;
            this.mapper = mapper;
        }

        // ... Các phương thức GET, POST, PUT, DELETE ...
        // Tất cả các phương thức trong controller này giờ đây đều yêu cầu xác thực
    }
}
```

Khi thuộc tính `[Authorize]` được áp dụng, bất kỳ yêu cầu nào đến các phương thức trong `RegionsController` đều yêu cầu người dùng phải được xác thực. Nếu người dùng không được xác thực, yêu cầu sẽ bị chặn trước khi đến logic của controller.

### 5.2. Quan Sát Phản Hồi `401 Unauthorized`

Nếu bạn chạy ứng dụng và cố gắng truy cập bất kỳ điểm cuối nào trong `RegionsController` (ví dụ: `/api/Regions`) thông qua Swagger hoặc một công cụ như Postman mà không đính kèm JWT hợp lệ, bạn sẽ nhận được phản hồi:

*   **Mã trạng thái HTTP:** `401 Unauthorized`
*   **Thông báo lỗi:** Thường là một thông báo trống hoặc một thông báo chung "Unauthorized", tùy thuộc vào cấu hình mặc định của ASP.NET Core.

> [!NOTE]
> Mã trạng thái `401 Unauthorized` có nghĩa là yêu cầu không có thông tin xác thực hợp lệ cho tài nguyên đích. Điều này là chính xác vì chúng ta chưa cung cấp bất kỳ thông tin xác thực nào (hoặc thông tin không hợp lệ). Nó khác với `403 Forbidden`, nghĩa là máy chủ hiểu yêu cầu nhưng từ chối cấp quyền truy cập vì người dùng không có quyền, mặc dù đã được xác thực.

Đây là dấu hiệu cho thấy hệ thống xác thực của chúng ta đang hoạt động như mong đợi. Bước tiếp theo là tạo ra một cơ chế để người dùng có thể đăng ký, đăng nhập và nhận được JWT hợp lệ để truy cập các tài nguyên được bảo vệ này.

> [!AI_VIBE]
> Để kiểm tra nhanh chóng phản hồi `401 Unauthorized` bằng Antigravity IDE, bạn có thể ra lệnh: "Gửi một yêu cầu GET đến `/api/Regions` mà không có tiêu đề `Authorization` và báo cáo mã trạng thái HTTP nhận được." Antigravity sẽ thực hiện yêu cầu, phân tích phản hồi và hiển thị mã trạng thái, giúp bạn xác minh ngay lập tức rằng điểm cuối đã được bảo vệ thành công.

---

## 6. Tóm Tắt Chương 10

Trong chương này, chúng ta đã đặt nền móng vững chắc cho hệ thống xác thực và ủy quyền của API bằng ASP.NET Core. Dưới đây là những điểm chính đã được thực hiện và các kiến thức đã được củng cố:

*   **Phân biệt Xác thực và Ủy quyền:** Nắm vững sự khác biệt giữa việc xác minh danh tính ("Bạn là ai?") và xác định quyền hạn truy cập ("Bạn được làm gì?"), cùng với ý nghĩa của mã trạng thái HTTP `401 Unauthorized` và `403 Forbidden`.
*   **Luồng Xác thực JWT:** Hiểu rõ cấu trúc ba phần của JWT (Header, Payload, Signature) và quy trình hoạt động của nó từ khi người dùng đăng nhập cho đến khi token được sử dụng để truy cập tài nguyên, cùng với các lưu ý quan trọng về bảo mật Payload và khả năng thu hồi token.
*   **Thiết lập môi trường xác thực JWT:**
    *   Cài đặt các gói NuGet cần thiết (`Microsoft.AspNetCore.Authentication.JwtBearer`, `Microsoft.AspNetCore.Identity.EntityFrameworkCore`, v.v.).
    *   Cấu hình thông tin JWT (Khóa bí mật, Nhà phát hành, Đối tượng) trong `appsettings.json`, kèm theo cảnh báo về bảo mật khóa trong môi trường sản phẩm.
    *   Đăng ký dịch vụ xác thực (`AddAuthentication`, `AddJwtBearer`) và cấu hình chi tiết `TokenValidationParameters` trong `Program.cs`.
*   **Thiết lập ASP.NET Core Identity:**
    *   Tạo chuỗi kết nối riêng cho cơ sở dữ liệu Identity (`NZWalksAuthConnectionString`) để tách biệt dữ liệu.
    *   Định nghĩa `NZWalksAuthDbContext` kế thừa từ `IdentityDbContext`, giải thích cách xử lý vấn đề đa `DbContext` và vai trò của `IdentityDbContext`.
    *   Gieo dữ liệu vai trò (`Reader`, `Writer`) vào cơ sở dữ liệu bằng phương thức `OnModelCreating`, kèm theo giải thích về `ConcurrencyStamp` và `NormalizedName`.
    *   Chạy Entity Framework Core Migrations để tạo cơ sở dữ liệu Identity và các bảng liên quan.
    *   Đăng ký các dịch vụ Identity cốt lõi (`AddIdentityCore`, `AddRoles`, `AddEntityFrameworkStores`) và cấu hình các tùy chọn mật khẩu trong `Program.cs`.
*   **Thêm Middleware Xác thực và Ủy quyền:** Đặt `app.UseAuthentication()` trước `app.UseAuthorization()` trong pipeline yêu cầu HTTP, nhấn mạnh tầm quan trọng của thứ tự này.
*   **Bảo vệ API và kiểm tra phản hồi:** Áp dụng thuộc tính `[Authorize]` lên controller và quan sát phản hồi `401 Unauthorized` khi truy cập không có token hợp lệ, xác nhận rằng hệ thống bảo mật đã được kích hoạt.
*   **Áp dụng Vibe Coding với Antigravity IDE:** Đã tích hợp các điểm hướng dẫn sử dụng Antigravity để tối ưu hóa quy trình cấu hình, thực thi lệnh và kiểm tra, giúp học viên làm quen với tư duy lập trình bằng AI.

Với những bước này, hệ thống của chúng ta đã sẵn sàng để quản lý người dùng và vai trò, cũng như xác thực các yêu cầu đến bằng JWT. Trong chương tiếp theo, chúng ta sẽ xây dựng các điểm cuối API cụ thể để người dùng có thể đăng ký và đăng nhập, từ đó nhận được JWT hợp lệ để truy cập các tài nguyên được bảo vệ.

<!-- REVIEWED_BY_AGENT -->
