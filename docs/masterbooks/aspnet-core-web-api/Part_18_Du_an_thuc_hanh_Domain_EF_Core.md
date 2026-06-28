# Phần 18: Xây dựng Nền tảng Dự án: Cấu hình Miền và Tích hợp Entity Framework Core

Chào mừng bạn đến với Phần 18 của khóa học, nơi chúng ta sẽ bắt tay vào xây dựng dự án thực tế "NZ Walks" – một RESTful Web API mạnh mẽ sử dụng ASP.NET Core và Entity Framework Core. Chương này là nền tảng, tập trung vào việc định nghĩa miền ứng dụng, chuyển đổi chúng thành các mô hình dữ liệu, và thiết lập Entity Framework Core để tương tác với cơ sở dữ liệu. Đặc biệt, chúng ta sẽ đi sâu vào Dependency Injection (DI) – một triết lý thiết kế cốt lõi trong ASP.NET Core, để hiểu cách nó giúp chúng ta quản lý các phụ thuộc như `DbContext` một cách hiệu quả.

Chúng ta sẽ khám phá các chủ đề sau:
*   Hiểu và định nghĩa miền ứng dụng theo hướng Domain-Driven Design (DDD).
*   Chuyển đổi các thực thể miền thành các lớp C# (Domain Models).
*   Tích hợp Entity Framework Core: Cài đặt các gói NuGet cần thiết.
*   Tạo và cấu hình lớp `DbContext` để quản lý phiên làm việc với cơ sở dữ liệu.
*   Thiết lập và bảo mật chuỗi kết nối đến cơ sở dữ liệu.
*   Phân tích sâu về Dependency Injection và cách ASP.NET Core áp dụng nó.
*   Đăng ký `DbContext` vào hệ thống Dependency Injection của ứng dụng, hiểu rõ về vòng đời của dịch vụ.

Hãy cùng kiến tạo cấu trúc xương sống cho ứng dụng của chúng ta!

## 1. Định nghĩa Miền (Domain) của Ứng dụng

Trước khi viết bất kỳ dòng mã nào, việc hiểu rõ miền vấn đề mà ứng dụng giải quyết là tối quan trọng. Đây là bước đầu tiên trong quá trình phát triển phần mềm, giúp chúng ta hình dung các thực thể (entities), các thuộc tính của chúng, và mối quan hệ giữa chúng một cách rõ ràng. Phương pháp này thường được gọi là **Domain-Driven Design (DDD)**, nơi trọng tâm là miền nghiệp vụ và ngôn ngữ chung (Ubiquitous Language) giữa các bên liên quan.

### 1.1. Giới thiệu Dự án "NZ Walks"

Dự án "NZ Walks" là một RESTful API được thiết kế để quản lý các chuyến đi bộ (walks) độc đáo tại New Zealand. Các chuyến đi bộ này sẽ được phân loại và liên kết với các khu vực (regions) địa lý cụ thể, và mỗi chuyến đi sẽ có một mức độ khó nhất định.

### 1.2. Xác định Các Thực thể Chính và Thuộc tính

Dựa trên yêu cầu nghiệp vụ, chúng ta xác định ba thực thể chính, đại diện cho các khái niệm cốt lõi trong miền "NZ Walks":

1.  **Region (Khu vực):** Đại diện cho một khu vực địa lý ở New Zealand.
    *   `Id` (GUID): Mã định danh duy nhất (Primary Key). GUID là lựa chọn tốt cho các khóa chính phân tán, tránh xung đột ID khi tạo dữ liệu ở nhiều nơi.
    *   `Code` (string): Mã ngắn gọn, duy nhất cho khu vực (ví dụ: "AKL" cho Auckland).
    *   `Name` (string): Tên đầy đủ và dễ đọc của khu vực.
    *   `Area` (double): Diện tích của khu vực tính bằng kilômét vuông (ví dụ: 1210.5).
    *   `Lat` (double): Vĩ độ trung tâm của khu vực (ví dụ: -36.8485).
    *   `Long` (double): Kinh độ trung tâm của khu vực (ví dụ: 174.7633).
    *   `Population` (long): Dân số ước tính của khu vực.

2.  **Walk (Chuyến đi bộ):** Đại diện cho một con đường đi bộ cụ thể.
    *   `Id` (GUID): Mã định danh duy nhất (Primary Key).
    *   `Name` (string): Tên của chuyến đi bộ (ví dụ: "Tongariro Alpine Crossing").
    *   `LengthInKm` (double): Chiều dài của chuyến đi bộ tính bằng kilômét.
    *   `WalkImageUrl` (string, nullable): URL đến hình ảnh minh họa cho chuyến đi bộ. Thuộc tính `nullable` cho phép giá trị này có thể để trống.
    *   `RegionId` (GUID): Khóa ngoại (Foreign Key) liên kết đến thực thể `Region`.
    *   `WalkDifficultyId` (GUID): Khóa ngoại liên kết đến thực thể `WalkDifficulty`.

