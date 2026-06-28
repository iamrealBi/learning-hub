# 📙 Phần 13: Multithreading & Asynchrony

> **Nội dung**: Threads, Tasks, async/await, cancellation, synchronization, advanced patterns  
> **Thời lượng ước tính**: 6–8 giờ

---

## 🎯 Mục Tiêu

- Hiểu Thread, Process, Concurrency vs Parallelism
- Sử dụng Task Parallel Library (TPL)
- Thành thạo async/await
- Xử lý exceptions trong async code
- Synchronization (Race condition, Locks)
- Cancellation Token

---

## 1. Khái Niệm Cơ Bản

### 1.1 Process vs Thread

```
Process (Tiến trình):
  - 1 ứng dụng đang chạy
  - Có bộ nhớ RIÊNG (isolated)
  - Ví dụ: Chrome, Visual Studio, mỗi cái là 1 process

Thread (Luồng):
  - 1 đơn vị thực thi TRONG process
  - CHIA SẺ bộ nhớ với các thread khác trong cùng process
  - 1 process có thể có NHIỀU threads
```

### 1.2 Concurrency vs Parallelism

```
Concurrency (Đồng thời):
  - Nhiều tasks TIẾN TRIỂN cùng lúc
  - Có thể trên 1 CPU core (time-slicing)
  - Ví dụ: Bạn nấu ăn + giặt đồ → chuyển qua lại giữa 2 việc

Parallelism (Song song):
  - Nhiều tasks THỰC SỰ chạy cùng lúc
  - Cần nhiều CPU cores
  - Ví dụ: 2 người nấu ăn + giặt đồ cùng lúc

Asynchrony (Bất đồng bộ):
  - KHÔNG CHỜ task hoàn thành mới làm tiếp
  - Ví dụ: Gửi email → làm việc khác → email về → xử lý
```

---

## 2. Thread Class

```csharp
// Tạo thread mới
Thread thread = new Thread(() =>
{
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine($"Thread: {i}");
        Thread.Sleep(500);
    }
});

thread.Start();   // Bắt đầu chạy

// Main thread tiếp tục chạy song song
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Main: {i}");
    Thread.Sleep(300);
}

thread.Join();  // Chờ thread kết thúc
```

### ThreadPool

```csharp
// ThreadPool: tái sử dụng threads (hiệu quả hơn tạo mới)
ThreadPool.QueueUserWorkItem(_ =>
{
    Console.WriteLine("Running on ThreadPool thread");
});

// ⚠️ Chi phí tạo Thread:
// - Mỗi thread ~ 1MB stack memory
// - Context switching có chi phí
// → Dùng ThreadPool hoặc Task thay vì Thread trực tiếp
```

---

## 3. Task Parallel Library (TPL)

### 3.1 Task Class

```csharp
// Task: đơn vị công việc bất đồng bộ (xây trên ThreadPool)
Task task = Task.Run(() =>
{
    Console.WriteLine("Running on background thread");
    Thread.Sleep(2000);
    Console.WriteLine("Done!");
});

// Chờ task hoàn thành
task.Wait();  // Blocking wait (chặn thread hiện tại)

// Task<T>: task trả về kết quả
Task<int> calcTask = Task.Run(() =>
{
    Thread.Sleep(1000);
    return 42;
});

int result = calcTask.Result;  // Blocking wait + lấy kết quả
```

### 3.2 Waiting

```csharp
Task t1 = Task.Run(() => DoWork1());
Task t2 = Task.Run(() => DoWork2());
Task t3 = Task.Run(() => DoWork3());

Task.WaitAll(t1, t2, t3);    // Chờ TẤT CẢ hoàn thành
Task.WaitAny(t1, t2, t3);    // Chờ BẤT KỲ 1 task hoàn thành
```

### 3.3 Continuations

```csharp
// ContinueWith: chạy task tiếp sau khi task trước hoàn thành
Task<string> fetchTask = Task.Run(() =>
{
    Thread.Sleep(1000);
    return "data from server";
});

Task processTask = fetchTask.ContinueWith(previousTask =>
{
    string data = previousTask.Result;
    Console.WriteLine($"Processing: {data}");
});

// Chaining continuations
Task.Run(() => FetchData())
    .ContinueWith(t => ProcessData(t.Result))
    .ContinueWith(t => SaveResult(t.Result))
    .ContinueWith(t => Console.WriteLine("All done!"));
```

