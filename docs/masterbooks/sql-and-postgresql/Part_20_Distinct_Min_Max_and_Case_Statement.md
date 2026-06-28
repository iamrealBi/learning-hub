# Bài 20: DISTINCT, GREATEST/LEAST và Mệnh Đề CASE

Chào mừng bạn đến với Chương 20 của khóa học chuyên sâu về PostgreSQL. Trong chương này, chúng ta sẽ đi sâu vào ba công cụ SQL mạnh mẽ và không thể thiếu: từ khóa `DISTINCT` để trích xuất các giá trị duy nhất, các hàm `GREATEST()` và `LEAST()` để xác định giá trị cực trị, và mệnh đề `CASE` để áp dụng logic điều kiện phức tạp.

Những công cụ này là nền tảng cho việc truy vấn và phân tích dữ liệu hiệu quả, cho phép bạn không chỉ lọc và tóm tắt thông tin mà còn chủ động định hình kết quả truy vấn dựa trên các quy tắc kinh doanh cụ thể. Chúng ta sẽ tập trung hoàn toàn vào cú pháp chuẩn và các tính năng đặc thù của PostgreSQL, đảm bảo bạn có thể áp dụng chúng một cách chính xác trong môi trường cơ sở dữ liệu của mình.

Mục tiêu của chương này là trang bị cho bạn khả năng:
*   Trích xuất các giá trị hoặc tập hợp giá trị duy nhất từ các cột.
*   Xác định giá trị lớn nhất hoặc nhỏ nhất từ một danh sách các biểu thức.
*   Thực thi logic điều kiện `IF-THEN-ELSE` trực tiếp trong các truy vấn SQL để tạo ra các kết quả động.

Hãy cùng bắt đầu hành trình khám phá những kỹ thuật SQL quan trọng này, mở rộng khả năng thao tác dữ liệu của bạn lên một tầm cao mới.

---

## 1. Lọc Giá Trị Duy Nhất và Các Bộ Giá Trị với `DISTINCT`

Trong quản lý dữ liệu, việc gặp phải các bản ghi trùng lặp là điều thường thấy. Từ khóa `DISTINCT` là công cụ chính của SQL để loại bỏ những sự trùng lặp này, chỉ giữ lại các giá trị hoặc tổ hợp giá trị riêng biệt trong tập kết quả của bạn.

### 1.1. Khái niệm và Cú pháp cơ bản của `DISTINCT`

Từ khóa `DISTINCT` được đặt ngay sau `SELECT` và áp dụng cho tất cả các cột được liệt kê trong mệnh đề `SELECT`. Nó hoạt động bằng cách xem xét toàn bộ hàng (hoặc các cột được chỉ định) và chỉ trả về những hàng mà tổ hợp giá trị của chúng là duy nhất.

**Cú pháp cơ bản:**

```sql
SELECT DISTINCT column_name
FROM table_name;
```

Để minh họa, chúng ta sẽ sử dụng bảng `products` sau:

```sql
-- Tạo bảng ví dụ
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    weight DECIMAL(10, 2) NOT NULL,
    manufacturer VARCHAR(100)
);

-- Chèn dữ liệu ví dụ
INSERT INTO products (name, department, price, weight, manufacturer) VALUES
('Toy Car', 'Toys', 25.00, 0.5, 'KidCo'),
('Outdoor Tent', 'Outdoors', 150.00, 3.0, 'AdventureGear'),
('Industrial Drill', 'Industrial', 300.00, 15.0, 'ProTools'),
('Toy Robot', 'Toys', 50.00, 1.0, 'RoboFun'),
('Camping Chair', 'Outdoors', 40.00, 2.0, 'AdventureGear'),
('Industrial Saw', 'Industrial', 450.00, 20.0, 'ProTools'),
('Toy Blocks', 'Toys', 30.00, 0.8, 'KidCo'),
('Luxury Yacht', 'Industrial', 1000.00, 5000.00, 'OceanCraft'),
('Toy Car', 'Toys', 28.00, 0.6, 'KidCo'); -- Thêm một sản phẩm trùng tên nhưng giá khác
```

Để liệt kê tất cả các phòng ban duy nhất có trong bảng `products`:

```sql
SELECT DISTINCT department
FROM products;
```

**Kết quả:**

```
 department 
------------
 Outdoors
 Industrial
 Toys
(3 rows)
```

`DISTINCT` đã loại bỏ các giá trị trùng lặp ('Toys', 'Outdoors', 'Industrial') và chỉ hiển thị mỗi giá trị một lần.

#### Cơ chế hoạt động của `DISTINCT` (Under the Hood)

Khi bạn sử dụng `DISTINCT`, PostgreSQL cần phải xác định các hàng hoặc giá trị duy nhất. Điều này thường được thực hiện thông qua một trong hai cơ chế chính:

1.  **Sắp xếp (Sorting)**: Cơ sở dữ liệu sẽ sắp xếp các hàng theo các cột được chỉ định trong `SELECT DISTINCT`. Sau khi sắp xếp, các hàng trùng lặp sẽ nằm cạnh nhau, giúp hệ thống dễ dàng loại bỏ chúng. Việc sắp xếp có thể tốn kém về hiệu suất, đặc biệt với tập dữ liệu lớn.
2.  **Hashing**: Một cách khác là sử dụng bảng băm (hash table). Các giá trị từ các hàng được băm và lưu trữ. Nếu một giá trị băm đã tồn tại, hàng đó được coi là trùng lặp và bị loại bỏ. Phương pháp này thường nhanh hơn sắp xếp cho các tập dữ liệu rất lớn.

Tối ưu hóa truy vấn (Query Optimizer) của PostgreSQL sẽ chọn phương pháp hiệu quả nhất dựa trên kích thước dữ liệu, chỉ mục và tài nguyên hệ thống hiện có.

### 1.2. `DISTINCT` trên Nhiều Cột: Tìm bộ giá trị duy nhất

Khi `DISTINCT` được áp dụng cho nhiều cột, nó sẽ trả về các hàng mà *kết hợp* của tất cả các cột được chỉ định là duy nhất. Điều này có nghĩa là một hàng chỉ được coi là trùng lặp nếu *tất cả* các giá trị trong các cột được chọn đều giống nhau với một hàng khác.

**Cú pháp:**

```sql
SELECT DISTINCT column1, column2, ...
FROM table_name;
```

**Ví dụ:**
Tìm tất cả các kết hợp duy nhất của `department` và `manufacturer` trong bảng `products`.

```sql
SELECT DISTINCT department, manufacturer
FROM products;
```

**Kết quả:**

```
 department |  manufacturer 
------------+----------------
 Outdoors   | AdventureGear
 Toys       | KidCo
 Industrial | ProTools
 Industrial | OceanCraft
 Toys       | RoboFun
(5 rows)
```

Ở đây, mặc dù 'Toys' và 'Industrial' xuất hiện nhiều lần trong cột `department`, nhưng `(Toys, KidCo)`, `(Toys, RoboFun)`, `(Industrial, ProTools)`, `(Industrial, OceanCraft)` là các cặp giá trị duy nhất.

> [!TIP]
> **Vibe Coding với Antigravity IDE:** Khi bạn đang trong giai đoạn khám phá dữ liệu (data exploration), Antigravity IDE có thể hỗ trợ "Vibe Coding" một cách mạnh mẽ. Bạn có thể nhanh chóng thử nghiệm các truy vấn `SELECT DISTINCT` với các tổ hợp cột khác nhau mà không cần lo lắng về cú pháp hay cấu trúc bảng. Các subagent của Antigravity có thể tự động đề xuất các cặp cột tiềm năng để kiểm tra tính duy nhất, giúp bạn nhanh chóng hiểu được các mối quan hệ và sự phân bố dữ liệu trong bảng của mình.

### 1.3. Đếm Giá Trị Duy Nhất: `COUNT(DISTINCT ...)`

Một trong những ứng dụng phổ biến nhất của `DISTINCT` là kết hợp nó với hàm tổng hợp `COUNT()` để đếm số lượng giá trị duy nhất.

**Cú pháp:**

```sql
SELECT COUNT(DISTINCT column_name)
FROM table_name;
```

**Ví dụ:**
Để đếm số lượng phòng ban duy nhất trong bảng `products`:

```sql
SELECT COUNT(DISTINCT department) AS unique_departments_count
FROM products;
```

**Kết quả:**

```
 unique_departments_count 
--------------------------
                        3
(1 row)
```

#### `COUNT(DISTINCT ...)` với nhiều cột trong PostgreSQL

PostgreSQL hỗ trợ đếm các *bộ giá trị* duy nhất trên nhiều cột một cách trực tiếp.

**Cú pháp trong PostgreSQL:**

```sql
SELECT COUNT(DISTINCT column1, column2)
FROM table_name;
```

**Ví dụ:**
Đếm số lượng kết hợp duy nhất của `department` và `name`:

```sql
SELECT COUNT(DISTINCT department, name) AS unique_department_name_combinations
FROM products;
```

**Kết quả:**

```
 unique_department_name_combinations 
-------------------------------------
                                   9
(1 row)
```

Trong ví dụ trên, chúng ta có 9 sản phẩm, và mỗi sản phẩm có một tên duy nhất (kể cả "Toy Car" xuất hiện 2 lần nhưng có `id` và `price` khác nhau, nên vẫn là các bản ghi khác nhau). Nếu chúng ta có hai sản phẩm hoàn toàn giống nhau về `department` và `name`, thì `COUNT(DISTINCT department, name)` sẽ chỉ đếm một lần.

### 1.4. Xử lý `NULL` với `DISTINCT`

Trong ngữ cảnh của `DISTINCT`, các giá trị `NULL` được coi là bằng nhau. Điều này có nghĩa là nếu có nhiều hàng chứa `NULL` trong cột được chọn `DISTINCT`, chỉ một giá trị `NULL` sẽ được trả về trong tập kết quả.

**Ví dụ:**
Giả sử chúng ta thêm một sản phẩm không có nhà sản xuất:

```sql
INSERT INTO products (name, department, price, weight, manufacturer) VALUES
('Unlabeled Item', 'Miscellaneous', 10.00, 0.1, NULL);

SELECT DISTINCT manufacturer
FROM products;
```

