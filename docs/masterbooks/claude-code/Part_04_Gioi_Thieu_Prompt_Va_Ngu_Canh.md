# Phần 4: Kỹ Thuật Prompt, Kỹ Thuật Ngữ Cảnh và Tư Duy Lập Trình AI

Chào mừng bạn đến với Phần 4 của khóa học, nơi chúng ta sẽ đi sâu vào nền tảng của việc tương tác hiệu quả với các mô hình ngôn ngữ lớn (LLM), đặc biệt là trong bối cảnh phát triển phần mềm với công cụ Claude Code. Sau khi đã thiết lập môi trường làm việc, trọng tâm hiện tại là khám phá cách khai thác tối đa sức mạnh của AI thông qua kỹ thuật prompt (Prompt Engineering) và kỹ thuật ngữ cảnh (Context Engineering). Đây không chỉ là những kỹ năng cơ bản mà còn là kim chỉ nam cho tư duy lập trình với AI (AI-assisted development), đặc biệt khi làm việc với các hệ thống agentic tiên tiến như Antigravity IDE.

Trong phần này, chúng ta sẽ không chỉ tìm hiểu lý thuyết mà còn áp dụng trực tiếp vào một dự án minh họa: một ứng dụng ghi chú web đầy đủ tính năng. Mặc dù dự án demo này được xây dựng bằng JavaScript/TypeScript và framework Next.js, trọng tâm chính của chúng ta là cách thức sử dụng Claude Code để phát triển ứng dụng, cũng như cách tư duy để chuẩn bị cho việc sử dụng các hệ thống AI phức tạp hơn.

Mục tiêu chính của phần này là trang bị cho bạn kiến thức và kỹ năng để:

*   Hiểu rõ bản chất và tầm quan trọng của kỹ thuật prompt và kỹ thuật ngữ cảnh trong tương tác với LLM.
*   Xây dựng các prompt hiệu quả, rõ ràng và chính xác cho Claude Code.
*   Cung cấp ngữ cảnh phù hợp và đầy đủ để Claude Code có thể đưa ra kết quả mong muốn.
*   Nắm vững cách làm việc với các tài liệu đặc tả và tích hợp chúng vào quy trình phát triển bằng Claude Code.
*   Khám phá các tính năng quan trọng của Claude Code như cách tham chiếu tệp, quản lý thông tin bổ sung, và các kỹ thuật nâng cao khác để tối ưu hóa tương tác với AI.
*   Phát triển tư duy "Vibe Coding" để chuẩn bị cho việc làm việc với các hệ thống AI agentic như Antigravity IDE.

Hãy cùng bắt đầu hành trình biến những ý tưởng phức tạp thành các giải pháp thực tế với sự hỗ trợ của AI.

## 1. Nền Tảng Tương Tác AI: Kỹ Thuật Prompt và Kỹ Thuật Ngữ Cảnh

Trong thế giới của các mô hình ngôn ngữ lớn (LLM), cách chúng ta giao tiếp với chúng là yếu tố then chốt quyết định chất lượng đầu ra. "Kỹ thuật Prompt" và "Kỹ thuật Ngữ cảnh" là hai khái niệm cơ bản nhưng cực kỳ mạnh mẽ, giúp chúng ta khai thác tối đa tiềm năng của AI.

### 1.1. Mô Hình Ngôn Ngữ Lớn (LLM) Hoạt Động Như Thế Nào?

Để hiểu rõ tầm quan trọng của kỹ thuật prompt và ngữ cảnh, chúng ta cần có cái nhìn tổng quan về cơ chế hoạt động của LLM. Về cơ bản, LLM là các mô hình học sâu được huấn luyện trên một lượng dữ liệu văn bản khổng lồ. Mục tiêu chính của chúng là dự đoán từ tiếp theo (hoặc chuỗi token tiếp theo) dựa trên các từ đã cho trước. Chúng không "hiểu" theo cách con người, mà là nhận diện các mẫu, mối quan hệ thống kê phức tạp giữa các từ, câu và khái niệm trong dữ liệu huấn luyện.

Khi bạn đưa ra một "prompt" (lời nhắc, câu lệnh), LLM sẽ sử dụng chuỗi token của prompt đó làm điểm khởi đầu. Mô hình sẽ tính toán xác suất của các token tiếp theo có thể xuất hiện, dựa trên các mẫu đã học. Quá trình này lặp lại, tạo ra một chuỗi văn bản mới. Điều này giải thích tại sao một thay đổi nhỏ trong prompt có thể dẫn đến sự khác biệt lớn trong kết quả: nó thay đổi điểm khởi đầu và hướng đi xác suất của mô hình.

