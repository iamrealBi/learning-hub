# Phần 5: Thiết Kế Cơ Sở Dữ Liệu và Khóa Ngoại

Chào mừng bạn đến với Phần 5 của khóa học, nơi chúng ta sẽ đi sâu vào một trong những khía cạnh nền tảng và quan trọng nhất của phát triển ứng dụng: thiết kế cơ sở dữ liệu quan hệ. Cho đến nay, trọng tâm của chúng ta là các câu lệnh SQL cơ bản trên các bảng dữ liệu độc lập. Tuy nhiên, trong môi trường phát triển phần mềm thực tế, các ứng dụng hiếm khi chỉ tương tác với một bảng dữ liệu duy nhất. Một cơ sở dữ liệu mạnh mẽ, hiệu quả thường bao gồm nhiều bảng liên kết chặt chẽ với nhau, phản ánh cấu trúc dữ liệu phức tạp của ứng dụng.

Mục tiêu chính của chương này là trang bị cho bạn kiến thức và kỹ năng để thiết kế các lược đồ cơ sở dữ liệu phức tạp hơn, tập trung vào việc hiểu sâu sắc và triển khai các mối quan hệ giữa các bảng. Chúng ta sẽ khám phá cách xác định các thực thể dữ liệu và mối quan hệ logic giữa chúng, sau đó tìm hiểu các khái niệm cốt lõi như Khóa Chính (Primary Key) và Khóa Ngoại (Foreign Key). Cuối cùng, chúng ta sẽ áp dụng những kiến thức này để xây dựng một cơ sở dữ liệu đa bảng hoàn chỉnh cho một ứng dụng chia sẻ ảnh thực tế bằng PostgreSQL, đảm bảo tính nhất quán, toàn vẹn và hiệu quả của dữ liệu. Chúng ta cũng sẽ khám phá cách các công cụ AI hiện đại như Antigravity IDE có thể hỗ trợ và tăng tốc đáng kể quy trình thiết kế này thông qua tư duy Vibe Coding.

## 1. Tại Sao Cần Thiết Kế Cơ Sở Dữ Liệu Đa Bảng?

Trong các bài học trước, chúng ta đã làm quen với việc thao tác dữ liệu trên các bảng đơn lẻ, ví dụ như bảng `cities` hoặc `phones`. Mặc dù cách tiếp cận này đơn giản và dễ hiểu khi mới bắt đầu, nó không phản ánh chính xác cách các ứng dụng thực tế hoạt động và nhanh chóng bộc lộ những hạn chế nghiêm trọng.

Một ứng dụng thực tế, dù lớn hay nhỏ, thường phải quản lý nhiều loại dữ liệu khác nhau có mối liên hệ chặt chẽ với nhau. Ví dụ, một ứng dụng chia sẻ ảnh không chỉ lưu trữ ảnh mà còn lưu trữ thông tin người dùng, bình luận của người dùng về ảnh, và số lượt thích cho mỗi ảnh. Nếu cố gắng lưu trữ tất cả thông tin này trong một bảng duy nhất, chúng ta sẽ gặp phải nhiều vấn đề nghiêm trọng:

*   **Dư thừa dữ liệu (Data Redundancy):** Thông tin người dùng (ví dụ: tên đăng nhập, email, tiểu sử) sẽ phải lặp lại cho mỗi bức ảnh, mỗi bình luận, mỗi lượt thích mà họ tạo ra. Điều này làm tăng đáng kể kích thước cơ sở dữ liệu, lãng phí không gian lưu trữ và gây khó khăn trong việc quản lý.
    *   *Ví dụ:* Nếu người dùng "Alice" đăng 100 bức ảnh, thông tin của Alice sẽ xuất hiện 100 lần trong cùng một bảng.
*   **Không nhất quán dữ liệu (Data Inconsistency):** Nếu thông tin người dùng thay đổi (ví dụ: Alice đổi tên hiển thị), chúng ta sẽ phải cập nhật ở nhiều nơi khác nhau trong bảng khổng lồ đó. Nguy cơ bỏ sót một bản ghi và dẫn đến dữ liệu không đồng nhất là rất cao, làm hỏng tính toàn vẹn của dữ liệu.
    *   *Ví dụ:* Alice đổi tên từ "Alice" thành "Alicia". Nếu không cập nhật tất cả 100 bản ghi ảnh của cô ấy, một số ảnh vẫn sẽ hiển thị tên cũ, gây nhầm lẫn.
*   **Khó khăn trong truy vấn và quản lý:** Việc tìm kiếm thông tin trở nên phức tạp và kém hiệu quả hơn. Ví dụ, để tìm tất cả bình luận của một người dùng cụ thể, chúng ta phải quét qua một bảng rất lớn chứa cả ảnh và lượt thích, làm chậm hiệu suất truy vấn.
*   **Hạn chế mở rộng (Scalability Issues):** Khi ứng dụng phát triển và thêm các tính năng mới, việc thêm cột vào một bảng khổng lồ sẽ trở nên khó khăn, phức tạp và tốn kém tài nguyên. Các thay đổi schema trên bảng lớn có thể gây gián đoạn dịch vụ.

Để giải quyết triệt để những vấn đề này, chúng ta cần thiết kế cơ sở dữ liệu với nhiều bảng, mỗi bảng tập trung vào một loại thực thể dữ liệu cụ thể (ví dụ: `users` cho người dùng, `photos` cho ảnh, `comments` cho bình luận). Sau đó, chúng ta sẽ thiết lập các mối quan hệ logic giữa các bảng này để liên kết dữ liệu một cách hiệu quả và đảm bảo tính toàn vẹn. Cách tiếp cận này giúp tối ưu hóa lưu trữ, tăng cường tính nhất quán dữ liệu và đơn giản hóa việc quản lý cũng như mở rộng ứng dụng trong tương lai.

## 2. Quy Trình Thiết Kế Cơ Sở Dữ Liệu Hiệu Quả

Việc thiết kế cơ sở dữ liệu là một kỹ năng quan trọng, đòi hỏi sự kết hợp giữa tư duy phân tích và kinh nghiệm thực tế. Dưới đây là các bước và mẹo để bạn có thể tiếp cận quá trình này một cách hiệu quả, đặc biệt là khi tận dụng sức mạnh của các công cụ AI hiện đại như Antigravity IDE.

### 2.1. Xác Định Thực Thể và Mối Quan Hệ (Entity-Relationship Modeling)

Khi bắt đầu thiết kế cơ sở dữ liệu cho một ứng dụng, hãy tự hỏi: "Ứng dụng này quản lý những loại tài nguyên hay thông tin nào?" Đây là bước đầu tiên trong quá trình mô hình hóa thực thể-quan hệ (Entity-Relationship Modeling - ERM), nơi chúng ta biến các khái niệm kinh doanh thành cấu trúc dữ liệu.

*   **Bước 1: Liệt kê các tài nguyên/thực thể chính.** Hãy cố gắng xác định các danh từ chính trong mô tả ứng dụng của bạn. Mỗi danh từ này thường tương ứng với một bảng trong cơ sở dữ liệu.
    *   *Ví dụ trong ứng dụng chia sẻ ảnh:* Người dùng, Ảnh, Bình luận, Lượt thích.
