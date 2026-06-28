# Phần 7: Kỹ Năng Agent và Tùy Biến Lệnh

Trong kỷ nguyên phát triển phần mềm hiện đại, việc tận dụng trí tuệ nhân tạo để nâng cao năng suất và chất lượng lập trình đã trở thành một xu hướng tất yếu. Claude Code, một công cụ AI mạnh mẽ được phát triển bởi Anthropic, nổi bật với khả năng hiểu sâu codebase và tương tác thông minh với dự án của bạn. Để tối ưu hóa tiềm năng của Claude Code, khái niệm "Kỹ Năng Agent" (Agent Skills) được giới thiệu, cho phép bạn trang bị cho AI ngữ cảnh động, các hướng dẫn cụ thể và thậm chí là các đoạn mã thực thi, biến nó thành một trợ lý lập trình thực sự tinh vi.

Chương này sẽ đi sâu vào cách bạn có thể định nghĩa, quản lý và sử dụng các kỹ năng agent tùy chỉnh để huấn luyện Claude Code, giúp nó nắm bắt các quy tắc, best practices và yêu cầu đặc thù của dự án. Chúng ta sẽ khám phá từ các khái niệm cơ bản đến việc triển khai chi tiết, bao gồm cả cơ chế Claude Code tự động khám phá và áp dụng các kỹ năng này, cũng như cách chúng có thể được sử dụng như các lệnh tùy chỉnh. Đồng thời, chúng ta sẽ liên hệ những kiến thức này với tư duy Vibe Coding và cách ứng dụng chúng trong một môi trường Agentic AI mạnh mẽ như Antigravity IDE, nơi các agent có thể tự động chạy script, gọi subagent trình duyệt, đọc ghi file và lập kế hoạch một cách tự chủ.

## 1. Kỹ Năng Agent trong Claude Code: Nền Tảng của Trợ Lý Lập Trình Thông Minh

Kỹ năng agent là một tính năng cốt lõi trong Claude Code, được thiết kế để mở rộng khả năng của AI bằng cách cung cấp các mảnh ghép ngữ cảnh bổ sung, được tải động khi cần thiết. Đây là một tiêu chuẩn mở, được hỗ trợ bởi nhiều công cụ lập trình AI khác, cho phép một hệ sinh thái các công cụ AI chia sẻ và tận dụng cùng một định dạng kỹ năng.

> [!NOTE]
> Kỹ năng agent không phải là một tập hợp các lệnh cứng nhắc hay một cơ sở dữ liệu tĩnh. Thay vào đó, chúng là các gói thông tin, hành động hoặc hướng dẫn mà Claude Code có thể tự động khám phá và sử dụng một cách linh hoạt dựa trên nhiệm vụ hiện tại và ngữ cảnh của dự án.

### 1.1. Khái Niệm và Mục Đích Chính

Mục đích chính của kỹ năng agent trong Claude Code là giải quyết một thách thức cố hữu của các mô hình ngôn ngữ lớn (LLM): giới hạn cửa sổ ngữ cảnh (context window) và khả năng tập trung. Thay vì nhồi nhét tất cả thông tin dự án vào ngữ cảnh ngay từ đầu (lãng phí token và làm chậm phản hồi), kỹ năng agent cho phép Claude Code tải thông tin cần thiết một cách *động*.

**Các mục đích chính của kỹ năng agent bao gồm:**

*   **Cung cấp thông tin bổ sung và kiến thức chuyên sâu:** Đây có thể là tài liệu về các best practices cụ thể (ví dụ: cách xây dựng component React tối ưu, quy tắc đặt tên lớp Python, hoặc các nguyên tắc kiến trúc microservices của dự án). Nó giúp AI truy cập kiến thức mà nó có thể chưa được huấn luyện hoặc kiến thức rất đặc thù của dự án.
*   **Thiết lập hướng dẫn và quy tắc ràng buộc:** Đảm bảo Claude Code tuân thủ các tiêu chuẩn mã hóa (coding standards), mẫu thiết kế (design patterns), quy ước đặt tên hoặc các yêu cầu bảo mật cụ thể mà bạn mong muốn. Điều này đặc biệt quan trọng để duy trì tính nhất quán và chất lượng mã nguồn.
*   **Thực thi các script tự động:** Cho phép AI chạy các script (Python, JavaScript, shell script) để thực hiện các tác vụ như dọn dẹp file, kiểm tra mã, khởi tạo dự án, hoặc tự động hóa các quy trình nhỏ trong workflow phát triển.

### 1.2. Cấu Trúc Tổng Quan của Một Kỹ Năng

Mỗi kỹ năng được định nghĩa bởi một cấu trúc thư mục và các tệp cụ thể, cho phép linh hoạt trong việc tổ chức thông tin.

