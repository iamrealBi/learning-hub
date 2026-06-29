# Phần 17: Xây Dựng RESTful Web API với ASP.NET Core: Khởi Tạo, Quản Lý Mã Nguồn & Cấu Trúc Nền Tảng

Trong chương này, chúng ta sẽ bắt tay vào xây dựng một dự án Web API thực tế sử dụng ASP.NET Core và Entity Framework Core. Mục tiêu chính là trang bị cho bạn những kỹ năng cơ bản nhưng thiết yếu để khởi tạo, phát triển và quản lý mã nguồn của một ứng dụng API hiện đại. Chúng ta sẽ bắt đầu từ việc thiết lập môi trường kiểm soát phiên bản với Git và GitHub, sau đó tạo một dự án ASP.NET Core Web API mới, hiểu cấu trúc cơ bản của nó, các nguyên tắc của kiến trúc RESTful và cách kiểm thử API của chúng ta bằng Swagger UI. Qua đó, bạn sẽ nắm vững quy trình làm việc từ khâu khởi tạo dự án đến khi có thể chạy và tương tác với API của mình, đồng thời làm quen với tư duy Vibe Coding cùng sự hỗ trợ của một hệ thống Agentic AI như Antigravity IDE.

## 1. Nền Tảng Quản Lý Mã Nguồn: Git và GitHub

Khi phát triển phần mềm, việc quản lý mã nguồn là một yếu tố then chốt, đặc biệt trong các dự án lớn hoặc khi làm việc nhóm. Hệ thống kiểm soát phiên bản (Version Control System - VCS) giúp chúng ta theo dõi mọi thay đổi, cộng tác hiệu quả với đồng đội và dễ dàng quay lại các phiên bản trước đó nếu cần. Git là một VCS phân tán phổ biến nhất hiện nay, và GitHub là một nền tảng lưu trữ mã nguồn dựa trên Git, cung cấp các công cụ mạnh mẽ cho việc cộng tác và quản lý dự án.

### 1.1. Tầm Quan Trọng của Hệ Thống Kiểm Soát Phiên Bản (VCS)

VCS như Git không chỉ là công cụ sao lưu mà còn là nền tảng cho quy trình phát triển chuyên nghiệp.

*   **Cộng tác hiệu quả:** Nhiều nhà phát triển có thể làm việc đồng thời trên cùng một dự án mà không gây xung đột mã nguồn. Git quản lý việc hợp nhất các thay đổi (merging) một cách thông minh.
*   **Lịch sử thay đổi minh bạch:** Mọi thay đổi, ai đã thay đổi gì, khi nào, và tại sao đều được ghi lại chi tiết. Điều này cực kỳ hữu ích cho việc kiểm tra, gỡ lỗi và hiểu rõ sự phát triển của dự án.
*   **Quản lý phiên bản linh hoạt:** Dễ dàng tạo các nhánh (branch) để phát triển tính năng mới, thử nghiệm ý tưởng mà không ảnh hưởng đến mã nguồn chính (main/master). Có thể chuyển đổi giữa các phiên bản và hoàn tác các thay đổi một cách nhanh chóng.
*   **Sao lưu an toàn:** Mã nguồn được lưu trữ an toàn trên máy chủ từ xa (như GitHub), giảm thiểu rủi ro mất mát dữ liệu cục bộ do hỏng ổ đĩa hoặc các sự cố khác.

> [!TIP]
> **Vibe Coding và Antigravity IDE với Git:**
> Trong tư duy Vibe Coding, việc hiểu "linh hồn" của mã nguồn không chỉ là đọc từng dòng code mà còn là nắm bắt lịch sử phát triển của nó. Một Agentic AI như Antigravity IDE có thể hỗ trợ bạn trong việc này bằng cách:
> *   **Phân tích lịch sử commit:** Antigravity có thể tóm tắt các thay đổi lớn trong lịch sử dự án, chỉ ra ai đã làm gì và khi nào, giúp bạn nhanh chóng nắm bắt bối cảnh của một phần mã.
> *   **Đề xuất thông điệp commit:** Dựa trên các thay đổi cục bộ, Antigravity có thể tự động phân tích và đề xuất các thông điệp commit rõ ràng, tuân thủ các quy tắc đặt tên.
> *   **Hỗ trợ giải quyết xung đột:** Trong trường hợp xung đột hợp nhất (merge conflict), Antigravity có thể hiển thị các thay đổi một cách trực quan, đề xuất các giải pháp hoặc thậm chí tự động giải quyết các xung đột đơn giản, giúp bạn duy trì "vibe" làm việc trôi chảy.
> *   **Tự động tạo nhánh và quản lý workflow:** Với khả năng *tự chạy script ngầm* và *lập kế hoạch tự động*, Antigravity có thể thực hiện các lệnh Git phức tạp để tạo nhánh tính năng, hợp nhất nhánh, hoặc quản lý các bản phát hành theo quy trình đã định.

### 1.2. Cài Đặt và Cấu Hình Git Cơ Bản

Để sử dụng Git, bạn cần cài đặt nó trên hệ thống của mình và cấu hình thông tin cá nhân.

