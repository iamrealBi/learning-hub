# Chương 2: Sử Dụng Cơ Bản, Cấu Hình Sâu và Tư Duy Vibe Coding với Claude Code

Trong chương này, chúng ta sẽ đi sâu vào việc làm chủ Claude Code – một trợ lý AI mạnh mẽ dựa trên giao diện dòng lệnh (CLI) từ Anthropic, được thiết kế đặc biệt cho lập trình viên. Mục tiêu không chỉ là giúp bạn nắm vững cách sử dụng cơ bản, mà còn là hiểu sâu cơ chế hoạt động, tích hợp nó một cách hiệu quả vào quy trình làm việc hàng ngày, tối ưu hóa trải nghiệm thông qua các tùy chọn cấu hình nâng cao, và khám phá những tính năng cốt lõi có thể nâng cao năng suất của bạn một cách đáng kể. Đặc biệt, chúng ta sẽ giới thiệu khái niệm **Vibe Coding** – một triết lý tương tác với AI để đạt được kết quả tối ưu, và liên hệ nó với cách các hệ thống AI Agentic như **Antigravity IDE** hoạt động.

Chúng ta sẽ bắt đầu từ việc cài đặt, những tương tác đầu tiên, cách Claude Code "hiểu" và phản hồi, đến việc quản lý các phiên làm việc và tùy chỉnh hành vi của nó, đồng thời luôn đặt trong bối cảnh tư duy Vibe Coding và so sánh với khả năng của Antigravity IDE.

---

## I. Bắt Đầu Với Claude Code: Cài Đặt, Tương Tác Cơ Bản và Tích Hợp

Claude Code là một công cụ dòng lệnh (CLI) được thiết kế để hỗ trợ lập trình viên trong nhiều tác vụ, từ viết mã, sửa lỗi, tạo bài kiểm tra đến tìm hiểu dự án. Là một công cụ CLI, Claude Code mang lại sự linh hoạt cao, cho phép tích hợp vào mọi môi trường phát triển, đồng thời cung cấp khả năng tích hợp sâu rộng với các môi trường phát triển tích hợp (IDE) phổ biến.

### 1. Cài Đặt và Khởi Động Claude Code

Trước khi có thể tương tác với Claude Code, bạn cần cài đặt nó trên hệ thống của mình. Quy trình cài đặt thường liên quan đến việc tải xuống và chạy một trình cài đặt hoặc sử dụng một trình quản lý gói phù hợp với hệ điều hành của bạn. Sau khi cài đặt, bạn có thể khởi động Claude Code bằng cách gõ lệnh `claude` trong terminal.

```bash
claude
```

Lệnh này sẽ đưa bạn vào một giao diện tương tác, nơi bạn có thể bắt đầu cuộc trò chuyện với AI.

### 2. Tương Tác Cơ Bản Qua Dòng Lệnh và Cơ Chế Hoạt Động

Khi bạn ở trong giao diện Claude Code, bạn sẽ nhập các "prompt" (lời nhắc) của mình. Prompt là cách bạn giao tiếp với AI, mô tả nhiệm vụ, câu hỏi hoặc yêu cầu bạn muốn nó thực hiện.

**Cơ chế hoạt động (Under the Hood):**
Khi nhận một prompt, Claude Code không chỉ đơn thuần là phản hồi văn bản. Nó hoạt động như một "agent" (tác nhân) với một vòng lặp suy nghĩ và hành động:

1.  **Phân tích ngữ cảnh (Perceive):** Claude Code sẽ quét thư mục dự án hiện tại, đọc các file quan trọng như `package.json`, `tsconfig.json`, `README.md`, hoặc các file mã nguồn liên quan để xây dựng một bức tranh tổng thể về dự án và ngữ cảnh của prompt. Nó sử dụng các công cụ nội bộ như `read` (đọc file) để thu thập thông tin này.
2.  **Lập kế hoạch (Plan):** Dựa trên prompt và ngữ cảnh đã thu thập, AI sẽ lập một kế hoạch hành động. Kế hoạch này có thể bao gồm việc xác định các file cần chỉnh sửa, các lệnh shell cần thực thi (`bash`), hoặc các thông tin cần hỏi thêm.
3.  **Thực thi (Act):** AI sẽ thực hiện các hành động trong kế hoạch, sử dụng các công cụ `read`, `write` (ghi file), `bash` (thực thi lệnh shell) để tương tác với hệ thống file và môi trường.
4.  **Phản ánh và Tinh chỉnh (Reflect & Refine):** Sau khi thực hiện hành động, AI sẽ đánh giá kết quả. Nếu kết quả chưa đạt yêu cầu hoặc có lỗi, nó sẽ điều chỉnh kế hoạch và lặp lại quá trình.

**Ví dụ thực tế: Thay đổi nội dung trang chủ của ứng dụng Next.js**

Giả sử bạn có một dự án Next.js và muốn thay đổi trang chủ để hiển thị "Hello World" đơn giản.

```typescript
// pages/index.tsx (Nội dung gốc)
import Image from 'next/image'

export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <div className="z-10 w-full max-w-5xl items-center justify-between font-mono text-sm lg:flex">
        <p className="fixed left-0 top-0 flex w-full justify-center border-b border-gray-300 bg-gradient-to-b from-zinc-200 pb-6 pt-8 backdrop-blur-2xl dark:border-neutral-800 dark:bg-zinc-800/30 dark:from-inherit lg:static lg:w-auto  lg:rounded-xl lg:border lg:bg-gray-200 lg:p-4 lg:dark:bg-zinc-800/30">
          Get started by editing&nbsp;
          <code className="font-mono font-bold">app/page.tsx</code>
        </p>
        {/* ... các thành phần UI khác ... */}
      </div>

      <div className="relative flex place-items-center before:absolute before:h-[300px] before:w-[480px] before:-translate-x-1/2 before:rounded-full before:bg-gradient-radial before:from-white before:to-transparent before:blur-2xl before:content-[''] after:absolute after:-z-20 after:h-[180px] after:w-[240px] after:translate-x-1/3 after:bg-gradient-conic after:from-sky-200 after:via-blue-200 after:blur-2xl after:content-[''] before:dark:bg-gradient-to-br before:dark:from-transparent before:dark:to-blue-700 before:dark:opacity-10 after:dark:from-sky-900 after:dark:via-[#0141ff] after:dark:opacity-40 before:lg:h-[360px]">
        <Image
          className="relative dark:drop-shadow-[0_0_0.3rem_#ffffff70] dark:invert"
          src="/next.svg"
          alt="Next.js Logo"
          width={180}
          height={37}
          priority
        />
      </div>

      <div className="mb-32 grid text-center lg:mb-0 lg:grid-cols-4 lg:text-left">
        {/* ... các liên kết và mô tả khác ... */}
      </div>
    </main>
  )
}
```