1.  **`Skill.md` (Bắt buộc):** Một tệp Markdown chứa metadata quan trọng (tên, mô tả) và nội dung chính của kỹ năng (hướng dẫn, kiến thức cốt lõi). Đây là "bộ mặt" của kỹ năng mà Claude Code sẽ đọc đầu tiên để quyết định có nên tải kỹ năng này hay không.
2.  **Tài liệu tham chiếu hoặc thư mục tài liệu bổ sung (Tùy chọn):** Các tệp Markdown khác chứa thông tin chi tiết hơn, ít được sử dụng thường xuyên hơn, được Claude Code tải động *chỉ khi AI xác định chúng cần thiết*. Điều này giúp giữ cho `Skill.md` chính luôn súc tích.
3.  **Script (Tùy chọn):** Các tệp script (ví dụ: `.py`, `.js`, `.sh`) mà AI có thể thực thi để tự động hóa các tác vụ. Đây là nơi kỹ năng agent bắt đầu chuyển từ cung cấp kiến thức sang thực hiện hành động.

Trong thực tế, các kỹ năng cung cấp kiến thức (knowledge skills) là hữu ích nhất cho việc xây dựng ứng dụng, giúp AI hiểu rõ hơn về cách bạn muốn xây dựng mọi thứ. Tuy nhiên, khả năng bao gồm script hoặc tài sản bổ sung (như hình ảnh với sơ đồ kiến trúc) cũng mang lại sự linh hoạt đáng kể, đặc biệt khi kết hợp với các hệ thống Agentic AI như Antigravity IDE.

### 1.3. Cơ Chế Khám Phá và Tải Ngữ Cảnh Động (Under the Hood)

Đây là "trái tim" của tính năng kỹ năng agent, giải quyết bài toán quản lý cửa sổ ngữ cảnh (context window) một cách hiệu quả.

1.  **Tải Metadata ban đầu:** Khi bạn khởi động Claude Code hoặc bắt đầu một phiên làm việc, tên (`name`) và mô tả (`description`) của TẤT CẢ các kỹ năng khả dụng (cả cục bộ và toàn cầu) sẽ được tải vào cửa sổ ngữ cảnh của Claude Code. Đây là một lượng token nhỏ, cho phép AI có một cái nhìn tổng quan nhanh chóng về "thư viện kiến thức" mà nó có thể truy cập.
2.  **Phân tích nhiệm vụ và Lời nhắc:** Khi bạn đưa ra một lời nhắc (prompt) hoặc Claude Code đang tự động làm việc trên một nhiệm vụ, nó sẽ phân tích cẩn thận nhiệm vụ đó, cùng với ngữ cảnh hiện tại của codebase.
3.  **Quyết định khám phá:** Claude Code sẽ so sánh nội dung của lời nhắc và ngữ cảnh dự án với tên và mô tả của các kỹ năng đã được tải metadata. Nếu AI xác định rằng một kỹ năng cụ thể có thể hữu ích cho nhiệm vụ hiện tại (ví dụ: lời nhắc yêu cầu tạo component React, và có một kỹ năng mô tả về "xây dựng component React hiện đại"), nó sẽ ra quyết định tải kỹ năng đó.
4.  **Tải nội dung chi tiết:** Chỉ khi AI ra quyết định, TOÀN BỘ nội dung của tệp `Skill.md` của kỹ năng được chọn (và các tài liệu tham chiếu động trong thư mục `references` nếu cần thiết) sẽ được tải vào cửa sổ ngữ cảnh.
5.  **Áp dụng kiến thức và Thực hiện:** Với ngữ cảnh được mở rộng, Claude Code sẽ sử dụng thông tin từ kỹ năng để đưa ra các đề xuất, viết mã, hoặc thực hiện các hành động phù hợp và chính xác hơn, tuân thủ các hướng dẫn đã được cung cấp.

> [!TIP]
> Cơ chế tải động này là chìa khóa để tối ưu hóa việc sử dụng cửa sổ ngữ cảnh. Thay vì tải tất cả mọi thứ ngay lập tức (có thể lãng phí token và làm chậm AI), Claude Code chỉ tải những gì thực sự cần thiết, giúp nó hoạt động hiệu quả, tập trung hơn và giảm thiểu chi phí token.

### 1.4. Vibe Coding và Tư Duy Agentic với Antigravity IDE

Để thực sự tận dụng tối đa kỹ năng agent, chúng ta cần áp dụng tư duy "Vibe Coding" và hiểu cách nó liên hệ với các hệ thống Agentic AI như Antigravity IDE.

**Vibe Coding** là một triết lý lập trình với AI, nơi bạn không chỉ đưa ra các yêu cầu cụ thể mà còn truyền tải "vibe" (ý định, phong cách, nguyên tắc ngầm) của dự án cho AI. Nó bao gồm việc:

*   **Định nghĩa rõ ràng phong cách:** Các quy tắc về đặt tên, cấu trúc thư mục, cách sử dụng framework.
*   **Thiết lập các ràng buộc:** Hạn chế sử dụng một số thư viện, yêu cầu tuân thủ các quy tắc bảo mật.
*   **Cung cấp ngữ cảnh đầy đủ:** Không chỉ code, mà cả kiến trúc, mục tiêu kinh doanh, và người dùng cuối.

