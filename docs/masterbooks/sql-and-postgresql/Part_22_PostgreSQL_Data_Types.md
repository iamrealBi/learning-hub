# Bài 22: Các Kiểu Dữ Liệu trong PostgreSQL – Nền Tảng Thiết Kế Cơ Sở Dữ Liệu Hiệu Quả

## 1. Giới Thiệu Tổng Quan về Kiểu Dữ Liệu

Trong lĩnh vực quản lý cơ sở dữ liệu, việc lựa chọn và ứng dụng đúng kiểu dữ liệu là một quyết định chiến lược, có tầm ảnh hưởng sâu rộng đến tính toàn vẹn của thông tin, hiệu suất truy vấn và hiệu quả sử dụng tài nguyên lưu trữ. PostgreSQL, với tư cách là một hệ quản trị cơ sở dữ liệu quan hệ đối tượng hàng đầu, cung cấp một hệ thống kiểu dữ liệu phong phú và linh hoạt, được thiết kế để đáp ứng đa dạng các yêu cầu lưu trữ thông tin.

Chương này sẽ cung cấp một cái nhìn toàn diện và chuyên sâu vào các kiểu dữ liệu cơ bản và được sử dụng rộng rãi nhất trong PostgreSQL. Chúng ta sẽ khám phá từ các kiểu dữ liệu nguyên thủy như số, ký tự và Boolean, cho đến các kiểu phức tạp hơn như ngày, giờ và khoảng thời gian. Mỗi kiểu dữ liệu sẽ được phân tích về cơ chế hoạt động, phạm vi giá trị hỗ trợ, và các quy tắc quan trọng cần tuân thủ khi sử dụng. Mục tiêu là trang bị cho bạn một nền tảng kiến thức vững chắc, giúp bạn đưa ra các lựa chọn kiểu dữ liệu thông minh, từ đó thiết kế các cơ sở dữ liệu tối ưu và hoạt động hiệu quả.

Chúng ta sẽ bắt đầu bằng việc làm rõ tầm quan trọng cốt lõi của kiểu dữ liệu, sau đó đi sâu vào từng danh mục cụ thể, cung cấp các ví dụ minh họa bằng cú pháp SQL chuẩn của PostgreSQL. Đặc biệt, chúng ta sẽ lồng ghép tư duy **Vibe Coding** và cách một hệ thống **Agentic AI** như Antigravity IDE sẽ tiếp cận và đưa ra quyết định về kiểu dữ liệu, giúp bạn không chỉ hiểu *cái gì* mà còn *tại sao* và *làm thế nào* để áp dụng kiến thức này một cách thông minh nhất.

## 2. Tầm Quan Trọng Cốt Lõi của Kiểu Dữ Liệu trong Thiết Kế Cơ Sở Dữ Liệu

Khi xây dựng một lược đồ cơ sở dữ liệu (database schema) trong PostgreSQL, việc gán một kiểu dữ liệu cụ thể cho mỗi cột không chỉ đơn thuần là định nghĩa loại thông tin mà cột đó có thể chứa (ví dụ: số, văn bản, ngày tháng). Nó còn thiết lập một bộ quy tắc chặt chẽ về cách thức dữ liệu được lưu trữ vật lý, cách nó được xử lý trong các phép toán, và thậm chí là cách nó được so sánh hoặc lập chỉ mục (indexed).

> [!NOTE]
> Việc lựa chọn kiểu dữ liệu chính xác là yếu tố then chốt vì những lý do sau:
> *   **Tính Toàn Vẹn Dữ Liệu (Data Integrity):** Kiểu dữ liệu đóng vai trò là lớp bảo vệ đầu tiên, đảm bảo rằng chỉ những dữ liệu hợp lệ và phù hợp với ngữ cảnh mới được lưu trữ trong cột. Ví dụ, một cột khai báo là `INTEGER` sẽ từ chối mọi nỗ lực chèn giá trị văn bản, ngăn chặn lỗi logic và hỏng dữ liệu.
> *   **Hiệu Suất Truy Vấn (Query Performance):** Các kiểu dữ liệu được tối ưu hóa cho các hoạt động cụ thể. Sử dụng kiểu dữ liệu phù hợp (ví dụ: `INTEGER` thay vì `VARCHAR` cho số ID) có thể tăng tốc độ tìm kiếm, sắp xếp, lọc và tính toán, đặc biệt khi kết hợp với việc lập chỉ mục hiệu quả.
> *   **Hiệu Quả Lưu Trữ (Storage Efficiency):** Mỗi kiểu dữ liệu chiếm một lượng không gian lưu trữ nhất định trên đĩa. Việc chọn kiểu dữ liệu nhỏ nhất có thể đáp ứng đầy đủ yêu cầu về phạm vi và độ chính xác của dữ liệu sẽ tối ưu hóa không gian đĩa, giảm chi phí lưu trữ và cải thiện hiệu suất I/O.
> *   **Hành Vi Đặc Biệt và Chức Năng Mở Rộng:** Một số kiểu dữ liệu đi kèm với các hành vi hoặc chức năng đặc biệt. Ví dụ, `SERIAL` tự động tăng giá trị, `INTERVAL` cho phép thực hiện các phép tính khoảng thời gian phức tạp, hoặc các kiểu `JSONB` cho phép lưu trữ dữ liệu phi cấu trúc và truy vấn hiệu quả.

### 2.1. Vibe Coding và Tư Duy Agentic AI trong Lựa Chọn Kiểu Dữ Liệu

Đối với một hệ thống **Agentic AI** như Antigravity IDE, việc lựa chọn kiểu dữ liệu không chỉ là tuân theo các quy tắc cứng nhắc mà còn là một quá trình **Vibe Coding** sâu sắc. Khi một subagent của Antigravity được giao nhiệm vụ thiết kế một lược đồ cơ sở dữ liệu từ yêu cầu nghiệp vụ (ví dụ: "cần lưu trữ thông tin người dùng bao gồm tên, tuổi, email và ngày đăng ký"), nó không chỉ đơn thuần ánh xạ các trường. Thay vào đó, nó sẽ:

1.  **Phân Tích Ngữ Cảnh Dữ Liệu:** Antigravity sẽ phân tích "vibe" của từng loại dữ liệu.
    *   *Tên người dùng:* Có thể là `TEXT` hoặc `VARCHAR(n)`? Nếu có giới hạn hiển thị hoặc validation, `VARCHAR(n)` có thể được ưu tiên. Nếu là văn bản tự do, `TEXT` linh hoạt hơn.
    *   *Tuổi:* Nên là `SMALLINT` hay `INTEGER`? Người dùng có thể có tuổi âm không? Giới hạn tuổi tối đa là bao nhiêu?
    *   *Email:* Cần duy nhất không? Độ dài tối đa? `VARCHAR(255)` là lựa chọn phổ biến, kết hợp với ràng buộc `UNIQUE`.
    *   *Ngày đăng ký:* Cần múi giờ không? `DATE` hay `TIMESTAMP WITH TIME ZONE`? Nếu ứng dụng toàn cầu, `TIMESTAMPTZ` là bắt buộc.
