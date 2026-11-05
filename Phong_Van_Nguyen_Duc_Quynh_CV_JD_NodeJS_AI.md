# Phỏng vấn Nguyễn Đức Quỳnh — Backend Node.js & AI

Tài liệu cá nhân hóa từ CV `CV_Backend_Developer_Nguyen_Duc_Quynh (1).pdf` và JD được cung cấp. Bao gồm đối chiếu yêu cầu, 50 câu hỏi có hướng trả lời và câu hỏi đào sâu, lịch ôn 14 ngày và kịch bản mock interview.

**Cách dùng:** Luyện nói câu trả lời chính trong 60–90 giây; phần hỏi sâu dùng để kiểm tra hiểu biết. Các câu trả lời là khung đề xuất. Chỉ nhận đã triển khai những chi tiết đúng với công việc thực tế; thay phần `[cần bổ sung]` bằng trải nghiệm đã kiểm chứng. Các giải pháp được nêu dưới dạng “em sẽ” là phương án xử lý tình huống, không phải xác nhận hệ thống cũ đã làm như vậy.

## 1. Đối chiếu CV với JD và thứ tự ưu tiên

| Yêu cầu | Bằng chứng trong CV | Nội dung cần chuẩn bị |
|---|---|---|
| Đại học liên quan | PTIT, Information Technology, 2021–2025 | Xác nhận tình trạng tốt nghiệp; giới thiệu ngắn |
| 2–3 năm Node.js | Heligate 06/2024–07/2025; Vitech 07/2025–hiện tại | Giải thích phạm vi làm backend và đóng góp cá nhân |
| Express/NestJS | MIAPET, JIHO dùng Express; CRMAgent, ViShop dùng NestJS | Request lifecycle, DI, error handling, phân lớp |
| PostgreSQL | Xuất hiện xuyên suốt các dự án | Transaction, concurrency, index, partition, query plan |
| MySQL, MongoDB | Có trong Skills, chưa có dự án cụ thể làm bằng chứng | Chuẩn bị ví dụ thật hoặc nói rõ mức độ thực hành |
| Redis, Elasticsearch | Redis ở nhiều dự án; Elasticsearch ở JIHO | Cache cũ, idempotency, retry; mapping và search relevance |
| MVC, Microservices, pattern | Controller–Service–Repository; DDD, CQRS, Outbox | Vẽ kiến trúc thật; phân biệt module, service và deployment |
| Git, Gitflow | Có Git, CI/CD, pre-commit | Gitflow chưa được mô tả rõ; chuẩn bị quy trình thật đã dùng |
| AI coding, Claude Code | CV có dùng AI hỗ trợ test và chất lượng code | CV chưa nêu công cụ cụ thể; cần ví dụ dùng agent và review |
| LLM API, RAG | CRMAgent: OpenAI/Anthropic; ViDoc: RAG, pgvector, Gemini | Luồng dữ liệu, đánh giá, chi phí, failure mode và quyền truy cập |
| MCP | Chưa được ghi trong CV | Học nguyên lý, thực hành nhỏ nếu có thời gian; phân biệt với kinh nghiệm production |
| Bảo mật AI | Có nền multi-tenant/RBAC, chưa mô tả quy trình dữ liệu AI | Chính sách dữ liệu, secret, log, prompt injection, giới hạn công cụ |

**Ưu tiên cao nhất:** ViShop về tồn kho, tenant isolation và queue; CRMAgent về LLM và GraphQL; ViDoc về pipeline RAG. Đây là thứ tự ôn đề xuất từ nội dung CV và JD, không phải dự đoán chắc chắn về đề của công ty.

### Những phát biểu trong CV dễ dẫn đến câu hỏi kiểm chứng

- **High-throughput:** chuẩn bị tải thực sự đã đo, môi trường, p95/p99 và nút thắt. CV không có số liệu nên không tự thêm request/giây.
- **Prisma tenant isolation:** giải thích phạm vi bao phủ, raw query, nested relation, background job và đường bypass.
- **Redis idempotency:** phân biệt chặn request trùng tạm thời với bảo vệ side effect bền vững.
- **Outbox và leader election:** chuẩn bị giải thích process chết ở từng bước và sự kiện trùng.
- **Partitioning:** nêu bảng, partition key, query được lợi và chi phí vận hành thật.
- **DDD/CQRS:** nêu module và invariant cụ thể; không đồng nhất với microservices.
- **AI coding:** cần một task thật, prompt/ngữ cảnh, lỗi AI tạo ra và kiểm tra trước merge.

## 2. Giới thiệu và xác minh kinh nghiệm

### 01. Em hãy giới thiệu bản thân theo vị trí này.

**Trả lời gợi ý:**

Em là Nguyễn Đức Quỳnh, học ngành Công nghệ thông tin tại PTIT giai đoạn 2021–2025. Em làm backend từ tháng 6/2024, chủ yếu với Node.js, Express, NestJS và PostgreSQL. Ở Heligate em tham gia MIAPET và JIHO; tại Vitech em tham gia CRMAgent, ViShop và ViDoc.

Phần em muốn trao đổi sâu là xử lý dữ liệu và quyền truy cập trong hệ thống multi-tenant, luồng tồn kho và các tác vụ bất đồng bộ. Em cũng có kinh nghiệm tích hợp OpenAI/Anthropic ở CRMAgent và làm pipeline xử lý tài liệu cho RAG ở ViDoc. Với vị trí này, em muốn phát huy nền backend và trình bày rõ cách kiểm chứng code, kể cả khi dùng AI hỗ trợ.

**Hỏi sâu:** Phần nào em tự thiết kế? Phần nào làm theo thiết kế nhóm? Vì sao đang tìm cơ hội mới?

### 02. Em làm nhiều dự án như vậy, mức độ tham gia từng dự án ra sao?

**Trả lời gợi ý:**

Theo CV, MIAPET là 06–12/2024, JIHO là 12/2024–07/2025, CRMAgent là 08/2025–01/2026, ViShop từ 01/2026 và ViDoc từ 07/2026. Em sẽ làm rõ từng giai đoạn mình phụ trách module nào và khoảng thời gian có tham gia song song.

Với mỗi dự án, em phân biệt phần tự thực hiện, phần cùng nhóm quyết định và phần chỉ tích hợp. Riêng ViShop và ViDoc đang cùng xuất hiện là dự án hiện tại, em cần bổ sung cách phân bổ công việc thực tế, không tự suy diễn từ CV.

**Cần bổ sung:** Vai trò chính ở từng giai đoạn, người review, một PR hoặc quyết định tiêu biểu đã được phép chia sẻ.

### 03. Dự án nào chứng minh em phù hợp nhất với JD?

**Trả lời gợi ý:**

Em chọn ViShop để thể hiện backend Node.js: NestJS, PostgreSQL, Redis, RabbitMQ, kiểm soát truy cập tenant/shop và nghiệp vụ tồn kho. Em dùng CRMAgent để nói về tích hợp LLM vào luồng CRM và xử lý hội thoại. ViDoc thể hiện kinh nghiệm làm dữ liệu đầu vào cho RAG, chunking và kiểm thử chất lượng tài liệu.

Khi đi sâu, em sẽ chọn một luồng em trực tiếp làm thay vì liệt kê toàn bộ stack. Ví dụ ở ViShop là một lần điều chuyển kho hoặc đồng bộ đơn hàng, có điều kiện lỗi và cách kiểm chứng.

