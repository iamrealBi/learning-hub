# 📅 TUẦN 05: Function Calling, Tool Use & ReAct Pattern

> **Mục tiêu**: Build AI agent có thể gọi tools, hiểu ReAct pattern  
> **Thời lượng**: 20-25 giờ | **Output**: 🏆 Research Agent (Portfolio Project #2)

---

## 🎯 Sau Tuần Này Bạn Sẽ:
- [ ] Implement OpenAI function calling
- [ ] Implement Claude tool_use
- [ ] Build ReAct agent from scratch (không dùng framework)
- [ ] Xử lý errors, retries, fallback cho tool calls
- [ ] Agent có memory (conversation history)
- [ ] 🏆 Research Agent tự tìm kiếm web

---

## 📚 Khóa Học — Đánh Giá Chất Lượng

### 🔴 Khóa chính — FREE (Tốt nhất cho topic này):

**1. [DeepLearning.AI: Functions, Tools and Agents with LangChain](https://www.deeplearning.ai/short-courses/functions-tools-agents-langchain/)** 🥇
- ⭐⭐⭐⭐⭐ | FREE | ~2 giờ | Andrew Ng + Harrison Chase
- Tại sao #1: Dạy bởi LangChain creator, chuẩn pattern mới nhất

**2. [Anthropic Docs: Tool Use Guide](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)** 🥇
- ⭐⭐⭐⭐⭐ | FREE | Official reference
- Claude tool_use spec chính xác nhất, có code examples

**3. [OpenAI: Function Calling Guide](https://platform.openai.com/docs/guides/function-calling)** 🥇
- ⭐⭐⭐⭐⭐ | FREE | Official reference

### 🟡 Bổ trợ — Coursera (lấy chứng chỉ):
**[AI Agent Developer Specialization (Vanderbilt)](https://www.coursera.org/specializations/ai-agent-developer)**
- Khóa 1: "AI Agents and Agentic AI with Python" — relevant cho tuần này
- Có certificate LinkedIn-ready

### 🟢 Tham khảo — Udemy Business (xem thêm project examples):
**[AI Engineer Agentic Track (Ed Donner)](https://ibmcsr.udemy.com/course/ai-engineer-agentic-track/)**
- ⭐ 4.7 | 17h — Sections về tool calling, agent architecture
- Dùng xem thêm SAU KHI nắm vững từ free sources

---

## 📅 Lịch Học Theo Ngày

### Thứ 2 — OpenAI Function Calling (3.5 giờ)
```python
from openai import OpenAI

client = OpenAI()

# Define tools
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["city"]
            }
        }
    }
]

# Let the model decide when to call tools
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "What's the weather in Saigon?"}],
    tools=tools,
    tool_choice="auto"
)

# Handle tool call
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    # Execute the function and send result back
```

### Thứ 3 — Claude Tool Use (3.5 giờ)
```python
import anthropic

client = anthropic.Anthropic()

tools = [
    {
        "name": "search_web",
        "description": "Search the web for information",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"}
            },
            "required": ["query"]
        }
    }
]

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "Find latest AI news in Vietnam"}]
)

# Handle tool_use blocks
for block in response.content:
    if block.type == "tool_use":
        tool_name = block.name
        tool_input = block.input
        # Execute tool and send result back
```

### Thứ 4 — ReAct Pattern From Scratch (4 giờ)
```python
class ReActAgent:
    """Reason-Act-Observe loop - NO framework, pure Python"""
    
    def __init__(self, tools: dict, llm_client):
        self.tools = tools  # {"tool_name": callable}
        self.llm = llm_client
        self.max_iterations = 5
    
    def run(self, query: str) -> str:
        thoughts = []
        for i in range(self.max_iterations):
            # REASON: Ask LLM what to do next
            prompt = self._build_prompt(query, thoughts)
            response = self.llm.think(prompt)
            
            if response.action == "FINAL_ANSWER":
                return response.answer
            
            # ACT: Execute the chosen tool
            try:
                result = self.tools[response.tool](response.tool_input)
            except Exception as e:
                result = f"Error: {e}"
            
            # OBSERVE: Record what happened
            thoughts.append({
                "thought": response.reasoning,
                "action": response.tool,
                "observation": result
            })
        
        return "Max iterations reached"
```

### Thứ 5 — Error Handling & Memory (3.5 giờ)
```python
# Retry decorator for tool calls
import time

def retry_tool(max_retries=3, backoff=1.0):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    time.sleep(backoff * (2 ** attempt))
        return wrapper
    return decorator

# Conversation memory
class ConversationMemory:
    def __init__(self, max_messages: int = 20):
        self.messages: list[dict] = []
        self.max_messages = max_messages
    
    def add(self, role: str, content: str):
        self.messages.append({"role": role, "content": content})
        if len(self.messages) > self.max_messages:
            # Keep system + summarize old messages
            self._summarize_old()
```

### Thứ 6 — Custom Tools (3.5 giờ)
```python
# Build real tools
import httpx, datetime

@retry_tool(max_retries=3)
def web_search(query: str) -> str:
    """Search web using SerpAPI or Tavily"""
    pass

def read_file(filepath: str) -> str:
    """Read local file content"""
    pass

def calculate(expression: str) -> str:
    """Safely evaluate math expressions"""
    pass

def get_current_time(timezone: str = "Asia/Ho_Chi_Minh") -> str:
    """Get current time in specified timezone"""
    pass
```

### Thứ 7 — 🏆 PROJECT 2: Research Agent (4 giờ)
```
research_agent/
├── main.py           # CLI interface
├── agent.py          # ReAct agent core
├── tools/
│   ├── web_search.py # Search tool
│   ├── scraper.py    # Web page reader
│   └── writer.py     # Markdown report writer
├── memory.py         # Conversation + research memory
├── config.py
├── requirements.txt
└── README.md
```

**Demo:**
```bash
$ python main.py "Nghiên cứu thị trường AI Agent tại Việt Nam 2026"

🔍 Thinking: Tôi cần tìm kiếm thông tin về thị trường AI agent VN...
🛠️ Action: web_search("AI agent market Vietnam 2026")
👀 Observation: Found 5 relevant results...
🔍 Thinking: Cần thêm data về lương...
🛠️ Action: web_search("AI engineer salary Vietnam 2026")
👀 Observation: ...
📝 Action: write_report(findings)

✅ Report saved: output/research-ai-vietnam.md
```

---

## ✅ Đánh Giá Cuối Tuần

| # | Tiêu chí | ✅/❌ |
|---|---------|------|
| 1 | Implement OpenAI function calling (ít nhất 3 tools) | |
| 2 | Implement Claude tool_use (ít nhất 3 tools) | |
| 3 | Build ReAct loop WITHOUT any framework | |
| 4 | Agent tự quyết định khi nào gọi tool nào | |
| 5 | Error handling: retry, fallback khi tool fails | |
| 6 | Conversation memory hoạt động | |
| 7 | Giải thích được ReAct pattern (Reason-Act-Observe) | |
| 8 | Research Agent tạo được báo cáo markdown | |
| 9 | Code trên GitHub với README + demo | |
| 10 | So sánh được OpenAI vs Claude tool calling | |

---

> **Tiếp theo**: [Tuần 06: LangChain Core →](Tuan-06-LangChain-Core.md)