---

## 4. Cancellation

```csharp
// CancellationTokenSource: điều khiển cancellation
var cts = new CancellationTokenSource();
CancellationToken token = cts.Token;

Task longTask = Task.Run(() =>
{
    for (int i = 0; i < 1000; i++)
    {
        // Kiểm tra có bị cancel không
        token.ThrowIfCancellationRequested();
        
        // Hoặc kiểm tra thủ công
        if (token.IsCancellationRequested)
        {
            Console.WriteLine("Cancelled!");
            return;
        }
        
        Thread.Sleep(100);
        Console.WriteLine($"Working... {i}");
    }
}, token);

// Sau 3 giây, cancel task
await Task.Delay(3000);
cts.Cancel();

try
{
    await longTask;
}
catch (OperationCanceledException)
{
    Console.WriteLine("Task was cancelled");
}
```

---

## 5. Synchronization

### 5.1 Race Condition

```csharp
// ❌ Race condition: kết quả phụ thuộc thứ tự thực thi
int counter = 0;

Task t1 = Task.Run(() => { for (int i = 0; i < 100000; i++) counter++; });
Task t2 = Task.Run(() => { for (int i = 0; i < 100000; i++) counter++; });

Task.WaitAll(t1, t2);
Console.WriteLine(counter);  // KHÔNG phải 200000! (ví dụ: 158723)
```

### 5.2 Lock

```csharp
// lock: chỉ 1 thread được vào critical section tại 1 thời điểm
object lockObj = new object();
int counter = 0;

Task t1 = Task.Run(() =>
{
    for (int i = 0; i < 100000; i++)
    {
        lock (lockObj)
        {
            counter++;  // Chỉ 1 thread thực hiện tại 1 thời điểm
        }
    }
});

Task t2 = Task.Run(() =>
{
    for (int i = 0; i < 100000; i++)
    {
        lock (lockObj)
        {
            counter++;
        }
    }
});

Task.WaitAll(t1, t2);
Console.WriteLine(counter);  // 200000 ✅
```

### 5.3 Atomic Operations

```csharp
// Interlocked: atomic operations (không cần lock)
int counter = 0;
Interlocked.Increment(ref counter);   // Tăng 1 (thread-safe)
Interlocked.Decrement(ref counter);   // Giảm 1
Interlocked.Add(ref counter, 5);      // Cộng 5
```

---

## 6. async/await

### 6.1 Basics

```csharp
// async: đánh dấu method là bất đồng bộ
// await: chờ task KHÔNG CHẶN thread

async Task<string> FetchDataAsync(string url)
{
    using HttpClient client = new();
    
    // await: thread được GIẢI PHÓNG trong khi chờ response
    string response = await client.GetStringAsync(url);
    
    // Code sau await chạy KHI response về
    return response;
}

// Gọi async method
string data = await FetchDataAsync("https://api.example.com/data");
Console.WriteLine(data);
```

### 6.2 async Method Signatures

```csharp
// Return type của async methods:
async Task DoSomethingAsync()        // void → Task
{
    await Task.Delay(1000);
}

async Task<int> GetNumberAsync()     // T → Task<T>
{
    await Task.Delay(1000);
    return 42;
}

// ❌ TRÁNH async void (ngoại trừ event handlers)
// async void DoSomething()  // Không catch được exception!
```

### 6.3 Asynchrony vs Multithreading

```
Async ≠ Multithreading!

Multithreading: tạo thread MỚI để chạy CPU-bound work
  → Thread.Sleep, Task.Run(() => HeavyCalculation())

Asynchrony: giải phóng thread KHI CHỜ I/O
  → await httpClient.GetAsync(), await File.ReadAllTextAsync()
  → Thread được trả về ThreadPool trong khi chờ
  → KHÔNG tạo thread mới!

Khi nào dùng gì?
  CPU-bound (tính toán nặng) → Task.Run + await
  I/O-bound (network, file)  → await async methods
```

### 6.4 Flow of an Async Program

```csharp
async Task MainAsync()
{
    Console.WriteLine("1. Start");
    
    Task<string> fetchTask = FetchDataAsync();
    Console.WriteLine("2. Fetch started (not awaited yet)");
    
    // Có thể làm việc khác trong khi fetch chạy
    DoOtherWork();
    Console.WriteLine("3. Other work done");
    
    string result = await fetchTask;  // Chờ ở đây
    Console.WriteLine($"4. Got result: {result}");
}
// Output: 1, 2, 3, 4 (3 có thể xen vào trước 4)
```

