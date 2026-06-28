# Phần 5: Thiết Lập Nền Tảng Dự Án và Tối Ưu Hóa Tương Tác với Claude Code

Trong kỷ nguyên phát triển phần mềm được hỗ trợ bởi trí tuệ nhân tạo, việc thiết lập một nền tảng vững chắc và tối ưu hóa giao tiếp với các công cụ AI là yếu tố then chốt quyết định hiệu quả và chất lượng sản phẩm. Phần này sẽ đi sâu vào các kỹ thuật và chiến lược để khởi tạo dự án với Claude Code – công cụ CLI AI mạnh mẽ của Anthropic – một cách chuyên nghiệp. Chúng ta sẽ khám phá cách chuẩn bị môi trường, tận dụng tệp `CLAUDE.MD` như một bộ não ngữ cảnh, và khai thác Chế độ Lập kế hoạch (Plan Mode) cùng các công cụ tích hợp để giải quyết các tác vụ phức tạp. Đặc biệt, chúng ta sẽ lồng ghép tư duy Vibe Coding, hướng dẫn bạn cách tương tác hiệu quả với cả Claude Code và hệ thống Agentic AI tiên tiến như Antigravity IDE.

## 1. Thiết Lập Nền Tảng Dự Án với Claude Code

Việc khởi tạo một dự án với sự hỗ trợ của AI không chỉ dừng lại ở việc tạo thư mục. Nó đòi hỏi một quy trình có chiến lược để đảm bảo AI có đủ thông tin và ngữ cảnh để hoạt động hiệu quả nhất.

### 1.1. Chuẩn Bị Môi Trường Phát Triển Thủ Công

Mặc dù các công cụ AI coding như Claude Code có khả năng tự động cài đặt các gói phụ thuộc, nhưng với tư cách là một kỹ sư phần mềm, việc chủ động thiết lập môi trường ban đầu mang lại nhiều lợi ích chiến lược:

*   **Kiểm soát phiên bản tuyệt đối:** AI có thể mắc lỗi khi lựa chọn phiên bản gói, đôi khi cài đặt các phiên bản cũ hơn hoặc không tương thích. Việc bạn tự cài đặt đảm bảo sử dụng các phiên bản mới nhất, ổn định nhất hoặc phiên bản cụ thể theo yêu cầu của dự án.
*   **Hiểu biết sâu sắc về hệ thống:** Quá trình cài đặt thủ công giúp bạn làm quen với các phụ thuộc chính, cách chúng tương tác và các cấu hình cần thiết, từ đó xây dựng một "mental model" vững chắc về dự án.
*   **Tối ưu hóa tài nguyên AI:** Khi AI không phải dành token và thời gian để xác định và cài đặt các gói cơ bản, nó có thể tập trung vào các tác vụ phức tạp hơn, tiết kiệm chi phí và tăng tốc độ xử lý.

**Ví dụ thực tế: Cài đặt phụ thuộc với Bun**

Trong ví dụ này, chúng ta sử dụng Bun, một runtime JavaScript/TypeScript hiệu năng cao, để quản lý các gói.

```bash
# Cài đặt các gói phụ thuộc chính thức của ứng dụng
bun add betterauth zod @tiptap/core @tiptap/pm @tiptap/starter-kit

# Cài đặt các gói chỉ dành cho phát triển (ví dụ: định nghĩa kiểu TypeScript)
bun add -D bun-types
```

*   `betterauth`: Thư viện xác thực.
*   `zod`: Thư viện xác thực schema mạnh mẽ, giúp định nghĩa kiểu dữ liệu và đảm bảo an toàn kiểu.
*   `@tiptap/core`, `@tiptap/pm`, `@tiptap/starter-kit`: Các gói cần thiết cho việc triển khai trình soạn thảo văn bản phong phú Tiptap.
*   `bun-types`: Định nghĩa kiểu TypeScript cho API của Bun, quan trọng để tận dụng Bun API mà không gặp lỗi TypeScript trong quá trình phát triển.

Sau khi cài đặt, việc cấu hình tệp `package.json` là cần thiết để đảm bảo các lệnh script của dự án luôn được thực thi bằng Bun, tận dụng tối đa các tính năng và hiệu suất của runtime này.

**Ví dụ cấu hình `package.json`:**

```json
{
  "name": "my-claude-project",
  "version": "1.0.0",
  "scripts": {
    "dev": "bun run --bun src/index.ts",
    "build": "bun run --bun build",
    "start": "bun run --bun dist/index.js",
    "lint": "bun run --bun lint"
  },
  "dependencies": {
    "@tiptap/core": "^2.1.12",
    "@tiptap/pm": "^2.1.12",
    "@tiptap/starter-kit": "^2.1.12",
    "betterauth": "^1.0.0",
    "zod": "^3.22.4"
  },
  "devDependencies": {
    "bun-types": "^1.0.14"
  }
}
```

