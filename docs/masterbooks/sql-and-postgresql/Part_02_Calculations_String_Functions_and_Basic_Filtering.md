# Phần 2: Biến Đổi, Thao Tác Chuỗi và Lọc Dữ Liệu Cơ Bản trong PostgreSQL

Trong Phần 1, chúng ta đã tiếp cận với việc truy vấn và trích xuất dữ liệu thô. Tuy nhiên, dữ liệu thô hiếm khi đủ để đưa ra các quyết định kinh doanh hoặc phân tích sâu rộng. Sức mạnh thực sự của SQL nằm ở khả năng biến đổi, xử lý và chọn lọc dữ liệu một cách thông minh, từ đó tạo ra thông tin có giá trị. Phần này sẽ trang bị cho bạn những công cụ nền tảng để không chỉ lấy dữ liệu mà còn biến dữ liệu thành thông tin hữu ích và có ý nghĩa. Chúng ta sẽ khám phá cách:

*   Tạo ra các cột dữ liệu mới dựa trên các phép tính toán học.
*   Thao tác và định dạng dữ liệu dạng chuỗi.
*   Lọc các hàng dữ liệu dựa trên các điều kiện phức tạp.

Mục tiêu là xây dựng kỹ năng để bạn có thể chủ động định hình tập dữ liệu theo nhu cầu phân tích, vượt xa việc chỉ hiển thị dữ liệu đã lưu trữ.

## I. Tạo Cột Tính Toán (Calculated Columns)

Khi làm việc với cơ sở dữ liệu, thông tin bạn cần thường không được lưu trữ trực tiếp trong một cột duy nhất. Thay vào đó, nó là kết quả của một phép tính từ một hoặc nhiều cột hiện có. SQL cho phép bạn tạo ra các "cột tính toán" (calculated columns) tạm thời ngay trong kết quả truy vấn. Điều này giúp bạn thực hiện các phân tích phức tạp mà không cần thay đổi cấu trúc bảng dữ liệu gốc, giữ cho schema sạch sẽ và linh hoạt.

### 1. Khái Niệm và Các Toán Tử Số Học Cơ Bản

Một cột tính toán là một cột ảo, được định nghĩa "theo yêu cầu" trong câu lệnh `SELECT`. Giá trị của nó được suy ra bằng cách áp dụng các phép toán số học, các hàm, hoặc kết hợp cả hai lên các giá trị từ các cột khác.

PostgreSQL hỗ trợ đầy đủ các toán tử số học cơ bản:

| Toán tử | Mô tả        | Ví dụ             | Kết quả |
| :------ | :----------- | :---------------- | :------ |
| `+`     | Cộng         | `5 + 3`           | `8`     |
| `-`     | Trừ          | `5 - 3`           | `2`     |
| `*`     | Nhân         | `5 * 3`           | `15`    |
| `/`     | Chia         | `5 / 3`           | `1` (chia số nguyên) |
| `%`     | Chia lấy dư  | `5 % 3`           | `2`     |
| `^`     | Lũy thừa     | `5 ^ 3`           | `125`   |
| `|/`    | Căn bậc hai  | `|/ 25`           | `5`     |
| `!!`    | Giai thừa    | `5 !!`            | `120`   |

> [!NOTE]
> **Cơ chế chia số nguyên trong PostgreSQL:**
> Khi thực hiện phép chia giữa hai số nguyên (kiểu `INTEGER`, `BIGINT`), PostgreSQL sẽ thực hiện phép chia số nguyên, tức là kết quả sẽ bị cắt cụt phần thập phân. Ví dụ, `5 / 2` sẽ cho kết quả là `2`, không phải `2.5`.
> Để có kết quả thập phân chính xác, ít nhất một trong các toán hạng phải được chuyển đổi sang kiểu số thực (ví dụ: `NUMERIC`, `REAL`, `DOUBLE PRECISION`). Cách phổ biến và an toàn nhất là sử dụng `CAST`:
> ```sql
> SELECT 5 / 2;               -- Kết quả: 2
> SELECT 5.0 / 2;             -- Kết quả: 2.5 (vì 5.0 là số thực)
> SELECT CAST(5 AS NUMERIC) / 2; -- Kết quả: 2.5
> SELECT 5 / CAST(2 AS NUMERIC); -- Kết quả: 2.5
> ```
> Việc hiểu rõ cơ chế này là rất quan trọng để tránh sai lệch dữ liệu trong các phép tính tỷ lệ hoặc trung bình.

### 2. Ví dụ Ứng Dụng: Tính Mật độ Dân số

