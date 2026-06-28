# Phần 4: Cập Nhật và Xóa Dữ Liệu (DML)

Trong thế giới quản lý cơ sở dữ liệu, việc thao tác với dữ liệu hiện có là một khía cạnh không thể thiếu và thường xuyên được thực hiện. Phần này sẽ đi sâu vào hai trong số các thao tác Ngôn ngữ Thao tác Dữ liệu (DML - Data Manipulation Language) quan trọng nhất: Cập nhật (UPDATE) và Xóa (DELETE) dữ liệu. Chúng ta sẽ khám phá cách thay đổi thông tin của các bản ghi đã tồn tại và loại bỏ các bản ghi không còn cần thiết, đồng thời tìm hiểu sâu về các cơ chế hoạt động, các phương pháp thực hành tốt nhất để đảm bảo tính toàn vẹn và an toàn của dữ liệu, đặc biệt trong môi trường PostgreSQL.

Mục tiêu của phần này là trang bị cho bạn kiến thức và kỹ năng để:
*   Hiểu rõ cú pháp, cơ chế và cách sử dụng câu lệnh `UPDATE` để thay đổi dữ liệu trong các bảng.
*   Nắm vững cú pháp, cơ chế và cách sử dụng câu lệnh `DELETE` để loại bỏ các bản ghi khỏi bảng.
*   Nhận thức sâu sắc về tầm quan trọng cốt lõi của mệnh đề `WHERE` trong cả hai câu lệnh để thực hiện các thao tác chính xác và an toàn.
*   Áp dụng các giải pháp và chiến lược nâng cao để tránh cập nhật hoặc xóa dữ liệu ngoài ý muốn, bao gồm việc sử dụng khóa chính (Primary Key), giao dịch (Transactions), và các tính năng PostgreSQL chuyên biệt.
*   Thực hành viết các truy vấn DML hiệu quả, an toàn và có thể kiểm chứng trong PostgreSQL, đồng thời liên hệ với tư duy Vibe Coding và sự hỗ trợ của hệ thống Antigravity IDE.

---

## 1. Tổng Quan về Ngôn Ngữ Thao Tác Dữ Liệu (DML)

Ngôn ngữ Thao tác Dữ liệu (DML) là một tập hợp các câu lệnh SQL được thiết kế để quản lý và tương tác với dữ liệu bên trong các đối tượng cơ sở dữ liệu (như bảng, view). DML là trái tim của mọi ứng dụng tương tác với cơ sở dữ liệu, cho phép người dùng và hệ thống truy xuất, thêm, sửa đổi và loại bỏ thông tin.

Các câu lệnh DML phổ biến bao gồm:
*   `SELECT`: Dùng để truy vấn và lấy dữ liệu từ cơ sở dữ liệu. Đây là câu lệnh được sử dụng nhiều nhất.
*   `INSERT`: Dùng để thêm các bản ghi (hàng) dữ liệu mới vào một bảng.
*   `UPDATE`: Dùng để thay đổi dữ liệu hiện có của một hoặc nhiều bản ghi trong bảng.
*   `DELETE`: Dùng để loại bỏ một hoặc nhiều bản ghi khỏi bảng.

Trong phần này, chúng ta sẽ tập trung chuyên sâu vào `UPDATE` và `DELETE`, hai câu lệnh cho phép chúng ta sửa đổi và loại bỏ dữ liệu đã tồn tại, mang theo mức độ rủi ro cao hơn nếu không được thực hiện cẩn thận.

> [!NOTE]
> Khác với Ngôn ngữ Định nghĩa Dữ liệu (DDL - Data Definition Language) như `CREATE TABLE`, `ALTER TABLE`, hay `DROP TABLE` dùng để định nghĩa và quản lý cấu trúc cơ sở dữ liệu (schema), DML tập trung vào việc quản lý chính bản thân dữ liệu bên trong các cấu trúc đó. DDL là về "cái khung", còn DML là về "nội dung" bên trong cái khung.

---

## 2. Cập Nhật Dữ Liệu Hiện Có (UPDATE)

Cập nhật dữ liệu là quá trình sửa đổi các giá trị của một hoặc nhiều cột trong các hàng (bản ghi) đã tồn tại trong một bảng. Đây là một thao tác rất phổ biến và cần thiết khi thông tin thay đổi theo thời gian, ví dụ: cập nhật trạng thái đơn hàng, điều chỉnh số lượng sản phẩm trong kho, thay đổi địa chỉ liên hệ của khách hàng, hoặc sửa lỗi nhập liệu.

### 2.1. Cú pháp Cơ bản của UPDATE

Cú pháp chuẩn của câu lệnh `UPDATE` trong PostgreSQL được thiết kế rõ ràng và mạnh mẽ:

```sql
UPDATE ten_bang
SET
    cot1 = gia_tri_moi1,
    cot2 = gia_tri_moi2,
    -- ... có thể thêm nhiều cột khác
WHERE dieu_kien;
```

*   `UPDATE ten_bang`: Chỉ định bảng mà bạn muốn thực hiện thao tác cập nhật.
*   `SET cot1 = gia_tri_moi1, cot2 = gia_tri_moi2, ...`: Liệt kê các cột bạn muốn thay đổi và giá trị mới của chúng. Bạn có thể cập nhật một hoặc nhiều cột cùng lúc. Các giá trị mới có thể là giá trị cố định, kết quả của một biểu thức, hoặc kết quả của một truy vấn con.
*   `WHERE dieu_kien`: Đây là mệnh đề *cực kỳ quan trọng* dùng để xác định chính xác những hàng nào sẽ bị ảnh hưởng bởi thao tác cập nhật. Chỉ những hàng thỏa mãn `dieu_kien` mới được cập nhật. Nếu không có mệnh đề `WHERE`, câu lệnh `UPDATE` sẽ áp dụng các thay đổi cho *tất cả* các hàng trong bảng.

