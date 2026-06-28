# Phần 15: Giới Thiệu Subquery và Subquery trong Mệnh đề SELECT

Trong thế giới của cơ sở dữ liệu quan hệ, việc truy vấn và trích xuất thông tin không phải lúc nào cũng là một nhiệm vụ đơn giản chỉ với một câu lệnh SQL duy nhất. Đôi khi, để giải quyết một yêu cầu phức tạp, chúng ta cần phải thực hiện nhiều bước logic: tìm một giá trị trước, sau đó sử dụng giá trị đó để lọc hoặc tính toán trong một truy vấn khác. Đây chính là lúc khái niệm **Subquery (Truy vấn con)** thể hiện sức mạnh vượt trội của nó.

Phần này sẽ đi sâu vào Subquery, giải thích bản chất, cách thức hoạt động, và đặc biệt tập trung vào việc sử dụng Subquery trong mệnh đề `SELECT` của PostgreSQL. Chúng ta sẽ khám phá lý do tại sao Subquery lại cần thiết, cách chúng giúp đơn giản hóa các truy vấn phức tạp, và những quy tắc quan trọng về cấu trúc dữ liệu trả về mà bạn cần nắm vững để viết các truy vấn hiệu quả và không lỗi.

## 1. Subquery là gì và tại sao chúng ta cần chúng?

**Subquery**, hay còn gọi là **truy vấn con** hoặc **truy vấn lồng**, là một câu lệnh SQL hoàn chỉnh được lồng ghép bên trong một câu lệnh SQL khác. Nó cho phép bạn thực hiện các phép tính, lọc dữ liệu, hoặc tạo ra các giá trị dựa trên kết quả của một truy vấn độc lập khác, tất cả trong cùng một câu lệnh SQL duy nhất.

Mục đích chính của Subquery là phá vỡ một bài toán phức tạp thành các phần nhỏ hơn, dễ quản lý hơn, sau đó kết hợp các phần đó lại để đạt được kết quả cuối cùng.

### 1.1. Minh họa về sự cần thiết của Subquery

Hãy xem xét một yêu cầu thực tế:

**Yêu cầu:** Liệt kê tên và giá của tất cả các sản phẩm có giá đắt hơn TẤT CẢ các sản phẩm trong bộ phận 'đồ chơi'.

Nếu chỉ nhìn vào dữ liệu mẫu nhỏ, bạn có thể dễ dàng nhận thấy sản phẩm đắt nhất trong bộ phận 'đồ chơi' có giá là 876. Khi đó, bạn có thể viết truy vấn như sau:

```sql
SELECT name, price
FROM products
WHERE price > 876;
```

> [!NOTE]
> Cách tiếp cận này, với việc gán cứng giá trị `876`, là không thực tế và nguy hiểm trong môi trường dữ liệu động. Giá trị này có thể thay đổi bất cứ lúc nào khi dữ liệu được cập nhật. Nếu có hàng nghìn, hàng triệu sản phẩm, việc kiểm tra thủ công là bất khả thi. Chúng ta cần một giải pháp tự động.

Trong thế giới thực, chúng ta cần một cách tự động để tìm ra giá sản phẩm đắt nhất trong bộ phận 'đồ chơi'. Bài toán này có thể được chia thành hai bước logic:

1.  **Bước 1: Tìm giá cao nhất của sản phẩm trong bộ phận 'đồ chơi'.**
    ```sql
    SELECT MAX(price)
    FROM products
    WHERE department = 'toys';
    ```
    Truy vấn này sẽ trả về một giá trị duy nhất, ví dụ `876`.

2.  **Bước 2: Sử dụng giá trị này để tìm tất cả các sản phẩm có giá cao hơn.**
    ```sql
    SELECT name, price
    FROM products
    WHERE price > 876; -- Thay 876 bằng kết quả từ Bước 1
    ```

Thay vì chạy hai truy vấn riêng biệt, Subquery cho phép chúng ta kết hợp chúng thành một câu lệnh liền mạch:

```sql
SELECT name, price
FROM products
WHERE price > (
    SELECT MAX(price)
    FROM products
    WHERE department = 'toys'
);
```

