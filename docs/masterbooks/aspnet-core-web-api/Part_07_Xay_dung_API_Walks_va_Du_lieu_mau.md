# Phần 7: Xây dựng API cho Hành trình (Walks) & Kiến trúc API RESTful

Trong phần này, chúng ta sẽ đi sâu vào việc xây dựng một API RESTful toàn diện cho tài nguyên "Hành trình" (Walks) trong ứng dụng của chúng ta. Mục tiêu không chỉ là tạo ra các điểm cuối (endpoints) để thực hiện đầy đủ các thao tác CRUD (Create, Read, Update, Delete), mà còn là tích hợp các khái niệm kiến trúc và lập trình cốt lõi như Seeding Data, Dependency Injection, Repository Pattern, Data Transfer Objects (DTOs), AutoMapper và cách quản lý Navigation Properties trong Entity Framework Core. Chúng ta sẽ tập trung vào việc áp dụng các phương pháp hay nhất để xây dựng một API mạnh mẽ, dễ bảo trì và mở rộng bằng ASP.NET Core và C#.

## 1. Khởi tạo Dữ liệu Ban đầu (Data Seeding) với Entity Framework Core

Trước khi có thể thao tác với các thực thể "Hành trình", chúng ta cần đảm bảo rằng các dữ liệu tham chiếu như "Mức độ Khó khăn" (Difficulties) và "Khu vực" (Regions) đã tồn tại trong cơ sở dữ liệu. Đây là những loại dữ liệu tĩnh, ít thay đổi và là điều kiện tiên quyết để các hành trình có thể tham chiếu hợp lệ. Entity Framework Core cung cấp một cơ chế hiệu quả để "gieo hạt" (seeding) dữ liệu ban đầu này.

### 1.1. Tầm quan trọng của Data Seeding

Data Seeding là quá trình thêm dữ liệu khởi tạo vào cơ sở dữ liệu khi ứng dụng được triển khai lần đầu hoặc khi cơ sở dữ liệu được tạo/cập nhật thông qua các migration. Việc này mang lại nhiều lợi ích thiết thực:

*   **Đảm bảo tính nhất quán:** Cung cấp các giá trị mặc định, danh mục cố định, hoặc cấu hình ban đầu cần thiết cho hoạt động của ứng dụng.
*   **Dữ liệu tham chiếu:** Khởi tạo các bảng lookup (như `Difficulties`, `Regions`, `Roles`) mà các thực thể khác sẽ dựa vào.
*   **Phát triển và Kiểm thử hiệu quả:** Tạo sẵn dữ liệu mẫu giúp nhà phát triển có thể kiểm thử các chức năng ngay lập tức mà không cần nhập liệu thủ công, tăng tốc quá trình phát triển.
*   **Trạng thái khởi tạo ổn định:** Đảm bảo môi trường phát triển, kiểm thử, hoặc thậm chí môi trường sản phẩm luôn bắt đầu với một tập dữ liệu cơ bản có thể dự đoán được.

### 1.2. Cơ chế Seeding trong Entity Framework Core

Trong EF Core, việc seeding dữ liệu được thực hiện bằng cách ghi đè phương thức `OnModelCreating` trong lớp `DbContext` của bạn. Phương thức này là nơi bạn cấu hình mô hình dữ liệu (model building) và cũng là nơi bạn có thể chỉ định dữ liệu ban đầu cho các thực thể.

**Cơ chế hoạt động (Under the Hood):**
Khi bạn gọi `Add-Migration`, EF Core sẽ phân tích các thay đổi trong mô hình của bạn. Nếu nó phát hiện các lệnh `HasData()`, nó sẽ tự động tạo ra mã SQL `INSERT` tương ứng trong phương thức `Up()` của file migration. Khi bạn chạy `Update-Database`, các lệnh SQL này sẽ được thực thi, chèn dữ liệu vào các bảng tương ứng. Điều này đảm bảo rằng dữ liệu seeding được quản lý như một phần của lịch sử database schema thông qua migrations.

**Các bước thực hiện:**

1.  **Mở `DbContext`:** Điều hướng đến tệp `[TênDựÁn]DbContext.cs` (thường nằm trong thư mục `Data`).
2.  **Ghi đè `OnModelCreating`:** Thêm phương thức `OnModelCreating` và sử dụng `modelBuilder.Entity<T>().HasData(...)` để cung cấp dữ liệu.

    ```csharp
    // Data/NzWalksDbContext.cs
    using Microsoft.EntityFrameworkCore;
    using NzWalks.API.Models.Domain; // Đảm bảo đúng namespace cho Domain Models

    namespace NzWalks.API.Data
    {
        public class NzWalksDbContext : DbContext
        {
            public NzWalksDbContext(DbContextOptions<NzWalksDbContext> dbContextOptions) : base(dbContextOptions)
            {
            }

            // DbSet cho các thực thể của bạn
            public DbSet<Difficulty> Difficulties { get; set; }
            public DbSet<Region> Regions { get; set; }
            public DbSet<Walk> Walks { get; set; }

            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                base.OnModelCreating(modelBuilder); // Luôn gọi base method trước

                // 1. Seeding dữ liệu cho Difficulties
                // Sử dụng List để dễ dàng quản lý nhiều bản ghi
                var difficulties = new List<Difficulty>()
                {
                    new Difficulty()
                    {
                        Id = Guid.Parse("54460f38-a536-4074-b580-c13636c0e5a4"),
                        Name = "Easy"
                    },
                    new Difficulty()
                    {
                        Id = Guid.Parse("9621516e-e72e-4b72-8703-e847055998a4"),
                        Name = "Medium"
                    },
                    new Difficulty()
                    {
                        Id = Guid.Parse("370b4f8d-c782-421f-8255-a50e5888d1d8"),
                        Name = "Hard"
                    }
                };
                modelBuilder.Entity<Difficulty>().HasData(difficulties);

                // 2. Seeding dữ liệu cho Regions
                var regions = new List<Region>()
                {
                    new Region
                    {
                        Id = Guid.Parse("f7248fc3-2587-4923-9d21-f57210f81ae1"),
                        Name = "Auckland",
                        Code = "AKL",
                        RegionImageUrl = "https://images.unsplash.com/photo-1549419142-205e492f1f1d" // Ví dụ URL
                    },
                    new Region
                    {
                        Id = Guid.Parse("6884f7d7-ad1f-4101-8df3-7a6fa7387d81"),
                        Name = "Northland",
                        Code = "NTL",
                        RegionImageUrl = null
                    },
                    new Region
                    {
                        Id = Guid.Parse("14ceba71-4b51-4777-9b17-01f11e427dd6"),
                        Name = "Bay Of Plenty",
                        Code = "BOP",
                        RegionImageUrl = "https://images.unsplash.com/photo-1623910543627-2c1b9b9a6b1e" // Ví dụ URL
                    },
                    new Region
                    {
                        Id = Guid.Parse("cfa06fdc-edcd-46a1-bb2b-a1789086dd7a"),
                        Name = "Wellington",
                        Code = "WGN",
                        RegionImageUrl = "https://images.unsplash.com/photo-1507699622108-4cd7ce5bb65d" // Ví dụ URL
                    },
                    new Region
                    {
                        Id = Guid.Parse("906cb139-4959-4406-b337-1cd20bdd8602"),
                        Name = "Nelson",
                        Code = "NSN",
                        RegionImageUrl = null
                    },
                    new Region
                    {
                        Id = Guid.Parse("2f2f7547-f010-4f51-939f-b9d198865e90"),
                        Name = "Southland",
                        Code = "STL",
                        RegionImageUrl = "https://images.unsplash.com/photo-1601618059639-6e6e23b2b0a3" // Ví dụ URL
                    }
                };
                modelBuilder.Entity<Region>().HasData(regions);
            }
        }
    }
    ```

