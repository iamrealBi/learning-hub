# Phần 4: Xây dựng API cho Tài nguyên "Vùng" (Regions) - Chức năng Đọc (Read)

## 1. Giới thiệu: Nền tảng cho API Đọc dữ liệu

Trong phần này, chúng ta sẽ bắt tay vào xây dựng các API endpoint (điểm cuối API) đầu tiên cho tài nguyên "Vùng" (Regions) trong ứng dụng ASP.NET Core Web API của chúng ta. Trọng tâm chính là triển khai các chức năng đọc dữ liệu, bao gồm việc lấy danh sách tất cả các vùng và truy xuất thông tin chi tiết của một vùng cụ thể dựa trên định danh (ID) của nó. Chúng ta sẽ áp dụng các nguyên tắc cốt lõi của RESTful Web API, sử dụng các HTTP Verb phù hợp, tích hợp chặt chẽ Entity Framework Core để tương tác với cơ sở dữ liệu, và tận dụng triệt để Dependency Injection (DI) cùng Repository Pattern để quản lý các phụ thuộc và tách biệt các lớp một cách hiệu quả.

**Mục tiêu chính của phần này:**

*   **Thiết kế và triển khai Controller chuyên biệt:** Tạo một Controller (`RegionsController`) để xử lý mọi yêu cầu liên quan đến tài nguyên `Region`.
*   **Xây dựng Action Method cho truy vấn danh sách:** Tạo endpoint `GET /api/regions` để lấy tất cả các vùng.
*   **Xây dựng Action Method cho truy vấn theo ID:** Tạo endpoint `GET /api/regions/{id}` để lấy thông tin một vùng cụ thể.
*   **Tích hợp Dependency Injection:** Hiểu và áp dụng DI để đưa `DbContext` (hoặc Repository) vào Controller.
*   **Áp dụng Repository Pattern:** Tách biệt logic truy cập dữ liệu khỏi Controller, tăng tính kiểm thử và bảo trì.
*   **Tương tác với cơ sở dữ liệu:** Sử dụng Entity Framework Core để thực thi các truy vấn đọc dữ liệu.
*   **Xử lý phản hồi HTTP chuẩn RESTful:** Trả về các mã trạng thái HTTP phù hợp (ví dụ: `200 OK`, `404 Not Found`) để thông báo kết quả cho client.

## 2. Kiến trúc RESTful Web API và Vai trò của Controller

Để xây dựng một API mạnh mẽ và dễ bảo trì, việc hiểu rõ các nguyên tắc kiến trúc là vô cùng quan trọng.

### 2.1. RESTful API: Nguyên tắc và Thực hành

REST (Representational State Transfer) là một phong cách kiến trúc phần mềm được sử dụng rộng rãi để thiết kế các hệ thống mạng, đặc biệt là các dịch vụ web. Một API được coi là RESTful khi nó tuân thủ các nguyên tắc của REST, giúp API trở nên có khả năng mở rộng, linh hoạt và dễ hiểu.

Hãy hình dung một thư viện sách. Bạn không cần phải biết bên trong thư viện được tổ chức như thế nào (cách họ sắp xếp sách, hệ thống quản lý kho). Bạn chỉ cần biết cách yêu cầu một cuốn sách (tên sách, tác giả) và thư viện sẽ trả lại cho bạn cuốn sách đó hoặc thông báo rằng không có. API cũng vậy, client không cần biết logic nghiệp vụ phức tạp của server, chỉ cần biết cách gửi yêu cầu và nhận phản hồi.

Các nguyên tắc cốt lõi của REST bao gồm:

*   **Client-Server:** Tách biệt rõ ràng trách nhiệm giữa client (giao diện người dùng, nơi gửi yêu cầu) và server (nơi xử lý logic nghiệp vụ và dữ liệu). Điều này cho phép client và server phát triển độc lập.
*   **Stateless (Không trạng thái):** Mỗi yêu cầu từ client đến server phải chứa tất cả thông tin cần thiết để server hiểu và xử lý yêu cầu đó. Server không lưu trữ bất kỳ "trạng thái phiên" nào của client giữa các yêu cầu. Điều này giúp server dễ dàng mở rộng theo chiều ngang (scale horizontally) và cải thiện khả năng chịu lỗi.
    *   *Ví dụ thực tế:* Khi bạn mua hàng online, mỗi lần bạn thêm một sản phẩm vào giỏ hàng, server không "nhớ" bạn là ai nếu bạn không gửi kèm thông tin định danh (như session token hoặc JWT). Mọi yêu cầu đều độc lập.
*   **Cacheable (Có thể lưu trữ cache):** Phản hồi từ server có thể được đánh dấu là có thể lưu trữ cache hoặc không. Nếu một tài nguyên không thay đổi thường xuyên, client hoặc các proxy trung gian có thể lưu trữ bản sao của phản hồi, giúp giảm tải cho server và cải thiện hiệu suất cho client.
*   **Layered System (Hệ thống phân lớp):** Client không cần biết liệu nó có đang kết nối trực tiếp với server cuối cùng hay thông qua một tầng trung gian nào đó (ví dụ: load balancer, proxy, gateway API). Điều này tăng tính linh hoạt và bảo mật.
*   **Uniform Interface (Giao diện đồng nhất):** Đây là nguyên tắc quan trọng nhất và là xương sống của REST. Nó bao gồm bốn ràng buộc:
    *   **Resource Identification in Requests:** Các tài nguyên được xác định duy nhất bằng URI (Uniform Resource Identifier). Ví dụ: `/api/regions` cho danh sách vùng, `/api/regions/123` cho vùng có ID là 123.
    *   **Resource Manipulation Through Representations:** Client thao tác với tài nguyên thông qua các "biểu diễn" (representations) của chúng. Thông thường là JSON hoặc XML. Client gửi JSON để tạo/cập nhật tài nguyên, và server trả về JSON để biểu diễn trạng thái hiện tại của tài nguyên.
    *   **Self-descriptive Messages:** Mỗi thông báo (yêu cầu hoặc phản hồi) phải chứa đủ thông tin để mô tả cách xử lý thông báo đó. Điều này bao gồm các HTTP header (Content-Type, Accept) và mã trạng thái HTTP.
    *   **Hypermedia as the Engine of Application State (HATEOAS):** Client tương tác với server thông qua các siêu liên kết được cung cấp trong phản hồi. Ví dụ, khi bạn lấy thông tin một vùng, phản hồi có thể bao gồm liên kết đến các hành động liên quan như "cập nhật vùng này" hoặc "xóa vùng này". (Đây là một nguyên tắc nâng cao và thường ít được triển khai đầy đủ trong các API cơ bản).

