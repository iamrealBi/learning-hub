# 🖥️ Bài 4: Cursor AI IDE — Hướng Dẫn Toàn Diện

> Cursor là IDE tích hợp AI mạnh nhất hiện nay, biến VS Code thành một Agentic Development Environment

---

## 1. Cursor AI là gì?

**Cursor** là một IDE (Integrated Development Environment) được xây dựng trên nền VS Code, tích hợp AI trực tiếp vào quy trình phát triển. Thay vì chỉ gợi ý code như autocomplete truyền thống, Cursor có thể **tự đọc toàn bộ dự án**, **sửa nhiều file cùng lúc**, và **chạy lệnh terminal** một cách tự động.

### Tại sao Cursor?
- 🧠 **Hiểu toàn bộ codebase** — không chỉ file đang mở
- 📁 **Multi-file editing** — sửa nhiều file trong 1 prompt
- 🤖 **Agent Mode** — AI tự đưa ra quyết định và thực thi
- 🔧 **Familiar UI** — Giống VS Code, dễ chuyển đổi

---

## 2. Các Tính Năng Chính

### Bảng tổng hợp:

| Tính năng | Phím tắt | Mục đích |
|-----------|----------|----------|
| **Tab Autocomplete** | `Tab` | Gợi ý code dòng tiếp theo |
| **Inline Edit** | `Ctrl + K` | Sửa nhanh trong 1 file |
| **AI Chat** | `Ctrl + L` | Hỏi đáp, giải thích code |
| **Composer** | `Ctrl + I` | Tạo/sửa nhiều file cùng lúc |
| **Agent Mode** | Chọn trong Composer | AI tự hành động end-to-end |

---

### 2.1 Tab Autocomplete

Tính năng cơ bản nhất — AI dự đoán dòng code tiếp theo:

```csharp
// Bạn gõ:
public async Task<ServiceResult> Create

// AI gợi ý (nhấn Tab để chấp nhận):
public async Task<ServiceResult> CreateBookAsync(BookCreateViewModel model)
{
    var book = new Book
    {
        Title = model.Title,
        Author = model.Author,
        // ...
    };
    _db.Books.Add(book);
    await _db.SaveChangesAsync();
    return ServiceResult.Ok();
}
```

### 2.2 Inline Edit (Ctrl + K)

Chọn đoạn code → `Ctrl + K` → Mô tả thay đổi:

```
Chọn hàm GetBookListAsync()
→ Ctrl + K
→ "Thêm phân trang, mỗi trang 10 items"
→ AI sửa trực tiếp trong file
```

### 2.3 AI Chat (Ctrl + L)

Mở panel chat bên phải, hỏi đáp về code:

```
Bạn: Giải thích hàm IsValidISBN13 hoạt động như thế nào?

AI: Hàm IsValidISBN13 kiểm tra tính hợp lệ của mã ISBN-13 
bằng thuật toán checksum:

1. Kiểm tra độ dài phải đúng 13 ký tự và toàn bộ là số
2. Tính tổng: các vị trí chẵn (0,2,4...) nhân 1, 
   vị trí lẻ (1,3,5...) nhân 3

3. Nếu tổng chia hết cho 10 → ISBN hợp lệ
```

### 2.4 Composer (Ctrl + I) — Sức mạnh chính

Composer cho phép AI **tạo hoặc sửa nhiều file cùng lúc**:

```
Prompt trong Composer:
"Tạo HomeService với tính năng hiển thị thống kê:

 - Tổng số sách
 - Tổng số reviews
 - Rating trung bình toàn hệ thống
 - Sách mới nhất (5 cuốn)
 Tạo interface, service, viewmodel, và unit tests"

Kết quả: Cursor tạo/sửa 4-5 files đồng thời:
 ✅ IHomeService.cs (interface)
 ✅ HomeService.cs (implementation)
 ✅ HomeViewModel.cs (viewmodel)
 ✅ HomeServiceTests.cs (tests)
 ✅ HomeController.cs (cập nhật)
```

### 2.5 Agent Mode — Chế độ tự trị

Agent Mode biến AI từ **trợ lý bị động** thành **agent chủ động**:

```
┌─────────────────────────────────────┐
│           AGENT MODE                │
│                                     │
│  1. 🔍 Tự tìm files liên quan      │
│  2. 📝 Tự đề xuất thay đổi         │
│  3. 💻 Tự chạy terminal commands    │
│  4. 🔄 Tự sửa lỗi khi build fail   │
│  5. ✅ Lặp lại cho đến khi thành    │
│        công                         │
└─────────────────────────────────────┘
```

