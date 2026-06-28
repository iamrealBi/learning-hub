# 🎯 Bài 3: Context Engineering — Nghệ Thuật Cung Cấp Ngữ Cảnh Cho AI

> *"Thành công không nằm ở một prompt hoàn hảo, mà ở việc xây dựng hệ thống cung cấp thông tin đúng, đủ, đúng lúc cho AI"*

---

## 1. Context Engineering là gì?

**Context Engineering** (Kỹ thuật ngữ cảnh) là bước tiến hóa tiếp theo của Prompt Engineering. Thay vì tập trung vào việc viết một prompt hoàn hảo, Context Engineering tập trung vào việc **thiết kế toàn bộ hệ sinh thái thông tin** mà AI agent cần để hoạt động hiệu quả.

### So sánh:

```
Prompt Engineering (Cũ)          Context Engineering (Mới)
━━━━━━━━━━━━━━━━━━━━             ━━━━━━━━━━━━━━━━━━━━━━━
- Viết 1 prompt tốt             - Thiết kế hệ thống context
- Tập trung 1 lần hỏi           - Tự động hóa quy trình
- Thủ công, lặp lại             - File cấu hình bền vững
- Phụ thuộc kỹ năng cá nhân     - Chuẩn hóa cho cả team
```

### Nguyên tắc "Goldilocks"
> Cung cấp tập **nhỏ nhất** các token có **chất lượng cao nhất** để tối đa hóa hiệu suất.

- ❌ Quá ít context → AI đoán mò, hallucinate
- ❌ Quá nhiều context → AI bối rối, chi phí cao, "lost in the middle"
- ✅ Vừa đủ context → AI chính xác, nhanh, tiết kiệm

---

## 2. Các File Context Quan Trọng

### 2.1 AGENTS.md — Tiêu chuẩn đa công cụ

**AGENTS.md** là file "README dành cho AI" — chứa hướng dẫn toàn dự án, hoạt động trên nhiều công cụ khác nhau (Cursor, Claude Code, Copilot, Antigravity).

**Vị trí**: Đặt ở **root** của dự án

```markdown
# AGENTS.md

## Project Overview
BookWormHub - Ứng dụng quản lý và đánh giá sách
ASP.NET Core MVC + EF Core + SQLite
FluentValidation cho validation

## Tech Stack
- .NET 8
- ASP.NET Core MVC
- Entity Framework Core (SQLite)
- FluentValidation
- xUnit + FluentAssertions (Testing)

## Architecture Pattern
- **Service Layer Pattern**: Mọi business logic nằm trong Services/
- **POCO Models**: Models/ chứa entities thuần túy
- **ViewModel Pattern**: ViewModels/ cho data binding
- **Interface-based DI**: Services/Interfaces/ cho dependency injection

## Build & Run Commands
```bash
dotnet build
dotnet run --project BookWormHub
dotnet test BookWormHub.Tests
```

## Code Standards
- Sử dụng tiếng Việt cho validation messages
- Service methods trả về ServiceResult hoặc ServiceResult<T>
- Mỗi service implement interface tương ứng
- FluentValidation cho mọi input validation
- Tests sử dụng InMemory database

## File Structure
```
BookWormHub/
├── Models/          # POCO entities
├── Services/        # Business logic
│   └── Interfaces/  # Service contracts
├── Validators/      # FluentValidation
├── ViewModels/      # View data binding
├── Controllers/     # MVC controllers
├── Data/           # DbContext
└── Views/          # Razor views
```

## IMPORTANT RULES
- KHÔNG sửa Models/ trừ khi được yêu cầu rõ ràng
- KHÔNG xóa hoặc sửa tests hiện có
- LUÔN tạo interface cho service mới
- LUÔN viết unit test cho logic mới
```

### 2.2 CLAUDE.md — Dành riêng cho Claude Code

