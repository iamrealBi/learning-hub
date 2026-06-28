# Phần 1: Giới Thiệu và Các Lệnh SQL Cơ Bản

Phần này cung cấp nền tảng vững chắc về cơ sở dữ liệu quan hệ, tập trung chuyên sâu vào PostgreSQL. Chúng ta sẽ bắt đầu từ các khái niệm cốt lõi về cách tổ chức và lưu trữ dữ liệu, tiến tới việc thiết kế cấu trúc cơ sở dữ liệu và sử dụng ngôn ngữ SQL để định nghĩa cũng như thao tác dữ liệu. Mục tiêu là trang bị cho bạn không chỉ kiến thức lý thuyết mà còn là kỹ năng thực hành để tương tác hiệu quả với PostgreSQL, từ việc tạo bảng đến việc chèn và truy xuất thông tin.

## 1. Kiến Trúc Cơ Sở Dữ Liệu và PostgreSQL

### 1.1. Cơ Sở Dữ Liệu là gì?

Ở cốt lõi, một **cơ sở dữ liệu** là một hệ thống có tổ chức được thiết kế để lưu trữ, quản lý và truy xuất một lượng lớn thông tin một cách hiệu quả và đáng tin cậy. Thay vì lưu trữ dữ liệu trong các tệp phẳng hoặc bảng tính rời rạc, cơ sở dữ liệu cung cấp một cấu trúc chặt chẽ, các quy tắc nhất quán và cơ chế mạnh mẽ để đảm bảo tính toàn vẹn và khả năng truy cập của dữ liệu.

Các hệ thống cơ sở dữ liệu hiện đại không chỉ đơn thuần là nơi chứa dữ liệu; chúng là các kiến trúc phức tạp bao gồm:
*   **Dữ liệu (Data)**: Thông tin thực tế được lưu trữ.
*   **Lược đồ (Schema)**: Cấu trúc logic định nghĩa cách dữ liệu được tổ chức (tên bảng, tên cột, kiểu dữ liệu, mối quan hệ).
*   **Hệ quản trị cơ sở dữ liệu (DBMS - Database Management System)**: Phần mềm cho phép người dùng và ứng dụng tương tác với cơ sở dữ liệu (tạo, đọc, cập nhật, xóa dữ liệu; quản lý quyền truy cập; đảm bảo an toàn và nhất quán).

### 1.2. PostgreSQL: Hệ Quản Trị Cơ Sở Dữ Liệu Quan Hệ (RDBMS) Mạnh Mẽ

**PostgreSQL** là một trong những hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) mã nguồn mở tiên tiến và mạnh mẽ nhất hiện nay. Nó được biết đến với:
*   **Độ tin cậy và tính bền vững (Reliability & Robustness)**: Được phát triển qua hơn 35 năm, PostgreSQL có một lịch sử dài về sự ổn định và khả năng phục hồi dữ liệu.
*   **Tính năng phong phú (Feature-rich)**: Hỗ trợ nhiều tính năng nâng cao không chỉ của SQL tiêu chuẩn mà còn các tính năng độc quyền như kiểu dữ liệu JSONB, các hàm cửa sổ (window functions), CTEs (Common Table Expressions), và các chỉ mục đa dạng.
*   **Tuân thủ tiêu chuẩn SQL (SQL Compliance)**: PostgreSQL tuân thủ chặt chẽ các tiêu chuẩn SQL, giúp các câu lệnh viết cho PostgreSQL có tính di động cao và dễ học đối với những người đã quen với SQL.
*   **Mã nguồn mở và cộng đồng lớn (Open-Source & Large Community)**: Cộng đồng phát triển và người dùng rộng lớn đảm bảo sự hỗ trợ liên tục, cập nhật thường xuyên và khả năng mở rộng.

PostgreSQL là lựa chọn hàng đầu cho nhiều ứng dụng từ các dự án nhỏ đến các hệ thống doanh nghiệp lớn, đặc biệt khi cần độ tin cậy cao và các tính năng SQL nâng cao.

### 1.3. Mô Hình Tương Tác: Client-Server và Ngôn Ngữ SQL

Để làm việc với cơ sở dữ liệu PostgreSQL, chúng ta sử dụng một mô hình **client-server**.
*   **Máy chủ (Server)**: Là quá trình (process) chạy PostgreSQL, quản lý các tệp dữ liệu, thực hiện các truy vấn, và xử lý các yêu cầu từ máy khách.
*   **Máy khách (Client)**: Là bất kỳ ứng dụng hoặc công cụ nào kết nối với máy chủ PostgreSQL để gửi các yêu cầu và nhận kết quả. Các máy khách phổ biến bao gồm:
    *   **Công cụ dòng lệnh**: Như `psql` (công cụ mặc định của PostgreSQL).
    *   **Ứng dụng GUI**: Như DBeaver, pgAdmin, DataGrip.
    *   **Ứng dụng web/di động**: Các backend server (Node.js, Python, Java) kết nối đến PostgreSQL.
    *   **Công cụ AI**: Như Antigravity IDE mà chúng ta sẽ thảo luận.

Khi máy khách kết nối với cơ sở dữ liệu, nó sẽ gửi các câu lệnh được viết bằng **SQL (Structured Query Language)**. SQL là ngôn ngữ tiêu chuẩn và phổ biến nhất để giao tiếp với cơ sở dữ liệu quan hệ. Nó được chia thành nhiều loại, nhưng trong phần này, chúng ta sẽ tập trung vào:
*   **DDL (Data Definition Language)**: Các lệnh để định nghĩa hoặc quản lý cấu trúc cơ sở dữ liệu (ví dụ: `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`).
*   **DML (Data Manipulation Language)**: Các lệnh để thao tác dữ liệu bên trong các bảng (ví dụ: `INSERT`, `SELECT`, `UPDATE`, `DELETE`).

