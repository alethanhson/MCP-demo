# Hướng dẫn về MCP mà tôi ước mình đã biết

Các tác nhân AI cuối cùng đã vượt ra ngoài khả năng chỉ đơn thuần trò chuyện. Chúng đang giải quyết các vấn đề đa bước, điều phối quy trình làm việc và hoạt động tự chủ. Đằng sau nhiều đột phá này là MCP.

MCP đang lan truyền mạnh mẽ. Nhưng nếu bạn cảm thấy choáng ngợp bởi các thuật ngữ, bạn không đơn độc.

Hôm nay, chúng ta sẽ khám phá lý do tại sao các công cụ AI hiện tại còn thiếu sót và cách MCP giải quyết vấn đề đó. Chúng ta sẽ đề cập đến các thành phần cốt lõi, tầm quan trọng của chúng, kiến trúc 3 lớp và những hạn chế của nó. Bạn cũng sẽ tìm thấy các ví dụ thực tế cùng các trường hợp sử dụng.

Đây là hướng dẫn mà tôi ước mình đã có khi mới bắt đầu.

---

### Nội dung bao gồm?

Tóm lại, chúng ta sẽ đi sâu vào các chủ đề sau:

- Vấn đề của các công cụ AI hiện tại.
- Giới thiệu về MCP và các thành phần cốt lõi của nó.
- MCP hoạt động như thế nào?
- Vấn đề mà MCP giải quyết và tại sao nó lại quan trọng.
- 3 Lớp của MCP (và cách tôi đã hiểu chúng).
- Cách dễ nhất để kết nối hơn 100 máy chủ MCP được quản lý với xác thực tích hợp.
- Sáu ví dụ thực tế với các bản demo.
- Một số hạn chế của MCP.

Chúng ta sẽ đề cập đến rất nhiều nội dung, vì vậy hãy bắt đầu.

---

## 1. Vấn đề của các công cụ AI hiện tại.

Nếu bạn đã từng thử xây dựng một tác nhân AI thực sự làm được những việc như kiểm tra email hoặc gửi tin nhắn Slack (dựa trên quy trình làm việc của bạn), bạn sẽ biết khó khăn như thế nào: quá trình này thường lộn xộn và kết quả không phải lúc nào cũng như mong đợi.

Chúng ta có các API tuyệt vời và các công cụ cần thiết. Nhưng khả năng sử dụng thực tế và độ tin cậy không phải lúc nào cũng cao.

Ngay cả các công cụ như Cursor (được quảng cáo rầm rộ trên Twitter) gần đây cũng nhận được những lời phàn nàn về hiệu suất không như mong đợi.

### 1. Quá nhiều API, thiếu ngữ cảnh

Mỗi công cụ bạn muốn AI sử dụng về cơ bản là một tích hợp API nhỏ. Hãy tưởng tượng một người dùng nói: "Anmol có gửi email cho tôi về báo cáo cuộc họp hôm qua không?"

Để một LLM trả lời, nó phải:

- Nhận ra đây là một tác vụ tìm kiếm email, không phải truy vấn Slack hay Notion.
- Chọn điểm cuối chính xác, ví dụ `search_email_messages`.
- Phân tích cú pháp và tóm tắt kết quả bằng ngôn ngữ tự nhiên.

Tất cả điều này phải diễn ra trong giới hạn của cửa sổ ngữ cảnh. Điều này đặt ra nhiều thách thức. Các mô hình thường quên, đoán hoặc "ảo giác" trong quá trình này.

Và nếu bạn không thể xác minh tính chính xác, bạn thậm chí còn không nhận ra vấn đề đang xảy ra.

### 2. API dựa trên các bước, nhưng LLM không giỏi ghi nhớ các bước.

Hãy lấy một ví dụ cơ bản về CRM:

- Đầu tiên, bạn lấy ID liên hệ → `get_contact_id`.
- Sau đó, tìm nạp dữ liệu hiện tại của họ → `read_contact`.
- Cuối cùng, vá bản cập nhật → `patch_contact`.

Trong mã truyền thống, bạn có thể trừu tượng hóa điều này thành một hàm và hoàn thành tác vụ. Nhưng với LLM? Mỗi bước là một cơ hội để mắc lỗi do sai tham số, thiếu trường hoặc chuỗi bị hỏng. Và đột nhiên "trợ lý AI" của bạn chỉ biết xin lỗi bằng ngôn ngữ tự nhiên thay vì thực hiện cập nhật.

### 3. Các cấu trúc dựa trên Prompt Engineering mong manh

API phát triển. Tài liệu thay đổi. Luồng xác thực được cập nhật. Bạn có thể thức dậy vào một buổi sáng và thấy rằng tác nhân hoạt động hoàn hảo của bạn giờ đây bị lỗi do các thay đổi từ bên thứ ba.

Và không giống như các ứng dụng truyền thống, không có khung hoặc lớp trừu tượng chung nào để dựa vào. Mỗi tích hợp công cụ AI là một cấu trúc dựa trên prompt engineering mong manh, tạo ra JSON. Điều này có nguy cơ làm hỏng khả năng thực hiện các tác vụ lặp đi lặp lại của tác nhân AI của bạn.

### 4. Khóa nhà cung cấp (Vendor Lock-in).

Đã xây dựng các công cụ của bạn cho GPT-4? Tuyệt vời. Nhưng bạn sẽ cần viết lại tất cả các mô tả hàm và lời nhắc hệ thống từ đầu nếu bạn chuyển sang các mô hình khác như Claude hoặc Gemini.

Đó không phải là vấn đề nhỏ và hiện tại không có giải pháp phổ quát nào cho việc này.

Phải có một cách để các công cụ và mô hình giao tiếp rõ ràng, mà không cần nhồi nhét logic phức tạp vào các lời nhắc dài dòng. Đó là lúc MCP ra đời.

---

## 2. Giới thiệu về MCP và các thành phần cốt lõi của nó.

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) là một giao thức mở mới nhằm chuẩn hóa cách các ứng dụng cung cấp ngữ cảnh và công cụ cho LLM.

Hãy coi nó như một trình kết nối phổ quát cho AI. MCP hoạt động như một hệ thống plugin cho Cursor, cho phép bạn mở rộng khả năng của Tác nhân bằng cách kết nối nó với các nguồn dữ liệu và công cụ khác nhau.

![mcp server](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmdh8ztjauqv05yzrk0gj.png)
_Nguồn ảnh: Greg Isenburg trên YouTube_

MCP giúp bạn xây dựng các tác nhân và quy trình làm việc phức tạp dựa trên LLM.

Ví dụ, một máy chủ MCP cho Obsidian giúp trợ lý AI tìm kiếm và đọc ghi chú từ kho Obsidian của bạn.

