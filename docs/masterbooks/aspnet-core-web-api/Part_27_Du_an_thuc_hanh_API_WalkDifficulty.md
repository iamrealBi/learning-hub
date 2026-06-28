# Phần 27: Xây dựng API Độ khó Hành trình (WalkDifficulty) với ASP.NET Core và Kiến trúc Tối ưu

Phần 27 của khóa học sẽ đưa bạn vào hành trình xây dựng một RESTful Web API hoàn chỉnh, tập trung vào tài nguyên "Độ khó Hành trình" (WalkDifficulty). Đây là cơ hội để tổng hợp và củng cố các kiến thức nền tảng về ASP.NET Core, Entity Framework Core, Repository Pattern và Dependency Injection. Mục tiêu là tạo ra một API bền vững, dễ bảo trì và mở rộng, đồng thời khám phá cách tư duy kiến trúc này tương tác với các công cụ lập trình AI tiên tiến như Antigravity IDE.

**Mục tiêu chính của phần này:**

*   **Hiểu sâu sắc và áp dụng Repository Pattern:** Tách biệt logic truy cập dữ liệu một cách hiệu quả.
*   **Thành thạo Dependency Injection (DI):** Quản lý các phụ thuộc và nâng cao khả năng kiểm thử của ứng dụng.
*   **Thiết kế Data Transfer Objects (DTOs):** Đảm bảo tính linh hoạt, bảo mật và hiệu quả của API.
*   **Triển khai toàn diện các phương thức HTTP (GET, POST, PUT, DELETE):** Xử lý đầy đủ các thao tác CRUD cho tài nguyên `WalkDifficulty`.
*   **Kiểm thử API bằng Swagger UI:** Xác minh hoạt động chính xác của các endpoint.
*   **Củng cố quy trình làm việc chuẩn:** Phát triển API chuyên nghiệp và có hệ thống.
*   **Tích hợp tư duy Vibe Coding và Antigravity IDE:** Khám phá cách các mẫu thiết kế này hỗ trợ việc lập trình bằng AI và tự động hóa.

---

## I. Nền tảng Kiến trúc RESTful API và ASP.NET Core

Trước khi đi vào triển khai, việc nắm vững các khái niệm cốt lõi là vô cùng quan trọng. Chúng ta sẽ đào sâu hơn vào kiến trúc và các công nghệ nền tảng.

### 1. RESTful API và HTTP Verbs: Ngôn ngữ của Web

REST (Representational State Transfer) là một phong cách kiến trúc phần mềm, không phải một giao thức hay tiêu chuẩn. Nó định nghĩa một tập hợp các nguyên tắc để thiết kế các hệ thống mạng có khả năng mở rộng, dễ bảo trì và linh hoạt. Một RESTful API cho phép các hệ thống giao tiếp với nhau bằng cách thao tác với các **tài nguyên** (resources) thông qua các **phương thức HTTP tiêu chuẩn** (HTTP Verbs).

**Các nguyên tắc cốt lõi của REST:**

*   **Client-Server:** Tách biệt giao diện người dùng (client) khỏi lưu trữ dữ liệu (server), cho phép phát triển độc lập.
*   **Stateless (Không trạng thái):** Mỗi yêu cầu từ client đến server phải chứa tất cả thông tin cần thiết để server hiểu và xử lý yêu cầu đó. Server không lưu trữ bất kỳ trạng thái phiên nào của client giữa các yêu cầu. Điều này giúp API có khả năng mở rộng tốt hơn.
*   **Cacheable (Có thể lưu trữ cache):** Phản hồi từ server phải được ngầm hoặc rõ ràng định nghĩa là có thể lưu trữ cache hay không, giúp cải thiện hiệu suất.
*   **Layered System (Hệ thống phân lớp):** Client không cần biết liệu nó đang kết nối trực tiếp với server cuối cùng hay một trung gian (ví dụ: proxy, load balancer). Điều này tăng tính linh hoạt.
*   **Uniform Interface (Giao diện đồng nhất):** Nguyên tắc quan trọng nhất, nó đơn giản hóa và tách rời kiến trúc hệ thống, cho phép mỗi phần phát triển độc lập. Giao diện đồng nhất bao gồm:
    *   **Resource Identification in Requests:** Tài nguyên được định danh duy nhất (thường bằng URL).
    *   **Resource Manipulation Through Representations:** Client thao tác với tài nguyên thông qua các biểu diễn (representations) của chúng (ví dụ: JSON, XML).
    *   **Self-descriptive Messages:** Mỗi thông điệp chứa đủ thông tin để client hiểu cách xử lý nó.
    *   **Hypermedia as the Engine of Application State (HATEOAS):** Client tương tác với ứng dụng hoàn toàn thông qua siêu liên kết được cung cấp trong phản hồi của server. (Đây là một nguyên tắc nâng cao và không phải tất cả các API đều tuân thủ nghiêm ngặt).

**HTTP Verbs và Ứng dụng CRUD:**

Các phương thức HTTP (hay HTTP Verbs) là những động từ tiêu chuẩn mà client sử dụng để biểu thị ý định của mình đối với một tài nguyên:

*   **GET (Read):**
    *   **Mục đích:** Lấy thông tin của một tài nguyên hoặc một tập hợp các tài nguyên.
    *   **Tính chất:** Idempotent (gọi nhiều lần cho cùng một kết quả), Safe (không thay đổi trạng thái server).
    *   **Ví dụ:** `GET /api/WalkDifficulties` (lấy tất cả), `GET /api/WalkDifficulties/123e4567-e89b-12d3-a456-426614174000` (lấy theo ID).
    *   **Mã trạng thái phổ biến:** `200 OK`, `404 Not Found`.
*   **POST (Create):**
    *   **Mục đích:** Tạo một tài nguyên mới trên máy chủ.
    *   **Tính chất:** Không Idempotent (gọi nhiều lần sẽ tạo nhiều tài nguyên mới), không Safe.
    *   **Ví dụ:** `POST /api/WalkDifficulties` với thân yêu cầu chứa dữ liệu của độ khó mới.
    *   **Mã trạng thái phổ biến:** `201 Created` (thành công và trả về URL của tài nguyên mới), `400 Bad Request` (dữ liệu không hợp lệ), `409 Conflict`.
*   **PUT (Update/Replace):**
    *   **Mục đích:** Cập nhật *toàn bộ* thông tin của một tài nguyên hiện có hoặc tạo mới nếu nó chưa tồn tại (ít phổ biến hơn trong thực tế).
    *   **Tính chất:** Idempotent (gọi nhiều lần cho cùng một kết quả cuối cùng).
    *   **Ví dụ:** `PUT /api/WalkDifficulties/123e4567-e89b-12d3-a456-426614174000` với thân yêu cầu chứa toàn bộ dữ liệu cập nhật.
    *   **Mã trạng thái phổ biến:** `200 OK`, `204 No Content` (nếu không trả về nội dung), `404 Not Found`, `400 Bad Request`.
