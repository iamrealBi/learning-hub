# Bài 6: Các Ràng Buộc Khóa Ngoại Nâng Cao trong PostgreSQL

## Giới Thiệu Chương

Trong thế giới của cơ sở dữ liệu quan hệ, việc tổ chức và liên kết dữ liệu một cách logic là nền tảng để xây dựng các hệ thống mạnh mẽ và đáng tin cậy. Chương này sẽ đưa bạn đi sâu vào một trong những cơ chế quan trọng nhất để đạt được mục tiêu đó: Khóa Ngoại (Foreign Key) và các ràng buộc nâng cao đi kèm. Khóa ngoại không chỉ đơn thuần là một công cụ để kết nối các bảng; chúng là trái tim của tính toàn vẹn tham chiếu, đảm bảo rằng dữ liệu của bạn luôn nhất quán và không bị rơi vào trạng thái "mồ côi" hay vô nghĩa.

Với tư cách là một chuyên gia lập trình cấp Senior, việc nắm vững các ràng buộc khóa ngoại không chỉ giúp bạn thiết kế schema cơ sở dữ liệu vững chắc mà còn cho phép bạn dự đoán và kiểm soát hành vi của hệ thống khi dữ liệu được thêm, sửa, hoặc xóa. Chúng ta sẽ đặc biệt tập trung vào cú pháp và các tính năng của PostgreSQL, một hệ quản trị cơ sở dữ liệu mạnh mẽ và tuân thủ chuẩn SQL.

**Mục tiêu của chương này:**
*   Nắm vững định nghĩa và tầm quan trọng cốt lõi của khóa ngoại trong thiết kế CSDL.
*   Thành thạo việc khai báo và quản lý các ràng buộc khóa ngoại trong PostgreSQL.
*   Hiểu và áp dụng các chính sách hành động khi xóa dữ liệu (`ON DELETE`) để duy trì tính toàn vẹn.
*   Xây dựng và truy vấn các mô hình dữ liệu phức tạp với nhiều khóa ngoại.
*   Liên hệ tư duy thiết kế CSDL với phương pháp Vibe Coding và cách một hệ thống Agentic AI như Antigravity IDE có thể hỗ trợ hiệu quả.

Chúng ta sẽ sử dụng các ví dụ thực tế dựa trên một ứng dụng chia sẻ ảnh để minh họa từng khái niệm, đảm bảo bạn có thể áp dụng ngay lập tức vào các dự án của mình.

## 1. Nền Tảng Khóa Ngoại: Cầu Nối Dữ Liệu và Tính Toàn Vẹn

### 1.1 Khóa Ngoại là gì và Tại sao lại Quan trọng?

Trong một hệ thống cơ sở dữ liệu quan hệ được thiết kế chuẩn mực, dữ liệu thường được phân tách thành nhiều bảng chuyên biệt để loại bỏ sự trùng lặp (redundancy) và tối ưu hóa hiệu suất lưu trữ cũng như truy vấn. Ví dụ, thông tin về người dùng (users) sẽ nằm trong một bảng, và thông tin về các bức ảnh (photos) mà họ đăng tải sẽ nằm trong một bảng khác.

**Khóa ngoại (Foreign Key - FK)** là một cột (hoặc tập hợp các cột) trong một bảng (gọi là **bảng con** hoặc **bảng tham chiếu**) mà giá trị của nó phải tương ứng với giá trị của một cột khóa chính (hoặc khóa ứng cử viên) trong một bảng khác (gọi là **bảng cha** hoặc **bảng được tham chiếu**).

**Vai trò cốt lõi của khóa ngoại:**
*   **Đảm bảo tính toàn vẹn tham chiếu (Referential Integrity):** Đây là mục đích chính yếu. Khóa ngoại đảm bảo rằng mọi "tham chiếu" từ bảng con đến bảng cha đều hợp lệ. Nói cách khác, một bản ghi trong bảng con không thể trỏ đến một bản ghi không tồn tại trong bảng cha. Điều này ngăn chặn tình trạng dữ liệu "mồ côi" hoặc "lơ lửng", nơi một bản ghi con không có bản ghi cha tương ứng.
*   **Thiết lập mối quan hệ giữa các bảng:** Khóa ngoại là cơ chế vật lý để biểu diễn các mối quan hệ logic như "một-nhiều" (ví dụ: một người dùng có thể đăng nhiều ảnh) hoặc "nhiều-nhiều" (thông qua một bảng trung gian).
*   **Cơ sở cho các phép nối (JOIN):** Khi truy vấn dữ liệu, khóa ngoại cung cấp điểm nối tự nhiên và hiệu quả để kết hợp thông tin từ nhiều bảng liên quan.

### 1.2 Cú pháp tạo Khóa Ngoại trong PostgreSQL

PostgreSQL cung cấp cú pháp rõ ràng và mạnh mẽ để định nghĩa khóa ngoại ngay trong câu lệnh `CREATE TABLE`.