> [!NOTE]
> **Tư duy Vibe Coding và Antigravity IDE trong thiết kế RESTful:**
> Khi bắt đầu thiết kế API, bạn có một "vibe" về các tài nguyên và hành động. Ví dụ, bạn muốn quản lý "Vùng" (Regions). Với Antigravity IDE, khi bạn bắt đầu nghĩ về `Region` và các thao tác CRUD, hệ thống Agentic AI này có thể tự động nhận diện "vibe" này. Nó có thể:
> *   **Gợi ý URI:** Tự động đề xuất các URI chuẩn RESTful như `/api/regions` (cho danh sách) và `/api/regions/{id}` (cho một mục cụ thể).
> *   **Phân loại HTTP Verb:** Đề xuất ngay lập tức `GET` cho chức năng đọc, `POST` cho tạo, `PUT`/`PATCH` cho cập nhật và `DELETE` cho xóa, dựa trên ngữ cảnh của tài nguyên.
> *   **Kiểm tra tính nhất quán:** Ngầm chạy các script để đảm bảo rằng tên tài nguyên là số nhiều, các URI tuân thủ quy ước, và các HTTP Verb được sử dụng đúng mục đích. Nếu bạn gõ `GET /api/region/all`, Antigravity có thể cảnh báo và gợi ý `GET /api/regions`.
> Điều này giúp học viên duy trì tính nhất quán và tuân thủ các nguyên tắc RESTful ngay từ đầu mà không cần phải ghi nhớ mọi quy tắc.

### 2.2. Controller trong ASP.NET Core: Trái tim của API Endpoint

Trong ASP.NET Core, `Controller` là một lớp đóng vai trò trung tâm trong việc xử lý các yêu cầu HTTP đến. Nó nhận yêu cầu từ client, điều phối logic nghiệp vụ (thường thông qua các service hoặc repository), và trả về một phản hồi HTTP phù hợp.

*   **`ControllerBase` vs `Controller`:**
    *   `ControllerBase`: Là lớp cơ sở cho các Controller API. Nó cung cấp các phương thức và thuộc tính cơ bản để xử lý yêu cầu HTTP và trả về các kết quả hành động (như `Ok()`, `NotFound()`, `BadRequest()`). `ControllerBase` không bao gồm các tính năng liên quan đến View, vì vậy nó lý tưởng cho Web API.
    *   `Controller`: Kế thừa từ `ControllerBase` và thêm các tính năng hỗ trợ View (ví dụ: `View()`, `PartialView()`), thường được sử dụng trong các ứng dụng MVC truyền thống.
