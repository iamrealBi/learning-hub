# Phần 8: Tương Tác Nâng Cao, Lệnh Tùy Chỉnh và Hooks với Claude Code và Antigravity IDE

Trong các phần trước, chúng ta đã xây dựng nền tảng vững chắc về việc tương tác với Claude Code, từ thiết lập môi trường đến sử dụng các agent và skill cơ bản. Phần này sẽ đưa bạn đi sâu hơn vào các kỹ thuật tương tác nâng cao, không chỉ giúp tối ưu hóa quy trình phát triển mà còn nâng cao hiệu quả làm việc với Claude Code, đặc biệt khi áp dụng tư duy "Vibe Coding" trong một hệ thống Agentic AI như Antigravity IDE. Chúng ta sẽ khám phá cách lặp lại và cải thiện ứng dụng một cách hiệu quả, xây dựng và sử dụng các lệnh tùy chỉnh để tái sử dụng các prompt phức tạp, tận dụng khả năng xử lý hình ảnh để cung cấp phản hồi trực quan, và cuối cùng là hiểu rõ cơ chế hooks để tự động hóa các tác vụ sau khi Claude Code thực hiện thay đổi.

## 1. Nâng Cao Hiệu Quả Phát Triển Lặp Lại với Claude Code

Claude Code là một công cụ mạnh mẽ hỗ trợ phát triển ứng dụng theo phương pháp lặp lại (iterative development). Để khai thác tối đa sức mạnh này, việc thiết lập dự án hiệu quả và áp dụng tư duy "Vibe Coding" là cực kỳ quan trọng.

### 1.1. Tầm Quan Trọng của Cấu Hình Dự Án và Vibe Coding

Một dự án được thiết lập tốt là chìa khóa để làm việc hiệu quả với Claude Code. Điều này bao gồm việc định nghĩa rõ ràng các agent và skill trong tệp `CLAUDE MD` của bạn ngay từ đầu. `CLAUDE MD` đóng vai trò như một bản thiết kế ban đầu, cung cấp cho Claude Code ngữ cảnh về khả năng, mục tiêu của dự án và các ràng buộc kỹ thuật.

> [!TIP]
> Luôn dành thời gian thiết lập tệp `CLAUDE MD` của bạn với các agent và skill phù hợp ngay khi bắt đầu một dự án mới. Điều này cung cấp cho Claude Code một bản đồ rõ ràng về khả năng và mục tiêu của nó, giảm thiểu nhu cầu chỉnh sửa sau này và tăng độ chính xác của các kế hoạch mà AI đề xuất.

**Vibe Coding: Thiết lập ý định và ngữ cảnh cho AI**

"Vibe Coding" là một triết lý lập trình với AI, nơi trọng tâm là thiết lập *ý định (intent)* và *ngữ cảnh (context)* một cách rõ ràng, cho phép AI tự mình tìm ra *cách thức (how)* thực hiện. Thay vì cung cấp các bước hướng dẫn chi tiết từng dòng mã, bạn định nghĩa "vibe" của những gì cần đạt được.

*   **Ý định:** Mục tiêu cuối cùng của tính năng hoặc tác vụ (ví dụ: "Thêm tính năng đăng nhập bảo mật").
*   **Ngữ cảnh:** Môi trường hiện tại, các ràng buộc công nghệ, phong cách mã hóa, các tệp liên quan (`CLAUDE MD`, `package.json`, cấu trúc thư mục).
*   **Vibe:** Sự kết hợp giữa ý định và ngữ cảnh, được truyền tải qua prompt, tệp cấu hình, và các phản hồi lặp lại.

Khi bạn thiết lập tệp `CLAUDE MD` và cung cấp các prompt chất lượng, bạn đang thực hành Vibe Coding. Bạn không chỉ nói "làm cái này" mà còn "làm cái này *trong ngữ cảnh này* và *theo cách này*".

**So sánh với Antigravity IDE: Ngữ cảnh động và khả năng tự động học**

Trong khi Claude Code dựa nhiều vào cấu hình tĩnh trong `CLAUDE MD` và prompt của người dùng để thiết lập ngữ cảnh, một hệ thống Agentic AI siêu việt như Antigravity IDE có khả năng thu thập và duy trì ngữ cảnh động hơn nhiều.

*   **Antigravity IDE:**
    *   **Ngữ cảnh Động:** Tự động đọc và hiểu toàn bộ codebase, tài liệu, các thay đổi gần đây, và thậm chí cả lịch sử tương tác của người dùng.
    *   **Sub-agents:** Có thể khởi tạo các sub-agent chuyên biệt (ví dụ: một sub-agent chuyên về UI, một sub-agent chuyên về backend) để xử lý các phần khác nhau của tác vụ, mỗi sub-agent duy trì ngữ cảnh riêng và phối hợp với nhau.
    *   **Tự động học:** Antigravity có thể học từ các tương tác trước, từ các mẫu mã thành công, và từ phản hồi của người dùng để cải thiện "vibe" của nó theo thời gian, thích ứng với phong cách và yêu cầu cụ thể của dự án mà không cần cấu hình quá tường minh.
    *   **Browser/CLI/File I/O:** Khả năng tự động duyệt web để tìm kiếm thông tin, chạy lệnh CLI, và đọc/ghi tệp giúp Antigravity xây dựng ngữ cảnh toàn diện hơn và thực thi các kế hoạch phức tạp.

Đối với người dùng Antigravity IDE, Vibe Coding trở nên mạnh mẽ hơn nữa: bạn có thể đặt ra các mục tiêu cấp cao, và Antigravity sẽ tự động điều chỉnh ngữ cảnh, lập kế hoạch, và thực thi các bước chi tiết. Vai trò của bạn chuyển từ "người hướng dẫn từng bước" sang "người định hướng chiến lược".