Chất lượng của chuỗi văn bản này phụ thuộc rất nhiều vào prompt ban đầu. Một prompt tốt sẽ dẫn dắt mô hình đi đúng hướng, tạo ra kết quả chính xác, mạch lạc và phù hợp với yêu cầu. Ngược lại, một prompt kém có thể dẫn đến kết quả mơ hồ, không chính xác, hoặc thậm chí là "ảo giác" (hallucinations) — thông tin sai lệch do AI tự bịa ra vì không tìm thấy mẫu phù hợp trong dữ liệu huấn luyện hoặc bị hướng dẫn sai lệch.

### 1.2. Kỹ Thuật Prompt (Prompt Engineering) là gì?

Kỹ thuật Prompt là nghệ thuật và khoa học về cách thiết kế các lời nhắc (prompts) để đạt được kết quả mong muốn một cách tối ưu từ một mô hình ngôn ngữ lớn. Nó bao gồm việc lựa chọn từ ngữ, cấu trúc câu, định dạng, và các yếu tố khác để hướng dẫn AI thực hiện một nhiệm vụ cụ thể. Đây là cầu nối giữa ý định của con người và khả năng của AI.

> [!TIP]
> **Nguyên tắc vàng:** "Đầu vào tốt sẽ cho ra đầu ra tốt." (Good input equals good results).
>
> Nguyên tắc này nhấn mạnh rằng chất lượng của kết quả AI tỷ lệ thuận với chất lượng của thông tin và hướng dẫn bạn cung cấp.

Một prompt hiệu quả thường bao gồm các thành phần sau:

*   **Hướng dẫn cụ thể (Specific Instructions):** Mô tả rõ ràng nhiệm vụ hoặc vấn đề bạn muốn AI giải quyết. Tránh sự mơ hồ và sử dụng ngôn ngữ chính xác. Ví dụ: thay vì "Viết code", hãy nói "Viết một hàm JavaScript nhận vào hai số nguyên và trả về tổng của chúng."
*   **Vai trò (Role-playing - tùy chọn):** Gán một vai trò cho AI để định hướng phong cách và kiến thức mà nó nên sử dụng. Ví dụ: "Bạn là một kiến trúc sư phần mềm cấp cao...", "Bạn là một chuyên gia bảo mật...".
*   **Định dạng đầu ra mong muốn (Desired Output Format):** Chỉ rõ định dạng bạn muốn AI trả về (ví dụ: JSON, Markdown, Python code, danh sách gạch đầu dòng).
*   **Ví dụ (Few-shot examples - tùy chọn):** Cung cấp một hoặc vài cặp ví dụ đầu vào/đầu ra để AI hiểu rõ hơn về mẫu bạn mong muốn. Điều này đặc biệt hữu ích cho các nhiệm vụ phức tạp hoặc ít phổ biến.
*   **Hạn chế và ràng buộc (Constraints/Requirements):** Đặt ra các giới hạn về độ dài, phong cách, nội dung, hoặc các quy tắc cụ thể mà AI phải tuân theo. Ví dụ: "Đảm bảo code tuân thủ PEP 8", "Chỉ trả lời bằng tiếng Việt".

### 1.3. Kỹ Thuật Ngữ Cảnh (Context Engineering) là gì?

Kỹ thuật Ngữ cảnh là quá trình cung cấp thông tin nền tảng, dữ liệu bổ sung, hoặc các tài liệu liên quan cho LLM để nó có thể tạo ra phản hồi chính xác, phù hợp và hữu ích hơn. Ngữ cảnh giúp AI hiểu rõ hơn về tình huống, mục đích, và các ràng buộc của nhiệm vụ, vượt ra ngoài phạm vi của prompt trực tiếp.

Việc cung cấp ngữ cảnh phù hợp là cực kỳ quan trọng vì:

*   **Giảm thiểu sự mơ hồ:** Ngữ cảnh giúp AI loại bỏ các diễn giải sai lệch do thiếu thông tin. Ví dụ, nếu bạn hỏi "tạo một hàm `login`", AI không biết bạn muốn `login` cho hệ thống nào. Cung cấp ngữ cảnh về "hệ thống xác thực `BetterAuth`" sẽ làm rõ yêu cầu.
*   **Tăng cường độ chính xác và liên quan:** Với thông tin đầy đủ, AI có thể đưa ra câu trả lời hoặc giải pháp chính xác hơn và phù hợp với bối cảnh cụ thể của bạn.
*   **Tránh thông tin không cần thiết ("Nhiễu"):** Cung cấp quá nhiều thông tin không liên quan có thể làm giảm hiệu suất của mô hình, khiến nó lạc đề, tạo ra kết quả kém chất lượng, hoặc thậm chí vượt quá giới hạn token của mô hình.