**Antigravity IDE** là một hệ thống Agentic AI siêu việt mà bạn đang sử dụng, được thiết kế để thực hiện Vibe Coding ở cấp độ cao nhất. Antigravity không chỉ đơn thuần là một công cụ sinh code; nó là một môi trường phát triển tự chủ, có khả năng:

*   **Tự động lập kế hoạch:** Chia nhỏ các nhiệm vụ phức tạp thành các bước nhỏ hơn.
*   **Chạy script ngầm:** Thực thi các lệnh shell, script Python/JavaScript để tương tác với hệ thống file, chạy test, deploy.
*   **Gọi subagent trình duyệt:** Tương tác với các trang web, đọc tài liệu trực tuyến, sử dụng các công cụ web.
*   **Đọc/ghi file:** Tự do điều chỉnh codebase, tạo file mới, sửa đổi file hiện có.

**Liên hệ giữa Kỹ Năng Agent của Claude Code và Antigravity IDE:**

Kỹ năng agent trong Claude Code cung cấp các "khối xây dựng" kiến thức và hướng dẫn. Trong bối cảnh Antigravity IDE, chúng trở thành các nguyên tắc cốt lõi mà các agent của Antigravity sẽ tuân thủ trong quá trình lập kế hoạch và thực thi:

*   **Kỹ năng kiến thức (Knowledge Skills):** Cung cấp cho Antigravity các "tài liệu hướng dẫn nội bộ" để nó hiểu được "vibe" của dự án. Ví dụ, kỹ năng `modern-best-practice-react-components` sẽ hướng dẫn Antigravity cách tạo các component React không chỉ hoạt động mà còn tuân thủ các tiêu chuẩn cao nhất.
*   **Kỹ năng hành động (Actionable Skills/Scripts):** Một kỹ năng agent có chứa script trong Claude Code có thể được Antigravity trực tiếp sử dụng như một "khả năng" mới. Ví dụ, một kỹ năng `/cleanup-project` với script xóa file tạm có thể được Antigravity kích hoạt như một bước trong kế hoạch tối ưu hóa dự án.
*   **Tư duy Vibe Coding trong Antigravity:** Bằng cách định nghĩa các kỹ năng agent chất lượng cao, bạn đang truyền tải "vibe" của dự án một cách rõ ràng cho Antigravity. Điều này giúp Antigravity:
    *   **Lập kế hoạch thông minh hơn:** Khi Antigravity lập kế hoạch để thêm một tính năng, nó sẽ xem xét các kỹ năng agent để đảm bảo kế hoạch đó phù hợp với kiến trúc và best practices của dự án.
    *   **Thực thi chính xác hơn:** Khi Antigravity thực thi các bước, nó sẽ sử dụng kiến thức từ kỹ năng để viết code, sửa đổi file hoặc chạy lệnh một cách tuân thủ.
    *   **Giảm thiểu "hallucination":** Ngữ cảnh phong phú và có cấu trúc từ kỹ năng agent giúp giảm khả năng Antigravity tạo ra giải pháp không phù hợp hoặc không tồn tại.

Tóm lại, kỹ năng agent là cầu nối giữa ý định của bạn và khả năng thực thi của AI. Với Antigravity IDE, việc định nghĩa các kỹ năng chất lượng cao là chìa khóa để biến một trợ lý AI thông thường thành một đối tác phát triển tự chủ, hiểu rõ và tuân thủ "vibe" của dự án.

## 2. Định Nghĩa và Quản Lý Kỹ Năng Tùy Chỉnh

Việc tạo kỹ năng tùy chỉnh trong Claude Code được thực hiện thông qua cấu trúc thư mục và các tệp Markdown cụ thể. Chúng ta sẽ khám phá cả kỹ năng cục bộ (chỉ dùng cho một dự án) và kỹ năng toàn cầu (có thể chia sẻ giữa nhiều dự án).

### 2.1. Kỹ Năng Cục Bộ (Project-Specific Skills)

Kỹ năng cục bộ được định nghĩa trong thư mục dự án của bạn và chỉ khả dụng cho dự án đó. Đây là phương pháp được khuyến nghị cho hầu hết các trường hợp, vì nó giúp kỹ năng luôn phù hợp với công nghệ và yêu cầu cụ thể của từng dự án, tránh làm ô nhiễm ngữ cảnh với thông tin không liên quan.

**Cấu trúc thư mục:**

Để thêm một kỹ năng cục bộ, bạn cần tạo một thư mục con tên là `skills` bên trong thư mục cấu hình `.claude` của dự án. Sau đó, mỗi kỹ năng sẽ là một thư mục con riêng biệt bên trong thư mục `skills` này.

Ví dụ, đối với một dự án React:

```
.
├── .claude/
│   └── skills/
│       └── modern-best-practice-react-components/
│           └── Skill.md
│           └── references/
│               └── YouDon'tNeedUseEffect.md
│           └── scripts/
│               └── generateComponent.py  # Ví dụ script tạo component
├── src/
│   └── components/
│       └── Button/
│           └── Button.jsx
│           └── Button.module.css
│   └── App.jsx
└── package.json
```