### 1.2. Quy Trình Phát Triển Lặp Lại Tối Ưu

Khi phát triển một ứng dụng, chúng ta thường làm việc theo các bước lặp lại liên tục:

1.  **Xác định Tính năng/Vấn đề:** Hiểu rõ yêu cầu hoặc lỗi cần khắc phục.
2.  **Gửi Prompt:** Cung cấp cho Claude Code (hoặc Antigravity) một prompt rõ ràng, bao gồm ý định và ngữ cảnh cần thiết.
3.  **Xem xét Kế hoạch:** Claude Code sẽ đề xuất một kế hoạch hành động. Đánh giá kế hoạch này về tính hợp lý, hiệu quả và an toàn.
4.  **Triển khai:** Chấp nhận kế hoạch để Claude Code thực hiện các thay đổi.
5.  **Kiểm tra:** Sau khi triển khai, kiểm tra chức năng, giao diện và hiệu suất.
6.  **Lặp lại:** Nếu có vấn đề hoặc cần cải tiến, quay lại bước 1 với prompt mới.

**Ví dụ về quy trình lặp lại:**

1.  **Triển khai tính năng Đăng nhập/Đăng ký và Bảo vệ Route:**
    *   **Prompt ban đầu (Vibe Coding):** "Triển khai tính năng đăng nhập/đăng ký với NextAuth.js. Sau khi đăng nhập thành công, chuyển hướng người dùng đến `/dashboard`. Bảo vệ tất cả các route liên quan đến ghi chú (ngoại trừ route ghi chú công khai) để chỉ người dùng đã xác thực mới có thể truy cập. Sử dụng các tính năng Next.js hiện đại (App Router) và bảo vệ route ở cấp độ từng route, không phải layout."
    *   **Claude Code phản hồi:** Tạo một kế hoạch chi tiết (ví dụ: tạo API routes cho NextAuth, thêm middleware, chỉnh sửa component đăng nhập).
    *   **Đánh giá & Chấp nhận:** Xem xét kế hoạch. Nếu kế hoạch hợp lý, chấp nhận.
    *   **Kiểm tra:** Đăng nhập thành công có chuyển hướng không? Xóa cookie xác thực có chuyển hướng về trang đăng nhập không? Truy cập route bảo vệ khi chưa đăng nhập có bị từ chối không?
    *   **Lặp lại (nếu cần):** Nếu có vấn đề, gửi prompt mới để sửa lỗi (ví dụ: "Người dùng không được chuyển hướng sau khi đăng nhập, kiểm tra logic `signIn` và `callbackUrl`").

2.  **Thêm tính năng Quản lý Ghi chú (Tạo, Xem, Sửa, Xóa):**
    *   **Xóa ngữ cảnh:** Khi bắt đầu một tác vụ mới, đặc biệt là một tác vụ phức tạp, việc xóa cửa sổ ngữ cảnh (context window) của phiên Claude Code có thể hữu ích để đảm bảo Claude Code tập trung hoàn toàn vào prompt mới.
    *   **Prompt phức tạp (Vibe Coding):** "Thêm module quản lý ghi chú. Bao gồm nút đăng xuất trong header (chỉ hiển thị khi đã xác thực), route `/new-note` để tạo ghi chú, form tạo ghi chú và lưu dữ liệu vào cơ sở dữ liệu. Đồng thời, hiển thị danh sách ghi chú của người dùng trên trang `/dashboard`."
    *   **Kiểm tra & Sửa lỗi UI/UX:**
        *   **Vấn đề 1:** Header không khớp với giao diện tổng thể (nền trắng trên nền tối), nút đăng xuất xuất hiện chậm.
        *   **Prompt sửa lỗi (Vibe Coding):** "Header hiện tại có vấn đề về kiểu dáng (nền trắng trên nền tối). Nút đăng xuất xuất hiện quá chậm do kiểm tra trạng thái xác thực phía client. Hãy sửa lại kiểu dáng header và tối ưu hóa việc kiểm tra trạng thái xác thực để nút đăng xuất hiển thị ngay lập tức bằng cách kiểm tra phía server trước khi gửi trang đến client."
        *   **Kết quả:** Claude Code sửa lỗi, giao diện và hành vi tốt hơn.
    *   **Thêm Rich Text Editor Toolbar:**
        *   **Prompt (Vibe Coding):** "Thêm thanh công cụ cho trình soạn thảo văn bản phong phú (TipTap) trên trang tạo/chỉnh sửa ghi chú. Thanh công cụ cần hỗ trợ các chức năng cơ bản như in đậm, in nghiêng, gạch chân, danh sách, và tiêu đề." (Các công cụ cụ thể thường được định nghĩa trong `CLAUDE MD` hoặc một tệp cấu hình riêng).
        *   **Kết quả:** Claude Code triển khai thanh công cụ, cải thiện trải nghiệm người dùng.

> [!NOTE]
> Mặc dù Claude Code rất mạnh mẽ, kết quả có thể khác nhau do tính ngẫu nhiên của AI. Việc thiết lập dự án tốt giúp tăng khả năng nhận được kết quả mong muốn, nhưng luôn cần kiểm tra và điều chỉnh thủ công. Đối với Antigravity IDE, khả năng tự kiểm tra và tự sửa lỗi cao hơn, nhưng phản hồi của người dùng vẫn là dữ liệu quý giá để nó học hỏi và cải thiện.

**Under the Hood: Cách Claude Code xử lý prompt và kế hoạch**

Khi bạn gửi một prompt, Claude Code thực hiện một quy trình phức tạp:

1.  **Phân tích Ngữ cảnh:** Đọc toàn bộ các tệp trong thư mục dự án được cấu hình, đặc biệt là `CLAUDE MD`, để hiểu cấu trúc dự án, các agent, skill, và các ràng buộc hiện có.
2.  **Hiểu Ý định:** Phân tích prompt của bạn để trích xuất ý định cốt lõi và các yêu cầu cụ thể.
3.  **Lập Kế hoạch:** Dựa trên ngữ cảnh và ý định, Claude Code sử dụng các mô hình ngôn ngữ lớn (LLM) để tạo ra một chuỗi các hành động (kế hoạch) bao gồm việc sử dụng các công cụ (ví dụ: `edit` để sửa tệp, `write` để tạo tệp mới, `read` để đọc tệp, `web-search` để tìm kiếm thông tin).
4.  **Thực thi & Quan sát:** Sau khi bạn chấp nhận kế hoạch, Claude Code thực thi từng bước. Nó quan sát kết quả của mỗi hành động (ví dụ: tệp đã được sửa đổi như thế nào, lỗi nào đã xảy ra).
5.  **Tái lập kế hoạch (nếu cần):** Nếu một hành động không thành công hoặc kết quả không như mong đợi, Claude Code có thể tự động điều chỉnh kế hoạch hoặc chờ phản hồi từ người dùng để tái lập kế hoạch.

**Áp dụng trong Antigravity IDE: Từ ý định cấp cao đến hành động tự động**

Antigravity IDE mở rộng quy trình này bằng cách tự động hóa nhiều bước hơn. Khi bạn cung cấp một ý định cấp cao cho Antigravity:

*   Nó sẽ tự động phân tích ngữ cảnh sâu hơn, bao gồm cả việc chạy các kiểm thử hiện có, kiểm tra linter, hoặc thậm chí mô phỏng hành vi người dùng bằng sub-agent trình duyệt.
*   Antigravity sẽ lập kế hoạch chi tiết hơn, có thể bao gồm việc gọi các sub-agent chuyên biệt để xử lý các phần khác nhau của tác vụ.
*   Nó không chỉ thực thi mà còn *quan sát* kết quả một cách chủ động (ví dụ: lỗi console, thay đổi UI, kết quả kiểm thử) và *tự động lặp lại* nếu phát hiện vấn đề, giảm thiểu nhu cầu phản hồi thủ công của bạn.
*   Khả năng tự chạy script ngầm cho phép Antigravity thực hiện các bước như cài đặt dependencies, chạy build, hoặc khởi động server mà không cần sự can thiệp trực tiếp của bạn.

### 1.3. Phản Hồi Trực Quan qua Ảnh Chụp Màn Hình (Image Vision)

Trong quá trình phát triển, đặc biệt là với các vấn đề liên quan đến giao diện người dùng (UI), việc mô tả vấn đề bằng lời có thể không đủ rõ ràng. Claude Code có khả năng xử lý hình ảnh (image vision), cho phép bạn gửi ảnh chụp màn hình cùng với prompt để cung cấp phản hồi trực quan.

**Khi nào nên sử dụng ảnh chụp màn hình:**

*   **Vấn đề UI/UX:** Khi có lỗi hiển thị, sai vị trí, sai kiểu dáng, hoặc bất kỳ vấn đề nào liên quan đến cách ứng dụng trông hoặc tương tác.
*   **Trình bày trực quan:** Giúp Claude Code "nhìn thấy" chính xác vấn đề mà bạn đang gặp phải, đặc biệt khi mô tả bằng văn bản trở nên phức tạp hoặc mơ hồ.
*   **So sánh trạng thái:** Để chỉ ra sự khác biệt giữa "mong muốn" và "thực tế" trong giao diện.

**Cơ chế Image Vision của Claude Code:**

Khi bạn gửi một ảnh chụp màn hình cùng với prompt, Claude Code sẽ sử dụng một mô hình thị giác máy tính (vision model) để phân tích hình ảnh. Mô hình này có thể nhận diện các thành phần UI, cấu trúc layout, màu sắc, phông chữ, và thậm chí cả các thông báo lỗi hiển thị trên màn hình. Thông tin trích xuất từ hình ảnh sau đó được kết hợp với prompt văn bản để cung cấp một ngữ cảnh phong phú hơn cho mô hình ngôn ngữ chính, giúp nó hiểu sâu sắc hơn về vấn đề UI và đề xuất giải pháp phù hợp.

**Ví dụ thực tế:**

1.  **Lỗi hiển thị dữ liệu thô:**
    *   **Vấn đề:** Claude Code hiển thị dữ liệu JSON thô của ghi chú thay vì nội dung được định dạng.
    *   **Phản hồi:** Chụp ảnh màn hình trang hiển thị JSON thô. Kèm theo prompt mô tả vấn đề và chỉ ra ảnh chụp màn hình: "Ghi chú đang hiển thị dữ liệu JSON thô thay vì nội dung được định dạng. Vui lòng xem ảnh chụp màn hình và sửa lỗi."
    *   **Kết quả:** Claude Code nhận diện vấn đề và triển khai bản sửa lỗi, hiển thị nội dung ghi chú đúng cách.

2.  **Lỗi vị trí và kiểu dáng hộp thoại:**
    *   **Vấn đề:** Hộp thoại xác nhận xóa ghi chú hiển thị sai vị trí và không có kiểu dáng phù hợp.
    *   **Phản hồi:** Chụp ảnh màn hình toàn bộ trang để Claude Code thấy rõ vị trí sai lệch của hộp thoại. Kèm theo prompt mô tả lỗi vị trí và kiểu dáng: "Hộp thoại xác nhận xóa ghi chú đang hiển thị sai vị trí và thiếu kiểu dáng. Hãy căn chỉnh nó vào giữa màn hình và áp dụng kiểu dáng Material Design."
    *   **Kết quả:** Claude Code sửa lỗi, hộp thoại hiển thị đúng vị trí và có kiểu dáng phù hợp.