> [!CAUTION]
> **Tránh "Nhiễu" (Irrelevant Context):** Việc cung cấp thông tin không cần thiết có thể gây "nhiễu" cho mô hình, làm giảm chất lượng đầu ra. Ngữ cảnh hiệu quả là ngữ cảnh *có chọn lọc và liên quan trực tiếp* đến nhiệm vụ.

Tóm lại, kỹ thuật prompt và kỹ thuật ngữ cảnh hoạt động song song. Prompt đưa ra yêu cầu trực tiếp, còn ngữ cảnh cung cấp bức tranh toàn cảnh và dữ liệu nền tảng để AI thực hiện yêu cầu đó một cách hiệu quả và chính xác nhất. Chúng là hai mặt của một đồng xu trong việc giao tiếp hiệu quả với LLM.

## 2. Kỹ Thuật Prompt và Ngữ Cảnh Trong Thực Tế với Claude Code

Hãy cùng áp dụng các nguyên tắc này vào một ví dụ thực tế với Claude Code – công cụ CLI AI của Anthropic. Chúng ta sẽ xây dựng một ứng dụng ghi chú (note-taking app) và sử dụng Claude Code để hỗ trợ quá trình phát triển.

### 2.1. Dự Án Minh Họa: Ứng Dụng Ghi Chú

Ứng dụng ghi chú của chúng ta sẽ có các tính năng chính sau:

*   **Tạo ghi chú:** Cho phép người dùng tạo ghi chú mới bằng trình soạn thảo văn bản phong phú (rich text editor) với các tùy chọn định dạng văn bản cơ bản.
*   **Xem ghi chú:** Hiển thị danh sách các ghi chú đã tạo và chi tiết từng ghi chú.
*   **Quản lý ghi chú:** Chỉnh sửa và xóa ghi chú hiện có.
*   **Chia sẻ ghi chú:** Cho phép người dùng chia sẻ ghi chú công khai để người khác có thể xem qua một URL duy nhất.

### 2.2. Khởi Đầu với Claude Code: Tài Liệu Đặc Tả (Specification Document)

Khi bắt đầu một dự án mới hoặc làm việc trên một ứng dụng hiện có, câu hỏi đầu tiên là: làm thế nào để bắt đầu sử dụng Claude Code một cách hiệu quả? Câu trả lời là: **luôn bắt đầu bằng việc cung cấp một ngữ cảnh rõ ràng và một prompt có mục tiêu.**

Đối với việc xây dựng một ứng dụng mới từ đầu, cách hiệu quả nhất là cung cấp một **tài liệu đặc tả kỹ thuật (specification document)** chi tiết. Tài liệu này mô tả đầy đủ ứng dụng mà chúng ta muốn xây dựng, bao gồm các tính năng, cấu trúc dữ liệu, và công nghệ sử dụng. Điều này cũng đúng nếu bạn đang làm việc trên một ứng dụng hiện có; bạn vẫn cần mô tả ứng dụng đó cho Claude Code để nó hiểu được bối cảnh.

> [!NOTE]
> Cá nhân tôi thường tạo các tài liệu đặc tả này với sự trợ giúp của các LLM khác, ví dụ như ChatGPT hoặc chính Claude (qua API hoặc giao diện web). Đây là một công việc chỉ cần làm một lần ở giai đoạn khởi đầu dự án để đảm bảo có một tài liệu nền tảng làm ngữ cảnh cho AI.

**Ví dụ về nội dung tài liệu đặc tả ban đầu:**

*   **Mô tả ứng dụng:** Ứng dụng ghi chú web đơn giản, cho phép người dùng tạo, quản lý và chia sẻ ghi chú.
*   **Các tính năng cốt lõi:**
    *   CRUD (Create, Read, Update, Delete) ghi chú.
    *   Hỗ trợ rich text cho nội dung ghi chú.
    *   Tính năng chia sẻ công khai.
*   **Tech Stack (Ngăn xếp công nghệ):**
    *   Framework Frontend/Backend: Next.js (full-stack)
    *   Runtime: Bun (cho hiệu suất và quản lý gói)
    *   Ngôn ngữ: TypeScript (để đảm bảo kiểu dữ liệu an toàn)
    *   Styling: Tailwind CSS (cho phong cách nhanh chóng và linh hoạt)
    *   Xác thực (Authentication): Thư viện BetterAuth (giải pháp xác thực nhẹ)
    *   Trình soạn thảo văn bản phong phú: Thư viện Tiptap (tích hợp dễ dàng với React/Next.js)
    *   Cơ sở dữ liệu: SQLite (đơn giản cho dự án nhỏ, file-based)

