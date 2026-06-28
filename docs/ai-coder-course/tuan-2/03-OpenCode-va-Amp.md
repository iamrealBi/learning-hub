# 🔓 Bài 3: OpenCode & Amp — CLI Agents Mã Nguồn Mở

---

## Phần A: OpenCode

### 1. OpenCode là gì?
OpenCode là CLI agent **mã nguồn mở**, **đa nhà cung cấp model** — bạn không bị khóa vào một vendor.

### 2. Cài đặt
```bash
# Linux/macOS
curl -fsSL https://opencode.ai/install | bash

# Node.js
npm install -g opencode-ai

# Windows (Chocolatey)
choco install opencode-ai
```

### 3. Cấu hình
```bash
# Thiết lập API key
export OPENAI_API_KEY="your-key"
# hoặc
export ANTHROPIC_API_KEY="your-key"

# Chạy trong project
cd BookWormHub/BookWormHub
opencode
```

### 4. Hai chế độ

| Chế độ | Mô tả | Phím tắt |
|--------|--------|----------|
| **Plan Mode** | Chỉ đọc, lên kế hoạch | Mặc định |
| **Build Mode** | Sửa file, chạy lệnh | `Tab` để chuyển |

### 5. Điểm mạnh
- ✅ **Free & Open Source** (MIT License)
- ✅ **75+ providers** (OpenAI, Anthropic, Gemini, Ollama, local models)
- ✅ **LSP Support** — code intelligence
- ✅ **MCP integration**
- ✅ **Privacy** — dữ liệu lưu local

### 6. Ví dụ sử dụng
```bash
# Interactive
opencode

# One-shot (CI/CD friendly)
opencode run "Thêm logging cho tất cả service methods trong BookWormHub"
```

---

## Phần B: Sourcegraph Amp

### 1. Amp là gì?
Amp là agent của **Sourcegraph** — nổi tiếng với khả năng **hiểu codebase quy mô lớn** nhờ code graph technology.

### 2. Điểm mạnh đặc biệt

| Tính năng | Mô tả |
|-----------|--------|
| **Deep Context** | Hiểu dependencies, symbols xuyên suốt organization |
| **Oracle Mode** | Reasoning sâu với o3-mini cho phân tích nhân quả |
| **Thread Sharing** | Chia sẻ threads giữa team members |
| **Enterprise** | Command allowlisting, audit logging |

### 3. Workflow
```
1. Intent Parsing: Bạn mô tả mục tiêu
   ↓
2. Planning: Amp tạo step-by-step plan → Review
   ↓
3. Execution: Amp code, build, test, iterate
   ↓
4. Review/Handoff: Xem kết quả, approve changes
```

### 4. AGENT.md cho Amp
```markdown
# AGENT.md

## Project: BookWormHub
- Stack: ASP.NET Core 8 + EF Core + SQLite
- Pattern: Service Layer + FluentValidation
- Testing: xUnit + FluentAssertions

## Rules
- Follow existing service patterns
- Always create matching interface
- Run dotnet test after changes
```

### 5. Amp vs Cody (cùng Sourcegraph)
- **Cody** = Assistant (chat, explain, autocomplete)
- **Amp** = Agent (plan, execute, test, iterate)

---

## So Sánh OpenCode vs Amp vs Claude Code

| Tiêu chí | OpenCode | Amp | Claude Code |
|----------|----------|-----|-------------|
| **Giá** | Free | Enterprise | API/Pro |
| **Open Source** | ✅ | ❌ | ❌ |
| **Model flexibility** | ✅ 75+ | ⚠️ Limited | ⚠️ Claude only |
| **Deep reasoning** | Tùy model | ✅ Oracle | ✅ Native |
| **Subagents** | ❌ | ❌ | ✅ |
| **Hooks/Skills** | ❌ | ❌ | ✅ |
| **Enterprise** | ❌ | ✅ | ⚠️ |
| **Best for** | Cá nhân, prototype | Enterprise, large codebases | Deep work, complex tasks |

---

> **Tiếp theo**: [Bài 4: MCP Protocol →](04-MCP-Protocol.md)