### 1.2. Cơ chế hoạt động (Under the Hood)

Khi PostgreSQL gặp một câu lệnh SQL chứa Subquery, nó sẽ thực hiện theo một trình tự logic (đối với các Subquery không tương quan - non-correlated subqueries):

1.  **Thực thi truy vấn con (Subquery):** Truy vấn bên trong `(SELECT MAX(price) FROM products WHERE department = 'toys')` sẽ được thực thi trước tiên.
2.  **Trả về kết quả:** Kết quả của truy vấn con (ở đây là một giá trị số `876`) sẽ được trả về cho truy vấn bên ngoài.
3.  **Thực thi truy vấn chính:** Truy vấn bên ngoài `SELECT name, price FROM products WHERE price > 876` sẽ được thực thi, sử dụng `876` làm điều kiện lọc.

Subquery là một công cụ cực kỳ linh hoạt và có thể được sử dụng ở nhiều vị trí khác nhau trong một câu lệnh SQL, bao gồm các mệnh đề `SELECT`, `FROM`, `WHERE`, `JOIN` và `HAVING`. Tuy nhiên, để sử dụng chúng một cách hiệu quả, bạn cần hiểu rõ về "hình dạng" hoặc "cấu trúc" của dữ liệu mà chúng trả về.

## 2. Các Dạng Subquery theo Cấu trúc Dữ liệu Trả về

Một trong những khía cạnh quan trọng nhất khi làm việc với Subquery là hiểu về cấu trúc dữ liệu mà truy vấn con trả về. Tùy thuộc vào vị trí mà Subquery được đặt trong truy vấn chính, nó sẽ yêu cầu một cấu trúc dữ liệu trả về cụ thể. Nếu Subquery trả về một cấu trúc dữ liệu không phù hợp với vị trí của nó, PostgreSQL sẽ báo lỗi.

Hãy xem xét các dạng cấu trúc dữ liệu cơ bản mà một truy vấn có thể trả về:

### 2.1. Scalar Subquery (Truy vấn con vô hướng)

**Định nghĩa:** Một Scalar Subquery là truy vấn con trả về **chỉ một hàng và chỉ một cột** (tức là một giá trị đơn lẻ).

**Ứng dụng:** Đây là dạng bắt buộc khi sử dụng Subquery trong mệnh đề `SELECT` hoặc với các toán tử so sánh đơn lẻ (`=`, `>`, `<`, `<=`, `>=`, `<>`) trong mệnh đề `WHERE` hoặc `HAVING`.

**Ví dụ:**
```sql
SELECT COUNT(*) FROM orders;          -- Trả về tổng số đơn hàng
SELECT MAX(price) FROM products;       -- Trả về giá cao nhất
SELECT AVG(salary) FROM employees;     -- Trả về mức lương trung bình
SELECT price FROM products WHERE id = 3; -- Trả về giá của sản phẩm có ID là 3 (giả định ID là duy nhất)
```
**Cấu trúc trả về:** Một giá trị đơn lẻ (ví dụ: `123`, `989.99`, `55000.00`).

### 2.2. Column Subquery (Truy vấn con cột)

**Định nghĩa:** Một Column Subquery là truy vấn con trả về **một cột dữ liệu với nhiều hàng**.

**Ứng dụng:** Dạng này thường được sử dụng với các toán tử như `IN`, `NOT IN`, `ANY`, `ALL`, `EXISTS`, `NOT EXISTS` trong mệnh đề `WHERE` hoặc `HAVING`.

**Ví dụ:**
```sql
SELECT id FROM products WHERE department = 'electronics'; -- Trả về danh sách các ID sản phẩm
SELECT DISTINCT department FROM products;                 -- Trả về danh sách các bộ phận duy nhất
```
**Cấu trúc trả về:** Một cột dữ liệu với nhiều hàng (ví dụ: `(101, 105, 210)`).

### 2.3. Row Subquery (Truy vấn con hàng)

**Định nghĩa:** Một Row Subquery là truy vấn con trả về **một hàng dữ liệu với nhiều cột**.

