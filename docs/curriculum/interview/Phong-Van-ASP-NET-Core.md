# 🎯 Câu Hỏi Phỏng Vấn ASP.NET Core — Tiếng Việt
## 50+ câu hỏi phổ biến + đáp án chi tiết

---

# 📌 Mục lục

## [1. Kiến thức nền tảng & .NET Core](#1-kiến-thức-nền-tảng--net-core)
- [Q1: ASP.NET Core khác ASP.NET Framework?](#q1-aspnet-core-khác-aspnet-framework-ở-điểm-nào)
- [Q2: Kestrel là gì?](#q2-kestrel-là-gì-tại-sao-cần-nó)
- [Q3: Program.cs .NET 6+ khác gì?](#q3-file-programcs-trong-net-6-khác-gì-so-với-trước-net-5)
- [Q4: appsettings.json và cách đọc config](#q4-giải-thích-appsettingsjson-và-cách-đọc-config)

## [2. MVC Pattern](#2-mvc-pattern)
- [Q5: MVC là gì?](#q5-mvc-là-gì-giải-thích-vai-trò-từng-thành-phần)
- [Q6: ViewBag vs ViewData vs TempData vs ViewModel](#q6-viewbag-viewdata-tempdata-viewmodel--khác-nhau-thế-nào)
- [Q7: Razor View Engine](#q7-razor-view-engine-là-gì--có-ý-nghĩa-gì)
- [Q8: Partial View vs View Component vs Tag Helper](#q8-partial-view-view-component-tag-helper--khi-nào-dùng-cái-nào)

## [3. Routing & Middleware](#3-routing--middleware)
- [Q9: Convention vs Attribute Routing](#q9-convention-routing-vs-attribute-routing--khác-nhau-thế-nào)
- [Q10: Middleware pipeline](#q10-middleware-là-gì-pipeline-hoạt-động-thế-nào)
- [Q11: Custom middleware](#q11-viết-custom-middleware-như-thế-nào)

## [4. Dependency Injection](#4-dependency-injection)
- [Q12: DI là gì? Tại sao quan trọng?](#q12-di-dependency-injection-là-gì-tại-sao-quan-trọng)
- [Q13: Transient vs Scoped vs Singleton](#q13-3-lifetime-của-di--transient-scoped-singleton)

## [5. Entity Framework Core & Database](#5-entity-framework-core--database)
- [Q14: Code-First vs Database-First](#q14-code-first-vs-database-first--khác-nhau-thế-nào)
- [Q15: Migration workflow](#q15-migration-là-gì-workflow-ra-sao)
- [Q16: Eager vs Lazy vs Explicit Loading](#q16-eager-loading-lazy-loading-explicit-loading--khác-nhau)
- [Q17: AsNoTracking()](#q17-asnotracking-là-gì-khi-nào-nên-dùng)
- [Q18: Repository Pattern](#q18-repository-pattern-có-cần-thiết-không)

## [6. Web API & REST](#6-web-api--rest)
- [Q19: RESTful API là gì?](#q19-restful-api-là-gì-có-những-nguyên-tắc-nào)
- [Q20: [ApiController] effects](#q20-apicontroller-có-hiệu-ứng-gì)
- [Q21: Controller vs ControllerBase](#q21-phân-biệt-controller-vs-controllerbase)

## [7. Authentication & Authorization](#7-authentication--authorization)
- [Q22: Authentication vs Authorization](#q22-authentication-vs-authorization--phân-biệt)
- [Q23: JWT hoạt động thế nào?](#q23-jwt-hoạt-động-thế-nào)
- [Q24: Cookie vs Token Authentication](#q24-cookie-based-vs-token-based-authentication)
- [Q25: Role vs Claims vs Policy Authorization](#q25-phân-biệt-role-based-claims-based-policy-based-authorization)

## [8. Performance & Best Practices](#8-performance--best-practices)
- [Q26: Async/Await](#q26-asyncawait-trong-aspnet-core--tại-sao-quan-trọng)
- [Q27: Caching](#q27-caching-trong-aspnet-core)
- [Q28: CORS](#q28-cors-là-gì-khi-nào-cần-cấu-hình)

## [9. Câu hỏi tình huống](#9-câu-hỏi-tình-huống)
- [Q29: Debug app chậm](#q29-ứng-dụng-bỗng-chậm-bạn-debug-thế-nào)
- [Q30: Thiết kế API e-commerce](#q30-thiết-kế-api-cho-hệ-thống-e-commerce--bạn-làm-thế-nào)
- [Q31: Chuyển Stored Procedure → EF Core](#q31-dự-án-đang-dùng-stored-procedure-sếp-bảo-chuyển-sang-ef-core-bạn-xử-lý-thế-nào)
- [Q32: Bảo vệ API khỏi tấn công](#q32-làm-sao-bảo-vệ-api-khỏi-tấn-công)

---

# 1. Kiến thức nền tảng & .NET Core

## Q1: ASP.NET Core khác ASP.NET Framework ở điểm nào?

**Đáp án:**

| | ASP.NET Framework | ASP.NET Core |
|---|---|---|
| **Platform** | Chỉ Windows | Cross-platform (Windows, Linux, macOS) |
| **Hosting** | Chỉ IIS | Kestrel + IIS/Nginx/Apache |
| **Open-source** | Không hoàn toàn | 100% open-source |
| **Performance** | Chậm hơn | Nhanh hơn 5-10x |
| **Dependency Injection** | Cần thêm thư viện | Có sẵn (built-in) |
| **Pipeline** | HttpModule/HttpHandler | Middleware pipeline |

> **Tip phỏng vấn**: Nhấn mạnh **cross-platform** và **performance** — đây là 2 lý do lớn nhất để dùng Core.

---

## Q2: Kestrel là gì? Tại sao cần nó?

**Đáp án:** Kestrel là **web server nhẹ, cross-platform** được tích hợp sẵn trong ASP.NET Core. Nó xử lý HTTP request trực tiếp.

**Trong production**, thường đặt Kestrel ĐẰNG SAU reverse proxy (IIS, Nginx) vì:
- Nginx/IIS xử lý: SSL termination, load balancing, static files
- Kestrel xử lý: logic ứng dụng C#

```
Client → Nginx (reverse proxy) → Kestrel → ASP.NET Core App
```

---

## Q3: File `Program.cs` trong .NET 6+ khác gì so với trước (.NET 5)?

**Đáp án:**
- **.NET 5 trở xuống**: Có `Startup.cs` (ConfigureServices + Configure) + `Program.cs` (CreateHostBuilder)
- **.NET 6+**: **Minimal Hosting** — gộp tất cả vào 1 file `Program.cs`, bỏ `Startup.cs`

```csharp
// .NET 6+ (Minimal)
var builder = WebApplication.CreateBuilder(args);  // Giai đoạn: đăng ký service
builder.Services.AddControllersWithViews();
var app = builder.Build();                          // Giai đoạn: cấu hình pipeline
app.MapControllerRoute(...);
app.Run();
```

> `.NET 6+` vẫn HỖ TRỢ Startup.cs nếu muốn, nhưng template mặc định không tạo nữa.

---

## Q4: Giải thích `appsettings.json` và cách đọc config.

**Đáp án:** `appsettings.json` là file cấu hình trung tâm (thay thế `web.config` cũ). Hỗ trợ **phân môi trường**: `appsettings.Development.json` ghi đè `appsettings.json` khi chạy Development.

```csharp
// Đọc 1 giá trị:
var connStr = builder.Configuration.GetConnectionString("Default");

// Đọc 1 section → bind vào class (Options Pattern):
builder.Services.Configure<JwtSettings>(builder.Configuration.GetSection("Jwt"));
// Inject: IOptions<JwtSettings> options → options.Value.SecretKey
```

**Thứ tự ưu tiên config (sau ghi đè trước):**
`appsettings.json` → `appsettings.{Env}.json` → Environment Variables → Command-line args

---

# 2. MVC Pattern

## Q5: MVC là gì? Giải thích vai trò từng thành phần.

**Đáp án:** MVC = **Model-View-Controller** — chia ứng dụng thành 3 phần độc lập:

| Thành phần | Vai trò | Ẩn dụ |
|---|---|---|
| **Model** | Dữ liệu + logic nghiệp vụ | Bếp — nơi nấu ăn |
| **View** | Giao diện (HTML/Razor) | Phòng ăn — khách thấy |
| **Controller** | Điều phối (nhận request → gọi Model → chọn View) | Phục vụ bàn — kết nối bếp với khách |

**Luồng**: Request → Controller nhận → gọi Model lấy data → truyền data cho View → trả HTML.

> **Tip**: Đừng để logic nghiệp vụ trong Controller (Fat Controller Anti-Pattern). Controller chỉ nên điều phối.

---

## Q6: ViewBag, ViewData, TempData, ViewModel — khác nhau thế nào?

**Đáp án:**

| | ViewBag | ViewData | TempData | ViewModel |
|---|---|---|---|---|
| **Kiểu** | `dynamic` | `Dictionary` | `Dictionary` | Strongly typed |
| **Scope** | 1 request | 1 request | Qua redirect | 1 request |
| **Compile-time check** | ❌ | ❌ | ❌ | ✅ |
| **Dùng khi** | Data phụ nhỏ | Data phụ nhỏ | Thông báo sau redirect | Data chính cho View |

```csharp
// ❌ Anti-pattern:
ViewBag.Products = productList;  // Không type-safe, dễ typo

// ✅ Best practice:
return View(productViewModel);   // Strongly typed → Intellisense, compile-time check
```

---

## Q7: Razor View Engine là gì? `@` có ý nghĩa gì?

**Đáp án:** Razor là **template engine** cho phép viết C# lẫn trong HTML.

| Cú pháp | Ý nghĩa |
|---|---|
| `@Model.Name` | In giá trị C# ra HTML |
| `@{ ... }` | Khối code C# (không in) |
| `@if (...) { }` | Điều kiện |
| `@foreach (var x in list)` | Lặp |
| `@@` | Escape — in ký tự `@` literal |
| `@Html.Raw(html)` | In HTML không encode (⚠️ cẩn thận XSS!) |

---

## Q8: Partial View, View Component, Tag Helper — khi nào dùng cái nào?

**Đáp án:**

| | Partial View | View Component | Tag Helper |
|---|---|---|---|
| **Mục đích** | Tái sử dụng HTML đơn giản | Logic phức tạp + view | Extend HTML syntax |
| **Có logic?** | ❌ Không | ✅ Có (InvokeAsync) | ✅ Code-behind |
| **Ví dụ** | Header, Footer | Menu sidebar, Shopping cart | `<form asp-action="...">` |

---

# 3. Routing & Middleware

## Q9: Convention Routing vs Attribute Routing — khác nhau thế nào?

**Đáp án:**

| | Convention Routing | Attribute Routing |
|---|---|---|
| **Khai báo** | `MapControllerRoute()` trong Program.cs | `[Route]`, `[HttpGet]` trên Controller |
| **Vị trí** | Tập trung — 1 chỗ | Phân tán — trên từng action |
| **Dùng cho** | MVC (Views) | Web API |
| **Ưu tiên** | Thấp hơn | Cao hơn |

```csharp
// Convention: /Products/Edit/5
app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");

// Attribute: /api/products/5
[Route("api/[controller]")]
public class ProductsController { [HttpGet("{id}")] ... }
```

---

## Q10: Middleware là gì? Pipeline hoạt động thế nào?

**Đáp án:** Middleware = **lớp xử lý** trong chuỗi pipeline. Request đi qua từng middleware theo thứ tự, response đi ngược lại.

```
Request → Exception → HTTPS → Static → Routing → Auth → Auth → Endpoint
         ←──────────── Response đi ngược lại ──────────────←
```

**Thứ tự SỐNG CÒN (không đổi được!):**
1. `UseExceptionHandler()` — bắt lỗi
2. `UseHttpsRedirection()` — chuyển HTTPS
3. `UseStaticFiles()` — file tĩnh
4. `UseRouting()` — tìm endpoint
5. `UseAuthentication()` — xác thực (**TRƯỚC Authorization!**)
6. `UseAuthorization()` — phân quyền
7. `MapControllers()` / `MapControllerRoute()` — endpoint

---

## Q11: Viết custom middleware như thế nào?

**Đáp án:**
```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    public LoggingMiddleware(RequestDelegate next) { _next = next; }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine($"Request: {context.Request.Path}");  // TRƯỚC
        await _next(context);                                    // Gọi middleware tiếp theo
        Console.WriteLine($"Response: {context.Response.StatusCode}"); // SAU
    }
}

// Đăng ký:
app.UseMiddleware<LoggingMiddleware>();
```

---

# 4. Dependency Injection

## Q12: DI (Dependency Injection) là gì? Tại sao quan trọng?

**Đáp án:** DI = **"Thuê nhân viên qua trung tâm"** thay vì tự tạo. Class KHAI BÁO cần gì (qua constructor), DI Container TỰ ĐỘNG tạo và truyền vào.

**Không có DI (coupling chặt):**
```csharp
public class OrderController {
    private readonly OrderService _svc = new OrderService(new SqlOrderRepo(new AppDbContext()));
    // ❌ Controller tự tạo MỌI THỨ → không test được, không đổi được
}
```

**Có DI (loose coupling):**
```csharp
public class OrderController {
    private readonly IOrderService _svc;
    public OrderController(IOrderService svc) { _svc = svc; }
    // ✅ Không biết cụ thể → dễ test (mock), dễ đổi implementation
}
```

---

## Q13: 3 Lifetime của DI — Transient, Scoped, Singleton?

**Đáp án:**

| Lifetime | Tạo khi nào | Hủy khi nào | Ẩn dụ |
|---|---|---|---|
| **Transient** | Mỗi lần inject | Ngay sau khi dùng | Ly giấy — dùng 1 lần vứt |
| **Scoped** | 1 lần per HTTP request | Cuối request | Khay đồ ăn — dùng suốt bữa |
| **Singleton** | 1 lần per app | App tắt | Bếp trưởng — cả nhà hàng dùng chung |

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddScoped<IOrderService, OrderService>();     // PHỔBIẾN NHẤT
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```

> **DbContext luôn Scoped** — vì mỗi request cần 1 unit of work riêng.

---

# 5. Entity Framework Core & Database

## Q14: Code-First vs Database-First — khác nhau thế nào?

**Đáp án:**

| | Code-First | Database-First |
|---|---|---|
| **Bắt đầu từ** | Viết C# class → EF tạo DB | Có DB sẵn → EF tạo class |
| **Migration** | `dotnet ef migrations add` | `dotnet ef dbcontext scaffold` |
| **Phù hợp** | Dự án MỚI, team kiểm soát schema | Dự án có DB LEGACY sẵn |
| **Ai kiểm soát schema?** | Dev (qua code) | DBA (qua SQL) |

> **Tip**: Hầu hết dự án mới dùng **Code-First** vì linh hoạt hơn.

---

## Q15: Migration là gì? Workflow ra sao?

**Đáp án:** Migration = **"nhật ký thay đổi schema"** — mỗi lần sửa Model → tạo migration → áp dụng lên DB.

```bash
# 1. Sửa Model (thêm property, thêm class)
# 2. Tạo migration:
dotnet ef migrations add ThemCotDiaChi

# 3. Áp dụng lên DB:
dotnet ef database update

# 4. Rollback:
dotnet ef database update TenMigrationTruoc

# 5. Xóa migration chưa áp dụng:
dotnet ef migrations remove
```

---

## Q16: Eager Loading, Lazy Loading, Explicit Loading — khác nhau?

**Đáp án:**

| | Eager Loading | Lazy Loading | Explicit Loading |
|---|---|---|---|
| **Cách dùng** | `.Include()` | Tự động khi truy cập | `.Entry().Collection().Load()` |
| **Khi nào load?** | Cùng lúc query chính | Khi truy cập navigation property | Khi gọi Load() |
| **N+1?** | ❌ Không | ✅ Dễ bị! | ❌ Kiểm soát |
| **Mặc định?** | Không | Không (cần thêm package) | Không |

```csharp
// Eager: 1 query JOIN
var orders = _db.Orders.Include(o => o.Items).ToList();

// Lazy: nhiều query ngầm (NGUY HIỂM!)
var orders = _db.Orders.ToList();
foreach (var o in orders) { var items = o.Items; } // N query thêm!

// Explicit: chủ động
var order = _db.Orders.Find(1);
_db.Entry(order).Collection(o => o.Items).Load();
```

---

## Q17: `AsNoTracking()` là gì? Khi nào nên dùng?

**Đáp án:** EF Core mặc định **theo dõi** (track) mọi entity đọc từ DB → tốn bộ nhớ. `AsNoTracking()` = "đọc xong quên luôn" → **nhanh hơn 30-50%**.

```csharp
// Chỉ ĐỌC, không sửa → dùng AsNoTracking:
var products = _db.Products.AsNoTracking().ToList();

// Cần SỬA → KHÔNG dùng AsNoTracking:
var product = _db.Products.Find(id);  // tracked
product.Name = "New Name";
_db.SaveChanges();  // EF biết cần UPDATE
```

---

## Q18: Repository Pattern có cần thiết không?

**Đáp án:** **Tranh cãi lớn** trong cộng đồng!

**Ủng hộ:**
- Abstraction → dễ test (mock IRepository)
- Tách biệt data access logic

**Phản đối:**
- DbContext **ĐÃ LÀ** Repository + Unit of Work
- Thêm 1 lớp trung gian → code phức tạp hơn mà không thêm giá trị

> **Thực tế**: Dự án nhỏ-trung: KHÔNG CẦN. Dự án lớn hoặc cần đổi ORM: nên có.

---

# 6. Web API & REST

## Q19: RESTful API là gì? Có những nguyên tắc nào?

**Đáp án:** REST = Representational State Transfer — kiến trúc cho API.

| Nguyên tắc | Ý nghĩa |
|---|---|
| **Stateless** | Server không nhớ client giữa các request |
| **Resource-based** | URL đại diện resource (`/api/products/5`) |
| **HTTP Methods** | GET=đọc, POST=tạo, PUT=sửa toàn bộ, PATCH=sửa phần, DELETE=xóa |
| **Status Codes** | 200=OK, 201=Created, 400=Bad Request, 404=Not Found, 500=Server Error |

```
GET    /api/products      → Lấy tất cả
GET    /api/products/5     → Lấy 1 sản phẩm
POST   /api/products       → Tạo mới
PUT    /api/products/5     → Cập nhật toàn bộ
DELETE /api/products/5     → Xóa
```

---

## Q20: `[ApiController]` có hiệu ứng gì?

**Đáp án:** 4 hiệu ứng tự động:
1. **Auto 400**: ModelState invalid → trả 400 tự động (không cần if !ModelState.IsValid)
2. **Auto [FromBody]**: Complex type tự hiểu đọc từ body
3. **Bắt buộc Attribute Routing**: Phải có `[Route]`
4. **ProblemDetails**: Lỗi trả chuỗi chuẩn RFC 7807

---

## Q21: Phân biệt `Controller` vs `ControllerBase`?

**Đáp án:**
- `ControllerBase` = nhẹ, chỉ có `Ok()`, `NotFound()`, `BadRequest()` → cho **API**
- `Controller` = kế thừa `ControllerBase` + thêm `View()`, `PartialView()`, `ViewBag` → cho **MVC**

> API không cần View → dùng `ControllerBase` (nhẹ hơn).

---

# 7. Authentication & Authorization

## Q22: Authentication vs Authorization — phân biệt?

**Đáp án:**

| | Authentication (AuthN) | Authorization (AuthZ) |
|---|---|---|
| **Câu hỏi** | BẠN LÀ AI? | BẠN ĐƯỢC LÀM GÌ? |
| **Ẩn dụ** | Quẹt CCCD vào tòa nhà | Quẹt thẻ lên tầng nào |
| **Lỗi** | 401 Unauthorized | 403 Forbidden |
| **Middleware** | `UseAuthentication()` | `UseAuthorization()` |
| **Thuộc tính** | `[Authorize]` | `[Authorize(Roles="Admin")]` |

---

## Q23: JWT hoạt động thế nào?

**Đáp án:**
```
1. Client POST /login (username + password)
2. Server kiểm tra → đúng → tạo JWT token (Header.Payload.Signature)
3. Server trả token về client
4. Client mỗi request gửi: Authorization: Bearer <token>
5. Server middleware đọc token → kiểm tra signature → gán User.Identity
```

**3 phần JWT:**
- **Header**: `{"alg":"HS256","typ":"JWT"}` (Base64)
- **Payload**: `{"sub":"user1","role":"Admin","exp":1234567}` (Base64)
- **Signature**: `HMAC-SHA256(header.payload, secretKey)` (không decode được)

> **JWT KHÔNG MÃ HÓA** — ai cũng đọc Payload. Nhưng không thể SỬA vì không có key.

---

## Q24: Cookie-based vs Token-based Authentication?

**Đáp án:**

| | Cookie | Token (JWT) |
|---|---|---|
| **Lưu ở** | Browser tự quản lý | Client lưu (localStorage, cookie) |
| **Stateful?** | ✅ Server giữ session | ❌ Stateless |
| **CSRF** | Dễ bị tấn công | Không bị (nếu không lưu cookie) |
| **Mobile?** | Khó | Dễ (gửi header) |
| **Scale** | Cần session store | Không cần |
| **Dùng cho** | Web MVC truyền thống | API, SPA, Mobile |

---

## Q25: Phân biệt Role-based, Claims-based, Policy-based Authorization?

**Đáp án:**

| Loại | Ví dụ | Khi nào dùng |
|---|---|---|
| **Role-based** | `[Authorize(Roles="Admin")]` | Đơn giản: Admin/User/Manager |
| **Claims-based** | `User.HasClaim("Department", "IT")` | Phức tạp: thuộc tính cụ thể |
| **Policy-based** | `[Authorize(Policy="Over18")]` | Kết hợp nhiều rule |

```csharp
// Policy kết hợp:
builder.Services.AddAuthorization(options =>
    options.AddPolicy("SeniorAdmin", p =>
        p.RequireRole("Admin")
         .RequireClaim("Experience", "5+years")));
```

---

# 8. Performance & Best Practices

## Q26: Async/Await trong ASP.NET Core — tại sao quan trọng?

**Đáp án:** ASP.NET Core xử lý request bằng **thread pool** (giới hạn). Nếu mỗi request **chờ DB 200ms** mà dùng **sync** → thread bị khóa → hết thread → app chết.

**Async**: Thread gửi query → **trả về pool** làm việc khác → DB trả kết quả → thread tiếp tục.

```csharp
// ❌ Sync: khóa thread suốt 200ms
var products = _db.Products.ToList();

// ✅ Async: thread tự do trong 200ms
var products = await _db.Products.ToListAsync();
```

> **Quy tắc**: Mọi I/O (DB, file, HTTP call) → dùng async. Dùng `async Task<>` cho Action method.

---

## Q27: Caching trong ASP.NET Core?

**Đáp án:** 3 loại:

| Loại | Lưu ở | Dùng khi | Code |
|---|---|---|---|
| **In-Memory** | RAM server | 1 server, data nhỏ | `IMemoryCache` |
| **Distributed** | Redis/SQL Server | Multi-server | `IDistributedCache` |
| **Response Caching** | Client/CDN | Static content | `[ResponseCache]` |

---

## Q28: CORS là gì? Khi nào cần cấu hình?

**Đáp án:** CORS = Cross-Origin Resource Sharing. Browser chặn request từ domain A gọi API domain B (security).

```csharp
builder.Services.AddCors(options =>
    options.AddPolicy("AllowFrontend", p =>
        p.WithOrigins("https://myfrontend.com")
         .AllowAnyHeader()
         .AllowAnyMethod()));
app.UseCors("AllowFrontend");
```

> Cần khi: Frontend (React/Angular) ở domain khác với Backend API.

---

# 9. Câu hỏi tình huống

## Q29: "Ứng dụng bỗng chậm, bạn debug thế nào?"

**Đáp án mẫu:**
1. **Logging**: Kiểm tra log xem request nào chậm
2. **Profiling**: Dùng MiniProfiler hoặc Application Insights
3. **DB queries**: Bật EF Core logging → tìm N+1, slow query
4. **Caching**: Có cache chưa? Cache hit rate?
5. **Async**: Có chỗ nào dùng sync I/O không?
6. **Memory**: Memory leak? (GC pressure?)

---

## Q30: "Thiết kế API cho hệ thống e-commerce — bạn làm thế nào?"

**Đáp án mẫu:**
```
/api/products          GET=list, POST=create
/api/products/{id}     GET=detail, PUT=update, DELETE=remove
/api/products/{id}/reviews   GET=reviews
/api/cart              GET=view, POST=add item, DELETE=clear
/api/orders            GET=history, POST=checkout
/api/auth/login        POST
/api/auth/register     POST
```
- Versioning: `/api/v1/products`
- Pagination: `?page=1&pageSize=20`
- Filtering: `?category=electronics&minPrice=100`
- 401/403 cho protected endpoints
- Rate limiting cho public endpoints

---

## Q31: "Dự án đang dùng Stored Procedure, sếp bảo chuyển sang EF Core. Bạn xử lý thế nào?"

**Đáp án mẫu:**
1. **Không refactor hết 1 lần** — rủi ro cao
2. Dùng **Database-First** scaffold existing tables
3. SP phức tạp → giữ lại, gọi qua `_db.Database.ExecuteSqlRaw()`
4. SP đơn giản (CRUD) → thay bằng LINQ
5. Viết **Integration Tests** cho từng SP trước khi thay
6. Migration: song song SP + EF Core cho đến khi chuyển hết

---

## Q32: "Làm sao bảo vệ API khỏi tấn công?"

**Đáp án:**
1. **Input Validation**: `[Required]`, `[StringLength]`, custom validators
2. **SQL Injection**: EF Core parameterized queries (tự động)
3. **XSS**: Razor auto-encode (tránh `@Html.Raw`)
4. **CSRF**: `[ValidateAntiForgeryToken]` cho mọi POST form
5. **Rate Limiting**: `builder.Services.AddRateLimiter()`
6. **HTTPS**: `UseHttpsRedirection()` + HSTS
7. **JWT**: Short-lived token + Refresh token
8. **CORS**: Chỉ cho phép domain cụ thể
9. **Helmet**: Security headers (X-Content-Type-Options, X-Frame-Options)

---

> 💡 **Lời khuyên phỏng vấn:**
> - Luôn cho **ví dụ code cụ thể** — không nói lý thuyết suông
> - Nếu không biết → nói thẳng "Tôi chưa dùng X nhưng tôi biết Y tương tự"
> - Hỏi lại nếu câu hỏi mơ hồ — thể hiện tư duy phản biện
