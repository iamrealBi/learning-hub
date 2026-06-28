# 📅 TUẦN 07: Advanced RAG Pipeline

> **Mục tiêu**: Hybrid search, reranking, query transformation, evaluation  
> **Output**: 🏆 Multi-source RAG System (Portfolio Project #3)

---

## 📚 Nguồn Học — Best Quality

### 🥇 PRIMARY — FREE:
1. **[DeepLearning.AI: Building Agentic RAG with LlamaIndex](https://www.deeplearning.ai/short-courses/building-agentic-rag-with-llamaindex/)** — Agentic RAG patterns
2. **[DeepLearning.AI: Knowledge Graphs for RAG](https://www.deeplearning.ai/short-courses/knowledge-graphs-rag/)** — Advanced retrieval
3. **[LangChain Docs: RAG Tutorials](https://python.langchain.com/docs/tutorials/rag/)** — Official, always current
4. **[RAGAS Docs](https://docs.ragas.io/)** — Evaluation framework, FREE

### 🥈 SUPPLEMENT — Coursera:
- **[RAG for GenAI Applications (IBM)](https://www.coursera.org/learn/rag-for-gen-ai)** — Certificate

### 🥉 OPTIONAL — Udemy:
- Udemy: Basic to Advanced RAG (Yash Thakker) — 2.5h, OK nhưng DLAI tốt hơn

---

## 📅 Tuần Này:
| Ngày | Topic | Source |
|------|-------|--------|
| T2 | Hybrid Search (keyword + semantic) | LangChain docs |
| T3 | Reranking (cross-encoder, Cohere) | DeepLearning.AI |
| T4 | Query Transformation (HyDE, Multi-query) | LangChain docs |
| T5 | Agentic RAG (agent decides what to retrieve) | DLAI Agentic RAG |
| T6 | Evaluation (RAGAS framework) | RAGAS official docs |
| T7 | 🏆 **Multi-source RAG** | Build project |

### 🏆 PROJECT 3: Multi-source RAG System
```
Input: Câu hỏi về BookWormHub
Sources: Code + Docs + API specs
Pipeline: Route → Retrieve → Rerank → Generate
Output: Answer + source citations + confidence score
Eval: RAGAS metrics ≥ 0.7
```

## ✅ Đánh Giá: ≥ 8/10 để tiếp tục
| # | Tiêu chí |
|---|---------|
| 1 | Implement hybrid search (BM25 + vector) |
| 2 | Reranking cải thiện relevance ≥ 15% |
| 3 | Multi-query retrieval hoạt động |
| 4 | Agent tự quyết định khi nào cần retrieve |
| 5 | RAGAS evaluation pipeline chạy tự động |
| 6 | Multi-source routing (code vs docs vs API) |
| 7 | Source citations trong mỗi answer |
| 8 | Benchmark: compare basic vs advanced RAG metrics |
| 9 | GitHub repo với evaluation results |
| 10 | Giải thích trade-offs: accuracy vs latency vs cost |

> **Tiếp theo**: [Tuần 08 →](Tuan-08-Vibe-Coding-Tools.md)