**Hỏi sâu:** Kết quả cụ thể là gì? Nếu bỏ công nghệ đó đi thì hệ thống mất khả năng nào?

## 3. ViShop — tồn kho, multi-tenant và xử lý bất đồng bộ

### 04. Em vẽ kiến trúc ViShop và giải thích DDD/CQRS được áp dụng ở đâu.

**Trả lời gợi ý:**

CV mô tả ViShop là ERP/WMS/OMS dùng NestJS, PostgreSQL, Redis, RabbitMQ và Nx. Em sẽ vẽ ranh giới theo nghiệp vụ như đơn hàng, tồn kho, tích hợp kênh và quyền truy cập, rồi đánh dấu đúng các process được triển khai thực tế.

DDD giúp nhóm phân chia mô hình theo nghiệp vụ và bảo vệ invariant. CQRS tách trách nhiệm thay đổi dữ liệu và đọc dữ liệu; không bắt buộc tách database hoặc triển khai thành microservices. Em cần đưa ví dụ command và query thật của module mình phụ trách.

**Hỏi sâu:** Aggregate nào bảo vệ invariant tồn kho? Dùng chung database không? Nx có quyết định kiến trúc runtime không?

### 05. Hai request cùng trừ tồn kho thì em chống âm kho thế nào?

**Trả lời gợi ý:**

CV có nêu conditional update, transaction và row-level locking. Với một dòng tồn kho, một phương án là cập nhật có điều kiện `available >= quantity`, kiểm tra số dòng cập nhật và lưu nghiệp vụ liên quan trong cùng transaction. Không chỉ đọc số lượng rồi trừ ở application.

Ví dụ SQL dưới đây là phương án minh họa, cần đối chiếu schema thật:

```sql
UPDATE inventory
SET available = available - $1
WHERE tenant_id = $2
  AND warehouse_id = $3
  AND sku_id = $4
  AND available >= $1;
```

Input phải là số nguyên dương; tổ hợp tenant/kho/SKU cần ràng buộc duy nhất phù hợp. Nếu không cập nhật dòng nào, transaction không được tiếp tục coi là giữ hàng thành công.

**Hỏi sâu:** Phân biệt on-hand, reserved và available thế nào? Nếu tạo order lỗi sau update? Hai request qua hai instance thì sao?

### 06. Deterministic lock ordering trong CV giải quyết gì?

**Trả lời gợi ý:**

Khi một giao dịch cần khóa nhiều dòng, em lấy khóa theo thứ tự thống nhất, ví dụ tổ hợp kho và SKU, thay vì theo thứ tự input. Điều này giảm tình huống giao dịch A giữ dòng 1 chờ dòng 2 trong khi B giữ dòng 2 chờ dòng 1.

Em giữ transaction ngắn và không gọi API ngoài trong lúc giữ khóa. Lock ordering không loại bỏ mọi deadlock; vẫn cần nhận diện lỗi phù hợp và retry toàn bộ transaction có giới hạn nếu nghiệp vụ cho phép.

**Hỏi sâu:** Hai yêu cầu chuyển kho ngược chiều thì thứ tự khóa là gì? Retry có thể tạo chứng từ trùng không?

Nguồn: [PostgreSQL — Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html).

### 07. Một luồng chuyển hàng giữa hai kho phải giữ invariant gì?

**Trả lời gợi ý:**

Em làm rõ chuyển ngay hay có giai đoạn đang vận chuyển. Với chuyển ngay trong cùng database, trừ kho nguồn, cộng kho đích và ghi chứng từ cần cùng transaction. Với quy trình xuất–vận chuyển–nhận, em cần trạng thái và số lượng tương ứng, không cộng ngay vào lượng có thể bán ở kho đích.

Mỗi bước phải kiểm tra tenant, quyền kho, số lượng hợp lệ và chống xử lý lặp. Em sẽ minh họa bằng workflow thật đã triển khai; không khẳng định mô hình tức thời hay ledger nếu hệ thống thực tế không dùng.

**Hỏi sâu:** Nhận thiếu hàng xử lý thế nào? Hủy sau khi đã xuất? Có sửa trực tiếp stock hay phải qua chứng từ?

### 08. Prisma Client Extensions đã bảo vệ tenant như thế nào? Có đủ tuyệt đối không?

**Trả lời gợi ý:**

CV nêu em dùng custom Prisma Client Extensions để hỗ trợ tenant isolation. Em sẽ giải thích tenant context lấy từ xác thực phía server, extension áp điều kiện ở những operation nào và policy khi thiếu context. Không nên lấy tenant tùy ý từ request body.

Em phải rà riêng raw query, nested operation, relation connect, transaction, aggregate và đường dùng client không được bọc. Không thể mặc định mọi truy cập đều được bảo vệ chỉ vì có extension. Ràng buộc dữ liệu và test tích hợp giúp kiểm chứng; RLS là một lớp có thể cân nhắc nếu phù hợp, chưa được CV xác nhận là đã dùng.

**Hỏi sâu:** Tenant A connect một record của tenant B thì sao? Job nền không có HTTP request lấy tenant ở đâu?