2.  **Đánh Giá Ảnh Hưởng Hiệu Suất và Lưu Trữ:** Dựa trên phân tích ngữ cảnh, Antigravity sẽ cân nhắc các yếu tố hiệu suất và lưu trữ. Ví dụ, cho một cột ID, nó sẽ *vibe* rằng `SERIAL` là tối ưu nhất vì nó tự động quản lý chuỗi số, đảm bảo tính duy nhất và hiệu quả. Đối với tiền tệ, nó sẽ ngay lập tức chọn `NUMERIC` thay vì `DOUBLE PRECISION` để tránh các lỗi làm tròn tiềm ẩn.
3.  **Dự Đoán Hành Vi Tương Lai:** Một agent thông minh sẽ không chỉ nhìn vào hiện tại mà còn dự đoán các yêu cầu tương lai. Liệu trường này có cần được lập chỉ mục thường xuyên? Liệu có các phép toán số học phức tạp trên nó không? Điều này ảnh hưởng đến việc chọn `NUMERIC` cho tính toán tài chính thay vì `FLOAT`.

Bằng cách này, Antigravity không chỉ "code" mà còn "thiết kế" với một sự hiểu biết sâu sắc về các đặc tính của dữ liệu, tối ưu hóa cơ sở dữ liệu ngay từ giai đoạn đầu, giảm thiểu các lỗi phổ biến mà lập trình viên con người thường mắc phải.

### 2.2. Kiểm Tra Kiểu Dữ Liệu Nhanh chóng với Cú Pháp `::`

Để minh họa và thử nghiệm các kiểu dữ liệu mà không cần tạo bảng thực tế, chúng ta có thể sử dụng cú pháp ép kiểu (type cast) `SELECT 'giá_trị'::kiểu_dữ_liệu;` trong công cụ truy vấn của PgAdmin hoặc bất kỳ client PostgreSQL nào. Cú pháp này yêu cầu PostgreSQL chuyển đổi một giá trị chuỗi sang một kiểu dữ liệu cụ thể và hiển thị kết quả. Đây là một công cụ cực kỳ hữu ích cho việc học và gỡ lỗi.

```sql
-- Ví dụ: Chuyển đổi chuỗi '123' thành kiểu số nguyên
SELECT '123'::integer AS integer_value;

-- Kiểm tra một giá trị ngày tháng
SELECT '2023-10-27'::date AS current_date;

-- Kiểm tra chuyển đổi Boolean
SELECT 'yes'::boolean AS is_active;
```

## 3. Các Kiểu Dữ Liệu Số (Numeric Data Types)

Các kiểu dữ liệu số được thiết kế để lưu trữ các giá trị số, bao gồm số nguyên (không có phần thập phân), số thập phân có độ chính xác cố định và số dấu phẩy động (có thể có phần thập phân). Việc lựa chọn kiểu số phù hợp phụ thuộc vào phạm vi giá trị bạn cần lưu trữ, mức độ chính xác yêu cầu và cân nhắc về hiệu suất.

### 3.1. Các Kiểu Số Nguyên (Integers)

Các kiểu số nguyên được dùng để lưu trữ các số nguyên không có phần thập phân. PostgreSQL cung cấp ba loại chính, khác nhau về phạm vi giá trị và không gian lưu trữ:

*   **`SMALLINT`**:
    *   Phạm vi: Từ -32,768 đến +32,767.
    *   Lưu trữ: 2 byte.
    *   Sử dụng khi bạn chắc chắn rằng các giá trị sẽ nằm trong phạm vi nhỏ này, ví dụ: số lượng mặt hàng trong kho nhỏ, mã trạng thái.
*   **`INTEGER`** (hoặc `INT`):
    *   Phạm vi: Từ -2,147,483,648 đến +2,147,483,647.
    *   Lưu trữ: 4 byte.
    *   Đây là lựa chọn phổ biến và mặc định cho hầu hết các số nguyên, cung cấp sự cân bằng tốt giữa phạm vi và hiệu quả lưu trữ.
*   **`BIGINT`**:
    *   Phạm vi: Từ -9,223,372,036,854,775,808 đến +9,223,372,036,854,775,807.
    *   Lưu trữ: 8 byte.
    *   Sử dụng cho các số rất lớn, chẳng hạn như số lượng bản ghi trong một bảng khổng lồ, tổng số lượt xem trang web toàn cầu, hoặc các ID được tạo ra bởi các hệ thống phân tán.

> [!TIP]
> Khi một giá trị có phần thập phân được chuyển đổi sang kiểu số nguyên, phần thập phân sẽ bị cắt bỏ (truncation), không làm tròn. Nếu giá trị nằm ngoài phạm vi của kiểu số nguyên được chỉ định, PostgreSQL sẽ báo lỗi `integer out of range`. PostgreSQL không có kiểu số nguyên không dấu (`UNSIGNED`). Để đảm bảo giá trị luôn dương, bạn cần sử dụng ràng buộc `CHECK`.

**Ví dụ:**

```sql
-- Chuyển đổi số có thập phân sang INTEGER (phần thập phân bị cắt bỏ)
SELECT '2.75'::integer AS truncated_value; -- Kết quả: 2
SELECT '-2.75'::integer AS negative_truncated_value; -- Kết quả: -2

-- Chuyển đổi số nằm trong phạm vi SMALLINT
SELECT 32000::smallint AS small_int_value; -- Kết quả: 32000

-- Chuyển đổi số nằm ngoài phạm vi SMALLINT (sẽ gây lỗi)
-- SELECT 33000::smallint; -- Lỗi: integer out of range
```

### 3.2. Các Kiểu Số Tự Động Tăng (Serial Types)

PostgreSQL cung cấp các kiểu `SERIAL` để tạo ra các số nguyên tự động tăng, thường được sử dụng cho khóa chính (primary key) của bảng. Điều quan trọng cần hiểu là các kiểu `SERIAL` không phải là kiểu dữ liệu độc lập thực sự. Thay vào đó, chúng là **cú pháp tiện lợi (syntactic sugar)** mà PostgreSQL cung cấp để tự động thực hiện ba hành động sau:

1.  **Tạo một đối tượng `SEQUENCE` (chuỗi số):** Một đối tượng sequence là một bộ đếm số độc lập, có khả năng sinh ra các số nguyên tuần tự.
2.  **Đặt giá trị mặc định (DEFAULT) cho cột:** Giá trị mặc định của cột sẽ được lấy từ `nextval()` của sequence vừa tạo.
3.  **Thêm ràng buộc `NOT NULL`:** Đảm bảo cột luôn có giá trị.

Các phiên bản của `SERIAL` tương ứng với các kiểu số nguyên:

*   **`SMALLSERIAL`**: Tương tự `SMALLINT`, tạo sequence với phạm vi `SMALLINT`.
*   **`SERIAL`**: Tương tự `INTEGER`, tạo sequence với phạm vi `INTEGER`. Đây là lựa chọn phổ biến nhất cho ID tự động tăng.
*   **`BIGSERIAL`**: Tương tự `BIGINT`, tạo sequence với phạm vi `BIGINT`.

> [!IMPORTANT]
> **Cơ chế ngầm (Under the Hood):** Khi bạn khai báo một cột là `SERIAL`, PostgreSQL thực chất sẽ thực hiện các lệnh tương đương sau:
> ```sql
> -- Ví dụ: Cho cột product_id SERIAL
> CREATE SEQUENCE products_product_id_seq AS integer; -- Tạo sequence
> CREATE TABLE products (
>     product_id integer NOT NULL DEFAULT nextval('products_product_id_seq'), -- Đặt giá trị mặc định
>     -- ... các cột khác ...
> );
> ALTER SEQUENCE products_product_id_seq OWNED BY products.product_id; -- Liên kết sequence với cột
> ```
> Việc hiểu cơ chế này giúp bạn có thể quản lý sequence thủ công nếu cần (ví dụ: đặt lại giá trị bắt đầu của sequence).