Lưu ý cách sử dụng `bun run --bun` cho mỗi script. Điều này chỉ định rõ ràng rằng Bun là runtime được sử dụng để chạy các lệnh này, thay vì Node.js hoặc một runtime khác.

### 1.2. Khởi Tạo Dự Án với Lệnh `/init` trong Claude Code

Khi bắt đầu một dự án mới hoặc đưa Claude Code vào một codebase hiện có, lệnh `/init` là bước khởi đầu quan trọng. Nó cho phép Claude Code xây dựng một "bức tranh tổng thể" ban đầu về dự án của bạn.

> [!TIP]
> **Khi nào nên dùng `/init`?**
> Luôn sử dụng `/init` khi bạn bắt đầu một dự án mới với Claude Code, hoặc khi bạn muốn Claude Code có cái nhìn sâu sắc về một codebase hiện có mà nó chưa từng phân tích.

**Cơ chế hoạt động của `/init`:**

1.  **Phân tích cấu trúc codebase:** Claude Code sẽ quét toàn bộ thư mục dự án, nhận diện cấu trúc file, thư mục, và các loại file (`.ts`, `.js`, `.json`, `.md`, v.v.).
2.  **Đánh giá dependencies:** Nó kiểm tra các tệp quản lý phụ thuộc (ví dụ: `package.json`, `bun.lockb`) để hiểu các thư viện đã cài đặt, phiên bản và cách chúng được cấu hình.
3.  **Nhận diện tệp `Spec.md`:** Nếu có tệp `Spec.md` (tệp mô tả chi tiết đặc tả dự án), Claude Code sẽ ghi nhận sự tồn tại của nó để tham chiếu sau này.
4.  **Tạo tệp `CLAUDE.MD` ban đầu:** Dựa trên tất cả thông tin thu thập được, Claude Code sẽ tự động tạo một tệp `CLAUDE.MD` ở thư mục gốc của dự án. Tệp này đóng vai trò là "bộ não" ban đầu, chứa các chỉ dẫn cơ bản cho Claude Code về dự án.

Sau khi chạy `/init`, Claude Code có thể yêu cầu bạn cấp quyền để khám phá nội dung các thư mục. Đây là một bước cần thiết để nó có thể xây dựng một ngữ cảnh toàn diện và chính xác về dự án.

**Liên hệ với Antigravity IDE:**
Trong một hệ thống như Antigravity IDE, quy trình "khởi tạo" này có thể diễn ra một cách ít rõ ràng hơn nhưng sâu sắc hơn. Các subagent chuyên biệt của Antigravity có thể tự động quét, lập chỉ mục và xây dựng một đồ thị phụ thuộc (dependency graph) của toàn bộ codebase ngay khi bạn mở dự án. Khác với Claude Code CLI yêu cầu lệnh `/init` tường minh, Antigravity có thể liên tục duy trì một "mental model" cập nhật về dự án, tự động thích nghi với các thay đổi trong thời gian thực.

## 2. CLAUDE.MD: Bộ Não Ngữ Cảnh của Dự Án

Tệp `CLAUDE.MD` là một thành phần cốt lõi trong bất kỳ dự án Claude Code nào, đóng vai trò là bộ nhớ dài hạn và nguồn hướng dẫn cơ bản cho AI. Việc quản lý tệp này một cách hiệu quả sẽ cải thiện đáng kể chất lượng, tốc độ và độ chính xác trong tương tác của bạn với Claude Code.

### 2.1. Vai Trò Trung Tâm của CLAUDE.MD

*   **Bộ nhớ dài hạn và hướng dẫn chung:** `CLAUDE.MD` chứa các quy tắc, nguyên tắc, thông tin về ngăn xếp công nghệ và các chỉ dẫn quan trọng mà Claude Code cần biết về dự án của bạn. Nó như một tài liệu hướng dẫn nội bộ dành riêng cho AI.
*   **Tự động tải vào ngữ cảnh:** Nội dung của tệp `CLAUDE.MD` ở thư mục gốc sẽ tự động được tải vào ngữ cảnh của mọi phiên làm việc mới với Claude Code. Điều này cũng xảy ra khi bạn xóa ngữ cảnh của một phiên bằng lệnh `/clear`. Cơ chế này đảm bảo Claude Code luôn có được thông tin cơ bản nhất về dự án mà không cần bạn phải nhắc lại liên tục.
*   **Vị trí chiến lược:** Mặc dù bạn có thể tạo nhiều tệp `CLAUDE.MD` ở các thư mục con, nhưng ít nhất một tệp nên nằm ở thư mục gốc của dự án để đảm bảo ngữ cảnh tổng thể luôn được nạp.

### 2.2. Xây Dựng Nội Dung CLAUDE.MD Hiệu Quả

Nội dung của `CLAUDE.MD` cần được cân nhắc kỹ lưỡng để cung cấp đủ thông tin mà không gây "ô nhiễm ngữ cảnh" (context pollution).