> [!CAUTION]
> **Cảnh báo nghiêm trọng:** Nếu bạn bỏ qua mệnh đề `WHERE` trong câu lệnh `UPDATE`, câu lệnh sẽ cập nhật *tất cả các hàng* trong bảng với các giá trị mới được chỉ định. Điều này có thể dẫn đến mất dữ liệu nghiêm trọng và không thể phục hồi nếu không phải là ý định của bạn hoặc nếu bạn không sử dụng giao dịch. Luôn luôn cẩn trọng khi sử dụng `UPDATE`.

### 2.2. Cơ chế hoạt động của UPDATE trong PostgreSQL

Khi một câu lệnh `UPDATE` được thực thi, PostgreSQL không thực sự "sửa" dữ liệu tại chỗ. Thay vào đó, nó hoạt động theo nguyên tắc của **Multiversion Concurrency Control (MVCC)**:
1.  **Đánh dấu hàng cũ là "đã xóa":** Hàng dữ liệu ban đầu không bị xóa vật lý ngay lập tức. Thay vào đó, nó được đánh dấu là "không còn hợp lệ" (dead tuple) đối với các giao dịch mới.
2.  **Chèn hàng mới:** Một phiên bản mới của hàng được tạo ra, chứa các giá trị đã cập nhật. Phiên bản mới này được coi là hàng "hiện tại" (live tuple).
3.  **Dọn dẹp:** Các hàng "đã xóa" (dead tuples) sẽ được dọn dẹp bởi quá trình `VACUUM` (tự động hoặc thủ công) sau này để giải phóng không gian lưu trữ và ngăn chặn sự phình to của bảng (table bloat).

Cơ chế MVCC này đảm bảo rằng các giao dịch đang chạy song song vẫn có thể nhìn thấy phiên bản dữ liệu nhất quán mà chúng bắt đầu, ngay cả khi dữ liệu đang được cập nhật bởi các giao dịch khác.

### 2.3. Ví dụ Thực tế: Quản lý Thông tin Thành phố

Hãy tưởng tượng chúng ta có một bảng `cities` (thành phố) với thông tin về tên, quốc gia, dân số và diện tích.

#### 2.3.1. Chuẩn bị Môi trường: Tạo bảng và dữ liệu mẫu

Để các ví dụ có thể chạy được, chúng ta sẽ tạo bảng `cities` và thêm một số dữ liệu ban đầu. Chúng ta sẽ thêm một cột `id` làm khóa chính (Primary Key) với kiểu `SERIAL` để dễ dàng xác định duy nhất từng thành phố.

```sql
-- Bước 1: Tạo bảng cities
CREATE TABLE cities (
    id SERIAL PRIMARY KEY, -- Khóa chính tự động tăng, đảm bảo tính duy nhất
    name VARCHAR(100) NOT NULL,
    country VARCHAR(100) NOT NULL,
    population BIGINT,       -- Kiểu BIGINT cho dân số lớn
    area NUMERIC(10, 2)      -- Diện tích, ví dụ km^2, với 10 chữ số tổng cộng, 2 chữ số sau dấu thập phân
);

-- Bước 2: Thêm dữ liệu mẫu vào bảng cities
INSERT INTO cities (name, country, population, area) VALUES
('Tokyo', 'Japan', 38505000, 2194.07),
('Delhi', 'India', 32941000, 1484.00),
('Shanghai', 'China', 29210000, 6340.50),
('Sao Paulo', 'Brazil', 22619000, 1521.00),
('Mexico City', 'Mexico', 22085000, 1485.00);

-- Bước 3: Kiểm tra dữ liệu ban đầu
SELECT * FROM cities ORDER BY id;
```

Kết quả `SELECT * FROM cities;` sẽ hiển thị:

| id | name        | country | population | area     |
| -- | ----------- | ------- | ---------- | -------- |
| 1  | Tokyo       | Japan   | 38505000   | 2194.07  |
| 2  | Delhi       | India   | 32941000   | 1484.00  |
| 3  | Shanghai    | China   | 29210000   | 6340.50  |
| 4  | Sao Paulo   | Brazil  | 22619000   | 1521.00  |
| 5  | Mexico City | Mexico  | 22085000   | 1485.00  |

#### 2.3.2. Cập nhật một cột đơn lẻ

Giả sử dân số của Tokyo đã tăng lên 39,505,000 người. Chúng ta sẽ cập nhật thông tin này:

```sql
-- Cập nhật dân số của Tokyo
UPDATE cities
SET population = 39505000
WHERE name = 'Tokyo'; -- Sử dụng tên thành phố làm điều kiện

-- Kiểm tra kết quả sau khi cập nhật
SELECT * FROM cities WHERE name = 'Tokyo';
```

Kết quả `SELECT` sau khi cập nhật:

| id | name  | country | population | area    |
| -- | ----- | ------- | ---------- | ------- |
| 1  | Tokyo | Japan   | 39505000   | 2194.07 |