**Cú pháp cơ bản:**
```sql
CREATE TABLE child_table (
    child_id SERIAL PRIMARY KEY,
    -- ... các cột khác ...
    foreign_key_column INTEGER REFERENCES parent_table(parent_primary_key_column)
    -- Hoặc với khai báo rõ ràng hơn
    -- foreign_key_column INTEGER,
    -- CONSTRAINT fk_name FOREIGN KEY (foreign_key_column) REFERENCES parent_table(parent_primary_key_column)
);
```
*   `foreign_key_column`: Cột trong bảng con sẽ giữ giá trị khóa ngoại. Kiểu dữ liệu của nó phải tương thích với kiểu dữ liệu của cột khóa chính trong bảng cha.
*   `parent_table`: Tên của bảng cha.
*   `parent_primary_key_column`: Tên của cột khóa chính (hoặc khóa ứng cử viên duy nhất) trong bảng cha mà khóa ngoại này tham chiếu đến.

### 1.3 Ví dụ Thực tế: Ứng dụng Chia sẻ Ảnh

Hãy xây dựng một mô hình cơ sở dữ liệu đơn giản cho một ứng dụng chia sẻ ảnh để minh họa các khái niệm. Chúng ta sẽ có hai bảng chính: `users` (người dùng) và `photos` (ảnh).

**Bước 1: Tạo bảng `users` (Bảng Cha)**

Bảng `users` sẽ lưu trữ thông tin cơ bản của người dùng. Cột `id` sẽ là khóa chính, tự động tăng giá trị. Trong PostgreSQL, kiểu dữ liệu `SERIAL` được thiết kế đặc biệt cho mục đích này, tương đương với việc tạo một `SEQUENCE` và gán giá trị mặc định cho cột.

```sql
-- Đảm bảo môi trường sạch trước khi tạo bảng
DROP TABLE IF EXISTS photos CASCADE; -- CASCADE sẽ xóa các bảng phụ thuộc
DROP TABLE IF EXISTS users;

-- Tạo bảng users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,              -- Khóa chính, tự động tăng giá trị
    username VARCHAR(50) NOT NULL UNIQUE, -- Tên người dùng, không được để trống và phải là duy nhất
    email VARCHAR(100) UNIQUE           -- Email người dùng, có thể NULL nhưng nếu có thì phải duy nhất
);

-- Chèn dữ liệu mẫu vào bảng users
INSERT INTO users (username, email) VALUES
('monahan', 'monahan@example.com'),
('alex', 'alex@example.com'),
('sarah', 'sarah@example.com'),
('john', 'john@example.com');

-- Kiểm tra dữ liệu
SELECT * FROM users;
```
**Giải thích `SERIAL`:** `SERIAL` trong PostgreSQL không phải là một kiểu dữ liệu thực sự mà là một "pseudo-type". Nó tự động tạo một sequence (chuỗi số) và đặt giá trị mặc định cho cột từ sequence đó, đồng thời thêm một ràng buộc `NOT NULL` và `OWNED BY` để quản lý sequence. Điều này đơn giản hóa việc tạo khóa chính tự động tăng mà không cần phải quản lý sequence thủ công.

**Bước 2: Tạo bảng `photos` (Bảng Con) với Khóa Ngoại**

Bảng `photos` sẽ chứa thông tin về các bức ảnh. Mỗi bức ảnh phải được liên kết với một người dùng cụ thể. Cột `user_id` trong bảng `photos` sẽ là khóa ngoại, tham chiếu đến cột `id` trong bảng `users`.

```sql
-- Tạo bảng photos với khóa ngoại tham chiếu đến bảng users
CREATE TABLE photos (
    id SERIAL PRIMARY KEY,              -- Khóa chính, tự động tăng
    url VARCHAR(255) NOT NULL,          -- URL của bức ảnh, không được để trống
    caption TEXT,                       -- Mô tả ảnh (có thể NULL)
    user_id INTEGER REFERENCES users(id) -- Khóa ngoại: user_id tham chiếu đến id của bảng users
);

-- Chèn dữ liệu mẫu vào bảng photos
INSERT INTO photos (url, caption, user_id) VALUES
('http://example.com/photo1.jpg', 'Ảnh hoàng hôn tuyệt đẹp', 1),
('http://example.com/photo2.jpg', 'Bữa sáng lành mạnh', 1),
('http://example.com/photo3.jpg', 'Chuyến đi biển', 1),
('http://example.com/photo4.jpg', 'Thú cưng của tôi', 2),
('http://example.com/photo5.jpg', 'Khám phá thành phố', 3),
('http://example.com/photo6.jpg', 'Sách hay cho cuối tuần', 4),
('http://example.com/photo7.jpg', 'Cafe buổi sáng', 4),
('http://example.com/photo8.jpg', 'Thiên nhiên hùng vĩ', 1);

-- Kiểm tra dữ liệu
SELECT * FROM photos;
```
**Cơ chế hoạt động của `REFERENCES`:** Khi bạn thêm `REFERENCES users(id)`, PostgreSQL sẽ tự động tạo một ràng buộc khóa ngoại. Từ thời điểm này, bất kỳ thao tác `INSERT` hoặc `UPDATE` nào trên cột `photos.user_id` đều sẽ được kiểm tra. Nếu giá trị `user_id` không tồn tại trong `users.id`, thao tác sẽ bị từ chối và một lỗi `foreign key constraint violation` sẽ được trả về. Điều này đảm bảo rằng không có bức ảnh nào "thuộc về" một người dùng không tồn tại.

