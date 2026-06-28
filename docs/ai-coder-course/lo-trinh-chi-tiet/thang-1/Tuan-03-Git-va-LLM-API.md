# 📅 TUẦN 03: Git, GitHub & LLM API

> **Mục tiêu**: Git workflow chuyên nghiệp + gọi OpenAI/Claude API bằng Python  
> **Thời lượng**: 20-25 giờ | **Output**: CLI Chatbot với conversation memory

---

## 🎯 Sau Tuần Này Bạn Sẽ:
- [ ] Git: init, add, commit, branch, merge, rebase, PR
- [ ] GitHub: remote, push, pull, fork, PR workflow
- [ ] Hiểu LLM: tokens, context window, temperature
- [ ] Prompt Engineering: system/user/assistant roles
- [ ] Gọi OpenAI Chat Completions API
- [ ] Gọi Anthropic Claude Messages API
- [ ] Build CLI Chatbot với conversation history

---

## 📚 Khóa Học

### 🔴 Khóa chính — Udemy Business:
**[Git & GitHub - The Practical Guide](https://ibmcsr.udemy.com/course/git-github-practical-guide/)**
- Giảng viên: Academind (Maximilian Schwarzmüller) | ⭐ 4.6 | 10.5 giờ
- **Tuần này học**: Sections 1-8 (Core Git + GitHub)

### 🟡 Coursera — Lý thuyết sâu + Chứng chỉ 🎓:
**[Generative AI with Large Language Models](https://www.coursera.org/learn/generative-ai-with-llms)** (DeepLearning.AI + AWS)
- 🥇 **PRIMARY cho LLM theory** — Andrew Ng, sâu nhất về transformer/inference
- **Tuần này**: Week 1 — LLM fundamentals, tokens, inference
- 🎓 Certificate LinkedIn-ready

**[Prompt Engineering for ChatGPT](https://www.coursera.org/learn/prompt-engineering)** (Vanderbilt University)
- 🥈 SUPPLEMENT — Prompt patterns academic-level
- **Tuần này**: Modules 1-3

### 🆓 Free:
- [DeepLearning.AI: ChatGPT Prompt Engineering for Devs](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/) — 1 giờ, FREE, xuất sắc
- [Anthropic Prompt Engineering Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/) — Official, luôn cập nhật
- [OpenAI Cookbook](https://cookbook.openai.com/) — Code examples

---

## 📅 Lịch Học Theo Ngày

### Thứ 2 — Git Basics (3.5 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.5h | 🎬 Video: Git & GitHub Practical Guide — Sections 1-3 |
| 1.0h | 💻 Practice: init, add, commit, log, diff, status |
| 0.5h | 💻 Exercise: Tạo repo cho tất cả bài tập tuần trước |
| 0.5h | 📝 Git cheat sheet |

### Thứ 3 — Git Branching & GitHub (3.5 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.5h | 🎬 Video: Sections 4-6 — Branching, Merging, Remote |
| 1.0h | 💻 Practice: branch, checkout, merge, resolve conflicts |
| 0.5h | 💻 Exercise: Fork → Branch → PR workflow |
| 0.5h | 📝 Setup GitHub profile: bio, avatar, pinned repos |

### Thứ 4 — LLM Fundamentals (3.5 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.5h | 🎬 Coursera: GenAI with LLMs — Week 1 |
| 1.0h | 🆓 DeepLearning.AI: Prompt Engineering short course |
| 0.5h | 📝 Notes: Tokens, context window, temperature, top_p |
| 0.5h | 💻 Exercise: Đếm tokens với tiktoken library |

**Bài tập:**
```python
import tiktoken

def count_tokens(text: str, model: str = "gpt-4") -> int:
    enc = tiktoken.encoding_for_model(model)
    return len(enc.encode(text))

# Test: đếm tokens cho prompts khác nhau
print(count_tokens("Hello, how are you?"))
print(count_tokens("Viết một bài thơ về Sài Gòn"))  # Tiếng Việt tốn nhiều tokens hơn!
```

### Thứ 5 — OpenAI API (3.5 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.0h | 📖 OpenAI API documentation |
| 1.5h | 💻 Practice: Chat completions, streaming, parameters |
| 0.5h | 💻 Exercise: System prompt engineering |
| 0.5h | 💻 Build: Simple Q&A bot |

**Bài tập:**
```python
from openai import OpenAI

client = OpenAI()  # Uses OPENAI_API_KEY env var

def chat(user_message: str, system_prompt: str = "You are a helpful assistant.") -> str:
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_message}
        ],
        temperature=0.7,
        max_tokens=500
    )
    return response.choices[0].message.content

# Test với các system prompts khác nhau
print(chat("Explain Python decorators", system_prompt="You are a Python expert. Explain in Vietnamese."))
```

### Thứ 6 — Anthropic Claude API (3.5 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.0h | 📖 Anthropic API documentation |
| 1.5h | 💻 Practice: Claude messages API, streaming |
| 0.5h | 💻 Compare: OpenAI vs Claude output quality |
| 0.5h | 💻 Build: Multi-model chat selector |

**Bài tập:**
```python
import anthropic

client = anthropic.Anthropic()  # Uses ANTHROPIC_API_KEY

def chat_claude(user_message: str, system: str = "You are helpful.") -> str:
    message = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=500,
        system=system,
        messages=[{"role": "user", "content": user_message}]
    )
    return message.content[0].text

# Multi-model wrapper
class AIChat:
    def __init__(self, provider: str = "openai"):
        self.provider = provider
    
    def chat(self, message: str) -> str:
        if self.provider == "openai":
            return chat_openai(message)
        elif self.provider == "claude":
            return chat_claude(message)
```

### Thứ 7 — 🏆 Project: CLI Chatbot (4 giờ)
```
chatbot/
├── main.py              # CLI interface (input loop)
├── llm_client.py        # OpenAI + Claude wrapper
├── conversation.py      # Conversation history management
├── config.py            # .env settings
├── requirements.txt
└── README.md
```

**Features:**
1. Chọn model (GPT-4o-mini / Claude Sonnet)
2. Conversation memory (lưu history)
3. System prompt customizable
4. `/clear` — Xóa history
5. `/save` — Lưu conversation ra file
6. `/model gpt` hoặc `/model claude` — Đổi model
7. Streaming output (print từng chunk)

---

## ✅ Đánh Giá Cuối Tuần

| # | Tiêu chí | ✅/❌ |
|---|---------|------|
| 1 | Git: commit, branch, merge, resolve conflict | |
| 2 | GitHub: push, PR, fork workflow | |
| 3 | Giải thích được: token, context window, temperature | |
| 4 | Viết system prompt hiệu quả (ít nhất 3 roles) | |
| 5 | Gọi OpenAI API thành công | |
| 6 | Gọi Anthropic API thành công | |
| 7 | Stream response từ API | |
| 8 | Chatbot có conversation memory | |
| 9 | Code push GitHub với proper .gitignore | |
| 10 | Tính được cost ước tính cho 1 conversation | |

---

> **Tiếp theo**: [Tuần 04: Embeddings & Vector DB →](Tuan-04-Embeddings-VectorDB.md)