#### 2.3.3. Cập nhật nhiều cột cùng lúc

Giả sử dân số của Delhi đã tăng lên 33,500,000 và diện tích cũng được điều chỉnh thành 1500.50 km².

```sql
-- Cập nhật dân số và diện tích của Delhi
UPDATE cities
SET
    population = 33500000,
    area = 1500.50
WHERE name = 'Delhi' AND country = 'India'; -- Sử dụng cả tên và quốc gia để đảm bảo tính duy nhất

-- Kiểm tra kết quả
SELECT * FROM cities WHERE name = 'Delhi';
```

#### 2.3.4. Cập nhật dựa trên giá trị hiện có (Sử dụng biểu thức)

Chúng ta có thể sử dụng các biểu thức hoặc các phép toán số học để cập nhật giá trị dựa trên chính giá trị hiện có của cột. Ví dụ, tăng dân số của Sao Paulo thêm 500,000 người.

```sql
-- Tăng dân số của Sao Paulo thêm 500,000
UPDATE cities
SET population = population + 500000
WHERE name = 'Sao Paulo';

-- Kiểm tra kết quả
SELECT * FROM cities WHERE name = 'Sao Paulo';
```

### 2.4. Nguy cơ và Giải pháp: Đảm bảo tính chính xác khi UPDATE

Như đã đề cập, mệnh đề `WHERE` là chìa khóa để đảm bảo bạn chỉ cập nhật những bản ghi mong muốn. Nếu mệnh đề `WHERE` không đủ cụ thể, bạn có thể vô tình cập nhật nhiều bản ghi hơn dự định, dẫn đến lỗi dữ liệu nghiêm trọng.

#### 2.4.1. Vấn đề với điều kiện WHERE không đủ cụ thể

Giả sử có nhiều thành phố trên thế giới có cùng tên. Để minh họa, hãy thêm một thành phố "Shanghai" khác vào bảng.

```sql
-- Thêm một Shanghai khác ở USA để minh họa vấn đề
INSERT INTO cities (name, country, population, area) VALUES
('Shanghai', 'USA', 100000, 50.00);

-- Kiểm tra dữ liệu hiện tại
SELECT * FROM cities WHERE name = 'Shanghai';
```

Kết quả:

| id | name     | country | population | area    |
| -- | -------- | ------- | ---------- | --------|
| 3  | Shanghai | China   | 29210000   | 6340.50 |
| 6  | Shanghai | USA     | 100000     | 50.00   |

Bây giờ, nếu bạn muốn cập nhật dân số của Shanghai (Trung Quốc) nhưng chỉ sử dụng `WHERE name = 'Shanghai'`:

```sql
-- Cập nhật dân số của Shanghai (ý định là chỉ Shanghai ở Trung Quốc)
UPDATE cities
SET population = 29500000
WHERE name = 'Shanghai'; -- Điều kiện này không đủ cụ thể!

-- Kiểm tra: Cả hai bản ghi Shanghai đều bị cập nhật!
SELECT * FROM cities WHERE name = 'Shanghai';
```

Kết quả `SELECT` sẽ cho thấy cả hai bản ghi Shanghai đều có `population = 29500000`, điều này không chính xác cho Shanghai (USA).

#### 2.4.2. Giải pháp: Sử dụng Khóa Chính (Primary Key) hoặc điều kiện cụ thể hơn

Cách tốt nhất để đảm bảo bạn cập nhật *chính xác một hàng* là sử dụng Khóa Chính (Primary Key) của hàng đó trong mệnh đề `WHERE`. Khóa chính là một cột (hoặc tập hợp các cột) có giá trị duy nhất cho mỗi hàng trong bảng. Trong ví dụ của chúng ta, cột `id` là khóa chính.

```sql
-- Bước 1: Đưa dữ liệu Shanghai (USA) về trạng thái ban đầu để minh họa lại
UPDATE cities
SET population = 100000
WHERE id = 6;

-- Bước 2: Cập nhật lại dân số của Shanghai (Trung Quốc) lên 29.5 triệu bằng id
-- Giả sử id của Shanghai ở Trung Quốc là 3 (theo dữ liệu mẫu ban đầu)
UPDATE cities
SET population = 29500000
WHERE id = 3; -- Sử dụng khóa chính để cập nhật chính xác

-- Bước 3: Kiểm tra kết quả
SELECT * FROM cities WHERE name = 'Shanghai'; -- Để thấy rằng Shanghai (USA) không bị ảnh hưởng
```

Kết quả `SELECT * FROM cities WHERE name = 'Shanghai';` sau khi sử dụng `id=3`:

| id | name     | country | population | area    |
| -- | -------- | ------- | ---------- | --------|
| 3  | Shanghai | China   | 29500000   | 6340.50 |
| 6  | Shanghai | USA     | 100000     | 50.00   |

Như bạn thấy, chỉ bản ghi có `id = 3` (Shanghai, China) được cập nhật, trong khi Shanghai (USA) vẫn giữ nguyên.

Ngoài khóa chính, bạn cũng có thể sử dụng tổ hợp các cột tạo thành một điều kiện duy nhất, ví dụ: `WHERE name = 'Shanghai' AND country = 'China'`.

