# Phần 24: Ràng Buộc UNIQUE và CHECK

## Giới Thiệu Tổng Quan: Nền Tảng Cho Toàn Vẹn Dữ Liệu

Trong kiến trúc hệ thống hiện đại, cơ sở dữ liệu đóng vai trò là kho lưu trữ trung tâm cho mọi thông tin quan trọng. Việc đảm bảo tính toàn vẹn (integrity) và độ chính xác (accuracy) của dữ liệu không chỉ là một yêu cầu kỹ thuật mà còn là yếu tố cốt lõi quyết định sự tin cậy và thành công của ứng dụng. Dữ liệu không hợp lệ có thể dẫn đến hàng loạt vấn đề nghiêm trọng: từ các lỗi logic khó phát hiện, báo cáo tài chính sai lệch, cho đến việc mất niềm tin từ người dùng và đối tác kinh doanh.

PostgreSQL, một hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) mạnh mẽ và tuân thủ tiêu chuẩn, cung cấp một bộ công cụ phong phú để thực thi các quy tắc nghiệp vụ và duy trì chất lượng dữ liệu ở cấp độ nền tảng nhất. Trong chương này, chúng ta sẽ đi sâu vào hai loại ràng buộc (constraint) quan trọng: `UNIQUE` và `CHECK`.

*   **Ràng buộc `UNIQUE`**: Đảm bảo rằng tất cả các giá trị trong một cột hoặc một nhóm các cột phải là duy nhất trên toàn bộ bảng, ngăn chặn hiệu quả sự trùng lặp không mong muốn.
*   **Ràng buộc `CHECK`**: Cho phép chúng ta định nghĩa các điều kiện tùy chỉnh mà dữ liệu phải thỏa mãn trước khi được chấp nhận vào bảng, kiểm soát phạm vi và mối quan hệ giữa các giá trị.

Nắm vững cách sử dụng các ràng buộc này không chỉ giúp bạn xây dựng các cơ sở dữ liệu mạnh mẽ, đáng tin cậy mà còn cải thiện hiệu suất truy vấn và khả năng bảo trì của hệ thống. Chúng ta sẽ khám phá cách áp dụng các ràng buộc này, cơ chế hoạt động ngầm định của chúng, cách quản lý chúng, và cuối cùng, thảo luận về chiến lược tối ưu để phân bổ trách nhiệm xác thực dữ liệu giữa cơ sở dữ liệu và tầng ứng dụng.

Khi phát triển với một hệ thống AI mạnh mẽ như **Antigravity IDE**, việc hiểu sâu về các ràng buộc này càng trở nên quan trọng. **Antigravity** có khả năng tự động sinh mã SQL, bao gồm cả việc định nghĩa các ràng buộc dựa trên ngữ cảnh và yêu cầu nghiệp vụ. Tuy nhiên, để thực hiện "Vibe Coding" một cách hiệu quả – tức là điều chỉnh và tinh chỉnh các đề xuất của AI bằng trực giác và kiến thức chuyên sâu – bạn cần nắm vững triết lý và cơ chế hoạt động của từng loại ràng buộc. Điều này giúp bạn không chỉ chấp nhận mà còn *hiểu rõ* tại sao Antigravity lại đề xuất một ràng buộc cụ thể, và làm thế nào để hướng dẫn nó tạo ra một lược đồ cơ sở dữ liệu tối ưu nhất.

## 1. Ràng Buộc UNIQUE: Đảm Bảo Tính Duy Nhất Của Dữ Liệu

Ràng buộc `UNIQUE` là một công cụ thiết yếu để duy trì tính nhất quán của dữ liệu. Nó đảm bảo rằng không có hai hàng nào trong bảng có cùng một giá trị (hoặc cùng một tập hợp giá trị) trong một cột hoặc một nhóm các cột được chỉ định. Điều này ngăn chặn dữ liệu trùng lặp không mong muốn, vốn có thể gây ra sai sót logic và khó khăn trong việc phân tích.

### 1.1. Cơ Chế Hoạt Động Ngầm và Tầm Quan Trọng

Khi bạn định nghĩa một ràng buộc `UNIQUE` trên một hoặc nhiều cột, PostgreSQL tự động tạo một **chỉ mục duy nhất (unique index)** trên các cột đó. Chính chỉ mục này là cơ chế ngầm định giúp PostgreSQL nhanh chóng kiểm tra tính duy nhất của dữ liệu mỗi khi có thao tác chèn (INSERT) hoặc cập nhật (UPDATE).

*   **Tốc độ kiểm tra:** Chỉ mục duy nhất cho phép PostgreSQL tìm kiếm các giá trị trùng lặp một cách cực kỳ hiệu quả, ngay cả trên các bảng có hàng triệu bản ghi.
*   **Hiệu suất:** Mặc dù việc duy trì chỉ mục có thêm một chút chi phí khi chèn hoặc cập nhật dữ liệu, nhưng lợi ích về tính toàn vẹn và tốc độ truy vấn thường vượt trội hơn nhiều. Các truy vấn tìm kiếm dựa trên các cột có ràng buộc `UNIQUE` cũng được hưởng lợi từ chỉ mục này.
*   **Mối liên hệ với `PRIMARY KEY`:** Một khóa chính (`PRIMARY KEY`) về bản chất là một ràng buộc `UNIQUE` kết hợp với `NOT NULL`. Điều này có nghĩa là cột khóa chính không chỉ phải duy nhất mà còn không được chứa giá trị `NULL`.