> [!IMPORTANT]
> **Tối ưu hóa token:**
> Mỗi khi `CLAUDE.MD` được tải, nội dung của nó sẽ chiếm một phần trong cửa sổ ngữ cảnh (context window) và tiêu tốn token. Một tệp quá lớn sẽ làm giảm không gian cho các chi tiết cụ thể của tác vụ hiện tại, tăng chi phí và có thể làm chậm quá trình xử lý. Mục tiêu là cung cấp thông tin cô đọng, chiến lược nhất.

**Các thành phần quan trọng của `CLAUDE.MD`:**

1.  **Thông tin về môi trường và công cụ:** Hướng dẫn Claude Code về các công cụ và công nghệ chính mà dự án đang sử dụng.
    *   Ví dụ: "Chúng ta đang sử dụng Bun làm runtime và quản lý gói. Luôn sử dụng `bun` để thực thi các script và cài đặt phụ thuộc trong dự án này."
2.  **Tham chiếu có điều kiện đến tài liệu chi tiết (`Spec.md`):** Nếu bạn có tệp `Spec.md` mô tả kiến trúc hoặc đặc tả ứng dụng, hãy tham chiếu nó. Điều quan trọng là phải hướng dẫn Claude Code *khi nào* nên đọc tệp này để tránh lãng phí token.
    *   Ví dụ: "Ứng dụng được mô tả chi tiết trong `@spec.md`. Hãy tham khảo tệp này khi xử lý các tác vụ liên quan đến kiến trúc tổng thể, cấu trúc cơ sở dữ liệu, ngăn xếp công nghệ hoặc thiết kế ứng dụng chung."
3.  **Chỉ dẫn về định dạng phản hồi:** Yêu cầu Claude Code cung cấp phản hồi ngắn gọn, súc tích, đi thẳng vào vấn đề, không phải những bài luận dài dòng với nhiều đoạn mã không cần thiết. Điều này giúp tiết kiệm token, giữ cho ngữ cảnh hội thoại rõ ràng và tăng tốc độ tương tác.
    *   Ví dụ: "Hãy cung cấp các phản hồi ngắn gọn, tập trung vào giải pháp và chỉ bao gồm các đoạn mã cần thiết. Tránh các giải thích dài dòng trừ khi được yêu cầu cụ thể."

**Ví dụ Code: Nội dung mẫu của `CLAUDE.MD`**

```markdown
# Hướng Dẫn Dự Án Claude Code: MyAwesomeApp

## Ngăn Xếp Công Nghệ & Môi Trường

*   **Runtime & Quản lý gói:** Bun (phiên bản 1.x)
*   **Ngôn ngữ:** TypeScript (phiên bản 5.x)
*   **Khung giao diện người dùng:** React (sử dụng Vite làm bundler)
*   **Cơ sở dữ liệu:** SQLite (sử dụng API tích hợp của Bun)
*   **Xác thực:** BetterAuth (phiên bản 1.x)
*   **Trình soạn thảo văn bản:** Tiptap

> [!TIP]
> **Nguyên tắc sử dụng Bun:**
> Luôn sử dụng `bun` cho mọi hoạt động liên quan đến gói và script. Ví dụ: `bun add <package>`, `bun run <script-name>`, `bun build`.

## Kiến Trúc Ứng Dụng & Quy Ước Mã Hóa

Ứng dụng này tuân thủ kiến trúc mô tả trong `@spec.md`.
**Hãy tham khảo `@spec.md` khi:**
*   Thiết kế hoặc sửa đổi các thành phần kiến trúc cấp cao.
*   Xác định cấu trúc dữ liệu hoặc schema cơ sở dữ liệu.
*   Cần hiểu rõ về luồng dữ liệu hoặc các module chính.
*   Có bất kỳ nghi ngờ nào về ngăn xếp công nghệ hoặc các thư viện cốt lõi.

## Nguyên Tắc Tương Tác với Claude Code

> [!NOTE]
> **Định dạng phản hồi ưu tiên:**
> Vui lòng cung cấp các phản hồi ngắn gọn, trực tiếp và chỉ bao gồm các thay đổi mã hoặc hướng dẫn cần thiết. Tránh các đoạn mã boilerplate hoặc các giải thích không cần thiết. Mục tiêu là hiệu quả, rõ ràng và giảm thiểu token sử dụng.
>
> **Ưu tiên giải pháp tối giản:**
> Luôn tìm kiếm giải pháp tối giản và hiệu quả nhất. Không thêm các tính năng hoặc thư viện không được yêu cầu cụ thể.
```

### 2.3. Quản Lý Ngữ Cảnh Phân Cấp với CLAUDE.MD

Claude Code cho phép bạn tạo nhiều tệp `CLAUDE.MD` trong các thư mục con của dự án. Đây là một tính năng mạnh mẽ để quản lý ngữ cảnh theo module hoặc tính năng.