**Ứng dụng:** Dạng này thường được sử dụng khi bạn cần so sánh nhiều cột cùng lúc trong mệnh đề `WHERE`.

**Ví dụ:**
```sql
SELECT name, price FROM products WHERE id = 1; -- Trả về tên và giá của sản phẩm có ID là 1
```
**Cấu trúc trả về:** Một hàng với nhiều cột (ví dụ: `('Laptop Pro', 1200.00)`). Thường được dùng trong `WHERE (col1, col2) = (SELECT val1, val2 FROM ...)`

### 2.4. Table Subquery (Truy vấn con bảng)

**Định nghĩa:** Một Table Subquery là truy vấn con trả về **nhiều hàng và nhiều cột** (giống như một bảng hoàn chỉnh).

**Ứng dụng:** Dạng này thường được sử dụng trong mệnh đề `FROM`, nơi nó hoạt động như một bảng tạm thời hoặc một tập hợp kết quả ảo. Nó còn được gọi là "derived table" (bảng dẫn xuất).

**Ví dụ:**
```sql
SELECT id, name, price FROM products WHERE price > 500; -- Trả về một tập hợp các sản phẩm
```
**Cấu trúc trả về:** Một tập hợp các hàng, mỗi hàng có nhiều cột (giống như một bảng hoàn chỉnh).

> [!IMPORTANT]
> Việc hiểu rõ cấu trúc dữ liệu trả về là chìa khóa để sử dụng Subquery chính xác. Mỗi vị trí trong câu lệnh SQL (như `SELECT`, `FROM`, `WHERE`) có những yêu cầu riêng về loại dữ liệu mà Subquery lồng vào phải cung cấp. Nếu Subquery trả về một cấu trúc dữ liệu không phù hợp với vị trí của nó, PostgreSQL sẽ báo lỗi.

## 3. Subquery trong Mệnh đề SELECT

Khi một Subquery được đặt trong mệnh đề `SELECT`, nó có nhiệm vụ cung cấp một giá trị cho một cột mới trong tập kết quả của truy vấn chính. Do đó, quy tắc cốt lõi là:

> [!RULE]
> Một Subquery trong mệnh đề `SELECT` **bắt buộc phải trả về một giá trị đơn (scalar value)**. Nếu nó trả về nhiều hàng hoặc nhiều cột, truy vấn sẽ thất bại với lỗi.

### 3.1. Scalar Subquery không tương quan (Non-correlated Scalar Subquery)

Đây là dạng Subquery mà truy vấn con có thể được thực thi hoàn toàn độc lập với truy vấn chính. Nó không tham chiếu bất kỳ cột nào từ truy vấn bên ngoài.

**Cơ chế hoạt động:** PostgreSQL sẽ thực thi Subquery này **một lần duy nhất** trước khi thực thi truy vấn chính. Kết quả của nó sẽ được sử dụng cho mọi hàng trong tập kết quả của truy vấn chính.

**Ví dụ 1: Hiển thị giá tối đa toàn cục**
Bạn muốn hiển thị tên, giá của từng sản phẩm, và thêm một cột nữa là giá tối đa của TẤT CẢ các sản phẩm trong bảng `products`.

```sql
SELECT
    name,
    price,
    (SELECT MAX(price) FROM products) AS max_overall_price -- Subquery trả về một giá trị đơn
FROM
    products;
```

**Giải thích:**
*   Subquery `(SELECT MAX(price) FROM products)` sẽ được thực thi một lần duy nhất và trả về một giá trị số duy nhất (ví dụ: `989.99`).
*   Giá trị này sau đó được thêm vào dưới dạng một cột mới (`max_overall_price`) cho mỗi hàng trong tập kết quả của truy vấn chính.
*   Vì Subquery này luôn trả về một giá trị duy nhất, nó hoàn toàn hợp lệ trong mệnh đề `SELECT`.

### 3.2. Scalar Subquery tương quan (Correlated Scalar Subquery)

