# CÂU TRẢ LỜI PHỎNG VẤN

JAVA CORE DEVELOPER • DOMAIN FINTECH

33 câu trả lời ngắn gọn • Middle / Senior

Cách dùng: thay phần [ ... ] bằng kinh nghiệm thật. Các ví dụ kỹ thuật dùng để giải thích cách làm; chỉ nhận đã triển khai khi đúng với thực tế của bạn.

Biên soạn theo: Bo_Cau_Hoi_Tham_Khao_PV_Java_Core_Fintech.pdf

## GIỚI THIỆU VÀ DỰ ÁN

### Câu 1. Giới thiệu dự án đã làm

“Em tham gia dự án [tên/lĩnh vực], phụ trách [module và công việc chính], sử dụng [công nghệ]. Một bài toán tiêu biểu là [vấn đề]; em xử lý bằng [giải pháp] và đạt [kết quả thực tế].”

Nếu hỏi luồng thanh toán: tạo yêu cầu → kiểm tra điều kiện, số dư → duyệt nếu có → gửi đối tác → cập nhật trạng thái → đối soát.

### Câu 2. Dự án có xử lý tiền thật không? Quy mô bao nhiêu TPS?

“Dự án của em [có/chưa] xử lý tiền thật. Hệ thống đạt khoảng [TPS trung bình/đỉnh, nếu có đo]; em phụ trách [phạm vi].”

Nếu chưa nắm số liệu: “Em chưa có số TPS được xác nhận nên không muốn đưa ra con số thiếu chính xác.”

### Câu 3. Cần ghi log gì để truy vết sự cố?

“Em ghi thời gian, service, trace ID, mã giao dịch, thao tác, trạng thái trước/sau, thời gian xử lý và mã lỗi. Khi gọi đối tác, em ghi mã tham chiếu cùng kết quả đã che dữ liệu nhạy cảm; không ghi mật khẩu, token hoặc thông tin thẻ đầy đủ.”

### Câu 4. Có làm được JavaScript không?

“Em có thể sử dụng JavaScript ở mức [thực tế], gồm xử lý sự kiện, gọi API, Promise và async/await. Em đã dùng trong [tính năng/dự án]; phần chuyên sâu nhất của em là [backend/frontend].”

## JAVA CORE, CONCURRENCY VÀ JVM

### Câu 5. Abstract Class và Interface khác nhau thế nào?

“Abstract class phù hợp khi các lớp con chia sẻ trạng thái và logic chung; có constructor, biến instance và phương thức cụ thể. Interface định nghĩa hợp đồng hành vi, hỗ trợ một lớp triển khai nhiều interface; có thể có default/static method. Một lớp chỉ kế thừa trực tiếp một class.”

### Câu 6. Override và Overload khác nhau thế nào?

“Overload là cùng tên phương thức nhưng khác danh sách tham số, được lựa chọn ở compile time; chỉ khác kiểu trả về thì chưa đủ. Override là lớp con định nghĩa lại phương thức instance của lớp cha với cùng chữ ký, được lựa chọn ở runtime theo đối tượng thực tế.”

### Câu 7. Đã áp dụng Design Pattern nào?

Ví dụ nếu phù hợp kinh nghiệm: “Em dùng Strategy để tách cách xử lý thanh toán theo từng đối tác, kết hợp Factory để chọn implementation. Cách này giúp thêm đối tác mới mà ít sửa luồng chính; Adapter dùng để chuẩn hóa request/response của từng bên.”

### Câu 8. Xử lý nhiều luồng cùng cập nhật dữ liệu thế nào?

“Em chọn theo phạm vi dữ liệu và thao tác cần bảo vệ.”

- synchronized: bảo vệ đoạn xử lý nhiều bước, tự giải phóng khóa.

- Lock: linh hoạt hơn với tryLock, timeout; cần unlock trong finally.

- Atomic: phù hợp thao tác nguyên tử trên một biến; không tự bảo vệ nhiều biến liên quan.

- ConcurrentHashMap: map an toàn khi truy cập đồng thời; dùng compute/merge cho cập nhật kết hợp.

“volatile đảm bảo khả năng nhìn thấy thay đổi, không làm count++ trở thành nguyên tử. Các cơ chế trên chỉ bảo vệ trong phạm vi JVM, không thay thế khóa dữ liệu giữa nhiều instance.”

### Câu 9. Khi nào dùng ExecutorService hoặc CompletableFuture?