> [!TIP]
> Đối với các thông báo lỗi hoặc vấn đề liên quan đến văn bản, việc sao chép và dán văn bản thô vào prompt thường hiệu quả hơn. Tuy nhiên, khi gặp các vấn đề về giao diện người dùng, ảnh chụp màn hình là một công cụ cực kỳ hữu ích để Claude Code hiểu rõ ngữ cảnh trực quan.

**Tích hợp trong Antigravity IDE: Quan sát môi trường trực quan và phản ứng**

Antigravity IDE, với khả năng điều khiển sub-agent trình duyệt, có thể tự động chụp ảnh màn hình hoặc thậm chí phân tích DOM (Document Object Model) của ứng dụng đang chạy. Điều này cho phép nó:

*   **Tự động phát hiện lỗi UI:** Antigravity có thể so sánh trạng thái UI hiện tại với một thiết kế mong muốn hoặc các quy tắc UI/UX đã biết và tự động phát hiện sai lệch.
*   **Phản hồi thông minh hơn:** Khi người dùng cung cấp ảnh chụp màn hình, Antigravity không chỉ "nhìn" mà còn có thể "tương tác" với UI ảo để thử nghiệm các giải pháp tiềm năng trước khi áp dụng chúng vào mã nguồn.
*   **Kiểm thử UI tự động:** Tích hợp khả năng vision vào quy trình kiểm thử tự động, cho phép Antigravity chạy các bài kiểm thử end-to-end và xác minh giao diện người dùng một cách trực quan.

## 2. Tối Ưu Hóa Quy Trình với Lệnh Tùy Chỉnh (Custom Commands)

Trong quá trình làm việc với Claude Code, bạn có thể nhận thấy mình thường xuyên gửi cùng một loại prompt cho các tác vụ lặp lại, chẳng hạn như "đánh giá mã" (code review), "tạo một component mới" hoặc "viết kiểm thử đơn vị". Việc lặp lại các prompt này có thể tốn thời gian, dễ gây lỗi và thiếu nhất quán. Đó là lúc các lệnh tùy chỉnh (Custom Commands) phát huy tác dụng.

### 2.1. Khái Niệm và Lợi Ích của Lệnh Tùy Chỉnh

Lệnh tùy chỉnh trong Claude Code về cơ bản là các prompt có thể tái sử dụng. Chúng cho phép bạn định nghĩa một prompt phức tạp một lần và sau đó gọi nó bằng một lệnh ngắn gọn, thậm chí có thể truyền đối số để làm cho prompt trở nên linh hoạt hơn, giống như việc gọi một hàm trong lập trình.

**Lợi ích của Lệnh Tùy Chỉnh:**

*   **Tái sử dụng:** Tránh lặp lại việc gõ các prompt dài và phức tạp, tiết kiệm thời gian và công sức.
*   **Nhất quán:** Đảm bảo rằng các tác vụ lặp lại được thực hiện theo cùng một cách mỗi lần, tuân thủ các tiêu chuẩn và quy tắc đã định.
*   **Linh hoạt:** Sử dụng đối số để tùy chỉnh hành vi của lệnh mà không cần sửa đổi prompt gốc, tạo ra các lệnh động.
*   **Hiệu quả:** Tăng tốc độ làm việc bằng cách giảm thiểu việc gõ, suy nghĩ về cấu trúc prompt và giảm tải nhận thức.
*   **Quản lý kiến thức:** Tập trung các prompt tốt nhất vào một nơi, dễ dàng chia sẻ và duy trì trong đội nhóm.

### 2.2. Cấu Trúc và Cơ Chế Hoạt Động

Một lệnh tùy chỉnh được định nghĩa trong một tệp Markdown (`.md`) trong thư mục `commands`. Thư mục này có thể là cục bộ trong dự án của bạn (ví dụ: `.claude/commands`) hoặc toàn cục trong thư mục cấu hình Claude Code của người dùng (thường là `~/.claude/commands`).

**Cấu trúc ví dụ: Lệnh đánh giá mã (`code-review.md`)**

```markdown
---
description: "Thực hiện đánh giá mã để tìm lỗi, lỗ hổng bảo mật hoặc các vấn đề khác."
allowedTools: ["read"]
---
Bạn là một chuyên gia đánh giá mã. Hãy xem xét mã trong ngữ cảnh hiện tại và cung cấp phản hồi chi tiết dựa trên chế độ được chỉ định.

Chế độ đánh giá: {$arguments}

Nếu chế độ là 'bugs', hãy tìm kiếm các lỗi logic, lỗi runtime và các vấn đề về hiệu suất.
Nếu chế độ là 'security', hãy tìm kiếm các lỗ hổng bảo mật tiềm ẩn, chẳng hạn như SQL injection, XSS, hoặc các vấn đề xác thực/ủy quyền.
Nếu chế độ là 'best-practices', hãy tìm kiếm các cơ hội cải thiện mã theo các tiêu chuẩn ngành và các mẫu thiết kế tốt.
Bạn có thể kết hợp các chế độ, ví dụ: 'bugs,security'.

Đảm bảo rằng phản hồi của bạn bao gồm:
- Danh sách các vấn đề được tìm thấy.
- Mức độ ưu tiên của mỗi vấn đề.
- Giải thích ngắn gọn về vấn đề.
- Gợi ý về cách khắc phục.
```

**Giải thích cấu trúc:**

