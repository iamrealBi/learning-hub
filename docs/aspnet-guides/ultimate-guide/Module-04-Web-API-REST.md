# 📘 Module 04: Web API & RESTful Services
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [Web API là gì? — Nhà hàng Takeaway](#1-web-api--nhà-hàng-takeaway)
2. [REST — Bộ luật ứng xử quốc tế](#2-rest)
3. [Tạo API Controller — Quầy Takeaway đầu tiên](#3-tạo-api-controller)
4. [Status Codes — Mã đáp án](#4-status-codes)
5. [Validation trong API](#5-validation-trong-api)
6. [Versioning & Swagger — Sổ menu theo phiên bản](#6-versioning--swagger)
7. [Error Handling — Chuông báo cháy](#7-error-handling)
8. [Content Negotiation — Khách muốn ăn kiểu gì?](#8-content-negotiation)
9. [CORS — Cho khách nước ngoài vào quán](#9-cors)
10. [Rate Limiting — Chống spam cổng](#10-rate-limiting)
11. [Caching — Tủ đông tăng tốc](#11-caching)
12. [Testing API](#12-testing-api)

---

# 1. Web API — Nhà hàng Takeaway

Ở Module 01, bạn đã biết MVC Controller = **nhà hàng phục vụ tại chỗ** — nấu xong bày đĩa đẹp (HTML) cho khách xem tại bàn.

**API Controller** = **quầy Takeaway** — nấu xong đóng hộp xốp (JSON) cho khách **mang về tự ăn**. Khách ở đây có thể là app mobile, trang React, hoặc máy IoT — bất cứ ai biết đọc hộp xốp JSON.

| | Phục vụ tại chỗ (MVC) | Takeaway (API) |
|---|---|---|
| **Khách** | Trình duyệt (con người) | Mobile app, React, máy móc |
| **Đóng gói** | Bày đĩa đẹp (HTML View) | Hộp xốp (JSON) |
| **Base class** | `Controller` | `ControllerBase` (nhẹ hơn) |
| **Dấu hiệu** | Không có | `[ApiController]` |

> **JSON là gì?** Cách đóng gói dữ liệu ai cũng đọc được: `{"Ten": "Phở", "Gia": 50000}`. Như cái hóa đơn viết rõ ràng, bất kỳ quốc gia nào cũng hiểu.

---

# 2. REST — Bộ luật ứng xử quốc tế

**REST** = quy ước quốc tế để tất cả quán Takeaway trên thế giới phục vụ theo cùng 1 cách. Nếu bạn biết gọi Takeaway ở Việt Nam, bạn cũng biết gọi ở Nhật.

## 2.1. Quy tắc 1: URL = Tên món, KHÔNG PHẢI hành động

```
❌ /api/layDanhSachMon, /api/taoMonMoi
✅ /api/mon-an  (URL chỉ là TÊN MÓN — "Tôi muốn gì đó với món ăn")
```

## 2.2. Quy tắc 2: Hành động nằm ở cách gọi (HTTP Method)

| Cách gọi | URL | Ý nghĩa | Ngoài đời |
|---|---|---|---|
| `GET` | `/api/mon-an` | Cho tôi XEM danh sách | "Menu có gì?" |
| `GET` | `/api/mon-an/5` | Cho tôi XEM món số 5 | "Món 5 là gì?" |
| `POST` | `/api/mon-an` | Tôi muốn TẠO MỚI món | "Thêm Phở vào menu!" |
| `PUT` | `/api/mon-an/5` | Tôi muốn SỬA TOÀN BỘ món 5 | "Đổi hết thông tin món 5" |
| `DELETE` | `/api/mon-an/5` | Tôi muốn XÓA món 5 | "Bỏ món 5 khỏi menu!" |

## 2.3. Bảng sai đúng — Kiểm tra nhanh

| Quy tắc | Đúng ✅ | Sai ❌ |
|---|---|---|
| URL là danh từ | `/api/products` | `/api/getProducts` |
| Số nhiều | `/api/products/5` | `/api/product/5` |
| Phân cấp | `/api/users/5/orders` | `/api/getUserOrders?id=5` |
| Đúng mã trả về | 201 khi tạo mới | 200 cho mọi thứ |

---

# 3. Tạo API Controller — Quầy Takeaway đầu tiên

```csharp
[ApiController]              // Biển "Quầy Takeaway"
[Route("api/[controller]")]  // Địa chỉ: /api/Products
public class ProductsController : ControllerBase
{
    private static List<Product> _kho = new()
    {
        new Product { Id = 1, Name = "Laptop", Price = 999 },
        new Product { Id = 2, Name = "Mouse", Price = 29 }
    };

    // GET /api/products → "Cho xem menu"
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetAll() => Ok(_kho);

    // GET /api/products/1 → "Cho xem món 1"
    [HttpGet("{id}")]
    public ActionResult<Product> GetById(int id)
    {
        var sp = _kho.FirstOrDefault(p => p.Id == id);
        if (sp == null) return NotFound();  // 404 — "Không có món này!"
        return Ok(sp);                      // 200 — "Đây, món của bạn"
    }

    // POST /api/products → "Thêm món mới vào menu"
    [HttpPost]
    public ActionResult<Product> Create([FromBody] Product sp)
    {
        sp.Id = _kho.Max(p => p.Id) + 1;
        _kho.Add(sp);
        // 201 — "Đã thêm! Xem món mới ở đường dẫn này nhé"
        return CreatedAtAction(nameof(GetById), new { id = sp.Id }, sp);
    }

    // PUT /api/products/1 → "Sửa toàn bộ món 1"
    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] Product update)
    {
        var existing = _kho.FirstOrDefault(p => p.Id == id);
        if (existing == null) return NotFound();
        existing.Name = update.Name;
        existing.Price = update.Price;
        return NoContent();  // 204 — "Đã sửa xong, không cần nói gì thêm"
    }

    // DELETE /api/products/1 → "Xóa món 1"
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        var sp = _kho.FirstOrDefault(p => p.Id == id);
        if (sp == null) return NotFound();
        _kho.Remove(sp);
        return NoContent();
    }
}
```

---

# 4. Status Codes — Mã đáp án

API trả mã số cho máy đọc, giống đèn giao thông cho xe:

| Đèn | Mã | Ý nghĩa | Helper method |
|---|---|---|---|
| 🟢 | **200** | "Tốt lắm, đây kết quả!" | `Ok(data)` |
| 🟢 | **201** | "Đã tạo mới! Xem ở đây nhé" | `Created(...)` |
| 🟢 | **204** | "Xong rồi, không cần nói gì thêm" | `NoContent()` |
| 🟡 | **400** | "Bạn gửi sai cái gì đó!" | `BadRequest()` |
| 🟡 | **401** | "Đưa thẻ trước rồi mới phục vụ!" | `Unauthorized()` |
| 🟡 | **403** | "Thẻ bạn không đủ quyền!" | `Forbid()` |
| 🟡 | **404** | "Không tìm thấy!" | `NotFound()` |
| 🔴 | **500** | "Bếp đang cháy, xin lỗi!" | `Problem(...)` |

---

# 5. Validation trong API

```csharp
public class CreateProductDto
{
    [Required(ErrorMessage = "Tên bắt buộc điền!")]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; }

    [Range(0, 999999, ErrorMessage = "Giá phải từ 0 đến 999,999")]
    public decimal Price { get; set; }
}
```

Khi có `[ApiController]` → phiên dịch viên VIP tự trả 400 nếu data sai. Không cần viết `if (!ModelState.IsValid)`.

---

# 6. Versioning & Swagger

## 6.1. API Versioning — Sổ menu theo phiên bản

Tại sao? App mobile đang dùng menu Mùa Hè (V1). Bạn sửa menu → app **CHẾT**. Giải pháp: giữ menu cũ, phát menu mới song song.

```csharp
[Route("api/v1/[controller]")]
public class ProductsV1Controller : ControllerBase { ... } // Menu cũ

[Route("api/v2/[controller]")]
public class ProductsV2Controller : ControllerBase { ... } // Menu mới, thêm ảnh
```

## 6.2. Swagger — Bảng menu điện tử

Trang web tương tác liệt kê TẤT CẢ API + cho phép test trực tiếp. Truy cập `https://localhost:xxxx/swagger`.

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
if (app.Environment.IsDevelopment()) { app.UseSwagger(); app.UseSwaggerUI(); }
```

---

# 7. Error Handling — Chuông báo cháy toàn quán

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";
        await context.Response.WriteAsJsonAsync(new
        {
            Error = "Hệ thống bảo trì, thử lại sau"
        });
        // Log lỗi thật cho dev — ĐỪNG BAO GIỜ trả stack trace cho khách hàng!
    });
});
```

> ⚠️ **KHÔNG BAO GIỜ** trả raw StackTrace cho client → Hacker thấy địa chỉ Database, connection string → Nguy hiểm!

---

# 8. Content Negotiation — Khách muốn ăn kiểu gì?

Khách gửi header `Accept` → Quán đóng gói đúng kiểu:
- `Accept: application/json` → Đóng hộp JSON 📦
- `Accept: application/xml` → Đóng hộp XML 📦

```csharp
builder.Services.AddControllers().AddXmlSerializerFormatters(); // Bật hỗ trợ XML
```

---

# 9. CORS — Cho khách nước ngoài vào quán

## 9.1. Vấn đề
Trình duyệt có luật: JS trên `cua-hang.com` **KHÔNG ĐƯỢC** gọi API ở `api.kho-hang.com`. Đây là **hàng rào an ninh** chống hacker, nhưng cũng chặn luôn dev muốn tách Frontend/Backend.

**CORS** = Quán nói: "Tôi tin tưởng `cua-hang.com`, cho nó gọi API của tôi."

## 9.2. Cấu hình

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("ChoPhepCuaHang", policy =>
    {
        policy.WithOrigins("https://cua-hang.com", "http://localhost:3000")
              .WithMethods("GET", "POST", "PUT", "DELETE")
              .WithHeaders("Content-Type", "Authorization")
              .AllowCredentials(); // Cho gửi cookie kèm
    });
});
app.UseCors("ChoPhepCuaHang");
```

## 9.3. Preflight Request — "Dò đường trước"
Trước POST/PUT/DELETE, trình duyệt tự gửi 1 request `OPTIONS` hỏi: "Tao có quyền gọi POST không?". Server OK → request thật mới gửi. Như gọi điện hỏi trước: "Quán còn phở không?" → "Còn" → mới chạy đến.

---

# 10. Rate Limiting — Chống spam cổng

Ông hàng xóm phá hoại → gọi API 10,000 lần/phút → Server quá tải. **Rate Limiting** = lắp **máy đếm ở cổng**: "Mỗi người tối đa 10 lần/phút. Quá → xin lỗi, về đi!"

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("gioi-han", opt =>
    {
        opt.PermitLimit = 10;                     // Tối đa 10 lần
        opt.Window = TimeSpan.FromMinutes(1);     // Trong 1 phút
        opt.QueueLimit = 2;                       // Cho xếp hàng 2 người
    });
    options.OnRejected = async (ctx, _) =>
    {
        ctx.HttpContext.Response.StatusCode = 429; // "Quá nhiều rồi!"
        await ctx.HttpContext.Response.WriteAsync("Bạn gọi quá nhiều! Chờ 1 phút.");
    };
});
app.UseRateLimiter();

[EnableRateLimiting("gioi-han")]
public IActionResult GetData() => Ok("data");
```

| Kiểu máy đếm | Ẩn dụ |
|---|---|
| **Fixed Window** | "Cứ mỗi phút reset đếm về 0" |
| **Sliding Window** | "Tính trượt liên tục, không reset đột ngột" |
| **Token Bucket** | "Mỗi giây thêm 1 xu, mỗi lần gọi tốn 1 xu" |
| **Concurrency** | "Tối đa 5 người trong quán cùng lúc" |

---

# 11. Caching — Tủ đông tăng tốc

## 11.1. Vấn đề
1000 khách cùng hỏi "Menu có gì?" → Server chạy vào kho 1000 lần → mệt!

**Cache** = Tủ đông: Lần 1 chạy vào kho → lấy menu → **bỏ vào tủ đông**. 999 lần sau mở tủ đông lấy → **0 giây!**

## 11.2. In-Memory Cache — Tủ đông trong quán
```csharp
builder.Services.AddMemoryCache();

public class ProductController(IMemoryCache tuDong) : ControllerBase
{
    [HttpGet("hot")]
    public IActionResult GetHot()
    {
        if (!tuDong.TryGetValue("mon-hot", out List<Product> ds))
        {
            ds = QueryDatabase(); // Chạy vào kho (lần đầu)
            tuDong.Set("mon-hot", ds, new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5), // Hết hạn sau 5 phút
                SlidingExpiration = TimeSpan.FromMinutes(2)                // Không ai mở 2 phút → bỏ
            });
        }
        return Ok(ds); // Lấy từ tủ đông — siêu nhanh!
    }
}
```

## 11.3. Distributed Cache (Redis) — Tủ đông dùng chung

Quán mở 3 chi nhánh → mỗi chi nhánh có tủ đông riêng → menu khác nhau! **Redis** = tủ đông **TRUNG TÂM** dùng chung.

```csharp
builder.Services.AddStackExchangeRedisCache(opts =>
{
    opts.Configuration = "localhost:6379";
    opts.InstanceName = "QuanAn_";
});
```

---

# 12. Testing API

## 12.1. Postman — Đóng vai khách hàng
Phần mềm giả làm mobile app: gõ URL, chọn GET/POST, nhập JSON → bấm Send → xem response.

## 12.2. Unit Test — Test từng nhân viên

```csharp
[Fact]
public void GetAll_TraVeOkVaSanPham()
{
    var fakeKho = new Mock<IProductRepository>();
    fakeKho.Setup(k => k.GetAll()).Returns(new List<Product> { new() { Name = "Laptop" } });
    var controller = new ProductController(fakeKho.Object);

    var result = controller.GetAll() as OkObjectResult;
    Assert.NotNull(result);
    Assert.Equal(200, result.StatusCode);
}
```

## 12.3. Integration Test — Test cả quán

```csharp
public class ApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public ApiTests(WebApplicationFactory<Program> factory)
        => _client = factory.CreateClient(); // Mở quán ảo!

    [Fact]
    public async Task GetProducts_TraVe200()
    {
        var response = await _client.GetAsync("/api/products");
        response.EnsureSuccessStatusCode(); // Phải thành công!
    }
}
```

---

> 🎯 **Module tiếp theo**: [Module 05 — Entity Framework Core](Module-05-Entity-Framework-Core.md)