“Em dùng ExecutorService để quản lý việc chạy task qua thread pool; dùng CompletableFuture để kết hợp các tác vụ bất đồng bộ, như gọi nhiều API độc lập. Pool được cấu hình theo số CPU, tỷ lệ chờ I/O, giới hạn kết nối và kết quả load test; cần kiểm soát queue, timeout và xử lý khi quá tải.”

### Câu 10. Phát hiện và xử lý memory leak, GC pause thế nào?

“Nếu gặp, em kiểm tra heap usage sau GC, GC log và thời gian pause. Heap dump giúp tìm đối tượng bị giữ tham chiếu như cache không giới hạn hoặc ThreadLocal chưa dọn; thread dump giúp kiểm tra luồng bị block/deadlock. Em sửa nguyên nhân rồi đo lại, thay vì chỉ tăng heap.”

Nếu chưa gặp trên production, nói rõ đây là cách bạn sẽ tiếp cận.

## SPRING VÀ MICROSERVICES

### Câu 11. Có những cách khai báo Bean nào?

“Các cách phổ biến là component scanning với @Component, @Service, @Repository, @Controller; khai báo phương thức @Bean trong class cấu hình; hoặc khai báo bằng XML. @Bean thường phù hợp khi cần quản lý đối tượng của thư viện bên ngoài.”

### Câu 12. Tạo class cấu hình dùng annotation nào?

“Em dùng @Configuration, kết hợp @Bean để khai báo các đối tượng Spring quản lý. Nếu gom cấu hình từ properties/YAML thành đối tượng, em thường dùng @ConfigurationProperties.”

### Câu 13. Class bất kỳ dùng @Value có được không?

“Có thể viết annotation, nhưng Spring chỉ tự inject khi đối tượng được Spring quản lý và xử lý. Nếu tự tạo bằng new, field @Value thông thường không được inject. Em đăng ký class thành bean hoặc truyền cấu hình qua constructor.”

### Câu 14. Authentication và Authorization khác nhau thế nào?

“Authentication xác minh người dùng là ai; Authorization xác định họ được phép làm gì. Với Spring Security, em cấu hình xác thực và phân quyền theo endpoint hoặc method; nếu dùng JWT thì kiểm tra chữ ký, thời hạn và các claim cần thiết trước khi cấp quyền.”

### Câu 15. Kafka/RabbitMQ và Redis giải quyết vấn đề gì?

“Kafka phù hợp luồng sự kiện cần lưu và đọc lại; RabbitMQ phù hợp hàng đợi công việc và định tuyến message. Redis thường dùng cho cache, session, rate limiting hoặc dữ liệu tạm có TTL. Với giao dịch tiền, dữ liệu tài chính phải có nơi lưu bền vững; consumer cần xử lý được message trùng.”

### Câu 16. API đối tác chậm hoặc lỗi thì xử lý thế nào?

“Em đặt connect/read timeout; retry có giới hạn với exponential backoff và jitter cho lỗi tạm thời. Circuit breaker giúp ngắt gọi khi đối tác lỗi kéo dài, rate limiting giới hạn tải. Với lệnh thanh toán, chỉ retry khi có cơ chế chống trùng; timeout chưa đủ để kết luận giao dịch thất bại.”

## CƠ SỞ DỮ LIỆU

### Câu 17. Đã dùng database nào? Có dùng Oracle không?

“Em đã dùng [database] cho [mục đích], làm việc với SQL, index và transaction. Với Oracle, em [đã dùng trong…/chưa dùng production]; phần em nắm là [SQL, sequence, execution plan, PL/SQL… theo thực tế].”

### Câu 18. Index là gì? Đánh cho cột nào? Nhược điểm?

“Index là cấu trúc hỗ trợ tìm dữ liệu nhanh, giảm lượng dữ liệu phải quét. Em cân nhắc các cột thường dùng trong WHERE, JOIN, ORDER BY, dựa vào truy vấn và độ chọn lọc; index ghép cần xét thứ tự cột. Nhược điểm là tốn dung lượng và làm chậm thao tác ghi do phải cập nhật index.”

### Câu 19. Tối ưu SQL chậm thế nào?

“Em xem execution plan và số liệu thực thi để xác định chậm do scan, join, sort hay chờ lock. Sau đó cân nhắc index, viết lại điều kiện, giảm dữ liệu trả về, tránh N+1 và chọn cách phân trang phù hợp. Cuối cùng đo lại trên dữ liệu đại diện.”

