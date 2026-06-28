# 🎤 Demo Script — Claude Code & AI Coding cho Team Sync

> Kịch bản demo 15-20 phút cho buổi meeting với Dũng, Phước, và Hiếu

---

## 📋 Agenda

| Thời gian | Nội dung | Phương pháp |
|-----------|----------|-------------|
| 0-3 phút | Giới thiệu AI Coding Landscape | Slides/talk |
| 3-8 phút | Demo 1: Context Engineering | Live demo |
| 8-13 phút | Demo 2: Claude Code viết tests | Live demo |
| 13-18 phút | Demo 3: Code Review Automation | Live demo |
| 18-20 phút | Q&A & Next Steps | Discussion |

---

## 🎬 Demo 1: Context Engineering (5 phút)

### Mục tiêu: Cho team thấy CLAUDE.md/AGENTS.md tạo khác biệt

**Script nói:**
> "Vấn đề lớn nhất khi dùng AI là nó không hiểu dự án của mình. Giải pháp là Context Engineering — tạo file hướng dẫn cho AI."

**Bước demo:**
1. Mở BookWormHub project
2. Show file CLAUDE.md đã tạo
3. Giải thích từng section: architecture, rules, commands
4. Show kết quả: AI hiểu đúng pattern, không cần giải thích lại mỗi lần

**Talking point:**
> "Trước đây mỗi lần dùng AI, mình phải giải thích 'dự án dùng Service Layer, FluentValidation...' Giờ AI tự đọc CLAUDE.md và biết tất cả."

---

## 🎬 Demo 2: AI Viết Edge-case Tests (5 phút)

### Mục tiêu: Chứng minh AI phát hiện edge cases mà human bỏ sót

**Script nói:**
> "Mình đã dùng AI để scan 38 tests hiện có và phát hiện 18+ edge cases thiếu. Để mình demo một cái."

**Bước demo:**
1. Mở `BookServiceTests.cs` — show tests hiện có
2. Highlight: "Không có test cho combined search + genre filter"
3. Mở terminal, chạy:
   ```
   Prompt cho AI: "Nhìn vào BookService.GetBookListAsync() và 
   BookServiceTests.cs. Viết test cho trường hợp search VÀ genre 
   filter cùng lúc. Follow pattern từ tests hiện có."
   ```
4. AI sinh test → paste vào → chạy `dotnet test`
5. Show: test PASS → edge case đã được cover

**Talking point:**
> "AI không thay thế mình viết tests, mà nó phát hiện những chỗ mình quên. 38 tests tăng lên 50+ chỉ trong 1 buổi chiều."

---

## 🎬 Demo 3: Code Review Automation (5 phút)

### Mục tiêu: Demo pre-commit review tự động

**Bước demo:**
1. Tạo một file mới cố ý vi phạm rules:
   ```csharp
   // Bad: Business logic trong Controller
   public class TestController : Controller
   {
       private readonly AppDbContext _db;
       
       public async Task<IActionResult> Index()
       {
           var books = _db.Books.ToList(); // ← Sync call!
           return View(books);
       }
   }
   ```
2. Chạy `pre-commit-review.ps1`
3. Show: Script bắt 2 violations:
   - ⚠️ DB access in Controller
   - ⚠️ Sync DB call (.ToList instead of .ToListAsync)
4. Sửa code → chạy lại → ✅ Pass

**Talking point:**
> "Script này chạy tự động trước mỗi commit. Ngoài ra, mình đã setup GitHub Action để review mọi PR. Không cần người review từng dòng code nữa — AI bắt lỗi pattern, mình focus vào logic."

---

## 💡 Key Messages Cho Team

### 1. AI Coding ≠ "AI viết code thay mình"
> "Đúng hơn là: AI là junior developer siêu nhanh. Mình là Tech Lead — set rules, review, approve."

### 2. Context Engineering là kỹ năng quan trọng nhất
> "Prompt tốt quan trọng, nhưng CONTEXT SYSTEM tốt quan trọng hơn. CLAUDE.md + AGENTS.md = ROI cao nhất."

### 3. Bắt đầu nhỏ, scale dần
> "Đừng cố để AI build cả feature ngay. Bắt đầu với: viết tests, review code, generate docs."

---

## 🎯 Proposed Next Steps

| Action | Ai | Timeline |
|--------|-----|----------|
| Tạo CLAUDE.md cho BookWormHub | Nghĩa | ✅ Done |
| Setup pre-commit guardrails | Nghĩa | ✅ Done |
| Team thử dùng AI viết tests | Cả team | Tuần tới |
| Setup GitHub Action AI review | Nghĩa | Tuần tới |
| Chia sẻ learnings & iterate | Cả team | Biweekly |

---

## 📎 Tài Liệu Kèm Theo

1. [CLAUDE.md template](CLAUDE.md)
2. [Edge-case Test Report](edge-case-tests-report.md)
3. [Code Quality Guardrails](code-quality-guardrails.md)
4. [Khóa học AI Coder — Full notes](../README.md)
