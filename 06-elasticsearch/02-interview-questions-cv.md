# 🎯 CHUYÊN ĐỀ 06.2: ELASTICSEARCH – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu gắn liền với dự án **JIHO E-Learning & E-Book Ecosystem**: **Full-text Search khóa học y khoa & sách điện tử**, **Đồng bộ PostgreSQL sang Elasticsearch**, **Thuật toán BM25 vs TF-IDF**, và **Tối ưu Filter Context vs Query Context**.

---

## PHẦN 1: DỰ ÁN JIHO – E-LEARNING & E-BOOK SEARCH ARCHITECTURE

### Case 1: Tìm kiếm Khóa học & Sách Điện tử Y khoa Chuyên sâu
> **Trích dẫn CV**: *"Technologies: Node.js, Express.js, TypeScript, Next.js, PostgreSQL, Redis, BullMQ, Prisma, Elasticsearch... Implemented Redis caching and Elasticsearch to improve API performance and search capabilities."*

#### Q1: Khi tìm kiếm khóa học và tài liệu y khoa trong dự án JIHO, em thiết kế Index Mapping và Custom Analyzer như thế nào để hỗ trợ tìm kiếm không dấu, có dấu và từ đồng nghĩa y khoa?
* 💡 **Câu trả lời ăn điểm**:
  * **Thách thức**: Người dùng Nhật Bản/Việt Nam có thể gõ tiếng Việt không dấu, viết tắt thuật ngữ chuyên ngành y khoa, hoặc gõ sai chính tả một vài ký tự.
  * **Thiết kế Custom Analyzer trong Elasticsearch**:
    1. **Char Filters**: Chuẩn hóa ký tự Unicode (NFC), loại bỏ ký tự rác HTML.
    2. **Tokenizer**: Sử dụng `standard` hoặc `icu_tokenizer` để bóc tách từ tố chính xác.
    3. **Token Filters**:
       * `lowercase`: Đưa về chữ thường.
       * `asciifolding`: Tự động chuyển đổi ký tự có dấu sang không dấu (giúp tìm `tieu duong` vẫn khớp với `tiểu đường`).
       * `synonym`: Nạp từ điển đồng nghĩa chuyên ngành y tế (ví dụ: `đái tháo đường, tiểu đường`, `HA, huyết áp`).
  * **Cấu hình Multi-Match Query kết hợp Fuzziness**:
    ```json
    POST /medical_courses/_search
    {
      "query": {
        "bool": {
          "must": [
            {
              "multi_match": {
                "query": "kháng sinh đồ",
                "fields": ["title^3", "description^1", "keywords^2"],
                "fuzziness": "AUTO",
                "prefix_length": 2
              }
            }
          ],
          "filter": [
            { "term": { "status": "PUBLISHED" } },
            { "term": { "category_id": "clinical_training" } }
          ]
        }
      }
    }
    ```
    *(Tăng trọng số `^3` cho trường `title` để đảm bảo bài học có tiêu đề khớp sẽ đứng đầu danh sách kết quả).*

---

### Case 2: Đồng bộ Dữ liệu từ PostgreSQL sang Elasticsearch trong JIHO
> **Trích dẫn CV**: *"Integrated PostgreSQL/Prisma for transactional data and AWS S3 for digital content storage... Elasticsearch to improve API performance and search capabilities."*

