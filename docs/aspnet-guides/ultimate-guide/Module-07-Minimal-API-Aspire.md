# 📘 Module 07: Minimal API & .NET Aspire
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [Minimal API — Quán ăn vỉa hè siêu nhanh](#1-minimal-api)
2. [So sánh với MVC](#2-so-sánh)
3. [Parameter Binding — Nhận order](#3-parameter-binding)
4. [Tổ chức code — Từ vỉa hè lên nhà hàng](#4-tổ-chức-code)
5. [.NET Aspire — Tổng quản chuỗi nhà hàng](#5-net-aspire)
6. [Bài tập: API Ngân hàng](#6-bài-tập)

---

# 1. Minimal API — Quán ăn vỉa hè siêu nhanh

MVC Controller = nhà hàng sang trọng — phải thuê Tiếp Tân, chia phòng, tổ chức bài bản. Mở quán mất cả tuần.

**Minimal API** = quán vỉa hè — kê 1 cái bàn, 1 cái nồi, phục vụ ngay! Không cần Tiếp Tân, không cần phòng riêng. Mọi thứ chỉ trong 1 file `Program.cs`:

```csharp
var app = WebApplication.CreateBuilder(args).Build();

app.MapGet("/", () => "Xin chào! 🍜");
app.MapGet("/mon/{id}", (int id) => $"Món số {id}");
app.MapPost("/mon", (MonDto dto) => Results.Created($"/mon/{dto.Id}", dto));

app.Run(); // Mở quán ngay lập tức!
```

---

# 2. So sánh

| | Nhà hàng (MVC) | Vỉa hè (Minimal API) |
|---|---|---|
| **Chuẩn bị** | Tạo Class, gắn biển, viết hàm | Lambda 1 dòng |
| **Tốc độ mở quán** | Chậm hơn | **Nhanh hơn** |
| **Phục vụ** | Bài bản, mở rộng dễ | Nhanh gọn, nhỏ xinh |
| **Khi nào** | Dự án lớn, team nhiều người | Microservice, prototype, API nhỏ |

---

# 3. Parameter Binding — Nhận order

```csharp
// Khách gọi tên (Route)
app.MapGet("/users/{id}", (int id) => $"User {id}");

// Khách ghi note (Query)
app.MapGet("/search", (string? q, int page) => $"Tìm: {q}, trang {page}");
// → /search?q=pho&page=2

// Khách gửi phiếu (Body)
app.MapPost("/users", ([FromBody] UserDto dto) => Results.Ok(dto));

// Lấy từ kho nội bộ (DI)
app.MapGet("/time", (ILogger<Program> log) => { log.LogInformation("OK"); return DateTime.UtcNow; });
```

### Gom nhiều ô vào 1 tờ (AsParameters)

```csharp
public record SearchParams([FromQuery] string? Q, [FromQuery] int Page = 1);

app.MapGet("/search", ([AsParameters] SearchParams p) => $"Tìm: {p.Q}");
```

---

# 4. Tổ chức code — Từ vỉa hè lên nhà hàng

`Program.cs` phình to? Chuyển sang **Extension Methods** — như mở thêm quầy phụ:

```csharp
// Endpoints/UserEndpoints.cs — Quầy chuyên phục vụ User
public static class UserEndpoints
{
    public static IEndpointRouteBuilder MapUserEndpoints(this IEndpointRouteBuilder app)
    {
        var group = app.MapGroup("/api/users").WithTags("Users");
        group.MapGet("/", GetAll);
        group.MapPost("/", Create);
        return app;
    }

    private static async Task<IResult> GetAll(AppDbContext db)
        => Results.Ok(await db.Users.ToListAsync());

    private static async Task<IResult> Create(UserDto dto, AppDbContext db)
    {
        db.Users.Add(new User { Name = dto.Name });
        await db.SaveChangesAsync();
        return Results.Created($"/api/users/{dto.Name}", dto);
    }
}

// Program.cs — gọn bằng 2 dòng!
app.MapUserEndpoints();
app.MapOrderEndpoints();
```

---

# 5. .NET Aspire — Tổng quản chuỗi nhà hàng

## 5.1. Aspire là gì?

Bạn có chuỗi 5 quán: API 1, API 2, PostgreSQL, Redis, Worker. Mỗi sáng phải tự mở cửa từng quán, kiểm tra từng cái → **MỆT!**

**.NET Aspire** = thuê **Tổng quản** — ổng tự:
- Mở cửa tất cả quán cùng lúc
- Kiểm tra quán nào đang chạy, quán nào chết
- Dashboard giám sát: logs, traces, metrics

## 5.2. Cấu trúc Project

```
MySolution/
├── MyApp.AppHost/           ← VĂN PHÒNG TỔNG QUẢN (orchestrator)
├── MyApp.ServiceDefaults/   ← BỘ QUY TẮC CHUNG (telemetry, health checks)
├── MyApp.Api/               ← QUÁN PHÚ MỸ HƯNG (API)
└── MyApp.MigrationService/  ← ĐỘI SỬA SÀNG (chạy DB migration)
```

## 5.3. AppHost — Tổng quản ra lệnh

```csharp
// AppHost/Program.cs
var builder = DistributedApplication.CreateBuilder(args);

// Tổng quản bật lò PostgreSQL
var postgres = builder.AddPostgres("pg")
    .WithDataVolume("pg-data")   // Dữ liệu không mất khi tắt
    .AddDatabase("mydb");

// Tổng quản bật quán API — nối ống dẫn đến PostgreSQL
var api = builder.AddProject<Projects.MyApp_Api>("api")
    .WithReference(postgres);

builder.Build().Run(); // TỔNG QUẢN BẬT TẤT CẢ!
```

## 5.4. Aspire Dashboard

Chạy AppHost → tự mở trang giám sát:
- **Resources**: Quán nào chạy 🟢, quán nào chết 🔴
- **Logs**: Nhật ký tất cả quán gom về 1 chỗ
- **Traces**: Theo dõi 1 đơn hàng đi qua bao nhiêu quán

---

# 6. Bài tập: API Ngân hàng

Xây Core Banking API với Minimal API + EF Core + .NET Aspire:

```csharp
// Entity
public class Account
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public required string AccountNumber { get; set; }
    public decimal Balance { get; set; }
}

// Handler — Chuyển khoản
public class TransferHandler(AppDbContext db)
{
    public async Task<IResult> Transfer(Guid fromId, Guid toId, decimal amount)
    {
        var from = await db.Accounts.FindAsync(fromId);
        var to = await db.Accounts.FindAsync(toId);
        if (from == null || to == null) return Results.NotFound();
        if (from.Balance < amount) return Results.BadRequest("Số dư không đủ!");

        from.Balance -= amount;
        to.Balance += amount;
        await db.SaveChangesAsync(); // Cả 2 thay đổi CÙNG LÚC (transaction)
        return Results.Ok("Chuyển thành công!");
    }
}
```

---

> 🎯 **Module tiếp theo**: [Module 08 — Testing](Module-08-Testing.md)