3.  **WalkDifficulty (Độ khó đi bộ):** Đại diện cho mức độ khó của một chuyến đi bộ. Đây là một bảng tra cứu (lookup table), giúp chuẩn hóa dữ liệu độ khó.
    *   `Id` (GUID): Mã định danh duy nhất (Primary Key).
    *   `Code` (string): Mã độ khó (ví dụ: "Easy", "Medium", "Hard").

### 1.3. Phân tích Mối Quan hệ Giữa Các Thực thể

Việc xác định mối quan hệ là rất quan trọng để Entity Framework Core có thể ánh xạ chúng chính xác vào cơ sở dữ liệu.

*   **Region và Walk:** Đây là mối quan hệ **một-nhiều (One-to-Many)**.
    *   Một `Region` có thể chứa nhiều `Walk`.
    *   Mỗi `Walk` chỉ thuộc về một `Region` duy nhất.
    *   Trong cơ sở dữ liệu, điều này được thể hiện bằng một khóa ngoại `RegionId` trong bảng `Walk` trỏ đến `Id` của bảng `Region`.

*   **Walk và WalkDifficulty:** Đây cũng là mối quan hệ **một-nhiều (One-to-Many)**.
    *   Một `WalkDifficulty` (ví dụ: "Easy") có thể áp dụng cho nhiều `Walk`.
    *   Mỗi `Walk` chỉ có một `WalkDifficulty` duy nhất.
    *   Trong cơ sở dữ liệu, điều này được thể hiện bằng một khóa ngoại `WalkDifficultyId` trong bảng `Walk` trỏ đến `Id` của bảng `WalkDifficulty`.

> [!NOTE]
> **Tư duy Vibe Coding với Antigravity IDE:**
> Imagine Antigravity IDE, với khả năng Agentic AI siêu việt, phân tích yêu cầu dự án "NZ Walks". Nó không chỉ liệt kê các thực thể; nó có thể tự động đề xuất một lược đồ chuẩn hóa, phát hiện các mối quan hệ tiềm năng, và thậm chí trình bày một biểu đồ ERD (Entity-Relationship Diagram) trực quan. Bạn, với tư duy Vibe Coding, sẽ tinh chỉnh các đề xuất này, tập trung vào ý nghĩa nghiệp vụ và luồng dữ liệu, trong khi Antigravity tự động xử lý các chi tiết kỹ thuật, đảm bảo rằng "vibe" của miền được truyền tải một cách chính xác vào cấu trúc dữ liệu.

## 2. Tạo Mô hình Miền (Domain Models) trong ASP.NET Web API

Sau khi đã định nghĩa miền và các mối quan hệ, bước tiếp theo là chuyển đổi các thực thể trừu tượng này thành các lớp C# cụ thể. Các lớp này được gọi là **mô hình miền (domain models)** hoặc **thực thể (entities)**. Trong kiến trúc ứng dụng, chúng ta sẽ đặt chúng trong thư mục `Models/Domain` để phân tách rõ ràng với các loại mô hình khác (ví dụ: Data Transfer Objects - DTOs).

```csharp
// Tạo thư mục: Models/Domain

// Models/Domain/Region.cs
using System; // Cần cho Guid
using System.Collections.Generic; // Cần cho IEnumerable

namespace NZWalks.API.Models.Domain
{
    public class Region
    {
        public Guid Id { get; set; } // Khóa chính
        public string Code { get; set; } // Mã ngắn gọn
        public string Name { get; set; } // Tên đầy đủ
        public double Area { get; set; } // Diện tích (km vuông)
        public double Lat { get; set; } // Vĩ độ
        public double Long { get; set; } // Kinh độ
        public long Population { get; set; } // Dân số

        // Thuộc tính điều hướng (Navigation Property)
        // Một khu vực có thể có nhiều chuyến đi bộ.
        // Đây là một tập hợp các thực thể phụ thuộc.
        public IEnumerable<Walk> Walks { get; set; }
    }
}

// Models/Domain/WalkDifficulty.cs
using System; // Cần cho Guid

namespace NZWalks.API.Models.Domain
{
    public class WalkDifficulty
    {
        public Guid Id { get; set; } // Khóa chính
        public string Code { get; set; } // Mã độ khó (ví dụ: "Easy", "Medium", "Hard")
    }
}

// Models/Domain/Walk.cs
using System; // Cần cho Guid

namespace NZWalks.API.Models.Domain
{
    public class Walk
    {
        public Guid Id { get; set; } // Khóa chính
        public string Name { get; set; } // Tên chuyến đi bộ
        public double LengthInKm { get; set; } // Chiều dài (km)
        public string? WalkImageUrl { get; set; } // URL hình ảnh (có thể null)

        // Khóa ngoại (Foreign Keys)
        // EF Core sẽ tự động nhận diện đây là khóa ngoại nếu tên theo quy ước (TênThựcThểChaId)
        public Guid RegionId { get; set; }
        public Guid WalkDifficultyId { get; set; }

        // Thuộc tính điều hướng (Navigation Properties)
        // Một chuyến đi bộ thuộc về một khu vực cụ thể.
        // Đây là một thực thể phụ thuộc đơn lẻ.
        public Region Region { get; set; }

        // Một chuyến đi bộ có một độ khó cụ thể.
        // Đây là một thực thể phụ thuộc đơn lẻ.
        public WalkDifficulty WalkDifficulty { get; set; }
    }
}
```