3.  **Tạo và Cập nhật Migration:** Sau khi thêm dữ liệu seeding vào `OnModelCreating`, bạn cần tạo một migration mới để EF Core phát hiện các lệnh `HasData` và sinh ra mã SQL tương ứng. Sau đó, cập nhật cơ sở dữ liệu để áp dụng những thay đổi này.
    *   Mở Package Manager Console (Tools -> NuGet Package Manager -> Package Manager Console).
    *   Chạy lệnh: `Add-Migration "SeedingDifficultyAndRegionData"`
    *   Chạy lệnh: `Update-Database`

> [!TIP]
> Để tạo các `Guid` duy nhất cho dữ liệu mẫu một cách nhanh chóng, bạn có thể sử dụng cửa sổ C# Interactive (View -> Other Windows -> C# Interactive) và gõ `Guid.NewGuid()` để nhận một `Guid` mới. Sau đó, sao chép giá trị này và sử dụng `Guid.Parse("chuỗi_guid_vừa_tạo")` trong code của bạn. Việc sử dụng các `Guid` cố định trong dữ liệu seeding giúp đảm bảo tính nhất quán và khả năng tái tạo dữ liệu giữa các lần triển khai.

Sau khi hoàn tất các bước trên, cơ sở dữ liệu của bạn sẽ có sẵn dữ liệu cho `Difficulties` và `Regions`, sẵn sàng để sử dụng khi tạo các "Hành trình".

## 2. Nền tảng Kiến trúc API RESTful & ASP.NET Core

Để xây dựng một API RESTful mạnh mẽ, dễ bảo trì và mở rộng, chúng ta sẽ áp dụng các nguyên tắc kiến trúc và mẫu thiết kế đã được chứng minh trong ASP.NET Core.

### 2.1. Tổng quan về API RESTful và vai trò của ASP.NET Core

**REST (Representational State Transfer)** là một tập hợp các nguyên tắc kiến trúc để thiết kế các dịch vụ web. Một API được coi là RESTful khi nó tuân thủ các nguyên tắc sau:

*   **Client-Server:** Tách biệt các mối quan tâm giữa giao diện người dùng (client) và lưu trữ dữ liệu (server).
*   **Stateless (Không trạng thái):** Mỗi yêu cầu từ client đến server phải chứa tất cả thông tin cần thiết để server hiểu yêu cầu đó. Server không lưu trữ bất kỳ trạng thái nào của client giữa các yêu cầu.
*   **Cacheable (Có thể lưu vào bộ nhớ đệm):** Phản hồi từ server có thể được đánh dấu là có thể lưu vào bộ nhớ đệm hoặc không, giúp cải thiện hiệu suất.
*   **Layered System (Hệ thống phân lớp):** Client không cần biết liệu nó đang kết nối trực tiếp với server cuối cùng hay một trung gian (ví dụ: proxy, load balancer).
*   **Uniform Interface (Giao diện đồng nhất):** Đây là nguyên tắc cốt lõi, bao gồm:
    *   **Resource Identification in Requests:** Các tài nguyên được xác định bằng URI duy nhất.
    *   **Resource Manipulation Through Representations:** Client thao tác với tài nguyên bằng cách gửi các biểu diễn (representations) của tài nguyên đó.
    *   **Self-Descriptive Messages:** Mỗi thông điệp chứa đủ thông tin để mô tả cách xử lý thông điệp đó.
    *   **Hypermedia as the Engine of Application State (HATEOAS):** Client tương tác với API hoàn toàn thông qua các siêu liên kết được cung cấp trong phản hồi. (Mặc dù HATEOAS là một phần của REST, nhiều API thực tế thường chỉ tuân thủ các nguyên tắc trước đó và được gọi là "REST-like" hoặc "HTTP APIs").

**ASP.NET Core** là một framework mạnh mẽ và linh hoạt để xây dựng các API RESTful. Nó cung cấp:

*   **Controllers:** Các lớp chịu trách nhiệm xử lý các yêu cầu HTTP đến các tài nguyên cụ thể.
*   **Action Methods:** Các phương thức trong Controller được ánh xạ tới các yêu cầu HTTP cụ thể (GET, POST, PUT, DELETE).
*   **HTTP Verbs:** Sử dụng các động từ HTTP chuẩn để chỉ định loại thao tác (Ví dụ: `GET` để truy xuất, `POST` để tạo, `PUT` để cập nhật, `DELETE` để xóa).
*   **Routing:** Cơ chế ánh xạ các URL đến các Action Methods.
*   **Model Binding:** Tự động ánh xạ dữ liệu từ yêu cầu HTTP (query string, route data, request body) vào các tham số của Action Method.

### 2.2. Dependency Injection (DI)

**Dependency Injection (DI)** là một kỹ thuật thiết kế phần mềm giúp giảm sự phụ thuộc cứng giữa các thành phần. Nó là một hình thức của nguyên tắc **Inversion of Control (IoC)**, nơi khung ứng dụng (framework) hoặc một vùng chứa (container) chịu trách nhiệm tạo và cung cấp các đối tượng phụ thuộc, thay vì lớp tự tạo ra chúng. Trong ASP.NET Core, DI được tích hợp sẵn và đóng vai trò trung tâm trong việc quản lý vòng đời của các dịch vụ và đối tượng.

