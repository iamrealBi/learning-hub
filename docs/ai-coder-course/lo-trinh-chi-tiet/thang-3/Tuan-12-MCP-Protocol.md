# 📅 TUẦN 12: MCP (Model Context Protocol)

> **Output**: 🏆 Custom Database MCP Server (Portfolio Project #6)

## 📚 Nguồn Học — Best Quality

### 🥇 PRIMARY — FREE (Official = duy nhất đáng tin cho MCP):
1. **[Anthropic Academy: Introduction to MCP](https://anthropic.skilljar.com/)** 🏅
   - Official Anthropic, FREE, **có certificate LinkedIn-ready**
   - ⭐⭐⭐⭐⭐ | MCP quá mới → CHỈ official source đáng tin
2. **[Anthropic Academy: MCP Advanced Topics](https://anthropic.skilljar.com/)** 🏅
   - Server building, security, production deployment
3. **[MCP Official Specification](https://modelcontextprotocol.io/)** 
   - The spec itself — reference
4. **[MCP Official Docs + Tutorials](https://modelcontextprotocol.io/docs)** 
   - Getting started, building servers/clients

### 🥈 SUPPLEMENT — Udemy (thêm project examples):
- **[MCP Crash Course (Eden Marco)](https://ibmcsr.udemy.com/course/mcp-crash-course/)** — 8.5h
  - ✅ Nhiều examples thực tế, tốt cho hands-on practice
  - Dùng SAU KHI nắm vững spec từ Anthropic Academy
- **[Complete MCP Developer Guide](https://ibmcsr.udemy.com/course/complete-mcp-guide/)** — 6.5h
  - Supplementary projects

### 🥈 SUPPLEMENT — Coursera (chứng chỉ) 🎓:
- **[IBM RAG and Agentic AI Professional Certificate](https://www.coursera.org/professional-certificates/ibm-rag-agentic-ai)** — MCP sections
  - IBM program covers MCP → hoàn thành và lấy certificate cuối tháng 3
  - 🎯 **Mục tiêu**: Nộp xong IBM certificate SAU tuần 12

---

| Ngày | Topic | Source |
|------|-------|--------|
| T2 | MCP Architecture: primitives, transport | Anthropic Academy |
| T3 | Using existing MCP servers | MCP docs + npm registry |
| T4 | Building MCP Server (Python FastMCP) | Anthropic Academy + docs |
| T5 | MCP in Claude Code + Cursor | Official docs |
| T6 | Production MCP: HTTP/SSE, auth | Anthropic Academy Advanced |
| T7 | 🏆 **Database MCP Server** | Build project |

### 🏆 PROJECT 6: BookWormHub Database MCP Server
```python
# FastMCP server for BookWormHub
from fastmcp import FastMCP

mcp = FastMCP("BookWormHub DB")

@mcp.tool()
def query_books(search: str, genre: str = None) -> list[dict]:
    """Search books by title/author, optionally filter by genre"""
    pass

@mcp.tool()
def get_statistics() -> dict:
    """Get library statistics: total books, genres, avg rating"""
    pass

@mcp.resource("schema://database")
def get_schema() -> str:
    """Return the database schema"""
    pass
```

## ✅ Đánh Giá: ≥ 8/10
| # | Tiêu chí |
|---|---------|
| 1 | Giải thích MCP architecture (host, client, server) |
| 2 | Configure ≥ 3 existing MCP servers |
| 3 | Build custom MCP server với FastMCP |
| 4 | Server có tools + resources |
| 5 | Connect từ Claude Code |
| 6 | Connect từ Cursor |
| 7 | HTTP/SSE transport hoạt động |
| 8 | BookWormHub MCP server deployed |

> 🎉 **Hoàn thành Tháng 3!** Bạn đã master LangGraph + CrewAI + MCP
> **Tiếp theo**: [Tháng 4 — Tuần 13: Docker & Cloud →](../Thang-4-Production/Tuan-13-Docker-Cloud.md)
