# Bài 23: Giới Thiệu Ràng Buộc và Giá Trị Mặc Định trong PostgreSQL

Trong kỷ nguyên số, dữ liệu là tài sản vô giá và là nền tảng cho mọi ứng dụng hiện đại. Việc đảm bảo dữ liệu luôn chính xác, nhất quán và đáng tin cậy không chỉ là yêu cầu kỹ thuật mà còn là yếu tố sống còn đối với bất kỳ hệ thống nào. Chương này sẽ đưa bạn đi sâu vào các cơ chế xác thực và bảo vệ dữ liệu mạnh mẽ mà PostgreSQL cung cấp, đặc biệt tập trung vào cách triển khai các ràng buộc (constraints) và giá trị mặc định (default values). Chúng ta sẽ tìm hiểu lý do tại sao những cơ chế này lại quan trọng, cách áp dụng chúng để ngăn chặn dữ liệu không hợp lệ, và cách tự động cung cấp các giá trị mặc định khi thông tin không được cung cấp.

Mục tiêu của chương này là trang bị cho bạn kiến thức và kỹ năng để:
*   Phân biệt và đánh giá tầm quan trọng của việc xác thực dữ liệu ở cấp độ ứng dụng và cấp độ cơ sở dữ liệu.
*   Thực hành tạo và quản lý cơ sở dữ liệu, bảng trong môi trường PgAdmin.
*   Áp dụng ràng buộc `NOT NULL` để đảm bảo các cột thiết yếu luôn chứa giá trị.
*   Xử lý các tình huống phức tạp khi thêm ràng buộc `NOT NULL` vào các bảng đã có dữ liệu.
*   Sử dụng giá trị mặc định (`DEFAULT`) cho các cột để tăng tính linh hoạt và duy trì sự nhất quán của dữ liệu.
*   Hiểu cách các ràng buộc này cung cấp "hàng rào bảo vệ" cho các hệ thống phức tạp, bao gồm cả các hệ thống được điều khiển bởi AI Agent như Antigravity IDE.

## I. Tầm Quan Trọng Của Xác Thực Dữ Liệu: Tuyến Phòng Thủ Đa Lớp

Xác thực dữ liệu là quá trình kiểm tra và đảm bảo rằng dữ liệu được nhập vào hệ thống tuân thủ các quy tắc, định dạng và giới hạn đã định. Mục tiêu là ngăn chặn việc lưu trữ dữ liệu không chính xác hoặc không hợp lệ, vốn có thể dẫn đến lỗi ứng dụng, báo cáo sai lệch, quyết định kinh doanh sai lầm và làm suy yếu độ tin cậy của toàn bộ hệ thống.

Để đạt được sự bảo vệ tối ưu, xác thực dữ liệu thường được thực hiện ở nhiều cấp độ khác nhau.

### 1.1. Xác Thực Ở Cấp Độ Ứng Dụng (Application Layer)

Xác thực ở cấp độ ứng dụng (ví dụ: máy chủ web, ứng dụng di động, giao diện người dùng) là tuyến phòng thủ đầu tiên và dễ tiếp cận nhất. Khi người dùng nhập dữ liệu qua một giao diện, ứng dụng sẽ kiểm tra tính hợp lệ của dữ liệu *trước khi* gửi nó đến cơ sở dữ liệu.

**Ví dụ thực tế:** Một biểu mẫu đăng ký người dùng trên website yêu cầu địa chỉ email.
*   **Xác thực ứng dụng:** Ngay khi người dùng nhập email và nhấp "Đăng ký", ứng dụng JavaScript trên trình duyệt (hoặc máy chủ web) sẽ kiểm tra xem chuỗi nhập vào có định dạng email hợp lệ hay không (chứa `@`, có tên miền, v.v.). Nếu không, một thông báo lỗi sẽ hiển thị ngay lập tức: "Địa chỉ email không hợp lệ."

**Ưu điểm:**
*   **Phản hồi tức thì:** Cải thiện trải nghiệm người dùng bằng cách cung cấp thông báo lỗi ngay lập tức.
*   **Giảm tải cho cơ sở dữ liệu:** Chỉ dữ liệu hợp lệ mới được gửi đến cơ sở dữ liệu, giảm thiểu các lỗi truy vấn và xử lý không cần thiết.
*   **Linh hoạt:** Dễ dàng triển khai các quy tắc xác thực phức tạp liên quan đến logic nghiệp vụ cụ thể của ứng dụng.

**Liên hệ với AI Coding và Antigravity IDE:**
Khi bạn sử dụng Antigravity IDE để phát triển ứng dụng, các Agent AI của nó sẽ được hướng dẫn để tạo ra mã nguồn ứng dụng bao gồm các cơ chế xác thực dữ liệu ở cấp độ giao diện người dùng và máy chủ. Điều này giúp đảm bảo rằng dữ liệu nhập liệu từ người dùng được lọc ban đầu, tạo ra một "vibe" tích cực về trải nghiệm người dùng. Tuy nhiên, nếu Antigravity hoặc bất kỳ hệ thống tự động nào khác (ví dụ: một script xử lý dữ liệu hàng loạt) trực tiếp tương tác với cơ sở dữ liệu mà không đi qua lớp ứng dụng này, thì tuyến phòng thủ cấp ứng dụng sẽ bị bỏ qua.