## 2. Tương Tác Dữ Liệu Với Khóa Ngoại

Khóa ngoại không chỉ là một cơ chế ràng buộc; chúng còn là xương sống cho việc truy vấn và thao tác dữ liệu liên quan một cách hiệu quả.

### 2.1 Đảm bảo Tính Toàn Vẹn khi Chèn Dữ liệu (`INSERT`)

Ràng buộc khóa ngoại hoạt động ngay lập tức khi bạn cố gắng chèn hoặc cập nhật dữ liệu.
1.  **Chèn thành công (tham chiếu hợp lệ):**
    ```sql
    -- Chèn một ảnh với user_id hợp lệ (ví dụ: user_id 2 tồn tại)
    INSERT INTO photos (url, caption, user_id) VALUES
    ('http://example.com/new_photo_by_alex.jpg', 'Ảnh mới của Alex', 2);
    ```
2.  **Chèn thất bại (tham chiếu không hợp lệ):**
    ```sql
    -- Thử chèn một bức ảnh với user_id không tồn tại (ví dụ: user_id 9999)
    INSERT INTO photos (url, caption, user_id) VALUES
    ('http://example.com/invalid.jpg', 'Ảnh lỗi', 9999);
    ```
    Bạn sẽ nhận được lỗi tương tự:
    `ERROR: insert or update on table "photos" violates foreign key constraint "photos_user_id_fkey"`
    `DETAIL: Key (user_id)=(9999) is not present in table "users".`
    Thông báo lỗi này rất rõ ràng, cho biết giá trị khóa ngoại `9999` không tồn tại trong bảng `users`.

3.  **Chèn với `NULL` (nếu cho phép):**
    Nếu cột khóa ngoại không có ràng buộc `NOT NULL`, bạn có thể chèn giá trị `NULL` vào đó. Điều này có nghĩa là bản ghi con không được liên kết với bất kỳ bản ghi cha cụ thể nào.
    ```sql
    -- Chèn một bức ảnh không liên kết với bất kỳ người dùng nào
    -- (Giả định user_id không có ràng buộc NOT NULL, mặc định là có thể NULL)
    INSERT INTO photos (url, caption, user_id) VALUES
    ('http://example.com/no_user.jpg', 'Ảnh không có người đăng', NULL);
    SELECT * FROM photos WHERE url = 'http://example.com/no_user.jpg';
    ```
    Đây là một thiết kế hữu ích trong nhiều trường hợp, ví dụ, khi một tài khoản người dùng có thể bị xóa nhưng các nội dung họ tạo ra vẫn được giữ lại dưới dạng "người dùng ẩn danh" hoặc "chưa được gán".

### 2.2 Truy vấn Dữ liệu Liên quan với `JOIN`

Khóa ngoại là cơ sở để kết hợp dữ liệu từ nhiều bảng thông qua các toán tử `JOIN`. `JOIN` cho phép bạn tạo ra một tập hợp kết quả duy nhất bằng cách ghép các hàng từ hai hoặc nhiều bảng dựa trên một điều kiện liên kết.

**`INNER JOIN` - Kết hợp dữ liệu trùng khớp:**
`INNER JOIN` là loại `JOIN` phổ biến nhất, nó chỉ trả về các hàng khi có sự trùng khớp ở cả hai bảng được nối.

**Ví dụ:** Lấy URL của ảnh và tên người dùng tương ứng của người đã đăng ảnh đó.
Chúng ta cần kết hợp thông tin từ bảng `photos` (có `url`) và bảng `users` (có `username`) dựa trên điều kiện `photos.user_id = users.id`.

```sql
-- Lấy URL ảnh và tên người dùng đã đăng ảnh
SELECT
    p.url,          -- URL từ bảng photos
    p.caption,      -- Mô tả ảnh từ bảng photos
    u.username      -- Tên người dùng từ bảng users
FROM
    photos AS p     -- Bảng photos, đặt bí danh là 'p'
INNER JOIN
    users AS u ON p.user_id = u.id; -- Kết hợp với bảng users (bí danh 'u')
                                    -- Điều kiện kết hợp: user_id của ảnh phải bằng id của người dùng
```
**Sử dụng bí danh (Alias):** Việc sử dụng bí danh như `p` cho `photos` và `u` cho `users` giúp truy vấn ngắn gọn hơn, dễ đọc hơn và tránh xung đột tên cột nếu cả hai bảng có cột trùng tên (ví dụ: cả `users` và `photos` đều có cột `id`).