*   **DELETE (Delete):**
    *   **Mục đích:** Xóa một tài nguyên cụ thể.
    *   **Tính chất:** Idempotent (xóa một lần hay nhiều lần thì tài nguyên vẫn không còn), không Safe.
    *   **Ví dụ:** `DELETE /api/WalkDifficulties/123e4567-e89b-12d3-a456-426614174000`.
    *   **Mã trạng thái phổ biến:** `200 OK`, `204 No Content`, `404 Not Found`.
*   **PATCH (Partial Update):**
    *   **Mục đích:** Cập nhật *một phần* thông tin của một tài nguyên hiện có.
    *   **Tính chất:** Không Idempotent.
    *   **Ví dụ:** `PATCH /api/WalkDifficulties/123e4567-e89b-12d3-a456-426614174000` với thân yêu cầu chỉ chứa các trường cần thay đổi. (Không được sử dụng trong phần này nhưng là một verb quan trọng).

> [!NOTE]
> **Tư duy Vibe Coding và Antigravity IDE:**
> Một hệ thống AI như Antigravity IDE, với khả năng hiểu ngữ cảnh và lập kế hoạch, có thể tự động suy luận và đề xuất phương thức HTTP cùng với cấu trúc URL phù hợp dựa trên mô tả yêu cầu của bạn. Ví dụ, nếu bạn chỉ định "tạo một độ khó hành trình mới", Antigravity sẽ tự động chọn `POST` và `/api/WalkDifficulties`. Nếu bạn nói "cập nhật mã của độ khó X", nó sẽ gợi ý `PUT` hoặc `PATCH` và cấu trúc URL với ID. Điều này giúp bạn tập trung vào *ý định* nghiệp vụ thay vì các chi tiết kỹ thuật của REST.

### 2. ASP.NET Core cho Web API: Nền tảng Mạnh mẽ

ASP.NET Core là một framework mã nguồn mở, đa nền tảng và hiệu suất cao từ Microsoft, được thiết kế để xây dựng các ứng dụng web hiện đại, API và microservices. Trong ngữ cảnh Web API, ASP.NET Core cung cấp:

*   **Controllers:** Các lớp xử lý yêu cầu HTTP đến. Mỗi phương thức công khai trong Controller thường tương ứng với một hành động (action) API.
*   **Routing:** Cơ chế ánh xạ các URL đến các hành động Controller cụ thể. ASP.NET Core hỗ trợ định tuyến dựa trên thuộc tính (attribute routing) giúp việc định tuyến trở nên rõ ràng và dễ quản lý.
*   **Dependency Injection (DI):** Một hệ thống DI tích hợp sẵn, là trái tim của việc quản lý phụ thuộc và cấu trúc ứng dụng linh hoạt.

### 3. Lập trình Bất đồng bộ (Async/Await): Chìa khóa hiệu suất và mở rộng

Trong các ứng dụng Web API, hiệu suất và khả năng mở rộng là tối quan trọng. Các thao tác I/O (Input/Output) như truy cập cơ sở dữ liệu, gọi API bên ngoài, hoặc đọc/ghi file thường mất thời gian và có thể làm chặn luồng thực thi của ứng dụng nếu được thực hiện đồng bộ.

**Cơ chế hoạt động:**

*   Khi một phương thức được đánh dấu `async`, nó có thể chứa các toán tử `await`.
*   Khi `await` được gọi trên một `Task` (ví dụ: `await _dbContext.WalkDifficulties.ToListAsync();`), luồng hiện tại sẽ được giải phóng và quay trở lại pool để phục vụ các yêu cầu khác.
*   Khi thao tác I/O hoàn thành, kết quả sẽ được đưa trở lại luồng đã được giải phóng (hoặc một luồng khác từ pool), và phần còn lại của phương thức `async` sẽ tiếp tục thực thi.

**Lợi ích:**

*   **Không chặn luồng:** Cải thiện khả năng phản hồi của ứng dụng, đặc biệt trong môi trường máy chủ web nơi mỗi yêu cầu thường được xử lý trên một luồng riêng.
*   **Tăng khả năng mở rộng:** Máy chủ có thể xử lý nhiều yêu cầu đồng thời hơn với số lượng luồng giới hạn, giảm thiểu việc tạo và hủy luồng, tiết kiệm tài nguyên.
*   **Trải nghiệm người dùng tốt hơn:** Trong các ứng dụng UI, UI không bị "đóng băng" khi chờ đợi các thao tác dài.

> [!TIP]
> **Antigravity IDE và Async/Await:**
> Một hệ thống AI như Antigravity IDE được thiết kế để tuân thủ các nguyên tắc hiệu suất cao. Khi nó nhận diện một thao tác I/O (như truy vấn cơ sở dữ liệu qua EF Core), nó sẽ tự động tạo mã bất đồng bộ bằng `async` và `await`. Bạn không cần phải nhớ thêm `Async()` vào tên phương thức hay `await` ở đâu; Antigravity sẽ xử lý điều này như một phần của "Vibe Coding", nơi AI hiểu được ngữ cảnh và áp dụng các mẫu thiết kế tối ưu nhất.

---

## II. Quản lý Dữ liệu với Entity Framework Core và Repository Pattern

Việc tương tác với cơ sở dữ liệu là cốt lõi của hầu hết các ứng dụng web. Chúng ta sẽ sử dụng Entity Framework Core và Repository Pattern để quản lý dữ liệu một cách hiệu quả và có cấu trúc.

### 1. Entity Framework Core (EF Core): ORM cho .NET

EF Core là một Object-Relational Mapper (ORM) cho .NET, giúp nhà phát triển làm việc với cơ sở dữ liệu bằng cách sử dụng các đối tượng .NET (Domain Models) thay vì viết các câu lệnh SQL truyền thống.

*   **`DbContext`:** Là cầu nối chính giữa ứng dụng và cơ sở dữ liệu. Nó đại diện cho một phiên làm việc với cơ sở dữ liệu và chứa các `DbSet` (tập hợp các thực thể) ánh xạ tới các bảng.
*   **Lợi ích của ORM:**
    *   **Tăng năng suất:** Giảm đáng kể lượng mã SQL phải viết thủ công.
    *   **Trừu tượng hóa cơ sở dữ liệu:** Cho phép thay đổi cơ sở dữ liệu mà không cần thay đổi logic nghiệp vụ.
    *   **An toàn kiểu dữ liệu:** Đảm bảo các thao tác dữ liệu được kiểm tra kiểu tại thời điểm biên dịch.
    *   **Tích hợp LINQ:** Cho phép truy vấn dữ liệu bằng cách sử dụng cú pháp LINQ quen thuộc.

### 2. Dependency Injection (DI): Giảm Phụ thuộc, Tăng Linh hoạt

Dependency Injection là một kỹ thuật thiết kế phần mềm giúp giảm sự phụ thuộc cứng nhắc giữa các thành phần của ứng dụng. Thay vì một lớp tự tạo ra các đối tượng mà nó cần (phụ thuộc), các đối tượng đó sẽ được "tiêm" (inject) vào nó từ bên ngoài.

**Cơ chế hoạt động trong ASP.NET Core:**

