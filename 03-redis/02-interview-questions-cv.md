# 🎯 CHUYÊN ĐỀ 03.2: REDIS – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu khai thác trực tiếp các bài toán Redis trong CV: **Distributed Leader Election**, **TikTok Shop Idempotency**, **Sliding Window Rate Limiter**, **Hot Key & Big Key Mitigation**, và **Redlock Critique**.

---

## PHẦN 1: DỰ ÁN VISHOP – REDIS CONCURRENCY & LEADER ELECTION

### Case 1: Distributed Leader Election ngăn trùng Scheduled Jobs đa Pods
> **Trích dẫn CV**: *"Built reliable asynchronous processing using Transactional Outbox, BullMQ, Redis, and distributed leader election to prevent duplicate scheduled jobs across multiple pods."*

#### Q1: Tại sao trong kiến trúc chạy 5–10 Pods NestJS trên Kubernetes, cron job lại bị chạy trùng lặp? Em implement Distributed Leader Election bằng Redis như thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Vấn đề trên K8s**:
    * Khi cấu hình `@Cron('0 0 * * *')` trong NestJS, nếu ứng dụng scale ra 5 Pods, cả 5 Pods sẽ cùng kích hoạt cron job tại đúng 00:00:00 $\rightarrow$ Dẫn đến gửi trùng email, đồng bộ hóa đơn trùng lặp, lãng phí tài nguyên và sai lệch số liệu.
  * **Giải pháp Distributed Leader Election với Redis `SET NX PX`**:
    * Pod nào giành được quyền làm **Leader** tại thời điểm đó mới được phép thực thi cron job:
    ```typescript
    async function runScheduledJobWithLeaderElection(redis: Redis, jobKey: string, jobFn: () => Promise<void>) {
      const lockKey = `cron_leader:${jobKey}`;
      const leaderId = crypto.randomUUID();
      const leaseTimeMs = 60000; // 60 giây TTL

      // Cố gắng giành quyền Leader
      const acquired = await redis.set(lockKey, leaderId, 'NX', 'PX', leaseTimeMs);
      if (!acquired) {
        // Pod này là Follower, bỏ qua không chạy
        return;
      }

      try {
        await jobFn();
      } finally {
        // Giải phóng lock an toàn bằng Lua Script
        const releaseScript = `
          if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
          else
            return 0
          end
        `;
        await redis.eval(releaseScript, 1, lockKey, leaderId);
      }
    }
    ```

---

### Case 2: Redis-based Idempotency cho Webhook TikTok Shop
> **Trích dẫn CV**: *"Developed bi-directional TikTok Shop Open API integrations handling OAuth2 token lifecycles, webhook event ingestion, rate limiting, and Redis-based idempotency to prevent duplicate order processing."*

#### Q2: TikTok Shop có thể bắn 2 webhook cập nhật đơn hàng cách nhau 50ms. Em thiết kế cơ chế Idempotency bằng Redis như thế nào để ngăn chặn xử lý 2 lần?
* 💡 **Câu trả lời ăn điểm**:
  * **Luồng xử lý chuẩn mực**:
    1. Trích xuất Unique Event ID từ Webhook Payload: `event_id = payload.event_id || payload.order_id + '_' + payload.update_time`.
    2. Sử dụng Redis `SET idemp:<event_id> "PROCESSING" NX EX 120`:
       * Nếu lệnh trả về `null` (key đã tồn tại) $\rightarrow$ Webhook này đang được xử lý hoặc đã xử lý bởi một luồng khác $\rightarrow$ Trả về ngay lập tức HTTP `200 OK` để TikTok Shop không retry.
       * Nếu trả về `'OK'` $\rightarrow$ Đây là request đầu tiên, tiếp tục xử lý nghiệp vụ.
    3. Sau khi ghi nhận đơn hàng vào Database thành công:
       * Cập nhật Redis key thành `COMPLETED` với TTL hợp lý (ví dụ: 24 giờ).
    4. Nếu xử lý thất bại do lỗi tạm thời (Transient Error):
       * Xóa key khỏi Redis để cho phép webhook retry tiếp theo được thực thi lại.

---

### Case 3: Sliding Window Rate Limiter cho Third-Party API
> **Trích dẫn CV**: *"Open API integrations handling OAuth2 token lifecycles, webhook event ingestion, rate limiting..."*

