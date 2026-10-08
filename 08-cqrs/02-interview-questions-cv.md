# 🎯 CHUYÊN ĐỀ 08.2: CQRS – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu về kiến trúc CQRS gắn liền với dự án **ViShop (WMS/OMS Platform)**: **Phân tách Command & Query Stack**, **Giải quyết Eventual Consistency Lag với Server-Sent Events (SSE)**, **Optimistic Concurrency Control (OCC)**, và **Chống lỗi Dual-Write với Transactional Outbox**.

---

## PHẦN 1: DỰ ÁN VISHOP – CQRS & EVENT-DRIVEN ARCHITECTURE

### Case 1: Phân tách Write Stack & Read Stack trong ViShop
> **Trích dẫn CV**: *"Architected a production-grade Nx Monorepo with 15 domain modules and 4 runnable microservices, adopting Domain-Driven Design (DDD) and CQRS... Engineered a high-integrity WMS inventory ledger with PostgreSQL triggers and command handlers..."*

#### Q1: Tại sao trong dự án ViShop em lại quyết định áp dụng CQRS thay vì kiến trúc Layered/CRUD truyền thống? Em phân tách luồng Command và luồng Query như thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Vấn đề của hệ thống E-Commerce WMS truyền thống**:
    * Màn hình tra cứu tồn kho, tìm kiếm đơn hàng từ nhiều sàn thương mại điện tử (TikTok Shop, Shopee) đòi hỏi dữ liệu denormalized, gom từ nhiều bảng (`orders`, `order_items`, `warehouse`, `customers`) $\rightarrow$ Các câu query đọc rất nặng, làm khóa hoặc cạnh tranh tài nguyên với luồng ghi.
    * Trong khi đó, luồng ghi đơn hàng và xuất kho WMS đòi hỏi tính toàn vẹn cao, phải ghi sổ cái kiểm toán (`inventory_ledger`) và kiểm tra bất biến tồn kho từng giây.
  * **Triển khai CQRS trong ViShop**:
    * **Command Stack (Tầng Ghi)**:
      * Nhận Command (`CreateOrderCommand`, `DeductStockCommand`).
      * Đi qua Command Handler $\rightarrow$ Nạp Aggregate Root $\rightarrow$ Validate quy tắc bất biến $\rightarrow$ Ghi vào PostgreSQL Write Tables (3NF) VÀ ghi vào bảng `outbox_events` trong cùng một transaction.
    * **Read Stack (Tầng Đọc)**:
      * Bỏ qua hoàn toàn Domain Model phức tạp.
      * Query Handler đọc trực tiếp từ các Read Projections đã được tối ưu hóa (Denormalized Views, Redis Cache hoặc Elasticsearch) $\rightarrow$ Trả về dữ liệu cho client với độ trễ $< 20ms$, không bao giờ làm ảnh hưởng đến luồng ghi kho của WMS.

---

### Case 2: Hóa giải Eventual Consistency Delay với Server-Sent Events (SSE)
> **Trích dẫn CV**: *"Developed Server-Sent Events (SSE) for real-time order, inventory, and synchronization status updates to connected clients."*

#### Q2: Trong kiến trúc CQRS với Polyglot Persistence hoặc Asynchronous Projections, luôn tồn tại độ trễ đồng bộ (Eventual Consistency Lag). Em đã dùng Server-Sent Events (SSE) để giải quyết trải nghiệm người dùng như thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Thách thức**: Sau khi merchant ấn "Đồng bộ đơn hàng từ TikTok Shop", Command trả về mã `202 Accepted` ngay lập tức để giải phóng HTTP thread. Tuy nhiên, nếu merchant refresh trang ngay, danh sách đơn hàng vẫn chưa cập nhật do worker đang xử lý ngầm $\rightarrow$ Gây hoang mang cho merchant.
  * **Giải pháp kết hợp CQRS + SSE**:
    1. Khi Command được chấp thuận, API trả về `202 Accepted` kèm theo một `syncTaskId`.
    2. Frontend mở kết nối HTTP stream tới SSE endpoint: `GET /api/v1/sync/events?taskId=...`.
    3. Ở backend, khi Outbox Relay Worker và Projection Worker cập nhật dữ liệu xong, nó phát sinh event `OrderSyncCompleted` lên Redis Pub/Sub.
    4. SSE Controller nhận event từ Redis và đẩy (Push) một event thông báo xuống trình duyệt merchant:
       ```
       event: SYNC_COMPLETED
       data: {"syncedOrdersCount": 45, "timestamp": 1728372000}
       ```
    5. Giao diện frontend tự động làm mới bảng danh sách đơn hàng một cách mượt mà mà merchant không cần phải liên tục F5 trang.