*   **Bước 2: Xác định các thuộc tính (attributes) cho mỗi thực thể.** Mỗi thực thể sẽ có các đặc điểm riêng. Các đặc điểm này sẽ trở thành các cột trong bảng.
    *   *Ví dụ:* `Người dùng` có `tên đăng nhập`, `email`, `mật khẩu`. `Ảnh` có `URL`, `tiêu đề`, `mô tả`.
*   **Bước 3: Xác định mối quan hệ hoặc quyền sở hữu giữa các thực thể.** Đây là bước quan trọng nhất để liên kết các bảng. Hãy nghĩ về cách các thực thể tương tác với nhau.
    *   *Ví dụ:* Một `Người dùng` `có nhiều` `Ảnh`. Một `Ảnh` `có nhiều` `Bình luận`.

### 2.2. Tối Ưu Hóa Quy Trình Thiết Kế với Antigravity IDE và Vibe Coding

Trong môi trường phát triển hiện đại với các công cụ Agentic AI như Antigravity IDE, quy trình thiết kế cơ sở dữ liệu không còn là một chuỗi bước tuyến tính cứng nhắc mà trở thành một chuỗi lặp lại linh hoạt, được hỗ trợ bởi khả năng phân tích và tổng hợp của AI. Tư duy Vibe Coding, được áp dụng thông qua Antigravity, cho phép bạn nhanh chóng phác thảo, kiểm tra và tinh chỉnh thiết kế của mình.

*   **Phân tích yêu cầu và giao diện người dùng (UI/UX) với Antigravity:**
    Hãy tưởng tượng bạn đang sử dụng Antigravity IDE và có các bản mockup giao diện người dùng (UI) hoặc mô tả tính năng của ứng dụng chia sẻ ảnh. Thay vì tự mình phân tích thủ công, bạn có thể "vibe" ý tưởng của mình bằng cách đưa các yêu cầu này trực tiếp cho Antigravity.

    *   **Hướng dẫn Antigravity:** Bạn có thể prompt Antigravity bằng các mô tả ngôn ngữ tự nhiên như:
        *   "Phân tích bản mockup này và liệt kê tất cả các loại dữ liệu chính mà ứng dụng cần quản lý."
        *   "Dựa trên các tính năng 'đăng ảnh', 'bình luận', 'thích', hãy đề xuất các thực thể CSDL cơ bản."
    *   **Cơ chế ngầm của Antigravity:** Antigravity, với khả năng gọi subagent trình duyệt, có thể "xem xét" các mockup trực tuyến hoặc phân tích các tài liệu yêu cầu. Với khả năng lập kế hoạch tự động, nó sẽ tự động xác định các danh từ chính và mối quan hệ tiềm năng, đưa ra các gợi ý ban đầu về cấu trúc bảng và các thuộc tính.

*   **Khám phá mẫu thiết kế và kiến trúc đã được kiểm chứng:**
    Nhiều ứng dụng web có các tính năng chung như xác thực người dùng, hệ thống thích, bình luận, v.v. Đối với những tính năng này, bạn không cần phải "phát minh lại bánh xe".

    *   **Hướng dẫn Antigravity:** Thay vì tự tìm kiếm "SQL schema for like system" trên Google, bạn có thể trực tiếp yêu cầu Antigravity đề xuất các lược đồ CSDL đã được kiểm chứng cho các tính năng phổ biến.
    *   **Cơ chế ngầm của Antigravity:** Antigravity có thể truy cập kiến thức rộng lớn của nó, và nếu cần, thực hiện "nghiên cứu" bằng cách gọi các subagent trình duyệt để tìm kiếm các mẫu thiết kế CSDL phổ biến và tổng hợp thông tin, đưa ra các gợi ý có cấu trúc, kèm theo giải thích ưu nhược điểm. Điều này giúp bạn nhanh chóng có được một nền tảng vững chắc cho thiết kế của mình.

*   **Lập kế hoạch và tạo schema tự động:**
    Với khả năng lập kế hoạch và chạy script ngầm, Antigravity có thể giúp bạn nhanh chóng phác thảo và tạo ra các câu lệnh `CREATE TABLE` ban đầu.

    *   **Hướng dẫn Antigravity:** Bạn có thể "vibe" ý tưởng của mình về các bảng và mối quan hệ (ví dụ: "Tôi muốn bảng `users` có `username` và `email`, bảng `photos` có `url` và thuộc về một user"), sau đó để Antigravity tự động chuyển hóa chúng thành mã SQL PostgreSQL chuẩn, kèm theo các ràng buộc khóa chính, khóa ngoại cần thiết.
    *   **Cơ chế ngầm của Antigravity:** Antigravity sẽ lập kế hoạch các bước tạo bảng, tự động viết các câu lệnh `CREATE TABLE` với `SERIAL PRIMARY KEY` và `REFERENCES` cùng các ràng buộc `NOT NULL`, thậm chí cả các hành động `ON DELETE`/`ON UPDATE` mà chúng ta sẽ tìm hiểu sau. Nó có thể chạy các script này trong môi trường sandbox hoặc kết nối trực tiếp với CSDL phát triển của bạn.

*   **Phản hồi tức thì và tinh chỉnh (Iterative Refinement):**
    Quá trình thiết kế CSDL là lặp đi lặp lại. Sau khi Antigravity tạo schema, bạn có thể yêu cầu nó chạy các lệnh `INSERT` mẫu và `SELECT` để kiểm tra tính đúng đắn của thiết kế và các mối quan hệ.

    *   **Hướng dẫn Antigravity:** "Chèn dữ liệu mẫu vào các bảng đã tạo và thực hiện một số truy vấn để kiểm tra xem các mối quan hệ có hoạt động như mong đợi không." "Nếu tôi xóa một người dùng, điều gì sẽ xảy ra với các ảnh của họ?"
    *   **Cơ chế ngầm của Antigravity:** Antigravity có thể tự động tạo và chạy các câu lệnh DML (Data Manipulation Language) để kiểm tra. Nếu có vấn đề về tính toàn vẹn dữ liệu (ví dụ: khóa ngoại không hoạt động đúng) hoặc hiệu suất (truy vấn chậm), Antigravity có thể giúp bạn chẩn đoán và đề xuất các chỉnh sửa, biến quá trình tinh chỉnh thành một vòng lặp phản hồi nhanh chóng và hiệu quả.

Việc áp dụng Vibe Coding với Antigravity IDE không chỉ giúp tăng tốc quá trình thiết kế mà còn giảm thiểu lỗi, cho phép bạn tập trung vào logic nghiệp vụ thay vì các chi tiết cú pháp SQL.

### 2.3. Ví Dụ Thực Tế: Phân Tích Ứng Dụng Chia Sẻ Ảnh (Instagram)

Hãy cùng phân tích một ứng dụng chia sẻ ảnh quen thuộc như Instagram để hiểu cách xác định các thực thể và mối quan hệ từ góc độ người dùng, điều này cũng tương tự như cách Antigravity sẽ phân tích các yêu cầu của bạn.

**Trang Hồ sơ Người dùng:**
Khi xem trang hồ sơ của một người dùng (ví dụ: tài khoản Instagram chính thức), chúng ta có thể thấy:

*   **Tên người dùng (Username):** `instagram`
*   **Tiểu sử (Bio):** Một đoạn văn bản mô tả về người dùng.
*   **Tính năng theo dõi (Follow/Followers):** Người dùng có thể theo dõi người khác và bị người khác theo dõi. Điều này gợi ý một mối quan hệ giữa các `Người dùng` với nhau.
*   **Danh sách ảnh:** Có một danh sách các bức ảnh mà người dùng này đã đăng. Điều này chỉ ra rằng có một tài nguyên `Ảnh` và một mối quan hệ `Người dùng SỞ HỮU Ảnh`.

**Trang Bài đăng/Ảnh Đơn lẻ:**
Khi xem một bài đăng ảnh cụ thể, chúng ta có thể thấy:

*   **Bình luận (Comments):** Có một danh sách các bình luận dưới ảnh. Các bình luận này được đăng bởi những `Người dùng` cụ thể và dành riêng cho bức `Ảnh` này. Điều này gợi ý mối quan hệ `Bình luận THUỘC VỀ Người dùng` và `Bình luận THUỘC VỀ Ảnh`.
*   **Lượt thích (Likes):** Người dùng có thể "thích" một bức ảnh. Điều này gợi ý một tài nguyên `Lượt thích` và mối quan hệ `Người dùng THÍCH Ảnh`.

Từ phân tích trên, chúng ta có thể xác định bốn thực thể chính cần được lưu trữ trong cơ sở dữ liệu của ứng dụng chia sẻ ảnh:

1.  **Người dùng (Users):** Lưu trữ thông tin về các tài khoản người dùng (tên đăng nhập, email, mật khẩu, v.v.).
2.  **Ảnh (Photos):** Lưu trữ thông tin về các bức ảnh được đăng (URL, mô tả, ngày đăng, v.v.).
3.  **Bình luận (Comments):** Lưu trữ nội dung bình luận của người dùng về ảnh.
4.  **Lượt thích (Likes):** Lưu trữ thông tin về những lượt thích của người dùng cho ảnh.

Và các mối quan hệ sơ bộ giữa chúng:

*   Một `Người dùng` có thể tạo và sở hữu nhiều `Ảnh`.
*   Một `Người dùng` có thể tạo và sở hữu nhiều `Bình luận`.
*   Một `Người dùng` có thể tạo và sở hữu nhiều `Lượt thích`.
*   Một `Ảnh` có thể có nhiều `Bình luận`.
*   Một `Ảnh` có thể có nhiều `Lượt thích`.
*   Nhiều `Người dùng` có thể `theo dõi` nhiều `Người dùng` khác (mối quan hệ tự tham chiếu).

Với những xác định ban đầu này, chúng ta đã có một cái nhìn sơ bộ về cấu trúc cơ sở dữ liệu. Bước tiếp theo là hiểu rõ hơn về các loại mối quan hệ và cách chúng ta sẽ triển khai chúng trong cơ sở dữ liệu quan hệ.

## 3. Các Loại Mối Quan Hệ Trong Cơ Sở Dữ Liệu Quan Hệ

Trong thiết kế cơ sở dữ liệu quan hệ, các bảng được liên kết với nhau thông qua các mối quan hệ. Có ba loại mối quan hệ chính mà bạn cần hiểu để xây dựng một lược đồ CSDL hiệu quả: Một-Nhiều (One-to-Many), Một-Một (One-to-One) và Nhiều-Nhiều (Many-to-Many).

### 3.1. Mối Quan Hệ Một-Nhiều (One-to-Many) và Nhiều-Một (Many-to-One)

Đây là loại mối quan hệ phổ biến nhất và là nền tảng của hầu hết các cơ sở dữ liệu quan hệ. Thực chất, Một-Nhiều và Nhiều-Một là hai cách nhìn khác nhau về cùng một mối quan hệ, tùy thuộc vào hướng mà bạn đang xem xét.

*   **Mối quan hệ Một-Nhiều (One-to-Many):** Xảy ra khi một bản ghi trong bảng A có thể liên kết với **nhiều** bản ghi trong bảng B.
    *   **Ví dụ:** Một `Người dùng` có thể đăng `nhiều Ảnh`. (Người dùng `Instagram` có nhiều ảnh).
    *   **Dấu hiệu nhận biết:** Khi bạn có thể diễn đạt mối quan hệ bằng cụm từ "có nhiều" hoặc "sở hữu nhiều". (Ví dụ: "Một người dùng *có nhiều* ảnh.")

*   **Mối quan hệ Nhiều-Một (Many-to-One):** Xảy ra khi nhiều bản ghi trong bảng B liên kết với **một** bản ghi duy nhất trong bảng A. Đây là mặt ngược lại của mối quan hệ Một-Nhiều.
    *   **Ví dụ:** Nhiều `Ảnh` thuộc về `một Người dùng` duy nhất. (Nhiều ảnh thuộc về người dùng `Instagram`).
    *   **Dấu hiệu nhận biết:** Khi bạn có thể diễn đạt mối quan hệ bằng cụm từ "thuộc về một" hoặc "được tạo bởi một". (Ví dụ: "Nhiều ảnh *thuộc về một* người dùng.")

**Ví dụ khác:**

*   **Ảnh và Bình luận:**
    *   Một `Ảnh` *có nhiều* `Bình luận` (Một-Nhiều).
    *   Nhiều `Bình luận` *thuộc về một* `Ảnh` (Nhiều-Một).
*   **Thuyền và Thủy thủ đoàn:**
    *   Một `Thuyền` *có nhiều* `Thành viên thủy thủ đoàn`.
    *   Một `Thành viên thủy thủ đoàn` (thường) *chỉ thuộc về một* `Thuyền` tại một thời điểm.
*   **Khoa và Sinh viên:**
    *   Một `Khoa` *có nhiều* `Sinh viên`.
    *   Một `Sinh viên` (thường) *chỉ thuộc về một* `Khoa` tại một thời điểm.

> [!NOTE]
> Mối quan hệ Một-Nhiều và Nhiều-Một là cùng một mối quan hệ nhìn từ hai phía khác nhau. Việc xác định phía nào là "một" và phía nào là "nhiều" rất quan trọng để triển khai trong cơ sở dữ liệu, vì khóa ngoại sẽ luôn được đặt ở phía "nhiều".

### 3.2. Mối Quan Hệ Một-Một (One-to-One)

Mối quan hệ Một-Một (One-to-One) xảy ra khi một bản ghi trong bảng A chỉ liên kết với duy nhất một bản ghi trong bảng B, và ngược lại, một bản ghi trong bảng B cũng chỉ liên kết với duy nhất một bản ghi trong bảng A. Đây là loại mối quan hệ ít phổ biến hơn nhưng có những trường hợp sử dụng cụ thể.

**Ví dụ:**

*   **Thuyền trưởng và Thuyền:** Mỗi `Thuyền` có một `Thuyền trưởng` và mỗi `Thuyền trưởng` chỉ chỉ huy một `Thuyền` tại một thời điểm.
*   **Công ty và CEO:** Một `Công ty` có một `CEO` và một `CEO` chỉ là CEO của một `Công ty` duy nhất tại một thời điểm.
*   **Quốc gia và Thủ đô:** Một `Quốc gia` có một `Thủ đô` và một `Thủ đô` chỉ là thủ đô của một `Quốc gia`.
*   **Người và Giấy phép lái xe:** Một `Người` có một `Giấy phép lái xe` và một `Giấy phép lái xe` thuộc về một `Người` duy nhất.