*   **Bộ chứa DI (DI Container):** ASP.NET Core có một bộ chứa DI tích hợp sẵn. Bạn đăng ký các dịch vụ (service) vào bộ chứa này trong phương thức `ConfigureServices` (hoặc trực tiếp trong `Program.cs` từ .NET 6 trở đi).
*   **Đăng ký dịch vụ:**
    *   `builder.Services.AddScoped<IMyService, MyServiceImplementation>();`
    *   Khi một lớp yêu cầu `IMyService` thông qua constructor của nó, bộ chứa DI sẽ tạo một thể hiện của `MyServiceImplementation` và cung cấp nó.

**Các loại vòng đời dịch vụ (Service Lifetimes):**

*   **`AddScoped`:**
    *   **Vòng đời:** Một thể hiện được tạo ra *một lần cho mỗi yêu cầu HTTP*. Nếu cùng một dịch vụ được yêu cầu nhiều lần trong cùng một yêu cầu HTTP, nó sẽ nhận được cùng một thể hiện.
    *   **Sử dụng:** Thích hợp cho các dịch vụ có trạng thái liên quan đến yêu cầu hiện tại, ví dụ: `DbContext` (đảm bảo tất cả các thao tác DB trong một yêu cầu sử dụng cùng một phiên).
*   **`AddTransient`:**
    *   **Vòng đời:** Một thể hiện *mới* được tạo ra *mỗi khi nó được yêu cầu*.
    *   **Sử dụng:** Thích hợp cho các dịch vụ nhẹ, không trạng thái, hoặc các dịch vụ cần một thể hiện độc lập mỗi khi được sử dụng.
*   **`AddSingleton`:**
    *   **Vòng đời:** Một thể hiện *duy nhất* được tạo ra *lần đầu tiên nó được yêu cầu* và được tái sử dụng trong suốt vòng đời của ứng dụng.
    *   **Sử dụng:** Thích hợp cho các dịch vụ không trạng thái, chia sẻ dữ liệu chung hoặc tài nguyên đắt tiền (ví dụ: logger, cache manager).

**Lợi ích của DI:**

*   **Giảm sự phụ thuộc (Loose Coupling):** Các lớp không cần biết cách tạo ra các phụ thuộc của chúng, chỉ cần biết cách sử dụng chúng thông qua interface.
*   **Tăng khả năng kiểm thử (Testability):** Dễ dàng thay thế các phụ thuộc thực bằng các đối tượng mock hoặc stub trong kiểm thử đơn vị.
*   **Dễ bảo trì và mở rộng:** Dễ dàng thay đổi việc triển khai một dịch vụ mà không ảnh hưởng đến các lớp sử dụng nó.

> [!NOTE]
> **Antigravity IDE và Dependency Injection:**
> Đối với Antigravity IDE, DI là một mẫu thiết kế lý tưởng. AI có thể dễ dàng phân tích các constructor và tự động đăng ký các dịch vụ cần thiết vào bộ chứa DI. Khi bạn áp dụng Vibe Coding và chỉ định "Tôi cần một Repository cho WalkDifficulty", Antigravity sẽ tự động tạo interface, lớp triển khai và cấu hình `AddScoped` trong `Program.cs`, hiểu rằng Repository thường cần một phiên `DbContext` cho mỗi yêu cầu. Nó sẽ đảm bảo mọi thứ được kết nối một cách chính xác mà không cần bạn phải viết từng dòng code đăng ký.

### 3. Repository Pattern: Trừu tượng hóa Tầng Truy Cập Dữ liệu

Repository Pattern là một mẫu thiết kế giúp tạo ra một lớp trừu tượng (abstraction layer) giữa logic nghiệp vụ của ứng dụng và cơ chế lưu trữ dữ liệu. Nó cung cấp một tập hợp các phương thức tập trung để truy cập và thao tác với dữ liệu, giúp ứng dụng không cần biết chi tiết về cách dữ liệu được lưu trữ hay truy xuất.

**Cấu trúc:**

*   **Interface Repository (`IRepository`):** Định nghĩa hợp đồng cho các thao tác dữ liệu mà Repository sẽ hỗ trợ (ví dụ: `GetAll`, `GetById`, `Add`, `Update`, `Delete`). Đây là phần mà logic nghiệp vụ tương tác.
*   **Lớp triển khai Repository (`Repository`):** Triển khai interface và chứa logic tương tác trực tiếp với cơ sở dữ liệu (sử dụng EF Core, Dapper, hoặc bất kỳ công nghệ truy cập dữ liệu nào khác).

**Lợi ích cốt lõi:**

*   **Tách biệt mối quan tâm (Separation of Concerns):** Logic truy cập dữ liệu được đóng gói hoàn toàn trong Repository, giữ cho Controller và các lớp nghiệp vụ khác "sạch" và chỉ tập trung vào nghiệp vụ.
*   **Khả năng kiểm thử (Testability):** Vì Controller chỉ phụ thuộc vào interface của Repository, bạn có thể dễ dàng tạo các đối tượng mock hoặc stub cho Repository trong các bài kiểm thử đơn vị của Controller, mà không cần tương tác với cơ sở dữ liệu thực.
*   **Dễ dàng thay đổi nguồn dữ liệu:** Nếu bạn quyết định chuyển từ SQL Server sang MongoDB hoặc một dịch vụ lưu trữ đám mây, bạn chỉ cần thay đổi lớp triển khai Repository mà không cần sửa đổi Controller hoặc các lớp nghiệp vụ.
*   **Tái sử dụng mã:** Các thao tác truy cập dữ liệu chung có thể được tái sử dụng trong nhiều phần của ứng dụng.

> [!IMPORTANT]
> **Antigravity IDE và Repository Pattern:**
> Repository Pattern là một ví dụ điển hình về cách các mẫu thiết kế giúp AI như Antigravity IDE hiểu và tạo mã hiệu quả. Khi bạn yêu cầu Antigravity "tạo một API cho tài nguyên WalkDifficulty", nó sẽ tự động nhận ra nhu cầu về một Repository. Với khả năng lập kế hoạch, nó sẽ:
> 1.  Tạo interface `IWalkDifficultyRepository` dựa trên các thao tác CRUD tiêu chuẩn.
> 2.  Tạo lớp `WalkDifficultyRepository` triển khai interface đó, sử dụng EF Core và `DbContext` đã được cấu hình.
> 3.  Đảm bảo rằng `WalkDifficultiesController` chỉ giao tiếp với `IWalkDifficultyRepository` thông qua DI.
> Điều này minh họa Vibe Coding: bạn tập trung vào *ý tưởng kiến trúc* (tách biệt dữ liệu) và Antigravity IDE sẽ tự động triển khai các chi tiết tuân thủ mẫu thiết kế đó.

---

## III. Thiết kế Mô hình Dữ liệu và Đối tượng Truyền Tải (DTOs)

Trong một hệ thống API, việc quản lý dữ liệu giữa các lớp và qua mạng đòi hỏi sự phân biệt rõ ràng giữa các mô hình nghiệp vụ và các mô hình trình bày.

### 1. Domain Model: Trái tim của Nghiệp vụ (`WalkDifficulty`)