*   **Vai trò và Lợi ích của DI:**
    *   **Giảm khớp nối (Decoupling):** Các lớp không cần biết cách tạo các đối tượng phụ thuộc của chúng, chỉ cần biết cách sử dụng chúng thông qua các giao diện (interface). Điều này làm cho hệ thống linh hoạt hơn.
    *   **Tăng khả năng kiểm thử (Testability):** Dễ dàng thay thế các phụ thuộc thực tế bằng các đối tượng giả (mock/stub) trong quá trình kiểm thử đơn vị, giúp cô lập logic của lớp đang kiểm thử.
    *   **Tăng khả năng mở rộng (Extensibility):** Dễ dàng thay đổi triển khai của một dịch vụ mà không ảnh hưởng đến các lớp sử dụng nó (chỉ cần thay đổi cấu hình đăng ký DI).
    *   **Quản lý vòng đời (Lifetime Management):** Vùng chứa DI quản lý việc tạo và hủy các đối tượng theo các chiến lược vòng đời khác nhau, tối ưu hóa tài nguyên.
    *   **Giảm Boilerplate Code:** Loại bỏ mã khởi tạo đối tượng lặp đi lặp lại.

*   **Đăng ký Dịch vụ trong `Program.cs`:**
    Các dịch vụ (như Repository, AutoMapper) được đăng ký trong vùng chứa DI của ASP.NET Core (thông qua `builder.Services` trong `Program.cs`) để vùng chứa biết cách tạo và cung cấp chúng khi cần.

    ```csharp
    // Program.cs
    // ...
    builder.Services.AddScoped<IWalkRepository, SQLWalkRepository>(); // Đăng ký Repository
    builder.Services.AddAutoMapper(typeof(AutoMapperProfiles));      // Đăng ký AutoMapper
    // ...
    ```

*   **Vòng đời Dịch vụ (Service Lifetimes) - Under the Hood:**
    Khi đăng ký dịch vụ, bạn cần chỉ định vòng đời của chúng. ASP.NET Core cung cấp ba vòng đời chính:

    *   `AddScoped`: Một thể hiện (instance) của dịch vụ được tạo **một lần cho mỗi yêu cầu HTTP**. Nếu một yêu cầu HTTP cần dịch vụ này nhiều lần, nó sẽ nhận cùng một thể hiện. Thích hợp cho các dịch vụ liên quan đến trạng thái của yêu cầu, như `DbContext` hoặc Repository.
    *   `AddTransient`: Một thể hiện mới của dịch vụ được tạo **mỗi khi nó được yêu cầu**. Thích hợp cho các dịch vụ nhẹ, không trạng thái hoặc các dịch vụ cần được tạo mới mỗi lần sử dụng.
    *   `AddSingleton`: Một thể hiện duy nhất của dịch vụ được tạo **lần đầu tiên nó được yêu cầu** và được tái sử dụng trong suốt vòng đời của ứng dụng. Thích hợp cho các dịch vụ không trạng thái hoặc các dịch vụ tốn kém để khởi tạo.

*   **Constructor Injection:**
    Cách phổ biến nhất để nhận các phụ thuộc là thông qua hàm tạo (constructor) của lớp. Vùng chứa DI sẽ tự động giải quyết và cung cấp các thể hiện của các phụ thuộc đã đăng ký.

    ```csharp
    public class WalksController : ControllerBase
    {
        private readonly IWalkRepository walkRepository;
        private readonly IMapper mapper; // IMapper được cung cấp bởi AutoMapper

        public WalksController(IWalkRepository walkRepository, IMapper mapper) // Constructor Injection
        {
            this.walkRepository = walkRepository;
            this.mapper = mapper;
        }
        // ... các Action Methods
    }
    ```

    > [!NOTE]
    > Việc áp dụng Dependency Injection và các nguyên tắc thiết kế tốt như Single Responsibility Principle (SRP) giúp tạo ra một codebase có "Vibe Coding" tốt. Vibe Coding là cảm giác trực quan về một codebase được tổ chức mạch lạc, dễ hiểu và dễ mở rộng. Một hệ thống có Vibe Coding tốt sẽ giúp các nhà phát triển con người, cũng như các công cụ lập trình AI tiên tiến như Antigravity IDE, dễ dàng điều hướng, phân tích cấu trúc, và tự động đề xuất hoặc thực thi các thay đổi một cách chính xác và hiệu quả.

### 2.3. Repository Pattern

**Repository Pattern** là một mẫu thiết kế giúp tách biệt logic truy cập dữ liệu khỏi logic nghiệp vụ (business logic) của ứng dụng. Nó cung cấp một lớp trừu tượng trên lớp dữ liệu, hoạt động như một bộ sưu tập các đối tượng miền (domain objects) trong bộ nhớ.

*   **Mục đích và Lợi ích:**
    *   **Tách biệt mối quan tâm (Separation of Concerns):** Logic truy cập dữ liệu (thao tác trực tiếp với `DbContext`, SQL, v.v.) được đóng gói hoàn toàn trong Repository. Các lớp nghiệp vụ không cần quan tâm đến chi tiết lưu trữ.
    *   **Dễ dàng thay đổi công nghệ dữ liệu:** Nếu bạn muốn chuyển từ Entity Framework Core sang Dapper, ADO.NET thuần, hoặc một ORM khác, bạn chỉ cần thay đổi triển khai của Repository mà không ảnh hưởng đến các lớp sử dụng nó (chẳng hạn như Controller).
    *   **Tăng khả năng kiểm thử (Testability):** Dễ dàng mock (giả lập) Repository trong các bài kiểm thử đơn vị, cho phép kiểm thử logic nghiệp vụ mà không cần một cơ sở dữ liệu thực.
    *   **Bảo vệ Domain Model:** Ngăn chặn việc lộ các chi tiết triển khai cơ sở dữ liệu hoặc các truy vấn phức tạp ra bên ngoài Domain Model.