### 6.5 Running Tasks in Parallel

```csharp
// ✅ Run multiple async operations concurrently
async Task<(string, string, string)> FetchAllAsync()
{
    Task<string> task1 = FetchFromApi1Async();
    Task<string> task2 = FetchFromApi2Async();
    Task<string> task3 = FetchFromApi3Async();
    
    // Chờ TẤT CẢ cùng lúc (chạy song song)
    await Task.WhenAll(task1, task2, task3);
    
    return (task1.Result, task2.Result, task3.Result);
}

// Task.WhenAny: chờ task ĐẦU TIÊN hoàn thành
Task<string> fastest = await Task.WhenAny(task1, task2, task3);
```

---

## 7. Exception Handling in Async

```csharp
try
{
    string data = await FetchDataAsync("https://bad-url.com");
}
catch (HttpRequestException ex)
{
    Console.WriteLine($"Network error: {ex.Message}");
}
catch (TaskCanceledException)
{
    Console.WriteLine("Request timed out!");
}

// Multiple tasks: AggregateException
try
{
    await Task.WhenAll(task1, task2, task3);
}
catch (Exception ex)
{
    // Chỉ catch exception ĐẦU TIÊN
    // Để lấy TẤT CẢ exceptions:
    // task.Exception.InnerExceptions
}
```

---

## 8. HttpClient Best Practices

```csharp
// ✅ Tái sử dụng HttpClient (tạo 1 lần, dùng nhiều lần)
private static readonly HttpClient _client = new HttpClient();

async Task<List<Quote>> SearchQuotesAsync(string query)
{
    string url = $"https://api.quotable.io/search/quotes?query={query}";
    
    HttpResponseMessage response = await _client.GetAsync(url);
    response.EnsureSuccessStatusCode();
    
    string json = await response.Content.ReadAsStringAsync();
    return JsonSerializer.Deserialize<List<Quote>>(json);
}

// ❌ ĐỪNG tạo HttpClient mới cho mỗi request
// using var client = new HttpClient();  // ← Socket exhaustion!
```

---

## 9. Downsides of Multithreading

```
1. Race conditions → bugs khó reproduce
2. Deadlocks → chương trình đứng vĩnh viễn
3. Complex debugging → khó theo dõi flow
4. Context switching overhead
5. Memory overhead (mỗi thread ~ 1MB stack)
6. Code phức tạp hơn → khó bảo trì

→ Chỉ dùng khi THỰC SỰ cần:
  - I/O-bound: async/await (đơn giản)
  - CPU-bound: Task.Run (khi cần tận dụng nhiều cores)
```

---

## 🧪 Coding Exercises

| # | Bài | Kiến thức |
|---|-----|-----------|
| 63 | Creating and starting new threads | Thread class |
| 64 | Tasks & waiting | Task, WaitAll |
| 65 | Continuations | ContinueWith |
| 66 | Handling exceptions with continuations | AggregateException |
| 67 | Async/await | async, await, Task |

---

## ❓ Câu Hỏi Kiểm Tra

1. Process vs Thread — khác nhau thế nào?
2. Concurrency vs Parallelism — giải thích bằng ví dụ đời thực.
3. `Task.Run` vs `Thread` — cái nào nên dùng? Tại sao?
4. `await` làm gì? Thread có bị chặn không?
5. `async void` tại sao NGUY HIỂM?
6. Race condition là gì? Cho ví dụ code.
7. `lock` giải quyết vấn đề gì? Trade-off?
8. `CancellationToken` dùng khi nào?
9. `Task.WhenAll` vs `Task.WhenAny` — khác nhau?
10. Tại sao KHÔNG nên tạo `HttpClient` mới cho mỗi request?

---

## 💼 Câu Hỏi Phỏng Vấn

> **Q: What is the difference between `async/await` and multithreading?**
> A: `async/await`: giải phóng thread khi chờ I/O (network, file) — KHÔNG tạo thread mới. Multithreading: tạo thread cho CPU-bound work. Dùng async cho I/O-bound, Task.Run cho CPU-bound. Async = efficiency, threads = parallelism.