*   **Metadata (Front Matter):** Phần này được bao bọc bởi `---` ở đầu tệp và chứa các thông tin cấu hình cho lệnh.
    *   `description`: Mô tả ngắn gọn về lệnh. Mô tả này sẽ hiển thị khi bạn gõ `/` để duyệt qua các lệnh có sẵn, giúp bạn dễ dàng tìm và hiểu chức năng của từng lệnh.
    *   `allowedTools`: Một mảng các công cụ mà Claude Code được phép sử dụng khi lệnh này được gọi.
        *   Ví dụ, `["read"]` chỉ cho phép Claude Code đọc các tệp trong dự án. Điều này cực kỳ hữu ích cho các tác vụ như đánh giá mã, nơi bạn muốn Claude Code phân tích mã mà không tự ý sửa đổi nó.
        *   Nếu không được chỉ định, Claude Code sẽ có thể sử dụng tất cả các công cụ trong ngữ cảnh xung quanh, bao gồm cả việc chỉnh sửa và ghi tệp.
*   **Prompt:** Nội dung chính của lệnh, đây là phần sẽ được gửi đến mô hình ngôn ngữ của Claude Code.
    *   `{$arguments}`: Đây là một placeholder đặc biệt. Khi bạn gọi lệnh và truyền đối số, placeholder này sẽ được thay thế bằng giá trị của đối số đó. Điều này cho phép bạn tạo các lệnh động và linh hoạt, tương tự như các tham số trong một hàm lập trình.
    *   **Logic prompt:** Bạn có thể định nghĩa logic bên trong prompt dựa trên giá trị của `$arguments`. Ví dụ trên kiểm tra giá trị của `mode` (được truyền qua `$arguments`) để điều chỉnh hành vi đánh giá mã.

**Under the Hood: Parsing và thay thế đối số**

Khi bạn gọi một lệnh tùy chỉnh, Claude Code thực hiện các bước sau:

1.  **Phân tích Cú pháp:** Nó phân tích cú pháp lệnh bạn nhập (ví dụ: `/code-review bugs,security`) để xác định tên lệnh (`code-review`) và các đối số (`bugs,security`).
2.  **Tải Prompt Template:** Claude Code tìm tệp `.md` tương ứng trong thư mục `commands` (ví dụ: `code-review.md`).
3.  **Thay thế Placeholder:** Nó thay thế tất cả các lần xuất hiện của `{$arguments}` trong nội dung prompt bằng các đối số đã được cung cấp.
4.  **Gửi Prompt Đã Xử lý:** Prompt cuối cùng, với các đối số đã được chèn, cùng với ngữ cảnh dự án và các ràng buộc `allowedTools`, sẽ được gửi đến mô hình ngôn ngữ của Claude Code để xử lý.

### 2.3. Hướng Dẫn Sử Dụng và Ví Dụ Thực Tế

1.  **Tạo tệp lệnh:** Đặt tệp `code-review.md` vào thư mục `.claude/commands` trong dự án của bạn.
2.  **Tải lệnh:** Sau khi định nghĩa lệnh tùy chỉnh, bạn cần đóng và khởi động lại phiên Claude Code để đảm bảo các lệnh mới được tải vào bộ nhớ.
3.  **Gọi lệnh:** Trong phiên Claude Code, gõ dấu gạch chéo (`/`) để xem danh sách các lệnh có sẵn. Chọn lệnh của bạn (ví dụ: `/code-review`).
4.  **Truyền đối số (tùy chọn):** Sau khi chọn lệnh, bạn có thể nhấn Enter để chạy nó không có đối số (nếu prompt cho phép), hoặc cung cấp các đối số.
    *   Ví dụ: `/code-review bugs,security` sẽ truyền chuỗi "bugs,security" vào placeholder `{$arguments}` trong prompt.
    *   Một ví dụ khác: `/create-component Button` có thể tạo một component React tên là `Button`.
5.  **Claude Code thực thi:** Claude Code sẽ sử dụng prompt đã định nghĩa cùng với các đối số được cung cấp để thực hiện tác vụ.

> [!NOTE]
> Các lệnh tùy chỉnh không được Claude Code tự động khám phá hoặc thực thi. Chúng được thiết kế để bạn chủ động gọi khi cần. Điều này khác với các agent hoặc skill được định nghĩa trong `CLAUDE MD` có thể được AI tự động sử dụng trong quá trình lập kế hoạch.

### 2.4. Lệnh Tùy Chỉnh và Antigravity IDE: Xây Dựng Quy Trình Nâng Cao

Trong Antigravity IDE, khái niệm "lệnh tùy chỉnh" có thể được hiểu rộng hơn. Chúng tương tự như "macros" hoặc "workflows" được định nghĩa trước mà Antigravity có thể kích hoạt.

*   **Tích hợp sâu sắc hơn:** Antigravity có thể không chỉ thực thi các lệnh mà còn *học* cách sử dụng chúng hiệu quả nhất dựa trên ngữ cảnh và mục tiêu.
*   **Tự động hóa lệnh:** Trong khi Claude Code yêu cầu bạn gọi lệnh thủ công, Antigravity có thể tự động quyết định khi nào nên sử dụng một "workflow" hoặc "macro" cụ thể để đạt được mục tiêu tổng thể của nó.
*   **Tạo lệnh động:** Antigravity có thể thậm chí tự động tạo hoặc sửa đổi các lệnh tùy chỉnh dựa trên các mẫu lặp lại mà nó quan sát được trong quá trình làm việc của bạn hoặc của các nhà phát triển khác.
*   **Phiên bản hóa và chia sẻ:** Với một hệ thống như Antigravity, các lệnh tùy chỉnh có thể được phiên bản hóa và chia sẻ dễ dàng giữa các dự án hoặc thành viên trong nhóm, tạo ra một thư viện các "hành vi" có thể tái sử dụng cho AI.

Việc định nghĩa lệnh tùy chỉnh trong Claude Code là một bước quan trọng để chuẩn hóa và tối ưu hóa các tương tác, và nó cũng là một hình thức Vibe Coding, nơi bạn định nghĩa "vibe" của một tác vụ lặp lại để AI có thể thực hiện một cách nhất quán.

