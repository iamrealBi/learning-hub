# 🧠 Bài 2: Nền Tảng LLM — Tokens, Context Window & Reasoning

> Hiểu cách LLM hoạt động là nền tảng để trở thành Agentic Engineer hiệu quả

---

## 1. Token & Tokenization

### Token là gì?
**Token** là đơn vị dữ liệu nhỏ nhất mà LLM xử lý. Token **không phải lúc nào cũng là một từ hoàn chỉnh** — nó có thể là:
- Một từ: `hello` → 1 token
- Một phần từ: `tokenization` → `token` + `ization` (2 tokens)
- Ký tự đặc biệt: `!`, `.` → mỗi ký tự 1 token
- Khoảng trắng: Thường gộp với từ tiếp theo

### Tại sao cần Tokenization?
LLM **không thể đọc text thô** — chúng cần số. Tokenization chuyển text thành chuỗi số (ID), sau đó thành **embedding vectors** (vector đa chiều mang ý nghĩa ngữ nghĩa).

```
Text: "Xin chào thế giới"
  ↓ Tokenization
Tokens: ["Xin", " chào", " thế", " giới"]
  ↓ Token IDs
IDs: [15234, 8901, 12045, 7832]
  ↓ Embeddings
Vectors: [[0.12, -0.45, ...], [0.78, 0.23, ...], ...]
```

### Subword Tokenization (BPE)
Hầu hết LLM hiện đại sử dụng **Byte-Pair Encoding (BPE)**:
- Cân bằng kích thước từ vựng với hiệu quả
- Xử lý được từ lạ, lỗi chính tả, thuật ngữ mới
- Ví dụ: `"programming"` → `"program"` + `"ming"`

### Tác động đến Developer

| Khía cạnh | Ảnh hưởng |
|-----------|-----------|
| **Chi phí** | Tính theo token → Prompt tối ưu = tiết kiệm tiền |
| **Tốc độ** | Nhiều token hơn = phản hồi chậm hơn |
| **Giới hạn** | Mỗi model có giới hạn token tối đa |
| **Tiếng Việt** | Tiếng Việt thường tốn nhiều token hơn tiếng Anh |

### Ví dụ so sánh chi phí:

```
Prompt kém:
"Hãy viết cho tôi một hàm trong ngôn ngữ lập trình Python, hàm này
nhận vào một danh sách các số nguyên và trả về tổng của tất cả 
các số chẵn trong danh sách đó, vui lòng sử dụng vòng lặp for"
→ ~60 tokens

Prompt tối ưu:
"Python: hàm tính tổng số chẵn trong list[int], dùng for loop"
→ ~20 tokens

→ Tiết kiệm ~67% chi phí, kết quả tương đương!
```

---

## 2. Context Window — "Bộ nhớ làm việc" của AI

### Context Window là gì?
Context Window là **số token tối đa** mà model có thể xem xét trong một lần request-response. Nó bao gồm **cả input VÀ output**.

```
┌─────────────────────────────────────────────┐
│           CONTEXT WINDOW (128K tokens)      │
│                                             │
│  ┌──────────────┐  ┌──────────────────────┐ │
│  │ INPUT        │  │ OUTPUT               │ │
│  │ - System     │  │ - Response từ AI     │ │
│  │   prompt     │  │ - Code sinh ra       │ │
│  │ - Lịch sử    │  │ - Giải thích         │ │
│  │   chat       │  │                      │ │
│  │ - File code  │  │                      │ │
│  │ - Documents  │  │                      │ │
│  └──────────────┘  └──────────────────────┘ │
│                                             │
│  ← Tổng KHÔNG được vượt quá context window →│
└─────────────────────────────────────────────┘
```

### Kích thước Context Window qua các model (2025-2026):

| Model | Context Window | Tương đương |
|-------|---------------|-------------|
| GPT-4 Turbo | 128K tokens | ~300 trang sách |
| Claude 3.5 Sonnet | 200K tokens | ~500 trang sách |
| Gemini 1.5 Pro | 1M tokens | ~2,500 trang sách |
| Claude 4 | 200K tokens | ~500 trang sách |
| GPT-o3 | 200K tokens | ~500 trang sách |

### Vấn đề "Lost in the Middle"