*   **`[ApiController]` Attribute:** Đây là một attribute cực kỳ quan trọng và phải được áp dụng cho các lớp Controller API. Nó tự động kích hoạt một loạt các tính năng tiện lợi và cải thiện trải nghiệm phát triển API:
    *   **Model state validation:** Tự động kiểm tra `ModelState` và trả về phản hồi HTTP `400 Bad Request` với chi tiết lỗi nếu model state không hợp lệ (ví dụ: dữ liệu gửi lên không đúng định dạng hoặc thiếu trường bắt buộc). Điều này giúp giảm đáng kể boilerplate code kiểm tra lỗi.
    *   **Binding source inference:** Tự động suy luận nguồn dữ liệu cho các tham số của Action Method (ví dụ: từ route, query string, request body). Bạn không cần phải luôn luôn sử dụng `[FromRoute]`, `[FromQuery]`, `[FromBody]`.
    *   **Consumes attribute inference:** Tự động suy luận kiểu dữ liệu mà API có thể tiêu thụ (thường là `application/json`).
    *   **Problem Details for error status codes:** Khi trả về các mã trạng thái lỗi (ví dụ: 400, 404, 500), `[ApiController]` tự động định dạng phản hồi theo tiêu chuẩn [RFC 7807 (Problem Details for HTTP APIs)](https://tools.ietf.org/html/rfc7807), cung cấp thông tin lỗi nhất quán và dễ phân tích cho client.
*   **`[Route("api/[controller]")]` Attribute:** Attribute này định nghĩa mẫu URL cơ sở (route template) cho Controller.
    *   `api/`: Là một tiền tố phổ biến để chỉ ra rằng đây là một API endpoint, giúp phân biệt với các endpoint MVC.
    *   `[controller]`: Là một placeholder sẽ được thay thế bằng tên của Controller (bỏ đi hậu tố `Controller`). Ví dụ: `RegionsController` sẽ trở thành `regions`.
    *   Do đó, các endpoint trong `RegionsController` sẽ có dạng `api/regions/...`.

> [!NOTE]
> **Routing trong ASP.NET Core:**
> Khi một yêu cầu HTTP đến server, hệ thống routing của ASP.NET Core sẽ so khớp URL của yêu cầu với các route template đã được định nghĩa trong ứng dụng (trên Controller và các Action Method).
> *   **Route Template:** Là một chuỗi định nghĩa cấu trúc của URL. Ví dụ: `"api/[controller]"` hoặc `"{id:Guid}"`.
> *   **Route Constraint:** Là các điều kiện được áp dụng cho các placeholder trong route template để đảm bảo giá trị khớp với một kiểu dữ liệu hoặc định dạng cụ thể. Ví dụ: `:Guid` yêu cầu giá trị phải là một GUID hợp lệ; `:int` yêu cầu phải là một số nguyên. Điều này giúp ngăn chặn các yêu cầu không hợp lệ ngay từ tầng routing và tăng cường bảo mật.

### 2.3. HTTP Verbs và Mã Trạng thái HTTP: Ngôn ngữ của Web

RESTful API sử dụng các động từ HTTP (HTTP Verbs) để biểu thị loại thao tác mà client muốn thực hiện trên tài nguyên, và các mã trạng thái HTTP để thông báo kết quả của yêu cầu.

**HTTP Verbs (Methods):**

*   **`GET`:** Đọc (lấy) tài nguyên. Không có tác dụng phụ (idempotent và safe).
    *   *Ví dụ:* Lấy danh sách tất cả sản phẩm, lấy thông tin chi tiết của một người dùng.
*   **`POST`:** Tạo tài nguyên mới. Thường không idempotent (gửi nhiều lần sẽ tạo nhiều tài nguyên).
    *   *Ví dụ:* Đăng ký tài khoản mới, tạo một bài viết.
*   **`PUT`:** Cập nhật toàn bộ tài nguyên đã tồn tại, hoặc tạo mới nếu chưa tồn tại. Idempotent (gửi nhiều lần với cùng dữ liệu sẽ không thay đổi trạng thái server sau lần đầu).
    *   *Ví dụ:* Cập nhật toàn bộ thông tin hồ sơ người dùng.
*   **`PATCH`:** Cập nhật một phần tài nguyên. Thường không idempotent.
    *   *Ví dụ:* Cập nhật chỉ số điện thoại của người dùng, thay đổi trạng thái đơn hàng.
*   **`DELETE`:** Xóa tài nguyên. Idempotent.
    *   *Ví dụ:* Xóa một bài viết, hủy kích hoạt tài khoản.

Trong phần này, chúng ta sẽ tập trung vào động từ `GET`.

**Mã Trạng thái HTTP (HTTP Status Codes):**

Các mã trạng thái HTTP là các con số ba chữ số được server trả về để cho client biết kết quả của yêu cầu.

*   **`2xx Success` (Thành công):**
    *   **`200 OK`:** Yêu cầu đã được xử lý thành công. Đây là mã mặc định cho `GET`, `PUT`, `PATCH`, `DELETE` khi thành công và có nội dung trả về.
    *   **`201 Created`:** Tài nguyên mới đã được tạo thành công. Thường dùng cho `POST` và phản hồi thường bao gồm URI của tài nguyên mới.
    *   **`204 No Content`:** Yêu cầu đã được xử lý thành công nhưng không có nội dung để trả về. Thường dùng cho `PUT`, `PATCH`, `DELETE` khi không cần trả về dữ liệu.
*   **`4xx Client Error` (Lỗi từ phía client):**
    *   **`400 Bad Request`:** Yêu cầu không hợp lệ (ví dụ: dữ liệu đầu vào sai định dạng, thiếu trường bắt buộc).
    *   **`401 Unauthorized`:** Client chưa được xác thực (chưa đăng nhập hoặc token không hợp lệ).
    *   **`403 Forbidden`:** Client đã được xác thực nhưng không có quyền truy cập vào tài nguyên hoặc hành động này.
    *   **`404 Not Found`:** Tài nguyên yêu cầu không tồn tại trên server.
    *   **`409 Conflict`:** Xảy ra xung đột khi cố gắng tạo hoặc cập nhật tài nguyên (ví dụ: tạo một tài nguyên với ID đã tồn tại).
*   **`5xx Server Error` (Lỗi từ phía server):**
    *   **`500 Internal Server Error`:** Lỗi không mong muốn xảy ra ở phía server khi xử lý yêu cầu.

Chúng ta sẽ sử dụng `200 OK` và `404 Not Found` trong phần này.

## 3. Chuẩn bị Controller: `RegionsController`

Để bắt đầu, chúng ta cần tạo một Controller mới để quản lý các yêu cầu liên quan đến tài nguyên `Region`.

### 3.1. Các bước tạo Controller

1.  Trong Solution Explorer của Visual Studio, click chuột phải vào thư mục `Controllers` trong dự án Web API của bạn.
2.  Chọn `Add` > `Controller...`.
3.  Trong cửa sổ `Add New Scaffolded Item` (hoặc `Add New Item` > `API Controller - Empty`), chọn `API` > `API Controller - Empty`.
4.  Click `Add`.
5.  Đặt tên cho Controller là `RegionsController.cs`. (Lưu ý: ASP.NET Core yêu cầu tên Controller phải kết thúc bằng hậu tố `Controller` để hệ thống routing có thể nhận diện).
6.  Click `Add`.

ASP.NET Core sẽ tạo ra một file `RegionsController.cs` với cấu trúc cơ bản như sau:

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace NewZealandWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RegionsController : ControllerBase
    {
        // Các Action Method sẽ được định nghĩa ở đây
    }
}
```

### 3.2. Dependency Injection (DI) và IoC Container: Nền tảng của ứng dụng hiện đại

Để Controller có thể tương tác với cơ sở dữ liệu hoặc các dịch vụ khác, nó cần các "phụ thuộc" (dependencies). Thay vì để Controller tự tạo ra các phụ thuộc này, chúng ta sử dụng Dependency Injection (DI).

#### 3.2.1. DI là gì? Inversion of Control (IoC)

> **Dependency Injection (DI)** là một kỹ thuật thiết kế phần mềm, một dạng cụ thể của nguyên tắc **Inversion of Control (IoC)**. Thay vì một đối tượng (gọi là "client") tự tạo ra hoặc tìm kiếm các đối tượng mà nó phụ thuộc vào (gọi là "service"), các đối tượng phụ thuộc này sẽ được "tiêm" (inject) vào client từ bên ngoài. Việc tiêm thường được thực hiện thông qua constructor, setter property, hoặc method.
>
> **Inversion of Control (IoC)** có nghĩa là "đảo ngược quyền điều khiển". Theo cách truyền thống, một đối tượng sẽ tự điều khiển việc tạo và quản lý các phụ thuộc của nó. Với IoC, quyền điều khiển này được chuyển giao cho một "bên thứ ba" – thường là một **IoC Container** (hoặc DI Container). IoC Container chịu trách nhiệm khởi tạo các đối tượng và tiêm các phụ thuộc cần thiết vào chúng.
>
> *   **Ví dụ minh họa:**
>     *   **Không dùng DI:** Một chiếc xe hơi tự sản xuất động cơ của nó. Nếu muốn thay động cơ, phải thay đổi mã nguồn của xe hơi.
>     *   **Dùng DI:** Một chiếc xe hơi được thiết kế để nhận động cơ từ bên ngoài. Khi lắp ráp, nhà máy (IoC Container) sẽ chọn một động cơ phù hợp và lắp vào xe. Bạn có thể dễ dàng thay đổi loại động cơ (ví dụ: xăng sang điện) mà không cần thay đổi thiết kế cơ bản của xe hơi.

#### 3.2.2. Lợi ích của DI:

*   **Loose Coupling (Giảm sự phụ thuộc):** Các thành phần ít phụ thuộc vào nhau hơn. Controller không cần biết cách khởi tạo `DbContext`; nó chỉ yêu cầu một `DbContext` và IoC Container sẽ cung cấp. Điều này giúp dễ dàng thay thế hoặc thay đổi một thành phần mà không ảnh hưởng đến các thành phần khác.
*   **Testability (Khả năng kiểm thử):** Đây là một lợi ích lớn. Trong các bài kiểm thử đơn vị (unit test), bạn có thể dễ dàng giả lập (mock) các phụ thuộc. Ví dụ, thay vì sử dụng `DbContext` thật để truy vấn database, bạn có thể tiêm một `DbContext` giả lập trả về dữ liệu cố định, giúp kiểm thử logic của Controller một cách độc lập và nhanh chóng.
*   **Maintainability (Dễ bảo trì):** Mã dễ hiểu và dễ quản lý hơn vì các phụ thuộc được khai báo rõ ràng.
*   **Extensibility (Khả năng mở rộng):** Dễ dàng thêm các chức năng mới hoặc thay đổi cách thức hoạt động của các thành phần bằng cách đăng ký các triển khai khác nhau của cùng một interface trong IoC Container.
*   **Code Reusability:** Các phụ thuộc có thể được tái sử dụng trên nhiều thành phần khác nhau.

#### 3.2.3. Cấu hình và Sử dụng DI cho `DbContext` (Constructor Injection)

ASP.NET Core có sẵn một IoC Container tích hợp. Để sử dụng `NewZealandWalksDbContext` trong Controller, bạn cần đảm bảo rằng nó đã được đăng ký với DI Container trong `Program.cs` (hoặc `Startup.cs` ở các phiên bản cũ hơn). Nếu bạn đã cấu hình Entity Framework Core, dòng code sau đây sẽ quen thuộc:

```csharp
// Trong Program.cs
builder.Services.AddDbContext<NewZealandWalksDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("NewZealandWalksConnectionString")));
```

Sau khi `DbContext` được đăng ký, bạn có thể "tiêm" nó vào `RegionsController` thông qua constructor (Constructor Injection):

1.  Mở file `RegionsController.cs`.
2.  Tạo một trường `private readonly` để lưu trữ thể hiện của `DbContext`.
3.  Tạo một constructor nhận `NewZealandWalksDbContext` làm tham số. IoC Container sẽ tự động cung cấp thể hiện này khi khởi tạo `RegionsController`.

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using NewZealandWalks.API.Data; // Import namespace của DbContext

namespace NewZealandWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RegionsController : ControllerBase
    {
        private readonly NewZealandWalksDbContext _dbContext; // Khai báo trường private

        // Constructor để inject NewZealandWalksDbContext
        public RegionsController(NewZealandWalksDbContext dbContext)
        {
            _dbContext = dbContext; // Gán thể hiện DbContext được inject vào trường
        }

        // Các Action Method sẽ được định nghĩa ở đây
    }
}
```

