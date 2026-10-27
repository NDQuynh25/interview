# 🎯 CHUYÊN ĐỀ 07.2: DOMAIN-DRIVEN DESIGN – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu về DDD kết hợp trực tiếp với kinh nghiệm trong CV: **ViShop Nx Monorepo 15 Domain Modules**, **Multimodal RAG Platform (Domain Contracts Decoupling)**, **Anti-Corruption Layer (ACL) cho Sàn TMĐT**, và **Transaction Boundary Invariants**.

---

## PHẦN 1: DỰ ÁN VISHOP & MULTIMODAL RAG – KIẾN TRÚC DDD THỰC CHIẾN

### Case 1: Phân ranh giới Bounded Contexts trong ViShop Nx Monorepo
> **Trích dẫn CV**: *"Architected a production-grade Nx Monorepo with 15 domain modules and 4 runnable microservices, adopting Domain-Driven Design (DDD) and CQRS."*

#### Q1: Khi thiết kế hệ thống ViShop với 15 domain modules, em đã phân chia các Bounded Contexts như thế nào? Làm sao em xử lý cùng một khái niệm "Sản phẩm" hoặc "Đơn hàng" xuất hiện ở nhiều context khác nhau?
* 💡 **Câu trả lời ăn điểm**:
  * **Phân chia 4 Bounded Contexts cốt lõi**:
    1. **OMS (Order Management System Context)**: Quản lý vòng đời đơn hàng, chính sách giảm giá, thanh toán từ các kênh TikTok Shop/Shopee.
    2. **WMS (Warehouse Management System Context)**: Quản lý vị trí lưu kho vật lý, số lượng tồn thực tế, luân chuyển kho và đóng gói hàng.
    3. **Marketplace Integration Context**: Giao tiếp với Open API của các sàn (OAuth2 token, Webhook, Rate limiting).
    4. **Reconciliation Context (Đối soát tài chính)**: Đối soát dòng tiền thanh toán và phí sàn.
  * **Xử lý khái niệm "Sản phẩm" ở các Context**:
    * Trong DDD, **không bao giờ dùng một Model `Product` khổng lồ dùng chung cho toàn bộ hệ thống**.
    * Tại **OMS Context**: `OrderProduct` là một Value Object chỉ chứa `skuId, name, sellPrice, quantity`.
    * Tại **WMS Context**: `InventoryItem` là một Entity chứa `skuId, barcode, weightGrams, dimensions, binLocationId`.
    * Hai context hoàn toàn độc lập về mặt codebase, chỉ liên kết với nhau thông qua mã định danh bất biến chung là **`skuId`**.

---

### Case 2: Anti-Corruption Layer (ACL) cho TikTok Shop Open API
> **Trích dẫn CV**: *"Developed bi-directional TikTok Shop Open API integrations handling OAuth2 token lifecycles, webhook event ingestion..."*

#### Q2: Tại sao em lại cần xây dựng Anti-Corruption Layer (ACL) khi tích hợp với các sàn thương mại điện tử như TikTok Shop? Nếu không có ACL thì hệ thống gặp rủi ro gì?
* 💡 **Câu trả lời ăn điểm**:
  * **Rủi ro nếu không có ACL**:
    * TikTok Shop hoặc Shopee có cấu trúc dữ liệu JSON webhook rất dị biệt, thường xuyên thay đổi phiên bản API (API Versioning, breaking changes), và tên trường không theo chuẩn nghiệp vụ của công ty (ví dụ: `order_line_item_list` vs `items`).
    * Nếu đưa trực tiếp payload của TikTok Shop vào tầng Core Domain hoặc Service, mô hình nghiệp vụ nội bộ sẽ bị "ô nhiễm" (Corrupted). Mỗi khi TikTok Shop đổi API, toàn bộ mã nguồn nghiệp vụ của ViShop đều bị vỡ và phải sửa lại.
  * **Triển khai Anti-Corruption Layer**:
    1. **Tầng Adapter bên ngoài**: Nhận webhook từ TikTok Shop, validate signature và parse payload của TikTok.
    2. **Tầng Translator (ACL)**: Ánh xạ (Map) cấu trúc dữ liệu của TikTok Shop sang **Clean Domain Command/Contract nội bộ** của ViShop:
       ```typescript
       // tiktok-order.translator.ts
       export class TikTokOrderAclTranslator {
         static toCreateOrderCommand(rawPayload: TikTokWebhookDto): CreateOrderCommand {
           return new CreateOrderCommand({
             externalOrderId: rawPayload.data.order_id,
             channel: SalesChannel.TIKTOK_SHOP,
             items: rawPayload.data.item_list.map((i) => ({
               skuId: i.seller_sku,
               quantity: i.quantity,
               price: Money.create(i.sku_display_price, 'VND'),
             })),
           });
         }
       }
       ```
    * Giúp Domain Model của ViShop hoàn toàn miễn nhiễm với mọi thay đổi từ các đối tác bên thứ ba.