> [!TIP]
> **Thuộc tính điều hướng (Navigation Properties) và Entity Framework Core:**
> Các thuộc tính như `IEnumerable<Walk> Walks` trong `Region` hoặc `Region Region` trong `Walk` không lưu trữ dữ liệu trực tiếp trong cơ sở dữ liệu. Thay vào đó, chúng là "cầu nối" giúp Entity Framework Core hiểu và quản lý các mối quan hệ giữa các thực thể. Khi bạn truy vấn một `Region` và muốn lấy các `Walk` liên quan, EF Core có thể tự động "tải" (load) dữ liệu này thông qua các thuộc tính điều hướng. Điều này đơn giản hóa đáng kể việc làm việc với dữ liệu quan hệ trong mã C# của bạn, cho phép bạn điều hướng qua các đối tượng như thể chúng đã được liên kết trong bộ nhớ.
>
> **Vibe Coding với Antigravity IDE:**
> Một khi miền đã được "vibe-checked" và định nghĩa rõ ràng, Antigravity IDE có thể tự động scaffolding (tạo khung) các mô hình miền C# này. Nó sẽ xử lý các không gian tên, kiểu dữ liệu (`Guid`, `string`, `double`, `long`), hỗ trợ nullability (`string?`), và thậm chí suy luận các thuộc tính điều hướng dựa trên mối quan hệ bạn đã xác định. Điều này giúp bạn tiết kiệm thời gian gõ code lặp đi lặp lại và đảm bảo tính nhất quán, để bạn có thể tập trung vào logic nghiệp vụ phức tạp hơn.

## 3. Tích hợp Entity Framework Core: Cài đặt NuGet Packages

Entity Framework Core (EF Core) là một ORM (Object-Relational Mapper) hiện đại và mạnh mẽ cho .NET. Nó đóng vai trò là một "phiên dịch viên" giữa các đối tượng C# của bạn (các mô hình miền) và cơ sở dữ liệu quan hệ. EF Core cho phép bạn làm việc với dữ liệu bằng cách sử dụng LINQ (Language Integrated Query) thay vì viết các truy vấn SQL thô, đồng thời cung cấp các tính năng như theo dõi thay đổi, quản lý lược đồ cơ sở dữ liệu thông qua Migrations, và hỗ trợ nhiều loại cơ sở dữ liệu khác nhau (SQL Server, PostgreSQL, MySQL, SQLite, v.v.).

Để sử dụng EF Core với SQL Server, chúng ta cần cài đặt hai gói NuGet chính:

1.  **`Microsoft.EntityFrameworkCore.SqlServer`**: Đây là nhà cung cấp cơ sở dữ liệu (database provider) cho SQL Server. Gói này chứa các triển khai cụ thể cho phép EF Core hiểu và tương tác với các máy chủ SQL Server.
2.  **`Microsoft.EntityFrameworkCore.Tools`**: Gói này cung cấp các công cụ dòng lệnh (CLI) và PowerShell để thực hiện các tác vụ quan trọng như tạo Migrations (để cập nhật lược đồ cơ sở dữ liệu), áp dụng Migrations, và quản lý `DbContext` từ cửa sổ Package Manager Console trong Visual Studio hoặc .NET CLI.

### 3.1. Các bước Cài đặt

Mở **Package Manager Console** trong Visual Studio (đi tới *Tools > NuGet Package Manager > Package Manager Console*) và chạy các lệnh sau:

```powershell
# Cài đặt gói SQL Server Provider
Install-Package Microsoft.EntityFrameworkCore.SqlServer

# Cài đặt gói Tools (cho Migrations và các lệnh khác)
Install-Package Microsoft.EntityFrameworkCore.Tools
```

Hoặc, nếu bạn ưa thích .NET CLI (có thể chạy từ bất kỳ terminal nào trong thư mục dự án):

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

> [!NOTE]
> Sau khi cài đặt thành công, các gói này sẽ được liệt kê trong phần `Dependencies/Packages` của dự án. Hãy đảm bảo bạn đã chọn đúng dự án trong Package Manager Console nếu bạn có nhiều dự án trong solution của mình.
>
> **Vibe Coding với Antigravity IDE:**
> Thay vì bạn phải nhớ và gõ các lệnh này, Antigravity IDE, nhận biết ý định của bạn khi bạn bắt đầu tạo `DbContext`, có thể tự động đề xuất hoặc thậm chí chạy ngầm các lệnh `dotnet add package` cần thiết. Một subagent của Antigravity có thể đọc file `.csproj`, kiểm tra các dependencies, và tự động cài đặt gói bị thiếu, sau đó chạy `dotnet restore`. Điều này giúp bạn duy trì "vibe" lập trình ở mức cao hơn, tập trung vào thiết kế kiến trúc thay vì quản lý các lệnh nhỏ nhặt.

## 4. Tạo Lớp DbContext: Cầu nối tới Cơ sở dữ liệu

