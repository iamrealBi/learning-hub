# 🔗 Bài 7: Tự Động Hóa Jira & GitHub

> AI agents tự pick task, code, push, và update ticket — không cần bạn can thiệp

---

## 1. Tổng Quan

```
┌──────┐     Pick Issue      ┌──────────┐     Push Code     ┌──────┐
│ Jira │ ←─────────────────→ │ AI Agent │ ──────────────── → │GitHub│
│      │     Update Status   │          │     Create PR      │      │
└──────┘                     └──────────┘                    └──────┘
```

---

## 2. GitHub Integration

### GitHub MCP Server:
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxx"
      }
    }
  }
}
```

### Workflow tự động:
```bash
claude "
1. Đọc GitHub Issues cho repo BookWormHub
2. Pick issue có label 'ai-ready' đầu tiên
3. Tạo branch feature/issue-{number}
4. Implement theo description trong issue
5. Chạy tests
6. Commit & push
7. Tạo Pull Request với description chi tiết
8. Comment trên issue: 'PR created: #xx'
"
```

---

## 3. Jira Integration

### Jira MCP Server:
```json
{
  "mcpServers": {
    "jira": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-jira"],
      "env": {
        "JIRA_URL": "https://your-org.atlassian.net",
        "JIRA_EMAIL": "your@email.com",
        "JIRA_API_TOKEN": "xxx"
      }
    }
  }
}
```

### Workflow:
```bash
claude "
1. Lấy tickets từ Jira sprint hiện tại, status = 'To Do'
2. Pick ticket đầu tiên, chuyển sang 'In Progress'
3. Đọc acceptance criteria trong ticket
4. Implement theo criteria
5. Chạy tests
6. Commit với message: 'BOOK-{number}: {summary}'
7. Chuyển ticket sang 'In Review'
8. Comment trên ticket với link commit
"
```

---

## 4. GitHub Actions + AI Agent

### `.github/workflows/ai-review.yml`:
```yaml
name: AI Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Get PR diff
        run: |
          git diff origin/main...HEAD > pr_diff.txt
          
      - name: AI Review
        run: |
          claude --print "
            Review the following PR diff for BookWormHub.
            Check for:
            1. Service Layer Pattern compliance
            2. FluentValidation usage
            3. Missing unit tests
            4. ServiceResult pattern adherence
            5. Vietnamese message consistency
            
            Diff:
            $(cat pr_diff.txt)
            
            Output review comments in markdown.
          " > review.md
          
      - name: Post review comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review.md', 'utf8');
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: review
            });
```

---

## 5. Ứng Dụng Cho BookWormHub (SEAP)

### Đề xuất cho team:
1. **Label system**: Tạo label `ai-ready` cho issues AI có thể tự xử lý
2. **PR template**: Yêu cầu mô tả rõ để AI review dễ hơn
3. **Branch naming**: `feature/BOOK-xxx-description`
4. **AI reviewer**: GitHub Action tự review mọi PR

---

> **Tiếp theo**: [Bài 8: Dự án Kanban →](08-Du-An-Kanban.md)
