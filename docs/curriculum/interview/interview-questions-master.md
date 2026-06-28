# 💼 C# Interview Questions — Master Compilation

> **75+ câu hỏi phỏng vấn** xếp theo chủ đề và mức độ khó

---

## 📋 Mục Lục

| Chủ đề | Số câu | File chi tiết |
|--------|--------|---------------|
| Fundamentals | 15 | [01-fundamentals.md](01-fundamentals.md) |
| OOP | 15 | [02-oop.md](02-oop.md) |
| Exceptions | 4 | [03-exceptions.md](03-exceptions.md) |
| Generics | 4 | [04-generics.md](04-generics.md) |
| LINQ | 3 | [05-linq.md](05-linq.md) |
| .NET Internals | 4 | [06-dotnet-internals.md](06-dotnet-internals.md) |
| Advanced Types | 5 | [07-advanced-types.md](07-advanced-types.md) |
| Collections | 5 | [08-collections.md](08-collections.md) |
| Strings & Projects | 4 | [09-projects-strings.md](09-projects-strings.md) |
| Events | 4 | [10-numerics-events.md](10-numerics-events.md) |
| Testing | 4 | [11-unit-testing.md](11-unit-testing.md) |
| Clean Code | 4 | [12-clean-code.md](12-clean-code.md) |
| Async | 6 | [13-async.md](13-async.md) |

---

## ⭐ Top 20 — Câu Hỏi HAY GẶP NHẤT

| # | Câu hỏi | Đáp án ngắn |
|---|---------|-------------|
| 1 | OOP 4 pillars? | Abstraction, Encapsulation, Inheritance, Polymorphism |
| 2 | `abstract class` vs `interface`? | Abstract: shared code, single inherit. Interface: contract, multi-implement |
| 3 | `struct` vs `class`? | Value type (stack) vs Reference type (heap) |
| 4 | SOLID principles? | SRP, OCP, LSP, ISP, DIP |
| 5 | `async/await` — how it works? | Giải phóng thread khi chờ I/O, state machine |
| 6 | Garbage Collector? | Mark-and-Sweep, 3 Generations, tự động quản lý Heap memory |
| 7 | `Func` vs `Action`? | Func có return, Action void |
| 8 | LINQ deferred execution? | Query chỉ chạy khi enumerate (foreach/ToList) |
| 9 | `string` immutability — why? | Thread safety, interning, security |
| 10 | `==` vs `.Equals()`? | Reference vs Value comparison (depends on override) |
| 11 | Dependency Injection? | Truyền dependency từ ngoài qua constructor/property/method |
| 12 | `throw` vs `throw ex`? | `throw` giữ stack trace, `throw ex` reset |
| 13 | Deadlock — what & prevention? | 2 threads chờ nhau. Lock ordering, timeout, avoid nesting |
| 14 | Mock vs Stub? | Stub = provide data, Mock = verify behavior |
| 15 | Value type vs Reference type? | Stack vs Heap, copy value vs copy reference |
| 16 | `IEnumerable` vs `IQueryable`? | In-memory vs Remote (SQL translation) |
| 17 | `yield` keyword? | Lazy iterator, trả về từng phần tử khi cần |
| 18 | Composition vs Inheritance? | HAS-A vs IS-A, favor composition for flexibility |
| 19 | Why avoid `async void`? | Can't catch exceptions, can't await |
| 20 | Boxing/Unboxing? | Value→Object (Heap), Object→Value (Stack), performance cost |

---

## ⭐⭐ Top 20 — Câu Hỏi TRUNG BÌNH

| # | Câu hỏi | Chủ đề |
|---|---------|--------|
| 1 | Covariance vs Contravariance? | Generics |
| 2 | `ConfigureAwait(false)` — when? | Async |
| 3 | What are Records (C# 9+)? | Types |
| 4 | `GetHashCode` contract? | Types |
| 5 | `Dispose` vs `Finalize`? | Memory |
| 6 | Memory leaks in C# — possible? | Memory |
| 7 | Generic constraints — types? | Generics |
| 8 | Strategy Pattern? | Patterns |
| 9 | Template Method Pattern? | Patterns |
| 10 | Observer Pattern? | Events |
| 11 | Events cause memory leaks? | Events |
| 12 | TDD — Red/Green/Refactor? | Testing |
| 13 | Code coverage — 100% good? | Testing |
| 14 | What makes code untestable? | Testing |
| 15 | `Select` vs `SelectMany`? | LINQ |
| 16 | `Task.Wait()` vs `await`? | Async |
| 17 | `WhenAll` vs `WhenAny`? | Async |
| 18 | Exception filters (`when`)? | Exceptions |
| 19 | Code smells — examples? | Clean Code |
| 20 | Shallow copy vs Deep copy? | Objects |

---

## ⭐⭐⭐ Top 15 — Câu Hỏi NÂNG CAO

| # | Câu hỏi | Chủ đề |
|---|---------|--------|
| 1 | Diamond problem — C# solution? | Inheritance |
| 2 | Expression trees — what? | LINQ |
| 3 | How does GC handle LOH? | Memory |
| 4 | `SemaphoreSlim` vs `lock`? | Threading |
| 5 | `ConcurrentDictionary` — thread-safe how? | Collections |
| 6 | `Span<T>` and `Memory<T>`? | Performance |
| 7 | JIT vs AOT compilation? | .NET Runtime |
| 8 | What is reflection? Performance cost? | Reflection |
| 9 | Middleware pipeline pattern? | ASP.NET Core |
| 10 | What is a closure in C#? | Lambda |
| 11 | `volatile` keyword? | Threading |
| 12 | IDisposable pattern — full implementation? | Memory |
| 13 | Value task vs Task? | Async |
| 14 | Weak references — when? | Memory |
| 15 | Source generators? | C# Compiler |

---

## 🎯 Tips Phỏng Vấn

### Chuẩn bị

1. **Hiểu sâu, không học vẹt** — interviewer thường hỏi follow-up
2. **Cho ví dụ code** — "Can you write an example?" rất phổ biến
3. **Nói pros/cons** — "When would you NOT use this?"
4. **Admit unknowns** — "I'm not sure, but I think..." tốt hơn đoán sai

### Trong phỏng vấn

1. **Clarify requirements** trước khi code
2. **Think out loud** — giải thích reasoning
3. **Start simple**, rồi optimize
4. **Handle edge cases** (null, empty, overflow)
5. **Name variables meaningfully** — clean code matters

### Common Follow-ups

- "Can you give a real-world example?"
- "What are the trade-offs?"
- "How would you test this?"
- "What if the data is very large?"
- "How would you make this thread-safe?"
