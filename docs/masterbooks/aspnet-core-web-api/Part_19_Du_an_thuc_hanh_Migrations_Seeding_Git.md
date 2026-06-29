# Phần 19: Quản lý Lược đồ, Khởi tạo Dữ liệu và Kiểm soát Phiên bản Mã nguồn

Trong hành trình xây dựng RESTful Web API với ASP.NET Core và Entity Framework Core, việc quản lý cơ sở dữ liệu và mã nguồn một cách có hệ thống là yếu tố then chốt cho sự thành công và tính bền vững của dự án. Chương này sẽ đưa bạn đi sâu vào ba khía cạnh thiết yếu: quản lý lược đồ cơ sở dữ liệu thông qua EF Core Migrations, khởi tạo dữ liệu ban đầu (seeding) và kiểm soát phiên bản toàn bộ dự án bằng Git. Chúng ta sẽ không chỉ tìm hiểu cách thực hiện các tác vụ này mà còn khám phá cơ chế hoạt động ngầm và cách tích hợp các công cụ hiện đại, bao gồm cả tư duy "Vibe Coding" với sự hỗ trợ của Antigravity IDE.

Mục tiêu chính của chương này là trang bị cho bạn năng lực để:

*   **Quản lý Lược đồ Cơ sở dữ liệu:** Tạo, cập nhật và hoàn nguyên lược đồ cơ sở dữ liệu một cách có kiểm soát, đồng bộ với mô hình dữ liệu trong mã nguồn C#.
*   **Khởi tạo Dữ liệu (Seeding):** Cung cấp dữ liệu mẫu hoặc dữ liệu cần thiết cho ứng dụng hoạt động, phục vụ cho mục đích phát triển, kiểm thử và triển khai.
*   **Kiểm soát Phiên bản Mã nguồn:** Sử dụng Git để theo dõi lịch sử thay đổi, cộng tác hiệu quả và đảm bảo khả năng khôi phục mã nguồn.

Hãy cùng đi sâu vào từng chủ đề, đồng thời khám phá cách một môi trường phát triển hiện đại như Antigravity IDE có thể nâng cao trải nghiệm của bạn.

## I. Quản lý Lược đồ Cơ sở dữ liệu với Entity Framework Core Migrations

Entity Framework Core (EF Core) Migrations là một tính năng mạnh mẽ cho phép bạn quản lý sự tiến hóa của lược đồ cơ sở dữ liệu theo thời gian. Thay vì phải viết các tập lệnh SQL thủ công phức tạp để tạo hoặc thay đổi bảng, EF Core Migrations cho phép bạn định nghĩa cấu trúc cơ sở dữ liệu thông qua các lớp mô hình (entity classes) trong C# và sau đó tự động tạo các tập lệnh thay đổi lược đồ tương ứng.

### 1. Tầm quan trọng và Cơ chế hoạt động của EF Core Migrations

**Tại sao cần Migrations?**
Trong quá trình phát triển ứng dụng, mô hình dữ liệu (các lớp entity) thường xuyên thay đổi. Việc duy trì sự đồng bộ giữa mô hình dữ liệu trong mã nguồn C# và lược đồ cơ sở dữ liệu thực tế là một thách thức. Migrations giải quyết vấn đề này bằng cách:

*   **Tự động hóa:** Tự động tạo các tập lệnh SQL cần thiết để tạo, sửa đổi hoặc xóa bảng, cột, khóa, chỉ mục, v.v., dựa trên sự khác biệt giữa mô hình hiện tại và mô hình đã được áp dụng.
*   **Kiểm soát phiên bản:** Mỗi migration là một "bản vá" (patch) cho lược đồ cơ sở dữ liệu, có thể được áp dụng hoặc hoàn nguyên. Lịch sử các migration được lưu trữ, cho phép bạn dễ dàng theo dõi sự phát triển của lược đồ.
*   **Cộng tác:** Giúp các thành viên trong nhóm làm việc trên cùng một cơ sở dữ liệu mà không lo xung đột lược đồ, vì mỗi thay đổi được gói gọn trong một migration riêng biệt.

**Cơ chế hoạt động ngầm (Under the Hood):**
Khi bạn tạo một migration, EF Core thực hiện các bước sau:

1.  **So sánh Mô hình:** EF Core so sánh trạng thái hiện tại của `DbContext` (các `DbSet`, cấu hình `OnModelCreating`, Data Annotations) với một "snapshot" (ảnh chụp nhanh) của mô hình đã được áp dụng trong migration trước đó.
2.  **Tạo Tệp Migration:** Dựa trên sự khác biệt, EF Core tạo ra một tệp C# trong thư mục `Migrations`. Tệp này chứa hai phương thức chính:
    *   `Up()`: Chứa các lệnh (ví dụ: `CreateTable`, `AddColumn`, `DropTable`) để áp dụng các thay đổi lược đồ lên cơ sở dữ liệu.
    *   `Down()`: Chứa các lệnh để hoàn tác các thay đổi được thực hiện bởi `Up()`, hữu ích khi bạn cần hoàn nguyên một migration.
3.  **Cập nhật Snapshot:** EF Core cũng tạo một tệp `[Timestamp]_MigrationName.Designer.cs` và cập nhật tệp `[DbContextModelSnapshot].cs` chứa một biểu diễn của mô hình hiện tại dưới dạng mã C#. Snapshot này được sử dụng cho các migration tiếp theo để so sánh và phát hiện thay đổi.
4.  **Bảng `__EFMigrationsHistory`:** Khi bạn áp dụng migration vào cơ sở dữ liệu bằng `Update-Database`, EF Core tạo một bảng đặc biệt có tên `__EFMigrationsHistory`. Bảng này ghi lại tất cả các migration đã được áp dụng thành công vào cơ sở dữ liệu, giúp EF Core biết trạng thái hiện tại của lược đồ và những migration nào cần được áp dụng tiếp theo.

### 2. Chuẩn bị Môi trường và Cấu hình DbContext

