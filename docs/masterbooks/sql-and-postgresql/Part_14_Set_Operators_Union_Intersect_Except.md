# Phần 14: Các Toán Tử Tập Hợp (UNION, INTERSECT, EXCEPT)

Trong quản lý cơ sở dữ liệu, khả năng kết hợp, so sánh và trích xuất thông tin từ nhiều tập dữ liệu là một kỹ năng nền tảng. PostgreSQL, với tư cách là một hệ quản trị cơ sở dữ liệu quan hệ mạnh mẽ, cung cấp các toán tử tập hợp (Set Operators) cho phép chúng ta thực hiện các phép toán dựa trên lý thuyết tập hợp để thao tác với các tập kết quả từ các truy vấn `SELECT` khác nhau. Chương này sẽ đi sâu vào ba toán tử tập hợp chính: `UNION`, `INTERSECT`, và `EXCEPT`, cùng với các biến thể `ALL` của chúng. Chúng ta sẽ khám phá cú pháp, quy tắc hoạt động, cơ chế xử lý ngầm định của PostgreSQL, và những lưu ý quan trọng để sử dụng chúng một cách hiệu quả.

Đặc biệt, chúng ta sẽ liên hệ cách tư duy và áp dụng các toán tử này trong môi trường lập trình hiện đại, đặc biệt là với hệ thống Agentic AI như Antigravity IDE. Antigravity, với khả năng tự động hóa và hiểu ngữ cảnh, có thể hỗ trợ bạn từ việc lên ý tưởng (Vibe Coding) đến việc kiểm thử và tối ưu hóa các truy vấn tập hợp phức tạp.

## 1. Giới Thiệu Tổng Quan về Toán Tử Tập Hợp

Trong toán học, lý thuyết tập hợp nghiên cứu về các tập hợp và các phép toán cơ bản như hợp (union), giao (intersection), và hiệu (difference). Trong SQL, các toán tử tập hợp cho phép chúng ta mở rộng các khái niệm này để thao tác với các tập kết quả (result sets) được trả về từ các câu lệnh `SELECT`.

Mỗi câu lệnh `SELECT` trả về một tập hợp các hàng. Các toán tử tập hợp giúp chúng ta:

*   **Kết hợp** các hàng từ nhiều truy vấn thành một tập kết quả duy nhất (`UNION`).
*   **Tìm kiếm các hàng chung** giữa nhiều truy vấn (`INTERSECT`).
*   **Xác định các hàng duy nhất** trong một truy vấn mà không xuất hiện trong truy vấn khác (`EXCEPT`).

Để sử dụng các toán tử tập hợp, các truy vấn `SELECT` tham gia phải tuân thủ các quy tắc nghiêm ngặt về cấu trúc của tập kết quả, bao gồm số lượng và kiểu dữ liệu của các cột. Việc hiểu rõ những quy tắc này là chìa khóa để viết các truy vấn chính xác và hiệu quả.

> [!NOTE]
> Tất cả các ví dụ mã SQL trong chương này đều tuân thủ cú pháp chuẩn của PostgreSQL.

### 1.1. Vibe Coding và Toán Tử Tập Hợp trong Antigravity IDE

Khi làm việc với các toán tử tập hợp, tư duy "Vibe Coding" rất hữu ích. Thay vì ngay lập tức nghĩ về cú pháp `UNION` hay `INTERSECT`, bạn hãy tập trung vào "ý định" của mình:

*   "Tôi muốn một danh sách tổng hợp của A và B." (Vibe: `UNION`)
*   "Tôi muốn tìm những gì A và B có chung." (Vibe: `INTERSECT`)
*   "Tôi muốn biết những gì chỉ có trong A mà không có trong B." (Vibe: `EXCEPT`)

Trong Antigravity IDE, bạn có thể diễn đạt những "vibe" này bằng ngôn ngữ tự nhiên. Hệ thống Agentic AI của Antigravity sẽ phân tích ý định, kiểm tra lược đồ cơ sở dữ liệu (schema), và đề xuất hoặc tự động tạo ra các truy vấn SQL phù hợp. Nó có thể thậm chí gợi ý sử dụng `ALL` nếu nhận thấy hiệu suất là ưu tiên, hoặc đề xuất các phương án thay thế như `JOIN` nếu chúng phù hợp hơn với ngữ cảnh.

## 2. Chuẩn Bị Dữ Liệu Thực Hành