### 1.2. Xác Thực Ở Cấp Độ Cơ Sở Dữ Liệu (Database Layer)

Xác thực ở cấp độ cơ sở dữ liệu là "tuyến phòng thủ cuối cùng" và quan trọng nhất. Nó đảm bảo rằng bất kể dữ liệu đến từ đâu – dù là từ ứng dụng, một script tự động, một công cụ quản trị cơ sở dữ liệu (như PgAdmin), hay thậm chí một hệ thống bên ngoài có lỗ hổng – dữ liệu đó luôn tuân thủ các quy tắc cốt lõi đã được định nghĩa trong schema cơ sở dữ liệu. Các ràng buộc được định nghĩa trực tiếp trên bảng hoặc cột và được hệ quản trị cơ sở dữ liệu (DBMS) PostgreSQL tự động kiểm tra trước khi cho phép dữ liệu được lưu trữ.

**Ví dụ:** Nếu một người dùng quản trị kết nối trực tiếp với PostgreSQL bằng PgAdmin và cố gắng thêm một sản phẩm với giá `-35` hoặc không có trọng lượng, cơ sở dữ liệu sẽ chấp nhận dữ liệu đó *nếu không có ràng buộc nào được định nghĩa*. Tuy nhiên, nếu có ràng buộc, cơ sở dữ liệu sẽ từ chối thao tác này.

**Ưu điểm:**
*   **Tính toàn vẹn dữ liệu tối cao:** Đảm bảo dữ liệu luôn đúng và nhất quán, bất kể nguồn gốc.
*   **Độc lập với ứng dụng:** Các quy tắc nghiệp vụ cốt lõi được nhúng vào cấu trúc cơ sở dữ liệu, không phụ thuộc vào logic của một ứng dụng cụ thể.
*   **Bảo vệ chống lại lỗi ứng dụng:** Ngay cả khi xác thực ở cấp ứng dụng bị bỏ qua hoặc gặp lỗi, cơ sở dữ liệu vẫn sẽ bảo vệ tính toàn vẹn của dữ liệu.

**Liên hệ với AI Coding và Antigravity IDE:**
Đối với các hệ thống phức tạp, đặc biệt là khi làm việc với Antigravity IDE và tư duy Vibe Coding, xác thực cấp cơ sở dữ liệu trở nên cực kỳ quan trọng. Antigravity là một hệ thống Agentic AI có khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file và lập kế hoạch tự động. Trong một môi trường như vậy, có rất nhiều điểm tiềm năng mà dữ liệu có thể được tạo ra hoặc sửa đổi.

*   **Guardrails cho AI Agents:** Các ràng buộc cơ sở dữ liệu đóng vai trò là "guardrails" (hàng rào bảo vệ) cho các Agent AI. Chúng đặt ra các quy tắc bất biến mà ngay cả một Agent thông minh cũng không thể vượt qua, đảm bảo rằng mọi hành động (ví dụ: `INSERT`, `UPDATE`) đều duy trì tính toàn vẹn của dữ liệu. Điều này ngăn chặn "bad vibes" do dữ liệu sai lệch gây ra, vốn có thể dẫn đến các quyết định sai lầm của Agent hoặc các lỗi cascading trong hệ thống tự động.
*   **Giảm thiểu rủi ro từ mã tự sinh:** Mặc dù Antigravity được thiết kế để tạo mã chất lượng cao, nhưng việc có các ràng buộc ở cấp độ cơ sở dữ liệu cung cấp một lớp bảo vệ bổ sung. Nếu có một lỗi logic nhỏ trong mã được tạo tự động hoặc trong kế hoạch thực thi của Agent, cơ sở dữ liệu vẫn sẽ từ chối dữ liệu không hợp lệ, ngăn chặn sự cố nghiêm trọng hơn.
*   **Đảm bảo tính nhất quán giữa các Agent/hệ thống:** Khi nhiều Agent hoặc các hệ thống con khác nhau tương tác với cùng một cơ sở dữ liệu, các ràng buộc đảm bảo rằng tất cả đều tuân thủ cùng một bộ quy tắc dữ liệu, duy trì sự nhất quán trên toàn bộ kiến trúc.

Trong chương này, chúng ta sẽ tập trung vào cách triển khai các ràng buộc này trong PostgreSQL để xây dựng một nền tảng dữ liệu vững chắc, đáng tin cậy cho mọi ứng dụng, dù được phát triển thủ công hay bởi AI.

## II. Chuẩn Bị Môi Trường và Khám Phá Vấn Đề Dữ Liệu

Để bắt đầu thực hành, chúng ta sẽ tạo một cơ sở dữ liệu mới và một bảng mẫu trong PgAdmin. Sau đó, chúng ta sẽ cố tình chèn dữ liệu không hợp lệ để minh họa rõ ràng các vấn đề mà ràng buộc cơ sở dữ liệu có thể giải quyết.

### 2.1. Tạo Cơ Sở Dữ Liệu Mới Trong PgAdmin

Việc tạo một cơ sở dữ liệu riêng biệt cho các bài thực hành giúp giữ cho môi trường làm việc của bạn sạch sẽ và có tổ chức.