**Ví dụ:**

```sql
-- Tạo bảng với cột ID tự động tăng
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY, -- product_id sẽ tự động tăng
    product_name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2)
);

-- Chèn dữ liệu mà không cần chỉ định product_id
INSERT INTO products (product_name, price) VALUES ('Laptop', 1200.00);
INSERT INTO products (product_name, price) VALUES ('Mouse', 25.50);
INSERT INTO products (product_name, price) VALUES ('Keyboard', 75.00);

-- Xem kết quả
SELECT * FROM products;

-- Khi Antigravity IDE sử dụng Vibe Coding để tạo schema:
-- Nếu Antigravity nhận diện một trường là khóa chính và cần tự động tăng,
-- nó sẽ tự động chọn SERIAL (hoặc BIGSERIAL nếu dữ liệu dự kiến rất lớn)
-- và hiểu rằng đây là một cú pháp tiện lợi cho việc quản lý sequence ngầm.
-- Điều này giúp Antigravity tạo ra các lược đồ robust và dễ bảo trì.
```

### 3.3. Các Kiểu Số Thập Phân Chính Xác (Exact Numeric Types)

Khi bạn cần lưu trữ các số có phần thập phân với độ chính xác tuyệt đối, chẳng hạn như tiền tệ, tính toán tài chính, khoa học hoặc số dư ngân hàng, bạn **phải** sử dụng kiểu `NUMERIC` hoặc `DECIMAL`. Hai kiểu này hoàn toàn tương đương trong PostgreSQL và tuân thủ tiêu chuẩn SQL.

*   **`NUMERIC(precision, scale)`** hoặc **`DECIMAL(precision, scale)`**:
    *   `precision`: Tổng số chữ số tối đa (cả phần nguyên và phần thập phân) mà số có thể chứa. Bắt buộc phải lớn hơn hoặc bằng `scale`.
    *   `scale`: Số chữ số tối đa sau dấu thập phân.
    *   Ví dụ: `NUMERIC(10, 2)` có thể lưu trữ số có tổng cộng 10 chữ số, trong đó có 2 chữ số sau dấu thập phân. Phạm vi giá trị sẽ là từ -9,999,999.99 đến 9,999,999.99.
    *   Nếu `scale` không được chỉ định, mặc định là 0. Nếu cả `precision` và `scale` không được chỉ định (chỉ `NUMERIC`), thì giá trị có thể lưu trữ bất kỳ độ chính xác nào, nhưng điều này không được khuyến khích vì có thể dẫn đến không nhất quán.

> [!TIP]
> Các kiểu `NUMERIC`/`DECIMAL` đảm bảo độ chính xác tuyệt đối bằng cách lưu trữ giá trị dưới dạng văn bản (textual representation) hoặc dưới dạng một mảng các chữ số, không phải dưới dạng nhị phân dấu phẩy động. Điều này đảm bảo rằng không có lỗi làm tròn nào xảy ra, dù cho đổi lại, chúng có thể kém hiệu quả hơn trong các phép tính so với các kiểu dấu phẩy động.
>
> **Vibe Coding cho dữ liệu tài chính:** Khi Antigravity IDE phân tích yêu cầu liên quan đến "giá", "số dư", "tiền tệ", nó sẽ tự động *vibe* rằng cần độ chính xác tuyệt đối và chọn `NUMERIC(p, s)` với `scale` phù hợp (thường là 2 cho tiền tệ).

**Ví dụ:**

```sql
-- Khai báo NUMERIC với 5 chữ số tổng cộng, 2 chữ số sau dấu thập phân
SELECT '123.45'::numeric(5, 2) AS exact_value; -- Kết quả: 123.45

-- Chèn giá trị vượt quá scale (sẽ làm tròn theo quy tắc làm tròn tiêu chuẩn)
SELECT '123.456'::numeric(5, 2) AS rounded_value; -- Kết quả: 123.46 (làm tròn lên)
SELECT '123.454'::numeric(5, 2) AS rounded_down_value; -- Kết quả: 123.45 (làm tròn xuống)

-- Chèn giá trị vượt quá precision (sẽ gây lỗi)
-- SELECT '1234.56'::numeric(5, 2); -- Lỗi: numeric field overflow

-- Tạo bảng cho giao dịch tài chính
CREATE TABLE transactions (
    transaction_id SERIAL PRIMARY KEY,
    amount NUMERIC(15, 2) NOT NULL, -- Tổng 15 chữ số, 2 chữ số thập phân
    description TEXT
);

INSERT INTO transactions (amount, description) VALUES (199999999.99, 'Largest allowed transaction');
INSERT INTO transactions (amount, description) VALUES (50.25, 'Coffee purchase');

SELECT * FROM transactions;
```

### 3.4. Các Kiểu Số Dấu Phẩy Động Gần Đúng (Approximate Numeric Types)

Các kiểu số dấu phẩy động được sử dụng để lưu trữ các số có phần thập phân khi độ chính xác tuyệt đối không phải là ưu tiên hàng đầu, và bạn cần hiệu suất cao hơn trong các phép tính. Chúng sử dụng biểu diễn dấu phẩy động chuẩn IEEE 754.

*   **`REAL`** (hoặc `FLOAT4`): Số dấu phẩy động chính xác đơn.
    *   Lưu trữ: 4 byte.
    *   Cung cấp khoảng 6 chữ số thập phân có nghĩa.
*   **`DOUBLE PRECISION`** (hoặc `FLOAT8`): Số dấu phẩy động chính xác kép.
    *   Lưu trữ: 8 byte.
    *   Cung cấp khoảng 15 chữ số thập phân có nghĩa. Đây là lựa chọn phổ biến hơn khi cần độ chính xác cao hơn `REAL`.

> [!CAUTION]
> Các kiểu dấu phẩy động như `REAL` và `DOUBLE PRECISION` sử dụng phép toán dấu phẩy động (floating-point arithmetic), vốn nổi tiếng là **không chính xác hoàn toàn** do cách biểu diễn nhị phân của các số thập phân. Điều này có nghĩa là các phép tính số học có thể tạo ra các kết quả có sai số nhỏ ở phần thập phân, đặc biệt khi thực hiện nhiều phép tính liên tiếp.
>
> **Khi nào sử dụng?** Khi dữ liệu của bạn vốn đã có sai số đo lường hoặc độ chính xác nhỏ không đáng kể. Ví dụ:
> *   Tọa độ địa lý (vĩ độ, kinh độ).
> *   Các phép đo khoa học không yêu cầu độ chính xác tuyệt đối (nhiệt độ, áp suất).
> *   "Số lít nước trong một hồ" hoặc "khối lượng rác trong bãi rác", nơi mà sự khác biệt nhỏ không đáng kể và dữ liệu có thể đã được đo lường không hoàn toàn chính xác.
>
> **Vibe Coding và sự hiểu biết về sai số:** Một Antigravity subagent sử dụng Vibe Coding sẽ nhận ra rằng đối với dữ liệu tài chính, `FLOAT` là một "anti-pattern" và sẽ từ chối sử dụng nó. Ngược lại, đối với dữ liệu địa lý, `DOUBLE PRECISION` là lựa chọn tối ưu về hiệu suất và độ chính xác chấp nhận được.

**Ví dụ về sự không chính xác của dấu phẩy động:**