**Tệp `Skill.md`:**

Mỗi thư mục kỹ năng (ví dụ: `modern-best-practice-react-components`) phải chứa một tệp bắt buộc tên là `Skill.md`. Tệp này chứa metadata và nội dung chính của kỹ năng. Đây là nơi bạn định nghĩa "vibe" của dự án cho AI.

**Metadata (Header YAML Front Matter):**

Tệp `Skill.md` phải bắt đầu bằng một khối metadata Markdown (YAML Front Matter), tối thiểu phải có `name` và `description`.

```markdown
---
name: modern-best-practice-react-components
description: Hướng dẫn xây dựng các component React hiện đại, sạch sẽ, áp dụng các best practices phổ biến, kiến trúc atomic design và tránh các lỗi thường gặp trong dự án này.
---

# Hướng dẫn xây dựng component React hiện đại theo Atomic Design

Khi làm việc với React trong dự án này, hãy luôn tuân thủ các nguyên tắc sau để đảm bảo mã nguồn dễ đọc, dễ bảo trì, hiệu suất cao và tuân thủ kiến trúc Atomic Design:

## 1. Triết lý Atomic Design

*   **Atoms:** Các thành phần UI cơ bản nhất không thể bị chia nhỏ hơn (ví dụ: Button, Input, Text). Chúng là nền tảng của mọi thứ.
*   **Molecules:** Kết hợp các Atoms lại với nhau để tạo thành các đơn vị chức năng đơn giản (ví dụ: SearchBar = Input + Button).
*   **Organisms:** Kết hợp Molecules và/hoặc Atoms để tạo thành các phần phức tạp hơn của giao diện (ví dụ: Header = Logo + Navigation + SearchBar).
*   **Templates:** Sắp xếp Organisms thành các bố cục trang, nhưng không chứa dữ liệu thực tế.
*   **Pages:** Là các instance cụ thể của Templates với dữ liệu thực tế, hiển thị giao diện cuối cùng cho người dùng.

## 2. Sử dụng Function Components và Hooks

Luôn ưu tiên Function Components kết hợp với React Hooks (useState, useEffect, useContext, useRef, useMemo, useCallback) thay vì Class Components. Điều này giúp mã nguồn gọn gàng, dễ kiểm thử và tái sử dụng logic.

## 3. Đặt tên và cấu trúc file theo Atomic Design

*   **Tên file component:** Sử dụng PascalCase (ví dụ: `Button.jsx`, `SearchBar.jsx`).
*   **Tên thư mục component:** Tổ chức mỗi component vào một thư mục riêng (ví dụ: `src/components/atoms/Button/`, `src/components/molecules/SearchBar/`).
*   **Index files:** Sử dụng `index.jsx` trong thư mục component để export component chính, giúp import gọn gàng hơn (ví dụ: `import Button from '@/components/atoms/Button';`).
*   **File styles:** Sử dụng CSS Modules (`.module.css`) cho styling cục bộ, hoặc Tailwind CSS nếu được cấu hình.
*   **File tests:** Đặt file test ngay bên cạnh component (`Button.test.jsx`).

## 4. Quản lý State

*   Sử dụng `useState` cho state cục bộ.
*   Sử dụng `useContext` hoặc các thư viện quản lý state (như Zustand, React Query) cho state toàn cục.
*   Tránh tạo state không cần thiết hoặc quá phức tạp. State nên được đặt ở cấp độ thấp nhất có thể.

## 5. Props

*   Sử dụng destructuring cho props để mã nguồn rõ ràng hơn.
*   Xác định `propTypes` (hoặc TypeScript interfaces) để kiểm tra kiểu dữ liệu của props.
*   Tránh truyền quá nhiều props (prop drilling) bằng cách sử dụng Context API hoặc composition.

## 6. Hiệu suất

*   Sử dụng `React.memo` cho các functional component để tránh re-render không cần thiết khi props không thay đổi.
*   Sử dụng `useCallback` và `useMemo` để memoize các hàm và giá trị, đặc biệt khi truyền chúng xuống các component con hoặc trong mảng phụ thuộc của `useEffect`.

... (Thêm các hướng dẫn chi tiết khác về accessbility, error boundaries, v.v.)
```

**Các trường metadata quan trọng:**

*   **`name` (Bắt buộc):** Tên của kỹ năng. Phải giống hệt với tên thư mục chứa `Skill.md`. Tên này giúp Claude Code nhận diện kỹ năng.
*   **`description` (Bắt buộc):** Mô tả ngắn gọn, súc tích và chính xác về mục đích của kỹ năng. Đây là phần cực kỳ quan trọng giúp Claude Code (và Antigravity IDE) quyết định khi nào nên khám phá và sử dụng kỹ năng này. Mô tả càng rõ ràng, Claude Code càng dễ dàng khám phá và áp dụng đúng lúc.

**Các trường metadata tùy chọn (Advanced Configuration):**