### 1.2. Áp Dụng Ràng Buộc UNIQUE Trên Một Cột Đơn

Ràng buộc `UNIQUE` trên một cột đơn đảm bảo rằng mỗi giá trị trong cột đó phải khác biệt so với các giá trị khác. Ví dụ, trong một bảng `products`, mỗi sản phẩm nên có một `name` (tên) duy nhất để tránh nhầm lẫn.

#### Cú pháp

Bạn có thể thêm ràng buộc `UNIQUE` khi tạo bảng (`CREATE TABLE`) hoặc sau khi bảng đã được tạo (`ALTER TABLE`).

**a. Khi tạo bảng (`CREATE TABLE`):**
Thêm từ khóa `UNIQUE` sau định nghĩa kiểu dữ liệu của cột.

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL, -- Tên sản phẩm phải duy nhất và không được NULL
    department VARCHAR(255),
    price NUMERIC(10, 2),
    weight NUMERIC(5, 2)
);
```
> [!NOTE]
> `SERIAL PRIMARY KEY` là cú pháp tiện lợi của PostgreSQL để tạo một cột số nguyên tự động tăng và đặt nó làm khóa chính. `PRIMARY KEY` tự động bao gồm `UNIQUE` và `NOT NULL`.

**b. Sau khi bảng đã được tạo (`ALTER TABLE`):**
Sử dụng lệnh `ALTER TABLE` để thêm ràng buộc vào một cột hiện có. Nên đặt tên rõ ràng cho ràng buộc để dễ quản lý sau này.

```sql
-- Bước 1: Tạo bảng ban đầu không có ràng buộc UNIQUE trên cột name
DROP TABLE IF EXISTS products;
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    department VARCHAR(255),
    price NUMERIC(10, 2),
    weight NUMERIC(5, 2)
);

