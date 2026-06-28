# 🔮 Bài 2: Claude Code — Deep Dive

> Claude Code là CLI agent mạnh nhất cho deep reasoning và multi-step engineering

---

## 1. Cài Đặt & Bắt Đầu

```bash
# Cài đặt global
npm install -g @anthropic-ai/claude-code

# Bắt đầu phiên
cd BookWormHub/BookWormHub
claude

# Đăng nhập lần đầu
/login
```

---

## 2. Slash Commands Quan Trọng

### Quản lý phiên:
| Command | Mục đích |
|---------|----------|
| `/clear` | Xóa toàn bộ lịch sử, context mới |
| `/compact` | Nén lịch sử, giữ context quan trọng |
| `/resume` hoặc `claude -c` | Tiếp tục phiên gần nhất |
| `/rename` | Đặt tên cho phiên hiện tại |
| `/cost` | Xem chi phí token đã dùng |
| `/model` | Chuyển đổi model AI |

### Workflow:
| Command | Mục đích |
|---------|----------|
| `/plan` | Chế độ lên kế hoạch (chỉ đọc) |
| `/review` | Review code changes |
| `/bug` | Báo cáo bug, yêu cầu debug |
| `/init` | Tạo CLAUDE.md cho dự án |

### Mở rộng:
| Command | Mục đích |
|---------|----------|
| `/skills` | Quản lý skills (macro workflows) |
| `/hooks` | Cấu hình hooks (lifecycle events) |
| `/mcp` | Quản lý MCP servers |
| `/permissions` | Cấu hình quyền |

---

## 3. CLAUDE.md — "Hiến pháp" của Agent

File quan trọng nhất — Claude đọc **tự động** mỗi phiên:

```markdown
# CLAUDE.md — BookWormHub

## Project Overview
BookWormHub là ứng dụng web quản lý và đánh giá sách.
ASP.NET Core 8 MVC + EF Core (SQLite) + FluentValidation

## Build Commands
- Build: `dotnet build`
- Run: `dotnet run --project BookWormHub`
- Test: `dotnet test BookWormHub.Tests`

## Architecture
- Service Layer Pattern
- POCO Models (no business logic in models)
- FluentValidation for input validation
- Interface-based Dependency Injection
- ServiceResult pattern for return values

## Code Standards
- File-scoped namespaces
- Async/await for all DB operations
- Vietnamese messages for user-facing strings
- Naming: MethodNameAsync for async methods

## IMPORTANT
- DO NOT modify existing Models without permission
- DO NOT delete existing tests
- ALWAYS create interfaces for new services
- ALWAYS write tests for new business logic
- RUN `dotnet test` after every change
```

---

## 4. Checkpoints — Mạng An Toàn

### Checkpoint là gì?
Claude Code **tự động lưu trạng thái** trước mỗi thay đổi lớn. Nếu AI sai, bạn có thể "quay ngược thời gian":

```bash
# Quay lại checkpoint trước
/rewind

# Hoặc nhấn Esc 2 lần nhanh
Esc Esc → Quay về trạng thái trước
```

### Workflow an toàn:
```
Code gốc ──→ [Checkpoint 1] ──→ AI sửa file A
                                     │
                                     ↓
                              [Checkpoint 2] ──→ AI sửa file B
                                                      │
                                                      ↓ Lỗi!
                              /rewind ──→ Quay về Checkpoint 2
```

---

## 5. Plan Mode — Lên Kế Hoạch Trước

### Tại sao cần Plan Mode?
Buộc Claude **phải suy nghĩ** trước khi code, tránh sai lầm tốn kém.

```bash
/plan

> Tôi muốn thêm tính năng export báo cáo PDF cho admin.
> Báo cáo bao gồm: thống kê sách, reviews, users.
> Hãy lên kế hoạch chi tiết.

Claude output:
═══════════════════════════════════════
📋 KẾ HOẠCH IMPLEMENT
═══════════════════════════════════════

1. Dependencies cần thêm:
   - QuestPDF (NuGet package cho PDF generation)

2. Files cần tạo mới:
   - Services/IReportService.cs (interface)
   - Services/ReportService.cs (implementation)
   - ViewModels/ReportViewModel.cs
   - Tests/ReportServiceTests.cs

3. Files cần sửa:
   - Controllers/AdminController.cs (thêm action)
   - Views/Admin/Index.cshtml (thêm nút export)
   - Program.cs (đăng ký DI)

4. Approach:
   - Query data từ AppDbContext
   - Map sang ReportViewModel
   - Render PDF bằng QuestPDF
   - Return FileResult từ controller

5. Tests:
   - ReportService tạo PDF có data
   - ReportService xử lý khi không có data
   - Integration: Admin action trả về PDF
═══════════════════════════════════════

> Bạn có đồng ý plan này không? (y/n)
```