**Các loại `JOIN` khác:** Mặc dù `INNER JOIN` là đủ cho ví dụ này, PostgreSQL hỗ trợ các loại `JOIN` khác như `LEFT JOIN`, `RIGHT JOIN`, `FULL JOIN`, và `CROSS JOIN`. Mỗi loại có mục đích riêng để xử lý các trường hợp khi không có sự trùng khớp ở một trong các bảng:
*   `LEFT JOIN`: Trả về tất cả các hàng từ bảng bên trái, và các hàng trùng khớp từ bảng bên phải (nếu không có trùng khớp, các cột của bảng bên phải sẽ là `NULL`).
*   `RIGHT JOIN`: Tương tự như `LEFT JOIN`, nhưng ưu tiên bảng bên phải.
*   `FULL JOIN`: Trả về tất cả các hàng khi có sự trùng khớp ở một trong hai bảng (nếu không có trùng khớp, các cột của bảng còn lại sẽ là `NULL`).
*   `CROSS JOIN`: Trả về tích Descartes của hai bảng (mỗi hàng của bảng thứ nhất kết hợp với mỗi hàng của bảng thứ hai).

Hiểu biết về các loại `JOIN` này là rất quan trọng để có thể truy vấn dữ liệu một cách linh hoạt và chính xác trong các tình huống phức tạp hơn.

## 3. Quản lý Hành vi Xóa Dữ Liệu với `ON DELETE`

Một trong những khía cạnh mạnh mẽ và quan trọng nhất của ràng buộc khóa ngoại là khả năng kiểm soát điều gì sẽ xảy ra với các bản ghi con khi bản ghi cha của chúng bị xóa. Nếu không có cơ chế này, việc xóa một bản ghi cha có thể dẫn đến các bản ghi con "mồ côi", gây mất tính toàn vẹn dữ liệu.

PostgreSQL cung cấp các tùy chọn `ON DELETE` để xác định chính sách hành vi này. Bạn định nghĩa chúng khi tạo ràng buộc khóa ngoại:

### 3.1 `ON DELETE RESTRICT` và `ON DELETE NO ACTION` (Mặc định)

*   **Hành vi:** Ngăn chặn việc xóa bản ghi cha nếu có bất kỳ bản ghi con nào đang tham chiếu đến nó. Thao tác xóa sẽ bị từ chối và gây ra lỗi.
*   **Khi nào sử dụng:** Khi bạn muốn đảm bảo rằng không có bản ghi cha nào bị xóa một cách vô tình mà vẫn còn các bản ghi con liên quan. Đây là lựa chọn an toàn nhất và thường là mặc định.
*   **Sự khác biệt giữa `RESTRICT` và `NO ACTION`:**
    *   `RESTRICT`: Kiểm tra ràng buộc ngay lập tức.
    *   `NO ACTION`: Kiểm tra ràng buộc vào cuối giao dịch (transaction).
    Trong hầu hết các trường hợp thực tế, chúng có cùng hiệu quả là ngăn chặn việc xóa nếu có bản ghi con. PostgreSQL mặc định sử dụng `NO ACTION` nếu bạn không chỉ định gì, nhưng về mặt hành vi, nó rất giống `RESTRICT` đối với các thao tác `DELETE` đơn lẻ.

**Ví dụ:** Sử dụng ràng buộc mặc định (hoặc `ON DELETE RESTRICT`)
```sql
-- Nếu bảng photos đã được tạo với ON DELETE CASCADE/SET NULL, hãy tạo lại nó
DROP TABLE IF EXISTS photos;
CREATE TABLE photos (
    id SERIAL PRIMARY KEY,
    url VARCHAR(255) NOT NULL,
    user_id INTEGER REFERENCES users(id) -- Mặc định là ON DELETE NO ACTION
);
INSERT INTO photos (url, user_id) VALUES
('http://example.com/restrict_photo1.jpg', 1),
('http://example.com/restrict_photo2.jpg', 1);

-- Thử xóa người dùng có ID 1 (người dùng 'monahan')
-- Lệnh này sẽ gây lỗi vì có ảnh liên quan
DELETE FROM users WHERE id = 1;
```
Bạn sẽ nhận được lỗi: `ERROR: update or delete on table "users" violates foreign key constraint "photos_user_id_fkey" on table "photos"`

### 3.2 `ON DELETE CASCADE`

*   **Hành vi:** Khi một bản ghi cha bị xóa, tất cả các bản ghi con liên quan trực tiếp đến nó cũng sẽ tự động bị xóa theo.
*   **Khi nào sử dụng:** Khi dữ liệu con không có ý nghĩa hoặc không nên tồn tại nếu không có bản ghi cha. Đây là lựa chọn phổ biến cho các mối quan hệ sở hữu mạnh mẽ.
*   **Cảnh báo:** Hãy thận trọng khi sử dụng `CASCADE`, vì nó có thể dẫn đến việc xóa hàng loạt dữ liệu không mong muốn nếu không được hiểu rõ.