1.  **Tải Git:** Truy cập trang web chính thức của Git tại [git-scm.com/downloads](https://git-scm.com/downloads). Chọn phiên bản phù hợp với hệ điều hành của bạn (Windows, macOS, Linux) và tải xuống trình cài đặt.
2.  **Cài đặt Git:** Chạy trình cài đặt và làm theo các bước hướng dẫn. Hầu hết các tùy chọn mặc định đều phù hợp cho người mới bắt đầu.
    > [!NOTE]
    > Trong quá trình cài đặt, bạn có thể chọn trình soạn thảo văn bản mặc định (ví dụ: Visual Studio Code, Notepad++). Đối với đa số trường hợp, giữ nguyên các tùy chọn được đề xuất là lựa chọn an toàn để Git hoạt động ổn định.
3.  **Cấu hình Git:** Sau khi cài đặt, bạn cần cấu hình tên người dùng và địa chỉ email của mình. Thông tin này sẽ được gắn với mọi cam kết (commit) của bạn, giúp theo dõi ai đã thực hiện thay đổi nào. Mở cửa sổ Command Prompt (Windows) hoặc Terminal (macOS/Linux) và chạy các lệnh sau:

    ```bash
    git config --global user.name "Tên của bạn"
    git config --global user.email "email_của_bạn@example.com"
    ```
    > [!NOTE]
    > Lệnh `git config --global` sẽ áp dụng cấu hình này cho tất cả các kho lưu trữ Git trên hệ thống của bạn. `user.name` có thể là bất kỳ tên nào bạn muốn hiển thị trong lịch sử Git. `user.email` nên là email bạn đã sử dụng để đăng ký tài khoản GitHub để dễ dàng liên kết các hoạt động.

### 1.3. Tạo Kho Lưu Trữ (Repository) trên GitHub

GitHub là nơi chúng ta sẽ lưu trữ mã nguồn từ xa. Để bắt đầu, bạn cần tạo một kho lưu trữ mới.

1.  **Đăng ký/Đăng nhập GitHub:** Truy cập [github.com](https://github.com) và đăng ký tài khoản nếu bạn chưa có, hoặc đăng nhập.
2.  **Tạo kho lưu trữ mới:** Trên trang chủ GitHub của bạn, tìm nút "New" hoặc "Create repository".
3.  **Cấu hình kho lưu trữ:**
    *   **Repository name:** Đặt tên cho kho lưu trữ của bạn. Ví dụ: `NZWalks`. Tên này nên ngắn gọn, súc tích và phản ánh mục đích của dự án.
    *   **Description (optional):** Mô tả ngắn gọn về dự án.
    *   **Public/Private:** Chọn "Public" để mã nguồn hiển thị công khai hoặc "Private" nếu bạn muốn giữ riêng tư.
    *   **Add a README file:** Tùy chọn này sẽ tạo một tệp `README.md` ban đầu, thường dùng để mô tả dự án, cách cài đặt và sử dụng.
    *   **Add .gitignore:** Rất quan trọng! Chọn mẫu `.gitignore` phù hợp với công nghệ bạn đang sử dụng. Đối với dự án ASP.NET Core trong Visual Studio, hãy tìm và chọn `Visual Studio`. Tệp `.gitignore` sẽ chỉ định những tệp và thư mục mà Git nên bỏ qua (ví dụ: tệp tạm, thư mục `bin`, `obj` chứa các tệp biên dịch, các tệp cấu hình chứa thông tin nhạy cảm). Việc này giúp kho lưu trữ gọn gàng và tránh đưa các tệp không cần thiết lên GitHub.
    *   **Choose a license (optional):** Chọn giấy phép mã nguồn mở (ví dụ: MIT, Apache 2.0) nếu bạn muốn xác định cách người khác có thể sử dụng mã nguồn của bạn.
4.  **Click "Create repository":** Kho lưu trữ của bạn sẽ được tạo.

> [!TIP]
> **Antigravity IDE và `.gitignore`:**
> Một Agentic AI như Antigravity IDE có thể giúp bạn không chỉ chọn đúng mẫu `.gitignore` mà còn phân tích cấu trúc dự án của bạn (bằng cách *đọc ghi file*) và tự động thêm các mục cần bỏ qua tùy chỉnh, đảm bảo rằng chỉ những tệp mã nguồn cần thiết mới được theo dõi bởi Git. Điều này giúp duy trì một kho lưu trữ sạch sẽ và hiệu quả.

### 1.4. Sao Chép Kho Lưu Trữ về Máy Cục Bộ (Git Clone)

Sau khi tạo kho lưu trữ trên GitHub, bạn cần sao chép nó về máy tính cục bộ để bắt đầu làm việc.

1.  **Lấy URL của kho lưu trữ:** Trên trang GitHub của kho lưu trữ bạn vừa tạo, tìm nút "Code" màu xanh lá cây. Nhấp vào đó và sao chép URL HTTPS.
2.  **Mở Command Prompt/Terminal:** Điều hướng đến thư mục mà bạn muốn lưu trữ dự án của mình. Ví dụ, nếu bạn có một thư mục `Code` trên ổ đĩa `D:`, bạn có thể dùng:
    ```bash
    D:
    cd Code
    ```
3.  **Sao chép kho lưu trữ:** Sử dụng lệnh `git clone` và dán URL bạn vừa sao chép:
    ```bash
    git clone https://github.com/your-username/NZWalks.git
    ```
    Lệnh này sẽ tạo một thư mục mới có tên `NZWalks` (hoặc tên kho lưu trữ của bạn) và tải về các tệp ban đầu (như `.gitignore`, `README.md`) vào đó.
    > [!NOTE]
    > Khi bạn `git clone`, Git không chỉ tải về các tệp mà còn thiết lập một liên kết giữa kho lưu trữ cục bộ của bạn và kho lưu trữ từ xa trên GitHub. Điều này cho phép bạn dễ dàng đồng bộ hóa các thay đổi sau này.

## 2. Khởi Tạo Dự Án ASP.NET Core Web API

Bây giờ chúng ta đã có một kho lưu trữ Git cục bộ, hãy tạo dự án ASP.NET Core Web API của chúng ta bên trong thư mục này.

### 2.1. Sử Dụng Visual Studio để Tạo Dự Án Mới

Visual Studio cung cấp một giao diện trực quan và mạnh mẽ để tạo và quản lý các dự án .NET.

1.  **Mở Visual Studio:** Khởi động Visual Studio (phiên bản 2022 trở lên được khuyến nghị).
2.  **Tạo dự án mới:** Chọn "Create a new project".
3.  **Chọn mẫu dự án:** Trong cửa sổ "Create a new project", tìm kiếm và chọn mẫu "ASP.NET Core Web API". Đảm bảo bạn đã chọn C# làm ngôn ngữ và "Web" làm loại dự án. Nhấp vào "Next".
4.  **Cấu hình dự án:**
    *   **Project name:** Đặt tên cho dự án API của bạn. Ví dụ: `NZWalks.API`.
    *   **Location:** Điều hướng đến thư mục mà bạn đã sao chép kho lưu trữ Git cục bộ (`D:\Code\NZWalks`).
    *   **Solution name:** Đặt tên cho Solution của bạn. Ví dụ: `NZWalks`.
    *   **Quan trọng:** Đảm bảo tùy chọn "Place solution and project in the same directory" **không được chọn**. Điều này tạo ra một cấu trúc thư mục tốt hơn, nơi Solution (`.sln`) nằm ở thư mục gốc của kho lưu trữ, và các dự án con (như `NZWalks.API`) nằm trong các thư mục riêng biệt bên dưới. Cấu trúc này giúp quản lý nhiều dự án trong cùng một Solution dễ dàng hơn.
    *   Nhấp vào "Next".
5.  **Cấu hình bổ sung:**
    *   **Framework:** Chọn phiên bản .NET mà bạn muốn sử dụng (ví dụ: .NET 8.0 là phiên bản LTS gần nhất).
    *   **Authentication type:** Giữ mặc định "None" cho dự án này.
    *   **Configure for HTTPS:** Giữ tùy chọn này được chọn để đảm bảo API của bạn sử dụng giao thức bảo mật.
    *   **Enable Docker:** Bỏ chọn nếu bạn không muốn sử dụng Docker ngay lập tức.
    *   **Enable OpenAPI support:** **Rất quan trọng!** Đảm bảo tùy chọn này được chọn. Đây là cách Visual Studio tích hợp Swagger UI vào dự án của bạn, cung cấp tài liệu API tự động và giao diện kiểm thử.
    *   **Use controllers (uncheck to use minimal APIs):** Đảm bảo tùy chọn này được chọn để sử dụng kiến trúc Controller truyền thống, phù hợp với mục tiêu khóa học này. Minimal APIs là một phong cách khác, gọn nhẹ hơn, phù hợp cho các API đơn giản.
    *   Nhấp vào "Create".

Visual Studio sẽ tạo một Solution `NZWalks` và một dự án `NZWalks.API` bên trong thư mục `NZWalks` của bạn.

> [!TIP]
> **Vibe Coding và Antigravity IDE trong Khởi tạo Dự án:**
> Với Antigravity IDE, quá trình tạo dự án có thể được tối ưu hóa đáng kể. Thay vì click chuột qua các bước, bạn có thể:
> *   **Mô tả dự án bằng ngôn ngữ tự nhiên:** "Hãy tạo một ASP.NET Core Web API với .NET 8, hỗ trợ Swagger, không dùng Docker, và đặt trong thư mục `D:\Code\NZWalks`."
> *   **Antigravity sẽ lập kế hoạch:** Hệ thống sẽ tự động chuyển đổi yêu cầu của bạn thành các lệnh `dotnet new` phù hợp (ví dụ: `dotnet new webapi -n NZWalks.API -o NZWalks.API --framework net8.0 --no-https --use-controllers --output D:\Code\NZWalks`) và *tự chạy script ngầm*.
> *   **Tùy chỉnh cấu trúc:** Antigravity có thể phân tích các yêu cầu về cấu trúc thư mục (ví dụ: "Solution ở ngoài, Project ở trong") và điều chỉnh các lệnh hoặc thực hiện các thao tác file/folder cần thiết (*đọc ghi file*) để đạt được cấu trúc mong muốn.

### 2.2. Cấu Trúc Dự Án ASP.NET Core Web API Ban Đầu

Sau khi tạo, dự án của bạn sẽ có cấu trúc cơ bản như sau:

```
NZWalks/
├── NZWalks.API/
│   ├── Controllers/
│   │   └── WeatherForecastController.cs
│   ├── Properties/
│   │   └── launchSettings.json
│   ├── appsettings.json
│   ├── appsettings.Development.json
│   ├── Program.cs
│   ├── NZWalks.API.csproj
│   └── ... (các tệp và thư mục khác)
├── NZWalks.sln
└── .gitignore
```

Đây là cấu trúc tiêu chuẩn cho một dự án ASP.NET Core Web API, với `NZWalks.sln` là tệp Solution quản lý một hoặc nhiều dự án, và `NZWalks.API` là thư mục chứa dự án API thực tế.

## 3. Đẩy Mã Nguồn Mới Lên GitHub

Sau khi tạo dự án API mới, mã nguồn này hiện chỉ tồn tại trên máy tính cục bộ của bạn. Để lưu trữ nó trên GitHub và chia sẻ với những người khác, chúng ta cần thực hiện các bước sau:

### 3.1. Thêm và Cam Kết Các Thay Đổi (Add & Commit)

1.  **Mở Command Prompt/Terminal:** Điều hướng vào thư mục gốc của kho lưu trữ Git của bạn (thư mục `NZWalks`), nơi chứa tệp `.git` ẩn.

    ```bash
    cd D:\Code\NZWalks
    ```
    > [!NOTE]
    > Đảm bảo bạn đang ở thư mục gốc của kho lưu trữ, không phải thư mục dự án con (`NZWalks.API`), để Git theo dõi toàn bộ Solution.

2.  **Thêm các tệp vào khu vực dàn dựng (Staging Area):** Lệnh `git add .` sẽ thêm tất cả các tệp mới và đã thay đổi (trừ những tệp bị bỏ qua bởi `.gitignore`) vào khu vực dàn dựng. Khu vực dàn dựng là một bước trung gian, cho phép bạn chọn lọc những thay đổi nào sẽ được đưa vào cam kết tiếp theo.

    ```bash
    git add .
    ```

3.  **Tạo cam kết (Commit):** Lệnh `git commit -m "Thông điệp cam kết"` sẽ tạo một bản ghi lịch sử về những thay đổi bạn vừa thêm vào khu vực dàn dựng. Thông điệp cam kết nên mô tả ngắn gọn và rõ ràng về những gì bạn đã thay đổi, tuân thủ các quy tắc đặt tên commit tốt để dễ dàng theo dõi lịch sử dự án sau này.

    ```bash
    git commit -m "Initial setup: Created new ASP.NET Core Web API project"
    ```
    > [!TIP]
    > Các cam kết lúc này chỉ tồn tại trên máy cục bộ của bạn. Chúng chưa được gửi lên GitHub.

### 3.2. Đẩy Mã Nguồn Lên GitHub (Git Push)

1.  **Đẩy mã nguồn lên GitHub (Push):** Lệnh `git push` sẽ gửi tất cả các cam kết cục bộ của bạn lên kho lưu trữ từ xa trên GitHub.

    ```bash
    git push
    ```
    Nếu đây là lần đầu tiên bạn đẩy lên kho lưu trữ mới, Git có thể yêu cầu bạn xác thực bằng tài khoản GitHub của mình thông qua trình duyệt web hoặc cung cấp token truy cập cá nhân. Sau khi xác thực thành công, mã nguồn của bạn sẽ được tải lên GitHub.

2.  **Xác nhận trên GitHub:** Truy cập lại kho lưu trữ `NZWalks` trên GitHub của bạn. Bạn sẽ thấy tất cả các tệp và thư mục của dự án API mới đã xuất hiện.

> [!TIP]
> **Antigravity IDE và Quy trình Git:**
> Antigravity IDE có thể tích hợp sâu vào quy trình Git của bạn:
> *   **Tự động hóa `add` và `commit`:** Antigravity có thể theo dõi các thay đổi trong workspace của bạn, tự động `git add` các tệp liên quan và đề xuất thông điệp `git commit` thông minh.
> *   **Quản lý `push`/`pull`:** Antigravity có thể *tự chạy script ngầm* để thực hiện các thao tác `git push` hoặc `git pull` khi cần, đồng thời thông báo cho bạn về trạng thái đồng bộ hóa với kho lưu trữ từ xa.
> *   **Hiển thị trạng thái Git trực quan:** Thay vì phải chạy nhiều lệnh Git, Antigravity có thể hiển thị trạng thái hiện tại của kho lưu trữ (các tệp đã thay đổi, số lượng commit chưa đẩy) một cách trực quan, giúp bạn luôn nắm bắt được "vibe" của mã nguồn.

## 4. Khám Phá Cấu Trúc Cốt Lõi của ASP.NET Core Web API

Sau khi dự án được tạo và đẩy lên GitHub, hãy cùng khám phá các thành phần cốt lõi của một ứng dụng ASP.NET Core Web API.

### 4.1. `Program.cs`: Điểm Khởi Đầu và Cấu Hình Ứng Dụng

Tệp `Program.cs` là điểm vào (entry point) của mọi ứng dụng ASP.NET Core. Khi bạn chạy ứng dụng, mã trong tệp này sẽ được thực thi đầu tiên, chịu trách nhiệm cấu hình các dịch vụ (services) và chuỗi các middleware xử lý yêu cầu HTTP (HTTP request pipeline).

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.OpenApi.Models; // Thêm namespace này cho Swagger

var builder = WebApplication.CreateBuilder(args); // 1. Khởi tạo WebApplicationBuilder

// 2. Đăng ký các dịch vụ (services) vào hệ thống Dependency Injection (DI)
builder.Services.AddControllers(); // Đăng ký dịch vụ cho Controllers
builder.Services.AddEndpointsApiExplorer(); // Cần thiết cho Swagger
builder.Services.AddSwaggerGen(options => // Cấu hình Swagger
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "NZWalks API", Version = "v1" });
});