*   **Cơ chế tải ngữ cảnh lồng nhau:** Các tệp `CLAUDE.MD` lồng nhau này sẽ chỉ được Claude Code đọc và tải vào ngữ cảnh khi nó làm việc trên các tệp trong chính thư mục con đó hoặc các thư mục con sâu hơn.
*   **Lợi ích của cấu trúc phân cấp:**
    *   **Ngữ cảnh cục bộ chính xác:** Đối với các dự án lớn, việc này cho phép bạn cung cấp các hướng dẫn cụ thể theo từng module hoặc thành phần của ứng dụng. Ví dụ, một tệp `src/features/auth/CLAUDE.MD` có thể chứa các quy tắc cụ thể cho module xác thực, đảm bảo Claude Code có ngữ cảnh chính xác khi xử lý các tệp trong đó.
    *   **Giảm thiểu ô nhiễm ngữ cảnh:** Chỉ những ngữ cảnh liên quan nhất mới được tải, giúp tối ưu hóa việc sử dụng token và tránh làm loãng thông tin quan trọng.
    *   **Dễ dàng mở rộng và bảo trì:** Khi dự án phát triển, bạn có thể dễ dàng thêm hoặc cập nhật các tệp `CLAUDE.MD` cho các module mới mà không ảnh hưởng đến ngữ cảnh chung của dự án.

**Ví dụ:**
Nếu Claude Code đang làm việc trên `src/features/auth/login.ts`, nó sẽ tải `CLAUDE.MD` ở thư mục gốc, sau đó `src/CLAUDE.MD` (nếu có), và cuối cùng là `src/features/auth/CLAUDE.MD`. Các quy tắc cụ thể nhất sẽ có ưu tiên.

## 3. Tư Duy Vibe Coding và Chế Độ Lập Kế Hoạch (Plan Mode)

Vibe Coding là một triết lý phát triển phần mềm tập trung vào việc giao tiếp ý định và "cảm nhận" về mã hóa với AI, thay vì chỉ cung cấp các hướng dẫn từng bước cứng nhắc. Chế độ Lập kế hoạch (Plan Mode) của Claude Code là công cụ lý tưởng để hiện thực hóa triết lý này.

### 3.1. Vibe Coding: Tối Ưu Hóa Giao Tiếp với AI Agent

Vibe Coding không chỉ là việc viết prompt; đó là nghệ thuật truyền đạt *tinh thần* và *mục tiêu* của tác vụ cho AI. Thay vì chỉ nói "tạo một hàm", bạn sẽ truyền đạt "tạo một hàm `authenticateUser` xử lý xác thực người dùng một cách an toàn, tuân thủ các nguyên tắc của `BetterAuth` và trả về một đối tượng người dùng đã được xác thực, theo phong cách mã hóa hiện đại và dễ đọc của dự án."

**Các nguyên tắc của Vibe Coding:**

*   **Rõ ràng về ý định, không phải chi tiết thực thi:** Tập trung vào *cái gì* cần đạt được và *tại sao*, để AI tự tìm ra *cách làm*.
*   **Cung cấp ngữ cảnh phong phú:** Sử dụng `CLAUDE.MD`, `Spec.md`, và các tệp liên quan để AI hiểu "vibe" của dự án.
*   **Phản hồi lặp đi lặp lại:** Xem xét kế hoạch của AI, cung cấp phản hồi tinh chỉnh để điều chỉnh "vibe" cho đến khi khớp với ý định của bạn.
*   **Tin tưởng vào khả năng lập kế hoạch của AI:** Cho phép AI đưa ra các đề xuất và khám phá giải pháp.

**Liên hệ với Antigravity IDE:**
Antigravity IDE với khả năng tự chạy script ngầm, gọi subagent trình duyệt, đọc ghi file và lập kế hoạch tự động là một môi trường lý tưởng cho Vibe Coding. Bạn giao cho Antigravity một mục tiêu cấp cao, và nó sẽ tự mình lập kế hoạch chi tiết, thực thi, kiểm tra và lặp lại. Vibe Coding ở đây là việc cung cấp cho Antigravity một "tầm nhìn" rõ ràng về sản phẩm cuối cùng, cho phép nó tự động điều chỉnh các bước nhỏ hơn để đạt được tầm nhìn đó.

### 3.2. Chuẩn Bị Phiên Làm Việc: Lệnh `/clear` và "Tấm Bảng Trắng"

Trước khi bắt đầu một tác vụ mới, việc xóa ngữ cảnh phiên hiện tại bằng lệnh `/clear` là một thực hành tốt.

*   **Mục đích:** Đảm bảo Claude Code bắt đầu với một "tấm bảng trắng," chỉ giữ lại thông tin từ tệp `CLAUDE.MD` của bạn. Điều này tránh bị nhiễu bởi các cuộc trò chuyện hoặc các tác vụ trước đó không liên quan, giúp AI tập trung hoàn toàn vào nhiệm vụ hiện tại.
*   **Lợi ích:** Tăng độ chính xác của phản hồi, giảm thiểu lỗi do ngữ cảnh cũ và tối ưu hóa việc sử dụng token.