1.  Mở PgAdmin.
2.  Trong cây điều hướng bên trái, mở rộng mục **Servers** (nếu chưa mở), sau đó mở rộng kết nối máy chủ của bạn (ví dụ: `PostgreSQL 16`).
3.  Nhấp chuột phải vào mục **Databases**.
4.  Chọn **Create** > **Database...**.
5.  Trong cửa sổ "Create - Database", nhập tên cơ sở dữ liệu là `validation_db`.
6.  Nhấp vào **Save**.

Bây giờ bạn đã có một cơ sở dữ liệu mới để làm việc.

> [!TIP]
> Luôn đảm bảo bạn đang kết nối với đúng cơ sở dữ liệu khi mở công cụ truy vấn. Trong PgAdmin, sau khi tạo `validation_db`, nhấp chuột phải vào nó và chọn **Query Tool** để mở một cửa sổ truy vấn mới được kết nối trực tiếp với cơ sở dữ liệu này. Mọi lệnh SQL bạn chạy trong cửa sổ này sẽ được thực thi trên `validation_db`.

### 2.2. Tạo Bảng `products` Ban Đầu (Không Có Ràng Buộc)

Chúng ta sẽ tạo một bảng `products` đơn giản với cấu trúc cơ bản bao gồm `id`, `name`, `department`, `price`, và `weight`. Quan trọng là, chúng ta sẽ **tạm thời không thêm bất kỳ ràng buộc nào** ngoài `PRIMARY KEY` cho cột `id` để minh họa các vấn đề về tính toàn vẹn dữ liệu.

```sql
-- Đảm bảo bạn đang kết nối đến cơ sở dữ liệu 'validation_db'
-- (Trong PgAdmin, nhấp chuột phải vào 'validation_db' và chọn 'Query Tool')

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(40),
    department VARCHAR(40),
    price INTEGER,
    weight INTEGER
);
```

**Giải thích chi tiết về các kiểu dữ liệu và ràng buộc ban đầu:**

*   `id SERIAL PRIMARY KEY`:
    *   `SERIAL`: Đây là một kiểu dữ liệu giả trong PostgreSQL. Khi bạn khai báo một cột là `SERIAL`, PostgreSQL sẽ tự động thực hiện ba hành động:
        1.  Tạo một đối tượng `SEQUENCE` (một bộ đếm số) riêng biệt cho cột này.
        2.  Đặt giá trị mặc định (`DEFAULT`) cho cột này là `nextval('tên_bảng_tên_cột_seq'::regclass)`, nghĩa là nó sẽ tự động lấy giá trị tiếp theo từ sequence mỗi khi một hàng mới được chèn.
        3.  Đặt cột này là `NOT NULL`.
    *   `PRIMARY KEY`: Ràng buộc này đảm bảo rằng:
        1.  Tất cả các giá trị trong cột `id` là **duy nhất** (unique).
        2.  Cột `id` **không được phép có giá trị `NULL`**.
        3.  PostgreSQL tự động tạo một chỉ mục (index) trên cột này để tối ưu hóa việc tìm kiếm và truy xuất dữ liệu.
        Ràng buộc `PRIMARY KEY` là nền tảng cho việc xác định duy nhất mỗi bản ghi trong bảng.
*   `name VARCHAR(40)`: Kiểu dữ liệu chuỗi có độ dài tối đa là 40 ký tự. Hiện tại, cột này có thể chứa `NULL` và chuỗi rỗng.
*   `department VARCHAR(40)`: Tương tự như `name`, có thể chứa `NULL` và chuỗi rỗng.
*   `price INTEGER`: Kiểu dữ liệu số nguyên. Hiện tại, cột này có thể chứa `NULL` và bất kỳ giá trị số nguyên nào, kể cả số âm.
*   `weight INTEGER`: Tương tự như `price`, có thể chứa `NULL` và bất kỳ giá trị số nguyên nào.

Để xem cấu trúc bảng vừa tạo trong PgAdmin:
1.  Trong cây điều hướng, mở rộng `validation_db` -> `Schemas` -> `public` -> `Tables`.
2.  Bạn sẽ thấy bảng `products`. Nhấp chuột phải vào `products` và chọn **Properties...** để xem chi tiết cấu trúc bảng, bao gồm các cột, kiểu dữ liệu và ràng buộc.
3.  Hoặc nhấp chuột phải vào `products` và chọn **View/Edit Data** > **All Rows** để xem dữ liệu (hiện tại chưa có).

### 2.3. Minh Họa Vấn Đề Dữ Liệu Không Hợp Lệ

Bây giờ, hãy chèn một số dữ liệu vào bảng `products`, bao gồm cả dữ liệu mà chúng ta biết là không hợp lệ theo logic nghiệp vụ thông thường, để minh họa cách cơ sở dữ liệu sẽ chấp nhận chúng nếu không có ràng buộc.

```sql
-- Chèn một sản phẩm hợp lệ ban đầu
INSERT INTO products (name, department, price, weight)
VALUES ('Áo sơ mi', 'Quần áo', 20, 1);

-- Chèn một sản phẩm thiếu giá (price sẽ là NULL)
INSERT INTO products (name, department, weight)
VALUES ('Quần jean', 'Quần áo', 3);

-- Chèn một sản phẩm với giá không hợp lệ (giá âm)
INSERT INTO products (name, department, price, weight)
VALUES ('Giày thể thao', 'Giày dép', -10, 2);

-- Chèn một sản phẩm thiếu trọng lượng (weight sẽ là NULL)
INSERT INTO products (name, department, price)
VALUES ('Mũ lưỡi trai', 'Phụ kiện', 15);
```

