# 📖 Thuật Ngữ & Kiến Thức Bổ Sung ASP.NET Core
## Những khái niệm QUAN TRỌNG chưa có trong bộ tài liệu study guide

---

# 📌 Mục lục

1. [Code-First vs Database-First vs Model-First](#1-code-first-vs-database-first-vs-model-first)
   - [1.1 Code-First](#11-code-first-viết-code-trước)
   - [1.2 Database-First](#12-database-first-có-db-sẵn)
   - [1.3 Model-First (lỗi thời)](#13-model-first-đã-lỗi-thời)
2. [Data Annotations vs Fluent API](#2-data-annotations-vs-fluent-api)
   - [2.1 Data Annotations](#21-data-annotations-trên-class)
   - [2.2 Fluent API](#22-fluent-api-trong-onmodelcreating)
3. [Dependency Injection — Chi tiết](#3-dependency-injection-di--chi-tiết-hơn)
   - [3.1 Ba loại lifetime](#31-ba-loại-lifetime)
   - [3.2 Interface-based DI](#32-interface-based-di)
4. [Middleware Pipeline — Chi tiết](#4-middleware-pipeline--chi-tiết)
   - [4.1 Custom Middleware](#41-custom-middleware)
   - [4.2 Thứ tự middleware](#42-thứ-tự-middleware-phải-đúng)
5. [Logging (ILogger)](#5-logging)
6. [Configuration & Options Pattern](#6-configuration--options-pattern)
7. [Deployment (Triển khai)](#7-deployment-triển-khai)
   - [7.1 Publish](#71-publish)
   - [7.2 Các cách deploy phổ biến](#72-các-cách-deploy-phổ-biến)
   - [7.3 Dockerfile](#73-dockerfile-cơ-bản)
8. [Unit Testing (xUnit + Moq)](#8-unit-testing)
9. [Design Patterns thường gặp](#9-design-patterns-thường-gặp)
10. [SignalR (Real-time)](#10-signalr-real-time)
11. [Background Tasks / Hosted Services](#11-background-tasks--hosted-services)
12. [Thuật ngữ khác hay gặp](#12-một-số-thuật-ngữ-khác-hay-gặp)
13. [Quick Reference — Những thứ hay quên](#13-bảng-quick-reference--những-thứ-hay-quên)

---

# 📋 Về nguồn gốc bộ tài liệu Study Guide

> **XÁC NHẬN**: Bộ tài liệu `aspnet-core-study-guide/` (4 Module) được tổng hợp từ **video transcripts + readings + quiz** của khóa **"Web Application Development with ASP.NET Core"** trên Coursera (như ghi ở README dòng 87-88). Đây là nội dung THỰC từ transcript, **KHÔNG phải suy luận từ tiêu đề**.
>
> Tuy nhiên, khóa Coursera có **giới hạn** — nó tập trung vào kiến thức cơ bản-trung cấp. File này bổ sung những kiến thức **thực tế khi đi làm và phỏng vấn** mà khóa học chưa đề cập hoặc chỉ lướt qua.

---

# 1. Code-First vs Database-First vs Model-First

## 1.1. Code-First (Viết code trước)

**Quy trình:** Viết class C# → EF tạo bảng từ class → Migration quản lý thay đổi.

```csharp
// 1. Viết class:
public class SanPham {
    public int Id { get; set; }
    public string TenSanPham { get; set; }
    public decimal Gia { get; set; }
}

// 2. Thêm DbSet:
public DbSet<SanPham> SanPhams { get; set; }

// 3. Tạo bảng:
dotnet ef migrations add TaoSanPham
dotnet ef database update
```

**Khi nào dùng:** Dự án MỚI, dev kiểm soát schema, muốn version control schema qua migration.

## 1.2. Database-First (Có DB sẵn)

**Quy trình:** DB đã tồn tại → Scaffold command tạo class C# từ bảng.

```bash
dotnet ef dbcontext scaffold "Host=localhost;Database=mydb;Username=postgres;Password=123" \
    Npgsql.EntityFrameworkCore.PostgreSQL -o Models -c AppDbContext
```

**Khi nào dùng:** Dự án legacy đã có DB, DBA quản lý schema, hoặc DB dùng chung nhiều hệ thống.

## 1.3. Model-First (Đã LỖI THỜI)

EF 6 cho phép thiết kế visual (EDMX designer) → tạo DB. **EF Core KHÔNG hỗ trợ**. Bỏ qua.

**Bảng so sánh:**

| | Code-First | Database-First |
|---|---|---|
| Bắt đầu từ | C# class | SQL/DB tool |
| Tạo migration | `migrations add` | `dbcontext scaffold` |
| Version control | ✅ Migration files trong Git | ❌ Schema ngoài code |
| Phù hợp | Greenfield project | Legacy system |
| Phổ biến | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

---

# 2. Data Annotations vs Fluent API

Study guide đề cập Data Annotations nhưng chưa deep dive **Fluent API**:

## 2.1. Data Annotations (Trên class)
```csharp
[Required]
[StringLength(100)]
[Column("product_name")]  // Đặt tên cột khác tên property
public string TenSanPham { get; set; }
```

## 2.2. Fluent API (Trong OnModelCreating)
```csharp
protected override void OnModelCreating(ModelBuilder builder)
{
    builder.Entity<SanPham>(entity => {
        entity.HasKey(s => s.Id);                           // Primary Key
        entity.Property(s => s.TenSanPham)
              .IsRequired()                                 // NOT NULL
              .HasMaxLength(100)                            // VARCHAR(100)
              .HasColumnName("product_name");               // Tên cột
        entity.HasIndex(s => s.TenSanPham).IsUnique();     // Unique index
        entity.HasOne(s => s.DanhMuc)                      // Relationship
              .WithMany(d => d.SanPhams)
              .HasForeignKey(s => s.DanhMucId)
              .OnDelete(DeleteBehavior.Cascade);            // Xóa cha → xóa con
    });
}
```

**Khi nào dùng cái nào?**
| | Data Annotations | Fluent API |
|---|---|---|
| Đơn giản | ✅ | Overkill |
| Composite key | ❌ Không hỗ trợ | ✅ `.HasKey(e => new {e.A, e.B})` |
| Relationship phức tạp | ❌ | ✅ |
| Index | ❌ | ✅ |
| Owned types | ❌ | ✅ |

> **Thực tế**: Dùng Data Annotations cho đơn giản (Required, StringLength). Dùng Fluent API cho relationship, index, composite key.

---

# 3. Dependency Injection (DI) — Chi tiết hơn

Study guide giới thiệu DI nhưng chưa nói rõ **3 lifetime** và **khi nào dùng gì**.

## 3.1. Ba loại lifetime

```csharp
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();
// TRANSIENT: Mỗi lần inject → tạo MỚI
// Dùng cho: Service nhẹ, stateless, không giữ data

builder.Services.AddScoped<IOrderService, OrderService>();
// SCOPED: 1 instance per HTTP request
// Dùng cho: DbContext, Service cần share data trong 1 request

builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
// SINGLETON: 1 instance cho TOÀN BỘ app
// Dùng cho: Config, Cache, Logger
// ⚠️ KHÔNG inject Scoped service vào Singleton → exception!
```

## 3.2. Interface-based DI
```csharp
// 1. Định nghĩa interface:
public interface IProductService {
    Task<List<Product>> GetAllAsync();
}

// 2. Implementation:
public class ProductService : IProductService {
    private readonly AppDbContext _db;
    public ProductService(AppDbContext db) { _db = db; }
    public async Task<List<Product>> GetAllAsync() => await _db.Products.ToListAsync();
}

// 3. Đăng ký:
builder.Services.AddScoped<IProductService, ProductService>();

// 4. Inject vào Controller:
public class ProductController(IProductService svc) : Controller { ... }
```

---

# 4. Middleware Pipeline — Chi tiết

## 4.1. Custom Middleware
```csharp
public class RequestTimingMiddleware {
    private readonly RequestDelegate _next;
    
    public RequestTimingMiddleware(RequestDelegate next) { _next = next; }

    public async Task InvokeAsync(HttpContext context) {
        var sw = Stopwatch.StartNew();
        await _next(context);           // Gọi middleware tiếp theo
        sw.Stop();
        context.Response.Headers["X-Response-Time"] = $"{sw.ElapsedMilliseconds}ms";
    }
}

// Đăng ký:
app.UseMiddleware<RequestTimingMiddleware>();
```

## 4.2. Thứ tự middleware (PHẢI đúng!)
```
ExceptionHandler → HSTS → HttpsRedirection → StaticFiles 
→ Routing → CORS → Authentication → Authorization → Endpoints
```

---

# 5. Logging

Study guide không đề cập **ILogger** chi tiết:

```csharp
public class ProductController : Controller {
    private readonly ILogger<ProductController> _logger;
    
    public ProductController(ILogger<ProductController> logger) {
        _logger = logger;
    }

    public IActionResult Index() {
        _logger.LogInformation("User truy cập trang product lúc {Time}", DateTime.Now);
        _logger.LogWarning("Product #{Id} sắp hết hàng", 42);
        _logger.LogError(ex, "Lỗi khi load products");
        return View();
    }
}
```

**6 mức log** (từ thấp → cao): `Trace → Debug → Information → Warning → Error → Critical`

---

# 6. Configuration & Options Pattern

## 6.1. Options Pattern (Đọc config type-safe)
```csharp
// appsettings.json:
{
    "Jwt": { "Key": "secret123", "Issuer": "MyApp", "ExpireMinutes": 60 }
}

// Class:
public class JwtSettings {
    public string Key { get; set; }
    public string Issuer { get; set; }
    public int ExpireMinutes { get; set; }
}

// Đăng ký:
builder.Services.Configure<JwtSettings>(builder.Configuration.GetSection("Jwt"));

// Inject:
public class AuthService(IOptions<JwtSettings> options) {
    var settings = options.Value;  // settings.Key, settings.Issuer...
}
```

---

# 7. Deployment (Triển khai)

Study guide không nói về deploy:

## 7.1. Publish
```bash
dotnet publish -c Release -o ./publish
```

## 7.2. Các cách deploy phổ biến
| Nền tảng | Cách |
|---|---|
| **IIS** (Windows) | Publish → copy vào IIS site → Application Pool = No Managed Code |
| **Azure App Service** | VS → Publish → Azure → tạo App Service |
| **Docker** | Dockerfile → `docker build` → `docker run` |
| **Linux + Nginx** | Publish → copy lên server → Nginx reverse proxy → systemd service |

## 7.3. Dockerfile cơ bản
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app
FROM runtime
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

---

# 8. Unit Testing

Study guide chưa đề cập testing:

```csharp
// Test Framework: xUnit (phổ biến nhất cho .NET)
// Mock: Moq

[Fact]
public async Task GetAll_ReturnsAllProducts() {
    // Arrange
    var mockService = new Mock<IProductService>();
    mockService.Setup(s => s.GetAllAsync()).ReturnsAsync(new List<Product> {
        new Product { Id = 1, Name = "Test" }
    });
    var controller = new ProductController(mockService.Object);

    // Act
    var result = await controller.Index() as ViewResult;

    // Assert
    var model = result.Model as List<Product>;
    Assert.Single(model);
    Assert.Equal("Test", model[0].Name);
}
```

**3 bước test**: **Arrange** (chuẩn bị) → **Act** (thực thi) → **Assert** (kiểm tra)

---

# 9. Design Patterns thường gặp

| Pattern | Ý nghĩa | Ví dụ trong ASP.NET |
|---|---|---|
| **Repository** | Tách data access khỏi business logic | `IProductRepository` + `ProductRepository` |
| **Unit of Work** | Gom nhiều repo dùng chung 1 DbContext | `IUnitOfWork { Save(); }` |
| **Factory** | Tạo object linh hoạt | `IServiceProvider`, DI container |
| **Strategy** | Đổi thuật toán runtime | `IPaymentStrategy` (credit card, momo, vnpay) |
| **Observer** | Thông báo khi có event | `IObserver<T>`, Event/Delegate |
| **Decorator** | Wrap thêm tính năng | Middleware pipeline |
| **CQRS** | Tách Read vs Write model | MediatR + Separate query/command handlers |

---

# 10. SignalR (Real-time)

Không có trong study guide nhưng rất hay hỏi phỏng vấn:

```csharp
// Hub:
public class ChatHub : Hub {
    public async Task SendMessage(string user, string message) {
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}

// Program.cs: builder.Services.AddSignalR();
// app.MapHub<ChatHub>("/chatHub");
```

**Dùng cho**: Chat, notification live, dashboard real-time, game multiplayer.

---

# 11. Background Tasks / Hosted Services

```csharp
public class CleanupService : BackgroundService {
    protected override async Task ExecuteAsync(CancellationToken ct) {
        while (!ct.IsCancellationRequested) {
            // Xóa file tạm, gửi email, cleanup...
            await Task.Delay(TimeSpan.FromHours(1), ct);
        }
    }
}
// builder.Services.AddHostedService<CleanupService>();
```

---

# 12. Một số thuật ngữ khác hay gặp

| Thuật ngữ | Giải thích |
|---|---|
| **DTO** (Data Transfer Object) | Class chỉ chứa data, dùng để truyền giữa các layer (không có logic) |
| **ViewModel** | DTO cho View — chứa đúng data View cần (không thừa, không thiếu) |
| **AutoMapper** | Thư viện tự map Entity ↔ DTO: `_mapper.Map<ProductDto>(product)` |
| **Minimal API** | .NET 6+: viết API không cần Controller: `app.MapGet("/api/hello", () => "Hi")` |
| **gRPC** | RPC framework of Google — nhanh hơn REST (dùng Protobuf thay JSON) |
| **Blazor** | C# chạy trên browser (thay JavaScript) — Server mode hoặc WASM |
| **Health Checks** | Endpoint `/health` kiểm tra app sống không → `builder.Services.AddHealthChecks()` |
| **API Versioning** | `/api/v1/products` vs `/api/v2/products` → `Asp.Versioning.Mvc` package |
| **Global Exception Handling** | Middleware bắt tất cả exception → trả JSON lỗi thống nhất |
| **Rate Limiting** | Giới hạn request/phút — chống DDoS: `.NET 7+` có built-in |
| **Swagger/OpenAPI** | Trang document + test API tự động: `Swashbuckle.AspNetCore` |
| **FluentValidation** | Thư viện validation mạnh hơn Data Annotations, tách logic validate khỏi Model |
| **MediatR** | Thư viện CQRS/Mediator pattern — tách Controller khỏi business logic |
| **EF Core Interceptors** | Hook vào SaveChanges/Query → audit, soft-delete, multi-tenant |
| **Specification Pattern** | Đóng gói query logic → tái sử dụng filter/sort/include |

---

# 13. Bảng "Quick Reference" — Những thứ hay quên

| Vấn đề | Giải pháp |
|---|---|
| Quên `await` | Compile warning → request trả về TRƯỚC khi DB xong! |
| `UseAuthentication` SAU `UseAuthorization` | [Authorize] không hoạt động → 401 vô tội vạ |
| Không có `[ValidateAntiForgeryToken]` | Dễ bị CSRF attack |
| `new DbContext()` thay vì DI | Không test được, không quản lý lifetime |
| String concatenation trong SQL | SQL Injection! Dùng parameterized query |
| Không `AsNoTracking()` cho read-only | Tốn RAM, chậm 30-50% |
| Quên `Include()` | Navigation property = null → NullReferenceException |
| Singleton inject Scoped | InvalidOperationException at runtime |