Giả sử chúng ta có bảng `cities` với các cột `name`, `population` (dân số) và `area` (diện tích theo km²). Để tính mật độ dân số (số người trên mỗi km²), chúng ta cần chia `population` cho `area`.

```sql
SELECT
    name,
    population,
    area,
    -- Để đảm bảo kết quả thập phân chính xác, ép kiểu một trong các toán hạng sang NUMERIC
    CAST(population AS NUMERIC) / area AS population_density
FROM
    cities;
```

Trong ví dụ này, `CAST(population AS NUMERIC) / area` là một cột tính toán. Kết quả của phép chia này sẽ được hiển thị như một cột mới trong tập kết quả, với độ chính xác thập phân mong muốn.

### 3. Đổi Tên Cột Tính Toán với `AS`

Khi bạn tạo một cột tính toán, PostgreSQL sẽ tự động gán cho nó một tên mặc định không rõ ràng (thường là `?column?` hoặc biểu thức của phép tính). Để làm cho kết quả dễ đọc, dễ hiểu và chuyên nghiệp hơn, bạn nên đổi tên cột tính toán bằng từ khóa `AS`.

```sql
SELECT
    name,
    population,
    area,
    CAST(population AS NUMERIC) / area AS density_per_sq_km -- Đổi tên cột thành 'density_per_sq_km'
FROM
    cities;
```

Bây giờ, cột mới sẽ có tên là `density_per_sq_km`, giúp người đọc dễ dàng hiểu ý nghĩa của các giá trị. Bạn có thể đặt bất kỳ tên hợp lệ nào cho cột này. Nếu tên cột chứa khoảng trắng hoặc ký tự đặc biệt, bạn cần đặt nó trong dấu ngoặc kép: `AS "Density (per km²)"`.

### 4. Ví dụ Thực Tế: Tính Doanh Thu Điện Thoại và Xử lý Kiểu Dữ liệu

Hãy xem xét một bảng `phones` với các cột `name`, `manufacturer`, `price` (giá) và `units_sold` (số lượng đã bán). Để tính tổng doanh thu cho mỗi điện thoại, chúng ta cần nhân `price` với `units_sold`.

```sql
-- Giả định bảng phones có dữ liệu như sau:
-- name        | manufacturer | price | units_sold
-- ------------|--------------|-------|-----------
-- iPhone 13   | Apple        | 999   | 1000000
-- Galaxy S22  | Samsung      | 799   | 800000
-- Pixel 6     | Google       | 599   | 300000

SELECT
    name,
    price,
    units_sold,
    price * units_sold AS revenue -- Tính toán doanh thu và đổi tên cột
FROM
    phones;
```

**Kết quả dự kiến:**

```
name        | price | units_sold | revenue
------------|-------|------------|----------
iPhone 13   | 999   | 1000000    | 999000000
Galaxy S22  | 799   | 800000     | 639200000
Pixel 6     | 599   | 300000     | 179700000
```

> [!CAUTION]
> **Giới hạn Kiểu Dữ liệu (Integer Overflow):**
> Kiểu `INTEGER` trong PostgreSQL có thể lưu trữ giá trị từ khoảng -2.1 tỷ đến +2.1 tỷ (2 * 10^9). Nếu kết quả của phép nhân `price * units_sold` vượt quá giới hạn này, bạn sẽ gặp lỗi "integer out of range" (hoặc "numeric field overflow" nếu kiểu dữ liệu không đủ lớn).
>
> Để tránh điều này, bạn cần ép kiểu (cast) một trong các toán hạng sang kiểu số lớn hơn như `BIGINT` (lên đến 9 * 10^18) hoặc `NUMERIC` (độ chính xác tùy ý):
> ```sql
> SELECT
>     name,
>     price,
>     units_sold,
>     CAST(price AS BIGINT) * units_sold AS revenue_bigint, -- Ép kiểu price sang BIGINT
>     CAST(price AS NUMERIC) * units_sold AS revenue_numeric -- Ép kiểu price sang NUMERIC
> FROM
>     phones;
> ```
> Luôn xem xét phạm vi giá trị có thể có của các cột tính toán để chọn kiểu dữ liệu phù hợp, đảm bảo tính toàn vẹn và chính xác của dữ liệu.

### 5. Vibe Coding & Antigravity IDE: Ứng Dụng với Cột Tính Toán

Trong môi trường Antigravity IDE, việc tạo cột tính toán trở nên trực quan và hiệu quả hơn:

*   **Vibe Coding:** Bạn "vibe" (cảm nhận) được nhu cầu: "Tôi cần một tỷ lệ ở đây," "Tôi muốn biết tổng doanh thu," "Tôi cần giá trị đã được chuyển đổi." Antigravity sẽ phản hồi lại ý định này.
*   **Gợi ý thông minh:** Khi bạn nhập `SELECT column1, column2, column1 * column2`, Antigravity có thể tự động gợi ý `AS revenue` dựa trên ngữ cảnh hoặc tên cột.
*   **Kiểm tra kiểu dữ liệu và cảnh báo tràn số:** Antigravity, với tư cách là một Agentic AI, có thể chạy một "script ngầm" để kiểm tra schema của `price` và `units_sold`. Nếu nhận thấy `price` là `INTEGER` và `units_sold` cũng là `INTEGER` nhưng có khả năng tích vượt quá `INTEGER` max, nó sẽ chủ động cảnh báo "Potential integer overflow. Consider casting to BIGINT or NUMERIC." và cung cấp cú pháp `CAST` mẫu.
*   **Xem trước kết quả:** Trước khi chạy toàn bộ truy vấn, Antigravity có thể cho phép bạn chạy thử chỉ phần tính toán `price * units_sold` trên một vài hàng mẫu để ngay lập tức thấy kết quả và kiểm tra độ chính xác, giúp bạn điều chỉnh phép tính hoặc kiểu dữ liệu kịp thời.
*   **Tự động hóa alias:** Sau khi bạn viết một biểu thức tính toán, Antigravity có thể tự động đề xuất một alias (tên cột) có ý nghĩa, ví dụ: `population / area` -> `AS population_density`.

## II. Làm Việc Với Chuỗi: Toán Tử và Hàm Chuỗi

Chuỗi (string) là một kiểu dữ liệu cực kỳ phổ biến, dùng để lưu trữ tên, địa chỉ, mô tả, mã sản phẩm, v.v. PostgreSQL cung cấp một bộ công cụ mạnh mẽ gồm các toán tử và hàm để thao tác, biến đổi và định dạng chuỗi, giúp bạn chuẩn hóa dữ liệu, tạo báo cáo dễ đọc hoặc trích xuất thông tin cụ thể.

### 1. Nối Chuỗi (Concatenation)

Nối chuỗi là hành động kết hợp hai hoặc nhiều chuỗi lại với nhau thành một chuỗi duy nhất. PostgreSQL cung cấp các cách sau:

#### a. Toán tử `||` (Pipe Operator)

Toán tử `||` là cách phổ biến và hiệu quả để nối chuỗi trong PostgreSQL, mang lại sự linh hoạt cao.

```sql
SELECT
    name,
    country,
    name || ', ' || country AS location_full, -- Nối tên, dấu phẩy và quốc gia
    'The city of ' || name || ' is in ' || country AS full_description
FROM
    cities;
```

**Giải thích:**
*   `name || ', ' || country`: Nối giá trị của cột `name`, một chuỗi ký tự cố định `', '` (dấu phẩy và khoảng trắng), và giá trị của cột `country`.
*   **Đặc điểm với `NULL`:** Nếu bất kỳ toán hạng nào trong phép nối `||` là `NULL`, toàn bộ kết quả của phép nối sẽ là `NULL`.
    *   Ví dụ: `'Hello' || NULL || ' World'` sẽ cho kết quả `NULL`.

#### b. Hàm `CONCAT()`

Hàm `CONCAT()` thực hiện chức năng tương tự như toán tử `||`, nhưng có cú pháp giống hàm hơn. Bạn truyền các chuỗi cần nối làm đối số cho hàm.

```sql
SELECT
    name,
    country,
    CONCAT(name, ', ', country) AS location_concat
FROM
    cities;
```

*   **Đặc điểm với `NULL`:** Không giống như `||`, hàm `CONCAT()` sẽ bỏ qua các đối số `NULL`. Điều này có nghĩa là nếu một trong các chuỗi là `NULL`, nó sẽ không làm cho toàn bộ kết quả trở thành `NULL` mà chỉ bỏ qua phần `NULL` đó.
    *   Ví dụ: `CONCAT('Hello', NULL, ' World')` sẽ cho kết quả `'Hello World'`.

#### c. Hàm `CONCAT_WS()` (Concatenate With Separator)

`CONCAT_WS()` là một hàm rất hữu ích khi bạn muốn nối nhiều chuỗi với một ký tự phân tách cố định và muốn bỏ qua các giá trị `NULL` một cách tự động.

```sql
SELECT
    first_name,
    last_name,
    CONCAT_WS(' ', first_name, middle_name, last_name) AS full_name_with_middle
FROM
    customers;
```

