# AGENTS.md — BookWormHub

## Project Overview
BookWormHub - Web application for book management and review system.
Built with ASP.NET Core 8 MVC + Entity Framework Core (SQLite) + FluentValidation.

## Tech Stack
- .NET 8 / C# 12
- ASP.NET Core MVC
- Entity Framework Core (SQLite provider)
- FluentValidation
- xUnit + FluentAssertions (Testing)
- ASP.NET Core Identity (Authentication)

## Architecture
- **Service Layer Pattern**: ALL business logic in Services/
- **POCO Models**: Models/ contains pure data entities only
- **ViewModel Pattern**: ViewModels/ for view data binding
- **Interface-based DI**: Services/Interfaces/ for contracts
- **FluentValidation**: Validators/ for input validation
- **ServiceResult Pattern**: Standardized return type for operations

## Build & Run
```bash
dotnet build
dotnet run --project BookWormHub
dotnet test BookWormHub.Tests
dotnet format
```

## Project Structure
```
BookWormHub/
├── Models/           # POCO entities (Book, Review, BannedWord, ApplicationUser)
├── Services/         # Business logic
│   └── Interfaces/   # Service contracts
├── Validators/       # FluentValidation validators
├── ViewModels/       # View data transfer objects
├── Controllers/      # MVC controllers (thin, route + call service)
├── Data/             # AppDbContext
├── Views/            # Razor views
└── Migrations/       # EF Core migrations

BookWormHub.Tests/
├── Services/         # Service unit tests
├── Validators/       # Validator tests
└── Helpers/          # TestDbContextFactory
```

## Code Standards
- File-scoped namespaces
- Async methods must end with "Async" suffix
- Vietnamese messages for all user-facing strings
- Use `var` when type is obvious
- String interpolation over concatenation
- Constructor injection only (no property injection)

## RULES
- NEVER put business logic in Controllers
- NEVER use DataAnnotations (use FluentValidation)
- NEVER throw exceptions for flow control (use ServiceResult.Fail)
- ALWAYS create interface for new services
- ALWAYS register DI in Program.cs
- ALWAYS write unit tests for new service methods
- ALWAYS run `dotnet test` after changes