Domain Model là các lớp đại diện cho các thực thể cốt lõi trong miền nghiệp vụ của ứng dụng. Chúng thường được ánh xạ trực tiếp tới các bảng trong cơ sở dữ liệu bởi Entity Framework Core.

*   **`WalkDifficulty.cs`:**
    ```csharp
    // Models/Domain/WalkDifficulty.cs
    using System;

    namespace NZWalks.API.Models.Domain
    {
        public class WalkDifficulty
        {
            public Guid Id { get; set; }
            public string Code { get; set; }
        }
    }
    ```
*   **`Guid` (Globally Unique Identifier):**
    *   **Ưu điểm:**
        *   **Tính duy nhất toàn cầu:** Rất khó xảy ra trùng lặp, ngay cả khi được tạo trên các hệ thống phân tán khác nhau.
        *   **Tạo ID phi tập trung:** Có thể tạo ID ở phía client hoặc ở tầng ứng dụng mà không cần truy vấn cơ sở dữ liệu, giảm tải cho DB.
        *   **Bảo mật:** Khó đoán hơn các ID tăng dần.
    *   **Nhược điểm:**
        *   **Kích thước lớn:** 128-bit, lớn hơn `int` hoặc `long`, có thể tốn thêm không gian lưu trữ và băng thông.
        *   **Ít thân thiện với con người:** Khó nhớ và gõ thủ công.
        *   **Hiệu suất cơ sở dữ liệu (có thể):** Nếu `Guid` được sử dụng làm khóa chính clustered index trong SQL Server mà không theo thứ tự (ví dụ: `Guid.NewGuid()`), nó có thể dẫn đến phân mảnh index nghiêm trọng, ảnh hưởng đến hiệu suất truy vấn. Tuy nhiên, các kỹ thuật như `sequential Guid` (ví dụ: `Comb Guid`) có thể giảm thiểu vấn đề này.

### 2. Data Transfer Objects (DTOs): Giao diện API linh hoạt và an toàn

Data Transfer Object (DTO) là các lớp được thiết kế đặc biệt để truyền dữ liệu giữa các lớp của ứng dụng hoặc giữa ứng dụng và client (qua API). DTOs không phải là Domain Models; chúng là *biểu diễn* của Domain Models cho một mục đích cụ thể.

*   **`WalkDifficultyDto.cs` (Phản hồi GET):**
    ```csharp
    // Models/DTO/WalkDifficultyDto.cs
    using System;

    namespace NZWalks.API.Models.DTO
    {
        public class WalkDifficultyDto
        {
            public Guid Id { get; set; }
            public string Code { get; set; }
        }
    }
    ```
*   **`AddWalkDifficultyRequestDto.cs` (Yêu cầu POST):**
    ```csharp
    // Models/DTO/AddWalkDifficultyRequestDto.cs
    namespace NZWalks.API.Models.DTO
    {
        public class AddWalkDifficultyRequestDto
        {
            public string Code { get; set; }
        }
    }
    ```
*   **`UpdateWalkDifficultyRequestDto.cs` (Yêu cầu PUT):**
    ```csharp
    // Models/DTO/UpdateWalkDifficultyRequestDto.cs
    namespace NZWalks.API.Models.DTO
    {
        public class UpdateWalkDifficultyRequestDto
        {
            public string Code { get; set; }
        }
    }
    ```

**Tại sao DTOs lại cần thiết?**

*   **Bảo mật:** Không để lộ tất cả các thuộc tính của Domain Model ra bên ngoài. Ví dụ, một Domain Model có thể chứa các trường nhạy cảm như `PasswordHash`, `InternalStatus`, hoặc các trường chỉ dùng nội bộ. DTOs cho phép bạn kiểm soát chính xác những gì được phơi bày qua API.
*   **Linh hoạt trong giao diện API:** Bạn có thể tạo các DTO khác nhau cho các trường hợp sử dụng khác nhau. Ví dụ, DTO để hiển thị danh sách có thể chỉ cần `Id` và `Name`, trong khi DTO chi tiết có thể chứa nhiều thông tin hơn.
*   **Giảm tải mạng:** Chỉ gửi những dữ liệu cần thiết, tối ưu hóa payload của yêu cầu/phản hồi HTTP, đặc biệt quan trọng với các ứng dụng di động hoặc mạng chậm.
*   **Ngăn chặn Over-Posting:** Khi client gửi dữ liệu để cập nhật, nếu bạn sử dụng trực tiếp Domain Model, client có thể cố gắng gửi các trường không mong muốn để cập nhật các thuộc tính nhạy cảm (ví dụ: thay đổi `IsAdmin` thành `true`). DTOs giúp bạn chỉ chấp nhận các trường được phép cập nhật.
*   **Tách biệt mối quan tâm:** Giữ cho Domain Model tập trung vào logic nghiệp vụ và không bị nhiễm bẩn bởi các yêu cầu của giao diện API.

> [!TIP]
> **Antigravity IDE và DTOs:**
> Antigravity IDE có thể tự động phân tích Domain Model của bạn và đề xuất các DTOs phù hợp cho các thao tác CRUD. Ví dụ, với `WalkDifficulty` Domain Model, Antigravity sẽ tự động tạo `AddWalkDifficultyRequestDto` (không có `Id`), `UpdateWalkDifficultyRequestDto` (có thể không có `Id` nếu ID được truyền qua URL), và `WalkDifficultyDto` (có `Id` và các thuộc tính khác để hiển thị). Điều này là một phần của Vibe Coding, nơi AI hiểu được các mẫu thiết kế API phổ biến và tự động tạo cấu trúc dữ liệu cần thiết.

### 3. Tự động Ánh xạ với AutoMapper

Việc chuyển đổi dữ liệu giữa Domain Models và DTOs (và ngược lại) là một tác vụ lặp đi lặp lại và dễ gây lỗi. AutoMapper là một thư viện phổ biến trong .NET giúp tự động hóa quá trình ánh xạ này.

*   **`WalkDifficultyProfile.cs`:**
    ```csharp
    // Mappings/WalkDifficultyProfile.cs
    using AutoMapper;
    using NZWalks.API.Models.Domain;
    using NZWalks.API.Models.DTO;

    namespace NZWalks.API.Mappings
    {
        public class WalkDifficultyProfile : Profile
        {
            public WalkDifficultyProfile()
            {
                // Ánh xạ từ Domain Model sang DTO (và ngược lại)
                CreateMap<WalkDifficulty, WalkDifficultyDto>().ReverseMap();

                // Ánh xạ từ DTO yêu cầu thêm mới sang Domain Model (và ngược lại)
                CreateMap<AddWalkDifficultyRequestDto, WalkDifficulty>().ReverseMap();

                // Ánh xạ từ DTO yêu cầu cập nhật sang Domain Model (và ngược lại)
                CreateMap<UpdateWalkDifficultyRequestDto, WalkDifficulty>().ReverseMap();
            }
        }
    }
    ```