-- Bước 2: Thêm một số dữ liệu ban đầu, bao gồm cả dữ liệu trùng lặp
INSERT INTO products (name, department, price, weight) VALUES
('Áo Sơ Mi', 'Quần Áo', 25.00, 0.5),
('Quần Jean', 'Quần Áo', 50.00, 1.2),
('Áo Sơ Mi', 'Quần Áo', 24.00, 0.4); -- Dữ liệu trùng lặp đã tồn tại
```

#### Xử lý dữ liệu hiện có và áp dụng ràng buộc

Một điểm cực kỳ quan trọng là bạn **không thể thêm ràng buộc `UNIQUE`** vào một cột nếu cột đó đã chứa các giá trị trùng lặp. PostgreSQL sẽ kiểm tra toàn bộ dữ liệu hiện có trong cột và nếu phát hiện bất kỳ sự trùng lặp nào, lệnh `ALTER TABLE` sẽ thất bại với lỗi.

**Ví dụ minh họa lỗi:**

Khi cố gắng chạy lệnh sau với dữ liệu mẫu ở trên:
```sql
ALTER TABLE products
ADD CONSTRAINT unique_product_name UNIQUE (name);
```
Bạn sẽ nhận được lỗi tương tự như:
```
ERROR:  duplicate key value violates unique constraint "unique_product_name"
DETAIL:  Key (name)=(Áo Sơ Mi) already exists.
```
Điều này cho thấy ràng buộc không thể được áp dụng vì đã có giá trị `Áo Sơ Mi` xuất hiện nhiều hơn một lần.

**Chiến lược khắc phục (Antigravity & Vibe Coding):**

Một hệ thống AI như **Antigravity IDE** sẽ không chỉ báo lỗi mà còn có thể tự động đề xuất các bước khắc phục. Với tư duy "Vibe Coding", bạn sẽ hiểu rằng lỗi này là một tín hiệu cần làm sạch dữ liệu.

1.  **Kiểm tra dữ liệu trùng lặp:**
    Antigravity có thể tự động chạy truy vấn này để xác định các giá trị vi phạm:
    ```sql
    SELECT name, COUNT(*)
    FROM products
    GROUP BY name
    HAVING COUNT(*) > 1;
    ```
    Kết quả sẽ cho thấy `Áo Sơ Mi` có 2 bản ghi.

2.  **Xử lý trùng lặp:**
    Bạn có thể xóa các hàng không cần thiết hoặc cập nhật các giá trị trùng lặp để chúng trở thành duy nhất. Antigravity có thể đề xuất các tùy chọn này. Ví dụ, chúng ta sẽ cập nhật một trong các sản phẩm "Áo Sơ Mi" thành "Áo Sơ Mi Đỏ".

    ```sql
    UPDATE products
    SET name = 'Áo Sơ Mi Đỏ'
    WHERE id = 3; -- Giả sử id của bản ghi trùng lặp là 3
    ```
    (Nếu làm việc trực tiếp trong PgAdmin, bạn có thể chỉnh sửa dữ liệu và lưu lại).

3.  **Áp dụng lại ràng buộc:**
    Sau khi dữ liệu đã duy nhất, bạn có thể thêm ràng buộc:
    ```sql
    ALTER TABLE products
    ADD CONSTRAINT unique_product_name UNIQUE (name);
    ```
    Lệnh này sẽ chạy thành công.

4.  **Kiểm tra ràng buộc:**
    Bây giờ, hãy thử chèn một sản phẩm có tên đã tồn tại:
    ```sql
    INSERT INTO products (name, department, price, weight) VALUES
    ('Áo Sơ Mi', 'Quần Áo', 26.00, 0.6);
    ```
    Bạn sẽ nhận được lỗi tương tự như trước, xác nhận rằng ràng buộc `UNIQUE` đang hoạt động hiệu quả.

> [!TIP]
> **Ràng buộc `UNIQUE` và `NULL` trong PostgreSQL:**
> Trong PostgreSQL (và theo tiêu chuẩn SQL), giá trị `NULL` được coi là không bằng bất kỳ giá trị nào khác, kể cả một `NULL` khác. Do đó, một cột với ràng buộc `UNIQUE` **có thể chứa nhiều giá trị `NULL`**.
> Nếu bạn muốn cột đó không cho phép `NULL` và phải duy nhất, hãy kết hợp `UNIQUE` với `NOT NULL` (ví dụ: `name VARCHAR(255) UNIQUE NOT NULL`). Khóa chính (`PRIMARY KEY`) là một dạng đặc biệt luôn là `UNIQUE NOT NULL`.

### 1.3. Áp Dụng Ràng Buộc UNIQUE Trên Nhiều Cột (Composite Unique Constraint)

Đôi khi, bạn không muốn một cột đơn lẻ là duy nhất, mà là sự kết hợp của các giá trị từ nhiều cột phải là duy nhất. Ví dụ, bạn có thể muốn đảm bảo rằng không có hai sản phẩm nào có cùng `name` *VÀ* cùng `department`. Điều này có nghĩa là "Áo Sơ Mi" trong "Quần Áo" là duy nhất, nhưng "Áo Sơ Mi" trong "Đồ Gia Dụng" lại là một sản phẩm khác hợp lệ.

#### Cú pháp

Tương tự như ràng buộc trên một cột, bạn có thể định nghĩa nó khi tạo bảng hoặc sau đó.

**a. Khi tạo bảng (`CREATE TABLE`):**
Liệt kê các cột trong ngoặc đơn sau từ khóa `UNIQUE`.

```sql
DROP TABLE IF EXISTS products; -- Xóa bảng cũ để tạo mới
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    department VARCHAR(255) NOT NULL,
    price NUMERIC(10, 2),
    weight NUMERIC(5, 2),
    CONSTRAINT unique_name_department UNIQUE (name, department) -- Ràng buộc UNIQUE trên nhiều cột
);
```

**b. Sau khi bảng đã được tạo (`ALTER TABLE`):**
Sử dụng `ALTER TABLE` để thêm ràng buộc.

```sql
-- Giả sử bảng products đã tồn tại
ALTER TABLE products
ADD CONSTRAINT unique_name_department UNIQUE (name, department);
```

#### Ví dụ minh họa

1.  **Chèn dữ liệu hợp lệ:**
    ```sql
    INSERT INTO products (name, department, price, weight) VALUES
    ('Áo Sơ Mi', 'Quần Áo', 25.00, 0.5); -- Hợp lệ
    
    INSERT INTO products (name, department, price, weight) VALUES
    ('Áo Sơ Mi', 'Đồ Gia Dụng', 30.00, 0.7); -- Hợp lệ vì department khác
    ```
    Cả hai lệnh trên sẽ thành công vì mặc dù `name` là "Áo Sơ Mi" bị trùng, nhưng sự kết hợp (`name`, `department`) là duy nhất.

2.  **Chèn dữ liệu vi phạm ràng buộc:**
    ```sql
    INSERT INTO products (name, department, price, weight) VALUES
    ('Áo Sơ Mi', 'Quần Áo', 26.00, 0.6); -- Vi phạm ràng buộc
    ```
    Bạn sẽ nhận được lỗi:
    ```
    ERROR:  duplicate key value violates unique constraint "unique_name_department"
    DETAIL:  Key (name, department)=(Áo Sơ Mi, Quần Áo) already exists.
    ```
    Lỗi này cho thấy ràng buộc `unique_name_department` đã ngăn chặn việc chèn bản ghi trùng lặp. Với **Antigravity IDE**, nó sẽ ngay lập tức chỉ ra rằng thao tác này vi phạm ràng buộc, giúp bạn nhanh chóng điều chỉnh logic hoặc dữ liệu.

## 2. Ràng Buộc CHECK: Kiểm Tra Tính Hợp Lệ Của Dữ Liệu

Ràng buộc `CHECK` cho phép bạn định nghĩa các quy tắc kiểm tra tùy chỉnh cho dữ liệu. Điều này có nghĩa là bạn có thể xác định một biểu thức boolean mà giá trị của cột hoặc các cột trong một hàng phải thỏa mãn. Nếu biểu thức trả về `FALSE`, thao tác chèn hoặc cập nhật sẽ bị từ chối.

### 2.1. Cơ Chế Hoạt Động Ngầm và Phạm Vi

Ràng buộc `CHECK` hoạt động bằng cách đánh giá một biểu thức boolean đã định nghĩa cho *mỗi hàng* mà bạn đang cố gắng chèn hoặc cập nhật.

*   **Chỉ kiểm tra hàng hiện tại:** Ràng buộc `CHECK` chỉ có thể hoạt động bằng cách kiểm tra các thuộc tính (cột) trên hàng mà bạn đang cố gắng chèn hoặc cập nhật.
*   **Không hỗ trợ truy vấn phụ (Subquery):** Một hạn chế quan trọng của ràng buộc `CHECK` trong PostgreSQL là nó **không thể thực hiện các truy vấn phụ (subquery)** để kiểm tra giá trị của các hàng khác trong cùng bảng hoặc các bảng khác. Ví dụ, bạn không thể tạo một ràng buộc `CHECK` để đảm bảo `price` luôn lớn hơn giá trung bình của tất cả sản phẩm, vì điều đó yêu cầu truy vấn toàn bộ bảng. Đối với các quy tắc phức tạp hơn như vậy, bạn sẽ cần sử dụng trigger hoặc xác thực ở tầng ứng dụng.
*   **Hiệu suất:** Đối với các biểu thức `CHECK` đơn giản, chi phí hiệu suất là rất nhỏ. PostgreSQL tối ưu hóa việc kiểm tra này để đảm bảo tốc độ.

### 2.2. Áp Dụng Ràng Buộc CHECK Trên Một Cột Đơn

Ràng buộc `CHECK` thường được sử dụng để giới hạn phạm vi giá trị của một cột. Ví dụ, giá sản phẩm không thể âm, tuổi phải lớn hơn 0, hoặc phần trăm chiết khấu phải nằm trong khoảng 0-100.

#### Cú pháp

Bạn có thể định nghĩa ràng buộc `CHECK` khi tạo bảng hoặc sau khi bảng đã được tạo.

**a. Khi tạo bảng (`CREATE TABLE`):**
Thêm từ khóa `CHECK` và biểu thức điều kiện trong ngoặc đơn sau định nghĩa cột.

```sql
DROP TABLE IF EXISTS products;
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    department VARCHAR(255) NOT NULL,
    price NUMERIC(10, 2) CHECK (price > 0), -- Giá phải lớn hơn 0
    weight NUMERIC(5, 2)
);
```

**b. Sau khi bảng đã được tạo (`ALTER TABLE`):**
Sử dụng `ALTER TABLE` để thêm ràng buộc `CHECK`. Nên đặt tên cho ràng buộc.

```sql
-- Giả sử bảng products đã tồn tại
ALTER TABLE products
ADD CONSTRAINT check_positive_price CHECK (price > 0);
```

#### Xử lý dữ liệu hiện có và áp dụng ràng buộc

Tương tự như ràng buộc `UNIQUE`, bạn không thể thêm ràng buộc `CHECK` nếu dữ liệu hiện có trong bảng vi phạm điều kiện của ràng buộc. Bạn phải sửa dữ liệu trước khi thêm ràng buộc. Antigravity sẽ giúp bạn phát hiện các bản ghi vi phạm và đề xuất cách sửa.

**Ví dụ minh họa:**

1.  **Chèn dữ liệu hợp lệ:**
    ```sql
    INSERT INTO products (name, department, price, weight) VALUES
    ('Thắt Lưng', 'Phụ Kiện', 19.99, 0.2); -- Hợp lệ
    ```

2.  **Chèn dữ liệu vi phạm ràng buộc:**
    ```sql
    INSERT INTO products (name, department, price, weight) VALUES
    ('Thắt Lưng Khuyến Mãi', 'Phụ Kiện', -99.00, 0.2); -- Vi phạm ràng buộc
    ```
    Bạn sẽ nhận được lỗi:
    ```
    ERROR:  new row for relation "products" violates check constraint "check_positive_price"
    DETAIL:  Failing row contains (4, Thắt Lưng Khuyến Mãi, Phụ Kiện, -99.00, 0.20).
    ```
    Lỗi này cho biết rằng ràng buộc `check_positive_price` đã bị vi phạm. Với **Antigravity IDE**, thông báo lỗi này sẽ được phân tích và hiển thị một cách thân thiện hơn, thậm chí gợi ý các giá trị hợp lệ.

### 2.3. Áp Dụng Ràng Buộc CHECK Trên Nhiều Cột

Ràng buộc `CHECK` cũng có thể áp dụng điều kiện liên quan đến nhiều cột trong cùng một hàng. Điều này rất hữu ích cho các quy tắc phức tạp hơn, ví dụ như đảm bảo ngày giao hàng ước tính phải sau ngày tạo đơn hàng.

#### Cú pháp

Khi định nghĩa ràng buộc `CHECK` trên nhiều cột trong `CREATE TABLE`, nó thường được đặt sau tất cả các định nghĩa cột, tương tự như ràng buộc `UNIQUE` nhiều cột.

```sql
DROP TABLE IF EXISTS orders;
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    product_name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    estimated_delivery_time TIMESTAMP NOT NULL,
    CONSTRAINT check_delivery_after_creation CHECK (estimated_delivery_time > created_at)
);
```

**b. Sau khi bảng đã được tạo (`ALTER TABLE`):**
```sql
ALTER TABLE orders
ADD CONSTRAINT check_delivery_after_creation CHECK (estimated_delivery_time > created_at);
```

#### Ví dụ minh họa

1.  **Chèn dữ liệu hợp lệ:**
    ```sql
    INSERT INTO orders (product_name, created_at, estimated_delivery_time) VALUES
    ('Laptop Gaming', '2023-11-20 01:00:00', '2023-11-25 01:00:00'); -- Hợp lệ
    ```
    `estimated_delivery_time` (25/11) là sau `created_at` (20/11), nên bản ghi được chèn thành công.

2.  **Chèn dữ liệu vi phạm ràng buộc:**
    ```sql
    INSERT INTO orders (product_name, created_at, estimated_delivery_time) VALUES
    ('Điện Thoại Mới', '2023-11-20 01:00:00', '2023-11-10 01:00:00'); -- Vi phạm ràng buộc
    ```
    Bạn sẽ nhận được lỗi:
    ```
    ERROR:  new row for relation "orders" violates check constraint "check_delivery_after_creation"
    DETAIL:  Failing row contains (2, Điện Thoại Mới, 2023-11-20 01:00:00, 2023-11-10 01:00:00).
    ```
    Lỗi này chỉ ra rằng `estimated_delivery_time` (10/11) không lớn hơn `created_at` (20/11). **Antigravity IDE** có thể dễ dàng phân tích ngữ cảnh của lỗi này và giải thích rằng ngày giao hàng không thể trước ngày tạo đơn.

## 3. Quản Lý Các Ràng Buộc

Việc quản lý các ràng buộc là một phần quan trọng của quản trị cơ sở dữ liệu. Bạn cần biết cách xem, xóa và đôi khi là tạm thời vô hiệu hóa chúng.

### 3.1. Xem Các Ràng Buộc Hiện Có

**a. Sử dụng PgAdmin:**
Trong PgAdmin, bạn có thể mở rộng cây điều hướng của bảng (`Databases` -> `your_database` -> `Schemas` -> `public` -> `Tables` -> `your_table` -> `Constraints`). Bạn sẽ thấy danh sách các ràng buộc, bao gồm `PRIMARY KEY`, `FOREIGN KEY` và các ràng buộc `UNIQUE`, `CHECK` mà bạn đã thêm. Tên ràng buộc sẽ là tên bạn đã chỉ định (ví dụ: `unique_product_name`, `check_positive_price`) hoặc một tên tự động tạo bởi PostgreSQL (ví dụ: `products_name_key`, `products_price_check`).

**b. Sử dụng Truy vấn SQL (Nâng cao):**
Để xem chi tiết hơn về các ràng buộc từ dòng lệnh hoặc trong các script, bạn có thể truy vấn các bảng hệ thống của PostgreSQL:

```sql
-- Xem tất cả các ràng buộc trên bảng 'products'
SELECT conname, contype, condef
FROM pg_constraint
WHERE conrelid = 'products'::regclass;
```
*   `conname`: Tên của ràng buộc.
*   `contype`: Loại ràng buộc (p = PRIMARY KEY, u = UNIQUE, f = FOREIGN KEY, c = CHECK).
*   `condef`: Định nghĩa của ràng buộc.

Hoặc sử dụng `information_schema` (tiêu chuẩn SQL):
```sql
SELECT constraint_name, constraint_type, check_clause
FROM information_schema.table_constraints
LEFT JOIN information_schema.check_constraints
       ON table_constraints.constraint_name = check_constraints.constraint_name
