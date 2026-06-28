# 📈 Bài 5: Capstone Project — Real-time Trading Workstation

> Dự án tổng hợp: multi-agent xây dựng ứng dụng trading

---

## 1. Mục Tiêu
Xây dựng **Trading Workstation** hoàn chỉnh sử dụng multi-agent workflow:
- Dashboard real-time với biểu đồ
- Virtual trading (giao dịch ảo)
- AI assistant cho chiến lược
- Live market data integration

---

## 2. Kiến Trúc Multi-Agent

```
┌─────────────────────────────────────────────┐
│          TRADING WORKSTATION                │
│                                             │
│  Agent 1: Backend API                       │
│  ├── Market data service                    │
│  ├── Trading engine                         │
│  └── Portfolio manager                      │
│                                             │
│  Agent 2: Frontend Dashboard                │
│  ├── Real-time charts (Chart.js/D3)         │
│  ├── Order panel                            │
│  └── Portfolio view                         │
│                                             │
│  Agent 3: AI Strategy Assistant             │
│  ├── Market analysis                        │
│  ├── Trade recommendations                  │
│  └── Risk assessment                        │
│                                             │
│  Agent 4: Testing & QA                      │
│  ├── Unit tests                             │
│  ├── Integration tests                      │
│  └── Performance tests                      │
│                                             │
│  Agent 5: Documentation                     │
│  ├── API docs                               │
│  ├── User guide                             │
│  └── Architecture docs                      │
└─────────────────────────────────────────────┘
```

---

## 3. Workflow

### Phase 1: Planning (Human + AI)
```bash
claude "/plan Design a trading workstation:
- React frontend with real-time charts
- Node.js/Express backend  
- WebSocket for live data
- Virtual portfolio with $100k starting balance
- AI chatbot for strategy advice"
```

### Phase 2: Parallel Implementation (Multi-Agent)
```bash
# Terminal 1: Backend Agent
cd worktree-backend && claude "Implement backend API..."

# Terminal 2: Frontend Agent  
cd worktree-frontend && claude "Implement React dashboard..."

# Terminal 3: AI Agent
cd worktree-ai && claude "Implement strategy chatbot..."

# Terminal 4: Test Agent
cd worktree-tests && claude "Write comprehensive tests..."
```

### Phase 3: Integration (Human Review)
```bash
git merge feature/backend feature/frontend feature/ai feature/tests
dotnet test  # or npm test
```

---

## 4. Bài Học Tổng Kết Khóa Học

| Tuần | Kỹ năng | Áp dụng trong Capstone |
|------|---------|----------------------|
| **Tuần 1** | Vibe Coding, IDE agents | Prototype nhanh UI |
| **Tuần 2** | CLI agents, MCP, automation | Backend development |
| **Tuần 3** | Multi-agent, orchestration | Phân công 5 agents |

### Hành trình hoàn chỉnh:
```
Vibe Coder → Vibe Engineer → Agentic Engineer
   (IDE)        (CLI)          (Multi-Agent)
   (Fun)      (Professional)    (Scalable)
```

---

## 5. Kết Nối Với SEAP & BookWormHub

Kỹ thuật từ Capstone có thể áp dụng cho BookWormHub:
1. **Multi-agent** cho tính năng phức tạp
2. **Real-time dashboard** cho admin panel
3. **AI assistant** cho book recommendations
4. **Automated testing** với agent teams

---

> 🎓 **Chúc mừng bạn đã hoàn thành khóa học!**
> Quay lại [README](../README.md) để xem tổng quan