Để các ví dụ trở nên sống động và dễ thực hành, chúng ta sẽ tạo một cơ sở dữ liệu đơn giản với hai bảng `products` và `phones`. Bạn có thể chạy các lệnh SQL sau trong PostgreSQL để thiết lập môi trường.

```sql
-- Xóa bảng nếu đã tồn tại để đảm bảo môi trường sạch
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS phones;

-- Bảng products: Lưu trữ thông tin về các sản phẩm
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    price NUMERIC(10, 2) NOT NULL,
    weight NUMERIC(10, 2),
    stock_quantity INT
);

-- Chèn dữ liệu mẫu vào bảng products
INSERT INTO products (name, category, price, weight, stock_quantity) VALUES
('Laptop Pro', 'Electronics', 1200.00, 1.8, 50),
('Mechanical Keyboard', 'Accessories', 150.00, 1.2, 100),
('Gaming Mouse', 'Accessories', 75.00, 0.2, 200),
('4K Monitor', 'Electronics', 450.00, 5.5, 30),
('USB-C Hub', 'Accessories', 40.00, 0.1, 150),
('External SSD 1TB', 'Storage', 100.00, 0.05, 80),
('Wireless Earbuds', 'Audio', 120.00, 0.03, 120),
('Smartwatch X', 'Wearables', 250.00, 0.08, 70),
('Tablet Mini', 'Electronics', 300.00, 0.4, 60),
('Ergonomic Chair', 'Furniture', 350.00, 15.0, 20),
('Webcam HD', 'Accessories', 60.00, 0.1, 90),
('Desk Lamp', 'Home Office', 30.00, 0.8, 110);


-- Bảng phones: Lưu trữ thông tin về các điện thoại
CREATE TABLE phones (
    id SERIAL PRIMARY KEY,
    model VARCHAR(100) NOT NULL,
    manufacturer VARCHAR(50) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    release_year INT
);

-- Chèn dữ liệu mẫu vào bảng phones
INSERT INTO phones (model, manufacturer, price, release_year) VALUES
('iPhone 13', 'Apple', 799.00, 2021),
('Galaxy S22', 'Samsung', 799.00, 2022),
('Pixel 6 Pro', 'Google', 899.00, 2021),
('iPhone SE', 'Apple', 429.00, 2022),
('Galaxy A53', 'Samsung', 449.00, 2022),
('Mi 12', 'Xiaomi', 699.00, 2022),
('iPhone 14', 'Apple', 899.00, 2022),
('Galaxy S23', 'Samsung', 899.00, 2023),
('Pixel 7', 'Google', 599.00, 2022),
('Redmi Note 11', 'Xiaomi', 249.00, 2022),
('Nokia G21', 'Nokia', 169.00, 2022);
```

## 3. Toán Tử Tập Hợp UNION: Kết Hợp Các Tập Kết Quả

Toán tử `UNION` được sử dụng để kết hợp tập kết quả của hai hoặc nhiều câu lệnh `SELECT` thành một tập kết quả duy nhất. Theo mặc định, `UNION` sẽ loại bỏ tất cả các hàng trùng lặp từ tập kết quả cuối cùng. Điều này có nghĩa là nếu một hàng xuất hiện trong cả hai truy vấn con, nó sẽ chỉ xuất hiện một lần trong kết quả cuối cùng.

### 3.1. Cú Pháp Cơ Bản của UNION

```sql
SELECT column1, column2, ...
FROM table1
WHERE condition1
UNION
SELECT column1, column2, ...
FROM table2
WHERE condition2;
```

### 3.2. Ví Dụ Thực Tế: Kết Hợp Danh Sách Sản Phẩm

Giả sử chúng ta muốn tìm ra danh sách các sản phẩm từ hai tiêu chí khác nhau:

1.  Bốn sản phẩm có giá cao nhất.
2.  Bốn sản phẩm có tỷ lệ giá trên trọng lượng cao nhất (thường cho thấy sản phẩm nhỏ gọn, giá trị cao, hiệu quả vận chuyển tốt).

Chúng ta có thể viết hai truy vấn riêng biệt như sau:

```sql
-- Truy vấn 1: 4 sản phẩm đắt nhất
SELECT id, name, price, weight
FROM products
ORDER BY price DESC
LIMIT 4;

-- Truy vấn 2: 4 sản phẩm có tỷ lệ giá/trọng lượng cao nhất
SELECT id, name, price, weight
FROM products
ORDER BY (price / weight) DESC
LIMIT 4;
```