> [!NOTE]
> Khóa học này đặc biệt nhấn mạnh vào **cú pháp chuẩn của PostgreSQL**. Mặc dù SQL là ngôn ngữ chung, mỗi RDBMS có thể có những biến thể hoặc tính năng mở rộng riêng. Chúng ta sẽ tránh nhầm lẫn với các hệ quản trị cơ sở dữ liệu khác như MySQL hay SQL Server để đảm bảo tính chính xác và hiệu quả khi làm việc với PostgreSQL.

### 1.4. Cơ Chế Hoạt Động Ngầm (Under the Hood) của SQL

Khi bạn gửi một câu lệnh SQL từ máy khách đến máy chủ PostgreSQL, một loạt các bước sẽ diễn ra:
1.  **Phân tích cú pháp (Parsing)**: Máy chủ kiểm tra cú pháp của câu lệnh SQL để đảm bảo nó hợp lệ.
2.  **Kiểm tra ngữ nghĩa (Semantic Analysis)**: Kiểm tra xem các bảng, cột, và hàm được tham chiếu có tồn tại và người dùng có quyền truy cập hay không.
3.  **Tối ưu hóa truy vấn (Query Optimization)**: Đây là một bước quan trọng. Bộ tối ưu hóa truy vấn (query optimizer) phân tích câu lệnh và tạo ra một "kế hoạch thực thi" (execution plan) hiệu quả nhất để lấy dữ liệu. Ví dụ, nó sẽ quyết định có nên sử dụng chỉ mục (index) hay không, hoặc thứ tự các bảng nên được nối (join) với nhau.
4.  **Thực thi (Execution)**: Kế hoạch thực thi được chạy. Máy chủ truy cập các tệp dữ liệu vật lý trên ổ đĩa, thực hiện các phép tính và tổng hợp dữ liệu.
5.  **Trả về kết quả (Result Return)**: Dữ liệu kết quả (hoặc thông báo thành công/thất bại) được gửi trở lại máy khách.

Việc hiểu cơ chế này giúp bạn viết các câu lệnh SQL không chỉ đúng cú pháp mà còn hiệu quả về hiệu suất.

## 2. Thách Thức và Quy Trình Phát Triển Cơ Sở Dữ Liệu Hiệu Quả

Phát triển với cơ sở dữ liệu là một quá trình phức tạp, đòi hỏi sự kết hợp giữa tư duy thiết kế hệ thống và kỹ năng lập trình. Trong khóa học này, chúng ta sẽ tập trung giải quyết bốn thách thức chính:

1.  **Thiết kế lược đồ (schema) cơ sở dữ liệu tối ưu**: Xây dựng cấu trúc bảng, định nghĩa các mối quan hệ và ràng buộc để lưu trữ dữ liệu một cách logic, giảm thiểu trùng lặp và đảm bảo tính toàn vẹn. Một thiết kế tốt là nền tảng cho hiệu suất và khả năng mở rộng.
2.  **Viết các truy vấn SQL hiệu quả**: Khai thác sức mạnh của SQL để truy xuất, thêm, cập nhật và xóa dữ liệu một cách nhanh chóng, chính xác, đặc biệt là với các bộ dữ liệu lớn và các yêu cầu phức tạp.
3.  **Sử dụng các tính năng nâng cao của PostgreSQL**: Nắm vững các tính năng độc đáo và mạnh mẽ của PostgreSQL để giải quyết các vấn đề cụ thể, tối ưu hóa hiệu suất và mở rộng khả năng của ứng dụng.
4.  **Quản lý cơ sở dữ liệu trong môi trường sản xuất**: Học cách thực hiện các tác vụ quản trị quan trọng như sao lưu (backup), phục hồi (recovery), giám sát hiệu suất và mở rộng (scaling) cơ sở dữ liệu để đáp ứng nhu cầu thực tế của người dùng.

Trong Phần 1 này, chúng ta sẽ tập trung vào hai thách thức đầu tiên: **thiết kế cơ sở dữ liệu quan hệ** và **viết các truy vấn DDL/DML cơ bản** để bắt đầu thao tác với dữ liệu.

## 3. Thiết Kế Lược Đồ Cơ Sở Dữ Liệu Quan Hệ

Thiết kế cơ sở dữ liệu là bước đầu tiên và quan trọng nhất, quyết định sự thành công và hiệu suất của toàn bộ hệ thống. Một thiết kế kém có thể dẫn đến dữ liệu trùng lặp, khó bảo trì và hiệu suất kém.

### 3.1. Tư Duy Thiết Kế: Từ Thực Thể đến Cấu Trúc Bảng

Quy trình thiết kế cơ bản mà chúng ta sẽ áp dụng xoay quanh việc phân tích các đối tượng trong thế giới thực và chuyển đổi chúng thành cấu trúc cơ sở dữ liệu:

1.  **Chúng ta đang lưu trữ loại đối tượng (entity) nào?** Xác định các "danh từ" chính trong hệ thống của bạn (ví dụ: người dùng, sản phẩm, đơn hàng, thành phố). Mỗi đối tượng này sẽ thường trở thành một bảng.
2.  **Đối tượng này có những thuộc tính (attribute) nào?** Liệt kê các đặc điểm hoặc thông tin mà bạn muốn lưu trữ về mỗi đối tượng (ví dụ: tên, giá, ngày tạo). Mỗi thuộc tính sẽ trở thành một cột trong bảng.
3.  **Mỗi thuộc tính đó chứa loại dữ liệu (data type) nào?** Xác định kiểu dữ liệu phù hợp nhất cho từng thuộc tính (ví dụ: chuỗi ký tự, số nguyên, ngày tháng).