---

### Case 3: Xử lý Xung đột Ghi đồng thời với Optimistic Concurrency Control (OCC)
> **Trích dẫn CV**: *"maintain inventory consistency and prevent race conditions during warehouse transfers and concurrent updates."*

#### Q3: Khi hai Command đồng thời cùng cố gắng sửa đổi một Aggregate trong mô hình CQRS, em xử lý xung đột ra sao để không bị mất dữ liệu?
* 💡 **Câu trả lời ăn điểm**:
  * **Triển khai Optimistic Concurrency Control (OCC) bằng cột `version`**:
    1. Mỗi Aggregate Root trong Write Database đều mang theo một trường `version INT DEFAULT 1`.
    2. Khi Command Handler nạp Aggregate từ DB, nó ghi nhận phiên bản hiện tại: ví dụ `expectedVersion = 3`.
    3. Khi lưu Aggregate sau khi thực thi nghiệp vụ, câu lệnh SQL bắt buộc phải kiểm tra version:
       ```sql
       UPDATE inventory_balances 
       SET available_quantity = $1, 
           version = version + 1, 
           updated_at = NOW()
       WHERE id = $2 AND version = 3;
       ```
    4. **Xử lý xung đột**:
       * Nếu có Command khác vừa cập nhật và đẩy version lên 4, câu lệnh trên sẽ trả về `affectedRows = 0`.
       * Command Handler phát hiện xung đột, ném ra ngoại lệ `OptimisticLockException`.
       * Tầng Use Case bắt lỗi và tự động thực hiện **Retry with Exponential Backoff** (nạp lại Aggregate ở version 4 và thực thi lại logic trừ kho).

---

## PHẦN 2: BỘ CÂU HỎI VỀ CQRS ARCHITECTURAL TRADE-OFFS

### Q4: Khi nào bạn KHÔNG NÊN sử dụng kiến trúc CQRS?
* 💡 **Câu trả lời ăn điểm**:
  * Tuyệt đối không nên áp dụng CQRS khi:
    1. **Hệ thống là CRUD cơ bản**: Số lượng thao tác đọc và ghi tương đương nhau, mô hình dữ liệu đọc không khác biệt so với mô hình ghi.
    2. **Đòi hỏi Strict Immediate Consistency trên toàn bộ hệ thống**: Nghiệp vụ bắt buộc mọi màn hình đều phải thấy dữ liệu mới nhất 100% tại cùng 1 mili-giây mà không thể chấp nhận mô hình Eventual Consistency.
    3. **Chi phí và nguồn lực hạn chế**: Áp dụng CQRS (đặc biệt là 2 Database độc lập) làm tăng gấp đôi chi phí bảo trì hạ tầng, đòi hỏi đội ngũ phải quản trị được Event Versioning, Schema Migration trên nhiều database và xử lý bài toán Dead-Letter Queue khi Projection bị lỗi.

---

### Q5: Phân biệt sự khác nhau giữa CQRS và Event Sourcing? Có bắt buộc phải dùng Event Sourcing khi làm CQRS không?
* 💡 **Câu trả lời ăn điểm**:
  * **Sự khác biệt**:
    * **CQRS (Command Query Responsibility Segregation)**: Là mô hình phân tách đường dẫn ghi (Command) và đường dẫn đọc (Query). **Không quy định cách bạn lưu trữ dữ liệu ở tầng ghi**. Tầng ghi hoàn toàn có thể là một cơ sở dữ liệu quan hệ PostgreSQL lưu trạng thái hiện tại (Current State).
    * **Event Sourcing (ES)**: Là mô hình lưu trữ dữ liệu dưới dạng **một chuỗi các sự kiện biến động bất biến (Append-Only Event Stream)** thay vì lưu trạng thái hiện tại. Trạng thái hiện tại được tái dựng bằng cách Replay lại toàn bộ các sự kiện từ quá khứ.
  * **Có bắt buộc dùng chung không?**:
    * **Hoàn toàn KHÔNG!** Đây là hai pattern độc lập. Trong dự án ViShop, em áp dụng **CQRS với State-based Database (PostgreSQL)** kết hợp Transactional Outbox. Điều này mang lại lợi ích tối đa về hiệu năng của việc phân tách đọc/ghi mà không phải gánh chịu sự phức tạp khổng lồ của Event Sourcing (Event Versioning, Snapshotting, Replay overhead).