Bạn có thể thêm các trường sau để tinh chỉnh hành vi của kỹ năng, đặc biệt hữu ích khi làm việc với các hệ thống Agentic AI đa năng như Antigravity:

*   **`allowedTools`:** Một mảng các công cụ (tools) mà kỹ năng này được phép sử dụng. Ví dụ: `["shell", "browser"]`. Nếu không được chỉ định, kỹ năng có thể sử dụng tất cả các công cụ có sẵn. Điều này giúp kiểm soát quyền hạn của kỹ năng, đặc biệt quan trọng nếu kỹ năng có script.
*   **`model`:** Chỉ định mô hình AI cụ thể mà kỹ năng này nên sử dụng. Ví dụ: `claude-3-opus-20240229`. Điều này có thể hữu ích cho các tác vụ đòi hỏi mô hình mạnh hơn hoặc tiết kiệm chi phí với mô hình nhỏ hơn.
*   **`context`:**
    *   `default` (Mặc định): Kỹ năng sẽ sử dụng cửa sổ ngữ cảnh hiện có của tác vụ.
    *   `fork`: Kỹ năng sẽ mở một cửa sổ ngữ cảnh mới, độc lập. Điều này hữu ích cho các tác vụ phức tạp, nhạy cảm hoặc cần một môi trường riêng biệt để tránh xung đột ngữ cảnh. Ví dụ, một kỹ năng kiểm tra bảo mật có thể cần một ngữ cảnh "sạch" để tập trung hoàn toàn vào việc tìm kiếm lỗ hổng mà không bị ảnh hưởng bởi các hướng dẫn phát triển khác.

> [!NOTE]
> Để xem danh sách đầy đủ các trường metadata và tài liệu chi tiết, bạn nên tham khảo tài liệu chính thức của Claude Code.

### 2.2. Tài Liệu Tham Chiếu Động (Dynamic Reference Documents)

Để giữ cho tệp `Skill.md` chính luôn súc tích và tập trung vào các nguyên tắc cốt lõi, bạn có thể tách các thông tin chi tiết hơn, ít được sử dụng thường xuyên hoặc có tính chuyên sâu vào các tài liệu tham chiếu riêng biệt. Claude Code có khả năng tự động khám phá và tải các tài liệu này nếu AI xác định rằng chúng cần thiết cho nhiệm vụ hiện tại. Điều này là một phần quan trọng của cơ chế tải ngữ cảnh động.

**Ví dụ:**

Tiếp tục với ví dụ về kỹ năng React, giả sử bạn có một bộ quy tắc phức tạp liên quan đến hook `useEffect` của React mà không phải lúc nào cũng cần tải. Bạn có thể đặt chúng vào một tệp riêng:

1.  **Tạo thư mục `references`:** Bên trong thư mục kỹ năng của bạn (ví dụ: `modern-best-practice-react-components`), tạo một thư mục con tên là `references`.
2.  **Tạo tệp Markdown tham chiếu:** Đặt tệp `YouDon'tNeedUseEffect.md` vào thư mục `references`.

```
.claude/
└── skills/
    └── modern-best-practice-react-components/
        └── Skill.md
        └── references/
            └── YouDon'tNeedUseEffect.md
```

**Nội dung `YouDon'tNeedUseEffect.md`:**

```markdown
# Quy tắc sử dụng useEffect trong React một cách hiệu quả

`useEffect` là một hook mạnh mẽ nhưng cũng dễ gây ra lỗi và vấn đề hiệu suất nếu không được sử dụng đúng cách. Dưới đây là các quy tắc và mẹo để tránh các vấn đề phổ biến và tối ưu hóa việc sử dụng `useEffect` trong dự án này:

## 1. Hiểu rõ Dependency Array (Mảng Phụ Thuộc)

*   **Luôn cung cấp một mảng phụ thuộc** cho `useEffect`. Nếu bạn không cung cấp, effect sẽ chạy sau mỗi lần render, thường dẫn đến các vòng lặp vô hạn hoặc hiệu suất kém.
*   **Mảng rỗng (`[]`):** Effect chỉ chạy một lần sau lần render đầu tiên (tương tự `componentDidMount` trong Class Components). Lý tưởng cho việc fetch dữ liệu một lần hoặc thiết lập các sự kiện chỉ cần khởi tạo.
*   **Bao gồm tất cả các giá trị (biến, hàm) mà effect sử dụng từ phạm vi bên ngoài trong mảng phụ thuộc.** Điều này đảm bảo effect chạy lại khi bất kỳ giá trị phụ thuộc nào thay đổi, ngăn ngừa lỗi "stale closures".
*   **Sử dụng `useCallback` và `useMemo`** cho các hàm và giá trị phức tạp được truyền vào mảng phụ thuộc để tránh re-render không cần thiết của effect.

## 2. Tránh Side Effects Không Cần Thiết

*   **Chỉ sử dụng `useEffect` cho các side effects:** Ví dụ: fetch dữ liệu, subscribe sự kiện, thao tác DOM trực tiếp, đồng bộ hóa với hệ thống bên ngoài.
*   **Không dùng `useEffect` để cập nhật state đồng bộ dựa trên state khác:** Thay vào đó, hãy sử dụng hàm cập nhật state của `useState` (ví dụ: `setCount(prevCount => prevCount + 1)`) hoặc `useReducer`. Cập nhật state trong `useEffect` có thể gây ra re-render không cần thiết và các vòng lặp.

## 3. Dọn dẹp Side Effects (Cleanup Function)

*   **Nếu effect của bạn tạo ra tài nguyên** (ví dụ: đăng ký sự kiện, hẹn giờ, kết nối WebSocket), hãy trả về một hàm dọn dẹp (cleanup function) từ `useEffect` để giải phóng tài nguyên khi component unmount hoặc khi dependencies thay đổi. Điều này ngăn chặn rò rỉ bộ nhớ và các hành vi không mong muốn.

```javascript
import React, { useEffect, useState } from 'react';