Để kết hợp kết quả của hai truy vấn này thành một danh sách duy nhất, chúng ta sử dụng `UNION`:

```sql
-- Kết hợp 4 sản phẩm đắt nhất và 4 sản phẩm có tỷ lệ giá/trọng lượng cao nhất
(SELECT id, name, price, weight
 FROM products
 ORDER BY price DESC
 LIMIT 4)
UNION
(SELECT id, name, price, weight
 FROM products
 ORDER BY (price / weight) DESC
 LIMIT 4);
```

> [!TIP]
> **Quy tắc dấu ngoặc đơn trong PostgreSQL:** Khi bạn áp dụng `ORDER BY` hoặc `LIMIT` cho các truy vấn con trước khi kết hợp chúng bằng toán tử tập hợp, bạn **bắt buộc phải bao quanh mỗi truy vấn con bằng dấu ngoặc đơn `()`**. Điều này là do toán tử tập hợp có độ ưu tiên thấp hơn `ORDER BY` và `LIMIT`. Nếu không có ngoặc đơn, PostgreSQL có thể hiểu sai rằng `ORDER BY` hoặc `LIMIT` đang được áp dụng cho toàn bộ tập kết quả `UNION`, hoặc sẽ báo lỗi cú pháp.

Khi chạy truy vấn trên, bạn có thể nhận thấy rằng tổng số hàng trả về có thể ít hơn 8 (ví dụ: 7 hàng). Điều này xảy ra vì `UNION` tự động loại bỏ các hàng trùng lặp. Nếu một sản phẩm cụ thể nằm trong cả danh sách "4 sản phẩm đắt nhất" và "4 sản phẩm có tỷ lệ giá/trọng lượng cao nhất", nó sẽ chỉ xuất hiện một lần trong kết quả cuối cùng.

### 3.3. UNION ALL: Giữ Lại Các Hàng Trùng Lặp

Nếu bạn muốn kết hợp tất cả các hàng từ các truy vấn `SELECT` mà không loại bỏ bất kỳ bản sao nào, bạn sử dụng `UNION ALL`. `UNION ALL` thường nhanh hơn `UNION` vì nó không cần thực hiện bước loại bỏ trùng lặp (duplication removal), vốn đòi hỏi sắp xếp hoặc băm dữ liệu.

```sql
-- Kết hợp tất cả các hàng, bao gồm cả bản sao
(SELECT id, name, price, weight
 FROM products
 ORDER BY price DESC
 LIMIT 4)
UNION ALL
(SELECT id, name, price, weight
 FROM products
 ORDER BY (price / weight) DESC
 LIMIT 4);
```

Với `UNION ALL`, nếu có một sản phẩm xuất hiện trong cả hai danh sách, nó sẽ được liệt kê hai lần trong tập kết quả cuối cùng. Tổng số hàng sẽ là tổng số hàng từ mỗi truy vấn con (trong ví dụ này là 4 + 4 = 8 hàng).

> [!NOTE]
> **Cơ chế ngầm (Under the Hood):**
> *   `UNION`: PostgreSQL sẽ thực hiện việc kết hợp tập kết quả từ các truy vấn con, sau đó áp dụng một thao tác `DISTINCT` trên toàn bộ tập kết quả đã hợp nhất. Điều này thường được thực hiện bằng cách sắp xếp (sort) hoặc băm (hash) dữ liệu để tìm và loại bỏ các hàng trùng lặp.
> *   `UNION ALL`: PostgreSQL chỉ đơn thuần nối (concatenate) các tập kết quả từ các truy vấn con. Đây là một thao tác đơn giản hơn và thường nhanh hơn đáng kể so với `UNION` khi số lượng hàng lớn, vì nó không cần chi phí cho việc loại bỏ trùng lặp.
> **Lời khuyên về hiệu suất:** Luôn ưu tiên sử dụng `UNION ALL` nếu bạn không cần loại bỏ các hàng trùng lặp, hoặc nếu bạn biết chắc chắn rằng không có hàng trùng lặp giữa các tập kết quả con.

### 3.4. Các Quy Tắc Quan Trọng khi Sử Dụng UNION/UNION ALL

Để các toán tử tập hợp hoạt động đúng, các truy vấn `SELECT` tham gia phải tuân thủ các quy tắc sau:

1.  **Số lượng cột phải giống nhau:** Mỗi truy vấn `SELECT` phải trả về cùng một số lượng cột.
2.  **Kiểu dữ liệu phải tương thích:** Các cột tương ứng trong mỗi truy vấn (ví dụ: cột thứ nhất của truy vấn 1 và cột thứ nhất của truy vấn 2) phải có kiểu dữ liệu tương thích. PostgreSQL sẽ cố gắng chuyển đổi kiểu dữ liệu ngầm định (implicit type coercion) nếu có thể, nhưng tốt nhất là nên đảm bảo chúng có cùng kiểu hoặc kiểu dữ liệu có thể chuyển đổi rõ ràng mà không mất dữ liệu.
    *   **Ví dụ về kiểu dữ liệu tương thích:** `INT` và `BIGINT`, `TEXT` và `VARCHAR`, `DATE` và `TIMESTAMP`. PostgreSQL sẽ tự động chuyển đổi sang kiểu dữ liệu "rộng hơn" hoặc "ưu tiên hơn".
    *   **Ví dụ về lỗi kiểu dữ liệu không tương thích:**
        ```sql
        SELECT name FROM products
        UNION
        SELECT price FROM products; -- Lỗi: Kiểu dữ liệu của 'name' (text) và 'price' (numeric) không tương thích.
        ```
    *   **Xử lý `NULL`:** Đối với các toán tử tập hợp, PostgreSQL coi hai giá trị `NULL` là bằng nhau khi so sánh các hàng để loại bỏ trùng lặp (trong `UNION`, `INTERSECT`, `EXCEPT`). Đây là một điểm khác biệt quan trọng so với các phép so sánh `NULL` trong mệnh đề `WHERE` (ví dụ: `NULL = NULL` là `UNKNOWN`).
3.  **Thứ tự cột:** Mặc dù không bắt buộc phải có cùng tên cột trong các truy vấn con, nhưng các cột tương ứng sẽ được ghép nối theo thứ tự vị trí của chúng trong câu lệnh `SELECT`. Tên cột của tập kết quả cuối cùng sẽ được lấy từ truy vấn `SELECT` đầu tiên. Bạn có thể sử dụng alias (bí danh) để đặt tên rõ ràng cho các cột trong truy vấn đầu tiên nếu cần.

> [!WARNING]
> Việc không tuân thủ các quy tắc này sẽ dẫn đến lỗi cú pháp hoặc lỗi kiểu dữ liệu. Antigravity IDE có thể giúp bạn xác định và sửa chữa những lỗi này bằng cách phân tích lược đồ và đề xuất các chỉnh sửa.

## 4. Toán Tử Tập Hợp INTERSECT: Giao Các Tập Kết Quả

Toán tử `INTERSECT` được sử dụng để trả về các hàng *chung* (có mặt trong cả hai tập kết quả) từ hai hoặc nhiều câu lệnh `SELECT`. Tương tự như `UNION`, `INTERSECT` theo mặc định sẽ loại bỏ các hàng trùng lặp.

### 4.1. Cú Pháp Cơ Bản của INTERSECT

```sql
SELECT column1, column2, ...
FROM table1
WHERE condition1
INTERSECT
SELECT column1, column2, ...
FROM table2
WHERE condition2;
```

### 4.2. Ví Dụ Thực Tế: Tìm Sản Phẩm Chung

Sử dụng lại ví dụ về sản phẩm, chúng ta có thể tìm xem sản phẩm nào vừa nằm trong top 4 đắt nhất, vừa nằm trong top 4 có tỷ lệ giá/trọng lượng cao nhất:

```sql
-- Tìm sản phẩm chung giữa 4 sản phẩm đắt nhất và 4 sản phẩm có tỷ lệ giá/trọng lượng cao nhất
(SELECT id, name, price, weight
 FROM products
 ORDER BY price DESC
 LIMIT 4)
INTERSECT
(SELECT id, name, price, weight
 FROM products
 ORDER BY (price / weight) DESC
 LIMIT 4);
```

Kết quả của truy vấn này sẽ chỉ hiển thị những sản phẩm xuất hiện trong *cả hai* tập kết quả con. Nếu một sản phẩm (ví dụ: 'Laptop Pro') xuất hiện trong cả hai danh sách, nó sẽ là kết quả duy nhất được trả về.

### 4.3. INTERSECT ALL: Giữ Lại Các Hàng Trùng Lặp (với sắc thái riêng)

