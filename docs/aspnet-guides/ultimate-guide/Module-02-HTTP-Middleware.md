# 📘 Module 02: HTTP, Middleware & Configuration
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [Middleware Pipeline — Dây chuyền kiểm tra an ninh sân bay](#1-middleware-pipeline)
2. [HttpRequest — Bức thư khách gửi lên](#2-httprequest)
3. [HttpResponse — Bức thư quán gửi lại](#3-httpresponse)
4. [Cookie & Session — Thẻ khách hàng thân thiết](#4-cookie--session)
5. [Configuration — Sổ tay bí kíp theo mùa](#5-configuration)
6. [Viết Custom Middleware — Tự đặt thêm trạm gác](#6-viết-custom-middleware)

---

# 1. Middleware Pipeline — Dây chuyền kiểm tra an ninh sân bay

## 1.1. Hình dung thế này

Bạn từ ngoài sân bay đi vào máy bay. Phải qua hàng loạt **trạm gác an ninh**:

```
Bạn → [Trạm soi hành lý] → [Trạm kiểm tra hộ chiếu] → [Trạm kiểm tra vé] → Lên máy bay ✈️
```

Mỗi trạm:

1. **Kiểm tra** — xem bạn có ổn không
2. **Cho qua** — gọi "Next!" đẩy bạn sang trạm sau
3. Hoặc **chặn lại** — "Anh kia! Quay lại! Hộ chiếu hết hạn!"

Trong ASP.NET Core, mỗi trạm gác = 1 **Middleware**. Toàn bộ dây chuyền = **Pipeline**.

```
Client → [Middleware 1] → [Middleware 2] → [Middleware 3] → Controller xử lý
                                                               ↓
Client ← [Middleware 1] ← [Middleware 2] ← [Middleware 3] ← Response quay lại
```

Response đi **ngược lại** qua từng trạm. Như khi bạn ra khỏi sân bay — phải qua lại trạm nhập cảnh.

## 1.2. THỨ TỰ TRẠM CỰC KỲ QUAN TRỌNG!

Đặt trạm kiểm tra vé **TRƯỚC** trạm kiểm tra hộ chiếu → ông nhân viên xem vé mà chưa biết bạn là ai → vô nghĩa!

```csharp
var app = builder.Build();

// Trạm 1: Chuông báo cháy — đứng ngoài cùng để bắt mọi sự cố
app.UseExceptionHandler("/Home/Error");

// Trạm 2: Ép đi cổng an ninh (HTTP → HTTPS)
app.UseHttpsRedirection();

// Trạm 3: Quầy duty-free (file tĩnh: CSS, JS, ảnh) — lấy đồ không cần vé
app.UseStaticFiles();

// Trạm 4: Bảng chỉ đường GPS — xem phòng chờ nào
app.UseRouting();

// Trạm 5: Kiểm tra hộ chiếu — "Bạn là AI?"
app.UseAuthentication();

// Trạm 6: Kiểm tra vé — "Bạn CÓ QUYỀN lên chuyến này không?"
app.UseAuthorization();

// Cổng lên máy bay — Controller xử lý
app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

> ⚠️ **Lỗi kinh điển**: Đặt `UseAuthorization` TRƯỚC `UseAuthentication` → trạm kiểm tra vé mà chưa biết bạn là ai → mọi request bị từ chối dù đã login!

## 1.3. Bảng tổng hợp các trạm gác có sẵn

| Lệnh gọi trạm | Nhân viên làm gì tại trạm |
|---|---|
| `UseExceptionHandler()` | Bắt mọi sự cố → trả trang lỗi sạch sẽ |
| `UseHsts()` | Dặn trình duyệt: "Lần sau tự đi cổng HTTPS, đừng ghé HTTP" |
| `UseHttpsRedirection()` | Chặn khách đi cổng HTTP → đá sang cổng HTTPS |
| `UseStaticFiles()` | Mở cửa hàng duty-free: CSS, JS, ảnh (không cần kiểm tra an ninh) |
| `UseRouting()` | Bảng chỉ đường: phân tích URL → tìm cổng boarding |
| `UseAuthentication()` | Kiểm tra hộ chiếu: "Bạn là AI?" |
| `UseAuthorization()` | Kiểm tra vé: "Bạn CÓ QUYỀN không?" |
| `UseSession()` | Mở ngăn tủ đồ riêng cho từng hành khách |
| `UseCors()` | Cho phép khách nước ngoài (domain khác) vào |
| `UseRateLimiter()` | Giới hạn số người vào cổng mỗi phút (chống quá tải) |

---

# 2. HttpRequest — Bức thư khách gửi lên

Mỗi khi bạn gõ URL hay bấm nút → trình duyệt viết 1 **bức thư** gửi lên server. Bức thư đó chính là `HttpRequest`.

## 2.1. Đọc thư ở đâu?
```csharp
Request              // Tắt đường (trong Controller)
HttpContext.Request  // Đầy đủ (dùng ở Middleware)
```

## 2.2. Thư có những gì?

| Phần trên thư | Ví dụ | Ý nghĩa |
|---|---|---|
| **Method** | `"GET"`, `"POST"` | Khách muốn XEM hay GỬI? |
| **Scheme** | `"https"` | Gửi qua đường an toàn hay không? |
| **Host** | `"localhost:7180"` | Gửi đến quán nào? |
| **Path** | `"/MonAn/ChiTiet"` | Gửi đến phòng nào? |
| **QueryString** | `"?id=5&color=red"` | Ghi chú thêm trên bì thư |
| **Headers** | 14 dòng metadata | Tem bưu điện, loại phong bì, ngôn ngữ |
| **Cookies** | Key=Value | Thẻ khách hàng cũ đính kèm |
| **Body** | JSON / Form data | Nội dung chính bên trong thư (POST) |

> 💡 **Mẹo debug**: Đặt breakpoint → mở **Watch Window** → gõ `Request` → xem toàn bộ nội dung bức thư.

---

# 3. HttpResponse — Bức thư quán gửi lại

Server đọc thư khách xong → viết 1 **bức thư trả lời** gửi về. Bức thư đó = `HttpResponse`.

## 3.1. Bốn phần trên thư trả lời

### a) Mã trạng thái — "Kết quả của bạn thế nào?"

| Mã | Ý nghĩa | Như ngoài đời |
|---|---|---|
| **200** | Thành công | "Đây, phở của bạn!" 🍜 |
| **301** | Đã chuyển địa chỉ | "Quán dọn sang số 5 rồi, qua đó nhé!" |
| **400** | Bạn gửi sai | "Order gì kỳ vậy? Gửi lại đi!" |
| **401** | Chưa đăng nhập | "Đưa thẻ khách hàng đã!" |
| **404** | Không tìm thấy | "Món này quán không có!" |
| **500** | Lỗi bếp | "Xin lỗi, bếp đang cháy!" 🔥 |

### b) Content-Type — "Anh gửi cái gì cho tôi?"
```
text/html           → Trang web đẹp
application/json    → Dữ liệu thô (cho mobile app)
image/png           → Hình ảnh
text/plain          → Text thường
```

### c) Headers — Tem dán trên phong bì trả lời
```csharp
Response.Headers.Append("X-Custom-Header", "xin-chao");
// Dán thêm 1 tờ tem tùy chỉnh. Dùng Append — nếu tem đã có thì thêm, không crash.
```

### d) Body — Nội dung bên trong bức thư
```csharp
// Cách 1: Qua IActionResult (ASP.NET tự ghi)
return Ok("Phở đây!");  // 200 + text

// Cách 2: Stream trực tiếp (gửi file lớn mà không cần load hết vào RAM)
Response.ContentType = "text/html";
using var fs = new FileInfo("file.html").OpenRead();
await fs.CopyToAsync(Response.Body);
// Như gửi thư bằng ống hơi — đẩy từng trang, không cần in hết rồi mới gửi
```

### View tìm ở đâu?
Khi Tiếp Tân nói `return View()`, ASP.NET tìm file Trang Trí theo thứ tự:

1. `/Views/{TênTiếpTân}/{TênNhiệmVụ}.cshtml` — kiếm phòng riêng trước
2. `/Views/Shared/{TênNhiệmVụ}.cshtml` — không có thì kiếm phòng chung

---

# 4. Cookie & Session — Thẻ khách hàng thân thiết

## 4.1. Vấn đề nhức đầu: HTTP = "Mất trí nhớ"

HTTP giống ông phục vụ **bị mất trí nhớ ngắn hạn**. Khách vừa gọi phở 2 phút trước, quay lại hỏi "Nước mắm đâu?" → ông ta: "Anh là ai? Tôi chưa thấy anh bao giờ!" 😱

Mỗi request = 1 cuộc nói chuyện hoàn toàn mới. Server **KHÔNG NHỚ** bạn là khách cũ.

## 4.2. Cookie — Thẻ khách hàng giấy

Giải pháp 1: Quán phát cho bạn 1 **tấm thẻ giấy** ghi "Tên: Nghĩa, VIP Gold". Mỗi lần vào quán, bạn đưa tấm thẻ → ông phục vụ nhìn thẻ biết bạn là ai.

```
Bạn vào quán → đưa TẤT CẢ thẻ cho quán
Quán xử lý → phát thẻ mới nếu cần
Bạn nhận → cất vào ví cho lần sau
```

**Nhược điểm thẻ giấy:**

1. 🏋️ **Nặng túi**: Mỗi lần vào phải đưa TẤT CẢ thẻ, kể cả thẻ không liên quan
2. 🔓 **Dễ giả mạo**: Bạn có thể tự sửa thẻ giấy (đổi "Normal" thành "VIP Gold")

## 4.3. Session — Tủ đồ có khóa ở quán

Giải pháp 2: Quán **giữ hộ đồ** trong tủ khóa phía sau quầy. Bạn chỉ nhận 1 chiếc **chìa khóa số** (Session ID). Mỗi lần vào → đưa chìa khóa → quán mở tủ lấy đồ của bạn ra.

| | Cookie (Thẻ giấy) | Session (Tủ khóa) |
|---|---|---|
| **Đồ để ở đâu** | Trong ví bạn (trình duyệt) | Trong tủ quán (server) |
| **Ai đọc được** | Bạn + quán + kẻ gian | Chỉ quán |
| **Bảo mật** | Thấp — sửa được | Cao — chỉ có chìa khóa |
| **Bạn mang theo cái gì** | Toàn bộ data | Chỉ mang chìa khóa nhỏ |

## 4.4. Dùng Session trong ASP.NET Core

```csharp
// Program.cs — Lắp tủ khóa
builder.Services.AddDistributedMemoryCache(); // Mua tủ loại "trong bộ nhớ"
builder.Services.AddSession();                // Lắp tủ vào quán
app.UseSession();                             // Mở tủ sẵn sàng phục vụ

// Controller — Sử dụng tủ
HttpContext.Session.SetString("ten", "Nghĩa");       // Bỏ đồ vào tủ
var ten = HttpContext.Session.GetString("ten");        // Lấy đồ ra
```

## 4.5. Bẫy trong Production: Nhiều quán chi nhánh

Quán lớn mở **3 chi nhánh** (3 server). Khách lần 1 vào chi nhánh A → gửi đồ tủ A. Lần 2 vào chi nhánh B → Tủ B rỗng! **Mất đồ!**

**Giải pháp**: Dùng kho chung **Redis** (hoặc SQL Server) — 3 chi nhánh đều mở được tủ chung.

```csharp
// Thay tủ riêng bằng tủ chung Redis
builder.Services.AddStackExchangeRedisCache(options =>
    options.Configuration = "redis-server:6379");
```

> ⚠️ **Session KHÔNG dùng cho Web API!** API phục vụ mobile, IoT — những thứ không có "quán chi nhánh". API phải **stateless** (mất trí nhớ có chủ đích) → dùng JWT Token thay thế (Module 06).

---

# 5. Configuration — Sổ tay bí kíp theo mùa

## 5.1. Tại sao cần Sổ tay?

Quán có nhiều thứ thay đổi theo mùa: WiFi password, địa chỉ kho, mật khẩu két sắt. **Không thể viết cứng vào code** — lỡ đổi WiFi → phải sửa code, build lại, deploy lại? Quá mệt!

**Configuration** = sổ tay riêng, quán viết 1 lần, đổi khi cần mà **KHÔNG sửa code**.

## 5.2. Nhiều cuốn sổ, cuốn nào thắng?

```
[MỌI NGƯỜI NGHE]      Command Line Arguments     ← Ông chủ hét to nhất
                       Environment Variables       ← Bảng hiệu treo tường
                       User Secrets (chỉ khi sửa quán)
                       appsettings.Development.json ← Sổ tay mùa sửa chữa
                       appsettings.json            ← Sổ tay chung quanh năm
[AI HÉT TO NHẤT THẮNG]
```

Cùng 1 key, ông chủ hét to nhất (Command Line) **thắng** cuốn sổ ghi dưới.

## 5.3. Ba cách đọc Sổ tay

### Cách 1: Đọc trực tiếp — "WiFi password là gì?"
```csharp
var wifi = _configuration["WiFiPassword"]; // Trả về string
```

### Cách 2: Gom thành Object — "Cho tôi cả trang cấu hình API"
```json
// appsettings.json
{
    "ApiSettings": {
        "BaseUrl": "https://api.example.com",
        "ApiKey": "abc123"
    }
}
```
```csharp
var settings = _configuration.GetSection("ApiSettings").Get<ApiSettings>();
```

### Cách 3: Options Pattern ✅ (KHUYÊN DÙNG — kiểu VIP)
```csharp
// Đăng ký 1 lần
builder.Services.Configure<ApiSettings>(
    builder.Configuration.GetSection("ApiSettings"));

// Dùng ở bất kỳ đâu — ASP.NET tự đem đến tận tay (DI)
public class HomeController(IOptions<ApiSettings> options) : Controller
{
    private readonly ApiSettings _api = options.Value;
}
```

## 5.4. Sổ tay tự cập nhật khi sửa

| Loại sổ | Cập nhật khi chỉnh sửa? | Ẩn dụ |
|---|---|---|
| `IOptions<T>` | ❌ Không — đọc 1 lần lúc mở quán | Sổ tay photo — đã in xong rồi |
| `IOptionsSnapshot<T>` | ✅ Có — mỗi khách vào đọc lại | Sổ tay kẹp bìa — thay trang mới mỗi lượt |
| `IOptionsMonitor<T>` | ✅ Có — realtime, rung chuông khi đổi | Sổ điện tử — tự nhấp nháy khi có thay đổi |

---

# 6. Viết Custom Middleware — Tự đặt thêm trạm gác

## 6.1. Khi nào tự đặt trạm?
Khi bạn muốn **kiểm tra something ở MỌI request** (ví dụ: kiểm tra API Key), lặp lại logic đó trong mỗi Controller → copy-paste 100 lần. Viết 1 Middleware → **1 lần, áp dụng cho tất cả**.

## 6.2. Quy tắc xây trạm gác

Mỗi trạm phải có:

- **Cổng vào**: Constructor nhận `RequestDelegate next` (biết trạm kế tiếp)
- **Nhân viên trực**: Hàm `InvokeAsync` (chạy mỗi khách đi qua)

```csharp
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;  // Biết trạm kế tiếp
    private readonly ILogger<ApiKeyMiddleware> _logger;

    // CỬA HÀNG MỞ CỬA — chạy 1 lần duy nhất (Singleton)
    public ApiKeyMiddleware(RequestDelegate next, ILogger<ApiKeyMiddleware> logger)
    {
        _next = next;
        _logger = logger;
        _logger.LogInformation("Trạm kiểm tra API Key đã mở!");
    }

    // NHÂN VIÊN TRỰC — chạy MỖI khách đi qua
    public async Task InvokeAsync(HttpContext context, IClientRepository repo)
    {
        // Khách có đưa thẻ VIP (API Key) không?
        var key = context.Request.Headers["api-key"].FirstOrDefault();

        if (!string.IsNullOrEmpty(key))
        {
            var khach = repo.TimKhach(key);
            if (khach != null)
            {
                context.Features.Set(khach); // Dán nhãn "Khách VIP" lên vali
                await _next(context);        // "NEXT!" — cho qua trạm kế tiếp
                return;
            }
        }

        // Không có thẻ hoặc thẻ giả → CHẶN LẠI, không gọi next()
        _logger.LogWarning("Khách lạ không có API Key!");
        context.Response.StatusCode = 401; // "Đuổi về!"
    }
}
```

## 6.3. Bẫy chết người: Constructor vs InvokeAsync

| Khai báo ở đâu | Sống bao lâu | Ẩn dụ |
|---|---|---|
| Constructor | **Mãi mãi** (Singleton) | Ông quản lý — 1 ông duy nhất |
| InvokeAsync parameter | **Mỗi khách** (Per-request) | Nhân viên temp — thuê mới mỗi lượt |

> ⚠️ **KHÔNG inject DbContext vào Constructor** — DbContext là Scoped (nhân viên temp). Ông quản lý (Singleton) giữ nhân viên temp → nhân viên đã nghỉ mà vẫn bị gọi → SẬP!

## 6.4. Features Collection — "Dán nhãn trên vali"

Trạm gác A dán nhãn "Khách VIP" lên vali. Trạm B, trạm C, Controller đều đọc được nhãn đó:

```csharp
// Middleware dán nhãn:
context.Features.Set(clientInfo);

// Controller đọc nhãn:
var khach = HttpContext.Features.Get<ClientInfo>();
// Không cần kiểm tra API Key lại nữa — đã dán nhãn rồi!
```

## 6.5. Đóng gói trạm gác — Viết chuẩn như Microsoft

```csharp
public static class ApiKeyExtensions
{
    // Mua dụng cụ cho trạm
    public static IServiceCollection AddApiKeyAuth(this IServiceCollection services)
    {
        services.AddSingleton<IClientRepository, ClientRepository>();
        return services;
    }

    // Lắp trạm vào dây chuyền
    public static IApplicationBuilder UseApiKeyAuth(this IApplicationBuilder app)
    {
        return app.UseMiddleware<ApiKeyMiddleware>();
    }
}

// Program.cs — gọn gàng như của Microsoft:
builder.Services.AddApiKeyAuth();  // Mua dụng cụ
app.UseApiKeyAuth();               // Lắp trạm (ĐẶT TRƯỚC UseStaticFiles nếu muốn chặn cả file tĩnh)
```

---

> 🎯 **Module tiếp theo**: [Module 03 — Routing & Model Binding](Module-03-Routing-Model-Binding.md)