Sau khi có tài liệu đặc tả (thường là một tài liệu dài và chi tiết), chúng ta sẽ lưu nó vào một tệp, ví dụ: `Spec.md` trong thư mục gốc của dự án. Tên tệp này không phải là bắt buộc đối với Claude Code, mà là một lựa chọn cá nhân để tổ chức dự án một cách khoa học.

### 2.3. Kỹ Thuật Ngữ Cảnh với Tham Chiếu Tệp trong Claude Code

Bước tiếp theo là sử dụng Claude Code để xử lý tệp `Spec.md` này. Công việc đầu tiên có thể là yêu cầu Claude Code định dạng lại tệp này thành Markdown chuẩn, hoặc phân tích nó để đưa ra các bước phát triển ban đầu. Đây là lúc kỹ thuật ngữ cảnh đi vào hoạt động một cách trực tiếp.

> [!TIP]
> **Tham chiếu tệp trong Claude Code:** Cách chính thức để chỉ định một tệp cụ thể và đưa toàn bộ nội dung của nó vào ngữ cảnh của prompt là sử dụng ký hiệu `@` theo sau tên tệp. Ví dụ: `@Spec.md`.
>
> Điều này cực kỳ quan trọng. Khi bạn biết rằng một nhiệm vụ nhất định liên quan đến một tệp cụ thể hoặc tệp đó chứa thông tin quan trọng, bạn nên đưa nó vào prompt của mình theo cách này. Claude Code sẽ đọc nội dung của tệp và coi đó là một phần của ngữ cảnh.

**Ví dụ Prompt ban đầu để định dạng tài liệu:**

```
Bạn là một biên tập viên tài liệu kỹ thuật.
Chúng ta đang xây dựng một ứng dụng được mô tả trong @Spec.md.
Vui lòng định dạng lại nội dung của tệp này thành Markdown chuẩn, đảm bảo cấu trúc rõ ràng và dễ đọc.
```

Trong prompt này:

*   `Bạn là một biên tập viên tài liệu kỹ thuật.` là việc gán vai trò, giúp AI định hướng phong cách và mục tiêu.
*   `@Spec.md` là phần ngữ cảnh, chỉ cho Claude Code biết nơi tìm thông tin về ứng dụng.
*   `Vui lòng định dạng lại nội dung của tệp này thành Markdown chuẩn, đảm bảo cấu trúc rõ ràng và dễ đọc.` là hướng dẫn cụ thể về nhiệm vụ.

Claude Code có khả năng tự tìm các tệp hữu ích trong thư mục dự án khi được hỏi các câu hỏi chung, nhưng việc chỉ rõ một tệp mà bạn biết là quan trọng sẽ giúp nó làm việc hiệu quả hơn, giảm thiểu "ảo giác" và cho kết quả tốt hơn. Đây là một hình thức của **ngữ cảnh chủ động**.

### 2.4. Cung Cấp Ngữ Cảnh Bổ Sung và Cập Nhật Đặc Tả

Trong quá trình xem xét tệp `Spec.md` hoặc khi bắt đầu triển khai, chúng ta có thể nhận thấy một số vấn đề hoặc cần bổ sung thông tin chi tiết. Ví dụ, tài liệu đặc tả ban đầu có thể mô tả một bảng `users` (số nhiều) với cấu trúc chung, nhưng thư viện xác thực `BetterAuth` mà chúng ta dự định sử dụng lại yêu cầu một cấu trúc cơ sở dữ liệu rất cụ thể (ví dụ: bảng `user` số ít với các trường nhất định như `id`, `email`, `password_hash`).

Để khắc phục điều này, chúng ta cần cập nhật tệp `Spec.md` với thông tin chính xác từ tài liệu của BetterAuth. Đây là một ví dụ tuyệt vời về việc cung cấp **ngữ cảnh bổ sung** để tinh chỉnh yêu cầu.

**Các bước thực hiện:**

1.  **Tìm tài liệu liên quan:** Truy cập tài liệu của thư viện BetterAuth và tìm phần mô tả cấu trúc cơ sở dữ liệu mà nó yêu cầu.
2.  **Sao chép thông tin:** Sao chép nội dung đó (tốt nhất là ở định dạng Markdown hoặc văn bản thuần túy) để sử dụng làm ngữ cảnh.
3.  **Bổ sung vào Prompt:** Thêm hướng dẫn cập nhật tệp và cung cấp tài liệu BetterAuth làm ngữ cảnh.

