# BÁO CÁO KẾT QUẢ BÀI TẬP CÁ NHÂN — RAG PIPELINE V2

* **Học viên:** Vũ Nhật Anh
* **MSSV:** 2A202600894
* **Lớp:** Batch02 — AI Product Labs
* **Môn học:** Search Engine / RAG Chatbot
* **Trạng thái bộ test:** 35/35 Passed (100% thành công)

---

## TỔNG QUAN HỆ THỐNG
Hệ thống xây dựng một RAG Pipeline thực tế, end-to-end, từ thu thập dữ liệu pháp luật và báo chí về ma tuý → xử lý → indexing → retrieval (hybrid + vectorless fallback) → generation có citation.
1. **Thu thập tài liệu & báo chí**: Thu thập văn bản pháp luật và bài báo.
2. **Tiền xử lý văn bản**: Trích xuất nội dung sang định dạng Markdown chuẩn hóa.
3. **Chunking & Indexing**: Phân đoạn văn bản theo cơ chế sliding window và nhúng vector bằng mô hình `text-embedding-3-small` của OpenAI, sau đó lưu trữ tại Vector Store cục bộ.
4. **Hybrid Retrieval Pipeline**: Truy xuất song song bằng Semantic Search (truy vấn vector tương đồng cosine) và Lexical Search (thuật toán BM25), sau đó hợp nhất bằng RRF (Reciprocal Rank Fusion) và định vị thứ tự chống hiện tượng trôi thông tin (Lost in the middle).
5. **PageIndex Fallback**: Tự động chuyển hướng tìm kiếm sang PageIndex API trong trường hợp kết quả hybrid search không đạt yêu cầu về độ chính xác (score < threshold).
6. **Generation có Citation**: Sinh câu trả lời chất lượng cao tiếng Việt bằng GPT-4o-mini kèm trích dẫn nguồn cụ thể dưới dạng `[Tên tài liệu, Năm/Điều]`.

---

## CHI TIẾT TRIỂN KHAI CÁC NHIỆM VỤ (TASK 1 - 10)

### Task 1 — Thu Thập Văn Bản Pháp Luật
* **Mục tiêu**: Thu thập tối thiểu 3 văn bản pháp luật dưới dạng PDF/DOCX có liên quan đến ma túy và chất cấm.
* **Chi tiết triển khai**:
  * Thư mục đích: `data/landing/legal/`
  * Danh sách tài liệu thu thập:
    1. `luat-phong-chong-ma-tuy-2021.pdf`: Luật Phòng, chống ma tuý số 73/2021/QH15.
    2. `bo-luat-hinh-su-chong-ma-tuy-2015.pdf`: Chương XX Bộ luật Hình sự 2015 về tội phạm ma tuý.
    3. `nghi-dinh-chi-tiet-huong-dan-2021.pdf`: Nghị định 105/2021/NĐ-CP hướng dẫn Luật phòng chống ma tuý.
* **Kết quả**: Đã kiểm tra dung lượng hợp lệ (> 1KB per file) và lưu đúng cấu trúc quy định.

### Task 2 — Crawl Bài Báo
* **Mục tiêu**: Thu thập tối thiểu 5 bài viết/tin tức báo chí liên quan đến các nghệ sĩ Việt Nam dính líu đến ma túy.
* **Chi tiết triển khai**:
  * Thư mục đích: `data/landing/news/`
  * Sử dụng API/crawler thu thập bài viết lưu dưới dạng các file `.json`.
  * Metadata lưu trữ bao gồm: `url` (đường dẫn gốc), `title` (tiêu đề), và các nội dung văn bản thô phục vụ phân tích.
* **Kết quả**: Crawl thành công 5 bài báo và ghi nhận đầy đủ metadata cấu trúc JSON.

### Task 3 — Convert Sang Markdown
* **Mục tiêu**: Chuyển đổi định dạng các file thô (PDF, JSON) trong thư mục `landing` sang Markdown (`.md`) sạch để phục vụ phân tách nội dung.
* **Chi tiết triển khai**:
  * Sử dụng thư viện `MarkItDown` của Microsoft.
  * Thư mục lưu kết quả chuẩn hóa: `data/standardized/` (chia nhánh con tương tự thư mục gốc gồm `legal/` và `news/`).
* **Kết quả**: Trích xuất nội dung văn bản rõ ràng, loại bỏ định dạng thừa, file có độ dài tối thiểu > 200 ký tự.