## 3. Tự Động Hóa Tác Vụ với Hooks

Mặc dù Claude Code có thể viết mã, nhưng có những tác vụ bổ sung mà chúng ta thường muốn thực hiện sau khi mã được tạo hoặc sửa đổi, chẳng hạn như định dạng mã, chạy kiểm thử, hoặc cập nhật tài liệu. Việc thực hiện thủ công các tác vụ này sau mỗi lần Claude Code thay đổi tệp có thể tốn thời gian, dễ bị bỏ sót và làm gián đoạn luồng công việc. Hooks của Claude Code cung cấp một giải pháp mạnh mẽ để tự động hóa các tác vụ này.

### 3.1. Hooks: Cơ Chế Phản Ứng Với Sự Kiện

Hooks trong Claude Code là các cơ chế cho phép bạn thực thi mã tùy chỉnh để phản ứng với các sự kiện nhất định được phát ra bởi Claude Code. Điều này giống như các "event listener" hoặc "callback" trong lập trình, nơi một hàm được gọi tự động khi một sự kiện cụ thể xảy ra.

> [!NOTE]
> Khái niệm hooks tương tự như các "pre-commit hooks" trong Git (chạy script trước khi commit) hoặc các bước trong quy trình CI/CD (chạy script ở các giai đoạn cụ thể của pipeline), nơi các script được chạy tự động ở các giai đoạn cụ thể của quy trình làm việc.

**Under the Hood: Hệ thống sự kiện của Claude Code**

Claude Code được xây dựng trên một kiến trúc hướng sự kiện. Khi một công cụ được sử dụng (ví dụ: `edit` một tệp), một tin nhắn được gửi đi, hoặc một phản hồi được nhận, hệ thống sẽ phát ra một sự kiện tương ứng. Các hooks được cấu hình sẽ "lắng nghe" các sự kiện này. Khi một sự kiện khớp với `matcher` của một hook, các hành động được định nghĩa trong hook đó sẽ được kích hoạt. Điều này cho phép một luồng công việc tự động hóa và không chặn, nơi các tác vụ nền có thể chạy mà không làm gián đoạn quá trình ra quyết định của AI.

### 3.2. Cấu Hình Hooks trong `settings.json`

Hooks được cấu hình trong tệp `settings.json` của Claude Code. Bạn có thể đặt chúng cục bộ trong thư mục `.claude/settings.json` của dự án (không nên đưa vào kiểm soát phiên bản nếu chứa thông tin nhạy cảm hoặc chỉ dành cho môi trường phát triển của bạn) hoặc toàn cục trên hệ thống của bạn (thường là `~/.claude/settings.json`).

**Cấu trúc ví dụ trong `settings.json`:**

```json
{
  "hooks": {
    "post-tool-use": [
      {
        "matcher": "edit|write",
        "hooks": [
          {
            "type": "command",
            "command": "cd \"$CLAUDE_PROJECT_DIR\" && npm run format --if-present 2>/dev/null || true"
          },
          {
            "type": "command",
            "command": "cd \"$CLAUDE_PROJECT_DIR\" && npm test --if-present 2>/dev/null || true"
          }
        ]
      }
    ],
    "on-message": [
        {
            "matcher": ".*error.*",
            "hooks": [
                {
                    "type": "prompt",
                    "prompt": "Tôi thấy một lỗi trong tin nhắn của bạn. Hãy phân tích lỗi này và đề xuất cách khắc phục."
                }
            ]
        }
    ]
  }
}
```

**Giải thích cấu trúc:**

*   **`hooks`:** Nút chính chứa tất cả các định nghĩa hook.
*   **`"post-tool-use"`:** Tên sự kiện mà hook sẽ phản ứng. Đây là một hook chạy *sau khi* một công cụ đã được gọi. Các công cụ trong Claude Code bao gồm `edit` (chỉnh sửa tệp), `write` (ghi tệp), `read` (đọc tệp), `web-search` (tìm kiếm web), v.v.
    *   **Các loại sự kiện hook khác:** Claude Code cung cấp nhiều loại sự kiện hook khác nhau, ví dụ:
        *   `pre-tool-use`: Trước khi sử dụng công cụ.
        *   `on-message`: Khi nhận được một tin nhắn từ Claude Code (hoặc từ người dùng).
        *   `on-plan-complete`: Khi Claude Code hoàn thành việc lập kế hoạch.
        *   `on-agent-start`: Khi một agent bắt đầu hoạt động.
        *   Bạn nên tham khảo tài liệu chính thức của Claude Code để biết danh sách đầy đủ và chi tiết về các sự kiện.
*   **Mảng các hook cho sự kiện:** Mỗi sự kiện có thể có nhiều bộ hook được định nghĩa.
    *   **`matcher`:** Một biểu thức chính quy (regex) xác định công cụ cụ thể nào đã được sử dụng để kích hoạt hook này.
        *   Đối với `post-tool-use`, chúng ta chỉ định `"edit|write"` để hook được kích hoạt khi Claude Code chỉnh sửa hoặc ghi một tệp. Sử dụng ký hiệu `|` để chỉ ra nhiều tùy chọn khớp.
        *   Đối với `on-message`, `" .*error.* "` sẽ kích hoạt hook nếu tin nhắn chứa từ "error".
    *   **`hooks` (mảng hành động):** Một mảng chứa các hành động sẽ được thực thi khi `matcher` khớp.
        *   **`type`:** Loại hành động sẽ được thực hiện. Các loại phổ biến là:
            *   `"command"`: Thực thi một lệnh shell (bash).
            *   `"prompt"`: Chèn một prompt mới vào cuộc trò chuyện Claude Code, như thể người dùng đã gõ nó.
            *   `"agent"`: Kích hoạt một agent khác đã được định nghĩa trong `CLAUDE MD`.
        *   **`command`:** (Chỉ áp dụng cho `type: "command"`) Chuỗi lệnh shell sẽ được thực thi.
            *   `cd "$CLAUDE_PROJECT_DIR"`: Điều hướng đến thư mục dự án hiện tại của Claude Code. `CLAUDE_PROJECT_DIR` là một biến môi trường đặc biệt được Claude Code cung cấp tự động, chứa đường dẫn tuyệt đối đến thư mục gốc của dự án.
            *   `npm run format --if-present`: Chạy script `format` được định nghĩa trong tệp `package.json`. `--if-present` là một cờ của npm đảm bảo lệnh không lỗi nếu script không tồn tại.
            *   `2>/dev/null`: Chuyển hướng lỗi chuẩn (stderr) đến `/dev/null` để tránh làm Claude Code bị gián đoạn bởi các thông báo lỗi không mong muốn từ formatter hoặc test runner.
            *   `|| true`: Đảm bảo lệnh shell luôn trả về mã thoát thành công, ngay cả khi `npm run format` hoặc `npm test` thất bại. Điều này ngăn Claude Code dừng công việc của nó chỉ vì việc định dạng hoặc kiểm thử không thành công (mà bạn có thể muốn xử lý riêng).