function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // Side effect: thiết lập một bộ đếm thời gian
    const intervalId = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000);

    // Hàm dọn dẹp: xóa bộ đếm thời gian khi component unmount
    return () => {
      console.log('Cleaning up interval...');
      clearInterval(intervalId);
    };
  }, []); // Mảng phụ thuộc rỗng, effect chỉ chạy một lần và dọn dẹp khi unmount

  return <div>Count: {count}</div>;
}
```

## 4. Các trường hợp nên tránh `useEffect`

*   **Tính toán giá trị phụ thuộc vào props/state khác:** Thay vì `useEffect` để cập nhật state `fullName` khi `firstName` hoặc `lastName` thay đổi, hãy tính toán `fullName` trực tiếp trong render hoặc sử dụng `useMemo`.
*   **Chỉ để log:** Sử dụng `console.log` trực tiếp hoặc các thư viện logging chuyên dụng thay vì `useEffect` chỉ để ghi log.

... (Thêm các quy tắc và ví dụ khác)
```

Khi Claude Code làm việc với các component React liên quan đến `useEffect`, nó có thể tự động nhận ra rằng cần thêm thông tin từ `YouDon'tNeedUseEffect.md` và tải nội dung đó vào ngữ cảnh. Đối với Antigravity IDE, điều này có nghĩa là khi Antigravity lập kế hoạch hoặc thực thi việc tạo/sửa đổi component có sử dụng `useEffect`, nó sẽ có đầy đủ các hướng dẫn chi tiết này để đảm bảo mã nguồn được viết đúng cách và hiệu quả.

### 2.3. Kỹ Năng Toàn Cầu (Global Skills)

Ngoài kỹ năng cục bộ, bạn cũng có thể tạo kỹ năng toàn cầu. Các kỹ năng này được lưu trữ trong thư mục cấu hình gốc của Claude Code trên hệ thống của bạn (bên ngoài bất kỳ dự án cụ thể nào) và có thể được chia sẻ và sử dụng trong tất cả các dự án bạn quản lý bằng Claude Code.

**Vị trí:**

Thư mục cấu hình gốc của Claude Code thường nằm ở vị trí như `~/.claude` (trên Linux/macOS) hoặc `%USERPROFILE%\.claude` (trên Windows). Bạn tạo một thư mục `skills` trong đó và thêm các thư mục kỹ năng con tương tự như kỹ năng cục bộ.

**Khi nào nên dùng kỹ năng toàn cầu?**

Mặc dù có vẻ tiện lợi khi chia sẻ kỹ năng, nhưng bạn nên cân nhắc kỹ. Các dự án khác nhau có thể sử dụng các công nghệ, framework và best practices rất khác nhau. Việc tải tên và mô tả của một kỹ năng không liên quan (ví dụ: kỹ năng Next.js cho một dự án chỉ dùng Vue.js) vẫn chiếm một phần nhỏ trong cửa sổ ngữ cảnh và có thể gây nhiễu cho AI.

> [!WARNING]
> Thông thường, kỹ năng cục bộ (project-specific skills) được ưu tiên hơn vì chúng đảm bảo tính phù hợp và hiệu quả tối đa cho từng dự án cụ thể, tránh việc tải các ngữ cảnh không cần thiết. Kỹ năng toàn cầu chỉ nên được sử dụng cho các quy tắc rất chung, các công cụ mà bạn chắc chắn sẽ dùng trong hầu hết các dự án (ví dụ: "Quy tắc viết Git commit message", "Hướng dẫn sử dụng Docker cơ bản") hoặc các kỹ năng hành động (script) có thể áp dụng rộng rãi.
>
> Trong bối cảnh Vibe Coding với Antigravity IDE, việc sử dụng kỹ năng toàn cầu cần phải rất chọn lọc. Bạn không muốn Antigravity bị "nhiễu loạn" bởi quá nhiều "vibe" không liên quan đến dự án hiện tại.

## 3. Kỹ Năng Agent và Tương Tác Lệnh

Một khía cạnh thú vị khác của kỹ năng agent là chúng tự động xuất hiện dưới dạng các lệnh slash (`/`) trong giao diện dòng lệnh của Claude Code. Khi bạn gõ `/` vào dấu nhắc lệnh của Claude Code, bạn sẽ thấy một danh sách các lệnh tích hợp sẵn và cả các kỹ năng tùy chỉnh của bạn.