WHERE table_name = 'products';
```
Cách này cung cấp thông tin chung, nhưng `pg_constraint` thường cung cấp chi tiết hơn về định nghĩa.

### 3.2. Xóa Các Ràng Buộc

Để xóa một ràng buộc, bạn cần biết tên của ràng buộc đó.

```sql
ALTER TABLE products
DROP CONSTRAINT unique_name_department; -- Thay unique_name_department bằng tên ràng buộc của bạn
```
Sau khi xóa, bạn có thể chèn dữ liệu trùng lặp hoặc không hợp lệ vào các cột đó mà không gặp lỗi.

> [!CAUTION]
> Việc xóa ràng buộc có thể làm mất đi lớp bảo vệ dữ liệu. Hãy cực kỳ cẩn thận khi thực hiện thao tác này trên môi trường sản xuất. **Antigravity IDE** sẽ yêu cầu xác nhận rõ ràng trước khi thực hiện các thay đổi schema như vậy, nhấn mạnh rủi ro tiềm ẩn.

### 3.3. Thêm Ràng Buộc Lớn Mà Không Gây Khóa Bảng (Professional Tip)

Trong môi trường sản xuất với các bảng lớn, việc thêm một ràng buộc `UNIQUE` hoặc `CHECK` có thể yêu cầu PostgreSQL quét toàn bộ bảng để xác thực dữ liệu hiện có. Điều này có thể khóa bảng trong một khoảng thời gian đáng kể, gây ảnh hưởng đến hoạt động của ứng dụng. PostgreSQL cung cấp một cách để thêm ràng buộc mà không cần khóa bảng ngay lập tức:

1.  **Thêm ràng buộc với `NOT VALID`:**
    ```sql
    ALTER TABLE products
    ADD CONSTRAINT unique_product_name UNIQUE (name) NOT VALID;
    ```
    Hoặc
    ```sql
    ALTER TABLE products
    ADD CONSTRAINT check_positive_price CHECK (price > 0) NOT VALID;
    ```
    Lệnh này sẽ tạo ràng buộc và một chỉ mục duy nhất (đối với `UNIQUE`), nhưng nó sẽ *không* kiểm tra dữ liệu hiện có. Nó sẽ chỉ kiểm tra các hàng mới được chèn hoặc cập nhật *sau* thời điểm này. Bảng sẽ không bị khóa trong quá trình này.

2.  **Xác thực ràng buộc sau:**
    Sau đó, bạn có thể chạy lệnh để xác thực dữ liệu hiện có mà vẫn cho phép các hoạt động đọc/ghi trên bảng:
    ```sql
    ALTER TABLE products
    VALIDATE CONSTRAINT unique_product_name;
    ```
    Lệnh này sẽ quét bảng để kiểm tra dữ liệu hiện có mà không khóa bảng cho các hoạt động khác. Nếu có dữ liệu vi phạm, lệnh sẽ thất bại. Khi hoàn tất, ràng buộc sẽ được kích hoạt đầy đủ.

Đây là một kỹ thuật nâng cao rất hữu ích khi làm việc với các hệ thống lớn, và **Antigravity IDE** có thể tự động áp dụng chiến lược này khi bạn yêu cầu nó thêm ràng buộc vào một bảng có kích thước đáng kể.

## 4. Vị Trí Áp Dụng Xác Thực: Cơ Sở Dữ Liệu Hay Tầng Ứng Dụng?

Khi nói đến việc xác thực dữ liệu, một câu hỏi quan trọng thường được đặt ra là: Chúng ta nên tập trung việc xác thực ở đâu? Tại tầng cơ sở dữ liệu hay tại tầng ứng dụng (máy chủ web/client)? Câu trả lời lý tưởng là **áp dụng xác thực ở cả hai tầng**, nhưng với các mục đích và loại quy tắc khác nhau. Đây chính là bản chất của "Vibe Coding" khi thiết kế hệ thống đáng tin cậy.

### 4.1. Xác Thực Tại Tầng Ứng Dụng (Máy Chủ Web/Client)

Xác thực tại tầng ứng dụng, thường là trên máy chủ web (sử dụng Node.js, Java, Python, v.v.) hoặc thậm chí trực tiếp trên trình duyệt (client-side), mang lại nhiều lợi ích và thường là nơi đầu tiên dữ liệu được kiểm tra.

#### Ưu điểm:
*   **Xử lý các xác thực phức tạp:** Dễ dàng thực hiện các quy tắc xác thực phức tạp yêu cầu logic nghiệp vụ phức tạp, truy vấn các API bên ngoài (ví dụ: kiểm tra giá cổ phiếu hiện tại, xác thực địa chỉ qua dịch vụ bên thứ ba), hoặc các phép tính phức tạp không phù hợp với SQL `CHECK` đơn giản.
*   **Dễ thay đổi quy tắc:** Việc thay đổi một quy tắc xác thực trong mã ứng dụng thường dễ dàng và nhanh chóng hơn. Bạn chỉ cần cập nhật mã, triển khai lại ứng dụng mà không cần phải thực hiện các thay đổi cấu trúc cơ sở dữ liệu có thể tiềm ẩn rủi ro.
*   **Thư viện hỗ trợ phong phú:** Hầu hết các ngôn ngữ lập trình đều có các thư viện xác thực mạnh mẽ (ví dụ: kiểm tra định dạng email, số điện thoại, độ dài chuỗi, biểu thức chính quy) giúp giảm thiểu công sức tự viết mã.
*   **Phản hồi ngay lập tức cho người dùng:** Xác thực ở tầng ứng dụng (hoặc thậm chí tầng client-side) cho phép cung cấp phản hồi lỗi ngay lập tức cho người dùng, cải thiện trải nghiệm người dùng mà không cần phải gửi yêu cầu đến cơ sở dữ liệu. **Antigravity IDE** sẽ tự động sinh mã xác thực ở tầng ứng dụng, giúp bạn tiết kiệm thời gian và đảm bảo phản hồi nhanh chóng.

#### Nhược điểm:
*   **Không đảm bảo toàn vẹn dữ liệu từ mọi nguồn:** Nếu dữ liệu có thể được chèn hoặc cập nhật vào cơ sở dữ liệu từ nhiều nguồn khác nhau (ví dụ: PgAdmin, ứng dụng khác, script batch), thì chỉ xác thực ở tầng ứng dụng sẽ không ngăn chặn dữ liệu không hợp lệ từ các nguồn đó.
*   **Nguy cơ bỏ sót:** Các nhà phát triển có thể vô tình bỏ qua việc áp dụng một số quy tắc xác thực nào đó ở một điểm nào đó trong mã, dẫn đến dữ liệu không hợp lệ.

### 4.2. Xác Thực Tại Tầng Cơ Sở Dữ Liệu

Xác thực tại tầng cơ sở dữ liệu thông qua các ràng buộc (`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, `CHECK`) là lớp phòng thủ cuối cùng và mạnh mẽ nhất. Đây là nơi bạn đặt các "quy tắc bất di bất dịch" của dữ liệu.