Nguồn: [Prisma — Query Extensions](https://docs.prisma.io/docs/orm/prisma-client/client-extensions/query). Cần đối chiếu phiên bản Prisma thực tế của dự án.

### 09. Tenant-level, shop-level và hierarchical RBAC phối hợp thế nào?

**Trả lời gợi ý:**

Em phân biệt tenant là ranh giới tổ chức, shop là phạm vi con và role/permission quyết định thao tác. Có quyền đọc đơn không đồng nghĩa được đọc đơn ở mọi shop. Backend cần kiểm tra cả hành động và tập tài nguyên được phép truy cập.

Với role phân cấp, em nêu quy tắc kế thừa thật, tránh quyền con vô tình rộng hơn quyền cha. Quyền và dữ liệu cache cần có phạm vi đúng, cùng cơ chế làm mới hoặc thu hồi phù hợp.

**Hỏi sâu:** User thuộc nhiều shop thì query list ra sao? Sau thu hồi quyền, token cũ có dùng tiếp được không?

### 10. Redis-based idempotency khi nhận webhook TikTok có giới hạn gì?

**Trả lời gợi ý:**

Redis giúp nhận diện nhanh event đã xử lý hoặc đang xử lý, nhưng cần xét TTL hết hạn, mất dữ liệu và process chết giữa lúc đánh dấu với lúc commit nghiệp vụ. Nếu đánh dấu trước rồi chết, event có thể bị bỏ qua; nếu đánh dấu sau, có cửa sổ xử lý trùng.

Với side effect quan trọng, em sẽ xem xét event ID hoặc khóa nghiệp vụ có unique constraint trong database, cập nhật trạng thái có điều kiện và transaction phù hợp. Khi nói về ViShop, em cần mô tả cơ chế thực tế trước, rồi nêu phần nào còn có thể tăng độ an toàn.

**Hỏi sâu:** Provider không có event ID thì chọn key gì? Hai event cùng order nhưng nội dung khác nhau? TTL bao lâu và vì sao?

### 11. Quản lý OAuth token của nhiều shop cần chú ý gì?

**Trả lời gợi ý:**

Em lưu token theo đúng tenant/shop và bảo vệ nơi lưu; không đưa token vào log. Khi token hết hạn, refresh cần được điều phối để nhiều worker không đồng thời ghi đè token mới bằng kết quả cũ. Nếu provider xoay refresh token, việc cập nhật cặp token càng cần nhất quán.

Em phân biệt lỗi token, quyền bị thu hồi và lỗi tạm thời; không retry vô hạn. Rate limit cần theo đúng phạm vi provider áp dụng, có backoff và quan sát trạng thái đồng bộ.

**Hỏi sâu:** Shop ngắt kết nối trong khi job đang chạy? Nếu token refresh thành công nhưng response bị mất?

### 12. CV ghi có Outbox: em giải thích được điểm chết giữa commit và publish không?

**Trả lời gợi ý:**

Nếu ghi nghiệp vụ rồi gửi message riêng, process có thể chết sau commit và làm thiếu message. Outbox ghi nghiệp vụ cùng bản ghi sự kiện trong một transaction. Relay đọc bản ghi đó để gửi ra broker.

Nếu relay gửi thành công rồi chết trước khi đánh dấu, sự kiện có thể được gửi lại. Vì vậy consumer phải xử lý trùng an toàn. Em cần nêu cụ thể nơi đã áp dụng, trạng thái outbox, retry và cách theo dõi backlog; CV nêu pattern ở phần kinh nghiệm Vitech, không xác định rõ module nên phải bổ sung đúng dự án.

**Hỏi sâu:** Nhiều relay claim cùng event thế nào? Có giữ thứ tự theo order không? Poison message được xử lý ra sao?

### 13. Distributed leader election trong CV dùng cho tác vụ gì?

**Trả lời gợi ý:**

Em cần bắt đầu bằng tác vụ thực tế đã dùng, chẳng hạn điều phối một công việc định kỳ; CV chưa ghi chi tiết này. Về nguyên lý, leader election hoặc lease giúp chọn một instance điều phối tại một thời điểm theo giả định của cơ chế đó.

Em sẽ giải thích TTL, renewal và chuyện leader cũ tạm dừng rồi chạy tiếp sau khi lease hết hạn. Lease đơn thuần chưa bảo vệ mọi side effect; tác vụ vẫn cần idempotency hoặc cơ chế fencing được hệ thống đích thực thi khi phù hợp.

**Hỏi sâu:** Redis bị partition thì sao? Process cũ có quyền hoàn thành việc đã bắt đầu không? Vì sao cần election thay vì queue?

### 14. Vì sao ViShop dùng RabbitMQ, còn CRMAgent/JIHO có BullMQ?

**Trả lời gợi ý:**

Em giải thích bằng yêu cầu thật: BullMQ phù hợp mô hình background job dựa trên Redis; RabbitMQ là message broker với cơ chế routing và giao tiếp giữa các thành phần. Cả hai đều đòi hỏi thiết kế retry, ack và xử lý trùng phù hợp.

Lý do lựa chọn ở từng dự án phải dựa trên workflow và năng lực vận hành, không chỉ nói công nghệ nào mạnh hơn. Em chuẩn bị một luồng job BullMQ và một luồng message RabbitMQ mình trực tiếp làm.

**Hỏi sâu:** Khi nào ack? Consumer commit xong rồi mất kết nối trước ack thì sao? Queue đầy hoặc broker lỗi có làm request thất bại không?

### 15. SSE ở ViShop giải quyết gì? Khi client reconnect thì sao?

**Trả lời gợi ý:**

SSE phù hợp luồng cập nhật từ server đến client như trạng thái đồng bộ đơn và tồn kho. Em kiểm tra xác thực, phạm vi tenant/shop và vòng đời kết nối; không broadcast dữ liệu nhạy cảm cho mọi client.

Khi reconnect, nếu hệ thống hỗ trợ event ID và replay thì có thể tiếp tục từ mốc đã nhận. Nếu không, client cần tải lại trạng thái chuẩn. SSE không tự tạo event log bền vững. Với nhiều instance, cần cơ chế phân phối sự kiện tới instance giữ kết nối và lưu ý proxy buffering/timeout.

**Hỏi sâu:** Vì sao không dùng Socket.IO? Token hết hạn giữa kết nối? Client chậm đọc thì xử lý sao?

### 16. Em đã dùng partitioning và composite index để tối ưu bảng nào?

**Trả lời gợi ý:**

CV nói có range partitioning và composite indexes nhưng không nêu bảng hoặc số liệu. Em phải bổ sung bảng thật, partition key, query phổ biến, retention và execution plan trước/sau.

Partition có ích khi query loại bỏ được phần dữ liệu không liên quan hoặc cần quản lý vòng đời dữ liệu theo khoảng. Nó không mặc định tăng tốc mọi query. Composite index cần khớp điều kiện và thứ tự truy cập, đổi lại tăng chi phí ghi và dung lượng. Em sẽ dùng số đo thực tế thay vì nói nhanh hơn một tỷ lệ không có bằng chứng.

**Hỏi sâu:** Quên tạo partition mới thì sao? Query không có partition key có được lợi không? Unique constraint trên bảng partition có ràng buộc gì?

### 17. Testcontainers và Vitest giúp kiểm chứng điều gì ở ViShop?

**Trả lời gợi ý:**

CV có Testcontainers/Vitest ở ViShop và integration test ở phần kinh nghiệm. Em chọn ví dụ cần database thật như transaction rollback, tenant isolation hoặc hai request cạnh tranh tồn kho. Container giúp tạo dependency phù hợp và lặp lại môi trường test, thay vì mock sai hành vi database.

Em cần nêu test mình thật sự đã viết, cách chuẩn bị và dọn dữ liệu. Pre-commit hỗ trợ phản hồi sớm, nhưng CI vẫn cần chạy các kiểm tra bắt buộc vì hook local có thể bị bỏ qua.

**Hỏi sâu:** Hai request dùng connection độc lập không? Test có thể pass vì vô tình chạy tuần tự? Container version có khớp production không?

## 4. CRMAgent — GraphQL, metadata và LLM trong sản phẩm

### 18. DataLoader giảm N+1 ở GraphQL như thế nào?

**Trả lời gợi ý:**

Nếu resolver lấy danh sách khách hàng rồi truy vấn riêng cho từng record liên quan, số query tăng theo số phần tử. DataLoader có thể gom các yêu cầu trong cửa sổ xử lý thành batch và cache trong phạm vi phù hợp.

Em thường tạo loader theo request và ngữ cảnh quyền; dùng cache toàn cục thiếu tenant có thể lộ dữ liệu. Batch function phải trả kết quả theo đúng thứ tự key đầu vào, có xử lý key không tồn tại. Em cần minh họa một resolver thật ở CRMAgent và số query đã quan sát.

**Hỏi sâu:** Dữ liệu đổi trong cùng request thì cache invalidation thế nào? Batch vẫn phải lọc quyền ở đâu?

### 19. Dynamic metadata engine lưu custom object/field thế nào?

**Trả lời gợi ý:**

CV xác nhận có custom object/field không cần đổi schema database, nhưng chưa nói dùng JSONB, EAV hay mô hình kết hợp. Em phải trình bày đúng thiết kế thật: metadata định nghĩa field, dữ liệu lưu thế nào, validate type và quyền ở đâu.

Nếu dùng JSONB, cần cân nhắc index và truy vấn field động. Nếu dùng EAV, cần xét số JOIN và kiểm soát kiểu. Em giải thích vì sao nhóm chọn phương án đó với yêu cầu lọc, báo cáo và thay đổi field thực tế.

**Hỏi sâu:** Đổi kiểu field khi đã có dữ liệu? Xóa field có mất dữ liệu không? Unique và required được thực thi ở đâu?

### 20. Luồng phân loại ý định mua hàng bằng OpenAI/Anthropic hoạt động thế nào?

**Trả lời gợi ý:**

CV mô tả phân loại purchase intent, gợi ý phản hồi và trích xuất lead từ hội thoại. Em chọn một use case để giải thích: lấy hội thoại được phép truy cập, chuẩn bị ngữ cảnh, gọi model, validate output rồi đưa kết quả vào workflow.

Các chi tiết cần bổ sung là model thật đã dùng, output schema, xử lý lỗi và cách người dùng kiểm chứng. Em không suy ra rằng kết quả AI được tự động gửi cho khách nếu CV chỉ ghi là gợi ý phản hồi.

**Hỏi sâu:** Model trả JSON sai thì sao? Không đủ dữ liệu có cho phép trả unknown không? Làm sao tránh nhầm người hoặc tenant?

### 21. Em đánh giá chất lượng tính năng AI của CRM thế nào?

**Trả lời gợi ý:**

Em sẽ lập bộ hội thoại đã được phép sử dụng và có nhãn hoặc kết quả mong đợi. Với classification, xem precision/recall theo nhóm, đặc biệt trường hợp false positive tốn chi phí. Với extraction, kiểm tra tính đúng của từng field; với gợi ý phản hồi, đánh giá sự chính xác, mức phù hợp và nội dung không được phép.

Em theo dõi phiên bản prompt/model, latency và chi phí để so sánh. Nếu dự án chưa có bộ đánh giá chính thức, em nói rõ cách kiểm tra đã làm và đề xuất cải thiện, không nhận đã đo các chỉ số chưa từng đo.

**Hỏi sâu:** Bộ test có đại diện cho tiếng Việt và hội thoại nhiễu không? Khi đổi model, làm sao phát hiện regression?

### 22. Em giảm chi phí, độ trễ và xử lý lỗi LLM ra sao?

**Trả lời gợi ý:**

Em giới hạn ngữ cảnh và output theo nhu cầu, chọn model phù hợp từng việc, đặt timeout và quota. Cache chỉ dùng khi dữ liệu và quyền cho phép, key cần chứa phạm vi tenant và phiên bản liên quan. Với việc không cần trả ngay, em cân nhắc xử lý nền.

Retry có giới hạn cho lỗi tạm thời; không retry vô hạn các lỗi input hoặc xác thực. Nếu response bị mất hoặc timeout, em tránh để việc gọi lại gây lặp side effect. Ghi nhận token/chi phí cần lọc thông tin nhạy cảm và không mặc định lưu toàn bộ hội thoại vào log.

**Hỏi sâu:** Provider lỗi thì tính năng fallback ra sao? Dùng hai provider có cần đánh giá lại chất lượng không?

### 23. Webhook Facebook/Zalo qua BullMQ có thể bị xử lý hai lần không?

**Trả lời gợi ý:**

Có thể. Nếu worker hoàn thành side effect nhưng chết trước khi xác nhận, job có thể được xử lý lại. CV nêu xử lý webhook bất đồng bộ bằng BullMQ, nên em cần trình bày cách xác định event, lưu trạng thái và bảo vệ phần ghi dữ liệu.

Job ID giúp deduplicate trong phạm vi tồn tại của job, không thay thế invariant bền vững. Em retry lỗi tạm thời với backoff, phân biệt lỗi vĩnh viễn và có chỗ quan sát job thất bại. Worker chạy CPU nặng còn có thể ảnh hưởng gia hạn lock.

**Hỏi sâu:** Xóa completed job có ảnh hưởng deduplication không? Hai event của cùng hội thoại đến sai thứ tự thì sao?

Nguồn: [BullMQ — Idempotent Jobs](https://docs.bullmq.io/patterns/idempotent-jobs), [Stalled Jobs](https://docs.bullmq.io/guide/workers/stalled-jobs).

## 5. ViDoc — RAG, tài liệu và kiểm chứng chất lượng

### 24. Em vẽ pipeline ViDoc từ PDF đến câu trả lời có nguồn.

**Trả lời gợi ý:**

Theo CV, pipeline có parsing layout, trích xuất hình/bảng, chuẩn hóa về Document IR, chunking giữ cấu trúc, embedding/vector retrieval bằng pgvector và tạo câu trả lời gắn nguồn. Stack là Python/FastAPI, PostgreSQL/pgvector và Gemini.

Phần trách nhiệm được ghi chi tiết nhất là ingestion, Document IR, normalization, chunking và validation. Khi mô tả retrieval và generation, em cần nói rõ mình trực tiếp làm phần nào, phần nào do thành viên khác thực hiện hoặc cùng tích hợp.

**Hỏi sâu:** Vì sao tách từng stage? Tài liệu lỗi ở một trang có làm fail cả pipeline không? Lưu trạng thái để chạy lại ở đâu?

### 25. Parser-agnostic Document IR có lợi gì? Một node cần chứa gì?

**Trả lời gợi ý:**

IR là mô hình trung gian để các stage sau không phụ thuộc trực tiếp output của một parser. Theo CV, ViDoc có cấu trúc phân cấp và shared contracts; em nên vẽ đúng schema thật của một trang, section, bảng hoặc hình.

Thông tin thường cần gồm ID ổn định, loại node, nội dung, quan hệ cha–con, thứ tự đọc và metadata nguồn. Với bbox và page reference, phải giữ quy ước rõ để citation chính xác. Khi đổi parser, em viết adapter và test contract thay vì sửa toàn bộ chunking.

**Hỏi sâu:** Parser mới thiếu bbox hoặc heading thì sao? Version IR thế nào? Phân biệt text gốc với mô tả do VLM sinh ra ra sao?

### 26. Adaptive chunking khác chia mỗi đoạn một số token cố định thế nào?

**Trả lời gợi ý:**

Chunk cố định đơn giản nhưng có thể cắt mất quan hệ heading–nội dung, định nghĩa–điều khoản hoặc header–dòng bảng. CV nêu chunking thích nghi cho tài liệu pháp lý, SOP và báo cáo tài chính, cùng topic-shift detection và sliding window.

Em cần giải thích một ví dụ thật: ranh giới cấu trúc nào được ưu tiên, khi nào split theo token, overlap ra sao và metadata nào được giữ. Ngưỡng semantic không phải con số đúng cho mọi tài liệu; phải so sánh trên dữ liệu và query đại diện.

**Hỏi sâu:** Điều khoản rất dài? Bảng trải nhiều trang? Overlap quá lớn ảnh hưởng retrieval và chi phí thế nào?

### 27. Structure-preserving table chunking được kiểm thử thế nào?

**Trả lời gợi ý:**

CV nói em có test table reconstruction, coordinate mapping và token limit. Em chọn một bảng thật hoặc dữ liệu giả tương đương để minh họa: header, ô gộp, dòng nhiều trang và cách gắn lại ngữ cảnh cho các đoạn.

Test nên kiểm tra dữ liệu không bị đảo cột, mất đơn vị hoặc gán sai nhãn hàng. Nếu một chunk chỉ có con số mà thiếu tên chỉ tiêu hoặc kỳ báo cáo thì retrieval có thể đúng từ khóa nhưng câu trả lời sai ý nghĩa. Em cần đưa ví dụ bug từng bắt được, không chỉ nói có nhiều test.

**Hỏi sâu:** Trích header lặp có làm trùng nội dung? Một ô vượt token limit thì xử lý sao?

### 28. Normalization và encoding repair có thể làm sai tài liệu không?

**Trả lời gợi ý:**

Có. Unicode normalization có thể giúp so sánh và tìm kiếm ổn định, nhưng thao tác sửa encoding hoặc ghép đoạn sai có thể thay đổi nội dung. Với pháp lý và tài chính, em giữ provenance và đối chiếu với bản gốc, tránh âm thầm sửa các trường hợp không chắc chắn.

Em dùng test có dấu tiếng Việt, ký tự tổ hợp, bảng, đơn vị và ký hiệu; kiểm tra cả dữ liệu đầu ra lẫn mapping về nguồn. Coverage cho biết nội dung đã được xử lý đến đâu, không tự chứng minh ý nghĩa được bảo toàn.

**Hỏi sâu:** Phát hiện mojibake khác OCR sai thế nào? Nếu normalization làm offset thay đổi thì citation dựa vào gì?

### 29. Chọn pgvector thay vì vector database riêng có đánh đổi gì?

**Trả lời gợi ý:**

pgvector giúp lưu embedding gần metadata và dữ liệu quan hệ trong PostgreSQL, phù hợp khi nhóm đã vận hành PostgreSQL. Đổi lại cần theo dõi tài nguyên, query plan và tải vector search cạnh tranh với nghiệp vụ khác.

Em phân biệt exact search với approximate search bằng HNSW/IVFFlat. Với bộ lọc tenant và index approximate, phải đo recall lẫn latency, không chỉ thấy nhanh là đủ. CV chưa ghi loại index đang dùng nên em cần xác nhận trước khi nói đã triển khai HNSW hay tinh chỉnh tham số.

**Hỏi sâu:** Đổi embedding model có cần re-embed không? Khác dimension thì lưu thế nào? Filter tenant ảnh hưởng số kết quả ra sao?

Nguồn: [pgvector — Tài liệu dự án](https://github.com/pgvector/pgvector).

### 30. Có citation rồi thì câu trả lời RAG đã đáng tin chưa?

**Trả lời gợi ý:**

Chưa. Citation có thể trỏ đến tài liệu thật nhưng không hỗ trợ phát biểu được tạo ra. Em kiểm tra riêng truy xuất có lấy đúng đoạn không, đoạn có đủ ngữ cảnh không và câu trả lời có bám bằng chứng không.

Em sẽ xây bộ câu hỏi có nguồn chuẩn, gồm câu không có đáp án, bảng số liệu, tài liệu mâu thuẫn và bản đã cũ. Khi không đủ bằng chứng, hệ thống cần thể hiện giới hạn thay vì tự đoán. Citation cũng phải kiểm tra quyền khi người dùng mở nguồn.

**Hỏi sâu:** Nguồn đúng nhưng số bị đọc nhầm? Tài liệu bị thu hồi quyền sau khi đã index? Xóa tài liệu có xóa chunk/cache không?

## 6. JIHO, MIAPET và nền tảng Node.js/database

### 31. Ở JIHO, vì sao dùng Elasticsearch cùng PostgreSQL?

**Trả lời gợi ý:**

PostgreSQL giữ dữ liệu nghiệp vụ và transaction; Elasticsearch phục vụ tìm kiếm theo cách phù hợp với nội dung. CV xác nhận dùng Elasticsearch ở JIHO nhưng chưa nói mapping, analyzer hoặc cách đồng bộ, nên em phải bổ sung cấu hình thật.

Em giải thích `text` cho full-text có phân tích và `keyword` cho đối sánh chính xác, sort/aggregation. Với dữ liệu tiếng Nhật, em cần nói rõ analyzer thực sự đã dùng và cách kiểm tra relevance, không tự nhận đã cấu hình một plugin chỉ vì sản phẩm phục vụ thị trường Nhật.

**Hỏi sâu:** DB cập nhật nhưng index còn cũ? Reindex không gián đoạn thế nào? Query số liệu quyền truy cập lấy từ đâu?

### 32. Đồng bộ E-Learning và E-Book bằng BullMQ có retry thì đã đủ chưa?

**Trả lời gợi ý:**

Retry giúp phục hồi lỗi tạm thời nhưng chưa giải quyết mất event trước khi enqueue, sự kiện trùng và sai thứ tự. Em cần nêu nguồn dữ liệu chính, khóa nghiệp vụ, trạng thái đồng bộ và cách đối soát khi một bên đã đổi còn bên kia chưa đổi.

Nếu cùng một user hoặc nội dung có nhiều cập nhật, consumer không nên để event cũ ghi đè bản mới. Em sẽ trình bày cơ chế thật đã dùng trước, rồi đề xuất versioning hoặc đối soát cho rủi ro chưa được bao phủ.

**Hỏi sâu:** Job hết retry thì ai biết? Người dùng có nhìn thấy trạng thái đồng bộ thất bại không?

### 33. Ở MIAPET, VNPay return URL và IPN nên được xử lý thế nào?

**Trả lời gợi ý:**

Luồng người dùng quay lại website phục vụ trải nghiệm hiển thị, còn cập nhật kết quả thanh toán cần căn cứ thông báo hoặc truy vấn đã được backend xác minh theo hợp đồng tích hợp. Không tin riêng trạng thái client gửi lên.

Em kiểm tra chữ ký và các trường liên quan như mã giao dịch, đơn, số tiền; cập nhật trạng thái có điều kiện và xử lý lặp an toàn. Với kết quả không rõ do timeout, em dùng query/đối soát thay vì tạo thêm giao dịch mù quáng. Các mã phản hồi và cách ký phải đối chiếu tài liệu phiên bản đã tích hợp.

**Hỏi sâu:** IPN đến trước return? Hai IPN giống nhau? Hoàn tiền timeout nhưng provider đã xử lý thì sao?

### 34. Redis caching ở các dự án giúp gì và có thể gây lỗi gì?

**Trả lời gợi ý:**

Em chọn một endpoint thật, nêu dữ liệu cache, key, TTL và cách invalidation. Cache có thể giảm đọc lặp nhưng tạo dữ liệu cũ, stampede hoặc lộ dữ liệu nếu key thiếu tenant/quyền. Khi cập nhật database xong, xóa cache là một cách phổ biến nhưng vẫn phải xét race với request đang đọc.

Nếu Redis lỗi, fallback về database cần bảo vệ tải. Với tồn kho hoặc thanh toán, quyết định quan trọng phải dựa trên nguồn dữ liệu được bảo vệ đúng, không dựa vào giá trị cache cũ.

**Hỏi sâu:** Xóa cache thất bại? Nhiều instance cùng miss? Làm sao đo cache hit rate và tác dụng thật?

### 35. Node.js xử lý nhiều request như thế nào? async/await có hết block không?

**Trả lời gợi ý:**

JavaScript trên một event loop thường chạy trên một thread; I/O được phối hợp với hệ điều hành và cơ chế nền thích hợp để nhiều request cùng chờ mà không chặn nhau ở phần chờ. Nhưng code CPU nặng vẫn có thể block thread đó, dù nằm trong hàm async.

Em liên hệ với xử lý webhook và job: parse payload quá lớn hoặc vòng lặp dài có thể làm chậm API và ảnh hưởng worker. Với việc nặng CPU, em cân nhắc worker thread/process riêng; Promise.all cũng cần giới hạn concurrency để không quá tải database hoặc provider.

**Hỏi sâu:** Promise.all reject có hủy tác vụ khác không? Vì sao dùng queue mà worker vẫn bị stalled?

Nguồn: [Node.js — Don't Block the Event Loop](https://nodejs.org/learn/asynchronous-work/dont-block-the-event-loop).

### 36. Express và NestJS khác nhau ở cách tổ chức xử lý request thế nào?

**Trả lời gợi ý:**

Ở Express em tổ chức middleware, route và lớp nghiệp vụ theo quy ước nhóm. NestJS bổ sung module, DI và các thành phần chuyên trách như guard, pipe, interceptor. Luồng tổng quát ở Nest là middleware, guard, interceptor trước handler, pipe, handler/service rồi interceptor xử lý response; exception filter xử lý lỗi phù hợp chưa được bắt.

Em dùng guard cho quyền truy cập, pipe cho validate/transform và service cho nghiệp vụ. Khung Controller–Service–Repository trong CV là kiến trúc phân lớp; nó không tự đồng nghĩa với đầy đủ MVC hoặc microservices.

**Hỏi sâu:** Vì sao không đặt tất cả logic vào controller? Guard kiểm tra quyền tài nguyên đến mức nào? Interface TypeScript có inject trực tiếp ở runtime không?

Nguồn: [NestJS — Request Lifecycle](https://docs.nestjs.com/faq/request-lifecycle).

### 37. API list đơn chậm: em tìm nguyên nhân và chứng minh đã sửa thế nào?

**Trả lời gợi ý:**

Em xem endpoint, tải, latency p95/p99 và trace để phân biệt chậm do query, pool, code CPU hay API ngoài. Với query, em xem execution plan, số dòng đọc, điều kiện tenant/filter và pagination, rồi cân nhắc index hoặc thay cách query.

N+1 có thể xử lý bằng batch hoặc JOIN phù hợp; offset rất lớn có thể cần keyset pagination theo thứ tự ổn định. Em so sánh trước/sau dưới điều kiện tương đương. Không có số đo thật thì em nói rõ kết quả định tính, không tự thêm phần trăm tối ưu.

**Hỏi sâu:** Query nhanh khi chạy riêng nhưng API vẫn chậm? Thêm index làm write chậm thì quyết định thế nào?

### 38. CV có MySQL và MongoDB nhưng dự án chủ yếu là PostgreSQL: em giải thích sao?

**Trả lời gợi ý:**

Kinh nghiệm dự án được trình bày rõ nhất trong CV của em là PostgreSQL. Với MySQL/MongoDB, em sẽ nói đúng phạm vi đã dùng ở `[dự án hoặc bài thực hành thật]`; nếu chỉ học hoặc thử nghiệm thì nêu rõ.

Về lựa chọn, em xét mô hình dữ liệu, query, transaction, ràng buộc và năng lực vận hành. MongoDB có document model và vẫn cần thiết kế schema; MySQL và PostgreSQL đều cần hiểu hành vi isolation/index thực tế. Em không khẳng định thành thạo production chỉ từ danh sách Skills.

**Hỏi sâu:** MongoDB nên embed hay reference? Index compound theo query nào? Khác biệt nào giữa hai SQL database đã trực tiếp gặp?

## 7. AI coding, bảo mật, Git và khoảng trống so với JD

### 39. Em dùng Claude Code/Cursor/Copilot thực tế thế nào?

**Trả lời gợi ý:**

CV xác nhận em dùng AI hỗ trợ phát triển test và chất lượng code, nhưng chưa ghi tên công cụ. Em cần bổ sung đúng công cụ đã dùng, tần suất, task và cách kiểm tra. Một câu trả lời tốt là: trong task `[thật]`, em giao AI `[phần việc]`, cung cấp `[ngữ cảnh]`, rồi phát hiện hoặc chỉnh `[điểm cụ thể]` trước merge.

Nếu chưa dùng Claude Code, em nói rõ công cụ tương tự đã quen và mức thực hành Claude Code hiện tại. Không dùng kinh nghiệm tích hợp Anthropic API để thay cho kinh nghiệm sử dụng Claude Code vì đây là hai loại công việc khác nhau.

**Hỏi sâu:** Agent đã đọc/sửa file và chạy lệnh gì? Có task nào AI làm sai? Em quyết định dừng agent khi nào?

### 40. Hãy giao cho AI task sửa lỗi truy cập đơn hàng khác tenant ở ViShop.

**Trả lời gợi ý:**

Em sẽ yêu cầu agent đọc luồng xác thực, tenant context, endpoint và Prisma extension trước khi đề xuất patch. Em chỉ rõ lỗi cần tái hiện, invariant và phạm vi để tránh sửa riêng một handler trong khi đường bypass vẫn còn.

**Prompt thực hành trên dữ liệu giả:**

```text
Mục tiêu: ngăn user tenant A truy cập order của tenant B.
Đọc trước: auth context, Orders module, Prisma extensions và test liên quan.
Không đọc secrets hoặc dữ liệu khách hàng; dùng fixture tổng hợp.
Trước khi sửa: nêu luồng hiện tại, đường bypass và cách tái hiện.
Yêu cầu:
- Tenant lấy từ context được server xác thực; thiếu context phải fail closed.
- Xem cả get/list/update, relation access và background job liên quan.
- Giữ response và quy ước lỗi hiện tại; không thêm dependency nếu không cần.
- Không refactor ngoài phạm vi.
Kiểm chứng: test A/B, thiếu context, quyền shop và regression hợp lệ.
Kết quả: báo diff, test đã chạy, test chưa chạy và rủi ro còn lại.
```

**Hỏi sâu:** Nếu lỗi nằm ở raw SQL? Làm sao biết test thật sự thất bại trước patch?

### 41. Em sẽ viết CLAUDE.md thế nào cho repository kiểu ViShop?

**Trả lời gợi ý:**

Em ghi cấu trúc module, quy tắc tenant/shop, invariant tồn kho, cách dùng transaction và lệnh kiểm tra đúng của repository. Nội dung phải ngắn và cập nhật theo code, không chứa token hay thông tin khách hàng. Hướng dẫn hỗ trợ agent, không thay thế phân quyền công cụ hoặc CI.

**Mẫu để điều chỉnh sau khi đọc repository thật:**

```markdown
# Hướng dẫn backend
- Đọc cấu hình Nx/package.json để xác định target build/test hiện có.
- Controller nhận request; nghiệp vụ ở application/domain phù hợp.
- Tenant lấy từ auth context, không lấy tùy ý từ body.
- Tồn kho phải bảo vệ trước request đồng thời và xử lý trùng.
- Query trong transaction dùng đúng transaction client/context.
- Giữ thứ tự khóa nhất quán khi thao tác nhiều dòng tồn kho.
- Dùng fixture giả; không đọc/ghi secrets vào prompt hoặc log.
- Báo rõ kiểm tra đã chạy và các giả định chưa xác minh.
```

**Hỏi sâu:** Quy tắc viết trong file có ngăn agent đọc secret tuyệt đối không? Tài liệu quá dài thì tách phần nào?

Nguồn: [Claude Code — CLAUDE.md và Memory](https://code.claude.com/docs/en/memory).

### 42. Review code AI trước merge: em kiểm tra gì?

**Trả lời gợi ý:**

Em đọc toàn bộ diff, kiểm tra nghiệp vụ, quyền truy cập, transaction/concurrency, query/hiệu năng và khả năng bảo trì. Em chú ý migration, dependency và cấu hình vì lỗi không chỉ nằm trong file nghiệp vụ.

Sau đó em kiểm chứng bằng test phù hợp với rủi ro: tenant A/B, request trùng, rollback, input biên và lỗi provider. Code và test do cùng AI viết có thể cùng sai một giả định, nên em suy test từ yêu cầu độc lập. Type check, lint và test xanh hỗ trợ review, không tự chứng minh đúng nghiệp vụ.

**Hỏi sâu:** Cho ví dụ test AI viết luôn pass nhưng không bắt lỗi. Nếu chưa giải thích được patch thì có merge không?

### 43. Làm sao dùng AI mà không lộ source, key hoặc dữ liệu hội thoại CRM?

**Trả lời gợi ý:**

Em dùng công cụ và phạm vi dữ liệu được tổ chức cho phép; xác nhận source nào được gửi tới dịch vụ bên ngoài, chính sách lưu trữ và cấu hình áp dụng. Em dùng fixture tổng hợp hoặc dữ liệu đã xử lý phù hợp, không paste token, file môi trường hay hội thoại khách hàng vào prompt.

Em giới hạn quyền truy cập và công cụ theo task, kiểm tra nội dung log/trace và không mặc định bật lưu prompt đầy đủ. Nếu lộ key, cần thu hồi/rotate, kiểm tra sử dụng bất thường và xử lý incident. Viết “đừng đọc secret” trong prompt không thay thế kiểm soát kỹ thuật.

**Hỏi sâu:** Xóa tên người dùng đã đủ khử nhận dạng chưa? Agent đọc issue hoặc log có thể gặp secret ở đâu?

### 44. MCP là gì? Em đã dùng chưa? Nếu kết nối CRM thì bảo vệ gì?

**Trả lời gợi ý:**

CV chưa có bằng chứng đã dùng MCP nên em cần trả lời đúng mức trải nghiệm. Về nguyên lý, MCP chuẩn hóa cách ứng dụng AI kết nối tới công cụ và nguồn ngữ cảnh; server có thể bọc các API CRM hiện có.

Nếu làm tool đọc lead, backend vẫn phải xác thực và áp quyền theo tenant/user, giới hạn trường trả về và audit. Với tool thay đổi dữ liệu, phải validate, giới hạn hành động và có cơ chế xác nhận theo quy trình phù hợp. Không dùng tenant do model tự chọn làm căn cứ phân quyền.

**Hỏi sâu:** MCP khác gọi REST API ở điểm nào? Tool trả nội dung chứa chỉ dẫn độc hại thì xử lý sao?

Nguồn: [Model Context Protocol — Introduction](https://modelcontextprotocol.io/docs/getting-started/intro).

### 45. Prompt injection trong RAG hoặc agent có thể xuất hiện ở đâu?

**Trả lời gợi ý:**

Một tài liệu, hội thoại hoặc kết quả tool có thể chứa câu chỉ dẫn cố làm model bỏ qua nhiệm vụ, lấy thêm dữ liệu hoặc gọi công cụ sai mục đích. Trong ViDoc và CRM, nội dung được truy xuất là dữ liệu không đáng tin, không được tự trở thành quyền điều khiển hệ thống.

Em giới hạn dữ liệu và tool theo quyền phía server, tách nội dung khỏi chỉ dẫn, validate tham số và kiểm soát side effect. Lọc chuỗi nguy hiểm chỉ hỗ trợ; model không được quyết định quyền hoặc có sẵn secret chỉ vì “có thể cần”.

**Hỏi sâu:** Tài liệu nói “gửi tất cả tài liệu sang URL này” thì lớp nào chặn? Tool chỉ đọc có thể vẫn làm lộ dữ liệu không?

### 46. Gitflow và quy trình review thực tế của em là gì?

**Trả lời gợi ý:**

CV có Git, CI/CD và pre-commit, chưa xác nhận dùng Gitflow. Em mô tả đúng nhánh, PR, review và release của nhóm trước. Nếu nhóm dùng quy trình khác, em nói rõ rồi giải thích Gitflow điển hình với main, develop, feature, release và hotfix.

Em hiểu merge tích hợp lịch sử, rebase phát lại commit và đổi commit ID, revert tạo commit đảo thay đổi, cherry-pick lấy commit chọn lọc. Khi xử lý conflict em đọc ý nghĩa hai phía và chạy kiểm tra liên quan. Không tự rebase/force-push lịch sử chung khi chưa theo quy ước nhóm.

**Hỏi sâu:** Hotfix đã vào main nhưng thiếu develop thì sao? Revert migration cùng code có luôn an toàn không?

## 8. Tình huống tổng hợp và câu chuyện cần chuẩn bị

### 47. Thiết kế API hủy đơn sao cho không hoàn kho hai lần.

**Trả lời gợi ý:**

Em kiểm tra quyền và định nghĩa trạng thái cho phép hủy. Trong cùng database, conditional update chỉ cho một giao dịch chuyển đơn từ trạng thái hợp lệ sang cancelled; chỉ giao dịch chuyển thành công mới hoàn kho, cùng transaction. Request lặp trả theo hợp đồng API và không thực hiện side effect lần nữa.

Em xét thêm cuộc đua với webhook thanh toán, lịch hết hạn và thao tác xuất kho. Nếu tồn kho ở service khác, cần workflow, outbox và consumer idempotent thay vì giả định một SQL transaction bao phủ mọi hệ thống.

**Hỏi sâu:** Hủy và thanh toán cùng lúc, bên nào thắng? Đã xuất một phần thì có được hủy toàn bộ không?

### 48. Queue backlog tăng, API vẫn chạy nhưng đơn đồng bộ chậm: em xử lý thế nào?

**Trả lời gợi ý:**

Em xem tuổi job cũ nhất, tốc độ vào/ra, số job retry/failed và latency của database/provider. Kiểm tra worker còn chạy, bị CPU block, cạn connection pool hay chạm rate limit. Em xác định nút thắt trước khi tăng concurrency vì tăng worker có thể làm provider hoặc database tệ hơn.

Em ưu tiên luồng quan trọng, giới hạn retry gây bão và báo trạng thái đồng bộ cho người dùng. Sau phục hồi, đối soát những job có kết quả không rõ và bảo đảm replay không làm trùng nghiệp vụ.

**Hỏi sâu:** Queue depth thấp nhưng một số đơn vẫn treo lâu? Làm sao phân biệt poison job với provider outage?

### 49. Kể một lần có bug production hoặc AI tạo code sai.

**Khung trả lời — phải điền sự kiện thật:**

Trong `[dự án]`, em làm `[task]`. Triệu chứng là `[sự cố]`, ảnh hưởng `[phạm vi đã xác nhận]`. Em dùng `[log/test/query]` để xác định nguyên nhân là `[cụ thể]`, rồi khôi phục hoặc sửa bằng `[cách làm]`. Em kiểm tra lại bằng `[bằng chứng]` và bổ sung `[test/monitoring/quy tắc]` để tránh lặp.

Nếu có AI tham gia, nói rõ AI làm phần nào, vì sao review ban đầu chưa phát hiện và điều chỉnh quy trình thế nào. CV không mô tả incident cụ thể nên không tự tạo câu chuyện hoặc số liệu.

**Hỏi sâu:** Em tự làm gì? Có ảnh hưởng dữ liệu không? Thay đổi nào khiến lỗi khó tái diễn hơn?

### 50. Em muốn hỏi lại công ty những gì?

**Câu hỏi đề xuất:**

- Trong ba tháng đầu, kết quả nào được xem là hoàn thành tốt ở vị trí này?
- Công việc ưu tiên là phát triển backend, dùng AI coding hay trực tiếp xây tính năng AI?
- Hệ thống đang có vấn đề gì về dữ liệu, hiệu năng, đồng bộ hoặc vận hành?
- Công ty kỳ vọng dùng Claude Code theo quy trình nào? Đã có hướng dẫn và cơ chế review chưa?
- Công ty cho phép đưa những loại source/dữ liệu nào vào AI? Có môi trường và cấu hình được duyệt không?
- MySQL, MongoDB, PostgreSQL và Elasticsearch được sử dụng ở những module nào?
- Nhóm kiểm thử, triển khai, quan sát hệ thống và xử lý incident ra sao?

## 9. Plan ôn phỏng vấn 14 ngày

Giả định mỗi ngày 2–3 giờ. Dùng chính dự án đã làm làm chất liệu, ưu tiên đào sâu thay vì tạo thêm một dự án lớn mới.

| Ngày | Câu cần luyện | Việc cần làm | Đầu ra kiểm chứng |
|---|---|---|---|
| 1 | 01–03 | Rà timeline và trách nhiệm từng dự án | Giới thiệu 90 giây; bảng phần tự làm/phối hợp |
| 2 | 04, 08–09 | Vẽ kiến trúc ViShop và luồng tenant/shop | Một sơ đồ thật; checklist đường bypass |
| 3 | 05–07, 47 | Ôn tồn kho, transaction và lock ordering | Giải được race mua/hủy/chuyển kho trên giấy |
| 4 | 10–11 | Đào sâu webhook, idempotency và OAuth | Bảng crash point, duplicate và token refresh |
| 5 | 12–14, 48 | Ôn outbox, queue, retry và election | Luồng publish/ack; cách xử lý lỗi từng bước |
| 6 | 15–17, 37 | SSE, index, partition và integration test | Một query plan hoặc ví dụ đã khử dữ liệu; test concurrency |
| 7 | 18–19 | Đào sâu GraphQL/DataLoader và metadata engine | Một resolver và schema thật có thể giải thích |
| 8 | 20–23 | Luồng AI CRM và tiêu chí chất lượng | Sơ đồ input/output, validation, fallback |
| 9 | 24–28 | ViDoc ingestion, IR, chunking và encoding | Một tài liệu giả chạy qua pipeline; ví dụ lỗi |
| 10 | 29–30, 44–45 | Retrieval, citation, MCP và prompt injection | Phân biệt rõ kinh nghiệm thật và phương án đề xuất |
| 11 | 39–43 | Thực hành giao task cho AI trên code mẫu | Prompt, hướng dẫn dự án, diff review, kết quả kiểm tra |
| 12 | 31–38, 46 | JIHO, MIAPET, Node cơ bản và khoảng trống JD | Trả lời VNPay, cache, Elasticsearch, Gitflow; rà MySQL/MongoDB |
| 13 | 49–50 | Chuẩn bị câu chuyện và hỏi ngược | Năm câu chuyện thật; danh sách điều chưa chắc |
| 14 | Toàn bộ phần ưu tiên | Mock interview, sửa câu trả lời yếu | Bản tự chấm và danh sách ôn cuối |

Mỗi buổi có thể dành 30 phút ôn ý, 60–90 phút vẽ hoặc thực hành, 30 phút trả lời thành tiếng. Tài liệu/code mang đi minh họa phải được phép chia sẻ và dùng dữ liệu tổng hợp.

### Nếu chỉ còn ba ngày

| Ngày | Trọng tâm |
|---|---|
| 1 | Giới thiệu; ViShop: câu 04–10, 12, 17, 47 |
| 2 | CRMAgent/ViDoc: câu 18, 20–21, 24–30; AI coding: 39–43 |
| 3 | Node/Nest, VNPay/cache, Gitflow và khoảng trống: 31–38, 44–46; mock 90 phút |

## 10. Kịch bản mock interview 90 phút

| Thời lượng | Nội dung | Câu tham chiếu |
|---|---|---|
| 5 phút | Giới thiệu, timeline và phần trực tiếp phụ trách | 01–03 |
| 25 phút | ViShop: kiến trúc, concurrency, tenant, idempotency | 04–12 |
| 10 phút | Node/Nest, query và test | 17, 35–37 |
| 15 phút | CRM LLM và ViDoc RAG | 20–21, 24–30 |
| 15 phút | Giao task AI, review, bảo mật và MCP | 39–45 |
| 10 phút | Bài tình huống hủy đơn hoặc queue backlog | 47–48 |
| 10 phút | Câu chuyện, Gitflow, hỏi ngược và phản hồi | 46, 49–50 |

### Thang tự chấm

- **0:** Chưa giải thích được hoặc đang đoán cơ chế hệ thống mình từng làm.
- **1:** Nêu được định nghĩa nhưng chưa có ví dụ cụ thể.
- **2:** Giải thích được luồng và đóng góp cá nhân, có test hoặc bằng chứng.
- **3:** Phân tích được đánh đổi, lỗi đồng thời, điểm chết và giới hạn của giải pháp.

Trước buổi phỏng vấn, ưu tiên đưa nhóm tồn kho/tenant/idempotency và AI review lên mức 2–3. Với công nghệ chưa vận hành, trả lời rõ giới hạn và cách tiếp cận thay vì học thuộc một câu khẳng định quá mức.

### Thông tin cần tự bổ sung trước khi học câu trả lời

- Công cụ AI đã dùng, task cụ thể và một lỗi đã phát hiện khi review.
- Phân bổ thời gian thực tế giữa ViShop và ViDoc.
- Kiến trúc triển khai thật: module nào cùng process, thành phần nào độc lập.
- Schema và thao tác transaction/locking mình trực tiếp làm.
- Bảng partition, query/index và số liệu hiệu năng có thể kiểm chứng.
- Cơ chế Outbox, leader election và idempotency đang triển khai thật ở đâu.
- Loại vector index, embedding model và phần retrieval/generation mình trực tiếp phụ trách.
- Trải nghiệm MySQL/MongoDB, Gitflow và MCP ở mức nào.

Nguồn thông tin cá nhân trong tài liệu là CV do người dùng cung cấp. Những câu hỏi đào sâu và giải pháp tình huống là nội dung chuẩn bị phỏng vấn được xây dựng từ CV và JD, không phải xác nhận chi tiết triển khai ngoài CV.