*   **Cấu trúc:**
    *   **Giao diện (`IRepository`):** Định nghĩa hợp đồng (contract) cho các phương thức mà Repository sẽ cung cấp (ví dụ: `CreateAsync`, `GetAllAsync`, `GetByIdAsync`, `UpdateAsync`, `DeleteAsync`). Điều này đảm bảo tính trừu tượng.
    *   **Triển khai cụ thể (`SQLRepository`):** Chứa logic thực tế để tương tác với cơ sở dữ liệu thông qua `DbContext` hoặc bất kỳ công nghệ truy cập dữ liệu nào khác.

    ```csharp
    // Repositories/IWalkRepository.cs (Giao diện)
    using NzWalks.API.Models.Domain;

    namespace NzWalks.API.Repositories
    {
        public interface IWalkRepository
        {
            Task<Walk> CreateAsync(Walk walk);
            Task<List<Walk>> GetAllAsync();
            Task<Walk?> GetByIdAsync(Guid id); // Sử dụng Task<Walk?> để chỉ rõ có thể trả về null
            Task<Walk?> UpdateAsync(Guid id, Walk walk);
            Task<Walk?> DeleteAsync(Guid id);
        }
    }

    // Repositories/SQLWalkRepository.cs (Triển khai)
    using Microsoft.EntityFrameworkCore;
    using NzWalks.API.Data;
    using NzWalks.API.Models.Domain;

    namespace NzWalks.API.Repositories
    {
        public class SQLWalkRepository : IWalkRepository
        {
            private readonly NzWalksDbContext dbContext;

            public SQLWalkRepository(NzWalksDbContext dbContext) // Nhận DbContext qua DI
            {
                this.dbContext = dbContext;
            }

            public async Task<Walk> CreateAsync(Walk walk)
            {
                await dbContext.Walks.AddAsync(walk);
                await dbContext.SaveChangesAsync();
                return walk;
            }

            public async Task<List<Walk>> GetAllAsync()
            {
                // Bao gồm (Include) Navigation Properties để lấy dữ liệu liên quan
                // Sẽ được giải thích chi tiết hơn ở phần sau.
                return await dbContext.Walks
                    .Include(x => x.Difficulty)
                    .Include(x => x.Region)
                    .ToListAsync();
            }
            // ... các phương thức khác sẽ được triển khai chi tiết ở phần sau
        }
    }
    ```

### 2.4. Data Transfer Objects (DTOs)

**Data Transfer Objects (DTOs)** là các đối tượng đơn giản được sử dụng để truyền dữ liệu giữa các lớp hoặc giữa các hệ thống. Trong ngữ cảnh của một API RESTful, DTO được sử dụng để định hình dữ liệu được gửi trong yêu cầu (request body) từ client và dữ liệu được trả về trong phản hồi (response body) cho client.

*   **Mục đích và Lợi ích:**
    *   **Tách biệt Domain Model khỏi API Model:** Ngăn chặn việc lộ cấu trúc cơ sở dữ liệu hoặc logic nghiệp vụ không cần thiết ra bên ngoài API. Domain Model của bạn có thể chứa các thuộc tính, phương thức, và mối quan hệ phức tạp, nhưng DTO chỉ hiển thị những gì client cần và được phép thấy.
    *   **Kiểm soát dữ liệu đầu vào/đầu ra:** Cho phép bạn chỉ hiển thị (cho đầu ra) hoặc chỉ chấp nhận (cho đầu vào) những trường dữ liệu cụ thể, không nhất thiết phải khớp hoàn toàn với Domain Model.
    *   **Bảo mật (chống Over-posting/Under-posting):** Ngăn chặn client gửi các trường dữ liệu không mong muốn (over-posting) hoặc bỏ qua các trường bắt buộc (under-posting) có thể gây ra lỗ hổng bảo mật hoặc lỗi logic.
    *   **Linh hoạt trong phiên bản API:** Cấu trúc DTO có thể khác với Domain Model và có thể được thay đổi độc lập để hỗ trợ các phiên bản API khác nhau mà không ảnh hưởng đến logic nghiệp vụ cốt lõi.
    *   **Giảm tải mạng:** Chỉ truyền tải dữ liệu cần thiết, giảm kích thước payload.

*   **Ví dụ:**
    *   `AddWalkRequestDto`: Dữ liệu client gửi để tạo một hành trình mới.
    *   `UpdateWalkRequestDto`: Dữ liệu client gửi để cập nhật một hành trình.
    *   `WalkDto`: Dữ liệu hành trình trả về cho client.
    *   `RegionDto`, `DifficultyDto`: Dữ liệu liên quan được nhúng trong `WalkDto` để cung cấp thông tin đầy đủ.

    ```csharp
    // Models/DTO/AddWalkRequestDto.cs (Input DTO)
    namespace NzWalks.API.Models.DTO
    {
        public class AddWalkRequestDto
        {
            public string Name { get; set; }
            public string Description { get; set; }
            public double LengthInKm { get; set; }
            public string? WalkImageUrl { get; set; } // Có thể null
            public Guid DifficultyId { get; set; }
            public Guid RegionId { get; set; }
        }
    }

    // Models/DTO/WalkDto.cs (Output DTO)
    namespace NzWalks.API.Models.DTO
    {
        public class WalkDto
        {
            public Guid Id { get; set; }
            public string Name { get; set; }
            public string Description { get; set; }
            public double LengthInKm { get; set; }
            public string? WalkImageUrl { get; set; }
            public DifficultyDto Difficulty { get; set; } // Bao gồm DTO của Difficulty
            public RegionDto Region { get; set; }       // Bao gồm DTO của Region
        }
    }

    // Models/DTO/DifficultyDto.cs (Nested DTO)
    namespace NzWalks.API.Models.DTO
    {
        public class DifficultyDto
        {
            public Guid Id { get; set; }
            public string Name { get; set; }
        }
    }

    // Models/DTO/RegionDto.cs (Nested DTO)
    namespace NzWalks.API.Models.DTO
    {
        public class RegionDto
        {
            public Guid Id { get; set; }
            public string Code { get; set; }
            public string Name { get; set; }
            public string? RegionImageUrl { get; set; }
        }
    }
    ```

### 2.5. AutoMapper

**AutoMapper** là một thư viện phổ biến giúp tự động ánh xạ giữa hai đối tượng có cấu trúc tương tự. Nó loại bỏ nhu cầu viết mã ánh xạ thủ công lặp đi lặp lại (`obj.PropA = otherObj.PropA; obj.PropB = otherObj.PropB;...`), giúp giảm boilerplate code và tăng năng suất.

