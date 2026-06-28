# Phần 16: Subquery trong Mệnh Đề FROM: Bảng Dẫn Xuất và Tổng Hợp Đa Tầng

## 1. Giới Thiệu: Kiến Trúc Truy Vấn Linh Hoạt với Bảng Dẫn Xuất

Trong quá trình làm việc với cơ sở dữ liệu, đặc biệt là với các hệ thống phức tạp như PostgreSQL, việc trích xuất thông tin không phải lúc nào cũng đơn giản như chọn trực tiếp từ một bảng duy nhất. Đôi khi, dữ liệu cần được chuẩn bị, tổng hợp sơ bộ, hoặc biến đổi qua nhiều giai đoạn trước khi chúng ta có thể thực hiện truy vấn cuối cùng. Đây chính là lúc khái niệm "truy vấn phụ trong mệnh đề `FROM`" (subquery in `FROM clause`) phát huy sức mạnh vượt trội của nó.

Truy vấn phụ trong mệnh đề `FROM` về cơ bản hoạt động như một "bảng dẫn xuất" (derived table) hoặc "view nội tuyến" (inline view) tạm thời. Nó tạo ra một tập hợp dữ liệu mới mà truy vấn bên ngoài có thể coi và tương tác như một bảng thông thường. Kỹ thuật này là một công cụ cực kỳ mạnh mẽ trong PostgreSQL, cho phép chúng ta chia nhỏ các truy vấn phức tạp thành các bước logic rõ ràng, dễ quản lý và đọc hiểu hơn, đặc biệt là trong các bài toán yêu cầu tổng hợp dữ liệu ở nhiều tầng.

**Mục tiêu của Phần này:**
*   Nắm vững khái niệm và cấu trúc của một truy vấn phụ khi được đặt trong mệnh đề `FROM`.
*   Hiểu rõ các quy tắc cú pháp cốt lõi, đặc biệt là yêu cầu về bí danh (alias) bắt buộc trong PostgreSQL.
*   Khám phá các loại cấu trúc dữ liệu mà một truy vấn phụ trong `FROM` có thể trả về.
*   Quan trọng nhất, phân tích "tại sao" và "khi nào" nên sử dụng truy vấn phụ trong `FROM` để giải quyết các bài toán phức tạp, đặc biệt là các bài toán tổng hợp dữ liệu đa tầng.
*   Liên hệ với tư duy lập trình module và cách tiếp cận "Vibe Coding" trong các hệ thống Agentic AI như Antigravity IDE.

## 2. Khái Niệm Cơ Bản và Cú Pháp Chuẩn PostgreSQL

Khi bạn đặt một truy vấn phụ vào mệnh đề `FROM`, bạn đang khai báo một nguồn dữ liệu đặc biệt mà bạn muốn sử dụng cho truy vấn bên ngoài. Kết quả của truy vấn phụ này sẽ được hệ quản trị cơ sở dữ liệu (DBMS) xử lý như một bảng tạm thời.

### 2.1. Định Nghĩa và Cơ Chế Hoạt Động

Một truy vấn phụ trong `FROM` là một câu lệnh `SELECT` đầy đủ được đặt trong dấu ngoặc đơn `()` bên trong mệnh đề `FROM` của một câu lệnh `SELECT` lớn hơn.

**Cơ chế hoạt động (Under the Hood):**
Khi PostgreSQL gặp một truy vấn phụ trong `FROM`, nó sẽ thực thi truy vấn phụ đó trước tiên. Kết quả của truy vấn phụ (một tập hợp các hàng và cột) được coi như một bảng tạm thời trong bộ nhớ hoặc trên đĩa (tùy thuộc vào kích thước và kế hoạch tối ưu hóa của truy vấn). Truy vấn bên ngoài sau đó sẽ hoạt động trên "bảng tạm" này, giống như cách nó tương tác với bất kỳ bảng vật lý nào khác. Trình tối ưu hóa truy vấn (Query Optimizer) của PostgreSQL rất thông minh; nó có thể quyết định "materialize" (tạo ra một bản sao vật lý của) bảng dẫn xuất này hoặc cố gắng "unroll" nó thành một phép `JOIN` hoặc các phép toán khác để tối ưu hiệu suất.