**Ví dụ: Thiết kế bảng lưu trữ danh sách các thành phố**

Hãy áp dụng quy trình này để lưu trữ thông tin về các thành phố lớn trên thế giới.

1.  **Loại đối tượng?** Chúng ta muốn lưu trữ thông tin về các **Thành phố**.
2.  **Thuộc tính của Thành phố?** Mỗi thành phố cần các thông tin sau:
    *   Tên (Name)
    *   Quốc gia (Country)
    *   Dân số (Population)
    *   Diện tích (Area)
    *   Một mã định danh duy nhất (ID)
3.  **Loại dữ liệu cho từng thuộc tính?**
    *   ID: Số nguyên tự động tăng (để đảm bảo tính duy nhất).
    *   Tên: Chuỗi ký tự.
    *   Quốc gia: Chuỗi ký tự.
    *   Dân số: Số nguyên (có thể rất lớn).
    *   Diện tích: Số nguyên.

### 3.2. Các Khái Niệm Cốt Lõi: Bảng, Cột, Hàng và Khóa Chính

Từ các câu trả lời trên, chúng ta có thể hình dung cấu trúc cơ sở dữ liệu quan hệ:

*   **Bảng (Table)**: Là đơn vị lưu trữ cơ bản trong cơ sở dữ liệu quan hệ, nơi một tập hợp các bản ghi có cùng ý nghĩa được tổ chức. Trong ví dụ của chúng ta, chúng ta sẽ tạo một bảng có tên là `cities`. Một cơ sở dữ liệu có thể chứa nhiều bảng, mỗi bảng đại diện cho một loại đối tượng (ví dụ: `users`, `products`, `orders`).

    > [!NOTE]
    > Tên bảng (và tên cột) trong PostgreSQL theo quy ước thường được viết thường, sử dụng dấu gạch dưới (`_`) để phân tách các từ (snake_case). Mặc dù SQL không phân biệt chữ hoa chữ thường đối với các từ khóa, nhưng việc tuân thủ quy ước này giúp mã dễ đọc và tránh các vấn đề tiềm ẩn khi sử dụng các công cụ khác nhau hoặc khi cần bao quanh tên bằng dấu ngoặc kép (`"`).

*   **Cột (Column)**: Mỗi bảng có nhiều cột, và mỗi cột đại diện cho một thuộc tính cụ thể của đối tượng mà bảng đó lưu trữ. Dựa trên thiết kế, bảng `cities` sẽ có năm cột: `id`, `name`, `country`, `population`, và `area`.

*   **Hàng (Row)**: Mỗi hàng trong bảng đại diện cho một bản ghi riêng lẻ của đối tượng. Khi chúng ta thêm dữ liệu về Tokyo, Delhi hay Thượng Hải, mỗi thành phố này sẽ là một hàng riêng biệt trong bảng `cities`.

    > [!TIP]
    > Bạn có thể hình dung một bảng cơ sở dữ liệu như một bảng tính Excel: các cột là tiêu đề cột, và mỗi hàng là một dòng dữ liệu.

*   **Khóa chính (Primary Key)**: Là một hoặc nhiều cột mà giá trị của chúng dùng để định danh duy nhất mỗi hàng trong bảng. Một bảng chỉ có thể có một khóa chính. Khóa chính đảm bảo rằng không có hai hàng nào có cùng một giá trị cho cột khóa chính, và giá trị này không bao giờ được phép là `NULL` (rỗng).
    *   Trong ví dụ `cities`, cột `id` sẽ là khóa chính, đảm bảo mỗi thành phố có một mã định danh duy nhất. Khóa chính là cực kỳ quan trọng cho việc tham chiếu và duy trì tính toàn vẹn dữ liệu.

### 3.3. Các Kiểu Dữ Liệu Chuẩn trong PostgreSQL

Việc chọn đúng kiểu dữ liệu cho mỗi cột là rất quan trọng để tối ưu hóa việc lưu trữ, hiệu suất truy vấn và đảm bảo tính toàn vẹn của dữ liệu. Dưới đây là các kiểu dữ liệu phổ biến trong PostgreSQL:

*   **Kiểu Chuỗi Ký Tự (Character Types)**:
    *   **`VARCHAR(n)`**: Chuỗi ký tự có độ dài biến đổi, với giới hạn tối đa `n` ký tự. Nếu bạn cố gắng chèn một chuỗi dài hơn `n`, PostgreSQL sẽ báo lỗi.
    *   **`TEXT`**: Chuỗi ký tự có độ dài biến đổi mà không có giới hạn độ dài cứng (thực tế giới hạn bởi dung lượng hệ thống). Thường được sử dụng cho các đoạn văn bản dài hơn như mô tả sản phẩm, nội dung bài viết.
    *   > [!NOTE]
        > Về mặt hiệu suất, đối với các chuỗi có độ dài điển hình (dưới vài trăm ký tự), sự khác biệt giữa `VARCHAR(n)` và `TEXT` trong PostgreSQL hiện đại là không đáng kể. `VARCHAR(n)` chủ yếu cung cấp một ràng buộc kiểm tra độ dài.

