# Phần 3: Tích hợp Cơ sở dữ liệu với Entity Framework Core – Xây dựng Nền tảng Dữ liệu Mạnh Mẽ

Chào mừng bạn đến với Phần 3, một cột mốc quan trọng trong hành trình xây dựng RESTful Web API với ASP.NET Core. Trong chương này, chúng ta sẽ thiết lập xương sống cho mọi ứng dụng web thực tế: **khả năng tương tác với cơ sở dữ liệu**. Chúng ta sẽ khai thác sức mạnh của Entity Framework Core (EF Core) để chuyển đổi các khái niệm nghiệp vụ thành cấu trúc dữ liệu, đồng thời khám phá các nguyên tắc thiết kế cốt lõi như Dependency Injection (DI) và vai trò của chúng trong việc xây dựng một kiến trúc bền vững, dễ mở rộng, đặc biệt là khi chuẩn bị cho Repository Pattern và Controllers xử lý các HTTP Verbs (GET, POST, PUT, DELETE) ở các phần sau.

Chúng ta sẽ không chỉ đi qua từng bước cài đặt và cấu hình, mà còn đào sâu vào "cơ chế ngầm" (under the hood) của EF Core và DI, giúp bạn không chỉ biết *cách làm* mà còn hiểu *tại sao làm* như vậy. Đặc biệt, chúng ta sẽ liên hệ cách một hệ thống Agentic AI như Antigravity IDE sẽ tiếp cận và tự động hóa các bước này, giúp bạn hình thành tư duy "Vibe Coding" – tập trung vào ý định và mục tiêu, để công cụ tự động thực hiện chi tiết.

## 1. Định hình Miền Ứng dụng (Domain Definition)

Trước khi viết bất kỳ dòng mã nào liên quan đến cơ sở dữ liệu, việc xác định rõ miền (domain) của ứng dụng là tối quan trọng. Miền là trái tim của ứng dụng, nơi chứa các quy tắc nghiệp vụ, dữ liệu và logic cốt lõi giải quyết vấn đề mà ứng dụng của bạn hướng tới.

### 1.1. Giới thiệu về Domain-Driven Design (DDD) và Vibe Coding

**Domain-Driven Design (DDD)** là một phương pháp phát triển phần mềm tập trung vào việc tạo ra một mô hình miền giàu tính biểu cảm, phản ánh chính xác các khái niệm và hành vi của nghiệp vụ. Mục tiêu là làm cho mã nguồn dễ hiểu, dễ bảo trì và mở rộng hơn bằng cách giữ cho kiến trúc phần mềm luôn căn chỉnh với yêu cầu nghiệp vụ.

Trong bối cảnh của một hệ thống Agentic AI như **Antigravity IDE**, tư duy DDD trở nên cực kỳ mạnh mẽ khi kết hợp với **Vibe Coding**. Thay vì phải mô tả chi tiết từng thuộc tính hay mối quan hệ, bạn có thể truyền tải "vibe" – tức là ý định và ngữ cảnh tổng thể của miền – cho Antigravity. Ví dụ, bạn có thể nói: "Tôi đang xây dựng một API quản lý các tuyến đường đi bộ ở New Zealand. Các tuyến đường này có tên, mô tả, chiều dài, và liên quan đến độ khó, khu vực." Antigravity với khả năng tự chạy script, gọi subagent trình duyệt để nghiên cứu, đọc ghi file và lập kế hoạch tự động, có thể:

1.  **Phân tích ngữ cảnh:** Dựa vào mô tả, Antigravity sẽ suy luận các thực thể tiềm năng (`Walk`, `Region`, `Difficulty`).
2.  **Đề xuất thuộc tính:** Dựa trên ngữ cảnh "quản lý tuyến đường đi bộ", nó có thể đề xuất các thuộc tính như `Name`, `Description`, `LengthInKm`, `ImageUrl` cho `Walk`.
3.  **Xác định mối quan hệ:** Tự động nhận diện mối quan hệ One-to-Many giữa `Walk` và `Region`/`Difficulty`.
4.  **Lựa chọn kiểu dữ liệu:** Đề xuất `Guid` cho ID để đảm bảo tính duy nhất phân tán, `string` cho tên/mô tả, `double` cho chiều dài.

Điều này cho phép bạn tập trung vào "cái gì" (what) thay vì "làm thế nào" (how), để Antigravity xử lý các chi tiết triển khai ban đầu, đẩy nhanh đáng kể giai đoạn khởi tạo dự án.

### 1.2. Các Thực thể Miền của Ứng dụng NZWalks

Ứng dụng NZWalks của chúng ta sẽ quản lý các tuyến đường đi bộ, các khu vực địa lý và các cấp độ khó của tuyến đường.

#### 1.2.1. Thực thể `Walk` (Tuyến đường đi bộ)

Đại diện cho một tuyến đường cụ thể.

*   **ID (Guid):** Mã định danh duy nhất. Sử dụng `Guid` giúp tránh xung đột ID khi làm việc với các hệ thống phân tán hoặc khi hợp nhất dữ liệu từ nhiều nguồn.
*   **Name (string):** Tên hiển thị (ví dụ: "Đường mòn núi Ngauruhoe").
*   **Description (string):** Mô tả chi tiết.
*   **LengthInKm (double):** Chiều dài tuyến đường.
*   **WalkImageUrl (string?):** URL hình ảnh của tuyến đường (tùy chọn, có thể null).
*   **DifficultyId (Guid):** Khóa ngoại (Foreign Key) tới `Difficulty`.
*   **RegionId (Guid):** Khóa ngoại tới `Region`.