---

### Case 3: Domain Contracts Decoupling trong Multimodal RAG Platform
> **Trích dẫn CV**: *"Architected a scalable uv Python Monorepo adopting Domain-Driven Design, decoupling shared domain contracts, document processing pipeline, core RAG engine... completely isolate heterogeneous layout parsing from downstream retrieval and embedding."*

#### Q3: Trong dự án RAG đa phương thức, em đã áp dụng DDD để tách biệt tầng phân tích Layout tài liệu (Layout Parsing) với tầng Vector Retrieval & Embedding như thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Thách thức**: Quá trình bóc tách tài liệu phức tạp (PDF, bảng biểu đa tầng, hình ảnh raster/bitmap) sử dụng nhiều engine khác nhau (OpenDataLoader-PDF, Vision Models) tạo ra các định dạng dữ liệu thô rất phức tạp. Nếu tầng RAG Engine và Vector Database (pgvector) phụ thuộc trực tiếp vào cấu trúc của parser, việc thay đổi hoặc nâng cấp parser sẽ làm hỏng toàn bộ pipeline retrieval.
  * **Áp dụng DDD Domain Contracts**:
    * Xây dựng module `shared-domain-contracts` độc lập: Định nghĩa các **Domain Value Objects** chuẩn mực như `DocumentChunk`, `BoundingBox`, `SerializedTable`, `ChunkMetadata`.
    * Parser Pipeline chỉ chịu trách nhiệm chuyển đổi các tài liệu thô thành các `DocumentChunk` Value Objects hợp lệ.
    * Core RAG Engine và Embedding Service chỉ giao tiếp thông qua các Interface và Contracts này mà hoàn toàn không cần biết tài liệu gốc ban đầu được parse bởi thư viện nào.

---

## PHẦN 2: BỘ CÂU HỎI VỀ TACTICAL DESIGN & INVARIANTS

### Q4: Quy tắc "Một Transaction chỉ được sửa DUY NHẤT một Aggregate Root" có ý nghĩa gì? Nếu một nghiệp vụ bắt buộc phải cập nhật cả Order và Customer thì em làm thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Ý nghĩa của quy tắc**: Aggregate Root là ranh giới cô lập tính nhất quán (Consistency Boundary). Nếu một transaction mở rộng ra nhiều Aggregate, ranh giới khóa dữ liệu sẽ phình to, dẫn đến xung đột locking, deadlock diện rộng và không thể scale ngang trên môi trường phân tán.
  * **Cách xử lý bài toán cập nhật 2 Aggregates**:
    * Áp dụng nguyên tắc **Eventual Consistency (Nhất quán cuối cùng)**:
      1. **Bước 1 (Transaction 1 - ACID)**: Chỉnh sửa `Order` Aggregate, lưu vào DB và đồng thời ghi nhận sự kiện `OrderPlacedDomainEvent` vào bảng **Transactional Outbox**.
      2. **Bước 2 (Asynchronous Relay)**: Worker đọc sự kiện từ Outbox và publish lên Message Bus (BullMQ/RabbitMQ).
      3. **Bước 3 (Transaction 2 - ACID)**: Consumer nhận sự kiện, nạp `Customer` Aggregate từ DB lên, gọi method `customer.accumulatePoints(orderTotal)` và commit transaction thứ hai độc lập.

---

### Q5: Thế nào là Anemic Domain Model (Mô hình thiếu máu)? Tại sao các dự án quy mô lớn rất sợ rơi vào mô hình này?
* 💡 **Câu trả lời ăn điểm**:
  * **Nhận diện Anemic Model**: Các class Entity chỉ là các túi chứa dữ liệu (Data Bags) chứa hàng loạt public getter/setter, không hề có bất kỳ business logic hay phương thức kiểm tra hợp lệ nào.
  * **Hậu quả nghiêm trọng**:
    * Toàn bộ logic nghiệp vụ bị phân tán rải rác ở hàng chục file `Service` khác nhau.
    * Khi cần thay đổi một quy tắc (ví dụ: "Đơn hàng trên 5 triệu phải được quản lý duyệt"), lập trình viên phải tìm và sửa ở khắp các Service, rất dễ bỏ sót.
    * Phá vỡ hoàn toàn tính đóng gói (Encapsulation) của hướng đối tượng.
  * **Rich Domain Model (Giải pháp của DDD)**:
    * Giấu kín trạng thái nội bộ (`private`).
    * Mọi biến đổi trạng thái đều phải thông qua các hàm nghiệp vụ có danh xưng rõ ràng: `order.applyVoucher(code)`, `order.dispatch()`, nơi tự thân Aggregate sẽ tự kiểm tra xem mình có thỏa mãn các quy tắc bất biến (Invariants) hay không.