**Ví dụ:** Xóa một người dùng và tất cả ảnh của người đó.
```sql
-- Bước 1: Xóa bảng photos hiện có để tạo lại với ràng buộc mới
DROP TABLE IF EXISTS photos;

-- Bước 2: Tạo lại bảng photos với ON DELETE CASCADE
CREATE TABLE photos (
    id SERIAL PRIMARY KEY,
    url VARCHAR(255) NOT NULL,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE -- Khi user bị xóa, các ảnh liên quan cũng bị xóa
);

-- Bước 3: Chèn lại dữ liệu mẫu vào photos
-- Đảm bảo user có ID 1, 2 tồn tại. (Nếu bạn đã xóa user 1 ở ví dụ trên, hãy chèn lại)
INSERT INTO photos (url, user_id) VALUES
('http://example.com/cascade_photo1.jpg', 1),
('http://example.com/cascade_photo2.jpg', 1),
('http://example.com/cascade_photo3.jpg', 2);

-- Bước 4: Xóa người dùng có ID 1
DELETE FROM users WHERE id = 1;

-- Bước 5: Kiểm tra xem các ảnh liên quan có bị xóa không
SELECT * FROM photos; -- Bạn sẽ thấy không còn bức ảnh nào có user_id là 1
```
Trong ví dụ này, khi người dùng có `id = 1` bị xóa, hai bức ảnh có `user_id = 1` cũng tự động biến mất khỏi bảng `photos`.

### 3.3 `ON DELETE SET NULL`

*   **Hành vi:** Khi một bản ghi cha bị xóa, giá trị khóa ngoại trong tất cả các bản ghi con liên quan sẽ được đặt thành `NULL`.
*   **Yêu cầu:** Cột khóa ngoại trong bảng con *phải được phép chứa giá trị `NULL`* (tức là không có ràng buộc `NOT NULL`).
*   **Khi nào sử dụng:** Khi bạn muốn giữ lại dữ liệu con ngay cả khi bản ghi cha bị xóa, nhưng muốn ngắt liên kết của chúng. Thường được dùng khi dữ liệu con vẫn có ý nghĩa độc lập hoặc cần được xử lý sau.

**Ví dụ:** Xóa một người dùng, nhưng vẫn giữ lại các bức ảnh của họ và gán chúng thành "không có người dùng".
```sql
-- Bước 1: Xóa bảng photos hiện có
DROP TABLE IF EXISTS photos;

-- Bước 2: Tạo lại bảng photos với ON DELETE SET NULL
-- Cột user_id không được có NOT NULL để SET NULL có thể hoạt động
CREATE TABLE photos (
    id SERIAL PRIMARY KEY,
    url VARCHAR(255) NOT NULL,
    user_id INTEGER REFERENCES users(id) ON DELETE SET NULL -- Khi user bị xóa, user_id trong ảnh sẽ thành NULL
);

-- Bước 3: Đảm bảo có đủ người dùng cho ví dụ này (ID 1, 2)
-- Nếu user 1 đã bị xóa, chèn lại hoặc dùng user_id khác
INSERT INTO users (username) VALUES ('monahan_reinserted'); -- Sẽ có ID mới, ví dụ 5
UPDATE users SET username = 'monahan' WHERE id = 1; -- Nếu muốn giữ ID 1

-- Chèn dữ liệu mẫu vào photos, liên kết với user_id 1 và 2
INSERT INTO photos (url, user_id) VALUES
('http://example.com/setnull_photo1.jpg', 1),
('http://example.com/setnull_photo2.jpg', 1),
('http://example.com/setnull_photo3.jpg', 2);

-- Bước 4: Xóa người dùng có ID 1
DELETE FROM users WHERE id = 1;

-- Bước 5: Kiểm tra xem user_id của các ảnh liên quan có được đặt thành NULL không
SELECT * FROM photos; -- Bạn sẽ thấy hai bức ảnh đầu tiên giờ đây có user_id là NULL
```
Sau khi xóa người dùng `id = 1`, các bức ảnh của họ vẫn tồn tại nhưng cột `user_id` của chúng đã được cập nhật thành `NULL`, cho thấy chúng không còn liên kết với một người dùng cụ thể nào.

### 3.4 `ON DELETE SET DEFAULT`

*   **Hành vi:** Khi một bản ghi cha bị xóa, giá trị khóa ngoại trong các bản ghi con liên quan sẽ được đặt thành giá trị mặc định đã được định nghĩa cho cột đó.
*   **Yêu cầu:**
    1.  Cột khóa ngoại phải có một ràng buộc `DEFAULT`.
    2.  Giá trị mặc định đó *phải tồn tại* như một khóa chính trong bảng cha.
*   **Khi nào sử dụng:** Khi bạn muốn gán lại các bản ghi con cho một bản ghi cha "mặc định" hoặc "placeholder" sau khi bản ghi cha ban đầu bị xóa.

**Ví dụ:** (Minh họa cú pháp và yêu cầu)
Để sử dụng `ON DELETE SET DEFAULT`, chúng ta cần một người dùng mặc định trong bảng `users` (ví dụ: một người dùng "Ẩn danh" với ID cố định) và cột `user_id` trong `photos` phải có giá trị `DEFAULT`.