> **Q: What is a deadlock? How to prevent it?**
> A: 2 threads chờ nhau mãi — Thread A lock X, đợi Y; Thread B lock Y, đợi X. Prevent: (1) lock ordering, (2) timeout, (3) tránh nested locks, (4) dùng `Monitor.TryEnter`, (5) prefer async/await over blocking.

> **Q: Why should you avoid `async void`?**
> A: Can't await → can't catch exceptions → app crashes. Can't compose with Task.WhenAll. CHỈ dùng cho event handlers. Mọi async method khác → return `Task` hoặc `Task<T>`.

> **Q: What is `CancellationToken` and how to use it?**
> A: Mechanism để cancel operations gracefully. `CancellationTokenSource.Cancel()` signals cancellation. Method kiểm tra `token.ThrowIfCancellationRequested()` hoặc `token.IsCancellationRequested`. Truyền token qua parameter.

> **Q: Explain `Task.WhenAll` vs sequential awaits.**
> A: Sequential: `await A(); await B(); await C();` → tổng thời gian = A+B+C. WhenAll: `await Task.WhenAll(A(), B(), C());` → tổng thời gian = max(A, B, C). WhenAll chạy đồng thời, nhanh hơn nhiều khi I/O-bound.

---

## 🏋️ Bài Tập Thực Hành

**BT1**: Viết program download 5 URLs đồng thời bằng `Task.WhenAll`, so sánh thời gian vs download tuần tự.

**BT2**: Implement `AsyncSemaphore` giới hạn concurrent operations (ví dụ: max 3 downloads cùng lúc).

**BT3**: Tạo `BackgroundTaskQueue` — producer/consumer pattern: add tasks, process async, support cancellation.

**BT4**: Demo race condition: 2 tasks tăng counter 1 triệu lần. Fix bằng lock, Interlocked, và so sánh performance.

---

## 📚 Bổ Sung: Advanced Async Patterns

### ValueTask\<T\> — Tránh allocation khi kết quả có sẵn

```csharp
// Task<T> luôn allocate object trên heap
// ValueTask<T> = struct → KHÔNG allocate nếu kết quả đã sẵn sàng

// Ví dụ: cache hit → trả về ngay, cache miss → async
private Dictionary<string, User> _cache = new();

public ValueTask<User> GetUserAsync(string id)
{
    if (_cache.TryGetValue(id, out var user))
        return new ValueTask<User>(user);  // ✅ Không allocation!
    
    return new ValueTask<User>(LoadUserFromDbAsync(id));  // Async path
}

// Khi nào dùng ValueTask?
// ✅ Hot path: method gọi rất nhiều lần, thường return synchronously
// ✅ Cache lookups, pooled objects
// ❌ KHÔNG await nhiều lần cùng 1 ValueTask
// ❌ KHÔNG dùng .Result/.GetAwaiter() trước khi complete
```

### IAsyncEnumerable\<T\> — Async Streams (C# 8+)

```csharp
// IAsyncEnumerable: stream data async — yield từng item
// Giống IEnumerable nhưng mỗi item có thể await

async IAsyncEnumerable<int> GenerateNumbersAsync(int count)
{
    for (int i = 0; i < count; i++)
    {
        await Task.Delay(100);  // Simulate async work
        yield return i;
    }
}

// Consume bằng await foreach
await foreach (int n in GenerateNumbersAsync(10))
{
    Console.WriteLine(n);  // Mỗi item arrive từ từ
}

// Thực tế: stream API responses
async IAsyncEnumerable<Product> GetProductsAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    int page = 1;
    while (true)
    {
        var products = await _api.GetPageAsync(page, ct);
        if (!products.Any()) yield break;
        
        foreach (var p in products)
            yield return p;
        page++;
    }
}

// LINQ cho async streams (System.Linq.Async NuGet)
var expensive = await GetProductsAsync()
    .Where(p => p.Price > 100)
    .Take(10)
    .ToListAsync();
```

### Channel\<T\> — Producer/Consumer Pattern

