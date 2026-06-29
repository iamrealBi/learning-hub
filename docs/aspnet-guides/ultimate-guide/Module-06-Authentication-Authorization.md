# 📘 Module 06: Authentication & Authorization
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [Authentication vs Authorization — Lễ tân và Bảo vệ](#1-authn-vs-authz)
2. [Kiến trúc Authentication — Hệ thống cổng an ninh](#2-kiến-trúc)
3. [Cookie Authentication — Vòng tay fluorescent](#3-cookie-auth)
4. [JWT Bearer — Tấm vé điện tử](#4-jwt)
5. [ASP.NET Core Identity — Phòng nhân sự đóng hộp](#5-identity)
6. [Two-Factor Authentication — Khóa cửa 2 lớp](#6-2fa)
7. [Role-based Authorization — Phân loại thẻ VIP](#7-role)
8. [Policy-based Authorization — Luật nội quy linh hoạt](#8-policy)
9. [OAuth 2.0 & OpenID Connect — Hộ chiếu quốc tế](#9-oauth)
10. [Bảo mật: XSS, CSRF, HSTS — Áo giáp chống hack](#10-bảo-mật)

---

# 1. Authentication vs Authorization — Lễ tân và Bảo vệ

Hai từ viết giống nhau kinh khủng, nhưng là **2 người khác nhau** hoàn toàn:

| | Lễ Tân (Authentication) | Bảo vệ (Authorization) |
|---|---|---|
| **Câu hỏi** | "Bạn là AI?" (Who?) | "Bạn được làm GÌ?" (What?) |
| **Ví dụ** | Bạn đưa CCCD → Lễ tân: "OK, anh Nghĩa" | Bạn vào thang máy → Bảo vệ: "Anh chỉ lên tầng 3, tầng Giám Đốc bị CẤM!" |
| **Middleware** | `app.UseAuthentication()` | `app.UseAuthorization()` |

> ⚠️ **THỨ TỰ LÀ SỐNG CÒN**: Lễ tân phải ngồi **TRƯỚC** Bảo vệ. Nếu Bảo vệ kiểm tra trước mà chưa biết bạn là ai → đuổi hết mọi người!

---

# 2. Kiến trúc — Hệ thống cổng an ninh

```
Cổng an ninh (Authentication Middleware)
    └── Phòng kiểm tra (Authentication Service)
            └── Danh sách loại thẻ (Schemes)
                    ├── Vòng tay fluorescent (Cookie Scheme)
                    ├── Tấm vé điện tử (JWT Bearer Scheme)
                    └── Hộ chiếu quốc tế (OpenID Connect Scheme)
```

### Mỗi loại thẻ có 3 chức năng:

| Chức năng | Khi nào | Ẩn dụ |
|---|---|---|
| **Authenticate** | Mỗi người vào → kiểm tra thẻ | Soi vòng tay: "Còn sáng không?" |
| **Challenge** | Chưa có thẻ mà đòi vào VIP | "Anh chưa đeo vòng! Về quầy Lễ tân!" |
| **Forbid** | Có thẻ nhưng không đủ quyền | "Vòng anh màu xanh — khu này cần vòng ĐỎ!" |

### Claims = Thông tin in trên thẻ

Thẻ của bạn in: Tên: Nghĩa, Tuổi: 28, Phòng Ban: IT, Chức vụ: Dev. Mỗi dòng chữ in = 1 **Claim**:

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "Nghĩa"),     // Tên in trên thẻ
    new Claim(ClaimTypes.Email, "a@b.com"),   // Email
    new Claim(ClaimTypes.Role, "Admin")       // Phân loại thẻ
};
```

---

# 3. Cookie Authentication — Vòng tay fluorescent

Bạn vào công viên nước. Lễ tân kiểm tra CCCD → **đeo vòng tay sáng** cho bạn. Cả ngày bạn đi đâu trong công viên, bảo vệ chỉ cần soi vòng tay → "Anh còn vòng, vào luôn!"

## 3.1. Đăng ký

```csharp
builder.Services.AddAuthentication(opt => opt.DefaultScheme = "MyCookie")
.AddCookie("MyCookie", opt =>
{
    opt.Cookie.Name = "MyApp.Auth";      // Tên vòng tay
    opt.LoginPath = "/Account/Login";    // Quầy Lễ tân
});

app.UseAuthentication();  // Đặt Lễ tân
app.UseAuthorization();   // Đặt Bảo vệ SAU Lễ tân
```

## 3.2. Đeo vòng tay (SignIn)

```csharp
[HttpPost]
public async Task<IActionResult> Login(LoginModel m)
{
    var user = _repo.FindByUsername(m.Username);
    if (user == null) return View("NotFound");

    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.Name, user.Username),
        new Claim(ClaimTypes.Role, "Admin")
    };

    // PHẢI truyền tên scheme — nếu không, IsAuthenticated = false MÃMÃI!
    var identity = new ClaimsIdentity(claims, "MyCookie");
    var principal = new ClaimsPrincipal(identity);

    await HttpContext.SignInAsync(principal); // ĐEO VÒNG TAY!
    return RedirectToAction("Index", "Home");
}
```

> 💡 Cookie Auth **KHÔNG dùng Session**: thông tin khách được **mã hóa rồi nhét luôn vào vòng tay**. Server không nhớ gì cả.

---

# 4. JWT Bearer — Tấm vé điện tử

## 4.1. Tại sao cần vé điện tử khi đã có vòng tay?

Vòng tay (Cookie) chỉ hoạt động trong **1 công viên** (1 domain). Nếu bạn có chuỗi 3 công viên (3 server), vòng tay chi nhánh A **không sáng** ở chi nhánh B!

**JWT** = tấm vé điện tử ghi sẵn thông tin + dấu mộc công ty. **Bất kỳ chi nhánh nào** soi dấu mộc → "Đúng dấu công ty → cho vào!" Không cần gọi về tổng đài kiểm tra → **Stateless**.

## 4.2. Cấu trúc vé điện tử

```
[Tiêu đề] . [Nội dung] . [Dấu mộc đỏ]
Header    . Payload     . Signature
```

- **Header**: "Tôi dùng thuật toán HS256"
- **Payload**: "Tên: Nghĩa, Role: Admin, Hết hạn: 15 phút nữa"
- **Signature**: Dấu mộc bí mật — **AI GIẢ MẠO ĐỀU BỊ PHÁT HIỆN**

> *"JWT không mã hóa để người khác không đọc. JWT KÝ để đảm bảo không ai sửa được."* — YouTube

## 4.3. Setup

```csharp
builder.Services.AddAuthentication()
    .AddJwtBearer(opt =>
    {
        opt.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,            // Vé này do AI cấp?
            ValidIssuer = "https://auth.com",
            ValidateLifetime = true,           // Vé còn hạn không?
            ValidateIssuerSigningKey = true,   // Dấu mộc có thật không?
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes("bi-mat-toi-thieu-32-ky-tu-nhe!!"))
        };
    });
```

## 4.4. Tạo vé

```csharp
[HttpPost("login")]
public IActionResult Login(LoginDto dto)
{
    // Xác thực username/password...
    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.Name, user.Name),
        new Claim(ClaimTypes.Role, "Admin")
    };

    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("bi-mat-32-ky-tu!!"));
    var token = new JwtSecurityToken(
        issuer: "https://auth.com",
        claims: claims,
        expires: DateTime.UtcNow.AddMinutes(15), // HẾT HẠN NGẮN!
        signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256));

    return Ok(new { Token = new JwtSecurityTokenHandler().WriteToken(token) });
}
```

## 4.5. Hết hạn ngắn + Refresh Token

Tại sao vé chỉ 15 phút? Nếu tên trộm chụp được vé → xài MÃMÃI → **NGUY**!

- **JWT** (15 phút) → hết hạn → tên trộm vứt sọt rác
- **Refresh Token** (dài hạn, lưu trong DB) → đổi JWT mới → dễ **khóa** khi nghi ngờ bị trộm

---

# 5. ASP.NET Core Identity — Phòng nhân sự đóng hộp

## 5.1. Tại sao ĐỪNG tự code đăng nhập?

Nếu lưu mật khẩu dạng text thuần (`Password = "123456"`) → DB rò rỉ → Hacker lấy mật khẩu → Đăng nhập Facebook, Ngân hàng của user → **BẠN CHỊU TRÁCH NHIỆM PHÁP LÝ!**

**Identity** = Microsoft giao bạn cả **phòng nhân sự có sẵn**:

- Tự tạo 7 bảng Security trong DB
- Tự **băm** mật khẩu thành mã không ai giải ngược được
- Quên mật khẩu, khóa tài khoản sai 5 lần, xác nhận email — TẤT CẢ CÓ SẴN

```csharp
builder.Services.AddIdentity<AppUser, IdentityRole>(opt =>
{
    opt.Password.RequiredLength = 8;            // Tối thiểu 8 ký tự
    opt.Password.RequireNonAlphanumeric = true; // Phải có @#!
    opt.Lockout.MaxFailedAccessAttempts = 5;    // Sai 5 lần → khóa 15 phút
})
.AddEntityFrameworkStores<AppDbContext>();