### 3.3. Ví Dụ Thực Tế: Tự Động Định Dạng Mã và Chạy Kiểm Thử

Hãy tưởng tượng bạn muốn tất cả mã được Claude Code tạo ra hoặc chỉnh sửa đều tuân thủ một tiêu chuẩn định dạng nhất định và bạn cũng muốn chạy kiểm thử cơ bản sau mỗi lần thay đổi mã.

1.  **Cài đặt Formatter và Test Runner:**
    *   Cài đặt một công cụ định dạng mã, ví dụ: `OxFormat`, Prettier, hoặc ESLint với `--fix`.
    *   Cài đặt một framework kiểm thử, ví dụ: Jest.
    *   `npm install --save-dev oxformat jest`
2.  **Cấu hình Formatter và Test Script:**
    *   Tạo tệp cấu hình cho formatter (ví dụ: `.oxformat.json`) và định nghĩa các quy tắc định dạng (ví dụ: `singleQuotes: true`).
    *   Thêm script để chạy formatter và kiểm thử trong tệp `package.json`:

    ```json
    // package.json
    {
      "name": "my-app",
      "version": "1.0.0",
      "scripts": {
        "format": "oxformat --fix .",
        "test": "jest"
      },
      "devDependencies": {
        "oxformat": "^0.x.x",
        "jest": "^2x.x"
      }
    }
    ```

3.  **Cấu hình Hook trong `settings.json`:**
    *   Thêm hook như đã mô tả ở trên vào tệp `settings.json` của Claude Code:

    ```json
    // .claude/settings.json (hoặc ~/.claude/settings.json)
    {
      "hooks": {
        "post-tool-use": [
          {
            "matcher": "edit|write",
            "hooks": [
              {
                "type": "command",
                "command": "cd \"$CLAUDE_PROJECT_DIR\" && npm run format --if-present 2>/dev/null || true"
              },
              {
                "type": "command",
                "command": "cd \"$CLAUDE_PROJECT_DIR\" && npm test --if-present 2>/dev/null || true"
              }
            ]
          }
        ]
      }
    }
    ```

4.  **Kiểm tra:**
    *   Khởi động lại phiên Claude Code để tải các hook mới.
    *   Yêu cầu Claude Code thực hiện một thay đổi nào đó vào một tệp mã (ví dụ: "Tạo một component React mới tên là `HelloWorld`").
    *   Sau khi Claude Code hoàn tất, kiểm tra tệp đó. Bạn sẽ thấy mã đã được định dạng tự động theo quy tắc của bạn và bạn có thể thấy kết quả của bài kiểm thử (nếu có) trong console hoặc log.

> [!TIP]
> Bạn có thể xem và quản lý hooks tương tác bằng cách gõ `/hooks` trong phiên Claude Code. Mặc dù việc chỉnh sửa trực tiếp trong tệp JSON thường dễ hơn cho các cấu hình phức tạp.

### 3.4. Lợi Ích và Ứng Dụng Nâng Cao của Hooks

*   **Tự động hóa:** Giảm thiểu công việc thủ công sau khi Claude Code thực hiện thay đổi, giải phóng thời gian cho các tác vụ phức tạp hơn.
*   **Nhất quán:** Đảm bảo các quy tắc (ví dụ: định dạng mã, kiểm thử, phân tích linter) luôn được áp dụng một cách tự động, duy trì chất lượng mã.
*   **Tích hợp liền mạch:** Tích hợp các công cụ bên ngoài (formatter, linter, test runner, công cụ build, công cụ triển khai) vào quy trình làm việc của Claude Code.
*   **Tăng hiệu suất:** Tăng tốc độ phát triển bằng cách tự động hóa các tác vụ lặp lại và giảm thiểu lỗi do quên thực hiện các bước cần thiết.
*   **Phản hồi nhanh:** Nhận phản hồi ngay lập tức về chất lượng hoặc tính đúng đắn của mã sau mỗi thay đổi, giúp phát hiện và sửa lỗi sớm.

**Hooks trong Antigravity IDE: Tích hợp sâu hơn vào quy trình tự động**

Trong Antigravity IDE, khái niệm hooks được mở rộng và tích hợp sâu sắc hơn vào kiến trúc Agentic AI:

*   **Internal Event System:** Antigravity có một hệ thống sự kiện nội bộ phức tạp hơn nhiều, không chỉ phản ứng với việc sử dụng công cụ mà còn với các sự kiện như "mã được tạo", "kiểm thử thất bại", "lỗi runtime được phát hiện", "thay đổi UI", v.v.
*   **AI-driven Hooks:** Thay vì chỉ thực thi các lệnh shell tĩnh, Antigravity có thể sử dụng các "AI-driven hooks" để tự động quyết định hành động tiếp theo. Ví dụ, nếu một kiểm thử thất bại sau khi nó thay đổi mã, một hook có thể kích hoạt một sub-agent chuyên trách để phân tích lỗi, đề xuất sửa chữa, và tự động áp dụng bản sửa lỗi.
*   **Workflow Orchestration:** Hooks trong Antigravity không chỉ là các lệnh độc lập mà có thể là một phần của một quy trình làm việc (workflow) lớn hơn. Ví dụ, một hook `post-code-change` có thể kích hoạt một workflow bao gồm: định dạng mã -> chạy linter -> chạy kiểm thử đơn vị -> chạy kiểm thử tích hợp -> triển khai lên môi trường staging (nếu tất cả kiểm thử đều thành công).
*   **Tự động phản hồi và học hỏi:** Antigravity có thể học từ kết quả của các hooks. Nếu một hook báo cáo lỗi thường xuyên, Antigravity có thể điều chỉnh cách nó tạo mã để tránh các lỗi đó trong tương lai.

Việc hiểu và cấu hình hooks trong Claude Code là một bước quan trọng để bạn bắt đầu xây dựng các quy trình làm việc tự động và hiệu quả hơn, chuẩn bị cho việc tương tác với các hệ thống Agentic AI tiên tiến như Antigravity IDE.

---

## Tóm Tắt Phần 8: Tương Tác Nâng Cao, Lệnh Tùy Chỉnh và Hooks với Claude Code và Antigravity IDE

*   **Nâng Cao Hiệu Quả Phát Triển Lặp Lại:**
    *   **Thiết lập dự án kỹ lưỡng** với tệp `CLAUDE MD` là nền tảng cho việc lặp lại hiệu quả và áp dụng tư duy Vibe Coding.
    *   **Vibe Coding** tập trung vào việc thiết lập ý định và ngữ cảnh rõ ràng cho AI, cho phép nó tự tìm ra cách thức triển khai.
    *   **Quy trình phát triển lặp lại** bao gồm xác định tính năng, gửi prompt, xem xét kế hoạch, triển khai, kiểm tra và lặp lại.
    *   **Claude Code xử lý prompt** bằng cách phân tích ngữ cảnh, hiểu ý định, lập kế hoạch và thực thi từng bước.
    *   **Antigravity IDE** mở rộng điều này với ngữ cảnh động, sub-agents, khả năng tự học và tự động hóa cao hơn.
*   **Sử dụng Ảnh Chụp Màn Hình để Phản Hồi:**
    *   **Claude Code có khả năng xử lý hình ảnh (image vision)**, cho phép bạn gửi ảnh chụp màn hình cùng với prompt để cung cấp phản hồi trực quan.
    *   Ảnh chụp màn hình đặc biệt hữu ích cho các vấn đề về **giao diện người dùng (UI)** như sai vị trí, sai kiểu dáng hoặc lỗi hiển thị trực quan.
    *   **Cơ chế image vision** phân tích hình ảnh và kết hợp với prompt văn bản để hiểu vấn đề.
    *   **Antigravity IDE** có thể tự động quan sát UI, phát hiện lỗi và phản ứng thông minh hơn.
*   **Lệnh Tùy Chỉnh (Custom Commands):**
    *   Là các **prompt có thể tái sử dụng**, giúp tránh lặp lại các prompt phức tạp cho các tác vụ thường xuyên, tăng tính nhất quán và hiệu quả.
    *   Được định nghĩa trong các tệp Markdown trong thư mục `commands` và có thể chứa metadata (mô tả, `allowedTools`) và placeholder `{$arguments}` để linh hoạt.
    *   **Cơ chế hoạt động** bao gồm phân tích cú pháp, thay thế placeholder và gửi prompt đã xử lý.
    *   Lệnh tùy chỉnh cần được tải lại bằng cách khởi động lại phiên Claude Code và được gọi thủ công bằng `/` trong terminal.
    *   Trong **Antigravity IDE**, chúng tương tự như "macros" hoặc "workflows" và có thể được tự động kích hoạt hoặc học hỏi.
*   **Hooks:**
    *   Là cơ chế **tự động hóa các tác vụ** bằng cách phản ứng với các sự kiện của Claude Code (ví dụ: `post-tool-use`, `pre-tool-use`, `on-message`).
    *   Được cấu hình trong tệp `settings.json` và bao gồm tên sự kiện, `matcher` (biểu thức chính quy) để lọc công cụ hoặc tin nhắn, và một mảng các hành động (`type: "command"`, `"prompt"`, `"agent"`).
    *   Biến `CLAUDE_PROJECT_DIR` hữu ích để thực thi lệnh trong thư mục dự án.
    *   Ví dụ điển hình là tự động định dạng mã và chạy kiểm thử bằng cách chạy một script formatter/test runner sau mỗi lần Claude Code chỉnh sửa hoặc ghi tệp.
    *   **Hệ thống sự kiện của Claude Code** là nền tảng cho hooks hoạt động.
    *   **Antigravity IDE** mở rộng hooks thành hệ thống sự kiện nội bộ phức tạp hơn, có khả năng AI-driven hooks và điều phối workflow.

Với các kỹ thuật tương tác nâng cao này, bạn có thể biến Claude Code thành một cộng sự phát triển mạnh mẽ và hiệu quả hơn, tự động hóa nhiều khía cạnh của quy trình làm việc và tập trung vào việc giải quyết các thách thức phức tạp hơn trong dự án của mình, đồng thời chuẩn bị tư duy để làm việc với các hệ thống Agentic AI tiên tiến như Antigravity IDE.

<!-- REVIEWED_BY_AGENT -->