*   **Kiểu Số Nguyên (Integer Types)**:
    *   **`SMALLINT`**: Số nguyên nhỏ, phạm vi từ -32,768 đến +32,767.
    *   **`INTEGER` (hoặc `INT`)**: Số nguyên tiêu chuẩn, phạm vi từ khoảng -2 tỷ đến +2 tỷ. Thường đủ cho hầu hết các trường hợp.
    *   **`BIGINT`**: Số nguyên lớn, phạm vi từ khoảng -9 triệu tỷ đến +9 triệu tỷ. Hữu ích cho dân số của các quốc gia, ID giao dịch, hoặc các giá trị đếm rất lớn.
    *   **`SERIAL` / `BIGSERIAL`**: Đây không phải là một kiểu dữ liệu độc lập mà là một cú pháp tiện lợi (syntactic sugar) của PostgreSQL. `SERIAL` tự động tạo một cột `INTEGER` với một chuỗi (sequence) được liên kết để tự động tăng giá trị và áp dụng ràng buộc `NOT NULL`. `BIGSERIAL` tương tự nhưng tạo một cột `BIGINT`. Chúng thường được sử dụng cho các khóa chính tự động tăng.

*   **Kiểu Số Thập Phân (Numeric Types)**:
    *   **`NUMERIC(p, s)` hoặc `DECIMAL(p, s)`**: Kiểu dữ liệu số chính xác với `p` tổng số chữ số (precision) và `s` chữ số sau dấu thập phân (scale). Thích hợp cho các giá trị tiền tệ, tính toán khoa học yêu cầu độ chính xác cao.
    *   **`REAL` / `DOUBLE PRECISION`**: Kiểu số dấu phẩy động (floating-point) với độ chính xác tương ứng. Thích hợp cho các giá trị gần đúng như tọa độ địa lý. Tránh dùng cho tiền tệ hoặc các phép tính cần độ chính xác tuyệt đối do vấn đề làm tròn số của số dấu phẩy động.

*   **Kiểu Boolean**:
    *   **`BOOLEAN`**: Lưu trữ giá trị logic `TRUE`, `FALSE` hoặc `NULL`.

*   **Kiểu Ngày Giờ (Date/Time Types)**:
    *   **`DATE`**: Lưu trữ ngày (ví dụ: '2023-10-27').
    *   **`TIME`**: Lưu trữ thời gian (ví dụ: '14:30:00').
    *   **`TIMESTAMP`**: Lưu trữ ngày và giờ. Có thể có múi giờ (`TIMESTAMP WITH TIME ZONE`) hoặc không (`TIMESTAMP WITHOUT TIME ZONE`).
    *   **`TIMESTAMPTZ`**: Viết tắt của `TIMESTAMP WITH TIME ZONE`. Đây là kiểu dữ liệu khuyên dùng khi bạn cần lưu trữ thời điểm chính xác trên toàn cầu, vì nó chuyển đổi và lưu trữ thời gian theo múi giờ UTC.

> [!TIP]
> Luôn chọn kiểu dữ liệu phù hợp nhất với loại thông tin bạn muốn lưu trữ. Việc này không chỉ giúp tiết kiệm không gian lưu trữ mà còn cải thiện hiệu suất truy vấn và đảm bảo tính toàn vẹn dữ liệu. Ví dụ, không nên dùng `TEXT` cho một trường chỉ cần `VARCHAR(50)`, hoặc `BIGINT` nếu `INTEGER` là đủ.

## 4. Môi Trường Thực Hành: pgsql.com và Antigravity IDE

Để bắt đầu thực hành ngay lập tức, chúng ta sẽ sử dụng một công cụ trực tuyến và tích hợp cách tiếp cận của Antigravity IDE.

### 4.1. pgsql.com: Nền Tảng Thực Hành Trực Tuyến

**pgsql.com** là một ứng dụng web cung cấp một môi trường cơ sở dữ liệu PostgreSQL ảo miễn phí, cho phép bạn chạy các câu lệnh SQL trực tiếp từ trình duyệt mà không cần cài đặt.

**Giao diện pgsql.com:**
*   **Trình soạn thảo mã (Code Editor)** (góc trên bên trái): Nơi bạn nhập các câu lệnh SQL.
*   **Kết quả truy vấn (Query Results)** (góc dưới bên trái): Hiển thị kết quả sau khi thực thi SQL.
*   **Lịch sử truy vấn (Query History)** (phía bên phải): Ghi lại các câu lệnh bạn đã chạy.

> [!NOTE]
> pgsql.com hoạt động như một máy khách, gửi các câu lệnh SQL của bạn đến một máy chủ PostgreSQL thực sự ở phía backend, sau đó nhận và hiển thị kết quả. Đây là một cách tuyệt vời để làm quen với SQL. Sau này trong khóa học, chúng ta sẽ tìm hiểu cách cài đặt PostgreSQL cục bộ trên máy tính của bạn để có môi trường phát triển đầy đủ hơn.

### 4.2. Tối Ưu Hóa Quy Trình với Antigravity IDE và Vibe Coding

Antigravity IDE là một hệ thống Agentic AI tiên tiến mà bạn đang sử dụng, có khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file, và lập kế hoạch tự động. Đây là một công cụ cực kỳ mạnh mẽ để tăng tốc quá trình phát triển cơ sở dữ liệu thông qua tư duy **Vibe Coding**.

**Vibe Coding** trong ngữ cảnh này là một phương pháp tiếp cận phát triển phần mềm dựa trên sự hợp tác chặt chẽ với AI, nơi bạn cung cấp "ý tưởng" hoặc "mục tiêu" (the vibe), và Antigravity IDE sẽ chuyển đổi chúng thành mã nguồn hoặc hành động cụ thể.

**Áp dụng Antigravity IDE và Vibe Coding vào phát triển PostgreSQL:**