// Đăng ký user — 1 dòng, Identity TỰ BĂMMẬT KHẨU
var result = await _userManager.CreateAsync(user, "P@ssword123");
```

---

# 6. 2FA — Khóa cửa 2 lớp

**Mật khẩu** = khóa cổng. Bị trộm chìa → vào được.
**2FA** = thêm khóa vân tay → trộm có chìa nhưng KHÔNG có ngón tay bạn → KHÔNG VÀO ĐƯỢC.

Ứng dụng thực tế: Login → nhập mật khẩu → mở Google Authenticator → nhập 6 số → xong.

---

# 7. Role-based Authorization — Phân loại thẻ VIP

```csharp
// Chỉ thẻ ĐỎ (Admin) mới vào
[Authorize(Roles = "Admin")]
public IActionResult Dashboard() => View();

// Thẻ ĐỎ HOẶC VÀNG đều vào được (OR)
[Authorize(Roles = "Admin,Manager")]
public IActionResult Reports() => View();

// Phải có CẢ thẻ ĐỎ VÀ VÀNG (AND)
[Authorize(Roles = "Admin")]
[Authorize(Roles = "Manager")]
public IActionResult SuperSecret() => View();

// Ai vào cũng được — cổng mở tự do
[AllowAnonymous]
public IActionResult Public() => View();
```

| Cú pháp | Logic | Ẩn dụ |
|---|---|---|
| `Roles = "A,B"` | **HOẶC** | "Có 1 trong 2 thẻ là vào" |
| `[Auth(A)]` + `[Auth(B)]` | **VÀ** | "Phải đeo CẢ 2 thẻ" |

---

# 8. Policy-based Authorization — Luật nội quy linh hoạt

Role đôi khi quá cứng nhắc. Policy = **luật nội quy** — linh hoạt hơn:

```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("PhongKeToan", policy =>
    {
        policy.RequireRole("Accounting");
        policy.RequireClaim("Department", "Finance");
    })
    .AddPolicy("NguoiLon", policy =>
    {
        policy.RequireAssertion(ctx =>
        {
            var dob = ctx.User.FindFirst(ClaimTypes.DateOfBirth)?.Value;
            return DateTime.TryParse(dob, out var date) &&
                   DateTime.Today.Year - date.Year >= 18;
        });
    });

