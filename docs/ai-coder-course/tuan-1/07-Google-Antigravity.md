# 🚀 Bài 7: Google Antigravity — Agentic Development Environment

> Antigravity không phải IDE có AI, mà là nền tảng agent có editor

---

## 1. Antigravity là gì?

**Google Antigravity** là Agentic Development Environment (ADE) ra mắt tháng 11/2025. Khác với IDE truyền thống "thêm AI vào", Antigravity được thiết kế từ đầu với **agent là trung tâm**.

### Điểm khác biệt:
```
IDE truyền thống + AI:              Antigravity (ADE):
━━━━━━━━━━━━━━━━━━━━━              ━━━━━━━━━━━━━━━━━━━━
Editor → Code → AI hỗ trợ          Agent → Plan → Execute
Con người viết, AI gợi ý            Agent viết, con người review
1 assistant                          Nhiều agents song song
```

---

## 2. Hai Giao Diện Chính

### 2.1 Editor View
- Giao diện VS Code quen thuộc
- Dùng khi bạn muốn code trực tiếp
- Tích hợp AI chat, inline editing

### 2.2 Manager View (Mission Control)
- Giao diện điều phối agents
- Spawn nhiều agents song song
- Theo dõi tiến trình từng agent
- Review artifacts (kế hoạch, screenshots, recordings)

```
┌───────────────────────────────────────────┐
│           MANAGER VIEW                    │
│                                           │
│  Agent 1: "Implement BookService"   [✅]  │
│  Agent 2: "Write unit tests"        [🔄]  │
│  Agent 3: "Update documentation"    [⏳]  │
│                                           │
│  [+ New Agent]  [View Artifacts]          │
└───────────────────────────────────────────┘
```

---

## 3. Artifact-Based Verification

Thay vì chỉ sinh code, agents trong Antigravity tạo ra **Artifacts** để bạn review:

| Loại Artifact | Mô tả |
|---------------|--------|
| **Implementation Plan** | Kế hoạch chi tiết trước khi code |
| **Task List** | TODO list với trạng thái |
| **Walkthrough** | Tóm tắt những gì đã làm |
| **Screenshots** | Ảnh chụp kết quả UI |
| **Browser Recordings** | Video ghi lại thao tác trên browser |

### Workflow:
1. Bạn giao task cho agent
2. Agent tạo **Implementation Plan** → Bạn review
3. Bạn comment feedback (kiểu Google Docs)
4. Agent điều chỉnh → Bắt đầu implement
5. Agent tạo **Walkthrough** khi xong → Bạn verify

---

## 4. Model Flexibility

Antigravity hỗ trợ nhiều model:

- **Google Gemini 3/3.5** (mặc định)
- **Anthropic Claude 4.6** 
- **GPT-OSS-120B**
- Có thể chuyển đổi tùy task

---

## 5. Ecosystem

| Component | Mô tả |
|-----------|--------|
| **Desktop App** | Ứng dụng chính trên Windows/Mac/Linux |
| **CLI Tool** | `antigravity` command line |
| **SDK** | Xây dựng custom agent workflows |
| **MCP Support** | Kết nối tools bên ngoài |

---

## 6. Khi Nào Dùng Antigravity?

| Tình huống | Phù hợp? |
|-----------|----------|
| Dự án mới từ đầu | ✅ Rất tốt |
| Multi-agent song song | ✅ Tốt nhất |
| Cần review chi tiết | ✅ Artifact system |
| Dự án C#/.NET hiện có | ✅ Hỗ trợ tốt |
| Cần free tool | ✅ Miễn phí (preview) |
| Team collaboration | ⚠️ Đang phát triển |

---

> **Tiếp theo**: [Bài 8: Dự án Game 3D →](08-Du-An-Game-3D.md)
