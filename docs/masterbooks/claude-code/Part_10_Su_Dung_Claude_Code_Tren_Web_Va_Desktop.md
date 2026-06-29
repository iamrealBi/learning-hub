# Bài 10: Tối Ưu Hóa Quy Trình Phát Triển với Claude Code: Từ CLI đến Giao Diện Đồ Họa và Tích Hợp Agentic

Trong kỷ nguyên phát triển phần mềm hiện đại, việc tối ưu hóa quy trình làm việc và tận dụng sức mạnh của trí tuệ nhân tạo để tăng năng suất là vô cùng quan trọng. Sự xuất hiện của các công cụ AI agentic như Antigravity IDE đã định hình lại cách chúng ta tiếp cận lập trình, cho phép hệ thống tự động thực thi các tác vụ phức tạp, từ việc chạy script ngầm, gọi subagent trình duyệt, đọc ghi file đến lập kế hoạch tự động. Trong bối cảnh đó, việc hiểu và tích hợp các công cụ AI khác như Claude Code trở nên thiết yếu để hoàn thiện một quy trình "Vibe Coding" mạnh mẽ.

Phần này sẽ đưa bạn khám phá sâu hơn về cách sử dụng công cụ Claude Code – một công cụ CLI AI mạnh mẽ từ Anthropic – không chỉ trên giao diện dòng lệnh (CLI) quen thuộc mà còn thông qua nền tảng web từ xa và ứng dụng desktop mạnh mẽ. Chúng ta sẽ tìm hiểu cách tích hợp Claude Code vào quy trình làm việc hàng ngày, từ việc giao phó các tác vụ phát triển cho môi trường từ xa đến việc tận dụng các tính năng nâng cao của ứng dụng desktop để xem trước, kiểm tra và tinh chỉnh code một cách hiệu quả. Mục tiêu là giúp bạn trở thành người dùng thành thạo Claude Code, có khả năng khai thác tối đa tiềm năng của nó trong mọi tình huống phát triển, đồng thời hiểu rõ cách nó bổ trợ cho hệ thống Antigravity IDE mà bạn đang sử dụng, tạo nên một trải nghiệm Vibe Coding liền mạch và siêu việt.

## I. Claude Code Web: Mở Rộng Sức Mạnh Agentic Ra Môi Trường Từ Xa

Claude Code không chỉ giới hạn ở máy tính cục bộ của bạn. Với khả năng tích hợp qua web, bạn có thể giao phó các tác vụ phát triển cho môi trường từ xa, giải phóng tài nguyên máy tính cá nhân và cho phép bạn tiếp tục công việc ngay cả khi bạn không ở trước máy tính. Đây là một mảnh ghép quan trọng trong hệ sinh thái Vibe Coding, đặc biệt khi bạn cần xử lý các tác vụ yêu cầu tài nguyên lớn hoặc cần chạy song song.

### 1. Tại Sao Lại Cần Claude Code Web?

Việc chuyển giao (offload) các tác vụ phát triển sang môi trường Claude Code Web mang lại nhiều lợi ích chiến lược, đặc biệt khi kết hợp với một hệ thống agentic như Antigravity IDE:

*   **Giải phóng tài nguyên cục bộ:** Các tác vụ nặng về tính toán (ví dụ: tối ưu hóa hình ảnh, biên dịch dự án lớn, chạy kiểm thử tích hợp mở rộng) hoặc cần thời gian dài có thể chạy trên máy chủ của Claude Code. Điều này giúp máy tính cục bộ của bạn rảnh để làm các công việc khác, duy trì hiệu suất cho Antigravity IDE hoặc các tác vụ phát triển tức thì.
*   **Làm việc không gián đoạn và song song:** Bạn có thể khởi tạo một tác vụ trên Claude Code Web và rời khỏi máy tính. Claude Code sẽ tiếp tục làm việc và thông báo khi hoàn thành. Điều này cho phép bạn khởi chạy nhiều tác vụ cùng lúc mà không ảnh hưởng đến hiệu suất máy tính cục bộ, một lợi thế lớn so với việc chỉ chạy các agent cục bộ của Antigravity.
*   **Tích hợp Git liền mạch:** Lý tưởng cho các dự án lưu trữ trên GitHub, Claude Code có thể trực tiếp truy cập, sửa đổi, kiểm tra và tạo Pull Request (PR). Điều này tự động hóa một phần quan trọng của quy trình CI/CD, giải phóng bạn khỏi các thao tác Git thủ công.
*   **Kỹ thuật ẩn sau (Under the Hood): Môi trường ảo hóa tạm thời.** Khi bạn gửi một tác vụ đến Claude Code Web, hệ thống sẽ tự động cấp phát một môi trường phát triển tạm thời (ephemeral development environment) trên các máy chủ của Anthropic. Môi trường này thường là một container hoặc một máy ảo (VM) được cấu hình sẵn với các công cụ phát triển cần thiết, có khả năng truy cập vào kho lưu trữ Git của bạn. Sau khi tác vụ hoàn thành, môi trường này có thể được giải phóng, đảm bảo tính cô lập và hiệu quả tài nguyên. Điều này khác biệt với Antigravity IDE, vốn chủ yếu hoạt động trong môi trường cục bộ của bạn, tận dụng tài nguyên máy tính của bạn.