*   **Mục đích và Lợi ích:**
    *   **Giảm Boilerplate Code:** Tự động chuyển đổi giữa Domain Model và DTOs (hoặc bất kỳ hai loại đối tượng nào) mà không cần viết mã ánh xạ tường minh cho từng thuộc tính.
    *   **Tăng năng suất:** Đơn giản hóa quá trình phát triển bằng cách tập trung vào logic nghiệp vụ thay vì logic ánh xạ.
    *   **Dễ bảo trì:** Các thay đổi trong mô hình sẽ dễ dàng được phản ánh trong ánh xạ nếu các tên thuộc tính được giữ nguyên.
    *   **Kiểm tra và Xác thực:** AutoMapper cung cấp các công cụ để kiểm tra cấu hình ánh xạ của bạn, đảm bảo rằng tất cả các thuộc tính cần thiết đều được ánh xạ.

*   **Cấu hình Profile:**
    Bạn định nghĩa các quy tắc ánh xạ trong một lớp kế thừa từ `AutoMapper.Profile`. Các profile này sau đó được đăng ký với vùng chứa DI.

    ```csharp
    // Mappings/AutoMapperProfiles.cs
    using AutoMapper;
    using NzWalks.API.Models.Domain;
    using NzWalks.API.Models.DTO;

    namespace NzWalks.API.Mappings
    {
        public class AutoMapperProfiles : Profile
        {
            public AutoMapperProfiles()
            {
                // Ánh xạ DTO sang Domain Model (và ngược lại với ReverseMap())
                CreateMap<AddWalkRequestDto, Walk>().ReverseMap();
                CreateMap<UpdateWalkRequestDto, Walk>().ReverseMap();
                
                // Ánh xạ Domain Model sang DTO (và ngược lại nếu cần)
                CreateMap<Walk, WalkDto>().ReverseMap();
                CreateMap<Region, RegionDto>().ReverseMap();
                CreateMap<Difficulty, DifficultyDto>().ReverseMap();
            }
        }
    }
    ```
    > [!NOTE]
    > `ReverseMap()` là một tiện ích của AutoMapper cho phép bạn ánh xạ cả hai chiều (từ nguồn đến đích và từ đích đến nguồn) chỉ với một cấu hình. Điều này rất tiện lợi khi DTO và Domain Model có cấu trúc tương tự và bạn cần chuyển đổi qua lại.

## 3. Xây dựng Controller: `WalksController`

`WalksController` sẽ là điểm truy cập chính cho tất cả các yêu cầu HTTP liên quan đến tài nguyên "Walks". Nó sẽ sử dụng Dependency Injection để nhận các thể hiện của `IWalkRepository` (để tương tác với dữ liệu) và `IMapper` (để chuyển đổi giữa Domain Model và DTO).

```csharp
// Controllers/WalksController.cs
using AutoMapper;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using NzWalks.API.Models.Domain;
using NzWalks.API.Models.DTO;
using NzWalks.API.Repositories;

namespace NzWalks.API.Controllers
{
    [Route("api/[controller]")] // Định nghĩa route cơ bản cho controller: /api/walks
    [ApiController]             // Chỉ định rằng đây là một API controller, kích hoạt các tính năng như Model Binding, Validation tự động
    public class WalksController : ControllerBase
    {
        private readonly IWalkRepository walkRepository;
        private readonly IMapper mapper;

        // Constructor Injection: Nhận IWalkRepository và IMapper từ vùng chứa DI
        public WalksController(IWalkRepository walkRepository, IMapper mapper)
        {
            this.walkRepository = walkRepository;
            this.mapper = mapper;
        }

        // Các Action Methods (phương thức hành động) cho CRUD sẽ được định nghĩa ở đây
    }
}
```

## 4. Triển khai Các Thao tác CRUD cho Tài nguyên 'Hành trình'

Chúng ta sẽ lần lượt xây dựng các phương thức hành động (Action Methods) trong `WalksController` để thực hiện các thao tác CRUD, đồng thời triển khai các phương thức tương ứng trong `SQLWalkRepository`.

### 4.1. Tạo Hành trình Mới (Create New Walk - HTTP POST)

Phương thức này cho phép client gửi dữ liệu để tạo một hành trình mới trong hệ thống.

*   **HTTP Verb:** `POST`
*   **Endpoint:** `/api/walks`
*   **Yêu cầu (Request):** Nhận một `AddWalkRequestDto` từ phần thân (body) của yêu cầu HTTP.
*   **Phản hồi (Response):** `201 Created` nếu thành công, kèm theo `WalkDto` của hành trình đã tạo.
*   **Logic:**
    1.  Ánh xạ `AddWalkRequestDto` nhận được từ client sang `Walk` Domain Model.
    2.  Gọi phương thức `CreateAsync` của `IWalkRepository` để lưu `Walk` Domain Model vào cơ sở dữ liệu.
    3.  Ánh xạ `Walk` Domain Model kết quả (đã có ID từ DB) sang `WalkDto`.
    4.  Trả về phản hồi `201 Created` kèm theo `WalkDto` đã tạo và một header `Location` chỉ định URL của tài nguyên mới.

```csharp
// Controllers/WalksController.cs
// ... (trong WalksController)

/// <summary>
/// Tạo một hành trình mới.
/// </summary>
/// <param name="addWalkRequestDto">Dữ liệu hành trình cần tạo từ client.</param>
/// <returns>Thông tin hành trình đã tạo và trạng thái 201 Created.</returns>
[HttpPost]
// [Route("api/walks")] // Có thể bỏ qua nếu route base là api/[controller]
public async Task<IActionResult> Create([FromBody] AddWalkRequestDto addWalkRequestDto)
{
    // [Bước 1] Ánh xạ DTO sang Domain Model
    var walkDomainModel = mapper.Map<Walk>(addWalkRequestDto);

    // [Bước 2] Gọi Repository để tạo hành trình trong cơ sở dữ liệu
    walkDomainModel = await walkRepository.CreateAsync(walkDomainModel);

    // [Bước 3] Ánh xạ Domain Model kết quả (đã được lưu vào DB) sang DTO
    var walkDto = mapper.Map<WalkDto>(walkDomainModel);

    // [Bước 4] Trả về 201 Created với URL của tài nguyên mới
    // CreatedAtAction là một phương thức tiện lợi để trả về 201 Created
    // Nó tự động tạo header Location với URL của tài nguyên mới
    return CreatedAtAction(nameof(GetById), new { id = walkDto.Id }, walkDto);
}
```