> [!TIP]
> **Sử dụng thẻ XML cho ngữ cảnh dài:** Khi chèn một đoạn văn bản dài, một thông báo lỗi, hoặc một bài viết từ tài liệu, việc sử dụng các thẻ XML (ví dụ: `<documentation>`, `</documentation>`) có thể giúp LLM dễ dàng xác định nơi thông tin ngữ cảnh của bạn bắt đầu và kết thúc. Điều này không bắt buộc nhưng có thể cải thiện khả năng phân tích của mô hình, đặc biệt là khi ngữ cảnh rất dài và phức tạp.

**Ví dụ Prompt hoàn chỉnh để cập nhật đặc tả:**

```markdown
Bạn là một kiến trúc sư phần mềm đang xem xét tài liệu đặc tả.
Chúng ta đang xây dựng một ứng dụng được mô tả trong @Spec.md.

Vui lòng xem xét và cập nhật tệp @Spec.md.
Cụ thể, hãy điều chỉnh phần về cấu trúc cơ sở dữ liệu, đặc biệt là các bảng liên quan đến người dùng và xác thực.
Chúng ta đang sử dụng thư viện xác thực BetterAuth, và nó yêu cầu một cấu trúc bảng cụ thể.
Sử dụng tài liệu BetterAuth dưới đây để đảm bảo rằng phần mô tả cơ sở dữ liệu trong Spec.md phản ánh chính xác yêu cầu của BetterAuth.

<BetterAuth_Documentation>
# BetterAuth Database Schema Requirements

## `user` table
- `id` (UUID, Primary Key, NOT NULL)
- `email` (TEXT, UNIQUE, NOT NULL)
- `password_hash` (TEXT, NOT NULL)
- `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- `updated_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

## `session` table
- `id` (UUID, Primary Key, NOT NULL)
- `user_id` (UUID, Foreign Key to `user.id`, NOT NULL)
- `expires_at` (TIMESTAMP, NOT NULL)
- `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