Sau khi chạy các lệnh `INSERT` này, hãy xem dữ liệu trong bảng `products`:

```sql
SELECT * FROM products;
```

Kết quả sẽ hiển thị như sau:

| id | name          | department | price | weight |
|----|---------------|------------|-------|--------|
| 1  | Áo sơ mi      | Quần áo    | 20    | 1      |
| 2  | Quần jean     | Quần áo    | NULL  | 3      |
| 3  | Giày thể thao | Giày dép   | -10   | 2      |
| 4  | Mũ lưỡi trai  | Phụ kiện   | 15    | NULL   |

Như bạn thấy, bảng `products` đã chấp nhận:
*   Giá trị `NULL` cho cột `price` (sản phẩm "Quần jean").
*   Giá trị `NULL` cho cột `weight` (sản phẩm "Mũ lưỡi trai").
*   Giá trị âm (`-10`) cho cột `price` (sản phẩm "Giày thể thao").

Những dữ liệu này rõ ràng là không hợp lệ trong một kịch bản kinh doanh thực tế. Chúng có thể gây ra lỗi trong các phép tính, báo cáo sai lệch, và làm suy giảm độ tin cậy của hệ thống. Đây chính là vấn đề mà các ràng buộc cơ sở dữ liệu được thiết kế để giải quyết.

## III. Ràng Buộc `NOT NULL`: Đảm Bảo Sự Hiện Diện Của Dữ Liệu

Ràng buộc `NOT NULL` là một trong những ràng buộc cơ bản và quan trọng nhất để duy trì tính toàn vẹn của dữ liệu. Nó đảm bảo rằng một cột không bao giờ chứa giá trị `NULL` (giá trị không xác định hoặc thiếu).

### 3.1. Ý Nghĩa và Tầm Quan Trọng của `NULL`

Trong SQL, `NULL` không phải là `0`, cũng không phải là chuỗi rỗng (`''`). `NULL` đại diện cho một giá trị không xác định, không có, hoặc không áp dụng.

**Vấn đề với giá trị `NULL`:**
Trong ví dụ trên, sản phẩm "Quần jean" có `price` là `NULL`. Điều này có thể dẫn đến nhiều vấn đề:
*   **Lỗi logic ứng dụng:** Nếu ứng dụng mong đợi một giá trị số để thực hiện tính toán (ví dụ: tổng doanh thu) và không xử lý `NULL` đúng cách, nó có thể gây ra lỗi runtime hoặc trả về kết quả không chính xác.
*   **Dữ liệu không nhất quán:** `NULL` có thể được hiểu khác nhau tùy thuộc vào ngữ cảnh (ví dụ: `0`, không có giá trị, hoặc một lỗi), dẫn đến sự không nhất quán trong cách dữ liệu được diễn giải và sử dụng.
*   **Khó khăn trong phân tích và báo cáo:** Các hàm tổng hợp (như `SUM`, `AVG`) thường bỏ qua giá trị `NULL`. Điều này có thể dẫn đến kết quả báo cáo không chính xác nếu bạn mong đợi `NULL` được tính là `0` hoặc một giá trị khác.
*   **Hiệu suất truy vấn:** Việc xử lý `NULL` trong các điều kiện `WHERE` hoặc `JOIN` đôi khi phức tạp hơn và có thể ảnh hưởng đến hiệu suất.

Để giải quyết những vấn đề này, chúng ta cần đảm bảo rằng các cột quan trọng như `name`, `department`, `price` không bao giờ được phép là `NULL`.

**Vibe Coding Perspective:** Sự hiện diện của `NULL` trong các cột quan trọng tạo ra một "uncertainty vibe" trong toàn bộ hệ thống. Nó giống như việc bạn có một hợp đồng nhưng thiếu những thông tin cơ bản. `NOT NULL` giúp loại bỏ sự mơ hồ này, mang lại một "clarity vibe" và sự tin cậy cao hơn cho dữ liệu.

### 3.2. Định Nghĩa Ràng Buộc `NOT NULL` Khi Tạo Bảng

Cách tốt nhất và được khuyến nghị là định nghĩa các ràng buộc `NOT NULL` ngay từ khi tạo bảng, phản ánh đúng yêu cầu nghiệp vụ của dữ liệu.

```sql
-- DROP TABLE IF EXISTS products; -- Lệnh này dùng để xóa bảng cũ nếu bạn muốn tạo lại từ đầu
-- Bạn có thể chạy lệnh này nếu muốn tạo lại bảng với các ràng buộc NOT NULL
-- Tuy nhiên, trong bài này chúng ta sẽ ALTER bảng đã có.

-- Ví dụ về cách tạo bảng với NOT NULL từ đầu:
-- CREATE TABLE products (
--     id SERIAL PRIMARY KEY,
--     name VARCHAR(40) NOT NULL,       -- Tên sản phẩm không được NULL
--     department VARCHAR(40) NOT NULL, -- Bộ phận không được NULL
--     price INTEGER NOT NULL,          -- Giá sản phẩm không được NULL
--     weight INTEGER                   -- Trọng lượng có thể NULL (ví dụ: chưa xác định)
-- );
```