[Authorize(Policy = "PhongKeToan")]
public IActionResult XemLuong() => Ok("Lương: 50 triệu");
```

---

# 9. OAuth 2.0 & OpenID Connect — Hộ chiếu quốc tế

## 9.1. Câu chuyện OAuth 2.0

Bạn vào 1 web xem phim. Nó đòi đăng nhập. Bạn lười tạo tài khoản → bấm **"Login bằng Google"**:

```
1. 🎬 Web phim: "Anh chưa có vé! Qua quầy Google lấy vé đi"
2. 🏢 Google: "Anh nhập mật khẩu Google đi" (web phim KHÔNG biết MK của bạn!)
3. 🏢 Google: "Web phim muốn xin Tên + Email. Cho?"
4. 👤 Bạn: "Cho!"
5. 🏢 Google cấp 1 tấm vé → ném bạn + vé về web phim
6. 🎬 Web phim cầm vé → gọi Google: "Vé này thật không?"
7. 🏢 Google: "Thật!" → Web phim mở cửa cho bạn xem phim 🎉
```

```csharp
// 2 dòng setup — cái flow phức tạp phía trên Microsoft lo hết!
builder.Services.AddAuthentication()
    .AddGoogle(opt =>
    {
        opt.ClientId = "MA_GOOGLE_CAP";
        opt.ClientSecret = "MAT_KHAU_GOOGLE_CAP";
    });
```

## 9.2. Symmetric vs Asymmetric Key

| | Chìa khóa chung (HS256) | Chìa khóa đôi (RS256) |
|---|---|---|
| **Ẩn dụ** | 1 chìa mở + khóa — ai có đều giả mạo được | 1 chìa khóa (private) + 1 chìa mở (public) |
| **Dùng cho** | Nội bộ công ty | Hệ thống phân tán, OIDC |

---

# 10. Bảo mật — Áo giáp chống hack

## 10.1. XSS — Chích thuốc độc vào HTML
Hacker comment: `"Bài hay! <script>alert('Hacked')</script>"`. Trình duyệt ngu ngốc chạy code đó!
**Chống**: Razor tự động biến `<script>` thành text vô hại. Bạn không cần làm gì!

## 10.2. CSRF — Giả mạo đơn hàng
Bạn login Facebook. Tab khác lừa submit form "Chuyển tiền" → Facebook tưởng bạn.
**Chống**: `[ValidateAntiForgeryToken]` — form có mã bí mật, trang giả không có mã → BỊ CHẶN.

## 10.3. HTTPS/HSTS — Áo giáp đường truyền
Hacker ở WiFi quán café chặn HTTP plaintext → đọc mật khẩu.
**Chống**: `UseHttpsRedirection()` + `UseHsts()` — mã hóa mọi thứ.

---

> 🎯 **Module tiếp theo**: [Module 07 — Minimal API & .NET Aspire](Module-07-Minimal-API-Aspire.md)
