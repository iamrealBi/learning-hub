# Phần 19: Correlated Subquery và SELECT Không Có FROM – Nâng Cao Hiệu Suất và Linh Hoạt Truy Vấn

Chào mừng bạn đến với Phần 19 của khóa học chuyên sâu về PostgreSQL. Trong chương này, chúng ta sẽ đi sâu vào hai kỹ thuật truy vấn nâng cao nhưng cực kỳ mạnh mẽ: Correlated Subquery (Truy vấn phụ tương quan) và cách sử dụng câu lệnh `SELECT` mà không cần mệnh đề `FROM`. Đây là những công cụ thiết yếu giúp bạn giải quyết các bài toán dữ liệu phức tạp, đôi khi là cách trực tiếp và hiệu quả nhất để đạt được kết quả mong muốn trong những tình huống đặc thù.

Mục tiêu của phần này là trang bị cho bạn khả năng:
*   Hiểu rõ cơ chế hoạt động, ưu nhược điểm và các trường hợp sử dụng tối ưu của Correlated Subquery.
*   Nắm vững cách viết và triển khai Correlated Subquery trong cả mệnh đề `WHERE` và `SELECT`.
*   Khám phá khi nào và tại sao `SELECT` không có mệnh đề `FROM` lại hữu ích, cùng với các ứng dụng thực tế.
*   Củng cố kiến thức thông qua các ví dụ minh họa chi tiết, tuân thủ cú pháp PostgreSQL chuẩn.
*   Áp dụng tư duy Vibe Coding và tận dụng sức mạnh của Antigravity IDE để khám phá, tối ưu và gỡ lỗi các truy vấn phức tạp.

## I. Ôn Tập Subquery Cơ Bản: Nền Tảng của Truy Vấn Lồng Ghép

Trước khi khám phá Correlated Subquery, hãy cùng củng cố lại kiến thức về Subquery nói chung.

### 1. Subquery là gì?

Subquery (truy vấn con, truy vấn lồng) là một truy vấn `SELECT` hoàn chỉnh được nhúng bên trong một truy vấn SQL khác. Truy vấn chứa subquery được gọi là Outer Query (truy vấn ngoài), và truy vấn bên trong là Inner Query (truy vấn trong) hay Subquery.

Subquery có thể được đặt linh hoạt trong nhiều mệnh đề của Outer Query, bao gồm `SELECT`, `FROM` (dưới dạng Derived Table hoặc Common Table Expression), `WHERE`, `HAVING`, và thậm chí trong các câu lệnh thao tác dữ liệu như `INSERT`, `UPDATE`, `DELETE`.

### 2. Các Loại Subquery Phổ Biến

Dựa trên số lượng giá trị trả về, subquery được phân loại như sau:
*   **Scalar Subquery**: Trả về **một giá trị đơn lẻ** (một hàng, một cột). Đây là loại subquery phổ biến nhất trong các mệnh đề `SELECT` hoặc `WHERE` khi so sánh.
*   **Row Subquery**: Trả về **một hàng duy nhất** với nhiều cột. Thường được sử dụng trong mệnh đề `WHERE` với các toán tử `IN`, `EXISTS` hoặc so sánh từng cặp giá trị.
*   **Table Subquery**: Trả về **một bảng** (nhiều hàng, nhiều cột). Thường được sử dụng trong mệnh đề `FROM` như một bảng tạm thời.

### 3. Cơ Chế Hoạt Động của Subquery Thông Thường (Non-Correlated)

Với một subquery thông thường, truy vấn bên trong (inner query) được thực thi **một lần duy nhất** trước khi truy vấn bên ngoài (outer query) bắt đầu. Kết quả của inner query sau đó được lưu trữ và truyền cho outer query để xử lý. Điều này có nghĩa là inner query hoàn toàn độc lập với outer query.

**Ví dụ:** Tìm tất cả sản phẩm có giá cao hơn giá trung bình của tất cả sản phẩm.

```sql
SELECT
    name,
    price
FROM
    products
WHERE
    price > (SELECT AVG(price) FROM products); -- Subquery này chạy một lần duy nhất
```
Trong ví dụ trên, `(SELECT AVG(price) FROM products)` sẽ tính toán giá trung bình của tất cả sản phẩm *một lần*. Giả sử kết quả là `388.90`. Sau đó, truy vấn ngoài sẽ lọc tất cả sản phẩm có `price > 388.90`.