> [!TIP]
> **Tư duy Vibe Coding và Antigravity IDE với Dependency Injection:**
> Khi bạn bắt đầu viết một Controller và cần truy cập dữ liệu, Antigravity IDE có thể nhận diện "vibe" này.
> *   **Tự động gợi ý Constructor Injection:** Nếu bạn khai báo một trường `private readonly` cho `DbContext` (hoặc một Repository interface) và chưa có constructor, Antigravity có thể tự động gợi ý và tạo constructor với tham số `dbContext` và gán nó.
> *   **Kiểm tra đăng ký DI:** Antigravity có thể chạy một "script ngầm" để kiểm tra `Program.cs` (hoặc `Startup.cs`) và xác nhận rằng `NewZealandWalksDbContext` đã được đăng ký trong DI container. Nếu chưa, nó có thể đề xuất dòng code cần thiết để đăng ký, giúp bạn tránh lỗi runtime do thiếu phụ thuộc.
> *   **Trực quan hóa Dependency Graph:** Đối với các dự án lớn, Antigravity có thể hiển thị một biểu đồ các phụ thuộc, giúp bạn hiểu rõ hơn cách các thành phần tương tác và đảm bảo không có vòng lặp phụ thuộc.

### 3.3. Áp dụng Repository Pattern để Tách biệt Lớp Truy cập Dữ liệu