var app = builder.Build(); // 3. Xây dựng đối tượng WebApplication

// 4. Cấu hình chuỗi các middleware xử lý yêu cầu HTTP
if (app.Environment.IsDevelopment()) // Chỉ sử dụng Swagger trong môi trường phát triển
{
    app.UseSwagger(); // Thêm middleware cho Swagger JSON endpoint
    app.UseSwaggerUI(options => // Thêm middleware cho Swagger UI
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json", "NZWalks API v1");
    });
}

app.UseHttpsRedirection(); // Chuyển hướng HTTP sang HTTPS
app.UseAuthorization(); // Xử lý xác thực/phân quyền

app.MapControllers(); // Ánh xạ các HTTP request tới các Controller

app.Run(); // 5. Chạy ứng dụng và lắng nghe các yêu cầu
```

#### 4.1.1. Tầng Dịch Vụ (Services Layer) và Dependency Injection (DI)

ASP.NET Core được xây dựng dựa trên nguyên tắc Dependency Injection (DI), một mẫu thiết kế giúp các thành phần của ứng dụng ít phụ thuộc vào nhau hơn (loose coupling) và dễ kiểm thử hơn.

> [!TIP]
> **Dependency Injection (DI) là gì?**
> Thay vì một đối tượng tự tạo ra hoặc tìm kiếm các đối tượng mà nó cần (các dependency), thì các dependency này sẽ được "tiêm" vào nó từ bên ngoài. Trong ASP.NET Core, điều này thường được thực hiện thông qua constructor của các lớp. Bộ chứa DI (DI container) chịu trách nhiệm tạo và cung cấp các dependency này.
>
> **Lợi ích của DI:**
> *   **Giảm sự phụ thuộc (Loose Coupling):** Các lớp không cần biết cách tạo ra các dependency của chúng, chỉ cần biết chúng cần gì.
> *   **Dễ kiểm thử (Testability):** Có thể dễ dàng thay thế các dependency bằng các đối tượng giả (mock objects) trong quá trình kiểm thử đơn vị (unit testing), giúp cô lập logic nghiệp vụ.
> *   **Khả năng bảo trì và mở rộng:** Mã nguồn dễ thay đổi và mở rộng hơn vì các thành phần độc lập hơn.
> *   **Tái sử dụng mã:** Các dịch vụ có thể được tái sử dụng ở nhiều nơi khác nhau mà không cần viết lại logic khởi tạo.

Trong `Program.cs`, khu vực `builder.Services` là nơi bạn đăng ký tất cả các dịch vụ mà ứng dụng của bạn sẽ sử dụng vào bộ chứa DI.

*   `builder.Services.AddControllers();`: Đăng ký các dịch vụ cần thiết để các Controller hoạt động, bao gồm cả việc đăng ký các Controller vào bộ chứa DI.
*   `builder.Services.AddSwaggerGen();`: Cấu hình dịch vụ để tạo tài liệu API cho Swagger.

> [!NOTE]
> **Vòng đời của dịch vụ (Service Lifetimes):**
> Khi đăng ký dịch vụ vào bộ chứa DI, bạn cần chỉ định vòng đời của chúng:
> *   **Singleton:** Một thể hiện duy nhất của dịch vụ được tạo ra và sử dụng lại trong suốt vòng đời của ứng dụng. Phù hợp cho các dịch vụ không trạng thái (stateless) hoặc tài nguyên dùng chung.
> *   **Scoped:** Một thể hiện của dịch vụ được tạo ra một lần cho mỗi yêu cầu HTTP và được sử dụng trong suốt yêu cầu đó. Phù hợp cho các dịch vụ cần duy trì trạng thái trong phạm vi một yêu cầu (ví dụ: DbContext của Entity Framework Core).
> *   **Transient:** Một thể hiện mới của dịch vụ được tạo ra mỗi khi nó được yêu cầu. Phù hợp cho các dịch vụ nhẹ, không trạng thái hoặc các đối tượng cần được khởi tạo mới mỗi lần sử dụng.
>
> Ví dụ: `builder.Services.AddScoped<IMyService, MyService>();`

> [!TIP]
> **Antigravity IDE và Dependency Injection:**
> Antigravity IDE có thể trở thành một trợ thủ đắc lực trong việc quản lý DI:
> *   **Phân tích cây phụ thuộc:** Antigravity có thể hiển thị biểu đồ các phụ thuộc giữa các lớp, giúp bạn dễ dàng nắm bắt "vibe" của kiến trúc ứng dụng.
> *   **Đề xuất đăng ký dịch vụ:** Khi bạn tạo một interface và một implementation mới (ví dụ: `IRepository` và `Repository`), Antigravity có thể tự động đề xuất dòng `builder.Services.AddScoped<IRepository, Repository>();` trong `Program.cs` và thậm chí *tự động ghi file* này.
> *   **Kiểm tra lỗi cấu hình DI:** Hệ thống có thể phát hiện các lỗi phổ biến như dịch vụ chưa được đăng ký, hoặc vòng đời dịch vụ không phù hợp có thể dẫn đến lỗi runtime hoặc memory leak.

#### 4.1.2. Chuỗi Middleware và Pipeline Xử Lý Yêu Cầu HTTP

Sau khi các dịch vụ được đăng ký, `app = builder.Build();` sẽ xây dựng đối tượng `WebApplication`. Tiếp theo, khu vực `app.Use...()` sẽ cấu hình chuỗi các middleware. Middleware là các thành phần phần mềm được sắp xếp trong một pipeline để xử lý các yêu cầu HTTP. Mỗi middleware có thể thực hiện một tác vụ cụ thể (như logging, xác thực, định tuyến) và sau đó chuyển yêu cầu cho middleware tiếp theo trong chuỗi hoặc tạo phản hồi.

*   `if (app.Environment.IsDevelopment())`: Chỉ định rằng các middleware bên trong khối này chỉ được áp dụng khi ứng dụng chạy trong môi trường phát triển (Development).
*   `app.UseSwagger()` và `app.UseSwaggerUI()`: Thêm middleware để hiển thị tài liệu API và giao diện người dùng tương tác của Swagger.
*   `app.UseHttpsRedirection()`: Đảm bảo rằng mọi yêu cầu HTTP không bảo mật sẽ được chuyển hướng sang HTTPS.
*   `app.UseAuthorization()`: Middleware xử lý logic xác thực và phân quyền, xác định xem người dùng có quyền truy cập vào tài nguyên yêu cầu hay không.
*   `app.MapControllers()`: Đây là middleware quan trọng định tuyến các yêu cầu HTTP đến các Controller và Action Method phù hợp dựa trên đường dẫn và động từ HTTP.
*   `app.Run()`: Bắt đầu ứng dụng, lắng nghe các yêu cầu HTTP đến và xử lý chúng thông qua pipeline middleware đã cấu hình.

### 4.2. `appsettings.json`: Quản Lý Cấu Hình Ứng Dụng

Tệp `appsettings.json` (và các biến thể như `appsettings.Development.json`, `appsettings.Production.json`) được sử dụng để lưu trữ các cài đặt cấu hình cho ứng dụng của bạn, chẳng hạn như chuỗi kết nối cơ sở dữ liệu, các khóa API, hoặc các biến môi trường.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  // Đây là nơi bạn sẽ thêm chuỗi kết nối cơ sở dữ liệu của mình
  "ConnectionStrings": {
    "NZWalksConnectionString": "Server=(localdb)\\mssqllocaldb;Database=NZWalksDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

> [!NOTE]
> *   `appsettings.Development.json` sẽ ghi đè các cài đặt trong `appsettings.json` khi ứng dụng chạy trong môi trường phát triển. Điều này hữu ích để có các cài đặt khác nhau cho môi trường phát triển, thử nghiệm và sản xuất mà không cần thay đổi mã nguồn.
> *   `ConnectionStrings` là một phần phổ biến để lưu trữ chuỗi kết nối đến cơ sở dữ liệu. Trong ví dụ trên, `NZWalksConnectionString` là chuỗi kết nối đến một cơ sở dữ liệu SQL Server LocalDB cục bộ.

> [!TIP]
> **Antigravity IDE và Cấu hình Ứng Dụng:**
> Antigravity IDE có thể giúp bạn quản lý cấu hình một cách thông minh:
> *   **Tạo cấu hình môi trường:** Dựa trên môi trường triển khai (ví dụ: Development, Staging, Production), Antigravity có thể *đọc ghi file* để tạo hoặc cập nhật các tệp `appsettings.{Environment}.json` với các giá trị phù hợp.
> *   **Đề xuất cấu hình an toàn:** Antigravity có thể cảnh báo bạn về việc lưu trữ các thông tin nhạy cảm (như khóa API, mật khẩu) trực tiếp trong `appsettings.json` và đề xuất sử dụng các phương pháp an toàn hơn như User Secrets (trong phát triển) hoặc Azure Key Vault (trong sản xuất).
> *   **Xác thực cấu hình:** AI có thể phân tích các định dạng cấu hình và báo lỗi nếu có cú pháp JSON sai hoặc thiếu các trường cần thiết.

### 4.3. Thư Mục `Controllers`: Trái Tim của API

Thư mục `Controllers` chứa các lớp Controller, là trái tim của ứng dụng Web API. Mỗi Controller chịu trách nhiệm xử lý các yêu cầu HTTP đến một tài nguyên cụ thể (ví dụ: `WeatherForecastController` xử lý các yêu cầu liên quan đến dự báo thời tiết).

```csharp
using Microsoft.AspNetCore.Mvc; // Chứa các thuộc tính và lớp cơ sở cho Controller
using Microsoft.Extensions.Logging; // Để ghi log