Toán tử `INTERSECT ALL` trả về tất cả các hàng chung, bao gồm cả các bản sao. Tuy nhiên, cách xử lý bản sao của `INTERSECT ALL` có một sắc thái riêng:
Nếu một hàng xuất hiện `m` lần trong truy vấn đầu tiên và `n` lần trong truy vấn thứ hai, thì `INTERSECT ALL` sẽ trả về hàng đó `MIN(m, n)` lần.

```sql
-- Tìm sản phẩm chung, bao gồm cả bản sao (MIN(m, n) lần)
-- Ví dụ này ít có khả năng tạo ra bản sao do LIMIT và ID duy nhất
(SELECT id, name, price, weight
 FROM products
 ORDER BY price DESC
 LIMIT 4)
INTERSECT ALL
(SELECT id, name, price, weight
 FROM products
 ORDER BY (price / weight) DESC
 LIMIT 4);
```

Để minh họa rõ hơn về `MIN(m, n)`, hãy xem xét ví dụ sau (không dùng `LIMIT` để dễ hình dung):

*   Truy vấn A trả về: `{1, 2, 2, 3}`
*   Truy vấn B trả về: `{2, 2, 2, 4}`
*   `A INTERSECT B` = `{2}` (loại bỏ trùng lặp)
*   `A INTERSECT ALL B` = `{2, 2}` (số `2` xuất hiện 2 lần trong A, 3 lần trong B. `MIN(2, 3) = 2`, nên `2` xuất hiện 2 lần trong kết quả).

> [!NOTE]
> Các quy tắc về số lượng cột và kiểu dữ liệu tương thích, cũng như xử lý `NULL`, tương tự như `UNION` cũng áp dụng cho `INTERSECT`.

### 4.4. Antigravity IDE và Phân Tích INTERSECT

Khi bạn yêu cầu Antigravity IDE "tìm các sản phẩm chung giữa hai danh sách," nó không chỉ tạo ra truy vấn `INTERSECT` mà còn có thể:

*   **Phân tích sự trùng lặp:** Nếu có khả năng xảy ra trùng lặp trong các truy vấn con, Antigravity có thể hỏi bạn có muốn giữ lại tất cả các bản sao (`INTERSECT ALL`) hay chỉ các giá trị duy nhất (`INTERSECT`), giúp bạn đưa ra quyết định dựa trên "vibe" chính xác hơn.
*   **Kiểm tra dữ liệu:** Sử dụng các subagent, Antigravity có thể chạy truy vấn trên dữ liệu mẫu hoặc dữ liệu thực để hiển thị ngay lập tức kết quả của cả `INTERSECT` và `INTERSECT ALL`, giúp bạn trực quan hóa sự khác biệt của `MIN(m, n)` trong các tình huống cụ thể.

## 5. Toán Tử Tập Hợp EXCEPT: Hiệu Các Tập Kết Quả

Toán tử `EXCEPT` được sử dụng để trả về các hàng có trong tập kết quả của truy vấn đầu tiên, *nhưng không có* trong tập kết quả của truy vấn thứ hai. `EXCEPT` cũng loại bỏ các hàng trùng lặp theo mặc định.

### 5.1. Cú Pháp Cơ Bản của EXCEPT

```sql
SELECT column1, column2, ...
FROM table1
WHERE condition1
EXCEPT
SELECT column1, column2, ...
FROM table2
WHERE condition2;
```

### 5.2. Ví Dụ Thực Tế: Tìm Sản Phẩm Duy Nhất

Để tìm các sản phẩm nằm trong top 4 đắt nhất nhưng KHÔNG nằm trong top 4 có tỷ lệ giá/trọng lượng cao nhất:

```sql
-- Tìm các sản phẩm trong top 4 đắt nhất, nhưng không có trong top 4 tỷ lệ giá/trọng lượng cao nhất
(SELECT id, name, price, weight
 FROM products
 ORDER BY price DESC
 LIMIT 4)
EXCEPT
(SELECT id, name, price, weight
 FROM products
 ORDER BY (price / weight) DESC
 LIMIT 4);
```

Kết quả sẽ là các hàng chỉ xuất hiện trong tập kết quả của truy vấn đầu tiên và không có trong tập kết quả của truy vấn thứ hai. Nếu sản phẩm 'Laptop Pro' là sản phẩm chung, nó sẽ bị loại bỏ khỏi kết quả này.