## II. Correlated Subquery: Sự Tương Quan Giữa Các Truy Vấn

Correlated Subquery đại diện cho một bước tiến quan trọng trong khả năng biểu diễn logic truy vấn phức tạp.

### 1. Định Nghĩa Correlated Subquery

Correlated Subquery là một loại subquery mà **truy vấn bên trong tham chiếu đến một cột hoặc một giá trị từ truy vấn bên ngoài**. Điều này tạo ra một "sự tương quan" hoặc "sự phụ thuộc" chặt chẽ giữa hai truy vấn: truy vấn bên trong không thể thực thi độc lập mà cần thông tin từ truy vấn bên ngoài.

### 2. Cơ Chế Hoạt Động: Mô Hình "Vòng Lặp Lồng Nhau"

Không giống như subquery thông thường, Correlated Subquery không chạy một lần duy nhất. Thay vào đó, nó sẽ được thực thi **một lần cho MỖI HÀNG** mà truy vấn bên ngoài đang xử lý. Bạn có thể hình dung quá trình này giống như một vòng lặp lồng nhau trong lập trình:

1.  Truy vấn bên ngoài (`Outer Query`) bắt đầu duyệt qua tập hợp các hàng của nó.
2.  Với **mỗi hàng** mà `Outer Query` đang xử lý (gọi là hàng hiện tại), `Correlated Subquery` được kích hoạt.
3.  `Correlated Subquery` sử dụng các giá trị từ hàng hiện tại của `Outer Query` để lọc, tính toán hoặc thực hiện các thao tác khác bên trong chính nó.
4.  Kết quả của `Correlated Subquery` (thường là một giá trị scalar) được trả về cho `Outer Query`.
5.  `Outer Query` sử dụng giá trị này để hoàn thành việc xử lý hàng hiện tại (ví dụ: so sánh trong mệnh đề `WHERE`, hiển thị trong mệnh đề `SELECT`, v.v.).
6.  Quá trình này lặp lại cho hàng tiếp theo của `Outer Query` cho đến khi tất cả các hàng đã được xử lý.

> [!NOTE]
> **Hiệu suất:** Do việc thực thi lặp đi lặp lại cho mỗi hàng của truy vấn bên ngoài, Correlated Subquery có thể kém hiệu quả hơn so với các phương pháp khác như `JOIN` hoặc `WINDOW FUNCTIONS` trên các tập dữ liệu lớn. Điều này đặc biệt đúng khi truy vấn con phức tạp hoặc bảng dữ liệu lớn. Tuy nhiên, trong một số tình huống, nó lại là giải pháp trực quan, dễ hiểu hoặc thậm chí là cách duy nhất để giải quyết một vấn đề cụ thể mà không làm phức tạp hóa truy vấn một cách không cần thiết.

### 3. Sử Dụng Bí Danh (Aliases) Trong Correlated Subquery

Để tham chiếu đến các cột từ truy vấn bên ngoài trong truy vấn bên trong, việc sử dụng bí danh (alias) là **bắt buộc** và cực kỳ quan trọng để tăng tính rõ ràng, tránh nhầm lẫn và giúp PostgreSQL hiểu rõ bạn đang muốn tham chiếu đến cột từ bảng nào.

**Ví dụ:** `products AS p1` (cho truy vấn ngoài) và `products AS p2` (cho truy vấn trong). Khi đó, `p1.department` sẽ tham chiếu đến cột `department` của hàng hiện tại từ truy vấn ngoài, trong khi `p2.department` sẽ tham chiếu đến cột `department` của hàng trong truy vấn con.

Để minh họa, chúng ta sẽ sử dụng một bảng `products` đơn giản:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    department VARCHAR(255) NOT NULL,
    price NUMERIC(10, 2) NOT NULL
);