namespace NZWalks.API.Controllers
{
    // [ApiController] cung cấp các tính năng tiện ích cho API Controller
    [ApiController]
    // [Route] định nghĩa đường dẫn cơ sở cho Controller này
    [Route("[controller]")] // Ví dụ: /WeatherForecast
    // Hoặc [Route("api/[controller]")] để có tiền tố /api/
    public class WeatherForecastController : ControllerBase // Kế thừa từ ControllerBase
    {
        private static readonly string[] Summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        private readonly ILogger<WeatherForecastController> _logger;

        // Constructor để tiêm ILogger thông qua Dependency Injection
        public WeatherForecastController(ILogger<WeatherForecastController> logger)
        {
            _logger = logger;
        }

        // Đây là một Action Method xử lý yêu cầu HTTP GET
        [HttpGet] // Thuộc tính chỉ định phương thức HTTP GET
        public IEnumerable<WeatherForecast> Get() // Phương thức trả về danh sách WeatherForecast
        {
            _logger.LogInformation("Get WeatherForecast API was called."); // Ghi log
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
                TemperatureC = Random.Shared.Next(-20, 55),
                Summary = Summaries[Random.Shared.Next(Summaries.Length)]
            })
            .ToArray();
        }
    }
}
```

> [!NOTE]
> *   **`ControllerBase`:** Các API Controller kế thừa từ lớp `ControllerBase` (thay vì `Controller` vì chúng không cần hỗ trợ View). Lớp này cung cấp các phương thức tiện ích cho việc xử lý yêu cầu HTTP, như `Ok()`, `NotFound()`, `BadRequest()`.
> *   **`[ApiController]`:** Thuộc tính này là một cải tiến quan trọng cho các API Controller. Nó thêm các hành vi hữu ích như:
    *   **Tự động kiểm tra lỗi mô hình (Automatic Model Validation):** Nếu dữ liệu đầu vào không hợp lệ (ví dụ: trường bắt buộc bị thiếu), ASP.NET Core sẽ tự động trả về phản hồi HTTP 400 Bad Request mà không cần bạn phải viết code kiểm tra thủ công.
    *   **Binding source inference:** Tự động suy luận nơi lấy dữ liệu cho các tham số (từ route, query string, hoặc body).
    *   **Trả về các phản hồi HTTP chuẩn:** Khuyến khích sử dụng các `IActionResult` để trả về các mã trạng thái HTTP thích hợp.
*   **`[Route("[controller]")]`:** Định nghĩa mẫu đường dẫn cho Controller. `[controller]` là một placeholder sẽ được thay thế bằng tên của Controller (ví dụ: `WeatherForecast` cho `WeatherForecastController`). Bạn cũng có thể dùng `[Route("api/[controller]")]` để có tiền tố `api/` cho tất cả các API.
*   **`[HttpGet]`:** Là một HTTP Verb. Nó chỉ ra rằng phương thức `Get()` này sẽ xử lý các yêu cầu HTTP GET đến đường dẫn được định nghĩa bởi `[Route]`. Các thuộc tính tương tự tồn tại cho `[HttpPost]`, `[HttpPut]`, `[HttpPatch]`, `[HttpDelete]`.
*   **Action Method:** Các phương thức công khai trong Controller được đánh dấu bằng các thuộc tính HTTP Verb (như `[HttpGet]`) được gọi là Action Method. Chúng là nơi chứa logic xử lý yêu cầu HTTP và trả về phản hồi.

#### 4.3.1. Giới Thiệu Repository Pattern và Entity Framework Core

Mặc dù chưa có trong dự án khởi tạo, khi xây dựng một Web API tương tác với cơ sở dữ liệu, chúng ta thường áp dụng các mẫu thiết kế và công nghệ để tổ chức mã nguồn một cách hiệu quả và bền vững.

> [!TIP]
> **Repository Pattern:**
> Mẫu Repository tạo ra một lớp trừu tượng giữa tầng nghiệp vụ (Business Logic Layer) và tầng truy cập dữ liệu (Data Access Layer). Thay vì Controller trực tiếp tương tác với Entity Framework Core (hoặc bất kỳ ORM nào), nó sẽ tương tác với một Interface của Repository.
>
> **Lợi ích của Repository Pattern:**
> *   **Tách biệt mối quan tâm (Separation of Concerns):** Controller không cần biết chi tiết về cách dữ liệu được lưu trữ hay truy vấn (ví dụ: SQL, NoSQL, in-memory database). Nó chỉ cần biết "tôi cần một danh sách các chuyến đi" và Repository sẽ cung cấp nó.
> *   **Dễ kiểm thử (Testability):** Có thể dễ dàng tạo các Repository giả (mock repository) cho kiểm thử đơn vị của Controller mà không cần một cơ sở dữ liệu thực.
> *   **Khả năng thay đổi (Flexibility):** Có thể thay đổi công nghệ cơ sở dữ liệu hoặc ORM mà không cần sửa đổi Controller hoặc logic nghiệp vụ cấp cao hơn. Chỉ cần thay đổi implementation của Repository.
> *   **Tái sử dụng code:** Logic truy cập dữ liệu được tập trung vào một nơi, dễ dàng tái sử dụng.
>
> **Ví dụ cấu trúc:**
> *   `Interfaces/IRegionRepository.cs` (Chỉ định các phương thức CRUD)
> *   `Repositories/SQLRegionRepository.cs` (Triển khai các phương thức đó bằng Entity Framework Core)
> *   `Controllers/RegionsController.cs` (Tiêm `IRegionRepository` vào constructor và sử dụng các phương thức của nó)

**Entity Framework Core (EF Core):**
EF Core là một công cụ Ánh xạ Quan hệ Đối tượng (Object-Relational Mapper - ORM) được phát triển bởi Microsoft. Nó cho phép các nhà phát triển .NET làm việc với cơ sở dữ liệu bằng cách sử dụng các đối tượng C# (POCO - Plain Old CLR Objects) thay vì viết các câu lệnh SQL truyền thống.

> [!NOTE]
> Trong các phần tiếp theo, chúng ta sẽ tích hợp EF Core và triển khai Repository Pattern để quản lý dữ liệu cho dự án NZWalks, tạo ra một kiến trúc sạch sẽ và dễ bảo trì.

> [!TIP]
> **Antigravity IDE với Repository Pattern và EF Core:**
> Đây là lĩnh vực mà Agentic AI như Antigravity IDE có thể phát huy tối đa sức mạnh của mình trong Vibe Coding:
> *   **Tạo mã boilerplate:** Từ một định nghĩa `Model` (ví dụ: `Region.cs`), Antigravity có thể tự động tạo ra `IRegionRepository.cs`, `SQLRegionRepository.cs` với các phương thức CRUD cơ bản, và thậm chí cả `DbContext` để tích hợp với EF Core. Điều này giúp giảm đáng kể công sức viết mã lặp lại.
> *   **Tích hợp DI:** Khi các Repository được tạo, Antigravity có thể tự động *ghi file* `Program.cs` để đăng ký chúng vào bộ chứa DI với vòng đời `Scoped` phù hợp.
> *   **Tối ưu hóa truy vấn:** Antigravity có thể phân tích các truy vấn LINQ của bạn và đề xuất các cách tối ưu hóa hiệu suất, hoặc cảnh báo về các vấn đề N+1 query.
> *   **Quản lý Migrations:** AI có thể hỗ trợ tạo và áp dụng các EF Core Migration, giúp bạn quản lý sự thay đổi của schema cơ sở dữ liệu một cách liền mạch.

## 5. Thiết Kế API Theo Kiến Trúc RESTful và HTTP Verbs

REST (Representational State Transfer) là một phong cách kiến trúc để thiết kế các hệ thống mạng phân tán, đặc biệt là các Web Service. Một API tuân thủ các nguyên tắc của REST được gọi là RESTful API.

### 5.1. Nguyên Lý Kiến Trúc REST (Representational State Transfer)

REST không phải là một giao thức mà là một tập hợp các nguyên tắc hướng dẫn cách thiết kế một API để nó có thể mở rộng, linh hoạt và dễ sử dụng.

> [!NOTE]
> **Nguyên tắc cốt lõi của REST:**
> *   **Client-Server:** Tách biệt giao diện người dùng (client) khỏi lưu trữ dữ liệu (server). Điều này cho phép client và server phát triển độc lập.
> *   **Stateless (Không trạng thái):** Mỗi yêu cầu từ client đến server phải chứa tất cả thông tin cần thiết để server hiểu và xử lý yêu cầu đó. Server không lưu trữ bất kỳ trạng thái nào của client giữa các yêu cầu. Điều này giúp API dễ dàng mở rộng theo chiều ngang.
> *   **Cacheable (Có thể lưu trữ vào bộ đệm):** Phản hồi từ server có thể được đánh dấu là có thể lưu vào bộ đệm để client hoặc các proxy trung gian có thể sử dụng lại, cải thiện hiệu suất và giảm tải cho server.
> *   **Layered System (Hệ thống phân lớp):** Client không nhất thiết phải kết nối trực tiếp với server cuối cùng; có thể có các lớp trung gian (proxy, load balancer, gateway) mà client không biết, nhưng vẫn hoạt động bình thường. Điều này tăng tính linh hoạt và bảo mật.
> *   **Uniform Interface (Giao diện đồng nhất):** Đây là nguyên tắc quan trọng nhất, đảm bảo tính nhất quán và đơn giản cho việc tương tác với API. Nó bao gồm:
    *   **Identification of resources:** Các tài nguyên được xác định bằng URI (Uniform Resource Identifier) duy nhất (ví dụ: `/api/walks`, `/api/walks/123`).
    *   **Manipulation of resources through representations:** Client tương tác với tài nguyên thông qua các biểu diễn (representations) của chúng (ví dụ: JSON, XML). Khi client gửi một biểu diễn, server thay đổi trạng thái của tài nguyên dựa trên biểu diễn đó.
    *   **Self-descriptive messages:** Mỗi thông điệp (yêu cầu và phản hồi) phải chứa đủ thông tin để client hiểu cách xử lý nó, bao gồm các tiêu đề HTTP (Content-Type, Accept) và mã trạng thái HTTP.
    *   **Hypermedia as the Engine of Application State (HATEOAS):** Client tương tác với API hoàn toàn thông qua các siêu liên kết (hyperlinks) được cung cấp trong phản hồi của server. Thay vì client phải "biết" các URL tiếp theo, server sẽ "hướng dẫn" client thông qua các liên kết. (Nguyên tắc này ít khi được triển khai đầy đủ trong các API thực tế nhưng là một phần cốt lõi của REST).

> [!TIP]
> **Antigravity IDE và Thiết kế RESTful API:**
> Trong Vibe Coding, Antigravity IDE có thể giúp bạn "cảm nhận" và thiết kế API theo các nguyên tắc REST:
> *   **Đề xuất URI và động từ HTTP:** Khi bạn mô tả một tài nguyên mới (ví dụ: "quản lý các chuyến đi"), Antigravity có thể đề xuất các URI chuẩn (`/api/walks`, `/api/walks/{id}`) và các động từ HTTP phù hợp cho các thao tác CRUD.
> *   **Tạo phản hồi chuẩn:** Antigravity có thể giúp bạn thiết kế các cấu trúc phản hồi JSON tuân thủ các nguyên tắc tự mô tả, bao gồm cả các liên kết HATEOAS nếu bạn muốn triển khai.
> *   **Kiểm tra tính nhất quán:** AI có thể phân tích toàn bộ API của bạn để đảm bảo tính nhất quán trong việc đặt tên tài nguyên, sử dụng động từ HTTP và cấu trúc phản hồi.

### 5.2. Các Động Từ HTTP (HTTP Verbs) và Ánh Xạ CRUD

RESTful API sử dụng các động từ HTTP (còn gọi là phương thức HTTP) để chỉ ra loại hành động mà client muốn thực hiện trên một tài nguyên. Các động từ này ánh xạ trực tiếp tới các thao tác CRUD (Create, Read, Update, Delete) cơ bản.

*   **GET**
    *   **Mục đích:** Truy xuất (đọc) một hoặc nhiều tài nguyên từ server.
    *   **Đặc điểm:**
        *   **Safe:** Không làm thay đổi trạng thái của server.
        *   **Idempotent:** Thực hiện nhiều lần cùng một yêu cầu GET sẽ luôn cho cùng một kết quả mà không gây ra tác dụng phụ.
        *   **Cacheable:** Phản hồi có thể được lưu vào bộ đệm để cải thiện hiệu suất.
    *   **Ánh xạ CRUD:** **Read**.
    *   **Ví dụ:**
        *   `GET /api/walks`: Lấy tất cả các chuyến đi.
        *   `GET /api/walks/{id}`: Lấy một chuyến đi cụ thể dựa trên ID.
        *   `GET /api/regions?search=north`: Lấy các vùng miền có tên chứa "north".

*   **POST**
    *   **Mục đích:** Tạo một tài nguyên mới trên server.
    *   **Đặc điểm:**
        *   **Không Safe:** Thay đổi trạng thái của server.
        *   **Không Idempotent:** Gửi nhiều lần cùng một yêu cầu POST có thể tạo ra nhiều tài nguyên mới (ví dụ: gửi cùng một form nhiều lần có thể tạo nhiều bản ghi).
        *   **Không Cacheable.**
    *   **Ánh xạ CRUD:** **Create**.
    *   **Ví dụ:**
        *   `POST /api/walks`: Tạo một chuyến đi mới với dữ liệu được gửi trong phần thân yêu cầu (request body).

*   **PUT**
    *   **Mục đích:** Cập nhật hoặc thay thế hoàn toàn một tài nguyên hiện có. Nếu tài nguyên không tồn tại, PUT có thể tạo mới (nhưng thường POST được dùng cho việc tạo).
    *   **Đặc điểm:**
        *   **Không Safe:** Thay đổi trạng thái của server.
        *   **Idempotent:** Gửi nhiều lần cùng một yêu cầu PUT sẽ cho cùng một kết quả cuối cùng (trạng thái của tài nguyên sau khi cập nhật là duy nhất, bất kể số lần gửi yêu cầu).
        *   **Không Cacheable.**
    *   **Ánh xạ CRUD:** **Update** (hoàn toàn).
    *   **Ví dụ:**
        *   `PUT /api/walks/{id}`: Cập nhật toàn bộ thông tin của một chuyến đi với ID đã cho bằng dữ liệu mới trong request body.

*   **PATCH**
    *   **Mục đích:** Cập nhật một phần của tài nguyên hiện có.
    *   **Đặc điểm:**
        *   **Không Safe:** Thay đổi trạng thái của server.
        *   **Không Idempotent:** Gửi nhiều lần có thể không cho cùng một kết quả nếu các thay đổi phụ thuộc vào trạng thái hiện tại của tài nguyên (ví dụ: `PATCH /api/users/123` để tăng số lượt xem lên 1).
        *   **Không Cacheable.**
    *   **Ánh xạ CRUD:** **Partial Update**.
    *   **Ví dụ:**
        *   `PATCH /api/walks/{id}`: Chỉ cập nhật tên hoặc mô tả của một chuyến đi, không cần gửi toàn bộ đối tượng chuyến đi.

*   **DELETE**
    *   **Mục đích:** Xóa một tài nguyên cụ thể.
    *   **Đặc điểm:**
        *   **Không Safe:** Thay đổi trạng thái của server.
        *   **Idempotent:** Xóa một tài nguyên nhiều lần sẽ vẫn đảm bảo tài nguyên đó bị xóa (kết quả cuối cùng là tài nguyên không tồn tại).
        *   **Không Cacheable.**
    *   **Ánh xạ CRUD:** **Delete**.
    *   **Ví dụ:**
        *   `DELETE /api/walks/{id}`: Xóa một chuyến đi cụ thể dựa trên ID.

> [!TIP]
> **JSON (JavaScript Object Notation):**
> JSON là định dạng dữ liệu phổ biến nhất để trao đổi dữ liệu trong các RESTful API. Nó nhẹ, dễ đọc cho cả người và máy, và được hỗ trợ rộng rãi bởi hầu hết các ngôn ngữ lập trình. Trong ASP.NET Core, các đối tượng C# thường được tự động serialize thành JSON khi gửi phản hồi và deserialize từ JSON khi nhận yêu cầu.

## 6. Chạy Ứng Dụng và Kiểm Thử API với Swagger UI

Sau khi hiểu về cấu trúc và nguyên lý, giờ là lúc chạy ứng dụng và kiểm thử API của chúng ta.

### 6.1. Khởi Chạy Dự Án trong Visual Studio

1.  **Chọn cấu hình chạy:** Trong Visual Studio, bạn có thể chọn cấu hình chạy từ thanh công cụ phía trên. Mặc định sẽ là "IIS Express" hoặc tên dự án của bạn (ví dụ: `NZWalks.API`).
2.  **Chạy không gỡ lỗi:** Nhấp vào nút mũi tên màu xanh lá cây hoặc chọn "Debug" > "Start Without Debugging" (Ctrl + F5).
    > [!NOTE]
    > Chạy không gỡ lỗi sẽ khởi động ứng dụng nhanh hơn và mở trình duyệt mặc định của bạn mà không gắn trình gỡ lỗi. Điều này phù hợp cho việc kiểm thử nhanh.

### 6.2. Kiểm Thử API Hiệu Quả với Swagger UI

Khi ứng dụng chạy, trình duyệt sẽ tự động mở và điều hướng đến trang Swagger UI (nếu bạn đã bật "Enable OpenAPI support" khi tạo dự án).

Swagger UI là một công cụ mạnh mẽ để:

*   **Tài liệu hóa API:** Hiển thị tất cả các điểm cuối (endpoints) của API, các động từ HTTP được hỗ trợ, tham số đầu vào, cấu trúc dữ liệu yêu cầu/phản hồi, và các mã trạng thái phản hồi có thể có.
*   **Kiểm thử API:** Cho phép bạn gửi các yêu cầu HTTP trực tiếp từ trình duyệt và xem phản hồi một cách trực quan.

**Cách sử dụng Swagger UI:**

1.  **Xem các điểm cuối:** Trên trang Swagger UI, bạn sẽ thấy danh sách các Controller và các phương thức HTTP (GET, POST, PUT, DELETE) mà chúng cung cấp. Ban đầu, bạn sẽ thấy `WeatherForecast Controller` với một phương thức `GET`.
2.  **Mở rộng một điểm cuối:** Nhấp vào một điểm cuối (ví dụ: `GET /WeatherForecast`) để xem chi tiết: mô tả, các tham số có thể có, và các mã trạng thái phản hồi.
3.  **Thử nghiệm điểm cuối:**
    *   Nhấp vào nút "Try it out".
    *   Điền các tham số nếu có (đối với `WeatherForecast` GET, không có tham số nào).
    *   Nhấp vào nút "Execute".
4.  **Xem phản hồi:** Swagger UI sẽ hiển thị yêu cầu HTTP đã gửi (Request URL, Headers), mã trạng thái HTTP (ví dụ: `200 OK` cho thành công), thời gian phản hồi, và dữ liệu phản hồi (Response body) ở định dạng JSON.

> [!TIP]
> **Antigravity IDE và Swagger UI:**
> Antigravity IDE có thể tương tác trực tiếp với Swagger UI hoặc các tệp OpenAPI/Swagger JSON:
> *   **Tự động tạo test case:** Dựa trên tài liệu Swagger, Antigravity có thể *gọi subagent trình duyệt* để điều hướng đến Swagger UI và tự động chạy các test case cho từng endpoint, hoặc phân tích định nghĩa API để tạo ra các test script cho Postman/Insomnia.
> *   **Phân tích phản hồi:** Khi bạn kiểm thử một API và nhận được phản hồi, Antigravity có thể phân tích JSON response, chỉ ra các trường quan trọng, hoặc cảnh báo về các lỗi trong cấu trúc dữ liệu.
> *   **Đề xuất cải tiến API:** Dựa trên việc phân tích Swagger, AI có thể đề xuất các cải tiến về đặt tên endpoint, các tham số, hoặc cấu trúc phản hồi để tuân thủ tốt hơn các nguyên tắc RESTful.

### 6.3. Sử Dụng Các Công Cụ Kiểm Thử Khác (Postman)

Ngoài Swagger UI, bạn cũng có thể sử dụng các công cụ khác như Postman hoặc Insomnia để kiểm thử API. Các công cụ này cung cấp nhiều tính năng nâng cao hơn cho việc quản lý, tổ chức và kiểm thử các yêu cầu HTTP phức tạp.

1.  **Sao chép URL điểm cuối:** Từ Swagger UI hoặc khi ứng dụng chạy, bạn có thể thấy URL của API (ví dụ: `https://localhost:7001/WeatherForecast`).
2.  **Mở Postman:** Tạo một yêu cầu mới (New Request).
3.  **Chọn phương thức HTTP:** Chọn `GET` (hoặc POST, PUT, DELETE tùy thuộc vào yêu cầu của bạn).
4.  **Dán URL:** Dán URL vào trường yêu cầu.
5.  **Gửi yêu cầu:** Nhấp vào nút "Send".
6.  **Xem phản hồi:** Postman sẽ hiển thị mã trạng thái, thời gian phản hồi, các tiêu đề và dữ liệu JSON trong phần thân phản hồi.