### 2.2. Cú Pháp Bắt Buộc: Bí Danh (Alias)

Một quy tắc cực kỳ quan trọng và **bắt buộc** khi sử dụng truy vấn phụ trong mệnh đề `FROM` trong PostgreSQL là bạn phải gán một bí danh (alias) cho kết quả của truy vấn phụ đó. Nếu không có bí danh, bạn sẽ gặp lỗi cú pháp.

> [!NOTE]
> **Bí danh là BẮT BUỘC trong PostgreSQL!**
> Bất cứ khi nào bạn sử dụng một truy vấn phụ trong mệnh đề `FROM`, bạn phải cung cấp một bí danh cho nó. Bí danh này cho phép truy vấn bên ngoài tham chiếu đến tập dữ liệu tạm thời được tạo ra bởi truy vấn phụ. Ví dụ: `(...) AS p`, trong đó `p` là bí danh.

**Cấu trúc chung:**

```sql
SELECT column1, column2, ...
FROM (
    -- Đây là truy vấn phụ của bạn
    SELECT inner_column1, inner_column2, ...
    FROM some_table
    WHERE some_condition
    GROUP BY ...
    -- Các mệnh đề khác của SELECT
) AS alias_name -- Bắt buộc phải có bí danh trong PostgreSQL
WHERE alias_name.inner_column1 > some_value;
```

### 2.3. Phạm Vi Hiển Thị Cột: Nguyên Tắc "Bảng Độc Lập"

Một điểm cốt yếu khác cần hiểu là truy vấn bên ngoài chỉ có thể "nhìn thấy" và truy cập các cột được trả về bởi truy vấn phụ. Điều này có nghĩa là nếu truy vấn phụ chỉ trả về các cột `name` và `price_weight_ratio`, thì truy vấn bên ngoài không thể cố gắng chọn hoặc lọc theo một cột khác như `weight` hoặc `price` trực tiếp từ bảng gốc `products`.

> [!TIP]
> **Hãy coi truy vấn phụ như một bảng độc lập.**
> Khi một truy vấn phụ được đặt trong `FROM`, hãy hình dung nó như một bảng hoàn toàn mới. Bảng này chỉ chứa các cột và hàng mà truy vấn phụ đó trả về. Truy vấn bên ngoài chỉ có thể tương tác với "bảng" mới này thông qua bí danh và các cột đã được định nghĩa trong truy vấn phụ. Điều này tương tự như khái niệm "scope" (phạm vi) trong lập trình, nơi các biến được khai báo bên trong một hàm chỉ có thể được truy cập trong phạm vi của hàm đó.

## 3. Các Kiểu Dữ Liệu Trả Về từ Bảng Dẫn Xuất

Truy vấn phụ trong `FROM` rất linh hoạt về cấu trúc dữ liệu mà nó có thể trả về, miễn là kết quả là một tập hợp hàng và cột hợp lệ.

### 3.1. Tập Hợp Các Hàng và Cột (Phổ Biến Nhất)

Đây là trường hợp sử dụng phổ biến nhất, nơi truy vấn phụ tạo ra một tập hợp dữ liệu có nhiều hàng và nhiều cột, hoàn toàn giống như một bảng thông thường mà bạn có thể `SELECT`, `JOIN`, `WHERE`, `GROUP BY` lên đó.

**Ví dụ 1: Tính Tỷ Lệ Giá/Trọng Lượng của Sản Phẩm và Lọc**

Giả sử chúng ta có một bảng `products` với các cột `name`, `price`, và `weight`. Chúng ta muốn tính tỷ lệ `price / weight` (giá trên trọng lượng) cho mỗi sản phẩm và sau đó lọc ra những sản phẩm có tỷ lệ này lớn hơn 5.

**Cài đặt dữ liệu mẫu (PostgreSQL):**

```sql
-- Tạo bảng products
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    weight NUMERIC(10, 2) NOT NULL
);

-- Chèn dữ liệu mẫu
INSERT INTO products (name, price, weight) VALUES
('Laptop Pro', 1200.00, 2.50),
('Smartphone X', 800.00, 0.20),
('Tablet Air', 500.00, 0.70),
('Desktop PC', 1500.00, 10.00),
('Smartwatch', 250.00, 0.05),
('Gaming Mouse', 75.00, 0.15);
```