INSERT INTO products (name, department, price) VALUES
('Industrial Widget A', 'Industrial', 876.00),
('Outdoor Tent Pro', 'Outdoor', 412.00),
('Grocery Item C', 'Grocery', 10.00),
('Industrial Gear B', 'Industrial', 328.00),
('Outdoor Backpack', 'Outdoor', 298.00),
('Grocery Milk', 'Grocery', 9.00),
('Industrial Robot Arm', 'Industrial', 796.00),
('Grocery Bread', 'Grocery', 8.00),
('Outdoor Boots', 'Outdoor', 450.00),
('Industrial Drill', 'Industrial', 915.00);
```

### 4. Ứng Dụng 1: Tìm Sản Phẩm Đắt Nhất của Mỗi Phòng Ban (Trong Mệnh Đề `WHERE`)

**Bài toán**: Hiển thị tên sản phẩm, tên phòng ban và giá của sản phẩm đắt nhất trong **mỗi** phòng ban.

**Phân tích vấn đề:**
Chúng ta muốn chọn các sản phẩm mà giá của nó bằng với giá cao nhất *trong chính phòng ban của sản phẩm đó*. Đây là một điều kiện phụ thuộc vào từng hàng, làm cho Correlated Subquery trở thành một giải pháp tự nhiên.

**Cách tiếp cận với Correlated Subquery:**
Đối với mỗi sản phẩm trong bảng `products` (truy vấn ngoài), chúng ta sẽ chạy một truy vấn con để tìm giá sản phẩm cao nhất TRONG CÙNG MỘT PHÒNG BAN với sản phẩm đó. Sau đó, chúng ta sẽ chỉ giữ lại những sản phẩm mà giá của nó bằng với giá cao nhất tìm được từ truy vấn con.

```sql
SELECT
    p1.name AS product_name,
    p1.department,
    p1.price
FROM
    products AS p1 -- p1 là bí danh cho truy vấn bên ngoài
WHERE
    p1.price = (
        SELECT
            MAX(p2.price)
        FROM
            products AS p2 -- p2 là bí danh cho truy vấn bên trong
        WHERE
            p2.department = p1.department -- ĐIỂM TƯƠNG QUAN: truy vấn con dùng department của hàng p1 hiện tại
    );
```

**Giải thích cơ chế thực thi chi tiết (Vibe Coding Perspective):**

Hãy hình dung Antigravity IDE đang thực thi truy vấn này. Nó sẽ làm việc như sau:

1.  **Antigravity (Outer Query):** Bắt đầu duyệt qua từng hàng của bảng `products` dưới bí danh `p1`.
2.  **Hàng 1 (`p1`):** `('Industrial Widget A', 'Industrial', 876.00, id=1)`
    *   **Antigravity (Inner Query):** Kích hoạt truy vấn con. Nó nhận `p1.department` là 'Industrial'.
    *   Thực thi: `SELECT MAX(p2.price) FROM products AS p2 WHERE p2.department = 'Industrial';`
    *   Kết quả inner query: `915.00` (giá của 'Industrial Drill').
    *   **Antigravity (Outer Query):** So sánh `p1.price` (`876.00`) với kết quả inner query (`915.00`). `876.00 = 915.00` là `FALSE`. Hàng này bị loại bỏ.
3.  **Hàng 2 (`p1`):** `('Outdoor Tent Pro', 'Outdoor', 412.00, id=2)`
    *   **Antigravity (Inner Query):** Kích hoạt lại truy vấn con. Nó nhận `p1.department` là 'Outdoor'.
    *   Thực thi: `SELECT MAX(p2.price) FROM products AS p2 WHERE p2.department = 'Outdoor';`
    *   Kết quả inner query: `450.00` (giá của 'Outdoor Boots').
    *   **Antigravity (Outer Query):** So sánh `p1.price` (`412.00`) với kết quả inner query (`450.00`). `412.00 = 450.00` là `FALSE`. Hàng này bị loại bỏ.
4.  ...
5.  **Hàng X (`p1`):** `('Industrial Drill', 'Industrial', 915.00, id=10)`
    *   **Antigravity (Inner Query):** Kích hoạt lại truy vấn con. Nó nhận `p1.department` là 'Industrial'.
    *   Thực thi: `SELECT MAX(p2.price) FROM products AS p2 WHERE p2.department = 'Industrial';`
    *   Kết quả inner query: `915.00`.
    *   **Antigravity (Outer Query):** So sánh `p1.price` (`915.00`) với kết quả inner query (`915.00`). `915.00 = 915.00` là `TRUE`. Hàng này được thêm vào tập kết quả.
6.  Quá trình này tiếp diễn cho tất cả các hàng còn lại.

**Kết quả mong đợi:**

| product_name       | department | price  |
| :----------------- | :--------- | :----- |
| Outdoor Boots      | Outdoor    | 450.00 |
| Industrial Drill   | Industrial | 915.00 |
| Grocery Item C     | Grocery    | 10.00  |

> [!TIP]
> **Vibe Coding với Antigravity:** Khi đối mặt với hiệu suất của correlated subquery trên dữ liệu lớn, bạn có thể yêu cầu Antigravity refactor truy vấn này sang sử dụng `WINDOW FUNCTIONS`. Ví dụ:
> ```sql
> SELECT
>     name AS product_name,
>     department,
>     price
> FROM (
>     SELECT
>         name,
>         department,
>         price,
>         MAX(price) OVER (PARTITION BY department) AS max_price_in_department
>     FROM
>         products
> ) AS sub
> WHERE
>     price = max_price_in_department;
> ```
> Sau đó, sử dụng tính năng `EXPLAIN ANALYZE` của Antigravity (hoặc chạy lệnh `EXPLAIN ANALYZE` trực tiếp) để so sánh kế hoạch thực thi và thời gian chạy của cả hai truy vấn. Điều này giúp bạn "vibe check" hiệu suất và chọn phương pháp tối ưu.

### 5. Ứng Dụng 2: Đếm Số Lượng Đơn Hàng cho Mỗi Sản Phẩm (Trong Mệnh Đề `SELECT`)

**Bài toán**: In ra tên sản phẩm và số lượng đơn hàng liên quan đến sản phẩm đó, mà không sử dụng `JOIN` hoặc `GROUP BY` trực tiếp cho truy vấn ngoài.

Để làm ví dụ này, chúng ta cần một bảng `orders` (đơn hàng):

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    product_id INT NOT NULL REFERENCES products(id),
    order_date DATE NOT NULL
);

INSERT INTO orders (product_id, order_date) VALUES
(1, '2023-01-01'), -- Industrial Widget A
(1, '2023-01-02'),
(2, '2023-01-03'), -- Outdoor Tent Pro
(3, '2023-01-04'), -- Grocery Item C
(2, '2023-01-05'),
(4, '2023-01-06'), -- Industrial Gear B
(4, '2023-01-07'),
(4, '2023-01-08');
```