### 3.3. Plan Mode (`Shift + Tab`): Biến Ý Tưởng Thành Kế Hoạch Chi Tiết

Plan Mode là một tính năng mạnh mẽ của Claude Code, giúp biến các prompt trung bình thành các prompt tuyệt vời bằng cách khuyến khích Claude Code lập kế hoạch trước khi thực thi. Đây là chế độ mặc định được khuyến nghị cho hầu hết các tác vụ, ngay cả những tác vụ có vẻ đơn giản.

> [!TIP]
> **Luôn bắt đầu với Plan Mode:**
> Đối với đại đa số các tác vụ, hãy bắt đầu trong Plan Mode (`Shift + Tab`). Chế độ này giúp Claude Code thu thập thông tin liên quan, đặt câu hỏi làm rõ và tạo ra một kế hoạch hành động chi tiết trước khi bắt đầu công việc.

**Mục đích và lợi ích của Plan Mode:**

*   **Thu thập thông tin chủ động:** Claude Code sẽ khám phá dự án của bạn, đọc các tệp liên quan, phân tích cấu trúc và phụ thuộc để hiểu rõ hơn về ngữ cảnh.
*   **Làm rõ yêu cầu:** Nếu prompt của bạn chưa đủ rõ ràng hoặc có thể dẫn đến nhiều cách hiểu, Claude Code có thể đặt câu hỏi để làm rõ nhiệm vụ, giúp bạn tinh chỉnh prompt.
*   **Cải thiện prompt (Vibe Refinement):** Plan Mode là một vòng lặp phản hồi tuyệt vời. Nó giúp bạn biến những prompt chung chung thành những prompt có cấu trúc tốt hơn, phản ánh đúng "vibe" của tác vụ.
*   **Không chỉnh sửa file (An toàn):** Trong Plan Mode, Claude Code không có quyền chỉnh sửa file. Nó chỉ tập trung vào việc phân tích, lập kế hoạch và giao tiếp, mang lại sự an toàn cho codebase của bạn.
*   **Lưu trữ kế hoạch tạm thời:** Các kế hoạch được tạo ra sẽ được lưu trữ cục bộ tạm thời, cho phép Claude Code tham chiếu lại chúng ngay cả khi ngữ cảnh phiên bị xóa, cung cấp một dạng "bộ nhớ ngắn hạn cho kế hoạch".

**Cơ chế "dưới vỏ bọc" của Plan Mode:**
Khi bạn nhập một prompt vào Plan Mode, Claude Code không trực tiếp tạo mã. Thay vào đó, nó sử dụng một chuỗi prompt nội bộ để:
1.  **Phân tích prompt của bạn:** Hiểu ý định cốt lõi.
2.  **Lập kế hoạch phân rã tác vụ:** Chia nhỏ nhiệm vụ lớn thành các bước nhỏ hơn, có thể quản lý được.
3.  **Thu thập thông tin bối cảnh:** Xác định những file nào cần đọc, những thư viện nào cần tham khảo.
4.  **Tạo ra một "kế hoạch hành động":** Đây là danh sách các bước mà nó tin rằng sẽ hoàn thành tác vụ, kèm theo lý do và các giả định. Kế hoạch này sau đó được trình bày cho bạn.

**So sánh với Antigravity IDE:**
Plan Mode của Claude Code là một bước tiến quan trọng so với các AI coding cơ bản. Tuy nhiên, Antigravity IDE đưa Plan Mode lên một tầm cao mới. Antigravity không chỉ "đề xuất" một kế hoạch; nó *tự động thực thi* một quy trình lập kế hoạch phức tạp bao gồm:
*   **Tạo subagent chuyên biệt:** Cho từng phần của kế hoạch (ví dụ: một subagent để tìm tài liệu, một subagent để viết test, một subagent để refactor).
*   **Chạy script ngầm:** Tự động chạy các lệnh như `git status`, `ls -l`, `grep` để thu thập thông tin mà không cần sự can thiệp của người dùng.
*   **Phản hồi vòng lặp nội bộ:** Các subagent có thể tự điều chỉnh kế hoạch dựa trên kết quả của các script hoặc tương tác với các hệ thống khác (ví dụ: gọi API).
*   **Lập kế hoạch tự động liên tục:** Antigravity có thể liên tục đánh giá lại và điều chỉnh kế hoạch khi codebase thay đổi hoặc khi có thông tin mới.

### 3.4. Đánh Giá và Tinh Chỉnh Kế Hoạch

Khi Claude Code trình bày kế hoạch, bạn có một cơ hội quý giá để đánh giá, tinh chỉnh và đảm bảo nó phù hợp với "vibe" của dự án và ý định của bạn.

