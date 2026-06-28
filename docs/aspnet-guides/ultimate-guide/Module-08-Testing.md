# 📘 Module 08: Testing
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [Tại sao cần Test? — Bảo hiểm cho code](#1-tại-sao)
2. [Unit Test — Kiểm tra từng viên gạch](#2-unit-test)
3. [Testing EF Core — Xây DB giả](#3-testing-ef-core)
4. [Integration Test — Kiểm tra cả tòa nhà](#4-integration-test)
5. [Mocking — Thuê diễn viên đóng thế](#5-mocking)

---

# 1. Tại sao cần Test? — Bảo hiểm cho code

| Không có test | Có test |
|---|---|
| Sửa phòng A → vô tình sập phòng B → **không biết** | Sửa phòng A → chuông báo: "Phòng B bị ảnh hưởng!" |
| Mỗi lần sửa code → tự bấm tay kiểm tra 100 trang | Bấm 1 nút → 1000 test tự chạy trong 10 giây |
| Deploy xong → khách la: "Lỗi!" 😱 | Deploy tự tin → test đã cover hết ✅ |

---

# 2. Unit Test — Kiểm tra từng viên gạch

## 2.1. Quy trình AAA (Arrange – Act – Assert)

Giống kiểm tra chất lượng 1 viên gạch:

```csharp
[Fact] // "Đây là 1 bài kiểm tra"
public void GuiTien_TangSoDu()
{
    // Arrange — CHUẨN BỊ: Lấy 1 viên gạch
    var tk = new Account { Balance = 100 };

    // Act — THỬ: Đập, uốn, bẻ
    tk.Balance += 50;

    // Assert — KIỂM TRA: Có đạt chuẩn không?
    Assert.Equal(150, tk.Balance); // Phải bằng 150!
}
```

## 2.2. Fact vs Theory

```csharp
[Fact] // Kiểm tra 1 trường hợp
public void Cong_TraVeTong() => Assert.Equal(5, Calculator.Add(2, 3));

[Theory] // Kiểm tra NHIỀU trường hợp — 1 công thức, nhiều bộ số
[InlineData(2, 3, 5)]     // 2+3=5
[InlineData(0, 0, 0)]     // 0+0=0
[InlineData(-1, 1, 0)]    // -1+1=0
public void Cong_NhieuTruongHop(int a, int b, int expected)
{
    Assert.Equal(expected, Calculator.Add(a, b));
}
```

## 2.3. Bảng "câu hỏi kiểm tra" phổ biến

```csharp
Assert.Equal(expected, actual);         // "Kết quả có đúng không?"
Assert.NotNull(obj);                     // "Có tồn tại không?"
Assert.True(condition);                  // "Có đúng không?"
Assert.Contains("text", list);          // "Có chứa cái này không?"
Assert.Throws<Exception>(() => code()); // "Phải nổ lỗi khi chạy code này!"
Assert.Empty(list);                      // "Phải rỗng!"
```

---

# 3. Testing EF Core — Xây DB giả

## 3.1. SQLite InMemory — DB mini trong RAM

Không cần SQL Server thật! Xây 1 **DB tí hon trong bộ nhớ** → test xong biến mất:

```csharp
public class AccountTests : IDisposable
{
    private readonly SqliteConnection _conn;
    private readonly AppDbContext _db;

    public AccountTests()
    {
        // Xây DB mini
        _conn = new SqliteConnection("DataSource=:memory:");
        _conn.Open(); // PHẢI mở! Đóng → DB biến mất!

        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseSqlite(_conn).Options;

        _db = new AppDbContext(options);
        _db.Database.EnsureCreated(); // Tạo bảng từ Entity models
    }

    [Fact]
    public async Task TaoKhachHang_LuuVaoDB()
    {
        var kh = new Customer { Name = "Test", Email = "test@test.com" };
        _db.Customers.Add(kh);
        await _db.SaveChangesAsync();

        var found = await _db.Customers.FirstOrDefaultAsync(c => c.Email == "test@test.com");
        Assert.NotNull(found);
        Assert.Equal("Test", found.Name);
    }

    public void Dispose() { _db.Dispose(); _conn.Dispose(); } // Dọn dẹp
}
```

---

# 4. Integration Test — Kiểm tra cả tòa nhà

Unit Test = kiểm tra từng viên gạch. **Integration Test** = kiểm tra **cả căn nhà**: mở cửa, bật điện, bật nước, đi từ phòng này sang phòng khác, mọi thứ có hoạt động cùng nhau không?

## 4.1. WebApplicationFactory — Xây quán ảo để test

```csharp
public class ApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public ApiTests(WebApplicationFactory<Program> factory)
        => _client = factory.CreateClient(); // Mở quán ảo hoàn chỉnh!

    [Fact]
    public async Task GetProducts_TraVe200()
    {
        var response = await _client.GetAsync("/api/products");
        response.EnsureSuccessStatusCode(); // Phải thành công!
    }

    [Fact]
    public async Task TaoVaXem_LuongDayDu()
    {
        // Tạo sản phẩm mới
        var resp = await _client.PostAsJsonAsync("/api/products",
            new { Name = "Test", Price = 99 });
        Assert.Equal(HttpStatusCode.Created, resp.StatusCode);

        // Xem sản phẩm vừa tạo
        var location = resp.Headers.Location!.ToString();
        var getResp = await _client.GetAsync(location);
        var product = await getResp.Content.ReadFromJsonAsync<Product>();
        Assert.Equal("Test", product!.Name);
    }
}
```

---

# 5. Mocking — Thuê diễn viên đóng thế

## 5.1. Tại sao Mock?

Unit test chỉ kiểm tra **1 viên gạch**. Nhưng viên gạch đó dính chung với bức tường (Database). Tách tường ra → phiền!

**Mock** = thuê **diễn viên đóng thế bức tường**. Diễn viên giả vờ là tường, nhưng bạn kiểm soát hoàn toàn: "Khi hỏi, trả lời cái này!"

```csharp
[Fact]
public void GetAll_GoiKhoHang()
{
    // Thuê diễn viên giả làm kho hàng
    var fakeKho = new Mock<IProductRepository>();
    fakeKho.Setup(k => k.GetAll()).Returns(new List<Product>
    {
        new Product { Id = 1, Name = "Laptop" }
    });

    var controller = new ProductController(fakeKho.Object); // Tiếp Tân nhận diễn viên

    var result = controller.GetAll() as OkObjectResult;

    Assert.NotNull(result);
    fakeKho.Verify(k => k.GetAll(), Times.Once); // Kiểm tra: đã gọi kho đúng 1 lần!
}
```

## 5.2. Khi nào Mock vs SQLite InMemory?

| | Diễn viên đóng thế (Mock) | DB mini (SQLite) |
|---|---|---|
| **Test gì** | Logic code (if/else, tính toán) | Query LINQ, EF Core |
| **Ẩn dụ** | Diễn viên giả trả câu trả lời bạn muốn | Xây phòng thí nghiệm mini |
| **Tốc độ** | Cực nhanh | Nhanh |

---

> 🎯 **Module tiếp theo**: [Module 09 — Docker & Deployment](Module-09-Docker-Deployment.md)