Bạn có thể nhập prompt vào Claude Code như sau:
`replace the starting page of this Next.js project with a page that says "Hello World" centered in the middle of the screen`

Claude Code sẽ thực hiện các bước sau:

1.  **Phân tích ngữ cảnh:** Đọc `package.json` để xác định đây là dự án Next.js.
2.  **Tìm kiếm file liên quan:** Tìm kiếm các file `page.tsx` hoặc `index.tsx` trong thư mục `pages` hoặc `app`.
3.  **Đọc và hiểu:** Đọc nội dung file `pages/index.tsx` để hiểu cấu trúc hiện tại.
4.  **Đề xuất thay đổi:** Dựa trên prompt và ngữ cảnh, Claude Code sẽ tạo ra một phiên bản mới của file.

### 3. Quản Lý Quyền Hạn và Phê Duyệt Thay Đổi

Để đảm bảo tính bảo mật và kiểm soát, lần đầu tiên bạn sử dụng Claude Code trong một dự án hoặc khi nó đề xuất thay đổi file, nó sẽ không có quyền chỉnh sửa ngay lập tức. Claude Code sẽ hỏi bạn có muốn thực hiện thay đổi được đề xuất hay không.

> [!NOTE]
> Theo mặc định, Claude Code hoạt động ở chế độ yêu cầu xác nhận cho mỗi thay đổi. Bạn có thể thay đổi hành vi này trong cấu hình để cho phép tất cả các chỉnh sửa trong một phiên làm việc, điều này tiện lợi hơn cho các tác vụ phức tạp liên quan đến nhiều file.

Khi Claude Code đề xuất thay đổi, nó sẽ hiển thị một công cụ so sánh mã (diff view) để bạn có thể xem rõ sự khác biệt giữa phiên bản gốc và phiên bản mới.

**Ví dụ về Diff View:**

*   **Bên trái (Original):** Mã nguồn trước khi Claude Code chỉnh sửa.
*   **Bên phải (Proposed):** Mã nguồn sau khi Claude Code chỉnh sửa.

Bạn sẽ thấy các dòng bị xóa (thường là màu đỏ) và các dòng được thêm mới (thường là màu xanh lá cây).

```diff
--- a/pages/index.tsx
+++ b/pages/index.tsx
@@ -1,30 +1,12 @@
-import Image from 'next/image'
-
 export default function Home() {
   return (

-    <main className="flex min-h-screen flex-col items-center justify-between p-24">
-      <div className="z-10 w-full max-w-5xl items-center justify-between font-mono text-sm lg:flex">
-        <p className="fixed left-0 top-0 flex w-full justify-center border-b border-gray-300 bg-gradient-to-b from-zinc-200 pb-6 pt-8 backdrop-blur-2xl dark:border-neutral-800 dark:bg-zinc-800/30 dark:from-inherit lg:static lg:w-auto  lg:rounded-xl lg:border lg:bg-gray-200 lg:p-4 lg:dark:bg-zinc-800/30">
-          Get started by editing&nbsp;
-          <code className="font-mono font-bold">app/page.tsx</code>
-        </p>
-        {/* ... các thành phần UI khác ... */}
+    <div className="flex min-h-screen items-center justify-center">
+      <h1 className="text-6xl font-bold">Hello World</h1>
+    </div>
+  )
+}
```

Bạn có các tùy chọn sau để phản hồi:

*   **Accept (Chấp nhận):** Áp dụng các thay đổi.
*   **Reject (Từ chối):** Hủy bỏ các thay đổi.
*   **Continue in terminal (Tiếp tục trong terminal):** Trở lại terminal để chọn `yes` hoặc `yes, allow all edits during the session` (có, cho phép tất cả các chỉnh sửa trong phiên này). Lựa chọn thứ hai rất hữu ích để tránh phải xác nhận nhiều lần cho các tác vụ phức tạp.

**So sánh với Antigravity IDE:**
Trong khi Claude Code CLI yêu cầu xác nhận từng thay đổi để đảm bảo an toàn, Antigravity IDE, với tư cách là một hệ thống agentic siêu việt, thường hoạt động ở cấp độ lập kế hoạch và thực thi tự động cao hơn. Antigravity được thiết kế để *tự lập kế hoạch*, *tự chạy script ngầm*, *gọi subagent trình duyệt*, *đọc ghi file* và *phản ánh kết quả*. Điều này có nghĩa là Antigravity sẽ tự động thực hiện nhiều bước trung gian mà Claude Code CLI yêu cầu xác nhận. Tuy nhiên, Antigravity vẫn cung cấp một giao diện tổng quan hoặc một "diff view" ở cấp độ *kế hoạch* hoặc *commit* cuối cùng, cho phép người dùng xem xét và phê duyệt *chuỗi hành động* hoặc *tổng thể các thay đổi* trước khi chúng được đưa vào kho mã nguồn. Mục tiêu là giảm gánh nặng xác nhận từng bước nhỏ mà vẫn giữ được sự kiểm soát ở cấp độ chiến lược.

### 4. Tích Hợp Với Môi Trường Phát Triển (IDE) và Ứng Dụng Desktop

Mặc dù Claude Code là một công cụ CLI, nó có khả năng tích hợp mạnh mẽ với các IDE phổ biến và cũng có sẵn dưới dạng ứng dụng desktop.

#### a. Tiện ích mở rộng Visual Studio Code (VS Code Extension)

Để có trải nghiệm tích hợp tốt nhất với VS Code, bạn cần cài đặt tiện ích mở rộng chính thức của Claude Code từ Anthropic. Sau khi cài đặt, bạn sẽ nhận được các lợi ích sau:

*   **Diff View Trực Quan:** Thay vì xem diff trong terminal, VS Code sẽ mở một trình chỉnh sửa so sánh mã tích hợp, tương tự như khi bạn so sánh các commit Git. Điều này giúp bạn dễ dàng xem xét và phê duyệt các thay đổi với giao diện phong phú hơn.
*   **Tương Tác Ngoài Terminal:** Bạn có thể tương tác với Claude Code trực tiếp từ thanh lệnh của VS Code (Command Palette). Gõ `Claude Code` và chọn `Focus Input` để mở một cửa sổ trò chuyện riêng biệt. Tại đây, bạn có thể nhập prompt, đính kèm file, hoặc thậm chí dán hình ảnh vào phiên làm việc.
*   **Duyệt và Quản lý Phiên:** Cửa sổ này cũng cho phép bạn duyệt qua các cuộc trò chuyện trước đây, bắt đầu các phiên mới, và quản lý ngữ cảnh một cách trực quan hơn, cung cấp một "bảng điều khiển" cho các phiên AI của bạn.