### 3.1. Khám Phá Kỹ Năng và Ra Quyết Định của AI trong Thực Tế

Để minh họa cơ chế khám phá và áp dụng kỹ năng, hãy xem xét một ví dụ thực tế khi bạn làm việc với Claude Code (hoặc Antigravity IDE):

Giả sử bạn đã định nghĩa kỹ năng `modern-best-practice-react-components` như trên, và bạn đưa ra một lời nhắc như:

```
Thêm một trang xác thực người dùng (authentication route page content) vào ứng dụng này.
Chỉ sử dụng xác thực email và mật khẩu.
Người dùng nên có thể chuyển đổi giữa chế độ đăng nhập/đăng ký bằng cách sử dụng search params.
Không cần tính năng đặt lại mật khẩu trong ứng dụng demo này.
```

Khi xử lý lời nhắc này, Claude Code (và các agent của Antigravity) sẽ thực hiện một chuỗi các bước ra quyết định:

1.  **Phân tích ngữ cảnh dự án:** Claude Code đọc `Claude.md` (hoặc các file cấu hình dự án khác) để hiểu cấu trúc tổng thể, các công nghệ đang được sử dụng (ví dụ: React, Vite, Tailwind CSS).
2.  **Phân tích lời nhắc:** AI phân tích lời nhắc để nhận diện các từ khóa và ý định chính: "trang xác thực", "React", "email/mật khẩu", "đăng nhập/đăng ký", "search params".
3.  **So sánh với metadata kỹ năng:** Claude Code kiểm tra tên và mô tả của TẤT CẢ các kỹ năng khả dụng. Nó nhận ra rằng kỹ năng `modern-best-practice-react-components` có mô tả liên quan trực tiếp đến "xây dựng các component React hiện đại", rất phù hợp với yêu cầu tạo trang xác thực.
4.  **Tải nội dung kỹ năng:** Dựa trên sự phù hợp, Claude Code quyết định tải toàn bộ nội dung của `Skill.md` của `modern-best-practice-react-components` vào ngữ cảnh. Nếu trong quá trình lập kế hoạch, AI nhận ra rằng việc triển khai xác thực có thể liên quan đến các side effects (ví dụ: fetch API, quản lý token), nó cũng sẽ tự động tải `YouDon'tNeedUseEffect.md` từ thư mục `references`.
5.  **Lập kế hoạch và Thực thi (Antigravity IDE):** Nếu bạn đang sử dụng Antigravity IDE, các agent của nó sẽ bắt đầu lập kế hoạch:
    *   **Bước 1: Tạo cấu trúc file:** Agent sẽ tạo các thư mục và file cần thiết cho trang xác thực, tuân thủ cấu trúc Atomic Design và quy ước đặt tên từ kỹ năng.
    *   **Bước 2: Viết code cho component:** Agent sẽ viết mã React cho các component (ví dụ: `LoginForm.jsx`, `RegisterForm.jsx`, `AuthPage.jsx`), sử dụng Function Components và Hooks, destructuring props, và các best practices khác được định nghĩa trong kỹ năng.
    *   **Bước 3: Xử lý logic `useEffect`:** Nếu có logic liên quan đến `useEffect` (ví dụ: fetch dữ liệu người dùng sau khi đăng nhập), agent sẽ tham khảo `YouDon'tNeedUseEffect.md` để đảm bảo `useEffect` được sử dụng đúng cách, có cleanup function và dependency array chính xác.
    *   **Bước 4: Tích hợp CSS Modules/Tailwind:** Agent sẽ áp dụng styling theo quy định của dự án, sử dụng CSS Modules hoặc Tailwind CSS như đã được "huấn luyện" qua kỹ năng.
6.  **Đưa ra giải pháp/Code:** Cuối cùng, Claude Code (hoặc Antigravity IDE) sẽ đưa ra mã nguồn hoàn chỉnh hoặc thực hiện các thay đổi cần thiết trong codebase, đảm bảo rằng giải pháp không chỉ hoạt động mà còn tuân thủ "vibe" và các tiêu chuẩn chất lượng của dự án đã được định nghĩa trong kỹ năng agent.

> [!TIP]
> Bạn có thể "gợi ý" cho Claude Code sử dụng các kỹ năng cụ thể bằng cách thêm các từ khóa liên quan vào lời nhắc của mình. Ví dụ, sau khi đưa ra lời nhắc, bạn có thể thêm một tin nhắn như "Sử dụng mã JSX hiện đại, tuân thủ Atomic Design và Tailwind CSS" để tăng khả năng Claude Code kích hoạt các kỹ năng liên quan mà bạn đã định nghĩa.

### 3.2. Kỹ Năng Agent như Lệnh Tùy Chỉnh (Agent Skills as Commands)

