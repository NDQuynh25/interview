# 🎯 CHUYÊN ĐỀ 06.2: ELASTICSEARCH – BỘ CÂU HỎI PHỎNG VẤN SENIOR & ÁP DỤNG DỰ ÁN THỰC CHIẾN TRONG CV

> **Ứng viên**: Nguyễn Đức Quỳnh – Backend / Fullstack Engineer  
> **Mục tiêu**: Bộ câu hỏi phỏng vấn chuyên sâu gắn liền với dự án **JIHO E-Learning & E-Book Ecosystem**: Nền tảng y tế phục vụ **thị trường Nhật Bản (Japanese Healthcare Market)**. Trọng tâm đào sâu: **Xử lý ngôn ngữ tự nhiên tiếng Nhật (Kuromoji vs N-gram)**, **Đồng bộ PostgreSQL sang Elasticsearch**, **Thuật toán BM25 vs TF-IDF**, và **Tối ưu hóa Filter Context**.

---

## PHẦN 1: DỰ ÁN JIHO – XỬ LÝ TÌM KIẾM TIẾNG NHẬT (JAPANESE NLP & ELASTICSEARCH)

### Case 1: Thách thức Ngôn ngữ phi Latinh trong Nền tảng Y tế Nhật Bản
> **Trích dẫn CV**: *"Enterprise medical E-Learning and digital publishing platform for the Japanese healthcare market, supporting B2B clinical training and B2C medical credentialing... Implemented Redis caching and Elasticsearch to improve API performance and search capabilities."*

#### Q1: Tiếng Nhật có cấu trúc hoàn toàn khác biệt với ngôn ngữ Latinh (không có khoảng trắng, dùng 4 hệ chữ viết Kanji, Hiragana, Katakana, Romaji). Trong dự án JIHO, em đã thiết kế Elasticsearch Analyzer như thế nào để tìm kiếm chính xác tài liệu y khoa?
* 💡 **Câu trả lời ăn điểm**:
  * **4 Thách thức đặc thù của tiếng Nhật so với tiếng Latinh**:
    1. **Không có dấu cách phân tách từ (No Whitespace Word Delimiter)**: Câu tiếng Nhật viết liền một mạch (`東京都の医師` - Bác sĩ tại Tokyo). Tokenizer mặc định của Lucene (`standard`) hoàn toàn bất lực vì nó chỉ biết cắt theo khoảng trắng.
    2. **Hỗn hợp 4 hệ chữ trong cùng 1 câu**: Kanji (`臨床` - Lâm sàng), Hiragana (`の`, `について`), Katakana (`ワクチン` - Vaccine), Romaji/English (`COVID-19`, `JIHO`).
    3. **Tìm kiếm theo cách đọc (Pronunciation / Reading search)**: Người dùng có thể gõ cách đọc bằng Hiragana (`りんしょう`) hoặc Romaji (`rinshou`) để tìm từ gốc chữ Kanji `臨床`.
    4. **Biến thể chữ (Orthographic & Width variation)**: Ký tự Half-width (`ｶﾀｶﾅ`, `123`) và Full-width (`カタカナ`, `１２３`).
  * **Giải pháp: Xây dựng Custom Analyzer với Plugin Kuromoji & ICU**:
    1. **Char Filter (`icu_normalizer`)**: Chuẩn hóa toàn bộ ký tự Half-width sang Full-width Katakana (Unicode NFKC), chuẩn hóa số và chữ cái.
    2. **Tokenizer (`kuromoji_tokenizer` với `mode: "search"`)**:
       * Sử dụng thuật toán phân tích hình thái học (Morphological Analysis) bóc tách từ vựng dựa trên từ điển MeCab.
       * Dùng `mode: "search"` để tự động chia nhỏ các từ ghép y khoa phức tạp (ví dụ: `急性虫垂炎` - Viêm ruột thừa cấp $\rightarrow$ bẻ nhỏ thành `急性` + `虫垂炎`), giúp tăng độ bao phủ (Recall).
    3. **Token Filters**:
       * `kuromoji_baseform`: Đưa động từ/tính từ biến đổi đuôi về thể nguyên thể từ điển (Dictionary Form: `診察した` $\rightarrow$ `診察する`).
       * `kuromoji_part_of_speech`: Lọc bỏ các trợ từ ngữ pháp vô nghĩa (`は`, `が`, `を`, `に`).
       * `kuromoji_readingform`: Chuyển đổi Kanji sang cách đọc Katakana lưu vào sub-field `reading` để phục vụ tìm kiếm theo phát âm.
       * `kuromoji_stemmer`: Chuẩn hóa trường âm Katakana (`コンピューター` $\rightarrow$ `コンピュータ`).

