# Phần 3: Nâng Cao về Lọc Dữ Liệu và Phép Tính trong PostgreSQL

Phần 3 của khóa học này sẽ đưa bạn đi sâu vào các kỹ thuật lọc dữ liệu và thực hiện các phép tính trực tiếp trong môi trường PostgreSQL. Chúng ta sẽ không chỉ dừng lại ở việc lọc dữ liệu dựa trên các điều kiện đơn giản mà còn khám phá cách kết hợp nhiều điều kiện phức tạp, sử dụng các toán tử nâng cao, và đặc biệt là cách thực hiện các phép tính để tạo ra dữ liệu mới hoặc sử dụng kết quả của phép tính đó để tinh chỉnh tập dữ liệu. Mục tiêu là trang bị cho bạn các công cụ mạnh mẽ nhất của PostgreSQL để truy vấn, phân tích và thao tác dữ liệu một cách hiệu quả, chính xác và linh hoạt.

## I. Mệnh Đề `WHERE`: Nền Tảng của Việc Lọc Dữ Liệu Có Điều Kiện

Mệnh đề `WHERE` là một thành phần cốt lõi trong các câu lệnh `SELECT`, `UPDATE`, và `DELETE` của SQL, cho phép chúng ta chỉ định các tiêu chí cụ thể để lọc bỏ các hàng dữ liệu không mong muốn. Thay vì xử lý toàn bộ tập dữ liệu, `WHERE` giúp chúng ta tập trung vào một tập con dữ liệu đã được tinh lọc, đáp ứng chính xác các yêu cầu nghiệp vụ.

### 1. Các Toán Tử So Sánh Cơ Bản

Các toán tử so sánh là phương pháp trực tiếp nhất để bắt đầu quá trình lọc dữ liệu. Chúng cho phép bạn đánh giá mối quan hệ giữa giá trị của một cột với một giá trị cố định, một biểu thức, hoặc giá trị của một cột khác.

| Toán tử | Mô tả                       | Ví dụ               |
| :------ | :-------------------------- | :------------------ |
| `=`     | Bằng                        | `price = 100`       |
| `!=`    | Không bằng (cũng có thể dùng `<>`) | `status != 'pending'` |
| `<`     | Nhỏ hơn                     | `age < 18`          |
| `>`     | Lớn hơn                     | `quantity > 0`      |
| `<=`    | Nhỏ hơn hoặc bằng           | `score <= 10`       |
| `>=`    | Lớn hơn hoặc bằng           | `salary >= 50000`   |

> [!NOTE]
> Trong PostgreSQL, tất cả các chuỗi ký tự (string literals) và giá trị ngày/giờ (date/time literals) luôn phải được bao quanh bởi dấu nháy đơn (`' '`).

**Ví dụ 1: Lọc điện thoại bán được hơn 5000 chiếc**

Giả sử chúng ta có một bảng `phones` với các cột như `name` (tên sản phẩm), `price` (giá bán), và `units_sold` (số đơn vị đã bán). Chúng ta cần tìm tên và giá của những chiếc điện thoại có số lượng bán ra vượt quá 5000.

```sql
SELECT
    name,        -- Chọn cột tên sản phẩm
    price        -- Chọn cột giá bán
FROM
    phones       -- Từ bảng phones
WHERE
    units_sold > 5000; -- Chỉ trả về các hàng có units_sold lớn hơn 5000
```

Trong truy vấn này, `units_sold > 5000` là điều kiện lọc. PostgreSQL sẽ quét qua bảng `phones`, đánh giá điều kiện này cho từng hàng và chỉ giữ lại những hàng nào mà điều kiện cho kết quả là `TRUE`.

### 2. Lọc Dữ Liệu với Giá Trị `NULL`

`NULL` trong SQL đại diện cho một giá trị không xác định, không tồn tại hoặc không áp dụng được. Điều quan trọng là `NULL` không tương đương với 0 hoặc một chuỗi rỗng (`''`). Do đó, bạn không thể sử dụng các toán tử so sánh thông thường (`=`, `!=`) để kiểm tra `NULL`. Thay vào đó, PostgreSQL cung cấp các toán tử chuyên biệt:

*   `IS NULL`: Kiểm tra xem một cột có giá trị là `NULL` hay không.
*   `IS NOT NULL`: Kiểm tra xem một cột có giá trị không phải là `NULL` hay không.

**Ví dụ 2: Tìm nhân viên chưa có địa chỉ email**

Giả sử bảng `employees` có cột `email`. Chúng ta muốn tìm tất cả nhân viên chưa được gán địa chỉ email.

```sql
SELECT
    first_name,
    last_name
FROM
    employees
WHERE
    email IS NULL; -- Lọc các nhân viên có email là NULL
```

### 3. Kết Hợp Nhiều Điều Kiện với Toán Tử Logic

Trong thực tế, việc lọc dữ liệu thường đòi hỏi nhiều hơn một điều kiện. PostgreSQL cung cấp các toán tử logic để kết hợp các điều kiện lọc, cho phép xây dựng các quy tắc phức tạp:

*   **`AND`**: Yêu cầu *tất cả* các điều kiện liên kết phải đúng để một hàng được chọn.
*   **`OR`**: Yêu cầu *ít nhất một* trong các điều kiện liên kết phải đúng để một hàng được chọn.
*   **`NOT`**: Đảo ngược kết quả của một điều kiện (từ `TRUE` thành `FALSE`, và ngược lại).

**Thứ Tự Ưu Tiên của Toán Tử Logic:**
Trong PostgreSQL, thứ tự ưu tiên của các toán tử logic là: `NOT` > `AND` > `OR`. Luôn sử dụng dấu ngoặc đơn `()` để nhóm các điều kiện và đảm bảo thứ tự đánh giá theo ý muốn của bạn, tránh các lỗi logic không mong muốn.

**Ví dụ 3: Lọc điện thoại theo nhà sản xuất (Apple HOẶC Samsung) và giá dưới 1000**

Chúng ta muốn tìm tên và nhà sản xuất của tất cả các điện thoại do Apple hoặc Samsung sản xuất, *và* có giá dưới 1000 đơn vị tiền tệ.

```sql
SELECT
    name,
    manufacturer,
    price
FROM
    phones
WHERE
    (manufacturer = 'Apple' OR manufacturer = 'Samsung') AND price < 1000;
    -- Dấu ngoặc đơn là cần thiết để đảm bảo (Apple HOẶC Samsung) được đánh giá trước
```

Nếu không có dấu ngoặc đơn, `AND price < 1000` sẽ được ưu tiên liên kết với `manufacturer = 'Samsung'`, dẫn đến kết quả sai lệch.

### 4. Các Toán Tử Lọc Nâng Cao

PostgreSQL cung cấp một loạt các toán tử để xử lý các tình huống lọc phức tạp hơn một cách hiệu quả và dễ đọc.

#### a. `IN` và `NOT IN`: Kiểm tra thành viên trong danh sách

Khi bạn cần kiểm tra xem giá trị của một cột có nằm trong một danh sách các giá trị cụ thể hay không, toán tử `IN` là lựa chọn tối ưu. Ngược lại, `NOT IN` kiểm tra xem giá trị có *không* nằm trong danh sách đó.

**Ví dụ 4: Lọc điện thoại từ Apple, Samsung hoặc Google**

```sql
SELECT
    name,
    manufacturer
FROM
    phones
WHERE
    manufacturer IN ('Apple', 'Samsung', 'Google'); -- Lấy điện thoại từ các nhà sản xuất trong danh sách
```

**Ví dụ 5: Lọc điện thoại KHÔNG phải từ Apple hoặc Samsung**

```sql
SELECT
    name,
    manufacturer
FROM
    phones
WHERE
    manufacturer NOT IN ('Apple', 'Samsung'); -- Lấy điện thoại từ các nhà sản xuất KHÔNG trong danh sách
```

> [!TIP]
> Toán tử `IN` thường dễ đọc và bảo trì hơn nhiều so với việc sử dụng chuỗi `OR` dài khi bạn có một danh sách lớn các giá trị cần so sánh.

#### b. `BETWEEN` và `NOT BETWEEN`: Lọc theo khoảng giá trị

Toán tử `BETWEEN` được sử dụng để lọc dữ liệu trong một phạm vi giá trị nhất định (bao gồm cả hai giá trị biên). `NOT BETWEEN` sẽ lọc các giá trị nằm ngoài phạm vi đó.