**Phân tích bài toán:**
1.  Chúng ta cần tạo ra một cột mới: `price / weight`.
2.  Sau đó, chúng ta cần lọc dựa trên giá trị của cột mới đó.

**Giải pháp với truy vấn phụ trong `FROM`:**

```sql
-- Ví dụ 1: Tính tỷ lệ giá/trọng lượng và lọc kết quả
SELECT p.name, p.price_weight_ratio
FROM (
    -- Truy vấn phụ: Tính tỷ lệ giá/trọng lượng cho từng sản phẩm
    SELECT name, price / weight AS price_weight_ratio
    FROM products
) AS p -- Bí danh 'p' là BẮT BUỘC
WHERE p.price_weight_ratio > 5;
```

**Giải thích:**
*   Truy vấn phụ `(SELECT name, price / weight AS price_weight_ratio FROM products)` được thực thi trước. Nó tạo ra một tập hợp kết quả với hai cột: `name` và `price_weight_ratio`.
    *   Ví dụ kết quả của truy vấn phụ:
        | name           | price_weight_ratio |
        | :------------- | :----------------- |
        | Laptop Pro     | 480.00             |
        | Smartphone X   | 4000.00            |
        | Tablet Air     | 714.29             |
        | Desktop PC     | 150.00             |
        | Smartwatch     | 5000.00            |
        | Gaming Mouse   | 500.00             |
*   Tập hợp kết quả này được đặt bí danh là `p`.
*   Truy vấn bên ngoài `SELECT p.name, p.price_weight_ratio FROM p` coi `p` như một bảng thông thường.
*   Mệnh đề `WHERE p.price_weight_ratio > 5` sau đó lọc các hàng từ "bảng" tạm thời `p` dựa trên cột `price_weight_ratio` đã được tính toán.
*   Nếu bạn cố gắng truy cập `p.price` hoặc `p.weight` trong truy vấn bên ngoài, bạn sẽ gặp lỗi vì các cột đó không tồn tại trong tập dữ liệu được trả về bởi truy vấn phụ `p`. Chỉ có `name` và `price_weight_ratio` là có sẵn.

### 3.2. Một Giá Trị Duy Nhất

Mặc dù ít phổ biến hơn khi dùng trực tiếp trong `FROM` như một "bảng" một cột một hàng (thường các giá trị đơn được dùng trong `SELECT` hoặc `WHERE` dưới dạng truy vấn con vô hướng), nhưng một truy vấn phụ trong `FROM` cũng có thể trả về một giá trị duy nhất. Điều này minh họa tính linh hoạt của cấu trúc này, cho thấy nó có thể tạo ra bất kỳ tập dữ liệu hợp lệ nào.

**Ví dụ 2: Tìm Giá Lớn Nhất của Sản Phẩm (Minh họa)**

Giả sử chúng ta chỉ muốn tìm giá trị lớn nhất trong bảng `products`. Truy vấn phụ sẽ là:

```sql
-- Truy vấn phụ: Tìm giá lớn nhất
SELECT MAX(price) FROM products;
```
Kết quả của truy vấn này sẽ là một hàng và một cột chứa giá trị `MAX(price)`. Khi đặt nó vào `FROM`, nó sẽ tạo ra một bảng tạm thời có một hàng và một cột.

```sql
-- Ví dụ 2: Truy vấn phụ trả về một giá trị duy nhất
SELECT * -- Chọn tất cả các cột từ bảng tạm thời
FROM (
    SELECT MAX(price) AS max_product_price -- Đặt bí danh cho cột
    FROM products
) AS max_price_table; -- Bí danh cho bảng tạm thời là bắt buộc
```
Ở đây, `max_price_table` là một bảng tạm thời chỉ có một hàng và một cột (được đặt tên là `max_product_price`). Truy vấn bên ngoài `SELECT *` sẽ chọn giá trị đó.