> [!NOTE]
> Việc hiểu và sử dụng Swagger UI là rất quan trọng cho quá trình phát triển API, giúp bạn nhanh chóng kiểm tra các thay đổi và đảm bảo API hoạt động như mong đợi. Các công cụ như Postman sẽ mở rộng khả năng kiểm thử của bạn lên một tầm cao mới.

## Tóm Tắt Phần 17

*   **Hệ thống kiểm soát phiên bản:** Git là công cụ quản lý mã nguồn cục bộ phân tán, GitHub là nền tảng lưu trữ và cộng tác từ xa.
*   **Thiết lập Git:** Cài đặt Git từ `git-scm.com` và cấu hình `user.name`, `user.email` toàn cục.
*   **Quản lý kho lưu trữ GitHub:** Tạo kho lưu trữ mới trên GitHub với `.gitignore` phù hợp (chọn `Visual Studio` cho dự án .NET), sau đó `git clone` về máy cục bộ.
*   **Tạo dự án ASP.NET Core Web API:** Sử dụng Visual Studio để tạo dự án, chọn mẫu "ASP.NET Core Web API", cấu hình tên dự án/solution, và đảm bảo bật "Enable OpenAPI support" cho Swagger. Luôn nhớ **không chọn** "Place solution and project in the same directory" để có cấu trúc thư mục tốt hơn.
*   **Lưu trữ mã nguồn với Git:** Dùng `git add .` để thêm các thay đổi vào khu vực dàn dựng, `git commit -m "..."` để tạo cam kết cục bộ, và `git push` để đẩy các cam kết lên GitHub.
*   **Cấu trúc API cơ bản:**
    *   `Program.cs`: Điểm khởi đầu của ứng dụng, nơi cấu hình Dependency Injection (DI) và chuỗi middleware xử lý yêu cầu HTTP.
    *   `appsettings.json`: Tệp cấu hình ứng dụng (chuỗi kết nối cơ sở dữ liệu, cài đặt môi trường).
    *   Thư mục `Controllers`: Chứa các lớp Controller kế thừa từ `ControllerBase`, được đánh dấu bằng `[ApiController]` và `[Route]`, xử lý các yêu cầu HTTP thông qua các Action Method (ví dụ: `[HttpGet]`).