### Task 4 — Chunking & Indexing
* **Mục tiêu**: Chia nhỏ văn bản và lập chỉ mục Vector tương đồng.
* **Cấu hình chi tiết**:
  * **Phương pháp Chunking**: Dịch chuyển cửa sổ (`sliding window` - recursive) giúp giữ tính liên kết văn bản.
  * **Tham số**: `CHUNK_SIZE = 500` ký tự, `CHUNK_OVERLAP = 50` ký tự.
  * **Embedding Model**: `text-embedding-3-small` (OpenAI, dimension = 1536).
  * **Vector Store**: Lưu trữ cục bộ dạng file JSON có cấu trúc tối ưu tại `data/vector_store.json` để dễ truy vấn offline và kiểm soát.
  * Có cơ chế tự động sleep và retry khi gặp lỗi giới hạn tần suất API (Rate Limit 429).
* **Kết quả**: Toàn bộ dữ liệu được nhúng và index thành công.

### Task 5 — Semantic Search Module
* **Mục tiêu**: Xây dựng module tìm kiếm ngữ nghĩa dùng vector embedding.
* **Chi tiết triển khai**:
  * Hàm đích: `semantic_search(query: str, top_k: int = 10) -> list[dict]`
  * Chuyển query đầu vào thành vector bằng mô hình nhúng `text-embedding-3-small`.
  * Tính toán độ tương đồng Cosine Similarity giữa vector câu truy vấn với vector của từng chunk dữ liệu trong Vector Store.
* **Kết quả**: Kết quả trả về gồm văn bản, metadata nguồn và điểm số tương đồng cosine, được sắp xếp giảm dần theo thứ tự điểm số.

### Task 6 — Lexical Search Module
* **Mục tiêu**: Xây dựng module tìm kiếm từ khóa chính xác bằng thuật toán BM25.
* **Chi tiết triển khai**:
  * Thư viện sử dụng: `rank-bm25` (`BM25Okapi`).
  * Thực hiện tiền xử lý văn bản, chuyển chữ thường (lowercase) và tokenize từ khóa thô của toàn bộ corpus.
* **Kết quả**: Tìm kiếm chuẩn xác theo số điều luật, thuật ngữ chuyên ngành ma túy chính xác cao. Chỉ trả về kết quả có `score > 0`.

### Task 7 — Reranking Module
* **Mục tiêu**: Chấm điểm lại độ tương quan các kết quả truy xuất lai (hybrid candidates) nhằm sắp xếp lại thứ tự tối ưu nhất.
* **Chi tiết triển khai**:
  * Hỗ trợ 3 phương pháp xếp hạng:
    1. **Cross-Encoder**: Kết nối `jina-reranker-v2-base-multilingual` qua API của Jina AI (hoặc dự phòng bằng model `BAAI/bge-reranker-base` chạy cục bộ).
    2. **MMR (Maximal Marginal Relevance)**: Giúp đa dạng hóa kết quả và giảm trùng lặp nội dung.
    3. **RRF (Reciprocal Rank Fusion)**: Hợp nhất thứ hạng kết quả từ dense search và lexical search.
* **Kết quả**: Sắp xếp chuẩn xác, loại bỏ nhiễu thông tin hiệu quả.

### Task 8 — PageIndex Vectorless RAG
* **Mục tiêu**: Tích hợp công cụ tìm kiếm Vectorless của PageIndex AI để dự phòng.
* **Chi tiết triển khai**:
  * Thư mục tích hợp: `src/task8_pageindex_vectorless.py`
  * Kết nối thông qua SDK `pageindex` và `PAGEINDEX_API_KEY`.
  * Hỗ trợ cơ chế tải tài liệu lên PageIndex và gọi truy vấn kết quả kèm thuộc tính `source = "pageindex"`.
* **Kết quả**: Hoạt động đồng bộ, tự động mock-up cục bộ nếu xảy ra sự cố kết nối nhằm đảm bảo hệ thống không bị crash.

### Task 9 — Retrieval Pipeline Hoàn Chỉnh
* **Mục tiêu**: Hợp nhất luồng truy xuất dữ liệu tối ưu có cơ chế Fallback dự phòng.
* **Chi tiết triển khai**:
  * Gọi song song truy vấn: Semantic Search (Dense) và Lexical Search (Sparse).
  * Hợp nhất kết quả bằng RRF và chấm điểm lại bằng bộ lọc Reranker.
  * **Fallback Logic**: Nếu kết quả hàng đầu của Hybrid Search có điểm tương quan dưới ngưỡng quy định (`score < 0.3`), hệ thống sẽ tự động chuyển đổi sang tìm kiếm thông tin bằng PageIndex API để đảm bảo phạm vi quét dữ liệu rộng hơn.
* **Kết quả**: Pipeline vận hành ổn định và trả về kết quả chính xác cao.