> [!TIP]
> Mối quan hệ Một-Một thường được sử dụng khi bạn muốn tách dữ liệu của một thực thể thành hai bảng vì các lý do sau:
> *   **Hiệu suất:** Tách các cột ít được truy cập hoặc rất lớn vào một bảng riêng biệt để giữ cho bảng chính nhỏ gọn và truy vấn nhanh hơn.
> *   **Bảo mật:** Lưu trữ thông tin nhạy cảm (ví dụ: thông tin cá nhân mở rộng) trong một bảng riêng biệt với các quyền truy cập hạn chế hơn.
> *   **Quản lý dữ liệu tùy chọn:** Tách các thuộc tính không thường xuyên được sử dụng hoặc chỉ áp dụng cho một tập hợp con các bản ghi.

### 3.3. Mối Quan Hệ Nhiều-Nhiều (Many-to-Many)

Mối quan hệ Nhiều-Nhiều (Many-to-Many) là mối quan hệ phức tạp nhất trong ba loại. Nó xảy ra khi một bản ghi trong bảng A có thể liên kết với **nhiều** bản ghi trong bảng B, và ngược lại, một bản ghi trong bảng B cũng có thể liên kết với **nhiều** bản ghi trong bảng A.

**Ví dụ:**

*   **Học sinh và Lớp học:** Một `Học sinh` có thể học `nhiều Lớp học`, và một `Lớp học` có thể có `nhiều Học sinh`.
*   **Nhiệm vụ và Kỹ sư:** Một `Nhiệm vụ` có thể được giao cho `nhiều Kỹ sư`, và một `Kỹ sư` có thể làm `nhiều Nhiệm vụ`.
*   **Phim và Diễn viên:** Một `Phim` có thể có `nhiều Diễn viên` đóng, và một `Diễn viên` có thể đóng trong `nhiều Phim`.
*   **Cuộc gọi hội nghị và Nhân viên:** (Trong một khoảng thời gian) Một `Cuộc gọi hội nghị` có thể có `nhiều Nhân viên` tham gia, và một `Nhân viên` có thể tham gia `nhiều Cuộc gọi hội nghị`.

> [!IMPORTANT]
> Mối quan hệ Nhiều-Nhiều không thể được triển khai trực tiếp bằng cách thêm một cột khóa ngoại vào một trong hai bảng gốc. Thay vào đó, nó luôn được giải quyết bằng cách tạo một **bảng trung gian** (còn gọi là bảng liên kết, bảng nối, hoặc bảng kết nối). Bảng trung gian này chứa ít nhất hai khóa ngoại, mỗi khóa ngoại trỏ đến khóa chính của một trong hai bảng gốc, và thường có một khóa chính tổng hợp (composite primary key) từ hai khóa ngoại đó. Bảng trung gian cũng có thể chứa các thuộc tính bổ sung mô tả chính mối quan hệ đó (ví dụ: ngày học, vai trò diễn viên).

Đối với cơ sở dữ liệu ứng dụng chia sẻ ảnh của chúng ta, các mối quan hệ chủ yếu sẽ là Một-Nhiều và Nhiều-Một, cùng với một mối quan hệ Nhiều-Nhiều cho tính năng "Lượt thích".

## 4. Khóa Chính (Primary Key) và Khóa Ngoại (Foreign Key): Nền Tảng Liên Kết Dữ Liệu

Để thiết lập và quản lý các mối quan hệ giữa các bảng trong cơ sở dữ liệu quan hệ, chúng ta sử dụng hai khái niệm cốt lõi: Khóa Chính (Primary Key) và Khóa Ngoại (Foreign Key). Chúng là những viên gạch xây dựng nên cấu trúc và tính toàn vẹn của dữ liệu.

### 4.1. Khóa Chính (Primary Key - PK)

Khóa Chính (Primary Key) là một cột hoặc tập hợp các cột trong một bảng được sử dụng để định danh duy nhất mỗi hàng (bản ghi) trong bảng đó. Nó đóng vai trò là "địa chỉ" duy nhất cho mỗi bản ghi.

**Đặc điểm cốt lõi của Khóa Chính:**

*   **Duy nhất (Unique):** Mỗi giá trị trong cột khóa chính phải là duy nhất trong toàn bộ bảng. Không có hai hàng nào có thể có cùng một giá trị khóa chính.
*   **Không rỗng (Not Null):** Giá trị của khóa chính không được phép rỗng (NULL). Một bản ghi phải luôn có một định danh.
*   **Không thay đổi (Immutable - Lý tưởng):** Giá trị của khóa chính lý tưởng là không thay đổi trong suốt vòng đời của bản ghi. Việc thay đổi khóa chính có thể gây ra sự không nhất quán dữ liệu ở các bảng khác có tham chiếu đến nó và làm giảm hiệu suất.
*   **Thường là số nguyên tự tăng (Surrogate Key):** Trong hầu hết các trường hợp, khóa chính là một số nguyên tự tăng hoặc một chuỗi ký tự duy nhất toàn cầu (UUID). Đây được gọi là khóa thay thế (surrogate key) vì nó không có ý nghĩa nghiệp vụ mà chỉ dùng để định danh duy nhất. Trái ngược với khóa tự nhiên (natural key) có ý nghĩa nghiệp vụ (ví dụ: số chứng minh thư, mã sản phẩm). Khóa thay thế thường được ưu tiên vì tính đơn giản, không thay đổi và hiệu quả hơn.
*   **Quy ước đặt tên:** Thường được đặt tên là `id` hoặc `[tên_bảng]_id` (ví dụ: `user_id`, `photo_id`). Trong khóa học này, chúng ta sẽ sử dụng `id` cho khóa chính.

**Ví dụ:**
Trong bảng `users`, cột `id` sẽ là khóa chính. Mỗi người dùng sẽ có một `id` duy nhất.
Nếu chúng ta truy vấn "người dùng có ID là 1", chúng ta sẽ luôn nhận được chính xác một người dùng cụ thể, bất kể dữ liệu trong bảng có thay đổi thứ tự hay không.

**Cơ chế tự động tạo Khóa Chính trong PostgreSQL (`SERIAL` và `BIGSERIAL`):**
PostgreSQL cung cấp kiểu dữ liệu `SERIAL` (hoặc `BIGSERIAL` cho các bảng lớn hơn có thể vượt quá giới hạn 2 tỷ bản ghi của `INTEGER`) để tự động tạo các giá trị số nguyên tăng dần cho khóa chính. Khi bạn khai báo một cột là `SERIAL PRIMARY KEY`:

1.  PostgreSQL sẽ tự động tạo một chuỗi số (sequence) ẩn và gán giá trị tiếp theo từ chuỗi đó mỗi khi một hàng mới được chèn vào mà không chỉ định giá trị cho cột `id`.
2.  Nó đảm bảo tính duy nhất và không rỗng của cột đó.
3.  Nó tạo ra một chỉ mục (index) trên cột đó một cách tự động, giúp tăng tốc độ truy vấn khi tìm kiếm bằng khóa chính, điều này rất quan trọng cho hiệu suất cơ sở dữ liệu.

> [!IMPORTANT]
> Luôn luôn gán một Khóa Chính cho mỗi bảng bạn tạo. Đây là một quy tắc vàng trong thiết kế cơ sở dữ liệu quan hệ, đảm bảo mỗi bản ghi có một định danh rõ ràng.

### 4.2. Khóa Ngoại (Foreign Key - FK)

