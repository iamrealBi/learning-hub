# 🔧 Bài 5: Plugins, Hooks, Skills & Subagents

> Bốn lớp mở rộng biến Claude Code từ chatbot thành nền tảng phát triển

---

## 1. Tổng Quan Delegation Layer

```
┌─────────────────────────────────────────────┐
│            CLAUDE CODE ENGINE               │
│                                             │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌───────────┐  │
│  │Skills│ │Hooks │ │MCPs  │ │Subagents  │  │
│  │      │ │      │ │      │ │           │  │
│  │Reuse │ │Enforce│ │Extend│ │Parallelize│  │
│  └──────┘ └──────┘ └──────┘ └───────────┘  │
└─────────────────────────────────────────────┘
```

| Lớp | Mục đích | Bắt buộc? |
|-----|----------|-----------|
| **Skills** | Workflow tái sử dụng | ❌ Advisory |
| **Hooks** | Quy tắc lifecycle | ✅ Mandatory |
| **MCP** | Kết nối tools ngoài | — |
| **Subagents** | Song song hóa tasks | — |

---

## 2. Skills — Chi Tiết

### Tạo skill cho BookWormHub:

**`.claude/skills/crud-feature.md`:**
```markdown
# CRUD Feature Generator

## Input Required
- EntityName: Tên entity (VD: Category)
- Properties: Danh sách properties

## Steps
1. Tạo Model trong Models/{EntityName}.cs
2. Tạo ViewModel trong ViewModels/{EntityName}ViewModel.cs
3. Tạo Validator bằng FluentValidation
4. Tạo Interface I{EntityName}Service.cs
5. Tạo Service {EntityName}Service.cs
6. Đăng ký DI trong Program.cs
7. Tạo Controller {EntityName}Controller.cs
8. Tạo Views (Index, Create, Edit, Details)
9. Thêm migration: dotnet ef migrations add Add{EntityName}
10. Tạo Tests {EntityName}ServiceTests.cs
11. Chạy: dotnet build && dotnet test
```

### Gọi skill:
```bash
/skills crud-feature EntityName=Category Properties="Id,Name,Description"
```

---

## 3. Hooks — Chi Tiết

### Các lifecycle events:

| Event | Khi nào | Ví dụ sử dụng |
|-------|---------|---------------|
| `PreToolUse` | Trước khi AI dùng tool | Block file nhạy cảm |
| `PostToolUse` | Sau khi AI dùng tool | Auto-format, lint |
| `Notification` | Khi có sự kiện | Alert khi sửa config |

### Ví dụ hooks thực tế cho BookWormHub:

```json
{
  "hooks": [
    {
      "event": "PostToolUse",
      "tool": "Write",
      "matcher": "*.cs",
      "command": "dotnet format --include ${file} --severity warn",
      "description": "Auto-format sau mỗi lần sửa C#"
    },
    {
      "event": "PreToolUse",
      "tool": "Write", 
      "matcher": "appsettings*.json",
      "command": "echo '⚠️ CẢNH BÁO: Đang sửa config file!' && exit 1",
      "description": "Chặn sửa config không có permission"
    },
    {
      "event": "PostToolUse",
      "tool": "Bash",
      "matcher": "dotnet test",
      "command": "echo '✅ Tests completed'",
      "description": "Thông báo khi tests chạy xong"
    }
  ]
}
```

---

## 4. Subagents — Song Song Hóa

### Khi nào dùng subagents:

```
Task lớn: "Refactor toàn bộ error handling trong BookWormHub"

Không subagent:                   Có subagent:
━━━━━━━━━━━━━━                   ━━━━━━━━━━━━
Sửa BookService                  Sub1: BookService    ┐
  → xong                         Sub2: ReviewService  ├─ Song song!
Sửa ReviewService                Sub3: AdminService   ┘
  → xong                         
Sửa AdminService                 Tổng kết → Merge
  → xong                         
                                  
Thời gian: 3x                    Thời gian: 1x + overhead
```

---

> **Tiếp theo**: [Bài 6: Ralph Loop & GSD →](06-Ralph-Loop-GSD.md)
