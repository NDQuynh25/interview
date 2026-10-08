# 🎯 CHUYÊN ĐỀ 01.2: NODE.JS – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu dành cho ứng viên Senior Node.js, khai thác trực tiếp các công nghệ và giải pháp thực chiến trong CV: **NestJS 11 + Fastify 5**, **Nx Monorepo (Webpack 5 + SWC)**, **Server-Sent Events (SSE)**, **GraphQL DataLoader**, và **Socket.IO scaling**.

---

## PHẦN 1: DỰ ÁN VISHOP – HIGH-THROUGHPUT NESTJS & ARCHITECTURE

### Case 1: Lựa chọn Fastify 5 thay vì Express trong NestJS 11
> **Trích dẫn CV**: *"Technologies: Node.js, NestJS 11, Fastify 5, TypeScript, PostgreSQL, Redis, BullMQ... Built high-throughput multi-tenant services using NestJS, Fastify, TypeScript, and Nx."*

#### Q1: Tại sao trong dự án ViShop em lại chọn Fastify adapter thay vì Express mặc định của NestJS? Bản chất kỹ thuật giúp Fastify đạt throughput cao hơn Express là gì?
* 💡 **Câu trả lời ăn điểm**:
  * **Hạn chế của Express**:
    * Express sử dụng thư viện routing cũ dựa trên RegEx duyệt mảng tuần tự $O(N)$.
    * Cơ chế serialize JSON của Express sử dụng `JSON.stringify` thuần của V8 mà không biết trước schema của payload $\rightarrow$ Chi phí reflection và parse runtime cao.
  * **Sức mạnh nội tại của Fastify**:
    1. **Radix Tree Routing (Find-My-Way)**: Fastify sử dụng cấu trúc dữ liệu cây Radix (Prefix Tree) để match route với độ phức tạp $O(K)$ (với $K$ là độ dài URL), độc lập hoàn toàn với số lượng API endpoints của hệ thống.
    2. **Schema-Based Serialization với `fast-json-stringify`**: Khi kết hợp với DTO/JSON Schema của NestJS, Fastify biên dịch trước (pre-compile) hàm serialize JSON. Nó serialize payload trực tiếp theo cấu trúc cố định, nhanh gấp **2–3 lần** so với `JSON.stringify` truyền thống.
    3. **Tối ưu hóa Object Allocation**: Fastify tái sử dụng các internal context objects để giảm áp lực dọn rác (Garbage Collection pressure) lên V8 engine.
  * **Hiệu quả thực tế**: Trong ViShop với các webhook sàn thương mại điện tử (TikTok Shop, Shopee) đẩy về hàng nghìn request/giây, Fastify giúp giảm độ trễ P99 khoảng 35% và chịu tải gấp 2.5 lần so với Express trên cùng cấu hình CPU/RAM.

---

### Case 2: Server-Sent Events (SSE) cho Real-Time Order & Inventory Tracking
> **Trích dẫn CV**: *"Developed Server-Sent Events (SSE) for real-time order, inventory, and synchronization status updates to connected clients."*

#### Q2: Tại sao em lại chọn Server-Sent Events (SSE) thay vì WebSocket để cập nhật trạng thái đơn hàng và tồn kho cho merchant? Khi scale ra nhiều Pods, em xử lý phân phối message và quản lý connection ra sao?
* 💡 **Câu trả lời ăn điểm**:
  * **Vì sao chọn SSE thay vì WebSocket**:
    * *Đặc thù nghiệp vụ*: Trạng thái đơn hàng và tồn kho WMS là luồng thông tin **một chiều từ Server đẩy xuống Client (Unidirectional)**. Merchant không cần gửi ngược luồng dữ liệu liên tục lên server qua cùng connection đó.
    * *Giao thức*: SSE chạy trên HTTP/1.1 hoặc HTTP/2 thuần túy, tương thích 100% với các API Gateway, Load Balancer (ALB/Nginx) và Firewall doanh nghiệp mà không cần cấu hình HTTP Upgrade hay lo ngại rớt kết nối WebSocket.
    * *Tính năng có sẵn*: SSE của trình duyệt (`EventSource`) tự động hỗ trợ **Auto-reconnection** và gửi header `Last-Event-ID` để server bù đắp các event bị bỏ lỡ trong thời gian mất mạng ngắn.
  * **Quản lý Connection & Scale Đa Pods**:
    * Sử dụng **Redis Pub/Sub** làm Message Broker trung gian giữa các Pod:
      * Khi có event cập nhật kho/đơn hàng, Pod xử lý nghiệp vụ publish event lên Redis channel: `merchant_updates:<tenant_id>`.
      * Tất cả các Pods NestJS đều subscribe vào Redis channel. Pod nào đang giữ kết nối SSE của merchant đó sẽ stream dữ liệu xuống client qua HTTP response chunked.
  * ⚠️ **Bẫy Connection Leak**: Luôn lắng nghe sự kiện `req.raw.on('close', ...)` để hủy subscription và giải phóng memory stream ngay khi merchant tắt tab trình duyệt, tránh rò rỉ RAM!