### 5.3. Thứ Tự Truy Vấn Quan Trọng với EXCEPT

Một điểm cực kỳ quan trọng cần lưu ý với `EXCEPT` là **thứ tự của các truy vấn con có ảnh hưởng đến kết quả cuối cùng**. `A EXCEPT B` sẽ trả về các hàng có trong `A` nhưng không có trong `B`. Ngược lại, `B EXCEPT A` sẽ trả về các hàng có trong `B` nhưng không có trong `A`. Hai kết quả này thường là khác nhau.

**Minh họa bằng tập hợp:**

*   **Tập A:** `{1, 2, 3, 4}`
*   **Tập B:** `{2, 4, 5, 6}`

*   `A EXCEPT B` = `{1, 3}` (Các phần tử trong A nhưng không có trong B)
*   `B EXCEPT A` = `{5, 6}` (Các phần tử trong B nhưng không có trong A)

### 5.4. EXCEPT ALL: Giữ Lại Các Hàng Trùng Lặp (với sắc thái riêng)

Toán tử `EXCEPT ALL` trả về tất cả các hàng có trong truy vấn đầu tiên nhưng không có trong truy vấn thứ hai, bao gồm cả các bản sao.
Nếu một hàng xuất hiện `m` lần trong truy vấn đầu tiên và `n` lần trong truy vấn thứ hai, thì `EXCEPT ALL` sẽ trả về hàng đó `MAX(0, m - n)` lần.

```sql
-- Tìm các sản phẩm trong top 4 đắt nhất, nhưng không có trong top 4 tỷ lệ giá/trọng lượng cao nhất, bao gồm cả bản sao
(SELECT id, name, price, weight
 FROM products
 ORDER BY price DESC
 LIMIT 4)
EXCEPT ALL
(SELECT id, name, price, weight
 FROM products
 ORDER BY (price / weight) DESC
 LIMIT 4);
```

Tương tự như `INTERSECT ALL`, trong các trường hợp thông thường với `LIMIT` và `id` duy nhất, `EXCEPT ALL` có thể cho kết quả tương tự như `EXCEPT`. Tuy nhiên, việc hiểu rõ cách nó xử lý các bản sao là rất quan trọng cho các tình huống phức tạp hơn.

Để minh họa `MAX(0, m - n)`:

*   Truy vấn A trả về: `{1, 2, 2, 3}`
*   Truy vấn B trả về: `{2, 2, 4}`
*   `A EXCEPT B` = `{1, 3}`
*   `A EXCEPT ALL B` = `{1, 3}` (Số `1` xuất hiện 1 lần trong A, 0 lần trong B. `MAX(0, 1-0) = 1`. Số `2` xuất hiện 2 lần trong A, 2 lần trong B. `MAX(0, 2-2) = 0`. Số `3` xuất hiện 1 lần trong A, 0 lần trong B. `MAX(0, 1-0) = 1`).

> [!NOTE]
> Các quy tắc về số lượng cột và kiểu dữ liệu tương thích, cũng như xử lý `NULL`, tương tự như `UNION` cũng áp dụng cho `EXCEPT`.

### 5.5. Antigravity IDE và Quyết Định Thứ Tự Truy Vấn với EXCEPT

Với `EXCEPT`, thứ tự là cực kỳ quan trọng. Antigravity IDE, thông qua khả năng lập kế hoạch và hiểu ngữ cảnh, có thể:

*   **Gợi ý các trường hợp sử dụng:** Nếu bạn diễn đạt "vibe" muốn tìm sự khác biệt, Antigravity có thể hỏi bạn muốn tìm `A trừ B` hay `B trừ A`, và giải thích sự khác biệt tiềm năng.
*   **Mô phỏng kết quả:** Trước khi bạn chạy truy vấn, Antigravity có thể hiển thị một bản xem trước hoặc mô tả kết quả của cả hai trường hợp `A EXCEPT B` và `B EXCEPT A`, giúp bạn chọn đúng ý định.
*   **Phân tích tác động của `ALL`:** Tương tự như `UNION` và `INTERSECT`, Antigravity có thể giúp bạn hiểu khi nào nên sử dụng `EXCEPT ALL` bằng cách minh họa cách nó xử lý các bản sao.

## 6. Thực Hành Ứng Dụng Tổng Hợp Các Toán Tử Tập Hợp