> [!TIP]
> Dù có tích hợp IDE mạnh mẽ, phần lớn khóa học này sẽ tập trung vào giao diện CLI vì nó hoạt động độc lập với mọi trình chỉnh sửa và là phương thức sử dụng chính của nhiều nhà phát triển, mang lại sự linh hoạt tối đa.

#### b. Ứng Dụng Desktop Claude Code

Ngoài CLI và tích hợp IDE, Claude Code còn có sẵn dưới dạng ứng dụng desktop cho macOS và Windows. Ứng dụng này cung cấp một giao diện người dùng đồ họa (GUI) để tương tác với Claude Code.

**Các Chế Độ trong Ứng Dụng Desktop:**

*   **Chế độ Claude Code:** Đây là chế độ trọng tâm, cho phép bạn gửi và quản lý các tác vụ code, tương tự như CLI nhưng với giao diện đồ họa.
*   **Chế độ Chat thông thường:** Giống như ChatGPT, chế độ này dành cho các cuộc trò chuyện chung không liên quan đến lập trình.
*   **Claude Cowork:** Một công cụ trợ lý làm việc tổng quát khác của Anthropic.

**Sử Dụng Ứng Dụng Desktop:**

*   Bạn có thể bắt đầu các phiên code mới bằng cách nhấp vào "New Session".
*   Chọn một thư mục làm việc (dự án) hiện có hoặc mở một dự án mới.
*   Nhập prompt và gửi tác vụ đến Claude Code qua giao diện này.

> [!NOTE]
> Anthropic cũng cung cấp một ứng dụng di động Claude với các tính năng tương tự. Dù vậy, trọng tâm của chúng ta vẫn là CLI và các nguyên tắc cốt lõi áp dụng cho mọi giao diện.

**Antigravity IDE: Môi trường tích hợp agentic tối thượng:**
Hãy hình dung Antigravity IDE là một bước tiến vượt trội so với các tích hợp IDE thông thường. Nó không chỉ là một tiện ích mở rộng hay một ứng dụng desktop; nó là một *hệ điều hành phát triển* nơi các agent AI như Claude Code (hoặc các thành phần tương tự) hoạt động như những "bộ não" cục bộ hoặc từ xa. Antigravity không chỉ hiển thị diff, mà nó *tự động tạo, thực thi và đánh giá* các thay đổi thông qua các script ngầm và subagent trình duyệt. Nó có một "bộ nhớ dự án" liên tục và một hệ thống lập kế hoạch phức tạp, cho phép nó duy trì ngữ cảnh sâu sắc và thực hiện các tác vụ đa bước mà không cần sự can thiệp liên tục của người dùng. Tích hợp với Antigravity có nghĩa là Claude Code trở thành một công cụ trong bộ công cụ rộng lớn hơn, được điều phối bởi trí tuệ lập kế hoạch của Antigravity để đạt được các mục tiêu phức tạp hơn.

---

## II. Cấu Hình Claude Code: Tùy Biến Sâu và Quản Lý Quyền Hạn

Việc cấu hình Claude Code đúng cách là chìa khóa để tối ưu hóa hiệu suất, đảm bảo an toàn và cá nhân hóa trải nghiệm. Claude Code cung cấp nhiều cấp độ cấu hình, từ toàn cục trên hệ thống đến cấp độ dự án cụ thể.

### 1. Cấu Hình Toàn Cục và CLI (`/config` Command)

Cấu hình toàn cục ảnh hưởng đến tất cả các phiên làm việc của Claude Code trên hệ thống của bạn. Đây là nơi bạn đặt các cài đặt mặc định cho mọi dự án.

*   **Vị trí file:** Trên macOS, file cấu hình toàn cục thường nằm ở `~/.claude/settings.json` (trong thư mục home của người dùng hiện tại). Trên Windows, vị trí thường là `%USERPROFILE%\.claude\settings.json`.
*   **Nội dung:** File `settings.json` chứa các cài đặt chung như giao diện (theme), quyền hạn mặc định, và các tùy chọn hành vi của AI.

**Ví dụ về `settings.json` toàn cục:**

```json
{
  "theme": "dark",
  "alwaysThinkingEnabled": true, // Cho phép AI suy nghĩ sâu hơn trước khi phản hồi
  "defaultEditor": "vscode",     // Mở diff view trong VS Code mặc định
  "permissions": [
    {
      "tool": "read",
      "deny": ["**/.env", "**/*.log"] // Từ chối đọc file .env và log ở bất kỳ đâu
    },
    {
      "tool": "write",
      "deny": ["**/node_modules/**"] // Ngăn AI chỉnh sửa trong node_modules
    },
    {
      "tool": "bash",
      "deny": [] // Cho phép thực thi lệnh bash mặc định, nhưng có thể hạn chế
    }
  ]
}
```

**Cấu hình qua CLI (`/config` Command):**
Bạn có thể dễ dàng xem và thay đổi các cài đặt cấu hình trực tiếp từ giao diện CLI của Claude Code bằng lệnh `/config`.
Khi bạn chạy `/config`, một menu cấu hình sẽ hiện ra, cho phép bạn duyệt qua các cài đặt hiện có và giá trị của chúng. Bạn có thể thay đổi các giá trị này một cách tương tác.

**Ví dụ:** Thay đổi theme hoặc tắt chế độ "thinking mode".

*   Chọn `theme` để thay đổi giao diện.
*   Chọn `alwaysThinkingEnabled` và đặt thành `false` nếu bạn muốn Claude Code phản hồi nhanh hơn cho các tác vụ đơn giản, tiết kiệm token, nhưng có thể ít "suy nghĩ" sâu hơn.

> [!NOTE]
> Bất kỳ thay đổi nào bạn thực hiện thông qua lệnh `/config` đều sẽ được lưu tự động vào file `settings.json` toàn cục của bạn. Điều này đảm bảo tính nhất quán giữa các phiên làm việc.

**Tư duy Vibe Coding: Cấu hình là thiết lập "Vibe" cơ bản**
Cấu hình toàn cục không chỉ là các cài đặt kỹ thuật; nó là việc thiết lập "vibe" cơ bản cho cách Claude Code tương tác với bạn. Ví dụ, `alwaysThinkingEnabled: true` tạo ra một "vibe" của sự cẩn trọng và suy nghĩ sâu, phù hợp cho các tác vụ phức tạp. Ngược lại, `false` tạo ra một "vibe" nhanh nhẹn, phù hợp cho các chỉnh sửa nhỏ. Các quyền hạn cũng thiết lập "vibe" về sự tin cậy và giới hạn an toàn, định hình cách AI cảm nhận ranh giới của nó.

#### Danh sách các Cài Đặt Khả Dụng

Claude Code có một danh sách dài các cài đặt có thể cấu hình, và danh sách này có thể thay đổi theo thời gian. Bạn nên tham khảo tài liệu chính thức của Anthropic để xem phiên bản cập nhật nhất của danh sách này. Các cài đặt thường bao gồm:

*   `theme`: Giao diện (dark/light).
*   `model`: Mô hình AI sử dụng (ví dụ: `claude-3-opus-20240229`).
*   `defaultEditor`: Trình soạn thảo mặc định cho diff (ví dụ: `vscode`, `vim`).
*   `alwaysThinkingEnabled`: Chế độ suy nghĩ sâu.
*   `permissions`: Các quy tắc quyền hạn.
*   `telemetryEnabled`: Có gửi dữ liệu sử dụng ẩn danh hay không.

### 2. Quản Lý Quyền Hạn (Permissions) Sâu Hơn

Đây là một cài đặt cực kỳ quan trọng để đảm bảo an toàn và bảo mật cho dự án của bạn. Hệ thống quyền hạn của Claude Code cho phép bạn kiểm soát những gì AI có thể đọc, ghi hoặc thực thi trên hệ thống file của bạn.

**Các công cụ chính và quyền hạn liên quan:**

*   `read`: Cho phép Claude Code đọc các file.
*   `write`: Cho phép Claude Code ghi vào các file.
*   `bash`: Cho phép Claude Code thực thi các lệnh shell (bash).

Bạn có thể chỉ định các quy tắc `deny` (từ chối) để ngăn Claude Code sử dụng các công cụ này trên các file hoặc thư mục cụ thể. Các quy tắc này hỗ trợ glob patterns (ví dụ: `**/*.env`).

**Ví dụ về cấu hình quyền hạn an toàn nâng cao:**
Một trong những quy tắc quan trọng nhất là từ chối Claude Code đọc các file `.env` của bạn, nơi thường chứa các khóa API, thông tin đăng nhập cơ sở dữ liệu và các bí mật nhạy cảm khác. Tương tự, bạn không muốn AI tự ý chỉnh sửa các thư mục `node_modules` hoặc `dist` đã được tạo ra trong quá trình build.

```json
{
  "permissions": [
    {
      "tool": "read",
      "deny": [
        "**/.env",          // Từ chối đọc tất cả file .env
        "**/secrets.json",  // Từ chối đọc các file secrets.json
        "**/node_modules/**", // Tránh đọc các file trong node_modules (thường rất lớn và không cần thiết)
        "**/dist/**",       // Tránh đọc các file đã build
        "**/.git/**"        // Tránh đọc các file .git nội bộ
      ]
    },
    {
      "tool": "write",
      "deny": [
        "**/node_modules/**", // Ngăn chặn ghi vào node_modules
        "**/dist/**",       // Ngăn chặn ghi vào thư mục build
        "**/.git/**",       // Ngăn chặn ghi vào repo Git
        "**/package-lock.json", // Tránh chỉnh sửa package-lock tự động
        "**/yarn.lock"
      ]
    },
    {
      "tool": "bash",
      "deny": [
        "rm -rf",           // Ngăn chặn các lệnh xóa nguy hiểm
        "sudo",             // Ngăn chặn leo thang quyền
        "format c:",        // Các lệnh hệ thống phá hoại (ví dụ)
        "npm publish",      // Ngăn chặn AI tự ý publish gói
        "git push --force"  // Ngăn chặn các hành động Git nguy hiểm
      ]
    }
  ]
}
```

> [!WARNING]
> Luôn cẩn thận khi cấu hình quyền hạn. Việc cấp quá nhiều quyền có thể dẫn đến rủi ro bảo mật nghiêm trọng (ví dụ: AI vô tình xóa file quan trọng hoặc tiết lộ bí mật), trong khi cấp quá ít quyền có thể hạn chế khả năng làm việc của Claude Code, khiến nó không thể hoàn thành các tác vụ yêu cầu truy cập tài nguyên.

**Antigravity IDE và Quản lý Quyền hạn (Agentic Safety):**
Trong Antigravity IDE, quản lý quyền hạn được thực hiện ở một cấp độ tinh vi hơn. Thay vì chỉ là các quy tắc `deny` cứng nhắc, Antigravity sử dụng một hệ thống "Agentic Safety" dựa trên việc lập kế hoạch và phản ánh. Các agent của Antigravity có thể có quyền truy cập rộng hơn vào hệ thống file để thu thập ngữ cảnh, nhưng chúng được huấn luyện để *tự đánh giá rủi ro* của các hành động ghi hoặc thực thi.

*   **Internal Guardrails:** Antigravity có các "guardrails" nội bộ, các mô hình phụ trách an toàn sẽ đánh giá mỗi bước trong kế hoạch của agent trước khi thực thi, đặc biệt là các hành động có khả năng phá hoại (`write`, `bash`).
*   **Human-in-the-Loop at Strategy Level:** Thay vì phê duyệt từng diff, Antigravity có thể yêu cầu phê duyệt ở cấp độ "kế hoạch hành động" lớn hơn, hoặc trước khi thực hiện một "commit" cuối cùng vào kho mã nguồn. Điều này cho phép người dùng kiểm soát chiến lược mà không bị quá tải bởi các chi tiết nhỏ.
*   **Sandboxing:** Đối với các tác vụ rủi ro cao, Antigravity có thể tự động chạy các agent trong môi trường sandbox (hộp cát) để cô lập các thay đổi và ngăn chặn tác động không mong muốn lên hệ thống chính.

### 3. Cấu Hình Cấp Độ Dự Án (Project-Specific Configuration)

Ngoài cấu hình toàn cục, bạn có thể tạo cấu hình cụ thể cho từng dự án. Điều này rất hữu ích khi bạn muốn Claude Code có hành vi khác nhau tùy thuộc vào dự án bạn đang làm việc (ví dụ: một dự án mã nguồn mở có thể có các quy tắc quyền hạn lỏng lẻo hơn so với một dự án bí mật của công ty).

*   **Vị trí file:** Trong thư mục gốc của dự án của bạn, tạo một thư mục `.claude` và bên trong đó, tạo file `settings.json`.
    *   `your-project-folder/.claude/settings.json`
*   **Ưu tiên:** Cài đặt trong file `settings.json` cấp độ dự án sẽ ghi đè các cài đặt toàn cục. Điều này có nghĩa là nếu một cài đặt được định nghĩa ở cả hai nơi, cài đặt trong dự án sẽ được áp dụng.

#### `settings.local.json`

Để có tính linh hoạt cao hơn, bạn cũng có thể tạo file `settings.local.json` trong thư mục `.claude` của dự án:

*   `your-project-folder/.claude/settings.local.json`
*   **Mục đích:** File này được thiết kế để chứa các cài đặt cá nhân của bạn mà *không nên được kiểm tra vào hệ thống kiểm soát phiên bản (như Git)*. Điều này rất hữu ích khi làm việc trong một nhóm, cho phép mỗi thành viên có các tùy chỉnh riêng (ví dụ: mô hình AI ưa thích, theme cá nhân) mà không ảnh hưởng đến người khác hoặc kho mã nguồn chung. Bạn nên thêm `/.claude/settings.local.json` vào file `.gitignore` của dự án.
*   **Ưu tiên:** Cài đặt trong `settings.local.json` sẽ ghi đè cả cài đặt toàn cục và cài đặt trong `settings.json` cấp độ dự án. Đây là cấp độ ưu tiên cao nhất.

**Thứ tự ưu tiên của cấu hình:**
`settings.local.json` (dự án) > `settings.json` (dự án) > `settings.json` (toàn cục)

Điều này đảm bảo rằng bạn có thể tinh chỉnh hành vi của Claude Code từ mức độ chung nhất đến mức độ cụ thể nhất cho từng cá nhân và từng dự án.

---

## III. Quản Lý Phiên, Ngữ Cảnh và Tư Duy Vibe Coding

Hiểu cách Claude Code quản lý "phiên" (session) và "ngữ cảnh" (context) là rất quan trọng để sử dụng nó một cách hiệu quả và áp dụng tư duy Vibe Coding. Mỗi tương tác với Claude Code diễn ra trong một phiên, và phiên đó có một ngữ cảnh riêng, giống như trí nhớ của AI về cuộc trò chuyện hiện tại.

### 1. Các Lệnh Slash Cơ Bản để Quản Lý Ngữ Cảnh

Trong giao diện CLI của Claude Code, bạn có thể sử dụng các "lệnh slash" (slash commands) để thực hiện các tác vụ quản lý.

*   `/clear`: Xóa giao diện terminal và, quan trọng hơn, xóa toàn bộ ngữ cảnh của phiên hiện tại. Điều này giống như bắt đầu một cuộc trò chuyện hoàn toàn mới với AI, một "vibe reset" hoàn toàn.
    *   **Khi sử dụng:** Hữu ích khi AI bị "kẹt" trong một vòng lặp không mong muốn, không thể giải quyết vấn đề, hoặc khi bạn muốn bắt đầu một nhiệm vụ hoàn toàn khác và không muốn ngữ cảnh cũ ảnh hưởng đến sự "suy nghĩ" của AI.
*   `/context`: Hiển thị thông tin chi tiết về cách cửa sổ ngữ cảnh của phiên hiện tại đang được sử dụng. Đây là công cụ quan trọng để hiểu "trí nhớ" của AI.
*   `/usage`: Hiển thị mức sử dụng token còn lại của bạn dựa trên gói dịch vụ Claude Code hiện tại. Hữu ích để theo dõi chi phí và tránh vượt quá giới hạn.

### 2. Hiểu Rõ Cửa Sổ Ngữ Cảnh (Context Window) và Bộ Đệm Tự Động Nén

Cửa sổ ngữ cảnh là "bộ nhớ" của AI cho một phiên làm việc cụ thể. Nó chứa tất cả các tương tác (prompt và phản hồi) trong phiên đó, cũng như các thông tin hệ thống mà AI cần để hoạt động. Kích thước của cửa sổ ngữ cảnh được giới hạn bởi số lượng token mà mô hình AI có thể xử lý.

Khi bạn chạy lệnh `/context`, bạn sẽ thấy một phân tích chi tiết về việc sử dụng token:

*   **System Prompt:** Một phần đáng kể token được dành cho prompt hệ thống do các kỹ sư của Anthropic thiết lập. Prompt này là "lời hướng dẫn" cấp cao nhất, định hướng hành vi cơ bản của Claude Code (ví dụ: "Bạn là một trợ lý lập trình, hãy giúp người dùng viết code, sửa lỗi, và trả lời các câu hỏi kỹ thuật"). Nó thiết lập "vibe" mặc định của AI.
*   **System Tools:** Mô tả các công cụ nội bộ mà Claude Code có thể sử dụng (ví dụ: `read`, `write`, `bash`). Những mô tả này giúp AI biết khi nào nên sử dụng công cụ nào và cách sử dụng chúng. Mỗi mô tả công cụ cũng tiêu tốn token.
*   **MCP Tools:** Nếu bạn đã cấu hình các máy chủ MCP (một tính năng nâng cao hơn để tích hợp các công cụ tùy chỉnh), một phần token cũng sẽ được dành cho mô tả các công cụ đó.
*   **Free Space:** Không gian token còn lại bạn có thể sử dụng cho các prompt và phản hồi của mình trong phiên hiện tại.

**Ví dụ về đầu ra của `/context`:**

```
Context Window Usage:
  Total Tokens: 200000 (Claude Code Opus)
  Used: 20000 (10%)
  Free: 180000 (90%)

Breakdown:

  - System Prompt: 10000 tokens (The core instructions for Claude Code)
  - System Tools: 9000 tokens (Descriptions of read, write, bash tools)
  - MCP Tools: 1000 tokens (if configured, descriptions of custom tools)
  - Auto Compact Buffer (Reserved): 20000 tokens (cannot be used for new prompts, reserved for compaction)
  - Your Conversation: 0 tokens (since just cleared, or the actual conversation history)
```

> [!NOTE]
> Các mô hình AI hiện đại như Claude Code Opus có cửa sổ ngữ cảnh rất lớn (ví dụ: 200,000 token hoặc hơn), cho phép nó "ghi nhớ" một lượng lớn thông tin. Tuy nhiên, việc quản lý ngữ cảnh vẫn rất quan trọng để tối ưu hóa hiệu suất và chi phí.

#### Bộ Đệm Tự Động Nén (Auto Compact Buffer)

Khi một phiên làm việc trở nên dài và tiêu thụ nhiều token, Claude Code sẽ tự động "nén" (compact) ngữ cảnh để giữ cho nó nằm trong giới hạn của cửa sổ ngữ cảnh.

*   **Cách hoạt động:** Claude Code sẽ tạo một bản tóm tắt các công việc đã được thực hiện và các công việc còn lại cần làm. Sau đó, nó sẽ loại bỏ (hoặc nén một cách mất mát) các phần ngữ cảnh cũ và chỉ giữ lại bản tóm tắt này, tiếp tục dựa trên đó.
*   **Mục đích:** Đảm bảo rằng Claude Code không bị hết bộ nhớ và có thể tiếp tục làm việc trên các tác vụ dài mà không mất đi ngữ cảnh quan trọng, dù có thể mất đi một số chi tiết nhỏ.
*   **Reserved Buffer:** Một phần của cửa sổ ngữ cảnh được "dành riêng" cho quá trình nén tự động. Phần này không thể được sử dụng cho các prompt thông thường của bạn mà là không gian làm việc nội bộ của AI để quản lý bộ nhớ.