#### Ưu điểm:
*   **Đảm bảo toàn vẹn dữ liệu tuyệt đối:** Bất kể dữ liệu đến từ đâu (ứng dụng web, ứng dụng di động, công cụ quản lý CSDL như PgAdmin, script ETL), các quy tắc xác thực sẽ luôn được áp dụng. Điều này là cực kỳ quan trọng đối với các quy tắc cốt lõi, không thể thay đổi.
*   **Dữ liệu hiện có luôn hợp lệ:** Khi bạn thêm một ràng buộc mới vào một bảng hiện có, PostgreSQL sẽ kiểm tra tất cả các dữ liệu hiện có để đảm bảo chúng thỏa mãn ràng buộc đó. Nếu có bất kỳ dữ liệu nào vi phạm, ràng buộc sẽ không được tạo. Điều này đảm bảo rằng cơ sở dữ liệu của bạn không bao giờ chứa dữ liệu không hợp lệ theo quy tắc mới.
*   **Hiệu suất:** Đối với các quy tắc đơn giản, cơ sở dữ liệu có thể thực thi các ràng buộc một cách hiệu quả hơn so với việc phải chạy logic xác thực tương tự trong ứng dụng.
*   **Tính nhất quán:** Đảm bảo rằng tất cả các ứng dụng tương tác với cơ sở dữ liệu đều tuân thủ cùng một bộ quy tắc dữ liệu cơ bản.