**Các tùy chọn sau khi có kế hoạch:**

*   **Chấp nhận kế hoạch, xóa ngữ cảnh, tự động chấp nhận chỉnh sửa:** Tùy chọn này cho phép Claude Code thực hiện kế hoạch, xóa bỏ ngữ cảnh hội thoại trước đó (nhưng vẫn giữ kế hoạch), và tự động áp dụng các thay đổi mà nó thực hiện. Thích hợp cho các tác vụ đơn giản, rõ ràng.
*   **Chấp nhận kế hoạch, không xóa ngữ cảnh, phê duyệt chỉnh sửa thủ công:** Claude Code thực hiện kế hoạch nhưng bạn cần xem xét và phê duyệt từng chỉnh sửa trước khi áp dụng vào codebase. Đây là lựa chọn an toàn và được khuyến nghị cho hầu hết các tác vụ.
*   **Cung cấp thêm thông tin/sửa đổi kế hoạch:** Nếu bạn không hài lòng với kế hoạch, muốn bổ sung chi tiết hoặc điều chỉnh hướng đi, bạn có thể cung cấp phản hồi. Claude Code sẽ quay lại Plan Mode và điều chỉnh kế hoạch dựa trên thông tin mới của bạn.

**Ví dụ: Tinh chỉnh kế hoạch cho lộ trình xác thực**

Giả sử bạn yêu cầu Claude Code thiết lập các lộ trình cốt lõi, và nó đề xuất nhiều lộ trình xác thực (ví dụ: `/login`, `/register`, `/forgot-password`). Bạn có thể phản hồi để tinh chỉnh "vibe" của kế hoạch:

```
"Kế hoạch này có vẻ tốt, nhưng tôi chỉ muốn một lộ trình xác thực duy nhất là `/authenticate` tại thời điểm này. Lộ trình này chỉ nên hỗ trợ xác thực bằng email và mật khẩu, không cần các phương thức khác. Hãy điều chỉnh kế hoạch."
```

Claude Code sẽ nhận thông tin này, quay lại Plan Mode, và cập nhật kế hoạch để phù hợp với yêu cầu mới của bạn, phản ánh đúng "vibe" mà bạn muốn truyền tải. Quá trình này có thể lặp lại cho đến khi kế hoạch hoàn toàn khớp với ý định của bạn.

### 3.5. Thực Thi và Kiểm Chứng Kết Quả

Sau khi bạn đã phê duyệt kế hoạch (và có thể chọn tự động chấp nhận chỉnh sửa), Claude Code sẽ bắt đầu thực hiện công việc.

*   **Chạy các công cụ (khi được cho phép):** Claude Code có thể yêu cầu bạn cho phép chạy các công cụ như linter (để kiểm tra lỗi cú pháp và phong cách), formatter (để định dạng mã), hoặc lệnh build (để kiểm tra xem dự án có biên dịch thành công không). Bạn nên cho phép các hoạt động này để đảm bảo chất lượng mã và phát hiện sớm các vấn đề.
*   **Kiểm tra kết quả (trách nhiệm của nhà phát triển):** Sau khi Claude Code hoàn thành, bạn *phải* tự mình kiểm tra kết quả.
    *   **Kiểm tra mã:** Đọc các thay đổi, đảm bảo chúng tuân thủ các nguyên tắc thiết kế và "vibe" của dự án.
    *   **Chạy ứng dụng:** Khởi động máy chủ phát triển và điều hướng đến các lộ trình hoặc tính năng đã tạo để xác minh rằng chúng hoạt động như mong đợi.

```bash
# Khởi động máy chủ phát triển sau khi Claude Code hoàn tất
bun run dev
```

Việc này cho phép bạn xác nhận rằng Claude Code đã thực hiện đúng yêu cầu và các chức năng cơ bản đã được thiết lập một cách chính xác.

## 4. Mở Rộng Khả Năng với Công Cụ Tích Hợp và Truy Cập Thông Tin

Khi các tác vụ trở nên phức tạp hơn, đòi hỏi sự tuân thủ các tài liệu cụ thể hoặc việc sử dụng các công nghệ nhất định, khả năng của Claude Code trong việc sử dụng các công cụ tích hợp và truy cập thông tin trở nên vô cùng giá trị.

### 4.1. Giải Quyết Tác Vụ Phức Tạp: Khi Tài Liệu Là Chìa Khóa

Các tác vụ như triển khai xác thực an toàn, tích hợp với API bên ngoài hoặc truy cập cơ sở dữ liệu thường yêu cầu Claude Code phải tuân thủ các quy tắc cụ thể của một thư viện, framework hoặc một công nghệ.