```sql
-- Phép trừ với REAL (có thể cho kết quả không chính xác tuyệt đối)
SELECT 1.99999::real - 1.99998::real AS float_subtraction;
-- Kết quả có thể là 0.0000100135806322, không phải 0.00001 chính xác.

-- So sánh với NUMERIC (cho kết quả chính xác)
SELECT 1.99999::numeric(7,5) - 1.99998::numeric(7,5) AS numeric_subtraction; -- Kết quả: 0.00001

-- Ví dụ về tích lũy sai số
SELECT (0.1 + 0.2)::double precision = 0.3::double precision AS float_comparison; -- Kết quả: FALSE!
SELECT (0.1::numeric + 0.2::numeric) = 0.3::numeric AS numeric_comparison; -- Kết quả: TRUE

-- Giải thích: 0.1 và 0.2 không thể biểu diễn chính xác trong hệ nhị phân dấu phẩy động.
-- Khi cộng chúng lại, kết quả không chính xác tuyệt đối là 0.3, dẫn đến so sánh FALSE.
```

### 3.5. Tóm Tắt Nhanh Các Quy Tắc Chọn Kiểu Số

> [!TIP]
> Dưới đây là bốn quy tắc nhanh để lựa chọn kiểu dữ liệu số, áp dụng tư duy Vibe Coding để tối ưu hóa thiết kế:
> 1.  **ID Tự Động Tăng:** Luôn sử dụng `SERIAL` (hoặc `SMALLSERIAL`, `BIGSERIAL` tùy phạm vi dự kiến) cho các cột ID là khóa chính. Điều này tự động hóa việc quản lý chuỗi số và đảm bảo tính duy nhất.
> 2.  **Số Nguyên Không Thập Phân:** Sử dụng `INTEGER` (hoặc `SMALLINT`, `BIGINT` tùy phạm vi) cho các số nguyên không có phần thập phân mà không cần tự động tăng. Đây là lựa chọn mặc định hiệu quả.
> 3.  **Số Chính Xác Tuyệt Đối:** Sử dụng `NUMERIC` hoặc `DECIMAL` cho các giá trị cần độ chính xác cao nhất, như tiền tệ, số dư ngân hàng, hoặc các phép tính khoa học mà sai số dù nhỏ cũng không thể chấp nhận được.
> 4.  **Số Gần Đúng:** Sử dụng `DOUBLE PRECISION` (hoặc `REAL`) cho các số có phần thập phân mà không yêu cầu độ chính xác tuyệt đối, khi hiệu suất tính toán là quan trọng hơn và dữ liệu nguồn có thể đã có sai số. Tránh sử dụng cho dữ liệu tài chính hoặc bất kỳ dữ liệu nào cần so sánh chính xác.

## 4. Các Kiểu Dữ Liệu Ký Tự (Character Types)

Các kiểu dữ liệu ký tự được sử dụng để lưu trữ văn bản (chuỗi). PostgreSQL cung cấp ba tùy chọn chính, mỗi tùy chọn có hành vi hơi khác nhau về cách xử lý độ dài chuỗi và ảnh hưởng đến hiệu suất/lưu trữ.

### 4.1. `CHAR(n)` (Fixed-Length Character String)

*   **`CHARACTER(n)`** (hoặc `CHAR(n)`): Lưu trữ chuỗi có độ dài cố định `n`.
    *   **Padding:** Nếu chuỗi đầu vào ngắn hơn `n`, PostgreSQL sẽ tự động thêm các khoảng trắng vào cuối (padding) để đạt đủ độ dài `n`.
    *   **Truncation:** Nếu chuỗi đầu vào dài hơn `n`, PostgreSQL sẽ cắt bớt chuỗi (truncation) để chỉ còn `n` ký tự.
    *   **Lưu trữ:** Luôn chiếm `n` byte (hoặc `n` ký tự nếu mã hóa đa byte như UTF-8, nhưng vẫn cố định theo `n` ký tự).

> [!CAUTION]
> `CHAR(n)` rất ít được sử dụng trong các ứng dụng hiện đại vì việc padding có thể gây ra các vấn đề không mong muốn:
> *   **So sánh chuỗi:** `CHAR(n)` sẽ so sánh các chuỗi đã được padding, điều này có thể dẫn đến kết quả không trực quan. Ví dụ, `'A'` (trong CHAR(3) sẽ là `'A  '`) không bằng `'A'` (trong TEXT).
> *   **Hiệu suất:** Padding và truncation có thể tạo ra chi phí xử lý không cần thiết.
> *   **Lưu trữ lãng phí:** Nếu hầu hết các chuỗi ngắn hơn `n`, không gian lưu trữ sẽ bị lãng phí.
>
> **Vibe Coding và `CHAR(n)`:** Một Antigravity subagent sử dụng Vibe Coding sẽ chỉ xem xét `CHAR(n)` trong các tình huống rất hiếm, ví dụ như khi cần tương thích với các hệ thống cũ hoặc khi dữ liệu có độ dài *luôn luôn* cố định và không có ý nghĩa ngữ cảnh nào khác (ví dụ: mã quốc gia 2 ký tự ISO-3166-1 alpha-2, mặc dù `VARCHAR(2)` thường vẫn được ưu tiên hơn).

**Ví dụ:**

```sql
-- Chuỗi ngắn hơn độ dài n (sẽ bị padding)
SELECT 'A'::char(3) AS padded_char; -- Kết quả: 'A  ' (có 2 khoảng trắng ở cuối)
SELECT length('A'::char(3)) AS padded_length; -- Kết quả: 3

-- Chuỗi dài hơn độ dài n (sẽ bị cắt bớt)
SELECT 'ABCDEFG'::char(3) AS truncated_char; -- Kết quả: 'ABC'
SELECT length('ABCDEFG'::char(3)) AS truncated_length; -- Kết quả: 3

-- So sánh với padding
SELECT 'A'::char(3) = 'A'::text AS char_text_comparison; -- Kết quả: FALSE (do padding)
SELECT trim('A'::char(3)) = 'A'::text AS trimmed_char_text_comparison; -- Kết quả: TRUE
```

### 4.2. `VARCHAR(n)` (Variable-Length Character String with Limit)

*   **`CHARACTER VARYING(n)`** (hoặc `VARCHAR(n)`): Lưu trữ chuỗi có độ dài thay đổi, tối đa là `n` ký tự.
    *   **No Padding:** Nếu chuỗi đầu vào ngắn hơn `n`, nó sẽ được lưu trữ nguyên vẹn mà không có khoảng trắng thừa.
    *   **Truncation:** Nếu chuỗi đầu vào dài hơn `n`, PostgreSQL sẽ cắt bớt chuỗi để chỉ còn `n` ký tự. (Lưu ý: Hành vi cắt bớt này có thể thay đổi tùy thuộc vào cấu hình `standard_conforming_strings` và phiên bản PostgreSQL; các phiên bản hiện đại thường báo lỗi thay vì cắt bớt nếu `standard_conforming_strings` là `on` và `bytea_output` là `escape`). Tuy nhiên, theo mặc định, nó sẽ cắt bớt nếu không có ràng buộc chặt chẽ khác.
    *   **Lưu trữ:** Chỉ chiếm không gian cần thiết để lưu trữ chuỗi thực tế, cộng thêm một vài byte overhead để lưu trữ độ dài.

> [!NOTE]
> Trong PostgreSQL, việc chỉ định độ dài `n` cho `VARCHAR(n)` chủ yếu mang tính chất **xác thực dữ liệu (data validation)**. Nó không mang lại lợi ích hiệu suất đáng kể so với `TEXT` về mặt lưu trữ hoặc truy vấn. Mục đích chính là ngăn chặn việc lưu trữ các chuỗi quá dài một cách vô ý, giúp duy trì tính nhất quán của dữ liệu và giới hạn kích thước trường.