Trong các ứng dụng lớn hơn hoặc khi bạn muốn có sự tách biệt rõ ràng giữa logic nghiệp vụ và logic truy cập dữ liệu, việc inject trực tiếp `DbContext` vào Controller không phải là cách tốt nhất. Thay vào đó, chúng ta thường sử dụng **Repository Pattern**.

#### 3.3.1. Tại sao cần Repository Pattern?

Repository Pattern là một mẫu thiết kế giúp trừu tượng hóa lớp truy cập dữ liệu. Nó tạo ra một lớp trung gian giữa domain model và các lớp ánh xạ dữ liệu.

*   **Loose Coupling:** Controller không còn phụ thuộc trực tiếp vào Entity Framework Core (`DbContext`). Thay vào đó, nó phụ thuộc vào một interface của Repository. Điều này có nghĩa là bạn có thể thay đổi công nghệ truy cập dữ liệu (ví dụ: từ EF Core sang Dapper hoặc một ORM khác) mà không cần sửa đổi Controller.
*   **Testability:** Dễ dàng kiểm thử Controller hơn nhiều. Trong unit test, bạn có thể tạo một mock (giả lập) của Repository interface để trả về dữ liệu cố định, mà không cần phải thiết lập một cơ sở dữ liệu thật.
*   **Separation of Concerns (Tách biệt các mối quan tâm):** Controller chỉ tập trung vào xử lý yêu cầu HTTP và điều phối logic nghiệp vụ. Repository tập trung vào việc lưu trữ và truy xuất dữ liệu. Điều này làm cho mã nguồn dễ hiểu, dễ bảo trì và dễ mở rộng hơn.
*   **Domain-Driven Design (DDD):** Repository là một khái niệm quan trọng trong DDD, giúp quản lý các tập hợp đối tượng domain (aggregate roots) và đảm bảo tính nhất quán của chúng.

#### 3.3.2. Thiết kế Interface `IRegionRepository`

Đầu tiên, chúng ta sẽ định nghĩa một interface cho Repository. Interface này sẽ khai báo các phương thức mà Controller cần để tương tác với dữ liệu vùng.

1.  Trong thư mục gốc của dự án API, tạo một thư mục mới tên là `Repositories`.
2.  Trong thư mục `Repositories`, tạo một interface mới tên là `IRegionRepository.cs`.

```csharp
// Repositories/IRegionRepository.cs
using NewZealandWalks.API.Models.Domain;

namespace NewZealandWalks.API.Repositories
{
    public interface IRegionRepository
    {
        Task<List<Region>> GetAllAsync(); // Phương thức bất đồng bộ để lấy tất cả vùng
        Task<Region?> GetByIdAsync(Guid id); // Phương thức bất đồng bộ để lấy vùng theo ID
    }
}
```

*   Chúng ta sử dụng `Task<List<Region>>` và `Task<Region?>` để chỉ ra rằng các phương thức này là bất đồng bộ (asynchronous). Điều này rất quan trọng trong các ứng dụng web để cải thiện khả năng mở rộng (scalability) bằng cách giải phóng thread của server trong khi chờ đợi các thao tác I/O (như truy vấn database) hoàn tất.
*   `Region?` sử dụng C# 8 Nullable Reference Types, cho phép phương thức trả về `null` nếu không tìm thấy vùng.

#### 3.3.3. Triển khai `SQLRegionRepository`

Tiếp theo, chúng ta sẽ tạo một lớp triển khai interface `IRegionRepository` bằng Entity Framework Core.

1.  Trong thư mục `Repositories`, tạo một lớp mới tên là `SQLRegionRepository.cs`.

```csharp
// Repositories/SQLRegionRepository.cs
using Microsoft.EntityFrameworkCore;
using NewZealandWalks.API.Data;
using NewZealandWalks.API.Models.Domain;

namespace NewZealandWalks.API.Repositories
{
    public class SQLRegionRepository : IRegionRepository
    {
        private readonly NewZealandWalksDbContext _dbContext;

        public SQLRegionRepository(NewZealandWalksDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task<List<Region>> GetAllAsync()
        {
            return await _dbContext.Regions.ToListAsync();
        }

        public async Task<Region?> GetByIdAsync(Guid id)
        {
            return await _dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id);
        }
    }
}
```

*   Lớp `SQLRegionRepository` nhận `NewZealandWalksDbContext` thông qua constructor injection.
*   `GetAllAsync()` sử dụng `_dbContext.Regions.ToListAsync()` để truy vấn tất cả các vùng một cách bất đồng bộ.
*   `GetByIdAsync()` sử dụng `_dbContext.Regions.FirstOrDefaultAsync(x => x.Id == id)` để tìm một vùng theo ID một cách bất đồng bộ. `FirstOrDefaultAsync` là lựa chọn tốt hơn `Find` khi bạn muốn linh hoạt hơn trong điều kiện tìm kiếm hoặc khi bạn cần bao gồm các mối quan hệ (ví dụ: `Include()`), và nó cũng hỗ trợ bất đồng bộ.