**Kết quả:**

```
  manufacturer   
-----------------
 KidCo
 AdventureGear
 ProTools
 OceanCraft
 RoboFun
 <NULL>
(6 rows)
```

Chỉ có một `NULL` được hiển thị, mặc dù có thể có nhiều sản phẩm với giá trị `NULL` trong cột `manufacturer`.

### 1.5. So sánh `DISTINCT` và `GROUP BY`: Chọn công cụ phù hợp

`DISTINCT` và `GROUP BY` thường bị nhầm lẫn vì cả hai đều có thể được sử dụng để tìm các giá trị duy nhất. Tuy nhiên, chúng có mục đích và khả năng khác nhau:

*   **`DISTINCT`**:
    *   **Mục đích**: Chỉ đơn thuần lọc các hàng để trả về các giá trị hoặc kết hợp giá trị duy nhất.
    *   **Khả năng**: Không cho phép bạn thực hiện các phép tính tổng hợp trên các nhóm dữ liệu được tạo ra.
    *   **Cú pháp**: Đặt trực tiếp sau `SELECT`.

*   **`GROUP BY`**:
    *   **Mục đích**: Nhóm các hàng có cùng giá trị trong (các) cột được chỉ định thành một nhóm logic.
    *   **Khả năng**: Sau đó, bạn có thể áp dụng các hàm tổng hợp (như `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) cho *mỗi nhóm*.
    *   **Cú pháp**: Là một mệnh đề riêng biệt sau `FROM` và `WHERE`.

**Khi nào nên dùng cái nào?**
*   Sử dụng `DISTINCT` khi bạn chỉ cần một danh sách các giá trị duy nhất mà không cần thực hiện bất kỳ phép tính tổng hợp nào trên các nhóm đó.
*   Sử dụng `GROUP BY` khi bạn muốn nhóm các hàng có giá trị chung và sau đó tính toán một giá trị tổng hợp cho mỗi nhóm (ví dụ: tổng doanh số cho mỗi phòng ban, số lượng nhân viên trong mỗi phòng ban).

**Ví dụ minh họa:**
Để lấy danh sách các phòng ban duy nhất, cả hai truy vấn sau đều cho cùng kết quả:

```sql
-- Sử dụng DISTINCT
SELECT DISTINCT department
FROM products;

-- Sử dụng GROUP BY
SELECT department
FROM products
GROUP BY department;
```

Tuy nhiên, nếu bạn muốn đếm số lượng sản phẩm trong mỗi phòng ban, bạn *phải* sử dụng `GROUP BY`:

```sql
-- Sử dụng GROUP BY để đếm sản phẩm theo phòng ban
SELECT department, COUNT(id) AS product_count
FROM products
GROUP BY department;
```

**Kết quả:**

```
  department   | product_count 
---------------+---------------
 Outdoors      |             2
 Industrial    |             3
 Toys          |             3
 Miscellaneous |             1
(4 rows)
```

Bạn không thể đạt được kết quả tương tự chỉ với `DISTINCT`. Do đó, `GROUP BY` là một công cụ mạnh mẽ và linh hoạt hơn khi cần phân tích dữ liệu theo nhóm.

#### Cân nhắc về hiệu suất

Về mặt hiệu suất, đối với việc chỉ lấy danh sách duy nhất, `DISTINCT` và `GROUP BY` thường có hiệu suất tương đương vì cả hai đều thường yêu cầu sắp xếp hoặc băm dữ liệu. Tuy nhiên, khi các truy vấn trở nên phức tạp hơn, việc chọn đúng công cụ có thể ảnh hưởng đáng kể. `GROUP BY` linh hoạt hơn cho các truy vấn phức tạp liên quan đến tổng hợp, trong khi `DISTINCT` là lựa chọn rõ ràng và ngắn gọn hơn cho việc chỉ lọc các giá trị duy nhất.

### 1.6. `DISTINCT ON`: Tính năng mạnh mẽ dành riêng cho PostgreSQL

PostgreSQL cung cấp một tính năng mở rộng hữu ích là `DISTINCT ON (expression [, ...])`. Tính năng này cho phép bạn chọn *tất cả các cột* của một hàng, nhưng chỉ giữ lại hàng đầu tiên trong mỗi nhóm được xác định bởi các biểu thức `DISTINCT ON`, sau khi đã sắp xếp các hàng.

Điều này cực kỳ hữu ích khi bạn muốn lấy "hàng hoàn chỉnh" dựa trên tiêu chí duy nhất của một hoặc một vài cột, nhưng vẫn muốn kiểm soát hàng nào được chọn trong trường hợp có nhiều hàng trùng khớp với tiêu chí duy nhất đó (thường là hàng mới nhất, đắt nhất, v.v.).

**Cú pháp:**

```sql
SELECT DISTINCT ON (column1, column2, ...) column1, column2, ..., other_column
FROM table_name
ORDER BY column1, column2, ..., sort_column [ASC|DESC];
```

*   Các cột trong `DISTINCT ON (...)` phải là các cột đầu tiên trong mệnh đề `ORDER BY`.
*   PostgreSQL sẽ chọn hàng đầu tiên trong mỗi nhóm duy nhất được xác định bởi `DISTINCT ON` sau khi đã sắp xếp toàn bộ tập kết quả theo `ORDER BY`.

**Ví dụ:**
Giả sử chúng ta muốn tìm sản phẩm "Toy Car" đắt nhất từ mỗi nhà sản xuất. (Lưu ý: chúng ta có 2 "Toy Car" với giá khác nhau).

```sql
SELECT DISTINCT ON (name, manufacturer)
       id, name, department, price, weight, manufacturer
FROM products
WHERE name = 'Toy Car'
ORDER BY name, manufacturer, price DESC;
```

**Kết quả:**

```
 id |   name  | department | price | weight | manufacturer 
----+---------+------------+-------+--------+--------------
  9 | Toy Car | Toys       | 28.00 |    0.6 | KidCo
(1 row)
```

**Giải thích:**
1.  Truy vấn lọc các sản phẩm có `name = 'Toy Car'`.
2.  `ORDER BY name, manufacturer, price DESC` sắp xếp các kết quả. Trong trường hợp này, nó sẽ nhóm các sản phẩm có cùng `name` và `manufacturer`, sau đó sắp xếp chúng theo `price` giảm dần.
3.  `DISTINCT ON (name, manufacturer)` đảm bảo rằng chỉ có một hàng duy nhất cho mỗi cặp `(name, manufacturer)` được trả về. Do `ORDER BY price DESC`, hàng có giá cao nhất sẽ được chọn.

Nếu không có `DISTINCT ON`, bạn sẽ cần sử dụng các kỹ thuật phức tạp hơn như `ROW_NUMBER()` với `PARTITION BY` hoặc subquery. `DISTINCT ON` là một cách thanh lịch và hiệu quả hơn nhiều trong PostgreSQL.

---

## 2. Xử lý Giá Trị Cực Trị với Các Hàm `GREATEST()` và `LEAST()`

PostgreSQL cung cấp các hàm tích hợp sẵn để tìm giá trị lớn nhất hoặc nhỏ nhất từ một danh sách các biểu thức: `GREATEST()` và `LEAST()`. Chúng là những công cụ tiện lợi để triển khai các quy tắc kinh doanh liên quan đến ngưỡng hoặc giới hạn.

### 2.1. Hàm `GREATEST()`: Tìm giá trị lớn nhất

Hàm `GREATEST()` trả về giá trị lớn nhất từ một danh sách các biểu thức. Các biểu thức này có thể là cột, giá trị cố định, hoặc kết quả của các phép tính.

**Cú pháp:**

```sql
GREATEST(expression1, expression2, ...)
```

**Ví dụ đơn giản:**

```sql
SELECT GREATEST(10, 200, 30, 500.5);
```

**Kết quả:**

```
 greatest 
----------
    500.5
(1 row)
```

**Ứng dụng thực tế: Tính chi phí vận chuyển tối thiểu**

Giả sử quy tắc tính chi phí vận chuyển cho một sản phẩm là: chi phí cơ bản là 30 đô la, nhưng nếu gấp đôi trọng lượng sản phẩm lớn hơn 30 đô la, thì chi phí vận chuyển sẽ là gấp đôi trọng lượng.

```sql
SELECT
    name,
    weight,
    price,
    GREATEST(30.00, weight * 2) AS shipping_cost
FROM
    products;
```

**Kết quả:**

```
       name       | weight |  price  | shipping_cost 
------------------+--------+---------+---------------
 Toy Car          |    0.5 |   25.00 |         30.00
 Outdoor Tent     |    3.0 |  150.00 |         30.00
 Industrial Drill |   15.0 |  300.00 |         30.00
 Toy Robot        |    1.0 |   50.00 |         30.00
 Camping Chair    |    2.0 |   40.00 |         30.00
 Industrial Saw   |   20.0 |  450.00 |         40.00
 Toy Blocks       |    0.8 |   30.00 |         30.00
 Luxury Yacht     | 5000.0 | 1000.00 |      10000.00
 Toy Car          |    0.6 |   28.00 |         30.00
 Unlabeled Item   |    0.1 |   10.00 |         30.00
(10 rows)
```

*   Đối với "Industrial Saw" (trọng lượng 20.0), `20 * 2 = 40`. `GREATEST(30, 40)` là 40.
*   Đối với "Toy Car" (trọng lượng 0.5), `0.5 * 2 = 1`. `GREATEST(30, 1)` là 30.
*   Đối với "Luxury Yacht" (trọng lượng 5000.0), `5000 * 2 = 10000`. `GREATEST(30, 10000)` là 10000.

### 2.2. Hàm `LEAST()`: Tìm giá trị nhỏ nhất

Ngược lại với `GREATEST()`, hàm `LEAST()` trả về giá trị nhỏ nhất từ một danh sách các biểu thức.

**Cú pháp:**

```sql
LEAST(expression1, expression2, ...)
```

**Ví dụ đơn giản:**

```sql
SELECT LEAST(1, 20, 100, -5);
```

**Kết quả:**

```
 least 
-------
    -5
(1 row)
```

**Ứng dụng thực tế: Tính giá bán khuyến mãi với giới hạn tối thiểu**

Giả sử các sản phẩm đang được giảm giá. Quy tắc tính giá bán mới là: giá gốc nhân 0.5, nhưng không bao giờ thấp hơn 20 đô la.

```sql
SELECT
    name,
    price,
    LEAST(price * 0.5, 20.00) AS sale_price
FROM
    products;
```

**Kết quả:**

```
       name       |  price  | sale_price 
------------------+---------+------------
 Toy Car          |   25.00 |      12.50
 Outdoor Tent     |  150.00 |      20.00
 Industrial Drill |  300.00 |      20.00
 Toy Robot        |   50.00 |      20.00
 Camping Chair    |   40.00 |      20.00
 Industrial Saw   |  450.00 |      20.00
 Toy Blocks       |   30.00 |      15.00
 Luxury Yacht     | 1000.00 |      20.00
 Toy Car          |   28.00 |      14.00
 Unlabeled Item   |   10.00 |       5.00
(10 rows)
```

*   Đối với "Toy Car" (giá 25.00), `25 * 0.5 = 12.50`. `LEAST(12.50, 20.00)` là 12.50.
*   Đối với "Outdoor Tent" (giá 150.00), `150 * 0.5 = 75.00`. `LEAST(75.00, 20.00)` là 20.00.
*   Đối với "Unlabeled Item" (giá 10.00), `10 * 0.5 = 5.00`. `LEAST(5.00, 20.00)` là 5.00.

### 2.3. Xử lý `NULL` và Quy tắc Kiểu Dữ liệu trong `GREATEST`/`LEAST`

#### Xử lý `NULL`

Một điểm quan trọng cần lưu ý là cách `GREATEST()` và `LEAST()` xử lý các giá trị `NULL`. Nếu bất kỳ biểu thức nào trong danh sách truyền vào `GREATEST()` hoặc `LEAST()` là `NULL`, thì hàm sẽ trả về `NULL`. Điều này đúng ngay cả khi có các giá trị không `NULL` khác.

**Ví dụ:**

```sql
SELECT GREATEST(10, 20, NULL, 5); -- Kết quả: NULL
SELECT LEAST(10, 20, NULL, 5);    -- Kết quả: NULL
```

Để xử lý `NULL` một cách hiệu quả, bạn có thể sử dụng hàm `COALESCE()` để thay thế `NULL` bằng một giá trị mặc định (hoặc một giá trị khác không `NULL`) trước khi truyền vào `GREATEST()` hoặc `LEAST()`.

**Ví dụ với `COALESCE`:**

```sql
SELECT GREATEST(10, 20, COALESCE(NULL, 0), 5); -- Kết quả: 20
SELECT LEAST(10, 20, COALESCE(NULL, 100), 5); -- Kết quả: 5
```

#### Quy tắc Kiểu Dữ liệu

Các biểu thức truyền vào `GREATEST()` và `LEAST()` phải có các kiểu dữ liệu tương thích. PostgreSQL sẽ cố gắng ép kiểu (cast) tất cả các biểu thức về một kiểu dữ liệu chung phù hợp nhất. Nếu không thể tìm thấy một kiểu dữ liệu chung, một lỗi sẽ xảy ra.

**Ví dụ:**

```sql
SELECT GREATEST(10, '20'); -- Lỗi: operator does not exist: integer > text
SELECT GREATEST(10::TEXT, '20'); -- Kết quả: '20' (so sánh chuỗi)
SELECT GREATEST(10, '20'::INT); -- Kết quả: 20 (so sánh số)
```

Luôn đảm bảo rằng bạn đang so sánh các giá trị có cùng kiểu dữ liệu hoặc có thể được ép kiểu một cách rõ ràng để tránh lỗi không mong muốn và đảm bảo logic so sánh chính xác.

> [!TIP]
> **Antigravity IDE và Kiểm thử Quy tắc Kinh doanh:** Khi triển khai các quy tắc kinh doanh sử dụng `GREATEST()` hoặc `LEAST()`, Antigravity IDE có thể giúp bạn kiểm thử và xác thực chúng một cách tự động. Bạn có thể định nghĩa các kịch bản kiểm thử với các giá trị đầu vào khác nhau (bao gồm cả `NULL`) và để Antigravity chạy các truy vấn, so sánh kết quả với kỳ vọng. Khả năng chạy script ngầm và phản hồi tức thì của Antigravity giúp bạn "vibe" với các quy tắc, nhanh chóng điều chỉnh và đảm bảo tính đúng đắn của logic mà không cần nhiều thao tác thủ công.

---

## 3. Logic Điều Kiện Mạnh Mẽ với Mệnh Đề `CASE`

Mệnh đề `CASE` là một trong những tính năng linh hoạt và mạnh mẽ nhất trong SQL, cho phép bạn nhúng logic điều kiện `IF-THEN-ELSE` trực tiếp vào các truy vấn cơ sở dữ liệu. Nó cho phép bạn tạo ra các giá trị khác nhau trong một cột mới dựa trên các điều kiện của dữ liệu hiện có.

### 3.1. Giới thiệu và Các Dạng của `CASE`

`CASE` cho phép bạn định nghĩa một tập hợp các điều kiện và các kết quả tương ứng. Khi một điều kiện được đáp ứng, giá trị kết quả được trả về. Nếu không có điều kiện nào được đáp ứng, một giá trị mặc định (nếu có) sẽ được sử dụng.

Có hai dạng chính của mệnh đề `CASE`:

1.  **`CASE` đơn giản (Simple `CASE`)**: So sánh một biểu thức với nhiều giá trị có thể.
    ```sql
    CASE expression
        WHEN value1 THEN result1
        WHEN value2 THEN result2
        ...
        ELSE default_result
    END
    ```
    Dạng này hữu ích khi bạn so sánh một cột với một tập hợp các giá trị cụ thể.

2.  **`CASE` tìm kiếm (Searched `CASE`)**: Kiểm tra nhiều điều kiện Boolean khác nhau. Đây là dạng phổ biến và linh hoạt hơn, thường được sử dụng trong các tình huống phức tạp hơn khi các điều kiện không chỉ dựa trên một cột hoặc một giá trị đơn lẻ. Chúng ta sẽ tập trung chủ yếu vào dạng này.
    ```sql
    CASE
        WHEN condition1 THEN result1
        WHEN condition2 THEN result2
        ...
        ELSE default_result
    END
    ```

### 3.2. Cú pháp `CASE` (Dạng tìm kiếm) và Nguyên tắc hoạt động

**Cú pháp:**

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    -- ... có thể có nhiều WHEN ... THEN
    [ELSE default_result]
END AS new_column_name
```

*   `CASE` và `END`: Từ khóa bắt đầu và kết thúc mệnh đề `CASE`.
*   `WHEN condition THEN result`: Mỗi cặp `WHEN-THEN` xác định một điều kiện. Nếu `condition` là `TRUE`, `result` sẽ được trả về.
*   `ELSE default_result`: (Tùy chọn) Nếu không có điều kiện `WHEN` nào đúng, `default_result` sẽ được trả về. Nếu `ELSE` bị bỏ qua và không có điều kiện `WHEN` nào đúng, kết quả sẽ là `NULL`.
*   `AS new_column_name`: (Tùy chọn) Đặt tên cho cột mới được tạo ra bởi mệnh đề `CASE`.

#### Nguyên tắc hoạt động: Thứ tự đánh giá

Các điều kiện `WHEN` được đánh giá theo thứ tự xuất hiện. Ngay khi một điều kiện `WHEN` là `TRUE`, `CASE` sẽ trả về `result` tương ứng và *bỏ qua tất cả các điều kiện `WHEN` còn lại*. Điều này rất quan trọng khi các điều kiện có thể chồng chéo.

**Ví dụ Thực Tế: Phân loại Giá Sản Phẩm**

Chúng ta hãy phân loại giá sản phẩm thành "Cao cấp", "Trung bình", hoặc "Phổ thông" dựa trên giá của chúng:
*   Nếu giá lớn hơn 500 đô la, là "Cao cấp".
*   Nếu giá lớn hơn 100 đô la (nhưng không lớn hơn 500), là "Trung bình".
*   Nếu không thì là "Phổ thông".

```sql
SELECT
    name,
    price,
    CASE
        WHEN price > 500 THEN 'Cao cấp'
        WHEN price > 100 THEN 'Trung bình'
        ELSE 'Phổ thông'
    END AS price_category
FROM
    products;
```

**Kết quả:**

```
       name       |  price  | price_category 
------------------+---------+----------------
 Toy Car          |   25.00 | Phổ thông
 Outdoor Tent     |  150.00 | Trung bình
 Industrial Drill |  300.00 | Trung bình
 Toy Robot        |   50.00 | Phổ thông
 Camping Chair    |   40.00 | Phổ thông
 Industrial Saw   |  450.00 | Trung bình
 Toy Blocks       |   30.00 | Phổ thông
 Luxury Yacht     | 1000.00 | Cao cấp
 Toy Car          |   28.00 | Phổ thông
 Unlabeled Item   |   10.00 | Phổ thông
(10 rows)
```

**Giải thích:**
*   `Luxury Yacht` có giá 1000.00, lớn hơn 500, nên `price_category` là 'Cao cấp'.
*   `Industrial Saw` có giá 450.00. Điều kiện `price > 500` là `FALSE`. Điều kiện `price > 100` là `TRUE`, nên `price_category` là 'Trung bình'.
*   Các sản phẩm khác có giá nhỏ hơn hoặc bằng 100, nên không đáp ứng hai điều kiện `WHEN` đầu tiên, và rơi vào `ELSE` là 'Phổ thông'.

### 3.3. Tầm quan trọng của Mệnh đề `ELSE`

Mệnh đề `ELSE` trong `CASE` là tùy chọn. Tuy nhiên, việc bỏ qua nó có thể dẫn đến kết quả `NULL` nếu không có điều kiện `WHEN` nào được đáp ứng. Điều này thường là nguồn gốc của các lỗi hoặc hành vi không mong muốn trong ứng dụng.

**Ví dụ (bỏ qua `ELSE`):**

```sql
SELECT
    name,
    price,
    CASE
        WHEN price > 500 THEN 'Cao cấp'
        WHEN price > 100 THEN 'Trung bình'
        -- ELSE bị bỏ qua ở đây
    END AS price_category_no_else
FROM
    products;
```

**Kết quả (chỉ hiển thị vài hàng):**

```
       name       |  price  | price_category_no_else 
------------------+---------+------------------------
 Toy Car          |   25.00 | <NULL>
 Outdoor Tent     |  150.00 | Trung bình
 Industrial Drill |  300.00 | Trung bình
 Toy Robot        |   50.00 | <NULL>
 ...
 Luxury Yacht     | 1000.00 | Cao cấp
(10 rows)
```

Các sản phẩm có giá nhỏ hơn hoặc bằng 100 giờ đây có `price_category_no_else` là `NULL`. Luôn xem xét việc sử dụng mệnh đề `ELSE` để xử lý tất cả các trường hợp có thể xảy ra và làm cho truy vấn của bạn trở nên mạnh mẽ và dễ đoán hơn.

### 3.4. Ứng dụng Nâng cao của `CASE`

Mệnh đề `CASE` có tính linh hoạt cao và có thể được sử dụng trong nhiều ngữ cảnh khác nhau trong truy vấn SQL:

#### 3.4.1. Sắp xếp tùy chỉnh với `CASE` trong `ORDER BY`

Bạn có thể sử dụng `CASE` trong mệnh đề `ORDER BY` để định nghĩa một thứ tự sắp xếp không chuẩn hoặc ưu tiên.

**Ví dụ:**
Sắp xếp sản phẩm theo phòng ban, nhưng ưu tiên 'Industrial' lên đầu, sau đó đến 'Outdoors', và cuối cùng là 'Toys' và các phòng ban khác theo thứ tự bảng chữ cái.

```sql
SELECT name, department, price
FROM products
ORDER BY
    CASE department
        WHEN 'Industrial' THEN 1
        WHEN 'Outdoors'   THEN 2
        WHEN 'Toys'       THEN 3
        ELSE 4 -- Các phòng ban khác
    END,
    price DESC; -- Sắp xếp phụ theo giá giảm dần
```

**Kết quả (một phần):**

```
       name       |  department  |  price  
------------------+--------------+---------
 Luxury Yacht     | Industrial   | 1000.00
 Industrial Saw   | Industrial   |  450.00
 Industrial Drill | Industrial   |  300.00
 Outdoor Tent     | Outdoors     |  150.00
 Camping Chair    | Outdoors     |   40.00
 Toy Robot        | Toys         |   50.00
 Toy Car          | Toys         |   28.00
 Toy Car          | Toys         |   25.00
 Toy Blocks       | Toys         |   30.00
 Unlabeled Item   | Miscellaneous|   10.00
(10 rows)
```

#### 3.4.2. Tổng hợp có điều kiện (Conditional Aggregation)

`CASE` là xương sống của tổng hợp có điều kiện, nơi bạn muốn tính toán các giá trị tổng hợp (như `COUNT`, `SUM`, `AVG`) dựa trên các điều kiện cụ thể trong cùng một truy vấn.

**Ví dụ:**
Đếm số lượng sản phẩm trong mỗi danh mục giá ('Cao cấp', 'Trung bình', 'Phổ thông') trong một hàng duy nhất.

```sql
SELECT
    COUNT(CASE WHEN price > 500 THEN 1 ELSE NULL END) AS high_end_products,
    COUNT(CASE WHEN price > 100 AND price <= 500 THEN 1 ELSE NULL END) AS mid_range_products,
    COUNT(CASE WHEN price <= 100 THEN 1 ELSE NULL END) AS low_end_products
FROM
    products;
```

**Kết quả:**

```
 high_end_products | mid_range_products | low_end_products 
-------------------+--------------------+------------------
                 1 |                  3 |                6
(1 row)
```

**Giải thích:**
*   `COUNT(expression)` chỉ đếm các giá trị không `NULL`. Bằng cách đặt `ELSE NULL`, chúng ta đảm bảo rằng chỉ những hàng đáp ứng điều kiện mới được đếm.
*   Cách tiếp cận này cho phép tạo các báo cáo tổng hợp phức tạp chỉ với một truy vấn duy nhất.

### 3.5. `CASE` và Hiệu suất: Khi nào nên chuyển logic xuống CSDL?

Mặc dù logic điều kiện thường có thể được xử lý ở lớp ứng dụng (ví dụ: trong mã Python, Java), việc sử dụng `CASE` trong SQL có thể mang lại lợi ích đáng kể về hiệu suất và khả năng bảo trì:

*   **Giảm tải mạng**: Bằng cách thực hiện các phép tính này trực tiếp trong cơ sở dữ liệu, bạn giảm lượng dữ liệu cần truyền tải qua mạng đến ứng dụng. Thay vì gửi toàn bộ dữ liệu thô và xử lý trên client, bạn chỉ gửi kết quả đã được xử lý.
*   **Tận dụng tối ưu hóa CSDL**: Hệ quản trị cơ sở dữ liệu (DBMS) được thiết kế để xử lý dữ liệu hiệu quả cao. Việc sử dụng `CASE` cho phép DBMS tối ưu hóa việc thực thi logic cùng với các hoạt động truy cập dữ liệu khác.
*   **Tính nhất quán**: Đảm bảo rằng logic điều kiện được áp dụng nhất quán trên tất cả các ứng dụng hoặc báo cáo truy cập cùng một dữ liệu.
*   **Đơn giản hóa mã ứng dụng**: Giảm độ phức tạp của mã ứng dụng bằng cách đẩy logic xuống tầng dữ liệu, nơi nó thuộc về một cách tự nhiên hơn.

> [!TIP]
> **Tối ưu hóa Logic với Antigravity IDE:** Khi bạn có các đoạn mã logic điều kiện phức tạp trong ứng dụng của mình (ví dụ: một chuỗi `if/elif/else` dài trong Python), Antigravity IDE có thể giúp bạn "refactor" chúng thành mệnh đề `CASE` trong SQL. Một agent chuyên biệt trong Antigravity có thể phân tích mã ứng dụng của bạn, đề xuất cấu trúc `CASE` tương ứng, và thậm chí chạy thử nghiệm hiệu suất để chứng minh lợi ích của việc chuyển logic này xuống tầng cơ sở dữ liệu. Điều này thể hiện tinh thần "Vibe Coding" - để AI hỗ trợ bạn tìm ra cách tiếp cận tối ưu một cách tự nhiên và hiệu quả, không chỉ về mặt cú pháp mà còn về kiến trúc và hiệu suất hệ thống.

---

## Tóm tắt Chương 20

Trong chương này, chúng ta đã khám phá các công cụ và kỹ thuật mạnh mẽ trong PostgreSQL giúp bạn thao tác và phân tích dữ liệu một cách hiệu quả hơn. Dưới đây là những điểm chính cần ghi nhớ:

*   **`DISTINCT`**:
    *   Sử dụng để lấy các giá trị duy nhất từ một hoặc nhiều cột. `NULL` được coi là một giá trị duy nhất.
    *   Cú pháp: `SELECT DISTINCT column1, column2 FROM table_name;`
    *   `COUNT(DISTINCT column)` được dùng để đếm số lượng giá trị duy nhất trong một cột.
    *   Trong PostgreSQL, `COUNT(DISTINCT column1, column2)` đếm số lượng các bộ giá trị duy nhất.
    *   **Khác với `GROUP BY`**: `DISTINCT` chỉ lọc duy nhất, còn `GROUP BY` nhóm dữ liệu để áp dụng các hàm tổng hợp.
    *   **`DISTINCT ON (expression [, ...])`**: Tính năng mạnh mẽ của PostgreSQL cho phép chọn toàn bộ hàng dựa trên tính duy nhất của một tập hợp con các cột, kết hợp với `ORDER BY` để xác định hàng nào được chọn.

*   **`GREATEST()` và `LEAST()`**:
    *   `GREATEST(expression1, expression2, ...)` trả về giá trị lớn nhất từ danh sách các biểu thức.
    *   `LEAST(expression1, expression2, ...)` trả về giá trị nhỏ nhất từ danh sách các biểu thức.
    *   Cả hai hàm đều trả về `NULL` nếu bất kỳ biểu thức đầu vào nào là `NULL`. Sử dụng `COALESCE()` để xử lý `NULL` nếu cần.
    *   Các biểu thức đầu vào phải có kiểu dữ liệu tương thích.
    *   Hữu ích để áp dụng các quy tắc kinh doanh liên quan đến ngưỡng tối thiểu/tối đa.

*   **Mệnh đề `CASE`**:
    *   Cung cấp khả năng thực hiện logic điều kiện (`IF-THEN-ELSE`) trực tiếp trong truy vấn SQL.
    *   Có hai dạng: `CASE` đơn giản và `CASE` tìm kiếm (linh hoạt hơn).
    *   Cú pháp dạng tìm kiếm: `CASE WHEN condition1 THEN result1 [WHEN condition2 THEN result2 ...] [ELSE default_result] END AS new_column_name`.
    *   Các điều kiện `WHEN` được đánh giá theo thứ tự và điều kiện đầu tiên đúng sẽ được sử dụng.
    *   Mệnh đề `ELSE` là tùy chọn nhưng được khuyến nghị để xử lý tất cả các trường hợp và tránh kết quả `NULL`.
    *   `CASE` rất linh hoạt, có thể dùng để phân loại, tính toán động, sắp xếp tùy chỉnh, và tổng hợp có điều kiện.
    *   Việc chuyển logic điều kiện từ ứng dụng xuống cơ sở dữ liệu với `CASE` có thể cải thiện hiệu suất và tính nhất quán.

Nắm vững các khái niệm này sẽ giúp bạn viết các truy vấn SQL mạnh mẽ, linh hoạt và tối ưu hơn trong môi trường PostgreSQL, đồng thời tận dụng khả năng của các công cụ AI như Antigravity IDE để tăng tốc quá trình phát triển và tối ưu hóa mã của bạn.

<!-- REVIEWED_BY_AGENT -->