**Cách tiếp cận với Correlated Subquery:**
Chúng ta sẽ lấy tên sản phẩm từ truy vấn ngoài. Đối với mỗi sản phẩm, chúng ta sẽ chạy một truy vấn con trong mệnh đề `SELECT` để đếm số lượng đơn hàng có `product_id` trùng với `id` của sản phẩm hiện tại.

```sql
SELECT
    p1.name AS product_name,
    (
        SELECT
            COUNT(o1.id) -- Đếm số lượng đơn hàng
        FROM
            orders AS o1 -- o1 là bí danh cho bảng orders trong truy vấn con
        WHERE
            o1.product_id = p1.id -- Tương quan: so sánh product_id của order với id của product hiện tại
    ) AS num_orders -- Đặt tên cho cột kết quả của truy vấn con
FROM
    products AS p1; -- p1 là bí danh cho bảng products trong truy vấn ngoài
```

**Giải thích cơ chế thực thi chi tiết:**

1.  **Outer Query:** Bắt đầu duyệt qua từng hàng trong bảng `products` (`p1`).
2.  Giả sử `p1` đang ở hàng đầu tiên: `('Industrial Widget A', ..., id=1)`.
3.  **Inner Query (trong SELECT):** Được thực thi.
    *   `p1.id` hiện tại là `1`.
    *   Truy vấn con đếm số lượng hàng trong bảng `orders` (`o1`) mà `o1.product_id` bằng `1`.
    *   Kết quả của truy vấn con là `2`.
4.  **Outer Query:** Hiển thị `product_name` là 'Industrial Widget A' và `num_orders` là `2`.
5.  Quá trình này lặp lại cho tất cả các sản phẩm khác, tính toán số đơn hàng tương ứng cho mỗi sản phẩm.

**Kết quả mong đợi:**

| product_name         | num_orders |
| :------------------- | :--------- |
| Industrial Widget A  | 2          |
| Outdoor Tent Pro     | 2          |
| Grocery Item C       | 1          |
| Industrial Gear B    | 3          |
| Outdoor Backpack     | 0          |
| Grocery Milk         | 0          |
| Industrial Robot Arm | 0          |
| Grocery Bread        | 0          |
| Outdoor Boots        | 0          |
| Industrial Drill     | 0          |

