# 🚀 ASP.NET Core Ultimate Guide — Zero to Hero
## Kết hợp kiến thức Coursera + YouTube (Học lập trình cùng Nam)

---

Tài liệu này tổng hợp và kết hợp kiến thức từ **hai nguồn chính**:

| Nguồn | Nội dung | Phong cách |
|---|---|---|
| **Coursera** — Web Application Development with ASP.NET Core | 4 Module: Foundations, Web API, EF Core, Auth | Zero-to-hero, ẩn dụ đời thường |
| **YouTube** — Học lập trình cùng Nam (46 videos) | 8 Module: Từ Controller đến Docker | Technical deep-dive, demo code |

Mỗi Module trong guide này là **tự chứa hoàn chỉnh** — không cần file bổ sung.

---

## 📚 Danh sách Module

| # | Module | Chủ đề chính |
|---|---|---|
| 01 | [Nền Tảng ASP.NET Core](Module-01-Nen-Tang-ASP-NET-Core.md) | Web basics, .NET history, MVC, Controller, DI, Routing, HttpContext, Razor, Validation |
| 02 | [HTTP & Middleware](Module-02-HTTP-Middleware.md) | Pipeline, HttpRequest/Response, Cookie/Session, Configuration, Options Pattern, Custom Middleware |
| 03 | [Routing & Model Binding](Module-03-Routing-Model-Binding.md) | Route constraints, Area routing, Model Binding edge cases, Distributed Session, HTTPS/HSTS |
| 04 | [Web API & RESTful](Module-04-Web-API-REST.md) | REST principles, CRUD, Status Codes, CORS, Rate Limiting, Caching (In-Memory, Redis, Output), Swagger, Testing |
| 05 | [Entity Framework Core](Module-05-Entity-Framework-Core.md) | ORM, Code-First, Migrations, Relationships, LINQ, Repository, Unit of Work, Performance, Seeding |
| 06 | [Authentication & Authorization](Module-06-Authentication-Authorization.md) | Cookie Auth, JWT, Identity, 2FA, Role/Policy-based AuthZ, OAuth 2.0, OpenID Connect, XSS/CSRF |
| 07 | [Minimal API & .NET Aspire](Module-07-Minimal-API-Aspire.md) | Minimal API syntax, MapGroup, .NET Aspire orchestration, Dashboard, Core Banking exercise |
| 08 | [Testing](Module-08-Testing.md) | xUnit, SQLite InMemory, Integration Tests, WebApplicationFactory, Mocking (Moq) |
| 09 | [Docker & Deployment](Module-09-Docker-Deployment.md) | Containers, Dockerfile, Docker Compose, Nginx Reverse Proxy, Load Balancing, Secrets |

---

## 🗺️ Lộ trình học gợi ý

```
Tuần 1-2: Module 01-02 (Nền tảng + HTTP/Middleware)
Tuần 3:   Module 03 (Routing & Model Binding)
Tuần 4:   Module 04 (Web API)
Tuần 5:   Module 05 (Entity Framework Core)
Tuần 6:   Module 06 (Authentication & Authorization)
Tuần 7:   Module 07 (Minimal API & Aspire)
Tuần 8:   Module 08 (Testing)
Tuần 9:   Module 09 (Docker & Deployment)
```

> 💡 **Mẹo**: Đọc lý thuyết → chạy code ví dụ → làm bài tập cuối module → sang module tiếp.

---

## 📁 Nguồn gốc

- `../aspnet-core-study-guide/` — Coursera modules
- `../youtube-aspnet-guide/` — YouTube modules (46 video transcripts)
