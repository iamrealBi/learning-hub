# 🔍 Edge-Case Test Report — BookWormHub

> Phân tích tests hiện có và đề xuất edge cases bổ sung
> Dựa trên khóa học "AI Coder" — Enhanced Test Automation bằng AI

---

## 1. Phân Tích Tests Hiện Có

### BookServiceTests.cs (8 tests):
| Test | Coverage |
|------|----------|
| GetBookListAsync_ReturnsAllBooks | ✅ Happy path |
| GetBookListAsync_SearchFiltersResults | ✅ Search |
| GetBookListAsync_GenreFiltersResults | ✅ Genre filter |
| GetBookDetailsAsync_ReturnsNull_WhenNotFound | ✅ Not found |
| CreateBookAsync_Success | ✅ Happy path |
| CreateBookAsync_DuplicateISBN_ReturnsError | ✅ Duplicate |
| CreateBookAsync_InvalidChecksum_ReturnsError | ✅ Invalid ISBN |
| DeleteBookAsync_Success | ✅ Happy path |
| IsValidISBN13 (Theory × 4) | ✅ ISBN validation |

### ReviewServiceTests.cs: ✅ Có coverage cơ bản
### AdminServiceTests.cs: ✅ Có coverage cơ bản
### ModerationServiceTests.cs: ✅ Có coverage cơ bản
### BookCreateValidatorTests.cs: ✅ Có coverage cơ bản

---

## 2. ⚠️ Edge Cases THIẾU — Cần Bổ Sung

### 2.1 BookService — Edge Cases

```csharp
// ❌ THIẾU: Search với ký tự đặc biệt
[Fact]
public async Task GetBookListAsync_SearchWithSpecialChars_HandlesGracefully()
{
    var sut = CreateService(out var db);
    db.Books.Add(new Book { Title = "C# Programming", Author = "Auth", 
        ISBN13 = "9780132350884", Genre = "Programming" });
    await db.SaveChangesAsync();

    // Ký tự đặc biệt trong search
    var result = await sut.GetBookListAsync("C#", null);
    result.Books.Should().HaveCount(1);
}

// ❌ THIẾU: Search empty string vs null  
[Theory]
[InlineData("")]
[InlineData("   ")]
[InlineData(null)]
public async Task GetBookListAsync_EmptyOrNullSearch_ReturnsAllBooks(string? search)
{
    var sut = CreateService(out var db);
    db.Books.Add(new Book { Title = "A", Author = "X", 
        ISBN13 = "9780132350884", Genre = "Fiction" });
    db.Books.Add(new Book { Title = "B", Author = "Y", 
        ISBN13 = "9781234567897", Genre = "Science" });
    await db.SaveChangesAsync();

    var result = await sut.GetBookListAsync(search, null);
    result.Books.Should().HaveCount(2);
}

// ❌ THIẾU: Combined search + genre filter
[Fact]
public async Task GetBookListAsync_SearchAndGenreCombined_FiltersCorrectly()
{
    var sut = CreateService(out var db);
    db.Books.Add(new Book { Title = "Clean Code", Author = "Martin", 
        ISBN13 = "9780132350884", Genre = "Programming" });
    db.Books.Add(new Book { Title = "Clean Cooking", Author = "Chef", 
        ISBN13 = "9781234567897", Genre = "Cooking" });
    await db.SaveChangesAsync();

    var result = await sut.GetBookListAsync("Clean", "Programming");
    result.Books.Should().HaveCount(1);
    result.Books[0].Genre.Should().Be("Programming");
}

// ❌ THIẾU: GetBookDetailsAsync trả về correct avg rating
[Fact]
public async Task GetBookDetailsAsync_CalculatesCorrectAvgRating()
{
    var sut = CreateService(out var db);
    var book = new Book { Title = "Test", Author = "A", 
        ISBN13 = "9780132350884", Genre = "Fiction" };
    db.Books.Add(book);
    await db.SaveChangesAsync();

    db.Reviews.AddRange(
        new Review { BookId = book.Id, Rating = 5, UserId = "u1", 
            Status = ReviewStatus.Approved },
        new Review { BookId = book.Id, Rating = 3, UserId = "u2", 
            Status = ReviewStatus.Approved },
        new Review { BookId = book.Id, Rating = 1, UserId = "u3", 
            Status = ReviewStatus.Hidden } // Hidden should NOT count
    );
    await db.SaveChangesAsync();

    var result = await sut.GetBookDetailsAsync(book.Id, null);
    result!.AvgRating.Should().Be(4.0); // (5+3)/2, NOT (5+3+1)/3
    result.ReviewCount.Should().Be(2);  // Only approved
}

// ❌ THIẾU: UpdateBookAsync - Book không tồn tại
[Fact]
public async Task UpdateBookAsync_NonExistentBook_ReturnsError()
{
    var sut = CreateService(out _);
    var model = new BookEditViewModel { Id = 999, Title = "Test", 
        Author = "A", ISBN13 = "9780132350884" };

    var result = await sut.UpdateBookAsync(999, model);
    result.Success.Should().BeFalse();
}

// ❌ THIẾU: UpdateBookAsync - ID mismatch
[Fact]
public async Task UpdateBookAsync_IdMismatch_ReturnsError()
{
    var sut = CreateService(out _);
    var model = new BookEditViewModel { Id = 1, Title = "Test", 
        Author = "A", ISBN13 = "9780132350884" };

    var result = await sut.UpdateBookAsync(2, model); // ID mismatch!
    result.Success.Should().BeFalse();
}

// ❌ THIẾU: DeleteBookAsync - Xóa sách không tồn tại
[Fact]
public async Task DeleteBookAsync_NonExistentBook_ReturnsError()
{
    var sut = CreateService(out _);
    var result = await sut.DeleteBookAsync(999);
    result.Success.Should().BeFalse();
}

// ❌ THIẾU: CreateBookAsync - Empty ISBN (null/empty handling)
[Fact]
public async Task CreateBookAsync_EmptyISBN_HandlesCorrectly()
{
    var sut = CreateService(out var db);
    var model = new BookCreateViewModel { Title = "No ISBN Book", 
        Author = "Auth", ISBN13 = "", Genre = "Fiction" };

    // Behavior depends on business rules - 
    // currently code skips validation for empty ISBN
    var result = await sut.CreateBookAsync(model);
    // Verify expected behavior
}
```