`DbContext` là một lớp trung tâm trong Entity Framework Core, đóng vai trò là "phiên làm việc" hoặc "đơn vị công việc" (Unit of Work) với cơ sở dữ liệu. Nó không chỉ là một cầu nối mà còn là một **Identity Map** (ánh xạ định danh), đảm bảo rằng mỗi thực thể được tải từ cơ sở dữ liệu chỉ có một thể hiện duy nhất trong bộ nhớ trong một phiên làm việc.

`DbContext` được sử dụng để:
*   **Truy vấn dữ liệu:** Chuyển đổi các truy vấn LINQ thành SQL và thực thi chúng trên cơ sở dữ liệu.
*   **Theo dõi thay đổi:** Theo dõi trạng thái của các thực thể (Added, Modified, Deleted, Unchanged) để biết những gì cần lưu vào cơ sở dữ liệu.
*   **Lưu thay đổi:** Ghi các thay đổi đã theo dõi vào cơ sở dữ liệu thông qua phương thức `SaveChanges()`.

Chúng ta sẽ tạo một lớp `DbContext` tùy chỉnh, kế thừa từ lớp `Microsoft.EntityFrameworkCore.DbContext`.

### 4.1. Các bước Tạo DbContext

1.  Tạo một thư mục mới có tên `Data` trong thư mục gốc của dự án.
2.  Trong thư mục `Data`, tạo một lớp mới có tên `NZWalksDbContext.cs`.
3.  Lớp này sẽ kế thừa từ `Microsoft.EntityFrameworkCore.DbContext`.

```csharp
// Tạo thư mục: Data

// Data/NZWalksDbContext.cs
using Microsoft.EntityFrameworkCore; // Cần cho việc kế thừa DbContext
using NZWalks.API.Models.Domain; // Import namespace chứa các Domain Models

namespace NZWalks.API.Data
{
    public class NZWalksDbContext : DbContext
    {
        // Constructor nhận DbContextOptions và chuyển nó lên lớp cơ sở (DbContext).
        // DbContextOptions chứa thông tin cấu hình như chuỗi kết nối và nhà cung cấp DB.
        public NZWalksDbContext(DbContextOptions<NZWalksDbContext> options) : base(options)
        {
        }

        // Định nghĩa các DbSet cho từng thực thể miền.
        // Mỗi DbSet sẽ tương ứng với một bảng trong cơ sở dữ liệu.
        // Tên thuộc tính (ví dụ: Regions) sẽ là tên bảng mặc định.
        public DbSet<Region> Regions { get; set; }
        public DbSet<Walk> Walks { get; set; }
        public DbSet<WalkDifficulty> WalkDifficulties { get; set; }
    }
}
```

> [!TIP]
> **`DbSet<T>`: Đại diện cho một tập hợp thực thể**
> Mỗi thuộc tính `DbSet<T>` trong `NZWalksDbContext` (ví dụ: `DbSet<Region> Regions`) đại diện cho một tập hợp các thực thể của loại `T` và ánh xạ tới một bảng trong cơ sở dữ liệu. Khi bạn tương tác với `_dbContext.Regions`, bạn đang thực hiện các thao tác trên bảng `Regions` trong cơ sở dữ liệu. EF Core sử dụng các `DbSet` này để xây dựng các truy vấn LINQ, theo dõi thay đổi và tạo các Migrations để cập nhật lược đồ cơ sở dữ liệu.
>
> **Vibe Coding với Antigravity IDE:**
> Antigravity IDE, khi bạn đã định nghĩa các mô hình miền, có thể tự động tạo khung lớp `NZWalksDbContext` này. Nó sẽ thêm constructor với `DbContextOptions` và tự động tạo các thuộc tính `DbSet` cho từng mô hình miền mà nó tìm thấy trong thư mục `Models/Domain`. Điều này giúp đảm bảo sự nhất quán và giảm thiểu lỗi chính tả, cho phép bạn duy trì "vibe" của mình trong việc thiết kế kiến trúc cao cấp.

## 5. Tạo Chuỗi Kết nối đến Cơ sở Dữ liệu

Để `DbContext` có thể thiết lập kết nối với cơ sở dữ liệu, nó cần một chuỗi kết nối (connection string). Trong các ứng dụng ASP.NET Core, chuỗi kết nối thường được lưu trữ trong tệp `appsettings.json` để dễ dàng cấu hình và quản lý môi trường.

### 5.1. Cấu hình `appsettings.json`

Mở tệp `appsettings.json` trong thư mục gốc của dự án và thêm một phần `ConnectionStrings` như sau:

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
    "NZWalksConnectionString": "Server=(localdb)\\mssqllocaldb;Database=NZWalksDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