Trước khi có thể tạo migration, bạn cần đảm bảo rằng dự án ASP.NET Core của bạn đã được cấu hình đúng cách với EF Core.

1.  **Cài đặt các gói NuGet cần thiết:**
    *   `Microsoft.EntityFrameworkCore.SqlServer` (hoặc nhà cung cấp DB khác như PostgreSQL, MySQL, SQLite)
    *   `Microsoft.EntityFrameworkCore.Tools` (cho các lệnh Migration trong Package Manager Console)
    *   `Microsoft.EntityFrameworkCore.Design` (đôi khi được yêu cầu bởi Tools)

2.  **Cấu hình Chuỗi kết nối (Connection String):**
    Chuỗi kết nối định nghĩa cách ứng dụng của bạn kết nối đến cơ sở dữ liệu. Nó thường được lưu trữ trong tệp `appsettings.json`.

    **Ví dụ: `appsettings.json`**
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
        "NewZealandWalksConnectionString": "Server=DOT;Database=NewZealandWalks.DB;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=true"
      }
    }
    ```
    *   `Server=DOT`: Kết nối đến SQL Server cục bộ. Bạn có thể thay thế bằng `localhost`, `(localdb)\\mssqllocaldb` hoặc tên máy chủ SQL Server của bạn.
    *   `Database=NewZealandWalks.DB`: Tên cơ sở dữ liệu sẽ được tạo hoặc kết nối tới.
    *   `Trusted_Connection=True`: Sử dụng xác thực Windows. Nếu dùng SQL Server Authentication, bạn sẽ cần `User ID=...;Password=...`.
    *   `TrustServerCertificate=True`: Bỏ qua việc kiểm tra chứng chỉ SSL, thường dùng trong môi trường phát triển.
    *   `MultipleActiveResultSets=true`: Đôi khi cần thiết cho các hoạt động EF Core nhất định.

3.  **Định nghĩa `DbContext` và các lớp Entity:**
    `DbContext` là cầu nối giữa các lớp entity của bạn và cơ sở dữ liệu. Mỗi `DbSet<TEntity>` trong `DbContext` tương ứng với một bảng trong cơ sở dữ liệu. Các lớp entity (ví dụ: `Region`, `WalkDifficulty`, `Walk`) định nghĩa cấu trúc của dữ liệu.

    **Ví dụ: `NewZealandWalksDbContext.cs`**
    ```csharp
    using Microsoft.EntityFrameworkCore;
    using NewZealandWalks.API.Models.Domain; // Giả sử các entity của bạn nằm trong namespace này

    namespace NewZealandWalks.API.Data
    {
        public class NewZealandWalksDbContext : DbContext
        {
            public NewZealandWalksDbContext(DbContextOptions<NewZealandWalksDbContext> options) : base(options)
            {
            }

            // Định nghĩa các DbSet cho các bảng trong cơ sở dữ liệu
            public DbSet<Region> Regions { get; set; }
            public DbSet<WalkDifficulty> WalkDifficulties { get; set; }
            public DbSet<Walk> Walks { get; set; }

            // Override OnModelCreating để cấu hình thêm hoặc seeding dữ liệu
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                base.OnModelCreating(modelBuilder);
                // Các cấu hình Fluent API hoặc seeding dữ liệu sẽ được thêm ở đây
            }
        }
    }
    ```

4.  **Đăng ký `DbContext` vào Hệ thống Dependency Injection:**
    Trong ASP.NET Core, `DbContext` được đăng ký vào container DI trong `Program.cs` (hoặc `Startup.cs`). Điều này cho phép các controller hoặc service khác yêu cầu một thể hiện của `DbContext` và được cung cấp tự động.

    **Ví dụ: `Program.cs`**
    ```csharp
    using Microsoft.EntityFrameworkCore;
    using NewZealandWalks.API.Data;

    var builder = WebApplication.CreateBuilder(args);

    // Add services to the container.
    builder.Services.AddControllers();
    builder.Services.AddEndpointsApiExplorer();
    builder.Services.AddSwaggerGen();

    // Cấu hình DbContext với chuỗi kết nối
    builder.Services.AddDbContext<NewZealandWalksDbContext>(options =>
    {
        options.UseSqlServer(builder.Configuration.GetConnectionString("NewZealandWalksConnectionString"));
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

### 3. Tạo Migration Ban đầu

Khi `DbContext` và các lớp entity đã được định nghĩa, bước đầu tiên là tạo một migration ban đầu để phản ánh cấu trúc cơ sở dữ liệu hiện tại.

1.  **Mở Package Manager Console (PMC):** Trong Visual Studio, điều hướng đến `Tools` > `NuGet Package Manager` > `Package Manager Console`.
2.  **Chọn Dự án Mặc định:** Đảm bảo rằng dự án chứa `DbContext` của bạn (ví dụ: `NewZealandWalks.API`) được chọn trong danh sách thả xuống "Default project" của Package Manager Console.
3.  **Chạy lệnh `Add-Migration`:**
    ```bash
    Add-Migration InitialMigration
    ```
    *   `InitialMigration` là tên bạn đặt cho migration này. Hãy chọn tên có ý nghĩa, mô tả ngắn gọn nội dung của migration (ví dụ: `AddUsersTable`, `UpdateProductNameLength`).
    *   Lệnh này sẽ quét `DbContext` của bạn, so sánh nó với trạng thái migration cuối cùng (hoặc trạng thái trống nếu là lần đầu tiên) và tạo các tệp migration cần thiết.

    Sau khi chạy lệnh, một thư mục `Migrations` sẽ được tạo trong dự án của bạn (nếu chưa có). Bên trong thư mục này, bạn sẽ thấy các tệp:

    *   `[Timestamp]_InitialMigration.cs`: Chứa các phương thức `Up()` và `Down()` với các lệnh SQL DDL (Data Definition Language) tương ứng.
    *   `[Timestamp]_InitialMigration.Designer.cs`: Chứa siêu dữ liệu về migration.
    *   `NewZealandWalksDbContextModelSnapshot.cs`: Một bản chụp nhanh của mô hình `DbContext` hiện tại, được EF Core sử dụng để so sánh trong các migration tiếp theo.

    **Ví dụ về nội dung tệp `[Timestamp]_InitialMigration.cs` (rút gọn):**
    ```csharp
    using Microsoft.EntityFrameworkCore.Migrations;

    #nullable disable

    namespace NewZealandWalks.API.Migrations
    {
        /// <inheritdoc />
        public partial class InitialMigration : Migration
        {
            /// <inheritdoc />
            protected override void Up(MigrationBuilder migrationBuilder)
            {
                migrationBuilder.CreateTable(
                    name: "Regions",
                    columns: table => new
                    {
                        Id = table.Column<Guid>(type: "uniqueidentifier", nullable: false),
                        Name = table.Column<string>(type: "nvarchar(max)", nullable: false),
                        Code = table.Column<string>(type: "nvarchar(max)", nullable: false),
                        RegionImageUrl = table.Column<string>(type: "nvarchar(max)", nullable: true)
                    },
                    constraints: table =>
                    {
                        table.PrimaryKey("PK_Regions", x => x.Id);
                    });

                // ... các lệnh tạo bảng khác cho WalkDifficulties và Walks
            }

            /// <inheritdoc />
            protected override void Down(MigrationBuilder migrationBuilder)
            {
                migrationBuilder.DropTable(
                    name: "Regions");

                // ... các lệnh xóa bảng khác
            }
        }
    }
    ```

### 4. Cập nhật Cơ sở dữ liệu

Việc tạo migration chỉ là bước chuẩn bị. Để áp dụng các thay đổi lược đồ vào cơ sở dữ liệu thực tế, bạn cần chạy lệnh `Update-Database`.

1.  **Mở Package Manager Console (PMC).**
2.  **Chạy lệnh `Update-Database`:**
    ```bash
    Update-Database
    ```
    *   Lệnh này sẽ tìm tất cả các migration chưa được áp dụng (bằng cách kiểm tra bảng `__EFMigrationsHistory`) và thực thi phương thức `Up()` của chúng theo thứ tự.
    *   Nếu cơ sở dữ liệu chưa tồn tại, nó sẽ được tạo với tên đã chỉ định trong chuỗi kết nối (ví dụ: `NewZealandWalks.DB`).
    *   Các bảng được định nghĩa trong `DbContext` của bạn (ví dụ: `Regions`, `WalkDifficulties`, `Walks`) sẽ được tạo.
    *   Bảng `__EFMigrationsHistory` cũng sẽ được tạo và cập nhật với thông tin về migration vừa được áp dụng.

    > [!NOTE]
    > Sau khi chạy `Update-Database`, bạn có thể sử dụng SQL Server Management Studio (SSMS) hoặc các công cụ quản lý cơ sở dữ liệu khác để kiểm tra. Bạn sẽ thấy cơ sở dữ liệu mới và các bảng đã được tạo. Hiện tại, các bảng này sẽ trống rỗng vì chúng ta chưa seeding dữ liệu.

### 5. Quản lý Migrations Nâng cao

*   **Hoàn nguyên Migration (Rollback):**
    Nếu bạn muốn quay lại trạng thái cơ sở dữ liệu trước một migration cụ thể, bạn có thể sử dụng `Update-Database` với tên migration (hoặc `0` để hoàn nguyên tất cả).
    ```bash
    Update-Database <PreviousMigrationName>
    # Hoặc để hoàn nguyên migration cuối cùng
    Update-Database -Migration:0
    ```
    Khi hoàn nguyên, EF Core sẽ thực thi phương thức `Down()` của các migration theo thứ tự ngược lại.

*   **Xóa Migration:**
    Nếu bạn tạo một migration và chưa áp dụng nó, bạn có thể xóa nó bằng lệnh `Remove-Migration`.
    ```bash
    Remove-Migration
    ```
    Lệnh này sẽ xóa migration cuối cùng chưa được áp dụng và hoàn nguyên tệp snapshot.

*   **Tạo Script SQL từ Migrations:**
    Trong môi trường sản xuất, bạn có thể không muốn chạy `Update-Database` trực tiếp. Thay vào đó, bạn có thể tạo một tập lệnh SQL từ các migration và tự mình thực thi nó.
    ```bash
    Script-Migration -From <PreviousMigrationName> -To <TargetMigrationName> -Output <FilePath.sql>
    # Hoặc tạo script cho tất cả các migration chưa được áp dụng
    Script-Migration -Output C:\Migrations\script.sql
    ```

### Antigravity IDE và Migrations: Nâng tầm Vibe Coding

Antigravity IDE, với khả năng Agentic AI siêu việt, có thể biến việc quản lý EF Core Migrations từ một tác vụ thủ công thành một trải nghiệm "Vibe Coding" mượt mà và trực quan.

*   **Nhận diện Thay đổi Mô hình và Đề xuất Migration:** Khi bạn thay đổi một lớp entity (thêm thuộc tính, đổi kiểu dữ liệu) hoặc `DbContext` (thêm `DbSet`), Antigravity IDE không chỉ phát hiện mà còn *tự động đề xuất* việc tạo một migration mới. Nó có thể phân tích các thay đổi và gợi ý một tên migration có ý nghĩa ngay lập tức (ví dụ: "AddUserEmailAndRenameAddressColumn"), giúp bạn duy trì dòng chảy tư duy mà không cần dừng lại để nghĩ tên. Đây chính là Vibe Coding: AI hiểu *ý định* của bạn và chuẩn bị sẵn công cụ.
*   **Xem trước Lược đồ và Cảnh báo Tự động:** Trước khi bạn chạy `Update-Database`, Antigravity có thể *trực quan hóa* các thay đổi lược đồ mà migration sẽ thực hiện. Nó có thể hiển thị một bảng so sánh "trước và sau", thậm chí *cảnh báo* về các thay đổi tiềm ẩn gây mất dữ liệu (ví dụ: giảm độ dài cột `string`, thay đổi kiểu dữ liệu không tương thích) và đề xuất các giải pháp hoặc xác nhận từ bạn.
*   **Tự động tạo và thực thi lệnh:** Thay vì gõ lệnh trong PMC, bạn có thể chỉ cần "ra lệnh" cho Antigravity (qua giọng nói hoặc văn bản ngắn gọn) như "tạo migration mới" hoặc "cập nhật database". Antigravity sẽ tự động chạy các lệnh tương ứng, theo dõi tiến trình và báo cáo kết quả.
*   **Quản lý lịch sử Migration thông minh:** Antigravity có thể cung cấp một giao diện trực quan để duyệt qua lịch sử các migration, cho phép bạn dễ dàng hoàn nguyên về một trạng thái cụ thể hoặc tạo các tập lệnh SQL với một cú nhấp chuột.

Với Antigravity, bạn không chỉ gõ lệnh; bạn *cảm nhận* sự tiến hóa của cơ sở dữ liệu, và AI làm phần việc nặng nhọc, cho phép bạn tập trung vào thiết kế mô hình dữ liệu và logic nghiệp vụ.

## II. Khởi tạo Dữ liệu (Seeding) cho Cơ sở dữ liệu

Sau khi cơ sở dữ liệu và các bảng đã được tạo thông qua migrations, bước tiếp theo thường là khởi tạo một số dữ liệu ban đầu. Quá trình này được gọi là "seeding" dữ liệu.

### 1. Mục đích và Các loại Dữ liệu Seeding

**Mục đích của Seeding:**

*   **Dữ liệu Tham chiếu/Cố định:** Cung cấp các giá trị cần thiết cho ứng dụng hoạt động (ví dụ: danh sách các vai trò người dùng, trạng thái đơn hàng, danh mục sản phẩm).
*   **Dữ liệu Mẫu (Sample Data):** Tạo dữ liệu giả định để phát triển và kiểm thử giao diện người dùng, logic nghiệp vụ mà không cần nhập thủ công.
*   **Dữ liệu Mặc định:** Thiết lập các cấu hình hoặc giá trị mặc định cho ứng dụng khi triển khai lần đầu.

**Các loại Dữ liệu Seeding:**

*   **Dữ liệu tĩnh, cố định:** Thường là các lookup table (bảng tra cứu) có giá trị ít thay đổi.
*   **Dữ liệu phát triển/kiểm thử:** Dữ liệu có thể được tạo ngẫu nhiên hoặc có cấu trúc để mô phỏng dữ liệu thực.
*   **Dữ liệu khởi tạo hệ thống:** Dữ liệu cần thiết để khởi chạy các chức năng cốt lõi của ứng dụng.

### 2. Phương pháp Seeding truyền thống: SQL Script

Đây là phương pháp đơn giản và trực tiếp nhất, đặc biệt phù hợp cho các dự án nhỏ hoặc khi bạn cần kiểm soát chính xác từng câu lệnh SQL. Bạn chuẩn bị một tập lệnh SQL chứa các câu lệnh `INSERT` và thực thi chúng trực tiếp trên cơ sở dữ liệu.

**Ví dụ: Tập lệnh SQL để Seeding dữ liệu**

```sql
-- Sử dụng cơ sở dữ liệu của bạn
USE NewZealandWalks.DB;
GO

-- Kiểm tra và chèn dữ liệu vào bảng Regions nếu chưa tồn tại
IF NOT EXISTS (SELECT 1 FROM Regions WHERE Code = 'AKL')
BEGIN
    INSERT INTO Regions (Id, Name, Code, RegionImageUrl) VALUES
    (NEWID(), 'Auckland', 'AKL', 'https://example.com/auckland.jpg');
END
IF NOT EXISTS (SELECT 1 FROM Regions WHERE Code = 'WLG')
BEGIN
    INSERT INTO Regions (Id, Name, Code, RegionImageUrl) VALUES
    (NEWID(), 'Wellington', 'WLG', 'https://example.com/wellington.jpg');
END
-- Thêm các vùng khác tương tự

-- Kiểm tra và chèn dữ liệu vào bảng WalkDifficulties nếu chưa tồn tại
IF NOT EXISTS (SELECT 1 FROM WalkDifficulties WHERE Code = 'Easy')
BEGIN
    INSERT INTO WalkDifficulties (Id, Code) VALUES
    (NEWID(), 'Easy');
END
IF NOT EXISTS (SELECT 1 FROM WalkDifficulties WHERE Code = 'Medium')
BEGIN
    INSERT INTO WalkDifficulties (Id, Code) VALUES
    (NEWID(), 'Medium');
END
IF NOT EXISTS (SELECT 1 FROM WalkDifficulties WHERE Code = 'Hard')
BEGIN
    INSERT INTO WalkDifficulties (Id, Code) VALUES
    (NEWID(), 'Hard');
END
GO

-- Chèn dữ liệu vào bảng Walks (ví dụ, cần lấy Id từ Regions và WalkDifficulties đã chèn)
-- Sử dụng biến để lưu trữ GUID đã chèn hoặc đã tồn tại
DECLARE @AucklandId UNIQUEIDENTIFIER;
SELECT @AucklandId = Id FROM Regions WHERE Code = 'AKL';

DECLARE @EasyId UNIQUEIDENTIFIER;
SELECT @EasyId = Id FROM WalkDifficulties WHERE Code = 'Easy';

IF NOT EXISTS (SELECT 1 FROM Walks WHERE Name = 'Mount Eden Summit Walk') AND @AucklandId IS NOT NULL AND @EasyId IS NOT NULL
BEGIN
    INSERT INTO Walks (Id, Name, Description, LengthInKm, WalkImageUrl, RegionId, WalkDifficultyId) VALUES
    (NEWID(), 'Mount Eden Summit Walk', 'A scenic walk to the summit of Mount Eden.', 2.5, 'https://example.com/mt_eden.jpg', @AucklandId, @EasyId);
END
GO
```

**Cách thực hiện:**

1.  Lưu tập lệnh SQL này vào một tệp (ví dụ: `SeedData.sql`).
2.  Mở SQL Server Management Studio (SSMS), Azure Data Studio hoặc một công cụ quản lý cơ sở dữ liệu tương tự.
3.  Kết nối đến máy chủ SQL của bạn.
4.  Mở tệp `SeedData.sql` và thực thi nó trên cơ sở dữ liệu `NewZealandWalks.DB`.

> [!WARNING]
> **Nhược điểm của SQL Script thủ công:**
> *   **Không tích hợp với Migrations:** Quá trình seeding hoàn toàn tách rời khỏi luồng EF Core Migrations. Nếu bạn hoàn nguyên cơ sở dữ liệu, bạn phải chạy lại script seeding thủ công.
> *   **Khó quản lý:** Với các dự án lớn, việc duy trì nhiều script SQL có thể trở nên phức tạp.
> *   **Phụ thuộc vào DB:** Script SQL có thể không tương thích hoàn toàn giữa các hệ quản trị cơ sở dữ liệu khác nhau.

### 3. Phương pháp Seeding tích hợp: EF Core `HasData()`

EF Core cung cấp một cách tích hợp hơn để seeding dữ liệu thông qua phương thức `HasData()` trong phương thức `OnModelCreating` của `DbContext`. Đây là phương pháp được khuyến nghị vì nó gắn liền với các migration, giúp dữ liệu seeding được áp dụng tự động mỗi khi bạn chạy `Update-Database` (sau khi tạo migration mới có chứa dữ liệu seeding).

**Cơ chế hoạt động của `HasData()`:**
Khi bạn thêm `HasData()` vào `OnModelCreating` và tạo một migration mới, EF Core sẽ phát hiện các đối tượng dữ liệu bạn đã cung cấp và tạo ra các lệnh `InsertData` hoặc `UpdateData` (nếu bạn thay đổi dữ liệu seeding) trong phương thức `Up()` của migration. Khi bạn chạy `Update-Database`, các lệnh này sẽ được thực thi. EF Core sử dụng khóa chính của các entity để theo dõi và cập nhật dữ liệu seeding. Do đó, việc cung cấp khóa chính (ví dụ: `Guid`) là bắt buộc.

**Ví dụ: Seeding dữ liệu trong `NewZealandWalksDbContext.cs`**

```csharp
using Microsoft.EntityFrameworkCore;
using NewZealandWalks.API.Models.Domain;
using System; // Cần thiết cho Guid
using System.Collections.Generic; // Cần thiết cho List

namespace NewZealandWalks.API.Data
{
    public class NewZealandWalksDbContext : DbContext
    {
        public NewZealandWalksDbContext(DbContextOptions<NewZealandWalksDbContext> options) : base(options)
        {
        }

        public DbSet<Region> Regions { get; set; }
        public DbSet<WalkDifficulty> WalkDifficulties { get; set; }
        public DbSet<Walk> Walks { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // --- Seeding Regions ---
            // Cần cung cấp các Id duy nhất cho mỗi đối tượng để EF Core có thể theo dõi và cập nhật
            var regions = new List<Region>
            {
                new Region
                {
                    Id = Guid.Parse("f7248fc3-2587-4921-8d57-b2eac599f2ae"),
                    Name = "Auckland",
                    Code = "AKL",
                    RegionImageUrl = "https://images.pexels.com/photos/5169056/pexels-photo-5169056.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1"
                },
                new Region
                {
                    Id = Guid.Parse("6884f7d7-ad1f-4101-8df3-7a6fa7387d81"),
                    Name = "Northland",
                    Code = "NTL",
                    RegionImageUrl = "https://images.pexels.com/photos/13918194/pexels-photo-13918194.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1"
                },
                new Region
                {
                    Id = Guid.Parse("14ceba71-4b51-4777-9b17-46602cf66153"),
                    Name = "Bay Of Plenty",
                    Code = "BOP",
                    RegionImageUrl = "https://images.pexels.com/photos/5169056/pexels-photo-5169056.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1"
                },
                new Region
                {
                    Id = Guid.Parse("cff032ed-27c9-496c-9411-7390772097e3"),
                    Name = "Wellington",
                    Code = "WLG",
                    RegionImageUrl = "https://images.pexels.com/photos/4350637/pexels-photo-4350637.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1"
                }
            };
            modelBuilder.Entity<Region>().HasData(regions);

            // --- Seeding WalkDifficulties ---
            var walkDifficulties = new List<WalkDifficulty>
            {
                new WalkDifficulty
                {
                    Id = Guid.Parse("0a256a14-419b-466d-961f-135e8064a30b"),
                    Code = "Easy"
                },
                new WalkDifficulty
                {
                    Id = Guid.Parse("e846743c-623e-4b71-ac32-901463e275f0"),
                    Code = "Medium"
                },
                new WalkDifficulty
                {
                    Id = Guid.Parse("f1a0e883-7b43-4e89-a292-15f5d8124971"),
                    Code = "Hard"
                }
            };
            modelBuilder.Entity<WalkDifficulty>().HasData(walkDifficulties);

            // --- Seeding Walks (ví dụ đơn giản) ---
            // Đảm bảo RegionId và WalkDifficultyId tham chiếu đến các Id đã được seeding ở trên
            var walks = new List<Walk>
            {
                new Walk
                {
                    Id = Guid.Parse("6a945934-8c83-4a8e-a22c-7b0f1d5d1c2a"),
                    Name = "Mission Bay Walk",
                    Description = "A pleasant walk along Mission Bay beach.",
                    LengthInKm = 5.2,
                    WalkImageUrl = "https://images.pexels.com/photos/1032395/pexels-photo-1032395.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1",
                    RegionId = Guid.Parse("f7248fc3-2587-4921-8d57-b2eac599f2ae"), // Auckland
                    WalkDifficultyId = Guid.Parse("0a256a14-419b-466d-961f-135e8064a30b") // Easy
                }
            };
            modelBuilder.Entity<Walk>().HasData(walks);
        }
    }
}
```

**Các bước thực hiện với `HasData()`:**

1.  Thêm phương thức `HasData()` vào `OnModelCreating` của `DbContext` với dữ liệu bạn muốn seeding.
2.  Sau khi thêm hoặc thay đổi dữ liệu seeding, bạn cần tạo một migration mới:
    ```bash
    Add-Migration SeedDataForRegionsAndDifficulties
    ```
    EF Core sẽ tạo một migration có chứa các lệnh `InsertData` (hoặc `UpdateData`) trong phương thức `Up()`.

3.  Áp dụng migration này vào cơ sở dữ liệu:
    ```bash
    Update-Database
    ```
    Lệnh này sẽ chèn dữ liệu vào các bảng tương ứng.

### 4. Lựa chọn Phương pháp Seeding phù hợp

*   **Sử dụng `HasData()`:**
    *   **Ưu điểm:** Tích hợp chặt chẽ với EF Core Migrations, tự động áp dụng và cập nhật dữ liệu seeding khi `Update-Database` được chạy. Dễ dàng quản lý dữ liệu seeding cùng với mã nguồn.
    *   **Nhược điểm:** Dữ liệu seeding được nhúng trực tiếp vào mã nguồn, có thể làm cho `OnModelCreating` trở nên dài dòng với lượng dữ liệu lớn. Không lý tưởng cho dữ liệu rất lớn hoặc dữ liệu cần thay đổi thường xuyên trong môi trường sản xuất.
    *   **Thích hợp cho:** Dữ liệu tham chiếu, dữ liệu cố định nhỏ, dữ liệu mẫu cho phát triển và kiểm thử.

*   **Sử dụng SQL Script:**
    *   **Ưu điểm:** Kiểm soát hoàn toàn các câu lệnh SQL. Có thể sử dụng cho dữ liệu rất lớn hoặc các kịch bản seeding phức tạp.
    *   **Nhược điểm:** Tách rời khỏi luồng EF Core Migrations. Yêu cầu chạy thủ công hoặc thông qua một cơ chế triển khai riêng.
    *   **Thích hợp cho:** Các kịch bản triển khai sản phẩm, khi cần nhập dữ liệu ban đầu từ nguồn ngoài, hoặc khi cần các thao tác SQL phức tạp không được EF Core hỗ trợ trực tiếp.

### Antigravity IDE và Seeding: Tự động hóa Dòng chảy Dữ liệu

Antigravity IDE có thể cách mạng hóa quá trình seeding dữ liệu, biến nó thành một phần không thể thiếu của Vibe Coding.

*   **Tự động tạo Dữ liệu Mẫu (Mock Data Generation):** Dựa trên các lớp entity của bạn, Antigravity có thể tự động tạo ra các đối tượng dữ liệu mẫu với các giá trị hợp lý (sử dụng các thư viện như Bogus hoặc AutoFixture). Bạn có thể chỉ định số lượng bản ghi, các quy tắc đặc biệt (ví dụ: email phải là duy nhất) và Antigravity sẽ tạo mã `HasData()` hoặc thậm chí một tập lệnh SQL `INSERT` cho bạn.
*   **Hỗ trợ Seeding Dữ liệu Tham chiếu:** Đối với các bảng tham chiếu như `WalkDifficulties`, Antigravity có thể gợi ý các giá trị phổ biến hoặc thậm chí tự động tạo danh sách các giá trị cơ bản, giúp bạn khởi tạo dữ liệu nhanh chóng.
*   **Kiểm tra tính nhất quán của Dữ liệu Seeding:** Khi bạn seeding dữ liệu có quan hệ (ví dụ: `Walk` cần `RegionId` và `WalkDifficultyId`), Antigravity có thể phân tích và đảm bảo rằng các `Id` được tham chiếu tồn tại trong dữ liệu seeding của các bảng khác, ngăn ngừa lỗi khóa ngoại.
*   **Tích hợp với Quy trình CI/CD:** Antigravity có thể giúp bạn thiết lập các quy trình tự động để áp dụng seeding data trong các môi trường khác nhau (dev, staging, production) thông qua các kịch bản triển khai thông minh.
*   **Vibe Coding aspect:** Bạn chỉ cần *cảm nhận* rằng ứng dụng cần dữ liệu mẫu cho một tính năng mới, và Antigravity sẽ *hiểu* ý định đó, tự động tạo và tích hợp dữ liệu seeding vào migration tiếp theo, cho phép bạn ngay lập tức bắt đầu phát triển tính năng mà không bị gián đoạn.

## III. Kiểm soát Phiên bản Mã nguồn với Git

Sau khi đã thực hiện các thay đổi đáng kể như tạo các mô hình miền mới, thiết lập EF Core, chạy migrations và seeding dữ liệu, việc lưu trữ và kiểm soát phiên bản các thay đổi này bằng Git là cực kỳ quan trọng. Git cho phép bạn theo dõi lịch sử thay đổi, cộng tác với người khác và dễ dàng khôi phục về các phiên bản trước đó nếu cần.

### 1. Tầm quan trọng của Git trong Phát triển Hiện đại

*   **Theo dõi Lịch sử:** Ghi lại mọi thay đổi trong mã nguồn, ai đã thay đổi gì, khi nào và tại sao.
*   **Cộng tác:** Cho phép nhiều nhà phát triển làm việc trên cùng một dự án một cách độc lập và sau đó hợp nhất công việc của họ.
*   **Phục hồi:** Dễ dàng quay lại các phiên bản trước của mã nguồn nếu có lỗi hoặc cần hoàn tác.
*   **Quản lý Nhánh (Branching):** Cho phép phát triển các tính năng mới hoặc sửa lỗi trong các nhánh riêng biệt mà không ảnh hưởng đến mã nguồn chính.
*   **Triển khai:** Là nền tảng cho các quy trình Tích hợp Liên tục/Triển khai Liên tục (CI/CD).

### 2. Thiết lập và Tích hợp Git trong Visual Studio

Visual Studio có tích hợp Git mạnh mẽ, giúp bạn quản lý mã nguồn trực tiếp từ IDE mà không cần dùng dòng lệnh (mặc dù hiểu các lệnh cơ bản vẫn rất hữu ích).

1.  **Đảm bảo Git được cài đặt:** Git cần được cài đặt trên hệ thống của bạn. Visual Studio thường đi kèm với một bản sao của Git, nhưng bạn có thể cần cập nhật hoặc cài đặt riêng nếu chưa có.
2.  **Thiết lập thông tin người dùng Git:**
    Trước khi commit, hãy cấu hình tên và email của bạn:
    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "your.email@example.com"
    ```
3.  **Khởi tạo Kho lưu trữ Git (Repository):**
    Nếu dự án của bạn chưa được kiểm soát bởi Git, bạn có thể khởi tạo một kho lưu trữ mới:

    *   Trong Visual Studio, đi tới `Git` > `Create Git Repository`.
    *   Chọn `Local Only` (để tạo kho lưu trữ cục bộ) hoặc `GitHub` (để tạo và liên kết với kho lưu trữ GitHub mới).
4.  **Mở cửa sổ Git Changes:**
    *   Trong Visual Studio, đi tới `View` > `Git Changes`.
    *   Cửa sổ này là trung tâm quản lý Git của bạn, hiển thị tất cả các tệp đã được sửa đổi, thêm mới hoặc xóa kể từ lần commit cuối cùng.

### 3. Staging và Commit các thay đổi

Quá trình lưu các thay đổi vào Git bao gồm hai bước chính: staging (dàn dựng) và committing (cam kết).

1.  **Staging các thay đổi:**
    *   Trong cửa sổ `Git Changes`, bạn sẽ thấy danh sách các "Unstaged Changes" (Thay đổi chưa được dàn dựng). Đây là tất cả các tệp mà bạn đã làm việc nhưng chưa chuẩn bị để đưa vào commit.
    *   Để chuẩn bị các tệp này cho commit, bạn có thể nhấp vào biểu tượng dấu `+` (Stage All) hoặc chọn từng tệp và nhấp vào biểu tượng `+` bên cạnh nó. Các tệp đã được dàn dựng sẽ chuyển sang phần "Staged Changes".
    *   **Tầm quan trọng của Staging:** Việc dàn dựng cho phép bạn chọn lọc các thay đổi cụ thể để đưa vào một commit. Điều này giúp các commit trở nên gọn gàng, có ý nghĩa và tập trung vào một tác vụ logic duy nhất, thay vì gom tất cả các thay đổi lộn xộn vào một commit lớn.

2.  **Viết thông điệp Commit:**
    *   Sau khi đã dàn dựng các thay đổi, bạn cần viết một "Commit Message" (Thông điệp commit) ngắn gọn nhưng mô tả rõ ràng những gì bạn đã thay đổi.
    *   **Nguyên tắc Thông điệp Commit tốt:**
        *   **Ngắn gọn:** Dòng đầu tiên (tiêu đề) không quá 50-72 ký tự.
        *   **Mô tả:** Phần thân (nếu có) giải thích chi tiết hơn về *tại sao* thay đổi này được thực hiện, không phải *cái gì* đã thay đổi (Git Diff đã làm điều đó).
        *   **Có ý nghĩa:** Bắt đầu bằng một động từ mệnh lệnh (ví dụ: "Add", "Fix", "Refactor", "Feat").
        *   **Ví dụ thông điệp commit tốt:** "Feat: Implement EF Core Migrations and Initial Seeding for Regions and Difficulties" hoặc "Fix: Correct connection string parsing in Program.cs".

3.  **Thực hiện Commit:**
    *   Sau khi viết thông điệp, nhấp vào nút `Commit Staged` (nếu chỉ muốn commit cục bộ) hoặc `Commit Staged and Push` (để commit cục bộ và ngay lập tức đẩy lên kho lưu trữ từ xa như GitHub).
    *   `Commit` sẽ tạo một bản ghi vĩnh viễn về các thay đổi của bạn trong lịch sử Git cục bộ.

### 4. Đẩy thay đổi lên kho lưu trữ từ xa (Remote Repository)

Để chia sẻ các thay đổi của bạn với nhóm hoặc lưu trữ chúng trên một dịch vụ như GitHub, GitLab, Azure DevOps, bạn cần đẩy (push) các commit cục bộ lên kho lưu trữ từ xa.

1.  **Thực hiện Push:**
    *   Nếu bạn đã chọn `Commit Staged and Push` ở bước trước, quá trình đẩy sẽ tự động diễn ra.
    *   Nếu bạn chỉ thực hiện `Commit Staged`, bạn có thể nhấp vào nút `Push` (biểu tượng mũi tên lên) trong cửa sổ `Git Changes` hoặc trên thanh trạng thái của Visual Studio.
    *   Quá trình này sẽ tải các commit mới từ kho lưu trữ cục bộ của bạn lên kho lưu trữ từ xa.
2.  **Xác nhận trên GitHub (hoặc dịch vụ Git khác):**
    *   Mở trình duyệt web và truy cập kho lưu trữ GitHub của dự án.
    *   Làm mới trang. Bạn sẽ thấy commit mới nhất của mình xuất hiện trong lịch sử commit, cùng với thông báo về thời gian push. Điều này xác nhận rằng các thay đổi của bạn đã được lưu trữ thành công trên dịch vụ Git từ xa.

> [!TIP]
> **Commit và Push thường xuyên:** Việc commit và push thường xuyên giúp bạn bảo vệ công việc của mình và cung cấp một lịch sử rõ ràng về sự phát triển của dự án. Luôn cố gắng viết các thông điệp commit có ý nghĩa để người khác (và bản thân bạn trong tương lai) có thể dễ dàng hiểu được mục đích của mỗi thay đổi.

### 5. Các Kỹ thuật Git Nâng cao (Giới thiệu)

*   **Branching (Phân nhánh):**
    *   Đây là một trong những tính năng mạnh mẽ nhất của Git, cho phép bạn tạo các "nhánh" riêng biệt của mã nguồn để phát triển tính năng mới hoặc sửa lỗi mà không ảnh hưởng đến nhánh chính (`main` hoặc `master`).
    *   Khi công việc hoàn tất, bạn có thể hợp nhất (merge) nhánh của mình trở lại nhánh chính.
    *   **Ví dụ:** `git checkout -b feature/add-users-api` (tạo và chuyển sang nhánh mới).
*   **Merging (Hợp nhất):**
    *   Kết hợp các thay đổi từ một nhánh này vào một nhánh khác.
    *   **Xung đột Hợp nhất (Merge Conflicts):** Xảy ra khi cùng một dòng mã được thay đổi khác nhau trên hai nhánh đang được hợp nhất. Git sẽ yêu cầu bạn giải quyết các xung đột này thủ công.
*   **Pull Request (Yêu cầu Hợp nhất):**
    *   Trong các nền tảng như GitHub, GitLab, đây là cơ chế chính để đề xuất hợp nhất các thay đổi từ một nhánh vào nhánh khác, thường đi kèm với quy trình xem xét mã (code review).
*   **Pull (Kéo):**
    *   `git pull` là lệnh để tải xuống các thay đổi từ kho lưu trữ từ xa và hợp nhất chúng vào nhánh cục bộ của bạn.

### Antigravity IDE và Git: Đồng bộ hóa Tư duy và Mã nguồn

Antigravity IDE có thể biến việc quản lý Git thành một trải nghiệm Vibe Coding, nơi bạn tập trung vào *ý nghĩa* của thay đổi, và AI lo phần *cơ chế*.

*   **Tự động Staging thông minh:** Antigravity có thể phân tích các thay đổi trong workspace của bạn và tự động dàn dựng các tệp liên quan đến một tác vụ logic cụ thể. Ví dụ, nếu bạn đang làm việc trên một tính năng, nó có thể tự động dàn dựng tất cả các tệp controller, service, repository và migration liên quan.
*   **Đề xuất Thông điệp Commit thông minh:** Dựa trên các thay đổi đã dàn dựng, lịch sử commit trước đó, và thậm chí cả các mô tả tác vụ trong hệ thống quản lý dự án của bạn, Antigravity có thể *tự động tạo hoặc đề xuất* thông điệp commit có ý nghĩa, tuân thủ các quy ước của dự án (ví dụ: Conventional Commits).
*   **Trực quan hóa Lịch sử và Nhánh:** Antigravity cung cấp một giao diện đồ họa trực quan, động để xem lịch sử commit, cấu trúc nhánh, và theo dõi các thay đổi. Nó có thể dự đoán và cảnh báo về các xung đột hợp nhất tiềm ẩn trước khi bạn thực hiện, giúp bạn giải quyết chúng chủ động.
*   **Tự động hóa Quy trình Gitflow/GitHub Flow:** Antigravity có thể hướng dẫn bạn qua các quy trình Gitflow hoặc GitHub Flow, tự động tạo nhánh tính năng, xử lý các pull request và hợp nhất sau khi code review được chấp thuận.
*   **Vibe Coding aspect:** Khi bạn hoàn thành một tác vụ, bạn chỉ cần *cảm nhận* rằng đã đến lúc lưu công việc. Antigravity IDE sẽ tự động chuẩn bị một commit hoàn chỉnh với thông điệp rõ ràng, sẵn sàng để bạn xác nhận hoặc chỉnh sửa nhỏ, sau đó tự động đẩy lên remote. Điều này giúp bạn duy trì "vibe" lập trình, không bị gián đoạn bởi các thao tác Git lặp đi lặp lại.

---

## Tóm tắt Phần 19

Trong chương này, chúng ta đã xây dựng nền tảng vững chắc cho việc quản lý cơ sở dữ liệu và mã nguồn cho dự án RESTful Web API của mình:

*   **Entity Framework Core Migrations:** Chúng ta đã tìm hiểu cách sử dụng `Add-Migration` để tạo các tệp mô tả thay đổi lược đồ dựa trên `DbContext` và các lớp entity, và `Update-Database` để áp dụng các thay đổi này vào cơ sở dữ liệu SQL Server. Chúng ta cũng đã khám phá cơ chế hoạt động ngầm và các kỹ thuật quản lý migration nâng cao.
*   **Seeding Dữ liệu:** Chúng ta đã xem xét hai phương pháp seeding dữ liệu: sử dụng tập lệnh SQL thủ công và phương pháp tích hợp hơn của EF Core thông qua `HasData()` trong `OnModelCreating`. Phương pháp `HasData()` được khuyến nghị cho dữ liệu tĩnh và mẫu vì tính tích hợp của nó với migrations.
*   **Quản lý Mã nguồn với Git:** Chúng ta đã nắm vững các bước cơ bản để sử dụng Git trong Visual Studio, bao gồm staging các tệp đã thay đổi, viết thông điệp commit có ý nghĩa và đẩy (push) các commit lên kho lưu trữ từ xa (như GitHub) để lưu trữ và chia sẻ công việc của chúng ta.
*   **Antigravity IDE và Vibe Coding:** Xuyên suốt chương, chúng ta đã liên hệ các khái niệm này với Antigravity IDE, một hệ thống Agentic AI, và tư duy Vibe Coding. Antigravity có thể tự động hóa, đề xuất, và trực quan hóa các tác vụ liên quan đến migrations, seeding và Git, giúp nhà phát triển duy trì dòng chảy tư duy và tập trung vào sáng tạo.

Với những kiến thức và công cụ này, bạn đã sẵn sàng để quản lý sự tiến hóa của dự án ASP.NET Core của mình một cách hiệu quả, chuyên nghiệp và với một trải nghiệm phát triển nâng cao.

<!-- REVIEWED_BY_AGENT -->