Tác nhân AI của bạn giờ đây có thể thực hiện các hành động như:

→ Gửi email qua Gmail
→ Tạo tác vụ trong Linear
→ Tìm kiếm tài liệu trong Notion
→ Đăng tin nhắn trong Slack
→ Cập nhật bản ghi trong Salesforce

Tất cả bằng cách gửi hướng dẫn bằng ngôn ngữ tự nhiên thông qua một giao diện chuẩn hóa.

Hãy nghĩ xem điều này có ý nghĩa gì đối với năng suất. Các tác vụ từng yêu cầu chuyển đổi giữa hơn 5 ứng dụng giờ đây có thể được thực hiện trong một cuộc trò chuyện duy nhất với tác nhân của bạn.

Về cốt lõi, MCP tuân theo kiến trúc máy khách-máy chủ, trong đó một ứng dụng máy khách có thể kết nối với nhiều máy chủ.

![mcp server](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4qblsimyt39tbg619b84.png)
_Nguồn ảnh: ByteByteGo_

### Các thành phần cốt lõi

Dưới đây là các thành phần cốt lõi trong hệ sinh thái MCP:

- `Máy khách MCP` - các ứng dụng như Claude Desktop, Cursor, Windsurf hoặc các công cụ AI muốn truy cập dữ liệu qua MCP.
- `Máy chủ MCP` - các chương trình nhẹ, mỗi chương trình hiển thị các khả năng cụ thể (như đọc tệp, truy vấn cơ sở dữ liệu...) thông qua Giao thức Ngữ cảnh Mô hình được chuẩn hóa.
- `Máy khách giao thức` - các thành phần duy trì kết nối 1:1 với các máy chủ MCP, hoạt động như cầu nối giao tiếp.
- `Nguồn dữ liệu cục bộ` - các tệp, cơ sở dữ liệu và dịch vụ trên máy tính của bạn mà các máy chủ MCP có thể truy cập an toàn. Ví dụ, một máy chủ MCP tự động hóa trình duyệt cần truy cập vào trình duyệt của bạn để hoạt động.
- `Dịch vụ từ xa` - các API bên ngoài và hệ thống dựa trên đám mây mà các máy chủ MCP có thể kết nối.