### Câu 20. Partition trong Oracle là gì? Dùng khi nào?

“Partition chia bảng hoặc index thành các phần nhỏ theo range, list hoặc hash nhưng vẫn truy vấn như một bảng. Ví dụ bảng giao dịch lớn có thể chia theo tháng để dễ quản lý dữ liệu cũ; truy vấn theo ngày có thể chỉ đọc partition liên quan. Hiệu quả phụ thuộc khóa partition và điều kiện truy vấn.”

### Câu 21. UNION và UNION ALL khác nhau thế nào?

“UNION gộp kết quả và loại dòng trùng; UNION ALL giữ cả dòng trùng nên thường ít tốn xử lý hơn. Hai bên cần cùng số cột và kiểu dữ liệu tương thích; em dùng UNION ALL khi không cần loại trùng.”

### Câu 22. Liên kết dữ liệu hai bảng thế nào?

“Em dùng JOIN theo khóa liên quan: INNER JOIN lấy các dòng khớp, LEFT JOIN giữ toàn bộ dòng bên trái. Cần xác định quan hệ một–một hay một–nhiều để tránh nhân bản dữ liệu ngoài ý muốn.”

```sql
SELECT t.id, c.name
FROM transactions t
JOIN customers c ON c.id = t.customer_id;
```

### Câu 23. ACID, isolation level và deadlock là gì?

“ACID gồm: Atomicity — toàn bộ hoặc không; Consistency — giữ các ràng buộc dữ liệu; Isolation — kiểm soát ảnh hưởng giữa giao dịch đồng thời; Durability — commit rồi thì được lưu bền vững.”

Theo mô hình SQL chuẩn:

- Read Uncommitted: có thể đọc dữ liệu chưa commit.

- Read Committed: tránh dirty read.

- Repeatable Read: thêm bảo vệ khỏi non-repeatable read.

- Serializable: mức cô lập cao nhất, ngăn thêm phantom read.

“Deadlock xảy ra khi các transaction giữ khóa và chờ nhau theo vòng. Em giảm bằng cách khóa cùng thứ tự, giữ transaction ngắn; khi bị chọn rollback thì retry toàn bộ transaction có giới hạn.”

Oracle chủ yếu cung cấp Read Committed mặc định và Serializable; hành vi cụ thể cần xét theo database.

### Câu 24. Hai request cùng trừ tiền một ví thì làm thế nào?

“Em đảm bảo ở database bằng transaction và cập nhật có điều kiện, thay vì chỉ khóa trong Java.”

```sql
UPDATE wallets
SET balance = balance - :amount
WHERE id = :wallet_id
  AND balance >= :amount;
```

“Với amount > 0, em kiểm tra số dòng cập nhật: một dòng mới được tiếp tục, không dòng thì từ chối hoặc kiểm tra nguyên nhân. Có thể dùng optimistic locking với version, hoặc pessimistic locking bằng SELECT ... FOR UPDATE; ghi sổ cùng transaction và kết hợp idempotency để tránh trừ lặp.”

### Câu 25. SQL lấy số điện thoại có tổng giao dịch trên 10 triệu trong một tháng

Giả sử dùng Oracle, :from_date là đầu tháng và :to_date là đầu tháng kế tiếp:

```sql
SELECT phone_number, SUM(amount) AS total_amount
FROM transactions
WHERE transaction_date >= :from_date
  AND transaction_date <  :to_date
GROUP BY phone_number
HAVING SUM(amount) > 10000000;
```

“Nếu bảng có cả giao dịch lỗi, em bổ sung điều kiện chỉ tính giao dịch thành công. Em cũng xác nhận ‘một tháng’ là tháng lịch hay một tháng gần nhất.”

## GIAO DỊCH TÀI CHÍNH VÀ HỆ THỐNG PHÂN TÁN

### Câu 26. Idempotency là gì? Tránh xử lý trùng thế nào?

“Idempotency là gọi lặp cùng một yêu cầu nhưng không tạo thêm tác động nghiệp vụ. Em lưu idempotency key có unique constraint, gắn với nội dung yêu cầu, trạng thái và kết quả. Request trùng được trả kết quả cũ hoặc trạng thái đang xử lý; cùng key nhưng khác nội dung thì bị từ chối.”