Khóa Ngoại (Foreign Key) là một cột hoặc tập hợp các cột trong một bảng (gọi là bảng con hoặc bảng tham chiếu), thiết lập một liên kết giữa dữ liệu trong bảng đó với dữ liệu trong một bảng khác (gọi là bảng cha hoặc bảng được tham chiếu). Khóa ngoại tham chiếu đến khóa chính (hoặc một khóa duy nhất khác) của bảng cha.

**Mục đích cốt lõi của Khóa Ngoại:**

*   **Thiết lập mối quan hệ:** Là cơ chế để liên kết các bản ghi giữa hai hoặc nhiều bảng, biến các bảng độc lập thành một hệ thống dữ liệu có tổ chức.
*   **Đảm bảo tính toàn vẹn tham chiếu (Referential Integrity):** Khóa ngoại đảm bảo rằng các giá trị trong cột khóa ngoại phải khớp với một giá trị tồn tại trong cột khóa chính của bảng được tham chiếu. Điều này ngăn chặn việc tạo ra các liên kết đến dữ liệu không tồn tại (còn gọi là "dữ liệu mồ côi" - orphan data), duy trì sự nhất quán và tin cậy của cơ sở dữ liệu.

**Quy tắc xác định vị trí Khóa Ngoại:**

*   Trong mối quan hệ Một-Nhiều (hoặc Nhiều-Một), cột khóa ngoại luôn được đặt ở **phía "Nhiều"** của mối quan hệ.
    *   *Ví dụ:* Một `Người dùng` có `Nhiều Ảnh`. Bảng `photos` (phía "Nhiều") sẽ có khóa ngoại `user_id` trỏ đến `users.id` (phía "Một").
    *   *Ví dụ:* Một `Ảnh` có `Nhiều Bình luận`. Bảng `comments` (phía "Nhiều") sẽ có khóa ngoại `photo_id` trỏ đến `photos.id` (phía "Một").
    *   *Ví dụ:* Một `Người dùng` có `Nhiều Bình luận`. Bảng `comments` (phía "Nhiều") sẽ có khóa ngoại `user_id` trỏ đến `users.id` (phía "Một").

**Đặc điểm của Khóa Ngoại:**

*   **Không nhất thiết phải duy nhất:** Một giá trị khóa ngoại có thể xuất hiện nhiều lần trong cột khóa ngoại (ví dụ: nhiều ảnh có thể có cùng `user_id` vì chúng thuộc về cùng một người dùng).
*   **Có thể chứa giá trị NULL:** Tùy thuộc vào thiết kế, một khóa ngoại có thể được phép rỗng nếu mối quan hệ là tùy chọn (ví dụ: một bài viết có thể không có tác giả). Tuy nhiên, trong nhiều trường hợp, chúng ta sẽ muốn nó là `NOT NULL` để đảm bảo mỗi bản ghi "phía nhiều" luôn có một bản ghi "phía một" tương ứng.
*   **Có thể thay đổi:** Giá trị của khóa ngoại có thể thay đổi nếu mối quan hệ thay đổi (ví dụ: chuyển quyền sở hữu một bức ảnh từ người dùng này sang người dùng khác).
*   **Kiểu dữ liệu:** Kiểu dữ liệu của cột khóa ngoại phải tương thích với kiểu dữ liệu của cột khóa chính mà nó tham chiếu (ví dụ: nếu `users.id` là `SERIAL` (tương đương `INTEGER`), thì `photos.user_id` cũng phải là `INTEGER`).

**Quy ước đặt tên cho Khóa Ngoại:**
Thông thường, khóa ngoại được đặt tên theo quy ước `[tên_bảng_tham_chiếu]_id`.

*   *Ví dụ:* Trong bảng `photos`, khóa ngoại trỏ đến bảng `users` sẽ được gọi là `user_id`.
*   *Ví dụ:* Trong bảng `comments`, khóa ngoại trỏ đến bảng `photos` sẽ được gọi là `photo_id`.

> [!TIP]
> **Sự khác biệt chính giữa Khóa Chính và Khóa Ngoại:**
> *   **Khóa Chính:** Định danh *duy nhất* một hàng trong *chính bảng của nó*. Luôn là duy nhất và không rỗng.
> *   **Khóa Ngoại:** Thiết lập *liên kết* đến một hàng trong *bảng khác*. Không nhất thiết phải duy nhất, có thể rỗng (nếu cho phép), và có thể thay đổi.

### 4.3. Hành Động Ràng Buộc Khóa Ngoại (Foreign Key Constraint Actions)

Khi bạn định nghĩa một khóa ngoại, bạn cũng có thể chỉ định cách cơ sở dữ liệu sẽ phản ứng khi một bản ghi trong bảng cha bị xóa hoặc cập nhật mà các bản ghi trong bảng con đang tham chiếu đến nó. Các hành động này được gọi là `ON DELETE` và `ON UPDATE` actions.

**Các tùy chọn chính cho `ON DELETE` và `ON UPDATE`:**

1.  **`NO ACTION` (Mặc định trong PostgreSQL, nhưng thường được hiểu là `RESTRICT` trong các ngữ cảnh khác):**
    *   Ngăn chặn việc xóa hoặc cập nhật bản ghi trong bảng cha nếu có bất kỳ bản ghi nào trong bảng con đang tham chiếu đến nó. Hành động này được kiểm tra ở cuối câu lệnh.
    *   *Ví dụ:* Nếu bạn cố gắng xóa một người dùng mà vẫn còn ảnh của họ trong bảng `photos`, thao tác sẽ thất bại.

2.  **`RESTRICT`:**
    *   Tương tự như `NO ACTION`, ngăn chặn hành động xóa hoặc cập nhật. Sự khác biệt nhỏ là `RESTRICT` được kiểm tra ngay lập tức tại thời điểm xảy ra thao tác. Trong PostgreSQL, hai tùy chọn này thường có hành vi tương tự.
    *   *Ví dụ:* Nếu bạn cố gắng xóa một người dùng có ảnh, `RESTRICT` sẽ ngăn chặn ngay lập tức.

3.  **`CASCADE`:**
    *   Khi một bản ghi trong bảng cha bị xóa hoặc cập nhật, **tất cả các bản ghi liên quan** trong bảng con cũng sẽ bị xóa hoặc cập nhật tương ứng. Đây là một hành động mạnh mẽ và cần được sử dụng cẩn thận.
    *   *Ví dụ:* Nếu một người dùng bị xóa, tất cả các bức ảnh do người dùng đó đăng và tất cả các bình luận/lượt thích của người dùng đó cũng sẽ tự động bị xóa.
    *   *Ví dụ:* Nếu `id` của một người dùng thay đổi (điều này không nên xảy ra với khóa chính surrogate, nhưng có thể với các khóa tự nhiên), `user_id` trong bảng `photos` cũng sẽ tự động cập nhật.

4.  **`SET NULL`:**
    *   Khi một bản ghi trong bảng cha bị xóa hoặc cập nhật, các giá trị khóa ngoại tương ứng trong bảng con sẽ được đặt thành `NULL`. Điều này chỉ hoạt động nếu cột khóa ngoại được phép chứa giá trị `NULL`.
    *   *Ví dụ:* Nếu một người dùng bị xóa, cột `user_id` trong các bản ghi ảnh của họ sẽ được đặt thành `NULL`, nghĩa là ảnh đó không còn chủ sở hữu rõ ràng nữa.