Giải thích chi tiết chuỗi kết nối:
*   `"NZWalksConnectionString"`: Đây là tên định danh mà chúng ta sẽ sử dụng để tham chiếu chuỗi kết nối này trong mã C#. Hãy chọn một tên rõ ràng và dễ hiểu.
*   `Server=(localdb)\\mssqllocaldb`: Chỉ định máy chủ cơ sở dữ liệu. `(localdb)\\mssqllocaldb` là một phiên bản cục bộ của SQL Server Express, thường được cài đặt cùng với Visual Studio, rất tiện lợi cho việc phát triển. Bạn có thể thay đổi nó thành tên máy chủ SQL Server của bạn (ví dụ: `DESKTOP-ABC\SQLEXPRESS`).
*   `Database=NZWalksDb`: Tên của cơ sở dữ liệu mà chúng ta muốn kết nối. Nếu cơ sở dữ liệu này chưa tồn tại, EF Core sẽ tạo nó khi chúng ta chạy các Migrations sau này.
*   `Trusted_Connection=True`: Cho phép ứng dụng sử dụng xác thực Windows để kết nối với cơ sở dữ liệu. Điều này tiện lợi và an toàn cho môi trường phát triển cục bộ vì không yêu cầu thông tin đăng nhập riêng.
*   `MultipleActiveResultSets=true`: (Tùy chọn nhưng hữu ích) Cho phép nhiều tập kết quả đang hoạt động trên một kết nối duy nhất. Điều này có thể cần thiết nếu bạn đang thực hiện nhiều thao tác dữ liệu cùng lúc hoặc sử dụng các thuộc tính điều hướng tải lười (lazy loading) phức tạp.

> [!CAUTION]
> **Bảo mật Chuỗi Kết nối trong Môi trường Sản xuất:**
> Mặc dù `Trusted_Connection=True` tiện lợi cho phát triển, trong môi trường sản xuất, bạn không nên lưu trữ chuỗi kết nối trực tiếp trong `appsettings.json` hoặc sử dụng xác thực Windows nếu ứng dụng của bạn chạy trên một máy chủ khác. Thay vào đó, hãy xem xét các phương pháp an toàn hơn như:
> *   **Biến môi trường (Environment Variables):** Đặc biệt hữu ích trong môi trường container (Docker, Kubernetes) hoặc cloud (Azure App Services, AWS Elastic Beanstalk).
> *   **Azure Key Vault (hoặc các dịch vụ tương tự):** Lưu trữ bí mật một cách an toàn và truy cập chúng qua mã.
> *   **Xác thực SQL Server:** Sử dụng tên người dùng và mật khẩu riêng biệt cho cơ sở dữ liệu.
>
> **Vibe Coding với Antigravity IDE:**
> Antigravity IDE có thể đề xuất thêm phần `ConnectionStrings` này vào `appsettings.json`. Hơn nữa, với tư cách là một chuyên gia lập trình cấp senior, Antigravity có thể đưa ra cảnh báo về bảo mật cho môi trường sản xuất và gợi ý các giải pháp lưu trữ chuỗi kết nối an toàn hơn, thậm chí tự động tạo cấu hình mẫu cho các phương pháp đó, giúp bạn duy trì "vibe" bảo mật ngay từ đầu.

## 6. Hiểu về Dependency Injection (DI): Nền tảng của ASP.NET Core

Trước khi chúng ta "inject" `DbContext` vào ứng dụng, việc nắm vững Dependency Injection (DI) là điều kiện tiên quyết để làm việc hiệu quả với ASP.NET Core. DI không chỉ là một tính năng; nó là một triết lý thiết kế cốt lõi của framework này.

### 6.1. Dependency Injection là gì?

**Dependency Injection (DI)** là một mẫu thiết kế phần mềm để đạt được **Inversion of Control (IoC)** (Đảo ngược quyền điều khiển). Ý tưởng chính là: thay vì một lớp tự tạo ra hoặc tự quản lý các đối tượng mà nó cần (các phụ thuộc - dependencies), các đối tượng phụ thuộc này sẽ được "tiêm" (injected) vào lớp đó từ bên ngoài.

Hãy so sánh hai cách tiếp cận:

**1. Cách tiếp cận truyền thống (Tight Coupling - Ghép nối chặt chẽ):**
Một lớp tự chịu trách nhiệm tạo ra các đối tượng mà nó cần.

```csharp
// Lớp này tự tạo ra dependency của nó
public class MyService
{
    private MyDatabaseLogger _logger;

    public MyService()
    {
        // MyService tự khởi tạo MyDatabaseLogger.
        // Nó phụ thuộc chặt chẽ vào một triển khai cụ thể.
        _logger = new MyDatabaseLogger();
    }

    public void ProcessData(string data)
    {
        _logger.Log($"Processing data: {data}");
        // ... logic xử lý dữ liệu ...
    }
}

public class MyDatabaseLogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[DB Logger] {message}");
        // Logic ghi vào cơ sở dữ liệu
    }
}
```
Vấn đề:
*   `MyService` phụ thuộc chặt chẽ vào `MyDatabaseLogger`. Nếu bạn muốn thay đổi cơ chế ghi nhật ký (ví dụ: ghi vào file thay vì DB), bạn phải sửa đổi `MyService`.
*   Kiểm thử `MyService` rất khó khăn vì bạn không thể dễ dàng thay thế `MyDatabaseLogger` bằng một đối tượng giả (mock) trong kiểm thử đơn vị.