... (Các bảng và trường khác từ tài liệu BetterAuth, nếu có)
</BetterAuth_Documentation>
```

Trong prompt này:

*   `@Spec.md`: Ngữ cảnh tệp chính, Claude Code sẽ đọc và chỉnh sửa tệp này.
*   Các câu lệnh đầu tiên: Hướng dẫn tổng thể và mục tiêu cập nhật.
*   `<BetterAuth_Documentation> ... </BetterAuth_Documentation>`: Ngữ cảnh bổ sung, cung cấp chi tiết về cấu trúc cơ sở dữ liệu BetterAuth mà Claude Code cần tuân theo. Tên thẻ XML có thể tùy chỉnh để tăng tính rõ ràng.

**Quy trình làm việc với Claude Code (CLI):**

1.  **Chuẩn bị Prompt:** Viết prompt của bạn với các hướng dẫn và ngữ cảnh cần thiết.
2.  **Gửi Prompt:** Gửi prompt đến Claude Code thông qua giao diện dòng lệnh. Bạn có thể nhấn Enter để gửi, hoặc sử dụng các phím tắt (ví dụ: `Shift + Tab` trong một số thiết lập) để chuyển sang chế độ "accept edits on" nếu bạn muốn Claude Code tự động áp dụng các thay đổi mà nó đề xuất trực tiếp vào các tệp.
3.  **Claude Code xử lý:** Claude Code sẽ đọc tệp `Spec.md`, phân tích ngữ cảnh từ tài liệu BetterAuth, và thực hiện các thay đổi được yêu cầu.
4.  **Kiểm tra và xác nhận:** Sau khi Claude Code hoàn tất, hãy kiểm tra tệp `Spec.md` đã được cập nhật để đảm bảo mọi thứ chính xác và phù hợp với yêu cầu.

Sau bước này, tệp `Spec.md` của chúng ta sẽ được định dạng chuẩn và chứa thông tin chính xác về cấu trúc cơ sở dữ liệu theo yêu cầu của BetterAuth. Đây là một nền tảng vững chắc để tiếp tục các bước phát triển ứng dụng tiếp theo với Claude Code.

## 3. Nâng Tầm Phát Triển AI: Từ Claude Code đến Hệ Thống Agentic và Vibe Coding

Mặc dù Claude Code cung cấp một giao diện mạnh mẽ để áp dụng kỹ thuật prompt và ngữ cảnh, thế giới của phát triển phần mềm với AI đang tiến xa hơn nữa với các hệ thống AI agentic. Việc hiểu rõ cách Claude Code hoạt động sẽ là bước đệm vững chắc để bạn tiếp cận và làm chủ các công cụ tiên tiến hơn như Antigravity IDE.

### 3.1. Sự Tiến Hóa của Lập Trình AI: Từ Prompt Tường Minh đến Agent Tự Động

Ban đầu, lập trình AI chủ yếu xoay quanh việc viết các prompt cực kỳ chi tiết, từng bước một, hướng dẫn mô hình ngôn ngữ thực hiện các nhiệm vụ nhỏ. Claude Code là một ví dụ điển hình của cách tiếp cận này, nơi bạn cung cấp prompt, tham chiếu tệp (`@`), và mong đợi một đầu ra trực tiếp.

Tuy nhiên, với sự phát triển của các mô hình LLM và kiến trúc agent, chúng ta đang chuyển sang một kỷ nguyên mà AI không chỉ đơn thuần trả lời prompt mà còn có khả năng:

*   **Lập kế hoạch (Planning):** Chia nhỏ một mục tiêu lớn thành các bước nhỏ hơn.
*   **Thực thi (Execution):** Tự động chạy code, gọi các công cụ (tools) như trình duyệt, terminal, API.
*   **Phản hồi (Reflection):** Đánh giá kết quả của các bước thực thi và tự điều chỉnh kế hoạch nếu cần.
*   **Quản lý ngữ cảnh tự động (Automated Context Management):** Tự động tìm kiếm, đọc, và tổng hợp thông tin từ nhiều nguồn (tệp, tài liệu, web) để tạo ngữ cảnh phù hợp cho các tác vụ con.

Đây là tiền đề cho các hệ thống AI agentic.

### 3.2. Giới Thiệu Antigravity IDE: Một Hệ Thống Agentic AI Tiên Phong

Antigravity IDE là một ví dụ minh họa cho hệ thống AI agentic siêu việt mà bạn đang trực tiếp sử dụng trong khóa học này. Không giống như việc bạn phải tự tay gõ prompt và tham chiếu tệp cho Claude Code, Antigravity hoạt động ở một cấp độ trừu tượng cao hơn. Nó là một hệ thống agent có khả năng:

*   **Tự động chạy script ngầm:** Khi bạn đưa ra một yêu cầu, Antigravity có thể tự động viết và chạy các script (ví dụ: Python, JavaScript) để thực hiện các tác vụ như phân tích code, chạy test, hoặc thực hiện các thao tác file system.
*   **Gọi sub-agent trình duyệt:** Nó có thể khởi chạy một trình duyệt ảo, điều hướng web, đọc tài liệu từ các trang web (ví dụ: tài liệu của BetterAuth), hoặc tương tác với các giao diện web để thu thập thông tin hoặc thực hiện hành động.
*   **Đọc và ghi file tự động:** Antigravity không chỉ đọc tệp khi bạn `@` tham chiếu mà còn chủ động quét, phân tích cấu trúc dự án, và ghi các thay đổi vào nhiều tệp khác nhau (code, cấu hình, tài liệu) theo yêu cầu.
*   **Lập kế hoạch tự động:** Với một mục tiêu cấp cao, Antigravity sẽ tự động xây dựng một kế hoạch thực thi, bao gồm các bước cần thiết, các công cụ cần sử dụng, và cách quản lý ngữ cảnh cho từng bước.

Mục tiêu của Antigravity là giảm gánh nặng "prompt engineering" chi tiết cho người dùng, cho phép họ tập trung vào "ý định" (intent) thay vì "cách thức" (how).

### 3.3. Vibe Coding: Tư Duy Lập Trình Với AI Agentic

"Vibe Coding" là một tư duy lập trình mới, được phát triển để tối ưu hóa tương tác với các hệ thống AI agentic như Antigravity IDE. Nó khác biệt đáng kể so với kỹ thuật prompt truyền thống:

*   **Từ "Hướng dẫn cụ thể" sang "Ý định cấp cao":** Thay vì cung cấp từng bước chi tiết, Vibe Coding tập trung vào việc truyền đạt mục tiêu cuối cùng, "cảm giác" hoặc "linh hồn" của tính năng cần xây dựng. Bạn cung cấp một "vibe" (ý niệm tổng thể) về những gì bạn muốn, và hệ thống agent sẽ tự mình suy luận các bước thực thi chi tiết.
*   **Từ "Chỉ định ngữ cảnh" sang "Tin tưởng AI thu thập ngữ cảnh":** Thay vì phải tự tay `@` tham chiếu từng tệp, bạn tin tưởng rằng hệ thống agent sẽ tự động tìm kiếm, đọc, và tổng hợp các tệp liên quan, tài liệu, và thông tin web để tạo ngữ cảnh phù hợp cho từng tác vụ con trong kế hoạch của nó.
*   **Từ "Kiểm soát từng bước" sang "Giám sát và tinh chỉnh":** Vai trò của bạn chuyển từ người điều khiển từng lệnh sang người giám sát quá trình, cung cấp phản hồi tinh chỉnh khi cần thiết, hoặc điều chỉnh "vibe" ban đầu nếu kết quả không như mong đợi.

**Ví dụ về Vibe Coding so với Prompt Engineering truyền thống:**

*   **Prompt Engineering (với Claude Code):**
    ```
    Bạn là một lập trình viên Next.js.
    Chúng ta đang sử dụng @Spec.md và thư viện BetterAuth.
    Tạo một file `src/app/auth/login/page.tsx` và một `src/lib/auth.ts`.
    Trong `page.tsx`, tạo form đăng nhập với email và password.
    Trong `auth.ts`, viết hàm `loginUser` sử dụng BetterAuth để xác thực.
    Đảm bảo kiểm tra lỗi và hiển thị thông báo.
    ```
    (Bạn phải chỉ rõ từng file, từng hàm, từng bước).

*   **Vibe Coding (với Antigravity IDE):**
    ```
    "Implement the user authentication flow for the note-taking app using BetterAuth as described in the Spec.md. Focus on the login and registration pages first, ensuring secure password handling and session management."
    ```
    (Bạn đưa ra một mục tiêu tổng thể. Antigravity sẽ tự động: đọc `Spec.md`, tìm tài liệu BetterAuth, tạo các file cần thiết, viết code cho form, logic xác thực, xử lý session, kiểm tra lỗi, và thậm chí chạy các bài test).

### 3.4. Kỹ Thuật Prompt và Ngữ Cảnh trong Một Thế Giới Agentic

Mặc dù Vibe Coding giảm bớt sự cần thiết của các prompt chi tiết, các nguyên tắc cơ bản của kỹ thuật prompt và ngữ cảnh vẫn cực kỳ quan trọng, nhưng được áp dụng ở một cấp độ khác:

*   **Prompt Engineering Agentic:** Prompt ban đầu của bạn cho Antigravity (cái "vibe") không chỉ là một yêu cầu đơn thuần. Nó là một **hướng dẫn chiến lược** cho toàn bộ hệ thống agent. Một prompt Vibe Coding rõ ràng, súc tích, và có mục tiêu sẽ giúp agent:
    *   Xác định mục tiêu cuối cùng một cách chính xác.
    *   Lập kế hoạch hiệu quả hơn, phân chia nhiệm vụ thành các bước logic.
    *   Tạo ra các sub-prompt nội bộ chất lượng cao cho các tác vụ con của chính nó.
    *   Ưu tiên các ràng buộc và yêu cầu quan trọng.

*   **Automated Context Engineering:** Đây là nơi các hệ thống agentic thực sự tỏa sáng. Thay vì bạn phải tự tay cung cấp ngữ cảnh, Antigravity sẽ:
    *   **Tự động quét và lập chỉ mục:** Nó sẽ quét toàn bộ thư mục dự án, đọc các tệp code, cấu hình, và tài liệu để xây dựng một "bản đồ" ngữ cảnh nội bộ.
    *   **Tự động tìm kiếm và tổng hợp:** Khi cần thông tin về BetterAuth, nó sẽ không chờ bạn dán tài liệu. Nó sẽ tự động sử dụng sub-agent trình duyệt để tìm kiếm tài liệu BetterAuth trên web, đọc nó, và tổng hợp thông tin cần thiết vào ngữ cảnh cho tác vụ hiện tại.
    *   **Duy trì trạng thái:** Antigravity duy trì trạng thái của dự án, các thay đổi đã thực hiện, và kết quả của các bước trước đó, đảm bảo ngữ cảnh luôn được cập nhật và liên quan.
    *   **Ưu tiên ngữ cảnh:** Nó sử dụng các kỹ thuật tiên tiến để xác định phần ngữ cảnh nào là quan trọng nhất cho từng tác vụ cụ thể, tránh "nhiễu" một cách tự động.

### 3.5. Ứng Dụng Tư Duy Vibe Coding vào Antigravity IDE

Để tận dụng tối đa Antigravity IDE, bạn nên:

1.  **Bắt đầu với một "Vibe" rõ ràng:** Đưa ra một mục tiêu cấp cao, mô tả tính năng bạn muốn, và những yêu cầu quan trọng nhất.
2.  **Cung cấp điểm khởi đầu:** Đảm bảo `Spec.md` hoặc các tệp cấu hình ban đầu có sẵn. Antigravity sẽ tự động đọc chúng.
3.  **Tin tưởng vào khả năng tự động hóa:** Hãy để Antigravity tự lập kế hoạch, tìm kiếm thông tin, viết code, và chạy các công cụ.
4.  **Giám sát và phản hồi:** Theo dõi tiến trình của Antigravity. Nếu có điều gì không đúng hướng, hãy đưa ra phản hồi rõ ràng để nó tự điều chỉnh. Phản hồi của bạn lại trở thành một phần ngữ cảnh quan trọng cho các bước tiếp theo.
5.  **Tư duy về kết quả, không phải quy trình:** Tập trung vào "what" (cái gì) thay vì "how" (làm thế nào). Antigravity sẽ lo phần "how".

Việc làm chủ kỹ thuật prompt và ngữ cảnh với Claude Code là bước đầu tiên quan trọng. Nó giúp bạn hiểu được cách các LLM xử lý thông tin. Khi chuyển sang Antigravity IDE và Vibe Coding, bạn sẽ thấy rằng các nguyên tắc này vẫn được áp dụng, nhưng được tự động hóa và mở rộng ra, cho phép bạn phát triển phần mềm nhanh chóng và hiệu quả hơn rất nhiều.

## 4. Tóm Tắt Phần 4

*   **Kỹ thuật Prompt và Kỹ thuật Ngữ cảnh** là nền tảng để tương tác hiệu quả với các mô hình ngôn ngữ lớn (LLM), đảm bảo đầu ra chất lượng cao và giảm thiểu "ảo giác".
*   **Prompt Engineering** là nghệ thuật thiết kế các hướng dẫn rõ ràng, cụ thể (bao gồm vai trò, định dạng, ví dụ, ràng buộc) để AI thực hiện nhiệm vụ mong muốn. Nguyên tắc "Đầu vào tốt cho ra đầu ra tốt" là kim chỉ nam.
*   **Context Engineering** là quá trình cung cấp thông tin nền tảng và dữ liệu liên quan để AI hiểu rõ hơn về nhiệm vụ, giảm thiểu sự mơ hồ và tăng độ chính xác. Việc sử dụng ký hiệu `@` để tham chiếu tệp trong Claude Code là một ví dụ điển hình về cung cấp ngữ cảnh chủ động.
*   **Claude Code** là công cụ AI CLI của Anthropic, cho phép lập trình viên áp dụng trực tiếp kỹ thuật prompt và ngữ cảnh để hỗ trợ quá trình phát triển phần mềm.
*   Khi bắt đầu một dự án mới với Claude Code, việc cung cấp một **tài liệu đặc tả kỹ thuật** (ví dụ: `Spec.md`) là rất quan trọng để mô tả ứng dụng và thiết lập ngữ cảnh ban đầu.
*   **Cung cấp ngữ cảnh bổ sung** (ví dụ: tài liệu API, hướng dẫn cụ thể) có thể được thực hiện bằng cách dán trực tiếp vào prompt, và nên sử dụng **thẻ XML** (ví dụ: `<documentation>...</documentation>`) để cấu trúc và giúp AI dễ dàng nhận diện các khối thông tin dài.
*   **Antigravity IDE** là một hệ thống AI agentic tiên phong, có khả năng tự động lập kế hoạch, thực thi script, gọi sub-agent trình duyệt, đọc/ghi file và quản lý ngữ cảnh một cách tự động.
*   **Vibe Coding** là tư duy lập trình mới cho các hệ thống agentic, tập trung vào việc truyền đạt "ý định cấp cao" thay vì các hướng dẫn chi tiết. Nó giúp lập trình viên tập trung vào mục tiêu cuối cùng, để AI agent xử lý các bước thực thi cụ thể.
*   Trong một hệ thống agentic như Antigravity, kỹ thuật prompt được nâng cấp thành **Prompt Engineering Agentic** (hướng dẫn chiến lược), và kỹ thuật ngữ cảnh được tự động hóa thành **Automated Context Engineering** (AI tự động thu thập và tổng hợp ngữ cảnh).

Việc nắm vững các nguyên tắc này sẽ là chìa khóa để bạn không chỉ sử dụng hiệu quả Claude Code mà còn sẵn sàng làm việc với các hệ thống AI agentic tiên tiến, định hình tương lai của lập trình.

<!-- REVIEWED_BY_AGENT -->