```sql
-- Bước 1: Đảm bảo có một người dùng mặc định trong bảng users
-- Ví dụ, chúng ta sẽ tạo một user 'anonymous' với ID 9999 (hoặc một ID đặc biệt khác)
-- Đầu tiên, xóa bảng users và photos để đảm bảo môi trường sạch
DROP TABLE IF EXISTS comments;
DROP TABLE IF EXISTS photos;
DROP TABLE IF EXISTS users CASCADE;

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE
);
INSERT INTO users (id, username) VALUES
(1, 'monahan_default'),
(2, 'alex_default'),
(3, 'sarah_default'),
(9999, 'anonymous'); -- Người dùng mặc định

-- Bước 2: Tạo lại bảng photos với ON DELETE SET DEFAULT
-- user_id có giá trị mặc định là 9999, và tham chiếu đến user 9999
CREATE TABLE photos (
    id SERIAL PRIMARY KEY,
    url VARCHAR(255) NOT NULL,
    user_id INTEGER DEFAULT 9999 REFERENCES users(id) ON DELETE SET DEFAULT
);

-- Bước 3: Chèn dữ liệu mẫu vào photos, liên kết với user_id 1 và 2
INSERT INTO photos (url, user_id) VALUES
('http://example.com/setdefault_photo1.jpg', 1),
('http://example.com/setdefault_photo2.jpg', 1),
('http://example.com/setdefault_photo3.jpg', 2);

-- Bước 4: Xóa người dùng có ID 1
DELETE FROM users WHERE id = 1;

-- Bước 5: Kiểm tra xem user_id của các ảnh liên quan có được đặt thành giá trị mặc định không
SELECT * FROM photos;
```
Trong trường hợp này, hai bức ảnh của người dùng `id = 1` sẽ được gán lại `user_id = 9999`, liên kết chúng với người dùng "anonymous".

## 4. Mở Rộng Mô Hình Dữ Liệu & Tư Duy Thiết Kế

Khóa ngoại cho phép chúng ta xây dựng các mô hình dữ liệu phức tạp, phản ánh các mối quan hệ đa chiều trong thế giới thực. Việc hiểu rõ cách các bảng liên kết với nhau là chìa khóa để thiết kế một CSDL mạnh mẽ và dễ bảo trì.

### 4.1 Thiết kế Mối quan hệ phức tạp: Bảng `comments`

Hãy mở rộng ứng dụng chia sẻ ảnh bằng cách thêm chức năng bình luận. Một bình luận sẽ có mối quan hệ với cả một bức ảnh (nó bình luận về ảnh nào) và một người dùng (ai là người bình luận). Điều này đòi hỏi bảng `comments` phải có hai khóa ngoại.

**Thiết kế bảng `comments`:**
*   `id`: Khóa chính cho mỗi bình luận.
*   `contents`: Nội dung văn bản của bình luận.
*   `user_id`: Khóa ngoại tham chiếu đến người dùng đã tạo bình luận.
*   `photo_id`: Khóa ngoại tham chiếu đến bức ảnh mà bình luận này dành cho.

Để minh họa tính linh hoạt của `ON DELETE`, chúng ta sẽ thiết lập `ON DELETE CASCADE` cho cả hai khóa ngoại trong bảng `comments`. Điều này có nghĩa là:
*   Nếu một người dùng bị xóa, tất cả bình luận của họ cũng sẽ bị xóa.
*   Nếu một bức ảnh bị xóa, tất cả bình luận trên bức ảnh đó cũng sẽ bị xóa.

```sql
-- Để đảm bảo các bảng users và photos ở trạng thái sạch và có dữ liệu,
-- chúng ta sẽ xóa và tạo lại chúng trước khi tạo bảng comments.
DROP TABLE IF EXISTS comments;
DROP TABLE IF EXISTS photos CASCADE; -- CASCADE để xóa các bảng phụ thuộc
DROP TABLE IF EXISTS users;

-- Tạo lại bảng users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE
);
INSERT INTO users (username) VALUES
('monahan'), -- ID 1
('alex'),    -- ID 2
('sarah'),   -- ID 3
('john');    -- ID 4

-- Tạo lại bảng photos với ON DELETE CASCADE
CREATE TABLE photos (
    id SERIAL PRIMARY KEY,
    url VARCHAR(255) NOT NULL,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE
);
INSERT INTO photos (url, user_id) VALUES
('http://example.com/photo_a.jpg', 1), -- photo ID 1
('http://example.com/photo_b.jpg', 2), -- photo ID 2
('http://example.com/photo_c.jpg', 1), -- photo ID 3
('http://example.com/photo_d.jpg', 3); -- photo ID 4

-- Tạo bảng comments với hai khóa ngoại và ON DELETE CASCADE
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    contents VARCHAR(240) NOT NULL,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,   -- Bình luận xóa nếu user xóa
    photo_id INTEGER REFERENCES photos(id) ON DELETE CASCADE  -- Bình luận xóa nếu photo xóa
);

-- Chèn dữ liệu mẫu vào bảng comments
INSERT INTO comments (contents, user_id, photo_id) VALUES
('Ảnh đẹp quá!', 2, 1),    -- Alex bình luận ảnh 1 (của Monahan)
('Thích bức này!', 3, 2),   -- Sarah bình luận ảnh 2 (của Alex)
('Tuyệt vời!', 1, 2),      -- Monahan bình luận ảnh 2 (của Alex)
('Rất ý nghĩa.', 4, 3);    -- John bình luận ảnh 3 (của Monahan)

-- Kiểm tra dữ liệu comments
SELECT * FROM comments;
```