**2. Cách tiếp cận với Dependency Injection (Loose Coupling - Ghép nối lỏng lẻo):**
Một lớp khai báo các dependency mà nó cần thông qua một giao diện, và một cơ chế bên ngoài sẽ cung cấp các dependency đó.

```csharp
// 1. Định nghĩa giao diện cho dependency
public interface ILogger
{
    void Log(string message);
}

// 2. Tạo các triển khai cụ thể của giao diện
public class MyDatabaseLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[DB Logger] {message}");
        // Logic ghi vào cơ sở dữ liệu
    }
}

public class MyFileLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[File Logger] {message}");
        // Logic ghi vào tệp
    }
}

// 3. Lớp MyService nhận dependency qua constructor (Constructor Injection)
public class MyService
{
    private readonly ILogger _logger; // Phụ thuộc vào giao diện, không phải triển khai cụ thể

    // Dependency được "tiêm" vào qua constructor
    public MyService(ILogger logger)
    {
        _logger = logger;
    }

    public void ProcessData(string data)
    {
        _logger.Log($"Processing data: {data}");
        // ... logic xử lý dữ liệu ...
    }
}
```
Ở đây, `MyService` không còn biết hoặc quan tâm đến việc `ILogger` được triển khai như thế nào. Nó chỉ biết rằng nó cần một đối tượng có khả năng `Log`.

### 6.2. Lợi ích của Dependency Injection

DI mang lại nhiều lợi ích quan trọng, đặc biệt trong các ứng dụng lớn và phức tạp:

*   **Loose Coupling (Ghép nối lỏng lẻo):** Các lớp không phụ thuộc vào các triển khai cụ thể mà phụ thuộc vào các giao diện/abstraction. Điều này giúp dễ dàng thay đổi triển khai mà không ảnh hưởng đến lớp sử dụng.
*   **Testability (Khả năng kiểm thử):** Bạn có thể dễ dàng thay thế các dependency bằng các đối tượng giả (mocks/stubs) trong các bài kiểm thử đơn vị, cho phép kiểm thử một lớp một cách độc lập và hiệu quả.
*   **Maintainability (Khả năng bảo trì):** Mã dễ đọc, dễ hiểu và dễ bảo trì hơn vì các phụ thuộc được khai báo rõ ràng.
*   **Reusability (Khả năng tái sử dụng):** Các thành phần có thể được tái sử dụng trong các ngữ cảnh khác nhau với các triển khai dependency khác nhau.
*   **Scalability (Khả năng mở rộng):** Dễ dàng thêm các chức năng mới hoặc thay đổi các thành phần hiện có mà không làm hỏng toàn bộ hệ thống.

### 6.3. Dependency Injection trong ASP.NET Core: Service Container

ASP.NET Core có một hệ thống Dependency Injection tích hợp sẵn được gọi là **Service Container** (hoặc IoC Container). Container này là một "nhà máy" thông minh chịu trách nhiệm:

1.  **Đăng ký (Register) các dịch vụ:** Bạn khai báo các giao diện và các triển khai tương ứng của chúng cho container biết.
2.  **Giải quyết (Resolve) các dịch vụ:** Khi một lớp yêu cầu một dependency trong constructor của nó, container sẽ tự động:
    *   Xác định triển khai nào cần được cung cấp.
    *   Tạo một thể hiện của dependency đó (hoặc sử dụng một thể hiện hiện có, tùy thuộc vào vòng đời đã đăng ký).
    *   Truyền thể hiện đó vào constructor của lớp yêu cầu.

Việc đăng ký dịch vụ thường được thực hiện trong tệp `Program.cs` (hoặc `Startup.cs` trong các phiên bản cũ hơn) bằng cách sử dụng đối tượng `IServiceCollection`.

> [!NOTE]
> **Vòng đời của Dịch vụ (Service Lifetimes):**
> Khi đăng ký dịch vụ, bạn cần chỉ định vòng đời của chúng:
> *   **Singleton:** Một thể hiện duy nhất được tạo và tái sử dụng trong suốt vòng đời của ứng dụng. (Ví dụ: dịch vụ cấu hình, bộ nhớ cache).
> *   **Scoped:** Một thể hiện được tạo một lần cho mỗi yêu cầu HTTP và được tái sử dụng trong phạm vi của yêu cầu đó. Khi yêu cầu kết thúc, thể hiện đó sẽ bị hủy. Đây là vòng đời phổ biến nhất cho các dịch vụ liên quan đến cơ sở dữ liệu như `DbContext`.
> *   **Transient:** Một thể hiện mới được tạo mỗi khi dịch vụ được yêu cầu. (Ví dụ: các dịch vụ nhẹ, không trạng thái).
> Việc lựa chọn vòng đời phù hợp là rất quan trọng để quản lý tài nguyên và tránh các vấn đề về trạng thái.