**Ví dụ 6: Tìm sản phẩm có giá từ 500 đến 1000**

```sql
SELECT
    name,
    price
FROM
    products
WHERE
    price BETWEEN 500 AND 1000; -- Giá từ 500 đến 1000 (bao gồm 500 và 1000)
```

#### c. `LIKE` và `ILIKE`: Tìm kiếm mẫu chuỗi ký tự

Để tìm kiếm các chuỗi ký tự theo một mẫu nhất định, PostgreSQL cung cấp toán tử `LIKE`. Đối với các trường hợp cần tìm kiếm không phân biệt chữ hoa/chữ thường, PostgreSQL có thêm toán tử `ILIKE` (đặc trưng của PostgreSQL).

*   `%`: Đại diện cho bất kỳ chuỗi ký tự nào (bao gồm chuỗi rỗng).
*   `_`: Đại diện cho bất kỳ một ký tự đơn nào.

**Ví dụ 7: Tìm kiếm sản phẩm có tên bắt đầu bằng 'Sam' (phân biệt chữ hoa/thường)**

```sql
SELECT
    name
FROM
    products
WHERE
    name LIKE 'Sam%'; -- Tìm tên sản phẩm bắt đầu bằng 'Sam'
```

**Ví dụ 8: Tìm kiếm sản phẩm có tên chứa 'phone' (không phân biệt chữ hoa/thường)**

```sql
SELECT
    name
FROM
    products
WHERE
    name ILIKE '%phone%'; -- Tìm tên sản phẩm chứa 'phone', không phân biệt chữ hoa/thường (ví dụ: 'Smartphone', 'headphone')
```

#### d. `SIMILAR TO` và Biểu Thức Chính Quy (Regular Expressions)

Đối với các mẫu tìm kiếm phức tạp hơn, PostgreSQL hỗ trợ `SIMILAR TO` (một phần của chuẩn SQL) và mạnh mẽ hơn là các toán tử biểu thức chính quy (`~` và `~*`).

*   `SIMILAR TO`: Cung cấp cú pháp mạnh hơn `LIKE` với một số ký tự đặc biệt như `|` (hoặc), `*` (không hoặc nhiều lần), `+` (một hoặc nhiều lần), `?` (không hoặc một lần).
*   `~`: Toán tử khớp biểu thức chính quy, phân biệt chữ hoa/thường.
*   `~*`: Toán tử khớp biểu thức chính quy, không phân biệt chữ hoa/thường.

**Ví dụ 9: Tìm kiếm tên sản phẩm bắt đầu bằng 'A' hoặc 'B' (dùng SIMILAR TO)**

```sql
SELECT
    name
FROM
    products
WHERE
    name SIMILAR TO '(A|B)%'; -- Tên sản phẩm bắt đầu bằng 'A' hoặc 'B'
```

**Ví dụ 10: Tìm kiếm tên sản phẩm chứa ít nhất một chữ số (dùng Biểu thức Chính quy)**

```sql
SELECT
    name
FROM
    products
WHERE
    name ~ '\d+'; -- Tên sản phẩm chứa ít nhất một chữ số (phân biệt chữ hoa/thường)
```

## II. Thực Hiện Các Phép Tính Trực Tiếp Trong Truy Vấn SQL

SQL không chỉ là ngôn ngữ truy xuất dữ liệu mà còn là một công cụ mạnh mẽ để thực hiện các phép tính số học, chuỗi và ngày/giờ trực tiếp trên các cột hoặc giá trị hằng. Khả năng này cho phép bạn tạo ra các cột dữ liệu mới, được tính toán động, mà không cần phải lưu trữ chúng vật lý trong cơ sở dữ liệu.

### 1. Các Toán Tử Số Học Cơ Bản và Xử Lý Kiểu Dữ Liệu

PostgreSQL hỗ trợ các toán tử số học tiêu chuẩn:

*   `+`: Cộng
*   `-`: Trừ
*   `*`: Nhân
*   `/`: Chia
*   `%`: Modulo (chia lấy phần dư)
*   `^`: Lũy thừa (PostgreSQL đặc trưng)