**Giải thích:**
*   Đối số đầu tiên `' '` là ký tự phân tách sẽ được chèn giữa các chuỗi còn lại.
*   Hàm sẽ chỉ chèn dấu phân tách giữa các chuỗi *không phải* `NULL`. Nếu `middle_name` là `NULL`, nó sẽ nối `first_name` và `last_name` với một khoảng trắng mà không để lại khoảng trắng thừa.

### 2. Các Hàm Chuỗi Phổ Biến Khác

PostgreSQL cung cấp một thư viện phong phú các hàm chuỗi để xử lý nhiều tình huống khác nhau:

*   `UPPER(string)`: Chuyển đổi tất cả các ký tự trong chuỗi thành chữ hoa.
*   `LOWER(string)`: Chuyển đổi tất cả các ký tự trong chuỗi thành chữ thường.
*   `LENGTH(string)`: Trả về độ dài (số ký tự) của chuỗi.
*   `TRIM([BOTH | LEADING | TRAILING] [characters] FROM string)`: Xóa các ký tự khoảng trắng (hoặc ký tự chỉ định) từ đầu, cuối hoặc cả hai đầu của chuỗi. Mặc định là `BOTH` và xóa khoảng trắng.
    *   `TRIM('   Hello World   ')` -> `'Hello World'`
    *   `TRIM(LEADING 'x' FROM 'xxHello Worldxx')` -> `'Hello Worldxx'`
*   `SUBSTRING(string FROM start [FOR length])`: Trích xuất một phần của chuỗi. `start` là vị trí bắt đầu (tính từ 1), `length` là số ký tự cần trích xuất.
    *   `SUBSTRING('PostgreSQL' FROM 1 FOR 4)` -> `'Post'`
    *   `SUBSTRING('PostgreSQL' FROM 5)` -> `'greSQL'`

#### Ví dụ về `UPPER()`, `LOWER()`, `LENGTH()`, `TRIM()`

```sql
SELECT
    name,
    country,
    UPPER(name) AS name_uppercase,           -- Tên thành phố viết hoa
    LOWER(country) AS country_lowercase,     -- Tên quốc gia viết thường
    LENGTH(name) AS name_length,             -- Độ dài tên thành phố
    TRIM('   ' || name || '   ') AS trimmed_name_example -- Loại bỏ khoảng trắng thừa
FROM
    cities;
```

### 3. Kết Hợp Các Hàm Chuỗi (Nesting Functions)

Bạn có thể kết hợp nhiều hàm chuỗi với nhau để thực hiện các thao tác phức tạp hơn. Các hàm được lồng sẽ thực thi từ trong ra ngoài.

```sql
SELECT
    name,
    country,
    -- Nối tên, quốc gia, sau đó chuyển thành chữ hoa, và cuối cùng lấy 10 ký tự đầu
    SUBSTRING(UPPER(CONCAT_WS(', ', name, country)) FROM 1 FOR 10) AS short_location_id
FROM
    cities;
```

Trong ví dụ này:
1.  `CONCAT_WS(', ', name, country)`: Nối tên và quốc gia với dấu phẩy và khoảng trắng (ví dụ: `Tokyo, Japan`).
2.  `UPPER(...)`: Chuyển chuỗi kết quả thành chữ hoa (ví dụ: `TOKYO, JAPAN`).
3.  `SUBSTRING(... FROM 1 FOR 10)`: Trích xuất 10 ký tự đầu tiên từ chuỗi chữ hoa (ví dụ: `TOKYO, JAP`).

### 4. Vibe Coding & Antigravity IDE: Ứng Dụng với Thao Tác Chuỗi

Antigravity IDE giúp bạn "vibe" và thực hiện các thao tác chuỗi một cách linh hoạt:

*   **Vibe Coding:** Bạn muốn "chuẩn hóa" dữ liệu, "trích xuất mã vùng," "tạo mã định danh duy nhất." Antigravity sẽ giúp bạn tìm hàm phù hợp.
*   **Gợi ý hàm thông minh:** Khi bạn nhập một tên cột kiểu chuỗi và gõ `.` hoặc bắt đầu gõ `UPPER`, Antigravity sẽ tự động gợi ý các hàm chuỗi phổ biến như `UPPER()`, `LOWER()`, `LENGTH()`, `TRIM()`, `SUBSTRING()`, `CONCAT()`.
*   **Xử lý `NULL` tự động:** Nếu bạn đang nối chuỗi và một cột có khả năng chứa `NULL`, Antigravity có thể gợi ý sử dụng `CONCAT_WS()` hoặc `COALESCE()` để xử lý `NULL` một cách duyên dáng, thay vì để toàn bộ kết quả trở thành `NULL`.
*   **Trình tạo chuỗi lồng nhau:** Đối với các hàm lồng nhau phức tạp, Antigravity có thể cung cấp một giao diện trực quan hoặc hướng dẫn từng bước để xây dựng biểu thức, hiển thị kết quả trung gian của mỗi hàm.
*   **Xem trước định dạng:** Antigravity có thể hiển thị một bản xem trước tức thì của dữ liệu chuỗi đã được biến đổi trên một vài hàng mẫu, giúp bạn tinh chỉnh các tham số (ví dụ: độ dài `SUBSTRING`, ký tự `TRIM`) mà không cần chạy toàn bộ truy vấn.
*   **Tạo mã định danh (ID) tự động:** Khi bạn muốn tạo một ID từ nhiều cột chuỗi, Antigravity có thể đề xuất các công thức nối chuỗi phổ biến (ví dụ: `SUBSTRING(UPPER(first_name), 1, 3) || SUBSTRING(UPPER(last_name), 1, 3) || id_number`) và kiểm tra tính duy nhất của chúng trên dữ liệu mẫu.