**Vibe Coding: Quản lý ngữ cảnh để duy trì "Vibe"**
Quản lý ngữ cảnh là cốt lõi của Vibe Coding. Một "vibe" tốt cho AI là một ngữ cảnh rõ ràng, không bị nhiễu và tập trung vào nhiệm vụ hiện tại.

*   **`/clear` là "Vibe Reset":** Khi AI bắt đầu lạc đề hoặc "quên" mục tiêu ban đầu, `/clear` giúp bạn thiết lập lại "vibe" từ đầu, đảm bảo AI tập trung vào nhiệm vụ mới mà không bị ảnh hưởng bởi các cuộc trò chuyện cũ.
*   **Tạo Prompt Hiệu Quả:** Khi cửa sổ ngữ cảnh đầy, AI sẽ phải nén thông tin. Để duy trì "vibe" mong muốn, hãy đảm bảo các prompt của bạn luôn ngắn gọn, rõ ràng và truyền tải được ý định cốt lõi. Hãy coi nó như việc viết một bản tóm tắt điều hành cho AI.
*   **Hiểu Giới Hạn:** Nhận thức về giới hạn token giúp bạn thiết kế các tác vụ phù hợp. Đối với các tác vụ rất dài, bạn có thể cần chia nhỏ chúng thành các phần nhỏ hơn, mỗi phần có một "vibe" riêng, và sử dụng `/clear` giữa các phần.

**Antigravity IDE: Quản lý ngữ cảnh tiên tiến hơn**
Antigravity IDE với kiến trúc multi-agent và hệ thống bộ nhớ bền vững (persistent memory) có khả năng quản lý ngữ cảnh vượt trội.

*   **Persistent Project Memory:** Antigravity duy trì một "bộ nhớ dự án" dài hạn, không chỉ giới hạn trong một phiên chat. Điều này giúp nó "ghi nhớ" cấu trúc dự án, các quyết định thiết kế trước đó, và các vấn đề đã giải quyết qua nhiều phiên làm việc.
*   **Multi-Agent Context:** Các sub-agent trong Antigravity có thể có cửa sổ ngữ cảnh riêng, chuyên biệt cho từng nhiệm vụ (ví dụ: một agent chuyên về UI, một agent chuyên về database). Một agent điều phối trung tâm tổng hợp thông tin từ các agent này, giảm thiểu sự cần thiết của auto-compaction mất mát ở cấp độ tổng thể.
*   **Contextual Planning:** Antigravity sử dụng ngữ cảnh để lập kế hoạch phức tạp. Nó không chỉ tóm tắt các cuộc trò chuyện, mà còn phân tích các file đã chỉnh sửa, các lỗi đã gặp, và các mục tiêu chưa hoàn thành để xây dựng một "bản đồ tư duy" liên tục về dự án. Điều này giúp nó duy trì "vibe" của dự án qua thời gian dài.

### 3. Quản Lý Nhiều Phiên Song Song

Bạn có thể chạy nhiều phiên Claude Code song song. Điều này có thể được thực hiện bằng cách mở nhiều cửa sổ terminal khác nhau và khởi động Claude Code trong mỗi cửa sổ.

*   **Lợi ích:** Cho phép bạn làm việc trên các tác vụ khác nhau hoặc thử nghiệm các giải pháp khác nhau cho cùng một dự án mà không ảnh hưởng đến ngữ cảnh của các phiên khác. Ví dụ, một phiên có thể tập trung vào refactoring, trong khi phiên khác tập trung vào việc viết bài kiểm tra.
*   **Lưu ý:** Nếu bạn đang làm việc trên cùng một dự án với nhiều phiên, hãy đảm bảo các tác vụ của AI không xung đột hoặc ghi đè lên nhau, đặc biệt là khi chỉnh sửa cùng một file. Sử dụng tính năng diff view và phê duyệt cẩn thận là rất quan trọng.

**Antigravity IDE: Tương tác đa tác vụ một cách tự nhiên**
Trong Antigravity IDE, khái niệm "nhiều phiên song song" được tích hợp một cách tự nhiên vào kiến trúc của nó. Antigravity được thiết kế để quản lý nhiều tác vụ đồng thời thông qua các agent hoặc luồng công việc riêng biệt. Bạn có thể giao cho Antigravity một mục tiêu lớn, và nó sẽ tự động phân chia thành các sub-task, giao cho các agent khác nhau, và quản lý ngữ cảnh cho từng sub-task một cách riêng biệt. Điều này tạo ra một môi trường làm việc "đa nhiệm" cho AI mà không cần người dùng phải quản lý các terminal riêng lẻ.

---

## IV. Các Tính Năng Nâng Cao và Ứng Dụng Thực Tế

Ngoài các tương tác và cấu hình cơ bản, Claude Code còn có một số tính năng cốt lõi giúp nâng cao hiệu quả làm việc, đặc biệt khi áp dụng tư duy Vibe Coding.

### 1. Gọi Claude Code với Prompt Ban Đầu và Chế Độ Không Tương Tác

Thay vì chạy lệnh `claude` và sau đó nhập prompt vào giao diện tương tác, bạn có thể cung cấp prompt ban đầu ngay lập tức khi gọi Claude Code.

**Cú pháp:** `claude <your_initial_prompt>`

**Ví dụ:**
`claude explain the main functionalities of this project to me`

Điều này sẽ khởi động Claude Code, nhập prompt đã cho và bắt đầu tạo phản hồi ngay lập tức, giúp bạn tiết kiệm thời gian. Claude Code vẫn sẽ vào chế độ tương tác sau khi đưa ra phản hồi đầu tiên, cho phép bạn tinh chỉnh hoặc tiếp tục cuộc trò chuyện.

#### Chế Độ Không Tương Tác (Non-Interactive Mode)

Nếu bạn chỉ muốn Claude Code thực hiện một tác vụ và trả về kết quả mà không cần duy trì phiên tương tác, bạn có thể sử dụng cờ `-p` (print).

**Cú pháp:** `claude -p <your_prompt>`

**Ví dụ:**
`claude -p summarize the main functionalities of this project in 3 bullet points`

Claude Code sẽ chạy tác vụ trong nền, thực hiện các phân tích và chỉnh sửa (nếu được cấp quyền), và sau đó in phản hồi trực tiếp ra terminal của bạn, mà không mở giao diện tương tác. Điều này rất hữu ích cho các tác vụ nhanh, tự động hóa bằng script, hoặc khi bạn muốn tích hợp Claude Code vào các pipeline CI/CD.

### 2. Claude Code: Hơn Cả Chỉnh Sửa Mã – Tư Duy Vibe Coding Thực Chiến

Một quan niệm sai lầm phổ biến là Claude Code chỉ dùng để viết hoặc sửa đổi mã. Trên thực tế, khả năng của nó rộng hơn nhiều, và việc khai thác tối đa những khả năng này đòi hỏi một tư duy tương tác đặc biệt: **Vibe Coding**.