### 4.2 Truy vấn Dữ liệu Với Nhiều Khóa Ngoại

Với bảng `comments` mới, chúng ta có thể thực hiện các truy vấn phức tạp hơn, kết hợp dữ liệu từ ba bảng hoặc nhiều hơn để có được cái nhìn tổng thể.

**Ví dụ:** Lấy nội dung bình luận cùng với tên người dùng đã viết bình luận và URL của bức ảnh được bình luận.

```sql
-- Lấy nội dung bình luận, tên người dùng và URL ảnh
SELECT
    c.contents,     -- Nội dung bình luận từ bảng comments
    u.username,     -- Tên người dùng từ bảng users
    p.url           -- URL ảnh từ bảng photos
FROM
    comments AS c
INNER JOIN
    users AS u ON c.user_id = u.id    -- Kết hợp comments với users qua user_id
INNER JOIN
    photos AS p ON c.photo_id = p.id; -- Kết hợp comments với photos qua photo_id
```
Truy vấn này sử dụng hai phép `INNER JOIN` liên tiếp để kết nối ba bảng `comments`, `users`, và `photos`. Mỗi `JOIN` được thực hiện trên một cặp khóa ngoại/khóa chính tương ứng. Kết quả là một tập hợp dữ liệu giàu thông tin, hiển thị đầy đủ ngữ cảnh của từng bình luận.

### 4.3 Tư Duy Vibe Coding và Antigravity IDE trong Thiết Kế CSDL

Khi làm việc với các ràng buộc khóa ngoại và thiết kế cơ sở dữ liệu, tư duy **Vibe Coding** trở nên cực kỳ quan trọng. Vibe Coding không chỉ là việc viết code; đó là khả năng thấu hiểu sâu sắc *mục đích*, *ngữ cảnh*, và *hệ quả* của từng dòng code, từng quyết định thiết kế. Đối với CSDL, điều này có nghĩa là:

1.  **Hiểu rõ mối quan hệ nghiệp vụ:** Trước khi viết `REFERENCES` hay `ON DELETE CASCADE`, bạn cần hiểu rõ mối quan hệ giữa các thực thể trong nghiệp vụ (ví dụ: một người dùng *sở hữu* các bức ảnh, một bình luận *thuộc về* một người dùng và một bức ảnh).
2.  **Dự đoán hành vi hệ thống:** Với `ON DELETE CASCADE`, bạn phải "cảm nhận" được rằng việc xóa một người dùng sẽ dẫn đến việc xóa hàng loạt các bản ghi khác. Đây là một "vibe" mạnh mẽ về tác động lan truyền của dữ liệu.
3.  **Đánh giá rủi ro và lợi ích:** `CASCADE` mang lại sự tiện lợi nhưng cũng tiềm ẩn rủi ro mất dữ liệu. `SET NULL` giữ lại dữ liệu nhưng yêu cầu xử lý các giá trị `NULL`. Vibe Coding giúp bạn cân nhắc những trade-off này.

**Antigravity IDE và Tư Duy Agentic AI:**

Hệ thống Antigravity IDE, với khả năng Agentic AI, có thể nâng cao đáng kể quá trình thiết kế và làm việc với khóa ngoại bằng cách áp dụng tư duy Vibe Coding một cách tự động:

*   **Tạo Schema Thông minh:** Khi bạn mô tả một ứng dụng hoặc các thực thể dữ liệu ở cấp độ cao (ví dụ: "Người dùng có thể đăng ảnh và bình luận về ảnh của người khác"), Antigravity có thể tự động đề xuất các câu lệnh `CREATE TABLE` với các khóa chính, khóa ngoại, và thậm chí cả các tùy chọn `ON DELETE` phù hợp nhất, dựa trên ngữ cảnh nghiệp vụ mà nó đã "cảm nhận" được. Nó có thể hỏi bạn: "Khi một người dùng bị xóa, bạn có muốn xóa tất cả ảnh và bình luận của họ không, hay chỉ muốn ẩn danh chúng?"
*   **Kiểm tra Tính Toàn vẹn Dữ liệu Chủ động:** Antigravity có thể tự động chạy các script kiểm tra (ví dụ: `SELECT * FROM photos WHERE user_id NOT IN (SELECT id FROM users) AND user_id IS NOT NULL;`) để phát hiện các bản ghi mồ côi hoặc vi phạm ràng buộc trước khi bạn gặp lỗi trong ứng dụng. Nếu phát hiện, nó có thể đề xuất các hành động khắc phục hoặc thậm chí tự động thực hiện chúng sau khi được xác nhận.
*   **Hỗ trợ Di chuyển Schema (Schema Migration):** Khi bạn thay đổi cấu trúc CSDL (ví dụ: thêm một khóa ngoại mới), Antigravity có thể phân tích các bảng hiện có, dự đoán các tác động, và tạo ra các script `ALTER TABLE` cần thiết, bao gồm cả việc thêm/bỏ ràng buộc khóa ngoại một cách an toàn, có tính đến thứ tự phụ thuộc.
*   **Gỡ lỗi Vi phạm Ràng buộc:** Nếu bạn gặp lỗi `foreign key constraint violation`, Antigravity có thể không chỉ chỉ ra dòng code gây lỗi mà còn phân tích trạng thái của CSDL, hiển thị các bản ghi liên quan trong bảng cha và con, và đề xuất các giải pháp khả thi (ví dụ: "Người dùng có ID 9999 không tồn tại. Bạn có muốn chèn người dùng này trước, hoặc cập nhật `user_id` thành một giá trị hợp lệ?").
*   **Phân tích Tác động của `ON DELETE`:** Trước khi bạn thực thi một lệnh `DELETE` có thể kích hoạt `CASCADE`, Antigravity có thể mô phỏng hoặc cảnh báo về số lượng bản ghi sẽ bị ảnh hưởng, giúp bạn tránh mất dữ liệu không mong muốn. Nó có thể "chạy script ngầm" để đếm số lượng bản ghi con trước khi bạn thực hiện xóa.