## III. Lọc Dữ Liệu Với Mệnh Đề `WHERE`

Cho đến nay, các truy vấn của chúng ta đều trả về tất cả các hàng từ một bảng. Tuy nhiên, trong phân tích thực tế, bạn thường chỉ quan tâm đến một tập hợp con các hàng đáp ứng một tiêu chí cụ thể. Mệnh đề `WHERE` là công cụ cốt lõi để lọc các hàng dữ liệu, chỉ giữ lại những hàng mà điều kiện được chỉ định là `TRUE`.

### 1. Thứ Tự Thực Thi Logic của Truy Vấn SQL (Under the Hood)

Để hiểu cách `WHERE` hoạt động và tối ưu hóa truy vấn, điều quan trọng là phải nắm được thứ tự thực thi *logic* của các mệnh đề trong một câu lệnh `SELECT` cơ bản. Mặc dù bạn viết `SELECT` trước `FROM` và `WHERE`, nhưng cơ sở dữ liệu xử lý chúng theo một trình tự khác:

1.  **`FROM`**: Đầu tiên, PostgreSQL xác định nguồn dữ liệu (các bảng) mà bạn đang truy vấn. Nó thu thập tất cả các hàng từ bảng được chỉ định, hoặc tạo ra một tập hợp kết quả tạm thời nếu có nhiều bảng tham gia (qua `JOIN` - sẽ học sau).
2.  **`WHERE`**: Tiếp theo, PostgreSQL áp dụng các điều kiện lọc trong mệnh đề `WHERE` cho *tất cả* các hàng đã lấy từ `FROM`. Chỉ những hàng nào mà điều kiện `WHERE` trả về `TRUE` mới được giữ lại để xử lý tiếp. Các hàng trả về `FALSE` hoặc `NULL` sẽ bị loại bỏ.
3.  **`SELECT`**: Cuối cùng, đối với các hàng còn lại sau khi lọc, PostgreSQL chọn các cột được chỉ định trong mệnh đề `SELECT` và thực hiện bất kỳ phép tính hoặc thao tác chuỗi nào được định nghĩa trong đó, bao gồm cả việc tạo cột tính toán bằng `AS`.

> [!CAUTION]
> **Hệ quả của Thứ tự Thực thi:**
> Việc hiểu thứ tự này rất quan trọng. Ví dụ, bạn **không thể tham chiếu một cột được tính toán bằng `AS` trong mệnh đề `WHERE` của cùng một câu lệnh `SELECT`**, vì `WHERE` được thực thi *trước khi* các cột tính toán trong `SELECT` được tạo ra.
> ```sql
> -- LỖI: Không thể sử dụng alias 'density' trong WHERE
> SELECT
>     name,
>     population,
>     area,
>     CAST(population AS NUMERIC) / area AS density
> FROM
>     cities
> WHERE
>     density > 5000;
> ```
> Để lọc dựa trên một cột tính toán, bạn phải lặp lại biểu thức tính toán trong `WHERE` hoặc sử dụng một subquery/CTE (chúng ta sẽ học sau).

### 2. Các Toán Tử So Sánh Cơ Bản

Mệnh đề `WHERE` sử dụng các toán tử so sánh để đánh giá các điều kiện. Các toán tử này trả về `TRUE`, `FALSE`, hoặc `NULL`.

| Toán tử | Mô tả                 | Ví dụ                    |
| :------ | :-------------------- | :----------------------- |
| `=`     | Bằng                  | `area = 8223`            |
| `!=`    | Không bằng (hoặc `<>`) | `area != 8223`           |
| `<>`    | Không bằng (hoặc `!=`) | `area <> 8223`           |
| `>`     | Lớn hơn               | `area > 4000`            |
| `<`     | Nhỏ hơn               | `population < 10000000`  |
| `>=`    | Lớn hơn hoặc bằng     | `population >= 20000000` |
| `<=`    | Nhỏ hơn hoặc bằng     | `area <= 3000`           |