> [!TIP]
> **Gỡ lỗi và Vibe Coding với Antigravity:** Nếu bạn thấy kết quả không như mong đợi, Antigravity có thể giúp bạn gỡ lỗi. Đối với từng hàng của truy vấn ngoài, bạn có thể "tách" truy vấn con ra và chạy độc lập với giá trị `p1.id` cụ thể. Ví dụ, để kiểm tra sản phẩm có `id = 1`:
> ```sql
> SELECT COUNT(o1.id) FROM orders AS o1 WHERE o1.product_id = 1;
> ```
> Bằng cách này, bạn có thể xác nhận logic của truy vấn con trước khi nó được lồng vào toàn bộ truy vấn phức tạp. Đây là một phần của Vibe Coding – hiểu cách từng thành phần hoạt động riêng lẻ và cách chúng tương tác.
>
> Mặc dù Correlated Subquery giải quyết được bài toán này, trong thực tế, việc sử dụng `LEFT JOIN` và `GROUP BY` thường là cách tiếp cận hiệu quả và dễ đọc hơn cho việc tổng hợp:
> ```sql
> SELECT
>     p.name AS product_name,
>     COUNT(o.id) AS num_orders
> FROM
>     products AS p
> LEFT JOIN
>     orders AS o ON p.id = o.product_id
> GROUP BY
>     p.id, p.name -- Cần group by cả p.id nếu p.name không phải là khóa chính hoặc unique
> ORDER BY
>     p.id;
> ```
> Correlated Subquery đặc biệt hữu ích khi bạn cần một giá trị scalar duy nhất cho mỗi hàng từ một tập dữ liệu khác mà việc `JOIN` có thể phức tạp, tạo ra các hàng trùng lặp không mong muốn hoặc không phù hợp với ngữ cảnh logic.

## III. SELECT Không Có Mệnh Đề FROM: Sức Mạnh của Tính Toán Độc Lập

Trong PostgreSQL (và nhiều hệ quản trị CSDL khác), bạn có thể sử dụng câu lệnh `SELECT` mà không cần mệnh đề `FROM`. Điều này có vẻ lạ lúc đầu, nhưng nó cực kỳ hữu ích cho một số tác vụ cụ thể, biến `SELECT` thành một công cụ tính toán và kiểm tra linh hoạt.

### 1. Nguyên Tắc Cơ Bản và Cơ Chế Hoạt Động

Bạn có thể sử dụng `SELECT` mà không có `FROM` **nếu tất cả các biểu thức trong mệnh đề `SELECT` đều là các giá trị hằng số, các phép tính toán, các hàm không yêu cầu dữ liệu bảng (ví dụ: `NOW()`, `RANDOM()`), hoặc các truy vấn phụ (subquery) trả về một giá trị scalar duy nhất.**

Khi bạn bỏ qua mệnh đề `FROM`, PostgreSQL không cần truy cập bất kỳ bảng nào. Thay vào đó, nó hoạt động như một máy tính hoặc một công cụ đánh giá biểu thức, trả về kết quả của các biểu thức bạn đã cung cấp. Về mặt kỹ thuật, PostgreSQL sử dụng một bảng "dummy" nội bộ (đôi khi được gọi là `DUAL` ở Oracle, nhưng PostgreSQL không yêu cầu `FROM DUAL`) để thực hiện các phép tính này.

Nếu một subquery được sử dụng trong `SELECT` không có `FROM` mà trả về nhiều hơn một hàng hoặc nhiều hơn một cột, bạn sẽ gặp lỗi vì ngữ cảnh yêu cầu một giá trị scalar duy nhất.

### 2. Khi Nào và Tại Sao Cần `SELECT` Không Có `FROM`?

Mệnh đề `SELECT` không có `FROM` rất hữu ích khi bạn muốn:

*   **Tính toán một giá trị duy nhất:** Thực hiện các phép toán số học đơn giản, sử dụng hàm ngày tháng, chuỗi, hoặc kết quả của một hàm tổng hợp trên toàn bộ bảng.
*   **Kiểm tra các biểu thức:** Nhanh chóng xem kết quả của một biểu thức phức tạp, một hàm tùy chỉnh hoặc một đoạn mã SQL mà không cần truy vấn từ một bảng cụ thể.
*   **Kết hợp các giá trị tổng hợp độc lập:** So sánh hoặc tính toán dựa trên các giá trị tổng hợp từ các bảng khác nhau mà không cần `JOIN` chúng.
*   **Tạo ra các giá trị hằng số hoặc dữ liệu giả:** Để sử dụng trong các script, kiểm tra nhanh, hoặc làm dữ liệu đầu vào cho các hàm khác.
*   **Kiểm tra môi trường:** Lấy thông tin về phiên làm việc hiện tại, phiên bản PostgreSQL, v.v.