```csharp
using System.Threading.Channels;

// Channel = thread-safe queue cho async producer/consumer

// Bounded channel (giới hạn capacity)
var channel = Channel.CreateBounded<string>(new BoundedChannelOptions(100)
{
    FullMode = BoundedChannelFullMode.Wait  // Producer đợi nếu đầy
});

// Producer
async Task ProduceAsync(ChannelWriter<string> writer)
{
    for (int i = 0; i < 1000; i++)
    {
        await writer.WriteAsync($"Message {i}");
        await Task.Delay(10);
    }
    writer.Complete();  // Báo hiệu xong
}

// Consumer
async Task ConsumeAsync(ChannelReader<string> reader)
{
    await foreach (var msg in reader.ReadAllAsync())
    {
        Console.WriteLine($"Received: {msg}");
    }
}

// Chạy song song
await Task.WhenAll(
    ProduceAsync(channel.Writer),
    ConsumeAsync(channel.Reader)
);

// Use cases:
// ✅ Background job processing
// ✅ Rate limiting API calls
// ✅ Pipeline processing (producer → transformer → consumer)
```

---

## ❓ Câu Hỏi Kiểm Tra Bổ Sung (Nâng Cao)

1. Thread vs Process — khác nhau?
2. Concurrency vs Parallelism — khác nhau?
3. async/await giải quyết vấn đề gì?
4. `Task.Run` vs `await` — khi nào dùng cái nào?
5. Race condition là gì? Cách phòng tránh?
6. `lock` vs `Interlocked` — khi nào dùng cái nào?
7. CancellationToken dùng để làm gì?
8. Tại sao KHÔNG nên dùng `async void`?

---

## 💼 Câu Hỏi Phỏng Vấn Bổ Sung (Nâng Cao)

> **Q: What is the difference between `async/await` and multithreading?**
> A: Async/await: giải phóng thread KHI CHỞ I/O (network, file). KHÔNG tạo thread mới. Thread pool thread được trả lại. Multithreading: tạo thread MỚI cho CPU-bound work. Async = I/O-bound. Multithreading = CPU-bound.

> **Q: What is a deadlock? How to prevent?**
> A: Deadlock: 2 threads chờ nhau giải phóng lock → cả 2 đứng mãi. Prevention: (1) Lock ordering — luôn lock cùng thứ tự, (2) Timeout — `Monitor.TryEnter`, (3) Avoid nested locks, (4) Dùng `SemaphoreSlim` thay lock.

> **Q: What is `ConfigureAwait(false)` and when do you use it?**
> A: Mặc định, await capture SynchronizationContext và resume trên thread gốc (UI thread). `ConfigureAwait(false)` skip context capture → resume trên bất kỳ thread pool thread. Dùng trong LIBRARY code (không cần UI thread). KHÔNG dùng trong UI/controller code.

> **Q: What is the difference between `Task.Wait()` and `await`?**
> A: `Wait()` BLOCK thread hiện tại (synchronous). `await` GIẢI PHÓNG thread (asynchronous). `Wait()` có thể gây deadlock trong UI apps. LUÔN dùng `await` thay `Wait()/.Result`.

> **Q: Why should you avoid `async void`?**
> A: (1) Exception KHÔNG thể catch bằng try-catch, (2) Caller không biết khi nào hoàn thành, (3) Không compose được (không `await`). Chỉ dùng cho event handlers. Mọi trường hợp khác: `async Task`.

> **Q: What is `Task.WhenAll` vs `Task.WhenAny`?**
> A: `WhenAll`: chờ TẤT CẢ tasks hoàn thành, trả về khi TẤT CẢ done. `WhenAny`: chờ BẤT KỲ 1 task hoàn thành, trả về task đầu tiên done. Dùng WhenAll cho parallel execution. Dùng WhenAny cho timeout/race.

---

## 🏋️ Bài Tập Bổ Sung (Nâng Cao)

**BT1**: Viết `async Task<string> FetchUrlAsync(string url)` dùng HttpClient. Fetch 3 URLs song song bằng `Task.WhenAll`.

**BT2**: Implement Producer-Consumer pattern dùng `ConcurrentQueue<T>` và `SemaphoreSlim`.

**BT3**: Viết program download 10 files song song, hiển thị progress bar, support cancellation bằng CancellationToken.

---

## 📎 Đáp Án Gợi Ý

- Câu hỏi kiểm tra/phỏng vấn: [99-answer-key-csharp.md#p13-async](./99-answer-key-csharp.md#p13-async)
- Bài tập thực hành: [99-answer-key-csharp.md#p13-async-exercises](./99-answer-key-csharp.md#p13-async-exercises)
- Đọc sâu lý thuyết: [97-csharp-theory-deep-dive.md#p13-async-deep](./97-csharp-theory-deep-dive.md#p13-async-deep)