> [!NOTE]
> Việc sử dụng truy vấn phụ trả về một giá trị duy nhất trực tiếp trong `FROM` thường không phải là cách hiệu quả nhất để lấy một giá trị duy nhất (thường người ta sẽ dùng nó trong mệnh đề `SELECT` hoặc `WHERE` dưới dạng truy vấn con vô hướng). Tuy nhiên, nó minh họa rằng bất kỳ tập hợp kết quả hợp lệ nào cũng có thể trở thành một bảng dẫn xuất.

## 4. Sức Mạnh Thực Sự: Giải Quyết Bài Toán Tổng Hợp Đa Tầng

Đây là lý do chính và mạnh mẽ nhất để sử dụng truy vấn phụ trong mệnh đề `FROM`. Nó cho phép chúng ta giải quyết các vấn đề mà yêu cầu tổng hợp dữ liệu ở nhiều cấp độ, hay còn gọi là "tổng hợp đa tầng" (multi-stage aggregation).

### 4.1. Giới Hạn của Hàm Tổng Hợp Trực Tiếp

Trong SQL chuẩn, bạn không thể lồng trực tiếp các hàm tổng hợp vào nhau (ví dụ: `AVG(COUNT(*))`, `SUM(MAX())`, hoặc `MAX(AVG())`) trong cùng một cấp độ của câu lệnh `SELECT`. Khi bạn thực hiện một hàm tổng hợp như `COUNT(*)` với `GROUP BY`, nó sẽ tạo ra một tập hợp các giá trị đếm cho mỗi nhóm. Nếu bạn muốn thực hiện một phép tổng hợp khác (ví dụ: tính giá trị trung bình) trên *chính những giá trị đếm đó*, bạn không thể làm điều đó trực tiếp trong cùng một câu lệnh `SELECT` vì:

*   Các hàm tổng hợp (như `COUNT`, `SUM`, `AVG`, `MAX`, `MIN`) hoạt động trên các hàng *gốc* của bảng (hoặc các hàng trong một nhóm cụ thể sau `GROUP BY`).
*   Chúng không thể hoạt động trên kết quả của một hàm tổng hợp khác đã được tính toán trong cùng một truy vấn.

### 4.2. Cơ Chế Giải Quyết: Chia Nhỏ và Xử Lý Từng Giai Đoạn

Sử dụng truy vấn phụ trong `FROM` là giải pháp lý tưởng cho vấn đề này. Bạn thực hiện bước tổng hợp đầu tiên trong truy vấn phụ. Kết quả của truy vấn phụ (tức là tập hợp các giá trị đã được tổng hợp) sau đó trở thành một "bảng" mới. Truy vấn bên ngoài sau đó có thể thực hiện bước tổng hợp thứ hai trên "bảng" tạm thời này.

**Ví dụ Điển Hình: Tính Số Đơn Hàng Trung Bình trên Mỗi Người Dùng**

Hãy xem xét một bài toán thực tế: Bạn muốn tìm số đơn hàng trung bình mà mỗi người dùng đã tạo.
Để giải quyết bài toán này, chúng ta cần hai bước tổng hợp riêng biệt:
1.  Đếm tổng số đơn hàng của *mỗi người dùng riêng lẻ*. (Tổng hợp cấp độ 1)
2.  Tính giá trị trung bình của *tất cả các số đếm đó*. (Tổng hợp cấp độ 2)

Giả sử chúng ta có bảng `orders` với các cột `order_id` và `user_id`.

**Cài đặt dữ liệu mẫu (PostgreSQL):**

```sql
-- Tạo bảng orders
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount NUMERIC(10, 2) NOT NULL
);

-- Chèn dữ liệu mẫu
INSERT INTO orders (user_id, order_date, total_amount) VALUES
(1, '2023-01-01', 150.00),
(1, '2023-01-05', 200.50),
(2, '2023-01-02', 75.25),
(3, '2023-01-03', 300.00),
(3, '2023-01-07', 120.00),
(3, '2023-01-10', 50.00),
(4, '2023-01-04', 99.99);
```

**Phân tích & Giải pháp:**