> [!TIP]
> **Tư duy Vibe Coding với Antigravity IDE:**
> Hiểu rõ DI là chìa khóa để "Vibe Coding" hiệu quả trong ASP.NET Core. Thay vì bận tâm về việc khởi tạo đối tượng, bạn chỉ cần khai báo "ý định" của mình (tôi cần một `ILogger`, tôi cần một `DbContext`). Antigravity IDE, với khả năng lập kế hoạch tự động, có thể tự động cấu hình container DI dựa trên các khai báo này, thậm chí gợi ý vòng đời phù hợp cho từng dịch vụ. Điều này giúp bạn duy trì luồng tư duy ở cấp độ kiến trúc và nghiệp vụ, trong khi AI xử lý các chi tiết triển khai bên dưới.

## 7. Đăng ký và Injecting DbContext Class

Bây giờ chúng ta đã hiểu về Dependency Injection, đã đến lúc đăng ký `NZWalksDbContext` của chúng ta vào Service Container của ASP.NET Core. Điều này sẽ cho phép chúng ta "inject" `NZWalksDbContext` vào bất kỳ lớp nào cần tương tác với cơ sở dữ liệu (ví dụ: các lớp Repository, Controllers) mà không cần phải tự tạo thể hiện của nó.

### 7.1. Cấu hình trong `Program.cs`

Mở tệp `Program.cs` và thêm đoạn mã sau vào phần cấu hình dịch vụ (thường là trước `var app = builder.Build();`):

```csharp
using Microsoft.EntityFrameworkCore; // Cần thiết cho phương thức mở rộng UseSqlServer
using NZWalks.API.Data; // Cần thiết để tham chiếu NZWalksDbContext

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Đăng ký NZWalksDbContext vào hệ thống Dependency Injection
// DbContext thường được đăng ký với vòng đời Scoped.
builder.Services.AddDbContext<NZWalksDbContext>(options =>
{
    // Cấu hình DbContext để sử dụng SQL Server.
    // Lấy chuỗi kết nối từ cấu hình ứng dụng (appsettings.json, biến môi trường, v.v.).
    options.UseSqlServer(builder.Configuration.GetConnectionString("NZWalksConnectionString"));
});

var app = builder.Build();

// Configure the HTTP request pipeline.
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

Giải thích đoạn mã:
*   `builder.Services.AddDbContext<NZWalksDbContext>(...)`: Đây là phương thức mở rộng được cung cấp bởi gói `Microsoft.EntityFrameworkCore.SqlServer` để đăng ký `DbContext` vào Service Container. Mặc định, `DbContext` được đăng ký với vòng đời **Scoped**. Điều này có nghĩa là một thể hiện mới của `NZWalksDbContext` sẽ được tạo cho mỗi yêu cầu HTTP và được sử dụng trong suốt quá trình xử lý yêu cầu đó. Khi yêu cầu hoàn tất, thể hiện `DbContext` sẽ được hủy bỏ. Đây là hành vi mong muốn để đảm bảo tính nhất quán dữ liệu và quản lý tài nguyên hiệu quả trong các ứng dụng web.
*   `options => { ... }`: Chúng ta truyền một lambda expression để cấu hình các tùy chọn cho `NZWalksDbContext`.
*   `options.UseSqlServer(...)`: Phương thức này chỉ định rằng `NZWalksDbContext` sẽ sử dụng SQL Server làm nhà cung cấp cơ sở dữ liệu.
*   `builder.Configuration.GetConnectionString("NZWalksConnectionString")`: Phương thức này đọc chuỗi kết nối có tên "NZWalksConnectionString" từ các nguồn cấu hình đã được tải, trong trường hợp này là từ tệp `appsettings.json`.

> [!NOTE]
> Sau khi `NZWalksDbContext` được đăng ký, bất kỳ lớp nào trong ứng dụng của bạn cần truy cập cơ sở dữ liệu chỉ cần khai báo `NZWalksDbContext` trong constructor của nó. Hệ thống DI của ASP.NET Core sẽ tự động cung cấp một thể hiện của `NZWalksDbContext` cho lớp đó. Đây là một ví dụ điển hình về việc sử dụng Dependency Injection để quản lý các phụ thuộc một cách hiệu quả.

Ví dụ về cách sử dụng `NZWalksDbContext` trong một lớp khác (ví dụ: một lớp Repository):

```csharp
// Ví dụ: Một Repository sẽ sử dụng DbContext để tương tác với cơ sở dữ liệu
using NZWalks.API.Data; // Cần thiết để tham chiếu NZWalksDbContext
using NZWalks.API.Models.Domain; // Cần thiết để tham chiếu Domain Models
using Microsoft.EntityFrameworkCore; // Cần để sử dụng các phương thức mở rộng như ToListAsync()

namespace NZWalks.API.Repositories
{
    public class RegionRepository
    {
        private readonly NZWalksDbContext _dbContext;