> [!TIP]
> Luôn ưu tiên sử dụng khóa chính (`id`) hoặc một tổ hợp các cột (`name` VÀ `country`) để tạo điều kiện `WHERE` đủ cụ thể khi thực hiện các thao tác `UPDATE` hoặc `DELETE` trên các bản ghi cụ thể. Điều này đặc biệt quan trọng khi làm việc với Antigravity IDE: khi bạn diễn đạt ý định của mình, Antigravity sẽ cố gắng tạo ra một truy vấn an toàn nhất, thường là bằng cách gợi ý sử dụng khóa chính hoặc một điều kiện kết hợp rõ ràng.

#### 2.4.3. Cập nhật dữ liệu từ bảng khác (UPDATE ... FROM)

PostgreSQL cung cấp một cú pháp mở rộng mạnh mẽ cho `UPDATE` cho phép bạn cập nhật một bảng dựa trên dữ liệu từ một hoặc nhiều bảng khác. Điều này rất hữu ích khi bạn cần đồng bộ hóa dữ liệu hoặc áp dụng các thay đổi phức tạp.

**Cú pháp:**

```sql
UPDATE ten_bang_chinh
SET
    cot1 = bang_phu.cot_tuong_ung1,
    cot2 = bang_phu.cot_tuong_ung2
FROM bang_phu
WHERE ten_bang_chinh.khoa_ket_noi = bang_phu.khoa_ket_noi
  AND dieu_kien_bo_sung;
```

**Ví dụ:** Giả sử chúng ta có một bảng `new_city_data` chứa thông tin cập nhật về dân số và diện tích của một số thành phố.

```sql
-- Tạo bảng tạm new_city_data
CREATE TEMPORARY TABLE new_city_data (
    city_name VARCHAR(100) NOT NULL,
    new_population BIGINT,
    new_area NUMERIC(10, 2)
);

-- Thêm dữ liệu cập nhật
INSERT INTO new_city_data (city_name, new_population, new_area) VALUES
('Mexico City', 22150000, 1490.00),
('Sao Paulo', 23000000, 1550.00);

-- Kiểm tra dữ liệu hiện tại của các thành phố này trong bảng chính
SELECT * FROM cities WHERE name IN ('Mexico City', 'Sao Paulo');

-- Cập nhật bảng cities từ new_city_data
UPDATE cities
SET
    population = ncd.new_population,
    area = ncd.new_area
FROM new_city_data AS ncd
WHERE cities.name = ncd.city_name;

-- Kiểm tra kết quả sau khi cập nhật
SELECT * FROM cities WHERE name IN ('Mexico City', 'Sao Paulo');

-- Xóa bảng tạm
DROP TABLE new_city_data;
```

Kết quả sẽ cho thấy dân số và diện tích của Mexico City và Sao Paulo đã được cập nhật từ bảng `new_city_data`. Tính năng này là một điểm cộng lớn của PostgreSQL so với một số hệ quản trị CSDL khác, nơi bạn có thể phải dùng truy vấn con hoặc CTE phức tạp hơn.

---

## 3. Xóa Dữ Liệu Khỏi Bảng (DELETE)

Xóa dữ liệu là quá trình loại bỏ một hoặc nhiều hàng khỏi một bảng. Tương tự như `UPDATE`, mệnh đề `WHERE` là yếu tố quyết định để xác định những hàng nào sẽ bị xóa. Đây là một thao tác **không thể hoàn tác** (nếu không có giao dịch), do đó đòi hỏi sự cẩn trọng tối đa.

### 3.1. Cú pháp Cơ bản của DELETE

Cú pháp chuẩn của câu lệnh `DELETE` trong PostgreSQL như sau:

```sql
DELETE FROM ten_bang
WHERE dieu_kien;
```

*   `DELETE FROM ten_bang`: Chỉ định bảng mà bạn muốn xóa dữ liệu. Từ khóa `FROM` là tùy chọn trong tiêu chuẩn SQL nhưng được khuyến nghị để tăng tính rõ ràng.
*   `WHERE dieu_kien`: Mệnh đề *cực kỳ quan trọng* này xác định chính xác những hàng nào sẽ bị xóa. Chỉ những hàng thỏa mãn `dieu_kien` mới bị loại bỏ.

> [!CAUTION]
> **Cảnh báo nghiêm trọng:** Tương tự như `UPDATE`, nếu bạn bỏ qua mệnh đề `WHERE`, câu lệnh `DELETE` sẽ xóa *tất cả* các hàng trong bảng. Điều này là một trong những lỗi phổ biến và tai hại nhất trong quản lý CSDL. Luôn luôn kiểm tra kỹ lưỡng điều kiện `WHERE` của bạn.

### 3.2. Cơ chế hoạt động của DELETE trong PostgreSQL

Cơ chế `DELETE` trong PostgreSQL cũng dựa trên MVCC, tương tự như `UPDATE`:
1.  **Đánh dấu hàng là "đã xóa":** Các hàng được chọn để xóa không bị loại bỏ vật lý ngay lập tức. Thay vào đó, chúng được đánh dấu là "không còn hợp lệ" (dead tuple) và không thể truy cập được bởi các giao dịch mới.
2.  **Dọn dẹp:** Giống như `UPDATE`, các hàng "đã xóa" này sẽ được dọn dẹp bởi quá trình `VACUUM` sau này để giải phóng không gian lưu trữ thực tế. Cho đến khi `VACUUM` chạy, không gian đĩa vật lý có thể chưa được giải phóng hoàn toàn.

### 3.3. Ví dụ Thực tế: Gỡ bỏ dữ liệu không cần thiết

Sử dụng lại bảng `cities` của chúng ta. Giả sử chúng ta muốn xóa bản ghi của Tokyo.