1.  **Thiết kế lược đồ thông minh hơn:**
    *   **Tư duy truyền thống:** Bạn phải tự mình nghĩ ra tất cả các bảng, cột, kiểu dữ liệu và ràng buộc.
    *   **Với Antigravity IDE (Vibe Coding):** Bạn có thể bắt đầu với một yêu cầu cấp cao.
        *   **Prompt ví dụ:** "Tôi muốn lưu trữ thông tin về các thành phố lớn. Mỗi thành phố có tên, quốc gia, dân số và diện tích. Hãy đề xuất một cấu trúc bảng PostgreSQL với các kiểu dữ liệu và ràng buộc phù hợp, bao gồm một khóa chính tự động tăng."
        *   Antigravity sẽ phân tích yêu cầu, đề xuất một câu lệnh `CREATE TABLE` hoàn chỉnh, và có thể giải thích lý do chọn từng kiểu dữ liệu hoặc ràng buộc. Bạn chỉ cần xem xét, tinh chỉnh và xác nhận.

2.  **Tạo câu lệnh SQL nhanh chóng và chính xác:**
    *   **Tư duy truyền thống:** Phải nhớ cú pháp chính xác, tên cột, tên bảng.
    *   **Với Antigravity IDE (Vibe Coding):** Diễn đạt ý định của bạn bằng ngôn ngữ tự nhiên.
        *   **Prompt ví dụ:** "Thêm thành phố 'Paris' vào bảng `cities` với dân số 2.1 triệu và diện tích 105 km vuông, thuộc 'Pháp'."
        *   Antigravity sẽ tạo ra câu lệnh `INSERT INTO` đúng cú pháp. Nó cũng có thể giúp bạn tạo các câu lệnh `SELECT` phức tạp để truy vấn dữ liệu cụ thể.

3.  **Khám phá và phân tích dữ liệu:**
    *   **Tư duy truyền thống:** Viết nhiều câu lệnh `SELECT` khác nhau để tìm kiếm thông tin.
    *   **Với Antigravity IDE (Vibe Coding):** Đặt câu hỏi trực tiếp về dữ liệu.
        *   **Prompt ví dụ:** "Hiển thị tên và dân số của 5 thành phố đông dân nhất trong bảng `cities`."
        *   Antigravity có thể tạo ra một truy vấn `SELECT` với `ORDER BY` và `LIMIT` để trả lời câu hỏi đó.

4.  **Học hỏi và gỡ lỗi hiệu quả:**
    *   Khi gặp lỗi SQL, thay vì tra cứu thủ công, bạn có thể dán lỗi vào Antigravity.
    *   **Prompt ví dụ:** "Tôi gặp lỗi này khi chạy câu lệnh `CREATE TABLE`: [dán thông báo lỗi]. Hãy giải thích lỗi và đề xuất cách khắc phục."
    *   Antigravity sẽ giúp bạn hiểu nguyên nhân gốc rễ và cung cấp giải pháp.

**Tóm lại, Antigravity IDE cho phép bạn: **
*   **Tập trung vào "cái gì" thay vì "làm thế nào"**: Bạn mô tả mục tiêu, Antigravity lo phần cú pháp và chi tiết kỹ thuật.
*   **Tăng tốc độ phát triển**: Giảm thời gian viết và gỡ lỗi SQL.
*   **Nâng cao chất lượng mã**: AI có thể đề xuất các phương pháp hay nhất và phát hiện lỗi tiềm ẩn.

Trong các phần tiếp theo, hãy luôn nghĩ về cách bạn có thể tận dụng Antigravity IDE để hỗ trợ quá trình học và thực hành của mình.

## 5. Ngôn Ngữ Định Nghĩa Dữ Liệu (DDL): Tạo Cấu Trúc Bảng

**DDL (Data Definition Language)** là tập hợp các câu lệnh SQL được sử dụng để định nghĩa, sửa đổi hoặc xóa cấu trúc của cơ sở dữ liệu. Lệnh `CREATE TABLE` là lệnh DDL cơ bản nhất, dùng để tạo một bảng mới.

### 5.1. `CREATE TABLE`: Định Hình Bảng Dữ Liệu

`CREATE TABLE` cho phép bạn xác định tên bảng, tên các cột, kiểu dữ liệu của từng cột và các ràng buộc (constraints) trên dữ liệu.

**Cú pháp cơ bản:**

```sql
CREATE TABLE ten_bang (
    ten_cot1 kieu_du_lieu1 [cac_rang_buoc],
    ten_cot2 kieu_du_lieu2 [cac_rang_buoc],
    ...
    ten_cotN kieu_du_lieuN [cac_rang_buoc]
);
```

*   **`CREATE TABLE`**: Từ khóa SQL báo hiệu bạn muốn tạo một bảng mới.
*   **`ten_bang`**: Tên định danh của bảng (ví dụ: `cities`).
*   **`ten_cot`**: Tên định danh của cột (ví dụ: `name`, `country`).
*   **`kieu_du_lieu`**: Kiểu dữ liệu mà cột đó sẽ lưu trữ (ví dụ: `VARCHAR(50)`, `INTEGER`).
*   **`[cac_rang_buoc]`**: Các ràng buộc tùy chọn để áp đặt quy tắc lên dữ liệu trong cột (ví dụ: `NOT NULL`, `PRIMARY KEY`).

> [!NOTE]
> Trong SQL, **từ khóa** (như `CREATE TABLE`) thường được viết hoa để dễ đọc và phân biệt, trong khi **định danh** (như tên bảng, tên cột) thường được viết thường. Mỗi định nghĩa cột phải được phân tách bằng dấu phẩy, trừ cột cuối cùng. Câu lệnh SQL kết thúc bằng dấu chấm phẩy (`;`).

