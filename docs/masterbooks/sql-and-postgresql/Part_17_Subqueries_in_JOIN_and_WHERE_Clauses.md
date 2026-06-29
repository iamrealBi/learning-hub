# Phần 17: Subquery trong JOIN và WHERE

Trong lĩnh vực quản lý cơ sở dữ liệu, việc truy vấn dữ liệu phức tạp thường đòi hỏi khả năng kết hợp thông tin từ nhiều nguồn hoặc lọc dữ liệu dựa trên các điều kiện động. Subquery (truy vấn con) là một công cụ mạnh mẽ trong SQL giúp chúng ta thực hiện điều này một cách hiệu quả. Phần này sẽ đi sâu vào cách sử dụng subquery trong hai mệnh đề quan trọng: `JOIN` và `WHERE`, đặc biệt tập trung vào cú pháp và các trường hợp sử dụng trong PostgreSQL. Mục tiêu của chúng ta là hiểu rõ cơ chế hoạt động, các quy tắc về cấu trúc dữ liệu trả về, và cách viết các truy vấn con hiệu quả để giải quyết các bài toán thực tế.

## Subquery là gì? (Kiến thức nền tảng)

Subquery, hay truy vấn con, là một câu lệnh `SELECT` được lồng bên trong một câu lệnh SQL khác. Truy vấn con thường được sử dụng để:

*   Lọc dữ liệu dựa trên kết quả của một truy vấn khác.
*   Cung cấp một tập hợp các giá trị để so sánh.
*   Tạo ra một bảng tạm thời (derived table) để tham gia (join) với các bảng khác.

Nguyên tắc cơ bản là truy vấn con sẽ được thực thi trước, và kết quả của nó sẽ được sử dụng bởi truy vấn bên ngoài (outer query).

Có bốn loại subquery chính dựa trên cấu trúc dữ liệu mà chúng trả về:

1.  **Scalar Subquery**: Trả về một giá trị duy nhất (một hàng, một cột).
    *   Ví dụ: `(SELECT AVG(price) FROM products)`
2.  **Column Subquery**: Trả về một cột gồm nhiều giá trị (nhiều hàng, một cột).
    *   Ví dụ: `(SELECT product_id FROM orders WHERE user_id = 101)`
3.  **Row Subquery**: Trả về một hàng gồm nhiều giá trị (một hàng, nhiều cột).
    *   Ví dụ: `(SELECT name, price FROM products WHERE id = 5)`
4.  **Table Subquery (Derived Table)**: Trả về một bảng gồm nhiều hàng và nhiều cột.
    *   Ví dụ: `(SELECT user_id, COUNT(*) AS total_orders FROM orders GROUP BY user_id)`

Việc hiểu rõ các loại subquery này là nền tảng để áp dụng chúng một cách chính xác trong các mệnh đề `JOIN` và `WHERE`.

## Sử dụng Subquery trong Mệnh đề JOIN

Khi một subquery được đặt trong mệnh đề `FROM` hoặc `JOIN`, nó hoạt động như một bảng ảo hoặc bảng tạm thời, được gọi là **Derived Table** (bảng dẫn xuất). Kết quả của subquery này sau đó có thể được nối (join) với các bảng khác trong truy vấn chính.

> [!NOTE]
> Khi sử dụng subquery làm derived table trong mệnh đề `FROM` hoặc `JOIN`, BẮT BUỘC phải gán một bí danh (alias) cho subquery đó. Nếu không, PostgreSQL sẽ báo lỗi cú pháp.

### Cú pháp cơ bản

```sql
SELECT
    t1.column_a,
    dt.column_b
FROM
    main_table AS t1
JOIN
    (SELECT column_x, column_y FROM another_table WHERE condition) AS dt -- 'dt' là bí danh bắt buộc
ON
    t1.column_a = dt.column_x;
```

### Ví dụ minh họa: Tìm tên người dùng đã đặt một sản phẩm cụ thể

Hãy tưởng tượng chúng ta muốn tìm tên của tất cả những người dùng đã đặt mua sản phẩm có `product_id` là `3`.

Đầu tiên, chúng ta xác định subquery để lấy `user_id` của những người đã đặt `product_id = 3`:

```sql
SELECT user_id
FROM orders
WHERE product_id = 3;
```
Kết quả của subquery này sẽ là một danh sách các `user_id`. Ví dụ:
```
 user_id
---------
       5
       5
       8
(3 rows)
```

Tiếp theo, chúng ta sẽ sử dụng kết quả này làm một bảng tạm thời và nối nó với bảng `users` để lấy tên người dùng.

```sql
-- Ví dụ: Tìm tên của tất cả người dùng đã đặt sản phẩm có ID là 3
SELECT
    u.name
FROM
    users AS u
JOIN
    (
        SELECT user_id
        FROM orders
        WHERE product_id = 3
    ) AS o_filtered -- 'o_filtered' là bí danh cho subquery
ON
    u.id = o_filtered.user_id;
```

**Giải thích:**

1.  Subquery `(SELECT user_id FROM orders WHERE product_id = 3)` được thực thi trước, trả về một tập hợp các `user_id` liên quan đến sản phẩm có ID là 3.
2.  Tập hợp này được coi là một bảng tạm thời với bí danh `o_filtered`.
3.  Truy vấn chính sau đó nối bảng `users` (`u`) với bảng tạm `o_filtered` dựa trên điều kiện `u.id = o_filtered.user_id`.
4.  Kết quả là tên của những người dùng thỏa mãn điều kiện.

### Khi nào nên dùng Subquery trong JOIN?

> [!NOTE]
> Mặc dù ví dụ trên minh họa cơ chế của subquery trong `JOIN`, nhưng trong nhiều trường hợp đơn giản, bạn có thể đạt được kết quả tương tự (và thường hiệu quả hơn) bằng cách sử dụng `JOIN` thông thường kết hợp với mệnh đề `WHERE` trên bảng thứ hai.
>
> Ví dụ, truy vấn trên có thể được viết lại như sau:
> ```sql
> SELECT
>     u.name
> FROM
>     users AS u
> JOIN
>     orders AS o ON u.id = o.user_id
> WHERE
>     o.product_id = 3;
> ```
> Cú pháp này thường dễ đọc và dễ hiểu hơn cho các trường hợp lọc đơn giản.

Subquery trong `JOIN` (dưới dạng derived table) trở nên thực sự hữu ích khi bạn cần:

*   **Tiền xử lý hoặc tổng hợp dữ liệu trước khi nối**: Ví dụ, bạn cần tính tổng doanh thu cho mỗi khách hàng và sau đó nối kết quả này với bảng khách hàng để lấy thông tin chi tiết.
*   **Lọc dữ liệu phức tạp trước khi nối**: Khi điều kiện lọc quá phức tạp hoặc yêu cầu các hàm cửa sổ, việc lọc trong subquery có thể làm cho truy vấn chính gọn gàng hơn.
*   **Tăng tính module hóa**: Chia nhỏ một truy vấn lớn thành các phần nhỏ hơn, dễ quản lý hơn.

## Sử dụng Subquery trong Mệnh đề WHERE (Ứng dụng phổ biến hơn)

Subquery trong mệnh đề `WHERE` là một trong những ứng dụng phổ biến và mạnh mẽ nhất của truy vấn con. Chúng cho phép chúng ta lọc các hàng của truy vấn chính dựa trên kết quả động từ truy vấn con.

### Nguyên tắc hoạt động

Truy vấn con trong `WHERE` cung cấp một hoặc một tập hợp các giá trị mà truy vấn bên ngoài sẽ sử dụng để so sánh, thường với một cột của bảng chính. Điều quan trọng nhất cần nhớ là **toán tử so sánh bạn sử dụng trong mệnh đề `WHERE` sẽ quyết định cấu trúc dữ liệu mà subquery phải trả về**.

### Mối quan hệ giữa Toán tử và Cấu trúc dữ liệu trả về

Dưới đây là bảng tóm tắt các toán tử phổ biến và cấu trúc dữ liệu tương ứng mà subquery phải trả về:

| Toán tử trong `WHERE` | Cấu trúc dữ liệu trả về từ Subquery | Ví dụ toán tử | Mô tả |
| :-------------------- | :---------------------------------- | :------------- | :---- |
| So sánh đơn giá trị   | Một giá trị duy nhất (Scalar)       | `=`, `>`, `<`, `>=`, `<=`, `!=` | So sánh một giá trị từ cột chính với một giá trị duy nhất từ subquery. |
| So sánh tập hợp       | Một cột giá trị duy nhất (Column)   | `IN`, `NOT IN` | Kiểm tra xem giá trị từ cột chính có nằm trong (hoặc không nằm trong) tập hợp các giá trị từ subquery hay không. |
| Kiểm tra sự tồn tại   | Bất kỳ số hàng nào (Table)           | `EXISTS`, `NOT EXISTS` | Kiểm tra xem subquery có trả về bất kỳ hàng nào hay không. Thường được dùng với subquery tương quan. |
| So sánh với bất kỳ/tất cả | Một cột giá trị duy nhất (Column) | `ANY`, `ALL` | So sánh một giá trị với *bất kỳ* giá trị nào (`ANY`) hoặc *tất cả* các giá trị (`ALL`) trong tập hợp kết quả của subquery. |

### Ví dụ 1: Subquery trả về một cột giá trị duy nhất với toán tử `IN`

**Bài toán:** Hiển thị ID của các đơn hàng liên quan đến một sản phẩm có tỷ lệ giá trên trọng lượng (`price / weight`) lớn hơn 50.

Đầu tiên, chúng ta cần tìm `product_id` của tất cả các sản phẩm thỏa mãn điều kiện `price / weight > 50`. Đây sẽ là subquery:

```sql
SELECT id
FROM products
WHERE (price / weight) > 50;
```
Kết quả của subquery này sẽ là một cột các `id` sản phẩm, ví dụ:
```
 id
----
  2
  4
  6
(3 rows)
```

Vì subquery trả về một cột giá trị duy nhất, chúng ta sẽ sử dụng toán tử `IN` trong mệnh đề `WHERE` của truy vấn chính.

```sql
-- Ví dụ: Tìm các đơn hàng cho sản phẩm có tỷ lệ giá/trọng lượng cao
SELECT
    id AS order_id,
    product_id
FROM
    orders
WHERE
    product_id IN (
        SELECT id
        FROM products
        WHERE (price / weight) > 50
    );
```

**Giải thích:**

1.  Subquery `(SELECT id FROM products WHERE (price / weight) > 50)` được thực thi trước, trả về một danh sách các `product_id`.
2.  Truy vấn chính sau đó lọc bảng `orders`, chỉ giữ lại những hàng mà `product_id` của đơn hàng nằm trong danh sách các `product_id` được trả về bởi subquery.

> [!TIP]
> Tương tự như subquery trong `JOIN`, nhiều truy vấn sử dụng `WHERE IN` với subquery cũng có thể được viết lại bằng `JOIN`. Tuy nhiên, đôi khi việc sử dụng subquery trong `WHERE` có thể dễ đọc và hiểu hơn, đặc biệt khi điều kiện lọc phức tạp hoặc khi bạn không muốn thấy các cột từ bảng phụ trong kết quả cuối cùng. Trình tối ưu hóa truy vấn của PostgreSQL thường rất thông minh và có thể xử lý cả hai cách tiếp cận với hiệu suất tương đương.

### Ví dụ 2: Subquery trả về một giá trị duy nhất với toán tử `>`

**Bài toán:** Hiển thị tên của tất cả sản phẩm có giá cao hơn giá sản phẩm trung bình.

Để giải quyết bài toán này, chúng ta cần tìm giá trung bình của tất cả các sản phẩm. Đây sẽ là subquery:

```sql
SELECT AVG(price) FROM products;
```
Kết quả của subquery này sẽ là một giá trị số duy nhất (scalar value), ví dụ: `498.50`.

Vì subquery trả về một giá trị duy nhất, chúng ta có thể sử dụng toán tử so sánh đơn giá trị như `>`, `<`, `=`, v.v. Trong trường hợp này là `>`.

```sql
-- Ví dụ: Tìm các sản phẩm có giá cao hơn giá trung bình
SELECT
    name,
    price
FROM
    products
WHERE
    price > (
        SELECT AVG(price)
        FROM products
    );
```

**Giải thích:**