#### 3.3.1. Kiểm tra dữ liệu hiện có

```sql
-- Kiểm tra các thành phố trước khi xóa
SELECT * FROM cities ORDER BY id;
```

#### 3.3.2. Xóa một bản ghi cụ thể

```sql
-- Xóa thành phố Tokyo
DELETE FROM cities
WHERE name = 'Tokyo';

-- Kiểm tra kết quả sau khi xóa
SELECT * FROM cities ORDER BY id; -- Tokyo sẽ không còn trong danh sách
```

Kết quả `SELECT * FROM cities;` sau khi xóa:

| id | name        | country | population | area     |
| -- | ----------- | ------- | ---------- | -------- |
| 2  | Delhi       | India   | 33500000   | 1500.50  |
| 3  | Shanghai    | China   | 29500000   | 6340.50  |
| 4  | Sao Paulo   | Brazil  | 23000000   | 1550.00  |
| 5  | Mexico City | Mexico  | 22150000   | 1490.00  |
| 6  | Shanghai    | USA     | 100000     | 50.00    |

Tokyo đã bị xóa khỏi bảng.

### 3.4. Nguy cơ và Giải pháp: Bảo vệ dữ liệu khỏi xóa nhầm

Giống như `UPDATE`, việc sử dụng mệnh đề `WHERE` không chính xác trong câu lệnh `DELETE` có thể dẫn đến việc xóa nhiều bản ghi hơn mong muốn, gây mất dữ liệu không thể phục hồi (nếu không có giao dịch).

#### 3.4.1. Vấn đề với điều kiện WHERE không chính xác

Sử dụng lại bảng `phones` để minh họa.

```sql
-- Tạo bảng phones
CREATE TABLE phones (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    manufacturer VARCHAR(100) NOT NULL,
    units_sold INT
);

-- Thêm dữ liệu mẫu
INSERT INTO phones (name, manufacturer, units_sold) VALUES
('iPhone 13', 'Apple', 7543),
('Galaxy S22', 'Samsung', 6120),
('Pixel 6', 'Google', 3200),
('iPhone 8', 'Apple', 8543),
('Galaxy Fold', 'Samsung', 1500); -- Thêm một điện thoại Samsung khác để minh họa

-- Kiểm tra dữ liệu hiện có trong bảng phones
SELECT * FROM phones ORDER BY id;
```

Kết quả `SELECT * FROM phones;`:

| id | name        | manufacturer | units_sold |
| -- | ----------- | ------------ | ---------- |
| 1  | iPhone 13   | Apple        | 7543       |
| 2  | Galaxy S22  | Samsung      | 6120       |
| 3  | Pixel 6     | Google       | 3200       |
| 4  | iPhone 8    | Apple        | 8543       |
| 5  | Galaxy Fold | Samsung      | 1500       |

Bây giờ, nếu bạn có ý định xóa một mẫu Samsung cụ thể (ví dụ: Galaxy S22) nhưng lại vô tình viết điều kiện `WHERE manufacturer = 'Samsung'`:

```sql
-- Ý định là xóa Galaxy S22, nhưng lại xóa tất cả điện thoại của Samsung
DELETE FROM phones
WHERE manufacturer = 'Samsung';

-- Kiểm tra: Tất cả điện thoại của Samsung đã biến mất!
SELECT * FROM phones WHERE manufacturer = 'Samsung';
```
Kết quả `SELECT` sẽ không trả về hàng nào, vì tất cả điện thoại của Samsung đã bị xóa.

#### 3.4.2. Giải pháp: Sử dụng Khóa Chính (Primary Key) hoặc điều kiện cụ thể hơn

Để xóa chính xác một hoặc một tập hợp các hàng, hãy luôn sử dụng Khóa Chính (Primary Key) hoặc một điều kiện `WHERE` đủ cụ thể và duy nhất.

```sql
-- Bước 1: Để minh họa, hãy thêm lại các điện thoại Samsung đã xóa
INSERT INTO phones (id, name, manufacturer, units_sold) VALUES
(2, 'Galaxy S22', 'Samsung', 6120),
(5, 'Galaxy Fold', 'Samsung', 1500); -- Sử dụng id cố định để dễ quản lý trong ví dụ

-- Bước 2: Xóa chỉ Galaxy S22 bằng cách sử dụng id
DELETE FROM phones
WHERE id = 2; -- Sử dụng khóa chính để xóa chính xác

-- Bước 3: Kiểm tra kết quả
SELECT * FROM phones ORDER BY id;
```

Kết quả `SELECT * FROM phones;` sau khi xóa `id=2`:

| id | name        | manufacturer | units_sold |
| -- | ----------- | ------------ | ---------- |
| 1  | iPhone 13   | Apple        | 7543       |
| 3  | Pixel 6     | Google       | 3200       |
| 4  | iPhone 8    | Apple        | 8543       |
| 5  | Galaxy Fold | Samsung      | 1500       |

Như bạn thấy, chỉ Galaxy S22 (với `id = 2`) bị xóa, trong khi Galaxy Fold vẫn còn.

### 3.5. Sự khác biệt giữa DELETE FROM và TRUNCATE TABLE

Mặc dù cả `DELETE FROM` (không có `WHERE`) và `TRUNCATE TABLE` đều có thể xóa tất cả các hàng khỏi một bảng, chúng hoạt động rất khác nhau ở cấp độ cơ bản và có những ảnh hưởng khác nhau đến hiệu suất và tính toàn vẹn dữ liệu.