5.  **`SET DEFAULT`:**
    *   Khi một bản ghi trong bảng cha bị xóa hoặc cập nhật, các giá trị khóa ngoại tương ứng trong bảng con sẽ được đặt thành giá trị mặc định (DEFAULT) đã được định nghĩa cho cột đó. Điều này chỉ hoạt động nếu cột khóa ngoại có giá trị mặc định và được phép chứa giá trị `NULL` (nếu giá trị mặc định là `NULL`).
    *   *Ví dụ:* Nếu một người dùng bị xóa, cột `user_id` trong ảnh của họ có thể được đặt thành một `id` của người dùng mặc định "Anonymous".

> [!WARNING]
> Việc sử dụng `CASCADE` cần hết sức thận trọng, đặc biệt trong môi trường sản phẩm. Một thao tác xóa hoặc cập nhật sai trên bảng cha có thể dẫn đến mất mát dữ liệu lớn ở các bảng con. Luôn kiểm tra kỹ lưỡng và hiểu rõ tác động của nó.

## 5. Triển Khai Thực Tế: Xây Dựng Cơ Sở Dữ Liệu Chia Sẻ Ảnh với PostgreSQL

Bây giờ chúng ta sẽ áp dụng các kiến thức về khóa chính, khóa ngoại và các hành động ràng buộc để xây dựng cơ sở dữ liệu cho ứng dụng chia sẻ ảnh của chúng ta trong PostgreSQL.

> [!NOTE]
> Để thực hành, bạn có thể sử dụng `psql` trong terminal hoặc một công cụ quản lý cơ sở dữ liệu đồ họa như DBeaver hoặc pgAdmin. Đảm bảo rằng bạn đang kết nối với một cơ sở dữ liệu PostgreSQL. Nếu bạn đang sử dụng Antigravity IDE, bạn có thể trực tiếp yêu cầu nó thực thi các câu lệnh SQL này trong môi trường làm việc của bạn.

### 5.1. Bảng `users`: Định Danh Người Dùng

Chúng ta sẽ bắt đầu bằng cách tạo bảng `users` với một cột `id` làm khóa chính tự động tăng và một cột `username` phải là duy nhất và không được để trống.

```sql
-- CẢNH BÁO QUAN TRỌNG: Lệnh DROP TABLE ... CASCADE; sẽ xóa bảng và TẤT CẢ các đối tượng phụ thuộc (ví dụ: các bảng có khóa ngoại trỏ đến nó).
-- CHỈ NÊN CHẠY LỆNH NÀY KHI THỰC HÀNH TRÊN MÔI TRƯỜNG PHÁT TRIỂN HOẶC TEST.
-- TUYỆT ĐỐI KHÔNG CHẠY TRÊN CƠ SỞ DỮ LIỆU THẬT MÀ KHÔNG SAO LƯU DỮ LIỆU CẨN THẬN.
DROP TABLE IF EXISTS users CASCADE; 

-- Tạo bảng users
CREATE TABLE users (
    id SERIAL PRIMARY KEY, -- Cột ID là Khóa Chính tự động tăng (INTEGER), đảm bảo duy nhất và không rỗng.
    username VARCHAR(50) NOT NULL UNIQUE, -- Cột tên người dùng, không được để trống và phải duy nhất.
    email VARCHAR(100) UNIQUE, -- Cột email, phải duy nhất (có thể rỗng nếu người dùng không cung cấp).
    bio TEXT, -- Tiểu sử người dùng (có thể rỗng).
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP -- Thời gian tạo tài khoản, mặc định là thời điểm hiện tại.
);
```

**Giải thích:**

*   `id SERIAL PRIMARY KEY`: Cột `id` sẽ là khóa chính, sử dụng kiểu `SERIAL` của PostgreSQL để tự động tạo ID tăng dần. Đây là một khóa thay thế (surrogate key) lý tưởng.
*   `username VARCHAR(50) NOT NULL UNIQUE`: `username` là tên hiển thị, không được phép rỗng và phải là duy nhất giữa các người dùng.
*   `email VARCHAR(100) UNIQUE`: `email` cũng phải duy nhất, nhưng ở đây chúng ta cho phép nó rỗng (không có `NOT NULL`).
*   `bio TEXT`: Kiểu `TEXT` cho phép lưu trữ chuỗi ký tự dài mà không giới hạn độ dài cụ thể.
*   `created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP`: Lưu trữ thời điểm tạo tài khoản, bao gồm thông tin múi giờ, và mặc định là thời gian hiện tại.

Bây giờ, hãy chèn một vài người dùng vào bảng `users`. Lưu ý rằng chúng ta không cần cung cấp giá trị cho cột `id` và `created_at` vì chúng sẽ được tự động tạo.

```sql
-- Chèn dữ liệu vào bảng users
INSERT INTO users (username, email, bio) VALUES
('Monahan93', 'mona@example.com', 'Nhiếp ảnh gia chuyên nghiệp.'),
('Pfeiffer_Battles', 'pbattles@example.com', NULL),
('TheOther93', 'other93@example.com', 'Yêu thích du lịch và khám phá.'),
('User99', 'user99@example.com', 'Người dùng mới.');

-- Kiểm tra dữ liệu trong bảng users
SELECT * FROM users;
```

**Kết quả dự kiến:**
```
 id |    username    |       email        |            bio             |          created_at          
----+------------------+--------------------+----------------------------+------------------------------
  1 | Monahan93        | mona@example.com   | Nhiếp ảnh gia chuyên nghiệp.| 2023-10-27 10:00:00+07
  2 | Pfeiffer_Battles | pbattles@example.com |                            | 2023-10-27 10:00:00+07
  3 | TheOther93       | other93@example.com| Yêu thích du lịch và khám phá.| 2023-10-27 10:00:00+07
  4 | User99           | user99@example.com | Người dùng mới.            | 2023-10-27 10:00:00+07
(4 rows)
```
Như bạn thấy, PostgreSQL đã tự động gán các `id` duy nhất và tăng dần cho mỗi người dùng được chèn.

### 5.2. Bảng `photos`: Quản Lý Ảnh và Liên Kết Chủ Sở Hữu

Tiếp theo, chúng ta sẽ tạo bảng `photos`. Bảng này sẽ có khóa chính `id` của riêng nó, một cột `url` cho ảnh, và một cột `user_id` đóng vai trò là khóa ngoại để liên kết với bảng `users`. Chúng ta sẽ sử dụng `ON DELETE CASCADE` để tự động xóa ảnh khi người dùng sở hữu bị xóa.

```sql
-- Xóa bảng photos nếu tồn tại
DROP TABLE IF EXISTS photos CASCADE;

-- Tạo bảng photos
CREATE TABLE photos (
    id SERIAL PRIMARY KEY, -- Khóa Chính tự động tăng cho bảng photos
    url VARCHAR(200) NOT NULL, -- URL của ảnh, không được để trống
    caption VARCHAR(255), -- Chú thích ảnh (có thể rỗng)
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE, -- Khóa Ngoại: liên kết với users.id.
                                                                    -- NOT NULL: mỗi ảnh phải có chủ sở hữu.
                                                                    -- ON DELETE CASCADE: nếu người dùng bị xóa, tất cả ảnh của họ cũng bị xóa.
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP -- Thời gian đăng ảnh.
);
```