#### Ví dụ: Lọc Thành phố có Diện tích Lớn hơn 4000 km²

```sql
SELECT
    name,
    area
FROM
    cities
WHERE
    area > 4000; -- Chỉ chọn các thành phố có diện tích lớn hơn 4000 km²
```

#### Ví dụ: Lọc Thành phố có Diện tích Cụ thể

```sql
SELECT
    name,
    area
FROM
    cities
WHERE
    area = 8223; -- Chỉ chọn thành phố có diện tích chính xác là 8223 km²
```

### 3. Lọc Nâng Cao: `BETWEEN`, `IN`, `NOT IN`, `LIKE`, `ILIKE`

Ngoài các toán tử so sánh cơ bản, `WHERE` còn cung cấp các toán tử mạnh mẽ hơn để lọc dữ liệu theo phạm vi, danh sách các giá trị hoặc theo mẫu chuỗi.

#### a. Lọc theo Phạm vi với `BETWEEN`

Toán tử `BETWEEN` cho phép bạn kiểm tra xem một giá trị có nằm trong một phạm vi xác định hay không. Phạm vi này là *bao gồm cả* giới hạn dưới và giới hạn trên.

```sql
SELECT
    name,
    area
FROM
    cities
WHERE
    area BETWEEN 2000 AND 4000; -- Chọn các thành phố có diện tích từ 2000 đến 4000 (bao gồm)
```

**Giải thích:** Truy vấn này tương đương với `area >= 2000 AND area <= 4000`.

#### b. Lọc theo Danh sách với `IN` và `NOT IN`

*   `IN`: Kiểm tra xem một giá trị có nằm trong một danh sách các giá trị được cung cấp hay không.
*   `NOT IN`: Kiểm tra xem một giá trị có KHÔNG nằm trong một danh sách các giá trị được cung cấp hay không.

```sql
SELECT
    name,
    country
FROM
    cities
WHERE
    name IN ('Delhi', 'Shanghai', 'Tokyo'); -- Chọn các thành phố có tên là Delhi HOẶC Shanghai HOẶC Tokyo
```

```sql
SELECT
    name,
    country
FROM
    cities
WHERE
    name NOT IN ('Delhi', 'Shanghai'); -- Chọn các thành phố có tên KHÔNG PHẢI là Delhi HOẶC Shanghai
```

Bạn có thể sử dụng `IN` và `NOT IN` với cả chuỗi, số, và các kiểu dữ liệu khác.

#### c. Lọc theo Mẫu Chuỗi với `LIKE` và `ILIKE`

Khi bạn cần tìm kiếm các chuỗi phù hợp với một mẫu nhất định thay vì các giá trị chính xác, `LIKE` và `ILIKE` là các toán tử lý tưởng.

*   `LIKE`: So khớp mẫu chuỗi có phân biệt chữ hoa/thường.
*   `ILIKE`: So khớp mẫu chuỗi KHÔNG phân biệt chữ hoa/thường (đặc trưng của PostgreSQL, rất hữu ích).

Các ký tự đại diện (wildcards) được sử dụng:
*   `%`: Đại diện cho bất kỳ chuỗi ký tự nào, bao gồm cả chuỗi rỗng.
*   `_`: Đại diện cho một ký tự đơn bất kỳ.

```sql
SELECT
    name,
    country
FROM
    cities
WHERE
    name LIKE 'T%'; -- Tìm các thành phố có tên bắt đầu bằng 'T' (ví dụ: 'Tokyo', 'Toronto')
```

```sql
SELECT
    name,
    country
FROM
    cities
WHERE
    name ILIKE '%go%'; -- Tìm các thành phố có tên chứa chuỗi 'go' (không phân biệt hoa thường, ví dụ: 'Tokyo', 'Chicago')
```

```sql
SELECT
    product_code
FROM
    products
WHERE
    product_code LIKE 'A_C%'; -- Tìm các mã sản phẩm bắt đầu bằng 'A', có ký tự bất kỳ ở vị trí thứ 2, và 'C' ở vị tự thứ 3, sau đó là bất kỳ chuỗi nào.
```

#### d. Lọc Giá trị `NULL` với `IS NULL` và `IS NOT NULL`

Khi làm việc với các cột có thể chứa giá trị `NULL` (không có giá trị), bạn không thể sử dụng toán tử `=` hoặc `!=` để kiểm tra `NULL`. Thay vào đó, bạn phải sử dụng `IS NULL` hoặc `IS NOT NULL`.