#### 3.3.4. Đăng ký Repository với DI Container

Để IoC Container biết cách cung cấp thể hiện của `IRegionRepository` khi được yêu cầu, chúng ta cần đăng ký nó trong `Program.cs`. Chúng ta sẽ đăng ký nó với `Scoped` lifetime, có nghĩa là một thể hiện mới của `SQLRegionRepository` sẽ được tạo cho mỗi yêu cầu HTTP.

```csharp
// Trong Program.cs, sau khi đăng ký DbContext
// Đăng ký Repository Pattern
builder.Services.AddScoped<IRegionRepository, SQLRegionRepository>();
```

#### 3.3.5. Cập nhật `RegionsController` để sử dụng Repository

Bây giờ, thay vì inject `NewZealandWalksDbContext` trực tiếp, chúng ta sẽ inject `IRegionRepository` vào `RegionsController`.

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using NewZealandWalks.API.Repositories; // Import namespace của Repository

namespace NewZealandWalks.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RegionsController : ControllerBase
    {
        private readonly IRegionRepository _regionRepository; // Thay đổi từ DbContext sang Repository interface

        // Constructor để inject IRegionRepository
        public RegionsController(IRegionRepository regionRepository)
        {
            _regionRepository = regionRepository;
        }

        // Các Action Method sẽ được định nghĩa ở đây
    }
}
```

> [!NOTE]
> **Tư duy Vibe Coding và Antigravity IDE với Repository Pattern:**
> Khi bạn bắt đầu viết logic truy cập dữ liệu trong Controller, Antigravity IDE có thể nhận diện "vibe" rằng bạn đang vi phạm nguyên tắc tách biệt mối quan tâm.
> *   **Gợi ý áp dụng Repository:** Antigravity có thể tự động đề xuất việc tạo một Repository interface và một lớp triển khai, cùng với các phương thức `GetAllAsync` và `GetByIdAsync` dựa trên các truy vấn bạn đang viết.
> *   **Tự động tạo boilerplate:** Với Antigravity, bạn có thể chỉ cần "vibe" rằng bạn muốn một Repository cho `Region`. Hệ thống có thể tự động tạo `IRegionRepository.cs`, `SQLRegionRepository.cs` với các phương thức cơ bản, và thậm chí cập nhật `Program.cs` để đăng ký nó.
> *   **Refactor tự động:** Nếu bạn đã có code truy cập `DbContext` trực tiếp trong Controller, Antigravity có thể chạy một "subagent trình duyệt" để phân tích mã và sau đó đề xuất refactor để sử dụng Repository, tự động cập nhật cả Controller và `Program.cs`.
> *   **Kiểm tra tính nhất quán:** Nó có thể kiểm tra xem tất cả các Controller có sử dụng Repository cho việc truy cập dữ liệu hay không, đảm bảo tuân thủ kiến trúc đã chọn.

## 4. Xây dựng Action Method: Lấy Tất cả Vùng (`GetAllRegions`)

Action method này sẽ chịu trách nhiệm trả về một danh sách tất cả các đối tượng `Region` có trong cơ sở dữ liệu.

### 4.1. Định nghĩa Endpoint

1.  **HTTP Verb:** Sử dụng `[HttpGet]` để chỉ định rằng đây là một endpoint `GET`.
2.  **Kiểu trả về:** `IActionResult` là một interface linh hoạt cho phép chúng ta trả về nhiều loại phản hồi HTTP khác nhau (ví dụ: `Ok()`, `NotFound()`, `BadRequest()`).
3.  **Async/Await:** Chúng ta sẽ sử dụng `async` và `await` để đảm bảo hoạt động I/O (truy vấn database) không chặn luồng chính của ứng dụng, cải thiện hiệu suất và khả năng mở rộng.

```csharp
// Trong RegionsController.cs
// GET ALL REGIONS
// GET: https://localhost:{portnumber}/api/regions
[HttpGet]
public async Task<IActionResult> GetAllRegions()
{
    // Lấy dữ liệu từ Repository một cách bất đồng bộ
    var regions = await _regionRepository.GetAllAsync();

    // Trả về phản hồi HTTP 200 OK cùng với danh sách các vùng
    return Ok(regions);
}
```

### 4.2. Giải thích Code

*   `[HttpGet]`: Attribute này đánh dấu phương thức là một Action Method xử lý các yêu cầu HTTP `GET`. Vì không có route template cụ thể nào được cung cấp ở đây, nó sẽ sử dụng route template cơ sở của Controller (`api/regions`).
*   `public async Task<IActionResult> GetAllRegions()`:
    *   `async`: Chỉ ra rằng phương thức này là bất đồng bộ và có thể sử dụng từ khóa `await`.
    *   `Task<IActionResult>`: Phương thức bất đồng bộ trả về một `Task` mà khi hoàn thành sẽ cung cấp một `IActionResult`.
*   `var regions = await _regionRepository.GetAllAsync();`: Đây là dòng code quan trọng.
    *   `_regionRepository.GetAllAsync()`: Gọi phương thức bất đồng bộ từ Repository để lấy tất cả các vùng.
    *   `await`: Tạm dừng thực thi của phương thức `GetAllRegions` cho đến khi `GetAllAsync()` hoàn thành. Trong thời gian chờ đợi, thread của server được giải phóng để xử lý các yêu cầu khác, giúp tối ưu hóa tài nguyên.
*   `return Ok(regions);`: Trả về một phản hồi HTTP `200 OK` cùng với danh sách các vùng dưới dạng body của phản hồi (thường là JSON, được ASP.NET Core tự động serialize). `Ok()` là một helper method của `ControllerBase`.

### 4.3. Kiểm thử với Swagger

Khi bạn chạy ứng dụng (thường là với `dotnet run` hoặc qua Visual Studio), Swagger UI sẽ tự động khởi động và hiển thị tài liệu API của bạn. Bạn sẽ thấy một endpoint `GET /api/regions`.

1.  Mở trình duyệt và điều hướng đến Swagger UI (thường là `https://localhost:{portnumber}/swagger`).
2.  Tìm và mở rộng endpoint `GET /api/regions`.
3.  Click vào nút `Try it out`.
4.  Click vào nút `Execute`.