**Ví dụ với BookWormHub:**
```
Prompt: "Thêm tính năng sắp xếp sách theo rating trung bình, 
năm xuất bản, và số lượng reviews. Cập nhật cả UI và tests."

Agent Mode sẽ tự:

1. Đọc BookService.cs, BookIndexViewModel.cs, Index.cshtml
2. Thêm parameter sortBy vào GetBookListAsync
3. Cập nhật ViewModel với SortBy options
4. Sửa View thêm dropdown sắp xếp
5. Chạy dotnet build → nếu lỗi → tự sửa
6. Thêm unit tests cho sorting
7. Chạy dotnet test → verify pass
```

---

## 3. Reference Context với @

### Sử dụng `@` để chỉ định context:

| Cú pháp | Mục đích | Ví dụ |
|---------|----------|-------|
| `@file` | Tham chiếu file cụ thể | `@BookService.cs` |
| `@folder` | Tham chiếu thư mục | `@Services/` |
| `@codebase` | Toàn bộ dự án | `@codebase tìm mọi nơi dùng AppDbContext` |
| `@docs` | Tài liệu URL | `@docs https://fluentvalidation.net` |
| `@web` | Tìm kiếm web | `@web EF Core pagination best practices` |
| `@git` | Lịch sử Git | `@git thay đổi gần nhất ở BookService` |

### Ví dụ kết hợp:
```
Dựa trên @BookService.cs và @ReviewService.cs, hãy tạo 
BadgeService theo cùng pattern, implement @IBadgeService.cs
```

---

## 4. Cấu hình .cursorrules

### Tạo file `.cursor/rules/`:

**`.cursor/rules/csharp.mdc`:**
```yaml
---
description: C# coding standards for BookWormHub
globs: ["**/*.cs"]
alwaysApply: true
---

## C# Standards
- Use .NET 8 / C# 12 features
- File-scoped namespaces
- Use `var` when type is obvious from context
- Prefer string interpolation: $"Hello {name}"
- Use null-conditional operators: ?.
- Async methods must end with "Async" suffix

## Architecture
- Services: contain ALL business logic
- Controllers: only route + call services
- Models: POCO only, no logic
- ViewModels: data binding, no logic

## Patterns
- Return ServiceResult for operations
- Use FluentValidation for all input
- Inject via constructor (no property injection)
- Use IAsyncEnumerable where appropriate
```

---

## 5. Workflow Thực Tế với Cursor

### Quy trình phát triển feature mới:

```
Bước 1: Mô tả yêu cầu trong Composer (Agent Mode)
   ↓
Bước 2: AI đề xuất plan → Bạn review
   ↓
Bước 3: AI implement → Bạn xem diff
   ↓
Bước 4: Accept/Reject từng thay đổi
   ↓
Bước 5: AI chạy tests → Fix nếu fail
   ↓
Bước 6: Commit & Push
```

### Tips pro:
1. **Luôn review diff** trước khi Accept
2. **Dùng `@` references** để AI có đủ context
3. **Chia nhỏ task** — đừng yêu cầu quá nhiều trong 1 prompt
4. **Cài .cursorrules** ngay từ đầu dự án
5. **Agent Mode** cho task phức tạp, **Inline Edit** cho sửa nhanh

---

## 6. So sánh Cursor vs VS Code + Copilot

| Tiêu chí | Cursor | VS Code + Copilot |
|----------|--------|-------------------|
| **Multi-file editing** | ✅ Composer (mạnh) | ⚠️ Mới có (Agent Mode) |
| **Agent Mode** | ✅ Mature, ổn định | ✅ Mới, đang phát triển |
| **Context awareness** | ✅ @codebase, @docs | ⚠️ #file (hạn chế hơn) |
| **Terminal integration** | ✅ Tự chạy commands | ✅ Tương tự |
| **Custom rules** | ✅ .cursorrules | ✅ copilot-instructions.md |
| **Giá** | $20/tháng (Pro) | $10/tháng (Copilot) |
| **Model flexibility** | ✅ Nhiều model | ⚠️ Chủ yếu GPT |

---

> **Tiếp theo**: [Bài 5: GitHub Copilot →](05-GitHub-Copilot.md)
