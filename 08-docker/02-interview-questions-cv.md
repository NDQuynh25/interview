# 🎯 CHUYÊN ĐỀ 08.2: DOCKER – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu về Containerization kết hợp trực tiếp với kinh nghiệm trong CV: **ViShop Nx Monorepo (15 Domain Modules & Docker Compose Local Dev)**, **Multimodal RAG Platform (Tối ưu Dockerfile với uv)**, **Multi-stage Build giảm 90% Image Size**, **OOMKilled Troubleshooting**, và **PID 1 Graceful Shutdown**.

---

## PHẦN 1: DỰ ÁN VISHOP & MULTIMODAL RAG – CONTAINERIZATION THỰC CHIẾN

### Case 1: Tối ưu Multi-Stage Dockerfile cho ViShop Nx Monorepo
> **Trích dẫn CV**: *"Architected a production-grade Nx Monorepo with 15 domain modules and 4 runnable microservices, adopting Domain-Driven Design (DDD) and CQRS."*

#### Q1: Trong một Nx Monorepo có 15 domain modules dùng chung, nếu viết Dockerfile thông thường thì mỗi lần build 1 microservice lẻ, Docker sẽ phải kéo toàn bộ mã nguồn của cả Monorepo khiến image phình to tới gần 2GB và mất cache liên tục. Em đã giải quyết bài toán này như thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Vấn đề cốt lõi của Monorepo**: Một microservice (ví dụ `order-service`) phụ thuộc vào các thư viện nội bộ (`libs/domain/order`, `libs/shared/kernel`). Nếu chỉ copy thư mục của service thì thiếu dependencies; nếu `COPY . .` toàn bộ monorepo thì bất kỳ commit nào vào một service khác cũng làm vỡ Docker Layer Cache của service này.
  * **Giải pháp 3 bước đã áp dụng**:
    1. **Tận dụng lệnh `nx generate @nx/node:dockerfile` hoặc `prune`**: Sử dụng công cụ cắt tỉa của Nx để chỉ trích xuất đúng cây phụ thuộc (Dependency Graph) của service cần đóng gói.
    2. **Multi-Stage Build 3 tầng**:
       * **Stage 1 (Deps)**: Chỉ copy `package.json`, `package-lock.json` và chạy `npm ci`. Tầng này được cache 100% nếu không thêm thư viện mới.
       * **Stage 2 (Builder)**: Copy mã nguồn các `libs` liên quan và chạy `npx nx build order-service --prod`. Sau đó chạy `npm prune --production` để vứt bỏ `typescript`, `@types/*`, `jest`.
       * **Stage 3 (Runner)**: Sử dụng Base Image siêu nhẹ `node:20-alpine` hoặc `gcr.io/distroless/nodejs20-debian12`. Chỉ copy thư mục `dist/apps/order-service` và `node_modules` đã tỉa gọn.
    3. **Kết quả**:
       * Kích thước Image giảm từ **1.8GB xuống chỉ còn ~115MB**.
       * Thời gian CI/CD build khi chỉ sửa code logic giảm từ **8 phút xuống dưới 45 giây** nhờ giữ nguyên Layer Cache của node_modules.

---

### Case 2: Thiết kế Docker Compose cho Môi trường Local Dev 15 Modules
> **Trích dẫn CV**: *"Designed a unified developer experience with Docker Compose orchestrating polyglot databases, message brokers, and background workers."*

#### Q2: Khi chạy cụm ViShop ở local gồm PostgreSQL, Redis, RabbitMQ, Elasticsearch và 4 microservices, làm sao em tránh được tình trạng race condition khi các service khởi động trước khi Database kịp sẵn sàng? Em cấu hình volume mount để hot-reload code như thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Xử lý Race Condition khi khởi động (Healthcheck + Depends_on)**:
    * Mặc định, chỉ thị `depends_on: [postgres]` trong Docker Compose chỉ đợi container PostgreSQL được *tạo ra (started)*, chứ **chưa chắc DB engine đã sẵn sàng nhận kết nối socket TCP**. Service Node.js khởi động ngay lập tức sẽ bị lỗi `ECONNREFUSED` và crash loop.
    * Giải pháp: Sử dụng **Container Healthcheck** kết hợp điều kiện `service_healthy`:
      ```yaml
      services:
        postgres:
          image: postgres:16-alpine
          environment:
            POSTGRES_DB: vishop
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: secretpassword
          healthcheck:
            test: ["CMD-SHELL", "pg_isready -U postgres -d vishop"]
            interval: 5s
            timeout: 5s
            retries: 5
            start_period: 10s

        order-service:
          build:
            context: .
            dockerfile: apps/order-service/Dockerfile.dev
          depends_on:
            postgres:
              condition: service_healthy # Chỉ chạy khi Postgres thật sự sẵn sàng!
            rabbitmq:
              condition: service_healthy
      ```
  * **Cấu hình Hot-Reloading với Volume Mount (Bẫy `node_modules`)**:
    * Khi mount toàn bộ thư mục code từ máy Host vào container (`./apps/order-service:/app`), thư mục `node_modules` ở Host có thể ghi đè lên `node_modules` đã cài trong container (gây lỗi binary compatibility giữa Windows/macOS và Linux Alpine).
    * Giải pháp: Khai báo một **Anonymous Volume** riêng cho `/app/node_modules`:
      ```yaml
      volumes:
        - .:/app
        - /app/node_modules # Giữ nguyên node_modules được build bên trong Linux container
      ```