Để củng cố kiến thức, hãy cùng giải quyết một bài tập thực tế sử dụng toán tử `UNION` và mở rộng với `INTERSECT`/`EXCEPT`.

### 6.1. Đề Bài 1: Kết Hợp Danh Sách Nhà Sản Xuất (UNION)

Hãy viết hai truy vấn khác nhau và kết hợp chúng bằng `UNION`:

1.  In ra tên nhà sản xuất (`manufacturer`) của tất cả các điện thoại có giá (`price`) dưới 170 đô la.
2.  In ra tên của tất cả các nhà sản xuất đã tạo ra nhiều hơn hai điện thoại.

### 6.2. Phân Tích & Giải Pháp Đề Bài 1

Chúng ta sẽ xây dựng từng truy vấn con một, sau đó kết hợp chúng.

**Bước 1: Truy vấn 1 - Điện thoại có giá dưới 170 đô la**
Truy vấn này khá đơn giản, chỉ cần chọn cột `manufacturer` từ bảng `phones` với điều kiện `price < 170`.

```sql
SELECT manufacturer
FROM phones
WHERE price < 170;
```

**Bước 2: Truy vấn 2 - Nhà sản xuất tạo ra nhiều hơn hai điện thoại**
Truy vấn này yêu cầu chúng ta đếm số lượng điện thoại cho mỗi nhà sản xuất và sau đó lọc ra những nhà sản xuất có số lượng lớn hơn 2. Đây là một trường hợp điển hình để sử dụng `GROUP BY` kết hợp với `HAVING`.

```sql
SELECT manufacturer
FROM phones
GROUP BY manufacturer
HAVING COUNT(*) > 2;
```

**Bước 3: Kết hợp hai truy vấn bằng UNION**
Bây giờ, chúng ta sẽ đặt hai truy vấn này lại với nhau bằng toán tử `UNION`. Trong trường hợp này, các truy vấn con không sử dụng `ORDER BY` hay `LIMIT` được áp dụng cho chính chúng, nên không cần dấu ngoặc đơn quanh từng `SELECT`. Tuy nhiên, việc sử dụng dấu ngoặc đơn đôi khi giúp tăng tính rõ ràng.

```sql
-- Kết hợp tên nhà sản xuất từ hai tiêu chí
SELECT manufacturer
FROM phones
WHERE price < 170
UNION
SELECT manufacturer
FROM phones
GROUP BY manufacturer
HAVING COUNT(*) > 2;
```

> [!TIP]
> Lưu ý rằng cả hai truy vấn con đều chọn cùng một cột (`manufacturer`) với cùng kiểu dữ liệu (`varchar`), đáp ứng các quy tắc của `UNION`. `UNION` sẽ tự động loại bỏ các nhà sản xuất xuất hiện trong cả hai danh sách.

### 6.3. Đề Bài 2: Tìm Nhà Sản Xuất Chung và Duy Nhất (INTERSECT & EXCEPT)

Sử dụng hai tập kết quả từ Đề Bài 1:

*   **Tập A:** Nhà sản xuất điện thoại giá dưới 170 đô la.
*   **Tập B:** Nhà sản xuất tạo ra nhiều hơn hai điện thoại.

1.  Tìm các nhà sản xuất vừa có điện thoại giá dưới 170 đô la, vừa tạo ra nhiều hơn hai điện thoại (sử dụng `INTERSECT`).
2.  Tìm các nhà sản xuất chỉ có điện thoại giá dưới 170 đô la, nhưng KHÔNG tạo ra nhiều hơn hai điện thoại (sử dụng `EXCEPT`).

### 6.4. Phân Tích & Giải Pháp Đề Bài 2

**Bước 1: Tìm nhà sản xuất chung bằng INTERSECT**

```sql
-- Tìm nhà sản xuất chung
(SELECT manufacturer
 FROM phones
 WHERE price < 170)
INTERSECT
(SELECT manufacturer
 FROM phones
 GROUP BY manufacturer
 HAVING COUNT(*) > 2);
```

**Bước 2: Tìm nhà sản xuất duy nhất bằng EXCEPT**

```sql
-- Tìm nhà sản xuất chỉ có điện thoại giá dưới 170$, nhưng KHÔNG tạo ra nhiều hơn hai điện thoại
(SELECT manufacturer
 FROM phones
 WHERE price < 170)
EXCEPT
(SELECT manufacturer
 FROM phones
 GROUP BY manufacturer
 HAVING COUNT(*) > 2);
```