**Xử lý Kiểu Dữ liệu (Type Coercion/Casting):**
PostgreSQL có cơ chế tự động chuyển đổi kiểu dữ liệu (implicit type coercion) trong nhiều trường hợp. Tuy nhiên, để đảm bảo tính chính xác và tránh các lỗi không mong muốn (ví dụ: chia số nguyên), bạn nên sử dụng chuyển đổi kiểu tường minh (explicit type casting).

*   **Chia số nguyên:** Khi chia hai số nguyên, PostgreSQL sẽ trả về kết quả là số nguyên (phần nguyên), bỏ qua phần thập phân.
    *   Ví dụ: `SELECT 5 / 2;` sẽ trả về `2`.
*   **Chia số thực:** Để có kết quả thập phân, ít nhất một trong các toán hạng phải là kiểu số thực (ví dụ: `NUMERIC`, `REAL`, `DOUBLE PRECISION`).
    *   Ví dụ: `SELECT 5.0 / 2;` hoặc `SELECT 5 / 2.0;` sẽ trả về `2.5`.
    *   Bạn có thể ép kiểu tường minh bằng cú pháp `::` hoặc `CAST()`: `SELECT 5::numeric / 2;` hoặc `SELECT CAST(5 AS numeric) / 2;`.

### 2. Tạo Cột Tính Toán (Computed Columns) trong Mệnh Đề `SELECT`

Bạn có thể thực hiện các phép tính trên một hoặc nhiều cột và hiển thị kết quả như một cột mới trong tập kết quả của bạn. Để đặt tên cho cột mới này, chúng ta sử dụng từ khóa `AS` (alias).

**Ví dụ 11: Tính mật độ dân số của các thành phố**

Giả sử chúng ta có bảng `cities` với các cột `name` (tên thành phố), `population` (dân số) và `area` (diện tích). Chúng ta muốn tính mật độ dân số cho mỗi thành phố.

```sql
SELECT
    name,                             -- Chọn cột tên
    population,                       -- Chọn cột dân số
    area,                             -- Chọn cột diện tích
    (population::numeric / area) AS density -- Tính mật độ dân số (ép kiểu để có kết quả thập phân) và đặt tên là 'density'
FROM
    cities;                           -- Từ bảng cities
```

Kết quả sẽ bao gồm một cột mới có tên `density` chứa giá trị mật độ dân số được tính toán, cho phép bạn dễ dàng phân tích hoặc xuất báo cáo.

### 3. Thứ Tự Ưu Tiên của Các Phép Toán

Giống như trong toán học thông thường, các phép toán trong SQL cũng tuân theo một thứ tự ưu tiên nhất định:

1.  **Dấu ngoặc đơn `()`**: Các phép toán trong dấu ngoặc đơn luôn được thực hiện trước.
2.  **Lũy thừa (`^`)**: Thực hiện sau dấu ngoặc.
3.  **Nhân (`*`), Chia (`/`), Modulo (`%`)**: Thực hiện từ trái sang phải.
4.  **Cộng (`+`), Trừ (`-`)**: Thực hiện từ trái sang phải.

> [!NOTE]
> Luôn sử dụng dấu ngoặc đơn để đảm bảo thứ tự các phép toán theo ý muốn của bạn, đặc biệt khi có sự kết hợp của nhiều toán tử. Điều này giúp mã của bạn dễ đọc, dễ hiểu và tránh lỗi logic.

## III. Lọc Dữ Liệu Dựa Trên Các Giá Trị Được Tính Toán

Một trong những khả năng mạnh mẽ của SQL là khả năng lọc dữ liệu không chỉ dựa trên các cột hiện có mà còn dựa trên các giá trị được tính toán. Điều này cho phép bạn tạo ra các điều kiện lọc rất linh hoạt và phân tích sâu hơn.

### 1. Áp Dụng Phép Tính Trực Tiếp trong Mệnh Đề `WHERE`

Bạn có thể đưa các biểu thức tính toán vào mệnh đề `WHERE` để lọc các hàng thỏa mãn điều kiện dựa trên kết quả của phép tính đó.

**Ví dụ 12: Lọc thành phố có mật độ dân số lớn hơn 6000**

Tiếp tục với ví dụ về bảng `cities`, chúng ta muốn tìm tên của tất cả các thành phố có mật độ dân số lớn hơn 6000.