### Câu 27. Giao dịch treo hoặc đối tác không phản hồi thì xử lý thế nào?

“Em đưa về PENDING/UNKNOWN, tra cứu bằng mã tham chiếu, chờ callback hoặc chạy đối soát để xác định kết quả. Khi đã xác nhận thất bại mới giải phóng tiền giữ hoặc thực hiện bút toán hoàn theo trạng thái thực tế. Thao tác hoàn/hủy cũng phải chống trùng; trường hợp chưa rõ được cảnh báo để xử lý tiếp.”

### Câu 28. Saga và Outbox là gì?

“Saga chia quy trình nhiều service thành các transaction cục bộ; khi một bước thất bại thì thực hiện hành động bù phù hợp với nghiệp vụ. Outbox ghi dữ liệu nghiệp vụ và event vào cùng transaction database, rồi tiến trình riêng phát event. Outbox tránh lệch giữa ghi DB và phát message; message vẫn có thể trùng nên consumer cần idempotent.”

### Câu 29. Đã làm đối soát hoặc hạch toán chưa?

Nếu chưa: “Em chưa trực tiếp phụ trách, nhưng em hiểu quy trình cơ bản.”

“Đối soát là lấy dữ liệu nội bộ và đối tác theo kỳ, chuẩn hóa rồi so khớp mã giao dịch, số tiền, trạng thái; sau đó phân loại chênh lệch và xử lý có kiểm soát. Hạch toán ghi nhận các bút toán Nợ/Có cân bằng; khi điều chỉnh thì tạo bút toán bổ sung hoặc đảo, giữ lịch sử truy vết.”

### Câu 30. Đã làm với NAPAS, VietQR, ISO 8583 hoặc Open API ngân hàng chưa?

“Em [đã tích hợp…/chưa trực tiếp tích hợp]. Em phân biệt NAPAS là mạng chuyển mạch tài chính; VietQR là chuẩn/thương hiệu QR thanh toán; ISO 8583 là chuẩn thông điệp giao dịch thẻ; Open API ngân hàng là các giao diện tích hợp. Khi triển khai, em bám đặc tả đối tác về xác thực, chữ ký, mã lỗi, timeout và đối soát.”

## DÀNH CHO ỨNG VIÊN SENIOR

### Câu 31. Từng dẫn dắt thiết kế giải pháp nào?

“Em phụ trách thiết kế [giải pháp] để giải quyết [vấn đề và yêu cầu tải]. Em so sánh [các phương án], chọn [phương án] vì [lý do], chấp nhận đánh đổi [chi phí/độ phức tạp/độ trễ]. Kết quả được đánh giá bằng [số liệu thực tế].”

### Câu 32. Hướng dẫn thành viên và review code theo tiêu chí nào?

“Em review tính đúng nghiệp vụ trước, tiếp theo là transaction, concurrency, bảo mật, khả năng vận hành và độ dễ bảo trì. Với giao dịch tiền, em chú ý tính toán chính xác bằng BigDecimal hoặc đơn vị tiền nhỏ nhất, quy tắc làm tròn và chống xử lý trùng. Khi hướng dẫn, em giải thích lý do, đưa ví dụ và để thành viên tự đề xuất cách sửa.”

### Câu 33. Hệ thống tải cao, độ trễ tăng thì phân tích thế nào?

“Em xác định phạm vi ảnh hưởng qua TPS, lỗi và độ trễ p95/p99; dùng metrics, trace và log để tìm điểm nghẽn. Sau đó kiểm tra CPU, GC, thread pool, connection pool, SQL/lock, queue và đối tác. Em giảm ảnh hưởng bằng biện pháp phù hợp như giới hạn tải hoặc rollback thay đổi gây lỗi, rồi sửa nguyên nhân và đo lại.”

## TÀI LIỆU THAM KHẢO

Java Concurrency (câu 8–9)

https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/concurrent/package-summary.html

Spring @Value (câu 13)

https://docs.spring.io/spring-framework/reference/6.2/core/beans/annotation-config/value-annotations.html

Oracle Partitioning (câu 20)

https://docs.oracle.com/en/database/oracle/oracle-database/26/vldbg/partition-concepts.html

Oracle Data Concurrency and Consistency (câu 23)

https://docs.oracle.com/en/database/oracle/oracle-database/26/cncpt/data-concurrency-and-consistency.html

AWS Transactional Outbox (câu 28)

https://docs.aws.amazon.com/en_en/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html
