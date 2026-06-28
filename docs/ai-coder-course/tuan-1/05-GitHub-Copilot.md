# 🤖 Bài 5: GitHub Copilot — Agent Mode & Multi-file Editing

> GitHub Copilot đã tiến hóa từ autocomplete đơn giản thành một autonomous coding agent

---

## 1. Tổng Quan GitHub Copilot

GitHub Copilot là AI coding assistant của GitHub (Microsoft), tích hợp trực tiếp vào VS Code. Từ 2025, Copilot đã thêm **Agent Mode** và **Edit Mode** — biến nó từ trợ lý gợi ý thành agent tự hành.

---

## 2. Ba Chế Độ Hoạt Động

### Bảng so sánh:

| Chế độ | Phím tắt | Mục đích | Mức tự trị |
|--------|----------|----------|------------|
| **Ask Mode** | Chat panel | Hỏi đáp, giải thích | Không (chỉ Q&A) |
| **Edit Mode** | Chat → Edit | Sửa nhiều file có kiểm soát | Trung bình |
| **Agent Mode** | Chat → Agent | Tự động end-to-end | Cao |

### 2.1 Ask Mode
Hỏi đáp thông thường, giải thích code:
```
Bạn: Giải thích pattern ServiceResult trong dự án này

Copilot: ServiceResult là một wrapper pattern dùng để...
```

### 2.2 Edit Mode
Đề xuất thay đổi nhiều file, bạn review diff trước khi apply:
```
Bạn: Đổi tên ReviewStatus.Hidden thành ReviewStatus.Pending 
     và cập nhật tất cả references

Copilot: Đề xuất thay đổi 5 files:
  📝 Review.cs           - Đổi enum value
  📝 ReviewService.cs    - 3 references
  📝 AdminService.cs     - 2 references  
  📝 ModerationService.cs - 1 reference
  📝 ReviewServiceTests.cs - 4 references
```

### 2.3 Agent Mode
AI tự lên kế hoạch, thực thi, chạy tests:
```
Bạn: Thêm export CSV cho danh sách sách, bao gồm 
     Title, Author, Genre, Rating trung bình

Agent Mode tự:
1. ✅ Tạo CsvExportService.cs
2. ✅ Tạo ICsvExportService.cs 
3. ✅ Cập nhật BookController thêm action Export
4. ✅ Thêm nút Export trên View
5. ✅ Chạy dotnet build → OK
6. ✅ Viết CsvExportServiceTests.cs
7. ✅ Chạy dotnet test → All pass
```

---

## 3. Cấu Hình copilot-instructions.md

Tạo file `.github/copilot-instructions.md` trong root project:

```markdown
# GitHub Copilot Instructions for BookWormHub

## Project Context
- ASP.NET Core 8 MVC application
- Entity Framework Core with SQLite
- FluentValidation for input validation
- Service Layer Pattern architecture

## Coding Standards
- File-scoped namespaces
- Async methods end with "Async"  
- All service methods return ServiceResult
- Use Vietnamese for user-facing messages
- Use FluentAssertions in tests

## Architecture Rules
- Business logic ONLY in Services/ folder
- Controllers must be thin (route + call service)
- Models are POCOs (no business logic)
- Every service implements an interface

## Testing Standards
- Use xUnit + FluentAssertions
- InMemory database for data tests
- Test naming: MethodName_Condition_Expected
- Always test both success and failure paths
```

---

## 4. Reference Context

### Sử dụng `#` để chỉ định context:

```
#file:BookService.cs Hãy thêm method SearchByAuthorAsync 
theo pattern giống GetBookListAsync

#workspace Tìm tất cả chỗ sử dụng BadgeService
```

---

## 5. MCP Integration

GitHub Copilot Agent Mode hỗ trợ **Model Context Protocol** để kết nối với tools bên ngoài:

```
┌──────────────┐     MCP      ┌─────────────┐
│  Copilot     │◄────────────►│  Database    │
│  Agent Mode  │              │  Server      │
│              │     MCP      ├─────────────┤
│              │◄────────────►│  GitHub API  │
│              │              │  Server      │
│              │     MCP      ├─────────────┤
│              │◄────────────►│  Jira        │
│              │              │  Server      │
└──────────────┘              └─────────────┘
```

---

## 6. Tips Sử Dụng Hiệu Quả

1. **Dùng Edit Mode** cho refactoring có kiểm soát
2. **Dùng Agent Mode** cho feature mới phức tạp
3. **Luôn review diff** — đừng auto-accept mọi thứ
4. **Tạo copilot-instructions.md** ngay khi bắt đầu dự án
5. **Kéo file vào chat** để cung cấp context chính xác

---

> **Tiếp theo**: [Bài 6: OpenAI Codex →](06-OpenAI-Codex.md)