```sql
SELECT
    name,
    (population::numeric / area) AS density -- Tính mật độ dân số
FROM
    cities
WHERE
    (population::numeric / area) > 6000; -- Lọc những thành phố có mật độ dân số > 6000
```

Trong ví dụ này, biểu thức `(population::numeric / area)` được tính toán cho mỗi hàng *trước khi* điều kiện `> 6000` được kiểm tra. Điều này đảm bảo rằng phép tính được thực hiện đúng cách để xác định các hàng cần lọc.

### 2. Hiểu Rõ Thứ Tự Thực Thi Logic của Truy Vấn SQL (Under the Hood)

Để sử dụng SQL một cách hiệu quả và tránh các lỗi phổ biến, việc hiểu rõ thứ tự logic mà PostgreSQL xử lý một truy vấn là cực kỳ quan trọng. Các mệnh đề trong một câu lệnh `SELECT` không được thực thi theo thứ tự chúng xuất hiện trong mã. Thay vào đó, chúng tuân theo một quy trình logic như sau:

1.  **`FROM`**: Xác định (các) bảng nguồn mà từ đó dữ liệu sẽ được lấy. Nếu có nhiều bảng, các phép `JOIN` sẽ được thực hiện tại đây.
2.  **`WHERE`**: Lọc các hàng dựa trên các điều kiện đã cho. Chỉ những hàng thỏa mãn điều kiện mới được chuyển sang bước tiếp theo.
3.  **`GROUP BY`** (sẽ học trong phần sau): Nhóm các hàng thành các nhóm dựa trên giá trị của một hoặc nhiều cột.
4.  **`HAVING`** (sẽ học trong phần sau): Lọc các *nhóm* được tạo bởi `GROUP BY` dựa trên các điều kiện tổng hợp.
5.  **`SELECT`**: Chọn các cột và biểu thức cuối cùng để hiển thị. Tại đây, các cột tính toán được tạo ra và các bí danh (aliases) được định nghĩa.
6.  **`DISTINCT`** (nếu có): Loại bỏ các hàng trùng lặp từ tập kết quả.
7.  **`ORDER BY`**: Sắp xếp tập kết quả theo một hoặc nhiều cột.
8.  **`LIMIT` / `OFFSET`**: Giới hạn số lượng hàng trả về hoặc bỏ qua một số hàng nhất định.

**Ý nghĩa quan trọng:** Mệnh đề `WHERE` được xử lý *trước* mệnh đề `SELECT`. Điều này có nghĩa là bất kỳ bí danh nào bạn định nghĩa trong mệnh đề `SELECT` (ví dụ: `AS density` hoặc `AS total_revenue`) sẽ **chưa tồn tại** khi mệnh đề `WHERE` được đánh giá.

### 3. Vấn Đề với Bí Danh (Aliases) trong `WHERE` và Giải Pháp

Do thứ tự thực thi logic đã nêu, bạn **không thể** sử dụng bí danh của một cột tính toán trong mệnh đề `WHERE` của cùng một truy vấn trực tiếp. Bạn phải lặp lại toàn bộ biểu thức tính toán trong mệnh đề `WHERE`.

**Ví dụ 13: Lọc điện thoại có tổng doanh thu lớn hơn 1 triệu**

Chúng ta muốn in ra tên và tổng doanh thu của tất cả các điện thoại có tổng doanh thu lớn hơn 1 triệu. Tổng doanh thu được tính bằng `price * units_sold`.

```sql
SELECT
    name,
    (price * units_sold) AS total_revenue -- Tính tổng doanh thu và đặt tên là 'total_revenue'
FROM
    phones
WHERE
    (price * units_sold) > 1000000;       -- PHẢI lặp lại biểu thức tính toán ở đây
    -- LƯU Ý: KHÔNG THỂ dùng: WHERE total_revenue > 1000000;
```

> [!CAUTION]
> Cố gắng sử dụng `WHERE total_revenue > 1000000` sẽ gây ra lỗi `column "total_revenue" does not exist`, vì `total_revenue` chỉ được định nghĩa và có sẵn sau khi `WHERE` đã được xử lý. Đây là một lỗi phổ biến mà người mới học SQL hay mắc phải.