> ⚠️ **Quan trọng**: LLM thường **bỏ sót thông tin nằm ở giữa** context dài. Chúng xử lý tốt nhất ở **đầu** và **cuối** context.

```
Độ chính xác truy hồi thông tin:

Đầu context  ████████████████████  95%
Giữa context ████████░░░░░░░░░░░░  40-60%
Cuối context ██████████████████░░  85%
```

### Best Practices cho Context Window:

1. **Đặt thông tin quan trọng ở đầu hoặc cuối** prompt
2. **Không nhồi nhét** — "nhiều hơn" không phải lúc nào cũng "tốt hơn"
3. **Sử dụng RAG** (Retrieval-Augmented Generation) cho tài liệu lớn
4. **Tóm tắt lịch sử chat** khi phiên quá dài (`/compact` trong Claude Code)

---

## 3. Reasoning & Chain-of-Thought (CoT)

### Chain-of-Thought là gì?
CoT là kỹ thuật buộc model **"suy nghĩ từng bước"** thay vì nhảy thẳng đến kết quả. Điều này **giảm đáng kể lỗi logic**.

### So sánh:

**Không có CoT:**
```
Prompt: "15% của 240 là bao nhiêu?"
AI: "36" ← Có thể sai, nhất là với phép tính phức tạp hơn
```

**Có CoT:**
```
Prompt: "15% của 240 là bao nhiêu? Hãy giải thích từng bước."
AI: 
"Bước 1: 15% = 15/100 = 0.15
 Bước 2: 0.15 × 240 = 36
 Đáp án: 36" ← Chính xác và kiểm chứng được
```

### Reasoning Models (o-series, thinking modes)

Các model hiện đại có khả năng **reasoning nội bộ** — sinh ra "thought tokens" ẩn để kiểm tra logic trước khi trả lời.

```
┌────────────────────────────────────────┐
│         REASONING MODEL               │
│                                       │
│  Input ──→ [Suy nghĩ ẩn]  ──→ Output │
│            - Phân tích     │          │
│            - Kiểm tra      │          │
│            - Tự sửa lỗi    │          │
│            (tốn token!)    │          │
└────────────────────────────────────────┘
```

### Khi nào dùng Reasoning Model?

| Tình huống | Dùng model nhanh | Dùng reasoning model |
|-----------|------------------|---------------------|
| Autocomplete code | ✅ | ❌ |
| Trả lời nhanh | ✅ | ❌ |
| Debug logic phức tạp | ❌ | ✅ |
| Thiết kế kiến trúc | ❌ | ✅ |
| Toán/thuật toán | ❌ | ✅ |
| Viết test edge-case | ❌ | ✅ |

---

## 4. Bảng Tóm Tắt Cho Developer

| Khái niệm | Ý nghĩa cho Developer |
|-----------|----------------------|
| **Token** | "Đơn vị tiền tệ" của chi phí — tối ưu prompt = tiết kiệm |
| **Tokenization** | Ảnh hưởng hiệu suất; hiểu cách tokenizer xử lý dữ liệu của bạn |
| **Context Window** | Bộ nhớ làm việc; dùng RAG hoặc tóm tắt, đừng "nhồi nhét" |
| **Chain-of-Thought** | Công cụ tăng độ chính xác; dùng cho logic phức tạp, tránh cho tác vụ đơn giản |

---

## 5. Bài Tập Thực Hành

### Bài 1: Đếm token
Truy cập [OpenAI Tokenizer](https://platform.openai.com/tokenizer) và thử:
- Nhập một đoạn code C# → đếm token
- Nhập cùng nội dung bằng tiếng Việt → so sánh
- Thử tối ưu prompt để giảm token

### Bài 2: Thử nghiệm "Lost in the Middle"
- Tạo một prompt dài với 10 yêu cầu
- Đặt yêu cầu quan trọng nhất ở giữa
- Quan sát xem AI có bỏ sót không
- Thử chuyển yêu cầu đó lên đầu hoặc cuối

### Bài 3: Chain-of-Thought
- Yêu cầu AI giải một bài toán phức tạp mà KHÔNG kèm "hãy giải thích từng bước"
- Sau đó thử lại với "hãy phân tích từng bước trước khi đưa ra đáp án"
- So sánh chất lượng hai kết quả

---

> **Tiếp theo**: [Bài 3: Context Engineering →](03-Context-Engineering.md)