#### Vibe Coding là gì?

Vibe Coding không chỉ là việc đưa ra các hướng dẫn rõ ràng cho AI; đó là việc truyền đạt *ý định, phong cách, ràng buộc và kết quả mong muốn* – tức là "vibe" của công việc – để AI có thể tự định hướng và đưa ra các giải pháp sáng tạo, phù hợp nhất. Nó là việc thiết lập một "tâm trạng" hoặc "bối cảnh" toàn diện cho AI, cho phép nó hành động một cách thông minh hơn, chủ động hơn.

**Các Nguyên Tắc Cốt Lõi của Vibe Coding:**

1.  **Clarity & Specificity (Rõ ràng & Cụ thể):** Luôn bắt đầu với mục tiêu rõ ràng.
    *   *Thay vì:* "Sửa cái này."
    *   *Hãy dùng:* "Refactor hàm `calculateTotal` để sử dụng cách tiếp cận Functional Programming, đảm bảo tính bất biến và dễ kiểm thử."

2.  **Contextual Richness (Ngữ cảnh phong phú):** Cung cấp đủ thông tin để AI hiểu sâu vấn đề.
    *   *Thay vì:* "Hàm này bị lỗi."
    *   *Hãy dùng:* "Hàm `processOrder(orderId)` đang gặp lỗi khi `orderId` không hợp lệ. Đây là mã nguồn của hàm và đây là schema database của bảng `Orders`. Hãy tìm lỗi và đề xuất cách xử lý ngoại lệ."

3.  **Iterative Refinement (Tinh chỉnh lặp đi lặp lại):** Đừng mong đợi sự hoàn hảo ngay từ đầu. Hãy coi AI là một đồng nghiệp cần được hướng dẫn.
    *   *Prompt 1:* "Viết một hàm `authenticateUser`."
    *   *AI phản hồi:* (Viết hàm cơ bản)
    *   *Prompt 2:* "Hàm này cần tích hợp với OAuth2 và sử dụng chiến lược JWT. Thêm logic refresh token."

4.  **Persona & Role-playing (Vai trò & Nhập vai):** Đôi khi, việc yêu cầu AI đóng một vai trò cụ thể có thể định hướng phong cách và độ sâu của phản hồi.
    *   *Ví dụ:* "Hãy đóng vai một kiến trúc sư hệ thống cấp cao. Đánh giá thiết kế microservices hiện tại của tôi và đề xuất các cải tiến về khả năng mở rộng và bảo mật."

5.  **Desired Output Format (Định dạng đầu ra mong muốn):** Yêu cầu AI trả về kết quả theo một cấu trúc cụ thể.
    *   *Ví dụ:* "Liệt kê các điểm yếu của mã này dưới dạng danh sách bullet points, sau đó đề xuất các giải pháp dưới dạng một bảng với các cột 'Vấn đề', 'Giải pháp', 'Ưu tiên'."

6.  **Implicit vs. Explicit Constraints (Ràng buộc ngầm và rõ ràng):**
    *   *Ngầm:* "Hãy viết mã hiệu suất cao." (AI sẽ tự động tối ưu)
    *   *Rõ ràng:* "Đảm bảo thời gian phản hồi của API dưới 50ms trong điều kiện tải 1000 request/giây."

**Ứng Dụng Thực Tế của Vibe Coding với Claude Code:**

*   **Khám phá mã và Giải thích dự án:**
    *   `claude explain the architecture of this project, focusing on how data flows from the frontend to the database.`
    *   `claude act as a new team member. Walk me through the purpose of the 'UserService' and its main methods in this codebase.`
*   **Hỏi đáp và Tìm hiểu khái niệm:**
    *   `claude what are the pros and cons of using GraphQL vs REST for this specific project structure? (Consider the existing data models)`
    *   `claude explain the concept of 'event sourcing' in the context of a payment processing system.`
*   **Thảo luận giải pháp và Đánh giá thiết kế:**
    *   `claude I'm trying to implement a caching layer for this API. What are 3 different strategies I could use, and what are their trade-offs for a high-traffic e-commerce site?`
    *   `claude review this pull request (link/paste code). Focus on potential security vulnerabilities and suggest improvements.`
*   **Tạo tài liệu và Comment:**
    *   `claude generate JSDoc comments for all functions in 'utils.js' based on their implementation.`
    *   `claude write a README.md section explaining how to set up the local development environment for this project.`
*   **Refactoring và Tối ưu hóa:**
    *   `claude refactor this monolithic component into smaller, reusable React components, ensuring clean prop drilling and state management.`
    *   `claude identify performance bottlenecks in this SQL query and suggest optimized alternatives.`

> [!TIP]
> Hãy coi Claude Code như một đồng nghiệp thông minh có thể đọc, hiểu, suy nghĩ về mã của bạn và thậm chí tự học hỏi từ ngữ cảnh. Đừng ngại hỏi nó những câu hỏi không liên quan trực tiếp đến việc chỉnh sửa mã, mà là về tư vấn, phân tích hoặc định hướng. Vibe Coding chính là nghệ thuật giao tiếp hiệu quả với đồng nghiệp AI này.

**Vibe Coding trong Antigravity IDE:**
Vibe Coding trở thành *cực kỳ quan trọng* và mạnh mẽ hơn nhiều khi làm việc với Antigravity IDE. Vì Antigravity là một hệ thống agentic tự động hóa cao, việc thiết lập "vibe" ban đầu một cách chính xác sẽ cho phép nó:

*   **Tự Lập Kế Hoạch Sâu:** Antigravity sẽ sử dụng "vibe" để xây dựng một kế hoạch hành động chi tiết, bao gồm việc gọi các subagent, chạy các script ngầm, và thậm chí tự điều chỉnh mục tiêu phụ.
*   **Thực Thi Tự Động Hóa Cao:** Với một "vibe" rõ ràng, Antigravity có thể thực thi toàn bộ chuỗi tác vụ mà không cần can thiệp liên tục của người dùng, từ việc đọc file, tìm kiếm trên trình duyệt, viết mã, chạy thử nghiệm, đến việc gửi pull request.
*   **Phản Ánh và Tự Sửa Lỗi:** "Vibe" ban đầu cũng định hướng cách Antigravity tự phản ánh về kết quả. Nếu kết quả không khớp với "vibe" mong muốn (ví dụ: mã không hiệu suất cao, không tuân thủ phong cách), nó sẽ tự động lặp lại và sửa lỗi.
*   **Delegation of Vision:** Với Antigravity, bạn không chỉ giao một nhiệm vụ, bạn giao một "tầm nhìn" về kết quả mong muốn. Vibe Coding là cách bạn truyền đạt tầm nhìn đó, cho phép Antigravity biến nó thành hiện thực với sự tự chủ cao nhất.