### Task 10 — Generation Có Citation
* **Mục tiêu**: Sinh câu trả lời tự nhiên kèm trích nguồn đáng tin cậy.
* **Chi tiết triển khai**:
  * Sử dụng kỹ thuật sắp xếp lại thứ tự chunks (`reorder_for_llm`) theo dạng đặt các thông tin quan trọng nhất ở đầu và cuối ngữ cảnh để triệt tiêu hiện tượng "lost in the middle" của LLM.
  * Inject ngữ cảnh vào Prompt, gọi API OpenAI GPT-4o-mini tạo văn bản phản hồi bằng Tiếng Việt.
  * Ràng buộc chặt chẽ: Nếu ngữ cảnh không chứa dữ liệu cần thiết, mô hình trả về: *"Tôi không thể xác minh thông tin này từ nguồn hiện có"* thay vì tự đoán (hallucination).
* **Kết quả**: Câu trả lời trơn tru, chuẩn xác và trích dẫn rõ tên tài liệu, điều luật sử dụng.

---

## KẾT QUẢ KIỂM THỬ TỰ ĐỘNG (PYTEST)
Để đảm bảo tất cả các Task cá nhân được cài đặt đúng đặc tả của bài tập, hệ thống đã chạy bộ test tự động và vượt qua tuyệt đối:

```text
============================= test session starts ==============================
platform darwin -- Python 3.12.2, pytest-9.0.3, pluggy-1.6.0
rootdir: /Users/HELLO/Project/lab-vinAI/2A202600894_VuNhatAnh_Day08
plugins: Faker-40.21.0, anyio-4.13.0, langsmith-0.8.8
collected 35 items

tests/test_individual.py::TestTask1::test_files_not_empty PASSED
tests/test_individual.py::TestTask1::test_landing_legal_dir_exists PASSED
tests/test_individual.py::TestTask1::test_minimum_3_legal_files PASSED
tests/test_individual.py::TestTask2::test_json_files_have_metadata PASSED
tests/test_individual.py::TestTask2::test_landing_news_dir_exists PASSED
tests/test_individual.py::TestTask2::test_minimum_5_news_files PASSED
tests/test_individual.py::TestTask2::test_news_files_have_content PASSED
tests/test_individual.py::TestTask3::test_converted_files_have_content PASSED
tests/test_individual.py::TestTask3::test_has_markdown_files PASSED
tests/test_individual.py::TestTask3::test_legal_and_news_both_converted PASSED
tests/test_individual.py::TestTask3::test_standardized_dir_exists PASSED
tests/test_individual.py::TestTask4::test_chunk_documents_produces_chunks PASSED
tests/test_individual.py::TestTask4::test_chunks_respect_size_limit PASSED
tests/test_individual.py::TestTask4::test_config_documented PASSED
tests/test_individual.py::TestTask4::test_load_documents_returns_list PASSED
tests/test_individual.py::TestTask5::test_respects_top_k PASSED
tests/test_individual.py::TestTask5::test_results_have_required_keys PASSED
tests/test_individual.py::TestTask5::test_results_sorted_descending PASSED
tests/test_individual.py::TestTask5::test_returns_list PASSED
tests/test_individual.py::TestTask6::test_keyword_match_scores_higher PASSED
tests/test_individual.py::TestTask6::test_results_have_required_keys PASSED
tests/test_individual.py::TestTask6::test_results_sorted_descending PASSED
tests/test_individual.py::TestTask6::test_returns_list PASSED
tests/test_individual.py::TestTask7::test_rerank_has_score PASSED
tests/test_individual.py::TestTask7::test_rerank_respects_top_k PASSED
tests/test_individual.py::TestTask7::test_rerank_returns_list PASSED
tests/test_individual.py::TestTask8::test_function_exists PASSED
tests/test_individual.py::TestTask8::test_returns_list_with_source_marker PASSED
tests/test_individual.py::TestTask9::test_fallback_logic_exists PASSED
tests/test_individual.py::TestTask9::test_respects_top_k PASSED
tests/test_individual.py::TestTask9::test_results_have_required_keys PASSED
tests/test_individual.py::TestTask9::test_retrieve_returns_list PASSED
tests/test_individual.py::TestTask10::test_format_context_includes_source PASSED
tests/test_individual.py::TestTask10::test_generate_returns_dict_with_answer PASSED
tests/test_individual.py::TestTask10::test_reorder_function_exists PASSED

============================= 35 passed in 30.15s ==============================
```

---
*Báo cáo được chuẩn bị và kiểm tra tự động thành công bởi hệ thống trợ lý ảo Antigravity.*