| Đặc điểm                  | `DELETE FROM ten_bang;` (không WHERE) | `TRUNCATE TABLE ten_bang;`               |
| :------------------------ | :------------------------------------ | :--------------------------------------- |
| **Loại câu lệnh**         | DML (Data Manipulation Language)      | DDL (Data Definition Language)           |
| **Cơ chế hoạt động**      | Xóa từng hàng một (logic).            | Giải phóng toàn bộ không gian bảng.     |
| **Khả năng hoàn tác**     | Có thể `ROLLBACK` (nếu trong giao dịch). | **Không thể `ROLLBACK`** (trừ khi được bọc trong giao dịch `BEGIN; ... COMMIT;` đặc biệt trong một số CSDL, nhưng không phải hành vi mặc định và không nên dựa vào). |
| **Kích hoạt Trigger**     | Kích hoạt các trigger `ON DELETE`.    | **Không** kích hoạt các trigger `ON DELETE`. |
| **Tạo bản ghi WAL**       | Tạo nhiều bản ghi Write-Ahead Log (WAL) cho mỗi hàng bị xóa. | Tạo ít bản ghi WAL hơn, chủ yếu cho việc giải phóng trang. |
| **Hiệu suất**             | Chậm hơn với các bảng lớn do xử lý từng hàng. | Nhanh hơn đáng kể với các bảng lớn.      |
| **Giải phóng không gian** | Không gian đĩa được đánh dấu để tái sử dụng, nhưng cần `VACUUM` để giải phóng vật lý hoàn toàn. | Giải phóng không gian đĩa vật lý ngay lập tức. |
| **Reset `SERIAL` / Identity** | **Không** reset giá trị của các cột `SERIAL` hoặc Identity. | Reset giá trị của các cột `SERIAL` hoặc Identity về giá trị khởi tạo. |
| **Mệnh đề `WHERE`**       | Có thể sử dụng để xóa có chọn lọc.    | **Không** thể sử dụng mệnh đề `WHERE`. Xóa toàn bộ. |
| **Khóa (Locks)**          | Khóa cấp hàng hoặc cấp bảng (tùy điều kiện). | Khóa cấp bảng (ACCESS EXCLUSIVE LOCK) ngăn chặn các thao tác khác. |

> [!TIP]
> *   Sử dụng `DELETE FROM` khi bạn cần xóa một số hàng cụ thể, hoặc khi bạn cần khả năng hoàn tác thao tác xóa (bằng giao dịch), hoặc khi bạn cần kích hoạt các trigger `ON DELETE`.
> *   Sử dụng `TRUNCATE TABLE` khi bạn muốn xóa *tất cả* dữ liệu khỏi một bảng một cách nhanh chóng, hiệu quả và không cần khả năng hoàn tác, đồng thời muốn reset các giá trị `SERIAL` hoặc Identity. Hãy coi `TRUNCATE` như một `DROP TABLE` và `CREATE TABLE` nhanh chóng.

---

## 4. Thực Hành Tốt Nhất và Chiến Lược An Toàn trong DML

Để đảm bảo tính toàn vẹn và an toàn của dữ liệu, đặc biệt khi thực hiện các thao tác DML có thể gây phá hủy (`UPDATE`, `DELETE`), hãy tuân thủ các thực hành tốt nhất sau. Các nguyên tắc của Vibe Coding (tập trung vào ý định, ngữ cảnh và xác minh) kết hợp với sức mạnh của Antigravity IDE có thể nâng cao đáng kể sự an toàn và hiệu quả của bạn.

### 4.1. Luôn Luôn Sử Dụng Mệnh đề WHERE (Trừ khi có chủ đích)

Đây là quy tắc vàng không thể phá vỡ cho `UPDATE` và `DELETE`. Không bao giờ chạy một câu lệnh `UPDATE` hoặc `DELETE` mà không có mệnh đề `WHERE` trừ khi bạn *hoàn toàn chắc chắn* rằng bạn muốn thay đổi hoặc xóa *tất cả* các hàng trong bảng và đã hiểu rõ hậu quả.

### 4.2. Ưu Tiên Khóa Chính (Primary Key) cho Thao tác đơn lẻ

Khi bạn muốn thao tác trên một bản ghi cụ thể, luôn sử dụng khóa chính của bản ghi đó trong mệnh đề `WHERE`. Điều này đảm bảo tính chính xác và duy nhất của thao tác, giảm thiểu rủi ro.

### 4.3. Kiểm Tra Bằng SELECT Trước Khi Thực Thi (Dry Run)

Trước khi chạy một câu lệnh `UPDATE` hoặc `DELETE` quan trọng, đặc biệt là trong môi trường sản xuất hoặc với dữ liệu nhạy cảm, hãy chạy một câu lệnh `SELECT` với *cùng điều kiện `WHERE`* để xem chính xác những hàng nào sẽ bị ảnh hưởng. Đây là một bước kiểm tra "khô" (dry run) cực kỳ quan trọng.

**Ví dụ:**
```sql
-- Bước 1: Kiểm tra những hàng sẽ bị ảnh hưởng bởi thao tác xóa
SELECT id, name, country, population FROM cities
WHERE name = 'Shanghai' AND country = 'USA';

-- Bước 2: Nếu kết quả của SELECT đúng như mong đợi, hãy thực hiện thao tác xóa
DELETE FROM cities
WHERE name = 'Shanghai' AND country = 'USA';
```