---

### Case 3: Cấu trúc Nx Monorepo (Webpack 5 + SWC) & Dependency Rules
> **Trích dẫn CV**: *"Architected a production-grade Nx Monorepo with 15 domain modules and 4 runnable microservices, adopting Domain-Driven Design (DDD) and CQRS... Nx Monorepo, Webpack 5 + SWC..."*

#### Q3: Khi dự án phình to lên 15 domain modules, em giải quyết bài toán thời gian build/test kéo dài và đảm bảo các module không import vòng (Circular Dependency) bằng cách nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Tối ưu tốc độ Compilation với SWC**:
    * Thay thế `tsc` (TypeScript Compiler) bằng **SWC (Rust-based compiler)** làm backend biên dịch cho Webpack 5.
    * SWC tận dụng đa nhân CPU thực tế của máy chủ CI/CD, tăng tốc độ compile code TypeScript lên gấp **10–15 lần**, giảm thời gian build từ 4 phút xuống còn ~25 giây.
  * **Kiểm soát ranh giới module với Nx Module Boundaries**:
    * Cấu hình ESLint rules của Nx (`@nx/enforce-module-boundaries`):
      * Gắn thẻ (tags) cho từng module: `scope:domain`, `scope:application`, `scope:infrastructure`.
      * Thiết lập luật phụ thuộc: Module `domain` tuyệt đối không được import bất kỳ file nào từ `infrastructure` hay `application`.
      * Ngăn chặn import chéo giữa các bounded context (ví dụ: `order-module` không được import trực tiếp file nội bộ của `inventory-module` mà phải qua API Contract).
  * **Nx Computation Caching**: Lưu trữ kết quả build và unit test của các module không bị thay đổi mã nguồn, giúp các lần chạy CI sau chỉ cần test đúng module bị ảnh hưởng (Affected builds).

---

## PHẦN 2: DỰ ÁN CRMAGENT – GRAPHQL, DATALOADER & WEBHOOKS

### Case 4: Triệt tiêu N+1 Query trong GraphQL với DataLoader
> **Trích dẫn CV**: *"Designed and developed backend microservices with NestJS and GraphQL, using DataLoader to eliminate N+1 queries across CRM entities."*

#### Q4: DataLoader hoạt động như thế nào ở tầng Event Loop của Node.js để gom các query đơn lẻ thành một batch query duy nhất?
* 💡 **Câu trả lời ăn điểm**:
  * **Cơ chế Batching dựa vào Microtask Queue**:
    * Khi các GraphQL resolvers thực thi đồng thời để resolve field `assignedUser` cho từng contact:
    * Thay vì gọi DB ngay lập tức, DataLoader lưu các `userId` cần fetch vào một mảng hàng đợi tạm thời (`queue`).
    * DataLoader lên lịch (schedule) hàm batch fetching thực thi ở **Microtask Phase** (thông qua `process.nextTick()` hoặc `Promise.resolve()`).
    * Khi toàn bộ Call Stack của tick hiện tại hoàn tất, Microtask Queue được kích hoạt. DataLoader gom toàn bộ 50 `userId` đã thu thập được thành một câu truy vấn duy nhất:
      ```sql
      SELECT * FROM users WHERE id IN ($1, $2, ..., $50);
      ```
    * Sau khi có kết quả, DataLoader phân phối lại từng object `User` cho các Promise tương ứng đang chờ của các resolver.
  * **Cơ chế Memoization Caching**:
    * Trong phạm vi một HTTP Request, DataLoader cache lại các key đã fetch. Nếu 10 contacts cùng thuộc về 1 `assignedUser`, DataLoader chỉ fetch user đó đúng 1 lần duy nhất.

---

## PHẦN 3: BỘ CÂU HỎI VỀ NODE.JS CORE & PRODUCTION INCIDENTS