*   **Dependency Injection (DI):** Một mẫu thiết kế cốt lõi trong ASP.NET Core giúp giảm sự phụ thuộc, tăng khả năng kiểm thử và bảo trì bằng cách tiêm các dependency vào các lớp thông qua constructor. Các dịch vụ được đăng ký trong `builder.Services`.
*   **Repository Pattern & Entity Framework Core:** Các mẫu và công nghệ quan trọng để tạo lớp trừu tượng cho việc truy cập dữ liệu, giúp Controller không cần biết chi tiết về cơ sở dữ liệu, tăng tính linh hoạt và kiểm thử.
*   **RESTful API:** Một phong cách kiến trúc Web Service dựa trên các nguyên tắc như Client-Server, Stateless, Cacheable, Layered System và Uniform Interface.
*   **HTTP Verbs:** Các động từ `GET`, `POST`, `PUT`, `PATCH`, `DELETE` được sử dụng để biểu thị các hành động CRUD (Create, Read, Update, Delete) trên tài nguyên. JSON là định dạng dữ liệu phổ biến nhất.
*   **Kiểm thử API:** Chạy dự án trong Visual Studio và sử dụng Swagger UI để xem tài liệu API và kiểm thử các điểm cuối một cách tương tác. Postman là một công cụ thay thế mạnh mẽ với nhiều tính năng nâng cao hơn.
*   **Vibe Coding và Antigravity IDE:** Hệ thống Agentic AI như Antigravity IDE có thể hỗ trợ xuyên suốt quá trình phát triển, từ quản lý Git, khởi tạo dự án, phân tích DI, tạo mã boilerplate cho Repository/EF Core, đến kiểm thử API, thông qua khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file và lập kế hoạch tự động.

<!-- REVIEWED_BY_AGENT -->