Với cấu trúc này, bất kỳ cố gắng chèn một hàng nào mà thiếu `name`, `department` hoặc `price` (tức là cung cấp `NULL` cho chúng) sẽ bị từ chối ngay lập tức bởi PostgreSQL.

### 3.3. Thêm Ràng Buộc `NOT NULL` Vào Cột Đã Tồn Tại (`ALTER TABLE`)

Trong nhiều trường hợp thực tế, bạn cần thêm ràng buộc `NOT NULL` vào một cột trong bảng đã tồn tại, đặc biệt là khi bạn nhận ra rằng dữ liệu hiện có đang gặp vấn đề hoặc khi yêu cầu nghiệp vụ thay đổi.

Để thêm ràng buộc `NOT NULL` cho cột `price` trong bảng `products` hiện có, chúng ta sử dụng lệnh `ALTER TABLE`:

```sql
ALTER TABLE products
ALTER COLUMN price SET NOT NULL;
```

> [!CAUTION]
> Khi bạn chạy lệnh trên với bảng `products` hiện tại của chúng ta (có sản phẩm "Quần jean" với `price` là `NULL`), bạn sẽ gặp lỗi. Đây là một "gotcha" (điểm dễ gây nhầm lẫn hoặc lỗi) phổ biến trong PostgreSQL và các hệ quản trị CSDL khác.

### 3.4. Xử Lý "Gotcha": Giá Trị `NULL` Hiện Có

Lỗi bạn sẽ nhận được khi cố gắng áp dụng `NOT NULL` cho một cột chứa `NULL` sẽ tương tự như:
`ERROR: column "price" contains null values`
Điều này là hoàn toàn hợp lý: bạn không thể áp dụng một quy tắc "không được phép `NULL`" trong khi cột đó đã có `NULL`. Để khắc phục, bạn có hai lựa chọn chính:

1.  **Xóa các hàng chứa `NULL`:** Nếu dữ liệu `NULL` đó là không hợp lệ và có thể bị loại bỏ mà không ảnh hưởng đến nghiệp vụ.
    ```sql
    DELETE FROM products WHERE price IS NULL;
    ```
2.  **Cập nhật các giá trị `NULL`:** Thay thế các giá trị `NULL` bằng một giá trị hợp lệ khác. Đây thường là cách tiếp cận an toàn và được ưu tiên hơn để không mất dữ liệu.

Trong trường hợp của chúng ta, chúng ta sẽ chọn phương án cập nhật các giá trị `NULL` trong cột `price` thành một giá trị tạm thời (ví dụ: `0` hoặc `9999`) trước khi áp dụng ràng buộc.

**Bước 1: Cập nhật các giá trị `NULL`**
Để tìm và cập nhật các giá trị `NULL`, chúng ta sử dụng toán tử `IS NULL`.

```sql
-- Cập nhật tất cả các hàng có price là NULL thành 0.
-- Chọn 0 làm giá trị mặc định tạm thời có ý nghĩa hơn 9999 trong nhiều ngữ cảnh.
UPDATE products
SET price = 0
WHERE price IS NULL;

-- Kiểm tra lại dữ liệu sau khi cập nhật
SELECT * FROM products;
```

Sau khi chạy lệnh `UPDATE`, cột `price` của sản phẩm "Quần jean" sẽ được thay đổi từ `NULL` thành `0`.

| id | name          | department | price | weight |
|----|---------------|------------|-------|--------|
| 1  | Áo sơ mi      | Quần áo    | 20    | 1      |
| 2  | Quần jean     | Quần áo    | 0     | 3      |
| 3  | Giày thể thao | Giày dép   | -10   | 2      |
| 4  | Mũ lưỡi trai  | Phụ kiện   | 15    | NULL   |

**Bước 2: Áp dụng lại ràng buộc `NOT NULL`**
Bây giờ cột `price` không còn giá trị `NULL` nào, chúng ta có thể áp dụng ràng buộc `NOT NULL` thành công:

```sql
ALTER TABLE products
ALTER COLUMN price SET NOT NULL;

-- Lệnh này sẽ chạy thành công.
```

**Bước 3: Kiểm tra ràng buộc `NOT NULL`**
Hãy thử chèn một hàng mới mà không cung cấp giá trị cho cột `price`:

```sql
-- Cố gắng chèn một sản phẩm thiếu giá
INSERT INTO products (name, department, weight)
VALUES ('Vớ', 'Quần áo', 0.1); -- Lưu ý: weight là INTEGER, 0.1 sẽ bị làm tròn nếu chèn, nhưng ở đây sẽ lỗi NOT NULL trước
```

Bạn sẽ nhận được một lỗi từ PostgreSQL, tương tự như: `ERROR: null value in column "price" violates not-null constraint`. Điều này xác nhận rằng ràng buộc `NOT NULL` đã được áp dụng và đang hoạt động hiệu quả.

### 3.5. Khi Nào Nên Sử Dụng `NOT NULL`?