**Ví dụ:**

```sql
-- Chuỗi ngắn hơn độ dài n (không bị padding)
SELECT 'A'::varchar(3) AS varchar_no_padding; -- Kết quả: 'A'
SELECT length('A'::varchar(3)) AS varchar_length; -- Kết quả: 1

-- Chuỗi dài hơn độ dài n (sẽ bị cắt bớt hoặc báo lỗi tùy cấu hình)
SELECT 'ABCDEFG'::varchar(3) AS truncated_varchar; -- Kết quả: 'ABC' (hành vi mặc định)

-- Tạo bảng với cột tên người dùng có giới hạn độ dài
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE, -- Tối đa 50 ký tự, phải duy nhất
    email VARCHAR(100)
);

INSERT INTO users (username, email) VALUES ('john_doe', 'john.doe@example.com');
-- INSERT INTO users (username, email) VALUES ('super_long_username_that_exceeds_fifty_characters_limit', 'test@example.com');
-- Lệnh trên sẽ gây lỗi nếu chuỗi quá dài và cấu hình PostgreSQL không cho phép cắt bớt.
```

### 4.3. `TEXT` (Variable-Length Character String, Unlimited)

*   **`TEXT`**: Lưu trữ chuỗi có độ dài thay đổi mà không có giới hạn độ dài cụ thể. Về cơ bản, nó tương đương với `VARCHAR` nhưng không có giới hạn `n` rõ ràng.
    *   **Lưu trữ:** Giống như `VARCHAR`, chỉ chiếm không gian cần thiết để lưu trữ chuỗi thực tế, cộng thêm một vài byte overhead. Đối với các chuỗi rất dài, PostgreSQL sử dụng cơ chế **TOAST (The Oversized-Attribute Storage Technique)** để lưu trữ chúng hiệu quả hơn bên ngoài hàng dữ liệu chính, giúp giữ cho các hàng dữ liệu chính nhỏ gọn và cải thiện hiệu suất.

> [!TIP]
> Trong PostgreSQL, không có sự khác biệt đáng kể về hiệu suất giữa `VARCHAR(n)` và `TEXT` đối với hầu hết các trường hợp sử dụng. Nhiều nhà phát triển ưa chuộng `TEXT` vì sự đơn giản và linh hoạt, trừ khi cần một giới hạn độ dài rõ ràng cho mục đích xác thực dữ liệu ở mức cơ sở dữ liệu.
>
> **Vibe Coding cho dữ liệu văn bản:** Khi Antigravity IDE cần lưu trữ các trường như "nội dung bài viết", "mô tả sản phẩm", "bình luận", nó sẽ *vibe* rằng độ dài là không cố định và có thể rất lớn. Trong trường hợp này, `TEXT` là lựa chọn ưu việt nhất vì nó không giới hạn độ dài và được tối ưu hóa bởi TOAST cho dữ liệu lớn. Nếu là "tên sản phẩm" hoặc "tiêu đề", `VARCHAR(255)` hoặc `VARCHAR(500)` có thể phù hợp hơn để có một giới hạn rõ ràng.

**Ví dụ:**

```sql
-- Lưu trữ một chuỗi bất kỳ với TEXT
SELECT 'Đây là một chuỗi văn bản dài mà không có giới hạn độ dài cụ thể.'::text AS long_text_example;

-- Tạo bảng cho bài viết blog
CREATE TABLE blog_posts (
    post_id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT, -- Nội dung bài viết không giới hạn độ dài
    author_id INTEGER
);

INSERT INTO blog_posts (title, content, author_id)
VALUES ('Giới thiệu về PostgreSQL', 'PostgreSQL là một hệ quản trị CSDL quan hệ đối tượng mã nguồn mở mạnh mẽ...', 1);
```

## 5. Kiểu Dữ Liệu Boolean (Boolean Data Types)

Kiểu dữ liệu Boolean được sử dụng để lưu trữ các giá trị logic: `TRUE` (đúng), `FALSE` (sai), và `NULL` (không xác định).

*   **`BOOLEAN`** (hoặc `BOOL`): Có thể chứa `TRUE`, `FALSE`, hoặc `NULL`.
*   **Lưu trữ:** 1 byte.

PostgreSQL rất linh hoạt trong việc chấp nhận các giá trị đầu vào cho kiểu Boolean và sẽ tự động chuyển đổi chúng. Điều này rất tiện lợi khi nhập dữ liệu từ các nguồn khác nhau hoặc khi tương tác với các ngôn ngữ lập trình có cách biểu diễn Boolean khác nhau.

*   **Chuyển đổi thành `TRUE`**: `'true'`, `'t'`, `'yes'`, `'y'`, `'on'`, `'1'`
*   **Chuyển đổi thành `FALSE`**: `'false'`, `'f'`, `'no'`, `'n'`, `'off'`, `'0'`
*   **`NULL`**: Đại diện cho sự vắng mặt của giá trị, không phải `TRUE` hay `FALSE`.

> [!NOTE]
> Khả năng chuyển đổi linh hoạt này được thiết kế để tương thích ngược với các hệ thống cơ sở dữ liệu và ngôn ngữ lập trình khác, nơi các giá trị Boolean có thể được biểu diễn bằng số hoặc chuỗi. Khi PostgreSQL hiển thị giá trị Boolean, nó thường sử dụng `'t'` cho `TRUE` và `'f'` cho `FALSE`.

**Ví dụ:**

```sql
-- Các giá trị chuyển đổi thành TRUE
SELECT 'true'::boolean AS is_true_string;
SELECT 'y'::boolean AS is_true_char;
SELECT '1'::boolean AS is_true_number;

-- Các giá trị chuyển đổi thành FALSE
SELECT 'false'::boolean AS is_false_string;
SELECT 'n'::boolean AS is_false_char;
SELECT '0'::boolean AS is_false_number;

-- Giá trị NULL
SELECT NULL::boolean AS null_boolean;

-- Tạo bảng cho trạng thái người dùng
CREATE TABLE user_status (
    user_id SERIAL PRIMARY KEY,
    is_active BOOLEAN DEFAULT TRUE, -- Mặc định là TRUE
    is_admin BOOLEAN NOT NULL DEFAULT FALSE
);

INSERT INTO user_status (is_active, is_admin) VALUES ('t', '1');
INSERT INTO user_status (is_active, is_admin) VALUES ('no', 'false');
INSERT INTO user_status (user_id, is_admin) VALUES (3, 'y'); -- is_active sẽ là TRUE (mặc định)

SELECT * FROM user_status;
```

## 6. Các Kiểu Dữ Liệu Ngày, Giờ và Dấu Thời Gian (Date, Time, and Timestamps)

PostgreSQL cung cấp một bộ sưu tập mạnh mẽ các kiểu dữ liệu để xử lý thông tin về ngày, giờ và dấu thời gian, bao gồm cả việc hỗ trợ múi giờ. Việc hiểu rõ sự khác biệt giữa các kiểu này, đặc biệt là cách chúng xử lý múi giờ, là cực kỳ quan trọng đối với các ứng dụng toàn cầu.

### 6.1. `DATE` (Date Only)

*   **`DATE`**: Lưu trữ chỉ phần ngày (năm, tháng, ngày).
*   **Phạm vi:** Từ 4713 BC đến 5874897 AD.
*   **Lưu trữ:** 4 byte.