**Ví dụ: Tạo bảng `cities` cơ bản**

Dựa trên thiết kế ban đầu của chúng ta, hãy tạo bảng `cities` trên pgsql.com:

```sql
-- Tạo bảng cities để lưu trữ thông tin về các thành phố
CREATE TABLE cities (
    name VARCHAR(50),      -- Tên thành phố, tối đa 50 ký tự
    country VARCHAR(50),   -- Tên quốc gia, tối đa 50 ký tự
    population INTEGER,    -- Dân số, kiểu số nguyên
    area INTEGER           -- Diện tích, kiểu số nguyên
);
```

Khi bạn thực thi lệnh này, PostgreSQL sẽ tạo một cấu trúc bảng rỗng. Về mặt kỹ thuật, máy chủ cơ sở dữ liệu sẽ cập nhật các bảng hệ thống (catalog tables) để ghi lại lược đồ mới này và có thể chuẩn bị các tệp vật lý trên ổ đĩa để lưu trữ dữ liệu của bảng `cities`.

### 5.2. Các Ràng Buộc (Constraints) và Tính Toàn Vẹn Dữ Liệu

Ràng buộc là các quy tắc được áp dụng cho các cột hoặc toàn bộ bảng để giới hạn loại dữ liệu có thể được lưu trữ. Chúng là công cụ quan trọng để đảm bảo **tính toàn vẹn dữ liệu (data integrity)**, nghĩa là dữ liệu trong cơ sở dữ liệu phải chính xác, nhất quán và đáng tin cậy.

Các ràng buộc cột phổ biến trong PostgreSQL:

*   **`NOT NULL`**: Đảm bảo rằng một cột không bao giờ được phép chứa giá trị `NULL`. `NULL` biểu thị sự vắng mặt của giá trị, không phải là 0 hoặc chuỗi rỗng. Ví dụ, tên thành phố không thể để trống.
*   **`PRIMARY KEY`**: Định danh duy nhất cho mỗi hàng trong bảng. Một bảng chỉ có thể có một khóa chính. Khóa chính ngụ ý `NOT NULL` và `UNIQUE`. Đây là ràng buộc quan trọng nhất để thiết lập mối quan hệ giữa các bảng.
*   **`UNIQUE`**: Đảm bảo rằng tất cả các giá trị trong một cột (hoặc một nhóm cột) là duy nhất. Khác với `PRIMARY KEY`, một bảng có thể có nhiều ràng buộc `UNIQUE`, và cột `UNIQUE` có thể chứa giá trị `NULL` (chỉ một lần).
*   **`DEFAULT value`**: Gán một giá trị mặc định cho cột nếu không có giá trị nào được cung cấp khi chèn dữ liệu mới.
*   **`CHECK (condition)`**: Đảm bảo rằng các giá trị trong một cột đáp ứng một điều kiện cụ thể (ví dụ: dân số phải là số dương).

**Ví dụ nâng cao: Tạo bảng `cities` với ràng buộc đầy đủ**

```sql
-- Tạo bảng cities với các ràng buộc cơ bản và nâng cao
CREATE TABLE cities (
    id BIGSERIAL PRIMARY KEY,           -- Khóa chính tự động tăng (BIGINT), duy nhất cho mỗi thành phố
    name VARCHAR(100) NOT NULL UNIQUE,  -- Tên thành phố, không được rỗng và phải là duy nhất
    country VARCHAR(50) NOT NULL,       -- Tên quốc gia, không được rỗng
    population BIGINT CHECK (population >= 0), -- Dân số, sử dụng BIGINT, phải là số không âm
    area INTEGER CHECK (area > 0),      -- Diện tích, phải là số dương
    created_at TIMESTAMPTZ DEFAULT NOW() -- Thời gian tạo bản ghi, mặc định là thời gian hiện tại theo múi giờ UTC
);
```

**Giải thích bổ sung:**
*   `id BIGSERIAL PRIMARY KEY`: Cột `id` sẽ là khóa chính, tự động tăng giá trị và đảm bảo mỗi hàng có một mã định danh duy nhất. `BIGSERIAL` là lựa chọn tốt hơn `SERIAL` cho các bảng có tiềm năng chứa rất nhiều bản ghi.
*   `name VARCHAR(100) NOT NULL UNIQUE`: Yêu cầu tên thành phố không được trống và không có hai thành phố nào có cùng tên (trong ngữ cảnh này, chúng ta giả định tên thành phố là duy nhất trên toàn cầu, hoặc ít nhất là trong phạm vi ứng dụng của chúng ta).
*   `population BIGINT CHECK (population >= 0)`: Sử dụng `BIGINT` cho dân số lớn và thêm ràng buộc `CHECK` để đảm bảo dân số luôn là số không âm.
*   `created_at TIMESTAMPTZ DEFAULT NOW()`: Tự động ghi lại thời điểm một hàng được tạo. `NOW()` là một hàm của PostgreSQL trả về thời gian hiện tại của máy chủ, và `TIMESTAMPTZ` sẽ lưu trữ nó dưới dạng UTC.

> [!TIP]
> **Vibe Coding với Antigravity IDE**: Khi thiết kế bảng, bạn có thể mô tả các quy tắc nghiệp vụ bằng ngôn ngữ tự nhiên cho Antigravity IDE. Ví dụ: "Tạo bảng `products`. Mỗi sản phẩm có một ID duy nhất, tên không được trống, giá phải dương và mô tả có thể dài. Cần thêm trường ngày tạo và ngày cập nhật tự động." Antigravity sẽ chuyển đổi yêu cầu này thành câu lệnh `CREATE TABLE` đầy đủ với các ràng buộc và kiểu dữ liệu phù hợp.