```csharp
// Repositories/SQLWalkRepository.cs
// ... (trong SQLWalkRepository)

public async Task<Walk> CreateAsync(Walk walk)
{
    await dbContext.Walks.AddAsync(walk); // Thêm thực thể vào DbContext
    await dbContext.SaveChangesAsync();   // Lưu thay đổi vào cơ sở dữ liệu
    return walk;                          // Trả về thực thể đã được cập nhật (có ID)
}
```

### 4.2. Lấy Tất cả Hành trình (Get All Walks - HTTP GET)

Phương thức này sẽ trả về danh sách tất cả các hành trình có trong cơ sở dữ liệu.

*   **HTTP Verb:** `GET`
*   **Endpoint:** `/api/walks`
*   **Yêu cầu:** Không có body yêu cầu.
*   **Phản hồi:** `200 OK` kèm theo danh sách `WalkDto`.
*   **Logic:**
    1.  Gọi phương thức `GetAllAsync` của `IWalkRepository` để lấy tất cả hành trình.
    2.  **Quan trọng:** Repository sẽ sử dụng Navigation Properties để tải dữ liệu của `Difficulty` và `Region` liên quan (Eager Loading).
    3.  Ánh xạ danh sách `Walk` Domain Model sang danh sách `WalkDto`.
    4.  Trả về phản hồi `200 OK` kèm theo danh sách `WalkDto`.

```csharp
// Controllers/WalksController.cs
// ... (trong WalksController)

/// <summary>
/// Lấy tất cả các hành trình.
/// </summary>
/// <returns>Danh sách các hành trình và trạng thái 200 OK.</returns>
[HttpGet]
// [Route("api/walks")]
public async Task<IActionResult> GetAll()
{
    // Gọi Repository để lấy tất cả hành trình từ cơ sở dữ liệu
    var walksDomainModel = await walkRepository.GetAllAsync();

    // Ánh xạ danh sách Domain Model sang danh sách DTO
    var walksDto = mapper.Map<List<WalkDto>>(walksDomainModel);

    return Ok(walksDto);
}
```

```csharp
// Repositories/SQLWalkRepository.cs
// ... (trong SQLWalkRepository)

public async Task<List<Walk>> GetAllAsync()
{
    // Bao gồm (Include) Navigation Properties để lấy dữ liệu liên quan
    // Sử dụng biểu thức lambda (x => x.Property) là cách type-safe hơn
    // so với chuỗi (Include("PropertyName")) và được khuyến khích.
    return await dbContext.Walks
        .Include(x => x.Difficulty) // Tải thông tin Difficulty liên quan
        .Include(x => x.Region)     // Tải thông tin Region liên quan
        .ToListAsync();             // Thực thi truy vấn và trả về danh sách
}
```

### 4.3. Navigation Properties và Eager Loading trong Entity Framework Core

> [!NOTE]
> **Navigation Properties** trong Entity Framework Core là các thuộc tính cho phép bạn điều hướng giữa các thực thể liên quan trong mô hình miền. Chúng biểu thị mối quan hệ giữa hai thực thể (ví dụ: một `Walk` có một `Difficulty` và một `Region`).
>
> **Ví dụ trong `Walk` Domain Model:**
> ```csharp
> // Models/Domain/Walk.cs
> using System;
>
> namespace NzWalks.API.Models.Domain
> {
>     public class Walk
>     {
>         public Guid Id { get; set; }
>         public string Name { get; set; }
>         public string Description { get; set; }
>         public double LengthInKm { get; set; }
>         public string? WalkImageUrl { get; set; }
>
>         public Guid DifficultyId { get; set; } // Foreign Key
>         public Guid RegionId { get; set; }     // Foreign Key
>
>         // Navigation Properties
>         public Difficulty Difficulty { get; set; } // Mối quan hệ 1-n (Walks -> Difficulty)
>         public Region Region { get; set; }         // Mối quan hệ 1-n (Walks -> Region)
>     }
> }
> ```
>
> **Vấn đề với Loading dữ liệu liên quan:**
> Theo mặc định, khi bạn truy vấn một thực thể (ví dụ: `Walk`), các Navigation Properties của nó sẽ không được tải cùng lúc. Điều này được gọi là **Lazy Loading**. Trong các phiên bản EF Core 3.0+ trở lên, Lazy Loading không được bật theo mặc định cho các Navigation Properties không ảo (non-virtual). Nếu bạn không tải dữ liệu liên quan, các thuộc tính `Difficulty` và `Region` trong đối tượng `Walk` sẽ là `null` hoặc không chứa dữ liệu đầy đủ, dẫn đến lỗi khi cố gắng truy cập `walk.Difficulty.Name`.
>
> **Giải pháp: Eager Loading với `Include()`:**
> Để tải dữ liệu của các thực thể liên quan cùng với thực thể chính trong một truy vấn duy nhất, bạn cần sử dụng kỹ thuật **Eager Loading** với phương thức `Include()`.
>
> *   `Include()` cho phép bạn chỉ định các Navigation Properties cần được tải cùng với thực thể chính.
> *   Bạn có thể xâu chuỗi nhiều lệnh `Include()` để tải nhiều cấp độ dữ liệu liên quan (ví dụ: `Include(x => x.Region).ThenInclude(r => r.Country)`).
> *   `Include()` có thể nhận một chuỗi (tên thuộc tính) hoặc một biểu thức lambda (type-safe). Sử dụng biểu thức lambda (`x => x.Property`) được khuyến khích mạnh mẽ để tránh lỗi chính tả và tận dụng kiểm tra thời gian biên dịch.
>
> **Ví dụ:**
> Khi `WalksController.GetAll()` gọi `walkRepository.GetAllAsync()`, phương thức này trong `SQLWalkRepository` sẽ sử dụng `.Include(x => x.Difficulty).Include(x => x.Region)` để đảm bảo rằng thông tin chi tiết về `Difficulty` và `Region` cũng được tải từ cơ sở dữ liệu và đính kèm vào mỗi đối tượng `Walk`. Điều này giúp client nhận được đầy đủ thông tin về hành trình, bao gồm tên của độ khó và khu vực, thay vì chỉ các ID, khi chúng được ánh xạ sang `WalkDto` và trả về.

### 4.4. Lấy Hành trình theo ID (Get Walk By Id - HTTP GET)

Phương thức này sẽ lấy thông tin chi tiết của một hành trình cụ thể dựa trên ID của nó.

