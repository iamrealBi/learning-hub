# 🌐 Bài 9: Dự Án — Website Cá Nhân với AI Digital Twin

> Xây dựng portfolio website có chatbot AI mô phỏng bạn

---

## 1. Mục Tiêu

Tạo **website cá nhân chuyên nghiệp** với tính năng đặc biệt: **AI Digital Twin** — một chatbot AI được train trên thông tin cá nhân của bạn, cho phép khách truy cập "nói chuyện" với phiên bản AI của bạn.

---

## 2. Kiến Trúc

```
┌──────────────────────────────────────┐
│           PERSONAL WEBSITE           │
│                                      │
│  ┌────────────┐  ┌───────────────┐   │
│  │ Portfolio   │  │ AI Digital    │   │
│  │ Sections    │  │ Twin Chatbot  │   │
│  │            │  │               │   │
│  │ - About    │  │ "Hỏi tôi bất │   │
│  │ - Projects │  │  kỳ điều gì!" │   │
│  │ - Skills   │  │               │   │
│  │ - Contact  │  │ [LLM API]     │   │
│  └────────────┘  └───────────────┘   │
└──────────────────────────────────────┘
```

---

## 3. Cách Xây Dựng AI Digital Twin

### Bước 1: Chuẩn bị "bản sao số" của bạn

Tạo file `my-context.md`:
```markdown
# Về Tôi
- Tên: [Tên bạn]
- Vai trò: Software Engineer / SEAP Intern
- Kỹ năng: C#, ASP.NET Core, EF Core, SQL

# Dự Án
- BookWormHub: Ứng dụng quản lý và đánh giá sách
  - ASP.NET Core MVC + EF Core + SQLite
  - Service Layer Pattern, FluentValidation
  - 38+ unit tests

# Sở Thích & Giá Trị
- Đam mê AI-assisted development
- Tin vào clean code và best practices
- Thích học công nghệ mới
```

### Bước 2: System Prompt cho chatbot

```javascript
const systemPrompt = `
Bạn là AI Digital Twin của [Tên]. Hãy trả lời như thể bạn là [Tên].
Sử dụng ngôn ngữ thân thiện, chuyên nghiệp.

Thông tin về bạn:
${myContext}

Quy tắc:
- Trả lời dựa trên thông tin có sẵn
- Nếu không biết, nói "Tôi chưa có thông tin về vấn đề này"
- Giữ tone chuyên nghiệp nhưng thân thiện
- Có thể chia sẻ về dự án và kỹ năng
`;
```

### Bước 3: Tích hợp LLM API

```javascript
async function chatWithTwin(userMessage) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: 'gpt-4o-mini',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userMessage }
      ],
      max_tokens: 500
    })
  });
  
  const data = await response.json();
  return data.choices[0].message.content;
}
```

---

## 4. Quy Trình Vibe Coding

Sử dụng Cursor Agent Mode:

```
Prompt 1: "Tạo personal website hiện đại với:
- Hero section fullscreen với animation gradient
- About section với timeline experience
- Projects grid với hover effects
- Skills section với progress bars
- AI chatbot widget ở góc phải
- Dark/light mode toggle
- Responsive design
- Dùng vanilla HTML/CSS/JS"

Prompt 2: "Kết nối chatbot với OpenAI API, 
sử dụng system prompt từ file my-context.md.
Thêm typing indicator và chat history."

Prompt 3: "Polish UI:
- Thêm smooth scroll
- Animation trên scroll (Intersection Observer)
- Glassmorphism effect cho chatbot
- Custom cursor
- Loading screen"
```

---

## 5. Kết Nối Với SEAP Program

Đây là cơ hội tuyệt vời để:
- 📋 **Showcase BookWormHub** trong portfolio
- 🤖 **Demo AI Digital Twin** cho team
- 📝 **Chứng minh kỹ năng** Vibe Coding cho mentors
- 🎯 **Áp dụng thực tế** kiến thức từ khóa học

---

> **Tuần 1 hoàn tất!** Tiếp theo: [Tuần 2: Agentic Engineering →](../Tuan-2-Agentic-Engineering/01-CLI-Coding-Agents.md)