Ngược lại với Subquery không tương quan, một **Correlated Subquery** là truy vấn con mà việc thực thi của nó **phụ thuộc vào mỗi hàng của truy vấn bên ngoài**. Nó thường chứa một điều kiện trong mệnh đề `WHERE` của nó mà tham chiếu đến một cột từ truy vấn chính.

**Cơ chế hoạt động:** PostgreSQL sẽ thực thi Correlated Subquery **một lần cho MỖI HÀNG** mà truy vấn chính đang xử lý. Điều này có thể ảnh hưởng đáng kể đến hiệu suất nếu bảng chính có nhiều hàng.

**Ví dụ: Hiển thị giá sản phẩm và giá trung bình của bộ phận đó**
Bạn muốn hiển thị tên, giá của từng sản phẩm, và thêm một cột nữa là giá trung bình của các sản phẩm TRONG CÙNG BỘ PHẬN với sản phẩm hiện tại.

```sql
SELECT
    p1.name,
    p1.price,
    (SELECT AVG(p2.price) FROM products p2 WHERE p2.department = p1.department) AS avg_department_price
FROM
    products p1;
```

**Giải thích:**
*   Truy vấn chính là `SELECT p1.name, p1.price FROM products p1`.
*   Subquery `(SELECT AVG(p2.price) FROM products p2 WHERE p2.department = p1.department)` là một Correlated Subquery vì nó tham chiếu đến `p1.department` từ truy vấn bên ngoài.
*   Đối với mỗi hàng sản phẩm `p1` trong truy vấn chính, Subquery sẽ được thực thi lại. Nó sẽ tính giá trung bình của tất cả các sản phẩm `p2` có cùng `department` với sản phẩm `p1` hiện tại.
*   Kết quả là một giá trị đơn (giá trung bình) cho mỗi hàng, và được thêm vào cột `avg_department_price`.

### 3.3. Xử lý trường hợp không có dữ liệu hoặc nhiều dữ liệu

Điều quan trọng cần ghi nhớ là Subquery trong `SELECT` *phải* trả về một giá trị đơn.

*   **Nếu Subquery trả về 0 hàng:**
    ```sql
    SELECT
        name,
        price,
        (SELECT price FROM products WHERE id = 999) AS price_of_nonexistent_product
    FROM
        products;
    ```
    Trong trường hợp này, `price_of_nonexistent_product` sẽ hiển thị giá trị `NULL` cho tất cả các hàng. Đây không phải là lỗi mà là hành vi dự kiến.

*   **Nếu Subquery trả về nhiều hơn 1 hàng:**
    ```sql
    -- Giả định có nhiều sản phẩm có department = 'toys'
    SELECT
        name,
        price,
        (SELECT price FROM products WHERE department = 'toys') AS toy_prices -- LỖI!
    FROM
        products;
    ```
    PostgreSQL sẽ báo lỗi tương tự như: `ERROR: more than one row returned by a subquery used as an expression`. Đây là một lỗi phổ biến khi vi phạm quy tắc "scalar subquery" trong `SELECT`.

### 3.4. Đổi tên cột kết quả (Aliasing)

Khi sử dụng Subquery trong mệnh đề `SELECT`, việc sử dụng `AS` để đổi tên (aliasing) cột kết quả là một thói quen tốt. Điều này giúp:

*   **Tăng cường khả năng đọc:** Tên cột `max_overall_price` hoặc `avg_department_price` ý nghĩa hơn nhiều so với tên cột mặc định có thể là `?column?`.
*   **Tránh xung đột tên cột:** Nếu Subquery trả về một cột có tên trùng với một cột hiện có trong bảng chính, việc đổi tên sẽ ngăn chặn xung đột và giúp bạn tham chiếu chính xác đến cột mong muốn.

```sql
SELECT
    name,
    price,
    (SELECT price FROM products WHERE id = 3) AS product_3_price -- Đổi tên cột
FROM
    products;
```

## 4. Ví dụ Thực tế và Bài tập

Để củng cố kiến thức, chúng ta sẽ bắt đầu với việc chuẩn bị dữ liệu và sau đó thực hiện một bài tập.

### 4.1. Chuẩn bị Dữ liệu