**CLAUDE.md** chứa hướng dẫn đặc biệt cho Claude/Claude Code, được đọc **tự động** mỗi khi bắt đầu phiên.

```markdown
# CLAUDE.md

## Memory
- Dự án BookWormHub sử dụng Service Layer Pattern
- Database: SQLite với EF Core
- Tất cả validation dùng FluentValidation
- Test framework: xUnit + FluentAssertions + InMemory DB

## Preferences
- Viết code C# hiện đại (.NET 8 features)
- Sử dụng var khi type rõ ràng
- Async/await cho mọi database operations
- Namespace dạng file-scoped

## Workflow
1. Đọc AGENTS.md trước khi bắt đầu
2. Kiểm tra tests hiện có trước khi sửa code
3. Chạy `dotnet test` sau mỗi thay đổi
4. Commit với conventional commit messages
```

### 2.3 .cursorrules — Dành riêng cho Cursor IDE

Cursor sử dụng thư mục `.cursor/rules/` để quản lý rules theo ngữ cảnh:

```
.cursor/
└── rules/
    ├── general.mdc          # Rules luôn áp dụng
    ├── csharp-coding.mdc    # Rules cho file .cs
    └── testing.mdc          # Rules cho thư mục Tests/
```

**Ví dụ `general.mdc`:**
```markdown
---
description: General coding rules
alwaysApply: true
---

- Use C# 12 / .NET 8 features
- Follow Service Layer Pattern
- Always use FluentValidation for input validation
- Write Vietnamese messages for user-facing strings
- Use async/await for all database operations
```

**Ví dụ `testing.mdc`:**
```markdown
---
description: Testing rules
globs: ["**/*.Tests/**/*.cs", "**/*Tests.cs"]
---

- Use xUnit test framework
- Use FluentAssertions for assertions
- Use InMemory database for data access tests
- Each test method should follow: Arrange → Act → Assert
- Name tests: MethodName_Condition_ExpectedResult
```

### 2.4 .github/copilot-instructions.md — Cho GitHub Copilot

```markdown
# Copilot Instructions

This is a C# ASP.NET Core MVC application.

## Coding Style
- Use file-scoped namespaces
- Prefer var when the type is obvious
- Use string interpolation over concatenation
- Follow existing patterns in Services/ folder

## Architecture
- Services handle business logic (never in Controllers)
- All services must implement an interface
- Use FluentValidation, NOT DataAnnotations
- Return ServiceResult from service methods
```

---

## 3. Phân Cấp File Context

### Cấu trúc phân cấp:

```
Root/
├── AGENTS.md              ← Global (tất cả tools đọc)
├── CLAUDE.md              ← Claude Code đọc
├── .cursor/rules/         ← Cursor đọc
├── .github/
│   └── copilot-instructions.md  ← GitHub Copilot đọc
│
├── src/
│   ├── AGENTS.md          ← Context cho thư mục src/
│   ├── api/
│   │   └── AGENTS.md      ← Context riêng cho api/
│   └── tests/
│       └── AGENTS.md      ← Context riêng cho tests/
```

### Nguyên tắc:
- **Global rules** → File ở root
- **Area-specific rules** → File trong thư mục con
- **Tool sẽ tự động merge** rules từ root + thư mục hiện tại

---

## 4. Best Practices

### ✅ NÊN làm:

1. **Cập nhật incremental**: Thêm rule mới khi AI sai lần thứ 3 về cùng vấn đề
2. **Giữ ngắn gọn**: Dùng bullet points, headings rõ ràng
3. **Bao gồm lệnh build/test**: AI cần biết cách verify code
4. **Liệt kê KHÔNG làm**: Guardrails rõ ràng
5. **Mô tả kiến trúc**: Tổng quan cấu trúc dự án

### ❌ KHÔNG nên:

1. ~~Viết paragraphs dài dòng~~ → Dùng bullet points
2. ~~Cố tạo file hoàn hảo từ đầu~~ → Iterate dần
3. ~~Copy-paste toàn bộ code~~ → Chỉ mô tả patterns
4. ~~Quên cập nhật khi dự án thay đổi~~ → Treat as living document

