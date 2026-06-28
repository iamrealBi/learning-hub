# 🏗️ Bài 3: Claude Agent SDK — Xây Dựng Agent Bằng Code

> Từ sử dụng agent → xây dựng agent: embed agentic loops vào ứng dụng của bạn

---

## 1. Claude Agent SDK là gì?

**Claude Agent SDK** (`claude-agent-sdk`) là thư viện Python chính thức cho phép nhúng engine của Claude Code vào ứng dụng của bạn — biến ứng dụng thành agent tự hành.

### So sánh:
| | Anthropic SDK | Agent SDK |
|---|---|---|
| **Mục đích** | API calls đơn lẻ | Agent loops tự động |
| **Tool calling** | Bạn quản lý loop | SDK quản lý tự động |
| **Complexity** | Thấp | Trung bình |
| **Use case** | Chatbot, Q&A | Autonomous agents |

---

## 2. Cài Đặt & Quick Start

```bash
pip install claude-agent-sdk
```

### Ví dụ cơ bản:
```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    response = await query(
        prompt="Phân tích thư mục hiện tại và liệt kê tất cả file .cs",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Bash", "Glob"]
        )
    )
    print(response)

asyncio.run(main())
```

---

## 3. Core Concepts

### Agent Loop:
```
Input → [Plan] → [Tool Call] → [Observe] → [Plan] → ... → Output
         ↑                                    │
         └────────── Loop until done ──────────┘
```

### Tools có sẵn:
| Tool | Mục đích |
|------|----------|
| `Read` | Đọc file |
| `Write` | Ghi file |
| `Bash` | Chạy shell commands |
| `Glob` | Tìm files theo pattern |
| `Grep` | Tìm text trong files |

---

## 4. Ví Dụ: BookWormHub Agent

### Agent phân tích code quality:
```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def analyze_code_quality():
    response = await query(
        prompt="""
        Phân tích chất lượng code trong BookWormHub/Services/:
        
        1. Kiểm tra mỗi service có interface không
        2. Kiểm tra error handling (dùng ServiceResult?)
        3. Kiểm tra async patterns
        4. Liệt kê methods thiếu unit tests
        5. Đề xuất improvements
        
        Output dạng markdown report.
        """,
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Glob", "Grep"],
            max_turns=20
        )
    )
    
    # Lưu report
    with open("quality_report.md", "w") as f:
        f.write(response)
    
    print("✅ Report saved to quality_report.md")

asyncio.run(analyze_code_quality())
```

### Agent tự viết tests:
```python
async def auto_generate_tests():
    response = await query(
        prompt="""
        Scan BookWormHub/Services/ và BookWormHub.Tests/Services/.
        
        1. Tìm service methods chưa có test
        2. Viết unit tests cho mỗi method thiếu
        3. Follow pattern từ BookServiceTests.cs:
           - Sử dụng TestDbContextFactory
           - Sử dụng FluentAssertions
           - Naming: MethodName_Condition_Expected
        4. Chạy dotnet test để verify
        """,
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Write", "Bash", "Glob", "Grep"],
            max_turns=50
        )
    )
    print(response)

asyncio.run(auto_generate_tests())
```

---

## 5. Hooks Trong SDK

```python
from claude_agent_sdk import ClaudeAgentOptions, Hook

options = ClaudeAgentOptions(
    allowed_tools=["Read", "Write", "Bash"],
    hooks=[
        Hook(
            event="PreToolUse",
            tool="Write",
            handler=lambda ctx: print(f"⚠️ Writing to: {ctx.file_path}")
        ),
        Hook(
            event="PostToolUse",
            tool="Bash",
            handler=lambda ctx: print(f"📋 Command output: {ctx.output[:200]}")
        )
    ]
)
```

---

## 6. Orchestration Patterns

### Pattern 1: Reflection
```python
# Agent kiểm tra output của chính nó
response = await query(
    prompt="""
    1. Viết hàm CalculateAverageRating cho BookService
    2. SAU KHI viết xong, review lại code của bạn:
       - Có handle edge cases không?
       - Có null checks không?
       - Có async đúng cách không?
    3. Sửa nếu cần
    """
)
```

### Pattern 2: Planning
```python
# AI lên kế hoạch trước
plan = await query(
    prompt="Lên kế hoạch chi tiết để thêm caching cho BookService",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob"])
)

# Rồi thực thi plan
result = await query(
    prompt=f"Thực hiện plan sau:\n{plan}",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Write", "Bash"])
)
```

### Pattern 3: Multi-Agent
```python
# Orchestrator chia task
tasks = [
    ("Sửa BookService", ["Read", "Write"]),
    ("Viết tests", ["Read", "Write", "Bash"]),
    ("Update docs", ["Read", "Write"]),
]

results = await asyncio.gather(*[
    query(prompt=task, options=ClaudeAgentOptions(allowed_tools=tools))
    for task, tools in tasks
])
```

---

## 7. Best Practices

1. **Scope hẹp** — Mỗi agent 1 task cụ thể
2. **Permission tối thiểu** — `allowed_tools` chỉ những gì cần
3. **Max turns** — Giới hạn iterations để tránh infinite loop
4. **Hooks cho logging** — Monitor mọi action
5. **Error recovery** — Xử lý khi agent fail

---

> **Tiếp theo**: [Bài 4: Frontier Tech →](04-Frontier-Tech.md)