        // DbContext được inject qua constructor bởi Service Container của ASP.NET Core
        public RegionRepository(NZWalksDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task<IEnumerable<Region>> GetAllAsync()
        {
            // Sử dụng _dbContext để truy vấn dữ liệu từ bảng Regions
            return await _dbContext.Regions.ToListAsync();
        }

        // Các phương thức khác để thêm, sửa, xóa Regions...
    }
}
```
Trong ví dụ trên, `RegionRepository` không cần biết cách tạo `NZWalksDbContext`. Nó chỉ khai báo rằng nó cần một thể hiện của `NZWalksDbContext` và hệ thống DI sẽ cung cấp nó. Điều này giúp `RegionRepository` dễ kiểm thử, tách biệt hơn khỏi chi tiết triển khai cơ sở dữ liệu, và tuân thủ nguyên tắc Single Responsibility Principle (SRP).

> [!TIP]
> **Tư duy Vibe Coding với Antigravity IDE:**
> Đây là nơi Antigravity IDE thực sự tỏa sáng. Dựa trên sự hiện diện của `NZWalksDbContext` và các mô hình miền, Antigravity có thể tự động thêm đoạn mã `builder.Services.AddDbContext` vào `Program.cs`. Nó sẽ tự động suy luận `UseSqlServer`, lấy chuỗi kết nối từ `appsettings.json`, và thậm chí giải thích lý do `Scoped` là vòng đời phù hợp cho `DbContext`. Khi bạn tạo `RegionRepository` và khai báo `NZWalksDbContext` trong constructor, Antigravity sẽ nhận ra mẫu Dependency Injection và đảm bảo mọi thứ được cấu hình đúng đắn. Với Antigravity, bạn không còn bận tâm về cú pháp hay vị trí cấu hình; bạn chỉ cần "vibe" với kiến trúc và Antigravity sẽ thực hiện các thao tác ngầm (chạy script, đọc/ghi file) để hiện thực hóa ý tưởng của bạn một cách liền mạch.

## Tóm tắt Phần 18

Trong phần này, chúng ta đã đặt nền móng vững chắc cho dự án "NZ Walks", từ việc định hình ý tưởng đến việc thiết lập các công cụ cần thiết để biến ý tưởng đó thành hiện thực. Đây là những điểm cốt lõi chúng ta đã học:

*   **Định nghĩa Miền (Domain Definition):** Chúng ta đã hiểu tầm quan trọng của việc xác định rõ ràng các thực thể (`Region`, `Walk`, `WalkDifficulty`) và mối quan hệ giữa chúng, đặt nền tảng cho một kiến trúc ứng dụng vững chắc theo hướng Domain-Driven Design.
*   **Mô hình Miền (Domain Models):** Chúng ta đã chuyển đổi các định nghĩa miền thành các lớp C# cụ thể (`Region.cs`, `Walk.cs`, `WalkDifficulty.cs`) trong thư mục `Models/Domain`, bao gồm cả việc sử dụng các thuộc tính điều hướng để EF Core hiểu các mối quan hệ.
*   **Entity Framework Core (EF Core):**
    *   Giới thiệu EF Core như một ORM mạnh mẽ giúp chúng ta tương tác với cơ sở dữ liệu bằng các đối tượng C#.
    *   Hướng dẫn cài đặt các gói NuGet cần thiết: `Microsoft.EntityFrameworkCore.SqlServer` và `Microsoft.EntityFrameworkCore.Tools`.
*   **DbContext: Đơn vị Công việc với Cơ sở dữ liệu:**
    *   Tạo lớp `NZWalksDbContext` kế thừa từ `DbContext`, đóng vai trò là phiên làm việc và Identity Map với cơ sở dữ liệu.
    *   Sử dụng `DbSet<T>` để ánh xạ các mô hình miền với các bảng trong cơ sở dữ liệu.
*   **Chuỗi Kết nối:** Cấu hình chuỗi kết nối (`NZWalksConnectionString`) trong `appsettings.json`, bao gồm các chi tiết về máy chủ, tên cơ sở dữ liệu, và phương thức xác thực, đồng thời nhấn mạnh các cân nhắc về bảo mật.
*   **Dependency Injection (DI): Triết lý Thiết kế Cốt lõi:**
    *   Phân tích sâu về khái niệm DI và Inversion of Control (IoC).
    *   Làm rõ các lợi ích của DI: ghép nối lỏng lẻo, khả năng kiểm thử, bảo trì và mở rộng.
    *   Giải thích cách thức hoạt động của Service Container trong ASP.NET Core và các loại vòng đời của dịch vụ (Singleton, Scoped, Transient), đặc biệt là `Scoped` cho `DbContext`.
*   **Injecting DbContext:** Đăng ký `NZWalksDbContext` vào Service Container của ASP.NET Core trong tệp `Program.cs` bằng phương thức `builder.Services.AddDbContext()`, cấu hình nó để sử dụng SQL Server và chuỗi kết nối đã định nghĩa.

Với những thiết lập này, ứng dụng của chúng ta đã sẵn sàng để tương tác với cơ sở dữ liệu thông qua Entity Framework Core, tận dụng sức mạnh của Dependency Injection để xây dựng một kiến trúc linh hoạt và mạnh mẽ. Trong các phần tiếp theo, chúng ta sẽ khám phá cách sử dụng `DbContext` để thực hiện các thao tác CRUD và triển khai Repository Pattern để quản lý dữ liệu một cách có tổ chức hơn.

<!-- REVIEWED_BY_AGENT -->
