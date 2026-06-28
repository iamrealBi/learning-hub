# Phần 16: Triển khai ứng dụng Web API lên Azure

Trong kỷ nguyên phát triển phần mềm hiện đại, việc đưa ứng dụng lên môi trường đám mây là một yêu cầu tất yếu. Azure, nền tảng đám mây của Microsoft, cung cấp một hệ sinh thái mạnh mẽ và linh hoạt để lưu trữ, quản lý, và mở rộng các ứng dụng web, đặc biệt là các RESTful Web API được xây dựng bằng ASP.NET Core. Phần này sẽ cung cấp một lộ trình chi tiết, từng bước để triển khai một ứng dụng ASP.NET Core Web API (sử dụng Entity Framework Core, Dependency Injection, Repository Pattern, Controllers và HTTP Verbs) lên Azure. Chúng ta sẽ đi từ việc thiết lập các tài nguyên đám mây, cấu hình cơ sở dữ liệu, đến việc xử lý các thách thức triển khai thường gặp và kiểm tra hoạt động của API trên môi trường thực tế.

Mục tiêu chính là trang bị cho bạn kiến thức và kỹ năng cần thiết để tự tin chuyển đổi ứng dụng Web API của mình từ môi trường phát triển cục bộ sang một môi trường sản xuất trên Azure, đảm bảo tính sẵn sàng, bảo mật và khả năng mở rộng.

## 1. Chuẩn bị Môi trường Azure: Nền Tảng Cho Ứng Dụng Đám Mây

Trước khi triển khai ứng dụng, việc thiết lập các tài nguyên cơ bản trên Azure là bước đầu tiên và quan trọng nhất.

### 1.1. Azure Subscription (Đăng ký Azure): Đơn Vị Quản Lý và Thanh Toán

Một Azure Subscription là một đơn vị logic hóa các dịch vụ Azure, đóng vai trò là ranh giới quản lý và thanh toán cho tất cả các tài nguyên bạn tạo. Mọi tài nguyên, từ máy ảo đến cơ sở dữ liệu, đều thuộc về một đăng ký cụ thể.

> [!NOTE]
> Nếu bạn chưa có tài khoản Azure, bạn có thể đăng ký tài khoản miễn phí. Azure cung cấp bản dùng thử miễn phí với một số tín dụng (thường là 200 USD) và quyền truy cập miễn phí vào các dịch vụ phổ biến trong 12 tháng. Bạn sẽ cần cung cấp thông tin thẻ tín dụng để xác minh danh tính, nhưng sẽ không bị tính phí trừ khi bạn chủ động nâng cấp lên gói trả phí. Đây là một cơ hội tuyệt vời để khám phá Azure mà không tốn kém.