```sql
SELECT
    customer_id,
    email
FROM
    customers
WHERE
    email IS NULL; -- Chọn khách hàng không có địa chỉ email
```

```sql
SELECT
    customer_id,
    phone_number
FROM
    customers
WHERE
    phone_number IS NOT NULL; -- Chọn khách hàng có số điện thoại
```

### 4. Kết Hợp Các Điều Kiện Lọc với `AND`, `OR`, và `NOT`

Để xây dựng các điều kiện lọc phức tạp hơn, bạn có thể kết hợp nhiều điều kiện bằng các toán tử logic:

*   **`AND`**: Cả hai điều kiện phải `TRUE` để hàng được chọn.
*   **`OR`**: Một trong hai (hoặc cả hai) điều kiện phải `TRUE` để hàng được chọn.
*   **`NOT`**: Đảo ngược kết quả của một điều kiện (từ `TRUE` thành `FALSE`, và ngược lại).

```sql
SELECT
    name,
    population,
    area
FROM
    cities
WHERE
    area > 4000 AND population > 20000000; -- Chọn thành phố có diện tích > 4000 VÀ dân số > 20 triệu
```

```sql
SELECT
    name,
    country
FROM
    cities
WHERE
    country = 'Japan' OR country = 'India'; -- Chọn thành phố ở Nhật Bản HOẶC Ấn Độ
```

Bạn có thể kết hợp nhiều toán tử `AND`, `OR`, `NOT` và sử dụng dấu ngoặc đơn `()` để kiểm soát thứ tự ưu tiên của các phép đánh giá điều kiện.

> [!TIP]
> **Thứ tự ưu tiên của toán tử logic:**
> Khi sử dụng `AND` và `OR` cùng nhau, `AND` có độ ưu tiên cao hơn `OR`. Điều này có nghĩa là các điều kiện được nối bởi `AND` sẽ được đánh giá trước.
>
> Ví dụ: `A OR B AND C` sẽ được đánh giá là `A OR (B AND C)`.
>
> Luôn sử dụng dấu ngoặc đơn `()` để nhóm các điều kiện và làm rõ ý định của bạn, tránh nhầm lẫn về thứ tự đánh giá và đảm bảo logic lọc chính xác.
>
> ```sql
> SELECT
>     name,
>     population,
>     area,
>     country
> FROM
>     cities
> WHERE
>     (country = 'Japan' OR country = 'India') -- Điều kiện 1: Ở Nhật Bản HOẶC Ấn Độ
>     AND population > 10000000;               -- VÀ Điều kiện 2: Dân số > 10 triệu
> ```
> Truy vấn này sẽ trả về các thành phố ở Nhật Bản hoặc Ấn Độ *và* có dân số trên 10 triệu. Nếu không có dấu ngoặc đơn, `AND` sẽ được đánh giá trước, dẫn đến logic khác: `country = 'Japan' OR (country = 'India' AND population > 10000000)`.

### 5. Vibe Coding & Antigravity IDE: Ứng Dụng với Lọc Dữ Liệu

Antigravity IDE giúp bạn "vibe" và xây dựng các điều kiện lọc phức tạp một cách hiệu quả:

*   **Vibe Coding:** Bạn "vibe" rằng "tôi chỉ cần dữ liệu từ năm ngoái," "tôi muốn thấy khách hàng VIP," "tôi cần loại bỏ các bản ghi bị thiếu thông tin." Antigravity sẽ giúp bạn chuyển những "vibe" này thành SQL.
*   **Gợi ý toán tử và giá trị:** Khi bạn nhập `WHERE column_name`, Antigravity sẽ gợi ý các toán tử so sánh (`=`, `>`, `LIKE`, `IS NULL`) và thậm chí đề xuất các giá trị phổ biến trong cột đó (ví dụ: `WHERE country = 'Japan'`, nếu 'Japan' là một giá trị thường xuyên).
*   **Trình tạo điều kiện `IN` và `BETWEEN`:** Đối với `IN`, Antigravity có thể cho phép bạn dán một danh sách các giá trị (từ Excel hoặc một file khác) và nó sẽ tự động định dạng thành `('value1', 'value2', ...)`. Đối với `BETWEEN`, nó có thể cung cấp một giao diện nhập liệu hai trường `start` và `end`.
*   **Tự động nhận diện `NULL`:** Nếu bạn cố gắng sử dụng `WHERE email = NULL`, Antigravity sẽ ngay lập tức cảnh báo lỗi và gợi ý chuyển sang `WHERE email IS NULL`, giải thích lý do.
*   **Trình xây dựng điều kiện logic:** Đối với các điều kiện kết hợp với `AND`, `OR`, `NOT`, Antigravity có thể cung cấp một giao diện kéo thả hoặc hướng dẫn theo từng bước để xây dựng logic, tự động thêm dấu ngoặc đơn để đảm bảo thứ tự ưu tiên chính xác.
*   **Xem trước hiệu quả lọc:** Antigravity có thể hiển thị số lượng hàng sẽ được trả về *sau khi* áp dụng mệnh đề `WHERE` mà không cần chạy toàn bộ truy vấn, giúp bạn nhanh chóng đánh giá hiệu quả của bộ lọc.
*   **Đánh giá hiệu suất:** Nếu một mệnh đề `WHERE` quá phức tạp hoặc có thể không hiệu quả, Antigravity (với khả năng chạy "script ngầm" và phân tích) có thể gợi ý các cách tối ưu hóa, như thêm index vào các cột được lọc (một chủ đề nâng cao hơn).

