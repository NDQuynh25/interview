# Câu hỏi và trả lời phỏng vấn Backend Node.js & AI

Dành cho vị trí có 2–3 năm kinh nghiệm, theo JD: Node.js, Express/NestJS, MySQL, PostgreSQL, MongoDB, Redis, Elasticsearch, MVC, Microservices, Gitflow và ứng dụng AI trong lập trình.

Các câu trả lời là mẫu để luyện nói, không phải mô tả kinh nghiệm đã được xác nhận của bạn. Thay phần `[trong ngoặc]` bằng thông tin thật; không nhận đã triển khai production nếu mới thực hành. Mỗi câu nên trả lời trong 1–2 phút, sau đó mở rộng bằng ví dụ khi được hỏi sâu.

## Mục lục

1. [Giới thiệu và kinh nghiệm](#1-giới-thiệu-và-kinh-nghiệm)
2. [Node.js và JavaScript](#2-nodejs-và-javascript)
3. [Express.js và NestJS](#3-expressjs-và-nestjs)
4. [Database, Redis và Elasticsearch](#4-database-redis-và-elasticsearch)
5. [Kiến trúc và thiết kế hệ thống](#5-kiến-trúc-và-thiết-kế-hệ-thống)
6. [Bảo mật, kiểm thử và Git](#6-bảo-mật-kiểm-thử-và-git)
7. [AI agent, Claude Code và review code](#7-ai-agent-claude-code-và-review-code)
8. [LLM API, RAG và MCP](#8-llm-api-rag-và-mcp)
9. [Tình huống tổng hợp và hỏi ngược](#9-tình-huống-tổng-hợp-và-hỏi-ngược)

## 1. Giới thiệu và kinh nghiệm

### 01. Em hãy giới thiệu bản thân và mức độ phù hợp với vị trí này.

**Trả lời mẫu:**

Em tốt nghiệp `[ngành, trường]`, có `[thời gian]` làm backend với `[công nghệ thực tế]`. Trong dự án `[tên]`, em phụ trách `[module]`, từ làm rõ yêu cầu, thiết kế API và dữ liệu đến kiểm thử và theo dõi sau triển khai. Một bài toán em trực tiếp giải quyết là `[vấn đề]`, với kết quả `[kết quả đã kiểm chứng]`.

Em dùng `[công cụ AI thực sự đã dùng]` cho các việc như phân tích code, đề xuất thay đổi và bổ sung test. Em đọc lại diff, kiểm tra nghiệp vụ và chịu trách nhiệm về phần code đưa vào sản phẩm. Vị trí này phù hợp với hướng phát triển backend và cách làm việc đó của em.

### 02. Hãy mô tả một dự án em hiểu sâu nhất.

**Trả lời mẫu:**

Em sẽ trình bày theo năm ý: bài toán người dùng, kiến trúc, phần em chịu trách nhiệm, quyết định kỹ thuật và kết quả. Ví dụ: `[sản phẩm]` giải quyết `[nhu cầu]`; backend dùng `[framework]`, dữ liệu lưu ở `[database]`. Em trực tiếp làm `[phạm vi]`.

Quyết định đáng chú ý là `[lựa chọn]` vì `[ràng buộc]`. Đánh đổi là `[chi phí hoặc giới hạn]`. Em kiểm chứng bằng `[test, log, metric]`. Nếu làm lại, em sẽ cải thiện `[điểm cụ thể]`.

**Chuẩn bị thêm:** Một sơ đồ kiến trúc và một luồng request từ client đến database. Phân biệt rõ phần mình làm và phần của nhóm.

### 03. Nếu chưa dùng một công nghệ trong JD ở production, em trả lời thế nào?

**Trả lời mẫu:**

Em chưa vận hành `[công nghệ]` ở production. Em đã `[đọc tài liệu/làm bài thực hành]` và hiểu vai trò của nó trong `[bài toán]`. Nếu nhận task, em sẽ đọc cách dự án hiện tại sử dụng, xác nhận yêu cầu dữ liệu và tải, làm thử phạm vi nhỏ, viết test và nhờ review những điểm vận hành chưa có kinh nghiệm. Em sẽ nói rõ phần đã kiểm chứng và phần còn là giả định.

## 2. Node.js và JavaScript

### 04. Node.js là single-threaded, vì sao xử lý được nhiều request?

**Trả lời mẫu:**

JavaScript trên một event loop thường chạy trên một thread. Khi chờ I/O, Node.js có thể giao việc cho hệ điều hành hoặc cơ chế nền phù hợp, rồi chạy callback khi kết quả sẵn sàng. Nhờ vậy, nhiều request có thể cùng chờ I/O mà không cần một JavaScript thread riêng cho mỗi request.

Nếu em chạy vòng lặp CPU dài trên thread đó, các request khác vẫn bị chậm. Với việc nặng CPU, em cân nhắc worker threads, process riêng hoặc dịch vụ xử lý nền.

Nguồn: [Node.js Event Loop](https://nodejs.org/learn/asynchronous-work/event-loop-timers-and-nexttick).

### 05. Dùng async/await có làm code hết block event loop không?

**Trả lời mẫu:**

Không. `async/await` giúp viết luồng bất đồng bộ dễ đọc, không tự chuyển tính toán sang thread khác. Hàm `async` chứa vòng lặp lớn hoặc xử lý JSON rất lớn vẫn có thể block. Em phân biệt thời gian chờ I/O với thời gian tính toán CPU trước khi chọn cách xử lý.

Nguồn: [Don't Block the Event Loop](https://nodejs.org/learn/asynchronous-work/dont-block-the-event-loop).

### 06. Khi nào dùng Promise.all, Promise.allSettled hoặc chạy tuần tự?

**Trả lời mẫu:**

`Promise.all` phù hợp khi các tác vụ độc lập và cần đủ kết quả; promise tổng reject khi có một tác vụ reject, nhưng các tác vụ đã khởi chạy không tự bị hủy. `Promise.allSettled` phù hợp khi cần thu kết quả thành công và thất bại của từng tác vụ. Em chạy tuần tự khi bước sau phụ thuộc bước trước.

Với hàng nghìn tác vụ, em giới hạn concurrency để tránh làm cạn connection pool hoặc chạm rate limit. Không gọi `Promise.all` trên toàn bộ danh sách một cách mặc định.

### 07. setTimeout(callback, 0) có chạy ngay không?

**Trả lời mẫu:**

Không. Nó lên lịch callback sau ngưỡng thời gian tối thiểu, còn thời điểm thực thi phụ thuộc event loop và công việc đang chờ. Trong tình huống đơn giản, code đồng bộ chạy trước, rồi các microtask của Promise được xử lý trước callback timer. Em không dùng timer để đảm bảo thứ tự nghiệp vụ hoặc tính đúng đắn của giao dịch.

### 08. Em xử lý lỗi bất đồng bộ như thế nào?

**Trả lời mẫu:**

Em `await` promise trong `try/catch` khi cần chuyển đổi lỗi hoặc xử lý tại chỗ; nếu không, em để lỗi được truyền đến cơ chế xử lý lỗi tập trung của framework. Với tác vụ chạy nền, phải có nơi nhận lỗi và theo dõi kết quả, tránh promise bị bỏ quên.

Em phân biệt lỗi nghiệp vụ, lỗi input và lỗi hạ tầng, trả mã phù hợp, ghi log có request ID và không trả stack trace hay thông tin nhạy cảm cho client. Em không catch rồi bỏ qua lỗi khiến hệ thống báo thành công giả.

### 09. API chậm thì em điều tra thế nào?

**Trả lời mẫu:**

Em xác định endpoint, thời điểm và nhóm request bị ảnh hưởng, rồi xem latency p95/p99, error rate và throughput. Em dùng log hoặc trace để tách thời gian ở ứng dụng, database và API bên ngoài. Sau đó kiểm tra CPU, memory, event loop lag, connection pool và slow query.

Nếu query chậm, em xem execution plan và lượng dữ liệu đọc. Nếu CPU cao, em profile đoạn xử lý. Mỗi thay đổi đều được so sánh trước/sau dưới tải tương đương; em không tăng server hoặc thêm cache trước khi xác định nguyên nhân.

### 10. Em nhận biết và xử lý memory leak ra sao?

**Trả lời mẫu:**

Em nghi ngờ leak khi memory tăng kéo dài qua nhiều chu kỳ tải và không giảm về mức hợp lý sau garbage collection. Em quan sát riêng heap, RSS và external memory, rồi dùng profiling hoặc so sánh heap snapshot trong môi trường phù hợp.

Những chỗ thường kiểm tra là collection không giới hạn, cache không TTL, listener không gỡ, timer không dọn và dữ liệu request bị giữ lại. Restart chỉ giúp khôi phục tạm thời; em vẫn cần tìm nơi giữ reference và kiểm chứng sau sửa. Heap snapshot có thể chứa dữ liệu nhạy cảm nên cần kiểm soát nơi lưu.

## 3. Express.js và NestJS

### 11. Express và NestJS khác nhau thế nào? Khi nào chọn mỗi loại?

**Trả lời mẫu:**

Express cung cấp các thành phần HTTP và routing gọn, cho phép nhóm tự tổ chức cấu trúc. NestJS đưa ra cấu trúc module, dependency injection và các thành phần như guard, pipe, interceptor, giúp thống nhất cách làm trong dự án lớn hơn.

Em chọn theo quy mô, độ phức tạp và kinh nghiệm nhóm. Express phù hợp khi cần cấu trúc gọn và nhóm có quy ước rõ. NestJS phù hợp khi cần nhiều module, dependency rõ và dễ thay thế khi test. Framework không tự đảm bảo chất lượng; cách chia trách nhiệm vẫn quyết định khả năng bảo trì.

### 12. Request lifecycle trong NestJS diễn ra thế nào?

**Trả lời mẫu:**

Ở mức tổng quát: middleware → guard → interceptor trước xử lý → pipe → controller và service → interceptor sau xử lý → response. Exception filter xử lý exception phù hợp khi lỗi chưa được bắt ở nơi khác. Luồng thực tế còn phụ thuộc phạm vi đăng ký và vị trí phát sinh lỗi.

Em dùng middleware cho xử lý request chung, guard để quyết định có cho truy cập, pipe cho validate/transform, interceptor cho hành vi bao quanh handler, và exception filter để chuẩn hóa lỗi.

Nguồn: [NestJS Request Lifecycle](https://docs.nestjs.com/faq/request-lifecycle).

### 13. Dependency Injection có lợi gì?

**Trả lời mẫu:**

DI đưa dependency từ bên ngoài vào thay vì để class tự tạo mọi thứ. Ví dụ `OrderService` nhận payment provider qua constructor; khi test có thể thay bằng fake provider, khi thay nhà cung cấp ít phải sửa nghiệp vụ.

Với TypeScript, interface không tồn tại ở runtime, nên khi cần inject qua abstraction em dùng token hoặc abstract class phù hợp. Em tránh tạo abstraction không có nhu cầu vì sẽ làm code khó lần theo.

### 14. Authentication và authorization khác nhau thế nào?

**Trả lời mẫu:**

Authentication xác định người gọi là ai; authorization xác định người đó được làm gì trên tài nguyên cụ thể. Có token hợp lệ chưa có nghĩa là được xem mọi đơn hàng. Với API lấy đơn, em kiểm tra quyền và điều kiện sở hữu hoặc tenant ở phía server.

Em không tin `userId`, `role` hay `tenantId` do client tự gửi làm căn cứ phân quyền. Test cần có trường hợp user A truy cập tài nguyên của user B.

### 15. Em tổ chức controller, service và lớp truy cập dữ liệu thế nào?

**Trả lời mẫu:**

Controller nhận request và chuyển thành lời gọi use case. Service xử lý quy tắc nghiệp vụ, orchestration và ranh giới transaction. Lớp truy cập dữ liệu đóng gói các thao tác query khi việc tách này thực sự có ích.

Ví dụ tạo đơn: controller nhận DTO; service kiểm tra nghiệp vụ, tính giá từ dữ liệu tin cậy và điều phối lưu dữ liệu; lớp truy cập dữ liệu thực hiện query. Các thao tác thuộc cùng transaction phải dùng đúng transaction context, tránh một query vô tình chạy ngoài transaction.

## 4. Database, Redis và Elasticsearch

### 16. Khi nào chọn MySQL, PostgreSQL hoặc MongoDB?

**Trả lời mẫu:**

MySQL và PostgreSQL phù hợp với dữ liệu quan hệ, ràng buộc và transaction như đơn hàng, tồn kho, thanh toán. Em chọn giữa chúng dựa trên nhu cầu query, tính năng cần dùng và khả năng vận hành của nhóm.

MongoDB phù hợp khi mô hình document khớp cách đọc/ghi, chẳng hạn một cụm dữ liệu thường được truy cập cùng nhau. Em cân nhắc embed hay reference theo kích thước, tần suất cập nhật và quan hệ dữ liệu. MongoDB vẫn cần thiết kế schema và có transaction; em không chọn chỉ vì cho rằng nó không có schema hoặc luôn nhanh hơn SQL.

### 17. Index giúp query nhanh bằng cách nào? Có đánh đổi gì?

**Trả lời mẫu:**

Index tạo cấu trúc truy cập giúp database tìm các bản ghi phù hợp mà có thể không phải quét toàn bảng. Đổi lại, index chiếm dung lượng và làm insert/update/delete tốn thêm công cập nhật.

Với query theo `customer_id`, `status`, rồi sắp theo `created_at`, em cân nhắc composite index dựa trên điều kiện thực tế và độ chọn lọc. Em dùng execution plan để kiểm chứng. Không phải cứ có index là optimizer sẽ dùng, đặc biệt khi query trả phần lớn bảng hoặc biểu thức không phù hợp index.

### 18. Transaction và ACID là gì?

**Trả lời mẫu:**

Transaction gom nhiều thao tác thành một đơn vị commit hoặc rollback. Atomicity là tính toàn vẹn của đơn vị đó; consistency là giữ các invariant được định nghĩa và thực thi đúng; isolation điều chỉnh cách các giao dịch đồng thời quan sát hoặc ảnh hưởng nhau; durability là dữ liệu đã commit được duy trì theo bảo đảm của hệ thống.

Ví dụ tạo order, order item và trừ tồn kho nên cùng transaction nếu ở chung database. Transaction không tự sửa logic nghiệp vụ sai và không tự rollback được một khoản tiền đã trừ ở hệ thống bên ngoài.

### 19. Hai request cùng mua sản phẩm cuối cùng, em xử lý thế nào?

**Trả lời mẫu:**

Em không chỉ đọc số lượng rồi trừ ở application vì hai request có thể cùng thấy còn hàng. Một cách là update có điều kiện trong transaction:

```sql
UPDATE products
SET stock = stock - :quantity
WHERE id = :productId AND stock >= :quantity;
```

Đây là cú pháp minh họa với bind parameter, không phải nối chuỗi SQL. Em validate `quantity` là số nguyên dương, kiểm tra số dòng bị ảnh hưởng: bằng 0 nghĩa là không tồn tại hoặc không đủ hàng. Nếu tạo đơn thất bại, transaction rollback cả việc trừ kho.

Khi có nhiều sản phẩm, em xử lý theo thứ tự nhất quán để giảm deadlock và có chiến lược retry giới hạn cho lỗi phù hợp. Nếu khóa hàng để đọc rồi cập nhật, em giữ transaction ngắn và tránh gọi API ngoài khi đang giữ khóa.

### 20. Isolation level và deadlock ảnh hưởng gì đến ứng dụng?

**Trả lời mẫu:**

Isolation level quyết định các bất thường đồng thời có thể xảy ra; hành vi cụ thể còn tùy database. Mức mạnh hơn có thể giảm một số bất thường nhưng tăng chờ hoặc buộc retry. Em chọn theo invariant cần bảo vệ, kết hợp constraint, conditional update hoặc locking phù hợp.

Deadlock xảy ra khi các giao dịch chờ tài nguyên của nhau theo vòng. Em giảm nguy cơ bằng thứ tự truy cập nhất quán, transaction ngắn và index phù hợp. Nếu database hủy một giao dịch do deadlock, em retry toàn bộ giao dịch khi thao tác đủ an toàn và có giới hạn.

### 21. N+1 query là gì? Em xử lý ra sao?

**Trả lời mẫu:**

Ví dụ lấy 50 order bằng một query, rồi mỗi order lại query customer riêng, tổng thành 51 query. Em có thể dùng JOIN, batch query theo danh sách ID hoặc batching ở tầng truy cập dữ liệu.

Em kiểm tra số query và dữ liệu trả về. JOIN nhiều quan hệ một-nhiều có thể làm phình kết quả và gây sai phân trang, nên không phải lúc nào eager load toàn bộ cũng tốt. Em chỉ lấy các trường cần thiết và tách query khi phù hợp.

### 22. Offset pagination và cursor pagination khác nhau thế nào?

**Trả lời mẫu:**

Offset đơn giản và thuận tiện khi nhảy đến trang cụ thể, nhưng offset lớn có thể tốn công bỏ qua nhiều dòng; dữ liệu thay đổi giữa các lần đọc cũng có thể khiến kết quả bị trùng hoặc thiếu.

Cursor dùng vị trí cuối đã đọc theo thứ tự ổn định, ví dụ `(created_at, id)`, để lấy trang tiếp theo với index phù hợp. Cách này hợp danh sách lớn hoặc cuộn liên tục, nhưng khó nhảy trực tiếp đến một trang bất kỳ. Cursor không tự tạo snapshot nhất quán cho toàn bộ phiên đọc.

### 23. Redis thường dùng làm gì? Cache-aside hoạt động thế nào?

**Trả lời mẫu:**

Redis có thể dùng cho cache, rate limit, session hoặc các nhu cầu dữ liệu tạm tùy thiết kế. Với cache-aside, ứng dụng đọc cache trước; nếu miss thì đọc database, đưa kết quả vào cache với TTL rồi trả về.

Em thiết kế key chứa đúng phạm vi, ví dụ tenant và bộ lọc. Khi Redis lỗi, có thể fallback về database nếu tải cho phép, kèm giới hạn để tránh làm database quá tải. Với dữ liệu quan trọng, em xác định rõ nguồn dữ liệu chính và mức độ cũ được chấp nhận.

### 24. Database đã cập nhật nhưng cache vẫn cũ thì làm gì?

**Trả lời mẫu:**

Sau khi transaction commit, em thường xóa key liên quan để lần đọc tiếp theo nạp lại. TTL giới hạn thời gian dữ liệu cũ tồn tại, nhưng không phải bảo đảm đồng bộ ngay lập tức.

Vẫn có race: request đọc có thể lấy dữ liệu cũ trước commit rồi ghi vào cache sau khi key vừa bị xóa. Với nghiệp vụ nhạy cảm, em dùng versioning, cơ chế cập nhật qua sự kiện có retry, hoặc đọc trực tiếp nguồn chính ở điểm cần tính đúng đắn. Việc quyết định cho phép mua hàng phải dựa vào cập nhật kho có kiểm soát ở database.

### 25. Cache stampede là gì?

**Trả lời mẫu:**

Khi key phổ biến hết hạn, nhiều request cùng miss và đổ vào database. Em có thể gộp request cùng key, thêm TTL jitter, refresh trước hạn hoặc phục vụ dữ liệu cũ trong khoảng cho phép khi làm mới nền.

Với nhiều instance, gộp request trong một process chưa đủ cho toàn hệ thống. Nếu dùng distributed lock, em phải xét TTL, quyền giải phóng khóa và tình huống process treo; không xem khóa là bảo đảm tuyệt đối cho mọi nghiệp vụ.

### 26. Elasticsearch khác database quan hệ ở điểm nào?

**Trả lời mẫu:**

Elasticsearch phù hợp cho full-text search, relevance và các kiểu truy vấn tìm kiếm. Trong hệ thống đơn hàng, em thường giữ database quan hệ làm nguồn chính, còn Elasticsearch là bản dữ liệu phục vụ tìm kiếm.

Khi cập nhật database, sự kiện hoặc CDC đồng bộ sang search index. Có độ trễ trong pipeline và khả năng hiển thị khi search nên kết quả có thể chưa mới ngay. Em cần cơ chế retry, xử lý sự kiện trùng, chống ghi đè bởi sự kiện cũ và khả năng reindex.

### 27. text và keyword trong Elasticsearch khác nhau thế nào?

**Trả lời mẫu:**

`text` thường được phân tích thành token để phục vụ tìm kiếm toàn văn; kết quả phụ thuộc analyzer. `keyword` phù hợp cho giá trị cần đối sánh chính xác, lọc, sort hoặc aggregation như mã sản phẩm và trạng thái.

Một tên sản phẩm có thể có cả trường text và subfield keyword. Em thiết kế mapping theo cách tìm kiếm thực tế, kiểm thử với dữ liệu tiếng Việt và tránh để số trường động tăng không kiểm soát.

## 5. Kiến trúc và thiết kế hệ thống

### 28. MVC là gì? Có giống kiến trúc phân lớp không?

**Trả lời mẫu:**

MVC chia trách nhiệm thành Model, View và Controller. Trong backend API, phần trình bày có thể là response serialization thay vì HTML. Kiến trúc phân lớp thường nói đến các lớp như controller, application/service và truy cập dữ liệu; hai cách phân chia có liên quan nhưng không đồng nghĩa hoàn toàn.

Điểm em áp dụng là tránh dồn nghiệp vụ và query vào controller, giữ ranh giới rõ để có thể kiểm thử và thay đổi từng phần.

### 29. Khi nào chọn monolith, khi nào chọn microservices?

**Trả lời mẫu:**

Em ưu tiên monolith có module rõ khi nhóm nhỏ, nghiệp vụ còn thay đổi và nhu cầu triển khai độc lập chưa lớn. Microservices hữu ích khi ranh giới nghiệp vụ rõ, các phần cần scale hoặc phát hành độc lập và nhóm có năng lực vận hành.

Chi phí gồm giao tiếp mạng, dữ liệu phân tán, quan sát hệ thống, version API và xử lý lỗi từng phần. Em chỉ tách khi có lý do cụ thể, chẳng hạn một module có tải hoặc vòng đời phát hành khác biệt.

### 30. Em đã áp dụng design pattern nào?

**Trả lời mẫu:**

Một ví dụ có thể là Strategy cho nhiều cách tính phí hoặc nhiều payment provider. Service gọi một interface chung, còn từng implementation xử lý quy tắc riêng. Khi thêm phương thức mới, phần orchestration ít bị ảnh hưởng.

Repository có ích khi cần gom logic truy cập dữ liệu hoặc tách khỏi nghiệp vụ. Em dùng pattern để giải quyết điểm thay đổi cụ thể, không cố bọc mọi thao tác CRUD thành nhiều lớp. Khi phỏng vấn, em sẽ minh họa bằng đoạn code hoặc tình huống mình thật sự đã làm.

### 31. Hai service nên gọi HTTP hay dùng message queue?

**Trả lời mẫu:**

HTTP phù hợp khi bên gọi cần kết quả ngay và chấp nhận phụ thuộc tính sẵn sàng của bên nhận. Queue phù hợp với xử lý nền, hấp thụ tải và tách thời điểm xử lý, nhưng thêm độ trễ và độ phức tạp theo dõi.

Với HTTP, em đặt timeout và retry có chọn lọc. Với queue, em xác định khi nào ack, cách retry, dead-letter và tính idempotent của consumer. Queue không tự bảo đảm side effect chỉ xảy ra đúng một lần.

### 32. Idempotency là gì? Áp dụng cho tạo đơn hoặc thanh toán thế nào?

**Trả lời mẫu:**

Idempotency giúp cùng một yêu cầu được gửi lại không tạo thêm tác động ngoài mong muốn. Em gắn idempotency key với người dùng hoặc tenant, loại thao tác và dấu vết payload. Database có unique constraint để chống hai request cùng tạo bản ghi.

Nếu key đã xử lý, em trả kết quả cũ; nếu đang xử lý, trả trạng thái theo hợp đồng API; nếu cùng key nhưng payload khác, từ chối. Với payment provider, em tận dụng idempotency của họ và có đối soát khi kết quả chưa rõ. Chỉ kiểm tra một key Redis trước rồi tạo đơn vẫn có thể bị race.

### 33. Làm sao vừa lưu database vừa gửi message mà không mất sự kiện?

**Trả lời mẫu:**

Nếu commit database rồi publish message trực tiếp, process có thể chết giữa hai bước. Em có thể dùng transactional outbox: ghi nghiệp vụ và bản ghi outbox trong cùng transaction; một relay đọc outbox rồi publish.

Relay có thể publish trùng nếu chết sau khi gửi nhưng trước khi đánh dấu hoàn thành, nên consumer cần deduplicate hoặc xử lý idempotent. Em theo dõi backlog, retry và cảnh báo, đồng thời cân nhắc thứ tự sự kiện theo từng aggregate.

### 34. Em thiết kế luồng tạo đơn, giữ hàng và thanh toán thế nào?

**Trả lời mẫu:**

Em định nghĩa trạng thái rõ như `PENDING_PAYMENT`, `PAID`, `CANCELLED`, `EXPIRED`. Tạo đơn và giữ hàng trong transaction nội bộ. Sau đó gọi thanh toán ngoài transaction, dùng idempotency key, rồi nhận webhook đã xác minh để chuyển trạng thái bằng cập nhật có điều kiện.

Nếu hết hạn, hệ thống giải phóng hàng đúng một lần. Nếu thanh toán thành công đến sau khi đơn hết hạn, cần chính sách rõ như xác minh lại hoặc hoàn tiền. Khi tách service, em dùng workflow hoặc saga với hành động bù, vì rollback SQL không bao phủ được hệ thống bên ngoài.

## 6. Bảo mật, kiểm thử và Git

### 35. Những lỗi bảo mật nào em ưu tiên kiểm tra ở API?

**Trả lời mẫu:**

Em kiểm tra phân quyền theo tài nguyên/tenant, validation đầu vào, query có bind parameter, giới hạn kích thước dữ liệu và rate limit ở endpoint nhạy cảm. Token phải được xác minh đúng chữ ký và các claim liên quan. Secret nằm ở cấu hình được quản lý, không đưa vào source hoặc log.

Nếu server nhận URL để tải tài liệu, em còn xét SSRF; nếu dùng cookie để xác thực thì xét CSRF và cấu hình cookie. Em gắn từng biện pháp với đường tấn công thực tế thay vì chỉ liệt kê tên công cụ.

### 36. Unit test, integration test và end-to-end test khác gì nhau?

**Trả lời mẫu:**

Unit test kiểm tra một đơn vị logic với dependency được kiểm soát. Integration test kiểm tra sự phối hợp thật, như service với database và transaction. End-to-end test đi qua luồng HTTP cùng các thành phần quan trọng.

Với tạo đơn, em unit test tính giá, integration test rollback và chống âm kho bằng database thật phù hợp, rồi end-to-end test xác thực, phân quyền và response. Em ưu tiên invariant và nhánh lỗi hơn việc chỉ đạt một con số coverage.

### 37. Test nào giúp chứng minh chống bán vượt tồn kho?

**Trả lời mẫu:**

Em tạo sản phẩm có tồn kho 1, gửi hai yêu cầu mua đồng thời qua các connection độc lập và kiểm tra chỉ một yêu cầu thành công. Sau đó kiểm tra stock bằng 0 và chỉ có một đơn hợp lệ. Test cần chạy với database thật cùng cơ chế transaction tương ứng, vì mock thường không mô phỏng được locking.

Em thêm ca tạo order thất bại sau khi đã trừ kho để xác nhận rollback, và ca request trùng để xác nhận idempotency. Một lần chạy qua chưa chứng minh hết mọi race nên em kết hợp test với phân tích cơ chế bảo vệ ở database.

### 38. Khi gặp lỗi production, em xử lý thế nào?

**Trả lời mẫu:**

Em xác định ảnh hưởng và ưu tiên khôi phục bằng rollback, tắt tính năng hoặc biện pháp giảm tải phù hợp. Em thông báo tình trạng theo quy trình nhóm, giữ log cần thiết và tránh sửa dữ liệu bằng phỏng đoán.

Sau khi ổn định, em tìm nguyên nhân gốc, thêm kiểm tra tái hiện và triển khai bản sửa có giám sát. Nếu có dữ liệu sai, em chuẩn bị phương án sửa có thể kiểm chứng. Khi kể kinh nghiệm, em nêu rõ thời điểm phát hiện, cách khoanh vùng, kết quả và thay đổi để phòng ngừa.

### 39. Gitflow hoạt động thế nào?

**Trả lời mẫu:**

Trong Gitflow điển hình, `main` giữ lịch sử phát hành, `develop` tích hợp công việc, feature branch tách từ develop. Release branch dùng để ổn định bản phát hành rồi đồng bộ về các nhánh liên quan; hotfix thường tách từ main và phải được đưa lại vào luồng phát triển.

Em tuân theo quy ước repository thực tế, giữ PR nhỏ và chạy CI trước merge. Gitflow có chi phí quản lý nhánh; với nhóm phát hành liên tục, quy trình đơn giản hơn có thể phù hợp hơn.

### 40. Merge, rebase, revert và cherry-pick khác nhau thế nào?

**Trả lời mẫu:**

Merge tích hợp lịch sử các nhánh, có thể tạo merge commit. Rebase phát lại commit lên nền mới, thay đổi commit ID nên cần thận trọng với lịch sử đã chia sẻ. Revert tạo commit đảo tác động của commit trước, phù hợp khi cần giữ lịch sử trên nhánh chung. Cherry-pick áp dụng thay đổi của một commit được chọn lên nhánh hiện tại.

Khi resolve conflict, em đọc ý nghĩa thay đổi ở cả hai phía và chạy lại phần kiểm tra liên quan; không chọn toàn bộ một phía chỉ để hết dấu conflict.

## 7. AI agent, Claude Code và review code

### 41. Em dùng AI trong quy trình lập trình hằng ngày thế nào?

**Trả lời mẫu — chỉ dùng nếu đúng trải nghiệm:**

Em dùng `[công cụ]` để đọc luồng code, đề xuất cách sửa, thực hiện thay đổi nhỏ và gợi ý test. Trước khi giao việc, em làm rõ yêu cầu và tiêu chí hoàn thành. Sau đó em đọc kế hoạch và diff, kiểm tra các giả định, chạy kiểm thử và tự xác nhận nghiệp vụ.

Một ví dụ cụ thể là `[task thật]`: AI hỗ trợ `[phần việc]`, nhưng em phát hiện `[vấn đề]` qua `[cách kiểm tra]` và sửa bằng `[giải pháp]`. Em chỉ đưa số liệu tiết kiệm thời gian nếu có cách đo đáng tin cậy.

### 42. Claude Code dạng agent khác gì hỏi chatbot sinh một đoạn code?

**Trả lời mẫu:**

Ở luồng agent, công cụ có thể làm việc qua nhiều bước với ngữ cảnh repository và các công cụ được cấp quyền, chẳng hạn đọc file, sửa code và chạy kiểm tra. Vì vậy em phải xác định phạm vi, quyền truy cập và điều kiện dừng rõ hơn.

Em bắt đầu bằng yêu cầu đọc luồng hiện tại, nêu kế hoạch rồi thực hiện một thay đổi đủ nhỏ để review. Em kiểm tra các lệnh đã chạy và kết quả thực tế, không coi lời báo hoàn thành của agent là bằng chứng duy nhất.

### 43. Em viết prompt giao task thế nào để code đúng và dễ bảo trì?

**Trả lời mẫu:**

Em cung cấp mục tiêu, ngữ cảnh file/module, ràng buộc nghiệp vụ, phạm vi thay đổi và tiêu chí nghiệm thu. Em chỉ rõ những điểm chưa chắc để agent kiểm tra trước khi code. Với task lớn, em chia theo các bước có thể review độc lập.

**Prompt minh họa:**

```text
Mục tiêu: thêm chức năng hủy đơn chưa thanh toán.
Ngữ cảnh: đọc Orders module, trạng thái đơn và cơ chế tồn kho hiện tại.
Yêu cầu:
- Người dùng chỉ hủy được đơn của mình trong đúng tenant.
- Chỉ cho phép trạng thái PENDING_PAYMENT chuyển sang CANCELLED.
- Giải phóng tồn kho đúng một lần, kể cả hai request đồng thời.
- Dùng transaction và quy ước lỗi hiện có.
Phạm vi: không đổi public API khác, không thêm dependency nếu chưa có lý do.
Trước khi sửa: chỉ ra file liên quan, giả định và phương án bảo vệ concurrency.
Nghiệm thu: test sai quyền, sai trạng thái, hủy trùng và rollback.
Kết quả: nêu file đã đổi, kiểm tra đã chạy và vấn đề còn lại.
```

### 44. File CLAUDE.md nên chứa gì?

**Trả lời mẫu:**

Em dùng `CLAUDE.md` làm hướng dẫn dự án cho Claude Code: cấu trúc module, lệnh build/test, quy ước code, các invariant quan trọng và cách kiểm chứng thay đổi. Nội dung cần ngắn, cụ thể và khớp repository. Chi tiết dài có thể tách thành tài liệu liên quan.

File hướng dẫn hỗ trợ hành vi của agent, không thay thế kiểm soát quyền truy cập, CI hay review. Em không đưa secrets hoặc dữ liệu khách hàng vào file này.

**Nội dung minh họa, cần thay lệnh theo dự án thật:**

```markdown
# Hướng dẫn dự án
- Backend dùng NestJS và TypeScript; code theo module nghiệp vụ.
- Controller không chứa logic tồn kho hoặc thanh toán.
- Query phải ràng buộc tenant theo ngữ cảnh xác thực của server.
- Thay đổi tồn kho phải bảo vệ trước request đồng thời.
- Không đọc hoặc đưa nội dung .env vào prompt, log hay báo cáo.
- Trước khi sửa, đọc package.json để xác định lệnh build/test.
- Bổ sung kiểm tra phù hợp; báo rõ kiểm tra chưa chạy được.
```

Nguồn: [Claude Code: Memory và CLAUDE.md](https://code.claude.com/docs/en/memory).

### 45. Em chia nhỏ task lớn cho AI như thế nào?

**Trả lời mẫu:**

Với tính năng hủy đơn, em chia thành: khảo sát luồng hiện tại; xác nhận trạng thái và invariant; thay đổi nghiệp vụ/transaction; mở API và phân quyền; thêm test; review diff và tài liệu. Mỗi bước có đầu ra rõ để kiểm tra trước khi tiếp tục.

Em cung cấp đúng file và hợp đồng liên quan, giữ các quyết định đã chốt trong tài liệu ngắn. Nếu agent bắt đầu đổi ngoài phạm vi hoặc lặp lại cách sửa thất bại, em dừng để làm rõ nguyên nhân thay vì tiếp tục tăng prompt mơ hồ.

### 46. Checklist review code AI trước khi merge gồm gì?

**Trả lời mẫu:**

Em review theo năm nhóm:

1. **Nghiệp vụ:** đúng yêu cầu, invariant, trạng thái và các tình huống cạnh tranh.
2. **Bảo mật:** quyền sở hữu/tenant, validation, injection, secrets và dữ liệu log.
3. **Dữ liệu và hiệu năng:** transaction, query, index, N+1, giới hạn tải và timeout.
4. **Bảo trì:** phù hợp cấu trúc hiện có, không thêm dependency hay abstraction thừa.
5. **Kiểm chứng:** test đi qua đúng nhánh rủi ro, build/type check và kiểm tra tích hợp phù hợp.

Em đọc toàn bộ diff, kể cả lockfile, migration và cấu hình. Nếu chưa giải thích được đoạn code hoặc tác động của nó, em chưa coi phần đó sẵn sàng merge.

### 47. AI viết cả code và test, vì sao test xanh vẫn có thể sai?

**Trả lời mẫu:**

Implementation và test có thể cùng dựa trên một giả định sai. Ví dụ AI viết API lấy đơn theo ID rồi viết test chỉ kiểm tra user hợp lệ, bỏ qua việc user khác truy cập cùng ID.

Em suy ra test từ yêu cầu và invariant độc lập với implementation, thêm negative case, boundary case và kiểm tra bằng tích hợp thật ở điểm cần thiết. Với bug đã biết, em xác nhận test tái hiện lỗi trước khi sửa để tránh test luôn xanh nhưng không bảo vệ gì.

### 48. Em bảo vệ source code, secrets và dữ liệu khách hàng khi dùng AI thế nào?

**Trả lời mẫu:**

Em chỉ dùng công cụ và cấu hình được tổ chức cho phép, kiểm tra loại dữ liệu nào được gửi ra ngoài và điều khoản áp dụng cho môi trường đang dùng. Em dùng dữ liệu giả hoặc đã khử nhận dạng, không đưa key, mật khẩu, token hay log chứa thông tin khách hàng vào prompt.

Em giới hạn quyền filesystem, network và công cụ theo nhu cầu; không cấp quyền production chỉ để sửa code. Quy tắc trong prompt hoặc file ignore không phải ranh giới bảo mật duy nhất. Nếu secret bị lộ, em xử lý theo quy trình incident, thu hồi/rotate key và kiểm tra dấu vết sử dụng; chỉ xóa đoạn chat là chưa đủ.

### 49. Nếu AI tự ý sửa nhiều file hoặc thêm dependency thì em làm gì?

**Trả lời mẫu:**

Em xem diff, xác định thay đổi nào thực sự cần cho task và yêu cầu giải thích những phần vượt phạm vi. Em giữ lại phần hợp lý, hoàn tác phần không cần sau khi kiểm tra không đè mất thay đổi của người khác, rồi chạy kiểm tra liên quan.

Với dependency mới, em xem nhu cầu thực tế, package nguồn, khả năng bảo trì, giấy phép và tác động bảo mật. Nếu giải pháp hiện có đáp ứng được, em tránh thêm thư viện chỉ vì agent đề xuất.

## 8. LLM API, RAG và MCP

### 50. Tích hợp LLM API vào backend cần quan tâm gì?

**Trả lời mẫu:**

Em gọi API từ backend để giữ key phía server, đặt timeout, giới hạn input/output, quota và theo dõi latency, lỗi, chi phí. Retry phải có giới hạn và không làm lặp side effect của công cụ hoặc nghiệp vụ liên quan.

Nếu cần output có cấu trúc, em yêu cầu định dạng phù hợp rồi validate phía ứng dụng. Em coi output là dữ liệu chưa tin cậy; không dùng trực tiếp để tạo SQL tùy ý hoặc quyết định quyền truy cập. Log được lọc dữ liệu nhạy cảm và tính năng có fallback khi model lỗi hoặc quá tải.

### 51. RAG là gì? Một luồng RAG cơ bản gồm những bước nào?

**Trả lời mẫu:**

RAG bổ sung tài liệu liên quan vào ngữ cảnh trước khi model trả lời. Luồng chuẩn bị gồm nạp tài liệu, làm sạch, chia đoạn, gắn metadata và lập chỉ mục. Khi hỏi, hệ thống xác thực người dùng, truy xuất các đoạn được phép xem, có thể rerank, rồi đưa ngữ cảnh vào prompt để tạo câu trả lời kèm nguồn.

Em đánh giá cả chất lượng truy xuất và câu trả lời bằng bộ câu hỏi có đáp án hoặc nguồn tham chiếu. RAG không bảo đảm hết bịa; cần xử lý trường hợp thiếu thông tin, tài liệu cũ và câu trả lời không được nguồn hỗ trợ.

### 52. Làm sao tránh RAG lộ tài liệu của tenant khác?

**Trả lời mẫu:**

Tenant và quyền truy cập phải lấy từ ngữ cảnh xác thực của server. Metadata truy xuất cần chứa phạm vi quyền, và hệ thống áp dụng bộ lọc bắt buộc trước khi tài liệu được đưa đến model. Khi cần cách ly mạnh hơn, em cân nhắc index hoặc kho dữ liệu riêng.

Em kiểm tra quyền ở bước truy xuất lẫn lúc trả nguồn/tải tài liệu, đồng thời đưa tenant vào cache key. Test có trường hợp hai tenant hỏi cùng một câu. Không thể chỉ nhắc model “đừng tiết lộ dữ liệu tenant khác” rồi coi là đã phân quyền.

### 53. MCP là gì? Khác gì REST API?

**Trả lời mẫu:**

MCP là giao thức chuẩn hóa cách ứng dụng AI kết nối với các khả năng như công cụ và nguồn ngữ cảnh do server cung cấp. Một MCP server có thể bọc các API hoặc hệ thống hiện có để client AI khám phá và gọi theo giao thức chung.

REST API là cách thiết kế giao tiếp HTTP phổ biến cho nhiều loại ứng dụng. MCP không tự thay thế mọi REST API và không tự cấp quyền truy cập. Em vẫn phải thực thi xác thực, phân quyền, validation và audit ở phía hệ thống cung cấp công cụ.

Nguồn: [Giới thiệu Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro).

### 54. Prompt injection là gì? Có nguy hiểm khi dùng RAG hoặc MCP không?

**Trả lời mẫu:**

Prompt injection là nội dung không đáng tin cố tác động để model làm lệch nhiệm vụ hoặc sử dụng công cụ sai mục đích. Nó có thể nằm trong tài liệu RAG, trang web hoặc kết quả công cụ, không chỉ trong câu hỏi người dùng.

Em phân biệt nội dung tài liệu với chỉ dẫn hệ thống, giới hạn công cụ và dữ liệu theo quyền, validate tham số và kiểm soát thao tác có side effect. Model không được tự quyết định quyền truy cập. Lọc chuỗi đáng ngờ có thể hỗ trợ, nhưng không thay được ranh giới bảo mật ở application.

## 9. Tình huống tổng hợp và hỏi ngược

### 55. Webhook thanh toán bị gửi lại hoặc đến sai thứ tự thì sao?

**Trả lời mẫu:**

Em xác minh chữ ký và các thuộc tính theo hợp đồng của provider, đối chiếu merchant, đơn hàng, số tiền và tiền tệ. Em lưu event ID với unique constraint và chuyển trạng thái bằng điều kiện hợp lệ để xử lý lặp an toàn.

Webhook cũ không được ghi đè trạng thái mới một cách mù quáng. Khi trạng thái mâu thuẫn hoặc chưa rõ, em đối soát với provider theo cơ chế hỗ trợ. Em chỉ báo nhận thành công sau khi đã lưu bền vững thông tin cần thiết hoặc hoàn thành xử lý theo thiết kế, để lỗi nội bộ không làm mất sự kiện.

### 56. Thiết kế API hủy đơn sao cho hai request không hoàn kho hai lần.

**Trả lời mẫu:**

Em mở transaction và chuyển trạng thái bằng conditional update: chỉ đơn thuộc đúng user/tenant, đang `PENDING_PAYMENT` mới được đổi sang `CANCELLED`. Chỉ transaction cập nhật thành công một dòng mới được hoàn kho, và hai thao tác nằm cùng transaction.

Request còn lại thấy trạng thái đã đổi sẽ trả kết quả theo hợp đồng API, không hoàn kho tiếp. Em xác định thêm cách phối hợp với webhook thanh toán vì hai luồng có thể tranh chấp trạng thái. Nếu tồn kho ở service khác, em phát sự kiện qua outbox và dùng idempotency ở bên nhận.

### 57. Em kể một lần dùng AI tạo ra code sai và cách em xử lý.

**Khung trả lời để điền kinh nghiệm thật:**

Trong task `[tên]`, em giao AI `[phạm vi]` và cung cấp `[ngữ cảnh]`. AI đề xuất `[thay đổi]`, nhưng em phát hiện `[lỗi cụ thể]` khi `[đọc diff/chạy test/kiểm tra nghiệp vụ]`. Nguyên nhân là `[giả định thiếu hoặc sai]`.

Em sửa bằng `[giải pháp]`, thêm kiểm tra cho `[tình huống]` và cập nhật hướng dẫn hoặc cách giao task. Bài học là `[điều cụ thể]`. Nếu chưa gặp tình huống thật, em có thể nói rõ đây là bài thực hành thay vì kể như kinh nghiệm công việc.

### 58. Khi deadline gấp, em có bỏ qua review code AI không?

**Trả lời mẫu:**

Em giảm phạm vi tính năng và tập trung kiểm tra vào rủi ro lớn nhất: phân quyền, tiền, tồn kho, dữ liệu và đường triển khai. Em giữ diff nhỏ, nhờ review phần nhạy cảm và dùng feature flag hoặc rollout giới hạn nếu hệ thống hỗ trợ.

Em báo rõ phần chưa hoàn thành và kiểm tra chưa chạy được. Code do AI tạo vẫn cần cùng tiêu chuẩn chịu trách nhiệm như code tự viết; deadline không làm mất rủi ro nghiệp vụ.

### 59. Em muốn hỏi gì về công việc và nhóm kỹ thuật?

**Câu hỏi gợi ý:**

- Trong ba tháng đầu, kết quả nào được xem là làm tốt ở vị trí này?
- Backend đang là monolith hay microservices, và vấn đề kỹ thuật lớn nhất hiện tại là gì?
- Nhóm phân chia trách nhiệm review, testing, triển khai và xử lý incident thế nào?
- Công ty cho phép những công cụ AI nào và những loại dữ liệu nào được đưa vào công cụ?
- Nhóm đã có hướng dẫn cho agent, quy trình review và cách đánh giá hiệu quả dùng AI chưa?
- Vai trò này chủ yếu dùng AI để lập trình hay còn trực tiếp phát triển tính năng LLM/RAG/MCP?

### 60. Trước buổi phỏng vấn, em cần chuẩn bị những gì?

**Trả lời/checklist thực hành:**

- Giới thiệu bản thân trong 90 giây và đào sâu hai dự án thật.
- Vẽ được luồng request, schema đơn hàng và một tình huống lỗi.
- Giải thích chống âm kho, idempotency, cache cũ và đồng bộ search.
- Có một ví dụ giao task AI, diff đã review và test kiểm chứng.
- Có câu chuyện về bug production, quyết định kỹ thuật và làm việc nhóm.
- Đánh dấu rõ công nghệ đã vận hành, đã thực hành và mới tìm hiểu.

## Lộ trình dùng tài liệu trong 14 ngày

| Ngày | Câu cần luyện | Bài thực hành |
|---|---|---|
| 1 | 01–03 | Viết giới thiệu và hai câu chuyện dự án |
| 2 | 04–08 | Promise, xử lý lỗi và giới hạn concurrency |
| 3 | 11–15 | Vẽ request lifecycle và luồng phân quyền |
| 4 | 16–22 | Schema, index và transaction tạo đơn |
| 5 | 23–27 | Cache-aside, invalidation và pipeline search |
| 6 | 28–30 | Giải thích kiến trúc và refactor một điểm thay đổi |
| 7 | 31–34 | Vẽ luồng thanh toán và các điểm lỗi |
| 8 | 09–10, 35–38 | Điều tra API chậm và test cạnh tranh dữ liệu |
| 9 | 41–45 | Giao một task nhỏ cho AI và soạn hướng dẫn dự án |
| 10 | 46–49 | Review diff và bổ sung test độc lập |
| 11 | 50–54 | Vẽ luồng LLM/RAG và ranh giới quyền truy cập |
| 12 | 39–40, 57–59 | Diễn tập Git và kể tình huống thực tế |
| 13 | 55–56 | Hoàn thiện bài thực hành đơn hàng |
| 14 | 60 và câu còn yếu | Mock interview 75–90 phút |

Mỗi buổi: ôn ý chính, làm một bài nhỏ rồi trả lời thành tiếng mà không nhìn tài liệu. Tự chấm mỗi câu theo ba mức: chưa giải thích được, giải thích được, giải thích được kèm ví dụ và đánh đổi.