---

## 6. Skills — Workflow Có Thể Tái Sử Dụng

### Skills là gì?
File Markdown đóng vai trò "macro" — workflow lặp lại bạn có thể gọi bất kỳ lúc nào.

### Tạo skill:

**`.claude/skills/add-service.md`:**
```markdown
# Skill: Add New Service

## Steps
1. Tạo interface trong Services/Interfaces/I{ServiceName}.cs
2. Tạo implementation trong Services/{ServiceName}.cs
3. Implement constructor injection cho AppDbContext
4. Đăng ký trong Program.cs: builder.Services.AddScoped<I{ServiceName}, {ServiceName}>()
5. Tạo unit tests trong Tests/Services/{ServiceName}Tests.cs
6. Chạy dotnet build && dotnet test

## Template Interface
```csharp
namespace BookWormHub.Services.Interfaces;

public interface I{ServiceName}
{
    // Define methods here
}
```

## Checklist
- [ ] Interface created
- [ ] Implementation created
- [ ] DI registered
- [ ] Tests written
- [ ] All tests pass
```

### Sử dụng skill:
```bash
/skills add-service
> ServiceName = StatisticsService
> Methods: GetDashboardStatsAsync()
```

---

## 7. Hooks — Quy Tắc Bắt Buộc

### Hooks vs Prompts:
- **Prompts** = Advisory (AI có thể bỏ qua)
- **Hooks** = Mandatory (PHẢI thực hiện)

### Lifecycle Events:

```
PreToolUse ──→ [AI sử dụng tool] ──→ PostToolUse
                                         │
                                         ↓
                                    Notification
```

### Ví dụ Hook: Auto-format trước khi commit

```json
// .claude/hooks.json
{
  "hooks": [
    {
      "event": "PreToolUse",
      "tool": "Write",
      "command": "dotnet format --include ${file}",
      "description": "Auto-format C# file trước khi ghi"
    },
    {
      "event": "PostToolUse", 
      "tool": "Write",
      "matcher": "*.cs",
      "command": "dotnet build --no-restore -q",
      "description": "Build check sau khi sửa file C#"
    }
  ]
}
```

---

## 8. Subagents — Phân Công Công Việc

### Subagent là gì?
**Claude con** được spawn từ Claude cha, xử lý subtask trong **context riêng biệt**, không "ô nhiễm" context chính.

```
┌──────────────────────┐
│   Claude Chính       │
│   "Refactor project" │
│                      │
│   Spawn subagents:   │
│   ┌────────────┐     │
│   │ Sub 1:     │     │
│   │ Sửa models │     │
│   └────────────┘     │
│   ┌────────────┐     │
│   │ Sub 2:     │     │
│   │ Sửa tests  │     │
│   └────────────┘     │
│   ┌────────────┐     │
│   │ Sub 3:     │     │
│   │ Update docs│     │
│   └────────────┘     │
│                      │
│   Thu thập kết quả   │
│   → Tổng hợp         │
└──────────────────────┘
```

### Ưu điểm:
- Context chính **không bị bloat**
- Subagent chạy **độc lập**, tập trung
- Có thể **chạy song song** (multi-agent preview)

---

## 9. Permissions — Kiểm Soát Quyền

```bash
/permissions

# Pre-approve safe actions
allow: dotnet build
allow: dotnet test
allow: dotnet format

# Block dangerous actions  
deny: rm -rf
deny: git push --force
deny: dotnet publish

# Ask for confirmation
ask: git commit
ask: dotnet add package
```

---

## 10. Best Practices

1. **Tạo CLAUDE.md ngay lập tức** — `/init` khi bắt đầu
2. **Dùng `/plan` cho task lớn** — nghĩ trước, code sau
3. **`/compact` thường xuyên** — giữ context sạch
4. **Checkpoint = bạn thân** — `/rewind` khi AI sai
5. **Skills cho task lặp** — automation = tiết kiệm thời gian
6. **Hooks cho quality gates** — enforce, không chỉ suggest

---

> **Tiếp theo**: [Bài 3: OpenCode & Amp →](03-OpenCode-va-Amp.md)