## 6. Ngôn Ngữ Thao Tác Dữ Liệu (DML): Chèn và Truy Vấn Dữ Liệu

**DML (Data Manipulation Language)** là tập hợp các câu lệnh SQL được sử dụng để thao tác dữ liệu bên trong các bảng. Chúng ta sẽ tìm hiểu hai lệnh DML cơ bản nhất: `INSERT INTO` để thêm dữ liệu và `SELECT` để truy xuất dữ liệu.

### 6.1. `INSERT INTO`: Đưa Dữ Liệu vào Bảng

Lệnh `INSERT INTO` được sử dụng để thêm một hoặc nhiều hàng dữ liệu mới vào một bảng.

**Cú pháp cơ bản:**

```sql
INSERT INTO ten_bang (cot1, cot2, cot3, ...)
VALUES (gia_tri1, gia_tri2, gia_tri3, ...);
```

*   **`INSERT INTO`**: Từ khóa SQL để chèn dữ liệu.
*   **`ten_bang`**: Tên của bảng bạn muốn chèn dữ liệu vào.
*   `(cot1, cot2, ...)`: Danh sách các cột mà bạn muốn cung cấp giá trị. **Thứ tự của các cột này phải khớp với thứ tự của các giá trị trong mệnh đề `VALUES`.**
*   **`VALUES`**: Từ khóa bắt đầu danh sách các giá trị sẽ được chèn.
*   `(gia_tri1, gia_tri2, ...)`: Danh sách các giá trị tương ứng với các cột đã liệt kê.
    *   Giá trị chuỗi (`VARCHAR`, `TEXT`) phải được đặt trong dấu nháy đơn (`'`).
    *   Giá trị số (`INTEGER`, `BIGINT`, `NUMERIC`) không cần dấu nháy đơn.
    *   Giá trị boolean là `TRUE` hoặc `FALSE`.
    *   Nếu một cột có giá trị mặc định (như `created_at` với `DEFAULT NOW()`) hoặc là `SERIAL`/`BIGSERIAL`, bạn có thể bỏ qua nó trong danh sách cột và giá trị, và PostgreSQL sẽ tự động điền.

> [!IMPORTANT]
> Nếu bạn không liệt kê các cột sau `ten_bang`, bạn phải cung cấp giá trị cho **TẤT CẢ** các cột trong bảng theo đúng thứ tự chúng được định nghĩa khi tạo bảng. Điều này dễ gây lỗi và không được khuyến khích. Luôn luôn liệt kê rõ ràng các cột bạn muốn chèn.

**Ví dụ: Chèn dữ liệu vào bảng `cities` (sử dụng bảng có `id BIGSERIAL`)**

Giả sử chúng ta đã tạo bảng `cities` như trong ví dụ nâng cao ở trên.

**Chèn một hàng:**

```sql
-- Chèn dữ liệu cho thành phố Tokyo
INSERT INTO cities (name, country, population, area)
VALUES ('Tokyo', 'Japan', 38505000, 8223);
```
*   Ở đây, chúng ta không cung cấp giá trị cho `id` (vì nó là `BIGSERIAL` tự động tăng) và `created_at` (vì nó có `DEFAULT NOW()`). PostgreSQL sẽ tự động điền các giá trị này.
*   Sau khi chạy lệnh, bạn sẽ thấy thông báo "INSERT 0 1" (PostgreSQL trả về OID và số hàng bị ảnh hưởng).

**Chèn nhiều hàng cùng lúc:**

Để chèn nhiều hàng trong một câu lệnh, bạn có thể liệt kê nhiều tập hợp giá trị, mỗi tập hợp cách nhau bằng dấu phẩy:

```sql
-- Chèn dữ liệu cho Delhi, Shanghai và Sao Paulo trong một câu lệnh
INSERT INTO cities (name, country, population, area)
VALUES
    ('Delhi', 'India', 20125000, 2240),
    ('Shanghai', 'China', 22000000, 6340),
    ('Sao Paulo', 'Brazil', 20935000, 3043);
```
Lệnh này sẽ chèn ba hàng mới vào bảng `cities` một cách hiệu quả hơn so với việc chạy ba lệnh `INSERT` riêng lẻ.

> [!TIP]
> **Vibe Coding với Antigravity IDE**: Nếu bạn có dữ liệu trong một định dạng khác (ví dụ: danh sách Python, JSON, CSV), bạn có thể yêu cầu Antigravity IDE chuyển đổi nó thành các câu lệnh `INSERT` phù hợp. Ví dụ: "Tôi có dữ liệu sau về các thành phố: [dán dữ liệu]. Hãy tạo các câu lệnh INSERT INTO cho bảng `cities`."

### 6.2. `SELECT`: Truy Xuất Thông Tin

Lệnh `SELECT` là lệnh DML được sử dụng để truy xuất dữ liệu từ một hoặc nhiều bảng. Đây là một trong những lệnh SQL được sử dụng thường xuyên nhất và là cốt lõi của việc làm việc với cơ sở dữ liệu.

**Cú pháp cơ bản:**

```sql
SELECT cot1, cot2, ...
FROM ten_bang;
```

*   **`SELECT`**: Từ khóa SQL để chỉ định rằng bạn muốn truy xuất dữ liệu.
*   **`cot1, cot2, ...`**: Danh sách các cột bạn muốn hiển thị trong kết quả.
*   **`FROM`**: Từ khóa để chỉ định bảng mà bạn muốn truy xuất dữ liệu.
*   **`ten_bang`**: Tên của bảng chứa dữ liệu.