Nếu bạn quan tâm đến việc đọc kiến trúc chi tiết, hãy xem [tài liệu chính thức](https://modelcontextprotocol.io/docs/concepts/architecture). Nó bao gồm các lớp giao thức, vòng đời kết nối và xử lý lỗi trong việc triển khai tổng thể.

Chúng ta sẽ đề cập đến mọi thứ, nhưng nếu bạn quan tâm đến việc đọc thêm về MCP, hãy xem hai bài viết blog này:

- [Model Context Protocol (MCP) là gì?](https://www.builder.io/blog/model-context-protocol) của nhóm Builder.io
- [MCP: Nó là gì và tại sao nó quan trọng](https://addyo.substack.com/p/mcp-what-it-is-and-why-it-matters) của Addy Osmani

---

## 3. MCP hoạt động như thế nào?

Hệ sinh thái MCP bao gồm [một số thành phần chính](https://www.builder.io/blog/model-context-protocol) hoạt động cùng nhau. Hãy cùng tìm hiểu sơ lược về chúng.

![mcp architecture](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn3wsl4nb7x1ratyyusws.png)
_Nguồn ảnh: Huggingface_

⚡ Máy khách (Client).

Máy khách là các ứng dụng bạn thực sự sử dụng như Cursor, Claude Desktop, và các ứng dụng khác. Nhiệm vụ của chúng là:

- Yêu cầu các khả năng có sẵn từ máy chủ MCP.
- Trình bày các khả năng đó (công cụ, tài nguyên, lời nhắc) cho mô hình AI.
- Chuyển tiếp các yêu cầu sử dụng công cụ của AI trở lại máy chủ và trả về kết quả.
- Cung cấp cho các mô hình tổng quan về giao thức MCP cơ bản để tương tác nhất quán.

Chúng xử lý giao tiếp giữa các thành phần trong hệ thống: người dùng, mô hình AI và máy chủ MCP.

⚡ Máy chủ.

Máy chủ MCP đóng vai trò trung gian giữa người dùng/AI và các dịch vụ bên ngoài. Chúng:

- Cung cấp giao diện [JSON-RPC](https://www.jsonrpc.org/) chuẩn hóa để truy cập công cụ và tài nguyên.
- Chuyển đổi các API hiện có thành các khả năng tương thích với MCP mà không yêu cầu thay đổi API.
- Xử lý xác thực, định nghĩa khả năng và tiêu chuẩn giao tiếp.

Chúng cung cấp ngữ cảnh, công cụ và lời nhắc cho máy khách.

⚡ Nhà cung cấp dịch vụ.

Đây là các hệ thống hoặc nền tảng bên ngoài (như Discord, Notion, Figma) thực hiện các tác vụ thực tế. Chúng không cần thay đổi API của mình để tương thích với MCP.

Toàn bộ thiết lập này cho phép các nhà phát triển kết nối bất kỳ API hiện có nào vào bất kỳ máy khách hỗ trợ MCP nào, tránh phụ thuộc vào các tích hợp tập trung của các nhà cung cấp AI lớn.

### Các khối xây dựng của MCP: Công cụ, Tài nguyên và Lời nhắc (Prompts)

⚡ [Công cụ](https://modelcontextprotocol.io/docs/concepts/tools).

Công cụ đại diện cho các hành động mà AI có thể thực hiện như `search_emails` hoặc `create_issue_linear`. Chúng tạo thành nền tảng cho cách các mô hình tương tác với thế giới thực thông qua MCP.

⚡ [Tài nguyên](https://modelcontextprotocol.io/docs/concepts/resources).

Tài nguyên đại diện cho bất kỳ loại dữ liệu nào mà máy chủ MCP muốn cung cấp cho máy khách. Điều này có thể bao gồm:

- Nội dung tệp
- Bản ghi cơ sở dữ liệu
- Phản hồi API
- Dữ liệu hệ thống trực tiếp
- Ảnh chụp màn hình và hình ảnh
- Tệp nhật ký
- Và nhiều hơn nữa

Mỗi tài nguyên được xác định bằng một URI duy nhất (như `file://user/prefs.json`), có thể là ghi chú dự án, tùy chọn mã hóa hoặc bất cứ điều gì cụ thể đối với bạn. Nó có thể chứa dữ liệu văn bản hoặc nhị phân.

Tài nguyên được xác định bằng cách sử dụng URI tuân theo định dạng sau:

```
[giao thức]://[máy chủ]/[đường dẫn]
```

Ví dụ:

- `file:///home/user/documents/report.pdf`
- `postgres://database/customers/schema`
- `screen://localhost/display1`

Máy chủ cũng có thể định nghĩa các lược đồ URI tùy chỉnh của riêng họ. Bạn có thể đọc thêm trên [tài liệu chính thức](https://modelcontextprotocol.io/docs/concepts/resources).

⚡ [Lời nhắc](https://modelcontextprotocol.io/docs/concepts/prompts).

Công cụ cho phép AI thực hiện hành động, nhưng lời nhắc hướng dẫn AI cách hành xử khi thực hiện hành động đó.

Nó giống như hướng dẫn cho mô hình trong quá trình sử dụng công cụ. Chúng hoạt động như các hướng dẫn giúp AI tuân theo các phong cách, quy trình làm việc hoặc giao thức an toàn cụ thể, ví dụ như tuân theo danh sách kiểm tra an toàn cụ thể trước khi nhấn nút `delete_everything`.

🎯 Hãy cùng khám phá một kịch bản thực tế:

Hãy tưởng tượng một máy chủ MCP của Google Calendar. API Lịch rất mạnh mẽ nhưng có thể trả về nhiều thông tin không cần thiết, mỗi sự kiện bao gồm các trường cho khách, múi giờ, lời nhắc, tệp đính kèm và nhiều hơn nữa. Nếu bạn yêu cầu mô hình AI `lên lịch lại tất cả các cuộc họp của tôi với Alice vào tuần tới`, nó có thể gặp khó khăn trong việc lọc dữ liệu liên quan khỏi nhiễu.

Đây là lúc `lời nhắc` và `tài nguyên` phát huy tác dụng.

Một lời nhắc MCP có thể hướng dẫn mô hình: "Khi làm việc với các sự kiện lịch, chỉ sửa đổi những sự kiện có tiêu đề hoặc người tham gia khớp. Trích xuất các sự kiện liên quan bằng công cụ `list-events`, sao chép chúng vào một tài nguyên tạm thời (`Tài nguyên B`), áp dụng các thay đổi ở đó và sử dụng `update-events-from-resource` để đồng bộ hóa chúng trở lại."
Cách tiếp cận này cho phép AI tập trung vào dữ liệu sạch, có thể chỉnh sửa ở trạng thái được kiểm soát (`tài nguyên`), được hướng dẫn bởi các hướng dẫn có thể tái sử dụng, được chuẩn hóa (`lời nhắc`) với các hành động phù hợp (`công cụ`).

![builder.io notion example](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6pk6nh004srmvyr59shf.png)
_Nguồn ảnh: Ví dụ Notion của Builder.io_

Bạn cũng nên đọc ví dụ Notion trên [Builder.io](https://www.builder.io/blog/model-context-protocol), họ đã đề cập đến nó trong phần lời nhắc MCP.

Khi máy khách kết nối với máy chủ MCP, nó yêu cầu danh sách các công cụ, tài nguyên và lời nhắc có sẵn. Dựa trên đầu vào của người dùng, máy khách chọn những gì sẽ hiển thị cho mô hình. Khi mô hình chọn một hành động, máy khách sẽ thực thi nó thông qua máy chủ và đảm bảo ủy quyền phù hợp với luồng dữ liệu.

---

## 4. Vấn đề mà MCP giải quyết và tại sao nó lại quan trọng.

Hãy cùng thảo luận ngắn gọn về các vấn đề mà MCP giải quyết:

⚡ Một giao thức chung = hàng nghìn công cụ.

Giao thức chung này có nghĩa là một AI có thể tích hợp với hàng nghìn công cụ miễn là các công cụ đó có giao diện MCP, loại bỏ nhu cầu tích hợp tùy chỉnh cho từng ứng dụng mới.

Các dịch vụ mô tả những gì chúng có thể làm ("gửi tin nhắn Discord", "tạo vé Linear") và cách thực hiện (tham số, phương thức xác thực) bằng cách sử dụng định dạng [JSON-RPC](https://www.jsonrpc.org/) nhất quán.

⚡ Phân tách rõ ràng các vai trò: mô hình suy nghĩ, công cụ hành động.

Nó tạo ra sự phân tách rõ ràng giữa mô hình AI (người suy nghĩ) và các công cụ bên ngoài (người thực hiện). Tác nhân AI của bạn sẽ không bị hỏng mỗi khi Slack điều chỉnh API của nó.

⚡ Với MCP, bạn không phải làm lại tất cả các mô tả công cụ của mình khi chuyển đổi giữa các mô hình như GPT, Claude hoặc Gemini. Các công cụ và logic của bạn vẫn giữ nguyên.

⚡ MCP hỗ trợ bộ nhớ và quy trình làm việc đa bước, nghĩa là tác nhân của bạn có thể ghi nhớ thông tin giữa các tác vụ và xâu chuỗi các hành động lại với nhau một cách thông minh.

⚡ Nó dẫn đến ít "ảo giác" hơn. Nói chung, MCP sử dụng các định nghĩa công cụ rõ ràng, có cấu trúc. Điều đó giúp AI hoạt động chính xác hơn.

### Tại sao MCP lại quan trọng?

MCP quan trọng vì:

- Nó biến `giấc mơ về một trợ lý AI phổ quát` cho các nhà phát triển thành hiện thực.
- Tiềm năng kết hợp các hành động này thành các quy trình làm việc phức tạp (với AI xử lý logic) sẽ dẫn đến `một kỷ nguyên mới của tự động hóa thông minh`.

Vì vậy, MCP giúp các nhà phát triển làm việc với AI dễ dàng và hiệu quả hơn rất nhiều.

---

## 5. 3 Lớp của MCP (và cách tôi đã hiểu chúng một cách đơn giản).

Đây là cách tôi đã hiểu khái niệm này. Tôi sẽ đưa ra một ví dụ phổ biến để giúp bạn hiểu nó nhanh chóng.

⚡ Mô hình ↔ Ngữ cảnh: “Giao tiếp với LLM theo cách nó hiểu”

Hãy tưởng tượng Mô hình là bộ não của một robot (`LLM`). Nó có thể xử lý thông tin nhưng cần hướng dẫn rõ ràng. Ngữ cảnh cung cấp những hướng dẫn đó để robot hoạt động chính xác.

Ví dụ: nếu bạn nói với một robot, "Làm cho tôi một chiếc bánh sandwich," điều đó quá mơ hồ. Nhưng nói "Sử dụng bánh mì, giăm bông và phô mai này để làm bánh sandwich" cung cấp cho robot ngữ cảnh cần thiết để hiểu và thực hiện tác vụ.

- Mô hình là robot (LLM).
- Ngữ cảnh là các hướng dẫn cụ thể bạn đưa cho nó (nguyên liệu làm bánh sandwich).

⚡ Ngữ cảnh ↔ Giao thức: “Cung cấp cho LLM bộ nhớ có cấu trúc, công cụ, trạng thái”

Khi robot có hướng dẫn (Ngữ cảnh), nó cần một cách để tuân theo chúng, ghi nhớ chi tiết và sử dụng công cụ. Điều đó được thực hiện bởi `Giao thức`, là hệ thống cho phép robot sử dụng bộ nhớ và công cụ để hoàn thành công việc.

Quay lại ví dụ về bánh sandwich. Cung cấp cho robot một giao thức sẽ giúp nó ghi nhớ các nguyên liệu, biết cách cầm dao và nhiều hơn nữa.

- Ngữ cảnh cho robot biết phải làm gì.
- Giao thức cung cấp cho nó các công cụ và bộ nhớ để làm điều đó.

Đó là cấu trúc để hoàn thành công việc.

⚡ Giao thức ↔ Thời gian chạy (Runtime): “Thực sự chạy Tác nhân AI”

Robot biết phải làm gì (Ngữ cảnh) và cách làm (Giao thức). Bây giờ nó cần thực sự làm điều đó, điều này có thể thực hiện được bằng cách sử dụng Thời gian chạy.

Quay trở lại ví dụ về bánh sandwich, Thời gian chạy là khoảnh khắc robot bắt đầu thực hiện. Nó giống như môi trường nơi tác vụ trở nên sống động (như nhà bếp).

- Giao thức cung cấp cho robot phương pháp để tuân theo.
- Thời gian chạy là môi trường nơi robot thực sự làm việc.

Hãy kết hợp cả ba lớp lại với nhau và xem điều gì xảy ra khi sử dụng `phiên bản nhà hàng`.

- `Mô hình` là đầu bếp. Họ có kiến thức và kỹ năng để làm thức ăn.
- `Ngữ cảnh` là thực đơn. Nó cho đầu bếp biết những nguyên liệu nào cần thiết và món ăn nên trông và có vị như thế nào.
- `Giao thức` là người phục vụ. Người phục vụ mang đơn đặt hàng đến cho đầu bếp, giao tiếp chính xác cách chế biến món ăn và thậm chí ghi nhớ nếu bạn bị dị ứng với thứ gì đó.
- `Thời gian chạy` là nhà bếp nơi đầu bếp thực sự chuẩn bị bữa ăn. Đó là nơi tất cả các công cụ, nhiệt độ và việc chuẩn bị diễn ra.

Khi bạn hiểu các thành phần cốt lõi như máy chủ và máy khách (được đề cập trong phần "MCP hoạt động như thế nào"), mọi thứ sẽ bắt đầu có ý nghĩa.

Mỗi lớp khớp với nhau để làm cho toàn bộ hệ thống hoạt động.

---

## 6. Cách dễ nhất để kết nối hơn 100 máy chủ MCP được quản lý với xác thực tích hợp.

Trong phần này, chúng ta sẽ khám phá cách dễ nhất để kết nối Cursor với các máy chủ MCP.

Nếu bạn muốn khám phá cách thêm và sử dụng các máy chủ MCP tùy chỉnh trong Cursor, hãy đọc [tài liệu chính thức](https://docs.cursor.com/context/model-context-protocol).

### Bước 1: Điều kiện tiên quyết.

Cài đặt Node.js và đảm bảo `npx` có sẵn trong hệ thống của bạn.

### Bước 2: Bật máy chủ MCP trong Cursor.

Bạn có thể mở bảng lệnh trong Cursor bằng `Ctrl + Shift + P` và tìm kiếm "cursor settings".

![cursor settings](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpyhip49rwsnk7w02bu6o.png)

Bạn sẽ tìm thấy tùy chọn MCP trên thanh bên.

![mcp option in sidebar](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frbz0pzrflzwqxor3j4sy.png)

### Bước 3: Sử dụng máy chủ MCP có sẵn.

Chúng ta cũng có thể tạo một máy chủ từ đầu, nhưng hãy sử dụng cái có sẵn để đơn giản.

Chúng ta sẽ sử dụng Composio cho các máy chủ vì chúng có xác thực tích hợp. Bạn có thể tìm thấy danh sách tại [mcp.composio.dev](https://mcp.composio.dev/).

⚡ Xác thực tích hợp đi kèm với hỗ trợ OAuth, khóa API, JWT và Xác thực cơ bản. Điều này có nghĩa là bạn không cần tạo hệ thống đăng nhập của riêng mình.
⚡ Các máy chủ được quản lý hoàn toàn loại bỏ nhu cầu thiết lập phức tạp, giúp dễ dàng tích hợp các tác nhân AI với hơn 250 công cụ như Gmail, Slack, Notion, Linear và nhiều hơn nữa.
⚡ Cung cấp hơn 20.000 hành động API được xây dựng sẵn để tích hợp nhanh chóng mà không cần viết mã.
⚡ Có thể hoạt động cục bộ hoặc từ xa tùy thuộc vào nhu cầu cấu hình của bạn.
⚡ Độ chính xác gọi công cụ tốt hơn cho phép các tác nhân AI tương tác mượt mà với các ứng dụng được tích hợp.
⚡ Nó tương thích với các tác nhân AI, nghĩa là nó có thể kết nối các tác nhân AI với các công cụ để thực hiện các tác vụ như gửi email, tạo tác vụ hoặc quản lý vé trong một cuộc trò chuyện duy nhất.

Điều này cũng có nghĩa là ít thời gian ngừng hoạt động và ít vấn đề bảo trì hơn. Bạn có thể kiểm tra trạng thái tại [status.composio.dev/](https://status.composio.dev/).

![status composio](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsm17sx2jy7mkn6wb99b0.png)

Bạn có thể dễ dàng tích hợp với một loạt các máy chủ MCP hữu ích mà không cần viết bất kỳ dòng mã nào.

![composio](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcpkd5wirfdv58bz1ss9x.png)

Với mỗi tùy chọn, bạn sẽ tìm thấy tổng số người dùng hoạt động, phiên bản hiện tại, thời gian cập nhật gần đây nhất và tất cả các hành động có sẵn.

Bạn sẽ tìm thấy hướng dẫn cài đặt bằng `TypeScript`, `Python` và nó hỗ trợ Claude (MacOS), Windsurf (MacOS) và Cursor làm máy khách MCP.

Nếu bạn quan tâm đến việc xây dựng máy khách MCP từ đầu, hãy xem [hướng dẫn từng bước](https://composio.dev/blog/mcp-client-step-by-step-guide-to-building-from-scratch/) này của Harsh trên Composio.

![gmail composio](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhssy1izsb2mbegq52b7l.png)

### Bước 4: Tích hợp máy chủ MCP.

Đã đến lúc tích hợp một máy chủ với Cursor. Bây giờ, chúng ta sẽ sử dụng máy chủ MCP Gmail.

Trước đây là với SSE nhưng Cursor gần đây đã thay đổi phương pháp này bằng lệnh `npx`. Chúng ta sẽ cần tạo lệnh terminal. Kiểm tra [trang này](https://mcp.composio.dev/hackernews/thundering-petite-vulture-guyWBb) để tạo lệnh của bạn.

Lệnh terminal sẽ trông như sau:

```
npx @composio/mcp@latest setup "https://mcp.composio.dev/gmail/xyzxyz..." --client cursor
```

Bạn có thể chạy lệnh này trong terminal và khởi động lại Cursor để thấy các thay đổi.

Nếu bạn đang sử dụng Python, đây là cách bạn có thể cài đặt composio-toolset:

```
pip install composio_openai
```

```python
from composio_openai import ComposioToolSet, App
from openai import OpenAI

openai_client = OpenAI()
composio_toolset = ComposioToolSet(entity_id="default")
tools = composio_toolset.get_tools(apps=[App.GMAIL])
```

Bạn có thể đặt cấu hình cuối cùng ở hai vị trí, tùy thuộc vào trường hợp sử dụng của bạn:

1. Đối với các công cụ dành riêng cho một dự án, hãy tạo file `.cursor/mcp.json` trong thư mục dự án của bạn. Điều này cho phép bạn định nghĩa các máy chủ MCP chỉ khả dụng trong dự án cụ thể đó.
2. Đối với các công cụ bạn muốn sử dụng trên tất cả các dự án, hãy tạo file `~/.cursor/mcp.json` trong thư mục home của bạn. Điều này làm cho các máy chủ MCP khả dụng trong tất cả các không gian làm việc của Cursor. Lệnh terminal sẽ áp dụng tùy chọn thứ hai, làm cho nó có thể truy cập toàn cục.

![npx command](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F39igz0rqfh98q877jemu.png)

Nó sẽ hiển thị các hành động cần thiết và dấu chấm màu xanh lá cây cho biết rằng nó đã được tích hợp thành công.

![mcp gmail server](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8wno3z6kp0uxszplfb3k.png)

File `mcp.json` sẽ trông như sau:

```json
{
  "mcpServers": {
    "gmail_composio": {
      "url": "https://mcp.composio.dev/gmail/freezing-wrong-dress-7RHVw0"
    }
  }
}
```

Bạn có thể xem danh sách [các máy chủ và triển khai mẫu](https://modelcontextprotocol.io/examples). Bạn có thể tích hợp các máy chủ cộng đồng bằng cách làm theo cấu trúc này (dựa trên lựa chọn của bạn).

✅ Cấu hình máy chủ SSE

Cấu hình này được hỗ trợ trong Cursor và bạn có thể chỉ định trường `url` để kết nối với máy chủ SSE của mình.

```json
// Ví dụ này minh họa một máy chủ MCP sử dụng định dạng SSE
// Người dùng nên thiết lập và chạy máy chủ theo cách thủ công
// Điều này có thể được kết nối mạng, cho phép người khác truy cập nó
{
  "mcpServers": {
    "server-name": {
      "url": "http://localhost:3000/sse",
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

✅ Cấu hình máy chủ STDIO (Python)

Điều này thiết lập một máy chủ MCP sử dụng giao thức nhập/xuất chuẩn (STDIO) với một tập lệnh Python. Cách tiếp cận này chủ yếu được sử dụng cho phát triển cục bộ.

```json
// nếu bạn đang sử dụng máy chủ CLI Python
// Ví dụ này minh họa một máy chủ MCP sử dụng định dạng stdio
// Cursor tự động chạy tiến trình này cho bạn
// Điều này sử dụng một máy chủ Python, chạy bằng `python`
{
  "mcpServers": {
    "server-name": {
      "command": "python",
      "args": ["mcp-server.py"],
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

✅ Cấu hình máy chủ STDIO (Node.js)

```json
// nếu bạn đang sử dụng máy chủ CLI Node.js
// Ví dụ này minh họa một máy chủ MCP sử dụng định dạng stdio
// Cursor tự động chạy tiến trình này cho bạn
// Điều này sử dụng một máy chủ Node.js, chạy bằng `npx`
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "mcp-server"],
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

### Bước 5: Sử dụng máy chủ trực tiếp trong Tác nhân.

Trước khi tiếp tục, hãy đảm bảo kiểm tra các hành động có sẵn trên [trang máy chủ mcp composio](https://mcp.composio.dev/gmail/freezing-wrong-dress-7RHVw0). Bạn cũng có thể tìm thấy các công cụ và hành động trên bảng điều khiển.

![tools](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxuq7ahgz2k87xpwprdtd.png)

Bạn có thể mở Chat bằng lệnh `Ctrl + I`.

Bạn có thể bật `Chế độ Tác nhân` là chế độ tự động nhất trong Cursor, được thiết kế để xử lý các tác vụ mã hóa phức tạp với hướng dẫn tối thiểu.

![agent mode](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F63aj5z7bu9u6w2raprfv.png)

Tôi thích có một số quyền kiểm soát trước khi thực thi, vì vậy tôi sẽ sử dụng chế độ mặc định. Bạn có thể nhập bất kỳ truy vấn nào và nhấp vào nút `chạy công cụ`.

Như bạn có thể thấy, nó sẽ gọi máy chủ MCP thích hợp (nếu bạn có nhiều máy chủ) và sử dụng hành động chính xác dựa trên lời nhắc của bạn.

![run tool option](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1oy7shupeqmuymvpzhiu.png)

Vì không có kết nối hoạt động, nó sẽ thiết lập một kết nối trước. Bạn sẽ cần ủy quyền cho quá trình này.

![cursor establish connection](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvn5q8u43brlsq2s189yh.png)

![gmail authorize](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwtt2is3b9x4733bzwup0.png)

Tôi đang sử dụng một tài khoản giả (mà tôi đã tạo từ lâu) và tôi khuyên bạn nên làm tương tự cho mục đích thử nghiệm. Sau khi bạn hài lòng, bạn có thể tự động hóa mọi thứ bằng tài khoản chính của mình.

Như bạn có thể thấy, nó đã tìm nạp email một cách chính xác.

![fetched emails](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Futh3owpr9lmq7w6v441a.png)

Hãy kiểm tra bằng cách gửi email đến `hi@anmolbaranwal.com` với chủ đề "Demo of Composio" và nói rằng đang thử nghiệm máy chủ MCP trong nội dung email.

![output](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7s2ebz8sxhy19q6j9pcg.png)

Như bạn có thể thấy, tôi đã nhận được email đó với chủ đề và nội dung chính xác như đã chỉ định trong lời nhắc.

![sent email](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmthdbkdhjc6mr608iytu.png)
_Đã gửi email_

![testing email](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Farjcxnbzf15itmgzjqoe.png)
_Đã nhận email_

Với máy chủ MCP này, bạn có thể làm rất nhiều điều tuyệt vời như `Get attachments`, `Create email draft`, `Modify thread labels`, `Reply to a thread`, `get contacts`, `delete message`, `move to trash`, `search people`, `send email` và nhiều hơn nữa.

Và hãy luôn nhớ rằng, có giới hạn cho những gì bạn có thể làm. Tôi đã thử nghiệm với hơn 15 lời nhắc để kiểm tra các trường hợp đặc biệt.

![conversation too long](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F934iwt6idiv3qr3oxido.png)

---

## 7. Sáu ví dụ thực tế với các bản demo.

Dưới đây là sáu ví dụ thực tế về các máy chủ MCP. Hãy cùng thảo luận về quy trình và xem ứng dụng của chúng.

### ✅ [Máy chủ MCP YouTube](https://mcp.composio.dev/youtube/freezing-wrong-dress-7RHVw0)

![youtube mcp server](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3wvh9rm6vopv1y85i46t.png)

Chúng ta sẽ làm theo quy trình tương tự như đã thảo luận trước đó. Bạn có thể kiểm tra máy chủ Composio cho YouTube nơi bạn có thể tạo url.

![generate url](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpoldjjcogcal5mp7xvh6.png)

Lệnh sẽ có cấu trúc như sau:

```
npx @composio/mcp@latest setup "https://mcp.composio.dev/youtube/freezing-wrong-dress-xyz" --client cursor
```

Nếu bạn để ý trong `mcp.json`, url cho youtube sẽ được thêm vào ngay sau khi bạn chạy lệnh terminal. Nó sẽ trông giống như sau:

```json
{
  "mcpServers": {
    "youtube_composio": {
      "url": "https://mcp.composio.dev/youtube/freezing-wrong-dress-7RHVw0"
    }
  }
}
```

Vì không có kết nối hoạt động, nó sẽ thiết lập một kết nối trước. Bạn sẽ cần xác thực bằng cách sao chép URL OAuth vào trình duyệt.

![establish connection](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhh53jfwz2f1uthjamu1f.png)

Bạn sẽ phải cấp quyền truy cập cho máy chủ để nó có thể thực hiện hành động dựa trên lời nhắc của bạn.

![access](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2C
format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3rm5m3l03bh1dnxshmmq.png)

Bây giờ, bạn có thể đưa ra bất kỳ lời nhắc nào như `Tìm nạp cho tôi 5 video hàng đầu về Model Context Protocol dựa trên lượt xem và lượt thích.`

Nó sẽ tạo ra phản hồi tương ứng.

![output](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F98b1nmliukvsljxa7btx.png)

![output](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flz1pncbs77dprr72bwni.png)

Với máy chủ MCP này, bạn có thể làm rất nhiều điều tuyệt vời như `Tìm kiếm video, kênh, danh sách phát trên youtube`, `tìm nạp số liệu thống kê video`, `tải phụ đề`, `đăng ký kênh`, `cập nhật siêu dữ liệu video`, `cập nhật hình thu nhỏ` và nhiều hơn nữa.

### ✅ [Máy chủ MCP Ahrefs](https://mcp.composio.dev/ahrefs/freezing-wrong-dress-7RHVw0)

Chúng ta sẽ làm theo quy trình tương tự như đã thảo luận trước đó. Bạn có thể kiểm tra [máy chủ Composio cho Ahrefs](https://mcp.composio.dev/ahrefs/freezing-wrong-dress-7RHVw0).

Nếu bạn chưa biết, Ahrefs là một nền tảng SEO và marketing cung cấp các công cụ kiểm tra trang web, nghiên cứu từ khóa, phân tích nội dung và thông tin chi tiết về đối thủ cạnh tranh nhằm cải thiện thứ hạng tìm kiếm và thúc đẩy lưu lượng truy cập tự nhiên.

![ahrefs](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb9xli5u9yjrpijpcp6tt.png)

Nếu bạn để ý trong `mcp.json`, url ahrefs sẽ được thêm vào.

```json
"ahrefs_composio": {
  "url": "https://mcp.composio.dev/ahrefs/freezing-wrong-xyz"
}
```

Như bạn có thể thấy, nó đang thiết lập kết nối lần đầu tiên.

![permission required for establish connection](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1r140mrfwakf9fcnm5sn.png)

Sau khi bạn làm điều đó, bạn sẽ có thể sử dụng tất cả các hành động như `truy xuất từ khóa tự nhiên`, `tìm nạp tất cả các liên kết ngược`, `lịch sử xếp hạng tên miền`, `tổng quan trang theo lưu lượng truy cập`, `truy xuất ip trình thu thập công khai`, `tìm nạp tổng quan đối thủ cạnh tranh`, `liệt kê tốt nhất theo liên kết ngoài`, `tìm nạp tổng lịch sử khối lượng tìm kiếm` và nhiều hơn nữa.

Xin lưu ý rằng bạn sẽ cần một API (đi kèm với gói cao cấp của Ahrefs) để hoàn tất tích hợp.

### ✅ [Máy chủ MCP LinkedIn](https://mcp.composio.dev/linkedin/old-damaged-thailand-X5G8GA)

Bạn có thể làm theo quy trình tương tự để tạo URL và chạy nó trong terminal. Sau đó, bạn sẽ cần thiết lập kết nối và xác thực bằng cách sao chép URL OAuth vào trình duyệt.

Bạn sẽ nhận được thông báo xác nhận sau khi hoàn tất.

![confirmation message](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3acmx5eq6mui7wjtzdvo.png)

Bạn cũng có thể kiểm tra điều đó dựa trên các hành động của máy chủ. Như bạn có thể thấy, có một kết nối hoạt động.

![active connection](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3beakqf1gb8h7h2n7dk8.png)

Tôi thường không khuyên bạn nên sử dụng nó trên các tài khoản chính thức của mình vì bạn không bao giờ có thể quá cẩn thận.

Với máy chủ MCP này, bạn có các tùy chọn như `lấy thông tin hồ sơ`, `tạo bài đăng`, `lấy thông tin công ty` và `xóa bài đăng`.

### ✅ [Tự động đảo ngược kỹ thuật ứng dụng bằng Máy chủ MCP Ghidra](https://github.com/lauriewired/ghidramcp).

Máy chủ MCP này cho phép LLM tự động đảo ngược kỹ thuật các ứng dụng. Nó hiển thị nhiều công cụ từ chức năng cốt lõi của [Ghidra](https://ghidra-sre.org/) cho các máy khách MCP.

Điều này bao gồm dịch ngược và phân tích các tệp nhị phân, tự động đổi tên các phương thức và dữ liệu, và liệt kê các phương thức, lớp, nhập và xuất.

Một vài trường hợp sử dụng:

⚡ Phân tích lỗ hổng tự động bằng LLM.
⚡ Đảo ngược kỹ thuật các mẫu phần mềm độc hại.

Đây là bản demo.

[![Ghidra MCP](https://img.youtube.com/vi/uH7Apjh-P_A/0.jpg)](https://www.youtube.com/watch?v=uH7Apjh-P_A)

[Kho lưu trữ GitHub](https://github.com/lauriewired/ghidramcp) có 4k sao.

### ✅ [Đọc và sửa đổi thiết kế Figma theo chương trình](https://github.com/sonnylazuardi/cursor-talk-to-figma-mcp)

Gần đây đã có những phát triển trong việc tạo thiết kế Figma thành các ứng dụng sẵn sàng sản xuất.

Dự án này triển khai điều tương tự bằng cách sử dụng tích hợp MCP giữa Cursor AI và Figma, cho phép Cursor giao tiếp với Figma để đọc thiết kế và sửa đổi chúng theo chương trình.

Nó cho phép tạo tài liệu và lựa chọn, chú thích, tạo phần tử, tạo kiểu, bố cục và nhiều hơn nữa.

Bạn chỉ cần nói: `design a modern-looking signup screen for mobile` và nó sẽ tạo ra nó mà không cần bạn tương tác với tệp Figma.

Đây là bản demo.

[![Cursor talk to figma MCP](https://img.youtube.com/vi/71_6kqzK8Ag/0.jpg)](https://www.youtube.com/watch?v=71_6kqzK8Ag)

Nó có 3.1k sao trên GitHub.

Bạn có thể kiểm tra [Kho lưu trữ GitHub](https://github.com/sonnylazuardi/cursor-talk-to-figma-mcp) và [Tweet chính thức](https://x.com/sonnylazuardi/status/1901325190388428999).

Có một [máy chủ MCP](https://github.com/GLips/Figma-Context-MCP) tuyệt vời khác (với 5k sao trên GitHub) cung cấp thông tin bố cục Figma cho các tác nhân mã hóa AI.

### ✅ [Tạo cảnh 3D bằng Blender MCP](https://github.com/ahujasid/blender-mcp)

Việc tạo các mô hình 3D luôn khiến nhiều người sáng tạo sợ hãi do sự phức tạp liên quan.

Điều này kết nối Blender với Claude AI thông qua Model Context Protocol (MCP), cho phép Claude tương tác trực tiếp và kiểm soát Blender.

Tích hợp này cho phép mô hình hóa 3D được hỗ trợ bằng lời nhắc, tạo và thao tác cảnh. Bạn có thể xem [hướng dẫn đầy đủ](https://www.youtube.com/watch?v=lCyQ717DuzQ) nếu bạn quan tâm đến việc sử dụng nó.

Ví dụ lời nhắc với video demo:

⚡ "Tạo một cảnh low poly trong một hầm ngục, với một con rồng canh giữ một nồi vàng"

[![Blender MCP Demo: AI Prompting a dragon in a dungeon](https://img.youtube.com/vi/DqgKuLYUv00/0.jpg)](https://www.youtube.com/watch?v=DqgKuLYUv00)

⚡ "Lấy thông tin về cảnh hiện tại và tạo một bản phác thảo threejs từ đó"

[![Blender MCP Demo: Creating a Threejs scene from Blender](https://img.youtube.com/vi/jxbNI5L7AH8/0.jpg)](https://www.youtube.com/watch?v=jxbNI5L7AH8)

⚡ "Tạo không khí bãi biển bằng HDRIs, kết cấu và các mô hình như đá và thực vật từ Poly Haven"

[![Blender MCP: Use Polyhaven assets](https://img.youtube.com/vi/I29rn92gkC4/0.jpg)](https://www.youtube.com/watch?v=I29rn92gkC4)

Nếu bạn quan tâm đến nhiều bản demo hơn, người sáng lập đã tạo một [chuỗi trên X với các ví dụ ấn tượng](https://x.com/sidahuj/status/1909986466723168344) về những gì người khác đã tạo.

[Kho lưu trữ GitHub](https://github.com/ahujasid/blender-mcp) có 10.3k sao trên GitHub.

---

## 8. Một số hạn chế của MCP.

Kỳ vọng và thực tế về MCP có thể rất khác nhau. Bạn sẽ hiểu ý tôi khi đọc qua các điểm sau.

![mcp expectations vs mcp reality](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5nfjuqt0h5vrvmpn7m9m.png)
_Tín dụng thuộc về nhóm Builder.io_

Đừng hiểu lầm, MCP rất hứa hẹn nhưng đây là một số hạn chế bạn nên biết:

⚡ Không phải tất cả các nền tảng AI đều hỗ trợ MCP.

Claude (đặc biệt là với ứng dụng desktop của nó) và các công cụ như Cursor hoặc Windsurf hỗ trợ MCP trực tiếp. Nhưng nếu bạn đang sử dụng thứ gì đó như ChatGPT hoặc mô hình LLaMA cục bộ, nó có thể không hoạt động ngay lập tức.

Có một số công cụ mã nguồn mở đang cố gắng giải quyết vấn đề này, nhưng cho đến khi MCP được áp dụng rộng rãi hơn, việc hỗ trợ trên tất cả các trợ lý AI là khó khăn.

⚡ Tính tự chủ của tác nhân chưa hoàn hảo.

MCP cung cấp khả năng nhưng khả năng phán đoán của AI vẫn chưa hoàn hảo.

Ví dụ, việc sử dụng công cụ phụ thuộc vào mức độ mô hình hiểu mô tả công cụ và ngữ cảnh sử dụng. Nó thường cần điều chỉnh lời nhắc hoặc logic phía tác nhân để cải thiện độ tin cậy.

⚡ Chi phí hiệu suất.

Sử dụng công cụ thông qua MCP làm tăng chi phí. Mỗi lệnh gọi là bên ngoài và có thể chậm hơn nhiều so với việc AI tự trả lời. Ví dụ, việc lấy dữ liệu từ một trang web thông qua công cụ MCP có thể mất vài giây, trong khi mô hình có thể đoán câu trả lời từ dữ liệu huấn luyện trong mili giây.

Bây giờ, nếu bạn đang điều phối nhiều công cụ, độ trễ sẽ cộng dồn, giống như gọi 5 máy chủ MCP khác nhau theo trình tự để:

- Tìm nạp tệp từ Google Drive
- Tóm tắt nội dung bằng công cụ LLM
- Dịch bản tóm tắt
- Tạo một tweet dựa trên bản dịch
- Lên lịch bằng công cụ truyền thông xã hội như Buffer

Chuỗi đó có thể mất 10–15 giây, tùy thuộc vào thời gian phản hồi của máy chủ.

Một số tác nhân có thể xử lý việc sử dụng công cụ song song để bạn có thể tối ưu hóa thêm quá trình này.

⚡ Vấn đề tin cậy.

Cho phép AI thực hiện các hành động thực tế có thể cảm thấy rủi ro. Ngay cả khi AI thường làm đúng, người dùng thường muốn xem xét mọi thứ trước khi chúng xảy ra.

Hiện tại, hầu hết các công cụ đều hoàn toàn tự chủ hoặc không tự chủ chút nào. Hiếm khi có một điểm trung gian nơi AI có thể tận dụng tính tự chủ nhưng vẫn trao quyền kiểm soát cho người dùng khi cần thiết. Tất cả chúng ta đều cần một `con người trong vòng lặp`.

❌ Cách tiếp cận tồi: AI gửi email ngay lập tức mà không hỏi.
✅ Cách tiếp cận tốt hơn: AI nói, `Tôi sắp gửi email X với tin nhắn này, có được phép gửi không?` và chỉ hành động sau khi bạn chấp thuận.

⚡ Vấn đề khả năng mở rộng.

Hầu hết các máy chủ MCP hiện nay được xây dựng cho người dùng đơn lẻ, thường chỉ chạy trên máy tính xách tay của nhà phát triển.

Một máy chủ MCP phục vụ nhiều tác nhân hoặc người dùng độc lập vẫn chưa được khám phá nhiều. Để làm được điều đó, các công ty cần xử lý các vấn đề phức tạp hơn như các yêu cầu đồng thời, ngữ cảnh dữ liệu riêng biệt và thực thi giới hạn tốc độ sử dụng.

Đây là một lĩnh vực mà hệ sinh thái vẫn còn chỗ để phát triển, đặc biệt là với các ý tưởng như cổng MCP hoặc các khung máy chủ MCP sẵn sàng cho doanh nghiệp.

⚡ Tiêu chuẩn bảo mật.

MCP không đi kèm với xác thực hoặc ủy quyền tích hợp.

`Xác thực & Ủy quyền`: MCP không có hỗ trợ tích hợp cho việc xác thực người dùng hoặc tác nhân. Nếu bạn hiển thị máy chủ MCP qua mạng, bạn phải thêm bảo mật của riêng mình.

Một số triển khai sử dụng OAuth 2.1 để thêm phạm vi quyền (`chỉ đọc hoặc chỉ ghi`), nhưng hiện tại không có cách tiếp cận chuẩn, vì vậy mỗi máy chủ xử lý xác thực khác nhau.

`Quyền chính xác`: Lý tưởng nhất là các tác nhân chỉ nên sử dụng các công cụ mà họ cần. Nhưng nếu có nhiều công cụ mạnh mẽ có sẵn (như truy cập trình duyệt và terminal), không có gì ngăn cản AI sử dụng sai công cụ, trừ khi bạn tắt nó thủ công.

`Tiêm lời nhắc`: AI có thể mắc lỗi nếu nó hiểu sai lời nhắc. Tệ hơn nữa, ai đó có thể tạo ra một lời nhắc độc hại để lừa AI làm điều gì đó có hại (`prompt injection`). Các biện pháp bảo vệ phụ thuộc vào cách mỗi máy chủ MCP được xây dựng.

Nếu bạn muốn hiểu cách giảm thiểu rủi ro bảo mật trong các triển khai MCP, hãy đọc bài viết này:

- [Hiểu và giảm thiểu rủi ro bảo mật trong các triển khai MCP](https://techcommunity.microsoft.com/blog/microsoft-security-blog/understanding-and-mitigating-security-risks-in-mcp-implementations/4404667) của Microsoft.
- [Rủi ro bảo mật của MCP](https://www.pillar.security/blog/the-security-risks-of-model-context-protocol-mcp) của Pillar.

MCP vẫn còn mới. Sẽ có những phát triển tiếp theo để giải quyết nhiều trường hợp đặc biệt hơn khi nhu cầu được phát hiện.

Về phía mô hình AI, chúng ta có thể sẽ thấy các mô hình được tinh chỉnh đặc biệt cho việc sử dụng công cụ và MCP. Anthropic đã đề cập đến các `mô hình AI được tối ưu hóa cho tương tác MCP` trong tương lai.

---

Dưới đây là một số tài nguyên hữu ích nếu bạn đang có kế hoạch xây dựng MCP:

- [mcp-chat](https://github.com/Flux159/mcp-chat) - là một máy khách chat CLI cho các máy chủ MCP. Được sử dụng để kiểm tra và đánh giá các máy chủ và tác nhân MCP.
- [mastra registry](https://mastra.ai/mcp-registry-registry) - một bộ sưu tập các thư mục máy chủ MCP để kết nối AI với các công cụ yêu thích của bạn.
- [smithery.ai](https://smithery.ai/) - mở rộng tác nhân của bạn với 4.630 khả năng thông qua các máy chủ MCP. Rất nhiều chi tiết bao gồm `lượt gọi công cụ hàng tháng`, `tùy chọn cục bộ`, `công cụ`, `API`, `hướng dẫn cài đặt cho các máy khách khác nhau`.
- [Thư mục máy chủ MCP phổ biến của nhóm chính thức](https://github.com/modelcontextprotocol/servers) - 20k sao trên GitHub.
- [Thư mục Cursor](https://cursor.directory/mcp) với hơn 1800 máy chủ MCP.
- [Những MCP đó hoàn toàn tăng gấp 10 lần quy trình làm việc của Cursor của tôi](https://www.youtube.com/watch?v=oAoigBWLZgE) - Video YouTube với các trường hợp sử dụng thực tế.

---

MCP vẫn đang phát triển nhưng các ý tưởng cốt lõi của nó sẽ tồn tại và tôi đã cố gắng hết sức để giải thích các khái niệm. Tôi hy vọng bạn tìm thấy điều gì đó hữu ích.

Một cuộc trò chuyện duy nhất với Tác nhân có thể giúp bạn tự động hóa các quy trình làm việc phức tạp.

Bây giờ hãy bắt tay vào xây dựng một điều gì đó ấn tượng với MCP và cho cả thế giới thấy.

Chúc một ngày tốt lành!
