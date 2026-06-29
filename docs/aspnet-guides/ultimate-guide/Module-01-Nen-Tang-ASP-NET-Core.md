# 📘 Module 01: Nền Tảng ASP.NET Core
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [Các khái niệm nền tảng](#1-các-khái-niệm-nền-tảng)
2. [Lịch sử ASP.NET — từ "kéo thả" đến "tên lửa"](#2-lịch-sử-aspnet)
3. [Tổng quan ASP.NET Core](#3-tổng-quan-aspnet-core)
4. [Thiết lập môi trường](#4-thiết-lập-môi-trường)
5. [Tạo dự án đầu tiên & Cấu trúc Project](#5-tạo-dự-án-đầu-tiên)
6. [MVC — Nhà hàng 3 nhân viên](#6-mvc--nhà-hàng-3-nhân-viên)
7. [Controller & Action — Nhân viên tiếp tân](#7-controller--action)
8. [Dependency Injection — Anh shipper nội bộ](#8-dependency-injection)
9. [Routing — Bản đồ GPS](#9-routing--bản-đồ-gps)
10. [HttpContext — Chiếc vali hành lý](#10-httpcontext)
11. [Client-Side: HTML, CSS, JavaScript](#11-client-side)
12. [AJAX & Async/Await — Gọi món mà không đứng chờ](#12-ajax--asyncawait)
13. [Form Handling & Validation — Phiếu đăng ký](#13-form-handling--validation)
14. [Bài tập thực hành](#14-bài-tập)

---

# 1. Các khái niệm nền tảng

## 1.1. Web App = Quán ăn Online
Bạn mở Chrome, gõ `facebook.com`. Thế là bạn đang **vào quán ăn online** — không cần tải app về máy, không cần cài đặt gì. Chỉ cần có WiFi và trình duyệt là bạn đã "ngồi vào bàn" rồi.

## 1.2. HTTP = Ngôn ngữ giao tiếp liên hành tinh
Trình duyệt (Chrome) = **Bạn** ở hành tinh Trái Đất.
Máy chủ (Server) = **Đầu bếp** ở hành tinh Mars.

Hai bên nói chuyện bằng ngôn ngữ chung tên là **HTTP**:

- **Request**: Bạn viết một tờ giấy ghi "Cho tôi trang chủ Google" rồi phóng ra vũ trụ.
- **Response**: Đầu bếp nhận tờ giấy, nấu xong, đóng gói giao diện HTML rồi bắn ngược lại Trái Đất.

Mỗi lần bạn gõ URL, nhấn nút, bấm link → một tờ giấy mới bay đi và bay về. HTTP đơn giản vậy thôi.

## 1.3. Framework = Gói "Xây nhà trọn bộ"
Viết web từ đầu = tự nung gạch, trộn xi măng, kéo dây điện, đấu ống nước. Mất cả đời chưa xong cái nhà.

**Framework** = mua gói "Xây nhà thô" — móng đã đổ, cột đã dựng, điện nước đã kéo xong. Bạn chỉ cần bước vào **sơn tường, kê bàn ghế** (viết phần logic riêng của mình). ASP.NET Core chính là một gói nhà thô như vậy — siêu chắc, siêu nhanh, do Microsoft xây.

---

# 2. Lịch sử ASP.NET

Hãy tưởng tượng Microsoft mở một chuỗi nhà hàng, trải qua 4 đời:

| Đời | Tên | Ẩn dụ | Vấn đề |
|---|---|---|---|
| 🧓 1 | **ASP** (Active Server Pages) | Quán vỉa hè — nấu tạm bằng script | Code rối, khó bảo trì |
| 👴 2 | **ASP.NET WebForms** | Nhà hàng kéo-thả — khách kéo bàn ghế lắp UI | Nặng nề, dữ liệu truyền qua lại quá nhiều |
| 👨 3 | **ASP.NET MVC** | Nhà hàng 3 khu vực rõ ràng (Bếp, Tiếp tân, Mặt tiền) | Chỉ chạy trên Windows |
| 🧑‍🚀 4 | **ASP.NET Core** ✅ | Nhà hàng quốc tế — chạy ở Mỹ, Nhật, Châu Âu đều OK | Đây là cái bạn đang học! |

> ⚠️ WebForms đã bị Microsoft "đóng cửa". Đừng học nó. Chỉ tập trung vào **ASP.NET Core**.

---

# 3. Tổng quan ASP.NET Core

## 3.1. Nó là gì?
Web Framework do Microsoft xây, viết bằng ngôn ngữ **C#**. Giúp bạn tạo website và API nhanh chóng, an toàn.

## 3.2. Giải mã "tiếng anh hack não"

| Thuật ngữ hack não | Dịch sang tiếng người | Ví dụ cực kỳ dễ hiểu |
|---|---|---|
| **Open-source** | Mã nguồn mở | Như công thức nấu ăn công khai — ai cũng xem, đóng góp, dùng FREE |
| **Cross-platform** | Chạy đa hệ điều hành | Nhà hàng mở chi nhánh ở Windows, Mac, Linux — không bị giam ở 1 phố nữa |
| **High-performance** | Siêu nhanh | Đầu bếp cũ nấu 100 món/giờ. Đầu bếp mới nấu 10,000 món/giờ. Nhanh vì bỏ thao tác thừa |
| **Modular** | Lắp ráp theo ý | Như xếp LEGO — chỉ lấy viên gạch cần dùng, app cực nhẹ |
| **Side-by-side** | Chạy song song | App A dùng .NET 6, App B dùng .NET 8, cùng 1 máy chủ — KHÔNG xung đột. Như 2 cái TV trong nhà, mỗi cái xem kênh khác |

## 3.3. Tính năng đáng yêu

| Tính năng | Ẩn dụ |
|---|---|
| **Dependency Injection** | Anh shipper nội bộ — bạn cần dao, anh tự đem tới, không cần bạn chạy ra kho lấy |
| **Kestrel Web Server** | Máy phát điện mini gắn trong xe — chạy ở đâu cũng có điện (Windows, Mac, Linux) |
| **Unified Web UI & API** | 1 căn bếp nấu được cả phở (trang web HTML) lẫn cơm hộp (dữ liệu JSON cho mobile) |

---

# 4. Thiết lập môi trường

## 4.1. .NET SDK — Bộ đồ nghề đầu bếp
Download: https://dotnet.microsoft.com/download

Chọn bản **LTS** (.NET 8.0) — Giống mua xe Toyota: Microsoft cam kết sửa chữa miễn phí **3 năm**. Bản thường chỉ được sửa 10 tháng → lỡ phát hiện lỗi bảo mật nghiêm trọng thì Microsoft mặc kệ bạn. Chọn LTS cho yên tâm.

## 4.2. Visual Studio 2022 — Nhà bếp 5 sao
Môi trường phát triển với gợi ý code, tô màu, debug trực quan:

- Tải bản **Community** (miễn phí, đủ xài)
- Chọn Workloads: ✅ **ASP.NET and web development**

## 4.3. .NET CLI — Nấu ăn bằng dòng lệnh
Đôi khi bạn muốn nấu nhanh mà không cần mở cả nhà bếp 5 sao:
```bash
dotnet --version          # Kiểm tra đồ nghề OK chưa
dotnet new mvc -n TiemCom  # Tạo nhà hàng mới tên "TiemCom"
dotnet run                 # Mở cửa đón khách!
```

---

# 5. Tạo dự án đầu tiên

## 5.1. Bước tạo project
1. Mở Visual Studio → **Create a new project**
2. Chọn **ASP.NET Core Web App (Model-View-Controller)**
3. Đặt tên, chọn **.NET 8.0 (LTS)** → Create

## 5.2. Sơ đồ căn nhà vừa xây xong

```
MyProject/
├── Program.cs               ← Cửa chính (bật công tắc điện, mở cổng đón khách)
├── appsettings.json          ← Sổ tay ghi cấu hình (WiFi password, địa chỉ kho...)
├── wwwroot/                  ← Phòng trưng bày (hình ảnh, CSS, JS — khách nhìn thấy)
│   ├── css/
│   ├── js/
│   └── lib/
├── Controllers/              ← Phòng Tiếp Tân (nhận yêu cầu, điều phối)
├── Models/                   ← Phòng Bếp (nguyên liệu + công thức nấu ăn)
└── Views/                    ← Phòng Trưng Bày Đẹp (.cshtml — bày ra cho khách xem)
```

## 5.3. Program.cs — "Chỉ là nút bấm ON"

Một sự thật bất ngờ: **Web App thực chất chỉ là app Console bình thường**, giống như cái chương trình in `Hello World` bạn từng viết. Điểm khác duy nhất là nó **mở một cổng cửa ra cho khách vào**.

```csharp
var builder = WebApplication.CreateBuilder(args); // Chuẩn bị vật liệu

builder.Services.AddControllersWithViews(); // Thuê 3 nhân viên MVC

var app = builder.Build(); // Xây xong nhà hàng

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error"); // Lắp chuông báo cháy
    app.UseHsts();                          // Lắp khóa cửa tự động
}

app.UseHttpsRedirection(); // Ép khách đi cổng an ninh (HTTPS)
app.UseStaticFiles();      // Mở phòng trưng bày (CSS, JS, ảnh)
app.UseRouting();          // Treo bảng chỉ đường
app.UseAuthorization();    // Đặt bảo vệ kiểm tra vé

// Bảng chỉ đường mặc định: /NhânViên/HànhĐộng/SốThứTự
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run(); // MỞ CỬA ĐÓN KHÁCH!
```

## 5.4. appsettings.json — Sổ tay cấu hình

Tưởng tượng quán có 2 cuốn sổ tay:
```
appsettings.json                 ← Sổ tay chung (luôn dùng)
appsettings.Development.json     ← Sổ tay riêng cho lúc đang sửa chữa quán
```
Nếu cả 2 sổ ghi cùng 1 thứ, sổ riêng **thắng**. Ví dụ: sổ chung ghi "WiFi: quancomlala", sổ Development ghi "WiFi: devtest123" → lúc đang sửa quán thì dùng WiFi devtest123.

## 5.5. HSTS vs HTTPS Redirection — Hai loại khóa cổng

| | HTTPS Redirect | HSTS |
|---|---|---|
| **Cơ chế** | Khách gõ cổng HTTP → Bảo vệ hét: "Đi sang cổng HTTPS kia!" | Bảo vệ dặn khách: "Nhớ lần sau tự đi cổng HTTPS, đừng vào đây nữa" |
| **Lần đầu** | Phải hét mỗi lần | Hét 1 lần, khách tự nhớ |

---

# 6. MVC — Nhà hàng 3 nhân viên

Tưởng tượng quán ăn có đúng 3 nhân viên, mỗi người làm 1 việc duy nhất:

## 6.1. Ba nhân viên

| Nhân viên | Tên code | Việc duy nhất | Quy tắc vàng |
|---|---|---|---|
| 🧑‍🍳 **Đầu Bếp** | **Model** | Nấu ăn + giữ công thức (dữ liệu + business logic) | Không bao giờ ra tiếp khách |
| 🤵 **Tiếp Tân** | **Controller** | Nhận order, giao cho Bếp, đem ra Mặt Tiền | Không bao giờ tự nấu, không bao giờ tự trang trí |
| 💅 **Trang Trí** | **View** | Bày biện đĩa thức ăn đẹp lung linh (.cshtml) | Không bao giờ tự nấu, không quan tâm nguyên liệu từ đâu |

## 6.2. Luồng phục vụ 1 khách

```
Khách gõ: nhahang.com/MonAn/ChiTiet/5

1. Bảng chỉ đường (Routing) đọc: "Ồ, gọi Tiếp Tân tên MonAn, yêu cầu ChiTiet, món số 5"
2. Tiếp Tân (Controller) chạy vào bếp: "Bếp ơi cho tôi món số 5!"
3. Đầu Bếp (Model) mở tủ lạnh (Database) → tìm Phở, giá 50,000đ
4. Tiếp Tân cầm đĩa Phở → đưa cho Trang Trí (View)
5. Trang Trí bày đĩa lên khay sứ, cắm hoa, thêm nến → giao cho khách
6. Khách thấy: Trang HTML đẹp lung linh hiển thị "Phở — 50,000đ"
```

> 💡 **"Fat Model, Skinny Controller"**: Logic kinh doanh (tính giá giảm 5%, kiểm tra tồn kho) thuộc về Bếp (Model). Tiếp Tân (Controller) chỉ nhận-giao-nhận-giao. Nếu Tiếp Tân tự nấu = code dính thành 1 cục → không thể test, không thể tái sử dụng.

## 6.3. Code ví dụ — Xem thử 3 nhân viên làm việc

```csharp
// === ĐẦU BẾP (Model): Models/Product.cs ===
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// === TIẾP TÂN (Controller): Controllers/ProductController.cs ===
public class ProductController : Controller
{
    public IActionResult Detail(int id)
    {
        // Tiếp tân vào bếp lấy món
        var mon = new Product { Id = id, Name = "Laptop Dell", Price = 999 };
        return View(mon);  // Đưa món cho Trang Trí bày
    }
}
```

```html
<!-- === TRANG TRÍ (View): Views/Product/Detail.cshtml === -->
@model Product

<h2>🍽️ Chi tiết sản phẩm</h2>
<ul>
    <li>Tên: <b>@Model.Name</b></li>
    <li>Giá: $@Model.Price</li>
</ul>
```

## 6.4. Layout & Partial View — Bộ khung và mảnh ghép

**`_Layout.cshtml`** = Bộ khung nhà hàng (Header logo, Footer bản quyền). Ở giữa để trống 1 khoảng `@RenderBody()` — ASP.NET sẽ tự nhét nội dung từng trang vào đó. Như cái khung ảnh: khung giữ nguyên, ảnh đổi tùy trang.

**`_Partial.cshtml`** = Mảnh ghép tái sử dụng. Ví dụ: thẻ hiển thị 1 sản phẩm — dùng 50 lần ở 50 trang mà chỉ viết 1 lần.

---

# 7. Controller & Action

## 7.1. Controller = Tiếp Tân, Action = Từng nhiệm vụ
Mỗi hàm `public` trong Controller = 1 **Action Method** = 1 nhiệm vụ cụ thể mà Tiếp Tân biết làm.

## 7.2. Tiếp Tân sinh ra rồi mất đi MỖI lượt khách

Mỗi khi có khách gõ URL → ASP.NET **thuê thêm 1 Tiếp Tân mới**, phục vụ xong → **cho nghỉ việc**. Khách khác vào → lại thuê người mới.

```csharp
public class HomeController : Controller
{
    public HomeController()
    {
        Console.WriteLine("Tiếp tân MỚI vừa được thuê!"); // In mỗi request
    }
}
```

Tại sao? Vì nếu 1 Tiếp Tân phục vụ 1000 khách cùng lúc → rối loạn order. Mỗi người phục vụ 1 khách = rõ ràng, an toàn.

## 7.3. Kiểu trả về — Tiếp Tân trả gì cho khách?

| Trả về | Khách nhận | Ví dụ |
|---|---|---|
| `View()` | Trang HTML đẹp | Trang sản phẩm |
| `Json(data)` | Dữ liệu JSON thô | API cho mobile app |
| `Redirect("/url")` | Chuyển hướng sang trang khác | Sau login → về trang chủ |
| `Content("text")` | Text thuần | Debug, test nhanh |
| `NotFound()` | Trang 404 | Món không có trong menu |

## 7.4. Attributes — "Biển cấm" gắn trên cửa Tiếp Tân

```csharp
[NonAction]      // "Hàm này KHÔNG phải nhiệm vụ" — khách không gọi được
[NonController]  // "Class này KHÔNG phải Tiếp Tân" — hệ thống bỏ qua

[HttpGet]        // "Chỉ phục vụ khi khách GỌI XEM"
[HttpPost]       // "Chỉ phục vụ khi khách GỬI ĐƠN"
[HttpDelete]     // "Chỉ phục vụ khi khách YÊU CẦU XÓA"
```

## 7.5. Nhóm `From*` — Chỉ định "Lấy thông tin từ đâu?"

Khách gửi thông tin từ nhiều chỗ. Bạn có thể chỉ Tiếp Tân lấy đúng chỗ:

```csharp
public IActionResult XuLy(
    [FromQuery] string search,            // Lấy từ ?search=... trên URL
    [FromHeader(Name = "X-Key")] string key,  // Lấy từ mũ bảo hiểm (header)
    [FromBody] OrderDto body,             // Lấy từ thư trong bao (body JSON)
    [FromForm] string field,              // Lấy từ phiếu đăng ký (form)
    [FromServices] IOrderRepo repo        // Lấy dụng cụ từ kho nội bộ (DI)
)
```

---

# 8. Dependency Injection

## 8.1. DI = Anh Shipper nội bộ

**Không có DI**: Mỗi sáng Tiếp Tân phải TỰ chạy ra chợ mua dao, mua thớt, mua nồi → mệt, chậm, mỗi người mua mỗi loại khác nhau.

**Có DI**: Quán thuê **anh Shipper nội bộ**. Tiếp Tân chỉ cần nói "Tôi cần con dao" → Anh shipper TỰ ĐEM ĐẾN. Tiếp Tân không cần biết dao mua ở đâu, hãng gì.

```csharp
// Tiếp Tân chỉ nói: "Tôi cần kho hàng"
public class HomeController : Controller
{
    private readonly ITodoRepository _kho;

    public HomeController(ITodoRepository kho) // Shipper tự đem đến
    {
        _kho = kho;
    }
}
```

## 8.2. Đăng ký với Shipper — Bảng kê khai kho

```csharp
// Program.cs — Dặn Shipper: "Khi ai xin ITodoRepository thì đưa InMemoryRepo"
builder.Services.AddSingleton<ITodoRepository>(
    sp => new InMemoryTodoRepository());

builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();
```

## 8.3. Ba loại hợp đồng Shipper

| Hợp đồng | Thuê/ngưng | Ví dụ dễ hiểu | Dùng cho |
|---|---|---|---|
| **Transient** | Mỗi lần xin → anh shipper mới | Ly giấy 1 lần — uống xong vứt | Email sender, Validator |
| **Scoped** | Mỗi khách vào → 1 shipper riêng, phục vụ xong mới nghỉ | Bồi bàn riêng phục vụ 1 bàn | **DbContext**, UnitOfWork (phổ biến nhất!) |
| **Singleton** | Thuê 1 người duy nhất, phục vụ CẢ ĐỜI quán | Ông bảo vệ đứng cổng — chỉ có 1 ông | Configuration, Cache |

> ⚠️ **Bẫy chết người**: Nếu kho hàng nằm trong RAM (`InMemoryRepository`) mà đăng ký Transient → mỗi khách tạo kho mới → data biến mất! Phải dùng **Singleton**.

---

# 9. Routing — Bản đồ GPS

## 9.1. Routing = Hệ thống GPS
Khách gõ URL → GPS (Routing) phân tích → tìm đúng nhà (Controller + Action). Không tìm được → **404 Not Found** (nhà không tồn tại).

## 9.2. Hai cách gắn GPS

### Cách 1: GPS Trung Tâm (Conventional Routing)
Gắn bản đồ ở cổng chính `Program.cs`, áp dụng cho toàn quán:
```csharp
// Pattern: /NhânViên/Hành/SốThứTự
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Bảng GPS phân tích URL:**

| Khách gõ | GPS hiểu | Nhân viên | Nhiệm vụ |
|---|---|---|---|
| `/` | Home / Index | Home | Index |
| `/Products` | Products / Index | Products | Index |
| `/Products/Edit/99` | Products / Edit / 99 | Products | Edit (món 99) |

### Cách 2: GPS Cá Nhân (Attribute Routing)
Mỗi Tiếp Tân tự dán biển chỉ đường riêng:
```csharp
[Route("danh-sach-nv-2026")]     // Gõ /danh-sach-nv-2026 → gọi hàm này
public IActionResult List() => View();

[Route("p/{id:int}")]            // Gõ /p/123 → id phải là số
public IActionResult Details(int id) => View();
```

## 9.3. Route Constraints — Barie chặn đường

Giống gác chặn ở đường hầm: "Xe cao trên 2m không được vào!"

| Constraint | Ý nghĩa | Ví dụ |
|---|---|---|
| `{id:int}` | Chỉ cho số nguyên qua | `/p/123` ✅, `/p/abc` ❌ |
| `{name:alpha}` | Chỉ cho chữ cái qua | `/p/laptop` ✅, `/p/123` ❌ |
| `{id:min(1)}` | Giá trị tối thiểu 1 | `/p/0` ❌, `/p/1` ✅ |
| `{name:maxlength(50)}` | Dài tối đa 50 ký tự | Chuỗi quá dài bị chặn |
| `{slug:regex(^[a-z-]+$)}` | Chỉ chữ thường + dấu gạch | `/p/my-product` ✅ |

---

# 10. HttpContext — Chiếc vali hành lý

## 10.1. HttpContext chứa gì?

Mỗi khách đi du lịch đều mang theo 1 chiếc vali. Trong vali có mọi thứ: hộ chiếu, vé máy bay, đồ dùng. Trong ASP.NET, **HttpContext** chính là chiếc vali đó — chứa mọi thông tin về request đang được xử lý.

| Ngăn trong vali | Kiểu | Chứa gì | Ai bỏ vào? |
|---|---|---|---|
| **Request** | `HttpRequest` | Thư khách gửi lên (URL, header, body) | Khách (trình duyệt) |
| **Response** | `HttpResponse` | Thư trả về cho khách | Server tự viết |
| **Connection** | `ConnectionInfo` | IP, Port kết nối | Hệ thống tự điền |
| **User** | `ClaimsPrincipal` | Thẻ VIP (sau khi đăng nhập) | Middleware Auth |
| **Session** | `ISession` | Giỏ đồ tạm thời (giỏ hàng) | Developer tự dùng |
| **Items** | `IDictionary` | Giấy note chia sẻ giữa các trạm gác | Middleware tự ghi |

## 10.2. Extension Method — Gắn thêm ngăn phụ vào vali

```csharp
// Tạo ngăn phụ "GetFullUrl" cho cái vali Request
public static class RequestExtension
{
    public static string GetFullUrl(this HttpRequest request)
        => $"{request.Scheme}://{request.Host}{request.Path}";
}

// Sử dụng: như thể ngăn này có sẵn từ đầu
var url = Request.GetFullUrl();
```

---

# 11. Client-Side

## 11.1. Bộ ba quyền lực — 3 anh em nhà Web

| Anh em | Vai trò | Ví dụ cực dễ |
|---|---|---|
| **HTML** | Xương thịt — cấu trúc | `<h1>Xin chào</h1>`, `<button>Bấm</button>` |
| **CSS** | Da và áo — ngoại hình | `h1 { color: red; font-size: 50px; }` |
| **JavaScript** | Thần kinh — hành vi | Bấm nút → đổi màu, hiện popup |

> 💡 Trong ASP.NET: File CSS/JS nằm trong `wwwroot/`. Dấu `~` = thư mục wwwroot. Ví dụ: `<link href="~/css/site.css" />`

## 11.2. Razor (.cshtml) — Nhúng C# vào HTML

File `.cshtml` = HTML có siêu năng lực — dùng `@` để nhúng code C#:

```html
@model Product
<h1>@Model.Name</h1>                 <!-- In tên sản phẩm -->

@if (Model.Price > 1000000)           <!-- Nếu giá > 1 triệu -->
{
    <span class="dat">Đắt quá xá! 💰</span>
}

@foreach (var item in Model.Items)    <!-- Lặp danh sách -->
{
    <li>@item.Name — @item.Price đ</li>
}
```

---

# 12. AJAX & Async/Await

## 12.1. Ví dụ cuộc sống: Gọi phở vs Gọi GrabFood

| | Ăn tại quán (Synchronous) | Gọi GrabFood (Asynchronous) |
|---|---|---|
| **Client** | Đi vào quán → ngồi chờ → nhận đĩa → ăn | Bấm app Grab → NẰM XEM TV → Chuông kêu → nhận đồ |
| **Trên web** | Bấm "Like" → MÀN HÌNH TRẮNG → reload cả trang | Bấm "Like" → nút tự đổi màu, trang KHÔNG reload |

## 12.2. AJAX = Gọi GrabFood ngầm
JavaScript gửi request ngầm → Server xử lý → Trả dữ liệu về → Cập nhật 1 phần trang. **Không reload cả bữa ăn** chỉ vì thiếu 1 chén nước mắm.

## 12.3. async/await — Bồi bàn thông minh

**Vấn đề**: Tiếp Tân hỏi Đầu Bếp nấu phở → Đầu Bếp nấu 3 phút → Tiếp Tân **ĐỨNG ĐỰC CHỜ** → 100 khách khác la hét: "Sao không ai phục vụ???". Tiếp Tân tắc đường → Quán **SẬP TIỆM**.

**Giải pháp**: Tiếp Tân dặn Đầu Bếp "Nấu xong gọi tôi nhé!" rồi **quay đi phục vụ khách khác**. Phở xong → Đầu Bếp hú → Tiếp Tân quay lại bưng.

```csharp
// Bồi bàn THÔNG MINH — không đứng chờ
public async Task<IActionResult> Index()
{
    var data = await db.Products.ToListAsync(); // "Kho ơi, xong thì gọi tôi"
    return View(data); // Kho xong rồi → bưng ra
}
```

---

# 13. Form Handling & Validation

## 13.1. Model Binding = Phiên dịch phiếu đăng ký

Khách điền form → nhấn Gửi → ASP.NET **tự động dịch** các ô input thành object C#. Bạn không cần viết 1 dòng code đọc form nào cả!

```csharp
// Phiếu đăng ký
public class DangKyModel
{
    [Required(ErrorMessage = "Tên bắt buộc phải điền!")]
    [StringLength(50, ErrorMessage = "Tên dài tối đa 50 ký tự!")]
    public string Name { get; set; }

    [RegularExpression(@"^[0-9]{10}$", ErrorMessage = "Phải nhập đúng 10 chữ số")]
    public string Phone { get; set; }
}

// Tiếp Tân nhận phiếu
[HttpPost]
public IActionResult DangKy(DangKyModel phieu) // ASP.NET tự dịch form → object
{
    if (!ModelState.IsValid) // Kiểm tra phiếu có hợp lệ không?
    {
        return View(phieu); // Phiếu sai → trả lại, khoanh đỏ chỗ sai
    }
    LuuVaoDatabase(phieu);   // Phiếu đúng → lưu
    return Content("Đăng ký thành công! 🎉");
}
```

## 13.2. Bảng "hàng rào" kiểm tra phổ biến

| Hàng rào | Chức năng | Ví dụ |
|---|---|---|
| `[Required]` | Bắt buộc điền | "Ô này không được để trống!" |
| `[StringLength(50)]` | Giới hạn độ dài | "Tên dài quá 50 ký tự rồi!" |
| `[Range(1, 100)]` | Giới hạn số | "Tuổi phải từ 1 đến 100" |
| `[EmailAddress]` | Phải đúng email | "Định dạng email sai kìa" |
| `[Compare("Password")]` | So khớp 2 ô | "Mật khẩu xác nhận không khớp!" |
| `[ValidateNever]` | Bỏ qua kiểm tra | "Ô này server tự điền, đừng kiểm tra" |

## 13.3. CSRF — Tên trộm giả mạo đơn hàng

**Câu chuyện**: Bạn đang login Facebook. Lúc đó bạn lỡ bấm vào 1 link lạ. Trang lạ đặt sẵn 1 form ngầm "Chuyển tiền 10 triệu" rồi **tự động gửi** đến Facebook. Vì trình duyệt đính kèm cookie đăng nhập → Facebook tưởng **bạn** thao tác!

**Giải pháp**: ASP.NET giấu 1 mã bí mật (Antiforgery Token) vào form. Submit phải kèm mã → Trang lạ không có mã → **BỊ CHẶN**.

```csharp
[HttpPost]
[ValidateAntiForgeryToken]  // Bật "khóa chống giả mạo"
public IActionResult ChuyenTien(TransferModel model) { ... }
```

---

# 14. Bài tập

## 14.1. Bài 1: Hello World — Hỏi thăm máy chủ

Tạo app MVC hiển thị CPU và hệ điều hành của server — chứng minh vòng đời MVC hoạt động:

```csharp
// Đầu Bếp (Model)
public class OSInfo { public int CPU { get; set; } public string OS { get; set; } }

// Tiếp Tân (Controller)
public IActionResult ThongTin()
{
    var info = new OSInfo
    {
        CPU = Environment.ProcessorCount,
        OS = Environment.OSVersion.ToString()
    };
    return View(info); // Đưa cho Trang Trí bày
}

// Trang Trí (View): @model OSInfo → @Model.CPU, @Model.OS
```

## 14.2. Bài 2: Todo List — Thực hành DI & Clean Architecture

Chia app thành 4 tầng — từ trong ra ngoài như bắp cải:

```
🧅 Lõi:        TodoList.Entity/         ← Chỉ có class TodoItem
🧅 Giữa:       TodoList.UseCase/        ← Interface ITodoRepository
🧅 Ngoài:      TodoList.Infrastructure/ ← InMemoryRepository (triển khai)
🧅 Vỏ ngoài:   TodoList.Web/            ← ASP.NET MVC (giao diện)
```

Lõi không biết Vỏ tồn tại. Vỏ không biết Lõi nấu cách nào. Chỉ giao tiếp qua Interface (hợp đồng) → đổi Database sau này chỉ cần viết class mới, **không sửa 1 dòng code cũ**.

---

> 🎯 **Module tiếp theo**: [Module 02 — HTTP & Middleware Pipeline](Module-02-HTTP-Middleware.md)