*   **HTTP Verb:** `GET`
*   **Endpoint:** `/api/walks/{id}`
*   **Yêu cầu:** Nhận `id` (kiểu `Guid`) từ tuyến đường (route).
*   **Phản hồi:** `200 OK` kèm theo `WalkDto` nếu tìm thấy, hoặc `404 Not Found` nếu không tìm thấy.
*   **Logic:**
    1.  Gọi phương thức `GetByIdAsync` của `IWalkRepository` với ID được cung cấp.
    2.  `GetByIdAsync` cũng sẽ sử dụng `Include` để tải dữ liệu liên quan.
    3.  Kiểm tra nếu kết quả trả về là `null` (không tìm thấy hành trình) thì trả về `404 Not Found`.
    4.  Nếu tìm thấy, ánh xạ `Walk` Domain Model sang `WalkDto`.
    5.  Trả về phản hồi `200 OK` kèm theo `WalkDto`.

```csharp
// Controllers/WalksController.cs
// ... (trong WalksController)

/// <summary>
/// Lấy thông tin chi tiết của một hành trình theo ID.
/// </summary>
/// <param name="id">ID của hành trình cần tìm kiếm.</param>
/// <returns>Thông tin hành trình đã tìm thấy và trạng thái 200 OK, hoặc 404 Not Found.</returns>
[HttpGet]
[Route("{id:Guid}")] // Định nghĩa route với tham số ID kiểu Guid
public async Task<IActionResult> GetById([FromRoute] Guid id)
{
    // Gọi Repository để lấy hành trình theo ID từ cơ sở dữ liệu
    var walkDomainModel = await walkRepository.GetByIdAsync(id);

    // Kiểm tra nếu không tìm thấy hành trình
    if (walkDomainModel == null)
    {
        return NotFound(); // Trả về 404 Not Found
    }

    // Ánh xạ Domain Model sang DTO
    var walkDto = mapper.Map<WalkDto>(walkDomainModel);

    return Ok(walkDto); // Trả về 200 OK với DTO
}
```

```csharp
// Repositories/SQLWalkRepository.cs
// ... (trong SQLWalkRepository)

public async Task<Walk?> GetByIdAsync(Guid id)
{
    // Sử dụng FirstOrDefaultAsync để tìm một hành trình theo ID
    // và bao gồm các Navigation Properties. Nếu không tìm thấy, trả về null.
    return await dbContext.Walks
        .Include(x => x.Difficulty)
        .Include(x => x.Region)
        .FirstOrDefaultAsync(x => x.Id == id);
}
```

### 4.5. Cập nhật Hành trình (Update Walk - HTTP PUT)

Phương thức này cho phép client cập nhật thông tin của một hành trình hiện có.

*   **HTTP Verb:** `PUT`
*   **Endpoint:** `/api/walks/{id}`
*   **Yêu cầu:** Nhận `id` (kiểu `Guid`) từ tuyến đường và `UpdateWalkRequestDto` từ phần thân.
*   **Phản hồi:** `200 OK` kèm theo `WalkDto` của hành trình đã cập nhật nếu thành công, hoặc `404 Not Found` nếu không tìm thấy hành trình.
*   **Logic:**
    1.  Ánh xạ `UpdateWalkRequestDto` nhận được từ client sang `Walk` Domain Model.
    2.  Gọi phương thức `UpdateAsync` của `IWalkRepository` với ID và Domain Model đã cập nhật.
    3.  Kiểm tra nếu kết quả trả về là `null` (không tìm thấy hành trình để cập nhật) thì trả về `404 Not Found`.
    4.  Nếu cập nhật thành công, ánh xạ `Walk` Domain Model kết quả sang `WalkDto`.
    5.  Trả về phản hồi `200 OK` kèm theo `WalkDto` đã cập nhật.

```csharp
// Models/DTO/UpdateWalkRequestDto.cs
namespace NzWalks.API.Models.DTO
{
    public class UpdateWalkRequestDto
    {
        public string Name { get; set; }
        public string Description { get; set; }
        public double LengthInKm { get; set; }
        public string? WalkImageUrl { get; set; }
        public Guid DifficultyId { get; set; }
        public Guid RegionId { get; set; }
    }
}
```

```csharp
// Controllers/WalksController.cs
// ... (trong WalksController)

/// <summary>
/// Cập nhật thông tin của một hành trình hiện có.
/// </summary>
/// <param name="id">ID của hành trình cần cập nhật.</param>
/// <param name="updateWalkRequestDto">Dữ liệu cập nhật cho hành trình từ client.</param>
/// <returns>Thông tin hành trình đã cập nhật và trạng thái 200 OK, hoặc 404 Not Found.</returns>
[HttpPut]
[Route("{id:Guid}")]
public async Task<IActionResult> Update([FromRoute] Guid id, [FromBody] UpdateWalkRequestDto updateWalkRequestDto)
{
    // [Bước 1] Ánh xạ DTO sang Domain Model
    var walkDomainModel = mapper.Map<Walk>(updateWalkRequestDto);

    // [Bước 2] Gọi Repository để cập nhật hành trình trong cơ sở dữ liệu
    walkDomainModel = await walkRepository.UpdateAsync(id, walkDomainModel);

    // Kiểm tra nếu không tìm thấy hành trình để cập nhật
    if (walkDomainModel == null)
    {
        return NotFound(); // Trả về 404 Not Found
    }

    // [Bước 3] Ánh xạ Domain Model kết quả sang DTO
    var walkDto = mapper.Map<WalkDto>(walkDomainModel);

    return Ok(walkDto); // Trả về 200 OK với DTO đã cập nhật
}
```

```csharp
// Repositories/SQLWalkRepository.cs
// ... (trong SQLWalkRepository)

public async Task<Walk?> UpdateAsync(Guid id, Walk walk)
{
    // Tìm hành trình hiện có trong DB
    var existingWalk = await dbContext.Walks.FirstOrDefaultAsync(x => x.Id == id);

    if (existingWalk == null)
    {
        return null; // Không tìm thấy hành trình để cập nhật
    }

    // Cập nhật các thuộc tính của hành trình hiện có từ đối tượng walk mới
    existingWalk.Name = walk.Name;
    existingWalk.Description = walk.Description;
    existingWalk.LengthInKm = walk.LengthInKm;
    existingWalk.WalkImageUrl = walk.WalkImageUrl;
    existingWalk.DifficultyId = walk.DifficultyId;
    existingWalk.RegionId = walk.RegionId;

    await dbContext.SaveChangesAsync(); // Lưu thay đổi vào cơ sở dữ liệu
    return existingWalk;               // Trả về thực thể đã được cập nhật
}
```

