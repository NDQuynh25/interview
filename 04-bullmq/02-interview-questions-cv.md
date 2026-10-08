# 🎯 CHUYÊN ĐỀ 04.2: BULLMQ – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu khai thác trực tiếp kinh nghiệm sử dụng BullMQ trong CV: **ViShop (Transactional Outbox Event Relay)**, **CRMAgent (Omnichannel Webhook Ingestion)**, **JIHO (Platform Data Synchronization)**, và **Stalled Jobs Mitigation**.

---

## PHẦN 1: DỰ ÁN VISHOP & CRMAGENT – PRODUCTION JOB QUEUES

### Case 1: Xử lý Stalled Jobs & Bảo vệ Idempotency trong ViShop
> **Trích dẫn CV**: *"Implemented the Transactional Outbox Pattern with BullMQ and Redis for reliable domain event processing... maintain inventory consistency..."*

#### Q1: Hiện tượng "Stalled Job" trong BullMQ xảy ra như thế nào? Trong hệ thống WMS của ViShop, nếu một job trừ kho hoặc gửi đơn hàng bị stalled và gắp lại cho Worker khác chạy, em phòng tránh double processing ra sao?
* 💡 **Câu trả lời ăn điểm**:
  * **Cơ chế phát hiện Stalled Job**:
    * Khi Worker nhận job, nó chuyển job vào `active` và đặt một Lock trên Redis với thời hạn `lockDuration` (mặc định 30s).
    * Trong quá trình chạy, Worker định kỳ gửi heartbeat để gia hạn lock (`lockRenewTime: 15s`).
    * Nếu Worker Pod bị sập đột ngột (Kubernetes OOMKilled do rò rỉ RAM hoặc node restart), lock trên Redis sẽ hết hạn.
    * Một tiến trình `checkStalledJobs` chạy trên các worker khác sẽ phát hiện lock hết hạn, tăng `stalledCounter`, và chuyển job ngược lại danh sách `wait` để chạy lại.
  * **Cách bảo vệ Idempotency trong ViShop**:
    1. **Mã khóa Idempotency Key**: Đặt `jobId` tương ứng với mã nghiệp vụ: `jobId = "outbox_event:" + event.id`. BullMQ đảm bảo không thể chèn 2 job có cùng `jobId` vào queue.
    2. **Kiểm tra trạng thái tại Database**: Trong worker handler, trước khi xử lý trừ kho hoặc gọi API bên thứ ba, luôn kiểm tra trạng thái nghiệp vụ trong PostgreSQL:
       ```typescript
       // Nếu event đã được đánh dấu là PROCESSED trong DB -> Return ngay lập tức!
       const event = await outboxRepo.findById(job.data.eventId);
       if (event.status === 'PROCESSED') {
         return { skipped: true, reason: 'Already processed' };
       }
       ```

---

### Case 2: Xử lý Webhook Facebook Messenger & Zalo OA Tải cao trong CRMAgent
> **Trích dẫn CV**: *"Built an omnichannel messaging hub integrating Facebook Messenger and Zalo OA/ZNS, processing webhooks asynchronously with BullMQ... OpenAI/Anthropic APIs..."*

#### Q2: Khi có chiến dịch Marketing bắn tin nhắn hàng loạt từ Zalo/Facebook đổ về webhook, làm sao em dùng BullMQ để bảo vệ hệ thống không bị sập và không vượt quá Rate Limit của AI APIs (OpenAI/Anthropic)?
* 💡 **Câu trả lời ăn điểm**:
  * **Phân tách Ingestion và Processing**:
    * Webhook Controller chỉ làm đúng 1 nhiệm vụ: Validate signature và đẩy raw payload vào BullMQ (`messages-queue`) với thời gian phản hồi `< 20ms`, sau đó trả về ngay HTTP `200 OK` cho Facebook/Zalo để tránh bị webhook timeout.
  * **Rate Limiting tại Worker với BullMQ Limiter**:
    * Các AI API (OpenAI/Anthropic) đều có giới hạn RPM (Requests Per Minute) hoặc TPM (Tokens Per Minute).
    * Cấu hình Worker với tính năng Rate Limiting tích hợp sẵn:
      ```typescript
      export const aiWorker = new Worker('ai-intent-classification', processAiTask, {
        connection: redisConnection,
        limiter: {
          max: 50,      // Tối đa 50 requests
          duration: 1000, // Trong vòng 1 giây
        },
      });
      ```
    * Khi chạm ngưỡng 50 req/s, BullMQ tự động giữ các job còn lại trong trạng thái `delayed` và giải phóng dần dần mà không làm rớt tin nhắn của khách hàng.

