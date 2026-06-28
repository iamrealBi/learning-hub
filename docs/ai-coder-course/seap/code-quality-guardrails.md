# 🛡️ Code Quality Guardrails — BookWormHub

> Structured AI Prompts hoạt động như Pre-Commit Reviewer
> Enforce: Service Layer, FluentValidation, POCO Patterns

---

## 1. Pre-Commit Review Prompt

Copy prompt này và dùng trước mỗi lần commit:

```
Hãy review code changes sau đây cho BookWormHub theo checklist:

## Architecture Compliance
- [ ] Business logic CHỈ trong Services/ (không trong Controllers/Models)
- [ ] Controller methods CHỈ gọi service → return view
- [ ] Models là POCO thuần túy (không có methods, không có logic)
- [ ] Mỗi service mới CÓ interface trong Services/Interfaces/

## Pattern Compliance  
- [ ] Return type là ServiceResult hoặc ServiceResult<T>
- [ ] Input validation dùng FluentValidation (KHÔNG dùng DataAnnotations)
- [ ] Dependency injection qua constructor (KHÔNG property injection)
- [ ] Async/await cho TẤT CẢ database operations

## Code Quality
- [ ] File-scoped namespaces
- [ ] Method names có suffix "Async" cho async methods
- [ ] Không có try/catch không cần thiết (dùng ServiceResult.Fail)
- [ ] Vietnamese messages cho user-facing strings
- [ ] Không có magic numbers/strings (dùng constants)

## Testing
- [ ] Unit tests cho mọi public service method mới
- [ ] Test cả happy path VÀ error path
- [ ] Sử dụng FluentAssertions
- [ ] Test naming: MethodName_Condition_Expected

Code cần review:
[PASTE DIFF Ở ĐÂY]
```

---

## 2. AGENTS.md Pre-Commit Hook

Đặt file này ở root BookWormHub để AI tự enforce rules:

```markdown
# AGENTS.md — BookWormHub Code Standards

## MANDATORY CHECKS (Hooks)
Before any code change, verify:

### ❌ NEVER DO
- Put business logic in Controllers
- Use DataAnnotations for validation
- Modify Models without creating migration
- Delete existing tests
- Return raw exceptions (use ServiceResult)
- Use property injection

### ✅ ALWAYS DO
- Create interface for every new service
- Register DI in Program.cs
- Write unit tests (xUnit + FluentAssertions)
- Use async/await for database calls
- Return ServiceResult from service methods
- Run `dotnet test` before committing
```

---

## 3. Automated Review Script

### PowerShell script chạy trước commit:

```powershell
# pre-commit-review.ps1
# Chạy AI review trên staged changes

$diff = git diff --cached --diff-filter=ACMR -- "*.cs"

if ([string]::IsNullOrWhiteSpace($diff)) {
    Write-Host "No C# files staged." -ForegroundColor Yellow
    exit 0
}

Write-Host "🔍 Running AI Code Review..." -ForegroundColor Cyan

# Kiểm tra patterns vi phạm
$violations = @()

# Check 1: Business logic trong Controller
if ($diff -match "class.*Controller.*\{" -and $diff -match "_db\.|DbContext") {
    $violations += "⚠️ Possible DB access in Controller (should be in Service)"
}

# Check 2: DataAnnotations thay vì FluentValidation
if ($diff -match "\[Required\]|\[MaxLength\]|\[StringLength\]") {
    $violations += "⚠️ DataAnnotations detected (use FluentValidation instead)"
}

# Check 3: Missing async
if ($diff -match "\.ToList\(\)" -and $diff -notmatch "\.ToListAsync\(\)") {
    $violations += "⚠️ Possible sync DB call (use ToListAsync)"
}

# Check 4: Missing interface
if ($diff -match "class \w+Service" -and $diff -notmatch "interface I\w+Service") {
    $violations += "⚠️ New service without interface detected"
}

if ($violations.Count -gt 0) {
    Write-Host "`n🚨 CODE REVIEW FINDINGS:" -ForegroundColor Red
    $violations | ForEach-Object { Write-Host "  $_" -ForegroundColor Yellow }
    Write-Host "`nReview and fix before committing." -ForegroundColor Red
    exit 1
} else {
    Write-Host "✅ All checks passed!" -ForegroundColor Green
    exit 0
}
```

### Cách sử dụng:
```powershell
# Chạy thủ công
.\pre-commit-review.ps1

# Hoặc thiết lập Git hook
# Copy vào .git/hooks/pre-commit
```

---

## 4. Continuous AI Review trong CI/CD

### GitHub Action cho automatic review:

```yaml
# .github/workflows/ai-code-review.yml
name: AI Code Quality Review

on:
  pull_request:
    types: [opened, synchronize]
    paths: ['**.cs']

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Get changed files
        id: changed
        run: |
          echo "files=$(git diff --name-only origin/main...HEAD -- '*.cs' | tr '\n' ' ')" >> $GITHUB_OUTPUT
          
      - name: Check patterns
        run: |
          VIOLATIONS=""
          
          for file in ${{ steps.changed.outputs.files }}; do
            # Check for DataAnnotations
            if grep -n "\[Required\]\|\[MaxLength\]\|\[StringLength\]" "$file" 2>/dev/null; then
              VIOLATIONS="${VIOLATIONS}⚠️ ${file}: Uses DataAnnotations instead of FluentValidation\n"
            fi
            
            # Check for sync DB calls
            if grep -n "\.ToList()\|\.FirstOrDefault()\|\.Find(" "$file" 2>/dev/null | grep -v "Async"; then
              VIOLATIONS="${VIOLATIONS}⚠️ ${file}: Possible synchronous DB call\n"  
            fi
          done
          
          if [ -n "$VIOLATIONS" ]; then
            echo "🚨 Code Quality Issues Found:"
            echo -e "$VIOLATIONS"
            exit 1
          fi
          
      - name: Run tests
        run: dotnet test BookWormHub.Tests --verbosity normal
```

---

## 5. Tóm Tắt: Guard Rails Stack

```
Layer 1: AGENTS.md / CLAUDE.md     → AI biết rules
Layer 2: pre-commit-review.ps1     → Kiểm tra local
Layer 3: GitHub Action              → Kiểm tra trên CI
Layer 4: Human PR Review            → Team review final
```

Mỗi layer bắt những lỗi mà layer trước bỏ sót → **defense in depth**.