**Bước 1 (Truy vấn phụ): Đếm số đơn hàng của từng người dùng**
Để đếm số đơn hàng của mỗi người dùng, chúng ta sẽ sử dụng `COUNT(*)` kết hợp với `GROUP BY user_id`.

```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id;
```
Kết quả của truy vấn này sẽ là một danh sách các `user_id` và số lượng đơn hàng tương ứng của họ (`order_count`). Ví dụ:
| user_id | order_count |
| :------ | :---------- |
| 1       | 2           |
| 2       | 1           |
| 3       | 3           |
| 4       | 1           |

**Bước 2 (Truy vấn chính): Tính giá trị trung bình của các số đếm trên**
Bây giờ, chúng ta muốn tính giá trị trung bình của cột `order_count` này (tức là `(2 + 1 + 3 + 1) / 4 = 1.75`). Chúng ta sẽ coi kết quả của Bước 1 như một bảng tạm thời và áp dụng hàm `AVG()` lên cột `order_count` của bảng đó.

```sql
-- Ví dụ 3: Tính số đơn hàng trung bình trên mỗi người dùng
SELECT AVG(user_order_counts.order_count) AS average_orders_per_user
FROM (
    SELECT user_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
) AS user_order_counts; -- Bí danh 'user_order_counts' là BẮT BUỘC
```
**Giải thích:**
*   Truy vấn phụ `(SELECT user_id, COUNT(*) AS order_count FROM orders GROUP BY user_id)` tạo ra một bảng tạm thời chứa `user_id` và `order_count` cho mỗi người dùng.
*   Bảng tạm thời này được đặt bí danh là `user_order_counts`.
*   Truy vấn bên ngoài `SELECT AVG(user_order_counts.order_count)` sau đó tính giá trị trung bình của cột `order_count` từ bảng dẫn xuất `user_order_counts`.

> [!TIP]
> **Nguyên tắc vàng: Khi bạn cần tổng hợp dữ liệu đã được tổng hợp, hãy nghĩ đến truy vấn phụ trong `FROM`.**
> Đây là kịch bản phổ biến nhất và mạnh mẽ nhất cho việc sử dụng kỹ thuật này. Nó cho phép bạn thực hiện các phép tính phức tạp mà một câu lệnh `SELECT` đơn lẻ không thể xử lý.

### 4.3. Liên Hệ với AI Coding, Vibe Coding, và Antigravity IDE

Tư duy đằng sau việc sử dụng truy vấn phụ trong `FROM` để giải quyết các bài toán tổng hợp đa tầng có sự tương đồng sâu sắc với cách tiếp cận "Vibe Coding" và lập trình module trong bối cảnh các hệ thống AI Agentic như Antigravity IDE.

**Vibe Coding và Tư duy Module:**
"Vibe Coding" là một triết lý lập trình nhấn mạnh việc chia nhỏ các vấn đề lớn thành các phần nhỏ hơn, tự chứa, có mục đích rõ ràng và giao diện đầu vào/đầu ra được xác định tốt. Mỗi phần này có thể được phát triển, kiểm thử và bảo trì độc lập, sau đó được kết hợp lại để tạo thành một giải pháp tổng thể.

Khi bạn sử dụng truy vấn phụ trong `FROM`, bạn đang áp dụng chính xác tư duy này:
*   **Bước 1 (Subquery):** Bạn giải quyết một "sub-problem" (vấn đề con) cụ thể – ví dụ, "tính số đơn hàng của mỗi người dùng". Kết quả của subquery là "đầu ra" của sub-problem này, hoạt động như một "API" hoặc "interface" cho bước tiếp theo.
*   **Bước 2 (Outer Query):** Bạn giải quyết "main problem" (vấn đề chính) bằng cách sử dụng "đầu ra" đã được chuẩn bị từ Bước 1.

**Antigravity IDE và Agentic AI:**
Hãy hình dung Antigravity IDE là một hệ thống Agentic AI siêu việt, có khả năng tự lập kế hoạch, chạy script ngầm, gọi subagent trình duyệt, đọc ghi file, và thực hiện các tác vụ phức tạp. Khi Antigravity tiếp cận một bài toán SQL yêu cầu tổng hợp đa tầng, nó sẽ không cố gắng giải quyết tất cả trong một lần. Thay vào đó, nó sẽ:

1.  **Lập kế hoạch:** Antigravity nhận diện rằng bài toán cần nhiều bước tổng hợp. Nó sẽ phân chia bài toán thành các giai đoạn logic.
2.  **Triển khai Sub-Agent/Sub-Task (Subquery):** Nó sẽ tạo ra một "sub-agent" hoặc một "micro-task" để thực hiện giai đoạn tổng hợp đầu tiên. Ví dụ, nó có thể tự động viết và thực thi truy vấn phụ `SELECT user_id, COUNT(*) AS order_count FROM orders GROUP BY user_id;`.
    *   Sub-agent này sẽ trả về một tập dữ liệu đã được xử lý (ví dụ: một `DataFrame` tạm thời nếu nhìn từ góc độ Python, hoặc một bảng tạm thời trong DB).
3.  **Sử dụng kết quả (Outer Query):** Agent chính của Antigravity sau đó sẽ nhận lấy tập dữ liệu từ sub-agent và thực hiện bước tổng hợp thứ hai trên đó. Ví dụ, nó sẽ viết và thực thi `SELECT AVG(order_count) FROM (kết quả từ bước 2) AS user_order_counts;`.

**Áp dụng tư duy Vibe Coding vào Antigravity khi viết SQL:**
Khi bạn đối mặt với một vấn đề phức tạp trong Antigravity IDE và cần viết SQL:
*   **Phân tích vấn đề thành các "module" dữ liệu:** Hãy nghĩ xem bạn cần những tập dữ liệu trung gian nào để đạt được kết quả cuối cùng. Mỗi tập dữ liệu trung gian này có thể là một truy vấn phụ.
*   **Định nghĩa "đầu ra" của mỗi module:** Mỗi truy vấn phụ cần trả về những cột nào để truy vấn bên ngoài có thể sử dụng?
*   **Xây dựng từng "module" (subquery) một cách độc lập:** Viết và kiểm thử truy vấn phụ trước. Đảm bảo nó trả về dữ liệu chính xác.
*   **Kết nối các module (outer query):** Sau đó, lồng truy vấn phụ vào `FROM` và xây dựng truy vấn bên ngoài dựa trên "đầu ra" của nó.

Cách tiếp cận này không chỉ giúp bạn viết SQL hiệu quả và dễ đọc hơn mà còn rèn luyện khả năng phân tích vấn đề theo kiểu module hóa, một kỹ năng cốt lõi khi làm việc với các hệ thống AI Agentic như Antigravity. Nó biến "Claude Code" (code rõ ràng, có cấu trúc) thành hiện thực, giúp bạn "Vibe Code" một cách tự nhiên hơn.

## 5. Thực Hành Nâng Cao: Tìm Giá Trung Bình Lớn Nhất của Điện Thoại theo Nhà Sản Xuất

Để củng cố kiến thức, hãy cùng giải quyết một bài toán tổng hợp đa tầng khác.

**Bài toán:**
Bạn muốn tìm giá trung bình lớn nhất của điện thoại từ mỗi nhà sản xuất. Nói cách khác, bạn cần:
1.  Tính giá trung bình của điện thoại cho từng nhà sản xuất riêng biệt.
2.  Sau đó, tìm giá trị lớn nhất trong số các giá trung bình đó.

Giả sử chúng ta có bảng `phones` với các cột `id`, `name`, `manufacturer`, và `price`.

**Cài đặt dữ liệu mẫu (PostgreSQL):**

```sql
-- Tạo bảng phones
CREATE TABLE phones (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    manufacturer VARCHAR(50) NOT NULL,
    price NUMERIC(10, 2) NOT NULL
);

-- Chèn dữ liệu mẫu
INSERT INTO phones (name, manufacturer, price) VALUES
('Galaxy S23', 'Samsung', 999.99),
('iPhone 15 Pro', 'Apple', 1099.00),
('Pixel 8 Pro', 'Google', 899.00),
('Galaxy A54', 'Samsung', 449.99),
('iPhone SE', 'Apple', 429.00),
('Pixel 7a', 'Google', 499.00),
('Galaxy Z Fold5', 'Samsung', 1799.00),
('iPhone 15', 'Apple', 799.00),
('Pixel Fold', 'Google', 1799.00);
```