1.  Subquery `(SELECT AVG(price) FROM products)` được thực thi trước, tính toán giá trị trung bình của tất cả các sản phẩm.
2.  Giá trị trung bình này được trả về cho truy vấn chính.
3.  Truy vấn chính sau đó lọc bảng `products`, chỉ giữ lại những hàng mà `price` của sản phẩm lớn hơn giá trị trung bình đó.

### Ví dụ 3: Bài tập thực hành - Subquery trả về một giá trị duy nhất

**Bài toán:** In ra tên và giá của những chiếc điện thoại có giá cao hơn chiếc điện thoại "S5620 Monte".

Đây là một trường hợp cổ điển để sử dụng subquery trả về một giá trị duy nhất. Thay vì tra cứu thủ công giá của "S5620 Monte" (ví dụ: 250) và mã hóa cứng nó vào truy vấn, chúng ta sẽ sử dụng một subquery để tìm giá trị đó một cách động. Điều này giúp truy vấn linh hoạt hơn nếu giá của "S5620 Monte" thay đổi trong tương lai.

Đầu tiên, subquery sẽ tìm giá của chiếc điện thoại "S5620 Monte":

```sql
SELECT price
FROM phones
WHERE name = 'S5620 Monte';
```
Giả sử chỉ có một điện thoại với tên này, subquery sẽ trả về một giá trị duy nhất, ví dụ: `250`.

Sau đó, chúng ta sẽ sử dụng giá trị này để lọc các điện thoại khác.

```sql
-- Ví dụ: Tìm điện thoại đắt hơn "S5620 Monte"
SELECT
    name,
    price
FROM
    phones
WHERE
    price > (
        SELECT price
        FROM phones
        WHERE name = 'S5620 Monte'
    );
```

**Giải thích:**

1.  Subquery `(SELECT price FROM phones WHERE name = 'S5620 Monte')` được thực thi để lấy giá của điện thoại "S5620 Monte".
2.  Giá trị này được sử dụng trong mệnh đề `WHERE` của truy vấn chính.
3.  Truy vấn chính sau đó chọn `name` và `price` từ bảng `phones` cho những điện thoại có giá cao hơn giá trị đã tìm được.

> [!TIP]
> Luôn kiểm tra kết quả của subquery một cách độc lập trước khi tích hợp nó vào truy vấn chính. Điều này giúp bạn đảm bảo rằng subquery trả về cấu trúc dữ liệu và các giá trị mong muốn, từ đó dễ dàng gỡ lỗi hơn.

## Tóm tắt Phần

*   **Subquery (truy vấn con)** là một câu lệnh `SELECT` lồng bên trong một câu lệnh SQL khác, dùng để giải quyết các bài toán truy vấn phức tạp.
*   Khi sử dụng subquery trong mệnh đề `FROM` hoặc `JOIN`, nó hoạt động như một **Derived Table** (bảng dẫn xuất) và **bắt buộc phải có bí danh**. Derived tables hữu ích cho việc tiền xử lý hoặc tổng hợp dữ liệu trước khi nối.
*   Subquery trong mệnh đề `WHERE` là ứng dụng phổ biến và mạnh mẽ hơn, dùng để lọc dữ liệu của truy vấn chính dựa trên kết quả động của truy vấn con.
*   **Mối quan hệ giữa toán tử `WHERE` và cấu trúc dữ liệu trả về của subquery là rất quan trọng**:
    *   Các toán tử so sánh đơn (`=`, `>`, `<`) yêu cầu subquery trả về **một giá trị duy nhất (scalar)**.
    *   Các toán tử tập hợp (`IN`, `NOT IN`) yêu cầu subquery trả về **một cột giá trị duy nhất (column)**.
    *   Các toán tử kiểm tra sự tồn tại (`EXISTS`, `NOT EXISTS`) chỉ cần subquery trả về **bất kỳ hàng nào**.
*   Trình tối ưu hóa truy vấn của PostgreSQL rất mạnh mẽ, thường có thể xử lý các truy vấn sử dụng subquery và các truy vấn tương đương sử dụng `JOIN` với hiệu suất tương đương.
*   Luôn kiểm tra subquery độc lập để đảm bảo kết quả chính xác trước khi tích hợp vào truy vấn chính.