Hãy tạo hai bảng mẫu `products` và `phones` với dữ liệu giả định để bạn có thể thực hành các truy vấn.

```sql
-- Bảng products
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50) NOT NULL,
    price NUMERIC(10, 2) NOT NULL
);

INSERT INTO products (name, department, price) VALUES
('Toy Car', 'toys', 15.50),
('Robot Action Figure', 'toys', 35.00),
('Board Game', 'toys', 25.75),
('Laptop Pro', 'electronics', 1200.00),
('Smartphone X', 'electronics', 850.00),
('Wireless Earbuds', 'electronics', 150.00),
('Kitchen Mixer', 'home goods', 250.00),
('Coffee Maker', 'home goods', 75.00),
('Designer Watch', 'accessories', 400.00),
('Leather Wallet', 'accessories', 60.00);

-- Bảng phones (dùng cho bài tập)
CREATE TABLE phones (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2) NOT NULL
);

INSERT INTO phones (name, price) VALUES
('iPhone 13', 999.00),
('Samsung S22', 899.00),
('Google Pixel 6', 699.00),
('OnePlus 10 Pro', 799.00),
('Xiaomi 12', 399.00);
```

### 4.2. Ví dụ: So sánh giá sản phẩm với giá trung bình toàn cục

Bạn muốn thấy mỗi sản phẩm đắt hơn hay rẻ hơn bao nhiêu so với giá trung bình của tất cả các sản phẩm.

```sql
SELECT
    name,
    price,
    (SELECT AVG(price) FROM products) AS global_average_price,
    price - (SELECT AVG(price) FROM products) AS price_difference_from_avg
FROM
    products;
```

**Giải thích:**
*   Subquery `(SELECT AVG(price) FROM products)` tính toán giá trị trung bình của tất cả các sản phẩm. Đây là một Scalar Subquery không tương quan, được thực thi một lần.
*   Kết quả này được sử dụng hai lần: một lần để hiển thị trong cột `global_average_price`, và một lần để tính toán `price_difference_from_avg`.

### 4.3. Bài tập Thực hành: Tính tỷ lệ giá điện thoại

**Yêu cầu:** Từ bảng `phones`, hãy in ra tên và giá của từng chiếc điện thoại. Ngoài ra, hãy thêm một cột thứ ba hiển thị **tỷ lệ giá** của chiếc điện thoại hiện tại so với giá tối đa của TẤT CẢ các điện thoại trong bảng. Cột mới này nên được đặt tên là `price_ratio`.

**Bảng `phones` mẫu (đã tạo ở trên):**

| id | name           | price  |
|----|----------------|--------|
| 1  | iPhone 13      | 999.00 |
| 2  | Samsung S22    | 899.00 |
| 3  | Google Pixel 6 | 699.00 |
| 4  | OnePlus 10 Pro | 799.00 |
| 5  | Xiaomi 12      | 399.00 |

**Mục tiêu đầu ra:**

| name           | price  | price_ratio |
|----------------|--------|-------------|
| iPhone 13      | 999.00 | 1.000000000 |
| Samsung S22    | 899.00 | 0.900000000 |
| Google Pixel 6 | 699.00 | 0.700000000 |
| OnePlus 10 Pro | 799.00 | 0.800000000 |
| Xiaomi 12      | 399.00 | 0.400000000 |

> [!TIP]
> Hãy nhớ rằng bạn không nên gán cứng giá tối đa (ví dụ: `999.00`). Thay vào đó, hãy sử dụng một Subquery để tự động tìm giá tối đa.
>
> **Lưu ý về phép chia trong PostgreSQL:** Nếu cột `price` là kiểu số nguyên (INTEGER), phép chia giữa hai số nguyên sẽ trả về phần nguyên (integer division). Để đảm bảo kết quả là số thập phân, bạn cần ép kiểu một trong các toán hạng thành kiểu số thực (ví dụ: `price * 1.0` hoặc `price::NUMERIC`).

## 5. Giải pháp Bài tập

Dưới đây là giải pháp cho bài tập trên, sử dụng Subquery trong mệnh đề `SELECT` để tính toán tỷ lệ giá:

```sql
SELECT
    name,
    price,
    -- Tính tỷ lệ giá: giá hiện tại chia cho giá tối đa toàn cục
    -- Ép kiểu price thành NUMERIC để đảm bảo phép chia trả về giá trị thập phân chính xác
    (price::NUMERIC / (SELECT MAX(price) FROM phones)) AS price_ratio
FROM
    phones;
```

**Giải thích chi tiết:**
*   `SELECT name, price`: Chọn các cột `name` và `price` từ bảng `phones`.
*   `(SELECT MAX(price) FROM phones)`: Đây là Subquery. Nó được thực thi một lần duy nhất để tìm giá trị tối đa trong cột `price` của bảng `phones`. Kết quả là một giá trị đơn (scalar value), ví dụ `999.00`.
*   `price::NUMERIC`: Toán tử `::` là một cú pháp tắt (shorthand) của PostgreSQL để ép kiểu dữ liệu. Nó chuyển đổi giá trị `price` của từng hàng hiện tại thành kiểu `NUMERIC`. Việc này là cần thiết vì nếu `price` là `INTEGER` (hoặc `NUMERIC` với scale 0) và `MAX(price)` cũng là `INTEGER` (hoặc `NUMERIC` với scale 0), phép chia có thể dẫn đến kết quả số nguyên bị cắt cụt. Ép kiểu thành `NUMERIC` đảm bảo kết quả là số thập phân chính xác. Một cách khác là `CAST(price AS NUMERIC)`.
*   `/ (SELECT MAX(price) FROM phones)`: Giá trị `price` đã được ép kiểu sẽ được chia cho giá trị tối đa tìm được từ Subquery.
*   `AS price_ratio`: Kết quả của phép tính tỷ lệ được đặt tên là `price_ratio`, tạo thành một cột mới trong tập kết quả.

## Tóm tắt Phần 15

*   **Subquery (Truy vấn con)** là một câu lệnh SQL được lồng vào bên trong một câu lệnh SQL khác, giúp giải quyết các yêu cầu dữ liệu phức tạp bằng cách kết hợp nhiều bước logic trong một truy vấn duy nhất.
*   Subquery được thực thi trước (hoặc lặp lại cho mỗi hàng của truy vấn ngoài đối với correlated subquery) và kết quả của nó được sử dụng làm đầu vào cho truy vấn bên ngoài.
*   Việc hiểu **cấu trúc dữ liệu trả về** của Subquery là rất quan trọng để tránh lỗi:
    *   **Scalar Subquery:** Trả về một giá trị đơn (một hàng, một cột).
    *   **Column Subquery:** Trả về một cột với nhiều hàng (dùng với `IN`, `ANY`, `ALL`).
    *   **Row Subquery:** Trả về một hàng với nhiều cột (dùng với `WHERE (col1, col2) = (...)`).
    *   **Table Subquery:** Trả về nhiều hàng và nhiều cột (dùng trong mệnh đề `FROM` như một bảng ảo).
*   Khi sử dụng Subquery trong mệnh đề `SELECT`, **bắt buộc nó phải là một Scalar Subquery** (trả về một giá trị đơn). Nếu không, PostgreSQL sẽ báo lỗi `more than one row returned by a subquery used as an expression`.
*   **Scalar Subquery không tương quan** được thực thi một lần duy nhất.
*   **Scalar Subquery tương quan** được thực thi một lần cho mỗi hàng của truy vấn bên ngoài, tham chiếu đến dữ liệu của hàng đó.
*   Sử dụng từ khóa `AS` để **đổi tên (aliasing)** các cột được tạo ra bởi Subquery trong mệnh đề `SELECT` giúp tăng tính dễ đọc và tránh xung đột tên cột.
*   Trong PostgreSQL, khi thực hiện phép chia, hãy chú ý đến kiểu dữ liệu. Để nhận được kết quả thập phân chính xác từ phép chia số nguyên, hãy ép kiểu một trong các toán hạng thành số thực (ví dụ: `value * 1.0` hoặc `value::NUMERIC`).

<!-- REVIEWED_BY_AGENT -->
