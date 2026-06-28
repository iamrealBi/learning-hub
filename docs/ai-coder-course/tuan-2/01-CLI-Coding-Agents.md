# ⌨️ Bài 1: CLI Coding Agents — Tổng Quan

> Chuyển từ IDE-based coding sang CLI agents: mạnh mẽ hơn, linh hoạt hơn, tự động hơn

---

## 1. Tại Sao CLI Agents?

### IDE vs CLI:

```
IDE Agents (Tuần 1):              CLI Agents (Tuần 2):
━━━━━━━━━━━━━━━━━━━━              ━━━━━━━━━━━━━━━━━━━
✅ GUI trực quan                  ✅ Scriptable, tự động hóa
✅ Dễ học, dễ dùng               ✅ Pipeline CI/CD integration
⚠️ Khó tự động hóa              ✅ Headless (không cần GUI)
⚠️ Giới hạn trong IDE            ✅ Kết hợp với bash/shell
                                  ✅ Chạy trên server
```

### Khi nào dùng CLI Agent?

| Tình huống | IDE Agent | CLI Agent |
|-----------|-----------|-----------|
| Prototype nhanh | ✅ | ❌ |
| CI/CD automation | ❌ | ✅ |
| Refactor quy mô lớn | ⚠️ | ✅ |
| Chạy trên server | ❌ | ✅ |
| Task lặp lại (batch) | ❌ | ✅ |
| Interactive debug | ✅ | ⚠️ |

---

## 2. Bảng So Sánh CLI Agents Hàng Đầu

| Agent | Nhà phát triển | Model mặc định | Điểm mạnh | Giá |
|-------|---------------|----------------|-----------|-----|
| **Claude Code** | Anthropic | Claude 4 | Deep reasoning, subagents | API/Pro |
| **OpenCode** | Open Source | Đa model | Free, provider-agnostic | Free |
| **Amp** | Sourcegraph | Frontier models | Codebase intelligence | Enterprise |
| **Codex CLI** | OpenAI | GPT-4o, o3 | Goal mode, sandbox | ChatGPT Plus |
| **Aider** | Open Source | Đa model | Git-native | Free |

---

## 3. Workflow Chung Của CLI Agent

```
┌─────────────────────────────────────────┐
│           CLI AGENT WORKFLOW            │
│                                         │
│  $ claude "Thêm phân trang cho books"   │
│     ↓                                   │
│  1. Agent đọc codebase                  │
│  2. Agent lên kế hoạch                  │
│  3. Agent viết code                     │
│  4. Agent chạy build/test               │
│  5. Nếu lỗi → tự sửa → lặp lại        │
│  6. Thành công → commit (nếu cho phép)  │
│     ↓                                   │
│  $ ✅ Done!                             │
└─────────────────────────────────────────┘
```

---

## 4. CLI Agent Trong Dự Án BookWormHub

### Ví dụ sử dụng Claude Code:

```bash
# Navigate đến project
cd BookWormHub/BookWormHub

# Bắt đầu phiên Claude Code
claude

# Trong phiên:
> Phân tích codebase và cho tôi biết architecture hiện tại

> Thêm HomeService với endpoint thống kê:
  - Tổng số sách
  - Tổng reviews approved
  - Top 5 sách rating cao nhất
  Tạo interface, service, tests

> Chạy dotnet test và sửa bất kỳ lỗi nào
```

---

## 5. Khái Niệm Quan Trọng

| Khái niệm | Giải thích |
|-----------|-----------|
| **Headless** | Chạy không cần GUI, phù hợp server/CI |
| **Scriptable** | Có thể đưa vào bash script tự động |
| **Session** | Phiên làm việc, giữ context conversation |
| **Sandbox** | Môi trường an toàn, giới hạn quyền |
| **Plan Mode** | AI lên kế hoạch trước khi code |
| **Build Mode** | AI thực sự sửa file và chạy lệnh |

---

> **Tiếp theo**: [Bài 2: Claude Code Deep Dive →](02-Claude-Code.md)