Hầu hết các cột trong bảng của bạn nên có ràng buộc `NOT NULL`, trừ khi có một lý do nghiệp vụ rất cụ thể để cho phép giá trị `NULL`.
*   **Luôn `NOT NULL`:** Các cột định danh (`id`), tên (`name`), các trường bắt buộc cho logic nghiệp vụ (ví dụ: `department`, `price`, `quantity`), các trường thời gian tạo/cập nhật (`created_at`, `updated_at`).
*   **Có thể `NULL`:** Các cột tùy chọn hoặc thông tin có thể chưa được biết. Ví dụ: `weight` (như trong ví dụ, nếu trọng lượng có thể chưa được biết tại thời điểm nhập liệu), `description` (nếu mô tả là tùy chọn), `end_date` (nếu một sự kiện có thể không có ngày kết thúc xác định).

> [!TIP]
> Khi thiết kế bảng, hãy suy nghĩ kỹ về ý nghĩa của `NULL` đối với từng cột. Nếu `NULL` có nghĩa là "không xác định" hoặc "không áp dụng", nó có thể được chấp nhận. Nếu `NULL` có nghĩa là "không có" hoặc "lỗi", thì cột đó có thể cần `NOT NULL` và một giá trị mặc định thay thế.

## IV. Giá Trị Mặc Định (`DEFAULT`): Tự Động Hóa và Tính Nhất Quán

Mặc dù ràng buộc `NOT NULL` đảm bảo rằng một cột luôn có giá trị, nhưng đôi khi chúng ta không biết giá trị cụ thể để cung cấp khi chèn một hàng mới. Trong những trường hợp này, việc sử dụng giá trị mặc định (`DEFAULT`) là một giải pháp hiệu quả. Nó cho phép bạn xác định một giá trị sẽ được tự động gán cho một cột nếu không có giá trị nào được cung cấp trong câu lệnh `INSERT`.

### 4.1. Khái Niệm và Lợi Ích của Giá Trị Mặc Định

Trong ví dụ về sản phẩm "Quần jean", chúng ta đã thay thế `NULL` bằng `0` để áp dụng ràng buộc `NOT NULL`. Tuy nhiên, `0` có thể không phải là giá trị mong muốn nếu giá thực sự chưa được xác định. Nếu chúng ta chèn một sản phẩm mới mà không biết giá, việc bắt buộc phải cung cấp một giá trị (do `NOT NULL`) có thể gây khó khăn cho người dùng hoặc ứng dụng.

Giá trị mặc định giải quyết vấn đề này bằng cách:
*   **Tự động điền dữ liệu:** Giảm công sức nhập liệu thủ công và đảm bảo rằng các cột luôn có giá trị hợp lệ, ngay cả khi không được chỉ định rõ ràng.
*   **Duy trì tính nhất quán:** Cung cấp một giá trị chuẩn cho các trường hợp không có dữ liệu cụ thể, giúp duy trì sự đồng nhất trong cơ sở dữ liệu.
*   **Phối hợp với `NOT NULL`:** Một cột có thể vừa có `NOT NULL` vừa có `DEFAULT`. Nếu không có giá trị nào được cung cấp, `DEFAULT` sẽ được sử dụng, từ đó thỏa mãn ràng buộc `NOT NULL`.

**Vibe Coding Perspective:** Việc sử dụng `DEFAULT` values tạo ra một "smooth workflow vibe". Nó giống như việc bạn có một quy trình làm việc được tối ưu hóa, nơi những chi tiết nhỏ được tự động xử lý, cho phép bạn tập trung vào những phần quan trọng hơn mà không lo lắng về các trường bị bỏ trống hoặc không hợp lệ. Điều này đặc biệt hữu ích khi các Agent AI của Antigravity tương tác với cơ sở dữ liệu, đảm bảo rằng ngay cả khi một Agent không cung cấp đầy đủ thông tin, dữ liệu vẫn được lưu trữ một cách hợp lệ.

### 4.2. Định Nghĩa Giá Trị Mặc Định Khi Tạo Bảng

Giống như `NOT NULL`, bạn có thể thiết lập giá trị mặc định ngay khi tạo bảng:

```sql
-- DROP TABLE IF EXISTS products; -- Lệnh này dùng để xóa bảng cũ nếu bạn muốn tạo lại từ đầu
-- Bạn có thể chạy lệnh này nếu muốn tạo lại bảng với các ràng buộc NOT NULL và DEFAULT
-- Tuy nhiên, trong bài này chúng ta sẽ ALTER bảng đã có.

-- Ví dụ về cách tạo bảng với DEFAULT từ đầu:
-- CREATE TABLE products (
--     id SERIAL PRIMARY KEY,
--     name VARCHAR(40) NOT NULL,
--     department VARCHAR(40) NOT NULL,
--     price INTEGER DEFAULT 0 NOT NULL, -- Giá mặc định là 0, và không được NULL
--     weight INTEGER DEFAULT 1          -- Trọng lượng mặc định là 1 (có thể NULL nếu không có DEFAULT)
-- );
```

Trong ví dụ này, nếu bạn chèn một sản phẩm mà không cung cấp `price`, nó sẽ tự động nhận giá trị `0`. Tương tự, `weight` sẽ là `1`.

