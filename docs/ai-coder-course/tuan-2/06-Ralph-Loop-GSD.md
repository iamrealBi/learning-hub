# 🔄 Bài 6: Ralph Loop & GSD Workflow

> Tự động hóa coding: set the loop and walk away

---

## 1. Ralph Loop là gì?

**Ralph Loop** (Ralph Wiggum Loop) là pattern tự động hóa coding đơn giản nhưng cực kỳ hiệu quả, phổ biến bởi Geoffrey Huntley:

### Triết lý:
> Thay vì orchestrator phức tạp, dùng **bash loop đơn giản** + **AI agent** + **fresh context mỗi iteration**.

```
┌──────────────────────────────────────┐
│           RALPH LOOP                 │
│                                      │
│  while tasks_remain:                 │
│    1. Đọc task list (progress.txt)   │
│    2. Spawn AI agent (fresh context) │
│    3. Agent pick task → implement    │
│    4. Agent chạy tests/lint          │
│    5. Nếu pass → commit, mark done   │
│    6. Nếu fail → fix hoặc skip      │
│    7. Agent exit                     │
│    8. Loop lại từ bước 1             │
│  done                                │
└──────────────────────────────────────┘
```

### Ưu điểm quan trọng: **Fresh Context**
- Mỗi iteration = context **mới hoàn toàn**
- Không bị "context bloat" như long session
- Không cần `/compact` — vì luôn bắt đầu lại
- **Convergence pressure**: Tests/lint buộc code phải đúng

---

## 2. Anatomy: Cấu Trúc Ralph Loop

### File specification (`prd.md`):
```markdown
# BookWormHub Enhancement PRD

## Tasks
1. [ ] Thêm phân trang cho BookService (10 items/page)
2. [ ] Thêm sắp xếp (title, author, rating, year)  
3. [ ] Tạo HomeService với dashboard stats
4. [ ] Export CSV cho danh sách sách
5. [ ] Thêm caching cho GetBookListAsync
6. [ ] Viết integration tests cho Controllers
```

### Progress tracker (`progress.txt`):
```
Task 1: DONE (commit abc123)
Task 2: DONE (commit def456)
Task 3: IN_PROGRESS
Task 4: TODO
Task 5: TODO
Task 6: TODO
```

### Runner script (`ralph.sh`):
```bash
#!/bin/bash

PRD="prd.md"
PROGRESS="progress.txt"

while grep -q "TODO\|IN_PROGRESS" "$PROGRESS"; do
    echo "========================================="
    echo "Starting new Ralph iteration..."
    echo "========================================="
    
    # Spawn Claude Code with fresh context
    claude --print \
        "Read $PRD and $PROGRESS. 
         Pick the first TODO task, mark it IN_PROGRESS.
         Implement it following AGENTS.md rules.
         Run 'dotnet build' and 'dotnet test'.
         If all pass: commit with descriptive message, mark DONE.
         If fail: try to fix up to 3 times, then mark FAILED.
         Update $PROGRESS and exit."
    
    echo "Iteration complete. Checking progress..."
    cat "$PROGRESS"
    echo ""
    
    # Safety: sleep 5 seconds between iterations
    sleep 5
done

echo "✅ All tasks completed!"
```

### Chạy:
```bash
chmod +x ralph.sh
./ralph.sh
# Rồi đi uống cafe ☕
```

---

## 3. GSD (Get Stuff Done) Workflow

### GSD là phiên bản "pro" của Ralph Loop:

```
GSD = Ralph Loop + Guard Rails + Human Checkpoints

┌──────────────────────────────────────────┐
│              GSD WORKFLOW                │
│                                          │
│  Plan ──→ Implement ──→ Verify ──→ Ship │
│   │           │            │         │   │
│   ▼           ▼            ▼         ▼   │
│  Human      Agent        Agent     Human │
│  Review     Works        Tests     PR    │
│  Plan       Code         Pass      Review│
└──────────────────────────────────────────┘
```

### PIV (Plan-Implement-Verify):
1. **Plan**: AI lên kế hoạch → Human review & approve
2. **Implement**: AI code dựa trên plan đã duyệt
3. **Verify**: AI chạy tests, lint → Human review PR

---

## 4. Ví dụ BookWormHub: GSD Workflow

```bash
# Step 1: Plan
claude "/plan Tôi cần thêm tính năng phân trang cho books.
Requirements:

- 10 items per page
- Giữ search/filter hoạt động
- Pagination UI ở dưới
- Unit tests

Hãy liệt kê files cần thay đổi và approach."

# Step 2: Review plan → Approve

# Step 3: Implement
claude "Thực hiện plan đã thống nhất ở trên.
Sau mỗi file, chạy dotnet build.
Khi xong tất cả, chạy dotnet test."

# Step 4: Verify
dotnet test
git diff  # Review changes
git add . && git commit -m "feat: add pagination to book listing"
```

---

## 5. Guardrails Cho Production

| Guardrail | Mô tả |
|-----------|--------|
| **Branch Protection** | AI push lên feature branch, human review PR |
| **Test Gate** | AI phải pass ALL tests trước khi mark DONE |
| **Lint Gate** | Code phải pass formatting rules |
| **Sandbox** | Chạy trong Docker container |
| **Max Iterations** | Giới hạn số lần retry (tránh loop vô hạn) |

---

## 6. Khi Nào Dùng Ralph Loop

| Tình huống | Ralph Loop? |
|-----------|------------|
| Migration code (porting) | ✅ Rất tốt |
| Repetitive refactoring | ✅ Tốt |
| Adding tests to existing code | ✅ Tốt |
| Greenfield với spec rõ ràng | ✅ Tốt |
| Complex architecture decisions | ❌ Cần human |
| Ambiguous requirements | ❌ Cần clarification |

---

> **Tiếp theo**: [Bài 7: Jira & GitHub Automation →](07-Jira-GitHub-Automation.md)