**Giải thích:**

*   `id SERIAL PRIMARY KEY`: Tương tự như bảng `users`, `id` là khóa chính tự động tăng của bảng `photos`.
*   `url VARCHAR(200) NOT NULL`: Cột lưu trữ URL của ảnh. `NOT NULL` đảm bảo mỗi ảnh phải có URL.
*   `caption VARCHAR(255)`: Chú thích cho ảnh, không bắt buộc.
*   `user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE`:
    *   `user_id INTEGER NOT NULL`: Khai báo một cột khóa ngoại kiểu số nguyên, không được phép rỗng, nghĩa là mỗi bức ảnh *phải* thuộc về một người dùng.
    *   `REFERENCES users(id)`: Thiết lập ràng buộc khóa ngoại, đảm bảo `user_id` phải tồn tại trong cột `id` của bảng `users`.
    *   `ON DELETE CASCADE`: Đây là một hành động ràng buộc quan trọng. Nếu một bản ghi người dùng trong bảng `users` bị xóa, tất cả các bản ghi ảnh trong bảng `photos` có `user_id` tương ứng cũng sẽ tự động bị xóa.

Hãy chèn một vài bức ảnh và liên kết chúng với các người dùng đã có:

```sql
-- Chèn dữ liệu vào bảng photos
INSERT INTO photos (url, caption, user_id) VALUES
('http://example.com/photo1.jpg', 'Bình minh trên biển', 4), -- Ảnh này của User99 (id=4)
('http://example.com/photo2.jpg', 'Mèo đáng yêu', 4), -- Ảnh này cũng của User99 (id=4)
('http://example.com/photo3.jpg', 'Phong cảnh núi', 1), -- Ảnh này của Monahan93 (id=1)
('http://example.com/photo4.jpg', 'Đồ ăn ngon', 2); -- Ảnh này của Pfeiffer_Battles (id=2)

-- Kiểm tra dữ liệu trong bảng photos
SELECT * FROM photos;
```

**Kết quả dự kiến:**
```
 id |          url          |    caption     | user_id |          created_at          
----+-----------------------+----------------+---------+------------------------------
  1 | http://example.com/photo1.jpg | Bình minh trên biển |       4 | 2023-10-27 10:00:00+07
  2 | http://example.com/photo2.jpg | Mèo đáng yêu       |       4 | 2023-10-27 10:00:00+07
  3 | http://example.com/photo3.jpg | Phong cảnh núi     |       1 | 2023-10-27 10:00:00+07
  4 | http://example.com/photo4.jpg | Đồ ăn ngon         |       2 | 2023-10-27 10:00:00+07
(4 rows)
```
Bây giờ, bạn có thể thấy rằng `user_id` trong bảng `photos` đã tạo một liên kết rõ ràng đến bảng `users`. Ví dụ, hai bức ảnh đầu tiên (id 1 và 2) đều thuộc về người dùng có `id` là 4 (`User99`).

### 5.3. Bảng `comments`: Ghi Lại Bình Luận

Để củng cố thêm, chúng ta sẽ tạo bảng `comments`. Bảng này sẽ có hai khóa ngoại: một trỏ đến `users` (ai đã bình luận) và một trỏ đến `photos` (bình luận cho ảnh nào). Chúng ta cũng sẽ sử dụng `ON DELETE CASCADE` cho cả hai khóa ngoại.

```sql
-- Xóa bảng comments nếu tồn tại
DROP TABLE IF EXISTS comments CASCADE;

-- Tạo bảng comments
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    contents VARCHAR(255) NOT NULL, -- Nội dung bình luận, không được để trống
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE, -- Khóa Ngoại: người dùng tạo bình luận.
                                                                     -- ON DELETE CASCADE: nếu người dùng bị xóa, bình luận của họ cũng bị xóa.
    photo_id INTEGER NOT NULL REFERENCES photos(id) ON DELETE CASCADE, -- Khóa Ngoại: ảnh được bình luận.
                                                                      -- ON DELETE CASCADE: nếu ảnh bị xóa, tất cả bình luận về ảnh đó cũng bị xóa.
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

**Giải thích:**

*   `user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE`: Liên kết bình luận với người dùng đã tạo ra nó. `NOT NULL` đảm bảo mỗi bình luận đều có một tác giả. `ON DELETE CASCADE` đảm bảo rằng nếu một người dùng bị xóa, tất cả các bình luận của họ cũng sẽ bị xóa.
*   `photo_id INTEGER NOT NULL REFERENCES photos(id) ON DELETE CASCADE`: Liên kết bình luận với bức ảnh mà nó thuộc về. `NOT NULL` đảm bảo mỗi bình luận đều thuộc về một ảnh. `ON DELETE CASCADE` đảm bảo rằng nếu một bức ảnh bị xóa, tất cả các bình luận liên quan đến bức ảnh đó cũng sẽ bị xóa.

Chèn một vài bình luận:

```sql
-- Chèn dữ liệu vào bảng comments
INSERT INTO comments (contents, user_id, photo_id) VALUES
('Tuyệt vời!', 4, 1), -- User99 bình luận cho Photo1
('Ảnh đẹp quá!', 2, 1), -- Pfeiffer_Battles bình luận cho Photo1
('Thích lắm!', 4, 3), -- User99 bình luận cho Photo3
('Chất lượng!', 1, 2); -- Monahan93 bình luận cho Photo2

-- Kiểm tra dữ liệu trong bảng comments
SELECT * FROM comments;
```

**Kết quả dự kiến:**
```
 id |   contents  | user_id | photo_id |          created_at          
----+-------------+---------+----------+------------------------------
  1 | Tuyệt vời!  |       4 |        1 | 2023-10-27 10:00:00+07
  2 | Ảnh đẹp quá!|       2 |        1 | 2023-10-27 10:00:00+07
  3 | Thích lắm!  |       4 |        3 | 2023-10-27 10:00:00+07
  4 | Chất lượng! |       1 |        2 | 2023-10-27 10:00:00+07
(4 rows)
```
Bây giờ, bảng `comments` đã liên kết thành công với cả bảng `users` và `photos` thông qua các khóa ngoại. Điều này cho phép chúng ta truy vấn để tìm ra ai đã bình luận gì và bình luận đó thuộc về bức ảnh nào.

### 5.4. Bảng `likes`: Triển Khai Mối Quan Hệ Nhiều-Nhiều

Để triển khai tính năng "Lượt thích", chúng ta cần một mối quan hệ Nhiều-Nhiều giữa `users` và `photos` (một người dùng có thể thích nhiều ảnh, và một ảnh có thể được thích bởi nhiều người dùng). Như đã thảo luận, mối quan hệ này được giải quyết bằng một bảng trung gian, mà chúng ta sẽ gọi là `likes`.

Bảng `likes` sẽ không cần một `id` tự tăng riêng biệt. Thay vào đó, nó sẽ sử dụng một khóa chính tổng hợp (composite primary key) bao gồm `user_id` và `photo_id`, đảm bảo rằng một người dùng chỉ có thể thích một bức ảnh một lần duy nhất.

```sql
-- Xóa bảng likes nếu tồn tại
DROP TABLE IF EXISTS likes CASCADE;

