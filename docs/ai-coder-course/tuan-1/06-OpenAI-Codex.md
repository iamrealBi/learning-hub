# 🧪 Bài 6: OpenAI Codex — CLI Agent Tự Động

> Codex CLI cho phép bạn giao task cho AI và ngồi xem nó tự code, test, commit

---

## 1. Codex là gì?

**OpenAI Codex** là nền tảng agentic từ OpenAI, cho phép giao các tác vụ coding cho AI thực hiện tự động. Codex hoạt động qua nhiều bề mặt: CLI, VS Code extension, và ChatGPT desktop/mobile.

### Cài đặt:
```bash
npm install -g @openai/codex
```

### Sử dụng cơ bản:
```bash
# Bắt đầu phiên interactive
codex

# One-shot task
codex "Thêm pagination cho BookService, 10 items mỗi trang"

# Goal mode (chạy đến khi hoàn thành)
codex --goal "Implement và test tính năng export CSV cho books"
```

---

## 2. Các Chế Độ Hoạt Động

| Chế độ | Mô tả | Khi nào dùng |
|--------|--------|-------------|
| **Interactive** | Chat qua lại | Debug, explore |
| **One-shot** | 1 lệnh, 1 kết quả | Task nhỏ, cụ thể |
| **Goal** | Chạy đến khi xong | Feature lớn, tự động |

### Goal Mode — "Đặt đích và quên"
```bash
codex --goal "Refactor BookWormHub:

1. Tách HomeService ra khỏi BookService
2. Tạo interface IHomeService
3. Đăng ký DI trong Program.cs
4. Viết unit tests
5. Đảm bảo tất cả tests pass"
```

Codex sẽ **tự chạy liên tục** cho đến khi hoàn thành tất cả 5 bước.

---

## 3. Sandboxed Environment

Codex chạy trong **sandbox** an toàn:

- Chỉ có quyền đọc/ghi trong project directory
- Không thể truy cập internet (trừ khi cho phép)
- Không thể chạy lệnh nguy hiểm
- Mọi thay đổi có thể undo

---

## 4. So sánh với Các CLI Agent Khác

| Tiêu chí | Codex | Claude Code | OpenCode |
|----------|-------|-------------|----------|
| **Nhà phát triển** | OpenAI | Anthropic | Open Source |
| **Model** | GPT-4o, o3 | Claude 4 | Đa model |
| **Giá** | ChatGPT Plus | API/Pro | Free |
| **Goal mode** | ✅ | ❌ (dùng /plan) | ✅ (Build mode) |
| **Mobile monitoring** | ✅ | ❌ | ❌ |
| **Sandbox** | ✅ | ✅ | ❌ |

---

## 5. Best Practices

1. **Cung cấp spec rõ ràng** — Codex hoạt động tốt nhất với yêu cầu cụ thể
2. **Dùng Goal mode cho task lớn** — set and forget
3. **Review kết quả** — sandbox an toàn nhưng vẫn cần human review
4. **Kết hợp với IDE** — dùng Codex cho task nặng, IDE cho tinh chỉnh

---

> **Tiếp theo**: [Bài 7: Google Antigravity →](07-Google-Antigravity.md)
