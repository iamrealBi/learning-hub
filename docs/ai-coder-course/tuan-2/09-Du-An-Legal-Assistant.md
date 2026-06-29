# ⚖️ Bài 9: Dự Án — SaaS Legal Assistant

> Xây dựng ứng dụng SaaS soạn thảo hợp đồng với AI và xuất PDF

---

## 1. Mục Tiêu
Tạo web app giúp soạn thảo hợp đồng/thỏa thuận pháp lý bằng AI, có khả năng xuất PDF chuyên nghiệp.

## 2. Tính Năng
- 📝 Template hợp đồng (NDA, Employment, Service Agreement)
- 🤖 AI drafting: mô tả yêu cầu → AI soạn hợp đồng
- ✏️ Editor WYSIWYG để chỉnh sửa
- 📄 Export PDF với format chuyên nghiệp
- 📁 Quản lý documents theo client/project

## 3. Kiến Trúc
```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────→│   Backend    │────→│  LLM API     │
│   React      │     │  ASP.NET Core│     │  Claude/GPT  │
│              │     │              │     │              │
│  - Editor    │     │  - Templates │     │  - Draft     │
│  - Preview   │     │  - PDF Gen   │     │  - Review    │
│  - History   │     │  - Storage   │     │  - Translate │
└──────────────┘     └──────────────┘     └──────────────┘
```

## 4. Bài Học Áp Dụng
- **Professional grade** AI integration
- **MCP** kết nối với document templates
- **Hooks** enforce review trước khi xuất
- **Tests** cho document generation logic

## 5. Kết Nối Với BookWormHub
Kỹ thuật tương tự có thể áp dụng:

- Template-based generation cho BookWormHub reports
- PDF export cho book catalog
- AI-assisted content moderation

---

> **Tuần 2 hoàn tất!** Tiếp theo: [Tuần 3: Multi-Agent →](../Tuan-3-Multi-Agent/01-Agent-Orchestration.md)