### 2. Thiết Lập Môi Trường Claude Code Web

Để sử dụng tính năng Claude Code Web, bạn cần có gói đăng ký phù hợp (thường là Pro, Max, Teams, hoặc Enterprise) và thực hiện các bước thiết lập sau:

1.  **Truy cập Giao diện Web:** Mở trình duyệt và truy cập `claude.ai/code`. Đây là cổng chính để quản lý các môi trường từ xa và theo dõi tác vụ.
2.  **Kết nối với GitHub:**
    *   Claude Code cần quyền truy cập vào các kho lưu trữ GitHub của bạn để có thể sao chép, sửa đổi và tạo Pull Request. Bạn sẽ được yêu cầu kết nối tài khoản Claude với GitHub thông qua quy trình OAuth.
    *   **Quản lý quyền truy cập:** Chọn cụ thể các kho lưu trữ mà bạn muốn Claude Code có thể truy cập và ủy quyền. Đây là một bước bảo mật quan trọng, đảm bảo rằng Claude Code chỉ làm việc với những dự án được phép, tương tự như cách bạn cấp quyền cho một agent của Antigravity truy cập vào các thư mục cục bộ cụ thể.
3.  **Tạo Môi Trường Claude Code Web:**
    *   Bạn cần tạo một môi trường phát triển trên Claude Code Web. Đặt tên cho môi trường này (có thể giữ tên mặc định để tiện quản lý).
    *   **Cấu hình quyền truy cập mạng:** Đây là một cài đặt quan trọng với các cân nhắc về bảo mật và chức năng:
        *   **Trusted Setup:** Quyền truy cập mạng hạn chế, thường đủ cho hầu hết các tác vụ code thông thường (ví dụ: cài đặt thư viện, chạy kiểm thử nội bộ). Đây là lựa chọn an toàn hơn.
        *   **Full Internet Access:** Cung cấp quyền truy cập internet đầy đủ. Cần thiết nếu ứng dụng của bạn cần gửi yêu cầu API đến các dịch vụ bên ngoài, tải tài nguyên từ internet, hoặc các thử nghiệm yêu cầu kết nối internet. Tuy nhiên, đi kèm với rủi ro bảo mật cao hơn do môi trường có thể truy cập bất kỳ tài nguyên nào trên web.

### 3. Vận Hành Claude Code Web: Các Phương Thức Tương Tác

Bạn có thể tương tác với Claude Code Web theo hai cách chính, cho phép linh hoạt tích hợp vào quy trình Vibe Coding của bạn.

#### a. Từ Giao Diện Web Trực Tiếp

Sau khi thiết lập môi trường, bạn có thể sử dụng giao diện web để khởi tạo tác vụ một cách trực quan:

*   Chọn kho lưu trữ (repository) và nhánh (branch) bạn muốn làm việc.
*   Chọn môi trường Claude Code Web đã tạo.
*   Mô tả tác vụ của bạn bằng ngôn ngữ tự nhiên (ví dụ: "change the main color from blue to an elegant purple").
*   Bạn có thể chọn mô hình AI cụ thể và thêm hình ảnh làm ngữ cảnh nếu cần.
*   Nhấn nút để khởi chạy tác vụ.

#### b. Điều Khiển Từ CLI Claude Code Cục Bộ

Để tích hợp liền mạch hơn vào quy trình làm việc cục bộ của Vibe Coding, bạn có thể điều khiển các tác vụ Claude Code Web ngay từ CLI Claude Code trên máy tính của mình. Điều này đặc biệt hữu ích khi bạn muốn kết hợp sức mạnh của Antigravity IDE với khả năng xử lý từ xa của Claude Code.

*   **Chọn môi trường từ xa:** Sử dụng lệnh `/remoteenv` để chọn môi trường Claude Code Web mà bạn muốn sử dụng cho các tác vụ tiếp theo.
    ```bash
    claude /remoteenv my-remote-dev-env
    ```
*   **Chạy tác vụ từ xa:**
    *   **Sử dụng ký hiệu `&` (chạy nền):** Thêm ký hiệu `&` vào cuối prompt của bạn để ra lệnh cho Claude Code thực thi tác vụ trong môi trường từ xa và chạy ở chế độ nền. Ví dụ:
        ```bash
        change the main color from blue to an elegant purple &
        ```
        Điều này rất phù hợp với Vibe Coding, nơi bạn có thể khởi động một tác vụ và tiếp tục tập trung vào việc khác mà không bị gián đoạn.

    *   **Sử dụng cờ `--remote` (hoặc `-r`):** Đây là cách rõ ràng hơn để chỉ định tác vụ chạy từ xa. Ví dụ:
        ```bash
        claude --remote "Change the main color from blue to an elegant purple across all components."
        ```

    > [!NOTE]
    > Bạn có thể chạy nhiều tác vụ song song trên Claude Code Web bằng cách mô tả chúng và thêm `&` vào cuối mỗi tác vụ, hoặc sử dụng cờ `--remote` cho từng tác vụ riêng biệt. Điều này mở ra khả năng xử lý đa nhiệm mạnh mẽ, tương tự như việc Antigravity IDE có thể quản lý nhiều subagent cùng lúc.

#### c. Theo Dõi Tiến Độ và Quy Trình Tự Động Hóa