> [!NOTE]
> **Áp dụng Vibe Coding và Antigravity IDE:**
> Với Antigravity IDE, bạn có thể áp dụng tư duy Vibe Coding một cách hiệu quả tại đây. Khi bạn có ý định "xóa thành phố Shanghai ở USA", bạn có thể diễn đạt ý định đó cho Antigravity. Antigravity, với khả năng lập trình và hiểu ngữ cảnh, sẽ tự động:
> 1.  **Gợi ý câu lệnh `SELECT`:** "Bạn có muốn xem trước các bản ghi sẽ bị xóa không?" và tạo ra `SELECT * FROM cities WHERE name = 'Shanghai' AND country = 'USA';`.
> 2.  **Chờ xác nhận:** Hiển thị kết quả `SELECT` và yêu cầu bạn xác nhận xem đây có phải là những bản ghi bạn muốn xóa.
> 3.  **Thực thi an toàn:** Chỉ sau khi bạn xác nhận, Antigravity mới thực thi câu lệnh `DELETE` đã được kiểm tra.
> Điều này biến bước "kiểm tra bằng SELECT" thành một quy trình tương tác, có hướng dẫn, giảm thiểu đáng kể lỗi do con người.

### 4.4. Tận Dụng Giao Dịch (Transactions) để Đảm Bảo An Toàn

Giao dịch là một chuỗi các thao tác SQL được thực hiện như một đơn vị logic duy nhất, đảm bảo tính toàn vẹn của dữ liệu thông qua các thuộc tính **ACID** (Atomicity, Consistency, Isolation, Durability). Toàn bộ giao dịch sẽ thành công (COMMIT) hoặc thất bại hoàn toàn (ROLLBACK). Đây là một cơ chế an toàn tuyệt vời cho các thao tác DML, cho phép bạn hoàn tác các thay đổi nếu có lỗi hoặc không mong muốn.

**Cú pháp giao dịch cơ bản trong PostgreSQL:**

```sql
BEGIN; -- Hoặc START TRANSACTION; Bắt đầu một giao dịch mới

    -- Các câu lệnh DML của bạn (UPDATE, DELETE, INSERT)
    UPDATE cities SET population = 40000000 WHERE id = 1;
    DELETE FROM phones WHERE manufacturer = 'Samsung';

    -- Bước kiểm tra trong giao dịch:
    -- Các thay đổi này chỉ hiển thị cho phiên làm việc hiện tại của bạn
    -- và chưa được ghi vĩnh viễn vào cơ sở dữ liệu.
    SELECT * FROM cities WHERE id = 1;
    SELECT * FROM phones WHERE manufacturer = 'Samsung';

-- Nếu mọi thứ đều đúng và bạn hài lòng với các thay đổi:
-- COMMIT; -- Ghi các thay đổi vĩnh viễn vào cơ sở dữ liệu

-- Nếu có lỗi, bạn phát hiện ra điều gì đó không mong muốn, hoặc bạn muốn hoàn tác:
-- ROLLBACK; -- Hủy bỏ tất cả các thay đổi đã thực hiện trong giao dịch này,
            -- đưa cơ sở dữ liệu về trạng thái trước khi BEGIN; được thực thi.
```

> [!IMPORTANT]
> Khi bạn chạy `BEGIN;`, các thay đổi bạn thực hiện sẽ không được lưu vĩnh viễn vào cơ sở dữ liệu cho đến khi bạn chạy `COMMIT;`. Nếu bạn chạy `ROLLBACK;`, tất cả các thay đổi trong giao dịch đó sẽ bị hủy bỏ, và cơ sở dữ liệu sẽ trở về trạng thái trước khi `BEGIN;` được thực thi. Đây là một công cụ cực kỳ mạnh mẽ để thử nghiệm và đảm bảo an toàn dữ liệu, đặc biệt khi bạn không chắc chắn về tác động của câu lệnh DML.

> [!NOTE]
> **Áp dụng Vibe Coding và Antigravity IDE:**
> Vibe Coding khuyến khích bạn thể hiện ý định "thực hiện một thao tác DML, nhưng với khả năng hoàn tác nếu có lỗi." Antigravity IDE có thể hỗ trợ bạn bằng cách:
> 1.  **Tự động bọc trong giao dịch:** Khi bạn yêu cầu Antigravity thực hiện một `UPDATE` hoặc `DELETE` phức tạp, nó có thể tự động bọc các câu lệnh đó trong một khối `BEGIN; ...` và yêu cầu bạn xác nhận `COMMIT;` hoặc `ROLLBACK;` sau khi bạn đã xem xét kết quả.
> 2.  **Hiển thị trạng thái giao dịch:** Antigravity có thể chỉ ra rõ ràng rằng bạn đang ở trong một giao dịch và các thay đổi chưa được lưu vĩnh viễn, giúp bạn đưa ra quyết định an toàn hơn.
> 3.  **Gợi ý `ROLLBACK`:** Nếu bạn phát hiện ra lỗi, Antigravity có thể ngay lập tức gợi ý và thực thi `ROLLBACK;` cho bạn.

### 4.5. Sử dụng Mệnh đề `RETURNING` để xác nhận các thay đổi (PostgreSQL-specific)