#### Nhược điểm:
*   **Khó thay đổi hơn:** Việc thay đổi hoặc thêm các ràng buộc vào cơ sở dữ liệu đang hoạt động (đặc biệt trong môi trường sản xuất) thường đòi hỏi kế hoạch cẩn thận và có thể liên quan đến các kịch bản di chuyển dữ liệu phức tạp.
*   **Hạn chế về logic:** Các ràng buộc `CHECK` có giới hạn về độ phức tạp của logic mà chúng có thể xử lý (không thể thực hiện truy vấn phụ, không thể gọi API bên ngoài).
*   **Thông báo lỗi ít thân thiện:** Thông báo lỗi từ cơ sở dữ liệu thường mang tính kỹ thuật và cần được dịch sang ngôn ngữ thân thiện hơn cho người dùng cuối ở tầng ứng dụng.

### 4.3. Mô Hình Tối Ưu: Chiến Lược Phân Tán Xác Thực (Antigravity & Vibe Coding)

Chiến lược tốt nhất là kết hợp cả hai phương pháp, phân tán các loại xác thực khác nhau đến nơi chúng phù hợp nhất. Đây là một ví dụ điển hình của "Vibe Coding" khi bạn đang cộng tác với **Antigravity IDE**. Antigravity, với khả năng lập kế hoạch và thực thi script ngầm, sẽ tự động áp dụng mô hình này, nhưng bạn cần hiểu triết lý đằng sau để hướng dẫn nó hiệu quả.