**Cơ chế hoạt động ngầm (Under the Hood):** Khi bạn gửi một truy vấn `SELECT`, bộ tối ưu hóa truy vấn của PostgreSQL sẽ xác định cách hiệu quả nhất để tìm và trả về dữ liệu. Điều này có thể bao gồm việc quét toàn bộ bảng (table scan) hoặc sử dụng các chỉ mục (indexes) nếu có để nhanh chóng định vị các hàng cần thiết.

**Truy xuất tất cả các cột (`SELECT *`)**

Để truy xuất tất cả các cột từ một bảng, bạn có thể sử dụng ký tự đại diện `*` (dấu sao):

```sql
-- Truy xuất tất cả các cột từ bảng cities
SELECT *
FROM cities;
```
Kết quả sẽ hiển thị tất cả các hàng và tất cả các cột của bảng `cities`, bao gồm cả `id` và `created_at` được tạo tự động.

> [!NOTE]
> Mặc dù `SELECT *` tiện lợi cho việc khám phá dữ liệu hoặc trong môi trường phát triển, nó thường không được khuyến khích trong mã nguồn sản xuất hoặc các ứng dụng thực tế. Lý do:
> *   **Hiệu suất**: Việc truy xuất tất cả các cột có thể tốn tài nguyên hơn nếu bạn chỉ cần một vài cột.
> *   **Tính ổn định**: Nếu cấu trúc bảng thay đổi (thêm/bớt cột), `SELECT *` có thể trả về các cột không mong muốn hoặc phá vỡ các ứng dụng phụ thuộc vào thứ tự cột cụ thể.
> *   **Băng thông**: Truyền tải nhiều dữ liệu hơn mức cần thiết qua mạng.
>   Luôn cố gắng liệt kê rõ ràng các cột bạn cần.

**Truy xuất các cột cụ thể**

Nếu bạn chỉ muốn xem một số cột nhất định, hãy liệt kê tên các cột đó, cách nhau bằng dấu phẩy:

```sql
-- Truy xuất chỉ các cột name và country từ bảng cities
SELECT name, country
FROM cities;
```

Bạn cũng có thể thay đổi thứ tự các cột trong câu lệnh `SELECT`, và chúng sẽ xuất hiện theo thứ tự đó trong kết quả:

```sql
-- Truy xuất area, name và population từ bảng cities theo thứ tự này
SELECT area, name, population
FROM cities;
```

> [!TIP]
> **Vibe Coding với Antigravity IDE**: Thay vì phải nhớ tên cột, bạn có thể hỏi Antigravity các câu hỏi bằng ngôn ngữ tự nhiên. Ví dụ: "Hiện thị tên và dân số của tất cả các thành phố." hoặc "Tôi muốn xem diện tích của từng thành phố, sau đó là tên và dân số của chúng." Antigravity sẽ tự động tạo truy vấn `SELECT` phù hợp.

## Tóm Tắt Phần 1: Giới Thiệu và Các Lệnh SQL Cơ Bản

*   **Cơ sở dữ liệu** là hệ thống có tổ chức để lưu trữ và quản lý thông tin, bao gồm dữ liệu, lược đồ và hệ quản trị (DBMS).
*   **PostgreSQL** là một RDBMS mã nguồn mở mạnh mẽ, đáng tin cậy, tuân thủ SQL và giàu tính năng.
*   Tương tác với PostgreSQL theo mô hình **client-server** sử dụng ngôn ngữ **SQL**.
*   **SQL** được chia thành **DDL** (định nghĩa cấu trúc) và **DML** (thao tác dữ liệu).
*   **Thiết kế cơ sở dữ liệu** bắt đầu bằng việc xác định đối tượng, thuộc tính, kiểu dữ liệu, và đặc biệt là **khóa chính** để đảm bảo tính duy nhất và toàn vẹn.
*   **Bảng (Table)**, **Cột (Column)**, **Hàng (Row)** là các khái niệm cốt lõi trong cơ sở dữ liệu quan hệ.
*   **Kiểu dữ liệu** như `VARCHAR`, `TEXT`, `INTEGER`, `BIGINT`, `NUMERIC`, `BOOLEAN`, `DATE`, `TIMESTAMP`, `SERIAL`/`BIGSERIAL` phải được chọn cẩn thận để tối ưu hóa lưu trữ và hiệu suất.
*   **pgsql.com** là môi trường trực tuyến tiện lợi để thực hành SQL.
*   **Antigravity IDE** và **Vibe Coding** là phương pháp mạnh mẽ để tăng tốc thiết kế, tạo và gỡ lỗi SQL thông qua tương tác AI bằng ngôn ngữ tự nhiên.
*   **`CREATE TABLE`** (DDL) được sử dụng để định nghĩa cấu trúc bảng, bao gồm tên cột, kiểu dữ liệu và các ràng buộc (`NOT NULL`, `PRIMARY KEY`, `UNIQUE`, `DEFAULT`, `CHECK`).
*   **`INSERT INTO`** (DML) được sử dụng để thêm một hoặc nhiều hàng dữ liệu mới vào bảng, với chú ý đến thứ tự cột và giá trị.
*   **`SELECT`** (DML) được sử dụng để truy xuất dữ liệu từ bảng.
    *   `SELECT *` truy xuất tất cả các cột (thường dùng để khám phá).
    *   `SELECT cot1, cot2` truy xuất các cột cụ thể (khuyến nghị cho mã sản xuất).

Trong các phần tiếp theo, chúng ta sẽ đi sâu hơn vào các lệnh DML để lọc, sắp xếp và kết hợp dữ liệu, cũng như khám phá các tính năng mạnh mẽ hơn của PostgreSQL.

<!-- REVIEWED_BY_AGENT -->