Nếu cơ sở dữ liệu của bạn có dữ liệu trong bảng `Regions`, bạn sẽ nhận được phản hồi `200 OK` với danh sách các vùng. Nếu bảng trống, bạn sẽ nhận được một mảng JSON rỗng (`[]`).

## 5. Xây dựng Action Method: Lấy Vùng theo ID (`GetRegionById`)

Action method này sẽ lấy một ID làm tham số từ URL và trả về thông tin chi tiết của vùng tương ứng. Nếu không tìm thấy vùng nào với ID đó, nó sẽ trả về lỗi `404 Not Found`.

### 5.1. Định nghĩa Endpoint với Route Constraint

1.  **HTTP Verb:** Vẫn là `[HttpGet]`, nhưng cần chỉ định thêm `route template` để nhận ID.
2.  **`Route Template`:** `[HttpGet("{id:Guid}")]` cho biết phương thức này sẽ xử lý các yêu cầu `GET` đến `api/regions/{id}`.
    *   `{id}`: Là một placeholder cho giá trị ID.
    *   `:Guid`: Là một `route constraint` quan trọng. Nó đảm bảo rằng giá trị ID được truyền vào phải là một GUID hợp lệ. Nếu client gửi một chuỗi không phải là GUID, yêu cầu sẽ không được định tuyến đến Action Method này mà sẽ trả về `404 Not Found` hoặc lỗi routing, giúp ngăn chặn các yêu cầu với ID không đúng định dạng.
3.  **Tham số phương thức:** `[FromRoute] Guid id` chỉ định rằng giá trị của tham số `id` sẽ được lấy từ phần route của URL. Mặc dù `[ApiController]` có thể suy luận điều này, việc sử dụng `[FromRoute]` giúp mã rõ ràng hơn.

```csharp
// Trong RegionsController.cs
// GET REGION BY ID
// GET: https://localhost:{portnumber}/api/regions/{id}
[HttpGet]
[Route("{id:Guid}")] // Route template để nhận ID với route constraint
public async Task<IActionResult> GetRegionById([FromRoute] Guid id)
{
    // Lấy dữ liệu từ Repository một cách bất đồng bộ
    var region = await _regionRepository.GetByIdAsync(id);

    if (region == null)
    {
        // Trả về 404 Not Found nếu không tìm thấy vùng
        return NotFound();
    }

    // Trả về 200 OK cùng với thông tin chi tiết của vùng
    return Ok(region);
}
```

### 5.2. Giải thích Code

*   `[HttpGet]`: Đánh dấu phương thức là một Action Method xử lý các yêu cầu HTTP `GET`.
*   `[Route("{id:Guid}")]`: Định nghĩa route template cụ thể cho phương thức này. Nó sẽ khớp với các URL như `api/regions/a1b2c3d4-e5f6-7890-1234-567890abcdef`.
*   `public async Task<IActionResult> GetRegionById([FromRoute] Guid id)`:
    *   `[FromRoute] Guid id`: Cho ASP.NET Core biết rằng tham số `id` của phương thức này sẽ được lấy từ phần route của URL.
*   `var region = await _regionRepository.GetByIdAsync(id);`: Gọi phương thức bất đồng bộ từ Repository để tìm một vùng theo ID.
*   `if (region == null) { return NotFound(); }`: Kiểm tra xem vùng có được tìm thấy hay không. Nếu `_regionRepository.GetByIdAsync(id)` trả về `null` (vì không có vùng nào khớp với ID), chúng ta trả về phản hồi `404 Not Found`. `NotFound()` là một helper method của `ControllerBase`.
*   `return Ok(region);`: Nếu tìm thấy, trả về phản hồi `200 OK` cùng với đối tượng `Region` đó.

> [!TIP]
> **`Find()` vs `FirstOrDefault()` trong Entity Framework Core:**
> *   **`Find(id)`:** Phương thức này được tối ưu để tìm kiếm một entity theo khóa chính (Primary Key). Nó sẽ tìm trong cache của `DbContext` trước, sau đó mới truy vấn cơ sở dữ liệu nếu không tìm thấy trong cache. `Find()` là đồng bộ và không hỗ trợ `async/await` trực tiếp.
> *   **`FirstOrDefault(predicate)` / `FirstOrDefaultAsync(predicate)`:** Phương thức này linh hoạt hơn. Nó cho phép bạn tìm kiếm theo bất kỳ thuộc tính nào của entity bằng cách cung cấp một biểu thức lambda (predicate). `FirstOrDefaultAsync` là phiên bản bất đồng bộ và là lựa chọn tốt khi bạn tìm kiếm không phải bằng khóa chính, hoặc muốn linh hoạt hơn trong điều kiện tìm kiếm và có thể `Include` các mối quan hệ.
> Trong ngữ cảnh của Repository Pattern và bất đồng bộ, `FirstOrDefaultAsync` thường là lựa chọn ưu tiên hơn khi truy vấn từ database.