*   **Đăng ký AutoMapper trong `Program.cs`:**
    ```csharp
    // Program.cs
    // ...
    builder.Services.AddAutoMapper(typeof(Program).Assembly);
    // ...
    ```
    Dòng này sẽ tìm tất cả các lớp kế thừa `Profile` trong assembly hiện tại và đăng ký chúng với AutoMapper.

> [!TIP]
> `ReverseMap()` là một tính năng tiện lợi của AutoMapper, cho phép bạn định nghĩa ánh xạ hai chiều (từ A sang B và từ B sang A) chỉ với một dòng code, giúp giảm thiểu sự lặp lại.

> [!NOTE]
> **Antigravity IDE và AutoMapper:**
> AutoMapper là một thư viện hoàn hảo cho việc tự động hóa bằng AI. Antigravity IDE có thể đọc các định nghĩa Domain Model và DTO của bạn, sau đó tự động tạo ra các `Profile` cho AutoMapper. Nó thậm chí có thể nhận diện các trường có tên khác nhau nhưng có ý nghĩa tương tự và đề xuất các quy tắc ánh xạ tùy chỉnh. Với Vibe Coding, bạn chỉ cần định nghĩa các cấu trúc dữ liệu và Antigravity sẽ xử lý việc kết nối chúng, đảm bảo dữ liệu được chuyển đổi chính xác và hiệu quả.

---

## IV. Triển khai Thực hành: API `WalkDifficulty`

Bây giờ, chúng ta sẽ áp dụng các kiến thức đã học để xây dựng API thực tế.

### 1. Chuẩn bị Mô hình Dữ liệu (Domain Model)

Tạo lớp `WalkDifficulty.cs` trong thư mục `Models/Domain` để đại diện cho thực thể `WalkDifficulty` trong cơ sở dữ liệu:

```csharp
// Models/Domain/WalkDifficulty.cs
using System;

namespace NZWalks.API.Models.Domain
{
    public class WalkDifficulty
    {
        public Guid Id { get; set; } // Khóa chính duy nhất
        public string Code { get; set; } // Mã độ khó (ví dụ: "Easy", "Medium", "Hard")
    }
}
```

### 2. Triển khai Repository Pattern cho `WalkDifficulty`

Chúng ta sẽ tạo một interface và một lớp triển khai để tách biệt logic truy cập dữ liệu.

#### 2.1. Tạo Interface `IWalkDifficultyRepository`

Interface này định nghĩa các hợp đồng cho các thao tác CRUD mà Repository sẽ hỗ trợ.

```csharp
// Repositories/IWalkDifficultyRepository.cs
using NZWalks.API.Models.Domain;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace NZWalks.API.Repositories
{
    public interface IWalkDifficultyRepository
    {
        Task<IEnumerable<WalkDifficulty>> GetAllAsync(); // Lấy tất cả
        Task<WalkDifficulty> GetByIdAsync(Guid id);     // Lấy theo ID
        Task<WalkDifficulty> AddAsync(WalkDifficulty walkDifficulty); // Thêm mới
        Task<WalkDifficulty> UpdateAsync(Guid id, WalkDifficulty walkDifficulty); // Cập nhật
        Task<WalkDifficulty> DeleteAsync(Guid id);      // Xóa
    }
}
```

#### 2.2. Triển khai Lớp `WalkDifficultyRepository`

Lớp này triển khai interface `IWalkDifficultyRepository` và tương tác trực tiếp với cơ sở dữ liệu thông qua `NZWalksDbContext`.

```csharp
// Repositories/WalkDifficultyRepository.cs
using Microsoft.EntityFrameworkCore;
using NZWalks.API.Data;
using NZWalks.API.Models.Domain;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace NZWalks.API.Repositories
{
    public class WalkDifficultyRepository : IWalkDifficultyRepository
    {
        private readonly NZWalksDbContext _dbContext;

        // Constructor injection: NZWalksDbContext được tiêm vào đây
        public WalkDifficultyRepository(NZWalksDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        /// <summary>
        /// Lấy tất cả các độ khó hành trình từ cơ sở dữ liệu.
        /// </summary>
        /// <returns>Một tập hợp các đối tượng WalkDifficulty.</returns>
        public async Task<IEnumerable<WalkDifficulty>> GetAllAsync()
        {
            // Sử dụng ToListAsync để truy vấn bất đồng bộ và trả về tất cả kết quả
            return await _dbContext.WalkDifficulties.ToListAsync();
        }

        /// <summary>
        /// Lấy một độ khó hành trình cụ thể theo ID.
        /// </summary>
        /// <param name="id">ID của độ khó hành trình cần lấy.</param>
        /// <returns>Đối tượng WalkDifficulty nếu tìm thấy, ngược lại là null.</returns>
        public async Task<WalkDifficulty> GetByIdAsync(Guid id)
        {
            // Sử dụng FirstOrDefaultAsync để tìm kiếm bất đồng bộ và trả về đối tượng đầu tiên hoặc null
            return await _dbContext.WalkDifficulties.FirstOrDefaultAsync(x => x.Id == id);
        }

        /// <summary>
        /// Thêm một độ khó hành trình mới vào cơ sở dữ liệu.
        /// </summary>
        /// <param name="walkDifficulty">Đối tượng WalkDifficulty cần thêm.</param>
        /// <returns>Đối tượng WalkDifficulty đã được thêm (bao gồm ID nếu được tạo bởi DB).</returns>
        public async Task<WalkDifficulty> AddAsync(WalkDifficulty walkDifficulty)
        {
            // Thêm đối tượng vào DbSet nhưng chưa lưu vào DB
            await _dbContext.WalkDifficulties.AddAsync(walkDifficulty);
            // Lưu các thay đổi vào cơ sở dữ liệu bất đồng bộ
            await _dbContext.SaveChangesAsync();
            return walkDifficulty;
        }

        /// <summary>
        /// Cập nhật một độ khó hành trình hiện có trong cơ sở dữ liệu.
        /// </summary>
        /// <param name="id">ID của độ khó hành trình cần cập nhật.</param>
        /// <param name="walkDifficulty">Đối tượng WalkDifficulty chứa thông tin cập nhật.</param>
        /// <returns>Đối tượng WalkDifficulty đã được cập nhật nếu tìm thấy, ngược lại là null.</returns>
        public async Task<WalkDifficulty> UpdateAsync(Guid id, WalkDifficulty walkDifficulty)
        {
            // Tìm đối tượng hiện có trong DB
            var existingWalkDifficulty = await _dbContext.WalkDifficulties.FirstOrDefaultAsync(x => x.Id == id);

            if (existingWalkDifficulty == null)
            {
                return null; // Không tìm thấy để cập nhật
            }

            // Cập nhật các thuộc tính của đối tượng hiện có
            existingWalkDifficulty.Code = walkDifficulty.Code;

            // Lưu các thay đổi vào cơ sở dữ liệu bất đồng bộ
            await _dbContext.SaveChangesAsync();
            return existingWalkDifficulty;
        }

        /// <summary>
        /// Xóa một độ khó hành trình khỏi cơ sở dữ liệu.
        /// </summary>
        /// <param name="id">ID của độ khó hành trình cần xóa.</param>
        /// <returns>Đối tượng WalkDifficulty đã bị xóa nếu tìm thấy, ngược lại là null.</returns>
        public async Task<WalkDifficulty> DeleteAsync(Guid id)
        {
            // Tìm đối tượng cần xóa trong DB
            var existingWalkDifficulty = await _dbContext.WalkDifficulties.FirstOrDefaultAsync(x => x.Id == id);

            if (existingWalkDifficulty == null)
            {
                return null; // Không tìm thấy để xóa
            }

            // Đánh dấu đối tượng để xóa nhưng chưa lưu vào DB
            _dbContext.WalkDifficulties.Remove(existingWalkDifficulty);
            // Lưu các thay đổi vào cơ sở dữ liệu bất đồng bộ
            await _dbContext.SaveChangesAsync();
            return existingWalkDifficulty;
        }
    }
}
```