*   **Thách thức của ngữ cảnh hạn chế:** Nếu bạn chỉ yêu cầu Claude Code "triển khai xác thực và truy cập cơ sở dữ liệu" mà không cung cấp ngữ cảnh chi tiết, khả năng cao là nó sẽ không thiết lập các tệp đúng cách hoặc không tuân theo các quy tắc của gói `BetterAuth` hay cách tương tác với SQLite tích hợp của Bun. AI có thể "halucinate" hoặc đưa ra các giải pháp chung chung không phù hợp với ngăn xếp công nghệ cụ thể của bạn.
*   **Tầm quan trọng của tài liệu:** Với tư cách là nhà phát triển, bạn biết rằng có các tài liệu hướng dẫn chính thức cho việc sử dụng `BetterAuth` hoặc cách tương tác với SQLite trong Bun. Claude Code cần được hướng dẫn để tham khảo các nguồn này.

### 4.2. Phương Pháp Cung Cấp Ngữ Cảnh cho AI

Để đảm bảo Claude Code thực hiện các tác vụ phức tạp một cách chính xác, bạn có thể cung cấp ngữ cảnh cần thiết thông qua các phương pháp sau:

1.  **Dán trực tiếp tài liệu vào prompt:** Nếu một phần tài liệu rất quan trọng và ngắn gọn, bạn có thể sao chép và dán trực tiếp nó vào prompt của mình.
    ```
    "Đây là hướng dẫn chính thức về cách sử dụng Bun với SQLite:
    ```typescript
    // [Dán nội dung tài liệu API SQLite của Bun tại đây]
    ```
    Vui lòng tuân thủ chặt chẽ các bước và API này để tạo và truy vấn cơ sở dữ liệu."
    ```
    > [!NOTE]
    > Phương pháp này hiệu quả cho các đoạn tài liệu ngắn nhưng có thể làm tăng kích thước prompt và tiêu tốn nhiều token nếu tài liệu dài, ảnh hưởng đến hiệu suất và chi phí.