> [!NOTE]
> Bạn có thể kết hợp `DEFAULT` với `NOT NULL`. Nếu một cột có `DEFAULT` và `NOT NULL`, cơ sở dữ liệu sẽ sử dụng giá trị mặc định nếu không có giá trị nào được cung cấp, và vì thế cột đó vẫn sẽ có giá trị và không vi phạm `NOT NULL`.

### 4.3. Thêm Giá Trị Mặc Định Vào Cột Đã Tồn Tại (`ALTER TABLE`)

Để thêm giá trị mặc định cho cột `price` (hiện đang là `NOT NULL` và không có `DEFAULT`) và cột `weight` (hiện có thể `NULL` và không có `DEFAULT`) trong bảng `products` hiện có:

```sql
-- Thêm giá trị mặc định cho cột price.
-- Lưu ý: Việc thêm DEFAULT không thay đổi các giá trị hiện có trong cột.
-- Nó chỉ áp dụng cho các lệnh INSERT trong tương lai nếu giá trị không được cung cấp.
ALTER TABLE products
ALTER COLUMN price SET DEFAULT 0; -- Sử dụng 0 làm giá trị mặc định cho giá chưa biết

-- Thêm giá trị mặc định cho cột weight
ALTER TABLE products
ALTER COLUMN weight SET DEFAULT 1; -- Trọng lượng mặc định là 1
```

Bây giờ, cột `price` có ràng buộc `NOT NULL` và giá trị mặc định là `0`. Cột `weight` có giá trị mặc định là `1` và vẫn có thể là `NULL` nếu một `INSERT` rõ ràng đặt nó là `NULL`. Tuy nhiên, nếu `INSERT` không cung cấp `weight`, nó sẽ tự động nhận `1`.

### 4.4. Minh Họa Cơ Chế Hoạt Động của `DEFAULT`

Hãy thử chèn một sản phẩm mới mà không cung cấp `price` và `weight`.

```sql
-- Chèn một sản phẩm mới, không cung cấp giá và trọng lượng
INSERT INTO products (name, department)
VALUES ('Găng tay', 'Dụng cụ');

-- Kiểm tra kết quả
SELECT * FROM products;
```

Kết quả sẽ tương tự như sau:

| id | name          | department | price | weight |
|----|---------------|------------|-------|--------|
| 1  | Áo sơ mi      | Quần áo    | 20    | 1      |
| 2  | Quần jean     | Quần áo    | 0     | 3      |
| 3  | Giày thể thao | Giày dép   | -10   | 2      |
| 4  | Mũ lưỡi trai  | Phụ kiện   | 15    | NULL   |
| 5  | Găng tay      | Dụng cụ    | 0     | 1      |

Sản phẩm "Găng tay" (id=5) đã tự động nhận `price` là `0` (do `DEFAULT` và `NOT NULL`) và `weight` là `1` (do `DEFAULT`). Lưu ý rằng các giá trị `NULL` và giá trị âm hiện có (`-10`) trong các hàng cũ không bị thay đổi bởi lệnh `ALTER TABLE ... SET DEFAULT`. Lệnh `DEFAULT` chỉ ảnh hưởng đến các thao tác `INSERT` *trong tương lai* khi cột không được chỉ định giá trị.

### 4.5. Các Loại Giá Trị Mặc Định Nâng Cao

Giá trị mặc định không chỉ giới hạn ở số nguyên hoặc chuỗi tĩnh. PostgreSQL cho phép sử dụng các hàm hoặc biểu thức làm giá trị mặc định, mang lại sự linh hoạt đáng kể:

*   **VARCHAR/TEXT:**
    ```sql
    ALTER TABLE products ADD COLUMN description TEXT DEFAULT 'Chưa có mô tả chi tiết';
    ```
*   **BOOLEAN:**
    ```sql
    ALTER TABLE products ADD COLUMN is_available BOOLEAN DEFAULT TRUE;
    ```
*   **DATE/TIMESTAMP:** Thường được sử dụng để tự động ghi lại thời điểm tạo hoặc cập nhật bản ghi.
    ```sql
    ALTER TABLE products ADD COLUMN created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP;
    ALTER TABLE products ADD COLUMN last_updated TIMESTAMP WITH TIME ZONE DEFAULT NOW();
    ```
    > [!NOTE]
    > *   `CURRENT_TIMESTAMP` và `NOW()` là các hàm trong PostgreSQL trả về thời điểm hiện tại của hệ thống (bao gồm cả múi giờ nếu sử dụng `WITH TIME ZONE`). Điều quan trọng là các hàm này được thực thi *tại thời điểm câu lệnh `INSERT` được chạy*, không phải tại thời điểm bảng được định nghĩa. Điều này đảm bảo rằng mỗi bản ghi mới sẽ có dấu thời gian chính xác của riêng nó.
    > *   `TIMESTAMP WITH TIME ZONE` là kiểu dữ liệu được khuyến nghị cho các trường thời gian để tránh nhầm lẫn về múi giờ.

*   **UUID (Globally Unique Identifier):**
    ```sql
    -- Trước tiên, bạn có thể cần kích hoạt extension 'uuid-ossp'
    -- CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
    -- Sau đó, thêm cột với default là UUID ngẫu nhiên
    ALTER TABLE products ADD COLUMN product_uuid UUID DEFAULT gen_random_uuid();
    ```
    Đây là cách tuyệt vời để tạo các định danh duy nhất không tuần tự, hữu ích cho các hệ thống phân tán hoặc khi bạn không muốn lộ số lượng bản ghi.