**Với tư duy "Vibe Coding" và sự hỗ trợ của Antigravity:**

*   **Tại tầng ứng dụng (do Antigravity sinh mã):**
    *   **Xác thực "nhẹ" hoặc phức tạp:** Kiểm tra độ dài tối thiểu/tối đa của `username`, định dạng email, kiểm tra mật khẩu phức tạp, gọi API xác thực bên ngoài (ví dụ: kiểm tra tính hợp lệ của mã bưu điện).
    *   **Phản hồi ngay lập tức:** Cung cấp thông báo lỗi thân thiện cho người dùng.
    *   **Quy tắc nghiệp vụ linh hoạt:** Các quy tắc có thể thay đổi thường xuyên hoặc phụ thuộc vào trạng thái ứng dụng.

*   **Tại tầng cơ sở dữ liệu (do Antigravity thiết kế schema):**
    *   **Ràng buộc "cốt lõi" và "quan trọng":** Những quy tắc mà dữ liệu phải tuân thủ trong mọi trường hợp, bất kể nguồn gốc. Đây là lớp bảo vệ cuối cùng.
    *   **Ví dụ điển hình:**
        *   `PRIMARY KEY` cho các định danh duy nhất không thể thiếu.
        *   `FOREIGN KEY` để đảm bảo tính toàn vẹn tham chiếu giữa các bảng.
        *   `UNIQUE` cho các thuộc tính phải là duy nhất (ví dụ: `email` người dùng, `SKU` sản phẩm).
        *   `NOT NULL` cho các cột bắt buộc.
        *   `CHECK` cho các phạm vi giá trị cơ bản và không thể thiếu (ví dụ: `price > 0`, `age >= 18`, `delivery_date > order_date`).

**Ví dụ cụ thể về tư duy "Vibe Coding" với Antigravity:**

