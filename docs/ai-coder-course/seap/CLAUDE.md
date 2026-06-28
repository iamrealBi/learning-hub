# CLAUDE.md — BookWormHub

## Project Overview
BookWormHub là ứng dụng web quản lý và đánh giá sách.
Stack: ASP.NET Core 8 MVC + Entity Framework Core (SQLite) + FluentValidation

## Build Commands
```bash
dotnet build
dotnet run --project BookWormHub
dotnet test BookWormHub.Tests
dotnet format
```

## Architecture
- **Service Layer Pattern**: Mọi business logic trong Services/
- **POCO Models**: Models/ chứa entities thuần túy (no logic)
- **ViewModel Pattern**: ViewModels/ cho data binding
- **Interface-based DI**: Services/Interfaces/ cho dependency injection
- **FluentValidation**: Validators/ cho input validation
- **ServiceResult Pattern**: Return type chuẩn cho service operations

## Key Entities
- Book (Id, Title, Author, ISBN13, Genre, Description, PublishedYear)
- Review (Id, Rating, Comment, Status[Approved/Hidden/Rejected], UserId, BookId)
- BannedWord (Id, Word, CreatedAt)
- ApplicationUser (extends IdentityUser + IsCritic, CrticSince)

## Services & Responsibilities
- BookService: CRUD sách, search, filter, ISBN validation
- ReviewService: CRUD reviews, moderation integration, badge trigger
- AdminService: Banned words management, review approval/rejection, badge revocation
- ModerationService: Check text for banned words (case-insensitive, substring match)
- BadgeService: Auto-award Critic badge when user reaches 10 approved reviews
- HomeService: Dashboard statistics

## Code Standards
- File-scoped namespaces
- Async/await for ALL database operations
- Naming: MethodNameAsync() for async methods
- Vietnamese messages cho user-facing strings
- Use `var` when type is obvious from context
- String interpolation: $"Hello {name}"

## Testing
- Framework: xUnit + FluentAssertions
- Database: InMemory (TestDbContextFactory)
- Naming: MethodName_Condition_ExpectedResult
- Pattern: Arrange → Act → Assert
- Location: BookWormHub.Tests/

## IMPORTANT RULES
- DO NOT modify existing Models without explicit permission
- DO NOT delete existing tests
- ALWAYS create interface for new services
- ALWAYS write unit tests for new business logic
- ALWAYS run `dotnet test` after changes
- ALWAYS use ServiceResult pattern for return values
- NEVER put business logic in Controllers
- NEVER use DataAnnotations (use FluentValidation instead)
