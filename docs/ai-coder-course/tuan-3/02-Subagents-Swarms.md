# 🐝 Bài 2: Subagents, Swarms & Agent Teams

> Chi tiết cách tổ chức và điều phối nhiều AI agents

---

## 1. Subagents Trong Claude Code

### Cách sử dụng:
```bash
claude "
Tôi cần refactor error handling trong BookWormHub.
Hãy sử dụng subagents cho:
- Subagent 1: Refactor BookService error handling
- Subagent 2: Refactor ReviewService error handling  
- Subagent 3: Refactor AdminService error handling
- Subagent 4: Update tất cả tests

Mỗi subagent làm việc độc lập, không sửa file của nhau.
Tổng hợp kết quả khi tất cả hoàn thành.
"
```

### Kết quả:
```
Main Agent: Phân tích và phân công...

[Subagent 1] Starting: BookService refactor
[Subagent 2] Starting: ReviewService refactor
[Subagent 3] Starting: AdminService refactor
[Subagent 4] Starting: Test updates

[Subagent 1] ✅ Done: 3 methods updated
[Subagent 3] ✅ Done: 4 methods updated
[Subagent 2] ✅ Done: 2 methods updated
[Subagent 4] ✅ Done: 12 tests updated, all pass

Main Agent: Tổng hợp: 9 methods refactored, 12 tests updated.
All tests passing. ✅
```

---

## 2. Swarms

### Swarm Pattern:
Nhiều agents **cùng loại** làm **cùng kiểu task** trên **các phần khác nhau** của codebase.

```
Swarm: "Thêm XML documentation cho tất cả public methods"

Agent 1 → BookService.cs (7 methods)
Agent 2 → ReviewService.cs (5 methods)
Agent 3 → AdminService.cs (6 methods)
Agent 4 → BadgeService.cs (1 method)
Agent 5 → ModerationService.cs (1 method)
Agent 6 → HomeService.cs (2 methods)

All agents: Cùng skill, cùng rules, khác file
```

### Khi nào dùng Swarm:
- Cùng một loại thay đổi, nhiều files
- Documentation generation
- Code migration/porting
- Adding tests to existing code
- Bulk refactoring

---

## 3. Agent Teams Trong Thực Tế

### Shared Task List:
```json
// tasks.json
{
  "tasks": [
    {
      "id": 1,
      "title": "Create Category model",
      "status": "done",
      "assignee": "agent-backend",
      "dependencies": []
    },
    {
      "id": 2,
      "title": "Create CategoryService",
      "status": "in_progress",
      "assignee": "agent-backend",
      "dependencies": [1]
    },
    {
      "id": 3,
      "title": "Create Category views",
      "status": "blocked",
      "assignee": "agent-frontend",
      "dependencies": [2]
    },
    {
      "id": 4,
      "title": "Write Category tests",
      "status": "in_progress",
      "assignee": "agent-testing",
      "dependencies": [1, 2]
    }
  ]
}
```

### Coordination Protocol:
```
Agent Backend:  "Task 2 done, CategoryService created."
Agent Frontend: "Unblocked! Starting task 3."
Agent Testing:  "Task 4 done. Found issue: 
                 CategoryService.Delete doesn't cascade."
Agent Backend:  "Fixing cascade delete... Done."
Agent Testing:  "Re-running tests... All pass ✅"
```

---

## 4. Công Cụ Orchestration

| Tool | Pattern | Mô tả |
|------|---------|--------|
| **Claude Code Subagents** | Hierarchical | Built-in, dễ dùng |
| **tmux + multiple claudes** | Manual teams | Flexible, visual |
| **Git Worktrees** | Isolation | Mỗi agent 1 copy riêng |
| **LangGraph** | Framework | Python, complex flows |
| **CrewAI** | Framework | Python, role-based |

### Git Worktrees cho Multi-Agent:
```bash
# Tạo worktree cho mỗi agent
git worktree add ../agent-backend feature/backend
git worktree add ../agent-frontend feature/frontend
git worktree add ../agent-testing feature/testing

# Mỗi agent chạy trong worktree riêng
cd ../agent-backend && claude "Implement backend..."
cd ../agent-frontend && claude "Implement frontend..."
cd ../agent-testing && claude "Write tests..."

# Merge khi xong
git merge feature/backend feature/frontend feature/testing
```

---

## 5. Anti-Patterns

| ❌ Đừng | ✅ Nên |
|---------|--------|
| Quá nhiều agents cho task nhỏ | 2-5 agents là optimal |
| Agents sửa cùng file | Mỗi agent = riêng files |
| Không có shared state | Dùng task list + git |
| Fire-and-forget | Monitor & intervene |

---

> **Tiếp theo**: [Bài 3: Claude Agent SDK →](03-Claude-Agent-SDK.md)