### 5.3. Kiểm thử với Swagger

1.  Chạy ứng dụng.
2.  Mở Swagger UI. Bạn sẽ thấy endpoint `GET /api/regions/{id}`.
3.  Mở rộng endpoint này và click `Try it out`.
4.  **Trường hợp 1: ID tồn tại.**
    *   Nhập một `Guid` của một vùng hiện có trong cơ sở dữ liệu vào trường `id`. (Bạn có thể lấy một ID từ kết quả của `GET /api/regions`).
    *   Click `Execute`.
    *   Bạn sẽ nhận được phản hồi `200 OK` với thông tin chi tiết của vùng.
5.  **Trường hợp 2: ID không tồn tại.**
    *   Nhập một `Guid` hợp lệ nhưng không tồn tại trong cơ sở dữ liệu (ví dụ: thay đổi một vài ký tự cuối của một ID có sẵn, hoặc tạo một GUID mới hoàn toàn).
    *   Click `Execute`.
    *   Bạn sẽ nhận được phản hồi `404 Not Found`.
6.  **Trường hợp 3: ID không hợp lệ (không phải GUID).**
    *   Nhập một chuỗi không phải là GUID (ví dụ: `abc`).
    *   Click `Execute`.
    *   Do `route constraint :Guid`, yêu cầu này có thể không khớp với Action Method của bạn và Swagger có thể báo lỗi hoặc bạn sẽ nhận được `404 Not Found` từ hệ thống routing trước khi nó đến được Controller.

> [!NOTE]
> **Tư duy Vibe Coding và Antigravity IDE trong kiểm thử và xử lý lỗi:**
> Khi bạn phát triển các Action Method như `GetRegionById`, Antigravity IDE có thể hỗ trợ mạnh mẽ:
> *   **Gợi ý xử lý lỗi:** Khi bạn viết `var region = await _regionRepository.GetByIdAsync(id);`, Antigravity có thể nhận diện "vibe" rằng `region` có thể là `null` và tự động gợi ý khối `if (region == null) { return NotFound(); }`.
> *   **Tạo test case tự động:** Dựa trên các endpoint bạn vừa tạo, Antigravity có thể tự động tạo các test case cơ bản cho unit tests hoặc integration tests. Ví dụ:
>     *   Test case cho `GetAllRegions`: Đảm bảo trả về `200 OK` và một danh sách (có thể rỗng hoặc có dữ liệu mẫu).
>     *   Test case cho `GetRegionById`:
>         *   Trả về `200 OK` khi ID tồn tại.
>         *   Trả về `404 Not Found` khi ID không tồn tại.
>         *   Test trường hợp ID không hợp lệ (nhờ `route constraint`).
> *   **Chạy script ngầm để kiểm tra:** Antigravity có thể chạy các script ngầm để kiểm tra các quy ước về mã trạng thái HTTP. Ví dụ, nó có thể cảnh báo nếu bạn trả về `200 OK` cho một thao tác `DELETE` mà không có nội dung, và gợi ý `204 No Content`.

## 6. Tóm tắt và Bước tiếp theo

Trong phần này, chúng ta đã đặt nền móng vững chắc cho ứng dụng RESTful Web API bằng cách xây dựng các chức năng đọc dữ liệu cơ bản cho tài nguyên "Vùng" trong ASP.NET Core:

*   **Tạo `RegionsController`**: Đã thiết lập một Controller chuyên biệt, sử dụng các attribute `[ApiController]` và `[Route]` chuẩn RESTful.
*   **Tích hợp Dependency Injection**: Đã hiểu sâu hơn về DI và IoC Container, và cách inject `IRegionRepository` vào Controller thông qua constructor, đảm bảo tính linh hoạt và khả năng kiểm thử.
*   **Áp dụng Repository Pattern**: Đã thiết kế và triển khai `IRegionRepository` và `SQLRegionRepository`, giúp tách biệt logic truy cập dữ liệu khỏi Controller, tăng cường khả năng bảo trì và mở rộng.
*   **Xây dựng `GetAllRegions` Action Method**: Đã triển khai một endpoint `GET /api/regions` để lấy về danh sách tất cả các vùng từ cơ sở dữ liệu một cách bất đồng bộ (`await _regionRepository.GetAllAsync()`).
*   **Xây dựng `GetRegionById` Action Method**: Đã triển khai một endpoint `GET /api/regions/{id}` để lấy thông tin của một vùng cụ thể theo ID. Chúng ta đã học cách sử dụng `[FromRoute]` và `route constraint` (`:Guid`), cũng như cách truy vấn dữ liệu bất đồng bộ bằng `_regionRepository.GetByIdAsync(id)`.
*   **Xử lý phản hồi HTTP**: Đã sử dụng `Ok()` để trả về `200 OK` khi thành công và `NotFound()` để trả về `404 Not Found` khi tài nguyên không tồn tại, tuân thủ các nguyên tắc RESTful.
*   **Kiểm thử với Swagger**: Đã sử dụng Swagger UI để dễ dàng kiểm thử các API endpoint vừa tạo trong các trường hợp thành công và thất bại.

Những kiến thức này là nền tảng vững chắc để tiếp tục phát triển các chức năng CRUD còn lại (Create, Update, Delete) cho API của chúng ta, đồng thời áp dụng các nguyên tắc thiết kế tốt nhất. Trong các phần tiếp theo, chúng ta sẽ mở rộng các kỹ thuật này để hoàn thiện bộ API cho tài nguyên "Vùng".

<!-- REVIEWED_BY_AGENT -->