Các kỹ năng agent tự động xuất hiện dưới dạng các lệnh slash (`/`) trong giao diện của Claude Code. Khi bạn gõ `/`, bạn sẽ thấy một danh sách các lệnh, bao gồm các kỹ năng bạn đã định nghĩa.

**Mục đích chính và phụ:**

*   **Mục đích chính:** Như đã đề cập, mục đích chính của kỹ năng là để Claude Code tự động khám phá và sử dụng chúng một cách chủ động, không cần sự can thiệp thủ công từ bạn.
*   **Mục đích phụ (tùy chọn):** Bạn CÓ THỂ gọi một kỹ năng như một lệnh slash.

**Khi nào việc gọi kỹ năng như lệnh có ý nghĩa?**

Nếu một kỹ năng chỉ chứa kiến thức hoặc hướng dẫn chung (như ví dụ về `modern-best-practice-react-components` ở trên), việc gọi nó như một lệnh (`/modern-best-practice-react-components`) sẽ không tạo ra bất kỳ hành động cụ thể nào. Claude Code có thể sẽ trả lời rằng nó không biết phải làm gì, vì kỹ năng đó chỉ cung cấp thông tin chứ không phải chỉ thị hành động.

Tuy nhiên, nếu bạn xây dựng một kỹ năng có chứa:

*   **Các hướng dẫn hành động cụ thể:** Ví dụ, một kỹ năng có tên `/cleanup-project` với nội dung `Skill.md` hướng dẫn AI cách xóa các file tạm hoặc thư mục build.
*   **Script đính kèm:** Một kỹ năng có script Python hoặc JavaScript để tự động hóa một tác vụ nào đó (ví dụ: `/run-tests-with-coverage`).

Trong những trường hợp này, việc gọi kỹ năng như một lệnh có thể hữu ích để kích hoạt hành động đó một cách trực tiếp.

**Kỹ năng Agent trong Antigravity IDE như các Lệnh Mạnh Mẽ hơn:**

Trong Antigravity IDE, khái niệm "kỹ năng agent như lệnh" được nâng cấp đáng kể. Khi bạn gọi một kỹ năng có chứa script hoặc hướng dẫn hành động, Antigravity có thể:

*   **Thực thi script trực tiếp:** Nếu kỹ năng `/run-tests-with-coverage` có một script Python, Antigravity sẽ chạy script đó trong môi trường dự án và báo cáo kết quả.
*   **Kích hoạt chuỗi hành động:** Một lệnh `/cleanup-project` có thể không chỉ xóa file mà còn liên quan đến việc gọi các subagent để phân tích thư mục log, nén file cũ, hoặc thậm chí tương tác với hệ thống quản lý phiên bản.
*   **Lập kế hoạch động cho lệnh:** Ngay cả khi một kỹ năng chỉ có hướng dẫn, Antigravity có thể lập kế hoạch các bước cụ thể để thực hiện hướng dẫn đó. Ví dụ, nếu bạn gọi `/refactor-legacy-code`, Antigravity sẽ không chỉ đọc hướng dẫn mà còn phân tích codebase hiện có, xác định các khu vực cần refactor, và tạo một kế hoạch từng bước để thực hiện.

> [!NOTE]
> Điều quan trọng là phải phân biệt giữa "kỹ năng agent được gọi như lệnh" và "lệnh tùy chỉnh" (custom commands) thực sự. Lệnh tùy chỉnh được thiết kế chủ yếu để bạn chủ động thực thi, trong khi kỹ năng agent chủ yếu dành cho AI tự động sử dụng. Tuy nhiên, khả năng gọi kỹ năng như lệnh mang lại một lớp linh hoạt bổ sung cho các trường hợp đặc biệt, đặc biệt khi kết hợp với khả năng tự chủ của Antigravity IDE.

## Kết Luận

Kỹ năng agent trong Claude Code là một công cụ mạnh mẽ, cho phép bạn tùy chỉnh và mở rộng khả năng của trợ lý lập trình AI. Bằng cách định nghĩa rõ ràng các kiến thức, quy tắc và hướng dẫn, bạn không chỉ giúp Claude Code hoạt động hiệu quả hơn mà còn truyền tải "vibe" độc đáo của dự án mình. Cơ chế tải ngữ cảnh động đảm bảo AI luôn tập trung và tối ưu hóa việc sử dụng tài nguyên.

Khi kết hợp với một hệ thống Agentic AI như Antigravity IDE, kỹ năng agent trở thành nền tảng cho một quy trình phát triển tự chủ và thông minh hơn. Chúng cho phép Antigravity không chỉ hiểu mà còn tự động lập kế hoạch và thực thi các nhiệm vụ phức tạp, tuân thủ nghiêm ngặt các tiêu chuẩn và phong cách bạn đã thiết lập. Việc nắm vững cách tạo và quản lý kỹ năng agent là chìa khóa để khai thác tối đa tiềm năng của AI trong môi trường phát triển hiện đại, biến Claude Code và Antigravity IDE thành những đối tác không thể thiếu trong hành trình xây dựng phần mềm của bạn.

<!-- REVIEWED_BY_AGENT -->
