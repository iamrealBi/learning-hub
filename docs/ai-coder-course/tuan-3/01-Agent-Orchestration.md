# 🎭 Bài 1: Agent Orchestration — Tổng Quan

> Từ 1 AI assistant → đội quân AI agents: the next frontier

---

## 1. Tại Sao Multi-Agent?

### Giới hạn của Single Agent:
```
Single Agent:                      Multi-Agent:
━━━━━━━━━━━━                      ━━━━━━━━━━━━

- 1 context window                - N context windows
- Xử lý tuần tự                  - Xử lý song song
- Context bloat                   - Context isolated
- 1 perspective                   - N perspectives
- Chậm với task lớn              - Nhanh gấp N lần
```

### Ví dụ: Xây dựng feature cho BookWormHub

**Single Agent** (tuần tự):
```
Claude → Sửa Model → Sửa Service → Sửa Controller → Viết Tests → Sửa Views
Thời gian: ████████████████████████████████ 30 phút
```

**Multi-Agent** (song song):
```
Agent 1: Model + Service    ██████████
Agent 2: Controller + Views ██████████  → Merge
Agent 3: Tests              ██████████
Thời gian: ██████████ 10 phút
```

---

## 2. Hai Pattern Chính

### Pattern 1: Subagents (Hierarchical)
```
         ┌─────────────┐
         │ Orchestrator │
         │    Agent     │
         └──────┬──────┘
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
   ┌────────┐ ┌────┐ ┌────────┐
   │Sub A   │ │Sub B│ │Sub C   │
   │Backend │ │Tests│ │Frontend│
   └────────┘ └────┘ └────────┘
        │       │       │
        └───────┼───────┘
                ▼
         Kết quả tổng hợp
```

**Đặc điểm:**

- Orchestrator phân chia task
- Subagents **không nói chuyện với nhau**
- Report kết quả lên orchestrator
- Best for: tasks độc lập

### Pattern 2: Agent Teams (Peer-to-Peer)
```
   ┌─────────────┐
   │    Lead      │
   │   Agent      │
   └──────┬──────┘
          │ Shared Task List
   ┌──────┼──────┐
   ▼      ▼      ▼
 ┌────┐ ┌────┐ ┌────┐
 │Mate│◄►│Mate│◄►│Mate│
 │ 1  │ │ 2  │ │ 3  │
 └────┘ └────┘ └────┘
   ↕      ↕      ↕
   Peer-to-peer communication
```

**Đặc điểm:**

- Teammates **có thể giao tiếp** với nhau
- Shared task list (source of truth)
- Lead define scope, teammates execute
- Best for: tasks có dependencies

---

## 3. So Sánh Patterns

| Tiêu chí | Subagents | Agent Teams |
|----------|-----------|-------------|
| **Communication** | Hierarchical | Peer-to-peer |
| **Independence** | Cao (isolated) | Trung bình (coordinated) |
| **Complexity** | Thấp | Cao |
| **Best for** | Independent tasks | Cross-cutting changes |
| **Risk** | Merge conflicts | Communication overhead |
| **Example tools** | Claude Code subagents | Claude Agent Teams |

---

## 4. Execution Backends

| Backend | Cách hoạt động | Ưu điểm |
|---------|----------------|---------|
| **tmux panes** | Mỗi agent = 1 terminal pane | Visual, easy debug |
| **Git worktrees** | Mỗi agent = 1 worktree riêng | Không conflict files |
| **Docker containers** | Mỗi agent = 1 container | Isolated completely |
| **In-process** | Async/coroutines | Fastest, no overhead |

---

## 5. Observability

> "Bạn không thể quản lý thứ bạn không nhìn thấy"

### Dashboard cho multi-agent:
```
┌──────────────────────────────────────────┐
│         AGENT MONITORING DASHBOARD       │
│                                          │
│  Agent 1 [Backend]    ██████████░░ 80%   │
│  Status: Writing ReviewService.cs        │
│                                          │
│  Agent 2 [Tests]      ████████████ 100%  │
│  Status: ✅ All 15 tests pass            │
│                                          │
│  Agent 3 [Frontend]   ██████░░░░░░ 50%   │
│  Status: Editing Views/Book/Index.cshtml │
│                                          │
│  [Pause All] [Resume] [Redirect Agent 3] │
└──────────────────────────────────────────┘
```

---

> **Tiếp theo**: [Bài 2: Subagents & Swarms →](02-Subagents-Swarms.md)