---

### Case 3: Tối ưu Dockerfile cho Python Multimodal RAG Platform
> **Trích dẫn CV**: *"Architected a scalable uv Python Monorepo adopting Domain-Driven Design, decoupling shared domain contracts, document processing pipeline, core RAG engine..."*

#### Q3: Trong dự án Multimodal RAG Platform viết bằng Python, em sử dụng package manager `uv`. Làm sao em tối ưu hóa Dockerfile để tăng tốc độ build gấp nhiều lần và tận dụng cache của `uv`?
* 💡 **Câu trả lời ăn điểm**:
  * **Bản chất**: Python pip truyền thống cài đặt rất chậm vì phải resolve dependency tuần tự và compile wheels. `uv` (viết bằng Rust) cho tốc độ resolve và install nhanh gấp 10-100 lần.
  * **Kỹ thuật BuildKit Cache Mount**:
    Sử dụng cờ `--mount=type=cache` của Docker BuildKit để lưu giữ thư mục cache của `uv` xuyên suốt các lần build trên CI/CD runner:
    ```dockerfile
    # syntax=docker/dockerfile:1.4
    FROM python:3.11-slim AS builder

    WORKDIR /app
    # Cài đặt uv nhị phân cực nhanh
    COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

    # 1. Chỉ copy lockfile để tận dụng layer cache
    COPY pyproject.toml uv.lock ./

    # 2. Đồng bộ thư viện vào virtualenv sử dụng Cache Mount
    RUN --mount=type=cache,target=/root/.cache/uv \
        uv sync --frozen --no-install-project --no-dev

    # 3. Copy mã nguồn và hoàn tất cài đặt
    COPY . .
    RUN --mount=type=cache,target=/root/.cache/uv \
        uv sync --frozen --no-dev

    # ================= Runtime Stage =================
    FROM python:3.11-slim AS runner
    WORKDIR /app

    # Copy duy nhất Virtual Environment đã đóng gói sẵn
    COPY --from=builder /app/.venv /app/.venv
    COPY --from=builder /app/src /app/src

    ENV PATH="/app/.venv/bin:$PATH"
    USER 10001
    CMD ["python", "-m", "rag_engine.main"]
    ```
  * **Hiệu quả**: Image size giảm mạnh, không chứa công cụ build gcc/cargo, và thời gian build khi thêm code chỉ mất **vài giây**.

---

## PHẦN 2: BỘ CÂU HỎI HÓC BÚA VỀ LINUX INTERNALS, SECURITY & TROUBLESHOOTING

### Q4: Trên Production, một container NestJS đột ngột biến mất và exit với mã lỗi `Exit Code 137`. Em điều tra nguyên nhân như thế nào và làm sao để phòng chống?
* 💡 **Câu trả lời ăn điểm**:
  * **Nhận diện lỗi**:
    * Trong Linux, mã exit code của tiến trình bị hủy bởi tín hiệu (Signal) bằng `128 + Signal_Number`.
    * `137 = 128 + 9 (SIGKILL)`. Điều này chứng tỏ container bị ép dừng bằng `SIGKILL` mà không có cơ hội xử lý graceful shutdown.
    * Nguyên nhân phổ biến nhất: **OOMKilled (Out Of Memory Killer)** do Linux Kernel kích hoạt khi cgroup memory của container vượt quá giới hạn.
  * **Các bước điều tra thực tế**:
    1. Kiểm tra trạng thái container: `docker inspect <container_id> --format='{{json .State}}'`. Quan sát trường `"OOMKilled": true`.
    2. Kiểm tra Kernel log trên máy chủ: `dmesg -T | grep -i oom` hoặc `journalctl -k`. Log sẽ chỉ rõ process nào bị kill, RSS memory lúc đó là bao nhiêu và cgroup nào vi phạm.
  * **Giải pháp căn cơ**:
    1. **Quy tắc phân bổ Heap RAM cho Node.js**:
       * Mặc định V8 engine tự nhận diện RAM của máy chủ Host thay vì RAM của Container cgroup (ở các bản Node.js cũ). Nếu container giới hạn 1GB mà Node.js tưởng máy có 16GB, nó sẽ trì hoãn Garbage Collection cho đến khi vượt 1GB $\rightarrow$ Bị Kernel OOMKilled ngay lập tức.
       * Giải pháp: Cấu hình biến môi trường `NODE_OPTIONS="--max-old-space-size=800"` (đặt bằng **75-80%** giới hạn RAM của container). 20-25% còn lại dành cho buffer, network socket và native C++ bindings.
    2. **Giám sát Memory Leak**: Dùng clinic.js hoặc Grafana/Prometheus theo dõi biểu đồ `container_memory_working_set_bytes`. Nếu đường biểu diễn dốc đều theo thời gian không hạ thì chắc chắn do Memory Leak (ví dụ: giữ mảng cache toàn cục không giải phóng).