---

## 5. Prompt Engineering Nâng Cao

### Template cho prompt hiệu quả:

```
[MỤC TIÊU]: Mô tả rõ bạn muốn gì
[NGỮ CẢNH]: Tình trạng hiện tại, hành vi hiện tại
[RÀNG BUỘC]: Giới hạn, rules phải tuân thủ
[TIÊU CHÍ HOÀN THÀNH]: Thế nào là "xong"
```

### Ví dụ thực tế với BookWormHub:

```
[MỤC TIÊU]: Thêm tính năng phân trang cho danh sách sách

[NGỮ CẢNH]: 
- Hiện tại GetBookListAsync() trả về TẤT CẢ sách
- BookIndexViewModel chứa List<Book> Books
- Trang index hiển thị toàn bộ sách, chưa có phân trang

[RÀNG BUỘC]:
- Tuân thủ Service Layer Pattern hiện có
- Dùng ServiceResult pattern
- Mỗi trang hiển thị 10 sách
- Giữ nguyên tính năng search và filter genre

[TIÊU CHÍ HOÀN THÀNH]:
- BookService có phân trang hoạt động
- ViewModel có PageNumber, TotalPages
- View có nút Previous/Next
- Unit tests cho phân trang
- Tất cả tests hiện có vẫn pass
```

### Meta-Prompting

Trước khi code, hãy yêu cầu AI **lên kế hoạch** trước:

```
Trước khi viết code, hãy:
1. Liệt kê các file cần sửa/tạo mới
2. Mô tả approach bạn sẽ dùng
3. Xác nhận có vi phạm bất kỳ rule nào trong AGENTS.md không
4. Liệt kê các edge cases cần test
```

---

## 6. Session Hygiene — Vệ sinh phiên làm việc

### Nguyên tắc:

| Tình huống | Hành động |
|-----------|-----------|
| Đổi task hoàn toàn | `/clear` hoặc mở chat mới |
| Phiên quá dài (>50 exchanges) | `/compact` để tóm tắt lịch sử |
| Context bị "ô nhiễm" | Reset session, bắt đầu lại |
| Nhiều task khác nhau | Mỗi task = 1 session riêng |

### Specialist Agents

Cho dự án phức tạp, chia AI thành các "chuyên gia":

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Code Agent  │  │  Test Agent  │  │  Docs Agent  │
│              │  │              │  │              │
│ - Viết code  │  │ - Viết tests │  │ - Viết docs  │
│ - Refactor   │  │ - Chạy tests │  │ - Comments   │
│ - Debug      │  │ - Coverage   │  │ - README     │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## 7. Ví dụ Thực Tế: Context Engineering cho BookWormHub

### Tạo bộ context files hoàn chỉnh:

```
BookWormHub/
├── AGENTS.md              ← Đã tạo ở mục 2.1
├── CLAUDE.md              ← Đã tạo ở mục 2.2
├── .cursor/
│   └── rules/
│       ├── general.mdc    ← Đã tạo ở mục 2.3
│       └── testing.mdc    ← Đã tạo ở mục 2.3
├── .github/
│   └── copilot-instructions.md  ← Đã tạo ở mục 2.4
└── BookWormHub/
    └── Services/
        └── AGENTS.md      ← Context riêng cho Services
```

**Services/AGENTS.md:**
```markdown
# Service Layer Guidelines

- Mỗi service class implement interface trong Interfaces/
- Inject dependencies qua constructor
- Trả về ServiceResult cho operations
- Sử dụng AppDbContext cho database access
- Async cho tất cả database operations
- Không throw exceptions, dùng ServiceResult.Fail()
```

---

> **Tiếp theo**: [Bài 4: Cursor AI IDE →](04-Cursor-AI-IDE.md)
