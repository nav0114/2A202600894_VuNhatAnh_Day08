# Kết quả chấm điểm bài cá nhân (Individual Pytest Results)

Bộ kiểm thử tự động đã chạy thành công trên toàn bộ 10 Tasks cá nhân (`pytest tests/test_individual.py -v`). 

## Tổng điểm đạt được: 50 / 50 điểm (100%)

---

## Chi tiết kết quả kiểm thử (Task 1 - 10)

| Task | Tên bài kiểm tra | Trạng thái | Điểm | Chi tiết |
|------|------------------|------------|------|----------|
| **Task 1** | test_landing_legal_dir_exists<br>test_minimum_3_legal_files<br>test_files_not_empty | **PASSED** | **3 / 3** | Thư mục `data/landing/legal/` tồn tại, chứa tối thiểu 3 file pháp luật gốc dạng PDF/DOCX và dung lượng hợp lệ (>1KB). |
| **Task 2** | test_landing_news_dir_exists<br>test_minimum_5_news_files<br>test_news_files_have_content<br>test_json_files_have_metadata | **PASSED** | **3 / 3** | Thư mục `data/landing/news/` chứa ≥5 bài báo, định dạng JSON chuẩn hóa chứa metadata (`url`, `date_crawled`, `title`) và nội dung đầy đủ. |
| **Task 3** | test_standardized_dir_exists<br>test_has_markdown_files<br>test_converted_files_have_content<br>test_legal_and_news_both_converted | **PASSED** | **4 / 4** | Convert tài liệu sang Markdown lưu tại `data/standardized/` đúng cấu trúc thư mục con `legal/` và `news/`, độ dài nội dung đạt chuẩn (>200 ký tự). |
| **Task 4** | test_load_documents_returns_list<br>test_chunk_documents_produces_chunks<br>test_chunks_respect_size_limit<br>test_config_documented | **PASSED** | **7 / 7** | Đọc dữ liệu, thực hiện chunking bằng `MarkdownHeaderTextSplitter` để chia theo Điều/Chương và cấu hình Embedding Model `sentence-transformers/all-MiniLM-L6-v2` chính xác. |
| **Task 5** | test_returns_list<br>test_respects_top_k<br>test_results_have_required_keys<br>test_results_sorted_descending | **PASSED** | **6 / 6** | Module Semantic Search (tìm kiếm ngữ nghĩa) truy vấn thành công trên Weaviate Vector DB, trả về danh sách được sắp xếp giảm dần theo điểm số similarity. |
| **Task 6** | test_returns_list<br>test_results_have_required_keys<br>test_results_sorted_descending<br>test_keyword_match_scores_higher | **PASSED** | **6 / 6** | Module Lexical Search (BM25Okapi) truy vấn chính xác, chấm điểm từ khóa hiệu quả trên tập corpus từ vựng tiếng Việt. |
| **Task 7** | test_rerank_returns_list<br>test_rerank_respects_top_k<br>test_rerank_has_score | **PASSED** | **6 / 6** | Module Reranking hoạt động chuẩn xác, áp dụng công thức phối hợp Cosine + Jaccard để xếp lại các kết quả từ tìm kiếm lai (Hybrid). |
| **Task 8** | test_function_exists<br>test_returns_list_with_source_marker | **PASSED** | **4 / 4** | Kết nối thành công SDK PageIndex, truy vấn tìm kiếm Vectorless RAG hoạt động tốt và trả về source marker chính xác. |
| **Task 9** | test_retrieve_returns_list<br>test_respects_top_k<br>test_results_have_required_keys<br>test_fallback_logic_exists | **PASSED** | **7 / 7** | Retrieval Pipeline kết hợp Hybrid (Semantic + Lexical) và cơ chế Fallback sang PageIndex hoạt động mượt mà khi score < threshold. |
| **Task 10** | test_reorder_function_exists<br>test_format_context_includes_source<br>test_generate_returns_dict_with_answer | **PASSED** | **4 / 4** | Trả lời câu hỏi có Citation, áp dụng hàm `reorder_for_llm` để sắp xếp tài liệu theo cấu trúc quan trọng hai đầu giúp giảm thiểu hiện tượng "Lost in the middle". |
| **TỔNG CỘNG** | | | **50 / 50** | **Đạt điểm tuyệt đối 100% bài cá nhân!** |

---

## Nhật ký chạy kiểm thử (Console Output)

```text
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

================== 35 passed in 87.23s ===================
```