PostgreSQL cung cấp mệnh đề `RETURNING` cho các câu lệnh `INSERT`, `UPDATE`, và `DELETE`. Mệnh đề này cho phép bạn lấy lại dữ liệu của các hàng đã bị ảnh hưởng bởi thao tác. Đây là một cách tuyệt vời để xác nhận chính xác những gì đã được thay đổi hoặc xóa.

**Ví dụ với `UPDATE`:**
```sql
-- Cập nhật dân số của Mexico City và trả về các cột đã thay đổi
UPDATE cities
SET population = population + 100000 -- Tăng thêm 100,000
WHERE name = 'Mexico City'
RETURNING id, name, population AS new_population, (population - 100000) AS old_population;
```

Kết quả:

| id | name        | new_population | old_population |
| -- | ----------- | -------------- | -------------- |
| 5  | Mexico City | 22250000       | 22150000       |

**Ví dụ với `DELETE`:**
```sql
-- Xóa thành phố Delhi và trả về id và tên của thành phố đã xóa
DELETE FROM cities
WHERE name = 'Delhi'
RETURNING id, name, country;
```

Kết quả:

| id | name  | country |
| -- | ----- | ------- |
| 2  | Delhi | India   |

Mệnh đề `RETURNING` cung cấp phản hồi tức thì, giúp bạn xác minh rằng thao tác DML đã ảnh hưởng đến đúng các bản ghi và theo cách mong muốn.

### 4.6. Quyền Hạn (Permissions) và Vai Trò (Roles)

Trong môi trường sản xuất, hãy luôn tuân thủ nguyên tắc "quyền hạn tối thiểu cần thiết" (least privilege). Cấp cho người dùng hoặc ứng dụng chỉ những quyền DML (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) mà họ thực sự cần.

**Ví dụ:**
*   Để cho phép một vai trò chỉ được đọc dữ liệu:
    ```sql
    GRANT SELECT ON cities TO readonly_user;
    ```
*   Để cho phép một vai trò được cập nhật dân số nhưng không được xóa:
    ```sql
    GRANT UPDATE (population) ON cities TO data_updater;
    REVOKE DELETE ON cities FROM data_updater; -- Đảm bảo không có quyền xóa
    ```
Việc quản lý quyền hạn chặt chẽ là một lớp bảo vệ quan trọng chống lại các thao tác DML ngoài ý muốn hoặc độc hại.

---

## Tóm Tắt Phần 4: Cập Nhật và Xóa Dữ Liệu (DML)

*   **UPDATE:** Dùng để thay đổi dữ liệu hiện có trong một bảng. Cú pháp cơ bản là `UPDATE ten_bang SET cot = gia_tri_moi WHERE dieu_kien;`. PostgreSQL hỗ trợ `UPDATE ... FROM` để cập nhật từ các bảng khác.
*   **DELETE:** Dùng để loại bỏ một hoặc nhiều hàng khỏi một bảng. Cú pháp cơ bản là `DELETE FROM ten_bang WHERE dieu_kien;`.
*   **Mệnh đề WHERE là Cực Kỳ Quan Trọng:** Luôn sử dụng `WHERE` để chỉ định chính xác các hàng bị ảnh hưởng. Nếu không có `WHERE`, thao tác sẽ ảnh hưởng đến *tất cả* các hàng trong bảng và có thể gây mất dữ liệu nghiêm trọng.
*   **Cơ chế MVCC:** Cả `UPDATE` và `DELETE` trong PostgreSQL đều hoạt động dựa trên Multiversion Concurrency Control, tạo ra các phiên bản mới của hàng hoặc đánh dấu hàng cũ là "đã xóa" thay vì sửa đổi tại chỗ, đảm bảo tính đồng thời và nhất quán.
*   **Khóa Chính (Primary Key):** Là cách tốt nhất để xác định duy nhất một hàng khi cập nhật hoặc xóa, giúp tránh các lỗi không mong muốn.
*   **Thực hành tốt nhất và an toàn dữ liệu:**
    *   Luôn kiểm tra bằng `SELECT` với cùng điều kiện `WHERE` trước khi thực thi `UPDATE` hoặc `DELETE` quan trọng (Dry Run).
    *   Sử dụng **Giao dịch (Transactions)** (`BEGIN;`, `COMMIT;`, `ROLLBACK;`) để có thể hoàn tác các thay đổi nếu cần, tăng cường an toàn dữ liệu.
    *   Sử dụng mệnh đề **`RETURNING`** (PostgreSQL-specific) để xác nhận chính xác các hàng đã bị ảnh hưởng.
    *   Quản lý chặt chẽ **Quyền Hạn** (Permissions) và **Vai Trò** (Roles).
*   **DELETE FROM vs TRUNCATE TABLE:** `DELETE FROM` là DML, có thể hoàn tác (trong giao dịch), kích hoạt trigger, có thể dùng `WHERE`, không reset `SERIAL`. `TRUNCATE TABLE` là DDL, nhanh hơn, không hoàn tác được (dễ dàng), không kích hoạt trigger, xóa toàn bộ bảng, reset `SERIAL`.
*   **Vibe Coding và Antigravity IDE:** Áp dụng tư duy Vibe Coding bằng cách thể hiện rõ ràng ý định của bạn. Antigravity IDE có thể hỗ trợ bạn bằng cách tự động tạo các câu lệnh `SELECT` để kiểm tra, bọc các thao tác DML trong giao dịch, và yêu cầu xác nhận trước khi thực thi, giúp bạn làm việc an toàn và hiệu quả hơn.

<!-- REVIEWED_BY_AGENT -->