#### Q3: Thuật toán Sliding Window Log Rate Limiter được cài đặt bằng Sorted Set (ZSet) trong Redis như thế nào để đảm bảo tính chính xác từng giây?
* 💡 **Câu trả lời ăn điểm**:
  * **Vì sao không dùng Fixed Window (Bộ đếm cố định)**:
    * Fixed Window bị lỗi "Traffic Burst" tại thời điểm giao thoa giữa 2 phút (ví dụ: 100 requests vào giây 59 và 100 requests vào giây 01 của phút sau $\rightarrow$ 200 requests chỉ trong 2 giây).
  * **Thuật toán Sliding Window Log bằng ZSet**:
    * Sử dụng ZSet với `score = timestamp hiện tại (milliseconds)`, `member = timestamp + random_nonce`.
    * **Thực thi nguyên tử qua Lua Script**:
    ```lua
    local key = KEYS[1]
    local now = tonumber(ARGV[1])
    local window_size = tonumber(ARGV[2]) -- 60000 ms
    local limit = tonumber(ARGV[3])       -- 100 requests

    -- 1. Xóa các request đã ra khỏi cửa sổ trượt (quá 60 giây trước)
    redis.call('ZREMRANGEBYSCORE', key, '-inf', now - window_size)

    -- 2. Đếm số lượng request còn lại trong cửa sổ
    local current_count = redis.call('ZCARD', key)

    -- 3. Kiểm tra giới hạn
    if current_count < limit then
        redis.call('ZADD', key, now, now .. '_' .. math.random(1000, 9999))
        redis.call('PEXPIRE', key, window_size)
        return 1 -- Cho phép request
    else
        return 0 -- Bị chặn (Rate limit exceeded)
    end
    ```

---

## PHẦN 2: BỘ CÂU HỎI VỀ REDIS CLUSTERING & SENIOR ARCHITECTURE

### Q4: Thuật toán Redlock của Salvatore Sanfilippo hoạt động ra sao? Martin Kleppmann đã chỉ ra lỗ hổng gì của nó và giải pháp thay thế là gì?
* 💡 **Câu trả lời ăn điểm**:
  * **Nguyên lý Redlock**: Triển khai trên $N$ master nodes độc lập (thường là 5 nodes, không dùng replication). Client cố gắng lấy lock trên tất cả các node. Nếu lấy thành công trên đa số node ($\ge \lfloor N/2 \rfloor + 1$) trong khoảng thời gian nhỏ hơn lock validity time $\rightarrow$ Lock thành công.
  * **Lỗ hổng do Martin Kleppmann chỉ ra**:
    1. **NTP Clock Drift (Trôi đồng hồ mạng)**: Nếu một node Redis bị trôi thời gian nhanh hơn các node khác, key lock trên node đó sẽ hết hạn sớm, cho phép client khác cướp lock.
    2. **Process Pause (Dừng tiến trình)**: Client 1 lấy được lock với TTL 30s. Ngay sau đó, Client 1 gặp một đợt Garbage Collection Pause (Stop-The-World) hoặc VM migration kéo dài 35 giây. Sau khi tỉnh lại, Client 1 vẫn đinh ninh mình đang giữ lock và thực hiện ghi đè DB, trong khi Client 2 đã lấy được lock mới $\rightarrow$ Vi phạm tính Mutual Exclusion!
  * **Giải pháp chuẩn xác**:
    * Trong các bài toán tài chính/kho hàng đòi hỏi tính đúng đắn tuyệt đối, sử dụng **Fencing Token** (một số thứ tự tăng dần kèm lock). Khi ghi vào Database, DB sẽ từ chối nếu token của Client 1 nhỏ hơn token cao nhất hiện tại.
    * Hoặc sử dụng chính **PostgreSQL Advisory Lock (`pg_advisory_xact_lock`)** để ràng buộc trực tiếp cùng ACID transaction của database.

---

### Q5: Khi Redis Cluster chia 16,384 Hash Slots, làm thế nào để thực hiện các thao tác Multi-Key (`MGET`, `MSET`, Lua Script) mà không bị lỗi `CROSSSLOT Keys in request don't hash to the same slot`?
* 💡 **Câu trả lời ăn điểm**:
  * **Bản chất Hash Slot**: Redis Cluster phân bổ 16,384 hash slots dựa trên công thức `CRC16(key) % 16384`. Hai key ngẫu nhiên thường nằm trên 2 slots khác nhau (có thể nằm ở 2 nodes vật lý khác nhau), Redis không hỗ trợ atomic multi-key giữa các node khác nhau.
  * **Giải pháp: Hash Tags `{...}`**:
    * Nếu key chứa cặp ngoặc nhọn `{...}`, Redis Cluster chỉ băm phần chuỗi nằm bên trong dấu ngoặc nhọn để tính toán slot.
    * *Áp dụng vào Multi-tenant E-Commerce (ViShop)*:
      * `user:{tenant_123}:profile`
      * `orders:{tenant_123}:recent`
      * Cả hai key trên đều được tính hash slot dựa trên chuỗi `tenant_123` $\rightarrow$ Đảm bảo **100% nằm cùng 1 Hash Slot trên cùng 1 Node**, cho phép chạy Transaction, Pipeline, và Lua Script an toàn.
