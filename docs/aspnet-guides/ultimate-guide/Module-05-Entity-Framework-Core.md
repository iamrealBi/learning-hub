# 📘 Module 05: Entity Framework Core
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [ORM — Thông dịch viên giữa C# và Database](#1-orm)
2. [Cài đặt EF Core](#2-cài-đặt)
3. [Entity Model & DbContext — Bản vẽ và Cây cầu](#3-entity--dbcontext)
4. [Code-First — Xây nhà rồi mới vẽ bản đồ](#4-code-first)
5. [Database Migrations — Thợ sửa nhà tự động](#5-migrations)
6. [Quan hệ dữ liệu — Gia phả](#6-quan-hệ)
7. [LINQ — Thần chú truy vấn](#7-linq)
8. [Repository & Unit of Work — Nhân viên kho](#8-repository--unit-of-work)
9. [Tối ưu hiệu suất — Bí kíp tăng tốc](#9-tối-ưu)
10. [Data Seeding & Encryption — Hàng mẫu và két sắt](#10-seeding--encryption)
11. [Pagination — Lật trang sách](#11-pagination)

---

# 1. ORM — Thông dịch viên giữa C# và Database

## 1.1. Hai thế giới khác ngôn ngữ

C# nói tiếng **Object**: `sach.TenSach = "Harry Potter"`.
Database nói tiếng **SQL Table**: `INSERT INTO Sach (TenSach) VALUES ('Harry Potter')`.

Hai bên không hiểu nhau. Bạn phải tự dịch SQL thủ công → **mệt, dễ sai, dễ bị hack SQL Injection**.

**ORM (Object-Relational Mapping)** = thuê 1 **thông dịch viên chuyên nghiệp**. Bạn chỉ nói tiếng C#, thông dịch viên tự dịch sang SQL.

```csharp
// Không có ORM — tự viết SQL (nguy hiểm!)
string sql = "INSERT INTO Sach (Ten) VALUES ('" + sach.Ten + "')"; // SQL Injection!

// Có ORM — chỉ cần nói tiếng C#
db.Sachs.Add(sach);        // Thông dịch viên tự dịch
await db.SaveChangesAsync(); // Tự gửi SQL an toàn
```

## 1.2. Entity Framework Core = Thông dịch viên chính hãng Microsoft
Hỗ trợ nhiều "quốc gia" (Database): SQL Server, PostgreSQL, MySQL, SQLite...
Đổi "quốc gia"? **Chỉ sửa 1 dòng khai báo** — toàn bộ code C# giữ nguyên!

---

# 2. Cài đặt

```bash
# "Từ điển" cho từng quốc gia
dotnet add package Microsoft.EntityFrameworkCore.SqlServer   # SQL Server
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL      # PostgreSQL

# Công cụ thợ sửa nhà (Migration)
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

---

# 3. Entity Model & DbContext

## 3.1. Entity Model = Bản vẽ kiến trúc

1 Class = 1 Bảng (Table). 1 Property = 1 Cột (Column). Như bản vẽ nhà: mỗi phòng = 1 bảng, mỗi ô kích thước = 1 cột.

```csharp
public class TacGia
{
    public int Id { get; set; } // EF Core thấy "Id" → hiểu đây là chìa khóa chính, tự tăng

    [Required]            // Cột này bắt buộc phải có giá trị
    [StringLength(50)]    // Dài tối đa 50 ký tự
    public string Ten { get; set; }

    // "1 Tác giả viết NHIỀU sách" — mối quan hệ gia đình
    public ICollection<Sach> CacCuonSach { get; set; }
}
```

## 3.2. DbContext = Cây cầu nối C# ↔ Database

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    // Mỗi DbSet = 1 cánh cổng dẫn vào 1 bảng
    public DbSet<TacGia> TacGias { get; set; }  // Cổng → bảng TacGia
    public DbSet<Sach> Sachs { get; set; }      // Cổng → bảng Sach

    protected override void OnModelCreating(ModelBuilder mb)
    {
        mb.Entity<TacGia>().ToTable("Authors"); // Đổi tên bảng nếu muốn
    }
}
```

## 3.3. Đăng ký cây cầu

```csharp
// Program.cs — Bảo ASP.NET: "Cầu của tôi nối đến DB ở đây"
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

```json
// appsettings.json — Địa chỉ kho
{ "ConnectionStrings": { "Default": "Server=(localdb)\\mssqllocaldb;Database=MyApp;..." } }
```

---

# 4. Code-First — Xây nhà rồi mới vẽ bản đồ

| | Code-First ✅ (Xây trước) | Database-First (Bản đồ trước) |
|---|---|---|
| **Luồng** | Viết Class C# → EF tạo DB tự động | DB có sẵn → Tool sinh Class |
| **Ẩn dụ** | Xây nhà → thợ tự vẽ bản đồ theo | Có bản đồ cũ → rập khuôn xây tiếp |
| **Khi nào** | Dự án mới, bạn kiểm soát mọi thứ | DB kế thừa từ đời trước |

> 💡 Cả Coursera lẫn YouTube đều dạy **Code-First** — đây là cách chuẩn của EF Core hiện đại.

---

# 5. Migrations — Thợ sửa nhà tự động

## 5.1. Vấn đề

Bạn thêm thuộc tính `public int Tuoi { get; set; }` vào Model. Chạy trên máy đồng nghiệp → DB đồng nghiệp **không có cột Tuoi** → SẬP. **Migration** = ghi sổ cần sửa gì → bất kỳ ai chạy đều tự cập nhật.

## 5.2. Quy trình 2 bước

```bash
# Bước 1: GHI SỔ — "Hôm nay tôi thêm cột Tuoi"
dotnet ef migrations add ThemCotTuoi

# Bước 2: THỢ ĐẬP TƯỜNG — áp dụng sổ vào DB thật
dotnet ef database update

# Quay lại xây tường cũ (rollback)
dotnet ef database update TenMigrationCu
```

> "Migration chỉ viết giấy. Phải `update` mới thấy tường thay đổi."

---

# 6. Quan hệ dữ liệu — Gia phả

## 6.1. One-to-Many (1 cha → Nhiều con)

1 **Tác giả** viết nhiều **Sách**:

```csharp
public class TacGia
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Sach> DsSach { get; set; } // "Tủ sách của cha"
}

public class Sach
{
    public int Id { get; set; }
    public string Title { get; set; }
    public int TacGiaId { get; set; }     // "Con thuộc cha nào?" (số ID cha)
    public TacGia TacGia { get; set; }    // "Cha tôi đây" (neo tham chiếu)
}
```

## 6.2. Many-to-Many (Nhiều ↔ Nhiều)

1 **Sinh viên** học nhiều **Môn**. 1 **Môn** có nhiều **Sinh viên**:

```csharp
public class SinhVien
{
    public int Id { get; set; }
    public ICollection<MonHoc> MonHocs { get; set; } // "Các môn tôi học"
}

public class MonHoc
{
    public int Id { get; set; }
    public ICollection<SinhVien> SinhViens { get; set; } // "Các trò trong lớp"
}
// EF Core 5+ tự tạo bảng trung gian ẩn — bạn không cần viết thêm!
```

## 6.3. Fluent API vs Data Annotations

| | Data Annotations | Fluent API |
|---|---|---|
| **Ẩn dụ** | Viết ghi chú bên lề bản vẽ | Viết tài liệu kỹ thuật riêng |
| **Khai báo** | `[Required]` trên Property | `modelBuilder.Entity<>()` |
| **Composite Key** | ❌ Không được | ✅ `HasKey(e => new { e.A, e.B })` |
| **Ưu tiên** | Thấp | **Cao** (ghi đè Annotations) |

---

# 7. LINQ — Thần chú truy vấn

Thay vì viết SQL, bạn dùng C# thuần → EF Core tự dịch:

```csharp
// Lọc — "Cho tôi sách giá trên 500K"
var dat = await db.Sachs.Where(s => s.Gia > 500000).ToListAsync();

// Chọn cột — "Chỉ lấy tên thôi, không cần hết" (tiết kiệm băng thông)
var ten = await db.Sachs.Select(s => s.Title).ToListAsync();

// Sắp xếp — "Sách mới nhất lên trên"
var moi = await db.Sachs.OrderByDescending(s => s.NgayXuat).ToListAsync();

// Lấy 1 — "Cho tôi sách số 5"
var sach = await db.Sachs.FirstOrDefaultAsync(s => s.Id == 5);
// FirstOrDefault → null nếu không có (an toàn)
// First → THROW EXCEPTION nếu không có (nguy hiểm!)

// Include — "Lấy tác giả KÈM tất cả sách của ổng" (1 query JOIN)
var tg = await db.TacGias.Include(a => a.DsSach).FirstOrDefaultAsync(a => a.Id == 1);
```

---

# 8. Repository & Unit of Work

## 8.1. Repository — Nhân viên kho chuyên nghiệp

Viết `db.Sachs.Add()` trực tiếp trong Controller = **Tiếp tân tự vào kho lấy đồ** → rối, đổi kho thì phải sửa mọi nơi.

**Repository** = thuê 1 nhân viên kho riêng. Tiếp tân chỉ nói "Lấy cho tôi sách ID 5" → nhân viên kho chạy đi lấy.

```csharp
// Hợp đồng (Interface)
public interface IRepository<T> where T : class
{
    Task<IEnumerable<T>> GetAllAsync();
    Task<T?> GetByIdAsync(int id);
    Task AddAsync(T entity);
}

// Nhân viên kho thật (Implementation)
public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _db;
    public Repository(AppDbContext db) { _db = db; }

    public async Task<IEnumerable<T>> GetAllAsync()
        => await _db.Set<T>().ToListAsync();
    // ...
}

// Đăng ký shipper (DI)
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
```

## 8.2. Unit of Work — Gom thành 1 giao dịch

Chuyển tiền: Trừ TK A, cộng TK B. Trừ xong mà cộng LỖI → phải **HỦY CẢ HAI**. Unit of Work = "hoặc cả 2 thành công, hoặc không cái nào":

```csharp
public class UnitOfWork(AppDbContext db) : IDisposable
{
    public IRepository<Sach> Sachs { get; } = new Repository<Sach>(db);
    public IRepository<TacGia> TacGias { get; } = new Repository<TacGia>(db);

    public async Task SaveAsync() => await db.SaveChangesAsync(); // Commit CÙNG LÚC
    public void Dispose() => db.Dispose();
}
```

---

# 9. Tối ưu hiệu suất

## 9.1. Loading Strategies — Mời bạn bè đến nhà

| Cách mời | Ẩn dụ | Khi nào dùng |
|---|---|---|
| **Eager Loading** ✅ | "Đón cả gia đình tới luôn" (`Include()`) | Biết cần data quan hệ |
| **Lazy Loading** ⚠️ | "Ai tới thì tới" — tự động load khi truy cập | Tiện nhưng **N+1 Problem** |
| **Explicit Loading** | "Gọi riêng từng người" | Load thêm sau |

### N+1 Problem — Cái bẫy kinh điển

```csharp
var authors = db.TacGias.ToList();       // 1 query lấy tác giả
foreach (var a in authors)
    Console.WriteLine(a.DsSach.Count);   // MỖI ông → 1 query riêng!
// 100 tác giả → 101 queries → DB NGHẸT THỞ!

// FIX: Đón cả gia đình 1 lần
var authors = db.TacGias.Include(a => a.DsSach).ToList(); // 1 query JOIN DUY NHẤT
```

## 9.2. AsNoTracking — Nhẹ 90% RAM

Bình thường EF Core **theo dõi** mọi object (ghi nhớ thay đổi). Nếu chỉ đọc (hiển thị) → theo dõi = lãng phí!

```csharp
// CHỈ ĐỌC → tắt theo dõi
var list = await db.Products.AsNoTracking().ToListAsync(); // Nhanh gấp đôi!
```

## 9.3. Select chỉ cột cần — Đừng khuân cả tủ

```csharp
// ❌ Lấy 50 cột mà chỉ dùng 2
var all = await db.Products.ToListAsync();

// ✅ Lấy đúng 2 cột cần
var names = await db.Products.Select(p => new { p.Id, p.Name }).ToListAsync();
```

---

# 10. Seeding & Encryption

## 10.1. Data Seeding — Hàng mẫu trưng bày

Khi mở quán mới, cần vài món mẫu sẵn trong menu:

```csharp
protected override void OnModelCreating(ModelBuilder mb)
{
    mb.Entity<TheLoai>().HasData(
        new TheLoai { Id = 1, Name = "Kinh dị" },
        new TheLoai { Id = 2, Name = "Lãng mạn" }
    );
}
// Chạy dotnet ef database update → hàng mẫu tự chèn
```

## 10.2. Value Converter — Két sắt tự động

Dữ liệu nhạy cảm (CCCD, thẻ tín dụng) lưu DB phải **mã hóa**:

```csharp
mb.Entity<User>()
    .Property(u => u.CCCD)
    .HasConversion(
        v => Encrypt(v),   // Bỏ vào két → mã hóa
        v => Decrypt(v)    // Lấy từ két → giải mã
    );
// Developer viết "sach.CCCD = 123456" → DB lưu "xYz@#!" → Đọc ra lại 123456
```

---

# 11. Pagination — Lật trang sách

```csharp
int trang = 1, soLuong = 10;

var ketQua = await db.Products
    .Where(p => p.Name.Contains("Harry"))    // Lọc
    .OrderBy(p => p.Price)                    // Sắp xếp
    .Skip((trang - 1) * soLuong)              // Bỏ qua trang trước
    .Take(soLuong)                            // Lấy đúng 10 sản phẩm
    .AsNoTracking()                           // Chỉ đọc → nhẹ
    .ToListAsync();

var tong = await db.Products.CountAsync(p => p.Name.Contains("Harry")); // Đếm tổng
// Gửi về: { Items: [...], Total: 35, Page: 1, PageSize: 10 }
```

---

> 🎯 **Module tiếp theo**: [Module 06 — Authentication & Authorization](Module-06-Authentication-Authorization.md)