**Giải pháp Nâng cao: Sử dụng Common Table Expressions (CTEs) hoặc Subqueries**

Để tránh lặp lại biểu thức tính toán và làm cho truy vấn dễ đọc hơn, đặc biệt với các biểu thức phức tạp, bạn có thể sử dụng Common Table Expressions (CTEs) hoặc Subqueries. Các kỹ thuật này cho phép bạn định nghĩa một tập kết quả tạm thời (bao gồm cả các cột tính toán với bí danh) và sau đó truy vấn từ tập kết quả đó.

**Ví dụ 14: Lọc điện thoại có tổng doanh thu lớn hơn 1 triệu (sử dụng CTE)**

```sql
WITH PhoneRevenue AS ( -- Định nghĩa một CTE tên là PhoneRevenue
    SELECT
        name,
        (price * units_sold) AS total_revenue -- Tính toán và đặt bí danh trong CTE
    FROM
        phones
)
SELECT
    name,
    total_revenue
FROM
    PhoneRevenue
WHERE
    total_revenue > 1000000; -- Bây giờ có thể sử dụng bí danh 'total_revenue'
```

Trong ví dụ này, `PhoneRevenue` là một bảng tạm thời được tạo ra với cột `total_revenue` đã được tính toán. Sau đó, truy vấn bên ngoài có thể tham chiếu đến `PhoneRevenue` và sử dụng bí danh `total_revenue` trong mệnh đề `WHERE` một cách hợp lệ. Đây là một kỹ thuật mạnh mẽ giúp tổ chức các truy vấn phức tạp trở nên mạch lạc và dễ quản lý hơn.

## Tóm Tắt Phần 3: Nâng Cao về Lọc Dữ Liệu và Phép Tính

*   **Mệnh đề `WHERE`** là công cụ chính để lọc các hàng dữ liệu dựa trên các điều kiện.
*   Sử dụng **toán tử so sánh** (`=`, `!=`, `<`, `>`, `<=`, `>=`) và **toán tử `IS NULL` / `IS NOT NULL`** để lọc giá trị.
*   **Toán tử logic** (`AND`, `OR`, `NOT`) cho phép kết hợp và phủ định các điều kiện, với thứ tự ưu tiên `NOT` > `AND` > `OR`. Luôn dùng dấu ngoặc đơn để kiểm soát thứ tự thực thi.
*   **Toán tử `IN` / `NOT IN`** cung cấp cách ngắn gọn để kiểm tra thành viên trong danh sách.
*   **Toán tử `BETWEEN` / `NOT BETWEEN`** dùng để lọc dữ liệu trong một khoảng giá trị.
*   **Toán tử `LIKE` / `ILIKE`** và **Biểu thức Chính Quy (`~`, `~*`)** cho phép tìm kiếm mẫu chuỗi ký tự linh hoạt. `ILIKE` và regex là đặc trưng mạnh mẽ của PostgreSQL.
*   **Các phép tính số học** (`+`, `-`, `*`, `/`, `%`, `^`) có thể được thực hiện trực tiếp trong truy vấn. Cần lưu ý về **chuyển đổi kiểu dữ liệu** (casting) để đảm bảo kết quả chính xác, đặc biệt với phép chia.
*   Bạn có thể tạo **cột tính toán mới** trong mệnh đề `SELECT` bằng cách sử dụng các biểu thức và đặt tên cho chúng bằng từ khóa `AS`.
*   Các **biểu thức tính toán** cũng có thể được sử dụng trực tiếp trong **mệnh đề `WHERE`** để lọc dữ liệu dựa trên kết quả của chúng.
*   **Thứ tự thực thi logic của SQL** là rất quan trọng: `WHERE` được xử lý trước `SELECT`. Do đó, **không thể sử dụng bí danh (alias)** được định nghĩa trong mệnh đề `SELECT` trong cùng mệnh đề `WHERE` mà phải lặp lại toàn bộ biểu thức tính toán.
*   Để khắc phục hạn chế của bí danh trong `WHERE`, bạn có thể sử dụng các kỹ thuật nâng cao như **Common Table Expressions (CTEs)** hoặc **Subqueries**.

<!-- REVIEWED_BY_AGENT -->