### 6.5. Tư Duy Antigravity IDE trong Thực Hành

Khi giải quyết các bài tập này trong Antigravity IDE, bạn có thể trải nghiệm quy trình làm việc như sau:

1.  **Vibe Coding:** Bạn có thể bắt đầu bằng cách mô tả ý tưởng của mình: "Tôi muốn xem các nhà sản xuất điện thoại giá rẻ VÀ các nhà sản xuất tạo ra nhiều mẫu điện thoại." Antigravity sẽ hiểu đây là một phép `UNION`.
2.  **Khám phá dữ liệu:** Antigravity có thể đề xuất các cột phù hợp (`manufacturer`, `price`, `COUNT(*)`) và giúp bạn xây dựng các truy vấn con.
3.  **Tối ưu hóa:** Khi bạn đã có truy vấn `UNION`, Antigravity có thể hỏi: "Bạn có quan tâm đến các nhà sản xuất trùng lặp không? Nếu không, `UNION ALL` có thể nhanh hơn."
4.  **Kiểm thử và so sánh:** Đối với các bài tập `INTERSECT` và `EXCEPT`, Antigravity có thể hiển thị kết quả của cả hai toán tử song song, hoặc thậm chí đề xuất đảo ngược thứ tự các truy vấn con trong `EXCEPT` để bạn thấy sự khác biệt, giúp bạn hiểu sâu sắc hơn về hành vi của chúng.
5.  **Tạo môi trường:** Antigravity có thể tự động chạy các script `CREATE TABLE` và `INSERT` để bạn có ngay một môi trường thực hành, loại bỏ bước thiết lập thủ công.

## 7. Tóm Tắt & Kết Luận

Trong chương này, chúng ta đã khám phá các toán tử tập hợp mạnh mẽ trong PostgreSQL, cho phép chúng ta thao tác và kết hợp các tập kết quả từ nhiều truy vấn `SELECT` khác nhau:

*   **`UNION`**: Kết hợp các hàng từ hai hoặc nhiều truy vấn, loại bỏ các bản sao.
*   **`UNION ALL`**: Kết hợp các hàng từ hai hoặc nhiều truy vấn, giữ lại tất cả các bản sao (thường nhanh hơn `UNION`).
*   **`INTERSECT`**: Trả về các hàng chung có mặt trong cả hai tập kết quả, loại bỏ các bản sao.
*   **`INTERSECT ALL`**: Trả về các hàng chung, bao gồm cả bản sao (số lần xuất hiện là `MIN(m, n)`).
*   **`EXCEPT`**: Trả về các hàng có trong truy vấn đầu tiên nhưng không có trong truy vấn thứ hai, loại bỏ các bản sao.
*   **`EXCEPT ALL`**: Trả về các hàng có trong truy vấn đầu tiên nhưng không có trong truy vấn thứ hai, bao gồm cả bản sao (số lần xuất hiện là `MAX(0, m - n)`).

**Các quy tắc và lưu ý quan trọng cần ghi nhớ:**

*   Tất cả các truy vấn con phải trả về cùng số lượng cột.
*   Các cột tương ứng phải có kiểu dữ liệu tương thích, và PostgreSQL xử lý `NULL` như các giá trị bằng nhau khi so sánh hàng cho các toán tử tập hợp.
*   Khi áp dụng `ORDER BY` hoặc `LIMIT` cho một truy vấn con trước khi kết hợp bằng toán tử tập hợp trong PostgreSQL, **bắt buộc phải bao quanh truy vấn con đó bằng dấu ngoặc đơn `()`**.
*   Thứ tự của các truy vấn con **quan trọng** đối với `EXCEPT` và `EXCEPT ALL`.
*   Sử dụng các phiên bản `ALL` khi có thể để tối ưu hóa hiệu suất, trừ khi việc loại bỏ trùng lặp là yêu cầu bắt buộc.

Việc nắm vững các toán tử tập hợp là rất quan trọng để thực hiện các phân tích dữ liệu phức tạp và tạo ra các báo cáo tổng hợp. Với sự hỗ trợ của các công cụ Agentic AI như Antigravity IDE, quá trình từ việc hình thành ý tưởng (Vibe Coding) đến việc triển khai, kiểm thử và tối ưu hóa các truy vấn này trở nên trực quan và hiệu quả hơn bao giờ hết.

<!-- REVIEWED_BY_AGENT -->