### 4.6. Xóa Hành trình (Delete Walk - HTTP DELETE)

Phương thức này cho phép client xóa một hành trình cụ thể khỏi cơ sở dữ liệu.

*   **HTTP Verb:** `DELETE`
*   **Endpoint:** `/api/walks/{id}`
*   **Yêu cầu:** Nhận `id` (kiểu `Guid`) từ tuyến đường.
*   **Phản hồi:** `200 OK` kèm theo `WalkDto` của hành trình đã xóa nếu thành công, hoặc `404 Not Found` nếu không tìm thấy hành trình.
*   **Logic:**
    1.  Gọi phương thức `DeleteAsync` của `IWalkRepository` với ID được cung cấp.
    2.  Kiểm tra nếu kết quả trả về là `null` (không tìm thấy hành trình để xóa) thì trả về `404 Not Found`.
    3.  Nếu xóa thành công, ánh xạ `Walk` Domain Model đã xóa sang `WalkDto`.
    4.  Trả về phản hồi `200 OK` kèm theo `WalkDto` của hành trình đã xóa.

```csharp
// Controllers/WalksController.cs
// ... (trong WalksController)

/// <summary>
/// Xóa một hành trình theo ID.
/// </summary>
/// <param name="id">ID của hành trình cần xóa.</param>
/// <returns>Thông tin hành trình đã xóa và trạng thái 200 OK, hoặc 404 Not Found.</returns>
[HttpDelete]
[Route("{id:Guid}")]
public async Task<IActionResult> Delete([FromRoute] Guid id)
{
    // Gọi Repository để xóa hành trình từ cơ sở dữ liệu
    var deletedWalkDomainModel = await walkRepository.DeleteAsync(id);

    // Kiểm tra nếu không tìm thấy hành trình để xóa
    if (deletedWalkDomainModel == null)
    {
        return NotFound(); // Trả về 404 Not Found
    }

    // Ánh xạ Domain Model đã xóa sang DTO
    var deletedWalkDto = mapper.Map<WalkDto>(deletedWalkDomainModel);

    return Ok(deletedWalkDto); // Trả về 200 OK với DTO của hành trình đã xóa
}
```

```csharp
// Repositories/SQLWalkRepository.cs
// ... (trong SQLWalkRepository)

public async Task<Walk?> DeleteAsync(Guid id)
{
    // Tìm hành trình hiện có trong DB
    var existingWalk = await dbContext.Walks.FirstOrDefaultAsync(x => x.Id == id);

    if (existingWalk == null)
    {
        return null; // Không tìm thấy hành trình để xóa
    }

    dbContext.Walks.Remove(existingWalk); // Đánh dấu thực thể để xóa
    await dbContext.SaveChangesAsync();   // Lưu thay đổi vào cơ sở dữ liệu
    return existingWalk;                 // Trả về thực thể đã xóa
}
```

## 5. Tóm tắt & Hướng tới Kiến trúc Vững chắc

Trong Phần 7 này, chúng ta đã xây dựng một API RESTful đầy đủ chức năng cho tài nguyên "Hành trình" (Walks) bằng cách áp dụng một cách có hệ thống các nguyên tắc kiến trúc và công nghệ cốt lõi của ASP.NET Core và Entity Framework Core:

*   **Data Seeding:** Đã triển khai cơ chế seeding dữ liệu ban đầu cho các thực thể `Difficulty` và `Region` bằng cách sử dụng phương thức `OnModelCreating` của `DbContext` và các lệnh Migration của EF Core, đảm bảo dữ liệu tham chiếu luôn sẵn sàng.
*   **Kiến trúc API RESTful:** Đã củng cố sự hiểu biết về vai trò của Controllers, Action Methods và HTTP Verbs trong việc xây dựng API RESTful tuân thủ các nguyên tắc REST.
*   **Dependency Injection (DI):** Đã tận dụng DI để quản lý và cung cấp các phụ thuộc như `IWalkRepository` và `IMapper` cho `WalksController`, giúp giảm khớp nối và tăng khả năng kiểm thử.
*   **Repository Pattern:** Đã thiết kế và triển khai Repository Pattern với giao diện `IWalkRepository` và lớp triển khai `SQLWalkRepository` để tách biệt logic truy cập dữ liệu khỏi logic nghiệp vụ, mang lại sự linh hoạt và dễ bảo trì.
*   **Data Transfer Objects (DTOs):** Đã tạo các DTO (`AddWalkRequestDto`, `UpdateWalkRequestDto`, `WalkDto`, `RegionDto`, `DifficultyDto`) để định hình dữ liệu đầu vào và đầu ra của API, tăng cường bảo mật, linh hoạt và kiểm soát hợp đồng API.
*   **AutoMapper:** Đã sử dụng AutoMapper để tự động ánh xạ giữa các Domain Model và DTO, giảm thiểu mã lặp lại và tăng năng suất phát triển.
*   **Navigation Properties & Eager Loading:** Đã khám phá và áp dụng Navigation Properties cùng với phương thức `Include()` của Entity Framework Core để tải dữ liệu liên quan (Eager Loading) từ các bảng `Difficulty` và `Region` một cách hiệu quả khi truy vấn các hành trình.
*   **Thao tác CRUD hoàn chỉnh:** Đã hoàn thành các phương thức hành động cho tất cả các thao tác CRUD:
    *   `POST /api/walks` để **Tạo** hành trình mới.
    *   `GET /api/walks` để **Lấy tất cả** hành trình.
    *   `GET /api/walks/{id}` để **Lấy** một hành trình theo ID.
    *   `PUT /api/walks/{id}` để **Cập nhật** một hành trình hiện có.
    *   `DELETE /api/walks/{id}` để **Xóa** một hành trình.

Phần này đã trang bị cho bạn kiến thức và kỹ năng cần thiết để xây dựng các API RESTful phức tạp hơn với kiến trúc vững chắc, dễ bảo trì và mở rộng. Việc tuân thủ các nguyên tắc thiết kế này không chỉ giúp bạn viết mã sạch hơn mà còn nâng cao "Vibe Coding" của toàn bộ dự án, giúp các công cụ lập trình AI như Antigravity IDE dễ dàng hiểu, hỗ trợ và tối ưu hóa quy trình phát triển của bạn.

<!-- REVIEWED_BY_AGENT -->