Khi một tác vụ được gửi đến Claude Code Web, bạn sẽ nhận được một liên kết đến phiên làm việc từ xa. Tại đó, bạn có thể theo dõi tiến độ, xem các bước mà Claude Code đang thực hiện và nhận thông báo.

Quá trình làm việc của Claude Code Web bao gồm các bước tự động hóa sau:

1.  **Checkout Repository:** Claude Code sẽ sao chép kho lưu trữ và nhánh đã chọn vào môi trường tạm thời của nó.
2.  **Thực hiện thay đổi:** AI sẽ phân tích code và thực hiện các thay đổi cần thiết dựa trên prompt của bạn.
3.  **Chạy kiểm thử (nếu có):** Nếu dự án có các bài kiểm thử (unit tests, integration tests), Claude Code có thể chạy chúng trong môi trường từ xa để đảm bảo các thay đổi không gây ra lỗi và duy trì chất lượng code.
4.  **Tạo Commit và Pull Request:** Sau khi hoàn thành, Claude Code sẽ tạo một commit mới trên một nhánh riêng biệt (không đẩy trực tiếp vào nhánh chính của bạn) và mở một Pull Request (PR) trên GitHub. Điều này cho phép bạn xem xét kỹ lưỡng các thay đổi, thảo luận và hợp nhất (merge) chúng vào nhánh chính khi bạn hài lòng.

    > [!TIP]
    > **Vibe Coding & Antigravity IDE:** Luôn xem xét kỹ các Pull Request do Claude Code tạo ra trước khi hợp nhất. Mặc dù các hệ thống agentic như Claude Code và Antigravity IDE rất mạnh mẽ, việc "human in the loop" là không thể thiếu để duy trì chất lượng code, đảm bảo các thay đổi phù hợp với "vibe" tổng thể của dự án và tránh các lỗi không mong muốn. Antigravity IDE cũng có thể tạo ra các thay đổi code, và việc xem xét các thay đổi đó trước khi đẩy lên Git là một nguyên tắc vàng.

#### d. Kết Hợp Các Chế Độ Làm Việc: Chiến Lược Vibe Coding Nâng Cao

Một trong những tính năng mạnh mẽ là khả năng kết hợp các chế độ làm việc, cho phép bạn tối ưu hóa quy trình Vibe Coding của mình. Ví dụ, bạn có thể bắt đầu một tác vụ cục bộ ở chế độ `plan` để Claude Code giúp bạn lên kế hoạch chi tiết, sau đó, khi kế hoạch đã sẵn sàng, bạn có thể chuyển giao việc triển khai cho Claude Code Web:

```bash
# Bắt đầu tác vụ cục bộ ở chế độ plan để Claude Code giúp bạn lên kế hoạch.
# Điều này giữ cho "vibe" của bạn tập trung vào chiến lược.
claude --plan "Add a confirmation dialog for public sharing in the app."

# Claude Code sẽ hỏi bạn một số câu hỏi và tạo ra một kế hoạch.
# Sau khi kế hoạch hoàn tất (hoặc bạn đã sửa đổi kế hoạch), bạn có thể ngắt phiên cục bộ (ví dụ: Esc).

# Sau đó, tiếp tục tác vụ nhưng chuyển sang thực thi từ xa, giải phóng tài nguyên cục bộ.
# Hoặc nếu phiên trước vẫn đang hoạt động và bạn muốn chuyển nó sang remote:
# Trong phiên đang chạy, gõ: "implement the plan &"
# Hoặc khởi tạo phiên mới từ CLI cục bộ để thực thi từ xa:
claude --remote -c "Implement the plan based on our previous discussion."
```
Đây là một ví dụ điển hình về Vibe Coding: bạn tận dụng AI để lập kế hoạch chiến lược, sau đó sử dụng khả năng agentic từ xa để thực thi, cho phép bạn chuyển sang nhiệm vụ khác mà không bị gián đoạn.

> [!NOTE]
> Tính năng Claude Code Web, đặc biệt khi tương tác từ CLI cục bộ, có thể vẫn đang trong giai đoạn thử nghiệm (beta hoặc research preview). Do đó, bạn có thể gặp phải một số lỗi hoặc hành vi không mong muốn. Nếu gặp vấn đề, hãy thử khởi tạo phiên từ giao diện web trực tiếp.

## II. Ứng Dụng Claude Code Desktop: Giao Diện Trực Quan cho Quy Trình Phát Triển Agentic

Bên cạnh CLI và tích hợp IDE, ứng dụng desktop của Claude Code cung cấp một giao diện người dùng đồ họa (GUI) toàn diện, kết hợp nhiều khả năng của Claude vào một nơi duy nhất. Điều này mang lại trải nghiệm làm việc trực quan và mạnh mẽ, đặc biệt hữu ích cho Vibe Coding khi bạn cần phản hồi trực quan và tương tác sâu hơn với code và UI.

### 1. Giới Thiệu và Cài Đặt

Để bắt đầu, bạn cần tải và cài đặt ứng dụng Claude Code Desktop từ trang web chính thức của Anthropic. Ứng dụng này không chỉ dành riêng cho việc lập trình mà còn gói gọn nhiều khả năng khác của Claude AI:

*   **Chat:** Chế độ trò chuyện tổng quát, tương tự như các chatbot AI khác nhưng sử dụng các mô hình Claude mạnh mẽ. Nó không liên quan trực tiếp đến lập trình mà là một trợ lý trò chuyện chung.
*   **Cowork:** Một trợ lý làm việc có mục đích chung, được thiết kế để hỗ trợ các tác vụ văn phòng như tạo slide, viết báo cáo, phân tích dữ liệu, v.v.
*   **Claude Code:** Đây là trọng tâm của khóa học này và là nơi bạn sẽ tương tác với AI để phát triển phần mềm. Chế độ này được tối ưu hóa cho các tác vụ lập trình và là nơi bạn sẽ thực hiện Vibe Coding với Claude.

Trong khi Antigravity IDE là một hệ thống agentic toàn diện, tự chạy script và quản lý quy trình, Claude Code Desktop cung cấp một giao diện GUI thân thiện hơn để tương tác với mô hình Claude, đặc biệt là trong việc cung cấp ngữ cảnh trực quan và xem xét thay đổi.

### 2. Quản Lý Phiên Làm Việc (Sessions) và Dự Án

Giao diện ứng dụng Claude Code Desktop được thiết kế để quản lý nhiều phiên làm việc (sessions) một cách hiệu quả, giúp bạn duy trì "vibe" của từng dự án:

*   **Quản lý phiên theo dự án:** Các phiên làm việc được tổ chức theo các dự án lập trình khác nhau mà bạn đang thực hiện. Điều này giúp bạn dễ dàng chuyển đổi ngữ cảnh và theo dõi tiến độ công việc cho từng dự án mà không bị nhầm lẫn.
*   **Hỗ trợ Claude Code Web Sessions:** Một điểm đáng chú ý là bạn có thể khởi tạo và quản lý các phiên Claude Code Web (từ xa) ngay từ trong ứng dụng desktop, mang lại sự linh hoạt tối đa. Điều này củng cố ý tưởng về một hệ sinh thái agentic nơi các công cụ khác nhau bổ trợ lẫn nhau.

### 3. Khởi Tạo Một Phiên Mới: Tối Ưu Hóa Ngữ Cảnh cho AI

Khi bạn muốn bắt đầu một tác vụ mới, bạn có thể nhấp vào nút "New Session" và cấu hình các tùy chọn sau để tối ưu hóa ngữ cảnh cho AI:

*   **Chọn dự án:** Chọn thư mục dự án mà bạn muốn Claude Code làm việc. Đây là bước đầu tiên để AI có thể "hiểu" bối cảnh code của bạn.
*   **Loại tác vụ:** Quyết định xem tác vụ sẽ được thực thi **cục bộ** trên máy tính của bạn hay trên môi trường **Claude Code Web** từ xa. Lựa chọn này phụ thuộc vào yêu cầu tài nguyên và tính liên tục của công việc.
*   **Sử Dụng Git Worktree (Tùy chọn):**
    *   **Git Worktree là gì?** Đây là một tính năng mạnh mẽ của Git cho phép bạn có nhiều thư mục làm việc được liên kết với cùng một kho lưu trữ Git nhưng ở các nhánh hoặc trạng thái khác nhau. Về cơ bản, nó tạo ra một bản sao cục bộ của kho lưu trữ của bạn (dựa trên cùng một nhánh đã được checkout) vào một đường dẫn khác trên hệ thống của bạn, mà không cần tạo một nhánh Git mới.
    *   **Lợi ích trong phát triển Agentic và Vibe Coding:** Tính năng này cực kỳ hữu ích khi bạn muốn Claude Code (hoặc bất kỳ AI agent nào khác) làm việc trên một bản sao độc lập của dự án mà không ảnh hưởng trực tiếp đến thư mục làm việc chính hoặc nhánh hiện tại của bạn. Bạn có thể có nhiều phiên Claude Code làm việc trên các worktree khác nhau của cùng một nhánh, cho phép AI thử nghiệm các thay đổi mà không gây rủi ro cho code gốc. Sau khi hoàn thành công việc, bạn có thể dễ dàng hợp nhất các thay đổi từ worktree đó vào nhánh chính của mình. Nếu bạn bật tùy chọn này, Claude Code sẽ tự động tạo một worktree mới cho mỗi phiên làm việc, đảm bảo môi trường làm việc cô lập và an toàn.
*   **Chọn nhánh:** Chọn nhánh Git mà bạn muốn Claude Code làm việc.

### 4. Tạo Prompt và Cung Cấp Ngữ Cảnh Toàn Diện (Vibe Prompting)

Tương tự như CLI, bạn sẽ nhập prompt mô tả tác vụ của mình. Tuy nhiên, ứng dụng desktop cung cấp các cách trực quan hơn để cung cấp ngữ cảnh, giúp AI "hiểu rõ vibe" bạn muốn:

*   **Tham chiếu file:** Sử dụng ký hiệu `@` để tham chiếu trực tiếp đến một file trong dự án của bạn (ví dụ: `@src/pages/EditPage.vue`). Claude Code sẽ sử dụng nội dung của file đó làm ngữ cảnh, giúp AI tập trung vào các đoạn code liên quan.
*   **Thêm file hoặc hình ảnh:** Nhấp vào biểu tượng `+` để thêm các file hoặc hình ảnh khác làm ngữ cảnh cho tác vụ. Đây là một điểm mạnh so với CLI, cho phép bạn cung cấp ngữ cảnh trực quan (ví dụ: một mockup UI) để AI hiểu rõ hơn về thiết kế mong muốn.
*   **Slash Commands:** Các lệnh slash quen thuộc từ CLI (ví dụ: `/skill <tên_skill>`) cũng có sẵn để kích hoạt các kỹ năng hoặc chức năng cụ thể, mở rộng khả năng của AI.
*   **Voice Dictation:** Nếu bạn thích, bạn có thể sử dụng tính năng đọc chính tả bằng giọng nói để nhập prompt, giúp quá trình Vibe Coding trở nên tự nhiên hơn.
*   **Quản lý quyền:** Bạn có thể cấu hình cách Claude Code yêu cầu quyền trước khi thực hiện các hành động:
    *   **Ask for permission:** Claude Code sẽ hỏi bạn trước khi thực hiện các hành động quan trọng (ví dụ: sửa đổi file, chạy lệnh). Đây là chế độ an toàn nhất.
    *   **Accept edits:** Tự động chấp nhận các thay đổi code (nhưng vẫn có thể hỏi cho các hành động nguy hiểm như xóa file).
    *   **Bypass permissions:** Không bao giờ hỏi quyền. Chế độ này làm cho Claude Code trở nên "agentic" hơn, tương tự như việc Antigravity IDE tự động thực thi các script. (Sẽ được cấu hình chi tiết hơn trong phần cài đặt).
*   **Chế độ Plan:** Bạn có thể chọn bắt đầu tác vụ ở chế độ `plan` để Claude Code giúp bạn lên kế hoạch trước khi thực hiện, một bước quan trọng trong Vibe Coding để đảm bảo AI đi đúng hướng.
*   **Chọn Model và Reasoning Effort:** Bạn có thể chọn mô hình Claude Code cụ thể và mức độ "nỗ lực suy luận" (reasoning effort) mà AI nên dành cho tác vụ. Mức độ nỗ lực cao hơn thường mang lại kết quả tốt hơn nhưng có thể tốn nhiều thời gian và tài nguyên hơn.

**Ví dụ thực tế về Vibe Prompting:**
Giả sử bạn muốn thay đổi độ rộng của một phần tử trong trang chỉnh sửa ghi chú của ứng dụng, đồng thời cung cấp một hình ảnh làm ngữ cảnh để AI hiểu rõ thiết kế tổng thể.
```
# Prompt trong ứng dụng desktop:
Change the width of the note editor on the 'Edit Note' page to take up the full width of its container,
as shown in the attached mockup.

# Thêm ngữ cảnh file và hình ảnh:
@src/pages/EditPage.vue
[Kéo và thả file mockup.png vào cửa sổ chat]
```
Sau khi cấu hình và gửi prompt, Claude Code sẽ bắt đầu làm việc và hiển thị kết quả trực tiếp trong ứng dụng desktop. So với Antigravity IDE, khả năng cung cấp ngữ cảnh hình ảnh trực quan này là một điểm mạnh của Claude Code Desktop cho các tác vụ liên quan đến UI/UX.

## III. Tính Năng Nâng Cao của Claude Code Desktop: Trực Quan Hóa và Tương Tác Agentic

Ứng dụng Claude Code Desktop không chỉ là một giao diện để chạy các tác vụ mà còn là một môi trường mạnh mẽ với nhiều công cụ giúp bạn hiểu, xem xét và tương tác với các thay đổi của AI, đặc biệt hữu ích cho quy trình Vibe Coding đòi hỏi phản hồi trực quan và lặp lại.

### 1. Quản Lý Đa Nhiệm và Đánh Giá Kết Quả

*   **Đa nhiệm:** Bạn có thể có nhiều phiên làm việc cùng lúc, thậm chí trên các dự án khác nhau, và dễ dàng chuyển đổi giữa chúng. Điều này giúp bạn duy trì năng suất cao, tương tự như việc quản lý nhiều tác vụ song song trong Antigravity IDE.
*   **Xem chi tiết đầu ra:** Khi một tác vụ hoàn thành, bạn có thể mở rộng phần đầu ra để xem danh sách các file đã được chỉnh sửa. Mở rộng thêm nữa sẽ hiển thị chi tiết `diff` (sự khác biệt) của từng file, cho bạn biết chính xác những dòng code nào đã được thêm, xóa hoặc sửa đổi. Đây là bước quan trọng để đánh giá "vibe" của thay đổi mà AI đã thực hiện.

### 2. Các Chế Độ Xem Chuyên Biệt cho Vibe Coding

Ở góc trên bên phải trong chế độ Claude Code, bạn sẽ tìm thấy các chế độ xem khác cung cấp cái nhìn sâu sắc hơn vào quá trình làm việc của AI và ứng dụng của bạn:

#### a. Plan Mode

Nếu bạn đã khởi tạo tác vụ ở chế độ `plan` hoặc Claude Code tự động tạo một kế hoạch, bạn có thể chuyển sang chế độ này để xem chi tiết các bước mà AI dự định thực hiện. Điều này giúp bạn hiểu rõ "tư duy" của Claude Code và can thiệp nếu cần, đảm bảo rằng kế hoạch của AI phù hợp với "vibe" và mục tiêu của bạn trước khi thực thi.