### Q5: Khi nào bạn chọn `setImmediate()` thay vì `process.nextTick()`? Điều gì xảy ra nếu bạn gọi đệ quy `process.nextTick()`?
* 💡 **Câu trả lời ăn điểm**:
  * **Sự khác biệt**:
    * `process.nextTick()` không thuộc bất kỳ phase nào của Libuv Event Loop. Nó thuộc **Microtask Queue** và được thực thi **ngay lập tức khi Call Stack hiện tại vừa rỗng**, trước khi Event Loop tiến sang phase tiếp theo.
    * `setImmediate()` được xếp vào phase **Check** của Event Loop (chạy sau phase Poll I/O).
  * **Hiểm họa Starvation**:
    * Nếu gọi đệ quy `process.nextTick()`, Node.js sẽ bị kẹt vĩnh viễn trong việc xử lý microtask. Toàn bộ các phase sau của Event Loop (Timers, I/O callbacks, Poll socket) đều bị tê liệt $\rightarrow$ **Server hoàn toàn không thể nhận hoặc phản hồi bất kỳ HTTP request nào khác!**
  * **Quy tắc chọn**: Dùng `setImmediate()` khi muốn nhường quyền điều khiển (yield) cho Event Loop xử lý I/O và các request khác trước khi tiếp tục công việc của mình.

---

### Q6: Làm sao để chẩn đoán và khắc phục Memory Leak trong ứng dụng Node.js đang chạy trên Kubernetes mà không gây gián đoạn traffic người dùng?
* 💡 **Câu trả lời ăn điểm**:
  * **Quy trình 4 bước chuẩn Production**:
    1. **Cô lập Pod**: Sử dụng lệnh `kubectl label pod <pod-name> app=isolated` hoặc gỡ Pod ra khỏi Service Endpoints để ngừng tiếp nhận traffic người dùng mới, tránh gây ảnh hưởng trải nghiệm khách hàng.
    2. **Chụp Heap Snapshot qua Port-Forward**:
       * Mở port debug qua `kubectl port-forward <pod-name> 9229:9229`.
       * Hoặc kích hoạt endpoint nội bộ được bảo vệ (Internal Admin API) sử dụng module `v8`:
         ```typescript
         import v8 from 'v8';
         v8.writeHeapSnapshot('/tmp/heap-dump-1.heapsnapshot');
         ```
       * Đợi 5 phút và chụp snapshot thứ hai: `heap-dump-2.heapsnapshot`.
    3. **Phân tích so sánh (Comparison View) trên Chrome DevTools**:
       * Mở Chrome DevTools $\rightarrow$ Memory Tab $\rightarrow$ Load cả 2 file snapshot.
       * Chuyển sang chế độ **Comparison View** để tìm các Object có delta tăng đột biến: thường là các mảng tích tụ (Arrays), `Closure Context` giữ tham chiếu ngoài ý muốn, hoặc các listener của `EventEmitter` quên gọi `removeListener()`.
    4. **Kiểm tra Retainer Tree**: Lần theo cây tham chiếu để xác định biến toàn cục hoặc service singleton nào đang ngăn cản Garbage Collector dọn dẹp các object đó.

---

### Q7: Tại sao `UV_THREADPOOL_SIZE` mặc định là 4? Trong trường hợp nào bạn bắt buộc phải nâng con số này lên?
* 💡 **Câu trả lời ăn điểm**:
  * **Vì sao là 4**: Libuv đặt mặc định 4 threads để tối ưu tài nguyên CPU trên các máy chủ đa nhân phổ thông, tránh chi phí chuyển ngữ cảnh (Context Switching) khi có quá nhiều thread cạnh tranh số core CPU hạn chế.
  * **Khi nào bắt buộc phải nâng lên (ví dụ 32 hoặc 64)**:
    1. **DNS Resolution nghẽn cổ chai**: Ứng dụng gọi hàng nghìn API bên ngoài (Third-party integrations như TikTok Shop, VNPay, Zalo API). Mặc định `dns.lookup` sử dụng Thread Pool (chỉ có 4 threads). Khi có hàng trăm request đồng thời, 4 threads này bị nghẽn $\rightarrow$ Kéo theo tất cả các tác vụ File System I/O (`fs`) bị xếp hàng chờ.
    2. **Mã hóa mật khẩu hoặc tính toán Crypto dày đặc**: Ứng dụng xử lý luồng đăng nhập lớn sử dụng `bcrypt` hoặc `crypto.pbkdf2`.
  * **Giải pháp thay thế ưu việt cho DNS**: Thay vì tăng threadpool quá cao, có thể cấu hình Node.js sử dụng `dns.resolve` (chạy qua thư viện c-ares bất đồng bộ, không hề tiêu tốn Worker Thread).