## Tóm Tắt Phần

Phần này đã trang bị cho bạn những công cụ cơ bản nhưng mạnh mẽ để biến đổi và chọn lọc dữ liệu trong PostgreSQL:

*   **Cột Tính Toán (Calculated Columns)**: Cho phép bạn tạo các cột dữ liệu mới trong tập kết quả bằng cách áp dụng các phép toán số học hoặc hàm lên các cột hiện có.
    *   **Toán Tử Số Học**: Bao gồm `+`, `-`, `*`, `/`, `%`, `^`, `|/`, `!!`. Cần đặc biệt lưu ý về phép chia số nguyên và giới hạn kiểu dữ liệu (`INTEGER` overflow), sử dụng `CAST(column AS NUMERIC/BIGINT)` để đảm bảo độ chính xác và tránh lỗi.
    *   **Đổi Tên Cột (`AS`)**: Sử dụng để gán tên có ý nghĩa cho các cột tính toán, giúp truy vấn dễ đọc và chuyên nghiệp hơn.
*   **Toán Tử và Hàm Chuỗi**:
    *   **Nối Chuỗi**: Sử dụng toán tử `||` (lưu ý `NULL` propagation), hàm `CONCAT()` (bỏ qua `NULL`), hoặc `CONCAT_WS()` (nối với dấu phân tách và bỏ qua `NULL`).
    *   **Hàm Biến Đổi**: `UPPER()` (chữ hoa), `LOWER()` (chữ thường), `LENGTH()` (độ dài), `TRIM()` (loại bỏ khoảng trắng), `SUBSTRING()` (trích xuất chuỗi con).
    *   **Kết Hợp Hàm**: Các hàm có thể được lồng vào nhau để thực hiện các biến đổi chuỗi phức tạp, thực thi từ trong ra ngoài.
*   **Lọc Dữ Liệu (`WHERE`)**: Mệnh đề `WHERE` được sử dụng để lọc các hàng, chỉ trả về những hàng thỏa mãn một hoặc nhiều điều kiện.
    *   **Thứ Tự Thực Thi Logic**: `FROM` -> `WHERE` -> `SELECT` là thứ tự mà PostgreSQL xử lý các mệnh đề. Điều này có nghĩa là bạn không thể tham chiếu alias của cột tính toán trong cùng một mệnh đề `WHERE`.
    *   **Toán Tử So Sánh**: Bao gồm `=`, `!=` (`<>`), `>`, `<`, `>=`, `<=`.
    *   **Lọc Phạm vi và Danh sách**:
        *   `BETWEEN`: Lọc các giá trị trong một phạm vi (bao gồm cả điểm cuối).
        *   `IN` / `NOT IN`: Lọc các giá trị có/không nằm trong một danh sách cụ thể.
    *   **Lọc Mẫu Chuỗi**: `LIKE` (phân biệt chữ hoa/thường) và `ILIKE` (không phân biệt chữ hoa/thường) với các ký tự đại diện `%` và `_`.
    *   **Xử lý `NULL`**: Sử dụng `IS NULL` và `IS NOT NULL` để kiểm tra giá trị `NULL`.
    *   **Kết Hợp Điều Kiện (`AND`, `OR`, `NOT`)**: Sử dụng để xây dựng logic lọc phức tạp. Luôn sử dụng dấu ngoặc đơn `()` để quản lý độ ưu tiên và làm rõ logic, vì `AND` có ưu tiên cao hơn `OR`.

Bằng cách thành thạo các kỹ thuật này, bạn có thể biến dữ liệu thô thành thông tin có tổ chức, dễ hiểu và sẵn sàng cho các phân tích sâu hơn.

<!-- REVIEWED_BY_AGENT -->