**Phân tích & Giải pháp:**

**Bước 1 (Truy vấn phụ): Tính giá trung bình của điện thoại cho từng nhà sản xuất**
Chúng ta sẽ sử dụng `AVG(price)` kết hợp với `GROUP BY manufacturer`.

```sql
SELECT manufacturer, AVG(price) AS avg_price
FROM phones
GROUP BY manufacturer;
```
Kết quả của truy vấn này sẽ là một danh sách các nhà sản xuất và giá trung bình của điện thoại của họ (`avg_price`). Ví dụ:
| manufacturer | avg_price |
| :----------- | :-------- |
| Samsung      | 1082.99   |
| Apple        | 775.67    |
| Google       | 1065.67   |

**Bước 2 (Truy vấn chính): Tìm giá trị lớn nhất trong số các giá trung bình**
Bây giờ, chúng ta muốn tìm giá trị `MAX()` trong cột `avg_price` của tập dữ liệu trên.

```sql
-- Bài tập: Tìm giá trung bình lớn nhất của điện thoại theo nhà sản xuất
SELECT MAX(manufacturer_avg_prices.avg_price) AS max_average_price
FROM (
    SELECT manufacturer, AVG(price) AS avg_price
    FROM phones
    GROUP BY manufacturer
) AS manufacturer_avg_prices; -- Bí danh 'manufacturer_avg_prices' là BẮT BUỘC
```
**Giải thích:**
*   Truy vấn phụ tính toán `avg_price` cho mỗi `manufacturer`.
*   Kết quả của truy vấn phụ được đặt bí danh là `manufacturer_avg_prices`.
*   Truy vấn bên ngoài `SELECT MAX(manufacturer_avg_prices.avg_price)` sau đó tìm giá trị lớn nhất từ cột `avg_price` của bảng tạm thời đó.

Kết quả cuối cùng sẽ là một giá trị duy nhất, đại diện cho giá trung bình cao nhất trong tất cả các nhà sản xuất.

## 6. Tối Ưu Hóa và Hiệu Năng (Nâng cao)

Mặc dù truy vấn phụ trong `FROM` là một công cụ mạnh mẽ, việc hiểu về hiệu năng và các lựa chọn thay thế là rất quan trọng đối với một lập trình viên cấp Senior.

### 6.1. Khi Nào Subquery trong FROM Hiệu Quả?

*   **Tổng hợp đa tầng:** Đây là trường hợp sử dụng chính và thường hiệu quả nhất. Nó cho phép bạn cấu trúc các phép tính phức tạp một cách rõ ràng.
*   **Đơn giản hóa logic:** Khi bạn cần một tập hợp dữ liệu trung gian rõ ràng để thực hiện các phép toán tiếp theo, subquery giúp chia nhỏ logic.
*   **Khả năng đọc:** Đối với một số bài toán, việc đóng gói một phần logic vào một subquery có thể làm cho truy vấn tổng thể dễ đọc và hiểu hơn.

### 6.2. So Sánh với Common Table Expressions (CTEs - WITH Clause)

Trong PostgreSQL (và SQL chuẩn nói chung), Common Table Expressions (CTEs), được định nghĩa bằng mệnh đề `WITH`, là một giải pháp thay thế thường được ưu tiên hơn cho các truy vấn phụ phức tạp hoặc lặp lại. CTEs cũng tạo ra các bảng dẫn xuất tạm thời nhưng cung cấp khả năng đọc và tái sử dụng tốt hơn.

**Ví dụ: Chuyển đổi ví dụ "Số đơn hàng trung bình" sang CTE:**