#### 1.2.2. Thực thể `Region` (Khu vực)

Đại diện cho một khu vực địa lý.

*   **ID (Guid):** Mã định danh duy nhất.
*   **Code (string):** Mã ngắn gọn (ví dụ: "AKL").
*   **Name (string):** Tên đầy đủ (ví dụ: "Auckland").
*   **RegionImageUrl (string?):** URL hình ảnh của khu vực (tùy chọn).

#### 1.2.3. Thực thể `Difficulty` (Độ khó)

Định nghĩa các cấp độ khó.

*   **ID (Guid):** Mã định danh duy nhất.
*   **Name (string):** Tên cấp độ (ví dụ: "Dễ", "Trung bình", "Khó").

### 1.3. Thiết lập Mối quan hệ giữa các Thực thể

Các mối quan hệ này là nền tảng để EF Core hiểu cách liên kết các bảng trong cơ sở dữ liệu:

*   **`Walk` và `Difficulty` (One-to-Many):** Một `Difficulty` có thể liên kết với nhiều `Walk`, nhưng mỗi `Walk` chỉ có một `Difficulty` duy nhất.
*   **`Walk` và `Region` (One-to-Many):** Một `Region` có thể chứa nhiều `Walk`, nhưng mỗi `Walk` chỉ thuộc về một `Region` duy nhất.

    ```mermaid
    flowchart TB
        subgraph CF["🔄 Code-First Migration Workflow"]
            direction TB
            A["📝 Định nghĩa\nDomain Models\n(C# Classes)"] --> B["⚙️ Tạo DbContext\n(Cấu hình kết nối)"]
            B --> C["🔧 Add-Migration\n(Tạo migration file)"]
            C --> D["📊 Update-Database\n(Áp dụng lên DB)"]
            D --> E["🗄️ Database\nđược tạo/cập nhật"]
        end
        subgraph DF["🔄 Database-First (ngược lại)"]
            direction TB
            F["🗄️ Database có sẵn"] --> G["⚙️ Scaffold-DbContext"]
            G --> H["📝 Models + DbContext\nđược tạo tự động"]
        end
        style CF fill:#e8f5e9,color:#000
        style DF fill:#e3f2fd,color:#000
    ```
    *Minh họa: Code-First vs Database-First. Khóa học này sử dụng Code-First — viết C# class trước, EF Core tạo database.*


## 2. Xây dựng Mô hình Miền (Domain Models) trong C#

Sau khi đã định hình miền, bước tiếp theo là chuyển các định nghĩa này thành các lớp C# (thường gọi là Domain Models hoặc Entity Classes). Đây là lúc Antigravity IDE tỏa sáng với khả năng Vibe Coding và tự động tạo mã.

### 2.1. Tổ chức Mã nguồn cho Mô hình Miền

Để duy trì cấu trúc dự án rõ ràng, chúng ta sẽ tổ chức các mô hình miền vào các thư mục logic:

1.  Trong thư mục gốc của dự án, tạo thư mục `Models`.
2.  Bên trong `Models`, tạo thư mục `Domain`.
    *   Cấu trúc: `ProjectName/Models/Domain/`.

**Antigravity IDE và Tổ chức mã:** Với tư duy Vibe Coding, bạn có thể chỉ định: "Tạo các domain models cho NZWalks API và đặt chúng trong thư mục `Models/Domain`." Antigravity sẽ tự động tạo cấu trúc thư mục này và đặt các tệp C# vào đúng vị trí, tuân thủ các quy ước tên tệp và namespace.

### 2.2. Triển khai các Lớp Mô hình Miền

Bây giờ, chúng ta sẽ tạo các lớp C# tương ứng.

#### 2.2.1. Lớp `Difficulty.cs`

Tạo tệp `Difficulty.cs` trong `Models/Domain`.

```csharp
namespace NZWalks.API.Models.Domain
{
    public class Difficulty
    {
        // Guid là kiểu dữ liệu phổ biến cho ID vì nó đảm bảo tính duy nhất trên toàn hệ thống
        // mà không cần phụ thuộc vào cơ sở dữ liệu tự động tăng.
        public Guid Id { get; set; }

        // string.Empty là cách an toàn để khởi tạo các thuộc tính string không nullable,
        // đảm bảo chúng không bao giờ là null khi một đối tượng được tạo mà không gán giá trị.
        public string Name { get; set; } = string.Empty;
    }
}
```

#### 2.2.2. Lớp `Region.cs`

Tạo tệp `Region.cs` trong `Models/Domain`.

```csharp
namespace NZWalks.API.Models.Domain
{
    public class Region
    {
        public Guid Id { get; set; }
        public string Code { get; set; } = string.Empty;
        public string Name { get; set; } = string.Empty;

        // Nullable Reference Types (C# 8.0+):
        // Dấu '?' sau 'string' (string?) biểu thị rằng thuộc tính này có thể chứa giá trị null.
        // Điều này giúp trình biên dịch C# cảnh báo bạn về các lỗi NullReferenceException tiềm ẩn
        // và ánh xạ tới các cột có thể null trong cơ sở dữ liệu.
        public string? RegionImageUrl { get; set; }
    }
}
```