### 3. Khôi Phục và Tiếp Tục Phiên Làm Việc

Đôi khi, bạn có thể cần quay lại một phiên làm việc cũ do sự cố, đóng nhầm hoặc đơn giản là muốn tiếp tục từ một điểm cụ thể. Claude Code cung cấp các lệnh để quản lý điều này.

*   `/resume`: Lệnh này cho phép bạn duyệt qua danh sách các phiên làm việc đã lưu trước đó. Bạn có thể chọn một phiên bất kỳ để tiếp tục, và Claude Code sẽ tải lại toàn bộ ngữ cảnh của phiên đó.
    *   Hữu ích khi bạn muốn quay lại một công việc cụ thể hoặc xem lại một cuộc trò chuyện cũ để tham khảo hoặc tiếp tục.
*   `claude -c`: Nếu bạn chỉ muốn tiếp tục phiên làm việc cuối cùng mà bạn đã tương tác, bạn có thể khởi động Claude Code với cờ `-c`.
    *   Rất tiện lợi để nhanh chóng quay lại công việc đang dang dở mà không cần phải duyệt qua danh sách.

**Ví dụ:**
`claude -c` // Khởi động Claude Code và tiếp tục phiên cuối cùng.

**Antigravity IDE: Quản lý trạng thái dự án liên tục**
Antigravity IDE, với tính chất agentic và "project memory" liên tục, vượt xa việc chỉ "khôi phục phiên". Nó duy trì một trạng thái dự án liên tục, bao gồm các kế hoạch chưa hoàn thành, các vấn đề đã biết, và các mục tiêu đang được theo đuổi. Khi bạn mở Antigravity IDE, nó không chỉ tải lại một phiên chat, mà nó tải lại *toàn bộ trạng thái làm việc của dự án*, cho phép các agent tiếp tục chính xác từ nơi chúng dừng lại, duy trì "vibe" và ngữ cảnh của công việc. Điều này giảm đáng kể nhu cầu sử dụng các lệnh `/resume` thủ công.

---

## V. Tóm Tắt Chương

Chương này đã cung cấp một cái nhìn toàn diện về Claude Code, từ những tương tác cơ bản đến các tính năng nâng cao và cấu hình sâu. Dưới đây là những điểm cốt lõi bạn cần nắm vững:

*   **Tương Tác Cơ Bản & Cơ Chế Agentic:** Claude Code là một công cụ CLI tương tác qua prompt, hoạt động theo vòng lặp Perceive -> Plan -> Act -> Reflect. Nó phân tích ngữ cảnh dự án (ví dụ: `package.json`) và sử dụng các công cụ nội bộ (`read`, `write`, `bash`) để hiểu và đề xuất thay đổi.
*   **Quản Lý Thay Đổi An Toàn:** Claude Code hiển thị diff view cho các thay đổi được đề xuất và yêu cầu xác nhận. Bạn có thể chấp nhận, từ chối hoặc cho phép tất cả các thay đổi trong phiên.
*   **Tích Hợp Môi Trường Phát Triển:** Có tiện ích mở rộng Visual Studio Code chính thức cung cấp diff view trực quan, tương tác qua Command Palette và quản lý phiên. Ứng dụng desktop cũng cung cấp giao diện GUI với các chế độ code, chat và Cowork. **Antigravity IDE** được giới thiệu như một hệ thống agentic tối thượng, tích hợp sâu hơn và tự động hóa cao hơn trong việc quản lý tác vụ và ngữ cảnh.
*   **Cấu Hình Linh Hoạt:**
    *   **Toàn cục:** Thông qua `~/.claude/settings.json` (Mac) hoặc lệnh `/config` trong CLI.
    *   **Cấp độ dự án:** Qua `.claude/settings.json` và `.claude/settings.local.json` (ghi đè cài đặt toàn cục), cho phép tùy biến theo dự án và cá nhân.
    *   **Quản lý Quyền hạn (Permissions):** Cực kỳ quan trọng để kiểm soát `read`, `write`, `bash` và từ chối truy cập các file nhạy cảm như `.env` hoặc các thư mục hệ thống. Antigravity IDE sử dụng "Agentic Safety" và lập kế hoạch để quản lý quyền hạn thông minh hơn.
*   **Quản Lý Phiên và Ngữ Cảnh:**
    *   Mỗi tương tác diễn ra trong một phiên với cửa sổ ngữ cảnh riêng (bộ nhớ của AI).
    *   Các lệnh slash như `/clear` (xóa ngữ cảnh), `/context` (kiểm tra sử dụng token), `/usage` (kiểm tra mức sử dụng).
    *   **Bộ Đệm Tự Động Nén:** Claude Code tự động nén ngữ cảnh dài để quản lý giới hạn token, dù có thể mất mát chi tiết. Antigravity IDE có khả năng quản lý ngữ cảnh tiên tiến hơn với bộ nhớ bền vững và kiến trúc multi-agent.
    *   Hỗ trợ nhiều phiên song song để làm việc trên các tác vụ khác nhau.
*   **Tính Năng Nâng Cao & Tư Duy Vibe Coding:**
    *   **Gọi trực tiếp:** `claude <prompt>` để khởi động với prompt ban đầu.
    *   **Chế độ không tương tác:** `claude -p <prompt>` để chạy tác vụ trong nền và in kết quả, hữu ích cho tự động hóa.
    *   **Vibe Coding:** Một triết lý tương tác với AI, không chỉ đưa ra lệnh mà còn truyền đạt ý định, phong cách, ràng buộc và kết quả mong muốn ("vibe"). Vibe Coding là chìa khóa để khai thác tối đa khả năng của Claude Code và đặc biệt quan trọng cho các hệ thống agentic như Antigravity IDE để đạt được sự tự chủ và hiệu quả cao nhất.
    *   **Đa năng:** Claude Code không chỉ để chỉnh sửa mã, mà còn để khám phá, hỏi đáp, thảo luận giải pháp, tạo tài liệu, refactoring và tối ưu hóa.
    *   **Khôi phục phiên:** `/resume` để duyệt và tiếp tục các phiên cũ, hoặc `claude -c` để tiếp tục phiên cuối cùng, trong khi Antigravity IDE duy trì trạng thái dự án liên tục.

Với nền tảng vững chắc này về cách sử dụng, cấu hình và tư duy Vibe Coding với Claude Code, bạn đã sẵn sàng khám phá sâu hơn các khả năng nâng cao và áp dụng công cụ mạnh mẽ này vào các dự án thực tế của mình, đồng thời chuẩn bị cho tương lai của phát triển phần mềm với các hệ thống AI Agentic như Antigravity IDE.

<!-- REVIEWED_BY_AGENT -->