### 3. Cấu hình Dependency Injection cho Repository

Đăng ký `WalkDifficultyRepository` vào hệ thống DI trong file `Program.cs`. Điều này cho phép các Controller yêu cầu `IWalkDifficultyRepository` và nhận một thể hiện của `WalkDifficultyRepository`.

```csharp
// Program.cs
// ...
builder.Services.AddScoped<IWalkDifficultyRepository, WalkDifficultyRepository>();
// ...
```
> [!NOTE]
> Sử dụng `AddScoped` cho Repository là lựa chọn phổ biến vì Repository thường chứa `DbContext` được đăng ký là `Scoped`. Điều này đảm bảo rằng một thể hiện `DbContext` duy nhất được sử dụng trong suốt vòng đời của một yêu cầu HTTP, duy trì tính nhất quán của dữ liệu trong một giao dịch.

### 4. Định nghĩa Data Transfer Objects (DTOs)

Tạo các lớp DTO sau trong thư mục `Models/DTO`:

```csharp
// Models/DTO/WalkDifficultyDto.cs
using System;

namespace NZWalks.API.Models.DTO
{
    public class WalkDifficultyDto
    {
        public Guid Id { get; set; }
        public string Code { get; set; }
    }
}

// Models/DTO/AddWalkDifficultyRequestDto.cs
namespace NZWalks.API.Models.DTO
{
    public class AddWalkDifficultyRequestDto
    {
        public string Code { get; set; } // Chỉ cần Code khi thêm mới
    }
}

// Models/DTO/UpdateWalkDifficultyRequestDto.cs
namespace NZWalks.API.Models.DTO
{
    public class UpdateWalkDifficultyRequestDto
    {
        public string Code { get; set; } // Chỉ cần Code khi cập nhật
    }
}
```

### 5. Cấu hình AutoMapper Profile

Tạo lớp `WalkDifficultyProfile.cs` trong thư mục `Mappings` để định nghĩa các quy tắc ánh xạ.

```csharp
// Mappings/WalkDifficultyProfile.cs
using AutoMapper;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTO;

namespace NZWalks.API.Mappings
{
    public class WalkDifficultyProfile : Profile
    {
        public WalkDifficultyProfile()
        {
            // Ánh xạ hai chiều từ Domain Model sang DTO và ngược lại
            CreateMap<WalkDifficulty, WalkDifficultyDto>().ReverseMap();

            // Ánh xạ hai chiều từ DTO yêu cầu thêm mới sang Domain Model và ngược lại
            CreateMap<AddWalkDifficultyRequestDto, WalkDifficulty>().ReverseMap();

            // Ánh xạ hai chiều từ DTO yêu cầu cập nhật sang Domain Model và ngược lại
            CreateMap<UpdateWalkDifficultyRequestDto, WalkDifficulty>().ReverseMap();
        }
    }
}
```
Và đừng quên đăng ký AutoMapper trong `Program.cs`:
```csharp
// Program.cs
// ...
builder.Services.AddAutoMapper(typeof(Program).Assembly);
// ...
```

### 6. Xây dựng API Controller (`WalkDifficultiesController`)

Controller sẽ là điểm tiếp nhận các yêu cầu HTTP, điều phối chúng tới Repository và trả về phản hồi.