#### 2.2.3. Lớp `Walk.cs`

Tạo tệp `Walk.cs` trong `Models/Domain`.

```csharp
namespace NZWalks.API.Models.Domain
{
    public class Walk
    {
        public Guid Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public double LengthInKm { get; set; }
        public string? WalkImageUrl { get; set; }

        // Khóa ngoại (Foreign Key) để thiết lập mối quan hệ One-to-Many.
        // EF Core sẽ tự động nhận diện đây là khóa ngoại nếu tên tuân theo quy ước:
        // [TênThuocTinhDieuHuong]Id hoặc [TenBangLienKet]Id.
        public Guid DifficultyId { get; set; }
        public Guid RegionId { get; set; }

        // Thuộc tính điều hướng (Navigation Properties):
        // Các thuộc tính này không lưu trữ dữ liệu trực tiếp trong cơ sở dữ liệu.
        // Thay vào đó, chúng giúp EF Core hiểu các mối quan hệ và cho phép bạn tải
        // các đối tượng liên quan (ví dụ: tải đối tượng Difficulty khi truy vấn Walk).
        //
        // Khởi tạo với 'null!':
        // Đây là toán tử "null-forgiving" (null-forgiving operator) trong C# 8.0+.
        // Nó báo cho trình biên dịch biết rằng bạn chắc chắn thuộc tính này sẽ không bao giờ null
        // trong quá trình chạy (ví dụ: EF Core sẽ tự động điền nó khi tải từ DB),
        // ngay cả khi kiểu của nó là non-nullable. Điều này giúp tránh các cảnh báo biên dịch
        // về khả năng null trong khi vẫn giữ kiểu non-nullable cho mục đích ngữ nghĩa.
        public Difficulty Difficulty { get; set; } = null!;
        public Region Region { get; set; } = null!;
    }
}
```

**Mối quan hệ ẩn và rõ ràng trong EF Core:**
EF Core có khả năng phát hiện các mối quan hệ dựa trên quy ước đặt tên (`DifficultyId` và thuộc tính điều hướng `Difficulty`). Tuy nhiên, bạn cũng có thể cấu hình mối quan hệ một cách rõ ràng hơn thông qua `Fluent API` trong phương thức `OnModelCreating` của `DbContext` (chúng ta sẽ tìm hiểu sâu hơn về điều này trong các phần sau), đặc biệt hữu ích cho các kịch bản phức tạp hoặc khi quy ước không đủ.

## 3. Tích hợp Entity Framework Core (EF Core)

Entity Framework Core là Object-Relational Mapper (ORM) mã nguồn mở và đa nền tảng của Microsoft cho .NET. Nó đóng vai trò là cầu nối giữa các đối tượng C# của bạn và cơ sở dữ liệu quan hệ, giúp bạn tương tác với dữ liệu một cách hướng đối tượng.

### 3.1. Cơ chế hoạt động của ORM và EF Core