> [!NOTE]
> **Vibe Coding với Antigravity:** Trong Antigravity IDE, `SELECT` không có `FROM` trở thành công cụ "scratchpad" tối thượng của bạn. Bạn có thể nhanh chóng kiểm tra cú pháp, đánh giá biểu thức, hoặc thử nghiệm các hàm PostgreSQL mà không cần tạo bảng tạm hay chèn dữ liệu. Điều này hỗ trợ tư duy Vibe Coding bằng cách cho phép bạn khám phá và thử nghiệm các ý tưởng SQL một cách linh hoạt, tương tự như việc chạy một dòng lệnh Python hay JavaScript trong console.

### 3. Ví Dụ 1: Trả Về Một Giá Trị Tổng Hợp Duy Nhất

Bạn có thể dễ dàng tìm giá trị lớn nhất trong bảng `products` mà không cần mệnh đề `FROM` cho truy vấn bên ngoài, vì bản thân subquery đã cung cấp giá trị scalar:

```sql
SELECT (SELECT MAX(price) FROM products);
```

**Kết quả:** `915.00`

Nếu bạn cố gắng trả về nhiều hàng hoặc cột, bạn sẽ gặp lỗi:

```sql
-- Sẽ gây lỗi: Subquery trả về nhiều hơn một cột
-- ERROR:  subquery must return only one column
SELECT (SELECT name, price FROM products WHERE id = 1);

-- Sẽ gây lỗi: Subquery trả về nhiều hơn một hàng
-- ERROR:  more than one row returned by a subquery used as an expression
SELECT (SELECT name FROM products);
```

### 4. Ví Dụ 2: Tính Toán Tỷ Lệ Giữa Các Giá Trị Tổng Hợp

Giả sử bạn muốn tìm tỷ lệ giữa giá sản phẩm đắt nhất và giá sản phẩm rẻ nhất.

```sql
SELECT
    (SELECT MAX(price) FROM products) / (SELECT MIN(price) FROM products) AS price_ratio;
```

**Kết quả:**
`114.3750000000000000` (915.00 / 8.00)

Bạn cũng có thể tính toán các tỷ lệ khác, ví dụ, tỷ lệ giữa giá cao nhất và giá trung bình:

```sql
SELECT
    (SELECT MAX(price) FROM products) / (SELECT AVG(price) FROM products) AS max_to_avg_price_ratio;
```

**Kết quả:**
`2.3526973515042426` (915.00 / 388.90...)

### 5. Ví Dụ 3: Trả Về Nhiều Giá Trị Tổng Hợp Độc Lập

Bạn có thể kết hợp nhiều subquery trả về scalar trong một câu lệnh `SELECT` duy nhất. Mỗi subquery sẽ trở thành một cột riêng biệt trong kết quả, cho phép bạn hiển thị nhiều thông tin tổng hợp cùng lúc.

```sql
SELECT
    (SELECT MAX(price) FROM products) AS max_product_price,
    (SELECT MIN(price) FROM products) AS min_product_price,
    (SELECT AVG(price) FROM products) AS avg_product_price;
```

**Kết quả:**

| max_product_price | min_product_price | avg_product_price    |
| :---------------- | :---------------- | :------------------- |
| 915.00            | 8.00              | 388.9000000000000000 |

### 6. Bài Tập Thực Hành và Giải Pháp: Thống Kê Giá Điện Thoại

Để củng cố kiến thức, chúng ta hãy giải một bài tập nhỏ: tìm giá tối đa, tối thiểu và trung bình cho một bảng `phones` giả định.

**Yêu cầu:** Tạo một bảng `phones` và sau đó sử dụng `SELECT` không có `FROM` để hiển thị `max_price`, `min_price`, và `avg_price` cho tất cả điện thoại.

**Tạo bảng `phones`:**

```sql
CREATE TABLE phones (
    id SERIAL PRIMARY KEY,
    model VARCHAR(255) NOT NULL,
    price NUMERIC(10, 2) NOT NULL
);

INSERT INTO phones (model, price) VALUES
('iPhone 15 Pro Max', 1200.00),
('Samsung Galaxy S24 Ultra', 1150.00),
('Google Pixel 8 Pro', 999.00),
('OnePlus 12', 799.00),
('Xiaomi 14 Ultra', 1050.00),
('Nokia 3310', 50.00);
```