```csharp
// Controllers/WalkDifficultiesController.cs
using AutoMapper;
using Microsoft.AspNetCore.Mvc;
using NZWalks.API.Models.Domain;
using NZWalks.API.Models.DTO;
using NZWalks.API.Repositories;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace NZWalks.API.Controllers
{
    [ApiController] // Đánh dấu đây là một API Controller, kích hoạt các tính năng như automatic model validation
    [Route("api/[controller]")] // Định tuyến attribute: URL sẽ là api/WalkDifficulties
    public class WalkDifficultiesController : ControllerBase // Kế thừa ControllerBase cho API controllers
    {
        private readonly IWalkDifficultyRepository _walkDifficultyRepository;
        private readonly IMapper _mapper;

        // Constructor Injection: Tiêm IWalkDifficultyRepository và IMapper
        public WalkDifficultiesController(IWalkDifficultyRepository walkDifficultyRepository, IMapper mapper)
        {
            _walkDifficultyRepository = walkDifficultyRepository;
            _mapper = mapper;
        }

        // --------------------------------------------------------------------------
        // GET: Lấy tất cả độ khó hành trình
        // Endpoint: GET /api/WalkDifficulties
        // Mô tả: Trả về danh sách tất cả các đối tượng WalkDifficulty.
        // Mã trạng thái thành công: 200 OK
        // --------------------------------------------------------------------------
        [HttpGet]
        public async Task<IActionResult> GetAllWalkDifficulties()
        {
            // 1. Gọi Repository để lấy dữ liệu Domain Model từ DB
            var walkDifficultiesDomain = await _walkDifficultyRepository.GetAllAsync();

            // 2. Ánh xạ danh sách Domain Model sang danh sách DTO để trả về client
            var walkDifficultiesDto = _mapper.Map<List<WalkDifficultyDto>>(walkDifficultiesDomain);

            // 3. Trả về phản hồi HTTP 200 OK kèm theo dữ liệu DTO
            return Ok(walkDifficultiesDto);
        }

        // --------------------------------------------------------------------------
        // GET: Lấy độ khó hành trình theo ID
        // Endpoint: GET /api/WalkDifficulties/{id}
        // Mô tả: Trả về một đối tượng WalkDifficulty cụ thể dựa trên ID.
        // Mã trạng thái thành công: 200 OK
        // Mã trạng thái thất bại: 404 Not Found (nếu không tìm thấy ID)
        // --------------------------------------------------------------------------
        [HttpGet("{id:Guid}")] // Định tuyến với tham số ID kiểu Guid từ URL
        public async Task<IActionResult> GetWalkDifficultyById([FromRoute] Guid id) // [FromRoute] chỉ định lấy id từ route
        {
            // 1. Gọi Repository để lấy dữ liệu Domain Model theo ID từ DB
            var walkDifficultyDomain = await _walkDifficultyRepository.GetByIdAsync(id);

            // 2. Kiểm tra nếu không tìm thấy Domain Model
            if (walkDifficultyDomain == null)
            {
                return NotFound(); // Trả về HTTP 404 Not Found
            }

            // 3. Ánh xạ Domain Model sang DTO để trả về client
            var walkDifficultyDto = _mapper.Map<WalkDifficultyDto>(walkDifficultyDomain);

            // 4. Trả về phản hồi HTTP 200 OK kèm theo dữ liệu DTO
            return Ok(walkDifficultyDto);
        }

        // --------------------------------------------------------------------------
        // POST: Thêm độ khó hành trình mới
        // Endpoint: POST /api/WalkDifficulties
        // Mô tả: Tạo một đối tượng WalkDifficulty mới.
        // Mã trạng thái thành công: 201 Created (kèm URL của tài nguyên mới)
        // Mã trạng thái thất bại: 400 Bad Request (nếu dữ liệu không hợp lệ)
        // --------------------------------------------------------------------------
        [HttpPost]
        public async Task<IActionResult> AddWalkDifficulty([FromBody] AddWalkDifficultyRequestDto addWalkDifficultyRequestDto) // [FromBody] chỉ định lấy dữ liệu từ thân yêu cầu
        {
            // 1. Ánh xạ DTO yêu cầu sang Domain Model
            var walkDifficultyDomain = _mapper.Map<WalkDifficulty>(addWalkDifficultyRequestDto);

            // 2. Gán một ID mới (GUID) cho Domain Model trước khi thêm vào DB
            walkDifficultyDomain.Id = Guid.NewGuid();

            // 3. Gọi Repository để thêm Domain Model vào DB
            walkDifficultyDomain = await _walkDifficultyRepository.AddAsync(walkDifficultyDomain);

            // 4. Ánh xạ Domain Model đã thêm sang DTO để trả về client
            var walkDifficultyDto = _mapper.Map<WalkDifficultyDto>(walkDifficultyDomain);

            // 5. Trả về phản hồi HTTP 201 Created.
            //    CreatedAtAction tạo ra một header Location chứa URL của tài nguyên mới
            //    và trả về DTO của tài nguyên đó.
            return CreatedAtAction(nameof(GetWalkDifficultyById), new { id = walkDifficultyDto.Id }, walkDifficultyDto);
        }

        // --------------------------------------------------------------------------
        // PUT: Cập nhật độ khó hành trình
        // Endpoint: PUT /api/WalkDifficulties/{id}
        // Mô tả: Cập nhật toàn bộ thông tin của một đối tượng WalkDifficulty hiện có.
        // Mã trạng thái thành công: 200 OK
        // Mã trạng thái thất bại: 404 Not Found (nếu không tìm thấy ID), 400 Bad Request
        // --------------------------------------------------------------------------
        [HttpPut("{id:Guid}")]
        public async Task<IActionResult> UpdateWalkDifficulty([FromRoute] Guid id, [FromBody] UpdateWalkDifficultyRequestDto updateWalkDifficultyRequestDto)
        {
            // 1. Ánh xạ DTO yêu cầu cập nhật sang Domain Model
            var walkDifficultyDomain = _mapper.Map<WalkDifficulty>(updateWalkDifficultyRequestDto);

            // 2. Gọi Repository để cập nhật Domain Model trong DB
            walkDifficultyDomain = await _walkDifficultyRepository.UpdateAsync(id, walkDifficultyDomain);

            // 3. Kiểm tra nếu không tìm thấy Domain Model để cập nhật
            if (walkDifficultyDomain == null)
            {
                return NotFound(); // Trả về HTTP 404 Not Found
            }

            // 4. Ánh xạ Domain Model đã cập nhật sang DTO để trả về client
            var walkDifficultyDto = _mapper.Map<WalkDifficultyDto>(walkDifficultyDomain);

            // 5. Trả về phản hồi HTTP 200 OK kèm theo dữ liệu DTO
            return Ok(walkDifficultyDto);
        }

        // --------------------------------------------------------------------------
        // DELETE: Xóa độ khó hành trình
        // Endpoint: DELETE /api/WalkDifficulties/{id}
        // Mô tả: Xóa một đối tượng WalkDifficulty cụ thể.
        // Mã trạng thái thành công: 200 OK (kèm DTO của đối tượng đã xóa) hoặc 204 No Content
        // Mã trạng thái thất bại: 404 Not Found (nếu không tìm thấy ID)
        // --------------------------------------------------------------------------
        [HttpDelete("{id:Guid}")]
        public async Task<IActionResult> DeleteWalkDifficulty([FromRoute] Guid id)
        {
            // 1. Gọi Repository để xóa Domain Model từ DB
            var deletedWalkDifficultyDomain = await _walkDifficultyRepository.DeleteAsync(id);

            // 2. Kiểm tra nếu không tìm thấy Domain Model để xóa
            if (deletedWalkDifficultyDomain == null)
            {
                return NotFound(); // Trả về HTTP 404 Not Found
            }

            // 3. Ánh xạ Domain Model đã xóa sang DTO để trả về client (tùy chọn, có thể trả về 204 No Content)
            var deletedWalkDifficultyDto = _mapper.Map<WalkDifficultyDto>(deletedWalkDifficultyDomain);

            // 4. Trả về phản hồi HTTP 200 OK kèm theo DTO của đối tượng đã xóa
            return Ok(deletedWalkDifficultyDto);
        }
    }
}
```

> [!NOTE]
> *   **`[ApiController]` attribute:** Tự động cung cấp các tính năng hữu ích cho API Controllers như:
    *   **Automatic HTTP 400 responses:** Tự động trả về HTTP 400 Bad Request nếu model binding hoặc validation thất bại.
    *   **Attribute routing requirement:** Yêu cầu sử dụng attribute routing.
    *   **Binding source parameter inference:** Tự động suy luận nguồn của tham số (ví dụ: `[FromRoute]`, `[FromBody]`) nếu không chỉ định rõ.
*   **`CreatedAtAction`:** Phương thức này lý tưởng cho phản hồi `POST` vì nó tuân thủ nguyên tắc RESTful API bằng cách:
    *   Trả về mã trạng thái `201 Created`.
    *   Bao gồm một header `Location` chứa URL đầy đủ của tài nguyên mới được tạo.
    *   Bao gồm DTO của tài nguyên mới trong phần thân phản hồi.

> [!IMPORTANT]
> **Antigravity IDE và Phát triển Controller:**
> Đây là nơi Antigravity IDE thể hiện sức mạnh của Vibe Coding. Với các định nghĩa về Domain Model, DTOs và Repository, Antigravity có thể:
> 1.  **Tự động tạo Controller:** Scaffold toàn bộ `WalkDifficultiesController` với các phương thức CRUD, các attribute định tuyến (`[Route]`, `[HttpGet]`, `[HttpPost]`, v.v.), và DI cho Repository và Mapper.
> 2.  **Suy luận logic:** Tự động viết logic gọi Repository, thực hiện ánh xạ DTO, và trả về các `IActionResult` với mã trạng thái HTTP chính xác (`Ok`, `NotFound`, `CreatedAtAction`).
> 3.  **Đảm bảo tính nhất quán:** Đảm bảo tất cả các endpoint tuân thủ các nguyên tắc RESTful và sử dụng các mẫu thiết kế đã định.
> Bạn có thể đưa ra một yêu cầu cấp cao như "Tạo một API CRUD đầy đủ cho WalkDifficulty, sử dụng Repository Pattern và AutoMapper", và Antigravity sẽ lập kế hoạch, tạo các file cần thiết (Domain, Repo, DTO, Mapper, Controller) và viết mã chi tiết.