**Object-Relational Mapper (ORM):** Một ORM tự động ánh xạ các đối tượng trong ngôn ngữ lập trình của bạn (ví dụ: các lớp C# `Walk`, `Region`) tới các bảng, cột và hàng trong cơ sở dữ liệu quan hệ. Điều này loại bỏ nhu cầu viết các câu lệnh SQL trực tiếp cho các thao tác CRUD cơ bản, giảm thiểu lỗi và tăng tốc độ phát triển.

**EF Core đơn giản hóa tương tác cơ sở dữ liệu bằng cách:**

*   **Ánh xạ tự động:** Chuyển đổi giữa đối tượng C# và dữ liệu cơ sở dữ liệu.
*   **Truy vấn LINQ:** Cho phép bạn viết các truy vấn dữ liệu bằng LINQ (Language Integrated Query), một cú pháp quen thuộc và an toàn kiểu, sau đó EF Core dịch chúng thành SQL tối ưu.
*   **Quản lý lược đồ (Migrations):** Cung cấp một cách có kiểm soát để thay đổi cấu trúc cơ sở dữ liệu theo thời gian, đồng bộ với các thay đổi trong mô hình miền của bạn.

**Antigravity IDE và ORM:** Antigravity IDE có thể tự động tạo ra các lớp mô hình miền, `DbContext` và thậm chí cả các truy vấn LINQ cơ bản dựa trên mô tả cấp cao. Khi bạn cần một thao tác dữ liệu, bạn có thể nói: "Lấy tất cả các tuyến đường đi bộ có độ khó 'Dễ' và chiều dài dưới 5km." Antigravity sẽ không chỉ tạo truy vấn LINQ mà còn có thể giải thích SQL mà EF Core sẽ tạo ra, giúp bạn hiểu rõ hơn về hiệu suất.

### 3.2. Cài đặt các Gói NuGet cần thiết

Để sử dụng EF Core với SQL Server, chúng ta cần cài đặt hai gói NuGet chính:

1.  **`Microsoft.EntityFrameworkCore.SqlServer`**: Cung cấp nhà cung cấp cơ sở dữ liệu cho SQL Server, cho phép EF Core hiểu và tương tác với SQL Server.
2.  **`Microsoft.EntityFrameworkCore.Tools`**: Chứa các công cụ dòng lệnh (CLI) và Package Manager Console (PMC) để thực hiện các thao tác EF Core như tạo và áp dụng migrations. Gói này thường được cài đặt dưới dạng `PackageReference` với `IncludeAssets="runtime; build; native; contentfiles; analyzers; buildtransitive"` và `PrivateAssets="all"` để đảm bảo nó chỉ là một công cụ phát triển và không được đóng gói vào ứng dụng cuối cùng.

**Các bước cài đặt:**

1.  Mở **Package Manager Console** trong Visual Studio (**Tools > NuGet Package Manager > Package Manager Console**).
2.  Đảm bảo dự án của bạn (NZWalks.API) được chọn trong danh sách thả xuống **Default project**.
3.  Chạy các lệnh sau:
    ```powershell
    Install-Package Microsoft.EntityFrameworkCore.SqlServer
    Install-Package Microsoft.EntityFrameworkCore.Tools
    ```
    Hoặc sử dụng giao diện **Manage NuGet Packages...** để tìm kiếm và cài đặt.

**Antigravity IDE và NuGet:** Antigravity có thể tự động phát hiện các phụ thuộc bị thiếu và đề xuất/thực hiện cài đặt NuGet. Với Vibe Coding, bạn có thể chỉ nói: "Thiết lập EF Core với SQL Server," và Antigravity sẽ tự động cài đặt các gói cần thiết vào đúng dự án.

### 3.3. Tạo Lớp `DbContext`

Lớp `DbContext` là thành phần trung tâm của EF Core, đóng vai trò là cầu nối giữa các mô hình miền của bạn và cơ sở dữ liệu.

**Cơ chế hoạt động của `DbContext`:**

*   **Đơn vị làm việc (Unit of Work):** Mỗi thể hiện của `DbContext` đại diện cho một "phiên làm việc" với cơ sở dữ liệu. Trong phiên này, nó theo dõi tất cả các đối tượng mà bạn tải, thêm, sửa hoặc xóa.
*   **Theo dõi thay đổi (Change Tracker):** Đây là một trong những tính năng mạnh mẽ nhất của EF Core. `DbContext` theo dõi trạng thái của mỗi thực thể (`Added`, `Modified`, `Deleted`, `Unchanged`). Khi bạn gọi `SaveChanges()`, nó sẽ tạo ra các câu lệnh SQL tương ứng để áp dụng các thay đổi này vào cơ sở dữ liệu.
*   **Cung cấp API truy vấn:** Thông qua các thuộc tính `DbSet`, nó cho phép bạn viết các truy vấn LINQ để truy xuất dữ liệu.

**Các bước tạo `DbContext`:**

1.  Trong thư mục gốc của dự án, tạo thư mục `Data`.
2.  Trong thư mục `Data`, tạo một lớp mới tên là `NZWalksDbContext.cs`.

#### 3.3.1. Cấu trúc của `NZWalksDbContext`

Lớp `NZWalksDbContext` sẽ kế thừa từ `Microsoft.EntityFrameworkCore.DbContext` và chứa các thuộc tính `DbSet` cho mỗi thực thể miền.

```csharp
using Microsoft.EntityFrameworkCore; // Cần thiết để kế thừa DbContext và sử dụng các phương thức mở rộng

namespace NZWalks.API.Data
{
    public class NZWalksDbContext : DbContext
    {
        // Constructor: Nhận DbContextOptions<NZWalksDbContext>.
        // Đây là cách tiêu chuẩn để cấu hình DbContext, cho phép chúng ta truyền
        // chuỗi kết nối và nhà cung cấp cơ sở dữ liệu (SQL Server) từ bên ngoài
        // (thường là từ Program.cs) thông qua Dependency Injection.
        public NZWalksDbContext(DbContextOptions<NZWalksDbContext> options) : base(options)
        {
        }

        // Các thuộc tính DbSet:
        // Mỗi DbSet<TEntity> đại diện cho một tập hợp các thực thể của kiểu TEntity
        // trong cơ sở dữ liệu, và sẽ ánh xạ tới một bảng trong lược đồ cơ sở dữ liệu.
        // Tên thuộc tính (ví dụ: Difficulties) sẽ thường là tên bảng trong DB.
        public DbSet<Models.Domain.Difficulty> Difficulties { get; set; }
        public DbSet<Models.Domain.Region> Regions { get; set; }
        public DbSet<Models.Domain.Walk> Walks { get; set; }

        // Mặc dù EF Core có thể tự động phát hiện các mối quan hệ dựa trên quy ước
        // (ví dụ: tên khóa ngoại và thuộc tính điều hướng),
        // bạn cũng có thể tùy chỉnh ánh xạ và mối quan hệ rõ ràng hơn ở đây
        // bằng cách ghi đè phương thức OnModelCreating.
        // Ví dụ:
        // protected override void OnModelCreating(ModelBuilder modelBuilder)
        // {
        //     base.OnModelCreating(modelBuilder);
        //
        //     // Cấu hình dữ liệu seed cho Difficulty (sẽ được tạo sẵn trong DB)
        //     var difficulties = new List<Models.Domain.Difficulty>()
        //     {
        //         new Models.Domain.Difficulty() { Id = Guid.Parse("54460f89-8d8a-4573-8b77-1d2a13f7c467"), Name = "Easy" },
        //         new Models.Domain.Difficulty() { Id = Guid.Parse("f070e137-b9c2-4809-9061-001099b2446a"), Name = "Medium" },
        //         new Models.Domain.Difficulty() { Id = Guid.Parse("c3848130-36a5-4811-9669-760f38446c7d"), Name = "Hard" }
        //     };
        //     modelBuilder.Entity<Models.Domain.Difficulty>().HasData(difficulties);
        // }
    }
}
```

**Dữ liệu Seed (Optional, nhưng hữu ích):**
Việc thêm dữ liệu khởi tạo (seed data) trực tiếp vào `OnModelCreating` là một cách tuyệt vời để đảm bảo rằng các bảng quan trọng (như `Difficulty` với các cấp độ "Easy", "Medium", "Hard") luôn có dữ liệu cần thiết ngay từ lần đầu tiên cơ sở dữ liệu được tạo hoặc cập nhật. Điều này giúp các phần sau của ứng dụng có dữ liệu để hoạt động ngay lập tức mà không cần nhập thủ công. Antigravity IDE có thể tự động tạo mã seed data dựa trên các yêu cầu ban đầu của bạn.

## 4. Cấu hình Chuỗi Kết nối Cơ sở dữ liệu

Để `DbContext` có thể kết nối với cơ sở dữ liệu SQL Server, nó cần một chuỗi kết nối (connection string). Chuỗi kết nối là một "tấm bản đồ" hướng dẫn cách thiết lập kết nối.

### 4.1. Vai trò của Chuỗi Kết nối và Cấu hình trong `appsettings.json`

Chuỗi kết nối là một chuỗi văn bản chứa các cặp khóa-giá trị mô tả chi tiết về máy chủ cơ sở dữ liệu, tên cơ sở dữ liệu, thông tin xác thực, v.v.

Trong ASP.NET Core, `appsettings.json` là nơi lý tưởng và được khuyến nghị để lưu trữ các cấu hình ứng dụng, bao gồm chuỗi kết nối.

Mở tệp `appsettings.json` trong dự án của bạn và thêm phần `ConnectionStrings`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  // Định nghĩa các chuỗi kết nối tại đây
  "ConnectionStrings": {
    // Tên chuỗi kết nối: NZWalksConnection
    "NZWalksConnection": "Server=localhost\\SQLEXPRESS;Database=NZWalksDb;Trusted_Connection=True;TrustServerCertificate=True"
  }
}
```

**Giải thích các thành phần của chuỗi kết nối:**

*   **`Server=localhost\\SQLEXPRESS`**: Địa chỉ máy chủ SQL Server.
    *   `localhost`: Chỉ ra rằng máy chủ nằm trên cùng máy tính.
    *   `\\SQLEXPRESS`: Tên của instance SQL Server Express (nếu bạn dùng phiên bản mặc định). Bạn cần thay thế bằng tên instance SQL Server thực tế của mình (ví dụ: `.` hoặc `(localdb)\\mssqllocaldb` cho LocalDB, hoặc tên máy chủ/IP nếu dùng SQL Server đầy đủ).
*   **`Database=NZWalksDb`**: Tên cơ sở dữ liệu mà EF Core sẽ kết nối hoặc tạo.
*   **`Trusted_Connection=True`**: Sử dụng xác thực Windows (tài khoản người dùng hiện tại của hệ điều hành). Nếu bạn dùng xác thực SQL Server, bạn sẽ thay thế bằng `User ID=YourUsername;Password=YourPassword`.
*   **`TrustServerCertificate=True`**: Kể từ .NET 6/7, tham số này thường cần thiết cho các kết nối cục bộ không có chứng chỉ SSL hợp lệ, để bỏ qua việc xác thực chứng chỉ. Trong môi trường sản phẩm, bạn nên sử dụng chứng chỉ hợp lệ và đặt `TrustServerCertificate=False`.

**Bảo mật Chuỗi Kết nối trong Môi trường Sản phẩm:**
Việc lưu trữ chuỗi kết nối trực tiếp trong `appsettings.json` là chấp nhận được cho môi trường phát triển. Tuy nhiên, trong môi trường sản phẩm (production), đây không phải là phương pháp bảo mật tốt nhất. Các phương pháp bảo mật hơn bao gồm:

*   **Biến môi trường (Environment Variables):** Được hệ điều hành quản lý và không lưu trữ trong mã nguồn.
*   **Azure Key Vault / AWS Secrets Manager:** Dịch vụ quản lý bí mật đám mây.
*   **ASP.NET Core Secret Manager:** Công cụ cục bộ cho phát triển, không được đưa vào production.

**Antigravity IDE và cấu hình:** Antigravity có thể tự động thêm chuỗi kết nối vào `appsettings.json`. Nếu bạn đang làm việc trong môi trường phát triển, nó sẽ sử dụng một chuỗi kết nối cục bộ mặc định. Nếu bạn chỉ định môi trường sản phẩm, Antigravity có thể hướng dẫn bạn cách cấu hình biến môi trường hoặc tích hợp với các dịch vụ quản lý bí mật.

## 5. Dependency Injection (DI) và Đăng ký `DbContext`

Dependency Injection (DI) là một trong những mẫu thiết kế quan trọng nhất và là cốt lõi của ASP.NET Core. Nó giúp ứng dụng của bạn trở nên linh hoạt, dễ kiểm thử và dễ mở rộng hơn.

### 5.1. Giới thiệu về Dependency Injection và Inversion of Control (IoC)

**Dependency Injection (DI):** Thay vì một lớp tự tạo hoặc tìm kiếm các đối tượng mà nó phụ thuộc (dependencies), các đối tượng đó sẽ được "tiêm" (injected) vào lớp đó thông qua hàm tạo (constructor), thuộc tính hoặc phương thức. Điều này giúp giảm sự ghép nối chặt chẽ (tight coupling) giữa các thành phần.

**Inversion of Control (IoC):** DI là một hình thức của IoC. IoC có nghĩa là "đảo ngược quyền điều khiển". Thay vì mã của bạn chủ động điều khiển quá trình tạo và quản lý các phụ thuộc, một vùng chứa IoC (IoC container) sẽ đảm nhiệm vai trò này.

**Lợi ích của DI:**

*   **Giảm ghép nối (Loose Coupling):** Các lớp không cần biết cách tạo ra các phụ thuộc của chúng, chỉ cần biết cách sử dụng chúng.
*   **Dễ kiểm thử (Testability):** Dễ dàng thay thế các phụ thuộc thực tế bằng các đối tượng giả lập (mocks) hoặc đối tượng giả (stubs) trong các bài kiểm thử đơn vị.
*   **Dễ bảo trì và mở rộng:** Dễ dàng thay đổi triển khai của một phụ thuộc mà không ảnh hưởng đến các lớp sử dụng nó.
*   **Quản lý vòng đời (Lifecycle Management):** Vùng chứa DI quản lý vòng đời của các dịch vụ (Singleton, Scoped, Transient).

Trong ASP.NET Core, DI được quản lý bởi một vùng chứa tích hợp (built-in IoC container) thông qua `IServiceCollection` và `IServiceProvider`.

### 5.2. Đăng ký `NZWalksDbContext` vào DI Container

Để `NZWalksDbContext` có thể được "tiêm" vào các Controller hoặc Repository, chúng ta cần đăng ký nó vào DI Container trong tệp `Program.cs`.

Mở tệp `Program.cs` và thêm đoạn mã sau vào trước `app.Run();`:

```csharp
using NZWalks.API.Data; // Cần thiết để tham chiếu NZWalksDbContext
using Microsoft.EntityFrameworkCore; // Cần thiết cho phương thức mở rộng UseSqlServer

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Đăng ký NZWalksDbContext vào DI Container
// AddDbContext là phương thức mở rộng được cung cấp bởi Microsoft.EntityFrameworkCore.
// Nó tự động đăng ký DbContext với vòng đời Scoped.
builder.Services.AddDbContext<NZWalksDbContext>(options =>
{
    // Cấu hình DbContext để sử dụng SQL Server.
    // Chuỗi kết nối được lấy từ phần "ConnectionStrings" trong appsettings.json
    // với khóa là "NZWalksConnection".
    options.UseSqlServer(builder.Configuration.GetConnectionString("NZWalksConnection"));
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

**Cơ chế hoạt động của `AddDbContext` và vòng đời Scoped:**
Khi bạn gọi `builder.Services.AddDbContext<TDbContext>(...)`, EF Core sẽ đăng ký `TDbContext` vào DI container với vòng đời **Scoped** theo mặc định.

*   **Scoped:** Có nghĩa là một thể hiện mới của `DbContext` sẽ được tạo ra cho *mỗi yêu cầu HTTP* và sẽ được hủy khi yêu cầu đó kết thúc. Đây là vòng đời được khuyến nghị cho `DbContext` trong các ứng dụng web vì nó:
    *   **Ngăn chặn rò rỉ bộ nhớ:** `DbContext` thường chứa bộ nhớ đệm (cache) và theo dõi các thực thể. Nếu nó là Singleton, bộ nhớ này sẽ không bao giờ được giải phóng và có thể gây rò rỉ.
    *   **Tránh các vấn đề đồng bộ hóa:** Mỗi yêu cầu HTTP hoạt động với một thể hiện `DbContext` riêng biệt, ngăn chặn các vấn đề về dữ liệu bị ghi đè hoặc xung đột do nhiều luồng cùng truy cập một `DbContext` duy nhất.
    *   **Đảm bảo tính nhất quán của dữ liệu:** Trong một yêu cầu duy nhất, tất cả các thao tác với cơ sở dữ liệu đều sử dụng cùng một thể hiện `DbContext`, đảm bảo tính nhất quán của dữ liệu trong phạm vi yêu cầu đó.

**Antigravity IDE và Dependency Injection:**
Antigravity IDE có thể tự động phân tích các phụ thuộc trong dự án của bạn và đề xuất/thực hiện việc đăng ký chúng vào DI container. Với Vibe Coding, bạn có thể chỉ thị: "Đăng ký `NZWalksDbContext` để sử dụng SQL Server với chuỗi kết nối 'NZWalksConnection'." Antigravity sẽ tìm đúng tệp `Program.cs`, thêm `using` statements cần thiết, và chèn đoạn mã `AddDbContext` một cách chính xác, đồng thời có thể giải thích lý do tại sao vòng đời `Scoped` là phù hợp nhất cho `DbContext`.

## 6. Quản lý Lược đồ Cơ sở dữ liệu với Migrations của EF Core

Sau khi đã định nghĩa các mô hình miền, tạo `DbContext` và cấu hình chuỗi kết nối, chúng ta cần tạo lược đồ cơ sở dữ liệu thực tế. EF Core Migrations là một công cụ mạnh mẽ cho phép bạn quản lý các thay đổi về lược đồ cơ sở dữ liệu theo thời gian một cách tự động và có kiểm soát.

### 6.1. Cơ chế hoạt động của EF Core Migrations

**EF Core Migrations:** Thay vì phải viết các câu lệnh SQL thủ công mỗi khi mô hình miền của bạn thay đổi (ví dụ: thêm một thuộc tính mới vào lớp `Walk`), EF Core sẽ tự động tạo ra các tệp migration chứa mã C# để thực hiện các thay đổi lược đồ cần thiết.

Quá trình migrations bao gồm hai bước chính:

1.  **Tạo Migration (`Add-Migration`):** EF Core so sánh trạng thái hiện tại của các mô hình miền (`DbSet` trong `DbContext`) với trạng thái lược đồ cơ sở dữ liệu được ghi lại lần cuối (được lưu trong tệp Snapshot của migration trước đó, hoặc là một lược đồ trống nếu là lần đầu tiên). Dựa trên sự khác biệt, nó tạo một tệp migration mới chứa các thay đổi cần thiết.
2.  **Áp dụng Migration (`Update-Database`):** Lệnh này sẽ thực thi mã C# trong tệp migration để tạo hoặc cập nhật lược đồ cơ sở dữ liệu thực tế. EF Core cũng duy trì một bảng đặc biệt (`__EFMigrationsHistory`) trong cơ sở dữ liệu để theo dõi những migration nào đã được áp dụng.

### 6.2. Các Bước Thực hiện Migrations

Bạn sẽ sử dụng Package Manager Console (PMC) trong Visual Studio để chạy các lệnh migration.

1.  **Mở Package Manager Console:** Trong Visual Studio, đi tới **Tools > NuGet Package Manager > Package Manager Console**.
2.  Đảm bảo rằng dự án của bạn (NZWalks.API) được chọn trong danh sách thả xuống **Default project**.

#### 6.2.1. Thêm Migration mới (`Add-Migration`)

Lệnh này sẽ tạo một tệp migration mới dựa trên các mô hình miền hiện tại của bạn.

```powershell
Add-Migration "InitialMigration" -Project NZWalks.API -StartupProject NZWalks.API
```

*   **`Add-Migration`**: Lệnh để tạo một migration mới.
*   **`"InitialMigration"`**: Tên của migration. Luôn đặt tên có ý nghĩa (ví dụ: "AddUsersTable", "UpdateProductPrice") để dễ dàng theo dõi lịch sử thay đổi.
*   **`-Project NZWalks.API`**: Chỉ định dự án chứa `DbContext` và các mô hình miền.
*   **`-StartupProject NZWalks.API`**: Chỉ định dự án khởi động (nơi chứa `Program.cs` và cấu hình DI) để EF Core biết cách xây dựng dịch vụ và lấy chuỗi kết nối. Trong trường hợp này, cả hai là cùng một dự án.

Sau khi chạy lệnh này, EF Core sẽ:

*   Tạo một thư mục `Migrations` trong dự án `NZWalks.API`.
*   Trong thư mục `Migrations`, tạo hai tệp:
    *   Một tệp C# (ví dụ: `20231027100000_InitialMigration.cs`): Chứa mã C# với hai phương thức:
        *   `Up()`: Định nghĩa các thay đổi lược đồ sẽ được áp dụng (ví dụ: tạo bảng `Difficulties`, `Regions`, `Walks`, thêm cột, khóa chính, khóa ngoại).
        *   `Down()`: Định nghĩa cách hoàn tác các thay đổi trong `Up()` (ví dụ: xóa bảng), hữu ích khi bạn muốn quay lại phiên bản lược đồ trước đó.
    *   Một tệp Snapshot (ví dụ: `NZWalksDbContextModelSnapshot.cs`): Lưu trữ một "ảnh chụp" của toàn bộ mô hình miền của bạn tại thời điểm migration được tạo. Tệp này được EF Core sử dụng để so sánh với các thay đổi mô hình trong tương lai và tạo ra các migration tiếp theo.

#### 6.2.2. Cập nhật Cơ sở dữ liệu (`Update-Database`)

Sau khi đã tạo tệp migration, bạn cần áp dụng nó vào cơ sở dữ liệu thực tế.

```powershell
Update-Database -Project NZWalks.API -StartupProject NZWalks.API
```

*   **`Update-Database`**: Lệnh này sẽ áp dụng tất cả các migration đang chờ xử lý vào cơ sở dữ liệu được chỉ định trong chuỗi kết nối của bạn.

Sau khi chạy lệnh này:

*   EF Core sẽ kết nối với SQL Server bằng chuỗi kết nối `NZWalksConnection` từ `appsettings.json`.
*   Nếu cơ sở dữ liệu `NZWalksDb` chưa tồn tại, nó sẽ được tạo.
*   Các bảng `Difficulties`, `Regions`, `Walks` sẽ được tạo cùng với các cột, khóa chính và khóa ngoại đã định nghĩa trong `InitialMigration.cs`.
*   EF Core cũng sẽ tạo một bảng `__EFMigrationsHistory` trong cơ sở dữ liệu để theo dõi các migration đã được áp dụng, đảm bảo rằng mỗi migration chỉ được chạy một lần.

**Kiểm tra kết quả:** Để xác nhận, bạn có thể mở SQL Server Management Studio (SSMS) hoặc Azure Data Studio, kết nối đến máy chủ SQL Server của bạn, làm mới danh sách cơ sở dữ liệu và kiểm tra `NZWalksDb` cùng với các bảng bên trong nó.

**Antigravity IDE và Migrations:**
Antigravity IDE có khả năng tự động thực hiện toàn bộ quy trình migration. Bạn có thể nói: "Tạo và áp dụng migration ban đầu cho cơ sở dữ liệu NZWalks." Antigravity sẽ:

1.  Chạy `Add-Migration "InitialMigration"`.
2.  Hiển thị tệp migration được tạo ra, cho phép bạn xem xét các thay đổi SQL tiềm năng.
3.  Khi bạn xác nhận, nó sẽ chạy `Update-Database`.
4.  Sau đó, nó có thể mở SSMS (nếu được cấu hình) hoặc thực hiện truy vấn để xác minh rằng các bảng đã được tạo thành công.
Điều này thể hiện sức mạnh của Antigravity trong việc tự động hóa các tác vụ lặp đi lặp lại và đảm bảo quy trình được thực hiện chính xác theo ý định của bạn.

---

## Tóm tắt Phần 3

Trong Phần 3, chúng ta đã xây dựng nền tảng vững chắc cho việc tương tác với cơ sở dữ liệu trong ứng dụng ASP.NET Core API của mình, đồng thời khám phá sâu sắc các cơ chế ngầm và cách tư duy Vibe Coding có thể được áp dụng:

*   **Định hình Miền Ứng dụng & DDD:** Chúng ta đã xác định các thực thể chính (`Walk`, `Region`, `Difficulty`) và mối quan hệ giữa chúng, áp dụng các nguyên tắc Domain-Driven Design để tạo ra một mô hình nghiệp vụ rõ ràng, và hình dung cách Antigravity IDE có thể hỗ trợ giai đoạn phân tích này.
*   **Xây dựng Mô hình Miền trong C#:** Các thực thể này đã được chuyển đổi thành các lớp C# (`Difficulty.cs`, `Region.cs`, `Walk.cs`) trong thư mục `Models/Domain`, sử dụng `Guid` cho ID, `string.Empty` cho các thuộc tính không nullable, `string?` cho nullable reference types, và `null!` cho navigation properties, đồng thời hiểu được vai trò của navigation properties trong việc ánh xạ mối quan hệ.
*   **Tích hợp Entity Framework Core:** Chúng ta đã cài đặt các gói NuGet cần thiết (`Microsoft.EntityFrameworkCore.SqlServer`, `Microsoft.EntityFrameworkCore.Tools`) và tìm hiểu về cơ chế hoạt động của một ORM như EF Core.
*   **Tạo Lớp `DbContext`:** Lớp `NZWalksDbContext` đã được tạo, kế thừa từ `DbContext` và chứa các `DbSet` cho mỗi thực thể, đóng vai trò là đơn vị làm việc và cầu nối giữa ứng dụng và cơ sở dữ liệu.
*   **Cấu hình Chuỗi Kết nối:** Chuỗi kết nối đến SQL Server đã được định nghĩa trong `appsettings.json`, cung cấp tất cả thông tin cần thiết để `DbContext` thiết lập kết nối, cùng với lưu ý về bảo mật trong môi trường production.
*   **Dependency Injection và Đăng ký `DbContext`:** Chúng ta đã hiểu về Dependency Injection, các lợi ích của nó, và cách đăng ký `NZWalksDbContext` vào DI Container trong `Program.cs` với vòng đời `Scoped` – một lựa chọn tối ưu cho các ứng dụng web.
*   **Quản lý Lược đồ với Migrations:** Cuối cùng, chúng ta đã sử dụng EF Core Migrations (`Add-Migration` và `Update-Database`) để tự động tạo cơ sở dữ liệu và lược đồ bảng dựa trên các mô hình miền đã định nghĩa, sẵn sàng cho các thao tác dữ liệu, và hình dung cách Antigravity IDE có thể tự động hóa toàn bộ quy trình này.

Phần này đã trang bị cho bạn kiến thức nền tảng vững chắc để ứng dụng có thể kết nối và quản lý dữ liệu. Trong các phần tiếp theo, chúng ta sẽ dựa trên nền tảng này để triển khai Repository Pattern, xây dựng các Controller và xử lý các HTTP Verbs (GET, POST, PUT, DELETE) để thực hiện các hoạt động CRUD trên các thực thể của chúng ta một cách có cấu trúc và hiệu quả.

<!-- REVIEWED_BY_AGENT -->
