# 📅 TUẦN 04: Embeddings, Vector Databases & Basic RAG

> **Mục tiêu**: Hiểu embeddings, setup vector DB, build pipeline RAG đầu tiên  
> **Thời lượng**: 20-25 giờ | **Output**: 🏆 PDF Q&A Bot (Portfolio Project #1)

---

## 🎯 Sau Tuần Này Bạn Sẽ:
- [ ] Hiểu text embeddings và semantic similarity
- [ ] Setup ChromaDB local
- [ ] Load & chunk documents (PDF, text, web)
- [ ] Build basic RAG pipeline: Query → Retrieve → Generate
- [ ] Evaluate RAG quality (relevance, hallucination)
- [ ] 🏆 Deploy PDF Q&A Bot

---

## 📚 Khóa Học

### 🔴 Khóa chính — FREE (Tốt hơn Udemy cho topic này):

**1. [DeepLearning.AI: LangChain — Chat with Your Data](https://www.deeplearning.ai/short-courses/langchain-chat-with-your-data/)** 🥇
- Giảng viên: **Harrison Chase** (CEO LangChain — creator dạy trực tiếp!)
- ⭐⭐⭐⭐⭐ | FREE | ~2 giờ | Hands-on Jupyter notebooks
- Covers: Document loaders, text splitting, vector stores, retrieval, Q&A
- **Tại sao tốt hơn Udemy**: Creator trực tiếp dạy, luôn cập nhật chuẩn mới nhất

**2. [DeepLearning.AI: Vector Databases — Embeddings to Applications](https://www.deeplearning.ai/short-courses/vector-databases-embeddings-applications/)** 🥇
- Giảng viên: **Sebastian Witalec** (Weaviate team)
- ⭐⭐⭐⭐⭐ | FREE | ~1.5 giờ | Hands-on
- Covers: Embeddings, similarity search, ANN algorithms, vector DB internals

**3. [DeepLearning.AI: Building Agentic RAG with LlamaIndex](https://www.deeplearning.ai/short-courses/building-agentic-rag-with-llamaindex/)** 🥇
- ⭐⭐⭐⭐⭐ | FREE | ~1.5 giờ
- Covers: Agentic RAG patterns, multi-document agents, router queries

**4. [ChromaDB Official Getting Started](https://docs.trychroma.com/getting-started)** 🥇
- FREE | Official docs | Luôn đúng phiên bản mới nhất

### 🟡 Bổ trợ — Coursera (có chứng chỉ):
**[Coursera: RAG for GenAI Applications](https://www.coursera.org/learn/rag-for-gen-ai)** (IBM)
- Structured, có graded assignments + certificate

### 🟢 Tham khảo thêm — Udemy Business (nếu muốn xem video dài hơn):
**[GenAI Architectures with LLM, RAG, Vector DB](https://ibmcsr.udemy.com/course/generative-ai-architectures/)** — Mehmet Ozkaya | 7.5h
- ⚠️ Overview level, không sâu bằng DeepLearning.AI
- Dùng khi muốn xem thêm ví dụ hoặc prefer video dài hơn

> 💡 **Tại sao không dùng Udemy làm PRIMARY?**
> DeepLearning.AI courses được dạy bởi CHÍNH creators của LangChain/Weaviate.
> Udemy instructors là third-party → có thể outdated hoặc miss best practices.

---

## 📅 Lịch Học Theo Ngày

### Thứ 2 — Embeddings (3.5 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.5h | 🎬 Udemy: GenAI Architectures — Embeddings sections |
| 1.0h | 💻 Thực hành: OpenAI embeddings API |
| 0.5h | 💻 Bài tập: Semantic similarity calculator |
| 0.5h | 🆓 DeepLearning.AI: Vector Databases short course |

**Bài tập:**
```python
from openai import OpenAI
import numpy as np

client = OpenAI()

def get_embedding(text: str) -> list[float]:
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

def cosine_similarity(a: list[float], b: list[float]) -> float:
    a, b = np.array(a), np.array(b)
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# Test: So sánh semantic similarity
texts = [
    "Python is a programming language",
    "Python là ngôn ngữ lập trình",     # Cùng nghĩa, khác ngôn ngữ
    "I love eating pizza",                # Khác nghĩa hoàn toàn
    "Java is used for software development"  # Liên quan nhưng khác
]
# Tính similarity matrix
```

### Thứ 3 — Vector Database (ChromaDB) (3.5 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.0h | 🎬 Udemy: Vector DB sections |
| 1.5h | 💻 Setup ChromaDB, CRUD operations |
| 0.5h | 💻 Bài tập: Store & query book descriptions |
| 0.5h | 📝 Notes: Vector DB comparison chart |

**Bài tập:**
```python
import chromadb

client = chromadb.Client()
collection = client.create_collection("books")

# Add documents
collection.add(
    documents=[
        "Clean Code by Robert Martin - A handbook of agile software",
        "Design Patterns by Gang of Four - Elements of reusable OO",
        "Python Crash Course - A hands-on project-based introduction",
    ],
    ids=["book1", "book2", "book3"],
    metadatas=[
        {"genre": "programming", "year": 2008},
        {"genre": "programming", "year": 1994},
        {"genre": "tutorial", "year": 2019},
    ]
)

# Semantic search
results = collection.query(
    query_texts=["How to write better code?"],
    n_results=2
)
print(results)  # Should return Clean Code + Design Patterns
```

### Thứ 4 — Document Loading & Chunking (3.5 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.0h | 🎬 Udemy: Document processing sections |
| 1.5h | 💻 PDF loader, text splitter, chunk strategies |
| 0.5h | 💻 Bài tập: Chunk một PDF thật |
| 0.5h | 📝 Compare chunking strategies |

**Bài tập:**
```python
# Bài 1: Simple text chunker
def chunk_text(text: str, chunk_size: int = 500, overlap: int = 50) -> list[str]:
    """Split text vào chunks với overlap"""
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunk = text[start:end]
        chunks.append(chunk)
        start = end - overlap
    return chunks

# Bài 2: PDF loader
from pypdf import PdfReader

def load_pdf(filepath: str) -> str:
    reader = PdfReader(filepath)
    text = ""
    for page in reader.pages:
        text += page.extract_text() + "\n"
    return text

# Bài 3: Smart chunker (split by paragraphs first)
def smart_chunk(text: str, max_size: int = 500) -> list[str]:
    """Split by paragraphs, merge small ones, split large ones"""
    pass
```

### Thứ 5 — Basic RAG Pipeline (4 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.0h | 🎬 Udemy: RAG architecture sections |
| 2.0h | 💻 Build complete RAG pipeline from scratch |
| 0.5h | 💻 Test with different queries |
| 0.5h | 🆓 DeepLearning.AI: Chat with Your Data |

**Bài tập:**
```python
class SimpleRAG:
    def __init__(self):
        self.chroma = chromadb.Client()
        self.collection = self.chroma.create_collection("docs")
        self.openai = OpenAI()
    
    def ingest(self, documents: list[str], ids: list[str]):
        """Load documents vào vector store"""
        self.collection.add(documents=documents, ids=ids)
    
    def retrieve(self, query: str, k: int = 3) -> list[str]:
        """Tìm top-k documents liên quan"""
        results = self.collection.query(query_texts=[query], n_results=k)
        return results['documents'][0]
    
    def generate(self, query: str, context: list[str]) -> str:
        """Generate answer dựa trên context"""
        context_text = "\n---\n".join(context)
        response = self.openai.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": f"Answer based ONLY on this context:\n{context_text}"},
                {"role": "user", "content": query}
            ]
        )
        return response.choices[0].message.content
    
    def ask(self, query: str) -> str:
        """Full RAG pipeline: retrieve → generate"""
        context = self.retrieve(query)
        return self.generate(query, context)
```

### Thứ 6 — RAG Evaluation (3 giờ)
| Thời gian | Hoạt động |
|-----------|-----------|
| 1.0h | 🎬 Udemy: Evaluation sections |
| 1.0h | 💻 Build evaluation framework |
| 0.5h | 💻 Test RAG quality metrics |
| 0.5h | 📝 Document findings |

**Metrics:**
```python
def evaluate_rag(rag, test_cases: list[dict]) -> dict:
    """
    test_case = {
        "question": "...",
        "expected_answer": "...",
        "expected_source": "..."
    }
    """
    results = {"correct": 0, "total": len(test_cases)}
    for case in test_cases:
        answer = rag.ask(case["question"])
        # Check if answer contains expected info
        # Check if no hallucination
    return results
```

### Thứ 7 — 🏆 PROJECT 1: PDF Q&A Bot (4 giờ)
```
pdf_qa_bot/
├── main.py           # CLI interface
├── rag.py            # RAG pipeline class
├── document.py       # PDF loader + chunker
├── vectordb.py       # ChromaDB wrapper
├── config.py         # Settings
├── evaluate.py       # Quality testing
├── requirements.txt
├── README.md         # With architecture diagram
└── sample_docs/      # Test PDFs
```

**Demo flow:**
```bash
$ python main.py --pdf sample_docs/python_guide.pdf
📄 Loading PDF... 42 pages
📦 Chunking... 156 chunks
🔢 Embedding... done
💾 Stored in ChromaDB

💬 Ask a question (or /quit):
> What is a decorator in Python?
🤖 Based on the document, a decorator is a function that...
   📎 Source: Page 23, Section 4.2
```

---

## ✅ Đánh Giá Cuối Tuần

| # | Tiêu chí | ✅/❌ |
|---|---------|------|
| 1 | Giải thích embeddings là gì, tại sao cần | |
| 2 | Tính cosine similarity giữa 2 text embeddings | |
| 3 | CRUD operations trên ChromaDB | |
| 4 | Load và chunk PDF thành text segments | |
| 5 | Build RAG pipeline: retrieve → generate | |
| 6 | RAG trả lời đúng ≥ 70% test questions | |
| 7 | Phát hiện được hallucination trong RAG output | |
| 8 | PDF Q&A Bot hoạt động end-to-end | |
| 9 | Code trên GitHub với README + architecture diagram | |
| 10 | So sánh được chunk_size khác nhau ảnh hưởng chất lượng | |

### 🏆 Portfolio Project #1 Checklist:
- [ ] GitHub repo public với README chuyên nghiệp
- [ ] Architecture diagram (Mermaid hoặc image)
- [ ] Demo GIF/video trong README
- [ ] Ít nhất 5 test cases documented
- [ ] Requirements.txt hoàn chỉnh

---

> 🎉 **Chúc mừng hoàn thành Tháng 1!** Bạn đã có nền tảng Python + LLM + RAG
> 
> **Tiếp theo**: [Tháng 2 — Tuần 05: Function Calling & ReAct →](../Thang-2-AI-Agent-Basics/Tuan-05-Function-Calling.md)