PostgreSQL rất linh hoạt trong việc chấp nhận các định dạng chuỗi khác nhau làm đầu vào và sẽ cố gắng phân tích chúng thành một giá trị `DATE` chuẩn (`YYYY-MM-DD`).

**Ví dụ:**

```sql
-- Các định dạng đầu vào khác nhau cho DATE
SELECT '1980-11-20'::date AS iso_date;
SELECT 'November 20, 1980'::date AS full_text_date;
SELECT '20 Nov 1980'::date AS abbreviated_date;
SELECT '11/20/1980'::date AS slash_date; -- Cẩn thận với định dạng này, có thể gây nhầm lẫn (MM/DD/YYYY hoặc DD/MM/YYYY)
```

### 6.2. `TIME` và `TIME WITH TIME ZONE` (Time of Day Only)

*   **`TIME`** (hoặc `TIME WITHOUT TIME ZONE`):
    *   Lưu trữ chỉ phần giờ, phút, giây, và micro giây mà không có thông tin múi giờ.
    *   Lưu trữ: 8 byte.
    *   Hữu ích khi bạn chỉ quan tâm đến thời gian trong ngày mà không liên quan đến ngày cụ thể hoặc múi giờ (ví dụ: giờ mở cửa của cửa hàng).
*   **`TIME WITH TIME ZONE`**:
    *   Lưu trữ phần giờ, phút, giây, micro giây cùng với thông tin múi giờ.
    *   **Cơ chế ngầm:** Khi được lưu trữ, giá trị này sẽ được chuyển đổi và chuẩn hóa thành Giờ Phối hợp Quốc tế (UTC). Khi truy xuất, nó sẽ được chuyển đổi trở lại múi giờ của phiên hiện tại của client.
    *   Lưu trữ: 12 byte.

> [!CAUTION]
> `TIME WITH TIME ZONE` có thể gây nhầm lẫn. Nó lưu trữ thời gian trong ngày *của một múi giờ cụ thể*, nhưng không lưu trữ ngày. Điều này có nghĩa là khi chuyển đổi sang UTC, nó cần một ngày tham chiếu (thường là ngày hiện tại) để thực hiện chuyển đổi, sau đó loại bỏ ngày đó đi. Do đó, kiểu này ít được sử dụng hơn `TIMESTAMP WITH TIME ZONE`.

**Ví dụ:**

```sql
-- TIME (không có múi giờ)
SELECT '01:23:45'::time AS simple_time;
SELECT '1:23 PM'::time AS pm_time;
SELECT '14:30:00.123456'::time AS time_with_microseconds;

-- TIME WITH TIME ZONE (chuyển đổi sang UTC và hiển thị theo múi giờ phiên)
-- Múi giờ mặc định của phiên làm việc có thể ảnh hưởng đến kết quả hiển thị.
-- Giả sử múi giờ phiên hiện tại là 'UTC' hoặc 'Asia/Ho_Chi_Minh' (+07)
SELECT '01:23:45 AM PST'::time with time zone AS time_pst_to_session; -- PST là -08:00
SELECT '01:23:45 AM EST'::time with time zone AS time_est_to_session; -- EST là -05:00
SELECT '01:23:45 AM Z'::time with time zone AS time_utc_to_session;  -- 'Z' biểu thị UTC

-- Để thấy rõ sự khác biệt, hãy thay đổi múi giờ của phiên
SET timezone = 'America/Los_Angeles'; -- Múi giờ PST/PDT
SELECT '01:23:45 AM Z'::time with time zone AS time_utc_in_la; -- Kết quả: 18:23:45-07 (nếu là PDT)
SET timezone = 'Asia/Ho_Chi_Minh'; -- Múi giờ Việt Nam (+07)
SELECT '01:23:45 AM Z'::time with time zone AS time_utc_in_hcm; -- Kết quả: 08:23:45+07
```

### 6.3. `TIMESTAMP` và `TIMESTAMP WITH TIME ZONE` (Date and Time)

Đây là hai kiểu dữ liệu quan trọng nhất để xử lý ngày và giờ kết hợp.

*   **`TIMESTAMP`** (hoặc `TIMESTAMP WITHOUT TIME ZONE`):
    *   Lưu trữ cả ngày và giờ (năm, tháng, ngày, giờ, phút, giây, micro giây), nhưng **không có thông tin múi giờ**.
    *   Lưu trữ: 8 byte.
    *   Giá trị được lưu trữ chính xác như những gì bạn cung cấp, không có sự chuyển đổi múi giờ nào xảy ra.
    *   Sử dụng khi bạn cần lưu trữ thời gian cục bộ hoặc khi tất cả các sự kiện diễn ra trong cùng một múi giờ và bạn không cần quan tâm đến múi giờ khác.
*   **`TIMESTAMP WITH TIME ZONE`** (hoặc `TIMESTAMPTZ`):
    *   Lưu trữ cả ngày và giờ cùng với thông tin múi giờ.
    *   **Cơ chế ngầm cực kỳ quan trọng:**
        1.  **Lưu trữ (Storage):** Khi một giá trị được chèn vào cột `TIMESTAMPTZ`, PostgreSQL sẽ sử dụng thông tin múi giờ được cung cấp (hoặc múi giờ của phiên hiện tại nếu không được cung cấp) để **chuyển đổi giá trị đó sang UTC** (Coordinated Universal Time). Giá trị UTC này sau đó được lưu trữ trong cơ sở dữ liệu.
        2.  **Truy xuất (Retrieval):** Khi bạn truy vấn một giá trị từ cột `TIMESTAMPTZ`, PostgreSQL sẽ lấy giá trị UTC đã lưu trữ và **chuyển đổi nó trở lại múi giờ của phiên hiện tại** của client trước khi hiển thị cho bạn.
    *   Lưu trữ: 8 byte (giống `TIMESTAMP` vì múi giờ được xử lý lúc chuyển đổi, không phải là một phần của giá trị lưu trữ).

> [!TIP]
> **Vibe Coding cho ứng dụng toàn cầu:** `TIMESTAMPTZ` là lựa chọn tối ưu và được khuyến nghị mạnh mẽ cho hầu hết các ứng dụng, đặc biệt là những ứng dụng có người dùng ở các múi giờ khác nhau hoặc cần theo dõi chuỗi sự kiện toàn cầu. Bằng cách lưu trữ mọi thứ ở UTC, bạn đảm bảo tính nhất quán và dễ dàng xử lý các phép toán thời gian mà không lo lắng về sự thay đổi múi giờ hoặc DST (Daylight Saving Time).
>
> **Antigravity IDE và `TIMESTAMPTZ`:** Khi một Antigravity subagent thiết kế một cột `created_at` hoặc `last_updated`, nó sẽ *vibe* rằng cột này cần theo dõi thời gian toàn cầu và sẽ tự động đề xuất `TIMESTAMPTZ` với giá trị mặc định `NOW()` hoặc `CURRENT_TIMESTAMP` để đảm bảo dữ liệu thời gian được nhất quán. Nếu Antigravity cần debug một vấn đề liên quan đến thời gian, nó có thể tự động thay đổi `SET timezone = '...'` để kiểm tra hành vi của dữ liệu ở các múi giờ khác nhau.

**Ví dụ:**