#### b. Tasks

Chế độ này hiển thị danh sách các tác vụ mà Claude Code đang cố gắng thực hiện trong phiên hiện tại. Nếu tác vụ đã hoàn thành, danh sách này có thể trống. Nó cung cấp một cái nhìn tổng quan về tiến trình của agent.

#### c. Preview Mode (Chế độ Xem Trước)

Đây là một tính năng cực kỳ hữu ích cho các nhà phát triển web và là cốt lõi của Vibe Coding khi làm việc với UI:

*   **Thiết lập:** Lần đầu tiên sử dụng trong một dự án, bạn cần cho Claude Code biết cách khởi động máy chủ phát triển (dev server) của ứng dụng. Ứng dụng sẽ tự động chạy một tác vụ để cố gắng tìm hiểu lệnh khởi động (ví dụ: `npm run dev`, `yarn start`). Thông tin này sẽ được lưu vào một file `launch.json` trong thư mục `.claude` của dự án. Sau khi thiết lập, Claude Code có thể tự động khởi động dev server cho bạn.
*   **Xem trước trực tiếp:** Bạn sẽ thấy một bản xem trước trực tiếp của ứng dụng web của mình ngay trong ứng dụng Claude Code Desktop. Điều này tạo ra một vòng lặp phản hồi cực kỳ nhanh, cho phép bạn đánh giá ngay lập tức các thay đổi về UI/UX.
*   **Tính năng tương tác:**
    *   **Chuyển đổi Dark/Light Mode:** Gửi tín hiệu đến ứng dụng để chuyển đổi giữa chế độ sáng và tối (nếu ứng dụng hỗ trợ), giúp bạn kiểm tra khả năng tương thích của UI.
    *   **Chuyển đổi Mobile/Desktop View:** Xem ứng dụng của bạn trông như thế nào trên các kích thước màn hình khác nhau, hỗ trợ phát triển responsive.
    *   **Chọn phần tử (Select Elements):** Đây là một tính năng đột phá và là một ví dụ tuyệt vời về Vibe Coding. Bạn có thể nhấp vào bất kỳ phần tử UI nào trên bản xem trước, và phần tử đó sẽ được chèn làm ngữ cảnh vào cửa sổ chat của bạn. Sau đó, bạn có thể tham chiếu trực tiếp đến nó trong prompt của mình. Ví dụ: "Make this selected button red." Điều này loại bỏ sự mơ hồ và cho phép bạn hướng dẫn Claude Code một cách cực kỳ chính xác. Bạn có thể chọn và thêm nhiều phần tử.

    > [!EXAMPLE]
    > Giả sử bạn chọn một nút "Submit" trên trang xem trước. Trong cửa sổ chat, bạn sẽ thấy một thẻ ngữ cảnh như `<element id="submit-button" ...>`. Sau đó, bạn có thể gõ prompt:
    > `Change the background color of the <element id="submit-button" ...> to a subtle green, and increase its font size by 2px.`
    >
    > **Vibe Coding & Antigravity IDE:** Mặc dù Antigravity IDE có khả năng gọi subagent trình duyệt để tương tác và nhận diện UI, tính năng "Select Elements" của Claude Code Desktop cho phép *bạn* trực tiếp chỉ định phần tử mà không cần phải mô tả bằng lời, tạo ra một kênh giao tiếp trực quan và hiệu quả hơn cho các tác vụ UI/UX cụ thể. Đây là sự bổ trợ hoàn hảo: Antigravity có thể tự động thực hiện các tác vụ phức tạp, còn Claude Code Desktop giúp bạn tinh chỉnh "vibe" giao diện một cách trực quan.

#### d. Session Logs

Hiển thị nhật ký từ máy chủ phát triển của bạn. Đây là nơi bạn có thể thấy các thông báo console, cảnh báo hoặc lỗi từ ứng dụng của bạn trong quá trình chạy, rất hữu ích cho việc gỡ lỗi.

#### e. Terminal

Một terminal tích hợp cho phép bạn chạy bất kỳ lệnh terminal nào trong ngữ cảnh dự án của mình, tương tự như terminal trong các IDE hiện đại như VS Code hoặc chính Antigravity IDE. Điều này cung cấp khả năng kiểm soát trực tiếp môi trường.

#### f. Diff Viewer (Trình xem Thay đổi)

Công cụ này cung cấp một cái nhìn tổng quan toàn diện về tất cả các file đã được Claude Code chỉnh sửa và những thay đổi cụ thể trong từng file.