-- Tạo bảng likes (bảng trung gian cho mối quan hệ Nhiều-Nhiều)
CREATE TABLE likes (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE, -- Khóa Ngoại: người dùng thích.
                                                                     -- ON DELETE CASCADE: nếu người dùng bị xóa, tất cả lượt thích của họ cũng bị xóa.
    photo_id INTEGER NOT NULL REFERENCES photos(id) ON DELETE CASCADE, -- Khóa Ngoại: ảnh được thích.
                                                                      -- ON DELETE CASCADE: nếu ảnh bị xóa, tất cả lượt thích cho ảnh đó cũng bị xóa.
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, photo_id) -- Khóa Chính tổng hợp: đảm bảo một người dùng chỉ thích một ảnh một lần.
);
```

**Giải thích:**

*   `user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE`: Khóa ngoại trỏ đến bảng `users`.
*   `photo_id INTEGER NOT NULL REFERENCES photos(id) ON DELETE CASCADE`: Khóa ngoại trỏ đến bảng `photos`.
*   `PRIMARY KEY (user_id, photo_id)`: Đây là điểm mấu chốt. Chúng ta định nghĩa một khóa chính bao gồm cả `user_id` và `photo_id`. Điều này có nghĩa là sự kết hợp của `user_id` và `photo_id` phải là duy nhất. Ví dụ, người dùng có `id=1` chỉ có thể thích ảnh có `id=2` một lần duy nhất. Nếu họ cố gắng thích lại, cơ sở dữ liệu sẽ báo lỗi vi phạm khóa chính.
*   `ON DELETE CASCADE` cho cả hai khóa ngoại: Đảm bảo tính toàn vẹn. Nếu người dùng hoặc ảnh bị xóa, các lượt thích tương ứng cũng sẽ tự động bị xóa.

Chèn một vài lượt thích:

```sql
-- Chèn dữ liệu vào bảng likes
INSERT INTO likes (user_id, photo_id) VALUES
(4, 1), -- User99 thích Photo1
(2, 1), -- Pfeiffer_Battles thích Photo1
(1, 1), -- Monahan93 thích Photo1
(4, 3), -- User99 thích Photo3
(1, 2); -- Monahan93 thích Photo2

-- Kiểm tra dữ liệu trong bảng likes
SELECT * FROM likes;
```

**Kết quả dự kiến:**
```
 user_id | photo_id |          created_at          
---------+----------+------------------------------
       4 |        1 | 2023-10-27 10:00:00+07
       2 |        1 | 2023-10-27 10:00:00+07
       1 |        1 | 2023-10-27 10:00:00+07
       4 |        3 | 2023-10-27 10:00:00+07
       1 |        2 | 2023-10-27 10:00:00+07
(5 rows)
```
Bảng `likes` giờ đây đã thiết lập thành công mối quan hệ Nhiều-Nhiều giữa `users` và `photos`, cho phép chúng ta theo dõi ai đã thích ảnh nào một cách hiệu quả và với tính toàn vẹn dữ liệu được đảm bảo.

> [!NOTE]
> Việc sử dụng khóa ngoại không chỉ giúp tổ chức dữ liệu mà còn là nền tảng cho các truy vấn phức tạp hơn như `JOIN`, cho phép chúng ta kết hợp dữ liệu từ nhiều bảng để lấy thông tin đầy đủ về một thực thể. Chúng ta sẽ khám phá `JOIN` và các kỹ thuật truy vấn nâng cao khác trong các phần tiếp theo của khóa học.

## Tóm Tắt Chương

Chương này đã cung cấp một cái nhìn toàn diện về thiết kế cơ sở dữ liệu đa bảng, tập trung vào các khái niệm cốt lõi và triển khai thực tế trong PostgreSQL.

*   **Sự cần thiết của CSDL đa bảng:** Các ứng dụng thực tế yêu cầu CSDL đa bảng để tránh dư thừa dữ liệu, đảm bảo tính nhất quán, tối ưu hóa truy vấn và dễ dàng mở rộng.
*   **Quy trình thiết kế CSDL:** Bao gồm việc xác định các thực thể, thuộc tính và mối quan hệ giữa chúng.
*   **Tối ưu hóa với Antigravity IDE và Vibe Coding:** Antigravity IDE có thể hỗ trợ mạnh mẽ quá trình thiết kế bằng cách phân tích yêu cầu, đề xuất schema dựa trên mẫu đã có, tự động tạo và kiểm thử các câu lệnh SQL, cho phép một quy trình thiết kế lặp lại và hiệu quả. Vibe Coding khuyến khích sự tương tác nhanh chóng, trực quan với AI để phát triển schema.
*   **Các loại mối quan hệ chính:**
    *   **Một-Nhiều / Nhiều-Một:** Phổ biến nhất, một bản ghi ở bảng này liên kết với nhiều bản ghi ở bảng kia (ví dụ: Người dùng có nhiều Ảnh). Khóa ngoại đặt ở phía "Nhiều".
    *   **Một-Một:** Một bản ghi ở bảng này liên kết với duy nhất một bản ghi ở bảng kia (ví dụ: Quốc gia có một Thủ đô). Thường dùng để tách dữ liệu vì lý do hiệu suất hoặc bảo mật.
    *   **Nhiều-Nhiều:** Nhiều bản ghi ở bảng này liên kết với nhiều bản ghi ở bảng kia (ví dụ: Sinh viên học nhiều Lớp học). Luôn được triển khai thông qua một **bảng trung gian** chứa các khóa ngoại từ hai bảng gốc và thường có khóa chính tổng hợp.
*   **Khóa Chính (Primary Key - PK):** Một cột (hoặc tập hợp các cột) định danh duy nhất mỗi hàng trong *chính bảng của nó*. Phải duy nhất, không rỗng và lý tưởng là không thay đổi. Trong PostgreSQL, `SERIAL PRIMARY KEY` được sử dụng để tự động tạo ID và tạo chỉ mục.
*   **Khóa Ngoại (Foreign Key - FK):** Một cột (hoặc tập hợp các cột) trong một bảng tham chiếu đến Khóa Chính của một bảng khác, thiết lập mối quan hệ giữa chúng. Đảm bảo **tính toàn vẹn tham chiếu**.
*   **Hành động ràng buộc Khóa Ngoại (`ON DELETE`, `ON UPDATE`):** Các tùy chọn như `CASCADE`, `RESTRICT`, `NO ACTION`, `SET NULL`, `SET DEFAULT` quy định cách CSDL phản ứng khi các bản ghi trong bảng cha bị xóa hoặc cập nhật. Việc lựa chọn hành động phù hợp là rất quan trọng để duy trì tính toàn vẹn và tránh mất mát dữ liệu.
*   **Triển khai thực tế:** Chúng ta đã thành công tạo các bảng `users`, `photos`, `comments`, và `likes` với các khóa chính và khóa ngoại phù hợp, bao gồm cả việc sử dụng `ON DELETE CASCADE` và khóa chính tổng hợp cho mối quan hệ Nhiều-Nhiều, đặt nền tảng vững chắc cho việc xây dựng một cơ sở dữ liệu quan hệ mạnh mẽ cho ứng dụng chia sẻ ảnh.

Các kiến thức và kỹ năng này là nền tảng không thể thiếu cho bất kỳ nhà phát triển ứng dụng nào, cho phép bạn xây dựng các hệ thống dữ liệu có cấu trúc, đáng tin cậy và dễ quản lý.

<!-- REVIEWED_BY_AGENT -->