**Giải pháp:**
Sử dụng ba truy vấn phụ scalar, mỗi truy vấn tính một giá trị tổng hợp (MAX, MIN, AVG) và gán bí danh cho từng cột kết quả.

```sql
SELECT
    (SELECT MAX(price) FROM phones) AS max_phone_price,
    (SELECT MIN(price) FROM phones) AS min_phone_price,
    (SELECT AVG(price) FROM phones) AS avg_phone_price;
```

**Kết quả:**

| max_phone_price | min_phone_price | avg_phone_price      |
| :-------------- | :-------------- | :------------------- |
| 1200.00         | 50.00           | 874.6666666666666667 |

## IV. Tóm Tắt Phần và Lời Khuyên Vibe Coding

Chương này đã trang bị cho bạn những công cụ mạnh mẽ để xử lý các truy vấn phức tạp và thực hiện các phép tính độc lập trong PostgreSQL:

*   **Correlated Subquery** là một truy vấn phụ mà truy vấn bên trong tham chiếu đến một giá trị từ truy vấn bên ngoài. Nó hoạt động theo mô hình "vòng lặp lồng nhau", thực thi **một lần cho mỗi hàng** được xử lý bởi truy vấn bên ngoài.
*   Việc sử dụng **bí danh** (`AS p1`, `AS p2`) là rất quan trọng để phân biệt và tham chiếu đúng các cột.
*   Correlated Subquery có thể được sử dụng trong mệnh đề `WHERE` để lọc các hàng dựa trên điều kiện phụ thuộc vào chính hàng đó, hoặc trong mệnh đề `SELECT` để tính toán một giá trị scalar cho mỗi hàng.
*   **`SELECT` không có `FROM`** là một cú pháp hợp lệ trong PostgreSQL khi tất cả các biểu thức hoặc subquery trong mệnh đề `SELECT` trả về một giá trị scalar duy nhất. Kỹ thuật này hữu ích cho việc tính toán nhanh các giá trị hằng số, tỷ lệ, hoặc kết hợp các hàm tổng hợp mà không cần cấu trúc bảng cho truy vấn ngoài.
*   Mặc dù mạnh mẽ, Correlated Subquery có thể có **vấn đề về hiệu suất** trên các tập dữ liệu lớn so với `JOIN` hoặc `WINDOW FUNCTIONS`. Hãy luôn cân nhắc và sử dụng `EXPLAIN ANALYZE` để đánh giá hiệu quả.

**Lời khuyên Vibe Coding với Antigravity IDE:**

*   **Khám phá Cơ chế:** Khi bạn viết một correlated subquery, hãy thử "chạy thử" truy vấn con với các giá trị đầu vào khác nhau từ truy vấn ngoài trong Antigravity. Điều này giúp bạn xây dựng "vibe" về cách dữ liệu chảy qua từng phần của truy vấn.
*   **Tối ưu hóa Thông minh:** Đừng chỉ viết một truy vấn và dừng lại. Với Antigravity, bạn có thể dễ dàng yêu cầu nó "refactor" một correlated subquery sang `JOIN` hoặc `WINDOW FUNCTIONS`. Sau đó, sử dụng `EXPLAIN ANALYZE` để so sánh hiệu suất. Antigravity giúp bạn không chỉ viết code mà còn *hiểu* và *tối ưu* code một cách chủ động.
*   **"Scratchpad" Linh hoạt:** `SELECT` không có `FROM` là công cụ lý tưởng cho các phép thử nhanh trong Antigravity. Cần kiểm tra một hàm PostgreSQL? Tính toán một biểu thức phức tạp? Hay chỉ đơn giản là xem ngày giờ hiện tại? Hãy dùng nó như một bảng trắng kỹ thuật số để thử nghiệm ý tưởng SQL của bạn.

Phần này đã mở ra cánh cửa cho việc giải quyết các bài toán dữ liệu phức tạp hơn. Hãy luyện tập và tận dụng Antigravity IDE để làm chủ các kỹ thuật này, biến việc lập trình CSDL thành một hành trình khám phá và tối ưu hóa liên tục!

<!-- REVIEWED_BY_AGENT -->