*   **Xem xét thay đổi:** Đây là nơi lý tưởng để bạn xem xét mọi thay đổi mà Claude Code đã thực hiện.
*   **Thêm bình luận trực tiếp vào code:** Một tính năng mạnh mẽ là bạn có thể di chuột qua các dòng code trong trình xem diff và thêm bình luận trực tiếp vào đó. Bình luận này sau đó sẽ được tự động thêm vào cửa sổ chat làm ngữ cảnh, cho phép bạn chỉ ra chính xác vị trí mà bạn muốn Claude Code sửa đổi hoặc xem xét lại.

    > [!EXAMPLE]
    > Giả sử Claude Code đã thay đổi một giá trị CSS từ `10px` thành `12px`. Bạn có thể thêm bình luận vào dòng đó:
    > `> [!COMMENT]
    >   This value should be 10px, not 12px. Please revert to 10px to match the design system.`
    > Bình luận này sẽ được thêm vào ngữ cảnh chat, và bạn có thể tiếp tục prompt Claude Code dựa trên đó.
    >
    > **Vibe Coding Iteration:** Tính năng này là một công cụ mạnh mẽ cho việc lặp lại trong Vibe Coding. Thay vì phải mô tả lại lỗi hoặc yêu cầu thay đổi trong một prompt mới, bạn có thể "chỉ" vào chính xác điểm cần sửa đổi, giúp AI hiểu rõ "vibe" điều chỉnh của bạn và thực hiện các thay đổi chính xác hơn. Antigravity IDE cũng có cơ chế phản hồi tương tự khi bạn chỉnh sửa file sau khi nó thực hiện thay đổi, nhưng khả năng bình luận trực tiếp trên diff của Claude Code Desktop là một giao diện trực quan hơn cho phản hồi chi tiết.

## IV. Cấu Hình và Mở Rộng Claude Code Desktop: Tối Ưu Hóa Môi Trường Agentic

Để tối ưu hóa trải nghiệm sử dụng và tích hợp Claude Code vào quy trình Vibe Coding với Antigravity IDE, việc cấu hình ứng dụng Claude Code Desktop theo nhu cầu của bạn là rất quan trọng. Bạn có thể truy cập các tùy chọn cài đặt bằng cách nhấp vào biểu tượng cài đặt (thường là bánh răng cưa).

### 1. Cài Đặt Chung và Quyền Riêng Tư

*   **Giao diện:** Tùy chỉnh giao diện người dùng, chủ đề (sáng/tối), font chữ, v.v., để tạo ra một môi trường làm việc phù hợp với "vibe" cá nhân của bạn.
*   **Tài khoản:** Quản lý tài khoản Anthropic của bạn, đăng xuất.
*   **Sử dụng:** Xem mức độ sử dụng token của bạn, giúp bạn theo dõi chi phí và tối ưu hóa cách sử dụng AI.
*   **Cài đặt Quyền Riêng Tư:** Trong phần này, bạn có thể quản lý dữ liệu nào được chia sẻ với Anthropic. Hãy đảm bảo bạn chỉ chia sẻ những gì bạn cảm thấy thoải mái, tương tự như việc cấu hình quyền riêng tư cho Antigravity IDE hoặc bất kỳ công cụ AI nào khác.

### 2. Cấu Hình Claude Code Cụ Thể

Phần này chứa các tùy chọn quan trọng ảnh hưởng đến cách Claude Code tương tác với dự án của bạn và mức độ "agentic" của nó:

*   **Bypass Permissions Mode (Bỏ qua chế độ quyền):**
    *   Khi được bật, Claude Code sẽ không bao giờ hỏi bạn bất kỳ quyền nào trước khi thực hiện hành động, kể cả các hành động nguy hiểm như xóa file hoặc thực hiện `git commit`.
    *   > [!WARNING]
        > Tính năng này làm cho Claude Code trở nên "agentic" hơn, hoạt động độc lập hơn, nhưng cũng tăng rủi ro. Chỉ nên sử dụng nếu bạn hoàn toàn tin tưởng Claude Code và hiểu rõ hậu quả. Nó tương đương với chế độ `dangerously-skip-permissions` trong CLI. Trong bối cảnh Antigravity IDE, bạn đã quen với việc một agent tự động thực thi các hành động. Tuy nhiên, với một công cụ bên ngoài như Claude Code, việc cân nhắc kỹ lưỡng mức độ tự chủ là cực kỳ quan trọng.
*   **Vị trí lưu trữ Worktree:** Cấu hình thư mục mà các bản sao Git Worktree sẽ được tạo và lưu trữ. Việc quản lý vị trí này giúp bạn tổ chức các môi trường thử nghiệm của AI một cách gọn gàng.
*   **Đưa ứng dụng lên foreground:** Tùy chọn này sẽ tự động đưa ứng dụng Claude Code lên trước màn hình khi nó cần bạn cấp quyền hoặc nhập liệu, đảm bảo bạn không bỏ lỡ các tương tác quan trọng.
*   **Bật/Tắt tính năng Preview:** Bạn có thể bật hoặc tắt chế độ xem trước (Preview Mode) tùy theo nhu cầu. Nếu bạn đang làm việc với backend hoặc CLI, việc tắt Preview có thể giúp giảm tải tài nguyên.

> [!TIP]
> Nên định kỳ kiểm tra các cài đặt này để đảm bảo chúng vẫn phù hợp với quy trình làm việc và sở thích của bạn, đặc biệt khi bạn tích hợp Claude Code vào một workflow Vibe Coding phức tạp với Antigravity IDE.

### 3. Tùy Chỉnh và Mở Rộng Khả Năng Agentic (Skills, Connectors, Plugins)

Nút "Customize" cung cấp quyền truy cập vào các công cụ mở rộng khả năng của Claude Code, biến nó thành một phần linh hoạt hơn trong hệ sinh thái agentic của bạn:

*   **Skills (Kỹ năng):**
    *   **Quản lý Skills:** Xem, thêm, tạo các kỹ năng mới. Bạn có thể tạo skill với sự trợ giúp của Claude Code hoặc viết thủ công. Skills giúp Claude Code thực hiện các tác vụ chuyên biệt hơn (ví dụ: kỹ năng để tạo file cấu hình cụ thể, chạy một loại kiểm thử nhất định, hoặc tương tác với một API nội bộ).
    *   **Duyệt Skills:** Khám phá và cài đặt các kỹ năng được Anthropic đề xuất sẵn.
    *   **Vibe Coding & Agentic Ecosystem:** Việc tạo và quản lý Skills cho Claude Code tương tự như việc bạn huấn luyện hoặc cung cấp các công cụ chuyên biệt cho các subagent của Antigravity IDE. Nó mở rộng khả năng của AI để thực hiện các tác vụ phức tạp hơn, phù hợp với "vibe" của dự án của bạn.
*   **Connectors (Kết nối):**
    *   Tích hợp Claude Code với các dịch vụ bên ngoài như GitHub. Ví dụ, thiết lập kết nối GitHub cho phép Claude Code tự động mở Pull Request sau khi hoàn thành tác vụ trên Claude Code Web. Điều này tự động hóa các bước trong quy trình CI/CD, giải phóng thời gian cho bạn.
*   **Plugins (Plugin):**
    *   **Quản lý Plugins:** Tương tự như Skills, bạn có thể quản lý các plugin đã cài đặt (ví dụ: plugin Playwright để kiểm thử end-to-end, hoặc plugin cho các công cụ khác).
    *   **Duyệt và Cài đặt Plugins:** Khám phá thư viện plugin để mở rộng chức năng của Claude Code. Điều này tương đương với việc sử dụng lệnh `/plugin` trong CLI.
    *   **Synergy với Antigravity IDE:** Việc mở rộng Claude Code thông qua Skills, Connectors và Plugins cho phép nó tích hợp sâu hơn vào quy trình phát triển tổng thể của bạn, có thể hoạt động song song hoặc bổ trợ cho các agent của Antigravity IDE. Ví dụ, Antigravity có thể quản lý luồng công việc tổng thể, trong khi Claude Code được giao nhiệm vụ viết kiểm thử Playwright thông qua plugin tương ứng.

## Kết Luận: Claude Code và Antigravity IDE – Hai Mảnh Ghép Hoàn Hảo cho Vibe Coding

Chương này đã cung cấp một cái nhìn toàn diện về cách sử dụng Claude Code trên nền tảng web và ứng dụng desktop, từ các tính năng cơ bản đến các khả năng nâng cao. Claude Code, với sức mạnh của mô hình Claude của Anthropic, mang lại những lợi ích đáng kể trong việc tăng năng suất và tự động hóa các tác vụ lập trình.

*   **Claude Code Web** cung cấp khả năng xử lý từ xa, giải phóng tài nguyên cục bộ, cho phép làm việc không gián đoạn và tích hợp liền mạch với Git, lý tưởng cho các tác vụ nặng hoặc chạy song song.
*   **Ứng dụng Claude Code Desktop** mang đến một giao diện đồ họa trực quan, quản lý phiên làm việc hiệu quả, khả năng cung cấp ngữ cảnh đa dạng thông qua Vibe Prompting, và các chế độ xem chuyên biệt như Preview Mode với tính năng "Select Elements" đột phá, cùng với Diff Viewer cho phép phản hồi trực tiếp vào code.
*   Các tùy chọn cấu hình và khả năng mở rộng thông qua Skills, Connectors, và Plugins cho phép bạn tùy chỉnh Claude Code để phù hợp với mọi quy trình làm việc agentic.

Khi kết hợp với **Antigravity IDE** – hệ thống agentic siêu việt với khả năng tự chạy script, gọi subagent trình duyệt, đọc ghi file và lập kế hoạch tự động – bạn sẽ có trong tay một bộ công cụ Vibe Coding vô cùng mạnh mẽ. Antigravity IDE có thể quản lý luồng công việc tổng thể, thực hiện các tác vụ phức tạp với sự tự chủ cao, trong khi Claude Code bổ trợ bằng cách:

*   **Offload tác vụ nặng:** Sử dụng Claude Code Web để giải phóng tài nguyên cho Antigravity IDE.
*   **Tương tác UI trực quan:** Tận dụng Preview Mode và "Select Elements" của Claude Code Desktop để tinh chỉnh giao diện một cách trực quan, một khía cạnh mà Antigravity có thể cần đến sự hỗ trợ của con người.
*   **Phản hồi chính xác:** Sử dụng Diff Viewer để cung cấp phản hồi trực tiếp vào code, giúp AI hiểu rõ "vibe" điều chỉnh của bạn.
*   **Thử nghiệm cô lập:** Với Git Worktree, cả hai hệ thống có thể thử nghiệm các thay đổi một cách an toàn.

Việc nắm vững cả Claude Code và Antigravity IDE, cùng với tư duy Vibe Coding, sẽ giúp bạn tối ưu hóa mọi khía cạnh của quy trình phát triển, từ lập kế hoạch, thực thi đến kiểm thử và tinh chỉnh, tạo ra một trải nghiệm lập trình hiệu quả và thú vị hơn bao giờ hết.

<!-- REVIEWED_BY_AGENT -->