---

## V. Kiểm thử và Ghi nhận Thay đổi

Sau khi hoàn tất việc triển khai, bước tiếp theo là kiểm thử và lưu trữ công việc của bạn.

### 1. Kiểm thử API với Swagger UI

Chạy ứng dụng (ví dụ: `dotnet run` hoặc F5 trong Visual Studio) và truy cập Swagger UI (thường là `https://localhost:port/swagger`) để kiểm thử các endpoint API.

*   **1.1. Kiểm thử GET All WalkDifficulties:**
    *   Mở endpoint `GET /api/WalkDifficulties`.
    *   Nhấp vào "Try it out" và sau đó "Execute".
    *   Quan sát phản hồi `200 OK` và danh sách các độ khó hành trình hiện có.

*   **1.2. Kiểm thử GET WalkDifficulty By ID:**
    *   Lấy một `Id` từ kết quả `GET All`.
    *   Mở endpoint `GET /api/WalkDifficulties/{id}`.
    *   Nhập `Id` vào trường tham số và nhấp "Execute".
    *   Bạn sẽ nhận được `200 OK` với thông tin chi tiết hoặc `404 Not Found` nếu ID không tồn tại.

*   **1.3. Kiểm thử POST Add WalkDifficulty:**
    *   Mở endpoint `POST /api/WalkDifficulties`.
    *   Nhấp vào "Try it out".
    *   Trong phần "Request body", nhập một đối tượng JSON với thuộc tính `Code` mới, ví dụ: `{"code": "SuperHard"}`.
    *   Nhấp "Execute".
    *   Quan sát phản hồi `201 Created` kèm theo DTO của độ khó hành trình mới, bao gồm cả `Id` được tạo tự động và header `Location`.

*   **1.4. Kiểm thử PUT Update WalkDifficulty:**
    *   Lấy một `Id` của độ khó hành trình hiện có (ví dụ: từ kết quả POST hoặc GET All).
    *   Mở endpoint `PUT /api/WalkDifficulties/{id}`.
    *   Nhập `Id` vào trường tham số.
    *   Trong phần "Request body", nhập một đối tượng JSON với `Code` mới, ví dụ: `{"code": "UltraHard"}`.
    *   Nhấp "Execute".
    *   Bạn sẽ nhận được `200 OK` với DTO của độ khó hành trình đã cập nhật hoặc `404 Not Found`.

*   **1.5. Kiểm thử DELETE WalkDifficulty:**
    *   Lấy một `Id` của độ khó hành trình hiện có.
    *   Mở endpoint `DELETE /api/WalkDifficulties/{id}`.
    *   Nhập `Id` vào trường tham số và nhấp "Execute".
    *   Bạn sẽ nhận được `200 OK` với DTO của độ khó hành trình đã bị xóa hoặc `404 Not Found`.

### 2. Quản lý Mã nguồn với Git

Sau khi kiểm thử thành công, hãy lưu trữ công việc của bạn vào hệ thống kiểm soát phiên bản Git.

```bash
git add .
git commit -m "feat(walkdifficulty): Implement full CRUD for WalkDifficulty API with DI, Repository, and DTOs"
git push origin main
```

> [!TIP]
> **Antigravity IDE và Git:**
> Antigravity IDE, với khả năng đọc và ghi file, cũng có thể tích hợp chặt chẽ với Git. Sau khi bạn hoàn thành một tác vụ bằng Vibe Coding, Antigravity có thể tự động dàn dựng các thay đổi (`git add .`), đề xuất một thông điệp commit ý nghĩa dựa trên các thay đổi đã thực hiện (ví dụ: "feat(walkdifficulty): Implement full CRUD API"), và thậm chí thực hiện `git commit` và `git push` cho bạn sau khi xác nhận. Điều này giúp duy trì lịch sử phiên bản sạch sẽ và nhất quán mà không làm gián đoạn luồng làm việc của bạn.

---

## VI. Tóm tắt và Tầm nhìn Tương lai

Trong Phần 27 này, chúng ta đã đi từ lý thuyết đến thực hành, xây dựng một API hoàn chỉnh cho tài nguyên `WalkDifficulty` với ASP.NET Core. Bạn đã củng cố các kỹ năng quan trọng:

*   **Repository Pattern:** Đã thiết kế và triển khai `IWalkDifficultyRepository` và `WalkDifficultyRepository` để tách biệt logic truy cập dữ liệu, tăng tính mô đun và khả năng kiểm thử.
*   **Dependency Injection:** Đã đăng ký và sử dụng `IWalkDifficultyRepository` và `IMapper` trong `WalkDifficultiesController`, minh họa cách quản lý phụ thuộc hiệu quả.
*   **Domain Models và DTOs:** Đã phân biệt rõ ràng giữa Domain Model (`WalkDifficulty`) và các Data Transfer Objects (`WalkDifficultyDto`, `AddWalkDifficultyRequestDto`, `UpdateWalkDifficultyRequestDto`) để đảm bảo tính linh hoạt, bảo mật và hiệu quả của API.
*   **AutoMapper:** Đã sử dụng AutoMapper để đơn giản hóa quá trình ánh xạ giữa Domain Models và DTOs, giảm thiểu mã lặp lại.
*   **RESTful Controller:** Đã tạo `WalkDifficultiesController` với đầy đủ các endpoint HTTP GET, POST, PUT, DELETE để thực hiện các thao tác CRUD, tuân thủ các nguyên tắc REST.
*   **Lập trình Bất đồng bộ:** Đã áp dụng `async` và `await` để cải thiện hiệu suất và khả năng mở rộng của API, đặc biệt với các thao tác I/O.
*   **Kiểm thử với Swagger:** Đã kiểm thử tất cả các endpoint API thông qua giao diện Swagger UI, xác nhận tính đúng đắn của triển khai.
*   **Quản lý phiên bản:** Đã thực hành lưu trữ các thay đổi vào Git, duy trì lịch sử phát triển có tổ chức.

Phần này không chỉ trang bị cho bạn kiến thức thực tế để xây dựng API mà còn khuyến khích bạn suy nghĩ về kiến trúc phần mềm một cách sâu sắc hơn. Các mẫu thiết kế như Repository và DI không chỉ có lợi cho lập trình viên mà còn tạo ra một codebase có cấu trúc tốt, dễ hiểu và dễ mở rộng cho các hệ thống tự động hóa và AI như Antigravity IDE. Bằng cách áp dụng tư duy Vibe Coding, bạn có thể tận dụng những công cụ này để tập trung vào thiết kế cấp cao, để AI xử lý các chi tiết triển khai, từ đó tăng tốc độ phát triển và chất lượng mã nguồn.

<!-- REVIEWED_BY_AGENT -->