Bằng cách hiểu sâu sắc về khóa ngoại và áp dụng tư duy Vibe Coding, bạn có thể hướng dẫn Antigravity làm việc hiệu quả hơn, biến nó thành một cộng sự đắc lực trong việc xây dựng và duy trì các hệ thống CSDL phức tạp. Antigravity không chỉ thực thi lệnh; nó *hiểu* cấu trúc và mối quan hệ dữ liệu, cho phép nó *lập kế hoạch* và *thực hiện* các hành động thông minh, giảm thiểu lỗi và tăng cường tính toàn vẹn.

## Tóm Tắt Chương

Chương này đã trang bị cho bạn kiến thức chuyên sâu về Khóa Ngoại trong PostgreSQL, từ nền tảng đến các ràng buộc nâng cao và ứng dụng thực tế:

*   **Khóa Ngoại** là công cụ thiết yếu để liên kết các bảng, đảm bảo **tính toàn vẹn tham chiếu** và ngăn chặn dữ liệu mồ côi.
*   Trong PostgreSQL, bạn định nghĩa khóa ngoại bằng cú pháp `REFERENCES <parent_table>(<parent_column>)` trong câu lệnh `CREATE TABLE`.
*   **Kiểu dữ liệu `SERIAL`** là lựa chọn lý tưởng cho các khóa chính tự động tăng trong PostgreSQL.
*   **Ràng buộc khi chèn dữ liệu:** Khóa ngoại đảm bảo rằng các bản ghi con chỉ có thể tham chiếu đến bản ghi cha tồn tại hoặc có thể là `NULL` (nếu được phép).
*   **Các chính sách `ON DELETE`** là cơ chế mạnh mẽ để quản lý hành vi của CSDL khi một bản ghi cha bị xóa:
    *   `ON DELETE RESTRICT` (hoặc `NO ACTION`): Ngăn chặn việc xóa bản ghi cha nếu có bản ghi con liên quan.
    *   `ON DELETE CASCADE`: Tự động xóa tất cả bản ghi con khi bản ghi cha bị xóa.
    *   `ON DELETE SET NULL`: Đặt giá trị khóa ngoại của bản ghi con thành `NULL` khi bản ghi cha bị xóa (yêu cầu cột cho phép `NULL`).
    *   `ON DELETE SET DEFAULT`: Đặt giá trị khóa ngoại của bản ghi con thành giá trị mặc định khi bản ghi cha bị xóa (yêu cầu cột có `DEFAULT` và giá trị mặc định tồn tại trong bảng cha).
*   **Toán tử `JOIN`** (đặc biệt là `INNER JOIN`) là công cụ chính để kết hợp dữ liệu từ nhiều bảng liên quan, tận dụng các khóa ngoại để tạo ra các tập hợp kết quả ý nghĩa.
*   Bạn có thể mở rộng mô hình dữ liệu với **nhiều khóa ngoại** trong một bảng để biểu diễn các mối quan hệ phức tạp hơn, như bảng `comments` liên kết với cả `users` và `photos`.
*   **Tư duy Vibe Coding** và việc tận dụng các hệ thống **Agentic AI** như Antigravity IDE giúp bạn thiết kế, quản lý và gỡ lỗi CSDL một cách thông minh, chủ động, và hiệu quả hơn, bằng cách hiểu sâu sắc về ngữ cảnh và tác động của các ràng buộc dữ liệu.

Việc nắm vững các khái niệm này là bước đệm vững chắc để bạn trở thành một chuyên gia lập trình cấp Senior, có khả năng xây dựng các hệ thống cơ sở dữ liệu không chỉ hoạt động mà còn bền vững và đáng tin cậy.

<!-- REVIEWED_BY_AGENT -->