```sql
-- TIMESTAMP (không có múi giờ) - Giá trị được lưu chính xác như nhập
SELECT '1980-11-20 01:23:45'::timestamp AS local_timestamp;

-- TIMESTAMP WITH TIME ZONE (chuyển đổi sang UTC khi lưu, hiển thị theo múi giờ phiên)
-- Giả sử múi giờ phiên hiện tại là 'Asia/Ho_Chi_Minh' (+07)
SET timezone = 'Asia/Ho_Chi_Minh';
SELECT '1980-11-20 01:23:45 AM PST'::timestamp with time zone AS timestamptz_pst_input;
-- PST là -08:00. 01:23:45 AM PST tương đương 09:23:45 AM UTC.
-- Khi hiển thị ở múi giờ +07, nó sẽ là 16:23:45 +07.
-- Kết quả: "1980-11-20 16:23:45+07"

SELECT '1980-11-20 01:23:45 AM EST'::timestamp with time zone AS timestamptz_est_input;
-- EST là -05:00. 01:23:45 AM EST tương đương 06:23:45 AM UTC.
-- Khi hiển thị ở múi giờ +07, nó sẽ là 13:23:45 +07.
-- Kết quả: "1980-11-20 13:23:45+07"

SELECT '1980-11-20 01:23:45 AM Z'::timestamp with time zone AS timestamptz_utc_input;
-- Z là UTC. 01:23:45 AM UTC.
-- Khi hiển thị ở múi giờ +07, nó sẽ là 08:23:45 +07.
-- Kết quả: "1980-11-20 08:23:45+07"

-- Tạo bảng cho các sự kiện với dấu thời gian
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    event_name VARCHAR(255),
    event_time_local TIMESTAMP, -- Thời gian cục bộ, không quan tâm múi giờ
    event_time_utc TIMESTAMPTZ DEFAULT NOW() -- Thời gian chuẩn UTC, mặc định là thời gian hiện tại
);

INSERT INTO events (event_name, event_time_local, event_time_utc)
VALUES ('Local Meeting', '2023-11-20 10:00:00', '2023-11-20 10:00:00 Asia/Ho_Chi_Minh');

INSERT INTO events (event_name, event_time_local)
VALUES ('Global Webinar', '2023-11-21 15:00:00'); -- event_time_utc sẽ là NOW() (tức thời gian hiện tại ở múi giờ phiên, sau đó chuyển sang UTC)

SELECT event_name, event_time_local, event_time_utc FROM events;

-- Để xem giá trị TIMESTAMPTZ ở múi giờ khác:
SET timezone = 'America/New_York'; -- Múi giờ EST/EDT
SELECT event_name, event_time_local, event_time_utc FROM events;
-- Bạn sẽ thấy event_time_utc thay đổi theo múi giờ New York, trong khi event_time_local không thay đổi.
```

## 7. Kiểu Dữ Liệu Khoảng Thời Gian (Interval Data Type)

Kiểu dữ liệu `INTERVAL` được sử dụng để lưu trữ một khoảng thời gian hoặc một khoảng thời lượng. Nó có thể biểu diễn khoảng thời gian theo năm, tháng, ngày, giờ, phút, giây, v.v., và là một công cụ mạnh mẽ để thực hiện các phép toán ngày/giờ.

*   **`INTERVAL`**: Đại diện cho một khoảng thời gian.
*   **Lưu trữ:** 16 byte.

PostgreSQL cho phép bạn chỉ định khoảng thời gian theo nhiều cách linh hoạt, từ chuỗi văn bản tự nhiên đến định dạng chuẩn SQL.

**Ví dụ về khai báo `INTERVAL`:**

```sql
-- Các cách khác nhau để khai báo INTERVAL
SELECT '1 day'::interval AS one_day;
SELECT '1 day 20 hours'::interval AS day_hours;
SELECT '20 hours 30 minutes 45 seconds'::interval AS hours_minutes_seconds;
SELECT '1 year 2 months 3 days 4 hours 5 minutes 6 seconds'::interval AS full_interval;
SELECT '5 years'::interval AS five_years;
SELECT '3 months'::interval AS three_months;
SELECT '10 minutes'::interval AS ten_minutes;
```

### 7.1. Sức Mạnh của `INTERVAL` trong Phép Toán Ngày/Giờ

Mặc dù việc lưu trữ `INTERVAL` trực tiếp trong một cột có thể không phổ biến (trừ khi bạn cần lưu trữ "thời lượng" một cách rõ ràng), nhưng sức mạnh thực sự của `INTERVAL` nằm ở khả năng thực hiện các phép toán số học với các kiểu dữ liệu ngày, giờ và dấu thời gian.

> [!IMPORTANT]
> Việc thực hiện các phép tính ngày/giờ/khoảng thời gian trực tiếp trong cơ sở dữ liệu bằng `INTERVAL` là một tính năng cực kỳ hữu ích. Nó giúp giảm tải cho tầng ứng dụng, cho phép logic xử lý thời gian phức tạp được thực hiện hiệu quả ngay tại nguồn dữ liệu mà không cần các thư viện chuyên biệt trong ngôn ngữ lập trình.
>
> **Vibe Coding cho logic thời gian:** Một Antigravity subagent khi phân tích các yêu cầu nghiệp vụ như "thêm 30 ngày vào ngày hết hạn", "tính tuổi của người dùng", "tìm sự kiện trong vòng 24 giờ tới", nó sẽ tự động *vibe* rằng `INTERVAL` là công cụ tối ưu để thực hiện các phép toán này trực tiếp trong SQL, thay vì kéo dữ liệu về ứng dụng để xử lý. Điều này không chỉ hiệu quả mà còn đảm bảo tính chính xác.

**Phép Toán Với `INTERVAL`:**

```sql
-- Cộng INTERVAL vào DATE
SELECT '2023-10-27'::date + '1 day'::interval AS next_day; -- Kết quả: 2023-10-28

-- Trừ INTERVAL từ TIMESTAMP
SELECT '2023-10-27 10:00:00'::timestamp - '2 hours'::interval AS two_hours_ago; -- Kết quả: 2023-10-27 08:00:00

-- Phép trừ giữa hai dấu thời gian (kết quả là INTERVAL)
SELECT '2023-10-27 10:00:00'::timestamp - '2023-10-26 08:00:00'::timestamp AS time_difference; -- Kết quả: 1 day 02:00:00

-- Phép trừ giữa hai dấu thời gian có múi giờ
SET timezone = 'Asia/Ho_Chi_Minh';
SELECT '2023-11-20 11:23 AM EST'::timestamptz - '2023-11-10 05:43 AM PST'::timestamptz AS complex_time_diff;
-- EST (-05:00) 11:23 AM = 16:23 UTC
-- PST (-08:00) 05:43 AM = 13:43 UTC
-- Chênh lệch: 2023-11-20 16:23 UTC - 2023-11-10 13:43 UTC = 10 days 02:40:00
-- Kết quả: 10 days 02:40:00 (lưu ý sự khác biệt múi giờ đã được chuẩn hóa sang UTC trước khi tính toán)

-- Tính ngày hết hạn
SELECT '2023-01-01'::date + '1 year 2 months'::interval AS subscription_expiry; -- Kết quả: 2024-03-01

-- Tính tuổi từ ngày sinh (sử dụng AGE() function hoặc phép trừ)
SELECT age('1990-05-15'::date) AS current_age;
SELECT NOW()::date - '1990-05-15'::date AS days_old;
```

## 8. Giá Trị `NULL`

Trong suốt chương này, chúng ta đã đề cập đến `NULL` trong ngữ cảnh của các kiểu dữ liệu Boolean và khả năng cho phép cột chứa `NULL`. `NULL` là một khái niệm tối quan trọng trong cơ sở dữ liệu và có thể được áp dụng cho hầu hết các kiểu dữ liệu.