**Các bước thực hiện:**
1.  Truy cập [portal.azure.com](https://portal.azure.com/).
2.  Đăng nhập bằng tài khoản Microsoft của bạn. Nếu chưa có, hãy tạo một tài khoản mới.
3.  Tìm kiếm "Subscriptions" trong thanh tìm kiếm và chọn nó.
4.  Nhấp vào nút "Add" hoặc "Create" để tạo một đăng ký mới.
5.  Chọn loại đăng ký phù hợp (ví dụ: "Free trial" hoặc "Azure for Students").
6.  Hoàn tất quá trình đăng ký bằng cách cung cấp thông tin cá nhân và chi tiết thẻ tín dụng.
7.  Sau khi đăng ký thành công, bạn sẽ có một Subscription ở trạng thái "Active".

### 1.2. Resource Group (Nhóm Tài nguyên): Tổ Chức Tài Nguyên Một Cách Hợp Lý

Resource Group là một bộ sưu tập các tài nguyên Azure được nhóm lại với nhau một cách logic. Việc nhóm các tài nguyên liên quan (như App Service, SQL Database, Application Insights, v.v.) vào cùng một Resource Group mang lại nhiều lợi ích:
*   **Quản lý tập trung:** Dễ dàng xem, giám sát và cấu hình tất cả các tài nguyên của một ứng dụng.
*   **Vòng đời đồng bộ:** Khi bạn muốn xóa một ứng dụng, bạn chỉ cần xóa Resource Group, tất cả tài nguyên bên trong sẽ được gỡ bỏ.
*   **Kiểm soát truy cập:** Áp dụng các chính sách Azure Role-Based Access Control (RBAC) cho toàn bộ nhóm.

> [!TIP]
> Một quy ước đặt tên tốt cho Resource Group giúp dễ quản lý và nhận diện. Ví dụ: `RG-<TênỨngDụng>-<VịTrí>-<MôiTrường>-<SốPhiênBản>`. Đối với ứng dụng NZWalks trong môi trường phát triển ở vùng East US, chúng ta có thể đặt tên là `RG-NZWalks-EastUS-Dev-001`.

**Các bước thực hiện:**
1.  Từ trang chủ Azure Portal, tìm kiếm "Resource Groups" và chọn nó.
2.  Nhấp vào nút "Create".
3.  **Subscription:** Chọn Subscription của bạn.
4.  **Resource Group name:** Nhập tên cho Resource Group (ví dụ: `RG-NZWalks-EastUS-Dev-001`).
5.  **Region:** Chọn Vùng (Region) nơi bạn muốn triển khai tài nguyên. Việc chọn cùng một Region cho tất cả các tài nguyên liên quan giúp giảm độ trễ (latency) và chi phí truyền dữ liệu giữa chúng. Ví dụ: "East US".
6.  Nhấp vào "Review + create", sau đó "Create".

> [!NOTE]
> **Antigravity IDE và Vibe Coding:** Trong giai đoạn chuẩn bị môi trường, Antigravity IDE có thể hỗ trợ bạn tạo các tài nguyên này thông qua Azure CLI. Thay vì điều hướng thủ công trên portal, bạn có thể mô tả ý định của mình, và Antigravity sẽ gợi ý hoặc thực thi các lệnh như `az group create --name RG-NZWalks-EastUS-Dev-001 --location eastus`. Điều này giúp bạn duy trì "vibe" lập trình, tập trung vào code và cấu hình hơn là các thao tác UI lặp lại.

## 2. Triển khai Dịch vụ Ứng dụng Web API (Azure App Service): Nơi Ứng Dụng Của Bạn Sống

Azure App Service là một dịch vụ Nền tảng dưới dạng Dịch vụ (Platform as a Service - PaaS) mạnh mẽ, cho phép bạn xây dựng, triển khai và mở rộng các ứng dụng web, API, và backend di động mà không cần bận tâm đến việc quản lý cơ sở hạ tầng máy chủ. Nó là lựa chọn lý tưởng để host ASP.NET Core Web API của chúng ta, giúp chúng ta tập trung vào logic nghiệp vụ và mã nguồn.

### 2.1. Tạo Azure App Service

**Các bước thực hiện:**
1.  Từ trang chủ Azure Portal, tìm kiếm "App Services" và chọn nó.
2.  Nhấp vào nút "Create" -> "Web App".
3.  **Basic Tab:**
    *   **Subscription:** Chọn Subscription của bạn.
    *   **Resource Group:** Chọn Resource Group bạn đã tạo (ví dụ: `RG-NZWalks-EastUS-Dev-001`).
    *   **Name:** Đặt tên duy nhất cho Web App của bạn. Tên này sẽ trở thành một phần của URL công khai của ứng dụng (ví dụ: `app-NZWalks-EastUS-Dev-001.azurewebsites.net`). Quy ước đặt tên tốt: `app-<TênỨngDụng>-<VịTrí>-<MôiTrường>-<SốPhiênBản>` (ví dụ: `app-NZWalks-EastUS-Dev-001`).
    *   **Publish:** Chọn "Code" (vì chúng ta sẽ triển khai mã nguồn .NET trực tiếp).
    *   **Runtime stack:** Chọn phiên bản .NET mà ứng dụng của bạn sử dụng (ví dụ: ".NET 7 (STS)"). Đảm bảo phiên bản này khớp với Target Framework của dự án của bạn.
    *   **Operating System:** Chọn "Windows". Mặc dù ASP.NET Core có thể chạy trên Linux, Windows thường được ưu tiên khi bạn cần tích hợp sâu với các dịch vụ khác của Microsoft hoặc có các yêu cầu đặc biệt.
    *   **Region:** Chọn cùng Region với Resource Group (ví dụ: "East US").
    *   **App Service Plan:** Đây là thành phần quan trọng xác định tài nguyên tính toán (CPU, RAM) và giá cả cho ứng dụng của bạn.
        *   Bạn có thể tạo một App Service Plan mới hoặc chọn một plan hiện có.
        *   Để tiết kiệm chi phí cho mục đích demo, hãy chọn một gói giá "Free" (F1) hoặc "Dev/Test" (B1/D1). Đối với môi trường sản xuất, bạn sẽ cần các gói cao cấp hơn như "Standard" hoặc "Premium" để đảm bảo hiệu suất và khả năng mở rộng.
4.  **Monitoring Tab:** (Tùy chọn) Bạn có thể cấu hình Application Insights để giám sát hiệu suất và lỗi của ứng dụng. Đây là một công cụ mạnh mẽ để hiểu hành vi của ứng dụng trong môi trường sản xuất.
5.  Nhấp vào "Review + create", sau đó "Create".

Quá trình triển khai App Service có thể mất vài phút. Sau khi hoàn tất, bạn có thể truy cập tài nguyên App Service của mình. URL mặc định sẽ hiển thị một trang chào mừng của Azure, vì ứng dụng của bạn chưa được triển khai.

### 2.2. Hiểu về Cấu hình Azure App Service (Under the Hood)

Azure App Service cung cấp một hệ thống cấu hình mạnh mẽ, nơi bạn có thể định nghĩa các biến môi trường, chuỗi kết nối và cài đặt ứng dụng. Khi ứng dụng ASP.NET Core của bạn chạy trên App Service, nó sẽ tự động đọc các cài đặt này.

*   **Application Settings:** Được ánh xạ thành biến môi trường trong ứng dụng. Ví dụ, một cài đặt tên `MySetting` với giá trị `MyValue` sẽ trở thành biến môi trường `MySetting=MyValue`. ASP.NET Core có thể đọc chúng bằng `Configuration.GetValue<string>("MySetting")`.
*   **Connection Strings:** Được định nghĩa riêng biệt và được App Service inject vào ứng dụng dưới dạng biến môi trường có tiền tố đặc biệt. Ví dụ, một chuỗi kết nối tên `DefaultConnection` sẽ được inject thành `ConnectionStrings:DefaultConnection` hoặc `SQLCONNSTR_DefaultConnection`. ASP.NET Core, đặc biệt là Entity Framework Core, được thiết kế để đọc các chuỗi kết nối này một cách tự động khi bạn cấu hình `DbContext` với `UseSqlServer`.

> [!TIP]
> **Dependency Injection và Cấu hình:** Trong ứng dụng ASP.NET Core của bạn, bạn sử dụng Dependency Injection để inject các đối tượng cấu hình (`IConfiguration`) vào các dịch vụ. Khi triển khai lên Azure, `IConfiguration` sẽ tự động bao gồm các giá trị từ Azure App Service settings, giúp bạn quản lý cấu hình môi trường một cách linh hoạt mà không cần thay đổi mã nguồn.

## 3. Thiết lập Cơ sở Dữ liệu SQL Server trên Azure (Azure SQL Database): Nơi Dữ Liệu Của Bạn An Toàn

Ứng dụng Web API của chúng ta sử dụng Entity Framework Core để tương tác với cơ sở dữ liệu SQL Server. Khi triển khai lên Azure, chúng ta cần một cơ sở dữ liệu SQL Server có thể truy cập được từ ứng dụng đám mây. Azure SQL Database là một dịch vụ cơ sở dữ liệu quan hệ được quản lý hoàn toàn (fully managed), giúp đơn giản hóa việc quản lý, sao lưu, và mở rộng cơ sở dữ liệu, loại bỏ gánh nặng vận hành máy chủ SQL Server truyền thống.

### 3.1. Tạo Azure SQL Server và Azure SQL Database

Trước tiên, chúng ta cần tạo một máy chủ SQL Server logic trên Azure để chứa các cơ sở dữ liệu của chúng ta.

**Các bước thực hiện:**
1.  Từ trang chủ Azure Portal, tìm kiếm "SQL databases" và chọn nó.
2.  Nhấp vào nút "Create".
3.  **Basic Tab:**
    *   **Subscription:** Chọn Subscription của bạn.
    *   **Resource Group:** Chọn Resource Group bạn đã tạo.
    *   **Database name:** Đặt tên cho cơ sở dữ liệu của bạn (ví dụ: `NZWalksDB-Dev`).
    *   **Server:** Nhấp vào "Create new" để tạo một SQL Server logic mới.
        *   **Server name:** Đặt tên duy nhất cho SQL Server (ví dụ: `sqlserver-NZWalks-EastUS-Dev-001`). Tên này phải là duy nhất trên toàn cầu trong Azure.
        *   **Location:** Chọn cùng Region với App Service và Resource Group của bạn.
        *   **Authentication method:** Chọn "Use SQL authentication". Đây là phương pháp phổ biến để ứng dụng kết nối. Bạn cũng có thể cân nhắc "Use Microsoft Entra authentication" cho các kịch bản nâng cao hơn về bảo mật và quản lý danh tính.
        *   **Server admin login:** Tạo một tên đăng nhập quản trị viên (ví dụ: `sqladmin`).
        *   **Password:** Đặt mật khẩu mạnh và *ghi nhớ nó*. Đây sẽ là thông tin đăng nhập mà ứng dụng của bạn sẽ sử dụng để kết nối.
        *   Nhấp "OK".
    *   **Want to use SQL elastic pool?**: Chọn "No" cho mục đích demo. Elastic pool hữu ích khi bạn có nhiều cơ sở dữ liệu với nhu cầu tài nguyên biến động, cho phép chia sẻ tài nguyên tính toán.
    *   **Compute + storage:** Chọn gói cơ bản hoặc Dev/Test (ví dụ: Basic, 250 MB, 5 DTU) để tiết kiệm chi phí cho mục đích demo. DTU (Database Transaction Units) là một đơn vị đo lường tổng hợp cho CPU, I/O và bộ nhớ. Đối với môi trường sản xuất, bạn sẽ cần các gói cao cấp hơn như "General Purpose" hoặc "Business Critical" với vCores để có hiệu suất và khả năng mở rộng tốt hơn.
    *   **Backup storage redundancy:** Chọn "Locally-redundant backup storage" (LRS) để tiết kiệm chi phí cho demo. Trong sản xuất, hãy cân nhắc "Geo-redundant backup storage" (GRS) để tăng cường khả năng phục hồi sau thảm họa.
4.  Nhấp vào "Review + create", sau đó "Create".

Quá trình tạo SQL Server và cơ sở dữ liệu có thể mất một thời gian.

### 3.2. Cấu hình Mạng (Networking) cho SQL Server: Đảm Bảo Khả Năng Truy Cập An Toàn

Theo mặc định, Azure SQL Server có tường lửa bảo vệ, ngăn chặn các kết nối từ bên ngoài để đảm bảo an ninh. Chúng ta cần cấu hình tường lửa để cho phép App Service của chúng ta và máy tính cục bộ của chúng ta (nếu bạn muốn truy cập bằng SQL Server Management Studio hoặc Azure Data Studio) kết nối.

**Các bước thực hiện:**
1.  Sau khi SQL Server được tạo, truy cập tài nguyên SQL Server đó (tìm kiếm "SQL servers" và chọn server của bạn).
2.  Trong menu bên trái, chọn "Networking".
3.  **Public network access:** Chọn "Selected networks". Điều này đảm bảo chỉ các kết nối được phép mới có thể truy cập.
4.  **Firewall rules:**
    *   **Add your client IP address:** Nhấp vào "Add your current client IP address" để thêm địa chỉ IP công cộng của máy tính bạn. Điều này cho phép bạn kết nối từ máy cục bộ bằng các công cụ như SSMS hoặc Azure Data Studio để quản lý cơ sở dữ liệu.
    *   **Allow Azure services and resources to access this server:** Đảm bảo bạn đã kiểm tra hộp này. Đây là *cực kỳ quan trọng* để App Service của bạn có thể kết nối với SQL Server. Nếu không có cài đặt này, ứng dụng của bạn sẽ không thể truy cập cơ sở dữ liệu và sẽ gặp lỗi.
5.  Nhấp vào "Save".

> [!CAUTION]
> Việc mở quyền truy cập mạng cho SQL Server tiềm ẩn rủi ro bảo mật. Trong môi trường sản xuất, bạn nên cân nhắc sử dụng các phương pháp bảo mật nâng cao hơn như **Private Link** hoặc **Virtual Network integration** để giới hạn quyền truy cập vào cơ sở dữ liệu chỉ từ các tài nguyên Azure trong mạng riêng của bạn, giảm thiểu bề mặt tấn công. Đối với mục đích demo và học tập, việc cho phép IP máy khách và dịch vụ Azure là chấp nhận được.

## 4. Chuẩn bị Triển khai từ Môi trường Phát triển (Visual Studio / Antigravity IDE)

Bây giờ chúng ta đã có các tài nguyên Azure cần thiết, chúng ta có thể tiến hành chuẩn bị ứng dụng để triển khai từ môi trường phát triển của mình.

### 4.1. Thêm Tài khoản Azure vào Visual Studio

Để Visual Studio có thể tương tác và triển khai lên Azure, bạn cần thêm tài khoản Azure của mình vào Visual Studio.

**Các bước thực hiện:**
1.  Mở Visual Studio.
2.  Đi tới `File` > `Account Settings...`.
3.  Trong cửa sổ "Account Settings", nhấp vào "Add an account..." và đăng nhập bằng tài khoản Microsoft mà bạn đã sử dụng để đăng ký Azure.

### 4.2. Tạo Hồ sơ Xuất bản (Publish Profile)

Hồ sơ xuất bản (`.pubxml`) là một tệp cấu hình chứa tất cả các cài đặt cần thiết để triển khai ứng dụng của bạn lên một đích cụ thể (trong trường hợp này là Azure App Service). Nó giúp tự động hóa quá trình triển khai và đảm bảo tính nhất quán.

**Các bước thực hiện:**
1.  Trong Solution Explorer của Visual Studio, nhấp chuột phải vào dự án Web API của bạn.
2.  Chọn "Publish...".
3.  Trong cửa sổ "Publish", chọn "Azure" làm mục tiêu, sau đó nhấp "Next".
4.  Chọn "Azure App Service (Windows)" làm mục tiêu cụ thể, sau đó nhấp "Next".
5.  **App Service:**
    *   Đảm bảo bạn đã chọn đúng tài khoản Azure của mình.
    *   Chọn App Service mà bạn đã tạo (ví dụ: `app-NZWalks-EastUS-Dev-001`).
    *   Nhấp "Next".
6.  **API Management:** Chọn "Skip this step" (vì chúng ta không sử dụng Azure API Management trong hướng dẫn này, nhưng nó là một dịch vụ hữu ích để quản lý, bảo mật và xuất bản API của bạn). Nhấp "Next".
7.  **Finish:** Nhấp "Finish" để tạo hồ sơ xuất bản. Visual Studio sẽ tạo một tệp `.pubxml` trong thư mục `Properties\PublishProfiles` của dự án của bạn. Tệp này có thể được kiểm soát phiên bản (version control) để đảm bảo các thành viên trong nhóm sử dụng cùng một cấu hình triển khai.

### 4.3. Cấu hình Chuỗi Kết nối An toàn (Connection Strings)

Ứng dụng ASP.NET Core Web API của bạn sử dụng Entity Framework Core và có thể có nhiều `DbContext` (ví dụ: `AppDbContext` cho dữ liệu ứng dụng và `AuthDbContext` cho xác thực/ủy quyền). Mỗi `DbContext` cần một chuỗi kết nối để biết cách kết nối với cơ sở dữ liệu.

Khi triển khai lên Azure, chúng ta *không nên* nhúng chuỗi kết nối nhạy cảm trực tiếp vào `appsettings.json` trong gói triển khai. Thay vào đó, chúng ta sẽ cấu hình chuỗi kết nối trực tiếp trong App Service của Azure. Visual Studio giúp chúng ta làm điều này thông qua hồ sơ xuất bản.

**Cơ chế ngầm (Under the Hood):**
Khi bạn lưu chuỗi kết nối vào cài đặt App Service, Azure sẽ lưu trữ chúng một cách an toàn và inject chúng vào ứng dụng của bạn dưới dạng biến môi trường khi ứng dụng khởi động. ASP.NET Core, thông qua hệ thống cấu hình linh hoạt của nó, sẽ tự động đọc các biến môi trường này và ưu tiên chúng hơn các giá trị trong `appsettings.json`. Điều này đảm bảo rằng thông tin nhạy cảm không được lưu trữ trong mã nguồn hoặc gói triển khai, tăng cường bảo mật.

**Các bước thực hiện:**
1.  Sau khi tạo hồ sơ xuất bản, Visual Studio sẽ hiển thị trang "Publish" cho hồ sơ đó.
2.  Cuộn xuống phần "Service Dependencies". Bạn sẽ thấy các `DbContext` của mình (ví dụ: `ApplicationDbContext` và `AuthDbContext`).
3.  Đối với mỗi `DbContext`:
    *   Nhấp vào biểu tượng dấu chấm lửng (`...`) bên cạnh `DbContext`.
    *   Chọn "Azure SQL Database" và nhấp "Next".
    *   **Connection:**
        *   Chọn SQL Server và cơ sở dữ liệu bạn đã tạo (ví dụ: `sqlserver-NZWalks-EastUS-Dev-001` và `NZWalksDB-Dev`).
        *   Cung cấp "User name" và "Password" của quản trị viên SQL Server mà bạn đã tạo trước đó.
        *   Đảm bảo tùy chọn "Save connection string in Azure App Service settings" được chọn. Điều này sẽ lưu chuỗi kết nối vào phần "Configuration" của App Service trên Azure, nơi ứng dụng của bạn có thể truy cập nó một cách an toàn.
    *   Nhấp "Next", sau đó "Finish".

### 4.4. Vibe Coding và Antigravity IDE trong Chuẩn bị Triển khai

Antigravity IDE có thể đóng vai trò là một "co-pilot" mạnh mẽ trong giai đoạn chuẩn bị triển khai, giúp bạn duy trì "vibe" lập trình và tăng tốc quá trình:

*   **Tự động hóa cấu hình:** Antigravity có thể phân tích các chuỗi kết nối trong `appsettings.Development.json` của bạn và tự động đề xuất các lệnh Azure CLI để tạo các cài đặt chuỗi kết nối tương ứng trong App Service. Ví dụ:
    ```bash
    az webapp config connection-string set --resource-group RG-NZWalks-EastUS-Dev-001 --name app-NZWalks-EastUS-Dev-001 --connection-string-type SQLAzure --settings "DefaultConnection='Server=tcp:sqlserver-NZWalks-EastUS-Dev-001.database.windows.net,1433;Initial Catalog=NZWalksDB-Dev;Persist Security Info=False;User ID=sqladmin;Password=<YourPassword>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;'"
    ```
*   **Kiểm tra trước triển khai (Pre-flight Checks):** Trước khi bạn nhấp "Publish", Antigravity có thể tự động chạy các kiểm tra để đảm bảo:
    *   Tường lửa SQL Server đã được cấu hình chính xác để cho phép truy cập từ App Service.
    *   Tên miền của App Service là duy nhất.
    *   Phiên bản .NET Runtime trên App Service khớp với dự án của bạn.
    *   Nó có thể sử dụng subagent trình duyệt để kiểm tra cấu hình trên Azure Portal hoặc thực thi lệnh `az cli` để xác nhận.
*   **Hỗ trợ tạo `.pubxml`:** Nếu Visual Studio gặp sự cố, Antigravity có thể hướng dẫn bạn tạo hoặc chỉnh sửa tệp `.pubxml` một cách thủ công, đảm bảo tất cả các cài đặt cần thiết được bao gồm.
*   **Đề xuất tối ưu hóa:** Antigravity có thể phân tích dự án của bạn và đề xuất các tối ưu hóa cho việc triển khai, ví dụ như loại bỏ các gói NuGet không cần thiết để giảm kích thước gói triển khai hoặc cấu hình các cài đặt App Service để cải thiện hiệu suất.

Bằng cách tận dụng Antigravity, bạn có thể biến quá trình chuẩn bị triển khai từ một chuỗi các bước thủ công thành một quy trình tự động hóa và được hỗ trợ thông minh, giữ cho bạn trong "vibe" sáng tạo.

## 5. Triển khai Ứng dụng Web API và Gỡ lỗi: Đưa Code Lên Đám Mây

Sau khi cấu hình xong chuỗi kết nối và các thiết lập khác, chúng ta có thể tiến hành triển khai ứng dụng.

### 5.1. Triển khai Lần đầu

1.  Trên trang "Publish" trong Visual Studio, nhấp vào nút "Publish".
2.  Visual Studio sẽ thực hiện các bước sau:
    *   Biên dịch ứng dụng của bạn (Build).
    *   Đóng gói ứng dụng thành một gói triển khai (Publish Profile).
    *   Tải gói triển khai lên Azure App Service.
3.  Sau khi triển khai thành công, Visual Studio sẽ tự động mở trình duyệt và điều hướng đến URL của App Service của bạn.

> [!WARNING]
> Rất có thể bạn sẽ gặp lỗi `HTTP 500` hoặc `HTTP 503` ngay sau lần triển khai đầu tiên. Đây là điều khá phổ biến và thường do các vấn đề về cấu hình, cơ sở dữ liệu chưa được khởi tạo, hoặc các tệp/thư mục cần thiết bị thiếu. Đừng lo lắng, đây là một phần bình thường của quá trình triển khai và chúng ta sẽ khắc phục chúng.

### 5.2. Khắc phục Sự cố Triển khai Phổ biến

#### 5.2.1. Vấn đề 1: Database Migrations chưa được áp dụng

**Triệu chứng:** Ứng dụng khởi động nhưng không thể tương tác với cơ sở dữ liệu, hoặc gặp lỗi `SqlException` liên quan đến việc không tìm thấy bảng. Cơ sở dữ liệu trên Azure vẫn trống rỗng hoặc chưa có cấu trúc bảng cần thiết mặc dù bạn đã tạo các Entity Framework Core Migrations trong dự án.

**Cơ chế ngầm (Under the Hood):** Entity Framework Core Migrations là các tệp mã nguồn mô tả các thay đổi cấu trúc cơ sở dữ liệu. Chúng cần được "áp dụng" (apply) lên cơ sở dữ liệu để tạo hoặc cập nhật các bảng. Khi triển khai lên Azure, tiến trình này không tự động xảy ra trừ khi bạn cấu hình rõ ràng.

**Giải pháp:** Áp dụng Migrations khi xuất bản.
1.  Trong Visual Studio, truy cập lại trang "Publish" cho hồ sơ của bạn.
2.  Trong phần "Service Dependencies", bên cạnh mỗi `DbContext` đã được cấu hình chuỗi kết nối, nhấp vào "Show all settings".
3.  Đảm bảo rằng hộp kiểm "Apply this migration on publish" được chọn cho tất cả các `DbContext` của bạn.
4.  Nhấp "Save" để lưu cài đặt.
5.  Nhấp "Publish" một lần nữa.

> [!TIP]
> Tùy chọn "Apply this migration on publish" sẽ tự động chạy lệnh `dotnet ef database update` trên cơ sở dữ liệu từ xa sau khi ứng dụng được triển khai, đảm bảo cấu trúc bảng được tạo và các seed data (nếu có) được thêm vào. Đây là cách tiện lợi cho môi trường phát triển/staging. Trong môi trường sản xuất, bạn có thể cân nhắc chạy migrations như một bước riêng biệt trong pipeline CI/CD để có quyền kiểm soát tốt hơn.

#### 5.2.2. Vấn đề 2: Thiếu thư mục/tệp tĩnh (Missing Static Files/Folders)

**Triệu chứng:** Ứng dụng gặp lỗi `DirectoryNotFoundException` hoặc `FileNotFoundException` khi cố gắng truy cập các tệp tĩnh (ví dụ: hình ảnh được tải lên trong thư mục `Images`) hoặc các thư mục không phải là mã nguồn. Điều này xảy ra vì các thư mục này có thể không được bao gồm trong gói triển khai mặc định.

**Giải pháp:** Đảm bảo thư mục được bao gồm trong gói xuất bản.
1.  Trong Solution Explorer, nhấp chuột phải vào thư mục cần thiết (ví dụ: thư mục `Images`).
2.  Chọn "Properties".
3.  Đảm bảo rằng thuộc tính "Build Action" của các tệp bên trong thư mục được đặt thành "Content" và "Copy to Output Directory" được đặt thành "Copy if newer" hoặc "Copy always".
4.  Ngoài ra, bạn có thể chỉnh sửa tệp `.csproj` của dự án để bao gồm các thư mục này trong quá trình xuất bản một cách rõ ràng. Ví dụ:
    ```xml
    <ItemGroup>
      <Content Include="Images\**">
        <CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
      </Content>
    </ItemGroup>
    ```
    Dòng này đảm bảo rằng tất cả các tệp trong thư mục `Images` và các thư mục con của nó sẽ được sao chép vào thư mục xuất bản.
5.  Nhấp "Publish" một lần nữa.

#### 5.2.3. Vấn đề 3: Kiểm tra nhật ký lỗi nâng cao (Advanced Tools - Kudu và Application Insights)

**Triệu chứng:** Lỗi `HTTP 500` chung chung mà không có thông tin chi tiết trên trình duyệt. Điều này cho thấy có một ngoại lệ chưa được xử lý trong ứng dụng của bạn.

**Cơ chế ngầm (Under the Hood):** Azure App Service cung cấp các công cụ chẩn đoán mạnh mẽ để giúp bạn hiểu điều gì đang xảy ra bên trong ứng dụng của mình.
*   **Kudu (Advanced Tools):** Là một cổng quản lý và gỡ lỗi cho App Service. Nó cung cấp quyền truy cập vào hệ thống tệp, nhật ký luồng (log stream), biến môi trường, và thậm chí là một bảng điều khiển dòng lệnh để chạy các lệnh trực tiếp trên máy chủ ứng dụng.
*   **Application Insights:** Là một dịch vụ giám sát hiệu suất ứng dụng (APM) trong Azure Monitor. Nó thu thập dữ liệu telemetry từ ứng dụng của bạn (yêu cầu, ngoại lệ, phụ thuộc, nhật ký) và cung cấp các công cụ mạnh mẽ để phân tích, chẩn đoán và hình dung hiệu suất.

**Giải pháp:** Sử dụng Kudu và Application Insights để gỡ lỗi.

**Sử dụng Kudu:**
1.  Trên Azure Portal, truy cập App Service của bạn.
2.  Trong menu bên trái, tìm và chọn "Advanced Tools" (hoặc "Kudu").
3.  Nhấp vào "Go" để mở trang Kudu.
4.  Trong Kudu, đi tới "Debug console" > "CMD".
5.  Điều hướng đến thư mục `site\wwwroot`. Đây là nơi ứng dụng của bạn được triển khai.
6.  Chạy ứng dụng bằng lệnh `dotnet <TênỨngDụng>.dll` (ví dụ: `dotnet NZWalks.API.dll`).
7.  Quan sát đầu ra của console để xem bất kỳ ngoại lệ nào chưa được xử lý hoặc thông báo lỗi chi tiết. Điều này sẽ giúp bạn xác định nguyên nhân gốc rễ của sự cố.
8.  Bạn cũng có thể xem nhật ký lỗi ứng dụng bằng cách đi tới `LogFiles` trong Kudu Debug Console hoặc tải xuống các tệp nhật ký từ mục "Diagnostic logs" trong App Service.

**Sử dụng Application Insights:**
Nếu bạn đã cấu hình Application Insights (khuyến nghị cho mọi ứng dụng sản xuất):
1.  Trên Azure Portal, truy cập tài nguyên Application Insights của bạn.
2.  Kiểm tra mục "Failures" để xem các ngoại lệ đã xảy ra.
3.  Sử dụng "Live Metrics Stream" để xem dữ liệu telemetry theo thời gian thực.
4.  "Performance" và "Dependencies" giúp bạn xác định các điểm nghẽn hoặc lỗi trong các dịch vụ mà ứng dụng của bạn phụ thuộc (ví dụ: SQL Database).

> [!NOTE]
> **Antigravity IDE và Vibe Coding trong Gỡ lỗi Triển khai:**
> Đây là lúc Antigravity IDE thực sự tỏa sáng. Thay vì tự mình điều hướng qua Kudu hoặc Portal và cố gắng giải mã các nhật ký, bạn có thể:
> 1.  **Yêu cầu Antigravity phân tích nhật ký:** Cung cấp cho Antigravity URL của Kudu hoặc Application Insights. Antigravity có thể sử dụng subagent trình duyệt của nó để truy cập, đọc và phân tích các nhật ký lỗi.
> 2.  **Chẩn đoán tự động:** Antigravity có thể nhận diện các mẫu lỗi phổ biến (ví dụ: `DirectoryNotFoundException`, lỗi kết nối SQL, lỗi migrations) và tự động đưa ra các chẩn đoán chính xác cùng với các bước khắc phục cụ thể.
> 3.  **Đề xuất và thực thi sửa lỗi:** Dựa trên chẩn đoán, Antigravity có thể đề xuất các thay đổi cấu hình (ví dụ: chỉnh sửa `.csproj`), hoặc thậm chí tạo và thực thi các lệnh Azure CLI để sửa lỗi (ví dụ: cập nhật cài đặt App Service, khởi động lại dịch vụ).
> 4.  **Vibe Coding for Debugging:** Quá trình gỡ lỗi trở nên nhanh chóng và ít gián đoạn hơn. Bạn có thể tập trung vào việc hiểu vấn đề cấp cao, trong khi Antigravity lo phần tìm kiếm và phân tích chi tiết. Vòng lặp "thay đổi -> triển khai -> kiểm tra -> gỡ lỗi" được tăng tốc đáng kể, giúp bạn duy trì "vibe" lập trình liên tục mà không bị mắc kẹt trong các công việc lặp lại.

### 5.3. Khởi động lại App Service

Sau khi áp dụng các thay đổi hoặc khắc phục lỗi (ví dụ: cấu hình chuỗi kết nối, áp dụng migrations), bạn nên khởi động lại App Service để đảm bảo các thay đổi có hiệu lực hoàn toàn và ứng dụng được tải lại với cấu hình mới.
1.  Trên Azure Portal, truy cập App Service của bạn.
2.  Trên trang "Overview", nhấp vào nút "Stop", sau đó nhấp vào nút "Start".

## 6. Kiểm tra API đã Triển khai: Xác Nhận Hoạt Động

Sau khi ứng dụng đã được triển khai thành công và các lỗi ban đầu đã được khắc phục, chúng ta cần kiểm tra xem API có hoạt động đúng như mong đợi hay không.

> [!NOTE]
> Ứng dụng Web API của bạn được bảo mật bằng JWT Authentication. Điều này có nghĩa là bạn không thể truy cập trực tiếp các endpoint được bảo vệ từ trình duyệt mà không có mã thông báo hợp lệ. Bạn sẽ cần sử dụng các công cụ như Postman, Swagger UI (nếu được cấu hình trong API của bạn) hoặc một ứng dụng khách để kiểm tra.

### 6.1. Truy cập API thông qua Swagger UI (hoặc Postman)

Giả sử ứng dụng của bạn đã được cấu hình để sử dụng Swagger/OpenAPI (đây là một thực hành tốt cho các Web API), bạn có thể truy cập UI của nó thông qua URL của App Service: `https://<TênAppService>.azurewebsites.net/swagger`.

### 6.2. Đăng ký Người Dùng và Đăng nhập (Authentication): Lấy JWT Token

Để kiểm tra các endpoint được bảo vệ (những endpoint có thuộc tính `[Authorize]` trên Controller hoặc action method), trước tiên chúng ta cần đăng ký một người dùng và sau đó đăng nhập để lấy mã thông báo JWT. Mã thông báo này sẽ được sử dụng trong tiêu đề `Authorization` của các yêu cầu tiếp theo.

**Ví dụ sử dụng Swagger UI/Postman:**

1.  **Đăng ký người dùng mới (Register):**
    *   **HTTP Verb:** `POST`
    *   **Endpoint:** `https://<TênAppService>.azurewebsites.net/api/Auth/Register`
    *   **Request Body (JSON):**
        ```json
        {
          "username": "nzwalksreader@example.com",
          "password": "Reader@123",
          "roles": ["Reader"]
        }
        ```
    *   **Expected Response:** `HTTP 200 OK` với thông báo "User registered! Please Login".

2.  **Đăng nhập để lấy mã thông báo JWT (Login):**
    *   **HTTP Verb:** `POST`
    *   **Endpoint:** `https://<TênAppService>.azurewebsites.net/api/Auth/Login`
    *   **Request Body (JSON):**
        ```json
        {
          "username": "nzwalksreader@example.com",
          "password": "Reader@123"
        }
        ```
    *   **Expected Response:** `HTTP 200 OK` với một đối tượng JSON chứa mã thông báo JWT. Sao chép mã thông báo này.

### 6.3. Truy cập Tài nguyên được Bảo vệ bằng JWT Token

Bây giờ, với mã thông báo JWT đã có, bạn có thể truy cập các endpoint yêu cầu xác thực.

**Ví dụ:**
1.  **HTTP Verb:** `GET`
2.  **Endpoint:** `https://<TênAppService>.azurewebsites.net/api/Regions`
3.  **Request Headers:**
    *   `Authorization: Bearer <MãThôngBáoJWTCủaBạn>` (thay `<MãThôngBáoJWTCủaBạn>` bằng mã thông báo bạn đã nhận được từ bước đăng nhập).
4.  **Expected Response:** `HTTP 200 OK` với danh sách các vùng (Regions) từ cơ sở dữ liệu. Điều này chứng tỏ:
    *   Ứng dụng đang chạy.
    *   Kết nối cơ sở dữ liệu thành công.
    *   Entity Framework Core (với Repository Pattern và Dependency Injection) đang hoạt động.
    *   Cơ chế xác thực JWT đang hoạt động.
    *   Controller và HTTP Verb (GET) đang xử lý yêu cầu đúng cách.

Nếu bạn nhận được phản hồi `200 OK` với dữ liệu mong muốn, điều đó có nghĩa là API của bạn đang hoạt động thành công trên Azure, bao gồm cả kết nối cơ sở dữ liệu và cơ chế xác thực JWT.

### 6.4. Xác nhận Dữ liệu trong Azure SQL Database

Để xác nhận rằng quá trình di chuyển đã hoạt động và dữ liệu gốc (seed data, nếu có) đã được thêm vào cơ sở dữ liệu, bạn có thể kết nối với Azure SQL Database của mình bằng SQL Server Management Studio (SSMS) hoặc Azure Data Studio.

1.  Mở SSMS hoặc Azure Data Studio.
2.  Sử dụng tên máy chủ SQL Server Azure của bạn (ví dụ: `sqlserver-NZWalks-EastUS-Dev-001.database.windows.net`), tên đăng nhập quản trị viên (ví dụ: `sqladmin`) và mật khẩu để kết nối.
3.  Truy cập cơ sở dữ liệu `NZWalksDB-Dev`.
4.  Kiểm tra các bảng đã được tạo (ví dụ: `Regions`, `Walks`, `Users`, `Roles`, `UserRoles`) và dữ liệu bên trong chúng. Nếu bạn thấy các bảng và dữ liệu seed đã được tạo, điều đó xác nhận rằng Entity Framework Core Migrations đã được áp dụng thành công.

## Tóm tắt Phần

Phần này đã hướng dẫn bạn qua toàn bộ quy trình triển khai một ứng dụng ASP.NET Core Web API lên Azure, bao gồm các bước sau:

*   **Thiết lập môi trường Azure:** Tạo một **Azure Subscription** để quản lý chi phí và tài nguyên, sau đó tạo một **Resource Group** để nhóm các tài nguyên liên quan lại với nhau, đảm bảo quản lý hiệu quả và vòng đời đồng bộ.
*   **Triển khai Azure App Service:** Sử dụng **Azure App Service** làm nền tảng (PaaS) để lưu trữ ứng dụng ASP.NET Core Web API, cấu hình tên, runtime stack (.NET 7), hệ điều hành và gói dịch vụ. Chúng ta cũng đã tìm hiểu về cách App Service quản lý cấu hình dưới dạng biến môi trường.
*   **Thiết lập Azure SQL Database:** Tạo **Azure SQL Server** logic và **Azure SQL Database** để lưu trữ dữ liệu của ứng dụng. Đặc biệt, chúng ta đã cấu hình **tường lửa mạng** để cho phép App Service và máy tính cục bộ kết nối một cách an toàn, đồng thời thảo luận về các cân nhắc bảo mật nâng cao.
*   **Chuẩn bị triển khai từ Visual Studio:** Thêm tài khoản Azure vào Visual Studio và tạo **hồ sơ xuất bản** (`.pubxml`) để tự động hóa quá trình triển khai.
*   **Cấu hình chuỗi kết nối an toàn:** Liên kết các `DbContext` (Entity Framework Core) với Azure SQL Database và lưu chuỗi kết nối vào **Azure App Service settings**. Điều này đảm bảo rằng thông tin nhạy cảm không được nhúng vào mã nguồn và được đọc an toàn bởi cơ chế cấu hình của ASP.NET Core thông qua Dependency Injection.
*   **Vibe Coding và Antigravity IDE:** Chúng ta đã khám phá cách Antigravity IDE có thể hỗ trợ tự động hóa việc tạo tài nguyên, kiểm tra trước triển khai, và đặc biệt là tăng tốc quá trình gỡ lỗi bằng cách phân tích nhật ký và đề xuất các hành động khắc phục, giúp bạn duy trì "vibe" lập trình liên tục.
*   **Xử lý lỗi triển khai phổ biến:**
    *   Kích hoạt tùy chọn "Apply this migration on publish" cho các `DbContext` để tự động tạo/cập nhật lược đồ cơ sở dữ liệu.
    *   Đảm bảo các thư mục tệp tĩnh (ví dụ: `Images`) được bao gồm trong gói xuất bản bằng cách cấu hình `.csproj`.
    *   Sử dụng **Advanced Tools (Kudu)** trên Azure Portal và **Application Insights** để kiểm tra nhật ký lỗi chi tiết và gỡ lỗi trực tiếp các vấn đề phát sinh.
*   **Kiểm tra API đã triển khai:** Sử dụng Swagger UI hoặc Postman để đăng ký người dùng, đăng nhập để lấy **JWT Token**, và sau đó sử dụng token này để truy cập các **endpoint được bảo vệ** (sử dụng HTTP Verbs và Controllers) của Web API trên Azure. Điều này xác nhận hoạt động của Authentication, Authorization, Dependency Injection và Repository Pattern.
*   **Xác nhận dữ liệu:** Kết nối với Azure SQL Database bằng SSMS/Azure Data Studio để kiểm tra các bảng và dữ liệu đã được áp dụng, đảm bảo tính toàn vẹn của cơ sở dữ liệu.

Với các bước này, bạn đã thành công triển khai ứng dụng ASP.NET Core Web API của mình lên Azure, sẵn sàng phục vụ người dùng trong một môi trường đám mây mạnh mẽ và có khả năng mở rộng.

<!-- REVIEWED_BY_AGENT -->