## V. Tóm Tắt và Thực Tiễn Tốt Nhất

Trong chương này, chúng ta đã khám phá các khái niệm cơ bản và cách triển khai các ràng buộc quan trọng trong PostgreSQL để xây dựng một nền tảng dữ liệu mạnh mẽ, đáng tin cậy.

*   **Xác thực dữ liệu** là cần thiết ở cả cấp độ ứng dụng và cơ sở dữ liệu. Xác thực cấp ứng dụng cung cấp phản hồi nhanh cho người dùng, trong khi xác thực cấp cơ sở dữ liệu đóng vai trò là tuyến phòng thủ cuối cùng, đảm bảo tính toàn vẹn dữ liệu bất kể nguồn gốc.
*   Chúng ta đã học cách tạo **cơ sở dữ liệu** và **bảng** trong PgAdmin, cũng như cách xem cấu trúc và dữ liệu của chúng.
*   **Ràng buộc `NOT NULL`** được sử dụng để đảm bảo rằng một cột luôn có giá trị và không được phép là `NULL`. Điều này loại bỏ sự mơ hồ và tăng cường độ tin cậy của dữ liệu.
*   Khi áp dụng `NOT NULL` cho một cột đã tồn tại chứa giá trị `NULL`, cần phải **cập nhật các giá trị `NULL`** đó thành một giá trị hợp lệ trước khi áp dụng ràng buộc. Toán tử `WHERE column IS NULL` được sử dụng để xác định các giá trị `NULL`.
*   **Giá trị mặc định (`DEFAULT`)** cho phép bạn tự động gán một giá trị cho một cột nếu không có giá trị nào được cung cấp trong câu lệnh `INSERT`. Điều này giúp duy trì tính toàn vẹn dữ liệu ngay cả khi thông tin không đầy đủ, đồng thời giảm thiểu công sức nhập liệu.
*   Các ràng buộc và giá trị mặc định có thể được thiết lập khi **tạo bảng** hoặc **thay đổi cấu trúc bảng** đã tồn tại bằng lệnh `ALTER TABLE`.
*   Việc lựa chọn sử dụng `NOT NULL` hay cho phép `NULL`, cũng như giá trị `DEFAULT` nào, cần được cân nhắc kỹ lưỡng dựa trên yêu cầu nghiệp vụ của ứng dụng và ý nghĩa thực tế của dữ liệu.

**Tổng kết về AI Coding và Antigravity IDE:**
Trong bối cảnh phát triển phần mềm hiện đại, đặc biệt với sự hỗ trợ của các hệ thống AI Agentic như Antigravity IDE, việc xây dựng một cơ sở dữ liệu vững chắc càng trở nên quan trọng. Các ràng buộc cơ sở dữ liệu không chỉ là quy tắc kỹ thuật; chúng là những "luật bất biến" của dữ liệu, tạo ra một nền tảng đáng tin cậy cho mọi hoạt động của hệ thống, dù được mã hóa thủ công hay tự động bởi AI.

*   **Guardrails cho Agentic AI:** Các ràng buộc cung cấp một lớp bảo vệ thiết yếu, hoạt động như những "hàng rào bảo vệ" cho các Agent AI của Antigravity. Chúng đảm bảo rằng ngay cả khi một Agent có thể tạo ra một hành động hoặc dữ liệu không mong muốn do lỗi logic hoặc hiểu sai ngữ cảnh, cơ sở dữ liệu sẽ từ chối thao tác đó, ngăn chặn "bad vibes" (dữ liệu sai lệch) lan truyền khắp hệ thống.
*   **Nền tảng của Vibe Coding:** Một cơ sở dữ liệu được định nghĩa tốt với các ràng buộc phù hợp sẽ tỏa ra một "integrity vibe" mạnh mẽ. Điều này có nghĩa là bạn và các Agent AI của bạn có thể tin tưởng vào dữ liệu, tập trung vào việc giải quyết các vấn đề nghiệp vụ phức tạp hơn mà không phải lo lắng về tính chính xác của thông tin cơ bản. Nó tạo điều kiện cho một luồng công việc mượt mà, nơi các Agent có thể thực thi các kế hoạch tự động, gọi subagent, và tương tác với các file một cách tự tin.
*   **Kỹ thuật cho sự kiên cường:** Bằng cách áp dụng các kỹ thuật này, bạn đang xây dựng các hệ thống không chỉ hoạt động tốt mà còn kiên cường trước các lỗi tiềm ẩn, dù chúng đến từ con người hay từ các hệ thống AI tự động.

Bằng cách nắm vững và áp dụng các kỹ thuật này, bạn sẽ có thể xây dựng các cơ sở dữ liệu mạnh mẽ, đáng tin cậy và dễ quản lý hơn, giảm thiểu rủi ro từ dữ liệu không hợp lệ và tạo ra một "vibe" tích cực về độ tin cậy trong toàn bộ kiến trúc phần mềm của bạn.

<!-- REVIEWED_BY_AGENT -->