#### Q2: Làm thế nào em đồng bộ dữ liệu sách/khóa học từ PostgreSQL sang Elasticsearch mà không làm chậm API và đảm bảo không bị mất mát dữ liệu khi server crash?
* 💡 **Câu trả lời ăn điểm**:
  * **Tại sao không Dual-Write trực tiếp trong API Handler**:
    * Nếu trong Controller vừa gọi Prisma `prisma.course.update()` vừa gọi `elasticsearch.update()`, nếu bước thứ hai timeout mạng thì dữ liệu giữa DB và Search Engine bị lệch nhau vĩnh viễn (Inconsistent).
  * **Mô hình Đồng bộ Bất đồng bộ với BullMQ**:
    1. Khi có sự kiện `CourseUpdated`, API lưu vào PostgreSQL và đẩy một event vào BullMQ `elasticsearch-sync-queue`.
    2. Worker bốc event và gọi **Bulk API (`_bulk`)** của Elasticsearch để cập nhật theo lô (Micro-batching 50 items/lần), giúp tối đa hóa throughput I/O của Elasticsearch.
    3. Nếu Elasticsearch bận hoặc lỗi, BullMQ tự động retry theo cơ chế Exponential Backoff.
  * **Giải pháp Zero-Downtime Reindexing (Index Alias Pattern)**:
    * Khi cần thay đổi Mapping (ví dụ đổi analyzer), không bao giờ xóa index đang chạy trực tiếp.
    * Tạo index mới `medical_courses_v2`, chạy script chuyển dữ liệu qua `_reindex`.
    * Sau khi index mới sẵn sàng, thực hiện hoán đổi Atomic Alias:
      ```json
      POST /_aliases
      {
        "actions": [
          { "remove": { "index": "medical_courses_v1", "alias": "medical_courses" } },
          { "add":    { "index": "medical_courses_v2", "alias": "medical_courses" } }
        ]
      }
      ```
      *(Client API luôn query vào alias `medical_courses`, người dùng hoàn toàn không cảm nhận thấy downtime).*

---

## PHẦN 2: BỘ CÂU HỎI VỀ ELASTICSEARCH INTERNALS & TUNING

### Q3: Phân biệt sự khác biệt cốt lõi giữa Filter Context và Query Context trong Elasticsearch. Tại sao Senior Developer luôn ưu tiên dùng Filter Context?
* 💡 **Câu trả lời ăn điểm**:
  * **Query Context (Mệnh đề `must`, `should`)**:
    * Trả lời câu hỏi: *"Tài liệu này khớp với điều kiện ở mức độ nào?"*.
    * **Tính toán Relevance Score (`_score`)**: Tiêu tốn chu kỳ CPU để chạy thuật toán BM25.
    * **Không được Cache**: Điểm số phụ thuộc vào từng từ khóa cụ thể nên không thể tái sử dụng cache.
  * **Filter Context (Mệnh đề `filter`, `must_not`)**:
    * Trả lời câu hỏi nhị phân: *"Tài liệu này có thỏa mãn điều kiện hay không (Yes/No)?"*.
    * **Không tính điểm số (`_score = 0`)**: Giảm tải CPU đáng kể.
    * **Tự động được Cache (Bitset Cache)**: Elasticsearch tự động lưu kết quả các filter phổ biến vào bộ nhớ RAM dưới dạng Roaring Bitmap $\rightarrow$ Các truy vấn sau kiểm tra cùng điều kiện chạy với tốc độ tức thì ($O(1)$).
  * **Quy tắc vàng**: Mọi điều kiện lọc trạng thái, ngày tháng, danh mục, phân quyền tenant bắt buộc phải đặt trong mệnh đề **`filter`**.

---

### Q4: Thuật toán Okapi BM25 hoạt động ra sao và nó khắc phục được nhược điểm gì của TF-IDF cổ điển?
* 💡 **Câu trả lời ăn điểm**:
  * **Hạn chế của TF-IDF**: Điểm số tăng tuyến tính theo tần suất xuất hiện của từ (Term Frequency - TF). Nếu một từ xuất hiện 100 lần trong bài viết, điểm số sẽ gấp 10 lần bài viết xuất hiện 10 lần $\rightarrow$ Dễ bị spam từ khóa (Keyword Stuffing).
  * **Hai cải tiến đột phá của Okapi BM25**:
    1. **Bão hòa tần suất từ (Term Frequency Saturation)**: Điểm số của BM25 tiệm cận một đường cong bão hòa (ngưỡng trần). Khi số lần xuất hiện của từ tăng lên từ 10 lên 50, điểm số tăng rất ít; vượt qua một ngưỡng nhất định, điểm số gần như đi ngang.
    2. **Chuẩn hóa độ dài văn bản (Document Length Normalization)**: BM25 so sánh độ dài của văn bản hiện tại với độ dài trung bình của toàn bộ các văn bản trong index. Nếu từ khóa xuất hiện trong một tiêu đề ngắn 5 từ, nó sẽ có trọng số cao hơn nhiều so với việc xuất hiện trong một cuốn sách dày 500 trang.