```json
PUT /jiho_medical_catalog
{
  "settings": {
    "analysis": {
      "char_filter": {
        "normalize_chars": {
          "type": "icu_normalizer",
          "name": "nfkc"
        }
      },
      "filter": {
        "kuromoji_pos_filter": {
          "type": "kuromoji_part_of_speech",
          "stoptags": ["助词", "助动词", "记号"]
        },
        "kuromoji_reading": {
          "type": "kuromoji_readingform",
          "use_romaji": false
        }
      },
      "analyzer": {
        "japanese_morph_analyzer": {
          "type": "custom",
          "char_filter": ["normalize_chars"],
          "tokenizer": "kuromoji_tokenizer",
          "filter": ["kuromoji_baseform", "kuromoji_pos_filter", "lowercase"]
        },
        "japanese_reading_analyzer": {
          "type": "custom",
          "char_filter": ["normalize_chars"],
          "tokenizer": "kuromoji_tokenizer",
          "filter": ["kuromoji_reading", "lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "japanese_morph_analyzer",
        "fields": {
          "reading": {
            "type": "text",
            "analyzer": "japanese_reading_analyzer"
          },
          "raw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

---

#### Q2: So sánh Morphological Analysis (Kuromoji) và N-gram (CJK Bi-gram) khi tìm kiếm tiếng Nhật? Tại sao trong JIHO em lại kết hợp cả hai (Hybrid Approach)?
* 💡 **Câu trả lời ăn điểm**:

| Tiêu chí | Morphological Analysis (Kuromoji) | N-gram (CJK Bi-gram) |
| :--- | :--- | :--- |
| **Cơ chế** | Phân tích ngữ pháp dựa trên từ điển MeCab. Tách đúng các từ có nghĩa. | Cắt cố định mỗi cụm 2 ký tự liên tiếp (`東京都` $\rightarrow$ `東京`, `京都`). Không cần từ điển. |
| **Độ chính xác (Precision)** | Cực cao. Không bị tìm kiếm sai ngữ cảnh. | Dễ bị **False Positive** (Ví dụ tìm `京都` - Kyoto có thể trả về văn bản chứa `東京都` - Tokyo!). |
| **Độ bao phủ (Recall)** | Phụ thuộc vào từ điển. Nếu gặp từ mới, thuật ngữ thuốc mới chưa có trong từ điển sẽ bị cắt sai. | **Recall 100%**. Bắt được mọi từ mới, tên riêng, mã thuốc, từ viết tắt. |
| **Kích thước Index** | Gọn nhẹ, tiết kiệm RAM. | Kích thước Index phình to gấp 2–3 lần. |

* **Chiến lược Hybrid của em trong dự án JIHO**:
  * Trường `title` và `content`: Đánh index chính bằng **Kuromoji** (`japanese_morph_analyzer`) để đảm bảo điểm liên quan (Relevance Score) chuẩn ngữ nghĩa.
  * Đính kèm sub-field `title.ngram` sử dụng CJK Bi-gram.
  * Khi người dùng tìm kiếm: Sử dụng `bool query` kết hợp `should`:
    * Khớp trên Kuromoji được nhân trọng số cao: `title^3`.
    * Khớp trên N-gram đóng vai trò cứu cánh dự phòng: `title.ngram^1`.
  * $\rightarrow$ Đảm bảo **vừa hiểu đúng ngữ pháp tiếng Nhật**, vừa **không bao giờ bỏ sót các thuật ngữ y khoa mới xuất hiện**!

---

#### Q3: Trong lĩnh vực Y tế (Medical), có rất nhiều tên thuốc, bệnh lý và thuật ngữ đặc thù không có trong từ điển Kuromoji mặc định. Em giải quyết bài toán "Cắt sai từ" (Segmentation Errors) như thế nào?
* 💡 **Câu trả lời ăn điểm**:
  * **Vấn đề**: Kuromoji sử dụng từ điển IPADIC phổ thông. Khi gặp một thuật ngữ y khoa chuyên sâu như `抗血小板薬` (Thuốc chống kết tập tiểu cầu), nếu không có trong từ điển, Kuromoji sẽ cắt vụn thành từng từ đơn vô nghĩa: `抗` + `血` + `小板` + `薬` $\rightarrow$ Làm giảm điểm tìm kiếm và sai lệch ngữ nghĩa.
  * **Giải pháp: Custom User Dictionary (`user_dictionary`)**:
    * Elasticsearch cho phép nạp thêm file từ điển tùy biến vào `kuromoji_tokenizer`:
      ```text
      # medical_user_dict.txt
      # Cấu trúc: <Từ gốc>,<Các từ tố con khi chia>,<Cách đọc Katakana>,<Từ loại>
      抗血小板薬,抗 血小板 薬,コウケッショウバンヤク,CustomMedicalTerm
      内視鏡下手術,内視鏡 下 手術,ナイシキョウカシュジュツ,CustomMedicalTerm
      ```
    * Cấu hình trong Elasticsearch settings:
      ```json
      "tokenizer": {
        "kuromoji_medical_tokenizer": {
          "type": "kuromoji_tokenizer",
          "mode": "search",
          "user_dictionary": "analysis/medical_user_dict.txt"
        }
      }
      ```
    * Giúp hệ thống nhận diện chính xác 100% các thuật ngữ y khoa phức tạp của thị trường Nhật Bản.

---

## PHẦN 2: ĐỒNG BỘ DỮ LIỆU & TỐI ƯU HỆ THỐNG TRONG JIHO

### Case 2: Đồng bộ Dữ liệu từ PostgreSQL sang Elasticsearch trong JIHO
> **Trích dẫn CV**: *"Integrated PostgreSQL/Prisma for transactional data and AWS S3 for digital content storage... Elasticsearch to improve API performance and search capabilities."*

#### Q4: Làm thế nào em đồng bộ dữ liệu sách/khóa học từ PostgreSQL sang Elasticsearch mà không làm chậm API và đảm bảo không bị mất mát dữ liệu khi server crash?
* 💡 **Câu trả lời ăn điểm**:
  * **Tại sao không Dual-Write trực tiếp trong API Handler**:
    * Nếu trong Controller vừa gọi Prisma `prisma.course.update()` vừa gọi `elasticsearch.update()`, nếu bước thứ hai timeout mạng thì dữ liệu giữa DB và Search Engine bị lệch nhau vĩnh viễn (Inconsistent).
  * **Mô hình Đồng bộ Bất đồng bộ với BullMQ**:
    1. Khi có sự kiện `CourseUpdated`, API lưu vào PostgreSQL và đẩy một event vào BullMQ `elasticsearch-sync-queue`.
    2. Worker bốc event và gọi **Bulk API (`_bulk`)** của Elasticsearch để cập nhật theo lô (Micro-batching 50 items/lần), giúp tối đa hóa throughput I/O của Elasticsearch.
    3. Nếu Elasticsearch bận hoặc lỗi, BullMQ tự động retry theo cơ chế Exponential Backoff.
  * **Giải pháp Zero-Downtime Reindexing (Index Alias Pattern)**:
    * Khi cần thay đổi Mapping hoặc nạp từ điển tiếng Nhật mới, không bao giờ xóa index đang chạy trực tiếp.
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

### Case 3: Tối ưu Query Context vs Filter Context trong Tìm kiếm Khóa học
> **Trích dẫn CV**: *"Refactored and optimized existing frontend/backend code to improve performance, reliability, and maintainability."*

#### Q5: Khi xây dựng trang Tìm kiếm & Lọc khóa học y khoa, em kết hợp Filter Context và Query Context như thế nào để tối ưu tốc độ CPU và tận dụng Cache?
* 💡 **Câu trả lời ăn điểm**:
  * **Nguyên tắc phân chia**:
    * **Query Context (`must`)**: Dành riêng cho ô tìm kiếm từ khóa người dùng gõ vào (`keyword`) để tính điểm số **BM25 Relevance Score** (`title`, `description`).
    * **Filter Context (`filter`)**: Dành cho tất cả các bộ lọc danh mục (`specialty_id`), trạng thái (`status = 'PUBLISHED'`), khoảng giá (`price_range`), cấp độ bác sĩ (`credential_level`).
  * **Lợi ích**:
    * Mọi điều kiện trong `filter` đều có điểm số `_score = 0`, không tốn CPU tính toán.
    * Elasticsearch tự động lưu kết quả các bộ lọc này vào **Bitset Cache (Roaring Bitmaps)** trong RAM $\rightarrow$ Khi nhiều học viên cùng chọn lọc theo chuyên khoa "Nội khoa", kết quả lọc được trả về tức thì ($O(1)$) từ RAM mà không cần quét lại đĩa.