---

### Case 3: Đồng bộ dữ liệu bất đồng bộ giữa E-Learning & E-Book trong JIHO
> **Trích dẫn CV**: *"Built asynchronous synchronization between E-Learning and E-Book platforms using BullMQ and Redis with retry mechanisms."*

#### Q3: Khi đồng bộ dữ liệu giữa 2 nền tảng qua mạng, nếu service đích bị sập trong 30 phút, em thiết kế cơ chế Retry và Dead-Letter Queue như thế nào để không làm nghẽn queue?
* 💡 **Câu trả lời ăn điểm**:
  * **Chiến lược Exponential Backoff**:
    * Cấu hình `attempts: 8` và `backoff: { type: 'exponential', delay: 5000 }`.
    * Khoảng cách giữa các lần thử lại sẽ tăng dần: 5s $\rightarrow$ 10s $\rightarrow$ 20s $\rightarrow$ 40s $\rightarrow$ 80s... tránh việc liên tục spam làm quá tải service đích đang bị lỗi.
  * **Dead-Letter Queue (DLQ) & Alerting**:
    * Khi job vượt quá 8 lần retry, BullMQ chuyển job sang trạng thái `failed`.
    * Lắng nghe sự kiện `worker.on('failed')`:
      * Ghi nhận event thất bại vào bảng `sync_failed_audit_log` của PostgreSQL.
      * Bắn cảnh báo về Slack / Telegram của đội trực vận hành (DevOps/Backend).
      * Sau khi service đích phục hồi, sử dụng Bull-Board UI hoặc script admin gọi `queue.retryJobs({ count: 100 })` để chạy lại toàn bộ các job lỗi chỉ với một lệnh.

---

## PHẦN 2: BỘ CÂU HỎI VỀ BULLMQ INTERNALS & TUNING

### Q4: Làm thế nào BullMQ hiện thực được tính năng Delayed Job (chạy sau $X$ giây/phút) trên Redis?
* 💡 **Câu trả lời ăn điểm**:
  * BullMQ lưu các delayed jobs vào một **Sorted Set (ZSet)** có tên `bull:<queue>:delayed`.
  * **Score** của mỗi job trong ZSet chính là mốc timestamp trong tương lai khi job được phép thực thi (`Date.now() + delayMs`).
  * Khi worker hoặc scheduler chạy, nó thực thi một **Lua Script** gọi lệnh `ZREMRANGEBYSCORE key 0 <current_timestamp>` để bốc nguyên tử các job có score $\le$ thời gian hiện tại và đẩy chúng vào danh sách `wait` (List/Stream) để các worker bắt đầu nhận việc.

---

### Q5: Khi nào nên dùng BullMQ và khi nào bắt buộc phải chuyển sang RabbitMQ hoặc Apache Kafka?
* 💡 **Câu trả lời ăn điểm**:

| Tiêu chí | BullMQ (Redis-based) | RabbitMQ (AMQP) | Apache Kafka |
| :--- | :--- | :--- | :--- |
| **Bản chất** | **Task / Job Queue** | **Enterprise Message Broker** | **Distributed Event Streaming** |
| **Định tuyến** | Đơn giản theo tên Queue. | Cực mạnh với Topic/Fanout Exchange. | Partition-based theo Message Key. |
| **Phù hợp nhất** | Background jobs nội bộ service, Cron jobs, Delay jobs, Rate limiting. | Giao tiếp giữa các microservice không đồng nhất, Transactional Outbox phức tạp. | Xử lý Big Data, CDC (Change Data Capture), Event Sourcing triệu events/giây. |
| **Lưu trữ** | In-Memory (Redis RAM). Dữ liệu hoàn tất cần dọn dẹp. | Lưu trên đĩa/RAM, xóa message ngay sau khi Consumer Ack. | Lưu trên đĩa (Log Append-Only), giữ message nhiều ngày cho nhiều consumer đọc lại. |