### 2.2 ReviewService — Edge Cases

```csharp
// ❌ THIẾU: Duplicate review → update thay vì tạo mới
[Fact]
public async Task CreateOrUpdateReviewAsync_ExistingReview_UpdatesInsteadOfCreate()
{
    // ... setup with existing review
    // Call CreateOrUpdate with same userId + bookId
    // Assert: update existing, NOT create new
}

// ❌ THIẾU: Review với banned word → Status = Hidden
[Fact]
public async Task CreateOrUpdateReviewAsync_BannedWord_SetsStatusHidden()
{
    // ... setup banned word "badword"
    // Create review with comment containing "badword"
    // Assert: Status == ReviewStatus.Hidden
}

// ❌ THIẾU: Badge NOT awarded when review is Hidden
[Fact]
public async Task CreateOrUpdateReviewAsync_HiddenReview_DoesNotAwardBadge()
{
    // User has 9 approved reviews
    // Create 10th review with banned word → Hidden
    // Assert: user.IsCritic == false (still not critic)
}
```

### 2.3 ModerationService — Edge Cases

```csharp
// ❌ THIẾU: Case insensitive matching
[Fact]
public async Task ContainsBannedWord_CaseInsensitive_DetectsWord()
{
    // Banned word: "spam"
    // Text: "This is SPAM content"
    // Assert: returns true
}

// ❌ THIẾU: Substring matching (có thể gây false positive)
[Fact]
public async Task ContainsBannedWord_SubstringMatch_MayFalsePositive()
{
    // Banned word: "ass"
    // Text: "This is a classic book" (contains "ass" in "classic")
    // ⚠️ Current implementation WILL flag this as banned!
    // This is a known limitation worth documenting
}

// ❌ THIẾU: Empty banned words list
[Fact]
public async Task ContainsBannedWord_EmptyBannedList_ReturnsFalse()
{
    // No banned words in DB
    // Any text → should return false
}
```

### 2.4 BadgeService — Edge Cases

```csharp
// ❌ THIẾU: User already a critic → skip
[Fact]
public async Task CheckAndAwardBadge_AlreadyCritic_DoesNothing()
{
    // User already has IsCritic = true
    // Even with 20 reviews
    // Should not overwrite CrticSince date
}

// ❌ THIẾU: User exactly at threshold
[Fact]
public async Task CheckAndAwardBadge_ExactlyAtThreshold_AwardsBadge()
{
    // User has exactly 10 approved reviews
    // Assert: IsCritic = true
}

// ❌ THIẾU: User below threshold
[Fact]
public async Task CheckAndAwardBadge_BelowThreshold_DoesNotAward()
{
    // User has 9 approved reviews
    // Assert: IsCritic = false
}
```

---

## 3. Tổng Kết

### Thống kê:
| Metric | Số lượng |
|--------|---------|
| Tests hiện có | ~38 |
| Edge cases thiếu phát hiện | ~18+ |
| Độ coverage ước tính hiện tại | ~65% |
| Độ coverage mục tiêu | ~90% |

### Priority Matrix:

| Priority | Edge Cases | Lý do |
|----------|-----------|-------|
| 🔴 **Cao** | Avg Rating chỉ tính Approved, Badge threshold, Banned word false positive | Logic lỗi tiềm ẩn |
| 🟡 **Trung bình** | Combined search+filter, Update non-existent, ID mismatch | Error handling |
| 🟢 **Thấp** | Special chars in search, Empty banned list | Edge cases hiếm gặp |

### Hướng dẫn tiếp theo:
1. Copy các test code ở trên vào BookWormHub.Tests/
2. Chạy `dotnet test` để xác nhận
3. Sửa code nếu tests reveal bugs thật
4. Cập nhật test count trong email report tiếp theo