---

### Q5: Khi gọi `docker stop`, container NestJS của em luôn mất đúng 10 giây mới tắt. Tại sao lại xảy ra hiện tượng này và em khắc phục như thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Bản chất vấn đề**:
    * Khi gọi `docker stop`, Docker gửi tín hiệu **`SIGTERM`** đến tiến trình có **`PID 1`** bên trong container và khởi động đồng hồ đếm ngược 10 giây.
    * Nếu trong Dockerfile sử dụng dạng Shell form: `CMD npm start` hoặc `CMD node main.js` (không có dấu ngoặc vuông `[]`), Docker sẽ bọc lệnh vào shell: `/bin/sh -c "npm start"`. Lúc này, `/bin/sh` trở thành **PID 1**, còn Node.js là tiến trình con.
    * `/bin/sh` mặc định **không chuyển tiếp (forward) tín hiệu SIGTERM** cho tiến trình Node.js con $\rightarrow$ Node.js không hề biết container sắp bị tắt.
    * Hết 10 giây timeout, Docker mất kiên nhẫn và gửi **`SIGKILL`** để ép dừng container $\rightarrow$ Các request dở dang bị đứt, transaction DB không kịp rollback, message RabbitMQ chưa kịp nack.
  * **Cách khắc phục triệt để**:
    1. **Luôn dùng Exec form (mảng JSON)**: `CMD ["node", "dist/main.js"]`.
    2. **Sử dụng `tini` làm Init Process (PID 1)**:
       * Tini nhận `SIGTERM` và chuyển tiếp ngay lập tức cho Node.js.
       * Đồng thời Tini chịu trách nhiệm dọn dẹp các Zombie Process mồ côi.
       ```dockerfile
       RUN apk add --no-cache tini
       ENTRYPOINT ["/sbin/tini", "--"]
       CMD ["node", "dist/main.js"]
       ```
    3. **Lắng nghe tín hiệu trong mã nguồn NestJS**:
       ```typescript
       // main.ts
       const app = await NestFactory.create(AppModule);
       // Kích hoạt lifecycle hooks để NestJS đóng DB connections, RabbitMQ channels
       app.enableShutdownHooks();
       await app.listen(3000);
       ```

---

### Q6: Kể tên 4 nguyên tắc bảo mật (Security Hardening) bắt buộc phải thực hiện trên Docker Image trước khi đưa lên Production?
* 💡 **Câu trả lời ăn điểm**:
  1. **Tuyệt đối không chạy dưới quyền `root` (Non-Root User)**:
     * Khai báo `USER node` hoặc tạo riêng UID `10001` không có quyền login shell.
     * Ngăn chặn hoàn toàn nguy cơ kẻ tấn công khai thác lỗ hổng ứng dụng (RCE) để leo thang đặc quyền thoát khỏi container (Container Breakout).
  2. **Hệ thống tệp tin chỉ đọc (`--read-only` root filesystem)**:
     * Cấu hình container chạy với cờ `read_only: true`. Kẻ tấn công không thể tải mã độc hoặc sửa đổi file nhị phân trong `/bin`, `/usr`.
     * Các thư mục cần ghi tạm (như `/tmp`) được mount vào `tmpfs` trong RAM.
  3. **Loại bỏ đặc quyền Linux (`Drop Capabilities`)**:
     * Mặc định container nhận một số capabilities của root. Trên production cần cấu hình:
       ```yaml
       security_opt:
         - no-new-privileges:true
       cap_drop:
         - ALL
       cap_add:
         - NET_BIND_SERVICE # Chỉ cấp đúng quyền mở port nếu cần
       ```
  4. **Sử dụng Distroless / Minimal Base Image & Quét lỗ hổng (Vulnerability Scanning)**:
     * Dùng image `distroless` (không chứa shell `/bin/sh`, không có `curl`, `apt`, `npm`). Nếu hacker inject được script, chúng cũng không có shell để thực thi lệnh.
     * Tích hợp công cụ quét lỗ hổng tĩnh (**Trivy** hoặc **Docker Scout**) trong CI/CD pipeline để tự động chặn build nếu phát hiện CVE mức độ `CRITICAL` hoặc `HIGH`.