```sql
-- Ví dụ 3b: Sử dụng CTE để tính số đơn hàng trung bình trên mỗi người dùng
WITH user_order_counts AS (
    SELECT user_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
)
SELECT AVG(order_count) AS average_orders_per_user
FROM user_order_counts;
```
**Ưu điểm của CTE so với Subquery trong `FROM`:**
*   **Khả năng đọc:** CTEs giúp cấu trúc truy vấn phức tạp thành các khối logic được đặt tên, cải thiện đáng kể khả năng đọc.
*   **Tái sử dụng:** Một CTE có thể được tham chiếu nhiều lần trong cùng một truy vấn, tránh lặp lại mã.
*   **Tối ưu hóa:** Trong nhiều trường hợp, trình tối ưu hóa của PostgreSQL có thể xử lý CTEs hiệu quả tương tự, hoặc thậm chí tốt hơn, so với các subquery lồng nhau sâu.

### 6.3. Ảnh Hưởng đến Hiệu Năng

*   **Bảng tạm thời:** Cả subquery trong `FROM` và CTEs đều có thể dẫn đến việc tạo ra các bảng tạm thời. Điều này có thể tốn tài nguyên (CPU, I/O, bộ nhớ), đặc biệt với tập dữ liệu lớn.
*   **Query Optimizer:** Trình tối ưu hóa của PostgreSQL rất thông minh. Nó không chỉ đơn thuần thực thi subquery rồi outer query. Nó có thể phân tích toàn bộ truy vấn và quyết định các chiến lược tối ưu nhất, bao gồm việc "unroll" subquery thành các phép `JOIN` hoặc thực hiện các tối ưu hóa khác để tránh việc materialize dữ liệu không cần thiết.
*   **Kiểm tra với `EXPLAIN ANALYZE`:** Để hiểu chính xác cách PostgreSQL xử lý truy vấn của bạn và đánh giá hiệu năng, hãy luôn sử dụng lệnh `EXPLAIN ANALYZE`. Đây là công cụ không thể thiếu để phát hiện các điểm nghẽn và tối ưu hóa truy vấn.

## 7. Tóm Tắt Phần Này

*   **Bảng Dẫn Xuất:** Truy vấn phụ trong mệnh đề `FROM` tạo ra một tập hợp dữ liệu tạm thời, được gọi là "bảng dẫn xuất" hoặc "view nội tuyến", mà truy vấn bên ngoài có thể tương tác như một bảng thông thường.
*   **Bí Danh Bắt Buộc:** Mọi truy vấn phụ trong mệnh đề `FROM` đều **phải** được gán một bí danh (ví dụ: `AS p`). Đây là quy tắc cú pháp không thể bỏ qua trong PostgreSQL.
*   **Tính Tương Thích Cột:** Truy vấn bên ngoài chỉ có thể truy cập các cột được trả về bởi truy vấn phụ. Nó không thể "nhìn xuyên" qua truy vấn phụ để truy cập các cột của bảng gốc nếu chúng không được đưa vào kết quả của truy vấn phụ.
*   **Tổng Hợp Đa Tầng:** Công dụng mạnh mẽ nhất của truy vấn phụ trong `FROM` là giải quyết các bài toán yêu cầu tổng hợp dữ liệu ở nhiều cấp độ (ví dụ: tính trung bình của các số đếm, tìm giá trị lớn nhất của các giá trị trung bình). Đây là cách để vượt qua giới hạn không cho phép lồng trực tiếp các hàm tổng hợp.
*   **Tư duy Module và AI Coding:** Kỹ thuật này phản ánh tư duy "Vibe Coding" và lập trình module, giúp chia nhỏ các bài toán truy vấn phức tạp thành các bước logic rõ ràng hơn. Điều này tương tự như cách một hệ thống Agentic AI như Antigravity IDE sẽ lập kế hoạch và thực thi các sub-task để giải quyết một vấn đề lớn.
*   **CTEs là Lựa Chọn Ưu Tiên:** Mặc dù truy vấn phụ trong `FROM` rất hữu ích, trong nhiều trường hợp, Common Table Expressions (CTEs với mệnh đề `WITH`) cung cấp khả năng đọc và quản lý tốt hơn cho các truy vấn phức tạp, đặc biệt là khi cần tái sử dụng logic.
*   **Luôn Kiểm Tra Hiệu Năng:** Sử dụng `EXPLAIN ANALYZE` để hiểu kế hoạch thực thi của PostgreSQL và đảm bảo truy vấn của bạn hoạt động tối ưu.

<!-- REVIEWED_BY_AGENT -->