*   **`NULL`**: Đại diện cho sự vắng mặt của giá trị. Nó không phải là số 0, không phải chuỗi rỗng (`''`), và không phải `FALSE`. Nó đơn giản là "không biết", "không có giá trị", hoặc "không áp dụng".

> [!NOTE]
> **Mặc định các cột có thể là `NULL`** trừ khi bạn áp dụng ràng buộc `NOT NULL`. Khi một cột được phép chứa giá trị `NULL`, điều đó có nghĩa là không có thông tin nào được cung cấp cho cột đó trong một hàng cụ thể.
>
> **So sánh với `NULL`:** Bất kỳ phép toán so sánh nào với `NULL` (ví dụ: `=`, `<`, `>`, `!=`) đều sẽ trả về `NULL` (hoặc "unknown"), không phải `TRUE` hay `FALSE`. Để kiểm tra xem một giá trị có phải là `NULL` hay không, bạn phải sử dụng toán tử `IS NULL` hoặc `IS NOT NULL`.
>
> **Vibe Coding và `NULL`:** Một Antigravity subagent sử dụng Vibe Coding sẽ tự động đánh giá xem một trường có bắt buộc phải có giá trị hay không. Nếu một trường như "email" của người dùng có thể không tồn tại, Antigravity sẽ không áp dụng `NOT NULL`, cho phép `NULL` để biểu thị sự vắng mặt của dữ liệu. Nếu một trường như "tên người dùng" là bắt buộc, Antigravity sẽ thêm ràng buộc `NOT NULL` để đảm bảo tính toàn vẹn.

**Ví dụ:**

```sql
-- Một cột có thể chứa giá trị NULL (email, start_date)
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL, -- Tên là bắt buộc
    email VARCHAR(100), -- Có thể là NULL nếu nhân viên không có email
    start_date DATE -- Có thể là NULL nếu ngày bắt đầu chưa xác định
);

INSERT INTO employees (name, email, start_date) VALUES ('Alice', 'alice@example.com', '2023-01-15');
INSERT INTO employees (name, start_date) VALUES ('Bob', '2023-03-01'); -- Email sẽ là NULL
INSERT INTO employees (name, email) VALUES ('Charlie', 'charlie@example.com'); -- Start_date sẽ là NULL

SELECT * FROM employees;

-- Tìm nhân viên không có email
SELECT * FROM employees WHERE email IS NULL;

-- Tìm nhân viên có email
SELECT * FROM employees WHERE email IS NOT NULL;

-- Ví dụ về so sánh với NULL
SELECT (1 = NULL) AS compare_one_null; -- Kết quả: NULL
SELECT ('Hello' = NULL) AS compare_string_null; -- Kết quả: NULL
SELECT (NULL IS NULL) AS null_is_null; -- Kết quả: TRUE
```

## 9. Tóm Tắt Phần

Chương này đã cung cấp một cái nhìn sâu sắc về các kiểu dữ liệu cơ bản và phổ biến nhất trong PostgreSQL, nhấn mạnh tầm quan trọng của việc lựa chọn đúng đắn để đảm bảo tính toàn vẹn, hiệu suất và hiệu quả lưu trữ của cơ sở dữ liệu.

*   **Kiểu dữ liệu** là nền tảng của mọi hệ thống cơ sở dữ liệu, định hình cách dữ liệu được lưu trữ, xử lý và tương tác. Việc hiểu rõ chúng là bước đầu tiên để thiết kế cơ sở dữ liệu mạnh mẽ.
*   Sử dụng cú pháp `SELECT 'giá_trị'::kiểu_dữ_liệu;` là một công cụ mạnh mẽ để kiểm tra và hiểu hành vi của các kiểu dữ liệu trong PostgreSQL một cách nhanh chóng.
*   **Kiểu số nguyên (`SMALLINT`, `INTEGER`, `BIGINT`)** dùng cho các số không có thập phân. `INTEGER` là lựa chọn phổ biến, cung cấp sự cân bằng tốt.
*   **Kiểu tự động tăng (`SMALLSERIAL`, `SERIAL`, `BIGSERIAL`)** là cú pháp tiện lợi cho các cột ID tự động tăng, được triển khai ngầm bằng cách sử dụng `SEQUENCE` và kiểu số nguyên tương ứng. `SERIAL` là lựa chọn phổ biến cho khóa chính.
*   **Kiểu số thập phân chính xác (`NUMERIC`, `DECIMAL`)** là bắt buộc cho các giá trị cần độ chính xác tuyệt đối như tiền tệ, số dư ngân hàng, tránh các vấn đề sai số dấu phẩy động.
*   **Kiểu số dấu phẩy động gần đúng (`REAL`, `DOUBLE PRECISION`)** dùng cho các giá trị thập phân khi hiệu suất quan trọng hơn độ chính xác tuyệt đối, nhưng cần lưu ý về vấn đề sai số dấu phẩy động.
*   **Kiểu ký tự (`CHAR(n)`, `VARCHAR(n)`, `TEXT`)**:
    *   `CHAR(n)` có độ dài cố định, bị padding và ít được khuyến nghị.
    *   `VARCHAR(n)` có độ dài thay đổi, `n` chủ yếu để xác thực.
    *   `TEXT` có độ dài thay đổi không giới hạn, là lựa chọn linh hoạt nhất.
    *   Trong PostgreSQL, `VARCHAR(n)` và `TEXT` có hiệu suất tương đương cho hầu hết các trường hợp.
*   **Kiểu Boolean (`BOOLEAN`)** lưu trữ `TRUE`, `FALSE`, hoặc `NULL`, với khả năng chuyển đổi linh hoạt từ nhiều định dạng đầu vào.
*   **Kiểu ngày, giờ (`DATE`, `TIME`, `TIMESTAMP`)** lưu trữ thông tin thời gian. Các kiểu có `WITH TIME ZONE` (như `TIMESTAMPTZ`) sẽ chuẩn hóa giá trị về UTC khi lưu trữ và chuyển đổi sang múi giờ phiên khi truy xuất, là lựa chọn ưu việt cho các ứng dụng toàn cầu.
*   **Kiểu khoảng thời gian (`INTERVAL`)** biểu diễn một khoảng thời lượng và là công cụ cực kỳ hữu ích cho các phép toán cộng/trừ với các kiểu ngày/giờ/dấu thời gian, cho phép thực hiện logic thời gian phức tạp trực tiếp trong cơ sở dữ liệu.
*   **`NULL`** đại diện cho sự vắng mặt của giá trị, không phải 0 hay chuỗi rỗng. Nó yêu cầu các toán tử so sánh đặc biệt (`IS NULL`, `IS NOT NULL`).
*   **Vibe Coding và Antigravity IDE:** Một hệ thống Agentic AI như Antigravity, khi được trang bị tư duy Vibe Coding, sẽ không chỉ tuân theo các quy tắc mà còn "cảm nhận" được bản chất của dữ liệu để đưa ra lựa chọn kiểu dữ liệu tối ưu, dự đoán hành vi tương lai và tối ưu hóa hiệu suất, đảm bảo tính toàn vẹn cho lược đồ cơ sở dữ liệu. Việc hiểu sâu về các kiểu dữ liệu là chìa khóa để bạn có thể áp dụng và kiểm soát những công cụ AI mạnh mẽ này một cách hiệu quả.

<!-- REVIEWED_BY_AGENT -->