Khi bạn yêu cầu Antigravity xây dựng một hệ thống thương mại điện tử:

*   **Antigravity** sẽ tự động nhận ra rằng `email` của người dùng phải là duy nhất và không được `NULL` (`email VARCHAR(255) UNIQUE NOT NULL`) ở cấp CSDL. Đây là quy tắc cốt lõi.
*   Nó cũng sẽ tạo ra một ràng buộc `CHECK (price > 0)` cho cột giá sản phẩm, vì giá âm là phi lý trong mọi trường hợp.
*   Tuy nhiên, việc kiểm tra xem `username` có độ dài từ 4 đến 20 ký tự hay không, hoặc mật khẩu có chứa ít nhất một ký tự đặc biệt không, sẽ được Antigravity triển khai ở tầng ứng dụng (client-side và server-side). Tại sao? Bởi vì những quy tắc này có thể thay đổi để cải thiện trải nghiệm người dùng hoặc chính sách bảo mật mà không cần can thiệp vào schema CSDL.
*   Nếu bạn thử chèn một sản phẩm có giá âm thông qua PgAdmin, CSDL sẽ từ chối. Nếu bạn cố gắng đăng ký với một email đã tồn tại qua ứng dụng, tầng ứng dụng có thể báo lỗi ngay lập tức, nhưng nếu bằng cách nào đó qua được, CSDL vẫn sẽ từ chối.

Bằng cách này, bạn có thể tận dụng tối đa sức mạnh của cả hai tầng để xây dựng một hệ thống mạnh mẽ, linh hoạt và đáng tin cậy. **Antigravity IDE** sẽ là đối tác lý tưởng giúp bạn hiện thực hóa chiến lược phân tán xác thực này một cách hiệu quả nhất.

## Tóm Tắt Phần 24: Ràng Buộc UNIQUE và CHECK

*   **Ràng buộc `UNIQUE`** đảm bảo tính duy nhất của dữ liệu trong một cột hoặc một nhóm các cột, ngăn chặn sự trùng lặp.
    *   Cơ chế hoạt động ngầm là tạo **chỉ mục duy nhất**.
    *   Có thể áp dụng trên **một cột đơn** (ví dụ: `name UNIQUE`) hoặc **nhiều cột** (ví dụ: `UNIQUE (name, department)`).
    *   PostgreSQL cho phép nhiều giá trị `NULL` trong một cột `UNIQUE` (trừ khi có thêm `NOT NULL`).
    *   Để thêm ràng buộc `UNIQUE` vào một bảng hiện có, tất cả dữ liệu trong cột/nhóm cột đó phải đã là duy nhất.
    *   Sử dụng `ALTER TABLE ... ADD CONSTRAINT ... UNIQUE (...)` để thêm và `ALTER TABLE ... DROP CONSTRAINT ...` để xóa.
    *   Đối với bảng lớn, có thể dùng `NOT VALID` và `VALIDATE CONSTRAINT` để tránh khóa bảng.
*   **Ràng buộc `CHECK`** cho phép định nghĩa các quy tắc kiểm tra điều kiện mà dữ liệu phải thỏa mãn.
    *   Chỉ kiểm tra các giá trị trong hàng đang được chèn/cập nhật và **không thể thực hiện truy vấn phụ**.
    *   Có thể áp dụng trên **một cột đơn** (ví dụ: `price NUMERIC CHECK (price > 0)`) hoặc **nhiều cột** trong cùng một hàng (ví dụ: `CHECK (estimated_delivery_time > created_at)`).
    *   Để thêm ràng buộc `CHECK` vào một bảng hiện có, tất cả dữ liệu hiện có phải thỏa mãn điều kiện kiểm tra.
*   **Quản lý ràng buộc:** Nên đặt tên rõ ràng cho các ràng buộc để dễ dàng xem và xóa chúng thông qua `ALTER TABLE`. Có thể truy vấn `pg_constraint` hoặc `information_schema.table_constraints`.
*   **Vị trí xác thực:**
    *   **Tầng ứng dụng (Web Server/Client):** Tốt cho các xác thực phức tạp, yêu cầu tương tác bên ngoài, dễ thay đổi, và cung cấp phản hồi nhanh cho người dùng.
    *   **Tầng cơ sở dữ liệu:** Cung cấp lớp bảo vệ cuối cùng, đảm bảo tính toàn vẹn dữ liệu từ mọi nguồn, và đảm bảo dữ liệu hiện có luôn hợp lệ khi thêm ràng buộc mới.
    *   **Chiến lược tối ưu** là kết hợp cả hai: Xác thực quan trọng, cốt lõi (như `price > 0`, `UNIQUE email`) ở CSDL; xác thực phức tạp hoặc ít nghiêm trọng hơn ở tầng ứng dụng.
*   **Antigravity IDE và Vibe Coding:** **Antigravity** sẽ tự động áp dụng chiến lược phân tán xác thực này. Việc hiểu rõ cơ chế và mục đích của từng ràng buộc giúp bạn thực hiện "Vibe Coding" hiệu quả, hướng dẫn AI thiết kế và triển khai hệ thống dữ liệu mạnh mẽ và đáng tin cậy.

<!-- REVIEWED_BY_AGENT -->
