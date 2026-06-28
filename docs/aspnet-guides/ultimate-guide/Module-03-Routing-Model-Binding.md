# 📘 Module 03: Routing & Model Binding (Chi tiết)
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [Routing nâng cao — GPS chuyên nghiệp](#1-routing-nâng-cao)
2. [Model Binding — Phiên dịch viên tự động](#2-model-binding)
3. [Data Validation — Hàng rào bảo vệ](#3-data-validation)
4. [Session thực tế — Kho đồ phân tán](#4-session-thực-tế)
5. [HTTPS — Lớp áo giáp đường truyền](#5-https)

---

# 1. Routing nâng cao — GPS chuyên nghiệp

## 1.1. Hai loại GPS nhắc lại

| | GPS Trung Tâm (Conventional) | GPS Cá Nhân (Attribute) |
|---|---|---|
| **Gắn ở đâu** | Cổng chính — `Program.cs` | Trên biển tên từng nhân viên |
| **Khi nào dùng** | Quán nhỏ, ít đường | Quán lớn, nhiều ngõ ngách |
| **Ưu tiên** | Đường nào khai trước thắng | Đường cụ thể hơn thắng |

## 1.2. Nhiều biển chỉ cùng 1 nơi
Giống quán có 2 cửa vào cùng dẫn đến 1 phòng:
```csharp
[Route("p/{id}")]         // Cửa 1: /p/123
[Route("product/{id}")]   // Cửa 2: /product/123
public IActionResult Details(int id) => View();
```

## 1.3. Barie chặn đường — Route Constraints

Tưởng tượng đường hầm có barie: "Xe cao trên 2m không vào!", "Xe máy cấm!"

| Barie | Ý nghĩa | Cho qua | Chặn lại |
|---|---|---|---|
| `{id:int}` | Chỉ cho số qua | `/p/123` ✅ | `/p/abc` ❌ |
| `{name:alpha}` | Chỉ cho chữ cái | `/p/laptop` ✅ | `/p/123` ❌ |
| `{id:min(1)}` | Tối thiểu 1 | `/p/1` ✅ | `/p/0` ❌ |
| `{id:range(1,100)}` | Trong khoảng 1-100 | `/p/50` ✅ | `/p/200` ❌ |
| `{name:maxlength(50)}` | Dài tối đa 50 ký tự | Ngắn ✅ | Dài quá ❌ |
| `{slug:regex(^[a-z-]+$)}` | Chỉ chữ thường + gạch | `/p/my-product` ✅ | `/p/My_Pro!` ❌ |

## 1.4. `[controller]` và `[action]` — Tên tự thay

```csharp
[Route("api/[controller]")]  // ASP.NET tự thay → api/Products
public class ProductsController : ControllerBase
{
    [HttpGet("[action]")]    // Tự thay → api/Products/GetAll
    public IActionResult GetAll() => Ok();
}
```

Như viết trên đồng phục: "Tôi là [tên nhân viên]" → mỗi người mặc vào, tên tự đổi.

## 1.5. Area — Chia quán thành tầng

Quán lớn quá? Chia thành **tầng riêng biệt** (Admin, Shop, Blog):

```
Areas/
├── Admin/        ← Tầng quản lý (chỉ ông chủ vào)
│   ├── Controllers/DashboardController.cs
│   └── Views/Dashboard/Index.cshtml
├── Shop/         ← Tầng bán hàng (khách vào)
│   └── Controllers/ProductController.cs
```

```csharp
[Area("Admin")]  // Nhân viên này làm ở tầng Admin
public class DashboardController : Controller { }

// GPS hiểu: /Admin/Dashboard/Index
```

---

# 2. Model Binding — Phiên dịch viên tự động

## 2.1. Model Binding là gì?

Khách gửi đơn hàng viết bằng **tiếng text thô** (chuỗi URL, form data). Nhưng Tiếp Tân cần **đơn hàng chuẩn** (object C#). **Model Binding** = phiên dịch viên tự động dịch text → object.

```
Khách gửi: "id=100&name=ABC&year=2024"  (text thô)
         ↓ [Phiên dịch viên]
Tiếp Tân nhận: Person { Id=100, Name="ABC", Year=2024 }  (object C#)
```

## 2.2. Phiên dịch viên tìm data ở đâu?

Phiên dịch tìm theo thứ tự, ai có trước dùng trước:

| Ưu tiên | Nơi tìm | Ẩn dụ |
|---|---|---|
| 1️⃣ | **Form Data** | Phiếu đăng ký khách điền bằng tay |
| 2️⃣ | **Route Values** | Số phòng khắc trên chìa khóa (`/users/{id}`) |
| 3️⃣ | **Query String** | Ghi chú dán bên ngoài phong bì (`?name=ABC`) |

## 2.3. Kiểu đơn giản — dịch 1 ô

```csharp
public IActionResult Index(int id, string name, int? year)
```

| Khách gửi | Phiên dịch hiểu | Lưu ý |
|---|---|---|
| `?id=1000&name=ABC` | id=1000, name="ABC" | ✅ Bình thường |
| `?id=abc` | id=**0** (mặc định) | ⚠️ Dịch sai nhưng KHÔNG báo lỗi! |
| `?id=abc` (kiểu `int?`) | id=**null** | Nullable thì trả null |

> ⚠️ Khi dịch thất bại, phiên dịch viên **im lặng gán mặc định** thay vì báo lỗi → phải tự kiểm tra `ModelState.IsValid`!

## 2.4. Kiểu phức tạp — dịch cả tờ đơn

```csharp
public class Person { public int Id { get; set; } public string Name { get; set; } }
public IActionResult Index(Person person) { ... }
```

**Hai chế độ điền đơn:**

| Cách điền | Query | Kết quả |
|---|---|---|
| Trực tiếp | `?id=100&name=ABC` | ✅ Dịch được |
| Có mã số đơn | `?person.id=100&person.name=ABC` | ✅ Dịch được |
| **Trộn lẫn** 💀 | `?person.id=100&name=ABC` | ❌ name="" (MẤT!) |

> ⚠️ **Bẫy chết người**: Nếu đã dùng `person.id` (có prefix) thì TẤT CẢ phải có prefix. Trộn lẫn = phiên dịch viên **bối rối**, bỏ sót data!

## 2.5. Dịch danh sách — nhiều ô cùng tên

```csharp
public IActionResult Index(string[] items) { ... }
```
```
GET /Demo?items=Phở&items=Bún&items=Mì
→ items = ["Phở", "Bún", "Mì"]  // Lặp lại tên = danh sách!
```

## 2.6. `[ApiController]` — Phiên dịch viên VIP

Khi Controller có `[ApiController]`, phiên dịch viên thông minh hơn:
- Object → tự hiểu `[FromBody]` (đọc body JSON)
- Đơn hàng sai → **tự động trả 400** (không cần viết `if (!ModelState.IsValid)`)

---

# 3. Data Validation — Hàng rào bảo vệ

## 3.1. Hai lớp hàng rào

| | Hàng rào bên ngoài (Client-side) | Hàng rào bên trong (Server-side) |
|---|---|---|
| **Chạy ở** | Trình duyệt (JavaScript) | Server (C#) |
| **Ẩn dụ** | Biển "Cấm vào" ở cổng — ai cũng thấy | Ổ khóa bên trong — không ai mở được |
| **Tốc độ** | Nhanh — phản hồi tức thì | Cần gửi về server |
| **Bảo mật** | ❌ Khách có thể tháo biển (tắt JavaScript) | ✅ Không tháo được |
| **Kết luận** | Trang trí đẹp nhưng KHÔNG ĐỦ | **BẮT BUỘC PHẢI CÓ** |

> ⚠️ **Hacker tắt JavaScript trong 2 giây** → mọi kiểm tra client-side biến mất. **Server-side là tuyến phòng thủ cuối cùng!**

## 3.2. Bảng hàng rào Data Annotations

| Hàng rào | Chức năng | Trong đời thực |
|---|---|---|
| `[Required]` | Bắt buộc điền | "Ô này không được để trống!" |
| `[StringLength(50)]` | Giới hạn chữ | "Tên dài quá 50 ký tự rồi!" |
| `[Range(0, 999999)]` | Giới hạn số | "Giá không thể âm!" |
| `[EmailAddress]` | Phải đúng email | "Cái này đâu phải email?" |
| `[Phone]` | Phải đúng SĐT | "SĐT gì kỳ vậy?" |
| `[Compare("MK")]` | Khớp với ô khác | "Mật khẩu xác nhận không khớp!" |
| `[ValidateNever]` | Bỏ qua kiểm tra | "Ô này server tự điền, kệ nó" |

## 3.3. ModelState — Bản kết quả kiểm tra

```csharp
[HttpPost]
public IActionResult TaoSanPham(ProductDto dto)
{
    if (!ModelState.IsValid) // Có hàng rào nào bị vi phạm không?
    {
        return BadRequest(ModelState); // Trả về danh sách lỗi chi tiết
    }
    // Qua hết hàng rào → xử lý
}
```

---

# 4. Session thực tế — Kho đồ phân tán

## 4.1. Cấu hình Session chi tiết

```csharp
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30); // 30 phút không vào → tủ tự khóa
    options.Cookie.Name = "MyApp.Session";
    options.Cookie.HttpOnly = true;   // JavaScript KHÔNG đọc được chìa khóa
    options.Cookie.IsEssential = true; // Không cần hỏi "Bạn có đồng ý dùng cookie?"
});
app.UseSession();
```

## 4.2. Hành vi chìa khóa (Session Cookie)

| Tình huống | Chuyện gì xảy ra |
|---|---|
| Mở tab mới cùng browser | **Dùng chung** tủ (cùng chìa khóa) |
| Mở trình duyệt khác | Tủ **MỚI** (chìa khóa khác) |
| 30 phút không vào | Tủ tự **KHÓA VĨNH VIỄN** |
| Chìa khóa bị mất (xóa cookie) | Tủ còn đó nhưng **không ai mở được** |
| Chìa khóa bị **TRỘM** 💀 | Tên trộm mở được tủ → lấy hết đồ! |

> Tác giả YouTube demo trực tiếp: copy cookie từ Chrome → dán vào Postman → **đọc được session data!** Căn cứ duy nhất để biết ai là ai = **chiếc chìa khóa cookie**.

---

# 5. HTTPS — Lớp áo giáp đường truyền

## 5.1. HTTP = Gửi thư bình thường, HTTPS = Gửi thư bảo mật

| | HTTP | HTTPS |
|---|---|---|
| **Ẩn dụ** | Gửi bưu thiếp — ai cầm cũng đọc được | Gửi phong bì đóng kín — chỉ người nhận mở |
| **Bảo mật** | ❌ Hacker ở quán café WiFi đọc hết | ✅ Mã hóa — chỉ thấy ký tự lộn xộn |
| **Port** | 80 | 443 |
| **SEO** | ❌ Google đánh thấp | ✅ Google ưu tiên |

## 5.2. Hai cách ép khách dùng HTTPS

```csharp
app.UseHttpsRedirection(); // Khách gõ HTTP → Quán hét: "Đi sang cổng HTTPS!"
app.UseHsts();             // Dặn khách: "Lần sau tự đi cổng HTTPS, đừng ghé HTTP nữa"
```

## 5.3. Mẹo cho API

Nếu viết API (phục vụ mobile, server khác) → **Đừng mở cổng HTTP**. Chỉ mở HTTPS. Đơn giản, dứt khoát!

Lý do: Trình duyệt sẽ tự nghe lệnh chuyển hướng. Nhưng app mobile/server khác thì không chắc — có thể bị lỗi.

## 5.4. Let's Encrypt — Áo giáp miễn phí

Ngày xưa phải mua "áo giáp" (SSL Certificate) tốn tiền. Giờ có **Let's Encrypt** — đăng ký miễn phí, tự động gia hạn. Món hời!

---

> 🎯 **Module tiếp theo**: [Module 04 — Web API & RESTful Services](Module-04-Web-API-REST.md)
