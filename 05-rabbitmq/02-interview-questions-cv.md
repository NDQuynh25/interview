# 🎯 CHUYÊN ĐỀ 05.2: RABBITMQ – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu về Enterprise Messaging với RabbitMQ: **Zero Message Loss**, **Consumer Idempotency**, **Dead Letter Exchange (DLX)**, **Message Ordering với Consistent Hash Exchange**, và **So sánh RabbitMQ vs Kafka vs BullMQ**.

---

## PHẦN 1: BỘ CÂU HỎI THỰC CHIẾN VỀ ĐỘ TIN CẬY & KIẾN TRÚC

### Q1: RabbitMQ chỉ đảm bảo chuẩn At-Least-Once Delivery. Khi mạng chập chờn làm mất ACK và message bị bắn lại lần 2, em thiết kế Consumer Idempotency như thế nào để không tạo 2 đơn hàng?
* 💡 **Câu trả lời ăn điểm**:
  * **Bản chất vấn đề**: Khi Consumer đã xử lý xong và commit vào DB, nhưng kết nối mạng bị rớt đúng lúc gửi `Basic.Ack` về RabbitMQ $\rightarrow$ RabbitMQ phát hiện TCP socket đóng $\rightarrow$ Tự động requeue message và gửi sang cho Consumer khác chạy lại $\rightarrow$ Gây duplicate processing.
  * **Giải pháp 3 lớp bảo vệ Idempotency**:
    1. **Correlation / Message ID**: Mọi message khi Publisher gửi đi đều phải có thuộc tính `properties.messageId` duy nhất (UUID) hoặc khóa nghiệp vụ (`order_id + '_' + event_type`).
    2. **Kiểm tra trạng thái tại Database (Deduplication Table)**:
       * Consumer mở một Database Transaction:
         ```sql
         BEGIN;
         -- Chèn message_id vào bảng processed_messages có UNIQUE constraint:
         INSERT INTO processed_messages (message_id, processed_at) VALUES ($1, NOW());
         -- Nếu gặp lỗi Unique Violation (23505) -> Rollback và gửi basic.ack ngay lập tức!

         -- Thực thi nghiệp vụ trừ kho / tạo đơn hàng...
         COMMIT;
         ```
       * Nhờ ràng buộc `UNIQUE (message_id)` trong cùng 1 transaction của PostgreSQL, nếu message bị bắn lại lần 2, câu lệnh INSERT sẽ fail ngay lập tức, ngăn chặn 100% việc thực thi nghiệp vụ lần thứ hai.
    3. **Gửi `basic.ack` sau khi commit DB**: Chỉ sau khi database commit thành công mới gọi `channel.ack(msg)`.

---

### Q2: Sự khác biệt giữa Classic Mirrored Queues và Quorum Queues trong RabbitMQ là gì? Tại sao trên Kubernetes hiện nay bắt buộc phải dùng Quorum Queues?
* 💡 **Câu trả lời ăn điểm**:
  * **Classic Mirrored Queues (Đã deprecated)**:
    * Hoạt động theo mô hình Master-Mirror tùy biến của RabbitMQ.
    * Khi xảy ra hiện tượng Network Partition (mạng giữa các node trong cluster bị đứt đoạn), rất dễ dẫn đến hiện tượng **Split-Brain** (hai node cùng tự phong làm Master và nhận ghi độc lập) $\rightarrow$ Dẫn đến mất mát hoặc mâu thuẫn dữ liệu nghiêm trọng khi mạng nối lại.
    * Thao tác đồng bộ dữ liệu sang mirror node mới khi một Pod restart làm nghẽn toàn bộ queue.
  * **Quorum Queues**:
    * Được xây dựng hoàn toàn dựa trên thuật toán đồng thuận **Raft Consensus** (tương tự thuật toán của etcd trong Kubernetes).
    * Bắt buộc phải có đa số phiếu bầu (Quorum = $\lfloor N/2 \rfloor + 1$) mới được commit message hoặc bầu Leader mới. Do đó, **miễn nhiễm 100% với hiện tượng Split-Brain**.
    * Dữ liệu được ghi thẳng xuống đĩa cứng (Disk-first) với chi phí thấp và throughput cao hơn nhiều, cực kỳ ổn định khi các Pods Kubernetes thường xuyên bị restart hoặc dời node.

---

### Q3: Nếu một message gặp lỗi không thể xử lý (Poison Pill), em thiết kế hệ thống Retry với Dead Letter Exchange (DLX) như thế nào để không làm nghẽn Main Queue và không gây crash loop?
* 💡 **Câu trả lời ăn điểm**:
  * **Hạn chế của việc Requeue**: Nếu gọi `channel.nack(msg, false, true)` (requeue = true), message lỗi sẽ quay lại đầu hàng đợi và gửi cho worker lập tức $\rightarrow$ Worker crash liên tục hàng trăm lần/giây, làm nghẽn các message bình thường khác.
  * **Mô hình Dead Letter Exchange + TTL Queue**:
    1. **Main Queue (`order.process`)**: Cấu hình tham số `'x-dead-letter-exchange': 'order.dlx'` và `'x-dead-letter-routing-key': 'retry'`.
    2. Khi worker gặp lỗi, nó gọi `channel.nack(msg, false, false)` (requeue = false) $\rightarrow$ Message tự động bay vào `order.dlx` và chuyển tới **`Retry Queue`**.
    3. **Retry Queue (`order.retry`)**: Queue này **không hề có Consumer nào lắng nghe**. Nó được cấu hình `'x-message-ttl': 30000` (30 giây) và `'x-dead-letter-exchange': 'order.process'`.
    4. Sau 30 giây nằm im trong Retry Queue, message hết hạn (TTL Expired) $\rightarrow$ RabbitMQ tự động đẩy message ngược trở lại `Main Queue` để worker thử lại lần thứ hai.
    5. Trong header của message, kiểm tra mảng `x-death`. Nếu số lần chết $\ge 3$, đẩy sang **`Parking / Final DLQ`** để dev vào kiểm tra thủ công.

---

### Q4: Trong trường hợp nghiệp vụ yêu cầu message phải được xử lý đúng theo thứ tự thời gian (Strict Ordering) nhưng vẫn muốn scale nhiều Consumer, em giải quyết bài toán này ra sao?
* 💡 **Câu trả lời ăn điểm**:
  * **Vấn đề**: Nếu 1 Queue có 5 Consumer Pods đọc song song, message 1 đến trước nhưng Consumer 1 chạy chậm hơn Consumer 2 $\rightarrow$ Message 2 được xử lý xong trước message 1 $\rightarrow$ **Mất thứ tự!**
  * **Giải pháp: Sharded Queues với Consistent Hash Exchange**:
    * Sử dụng plugin `rabbitmq_consistent_hash_exchange`.
    * Phân chia thành $M$ queues con (ví dụ: `order.shard.0`, `order.shard.1`, `order.shard.2`...).
    * Mỗi shard queue chỉ gắn với **duy nhất 1 Consumer process**.
    * Khi publish message, gắn Routing Key là `customer_id` hoặc `tenant_id`. Consistent Hash Exchange sẽ băm ID này để đảm bảo rằng: **Mọi message của cùng một Customer/Tenant luôn luôn rơi vào cùng 1 Shard Queue duy nhất** $\rightarrow$ Đảm bảo đúng thứ tự 100% cho từng khách hàng, trong khi toàn bộ hệ thống vẫn scale ngang xử lý song song trên nhiều sharded queues.