2.  **Cung cấp liên kết (Claude Code có khả năng truy cập web):** Claude Code có khả năng truy cập các trang web. Bạn có thể cung cấp các URL đến tài liệu chính thức và yêu cầu nó truy cập để tìm thông tin.
    ```
    "Vui lòng truy cập các trang tài liệu chính thức sau để triển khai xác thực bằng BetterAuth và tương tác với SQLite trong Bun:
    - Tài liệu BetterAuth: `https://docs.betterauth.com/`
    - Tài liệu Bun SQLite: `https://bun.sh/docs/api/sqlite`
    Hãy đọc kỹ và áp dụng các hướng dẫn từ các nguồn này."
    ```
    Đây là một cách hiệu quả để cung cấp lượng lớn thông tin mà không làm quá tải ngữ cảnh prompt ban đầu.

3.  **Sử dụng tìm kiếm web (Claude Code có khả năng tìm kiếm web):** Bạn có thể hướng dẫn Claude Code tự tìm kiếm tài liệu liên quan nếu bạn không có sẵn liên kết cụ thể.
    ```
    "Sử dụng tìm kiếm web để tìm tài liệu chính thức về cách triển khai xác thực người dùng an toàn bằng thư viện BetterAuth và cách tương tác với cơ sở dữ liệu SQLite thông qua API của Bun. Sau đó, áp dụng các hướng dẫn đó để xây dựng module xác thực."
    ```
    Phương pháp này cho phép Claude Code chủ động tìm kiếm thông tin, nhưng có thể cần sự giám sát để đảm bảo nó tìm được nguồn đáng tin cậy và chính xác nhất.

**So sánh với Antigravity IDE:**
Antigravity IDE với "subagent trình duyệt" thể hiện khả năng truy cập thông tin một cách vượt trội. Không chỉ đọc trang web, subagent có thể *tương tác* với các trang web, điền form, click nút, thậm chí chạy các đoạn JavaScript trên trình duyệt để thu thập thông tin động. Điều này cho phép Antigravity không chỉ đọc tài liệu mà còn "học hỏi" từ các ví dụ trực tuyến, các sandbox hoặc các công cụ tương tác, mang lại khả năng giải quyết vấn đề linh hoạt và mạnh mẽ hơn nhiều.

### 4.3. Tương Tác với Hệ Sinh Thái Công Cụ Qua MCP Server

Trong một số trường hợp nâng cao, Claude Code cũng có thể tương tác với các công cụ hoặc dịch vụ bên ngoài thông qua một "MCP server" (Model Control Plane server).

*   **Mục đích:** MCP server hoạt động như một cầu nối, cho phép Claude Code gửi yêu cầu và nhận phản hồi từ các công cụ tùy chỉnh, API nội bộ của tổ chức, hoặc môi trường thực thi cụ thể mà nó không thể truy cập trực tiếp.
*   **Cơ chế hoạt động:** Claude Code sẽ gửi một "lệnh" hoặc "yêu cầu" được cấu trúc đến MCP server. MCP server sau đó sẽ dịch yêu cầu này thành các lệnh thực thi trên hệ thống đích (ví dụ: chạy một script bash tùy chỉnh, gọi một API nội bộ, tương tác với một hệ thống CI/CD), thu thập kết quả và trả về cho Claude Code.
*   **Tiềm năng:** Điều này mở rộng đáng kể khả năng của Claude Code, cho phép nó tích hợp sâu rộng vào quy trình làm việc và hệ sinh thái công cụ hiện có của bạn, vượt ra ngoài giới hạn của môi trường code trực tiếp. Ví dụ, nó có thể được cấu hình để tự động triển khai mã, chạy các bài kiểm thử phức tạp trên môi trường staging, hoặc tương tác với các hệ thống quản lý dự án.

**Liên hệ với Antigravity IDE:**
Khái niệm MCP server rất gần với cách Antigravity IDE hoạt động. Antigravity có thể coi là một hệ thống Agentic AI tích hợp sẵn một "MCP server" nội bộ, cho phép các agent của nó:
*   **Chạy script ngầm:** Thực hiện các lệnh hệ thống, công cụ CLI.
*   **Đọc/ghi file:** Tương tác trực tiếp với codebase.
*   **Gọi subagent trình duyệt:** Truy cập và tương tác với web.
*   **Tương tác với API:** Gửi yêu cầu đến các dịch vụ bên ngoài.

Antigravity IDE cung cấp một framework toàn diện để quản lý và điều phối các "công cụ" này, cho phép AI tự động lựa chọn và sử dụng công cụ phù hợp nhất để đạt được mục tiêu, làm cho nó trở thành một hệ thống "tự trị" hơn trong việc tương tác với môi trường phát triển.

## Tóm Tắt Phần 5: Khởi Tạo Dự Án, Tệp CLAUDE.MD và Công Cụ Tích Hợp

*   **Khởi tạo dự án hiệu quả:** Bắt đầu bằng việc cài đặt thủ công các gói phụ thuộc quan trọng (ví dụ: `betterauth`, `zod`, `tiptap` với Bun) và cấu hình `package.json` để đảm bảo sử dụng runtime chính xác. Điều này giúp kiểm soát phiên bản và tối ưu hóa tài nguyên AI.
*   **Lệnh `/init` trong Claude Code:** Sử dụng `/init` để phân tích codebase, các phụ thuộc và tệp `Spec.md` của dự án, tự động tạo tệp `CLAUDE.MD` ban đầu, giúp Claude Code xây dựng một "bức tranh tổng thể" về dự án.
*   **Tệp `CLAUDE.MD` là bộ nhớ dài hạn:** Đặt tệp `CLAUDE.MD` ở thư mục gốc chứa các quy tắc chung, thông tin về ngăn xếp công nghệ và hướng dẫn tương tác cho Claude Code. Nó được tự động tải vào mọi phiên làm việc.
*   **Tối ưu hóa `CLAUDE.MD`:** Giữ nội dung gọn nhẹ để tránh ô nhiễm ngữ cảnh và tối ưu hóa token, tham chiếu `Spec.md` có điều kiện, và yêu cầu Claude Code cung cấp phản hồi ngắn gọn, súc tích.
*   **CLAUDE.MD phân cấp:** Sử dụng nhiều tệp `CLAUDE.MD` trong các thư mục con để cung cấp ngữ cảnh cụ thể cho từng phần của dự án, tăng độ chính xác và giảm thiểu ô nhiễm ngữ cảnh.
*   **Tư duy Vibe Coding và Plan Mode:** Áp dụng triết lý Vibe Coding bằng cách bắt đầu các tác vụ trong Plan Mode (`Shift + Tab`) để Claude Code lập kế hoạch, hỏi làm rõ và đưa ra các đề xuất trước khi thực thi. Điều này giúp cải thiện chất lượng prompt và kết quả, đồng thời cho phép bạn tinh chỉnh "vibe" của tác vụ.
*   **Quản lý kế hoạch:** Đánh giá, chấp nhận, sửa đổi hoặc cung cấp thêm thông tin cho kế hoạch của Claude Code trước khi cho phép nó thực hiện các thay đổi, đảm bảo kế hoạch phù hợp với ý định của bạn.
*   **Khai thác công cụ tích hợp:** Đối với các tác vụ phức tạp, cung cấp ngữ cảnh cho Claude Code bằng cách dán tài liệu, cung cấp liên kết web hoặc yêu cầu nó sử dụng tìm kiếm web để tìm tài liệu chính thức. Khả năng truy cập web và tìm kiếm cho phép Claude Code vượt qua những hạn chế về ngữ cảnh ban đầu, giải quyết các vấn đề phức tạp dựa trên thông tin cập nhật.
*   **MCP Server:** Tương tác với các công cụ và dịch vụ bên ngoài thông qua một MCP server để mở rộng khả năng của Claude Code, cho phép nó tích hợp sâu rộng vào quy trình làm việc và hệ sinh thái công cụ hiện có của bạn.

<!-- REVIEWED_BY_AGENT -->
